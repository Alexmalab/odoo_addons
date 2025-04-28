# Odoo Module: calendar

Category: Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Calendar',
    'version': '1.0',
    'sequence': 130,
    'depends': ['base', 'mail'],
    'summary': "Schedule employees' meetings",
    'description': """
This is a full-featured calendar system.
========================================

It supports:
------------
    - Calendar of events
    - Recurring events

If you need to manage your meetings, you should install the CRM module.
    """,
    'category': 'Tools',
    'demo': [
        'data/calendar_demo.xml'
    ],
    'data': [
        'security/ir.model.access.csv',
        'security/calendar_security.xml',
        'data/calendar_cron.xml',
        'data/calendar_data.xml',
        'data/mail_data.xml',
        'views/mail_activity_views.xml',
        'views/calendar_templates.xml',
        'views/calendar_views.xml',
        'data/mail_activity_data.xml',
    ],
    'qweb': ['static/src/xml/base_calendar.xml'],
    'installable': True,
    'application': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: controllers\bus.py

```python
# -*- coding: utf-8 -*

from odoo.addons.bus.controllers.main import BusController
from odoo.http import request


class CalendarBusController(BusController):
    # --------------------------
    # Extends BUS Controller Poll
    # --------------------------
    def _poll(self, dbname, channels, last, options):
        if request.session.uid:
            channels = list(channels)
            channels.append((request.db, 'calendar.alarm', request.env.user.partner_id.id))
        return super(CalendarBusController, self)._poll(dbname, channels, last, options)

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import werkzeug

from odoo.api import Environment
import odoo.http as http

from odoo.http import request
from odoo import SUPERUSER_ID
from odoo import registry as registry_get
from odoo.tools.misc import get_lang


class CalendarController(http.Controller):

    @http.route('/calendar/meeting/accept', type='http', auth="calendar")
    def accept(self, db, token, action, id, **kwargs):
        registry = registry_get(db)
        with registry.cursor() as cr:
            env = Environment(cr, SUPERUSER_ID, {})
            attendee = env['calendar.attendee'].search([('access_token', '=', token), ('state', '!=', 'accepted')])
            if attendee:
                attendee.do_accept()
        return self.view(db, token, action, id, view='form')

    @http.route('/calendar/meeting/decline', type='http', auth="calendar")
    def declined(self, db, token, action, id):
        registry = registry_get(db)
        with registry.cursor() as cr:
            env = Environment(cr, SUPERUSER_ID, {})
            attendee = env['calendar.attendee'].search([('access_token', '=', token), ('state', '!=', 'declined')])
            if attendee:
                attendee.do_decline()
        return self.view(db, token, action, id, view='form')

    @http.route('/calendar/meeting/view', type='http', auth="calendar")
    def view(self, db, token, action, id, view='calendar'):
        registry = registry_get(db)
        with registry.cursor() as cr:
            # Since we are in auth=none, create an env with SUPERUSER_ID
            env = Environment(cr, SUPERUSER_ID, {})
            attendee = env['calendar.attendee'].search([('access_token', '=', token), ('event_id', '=', int(id))])
            if not attendee:
                return request.not_found()
            timezone = attendee.partner_id.tz
            lang = attendee.partner_id.lang or get_lang(request.env).code
            event = env['calendar.event'].with_context(tz=timezone, lang=lang).browse(int(id))

            # If user is internal and logged, redirect to form view of event
            # otherwise, display the simplifyed web page with event informations
            if request.session.uid and request.env['res.users'].browse(request.session.uid).user_has_groups('base.group_user'):
                return werkzeug.utils.redirect('/web?db=%s#id=%s&view_type=form&model=calendar.event' % (db, id))

            # NOTE : we don't use request.render() since:
            # - we need a template rendering which is not lazy, to render before cursor closing
            # - we need to display the template in the language of the user (not possible with
            #   request.render())
            response_content = env['ir.ui.view'].with_context(lang=lang).render_template(
                'calendar.invitation_page_anonymous', {
                    'event': event,
                    'attendee': attendee,
                })
            return request.make_response(response_content, headers=[('Content-Type', 'text/html')])

    # Function used, in RPC to check every 5 minutes, if notification to do for an event or not
    @http.route('/calendar/notify', type='json', auth="user")
    def notify(self):
        return request.env['calendar.alarm_manager'].get_next_notif()

    @http.route('/calendar/notify_ack', type='json', auth="user")
    def notify_ack(self, type=''):
        return request.env['res.partner'].sudo()._set_calendar_last_notif_ack()

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main
from . import bus

```

## File: data\calendar_cron.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">
        <!-- Scheduler for Event Alarm-->
        <record forcecreate="True" id="ir_cron_scheduler_alarm" model="ir.cron">
            <field name="name">Calendar: Event Reminder</field>
            <field name="model_id" ref="model_calendar_alarm_manager"/>
            <field name="state">code</field>
            <field name="code">model.get_next_mail()</field>
            <field eval="False" name="active" />
            <field name="user_id" ref="base.user_root" />
            <field name="interval_number">30</field>
            <field name="interval_type">minutes</field>
            <field name="numbercall">-1</field>
            <field eval="False" name="doall" />
        </record>
    </data>
</odoo>
```

## File: data\calendar_data.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">
        <record id="calendar.subtype_invitation" model="mail.message.subtype">
            <field name="name">Invitation</field>
            <field name="res_model">calendar.event</field>
            <field name="description" eval="False"/>
            <field name="default" eval="False"/>
        </record>
        <record id="alarm_notif_1" model="calendar.alarm">
            <field name="name">Notification - 15 Minutes</field>
            <field name="duration" eval="15" />
            <field name="interval">minutes</field>
            <field name="alarm_type">notification</field>
        </record>
        <record id="alarm_notif_2" model="calendar.alarm">
            <field name="name">Notification - 30 Minutes</field>
            <field name="duration" eval="30" />
            <field name="interval">minutes</field>
            <field name="alarm_type">notification</field>
        </record>
        <record id="alarm_notif_3" model="calendar.alarm">
            <field name="name">Notification - 1 Hours</field>
            <field name="duration" eval="1" />
            <field name="interval">hours</field>
            <field name="alarm_type">notification</field>
        </record>
        <record id="alarm_notif_4" model="calendar.alarm">
            <field name="name">Notification - 2 Hours</field>
            <field name="duration" eval="2" />
            <field name="interval">hours</field>
            <field name="alarm_type">notification</field>
        </record>
        <record id="alarm_notif_5" model="calendar.alarm">
            <field name="name">Notification - 1 Days</field>
            <field name="duration" eval="1" />
            <field name="interval">days</field>
            <field name="alarm_type">notification</field>
        </record>
        <record id="alarm_mail_1" model="calendar.alarm">
            <field name="name">Email - 3 Hours</field>
            <field name="duration" eval="3" />
            <field name="interval">hours</field>
            <field name="alarm_type">email</field>
        </record>
        <record id="alarm_mail_2" model="calendar.alarm">
            <field name="name">Email - 6 Hours</field>
            <field name="duration" eval="6" />
            <field name="interval">hours</field>
            <field name="alarm_type">email</field>
        </record>
    </data>
</odoo>
```

## File: data\calendar_demo.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">

        <record id="cal_contact_1" model="calendar.contacts">
            <field name="active" eval="True"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="partner_id" ref="base.res_partner_1"/>
        </record>

        <record id="cal_contact_2" model="calendar.contacts">
            <field name="active" eval="True"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="partner_id" ref="base.partner_demo"/>
        </record>

        <record id="categ_meet1" model="calendar.event.type">
            <field name="name">Customer Meeting</field>
        </record>

        <record id="categ_meet2" model="calendar.event.type">
            <field name="name">Internal Meeting</field>
        </record>

        <record id="categ_meet3" model="calendar.event.type">
            <field name="name">Off-site Meeting</field>
        </record>

        <record id="categ_meet4" model="calendar.event.type">
            <field name="name">Open Discussion</field>
        </record>

        <record id="categ_meet5" model="calendar.event.type">
            <field name="name">Feedback Meeting</field>
        </record>

        <record id="calendar_event_1" model="calendar.event">
            <field name="active" eval="True"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="partner_ids" eval="[(6,0,[ref('base.res_partner_1')])]"/>
            <field name="name">Follow-up for Project proposal</field>
            <field name="description">Meeting to discuss project plan and hash out the details of implementation.</field>
            <field name="start" eval="time.strftime('%Y-%m-03 10:20:00')"/>
            <field name="categ_ids" eval="[(6,0,[ref('categ_meet1')])]"/>
            <field name="stop" eval="time.strftime('%Y-%m-03 16:30:00')"/>
            <field name="duration" eval="6.3"/>
            <field name="allday" eval="False"/>
            <field name="state">open</field>
        </record>

        <record id="calendar_event_2" model="calendar.event">
            <field name="active" eval="True"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="partner_ids" eval="[(6,0,[ref('base.partner_root'),ref('base.res_partner_4'),ref('base.res_partner_3')])]"/>
            <field name="name">Initial discussion</field>
            <field name="description">Discussion with partner for product.</field>
            <field name="categ_ids" eval="[(6,0,[ref('categ_meet3')])]"/>
            <field name="start" eval="time.strftime('%Y-%m-05 12:00:00')"/>
            <field name="stop" eval="time.strftime('%Y-%m-05 19:00:00')"/>
            <field name="allday" eval="False"/>
            <field name="duration" eval="7.0"/>
            <field name="state">draft</field>
        </record>

        <record id="calendar_event_3" model="calendar.event">
            <field name="active" eval="True"/>
            <field name="partner_ids" eval="[(6,0,[ref('base.partner_admin')])]"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="name">Pricing Discussion</field>
            <field name="description">Internal meeting for discussion for new pricing for product and services.</field>
            <field name="categ_ids" eval="[(6,0,[ref('categ_meet1'), ref('categ_meet2')])]"/>
            <field name="start" eval="time.strftime('%Y-%m-12 15:55:05')"/>
            <field name="stop" eval="time.strftime('%Y-%m-12 18:55:05')"/>
            <field name="duration" eval="3.0"/>
            <field name="allday" eval="False"/>
            <field name="state">open</field>
        </record>

        <record id="calendar_event_4" model="calendar.event">
            <field name="active" eval="True"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="partner_ids" eval="[(6,0,[ref('base.partner_demo'),ref('base.res_partner_1')])]"/>
            <field name="name">Requirements review</field>
            <field name="categ_ids" eval="[(6,0,[ref('categ_meet3')])]"/>
            <field name="start" eval="time.strftime('%Y-%m-20 08:00:00')"/>
            <field name="stop" eval="time.strftime('%Y-%m-20 10:30:00')"/>
            <field name="duration" eval="2.5"/>
            <field name="allday" eval="False"/>
            <field name="state">open</field>
        </record>

        <record id="calendar_event_5" model="calendar.event">
            <field name="active" eval="True"/>
            <field name="partner_ids" eval="[(6,0,[ref('base.partner_admin'),ref('base.res_partner_12')])]"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="name">Changes in Designing</field>
            <field name="categ_ids" eval="[(6,0,[ref('categ_meet1')])]"/>
            <field name="start" eval="time.strftime('%Y-%m-22')"/>
            <field name="stop" eval="time.strftime('%Y-%m-22')"/>
            <field name="allday" eval="True"/>
            <field name="state">open</field>
        </record>

        <record id="calendar_event_6" model="calendar.event">
            <field name="active" eval="True"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="partner_ids" eval="[(6,0,[ref('base.partner_root'),ref('base.res_partner_4'),ref('base.res_partner_1'),ref('base.res_partner_12')])]"/>
            <field name="name">Presentation for new Services</field>
            <field name="categ_ids" eval="[(6,0,[ref('categ_meet1'), ref('categ_meet2')])]"/>
            <field name="start" eval="time.strftime('%Y-%m-18 02:00:00')"/>
            <field name="stop" eval="time.strftime('%Y-%m-18 10:30:00')"/>
            <field name="duration" eval="8.5"/>
            <field name="allday" eval="False"/>
            <field name="state">draft</field>
        </record>

        <record id="calendar_event_7" model="calendar.event">
            <field name="active" eval="True"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="partner_ids" eval="[(6,0,[ref('base.res_partner_4')])]"/>
            <field name="name">Presentation of the new Calendar</field>
            <field name="categ_ids" eval="[(6,0,[ref('categ_meet1'), ref('categ_meet2')])]"/>
            <field name="start" eval="time.strftime('%Y-%m-16')"/>
            <field name="stop" eval="time.strftime('%Y-%m-16')"/>
            <field name="duration" eval="8.5"/>
            <field name="allday" eval="True"/>
            <field name="state">draft</field>
        </record>


    </data>
</odoo>

```

## File: data\mail_activity_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mail.mail_activity_data_meeting" model="mail.activity.type">
            <field name="category">meeting</field>
        </record>
    </data>
</odoo>

```

## File: data\mail_data.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">
        <record id="calendar_template_meeting_invitation" model="mail.template">
            <field name="name">Calendar: Meeting Invitation</field>
            <field name="model_id" ref="calendar.model_calendar_attendee"/>
            <field name="subject">Invitation to ${object.event_id.name}</field>
            <field name="email_from">${(object.event_id.user_id.email_formatted or user.email_formatted or '') | safe}</field>
            <field name="email_to">${('' if object.partner_id.email and object.partner_id.email == object.email else object.email) | safe}</field>
            <field name="partner_to">${object.partner_id.id if object.partner_id.email and object.partner_id.email == object.email else False}</field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="body_html" type="html">
<div>
    % set colors = {'needsAction': 'grey', 'accepted': 'green', 'tentative': '#FFFF00',  'declined': 'red'}
    <p>
        Hello ${object.common_name},<br/><br/>
        ${object.event_id.user_id.partner_id.name} invited you for the ${object.event_id.name} meeting of ${object.event_id.user_id.company_id.name}.
    </p>
    <div style="text-align: center; margin: 16px 0px 16px 0px;">
        <a href="/calendar/meeting/accept?db=${'dbname' in ctx and ctx['dbname'] or ''}&amp;token=${object.access_token}&amp;action=${'action_id' in ctx and ctx['action_id'] or ''}&amp;id=${object.event_id.id}" 
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            Accept</a>
        <a href="/calendar/meeting/decline?db=${'dbname' in ctx and ctx['dbname'] or '' }&amp;token=${object.access_token}&amp;action=${'action_id' in ctx and ctx['action_id'] or ''}&amp;id=${object.event_id.id}" 
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            Decline</a>
        <a href="/calendar/meeting/view?db=${'dbname' in ctx and ctx['dbname'] or ''}&amp;token=${object.access_token}&amp;action=${'action_id' in ctx and ctx['action_id'] or ''}&amp;id=${object.event_id.id}" 
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            View</a>
    </div>
    <table border="0" cellpadding="0" cellspacing="0"><tr>
        <td width="130px;">
            <div style="border-top-left-radius: 3px; border-top-right-radius: 3px; font-size: 12px; border-collapse: separate; text-align: center; font-weight: bold; color: #ffffff; min-height: 18px; background-color: #875A7B; border: 1px solid #875A7B;">
                ${object.event_id.get_interval('dayname', tz=object.partner_id.tz if not object.event_id.allday else None)}
            </div>
            <div style="font-size: 48px; min-height: auto; font-weight: bold; text-align: center; color: #5F5F5F; background-color: #F8F8F8; border: 1px solid #875A7B;">
                ${object.event_id.get_interval('day', tz=object.partner_id.tz if not object.event_id.allday else None)}
            </div>
            <div style='font-size: 12px; text-align: center; font-weight: bold; color: #ffffff; background-color: #875A7B;'>
                ${object.event_id.get_interval('month', tz=object.partner_id.tz if not object.event_id.allday else None)}
            </div>
            <div style="border-collapse: separate; color: #5F5F5F; text-align: center; font-size: 12px; border-bottom-right-radius: 3px; font-weight: bold; border: 1px solid #875A7B; border-bottom-left-radius: 3px;">
                ${not object.event_id.allday and object.event_id.get_interval('time', tz=object.partner_id.tz) or ''}
            </div>
        </td>
        <td width="20px;"/>
        <td style="padding-top: 5px;">
            <p><strong>Details of the event</strong></p>
            <ul>
                % if object.event_id.location:
                    <li>Location: ${object.event_id.location}
                        (<a target="_blank" href="http://maps.google.com/maps?oi=map&amp;q=${object.event_id.location}">View Map</a>)
                    </li>
                % endif
                % if object.event_id.description :
                    <li>Description: ${object.event_id.description}</li>
                % endif
                % if not object.event_id.allday and object.event_id.duration
                    <li>Duration: ${('%dH%02d' % (object.event_id.duration,round(object.event_id.duration*60)%60))}</li>
                % endif
                <li>Attendees
                <ul>
                % for attendee in object.event_id.attendee_ids:
                    <li>
                        <div style="display: inline-block; border-radius: 50%; width: 10px; height: 10px; background:${colors[attendee.state] or 'white'};"> </div>
                        % if attendee.common_name != object.common_name:
                            <span style="margin-left:5px">${attendee.common_name}</span>
                        % else:
                            <span style="margin-left:5px">You</span>
                        % endif
                    </li>
                % endfor
                </ul></li>
            </ul>
        </td>
    </tr></table>
    <br/>
    Thank you,
    <br/>
    % if object.event_id.user_id and object.event_id.user_id.signature:
        ${object.event_id.user_id.signature | safe}
    % endif
</div>
            </field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="auto_delete" eval="True"/>
            <field name="user_signature" eval="False"/>
        </record>

        <record id="calendar_template_meeting_changedate" model="mail.template">
            <field name="name">Calendar: Date updated</field>
            <field name="model_id" ref="calendar.model_calendar_attendee"/>
            <field name="subject">${object.event_id.name}: Date updated</field>
            <field name="email_from">${(object.event_id.user_id.email_formatted or user.email_formatted or '') | safe}</field>
            <field name="email_to">${('' if object.partner_id.email and object.partner_id.email == object.email else object.email) | safe}</field>
            <field name="partner_to">${object.partner_id.id if object.partner_id.email and object.partner_id.email == object.email else False}</field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="body_html" type="html">
