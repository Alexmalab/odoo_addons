# Odoo Module: calendar

Category: Productivity/Calendar

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Calendar',
    'version': '1.1',
    'sequence': 165,
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
        'data/mail_data_various.xml',
        'views/mail_activity_views.xml',
        'views/calendar_templates.xml',
        'views/calendar_views.xml',
        'views/res_partner_views.xml',
    ],
    'installable': True,
    'application': True,
    'auto_install': False,
    'assets': {
        'web.assets_backend': [
            'calendar/static/src/models/activity/activity.js',
            'calendar/static/src/components/activity/activity.js',
            'calendar/static/src/scss/calendar.scss',
            'calendar/static/src/js/base_calendar.js',
            'calendar/static/src/js/calendar_renderer.js',
            'calendar/static/src/js/calendar_controller.js',
            'calendar/static/src/js/calendar_model.js',
            'calendar/static/src/js/calendar_view.js',
            'calendar/static/src/js/systray_activity_menu.js',
            'calendar/static/src/js/services/calendar_notification_service.js',
        ],
        'web.qunit_suite_tests': [
            'calendar/static/tests/**/*',
        ],
        'web.assets_qweb': [
            'calendar/static/src/xml/base_calendar.xml',
            'calendar/static/src/components/activity/activity.xml',
        ],
        'web.assets_tests': [
            'calendar/static/tests/tours/calendar_tour.js'
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

## File: data\mail_data_various.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">
        <record id="mail.mail_activity_data_meeting" model="mail.activity.type">
            <field name="category">meeting</field>
        </record>

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
            <field name="body_html" type="html">
<div>
    <t t-set="colors" t-value="{'needsAction': 'grey', 'accepted': 'green', 'tentative': '#FFFF00', 'declined': 'red'}"/>
    <t t-set="is_online" t-value="'appointment_type_id' in object.event_id and object.event_id.appointment_type_id"/>
    <t t-set="customer" t-value=" object.event_id.find_partner_customer()"/>
    <t t-set="target_responsible" t-value="object.partner_id == object.event_id.partner_id"/>
    <t t-set="target_customer" t-value="object.partner_id == customer"/>
    <t t-set="recurrent" t-value="object.recurrence_id and not ctx.get('calendar_template_ignore_recurrence')"/>

    <p>
        Hello <t t-out="object.common_name or ''">Wood Corner</t>,<br/><br/>

        <t t-if="is_online and target_customer">
            Your appointment <strong t-out="object.event_id.appointment_type_id.name or ''">Schedule a Demo</strong> <t t-if="object.event_id.appointment_type_id.category != 'custom'"> with <t t-out="object.event_id.user_id.name or ''">Ready Mat</t></t> has been booked.
        </t>
        <t t-elif="is_online and target_responsible">
            <t t-if="customer">
                <t t-out="customer.name or ''"></t> scheduled the following appointment <strong t-out="object.event_id.appointment_type_id.name or ''">Schedule a Demo</strong> with you.
            </t>
            <t t-else="">
                Your appointment <strong t-out="object.event_id.appointment_type_id.name or ''">Schedule a Demo</strong> has been booked.
            </t>
        </t>
        <t t-elif="not target_responsible">
            <t t-out="object.event_id.user_id.partner_id.name or ''">Colleen Diaz</t> invited you for the <strong t-out="object.event_id.name or ''">Follow-up for Project proposal</strong> meeting.
        </t>
        <t t-else="">
            Your meeting <strong t-out="object.event_id.name or ''">Follow-up for Project proposal</strong> has been booked.
        </t>

    </p>
    <div style="text-align: center; padding: 16px 0px 16px 0px;">
        <t t-if="not is_online or object.state != 'accepted'">
            <a t-attf-href="/calendar/meeting/accept?token={{object.access_token}}&amp;id={{object.event_id.id}}"
                style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
                Accept</a>
            <a t-attf-href="/calendar/meeting/decline?token={{object.access_token}}&amp;id={{object.event_id.id}}"
                style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px">
                Decline</a>
        </t>
        <a t-attf-href="/calendar/meeting/view?token={{object.access_token}}&amp;id={{object.event_id.id}}"
            style="padding: 5px 10px; color: #FFFFFF; text-decoration: none; background-color: #875A7B; border: 1px solid #875A7B; border-radius: 3px"
            ><t t-out="'Reschedule' if is_online and target_customer else 'View'">View</t></a>
    </div>
    <table border="0" cellpadding="0" cellspacing="0"><tr>
        <td width="130px;">
            <div style="border-top-left-radius: 3px; border-top-right-radius: 3px; font-size: 12px; border-collapse: separate; text-align: center; font-weight: bold; color: #ffffff; min-height: 18px; background-color: #875A7B; border: 1px solid #875A7B;">
                <t t-out="format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format='EEEE', lang_code=object.env.lang) or ''">Tuesday</t>
            </div>
            <div style="font-size: 48px; min-height: auto; font-weight: bold; text-align: center; color: #5F5F5F; background-color: #F8F8F8; border: 1px solid #875A7B;">
                <t t-out="format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format='d', lang_code=object.env.lang) or ''">4</t>
            </div>
            <div style='font-size: 12px; text-align: center; font-weight: bold; color: #ffffff; background-color: #875A7B;'>
                <t t-out="format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format='MMMM y', lang_code=object.env.lang) or ''">May 2021</t>
            </div>
            <div style="border-collapse: separate; color: #5F5F5F; text-align: center; font-size: 12px; border-bottom-right-radius: 3px; font-weight: bold ; border: 1px solid #875A7B; border-bottom-left-radius: 3px;">
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
                <t t-if="is_online">
                    <li>Appointment Type: <t t-out="object.event_id.appointment_type_id.name or ''">Schedule a Demo</t></li>
                </t>
                <t t-if="object.event_id.location">
                    <li>Location: <t t-out="object.event_id.location or ''">Bruxelles</t>
                        (<a target="_blank" t-attf-href="http://maps.google.com/maps?oi=map&amp;q={{object.event_id.location}}">View Map</a>)
                    </li>
                </t>
                <t t-if="recurrent">
                    <li>When: <t t-out="object.recurrence_id.name or ''">Every 1 Weeks, for 3 events</t></li>
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
                    <li>Meeting URL: <a t-att-href="object.event_id.videocall_location" target="_blank" t-out="object.event_id.videocall_location or ''">https://meet.jit.si/odoo-xyz</a></li>
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
            <field name="name">Calendar: Date updated</field>
            <field name="model_id" ref="calendar.model_calendar_attendee"/>
            <field name="subject">{{ object.event_id.name }}: Date updated</field>
            <field name="email_from">{{ (object.event_id.user_id.email_formatted or user.email_formatted or '') }}</field>
            <field name="email_to">{{ ('' if object.partner_id.email and object.partner_id.email == object.email else object.email) }}</field>
            <field name="partner_to">{{ object.partner_id.id if object.partner_id.email and object.partner_id.email == object.email else False }}</field>
            <field name="lang">{{ object.partner_id.lang }}</field>
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
        <td width="130px;">
            <div style="border-top-left-radius: 3px; border-top-right-radius: 3px; font-size: 12px; border-collapse: separate; text-align: center; font-weight: bold; color: #ffffff; min-height: 18px; background-color: #875A7B; border: 1px solid #875A7B;">
                <t t-out='format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format="EEEE", lang_code=object.env.lang) or ""'>Tuesday</t>
            </div>
            <div style="font-size: 48px; min-height: auto; font-weight: bold; text-align: center; color: #5F5F5F; background-color: #F8F8F8; border: 1px solid #875A7B;">
                <t t-out="format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format='d', lang_code=object.env.lang) or ''">4</t>
            </div>
            <div style='font-size: 12px; text-align: center; font-weight: bold; color: #ffffff; background-color: #875A7B;'>
                <t t-out='format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format="MMMM y", lang_code=object.env.lang) or ""'>May 2021</t>
            </div>
            <div style="border-collapse: separate; color: #5F5F5F; text-align: center; font-size: 12px; border-bottom-right-radius: 3px; font-weight: bold; border: 1px solid #875A7B; border-bottom-left-radius: 3px;">
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
                    <li>When: <t t-out="object.recurrence_id.name or ''">Every 1 Weeks, for 3 events</t></li>
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
                    <li>Meeting URL: <a t-att-href="object.event_id.videocall_location" target="_blank" t-out="object.event_id.videocall_location or ''">https://meet.jit.si/odoo-xyz</a></li>
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
            <field name="body_html" type="html">
<div>
    <t t-set="colors" t-value="{'needsAction': 'grey', 'accepted': 'green', 'tentative': '#FFFF00', 'declined': 'red'}" />
    <t t-set="is_online" t-value="'appointment_type_id' in object.event_id and object.event_id.appointment_type_id" />
    <t t-set="recurrent" t-value="object.recurrence_id and not ctx.get('calendar_template_ignore_recurrence')" />
    <p>
        Hello <t t-out="object.common_name or ''">Gemini Furniture</t>,<br/><br/>
        This is a reminder for the below event :
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
        <td width="130px;">
            <div style="border-top-left-radius: 3px; border-top-right-radius: 3px; font-size: 12px; border-collapse: separate; text-align: center; font-weight: bold; color: #ffffff; min-height: 18px; background-color: #875A7B; border: 1px solid #875A7B;">
                <t t-out='format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format="EEEE", lang_code=object.env.lang) or ""'>Tuesday</t>
            </div>
            <div style="font-size: 48px; min-height: auto; font-weight: bold; text-align: center; color: #5F5F5F; background-color: #F8F8F8; border: 1px solid #875A7B;">
                <t t-out="format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format='d', lang_code=object.env.lang) or ''">4</t>
            </div>
            <div style='font-size: 12px; text-align: center; font-weight: bold; color: #ffffff; background-color: #875A7B;'>
                <t t-out='format_datetime(dt=object.event_id.start, tz=object.mail_tz if not object.event_id.allday else None, dt_format="MMMM y", lang_code=object.env.lang) or ""'>May 2021</t>
            </div>
            <div style="border-collapse: separate; color: #5F5F5F; text-align: center; font-size: 12px; border-bottom-right-radius: 3px; font-weight: bold; border: 1px solid #875A7B; border-bottom-left-radius: 3px;">
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
                    <li>When: <t t-out="object.recurrence_id.name or ''">Every 1 Weeks, for 3 events</t></li>
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
                    <li>Meeting URL: <a t-attf-href="{{ object.event_id.videocall_location }}" target="_blank" t-out="object.event_id.videocall_location or ''">https://meet.jit.si/odoo-xyz</a></li>
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
            <field name="body_html" type="html">
<div>
    <t t-set="colors" t-value="{'needsAction': 'grey', 'accepted': 'green', 'tentative': '#FFFF00', 'declined': 'red'}" />
    <t t-set="is_online = 'appointment_type_id' in object and object.appointment_type_id" />
    <t t-set="target_responsible = object.partner_id == object.partner_id" />
    <t t-set="target_customer = object.partner_id == customer" />
    <t t-set="recurrent = object.recurrence_id and not ctx.get('calendar_template_ignore_recurrence'" />
    <t t-set="mail_tz" t-value="object._get_mail_tz() or ctx.get('mail_tz')" />
    <div>
        <table border="0" cellpadding="0" cellspacing="0">
            <tr>
                <td width="130px;">
                    <div style="border-top-left-radius: 3px; border-top-right-radius: 3px; font-size: 12px; border-collapse: separate; text-align: center; font-weight: bold; color: #ffffff; min-height: 18px; background-color: #875A7B; border: 1px solid #875A7B;">
                        <t t-out="format_datetime(dt=object.start, tz=mail_tz if not object.allday else None, dt_format='EEEE', lang_code=object.env.lang) "></t>
                    </div>
                    <div style="font-size: 48px; min-height: auto; font-weight: bold; text-align: center; color: #5F5F5F; background-color: #F8F8F8; border: 1px solid #875A7B;">
                        <t t-out="format_datetime(dt=object.start, tz=mail_tz if not object.allday else None, dt_format='d', lang_code=object.env.lang)"></t>
                    </div>
                    <div style='font-size: 12px; text-align: center; font-weight: bold; color: #ffffff; background-color: #875A7B;'>
                        <t t-out="format_datetime(dt=object.start, tz=mail_tz if not object.allday else None, dt_format='MMMM y', lang_code=object.env.lang)"></t>
                    </div>
                    <div style="border-collapse: separate; color: #5F5F5F; text-align: center; font-size: 12px; border-bottom-right-radius: 3px; font-weight: bold; border: 1px solid #875A7B; border-bottom-left-radius: 3px;">
                        <t t-if="not object.allday">
                            <div>
                                <t t-out="format_time(time=object.start, tz=mail_tz, time_format='short', lang_code=object.env.lang)"></t>
                            </div>
                            <t t-if="mail_tz">
                                <div style="font-size: 10px; font-weight: normal">
                                    (<t t-out="mail_tz"></t>)
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
                            <li>Meeting URL:
                                <a t-att-href="object.videocall_location" target="_blank">
                                    <t t-out="object.videocall_location or ''"></t>
                                </a>
                            </li>
                        </t>
                        <t t-if="object.location">
                            <li>Location: <t t-out="object.location or ''"></t>
                                (<a target="_blank"
                                    t-attf-href="http://maps.google.com/maps?oi=map&amp;q={{object.location}}">View Map</a>)
                            </li>
                        </t>
                        <t t-if="recurrent">
                            <li>When: <t t-out="object.recurrence_id.name or ''"></t></li>
                        </t>
                        <t t-if="not object.allday and object.duration">
                            <li>Duration:
                                <t t-out="('%dH%02d' % (object.duration,round(object.duration*60)%60))"></t>
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
        search='_search_duration_minutes', compute='_compute_duration_minutes',
        help="Duration in minutes")
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
                            WHEN cal.recurrency THEN rrule.until
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

        self.flush()
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
               AND "event"."start" - CAST("alarm"."duration" || ' ' || "alarm"."interval" AS Interval) < now() at time zone 'utc'
             )''', [alarm_type, lastcall])

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
        attendees = events.attendee_ids.filtered(lambda a: a.state != 'declined')
        alarms = self.env['calendar.alarm'].browse(events_by_alarm.keys())
        for alarm in alarms:
            alarm_attendees = attendees.filtered(lambda attendee: attendee.event_id.id in events_by_alarm[alarm.id])
            alarm_attendees.with_context(
                mail_notify_force_send=True,
                calendar_template_ignore_recurrence=True,
                mail_notify_author=True
            )._send_mail_to_attendees(
                alarm.mail_template_id,
                force_send=True
            )

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
        users = self.env['res.users'].search([('partner_id', 'in', tuple(partner_ids))])
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
        ('tentative', 'Uncertain'),
        ('declined', 'Declined'),
        ('accepted', 'Accepted'),
    ]

    # event
    event_id = fields.Many2one('calendar.event', 'Meeting linked', required=True, ondelete='cascade')
    recurrence_id = fields.Many2one('calendar.recurrence', related='event_id.recurrence_id')
    # attendee
    partner_id = fields.Many2one('res.partner', 'Attendee', required=True, readonly=True)
    email = fields.Char('Email', related='partner_id.email', help="Email of Invited Person")
    phone = fields.Char('Phone', related='partner_id.phone', help="Phone number of Invited Person")
    common_name = fields.Char('Common name', compute='_compute_common_name', store=True)
    access_token = fields.Char('Invitation Token', default=_default_access_token)
    mail_tz = fields.Selection(_tz_get, compute='_compute_mail_tz', help='Timezone used for displaying time in the mail template')
    # state
    state = fields.Selection(STATE_SELECTION, string='Status', readonly=True, default='needsAction',
                             help="Status of the attendee's participation")
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
            events.message_subscribe(partner_ids=partners.ids)

    def _unsubscribe_partner(self):
        for event in self.event_id:
            partners = (event.attendee_ids & self).partner_id & event.message_partner_ids
            event.message_unsubscribe(partner_ids=partners.ids)

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

                attachment_values = []
                if ics_file:
                    attachment_values = [
                        (0, 0, {'name': 'invitation.ics',
                                'mimetype': 'text/calendar',
                                'res_id': event_id,
                                'res_model': 'calendar.event',
                                'datas': base64.b64encode(ics_file)})
                    ]
                body = mail_template._render_field(
                    'body_html',
                    attendee.ids,
                    compute_lang=True,
                    post_process=True)[attendee.id]
                subject = mail_template._render_field(
                    'subject',
                    attendee.ids,
                    compute_lang=True)[attendee.id]
                attendee.event_id.with_context(no_document=True).message_notify(
                    email_from=attendee.event_id.user_id.email_formatted or self.env.user.email_formatted,
                    author_id=attendee.event_id.user_id.partner_id.id or self.env.user.partner_id.id,
                    body=body,
                    subject=subject,
                    partner_ids=attendee.partner_id.ids,
                    email_layout_xmlid='mail.mail_notification_light',
                    attachment_ids=attachment_values,
                    force_send=force_send,
                    message_type='auto_comment',
                )

    def _should_notify_attendee(self):
        """ Utility method that determines if the attendee should be notified.
            By default, we do not want to notify (aka no message and no mail) the current user
            if he is part of the attendees.
            (Override in appointment to ignore that rule and notify all attendees if it's an appointment)
        """
        self.ensure_one()
        return self.partner_id != self.env.user.partner_id

    def do_tentative(self):
        """ Makes event invitation as Tentative. """
        return self.write({'state': 'tentative'})

    def do_accept(self):
        """ Marks event invitation as Accepted. """
        for attendee in self:
            attendee.event_id.message_post(
                body=_("%s has accepted invitation") % (attendee.common_name),
                subtype_xmlid="calendar.subtype_invitation")
        return self.write({'state': 'accepted'})

    def do_decline(self):
        """ Marks event invitation as Declined. """
        for attendee in self:
            attendee.event_id.message_post(
                body=_("%s has declined invitation") % (attendee.common_name),
                subtype_xmlid="calendar.subtype_invitation")
        return self.write({'state': 'declined'})

```

## File: models\calendar_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import math
import logging
from datetime import timedelta
from itertools import repeat

import pytz

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
from odoo.tools import pycompat, html2plaintext, is_html_empty
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

    # description
    name = fields.Char('Meeting Subject', required=True)
    description = fields.Html('Description')
    user_id = fields.Many2one('res.users', 'Organizer', default=lambda self: self.env.user)
    partner_id = fields.Many2one(
        'res.partner', string='Scheduled by', related='user_id.partner_id', readonly=True)
    location = fields.Char('Location', tracking=True, help="Location of Event")
    videocall_location = fields.Char('Meeting URL')
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
        'Start', required=True, tracking=True, default=fields.Date.today,
        help="Start date of an event, without time for full days events")
    stop = fields.Datetime(
        'Stop', required=True, tracking=True, default=lambda self: fields.Datetime.today() + timedelta(hours=1),
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
    # LUL TODO use fields.Reference ?
    res_id = fields.Integer('Document ID')
    res_model_id = fields.Many2one('ir.model', 'Document Model', ondelete='cascade')
    res_model = fields.Char(
        'Document Model Name', related='res_model_id.model', readonly=True, store=True)
    # messaging
    activity_ids = fields.One2many('mail.activity', 'calendar_event_id', string='Activities')
    # attendees
    attendee_ids = fields.One2many(
        'calendar.attendee', 'event_id', 'Participant')
    attendee_status = fields.Selection(
        Attendee.STATE_SELECTION, string='Attendee Status', compute='_compute_attendee')
    partner_ids = fields.Many2many(
        'res.partner', 'calendar_event_res_partner_rel',
        string='Attendees', default=_default_partners)
    # alarms
    alarm_ids = fields.Many2many(
        'calendar.alarm', 'calendar_alarm_calendar_event_rel',
        string='Reminders', ondelete="restrict",
        help="Notifications sent to all attendees to remind of the meeting.")
    # RECURRENCE FIELD
    recurrency = fields.Boolean('Recurrent')
    recurrence_id = fields.Many2one(
        'calendar.recurrence', string="Recurrence Rule", index=True)
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
    rrule_type = fields.Selection(RRULE_TYPE_SELECTION, string='Recurrence',
                                  help="Let the event automatically repeat at that interval",
                                  compute='_compute_recurrence', readonly=False)
    event_tz = fields.Selection(
        _tz_get, string='Timezone', compute='_compute_recurrence', readonly=False)
    end_type = fields.Selection(
        END_TYPE_SELECTION, string='Recurrence Termination',
        compute='_compute_recurrence', readonly=False)
    interval = fields.Integer(
        string='Repeat Every', compute='_compute_recurrence', readonly=False,
        help="Repeat every (Days/Week/Month/Year)")
    count = fields.Integer(
        string='Repeat', help="Repeat x times", compute='_compute_recurrence', readonly=False)
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
    byday = fields.Selection(BYDAY_SELECTION, compute='_compute_recurrence', readonly=False)
    until = fields.Date(compute='_compute_recurrence', readonly=False)
    # UI Fields.
    display_description = fields.Boolean(compute='_compute_display_description')

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
                enddate = fields.Datetime.from_string(meeting.stop_date)
                enddate = enddate.replace(hour=18)

                startdate = fields.Datetime.from_string(meeting.start_date)
                startdate = startdate.replace(hour=8)  # Set 8 AM

                meeting.write({
                    'start': startdate.replace(tzinfo=None),
                    'stop': enddate.replace(tzinfo=None)
                })

    def _compute_attendee(self):
        for meeting in self:
            attendee = meeting._find_attendee()
            meeting.attendee_status = attendee.state if attendee else 'needsAction'

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

    @api.depends('recurrence_id', 'recurrency')
    def _compute_recurrence(self):
        recurrence_fields = self._get_recurrent_fields()
        false_values = {field: False for field in recurrence_fields}  # computes need to set a value
        defaults = self.env['calendar.recurrence'].default_get(recurrence_fields)
        default_rrule_values = self.recurrence_id.default_get(recurrence_fields)
        for event in self:
            if event.recurrency:
                event.update(defaults)  # default recurrence values are needed to correctly compute the recurrence params
                event_values = event._get_recurrence_params()
                rrule_values = {
                    field: event.recurrence_id[field]
                    for field in recurrence_fields
                    if event.recurrence_id[field]
                }
                rrule_values = rrule_values or default_rrule_values
                event.update({**false_values, **event_values, **rrule_values})
            else:
                event.update(false_values)

    @api.depends('description')
    def _compute_display_description(self):
        for event in self:
            event.display_description = not is_html_empty(event.description)

    # ------------------------------------------------------------
    # CRUD
    # ------------------------------------------------------------

    @api.model_create_multi
    def create(self, vals_list):
        # Prevent sending update notification when _inverse_dates is called
        self = self.with_context(is_calendar_event_new=True)

        vals_list = [  # Else bug with quick_create when we are filter on an other user
            dict(vals, user_id=self.env.user.id) if not 'user_id' in vals else vals
            for vals in vals_list
        ]

        defaults = self.default_get(['activity_ids', 'res_model_id', 'res_id', 'user_id', 'res_model', 'partner_ids'])
        meeting_activity_type = self.env['mail.activity.type'].search([('category', '=', 'meeting')], limit=1)
        # get list of models ids and filter out None values directly
        model_ids = list(filter(None, {values.get('res_model_id', defaults.get('res_model_id')) for values in vals_list}))
        model_name = defaults.get('res_model')
        valid_activity_model_ids = model_name and self.env[model_name].sudo().browse(model_ids).filtered(lambda m: 'activity_ids' in m).ids or []
        if meeting_activity_type and not defaults.get('activity_ids'):
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

        events.filtered(lambda event: event.start > fields.Datetime.now()).attendee_ids._send_mail_to_attendees(
            self.env.ref('calendar.calendar_template_meeting_invitation', raise_if_not_found=False)
        )
        events._sync_activities(fields={f for vals in vals_list for f in vals.keys()})
        if not self.env.context.get('dont_notify'):
            events._setup_alarms()

        return events.with_context(is_calendar_event_new=False)

    def _compute_field_value(self, field):
        if field.compute_sudo:
            return super(Meeting, self.with_context(prefetch_fields=False))._compute_field_value(field)
        return super()._compute_field_value(field)

    def _read(self, fields):
        if self.env.is_system():
            super()._read(fields)
            return

        fields = set(fields)
        private_fields = fields - self._get_public_fields()
        if not private_fields:
            super()._read(fields)
            return

        private_fields.add('partner_ids')
        super()._read(fields | {'privacy', 'user_id', 'partner_ids'})
        current_partner_id = self.env.user.partner_id
        others_private_events = self.filtered(
            lambda e: e.privacy == 'private' \
                  and e.user_id != self.env.user \
                  and current_partner_id not in e.partner_ids
        )
        if not others_private_events:
            return

        for field_name in private_fields:
            field = self._fields[field_name]
            replacement = field.convert_to_cache(
                _('Busy') if field_name == 'name' else False,
                others_private_events)
            self.env.cache.update(others_private_events, field, repeat(replacement))

    def write(self, values):
        detached_events = self.env['calendar.event']
        recurrence_update_setting = values.pop('recurrence_update', None)
        update_recurrence = recurrence_update_setting in ('all_events', 'future_events') and len(self) == 1
        break_recurrence = values.get('recurrency') is False

        update_alarms = False
        update_time = False
        if 'partner_ids' in values:
            values['attendee_ids'] = self._attendees_values(values['partner_ids'])
            update_alarms = True

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
        if update_recurrence:
            if break_recurrence:
                # Update this event
                detached_events |= self._break_recurrence(future=recurrence_update_setting == 'future_events')
            else:
                future_update_start = self.start if recurrence_update_setting == 'future_events' else None
                time_values = {field: values.pop(field) for field in time_fields if field in values}
                if recurrence_update_setting == 'all_events':
                    # Update all events: we create a new reccurrence and dismiss the existing events
                    self._rewrite_recurrence(values, time_values, recurrence_values)
                else:
                    # Update future events
                    detached_events |= self._split_recurrence(time_values)
                    self.recurrence_id._write_events(values, dtstart=future_update_start)
        else:
            super().write(values)
            self._sync_activities(fields=values.keys())

        # We reapply recurrence for future events and when we add a rrule and 'recurrency' == True on the event
        if recurrence_update_setting not in ['self_only', 'all_events'] and not break_recurrence:
            detached_events |= self._apply_recurrence_values(recurrence_values, future=recurrence_update_setting == 'future_events')

        (detached_events & self).active = False
        (detached_events - self).with_context(archive_on_error=True).unlink()

        # Notify attendees if there is an alarm on the modified event, or if there was an alarm
        # that has just been removed, as it might have changed their next event notification
        if not self.env.context.get('dont_notify') and update_alarms:
            self._setup_alarms()
        attendee_update_events = self.filtered(lambda ev: ev.user_id != self.env.user)
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
        """ Checks if the event is private, returning True if the conditions match and False otherwise. """
        self.ensure_one()
        event_is_private = self.privacy == 'private'
        user_is_not_partner = self.user_id.id != self.env.uid and self.env.user.partner_id not in self.partner_ids
        return event_is_private and user_is_not_partner

    def name_get(self):
        """ Hide private events' name for events which don't belong to the current user
        """
        hidden = self.filtered(lambda evt: evt._check_private_event_conditions())
        shown = self - hidden
        shown_names = super(Meeting, shown).name_get()
        obfuscated_names = [(eid, _('Busy')) for eid in hidden.ids]
        return shown_names + obfuscated_names

    def read(self, fields):
        """
        Return the events information to be shown on calendar/tree/form views.
        Private events will have their sensitive fields hidden by default.
        """
        records = super().read(fields)
        if fields:
            # Define the private fields and filter the private events from self.
            # Since 'partner_ids' is extensively used for rendering, it can't be hidden.
            private_fields = set(fields) - self._get_public_fields()
            private_fields.discard('partner_ids')
            private_events = self.filtered(lambda ev: ev._check_private_event_conditions())

            # Hide the private information of the event by changing their values to 'Busy' and False.
            for event in records:
                if event['id'] in private_events.ids:
                    for field in private_fields:
                        if self._fields[field].type in ('one2many', 'many2many'):
                            event[field] = []
                        else:
                            event[field] = _('Busy') if field in ('name', 'display_name') else False

            # Update the cache with the new hidden values.
            for field_name in private_fields:
                field = self._fields[field_name]
                value = False
                if field.type in ('one2many', 'many2many'):
                    value = []
                elif field_name in ('name', 'display_name'):
                    value = _('Busy')
                for private_event in private_events:
                    replacement = field.convert_to_cache(value, private_event)
                    self.env.cache.update(private_event, field, repeat(replacement))
        return records

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        groupby = [groupby] if isinstance(groupby, str) else groupby
        grouped_fields = set(group_field.split(':')[0] for group_field in groupby)
        private_fields = grouped_fields - self._get_public_fields()
        if not self.env.su and private_fields:
            # display public and confidential events
            domain = AND([domain, ['|', ('privacy', '!=', 'private'), ('user_id', '=', self.env.user.id)]])
            self.env['bus.bus']._sendone(self.env.user.partner_id, 'simple_notification', {
                'title': _('Private Event Excluded'),
                'message': _('Grouping by %s is not allowed on private events.', ', '.join([self._fields[field_name].string for field_name in private_fields]))
            })
            return super(Meeting, self).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)
        return super(Meeting, self).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

    def unlink(self):
        # Get concerned attendees to notify them if there is an alarm on the unlinked events,
        # as it might have changed their next event notification
        events = self.filtered_domain([('alarm_ids', '!=', False)])
        partner_ids = events.mapped('partner_ids').ids

        # don't forget to update recurrences if there are some base events in the set to unlink,
        # but after having removed the events ;-)
        recurrences = self.env["calendar.recurrence"].search([
            ('base_event_id.id', 'in', [e.id for e in self])
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
            if op in (2, 3):  # Remove partner
                removed_partner_ids += [command[1]]
            elif op == 6:  # Replace all
                removed_partner_ids += set(self.partner_ids.ids) - set(command[2])  # Don't recreate attendee if partner already attend the event
                added_partner_ids += set(command[2]) - set(self.partner_ids.ids)
            elif op == 4:
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

    # ------------------------------------------------------------
    # ACTIONS
    # ------------------------------------------------------------

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
        composition_mode = self.env.context.get('composition_mode', 'comment')
        compose_ctx = dict(
            default_composition_mode=composition_mode,
            default_model='calendar.event',
            default_res_ids=self.ids,
            default_use_template=bool(template_id),
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
            self.recurrence_id.calendar_event_ids.write({'active': False})
        elif recurrence_update_setting == 'future_events' and self.recurrence_id:
            detached_events = self.recurrence_id._stop_at(self)
            detached_events.write({'active': False})

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

        for event in self:
            for alarm in (alarm for alarm in event.alarm_ids if alarm.alarm_type in alarm_types):
                at = event.start - timedelta(minutes=alarm.duration_minutes)
                if not cron.lastcall or at > cron.lastcall:
                    # Don't trigger for past alarms, they would be skipped by design
                    cron._trigger(at=at)
            if any(alarm.alarm_type == 'notification' for alarm in event.alarm_ids):
                # filter events before notifying attendees through calendar_alarm_manager
                events_to_notify |= event.filtered(lambda ev: ev.alarm_ids and ev.stop >= fields.Datetime.now())
        if events_to_notify:
            self.env['calendar.alarm_manager']._notify_next_alarm(events_to_notify.partner_ids.ids)

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

    def _rewrite_recurrence(self, values, time_values, recurrence_values):
        """ Recreate the whole recurrence when all recurrent events must be moved
        time_values corresponds to date times for one specific event. We need to update the base_event of the recurrence
        and reapply the recurrence later. All exceptions are lost.
        """
        self.ensure_one()
        base_event = self.recurrence_id.base_event_id
        if not base_event:
            raise UserError(_("You can't update a recurrence without base event."))
        [base_time_values] = self.recurrence_id.base_event_id.read(['start', 'stop', 'allday'])
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

        time_values.update(update_dict)
        if time_values or recurrence_values:
            rec_fields = list(self._get_recurrent_fields())
            [rec_vals] = base_event.read(rec_fields)
            old_recurrence_values = {field: rec_vals.pop(field) for field in rec_fields if
                                     field in rec_vals}
            base_event.write({**values, **time_values})
            # Delete all events except the base event and the currently modified
            expandable_events = self.recurrence_id.calendar_event_ids - (self.recurrence_id.base_event_id + self)
            self.recurrence_id.with_context(archive_on_error=True).unlink()
            expandable_events.with_context(archive_on_error=True).unlink()
            # Make sure to recreate a new recurrence. Needed to prevent sync issues
            base_event.recurrence_id = False
            # Recreate all events and the recurrence: override updated values
            new_values = {
                **old_recurrence_values,
                **base_event._get_recurrence_params(),
                **recurrence_values,
            }
            new_values.pop('rrule')
            detached_events = base_event._apply_recurrence_values(new_values)
            detached_events.write({'active': False})
            # archive the current event if all the events were recreated
            if self != self.recurrence_id.base_event_id and time_values:
                self.active = False
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
            all_events = recurrence_update_setting == 'all_events'
            return attendee.with_context(all_events=all_events).do_accept()
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

    def _find_attendee(self):
        """ Return the first attendee where the user connected has been invited
            or the attendee selected in the filter that is the owner
            from all the meeting_ids in parameters.
        """
        self.ensure_one()

        my_attendee = self.attendee_ids.filtered(lambda att: att.partner_id == self.env.user.partner_id)
        if my_attendee:
            return my_attendee[:1]

        event_checked_attendees = self.env['calendar.filters'].search([
            ('user_id', '=', self.env.user.id),
            ('partner_id', 'in', self.attendee_ids.partner_id.ids),
            ('partner_checked', '=', True)
        ]).mapped('partner_id')
        if self.partner_id in event_checked_attendees and self.partner_id in self.attendee_ids.partner_id:
            return self.attendee_ids.filtered(lambda attendee: attendee.partner_id == self.partner_id)[:1]

        attendee = self.attendee_ids.filtered(lambda att: att.partner_id in event_checked_attendees and att.state != "needsAction")
        return attendee[:1]

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
            event.add('summary').value = meeting.name
            if not is_html_empty(meeting.description):
                if 'appointment_type_id' in meeting._fields and self.appointment_type_id:
                    # convert_online_event_desc_to_text method for correct data formatting in external calendars
                    event.add('description').value = self.convert_online_event_desc_to_text(meeting.description)
                else:
                    event.add('description').value = html2plaintext(meeting.description)
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
            event.add('organizer').value = u'MAILTO:' + (meeting.user_id.email or u'')
            result[meeting.id] = cal.serialize().encode('utf-8')

        return result

    def convert_online_event_desc_to_text(self, description):
        """
        We can sync the calendar events with google calendar, iCal and Outlook, and we
        also pass the event description along with other data. This description needs
        to be in plaintext to be displayed properly in above platforms. Because online
        events have fixed format for the description, this method removes some specific
        html tags, and converts it into readable plaintext (to be used in external
        calendars). Note that for regular (offline) events, we simply use the standard
        `html2plaintext` method instead.
        """
        desc_str = str(description)
        tags_to_replace = ["<ul>", "</ul>", "<li>"]
        for tag in tags_to_replace:
            desc_str = desc_str.replace(tag, "")
        desc_str = desc_str.replace("</li>", "<br/>")
        return html2plaintext(desc_str)

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
        ('name_uniq', 'unique (name)', "Tag name already exists !"),
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

    user_id = fields.Many2one('res.users', 'Me', required=True, default=lambda self: self.env.user)
    partner_id = fields.Many2one('res.partner', 'Employee', required=True)
    active = fields.Boolean('Active', default=True)
    partner_checked = fields.Boolean('Checked', default=True,
        help="This field is used to know if the partner is checked in the filter of the calendar view for the user_id.")

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

from dateutil import rrule
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.exceptions import UserError

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

    _sql_constraints = [
        ('month_day',
         "CHECK (rrule_type != 'monthly' "
                "OR month_by != 'day' "
                "OR day >= 1 AND day <= 31 "
                "OR weekday in %s AND byday in %s)"
                % (tuple(wd[0] for wd in WEEKDAY_SELECTION), tuple(bd[0] for bd in BYDAY_SELECTION)),
         "The day must be between 1 and 31"),
    ]

    @api.depends('rrule')
    def _compute_name(self):
        for recurrence in self:
            period = dict(RRULE_TYPE_SELECTION)[recurrence.rrule_type]
            every = _("Every %(count)s %(period)s", count=recurrence.interval, period=period)

            if recurrence.end_type == 'count':
                end = _("for %s events", recurrence.count)
            elif recurrence.end_type == 'end_date':
                end = _("until %s", recurrence.until)
            else:
                end = ''

            if recurrence.rrule_type == 'weekly':
                weekdays = recurrence._get_week_days()
                # Convert Weekday object
                weekdays = [str(w) for w in weekdays]
                # We need to get the day full name from its three first letters.
                week_map = {v: k for k, v in RRULE_WEEKDAYS.items()}
                weekday_short = [week_map[w] for w in weekdays]
                day_strings = [d[1] for d in WEEKDAY_SELECTION if d[0] in weekday_short]
                on = _("on %s") % ", ".join([day_name for day_name in day_strings])
            elif recurrence.rrule_type == 'monthly':
                if recurrence.month_by == 'day':
                    weekday_label = dict(BYDAY_SELECTION)[recurrence.byday]
                    on = _("on the %(position)s %(weekday)s", position=recurrence.byday, weekday=weekday_label)
                else:
                    on = _("day %s", recurrence.day)
            else:
                on = ''
            recurrence.name = ' '.join(filter(lambda s: s, [every, on, end]))

    @api.depends('calendar_event_ids.start')
    def _compute_dtstart(self):
        groups = self.env['calendar.event'].read_group([('recurrence_id', 'in', self.ids)], ['start:min'], ['recurrence_id'])
        start_mapping = {
            group['recurrence_id'][0]: group['start']
            for group in groups
        }
        for recurrence in self:
            recurrence.dtstart = start_mapping.get(recurrence.id)

    @api.depends(
        'byday', 'until', 'rrule_type', 'month_by', 'interval', 'count', 'end_type',
        'mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun', 'day', 'weekday')
    def _compute_rrule(self):
        for recurrence in self:
            recurrence.rrule = recurrence._rrule_serialize()

    def _inverse_rrule(self):
        for recurrence in self:
            if recurrence.rrule:
                values = self._rrule_parse(recurrence.rrule, recurrence.dtstart)
                recurrence.write(values)

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
        self.env['calendar.event'].with_context(no_mail_to_attendees=True, mail_create_nolog=True).create(event_vals)
        return detached_events

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
        events.write({
            'recurrence_id': False,
            'recurrency': False,
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
        token = request.params.get('token', '')

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

from odoo import api, models, fields, tools, _
from odoo.tools import is_html_empty

class MailActivityType(models.Model):
    _inherit = "mail.activity.type"

    category = fields.Selection(selection_add=[('meeting', 'Meeting')])


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
        }
        return action

    def _action_done(self, feedback=False, attachment_ids=False):
        # To avoid the feedback to be included in the activity note (due to the synchronization in event.write
        # that updates the related activity note each time the event description is updated),
        # when the activity is written as a note in the chatter in _action_done (leading to duplicate feedback),
        # we call super before updating the description. As self is deleted in super, we load the related events before.
        events = self.mapped('calendar_event_id')
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
            all_partners = self.with_context(active_test=False).search_read([('id', 'child_of', self.ids)], ["parent_id"])
            all_partners_parents = {p["id"]: p['parent_id'][0] for p in all_partners if p.get('parent_id')}

            event_id = self.env['calendar.event']._search([])  # ir.rules will be applied
            subquery_string, subquery_params = event_id.select()
            subquery = self.env.cr.mogrify(subquery_string, subquery_params).decode()

            self.env.cr.execute("""
                SELECT res_partner_id, calendar_event_id, count(1)
                  FROM calendar_event_res_partner_rel
                 WHERE res_partner_id IN %s AND calendar_event_id IN ({})
              GROUP BY res_partner_id, calendar_event_id
            """.format(subquery), [tuple(p["id"] for p in all_partners)])

            meeting_data = self.env.cr.fetchall()

            # Create a dict {partner_id: event_ids} and fill with events linked to the partner
            meetings = {}
            for p_id, m_id, _ in meeting_data:
                meetings.setdefault(p_id, set()).add(m_id)

            # Add the events linked to the children of the partner
            for meeting_pid in set(meetings):
                partner_id = meeting_pid
                while partner_id in all_partners_parents:
                    partner_id = all_partners_parents[partner_id]
                    if partner_id in self.ids:
                        meetings[partner_id] = meetings.get(partner_id, set()) | meetings[meeting_pid]
            return {p_id: list(meetings.get(p_id, set())) for p_id in self.ids}
        return {}

    def get_attendee_detail(self, meeting_ids):
        """ Return a list of dict of the given meetings with the attendees details
            Used by:
                - base_calendar.js : Many2ManyAttendee
                - calendar_model.js (calendar.CalendarModel)
        """
        attendees_details = []
        meetings = self.env['calendar.event'].browse(meeting_ids)
        meetings_attendees = meetings.mapped('attendee_ids')
        for partner in self:
            partner_info = partner.name_get()[0]
            for attendee in meetings_attendees.filtered(lambda att: att.partner_id == partner):
                attendee_is_organizer = self.env.user == attendee.event_id.user_id and attendee.partner_id == self.env.user.partner_id
                attendees_details.append({
                    'id': partner_info[0],
                    'name': partner_info[1],
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
# -*- coding: utf-8 -*-
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
        now_utc = datetime.datetime.utcnow()
        start_dt_utc = start_dt = now_utc.replace(tzinfo=UTC)
        stop_dt_utc = datetime.datetime.combine(now_utc.date(), datetime.time.max).replace(tzinfo=UTC)

        tz = self.env.user.tz
        if tz:
            user_tz = timezone(tz)
            start_dt = start_dt_utc.astimezone(user_tz)
            stop_dt = datetime.datetime.combine(start_dt.date(), datetime.time.max).replace(tzinfo=user_tz)
            stop_dt_utc = stop_dt.astimezone(UTC)

        start_date = start_dt.date()

        return ['&', '|',
                '&',
                    '|',
                        ['start', '>=', fields.Datetime.to_string(start_dt_utc)],
                        ['stop', '>=', fields.Datetime.to_string(start_dt_utc)],
                    ['start', '<=', fields.Datetime.to_string(stop_dt_utc)],
                '&',
                    ['allday', '=', True],
                    ['start_date', '=', fields.Date.to_string(start_date)],
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

## File: static\src\components\activity\activity.js

```javascript
/** @odoo-module */

import { Activity } from '@mail/components/activity/activity';
import { patch } from 'web.utils';
import Dialog from 'web.Dialog';
import core from 'web.core';
const _t = core._t;

patch(Activity.prototype, 'calendar/static/src/components/activity/activity.js', {
    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Small override that asks for confirmation in case there is a meeting linked to this activity.
     *
     * @override
     */
    _onClickCancel(ev) {
        const superMethod = this._super;
        if (!this.activity.calendar_event_id){
            this._super(ev);
        } else {
            Dialog.confirm(
                this,
                _t("The activity is linked to a meeting. Deleting it will remove the meeting as well. Do you want to proceed ?"), {
                    confirm_callback: function () {
                        superMethod(ev);
                    },
                }
            );
        }
    }

});

```

## File: static\src\components\activity\activity.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-inherit="mail.Activity" t-inherit-mode="extension">
        <xpath expr="//button[hasclass('o_Activity_editButton')]" position="attributes">
            <attribute name="t-if">!activity.calendar_event_id</attribute>
        </xpath>
        <xpath expr="//button[hasclass('o_Activity_editButton')]" position="after">
            <t t-if="activity.calendar_event_id">
                <button class="o_Activity_toolButton o_Activity_editButton btn btn-link" t-on-click="_onClickEdit">
                    <i class="fa fa-calendar"/> Reschedule
                </button>
            </t>
        </xpath>
    </t>

</templates>

```

## File: static\src\js\base_calendar.js

```javascript
/** @odoo-module **/

import BasicModel from 'web.BasicModel';
import fieldRegistry from 'web.field_registry';
import relationalFields from 'web.relational_fields';

const FieldMany2ManyTagsAvatar = relationalFields.FieldMany2ManyTagsAvatar;

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
            args: [attendeeIDs, [meetingID]],
            context: context,
        }).then(function (result) {
            return result;
        });
    },
});

const Many2ManyAttendee = FieldMany2ManyTagsAvatar.extend({
    // as this widget is model dependant (rpc on res.partner), use it in another
    // context probably won't work
    // supportedFieldTypes: ['many2many'],
    specialData: "_fetchSpecialAttendeeStatus",
    className: 'o_field_many2manytags avatar',

    init: function () {
        this._super.apply(this, arguments);
        this.className += this.nodeOptions.block ? ' d-block' : '';
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     * @private
     */
    _renderTags: function () {
        this._super.apply(this, arguments);
        const avatars = this.el.querySelectorAll('.o_m2m_avatar');
        for (const avatar of avatars) {
            const partner_id = parseInt(avatar.dataset["id"]);
            const partner_data = this.record.specialData.partner_ids.find(partner => partner.id === partner_id);
            if (partner_data) {
                avatar.classList.add('o_attendee_border', "o_attendee_border_" + partner_data.status);
            }
        }
    },
    /**
     * @override
     * @private
     */
    _getRenderTagsContext: function () {
        let result = this._super.apply(this, arguments);
        result.attendeesData = this.record.specialData.partner_ids;
        // Sort attendees to have the organizer on top.
        // partner_ids are sorted by default according to their id/display_name in the "elements" FieldMany2ManyTag
        // This method sort them to put the organizer on top
        const organizer = result.attendeesData.find(item => item.is_organizer);
        if (organizer) {
            const org_id = organizer.id
            // sort elements according to the partner id
            result.elements.sort((a, b) => {
                const a_org = a.id === org_id;
                return a_org ? -1 : 1;
             });
        }
        return result;
    },
});

fieldRegistry.add('many2manyattendee', Many2ManyAttendee);

```

## File: static\src\js\calendar_controller.js

```javascript
/** @odoo-module alias=calendar.CalendarController **/

    import Controller from 'web.CalendarController';
    import Dialog from 'web.Dialog';
    import { qweb as QWeb, _t } from 'web.core';

    const CalendarController = Controller.extend({

    renderButtons: function ($node) {
        this._super.apply(this, arguments);
        const $addButton =  $(QWeb.render('Calendar.calendar_add_buttons'));
        this.$buttons.prepend($addButton)
        const self = this;
        // When clicking on "Add", create a new record in form view
        this.$buttons.on('click', 'button.o-calendar-button-new', () => {
            // TODO: switch to ir.action.act_window in master
            return self.do_action('calendar.action_calendar_event_notify', {
                additional_context: self.context,
            });
        });
    },

        _askRecurrenceUpdatePolicy() {
            return new Promise((resolve, reject) => {
                new Dialog(this, {
                    title: _t('Edit Recurrent event'),
                    size: 'small',
                    $content: $(QWeb.render('calendar.RecurrentEventUpdate')),
                    buttons: [{
                        text: _t('Confirm'),
                        classes: 'btn-primary',
                        close: true,
                        click: function () {
                            resolve(this.$('input:checked').val());
                        },
                    }],
                }).open();
            });
        },

        /**
         * If the event comes from the organizer we delete the event, else we the event is decline for the attendee
         * @override 
         */
        _onDeleteRecord: function (ev) {
            const event = _.find(this.model.data.data, e => e.id === ev.data.id && e.attendee_id === ev.data.event.attendee_id);
            if (this.getSession().partner_id === event.attendee_id && this.getSession().partner_id === event.record.partner_id[0]) {
                this._super(...arguments);
            } else {
                var self = this;
                this.model.declineEvent(event).then(function () {
                    self.reload();
                });
            }
        },

        /**
         * @override
         * @private
         * @param {OdooEvent} event
         */
        async _onDropRecord(event) {
            const _super = this._super; // reference to this._super is lost after async call
            await this._dropdUpdateRecord(event);
            _super.apply(this, arguments);
        },

        /**
         * @override
         * @private
         * @param {OdooEvent} event
         */
        async _onUpdateRecord(event) {
            const _super = this._super;  // reference to this._super is lost after async call
            await this._dropdUpdateRecord(event);
            _super.apply(this, arguments);
        },

        async _dropdUpdateRecord(event) {
            if (event.data.record.recurrency) {
                const recurrenceUpdate = await this._askRecurrenceUpdatePolicy();
                event.data = _.extend({}, event.data, {
                    'recurrenceUpdate': recurrenceUpdate,
                });
            }
        }

    });

    export default CalendarController;

```

## File: static\src\js\calendar_model.js

```javascript
/** @odoo-module alias=calendar.CalendarModel **/

    import Model from 'web.CalendarModel';

    const CalendarModel = Model.extend({

        /**
         * @override
         * Transform fullcalendar event object to odoo Data object
         */
        calendarEventToRecord(event) {
            const data = this._super(event);
            return _.extend({}, data, {
                'recurrence_update': event.recurrenceUpdate,
            });
        },
        async _getCalendarEventData(events) {
            const calendarEventData = await this._super(...arguments);
            const calendarEventByAttendeeData = await this._calendarEventByAttendee(calendarEventData);
            return calendarEventByAttendeeData;
        },
        /**
         * Split the events to display an event for each attendee with the correct status.
         * If the all filter is activated, we don't display an event for each attendee and keep
         * the previous behavior to display a single event.
         * 
         */
        async _calendarEventByAttendee(eventsData) {
            const self = this;
            let eventsDataByAttendee = [];
            const attendeeFilters = self.loadParams.filters.partner_ids;
            const everyoneFilter = attendeeFilters && (attendeeFilters.filters.find(f => f.value === "all") || {}).active || false;
            const attendeeIDs = attendeeFilters && _.filter(attendeeFilters.filters.map(partner => partner.value !== 'all' ? partner.value : false), id => id !== false);
            const eventIDs = eventsData.map(event => event.id);
            // Fetch the attendees' info from the partners selected in the filter to display the events
            this.attendees = await self._rpc({
                model: 'res.partner',
                method: 'get_attendee_detail',
                args: [attendeeIDs, eventIDs],
            });
            if (!everyoneFilter) {
                const currentPartnerId = this.getSession().partner_id;
                eventsData.forEach(event => {
                    const attendees = event.record.partner_ids && event.record.partner_ids.length ? event.record.partner_ids : [event.record.partner_id[0]];
                    // Get the list of partner_id corresponding to active filters present in the current event
                    const attendees_filtered = attendeeFilters.filters.reduce((acc, filter) => {
                        if (filter.active && attendees.includes(filter.value)) {
                            acc.push(filter.value);
                        }
                        return acc;
                    }, []);

                    // Create Event data for each attendee found
                    attendees_filtered.forEach(attendee => {
                        let e = $.extend(true, {}, event);
                        e.attendee_id = attendee;
                        const attendee_info = self.attendees.find(a => a.id === attendee && a.event_id === e.record.id);
                        if (attendee_info) {
                            e.record.attendee_status = attendee_info.status;
                            e.record.is_alone = attendee_info.is_alone;
                            // check if this event data corresponds to the current partner
                            e.record.is_current_partner = currentPartnerId === attendee_info.id;
                        }
                        eventsDataByAttendee.push(e);
                    });
                });
            } else {
                eventsData.forEach(event => {
                    event.attendee_id = event.record.partner_id && event.record.partner_id[0];
                    const attendee_info = self.attendees.find(a => a.id === self.getSession().partner_id && a.event_id === event.record.id);
                    if (attendee_info) {
                        event.record.is_alone = attendee_info.is_alone;
                    }
                });
            }
            return eventsDataByAttendee.length ? eventsDataByAttendee : eventsData;
        },

        /**
         * Decline an event for the actual attendee
         * @param {Integer} eventId
         */
        declineEvent: function (event) {
            return this._rpc({
                model: 'calendar.attendee',
                method: 'do_decline',
                args: [this.attendees.find(attendee => attendee.event_id === event.id && attendee.id === this.getSession().partner_id).attendee_id],
            });
        },

    /**
     * Set the event color according to the filters values.
     * When Everybodies'events are displayed, the color are set according to the first attendee_id to decrease confusion.
     * Else, the event color are defined according to the existing filters colors.
     * @private
     * @param {any} element
     * @param {any} events
     * @returns {Promise}
     */
    _loadColors: function (element, events) {
        if (this.fieldColor) {
            const fieldName = this.fieldColor;
            for (const event of events) {
                // list of partners in case of calendar event
                const value = event.record[fieldName];
                const colorRecord = value[0];
                const filter = this.loadParams.filters[fieldName];
                const colorFilter = filter && filter.filters.map(f => f.value) || [colorRecord];
                const everyoneFilter = filter && (filter.filters.find(f => f.value === "all") || {}).active || false;
                let colorValue;
                if (!everyoneFilter) {
                    colorValue = event.attendee_id;
                } else {
                    const partner_id = this.getSession().partner_id
                    colorValue = value.includes(partner_id) ? partner_id : colorRecord;
                }
                event.color_index = this._getColorIndex(colorFilter, colorValue);
            }
            this.model_color = this.fields[fieldName].relation || element.model;

        }
        return Promise.resolve();
    },

    });

    export default CalendarModel;

```

## File: static\src\js\calendar_renderer.js

```javascript
/** @odoo-module alias=calendar.CalendarRenderer **/

import CalendarRenderer from 'web.CalendarRenderer';
import CalendarPopover from 'web.CalendarPopover';
import session from 'web.session';

const AttendeeCalendarPopover = CalendarPopover.extend({
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
        // Show status dropdown if user is in attendees list
        if (this.isCurrentPartnerAttendee()) {
            this.statusColors = {accepted: 'text-success', declined: 'text-danger', tentative: 'text-muted', needsAction: 'text-dark'};
            this.statusInfo = {};
            _.each(this.fields.attendee_status.selection, function (selection) {
                self.statusInfo[selection[0]] = {text: selection[1], color: self.statusColors[selection[0]]};
            });
            this.selectedStatusInfo = this.statusInfo[this.event.extendedProps.record.attendee_status];
        }
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @return {boolean}
     */
    isCurrentPartnerOrganizer() {
        return this.event.extendedProps.record.partner_id[0] === session.partner_id;
    },
    /**
     * @return {boolean}
     */
    isCurrentPartnerAttendee() {
        return this.event.extendedProps.record.partner_ids.includes(session.partner_id);
    },
    /**
     * @override
     * @return {boolean}
     */
    isEventDeletable() {
        return this._super() && this.isCurrentPartnerAttendee();
    },
    /**
     * @override
     * @return {boolean}
     */
    isEventDetailsVisible() {
        return this._isEventPrivate() ? this.isCurrentPartnerAttendee() : this._super();
    },
    /**
     * @override
     * @return {boolean}
     */
    isEventEditable() {
        return this._isEventPrivate() ? this.isCurrentPartnerAttendee() : this._super();
    },
    /**
     * Check if we are a partner and if we are the only attendee.
     * This avoid to display attendee answer dropdown for single user attendees
     * @return {boolean}
     */
    displayAttendeeAnswerChoice() {
        const isCurrentpartner = (currentValue) => currentValue === session.partner_id;
        const onlyAttendee =  this.event.extendedProps.record.partner_ids.every(isCurrentpartner);
        return this.isCurrentPartnerAttendee() && this.event.extendedProps.record.is_current_partner && !onlyAttendee;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @return {boolean}
     */
    _isEventPrivate() {
        return this.event.extendedProps.record.privacy === 'private';
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
        const selectedStatus = $(ev.currentTarget).attr('data-action');
        this.trigger_up('AttendeeStatus', {id: parseInt(this.event.id), record: this.event.extendedProps.record,
        selectedStatus: selectedStatus});
    },
});


const AttendeeCalendarRenderer = CalendarRenderer.extend({
    template: 'calendar.CalendarView',

	config: _.extend({}, CalendarRenderer.prototype.config, {
        CalendarPopover: AttendeeCalendarPopover,
        eventTemplate: 'Calendar.calendar-box',
    }),
    /**
     * Add the attendee-id attribute in order to distinct the events when there are
     * several attendees in the event.
     * @override
     */
    _addEventAttributes: function (element, event) {
        this._super(...arguments);
        element.attr('data-attendee-id', event.extendedProps.attendee_id);
    },
    /**
     * If an attendee_id has been set on the event, we check also the attendee-id attribute
     * to select the good event on which the CSS class will be applied.
     * @override
     */
    _computeEventSelector: function (info) {
        let selector = this._super(...arguments);
        if (info.event.extendedProps.attendee_id) {
            selector += `[data-attendee-id=${info.event.extendedProps.attendee_id}]`;
        }
        return selector;
    },
});

export default {
    AttendeeCalendarRenderer: AttendeeCalendarRenderer,
    AttendeeCalendarPopover: AttendeeCalendarPopover,
};

```

## File: static\src\js\calendar_view.js

```javascript
/** @odoo-module alias=calendar.CalendarView **/

import CalendarController from '@calendar/js/calendar_controller';
import CalendarModel from '@calendar/js/calendar_model';
import AttendeeCalendarRenderer from '@calendar/js/calendar_renderer';
import CalendarView from 'web.CalendarView';
import viewRegistry from 'web.view_registry';

const CalendarRenderer = AttendeeCalendarRenderer.AttendeeCalendarRenderer;

var AttendeeCalendarView = CalendarView.extend({
    config: _.extend({}, CalendarView.prototype.config, {
        Renderer: CalendarRenderer,
        Controller: CalendarController,
        Model: CalendarModel,
    }),
});

viewRegistry.add('attendee_calendar', AttendeeCalendarView);

export default AttendeeCalendarView;

```

## File: static\src\js\systray_activity_menu.js

```javascript
/** @odoo-module **/

import ActivityMenu from '@mail/js/systray/systray_activity_menu';
import fieldUtils from 'web.field_utils';

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

```

## File: static\src\js\services\calendar_notification_service.js

```javascript
/** @odoo-module **/

import { browser } from "@web/core/browser/browser";
import { ConnectionLostError } from "@web/core/network/rpc_service";
import { registry } from "@web/core/registry";

export const calendarNotificationService = {
    dependencies: ["action", "notification", "rpc"],

    start(env, { action, notification, rpc }) {
        let calendarNotifTimeouts = {};
        let nextCalendarNotifTimeout = null;
        const displayedNotifications = new Set();

        env.bus.on("WEB_CLIENT_READY", null, async () => {
            const legacyEnv = owl.Component.env;
            legacyEnv.services.bus_service.onNotification(this, (notifications) => {
                for (const { payload, type } of notifications) {
                    if (type === "calendar.alarm") {
                        displayCalendarNotification(payload);
                    }
                }
            });
            legacyEnv.services.bus_service.startPolling();
        });

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
                                name: env._t("OK"),
                                primary: true,
                                onClick: async () => {
                                    await rpc("/calendar/notify_ack");
                                    notificationRemove();
                                },
                            },
                            {
                                name: env._t("Details"),
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
                                name: env._t("Snooze"),
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

## File: static\src\models\activity\activity.js

```javascript
/** @odoo-module **/

import {
    registerClassPatchModel,
    registerInstancePatchModel,
    registerFieldPatchModel
} from'@mail/model/model_core';
import { attr } from '@mail/model/model_field';

registerFieldPatchModel('mail.activity', 'calendar/static/src/models/activity/activity.js', {
    calendar_event_id: attr({ default: false }),
});

registerClassPatchModel('mail.activity', 'calendar/static/src/models/activity/activity.js', {
    /**
     * @override
     */
    convertData(data) {
        const res = this._super(data);
        if ('calendar_event_id' in data) {
            res.calendar_event_id = data.calendar_event_id[0];
        }
        return res;
    },
});

registerInstancePatchModel('mail.activity', 'calendar/static/src/models/activity/activity.js', {
    /**
     * @override
     */
    async deleteServerRecord() {
        if (!this.calendar_event_id){
            await this._super();
        } else {
            await this.async(() => this.env.services.rpc({
                model: 'mail.activity',
                method: 'unlink_w_meeting',
                args: [[this.id]],
            }));
            this.delete();
        }
    },

    /**
     * In case the activity is linked to a meeting, we want to open the calendar view instead.
     *
     * @override
     */
    async edit() {
        if (!this.calendar_event_id){
            await this._super();
        } else {
            const action = await this.async(() => this.env.services.rpc({
                model: 'mail.activity',
                method: 'action_create_calendar_event',
                args: [[this.id]],
            }));
            this.env.bus.trigger('do-action', {
                action
            });
        }
    }
});

```

## File: static\src\xml\base_calendar.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<template>

    <t t-name="calendar.CalendarView" t-extend="CalendarView">
        <t t-jquery="div.o_calendar_sidebar" t-operation="append">
            <div id="calendar_sync" class="o_calendar_sync container inline btn-group"/>
        </t>
    </t>

    <div t-name="calendar.RecurrentEventUpdate">
        <div class="form-check o_radio_item">
            <input name="recurrence-update" type="radio" class="form-check-input o_radio_input" checked="true" value="self_only" id="self_only"/>
            <label class="form-check-label o_form_label" for="self_only">This event</label>
        </div>

        <div class="form-check o_radio_item">
            <input name="recurrence-update" type="radio" class="form-check-input o_radio_input" value="future_events" id="future_events"/>
            <label class="form-check-label o_form_label" for="future_events">This and following events</label>
        </div>

        <div class="form-check o_radio_item">
            <input name="recurrence-update" type="radio" class="form-check-input o_radio_input" value="all_events" id="all_events"/>
            <label class="form-check-label o_form_label" for="all_events">All events</label>
        </div>

    </div>

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
                                <t t-esc="moment(meeting.start).local().format(Time.strftime_to_moment_format(_t.database.parameters.time_format))"/>
                            </t>
                        </span>
                    </div>
                </t>
            </div>
        </t>
    </t>

    <t t-name="Calendar.attendee.status.popover" t-extend="CalendarView.event.popover">
        <t t-jquery=".o_cw_popover_delete" t-operation="after">
            <div t-if="widget.displayAttendeeAnswerChoice()" class="btn-group o-calendar-attendee-status ml-2">
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

     <t t-name="Calendar.calendar_add_buttons">
        <span class="o_calendar_create_buttons">
            <button class="btn btn-primary o-calendar-button-new" aria-label="Add" title="Add">Add</button>
        </span>
    </t>

    <t t-name="Calendar.calendar-box">
        <div t-attf-class="#{record.is_highlighted ? 'o_event_hightlight' : ''} #{typeof color === 'number' ? _.str.sprintf('o_calendar_color_%s', color) : 'o_calendar_color_1'} fc-event o_event o_attendee_status_#{record.is_alone ? 'alone' : record.attendee_status} py-0" >
            <div class="o_event_title mr-2">
                <span t-if="record.is_alone" class="fa fa-exclamation-circle"/>
                <t t-esc="record.display_name"/>
            </div>
            <span class="fc-time"/>
        </div>
    </t>

</template>

```

## File: views\calendar_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Template rendered in route auth=None, for anonymous user. This allow them to see meeting details -->
    <template id="invitation_page_anonymous" name="Calendar Invitation Page for anonymous users">
        <t t-call="web.layout">
            <t t-set="head">
                <t t-call-assets="web.assets_common" t-js="false"/>
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
                            <span class="float-right badge badge-info">
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
                <sheet>
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
                        <group attrs="{'invisible': [('alarm_type','=','notification')]}">
                            <field name="mail_template_id" attrs="{'invisible': [('alarm_type','!=','email')], 'required': [('alarm_type', '=', 'email')]}"
                                context="{'default_model': 'calendar.event'}"/>
                        </group>
                    </group>
                    <group attrs="{'invisible': [('alarm_type','!=','notification')]}">
                        <field name="body"/>
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
                    <button name="action_open_composer" type="object" context="{'composition_mode':'mass_mail'}"
                            string="Send Mail"/>
                </header>
                <field name="name" string="Subject" decoration-bf="1" attrs="{'readonly':[('recurrency','=',True)]}"/>
                <field name="start" string="Start Date" readonly="1"/>
                <field name="stop" string="End Date" readonly="1"/>
                <field name="user_id" widget="many2one_avatar_user" attrs="{'readonly':[('recurrency','=',True)]}" optional="hide"/>
                <field name="partner_ids" widget="many2many_tags" attrs="{'readonly':[('recurrency','=',True)]}" optional="show"/>
                <field name="alarm_ids" widget="many2many_tags" optional="hide" attrs="{'readonly':[('recurrency','=',True)]}"/>
                <field name="categ_ids" widget="many2many_tags" optional="hide" attrs="{'readonly':[('recurrency','=',True)]}"/>
                <field name="recurrency" optional="hide" readonly="1"/>
                <field name="privacy" optional="hide" attrs="{'readonly':[('recurrency','=',True)]}"/>
                <field name="show_as" optional="hide" attrs="{'readonly':[('recurrency','=',True)]}"/>
                <field name="location" optional="show" attrs="{'readonly':[('recurrency','=',True)]}"/>
                <field name="duration" widget="float_time" readonly="1"/>
                <field name="description" optional="hide" attrs="{'readonly':[('recurrency','=',True)]}"/>
                <field name="allday" invisible="1"/>
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
                <div attrs="{'invisible': [('recurrence_id','=',False)]}" class="alert alert-info oe_edit_only" role="status">
                    <p>Edit recurring event</p>
                    <field name="recurrence_update" widget="radio"/>
                </div>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button string="Document" icon="fa-bars" type="object" name="action_open_calendar_event" attrs="{'invisible': ['|', ('res_model', '=', False), ('res_id', '=', False)]}"/>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <field name="res_model" invisible="1" />
                    <field name="res_id" invisible="1" />
                    <field name="attendee_status" invisible="1"/>
                    <field name="active" invisible="1"/>
                    <div class="oe_title mb-3">
                        <div>
                            <label for="name"/>
                        </div>
                        <h1>
                            <field name="name" placeholder="e.g. Business Lunch"/>
                        </h1>
                    </div>
                    <div class="d-flex align-items-baseline">
                        <field name="partner_ids" widget="many2manyattendee"
                            placeholder="Select attendees..."
                            context="{'force_email':True}"
                            domain="[('type','!=','private')]"
                            class="oe_inline o_calendar_attendees"
                        />
                        <div name="send_buttons" class="sm-2">
                            <button name="action_open_composer" help="Send Email to attendees" type="object" string=" EMAIL" icon="fa-envelope"/>
                        </div>
                    </div>
                    <notebook>
                        <page name="page_details" string="Meeting Details">
                            <group>
                                <group>

                                    <field name="start_date" string="Starting at" attrs="{'required': [('allday','=',True)], 'invisible': [('allday','=',False)]}" force_save="1"/>
                                    <field name="stop_date" string="Ending at" attrs="{'required': [('allday','=',True)],'invisible': [('allday','=',False)]}" force_save="1"/>

                                    <field name="start" string="Starting at" attrs="{'required': [('allday','=',False)], 'invisible': [('allday','=',True)]}"/>
                                    <field name="stop" string="Ending At" attrs="{'invisible': [('allday','=',True)]}"/>
                                    <label for="duration" attrs="{'invisible': [('allday','=',True)]}"/>
                                    <div attrs="{'invisible': [('allday','=',True)]}">
                                        <field name="duration" widget="float_time" string="Duration" class="oe_inline" attrs="{'readonly': [('id', '!=', False), ('recurrency','=',True)]}"/>
                                        <span> hours</span>
                                    </div>
                                     <field name="event_tz" attrs="{'invisible': [('recurrency', '=', False)]}"/>
                                    <field name="allday" force_save="1"/>
                                    <field name="user_id" widget="many2one_avatar_user"/>
                                </group>
                                <group>
                                    <field name="alarm_ids" widget="many2many_tags" options="{'no_quick_create': True}"/>
                                    <field name="location" />
                                    <field name="videocall_location"/>
                                    <field name="categ_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                                </group>
                            </group>
                            <group>
                                <field name="description"/>
                            </group>
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
                                            <div class="o_col">
                                                <div class="o_row">
                                                    <field name="interval" attrs="{'required': [('recurrency', '=', True)]}"/>
                                                    <field name="rrule_type" attrs="{'required': [('recurrency', '=', True)]}"/>
                                                </div>
                                                <widget name="week_days" attrs="{'invisible': [('rrule_type', '!=', 'weekly')]}"/>
                                            </div>
                                            <label string="Until" for="end_type"/>
                                            <div class="o_row">
                                                <field name="end_type" attrs="{'required': [('recurrency', '=', True)]}"/>
                                                <field name="count" attrs="{'invisible': [('end_type', '!=', 'count')], 'required': [('recurrency', '=', True)]}"/>
                                                <field name="until" attrs="{'invisible': [('end_type', '!=', 'end_date')], 'required': [('end_type', '=', 'end_date'), ('recurrency', '=', True)]}"/>
                                            </div>
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
                                                <field name="weekday" nolabel="1"
                                                    attrs="{'required': [('recurrency', '=', True), ('month_by', '=', 'day'), ('rrule_type', '=', 'monthly')],
                                                            'invisible': [('month_by', '!=', 'day')]}"/>
                                            </div>
                                        </group>
                                    </div>
                                </div>
                                <group>
                                    <field name="privacy"/>
                                    <field name="show_as"/>
                                    <field name="recurrence_id" invisible="1" />
                                </group>
                            </group>
                        </page>

                        <page name="page_invitations" string="Invitations" groups="base.group_no_one">
                            <button name="action_sendmail" type="object" string="Send Invitations" icon="fa-envelope" class="oe_link"/>
                            <field name="attendee_ids" widget="one2many" mode="tree,kanban" readonly="1">
                                <tree string="Invitation details" editable="top" create="false" delete="false">
                                    <field name="partner_id" />
                                    <field name="email" widget="email"/>
                                    <field name="phone" widget="phone"/>
                                    <field name="state" />

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
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="message_ids" />
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
                color="partner_ids">
                <field name="attendee_status" invisible="1"/>
                <field name="partner_ids" options="{'block': True, 'icon': 'fa fa-users'}"
                       filters="1" widget="many2manyattendee" write_model="calendar.filters"
                       write_field="partner_id" filter_field="partner_checked" avatar_field="avatar_128"
                />
                <field name="is_highlighted" invisible="1"/>
                <field name="is_organizer_alone" invisible="1"/>
                <field name="display_description" invisible="1"/>
                <field name="location" attrs="{'invisible': [('location', '=', False)]}"/>
                <field name="description" attrs="{'invisible': [('display_description', '=', False)]}"/>
                <field name="privacy"/>
                <field name="alarm_ids" attrs="{'invisible': [('alarm_ids', '=', [])]}"/>
                <field name="categ_ids" attrs="{'invisible': [('categ_ids', '=', [])]}"/>
                <!-- For recurrence update Dialog -->
                <field name="recurrency" invisible="1"/>
                <field name="recurrence_update" invisible="1"/>
                <field name="partner_id" string="Organizer"/>
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
                    <filter string="Availability" name="availability" domain="[]" context="{'group_by': 'show_as'}"/>
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
                    <page string="Calendar" name="calendar" invisible="1" groups="base.group_system">
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

    <!-- Menus -->
    <menuitem
        id="mail_menu_calendar"
        name="Calendar"
        sequence="10"
        action="action_calendar_event"
        web_icon="calendar,static/description/icon.png"
        groups="base.group_user"/>

    <menuitem
        id="calendar_menu_config"
        parent="calendar.mail_menu_calendar"
        name="Configuration"
        sequence="40"
        action="calendar.action_calendar_event"
        groups="base.group_no_one"/>

    <menuitem
        id="calendar_submenu_reminders"
        parent="calendar_menu_config"
        name="Reminders"
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

    <!-- called in js from '/js/base_calendar.js' -->
    <!-- TODO: remove in master -->
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
            <xpath expr="//button[@id='mail_activity_schedule']" position="attributes">
                  <attribute name="attrs">{'invisible': ['|', ('activity_category', 'in', ['meeting', 'phonecall']), ('id', '!=', False)]}</attribute>
            </xpath>
            <xpath expr="//button[@id='mail_activity_save']" position="attributes">
                  <attribute name="attrs">{'invisible': [('activity_category', '!=', 'phonecall'), ('id', '=', False)]}</attribute>
            </xpath>
            <xpath expr="//button[@name='action_done']" position="attributes">
                  <attribute name="attrs">{'invisible': ['|', ('activity_category', '=', 'meeting'), ('chaining_type', '=', 'trigger')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='note']" position="attributes">
                  <attribute name="attrs">{'invisible': [('activity_category', '=', 'meeting')]}</attribute>
            </xpath>
            <xpath expr="//button[@name='action_close_dialog']" position="before">
                  <field name="calendar_event_id" invisible="1" />
                  <button string="Open Calendar"
                        attrs="{'invisible': ['|', ('activity_category', 'not in', ['meeting', 'phonecall']), ('calendar_event_id', '!=', False)]}"
                        name="action_create_calendar_event"
                        type="object"
                        class="btn-primary"/>
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
                <xpath expr="//span[hasclass('oe_kanban_partner_links')]" position="inside">
                    <span class="badge badge-pill" t-if="record.meeting_count.value>0">
                        <i class="fa fa-fw fa-calendar" aria-label="Meetings" role="img" title="Meetings"/>
                        <t t-esc="record.meeting_count.value"/>
                    </span>
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
                            context="{'partner_id': active_id, 'partner_name': name}">
                            <field string="Meetings" name="meeting_count" widget="statinfo"/>
                        </button>
                    </div>
                </data>
            </field>
        </record>

</odoo>

```

