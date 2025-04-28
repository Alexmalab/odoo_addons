# Odoo Module: event

Category: Marketing/Events

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': 'Events Organization',
    'version': '1.3',
    'website': 'https://www.odoo.com/page/events',
    'category': 'Marketing/Events',
    'summary': 'Trainings, Conferences, Meetings, Exhibitions, Registrations',
    'description': """
Organization and management of Events.
======================================

The event module allows you to efficiently organize events and all related tasks: planning, registration tracking,
attendances, etc.

Key Features
------------
* Manage your Events and Registrations
* Use emails to automatically confirm and send acknowledgments for any event registration
""",
    'depends': ['base_setup', 'mail', 'portal', 'utm'],
    'data': [
        'security/event_security.xml',
        'security/ir.model.access.csv',
        'views/event_menu_views.xml',
        'views/event_ticket_views.xml',
        'views/event_views.xml',
        'views/event_stage_views.xml',
        'report/event_event_templates.xml',
        'report/event_event_reports.xml',
        'data/email_template_data.xml',
        'data/event_data.xml',
        'views/res_config_settings_views.xml',
        'views/event_templates.xml',
        'views/res_partner_views.xml',
        'views/event_tag_views.xml'
    ],
    'demo': [
        'data/res_users_demo.xml',
        'data/res_partner_demo.xml',
        'data/event_demo_misc.xml',
        'data/event_demo.xml',
        'data/event_registration_demo.xml',
    ],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.exceptions import NotFound

from odoo.http import Controller, request, route, content_disposition


class EventController(Controller):

    @route(['''/event/<model("event.event"):event>/ics'''], type='http', auth="public")
    def event_ics_file(self, event, **kwargs):
        if request.env.user._is_public():
            frontend_lang = request.httprequest.cookies.get('frontend_lang')
            if frontend_lang:
                event = event.with_context(lang=frontend_lang)
        files = event._get_ics_file()
        if not event.id in files:
            return NotFound()
        content = files[event.id]
        return request.make_response(content, [
            ('Content-Type', 'application/octet-stream'),
            ('Content-Length', len(content)),
            ('Content-Disposition', content_disposition('%s.ics' % event.name))
        ])

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\email_template_data.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">

        <record id="event_registration_mail_template_badge" model="mail.template">
            <field name="name">Event: Registration Badge</field>
            <field name="model_id" ref="event.model_event_registration"/>
            <field name="subject">Your badge for ${object.event_id.name}</field>
            <field name="email_from">${(object.event_id.organizer_id.email_formatted or object.event_id.user_id.email_formatted or '') | safe}</field>
            <field name="email_to">${(object.email and '"%s" &lt;%s&gt;' % (object.name, object.email) or object.partner_id.email_formatted or '') | safe}</field>
            <field name="body_html" type="html">
<div>
    Dear ${object.name},<br/>
    Thank you for your inquiry.<br/>
    Here is your badge for the event ${object.event_id.name}.<br/>
    If you have any questions, please let us know.
    <br/><br/>
    Thank you,
    % if object.event_id.user_id.signature:
        <br />
        ${object.event_id.user_id.signature | safe}
    % endif
</div></field>
            <field name="report_template" ref="report_event_registration_badge"/>
            <field name="report_name">badge_of_${(object.event_id.name or '').replace('/','_')}</field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <record id="event_subscription" model="mail.template">
            <field name="name">Event: Registration</field>
            <field name="model_id" ref="event.model_event_registration"/>
            <field name="subject">Your registration at ${object.event_id.name}</field>
            <field name="email_from">${(object.event_id.organizer_id.email_formatted or object.event_id.user_id.email_formatted or '') | safe}</field>
            <field name="email_to">${(object.email and '"%s" &lt;%s&gt;' % (object.name, object.email) or object.partner_id.email_formatted or '') | safe}</field>
            <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
% set date_begin = format_datetime(object.event_id.date_begin, tz='UTC', dt_format="yyyyMMdd'T'HHmmss'Z'")
% set date_end = format_datetime(object.event_id.date_end, tz='UTC', dt_format="yyyyMMdd'T'HHmmss'Z'")
% set is_online = 'is_published' in object.event_id and object.event_id.is_published
% set event_organizer = object.event_id.organizer_id
% set event_address = object.event_id.address_id
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 16px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table width="590" border="0" cellpadding="0" cellspacing="0" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Your registration</span><br/>
                    <span style="font-size: 20px; font-weight: bold;">
                        ${object.name}
                    </span>
                </td><td valign="middle" align="right">
                    % if is_online
                    <a href="${object.event_id.website_url}"
                        style="padding: 8px 12px; font-size: 12px; color: #FFFFFF; text-decoration: none !important; font-weight: 400; background-color: #875A7B; border: 0px solid #875A7B; border-radius:3px">
                        View Event
                    </a>
                    % else
                    <img src="${'/logo.png?company=%s' % object.company_id.id}" style="padding: 0px; margin: 0px; height: auto; width: 80px;" alt="${'%s' % object.company_id.name}"/>
                    % endif
                </td></tr>
                <tr><td colspan="2" style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin:16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- EVENT DESCRIPTION -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table width="590" border="0" cellpadding="0" cellspacing="0" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="top" style="font-size: 14px;">
                    <div>
                        Hello ${object.name or ''},<br/>
                        We are happy to confirm your registration to the event
                        % if is_online:
                            <a href="${object.event_id.website_url}" style="color:#875A7B;text-decoration:none;">${object.event_id.name}</a>
                        % else:
                            <strong>${object.event_id.name}</strong>
                        % endif
                        for attendee ${object.name}.
                    </div>
                    <div>
                        <br />
                        <strong>Add this event to your calendar</strong>
                        <a href="https://www.google.com/calendar/render?action=TEMPLATE&amp;text=${object.event_id.name}&amp;dates=${date_begin}/${date_end}&amp;location=${location}" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;" target="new"><img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> Google</a>
                        <a href="/event/${slug(object.event_id)}/ics" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;"><img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> iCal/Outlook</a>
                        <a href="https://calendar.yahoo.com/?v=60&amp;view=d&amp;type=20&amp;title=${object.event_id.name}&amp;in_loc=${location}&amp;st=${format_datetime(object.event_id.date_begin, tz='UTC', dt_format='yyyyMMdd\'T\'HHmmss')}&amp;et=${format_datetime(object.event_id.date_end, tz='UTC', dt_format='yyyyMMdd\'T\'HHmmss')}" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;" target="new">
                            <img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> Yahoo
                        </a>
                        <br /><br />
                    </div>
                    <div>
                        See you soon,<br/>
                        <span style="color: #454748;">
                        -- <br/>
                        % if event_organizer:
                            ${event_organizer.name}
                        % else:
                            The ${object.event_id.name} Team
                        % endif
                        </span>
                    </div>
                </td></tr>
                <tr><td style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- DETAILS -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table width="590" border="0" cellpadding="0" cellspacing="0" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="top" style="font-size: 14px;">
                    <table style="width:100%;">
                        <tr>
                            <td style="vertical-align:top;">
                                <img src="/web_editor/font_to_img/61555/rgb(81,81,102)/34" style="padding:4px;max-width:inherit;" height="34" alt=""/>
                            </td>
                            <td style="padding: 0px 10px 0px 10px;width:50%;line-height:20px;vertical-align:top;">
                                <div><strong>From</strong> ${object.event_id.date_begin_located}</div>
                                <div><strong>To</strong> ${object.event_id.date_end_located}</div>
                                <div style="font-size:12px;color:#9e9e9e"><i><strong>TZ</strong> ${object.event_id.date_tz}</i></div>
                            </td>
                            <td style="vertical-align:top;">
                                % if event_address:
                                <img src="/web_editor/font_to_img/61505/rgb(81,81,102)/34" style="padding:4px;max-width:inherit;" height="34" alt=""/>
                                % endif
                            </td>
                            <td style="padding: 0px 10px 0px 10px;width:50%;vertical-align:top;">
                                % if event_address:
                                % set location = ''
                                % if object.event_id.address_id.name:
                                    <div>${object.event_id.address_id.name}</div>
                                % endif
                                % if object.event_id.address_id.street:
                                    <div>${object.event_id.address_id.street}</div>
                                    % set location = object.event_id.address_id.street
                                % endif
                                % if object.event_id.address_id.street2:
                                    <div>${object.event_id.address_id.street2}</div>
                                    % set location = '%s, %s' % (location, object.event_id.address_id.street2)
                                % endif
                                <div>
                                % if object.event_id.address_id.city:
                                    ${object.event_id.address_id.city},
                                    % set location = '%s, %s' % (location, object.event_id.address_id.city)
                                % endif
                                % if object.event_id.address_id.state_id.name:
                                    ${object.event_id.address_id.state_id.name},
                                    % set location = '%s, %s' % (location, object.event_id.address_id.state_id.name)
                                % endif
                                % if object.event_id.address_id.zip:
                                    ${object.event_id.address_id.zip}
                                    % set location = '%s, %s' % (location, object.event_id.address_id.zip)
                                % endif
                                </div>
                                % if object.event_id.address_id.country_id.name:
                                    <div>${object.event_id.address_id.country_id.name}</div>
                                    % set location = '%s, %s' % (location, object.event_id.address_id.country_id.name)
                                % endif
                                % endif
                            </td>
                        </tr>
                    </table>
                </td></tr>
                <tr><td style="text-align:center;">
                    % if event_organizer
                    <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                    % endif
                </td></tr>

                <tr><td valign="top" style="font-size: 14px;">
                    <!-- CONTACT ORGANIZER -->
                    % if event_organizer:
                    <div>
                        <span style="font-weight:300;margin:10px 0px">Questions about this event?</span>
                        <div>Please contact the organizer:</div>
                        <ul>
                            <li>${event_organizer.name}</li>
                            % if event_organizer.email
                                <li>Mail: <a href="mailto:${event_organizer.email}" style="text-decoration:none;color:#875A7B;">${event_organizer.email}</a></li>
                            % endif
                            % if event_organizer.phone
                                <li>Phone: ${event_organizer.phone}</li>
                            % endif
                        </ul>
                    </div>
                    % endif
                </td></tr>
                <tr><td style="text-align:center;">
                    <!-- CONTACT ORGANIZER SEPARATION -->
                    % if is_online or event_address:
                    <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                    % endif
                </td></tr>

                <tr><td valign="top" style="font-size: 14px;">
                    <!-- PWA MARKGETING -->
                    % if is_online:
                    <div>
                        <strong>Get the best mobile experience.</strong>
                        <a href="/event">Install our mobile app</a>
                    </div>
                    % endif
                </td></tr>
                <tr><td style="text-align:center;">
                    <!-- PWA MARKGETING SEPARATION-->
                    % if is_online and event_address:
                    <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                    % endif
                </td></tr>

                <tr><td valign="top" style="font-size: 14px;">
                    <!-- GOOGLE MAPS LINK -->
                    % if event_address:
                    <table style="width:100%;"><tr><td>
                        <div>
                            <a href="https://maps.google.com/maps?q=${location}" target="new">
                                <img src="http://maps.googleapis.com/maps/api/staticmap?autoscale=1&amp;size=598x200&amp;maptype=roadmap&amp;format=png&amp;visual_refresh=true&amp;markers=size:mid%7Ccolor:0xa5117d%7Clabel:%7C${location}" style="vertical-align:bottom; width: 100%;" alt="Google Maps"/>
                            </a>
                        </div>
                    </td></tr></table>
                    % endif
                </td></tr>
            </table>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- FOOTER BY -->
<tr><td align="center" style="min-width: 590px;">
    % if object.company_id
    <table width="590" border="0" cellpadding="0" cellspacing="0" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
      <tr><td style="text-align: center; font-size: 14px;">
        Sent by <a target="_blank" href="${object.company_id.website}" style="color: #875A7B;">${object.company_id.name}</a>
        % if is_online:
        <br />
        Discover <a href="/event" style="text-decoration:none;color:#717188;">all our events</a>.
        % endif
      </td></tr>
    </table>
    % endif
</td></tr>
</table>
            </field>
            <field name="lang">${object.partner_id.lang}</field>
        </record>

        <record id="event_reminder" model="mail.template">
            <field name="name">Event: Reminder</field>
            <field name="model_id" ref="event.model_event_registration"/>
            <field name="subject">${object.event_id.name}: ${object.get_date_range_str()}</field>
            <field name="email_from">${(object.event_id.organizer_id.email_formatted or object.event_id.user_id.email_formatted or '') | safe}</field>
            <field name="email_to">${(object.email and '"%s" &lt;%s&gt;' % (object.name, object.email) or object.partner_id.email_formatted or '') | safe}</field>
            <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
% set date_begin = format_datetime(object.event_id.date_begin, tz='UTC', dt_format="yyyyMMdd'T'HHmmss'Z'")
% set date_end = format_datetime(object.event_id.date_end, tz='UTC', dt_format="yyyyMMdd'T'HHmmss'Z'")
% set is_online = 'is_published' in object.event_id and object.event_id.is_published
% set event_organizer = object.event_id.organizer_id
% set event_address = object.event_id.address_id
<table border="0" cellpadding="0" cellspacing="0"  width="590" style="padding: 16px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table width="590" border="0" cellpadding="0" cellspacing="0" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Your registration</span><br/>
                    <span style="font-size: 20px; font-weight: bold;">
                        ${object.name}
                    </span>
                </td><td valign="middle" align="right">
                    % if is_online
                    <a href="${object.event_id.website_url}"
                        style="padding: 8px 12px; font-size: 12px; color: #FFFFFF; text-decoration: none !important; font-weight: 400; background-color: #875A7B; border: 0px solid #875A7B; border-radius:3px">
                        View Event
                    </a>
                    % else
                    <img src="${'/logo.png?company=%s' % object.company_id.id}" style="padding: 0px; margin: 0px; height: auto; width: 80px;" alt="${'%s' % object.company_id.name}"/>
                    % endif
                </td></tr>
                <tr><td colspan="2" style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin:16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- EVENT DESCRIPTION -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table width="590" border="0" cellpadding="0" cellspacing="0" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="top" style="font-size: 14px;">
                    <div>
                        Hello ${object.name},<br/>
                        We are excited to remind you that the event
                        % if is_online:
                            <a href="${object.event_id.website_url}" style="color:#875A7B;text-decoration:none;">${object.event_id.name}</a>
                        % else:
                            <strong>${object.event_id.name}</strong>
                        % endif
                        is starting <strong>${object.get_date_range_str()}</strong>.
                    </div>
                    <div>
                        <br />
                        <strong>Add this event to your calendar</strong>
                        <a href="https://www.google.com/calendar/render?action=TEMPLATE&amp;text=${object.event_id.name}&amp;dates=${date_begin}/${date_end}&amp;location=${location}" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;" target="new"><img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> Google</a>
                        <a href="/event/${slug(object.event_id)}/ics" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;"><img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> iCal/Outlook</a>
                        <a href="https://calendar.yahoo.com/?v=60&amp;view=d&amp;type=20&amp;title=${object.event_id.name}&amp;in_loc=${location}&amp;st=${format_datetime(object.event_id.date_begin, tz='UTC', dt_format='yyyyMMdd\'T\'HHmmss')}&amp;et=${format_datetime(object.event_id.date_end, tz='UTC', dt_format='yyyyMMdd\'T\'HHmmss')}" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;" target="new">
                            <img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> Yahoo
                        </a>
                        <br /><br />
                    </div>
                    <div>
                        We confirm your registration and hope to meet you there,<br/>
                        <span style="color: #454748;">
                        -- <br/>
                        % if event_organizer:
                            ${event_organizer.name}
                        % else:
                            The ${object.event_id.name} Team
                        % endif
                        </span>
                    </div>
                </td></tr>
                <tr><td style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- DETAILS -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table width="590" border="0" cellpadding="0" cellspacing="0" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="top" style="font-size: 14px;">
                    <table style="width:100%;">
                        <tr>
                            <td style="vertical-align:top;">
                                <img src="/web_editor/font_to_img/61555/rgb(81,81,102)/34" style="padding:4px;max-width:inherit;" height="34" alt=""/>
                            </td>
                            <td style="padding: 0px 10px 0px 10px;width:50%;line-height:20px;vertical-align:top;">
                                <div><strong>From</strong> ${object.event_id.date_begin_located}</div>
                                <div><strong>To</strong> ${object.event_id.date_end_located}</div>
                                <div style="font-size:12px;color:#9e9e9e"><i><strong>TZ</strong> ${object.event_id.date_tz}</i></div>
                            </td>
                            <td style="vertical-align:top;">
                                % if event_address:
                                <img src="/web_editor/font_to_img/61505/rgb(81,81,102)/34" style="padding:4px;max-width:inherit;" height="34" alt=""/>
                                % endif
                            </td>
                            <td style="padding: 0px 10px 0px 10px;width:50%;vertical-align:top;">
                                % if event_address:
                                % set location = ''
                                % if object.event_id.address_id.name:
                                    <div>${object.event_id.address_id.name}</div>
                                % endif
                                % if object.event_id.address_id.street:
                                    <div>${object.event_id.address_id.street}</div>
                                    % set location = object.event_id.address_id.street
                                % endif
                                % if object.event_id.address_id.street2:
                                    <div>${object.event_id.address_id.street2}</div>
                                    % set location = '%s, %s' % (location, object.event_id.address_id.street2)
                                % endif
                                <div>
                                % if object.event_id.address_id.city:
                                    ${object.event_id.address_id.city},
                                    % set location = '%s, %s' % (location, object.event_id.address_id.city)
                                % endif
                                % if object.event_id.address_id.state_id.name:
                                    ${object.event_id.address_id.state_id.name},
                                    % set location = '%s, %s' % (location, object.event_id.address_id.state_id.name)
                                % endif
                                % if object.event_id.address_id.zip:
                                    ${object.event_id.address_id.zip}
                                    % set location = '%s, %s' % (location, object.event_id.address_id.zip)
                                % endif
                                </div>
                                % if object.event_id.address_id.country_id.name:
                                    <div>${object.event_id.address_id.country_id.name}</div>
                                    % set location = '%s, %s' % (location, object.event_id.address_id.country_id.name)
                                % endif
                                % endif
                            </td>
                        </tr>
                    </table>
                </td></tr>
                <tr><td style="text-align:center;">
                    % if event_organizer
                    <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                    % endif
                </td></tr>

                <tr><td valign="top" style="font-size: 14px;">
                    <!-- CONTACT ORGANIZER -->
                    % if event_organizer:
                    <div>
                        <span style="font-weight:300;margin:10px 0px">Questions about this event?</span>
                        <div>Please contact the organizer:</div>
                        <ul>
                            <li>${event_organizer.name}</li>
                            % if event_organizer.email
                                <li>Mail: <a href="mailto:${event_organizer.email}" style="text-decoration:none;color:#875A7B;">${event_organizer.email}</a></li>
                            % endif
                            % if event_organizer.phone
                                <li>Phone: ${event_organizer.phone}</li>
                            % endif
                        </ul>
                    </div>
                    % endif
                </td></tr>
                <tr><td style="text-align:center;">
                    <!-- CONTACT ORGANIZER SEPARATION -->
                    % if is_online or event_address:
                    <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                    % endif
                </td></tr>

                <tr><td valign="top" style="font-size: 14px;">
                    <!-- PWA MARKGETING -->
                    % if is_online:
                    <div>
                        <strong>Get the best mobile experience.</strong>
                        <a href="/event">Install our mobile app</a>
                    </div>
                    % endif
                </td></tr>
                <tr><td style="text-align:center;">
                    <!-- PWA MARKGETING SEPARATION-->
                    % if is_online and event_address:
                    <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                    % endif
                </td></tr>

                <tr><td valign="top" style="font-size: 14px;">
                    <!-- GOOGLE MAPS LINK -->
                    % if event_address:
                    <table style="width:100%;"><tr><td>
                        <div>
                            <a href="https://maps.google.com/maps?q=${location}" target="new">
                                <img src="http://maps.googleapis.com/maps/api/staticmap?autoscale=1&amp;size=598x200&amp;maptype=roadmap&amp;format=png&amp;visual_refresh=true&amp;markers=size:mid%7Ccolor:0xa5117d%7Clabel:%7C${location}" style="vertical-align:bottom; width: 100%;" alt="Google Maps"/>
                            </a>
                        </div>
                    </td></tr></table>
                    % endif
                </td></tr>
            </table>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- FOOTER BY -->
<tr><td align="center" style="min-width: 590px;">
    % if object.company_id
    <table width="590" border="0" cellpadding="0" cellspacing="0" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
      <tr><td style="text-align: center; font-size: 14px;">
        Sent by <a target="_blank" href="${object.company_id.website}" style="color: #875A7B;">${object.company_id.name}</a>
        % if 'website_url' in object.event_id and object.event_id.website_url:
        <br />
        Discover <a href="/event" style="text-decoration:none;color:#717188;">all our events</a>.
        % endif
      </td></tr>
    </table>
    % endif
</td></tr>
</table>
            </field>
            <field name="lang">${object.partner_id.lang}</field>
        </record>

    </data>
</odoo>

