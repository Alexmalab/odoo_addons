# Odoo Module: mass_mailing_slides

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
    'name': 'Mass mailing on course members',
    'category': 'Hidden',
    'version': '1.0',
    'description':
        """
Mass mail course members
========================

Bridge module adding UX requirements to ease mass mailing of course members.
        """,
    'depends': ['website_slides', 'mass_mailing'],
    'data': [
        'views/slide_channel_views.xml'
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\slide_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _


class Course(models.Model):
    _inherit = "slide.channel"

    def action_mass_mailing_attendees(self):
        domain = repr([('slide_channel_ids', 'in', self.ids)])
        mass_mailing_action = dict(
            name=_('Mass Mail Course Members'),
            type='ir.actions.act_window',
            res_model='mailing.mailing',
            view_mode='form',
            target='current',
            context=dict(
                default_mailing_model_id=self.env.ref('base.model_res_partner').id,
                default_mailing_domain=domain,
            ),
        )
        return mass_mailing_action

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import slide_channel

```

## File: views\slide_channel_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="slide_channel_view_form" model="ir.ui.view">
        <field name="name">slide.channel.view.form.inherit.mass.mailing</field>
        <field name="model">slide.channel</field>
        <field name="inherit_id" ref="website_slides.view_slide_channel_form"/>
        <field name="arch" type="xml">
            <button name="action_channel_enroll" position="before">
                <field name="members_count" invisible="1"/>
                <button name="action_mass_mailing_attendees" string="Contact Attendees" type="object"
                        class="oe_highlight" invisible="members_count == 0"
                        groups="mass_mailing.group_mass_mailing_user"/>
            </button>
        </field>
    </record>

    <record id="slide_channel_view_kanban" model="ir.ui.view">
        <field name="name">slide.channel.view.kanban.inherit.mass.mailing</field>
        <field name="model">slide.channel</field>
        <field name="inherit_id" ref="website_slides.slide_channel_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//a[@name='action_channel_invite']" position="after">
                <a role="menuitem" name="action_mass_mailing_attendees" type="object"
                   groups="mass_mailing.group_mass_mailing_user" class="dropdown-item">
                    Contact Attendees
                </a>
            </xpath>
        </field>
    </record>
</odoo>

```

