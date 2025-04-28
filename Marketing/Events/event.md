# Odoo Module: event

Category: Marketing/Events

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import wizard
from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': 'Events Organization',
    'version': '1.0',
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
    'depends': ['base_setup', 'mail', 'portal'],
    'data': [
        'security/event_security.xml',
        'security/ir.model.access.csv',
        'wizard/event_confirm_view.xml',
        'views/event_views.xml',
        'report/event_event_templates.xml',
        'report/event_event_reports.xml',
        'data/email_template_data.xml',
        'data/event_data.xml',
        'views/res_config_settings_views.xml',
        'views/event_templates.xml',
        'views/res_partner_views.xml',
    ],
    'demo': [
        'data/event_demo.xml',
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

    @route(['''/event/<model("event.event", "[('state', 'in', ('confirm', 'done'))]"):event>/ics'''], type='http', auth="public")
    def event_ics_file(self, event, **kwargs):
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
    Thank you,<br/>
    % if object.event_id.user_id and object.event_id.user_id.signature:
        ${object.event_id.user_id.signature | safe}
    % endif
</div></field>
            <field name="report_template" ref="report_event_registration_badge"/>
            <field name="report_name">badge_of_${(object.event_id.name or '').replace('/','_')}</field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="auto_delete" eval="True"/>
            <field name="user_signature" eval="False"/>
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
                    <img src="${'/logo.png?company=%s' % object.company_id.id}" style="padding: 0px; margin: 0px; height: auto; width: 80px;" alt="${'%s' % object.company_id.name}"/>
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
                        % if 'website_published' in object.event_id and object.event_id.website_published:
                            <a href="${object.event_id.website_url}" style="color:#875A7B;text-decoration:none;">${object.event_id.name}</a>
                        % else:
                            <strong>${object.event_id.name}</strong>
                        % endif
                        for attendee ${object.name}.
                    </div>
                    % if 'website_published' in object.event_id and object.event_id.website_published:
                    <div style="margin: 16px 0px 16px 0px;">
                        <a href="${object.event_id.website_url}"
                            style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:16px;">View Event</a><br />
                    </div>
                    % endif
                    <div>
                        See you soon,<br/>
                        <span style="color: #454748;">
                        -- <br/>
                        % if object.event_id.organizer_id:
                            ${object.event_id.organizer_id.name}
                        % else:
                            The organizers.
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
                            % if object.event_id.address_id.country_id.name:
                                <td style="vertical-align:top;">
                                    <img src="/web_editor/font_to_img/61505/rgb(81,81,102)/34" style="padding:4px;max-width:inherit;" height="34" alt=""/>
                                </td>
                                <td style="padding: 0px 10px 0px 10px;width:50%;vertical-align:top;">
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
                                </td>
                            % endif
                        </tr>
                    </table>
                </td></tr>
                <tr><td style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
                % if object.event_id.organizer_id:
                <tr><td valign="top" style="font-size: 14px;">
                    <div>
                        <span style="font-weight:300;margin:10px 0px">Questions about this event?</span>
                        <div>Please contact the organizer:</div>
                        <ul>
                            <li>${object.event_id.organizer_id.name}</li>
                            % if object.event_id.organizer_id.email
                                <li>Mail: <a href="mailto:${object.event_id.organizer_id.email}" style="text-decoration:none;color:#875A7B;">${object.event_id.organizer_id.email}</a></li>
                            % endif
                            % if object.event_id.organizer_id.phone
                                <li>Phone: ${object.event_id.organizer_id.phone}</li>
                            % endif
                        </ul>
                    </div>
                </td></tr>
                % endif
                <tr><td valign="top" style="font-size: 14px;">
                    <table style="width:100%;border-top:1px solid #e1e1e1;">
                        <tr>
                            <td style="padding:25px 0px;">
                                <strong>Add this event to your calendar</strong>
                                <a href="https://www.google.com/calendar/render?action=TEMPLATE&amp;text=${object.event_id.name}&amp;dates=${date_begin}/${date_end}&amp;location=${location}" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;" target="new"><img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> Google</a>
                                <a href="/event/${slug(object.event_id)}/ics" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;"><img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> iCal/Outlook</a>
                                <a href="https://calendar.yahoo.com/?v=60&amp;view=d&amp;type=20&amp;title=${object.event_id.name}&amp;in_loc=${location}&amp;st=${format_datetime(object.event_id.date_begin, tz='UTC', dt_format='yyyyMMdd\'T\'HHmmss')}&amp;et=${format_datetime(object.event_id.date_end, tz='UTC', dt_format='yyyyMMdd\'T\'HHmmss')}" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;" target="new">
                                    <img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> Yahoo</a>
                            </td>
                        </tr>
                    </table>
                </td></tr>
                <tr><td style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
                % if object.event_id.address_id:
                <tr><td valign="top" style="font-size: 14px;">
                    <table style="width:100%;"><tr><td>
                        <div>
                            <a href="https://maps.google.com/maps?q=${location}" target="new">
                                <img src="http://maps.googleapis.com/maps/api/staticmap?autoscale=1&amp;size=598x200&amp;maptype=roadmap&amp;format=png&amp;visual_refresh=true&amp;markers=size:mid%7Ccolor:0xa5117d%7Clabel:%7C${location}" style="vertical-align:bottom; width: 100%;" alt="Google Maps"/>
                            </a>
                        </div>
                    </td></tr></table>
                </td></tr>
                % endif
            </table>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- FOOTER BY -->
% if object.company_id
<tr><td align="center" style="min-width: 590px;">
    <table width="590" border="0" cellpadding="0" cellspacing="0" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
      <tr><td style="text-align: center; font-size: 14px;">
        Sent by <a target="_blank" href="${object.company_id.website}" style="color: #875A7B;">${object.company_id.name}</a>
        % if 'website_url' in object.event_id and object.event_id.website_url:
        <br />
        Discover <a href="/event" style="text-decoration:none;color:#717188;">all our events</a>.
        % endif
      </td></tr>
    </table>
</td></tr>
% endif
</table>
            </field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="user_signature" eval="False"/>
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
                    <img src="${'/logo.png?company=%s' % object.company_id.id}" style="padding: 0px; margin: 0px; height: auto; width: 80px;" alt="${'%s' % object.company_id.name}"/>
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
                        % if 'website_published' in object.event_id and object.event_id.website_published:
                            <a href="${object.event_id.website_url}" style="color:#875A7B;text-decoration:none;">${object.event_id.name}</a>
                        % else:
                            <strong>${object.event_id.name}</strong>
                        % endif
                        is starting <strong>${object.get_date_range_str()}</strong>.
                    </div>
                    % if 'website_published' in object.event_id and object.event_id.website_published:
                    <div style="margin: 16px 0px 16px 0px;">
                        <a href="${object.event_id.website_url}"
                            style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:16px;">View Event</a><br />
                    </div>
                    % endif
                    <div>
                        We confirm your registration and hope to meet you there,<br/>
                        <span style="color: #454748;">
                        -- <br/>
                        % if object.event_id.organizer_id:
                            ${object.event_id.organizer_id.name}
                        % else:
                            The organizers.
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
                            % if object.event_id.address_id.country_id.name:
                                <td style="vertical-align:top;">
                                    <img src="/web_editor/font_to_img/61505/rgb(81,81,102)/34" style="padding:4px;max-width:inherit;" height="34" alt=""/>
                                </td>
                                <td style="padding: 0px 10px 0px 10px;width:50%;vertical-align:top;">
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
                                </td>
                            % endif
                        </tr>
                    </table>
                </td></tr>
                <tr><td style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
                % if object.event_id.organizer_id:
                <tr><td valign="top" style="font-size: 14px;">
                    <div>
                        <span style="font-weight:300;margin:10px 0px">Questions about this event?</span>
                        <div>Please contact the organizer:</div>
                        <ul>
                            <li>${object.event_id.organizer_id.name}</li>
                            % if object.event_id.organizer_id.email
                                <li>Mail: <a href="mailto:${object.event_id.organizer_id.email}" style="text-decoration:none;color:#875A7B;">${object.event_id.organizer_id.email}</a></li>
                            % endif
                            % if object.event_id.organizer_id.phone
                                <li>Phone: ${object.event_id.organizer_id.phone}</li>
                            % endif
                        </ul>
                    </div>
                </td></tr>
                % endif
                <tr><td valign="top" style="font-size: 14px;">
                    <table style="width:100%;border-top:1px solid #e1e1e1;">
                        <tr>
                            <td style="padding:25px 0px;">
                                <strong>Add this event to your calendar</strong>
                                <a href="https://www.google.com/calendar/render?action=TEMPLATE&amp;text=${object.event_id.name}&amp;dates=${date_begin}/${date_end}&amp;location=${location}" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;" target="new"><img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> Google</a>
                                <a href="/event/${slug(object.event_id)}/ics" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;"><img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> iCal/Outlook</a>
                                <a href="https://calendar.yahoo.com/?v=60&amp;view=d&amp;type=20&amp;title=${object.event_id.name}&amp;in_loc=${location}&amp;st=${format_datetime(object.event_id.date_begin, tz='UTC', dt_format='yyyyMMdd\'T\'HHmmss')}&amp;et=${format_datetime(object.event_id.date_end, tz='UTC', dt_format='yyyyMMdd\'T\'HHmmss')}" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;" target="new">
                                    <img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> Yahoo</a>
                            </td>
                        </tr>
                    </table>
                </td></tr>
                <tr><td style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
                % if object.event_id.address_id:
                <tr><td valign="top" style="font-size: 14px;">
                    <table style="width:100%;"><tr><td>
                        <div>
                            <a href="https://maps.google.com/maps?q=${location}" target="new">
                                <img src="http://maps.googleapis.com/maps/api/staticmap?autoscale=1&amp;size=598x200&amp;maptype=roadmap&amp;format=png&amp;visual_refresh=true&amp;markers=size:mid%7Ccolor:0xa5117d%7Clabel:%7C${location}" style="vertical-align:bottom; width: 100%;" alt="Google Maps"/>
                            </a>
                        </div>
                    </td></tr></table>
                </td></tr>
                % endif
            </table>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- FOOTER BY -->
% if object.company_id
<tr><td align="center" style="min-width: 590px;">
    <table width="590" border="0" cellpadding="0" cellspacing="0" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
      <tr><td style="text-align: center; font-size: 14px;">
        Sent by <a target="_blank" href="${object.company_id.website}" style="color: #875A7B;">${object.company_id.name}</a>
        % if 'website_url' in object.event_id and object.event_id.website_url:
        <br />
        Discover <a href="/event" style="text-decoration:none;color:#717188;">all our events</a>.
        % endif
      </td></tr>
    </table>
</td></tr>
% endif
</table>
            </field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="user_signature" eval="False"/>
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

        <record id="event_type_data_online" model="event.type">
            <field name="name">Online</field>
            <field name="is_online" eval="True"/>
            <field name="auto_confirm" eval="True"/>
            <field name="use_mail_schedule" eval="False"/>
        </record>

        <record id="event_type_data_physical" model="event.type">
            <field name="name">Physical Event</field>
            <field name="is_online" eval="False"/>
            <field name="auto_confirm" eval="True"/>
            <field name="use_mail_schedule" eval="False"/>
        </record>
    </data>
</odoo>

```

## File: data\event_demo.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
    <record id="base.user_demo" model="res.users">
        <field name="groups_id" eval="[(4, ref('event.group_event_user'))]"/>
     </record>

    <!-- Event Type -->
    <record id="event_type_0" model="event.type">
        <field name="name">Seminar</field>
    </record>

    <record id="event_type_1" model="event.type">
        <field name="name">Exhibition</field>
    </record>

    <record id="event_type_2" model="event.type">
        <field name="name">Conference</field>
    </record>

    <record id="event_type_3" model="event.type">
        <field name="name">Show</field>
    </record>

    <record id="event_type_4" model="event.type">
        <field name="name">Training</field>
    </record>

    <record id="event_type_5" model="event.type">
        <field name="name">Sport</field>
    </record>

    <!-- Event -->
    <record id="event_0" model="event.event">
        <field name="name">Design Fair Los Angeles</field>
        <field name="user_id" ref="base.user_admin"/>
        <field eval="(DateTime.now() + timedelta(days=1)).strftime('%Y-%m-%d 08:00:00')" name="date_begin"/>
        <field eval="(DateTime.now() + timedelta(days=5)).strftime('%Y-%m-%d 18:00:00')" name="date_end"/>
        <field name="seats_availability">limited</field>
        <field name="seats_max">500</field>
        <field name="address_id" ref="base.res_partner_1"/>
        <field name="event_type_id" ref="event_type_1"/>
    </record>

    <record id="event.event_1" model="event.event">
        <field name="name">Great Reno Ballon Race</field>
        <field name="user_id" ref="base.user_admin"/>
        <field eval="(DateTime.today()+ timedelta(days=100)).strftime('%Y-%m-%d 20:15:00')" name="date_begin"/>
        <field eval="(DateTime.today()+ timedelta(days=101)).strftime('%Y-%m-%d 00:30:00')" name="date_end"/>
        <field name="event_type_id" ref="event_type_1"/>
        <field name="address_id" ref="base.res_partner_1"/>
        <field name="seats_availability">unlimited</field>
    </record>

    <record id="event_2" model="event.event">
        <field name="name">Conference for Architects</field>
        <field name="user_id" ref="base.user_admin"/>
        <field eval="(DateTime.today()+ timedelta(days=5)).strftime('%Y-%m-%d 07:00:00')" name="date_begin"/>
        <field eval="(DateTime.today()+ timedelta(days=5)).strftime('%Y-%m-%d 16:30:00')" name="date_end"/>
        <field name="event_type_id" ref="event_type_2"/>
        <field name="address_id" ref="base.res_partner_4"/>
        <field name="seats_availability">limited</field>
        <field name="seats_max">200</field>
    </record>

    <record id="event.event_3" model="event.event">
        <field name="name">Live Music Festival</field>
        <field name="user_id" ref="base.user_admin"/>
        <field eval="(DateTime.today()+ timedelta(days=130)).strftime('%Y-%m-%d 20:15:00')" name="date_begin"/>
        <field eval="(DateTime.today()+ timedelta(days=133)).strftime('%Y-%m-%d 00:30:00')" name="date_end"/>
        <field name="event_type_id" ref="event_type_3"/>
        <field name="address_id" ref="base.res_partner_3"/>
        <field name="seats_availability">unlimited</field>
    </record>

    <record id="event.event_4" model="event.event">
        <field name="name">Business workshops</field>
        <field name="user_id" ref="base.user_admin"/>
        <field eval="(DateTime.today()+ timedelta(days=50)).strftime('%Y-%m-%d 18:00:00')" name="date_begin"/>
        <field eval="(DateTime.today()+ timedelta(days=50)).strftime('%Y-%m-%d 22:30:00')" name="date_end"/>
        <field name="event_type_id" ref="event_type_4"/>
        <field name="address_id" ref="base.res_partner_4"/>
    </record>

    <record id="event.event_5" model="event.event">
        <field name="name">Hockey Tournament</field>
        <field name="user_id" ref="base.user_admin"/>
        <field eval="(DateTime.today()+ timedelta(days=370)).strftime('%Y-%m-%d 09:00:00')" name="date_begin"/>
        <field eval="(DateTime.today()+ timedelta(days=371)).strftime('%Y-%m-%d 17:00:00')" name="date_end"/>
        <field name="event_type_id" ref="event_type_5"/>
        <field name="address_id" ref="base.res_partner_1"/>
    </record>

    <record id="event.event_6" model="event.event">
        <field name="name">An unpublished event</field>
        <field name="user_id" ref="base.user_admin"/>
        <field eval="(DateTime.today()+ timedelta(days=30)).strftime('%Y-%m-%d 09:30:00')" name="date_begin"/>
        <field eval="(DateTime.today()+ timedelta(days=30)).strftime('%Y-%m-%d 17:30:00')" name="date_end"/>
        <field name="event_type_id" ref="event_type_1"/>
        <field name="address_id" ref="base.res_partner_1"/>
    </record>

    <function model="event.event" name="button_confirm" eval="[ref('event_0')]"/>
    <function model="event.event" name="button_confirm" eval="[ref('event_1')]"/>
    <function model="event.event" name="button_confirm" eval="[ref('event_2')]"/>
    <function model="event.event" name="button_confirm" eval="[ref('event_3')]"/>
    <function model="event.event" name="button_confirm" eval="[ref('event_4')]"/>
    <function model="event.event" name="button_confirm" eval="[ref('event_5')]"/>
    <function model="event.event" name="button_confirm" eval="[ref('event_6')]"/>

    <!-- Attendee -->
    <record id="reg_2_0" model="event.registration">
        <field name="name">Camptocamp</field>
        <field name="email">odoo@camptocamp.com</field>
        <field name="phone">+41 21 619 10 04 </field>
        <field name="event_id" ref="event_2"/>
        <field name="partner_id" ref="base.res_partner_12"/>
    </record>

    </data>
</odoo>

```

## File: models\event.py

```python
# -*- coding: utf-8 -*-

import logging
import pytz

from odoo import _, api, fields, models, SUPERUSER_ID
from odoo.tools import format_datetime
from odoo.exceptions import AccessError, UserError, ValidationError
from odoo.tools.translate import html_translate

from dateutil.relativedelta import relativedelta

_logger = logging.getLogger(__name__)

try:
    import vobject
except ImportError:
    _logger.warning("`vobject` Python module not found, iCal file generation disabled. Consider installing this module if you want to generate iCal files")
    vobject = None


class EventType(models.Model):
    _name = 'event.type'
    _description = 'Event Category'
    _order = 'sequence, id'

    @api.model
    def _get_default_event_type_mail_ids(self):
        return [(0, 0, {
            'notification_type': 'mail',
            'interval_unit': 'now',
            'interval_type': 'after_sub',
            'template_id': self.env.ref('event.event_subscription').id,
        }), (0, 0, {
            'notification_type': 'mail',
            'interval_nbr': 1,
            'interval_unit': 'days',
            'interval_type': 'before_event',
            'template_id': self.env.ref('event.event_reminder').id,
        }), (0, 0, {
            'notification_type': 'mail',
            'interval_nbr': 10,
            'interval_unit': 'days',
            'interval_type': 'before_event',
            'template_id': self.env.ref('event.event_reminder').id,
        })]

    name = fields.Char('Event Category', required=True, translate=True)
    sequence = fields.Integer()
    # registration
    has_seats_limitation = fields.Boolean(
        'Limited Seats', default=False)
    default_registration_min = fields.Integer(
        'Minimum Registrations', default=0,
        help="It will select this default minimum value when you choose this event")
    default_registration_max = fields.Integer(
        'Maximum Registrations', default=0,
        help="It will select this default maximum value when you choose this event")
    auto_confirm = fields.Boolean(
        'Automatically Confirm Registrations', default=True,
        help="Events and registrations will automatically be confirmed "
             "upon creation, easing the flow for simple events.")
    # location
    is_online = fields.Boolean(
        'Online Event', help='Online events like webinars do not require a specific location and are hosted online.')
    use_timezone = fields.Boolean('Use Default Timezone')
    default_timezone = fields.Selection(
        '_tz_get', string='Timezone',
        default=lambda self: self.env.user.tz)
    # communication
    use_hashtag = fields.Boolean('Use Default Hashtag')
    default_hashtag = fields.Char('Twitter Hashtag')
    use_mail_schedule = fields.Boolean(
        'Automatically Send Emails', default=True)
    event_type_mail_ids = fields.One2many(
        'event.type.mail', 'event_type_id', string='Mail Schedule',
        copy=False,
        default=lambda self: self._get_default_event_type_mail_ids())

    @api.onchange('has_seats_limitation')
    def _onchange_has_seats_limitation(self):
        if not self.has_seats_limitation:
            self.default_registration_min = 0
            self.default_registration_max = 0

    @api.model
    def _tz_get(self):
        return [(x, x) for x in pytz.all_timezones]


class EventEvent(models.Model):
    """Event"""
    _name = 'event.event'
    _description = 'Event'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'date_begin'

    name = fields.Char(
        string='Event', translate=True, required=True,
        readonly=False, states={'done': [('readonly', True)]})
    active = fields.Boolean(default=True)
    user_id = fields.Many2one(
        'res.users', string='Responsible',
        default=lambda self: self.env.user,
        tracking=True,
        readonly=False, states={'done': [('readonly', True)]})
    company_id = fields.Many2one(
        'res.company', string='Company', change_default=True,
        default=lambda self: self.env.company,
        required=False, readonly=False, states={'done': [('readonly', True)]})
    organizer_id = fields.Many2one(
        'res.partner', string='Organizer',
        tracking=True,
        default=lambda self: self.env.company.partner_id,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    event_type_id = fields.Many2one(
        'event.type', string='Category',
        readonly=False, states={'done': [('readonly', True)]})
    color = fields.Integer('Kanban Color Index')
    event_mail_ids = fields.One2many('event.mail', 'event_id', string='Mail Schedule', copy=True)

    # Seats and computation
    seats_max = fields.Integer(
        string='Maximum Attendees Number',
        readonly=True, states={'draft': [('readonly', False)], 'confirm': [('readonly', False)]},
        help="For each event you can define a maximum registration of seats(number of attendees), above this numbers the registrations are not accepted.")
    seats_availability = fields.Selection(
        [('limited', 'Limited'), ('unlimited', 'Unlimited')],
        'Maximum Attendees', required=True, default='unlimited')
    seats_min = fields.Integer(
        string='Minimum Attendees',
        help="For each event you can define a minimum reserved seats (number of attendees), if it does not reach the mentioned registrations the event can not be confirmed (keep 0 to ignore this rule)")
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
    registration_ids = fields.One2many(
        'event.registration', 'event_id', string='Attendees',
        readonly=False, states={'done': [('readonly', True)]})
    # Date fields
    date_tz = fields.Selection('_tz_get', string='Timezone', required=True, default=lambda self: self.env.user.tz or 'UTC')
    date_begin = fields.Datetime(
        string='Start Date', required=True,
        tracking=True, states={'done': [('readonly', True)]})
    date_end = fields.Datetime(
        string='End Date', required=True,
        tracking=True, states={'done': [('readonly', True)]})
    date_begin_located = fields.Char(string='Start Date Located', compute='_compute_date_begin_tz')
    date_end_located = fields.Char(string='End Date Located', compute='_compute_date_end_tz')
    is_one_day = fields.Boolean(compute='_compute_field_is_one_day')

    state = fields.Selection([
        ('draft', 'Unconfirmed'), ('cancel', 'Cancelled'),
        ('confirm', 'Confirmed'), ('done', 'Done')],
        string='Status', default='draft', readonly=True, required=True, copy=False,
        help="If event is created, the status is 'Draft'. If event is confirmed for the particular dates the status is set to 'Confirmed'. If the event is over, the status is set to 'Done'. If event is cancelled the status is set to 'Cancelled'.")
    auto_confirm = fields.Boolean(string='Autoconfirm Registrations')
    is_online = fields.Boolean('Online Event')
    address_id = fields.Many2one(
        'res.partner', string='Location',
        default=lambda self: self.env.company.partner_id,
        readonly=False, states={'done': [('readonly', True)]},
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        tracking=True)
    country_id = fields.Many2one('res.country', 'Country',  related='address_id.country_id', store=True, readonly=False)
    twitter_hashtag = fields.Char('Twitter Hashtag')
    description = fields.Html(
        string='Description', translate=html_translate, sanitize_attributes=False,
        readonly=False, states={'done': [('readonly', True)]})
    # badge fields
    badge_front = fields.Html(string='Badge Front')
    badge_back = fields.Html(string='Badge Back')
    badge_innerleft = fields.Html(string='Badge Inner Left')
    badge_innerright = fields.Html(string='Badge Inner Right')
    event_logo = fields.Html(string='Event Logo')

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
            state_field = {
                'draft': 'seats_unconfirmed',
                'open': 'seats_reserved',
                'done': 'seats_used',
            }
            query = """ SELECT event_id, state, count(event_id)
                        FROM event_registration
                        WHERE event_id IN %s AND state IN ('draft', 'open', 'done')
                        GROUP BY event_id, state
                    """
            self.env['event.registration'].flush(['event_id', 'state'])
            self._cr.execute(query, (tuple(self.ids),))
            for event_id, state, num in self._cr.fetchall():
                results[event_id][state_field[state]] += num

        for event in self:
            event.update(results.get(event._origin.id or event.id, base_vals))
            if event.seats_max > 0:
                event.seats_available = event.seats_max - (event.seats_reserved + event.seats_used)

    @api.depends('seats_unconfirmed', 'seats_reserved', 'seats_used')
    def _compute_seats_expected(self):
        for event in self:
            event.seats_expected = event.seats_unconfirmed + event.seats_reserved + event.seats_used

    @api.model
    def _tz_get(self):
        return [(x, x) for x in pytz.all_timezones]

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

    @api.depends('date_begin', 'date_end', 'date_tz')
    def _compute_field_is_one_day(self):
        for event in self:
            # Need to localize because it could begin late and finish early in
            # another timezone
            event = event.with_context(tz=event.date_tz)
            begin_tz = fields.Datetime.context_timestamp(event, event.date_begin)
            end_tz = fields.Datetime.context_timestamp(event, event.date_end)
            event.is_one_day = (begin_tz.date() == end_tz.date())

    @api.onchange('is_online')
    def _onchange_is_online(self):
        if self.is_online:
            self.address_id = False

    @api.onchange('event_type_id')
    def _onchange_type(self):
        if self.event_type_id:
            self.seats_min = self.event_type_id.default_registration_min
            self.seats_max = self.event_type_id.default_registration_max
            if self.event_type_id.default_registration_max:
                self.seats_availability = 'limited'

            if self.event_type_id.auto_confirm:
                self.auto_confirm = self.event_type_id.auto_confirm

            if self.event_type_id.use_hashtag:
                self.twitter_hashtag = self.event_type_id.default_hashtag

            if self.event_type_id.use_timezone:
                self.date_tz = self.event_type_id.default_timezone

            self.is_online = self.event_type_id.is_online

            if self.event_type_id.event_type_mail_ids:
                self.event_mail_ids = [(5, 0, 0)] + [
                    (0, 0, {
                        attribute_name: line[attribute_name] if not isinstance(line[attribute_name], models.BaseModel) else line[attribute_name].id
                        for attribute_name in self.env['event.type.mail']._get_event_mail_fields_whitelist()
                        })
                    for line in self.event_type_id.event_type_mail_ids]

    @api.constrains('seats_min', 'seats_max', 'seats_availability')
    def _check_seats_min_max(self):
        if any(event.seats_availability == 'limited' and event.seats_min > event.seats_max for event in self):
            raise ValidationError(_('Maximum attendees number should be greater than minimum attendees number.'))

    @api.constrains('seats_max', 'seats_available')
    def _check_seats_limit(self):
        if any(event.seats_availability == 'limited' and event.seats_max and event.seats_available < 0 for event in self):
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
    def create(self, vals):
        res = super(EventEvent, self).create(vals)
        if res.organizer_id:
            res.message_subscribe([res.organizer_id.id])
        if res.auto_confirm:
            res.button_confirm()
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

    def button_draft(self):
        self.write({'state': 'draft'})

    def button_cancel(self):
        if any('done' in event.mapped('registration_ids.state') for event in self):
            raise UserError(_("There are already attendees who attended this event. Please reset it to draft if you want to cancel this event."))
        self.registration_ids.write({'state': 'cancel'})
        self.state = 'cancel'

    def button_done(self):
        self.write({'state': 'done'})

    def button_confirm(self):
        self.write({'state': 'confirm'})

    def mail_attendees(self, template_id, force_send=False, filter_func=lambda self: self.state != 'cancel'):
        for event in self:
            for attendee in event.registration_ids.filtered(filter_func):
                self.env['mail.template'].browse(template_id).send_mail(attendee.id, force_send=force_send)

    def _is_event_registrable(self):
        return self.date_end > fields.Datetime.now()

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


class EventRegistration(models.Model):
    _name = 'event.registration'
    _description = 'Event Registration'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'name, create_date desc'

    # event
    origin = fields.Char(
        string='Source Document', readonly=True,
        help="Reference of the document that created the registration, for example a sales order")
    event_id = fields.Many2one(
        'event.event', string='Event', required=True,
        readonly=True, states={'draft': [('readonly', False)]})
    # attendee
    partner_id = fields.Many2one(
        'res.partner', string='Contact',
        states={'done': [('readonly', True)]})
    name = fields.Char(string='Attendee Name', index=True)
    email = fields.Char(string='Email')
    phone = fields.Char(string='Phone')
    mobile = fields.Char(string='Mobile')
    # organization
    date_open = fields.Datetime(string='Registration Date', readonly=True, default=lambda self: fields.Datetime.now())  # weird crash is directly now
    date_closed = fields.Datetime(string='Attended Date', readonly=True)
    event_begin_date = fields.Datetime(string="Event Start Date", related='event_id.date_begin', readonly=True)
    event_end_date = fields.Datetime(string="Event End Date", related='event_id.date_end', readonly=True)
    company_id = fields.Many2one(
        'res.company', string='Company', related='event_id.company_id',
        store=True, readonly=True, states={'draft': [('readonly', False)]})
    state = fields.Selection([
        ('draft', 'Unconfirmed'), ('cancel', 'Cancelled'),
        ('open', 'Confirmed'), ('done', 'Attended')],
        string='Status', default='draft', readonly=True, copy=False, tracking=True)

    @api.constrains('event_id', 'state')
    def _check_seats_limit(self):
        for registration in self:
            if registration.event_id.seats_availability == 'limited' and registration.event_id.seats_max and registration.event_id.seats_available < (1 if registration.state == 'draft' else 0):
                raise ValidationError(_('No more seats available for this event.'))

    def _check_auto_confirmation(self):
        if self._context.get('registration_force_draft'):
            return False
        if any(registration.event_id.state != 'confirm' or
               not registration.event_id.auto_confirm or
               (not registration.event_id.seats_available and registration.event_id.seats_availability == 'limited') for registration in self):
            return False
        return True

    @api.model
    def create(self, vals):
        registration = super(EventRegistration, self).create(vals)
        if registration._check_auto_confirmation():
            registration.sudo().confirm_registration()
        return registration

    @api.model
    def check_access_rights(self, operation, raise_exception=True):
        if operation == 'read' and not self.env.is_admin() and not self.user_has_groups('base.group_user'):
            if raise_exception:
                raise AccessError(_('Only internal users are allowed to read registrations.'))
            return False
        elif operation != 'read' and not self.env.is_admin() and not self.user_has_groups('event.group_event_user'):
            if raise_exception:
                raise AccessError(_('Only event users or managers are allowed to create or update registrations.'))
            return False
        return super(EventRegistration, self).check_access_rights(operation, raise_exception)

    @api.model
    def _prepare_attendee_values(self, registration):
        """ Method preparing the values to create new attendees based on a
        sales order line. It takes some registration data (dict-based) that are
        optional values coming from an external input like a web page. This method
        is meant to be inherited in various addons that sell events. """
        partner_id = registration.pop('partner_id', self.env.user.partner_id)
        event_id = registration.pop('event_id', False)
        data = {
            'name': registration.get('name', partner_id.name),
            'phone': registration.get('phone', partner_id.phone),
            'mobile': registration.get('mobile', partner_id.mobile),
            'email': registration.get('email', partner_id.email),
            'partner_id': partner_id.id,
            'event_id': event_id and event_id.id or False,
        }
        data.update({
            key: value for key, value in registration.items()
            if key in self._fields and key not in data and not self._fields[key].default
        })

        return data

    def do_draft(self):
        self.write({'state': 'draft'})

    def confirm_registration(self):
        self.write({'state': 'open'})

        # auto-trigger after_sub (on subscribe) mail schedulers, if needed
        onsubscribe_schedulers = self.event_id.event_mail_ids.filtered(
            lambda s: s.interval_type == 'after_sub')
        onsubscribe_schedulers.with_user(SUPERUSER_ID).execute()

    def button_reg_close(self):
        """ Close Registration """
        for registration in self:
            today = fields.Datetime.now()
            if registration.event_id.date_begin <= today and registration.event_id.state == 'confirm':
                registration.write({'state': 'done', 'date_closed': today})
            elif registration.event_id.state == 'draft':
                raise UserError(_("You must wait the event confirmation before doing this action."))
            else:
                raise UserError(_("You must wait the event starting day before doing this action."))

    def button_reg_cancel(self):
        self.write({'state': 'cancel'})

    @api.onchange('partner_id')
    def _onchange_partner(self):
        if self.partner_id:
            contact_id = self.partner_id.address_get().get('contact', False)
            if contact_id:
                contact = self.env['res.partner'].browse(contact_id)
                self.name = contact.name or self.name
                self.email = contact.email or self.email
                self.phone = contact.phone or self.phone
                self.mobile = contact.mobile or self.mobile

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
        return {r.id: {
            'partner_ids': [],
            'email_to': r.email,
            'email_cc': False}
            for r in self}

    def _message_post_after_hook(self, message, msg_vals):
        if self.email and not self.partner_id:
            # we consider that posting a message with a specified recipient (not a follower, a specific one)
            # on a document without customer means that it was created through the chatter using
            # suggested recipients. This heuristic allows to avoid ugly hacks in JS.
            new_partner = message.partner_ids.filtered(lambda partner: partner.email == self.email)
            if new_partner:
                self.search([
                    ('partner_id', '=', False),
                    ('email', '=', new_partner.email),
                    ('state', 'not in', ['cancel']),
                ]).write({'partner_id': new_partner.id})
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
            return _('on ') + format_datetime(self.env, self.event_begin_date, tz=self.event_id.date_tz, dt_format='medium')

    def summary(self):
        self.ensure_one()
        return {'information': []}

```

## File: models\event_mail.py

```python
# -*- coding: utf-8 -*-

from datetime import datetime
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, tools
from odoo.tools import exception_to_unicode
from odoo.tools.translate import _

import random
import logging
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

    @api.depends('event_id.state', 'event_id.date_begin', 'interval_type', 'interval_unit', 'interval_nbr')
    def _compute_scheduled_date(self):
        for mail in self:
            if mail.event_id.state not in ['confirm', 'done']:
                mail.scheduled_date = False
            else:
                if mail.interval_type == 'after_sub':
                    date, sign = mail.event_id.create_date, 1
                elif mail.interval_type == 'before_event':
                    date, sign = mail.event_id.date_begin, -1
                else:
                    date, sign = mail.event_id.date_end, 1
                mail.scheduled_date = date + _INTERVALS[mail.interval_unit](sign * mail.interval_nbr)

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
                mail.mail_registration_ids.filtered(lambda reg: reg.scheduled_date and reg.scheduled_date <= now).execute()
            else:
                # Do not send emails if the mailing was scheduled before the event but the event is over
                if not mail.mail_sent and (mail.interval_type != 'before_event' or mail.event_id.date_end > now) and mail.notification_type == 'mail':
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
                subject = _("WARNING: Event Scheduler Error for event: %s" % event.name)
                body = _("""Event Scheduler for:
                              - Event: %s (%s)
                              - Scheduled: %s
                              - Template: %s (%s)

                            Failed with error:
                              - %s

                            You receive this email because you are:
                              - the organizer of the event,
                              - or the responsible of the event,
                              - or the last writer of the template."""
                         % (event.name, event.id, scheduler.scheduled_date, template.name, template.id, ex_s))
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
        schedulers = self.search([('done', '=', False), ('scheduled_date', '<=', datetime.strftime(fields.datetime.now(), tools.DEFAULT_SERVER_DATETIME_FORMAT))])
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
                if autocommit:
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
        for mail in self:
            if mail.registration_id.state in ['open', 'done'] and not mail.mail_sent and mail.scheduler_id.notification_type == 'mail':
                organizer = mail.scheduler_id.event_id.organizer_id
                company = self.env.company
                author = self.env.ref('base.user_root')
                if organizer.email:
                    author = organizer
                elif company.email:
                    author = company.partner_id
                elif self.env.user.email:
                    author = self.env.user
                
                email_values = {
                    'email_from': author.email_formatted,
                    'author_id': author.id,
                }
                mail.scheduler_id.template_id.send_mail(mail.registration_id.id, email_values=email_values)
                mail.write({'mail_sent': True})

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

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models

class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    module_event_sale = fields.Boolean("Tickets")
    module_website_event_track = fields.Boolean("Tracks and Agenda")
    module_website_event_questions = fields.Boolean("Registration Survey")
    module_event_barcode = fields.Boolean("Barcode")
    module_website_event_sale = fields.Boolean("Online Ticketing")

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models

class ResPartner(models.Model):
    _inherit = 'res.partner'

    event_count = fields.Integer("Events", compute='_compute_event_count', help="Number of events the partner has participated.")

    def _compute_event_count(self):
        self.event_count = 0
        if not self.user_has_groups('event.group_event_user'):
            return
        for partner in self:
            partner.event_count = self.env['event.event'].search_count([('registration_ids.partner_id', 'child_of', partner.ids)])

    def action_event_view(self):
        action = self.env.ref('event.action_event_view').read()[0]
        action['context'] = {}
        action['domain'] = [('registration_ids.partner_id', 'child_of', self.ids)]
        return action

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event
from . import event_mail
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

    <report
        id="report_event_registration_badge"
        model="event.registration"
        string="Registration Badge"
        report_type="qweb-pdf"
        name="event.event_registration_report_template_badge"
        file="event.event_registration_report_template_badge"
        paperformat="event.paperformat_euro_lowmargin"
        print_report_name="'Registration Event - %s' % (object.name or 'Attendee').replace('/','')"
    />

     <report
        id="report_event_event_badge"
        model="event.event"
        string="Event Badge"
        report_type="qweb-html"
        name="event.event_event_report_template_badge"
        file="event.event_event_report_template_badge"
        menu="False"
        paperformat="event.paperformat_euro_lowmargin"/>

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
            <field name="global" eval="True"/>
            <field name="domain_force">['|',
                                            ('company_id', '=', False),
                                            ('company_id', 'in', company_ids),
                                        ]
            </field>
        </record>
        <record model="ir.rule" id="event_registration_company_rule">
            <field name="name">Event/Registration: multi-company</field>
            <field name="model_id" ref="model_event_registration"/>
            <field name="global" eval="True"/>
            <field name="domain_force">['|',
                                            ('company_id', '=', False),
                                            ('company_id', 'in', company_ids),
                                        ]
            </field>
        </record>
        <record model="ir.rule" id="event_registration_portal">
            <field name="name">Event/Registration: Portal</field>
            <field name="model_id" ref="model_event_registration"/>
            <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
            <field name="domain_force">['|', ('email', '=', user.partner_id.email), ('partner_id', '=', user.partner_id.id)]
            </field>
        </record>
    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_event_type,event.type,model_event_type,event.group_event_user,1,0,0,0
access_event_type_manager,event.type manager,model_event_type,event.group_event_manager,1,1,1,1
access_event_event_portal,event.event.portal,model_event_event,,1,0,0,0
access_event_event_user,event.event.user,model_event_event,event.group_event_user,1,0,0,0
access_event_event_manager,event.event.manager,model_event_event,event.group_event_manager,1,1,1,1
access_event_registration,event.registration,model_event_registration,event.group_event_user,1,1,1,1
access_event_registration_employee,event.registration,model_event_registration,base.group_user,1,0,0,0
access_event_registration_portal,event.registration,model_event_registration,base.group_portal,1,0,0,0
access_event_mail,event.mail,model_event_mail,event.group_event_user,1,0,0,0
access_event_mail_manager,event.mail manager,model_event_mail,event.group_event_manager,1,1,1,1
access_event_mail_registration,event.mail.registration,model_event_mail_registration,event.group_event_user,1,0,0,0
access_event_mail_registration_manager,event.mail.registration.manager,model_event_mail_registration,event.group_event_manager,1,1,1,1
access_event_type_mail_event_user,event.type.mail.event.user,model_event_type_mail,event.group_event_user,1,0,0,0
access_event_type_mail_event_manager,event.type.mail.event.manager,model_event_type_mail,event.group_event_manager,1,1,1,1

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
    </data>
</odoo>

```

## File: views\event_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>

        <!-- Main menu items -->
        <menuitem name="Events" id="event_main_menu" sequence="65" groups="event.group_event_user" web_icon="event,static/description/icon.png"/>
        <menuitem parent="event_main_menu" id="menu_reporting_events" sequence="99" groups="event.group_event_manager" name="Reporting"/>

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
                                    <field name="is_online"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="is_online"/>
                                    <div class="row">
                                        <div class="col-lg-8 mt16 text-muted">
                                            Online events like webinars do not require a specific location
                                            and are hosted online.
                                        </div>
                                    </div>
                                </div>
                            </div>
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
                                        <div class="col-lg-9">
                                            <field name="event_type_mail_ids">
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
                            <div class="col-12 col-lg-6 o_setting_box">
                                <div class="o_setting_left_pane">
                                    <field name="use_hashtag"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="use_hashtag"/>
                                    <div class="row">
                                        <div class="col-lg-8 mt16" attrs="{'invisible': [('use_hashtag', '=', False)]}">
                                            <label for="default_hashtag"/>
                                            <field name="default_hashtag"
                                                attrs="{'required': [('use_hashtag', '=', True)]}"/>
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
                                                <label for="default_registration_min"/> <field name="default_registration_min"/>
                                            </div>
                                            <div>
                                                <label for="default_registration_max"/> <field name="default_registration_max"/>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <h2>Attendees</h2>
                        <div class="row mt16 o_settings_container" name="event_type_attendees">
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
                <tree string="Event Category">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                </tree>
            </field>
        </record>

        <record id="event_type_view_search" model="ir.ui.view">
            <field name="name">event.type.search</field>
            <field name="model">event.type</field>
            <field name="arch" type="xml">
                <search string="Event Categories">
                    <field name="name"/>
                    <filter string="Online Event" name="online_event" help="Online Event" domain="[('is_online', '=', True)]"/>
                </search>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_event_type">
            <field name="name">Event Categories</field>
            <field name="res_model">event.type</field>
        </record>

        <menuitem id="menu_event_configuration" name="Configuration" parent="event_main_menu"
            sequence="100"/>
        <menuitem name="Event Categories" id="menu_event_type" action="action_event_type" parent="menu_event_configuration"/>

        <!-- EVENT.REGISTRATION ACTIONS -->
        <record id="act_event_registration_from_event" model="ir.actions.act_window">
            <field name="res_model">event.registration</field>
            <field name="name">Attendees</field>
            <field name="view_mode">kanban,tree,form,calendar,graph</field>
            <field name="context">{'search_default_event_id': active_id, 'default_event_id': active_id, 'search_default_expected': True}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Add a new attendee
                </p>
            </field>
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
                <form string="Events">
                    <header>
                        <button string="Confirm Event" name="button_confirm" states="draft" type="object" class="oe_highlight" groups="base.group_user"/>
                        <button string="Finish Event" name="button_done" states="confirm" type="object" class="oe_highlight" groups="base.group_user"/>
                        <button string="Set To Draft" name="button_draft" states="cancel,done" type="object" groups="base.group_user"/>
                        <button string="Cancel Event" name="button_cancel" type="object" groups="base.group_user" attrs="{'invisible': ['|', ('seats_expected', '!=', 0), ('state', 'not in', ['draft', 'confirm'])]}"/>
                        <button string="Cancel Event" name="button_cancel" type="object" groups="base.group_user" confirm="Are you sure you want to cancel this event? All the linked attendees will be cancelled as well." attrs="{'invisible': ['|', ('seats_expected', '=', 0), ('state', 'not in', ['draft', 'confirm'])]}"/>
                        <field name="state" widget="statusbar" statusbar_visible="draft,confirm,done"/>
                    </header>
                    <sheet>
                        <div class="oe_button_box" name="button_box" groups="base.group_user">
                            <button name="%(event.act_event_registration_from_event)d"
                                    type="action"
                                    class="oe_stat_button"
                                    icon="fa-users"
                                    help="Register with this event">
                                <field name="seats_expected" widget="statinfo" string="Attendees"/>
                            </button>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <div class="oe_title">
                            <label for="name" class="oe_edit_only"/>
                            <h1><field name="name" placeholder="Event Name"/></h1>
                            <field name="is_online" groups="base.group_no_one"/>
                            <label for="is_online" string="Online" groups="base.group_no_one"/>
                        </div>
                        <group>
                            <group>
                                <field name="active" invisible="1"/>
                                <field name="organizer_id"/>
                                <field name="address_id"
                                    attrs="{'invisible': [('is_online', '=', True)]}"
                                    context="{'show_address': 1}"
                                    options='{"always_reload": True}'/>
                                <field name="company_id" groups="base.group_multi_company"/>
                                <field name="user_id"/>
                            </group>
                            <group>
                                <field name="event_type_id" options="{'no_create':True}"/>
                                <label for="twitter_hashtag"/>
                                <div>
                                    <span class="oe_inline"># </span>
                                    <field name="twitter_hashtag" nolabel="1" class="oe_inline"/>
                                </div>
                                <field name="auto_confirm" groups="base.group_no_one"/>
                            </group>
                            <group>
                                <label for="date_begin" string="Date"/>
                                <div class="o_row">
                                    <field name="date_begin" widget="daterange" nolabel="1" class="oe_inline" options="{'related_end_date': 'date_end'}"/>
                                    <i class="fa fa-long-arrow-right mx-2" aria-label="Arrow icon" title="Arrow"/>
                                    <field name="date_end" widget="daterange" nolabel="1" class="oe_inline" options="{'related_start_date': 'date_begin'}"/>
                                </div>
                                <field name="date_tz"/>
                            </group>
                            <group>
                                <field name="seats_min"/>
                                <label for="seats_availability"/>
                                <div>
                                    <field name="seats_availability" widget='radio'/>
                                    <span  attrs="{'invisible': [('seats_availability', '=', 'unlimited')]}" class="oe_read_only">
                                        to
                                    </span>
                                    <field name="seats_max" attrs="{'invisible': [('seats_availability', '=', 'unlimited')], 'required': [('seats_availability', '=', 'limited')]}"/>
                                </div>
                            </group>
                        </group>
                        <notebook>
                            <page string="Communication" name="event_communication">
                                <field name="event_mail_ids">
                                    <tree string="Communication" editable="bottom">
                                        <field name="sequence" widget="handle"/>
                                        <field name="notification_type" invisible="1"/>
                                        <field name="template_id" attrs="{'required': [('notification_type', '=', 'mail')]}"/>
                                        <field name="interval_nbr" attrs="{'readonly':[('interval_unit','=','now')]}"/>
                                        <field name="interval_unit"/>
                                        <field name="interval_type"/>
                                        <field name="done"/>
                                    </tree>
                                </field>
                            </page>
                        </notebook>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" widget="mail_followers" groups="base.group_user"/>
                        <field name="activity_ids" widget="mail_activity"/>
                        <field name="message_ids" widget="mail_thread"/>
                    </div>
                </form>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_tree">
            <field name="name">event.event.tree</field>
            <field name="model">event.event</field>
            <field name="arch" type="xml">
                <tree string="Events" decoration-bf="message_needaction==True" decoration-danger="(seats_min and seats_min&gt;seats_reserved) or (seats_max and seats_max&lt;seats_reserved)" decoration-muted="state=='cancel'">
                    <field name="name"/>
                    <field name="event_type_id"/>
                    <field name="date_begin"/>
                    <field name="date_end"/>
                    <field name="seats_reserved" sum="Total"/>
                    <field name="seats_min"/>
                    <field name="seats_max" invisible="1"/>
                    <field name="user_id"/>
                    <field name="state"/>
                    <field name="message_needaction" invisible="1"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="activity_exception_decoration" widget="activity_exception"/>
                </tree>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_kanban">
            <field name="name">event.event.kanban</field>
            <field name="model">event.event</field>
            <field name="arch" type="xml">
                <kanban class="o_event_kanban_view">
                    <field name="user_id"/>
                    <field name="state"/>
                    <field name="country_id"/>
                    <field name="date_begin"/>
                    <field name="date_end"/>
                    <field name="seats_unconfirmed"/>
                    <field name="seats_reserved"/>
                    <field name="seats_used"/>
                    <field name="seats_expected"/>
                    <field name="color"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="{{!selection_mode ? 'oe_kanban_color_' + kanban_getcolor(record.color.raw_value) : ''}} oe_kanban_card oe_kanban_global_click">
                                <div class="o_dropdown_kanban dropdown">

                                    <a class="dropdown-toggle o-no-caret btn" role="button" data-toggle="dropdown" data-display="static" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                                        <span class="fa fa-ellipsis-v"/>
                                    </a>
                                    <div class="dropdown-menu" role="menu">
                                        <t t-if="widget.deletable">
                                            <a role="menuitem" type="delete" class="dropdown-item">Delete</a>
                                        </t>
                                        <ul class="oe_kanban_colorpicker" data-field="color" role="menu"/>
                                    </div>
                                </div>
                                <div class="o_event_left">
                                    <div class="o_day"><t t-esc="record.date_begin.raw_value.getDate()"/></div>
                                    <div>
                                        <t t-esc="moment(record.date_begin.raw_value).format('MMM')"/>
                                        <t t-esc="record.date_begin.raw_value.getFullYear()"/>
                                    </div>
                                    <div><t t-esc="moment(record.date_begin.raw_value).format('LT')"/></div>
                                </div>
                                <div class="o_event_right">
                                    <h4 class="o_kanban_record_title"><field name="name"/></h4>
                                    <div>
                                        <t t-if="record.country_id.raw_value"> <b class="o_kanban_record_subtitle"> <field name="country_id"/> </b> <br/> </t>
                                        <b><i class="fa fa-clock-o"/>
                                        To</b> <t t-esc="moment(record.date_end.raw_value).format('lll')"/>
                                    </div>
                                    <h4>
                                        <a name="%(act_event_registration_from_event)d" type="action">
                                            <t t-esc="record.seats_expected.raw_value"/> Expected attendees
                                        </a>
                                        <t t-if="(record.seats_reserved.raw_value + record.seats_used.raw_value) > 0 "><br/>
                                            <a name="%(event_event_action_pivot)d" type="action">
                                                <t t-esc="record.seats_reserved.raw_value + record.seats_used.raw_value"/> Confirmed attendees
                                            </a>
                                        </t>
                                    </h4>
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
                    <field name="state"/>
                    <field name="seats_expected"/>
                    <field name="seats_reserved"/>
                    <field name="seats_used"/>
                    <field name="seats_unconfirmed"/>
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
                    <filter string="My Events" name="myevents" help="My Events" domain="[('user_id', '=', uid)]"/>
                    <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction', '=', True)]"/>
                    <separator/>
                    <filter string="Confirmed" name="confirm" domain="[('state', '=', 'confirm')]" help="Confirmed events"/>
                    <filter string="Unconfirmed" name="draft" domain="[('state', '=', 'draft')]" help="Events in New state"/>
                    <separator/>
                    <filter string="Upcoming/Running" name="upcoming"
                        domain="['&amp;', ('state', '!=', 'cancel'), ('date_end', '&gt;=', datetime.datetime.combine(context_today(), datetime.time(0,0,0)))]" help="Upcoming events from today" />
                    <filter string="Online Events" name="online_events" help="Online Events" domain="[('is_online', '=', True)]"/>
                    <separator/>
                    <filter string="Start Date" name="start_date" date="date_begin"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <separator/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('activity_ids.date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('activity_ids.date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('activity_ids.date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <group expand="0" string="Group By">
                        <filter string="Responsible" name="responsible" context="{'group_by': 'user_id'}"/>
                        <filter string="Event Category" name="event_type_id" context="{'group_by': 'event_type_id'}"/>
                        <filter string="Status" name="status" context="{'group_by': 'state'}"/>
                        <filter string="Start Date" name="date_begin" domain="[]" context="{'group_by': 'date_begin'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="event_event_view_pivot" model="ir.ui.view" >
            <field name="name">event.event.view.pivot</field>
            <field name="model">event.event</field>
            <field name="arch" type="xml">
                <pivot string="Event">
                    <field name="name" type="row"/>
                    <field name="seats_reserved" type="measure"/>
                </pivot>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_event_view">
           <field name="name">Events</field>
           <field name="type">ir.actions.act_window</field>
           <field name="res_model">event.event</field>
           <field name="view_mode">kanban,calendar,tree,form,pivot</field>
           <field name="context">{"search_default_upcoming":1}</field>
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

        <menuitem name="Events" id="menu_event_event" action="action_event_view" parent="event.event_main_menu" />
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
                <tree string="Registration" decoration-bf="message_needaction==True">
                    <field name="create_date"/>
                    <field name="partner_id"/>
                    <field name="name"/>
                    <field name="email"/>
                    <field name="event_id" />
                    <field name="state"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="message_needaction" invisible="1"/>
                    <button name="confirm_registration" string="Confirm Registration" states="draft" type="object" icon="fa-check text-success"/>
                    <button name="button_reg_close" string="Attended the Event" states="open" type="object" icon="fa-level-down text-success"/>
                    <button name="button_reg_cancel" string="Cancel Registration" states="draft,open" type="object" icon="fa-times-circle text-danger"/>
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
                        <button name="confirm_registration" string="Confirm" states="draft" type="object" class="oe_highlight"/>
                        <button name="button_reg_close" string="Attended" states="open" type="object" class="oe_highlight"/>
                        <button string="Set To Unconfirmed" name="do_draft" states="cancel,done" type="object" />
                        <button name="button_reg_cancel" string="Cancel Registration" states="draft,open" type="object"/>
                        <field name="state" nolabel="1" colspan="2" widget="statusbar" statusbar_visible="draft,open,done"/>
                    </header>
                    <sheet string="Registration">
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
                                <field name="date_open" groups="base.group_no_one"/>
                                <field name="date_closed" groups="base.group_no_one"/>
                            </group>
                        </group>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" widget="mail_followers"/>
                        <field name="activity_ids" widget="mail_activity"/>
                        <field name="message_ids" widget="mail_thread" options="{'post_refresh': 'recipients'}"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="event_registration_view_kanban" model="ir.ui.view">
            <field name="name">event.registration.kanban</field>
            <field name="model">event.registration</field>
            <field name="priority">10</field>
            <field name="arch" type="xml">
                <kanban class="o_event_attendee_kanban">
                    <field name="name"/>
                    <field name="partner_id"/>
                    <field name="state"/>
                    <field name="email"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div class="oe_kanban_global_click">
                                <div class="oe_kanban_details">
                                    <div class="o_kanban_record_top">
                                        <div class="o_kanban_record_headings">
                                            <strong class="o_kanban_record_title">
                                                <field name="name"/>
                                            </strong>
                                        </div>
                                        <div class="float-right">
                                            <a class="btn btn-sm btn-success" string="Confirm Registration" name="confirm_registration" type="object" states="draft" role="button">
                                                <i class="fa fa-check" role="img" aria-label="Confirm button" title="Confirm Registration"/>
                                            </a>
                                            <a class="btn btn-sm btn-success" string="Attended the Event" name="button_reg_close" type="object" states="open" role="button">
                                                <i class="fa fa-level-down" role="img" aria-label="Attended button" title="Attended the Event"/>
                                            </a>
                                            <a class="btn btn-sm btn-danger" string="Cancel Registration" name="button_reg_cancel" type="object" states="draft,open" role="button">
                                                <i class="fa fa-times" role="img" aria-label="Cancel button" title="Cancel Registration"/>
                                            </a>
                                        </div>
                                    </div>
                                    <ul>
                                        <li><field name="partner_id"/></li>
                                        <li><field name="email"/></li>
                                    </ul>
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
                    <field name="event_id"/>
                    <field name="name"/>
                </calendar>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_registration_pivot">
            <field name="name">event.registration.pivot</field>
            <field name="model">event.registration</field>
            <field name="arch" type="xml">
                <pivot string="Registration" display_quantity="True">
                    <field name="event_id" type="row"/>
                </pivot>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_registration_graph">
            <field name="name">event.registration.graph</field>
            <field name="model">event.registration</field>
            <field name="arch" type="xml">
                <graph string="Registration">
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
                    <field name="name" string="Participant" filter_domain="['|', '|', ('name', 'ilike', self), ('email', 'ilike', self), ('origin', 'ilike', self)]"/>
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
                        domain="[('activity_ids.date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('activity_ids.date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('activity_ids.date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <group expand="0" string="Group By">
                        <filter string="Partner" name="partner" domain="[]" context="{'group_by':'partner_id'}"/>
                        <filter string="Event" name="group_event" domain="[]" context="{'group_by':'event_id'}"/>
                        <filter string="Status" name="status" domain="[]" context="{'group_by':'state'}"/>
                        <filter string="Registration Date" name="createmonth" domain="[]" context="{'group_by': 'create_date:month'}"/>
                   </group>
                </search>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_registration">
          <field name="name">Attendees</field>
          <field name="res_model">event.registration</field>
          <field name="domain"></field>
          <field name="view_mode">pivot,graph,tree,form</field>
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
                            <page string="Registration Mails">
                                <field name="mail_registration_ids">
                                    <tree string="Registration mail" editable="bottom">
                                        <field name="registration_id"/>
                                        <field name="scheduled_date"/>
                                        <field name="mail_sent"/>
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
                    <field name="mail_sent"/>
                    <field name="done"/>
                </tree>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_event_mail">
            <field name="name">Events Mail Schedulers</field>
            <field name="res_model">event.mail</field>
        </record>

        <menuitem name="Mail Schedulers" id="menu_event_mail_schedulers" action="action_event_mail" parent="menu_event_configuration" groups="base.group_no_one"/>

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
                        <div class="row mt16 o_settings_container">
                            <div class="col-12 col-lg-6 o_setting_box" title="Add a navigation menu to your event web pages with schedule, tracks, a track proposal form, etc.">
                                <div class="o_setting_left_pane">
                                    <field name="module_website_event_track"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label string="Schedule &amp; Tracks" for="module_website_event_track"/>
                                    <div class="text-muted">
                                        Manage &amp; publish a schedule with tracks
                                    </div>
                                </div>
                            </div>
                        </div>
                        <h2>Registration</h2>
                        <div class="row mt16 o_settings_container">
                            <div class="col-12 col-lg-6 o_setting_box">
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
                        <div class="row mt16 o_settings_container">
                            <div class="col-12 col-lg-6 o_setting_box">
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

## File: wizard\event_confirm.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api


class event_confirm(models.TransientModel):
    """Event Confirmation"""
    _name = "event.confirm"
    _description = 'Event Confirmation'

    def confirm(self):
        events = self.env['event.event'].browse(self._context.get('event_ids', []))
        events.do_confirm()
        return {'type': 'ir.actions.act_window_close'}

```

## File: wizard\event_confirm_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="view_event_confirm" model="ir.ui.view">
            <field name="name">Event Confirmation</field>
            <field name="model">event.confirm</field>
            <field name="arch" type="xml">
              <form string="Event Confirmation">
                  <span class="o_form_label">Warning: This Event has not reached its Minimum Registration Limit. Are you sure you want to confirm it?</span>
                  <footer>
                      <button name="confirm" string="Confirm Anyway" type="object" class="btn-primary"/>
                      <button string="Cancel" class="btn-secondary" special="cancel" />
                  </footer>
            </form>
            </field>
        </record>

        <record id="action_event_confirm" model="ir.actions.act_window">
            <field name="name">Event Confirmation</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">event.confirm</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="view_event_confirm"/>
            <field name="context">{'record_id' : active_id}</field>
            <field name="target">new</field>
        </record>

    </data>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from . import event_confirm

```

