# Odoo Module: mass_mailing_event_track

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
    'name': 'Mass mailing on track speakers',
    'category': 'Hidden',
    'version': '1.0',
    'description':
        """
Mass mail event track speakers
==============================

Bridge module adding UX requirements to ease mass mailing of event track speakers.
        """,
    'depends': ['website_event_track', 'mass_mailing'],
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

from odoo import models


class Event(models.Model):
    _inherit = "event.event"

    def action_mass_mailing_track_speakers(self):
        mass_mailing_action = dict(
            name='Mass Mail Attendees',
            type='ir.actions.act_window',
            res_model='mailing.mailing',
            view_mode='form',
            target='current',
            context=dict(
                default_mailing_model_id=self.env.ref('website_event_track.model_event_track').id,
                default_mailing_domain="[('event_id', 'in', %s), ('stage_id.is_cancel', '!=', True)]" % self.ids,
            ),
        )
        return mass_mailing_action

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
    <record id="event_event_view_form_inherit_mass_mailing_track" model="ir.ui.view">
        <field name="name">event.event.view.form.inherit.mass.mailing.track</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="website_event_track.view_event_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='button_cancel']" position="after">
                <button name="action_mass_mailing_track_speakers" string="Contact Track Speakers" type="object" attrs="{'invisible': [('track_count', '=', 0)]}" groups="mass_mailing.group_mass_mailing_user"/>
            </xpath>
        </field>
    </record>
</odoo>

```