```

## File: data\event_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Event Mail Scheduler-->
        <record model="ir.cron" forcecreate="True" id="event_mail_scheduler">
            <field name="name">Event: Mail Scheduler</field>
            <field name="model_id" ref="model_event_mail"/>
            <field name="state">code</field>
            <field name="code">model.run(True)</field>
            <field name="user_id" ref="base.user_root"/>
            <field name="interval_number">1</field>
            <field name="interval_type">hours</field>
            <field name="numbercall">-1</field>
            <field name="doall" eval="False" />
        </record>

        <!-- Event Categories -->
        <record id="event_type_data_ticket" model="event.type">
            <field name="name">Ticketing</field>
            <field name="auto_confirm" eval="False"/>
            <field name="use_ticket" eval="True"/>
        </record>
        <record id="event_type_data_conference" model="event.type">
            <field name="name">Conference</field>
            <field name="auto_confirm" eval="True"/>
        </record>

        <!-- Event stages -->
        <record id="event_stage_new" model="event.stage">
            <field name="name">New</field>
            <field name="description">Freshly created</field>
            <field name="sequence">1</field>
        </record>
        <record id="event_stage_booked" model="event.stage">
            <field name="name">Booked</field>
            <field name="description">The place has been reserved</field>
            <field name="sequence">2</field>
        </record>
        <record id="event_stage_announced" model="event.stage">
            <field name="name">Announced</field>
            <field name="description">The event has been publicly announced</field>
            <field name="sequence">3</field>
        </record>
        <record id="event_stage_done" model="event.stage">
            <field name="name">Ended</field>
            <field name="description">Fully ended</field>
            <field name="sequence">5</field>
            <field name="pipe_end" eval="True"/>
            <field name="fold" eval="True"/>
        </record>
        <record id="event_stage_cancelled" model="event.stage">
            <field name="name">Cancelled</field>
            <field name="description">The event has been cancelled</field>
            <field name="sequence">6</field>
            <field name="pipe_end" eval="True"/>
            <field name="fold" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: data\event_demo.xml

```xml
<?xml version="1.0"?>
<odoo><data>

    <!-- Event -->
    <record id="event.event_0" model="event.event">
        <field name="name">Design Fair Los Angeles</field>
        <field name="user_id" ref="base.user_demo"/>
        <field name="date_begin" eval="(DateTime.now() + timedelta(days=10)).strftime('%Y-%m-%d 08:00:00')"/>
        <field name="date_end" eval="(DateTime.now() + timedelta(days=14)).strftime('%Y-%m-%d 18:00:00')"/>
        <field name="seats_limited">True</field>
        <field name="seats_max">50</field>
        <field name="address_id" ref="event.res_partner_location_2"/>
        <field name="date_tz">US/Pacific</field>
        <field name="event_type_id" ref="event_type_0"/>
        <field name="stage_id" ref="event_stage_booked"/>
        <field name="tag_ids" eval="[(4, ref('event.event_tag_category_1_tag_1')), (4, ref('event.event_tag_category_2_tag_1'))]"/>
    </record>
    <record id="event_0_ticket_0" model="event.event.ticket">
        <field name="name">Free</field>
        <field name="description">Free entrance, no food !</field>
        <field name="event_id" ref="event.event_0"/>
        <field name="start_sale_date" eval="(DateTime.today() + timedelta(days=5)).strftime('%Y-%m-%d')"/>
        <field name="end_sale_date" eval="(DateTime.today() + timedelta(days=10)).strftime('%Y-%m-%d')"/>
        <field name="seats_max">0</field>
    </record>
    <record id="event_0_ticket_1" model="event.event.ticket">
        <field name="name">Standard</field>
        <field name="description">For only 10, you gain access to catering. Yum yum.</field>
        <field name="event_id" ref="event.event_0"/>
        <field name="start_sale_date" eval="(DateTime.today() + timedelta(days=5)).strftime('%Y-%m-%d')"/>
        <field name="end_sale_date" eval="(DateTime.today() + timedelta(days=10)).strftime('%Y-%m-%d')"/>
        <field name="seats_max">50</field>
    </record>
    <record id="event_0_ticket_2" model="event.event.ticket">
        <field name="name">VIP</field>
        <field name="description">You are truly among the best.</field>
        <field name="event_id" ref="event.event_0"/>
        <field name="start_sale_date" eval="(DateTime.today() + timedelta(days=5)).strftime('%Y-%m-%d')"/>
        <field name="end_sale_date" eval="(DateTime.today() + timedelta(days=10)).strftime('%Y-%m-%d')"/>
        <field name="seats_max">10</field>
    </record>

    <record id="event.event_1" model="event.event">
        <field name="name">Great Reno Ballon Race</field>
        <field name="user_id" ref="base.user_admin"/>
        <field eval="(DateTime.today()+ timedelta(days=100)).strftime('%Y-%m-%d 20:15:00')" name="date_begin"/>
        <field eval="(DateTime.today()+ timedelta(days=101)).strftime('%Y-%m-%d 00:30:00')" name="date_end"/>
        <field name="event_type_id" ref="event_type_2"/>
        <field name="address_id" ref="event.res_partner_location_0"/>
        <field name="stage_id" ref="event_stage_booked"/>
        <field name="kanban_state">blocked</field>
        <field name="tag_ids" eval="[(4, ref('event.event_tag_category_1_tag_4')), (4, ref('event.event_tag_category_2_tag_3'))]"/>
    </record>

    <record id="message_event_1_0" model="mail.message">
        <field name="model">event.event</field>
        <field name="res_id" ref="event.event_1"/>
        <field name="body" type="html"><p>Hello Marc Demo,<br/>
            Our flight authorizations have been revoked due to insurance issues.<br/>
            Could you take care of it as soon as possible ?</p>
        </field>
        <field name="message_type">comment</field>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="author_id" ref="base.partner_admin"/>
    </record>
    <record id="message_event_1_1" model="mail.message">
        <field name="model">event.event</field>
        <field name="res_id" ref="event.event_1"/>
        <field name="parent_id" ref="message_event_1_0"/>
        <field name="body" type="html"><p>Hi Mitchell Admin,<br/>I will take care of it today !</p></field>
        <field name="message_type">comment</field>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="author_id" ref="base.partner_demo"/>
    </record>
    <record id="message_event_1_2" model="mail.message">
        <field name="model">event.event</field>
        <field name="res_id" ref="event.event_1"/>
        <field name="parent_id" ref="message_event_1_1"/>
        <field name="body" type="html"><p>Great ! This event will stay "blocked" until it is fixed.<br/>
        Feel free to green it once everything is in order.</p>
        </field>
        <field name="message_type">comment</field>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="author_id" ref="base.partner_admin"/>
    </record>
    <record id="activity_event_1_0" model="mail.activity">
        <field name="res_id" ref="event.event_1" />
        <field name="res_model_id" ref="event.model_event_event"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_call"/>
        <field name="summary">Call the local state house.</field>
        <field name="date_deadline" eval="DateTime.today()"/>
        <field name="create_uid" ref="base.user_demo"/>
        <field name="user_id" ref="base.user_demo"/>
    </record>

    <record id="event_2" model="event.event">
        <field name="name">Conference for Architects</field>
        <field name="user_id" ref="base.user_admin"/>
        <field eval="(DateTime.today()+ timedelta(days=5)).strftime('%Y-%m-%d 07:00:00')" name="date_begin"/>
        <field eval="(DateTime.today()+ timedelta(days=5)).strftime('%Y-%m-%d 16:30:00')" name="date_end"/>
        <field name="event_type_id" ref="event_type_data_conference"/>
        <field name="address_id" ref="event.res_partner_location_2"/>
        <field name="seats_limited">True</field>
        <field name="seats_max">200</field>
        <field name="stage_id" ref="event_stage_booked"/>
        <field name="tag_ids" eval="[(4, ref('event.event_tag_category_1_tag_4')), (4, ref('event.event_tag_category_2_tag_1'))]"/>
    </record>
    <record id="event_2_ticket_1" model="event.event.ticket">
        <field name="name">Standard</field>
        <field name="event_id" ref="event.event_2"/>
        <field name="end_sale_date" eval="(DateTime.today() + timedelta(90)).strftime('%Y-%m-%d')"/>
        <field name="seats_max">50</field>
    </record>
    <record id="event_2_ticket_2" model="event.event.ticket">
        <field name="name">VIP</field>
        <field name="event_id" ref="event.event_2"/>
        <field name="end_sale_date" eval="(DateTime.today() + timedelta(60)).strftime('%Y-%m-%d')"/>
        <field name="seats_max">5</field>
    </record>
    <record id="activity_event_2_0" model="mail.activity">
        <field name="res_id" ref="event.event_2" />
        <field name="res_model_id" ref="event.model_event_event"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_call"/>
        <field name="summary">Call the caterer.</field>
        <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=3)).strftime('%Y-%m-%d %H:%M')"/>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_2_mail_0" model="event.mail">
        <field name="event_id" ref="event.event_2"/>
        <field name="template_id" ref="event.event_subscription"/>
    </record>

    <record id="event.event_3" model="event.event">
        <field name="name">Live Music Festival</field>
        <field name="user_id" ref="base.user_demo"/>
        <field name="date_begin" eval="(DateTime.today()+ timedelta(days=130)).strftime('%Y-%m-%d 20:15:00')"/>
        <field name="date_end" eval="(DateTime.today()+ timedelta(days=133)).strftime('%Y-%m-%d 00:30:00')"/>
        <field name="event_type_id" ref="event_type_0"/>
        <field name="address_id" ref="event.res_partner_location_1"/>
        <field name="stage_id" ref="event_stage_announced"/>
        <field name="tag_ids" eval="[(4, ref('event.event_tag_category_1_tag_3')), (4, ref('event.event_tag_category_2_tag_2'))]"/>
    </record>
    <record id="event_3_ticket_0" model="event.event.ticket">
        <field name="name">Standard</field>
        <field name="event_id" ref="event.event_3"/>
        <field name="end_sale_date" eval="(DateTime.today() + timedelta(days=20)).strftime('%Y-%m-%d')"/>
        <field name="seats_max">1200</field>
    </record>
    <record id="event_3_ticket_1" model="event.event.ticket">
        <field name="name">VIP</field>
        <field name="event_id" ref="event.event_3"/>
        <field name="end_sale_date" eval="(DateTime.today() + timedelta(days=20)).strftime('%Y-%m-%d')"/>
        <field name="seats_max">50</field>
    </record>
    <record id="activity_event_3_0" model="mail.activity">
        <field name="res_id" ref="event.event_3" />
        <field name="res_model_id" ref="event.model_event_event"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_call"/>
        <field name="summary">Prepare interview with local media.</field>
        <field name="date_deadline" eval="DateTime.today().strftime('%Y-%m-%d %H:%M')"/>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_3_mail_0" model="event.mail">
        <field name="event_id" ref="event.event_3"/>
        <field name="template_id" ref="event.event_subscription"/>
    </record>

    <!-- EVENT_4: very limited, intended to test seats reservation -->
    <record id="event.event_4" model="event.event">
        <field name="name">Business workshops</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="date_begin" eval="(DateTime.today() - timedelta(days=5)).strftime('%Y-%m-%d 18:00:00')"/>
        <field name="date_end" eval="(DateTime.today() - timedelta(days=5)).strftime('%Y-%m-%d 22:30:00')"/>
        <field name="seats_limited">True</field>
        <field name="seats_max">4</field>
        <field name="address_id" ref="event.res_partner_location_2"/>
        <field name="date_tz">US/Pacific</field>
        <field name="event_type_id" ref="event_type_1"/>
        <field name="stage_id" ref="event_stage_done"/>
        <field name="kanban_state">done</field>
        <field name="tag_ids" eval="[(4, ref('event.event_tag_category_1_tag_4')), (4, ref('event.event_tag_category_2_tag_1'))]"/>
    </record>
    <record id="event_4_ticket_0" model="event.event.ticket">
        <field name="name">General Admission</field>
        <field name="event_id" ref="event.event_4"/>
        <field name="end_sale_date" eval="(DateTime.today() - timedelta(30)).strftime('%Y-%m-%d')"/>
        <field name="seats_max">4</field>
    </record>
    <record id="activity_event_4_0" model="mail.activity">
        <field name="res_id" ref="event.event_4" />
        <field name="res_model_id" ref="event.model_event_event"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_call"/>
        <field name="summary">Prepare after movie.</field>
        <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=3)).strftime('%Y-%m-%d %H:%M')"/>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>

    <record id="event.event_5" model="event.event">
        <field name="name">Hockey Tournament</field>
        <field name="user_id" ref="base.user_demo"/>
        <field eval="(DateTime.today()+ timedelta(days=370)).strftime('%Y-%m-%d 09:00:00')" name="date_begin"/>
        <field eval="(DateTime.today()+ timedelta(days=371)).strftime('%Y-%m-%d 17:00:00')" name="date_end"/>
        <field name="event_type_id" ref="event_type_2"/>
        <field name="address_id" ref="event.res_partner_location_1"/>
        <field name="tag_ids" eval="[(6, 0, [ref('event.event_tag_category_1_tag_2'), ref('event.event_tag_category_2_tag_3')])]"/>
    </record>

    <record id="event.event_6" model="event.event">
        <field name="name">An unpublished event</field>
        <field name="user_id" ref="base.user_admin"/>
        <field eval="(DateTime.today()+ timedelta(days=30)).strftime('%Y-%m-%d 09:30:00')" name="date_begin"/>
        <field eval="(DateTime.today()+ timedelta(days=30)).strftime('%Y-%m-%d 17:30:00')" name="date_end"/>
        <field name="event_type_id" ref="event_type_0"/>
        <field name="address_id" ref="event.res_partner_location_1"/>
    </record>

    <record id="event.event_7" model="event.event">
        <field name="name">OpenWood Collection Online Reveal</field>
        <field name="date_tz">Europe/Brussels</field>
        <field name="event_type_id" ref="event_type_0"/>
        <field name="stage_id" ref="event.event_stage_booked"/>
        <field name="user_id" ref="base.user_demo"/>
        <field name="auto_confirm" eval="True"/>
        <field name="date_begin" eval="(DateTime.now() - timedelta(days=1)).strftime('%Y-%m-%d 05:00:00')"/>
        <field name="date_end" eval="(DateTime.now() + timedelta(days=1)).strftime('%Y-%m-%d 15:00:00')"/>
        <field name="address_id" eval="False"/>
        <field name="tag_ids" eval="[(4, ref('event.event_tag_category_3_tag_1'))]"/>
        <field name="description" type="html">
<div class="oe_structure">
    <h5>The finest OpenWood furnitures are coming to your house in a brand new collection</h5>
    <p>And this time, we go fully ONLINE! Meet us in our live streams from the comfort of your house.<br/>
    Special discount codes will be handed out during the various streams, make sure to be there on time.</p>
    <p class="mb-3">For any additional information, please contact us at <a href="mailto:events@idea.com">events@idea.com</a>.</p>
    <div class="bg-light rounded-right border-left border-secondary p-3 mb-5" style="border-left-width: 3px !important;">
        <p class="mb-1">This event is fully online and FREE, if you have paid for tickets, you should get a refund.<br/>
        It will require a good Internet connection to get the best video quality.</p>
    </div>
</div>
        </field>
    </record>
    <record id="event_7_ticket_1" model="event.event.ticket">
        <field name="name">Standard</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="end_sale_date" eval="(DateTime.now() + timedelta(days=2)).strftime('%Y-%m-%d 15:00:00')"/>
    </record>
    <record id="event_7_ticket_2" model="event.event.ticket">
        <field name="name">VIP</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="end_sale_date" eval="(DateTime.now() + timedelta(days=2)).strftime('%Y-%m-%d 15:00:00')"/>
        <field name="seats_max">10</field>
    </record>

</data></odoo>

```

## File: data\event_demo_misc.xml

```xml
<?xml version="1.0"?>
<odoo><data>

    <!-- Event Type -->
    <record id="event_type_0" model="event.type">
        <field name="name">Exhibition</field>
        <field name="auto_confirm" eval="False"/>
        <field name="use_mail_schedule" eval="False"/>
    </record>
    <record id="event_type_1" model="event.type">
        <field name="name">Training</field>
        <field name="auto_confirm" eval="False"/>
        <field name="use_mail_schedule" eval="True"/>
    </record>
    <record id="event_type_2" model="event.type">
        <field name="name">Sport</field>
        <field name="auto_confirm" eval="False"/>
        <field name="use_mail_schedule" eval="False"/>
        <field name="use_timezone" eval="True"/>
        <field name="default_timezone">US/Pacific</field>
    </record>
    <record id="event_type_data_conference" model="event.type">
        <field name="use_timezone" eval="True"/>
        <field name="default_timezone">Europe/Brussels</field>
    </record>

    <!-- Category and Tags -->
    <record id="event_tag_category_1" model="event.tag.category">
        <field name="name">Age</field>
        <field name="sequence">3</field>
    </record>
    <record id="event_tag_category_2" model="event.tag.category">
        <field name="name">Activity</field>
        <field name="sequence">1</field>
    </record>
    <record id="event_tag_category_3" model="event.tag.category">
        <field name="name">Type</field>
        <field name="sequence">2</field>
    </record>

    <record id="event_tag_category_1_tag_1" model="event.tag">
        <field name="name">5-10</field>
        <field name="sequence">1</field>
        <field name="category_id" ref="event_tag_category_1"/>
        <field name="color">1</field>
    </record>

    <record id="event_tag_category_1_tag_2" model="event.tag">
        <field name="name">10-14</field>
        <field name="sequence">2</field>
        <field name="category_id" ref="event_tag_category_1"/>
        <field name="color">2</field>
    </record>

    <record id="event_tag_category_1_tag_3" model="event.tag">
        <field name="name">15-18</field>
        <field name="sequence">3</field>
        <field name="category_id" ref="event_tag_category_1"/>
        <field name="color">3</field>
    </record>

    <record id="event_tag_category_1_tag_4" model="event.tag">
        <field name="name">18+</field>
        <field name="sequence">4</field>
        <field name="category_id" ref="event_tag_category_1"/>
        <field name="color">4</field>
    </record>

    <record id="event_tag_category_2_tag_1" model="event.tag">
        <field name="name">Culture</field>
        <field name="sequence">10</field>
        <field name="category_id" ref="event_tag_category_2"/>
        <field name="color">5</field>
    </record>
    <record id="event_tag_category_2_tag_2" model="event.tag">
        <field name="name">Music</field>
        <field name="sequence">11</field>
        <field name="category_id" ref="event_tag_category_2"/>
        <field name="color">6</field>
    </record>
    <record id="event_tag_category_2_tag_3" model="event.tag">
        <field name="name">Sport</field>
        <field name="sequence">12</field>
        <field name="category_id" ref="event_tag_category_2"/>
        <field name="color">7</field>
    </record>

    <record id="event_tag_category_3_tag_1" model="event.tag">
        <field name="name">Online</field>
        <field name="sequence">20</field>
        <field name="category_id" ref="event_tag_category_3"/>
        <field name="color">8</field>
    </record>
    <record id="event_tag_category_3_tag_2" model="event.tag">
        <field name="name">Conference</field>
        <field name="sequence">21</field>
        <field name="category_id" ref="event_tag_category_3"/>
        <field name="color">9</field>
    </record>

</data></odoo>

