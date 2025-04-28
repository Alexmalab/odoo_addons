# Odoo Module: calendar_sms

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
    'name': "Calendar - SMS",
    'version': "1.1",
    'summary': 'Send text messages as event reminders',
    'description': "Send text messages as event reminders",
    'category': 'Hidden',
    'depends': ['calendar', 'sms'],
    'data': [
        'data/sms_data.xml',
        'views/calendar_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\sms_data.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <data noupdate="1">
        <record id="sms_template_data_calendar_reminder" model="sms.template">
            <field name="name">Calendar Event: Reminder</field>
            <field name="model_id" ref="calendar.model_calendar_event"/>
            <field name="body">Event reminder: {{ object.name }}, {{ object.get_display_time_tz(object.partner_id.tz) }}</field>
        </record>
    </data>
</odoo>

```

## File: models\calendar_alarm.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models


class CalendarAlarm(models.Model):
    _inherit = 'calendar.alarm'

    alarm_type = fields.Selection(selection_add=[
        ('sms', 'SMS Text Message')
    ], ondelete={'sms': 'set default'})
    sms_template_id = fields.Many2one(
        'sms.template', string="SMS Template",
        domain=[('model', 'in', ['calendar.event'])],
        compute='_compute_sms_template_id', readonly=False, store=True,
        help="Template used to render SMS reminder content.")
    sms_notify_responsible = fields.Boolean("Notify Responsible")

    @api.depends('alarm_type', 'sms_template_id')
    def _compute_sms_template_id(self):
        for alarm in self:
            if alarm.alarm_type == 'sms' and not alarm.sms_template_id:
                alarm.sms_template_id = self.env['ir.model.data']._xmlid_to_res_id('calendar_sms.sms_template_data_calendar_reminder')
            elif alarm.alarm_type != 'sms' or not alarm.sms_template_id:
                alarm.sms_template_id = False

    @api.onchange('duration', 'interval', 'alarm_type', 'sms_notify_responsible')
    def _onchange_duration_interval(self):
        super()._onchange_duration_interval()
        if self.alarm_type != 'sms':
            self.sms_notify_responsible = False
        elif self.sms_notify_responsible:
            self.name += " - " + _("Notify Responsible")

```

## File: models\calendar_alarm_manager.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class AlarmManager(models.AbstractModel):
    _inherit = 'calendar.alarm_manager'

    @api.model
    def _send_reminder(self):
        """ Cron method, overridden here to send SMS reminders as well
        """
        super()._send_reminder()
        events_by_alarm = self._get_events_by_alarm_to_notify('sms')
        if not events_by_alarm:
            return

        event_ids = list(set(event_id for event_ids in events_by_alarm.values() for event_id in event_ids))
        events = self.env['calendar.event'].browse(event_ids)
        alarms = self.env['calendar.alarm'].browse(events_by_alarm.keys())
        for event in events:
            alarm = event.alarm_ids.filtered(lambda alarm: alarm.id in alarms.ids)
            event._do_sms_reminder(alarm)
            if event.recurrence_id:
                next_date = event.get_next_alarm_date(events_by_alarm)
                event.recurrence_id.with_context(date=next_date)._setup_alarms()

```

## File: models\calendar_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from odoo.exceptions import UserError

class CalendarEvent(models.Model):
    _inherit = 'calendar.event'

    def _do_sms_reminder(self, alarms):
        """ Send an SMS text reminder to attendees that haven't declined the event """
        for event in self:
            declined_partners = event.attendee_ids.filtered_domain([('state', '=', 'declined')]).partner_id
            for alarm in alarms:
                partners = event._mail_get_partners()[event.id].filtered(
                    lambda partner: partner.phone_sanitized and partner not in declined_partners
                )
                if event.user_id and not alarm.sms_notify_responsible:
                    partners -= event.user_id.partner_id
                event._message_sms_with_template(
                    template=alarm.sms_template_id,
                    template_fallback=_("Event reminder: %(name)s, %(time)s.", name=event.name, time=event.display_time),
                    partner_ids=partners.ids,
                    put_in_queue=False
                )

    def action_send_sms(self):
        if not self.partner_ids:
            raise UserError(_("There are no attendees on these events"))
        return {
            'type': 'ir.actions.act_window',
            'name': _("Send SMS Text Message"),
            'res_model': 'sms.composer',
            'view_mode': 'form',
            'target': 'new',
            'context': {
                'default_composition_mode': 'mass',
                'default_res_model': 'res.partner',
                'default_res_ids': self.partner_ids.ids,
                'default_mass_keep_log': True,
            },
        }

    def _get_trigger_alarm_types(self):
        return super()._get_trigger_alarm_types() + ['sms']

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import calendar_alarm
from . import calendar_alarm_manager
from . import calendar_event

```

## File: views\calendar_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="calendar_alarm_view_form" model="ir.ui.view">
        <field name="name">calendar.alarm.view.form.inherit.calendar.sms</field>
        <field name="model">calendar.alarm</field>
        <field name="inherit_id" ref="calendar.calendar_alarm_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='mail_template_id']" position="after">
                <field name="sms_template_id" invisible="alarm_type != 'sms'" required="alarm_type == 'sms'"
                    context="{'default_model': 'calendar.event'}"/>
            </xpath>
            <xpath expr="//group[@name='right_details']" position="inside">
                <field name="sms_notify_responsible" invisible="alarm_type != 'sms'"/>
            </xpath>
        </field>
    </record>

    <record id="view_calendar_event_tree_inherited" model="ir.ui.view">
        <field name="name">calendar.event.tree.calendar_sms</field>
        <field name="model">calendar.event</field>
        <field name="inherit_id" ref="calendar.view_calendar_event_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <button name="action_send_sms" type="object"
                        string="Send SMS"/>
            </xpath>
        </field>
    </record>

    <record id="view_calendar_event_form_inherited" model="ir.ui.view">
        <field name="name">calendar.event.form.calendar_sms</field>
        <field name="model">calendar.event</field>
        <field name="inherit_id" ref="calendar.view_calendar_event_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='send_buttons']" position="inside">
                <button name="action_send_sms" help="Send SMS to attendees" type="object" string="SMS" icon="fa-mobile" invisible="not user_can_edit"/>
            </xpath>
            <xpath expr="//field[@name='phone']" position="attributes">
                <attribute name="options">{'enable_sms': false}</attribute>
            </xpath>
        </field>
    </record>

</odoo>

```

