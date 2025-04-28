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
    'summary': 'Send text messages as event reminders',
    'description': "Send text messages as event reminders",
    'category': 'Hidden',
    'version': '1.0',
    'depends': ['calendar', 'sms'],
    'data': [
        'security/sms_security.xml',
        'data/sms_data.xml',
        'views/calendar_views.xml',
    ],
    'application': False,
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
            <field name="body">Event reminder: ${object.name}, ${object.get_display_time_tz(object.partner_id.tz)}</field>
        </record>
    </data>
</odoo>

```

## File: models\calendar.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, fields, models, _

_logger = logging.getLogger(__name__)


class CalendarEvent(models.Model):
    _inherit = 'calendar.event'

    def _sms_get_default_partners(self):
        """ Method overridden from mail.thread (defined in the sms module).
            SMS text messages will be sent to attendees that haven't declined the event(s).
        """
        return self.mapped('attendee_ids').filtered(lambda att: att.state != 'declined' and att.partner_id.phone_sanitized).mapped('partner_id')

    def _do_sms_reminder(self):
        """ Send an SMS text reminder to attendees that haven't declined the event """
        for event in self:
            event._message_sms_with_template(
                template_xmlid='calendar_sms.sms_template_data_calendar_reminder',
                template_fallback=_("Event reminder: %(name)s, %(time)s.", name=event.name, time=event.display_time),
                partner_ids=self._sms_get_default_partners().ids,
                put_in_queue=False
            )


class CalendarAlarm(models.Model):
    _inherit = 'calendar.alarm'

    alarm_type = fields.Selection(selection_add=[
        ('sms', 'SMS Text Message')
    ], ondelete={'sms': 'set default'})


class AlarmManager(models.AbstractModel):
    _inherit = 'calendar.alarm_manager'

    @api.model
    def get_next_mail(self):
        """ Cron method, overridden here to send SMS reminders as well
        """
        result = super(AlarmManager, self).get_next_mail()

        cron = self.env.ref('calendar.ir_cron_scheduler_alarm', raise_if_not_found=False)
        if not cron:
            # Like the super method, do nothing if cron doesn't exist anymore
            return result

        now = fields.Datetime.to_string(fields.Datetime.now())
        last_sms_cron = cron.lastcall

        interval_to_second = {
            "weeks": 7 * 24 * 60 * 60,
            "days": 24 * 60 * 60,
            "hours": 60 * 60,
            "minutes": 60,
            "seconds": 1
        }

        cron_interval = cron.interval_number * interval_to_second[cron.interval_type]
        events_data = self._get_next_potential_limit_alarm('sms', seconds=cron_interval)

        for event in self.env['calendar.event'].browse(events_data):
            max_delta = events_data[event.id]['max_duration']
            event_start = fields.Datetime.from_string(event.start)
            for alert in self.do_check_alarm_for_one_date(event_start, event, max_delta, 0, 'sms', after=last_sms_cron, missing=True):
                event.browse(alert['event_id'])._do_sms_reminder()
        return result

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import calendar

```

## File: security\sms_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_rule_sms_template_system" model="ir.rule">
        <field name="name">SMS Template: unnecessary rule for system group</field>
        <field name="model_id" ref="sms.model_sms_template"/>
        <field name="groups" eval="[(4, ref('base.group_system'))]"/>
        <!-- TDE NOTE: remove me in master, broken fix from odoo/odoo#64626 -->
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="perm_read" eval="False"/>
    </record>
</odoo>

```

## File: views\calendar_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

</odoo>

```