<div>
    % set colors = {'needsAction': 'grey', 'accepted': 'green', 'tentative': '#FFFF00',  'declined': 'red'}
    <p>
        Hello ${object.common_name},<br/><br/>
        The date of the meeting has been updated. The meeting ${object.event_id.name} created by ${object.event_id.user_id.partner_id.name} is now scheduled for ${object.event_id.get_display_time_tz(tz=object.partner_id.tz)}.
    </p>
    <div style="text-align: center; margin: 16px 0px 16px 0px;">
        <a href="/calendar/meeting/accept?db=${'dbname' in ctx and ctx['dbname'] or ''}&amp;token=${object.access_token}&amp;action=${'action_id' in ctx and ctx['action_id'] or ''}&amp;id=${object.event_id.id}" 
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            Accept</a>
        <a href="/calendar/meeting/decline?db=${'dbname' in ctx and ctx['dbname'] or '' }&amp;token=${object.access_token}&amp;action=${'action_id' in ctx and ctx['action_id'] or ''}&amp;id=${object.event_id.id}"
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            Decline</a>
        <a href="/calendar/meeting/view?db=${'dbname' in ctx and ctx['dbname'] or ''}&amp;token=${object.access_token}&amp;action=${'action_id' in ctx and ctx['action_id'] or ''}&amp;id=${object.event_id.id}"
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            View</a>
    </div>
    <table border="0" cellpadding="0" cellspacing="0"><tr>
        <td width="130px;">
            <div style="border-top-left-radius: 3px; border-top-right-radius: 3px; font-size: 12px; border-collapse: separate; text-align: center; font-weight: bold; color: #ffffff; min-height: 18px; background-color: #875A7B; border: 1px solid #875A7B;">
                ${object.event_id.get_interval('dayname', tz=object.partner_id.tz if not object.event_id.allday else None)}
            </div>
            <div style="font-size: 48px; min-height: auto; font-weight: bold; text-align: center; color: #5F5F5F; background-color: #F8F8F8; border: 1px solid #875A7B;">
                ${object.event_id.get_interval('day', tz=object.partner_id.tz if not object.event_id.allday else None)}
            </div>
            <div style='font-size: 12px; text-align: center; font-weight: bold; color: #ffffff; background-color: #875A7B;'>
                ${object.event_id.get_interval('month', tz=object.partner_id.tz if not object.event_id.allday else None)}
            </div>
            <div style="border-collapse: separate; color: #5F5F5F; text-align: center; font-size: 12px; border-bottom-right-radius: 3px; font-weight: bold; border: 1px solid #875A7B; border-bottom-left-radius: 3px;">
                ${not object.event_id.allday and object.event_id.get_interval('time', tz=object.partner_id.tz) or ''}
            </div>
        </td>
        <td width="20px;"/>
        <td style="padding-top: 5px;">
            <p><strong>Details of the event</strong></p>
            <ul>
                % if object.event_id.location:
                    <li>Location: ${object.event_id.location}
                        (<a target="_blank" href="http://maps.google.com/maps?oi=map&amp;q=${object.event_id.location}">View Map</a>)
                    </li>
                % endif
                % if object.event_id.description :
                    <li>Description: ${object.event_id.description}</li>
                % endif
                % if not object.event_id.allday and object.event_id.duration
                    <li>Duration: ${('%dH%02d' % (object.event_id.duration,round(object.event_id.duration*60)%60))}</li>
                % endif
                <li>Attendees
                <ul>
                % for attendee in object.event_id.attendee_ids:
                    <li>
                        <div style="display: inline-block; border-radius: 50%; width: 10px; height: 10px; background: ${colors[attendee.state] or 'white'};"> </div>
                        % if attendee.common_name != object.common_name:
                            <span style="margin-left:5px">${attendee.common_name}</span>
                        % else:
                            <span style="margin-left:5px">You</span>
                        % endif
                    </li>
                % endfor
                </ul></li>
            </ul>
        </td>
    </tr></table>
    <br/>
    Thank you,
    <br/>
    % if object.event_id.user_id and object.event_id.user_id.signature:
        ${object.event_id.user_id.signature | safe}
    % endif
</div>
            </field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="auto_delete" eval="True"/>
            <field name="user_signature" eval="False"/>
        </record>

        <record id="calendar_template_meeting_reminder" model="mail.template">
            <field name="name">Calendar: Reminder</field>
            <field name="model_id" ref="calendar.model_calendar_attendee"/>
            <field name="subject">${object.event_id.name} - Reminder</field>
            <field name="email_from">${(object.event_id.user_id.email_formatted or user.email_formatted or '') | safe}</field>
            <field name="email_to">${('' if object.partner_id.email and object.partner_id.email == object.email else object.email) | safe}</field>
            <field name="partner_to">${object.partner_id.id if object.partner_id.email and object.partner_id.email == object.email else False}</field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="body_html" type="html">
<div>
    % set colors = {'needsAction': 'grey', 'accepted': 'green', 'tentative': '#FFFF00',  'declined': 'red'}
    <!--
        In a recurring event case, the object.event_id is always the first event
        This makes the event date (and a lot of other information) incorrect
    -->
    % set event_id = ctx.get('force_event_id') or object.event_id
    <p>
        Hello ${object.common_name},<br/><br/>
        This is a reminder for the below event :
    </p>
    <div style="text-align: center; margin: 16px 0px 16px 0px;">
        <a href="/calendar/meeting/accept?db=${'dbname' in ctx and ctx['dbname'] or ''}&amp;token=${object.access_token}&amp;action=${'action_id' in ctx and ctx['action_id'] or ''}&amp;id=${object.event_id.id}" 
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            Accept</a>
        <a href="/calendar/meeting/decline?db=${'dbname' in ctx and ctx['dbname'] or '' }&amp;token=${object.access_token}&amp;action=${'action_id' in ctx and ctx['action_id'] or ''}&amp;id=${object.event_id.id}"
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            Decline</a>
        <a href="/calendar/meeting/view?db=${'dbname' in ctx and ctx['dbname'] or ''}&amp;token=${object.access_token}&amp;action=${'action_id' in ctx and ctx['action_id'] or ''}&amp;id=${object.event_id.id}" 
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            View</a>
    </div>
    <table border="0" cellpadding="0" cellspacing="0"><tr>
        <td width="130px;">
            <div style="border-top-left-radius: 3px; border-top-right-radius: 3px; font-size: 12px; border-collapse: separate; text-align: center; font-weight: bold; color: #ffffff; min-height: 18px; background-color: #875A7B; border: 1px solid #875A7B;">
                ${event_id.get_interval('dayname', tz=object.partner_id.tz if not event_id.allday else None)}
            </div>
            <div style="font-size: 48px; min-height: auto; font-weight: bold; text-align: center; color: #5F5F5F; background-color: #F8F8F8; border: 1px solid #875A7B;">
                ${event_id.get_interval('day', tz=object.partner_id.tz if not event_id.allday else None)}
            </div>
            <div style='font-size: 12px; text-align: center; font-weight: bold; color: #ffffff; background-color: #875A7B;'>
                ${event_id.get_interval('month', tz=object.partner_id.tz if not event_id.allday else None)}
            </div>
            <div style="border-collapse: separate; color: #5F5F5F; text-align: center; font-size: 12px; border-bottom-right-radius: 3px; font-weight: bold; border: 1px solid #875A7B; border-bottom-left-radius: 3px;">
                ${not event_id.allday and event_id.get_interval('time', tz=object.partner_id.tz) or ''}
            </div>
        </td>
        <td width="20px;"/>
        <td style="padding-top: 5px;">
            <p><strong>Details of the event</strong></p>
            <ul>
                % if object.event_id.location:
                    <li>Location: ${object.event_id.location}
                        (<a target="_blank" href="http://maps.google.com/maps?oi=map&amp;q=${object.event_id.location}">View Map</a>)
                    </li>
                % endif
                % if object.event_id.description :
                    <li>Description: ${object.event_id.description}</li>
                % endif
                % if not object.event_id.allday and object.event_id.duration
                    <li>Duration: ${('%dH%02d' % (object.event_id.duration,(object.event_id.duration*60)%60))}</li>
                % endif
                <li>Attendees
                <ul>
                % for attendee in object.event_id.attendee_ids:
                    <li>
                        <div style="display: inline-block; border-radius: 50%; width: 10px; height: 10px; background:${colors[attendee.state] or 'white'};"> </div>
                        % if attendee.common_name != object.common_name:
                            <span style="margin-left:5px">${attendee.common_name}</span>
                        % else:
                            <span style="margin-left:5px">You</span>
                        % endif
                    </li>
                % endfor
                </ul></li>
            </ul>
        </td>
    </tr></table>
    <br/>
    Thank you,
    <br/>
    % if object.event_id.user_id and object.event_id.user_id.signature:
        ${object.event_id.user_id.signature | safe}
    % endif
</div>
            </field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="auto_delete" eval="True"/>
            <field name="user_signature" eval="False"/>
        </record>
    </data>
</odoo>

