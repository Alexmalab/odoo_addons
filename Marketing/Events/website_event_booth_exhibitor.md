# Odoo Module: website_event_booth_exhibitor

Category: Marketing/Events

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    'name': 'Booths/Exhibitors Bridge',
    'category': 'Marketing/Events',
    'version': '1.1',
    'summary': 'Event Booths, automatically create a sponsor.',
    'description': """
Automatically create a sponsor when renting a booth.
    """,
    'depends': ['website_event_exhibitor', 'website_event_booth'],
    'data': [
        'data/event_booth_category_data.xml',

        'views/event_booth_category_views.xml',
        'views/event_booth_views.xml',

        'views/event_booth_registration_templates.xml',
        'views/event_booth_templates.xml',
        'views/mail_templates.xml'
    ],
    'auto_install': True,
    'assets': {
        'web.assets_frontend': [
            '/website_event_booth_exhibitor/static/src/js/booth_sponsor_details.js',
        ],
        'web.assets_tests': [
            'website_event_booth_exhibitor/static/tests/tours/website_event_booth_exhibitor.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\event_booth.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64

from odoo.addons.website_event.controllers.main import WebsiteEventController
from odoo.tools import plaintext2html


class WebsiteEventBoothController(WebsiteEventController):

    def _prepare_booth_registration_values(self, event, kwargs):
        booth_values = super(WebsiteEventBoothController, self)._prepare_booth_registration_values(event, kwargs)
        if not booth_values.get('contact_email'):
            booth_values['contact_email'] = kwargs.get('sponsor_email')
        if not booth_values.get('contact_name'):
            booth_values['contact_name'] = kwargs.get('sponsor_name')
        if not booth_values.get('contact_mobile'):
            booth_values['contact_mobile'] = kwargs.get('sponsor_mobile')
        if not booth_values.get('contact_phone'):
            booth_values['contact_phone'] = kwargs.get('sponsor_phone')

        booth_values.update(**self._prepare_booth_registration_sponsor_values(event, booth_values, kwargs))
        return booth_values

    def _prepare_booth_registration_partner_values(self, event, kwargs):
        if not kwargs.get('contact_email') and kwargs.get('sponsor_email'):
            kwargs['contact_email'] = kwargs['sponsor_email']
        if not kwargs.get('contact_name') and kwargs.get('sponsor_name'):
            kwargs['contact_name'] = kwargs['sponsor_name']
        if not kwargs.get('contact_phone') and kwargs.get('sponsor_phone'):
            kwargs['contact_phone'] = kwargs['sponsor_phone']
        return super(WebsiteEventBoothController, self)._prepare_booth_registration_partner_values(event, kwargs)

    def _prepare_booth_registration_sponsor_values(self, event, booth_values, kwargs):
        sponsor_values = {
            'sponsor_name': kwargs.get('sponsor_name') or booth_values.get('contact_name'),
            'sponsor_email': kwargs.get('sponsor_email') or booth_values.get('contact_email'),
            'sponsor_mobile': kwargs.get('sponsor_mobile') or booth_values.get('contact_mobile'),
            'sponsor_phone': kwargs.get('sponsor_phone') or booth_values.get('contact_phone'),
            'sponsor_subtitle': kwargs.get('sponsor_slogan'),
            'sponsor_website_description': plaintext2html(kwargs.get('sponsor_description')) if kwargs.get('sponsor_description') else '',
            'sponsor_image_512': base64.b64encode(kwargs['sponsor_image'].read()) if kwargs.get('sponsor_image') else False,
        }
        return sponsor_values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_booth

```

## File: data\event_booth_category_data.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_booth.event_booth_category_premium" model="event.booth.category">
        <field name="use_sponsor" eval="True"/>
        <field name="sponsor_type_id" ref="website_event_exhibitor.event_sponsor_type2"/>
        <field name="exhibitor_type">exhibitor</field>
    </record>

    <record id="event_booth.event_booth_category_vip" model="event.booth.category">
        <field name="use_sponsor" eval="True"/>
        <field name="sponsor_type_id" ref="website_event_exhibitor.event_sponsor_type3"/>
        <field name="exhibitor_type">online</field>
    </record>

</data></odoo>

```

## File: models\event_booth.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class EventBooth(models.Model):
    _inherit = 'event.booth'

    use_sponsor = fields.Boolean(related='booth_category_id.use_sponsor')
    sponsor_type_id = fields.Many2one(related='booth_category_id.sponsor_type_id')
    sponsor_id = fields.Many2one('event.sponsor', string='Sponsor', copy=False)
    sponsor_name = fields.Char(string='Sponsor Name', related='sponsor_id.name')
    sponsor_email = fields.Char(string='Sponsor Email', related='sponsor_id.email')
    sponsor_mobile = fields.Char(string='Sponsor Mobile', related='sponsor_id.mobile')
    sponsor_phone = fields.Char(string='Sponsor Phone', related='sponsor_id.phone')
    sponsor_subtitle = fields.Char(string='Sponsor Slogan', related='sponsor_id.subtitle')
    sponsor_website_description = fields.Html(string='Sponsor Description', related='sponsor_id.website_description')
    sponsor_image_512 = fields.Image(string='Sponsor Logo', related='sponsor_id.image_512')

    def action_view_sponsor(self):
        action = self.env['ir.actions.act_window']._for_xml_id('website_event_exhibitor.event_sponsor_action')
        action['views'] = [(False, 'form')]
        action['res_id'] = self.sponsor_id.id
        return action

    def _get_or_create_sponsor(self, vals):
        self.ensure_one()
        sponsor_id = self.env['event.sponsor'].sudo().search([
            ('partner_id', '=', self.partner_id.id),
            ('sponsor_type_id', '=', self.sponsor_type_id.id),
            ('exhibitor_type', '=', self.booth_category_id.exhibitor_type),
            ('event_id', '=', self.event_id.id),
        ], limit=1)
        if not sponsor_id:
            values = {
                'event_id': self.event_id.id,
                'sponsor_type_id': self.sponsor_type_id.id,
                'exhibitor_type': self.booth_category_id.exhibitor_type,
                'partner_id': self.partner_id.id,
                **{key.partition('sponsor_')[2]: value for key, value in vals.items() if key.startswith('sponsor_')},
            }
            # If confirmed from backend, we don't have _prepare_booth_registration_values
            if not values.get('name'):
                values['name'] = self.partner_id.name
            if self.booth_category_id.exhibitor_type == 'online':
                values.update({
                    'room_name': 'odoo-exhibitor-%s' % self.partner_id.name,
                })
            sponsor_id = self.env['event.sponsor'].sudo().create(values)
        return sponsor_id.id

    def _action_post_confirm(self, write_vals):
        for booth in self:
            if booth.use_sponsor and booth.partner_id:
                booth.sponsor_id = booth._get_or_create_sponsor(write_vals)
        super(EventBooth, self)._action_post_confirm(write_vals)

```

## File: models\event_booth_category.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class EventBoothCategory(models.Model):
    _inherit = 'event.booth.category'

    @api.model
    def _get_exhibitor_type(self):
        return self.env['event.sponsor']._fields['exhibitor_type'].selection

    use_sponsor = fields.Boolean(string='Create Sponsor', help="If set, when booking a booth a sponsor will be created for the user")
    sponsor_type_id = fields.Many2one('event.sponsor.type', string='Sponsor Level')
    exhibitor_type = fields.Selection(_get_exhibitor_type, string='Sponsor Type')

    @api.onchange('use_sponsor')
    def _onchange_use_sponsor(self):
        if self.use_sponsor:
            if not self.sponsor_type_id:
                self.sponsor_type_id = self.env['event.sponsor.type'].search([], order="sequence desc", limit=1).id
            if not self.exhibitor_type:
                self.exhibitor_type = self._get_exhibitor_type()[0][0]

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_booth
from . import event_booth_category

```

## File: static\src\js\booth_sponsor_details.js

```javascript
odoo.define('website_event_booth_exhibitor.booth_sponsor_details', function (require) {
'use strict';

var publicWidget = require('web.public.widget');

publicWidget.registry.boothSponsorDetails = publicWidget.Widget.extend({
    selector: '#o_wbooth_contact_details_form',
    events: {
        'click input[id="contact_details"]': '_onClickContactDetails',
    },

    //--------------------------------------------------------------------------
    // Handler
    //--------------------------------------------------------------------------

    _onClickContactDetails(ev) {
        this.useContactDetails = ev.currentTarget.checked;
        this.$('#o_wbooth_contact_details').toggleClass('d-none', !this.useContactDetails);
        this.$('label[for="sponsor_name"] > .mandatory_mark, label[for="sponsor_email"] > .mandatory_mark').toggleClass('d-none', this.useContactDetails);
        this.$('input[name="contact_name"], input[name="contact_email"]').attr('required', this.useContactDetails);
    },

});

    return {
        boothSponsorDetails: publicWidget.registry.boothSponsorDetails,
    };

});

```

## File: views\event_booth_category_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_booth_category_view_form" model="ir.ui.view">
        <field name="name">event.booth.category.view.form.inherit.website.event.booth.exhibitor</field>
        <field name="model">event.booth.category</field>
        <field name="inherit_id" ref="event_booth.event_booth_category_view_form"/>
        <field name="priority" eval="2"/>
        <field name="arch" type="xml">
            <group name="main" position="inside">
                <group string="Sponsorship" name="sponsor">
                    <field name="use_sponsor"/>
                    <field name="sponsor_type_id" attrs="{'invisible': [('use_sponsor', '=', False)], 'required': [('use_sponsor', '=', True)]}"/>
                    <field name="exhibitor_type" attrs="{'invisible': [('use_sponsor', '=', False)], 'required': [('use_sponsor', '=', True)]}"/>
                </group>
            </group>
        </field>
    </record>

    <record id="event_booth_category_view_tree" model="ir.ui.view">
        <field name="name">event.booth.category.view.tree.inherit.website.event.booth.exhibitor</field>
        <field name="model">event.booth.category</field>
        <field name="inherit_id" ref="event_booth.event_booth_category_view_tree"/>
        <field name="priority">5</field>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="use_sponsor"/>
                <field name="sponsor_type_id" optional="hide"/>
                <field name="exhibitor_type" optional="hide"/>
            </field>
        </field>
    </record>

    <record id="event_booth_category_view_search" model="ir.ui.view">
        <field name="name">event.booth.category.view.search.inherit.website.event.booth.exhibitor</field>
        <field name="model">event.booth.category</field>
        <field name="inherit_id" ref="event_booth.event_booth_category_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='name']" position="after">
                <field name="use_sponsor"/>
                <field name="sponsor_type_id"/>
                <field name="exhibitor_type"/>
                <group expand="0" string="Group By">
                    <filter name="group_by_sponsor_type" string="Sponsor type" context="{'group_by': 'sponsor_type_id'}"/>
                    <filter name="group_by_exhibitor_type" string="Exhibitor type" context="{'group_by':'exhibitor_type'}"/>
                </group>
            </xpath>
        </field>
    </record>

</data></odoo>

```

## File: views\event_booth_registration_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <template id="event_booth_registration_details" inherit_id="website_event_booth.event_booth_registration_details">
        <form id="o_wbooth_contact_details_form" position="attributes">
            <attribute name="enctype">multipart/form-data</attribute>
        </form>
        <xpath expr="//div[@id='o_wbooth_contact_details']" position="before">
            <div id="o_wbooth_sponsor_details" t-if="booth_category.use_sponsor">
                <h4 class="mt32">
                    <strong>Sponsor Details</strong>
                </h4>
                <div class="form-text text-muted mb16">
                    This booth type allows you to have visibility on the event website. Please fill in this form
                </div>
                <div class="row mb-3">
                    <label class="col-form-label col-sm-auto" for="sponsor_name">
                        <span>Name</span>
                        <span class="mandatory_mark"> *</span>
                    </label>
                    <div class="col-sm">
                        <input class="form-control" type="text" name="sponsor_name" id="sponsor_name" required="True"/>
                    </div>
                </div>
                <div class="row mb-3">
                    <label class="col-form-label col-sm-auto" for="sponsor_email">
                        <span>Email</span>
                        <span class="mandatory_mark"> *</span>
                    </label>
                    <div class="col-sm">
                        <input class="form-control" type="email" name="sponsor_email" id="sponsor_email" required="True"/>
                    </div>
                </div>
                <div class="row mb-3">
                    <label class="col-form-label col-sm-auto" for="sponsor_phone">Phone</label>
                    <div class="col-sm">
                        <input class="form-control" type="text" name="sponsor_phone" id="sponsor_phone"/>
                    </div>
                </div>
                <div class="row mb-3">
                    <label class="col-form-label col-sm-auto" for="sponsor_slogan">Slogan</label>
                    <div class="col-sm">
                        <input class="form-control" type="text" name="sponsor_slogan" id="sponsor_slogan"/>
                    </div>
                </div>
                <div class="row mb-3" t-if="booth_category.exhibitor_type != 'sponsor'">
                    <label class="col-form-label col-sm-auto" for="sponsor_description">Description</label>
                    <div class="col-sm">
                        <textarea class="form-control" name="sponsor_description" id="sponsor_description"/>
                    </div>
                </div>
                <div class="row mb-3">
                    <label class="col-form-label col-sm-auto" for="sponsor_image">Picture</label>
                    <div class="col-sm">
                        <input name="sponsor_image" type="file" accept="image/*"/>
                    </div>
                </div>
                <div class="form-check">
                    <input class="form-check-input" type="checkbox" id="contact_details"/>
                    <label class="fw-normal" for="contact_details">Contact me through a different email/phone.</label>
                </div>
            </div>
        </xpath>
        <xpath expr="//div[@id='o_wbooth_contact_details']" position="attributes">
            <attribute name="t-att-class">'d-none' if booth_category.use_sponsor else ''</attribute>
        </xpath>
        <xpath expr="//input[@name='contact_name']" position="attributes">
            <attribute name="t-att-required">False if booth_category.use_sponsor else True</attribute>
        </xpath>
        <xpath expr="//input[@name='contact_email']" position="attributes">
            <attribute name="t-att-required">False if booth_category.use_sponsor else True</attribute>
        </xpath>
    </template>

</data></odoo>

```

## File: views\event_booth_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <template id="event_booth_registration" inherit_id="website_event_booth.event_booth_registration">
        <xpath expr="//input[@name='booth_category_id']" position="attributes">
            <attribute name="t-att-data-use-sponsor">'true' if booth_category.use_sponsor else 'false'</attribute>
        </xpath>
    </template>

</data></odoo>

```

## File: views\event_booth_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_booth_view_form_from_event" model="ir.ui.view">
        <field name="name">event.booth.view.form.inherit.website.event.booth.exhibitor</field>
        <field name="model">event.booth</field>
        <field name="inherit_id" ref="event_booth.event_booth_view_form_from_event"/>
        <field name="priority">5</field>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <field name="sponsor_id" invisible="1"/>
                <button name="action_view_sponsor" type="object" class="oe_stat_button"
                        icon="fa-black-tie" string="Sponsor" attrs="{'invisible': [('sponsor_id', '=', False)]}">
                </button>
            </div>
        </field>
    </record>

</data></odoo>

```

## File: views\mail_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <template id="event_booth_booked_template" inherit_id="event_booth.event_booth_booked_template">
        <ul name="contact_details" position="inside">
            <t t-set="renter_sponsor" t-value="booth.sponsor_id"/>
            <li t-if="renter_sponsor">
                <b>Sponsor</b>: <a href="#" t-att-data-oe-model="booth.sponsor_id._name" t-att-data-oe-id="booth.sponsor_id.id" t-out="booth.sponsor_id.name"/>
            </li>
        </ul>
    </template>

</data></odoo>

```

