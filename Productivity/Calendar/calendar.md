# Odoo Module: calendar

Category: Productivity/Calendar

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import populate
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Calendar',
    'version': '1.1',
    'sequence': 165,
    'depends': ['base', 'mail', 'onboarding'],
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
    'category': 'Productivity/Calendar',
    'demo': [
        'data/calendar_demo.xml'
    ],
    'data': [
        'security/ir.model.access.csv',
        'security/calendar_security.xml',
        'data/calendar_cron.xml',
        'data/mail_template_data.xml',
        'data/calendar_data.xml',
        'data/calendar_onboarding_data.xml',
        'data/mail_activity_type_data.xml',
        'data/mail_message_subtype_data.xml',
        'views/mail_activity_views.xml',
        'views/calendar_templates.xml',
        'views/calendar_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
        'wizard/calendar_provider_config.xml',
        'wizard/calendar_popover_delete_wizard.xml',
        'wizard/mail_activity_schedule_views.xml',
    ],
    'installable': True,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'calendar/static/src/**/*',
        ],
        'web.qunit_suite_tests': [
            'calendar/static/tests/**/*',
        ],
        'web.assets_tests': [
            'calendar/static/tests/tours/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import odoo.http as http

from odoo.http import request
from odoo.tools.misc import get_lang


class CalendarController(http.Controller):

    # YTI Note: Keep id and kwargs only for retrocompatibility purpose
    @http.route('/calendar/meeting/accept', type='http', auth="calendar")
    def accept_meeting(self, token, id, **kwargs):
        attendee = request.env['calendar.attendee'].sudo().search([
            ('access_token', '=', token),
            ('state', '!=', 'accepted')])
        attendee.do_accept()
        return self.view_meeting(token, id)

    @http.route('/calendar/recurrence/accept', type='http', auth="calendar")
    def accept_recurrence(self, token, id, **kwargs):
        attendee = request.env['calendar.attendee'].sudo().search([
            ('access_token', '=', token),
            ('state', '!=', 'accepted')])
        if attendee:
            attendees = request.env['calendar.attendee'].sudo().search([
                ('event_id', 'in', attendee.event_id.recurrence_id.calendar_event_ids.ids),
                ('partner_id', '=', attendee.partner_id.id),
                ('state', '!=', 'accepted'),
            ])
            attendees.do_accept()
        return self.view_meeting(token, id)

    @http.route('/calendar/meeting/decline', type='http', auth="calendar")
    def decline_meeting(self, token, id, **kwargs):
        attendee = request.env['calendar.attendee'].sudo().search([
            ('access_token', '=', token),
            ('state', '!=', 'declined')])
        attendee.do_decline()
        return self.view_meeting(token, id)

    @http.route('/calendar/recurrence/decline', type='http', auth="calendar")
    def decline_recurrence(self, token, id, **kwargs):
        attendee = request.env['calendar.attendee'].sudo().search([
            ('access_token', '=', token),
            ('state', '!=', 'declined')])
        if attendee:
            attendees = request.env['calendar.attendee'].sudo().search([
                ('event_id', 'in', attendee.event_id.recurrence_id.calendar_event_ids.ids),
                ('partner_id', '=', attendee.partner_id.id),
                ('state', '!=', 'declined'),
            ])
            attendees.do_decline()
        return self.view_meeting(token, id)

    @http.route('/calendar/meeting/view', type='http', auth="calendar")
    def view_meeting(self, token, id, **kwargs):
        attendee = request.env['calendar.attendee'].sudo().search([
            ('access_token', '=', token),
            ('event_id', '=', int(id))])
        if not attendee:
            return request.not_found()
        timezone = attendee.partner_id.tz
        lang = attendee.partner_id.lang or get_lang(request.env).code
        event = request.env['calendar.event'].with_context(tz=timezone, lang=lang).sudo().browse(int(id))
        company = event.user_id and event.user_id.company_id or event.create_uid.company_id

        # If user is internal and logged, redirect to form view of event
        # otherwise, display the simplifyed web page with event informations
        if request.session.uid and request.env['res.users'].browse(request.session.uid).user_has_groups('base.group_user'):
            return request.redirect('/web?db=%s#id=%s&view_type=form&model=calendar.event' % (request.env.cr.dbname, id))

        # NOTE : we don't use request.render() since:
        # - we need a template rendering which is not lazy, to render before cursor closing
        # - we need to display the template in the language of the user (not possible with
        #   request.render())
        response_content = request.env['ir.ui.view'].with_context(lang=lang)._render_template(
            'calendar.invitation_page_anonymous', {
                'company': company,
                'event': event,
                'attendee': attendee,
            })
        return request.make_response(response_content, headers=[('Content-Type', 'text/html')])

    @http.route('/calendar/meeting/join', type='http', auth="user", website=True)
    def calendar_join_meeting(self, token, **kwargs):
        event = request.env['calendar.event'].sudo().search([
            ('access_token', '=', token)])
        if not event:
            return request.not_found()
        event.action_join_meeting(request.env.user.partner_id.id)
        attendee = request.env['calendar.attendee'].sudo().search([('partner_id', '=', request.env.user.partner_id.id), ('event_id', '=', event.id)])
        return request.redirect('/calendar/meeting/view?token=%s&id=%s' % (attendee.access_token, event.id))

    # Function used, in RPC to check every 5 minutes, if notification to do for an event or not
    @http.route('/calendar/notify', type='json', auth="user")
    def notify(self):
        return request.env['calendar.alarm_manager'].get_next_notif()

    @http.route('/calendar/notify_ack', type='json', auth="user")
    def notify_ack(self):
        return request.env['res.partner'].sudo()._set_calendar_last_notif_ack()

    @http.route('/calendar/join_videocall/<string:access_token>', type='http', auth='public')
    def calendar_join_videocall(self, access_token):
        event = request.env['calendar.event'].sudo().search([('access_token', '=', access_token)])
        if not event:
            return request.not_found()

        # if channel doesn't exist
        if not event.videocall_channel_id:
            event._create_videocall_channel()

        return request.redirect(event.videocall_channel_id.invitation_url)

    @http.route('/calendar/check_credentials', type='json', auth='user')
    def check_calendar_credentials(self):
        # method should be overwritten by sync providers
        return request.env['res.users'].check_calendar_credentials()

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

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
            <field name="code">model._send_reminder()</field>
            <field eval="True" name="active" />
            <field name="user_id" ref="base.user_root" />
            <field name="interval_number">1</field>
            <field name="interval_type">days</field>
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
            <field name="mail_template_id" ref="calendar.calendar_template_meeting_reminder"/>
        </record>
        <record id="alarm_mail_2" model="calendar.alarm">
            <field name="name">Email - 6 Hours</field>
            <field name="duration" eval="6" />
            <field name="interval">hours</field>
            <field name="alarm_type">email</field>
            <field name="mail_template_id" ref="calendar.calendar_template_meeting_reminder"/>
        </record>
    </data>
</odoo>

```

## File: data\calendar_demo.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">

        <record id="cal_contact_1" model="calendar.filters">
            <field name="active" eval="True"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="partner_id" ref="base.res_partner_1"/>
        </record>

        <record id="cal_contact_2" model="calendar.filters">
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
        </record>


    </data>
</odoo>

```

## File: data\calendar_onboarding_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- ONBOARDING STEPS -->
        <record id="onboarding_onboarding_step_setup_calendar_integration" model="onboarding.onboarding.step">
            <field name="title">Connect your calendar</field>
            <field name="description">With Outlook or Google</field>
            <field name="button_text">Add</field>
            <field name="done_text">Done!</field>
            <field name="panel_step_open_action_name">action_view_start_calendar_sync</field>
            <field name="step_image" type="base64" file="base/static/img/onboarding_calendar.png"></field>
            <field name="step_image_filename">onboarding_calendar.png</field>
            <field name="step_image_alt">Onboarding Calendar Synchronization</field>
            <field name="sequence">1</field>
        </record>

        <record id="onboarding_onboarding_calendar" model="onboarding.onboarding">
            <field name="name">Calendar Onboarding</field>
            <field name="step_ids" eval="[(4, ref('calendar.onboarding_onboarding_step_setup_calendar_integration'))]"/>
            <field name="panel_close_action_name">action_close_calendar_onboarding</field>
            <field name="route_name">calendar</field>
        </record>
    </data>
</odoo>

```

## File: data\mail_activity_type_data.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">
        <record id="mail.mail_activity_data_meeting" model="mail.activity.type">
            <field name="category">meeting</field>
        </record>
    </data>
</odoo>

```

## File: data\mail_message_subtype_data.xml

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
    </data>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">
        <record id="calendar_template_meeting_invitation" model="mail.template">
            <field name="name">Calendar: Meeting Invitation</field>
            <field name="model_id" ref="calendar.model_calendar_attendee"/>
            <field name="subject">Invitation to {{ object.event_id.name }}</field>
            <field name="email_from">{{ (object.event_id.user_id.email_formatted or user.email_formatted or '') }}</field>
            <field name="email_to">{{ ('' if object.partner_id.email and object.partner_id.email == object.email else object.email) }}</field>
            <field name="partner_to">{{ object.partner_id.id if object.partner_id.email and object.partner_id.email == object.email else False }}</field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="description">Invitation email to new attendees</field>
            <field name="body_html" type="html">
<div>
    <t t-set="colors" t-value="{'needsAction': 'grey', 'accepted': 'green', 'tentative': '#FFFF00', 'declined': 'red'}"/>
    <t t-set="customer" t-value=" object.event_id.find_partner_customer()"/>
    <t t-set="target_responsible" t-value="object.partner_id == object.event_id.partner_id"/>
    <t t-set="target_customer" t-value="object.partner_id == customer"/>
    <t t-set="recurrent" t-value="object.recurrence_id and not ctx.get('calendar_template_ignore_recurrence')"/>

    <p>
        Hello <t t-out="object.common_name or ''">Wood Corner</t>,<br/><br/>

        <t t-if="not target_responsible">
            <t t-out="object.event_id.user_id.partner_id.name or ''">Colleen Diaz</t> invited you for the <strong t-out="object.event_id.name or ''">Follow-up for Project proposal</strong> meeting.
        </t>
        <t t-else="">
            Your meeting <strong t-out="object.event_id.name or ''">Follow-up for Project proposal</strong> has been booked.
        </t>

    </p>
    <div style="text-align: center; padding: 16px 0px 16px 0px;">
        <a t-attf-href="/calendar/meeting/accept?token={{object.access_token}}&amp;id={{object.event_id.id}}"
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            Accept</a>
        <a t-attf-href="/calendar/meeting/decline?token={{object.access_token}}&amp;id={{object.event_id.id}}"
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            Decline</a>
        <a t-attf-href="/calendar/meeting/view?token={{object.access_token}}&amp;id={{object.event_id.id}}"
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px"
            >View</a>
    </div>
    <table border="0" cellpadding="0" cellspacing="0"><tr>
        <td width="130px;" style="min-width: 130px;">
            <div style="border-top-start-radius: 3px; border-top-end-radius: 3px; font-size: 12px; border-collapse: separate; text-align: center; font-weight: bold; color: #ffffff; min-height: 18px; background-color: #875A7B; border: 1px solid #875A7B;">
                <t t-out="format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format='EEEE', lang_code=object.env.lang) or ''">Tuesday</t>
            </div>
            <div style="font-size: 48px; min-height: auto; font-weight: bold; text-align: center; color: #5F5F5F; background-color: #F8F8F8; border: 1px solid #875A7B;">
                <t t-out="format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format='d', lang_code=object.env.lang) or ''">4</t>
            </div>
            <div style='font-size: 12px; text-align: center; font-weight: bold; color: #ffffff; background-color: #875A7B;'>
                <t t-out="format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format='MMMM y', lang_code=object.env.lang) or ''">May 2021</t>
            </div>
            <div style="border-collapse: separate; color: #5F5F5F; text-align: center; font-size: 12px; border-bottom-end-radius: 3px; font-weight: bold ; border: 1px solid #875A7B; border-bottom-start-radius: 3px;">
                <t t-if="not object.event_id.allday">
                    <div>
                        <t t-out="format_time(time=object.event_id.start, tz=object.mail_tz, time_format='short', lang_code=object.env.lang) or ''">11:00 AM</t>
                    </div>
                    <t t-if="object.mail_tz">
                        <div style="font-size: 10px; font-weight: normal">
                            (<t t-out="object.mail_tz or ''">Europe/Brussels</t>)
                        </div>
                    </t>
                </t>
            </div>
        </td>
        <td width="20px;"/>
        <td style="padding-top: 5px;">
            <p><strong>Details of the event</strong></p>
            <ul>
                <t t-if="object.event_id.location">
                    <li>Location: <t t-out="object.event_id.location or ''">Bruxelles</t>
                        (<a target="_blank" t-attf-href="http://maps.google.com/maps?oi=map&amp;q={{object.event_id.location}}">View Map</a>)
                    </li>
                </t>
                <t t-if="recurrent">
                    <li>When: <t t-out="object.recurrence_id.get_recurrence_name()">Every 1 Weeks, for 3 events</t></li>
                </t>
                <t t-if="not object.event_id.allday and object.event_id.duration">
                    <li>Duration: <t t-out="('%dH%02d' % (object.event_id.duration,round(object.event_id.duration*60)%60)) or ''">0H30</t></li>
                </t>
                <li>Attendees
                <ul>
                    <li t-foreach="object.event_id.attendee_ids" t-as="attendee">
                        <div t-attf-style="display: inline-block; border-radius: 50%; width: 10px; height: 10px; background:{{ colors.get(attendee.state) or 'white' }};"> </div>
                        <t t-if="attendee.common_name != object.common_name">
                            <span style="margin-left:5px" t-out="attendee.common_name or ''">Mitchell Admin</span>
                        </t>
                        <t t-else="">
                            <span style="margin-left:5px">You</span>
                        </t>
                    </li>
                </ul></li>
                <t t-if="object.event_id.videocall_location">
                    <li>
                        How to Join:
                        <t t-if="object.get_base_url() in object.event_id.videocall_location"> Join with Odoo Discuss</t>
                        <t t-else=""> Join at</t><br/>
                        <a t-att-href="object.event_id.videocall_location" target="_blank" t-out="object.event_id.videocall_location or ''">www.mycompany.com/calendar/join_videocall/xyz</a>
                    </li>
                </t>
                <t t-if="not is_html_empty(object.event_id.description)">
                    <li>Description of the event:
                    <t t-out="object.event_id.description">Internal meeting for discussion for new pricing for product and services.</t></li>
                </t>
            </ul>
        </td>
    </tr></table>
    <br/>
    Thank you,
    <t t-if="object.event_id.user_id.signature">
        <br />
        <t t-out="object.event_id.user_id.signature or ''">--<br/>Mitchell Admin</t>
    </t>
</div>
            </field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <record id="calendar_template_meeting_changedate" model="mail.template">
            <field name="name">Calendar: Date Updated</field>
            <field name="model_id" ref="calendar.model_calendar_attendee"/>
            <field name="subject">{{ object.event_id.name }}: Date updated</field>
            <field name="email_from">{{ (object.event_id.user_id.email_formatted or user.email_formatted or '') }}</field>
            <field name="email_to">{{ ('' if object.partner_id.email and object.partner_id.email == object.email else object.email) }}</field>
            <field name="partner_to">{{ object.partner_id.id if object.partner_id.email and object.partner_id.email == object.email else False }}</field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="description">Sent to all attendees if the schedule change</field>
            <field name="body_html" type="html">
<div>

    <t t-set="colors" t-value="{'needsAction': 'grey', 'accepted': 'green', 'tentative': '#FFFF00', 'declined': 'red'}"/>
    <t t-set="is_online" t-value="'appointment_type_id' in object.event_id and object.event_id.appointment_type_id"/>
    <t t-set="customer" t-value="object.event_id.find_partner_customer()"/>
    <t t-set="target_responsible" t-value="object.partner_id == object.event_id.partner_id"/>
    <t t-set="target_customer" t-value="object.partner_id == customer"/>
     <t t-set="recurrent" t-value="object.recurrence_id and not ctx.get('calendar_template_ignore_recurrence')"/>

    <p>
        Hello <t t-out="object.common_name or ''">Ready Mat</t>,<br/><br/>
        <t t-if="is_online and target_responsible">
            <t t-if="customer">
                The date of your appointment with <t t-out="customer.name or ''">Jesse Brown</t> has been updated.
            </t>
            <t t-else="">
                Your appointment has been updated.
            </t>
            The appointment <strong t-out="object.event_id.appointment_type_id.name or ''">Schedule a Demo</strong> is now scheduled for
            <t t-out="object.event_id.get_display_time_tz(tz=object.partner_id.tz) or ''">05/04/2021 at (11:00:00 To 11:30:00) (Europe/Brussels)</t>
        </t>
        <t t-elif="is_online and target_customer">
            The date of your appointment with <t t-out="object.event_id.user_id.partner_id.name or ''">Colleen Diaz</t> has been updated.
            The appointment <strong t-out="object.event_id.appointment_type_id.name or ''"></strong> is now scheduled for
            <t t-out="object.event_id.get_display_time_tz(tz=object.partner_id.tz) or ''">05/04/2021 at (11:00:00 To 11:30:00) (Europe/Brussels)</t>.
        </t>
        <t t-else="">
            The date of the meeting has been updated.
            The meeting <strong t-out="object.event_id.name or ''">Follow-up for Project proposal</strong> created by <t t-out="object.event_id.user_id.partner_id.name or ''">Colleen Diaz</t> is now scheduled for
            <t t-out="object.event_id.get_display_time_tz(tz=object.partner_id.tz) or ''">05/04/2021 at (11:00:00 To 11:30:00) (Europe/Brussels)</t>.
        </t>
    </p>
    <div style="text-align: center; padding: 16px 0px 16px 0px;">
        <a t-attf-href="/calendar/meeting/accept?token={{ object.access_token }}&amp;id={{ object.event_id.id }}"
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            Accept</a>
        <a t-attf-href="/calendar/meeting/decline?token={{ object.access_token }}&amp;id={{ object.event_id.id }}"
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            Decline</a>
        <a t-attf-href="/calendar/meeting/view?token={{ object.access_token }}&amp;id={{ object.event_id.id }}"
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            View</a>
    </div>
    <table border="0" cellpadding="0" cellspacing="0"><tr>
        <td width="130px;" style="min-width: 130px;">
            <div style="border-top-start-radius: 3px; border-top-end-radius: 3px; font-size: 12px; border-collapse: separate; text-align: center; font-weight: bold; color: #ffffff; min-height: 18px; background-color: #875A7B; border: 1px solid #875A7B;">
                <t t-out='format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format="EEEE", lang_code=object.env.lang) or ""'>Tuesday</t>
            </div>
            <div style="font-size: 48px; min-height: auto; font-weight: bold; text-align: center; color: #5F5F5F; background-color: #F8F8F8; border: 1px solid #875A7B;">
                <t t-out="format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format='d', lang_code=object.env.lang) or ''">4</t>
            </div>
            <div style='font-size: 12px; text-align: center; font-weight: bold; color: #ffffff; background-color: #875A7B;'>
                <t t-out='format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format="MMMM y", lang_code=object.env.lang) or ""'>May 2021</t>
            </div>
            <div style="border-collapse: separate; color: #5F5F5F; text-align: center; font-size: 12px; border-bottom-end-radius: 3px; font-weight: bold; border: 1px solid #875A7B; border-bottom-start-radius: 3px;">
                 <t t-if="not object.event_id.allday">
                    <div>
                        <t t-out='format_time(time=object.event_id.start, tz=object.mail_tz, time_format="short", lang_code=object.env.lang) or ""'>11:00 AM</t>
                    </div>
                    <t t-if="object.mail_tz">
                        <div style="font-size: 10px; font-weight: normal">
                            (<t t-out="object.mail_tz or ''">Europe/Brussels</t>)
                        </div>
                    </t>
                </t>
            </div>
        </td>
        <td width="20px;"/>
        <td style="padding-top: 5px;">
            <p><strong>Details of the event</strong></p>
            <ul>
                <t t-if="object.event_id.location">
                    <li>Location: <t t-out="object.event_id.location or ''">Bruxelles</t>
                        (<a target="_blank" t-attf-href="http://maps.google.com/maps?oi=map&amp;q={{ object.event_id.location }}">View Map</a>)
                    </li>
                </t>
                <t t-if="recurrent">
                    <li>When: <t t-out="object.recurrence_id.get_recurrence_name()">Every 1 Weeks, for 3 events</t></li>
                </t>
                <t t-if="not object.event_id.allday and object.event_id.duration">
                    <li>Duration: <t t-out="('%dH%02d' % (object.event_id.duration,round(object.event_id.duration*60)%60)) or ''">0H30</t></li>
                </t>
                <li>Attendees
                <ul>
                    <li t-foreach="object.event_id.attendee_ids" t-as="attendee">
                        <div t-attf-style="display: inline-block; border-radius: 50%; width: 10px; height: 10px; background: {{ colors.get(attendee.state) or 'white' }};"> </div>
                        <t t-if="attendee.common_name != object.common_name">
                            <span style="margin-left:5px" t-out="attendee.common_name or ''">Mitchell Admin</span>
                        </t>
                        <t t-else="">
                            <span style="margin-left:5px">You</span>
                        </t>
                    </li>
                </ul></li>
                <t t-if="object.event_id.videocall_location">
                    <li>
                        How to Join:
                        <t t-if="object.get_base_url() in object.event_id.videocall_location"> Join with Odoo Discuss</t>
                        <t t-else=""> Join at</t><br/>
                        <a t-att-href="object.event_id.videocall_location" target="_blank" t-out="object.event_id.videocall_location or ''">www.mycompany.com/calendar/join_videocall/xyz</a>
                    </li>
                </t>
                <t t-if="not is_html_empty(object.event_id.description)">
                    <li>Description of the event:
                    <t t-out="object.event_id.description">Internal meeting for discussion for new pricing for product and services.</t></li>
                </t>
            </ul>
        </td>
    </tr></table>
    <br/>
    Thank you,
    <t t-if="object.event_id.user_id.signature">
        <br />
        <t t-out="object.event_id.user_id.signature or ''">--<br/>Mitchell Admin</t>
    </t>
</div>
            </field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <record id="calendar_template_meeting_reminder" model="mail.template">
            <field name="name">Calendar: Reminder</field>
            <field name="model_id" ref="calendar.model_calendar_attendee"/>
            <field name="subject">{{ object.event_id.name }} - Reminder</field>
            <field name="email_from">{{ (object.event_id.user_id.email_formatted or user.email_formatted or '') }}</field>
            <field name="email_to">{{ ('' if object.partner_id.email and object.partner_id.email == object.email else object.email) }}</field>
            <field name="partner_to">{{ object.partner_id.id if object.partner_id.email and object.partner_id.email == object.email else False }}</field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="description">Sent to all attendees if a reminder is set</field>
            <field name="body_html" type="html">
<div>
    <t t-set="colors" t-value="{'needsAction': 'grey', 'accepted': 'green', 'tentative': '#FFFF00', 'declined': 'red'}" />
    <t t-set="is_online" t-value="'appointment_type_id' in object.event_id and object.event_id.appointment_type_id" />
    <t t-set="recurrent" t-value="object.recurrence_id and not ctx.get('calendar_template_ignore_recurrence')" />
    <p>
        Hello <t t-out="object.common_name or ''">Gemini Furniture</t>,<br/><br/>
        This is a reminder for the below event:
    </p>
    <div style="text-align: center; padding: 16px 0px 16px 0px;">
        <a t-attf-href="/calendar/{{ 'recurrence' if recurrent else 'meeting' }}/accept?token={{ object.access_token }}&amp;id={{ object.event_id.id }}"
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            Accept</a>
        <a t-attf-href="/calendar/{{ 'recurrence' if recurrent else 'meeting' }}/decline?token={{ object.access_token }}&amp;id={{ object.event_id.id }}"
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            Decline</a>
        <a t-attf-href="/calendar/meeting/view?token={{ object.access_token }}&amp;id={{ object.event_id.id }}" 
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
            View</a>
    </div>
    <table border="0" cellpadding="0" cellspacing="0"><tr>
        <td width="130px;" style="min-width: 130px;">
            <div style="border-top-start-radius: 3px; border-top-end-radius: 3px; font-size: 12px; border-collapse: separate; text-align: center; font-weight: bold; color: #ffffff; min-height: 18px; background-color: #875A7B; border: 1px solid #875A7B;">
                <t t-out='format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format="EEEE", lang_code=object.env.lang) or ""'>Tuesday</t>
            </div>
            <div style="font-size: 48px; min-height: auto; font-weight: bold; text-align: center; color: #5F5F5F; background-color: #F8F8F8; border: 1px solid #875A7B;">
                <t t-out="format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format='d', lang_code=object.env.lang) or ''">4</t>
            </div>
            <div style='font-size: 12px; text-align: center; font-weight: bold; color: #ffffff; background-color: #875A7B;'>
                <t t-out='format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format="MMMM y", lang_code=object.env.lang) or ""'>May 2021</t>
            </div>
            <div style="border-collapse: separate; color: #5F5F5F; text-align: center; font-size: 12px; border-bottom-end-radius: 3px; font-weight: bold; border: 1px solid #875A7B; border-bottom-start-radius: 3px;">
                <t t-if="not object.event_id.allday">
                    <div>
                        <t t-out='format_time(time=object.event_id.start, tz=object.mail_tz, time_format="short", lang_code=object.env.lang) or ""'>11:00 AM</t>
                    </div>
                    <t t-if="object.mail_tz">
                        <div style="font-size: 10px; font-weight: normal">
                            (<t t-out="object.mail_tz or ''">Europe/Brussels</t>)
                        </div>
                    </t>
                </t>
            </div>
        </td>
        <td width="20px;"/>
        <td style="padding-top: 5px;">
            <p><strong>Details of the event</strong></p>
            <ul>
                <t t-if="object.event_id.location">
                    <li>Location: <t t-out="object.event_id.location or ''">Bruxelles</t>
                        (<a target="_blank" t-attf-href="http://maps.google.com/maps?oi=map&amp;q={{ object.event_id.location }}">View Map</a>)
                    </li>
                </t>
                <t t-if="recurrent">
                    <li>When: <t t-out="object.recurrence_id.get_recurrence_name()">Every 1 Weeks, for 3 events</t></li>
                </t>
                <t t-if="not object.event_id.allday and object.event_id.duration">
                    <li>Duration: <t t-out="('%dH%02d' % (object.event_id.duration,round(object.event_id.duration*60)%60)) or ''">0H30</t></li>
                </t>
                <li>Attendees
                <ul>
                    <li t-foreach="object.event_id.attendee_ids" t-as="attendee">
                        <div t-attf-style="display: inline-block; border-radius: 50%; width: 10px; height: 10px; background:{{ colors.get(attendee.state) or 'white' }};"> </div>
                        <t t-if="attendee.common_name != object.common_name">
                            <span style="margin-left:5px" t-out="attendee.common_name or ''">Mitchell Admin</span>
                        </t>
                        <t t-else="">
                            <span style="margin-left:5px">You</span>
                        </t>
                    </li>
                </ul></li>
                <t t-if="object.event_id.videocall_location">
                    <li>
                        How to Join:
                        <t t-if="object.get_base_url() in object.event_id.videocall_location"> Join with Odoo Discuss</t>
                        <t t-else=""> Join at</t><br/>
                        <a t-att-href="object.event_id.videocall_location" target="_blank" t-out="object.event_id.videocall_location or ''">www.mycompany.com/calendar/join_videocall/xyz</a>
                    </li>
                </t>
                <t t-if="not is_html_empty(object.event_id.description)">
                    <li>Description of the event:
                    <t t-out="object.event_id.description">Internal meeting for discussion for new pricing for product and services.</t></li>
                </t>
            </ul>
        </td>
    </tr></table>
    <br/>
    Thank you,
    <t t-if="object.event_id.user_id.signature">
        <br />
        <t t-out="object.event_id.user_id.signature or ''">--<br/>Mitchell Admin</t>
    </t>
</div>
            </field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <record id="calendar_template_meeting_update" model="mail.template">
            <field name="name">Calendar: Event Update</field>
            <field name="model_id" ref="calendar.model_calendar_event"/>
            <field name="subject">{{object.name}}: Event update</field>
            <field name="email_from">{{ (object.user_id.email_formatted or user.email_formatted or '') }}</field>
            <field name="email_to">{{ object._get_attendee_emails() }}</field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="description">Used to manually notifiy attendees</field>
            <field name="body_html" type="html">
<div>
    <t t-set="colors" t-value="{'needsAction': 'grey', 'accepted': 'green', 'tentative': '#FFFF00', 'declined': 'red'}" />
    <t t-set="is_online" t-value="'appointment_type_id' in object and object.appointment_type_id" />
    <t t-set="target_responsible" t-value="object.partner_id == object.partner_id" />
    <t t-set="target_customer" t-value="object.partner_id == customer" />
    <t t-set="recurrent" t-value="object.recurrence_id and not ctx.get('calendar_template_ignore_recurrence')" />
    <t t-set="mail_tz" t-value="object._get_mail_tz() or ctx.get('mail_tz')" />
    <div>
        <table border="0" cellpadding="0" cellspacing="0">
            <tr>
                <td width="130px;" style="min-width: 130px;">
                    <div style="border-top-start-radius: 3px; border-top-end-radius: 3px; font-size: 12px; border-collapse: separate; text-align: center; font-weight: bold; color: #ffffff; min-height: 18px; background-color: #875A7B; border: 1px solid #875A7B;">
                        <t t-out="format_datetime(dt=object.start, tz=mail_tz if not object.allday else None, dt_format='EEEE', lang_code=object.env.lang) ">Tuesday</t>
                    </div>
                    <div style="font-size: 48px; min-height: auto; font-weight: bold; text-align: center; color: #5F5F5F; background-color: #F8F8F8; border: 1px solid #875A7B;">
                        <t t-out="format_datetime(dt=object.start, tz=mail_tz if not object.allday else None, dt_format='d', lang_code=object.env.lang)">4</t>
                    </div>
                    <div style='font-size: 12px; text-align: center; font-weight: bold; color: #ffffff; background-color: #875A7B;'>
                        <t t-out="format_datetime(dt=object.start, tz=mail_tz if not object.allday else None, dt_format='MMMM y', lang_code=object.env.lang)">May 2021</t>
                    </div>
                    <div style="border-collapse: separate; color: #5F5F5F; text-align: center; font-size: 12px; border-bottom-end-radius: 3px; font-weight: bold; border: 1px solid #875A7B; border-bottom-start-radius: 3px;">
                        <t t-if="not object.allday">
                            <div>
                                <t t-out="format_time(time=object.start, tz=mail_tz, time_format='short', lang_code=object.env.lang)">11:00 AM</t>
                            </div>
                            <t t-if="mail_tz">
                                <div style="font-size: 10px; font-weight: normal">
                                    (<t t-out="mail_tz"> Europe/Brussels</t>)
                                </div>
                            </t>
                        </t>
                    </div>
                </td>
                <td width="20px;"/>
                <td style="padding-top: 5px;">
                    <p>
                        <strong>Details of the event</strong>
                    </p>
                    <ul>
                        <t t-if="not is_html_empty(object.description)">
                            <li>Description:
                            <t t-out="object.description">Internal meeting for discussion for new pricing for product and services.</t></li>
                        </t>
                        <t t-if="object.videocall_location">
                            <li>
                                How to Join:
                                <t t-if="object.get_base_url() in object.videocall_location"> Join with Odoo Discuss</t>
                                <t t-else=""> Join at</t><br/>
                                <a t-att-href="object.videocall_location" target="_blank" t-out="object.videocall_location or ''">www.mycompany.com/calendar/join_videocall/xyz</a>
                            </li>
                        </t>
                        <t t-if="object.location">
                            <li>Location: <t t-out="object.location or ''">Bruxelles</t>
                                (<a target="_blank"
                                    t-attf-href="http://maps.google.com/maps?oi=map&amp;q={{object.location}}">View Map</a>)
                            </li>
                        </t>
                        <t t-if="recurrent">
                            <li>When: <t t-out="object.recurrence_id.get_recurrence_name()">Every 1 Weeks, for 3 events</t></li>
                        </t>
                        <t t-if="not object.allday and object.duration">
                            <li>Duration:
                                <t t-out="('%dH%02d' % (object.duration,round(object.duration*60)%60))">0H30</t>
                            </li>
                        </t>
                    </ul>
                </td>
            </tr>
        </table>
    </div>
    <div class="user_input">
        <hr/>
        <p placeholder="Enter your message here"><br/></p>

    </div>
    <t t-if="object.user_id.signature">
        <br />
        <t t-out="object.user_id.signature or ''">--<br/>Mitchell Admin</t>
    </t>
</div>
            </field>
        </record>
    </data>
</odoo>

```

## File: models\calendar_alarm.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Alarm(models.Model):
    _name = 'calendar.alarm'
    _description = 'Event Alarm'

    _interval_selection = {'minutes': 'Minutes', 'hours': 'Hours', 'days': 'Days'}

    name = fields.Char('Name', translate=True, required=True)
    alarm_type = fields.Selection(
        [('notification', 'Notification'), ('email', 'Email')],
        string='Type', required=True, default='email')
    duration = fields.Integer('Remind Before', required=True, default=1)
    interval = fields.Selection(
        list(_interval_selection.items()), 'Unit', required=True, default='hours')
    duration_minutes = fields.Integer(
        'Duration in minutes', store=True,
        search='_search_duration_minutes', compute='_compute_duration_minutes')
    mail_template_id = fields.Many2one(
        'mail.template', string="Email Template",
        domain=[('model', 'in', ['calendar.attendee'])],
        compute='_compute_mail_template_id', readonly=False, store=True,
        help="Template used to render mail reminder content.")
    body = fields.Text("Additional Message", help="Additional message that would be sent with the notification for the reminder")

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

    @api.depends('alarm_type', 'mail_template_id')
    def _compute_mail_template_id(self):
        for alarm in self:
            if alarm.alarm_type == 'email' and not alarm.mail_template_id:
                alarm.mail_template_id = self.env['ir.model.data']._xmlid_to_res_id('calendar.calendar_template_meeting_reminder')
            elif alarm.alarm_type != 'email' or not alarm.mail_template_id:
                alarm.mail_template_id = False

    def _search_duration_minutes(self, operator, value):
        return [
            '|', '|',
            '&', ('interval', '=', 'minutes'), ('duration', operator, value),
            '&', ('interval', '=', 'hours'), ('duration', operator, value / 60),
            '&', ('interval', '=', 'days'), ('duration', operator, value / 60 / 24),
        ]

    @api.onchange('duration', 'interval', 'alarm_type')
    def _onchange_duration_interval(self):
        display_interval = self._interval_selection.get(self.interval, '')
        display_alarm_type = {
            key: value for key, value in self._fields['alarm_type']._description_selection(self.env)
        }[self.alarm_type]
        self.name = "%s - %s %s" % (display_alarm_type, self.duration, display_interval)

```

## File: models\calendar_alarm_manager.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from datetime import timedelta
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models
from odoo.tools import plaintext2html

_logger = logging.getLogger(__name__)


class AlarmManager(models.AbstractModel):
    _name = 'calendar.alarm_manager'
    _description = 'Event Alarm Manager'

    def _get_next_potential_limit_alarm(self, alarm_type, seconds=None, partners=None):
        # flush models before making queries
        for model_name in ('calendar.alarm', 'calendar.event', 'calendar.recurrence'):
            self.env[model_name].flush_model()

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
                            WHEN cal.recurrency THEN rrule.until - interval '1' minute  * calcul_delta.min_delta
                            ELSE cal.stop - interval '1' minute  * calcul_delta.min_delta
                        END as last_alarm,
                        cal.start as first_event_date,
                        CASE
                            WHEN cal.recurrency AND rrule.end_type = 'end_date' THEN rrule.until
                            ELSE cal.stop
                        END as last_event_date,
                        calcul_delta.min_delta,
                        calcul_delta.max_delta,
                        rrule.rrule AS rule
                    FROM
                        calendar_event AS cal
                    RIGHT JOIN calcul_delta ON calcul_delta.calendar_event_id = cal.id
                    LEFT JOIN calendar_recurrence as rrule ON rrule.id = cal.recurrence_id
             """

        filter_user = """
                RIGHT JOIN calendar_event_res_partner_rel AS part_rel ON part_rel.calendar_event_id = cal.id
                    AND part_rel.res_partner_id IN %s
        """

        # Add filter on alarm type
        tuple_params = (alarm_type,)

        # Add filter on partner_id
        if partners:
            base_request += filter_user
            tuple_params += (tuple(partners.ids), )

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

        self.env.flush_all()
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
        past = one_date - timedelta(minutes=(missing * event_maxdelta))
        future = fields.Datetime.now() + timedelta(seconds=in_the_next_X_seconds)
        if future <= past:
            return result
        for alarm in event.alarm_ids:
            if alarm.alarm_type != alarm_type:
                continue
            past = one_date - timedelta(minutes=(missing * alarm.duration_minutes))
            if future <= past:
                continue
            if after and past <= fields.Datetime.from_string(after):
                continue
            result.append({
                'alarm_id': alarm.id,
                'event_id': event.id,
                'notify_at': one_date - timedelta(minutes=alarm.duration_minutes),
            })
        return result

    def _get_events_by_alarm_to_notify(self, alarm_type):
        """
        Get the events with an alarm of the given type between the cron
        last call and now.

        Please note that all new reminders created since the cron last
        call with an alarm prior to the cron last call are skipped by
        design. The attendees receive an invitation for any new event
        already.
        """
        lastcall = self.env.context.get('lastcall', False) or fields.date.today() - relativedelta(weeks=1)
        now = fields.Datetime.now()
        self.env.cr.execute('''
            SELECT "alarm"."id", "event"."id"
              FROM "calendar_event" AS "event"
              JOIN "calendar_alarm_calendar_event_rel" AS "event_alarm_rel"
                ON "event"."id" = "event_alarm_rel"."calendar_event_id"
              JOIN "calendar_alarm" AS "alarm"
                ON "event_alarm_rel"."calendar_alarm_id" = "alarm"."id"
             WHERE (
                   "alarm"."alarm_type" = %s
               AND "event"."active"
               AND "event"."start" - CAST("alarm"."duration" || ' ' || "alarm"."interval" AS Interval) >= %s
               AND "event"."start" - CAST("alarm"."duration" || ' ' || "alarm"."interval" AS Interval) < %s
             )''', [alarm_type, lastcall, now])

        events_by_alarm = {}
        for alarm_id, event_id in self.env.cr.fetchall():
            events_by_alarm.setdefault(alarm_id, list()).append(event_id)
        return events_by_alarm

    @api.model
    def _send_reminder(self):
        # Executed via cron
        events_by_alarm = self._get_events_by_alarm_to_notify('email')
        if not events_by_alarm:
            return

        event_ids = list(set(event_id for event_ids in events_by_alarm.values() for event_id in event_ids))
        events = self.env['calendar.event'].browse(event_ids)
        now = fields.Datetime.now()
        attendees = events.filtered(lambda e: e.stop > now).attendee_ids.filtered(lambda a: a.state != 'declined')
        alarms = self.env['calendar.alarm'].browse(events_by_alarm.keys())
        for alarm in alarms:
            alarm_attendees = attendees.filtered(lambda attendee: attendee.event_id.id in events_by_alarm[alarm.id])
            alarm_attendees.with_context(
                calendar_template_ignore_recurrence=True,
                mail_notify_author=True,
            )._send_mail_to_attendees(
                alarm.mail_template_id,
                force_send=True
            )

        for event in events:
            if event.recurrence_id:
                next_date = event.get_next_alarm_date(events_by_alarm)
                # In cron, setup alarm only when there is a next date on the target. Otherwise the 'now()'
                # check in the call below can generate undeterministic behavior and setup random alarms.
                if next_date:
                    event.recurrence_id.with_context(date=next_date)._setup_alarms()

    @api.model
    def get_next_notif(self):
        partner = self.env.user.partner_id
        all_notif = []

        if not partner:
            return []

        all_meetings = self._get_next_potential_limit_alarm('notification', partners=partner)
        time_limit = 3600 * 24  # return alarms of the next 24 hours
        for event_id in all_meetings:
            max_delta = all_meetings[event_id]['max_duration']
            meeting = self.env['calendar.event'].browse(event_id)
            in_date_format = fields.Datetime.from_string(meeting.start)
            last_found = self.do_check_alarm_for_one_date(in_date_format, meeting, max_delta, time_limit, 'notification', after=partner.calendar_last_notif_ack)
            if last_found:
                for alert in last_found:
                    all_notif.append(self.do_notif_reminder(alert))
        return all_notif

    def do_notif_reminder(self, alert):
        alarm = self.env['calendar.alarm'].browse(alert['alarm_id'])
        meeting = self.env['calendar.event'].browse(alert['event_id'])

        if alarm.alarm_type == 'notification':
            message = meeting.display_time
            if alarm.body:
                message += '<p>%s</p>' % plaintext2html(alarm.body)

            delta = alert['notify_at'] - fields.Datetime.now()
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
        users = self.env['res.users'].search([
            ('partner_id', 'in', tuple(partner_ids)),
            ('groups_id', 'in', self.env.ref('base.group_user').ids),
        ])
        for user in users:
            notif = self.with_user(user).with_context(allowed_company_ids=user.company_ids.ids).get_next_notif()
            notifications.append([user.partner_id, 'calendar.alarm', notif])
        if len(notifications) > 0:
            self.env['bus.bus']._sendmany(notifications)

```

## File: models\calendar_attendee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import uuid
import base64
import logging

from collections import defaultdict
from odoo import api, fields, models, _
from odoo.addons.base.models.res_partner import _tz_get
from odoo.exceptions import UserError
from odoo.tools.misc import clean_context
from odoo.tools import split_every

_logger = logging.getLogger(__name__)


class Attendee(models.Model):
    """ Calendar Attendee Information """
    _name = 'calendar.attendee'
    _rec_name = 'common_name'
    _description = 'Calendar Attendee Information'
    _order = 'create_date ASC'

    def _default_access_token(self):
        return uuid.uuid4().hex

    STATE_SELECTION = [
        ('needsAction', 'Needs Action'),
        ('tentative', 'Maybe'),
        ('declined', 'No'),
        ('accepted', 'Yes'),
    ]

    # event
    event_id = fields.Many2one('calendar.event', 'Meeting linked', required=True, ondelete='cascade')
    recurrence_id = fields.Many2one('calendar.recurrence', related='event_id.recurrence_id')
    # attendee
    partner_id = fields.Many2one('res.partner', 'Attendee', required=True, readonly=True, ondelete='cascade')
    email = fields.Char('Email', related='partner_id.email')
    phone = fields.Char('Phone', related='partner_id.phone')
    common_name = fields.Char('Common name', compute='_compute_common_name', store=True)
    access_token = fields.Char('Invitation Token', default=_default_access_token)
    mail_tz = fields.Selection(_tz_get, compute='_compute_mail_tz', help='Timezone used for displaying time in the mail template')
    # state
    state = fields.Selection(STATE_SELECTION, string='Status', default='needsAction')
    availability = fields.Selection(
        [('free', 'Available'), ('busy', 'Busy')], 'Available/Busy', readonly=True)

    @api.depends('partner_id', 'partner_id.name', 'email')
    def _compute_common_name(self):
        for attendee in self:
            attendee.common_name = attendee.partner_id.name or attendee.email

    def _compute_mail_tz(self):
        for attendee in self:
            attendee.mail_tz = attendee.partner_id.tz

    @api.model_create_multi
    def create(self, vals_list):
        for values in vals_list:
            # by default, if no state is given for the attendee corresponding to the current user
            # that means he's the event organizer so we can set his state to "accepted"
            if 'state' not in values and values.get('partner_id') == self.env.user.partner_id.id:
                values['state'] = 'accepted'
            if not values.get("email") and values.get("common_name"):
                common_nameval = values.get("common_name").split(':')
                email = [x for x in common_nameval if '@' in x]
                values['email'] = email[0] if email else ''
                values['common_name'] = values.get("common_name")
        attendees = super().create(vals_list)
        attendees._subscribe_partner()
        return attendees

    def unlink(self):
        self._unsubscribe_partner()
        return super().unlink()

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        raise UserError(_('You cannot duplicate a calendar attendee.'))

    def _subscribe_partner(self):
        mapped_followers = defaultdict(lambda: self.env['calendar.event'])
        for event in self.event_id:
            partners = (event.attendee_ids & self).partner_id - event.message_partner_ids
            # current user is automatically added as followers, don't add it twice.
            partners -= self.env.user.partner_id
            mapped_followers[partners] |= event
        for partners, events in mapped_followers.items():
            if not partners:
                continue
            events.message_subscribe(partner_ids=partners.ids)

    def _unsubscribe_partner(self):
        for event in self.event_id:
            partners = (event.attendee_ids & self).partner_id & event.message_partner_ids
            event.message_unsubscribe(partner_ids=partners.ids)

    def _send_invitation_emails(self):
        """ Hook to be able to override the invitation email sending process.
         Notably inside appointment to use a different mail template from the appointment type. """
        self._send_mail_to_attendees(
            self.env.ref('calendar.calendar_template_meeting_invitation', raise_if_not_found=False)
        )

    def _send_mail_to_attendees(self, mail_template, force_send=False):
        """ Send mail for event invitation to event attendees.
            :param mail_template: a mail.template record
            :param force_send: if set to True, the mail(s) will be sent immediately (instead of the next queue processing)
        """
        if isinstance(mail_template, str):
            raise ValueError('Template should be a template record, not an XML ID anymore.')
        if self.env['ir.config_parameter'].sudo().get_param('calendar.block_mail') or self._context.get("no_mail_to_attendees"):
            return False
        if not mail_template:
            _logger.warning("No template passed to %s notification process. Skipped.", self)
            return False

        # get ics file for all meetings
        ics_files = self.mapped('event_id')._get_ics_file()

        for attendee in self:
            if attendee.email and attendee._should_notify_attendee():
                event_id = attendee.event_id.id
                ics_file = ics_files.get(event_id)

                # Copy and add template attachments copies to the attendee's email, if available
                attachment_ids = [a.copy({'res_id': 0, 'res_model': 'mail.compose.message'}).id for a in mail_template.attachment_ids]

                if ics_file:
                    context = {
                        **clean_context(self.env.context),
                        'no_document': True, # An ICS file must not create a document
                    }
                    attachment_ids += self.env['ir.attachment'].with_context(context).create({
                        'datas': base64.b64encode(ics_file),
                        'description': 'invitation.ics',
                        'mimetype': 'text/calendar',
                        'res_id': 0,
                        'res_model': 'mail.compose.message',
                        'name': 'invitation.ics',
                    }).ids

                body = mail_template._render_field(
                    'body_html',
                    attendee.ids,
                    compute_lang=True)[attendee.id]
                subject = mail_template._render_field(
                    'subject',
                    attendee.ids,
                    compute_lang=True)[attendee.id]
                attendee.event_id.with_context(no_document=True).sudo().message_notify(
                    email_from=attendee.event_id.user_id.email_formatted or self.env.user.email_formatted,
                    author_id=attendee.event_id.user_id.partner_id.id or self.env.user.partner_id.id,
                    body=body,
                    subject=subject,
                    partner_ids=attendee.partner_id.ids,
                    email_layout_xmlid='mail.mail_notification_light',
                    attachment_ids=attachment_ids,
                    force_send=force_send,
                )

    def _should_notify_attendee(self):
        """ Utility method that determines if the attendee should be notified.
            By default, we do not want to notify (aka no message and no mail) the current user
            if he is part of the attendees. But for reminders, mail_notify_author could be forced
            (Override in appointment to ignore that rule and notify all attendees if it's an appointment)
        """
        self.ensure_one()
        partner_not_sender = self.partner_id != self.env.user.partner_id
        mail_notify_author = self.env.context.get('mail_notify_author')
        return partner_not_sender or mail_notify_author

    def do_tentative(self):
        """ Makes event invitation as Tentative. """
        return self.write({'state': 'tentative'})

    def do_accept(self):
        """ Marks event invitation as Accepted. """
        for attendee in self:
            attendee.event_id.message_post(
                author_id=attendee.partner_id.id,
                body=_("%s has accepted the invitation", attendee.common_name),
                subtype_xmlid="calendar.subtype_invitation",
            )
        return self.write({'state': 'accepted'})

    def do_decline(self):
        """ Marks event invitation as Declined. """
        for attendee in self:
            attendee.event_id.message_post(
                author_id=attendee.partner_id.id,
                body=_("%s has declined the invitation", attendee.common_name),
                subtype_xmlid="calendar.subtype_invitation",
            )
        return self.write({'state': 'declined'})

```

## File: models\calendar_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import math
from datetime import datetime, timedelta
from itertools import repeat
from werkzeug.urls import url_parse

import pytz
import uuid

from odoo import api, fields, models, Command
from odoo.osv.expression import AND
from odoo.addons.base.models.res_partner import _tz_get
from odoo.addons.calendar.models.calendar_attendee import Attendee
from odoo.addons.calendar.models.calendar_recurrence import (
    weekday_to_field,
    RRULE_TYPE_SELECTION,
    END_TYPE_SELECTION,
    MONTH_BY_SELECTION,
    WEEKDAY_SELECTION,
    BYDAY_SELECTION
)
from odoo.tools.translate import _
from odoo.tools.misc import get_lang
from odoo.tools import pycompat, html2plaintext, is_html_empty, single_email_re
from odoo.exceptions import UserError, ValidationError

_logger = logging.getLogger(__name__)

try:
    import vobject
except ImportError:
    _logger.warning("`vobject` Python module not found, iCal file generation disabled. Consider installing this module if you want to generate iCal files")
    vobject = None

SORT_ALIASES = {
    'start': 'sort_start',
    'start_date': 'sort_start',
}

RRULE_TYPE_SELECTION_UI = [
    ('daily', 'Daily'),
    ('weekly', 'Weekly'),
    ('monthly', 'Monthly'),
    ('yearly', 'Yearly'),
    ('custom', 'Custom')
]

def get_weekday_occurence(date):
    """
    :returns: ocurrence

    >>> get_weekday_occurence(date(2019, 12, 17))
    3  # third Tuesday of the month

    >>> get_weekday_occurence(date(2019, 12, 25))
    -1  # last Friday of the month
    """
    occurence_in_month = math.ceil(date.day/7)
    if occurence_in_month in {4, 5}:  # fourth or fifth week on the month -> last
        return -1
    return occurence_in_month


class Meeting(models.Model):
    _name = 'calendar.event'
    _description = "Calendar Event"
    _order = "start desc"
    _inherit = ["mail.thread"]
    _systray_view = 'calendar'

    DISCUSS_ROUTE = 'calendar/join_videocall'

    @api.model
    def get_state_selections(self):
        return Attendee.STATE_SELECTION

    @api.model
    def default_get(self, fields):
        # super default_model='crm.lead' for easier use in addons
        if self.env.context.get('default_res_model') and not self.env.context.get('default_res_model_id'):
            self = self.with_context(
                default_res_model_id=self.env['ir.model']._get_id(self.env.context['default_res_model'])
            )

        defaults = super(Meeting, self).default_get(fields)

        # support active_model / active_id as replacement of default_* if not already given
        if 'res_model_id' not in defaults and 'res_model_id' in fields and \
                self.env.context.get('active_model') and self.env.context['active_model'] != 'calendar.event':
            defaults['res_model_id'] = self.env['ir.model']._get_id(self.env.context['active_model'])
            defaults['res_model'] = self.env.context.get('active_model')
        if 'res_id' not in defaults and 'res_id' in fields and \
                defaults.get('res_model_id') and self.env.context.get('active_id'):
            defaults['res_id'] = self.env.context['active_id']

        return defaults

    @api.model
    def _default_partners(self):
        """ When active_model is res.partner, the current partners should be attendees """
        partners = self.env.user.partner_id
        active_id = self._context.get('active_id')
        if self._context.get('active_model') == 'res.partner' and active_id and active_id not in partners.ids:
                partners |= self.env['res.partner'].browse(active_id)
        return partners

    @api.model
    def _default_start(self):
        now = fields.Datetime.now()
        return now + (datetime.min - now) % timedelta(minutes=30)

    @api.model
    def _default_stop(self):
        now = fields.Datetime.now()
        start = now + (datetime.min - now) % timedelta(minutes=30)
        return start + timedelta(hours=1)

    # description
    name = fields.Char('Meeting Subject', required=True)
    description = fields.Html('Description')
    user_id = fields.Many2one('res.users', 'Organizer', default=lambda self: self.env.user)
    partner_id = fields.Many2one(
        'res.partner', string='Scheduled by', related='user_id.partner_id', readonly=True)
    location = fields.Char('Location', tracking=True)
    videocall_location = fields.Char('Meeting URL', compute='_compute_videocall_location', store=True, copy=True)
    access_token = fields.Char('Invitation Token', store=True, copy=False, index=True)
    videocall_source = fields.Selection([('discuss', 'Discuss'), ('custom', 'Custom')], compute='_compute_videocall_source')
    videocall_channel_id = fields.Many2one('discuss.channel', 'Discuss Channel')
    # visibility
    privacy = fields.Selection(
        [('public', 'Public'),
         ('private', 'Private'),
         ('confidential', 'Only internal users')],
        'Privacy', default='public', required=True,
        help="People to whom this event will be visible.")
    show_as = fields.Selection(
        [('free', 'Available'),
         ('busy', 'Busy')], 'Show as', default='busy', required=True,
        help="If the time is shown as 'busy', this event will be visible to other people with either the full \
        information or simply 'busy' written depending on its privacy. Use this option to let other people know \
        that you are unavailable during that period of time. \n If the event is shown as 'free', other users know \
        that you are available during that period of time.")
    is_highlighted = fields.Boolean(
        compute='_compute_is_highlighted', string='Is the Event Highlighted')
    is_organizer_alone = fields.Boolean(compute='_compute_is_organizer_alone', string="Is the Organizer Alone",
        help="""Check if the organizer is alone in the event, i.e. if the organizer is the only one that hasn't declined
        the event (only if the organizer is not the only attendee)""")
    # filtering
    active = fields.Boolean(
        'Active', default=True,
        tracking=True,
        help="If the active field is set to false, it will allow you to hide the event alarm information without removing it.")
    categ_ids = fields.Many2many(
        'calendar.event.type', 'meeting_category_rel', 'event_id', 'type_id', 'Tags')
    # timing
    start = fields.Datetime(
        'Start', required=True, tracking=True, default=_default_start,
        help="Start date of an event, without time for full days events")
    stop = fields.Datetime(
        'Stop', required=True, tracking=True, default=_default_stop,
        compute='_compute_stop', readonly=False, store=True,
        help="Stop date of an event, without time for full days events")
    display_time = fields.Char('Event Time', compute='_compute_display_time')
    allday = fields.Boolean('All Day', default=False)
    start_date = fields.Date(
        'Start Date', store=True, tracking=True,
        compute='_compute_dates', inverse='_inverse_dates')
    stop_date = fields.Date(
        'End Date', store=True, tracking=True,
        compute='_compute_dates', inverse='_inverse_dates')
    duration = fields.Float('Duration', compute='_compute_duration', store=True, readonly=False)
    # linked document
    res_id = fields.Many2oneReference('Document ID', model_field='res_model')
    res_model_id = fields.Many2one('ir.model', 'Document Model', ondelete='cascade')
    res_model = fields.Char(
        'Document Model Name', related='res_model_id.model', readonly=True, store=True)
    res_model_name = fields.Char(related='res_model_id.name')
    # messaging
    activity_ids = fields.One2many('mail.activity', 'calendar_event_id', string='Activities')
    # attendees
    attendee_ids = fields.One2many(
        'calendar.attendee', 'event_id', 'Participant')
    current_attendee = fields.Many2one("calendar.attendee", compute="_compute_current_attendee", search="_search_current_attendee")
    current_status = fields.Selection(string="Attending?", related="current_attendee.state", readonly=False)
    should_show_status = fields.Boolean(compute="_compute_should_show_status")
    partner_ids = fields.Many2many(
        'res.partner', 'calendar_event_res_partner_rel',
        string='Attendees', default=_default_partners)
    invalid_email_partner_ids = fields.Many2many('res.partner', compute='_compute_invalid_email_partner_ids')
    # alarms
    alarm_ids = fields.Many2many(
        'calendar.alarm', 'calendar_alarm_calendar_event_rel',
        string='Reminders', ondelete="restrict",
        help="Notifications sent to all attendees to remind of the meeting.")
    # RECURRENCE FIELD
    recurrency = fields.Boolean('Recurrent')
    recurrence_id = fields.Many2one(
        'calendar.recurrence', string="Recurrence Rule")
    follow_recurrence = fields.Boolean(default=False) # Indicates if an event follows the recurrence, i.e. is not an exception
    recurrence_update = fields.Selection([
        ('self_only', "This event"),
        ('future_events', "This and following events"),
        ('all_events', "All events"),
    ], store=False, copy=False, default='self_only',
       help="Choose what to do with other events in the recurrence. Updating All Events is not allowed when dates or time is modified")
    # Those field are pseudo-related fields of recurrence_id.
    # They can't be "real" related fields because it should work at record creation
    # when recurrence_id is not created yet.
    # If some of these fields are set and recurrence_id does not exists,
    # a `calendar.recurrence.rule` will be dynamically created.
    rrule = fields.Char('Recurrent Rule', compute='_compute_recurrence', readonly=False)
    rrule_type_ui = fields.Selection(RRULE_TYPE_SELECTION_UI, string='Repeat',
                                     compute="_compute_rrule_type_ui",
                                     readonly=False,
                                     help="Let the event automatically repeat at that interval")
    rrule_type = fields.Selection(RRULE_TYPE_SELECTION, string='Recurrence',
                                  help="Let the event automatically repeat at that interval",
                                  compute='_compute_recurrence', readonly=False)
    event_tz = fields.Selection(
        _tz_get, string='Timezone', compute='_compute_recurrence', readonly=False)
    end_type = fields.Selection(
        END_TYPE_SELECTION, string='Recurrence Termination',
        compute='_compute_recurrence', readonly=False)
    interval = fields.Integer(
        string='Repeat On', compute='_compute_recurrence', readonly=False,
        help="Repeat every (Days/Week/Month/Year)")
    count = fields.Integer(
        string='Number of Repetitions', help="Repeat x times", compute='_compute_recurrence', readonly=False)
    mon = fields.Boolean(compute='_compute_recurrence', readonly=False)
    tue = fields.Boolean(compute='_compute_recurrence', readonly=False)
    wed = fields.Boolean(compute='_compute_recurrence', readonly=False)
    thu = fields.Boolean(compute='_compute_recurrence', readonly=False)
    fri = fields.Boolean(compute='_compute_recurrence', readonly=False)
    sat = fields.Boolean(compute='_compute_recurrence', readonly=False)
    sun = fields.Boolean(compute='_compute_recurrence', readonly=False)
    month_by = fields.Selection(
        MONTH_BY_SELECTION, string='Option', compute='_compute_recurrence', readonly=False)
    day = fields.Integer('Date of month', compute='_compute_recurrence', readonly=False)
    weekday = fields.Selection(WEEKDAY_SELECTION, compute='_compute_recurrence', readonly=False)
    byday = fields.Selection(BYDAY_SELECTION, string="By day", compute='_compute_recurrence', readonly=False)
    until = fields.Date(compute='_compute_recurrence', readonly=False)
    # UI Fields.
    display_description = fields.Boolean(compute='_compute_display_description')
    attendees_count = fields.Integer(compute='_compute_attendees_count')
    accepted_count = fields.Integer(compute='_compute_attendees_count')
    declined_count = fields.Integer(compute='_compute_attendees_count')
    tentative_count = fields.Integer(compute='_compute_attendees_count')
    awaiting_count = fields.Integer(compute="_compute_attendees_count")
    user_can_edit = fields.Boolean(compute='_compute_user_can_edit')

    @api.depends("attendee_ids")
    def _compute_should_show_status(self):
        for event in self:
            event.should_show_status = event.current_attendee and any(attendee.partner_id != self.env.user.partner_id for attendee in event.attendee_ids)

    @api.depends('attendee_ids', 'attendee_ids.state')
    def _compute_current_attendee(self):
        for event in self:
            current_attendee = event.attendee_ids.filtered(lambda attendee: attendee.partner_id == self.env.user.partner_id)
            event.current_attendee = current_attendee and current_attendee[0]

    def _search_current_attendee(self, operator, value):
        return [("id", operator, value)]

    @api.depends('attendee_ids', 'attendee_ids.state', 'partner_ids')
    def _compute_attendees_count(self):
        for event in self:
            count_event = {}
            for attendee in event.attendee_ids:
                count_event[attendee.state] = count_event.get(attendee.state, 0) + 1

            accepted_count = count_event.get('accepted', 0)
            declined_count = count_event.get('declined', 0)
            tentative_count = count_event.get('tentative', 0)
            attendees_count = len(event.partner_ids)
            event.update({
                'accepted_count': accepted_count,
                'declined_count': declined_count,
                'tentative_count': tentative_count,
                'attendees_count': attendees_count,
                'awaiting_count': attendees_count - accepted_count - declined_count - tentative_count
            })

    @api.depends('partner_ids')
    @api.depends_context('uid')
    def _compute_user_can_edit(self):
        for event in self:
            # By default, only current attendees and the organizer can edit the event.
            editor_candidates = set(event.partner_ids.user_ids + event.user_id)
            # Right before saving the event, old partners must be able to save changes.
            if event._origin:
                editor_candidates |= set(event._origin.partner_ids.user_ids)
            # Non-private events must be editable by uninvited administrators.
            if self.env.user.has_group('base.group_system') and event.privacy != 'private':
                editor_candidates.add(self.env.user)
            event.user_can_edit = self.env.user in editor_candidates

    @api.depends('attendee_ids')
    def _compute_invalid_email_partner_ids(self):
        for event in self:
            event.invalid_email_partner_ids = event.partner_ids.filtered(
                lambda a: not (a.email and single_email_re.match(a.email))
            )

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

    @api.depends('partner_id', 'attendee_ids')
    def _compute_is_organizer_alone(self):
        """
            Check if the organizer of the event is the only one who has accepted the event.
            It does not apply if the organizer is the only attendee of the event because it
            would represent a personnal event.
            The goal of this field is to highlight to the user that the others attendees are
            not available for this event.
        """
        for event in self:
            organizer = event.attendee_ids.filtered(lambda a: a.partner_id == event.partner_id)
            all_declined = not any((event.attendee_ids - organizer).filtered(lambda a: a.state != 'declined'))
            event.is_organizer_alone = len(event.attendee_ids) > 1 and all_declined

    def _compute_display_time(self):
        for meeting in self:
            meeting.display_time = self._get_display_time(meeting.start, meeting.stop, meeting.duration, meeting.allday)

    @api.depends('allday', 'start', 'stop')
    def _compute_dates(self):
        """ Adapt the value of start_date(time)/stop_date(time)
            according to start/stop fields and allday. Also, compute
            the duration for not allday meeting ; otherwise the
            duration is set to zero, since the meeting last all the day.
        """
        for meeting in self:
            if meeting.allday and meeting.start and meeting.stop:
                meeting.start_date = meeting.start.date()
                meeting.stop_date = meeting.stop.date()
            else:
                meeting.start_date = False
                meeting.stop_date = False

    @api.depends('stop', 'start')
    def _compute_duration(self):
        for event in self:
            event.duration = self._get_duration(event.start, event.stop)

    @api.depends('start', 'duration')
    def _compute_stop(self):
        # stop and duration fields both depends on the start field.
        # But they also depends on each other.
        # When start is updated, we want to update the stop datetime based on
        # the *current* duration. In other words, we want: change start => keep the duration fixed and
        # recompute stop accordingly.
        # However, while computing stop, duration is marked to be recomputed. Calling `event.duration` would trigger
        # its recomputation. To avoid this we manually mark the field as computed.
        duration_field = self._fields['duration']
        self.env.remove_to_compute(duration_field, self)
        for event in self:
            # Round the duration (in hours) to the minute to avoid weird situations where the event
            # stops at 4:19:59, later displayed as 4:19.
            event.stop = event.start and event.start + timedelta(minutes=round((event.duration or 1.0) * 60))
            if event.allday:
                event.stop -= timedelta(seconds=1)

    @api.onchange('start_date', 'stop_date')
    def _onchange_date(self):
        """ This onchange is required for cases where the stop/start is False and we set an allday event.
            The inverse method is not called in this case because start_date/stop_date are not used in any
            compute/related, so we need an onchange to set the start/stop values in the form view
        """
        for event in self:
            if event.stop_date and event.start_date:
                event.with_context(is_calendar_event_new=True).write({
                    'start': fields.Datetime.from_string(event.start_date).replace(hour=8),
                    'stop': fields.Datetime.from_string(event.stop_date).replace(hour=18),
                })

    def _inverse_dates(self):
        """ This method is used to set the start and stop values of all day events.
            The calendar view needs date_start and date_stop values to display correctly the allday events across
            several days. As the user edit the {start,stop}_date fields when allday is true,
            this inverse method is needed to update the  start/stop value and have a relevant calendar view.
        """
        for meeting in self:
            if meeting.allday:

                # Convention break:
                # stop and start are NOT in UTC in allday event
                # in this case, they actually represent a date
                # because fullcalendar just drops times for full day events.
                # i.e. Christmas is on 25/12 for everyone
                # even if people don't celebrate it simultaneously
                enddate = fields.Datetime.from_string(meeting.stop_date or meeting.stop)
                enddate = enddate.replace(hour=18)

                startdate = fields.Datetime.from_string(meeting.start_date or meeting.start)
                startdate = startdate.replace(hour=8)  # Set 8 AM

                if meeting.start_date and meeting.stop_date:
                    # If start_date or stop_date is set, use start_date and stop_date;
                    # otherwise, use start and stop.
                    meeting.write({
                        'start': startdate.replace(tzinfo=None),
                        'stop': enddate.replace(tzinfo=None)
                    })
                else:
                    meeting.write({
                        'start_date': startdate.replace(tzinfo=None),
                        'stop_date': enddate.replace(tzinfo=None)
                    })

    @api.constrains('start', 'stop', 'start_date', 'stop_date')
    def _check_closing_date(self):
        for meeting in self:
            if not meeting.allday and meeting.start and meeting.stop and meeting.stop < meeting.start:
                raise ValidationError(
                    _('The ending date and time cannot be earlier than the starting date and time.') + '\n' +
                    _("Meeting '%(name)s' starts '%(start_datetime)s' and ends '%(end_datetime)s'",
                      name=meeting.name,
                      start_datetime=meeting.start,
                      end_datetime=meeting.stop
                    )
                )
            if meeting.allday and meeting.start_date and meeting.stop_date and meeting.stop_date < meeting.start_date:
                raise ValidationError(
                    _('The ending date cannot be earlier than the starting date.') + '\n' +
                    _("Meeting '%(name)s' starts '%(start_datetime)s' and ends '%(end_datetime)s'",
                      name=meeting.name,
                      start_datetime=meeting.start,
                      end_datetime=meeting.stop
                    )
                )

    def _check_organizer_validation_conditions(self, vals_list):
        """ Method for check in the microsoft_calendar module that needs to be
            overridden in appointment.
        """
        return [True] * len(vals_list)

    @api.depends('recurrence_id', 'recurrency')
    def _compute_rrule_type_ui(self):
        defaults = self.env["calendar.recurrence"].default_get(["interval", "rrule_type"])
        for event in self:
            if event.recurrency:
                if event.recurrence_id:
                    event.rrule_type_ui = 'custom' if event.recurrence_id.interval != 1 else (event.recurrence_id.rrule_type)
                else:
                    event.rrule_type_ui = defaults["rrule_type"]

    @api.depends('recurrence_id', 'recurrency', 'rrule_type_ui')
    def _compute_recurrence(self):
        recurrence_fields = self._get_recurrent_fields()
        false_values = {field: False for field in recurrence_fields}  # computes need to set a value
        defaults = self.env['calendar.recurrence'].default_get(recurrence_fields)
        default_rrule_values = self.recurrence_id.default_get(recurrence_fields)
        for event in self:
            if event.recurrency:
                current_rrule = (event.rrule_type if event.rrule_type_ui == "custom" else event.rrule_type_ui)
                event.update(defaults)  # default recurrence values are needed to correctly compute the recurrence params
                event_values = event._get_recurrence_params()
                rrule_values = {
                    field: event.recurrence_id[field]
                    for field in recurrence_fields
                    if event.recurrence_id[field]
                }
                rrule_values = rrule_values or default_rrule_values
                rrule_values['rrule_type'] = current_rrule or rrule_values.get('rrule_type') or defaults['rrule_type']
                event.update({**false_values, **defaults, **event_values, **rrule_values})
            else:
                event.update(false_values)

    @api.depends('description')
    def _compute_display_description(self):
        for event in self:
            event.display_description = not is_html_empty(event.description)

    @api.depends('videocall_source', 'access_token')
    def _compute_videocall_location(self):
        for event in self:
            if event.videocall_source == 'discuss':
                event._set_discuss_videocall_location()

    @api.model
    def _set_videocall_location(self, vals_list):
        for vals in vals_list:
            if not vals.get('videocall_location'):
                continue
            url = url_parse(vals['videocall_location'])
            if url.scheme in ('http', 'https'):
                continue
            # relative url to convert to absolute
            base = url_parse(self.get_base_url())
            vals['videocall_location'] = url.replace(scheme=base.scheme, netloc=base.netloc).to_url()

    @api.depends('videocall_location')
    def _compute_videocall_source(self):
        for event in self:
            if event.videocall_location and self.DISCUSS_ROUTE in event.videocall_location:
                event.videocall_source = 'discuss'
            else:
                event.videocall_source = 'custom'

    def _set_discuss_videocall_location(self):
        """
        This method sets the videocall_location to a discuss route.
        If no access_token exists for this event, we create one.
        Note that recurring events will have different access_tokens.
        This is done by design to prevent users not being able to join a discuss meeting because the base event of the recurrency was deleted.
        """
        if not self.access_token:
            self.access_token = uuid.uuid4().hex
        self.videocall_location = f"{self.get_base_url()}/{self.DISCUSS_ROUTE}/{self.access_token}"

    @api.model
    def get_discuss_videocall_location(self):
        access_token = uuid.uuid4().hex
        return f"{self.get_base_url()}/{self.DISCUSS_ROUTE}/{access_token}"

    # ------------------------------------------------------------
    # CRUD
    # ------------------------------------------------------------

    @api.model_create_multi
    def create(self, vals_list):
        # Prevent sending update notification when _inverse_dates is called
        self = self.with_context(is_calendar_event_new=True)
        defaults = self.default_get(['activity_ids', 'res_model_id', 'res_id', 'user_id', 'res_model', 'partner_ids'])

        vals_list = [  # Else bug with quick_create when we are filter on an other user
            dict(vals, user_id=defaults.get('user_id', self.env.user.id)) if not 'user_id' in vals else vals
            for vals in vals_list
        ]
        meeting_activity_type = self.env['mail.activity.type'].search([('category', '=', 'meeting')], limit=1)
        # get list of models ids and filter out None values directly
        model_ids = list(filter(None, {values.get('res_model_id', defaults.get('res_model_id')) for values in vals_list}))
        model_name = defaults.get('res_model')
        valid_activity_model_ids = model_name and self.env[model_name].sudo().browse(model_ids).filtered(lambda m: 'activity_ids' in m).ids or []

        # if user is creating an event for an activity that already has one, create a second activity
        existing_event = False
        orig_activity_ids = self.env['mail.activity'].browse(self._context.get('orig_activity_ids', []))
        if len(orig_activity_ids) == 1:
            existing_event = orig_activity_ids.calendar_event_id
            if existing_event and orig_activity_ids.activity_type_id.category == 'meeting':
                meeting_activity_type = orig_activity_ids.activity_type_id

        if meeting_activity_type and (not defaults.get('activity_ids') or existing_event):
            for values in vals_list:
                # created from calendar: try to create an activity on the related record
                if values.get('activity_ids'):
                    continue
                res_model_id = values.get('res_model_id', defaults.get('res_model_id'))
                res_id = values.get('res_id', defaults.get('res_id'))
                user_id = values.get('user_id', defaults.get('user_id'))
                if not res_model_id or not res_id:
                    continue
                if res_model_id not in valid_activity_model_ids:
                    continue
                activity_vals = {
                    'res_model_id': res_model_id,
                    'res_id': res_id,
                    'activity_type_id': meeting_activity_type.id,
                }
                if user_id:
                    activity_vals['user_id'] = user_id
                values['activity_ids'] = [(0, 0, activity_vals)]
        self._set_videocall_location(vals_list)

        # Add commands to create attendees from partners (if present) if no attendee command
        # is already given (coming from Google event for example).
        # Automatically add the current partner when creating an event if there is none (happens when we quickcreate an event)
        default_partners_ids = defaults.get('partner_ids') or ([(4, self.env.user.partner_id.id)])
        vals_list = [
            dict(vals, attendee_ids=self._attendees_values(vals.get('partner_ids', default_partners_ids)))
            if not vals.get('attendee_ids')
            else vals
            for vals in vals_list
        ]
        recurrence_fields = self._get_recurrent_fields()
        recurring_vals = [vals for vals in vals_list if vals.get('recurrency')]
        other_vals = [vals for vals in vals_list if not vals.get('recurrency')]
        events = super().create(other_vals)

        for vals in recurring_vals:
            vals['follow_recurrence'] = True
        recurring_events = super().create(recurring_vals)
        events += recurring_events

        for event, vals in zip(recurring_events, recurring_vals):
            recurrence_values = {field: vals.pop(field) for field in recurrence_fields if field in vals}
            if vals.get('recurrency'):
                detached_events = event._apply_recurrence_values(recurrence_values)
                detached_events.active = False

        events.filtered(lambda event: event.start > fields.Datetime.now()).attendee_ids._send_invitation_emails()

        events._sync_activities(fields={f for vals in vals_list for f in vals.keys()})
        if not self.env.context.get('dont_notify'):
            alarm_events = self.env['calendar.event']
            for event, values in zip(events, vals_list):
                if values.get('allday'):
                    # All day events will trigger the _inverse_date method which will create the trigger.
                    continue
                alarm_events |= event
            recurring_events = alarm_events.filtered('recurrence_id')
            recurring_events.recurrence_id._setup_alarms()
            (alarm_events - recurring_events)._setup_alarms()
        return events.with_context(is_calendar_event_new=False)

    def _compute_field_value(self, field):
        if field.compute_sudo:
            return super(Meeting, self.with_context(prefetch_fields=False))._compute_field_value(field)
        return super()._compute_field_value(field)

    def _fetch_query(self, query, fields):
        if self.env.su:
            return super()._fetch_query(query, fields)

        public_fnames = self._get_public_fields()
        private_fields = [field for field in fields if field.name not in public_fnames]
        if not private_fields:
            return super()._fetch_query(query, fields)

        fields_to_fetch = list(fields) + [self._fields[name] for name in ('privacy', 'user_id', 'partner_ids')]
        events = super()._fetch_query(query, fields_to_fetch)

        # determine private events to which the user does not participate
        others_private_events = events.filtered(lambda ev: ev._check_private_event_conditions())
        if not others_private_events:
            return events

        private_fields.append(self._fields['partner_ids'])
        for field in private_fields:
            replacement = field.convert_to_cache(
                _('Busy') if field.name == 'name' else False,
                others_private_events)
            self.env.cache.update(others_private_events, field, repeat(replacement))

        return events

    def write(self, values):
        detached_events = self.env['calendar.event']
        recurrence_update_setting = values.pop('recurrence_update', None)
        update_recurrence = recurrence_update_setting in ('all_events', 'future_events') and len(self) == 1 and self.recurrence_id
        break_recurrence = values.get('recurrency') is False

        if any(vals in self._get_recurrent_fields() for vals in values) and not (update_recurrence or values.get('recurrency')):
            raise UserError(_('Unable to save the recurrence with "This Event"'))

        update_alarms = False
        update_time = False
        self._set_videocall_location([values])
        if 'partner_ids' in values:
            values['attendee_ids'] = self._attendees_values(values['partner_ids'])
            update_alarms = True
            if self.videocall_channel_id:
                new_partner_ids = []
                for command in values['partner_ids']:
                    if command[0] == Command.LINK:
                        new_partner_ids.append(command[1])
                    elif command[0] == Command.SET:
                        new_partner_ids.extend(command[2])
                self.videocall_channel_id.add_members(new_partner_ids)

        time_fields = self.env['calendar.event']._get_time_fields()
        if any([values.get(key) for key in time_fields]):
            update_alarms = True
            update_time = True
        if 'alarm_ids' in values:
            update_alarms = True

        if (not recurrence_update_setting or recurrence_update_setting == 'self_only' and len(self) == 1) and 'follow_recurrence' not in values:
            if any({field: values.get(field) for field in time_fields if field in values}):
                values['follow_recurrence'] = False

        previous_attendees = self.attendee_ids

        recurrence_values = {field: values.pop(field) for field in self._get_recurrent_fields() if field in values}
        future_edge_case = recurrence_update_setting == 'future_events' and self == self.recurrence_id.base_event_id
        if update_recurrence:
            if break_recurrence:
                # Update this event
                detached_events |= self._break_recurrence(future=recurrence_update_setting == 'future_events')
            else:
                time_values = {field: values.pop(field) for field in time_fields if field in values}
                if 'access_token' in values:
                    values.pop('access_token')  # prevents copying access_token to other events in recurrency
                if recurrence_update_setting == 'all_events' or future_edge_case:
                    # Update all events: we create a new reccurrence and dismiss the existing events
                    self._rewrite_recurrence(values, time_values, recurrence_values)
                else:
                    # Update future events: trim recurrence, delete remaining events except base event and recreate it
                    # All the recurrent events processing is done within the following method
                    self._update_future_events(values, time_values, recurrence_values)
        else:
            super().write(values)
            self._sync_activities(fields=values.keys())

        # We reapply recurrence for future events and when we add a rrule and 'recurrency' == True on the event
        if recurrence_update_setting not in ['self_only', 'all_events'] and not future_edge_case and not break_recurrence:
            detached_events |= self._apply_recurrence_values(recurrence_values, future=recurrence_update_setting == 'future_events')

        (detached_events & self).active = False
        (detached_events - self).with_context(archive_on_error=True).unlink()

        # Notify attendees if there is an alarm on the modified event, or if there was an alarm
        # that has just been removed, as it might have changed their next event notification
        if not self.env.context.get('dont_notify') and update_alarms:
            self.recurrence_id._setup_alarms(recurrence_update=True)
            if not self.recurrence_id:
                self._setup_alarms()
        attendee_update_events = self.filtered(lambda ev: ev.user_id and ev.user_id != self.env.user)
        if update_time and attendee_update_events:
            # Another user update the event time fields. It should not be auto accepted for the organizer.
            # This prevent weird behavior when a user modified future events time fields and
            # the base event of a recurrence is accepted by the organizer but not the following events
            attendee_update_events.attendee_ids.filtered(lambda att: self.user_id.partner_id == att.partner_id).write({'state': 'needsAction'})

        current_attendees = self.filtered('active').attendee_ids
        if 'partner_ids' in values:
            # we send to all partners and not only the new ones
            (current_attendees - previous_attendees)._send_mail_to_attendees(
                self.env.ref('calendar.calendar_template_meeting_invitation', raise_if_not_found=False)
            )
        if not self.env.context.get('is_calendar_event_new') and 'start' in values:
            start_date = fields.Datetime.to_datetime(values.get('start'))
            # Only notify on future events
            if start_date and start_date >= fields.Datetime.now():
                (current_attendees & previous_attendees).with_context(
                    calendar_template_ignore_recurrence=not update_recurrence
                )._send_mail_to_attendees(
                    self.env.ref('calendar.calendar_template_meeting_changedate', raise_if_not_found=False)
                )

        # Change base event when the main base event is archived. If it isn't done when trying to modify
        # all events of the recurrence an error can be thrown or all the recurrence can be deleted.
        if values.get("active") is False:
            recurrences = self.env["calendar.recurrence"].search([
                ('base_event_id', 'in', self.ids)
            ])
            recurrences._select_new_base_event()

        return True

    def _check_private_event_conditions(self):
        """
        Checks if the event is private, returning True if the conditions match and False otherwise.
        The event is private if it is explicetely defined and the user is neither the organizer or a partner of it.
        """
        self.ensure_one()
        event_is_private = self.privacy == 'private'
        user_is_not_partner = self.user_id.id != self.env.uid and self.env.user.partner_id not in self.partner_ids
        return event_is_private and user_is_not_partner

    @api.depends('privacy', 'user_id')
    def _compute_display_name(self):
        """ Hide private events' name for events which don't belong to the current user
        """
        hidden = self.filtered(lambda event: event._check_private_event_conditions())
        hidden.display_name = _('Busy')
        super(Meeting, self - hidden)._compute_display_name()

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        groupby = [groupby] if isinstance(groupby, str) else groupby
        fields_aggregates = [
            field_name for field_name in (fields or list(self._fields))
            if ':' in field_name or (field_name in self and self._fields[field_name].group_operator)
        ]
        grouped_fields = {group_field.split(':')[0] for group_field in groupby + fields_aggregates}
        private_fields = grouped_fields - self._get_public_fields()
        if not self.env.su and private_fields:
            # display public and confidential events
            domain = AND([domain, ['|', ('privacy', '!=', 'private'), ('user_id', '=', self.env.user.id)]])
            return super(Meeting, self).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)
        return super(Meeting, self).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

    def unlink(self):
        if not self:
            return super().unlink()

        # Get concerned attendees to notify them if there is an alarm on the unlinked events,
        # as it might have changed their next event notification
        events = self.filtered_domain([('alarm_ids', '!=', False)])
        partner_ids = events.mapped('partner_ids').ids

        # don't forget to update recurrences if there are some base events in the set to unlink,
        # but after having removed the events ;-)
        recurrences = self.env["calendar.recurrence"].search([
            ('base_event_id', 'in', [e.id for e in self])
        ])

        result = super().unlink()

        if recurrences:
            recurrences._select_new_base_event()

        # Notify the concerned attendees (must be done after removing the events)
        self.env['calendar.alarm_manager']._notify_next_alarm(partner_ids)
        return result

    def copy(self, default=None):
        """When an event is copied, the attendees should be recreated to avoid sharing the same attendee records
         between copies
         """
        self.ensure_one()
        if not default:
            default = {}
        # We need to make sure that the attendee_ids are recreated with new ids to avoid sharing attendees between events
        # The copy should not have the same attendee status than the original event
        default.update(partner_ids=[Command.set([])], attendee_ids=[Command.set([])])
        copied_event = super().copy(default)
        copied_event.write({'partner_ids': [(Command.set(self.partner_ids.ids))]})
        return copied_event

    @api.model
    def _get_mail_message_access(self, res_ids, operation, model_name=None):
        if operation == 'read' and (not model_name or model_name == 'event.event'):
            for event in self.browse(res_ids):
                if event.privacy == "private" and self.env.user.partner_id not in event.attendee_ids.partner_id:
                    return 'write'
        return super()._get_mail_message_access(res_ids, operation, model_name=model_name)

    def _attendees_values(self, partner_commands):
        """
        :param partner_commands: ORM commands for partner_id field (0 and 1 commands not supported)
        :return: associated attendee_ids ORM commands
        """
        attendee_commands = []

        removed_partner_ids = []
        added_partner_ids = []

        # if commands are just integers, assume they are ids with the intent to `Command.set`
        if partner_commands and isinstance(partner_commands[0], int):
            partner_commands = [Command.set(partner_commands)]

        for command in partner_commands:
            op = command[0]
            if op in (2, 3, Command.delete, Command.unlink):  # Remove partner
                removed_partner_ids += [command[1]]
            elif op in (6, Command.set):  # Replace all
                removed_partner_ids += set(self.partner_ids.ids) - set(command[2])  # Don't recreate attendee if partner already attend the event
                added_partner_ids += set(command[2]) - set(self.partner_ids.ids)
            elif op in (4, Command.link):
                added_partner_ids += [command[1]] if command[1] not in self.partner_ids.ids else []
            # commands 0 and 1 not supported

        if not self:
            attendees_to_unlink = self.env['calendar.attendee']
        else:
            attendees_to_unlink = self.env['calendar.attendee'].search([
                ('event_id', 'in', self.ids),
                ('partner_id', 'in', removed_partner_ids),
            ])
        attendee_commands += [[2, attendee.id] for attendee in attendees_to_unlink]  # Removes and delete

        attendee_commands += [
            [0, 0, dict(partner_id=partner_id)]
            for partner_id in added_partner_ids
        ]
        return attendee_commands

    def _create_videocall_channel(self):
        if self.recurrency:
            # check if any of the events have videocall_channel_id, if not create one
            event_with_channel = self.env['calendar.event'].search([
                ('recurrence_id', '=', self.recurrence_id.id),
                ('videocall_channel_id', '!=', False)
            ], limit=1)
            if event_with_channel:
                self.videocall_channel_id = event_with_channel.videocall_channel_id
                return
        self.videocall_channel_id = self._create_videocall_channel_id(self.name, self.partner_ids.ids)
        self.videocall_channel_id.channel_change_description(self.recurrence_id.name if self.recurrency else self.display_time)

    def _create_videocall_channel_id(self, name, partner_ids):
        videocall_channel = self.env['discuss.channel'].create_group(partner_ids, default_display_mode='video_full_screen', name=name)
        # if recurrent event, set channel to all other records of the same recurrency
        if self.recurrency:
            recurrent_events_without_channel = self.env['calendar.event'].search([
                ('recurrence_id', '=', self.recurrence_id.id), ('videocall_channel_id', '=', False)
            ])
            recurrent_events_without_channel.videocall_channel_id = videocall_channel
        return videocall_channel

    # ------------------------------------------------------------
    # ACTIONS
    # ------------------------------------------------------------

    # dummy method. this method is intercepted in the frontend and the value is set locally
    def set_discuss_videocall_location(self):
        return True

    # dummy method. this method is intercepted in the frontend and the value is set locally
    def clear_videocall_location(self):
        return True

    def action_open_calendar_event(self):
        if self.res_model and self.res_id:
            return self.env[self.res_model].browse(self.res_id).get_formview_action()
        return False

    def action_sendmail(self):
        email = self.env.user.email
        if email:
            for meeting in self:
                meeting.attendee_ids._send_mail_to_attendees(
                    self.env.ref('calendar.calendar_template_meeting_invitation', raise_if_not_found=False)
                )
        return True

    def action_open_composer(self):
        if not self.partner_ids:
            raise UserError(_("There are no attendees on these events"))
        template_id = self.env['ir.model.data']._xmlid_to_res_id('calendar.calendar_template_meeting_update', raise_if_not_found=False)
        # The mail is sent with datetime corresponding to the sending user TZ
        default_composition_mode = self.env.context.get('default_composition_mode', self.env.context.get('composition_mode', 'comment'))
        compose_ctx = dict(
            default_composition_mode=default_composition_mode,
            default_model='calendar.event',
            default_res_ids=self.ids,
            default_template_id=template_id,
            default_partner_ids=self.partner_ids.ids,
            mail_tz=self.env.user.tz,
        )
        return {
            'type': 'ir.actions.act_window',
            'name': _('Contact Attendees'),
            'view_mode': 'form',
            'res_model': 'mail.compose.message',
            'views': [(False, 'form')],
            'view_id': False,
            'target': 'new',
            'context': compose_ctx,
        }

    def action_join_video_call(self):
        return {
            'type': 'ir.actions.act_url',
            'url': self.videocall_location,
            'target': 'new'
        }

    def action_join_meeting(self, partner_id):
        """ Method used when an existing user wants to join
        """
        self.ensure_one()
        partner = self.env['res.partner'].browse(partner_id)
        if partner not in self.partner_ids:
            self.write({'partner_ids': [(4, partner.id)]})

    def action_mass_deletion(self, recurrence_update_setting):
        self.ensure_one()
        if recurrence_update_setting == 'all_events':
            events = self.recurrence_id.calendar_event_ids
            self.recurrence_id.unlink()
            events.unlink()
        elif recurrence_update_setting == 'future_events':
            future_events = self.recurrence_id.calendar_event_ids.filtered(lambda ev: ev.start >= self.start)
            future_events.unlink()

    def action_mass_archive(self, recurrence_update_setting):
        """
        The aim of this action purpose is to be called from sync calendar module when mass deletion is not possible.
        """
        self.ensure_one()
        if recurrence_update_setting == 'all_events':
            self.recurrence_id.calendar_event_ids.write(self._get_archive_values())
        elif recurrence_update_setting == 'future_events':
            detached_events = self.recurrence_id._stop_at(self)
            detached_events.write(self._get_archive_values())
        elif recurrence_update_setting == 'self_only':
            self.write({
                'active': False,
                'recurrence_update': 'self_only'
            })
            if len(self.recurrence_id.calendar_event_ids) == 0:
                self.recurrence_id.unlink()
            elif self == self.recurrence_id.base_event_id:
                self.recurrence_id._select_new_base_event()

    # ------------------------------------------------------------
    # MAILING
    # ------------------------------------------------------------

    def _get_attendee_emails(self):
        """ Get comma-separated attendee email addresses. """
        self.ensure_one()
        return ",".join([e for e in self.attendee_ids.mapped("email") if e])

    def _get_mail_tz(self):
        self.ensure_one()
        return self.event_tz or self.env.user.tz

    def _sync_activities(self, fields):
        # update activities
        for event in self:
            if event.activity_ids:
                activity_values = {}
                if 'name' in fields:
                    activity_values['summary'] = event.name
                if 'description' in fields:
                    activity_values['note'] = event.description
                if 'start' in fields:
                    # self.start is a datetime UTC *only when the event is not allday*
                    # activty.date_deadline is a date (No TZ, but should represent the day in which the user's TZ is)
                    # See 72254129dbaeae58d0a2055cba4e4a82cde495b7 for the same issue, but elsewhere
                    deadline = event.start
                    user_tz = self.env.context.get('tz')
                    if user_tz and not event.allday:
                        deadline = pytz.utc.localize(deadline)
                        deadline = deadline.astimezone(pytz.timezone(user_tz))
                    activity_values['date_deadline'] = deadline.date()
                if 'user_id' in fields:
                    activity_values['user_id'] = event.user_id.id
                if activity_values.keys():
                    event.activity_ids.write(activity_values)

    # ------------------------------------------------------------
    # ALARMS
    # ------------------------------------------------------------

    def _get_trigger_alarm_types(self):
        return ['email']

    def _setup_alarms(self):
        """ Schedule cron triggers for future events """
        cron = self.env.ref('calendar.ir_cron_scheduler_alarm').sudo()
        alarm_types = self._get_trigger_alarm_types()
        events_to_notify = self.env['calendar.event']
        triggers_by_events = {}
        for event in self:
            existing_trigger = event.recurrence_id.trigger_id
            for alarm in (alarm for alarm in event.alarm_ids if alarm.alarm_type in alarm_types):
                at = event.start - timedelta(minutes=alarm.duration_minutes)
                create_trigger = not existing_trigger or existing_trigger and existing_trigger.call_at != at
                if create_trigger and (not cron.lastcall or at > cron.lastcall):
                    # Don't trigger for past alarms, they would be skipped by design
                    trigger = cron._trigger(at=at)
                    triggers_by_events[event.id] = trigger.id
            if any(alarm.alarm_type == 'notification' for alarm in event.alarm_ids):
                # filter events before notifying attendees through calendar_alarm_manager
                events_to_notify |= event.filtered(lambda ev: ev.alarm_ids and ev.stop >= fields.Datetime.now())
        if events_to_notify:
            self.env['calendar.alarm_manager']._notify_next_alarm(events_to_notify.partner_ids.ids)
        return triggers_by_events

    def get_next_alarm_date(self, events_by_alarm):
        self.ensure_one()
        now = fields.datetime.now()
        sorted_alarms = self.alarm_ids.sorted("duration_minutes")
        triggered_alarms = sorted_alarms.filtered(lambda alarm: alarm.id in events_by_alarm)[0]
        event_has_future_alarms = sorted_alarms[0] != triggered_alarms
        next_date = None
        if self.recurrence_id.trigger_id and self.recurrence_id.trigger_id.call_at <= now:
            next_date = self.start - timedelta(minutes=sorted_alarms[0].duration_minutes) \
                if event_has_future_alarms \
                else self.start
        # For recurrent events, when there is no next_date and no trigger in the recurence, set the next
        # date as the date of the next event. This keeps the single alarm alive in the recurrence.
        recurrence_has_no_trigger = self.recurrence_id and not self.recurrence_id.trigger_id
        if recurrence_has_no_trigger and not next_date and len(sorted_alarms) > 0:
            future_recurrent_events = self.recurrence_id.calendar_event_ids.filtered(lambda ev: ev.start > self.start)
            if future_recurrent_events:
                # The next event (minus the alarm duration) will be the next date.
                next_recurrent_event = future_recurrent_events.sorted("start")[0]
                next_date = next_recurrent_event.start - timedelta(minutes=sorted_alarms[0].duration_minutes)
        return next_date

    # ------------------------------------------------------------
    # RECURRENCY
    # ------------------------------------------------------------

    def _apply_recurrence_values(self, values, future=True):
        """Apply the new recurrence rules in `values`. Create a recurrence if it does not exist
        and create all missing events according to the rrule.
        If the changes are applied to future
        events only, a new recurrence is created with the updated rrule.

        :param values: new recurrence values to apply
        :param future: rrule values are applied to future events only if True.
                       Rrule changes are applied to all events in the recurrence otherwise.
                       (ignored if no recurrence exists yet).
        :return: events detached from the recurrence
        """
        if not values:
            return self.browse()
        recurrence_vals = []
        to_update = self.env['calendar.recurrence']
        for event in self:
            if not event.recurrence_id:
                recurrence_vals += [dict(values, base_event_id=event.id, calendar_event_ids=[(4, event.id)])]
            elif future:
                to_update |= event.recurrence_id._split_from(event, values)
        self.write({'recurrency': True, 'follow_recurrence': True})
        to_update |= self.env['calendar.recurrence'].create(recurrence_vals)
        return to_update._apply_recurrence()

    def _get_recurrence_params(self):
        if not self:
            return {}
        event_date = self._get_start_date()
        weekday_field_name = weekday_to_field(event_date.weekday())
        return {
            weekday_field_name: True,
            'weekday': weekday_field_name.upper(),
            'byday': str(get_weekday_occurence(event_date)),
            'day': event_date.day,
        }

    @api.model
    def _get_recurrence_params_by_date(self, event_date):
        """ Return the recurrence parameters from a date object. """
        weekday_field_name = weekday_to_field(event_date.weekday())
        return {
            weekday_field_name: True,
            'weekday': weekday_field_name.upper(),
            'byday': str(get_weekday_occurence(event_date)),
            'day': event_date.day,
        }

    def _split_recurrence(self, time_values):
        """Apply time changes to events and update the recurrence accordingly.

        :return: detached events
        """
        self.ensure_one()
        if not time_values:
            return self.browse()
        if self.follow_recurrence and self.recurrency:
            previous_week_day_field = weekday_to_field(self._get_start_date().weekday())
        else:
            # When we try to change recurrence values of an event not following the recurrence, we get the parameters from
            # the base_event
            previous_week_day_field = weekday_to_field(self.recurrence_id.base_event_id._get_start_date().weekday())
        self.write(time_values)
        return self._apply_recurrence_values({
            previous_week_day_field: False,
            **self._get_recurrence_params(),
        }, future=True)

    def _break_recurrence(self, future=True):
        """Breaks the event's recurrence.
        Stop the recurrence at the current event if `future` is True, leaving past events in the recurrence.
        If `future` is False, all events in the recurrence are detached and the recurrence itself is unlinked.
        :return: detached events excluding the current events
        """
        recurrences_to_unlink = self.env['calendar.recurrence']
        detached_events = self.env['calendar.event']
        for event in self:
            recurrence = event.recurrence_id
            if future:
                detached_events |= recurrence._stop_at(event)
            else:
                detached_events |= recurrence.calendar_event_ids
                recurrence.calendar_event_ids.recurrence_id = False
                recurrences_to_unlink |= recurrence
        recurrences_to_unlink.with_context(archive_on_error=True).unlink()
        return detached_events - self

    def _get_time_update_dict(self, base_event, time_values):
        """ Return the update dictionary for shifting the base_event's time to the new date. """
        if not base_event:
            raise UserError(_("You can't update a recurrence without base event."))
        [base_time_values] = base_event.read(['start', 'stop', 'allday'])
        update_dict = {}
        start_update = fields.Datetime.to_datetime(time_values.get('start'))
        stop_update = fields.Datetime.to_datetime(time_values.get('stop'))
        # Convert the base_event_id hours according to new values: time shift
        if start_update or stop_update:
            if start_update:
                start = base_time_values['start'] + (start_update - self.start)
                stop = base_time_values['stop'] + (start_update - self.start)
                start_date = base_time_values['start'].date() + (start_update.date() - self.start.date())
                stop_date = base_time_values['stop'].date() + (start_update.date() - self.start.date())
                update_dict.update({'start': start, 'start_date': start_date, 'stop': stop, 'stop_date': stop_date})
            if stop_update:
                if not start_update:
                    # Apply the same shift for start
                    start = base_time_values['start'] + (stop_update - self.stop)
                    start_date = base_time_values['start'].date() + (stop_update.date() - self.stop.date())
                    update_dict.update({'start': start, 'start_date': start_date})
                stop = base_time_values['stop'] + (stop_update - self.stop)
                stop_date = base_time_values['stop'].date() + (stop_update.date() - self.stop.date())
                update_dict.update({'stop': stop, 'stop_date': stop_date})
        return update_dict

    @api.model
    def _get_archive_values(self):
        """ Return parameters for archiving events in calendar module. """
        return {'active': False}

    @api.model
    def _check_values_to_sync(self, values):
        """ Method to be overriden: return candidate values to be synced within rewrite_recurrence function scope. """
        return False

    @api.model
    def _get_update_future_events_values(self):
        """ Return parameters for updating future events within _update_future_events function scope. """
        return {}

    @api.model
    def _get_remove_sync_id_values(self):
        """ Return parameters for removing event synchronization id within _update_future_events function scope. """
        return {}

    def _get_updated_recurrence_values(self, new_start_date):
        """ Copy values from current recurrence and update the start date weekday. """
        [previous_recurrence_values] = self.recurrence_id.copy_data()
        if self.start.weekday() != new_start_date.weekday():
            previous_recurrence_values.pop(weekday_to_field(self.start.weekday()), None)
        return previous_recurrence_values

    def _update_future_events(self, values, time_values, recurrence_values):
        """
            Trim the current recurrence detaching the occurrences after current event,
            deactivate the detached events except for the updated event and apply recurrence values.
        """
        self.ensure_one()
        base_event = self
        update_dict = self._get_time_update_dict(base_event, time_values)
        time_values.update(update_dict)
        # Get base values from the previous recurrence and update the start date weekday field.
        start_date = time_values['start'].date() if 'start' in time_values else self.start.date()
        previous_recurrence_values = self._get_updated_recurrence_values(start_date)

        # Trim previous recurrence at current event, deleting following events except for the updated event.
        detached_events_split = self.recurrence_id._stop_at(self)
        (detached_events_split - self).write({'active': False, **self._get_remove_sync_id_values()})

        # Update the current event with the new recurrence information.
        if values or time_values:
            self.write({
                **time_values, **values,
                **self._get_remove_sync_id_values(),
                **self._get_update_future_events_values()
            })

        # Combine parameters from previous recurrence with the new recurrence parameters.
        new_values = {
            **previous_recurrence_values,
            **self._get_recurrence_params_by_date(start_date),
            **recurrence_values,
            'count': recurrence_values.get('count', 0) or len(detached_events_split)
        }
        new_values.pop('rrule', None)

        # Generate the new recurrence by patching the updated event and return an empty list.
        self._apply_recurrence_values(new_values)

    def _rewrite_recurrence(self, values, time_values, recurrence_values):
        """ Delete the current recurrence, reactivate base event and apply updated recurrence values. """
        self.ensure_one()
        base_event = self.recurrence_id.base_event_id or self.recurrence_id._get_first_event(include_outliers=False)
        update_dict = self._get_time_update_dict(base_event, time_values)
        time_values.update(update_dict)

        if self._check_values_to_sync(values) or time_values or recurrence_values:
            # Get base values from the previous recurrence and update the start date weekday field.
            start_date = time_values['start'].date() if 'start' in time_values else self.start.date()
            old_recurrence_values = self._get_updated_recurrence_values(start_date)

            # Archive all events and delete recurrence, reactivate base event and apply updated values.
            base_event.action_mass_archive("all_events")
            base_event.recurrence_id.unlink()
            base_event.write({
                'active': True,
                'recurrence_id': False,
                **values, **time_values
            })

            # Combine parameters from previous recurrence with the new recurrence parameters.
            new_values = {
                **old_recurrence_values,
                **base_event._get_recurrence_params(),
                **recurrence_values,
            }
            new_values.pop('rrule', None)

            # Patch base event with updated recurrence parameters: this will recreate the recurrence.
            detached_events = base_event._apply_recurrence_values(new_values)
            detached_events.write({'active': False})
        else:
            # Write on all events. Carefull, it could trigger a lot of noise to Google/Microsoft...
            self.recurrence_id._write_events(values)

    # ------------------------------------------------------------
    # MANAGEMENT
    # ------------------------------------------------------------

    def change_attendee_status(self, status, recurrence_update_setting):
        self.ensure_one()
        if recurrence_update_setting == 'all_events':
            events = self.recurrence_id.calendar_event_ids
        elif recurrence_update_setting == 'future_events':
            events = self.recurrence_id.calendar_event_ids.filtered(lambda ev: ev.start >= self.start)
        else:
            events = self
        attendee = events.attendee_ids.filtered(lambda x: x.partner_id == self.env.user.partner_id)
        if status == 'accepted':
            return attendee.do_accept()
        if status == 'declined':
            return attendee.do_decline()
        return attendee.do_tentative()

    def find_partner_customer(self):
        self.ensure_one()
        return next(
            (attendee.partner_id for attendee in self.attendee_ids.sorted('create_date')
             if attendee.partner_id != self.user_id.partner_id),
            self.env['calendar.attendee']
        )

    # ------------------------------------------------------------
    # TOOLS
    # ------------------------------------------------------------

    def _get_start_date(self):
        """Return the event starting date in the event's timezone.
        If no starting time is assigned (yet), return today as default
        :return: date
        """
        if not self.start:
            return fields.Date.today()
        if self.recurrency and self.event_tz:
            tz = pytz.timezone(self.event_tz)
            # Ensure that all day events date are not calculated around midnight. TZ shift would potentially return bad date
            start = self.start if not self.allday else self.start.replace(hour=12)
            return pytz.utc.localize(start).astimezone(tz).date()
        return self.start.date()

    def _range(self):
        self.ensure_one()
        return (self.start, self.stop)

    def get_display_time_tz(self, tz=False):
        """ get the display_time of the meeting, forcing the timezone. This method is called from email template, to not use sudo(). """
        self.ensure_one()
        if tz:
            self = self.with_context(tz=tz)
        return self._get_display_time(self.start, self.stop, self.duration, self.allday)

    def _get_ics_file(self):
        """ Returns iCalendar file for the event invitation.
            :returns a dict of .ics file content for each meeting
        """
        result = {}

        def ics_datetime(idate, allday=False):
            if idate:
                if allday:
                    return idate
                return idate.replace(tzinfo=pytz.timezone('UTC'))
            return False

        if not vobject:
            return result

        for meeting in self:
            cal = vobject.iCalendar()
            event = cal.add('vevent')

            if not meeting.start or not meeting.stop:
                raise UserError(_("First you have to specify the date of the invitation."))
            event.add('created').value = ics_datetime(fields.Datetime.now())
            event.add('dtstart').value = ics_datetime(meeting.start, meeting.allday)
            event.add('dtend').value = ics_datetime(meeting.stop, meeting.allday)
            event.add('summary').value = meeting._get_customer_summary()
            description = meeting._get_customer_description()
            if description:
                event.add('description').value = description
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

            # Add "organizer" field if email available
            if meeting.partner_id.email:
                organizer = event.add('organizer')
                organizer.value = u'MAILTO:' + meeting.partner_id.email
                if meeting.partner_id.name:
                    organizer.params['CN'] = [meeting.partner_id.display_name.replace('\"', '\'')]

            result[meeting.id] = cal.serialize().encode('utf-8')

        return result

    def _get_customer_description(self):
        """:return (str): The description to include in calendar exports"""
        return html2plaintext(self.description) if self.description else ''

    def _get_customer_summary(self):
        """:return (str): The summary to include in calendar exports"""
        return self.name or ''

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
            display_time = _("All Day, %(day)s", day=date_str)
        elif zduration < 24:
            duration = date + timedelta(minutes=round(zduration*60))
            duration_time = to_text(duration.strftime(format_time))
            display_time = _(
                u"%(day)s at (%(start)s To %(end)s) (%(timezone)s)",
                day=date_str,
                start=time_str,
                end=duration_time,
                timezone=timezone,
            )
        else:
            dd_date = to_text(date_deadline.strftime(format_date))
            dd_time = to_text(date_deadline.strftime(format_time))
            display_time = _(
                u"%(date_start)s at %(time_start)s To\n %(date_end)s at %(time_end)s (%(timezone)s)",
                date_start=date_str,
                time_start=time_str,
                date_end=dd_date,
                time_end=dd_time,
                timezone=timezone,
            )
        return display_time

    def _get_duration(self, start, stop):
        """ Get the duration value between the 2 given dates. """
        if not start or not stop:
            return 0
        duration = (stop - start).total_seconds() / 3600
        return round(duration, 2)

    @api.model
    def _get_date_formats(self):
        """ get current date and time format, according to the context lang
            :return: a tuple with (format date, format time)
        """
        lang = get_lang(self.env)
        return (lang.date_format, lang.time_format)

    @api.model
    def _get_recurrent_fields(self):
        return {'byday', 'until', 'rrule_type', 'month_by', 'event_tz', 'rrule',
                'interval', 'count', 'end_type', 'mon', 'tue', 'wed', 'thu', 'fri', 'sat',
                'sun', 'day', 'weekday'}

    @api.model
    def _get_time_fields(self):
        return {'start', 'stop', 'start_date', 'stop_date'}

    @api.model
    def _get_custom_fields(self):
        all_fields = self.fields_get(attributes=['manual'])
        return {fname for fname in all_fields if all_fields[fname]['manual']}

    @api.model
    def _get_public_fields(self):
        return self._get_recurrent_fields() | self._get_time_fields() | self._get_custom_fields() | {
            'id', 'active', 'allday',
            'duration', 'user_id', 'interval', 'partner_id',
            'count', 'rrule', 'recurrence_id', 'show_as', 'privacy'}

```

## File: models\calendar_event_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint

from odoo import fields, models


class MeetingType(models.Model):

    _name = 'calendar.event.type'
    _description = 'Event Meeting Type'

    def _default_color(self):
        return randint(1, 11)

    name = fields.Char('Name', required=True)
    color = fields.Integer('Color', default=_default_color)

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Tag name already exists!"),
    ]

```

## File: models\calendar_filter.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Contacts(models.Model):
    _name = 'calendar.filters'
    _description = 'Calendar Filters'

    user_id = fields.Many2one('res.users', 'Me', required=True, default=lambda self: self.env.user, index=True, ondelete='cascade')
    partner_id = fields.Many2one('res.partner', 'Employee', required=True, index=True)
    active = fields.Boolean('Active', default=True)
    partner_checked = fields.Boolean('Checked', default=True)  # used to know if the partner is checked in the filter of the calendar view for the user_id.

    _sql_constraints = [
        ('user_id_partner_id_unique', 'UNIQUE(user_id, partner_id)', 'A user cannot have the same contact twice.')
    ]

    @api.model
    def unlink_from_partner_id(self, partner_id):
        return self.search([('partner_id', '=', partner_id)]).unlink()

```

## File: models\calendar_recurrence.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, time
import pytz
import re

from dateutil import rrule
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools.misc import clean_context

from odoo.addons.base.models.res_partner import _tz_get


MAX_RECURRENT_EVENT = 720

SELECT_FREQ_TO_RRULE = {
    'daily': rrule.DAILY,
    'weekly': rrule.WEEKLY,
    'monthly': rrule.MONTHLY,
    'yearly': rrule.YEARLY,
}

RRULE_FREQ_TO_SELECT = {
    rrule.DAILY: 'daily',
    rrule.WEEKLY: 'weekly',
    rrule.MONTHLY: 'monthly',
    rrule.YEARLY: 'yearly',
}

RRULE_WEEKDAY_TO_FIELD = {
    rrule.MO.weekday: 'mon',
    rrule.TU.weekday: 'tue',
    rrule.WE.weekday: 'wed',
    rrule.TH.weekday: 'thu',
    rrule.FR.weekday: 'fri',
    rrule.SA.weekday: 'sat',
    rrule.SU.weekday: 'sun',
}

RRULE_WEEKDAYS = {'SUN': 'SU', 'MON': 'MO', 'TUE': 'TU', 'WED': 'WE', 'THU': 'TH', 'FRI': 'FR', 'SAT': 'SA'}

RRULE_TYPE_SELECTION = [
    ('daily', 'Days'),
    ('weekly', 'Weeks'),
    ('monthly', 'Months'),
    ('yearly', 'Years'),
]

END_TYPE_SELECTION = [
    ('count', 'Number of repetitions'),
    ('end_date', 'End date'),
    ('forever', 'Forever'),
]

MONTH_BY_SELECTION = [
    ('date', 'Date of month'),
    ('day', 'Day of month'),
]

WEEKDAY_SELECTION = [
    ('MON', 'Monday'),
    ('TUE', 'Tuesday'),
    ('WED', 'Wednesday'),
    ('THU', 'Thursday'),
    ('FRI', 'Friday'),
    ('SAT', 'Saturday'),
    ('SUN', 'Sunday'),
]

BYDAY_SELECTION = [
    ('1', 'First'),
    ('2', 'Second'),
    ('3', 'Third'),
    ('4', 'Fourth'),
    ('-1', 'Last'),
]

def freq_to_select(rrule_freq):
    return RRULE_FREQ_TO_SELECT[rrule_freq]


def freq_to_rrule(freq):
    return SELECT_FREQ_TO_RRULE[freq]


def weekday_to_field(weekday_index):
    return RRULE_WEEKDAY_TO_FIELD.get(weekday_index)


class RecurrenceRule(models.Model):
    _name = 'calendar.recurrence'
    _description = 'Event Recurrence Rule'

    name = fields.Char(compute='_compute_name', store=True)
    base_event_id = fields.Many2one(
        'calendar.event', ondelete='set null', copy=False)  # store=False ?
    calendar_event_ids = fields.One2many('calendar.event', 'recurrence_id')
    event_tz = fields.Selection(
        _tz_get, string='Timezone',
        default=lambda self: self.env.context.get('tz') or self.env.user.tz)
    rrule = fields.Char(compute='_compute_rrule', inverse='_inverse_rrule', store=True)
    dtstart = fields.Datetime(compute='_compute_dtstart')
    rrule_type = fields.Selection(RRULE_TYPE_SELECTION, default='weekly')
    end_type = fields.Selection(END_TYPE_SELECTION, default='count')
    interval = fields.Integer(default=1)
    count = fields.Integer(default=1)
    mon = fields.Boolean()
    tue = fields.Boolean()
    wed = fields.Boolean()
    thu = fields.Boolean()
    fri = fields.Boolean()
    sat = fields.Boolean()
    sun = fields.Boolean()
    month_by = fields.Selection(MONTH_BY_SELECTION, default='date')
    day = fields.Integer(default=1)
    weekday = fields.Selection(WEEKDAY_SELECTION, string='Weekday')
    byday = fields.Selection(BYDAY_SELECTION, string='By day')
    until = fields.Date('Repeat Until')
    trigger_id = fields.Many2one('ir.cron.trigger')

    _sql_constraints = [
        ('month_day',
         "CHECK (rrule_type != 'monthly' "
                "OR month_by != 'day' "
                "OR day >= 1 AND day <= 31 "
                "OR weekday in %s AND byday in %s)"
                % (tuple(wd[0] for wd in WEEKDAY_SELECTION), tuple(bd[0] for bd in BYDAY_SELECTION)),
         "The day must be between 1 and 31"),
    ]

    def _get_daily_recurrence_name(self):
        if self.end_type == 'count':
            return _("Every %(interval)s Days for %(count)s events", interval=self.interval, count=self.count)
        if self.end_type == 'end_date':
            return _("Every %(interval)s Days until %(until)s", interval=self.interval, until=self.until)
        return _("Every %(interval)s Days", interval=self.interval)

    def _get_weekly_recurrence_name(self):
        weekday_selection = dict(self._fields['weekday']._description_selection(self.env))
        weekdays = self._get_week_days()
        # Convert Weekday object
        weekdays = [str(w) for w in weekdays]
        # We need to get the day full name from its three first letters.
        week_map = {v: k for k, v in RRULE_WEEKDAYS.items()}
        weekday_short = [week_map[w] for w in weekdays]
        day_strings = [weekday_selection[day] for day in weekday_short]
        days = ", ".join(day_strings)

        if self.end_type == 'count':
            return _("Every %(interval)s Weeks on %(days)s for %(count)s events", interval=self.interval, days=days, count=self.count)
        if self.end_type == 'end_date':
            return _("Every %(interval)s Weeks on %(days)s until %(until)s", interval=self.interval, days=days, until=self.until)
        return _("Every %(interval)s Weeks on %(days)s", interval=self.interval, days=days)

    def _get_monthly_recurrence_name(self):
        if self.month_by == 'day':
            weekday_selection = dict(self._fields['weekday']._description_selection(self.env))
            byday_selection = dict(self._fields['byday']._description_selection(self.env))
            position_label = byday_selection[self.byday]
            weekday_label = weekday_selection[self.weekday]

            if self.end_type == 'count':
                return _("Every %(interval)s Months on the %(position)s %(weekday)s for %(count)s events", interval=self.interval, position=position_label, weekday=weekday_label, count=self.count)
            if self.end_type == 'end_date':
                return _("Every %(interval)s Months on the %(position)s %(weekday)s until %(until)s", interval=self.interval, position=position_label, weekday=weekday_label, until=self.until)
            return _("Every %(interval)s Months on the %(position)s %(weekday)s", interval=self.interval, position=position_label, weekday=weekday_label)
        else:
            if self.end_type == 'count':
                return _("Every %(interval)s Months day %(day)s for %(count)s events", interval=self.interval, day=self.day, count=self.count)
            if self.end_type == 'end_date':
                return _("Every %(interval)s Months day %(day)s until %(until)s", interval=self.interval, day=self.day, until=self.until)
            return _("Every %(interval)s Months day %(day)s", interval=self.interval, day=self.day)

    def _get_yearly_recurrence_name(self):
        if self.end_type == 'count':
            return _("Every %(interval)s Years for %(count)s events", interval=self.interval, count=self.count)
        if self.end_type == 'end_date':
            return _("Every %(interval)s Years until %(until)s", interval=self.interval, until=self.until)
        return _("Every %(interval)s Years", interval=self.interval)

    def get_recurrence_name(self):
        if self.rrule_type == 'daily':
            return self._get_daily_recurrence_name()
        if self.rrule_type == 'weekly':
            return self._get_weekly_recurrence_name()
        if self.rrule_type == 'monthly':
            return self._get_monthly_recurrence_name()
        if self.rrule_type == 'yearly':
            return self._get_yearly_recurrence_name()

    @api.depends('rrule')
    def _compute_name(self):
        for recurrence in self:
            recurrence.name = recurrence.get_recurrence_name()

    @api.depends('calendar_event_ids.start')
    def _compute_dtstart(self):
        groups = self.env['calendar.event']._read_group([('recurrence_id', 'in', self.ids)], ['recurrence_id'], ['start:min'])
        start_mapping = {recurrence.id: start_min for recurrence, start_min in groups}
        for recurrence in self:
            recurrence.dtstart = start_mapping.get(recurrence.id)

    @api.depends(
        'byday', 'until', 'rrule_type', 'month_by', 'interval', 'count', 'end_type',
        'mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun', 'day', 'weekday')
    def _compute_rrule(self):
        for recurrence in self:
            current_rule = recurrence._rrule_serialize()
            if recurrence.rrule != current_rule:
                recurrence.write({'rrule': current_rule})

    def _inverse_rrule(self):
        for recurrence in self:
            if recurrence.rrule:
                values = self._rrule_parse(recurrence.rrule, recurrence.dtstart)
                recurrence.with_context(dont_notify=True).write(values)

    def _reconcile_events(self, ranges):
        """
        :param ranges: iterable of tuples (datetime_start, datetime_stop)
        :return: tuple (events of the recurrence already in sync with ranges,
                 and ranges not covered by any events)
        """
        ranges = set(ranges)

        synced_events = self.calendar_event_ids.filtered(lambda e: e._range() in ranges)

        existing_ranges = set(event._range() for event in synced_events)
        ranges_to_create = (event_range for event_range in ranges if event_range not in existing_ranges)
        return synced_events, ranges_to_create

    def _select_new_base_event(self):
        """
        when the base event is no more available (archived, deleted, etc.), a new one should be selected
        """
        for recurrence in self:
            recurrence.base_event_id = recurrence._get_first_event()

    def _apply_recurrence(self, specific_values_creation=None, no_send_edit=False, generic_values_creation=None):
        """Create missing events in the recurrence and detach events which no longer
        follow the recurrence rules.
        :return: detached events
        """
        event_vals = []
        keep = self.env['calendar.event']
        if specific_values_creation is None:
            specific_values_creation = {}

        for recurrence in self.filtered('base_event_id'):
            recurrence.calendar_event_ids |= recurrence.base_event_id
            event = recurrence.base_event_id or recurrence._get_first_event(include_outliers=False)
            duration = event.stop - event.start
            if specific_values_creation:
                ranges = set([(x[1], x[2]) for x in specific_values_creation if x[0] == recurrence.id])
            else:
                ranges = recurrence._range_calculation(event, duration)

            events_to_keep, ranges = recurrence._reconcile_events(ranges)
            keep |= events_to_keep
            [base_values] = event.copy_data()
            values = []
            for start, stop in ranges:
                value = dict(base_values, start=start, stop=stop, recurrence_id=recurrence.id, follow_recurrence=True)
                if (recurrence.id, start, stop) in specific_values_creation:
                    value.update(specific_values_creation[(recurrence.id, start, stop)])
                if generic_values_creation and recurrence.id in generic_values_creation:
                    value.update(generic_values_creation[recurrence.id])
                values += [value]
            event_vals += values

        events = self.calendar_event_ids - keep
        detached_events = self._detach_events(events)
        context = {
            **clean_context(self.env.context),
            **{'no_mail_to_attendees': True, 'mail_create_nolog': True},
        }
        self.env['calendar.event'].with_context(context).create(event_vals)
        return detached_events

    def _setup_alarms(self, recurrence_update=False):
        """ Schedule cron triggers for future events
        Create one ir.cron.trigger per recurrence.
        :param recurrence_update: boolean: if true, update all recurrences in self, else only the recurrences
               without trigger
        """
        now = self.env.context.get('date') or fields.Datetime.now()
        # get next events
        self.env['calendar.event'].flush_model(fnames=['recurrence_id', 'start'])
        if not self.calendar_event_ids.ids:
            return

        self.env.cr.execute("""
            SELECT DISTINCT ON (recurrence_id) id event_id, recurrence_id
                    FROM calendar_event 
                   WHERE start > %s
                     AND id IN %s
                ORDER BY recurrence_id,start ASC;
        """, (now, tuple(self.calendar_event_ids.ids)))
        result = self.env.cr.dictfetchall()
        if not result:
            return
        events = self.env['calendar.event'].browse(value['event_id'] for value in result)
        triggers_by_events = events._setup_alarms()
        for vals in result:
            trigger_id = triggers_by_events.get(vals['event_id'])
            if not trigger_id:
                continue
            recurrence = self.env['calendar.recurrence'].browse(vals['recurrence_id'])
            recurrence.trigger_id = trigger_id

    def _split_from(self, event, recurrence_values=None):
        """Stops the current recurrence at the given event and creates a new one starting
        with the event.
        :param event: starting point of the new recurrence
        :param recurrence_values: values applied to the new recurrence
        :return: new recurrence
        """
        if recurrence_values is None:
            recurrence_values = {}
        event.ensure_one()
        if not self:
            return
        [values] = self.copy_data()
        detached_events = self._stop_at(event)

        count = recurrence_values.get('count', 0) or len(detached_events)
        return self.create({
            **values,
            **recurrence_values,
            'base_event_id': event.id,
            'calendar_event_ids': [(6, 0, detached_events.ids)],
            'count': max(count, 1),
        })

    def _stop_at(self, event):
        """Stops the recurrence at the given event. Detach the event and all following
        events from the recurrence.

        :return: detached events from the recurrence
        """
        self.ensure_one()
        events = self._get_events_from(event.start)
        detached_events = self._detach_events(events)
        if not self.calendar_event_ids:
            self.with_context(archive_on_error=True).unlink()
            return detached_events

        if event.allday:
            until = self._get_start_of_period(event.start_date)
        else:
            until_datetime = self._get_start_of_period(event.start)
            until_timezoned = pytz.utc.localize(until_datetime).astimezone(self._get_timezone())
            until = until_timezoned.date()
        self.write({
            'end_type': 'end_date',
            'until': until - relativedelta(days=1),
        })
        return detached_events

    @api.model
    def _detach_events(self, events):
        events.with_context(dont_notify=True).write({
            'recurrence_id': False,
            'recurrency': True,
        })
        return events

    def _write_events(self, values, dtstart=None):
        """
        Write values on events in the recurrence.
        :param values: event values
        :param dstart: if provided, only write events starting from this point in time
        """
        events = self._get_events_from(dtstart) if dtstart else self.calendar_event_ids
        return events.with_context(no_mail_to_attendees=True, dont_notify=True).write(dict(values, recurrence_update='self_only'))

    def _rrule_serialize(self):
        """
        Compute rule string according to value type RECUR of iCalendar
        :return: string containing recurring rule (empty if no rule)
        """
        if self.interval <= 0:
            raise UserError(_('The interval cannot be negative.'))
        if self.end_type == 'count' and self.count <= 0:
            raise UserError(_('The number of repetitions cannot be negative.'))

        return str(self._get_rrule()) if self.rrule_type else ''

    @api.model
    def _rrule_parse(self, rule_str, date_start):
        # LUL TODO clean this mess
        data = {}
        day_list = ['mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun']

        # Skip X-named RRULE extensions
        # TODO Remove patch when dateutils contains the fix
        # HACK https://github.com/dateutil/dateutil/pull/1374
        # Optional parameters starts with X- and they can be placed anywhere in the RRULE string.
        # RRULE:FREQ=MONTHLY;INTERVAL=3;X-RELATIVE=1
        # RRULE;X-EVOLUTION-ENDDATE=20200120:FREQ=WEEKLY;COUNT=3;BYDAY=MO
        # X-EVOLUTION-ENDDATE=20200120:FREQ=WEEKLY;COUNT=3;BYDAY=MO
        rule_str = re.sub(r';?X-[-\w]+=[^;:]*', '', rule_str).replace(":;", ":").lstrip(":;")

        if 'Z' in rule_str and date_start and not date_start.tzinfo:
            date_start = pytz.utc.localize(date_start)
        rule = rrule.rrulestr(rule_str, dtstart=date_start)

        data['rrule_type'] = freq_to_select(rule._freq)
        data['count'] = rule._count
        data['interval'] = rule._interval
        data['until'] = rule._until
        # Repeat weekly
        if rule._byweekday:
            for weekday in day_list:
                data[weekday] = False  # reset
            for weekday_index in rule._byweekday:
                weekday = rrule.weekday(weekday_index)
                data[weekday_to_field(weekday.weekday)] = True
                data['rrule_type'] = 'weekly'

        # Repeat monthly by nweekday ((weekday, weeknumber), )
        if rule._bynweekday:
            data['weekday'] = day_list[list(rule._bynweekday)[0][0]].upper()
            data['byday'] = str(list(rule._bynweekday)[0][1])
            data['month_by'] = 'day'
            data['rrule_type'] = 'monthly'

        if rule._bymonthday:
            data['day'] = list(rule._bymonthday)[0]
            data['month_by'] = 'date'
            data['rrule_type'] = 'monthly'

        # Repeat yearly but for odoo it's monthly, take same information as monthly but interval is 12 times
        if rule._bymonth:
            data['interval'] *= 12

        if data.get('until'):
            data['end_type'] = 'end_date'
        elif data.get('count'):
            data['end_type'] = 'count'
        else:
            data['end_type'] = 'forever'
        return data

    def _get_lang_week_start(self):
        lang = self.env['res.lang']._lang_get(self.env.user.lang)
        week_start = int(lang.week_start)  # lang.week_start ranges from '1' to '7'
        return rrule.weekday(week_start - 1) # rrule expects an int from 0 to 6

    def _get_start_of_period(self, dt):
        if self.rrule_type == 'weekly':
            week_start = self._get_lang_week_start()
            start = dt + relativedelta(weekday=week_start(-1))
        elif self.rrule_type == 'monthly':
            start = dt + relativedelta(day=1)
        else:
            start = dt
        # Comparaison of DST (to manage the case of going too far back in time).
        # If we detect a change in the DST between the creation date of an event
        # and the date used for the occurrence period, we use the creation date of the event.
        # This is a hack to avoid duplication of events (for example on google calendar).
        if isinstance(dt, datetime):
            timezone = self._get_timezone()
            dst_dt = timezone.localize(dt).dst()
            dst_start = timezone.localize(start).dst()
            if dst_dt != dst_start:
                start = dt
        return start

    def _get_first_event(self, include_outliers=False):
        if not self.calendar_event_ids:
            return self.env['calendar.event']
        events = self.calendar_event_ids.sorted('start')
        if not include_outliers:
            events -= self._get_outliers()
        return events[:1]

    def _get_outliers(self):
        synced_events = self.env['calendar.event']
        for recurrence in self:
            if recurrence.calendar_event_ids:
                start = min(recurrence.calendar_event_ids.mapped('start'))
                starts = set(recurrence._get_occurrences(start))
                synced_events |= recurrence.calendar_event_ids.filtered(lambda e: e.start in starts)
        return self.calendar_event_ids - synced_events

    def _range_calculation(self, event, duration):
        """ Calculate the range of recurrence when applying the recurrence
        The following issues are taken into account:
            start of period is sometimes in the past (weekly or monthly rule).
            We can easily filter these range values but then the count value may be wrong...
            In that case, we just increase the count value, recompute the ranges and dismiss the useless values
        """
        self.ensure_one()
        original_count = self.end_type == 'count' and self.count
        ranges = set(self._get_ranges(event.start, duration))
        future_events = set((x, y) for x, y in ranges if x.date() >= event.start.date() and y.date() >= event.start.date())
        if original_count and len(future_events) < original_count:
            # Rise count number because some past values will be dismissed.
            self.count = (2*original_count) - len(future_events)
            ranges = set(self._get_ranges(event.start, duration))
            # We set back the occurrence number to its original value
            self.count = original_count
        # Remove ranges of events occurring in the past
        ranges = set((x, y) for x, y in ranges if x.date() >= event.start.date() and y.date() >= event.start.date())
        return ranges


    def _get_ranges(self, start, event_duration):
        starts = self._get_occurrences(start)
        return ((start, start + event_duration) for start in starts)

    def _get_timezone(self):
        return pytz.timezone(self.event_tz or self.env.context.get('tz') or 'UTC')

    def _get_occurrences(self, dtstart):
        """
        Get ocurrences of the rrule
        :param dtstart: start of the recurrence
        :return: iterable of datetimes
        """
        self.ensure_one()
        dtstart = self._get_start_of_period(dtstart)
        if self._is_allday():
            return self._get_rrule(dtstart=dtstart)

        timezone = self._get_timezone()
        # Localize the starting datetime to avoid missing the first occurrence
        dtstart = pytz.utc.localize(dtstart).astimezone(timezone)
        # dtstart is given as a naive datetime, but it actually represents a timezoned datetime
        # (rrule package expects a naive datetime)
        occurences = self._get_rrule(dtstart=dtstart.replace(tzinfo=None))

        # Special timezoning is needed to handle DST (Daylight Saving Time) changes.
        # Given the following recurrence:
        #   - monthly
        #   - 1st of each month
        #   - timezone America/New_York (UTC−05:00)
        #   - at 6am America/New_York = 11am UTC
        #   - from 2019/02/01 to 2019/05/01.
        # The naive way would be to store:
        # 2019/02/01 11:00 - 2019/03/01 11:00 - 2019/04/01 11:00 - 2019/05/01 11:00 (UTC)
        #
        # But a DST change occurs on 2019/03/10 in America/New_York timezone. America/New_York is now UTC−04:00.
        # From this point in time, 11am (UTC) is actually converted to 7am (America/New_York) instead of the expected 6am!
        # What should be stored is:
        # 2019/02/01 11:00 - 2019/03/01 11:00 - 2019/04/01 10:00 - 2019/05/01 10:00 (UTC)
        #                                                  *****              *****
        return (timezone.localize(occurrence, is_dst=False).astimezone(pytz.utc).replace(tzinfo=None) for occurrence in occurences)

    def _get_events_from(self, dtstart):
        return self.env['calendar.event'].search([
            ('id', 'in', self.calendar_event_ids.ids),
            ('start', '>=', dtstart)
        ])

    def _get_week_days(self):
        """
        :return: tuple of rrule weekdays for this recurrence.
        """
        return tuple(
            rrule.weekday(weekday_index)
            for weekday_index, weekday in {
                rrule.MO.weekday: self.mon,
                rrule.TU.weekday: self.tue,
                rrule.WE.weekday: self.wed,
                rrule.TH.weekday: self.thu,
                rrule.FR.weekday: self.fri,
                rrule.SA.weekday: self.sat,
                rrule.SU.weekday: self.sun,
            }.items() if weekday
        )

    def _is_allday(self):
        """Returns whether a majority of events are allday or not (there might be some outlier events)
        """
        score = sum(1 if e.allday else -1 for e in self.calendar_event_ids)
        return score >= 0

    def _get_rrule(self, dtstart=None):
        self.ensure_one()
        freq = self.rrule_type
        rrule_params = dict(
            dtstart=dtstart,
            interval=self.interval,
        )
        if freq == 'monthly' and self.month_by == 'date':  # e.g. every 15th of the month
            rrule_params['bymonthday'] = self.day
        elif freq == 'monthly' and self.month_by == 'day':  # e.g. every 2nd Monday in the month
            rrule_params['byweekday'] = getattr(rrule, RRULE_WEEKDAYS[self.weekday])(int(self.byday))  # e.g. MO(+2) for the second Monday of the month
        elif freq == 'weekly':
            weekdays = self._get_week_days()
            if not weekdays:
                raise UserError(_("You have to choose at least one day in the week"))
            rrule_params['byweekday'] = weekdays
            rrule_params['wkst'] = self._get_lang_week_start()

        if self.end_type == 'count':  # e.g. stop after X occurence
            rrule_params['count'] = min(self.count, MAX_RECURRENT_EVENT)
        elif self.end_type == 'forever':
            rrule_params['count'] = MAX_RECURRENT_EVENT
        elif self.end_type == 'end_date':  # e.g. stop after 12/10/2020
            rrule_params['until'] = datetime.combine(self.until, time.max)
        return rrule.rrule(
            freq_to_rrule(freq), **rrule_params
        )

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.exceptions import BadRequest

from odoo import models
from odoo.http import request


class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    @classmethod
    def _auth_method_calendar(cls):
        token = request.get_http_params().get('token', '')

        error_message = False

        attendee = request.env['calendar.attendee'].sudo().search([('access_token', '=', token)], limit=1)
        if not attendee:
            error_message = """Invalid Invitation Token."""
        elif request.session.uid and request.session.login != 'anonymous':
            # if valid session but user is not match
            user = request.env['res.users'].sudo().browse(request.session.uid)
            if attendee.partner_id != user.partner_id:
                error_message = """Invitation cannot be forwarded via email. This event/meeting belongs to %s and you are logged in as %s. Please ask organizer to add you.""" % (attendee.email, user.email)
        if error_message:
            raise BadRequest(error_message)

        cls._auth_method_public()

```

## File: models\mail_activity.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, tools, _
from odoo.tools import is_html_empty


class MailActivity(models.Model):
    _inherit = "mail.activity"

    calendar_event_id = fields.Many2one('calendar.event', string="Calendar Meeting", ondelete='cascade')

    def action_create_calendar_event(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("calendar.action_calendar_event")
        action['context'] = {
            'default_activity_type_id': self.activity_type_id.id,
            'default_res_id': self.env.context.get('default_res_id'),
            'default_res_model': self.env.context.get('default_res_model'),
            'default_name': self.summary or self.res_name,
            'default_description': self.note if not is_html_empty(self.note) else '',
            'default_activity_ids': [(6, 0, self.ids)],
            'default_partner_ids': self.user_id.partner_id.ids,
            'default_user_id': self.user_id.id,
            'initial_date': self.date_deadline,
            'default_calendar_event_id': self.calendar_event_id.id,
            'orig_activity_ids': self.ids,
        }
        return action

    def _action_done(self, feedback=False, attachment_ids=False):
        events = self.calendar_event_id
        # To avoid the feedback to be included in the activity note (due to the synchronization in event.write
        # that updates the related activity note each time the event description is updated),
        # when the activity is written as a note in the chatter in _action_done (leading to duplicate feedback),
        # we call super before updating the description. As self is deleted in super, we load the related events before.
        messages, activities = super(MailActivity, self)._action_done(feedback=feedback, attachment_ids=attachment_ids)
        if feedback:
            for event in events:
                description = event.description
                description = '%s<br />%s' % (
                    description if not tools.is_html_empty(description) else '',
                    _('Feedback: %(feedback)s', feedback=tools.plaintext2html(feedback)) if feedback else '',
                )
                event.write({'description': description})
        return messages, activities

    def unlink_w_meeting(self):
        events = self.mapped('calendar_event_id')
        res = self.unlink()
        events.unlink()
        return res

```

## File: models\mail_activity_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models

class MailActivityMixin(models.AbstractModel):
    _inherit = 'mail.activity.mixin'

    activity_calendar_event_id = fields.Many2one(
        'calendar.event', string="Next Activity Calendar Event",
        compute='_compute_activity_calendar_event_id', groups="base.group_user")

    @api.depends('activity_ids.calendar_event_id')
    def _compute_activity_calendar_event_id(self):
        """This computes the calendar event of the next activity.
        It evaluates to false if there is no such event."""
        for record in self:
            record.activity_calendar_event_id = fields.first(record.activity_ids).calendar_event_id

```

## File: models\mail_activity_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class MailActivityType(models.Model):
    _inherit = "mail.activity.type"

    category = fields.Selection(selection_add=[('meeting', 'Meeting')])

```

## File: models\onboarding_onboarding.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class Onboarding(models.Model):
    _inherit = 'onboarding.onboarding'

    @api.model
    def action_close_calendar_onboarding(self):
        self.action_close_panel('calendar.onboarding_onboarding_calendar')

    def _prepare_rendering_values(self):
        """Compute existence of invoices for company."""
        self.ensure_one()
        if self == self.env.ref('calendar.onboarding_onboarding_calendar', raise_if_not_found=False):
            step = self.env.ref('calendar.onboarding_onboarding_step_setup_calendar_integration', raise_if_not_found=False)
            if step and step.current_step_state == 'not_done':
                credentials = self.env['res.users'].check_calendar_credentials()
                if any(credentials[service] for service in credentials):
                    step.action_set_just_done()
        return super()._prepare_rendering_values()

```

## File: models\onboarding_onboarding_step.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class OnboardingStep(models.Model):
    _inherit = 'onboarding.onboarding.step'

    @api.model
    def action_view_start_calendar_sync(self):
        action = self.env["ir.actions.actions"]._for_xml_id("calendar.action_view_start_calendar_sync")
        return action

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime

from odoo import api, fields, models


class Partner(models.Model):
    _inherit = 'res.partner'

    meeting_count = fields.Integer("# Meetings", compute='_compute_meeting_count')
    meeting_ids = fields.Many2many('calendar.event', 'calendar_event_res_partner_rel', 'res_partner_id',
                                   'calendar_event_id', string='Meetings', copy=False)

    calendar_last_notif_ack = fields.Datetime(
        'Last notification marked as read from base Calendar', default=fields.Datetime.now)

    def _compute_meeting_count(self):
        result = self._compute_meeting()
        for p in self:
            p.meeting_count = len(result.get(p.id, []))

    def _compute_meeting(self):
        if self.ids:
            # prefetch 'parent_id'
            all_partners = self.with_context(active_test=False).search_fetch(
                [('id', 'child_of', self.ids)], ['parent_id'],
            )

            query = self.env['calendar.event']._search([])  # ir.rules will be applied
            query_str, params = query.subselect()

            self.env.cr.execute(f"""
                SELECT res_partner_id, calendar_event_id, count(1)
                  FROM calendar_event_res_partner_rel
                 WHERE res_partner_id IN %s AND calendar_event_id IN {query_str}
              GROUP BY res_partner_id, calendar_event_id
            """, [tuple(all_partners.ids)] + params)

            meeting_data = self.env.cr.fetchall()

            # Create a dict {partner_id: event_ids} and fill with events linked to the partner
            meetings = {}
            for p_id, m_id, _ in meeting_data:
                meetings.setdefault(p_id, set()).add(m_id)

            # Add the events linked to the children of the partner
            for p in self.browse(meetings.keys()):
                partner = p
                while partner.parent_id:
                    partner = partner.parent_id
                    if partner in self:
                        meetings[partner.id] = meetings.get(partner.id, set()) | meetings[p.id]
            return {p_id: list(meetings.get(p_id, set())) for p_id in self.ids}
        return {}

    def get_attendee_detail(self, meeting_ids):
        """ Return a list of dict of the given meetings with the attendees details
            Used by:
                - many2many_attendee.js: Many2ManyAttendee
                - calendar_model.js (calendar.CalendarModel)
        """
        attendees_details = []
        meetings = self.env['calendar.event'].browse(meeting_ids)
        for attendee in meetings.attendee_ids:
            if attendee.partner_id not in self:
                continue
            attendee_is_organizer = self.env.user == attendee.event_id.user_id and attendee.partner_id == self.env.user.partner_id
            attendees_details.append({
                'id': attendee.partner_id.id,
                'name': attendee.partner_id.display_name,
                'status': attendee.state,
                'event_id': attendee.event_id.id,
                'attendee_id': attendee.id,
                'is_alone': attendee.event_id.is_organizer_alone and attendee_is_organizer,
                # attendees data is sorted according to this key in JS.
                'is_organizer': 1 if attendee.partner_id == attendee.event_id.user_id.partner_id else 0,
            })
        return attendees_details

    @api.model
    def _set_calendar_last_notif_ack(self):
        partner = self.env['res.users'].browse(self.env.context.get('uid', self.env.uid)).partner_id
        partner.write({'calendar_last_notif_ack': datetime.now()})

    def schedule_meeting(self):
        self.ensure_one()
        partner_ids = self.ids
        partner_ids.append(self.env.user.partner_id.id)
        action = self.env["ir.actions.actions"]._for_xml_id("calendar.action_calendar_event")
        action['context'] = {
            'default_partner_ids': partner_ids,
        }
        action['domain'] = ['|', ('id', 'in', self._compute_meeting()[self.id]), ('partner_ids', 'in', self.ids)]
        return action

```

## File: models\res_users.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime

from odoo import api, fields, models, modules, _
from pytz import timezone, UTC


class Users(models.Model):
    _inherit = 'res.users'

    def _systray_get_calendar_event_domain(self):
        # Determine the domain for which the users should be notified. This method sends notification to
        # events occurring between now and the end of the day. Note that "now" needs to be computed in the
        # user TZ and converted into UTC to compare with the records values and "the end of the day" needs
        # also conversion. Otherwise TZ diverting a lot from UTC would send notification for events occurring
        # tomorrow.
        # The user is notified if the start is occurring between now and the end of the day
        # if the event is not finished.
        #   |           |
        #   |===========|===> DAY A (`start_dt`): now in the user TZ
        #   |           |
        #   |           | <--- `start_dt_utc`: now is on the right if the user lives
        #   |           |               in West Longitude (America for example)
        #   |           |
        #   |  -------  | <--- `start`: the start of the event (in UTC)
        #   | | event | |
        #   |  -------  | <--- `stop`: the stop of the event (in UTC)
        #   |           |
        #   |           |
        #   |           | <--- `stop_dt_utc` = `stop_dt` if user lives in an area of East longitude (positive shift compared to UTC, Belgium for example)
        #   |           |
        #   |           |
        #   |-----------| <--- `stop_dt` = end of the day for DAY A from user point of view (23:59 in this TZ)
        #   |===========|===> DAY B
        #   |           |
        #   |           | <--- `stop_dt_utc` = `stop_dt` if user lives in an area of West longitude (positive shift compared to UTC, America for example)
        #   |           |
        start_dt_utc = start_dt = datetime.datetime.now(UTC)
        stop_dt_utc = UTC.localize(datetime.datetime.combine(start_dt.date(), datetime.time.max))

        tz = self.env.user.tz
        if tz:
            user_tz = timezone(tz)
            start_dt = start_dt_utc.astimezone(user_tz)
            stop_dt = user_tz.localize(datetime.datetime.combine(start_dt.date(), datetime.time.max))
            stop_dt_utc = stop_dt.astimezone(UTC)

        start_date = start_dt.date()

        current_user_non_declined_attendee_ids = self.env['calendar.attendee']._search([
            ('partner_id', '=', self.env.user.partner_id.id),
            ('state', '!=', 'declined'),
        ])

        return ['&', '|',
                '&',
                    '|',
                        ['start', '>=', fields.Datetime.to_string(start_dt_utc)],
                        ['stop', '>=', fields.Datetime.to_string(start_dt_utc)],
                    ['start', '<=', fields.Datetime.to_string(stop_dt_utc)],
                '&',
                    ['allday', '=', True],
                    ['start_date', '=', fields.Date.to_string(start_date)],
                ('attendee_ids', 'in', current_user_non_declined_attendee_ids)]

    @api.model
    def systray_get_activities(self):
        res = super(Users, self).systray_get_activities()

        EventModel = self.env['calendar.event']
        meetings_lines = EventModel.search_read(
            self._systray_get_calendar_event_domain(),
            ['id', 'start', 'name', 'allday'],
            order='start')
        if meetings_lines:
            meeting_label = _("Today's Meetings")
            meetings_systray = {
                'id': self.env['ir.model']._get('calendar.event').id,
                'type': 'meeting',
                'name': meeting_label,
                'model': 'calendar.event',
                'icon': modules.module.get_module_icon(EventModel._original_module),
                'meetings': meetings_lines,
                "view_type": EventModel._systray_view,
            }
            res.insert(0, meetings_systray)

        return res

    @api.model
    def check_calendar_credentials(self):
        return {}

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import res_partner
from . import calendar_event
from . import calendar_alarm
from . import calendar_alarm_manager
from . import calendar_attendee
from . import calendar_filter
from . import calendar_event_type
from . import calendar_recurrence
from . import mail_activity
from . import mail_activity_mixin
from . import mail_activity_type
from . import res_users
from . import onboarding_onboarding
from . import onboarding_onboarding_step

```

## File: populate\calendar_alarm.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.addons.calendar.populate import data
from odoo.tools import populate


class Alarm(models.Model):
    _inherit = 'calendar.alarm'
    _populate_sizes = {'small': 3, 'medium': 10, 'large': 30}

    def _populate_factories(self):
        def get_name(values, counter, **kwargs):
            return f"{str.capitalize(values['alarm_type'])} - {values['duration']} {values['interval']} (#{counter})."

        return [
            *((field_name, populate.iterate(*zip(*data.calendar_alarm[field_name].items())))
              for field_name in ["alarm_type", "duration", "interval"]),
            ('name', populate.compute(get_name)),
        ]

```

## File: populate\data.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

calendar_alarm = {
    "alarm_type": {"email": 0.9, "notification": 0.1},
    "duration": {
        1: 0.8,
        2: 0.09,
        3: 0.11
    },
    "interval": {"hours": 0.8, "minutes": 0.1, "days": 0.1}
}

```

## File: populate\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import calendar_alarm
from . import data

```

## File: security\calendar_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="calendar_event_rule_my" model="ir.rule">
            <field name="name">Own events</field>
            <field ref="model_calendar_event" name="model_id"/>
            <field name="domain_force">[('partner_ids', 'in', user.partner_id.id)]</field>
            <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
        </record>

        <record id="calendar_event_rule_employee" model="ir.rule">
            <field ref="model_calendar_event" name="model_id"/>
            <field name="name">All Calendar Event for employees</field>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field eval="[(4,ref('base.group_user'))]" name="groups"/>
        </record>

        <record id="calendar_attendee_rule_my" model="ir.rule">
            <field name="name">Own attendees</field>
            <field ref="model_calendar_attendee" name="model_id"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
        </record>

        <record id="calendar_event_rule_private" model="ir.rule">
            <field ref="model_calendar_event" name="model_id"/>
            <field name="name">Private events</field>
            <field name="domain_force">['|', ('privacy', '!=', 'private'), '&amp;', ('privacy', '=', 'private'), '|', ('user_id', '=', user.id), ('partner_ids', 'in', user.partner_id.id)]</field>
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
access_calendar_attendee_employee_delete_wizard,calendar.attendee_employee_delete_wizard,model_calendar_popover_delete_wizard,base.group_user,1,1,1,1
access_calendar_alarm,calendar.alarm,model_calendar_alarm,base.group_user,1,1,1,1
access_calendar_event_all_user,calendar.event_all_user,model_calendar_event,base.group_portal,1,0,0,0
access_calendar_event_all_employee,calendar.event_all_employee,model_calendar_event,base.group_user,1,1,1,1
access_calendar_event_partner_manager,calendar.event.partner.manager,model_calendar_event,base.group_partner_manager,1,1,1,1
access_calendar_event_type_all,calendar.event.type.all,model_calendar_event_type,base.group_user,1,0,0,0
access_calendar_event_type_manager,calendar.event.type.manager,model_calendar_event_type,base.group_system,1,1,1,1
access_calendar_alarm_manager,access_calendar_alarm_manager,model_calendar_alarm_manager,base.group_user,1,1,1,1
access_calendar_filters_all,access_calendar_contacts_all,model_calendar_filters,base.group_user,1,1,1,1
access_calendar_filters,access_calendar_contacts,model_calendar_filters,base.group_system,1,1,1,1
access_calendar_recurrence,access_calendar_recurrence,model_calendar_recurrence,base.group_user,1,1,1,1
access_calendar_provider_config_all,calendar.provider.config.all,model_calendar_provider_config,,0,0,0,0
access_calendar_provider_config_manager,calendar.provider.config.manager,model_calendar_provider_config,base.group_system,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M16 38c3 0 6-1 6-5s-2.306-6-6-6a3 3 0 0 1 3-3h4c3.593 1.3 6 4 6 9 0 8-6 11-13 11S4 41 4 33h7c0 3 2 5 5 5Z" fill="#985184"/><path d="M23 24h-5a2 2 0 0 1-2-2c4 0 5-3 5-5 0-3-1.964-5-5-5-3 0-5 2-5 4H4C4 9.616 10 6 16 6s12 4 12 10-5 8-5 8Z" fill="#FBB945"/><path d="m23 24-7 3h-4v-2c0-2 1-3 4-3l7 2Z" fill="#FC868B"/><path d="M46 11v33h-3c-2 0-4-2-4-4V14l7-3Z" fill="#FBB945"/><path d="m39 7.334-7 4.667-1 6 8-4V7.334Z" fill="#985184"/><path d="M39 7.333 41 6h1c2 0 4 2 4 4v1l-7 3V7.333Z" fill="#FC868B"/></svg>

```

## File: static\src\activity\activity_list_popover_item_patch.js

```javascript
/** @odoo-module */

import { ActivityListPopoverItem } from "@mail/core/web/activity_list_popover_item";
import { patch } from "@web/core/utils/patch";

patch(ActivityListPopoverItem.prototype, {
    get hasEditButton() {
        return super.hasEditButton && !this.props.activity.calendar_event_id;
    },

    async onClickReschedule() {
        await this.env.services["mail.activity"].rescheduleMeeting(this.props.activity.id);
    },
});

```

## File: static\src\activity\activity_list_popover_item_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.ActivityListPopoverItem" t-inherit-mode="extension">
        <xpath expr="//button[hasclass('o-mail-ActivityListPopoverItem-editbtn')]" position="after">
            <button t-if="props.activity.calendar_event_id" class="o-mail-ActivityListPopoverItem-editbtn btn btn-sm btn-success btn-link" t-on-click="onClickReschedule">
                <i class="fa fa-calendar"/>
            </button>
        </xpath>
    </t>
</templates>

```

## File: static\src\activity\activity_menu_patch.js

```javascript
/** @odoo-module */

import { ActivityMenu } from "@mail/core/web/activity_menu";
import { patch } from "@web/core/utils/patch";
import { deserializeDateTime, formatDateTime } from "@web/core/l10n/dates";
import { localization } from "@web/core/l10n/localization";

patch(ActivityMenu.prototype, {
    async fetchSystrayActivities() {
        await super.fetchSystrayActivities();
        for (const group of Object.values(this.store.activityGroups)) {
            if (group.type === "meeting") {
                for (const meeting of group.meetings) {
                    if (meeting.start) {
                        const date = deserializeDateTime(meeting.start);
                        meeting.formattedStart = formatDateTime(date, { format: localization.timeFormat });
                    }
                }
            }
        }
    },

    openActivityGroup(group) {
        if (group.model === "calendar.event") {
            document.body.click();
            this.action.doAction("calendar.action_calendar_event", {
                additionalContext: {
                    default_mode: "day",
                    search_default_mymeetings: 1,
                },
                clearBreadcrumbs: true,
            });
        } else {
            super.openActivityGroup(...arguments);
        }
    },
});

```

## File: static\src\activity\activity_menu_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.ActivityMenu" t-inherit-mode="extension">
        <xpath expr="//*[@name='activityTitle']" position="after">
            <div t-if="group.type === 'meeting'">
                <t t-set="is_next_meeting" t-value="true"/>
                <t t-foreach="group.meetings" t-as="meeting" t-key="meeting.id">
                    <div class="o-calendar-meeting d-flex px-2">
                        <span class="flex-grow-1 text-truncate" t-att-class="!meeting.allday and is_next_meeting ? 'fw-bold' : ''" t-esc="meeting.name"/>
                        <span t-if="meeting.formattedStart">
                            <t t-if="meeting.allday">All Day</t>
                            <t t-else=''>
                                <t t-set="is_next_meeting" t-value="false"/>
                                <t t-esc="meeting.formattedStart"/>
                            </t>
                        </span>
                    </div>
                </t>
            </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\activity\activity_model_patch.js

```javascript
/** @odoo-module */

import { Activity } from "@mail/core/web/activity_model";
import { assignIn } from "@mail/utils/common/misc";
import { patch } from "@web/core/utils/patch";

patch(Activity, {
    _insert(data) {
        const activity = super._insert(...arguments);
        assignIn(activity, data, ["calendar_event_id"]);
        return activity;
    },
});

```

## File: static\src\activity\activity_patch.js

```javascript
/** @odoo-module */

import { Activity } from "@mail/core/web/activity";
import { useService } from "@web/core/utils/hooks";
import { patch } from "@web/core/utils/patch";

patch(Activity.prototype, {
    setup() {
        super.setup();
        this.orm = useService("orm");
    },
    async onClickReschedule() {
        await this.env.services["mail.activity"].rescheduleMeeting(this.props.data.id);
    },
    /**
     * @override
     */
    async unlink() {
        if (this.props.data.calendar_event_id) {
            const thread = this.thread;
            this.activityService.delete(this.props.data);
            await this.orm.call("mail.activity", "unlink_w_meeting", [[this.props.data.id]]);
            this.props.onUpdate(thread);
        } else {
            super.unlink();
        }
    },
});

```

## File: static\src\activity\activity_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-inherit="mail.Activity" t-inherit-mode="extension">
        <xpath expr="//span[hasclass('o-mail-Activity-edit')]" position="attributes">
            <attribute name="t-if">!activity.calendar_event_id</attribute>
        </xpath>
        <xpath expr="//span[hasclass('o-mail-Activity-edit')]" position="after">
            <span t-if="activity.calendar_event_id" class="btn btn-link p-0 me-3" t-on-click="onClickReschedule"><i class="fa fa-calendar"/> Reschedule</span>
        </xpath>
    </t>

</templates>

```

## File: static\src\activity\activity_service_patch.js

```javascript
/** @odoo-module */

import { ActivityService } from "@mail/core/web/activity_service";
import { patch } from "@web/core/utils/patch";

patch(ActivityService.prototype, {
    async rescheduleMeeting(activityId) {
        const action = await this.orm.call("mail.activity", "action_create_calendar_event", [
            [activityId],
        ]);
        this.env.services.action.doAction(action);
    },
});

```

## File: static\src\components\calendar_provider_config\calendar_connect_provider.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { session } from "@web/session";
import { standardWidgetProps } from "@web/views/widgets/standard_widget_props";
import { useService } from "@web/core/utils/hooks";
import { Component } from "@odoo/owl";

const providerData = {
    google: {
        restart_sync_method: "restart_google_synchronization",
        sync_route: "/google_calendar/sync_data",
    },
    microsoft: {
        restart_sync_method: "restart_microsoft_synchronization",
        sync_route: "/microsoft_calendar/sync_data",
    },
};

export class CalendarConnectProvider extends Component {
    setup() {
        super.setup();
        this.orm = useService("orm");
        this.rpc = useService("rpc");
    }

    /**
     * Activate the external sync for the first time, after installing the
     * relevant submodule if necessary.
     *
     * @private
     */
    async onConnect(ev) {
        ev.preventDefault();
        ev.stopImmediatePropagation();
        if (!(await this.props.record.save())) {
            return; // handled by view
        }
        await this.orm.call(
            this.props.record.resModel,
            "action_calendar_prepare_external_provider_sync",
            [this.props.record.resId]
        );
        // See google/microsoft_calendar for the origin of this shortened version
        const { restart_sync_method, sync_route } =
            providerData[this.props.record.data.external_calendar_provider];
        await this.orm.call("res.users", restart_sync_method, [[session.uid]]);
        const response = await this.rpc(sync_route, {
            model: "calendar.event",
            fromurl: window.location.href,
        });
        await this._beforeLeaveContext();
        if (response.status === "need_auth") {
            window.location.assign(response.url);
        } else if (response.status === "need_refresh") {
            window.location.reload();
        }
    }
    /**
     * Hook to perform additional work before redirecting to external url or reloading.
     *
     * @private
     */
    async _beforeLeaveContext() {
        return Promise.resolve();
    }
}
CalendarConnectProvider.props = {
    ...standardWidgetProps,
};
CalendarConnectProvider.template = "calendar.CalendarConnectProvider";

const calendarConnectProvider = {
    component: CalendarConnectProvider,
};
registry.category("view_widgets").add("calendar_connect_provider", calendarConnectProvider);

```

## File: static\src\components\calendar_provider_config\calendar_connect_provider.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="calendar.CalendarConnectProvider">
        <a role="button" title="Connect" class="o_calendar_activate_external_cal btn btn-primary"
            t-on-click="onConnect">Connect</a>
    </t>
</templates>

```

## File: static\src\js\services\calendar_notification_service.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { browser } from "@web/core/browser/browser";
import { ConnectionLostError } from "@web/core/network/rpc_service";
import { registry } from "@web/core/registry";

export const calendarNotificationService = {
    dependencies: ["action", "bus_service", "notification", "rpc"],

    start(env, { action, bus_service, notification, rpc }) {
        let calendarNotifTimeouts = {};
        let nextCalendarNotifTimeout = null;
        const displayedNotifications = new Set();

        bus_service.subscribe("calendar.alarm", (payload) => {
            displayCalendarNotification(payload);
        });
        bus_service.start();

        /**
         * Displays the Calendar notification on user's screen
         */
        function displayCalendarNotification(notifications) {
            let lastNotifTimer = 0;

            // Clear previously set timeouts and destroy currently displayed calendar notifications
            browser.clearTimeout(nextCalendarNotifTimeout);
            Object.values(calendarNotifTimeouts).forEach((notif) => browser.clearTimeout(notif));
            calendarNotifTimeouts = {};

            // For each notification, set a timeout to display it
            notifications.forEach(function (notif) {
                const key = notif.event_id + "," + notif.alarm_id;
                if (displayedNotifications.has(key)) {
                    return;
                }
                calendarNotifTimeouts[key] = browser.setTimeout(function () {
                    const notificationRemove = notification.add(notif.message, {
                        title: notif.title,
                        type: "warning",
                        sticky: true,
                        onClose: () => {
                            displayedNotifications.delete(key);
                        },
                        buttons: [
                            {
                                name: _t("OK"),
                                primary: true,
                                onClick: async () => {
                                    await rpc("/calendar/notify_ack");
                                    notificationRemove();
                                },
                            },
                            {
                                name: _t("Details"),
                                onClick: async () => {
                                    await action.doAction({
                                        type: 'ir.actions.act_window',
                                        res_model: 'calendar.event',
                                        res_id: notif.event_id,
                                        views: [[false, 'form']],
                                    }
                                    );
                                    notificationRemove();
                                },
                            },
                            {
                                name: _t("Snooze"),
                                onClick: () => {
                                    notificationRemove();
                                },
                            },
                        ],
                    });
                    displayedNotifications.add(key);
                }, notif.timer * 1000);
                lastNotifTimer = Math.max(lastNotifTimer, notif.timer);
            });

            // Set a timeout to get the next notifications when the last one has been displayed
            if (lastNotifTimer > 0) {
                nextCalendarNotifTimeout = browser.setTimeout(
                    getNextCalendarNotif,
                    lastNotifTimer * 1000
                );
            }
        }

        async function getNextCalendarNotif() {
            try {
                const result = await rpc("/calendar/notify", {}, { silent: true });
                displayCalendarNotification(result);
            } catch (error) {
                if (!(error instanceof ConnectionLostError)) {
                    throw error;
                }
            }
        }
    },
};

registry.category("services").add("calendarNotification", calendarNotificationService);

```

## File: static\src\views\ask_recurrence_update_policy_dialog.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { Dialog } from "@web/core/dialog/dialog";
import { Component } from "@odoo/owl";

export class AskRecurrenceUpdatePolicyDialog extends Component {
    setup() {
        this.possibleValues = {
            self_only: {
                checked: true,
                label: _t("This event"),
            },
            future_events: {
                checked: false,
                label: _t("This and following events"),
            },
            all_events: {
                checked: false,
                label: _t("All events"),
            },
        };
    }

    get selected() {
        return Object.entries(this.possibleValues).find((state) => state[1].checked)[0];
    }

    set selected(val) {
        this.possibleValues[this.selected].checked = false;
        this.possibleValues[val].checked = true;
    }

    confirm() {
        this.props.confirm(this.selected);
        this.props.close();
    }
}
AskRecurrenceUpdatePolicyDialog.template = "calendar.AskRecurrenceUpdatePolicyDialog";
AskRecurrenceUpdatePolicyDialog.components = {
    Dialog,
};

```

## File: static\src\views\ask_recurrence_update_policy_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="calendar.AskRecurrenceUpdatePolicyDialog">
        <t t-set="edit_recurrent_event">Edit Recurrent event</t>
        <Dialog size="'sm'" title="edit_recurrent_event">
            <t t-foreach="Object.entries(possibleValues)" t-as="value" t-key="value[0]">
                <t t-set="name" t-value="value[0]"/>
                <t t-set="state" t-value="value[1]"/>
                <div class="form-check o_radio_item">
                    <input name="recurrence-update" type="radio" class="form-check-input o_radio_input" t-att-checked="state.checked" t-att-value="name" t-att-id="name" t-on-click="(ev) => this.selected = ev.target.value"/>
                    <label class="form-check-label o_form_label" t-att-for="name" t-esc="state.label"/>
                </div>
            </t>
            <t t-set-slot="footer">
                <button class="btn btn-primary" t-on-click="confirm">
                    Confirm
                </button>
            </t>
        </Dialog>
    </t>
</templates>

```

## File: static\src\views\ask_recurrence_update_policy_hook.js

```javascript
/** @odoo-module **/

import { useService } from "@web/core/utils/hooks";
import { AskRecurrenceUpdatePolicyDialog } from "@calendar/views/ask_recurrence_update_policy_dialog";

export function askRecurrenceUpdatePolicy(dialogService) {
    return new Promise((resolve) => {
        dialogService.add(AskRecurrenceUpdatePolicyDialog, {
            confirm: resolve,
        }, {
            onClose: resolve.bind(null, false),
        });
    });
}

export function useAskRecurrenceUpdatePolicy() {
    const dialogService = useService("dialog");
    return askRecurrenceUpdatePolicy.bind(null, dialogService);
}

```

## File: static\src\views\attendee_calendar\attendee_calendar_controller.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { CalendarController } from "@web/views/calendar/calendar_controller";
import { useService } from "@web/core/utils/hooks";
import { onWillStart } from "@odoo/owl";
import { CalendarQuickCreate } from "@calendar/views/calendar_form/calendar_quick_create";
export class AttendeeCalendarController extends CalendarController {
    setup() {
        super.setup();
        this.actionService = useService("action");
        this.user = useService("user");
        this.orm = useService("orm");
        onWillStart(async () => {
            this.isSystemUser = await this.user.hasGroup('base.group_system');
        });
    }

    onClickAddButton() {
        this.actionService.doAction({
            type: 'ir.actions.act_window',
            res_model: 'calendar.event',
            views: [[false, 'form']],
        }, {
            additionalContext: this.props.context,
        });
    }

    goToFullEvent (resId, additionalContext) {
        this.actionService.doAction({
            type: 'ir.actions.act_window',
            res_model: 'calendar.event',
            views: [[false, 'form']],
            res_id: resId || false,
        }, {
            additionalContext: {
                ...this.props.context,
                ...additionalContext
            }
        });
    }

    getQuickCreateFormViewProps(record) {
        const props = super.getQuickCreateFormViewProps(record);
        const onDialogClosed = () => {
            this.model.load();
        };
        return {
            ...props,
            size: "md",
            goToFullEvent: (contextData) => {
                const fullContext = {
                    ...props.context,
                    ...contextData
                };
                this.goToFullEvent(false, fullContext)
            },
            onRecordSaved: () => onDialogClosed(),
        };
    }

    async editRecord(record, context = {}) {
        if (record.id) {
            return this.goToFullEvent(record.id, context);
        }
    }

    /**
     * @override
     *
     * If the event is deleted by the organizer, the event is deleted, otherwise it is declined.
     */
    deleteRecord(record) {
        if (this.user.partnerId === record.attendeeId && this.user.partnerId === record.rawRecord.partner_id[0]) {
            if (record.rawRecord.recurrency) {
                this.openRecurringDeletionWizard(record);
            } else {
                super.deleteRecord(...arguments);
            }
        } else {
            // Decline event
            this.orm.call(
                "calendar.attendee",
                "do_decline",
                [record.calendarAttendeeId],
            ).then(this.model.load.bind(this.model));
        }
    }

    openRecurringDeletionWizard(record) {
        this.actionService.doAction({
            type: 'ir.actions.act_window',
            res_model: 'calendar.popover.delete.wizard',
            views: [[false, 'form']],
            view_mode: "form",
            name: 'Delete Recurring Event',
            context: {'default_record': record.id},
            target: 'new'
        }, {
            onClose: () => {
                location.reload();
            },
        });
    }

    configureCalendarProviderSync(providerName) {
        this.actionService.doAction({
            name: _t('Connect your Calendar'),
            type: 'ir.actions.act_window',
            res_model: 'calendar.provider.config',
            views: [[false, "form"]],
            view_mode: "form",
            target: 'new',
            context: {
                'default_external_calendar_provider': providerName,
                'dialog_size': 'medium',
            }
        });
    }
}
AttendeeCalendarController.template = "calendar.AttendeeCalendarController";
AttendeeCalendarController.components = {
    ...AttendeeCalendarController.components,
    QuickCreateFormView: CalendarQuickCreate,
}

```

## File: static\src\views\attendee_calendar\attendee_calendar_controller.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="calendar.AttendeeCalendarController" t-inherit="web.CalendarController" t-inherit-mode="primary">
        <xpath expr="//Layout" position="inside">
            <t t-set-slot="control-panel-create-button">
                <button class="btn btn-primary o-calendar-button-new me-1" t-on-click="onClickAddButton">New</button>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\attendee_calendar\attendee_calendar_model.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { CalendarModel } from "@web/views/calendar/calendar_model";
import { askRecurrenceUpdatePolicy } from "@calendar/views/ask_recurrence_update_policy_hook";
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { onWillStart } from "@odoo/owl";

export class AttendeeCalendarModel extends CalendarModel {
    setup(params, { dialog, rpc }) {
        super.setup(...arguments);
        this.dialog = dialog;
        this.rpc = rpc;
        onWillStart(async () => {
            this.credentialStatus = await this.rpc("/calendar/check_credentials");
        });
    }

    get attendees() {
        return this.data.attendees;
    }

    /**
     * @override
     *
     * Upon updating a record with recurrence, we need to ask how it will affect recurrent events.
     */
    async updateRecord(record) {
        const rec = this.records[record.id];
        if (rec.rawRecord.recurrency) {
            const recurrenceUpdate = await askRecurrenceUpdatePolicy(this.dialog);
            if (!recurrenceUpdate) {
                return this.notify();
            }
            record.recurrenceUpdate = recurrenceUpdate;
        }
        return await super.updateRecord(...arguments);
    }

    /**
     * @override
     */
    buildRawRecord(partialRecord, options = {}) {
        const result = super.buildRawRecord(...arguments);
        if (partialRecord.recurrenceUpdate) {
            result.recurrence_update = partialRecord.recurrenceUpdate;
        }
        return result;
    }

    /**
     * @override
     */


    /**
     * @override
     */
    async updateData(data) {
        await super.updateData(...arguments);
        await this.updateAttendeeData(data);
    }

    /**
     * Split the events to display an event for each attendee with the correct status.
     * If the all filter is activated, we don't display an event for each attendee and keep
     * the previous behavior to display a single event.
     */
    async updateAttendeeData(data) {
        const attendeeFilters = data.filterSections.partner_ids;
        let isEveryoneFilterActive = false;
        let attendeeIds = [];
        const eventIds = Object.keys(data.records).map(id => Number.parseInt(id));
        if (attendeeFilters) {
            const allFilter = attendeeFilters.filters.find(filter => filter.type === "all")
            isEveryoneFilterActive = allFilter && allFilter.active || false;
            attendeeIds = attendeeFilters.filters.filter(filter => filter.type !== "all" && filter.value).map(filter => filter.value);
        }
        data.attendees = await this.orm.call(
            "res.partner",
            "get_attendee_detail",
            [attendeeIds, eventIds],
        );
        const currentPartnerId = this.user.partnerId;
        if (!isEveryoneFilterActive) {
            const activeAttendeeIds = new Set(attendeeFilters.filters
                .filter(filter => filter.type !== "all" && filter.value && filter.active)
                .map(filter => filter.value)
            );
            // Duplicate records per attendee
            const newRecords = {};
            let duplicatedRecordIdx = -1;
            for (const event of Object.values(data.records)) {
                const eventData = event.rawRecord;
                const attendees = eventData.partner_ids && eventData.partner_ids.length ? eventData.partner_ids : [eventData.partner_id[0]];
                let duplicatedRecords = 0;
                for (const attendee of attendees) {
                    if (!activeAttendeeIds.has(attendee)) {
                        continue;
                    }
                    // Records will share the same rawRecord.
                    const record = { ...event };
                    const attendeeInfo = data.attendees.find(a => (
                        a.id === attendee &&
                        a.event_id === event.id
                    ));
                    record.attendeeId = attendee;
                    // Colors are linked to the partner_id but in this case we want it linked
                    // to attendeeId
                    record.colorIndex = attendee;
                    if (attendeeInfo) {
                        record.attendeeStatus = attendeeInfo.status;
                        record.isAlone = attendeeInfo.is_alone;
                        record.isCurrentPartner = attendeeInfo.id === currentPartnerId;
                        record.calendarAttendeeId = attendeeInfo.attendee_id;
                    }
                    const recordId = duplicatedRecords ? duplicatedRecordIdx-- : record.id;
                    // Index in the records
                    record._recordId = recordId;
                    newRecords[recordId] = record;
                    duplicatedRecords++;
                }
            }
            data.records = newRecords;
        } else {
            for (const event of Object.values(data.records)) {
                const eventData = event.rawRecord;
                event.attendeeId = eventData.partner_id && eventData.partner_id[0]
                const attendeeInfo = data.attendees.find(a => (
                    a.id === currentPartnerId &&
                    a.event_id === event.id
                ));
                if (attendeeInfo) {
                    event.isAlone = attendeeInfo.is_alone;
                    event.calendarAttendeeId = attendeeInfo.attendee_id;
                }
            }
        }
    }

    /**
     * Archives a record, ask for the recurrence update policy in case of recurrent event.
     */
    async archiveRecord(record) {
        let recurrenceUpdate = false;
        if (record.rawRecord.recurrency) {
            recurrenceUpdate = await askRecurrenceUpdatePolicy(this.dialog);
            if (!recurrenceUpdate) {
                return;
            }
        } else {
            const confirm = await new Promise((resolve) => {
                this.dialog.add(ConfirmationDialog, {
                    body: _t("Are you sure you want to delete this record?"),
                    confirm: resolve.bind(null, true),
                }, {
                    onClose: resolve.bind(null, false),
                });
            })
            if (!confirm) {
                return;
            }
        }
        await this._archiveRecord(record.id, recurrenceUpdate);
    }

    async _archiveRecord(id, recurrenceUpdate) {
        if (!recurrenceUpdate && recurrenceUpdate !== "self_only") {
            await this.orm.call(
                this.resModel,
                "action_archive",
                [[id]],
            );
        } else {
            await this.orm.call(
                this.resModel,
                "action_mass_archive",
                [[id], recurrenceUpdate],
            );
        }
        await this.load();
    }
}
AttendeeCalendarModel.services = [...CalendarModel.services, "dialog", "orm"];

```

## File: static\src\views\attendee_calendar\attendee_calendar_renderer.js

```javascript
/** @odoo-module **/

import { CalendarRenderer } from "@web/views/calendar/calendar_renderer";
import { AttendeeCalendarCommonRenderer } from "@calendar/views/attendee_calendar/common/attendee_calendar_common_renderer";
import { AttendeeCalendarYearRenderer } from "@calendar/views/attendee_calendar/year/attendee_calendar_year_renderer";

export class AttendeeCalendarRenderer extends CalendarRenderer {}
AttendeeCalendarRenderer.components = {
    ...CalendarRenderer.components,
    day: AttendeeCalendarCommonRenderer,
    week: AttendeeCalendarCommonRenderer,
    month: AttendeeCalendarCommonRenderer,
    year: AttendeeCalendarYearRenderer,
};

```

## File: static\src\views\attendee_calendar\attendee_calendar_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { calendarView } from "@web/views/calendar/calendar_view";
import { AttendeeCalendarController } from "@calendar/views/attendee_calendar/attendee_calendar_controller";
import { AttendeeCalendarModel } from "@calendar/views/attendee_calendar/attendee_calendar_model";
import { AttendeeCalendarRenderer } from "@calendar/views/attendee_calendar/attendee_calendar_renderer";

export const attendeeCalendarView = {
    ...calendarView,
    Controller: AttendeeCalendarController,
    Model: AttendeeCalendarModel,
    Renderer: AttendeeCalendarRenderer,
};

registry.category("views").add("attendee_calendar", attendeeCalendarView);

```

## File: static\src\views\attendee_calendar\common\attendee_calendar_common_popover.js

```javascript
/** @odoo-module **/

import { onWillStart } from "@odoo/owl";
import { CalendarCommonPopover } from "@web/views/calendar/calendar_common/calendar_common_popover";
import { useService } from "@web/core/utils/hooks";
import { useAskRecurrenceUpdatePolicy } from "@calendar/views/ask_recurrence_update_policy_hook";
import { Dropdown } from "@web/core/dropdown/dropdown";
import { DropdownItem } from "@web/core/dropdown/dropdown_item";

export class AttendeeCalendarCommonPopover extends CalendarCommonPopover {
    setup() {
        super.setup();
        this.user = useService("user");
        this.orm = useService("orm");
        this.actionService = useService("action");
        this.askRecurrenceUpdatePolicy = useAskRecurrenceUpdatePolicy();

        onWillStart(this.onWillStart);
    }

    async onWillStart() {
        // Show status dropdown if user is in attendees list
        if (this.isEventEditable) {
            const stateSelections = await this.env.services.orm.call(
                this.props.model.resModel,
                "get_state_selections",
            );
            this.statusColors = {
                accepted: "text-success",
                declined: "text-danger",
                tentative: "text-muted",
                needsAction: "false",
            };
            this.statusInfo = {};
            for (const selection of stateSelections) {
                this.statusInfo[selection[0]] = {
                    text: selection[1],
                    color: this.statusColors[selection[0]],
                };
            }
            this.selectedStatusInfo = this.statusInfo[this.props.record.attendeeStatus];
        }
    }

    get isCurrentUserAttendee() {
        return this.props.record.rawRecord.partner_ids.includes(this.user.partnerId) || this.props.record.rawRecord.partner_id[0] === this.user.partnerId;
    }

    get isCurrentUserOrganizer() {
        return this.props.record.rawRecord.partner_id[0] === this.user.partnerId;
    }

    get isEventPrivate() {
        return this.props.record.rawRecord.privacy === "private";
    }

    get displayAttendeeAnswerChoice() {
        return (
            this.props.record.rawRecord.partner_ids.some((partner) => partner !== this.user.partnerId) &&
            this.props.record.isCurrentPartner
        );
    }

    get isEventDetailsVisible() {
        return this.isEventPrivate ? this.isEventEditable : true;
    }

    get isEventArchivable() {
        return false;
    }
    
    async onClickOpenRecord() {
        const action = await this.orm.call(
            'calendar.event', 
            'action_open_calendar_event', 
            [this.props.record.id]
        );
        this.actionService.doAction(action); 
    }

    /**
     * @override
     */
    get isEventDeletable() {
        return super.isEventDeletable && this.isEventEditable && !this.isEventArchivable;
    }

    /**
     * @override
     */
    get isEventEditable() {
        return this.props.record.rawRecord.user_can_edit;
    }

    get isEventViewable() {
        return this.isEventPrivate ? this.isEventEditable : super.isEventEditable;
    }

    /**
     * @override
     */
     get hasFooter() {
        return this.isEventViewable || super.hasFooter;
     }

    async changeAttendeeStatus(selectedStatus) {
        const record = this.props.record;
        if (record.attendeeStatus === selectedStatus) {
            return this.props.close();
        }
        let recurrenceUpdate = false;
        if (record.rawRecord.recurrency) {
            recurrenceUpdate = await this.askRecurrenceUpdatePolicy();
            if (!recurrenceUpdate) {
                return this.props.close();
            }
        }
        await this.env.services.orm.call(
            this.props.model.resModel,
            "change_attendee_status",
            [[record.id], selectedStatus, recurrenceUpdate],
        );
        await this.props.model.load();
        this.props.close();
    }

    async onClickArchive() {
        this.props.close();
        await this.props.model.archiveRecord(this.props.record);
    }
}
AttendeeCalendarCommonPopover.components = {
    ...CalendarCommonPopover.components,
    Dropdown,
    DropdownItem,
};
AttendeeCalendarCommonPopover.subTemplates = {
    ...CalendarCommonPopover.subTemplates,
    body: "calendar.AttendeeCalendarCommonPopover.body",
    footer: "calendar.AttendeeCalendarCommonPopover.footer",
};

```

## File: static\src\views\attendee_calendar\common\attendee_calendar_common_popover.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="calendar.AttendeeCalendarCommonPopover.body" t-inherit="web.CalendarCommonPopover.body" t-inherit-mode="primary">
        <xpath expr="//ul[hasclass('o_cw_popover_fields_secondary')]" position="attributes">
            <attribute name="t-if">isEventDetailsVisible</attribute>
        </xpath>
        <xpath expr="//div[hasclass('flex-grow-1')]" position="attributes">
            <attribute name="t-if">!fieldInfo.options.shouldOpenRecord</attribute>
        </xpath>
        <xpath expr="//div[hasclass('flex-grow-1')]" position="after">
            <t t-if="fieldInfo.options.shouldOpenRecord">
                <a href="#" t-on-click="onClickOpenRecord">
                    <t t-out="props.record.rawRecord.res_model_name"/>
                </a>
        </t>
        </xpath>
    </t>

    <t t-name="calendar.AttendeeCalendarCommonPopover.footer" t-inherit="web.CalendarCommonPopover.footer" t-inherit-mode="primary">
        <xpath expr="//t[@t-if='isEventEditable']" position="after">
            <t t-elif="isEventViewable">
                <a href="#" class="btn btn-primary o_cw_popover_edit" t-on-click="onEditEvent">View</a>
            </t>
        </xpath>
        <xpath expr="//t[@t-if='isEventDeletable']" position="after">
            <a t-if="isEventArchivable and isEventDetailsVisible" href="#" class="btn btn-secondary o_cw_popover_archive_g" t-on-click="onClickArchive">Delete</a>
            <t t-if="displayAttendeeAnswerChoice">
                <div class="btn-group w-100 w-lg-auto ms-lg-auto order-first order-lg-0 px-2 px-lg-0">
                    <button class="btn"
                            t-attf-class="#{selectedStatusInfo.text === 'Yes' ? 'btn-secondary active' : 'btn-outline-secondary'}"
                            t-on-click="() => this.changeAttendeeStatus('accepted')">Yes</button>
                    <button class="btn"
                            t-attf-class="#{selectedStatusInfo.text === 'No' ? 'btn-secondary active' : 'btn-outline-secondary'}"
                            t-on-click="() => this.changeAttendeeStatus('declined')">No</button>
                    <button class="btn"
                            t-attf-class="#{selectedStatusInfo.text === 'Maybe' ? 'btn-secondary active' : 'btn-outline-secondary'}"
                            t-on-click="() => this.changeAttendeeStatus('tentative')">Maybe</button>
                </div>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\attendee_calendar\common\attendee_calendar_common_renderer.js

```javascript
/** @odoo-module **/

import { useService } from "@web/core/utils/hooks";
import { CalendarCommonRenderer } from "@web/views/calendar/calendar_common/calendar_common_renderer";
import { AttendeeCalendarCommonPopover } from "@calendar/views/attendee_calendar/common/attendee_calendar_common_popover";

export class AttendeeCalendarCommonRenderer extends CalendarCommonRenderer {

    setup() {
        super.setup();
        this.user = useService("user");
    }
    /**
     * @override
     *
     * Give a new key to our fc records to be able to iterate through in templates
     */
    convertRecordToEvent(record) {
        let editable = false;
        if (record && record.rawRecord) {
            editable = record.rawRecord.user_can_edit;
        }
        return {
            ...super.convertRecordToEvent(record),
            id: record._recordId || record.id,
            editable: editable,
        };
    }

    /**
     * @override
     */
    onEventRender(info) {
        super.onEventRender(...arguments);
        const { el, event } = info;
        const record = this.props.model.records[event.id];

        if (record) {
            if (this.env.searchModel?.context?.default_calendar_event_id === parseInt(event.id)) {
                this.openPopover(info.el, record);
            }
            if (record.rawRecord.is_highlighted) {
                el.classList.add("o_event_highlight");
            }
            if (record.isAlone) {
                el.classList.add("o_attendee_status_alone");
            } else {
                el.classList.add(`o_attendee_status_${record.attendeeStatus}`);
            }
        }
    }

    /**
     * @override
     *
     * Allow slots to be selected over multiple days
     */
    isSelectionAllowed(event) {
        return true;
    }
}
AttendeeCalendarCommonRenderer.eventTemplate = "calendar.AttendeeCalendarCommonRenderer.event";
AttendeeCalendarCommonRenderer.components = {
    ...CalendarCommonRenderer.components,
    Popover: AttendeeCalendarCommonPopover,
};

```

## File: static\src\views\attendee_calendar\common\attendee_calendar_common_renderer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="calendar.AttendeeCalendarCommonRenderer.event">
        <div class="fc-content" t-att-class="isMonth ? 'd-flex gap-1 text-truncate' : ''">
            <span t-if="!isTimeHidden and isMonth" class="d-none d-lg-inline fc-time fw-normal" t-esc="startTime" />
            <div t-attf-class="o_event_title text-truncate">
                <i t-if="isAlone" class="fa fa-exclamation-circle align-self-top me-1" data-tooltip="You're alone in this meeting"/>
                <span t-esc="title" t-attf-class="#{(attendeeStatus == 'accepted' || rawRecord.is_highlighted)  ? 'fw-bolder' : ''}"/>
                    <t t-if="!isAllDay and !isMonth and !isSmall">
                        <t t-if="24 > duration and duration > 0.5">
                            <br/><span class="fw-normal mt-1"><t t-esc="startTime"/> - <t t-esc="endTime"/></span>
                        </t>
                        <t t-else="">
                            , <t t-esc="startTime"/>
                        </t>
                    </t>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\views\attendee_calendar\year\attendee_calendar_year_popover.js

```javascript
/** @odoo-module **/

import { CalendarYearPopover } from "@web/views/calendar/calendar_year/calendar_year_popover";

export class AttendeeCalendarYearPopover extends CalendarYearPopover {
    getRecordClass(record) {
        const classes = [super.getRecordClass(record)];
        if (record.isAlone) {
            classes.push("o_attendee_status_alone");
        } else {
            classes.push(`o_attendee_status_${record.attendeeStatus}`);
        }
        return classes.join(" ");
    }
}
AttendeeCalendarYearPopover.subTemplates = {
    ...CalendarYearPopover.subTemplates,
    body: "calendar.AttendeeCalendarYearPopover.body",
};

```

## File: static\src\views\attendee_calendar\year\attendee_calendar_year_popover.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="calendar.AttendeeCalendarYearPopover.body" t-inherit="web.CalendarYearPopover.body" t-inherit-mode="primary">
        <!-- Since we duplicate the event this is needed to avoid duplicated key errors -->
        <xpath expr="//t[@t-foreach='recordGroup.records']" position="attributes">
            <attribute name="t-key">record._recordId</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\attendee_calendar\year\attendee_calendar_year_renderer.js

```javascript
/** @odoo-module **/

import { CalendarYearRenderer } from "@web/views/calendar/calendar_year/calendar_year_renderer";
import { AttendeeCalendarYearPopover } from "@calendar/views/attendee_calendar/year/attendee_calendar_year_popover";

export class AttendeeCalendarYearRenderer extends CalendarYearRenderer {}
AttendeeCalendarYearRenderer.components = {
    ...CalendarYearRenderer.components,
    Popover: AttendeeCalendarYearPopover,
};

```

## File: static\src\views\calendar_form\calendar_form_controller.js

```javascript
/** @odoo-module **/

import { FormController } from "@web/views/form/form_controller";
import { useService } from "@web/core/utils/hooks";
import { onWillStart } from "@odoo/owl";

export class CalendarFormController extends FormController {
    setup() {
        super.setup();
        const ormService = useService("orm");

        onWillStart(async () => {
            this.discussVideocallLocation = await ormService.call(
                "calendar.event",
                "get_discuss_videocall_location"
            );
        });
    }

    /**
     * @override
     */
    async beforeExecuteActionButton(clickParams) {
        const action = clickParams.name;
        if (action == "clear_videocall_location" || action === "set_discuss_videocall_location") {
            let newVal = "";
            let videoCallSource = "custom";
            let changes = {};
            if (action === "set_discuss_videocall_location") {
                newVal = this.discussVideocallLocation;
                videoCallSource = "discuss";
                changes.access_token = this.discussVideocallLocation.split("/").pop();
            }
            changes = Object.assign(changes, {
                videocall_location: newVal,
                videocall_source: videoCallSource,
            });
            this.model.root.update(changes);
            return false; // no continue
        }
        return super.beforeExecuteActionButton(...arguments);
    }
}

```

## File: static\src\views\calendar_form\calendar_form_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { formView } from "@web/views/form/form_view";
import { CalendarFormController } from "@calendar/views/calendar_form/calendar_form_controller";

export const CalendarFormView = {
    ...formView,
    Controller: CalendarFormController,
};

registry.category("views").add("calendar_form", CalendarFormView);

```

## File: static\src\views\calendar_form\calendar_quick_create.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { FormViewDialog } from "@web/views/view_dialogs/form_view_dialog";
import { CalendarFormView } from "./calendar_form_view";
import { CalendarFormController } from "./calendar_form_controller";
import { serializeDate, serializeDateTime } from "@web/core/l10n/dates";

const QUICK_CREATE_CALENDAR_EVENT_FIELDS = {
    name: { type: "string" },
    start: { type: "datetime" },
    start_date: { type: "date" },
    stop_date: { type: "date" },
    stop: { type: "datetime" },
    allday: { type: "boolean" },
    partner_ids: { type: "many2many" },
    videocall_location: { type: "string" },
    description: { type: "string" }
};

function getDefaultValuesFromRecord(data) {
    const context = {};
    for (let fieldName in QUICK_CREATE_CALENDAR_EVENT_FIELDS) {
        if (fieldName in data) {
            let value = data[fieldName];
            const { type } = QUICK_CREATE_CALENDAR_EVENT_FIELDS[fieldName]
            if (type === 'many2many') {
                value = value.records.map((record) => record.resId);
            } else if (type === 'date') {
                value = value && serializeDate(value);
            } else if (type === "datetime") {
                value = value && serializeDateTime(value);
            }
            context[`default_${fieldName}`] = value || false;
        }
    }
    return context;
}

export class CalendarQuickCreateFormController extends CalendarFormController {
    static props = {
        ...CalendarFormController.props,
        goToFullEvent: Function,
    };

    goToFullEvent() {
        const context = getDefaultValuesFromRecord(this.model.root.data)
        this.props.goToFullEvent(context);
    }
}

registry.category("views").add("calendar_quick_create_form_view", {
    ...CalendarFormView,
    Controller: CalendarQuickCreateFormController,
});

export class CalendarQuickCreate extends FormViewDialog {
    static props = {
        ...FormViewDialog.props,
        goToFullEvent: Function,
    };

    setup() {
        super.setup();
        Object.assign(this.viewProps, {
            ...this.viewProps,
            buttonTemplate: "calendar.CalendarQuickCreateButtons",
            goToFullEvent: (contextData) => {
                this.props.goToFullEvent(contextData);
            }
        });
    }
}

```

## File: static\src\views\calendar_form\calendar_quick_create.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="calendar.CalendarQuickCreateButtons" t-inherit="web.FormViewDialog.ToOne.buttons">
        <xpath expr="//button[contains(@class, 'o_form_button_cancel')]" position="after">
            <button class="btn btn-link ms-auto" t-on-click="goToFullEvent">More Options</button>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\fields\attendee_tags_list.js

```javascript
/** @odoo-module **/

import { TagsList } from "@web/core/tags_list/tags_list";

export class AttendeeTagsList extends TagsList {}

AttendeeTagsList.template = "calendar.AttendeeTagsList";

```

## File: static\src\views\fields\attendee_tags_list.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="calendar.AttendeeTagsList" t-inherit="web.TagsList" t-inherit-mode="primary">
        <xpath expr="//span[hasclass('o_avatar_backdrop')]" position="attributes">
            <attribute name="class" remove="opacity-100-hover" separator=" "/>
        </xpath>

        <xpath expr="//img[hasclass('o_m2m_avatar')]" position="replace">
            <div class="o_attendee_tags_list">
                <img t-if="tag.img"
                    t-att-src="tag.img"
                    class="o_avatar o_m2m_avatar position-relative rounded"
                    t-att-class="tag.imageClass"/>
                <div t-if="tag.status &amp;&amp; tag.statusIcon" t-attf-class="attendee_tag_status o_attendee_status_{{tag.status}} fa fa-fw {{tag.statusIcon}} rounded-circle"/>
            </div>
        </xpath>

        <xpath expr="//span[hasclass('o_m2m_avatar_empty')]" position="attributes">
            <attribute name="class" remove="text-center" separator=" "/>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\fields\many2many_attendee.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import {
    Many2ManyTagsAvatarField,
    many2ManyTagsAvatarField,
} from "@web/views/fields/many2many_tags_avatar/many2many_tags_avatar_field";
import { useSpecialData } from "@web/views/fields/relational_utils";
import { AttendeeTagsList } from "@calendar/views/fields/attendee_tags_list";

const ICON_BY_STATUS = {
    accepted: "fa-check",
    declined: "fa-times",
    tentative: "fa-question",
};
export class Many2ManyAttendee extends Many2ManyTagsAvatarField {
    setup() {
        super.setup();
        this.specialData = useSpecialData((orm, props) => {
            const { context, name, record } = props;
            return orm.call(
                "res.partner",
                "get_attendee_detail",
                [record.data[name].records.map((rec) => rec.resId), [record.resId || false]],
                {
                    context,
                }
            );
        });
    }

    get tags() {
        const partnerIds = this.specialData.data;
        const tags = super.tags.map((tag) => {
            const partner = partnerIds.find((partner) => tag.resId === partner.id);
            if (partner) {
                tag.status = partner.status;
                tag.statusIcon = ICON_BY_STATUS[partner.status];
            }
            return tag;
        });

        const organizer = partnerIds.find((partner) => partner.is_organizer);
        if (organizer) {
            const orgId = organizer.id;
            // sort elements according to the partner id
            tags.sort((a, b) => {
                const a_org = a.resId === orgId;
                return a_org ? -1 : 1;
            });
        }
        return tags;
    }
}

Many2ManyAttendee.template = "calendar.Many2ManyAttendee";
Many2ManyAttendee.components = {
    ...Many2ManyAttendee.components,
    TagsList: AttendeeTagsList,
};

export const many2ManyAttendee = {
    ...many2ManyTagsAvatarField,
    component: Many2ManyAttendee,
    additionalClasses: ["o_field_many2many_tags_avatar", "w-100"],
};

registry.category("fields").add("many2manyattendee", many2ManyAttendee);

```

## File: static\src\views\fields\many2many_attendee.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="calendar.Many2ManyAttendee" t-inherit="web.Many2ManyTagsAvatarField" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('many2many_tags_avatar_field_container')]" position="attributes">
            <attribute name="class" add="flex-column" separator=" "/>
        </xpath>
        <xpath expr="//Many2XAutocomplete" position="attributes">
            <attribute name="placeholder">props.placeholder</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\fields\many2many_attendee_expandable.js

```javascript
/** @odoo-module **/

import { useState, useEffect } from "@odoo/owl";
import { registry } from "@web/core/registry";
import { reposition } from "@web/core/position_hook";
import {
    Many2ManyAttendee,
    many2ManyAttendee,
} from "@calendar/views/fields/many2many_attendee";

export class Many2ManyAttendeeExpandable extends Many2ManyAttendee {
    state = useState({ expanded: false });

    setup() {
        super.setup();
        this.attendeesCount = this.props.record.data.attendees_count;
        this.acceptedCount = this.props.record.data.accepted_count;
        this.declinedCount = this.props.record.data.declined_count;
        this.uncertainCount = this.attendeesCount - this.acceptedCount - this.declinedCount;

        if (!this.env.isSmall) {
            useEffect(() => {
                const popover = document.querySelector(".o_field_many2manyattendeeexpandable")
                    .closest(".o_popover");
                const target = document.querySelector(`.fc-event[data-event-id="${this.props.record.resId}"]`);
                reposition(popover, target, { position: "right", margin: 0 });
            }, () => [ this.state.expanded ]);
        }
    }

    onExpanderClick() {
        this.state.expanded = !this.state.expanded;
    }
}

Many2ManyAttendeeExpandable.template = "calendar.Many2ManyAttendeeExpandable";

export const many2ManyAttendeeExpandable = {
    ...many2ManyAttendee,
    component: Many2ManyAttendeeExpandable,
};

registry.category("fields").add("many2manyattendeeexpandable", many2ManyAttendeeExpandable);

```

## File: static\src\views\fields\many2many_attendee_expandable.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="calendar.Many2ManyAttendeeExpandable" t-inherit="web.Many2ManyTagsAvatarField" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('many2many_tags_avatar_field_container')]" position="attributes">
            <attribute name="class" remove="mw-100" add="w-100" separator=" "/>
        </xpath>
        <xpath expr="//TagsList" position="attributes">
            <attribute name="itemsVisible">state.expanded ? tags.length : 5</attribute>
        </xpath>
        <xpath expr="//TagsList" position="before">
            <t t-if="tags.length > 5">
                <div class="d-flex w-100">
                    <div class="flex-grow-1">
                        <strong><t t-out="attendeesCount"/> attendees</strong>
                        <br/><t t-out="acceptedCount"/> accepted
                        <br/><t t-out="declinedCount"/> declined
                        <br/><t t-out="uncertainCount"/> uncertain
                    </div>
                    <i role="button" t-on-click="onExpanderClick" t-att-class="state.expanded ? 'oi oi-chevron-up' : 'oi oi-chevron-down'"></i>
                </div>
            </t>
        </xpath>
        <xpath expr="//TagsList" position="replace">
            <t t-if="!state.expanded &amp;&amp; tags.length > 10"> </t>
            <t t-else="">$0</t>
        </xpath>
    </t>

</templates>

```

## File: static\src\views\widgets\calendar_week_days.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { WeekDays, weekDays } from "@web/views/widgets/week_days/week_days";

export class CalendarWeekDays extends WeekDays {
    onChange(day) {
        this.props.record.update({ [day]: !this.data[day] });
    }
};
CalendarWeekDays.template = 'calendar.WeekDays';

export const calendarWeekDays = {
    component: CalendarWeekDays,
    fieldDependencies: weekDays.fieldDependencies,
};

registry.category("view_widgets").add("calendar_week_days", calendarWeekDays);

```

## File: static\src\views\widgets\calendar_week_days.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="calendar.WeekDays">
        <div class="o_recurrent_weekdays mb-2 d-flex">
        <t t-foreach="weekdays" t-as="day" t-key="day">
                <div t-attf-class="o_calendar_week_days_rounded {{data[day] ? 'text-bg-action' : 'btn-secondary'}}"
                    t-on-click="this.props.readonly? () => {} : () => this.onChange(day)">
                    <t t-esc="props.record.fields[day].string[0]"/>
                </div>
            </t>
        </div>
    </t>
</templates>

```

## File: views\calendar_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Template rendered in route auth=None, for anonymous user. This allow them to see meeting details -->
    <template id="invitation_page_anonymous" name="Calendar Invitation Page for anonymous users">
        <t t-call="web.layout">
            <t t-set="head">
                <t t-call-assets="web.assets_frontend" t-js="false"/>
            </t>

            <div class="container">
                <div class="o_logo">
                    <img t-attf-src="/web/binary/company_logo?company={{ company.id }}" alt="Logo"/>
                </div>

                <div class="card">
                    <div class="card-header">
                        <h2>Calendar Invitation <small><t t-esc="event.name"/></small></h2>
                    </div>
                    <div class="card-body">
                        <div class="clearfix mb16" t-if="attendee.state != 'needsAction'">
                            <span class="float-end badge text-bg-info">
                                <t t-if="attendee.state == 'accepted'">Yes I'm going.</t>
                                <t t-if="attendee.state == 'declined'">No I'm not going.</t>
                                <t t-if="attendee.state == 'tentative'">Tentative</t>
                                <t t-if="attendee.state == 'needsAction'">No feedback yet</t>
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
                                <tr>
                                    <th>Description</th>
                                    <td><t t-esc="event.description or '-'"/></td>
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
            <tree string="Meeting Types" sample="1" editable="bottom">
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
            <tree string="Calendar Alarm" sample="1">
                <field name="name" column_invisible="True"/>
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
                <sheet>
                    <group>
                        <group name="left_details">
                            <field name="name" invisible="1"/>
                            <field name="alarm_type"/>
                            <field name="mail_template_id" context="{'default_model': 'calendar.event'}"
                                   invisible="alarm_type != 'email'"
                                   required="alarm_type == 'email'"/>
                            <field name="body" invisible="alarm_type != 'notification'"/>
                        </group>
                        <group name="right_details">
                            <label for="duration"/>
                            <div class="o_row">
                                <field name="duration"/>
                                <field name="interval"/>
                            </div>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="action_calendar_alarm" model="ir.actions.act_window">
        <field name="name">Calendar Alarm</field>
        <field name="res_model">calendar.alarm</field>
        <field name="view_mode">tree,form</field>
        <field name="view_id" ref="view_calendar_alarm_tree"/>
    </record>

    <!-- Calendar Events : Views and Actions  -->
    <record id="view_calendar_event_tree" model="ir.ui.view">
        <field name="name">calendar.event.tree</field>
        <field name="model">calendar.event</field>
        <field name="arch" type="xml">
            <tree string="Meetings" sample="1" multi_edit="1">
                <header>
                    <button name="action_open_composer" type="object" context="{'default_composition_mode':'mass_mail'}"
                            string="Send Mail"/>
                </header>
                <field name="name" string="Subject" decoration-bf="1" readonly="recurrency"/>
                <field name="start" string="Start Date" readonly="1"/>
                <field name="stop" string="End Date" readonly="1"/>
                <field name="user_id" widget="many2one_avatar_user" readonly="recurrency" optional="hide"/>
                <field name="partner_ids" widget="many2many_tags" readonly="recurrency" optional="show"/>
                <field name="alarm_ids" widget="many2many_tags" optional="hide" readonly="recurrency"/>
                <field name="categ_ids" widget="many2many_tags" optional="hide" readonly="recurrency" options="{'color_field': 'color'}"/>
                <field name="recurrency" optional="hide" readonly="1"/>
                <field name="privacy" optional="hide" readonly="recurrency"/>
                <field name="show_as" optional="hide" readonly="recurrency"/>
                <field name="location" optional="show" readonly="recurrency"/>
                <field name="duration" widget="float_time" readonly="1"/>
                <field name="description" optional="hide" readonly="recurrency"/>
                <field name="allday" column_invisible="True"/>
                <field name="message_needaction" column_invisible="True"/>
            </tree>
        </field>
    </record>

    <record id="view_calendar_event_form" model="ir.ui.view">
        <field name="name">calendar.event.form</field>
        <field name="model">calendar.event</field>
        <field name="priority" eval="1"/>
        <field name="arch" type="xml">
            <form string="Meetings" js_class="calendar_form">
                <div invisible="not recurrence_id" class="alert alert-info oe_edit_only" role="status">
                    <p>Edit recurring event</p>
                    <field name="recurrence_update" widget="radio"/>
                </div>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button icon="fa-bars" type="object" name="action_open_calendar_event" invisible="not (res_id and res_model_name)">
                            <field name="res_model_name"/>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <field name="res_id" invisible="1" />
                    <field name="active" invisible="1"/>
                    <field name="user_can_edit" invisible="1"/>
                    <div class="oe_title mb-3">
                        <div>
                            <label for="name"/>
                        </div>
                        <h1>
                            <field name="name" placeholder="e.g. Business Lunch" readonly="not user_can_edit"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="start_date" widget="daterange" options="{'end_date_field': 'stop_date'}" invisible="not allday" required="allday" readonly="not user_can_edit"/>
                            <field name="start" widget="daterange" options="{'end_date_field': 'stop'}" invisible="allday" required="not allday" readonly="not user_can_edit"/>
                            <field name="stop_date" invisible="1"/>
                            <field name="stop" invisible="1" />
                            <label for="duration" class="fw-bold text-900 opacity-100"/>
                            <div class="d-flex gap-2">
                                <div class="d-flex align-items-start" invisible="allday">
                                    <field name="duration" widget="float_time" string="Duration" class="oe_inline pe-2" readonly="(id and recurrency) or not user_can_edit"/>
                                    <span> hours</span>
                                </div>
                                <span invisible="allday" style="white-space: pre;"> or </span>
                                <div class="d-flex gap-2">
                                    <label for="allday" class=""/>
                                    <field name="allday" class="oe_inline" widget="boolean_toggle" force_save="1" options="{'autosave': False}" readonly="not user_can_edit"/>
                                </div>
                            </div>
                            <field name="event_tz" invisible="not recurrency" readonly="not user_can_edit"/>


                            <field name="recurrence_id" invisible="1" />
                            <field name="rrule_type" invisible="1" />
                            <field name="recurrency" readonly="not user_can_edit"/>
                            <field name="rrule_type_ui" class="w-auto" invisible="not recurrency" required="recurrency" readonly="not user_can_edit"/>
                            <label for="interval" class="fw-bold text-900" invisible="not recurrency or rrule_type_ui != 'custom'"/>
                            <div class="d-flex gap-1" invisible="not recurrency or rrule_type_ui != 'custom'">
                                <field name="interval" string="Repeat every" class="oe_inline w-auto" />
                                <field name="rrule_type" nolabel="1" class="oe_inline" required="rrule_type_ui == 'custom'"/>
                            </div>
                            <span class="fw-bold text-nowrap" invisible="rrule_type_ui not in ['weekly', 'custom'] or (rrule_type_ui == 'custom' and rrule_type != 'weekly')">Repeat on</span>
                            <div invisible="rrule_type_ui not in ['weekly', 'custom'] or (rrule_type_ui == 'custom' and rrule_type != 'weekly')" ><widget name="calendar_week_days" readonly="not user_can_edit"/></div>
                            <field name="recurrence_id" invisible="1" />

                            <label string="Day of Month" for="month_by" class="fw-bold text-900" style="width: fit-content;" invisible="rrule_type_ui not in ['monthly', 'custom'] or (rrule_type_ui == 'custom' and rrule_type != 'monthly')"/>
                            <div class="d-flex gap-2" invisible="rrule_type_ui not in ['monthly', 'custom'] or (rrule_type_ui == 'custom' and rrule_type != 'monthly')">
                                <field name="month_by" nolabel="1" class="oe_inline w-auto" required="rrule_type_ui == 'monthly' or rrule_type == 'monthly'"/>
                                <field name="day" nolabel="1" class="oe_inline w-auto"
                                    required="(rrule_type_ui == 'monthly' or rrule_type == 'monthly') and month_by == 'date'"
                                    invisible="month_by != 'date'"
                                />
                                <field name="byday" string="The" class="oe_inline w-auto" nolabel="1"
                                    required="(rrule_type_ui == 'monthly' or rrule_type == 'monthly') and month_by == 'day'"
                                    invisible="month_by != 'day'"
                                />
                                <field name="weekday" nolabel="1" class="oe_inline w-auto"
                                    required="(rrule_type_ui == 'monthly' or rrule_type == 'monthly') and month_by == 'day'"
                                    invisible="month_by != 'day'"
                                />
                            </div>
                            <label string="Until" for="end_type" class="fw-bold text-900" invisible="not recurrency"/>
                            <div class="d-flex gap-2" invisible="not recurrency">
                                <field name="end_type" class="oe_inline w-auto" nolabel="1" required="recurrency" readonly="not user_can_edit"/>
                                <field name="count" class="oe_inline w-auto" nolabel="1" invisible="end_type != 'count'" required="recurrency" readonly="not user_can_edit"/>
                                <field name="until" class="oe_inline w-auto" nolabel="1" invisible="end_type != 'end_date'" readonly="not user_can_edit"
                                    required="recurrency and end_type == 'end_date'"
                                    placeholder="e.g: 12/31/2023"
                                />
                            </div>

                            <field name="location" placeholder="Online Meeting" readonly="not user_can_edit"/>
                            <label for="videocall_location" class="opacity-100"/>
                            <div col="2">
                                <field name="videocall_location" string="Videocall URL" widget="CopyClipboardChar" force_save="1" readonly="videocall_source == 'discuss' or not user_can_edit"/>
                                <button name="clear_videocall_location" type="object" class="btn btn-link ps-0"
                                    invisible="not videocall_location or not user_can_edit" context="{'recurrence_update': recurrence_update}">
                                    <span class="fa fa-times"></span><span> Clear meeting</span>
                                </button>
                                <button name="set_discuss_videocall_location" type="object" class="btn btn-link ps-0"
                                    invisible="videocall_location or not user_can_edit" context="{'recurrence_update': recurrence_update}">
                                    <span class="fa fa-plus"></span><span> Odoo meeting</span>
                                </button>
                                <button name="action_join_video_call" class="btn btn-link" help="Join Video Call" type="object" invisible="not videocall_location or not user_can_edit">
                                    <span class="oi oi-arrow-right"></span><span> Join video call</span>
                                </button>
                            </div>
                            <field name="videocall_source" invisible="1"/>
                            <field name="access_token" invisible="1" force_save="1"/>
                            <field name="categ_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}" readonly="not user_can_edit"/>
                            <label for="privacy"/>
                                    <div class="o_row">
                                        <field name="show_as" readonly="not user_can_edit" nolabel="1"/>
                                        <field name="privacy" readonly="not user_can_edit" nolabel="1"/>
                                    </div>
                                    <field name="user_id" widget="many2one_avatar_user" readonly="not user_can_edit"/>
                                    <field name="description" options="{'height': 100}" placeholder="Add description" readonly="not user_can_edit"/>
                        </group>
                        <group>
                            <field name="should_show_status" invisible="1"/>
                            <field name="current_status" class="w-auto" required="should_show_status" readonly="not should_show_status" invisible="not should_show_status"/>
                            <field name="alarm_ids" widget="many2many_tags" options="{'no_quick_create': True}" readonly="not user_can_edit"/>
                            <div class="o_row" colspan="2">
                                <div colspan="1">
                                    <field name="attendees_count" class="w-auto oe_inline" nolabel="1"/><span> Attendees</span>
                                </div>
                                <div name="send_buttons" class="d-flex gap-2 justify-content-end" colspan="1">
                                    <button name="action_open_composer" help="Send Email to attendees" type="object" string=" EMAIL" icon="fa-envelope" invisible="not user_can_edit"/>
                                </div>
                            </div>
                            <div colspan="2">
                                <field name="accepted_count" class="w-auto oe_inline" nolabel="1"/> yes,
                                <field name="tentative_count" class="w-auto oe_inline" nolabel="1"/> maybe,
                                <field name="declined_count" class="w-auto oe_inline" nolabel="1"/> no,
                                <field name="awaiting_count" class="w-auto oe_inline" nolabel="1"/> awaiting
                            </div>
                            <div class="d-flex align-items-baseline" colspan="2">
                                <field name="partner_ids" widget="many2manyattendee"
                                    placeholder="Select attendees..."
                                    options="{'no_quick_create': True}"
                                    context="{'form_view_ref': 'base.view_partner_simple_form'}"
                                    domain="[('type','!=','private')]"
                                    class="oe_inline"
                                    readonly="not user_can_edit"
                                />
                            </div>
                            <div class="alert alert-warning o_form_header mt-2" colspan="2" invisible="not invalid_email_partner_ids" role="status">
                                <p><strong>The following attendees have invalid email addresses and won't receive any email notifications:</strong></p>
                                <field name="invalid_email_partner_ids" widget="many2manyattendee" class="oe_inline"/>
                            </div>
                        </group>
                    </group>
                    <notebook>
                        <page name="page_invitations" string="Invitations" groups="base.group_no_one" invisible="not user_can_edit">
                            <button name="action_sendmail" type="object" string="Send Invitations" icon="fa-envelope" class="oe_link"/>
                            <field name="attendee_ids" widget="one2many" mode="tree,kanban" readonly="1">
                                <tree string="Invitation details" editable="top" create="false" delete="false">
                                    <field name="partner_id" />
                                    <field name="email" widget="email"/>
                                    <field name="phone" widget="phone"/>
                                    <field name="state" />
                                    <button name="do_tentative" invisible="state not in ('needsAction', 'declined', 'accepted')" string="Uncertain" type="object" icon="fa-asterisk"/>
                                    <button name="do_accept" string="Accept" invisible="state not in ('needsAction', 'tentative', 'declined')" type="object" icon="fa-check text-success"/>
                                    <button name="do_decline" string="Decline" invisible="state not in ('needsAction', 'tentative', 'accepted')" type="object" icon="fa-times-circle text-danger"/>
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

                                                <div class="text-end">
                                                    <button name="do_tentative" invisible="state not in ('needsAction', 'declined', 'accepted')" string="Uncertain" type="object" class="btn fa fa-asterisk"/>
                                                    <button name="do_accept" invisible="state not in ('needsAction', 'tentative', 'declined')" string="Accept" type="object" class="btn fa fa-check text-success"/>
                                                    <button name="do_decline" invisible="state not in ('needsAction', 'tentative', 'accepted')" string="Decline" type="object" class="btn fa fa-times-circle text-danger"/>
                                                </div>
                                            </div>
                                        </t>
                                    </templates>
                                </kanban>
                            </field>
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids" readonly="not user_can_edit"/>
                    <field name="message_ids" readonly="not user_can_edit"/>
                </div>
            </form>
        </field>
    </record>

    <record id="view_calendar_event_form_quick_create" model="ir.ui.view">
        <field name="name">calendar.event.form.quick_create</field>
        <field name="model">calendar.event</field>
        <field name="priority" eval="2"/>
        <field name="arch" type="xml">
            <form string="Meetings" js_class="calendar_quick_create_form_view">
                <sheet>
                    <field name="recurrence_update" invisible="1"/>
                    <field name="videocall_source" invisible="1"/>
                    <field name="access_token" invisible="1" force_save="1"/>
                    <div class="o_row">
                        <h1 class="w-100"><field name="name" nolabel="1" placeholder="Add title" colspan="2"/></h1>
                    </div>
                    <group>
                        <field name="start_date" widget="daterange" options="{'end_date_field': 'stop_date'}" invisible="not allday" required="allday"/>
                        <field name="start" widget="daterange" options="{'end_date_field': 'stop'}" invisible="allday" required="not allday"/>
                        <field name="stop_date" invisible="1" />
                        <field name="stop" invisible="1" />
                        <field name="allday"/>
                        <field name="partner_ids" widget="many2manyattendee"
                            placeholder="Add attendees..."
                            options="{'no_quick_create': True}"
                            context="{'form_view_ref': 'base.view_partner_simple_form'}"
                            class="oe_inline"
                        />
                        <label for="videocall_location" class="opacity-100"/>
                        <div col="2">
                            <field name="videocall_location" string="Videocall URL" widget="CopyClipboardChar" force_save="1" readonly="videocall_source == 'discuss'"/>
                            <button name="clear_videocall_location" type="object" class="btn btn-link p-0"
                                invisible="not videocall_location" context="{'recurrence_update': recurrence_update}">
                                <span class="fa fa-times"></span><span> Clear meeting</span>
                            </button>
                            <button name="set_discuss_videocall_location" type="object" class="btn btn-link p-0"
                                invisible="videocall_location" context="{'recurrence_update': recurrence_update}">
                                <span class="fa fa-plus"></span> <span>Odoo meeting</span>
                            </button>
                            <button name="action_join_video_call" class="btn btn-link" help="Join Video Call" type="object" invisible="not videocall_location">
                                <span class="oi oi-arrow-right"></span><span> Join video call</span>
                            </button>
                        </div>
                        <field name="description" placeholder="Describe your meeting"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="view_calendar_event_calendar" model="ir.ui.view">
        <field name="name">calendar.event.calendar</field>
        <field name="model">calendar.event</field>
        <field name="priority" eval="2"/>
        <field name="arch" type="xml">
            <calendar js_class="attendee_calendar" string="Meetings" banner_route="/onboarding/calendar" date_start="start" date_stop="stop" date_delay="duration" all_day="allday"
                event_open_popup="true"
                event_limit="5"
                quick_create="true"
                quick_create_view_id="%(calendar.view_calendar_event_form_quick_create)d"
                color="partner_ids">
                <field name="location" invisible="not location" options="{'icon': 'fa fa-map-marker'}"/>
                <field name="attendees_count" invisible="1"/>
                <field name="accepted_count" invisible="1"/>
                <field name="declined_count" invisible="1"/>
                <field name="user_can_edit" invisible="1"/>
                <field name="partner_ids" options="{'block': True, 'icon': 'fa fa-users'}"
                       filters="1" widget="many2manyattendeeexpandable" write_model="calendar.filters"
                       write_field="partner_id" filter_field="partner_checked" avatar_field="avatar_128"
                />
                <field name="videocall_location" widget="url" text="Join Video Call" options="{'icon': 'fa fa-lg fa-video-camera'}" invisible="not videocall_location"/>
                <field name="is_highlighted" invisible="1"/>
                <field name="is_organizer_alone" invisible="1"/>
                <field name="display_description" invisible="1"/>
                <field name="description" invisible="not display_description" options="{'icon': 'fa fa-bars'}"/>
                <field name="privacy" invisible="1"/>
                <field name="res_model_name" invisible="not res_model_name"
                       options="{'icon': 'fa fa-link', 'shouldOpenRecord': true}"/>
                <field name="alarm_ids" invisible="not alarm_ids" options="{'icon': 'fa fa-bell-o'}"/>
                <field name="categ_ids" invisible="not categ_ids" options="{'icon': 'fa fa-tag', 'color_field': 'color'}"/>
                <!-- For recurrence update Dialog -->
                <field name="recurrency" invisible="1"/>
                <field name="recurrence_update" invisible="1"/>
                <field name="partner_id" string="Organizer" options="{'icon': 'fa fa-user-o'}"/>
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
                <field name="user_id"/>
                <field name="location"/>
                <field name="show_as"/>
                <field name="categ_ids"/>
                <field name="description"/>
                <filter string="My Meetings" help="My Meetings" name="mymeetings" domain="[('partner_ids.user_ids', 'in', [uid])]"/>
                <separator/>
                <filter string="Date" name="filter_start_date" date="start"/>
                <separator/>
                <filter string="Busy" name="busy" domain="[('show_as', '=', 'busy')]"/>
                <filter string="Free" name="free" domain="[('show_as', '=', 'free')]"/>
                <separator/>
                <filter string="Public" name="public" domain="[('privacy', '=', 'public')]"/>
                <filter string="Private" name="private" domain="[('privacy', '=', 'private')]"/>
                <filter string="Only Internal Users" name="confidential" domain="[('privacy', '=', 'confidential')]"/>
                <separator/>
                <filter string="Recurrent" name="recurrent" domain="[('recurrency', '=', True)]"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                <group expand="0" string="Group By">
                    <filter string="Responsible" name="responsible" domain="[]" context="{'group_by': 'user_id'}"/>
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
            No meetings found. Let's schedule one!
          </p><p>
            The calendar is shared between employees and fully integrated with
            other applications such as the employee leaves or the business
            opportunities.
          </p>
        </field>
    </record>

    <record id="res_users_view_form" model="ir.ui.view">
            <field name="name">res.users.view.form.inherit.calendar</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="base.view_users_form"/>
            <field name="arch" type="xml">
                <notebook colspan="4" position="inside">
                    <!-- Placeholder container to hold information about external accounts (Google calendar, Microsoft calendar, ...) -->
                    <page string="Calendar" name="calendar" groups="base.group_system"
                         invisible="1">
                        <group name="calendar_accounts"/>
                    </page>
                </notebook>
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

    <record id="action_view_start_calendar_sync" model="ir.actions.act_window">
        <field name="name">Connect your Calendar</field>
        <field name="res_model">calendar.provider.config</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="context">{'dialog_size': 'medium'}</field>
    </record>

    <record id="calendar_settings_action" model="ir.actions.act_window">
        <field name="name">Settings</field>
        <field name="res_model">res.config.settings</field>
        <field name="view_mode">form</field>
        <field name="target">inline</field>
        <field name="context">{'module' : 'calendar', 'bin_size': False}</field>
    </record>

    <!-- Menus -->
    <menuitem
        id="mail_menu_calendar"
        name="Calendar"
        sequence="10"
        web_icon="calendar,static/description/icon.png"
        groups="base.group_user"/>

    <menuitem
        id="calendar_event_menu"
        name="Calendar"
        sequence="1"
        action="action_calendar_event"
        parent="mail_menu_calendar"
        groups="base.group_user"/>

    <menuitem
        id="calendar_menu_config"
        parent="calendar.mail_menu_calendar"
        name="Configuration"
        sequence="40"
        action="calendar.action_calendar_event"
        groups="base.group_system,base.group_no_one"/>

    <menuitem
        id="menu_calendar_settings"
        parent="calendar_menu_config"
        name="Settings"
        sequence="45"
        action="calendar_settings_action"
        groups="base.group_system"/>

    <menuitem
        id="calendar_submenu_reminders"
        parent="calendar_menu_config"
        name="Reminders"
        sequence="50"
        action="action_calendar_alarm"
        groups="base.group_no_one"/>

    <menuitem
        id="menu_calendar_configuration"
        name="Calendar"
        parent="base.menu_custom"
        sequence="30"
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
                  <attribute name="invisible">activity_category == 'meeting'</attribute>
            </xpath>
            <xpath expr="//field[@name='user_id']" position="attributes">
                  <attribute name="invisible">activity_category == 'meeting'</attribute>
            </xpath>
            <xpath expr="//button[@id='mail_activity_schedule']" position="attributes">
                  <attribute name="invisible">activity_category in ['meeting', 'phonecall'] or id</attribute>
            </xpath>
            <xpath expr="//button[@id='mail_activity_save']" position="attributes">
                  <attribute name="invisible">activity_category != 'phonecall' and not id</attribute>
            </xpath>
            <xpath expr="//button[@name='action_done']" position="attributes">
                  <attribute name="invisible">activity_category == 'meeting' or chaining_type == 'trigger'</attribute>
            </xpath>
            <xpath expr="//field[@name='note']" position="attributes">
                  <attribute name="invisible">activity_category == 'meeting'</attribute>
            </xpath>
            <xpath expr="//button[@name='action_close_dialog']" position="before">
                  <field name="calendar_event_id" invisible="1" />
                  <button string="Open Calendar"
                        close="1"
                        invisible="activity_category not in ['meeting', 'phonecall'] or calendar_event_id"
                        name="action_create_calendar_event"
                        type="object"
                        class="btn-primary"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.calendar</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base.res_config_settings_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//form" position="inside">
                <app data-string="Calendar" string="Calendar" name="calendar" groups="base.group_system">
                    <block title="Calendar Settings" name="calendar">
                        <setting string="Outlook Calendar" documentation="/applications/productivity/calendar/outlook.html" help="Synchronize your calendar with Outlook" id="sync_outlook_calendar_setting">
                            <field name="module_microsoft_calendar" />
                            <div class="content-group" invisible="not module_microsoft_calendar" id="msg_module_microsoft_calendar">
                                <div class="text-warning mt16"><strong>Save</strong> this page and come back here to set up the feature.</div>
                            </div>
                        </setting>
                        <setting string="Google Calendar" documentation="/applications/productivity/calendar/google.html" help="Synchronize your calendar with Google Calendar" id="sync_google_calendar_setting">
                            <field name="module_google_calendar"/>
                            <div class="content-group" invisible="not module_google_calendar" id="msg_module_google_calendar">
                                <div class="text-warning mt16"><strong>Save</strong> this page and come back here to set up the feature.</div>
                            </div>
                        </setting>
                    </block>
                </app>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0"?>
<odoo>

        <!-- Partner kanban view inherit -->
        <record id="res_partner_kanban_view" model="ir.ui.view">
            <field name="name">res.partner.view.kanban.calendar</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.res_partner_kanban_view"/>
            <field name="priority" eval="10"/>
            <field name="arch" type="xml">
                <field name="mobile" position="after">
                    <field name="meeting_count"/>
                </field>
                <xpath expr="//div[hasclass('oe_kanban_bottom_left')]" position="inside">
                    <a t-if="record.meeting_count.value>0" data-type="object" data-name="schedule_meeting"
                       href="#" class="oe_kanban_action oe_kanban_action_a me-1">
                        <span class="badge rounded-pill">
                            <i class="fa fa-fw fa-calendar" aria-label="Meetings" role="img" title="Meetings"/>
                            <t t-out="record.meeting_count.value"/>
                        </span>
                    </a>
                </xpath>
            </field>
        </record>

        <!-- Add contextual button on partner form view -->
        <record id="view_partners_form" model="ir.ui.view">
            <field name="name">res_partner.view.form.calendar</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field eval="1" name="priority"/>
            <field name="arch" type="xml">
                <data>
                    <div name="button_box" position="inside">
                        <button class="oe_stat_button" type="object"
                            name="schedule_meeting"
                            icon="fa-calendar"
                            context="{'partner_id': id, 'partner_name': name}">
                            <field string="Meetings" name="meeting_count" widget="statinfo"/>
                        </button>
                    </div>
                </data>
            </field>
        </record>

</odoo>

```

## File: wizard\calendar_popover_delete_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class CalendarProviderConfig(models.TransientModel):
    _name = 'calendar.popover.delete.wizard'
    _description = 'Calendar Popover Delete Wizard'


    record = fields.Many2one('calendar.event', 'Calendar Event')
    delete = fields.Selection([('one', 'Delete this event'), ('next', 'Delete this and following events'), ('all', 'Delete all the events')], default='one')

    def close(self):
        if not self.record or not self.delete:
            pass
        elif self.delete == 'one':
            self.record.unlink()
        else:
            switch = {
                'next': 'future_events',
                'all': 'all_events'
            }
            self.record.action_mass_deletion(switch.get(self.delete, ''))

```

## File: wizard\calendar_popover_delete_wizard.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <record id="calendar_popover_delete_view" model="ir.ui.view">
        <field name="name">calendar.popover.delete.wizard.view.form</field>
        <field name="model">calendar.popover.delete.wizard</field>
        <field name="arch" type="xml">
            <form string="Delete Event">
                <field name="delete" widget="radio"/>
                <footer>
                    <button name="close" string="Submit" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\calendar_provider_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.addons.base.models.ir_module import assert_log_admin_access
from odoo.tools import str2bool


class CalendarProviderConfig(models.TransientModel):
    _name = 'calendar.provider.config'
    _description = 'Calendar Provider Configuration Wizard'

    external_calendar_provider = fields.Selection([
        ('google', 'Google'), ('microsoft', 'Outlook')],
        "Choose an external calendar to configure", default='google')

    # Allow to sync with eventually existing ICP keys without creating them if respective module is not installed
    # Using same field names and strings as their respective res.config.settings
    cal_client_id = fields.Char(
        "Google Client_id",
        default=lambda self: self.env['ir.config_parameter'].get_param('google_calendar_client_id'))
    cal_client_secret = fields.Char(
        "Google Client_key",
        default=lambda self: self.env['ir.config_parameter'].get_param('google_calendar_client_secret'))
    cal_sync_paused = fields.Boolean(
        "Google Synchronization Paused",
        default=lambda self: str2bool(self.env['ir.config_parameter'].get_param('google_calendar_sync_paused'), default=False))
    microsoft_outlook_client_identifier = fields.Char(
        "Outlook Client Id",
        default=lambda self: self.env['ir.config_parameter'].get_param('microsoft_calendar_client_id'))
    microsoft_outlook_client_secret = fields.Char(
        "Outlook Client Secret",
        default=lambda self: self.env['ir.config_parameter'].get_param('microsoft_calendar_client_secret'))
    microsoft_outlook_sync_paused = fields.Boolean(
        "Outlook Synchronization Paused",
        default=lambda self: str2bool(self.env['ir.config_parameter'].get_param('microsoft_calendar_sync_paused'), default=False))

    @assert_log_admin_access
    def action_calendar_prepare_external_provider_sync(self):
        """ Called by the wizard to configure an external calendar provider without requiring users
        to access the general settings page.
        Make sure that the provider calendar module is installed or install it. Then, set
        the API keys into the applicable config parameters.
        """
        self.ensure_one()
        calendar_module = self.env['ir.module.module'].search([
            ('name', '=', f'{self.external_calendar_provider}_calendar')])

        if calendar_module.state != 'installed':
            calendar_module.button_immediate_install()

        if self.external_calendar_provider == 'google':
            self.env['ir.config_parameter'].set_param('google_calendar_client_id', self.cal_client_id)
            self.env['ir.config_parameter'].set_param('google_calendar_client_secret', self.cal_client_secret)
            self.env['ir.config_parameter'].set_param('google_calendar_sync_paused', self.cal_sync_paused)
        elif self.external_calendar_provider == 'microsoft':
            self.env['ir.config_parameter'].set_param('microsoft_calendar_client_id', self.microsoft_outlook_client_identifier)
            self.env['ir.config_parameter'].set_param('microsoft_calendar_client_secret', self.microsoft_outlook_client_secret)
            self.env['ir.config_parameter'].set_param('microsoft_calendar_sync_paused', self.microsoft_outlook_sync_paused)

```

## File: wizard\calendar_provider_config.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <record id="calendar_provider_config_view_form" model="ir.ui.view">
        <field name="name">calendar.provider.config.view.form</field>
        <field name="model">calendar.provider.config</field>
        <field name="arch" type="xml">
            <form>
                <field name="external_calendar_provider" widget="radio" options="{'horizontal': true}"/>
                <div invisible="external_calendar_provider != 'google'">
                    <img alt="Google Calendar icon" src="/calendar/static/src/img/google_calendar_40.png" style="height: 40px; margin-right: 5px"/>
                    <span class="me-1 o_form_label">Google Calendar</span>
                    <a href="https://www.odoo.com/documentation/17.0/applications/productivity/calendar/google.html" title="Read More" class="o_doc_link" target="_blank"></a>
                    <div class="text-muted mt-2">
                        Synchronize your calendar with Google Calendar
                    </div>
                    <group>
                        <field name="cal_client_id" string="Client ID" required="external_calendar_provider == 'google'"/>
                        <field name="cal_client_secret" string="Client Secret" password="True" required="external_calendar_provider == 'google'"/>
                        <field name="cal_sync_paused" required="external_calendar_provider == 'google'"/>
                    </group>
                </div>
                <div invisible="external_calendar_provider != 'microsoft'">
                    <img alt="Microsoft Outlook icon" src="/calendar/static/src/img/microsoft_calendar_40.png" style="height: 40px; margin-right: 5px"/>
                    <span class="me-1 o_form_label">Outlook Calendar</span>
                    <a href="https://www.odoo.com/documentation/17.0/applications/productivity/calendar/outlook.html" title="Read More" class="o_doc_link" target="_blank"></a>
                    <div class="text-muted mt-2">
                        Synchronize your calendar with Outlook
                    </div>
                    <group>
                        <field name="microsoft_outlook_client_identifier" string="Client ID" required="external_calendar_provider == 'microsoft'"/>
                        <field name="microsoft_outlook_client_secret" string="Client Secret" password="True" required="external_calendar_provider == 'microsoft'"/>
                        <field name="microsoft_outlook_sync_paused" required="external_calendar_provider == 'microsoft'"/>
                    </group>
                </div>
                <footer>
                    <widget name="calendar_connect_provider"/>
                    <button string="Cancel" class="btn btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\mail_activity_schedule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.exceptions import UserError
from odoo.tools.translate import _


class MailActivitySchedule(models.TransientModel):
    _inherit = 'mail.activity.schedule'

    def action_create_calendar_event(self):
        self.ensure_one()
        if self.is_batch_mode:
            raise UserError(_("Scheduling an activity using the calendar is not possible on more than one record."))
        return self.with_context({
            'default_res_model': self.res_model,
            'default_res_id': self._evaluate_res_ids()[0],
        })._action_schedule_activities().action_create_calendar_event()

```

## File: wizard\mail_activity_schedule_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
    <record id="mail_activity_schedule_view_form" model="ir.ui.view">
        <field name="name">mail.activity.schedule.inherit.calendar</field>
        <field name="model">mail.activity.schedule</field>
        <field name="inherit_id" ref="mail.mail_activity_schedule_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='date_deadline']" position="attributes">
                  <attribute name="invisible">activity_category == 'meeting'</attribute>
            </xpath>
            <xpath expr="//field[@name='activity_user_id']" position="attributes">
                  <attribute name="invisible">activity_category == 'meeting'</attribute>
            </xpath>
            <xpath expr="//button[@name='action_schedule_activities']" position="attributes">
                  <attribute name="invisible">has_error or activity_category == 'meeting' or id</attribute>
            </xpath>
            <xpath expr="//field[@name='note']" position="attributes">
                <attribute name="invisible">activity_category == 'meeting'</attribute>
            </xpath>
            <xpath expr="//button[@name='action_schedule_activities']" position="before">
                  <button string="Open Calendar"
                          invisible="activity_category not in ('meeting', 'phonecall')"
                          name="action_create_calendar_event"
                          type="object"
                          class="btn-primary"/>
            </xpath>
        </field>
    </record>
    </data>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from . import calendar_provider_config
from . import calendar_popover_delete_wizard
from . import mail_activity_schedule

```