```

## File: models\calendar.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64

import babel.dates
import collections
import datetime
from datetime import timedelta, MAXYEAR
from dateutil import rrule
from dateutil.relativedelta import relativedelta
import logging
from operator import itemgetter
import pytz
import re
import time
import uuid

from odoo import api, fields, models
from odoo import tools
from odoo.addons.base.models.res_partner import _tz_get
from odoo.osv import expression
from odoo.tools.translate import _
from odoo.tools.misc import get_lang
from odoo.tools import DEFAULT_SERVER_DATE_FORMAT, DEFAULT_SERVER_DATETIME_FORMAT, pycompat
from odoo.exceptions import UserError, ValidationError


_logger = logging.getLogger(__name__)

VIRTUALID_DATETIME_FORMAT = "%Y%m%d%H%M%S"


def calendar_id2real_id(calendar_id=None, with_date=False):
    """ Convert a "virtual/recurring event id" (type string) into a real event id (type int).
        E.g. virtual/recurring event id is 4-20091201100000, so it will return 4.
        :param calendar_id: id of calendar
        :param with_date: if a value is passed to this param it will return dates based on value of withdate + calendar_id
        :return: real event id
    """
    if calendar_id and isinstance(calendar_id, str):
        res = [bit for bit in calendar_id.split('-') if bit]
        if len(res) == 2:
            real_id = res[0]
            if with_date:
                real_date = time.strftime(DEFAULT_SERVER_DATETIME_FORMAT, time.strptime(res[1], VIRTUALID_DATETIME_FORMAT))
                start = datetime.datetime.strptime(real_date, DEFAULT_SERVER_DATETIME_FORMAT)
                end = start + timedelta(hours=with_date)
                return (int(real_id), real_date, end.strftime(DEFAULT_SERVER_DATETIME_FORMAT))
            return int(real_id)
    return calendar_id and int(calendar_id) or calendar_id


def get_real_ids(ids):
    if isinstance(ids, (str, int)):
        return calendar_id2real_id(ids)

    if isinstance(ids, (list, tuple)):
        return [calendar_id2real_id(_id) for _id in ids]


def real_id2calendar_id(record_id, date):
    return '%s-%s' % (record_id, date.strftime(VIRTUALID_DATETIME_FORMAT))

def any_id2key(record_id):
    """ Creates a (real_id: int, thing: str) pair which allows ordering mixed
    collections of real and virtual events.

    The first item of the pair is the event's real id, the second one is
    either an empty string (for real events) or the datestring (for virtual
    ones)

    :param record_id:
    :type record_id: int | str
    :rtype: (int, str)
    """
    if isinstance(record_id, int):
        return record_id, u''

    (real_id, virtual_id) = record_id.split('-')
    return int(real_id), virtual_id

def is_calendar_id(record_id):
    return len(str(record_id).split('-')) != 1


SORT_ALIASES = {
    'start': 'sort_start',
    'start_date': 'sort_start',
    'start_datetime': 'sort_start',
}
def sort_remap(f):
    return SORT_ALIASES.get(f, f)


class Contacts(models.Model):
    _name = 'calendar.contacts'
    _description = 'Calendar Contacts'

    user_id = fields.Many2one('res.users', 'Me', required=True, default=lambda self: self.env.user)
    partner_id = fields.Many2one('res.partner', 'Employee', required=True)
    active = fields.Boolean('Active', default=True)

    _sql_constraints = [
        ('user_id_partner_id_unique', 'UNIQUE(user_id,partner_id)', 'An user cannot have twice the same contact.')
    ]

    @api.model
    def unlink_from_partner_id(self, partner_id):
        return self.search([('partner_id', '=', partner_id)]).unlink()


class Attendee(models.Model):
    """ Calendar Attendee Information """

    _name = 'calendar.attendee'
    _rec_name = 'common_name'
    _description = 'Calendar Attendee Information'

    def _default_access_token(self):
        return uuid.uuid4().hex

    STATE_SELECTION = [
        ('needsAction', 'Needs Action'),
        ('tentative', 'Uncertain'),
        ('declined', 'Declined'),
        ('accepted', 'Accepted'),
    ]

    state = fields.Selection(STATE_SELECTION, string='Status', readonly=True, default='needsAction',
        help="Status of the attendee's participation")
    common_name = fields.Char('Common name', compute='_compute_common_name', store=True)
    partner_id = fields.Many2one('res.partner', 'Contact', readonly=True)
    email = fields.Char('Email', help="Email of Invited Person")
    availability = fields.Selection([('free', 'Free'), ('busy', 'Busy')], 'Free/Busy', readonly=True)
    access_token = fields.Char('Invitation Token', default=_default_access_token)
    event_id = fields.Many2one('calendar.event', 'Meeting linked', ondelete='cascade')

    @api.depends('partner_id', 'partner_id.name', 'email')
    def _compute_common_name(self):
        for attendee in self:
            attendee.common_name = attendee.partner_id.name or attendee.email

    @api.onchange('partner_id')
    def _onchange_partner_id(self):
        """ Make entry on email and availability on change of partner_id field. """
        self.email = self.partner_id.email

    @api.model_create_multi
    def create(self, vals_list):
        for values in vals_list:
            if not values.get("email") and values.get("common_name"):
                common_nameval = values.get("common_name").split(':')
                email = [x for x in common_nameval if '@' in x] # TODO JEM : should be refactored
                values['email'] = email and email[0] or ''
                values['common_name'] = values.get("common_name")
        return super(Attendee, self).create(vals_list)

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        raise UserError(_('You cannot duplicate a calendar attendee.'))

    def _send_mail_to_attendees(self, template_xmlid, force_send=False, force_event_id=None):
        """ Send mail for event invitation to event attendees.
            :param template_xmlid: xml id of the email template to use to send the invitation
            :param force_send: if set to True, the mail(s) will be sent immediately (instead of the next queue processing)
        """
        res = False

        if self.env['ir.config_parameter'].sudo().get_param('calendar.block_mail') or self._context.get("no_mail_to_attendees"):
            return res

        calendar_view = self.env.ref('calendar.view_calendar_event_calendar')
        invitation_template = self.env.ref(template_xmlid)

        # get ics file for all meetings
        ics_files = force_event_id._get_ics_file() if force_event_id else self.mapped('event_id')._get_ics_file()

        # prepare rendering context for mail template
        colors = {
            'needsAction': 'grey',
            'accepted': 'green',
            'tentative': '#FFFF00',
            'declined': 'red'
        }
        rendering_context = dict(self._context)
        rendering_context.update({
            'color': colors,
            'action_id': self.env['ir.actions.act_window'].search([('view_id', '=', calendar_view.id)], limit=1).id,
            'dbname': self._cr.dbname,
            'base_url': self.env['ir.config_parameter'].sudo().get_param('web.base.url', default='http://localhost:8069'),
            'force_event_id': force_event_id,
        })
        invitation_template = invitation_template.with_context(rendering_context)

        # send email with attachments
        mail_ids = []
        for attendee in self:
            if attendee.email or attendee.partner_id.email:
                # FIXME: is ics_file text or bytes?
                event_id = force_event_id.id if force_event_id else attendee.event_id.id
                ics_file = ics_files.get(event_id)

                email_values = {
                    'model': None,  # We don't want to have the mail in the tchatter while in queue!
                    'res_id': None,
                    'author_id': attendee.event_id.user_id.partner_id.id or self.env.user.partner_id.id,
                }
                if ics_file:
                    email_values['attachment_ids'] = [
                        (0, 0, {'name': 'invitation.ics',
                                'mimetype': 'text/calendar',
                                'datas': base64.b64encode(ics_file)})
                    ]
                    # sudo is needed when the current user hasn't been added to the loop (i.e. neither in attendees, nor in owner)
                    mail_ids.append(invitation_template.with_context(no_document=True).sudo().send_mail(attendee.id, email_values=email_values, notif_layout='mail.mail_notification_light'))
                else:
                    mail_ids.append(invitation_template.send_mail(attendee.id, email_values=email_values, notif_layout='mail.mail_notification_light'))

        if force_send and mail_ids:
            res = self.env['mail.mail'].browse(mail_ids).send()

        return res

    def do_tentative(self):
        """ Makes event invitation as Tentative. """
        return self.write({'state': 'tentative'})

    def do_accept(self):
        """ Marks event invitation as Accepted. """
        result = self.write({'state': 'accepted'})
        for attendee in self:
            if attendee.event_id:
                attendee.event_id.message_post(body=_("%s has accepted invitation") % (attendee.common_name), subtype="calendar.subtype_invitation")
        return result

    def do_decline(self):
        """ Marks event invitation as Declined. """
        res = self.write({'state': 'declined'})
        for attendee in self:
            if attendee.event_id:
                attendee.event_id.message_post(body=_("%s has declined invitation") % (attendee.common_name), subtype="calendar.subtype_invitation")
        return res


class AlarmManager(models.AbstractModel):

    _name = 'calendar.alarm_manager'
    _description = 'Event Alarm Manager'

    def _get_next_potential_limit_alarm(self, alarm_type, seconds=None, partner_id=None):
        result = {}
        delta_request = """
            SELECT
                rel.calendar_event_id, max(alarm.duration_minutes) AS max_delta,min(alarm.duration_minutes) AS min_delta
            FROM
                calendar_alarm_calendar_event_rel AS rel
            LEFT JOIN calendar_alarm AS alarm ON alarm.id = rel.calendar_alarm_id
            WHERE alarm.alarm_type = %s
            GROUP BY rel.calendar_event_id
        """
        base_request = """
                    SELECT
                        cal.id,
                        cal.start - interval '1' minute  * calcul_delta.max_delta AS first_alarm,
                        CASE
                            WHEN cal.recurrency THEN cal.final_date - interval '1' minute  * calcul_delta.min_delta
                            ELSE cal.stop - interval '1' minute  * calcul_delta.min_delta
                        END as last_alarm,
                        cal.start as first_event_date,
                        CASE
                            WHEN cal.recurrency THEN cal.final_date
                            ELSE cal.stop
                        END as last_event_date,
                        calcul_delta.min_delta,
                        calcul_delta.max_delta,
                        cal.rrule AS rule
                    FROM
                        calendar_event AS cal
                    RIGHT JOIN calcul_delta ON calcul_delta.calendar_event_id = cal.id
             """

        filter_user = """
                RIGHT JOIN calendar_event_res_partner_rel AS part_rel ON part_rel.calendar_event_id = cal.id
                    AND part_rel.res_partner_id = %s
        """

        # Add filter on alarm type
        tuple_params = (alarm_type,)

        # Add filter on partner_id
        if partner_id:
            base_request += filter_user
            tuple_params += (partner_id, )

        # Upper bound on first_alarm of requested events
        first_alarm_max_value = ""
        if seconds is None:
            # first alarm in the future + 3 minutes if there is one, now otherwise
            first_alarm_max_value = """
                COALESCE((SELECT MIN(cal.start - interval '1' minute  * calcul_delta.max_delta)
                FROM calendar_event cal
                RIGHT JOIN calcul_delta ON calcul_delta.calendar_event_id = cal.id
                WHERE cal.start - interval '1' minute  * calcul_delta.max_delta > now() at time zone 'utc'
            ) + interval '3' minute, now() at time zone 'utc')"""
        else:
            # now + given seconds
            first_alarm_max_value = "(now() at time zone 'utc' + interval '%s' second )"
            tuple_params += (seconds,)

        self._cr.execute("""
                    WITH calcul_delta AS (%s)
                    SELECT *
                        FROM ( %s WHERE cal.active = True ) AS ALL_EVENTS
                       WHERE ALL_EVENTS.first_alarm < %s
                         AND ALL_EVENTS.last_event_date > (now() at time zone 'utc')
                   """ % (delta_request, base_request, first_alarm_max_value), tuple_params)

        for event_id, first_alarm, last_alarm, first_meeting, last_meeting, min_duration, max_duration, rule in self._cr.fetchall():
            result[event_id] = {
                'event_id': event_id,
                'first_alarm': first_alarm,
                'last_alarm': last_alarm,
                'first_meeting': first_meeting,
                'last_meeting': last_meeting,
                'min_duration': min_duration,
                'max_duration': max_duration,
                'rrule': rule
            }

        # determine accessible events
        events = self.env['calendar.event'].browse(result)
        result = {
            key: result[key]
            for key in set(events._filter_access_rules('read').ids)
        }
        return result

    def do_check_alarm_for_one_date(self, one_date, event, event_maxdelta, in_the_next_X_seconds, alarm_type, after=False, missing=False):
        """ Search for some alarms in the interval of time determined by some parameters (after, in_the_next_X_seconds, ...)
            :param one_date: date of the event to check (not the same that in the event browse if recurrent)
            :param event: Event browse record
            :param event_maxdelta: biggest duration from alarms for this event
            :param in_the_next_X_seconds: looking in the future (in seconds)
            :param after: if not False: will return alert if after this date (date as string - todo: change in master)
            :param missing: if not False: will return alert even if we are too late
            :param notif: Looking for type notification
            :param mail: looking for type email
        """
        result = []
        # TODO: remove event_maxdelta and if using it
        if one_date - timedelta(minutes=(missing and 0 or event_maxdelta)) < datetime.datetime.now() + timedelta(seconds=in_the_next_X_seconds):  # if an alarm is possible for this date
            for alarm in event.alarm_ids:
                if alarm.alarm_type == alarm_type and \
                    one_date - timedelta(minutes=(missing and 0 or alarm.duration_minutes)) < datetime.datetime.now() + timedelta(seconds=in_the_next_X_seconds) and \
                        (not after or one_date - timedelta(minutes=alarm.duration_minutes) > fields.Datetime.from_string(after)):
                        alert = {
                            'alarm_id': alarm.id,
                            'event_id': event.id,
                            'notify_at': one_date - timedelta(minutes=alarm.duration_minutes),
                        }
                        result.append(alert)
        return result

    @api.model
    def get_next_mail(self):
        last_notif_mail = fields.Datetime.to_string(self.env.context.get('lastcall') or fields.Datetime.now())

        cron = self.env.ref('calendar.ir_cron_scheduler_alarm', raise_if_not_found=False)
        if not cron:
            _logger.error("Cron for " + self._name + " can not be identified !")
            return False

        interval_to_second = {
            "weeks": 7 * 24 * 60 * 60,
            "days": 24 * 60 * 60,
            "hours": 60 * 60,
            "minutes": 60,
            "seconds": 1
        }

        if cron.interval_type not in interval_to_second:
            _logger.error("Cron delay can not be computed !")
            return False

        cron_interval = cron.interval_number * interval_to_second[cron.interval_type]

        all_meetings = self._get_next_potential_limit_alarm('email', seconds=cron_interval)

        for meeting in self.env['calendar.event'].browse(all_meetings):
            max_delta = all_meetings[meeting.id]['max_duration']

            if meeting.recurrency:
                at_least_one = False
                last_found = False
                for one_date in meeting._get_recurrent_date_by_event():
                    in_date_format = one_date.replace(tzinfo=None)
                    last_found = self.do_check_alarm_for_one_date(in_date_format, meeting, max_delta, 0, 'email', after=last_notif_mail, missing=True)
                    for alert in last_found:
                        self.do_mail_reminder(alert)
                        at_least_one = True  # if it's the first alarm for this recurrent event
                    if at_least_one and not last_found:  # if the precedent event had an alarm but not this one, we can stop the search for this event
                        break
            else:
                in_date_format = meeting.start
                last_found = self.do_check_alarm_for_one_date(in_date_format, meeting, max_delta, 0, 'email', after=last_notif_mail, missing=True)
                for alert in last_found:
                    self.do_mail_reminder(alert)

    @api.model
    def get_next_notif(self):
        partner = self.env.user.partner_id
        all_notif = []

        if not partner:
            return []

        all_meetings = self._get_next_potential_limit_alarm('notification', partner_id=partner.id)
        time_limit = 3600 * 24  # return alarms of the next 24 hours
        for event_id in all_meetings:
            max_delta = all_meetings[event_id]['max_duration']
            meeting = self.env['calendar.event'].browse(event_id)
            if meeting.recurrency:
                b_found = False
                last_found = False
                for one_date in meeting._get_recurrent_date_by_event():
                    in_date_format = one_date.replace(tzinfo=None)
                    last_found = self.do_check_alarm_for_one_date(in_date_format, meeting, max_delta, time_limit, 'notification', after=partner.calendar_last_notif_ack)
                    if last_found:
                        for alert in last_found:
                            all_notif.append(self.do_notif_reminder(alert))
                        if not b_found:  # if it's the first alarm for this recurrent event
                            b_found = True
                    if b_found and not last_found:  # if the precedent event had alarm but not this one, we can stop the search fot this event
                        break
            else:
                in_date_format = fields.Datetime.from_string(meeting.start)
                last_found = self.do_check_alarm_for_one_date(in_date_format, meeting, max_delta, time_limit, 'notification', after=partner.calendar_last_notif_ack)
                if last_found:
                    for alert in last_found:
                        all_notif.append(self.do_notif_reminder(alert))
        return all_notif

    def do_mail_reminder(self, alert):
        meeting = self.env['calendar.event'].browse(alert['event_id'])
        alarm = self.env['calendar.alarm'].browse(alert['alarm_id'])

        result = False
        if alarm.alarm_type == 'email':
            result = meeting.attendee_ids.filtered(lambda r: r.state != 'declined')._send_mail_to_attendees('calendar.calendar_template_meeting_reminder', force_send=True, force_event_id=meeting)
        return result

    def do_notif_reminder(self, alert):
        alarm = self.env['calendar.alarm'].browse(alert['alarm_id'])
        meeting = self.env['calendar.event'].browse(alert['event_id'])

        if alarm.alarm_type == 'notification':
            message = meeting.display_time

            delta = alert['notify_at'] - datetime.datetime.now()
            delta = delta.seconds + delta.days * 3600 * 24

            return {
                'alarm_id': alarm.id,
                'event_id': meeting.id,
                'title': meeting.name,
                'message': message,
                'timer': delta,
                'notify_at': fields.Datetime.to_string(alert['notify_at']),
            }

    def _notify_next_alarm(self, partner_ids):
        """ Sends through the bus the next alarm of given partners """
        notifications = []
        users = self.env['res.users'].search([('partner_id', 'in', tuple(partner_ids))])
        for user in users:
            notif = self.with_user(user).get_next_notif()
            notifications.append([(self._cr.dbname, 'calendar.alarm', user.partner_id.id), notif])
        if len(notifications) > 0:
            self.env['bus.bus'].sendmany(notifications)


class Alarm(models.Model):
    _name = 'calendar.alarm'
    _description = 'Event Alarm'

    @api.depends('interval', 'duration')
    def _compute_duration_minutes(self):
        for alarm in self:
            if alarm.interval == "minutes":
                alarm.duration_minutes = alarm.duration
            elif alarm.interval == "hours":
                alarm.duration_minutes = alarm.duration * 60
            elif alarm.interval == "days":
                alarm.duration_minutes = alarm.duration * 60 * 24
            else:
                alarm.duration_minutes = 0

    _interval_selection = {'minutes': 'Minutes', 'hours': 'Hours', 'days': 'Days'}

    name = fields.Char('Name', translate=True, required=True)
    alarm_type = fields.Selection([('notification', 'Notification'), ('email', 'Email')], string='Type', required=True, default='email')
    duration = fields.Integer('Remind Before', required=True, default=1)
    interval = fields.Selection(list(_interval_selection.items()), 'Unit', required=True, default='hours')
    duration_minutes = fields.Integer('Duration in minutes', compute='_compute_duration_minutes', store=True, help="Duration in minutes")

    @api.onchange('duration', 'interval', 'alarm_type')
    def _onchange_duration_interval(self):
        display_interval = self._interval_selection.get(self.interval, '')
        display_alarm_type = {key: value for key, value in self._fields['alarm_type']._description_selection(self.env)}[self.alarm_type]
        self.name = "%s - %s %s" % (display_alarm_type, self.duration, display_interval)

    def _update_cron(self):
        try:
            cron = self.env['ir.model.data'].sudo().get_object('calendar', 'ir_cron_scheduler_alarm')
        except ValueError:
            return False
        return cron.toggle(model=self._name, domain=[('alarm_type', '=', 'email')])

    @api.model
    def create(self, values):
        result = super(Alarm, self).create(values)
        self._update_cron()
        return result

    def write(self, values):
        result = super(Alarm, self).write(values)
        self._update_cron()
        return result

    def unlink(self):
        result = super(Alarm, self).unlink()
        self._update_cron()
        return result


class MeetingType(models.Model):

    _name = 'calendar.event.type'
    _description = 'Event Meeting Type'

    name = fields.Char('Name', required=True)

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Tag name already exists !"),
    ]


class Meeting(models.Model):
    """ Model for Calendar Event

        Special context keys :
            - `no_mail_to_attendees` : disabled sending email to attendees when creating/editing a meeting
    """

    _name = 'calendar.event'
    _description = "Calendar Event"
    _order = "id desc"
    _inherit = ["mail.thread"]

    @api.model
    def default_get(self, fields):
        # super default_model='crm.lead' for easier use in addons
        if self.env.context.get('default_res_model') and not self.env.context.get('default_res_model_id'):
            self = self.with_context(
                default_res_model_id=self.env['ir.model'].sudo().search([
                    ('model', '=', self.env.context['default_res_model'])
                ], limit=1).id
            )

        defaults = super(Meeting, self).default_get(fields)

        # support active_model / active_id as replacement of default_* if not already given
        if 'res_model_id' not in defaults and 'res_model_id' in fields and \
                self.env.context.get('active_model') and self.env.context['active_model'] != 'calendar.event':
            defaults['res_model_id'] = self.env['ir.model'].sudo().search([('model', '=', self.env.context['active_model'])], limit=1).id
        if 'res_id' not in defaults and 'res_id' in fields and \
                defaults.get('res_model_id') and self.env.context.get('active_id'):
            defaults['res_id'] = self.env.context['active_id']

        return defaults

    @api.model
    def _default_partners(self):
        """ When active_model is res.partner, the current partners should be attendees """
        partners = self.env.user.partner_id
        active_id = self._context.get('active_id')
        if self._context.get('active_model') == 'res.partner' and active_id:
            if active_id not in partners.ids:
                partners |= self.env['res.partner'].browse(active_id)
        return partners

    def _get_recurrent_dates_by_event(self):
        """ Get recurrent start and stop dates based on Rule string"""
        start_dates = self._get_recurrent_date_by_event(date_field='start')
        stop_dates = self._get_recurrent_date_by_event(date_field='stop')
        return list(zip(start_dates, stop_dates))

    def _get_recurrent_date_by_event(self, date_field='start'):
        """ Get recurrent dates based on Rule string and all event where recurrent_id is child

        date_field: the field containing the reference date information for recurrence computation
        """
        self.ensure_one()
        if date_field in self._fields and self._fields[date_field].type in ('date', 'datetime'):
            reference_date = self[date_field]
        else:
            reference_date = self.start

        timezone = pytz.timezone(self.event_tz) if self.event_tz else pytz.timezone(self._context.get('tz') or 'UTC')
        event_date = pytz.UTC.localize(fields.Datetime.from_string(reference_date))  # Add "+hh:mm" timezone
        if not event_date:
            event_date = datetime.datetime.now()

        use_naive_datetime = self.allday and self.rrule and 'UNTIL' in self.rrule and 'Z' not in self.rrule
        if not use_naive_datetime:
            # Convert the event date to saved timezone (or context tz) as it'll
            # define the correct hour/day asked by the user to repeat for recurrence.
            event_date = event_date.astimezone(timezone)

        # The start date is naive
        # the timezone will be applied, if necessary, at the very end of the process
        # to allow for DST timezone reevaluation
        rset1 = rrule.rrulestr(str(self.rrule), dtstart=event_date.replace(tzinfo=None), forceset=True, ignoretz=True)

        recurring_meetings_ids = self.env.context.get('recurrent_siblings_cache', {}).get(self.id)
        if recurring_meetings_ids is not None:
            recurring_meetings = self.browse(recurring_meetings_ids)
        else:
            recurring_meetings = self.with_context(active_test=False).search([('recurrent_id', '=', self.id)])

        # We handle a maximum of 50,000 meetings at a time, and clear the cache at each step to
        # control the memory usage.
        invalidate = False
        for meetings in self.env.cr.split_for_in_conditions(recurring_meetings, size=50000):
            if invalidate:
                self.invalidate_cache()
            for meeting in meetings:
                recurring_date = fields.Datetime.from_string(meeting.recurrent_id_date)
                if use_naive_datetime:
                    recurring_date = recurring_date.replace(tzinfo=None)
                else:
                    if not recurring_date.tzinfo:
                        recurring_date = pytz.UTC.localize(recurring_date)
                    recurring_date = recurring_date.astimezone(timezone).replace(tzinfo=None)
                if date_field == "stop":
                    recurring_date += timedelta(hours=self.duration)
                rset1.exdate(recurring_date)
            invalidate = True

        def naive_tz_to_utc(d):
            return timezone.localize(d.replace(tzinfo=None), is_dst=True).astimezone(pytz.UTC)
        return [naive_tz_to_utc(d) if not use_naive_datetime else d for d in rset1 if d.year < MAXYEAR]

    def _get_recurrency_end_date(self):
        """ Return the last date a recurring event happens, according to its end_type. """
        self.ensure_one()
        data = self.read(['final_date', 'recurrency', 'rrule_type', 'count', 'end_type', 'stop', 'interval'])[0]

        if not data.get('recurrency'):
            return False

        end_type = data.get('end_type')
        final_date = data.get('final_date')
        if end_type == 'count' and all(data.get(key) for key in ['count', 'rrule_type', 'stop', 'interval']):
            count = (data['count'] + 1) * data['interval']
            delay, mult = {
                'daily': ('days', 1),
                'weekly': ('days', 7),
                'monthly': ('months', 1),
                'yearly': ('years', 1),
            }[data['rrule_type']]

            deadline = fields.Datetime.from_string(data['stop'])
            computed_final_date = False
            while not computed_final_date and count > 0:
                try:  # may crash if year > 9999 (in case of recurring events)
                    computed_final_date = deadline + relativedelta(**{delay: count * mult})
                except ValueError:
                    count -= data['interval']
            return computed_final_date or deadline
        return final_date

    def _find_my_attendee(self):
        """ Return the first attendee where the user connected has been invited
            from all the meeting_ids in parameters.
        """
        self.ensure_one()
        for attendee in self.attendee_ids:
            if self.env.user.partner_id == attendee.partner_id:
                return attendee
        return False

    @api.model
    def _get_date_formats(self):
        """ get current date and time format, according to the context lang
            :return: a tuple with (format date, format time)
        """
        lang = get_lang(self.env)
        return (lang.date_format, lang.time_format)

    @api.model
    def _get_recurrent_fields(self):
        return ['byday', 'recurrency', 'final_date', 'rrule_type', 'month_by',
                'interval', 'count', 'end_type', 'mo', 'tu', 'we', 'th', 'fr', 'sa',
                'su', 'day', 'week_list']

    @api.model
    def _get_display_time(self, start, stop, zduration, zallday):
        """ Return date and time (from to from) based on duration with timezone in string. Eg :
                1) if user add duration for 2 hours, return : August-23-2013 at (04-30 To 06-30) (Europe/Brussels)
                2) if event all day ,return : AllDay, July-31-2013
        """
        timezone = self._context.get('tz') or self.env.user.partner_id.tz or 'UTC'

        # get date/time format according to context
        format_date, format_time = self._get_date_formats()

        # convert date and time into user timezone
        self_tz = self.with_context(tz=timezone)
        date = fields.Datetime.context_timestamp(self_tz, fields.Datetime.from_string(start))
        date_deadline = fields.Datetime.context_timestamp(self_tz, fields.Datetime.from_string(stop))

        # convert into string the date and time, using user formats
        to_text = pycompat.to_text
        date_str = to_text(date.strftime(format_date))
        time_str = to_text(date.strftime(format_time))

        if zallday:
            display_time = _("AllDay , %s") % (date_str)
        elif zduration < 24:
            duration = date + timedelta(minutes=round(zduration*60))
            duration_time = to_text(duration.strftime(format_time))
            display_time = _(u"%s at (%s To %s) (%s)") % (
                date_str,
                time_str,
                duration_time,
                timezone,
            )
        else:
            dd_date = to_text(date_deadline.strftime(format_date))
            dd_time = to_text(date_deadline.strftime(format_time))
            display_time = _(u"%s at %s To\n %s at %s (%s)") % (
                date_str,
                time_str,
                dd_date,
                dd_time,
                timezone,
            )
        return display_time

    def _get_duration(self, start, stop):
        """ Get the duration value between the 2 given dates. """
        if start and stop:
            diff = fields.Datetime.from_string(stop) - fields.Datetime.from_string(start)
            if diff:
                duration = float(diff.days) * 24 + (float(diff.seconds) / 3600)
                return round(duration, 2)
            return 0.0

    def _compute_is_highlighted(self):
        if self.env.context.get('active_model') == 'res.partner':
            partner_id = self.env.context.get('active_id')
            for event in self:
                if event.partner_ids.filtered(lambda s: s.id == partner_id):
                    event.is_highlighted = True
                else:
                    event.is_highlighted = False
        else:
            for event in self:
                event.is_highlighted = False

    name = fields.Char('Meeting Subject', required=True, states={'done': [('readonly', True)]})
    state = fields.Selection([('draft', 'Unconfirmed'), ('open', 'Confirmed')], string='Status', readonly=True, tracking=True, default='draft')

    is_attendee = fields.Boolean('Attendee', compute='_compute_attendee')
    attendee_status = fields.Selection(Attendee.STATE_SELECTION, string='Attendee Status', compute='_compute_attendee')
    display_time = fields.Char('Event Time', compute='_compute_display_time')
    display_start = fields.Char('Date', compute='_compute_display_start', store=True)
    start = fields.Datetime('Start', required=True, help="Start date of an event, without time for full days events")
    stop = fields.Datetime('Stop', required=True, help="Stop date of an event, without time for full days events")

    allday = fields.Boolean('All Day', states={'done': [('readonly', True)]}, default=False)
    start_date = fields.Date('Start Date', compute='_compute_dates', inverse='_inverse_dates', store=True, states={'done': [('readonly', True)]}, tracking=True)
    start_datetime = fields.Datetime('Start DateTime', compute='_compute_dates', inverse='_inverse_dates', store=True, states={'done': [('readonly', True)]}, tracking=True)
    stop_date = fields.Date('End Date', compute='_compute_dates', inverse='_inverse_dates', store=True, states={'done': [('readonly', True)]}, tracking=True)
    stop_datetime = fields.Datetime('End Datetime', compute='_compute_dates', inverse='_inverse_dates', store=True, states={'done': [('readonly', True)]}, tracking=True)  # old date_deadline
    event_tz = fields.Selection('_event_tz_get', string='Timezone', default=lambda self: self.env.context.get('tz') or self.user_id.tz)
    duration = fields.Float('Duration', states={'done': [('readonly', True)]})
    description = fields.Text('Description', states={'done': [('readonly', True)]})
    privacy = fields.Selection([('public', 'Everyone'), ('private', 'Only me'), ('confidential', 'Only internal users')], 'Privacy', default='public', states={'done': [('readonly', True)]})
    location = fields.Char('Location', states={'done': [('readonly', True)]}, tracking=True, help="Location of Event")
    show_as = fields.Selection([('free', 'Free'), ('busy', 'Busy')], 'Show Time as', states={'done': [('readonly', True)]}, default='busy')

    # linked document
    res_id = fields.Integer('Document ID')
    res_model_id = fields.Many2one('ir.model', 'Document Model', ondelete='cascade')
    res_model = fields.Char('Document Model Name', related='res_model_id.model', readonly=True, store=True)
    activity_ids = fields.One2many('mail.activity', 'calendar_event_id', string='Activities')

    #redifine message_ids to remove autojoin to avoid search to crash in get_recurrent_ids
    message_ids = fields.One2many(auto_join=False)

    # RECURRENCE FIELD
    rrule = fields.Char('Recurrent Rule', compute='_compute_rrule', inverse='_inverse_rrule', store=True)
    rrule_type = fields.Selection([
        ('daily', 'Days'),
        ('weekly', 'Weeks'),
        ('monthly', 'Months'),
        ('yearly', 'Years')
    ], string='Recurrence', states={'done': [('readonly', True)]}, help="Let the event automatically repeat at that interval")
    recurrency = fields.Boolean('Recurrent', help="Recurrent Meeting")
    recurrent_id = fields.Integer('Recurrent ID')
    recurrent_id_date = fields.Datetime('Recurrent ID date')
    end_type = fields.Selection([
        ('count', 'Number of repetitions'),
        ('end_date', 'End date')
    ], string='Recurrence Termination', default='count')
    interval = fields.Integer(string='Repeat Every', default=1, help="Repeat every (Days/Week/Month/Year)")
    count = fields.Integer(string='Repeat', help="Repeat x times", default=1)
    mo = fields.Boolean('Mon')
    tu = fields.Boolean('Tue')
    we = fields.Boolean('Wed')
    th = fields.Boolean('Thu')
    fr = fields.Boolean('Fri')
    sa = fields.Boolean('Sat')
    su = fields.Boolean('Sun')
    month_by = fields.Selection([
        ('date', 'Date of month'),
        ('day', 'Day of month')
    ], string='Option', default='date')
    day = fields.Integer('Date of month', default=1)
    week_list = fields.Selection([
        ('MO', 'Monday'),
        ('TU', 'Tuesday'),
        ('WE', 'Wednesday'),
        ('TH', 'Thursday'),
        ('FR', 'Friday'),
        ('SA', 'Saturday'),
        ('SU', 'Sunday')
    ], string='Weekday')
    byday = fields.Selection([
        ('1', 'First'),
        ('2', 'Second'),
        ('3', 'Third'),
        ('4', 'Fourth'),
        ('5', 'Fifth'),
        ('-1', 'Last')
    ], string='By day')
    final_date = fields.Date('Repeat Until')
    user_id = fields.Many2one('res.users', 'Owner', states={'done': [('readonly', True)]}, default=lambda self: self.env.user)
    partner_id = fields.Many2one('res.partner', string='Responsible', related='user_id.partner_id', readonly=True)
    active = fields.Boolean('Active', default=True, help="If the active field is set to false, it will allow you to hide the event alarm information without removing it.")
    categ_ids = fields.Many2many('calendar.event.type', 'meeting_category_rel', 'event_id', 'type_id', 'Tags')
    attendee_ids = fields.One2many('calendar.attendee', 'event_id', 'Participant', ondelete='cascade')
    partner_ids = fields.Many2many('res.partner', 'calendar_event_res_partner_rel', string='Attendees', states={'done': [('readonly', True)]}, default=_default_partners)
    alarm_ids = fields.Many2many('calendar.alarm', 'calendar_alarm_calendar_event_rel', string='Reminders', ondelete="restrict", copy=False)
    is_highlighted = fields.Boolean(compute='_compute_is_highlighted', string='Is the Event Highlighted')

    def _compute_attendee(self):
        for meeting in self:
            attendee = meeting._find_my_attendee()
            meeting.is_attendee = bool(attendee)
            meeting.attendee_status = attendee.state if attendee else 'needsAction'

    def _compute_display_time(self):
        for meeting in self:
            meeting.display_time = self._get_display_time(meeting.start, meeting.stop, meeting.duration, meeting.allday)

    @api.depends('allday', 'start_date', 'start_datetime')
    def _compute_display_start(self):
        for meeting in self:
            meeting.display_start = meeting.start_date if meeting.allday else meeting.start_datetime

    @api.depends('allday', 'start', 'stop')
    def _compute_dates(self):
        """ Adapt the value of start_date(time)/stop_date(time) according to start/stop fields and allday. Also, compute
            the duration for not allday meeting ; otherwise the duration is set to zero, since the meeting last all the day.
        """
        for meeting in self:
            if meeting.allday and meeting.start and meeting.stop:
                meeting.start_date = meeting.start.date()
                meeting.start_datetime = False
                meeting.stop_date = meeting.stop.date()
                meeting.stop_datetime = False

                meeting.duration = self._get_duration(meeting.start, meeting.stop)
            else:
                meeting.start_date = False
                meeting.start_datetime = meeting.start
                meeting.stop_date = False
                meeting.stop_datetime = meeting.stop

                meeting.duration = self._get_duration(meeting.start, meeting.stop)

    def _inverse_dates(self):
        for meeting in self:
            if meeting.allday:

                # Convention break:
                # stop and start are NOT in UTC in allday event
                # in this case, they actually represent a date
                # i.e. Christmas is on 25/12 for everyone
                # even if people don't celebrate it simultaneously
                enddate = fields.Datetime.from_string(meeting.stop_date)
                enddate = enddate.replace(hour=18)

                startdate = fields.Datetime.from_string(meeting.start_date)
                startdate = startdate.replace(hour=8)  # Set 8 AM

                meeting.write({
                    'start': startdate.replace(tzinfo=None),
                    'stop': enddate.replace(tzinfo=None)
                })
            else:
                meeting.write({'start': meeting.start_datetime,
                               'stop': meeting.stop_datetime})

    @api.depends('byday', 'recurrency', 'final_date', 'rrule_type', 'month_by', 'interval', 'count', 'end_type', 'mo', 'tu', 'we', 'th', 'fr', 'sa', 'su', 'day', 'week_list')
    def _compute_rrule(self):
        """ Gets Recurrence rule string according to value type RECUR of iCalendar from the values given.
            :return dictionary of rrule value.
        """
        for meeting in self:
            if meeting.recurrency:
                meeting.rrule = meeting._rrule_serialize()
            else:
                meeting.rrule = ''

    def _inverse_rrule(self):
        for meeting in self:
            if meeting.rrule:
                data = self._rrule_default_values()
                data['recurrency'] = True
                data.update(self._rrule_parse(meeting.rrule, data, meeting.start))
                meeting.write(data)

    @api.model
    def _event_tz_get(self):
        return _tz_get(self)

    @api.constrains('start_datetime', 'stop_datetime', 'start_date', 'stop_date')
    def _check_closing_date(self):
        for meeting in self:
            if meeting.start_datetime and meeting.stop_datetime and meeting.stop_datetime < meeting.start_datetime:
                raise ValidationError(
                    _('The ending date and time cannot be earlier than the starting date and time.') + '\n' +
                    _("Meeting '%s' starts '%s' and ends '%s'") % (meeting.name, meeting.start_datetime, meeting.stop_datetime)
                )
            if meeting.start_date and meeting.stop_date and meeting.stop_date < meeting.start_date:
                raise ValidationError(
                    _('The ending date cannot be earlier than the starting date.') + '\n' +
                    _("Meeting '%s' starts '%s' and ends '%s'") % (meeting.name, meeting.start_date, meeting.stop_date)
                )

    @api.onchange('start_datetime', 'duration')
    def _onchange_duration(self):
        if self.start_datetime:
            start = self.start_datetime
            self.start = self.start_datetime
            # Round the duration (in hours) to the minute to avoid weird situations where the event
            # stops at 4:19:59, later displayed as 4:19.
            self.stop = start + timedelta(minutes=round((self.duration or 1.0) * 60))
            if self.allday:
                self.stop -= timedelta(seconds=1)

    @api.onchange('start_date')
    def _onchange_start_date(self):
        if self.start_date:
            self.start = datetime.datetime.combine(self.start_date, datetime.time.min)

    @api.onchange('stop_date')
    def _onchange_stop_date(self):
        if self.stop_date:
            self.stop = datetime.datetime.combine(self.stop_date, datetime.time.max)

    ####################################################
    # Calendar Business, Reccurency, ...
    ####################################################

    def _get_ics_file(self):
        """ Returns iCalendar file for the event invitation.
            :returns a dict of .ics file content for each meeting
        """
        result = {}

        def ics_datetime(idate, allday=False):
            if idate:
                if allday:
                    return fields.Date.to_date(idate)
                else:
                    idate = fields.Datetime.to_datetime(idate)
                    return idate.replace(tzinfo=pytz.timezone('UTC'))
            return False

        try:
            # FIXME: why isn't this in CalDAV?
            import vobject
        except ImportError:
            _logger.warning("The `vobject` Python module is not installed, so iCal file generation is unavailable. Please install the `vobject` Python module")
            return result

        for meeting in self:
            cal = vobject.iCalendar()
            event = cal.add('vevent')

            if not meeting.start or not meeting.stop:
                raise UserError(_("First you have to specify the date of the invitation."))
            event.add('created').value = ics_datetime(fields.Datetime.now())
            event.add('dtstart').value = ics_datetime(meeting.start, meeting.allday)
            event.add('dtend').value = ics_datetime(meeting.stop, meeting.allday)
            event.add('summary').value = meeting.name
            if meeting.description:
                event.add('description').value = meeting.description
            if meeting.location:
                event.add('location').value = meeting.location
            if meeting.rrule:
                event.add('rrule').value = meeting.rrule

            if meeting.alarm_ids:
                for alarm in meeting.alarm_ids:
                    valarm = event.add('valarm')
                    interval = alarm.interval
                    duration = alarm.duration
                    trigger = valarm.add('TRIGGER')
                    trigger.params['related'] = ["START"]
                    if interval == 'days':
                        delta = timedelta(days=duration)
                    elif interval == 'hours':
                        delta = timedelta(hours=duration)
                    elif interval == 'minutes':
                        delta = timedelta(minutes=duration)
                    trigger.value = delta
                    valarm.add('DESCRIPTION').value = alarm.name or u'Odoo'
            for attendee in meeting.attendee_ids:
                attendee_add = event.add('attendee')
                attendee_add.value = u'MAILTO:' + (attendee.email or u'')
            result[meeting.id] = cal.serialize().encode('utf-8')

        return result

    def create_attendees(self):
        current_user = self.env.user
        result = {}
        for meeting in self:
            alreay_meeting_partners = meeting.attendee_ids.mapped('partner_id')
            meeting_attendees = self.env['calendar.attendee']
            meeting_partners = self.env['res.partner']
            for partner in meeting.partner_ids.filtered(lambda partner: partner not in alreay_meeting_partners):
                values = {
                    'partner_id': partner.id,
                    'email': partner.email,
                    'event_id': meeting.id,
                }

                if self._context.get('google_internal_event_id', False):
                    values['google_internal_event_id'] = self._context.get('google_internal_event_id')

                # current user don't have to accept his own meeting
                if partner == self.env.user.partner_id:
                    values['state'] = 'accepted'

                attendee = self.env['calendar.attendee'].create(values)

                meeting_attendees |= attendee
                meeting_partners |= partner

            if meeting_attendees and not self._context.get('detaching'):
                to_notify = meeting_attendees.filtered(lambda a: a.email != current_user.email)
                to_notify._send_mail_to_attendees('calendar.calendar_template_meeting_invitation')

            if meeting_attendees:
                meeting.write({'attendee_ids': [(4, meeting_attendee.id) for meeting_attendee in meeting_attendees]})

            if meeting_partners:
                meeting.message_subscribe(partner_ids=meeting_partners.ids)

            # We remove old attendees who are not in partner_ids now.
            all_partners = meeting.partner_ids
            all_partner_attendees = meeting.attendee_ids.mapped('partner_id')
            old_attendees = meeting.attendee_ids
            partners_to_remove = all_partner_attendees + meeting_partners - all_partners

            attendees_to_remove = self.env["calendar.attendee"]
            if partners_to_remove:
                attendees_to_remove = self.env["calendar.attendee"].search([('partner_id', 'in', partners_to_remove.ids), ('event_id', '=', meeting.id)])
                attendees_to_remove.unlink()

            result[meeting.id] = {
                'new_attendees': meeting_attendees,
                'old_attendees': old_attendees,
                'removed_attendees': attendees_to_remove,
                'removed_partners': partners_to_remove
            }
        return result

    def get_search_fields(self, order_fields, r_date=None):
        sort_fields = {}
        for field in order_fields:
            if field == 'id' and r_date:
                sort_fields[field] = real_id2calendar_id(self.id, r_date)
            else:
                sort_fields[field] = self[field]
                if isinstance(self[field], models.BaseModel):
                    name_get = self[field].mapped('display_name')
                    sort_fields[field] = name_get and name_get[0] or ''
        if r_date:
            sort_fields['sort_start'] = r_date.strftime(VIRTUALID_DATETIME_FORMAT)
        else:
            display_start = self.display_start
            sort_fields['sort_start'] = display_start.replace(' ', '').replace('-', '') if display_start else False
        return sort_fields

    def get_recurrent_ids(self, domain, order=None):
        """ Gives virtual event ids for recurring events. This method gives ids of dates
            that comes between start date and end date of calendar views
            :param order:   The fields (comma separated, format "FIELD {DESC|ASC}") on which
                            the events should be sorted
        """
        if order:
            order_fields = [field.split()[0] for field in order.split(',')]
        else:
            # fallback on self._order defined on the model
            order_fields = [field.split()[0] for field in self._order.split(',')]

        if 'id' not in order_fields:
            order_fields.append('id')

        # code does not handle '!' operator
        domain = expression.distribute_not(expression.normalize_domain(domain))

        leaf_evaluations = None
        recurrent_ids = [meeting.id for meeting in self if meeting.recurrency and meeting.rrule]
        #compose a query of the type SELECT id, condition1 as domain1, condition2 as domaine2
        #This allows to load leaf interpretation of the where clause in one query
        #leaf_evaluations is then used when running custom interpretation of domain for recuring events
        if self and recurrent_ids:
            select_fields = ["id"]
            where_params_list = []
            for pos, arg in enumerate(domain):
                # hack: if we received recurrent ids, we need to clean them
                if arg[0] == 'id' and arg[1] in ('in', 'not in'):
                    arg = (arg[0], arg[1], [calendar_id2real_id(id) for id in arg[2]])
                if not arg[0] in ('start', 'stop', 'final_date', '&', '|'):
                    e = expression.expression([arg], self)
                    where_clause, where_params = e.to_sql()  # CAUTION, wont work if field is autojoin, not supported
                    select_fields.append("%s as \"%s\"" % (where_clause, str(pos)))
                    where_params_list += where_params
            if len(select_fields) > 1:
                query = "SELECT %s FROM calendar_event WHERE id in %%s" % (", ".join(select_fields))  # could be improved by only taking event with recurency ?
                where_params_list += [tuple(recurrent_ids)]
                self._cr.execute(query, where_params_list)
                leaf_evaluations = dict([(row['id'], row) for row in self._cr.dictfetchall()])
        result_data = []
        result = []

        recurrent_siblings_cache = {i: [] for i in recurrent_ids} # create empty entries to avoid additional queries for missing entries
        children_ids = super(Meeting, self.with_context(active_test=False))._search([('recurrent_id', 'in', recurrent_ids)])
        for item in self.browse(children_ids).read(['recurrent_id']):
            recurrent_siblings_cache[item['recurrent_id']].append(item['id'])

        recurrent_env = self.with_context(recurrent_siblings_cache=recurrent_siblings_cache).env
        for meeting in self:
            if not meeting.recurrency or not meeting.rrule:
                result.append(meeting.id)
                result_data.append(meeting.get_search_fields(order_fields))
                continue
            rdates = meeting.with_env(recurrent_env)._get_recurrent_dates_by_event()

            for r_start_date, r_stop_date in rdates:
                # fix domain evaluation
                # step 1: check date and replace expression by True or False, replace other expressions by True
                # step 2: evaluation of & and |
                # check if there are one False
                pile = []
                ok = True
                r_date = r_start_date  # default for empty domain
                for pos, arg in enumerate(domain):
                    if str(arg[0]) in ('start', 'stop', 'final_date'):
                        if str(arg[0]) == 'start':
                            r_date = r_start_date
                        else:
                            r_date = r_stop_date
                        if arg[2] and len(arg[2]) > len(r_date.strftime(DEFAULT_SERVER_DATE_FORMAT)):
                            dformat = DEFAULT_SERVER_DATETIME_FORMAT
                        else:
                            dformat = DEFAULT_SERVER_DATE_FORMAT
                        if (arg[1] == '='):
                            ok = r_date.strftime(dformat) == arg[2]
                        if (arg[1] == '>'):
                            ok = r_date.strftime(dformat) > arg[2]
                        if (arg[1] == '<'):
                            ok = r_date.strftime(dformat) < arg[2]
                        if (arg[1] == '>='):
                            ok = r_date.strftime(dformat) >= arg[2]
                        if (arg[1] == '<='):
                            ok = r_date.strftime(dformat) <= arg[2]
                        if (arg[1] == '!='):
                            ok = r_date.strftime(dformat) != arg[2]
                        pile.append(ok)
                    elif str(arg) == str('&') or str(arg) == str('|'):
                        pile.append(arg)
                    elif leaf_evaluations and meeting.id in leaf_evaluations:
                        pile.append(bool(leaf_evaluations[meeting.id][str(pos)]))
                    else:
                        pile.append(True)
                pile.reverse()
                new_pile = []
                for item in pile:
                    if not isinstance(item, str):
                        res = item
                    elif str(item) == str('&'):
                        first = new_pile.pop()
                        second = new_pile.pop()
                        res = first and second
                    elif str(item) == str('|'):
                        first = new_pile.pop()
                        second = new_pile.pop()
                        res = first or second
                    new_pile.append(res)

                if [True for item in new_pile if not item]:
                    continue
                result_data.append(meeting.get_search_fields(order_fields, r_date=r_start_date))

        # seq of (field, should_reverse)
        sort_spec = list(tools.unique(
            (sort_remap(key.split()[0]), key.lower().endswith(' desc'))
            for key in (order or self._order).split(',')
        ))
        def key(record):
            # we need to deal with undefined fields, as sorted requires an homogeneous iterable
            def boolean_product(x):
                x = False if (isinstance(x, models.Model) and not x) else x
                if isinstance(x, bool):
                    return (x, x)
                return (True, x)
            # first extract the values for each key column (ids need special treatment)
            vals_spec = (
                (any_id2key(record[name]) if name == 'id' else boolean_product(record[name]), desc)
                for name, desc in sort_spec
            )
            # then Reverse if the value matches a "desc" column
            return tuple(
                (tools.Reverse(v) if desc else v)
                for v, desc in vals_spec
            )
        return [r['id'] for r in sorted(result_data, key=key)]

    def _rrule_serialize(self):
        """ Compute rule string according to value type RECUR of iCalendar
            :return: string containing recurring rule (empty if no rule)
        """
        if self.interval <= 0:
            raise UserError(_('The interval cannot be negative.'))
        if self.end_type == 'count' and self.count < 0:
            raise UserError(_('The number of repetitions  cannot be negative.'))

        def get_week_string(freq):
            weekdays = ['mo', 'tu', 'we', 'th', 'fr', 'sa', 'su']
            if freq == 'weekly':
                byday = [field.upper() for field in weekdays if self[field]]
                if byday:
                    return ';BYDAY=' + ','.join(byday)
            return ''

        def get_month_string(freq):
            if freq == 'monthly':
                if self.month_by == 'date' and (self.day < 1 or self.day > 31):
                    raise UserError(_("Please select a proper day of the month."))

                if self.month_by == 'day' and self.byday and self.week_list:  # Eg : Second Monday of the month
                    return ';BYDAY=' + self.byday + self.week_list
                elif self.month_by == 'date':  # Eg : 16th of the month
                    return ';BYMONTHDAY=' + str(self.day)
            return ''

        def get_end_date():
            final_date = fields.Date.to_string(self.final_date)
            end_date_new = ''.join((re.compile('\d')).findall(final_date)) + 'T235959Z' if final_date else False
            return (self.end_type == 'count' and (';COUNT=' + str(max(1, self.count))) or '') +\
                ((end_date_new and self.end_type == 'end_date' and (';UNTIL=' + end_date_new)) or '')

        freq = self.rrule_type  # day/week/month/year
        result = ''
        if freq:
            interval_string = self.interval and (';INTERVAL=' + str(self.interval)) or ''
            result = 'FREQ=' + freq.upper() + get_week_string(freq) + interval_string + get_end_date() + get_month_string(freq)
        return result

    def _rrule_default_values(self):
        return {
            'byday': False,
            'recurrency': False,
            'final_date': False,
            'rrule_type': False,
            'month_by': False,
            'interval': 0,
            'count': False,
            'end_type': False,
            'mo': False,
            'tu': False,
            'we': False,
            'th': False,
            'fr': False,
            'sa': False,
            'su': False,
            'day': False,
            'week_list': False
        }

    def _rrule_parse(self, rule_str, data, date_start):
        day_list = ['mo', 'tu', 'we', 'th', 'fr', 'sa', 'su']
        rrule_type = ['yearly', 'monthly', 'weekly', 'daily']
        ddate = fields.Datetime.from_string(date_start)
        if 'Z' in rule_str and not ddate.tzinfo:
            ddate = ddate.replace(tzinfo=pytz.timezone('UTC'))
            rule = rrule.rrulestr(rule_str, dtstart=ddate)
        else:
            rule = rrule.rrulestr(rule_str, dtstart=ddate)

        if rule._freq > 0 and rule._freq < 4:
            data['rrule_type'] = rrule_type[rule._freq]
        data['count'] = rule._count
        data['interval'] = rule._interval
        data['final_date'] = rule._until and rule._until.strftime(DEFAULT_SERVER_DATETIME_FORMAT)
        #repeat weekly
        if rule._byweekday:
            for i in range(0, 7):
                if i in rule._byweekday:
                    data[day_list[i]] = True
            data['rrule_type'] = 'weekly'
        #repeat monthly by nweekday ((weekday, weeknumber), )
        if rule._bynweekday:
            data['week_list'] = day_list[list(rule._bynweekday)[0][0]].upper()
            data['byday'] = str(list(rule._bynweekday)[0][1])
            data['month_by'] = 'day'
            data['rrule_type'] = 'monthly'

        if rule._bymonthday:
            data['day'] = list(rule._bymonthday)[0]
            data['month_by'] = 'date'
            data['rrule_type'] = 'monthly'

        #repeat yearly but for odoo it's monthly, take same information as monthly but interval is 12 times
        if rule._bymonth:
            data['interval'] = data['interval'] * 12

        #FIXEME handle forever case
        #end of recurrence
        #in case of repeat for ever that we do not support right now
        if not (data.get('count') or data.get('final_date')):
            data['count'] = 100
        if data.get('count'):
            data['end_type'] = 'count'
        else:
            data['end_type'] = 'end_date'
        return data

    def get_interval(self, interval, tz=None):
        """ Format and localize some dates to be used in email templates
            :param string interval: Among 'day', 'month', 'dayname' and 'time' indicating the desired formatting
            :param string tz: Timezone indicator (optional)
            :return unicode: Formatted date or time (as unicode string, to prevent jinja2 crash)
        """
        self.ensure_one()
        date = fields.Datetime.from_string(self.start)

        if tz:
            timezone = pytz.timezone(tz or 'UTC')
            date = date.replace(tzinfo=pytz.timezone('UTC')).astimezone(timezone)

        if interval == 'day':
            # Day number (1-31)
            result = str(date.day)

        elif interval == 'month':
            # Localized month name and year
            result = babel.dates.format_date(date=date, format='MMMM y', locale=get_lang(self.env).code)

        elif interval == 'dayname':
            # Localized day name
            result = babel.dates.format_date(date=date, format='EEEE', locale=get_lang(self.env).code)

        elif interval == 'time':
            # Localized time
            # FIXME: formats are specifically encoded to bytes, maybe use babel?
            dummy, format_time = self._get_date_formats()
            result = tools.ustr(date.strftime(format_time + " %Z"))

        return result

    def get_display_time_tz(self, tz=False):
        """ get the display_time of the meeting, forcing the timezone. This method is called from email template, to not use sudo(). """
        self.ensure_one()
        if tz:
            self = self.with_context(tz=tz)
        return self._get_display_time(self.start, self.stop, self.duration, self.allday)

    def detach_recurring_event(self, values=None):
        """ Detach a virtual recurring event by duplicating the original and change reccurent values
            :param values : dict of value to override on the detached event
        """
        if not values:
            values = {}

        real_id = calendar_id2real_id(self.id)
        meeting_origin = self.browse(real_id)

        data = self.read(['allday', 'start', 'stop', 'rrule', 'duration'])[0]
        if data.get('rrule'):
            data.update(
                values,
                recurrent_id=real_id,
                recurrent_id_date=data.get('start'),
                rrule_type=False,
                rrule='',
                recurrency=False,
                final_date=False,
                end_type=False
            )

            # do not copy the id
            if data.get('id'):
                del data['id']
            return meeting_origin.with_context(detaching=True).copy(default=data)

    def action_detach_recurring_event(self):
        meeting = self.detach_recurring_event()
        return {
            'type': 'ir.actions.act_window',
            'res_model': 'calendar.event',
            'view_mode': 'form',
            'res_id': meeting.id,
            'target': 'current',
            'flags': {'form': {'action_buttons': True, 'options': {'mode': 'edit'}}}
        }

    def action_open_calendar_event(self):
        if self.res_model and self.res_id:
            return self.env[self.res_model].browse(self.res_id).get_formview_action()
        return False

    def action_sendmail(self):
        email = self.env.user.email
        if email:
            for meeting in self:
                meeting.attendee_ids._send_mail_to_attendees('calendar.calendar_template_meeting_invitation')
        return True

    ####################################################
    # Messaging
    ####################################################

    def _get_message_unread(self):
        self.message_unread_counter = False
        self.message_unread = False
        id_map = {x: calendar_id2real_id(x) for x in self.ids}
        real = self.browse(set(id_map.values()))
        super(Meeting, real)._get_message_unread()
        for event in self:
            if event._origin.id == id_map[event._origin.id]:
                continue
            rec = self.browse(id_map[event._origin.id])
            event.message_unread_counter = rec.message_unread_counter
            event.message_unread = rec.message_unread

    def _get_message_needaction(self):
        self.message_needaction_counter = False
        self.message_needaction = False
        id_map = {x: calendar_id2real_id(x) for x in self.ids}
        real = self.browse(set(id_map.values()))
        super(Meeting, real)._get_message_needaction()
        for event in self:
            if event._origin.id == id_map[event._origin.id]:
                continue
            rec = self.browse(id_map[event._origin.id])
            event.message_needaction_counter = rec.message_needaction_counter
            event.message_needaction = rec.message_needaction

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, **kwargs):
        thread_id = self.id
        if isinstance(self.id, str):
            thread_id = get_real_ids(self.id)
        if self.env.context.get('default_date'):
            context = dict(self.env.context)
            del context['default_date']
            self = self.with_context(context)
        return super(Meeting, self.browse(thread_id)).message_post(**kwargs)

    def message_subscribe(self, partner_ids=None, channel_ids=None, subtype_ids=None):
        records = self.browse(get_real_ids(self.ids))
        return super(Meeting, records).message_subscribe(partner_ids=partner_ids, channel_ids=channel_ids, subtype_ids=subtype_ids)

    def _message_subscribe(self, partner_ids=None, channel_ids=None, subtype_ids=None, customer_ids=None):
        records = self.browse(get_real_ids(self.ids))
        return super(Meeting, records)._message_subscribe(partner_ids=partner_ids, channel_ids=channel_ids, subtype_ids=subtype_ids, customer_ids=customer_ids)

    def message_unsubscribe(self, partner_ids=None, channel_ids=None):
        records = self.browse(get_real_ids(self.ids))
        return super(Meeting, records).message_unsubscribe(partner_ids=partner_ids, channel_ids=channel_ids)

    ####################################################
    # ORM Overrides
    ####################################################

    def get_metadata(self):
        real = self.browse({calendar_id2real_id(x) for x in self.ids})
        return super(Meeting, real).get_metadata()

    @api.model
    def _name_search(self, name='', args=None, operator='ilike', limit=100, name_get_uid=None):
        for arg in args:
            if arg[0] == 'id':
                for n, calendar_id in enumerate(arg[2]):
                    if isinstance(calendar_id, str):
                        arg[2][n] = calendar_id.split('-')[0]
        return super(Meeting, self)._name_search(name=name, args=args, operator=operator, limit=limit, name_get_uid=name_get_uid)

    def write(self, values):
        # FIXME: neverending recurring events
        if 'rrule' in values:
            values['rrule'] = self._fix_rrule(values)

        # compute duration, only if start and stop are modified
        if not 'duration' in values and 'start' in values and 'stop' in values:
            values['duration'] = self._get_duration(values['start'], values['stop'])

        self._sync_activities(values)

        # process events one by one
        for meeting in self:
            # special write of complex IDS
            real_ids = []
            new_ids = []
            if not is_calendar_id(meeting.id):
                real_ids = [int(meeting.id)]
            else:
                real_event_id = calendar_id2real_id(meeting.id)

                # if we are setting the recurrency flag to False or if we are only changing fields that
                # should be only updated on the real ID and not on the virtual (like message_follower_ids):
                # then set real ids to be updated.
                blacklisted = any(key in values for key in ('start', 'stop', 'active'))
                if not values.get('recurrency', True) or not blacklisted:
                    real_ids = [real_event_id]
                else:
                    data = meeting.read(['start', 'stop', 'rrule', 'duration'])[0]
                    if data.get('rrule'):
                        new_ids = meeting.with_context(dont_notify=True).detach_recurring_event(values).ids  # to prevent multiple _notify_next_alarm

            new_meetings = self.browse(new_ids)
            real_meetings = self.browse(real_ids)
            all_meetings = real_meetings + new_meetings
            super(Meeting, real_meetings).write(values)

            # set end_date for calendar searching
            if any(field in values for field in ['recurrency', 'end_type', 'count', 'rrule_type', 'start', 'stop']):
                for real_meeting in real_meetings:
                    if real_meeting.recurrency and real_meeting.end_type == u'count':
                        final_date = real_meeting._get_recurrency_end_date()
                        super(Meeting, real_meeting).write({'final_date': final_date})

            attendees_create = False
            if values.get('partner_ids', False):
                attendees_create = all_meetings.with_context(dont_notify=True).create_attendees()  # to prevent multiple _notify_next_alarm

            # Notify attendees if there is an alarm on the modified event, or if there was an alarm
            # that has just been removed, as it might have changed their next event notification
            if not self._context.get('dont_notify'):
                if len(meeting.alarm_ids) > 0 or values.get('alarm_ids'):
                    partners_to_notify = meeting.partner_ids.ids
                    event_attendees_changes = attendees_create and real_ids and attendees_create[real_ids[0]]
                    if event_attendees_changes:
                        partners_to_notify.extend(event_attendees_changes['removed_partners'].ids)
                    self.env['calendar.alarm_manager']._notify_next_alarm(partners_to_notify)

            if (values.get('start_date') or values.get('start_datetime') or
                    (values.get('start') and self.env.context.get('from_ui'))) and values.get('active', True):
                for current_meeting in all_meetings:
                    if attendees_create:
                        attendees_create = attendees_create[current_meeting.id]
                        attendee_to_email = attendees_create['old_attendees'] - attendees_create['removed_attendees']
                    else:
                        attendee_to_email = current_meeting.attendee_ids

                    if attendee_to_email:
                        attendee_to_email._send_mail_to_attendees('calendar.calendar_template_meeting_changedate')
        return True

    @api.model_create_multi
    def create(self, vals_list):
        defaults = self.default_get(['activity_ids', 'res_model_id', 'res_id', 'user_id'])
        meeting_activity_type = self.env['mail.activity.type'].search([('category', '=', 'meeting')], limit=1)
        model_has_activity_ids = {}
        for values in vals_list:
            # FIXME: neverending recurring events
            if 'rrule' in values:
                values['rrule'] = self._fix_rrule(values)

            if not 'user_id' in values:  # Else bug with quick_create when we are filter on an other user
                values['user_id'] = self.env.user.id

            # compute duration, if not given
            if not 'duration' in values:
                values['duration'] = self._get_duration(values['start'], values['stop'])

            # created from calendar: try to create an activity on the related record
            if not values.get('activity_ids'):
                res_model_id = values.get('res_model_id', defaults.get('res_model_id'))
                res_id = values.get('res_id', defaults.get('res_id'))
                user_id = values.get('user_id', defaults.get('user_id'))
                if not defaults.get('activity_ids') and res_model_id and res_id:
                    if res_model_id in model_has_activity_ids:
                        has_activity_ids = model_has_activity_ids[res_model_id]
                    else:
                        has_activity_ids = hasattr(self.env[self.env['ir.model'].sudo().browse(res_model_id).model], 'activity_ids')
                        model_has_activity_ids[res_model_id] = has_activity_ids
                    if has_activity_ids:
                        if meeting_activity_type:
                            activity_vals = {
                                'res_model_id': res_model_id,
                                'res_id': res_id,
                                'activity_type_id': meeting_activity_type.id,
                            }
                            if user_id:
                                activity_vals['user_id'] = user_id
                            values['activity_ids'] = [(0, 0, activity_vals)]

        meetings = super(Meeting, self).create(vals_list)

        for meeting, vals in zip(meetings, vals_list):
            meeting._sync_activities(vals)

        for meeting in meetings:
            final_date = meeting._get_recurrency_end_date()
            # `dont_notify=True` in context to prevent multiple _notify_next_alarm
            meeting.with_context(dont_notify=True).write({'final_date': final_date})
            meeting.with_context(dont_notify=True).create_attendees()

            # Notify attendees if there is an alarm on the created event, as it might have changed their
            # next event notification
            if not self._context.get('dont_notify'):
                if len(meeting.alarm_ids) > 0:
                    self.env['calendar.alarm_manager']._notify_next_alarm(meeting.partner_ids.ids)
        return meetings

    def export_data(self, fields_to_export):
        """ Override to convert virtual ids to ids """
        records = self.browse(set(get_real_ids(self.ids)))
        return super(Meeting, records).export_data(fields_to_export)

    def _read(self, fields):
        select = [(x, calendar_id2real_id(x)) for x in self.ids]
        result = super(Meeting, self.browse(real_id for calendar_id, real_id in select))._read(fields)
        for calendar_id, real_id in select:
            if real_id != calendar_id:
                calendar = self.browse(calendar_id)
                real = self.browse(real_id)
                ls = calendar_id2real_id(calendar_id, with_date=real.duration or 1)
                for field in fields:
                    f = self._fields[field]
                    if field in ('start', 'start_date', 'start_datetime'):
                        value = ls[1]
                    elif field in ('stop', 'stop_date', 'stop_datetime'):
                        value = ls[2]
                    elif field == 'display_time':
                        value = self._get_display_time(ls[1], ls[2], real.duration, real.allday)
                    else:
                        value = self.env.cache.get(real, f)
                    self.env.cache.set(calendar, f, value)
        return result

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        if 'date' in groupby:
            raise UserError(_('Group by date is not supported, use the calendar view instead.'))
        return super(Meeting, self.with_context(virtual_id=False)).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

    def read(self, fields=None, load='_classic_read'):
        if not fields:
            fields = list(self._fields)
        fields2 = fields and fields[:]
        EXTRAFIELDS = ('privacy', 'user_id', 'duration', 'allday', 'start', 'rrule')
        for f in EXTRAFIELDS:
            if fields and (f not in fields):
                fields2.append(f)

        select = [(x, calendar_id2real_id(x)) for x in self.ids]
        real_events = self.browse([real_id for calendar_id, real_id in select])
        real_data = super(Meeting, real_events).read(fields=fields2, load=load)
        real_data = dict((d['id'], d) for d in real_data)

        result = []
        for calendar_id, real_id in select:
            if not real_data.get(real_id):
                continue
            res = real_data[real_id].copy()
            ls = calendar_id2real_id(calendar_id, with_date=res and res.get('duration', 0) > 0 and res.get('duration') or 1)
            if not isinstance(ls, (str, int)) and len(ls) >= 2:
                res['start'] = ls[1]
                res['stop'] = ls[2]
                if 'display_time' in fields:
                    res['display_time'] = self._get_display_time(ls[1], ls[2], res['duration'], res['allday'])

            res['id'] = calendar_id
            result.append(res)

        recurrent_fields = self._get_recurrent_fields()
        public_fields = set(recurrent_fields + ['id', 'active', 'allday', 'start', 'stop', 'display_start', 'display_stop', 'duration', 'user_id', 'state', 'interval', 'count', 'recurrent_id', 'recurrent_id_date', 'rrule'])
        for r in result:
            if r['user_id']:
                user_id = type(r['user_id']) in (tuple, list) and r['user_id'][0] or r['user_id']
                partner_id = self.env.user.partner_id.id
                if user_id == self.env.user.id or partner_id in r.get("partner_ids", []):
                    continue
            if r['privacy'] == 'private':
                for f in r:
                    if f not in public_fields:
                        if isinstance(r[f], list):
                            r[f] = []
                        else:
                            r[f] = False
                    if f in ['name', 'display_name']:
                        r[f] = _('Busy')

        for r in result:
            for k in EXTRAFIELDS:
                if (k in r) and (fields and (k not in fields)):
                    del r[k]
        return result

    def name_get(self):
        """ Hide private events' name for events which don't belong to the current user
        """
        hidden = self.filtered(
            lambda evt:
                evt.privacy == 'private' and
                evt.user_id.id != self.env.uid and
                self.env.user.partner_id not in evt.partner_ids
        )

        shown = self - hidden
        shown_names = super(Meeting, shown).name_get()
        obfuscated_names = [(eid, _('Busy')) for eid in hidden.ids]
        return shown_names + obfuscated_names

    def unlink(self, can_be_deleted=True):
        # Get concerned attendees to notify them if there is an alarm on the unlinked events,
        # as it might have changed their next event notification
        events = self.search([('id', 'in', self.ids), ('alarm_ids', '!=', False)])
        partner_ids = events.mapped('partner_ids').ids

        records_to_exclude = self.env['calendar.event']
        records_to_unlink = self.env['calendar.event'].with_context(recompute=False)

        for meeting in self:
            if can_be_deleted and not is_calendar_id(meeting.id):  # if  ID REAL
                if meeting.recurrent_id:
                    records_to_exclude |= meeting
                else:
                    # int() required because 'id' from calendar view is a string, since it can be calendar virtual id
                    records_to_unlink |= self.browse(int(meeting.id))
            else:
                records_to_exclude |= meeting

        result = False
        if records_to_unlink:
            result = super(Meeting, records_to_unlink).unlink()
        if records_to_exclude:
            result = records_to_exclude.with_context(dont_notify=True).write({'active': False})

        # Notify the concerned attendees (must be done after removing the events)
        self.env['calendar.alarm_manager']._notify_next_alarm(partner_ids)
        return result

    @api.model
    def _search(self, args, offset=0, limit=None, order=None, count=False, access_rights_uid=None):
        if self._context.get('mymeetings'):
            args += [('partner_ids', 'in', self.env.user.partner_id.ids)]

        new_args = []
        for arg in args:
            new_arg = arg
            if arg[0] in ('stop_date', 'stop_datetime', 'stop',) and arg[1] in ('>=', '>', '=',):
                if self._context.get('virtual_id', True):
                    new_args += ['|', '&', ('recurrency', '=', 1), ('final_date', arg[1], arg[2])]
            elif arg[0] == "id":
                new_arg = (arg[0], arg[1], get_real_ids(arg[2]))
            new_args.append(new_arg)

        # update_custom_fields: context used by the ORM to check if custom fields (studio) should be updated
        virtual_id_fallback = not self._context.get('update_custom_fields')
        if not self._context.get('virtual_id', virtual_id_fallback):
            return super(Meeting, self)._search(new_args, offset=offset, limit=limit, order=order, count=count, access_rights_uid=access_rights_uid)

        if any(arg[0] == 'start' for arg in args) and \
           not any(arg[0] in ('stop', 'final_date') for arg in args):
            # domain with a start filter but with no stop clause should be extended
            # e.g. start=2017-01-01, count=5 => virtual occurences must be included in ('start', '>', '2017-01-02')
            start_args = new_args
            new_args = []
            for arg in start_args:
                new_arg = arg
                if arg[0] in ('start_date', 'start_datetime', 'start',):
                    new_args += ['|', '&', ('recurrency', '=', 1), ('final_date', arg[1], arg[2])]
                new_args.append(new_arg)

        # offset, limit, order and count must be treated separately as we may need to deal with virtual ids
        event_ids = super(Meeting, self)._search(new_args, offset=0, limit=0, order=None, count=False, access_rights_uid=access_rights_uid)
        events = self.browse(event_ids)
        events = self.browse(events.get_recurrent_ids(args, order=order))
        if count:
            return len(events)
        elif limit:
            return events[offset: offset + limit].ids
        return events.ids

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        self.ensure_one()
        default = default or {}
        return super(Meeting, self.browse(calendar_id2real_id(self.id))).copy(default)

    def _sync_activities(self, values):
        # update activities
        if self.mapped('activity_ids'):
            activity_values = {}
            if values.get('name'):
                activity_values['summary'] = values['name']
            if values.get('description'):
                activity_values['note'] = tools.plaintext2html(values['description'])
            if values.get('start'):
                # self.start is a datetime UTC *only when the event is not allday*
                # activty.date_deadline is a date (No TZ, but should represent the day in which the user's TZ is)
                # See 72254129dbaeae58d0a2055cba4e4a82cde495b7 for the same issue, but elsewhere
                deadline = fields.Datetime.from_string(values['start'])
                user_tz = self.env.context.get('tz')
                if user_tz and not self.allday:
                    deadline = pytz.UTC.localize(deadline)
                    deadline = deadline.astimezone(pytz.timezone(user_tz))
                activity_values['date_deadline'] = deadline.date()
            if values.get('user_id'):
                activity_values['user_id'] = values['user_id']
            if activity_values.keys():
                self.mapped('activity_ids').write(activity_values)

    @api.model
    def _fix_rrule(self, values):
        rule_str = values.get('rrule')
        if rule_str:
            if 'UNTIL' not in rule_str and 'COUNT' not in rule_str:
                rule_str += ';COUNT=100'
        return rule_str

    def change_attendee_status(self, status):
        attendee = self.attendee_ids.filtered(lambda x: x.partner_id == self.env.user.partner_id)
        if status == 'accepted':
            return attendee.do_accept()
        elif status == 'declined':
            return attendee.do_decline()
        else:
            return attendee.do_tentative()

```

