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

from odoo import models


class Event(models.Model):
    _inherit = "event.event"

    def action_mass_mailing_attendees(self):
        if len(self) == 1:
            domain = "[('event_id', '=', {})]".format(self.id)
        else:
            domain = "[('event_id', 'in', {})]".format(self.ids)
        mass_mailing_action = dict(
            name='Mass Mail Attendees',
            type='ir.actions.act_window',
            res_model='mailing.mailing',
            view_mode='form',
            target='current',
            context=dict(
                default_mailing_model_id=self.env.ref('event.model_event_registration').id,
                default_mailing_domain=domain,
            ),
        )
        return mass_mailing_action

```

## File: models\mailing_mailing.py

```python
from odoo import models, api


class MassMailingCampaign(models.Model):
    _inherit = "mailing.mailing"

    @api.onchange('mailing_model_real', 'contact_list_ids')
    def _onchange_model_and_list(self):
        # TDE FIXME: whuuut ?
        result = super(MassMailingCampaign, self)._onchange_model_and_list()
        if self.mailing_model_name == 'event.registration' and self.mailing_domain == '[]':
            self.mailing_domain = self.env.context.get('default_mailing_domain', '[]')
        return result

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event
from . import mailing_mailing

```

## File: views\event_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="event_event_view_form_inherit_mass_mailing" model="ir.ui.view">
        <field name="name">event.event.view.form.inherit.mass.mailing</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="event.view_event_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='button_cancel']" position="after">
                <button name="action_mass_mailing_attendees" string="Contact Attendees" type="object" attrs="{'invisible': [('seats_expected', '=', 0)]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