```

## File: data\event_registration_demo.xml

```xml
<?xml version="1.0"?>
<odoo><data>
    <!-- Design fair -->
    <record id="event_registration_0_0" model="event.registration">
        <field name="event_id" ref="event.event_0"/>
        <field name="event_ticket_id" ref="event.event_0_ticket_1"/>
        <field name="partner_id" ref="base.res_partner_address_1"/>
    </record>
    <record id="event_registration_0_1" model="event.registration">
        <field name="event_id" ref="event.event_0"/>
        <field name="event_ticket_id" ref="event.event_0_ticket_1"/>
        <field name="partner_id" ref="base.res_partner_address_2"/>
    </record>
    <record id="event_registration_0_2" model="event.registration">
        <field name="event_id" ref="event.event_0"/>
        <field name="event_ticket_id" ref="event.event_0_ticket_0"/>
        <field name="name">Tucker Carlson</field>
        <field name="email">tuck@test.example.com</field>
        <field name="partner_id" eval="False"/>
    </record>

    <!-- Reno Ballon Race -->
    <record id="event_registration_1_0" model="event.registration">
        <field name="event_id" ref="event.event_1"/>
        <field name="partner_id" ref="base.res_partner_address_1"/>
    </record>
    <record id="event_registration_1_1" model="event.registration">
        <field name="event_id" ref="event.event_1"/>
        <field name="partner_id" ref="base.res_partner_address_2"/>
    </record>
    <record id="event_registration_1_2" model="event.registration">
        <field name="event_id" ref="event.event_1"/>
        <field name="name">Piers Morgan</field>
        <field name="email">piersm@test.example.com</field>
        <field name="partner_id" eval="False"/>
    </record>
    <record id="event_registration_1_3" model="event.registration">
        <field name="event_id" ref="event.event_1"/>
        <field name="partner_id" ref="base.res_partner_address_3"/>
    </record>
    <record id="event_registration_1_4" model="event.registration">
        <field name="event_id" ref="event.event_1"/>
        <field name="partner_id" ref="base.res_partner_address_4"/>
    </record>
    <record id="event_registration_1_5" model="event.registration">
        <field name="event_id" ref="event.event_1"/>
        <field name="name">Nigel Woodfire</field>
        <field name="email">nigelw@test.example.com</field>
        <field name="partner_id" eval="False"/>
    </record>

    <!-- Conference for architects -->
    <record id="event_registration_2_0" model="event.registration">
        <field name="event_id" ref="event.event_2"/>
        <field name="event_ticket_id" ref="event.event_2_ticket_1"/>
        <field name="partner_id" ref="base.res_partner_address_1"/>
    </record>
    <record id="event_registration_2_1" model="event.registration">
        <field name="event_id" ref="event.event_2"/>
        <field name="event_ticket_id" ref="event.event_2_ticket_1"/>
        <field name="partner_id" ref="base.res_partner_address_2"/>
    </record>
    <record id="event_registration_2_2" model="event.registration">
        <field name="event_id" ref="event.event_2"/>
        <field name="event_ticket_id" ref="event.event_2_ticket_2"/>
        <field name="name">Piers Morgan</field>
        <field name="email">piersm@test.example.com</field>
        <field name="partner_id" eval="False"/>
    </record>
    <record id="event_registration_2_3" model="event.registration">
        <field name="event_id" ref="event.event_2"/>
        <field name="event_ticket_id" ref="event.event_2_ticket_1"/>
        <field name="partner_id" ref="base.res_partner_address_3"/>
    </record>
    <record id="event_registration_2_4" model="event.registration">
        <field name="event_id" ref="event.event_2"/>
        <field name="event_ticket_id" ref="event.event_2_ticket_1"/>
        <field name="partner_id" ref="base.res_partner_address_4"/>
    </record>

    <!-- Live Music Festival -->
    <record id="event_registration_3_0" model="event.registration">
        <field name="event_id" ref="event.event_3"/>
        <field name="partner_id" ref="base.res_partner_address_1"/>
    </record>
    <record id="event_registration_3_1" model="event.registration">
        <field name="event_id" ref="event.event_3"/>
        <field name="partner_id" ref="base.res_partner_address_2"/>
    </record>
    <record id="event_registration_3_2" model="event.registration">
        <field name="event_id" ref="event.event_3"/>
        <field name="name">Piers Morgan</field>
        <field name="email">piersm@test.example.com</field>
        <field name="partner_id" eval="False"/>
    </record>
    <record id="event_registration_3_3" model="event.registration">
        <field name="event_id" ref="event.event_3"/>
        <field name="partner_id" ref="base.res_partner_address_3"/>
    </record>
    <record id="event_registration_3_4" model="event.registration">
        <field name="event_id" ref="event.event_3"/>
        <field name="partner_id" ref="base.res_partner_address_4"/>
    </record>
    <record id="event_registration_3_5" model="event.registration">
        <field name="event_id" ref="event.event_3"/>
        <field name="name">Nigel Woodfire</field>
        <field name="email">nigelw@test.example.com</field>
        <field name="partner_id" eval="False"/>
    </record>

    <!-- Business Workshop -->
    <record id="event_registration_4_0" model="event.registration">
        <field name="event_id" ref="event.event_4"/>
        <field name="event_ticket_id" ref="event.event_4_ticket_0"/>
        <field name="partner_id" ref="base.res_partner_address_7"/>
    </record>
    <record id="event_registration_4_1" model="event.registration">
        <field name="event_id" ref="event.event_4"/>
        <field name="event_ticket_id" ref="event.event_4_ticket_0"/>
        <field name="partner_id" ref="base.res_partner_address_13"/>
    </record>
    <record id="event_registration_4_2" model="event.registration">
        <field name="event_id" ref="event.event_4"/>
        <field name="event_ticket_id" ref="event.event_4_ticket_0"/>
        <field name="partner_id" ref="base.res_partner_address_14"/>
    </record>

    <!-- OpenWood Collection Online Reveal: Gemini (all) -->
    <record id="event_registration_7_0" model="event.registration">
        <field name="event_id" ref="event.event_7"/>
        <field name="event_ticket_id" ref="event.event_7_ticket_1"/>
        <field name="partner_id" ref="base.res_partner_address_5"/>
    </record>
    <record id="event_registration_7_1" model="event.registration">
        <field name="event_id" ref="event.event_7"/>
        <field name="event_ticket_id" ref="event.event_7_ticket_1"/>
        <field name="partner_id" ref="base.res_partner_address_10"/>
    </record>
    <record id="event_registration_7_2" model="event.registration">
        <field name="event_id" ref="event.event_7"/>
        <field name="event_ticket_id" ref="event.event_7_ticket_2"/>
        <field name="partner_id" ref="base.res_partner_address_11"/>
    </record>
    <record id="event_registration_7_3" model="event.registration">
        <field name="event_id" ref="event.event_7"/>
        <field name="event_ticket_id" ref="event.event_7_ticket_2"/>
        <field name="partner_id" ref="base.res_partner_address_25"/>
    </record>

    <function model="event.registration"
        name="action_confirm"
        eval="[[ref('event_registration_0_0'), ref('event_registration_0_1'),
                ref('event_registration_1_0'), ref('event_registration_1_1'), ref('event_registration_1_2'),
                ref('event_registration_2_0'), ref('event_registration_2_1'), ref('event_registration_2_2'), ref('event_registration_2_3'),
                ref('event_registration_4_2')]]"
    />

    <function model="event.registration"
        name="action_set_done"
        eval="[[ref('event_registration_4_0'), ref('event_registration_4_1')]]"
    />

</data></odoo>
```

## File: data\res_partner_demo.xml

```xml
<?xml version="1.0"?>
<odoo><data>

    <!-- LOCATIONS -->
    <record id="res_partner_location_0" model="res.partner">
        <field name="name">Reno Airfield</field>
        <field name="is_company">1</field>
        <field name="street">1235 Columbia Hill Rd</field>
        <field name="city">Reno</field>
        <field name="state_id" ref='base.state_us_23'/>
        <field name="zip">89508</field>
        <field name="country_id" ref="base.us"/>
    </record>

    <record id="res_partner_location_1" model="res.partner">
        <field name="name">Wembley Stadium</field>
        <field name="is_company">1</field>
        <field name="street">Wembley HA9 0WS</field>
        <field name="city">London</field>
        <field name="state_id" ref='base.state_uk117'/>
        <field name="country_id" ref="base.uk"/>
    </record>

    <record id="res_partner_location_2" model="res.partner">
        <field name="name">Los Angeles Convention Center</field>
        <field name="is_company">1</field>
        <field name="street">1201 S Figueroa St</field>
        <field name="city">Los Angeles</field>
        <field name="state_id" ref='base.state_us_5'/>
        <field name="zip">90015</field>
        <field name="country_id" ref="base.us"/>
    </record>

    <!-- SPONSORS / OTHER COUNTRIES -->
    <record id="res_partner_event_1" model="res.partner">
        <field name="name">Bloem GmbH</field>
        <field name="is_company" eval="True"/>
        <field name="image_1920" type="base64" file="event/static/src/img/partner_bloem.png"/>
        <field name="street">Behrenstraße 55</field>
        <field name="zip">10117</field>
        <field name="city">Berlin</field>
        <field name="country_id" ref="base.de"/>
        <field name="phone">+49 30 12345678</field>
        <field name="mobile">+49 30 87654321</field>
        <field name="email">flower@example.com</field>
        <field name="website">www.flower.example.com</field>
    </record>
    <record id="res_partner_event_2" model="res.partner">
        <field name="name">OpenWood</field>
        <field name="is_company" eval="True"/>
        <field name="image_1920" type="base64" file="event/static/src/img/partner_open_wood.png"/>
        <field name="street">Orval 1</field>
        <field name="zip">6823</field>
        <field name="city">Florenville</field>
        <field name="country_id" ref="base.be"/>
        <field name="phone">+32 987 65 43 21</field>
        <field name="mobile">+32 987 65 43 21</field>
        <field name="email">wow@example.com</field>
        <field name="website">www.openwood.example.com</field>
    </record>
    <record id="res_partner_event_3" model="res.partner">
        <field name="name">Tree Dealers SP</field>
        <field name="is_company" eval="True"/>
        <field name="image_1920" type="base64" file="event/static/src/img/partner_tree_dealers.png"/>
        <field name="street">Place d'Youville, 995</field>
        <field name="zip">QC G1R 3P1</field>
        <field name="city">Ville de Quebec</field>
        <field name="country_id" ref="base.ca"/>
        <field name="phone">+1 418 123 4567</field>
        <field name="mobile">+1 418 765 4321</field>
        <field name="email">tree@example.com</field>
        <field name="website">www.tree.example.com</field>
    </record>
    <record id="res_partner_event_4" model="res.partner">
        <field name="name">Shangai Pterocarpus Furniture Co., Ltd.</field>
        <field name="is_company" eval="True"/>
        <field name="image_1920" type="base64" file="event/static/src/img/partner_pterocarpus.png"/>
        <field name="street">68 Taicang Rd, Shi Men Er Lu Jie Dao, Huangpu Qu</field>
        <field name="zip">200000</field>
        <field name="city">Shanghai Shi</field>
        <field name="country_id" ref="base.cn"/>
        <field name="phone">+86 21 1234 5678</field>
        <field name="mobile">+86 21 8765 4321</field>
        <field name="email">ptero@example.com</field>
        <field name="website">www.pterocarpus.example.com</field>
    </record>

</data></odoo>

```

## File: data\res_users_demo.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="base.user_demo" model="res.users">
        <field name="groups_id" eval="[(4, ref('event.group_event_user'))]"/>
    </record>
 </odoo>
```

## File: models\event_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pytz

from odoo import _, api, fields, models
from odoo.addons.base.models.res_partner import _tz_get
from odoo.tools import format_datetime
from odoo.exceptions import ValidationError
from odoo.tools.translate import html_translate

_logger = logging.getLogger(__name__)

try:
    import vobject
except ImportError:
    _logger.warning("`vobject` Python module not found, iCal file generation disabled. Consider installing this module if you want to generate iCal files")
    vobject = None


class EventType(models.Model):
    _name = 'event.type'
    _description = 'Event Template'
    _order = 'sequence, id'

    name = fields.Char('Event Template', required=True, translate=True)
    sequence = fields.Integer()
    # tickets
    use_ticket = fields.Boolean('Ticketing')
    event_type_ticket_ids = fields.One2many(
        'event.type.ticket', 'event_type_id',
        string='Tickets', compute='_compute_event_type_ticket_ids',
        readonly=False, store=True)
    tag_ids = fields.Many2many('event.tag', string="Tags")
    # registration
    has_seats_limitation = fields.Boolean('Limited Seats')
    seats_max = fields.Integer(
        'Maximum Registrations', compute='_compute_default_registration',
        readonly=False, store=True,
        help="It will select this default maximum value when you choose this event")
    auto_confirm = fields.Boolean(
        'Automatically Confirm Registrations', default=True,
        help="Events and registrations will automatically be confirmed "
             "upon creation, easing the flow for simple events.")
    # location
    use_timezone = fields.Boolean('Use Default Timezone')
    default_timezone = fields.Selection(
        _tz_get, string='Timezone', default=lambda self: self.env.user.tz or 'UTC')
    # communication
    use_mail_schedule = fields.Boolean(
        'Automatically Send Emails', default=True)
    event_type_mail_ids = fields.One2many(
        'event.type.mail', 'event_type_id',
        string='Mail Schedule', compute='_compute_event_type_mail_ids',
        readonly=False, store=True)

    @api.depends('use_mail_schedule')
    def _compute_event_type_mail_ids(self):
        for template in self:
            if not template.use_mail_schedule:
                template.event_type_mail_ids = [(5, 0)]
            elif not template.event_type_mail_ids:
                template.event_type_mail_ids = [(0, 0, {
                    'notification_type': 'mail',
                    'interval_unit': 'now',
                    'interval_type': 'after_sub',
                    'template_id': self.env.ref('event.event_subscription').id,
                }), (0, 0, {
                    'notification_type': 'mail',
                    'interval_nbr': 10,
                    'interval_unit': 'days',
                    'interval_type': 'before_event',
                    'template_id': self.env.ref('event.event_reminder').id,
                })]

    @api.depends('use_ticket')
    def _compute_event_type_ticket_ids(self):
        for template in self:
            if not template.use_ticket:
                template.event_type_ticket_ids = [(5, 0)]
            elif not template.event_type_ticket_ids:
                template.event_type_ticket_ids = [(0, 0, {
                    'name': _('Registration'),
                })]

    @api.depends('has_seats_limitation')
    def _compute_default_registration(self):
        for template in self:
            if not template.has_seats_limitation:
                template.seats_max = 0