## File: models\ir_attachment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models

from odoo.addons.calendar.models.calendar import get_real_ids


class Attachment(models.Model):

    _inherit = "ir.attachment"

    @api.model
    def _search(self, args, offset=0, limit=None, order=None, count=False, access_rights_uid=None):
        """ Convert the search on real ids in the case it was asked on virtual ids, then call super() """
        args = list(args)
        if any([leaf for leaf in args if leaf[0] == "res_model" and leaf[2] == 'calendar.event']):
            for index in range(len(args)):
                if args[index][0] == "res_id" and isinstance(args[index][2], str):
                    args[index] = (args[index][0], args[index][1], get_real_ids(args[index][2]))
        return super(Attachment, self)._search(args, offset=offset, limit=limit, order=order, count=count, access_rights_uid=access_rights_uid)

    def write(self, vals):
        """ When posting an attachment (new or not), convert the virtual ids in real ids. """
        if isinstance(vals.get('res_id'), str):
            vals['res_id'] = get_real_ids(vals.get('res_id'))
        return super(Attachment, self).write(vals)

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import odoo
from odoo import models, SUPERUSER_ID
from odoo.http import request
from odoo.api import Environment

from werkzeug.exceptions import BadRequest


class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    @classmethod
    def _auth_method_calendar(cls):
        token = request.params['token']
        dbname = request.params['db']

        registry = odoo.registry(dbname)
        error_message = False
        with registry.cursor() as cr:
            env = Environment(cr, SUPERUSER_ID, {})

            attendee = env['calendar.attendee'].sudo().search([('access_token', '=', token)], limit=1)
            if not attendee:
                error_message = """Invalid Invitation Token."""
            elif request.session.uid and request.session.login != 'anonymous':
                # if valid session but user is not match
                user = env['res.users'].sudo().browse(request.session.uid)
                if attendee.partner_id != user.partner_id:
                    error_message = """Invitation cannot be forwarded via email. This event/meeting belongs to %s and you are logged in as %s. Please ask organizer to add you.""" % (attendee.email, user.email)
        if error_message:
            raise BadRequest(error_message)

        return True

