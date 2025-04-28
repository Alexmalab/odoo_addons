# Odoo Module: event_booth

Category: Marketing/Events

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Events Booths",
    'category': 'Marketing/Events',
    'version': '1.0',
    'summary': "Manage event booths",
    'description': """
Create booths for your favorite event.
    """,
    'depends': ['event'],
    'data': [
        'security/ir.model.access.csv',
        'views/event_booth_category_views.xml',
        'views/event_type_booth_views.xml',
        'views/event_booth_views.xml',
        'views/event_type_views.xml',
        'views/event_event_views.xml',
        'views/event_menus.xml',
        'data/event_booth_category_data.xml',
        'data/mail_message_subtype_data.xml',
        'data/mail_templates.xml',
    ],
    'demo': [
        'data/event_booth_demo.xml',
        'data/event_type_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\event_booth_category_data.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data noupdate="1">

    <record id="event_booth_category_standard" model="event.booth.category">
        <field name="name">Standard Booth</field>
        <field name="sequence">1</field>
        <field name="image_1920" type="base64" file="event_booth/static/src/img/standard-booth.jpeg"/>
        <field name="description" type="html">
            <div class="card text-center shadow-sm mt32">
                <div class="o_wevent_theme_bg_dark card-header p-4">
                    <h4 class="text-white m-0">Standard</h4>
                </div>
                <ul class="list-group list-group-flush">
                    <li class="list-group-item">
                        <span>1 Branded Booth</span>
                    </li>
                    <li class="o_wevent_theme_bg_dark list-group-item">
                        <span class="text-white">4m²</span>
                    </li>
                    <li class="list-group-item">
                        <span>46" display screen</span>
                    </li>
                    <li class="list-group-item">
                        <span>1 desk</span>
                    </li>
                    <li class="list-group-item">
                        <span>Logo &amp; link on website</span>
                    </li>
                </ul>
            </div>
        </field>
    </record>

    <record id="event_booth_category_premium" model="event.booth.category">
        <field name="name">Premium Booth</field>
        <field name="sequence">2</field>
        <field name="image_1920" type="base64" file="event_booth/static/src/img/premium-booth.jpeg"/>
        <field name="description" type="html">
            <div class="card text-center shadow-sm mt32">
                <div class="card-header p-4 bg-secondary">
                    <h4 class="text-white m-0">Premium</h4>
                </div>
                <ul class="list-group list-group-flush">
                    <li class="list-group-item">
                        <span>1 Branded Booth</span>
                    </li>
                    <li class="list-group-item bg-secondary">
                        <span class="text-white">4m²</span>
                    </li>
                    <li class="list-group-item">
                        <span>46" display screen</span>
                    </li>
                    <li class="list-group-item">
                        <span>1 desk</span>
                    </li>
                    <li class="list-group-item">
                        <span>Logo &amp; link on website</span>
                    </li>
                    <li class="list-group-item">
                        <span>50 words description on website</span>
                    </li>
                    <li class="list-group-item">
                        <span>10 + 1 passes</span>
                    </li>
                </ul>
            </div>
        </field>
    </record>

    <record id="event_booth_category_vip" model="event.booth.category">
        <field name="name">VIP Booth</field>
        <field name="sequence">3</field>
        <field name="image_1920" type="base64" file="event_booth/static/src/img/vip-booth.jpeg"/>
        <field name="description" type="html">
            <div class="card text-center shadow-sm mt32">
                <div class="card-header p-4 bg-primary">
                    <h4 class="text-white m-0">VIP</h4>
                </div>
                <ul class="list-group list-group-flush">
                    <li class="list-group-item">
                        <span>2 Branded Booth</span>
                    </li>
                    <li class="list-group-item bg-primary">
                        <span class="text-white">8m²</span>
                    </li>
                    <li class="list-group-item">
                        <span>2 x 46" display screens</span>
                    </li>
                    <li class="list-group-item">
                        <span>2 desks</span>
                    </li>
                    <li class="list-group-item">
                        <span>Logo &amp; link on website</span>
                    </li>
                    <li class="list-group-item">
                        <span>100 words description on website</span>
                    </li>
                    <li class="list-group-item">
                        <span>10 + 1 passes</span>
                    </li>
                </ul>
            </div>
        </field>
    </record>

</data></odoo>

```

## File: data\event_booth_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <!-- EVENT BOOTH FOR "Design Fair Los Angeles" EVENT -->
    <record id="event_booth_0_event_0" model="event.booth">
        <field name="name">Booth A1</field>
        <field name="event_id" ref="event.event_0"/>
        <field name="booth_category_id" ref="event_booth_category_standard"/>
    </record>
    <record id="event_booth_1_event_0" model="event.booth">
        <field name="name">Booth A2</field>
        <field name="event_id" ref="event.event_0"/>
        <field name="booth_category_id" ref="event_booth_category_standard"/>
    </record>
    <record id="event_booth_2_event_0" model="event.booth">
        <field name="name">Booth A3</field>
        <field name="event_id" ref="event.event_0"/>
        <field name="booth_category_id" ref="event_booth_category_standard"/>
    </record>
    <record id="event_booth_3_event_0" model="event.booth">
        <field name="name">Premium Booth A4</field>
        <field name="event_id" ref="event.event_0"/>
        <field name="booth_category_id" ref="event_booth_category_premium"/>
    </record>
    <record id="event_booth_4_event_0" model="event.booth">
        <field name="name">VIP Booth A5</field>
        <field name="event_id" ref="event.event_0"/>
        <field name="booth_category_id" ref="event_booth_category_vip"/>
    </record>

    <!-- EVENT BOOTH FOR "Conference for Architects" EVENT -->
    <record id="event.event_2" model="event.event">
        <field name="event_booth_ids" eval="[(5, 0)]"/>
    </record>
    <record id="event_booth_0_event_2" model="event.booth">
        <field name="name">Showbooth 1</field>
        <field name="event_id" ref="event.event_2"/>
        <field name="booth_category_id" ref="event_booth_category_standard"/>
    </record>
    <record id="event_booth_1_event_2" model="event.booth">
        <field name="name">Showbooth 2</field>
        <field name="event_id" ref="event.event_2"/>
        <field name="booth_category_id" ref="event_booth_category_standard"/>
    </record>
    <record id="event_booth_2_event_2" model="event.booth">
        <field name="name">Premium Showbooth 1</field>
        <field name="event_id" ref="event.event_2"/>
        <field name="booth_category_id" ref="event_booth_category_premium"/>
    </record>
    <record id="event_booth_3_event_2" model="event.booth">
        <field name="name">Premium Showbooth 2</field>
        <field name="event_id" ref="event.event_2"/>
        <field name="booth_category_id" ref="event_booth_category_premium"/>
    </record>

    <!-- EVENT BOOTH FOR "OpenWood Collection Online Reveal" EVENT -->
    <record id="event_booth_00_event_7" model="event.booth">
        <field name="name">Booth A1</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="booth_category_id" ref="event_booth_category_standard"/>
    </record>
    <record id="event_booth_01_event_7" model="event.booth">
        <field name="name">Booth A2</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="booth_category_id" ref="event_booth_category_standard"/>
    </record>
    <record id="event_booth_02_event_7" model="event.booth">
        <field name="name">Booth A3</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="booth_category_id" ref="event_booth_category_standard"/>
    </record>
    <record id="event_booth_10_event_7" model="event.booth">
        <field name="name">OpenWood Demonstrator 1</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="booth_category_id" ref="event_booth_category_premium"/>
    </record>
    <record id="event_booth_11_event_7" model="event.booth">
        <field name="name">OpenWood Demonstrator 2</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="booth_category_id" ref="event_booth_category_premium"/>
    </record>
    <record id="event_booth_12_event_7" model="event.booth">
        <field name="name">OpenWood Demonstrator 3</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="booth_category_id" ref="event_booth_category_premium"/>
    </record>
    <record id="event_booth_20_event_7" model="event.booth">
        <field name="name">Gold Booth 1</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="booth_category_id" ref="event_booth_category_vip"/>
    </record>
    <record id="event_booth_21_event_7" model="event.booth">
        <field name="name">Gold Booth 2</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="booth_category_id" ref="event_booth_category_vip"/>
    </record>
    <record id="event_booth_22_event_7" model="event.booth">
        <field name="name">Gold Booth 3</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="booth_category_id" ref="event_booth_category_vip"/>
    </record>

</data></odoo>

```

## File: data\event_type_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_type_booth_demo_conference_0" model="event.type.booth">
        <field name="name">Showbooth 1</field>
        <field name="event_type_id" ref="event.event_type_data_conference"/>
        <field name="booth_category_id" ref="event_booth_category_standard"/>
    </record>
    <record id="event_type_booth_demo_conference_1" model="event.type.booth">
        <field name="name">Showbooth 2</field>
        <field name="event_type_id" ref="event.event_type_data_conference"/>
        <field name="booth_category_id" ref="event_booth_category_standard"/>
    </record>
    <record id="event_type_booth_demo_conference_2" model="event.type.booth">
        <field name="name">Premium Showbooth 1</field>
        <field name="event_type_id" ref="event.event_type_data_conference"/>
        <field name="booth_category_id" ref="event_booth_category_premium"/>
    </record>
    <record id="event_type_booth_demo_conference_3" model="event.type.booth">
        <field name="name">Premium Showbooth 2</field>
        <field name="event_type_id" ref="event.event_type_data_conference"/>
        <field name="booth_category_id" ref="event_booth_category_premium"/>
    </record>

</data></odoo>

```

## File: data\mail_message_subtype_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <record id="mt_event_booth_booked" model="mail.message.subtype">
        <field name="name">Booth Booked</field>
        <field name="res_model">event.event</field>
        <field name="default" eval="False"/>
    </record>

</data></odoo>

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <!-- This template is rendered on the event.event model without referring to it. -->
    <!-- Instead it shows details about event.booth model. -->
    <template id="event_booth_booked_template">
        <p>
            <span>
                Booth
                <a href="#" t-att-data-oe-model="booth._name" t-att-data-oe-id="booth.id" t-out="booth.name"/>
            </span>
            <span t-out="'booked by' if booth.partner_id else 'has been reserved by'"/>
            <span>
                <a href="#" t-att-data-oe-model="booth.partner_id._name or booth.env.user.partner_id._name"
                   t-att-data-oe-id="booth.partner_id.id or booth.env.user.partner_id.id"
                   t-out="booth.partner_id.name or booth.env.user.partner_id.name"/>
            </span>
        </p>
        <ul t-if="booth.partner_id" name="contact_details">
            <t t-set="contact_name" t-value="booth.contact_name"/>
            <li t-if="contact_name">
                <b>Renter Name</b>: <span t-out="contact_name"/>
            </li>
            <t t-set="contact_email" t-value="booth.contact_email"/>
            <li t-if="contact_email">
                <b>Renter Email</b>: <span t-out="contact_email"/>
            </li>
            <t t-set="contact_mobile" t-value="booth.contact_mobile"/>
            <li t-if="contact_mobile">
                <b>Renter Mobile</b>: <span t-out="contact_mobile"/>
            </li>
            <t t-set="contact_phone" t-value="booth.contact_phone"/>
            <li t-if="contact_phone">
                <b>Renter Phone</b>: <span t-out="contact_phone"/>
            </li>
        </ul>
    </template>

</data></odoo>

```

## File: models\event_booth.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.osv import expression


class EventBooth(models.Model):
    _name = 'event.booth'
    _description = 'Event Booth'
    _inherit = [
        'event.type.booth',
        'mail.thread',
        'mail.activity.mixin'
    ]

    # owner
    event_type_id = fields.Many2one(ondelete='set null', required=False)
    event_id = fields.Many2one('event.event', string='Event', ondelete='cascade', required=True)
    # customer
    partner_id = fields.Many2one('res.partner', string='Renter', tracking=True, copy=False)
    contact_name = fields.Char('Renter Name', compute='_compute_contact_name', readonly=False, store=True, copy=False)
    contact_email = fields.Char('Renter Email', compute='_compute_contact_email', readonly=False, store=True, copy=False)
    contact_mobile = fields.Char('Renter Mobile', compute='_compute_contact_mobile', readonly=False, store=True, copy=False)
    contact_phone = fields.Char('Renter Phone', compute='_compute_contact_phone', readonly=False, store=True, copy=False)
    # state
    state = fields.Selection(
        [('available', 'Available'), ('unavailable', 'Unavailable')],
        string='Status', group_expand='_group_expand_states',
        default='available', required=True, tracking=True)
    is_available = fields.Boolean(compute='_compute_is_available', search='_search_is_available')

    @api.depends('partner_id')
    def _compute_contact_name(self):
        for booth in self:
            if not booth.contact_name:
                booth.contact_name = booth.partner_id.name or False

    @api.depends('partner_id')
    def _compute_contact_email(self):
        for booth in self:
            if not booth.contact_email:
                booth.contact_email = booth.partner_id.email or False

    @api.depends('partner_id')
    def _compute_contact_mobile(self):
        for booth in self:
            if not booth.contact_mobile:
                booth.contact_mobile = booth.partner_id.mobile or False

    @api.depends('partner_id')
    def _compute_contact_phone(self):
        for booth in self:
            if not booth.contact_phone:
                booth.contact_phone = booth.partner_id.phone or False

    @api.depends('state')
    def _compute_is_available(self):
        for booth in self:
            booth.is_available = booth.state == 'available'

    def _search_is_available(self, operator, operand):
        negative = operator in expression.NEGATIVE_TERM_OPERATORS
        if (negative and operand) or not operand:
            return [('state', '=', 'unavailable')]
        return [('state', '=', 'available')]

    def _group_expand_states(self, states, domain, order):
        return [key for key, val in self._fields['state'].selection]

    @api.model_create_multi
    def create(self, vals_list):
        res = super(EventBooth, self.with_context(mail_create_nosubscribe=True)).create(vals_list)
        unavailable_booths = res.filtered(lambda booth: not booth.is_available)
        unavailable_booths._post_confirmation_message()
        return res

    def write(self, vals):
        to_confirm = self.filtered(lambda booth: booth.state == 'available')
        wpartner = {}
        if 'state' in vals or 'partner_id' in vals:
            wpartner = dict(
                (booth, booth.partner_id.ids)
                for booth in self.filtered(lambda booth: booth.partner_id)
            )

        res = super(EventBooth, self).write(vals)

        if vals.get('state') == 'unavailable' or vals.get('partner_id'):
            for booth in self:
                booth.message_subscribe(booth.partner_id.ids)
        for booth in self:
            if wpartner.get(booth) and booth.partner_id.id not in wpartner[booth]:
                booth.message_unsubscribe(wpartner[booth])

        if vals.get('state') == 'unavailable':
            to_confirm._action_post_confirm(vals)

        return res

    def _post_confirmation_message(self):
        for booth in self:
            booth.event_id.message_post_with_view(
                'event_booth.event_booth_booked_template',
                values={
                    'booth': booth,
                },
                subtype_id=self.env.ref('event_booth.mt_event_booth_booked').id,
            )

    def action_confirm(self, additional_values=None):
        write_vals = dict({'state': 'unavailable'}, **additional_values or {})
        self.write(write_vals)

    def _action_post_confirm(self, write_vals):
        self._post_confirmation_message()

```

## File: models\event_booth_category.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class EventBoothCategory(models.Model):
    _name = 'event.booth.category'
    _description = 'Event Booth Category'
    _inherit = ['image.mixin']
    _order = 'sequence ASC'

    active = fields.Boolean(default=True)
    name = fields.Char(string='Name', required=True, translate=True)
    sequence = fields.Integer(string='Sequence', default=10)
    description = fields.Html(string='Description', translate=True, sanitize_attributes=False)
    booth_ids = fields.One2many(
        'event.booth', 'booth_category_id', string='Booths', groups='event.group_event_registration_desk')

```

## File: models\event_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo import Command


class Event(models.Model):
    _inherit = 'event.event'

    event_booth_ids = fields.One2many(
        'event.booth', 'event_id', string='Booths', copy=True,
        compute='_compute_event_booth_ids', readonly=False, store=True)
    event_booth_count = fields.Integer(
        string='Total Booths',
        compute='_compute_event_booth_count')
    event_booth_count_available = fields.Integer(
        string='Available Booths',
        compute='_compute_event_booth_count')
    event_booth_category_ids = fields.Many2many(
        'event.booth.category', compute='_compute_event_booth_category_ids')
    event_booth_category_available_ids = fields.Many2many(
        'event.booth.category', compute='_compute_event_booth_category_available_ids',
        help='Booth Category for which booths are still available. Used in frontend')

    @api.depends('event_type_id')
    def _compute_event_booth_ids(self):
        """ Update event configuration from its event type. Depends are set only
        on event_type_id itself, not its sub fields. Purpose is to emulate an
        onchange: if event type is changed, update event configuration. Changing
        event type content itself should not trigger this method.

        When synchronizing booths:

          * lines that are available are removed;
          * template lines are added;
        """
        for event in self:
            if not event.event_type_id and not event.event_booth_ids:
                event.event_booth_ids = False
                continue

            # booths to keep: those that are not available
            booths_to_remove = event.event_booth_ids.filtered(lambda booth: booth.is_available)
            command = [Command.unlink(booth.id) for booth in booths_to_remove]
            if event.event_type_id.event_type_booth_ids:
                command += [
                    Command.create({
                        attribute_name: line[attribute_name] if not isinstance(line[attribute_name], models.BaseModel) else line[attribute_name].id
                        for attribute_name in self.env['event.type.booth']._get_event_booth_fields_whitelist()
                    }) for line in event.event_type_id.event_type_booth_ids
                ]
            event.event_booth_ids = command

    def _get_booth_stat_count(self):
        elements = self.env['event.booth'].sudo()._read_group(
            [('event_id', 'in', self.ids)],
            ['event_id', 'state'], ['event_id', 'state'], lazy=False
        )
        elements_total_count = dict()
        elements_available_count = dict()
        for element in elements:
            event_id = element['event_id'][0]
            if element['state'] == 'available':
                elements_available_count[event_id] = element['__count']
            elements_total_count.setdefault(event_id, 0)
            elements_total_count[event_id] += element['__count']
        return elements_available_count, elements_total_count

    @api.depends('event_booth_ids', 'event_booth_ids.state')
    def _compute_event_booth_count(self):
        if self.ids and all(bool(event.id) for event in self):  # no new/onchange mode -> optimized
            booths_available_count, booths_total_count = self._get_booth_stat_count()
            for event in self:
                event.event_booth_count_available = booths_available_count.get(event.id, 0)
                event.event_booth_count = booths_total_count.get(event.id, 0)
        else:
            for event in self:
                event.event_booth_count = len(event.event_booth_ids)
                event.event_booth_count_available = len(event.event_booth_ids.filtered(lambda booth: booth.is_available))

    @api.depends('event_booth_ids.booth_category_id')
    def _compute_event_booth_category_ids(self):
        for event in self:
            event.event_booth_category_ids = event.event_booth_ids.mapped('booth_category_id')

    @api.depends('event_booth_ids.is_available')
    def _compute_event_booth_category_available_ids(self):
        for event in self:
            event.event_booth_category_available_ids = event.event_booth_ids.filtered(lambda booth: booth.is_available).mapped('booth_category_id')

```

## File: models\event_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class EventType(models.Model):
    _inherit = 'event.type'

    event_type_booth_ids = fields.One2many(
        'event.type.booth', 'event_type_id',
        string='Booths', readonly=False, store=True)

```

## File: models\event_type_booth.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class EventBooth(models.Model):
    _name = 'event.type.booth'
    _description = 'Event Booth Template'

    def _get_default_booth_category(self):
        """Assign booth category by default if only one exists"""
        category_id = self.env['event.booth.category'].search([])
        if category_id and len(category_id) == 1:
            return category_id

    name = fields.Char(string='Name', required=True, translate=True)
    event_type_id = fields.Many2one(
        'event.type', string='Event Category',
        ondelete='cascade', required=True)
    booth_category_id = fields.Many2one(
        'event.booth.category', string='Booth Category',
        default=_get_default_booth_category, ondelete='restrict', required=True)

    @api.model
    def _get_event_booth_fields_whitelist(self):
        return ['name', 'booth_category_id']

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_type_booth
from . import event_booth
from . import event_booth_category
from . import event_event
from . import event_type

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_event_booth_category,event.booth.category,model_event_booth_category,,0,0,0,0
access_event_booth_category_desk,event.booth.category.desk,model_event_booth_category,event.group_event_registration_desk,1,0,0,0
access_event_booth_category_manager,event.booth.category.manager,model_event_booth_category,event.group_event_manager,1,1,1,1
access_event_booth_all,event.booth.public,model_event_booth,,0,0,0,0
access_event_booth_user,event.booth.user,model_event_booth,event.group_event_registration_desk,1,0,0,0
access_event_booth_manager,event.booth.manager,model_event_booth,event.group_event_manager,1,1,1,1
access_event_type_booth_user,event.type.booth.user,model_event_type_booth,event.group_event_registration_desk,1,0,0,0
access_event_type_booth_manager,event.type.booth.manager,model_event_type_booth,event.group_event_manager,1,1,1,1

```

## File: views\event_booth_category_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_booth_category_view_form" model="ir.ui.view">
        <field name="name">event.booth.category.view.form</field>
        <field name="model">event.booth.category</field>
        <field name="arch" type="xml">
            <form string="Booth Type">
                <sheet>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <field name="image_1920" widget='image' class="oe_avatar" options='{"preview_image": "image_128"}'/>
                    <div class="oe_title">
                        <label for="name" string="Booth Category"/>
                        <h1><field name="name" placeholder="e.g. Premium Booth"/></h1>
                    </div>
                    <group name="main">
                        <field name="active" invisible="1"/>
                    </group>
                    <notebook>
                        <page string="Description" name="description">
                            <field name="description" placeholder='e.g. "Those stands will be place near the entrance and..."'/>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="event_booth_category_view_tree" model="ir.ui.view">
        <field name="name">event.booth.category.view.tree</field>
        <field name="model">event.booth.category</field>
        <field name="arch" type="xml">
            <tree>
                <field name="sequence" widget="handle"/>
                <field name="name"/>
            </tree>
        </field>
    </record>

    <record id="event_booth_category_view_search" model="ir.ui.view">
        <field name="name">event.booth.category.view.search</field>
        <field name="model">event.booth.category</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <filter string="Archived" name="archived" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="event_booth_category_action" model="ir.actions.act_window">
        <field name="name">Booth Category</field>
        <field name="res_model">event.booth.category</field>
        <field name="view_mode">tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Booth Category
            </p>
            <p>
                Booth categories are used to represent the different types of booths you rent (Premium Booth, Table and Chairs, ...)
            </p>
        </field>
    </record>

</data></odoo>

```

## File: views\event_booth_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_booth_view_form_from_event" model="ir.ui.view">
        <field name="name">event.booth.view.form.from.event</field>
        <field name="model">event.booth</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <form string="Booths">
                <header>
                    <field name="state" widget="statusbar" options="{'clickable': '1'}"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box" attrs="{'invisible': [('id', '=', False)]}"/>
                    <div class="oe_title">
                        <label for="name" class="oe_edit_only" string="Name"/>
                        <h1><field name="name" placeholder="e.g. First Booth Alley 1"/></h1>
                    </div>
                    <group>
                        <group name="details">
                            <field name="booth_category_id" placeholder="Pick a Booth Category..."/>
                        </group>
                        <group name="renter">
                            <field name="partner_id"/>
                            <field name="contact_name"/>
                            <field name="contact_email" widget="email"/>
                            <field name="contact_phone" widget="phone"/>
                            <field name="contact_mobile" widget="phone"/>
                        </group>
                    </group>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids" groups="base.group_user"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <record id="event_booth_view_form" model="ir.ui.view">
        <field name="name">event.booth.view.form</field>
        <field name="model">event.booth</field>
        <field name="inherit_id" ref="event_booth_view_form_from_event"/>
        <field name="mode">primary</field>
        <field name="priority">16</field>
        <field name="arch" type="xml">
            <field name="booth_category_id" position="before">
                <field name="event_id"/>
            </field>
        </field>
    </record>

    <record id="event_booth_view_form_simple_from_event" model="ir.ui.view">
        <field name="name">event.booth.view.form.simple.from.event</field>
        <field name="model">event.booth</field>
        <field name="inherit_id" ref="event_booth_view_form_from_event"/>
        <field name="mode">primary</field>
        <field name="priority">48</field>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="replace"/>
            <xpath expr="//div[hasclass('oe_chatter')]" position="replace"/>
        </field>
    </record>

    <record id="event_booth_view_tree_from_event" model="ir.ui.view">
        <field name="name">event.booth.view.tree.from.event</field>
        <field name="model">event.booth</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <tree string="Booths" sample="1" expand="1">
                <field name="name" decoration-bf="1"/>
                <field name="booth_category_id"/>
                <field name="partner_id"/>
                <field name="contact_name" optional="hide"/>
                <field name="contact_email" optional="hide"/>
                <field name="contact_phone" optional="hide"/>
                <field name="contact_mobile" optional="hide"/>
                <field name="state" widget="badge"
                       decoration-info="state == 'available'"
                       decoration-success="state == 'unavailable'"/>
            </tree>
        </field>
    </record>

    <record id="event_booth_view_tree" model="ir.ui.view">
        <field name="name">event.booth.view.tree</field>
        <field name="model">event.booth</field>
        <field name="inherit_id" ref="event_booth_view_tree_from_event"/>
        <field name="mode">primary</field>
        <field name="priority">16</field>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="event_id"/>
            </field>
        </field>
    </record>

    <record id="event_booth_view_kanban_from_event" model="ir.ui.view">
        <field name="name">event.booth.view.kanban</field>
        <field name="model">event.booth</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <kanban default_group_by="state" quick_create_view="event_booth.event_booth_view_form_quick_create" sample="1">
                <field name="name"/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="o_kanban_content oe_kanban_global_click">
                            <div class="o_kanban_record_title">
                                <field name="name"/>
                            </div>
                            <div class="d-flex flex-column">
                                <div class="o_kanban_record_bottom">
                                    <div class="oe_kanban_bottom_left">
                                        <field name="booth_category_id"/>
                                    </div>
                                    <div class="oe_kanban_bottom_right">
                                        <field name="activity_ids" widget="kanban_activity"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="event_booth_view_kanban" model="ir.ui.view">
        <field name="name">event.booth.view.kanban</field>
        <field name="model">event.booth</field>
        <field name="inherit_id" ref="event_booth_view_kanban_from_event"/>
        <field name="mode">primary</field>
        <field name="priority">16</field>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('o_kanban_record_bottom')]" position="before">
                <field name="event_id"/>
            </xpath>
        </field>
    </record>

    <record id="event_booth_view_form_quick_create" model="ir.ui.view">
        <field name="name">event.booth.view.form.quick_create</field>
        <field name="model">event.booth</field>
        <field name="arch" type="xml">
            <form>
                <group>
                    <field name="name" placeholder="e.g. First Booth Alley 1"/>
                    <field name="booth_category_id" placeholder="Pick a Booth Category..."/>
                </group>
            </form>
        </field>
    </record>

    <record id="event_booth_view_search" model="ir.ui.view">
        <field name="name">event.booth.view.search</field>
        <field name="model">event.booth</field>
        <field name="arch" type="xml">
            <search string="Event Booth">
                <field name="name" string="Name" filter_domain="[('name', 'ilike', self)]"/>
                <field name="contact_name" string="Renter Name" filter_domain="[('contact_name', 'ilike', self)]"/>
                <field name="contact_email" string="Renter Email" filter_domain="[('contact_email', 'ilike', self)]"/>
                <field name="event_id"/>
                <filter string="Available" name="filter_booth_available"
                    domain="[('state', '=', 'available')]"/>
                <filter string="Unavailable" name="filter_booth_unavailable"
                    domain="[('state', '=', 'unavailable')]"/>
                <group expand="0" string="Group By">
                    <filter name="group_by_state" context="{'group_by': 'state'}"/>
                    <filter name="group_by_partner_id" context="{'group_by': 'partner_id'}"/>
                    <filter name="group_by_booth_category_id" context="{'group_by': 'booth_category_id'}"/>
                    <filter string="Event" name="group_by_event_id" context="{'group_by': 'event_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="event_booth_view_graph" model="ir.ui.view">
        <field name="name">event.booth.view.graph</field>
        <field name="model">event.booth</field>
        <field name="arch" type="xml">
            <graph string="Event booth" sample="1" type="pie">
                <field name="booth_category_id"/>
            </graph>
        </field>
    </record>

    <record id="event_booth_view_pivot" model="ir.ui.view">
        <field name="name">event.booth.view.pivot</field>
        <field name="model">event.booth</field>
        <field name="arch" type="xml">
            <pivot string="Event booth" sample="1">
                <field name="booth_category_id" type="row"/>
            </pivot>
        </field>
    </record>

    <record id="event_booth_action" model="ir.actions.act_window">
        <field name="name">Booths</field>
        <field name="res_model">event.booth</field>
        <field name="view_mode">kanban,tree,form,graph,pivot</field>
        <field name="domain">[]</field>
        <field name="context">{'search_default_group_by_state': 1}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Booth
            </p><p>
                Booths are the physical stands that you rent during your event.
            </p>
        </field>
    </record>

    <record id="event_booth_action_from_event" model="ir.actions.act_window">
        <field name="name">Booths</field>
        <field name="res_model">event.booth</field>
        <field name="view_mode">kanban,tree,form,graph,pivot</field>
        <field name="domain">[('event_id', '=', active_id)]</field>
        <field name="context">{'default_event_id': active_id, 'search_default_group_by_state': 1}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Booth
            </p><p>
                Booths are the physical stands that you rent during your event.
            </p>
        </field>
    </record>
    <record id="event_booth_action_from_event_view_kanban" model="ir.actions.act_window.view">
        <field name="sequence">1</field>
        <field name="view_mode">kanban</field>
        <field name="view_id" ref="event_booth_view_kanban_from_event"/>
        <field name="act_window_id" ref="event_booth_action_from_event"/>
    </record>
    <record id="event_booth_action_from_event_view_tree" model="ir.actions.act_window.view">
        <field name="sequence">2</field>
        <field name="view_mode">tree</field>
        <field name="view_id" ref="event_booth_view_tree_from_event"/>
        <field name="act_window_id" ref="event_booth_action_from_event"/>
    </record>
    <record id="event_booth_action_from_event_view_form" model="ir.actions.act_window.view">
        <field name="sequence">3</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="event_booth_view_form_from_event"/>
        <field name="act_window_id" ref="event_booth_action_from_event"/>
    </record>

