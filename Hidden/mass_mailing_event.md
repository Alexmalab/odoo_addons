# Odoo Module: mass_mailing_event

Category: Hidden

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
    'name': 'Mass mailing on attendees',
    'category': 'Hidden',
    'version': '1.0',
    'description':
        """
Mass mail event attendees
=========================

Bridge module adding UX requirements to ease mass mailing of event attendees.
        """,
    'depends': ['event', 'mass_mailing'],
    'data': [
        'views/event_views.xml'
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _


class Event(models.Model):
    _inherit = "event.event"

    def action_mass_mailing_attendees(self):
        return {
            'name': 'Mass Mail Attendees',
            'type': 'ir.actions.act_window',
            'res_model': 'mailing.mailing',
            'view_mode': 'form',
            'target': 'current',
            'context': {
                'default_mailing_model_id': self.env.ref('event.model_event_registration').id,
                'default_mailing_domain': repr([('event_id', 'in', self.ids), ('state', '!=', 'cancel')])
            },
        }

    def action_invite_contacts(self):
        return {
            'name': 'Mass Mail Invitation',
            'type': 'ir.actions.act_window',
            'res_model': 'mailing.mailing',
            'view_mode': 'form',
            'target': 'current',
            'context': {'default_mailing_model_id': self.env.ref('base.model_res_partner').id},
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event

```

## File: views\event_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="event_event_view_form_inherit_mass_mailing" model="ir.ui.view">
        <field name="name">event.event.view.form.inherit.mass.mailing</field>
        <field name="model">event.event</field>
        <field name="priority" eval="4"/>
        <field name="inherit_id" ref="event.view_event_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='stage_id']" position="before">
                <field name="event_registrations_open" invisible="1"/>
                <button name="action_invite_contacts" type="object" string="Invite"
                    class="btn btn-primary"
                    groups="mass_mailing.group_mass_mailing_user"
                    attrs="{'invisible': [('event_registrations_open', '=', False)]}"/>
                <button name="action_invite_contacts" type="object" string="Invite"
                    class="btn btn-secondary"
                    groups="mass_mailing.group_mass_mailing_user"
                    attrs="{'invisible': [('event_registrations_open', '=', True)]}"/>
                <button name="action_mass_mailing_attendees" type="object" string="Contact Attendees"
                    groups="mass_mailing.group_mass_mailing_user"
                    attrs="{'invisible': [('seats_expected', '=', 0)]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