```

## File: models\mail_activity.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields, tools, _


class MailActivityType(models.Model):
    _inherit = "mail.activity.type"

    category = fields.Selection(selection_add=[('meeting', 'Meeting')])


class MailActivity(models.Model):
    _inherit = "mail.activity"

    calendar_event_id = fields.Many2one('calendar.event', string="Calendar Meeting", ondelete='cascade')

    def action_create_calendar_event(self):
        self.ensure_one()
        action = self.env.ref('calendar.action_calendar_event').read()[0]
        action['context'] = {
            'default_activity_type_id': self.activity_type_id.id,
            'default_res_id': self.env.context.get('default_res_id'),
            'default_res_model': self.env.context.get('default_res_model'),
            'default_name': self.summary or self.res_name,
            'default_description': self.note and tools.html2plaintext(self.note).strip() or '',
            'default_activity_ids': [(6, 0, self.ids)],
            'initial_date': self.date_deadline,
        }
        return action

    def _action_done(self, feedback=False, attachment_ids=False):
        events = self.mapped('calendar_event_id')
        messages, activities = super(MailActivity, self)._action_done(feedback=feedback, attachment_ids=attachment_ids)
        if feedback:
            for event in events:
                description = event.description
                description = '%s\n%s%s' % (description or '', _("Feedback: "), feedback)
                event.write({'description': description})
        return messages, activities

    def unlink_w_meeting(self):
        events = self.mapped('calendar_event_id')
        res = self.unlink()
        events.unlink()
        return res

```