</data></odoo>

```

## File: views\event_event_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_event_view_form" model="ir.ui.view">
        <field name="name">event.event.view.form.inherit.event.booth</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="event.view_event_form"/>
        <field name="priority" eval="4"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button class="oe_stat_button" name="%(event_booth_action_from_event)d"
                        type="action" icon="fa-university">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <span attrs="{'invisible': [('event_booth_count', '=', 0)]}">
                                <field name="event_booth_count_available"/> /
                            </span>
                            <field name="event_booth_count"/>
                        </span>
                        <span class="o_stat_text">Booths</span>
                    </div>
                </button>
            </div>
        </field>
    </record>

</data></odoo>

```

## File: views\event_menus.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <menuitem name="Booth Categories"
        id="menu_event_booth_category"
        action="event_booth_category_action"
        sequence="20"
        parent="event.menu_event_configuration"/>
    <menuitem name="Booths"
        id="menu_event_booth"
        action="event_booth_action"
        sequence="21"
        parent="event.menu_event_configuration"
        groups="base.group_no_one"/>

</data></odoo>

```

## File: views\event_type_booth_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_type_booth_view_form_from_type" model="ir.ui.view">
        <field name="name">event.type.booth.view.form.from.type</field>
        <field name="model">event.type.booth</field>
        <field name="arch" type="xml">
            <form string="Event Type Booth">
                <sheet>
                    <div class="oe_title">
                        <label for="name" class="oe_edit_only" string="Name"/>
                        <h1><field name="name" placeholder="e.g. First Booth Alley 1"/></h1>
                    </div>
                    <group>
                        <field name="booth_category_id" placeholder="Pick a Booth Category..."/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="event_type_booth_view_form" model="ir.ui.view">
        <field name="name">event.type.booth.view.form</field>
        <field name="model">event.type.booth</field>
        <field name="inherit_id" ref="event_type_booth_view_form_from_type"/>
        <field name="mode">primary</field>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='booth_category_id']" position="before">
                <field name="event_type_id"/>
            </xpath>
        </field>
    </record>

    <record id="event_type_booth_view_tree_from_type" model="ir.ui.view">
        <field name="name">event.type.booth.view.tree.from.type</field>
        <field name="model">event.type.booth</field>
        <field name="arch" type="xml">
            <tree string="Event Type Booths">
                <field name="name"/>
                <field name="booth_category_id"/>
            </tree>
        </field>
    </record>

    <record id="event_type_booth_view_tree" model="ir.ui.view">
        <field name="name">event.type.booth.view.tree</field>
        <field name="model">event.type.booth</field>
        <field name="inherit_id" ref="event_type_booth_view_tree_from_type"/>
        <field name="mode">primary</field>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='booth_category_id']" position="before">
                <field name="event_type_id"/>
            </xpath>
        </field>
    </record>

    <record id="event_type_booth_view_search" model="ir.ui.view">
        <field name="name">event.type.booth.view.search</field>
        <field name="model">event.type.booth</field>
        <field name="arch" type="xml">
            <search string="Event Type Booths">
                <field name="name"/>
                <group expand="0" string="Group By">
                    <filter string="Booth Type" name="booth_category" context="{'group_by': 'booth_category_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="event_type_booth_action" model="ir.actions.act_window">
        <field name="name">Event Type Booths</field>
        <field name="res_model">event.type.booth</field>
        <field name="view_mode">tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Type Booth
            </p><p>
                Booths are the physical stands that you rent during your event.
            </p>
        </field>
    </record>

</data></odoo>

```

## File: views\event_type_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_type_view_form" model="ir.ui.view">
        <field name="name">event.type.view.form.inherit.event.booth</field>
        <field name="model">event.type</field>
        <field name="inherit_id" ref="event.view_event_type_form"/>
        <field name="arch" type="xml">
            <page name="event_type_communication" position="after">
                <page string="Booths">
                    <field name="event_type_booth_ids"
                           class="w-100"
                           context="{
                               'tree_view_ref': 'event_booth.event_type_booth_view_tree_from_type',
                               'form_view_ref': 'event_booth.event_type_booth_view_form_from_type'
                           }"/>
                </page>
            </page>
        </field>
    </record>

</data></odoo>

```