class EventEvent(models.Model):
    """Event"""
    _name = 'event.event'
    _description = 'Event'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'date_begin'

    def _get_default_stage_id(self):
        event_stages = self.env['event.stage'].search([])
        return event_stages[0] if event_stages else False

    def _default_description(self):
        # avoid template branding with rendering_bundle=True
        return self.env['ir.ui.view'].with_context(rendering_bundle=True) \
            ._render_template('event.event_default_descripton')

    name = fields.Char(string='Event', translate=True, required=True)
    note = fields.Text(string='Note')
    description = fields.Html(string='Description', translate=html_translate, sanitize_attributes=False, sanitize_form=False, default=_default_description)
    active = fields.Boolean(default=True)
    user_id = fields.Many2one(
        'res.users', string='Responsible', tracking=True,
        default=lambda self: self.env.user)
    company_id = fields.Many2one(
        'res.company', string='Company', change_default=True,
        default=lambda self: self.env.company,
        required=False)
    organizer_id = fields.Many2one(
        'res.partner', string='Organizer', tracking=True,
        default=lambda self: self.env.company.partner_id,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    event_type_id = fields.Many2one('event.type', string='Template', ondelete='set null')
    event_mail_ids = fields.One2many(
        'event.mail', 'event_id', string='Mail Schedule', copy=True,
        compute='_compute_event_mail_ids', readonly=False, store=True)
    tag_ids = fields.Many2many(
        'event.tag', string="Tags", readonly=False,
        store=True, compute="_compute_tag_ids")
    # Kanban fields
    kanban_state = fields.Selection([('normal', 'In Progress'), ('done', 'Done'), ('blocked', 'Blocked')], default='normal')
    kanban_state_label = fields.Char(
        string='Kanban State Label', compute='_compute_kanban_state_label',
        store=True, tracking=True)
    stage_id = fields.Many2one(
        'event.stage', ondelete='restrict', default=_get_default_stage_id,
        group_expand='_read_group_stage_ids', tracking=True)
    legend_blocked = fields.Char(related='stage_id.legend_blocked', string='Kanban Blocked Explanation', readonly=True)
    legend_done = fields.Char(related='stage_id.legend_done', string='Kanban Valid Explanation', readonly=True)
    legend_normal = fields.Char(related='stage_id.legend_normal', string='Kanban Ongoing Explanation', readonly=True)
    # Seats and computation
    seats_max = fields.Integer(
        string='Maximum Attendees Number',
        compute='_compute_seats_max', readonly=False, store=True,
        help="For each event you can define a maximum registration of seats(number of attendees), above this numbers the registrations are not accepted.")
    seats_limited = fields.Boolean('Maximum Attendees', required=True, compute='_compute_seats_limited',
                                   readonly=False, store=True)
    seats_reserved = fields.Integer(
        string='Reserved Seats',
        store=True, readonly=True, compute='_compute_seats')
    seats_available = fields.Integer(
        string='Available Seats',
        store=True, readonly=True, compute='_compute_seats')
    seats_unconfirmed = fields.Integer(
        string='Unconfirmed Seat Reservations',
        store=True, readonly=True, compute='_compute_seats')
    seats_used = fields.Integer(
        string='Number of Participants',
        store=True, readonly=True, compute='_compute_seats')
    seats_expected = fields.Integer(
        string='Number of Expected Attendees',
        compute_sudo=True, readonly=True, compute='_compute_seats_expected')
    # Registration fields
    auto_confirm = fields.Boolean(
        string='Autoconfirmation', compute='_compute_auto_confirm', readonly=False, store=True,
        help='Autoconfirm Registrations. Registrations will automatically be confirmed upon creation.')
    registration_ids = fields.One2many('event.registration', 'event_id', string='Attendees')
    event_ticket_ids = fields.One2many(
        'event.event.ticket', 'event_id', string='Event Ticket', copy=True,
        compute='_compute_event_ticket_ids', readonly=False, store=True)
    event_registrations_open = fields.Boolean(
        'Registration open', compute='_compute_event_registrations_open', compute_sudo=True,
        help="Registrations are open if:\n"
        "- the event is not ended\n"
        "- there are seats available on event\n"
        "- the tickets are sellable (if ticketing is used)")
    event_registrations_sold_out = fields.Boolean(
        'Sold Out', compute='_compute_event_registrations_sold_out', compute_sudo=True,
        help='The event is sold out if no more seats are available on event. If ticketing is used and all tickets are sold out, the event will be sold out.')
    start_sale_date = fields.Date(
        'Start sale date', compute='_compute_start_sale_date',
        help='If ticketing is used, contains the earliest starting sale date of tickets.')
    # Date fields
    date_tz = fields.Selection(
        _tz_get, string='Timezone', required=True,
        compute='_compute_date_tz', readonly=False, store=True)
    date_begin = fields.Datetime(string='Start Date', required=True, tracking=True)
    date_end = fields.Datetime(string='End Date', required=True, tracking=True)
    date_begin_located = fields.Char(string='Start Date Located', compute='_compute_date_begin_tz')
    date_end_located = fields.Char(string='End Date Located', compute='_compute_date_end_tz')
    is_ongoing = fields.Boolean('Is Ongoing', compute='_compute_is_ongoing', search='_search_is_ongoing')
    is_one_day = fields.Boolean(compute='_compute_field_is_one_day')
    # Location and communication
    address_id = fields.Many2one(
        'res.partner', string='Venue', default=lambda self: self.env.company.partner_id.id,
        tracking=True, domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    country_id = fields.Many2one(
        'res.country', 'Country', related='address_id.country_id', readonly=False, store=True)
    # badge fields
    badge_front = fields.Html(string='Badge Front')
    badge_back = fields.Html(string='Badge Back')
    badge_innerleft = fields.Html(string='Badge Inner Left')
    badge_innerright = fields.Html(string='Badge Inner Right')
    event_logo = fields.Html(string='Event Logo')

    @api.depends('stage_id', 'kanban_state')
    def _compute_kanban_state_label(self):
        for event in self:
            if event.kanban_state == 'normal':
                event.kanban_state_label = event.stage_id.legend_normal
            elif event.kanban_state == 'blocked':
                event.kanban_state_label = event.stage_id.legend_blocked
            else:
                event.kanban_state_label = event.stage_id.legend_done

    @api.depends('seats_max', 'registration_ids.state')
    def _compute_seats(self):
        """ Determine reserved, available, reserved but unconfirmed and used seats. """
        # initialize fields to 0
        for event in self:
            event.seats_unconfirmed = event.seats_reserved = event.seats_used = event.seats_available = 0
        # aggregate registrations by event and by state
        state_field = {
            'draft': 'seats_unconfirmed',
            'open': 'seats_reserved',
            'done': 'seats_used',
        }
        base_vals = dict((fname, 0) for fname in state_field.values())
        results = dict((event_id, dict(base_vals)) for event_id in self.ids)
        if self.ids:
            query = """ SELECT event_id, state, count(event_id)
                        FROM event_registration
                        WHERE event_id IN %s AND state IN ('draft', 'open', 'done')
                        GROUP BY event_id, state
                    """
            self.env['event.registration'].flush(['event_id', 'state'])
            self._cr.execute(query, (tuple(self.ids),))
            res = self._cr.fetchall()
            for event_id, state, num in res:
                results[event_id][state_field[state]] = num

        # compute seats_available
        for event in self:
            event.update(results.get(event._origin.id or event.id, base_vals))
            if event.seats_max > 0:
                event.seats_available = event.seats_max - (event.seats_reserved + event.seats_used)

    @api.depends('seats_unconfirmed', 'seats_reserved', 'seats_used')
    def _compute_seats_expected(self):
        for event in self:
            event.seats_expected = event.seats_unconfirmed + event.seats_reserved + event.seats_used

    @api.depends('date_tz', 'start_sale_date', 'date_end', 'seats_available', 'seats_limited', 'event_ticket_ids.sale_available')
    def _compute_event_registrations_open(self):
        """ Compute whether people may take registrations for this event

          * event.date_end -> if event is done, registrations are not open anymore;
          * event.start_sale_date -> lowest start date of tickets (if any; start_sale_date
            is False if no ticket are defined, see _compute_start_sale_date);
          * any ticket is available for sale (seats available) if any;
          * seats are unlimited or seats are available;
        """
        for event in self:
            event = event._set_tz_context()
            current_datetime = fields.Datetime.context_timestamp(event, fields.Datetime.now())
            date_end_tz = event.date_end.astimezone(pytz.timezone(event.date_tz or 'UTC')) if event.date_end else False
            event.event_registrations_open = (event.start_sale_date <= current_datetime.date() if event.start_sale_date else True) and \
                (date_end_tz >= current_datetime if date_end_tz else True) and \
                (not event.seats_limited or event.seats_available) and \
                (not event.event_ticket_ids or any(ticket.sale_available for ticket in event.event_ticket_ids))

    @api.depends('event_ticket_ids.start_sale_date')
    def _compute_start_sale_date(self):
        """ Compute the start sale date of an event. Currently lowest starting sale
        date of tickets if they are used, of False. """
        for event in self:
            start_dates = [ticket.start_sale_date for ticket in event.event_ticket_ids if not ticket.is_expired]
            event.start_sale_date = min(start_dates) if start_dates and all(start_dates) else False

    @api.depends('event_ticket_ids.sale_available')
    def _compute_event_registrations_sold_out(self):
        for event in self:
            if event.seats_limited and not event.seats_available:
                event.event_registrations_sold_out = True
            elif event.event_ticket_ids:
                event.event_registrations_sold_out = not any(
                    ticket.seats_available > 0 if ticket.seats_limited else True for ticket in event.event_ticket_ids
                )
            else:
                event.event_registrations_sold_out = False

    @api.depends('date_tz', 'date_begin')
    def _compute_date_begin_tz(self):
        for event in self:
            if event.date_begin:
                event.date_begin_located = format_datetime(
                    self.env, event.date_begin, tz=event.date_tz, dt_format='medium')
            else:
                event.date_begin_located = False

    @api.depends('date_tz', 'date_end')
    def _compute_date_end_tz(self):
        for event in self:
            if event.date_end:
                event.date_end_located = format_datetime(
                    self.env, event.date_end, tz=event.date_tz, dt_format='medium')
            else:
                event.date_end_located = False

    @api.depends('date_begin', 'date_end')
    def _compute_is_ongoing(self):
        now = fields.Datetime.now()
        for event in self:
            event.is_ongoing = event.date_begin <= now < event.date_end

    def _search_is_ongoing(self, operator, value):
        if operator not in ['=', '!=']:
            raise ValueError(_('This operator is not supported'))
        if not isinstance(value, bool):
            raise ValueError(_('Value should be True or False (not %s)'), value)
        now = fields.Datetime.now()
        if (operator == '=' and value) or (operator == '!=' and not value):
            domain = [('date_begin', '<=', now), ('date_end', '>', now)]
        else:
            domain = ['|', ('date_begin', '>', now), ('date_end', '<=', now)]
        event_ids = self.env['event.event']._search(domain)
        return [('id', 'in', event_ids)]

    @api.depends('date_begin', 'date_end', 'date_tz')
    def _compute_field_is_one_day(self):
        for event in self:
            # Need to localize because it could begin late and finish early in
            # another timezone
            event = event._set_tz_context()
            begin_tz = fields.Datetime.context_timestamp(event, event.date_begin)
            end_tz = fields.Datetime.context_timestamp(event, event.date_end)
            event.is_one_day = (begin_tz.date() == end_tz.date())

    @api.depends('event_type_id')
    def _compute_date_tz(self):
        for event in self:
            if event.event_type_id.use_timezone and event.event_type_id.default_timezone:
                event.date_tz = event.event_type_id.default_timezone
            if not event.date_tz:
                event.date_tz = self.env.user.tz or 'UTC'

    # seats

    @api.depends('event_type_id')
    def _compute_seats_max(self):
        """ Update event configuration from its event type. Depends are set only
        on event_type_id itself, not its sub fields. Purpose is to emulate an
        onchange: if event type is changed, update event configuration. Changing
        event type content itself should not trigger this method. """
        for event in self:
            if not event.event_type_id:
                event.seats_max = event.seats_max or 0
            else:
                event.seats_max = event.event_type_id.seats_max or 0

    @api.depends('event_type_id')
    def _compute_seats_limited(self):
        """ Update event configuration from its event type. Depends are set only
        on event_type_id itself, not its sub fields. Purpose is to emulate an
        onchange: if event type is changed, update event configuration. Changing
        event type content itself should not trigger this method. """
        for event in self:
            if event.event_type_id.has_seats_limitation != event.seats_limited:
                event.seats_limited = event.event_type_id.has_seats_limitation
            if not event.seats_limited:
                event.seats_limited = False

    @api.depends('event_type_id')
    def _compute_auto_confirm(self):
        """ Update event configuration from its event type. Depends are set only
        on event_type_id itself, not its sub fields. Purpose is to emulate an
        onchange: if event type is changed, update event configuration. Changing
        event type content itself should not trigger this method. """
        for event in self:
            event.auto_confirm = event.event_type_id.auto_confirm

    @api.depends('event_type_id')
    def _compute_event_mail_ids(self):
        """ Update event configuration from its event type. Depends are set only
        on event_type_id itself, not its sub fields. Purpose is to emulate an
        onchange: if event type is changed, update event configuration. Changing
        event type content itself should not trigger this method.

        When synchronizing mails:

          * lines that are not sent and have no registrations linked are remove;
          * type lines are added;
        """
        for event in self:
            if not event.event_type_id and not event.event_mail_ids:
                event.event_mail_ids = False
                continue

            # lines to keep: those with already sent emails or registrations
            mails_toremove = event._origin.event_mail_ids.filtered(lambda mail: not mail.mail_sent and not(mail.mail_registration_ids))
            command = [(3, mail.id) for mail in mails_toremove]
            if event.event_type_id.use_mail_schedule:
                command += [
                    (0, 0, {
                        attribute_name: line[attribute_name] if not isinstance(line[attribute_name], models.BaseModel) else line[attribute_name].id
                        for attribute_name in self.env['event.type.mail']._get_event_mail_fields_whitelist()
                    }) for line in event.event_type_id.event_type_mail_ids
                ]
            if command:
                event.event_mail_ids = command

    @api.depends('event_type_id')
    def _compute_tag_ids(self):
        """ Update event configuration from its event type. Depends are set only
        on event_type_id itself, not its sub fields. Purpose is to emulate an
        onchange: if event type is changed, update event configuration. Changing
        event type content itself should not trigger this method. """
        for event in self:
            if not event.tag_ids and event.event_type_id.tag_ids:
                event.tag_ids = event.event_type_id.tag_ids

    @api.depends('event_type_id')
    def _compute_event_ticket_ids(self):
        """ Update event configuration from its event type. Depends are set only
        on event_type_id itself, not its sub fields. Purpose is to emulate an
        onchange: if event type is changed, update event configuration. Changing
        event type content itself should not trigger this method.

        When synchronizing tickets:

          * lines that have no registrations linked are remove;
          * type lines are added;

        Note that updating event_ticket_ids triggers _compute_start_sale_date
        (start_sale_date computation) so ensure result to avoid cache miss.
        """
        if self.ids or self._origin.ids:
            # lines to keep: those with already sent emails or registrations
            tickets_tokeep_ids = self.env['event.registration'].search(
                [('event_id', 'in', self.ids or self._origin.ids)]
            ).event_ticket_id.ids
        else:
            tickets_tokeep_ids = []
        for event in self:
            if not event.event_type_id and not event.event_ticket_ids:
                event.event_ticket_ids = False
                continue

            # lines to keep: those with existing registrations
            if tickets_tokeep_ids:
                tickets_toremove = event._origin.event_ticket_ids.filtered(lambda ticket: ticket.id not in tickets_tokeep_ids)
                command = [(3, ticket.id) for ticket in tickets_toremove]
            else:
                command = [(5, 0)]
            if event.event_type_id.use_ticket:
                command += [
                    (0, 0, {
                        attribute_name: line[attribute_name] if not isinstance(line[attribute_name], models.BaseModel) else line[attribute_name].id
                        for attribute_name in self.env['event.type.ticket']._get_event_ticket_fields_whitelist()
                    }) for line in event.event_type_id.event_type_ticket_ids
                ]
            event.event_ticket_ids = command

    @api.constrains('seats_max', 'seats_available', 'seats_limited')
    def _check_seats_limit(self):
        if any(event.seats_limited and event.seats_max and event.seats_available < 0 for event in self):
            raise ValidationError(_('No more available seats.'))

    @api.constrains('date_begin', 'date_end')
    def _check_closing_date(self):
        for event in self:
            if event.date_end < event.date_begin:
                raise ValidationError(_('The closing date cannot be earlier than the beginning date.'))

    @api.depends('name', 'date_begin', 'date_end')
    def name_get(self):
        result = []
        for event in self:
            date_begin = fields.Datetime.from_string(event.date_begin)
            date_end = fields.Datetime.from_string(event.date_end)
            dates = [fields.Date.to_string(fields.Datetime.context_timestamp(event, dt)) for dt in [date_begin, date_end] if dt]
            dates = sorted(set(dates))
            result.append((event.id, '%s (%s)' % (event.name, ' - '.join(dates))))
        return result

    @api.model
    def _read_group_stage_ids(self, stages, domain, order):
        return self.env['event.stage'].search([])

    @api.model
    def create(self, vals):
        # Temporary fix for ``seats_limited`` and ``date_tz`` required fields
        vals.update(self._sync_required_computed(vals))

        res = super(EventEvent, self).create(vals)
        if res.organizer_id:
            res.message_subscribe([res.organizer_id.id])
        res.flush()
        return res

    def write(self, vals):
        res = super(EventEvent, self).write(vals)
        if vals.get('organizer_id'):
            self.message_subscribe([vals['organizer_id']])
        return res

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        self.ensure_one()
        default = dict(default or {}, name=_("%s (copy)") % (self.name))
        return super(EventEvent, self).copy(default)

    def _sync_required_computed(self, values):
        # TODO: See if the change to seats_limited affects this ?
        """ Call compute fields in cache to find missing values for required fields
        (seats_limited and date_tz) in case they are not given in values """
        missing_fields = list(set(['seats_limited', 'date_tz']).difference(set(values.keys())))
        if missing_fields and values:
            cache_event = self.new(values)
            cache_event._compute_seats_limited()
            cache_event._compute_date_tz()
            return dict((fname, cache_event[fname]) for fname in missing_fields)
        else:
            return {}

    def _set_tz_context(self):
        self.ensure_one()
        return self.with_context(tz=self.date_tz or 'UTC')

    def action_set_done(self):
        """
        Action which will move the events
        into the first next (by sequence) stage defined as "Ended"
        (if they are not already in an ended stage)
        """
        first_ended_stage = self.env['event.stage'].search([('pipe_end', '=', True)], order='sequence')
        if first_ended_stage:
            self.write({'stage_id': first_ended_stage[0].id})

    def mail_attendees(self, template_id, force_send=False, filter_func=lambda self: self.state != 'cancel'):
        for event in self:
            for attendee in event.registration_ids.filtered(filter_func):
                self.env['mail.template'].browse(template_id).send_mail(attendee.id, force_send=force_send)

    def _get_ics_file(self):
        """ Returns iCalendar file for the event invitation.
            :returns a dict of .ics file content for each event
        """
        result = {}
        if not vobject:
            return result

        for event in self:
            cal = vobject.iCalendar()
            cal_event = cal.add('vevent')

            cal_event.add('created').value = fields.Datetime.now().replace(tzinfo=pytz.timezone('UTC'))
            cal_event.add('dtstart').value = fields.Datetime.from_string(event.date_begin).replace(tzinfo=pytz.timezone('UTC'))
            cal_event.add('dtend').value = fields.Datetime.from_string(event.date_end).replace(tzinfo=pytz.timezone('UTC'))
            cal_event.add('summary').value = event.name
            if event.address_id:
                cal_event.add('location').value = event.sudo().address_id.contact_address

            result[event.id] = cal.serialize().encode('utf-8')
        return result

    @api.autovacuum
    def _gc_mark_events_done(self):
        """ move every ended events in the next 'ended stage' """
        ended_events = self.env['event.event'].search([
            ('date_end', '<', fields.Datetime.now()),
            ('stage_id.pipe_end', '=', False),
        ])
        if ended_events:
            ended_events.action_set_done()

```

## File: models\event_mail.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import random
import threading

from datetime import datetime
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, tools
from odoo.tools import exception_to_unicode
from odoo.tools.translate import _

_logger = logging.getLogger(__name__)

_INTERVALS = {
    'hours': lambda interval: relativedelta(hours=interval),
    'days': lambda interval: relativedelta(days=interval),
    'weeks': lambda interval: relativedelta(days=7*interval),
    'months': lambda interval: relativedelta(months=interval),
    'now': lambda interval: relativedelta(hours=0),
}


class EventTypeMail(models.Model):
    """ Template of event.mail to attach to event.type. Those will be copied
    upon all events created in that type to ease event creation. """
    _name = 'event.type.mail'
    _description = 'Mail Scheduling on Event Category'

    event_type_id = fields.Many2one(
        'event.type', string='Event Type',
        ondelete='cascade', required=True)
    notification_type = fields.Selection([('mail', 'Mail')], string='Send', default='mail', required=True)
    interval_nbr = fields.Integer('Interval', default=1)
    interval_unit = fields.Selection([
        ('now', 'Immediately'),
        ('hours', 'Hours'), ('days', 'Days'),
        ('weeks', 'Weeks'), ('months', 'Months')],
        string='Unit', default='hours', required=True)
    interval_type = fields.Selection([
        ('after_sub', 'After each registration'),
        ('before_event', 'Before the event'),
        ('after_event', 'After the event')],
        string='Trigger', default="before_event", required=True)
    template_id = fields.Many2one(
        'mail.template', string='Email Template',
        domain=[('model', '=', 'event.registration')], ondelete='restrict',
        help='This field contains the template of the mail that will be automatically sent')

    @api.model
    def _get_event_mail_fields_whitelist(self):
        """ Whitelist of fields that are copied from event_type_mail_ids to event_mail_ids when
        changing the event_type_id field of event.event """
        return ['notification_type', 'template_id', 'interval_nbr', 'interval_unit', 'interval_type']


class EventMailScheduler(models.Model):
    """ Event automated mailing. This model replaces all existing fields and
    configuration allowing to send emails on events since Odoo 9. A cron exists
    that periodically checks for mailing to run. """
    _name = 'event.mail'
    _rec_name = 'event_id'
    _description = 'Event Automated Mailing'

    event_id = fields.Many2one('event.event', string='Event', required=True, ondelete='cascade')
    sequence = fields.Integer('Display order')
    notification_type = fields.Selection([('mail', 'Mail')], string='Send', default='mail', required=True)
    interval_nbr = fields.Integer('Interval', default=1)
    interval_unit = fields.Selection([
        ('now', 'Immediately'),
        ('hours', 'Hours'), ('days', 'Days'),
        ('weeks', 'Weeks'), ('months', 'Months')],
        string='Unit', default='hours', required=True)
    interval_type = fields.Selection([
        ('after_sub', 'After each registration'),
        ('before_event', 'Before the event'),
        ('after_event', 'After the event')],
        string='Trigger ', default="before_event", required=True)
    template_id = fields.Many2one(
        'mail.template', string='Email Template',
        domain=[('model', '=', 'event.registration')], ondelete='restrict',
        help='This field contains the template of the mail that will be automatically sent')
    scheduled_date = fields.Datetime('Scheduled Sent Mail', compute='_compute_scheduled_date', store=True)
    mail_registration_ids = fields.One2many('event.mail.registration', 'scheduler_id')
    mail_sent = fields.Boolean('Mail Sent on Event', copy=False)
    done = fields.Boolean('Sent', compute='_compute_done', store=True)

    @api.depends('mail_sent', 'interval_type', 'event_id.registration_ids', 'mail_registration_ids')
    def _compute_done(self):
        for mail in self:
            if mail.interval_type in ['before_event', 'after_event']:
                mail.done = mail.mail_sent
            else:
                mail.done = len(mail.mail_registration_ids) == len(mail.event_id.registration_ids) and all(mail.mail_sent for mail in mail.mail_registration_ids)

    @api.depends('event_id.date_begin', 'interval_type', 'interval_unit', 'interval_nbr')
    def _compute_scheduled_date(self):
        for mail in self:
            if mail.interval_type == 'after_sub':
                date, sign = mail.event_id.create_date, 1
            elif mail.interval_type == 'before_event':
                date, sign = mail.event_id.date_begin, -1
            else:
                date, sign = mail.event_id.date_end, 1

            mail.scheduled_date = date + _INTERVALS[mail.interval_unit](sign * mail.interval_nbr) if date else False

    def execute(self):
        for mail in self:
            now = fields.Datetime.now()
            if mail.interval_type == 'after_sub':
                # update registration lines
                lines = [
                    (0, 0, {'registration_id': registration.id})
                    for registration in (mail.event_id.registration_ids - mail.mapped('mail_registration_ids.registration_id'))
                ]
                if lines:
                    mail.write({'mail_registration_ids': lines})
                # execute scheduler on registrations
                mail.mail_registration_ids.execute()
            else:
                # Do not send emails if the mailing was scheduled before the event but the event is over
                if not mail.mail_sent and mail.scheduled_date <= now and mail.notification_type == 'mail' and \
                        (mail.interval_type != 'before_event' or mail.event_id.date_end > now):
                    mail.event_id.mail_attendees(mail.template_id.id)
                    mail.write({'mail_sent': True})
        return True

    @api.model
    def _warn_template_error(self, scheduler, exception):
        # We warn ~ once by hour ~ instead of every 10 min if the interval unit is more than 'hours'.
        if random.random() < 0.1666 or scheduler.interval_unit in ('now', 'hours'):
            ex_s = exception_to_unicode(exception)
            try:
                event, template = scheduler.event_id, scheduler.template_id
                emails = list(set([event.organizer_id.email, event.user_id.email, template.write_uid.email]))
                subject = _("WARNING: Event Scheduler Error for event: %s", event.name)
                body = _("""Event Scheduler for:
  - Event: %(event_name)s (%(event_id)s)
  - Scheduled: %(date)s
  - Template: %(template_name)s (%(template_id)s)

Failed with error:
  - %(error)s

You receive this email because you are:
  - the organizer of the event,
  - or the responsible of the event,
  - or the last writer of the template.
""",
                         event_name=event.name,
                         event_id=event.id,
                         date=scheduler.scheduled_date,
                         template_name=template.name,
                         template_id=template.id,
                         error=ex_s)
                email = self.env['ir.mail_server'].build_email(
                    email_from=self.env.user.email,
                    email_to=emails,
                    subject=subject, body=body,
                )
                self.env['ir.mail_server'].send_email(email)
            except Exception as e:
                _logger.error("Exception while sending traceback by email: %s.\n Original Traceback:\n%s", e, exception)
                pass

    @api.model
    def run(self, autocommit=False):
        schedulers = self.search([
            ('event_id.active', '=', True),
            ('done', '=', False),
            ('scheduled_date', '<=', datetime.strftime(fields.datetime.now(), tools.DEFAULT_SERVER_DATETIME_FORMAT))
        ])
        for scheduler in schedulers:
            try:
                with self.env.cr.savepoint():
                    # Prevent a mega prefetch of the registration ids of all the events of all the schedulers
                    self.browse(scheduler.id).execute()
            except Exception as e:
                _logger.exception(e)
                self.invalidate_cache()
                self._warn_template_error(scheduler, e)
            else:
                if autocommit and not getattr(threading.currentThread(), 'testing', False):
                    self.env.cr.commit()
        return True


class EventMailRegistration(models.Model):
    _name = 'event.mail.registration'
    _description = 'Registration Mail Scheduler'
    _rec_name = 'scheduler_id'
    _order = 'scheduled_date DESC'

    scheduler_id = fields.Many2one('event.mail', 'Mail Scheduler', required=True, ondelete='cascade')
    registration_id = fields.Many2one('event.registration', 'Attendee', required=True, ondelete='cascade')
    scheduled_date = fields.Datetime('Scheduled Time', compute='_compute_scheduled_date', store=True)
    mail_sent = fields.Boolean('Mail Sent')

    def execute(self):
        now = fields.Datetime.now()
        todo = self.filtered(lambda reg_mail:
            not reg_mail.mail_sent and \
            reg_mail.registration_id.state in ['open', 'done'] and \
            (reg_mail.scheduled_date and reg_mail.scheduled_date <= now) and \
            reg_mail.scheduler_id.notification_type == 'mail'
        )
        for reg_mail in todo:
            organizer = reg_mail.scheduler_id.event_id.organizer_id
            company = self.env.company
            author = self.env.ref('base.user_root')
            if organizer.email:
                author = organizer
            elif company.email:
                author = company.partner_id
            elif self.env.user.email:
                author = self.env.user

            email_values = {
                'author_id': author.id,
            }
            if not reg_mail.scheduler_id.template_id.email_from:
                email_values['email_from'] = author.email_formatted
            reg_mail.scheduler_id.template_id.send_mail(reg_mail.registration_id.id, email_values=email_values)
        todo.write({'mail_sent': True})

    @api.depends('registration_id', 'scheduler_id.interval_unit', 'scheduler_id.interval_type')
    def _compute_scheduled_date(self):
        for mail in self:
            if mail.registration_id:
                date_open = mail.registration_id.date_open
                date_open_datetime = date_open or fields.Datetime.now()
                mail.scheduled_date = date_open_datetime + _INTERVALS[mail.scheduler_id.interval_unit](mail.scheduler_id.interval_nbr)
            else:
                mail.scheduled_date = False

```

## File: models\event_registration.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from dateutil.relativedelta import relativedelta

from odoo import _, api, fields, models, SUPERUSER_ID
from odoo.tools import format_datetime, email_normalize, email_normalize_all
from odoo.exceptions import AccessError, ValidationError


class EventRegistration(models.Model):
    _name = 'event.registration'
    _description = 'Event Registration'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'id desc'

    # event
    event_id = fields.Many2one(
        'event.event', string='Event', required=True,
        readonly=True, states={'draft': [('readonly', False)]})
    event_ticket_id = fields.Many2one(
        'event.event.ticket', string='Event Ticket', readonly=True, ondelete='restrict',
        states={'draft': [('readonly', False)]})
    # utm informations
    utm_campaign_id = fields.Many2one('utm.campaign', 'Campaign',  index=True, ondelete='set null')
    utm_source_id = fields.Many2one('utm.source', 'Source', index=True, ondelete='set null')
    utm_medium_id = fields.Many2one('utm.medium', 'Medium', index=True, ondelete='set null')
    # attendee
    partner_id = fields.Many2one(
        'res.partner', string='Booked by',
        states={'done': [('readonly', True)]})
    name = fields.Char(
        string='Attendee Name', index=True,
        compute='_compute_name', readonly=False, store=True, tracking=10)
    email = fields.Char(string='Email', compute='_compute_email', readonly=False, store=True, tracking=11)
    phone = fields.Char(string='Phone', compute='_compute_phone', readonly=False, store=True, tracking=12)
    mobile = fields.Char(string='Mobile', compute='_compute_mobile', readonly=False, store=True, tracking=13)
    # organization
    date_open = fields.Datetime(string='Registration Date', readonly=True, default=lambda self: fields.Datetime.now())  # weird crash is directly now
    date_closed = fields.Datetime(
        string='Attended Date', compute='_compute_date_closed',
        readonly=False, store=True)
    event_begin_date = fields.Datetime(string="Event Start Date", related='event_id.date_begin', readonly=True)
    event_end_date = fields.Datetime(string="Event End Date", related='event_id.date_end', readonly=True)
    company_id = fields.Many2one(
        'res.company', string='Company', related='event_id.company_id',
        store=True, readonly=True, states={'draft': [('readonly', False)]})
    state = fields.Selection([
        ('draft', 'Unconfirmed'), ('cancel', 'Cancelled'),
        ('open', 'Confirmed'), ('done', 'Attended')],
        string='Status', default='draft', readonly=True, copy=False, tracking=True)

    @api.onchange('partner_id')
    def _onchange_partner_id(self):
        """ Keep an explicit onchange on partner_id. Rationale : if user explicitly
        changes the partner in interface, he want to update the whole customer
        information. If partner_id is updated in code (e.g. updating your personal
        information after having registered in website_event_sale) fields with a
        value should not be reset as we don't know which one is the right one.

        In other words
          * computed fields based on partner_id should only update missing
            information. Indeed automated code cannot decide which information
            is more accurate;
          * interface should allow to update all customer related information
            at once. We consider event users really want to update all fields
            related to the partner;
        """
        for registration in self:
            if registration.partner_id:
                registration.update(registration._synchronize_partner_values(registration.partner_id))

    @api.depends('partner_id')
    def _compute_name(self):
        for registration in self:
            if not registration.name and registration.partner_id:
                registration.name = registration._synchronize_partner_values(
                    registration.partner_id,
                    fnames=['name']
                ).get('name') or False

    @api.depends('partner_id')
    def _compute_email(self):
        for registration in self:
            if not registration.email and registration.partner_id:
                registration.email = registration._synchronize_partner_values(
                    registration.partner_id,
                    fnames=['email']
                ).get('email') or False

    @api.depends('partner_id')
    def _compute_phone(self):
        for registration in self:
            if not registration.phone and registration.partner_id:
                registration.phone = registration._synchronize_partner_values(
                    registration.partner_id,
                    fnames=['phone']
                ).get('phone') or False

    @api.depends('partner_id')
    def _compute_mobile(self):
        for registration in self:
            if not registration.mobile and registration.partner_id:
                registration.mobile = registration._synchronize_partner_values(
                    registration.partner_id,
                    fnames=['mobile']
                ).get('mobile') or False

    @api.depends('state')
    def _compute_date_closed(self):
        for registration in self:
            if not registration.date_closed:
                if registration.state == 'done':
                    registration.date_closed = fields.Datetime.now()
                else:
                    registration.date_closed = False

    @api.constrains('event_id', 'state')
    def _check_seats_limit(self):
        for registration in self:
            if registration.event_id.seats_limited and registration.event_id.seats_max and registration.event_id.seats_available < (1 if registration.state == 'draft' else 0):
                raise ValidationError(_('No more seats available for this event.'))

    @api.constrains('event_ticket_id', 'state')
    def _check_ticket_seats_limit(self):
        for record in self:
            if record.event_ticket_id.seats_max and record.event_ticket_id.seats_available < 0:
                raise ValidationError(_('No more available seats for this ticket'))

    @api.constrains('event_id', 'event_ticket_id')
    def _check_event_ticket(self):
        if any(registration.event_id != registration.event_ticket_id.event_id for registration in self if registration.event_ticket_id):
            raise ValidationError(_('Invalid event / ticket choice'))

    def _synchronize_partner_values(self, partner, fnames=None):
        if fnames is None:
            fnames = ['name', 'email', 'phone', 'mobile']
        if partner:
            contact_id = partner.address_get().get('contact', False)
            if contact_id:
                contact = self.env['res.partner'].browse(contact_id)
                return dict((fname, contact[fname]) for fname in fnames if contact[fname])
        return {}

    # ------------------------------------------------------------
    # CRUD
    # ------------------------------------------------------------

    @api.model_create_multi
    def create(self, vals_list):
        registrations = super(EventRegistration, self).create(vals_list)
        if registrations._check_auto_confirmation():
            registrations.sudo().action_confirm()

        return registrations

    def write(self, vals):
        ret = super(EventRegistration, self).write(vals)

        if vals.get('state') == 'open':
            # auto-trigger after_sub (on subscribe) mail schedulers, if needed
            onsubscribe_schedulers = self.mapped('event_id.event_mail_ids').filtered(lambda s: s.interval_type == 'after_sub')
            onsubscribe_schedulers.with_user(SUPERUSER_ID).execute()

        return ret

    def name_get(self):
        """ Custom name_get implementation to better differentiate registrations
        linked to a given partner but with different name (one partner buying
        several registrations)

          * name, partner_id has no name -> take name
          * partner_id has name, name void or same -> take partner name
          * both have name: partner + name
        """
        ret_list = []
        for registration in self:
            if registration.partner_id.name:
                if registration.name and registration.name != registration.partner_id.name:
                    name = '%s, %s' % (registration.partner_id.name, registration.name)
                else:
                    name = registration.partner_id.name
            else:
                name = registration.name
            ret_list.append((registration.id, name))
        return ret_list

    def _check_auto_confirmation(self):
        if any(not registration.event_id.auto_confirm or
               (not registration.event_id.seats_available and registration.event_id.seats_limited) for registration in self):
            return False
        return True

    # ------------------------------------------------------------
    # ACTIONS / BUSINESS
    # ------------------------------------------------------------

    def action_set_draft(self):
        self.write({'state': 'draft'})

    def action_confirm(self):
        self.write({'state': 'open'})

    def action_set_done(self):
        """ Close Registration """
        self.write({'state': 'done'})

    def action_cancel(self):
        self.write({'state': 'cancel'})

    def _message_get_suggested_recipients(self):
        recipients = super(EventRegistration, self)._message_get_suggested_recipients()
        public_users = self.env['res.users'].sudo()
        public_groups = self.env.ref("base.group_public", raise_if_not_found=False)
        if public_groups:
            public_users = public_groups.sudo().with_context(active_test=False).mapped("users")
        try:
            for attendee in self:
                is_public = attendee.sudo().with_context(active_test=False).partner_id.user_ids in public_users if public_users else False
                if attendee.partner_id and not is_public:
                    attendee._message_add_suggested_recipient(recipients, partner=attendee.partner_id, reason=_('Customer'))
                elif attendee.email:
                    attendee._message_add_suggested_recipient(recipients, email=attendee.email, reason=_('Customer Email'))
        except AccessError:     # no read access rights -> ignore suggested recipients
            pass
        return recipients

    def _message_get_default_recipients(self):
        # Prioritize registration email over partner_id, which may be shared when a single
        # partner booked multiple seats
        return {r.id:
            {
                'partner_ids': [],
                'email_to': ','.join(email_normalize_all(r.email)) or r.email,
                'email_cc': False,
            } for r in self
        }

    def _message_post_after_hook(self, message, msg_vals):
        if self.email and not self.partner_id:
            # we consider that posting a message with a specified recipient (not a follower, a specific one)
            # on a document without customer means that it was created through the chatter using
            # suggested recipients. This heuristic allows to avoid ugly hacks in JS.
            email_normalized = email_normalize(self.email)
            new_partner = message.partner_ids.filtered(
                lambda partner: partner.email == self.email or (email_normalized and partner.email_normalized == email_normalized)
            )
            if new_partner:
                if new_partner[0].email_normalized:
                    email_domain = ('email', 'in', [new_partner[0].email, new_partner[0].email_normalized])
                else:
                    email_domain = ('email', '=', new_partner[0].email)
                self.search([
                    ('partner_id', '=', False), email_domain, ('state', 'not in', ['cancel']),
                ]).write({'partner_id': new_partner[0].id})
        return super(EventRegistration, self)._message_post_after_hook(message, msg_vals)

    def action_send_badge_email(self):
        """ Open a window to compose an email, with the template - 'event_badge'
            message loaded by default
        """
        self.ensure_one()
        template = self.env.ref('event.event_registration_mail_template_badge')
        compose_form = self.env.ref('mail.email_compose_message_wizard_form')
        ctx = dict(
            default_model='event.registration',
            default_res_id=self.id,
            default_use_template=bool(template),
            default_template_id=template.id,
            default_composition_mode='comment',
            custom_layout="mail.mail_notification_light",
        )
        return {
            'name': _('Compose Email'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'mail.compose.message',
            'views': [(compose_form.id, 'form')],
            'view_id': compose_form.id,
            'target': 'new',
            'context': ctx,
        }

    def get_date_range_str(self):
        self.ensure_one()
        today = fields.Datetime.now()
        event_date = self.event_begin_date
        diff = (event_date.date() - today.date())
        if diff.days <= 0:
            return _('today')
        elif diff.days == 1:
            return _('tomorrow')
        elif (diff.days < 7):
            return _('in %d days') % (diff.days, )
        elif (diff.days < 14):
            return _('next week')
        elif event_date.month == (today + relativedelta(months=+1)).month:
            return _('next month')
        else:
            return _('on %(date)s', date=format_datetime(self.env, self.event_begin_date, tz=self.event_id.date_tz, dt_format='medium'))

    def _get_registration_summary(self):
        self.ensure_one()
        return {
            'id': self.id,
            'name': self.name,
            'partner_id': self.partner_id.id,
            'ticket_name': self.event_ticket_id.name or _('None'),
            'event_id': self.event_id.id,
            'event_display_name': self.event_id.display_name,
            'company_name': self.event_id.company_id and self.event_id.company_id.name or False,
        }

```

## File: models\event_stage.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models


class EventStage(models.Model):
    _name = 'event.stage'
    _description = 'Event Stage'
    _order = 'sequence, name'

    name = fields.Char(string='Stage Name', required=True, translate=True)
    description = fields.Text(string='Stage description', translate=True)
    sequence = fields.Integer('Sequence', default=1)
    fold = fields.Boolean(string='Folded in Kanban', default=False)
    pipe_end = fields.Boolean(
        string='End Stage', default=False,
        help='Events will automatically be moved into this stage when they are finished. The event moved into this stage will automatically be set as green.')
    legend_blocked = fields.Char(
        'Red Kanban Label', default=lambda s: _('Blocked'), translate=True, required=True,
        help='Override the default value displayed for the blocked state for kanban selection.')
    legend_done = fields.Char(
        'Green Kanban Label', default=lambda s: _('Ready for Next Stage'), translate=True, required=True,
        help='Override the default value displayed for the done state for kanban selection.')
    legend_normal = fields.Char(
        'Grey Kanban Label', default=lambda s: _('In Progress'), translate=True, required=True,
        help='Override the default value displayed for the normal state for kanban selection.')

```

## File: models\event_tag.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint

from odoo import api, fields, models


class EventTagCategory(models.Model):
    _name = "event.tag.category"
    _description = "Event Tag Category"
    _order = "sequence"

    name = fields.Char("Name", required=True, translate=True)
    sequence = fields.Integer('Sequence', default=0)
    tag_ids = fields.One2many('event.tag', 'category_id', string="Tags")

class EventTag(models.Model):
    _name = "event.tag"
    _description = "Event Tag"
    _order = "sequence"

    def _default_color(self):
        return randint(1, 11)

    name = fields.Char("Name", required=True, translate=True)
    sequence = fields.Integer('Sequence', default=0)
    category_id = fields.Many2one("event.tag.category", string="Category", required=True, ondelete='cascade')
    color = fields.Integer(
        string='Color Index', default=lambda self: self._default_color(),
        help='Tag color. No color means no display in kanban or front-end, to distinguish internal tags from public categorization tags.')

    def name_get(self):
        return [(tag.id, "%s: %s" % (tag.category_id.name, tag.name)) for tag in self]

```

## File: models\event_ticket.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError, UserError


class EventTemplateTicket(models.Model):
    _name = 'event.type.ticket'
    _description = 'Event Template Ticket'

    # description
    name = fields.Char(
        string='Name', default=lambda self: _('Registration'),
        required=True, translate=True)
    description = fields.Text(
        'Description', translate=True,
        help="A description of the ticket that you want to communicate to your customers.")
    event_type_id = fields.Many2one(
        'event.type', string='Event Category', ondelete='cascade', required=True)
    # seats
    seats_limited = fields.Boolean(string='Seats Limit', readonly=True, store=True,
                                   compute='_compute_seats_limited')
    seats_max = fields.Integer(
        string='Maximum Seats',
        help="Define the number of available tickets. If you have too many registrations you will "
             "not be able to sell tickets anymore. Set 0 to ignore this rule set as unlimited.")

    @api.depends('seats_max')
    def _compute_seats_limited(self):
        for ticket in self:
            ticket.seats_limited = ticket.seats_max

    @api.model
    def _get_event_ticket_fields_whitelist(self):
        """ Whitelist of fields that are copied from event_type_ticket_ids to event_ticket_ids when
        changing the event_type_id field of event.event """
        return ['name', 'description', 'seats_max']


class EventTicket(models.Model):
    """ Ticket model allowing to have differnt kind of registrations for a given
    event. Ticket are based on ticket type as they share some common fields
    and behavior. Those models come from <= v13 Odoo event.event.ticket that
    modeled both concept: tickets for event templates, and tickets for events. """
    _name = 'event.event.ticket'
    _inherit = 'event.type.ticket'
    _description = 'Event Ticket'

    @api.model
    def default_get(self, fields):
        res = super(EventTicket, self).default_get(fields)
        if 'name' in fields and (not res.get('name') or res['name'] == _('Registration')) and self.env.context.get('default_event_name'):
            res['name'] = _('Registration for %s', self.env.context['default_event_name'])
        return res

    # description
    event_type_id = fields.Many2one(ondelete='set null', required=False)
    event_id = fields.Many2one(
        'event.event', string="Event",
        ondelete='cascade', required=True)
    company_id = fields.Many2one('res.company', related='event_id.company_id')
    # sale
    start_sale_date = fields.Date(string="Registration Start")
    end_sale_date = fields.Date(string="Registration End")
    is_expired = fields.Boolean(string='Is Expired', compute='_compute_is_expired')
    sale_available = fields.Boolean(string='Is Available', compute='_compute_sale_available', compute_sudo=True)
    registration_ids = fields.One2many('event.registration', 'event_ticket_id', string='Registrations')
    # seats
    seats_reserved = fields.Integer(string='Reserved Seats', compute='_compute_seats', store=True)
    seats_available = fields.Integer(string='Available Seats', compute='_compute_seats', store=True)
    seats_unconfirmed = fields.Integer(string='Unconfirmed Seats', compute='_compute_seats', store=True)
    seats_used = fields.Integer(string='Used Seats', compute='_compute_seats', store=True)

    @api.depends('end_sale_date', 'event_id.date_tz')
    def _compute_is_expired(self):
        for ticket in self:
            ticket = ticket._set_tz_context()
            current_date = fields.Date.context_today(ticket)
            if ticket.end_sale_date:
                ticket.is_expired = ticket.end_sale_date < current_date
            else:
                ticket.is_expired = False

    @api.depends('is_expired', 'start_sale_date', 'event_id.date_tz', 'seats_available', 'seats_max')
    def _compute_sale_available(self):
        for ticket in self:
            if not ticket.is_launched() or ticket.is_expired or (ticket.seats_max and ticket.seats_available <= 0):
                ticket.sale_available = False
            else:
                ticket.sale_available = True

    @api.depends('seats_max', 'registration_ids.state')
    def _compute_seats(self):
        """ Determine reserved, available, reserved but unconfirmed and used seats. """
        # initialize fields to 0 + compute seats availability
        for ticket in self:
            ticket.seats_unconfirmed = ticket.seats_reserved = ticket.seats_used = ticket.seats_available = 0
        # aggregate registrations by ticket and by state
        results = {}
        if self.ids:
            state_field = {
                'draft': 'seats_unconfirmed',
                'open': 'seats_reserved',
                'done': 'seats_used',
            }
            query = """ SELECT event_ticket_id, state, count(event_id)
                        FROM event_registration
                        WHERE event_ticket_id IN %s AND state IN ('draft', 'open', 'done')
                        GROUP BY event_ticket_id, state
                    """
            self.env['event.registration'].flush(['event_id', 'event_ticket_id', 'state'])
            self.env.cr.execute(query, (tuple(self.ids),))
            for event_ticket_id, state, num in self.env.cr.fetchall():
                results.setdefault(event_ticket_id, {})[state_field[state]] = num

        # compute seats_available
        for ticket in self:
            ticket.update(results.get(ticket._origin.id or ticket.id, {}))
            if ticket.seats_max > 0:
                ticket.seats_available = ticket.seats_max - (ticket.seats_reserved + ticket.seats_used)

    @api.constrains('start_sale_date', 'end_sale_date')
    def _constrains_dates_coherency(self):
        for ticket in self:
            if ticket.start_sale_date and ticket.end_sale_date and ticket.start_sale_date > ticket.end_sale_date:
                raise UserError(_('The stop date cannot be earlier than the start date.'))

    @api.constrains('seats_available', 'seats_max')
    def _constrains_seats_available(self):
        if any(record.seats_max and record.seats_available < 0 for record in self):
            raise ValidationError(_('No more available seats for this ticket.'))

    def _get_ticket_multiline_description(self):
        """ Compute a multiline description of this ticket. It is used when ticket
        description are necessary without having to encode it manually, like sales
        information. """
        return '%s\n%s' % (self.display_name, self.event_id.display_name)

    def _set_tz_context(self):
        self.ensure_one()
        return self.with_context(tz=self.event_id.date_tz or 'UTC')

    def is_launched(self):
        # TDE FIXME: in master, make a computed field, easier to use
        self.ensure_one()
        if self.start_sale_date:
            ticket = self._set_tz_context()
            current_date = fields.Date.context_today(ticket)
            return ticket.start_sale_date <= current_date
        else:
            return True

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models

class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    module_event_sale = fields.Boolean("Tickets")
    module_website_event_meet = fields.Boolean("Discussion Rooms")
    module_website_event_track = fields.Boolean("Tracks and Agenda")
    module_website_event_track_live = fields.Boolean("Live Mode")
    module_website_event_track_quiz = fields.Boolean("Quiz on Tracks")
    module_website_event_track_exhibitor = fields.Boolean("Advanced Sponsors")
    module_website_event_questions = fields.Boolean("Registration Survey")
    module_event_barcode = fields.Boolean("Barcode")
    module_website_event_sale = fields.Boolean("Online Ticketing")

    @api.onchange('module_website_event_track')
    def _onchange_module_website_event_track(self):
        """ Reset sub-modules, otherwise you may have track to False but still
        have track_live or track_quiz to True, meaning track will come back due
        to dependencies of modules. """
        for config in self:
            if not config.module_website_event_track:
                config.module_website_event_track_live = False
                config.module_website_event_track_quiz = False
                config.module_website_event_track_exhibitor = False

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    event_count = fields.Integer(
        '# Events', compute='_compute_event_count', groups='event.group_event_user',
        help='Number of events the partner has participated.')

    def _compute_event_count(self):
        self.event_count = 0
        if not self.user_has_groups('event.group_event_user'):
            return
        for partner in self:
            partner.event_count = self.env['event.event'].search_count([('registration_ids.partner_id', 'child_of', partner.ids)])

    def action_event_view(self):
        action = self.env["ir.actions.actions"]._for_xml_id("event.action_event_view")
        action['context'] = {}
        action['domain'] = [('registration_ids.partner_id', 'child_of', self.ids)]
        return action

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_event
from . import event_mail
from . import event_registration
from . import event_stage
from . import event_tag
from . import event_ticket
from . import res_config_settings
from . import res_partner

```

## File: report\event_event_reports.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="paperformat_euro_lowmargin" model="report.paperformat">
        <field name="name">European A4 low margin</field>
        <field name="default" eval="True"/>
        <field name="format">A4</field>
        <field name="page_height">0</field>
        <field name="page_width">0</field>
        <field name="orientation">Portrait</field>
        <field name="margin_top">5</field>
        <field name="margin_bottom">5</field>
        <field name="margin_left">5</field>
        <field name="margin_right">5</field>
        <field name="header_line" eval="False"/>
        <field name="header_spacing">0</field>
        <field name="dpi">80</field>
    </record>

    <record id="report_event_registration_badge" model="ir.actions.report">
        <field name="name">Registration Badge</field>
        <field name="model">event.registration</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">event.event_registration_report_template_badge</field>
        <field name="report_file">event.event_registration_report_template_badge</field>
        <field name="print_report_name">'Registration Event - %s' % (object.name or 'Attendee').replace('/','')</field>
        <field name="paperformat_id" ref="event.paperformat_euro_lowmargin"/>
        <field name="binding_model_id" ref="model_event_registration"/>
        <field name="binding_type">report</field>
    </record>

     <record id="report_event_event_badge" model="ir.actions.report">
         <field name="name">Event Badge</field>
         <field name="model">event.event</field>
         <field name="report_type">qweb-html</field>
         <field name="report_name">event.event_event_report_template_badge</field>
         <field name="report_file">event.event_event_report_template_badge</field>
         <field name="paperformat_id" ref="event.paperformat_euro_lowmargin"/>
     </record>

</odoo>

```

## File: report\event_event_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- template use to render a Registration Badge -->
    <template id="event_registration_report_template_badge">
        <t t-call="web.basic_layout">
            <t t-foreach="docs" t-as="o">
                <div class="page">
                    <div class="row">
                        <!-- Front -->
                        <div class="col-6 text-center" style="padding-left:7mm; border-left:1px dashed black; height: 149mm; -webkit-transform:rotate(180deg); padding-top: 7mm">
                            <div>
                                <div class="col-12">
                                    <t t-if="o.event_id.event_logo">
                                        <div t-field="o.event_id.event_logo"/>
                                    </t>
                                    <span t-if="not o.event_id.event_logo and o.event_id.organizer_id.company_id.logo_web and o.event_id.organizer_id.is_company">
                                        <img t-att-src="image_data_uri(o.event_id.organizer_id.company_id.logo_web)" style="max-height:1cm; max-width:4cm;" alt="Logo"/>
                                    </span>
                                    <h5 t-field="o.event_id.name"/>
                                    <h5>( <i class="fa fa-clock-o" aria-label="Period" role="img" title="Period"></i> <span itemprop="startDate" t-field="o.event_id.with_context(tz=o.event_id.date_tz).date_begin" t-options='{"hide_seconds": True}'> </span> <i>to</i> <span itemprop="endDate" t-field="o.event_id.with_context(tz=o.event_id.date_tz).date_end" t-options='{"hide_seconds": True}'> </span> )</h5>
                                </div>
                                <div>
                                    <div class="col-12 text-center" id="o_event_name">
                                        <small>
                                            <h3 t-field="o.name"/>
                                        </small>
                                    </div>
                                </div>
                                <div>
                                    <div t-field="o.event_id.badge_front"></div>
                                </div>
                            </div>
                        </div>
                        <!-- Back -->
                        <div class="col-6" style="padding-right:7mm; height: 149mm; -webkit-transform:rotate(180deg); padding-top: 2mm;">
                            <span t-field="o.event_id.badge_back"/>
                        </div>
                    </div>
                    <div class="row">
                        <!-- Inner left -->
                        <div class="col-6 text-center" style="padding-right:7mm; border-top:1px dashed black; height: 148mm;">
                            <span t-field="o.event_id.badge_innerleft"/>
                        </div>
                        <!-- Inner right -->
                        <div class="col-6 text-center" style="border-left:1px dashed black; border-top:1px dashed black; height: 148mm; text-center">
                            <span t-field="o.event_id.badge_innerright"/>
                        </div>
                    </div>
                </div>
            </t>
        </t>
    </template>


    <!-- template use to edit Event Badget (allow to set the HTML badge fields) -->
    <template id="event_event_report_template_badge">
        <t t-call="web.basic_layout">
            <div class="page">
                <t t-foreach="docs" t-as="event">
                    <div class="row">
                        <!-- Front -->
                        <div class="col-6 text-center" style="padding-left:7mm; border-right:1px dashed black; height: 149mm; padding-top: 7mm">
                            <div class="row" t-ignore="true">
                                <div class="col-12">
                                    <span t-if="event.organizer_id.is_company and event.organizer_id.company_id.logo_web">
                                        <div t-field="event.event_logo">
                                            <img t-att-src="image_data_uri(event.organizer_id.company_id.logo_web)" style="max-height:1cm; max-width:4cm;" alt="Logo"/>
                                        </div>
                                    </span>
                                    <h4 t-field="event.name"/>
                                    <h5>( <i class="fa fa-clock-o" aria-label="Period" role="img" title="Period"></i> <span itemprop="startDate" t-field="event.date_begin" t-options='{"hide_seconds": True}'> </span> <i>to</i> <span itemprop="endDate" t-field="event.date_end" t-options='{"hide_seconds": True}'> </span> )</h5>
                                </div>
                            </div>
                            <div class="row" t-ignore="true">
                                <div class="col-12 text-center" id="o_event_attendee_name">
                                    <small><h3>Attendee Name</h3></small>
                                </div>
                            </div>
                            <div class="row">
                                <div t-field="event.badge_front"></div>
                            </div>
                        </div>
                        <!-- Back -->
                        <div class="col-6" style="padding-right:7mm; height: 149mm; padding-top: 2mm;">
                            <div t-field="event.badge_back"/>
                        </div>
                    </div>
                    <div class="row">
                        <!-- Inner left -->
                        <div class="col-6 text-center" style="padding-right:7mm; border-top:1px dashed black; height: 148mm;">
                            <div t-field="event.badge_innerleft"/>
                        </div>
                        <!-- Inner right -->
                        <div class="col-6 text-center" style="border-left:1px dashed black; border-top:1px dashed black; height: 148mm; text-center">
                            <div t-field="event.badge_innerright"/>
                        </div>
                    </div>
                </t>
            </div>
        </t>
    </template>

</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: security\event_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <record model="ir.module.category" id="base.module_category_marketing_events">
            <field name="description">Helps you manage your Events.</field>
            <field name="sequence">18</field>
        </record>

        <record id="group_event_user" model="res.groups">
            <field name="name">User</field>
            <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
            <field name="category_id" ref="base.module_category_marketing_events"/>
        </record>

        <record id="group_event_manager" model="res.groups">
            <field name="name">Administrator</field>
            <field name="category_id" ref="base.module_category_marketing_events"/>
            <field name="implied_ids" eval="[(4, ref('group_event_user'))]"/>
            <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        </record>
    </data>

    <data noupdate="1">
        <record id="base.default_user" model="res.users">
            <field name="groups_id" eval="[(4,ref('event.group_event_manager'))]"/>
        </record>

        <!-- Multi - Company Rules -->
        <record model="ir.rule" id="event_event_company_rule">
            <field name="name">Event: multi-company</field>
            <field name="model_id" ref="model_event_event"/>
            <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
        </record>
        <record model="ir.rule" id="event_registration_company_rule">
            <field name="name">Event/Registration: multi-company</field>
            <field name="model_id" ref="model_event_registration"/>
            <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
        </record>
        <record id="ir_rule_event_event_ticket_company" model="ir.rule">
            <field name="name">Event/Ticket: multi-company</field>
            <field name="model_id" ref="model_event_event_ticket"/>
            <field name="domain_force">[('event_id.company_id', 'in', company_ids + [False])]</field>
        </record>

    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_event_type,event.type,model_event_type,event.group_event_user,1,0,0,0
access_event_type_manager,event.type manager,model_event_type,event.group_event_manager,1,1,1,1
access_event_type_ticket,event.type.ticket.user,model_event_type_ticket,event.group_event_user,1,0,0,0
access_event_type_ticket_manager,event.type.ticket.manager,model_event_type_ticket,event.group_event_manager,1,1,1,1
access_event_event_portal,event.event.portal,model_event_event,,1,0,0,0
access_event_event_user,event.event.user,model_event_event,event.group_event_user,1,0,0,0
access_event_event_manager,event.event.manager,model_event_event,event.group_event_manager,1,1,1,1
access_event_event_ticket_user,event.event.ticket.user,model_event_event_ticket,event.group_event_user,1,0,0,0
access_event_event_ticket_manager,event.event.ticket.manager,model_event_event_ticket,event.group_event_manager,1,1,1,1
access_event_registration,event.registration,model_event_registration,event.group_event_user,1,1,1,1
access_event_registration_all,event.registration,model_event_registration,,0,0,0,0
access_event_mail,event.mail,model_event_mail,event.group_event_user,1,0,0,0
access_event_mail_manager,event.mail manager,model_event_mail,event.group_event_manager,1,1,1,1
access_event_mail_registration,event.mail.registration,model_event_mail_registration,event.group_event_user,1,0,0,0
access_event_mail_registration_manager,event.mail.registration.manager,model_event_mail_registration,event.group_event_manager,1,1,1,1
access_event_type_mail_event_user,event.type.mail.event.user,model_event_type_mail,event.group_event_user,1,0,0,0
access_event_type_mail_event_manager,event.type.mail.event.manager,model_event_type_mail,event.group_event_manager,1,1,1,1
access_event_stage_user,event.stage.user,model_event_stage,event.group_event_user,1,1,1,1
access_event_stage_manager,event.stage.manager,model_event_stage,event.group_event_manager,1,1,1,1
access_event_category,event.tag.category,model_event_tag_category,event.group_event_user,1,0,0,0
access_event_category_manager,event.tag.category manager,model_event_tag_category,event.group_event_manager,1,1,1,1
access_event_tag,event.tag,model_event_tag,event.group_event_user,1,1,0,0
access_event_tag_manager,event.tag manager,model_event_tag,event.group_event_manager,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70">
    <defs>
        <path id="icon-a" d="M4,5.35309892e-14 C36.4160122,9.87060235e-15 58.0836068,-3.97961823e-14 65,5.07020818e-14 C69,6.733808e-14 70,1 70,5 C70,43.0488877 70,62.4235458 70,65 C70,69 69,70 65,70 C61,70 9,70 4,70 C1,70 7.10542736e-15,69 7.10542736e-15,65 C7.25721566e-15,62.4676575 3.83358709e-14,41.8005206 3.60818146e-14,5 C-1.13686838e-13,1 1,5.75716207e-14 4,5.35309892e-14 Z"/>
        <linearGradient id="icon-c" x1="100%" x2="0%" y1="0%" y2="100%">
            <stop offset="0%" stop-color="#269396"/>
            <stop offset="100%" stop-color="#218689"/>
        </linearGradient>
        <path id="icon-d" d="M24.4715571,29.3561696 L47.6197052,29.3561696 L47.6197052,43.4186696 L24.4715571,43.4186696 L24.4715571,29.3561696 Z M53.4067423,36.3874196 C53.4067423,38.32907 54.9612851,39.9030446 56.8789645,39.9030446 L56.8789645,46.9342946 C56.8789645,48.875945 55.3244217,50.4499196 53.4067423,50.4499196 L18.6845201,50.4499196 C16.7668407,50.4499196 15.2122978,48.875945 15.2122978,46.9342946 L15.2122978,39.9030446 C17.1299772,39.9030446 18.6845201,38.32907 18.6845201,36.3874196 C18.6845201,34.4457692 17.1299772,32.8717946 15.2122978,32.8717946 L15.2122978,25.8405446 C15.2122978,23.8988942 16.7668407,22.3249196 18.6845201,22.3249196 L53.4067423,22.3249196 C55.3244217,22.3249196 56.8789645,23.8988942 56.8789645,25.8405446 L56.8789645,32.8717946 C54.9612851,32.8717946 53.4067423,34.4457692 53.4067423,36.3874196 Z M49.9345201,28.7702321 C49.9345201,27.7994069 49.1572486,27.0124196 48.1984089,27.0124196 L23.8928534,27.0124196 C22.9340137,27.0124196 22.1567423,27.7994069 22.1567423,28.7702321 L22.1567423,44.0046071 C22.1567423,44.9754323 22.9340137,45.7624196 23.8928534,45.7624196 L48.1984089,45.7624196 C49.1572486,45.7624196 49.9345201,44.9754323 49.9345201,44.0046071 L49.9345201,28.7702321 Z"/>
        <path id="icon-e" d="M24.4715571,27.3561696 L47.6197052,27.3561696 L47.6197052,41.4186696 L24.4715571,41.4186696 L24.4715571,27.3561696 Z M53.4067423,34.3874196 C53.4067423,36.32907 54.9612851,37.9030446 56.8789645,37.9030446 L56.8789645,44.9342946 C56.8789645,46.875945 55.3244217,48.4499196 53.4067423,48.4499196 L18.6845201,48.4499196 C16.7668407,48.4499196 15.2122978,46.875945 15.2122978,44.9342946 L15.2122978,37.9030446 C17.1299772,37.9030446 18.6845201,36.32907 18.6845201,34.3874196 C18.6845201,32.4457692 17.1299772,30.8717946 15.2122978,30.8717946 L15.2122978,23.8405446 C15.2122978,21.8988942 16.7668407,20.3249196 18.6845201,20.3249196 L53.4067423,20.3249196 C55.3244217,20.3249196 56.8789645,21.8988942 56.8789645,23.8405446 L56.8789645,30.8717946 C54.9612851,30.8717946 53.4067423,32.4457692 53.4067423,34.3874196 Z M49.9345201,26.7702321 C49.9345201,25.7994069 49.1572486,25.0124196 48.1984089,25.0124196 L23.8928534,25.0124196 C22.9340137,25.0124196 22.1567423,25.7994069 22.1567423,26.7702321 L22.1567423,42.0046071 C22.1567423,42.9754323 22.9340137,43.7624196 23.8928534,43.7624196 L48.1984089,43.7624196 C49.1572486,43.7624196 49.9345201,42.9754323 49.9345201,42.0046071 L49.9345201,26.7702321 Z"/>
    </defs>
    <g fill="none" fill-rule="evenodd">
        <mask id="icon-b" fill="#fff">
            <use xlink:href="#icon-a"/>
        </mask>
        <g mask="url(#icon-b)">
            <rect width="70" height="70" fill="url(#icon-c)"/>
            <path fill="#FFF" fill-opacity=".383" d="M4,1.8 L65,1.8 C67.6666667,1.8 69.3333333,1.13333333 70,-0.2 C70,2.46666667 70,3.46666667 70,2.8 L1.10547097e-14,2.8 C-1.65952376e-14,3.46666667 -2.9161925e-14,2.46666667 -2.66453526e-14,-0.2 C0.666666667,1.13333333 2,1.8 4,1.8 Z" transform="matrix(1 0 0 -1 0 2.8)"/>
            <path fill="#393939" d="M35,58 L4,58 C2,58 -7.10542736e-15,57.8520408 0,53.8571429 L2.0734159e-16,30.9596526 L29,0 L37,8.28571429 L58,29 L35,58 Z" opacity=".324" transform="translate(0 12)"/>
            <path fill="#000" fill-opacity=".383" d="M4,4 L65,4 C67.6666667,4 69.3333333,3 70,1 C70,3.66666667 70,5 70,5 L1.77635684e-15,5 C1.77635684e-15,5 1.77635684e-15,3.66666667 1.77635684e-15,1 C0.666666667,3 2,4 4,4 Z" transform="translate(0 65)"/>
            <use fill="#000" fill-rule="nonzero" opacity=".3" transform="rotate(45 36.046 36.387)" xlink:href="#icon-d"/>
            <use fill="#FFF" fill-rule="nonzero" transform="rotate(45 36.046 34.387)" xlink:href="#icon-e"/>
        </g>
    </g>
</svg>

```

## File: static\src\js\tours\event_tour.js

```javascript
odoo.define('event.event_steps', function (require) {
"use strict";

var core = require('web.core');

var EventAdditionalTourSteps = core.Class.extend({

    _get_website_event_steps: function () {
        return [false];
    },

});

return EventAdditionalTourSteps;

});

odoo.define('event.event_tour', function (require) {
"use strict";

var core = require('web.core');
var _t = core._t;

var tour = require('web_tour.tour');
var EventAdditionalTourSteps = require('event.event_steps');

tour.register('event_tour', {
    url: '/web',
    rainbowManMessage: _t("Great! Now all you have to do is wait for your attendees to show up!"),
    sequence: 210,
}, [tour.stepUtils.showAppsMenuItem(), {
    trigger: '.o_app[data-menu-xmlid="event.event_main_menu"]',
    content: _t("Ready to <b>organize events</b> in a few minutes? Let's get started!"),
    position: 'bottom',
    edition: 'enterprise',
}, {
    trigger: '.o_app[data-menu-xmlid="event.event_main_menu"]',
    content: _t("Ready to <b>organize events</b> in a few minutes? Let's get started!"),
    edition: 'community',
}, {
    trigger: '.o-kanban-button-new',
    extra_trigger: '.o_event_kanban_view',
    content: _t("Let's create your first <b>event</b>."),
    position: 'bottom',
    width: 175,
}, {
    trigger: '.o_event_form_view input[name="name"]',
    content: _t("This is the <b>name</b> your guests will see when registering."),
    run: 'text Odoo Experience 2020',
}, {
    trigger: '.o_event_form_view input[name="date_end"]',
    content: _t("When will your event take place? <b>Select</b> the start and end dates <b>and click Apply</b>."),
    run: function () {
        $('input[name="date_begin"]').val('09/30/2020 08:00:00').change();
        $('input[name="date_end"]').val('10/02/2020 23:00:00').change();
    },
}, {
    trigger: '.o_event_form_view div[name="event_ticket_ids"] .o_field_x2many_list_row_add a',
    content: _t("Ticket types allow you to distinguish your attendees. Let's <b>create</b> a new one."),
}, ...new EventAdditionalTourSteps()._get_website_event_steps(), {
    trigger: '.o_event_form_view div[name="stage_id"]',
    extra_trigger: 'div.o_form_buttons_view:not(.o_hidden)',
    content: _t("Now that your event is ready, click here to move it to another stage."),
    position: 'bottom',
}, {
    trigger: 'ol.breadcrumb li.breadcrumb-item:first',
    extra_trigger: '.o_event_form_view div[name="stage_id"]',
    content: _t("Use the <b>breadcrumbs</b> to go back to your kanban overview."),
    position: 'bottom',
    run: 'click',
}, {
    trigger: '.o_event_kanban_view div.o_quick_create_folded',
    content: _t("This pipeline can be customized on the fly to fit your organizational needs. For example, let's create a new stage."),
    position: 'bottom',
    run: function (actions) {
        actions.click();
        $('div.o_kanban_header input[type="text"]').val('New Stage');
    },
}, {
    trigger: '.o_event_kanban_view button.o_kanban_add',
    content: _t("Click <b>add</b> to create a new stage."),
    position: 'bottom',
    width: 200,
    run: 'click',
}].filter(Boolean));

});

```

## File: views\event_menu_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>

    <!-- MAIN MENU -->
    <menuitem name="Events"
        id="event_main_menu"
        sequence="65"
        groups="event.group_event_user"
        web_icon="event,static/description/icon.png"/>

    <!-- HEADER: EVENTS -->
    <menuitem name="Events"
        id="menu_event_event"
        sequence="1"
        parent="event.event_main_menu"
        groups="event.group_event_user"/>

    <!-- HEADER: REPORTING -->
    <menuitem name="Reporting"
        id="menu_reporting_events"
        sequence="50"
        parent="event_main_menu"
        groups="event.group_event_manager"/>

    <!-- HEADER: CONFIGURATION -->
    <menuitem name="Configuration"
        id="menu_event_configuration"
        sequence="99"
        parent="event_main_menu"
        groups="event.group_event_user"/>
    <menuitem name="Event Templates"
        id="menu_event_type"
        sequence="1"
        parent="menu_event_configuration"/>
    <menuitem name="Event Stages"
        id="event_stage_menu"
        sequence="2"
        parent="menu_event_configuration"/>
    <menuitem name="Mail Schedulers"
        id="menu_event_mail_schedulers"
        sequence="10"
        parent="menu_event_configuration"
        groups="base.group_no_one"/>
    <menuitem name="Event Tags Categories"
        id="menu_event_category"
        sequence="20"
        parent="menu_event_configuration"/>

</data></odoo>

```

## File: views\event_stage_views.xml

```xml
<?xml version="1.0"?>
<odoo>
<data>
    <record id="event_stage_view_form" model="ir.ui.view">
        <field name="name">event.stage.view.form</field>
        <field name="model">event.stage</field>
        <field name="arch" type="xml">
            <form string="Events Stage">
                <sheet>
                    <group>
                        <group>
                            <field name="name"/>
                            <field name="pipe_end"/>
                        </group>
                        <group>
                            <field name="fold"/>
                            <field name="sequence"/>
                        </group>
                    </group>
                    <group string="Stage Description and Tooltips">
                        <p class="text-muted" colspan="2">
                            You can define here labels that will be displayed for the state instead
                            of the default labels in the kanban view.
                        </p>
                        <label for="legend_normal" string=" " class="o_status " title="Task in progress. Click to block or set as done." aria-label="Task in progress. Click to block or set as done." role="img"/>
                        <field name="legend_normal" nolabel="1"/>
                        <label for="legend_blocked" string=" " class="o_status o_status_red" title="Task is blocked. Click to unblock or set as done." aria-label="Task is blocked. Click to unblock or set as done." role="img"/>
                        <field name="legend_blocked" nolabel="1"/>
                        <label for="legend_done" string=" " class="o_status o_status_green" title="This step is done. Click to block or set in progress." aria-label="This step is done. Click to block or set in progress." role="img"/>
                        <field name="legend_done" nolabel="1"/>

                        <p class="text-muted" colspan="2">
                            You can also add a description to help your coworkers understand the meaning and purpose of the stage.
                        </p>
                        <field name="description" placeholder="Add a description..." nolabel="1" colspan="2"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="event_stage_view_tree" model="ir.ui.view">
        <field name="name">event.stage.view.tree</field>
        <field name="model">event.stage</field>
        <field name="arch" type="xml">
            <tree string="Events Stage">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
            </tree>
        </field>
    </record>

    <record id="event_stage_action" model="ir.actions.act_window">
        <field name="name">Event Stages</field>
        <field name="res_model">event.stage</field>
        <field name="view_mode">tree,form</field>
    </record>

    <record id="event_stage_menu" model="ir.ui.menu">
        <field name="action" ref="event.event_stage_action"/>
    </record>

</data>
</odoo>

```

## File: views\event_tag_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <!-- EVENT.TAG.CATEGORY VIEWS -->
        <record id="event_tag_category_view_tree" model="ir.ui.view">
            <field name="name">event.tag.category.view.tree</field>
            <field name="model">event.tag.category</field>
            <field name="arch" type="xml">
                <tree string="Event Category">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                </tree>
            </field>
        </record>

        <record id="event_tag_category_view_form" model="ir.ui.view">
            <field name="name">event.tag.category.view.form</field>
            <field name="model">event.tag.category</field>
            <field name="arch" type="xml">
                <form string="Event Category">
                    <sheet>
                        <div class="oe_title">
                            <h1><field nolabel="1" name="name"/></h1>
                        </div>
                        <group>
                            <field name="tag_ids" context="{'default_category_id': active_id}">
                                <tree string="Tags" editable="bottom">
                                    <field name="sequence" widget="handle"/>
                                    <field name="name"/>
                                    <field name="color" widget="color_picker"/>
                                </tree>
                            </field>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="event_tag_category_action_tree" model="ir.actions.act_window" >
            <field name="name">Event Tags Categories</field>
            <field name="res_model">event.tag.category</field>
            <field name="view_mode">tree,form</field>
        </record>

        <!-- EVENT.TAG VIEWS -->
        <record id="event_tag_view_tree" model="ir.ui.view">
            <field name="name">event.tag.view.tree</field>
            <field name="model">event.tag</field>
            <field name="arch" type="xml">
                <tree string="Event Tags Categories">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                    <field name="category_id"/>
                    <field name="color" widget="color_picker"/>
                </tree>
            </field>
        </record>

        <record id="event_tag_view_form" model="ir.ui.view">
            <field name="name">event.tag.view.form</field>
            <field name="model">event.tag</field>
            <field name="arch" type="xml">
                <form string="Event Category Tag">
                    <sheet>
                        <group>
                            <field name="name"/>
                            <field name="category_id" widget="many2one"/>
                            <field name="color" widget="color_picker"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="menu_event_category" model="ir.ui.menu">
            <field name="action" ref="event.event_tag_category_action_tree"/>
        </record>

    </data>
</odoo>

```

## File: views\event_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="assets_backend" name="event assets" inherit_id="web.assets_backend">
            <xpath expr="." position="inside">
                <link rel="stylesheet" type="text/scss" href="/event/static/src/scss/event.scss"/>
            </xpath>
        </template>

        <template id="assets_common" name="Event Assets Common" inherit_id="web.assets_common">
            <xpath expr="//script[last()]" position="after">
                <script type="text/javascript" src="/event/static/src/js/tours/event_tour.js"></script>
            </xpath>
        </template>
    </data>
    <template id="event_default_descripton" name="Event default description">
        <section class="s_text_block">
            <h5>Join us for this 24 hours Event</h5>
            <p>Every year we invite our community, partners and end-users to come and meet us! It's the ideal event to get together and present new features, roadmap of future versions, achievements of the software, workshops, training sessions, etc....
            This event is also an opportunity to showcase our partners' case studies, methodology or developments. Be there and see directly from the source the features of the new version!</p>
        </section>
    </template>
</odoo>

```

## File: views\event_ticket_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>

	<!-- EVENT.TYPE.TICKET -->
	<record id="event_type_ticket_view_tree_from_type" model="ir.ui.view">
        <field name="name">event.type.ticket.view.tree.from.type</field>
        <field name="model">event.type.ticket</field>
        <field name="priority" eval="20"/>
        <field name="arch" type="xml">
            <tree string="Event Template Tickets" editable="bottom">
                <field name="name"/>
                <field name="description"/>
                <field name="seats_max"/>
                <field name="seats_limited"/>
            </tree>
        </field>
	</record>

    <record id="event_type_ticket_view_form_from_type" model="ir.ui.view">
        <field name="name">event.type.ticket.view.form.from.type</field>
        <field name="model">event.type.ticket</field>
        <field name="priority" eval="20"/>
        <field name="arch" type="xml">
        <form string="Event Template Ticket">
            <sheet>
                <group>
                    <field name="name"/>
                    <field name="description"/>
                    <field name="seats_limited"/>
                    <field name="seats_max"/>
                </group>
            </sheet>
        </form>
        </field>
    </record>

    <record id="event_type_ticket_view_tree" model="ir.ui.view">
        <field name="name">event.type.ticket.view.tree</field>
        <field name="model">event.type.ticket</field>
        <field name="inherit_id" ref="event_type_ticket_view_tree_from_type"/>
        <field name="mode">primary</field>
        <field name="priority" eval="10"/>
        <field name="arch" type="xml">
            <xpath expr="//tree" position="attributes">
                <attribute name="editable"></attribute>
            </xpath>
            <xpath expr="//field[@name='name']" position="after">
                <field name="event_type_id"/>
            </xpath>
        </field>
    </record>

    <record id="event_type_ticket_view_form" model="ir.ui.view">
        <field name="name">event.type.ticket.view.form</field>
        <field name="model">event.type.ticket</field>
        <field name="inherit_id" ref="event_type_ticket_view_form_from_type"/>
        <field name="mode">primary</field>
        <field name="priority" eval="10"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='name']" position="after">
                <field name="event_type_id"/>
            </xpath>
        </field>
    </record>

    <!-- EVENT.TICKET -->
    <record id="event_event_ticket_view_tree_from_event" model="ir.ui.view">
        <field name="name">event.event.ticket.view.tree.from.event</field>
        <field name="model">event.event.ticket</field>
        <field name="priority" eval="20"/>
        <field name="arch" type="xml">
            <tree string="Tickets" editable="bottom">
                <field name="name"/>
                <field name="description" optional="hide"/>
                <field name="start_sale_date" optional="show"/>
                <field name="end_sale_date" optional="show"/>
                <field name="seats_max" sum="Total" width="105px" string="Maximum"/>
                <field name="seats_reserved" sum="Total" width="105px" string="Confirmed"/>
                <field name="seats_unconfirmed" sum="Total" width="110px" string="Unconfirmed"/>
            </tree>
        </field>
    </record>

    <record id="event_event_ticket_view_form_from_event" model="ir.ui.view">
        <field name="name">event.event.ticket.view.form.from.event</field>
        <field name="model">event.event.ticket</field>
        <field name="priority" eval="20"/>
        <field name="arch" type="xml">
            <form string="Ticket">
                <sheet>
                    <group>
                        <group>
                            <field name="name"/>
                            <field name="description"/>
                            <field name="start_sale_date"/>
                            <field name="end_sale_date"/>
                        </group><group>
                            <field name="seats_max"/>
                            <field name="seats_reserved"/>
                            <field name="seats_unconfirmed"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="event_event_ticket_view_kanban_from_event" model="ir.ui.view">
        <field name="name">event.event.ticket.view.kanban.from.event</field>
        <field name="model">event.event.ticket</field>
        <field name="priority" eval="20"/>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile">
                <field name="name"/>
                <field name="seats_max"/>
                <field name="seats_reserved"/>
                <field name="seats_unconfirmed"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                            <div class="row">
                                <div class="col-8">
                                    <strong><t t-esc="record.name.value"/></strong>
                                </div>
                            </div>
                            <div><i>
                            <t t-esc="record.seats_reserved.value"/> reserved + <t t-esc="record.seats_reserved.value"/> unconfirmed
                            </i></div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="event_event_ticket_view_tree" model="ir.ui.view">
        <field name="name">event.event.ticket.view.tree</field>
        <field name="model">event.event.ticket</field>
        <field name="inherit_id" ref="event_event_ticket_view_tree_from_event"/>
        <field name="mode">primary</field>
        <field name="priority" eval="10"/>
        <field name="arch" type="xml">
            <xpath expr="//tree" position="attributes">
                <attribute name="editable"></attribute>
            </xpath>
            <field name="name" position="after">
                <field name="event_id"/>
            </field>
        </field>
    </record>

    <record id="event_event_ticket_form_view" model="ir.ui.view">
        <field name="name">event.event.ticket.view.form</field>
        <field name="model">event.event.ticket</field>
        <field name="arch" type="xml">
            <form string="Event's Ticket">
                <sheet>
                    <div class="oe_title">
                        <label for="name" class="oe_edit_only"/>
                        <h1><field name="name" placeholder="Event Name"/></h1>
                    </div>
                    <group>
                        <group>
                            <field name="event_id"/>
                            <field name="seats_limited"/>
                            <field name="seats_available"/>
                            <field name="start_sale_date"/>
                            <field name="end_sale_date"/>
                        </group>
                        <group>
                            <field name="seats_max"/>
                            <field name="seats_reserved"/>
                            <field name="seats_unconfirmed"/>
                            <field name="seats_used"/>
                            <field name="is_expired"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
</data></odoo>
```

## File: views\event_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>

        <!-- EVENT.TYPE VIEWS -->
        <record model="ir.ui.view" id="view_event_type_form">
            <field name="name">event.type.form</field>
            <field name="model">event.type</field>
            <field name="arch" type="xml">
                <form string="Event Category">
                   <sheet>
                        <div class="oe_title">
                            <label for="name" class="oe_edit_only"/>
                            <h1><field name="name"/></h1>
                        </div>
                        <h2>Location</h2>
                        <div class="row mt16 o_settings_container" name="event_type_location">
                            <div class="col-12 col-lg-6 o_setting_box">
                                <div class="o_setting_left_pane">
                                    <field name="use_timezone"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="use_timezone"/>
                                    <div class="row">
                                        <div class="col-lg-8 mt16" attrs="{'invisible': [('use_timezone', '=', False)]}">
                                            <label for="default_timezone"/>
                                            <field name="default_timezone"
                                                attrs="{'required': [('use_timezone', '=', True)]}"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <h2>Communication</h2>
                        <div class="row mt16 o_settings_container" name="event_type_communication">
                            <div class="col-12 col-lg-12 o_setting_box">
                                <div class="o_setting_left_pane">
                                    <field name="use_mail_schedule"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="use_mail_schedule"/>
                                    <div class="row mt16" attrs="{'invisible': [('use_mail_schedule', '=', False)]}">
                                        <div class="col-12">
                                            <field name="event_type_mail_ids" style="width: 100%;">
                                                <tree string="Communication" editable="bottom">
                                                    <field name="notification_type" invisible="1"/>
                                                    <field name="template_id" attrs="{'required': [('notification_type', '=', 'mail')]}"/>
                                                    <field name="interval_nbr" attrs="{'readonly':[('interval_unit', '=', 'now')]}"/>
                                                    <field name="interval_unit"/>
                                                    <field name="interval_type"/>
                                                </tree>
                                            </field>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <h2>Visibility</h2>
                        <div class="row mt16 o_settings_container" name="event_type_visibility">
                            <div class="col-12 col-lg-6 o_setting_box" name="event_type_visibility_seats">
                                <div class="o_setting_left_pane">
                                    <field name="has_seats_limitation"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="has_seats_limitation"/>
                                    <div class="row">
                                        <div class="col-lg-8 mt16" attrs="{'invisible': [('has_seats_limitation', '=', False)]}">
                                            <div>
                                                <label for="seats_max"/> <field name="seats_max"/>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" name="event_type_visibility_tags">
                                <div class="o_setting_left_pane"/>
                                <div class="o_setting_right_pane">
                                    <label for="tag_ids" string="Tags"/>
                                    <div class="row">
                                        <div class="col-12 mt16">
                                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_quick_create': True}"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <h2>Attendees</h2>
                        <div class="row mt16 o_settings_container" name="event_type_attendees">
                            <div class="col-12 o_setting_box" name="event_type_attendees_tickets">
                                <div class="o_setting_left_pane">
                                    <field name="use_ticket"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="use_ticket"/>
                                    <div class="row mt16" attrs="{'invisible': [('use_ticket', '=', False)]}">
                                        <div class="col">
                                            <field name="event_type_ticket_ids"
                                                class="w-100"
                                                context="{
                                                    'tree_view_ref': 'event.event_type_ticket_view_tree_from_type',
                                                    'form_view_ref': 'event.event_type_ticket_view_form_from_type'
                                                }"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" name="event_type_attendees_auto_confirm">
                                <div class="o_setting_left_pane">
                                    <field name="auto_confirm"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="auto_confirm"/>
                                    <div class="row">
                                        <div class="col-lg-8 mt16 text-muted">
                                            Events and registrations will automatically be confirmed
                                            upon creation, easing the flow for simple events.
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </sheet>
                </form>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_type_tree">
            <field name="name">event.type.tree</field>
            <field name="model">event.type</field>
            <field name="arch" type="xml">
                <tree string="Event Template">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                </tree>
            </field>
        </record>

        <record id="event_type_view_search" model="ir.ui.view">
            <field name="name">event.type.search</field>
            <field name="model">event.type</field>
            <field name="arch" type="xml">
                <search string="Event Templates">
                    <field name="name"/>
                </search>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_event_type">
            <field name="name">Event Templates</field>
            <field name="res_model">event.type</field>
        </record>

        <record id="menu_event_type" model="ir.ui.menu">
            <field name="action" ref="event.action_event_type"/>
        </record>

        <!-- EVENT.REGISTRATION ACTIONS -->
        <record id="act_event_registration_from_event" model="ir.actions.act_window">
            <field name="res_model">event.registration</field>
            <field name="name">Attendees</field>
            <field name="view_mode">kanban,tree,form,calendar,graph</field>
            <field name="domain">[('event_id', '=', active_id)]</field>
            <field name="context">{'default_event_id': active_id}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create an Attendee
                </p>
            </field>
        </record>

        <record id="event_registration_action" model="ir.actions.act_window">
            <field name="res_model">event.registration</field>
            <field name="name">Attendees</field>
            <field name="view_mode">kanban,tree,form,calendar,graph</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create an Attendee
                </p>
            </field>
        </record>

        <record id="event_registration_action_tree" model="ir.actions.act_window">
           <field name="name">Event registrations</field>
           <field name="type">ir.actions.act_window</field>
           <field name="res_model">event.registration</field>
           <field name="view_mode">tree,kanban,form,calendar,graph</field>
        </record>

        <!-- EVENT.EVENT VIEWS -->
        <record id="event_event_action_pivot" model="ir.actions.act_window" >
            <field name="name">Events Analysis</field>
            <field name="res_model">event.event</field>
            <field name="view_mode">pivot,graph</field>
        </record>

        <record model="ir.ui.view" id="view_event_form">
            <field name="name">event.event.form</field>
            <field name="model">event.event</field>
            <field name="arch" type="xml">
                <form string="Events" class="o_event_form_view">
                    <header>
                        <field name="stage_id" widget="statusbar" options="{'clickable': '1'}"/>
                    </header>
                    <sheet>
                        <div class="oe_button_box" name="button_box" groups="base.group_user">
                            <button name="%(event.act_event_registration_from_event)d"
                                    type="action"
                                    context="{'search_default_expected': True}"
                                    class="oe_stat_button"
                                    icon="fa-users"
                                    help="Total Registrations for this Event">
                                <field name="seats_expected" widget="statinfo" string="Attendees"/>
                            </button>
                        </div>
                        <field name="legend_blocked" invisible="1"/>
                        <field name="legend_normal" invisible="1"/>
                        <field name="legend_done" invisible="1"/>
                        <widget name="web_ribbon" text="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <field name="kanban_state" widget="state_selection" class="ml-auto float-right"/>
                        <div class="oe_title">
                            <label for="name" class="oe_edit_only" string="Event Name"/>
                            <h1><field name="name" placeholder="e.g. Conference for Architects"/></h1>
                        </div>
                        <group>
                            <group>
                                <field name="active" invisible="1"/>
                                <label for="date_begin" string="Date"/>
                                <div class="o_row">
                                    <field name="date_begin" widget="daterange" nolabel="1" class="oe_inline" options="{'related_end_date': 'date_end'}"/>
                                    <i class="fa fa-long-arrow-right mx-2" aria-label="Arrow icon" title="Arrow"/>
                                    <field name="date_end" widget="daterange" nolabel="1" class="oe_inline" options="{'related_start_date': 'date_begin'}"/>
                                </div>
                                <field name="date_tz"/>
                                <field name="event_type_id" string="Template" options="{'no_create':True}"/>
                                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_quick_create': True}"/>
                            </group>
                            <group>
                                <field name="organizer_id"/>
                                <field name="user_id" domain="[('share', '=', False)]"/>
                                <field name="company_id" groups="base.group_multi_company"/>
                                <field name="address_id"
                                    context="{'show_address': 1}"
                                    options='{"always_reload": True}'/>
                                <label for="seats_limited" string="Limit Registrations"/>
                                <div>
                                    <field name="seats_limited"/>
                                    <span attrs="{'invisible': [('seats_limited', '=', False)], 'required': [('seats_limited', '=', False)]}">to <field name="seats_max" class="oe_inline"/> Attendees</span>
                                </div>
                                <field name="auto_confirm"/>
                            </group>
                        </group>
                        <notebook>
                            <page string="Tickets" name="tickets">
                                <field name="event_ticket_ids" context="{
                                    'default_event_name': name,
                                    'tree_view_ref': 'event.event_event_ticket_view_tree_from_event',
                                    'form_view_ref': 'event.event_event_ticket_view_form_from_event',
                                    'kanban_view_ref': 'event.event_event_ticket_view_kanban_from_event'}" mode="tree,kanban"/>
                            </page>
                            <page string="Communication" name="event_communication">
                                <field name="event_mail_ids">
                                    <tree string="Communication" editable="bottom">
                                        <field name="sequence" widget="handle"/>
                                        <field name="notification_type" invisible="1"/>
                                        <field name="template_id" attrs="{'required': [('notification_type', '=', 'mail')]}" context="{'default_model': 'event.registration'}"/>
                                        <field name="interval_nbr" attrs="{'readonly':[('interval_unit','=','now')]}"/>
                                        <field name="interval_unit"/>
                                        <field name="interval_type"/>
                                        <field name="done"/>
                                    </tree>
                                </field>
                            </page>
                            <page string="Notes" name="event_notes">
                                <field name="note" placeholder="Add a note..."/>
                            </page>
                        </notebook>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" groups="base.group_user"/>
                        <field name="activity_ids"/>
                        <field name="message_ids"/>
                    </div>
                </form>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_tree">
            <field name="name">event.event.tree</field>
            <field name="model">event.event</field>
            <field name="arch" type="xml">
                <tree string="Events"
                    decoration-danger="(seats_max and seats_max&lt;seats_reserved)"
                    multi_edit="1"
                    sample="1">
                    <field name="name"/>
                    <field name="address_id" readonly="1"/>
                    <field name="organizer_id" readonly="1" optional="hide"/>
                    <field name="user_id" readonly="1" widget="many2one_avatar_user"/>
                    <field name="company_id" groups="base.group_multi_company" readonly="1" optional="show"/>
                    <field name="date_begin" readonly="1" widget="date"/>
                    <field name="date_end" readonly="1" widget="date"/>
                    <field name="stage_id" readonly="1"/>
                    <field name="seats_expected" string="Expected Attendees" sum="Total" readonly="1"/>
                    <field name="seats_used" sum="Total" readonly="1"/>
                    <field name="seats_max" string="Maximum Seats" sum="Total" readonly="1" optional="hide"/>
                    <field name="seats_reserved" sum="Total" readonly="1" optional="hide"/>
                    <field name="seats_unconfirmed" string="Unconfirmed Seats" sum="Total" readonly="1" optional="hide"/>
                    <field name="message_needaction" invisible="1" readonly="1"/>
                    <field name="activity_exception_decoration" widget="activity_exception" readonly="1"/>
                </tree>
            </field>
        </record>

        <record id="event_event_view_form_quick_create" model="ir.ui.view">
            <field name="name">event.event.form.quick_create</field>
            <field name="model">event.event</field>
            <field name="priority">1000</field>
            <field name="arch" type="xml">
                <form>
                    <group>
                        <field name="name"/>
                        <field name="date_begin"/>
                        <field name="date_end"/>
                    </group>
                </form>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_kanban">
            <field name="name">event.event.kanban</field>
            <field name="model">event.event</field>
            <field name="arch" type="xml">
                <kanban class="o_event_kanban_view" default_group_by="stage_id" quick_create_view="event.event_event_view_form_quick_create" sample="1">
                    <field name="user_id"/>
                    <field name="name"/>
                    <field name="stage_id" options='{"group_by_tooltip": {"description": "Description"}}'/>
                    <field name="address_id"/>
                    <field name="date_begin"/>
                    <field name="date_end"/>
                    <field name="auto_confirm"/>
                    <field name="seats_unconfirmed"/>
                    <field name="seats_reserved"/>
                    <field name="seats_used"/>
                    <field name="seats_expected"/>
                    <field name="legend_blocked"/>
                    <field name="legend_normal"/>
                    <field name="legend_done"/>
                    <field name="activity_ids"/>
                    <field name="activity_state"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="d-flex flex-column p-0 oe_kanban_card oe_kanban_global_click">
                                <div class="o_kanban_content p-0 m-0 position-relative row d-flex flex-fill">
                                    <div class="col-3 bg-primary p-2 text-center d-flex flex-column justify-content-center">
                                        <div t-esc="record.date_begin.raw_value.getDate()" class="o_event_fontsize_20"/>
                                        <div>
                                            <t t-esc="moment(record.date_begin.raw_value).format('MMM')"/>
                                            <t t-esc="record.date_begin.raw_value.getFullYear()"/>
                                        </div>
                                        <div><t t-esc="moment(record.date_begin.raw_value).format('LT')"/></div>
                                        <div t-if="moment(record.date_begin.raw_value).dayOfYear() !== moment(record.date_end.raw_value).dayOfYear()">
                                            <i class="fa fa-arrow-right o_event_fontsize_09" title="End date"/>
                                            <t t-esc="moment(record.date_end.raw_value).format('D MMM')"/>
                                         </div>
                                    </div>
                                    <div class="col-9 py-2 px-3 d-flex flex-column justify-content-between pt-3">
                                        <div>
                                            <div class="o_kanban_record_title o_text_overflow" t-att-title="record.name.value">
                                                <field name="name"/>
                                            </div>
                                            <div t-if="record.address_id.value"><i class="fa fa-map-marker" title="Location"/> <span class="o_text_overflow o_event_kanban_location" t-esc="record.address_id.value"/></div>
                                        </div>
                                        <h5 class="o_event_fontsize_11 p-0">
                                            <a name="%(act_event_registration_from_event)d" type="action" context="{'search_default_expected': True}">
                                                <t t-esc="record.seats_expected.raw_value"/> Expected attendees
                                            </a>
                                            <t t-set="total_seats" t-value="record.seats_reserved.raw_value + record.seats_used.raw_value"/>
                                            <div  class="pt-2 pt-md-0" t-if="total_seats > 0 and ! record.auto_confirm.raw_value"><br/>
                                                <a class="pl-2" name="%(act_event_registration_from_event)d" type="action" context="{'search_default_confirmed': True}">
                                                    <i class="fa fa-level-up fa-rotate-90" title="Confirmed"/><span class="pl-2"><t t-esc="total_seats"/> Confirmed</span>
                                                </a>
                                            </div>
                                        </h5>
                                        <div class="o_kanban_record_bottom">
                                            <div class="oe_kanban_bottom_left">
                                                <field name="activity_ids" widget="kanban_activity"/>
                                            </div>
                                            <div class="oe_kanban_bottom_right">
                                                <field name="kanban_state" widget="state_selection"/>
                                                <field name="user_id" widget="many2one_avatar_user"/>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_calendar">
            <field name="name">event.event.calendar</field>
            <field name="model">event.event</field>
            <field eval="2" name="priority"/>
            <field name="arch" type="xml">
                <calendar date_start="date_begin" date_stop="date_end" string="Event Organization" mode="month" color="event_type_id" event_limit="5">
                    <field name="user_id" avatar_field="image_128"/>
                    <field name="seats_expected"/>
                    <field name="seats_reserved"/>
                    <field name="seats_used"/>
                    <field name="seats_unconfirmed"/>
                    <field name="event_type_id" filters="1" invisible="1"/>
                </calendar>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_search">
            <field name="name">event.event.search</field>
            <field name="model">event.event</field>
            <field name="arch" type="xml">
                <search string="Events">
                    <field name="name" string="Event"/>
                    <field name="event_type_id"/>
                    <field name="user_id"/>
                    <field name="stage_id"/>
                    <filter string="My Events" name="myevents" help="My Events" domain="[('user_id', '=', uid)]"/>
                    <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction', '=', True)]"/>
                    <separator/>
                    <separator/>
                    <filter string="Upcoming/Running" name="upcoming"
                        domain="[('date_end', '&gt;=', datetime.datetime.combine(context_today(), datetime.time(0,0,0)))]" help="Upcoming events from today" />
                    <separator/>
                    <filter string="Start Date" name="start_date" date="date_begin"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <separator/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <group expand="0" string="Group By">
                        <filter string="Responsible" name="responsible" context="{'group_by': 'user_id'}"/>
                        <filter string="Template" name="event_type_id" context="{'group_by': 'event_type_id'}"/>
                        <filter string="Stage" name="stage_id" context="{'group_by': 'stage_id'}"/>
                        <filter string="Start Date" name="date_begin" domain="[]" context="{'group_by': 'date_begin'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="event_event_view_pivot" model="ir.ui.view" >
            <field name="name">event.event.view.pivot</field>
            <field name="model">event.event</field>
            <field name="arch" type="xml">
                <pivot string="Event" sample="1">
                    <field name="name" type="row"/>
                    <field name="seats_reserved" type="measure"/>
                </pivot>
            </field>
        </record>

        <record id="event_event_view_graph" model="ir.ui.view" >
            <field name="name">event.event.view.graph</field>
            <field name="model">event.event</field>
            <field name="arch" type="xml">
                <graph string="Events" sample="1">
                    <field name="name" type="row"/>
                    <field name="seats_available" type="measure"/>
                </graph>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_event_view">
           <field name="name">Events</field>
           <field name="type">ir.actions.act_window</field>
           <field name="res_model">event.event</field>
           <field name="view_mode">kanban,calendar,tree,form,pivot,graph</field>
           <field name="search_view_id" ref="view_event_search"/>
           <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Add a new event
              </p><p>
                Schedule and organize your events efficiently:
                track registrations and participations, automate the confirmation emails, sell tickets, etc.
              </p>
            </field>
        </record>

        <record id="event.menu_event_event" model="ir.ui.menu">
            <field name="action" ref="event.action_event_view"/>
        </record>
        <menuitem name="Events"
            id="event_event_menu_pivot_report"
            action="event_event_action_pivot"
            sequence="3"
            parent="event.menu_reporting_events"
            groups="event.group_event_manager"/>

        <!-- EVENT.REGISTRATION VIEWS -->
        <record model="ir.ui.view" id="view_event_registration_tree">
            <field name="name">event.registration.tree</field>
            <field name="model">event.registration</field>
            <field name="arch" type="xml">
                <tree string="Registration" multi_edit="1" sample="1">
                    <field name="create_date" optional="show" string="Registration Date"/>
                    <field name="date_open" optional="hide"/>
                    <field name="name"/>
                    <field name="partner_id" optional="hide"/>
                    <field name="email" optional="show"/>
                    <field name="phone" optional="show"/>
                    <field name="mobile" optional="hide"/>
                    <field name="event_id" invisible="context.get('default_event_id')"/>
                    <field name="event_ticket_id" domain="[('event_id', '=', event_id)]"/>
                    <field name="state" readonly="0"/>
                    <field name="company_id" groups="base.group_multi_company" optional="hide"/>
                    <field name="message_needaction" invisible="1"/>
                    <button name="action_confirm" string="Confirm" states="draft" type="object" icon="fa-check"/>
                    <button name="action_set_done" string="Mark as Attending" states="open" type="object" icon="fa-level-down"/>
                    <button name="action_cancel" string="Cancel" states="draft,open" type="object" icon="fa-times"/>
                    <field name="activity_exception_decoration" widget="activity_exception"/>
                </tree>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_registration_form">
            <field name="name">event.registration.form</field>
            <field name="model">event.registration</field>
            <field name="arch" type="xml">
                <form string="Event Registration">
                    <header>
                        <button name="action_send_badge_email" string="Send by Email" type="object" states="open,done" class="oe_highlight"/>
                        <button name="action_confirm" string="Confirm" states="draft" type="object" class="oe_highlight"/>
                        <button name="action_set_done" string="Attended" states="open" type="object" class="oe_highlight"/>
                        <button name="action_set_draft" string="Set To Unconfirmed" states="cancel,done" type="object" />
                        <button name="action_cancel" string="Cancel Registration" states="draft,open" type="object"/>
                        <field name="state" nolabel="1" colspan="2" widget="statusbar" statusbar_visible="draft,open,done"/>
                    </header>
                    <sheet string="Registration">
                        <div class="oe_button_box" name="button_box"/>
                        <group>
                            <group string="Attendee" name="attendee">
                                <field name="partner_id" attrs="{'readonly':[('state', '!=', 'draft')]}"/>
                                <field name="name"/>
                                <field name="email"/>
                                <field name="phone" class="o_force_ltr"/>
                                <field name="mobile" class="o_force_ltr"/>
                            </group>
                            <group string="Event Information" name="event">
                                <field name="event_id" attrs="{'readonly': [('state', '!=', 'draft')]}" options="{'no_create': True}"/>
                                <field name="event_ticket_id"
                                    domain="[
                                        ('event_id', '=', event_id),
                                        '|', ('seats_limited', '=', False), ('seats_available', '>', 0)
                                    ]"
                                    attrs="{'invisible': [('event_id', '=', False)]}"
                                    options="{'no_open': True, 'no_create': True}"/>
                                <field name="date_open" groups="base.group_no_one"/>
                                <field name="date_closed" groups="base.group_no_one"/>
                            </group>
                            <group string="Marketing" name="utm_link" groups="base.group_no_one">
                                <field name="utm_campaign_id"/>
                                <field name="utm_medium_id"/>
                                <field name="utm_source_id"/>
                            </group>
                        </group>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids"/>
                        <field name="activity_ids"/>
                        <field name="message_ids" options="{'post_refresh': 'recipients'}"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="event_registration_view_kanban" model="ir.ui.view">
            <field name="name">event.registration.kanban</field>
            <field name="model">event.registration</field>
            <field name="priority">10</field>
            <field name="arch" type="xml">
                <kanban class="o_event_attendee_kanban_view" default_order="name, create_date desc" sample="1">
                    <field name="name"/>
                    <field name="partner_id"/>
                    <field name="state"/>
                    <field name="email"/>
                    <field name="event_ticket_id"/>
                    <templates>
                        <t t-name="event_attendees_kanban_icons_desktop">
                            <div class="d-none d-md-block h-100">
                                <div id="event_attendees_kanban_icons_desktop" class="h-100 float-right p-2 d-flex align-items-end flex-column">
                                    <a class="btn btn-md btn-primary" string="Confirm Registration" name="action_confirm" type="object" states="draft" role="button">
                                        <i class="fa fa-check" role="img" aria-label="Confirm button" title="Confirm Registration"/>
                                    </a>
                                    <a class="btn btn-md btn-primary" string="Confirm Attendance" name="action_set_done" type="object" states="open" role="button">
                                        <i class="fa fa-user-plus" role="img" aria-label="Attended button" title="Confirm Attendance"/>
                                    </a>
                                    <span class="text-muted" states="done">Attended</span>
                                    <span class="text-muted" states="cancel">Canceled</span>
                                </div>
                            </div>
                        </t>
                        <t t-name="event_attendees_kanban_icons_mobile">
                            <div id="event_attendees_kanban_icons_mobile" class="d-md-none h-100 pl-4">
                                <a class="btn btn-primary d-flex justify-content-center align-items-center h-100 w-100"
                                    string="Confirm Registration" name="action_confirm" type="object" states="draft" role="button">
                                    <i class="fa fa-check fa-3x" role="img" aria-label="Confirm button" title="Confirm Registration"/>
                                </a>
                                <a class="btn btn-primary d-flex justify-content-center align-items-center h-100 w-100"
                                    string="Confirm Attendance" name="action_set_done" type="object" states="open" role="button">
                                    <i class="fa fa-user-plus fa-3x" role="img" aria-label="Attended button" title="Confirm Attendance"/>
                                </a>
                                <div class="d-flex justify-content-center align-items-center h-100 w-100">
                                    <span class="text-muted" states="done" >Attended</span>
                                    <span class="text-muted" states="cancel" >Canceled</span>
                                </div>
                            </div>
                        </t>
                        <t t-name="kanban-box">
                            <div class="oe_kanban_global_click o_event_registration_kanban container-fluid p-0">
                                <div class="row h-100">
                                    <div class="col-9 pr-0">
                                        <div class="oe_kanban_content h-100">
                                            <div class="o_kanban_record_body pt-1 pl-2 h-100 d-flex flex-column">
                                                <b class="o_kanban_record_title"><field name="name"/></b>
                                                <field name="event_id" invisible="context.get('default_event_id')" />
                                                <span attrs="{'invisible': [('partner_id', '=', False)]}">Booked by <field name="partner_id" /></span>
                                                <div id="event_ticket_id" class="o_field_many2manytags o_field_widget d-flex mt-auto">
                                                    <t t-if="record.event_ticket_id.raw_value">
                                                        <div t-attf-class="badge badge-pill o_tag_color_#{(record.event_ticket_id.raw_value % 11) + 1}" >
                                                            <b><span class="o_badge_text"><t t-esc="record.event_ticket_id.value"/></span></b>
                                                        </div>
                                                    </t>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                    <div id="event_attendees_kanban_icons" class="col-3 pl-0">
                                        <t t-call="event_attendees_kanban_icons_desktop"/>
                                        <t t-call="event_attendees_kanban_icons_mobile"/>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="view_event_registration_calendar" model="ir.ui.view">
            <field name="name">event.registration.calendar</field>
            <field name="model">event.registration</field>
            <field eval="2" name="priority"/>
            <field name="arch" type="xml">
                <calendar date_start="event_begin_date" date_stop="event_end_date" string="Event Registration" color="event_id" event_limit="5">
                    <field name="event_id" filters="1"/>
                    <field name="name"/>
                </calendar>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_registration_pivot">
            <field name="name">event.registration.pivot</field>
            <field name="model">event.registration</field>
            <field name="arch" type="xml">
                <pivot string="Registration" display_quantity="True" sample="1">
                    <field name="event_id" type="row"/>
                </pivot>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_registration_graph">
            <field name="name">event.registration.graph</field>
            <field name="model">event.registration</field>
            <field name="arch" type="xml">
                <graph string="Registration" sample="1">
                    <field name="event_id"/>
                </graph>
            </field>
        </record>

        <record model="ir.ui.view" id="view_registration_search">
            <field name="name">event.registration.search</field>
            <field name="model">event.registration</field>
            <field name="arch" type="xml">
                <search string="Event Registration">
                    <field name="id" string="Registration ID"/>
                    <field name="name" string="Participant" filter_domain="['|', ('name', 'ilike', self), ('email', 'ilike', self)]"/>
                    <filter string="Ongoing Events" name="filter_is_ongoing" domain="[('event_id.is_ongoing', '=', True)]"/>
                    <separator/>
                    <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction','=',True)]"/>
                    <filter string="Expected" name="expected" domain="[('state', 'in', ['draft', 'open', 'done'])]"/>
                    <separator/>
                    <filter string="Unconfirmed" name="unconfirmed" domain="[('state', '=', 'draft')]"/>
                    <filter string="Confirmed" name="confirmed" domain="[('state', '=', 'open')]"/>
                    <filter string="Attended" name="attended" domain="[('state', '=', 'done')]"/>
                    <separator/>
                    <filter string="Registration Date" name="filter_date_open" date="date_open"/>
                    <filter string="Event Start Date" name="filter_event_begin_date" date="event_begin_date"/>
                    <filter string="Attended Date" name="filter_date_closed" date="date_closed"/>
                    <field name="event_id"/>
                    <field name="partner_id"/>
                    <field name="company_id"/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <group expand="0" string="Group By">
                        <filter string="Partner" name="partner" domain="[]" context="{'group_by':'partner_id'}"/>
                        <filter string="Event" name="group_event" domain="[]" context="{'group_by':'event_id'}"/>
                        <filter string="Ticket Type" name ="group_event_ticket_id" domain="[]" context="{'group_by': 'event_ticket_id'}"/>
                        <filter string="Status" name="status" domain="[]" context="{'group_by':'state'}"/>
                        <filter string="Registration Date" name="createmonth" domain="[]" context="{'group_by': 'create_date:month'}"/>
                   </group>
                </search>
            </field>
        </record>

        <record id="action_registration" model="ir.actions.act_window">
            <field name="name">Attendees</field>
            <field name="res_model">event.registration</field>
            <field name="domain"></field>
            <field name="view_mode">pivot,graph,kanban,tree,form</field>
            <field name="context">{}</field>
            <field name="search_view_id" ref="view_registration_search"/>
        </record>

        <menuitem name="Attendees"
            id="menu_action_registration"
            parent="event.menu_reporting_events"
            sequence="4"
            action="action_registration"
            groups="event.group_event_manager"/>

        <!-- EVENT.MAIL VIEWS -->
        <record model="ir.ui.view" id="view_event_mail_form">
            <field name="name">event.mail.form</field>
            <field name="model">event.mail</field>
            <field name="arch" type="xml">
                <form string="Event Mail Scheduler">
                    <sheet>
                        <group>
                            <group>
                                <field name="event_id"/>
                                <field name="notification_type" invisible="1"/>
                                <field name="template_id" attrs="{'required': [('notification_type', '=', 'mail')]}"/>
                                <field name="mail_sent"/>
                            </group>
                            <group>
                                <label for="interval_nbr"/>
                                <div class="o_row">
                                    <field name="interval_nbr" attrs="{'invisible': [('interval_unit', '=', 'now')], 'readonly': [('interval_unit', '=', 'now')]}"/>
                                    <field name="interval_unit"/>
                                </div>
                                <field name="interval_type"/>
                                <field name="scheduled_date"/>
                            </group>
                        </group>
                        <notebook groups="base.group_no_one">
                            <page string="Registration Mails" name="registration_mails">
                                <field name="mail_registration_ids">
                                    <tree string="Registration mail" editable="bottom">
                                        <field name="registration_id"/>
                                        <field name="scheduled_date"/>
                                        <field name="mail_sent" string="Sent"/>
                                    </tree>
                                </field>
                            </page>
                        </notebook>
                    </sheet>
                </form>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_mail_tree">
            <field name="name">event.mail.tree</field>
            <field name="model">event.mail</field>
            <field name="arch" type="xml">
                <tree string="Event Mail Schedulers">
                    <field name="event_id"/>
                    <field name="notification_type" invisible="1"/>
                    <field name="template_id" attrs="{'required': [('notification_type', '=', 'mail')]}"/>
                    <field name="scheduled_date"/>
                    <field name="mail_sent" string="Sent"/>
                    <field name="done"/>
                </tree>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_event_mail">
            <field name="name">Events Mail Schedulers</field>
            <field name="res_model">event.mail</field>
        </record>

        <record id="menu_event_mail_schedulers" model="ir.ui.menu">
            <field name="action" ref="event.action_event_mail"/>
        </record>
    </data>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.event</field>
            <field name="model">res.config.settings</field>
            <field name="priority" eval="65"/>
            <field name="inherit_id" ref="base.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//div[hasclass('settings')]" position="inside">
                    <div class="app_settings_block" data-string="Events" string="Events" data-key="event" groups="event.group_event_manager">
                        <h2>Events</h2>
                        <div class="row mt16 o_settings_container" name="events_setting_container">
                            <div class="col-12 col-lg-6 o_setting_box"
                                id="manage_tracks"
                                title="Add a navigation menu to your event web pages with schedule, tracks, a track proposal form, etc.">
                                <div class="o_setting_left_pane">
                                    <field name="module_website_event_track"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label string="Schedule &amp; Tracks" for="module_website_event_track"/>
                                    <div class="text-muted">
                                        Manage &amp; publish a schedule with tracks
                                    </div>
                                    <div class="mt-3 d-flex" attrs="{'invisible': [('module_website_event_track', '=', False)]}">
                                        <field name="module_website_event_track_live" class="w-auto"/>
                                        <div>
                                            <label string="Live Broadcast" for="module_website_event_track_live"/><br/>
                                            <span class="text-muted">Air your tracks online through a Youtube integration</span>
                                        </div>
                                    </div>
                                    <div class="mt-3 d-flex" attrs="{'invisible': [('module_website_event_track', '=', False)]}">
                                        <field name="module_website_event_track_quiz" class="w-auto"/>
                                        <div>
                                            <label string="Event Gamification" for="module_website_event_track_quiz"/><br/>
                                            <span class="text-muted">Share a quiz to your attendees once a track is over</span>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box">
                                <div>
                                    <div class="o_setting_left_pane">
                                        <field name="module_website_event_meet"/>
                                    </div>
                                    <div class="o_setting_right_pane">
                                        <label string="Community Chat Rooms" for="module_website_event_meet"/>
                                        <div class="text-muted">
                                            Foster interactions between attendees by creating virtual conference rooms
                                        </div>
                                    </div>
                                </div>
                                <div class="mt-3">
                                    <div class="o_setting_left_pane">
                                        <field name="module_website_event_track_exhibitor"/>
                                    </div>
                                    <div class="o_setting_right_pane">
                                        <label string="Online Exhibitors" for="module_website_event_track_exhibitor"/>
                                        <div class="text-muted">
                                            Upgrade Sponsors into Exhibitors with virtual conference booths
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <h2>Registration</h2>
                        <div class="row mt16 o_settings_container" name="registration_setting_container">
                            <div class="col-12 col-lg-6 o_setting_box" id="sell_tickets">
                                <div class="o_setting_left_pane">
                                    <field name="module_event_sale"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_event_sale"/>
                                    <div class="text-muted">
                                        Sell tickets with sales orders
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" name="event_settings_website">
                                <div class="o_setting_left_pane">
                                    <field name="module_website_event_sale"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_website_event_sale"/>
                                    <div class="text-muted">
                                        Sell tickets on your website
                                    </div>
                                </div>
                            </div>
                        </div>
                        <h2>Attendance</h2>
                        <div class="row mt16 o_settings_container" name="attendance_setting_container">
                            <div class="col-12 col-lg-6 o_setting_box" id="event_barcode">
                                <div class="o_setting_left_pane">
                                    <field name="module_event_barcode" widget="upgrade_boolean"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_event_barcode"/>
                                    <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." groups="base.group_multi_company"/>
                                    <div class="text-muted" name="event_barcode">
                                        Scan badges to confirm attendances
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </xpath>
            </field>
        </record>

        <record id="action_event_configuration" model="ir.actions.act_window">
            <field name="name">Settings</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">res.config.settings</field>
            <field name="view_mode">form</field>
            <field name="target">inline</field>
            <field name="context">{'module' : 'event', 'bin_size': False}</field>
        </record>

        <menuitem id="menu_event_global_settings" name="Settings"
            parent="menu_event_configuration" sequence="0" action="action_event_configuration" groups="base.group_system"/>
    </data>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record id="res_partner_view_tree" model="ir.ui.view">
            <field name="name">view.res.partner.form.event.inherited</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="groups_id" eval="[(5,)]"/>
            <field name="priority" eval="6"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <button class="oe_stat_button"
                        groups="event.group_event_user"
                        type="object"
                        icon="fa-ticket"
                        name="action_event_view" attrs="{'invisible': [('event_count','=', 0)]}">
                        <field string="Events" name="event_count" widget="statinfo"/>
                    </button>
                </div>
            </field>
        </record>
    </data>
</odoo>

```