## File: models\mail_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models

from odoo.addons.calendar.models.calendar import get_real_ids


class Message(models.Model):

    _inherit = "mail.message"

    @api.model
    def _search(self, args, offset=0, limit=None, order=None, count=False, access_rights_uid=None):
        """ Convert the search on real ids in the case it was asked on virtual ids, then call super() """
        args = list(args)
        for index in range(len(args)):
            if args[index][0] == "res_id":
                if isinstance(args[index][2], str):
                    args[index] = (args[index][0], args[index][1], get_real_ids(args[index][2]))
                elif isinstance(args[index][2], list):
                    args[index] = (args[index][0], args[index][1], [get_real_ids(x) for x in args[index][2]])
        return super(Message, self)._search(args, offset=offset, limit=limit, order=order, count=count, access_rights_uid=access_rights_uid)

    @api.model
    def _find_allowed_model_wise(self, doc_model, doc_dict):
        if doc_model == 'calendar.event':
            order = self._context.get('order', self.env[doc_model]._order)
            for virtual_id in self.env[doc_model].with_context(active_test=False).search([('id', 'in', list(doc_dict))], order=order).ids:
                doc_dict.setdefault(virtual_id, doc_dict[get_real_ids(virtual_id)])
        return super(Message, self)._find_allowed_model_wise(doc_model, doc_dict)

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime

from odoo import api, fields, models

from odoo.addons.calendar.models.calendar import get_real_ids


class Partner(models.Model):
    _inherit = 'res.partner'

    calendar_last_notif_ack = fields.Datetime('Last notification marked as read from base Calendar', default=fields.Datetime.now)

    def get_attendee_detail(self, meeting_id):
        """ Return a list of tuple (id, name, status)
            Used by base_calendar.js : Many2ManyAttendee
        """
        datas = []
        meeting = None
        if meeting_id:
            meeting = self.env['calendar.event'].browse(get_real_ids(meeting_id))

        for partner in self:
            data = partner.name_get()[0]
            data = [data[0], data[1], False, partner.color]
            if meeting:
                for attendee in meeting.attendee_ids:
                    if attendee.partner_id.id == partner.id:
                        data[2] = attendee.state
            datas.append(data)
        return datas

    @api.model
    def _set_calendar_last_notif_ack(self):
        partner = self.env['res.users'].browse(self.env.context.get('uid',self.env.uid)).partner_id
        partner.write({'calendar_last_notif_ack': datetime.now()})
        return

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime

from odoo import api, fields, models, modules, _
from pytz import timezone, UTC


class Users(models.Model):
    _inherit = 'res.users'

    def _systray_get_calendar_event_domain(self):
        tz = self.env.user.tz
        start_dt = datetime.datetime.utcnow()
        if tz:
            start_date = timezone(tz).localize(start_dt).astimezone(UTC).date()
        else:
            start_date = datetime.date.today()
        end_dt = datetime.datetime.combine(start_date, datetime.time.max)
        if tz:
            end_dt = timezone(tz).localize(end_dt).astimezone(UTC)

        return ['&',
                '&',
                    '|', ['start', '>=', fields.Datetime.to_string(start_dt)], ['stop', '>=', fields.Datetime.to_string(start_dt)],
                    ['start', '<=', fields.Datetime.to_string(end_dt)],
                ('attendee_ids.partner_id', '=', self.env.user.partner_id.id)]

    @api.model
    def systray_get_activities(self):
        res = super(Users, self).systray_get_activities()

        meetings_lines = self.env['calendar.event'].search_read(
            self._systray_get_calendar_event_domain(),
            ['id', 'start', 'name', 'allday', 'attendee_status'],
            order='start')
        meetings_lines = [line for line in meetings_lines if line['attendee_status'] != 'declined']
        if meetings_lines:
            meeting_label = _("Today's Meetings")
            meetings_systray = {
                'type': 'meeting',
                'name': meeting_label,
                'model': 'calendar.event',
                'icon': modules.module.get_module_icon(self.env['calendar.event']._original_module),
                'meetings': meetings_lines,
            }
            res.insert(0, meetings_systray)

        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_attachment
from . import ir_http
from . import res_partner
from . import mail_message
from . import calendar
from . import mail_activity
from . import res_users

```

## File: security\calendar_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="calendar_event_rule_my" model="ir.rule">
            <field name="name">Own events</field>
            <field ref="model_calendar_event" name="model_id"/>
            <field name="domain_force">[('partner_ids','in',user.partner_id.id)]</field>
            <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
        </record>

        <record id="calendar_event_rule_employee" model="ir.rule">
            <field ref="model_calendar_event" name="model_id"/>
            <field name="name">All Calendar Event for employees</field>
            <field name="domain_force">[(1,'=',1)]</field>
            <field eval="[(4,ref('base.group_user'))]" name="groups"/>
        </record>

        <record id="calendar_attendee_rule_my" model="ir.rule">
            <field name="name">Own attendees</field>
            <field ref="model_calendar_attendee" name="model_id"/>
            <field name="domain_force">[(1,'=',1)]</field>
            <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
        </record>

        <record id="calendar_event_rule_private" model="ir.rule">
            <field ref="model_calendar_event" name="model_id"/>
            <field name="name">Private events</field>
            <field name="domain_force">['|', ('privacy', '!=', 'private'), '&amp;', ('privacy', '=', 'private'), ('partner_ids', 'in', user.partner_id.id)]</field>
            <field name="perm_read" eval="False"/>
            <field name="perm_write" eval="True"/>
            <field name="perm_create" eval="True"/>
            <field name="perm_unlink" eval="True"/>
        </record>

    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_calendar_attendee_portal,calendar.attendee_portal,model_calendar_attendee,base.group_portal,0,0,0,0
access_calendar_attendee_employee,calendar.attendee_employee,model_calendar_attendee,base.group_user,1,1,1,1
access_calendar_alarm,calendar.alarm,model_calendar_alarm,base.group_user,1,1,1,1
access_calendar_event_all_user,calendar.event_all_user,model_calendar_event,base.group_portal,1,0,0,0
access_calendar_event_all_employee,calendar.event_all_employee,model_calendar_event,base.group_user,1,1,1,1
access_calendar_event_partner_manager,calendar.event.partner.manager,model_calendar_event,base.group_partner_manager,1,1,1,1
access_calendar_event_type_all,calendar.event.type.all,model_calendar_event_type,base.group_user,1,0,0,0
access_calendar_event_type_manager,calendar.event.type.manager,model_calendar_event_type,base.group_system,1,1,1,1
access_calendar_alarm_manager,access_calendar_alarm_manager,model_calendar_alarm_manager,base.group_user,1,1,1,1
access_calendar_contacts_all,access_calendar_contacts_all,model_calendar_contacts,base.group_user,1,1,1,1
access_calendar_contacts,access_calendar_contacts,model_calendar_contacts,base.group_system,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70">
    <defs>
        <path id="icon-a" d="M4,5.35309892e-14 C36.4160122,9.87060235e-15 58.0836068,-3.97961823e-14 65,5.07020818e-14 C69,6.733808e-14 70,1 70,5 C70,43.0488877 70,62.4235458 70,65 C70,69 69,70 65,70 C61,70 9,70 4,70 C1,70 7.10542736e-15,69 7.10542736e-15,65 C7.25721566e-15,62.4676575 3.83358709e-14,41.8005206 3.60818146e-14,5 C-1.13686838e-13,1 1,5.75716207e-14 4,5.35309892e-14 Z"/>
        <linearGradient id="icon-c" x1="100%" x2="0%" y1="0%" y2="100%">
            <stop offset="0%" stop-color="#CDC484"/>
            <stop offset="100%" stop-color="#B5AA59"/>
        </linearGradient>
        <path id="icon-d" d="M28.3839286,37.7083333 L24.9017857,37.7083333 C24.3272321,37.7083333 23.8571429,37.2513021 23.8571429,36.6927083 L23.8571429,33.3072917 C23.8571429,32.7486979 24.3272321,32.2916667 24.9017857,32.2916667 L28.3839286,32.2916667 C28.9584821,32.2916667 29.4285714,32.7486979 29.4285714,33.3072917 L29.4285714,36.6927083 C29.4285714,37.2513021 28.9584821,37.7083333 28.3839286,37.7083333 Z M37.7857143,36.6927083 L37.7857143,33.3072917 C37.7857143,32.7486979 37.315625,32.2916667 36.7410714,32.2916667 L33.2589286,32.2916667 C32.684375,32.2916667 32.2142857,32.7486979 32.2142857,33.3072917 L32.2142857,36.6927083 C32.2142857,37.2513021 32.684375,37.7083333 33.2589286,37.7083333 L36.7410714,37.7083333 C37.315625,37.7083333 37.7857143,37.2513021 37.7857143,36.6927083 Z M46.1428571,36.6927083 L46.1428571,33.3072917 C46.1428571,32.7486979 45.6727679,32.2916667 45.0982143,32.2916667 L41.6160714,32.2916667 C41.0415179,32.2916667 40.5714286,32.7486979 40.5714286,33.3072917 L40.5714286,36.6927083 C40.5714286,37.2513021 41.0415179,37.7083333 41.6160714,37.7083333 L45.0982143,37.7083333 C45.6727679,37.7083333 46.1428571,37.2513021 46.1428571,36.6927083 Z M37.7857143,44.8177083 L37.7857143,41.4322917 C37.7857143,40.8736979 37.315625,40.4166667 36.7410714,40.4166667 L33.2589286,40.4166667 C32.684375,40.4166667 32.2142857,40.8736979 32.2142857,41.4322917 L32.2142857,44.8177083 C32.2142857,45.3763021 32.684375,45.8333333 33.2589286,45.8333333 L36.7410714,45.8333333 C37.315625,45.8333333 37.7857143,45.3763021 37.7857143,44.8177083 Z M29.4285714,44.8177083 L29.4285714,41.4322917 C29.4285714,40.8736979 28.9584821,40.4166667 28.3839286,40.4166667 L24.9017857,40.4166667 C24.3272321,40.4166667 23.8571429,40.8736979 23.8571429,41.4322917 L23.8571429,44.8177083 C23.8571429,45.3763021 24.3272321,45.8333333 24.9017857,45.8333333 L28.3839286,45.8333333 C28.9584821,45.8333333 29.4285714,45.3763021 29.4285714,44.8177083 Z M46.1428571,44.8177083 L46.1428571,41.4322917 C46.1428571,40.8736979 45.6727679,40.4166667 45.0982143,40.4166667 L41.6160714,40.4166667 C41.0415179,40.4166667 40.5714286,40.8736979 40.5714286,41.4322917 L40.5714286,44.8177083 C40.5714286,45.3763021 41.0415179,45.8333333 41.6160714,45.8333333 L45.0982143,45.8333333 C45.6727679,45.8333333 46.1428571,45.3763021 46.1428571,44.8177083 Z M54.5,22.8125 L54.5,52.6041667 C54.5,54.8470052 52.6283482,56.6666667 50.3214286,56.6666667 L19.6785714,56.6666667 C17.3716518,56.6666667 15.5,54.8470052 15.5,52.6041667 L15.5,22.8125 C15.5,20.5696615 17.3716518,18.75 19.6785714,18.75 L23.8571429,18.75 L23.8571429,14.3489583 C23.8571429,13.7903646 24.3272321,13.3333333 24.9017857,13.3333333 L28.3839286,13.3333333 C28.9584821,13.3333333 29.4285714,13.7903646 29.4285714,14.3489583 L29.4285714,18.75 L40.5714286,18.75 L40.5714286,14.3489583 C40.5714286,13.7903646 41.0415179,13.3333333 41.6160714,13.3333333 L45.0982143,13.3333333 C45.6727679,13.3333333 46.1428571,13.7903646 46.1428571,14.3489583 L46.1428571,18.75 L50.3214286,18.75 C52.6283482,18.75 54.5,20.5696615 54.5,22.8125 Z M50.3214286,52.0963542 L50.3214286,26.875 L19.6785714,26.875 L19.6785714,52.0963542 C19.6785714,52.375651 19.9136161,52.6041667 20.2008929,52.6041667 L49.7991071,52.6041667 C50.0863839,52.6041667 50.3214286,52.375651 50.3214286,52.0963542 Z"/>
    </defs>
    <g fill="none" fill-rule="evenodd">
        <mask id="icon-b" fill="#fff">
            <use xlink:href="#icon-a"/>
        </mask>
        <g mask="url(#icon-b)">
            <rect width="70" height="70" fill="url(#icon-c)"/>
            <path fill="#FFF" fill-opacity=".383" d="M4,1.8 L65,1.8 C67.6666667,1.8 69.3333333,1.13333333 70,-0.2 C70,2.46666667 70,3.46666667 70,2.8 L1.10547097e-14,2.8 C-1.65952376e-14,3.46666667 -2.9161925e-14,2.46666667 -2.66453526e-14,-0.2 C0.666666667,1.13333333 2,1.8 4,1.8 Z" transform="matrix(1 0 0 -1 0 2.8)"/>
            <path fill="#393939" d="M43.4796282,58 L3.97692274,58 C1.98846137,58 -7.06443391e-15,57.8520408 0,53.8571429 L2.26109583e-16,28.7421381 L18.1939643,6.70980496 L20,6 L24,0 L29.235775,0.958543565 L30.1559259,9.58543565 L42.1178875,8.51357709e-15 L45.7984911,0 L45.7984911,8 L54,8 L53.7495748,40.7451155 L43.4796282,58 Z" opacity=".324" transform="translate(0 12)"/>
            <path fill="#000" fill-opacity=".383" d="M4,4 L65,4 C67.6666667,4 69.3333333,3 70,1 C70,3.66666667 70,5 70,5 L1.77635684e-15,5 C1.77635684e-15,5 1.77635684e-15,3.66666667 1.77635684e-15,1 C0.666666667,3 2,4 4,4 Z" transform="translate(0 65)"/>
            <use fill="#000" fill-rule="nonzero" opacity=".35" xlink:href="#icon-d"/>
            <path fill="#FFF" fill-rule="nonzero" d="M28.3839286,35.7083333 L24.9017857,35.7083333 C24.3272321,35.7083333 23.8571429,35.2513021 23.8571429,34.6927083 L23.8571429,31.3072917 C23.8571429,30.7486979 24.3272321,30.2916667 24.9017857,30.2916667 L28.3839286,30.2916667 C28.9584821,30.2916667 29.4285714,30.7486979 29.4285714,31.3072917 L29.4285714,34.6927083 C29.4285714,35.2513021 28.9584821,35.7083333 28.3839286,35.7083333 Z M37.7857143,34.6927083 L37.7857143,31.3072917 C37.7857143,30.7486979 37.315625,30.2916667 36.7410714,30.2916667 L33.2589286,30.2916667 C32.684375,30.2916667 32.2142857,30.7486979 32.2142857,31.3072917 L32.2142857,34.6927083 C32.2142857,35.2513021 32.684375,35.7083333 33.2589286,35.7083333 L36.7410714,35.7083333 C37.315625,35.7083333 37.7857143,35.2513021 37.7857143,34.6927083 Z M46.1428571,34.6927083 L46.1428571,31.3072917 C46.1428571,30.7486979 45.6727679,30.2916667 45.0982143,30.2916667 L41.6160714,30.2916667 C41.0415179,30.2916667 40.5714286,30.7486979 40.5714286,31.3072917 L40.5714286,34.6927083 C40.5714286,35.2513021 41.0415179,35.7083333 41.6160714,35.7083333 L45.0982143,35.7083333 C45.6727679,35.7083333 46.1428571,35.2513021 46.1428571,34.6927083 Z M37.7857143,42.8177083 L37.7857143,39.4322917 C37.7857143,38.8736979 37.315625,38.4166667 36.7410714,38.4166667 L33.2589286,38.4166667 C32.684375,38.4166667 32.2142857,38.8736979 32.2142857,39.4322917 L32.2142857,42.8177083 C32.2142857,43.3763021 32.684375,43.8333333 33.2589286,43.8333333 L36.7410714,43.8333333 C37.315625,43.8333333 37.7857143,43.3763021 37.7857143,42.8177083 Z M29.4285714,42.8177083 L29.4285714,39.4322917 C29.4285714,38.8736979 28.9584821,38.4166667 28.3839286,38.4166667 L24.9017857,38.4166667 C24.3272321,38.4166667 23.8571429,38.8736979 23.8571429,39.4322917 L23.8571429,42.8177083 C23.8571429,43.3763021 24.3272321,43.8333333 24.9017857,43.8333333 L28.3839286,43.8333333 C28.9584821,43.8333333 29.4285714,43.3763021 29.4285714,42.8177083 Z M46.1428571,42.8177083 L46.1428571,39.4322917 C46.1428571,38.8736979 45.6727679,38.4166667 45.0982143,38.4166667 L41.6160714,38.4166667 C41.0415179,38.4166667 40.5714286,38.8736979 40.5714286,39.4322917 L40.5714286,42.8177083 C40.5714286,43.3763021 41.0415179,43.8333333 41.6160714,43.8333333 L45.0982143,43.8333333 C45.6727679,43.8333333 46.1428571,43.3763021 46.1428571,42.8177083 Z M54.5,20.8125 L54.5,50.6041667 C54.5,52.8470052 52.6283482,54.6666667 50.3214286,54.6666667 L19.6785714,54.6666667 C17.3716518,54.6666667 15.5,52.8470052 15.5,50.6041667 L15.5,20.8125 C15.5,18.5696615 17.3716518,16.75 19.6785714,16.75 L23.8571429,16.75 L23.8571429,12.3489583 C23.8571429,11.7903646 24.3272321,11.3333333 24.9017857,11.3333333 L28.3839286,11.3333333 C28.9584821,11.3333333 29.4285714,11.7903646 29.4285714,12.3489583 L29.4285714,16.75 L40.5714286,16.75 L40.5714286,12.3489583 C40.5714286,11.7903646 41.0415179,11.3333333 41.6160714,11.3333333 L45.0982143,11.3333333 C45.6727679,11.3333333 46.1428571,11.7903646 46.1428571,12.3489583 L46.1428571,16.75 L50.3214286,16.75 C52.6283482,16.75 54.5,18.5696615 54.5,20.8125 Z M50.3214286,50.0963542 L50.3214286,24.875 L19.6785714,24.875 L19.6785714,50.0963542 C19.6785714,50.375651 19.9136161,50.6041667 20.2008929,50.6041667 L49.7991071,50.6041667 C50.0863839,50.6041667 50.3214286,50.375651 50.3214286,50.0963542 Z"/>
        </g>
    </g>
</svg>

```

