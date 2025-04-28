# Odoo Module: event_sms

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
    'name': 'SMS on Events',
    'version': '1.0',
    'category': 'Marketing/Events',
    'description': """Schedule SMS in event management""",
    'depends': ['event', 'sms'],
    'data': [
        'data/sms_data.xml',
        'views/event_views.xml',
        'views/event_mail_views.xml',
        'security/ir.model.access.csv',
        'security/sms_security.xml',
    ],
    'demo': [
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\sms_data.xml

```xml
<?xml version="1.0"?>
<odoo><data noupdate="1">

    <record id="sms_template_data_event_registration" model="sms.template">
        <field name="name">Event: Registration</field>
        <field name="model_id" ref="event.model_event_registration"/>
        <field name="body">${object.event_id.organizer_id.name or object.event_id.company_id.name or user.env.company.name}: We are happy to confirm your registration for the ${object.event_id.name} event.</field>
        <field name="lang">${object.partner_id.lang}</field>
    </record>

    <record id="sms_template_data_event_reminder" model="sms.template">
        <field name="name">Event: Reminder</field>
        <field name="model_id" ref="event.model_event_registration"/>
        <field name="body">${object.event_id.organizer_id.name or object.event_id.company_id.name or user.env.company.name}: We are excited to remind you that the ${object.event_id.name} event is starting ${object.get_date_range_str()}. We confirm your registration and hope to meet you there.</field>
        <field name="lang">${object.partner_id.lang}</field>
    </record>

</data></odoo>

```

## File: models\event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Registration(models.Model):
    _inherit = 'event.registration'

    def _sms_get_number_fields(self):
        """ This method returns the fields to use to find the number to use to
        send an SMS on a record. """
        return ['mobile', 'phone']

```

## File: models\event_mail.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class EventTypeMail(models.Model):
    _inherit = 'event.type.mail'

    notification_type = fields.Selection(selection_add=[('sms', 'SMS')])
    sms_template_id = fields.Many2one(
        'sms.template', string='SMS Template',
        domain=[('model', '=', 'event.registration')], ondelete='restrict',
        help='This field contains the template of the SMS that will be automatically sent')

    @api.model
    def _get_event_mail_fields_whitelist(self):
        return super(EventTypeMail, self)._get_event_mail_fields_whitelist() + ['sms_template_id']


class EventMailScheduler(models.Model):
    _inherit = 'event.mail'

    notification_type = fields.Selection(selection_add=[('sms', 'SMS')])
    sms_template_id = fields.Many2one(
        'sms.template', string='SMS Template',
        domain=[('model', '=', 'event.registration')], ondelete='restrict',
        help='This field contains the template of the SMS that will be automatically sent')

    def execute(self):
        for mail in self:
            now = fields.Datetime.now()
            if mail.interval_type != 'after_sub':
                # Do not send SMS if the communication was scheduled before the event but the event is over
                if not mail.mail_sent and (mail.interval_type != 'before_event' or mail.event_id.date_end > now) and mail.notification_type == 'sms' and mail.sms_template_id:
                    self.env['event.registration']._message_sms_schedule_mass(
                        template=mail.sms_template_id,
                        active_domain=[('event_id', '=', mail.event_id.id), ('state', '!=', 'cancel')],
                        mass_keep_log=True
                    )
                    mail.write({'mail_sent': True})
        return super(EventMailScheduler, self).execute()


class EventMailRegistration(models.Model):
    _inherit = 'event.mail.registration'

    def execute(self):
        for record in self:
            if record.registration_id.state in ['open', 'done'] and not record.mail_sent and record.scheduler_id.notification_type == 'sms':
                record.registration_id._message_sms_schedule_mass(template=record.scheduler_id.sms_template_id, mass_keep_log=True)
                record.write({'mail_sent': True})
        return super(EventMailRegistration, self).execute()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event
from . import event_mail

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_sms_template_event_manager,access.sms.template.event.manager,sms.model_sms_template,event.group_event_manager,1,1,1,1

```

## File: security\sms_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_rule_sms_template_event_manager" model="ir.rule">
        <field name="name">SMS Template: event manager CUD on event / registrations templates</field>
        <field name="model_id" ref="sms.model_sms_template"/>
        <field name="groups" eval="[(4, ref('event.group_event_manager'))]"/>
        <field name="domain_force">[('model_id.model', 'in', ('event.event', 'event.registration'))]</field>
        <field name="perm_read" eval="False"/>
    </record>
</odoo>

```

## File: views\event_mail_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>
    <record id="event_mail_view_form" model="ir.ui.view">
        <field name="name">event.mail.view.form.inherit.sms</field>
        <field name="model">event.mail</field>
        <field name="inherit_id" ref="event.view_event_mail_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='notification_type']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//field[@name='template_id']" position="attributes">
                <attribute name="attrs">{'readonly': [('notification_type', '!=', 'mail')], 'required': [('notification_type', '=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='template_id']" position="after">
                <field name="sms_template_id"
                    attrs="{'readonly': [('notification_type', '!=', 'sms')], 'required': [('notification_type', '=', 'sms')]}"
                    context="{'default_model': 'event.registration'}"/>
            </xpath>
        </field>
    </record>

    <record id="event_mail_view_tree" model="ir.ui.view">
        <field name="name">event.mail.view.tree.inherit.sms</field>
        <field name="model">event.mail</field>
        <field name="inherit_id" ref="event.view_event_mail_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='notification_type']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//field[@name='template_id']" position="attributes">
                <attribute name="attrs">{'readonly': [('notification_type', '!=', 'mail')], 'required': [('notification_type', '=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='template_id']" position="after">
                <field name="sms_template_id"
                    attrs="{'readonly': [('notification_type', '!=', 'sms')], 'required': [('notification_type', '=', 'sms')]}"
                    context="{'default_model': 'event.registration'}"/>
            </xpath>
        </field>
    </record>

</data></odoo>

```

## File: views\event_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>

    <record id="event_type_view_form" model="ir.ui.view">
        <field name="name">event.type.view.form.inherit.sms</field>
        <field name="model">event.type</field>
        <field name="inherit_id" ref="event.view_event_type_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='notification_type']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//field[@name='template_id']" position="attributes">
                <attribute name="attrs">{'readonly': [('notification_type', '!=', 'mail')], 'required': [('notification_type', '=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='template_id']" position="after">
                <field name="sms_template_id"
                    attrs="{'readonly': [('notification_type', '!=', 'sms')], 'required': [('notification_type', '=', 'sms')]}"
                    context="{'default_model': 'event.registration'}"/>
            </xpath>
        </field>
    </record>

    <record id="event_event_view_form_inherit_sms" model="ir.ui.view">
        <field name="name">event.event.view.form</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="event.view_event_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='notification_type']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//field[@name='template_id']" position="attributes">
                <attribute name="attrs">{'readonly': [('notification_type', '!=', 'mail')], 'required': [('notification_type', '=', 'mail')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='template_id']" position="after">
                <field name="sms_template_id"
                    attrs="{'readonly': [('notification_type', '!=', 'sms')], 'required': [('notification_type', '=', 'sms')]}"
                    context="{'default_model': 'event.registration'}"/>
            </xpath>
        </field>
    </record>

</data></odoo>

```