## File: static\src\js\base_calendar.js

```javascript
odoo.define('base_calendar.base_calendar', function (require) {
"use strict";

var BasicModel = require('web.BasicModel');
var fieldRegistry = require('web.field_registry');
var Notification = require('web.Notification');
var relationalFields = require('web.relational_fields');
var session = require('web.session');
var WebClient = require('web.WebClient');

var FieldMany2ManyTags = relationalFields.FieldMany2ManyTags;


var CalendarNotification = Notification.extend({
    template: "CalendarNotification",
    xmlDependencies: (Notification.prototype.xmlDependencies || [])
        .concat(['/calendar/static/src/xml/notification_calendar.xml']),

    init: function(parent, params) {
        this._super(parent, params);
        this.eid = params.eventID;
        this.sticky = true;

        this.events = _.extend(this.events || {}, {
            'click .link2event': function() {
                var self = this;

                this._rpc({
                        route: '/web/action/load',
                        params: {
                            action_id: 'calendar.action_calendar_event_notify',
                        },
                    })
                    .then(function(r) {
                        r.res_id = self.eid;
                        return self.do_action(r);
                    });
            },

            'click .link2recall': function() {
                this.close();
            },

            'click .link2showed': function() {
                this._rpc({route: '/calendar/notify_ack'})
                    .then(this.close.bind(this, false), this.close.bind(this, false));
            },
        });
    },
});

WebClient.include({
    display_calendar_notif: function(notifications) {
        var self = this;
        var last_notif_timer = 0;

        // Clear previously set timeouts and destroy currently displayed calendar notifications
        clearTimeout(this.get_next_calendar_notif_timeout);
        _.each(this.calendar_notif_timeouts, clearTimeout);
        this.calendar_notif_timeouts = {};

        // For each notification, set a timeout to display it
        _.each(notifications, function(notif) {
            var key = notif.event_id + ',' + notif.alarm_id;
            if (key in self.calendar_notif) {
                return;
            }
            self.calendar_notif_timeouts[key] = setTimeout(function () {
                var notificationID = self.call('notification', 'notify', {
                    Notification: CalendarNotification,
                    title: notif.title,
                    message: notif.message,
                    eventID: notif.event_id,
                    onClose: function () {
                        delete self.calendar_notif[key];
                    },
                });
                self.calendar_notif[key] = notificationID;
            }, notif.timer * 1000);
            last_notif_timer = Math.max(last_notif_timer, notif.timer);
        });

        // Set a timeout to get the next notifications when the last one has been displayed
        if (last_notif_timer > 0) {
            this.get_next_calendar_notif_timeout = setTimeout(this.get_next_calendar_notif.bind(this), last_notif_timer * 1000);
        }
    },
    get_next_calendar_notif: function() {
        session.rpc("/calendar/notify", {}, {shadow: true})
            .then(this.display_calendar_notif.bind(this))
            .guardedCatch(function(reason) { //
                var err = reason.message;
                var ev = reason.event;
                if(err.code === -32098) {
                    // Prevent the CrashManager to display an error
                    // in case of an xhr error not due to a server error
                    ev.preventDefault();
                }
            });
    },
    show_application: function() {
        // An event is triggered on the bus each time a calendar event with alarm
        // in which the current user is involved is created, edited or deleted
        this.calendar_notif_timeouts = {};
        this.calendar_notif = {};
        this.call('bus_service', 'onNotification', this, function (notifications) {
            _.each(notifications, (function (notification) {
                if (notification[0][1] === 'calendar.alarm') {
                    this.display_calendar_notif(notification[1]);
                }
            }).bind(this));
        });
        return this._super.apply(this, arguments).then(this.get_next_calendar_notif.bind(this));
    },
});

BasicModel.include({
    /**
     * @private
     * @param {Object} record
     * @param {string} fieldName
     * @returns {Promise}
     */
    _fetchSpecialAttendeeStatus: function (record, fieldName) {
        var context = record.getContext({fieldName: fieldName});
        var attendeeIDs = record.data[fieldName] ? this.localData[record.data[fieldName]].res_ids : [];
        var meetingID = _.isNumber(record.res_id) ? record.res_id : false;
        return this._rpc({
            model: 'res.partner',
            method: 'get_attendee_detail',
            args: [attendeeIDs, meetingID],
            context: context,
        }).then(function (result) {
            return _.map(result, function (d) {
                return _.object(['id', 'display_name', 'status', 'color'], d);
            });
        });
    },
});

var Many2ManyAttendee = FieldMany2ManyTags.extend({
    // as this widget is model dependant (rpc on res.partner), use it in another
    // context probably won't work
    // supportedFieldTypes: ['many2many'],
    tag_template: "Many2ManyAttendeeTag",
    specialData: "_fetchSpecialAttendeeStatus",

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     * @private
     */
    _getRenderTagsContext: function () {
        var result = this._super.apply(this, arguments);
        result.attendeesData = this.record.specialData.partner_ids;
        return result;
    },
});

fieldRegistry.add('many2manyattendee', Many2ManyAttendee);

});

```

## File: static\src\js\calendar_view.js

```javascript
odoo.define('calendar.CalendarView', function (require) {
"use strict";

var CalendarPopover = require('web.CalendarPopover');
var CalendarRenderer = require('web.CalendarRenderer');
var CalendarView = require('web.CalendarView');
var viewRegistry = require('web.view_registry');

var AttendeeCalendarPopover = CalendarPopover.extend({
    template: 'Calendar.attendee.status.popover',
    events: _.extend({}, CalendarPopover.prototype.events, {
        'click .o-calendar-attendee-status .dropdown-item': '_onClickAttendeeStatus'
    }),
    /**
     * @constructor
     */
    init: function () {
        var self = this;
        this._super.apply(this, arguments);
        var session = this.getSession();
        // Show status dropdown if user is in attendees list
        this.showStatusDropdown = _.contains(this.event.record.partner_ids, session.partner_id);
        if (this.showStatusDropdown) {
            this.statusColors = {accepted: 'text-success', declined: 'text-danger', tentative: 'text-muted', needsAction: 'text-dark'};
            this.statusInfo = {};
            _.each(this.fields.attendee_status.selection, function (selection) {
                self.statusInfo[selection[0]] = {text: selection[1], color: self.statusColors[selection[0]]};
            });
            this.selectedStatusInfo = this.statusInfo[this.event.record.attendee_status];
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickAttendeeStatus: function (ev) {
        ev.preventDefault();
        var self = this;
        var selectedStatus = $(ev.currentTarget).attr('data-action');
        this._rpc({
            model: 'calendar.event',
            method: 'change_attendee_status',
            args: [this.event.id, selectedStatus],
        }).then(function () {
            self.event.record.attendee_status = selectedStatus;  // FIXEME: Maybe we have to reload view
            self.$('.o-calendar-attendee-status-text').text(self.statusInfo[selectedStatus].text);
            self.$('.o-calendar-attendee-status-icon').removeClass(_.values(self.statusColors).join(' ')).addClass(self.statusInfo[selectedStatus].color);
        });
    },
});

var AttendeeCalendarRenderer = CalendarRenderer.extend({
    config: _.extend({}, CalendarRenderer.prototype.config, {
        CalendarPopover: AttendeeCalendarPopover,
    }),
});


var AttendeeCalendarView = CalendarView.extend({
    config: _.extend({}, CalendarView.prototype.config, {
        Renderer: AttendeeCalendarRenderer,
    }),
});

viewRegistry.add('attendee_calendar', AttendeeCalendarView);

});

```

## File: static\src\js\mail_activity.js

```javascript
odoo.define('calendar.Activity', function (require) {
"use strict";

var Activity = require('mail.Activity');
var Dialog = require('web.Dialog');
var core = require('web.core');
var _t = core._t;

Activity.include({

    /**
     * Override behavior to redirect to calendar event instead of activity
     *
     * @override
     */
    _onEditActivity: function (event, options) {
        var self = this;
        var activity_id = $(event.currentTarget).data('activity-id');
        var activity = _.find(this.activities, function (act) { return act.id === activity_id; });
        if (activity && activity.activity_category === 'meeting' && activity.calendar_event_id) {
            return self._super(event, _.extend({
                res_model: 'calendar.event',
                res_id: activity.calendar_event_id[0],
            }));
        }
        return self._super(event, options);
    },

    /**
     * Override behavior to warn that the calendar event is about to be removed as well
     *
     * @override
     */
    _onUnlinkActivity: function (event, options) {
        event.preventDefault();
        var self = this;
        var activity_id = $(event.currentTarget).data('activity-id');
        var activity = _.find(this.activities, function (act) { return act.id === activity_id; });
        if (activity && activity.activity_category === 'meeting' && activity.calendar_event_id) {
            Dialog.confirm(
                self,
                _t("The activity is linked to a meeting. Deleting it will remove the meeting as well. Do you want to proceed ?"), {
                    confirm_callback: function () {
                        return self._rpc({
                            model: 'mail.activity',
                            method: 'unlink_w_meeting',
                            args: [[activity_id]],
                        })
                        .then(self._reload.bind(self, {activity: true}));
                    },
                }
            );
        }
        else {
            return self._super(event, options);
        }
    },
});

});

```

## File: static\src\js\systray_activity_menu.js

```javascript
odoo.define('calendar.systray.ActivityMenu', function (require) {
"use strict";

var ActivityMenu = require('mail.systray.ActivityMenu');
var fieldUtils = require('web.field_utils');

ActivityMenu.include({

    //-----------------------------------------
    // Private
    //-----------------------------------------

    /**
     * parse date to server value
     *
     * @private
     * @override
     */
    _getActivityData: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            var meeting = _.find(self._activities, {type: 'meeting'});
            if (meeting && meeting.meetings)  {
                _.each(meeting.meetings, function (res) {
                    res.start = fieldUtils.parse.datetime(res.start, false, {isUTC: true});
                });
            }
        });
    },

    //-----------------------------------------
    // Handlers
    //-----------------------------------------

    /**
     * @private
     * @override
     */
    _onActivityFilterClick: function (ev) {
        var $el = $(ev.currentTarget);
        var data = _.extend({}, $el.data());
        if (data.res_model === "calendar.event" && data.filter === "my") {
            this.do_action('calendar.action_calendar_event', {
                additional_context: {
                    default_mode: 'day',
                    search_default_mymeetings: 1,
                },
               clear_breadcrumbs: true,
            });
        } else {
            this._super.apply(this, arguments);
        }
    },
});

});

```

## File: static\src\xml\base_calendar.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<template>
    <t t-name="Many2ManyAttendeeTag" t-extend="FieldMany2ManyTag">
        <t t-jquery="span:first" t-operation="prepend">
            <span t-if="attendeesData[el_index]" t-attf-class="o_calendar_invitation #{attendeesData[el_index].status}"/>
        </t>
    </t>

    <t t-extend="mail.systray.ActivityMenu.Previews">
        <t t-jquery="div.o_preview_title" t-operation="after">
            <div t-if="activity and activity.type == 'meeting'">
                <t t-set="is_next_meeting" t-value="true"/>
                <t t-foreach="activity.meetings" t-as="meeting">
                    <div>
                        <span t-att-class="!meeting.allday and is_next_meeting ? 'o_meeting_filter o_meeting_bold' : 'o_meeting_filter'" t-att-data-res_model="activity.model" t-att-data-res_id="meeting.id" t-att-data-model_name="activity.name" t-att-title="meeting.name">
                            <span><t t-esc="meeting.name"/></span>
                        </span>
                        <span t-if="meeting.start" class="float-right">
                            <t t-if="meeting.allday">All Day</t>
                            <t t-else=''>
                                <t t-set="is_next_meeting" t-value="false"/>
                                <t t-esc="moment(meeting.start).local().format('hh:mm A')"/>
                            </t>
                        </span>
                    </div>
                </t>
            </div>
        </t>
    </t>

    <t t-name="Calendar.attendee.status.popover" t-extend="CalendarView.event.popover">
        <t t-jquery=".o_cw_popover_delete" t-operation="after">
            <div t-if="widget.showStatusDropdown" class="btn-group o-calendar-attendee-status ml-2">
                <a href="#" class="btn btn-secondary dropdown-toggle" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                    <i t-attf-class="fa fa-circle o-calendar-attendee-status-icon #{widget.selectedStatusInfo.color}"/> <span class="o-calendar-attendee-status-text" t-esc="widget.selectedStatusInfo.text"></span>
                </a>
                <div class="dropdown-menu overflow-hidden">
                    <a class="dropdown-item" href="#" data-action="accepted"><i class="fa fa-circle text-success"/> Accept</a>
                    <a class="dropdown-item" href="#" data-action="declined"><i class="fa fa-circle text-danger"/> Decline</a>
                    <a class="dropdown-item" href="#" data-action="tentative"><i class="fa fa-circle text-muted"/> Uncertain</a>
                </div>
            </div>
        </t>
    </t>

</template>

```

## File: static\src\xml\notification_calendar.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<template>

<t t-name="CalendarNotification" t-extend="Notification">
    <t t-jquery=".o_notification_title > t" t-operation="replace">
        <span  t-attf-class="link2event eid_{{widget.eid}}">
            <t t-esc="widget.title"/>
        </span>
    </t>
    <t t-jquery=".o_notification_content" t-operation="after">
        <div class="mt-2">
            <button type="button" class="btn btn-primary link2showed oe_highlight oe_form oe_button"><span>OK</span></button>
            <button type="button" class="btn btn-link link2event">Details</button>
            <button type="button" class="btn btn-link link2recall">Snooze</button>
        </div>
    </t>
</t>

</template>

```

## File: views\calendar_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="calendar assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/calendar/static/src/scss/calendar.scss"/>
            <script type="text/javascript" src="/calendar/static/src/js/base_calendar.js"></script>
            <script type="text/javascript" src="/calendar/static/src/js/calendar_view.js"></script>
            <script type="text/javascript" src="/calendar/static/src/js/mail_activity.js"></script>
            <script type="text/javascript" src="/calendar/static/src/js/systray_activity_menu.js"></script>
        </xpath>
    </template>

    <template id="qunit_suite" inherit_id="web.qunit_suite">
        <xpath expr="//script[last()]" position="after">
            <script type="text/javascript" src="/calendar/static/tests/calendar_tests.js"/>
            <script type="text/javascript" src="/calendar/static/tests/systray_activity_menu_tests.js"/>
        </xpath>
    </template>

    <!-- Template rendered in route auth=None, for anonymous user. This allow them to see meeting details -->
    <template id="invitation_page_anonymous" name="Calendar Invitation Page for anonymous users">
        <t t-call="web.layout">
            <t t-set="head">
                <t t-call-assets="web.assets_common" t-js="false"/>
                <t t-call-assets="web.assets_frontend" t-js="false"/>
            </t>

            <div class="container">
                <div class="o_logo">
                    <img class="img img-fluid d-block mx-auto" src="/web/binary/company_logo" alt="Logo"/>
                </div>

                <div class="card">
                    <div class="card-header">
                        <h2>Calendar Invitation <small><t t-esc="event.name"/></small></h2>
                    </div>
                    <div class="card-body">
                        <div class="clearfix mb16" t-if="attendee.state != 'needsAction'">
                            <span class="float-right badge badge-info">
                                <t t-if="attendee.state == 'accepted'">Yes I'm going.</t>
                                <t t-if="attendee.state == 'declined'">No I'm not going.</t>
                            </span>
                        </div>

                        <div class="table-responsive">
                            <table class="o_event_table table table-striped">
                                <tr>
                                    <th>Invitation for</th>
                                    <td><t t-esc="attendee.common_name"/> (<t t-esc="attendee.email"/>)</td>
                                </tr>
                                <tr>
                                    <th>Date</th>
                                    <td><t t-esc="event.display_time"/></td>
                                </tr>
                                <tr>
                                    <th>Location</th>
                                    <td><t t-esc="event.location or '-'"/></td>
                                </tr>
                                <tr>
                                    <th>Attendees</th>
                                    <td>
                                        <ul>
                                            <li t-foreach="event.attendee_ids" t-as="attendee" t-attf-class="o_#{attendee.state}">
                                                <t t-esc="attendee.common_name"/>
                                            </li>
                                        </ul>
                                    </td>
                                </tr>
                            </table>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>
</odoo>

```

## File: views\calendar_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <!-- Calendar Events Types : Views and Actions -->
    <record id="view_calendar_event_type_tree" model="ir.ui.view">
        <field name="name">calendar.event.type</field>
        <field name="model">calendar.event.type</field>
        <field name="arch" type="xml">
            <tree string="Meeting Types" editable="bottom">
                <field name="name"/>
            </tree>
        </field>
    </record>

    <record id="action_calendar_event_type" model="ir.actions.act_window">
        <field name="name">Meeting Types</field>
        <field name="res_model">calendar.event.type</field>
        <field name="view_id" ref="view_calendar_event_type_tree"/>
    </record>

    <!-- Calendar Alarm : -->
    <record id="view_calendar_alarm_tree" model="ir.ui.view">
        <field name="name">calendar.alarm.tree</field>
        <field name="model">calendar.alarm</field>
        <field name="arch" type="xml">
            <tree string="Calendar Alarm" editable="bottom">
                <field name="name" invisible="1"/>
                <field name="alarm_type"/>
                <field name="duration"/>
                <field name="interval"/>
            </tree>
        </field>
    </record>

    <record id="calendar_alarm_view_form" model="ir.ui.view">
            <field name="name">calendar.alarm.form</field>
            <field name="model">calendar.alarm</field>
            <field name="arch" type="xml">
                <form string="Calendar Alarm">
                 <group>
                    <group>
                        <field name="name" invisible="1"/>
                        <field name="alarm_type"/>
                    </group>
                    <group>
                        <label for="duration"/>
                        <div class="o_row">
                            <field name="duration"/>
                            <field name="interval"/>
                        </div>
                    </group>
                </group>
            </form>
        </field>
    </record>

    <record id="action_calendar_alarm" model="ir.actions.act_window">
        <field name="name">Calendar Alarm</field>
        <field name="res_model">calendar.alarm</field>
        <field name="view_id" ref="view_calendar_alarm_tree"/>
    </record>

    <!-- Calendar Events : Views and Actions  -->
    <record id="view_calendar_event_tree" model="ir.ui.view">
        <field name="name">calendar.event.tree</field>
        <field name="model">calendar.event</field>
        <field name="arch" type="xml">
            <tree string="Meetings" decoration-bf="message_needaction==True">
                <field name="name" string="Subject"/>
                <field name="allday" invisible="True"/>
                <field name="start" string="Start Date"/>
                <field name="stop" string="End Date"/>
                <field name="partner_ids"/>
                <field name="location"/>
                <field name="state" invisible="True"/>
                <field name="duration" widget="float_time"/>
                <field name="message_needaction" invisible="1"/>
            </tree>
        </field>
    </record>

    <record id="view_calendar_event_form" model="ir.ui.view">
        <field name="name">calendar.event.form</field>
        <field name="model">calendar.event</field>
        <field name="priority" eval="1"/>
        <field name="arch" type="xml">
            <form string="Meetings">
                <sheet>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <button string="Document" class="oe_stat_button float-right" icon="fa-bars" type="object" name="action_open_calendar_event" attrs="{'invisible': ['|', ('res_model', '=', False), ('res_id', '=', False)]}"/>
                    <field name="res_model" invisible="1" />
                    <field name="res_id" invisible="1" />
                    <field name="state" invisible="1"/>
                    <field name="is_attendee" invisible="1"/>
                    <field name="attendee_status" invisible="1"/>
                    <field name="active" invisible="1"/>
                    <div class="oe_title">
                        <div class="oe_edit_only">
                            <label for="name"/>
                        </div>
                        <h1>
                            <field name="name" placeholder="e.g. Business Lunch"/>
                        </h1>
                        <label for="partner_ids" string="Attendees" class="oe_edit_only"/>
                        <h2>
                            <field name="partner_ids" widget="many2manyattendee"
                                placeholder="Select attendees..."
                                context="{'force_email':True}"
                                class="oe_inline"/>
                        </h2>
                    </div>
                    <notebook>
                        <page name="page_details" string="Meeting Details">
                            <group attrs="{'invisible': [('recurrency','==',False)]}" class="oe_edit_only ">
                                <p class='alert alert-warning' role="status"> This event is linked to a recurrence...<br/>
                                    <button type="object" name="action_detach_recurring_event"  string="Update only this instance"  help="Click here to update only this instance and not all recurrences. " class="oe_link"/>
                                </p>
                            </group>
                            <group>
                                <group>
                                    <field name="start" attrs="{'invisible': True}"/>
                                    <field name="stop" attrs="{'invisible': True}"/>
                                    <field name="id" attrs="{'invisible': True}"/>

                                    <field name="start_date" string="Starting at" attrs="{'required': [('allday','=',True)], 'invisible': [('allday','=',False)], 'readonly': [('id', '!=', False), ('recurrency','=',True)]}" force_save="1"/>
                                    <field name="stop_date" string="Ending at" attrs="{'required': [('allday','=',True)],'invisible': [('allday','=',False)], 'readonly': [('id', '!=', False), ('recurrency','=',True)]}" force_save="1"/>

                                    <field name="start_datetime" string="Starting at" attrs="{'required': [('allday','=',False)], 'invisible': [('allday','=',True)], 'readonly': [('id', '!=', False), ('recurrency','=',True)]}"/>
                                    <field name="stop_datetime" invisible="1"/>
                                    <label for="duration" attrs="{'invisible': [('allday','=',True)]}"/>
                                    <div attrs="{'invisible': [('allday','=',True)]}">
                                        <field name="duration" widget="float_time" string="Duration" class="oe_inline" attrs="{'readonly': [('id', '!=', False), ('recurrency','=',True)]}"/>
                                        <span> hours</span>
                                    </div>
                                    <field name="allday" attrs="{'readonly': [('id', '!=', False), ('recurrency','=',True)]}" force_save="1"/>
                                </group>
                                <group>
                                    <field name="categ_ids" widget="many2many_tags" options="
                                    {'no_create_edit': True}"/>
                                    <field name="alarm_ids" widget="many2many_tags" />
                                    <field name="location" />
                                    <field name="event_tz" attrs="{'invisible': [('recurrency', '=', False)]}"/>
                                </group>

                            </group>
                            <label for="description"/>
                            <field name="description"/>
                        </page>
                        <page name="page_options" string="Options">
                            <group>
                                <div>
                                    <group>
                                        <field name="recurrency"/>
                                    </group>
                                    <div attrs="{'invisible': [('recurrency', '=', False)]}">
                                        <group>
                                            <label for="interval"/>
                                            <div class="o_row">
                                                <field name="interval" attrs="{'required': [('recurrency', '=', True)]}"/>
                                                <field name="rrule_type" attrs="{'required': [('recurrency', '=', True)]}"/>
                                            </div>
                                            <label string="Until" for="end_type"/>
                                            <div class="o_row">
                                                <field name="end_type" attrs="{'required': [('recurrency', '=', True)]}"/>
                                                <field name="count" attrs="{'invisible': [('end_type', '!=', 'count')], 'required': [('recurrency', '=', True)]}"/>
                                                <field name="final_date" attrs="{'invisible': [('end_type', '!=', 'end_date')], 'required': [('end_type', '=', 'end_date'), ('recurrency', '=', True)]}"/>
                                            </div>
                                        </group>
                                        <group attrs="{'invisible': [('rrule_type', '!=', 'weekly')]}" name="weekdays">
                                            <field name="mo"/>
                                            <field name="tu"/>
                                            <field name="we"/>
                                            <field name="th"/>
                                            <field name="fr"/>
                                            <field name="sa"/>
                                            <field name="su"/>
                                        </group>
                                        <group attrs="{'invisible': [('rrule_type', '!=', 'monthly')]}">
                                            <label string="Day of Month" for="month_by"/>
                                            <div class="o_row">
                                                <field name="month_by"/>
                                                <field name="day"
                                                    attrs="{'required': [('month_by', '=', 'date'), ('rrule_type', '=', 'monthly')],
                                                            'invisible': [('month_by', '!=', 'date')]}"/>
                                                <field name="byday" string="The"
                                                    attrs="{'required': [('recurrency', '=', True), ('month_by', '=', 'day'), ('rrule_type', '=', 'monthly')],
                                                            'invisible': [('month_by', '!=', 'day')]}"/>
                                                <field name="week_list" nolabel="1"
                                                    attrs="{'required': [('recurrency', '=', True), ('month_by', '=', 'day'), ('rrule_type', '=', 'monthly')],
                                                            'invisible': [('month_by', '!=', 'day')]}"/>
                                            </div>
                                        </group>
                                    </div>
                                </div>
                                <group>
                                    <field name="privacy"/>
                                    <field name="show_as"/>
                                    <field name="recurrent_id" invisible="1" />
                                </group>
                            </group>
                        </page>

                        <page name="page_invitations" string="Invitations" groups="base.group_no_one">
                            <button name="action_sendmail" type="object" string="Send mail" icon="fa-envelope" class="oe_link"/>
                            <field name="attendee_ids" widget="one2many" mode="tree,kanban">
                                <tree string="Invitation details" editable="top" create="false" delete="false">
                                    <field name="partner_id" />
                                    <field name="state" />
                                    <field name="email" widget="email"/>

                                    <button name="do_tentative" states="needsAction,declined,accepted" string="Uncertain" type="object" icon="fa-asterisk" />
                                    <button name="do_accept" string="Accept" states="needsAction,tentative,declined" type="object" icon="fa-check text-success"/>
                                    <button name="do_decline" string="Decline" states="needsAction,tentative,accepted" type="object" icon="fa-times-circle text-danger"/>
                                </tree>
                                <kanban class="o_kanban_mobile" create="false" delete="false">
                                    <field name="partner_id" />
                                    <field name="state" />
                                    <field name="email" widget="email"/>

                                    <templates>
                                        <t t-name="kanban-box">
                                            <div class="d-flex flex-column justify-content-between">
                                                <field name="partner_id"/>
                                                <field name="email" widget="email"/>
                                                <span>Status: <field name="state" /></span>

                                                <div class="text-right">
                                                    <button name="do_tentative" states="needsAction,declined,accepted" string="Uncertain" type="object" class="btn fa fa-asterisk"/>
                                                    <button name="do_accept" states="needsAction,tentative,declined" string="Accept" type="object" class="btn fa fa-check text-success"/>
                                                    <button name="do_decline" states="needsAction,tentative,accepted" string="Decline" type="object" class="btn fa fa-times-circle text-danger"/>
                                                </div>
                                            </div>
                                        </t>
                                    </templates>
                                </kanban>
                            </field>
                        </page>
                        <page name="page_misc" string="Misc" groups="base.group_no_one">
                            <group>
                                <label for="user_id" string="Owner"/>
                                <field name="user_id" nolabel="1"/>
                            </group>
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids" widget="mail_followers"/>
                    <field name="message_ids" widget="mail_thread" />
                </div>
            </form>
        </field>
    </record>

    <record id="view_calendar_event_calendar" model="ir.ui.view">
        <field name="name">calendar.event.calendar</field>
        <field name="model">calendar.event</field>
        <field name="priority" eval="2"/>
        <field name="arch" type="xml">
            <calendar js_class="attendee_calendar" string="Meetings" date_start="start" date_stop="stop" date_delay="duration" all_day="allday"
                event_open_popup="true"
                event_limit="5"
                color="partner_id">
                <field name="attendee_status"/>
                <field name="partner_id"/>
                <field name="partner_ids" widget="many2many_tags_avatar" write_model="calendar.contacts" write_field="partner_id" avatar_field="image_128"/>
                <field name="is_highlighted" invisible="1"/>
                <field name="description"/>
            </calendar>
        </field>
    </record>

    <record id="view_calendar_event_search" model="ir.ui.view">
        <field name="name">calendar.event.search</field>
        <field name="model">calendar.event</field>
        <field name="arch" type="xml">
            <search string="Search Meetings">
                <field name="name" string="Meeting" filter_domain="[('name', 'ilike', self)]"/>
                <field name="partner_ids"/>
                <field name="categ_ids"/>
                <field name="user_id"/>
                <field name="show_as"/>
                <field name="privacy" string="Privacy"/>
                <filter string="My Meetings" help="My Meetings" name="mymeetings" context='{"mymeetings": 1}'/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                <group expand="0" string="Group By">
                    <filter string="Responsible" name="responsible" domain="[]" context="{'group_by': 'user_id'}"/>
                    <filter string="Availability" name="availability" domain="[]" context="{'group_by': 'show_as'}"/>
                    <filter string="Privacy" name="privacy" domain="[]" context="{'group_by': 'privacy'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="action_calendar_event" model="ir.actions.act_window">
        <field name="name">Meetings</field>
        <field name="res_model">calendar.event</field>
        <field name="view_mode">calendar,tree,form</field>
        <field name="view_id" ref="view_calendar_event_calendar"/>
        <field name="search_view_id" ref="view_calendar_event_search"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Schedule a new meeting
          </p><p>
            The calendar is shared between employees and fully integrated with
            other applications such as the employee leaves or the business
            opportunities.
          </p>
        </field>
    </record>

    <record id="action_view_calendar_event_calendar" model="ir.actions.act_window.view">
        <field name="act_window_id" ref="action_calendar_event"/>
        <field name="sequence" eval="1"/>
        <field name="view_mode">calendar</field>
        <field name="view_id" ref="view_calendar_event_calendar"/>
    </record>

    <record id="action_view_calendar_event_tree" model="ir.actions.act_window.view">
        <field name="act_window_id" ref="action_calendar_event"/>
        <field name="sequence" eval="2"/>
        <field name="view_mode">tree</field>
        <field name="view_id" ref="view_calendar_event_tree"/>
    </record>

    <record id="action_view_calendar_event_form" model="ir.actions.act_window.view">
        <field name="act_window_id" ref="action_calendar_event"/>
        <field name="sequence" eval="3"/>
        <field name="view_mode">form</field>
        <field name="view_id" ref="view_calendar_event_form"/>
    </record>

    <!-- Menus -->
    <menuitem
        id="mail_menu_calendar"
        name="Calendar"
        sequence="2"
        action="action_calendar_event"
        web_icon="calendar,static/description/icon.png"
        groups="base.group_user"/>

    <menuitem
        id="menu_calendar_configuration"
        name="Calendar"
        parent="base.menu_custom"
        groups="base.group_no_one"/>

    <menuitem
        id="menu_calendar_event_type"
        parent="menu_calendar_configuration"
        action="action_calendar_event_type"
        groups="base.group_no_one"/>

    <menuitem
        id="menu_calendar_alarm"
        parent="menu_calendar_configuration"
        action="action_calendar_alarm"
        groups="base.group_no_one"/>

    <!-- called in js from '/js/base_calendar.js' -->
    <record id="action_calendar_event_notify" model="ir.actions.act_window">
        <field name="name">Meetings</field>
        <field name="res_model">calendar.event</field>
        <field name="view_mode">form,calendar,tree</field>
        <field name="view_id" ref="view_calendar_event_form"/>
    </record>

</odoo>

```

## File: views\mail_activity_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="mail_activity_view_form_popup" model="ir.ui.view">
        <field name="name">mail.activity.form.inherit.calendar</field>
        <field name="model">mail.activity</field>
        <field name="inherit_id" ref="mail.mail_activity_view_form_popup"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='date_deadline']" position="attributes">
                  <attribute name="attrs">{'invisible': [('activity_category', '=', 'meeting')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='user_id']" position="attributes">
                  <attribute name="attrs">{'invisible': [('activity_category', '=', 'meeting')]}</attribute>
            </xpath>
            <xpath expr="//button[@name='action_close_dialog']" position="attributes">
                  <attribute name="attrs">{'invisible': ['|', ('activity_category', '=', 'meeting'), ('id', '!=', False)]}</attribute>
            </xpath>
            <xpath expr="//button[@name='action_done']" position="attributes">
                  <attribute name="attrs">{'invisible': ['|', ('activity_category', '=', 'meeting'), ('force_next', '=', True)]}</attribute>
            </xpath>
            <xpath expr="//button[@special='cancel']" position="attributes">
                  <attribute name="attrs">{'invisible': [('activity_category', '=', 'meeting')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='note']" position="attributes">
                  <attribute name="attrs">{'invisible': [('activity_category', '=', 'meeting')]}</attribute>
            </xpath>
            <xpath expr="//button[@name='action_close_dialog']" position="before">
                <button string="Open Calendar"
                    attrs="{'invisible': [('activity_category', '!=', 'meeting')]}"
                    name="action_create_calendar_event"
                    type="object"
                    class="btn-primary"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: wizard\mail_invite.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models

from odoo.addons.calendar.models.calendar import get_real_ids


class MailInvite(models.TransientModel):

    _inherit = 'mail.wizard.invite'

    @api.model
    def default_get(self, fields):
        """ In case someone clicked on 'invite others' wizard in the followers widget, transform virtual ids in real ids """
        if 'default_res_id' in self._context:
            self = self.with_context(default_res_id=get_real_ids(self._context['default_res_id']))

        result = super(MailInvite, self).default_get(fields)
        if 'res_id' in result:
            result['res_id'] = get_real_ids(result['res_id'])
        return result

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mail_invite

```

