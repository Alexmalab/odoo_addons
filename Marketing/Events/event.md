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
    'version': '1.7',
    'website': 'https://www.odoo.com/app/events',
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
        'views/event_mail_views.xml',
        'views/event_registration_views.xml',
        'views/event_type_views.xml',
        'views/event_event_views.xml',
        'views/event_stage_views.xml',
        'report/event_event_templates.xml',
        'report/event_event_reports.xml',
        'data/ir_cron_data.xml',
        'data/mail_template_data.xml',
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
    'assets': {
        'web.assets_backend': [
            'event/static/src/scss/event.scss',
            'event/static/src/icon_selection_field/icon_selection_field.js',
            'event/static/src/icon_selection_field/icon_selection_field.xml',
        ],
        'web.assets_common': [
            'event/static/src/js/tours/**/*',
        ],
        'web.assets_frontend': [
            'event/static/src/js/tours/**/*',
        ],
        'web.report_assets_common': [
            '/event/static/src/scss/event_foldable_badge_report.scss',
            '/event/static/src/scss/event_full_page_ticket_report.scss',
        ],
        'web.report_assets_pdf': [
            '/event/static/src/scss/event_full_page_ticket_report_pdf.scss',
        ],
    },
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
        lang = request.context.get('lang', request.env.user.lang)
        if request.env.user._is_public():
            lang = request.httprequest.cookies.get('frontend_lang')
        event = event.with_context(lang=lang)
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

## File: data\event_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Event Categories -->
        <record id="event_type_data_ticket" model="event.type">
            <field name="name">Ticketing</field>
            <field name="auto_confirm" eval="False"/>
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
        <field name="date_tz">America/Los_Angeles</field>
        <field name="event_type_id" ref="event_type_0"/>
        <field name="stage_id" ref="event_stage_booked"/>
        <field name="tag_ids" eval="[(4, ref('event.event_tag_category_1_tag_1')), (4, ref('event.event_tag_category_2_tag_1'))]"/>
        <field name="ticket_instructions" type="html">
<div class="text-center fw-bold py-3">Important ticket information</div>
<ul>
    <li>Please come <b>at least</b> 30 minutes before the beginning of the event.</li>
    <li>Tickets can be printed or scanned directly from your phone.</li>
    <li>If you don't have this ticket, you will <b>not</b> be allowed entry!</li>
</ul>
        </field>
    </record>
    <record id="event_0_ticket_0" model="event.event.ticket">
        <field name="name">Free</field>
        <field name="description">Free entrance, no food !</field>
        <field name="event_id" ref="event.event_0"/>
        <field name="start_sale_datetime" eval="(DateTime.today() + timedelta(days=5)).strftime('%Y-%m-%d 00:00:00')"/>
        <field name="end_sale_datetime" eval="(DateTime.today() + timedelta(days=10)).strftime('%Y-%m-%d 23:00:00')"/>
        <field name="seats_max">0</field>
    </record>
    <record id="event_0_ticket_1" model="event.event.ticket">
        <field name="name">Standard</field>
        <field name="description">For only 10, you gain access to catering. Yum yum.</field>
        <field name="event_id" ref="event.event_0"/>
        <field name="start_sale_datetime" eval="(DateTime.today() + timedelta(days=5)).strftime('%Y-%m-%d 00:00:00')"/>
        <field name="end_sale_datetime" eval="(DateTime.today() + timedelta(days=10)).strftime('%Y-%m-%d 23:00:00')"/>
        <field name="seats_max">50</field>
    </record>
    <record id="event_0_ticket_2" model="event.event.ticket">
        <field name="name">VIP</field>
        <field name="description">You are truly among the best.</field>
        <field name="event_id" ref="event.event_0"/>
        <field name="start_sale_datetime" eval="(DateTime.today() + timedelta(days=5)).strftime('%Y-%m-%d 00:00:00')"/>
        <field name="end_sale_datetime" eval="(DateTime.today() + timedelta(days=10)).strftime('%Y-%m-%d 23:00:00')"/>
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
        <field name="end_sale_datetime" eval="(DateTime.today() + timedelta(90)).strftime('%Y-%m-%d 23:00:00')"/>
        <field name="seats_max">50</field>
    </record>
    <record id="event_2_ticket_2" model="event.event.ticket">
        <field name="name">VIP</field>
        <field name="event_id" ref="event.event_2"/>
        <field name="end_sale_datetime" eval="(DateTime.today() + timedelta(60)).strftime('%Y-%m-%d 23:00:00')"/>
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
        <field name="template_ref" eval="'mail.template,%i' % ref('event.event_subscription')"/>
    </record>

    <record id="event.event_3" model="event.event">
        <field name="name">Live Music Festival</field>
        <field name="user_id" ref="base.user_demo"/>
        <field name="date_begin" eval="(DateTime.today()+ timedelta(days=130)).strftime('%Y-%m-%d 20:15:00')"/>
        <field name="date_end" eval="(DateTime.today()+ timedelta(days=133)).strftime('%Y-%m-%d 00:30:00')"/>
        <field name="date_tz">Europe/London</field>
        <field name="event_type_id" ref="event_type_0"/>
        <field name="address_id" ref="event.res_partner_location_1"/>
        <field name="stage_id" ref="event_stage_announced"/>
        <field name="tag_ids" eval="[(4, ref('event.event_tag_category_1_tag_3')), (4, ref('event.event_tag_category_2_tag_2'))]"/>
    </record>
    <record id="event_3_ticket_0" model="event.event.ticket">
        <field name="name">Standard</field>
        <field name="event_id" ref="event.event_3"/>
        <field name="end_sale_datetime" eval="(DateTime.today() + timedelta(days=20)).strftime('%Y-%m-%d 23:00:00')"/>
        <field name="seats_max">1200</field>
    </record>
    <record id="event_3_ticket_1" model="event.event.ticket">
        <field name="name">VIP</field>
        <field name="event_id" ref="event.event_3"/>
        <field name="end_sale_datetime" eval="(DateTime.today() + timedelta(days=20)).strftime('%Y-%m-%d 23:00:00')"/>
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
        <field name="template_ref" eval="'mail.template,%i' % ref('event.event_subscription')"/>
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
        <field name="date_tz">America/Los_Angeles</field>
        <field name="event_type_id" ref="event_type_1"/>
        <field name="stage_id" ref="event_stage_done"/>
        <field name="kanban_state">done</field>
        <field name="tag_ids" eval="[(4, ref('event.event_tag_category_1_tag_4')), (4, ref('event.event_tag_category_2_tag_1'))]"/>
    </record>
    <record id="event_4_ticket_0" model="event.event.ticket">
        <field name="name">General Admission</field>
        <field name="event_id" ref="event.event_4"/>
        <field name="end_sale_datetime" eval="(DateTime.today() - timedelta(30)).strftime('%Y-%m-%d 23:00:00')"/>
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
    <div class="bg-light rounded-end border-start border-secondary p-3 mb-5" style="border-start-width: 3px !important;">
        <p class="mb-1">This event is fully online and FREE, if you have paid for tickets, you should get a refund.<br/>
        It will require a good Internet connection to get the best video quality.</p>
    </div>
</div>
        </field>
    </record>
    <record id="event_7_ticket_1" model="event.event.ticket">
        <field name="name">Standard</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="end_sale_datetime" eval="(DateTime.now() + timedelta(days=2)).strftime('%Y-%m-%d 15:00:00')"/>
    </record>
    <record id="event_7_ticket_2" model="event.event.ticket">
        <field name="name">VIP</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="end_sale_datetime" eval="(DateTime.now() + timedelta(days=2)).strftime('%Y-%m-%d 15:00:00')"/>
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
    </record>
    <record id="event_type_1" model="event.type">
        <field name="name">Training</field>
        <field name="auto_confirm" eval="False"/>
    </record>
    <record id="event_type_2" model="event.type">
        <field name="name">Sport</field>
        <field name="auto_confirm" eval="False"/>
        <field name="default_timezone">America/Los_Angeles</field>
    </record>
    <record id="event_type_data_conference" model="event.type">
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
        <field name="create_date" eval="DateTime.now() - relativedelta(days=2)"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="event_ticket_id" ref="event.event_0_ticket_1"/>
        <field name="partner_id" ref="base.res_partner_address_1"/>
    </record>
    <record id="event_registration_0_1" model="event.registration">
        <field name="create_date" eval="DateTime.now() - relativedelta(days=2)"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="event_ticket_id" ref="event.event_0_ticket_1"/>
        <field name="partner_id" ref="base.res_partner_address_2"/>
    </record>
    <record id="event_registration_0_2" model="event.registration">
        <field name="create_date" eval="DateTime.now() - relativedelta(days=2)"/>
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
        <field name="create_date" eval="DateTime.now() - relativedelta(days=0.5)"/>
        <field name="event_id" ref="event.event_2"/>
        <field name="event_ticket_id" ref="event.event_2_ticket_1"/>
        <field name="partner_id" ref="base.res_partner_address_1"/>
    </record>
    <record id="event_registration_2_1" model="event.registration">
        <field name="create_date" eval="DateTime.now() - relativedelta(days=0.5)"/>
        <field name="event_id" ref="event.event_2"/>
        <field name="event_ticket_id" ref="event.event_2_ticket_1"/>
        <field name="partner_id" ref="base.res_partner_address_2"/>
    </record>
    <record id="event_registration_2_2" model="event.registration">
        <field name="create_date" eval="DateTime.now() - relativedelta(days=0.5)"/>
        <field name="event_id" ref="event.event_2"/>
        <field name="event_ticket_id" ref="event.event_2_ticket_2"/>
        <field name="name">Piers Morgan</field>
        <field name="email">piersm@test.example.com</field>
        <field name="partner_id" eval="False"/>
    </record>
    <record id="event_registration_2_3" model="event.registration">
        <field name="create_date" eval="DateTime.now() - relativedelta(days=1)"/>
        <field name="event_id" ref="event.event_2"/>
        <field name="event_ticket_id" ref="event.event_2_ticket_1"/>
        <field name="partner_id" ref="base.res_partner_address_3"/>
    </record>
    <record id="event_registration_2_4" model="event.registration">
        <field name="create_date" eval="DateTime.now() - relativedelta(days=1)"/>
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
        <field name="create_date" eval="DateTime.now() - relativedelta(days=8)"/>
        <field name="event_id" ref="event.event_4"/>
        <field name="event_ticket_id" ref="event.event_4_ticket_0"/>
        <field name="partner_id" ref="base.res_partner_address_7"/>
    </record>
    <record id="event_registration_4_1" model="event.registration">
        <field name="create_date" eval="DateTime.now() - relativedelta(days=7)"/>
        <field name="event_id" ref="event.event_4"/>
        <field name="event_ticket_id" ref="event.event_4_ticket_0"/>
        <field name="partner_id" ref="base.res_partner_address_13"/>
    </record>
    <record id="event_registration_4_2" model="event.registration">
        <field name="create_date" eval="DateTime.now() - relativedelta(days=7)"/>
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
        context="{'install_mode' : True}"
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

## File: data\ir_cron_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!-- Event Mail Scheduler-->
    <record model="ir.cron" forcecreate="True" id="event_mail_scheduler">
        <field name="name">Event: Mail Scheduler</field>
        <field name="model_id" ref="model_event_mail"/>
        <field name="state">code</field>
        <field name="code">model.schedule_communications(autocommit=True)</field>
        <field name="user_id" ref="base.user_root"/>
        <field name="interval_number">1</field>
        <field name="interval_type">hours</field>
        <field name="numbercall">-1</field>
        <field name="doall" eval="False" />
    </record>
</data></odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">

        <record id="event_registration_mail_template_badge" model="mail.template">
            <field name="name">Event: Registration Badge</field>
            <field name="model_id" ref="event.model_event_registration"/>
            <field name="subject">Your badge for {{ object.event_id.name }}</field>
            <field name="email_from">{{ (object.event_id.organizer_id.email_formatted or object.event_id.user_id.email_formatted or '') }}</field>
            <field name="email_to">{{ (object.email and '"%s" &lt;%s&gt;' % (object.name, object.email) or object.partner_id.email_formatted or '') }}</field>
            <field name="description">Sent automatically to someone after they registered to an event</field>
            <field name="body_html" type="html">
<div>
    Dear <t t-out="object.name or ''">Oscar Morgan</t>,<br/>
    Thank you for your inquiry.<br/>
    Here is your badge for the event <t t-out="object.event_id.name or ''">OpenWood Collection Online Reveal</t>.<br/>
    If you have any questions, please let us know.
    <br/><br/>
    Thank you,
    <t t-if="object.event_id.user_id.signature">
        <br />
        <t t-out="object.event_id.user_id.signature or ''">--<br/>Mitchell Admin</t>
    </t>
</div></field>
            <field name="report_template" ref="action_report_event_registration_foldable_badge"/>
            <field name="report_name">Foldable Badge - {{ (object.event_id.name or 'Event').replace('/','_') }}</field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <record id="event_subscription" model="mail.template">
            <field name="name">Event: Registration Confirmation</field>
            <field name="model_id" ref="event.model_event_registration"/>
            <field name="subject">Your registration at {{ object.event_id.name }}</field>
            <field name="email_from">{{ (object.event_id.organizer_id.email_formatted or object.event_id.user_id.email_formatted or '') }}</field>
            <field name="email_to">{{ (object.email and '"%s" &lt;%s&gt;' % (object.name, object.email) or object.partner_id.email_formatted or '') }}</field>
            <field name="description">Sent to attendees after registering to an event</field>
            <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<t t-set="date_begin" t-value="format_datetime(object.event_id.date_begin, tz='UTC', dt_format=&quot;yyyyMMdd'T'HHmmss'Z'&quot;)"/>
<t t-set="date_end" t-value="format_datetime(object.event_id.date_end, tz='UTC', dt_format=&quot;yyyyMMdd'T'HHmmss'Z'&quot;)"/>
<t t-set="is_online" t-value="'is_published' in object.event_id and object.event_id.is_published"/>
<t t-set="event_organizer" t-value="object.event_id.organizer_id"/>
<t t-set="event_address" t-value="object.event_id.address_id"/>
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 16px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table width="590" border="0" cellpadding="0" cellspacing="0" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Your registration</span><br/>
                    <span style="font-size: 20px; font-weight: bold;">
                        <t t-out="object.name or ''">Oscar Morgan</t>
                    </span>
                </td><td valign="middle" align="right">
                    <t t-if="is_online">
                        <a t-att-href="object.event_id.website_url"
                            style="padding: 8px 12px; font-size: 12px; color: #FFFFFF; text-decoration: none !important; font-weight: 400; background-color: #875A7B; border: 0px solid #875A7B; border-radius:3px">
                            View Event
                        </a>
                    </t>
                    <t t-else="">
                        <img t-att-src="'/logo.png?company=%s' % object.company_id.id" style="padding: 0px; margin: 0px; height: auto; width: 80px;" t-att-alt="'%s' % object.company_id.name"/>
                    </t>
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
                        Hello <t t-out="object.name or ''">Oscar Morgan</t>,<br/>
                        We are happy to confirm your registration to the event
                        <t t-if="is_online">
                            <a t-att-href="object.event_id.website_url" style="color:#875A7B;text-decoration:none;" t-out="object.event_id.name or ''">OpenWood Collection Online Reveal</a>
                        </t>
                        <t t-else="">
                            <strong t-out="object.event_id.name or ''">OpenWood Collection Online Reveal</strong>
                        </t>
                        for attendee <t t-out="object.name or ''">Oscar Morgan</t>.
                    </div>
                    <div>
                        <br />
                        <strong>Add this event to your calendar</strong>
                        <a t-attf-href="https://www.google.com/calendar/render?action=TEMPLATE&amp;text={{ object.event_id.name }}&amp;dates={{ date_begin }}/{{ date_end }}&amp;location={{ location }}" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;" target="new"><img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> Google</a>
                        <a t-attf-href="/event/{{ slug(object.event_id) }}/ics" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;"><img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> iCal/Outlook</a>
                        <a t-attf-href="https://calendar.yahoo.com/?v=60&amp;view=d&amp;type=20&amp;title={{ object.event_id.name }}&amp;in_loc={{ location }}&amp;st={{ format_datetime(object.event_id.date_begin, tz='UTC', dt_format='yyyyMMdd\'T\'HHmmss') }}&amp;et={{ format_datetime(object.event_id.date_end, tz='UTC', dt_format='yyyyMMdd\'T\'HHmmss') }}" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;" target="new">
                            <img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> Yahoo
                        </a>
                        <br /><br />
                    </div>
                    <div>
                        See you soon,<br/>
                        <span style="color: #454748;">
                        -- <br/>
                        <t t-if="event_organizer">
                            <t t-out="event_organizer.name or ''">YourCompany</t>
                        </t>
                        <t t-else="">
                            The <t t-out="object.event_id.name or ''">OpenWood Collection Online Reveal</t> Team
                        </t>
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
                                <div><strong>From</strong> <t t-out="object.event_id.date_begin_located or ''">May 4, 2021, 7:00:00 AM</t></div>
                                <div><strong>To</strong> <t t-out="object.event_id.date_end_located or ''">May 6, 2021, 5:00:00 PM</t></div>
                                <div style="font-size:12px;color:#9e9e9e"><i>(<t t-out="object.event_id.date_tz or ''">Europe/Brussels</t>)</i></div>
                            </td>
                            <td style="vertical-align:top;">
                                <t t-if="event_address">
                                    <img src="/web_editor/font_to_img/61505/rgb(81,81,102)/34" style="padding:4px;max-width:inherit;" height="34" alt=""/>
                                </t>
                            </td>
                            <td style="padding: 0px 10px 0px 10px;width:50%;vertical-align:top;">
                                <t t-if="event_address">
                                    <t t-set="location" t-value="''"/>
                                    <t t-if="object.event_id.address_id.name">
                                        <div t-out="object.event_id.address_id.name or ''">Teksa SpA</div>
                                    </t>
                                    <t t-if="object.event_id.address_id.street">
                                        <div t-out="object.event_id.address_id.street or ''">Puerto Madero 9710</div>
                                        <t t-set="location" t-value="object.event_id.address_id.street"/>
                                    </t>
                                    <t t-if="object.event_id.address_id.street2">
                                        <div t-out="object.event_id.address_id.street2 or ''">Of A15, Santiago (RM)</div>
                                        <t t-set="location" t-value="'%s, %s' % (location, object.event_id.address_id.street2)"/>
                                    </t>
                                    <div>
                                    <t t-if="object.event_id.address_id.city">
                                        <t t-out="object.event_id.address_id.city or ''">Pudahuel</t>,
                                        <t t-set="location" t-value="'%s, %s' % (location, object.event_id.address_id.city)"/>
                                    </t>
                                    <t t-if="object.event_id.address_id.state_id.name">
                                        <t t-out="object.event_id.address_id.state_id.name or ''">C1</t>,
                                        <t t-set="location" t-value="'%s, %s' % (location, object.event_id.address_id.state_id.name)"/>
                                    </t>
                                    <t t-if="object.event_id.address_id.zip">
                                        <t t-out="object.event_id.address_id.zip or ''">98450</t>
                                        <t t-set="location" t-value="'%s, %s' % (location, object.event_id.address_id.zip)"/>
                                    </t>
                                    </div>
                                    <t t-if="object.event_id.address_id.country_id.name">
                                        <div t-out="object.event_id.address_id.country_id.name or ''">Argentina</div>
                                        <t t-set="location" t-value="'%s, %s' % (location, object.event_id.address_id.country_id.name)"/>
                                    </t>
                                </t>
                            </td>
                        </tr>
                    </table>
                </td></tr>
                <tr><td style="text-align:center;">
                    <t t-if="event_organizer">
                        <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                    </t>
                </td></tr>

                <tr><td valign="top" style="font-size: 14px;">
                    <!-- CONTACT ORGANIZER -->
                    <t t-if="event_organizer">
                        <div>
                            <span style="font-weight:300;margin:10px 0px">Questions about this event?</span>
                            <div>Please contact the organizer:</div>
                            <ul>
                                <li><t t-out="event_organizer.name or ''">YourCompany</t></li>
                                <t t-if="event_organizer.email">
                                    <li>Mail: <a t-attf-href="mailto:{{ event_organizer.email }}" style="text-decoration:none;color:#875A7B;" t-out="event_organizer.email or ''">info@yourcompany.com</a></li>
                                </t>
                                <t t-if="event_organizer.phone">
                                    <li>Phone: <t t-out="event_organizer.phone or ''">+1 650-123-4567</t></li>
                                </t>
                            </ul>
                        </div>
                    </t>
                </td></tr>
                <tr><td style="text-align:center;">
                    <!-- CONTACT ORGANIZER SEPARATION -->
                    <t t-if="is_online or event_address">
                        <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                    </t>
                </td></tr>

                <tr><td valign="top" style="font-size: 14px;">
                    <!-- PWA MARKGETING -->
                    <t t-if="is_online">
                        <div>
                            <strong>Get the best mobile experience.</strong>
                            <a href="/event">Install our mobile app</a>
                        </div>
                    </t>
                </td></tr>
                <tr><td style="text-align:center;">
                    <!-- PWA MARKGETING SEPARATION-->
                    <t t-if="is_online and event_address">
                        <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                    </t>
                </td></tr>

                <tr><td valign="top" style="font-size: 14px;">
                    <!-- GOOGLE MAPS LINK -->
                    <t t-if="event_address and location">
                        <table style="width:100%;"><tr><td>
                            <div>
                                <i class="fa fa-map-marker"/>
                                <a t-attf-href="https://maps.google.com/maps?q={{ location }}" target="new">
                                    See location on Google Maps
                                </a>
                            </div>
                        </td></tr></table>
                    </t>
                </td></tr>
            </table>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- FOOTER BY -->
<tr><td align="center" style="min-width: 590px;">
    <t t-if="object.company_id">
        <table width="590" border="0" cellpadding="0" cellspacing="0" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
        <tr><td style="text-align: center; font-size: 14px;">
            Sent by <a target="_blank" t-attf-href="{{ object.company_id.website }}" style="color: #875A7B;" t-out="object.company_id.name or ''">YourCompany</a>
            <t t-if="is_online">
                <br />
                Discover <a href="/event" style="color:#875A7B;">all our events</a>.
            </t>
        </td></tr>
        </table>
    </t>
</td></tr>
</table>
            </field>
            <field name="report_template" ref="action_report_event_registration_full_page_ticket"/>
            <field name="report_name">Full Page Ticket - {{ (object.event_id.name or 'Event').replace('/','') }}</field>
            <field name="lang">{{ object.partner_id.lang }}</field>
        </record>

        <record id="event_reminder" model="mail.template">
            <field name="name">Event: Reminder</field>
            <field name="model_id" ref="event.model_event_registration"/>
            <field name="subject">{{ object.event_id.name }}: {{ object.get_date_range_str() }}</field>
            <field name="email_from">{{ (object.event_id.organizer_id.email_formatted or object.event_id.user_id.email_formatted or '') }}</field>
            <field name="email_to">{{ (object.email and '"%s" &lt;%s&gt;' % (object.name, object.email) or object.partner_id.email_formatted or '') }}</field>
            <field name="description">Sent automatically to attendees if there is a reminder defined on the event</field>
            <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<t t-set="date_begin" t-value="format_datetime(object.event_id.date_begin, tz='UTC', dt_format=&quot;yyyyMMdd'T'HHmmss'Z'&quot;)"/>
<t t-set="date_end" t-value="format_datetime(object.event_id.date_end, tz='UTC', dt_format=&quot;yyyyMMdd'T'HHmmss'Z'&quot;)"/>
<t t-set="is_online" t-value="'is_published' in object.event_id and object.event_id.is_published"/>
<t t-set="event_organizer" t-value="object.event_id.organizer_id"/>
<t t-set="event_address" t-value="object.event_id.address_id"/>
<table border="0" cellpadding="0" cellspacing="0"  width="590" style="padding: 16px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table width="590" border="0" cellpadding="0" cellspacing="0" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Your registration</span><br/>
                    <span style="font-size: 20px; font-weight: bold;" t-out="object.name or ''">Oscar Morgan</span>
                </td><td valign="middle" align="right">
                    <t t-if="is_online">
                        <a t-attf-href="{{ object.event_id.website_url }}"
                            style="padding: 8px 12px; font-size: 12px; color: #FFFFFF; text-decoration: none !important; font-weight: 400; background-color: #875A7B; border: 0px solid #875A7B; border-radius:3px">
                            View Event
                        </a>
                    </t>
                    <t t-else="">
                        <img t-att-src="'/logo.png?company=%s' % object.company_id.id" style="padding: 0px; margin: 0px; height: auto; width: 80px;" t-att-alt="'%s' % object.company_id.name"/>
                    </t>
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
                        Hello <t t-out="object.name or ''">Oscar Morgan</t>,<br/>
                        We are excited to remind you that the event
                        <t t-if="is_online">
                            <a t-att-href="object.event_id.website_url" style="color:#875A7B;text-decoration:none;" t-out="object.event_id.name or ''">OpenWood Collection Online Reveal</a>
                        </t>
                        <t t-else="">
                            <strong t-out="object.event_id.name or ''">OpenWood Collection Online Reveal</strong>
                        </t>
                        is starting <strong t-out="object.get_date_range_str() or ''">today</strong>.
                    </div>
                    <div>
                        <br />
                        <strong>Add this event to your calendar</strong>
                        <a t-attf-href="https://www.google.com/calendar/render?action=TEMPLATE&amp;text={{ object.event_id.name }}&amp;dates={{ date_begin }}/{{ date_end }}&amp;location={{ location }}" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;" target="new"><img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> Google</a>
                        <a t-attf-href="/event/{{ slug(object.event_id) }}/ics" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;"><img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> iCal/Outlook</a>
                        <a t-attf-href="https://calendar.yahoo.com/?v=60&amp;view=d&amp;type=20&amp;title={{ object.event_id.name }}&amp;in_loc={{ location }}&amp;st={{ format_datetime(object.event_id.date_begin, tz='UTC', dt_format='yyyyMMdd\'T\'HHmmss') }}&amp;et={{ format_datetime(object.event_id.date_end, tz='UTC', dt_format='yyyyMMdd\'T\'HHmmss') }}" style="padding:3px 5px;border:1px solid #875A7B;color:#875A7B;text-decoration:none;border-radius:3px;" target="new">
                            <img src="/web_editor/font_to_img/61525/rgb(135,90,123)/16" style="vertical-align:middle;" height="16" alt=""/> Yahoo
                        </a>
                        <br /><br />
                    </div>
                    <div>
                        We confirm your registration and hope to meet you there,<br/>
                        <span style="color: #454748;">
                        -- <br/>
                        <t t-if="event_organizer">
                            <t t-out="event_organizer.name or ''">YourCompany</t>
                        </t>
                        <t t-else="">
                            The <t t-out="object.event_id.name or ''">OpenWood Collection Online Reveal</t> Team
                        </t>
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
                                <div><strong>From</strong> <t t-out="object.event_id.date_begin_located or ''">May 4, 2021, 7:00:00 AM</t></div>
                                <div><strong>To</strong> <t t-out="object.event_id.date_end_located or ''">May 6, 2021, 5:00:00 PM</t></div>
                                <div style="font-size:12px;color:#9e9e9e"><i><t t-out="object.event_id.date_tz or ''">Europe/Brussels</t></i></div>
                            </td>
                            <td style="vertical-align:top;">
                                <t t-if="event_address">
                                    <img src="/web_editor/font_to_img/61505/rgb(81,81,102)/34" style="padding:4px;max-width:inherit;" height="34" alt=""/>
                                </t>
                            </td>
                            <td style="padding: 0px 10px 0px 10px;width:50%;vertical-align:top;">
                                <t t-if="event_address">
                                    <t t-set="location" t-value="''"/>
                                    <t t-if="object.event_id.address_id.name">
                                        <div t-out="object.event_id.address_id.name or ''">Teksa SpA</div>
                                    </t>
                                    <t t-if="object.event_id.address_id.street">
                                        <div t-out="object.event_id.address_id.street or ''">Puerto Madero 9710</div>
                                        <t t-set="location" t-value="object.event_id.address_id.street"/>
                                    </t>
                                    <t t-if="object.event_id.address_id.street2">
                                        <div t-out="object.event_id.address_id.street2 or ''">Of A15, Santiago (RM)</div>
                                        <t t-set="location" t-value="'%s, %s' % (location, object.event_id.address_id.street2)"/>
                                    </t>
                                    <div>
                                    <t t-if="object.event_id.address_id.city">
                                        <t t-out="object.event_id.address_id.city or ''">Pudahuel</t>,
                                        <t t-set="location" t-value="'%s, %s' % (location, object.event_id.address_id.city)"/>
                                    </t>
                                    <t t-if="object.event_id.address_id.state_id.name">
                                        <t t-out="object.event_id.address_id.state_id.name or ''">C1</t>,
                                        <t t-set="location" t-value="'%s, %s' % (location, object.event_id.address_id.state_id.name)"/>
                                    </t>
                                    <t t-if="object.event_id.address_id.zip">
                                        <t t-out="object.event_id.address_id.zip or ''">98450</t>
                                        <t t-set="location" t-value="'%s, %s' % (location, object.event_id.address_id.zip)"/>
                                    </t>
                                    </div>
                                    <t t-if="object.event_id.address_id.country_id.name">
                                        <div t-out="object.event_id.address_id.country_id.name or ''">Argentina</div>
                                        <t t-set="location" t-value="'%s, %s' % (location, object.event_id.address_id.country_id.name)"/>
                                    </t>
                                </t>
                            </td>
                        </tr>
                    </table>
                </td></tr>
                <tr><td style="text-align:center;">
                    <t t-if="event_organizer">
                        <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                    </t>
                </td></tr>

                <tr><td valign="top" style="font-size: 14px;">
                    <!-- CONTACT ORGANIZER -->
                    <t t-if="event_organizer">
                        <div>
                            <span style="font-weight:300;margin:10px 0px">Questions about this event?</span>
                            <div>Please contact the organizer:</div>
                            <ul>
                                <li t-out="event_organizer.name or ''">YourCompany</li>
                                <t t-if="event_organizer.email">
                                    <li>Mail: <a t-attf-href="mailto:{{ event_organizer.email }}" style="text-decoration:none;color:#875A7B;" t-out="event_organizer.email or ''"></a></li>
                                </t>
                                <t t-if="event_organizer.phone">
                                    <li>Phone: <t t-out="event_organizer.phone or ''"></t></li>
                                </t>
                            </ul>
                        </div>
                    </t>
                </td></tr>
                <tr><td style="text-align:center;">
                    <!-- CONTACT ORGANIZER SEPARATION -->
                    <hr t-if="is_online or event_address" width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>

                <tr><td valign="top" style="font-size: 14px;">
                    <!-- PWA MARKGETING -->
                    <div t-if="is_online">
                        <strong>Get the best mobile experience.</strong>
                        <a href="/event">Install our mobile app</a>
                    </div>
                </td></tr>
                <tr><td style="text-align:center;">
                    <!-- PWA MARKGETING SEPARATION-->
                    <hr t-if="is_online and event_address" width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>

                <tr><td valign="top" style="font-size: 14px;">
                    <!-- GOOGLE MAPS LINK -->
                    <table t-if="event_address" style="width:100%;"><tr><td>
                        <div>
                            <i class="fa fa-map-marker"/>
                            <a t-attf-href="https://maps.google.com/maps?q={{ location }}" target="new">
                                See location on Google Maps
                            </a>
                        </div>
                    </td></tr></table>
                </td></tr>
            </table>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- FOOTER BY -->
<tr><td align="center" style="min-width: 590px;">
    <table t-if="object.company_id" width="590" border="0" cellpadding="0" cellspacing="0" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
      <tr><td style="text-align: center; font-size: 14px;">
        Sent by <a target="_blank" t-attf-href="{{ object.company_id.website }}" style="color: #875A7B;" t-out="object.company_id.name or ''">YourCompany</a>
        <t t-if="'website_url' in object.event_id and object.event_id.website_url">
            <br />
            Discover <a href="/event" style="color:#875A7B;">all our events</a>.
        </t>
      </td></tr>
    </table>
</td></tr>
</table>
            </field>
            <field name="lang">{{ object.partner_id.lang }}</field>
        </record>

    </data>
</odoo>

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

from datetime import timedelta

from odoo import _, api, Command, fields, models
from odoo.addons.base.models.res_partner import _tz_get
from odoo.tools import format_datetime, is_html_empty
from odoo.exceptions import UserError, ValidationError
from odoo.tools.misc import formatLang
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

    def _default_event_mail_type_ids(self):
        return [(0, 0,
                 {'notification_type': 'mail',
                  'interval_nbr': 0,
                  'interval_unit': 'now',
                  'interval_type': 'after_sub',
                  'template_ref': 'mail.template, %i' % self.env.ref('event.event_subscription').id,
                 }),
                (0, 0,
                 {'notification_type': 'mail',
                  'interval_nbr': 1,
                  'interval_unit': 'hours',
                  'interval_type': 'before_event',
                  'template_ref': 'mail.template, %i' % self.env.ref('event.event_reminder').id,
                 }),
                (0, 0,
                 {'notification_type': 'mail',
                  'interval_nbr': 3,
                  'interval_unit': 'days',
                  'interval_type': 'before_event',
                  'template_ref': 'mail.template, %i' % self.env.ref('event.event_reminder').id,
                 })]

    name = fields.Char('Event Template', required=True, translate=True)
    note = fields.Html(string='Note')
    sequence = fields.Integer()
    # tickets
    event_type_ticket_ids = fields.One2many('event.type.ticket', 'event_type_id', string='Tickets')
    tag_ids = fields.Many2many('event.tag', string="Tags")
    # registration
    has_seats_limitation = fields.Boolean('Limited Seats')
    seats_max = fields.Integer(
        'Maximum Registrations', compute='_compute_seats_max',
        readonly=False, store=True,
        help="It will select this default maximum value when you choose this event")
    auto_confirm = fields.Boolean(
        'Automatically Confirm Registrations', default=True,
        help="Events and registrations will automatically be confirmed "
             "upon creation, easing the flow for simple events.")
    default_timezone = fields.Selection(
        _tz_get, string='Timezone', default=lambda self: self.env.user.tz or 'UTC')
    # communication
    event_type_mail_ids = fields.One2many(
        'event.type.mail', 'event_type_id', string='Mail Schedule',
        default=_default_event_mail_type_ids)
    # ticket reports
    ticket_instructions = fields.Html('Ticket Instructions', translate=True,
        help="This information will be printed on your tickets.")

    @api.depends('has_seats_limitation')
    def _compute_seats_max(self):
        for template in self:
            if not template.has_seats_limitation:
                template.seats_max = 0


class EventEvent(models.Model):
    """Event"""
    _name = 'event.event'
    _description = 'Event'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'date_begin'

    @api.model
    def default_get(self, fields_list):
        result = super().default_get(fields_list)
        if 'date_begin' in fields_list and 'date_begin' not in result:
            now = fields.Datetime.now()
            # Round the datetime to the nearest half hour (e.g. 08:17 => 08:30 and 08:37 => 09:00)
            result['date_begin'] = now.replace(second=0, microsecond=0) + timedelta(minutes=-now.minute % 30)
        if 'date_end' in fields_list and 'date_end' not in result and result.get('date_begin'):
            result['date_end'] = result['date_begin'] + timedelta(days=1)
        return result

    def _get_default_stage_id(self):
        return self.env['event.stage'].search([], limit=1)

    def _default_description(self):
        # avoid template branding with rendering_bundle=True
        return self.env['ir.ui.view'].with_context(rendering_bundle=True) \
            ._render_template('event.event_default_descripton')

    def _default_event_mail_ids(self):
        return self.env['event.type']._default_event_mail_type_ids()

    name = fields.Char(string='Event', translate=True, required=True)
    note = fields.Html(string='Note', store=True, compute="_compute_note", readonly=False)
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
    kanban_state = fields.Selection([('normal', 'In Progress'), ('done', 'Done'), ('blocked', 'Blocked')], default='normal', copy=False)
    kanban_state_label = fields.Char(
        string='Kanban State Label', compute='_compute_kanban_state_label',
        store=True, tracking=True)
    stage_id = fields.Many2one(
        'event.stage', ondelete='restrict', default=_get_default_stage_id,
        group_expand='_read_group_stage_ids', tracking=True, copy=False)
    legend_blocked = fields.Char(related='stage_id.legend_blocked', string='Kanban Blocked Explanation', readonly=True)
    legend_done = fields.Char(related='stage_id.legend_done', string='Kanban Valid Explanation', readonly=True)
    legend_normal = fields.Char(related='stage_id.legend_normal', string='Kanban Ongoing Explanation', readonly=True)
    # Seats and computation
    seats_max = fields.Integer(
        string='Maximum Attendees',
        compute='_compute_seats_max', readonly=False, store=True,
        help="For each event you can define a maximum registration of seats(number of attendees), above this numbers the registrations are not accepted.")
    seats_limited = fields.Boolean('Limit Attendees', required=True, compute='_compute_seats_limited',
                                   precompute=True, readonly=False, store=True)
    seats_reserved = fields.Integer(
        string='Number of Registrations',
        store=False, readonly=True, compute='_compute_seats')
    seats_available = fields.Integer(
        string='Available Seats',
        store=False, readonly=True, compute='_compute_seats')
    seats_unconfirmed = fields.Integer(
        string='Unconfirmed Registrations',
        store=False, readonly=True, compute='_compute_seats')
    seats_used = fields.Integer(
        string='Number of Attendees',
        store=False, readonly=True, compute='_compute_seats')
    seats_expected = fields.Integer(
        string='Number of Expected Attendees',
        store=False, readonly=True, compute='_compute_seats')
    # Registration fields
    auto_confirm = fields.Boolean(
        string='Autoconfirmation', compute='_compute_auto_confirm', readonly=False, store=True,
        help='Autoconfirm Registrations. Registrations will automatically be confirmed upon creation.')
    registration_ids = fields.One2many('event.registration', 'event_id', string='Attendees')
    event_ticket_ids = fields.One2many(
        'event.event.ticket', 'event_id', string='Event Ticket', copy=True,
        compute='_compute_event_ticket_ids', readonly=False, store=True)
    event_registrations_started = fields.Boolean(
        'Registrations started', compute='_compute_event_registrations_started',
        help="registrations have started if the current datetime is after the earliest starting date of tickets."
    )
    event_registrations_open = fields.Boolean(
        'Registration open', compute='_compute_event_registrations_open', compute_sudo=True,
        help="Registrations are open if:\n"
        "- the event is not ended\n"
        "- there are seats available on event\n"
        "- the tickets are sellable (if ticketing is used)")
    event_registrations_sold_out = fields.Boolean(
        'Sold Out', compute='_compute_event_registrations_sold_out', compute_sudo=True,
        help='The event is sold out if no more seats are available on event. If ticketing is used and all tickets are sold out, the event will be sold out.')
    start_sale_datetime = fields.Datetime(
        'Start sale date', compute='_compute_start_sale_date',
        help='If ticketing is used, contains the earliest starting sale date of tickets.')

    # Date fields
    date_tz = fields.Selection(
        _tz_get, string='Display Timezone', required=True,
        compute='_compute_date_tz', precompute=True, readonly=False, store=True,
        help="Indicates the timezone in which the event dates/times will be displayed on the website.")
    date_begin = fields.Datetime(string='Start Date', required=True, tracking=True,
        help="When the event is scheduled to take place (expressed in your local timezone on the form view).")
    date_end = fields.Datetime(string='End Date', required=True, tracking=True)
    date_begin_located = fields.Char(string='Start Date Located', compute='_compute_date_begin_tz')
    date_end_located = fields.Char(string='End Date Located', compute='_compute_date_end_tz')
    is_ongoing = fields.Boolean('Is Ongoing', compute='_compute_is_ongoing', search='_search_is_ongoing')
    is_one_day = fields.Boolean(compute='_compute_field_is_one_day')
    is_finished = fields.Boolean(compute='_compute_is_finished', search='_search_is_finished')
    # Location and communication
    address_id = fields.Many2one(
        'res.partner', string='Venue', default=lambda self: self.env.company.partner_id.id,
        tracking=True, domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    address_search = fields.Many2one(
        'res.partner', string='Address', compute='_compute_address_search', search='_search_address_search')
    address_inline = fields.Char(
        string='Venue (formatted for one line uses)', compute='_compute_address_inline',
        compute_sudo=True)
    country_id = fields.Many2one(
        'res.country', 'Country', related='address_id.country_id', readonly=False, store=True)
    # ticket reports
    ticket_instructions = fields.Html('Ticket Instructions', translate=True,
        compute='_compute_ticket_instructions', store=True, readonly=False,
        help="This information will be printed on your tickets.")

    @api.depends('stage_id', 'kanban_state')
    def _compute_kanban_state_label(self):
        for event in self:
            if event.kanban_state == 'normal':
                event.kanban_state_label = event.stage_id.legend_normal
            elif event.kanban_state == 'blocked':
                event.kanban_state_label = event.stage_id.legend_blocked
            else:
                event.kanban_state_label = event.stage_id.legend_done

    @api.depends('seats_max', 'registration_ids.state', 'registration_ids.active')
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
                        WHERE event_id IN %s AND state IN ('draft', 'open', 'done') AND active = true
                        GROUP BY event_id, state
                    """
            self.env['event.registration'].flush_model(['event_id', 'state', 'active'])
            self._cr.execute(query, (tuple(self.ids),))
            res = self._cr.fetchall()
            for event_id, state, num in res:
                results[event_id][state_field[state]] = num

        # compute seats_available and expected
        for event in self:
            event.update(results.get(event._origin.id or event.id, base_vals))
            if event.seats_max > 0:
                event.seats_available = event.seats_max - (event.seats_reserved + event.seats_used)

            event.seats_expected = event.seats_unconfirmed + event.seats_reserved + event.seats_used

    @api.depends('date_tz', 'start_sale_datetime')
    def _compute_event_registrations_started(self):
        for event in self:
            event = event._set_tz_context()
            if event.start_sale_datetime:
                current_datetime = fields.Datetime.context_timestamp(event, fields.Datetime.now())
                start_sale_datetime = fields.Datetime.context_timestamp(event, event.start_sale_datetime)
                event.event_registrations_started = (current_datetime >= start_sale_datetime)
            else:
                event.event_registrations_started = True

    @api.depends('date_tz', 'event_registrations_started', 'date_end', 'seats_available', 'seats_limited', 'seats_max',
                 'event_ticket_ids.sale_available')
    def _compute_event_registrations_open(self):
        """ Compute whether people may take registrations for this event

          * event.date_end -> if event is done, registrations are not open anymore;
          * event.start_sale_datetime -> lowest start date of tickets (if any; start_sale_datetime
            is False if no ticket are defined, see _compute_start_sale_date);
          * any ticket is available for sale (seats available) if any;
          * seats are unlimited or seats are available;
        """
        for event in self:
            event = event._set_tz_context()
            current_datetime = fields.Datetime.context_timestamp(event, fields.Datetime.now())
            date_end_tz = event.date_end.astimezone(pytz.timezone(event.date_tz or 'UTC')) if event.date_end else False
            event.event_registrations_open = event.event_registrations_started and \
                (date_end_tz >= current_datetime if date_end_tz else True) and \
                (not event.seats_limited or not event.seats_max or event.seats_available) and \
                (not event.event_ticket_ids or any(ticket.sale_available for ticket in event.event_ticket_ids))

    @api.depends('event_ticket_ids.start_sale_datetime')
    def _compute_start_sale_date(self):
        """ Compute the start sale date of an event. Currently lowest starting sale
        date of tickets if they are used, of False. """
        for event in self:
            start_dates = [ticket.start_sale_datetime for ticket in event.event_ticket_ids if not ticket.is_expired]
            event.start_sale_datetime = min(start_dates) if start_dates and all(start_dates) else False

    @api.depends('event_ticket_ids.sale_available', 'seats_available', 'seats_limited')
    def _compute_event_registrations_sold_out(self):
        """Note that max seats limits for events and sum of limits for all its tickets may not be
        equal to enable flexibility.
        E.g. max 20 seats for ticket A, 20 seats for ticket B
            * With max 20 seats for the event
            * Without limit set on the event (=40, but the customer didn't explicitly write 40)
        """
        for event in self:
            event.event_registrations_sold_out = (
                (event.seats_limited and event.seats_max and not event.seats_available)
                or (event.event_ticket_ids and all(ticket.is_sold_out for ticket in event.event_ticket_ids))
            )

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
            raise UserError(_('This operator is not supported'))
        if not isinstance(value, bool):
            raise UserError(_('Value should be True or False (not %s)') % value)
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

    @api.depends('date_end')
    def _compute_is_finished(self):
        for event in self:
            if not event.date_end:
                event.is_finished = False
                continue
            event = event._set_tz_context()
            current_datetime = fields.Datetime.context_timestamp(event, fields.Datetime.now())
            datetime_end = fields.Datetime.context_timestamp(event, event.date_end)
            event.is_finished = datetime_end <= current_datetime

    def _search_is_finished(self, operator, value):
        if operator not in ['=', '!=']:
            raise ValueError(_('This operator is not supported'))
        if not isinstance(value, bool):
            raise ValueError(_('Value should be True or False (not %s)'), value)
        now = fields.Datetime.now()
        if (operator == '=' and value) or (operator == '!=' and not value):
            domain = [('date_end', '<=', now)]
        else:
            domain = [('date_end', '>', now)]
        event_ids = self.env['event.event']._search(domain)
        return [('id', 'in', event_ids)]

    @api.depends('event_type_id')
    def _compute_date_tz(self):
        for event in self:
            if event.event_type_id.default_timezone:
                event.date_tz = event.event_type_id.default_timezone
            if not event.date_tz:
                event.date_tz = self.env.user.tz or 'UTC'

    @api.depends('address_id')
    def _compute_address_search(self):
        for event in self:
            event.address_search = event.address_id

    def _search_address_search(self, operator, value):
        if operator != 'ilike' or not isinstance(value, str):
            raise NotImplementedError(_('Operation not supported.'))

        address_ids = self.env['res.partner']._search([
            '|', '|', '|', '|', '|',
            ('street', 'ilike', value),
            ('street2', 'ilike', value),
            ('city', 'ilike', value),
            ('zip', 'ilike', value),
            ('state_id', 'ilike', value),
            ('country_id', 'ilike', value),
        ])

        return [('address_id', 'in', address_ids)]

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
                event.event_mail_ids = self._default_event_mail_ids()
                continue

            # lines to keep: those with already sent emails or registrations
            mails_to_remove = event.event_mail_ids.filtered(
                lambda mail: not(mail._origin.mail_done) and not(mail._origin.mail_registration_ids)
            )
            command = [Command.unlink(mail.id) for mail in mails_to_remove]

            # lines to add: those which do not have the exact copy available in lines to keep
            if event.event_type_id.event_type_mail_ids:
                mails_to_keep_vals = {mail._prepare_event_mail_values() for mail in event.event_mail_ids - mails_to_remove}
                for mail in event.event_type_id.event_type_mail_ids:
                    mail_values = mail._prepare_event_mail_values()
                    if mail_values not in mails_to_keep_vals:
                        command.append(Command.create(mail_values._asdict()))
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
        (start_sale_datetime computation) so ensure result to avoid cache miss.
        """
        for event in self:
            if not event.event_type_id and not event.event_ticket_ids:
                event.event_ticket_ids = False
                continue

            # lines to keep: those with existing registrations
            tickets_to_remove = event.event_ticket_ids.filtered(lambda ticket: not ticket._origin.registration_ids)
            command = [Command.unlink(ticket.id) for ticket in tickets_to_remove]
            if event.event_type_id.event_type_ticket_ids:
                command += [
                    Command.create({
                        attribute_name: line[attribute_name] if not isinstance(line[attribute_name], models.BaseModel) else line[attribute_name].id
                        for attribute_name in self.env['event.type.ticket']._get_event_ticket_fields_whitelist()
                    }) for line in event.event_type_id.event_type_ticket_ids
                ]
            event.event_ticket_ids = command

    @api.depends('event_type_id')
    def _compute_note(self):
        for event in self:
            if event.event_type_id and not is_html_empty(event.event_type_id.note):
                event.note = event.event_type_id.note

    @api.depends('event_type_id')
    def _compute_ticket_instructions(self):
        for event in self:
            if is_html_empty(event.ticket_instructions) and not \
               is_html_empty(event.event_type_id.ticket_instructions):
                event.ticket_instructions = event.event_type_id.ticket_instructions

    @api.depends('address_id')
    def _compute_address_inline(self):
        """Use venue address if available, otherwise its name, finally ''. """
        for event in self:
            if (event.address_id.contact_address or '').strip():
                event.address_inline = ', '.join(
                    frag.strip()
                    for frag in event.address_id.contact_address.split('\n') if frag.strip()
                )
            else:
                event.address_inline = event.address_id.name or ''

    @api.constrains('seats_max', 'seats_limited', 'registration_ids')
    def _check_seats_availability(self, minimal_availability=0):
        sold_out_events = []
        for event in self:
            if event.seats_limited and event.seats_max and event.seats_available < minimal_availability:
                sold_out_events.append(
                    (_('- "%(event_name)s": Missing %(nb_too_many)i seats.',
                        event_name=event.name, nb_too_many=-event.seats_available)))
        if sold_out_events:
            raise ValidationError(_('There are not enough seats available for:')
                                  + '\n%s\n' % '\n'.join(sold_out_events))

    @api.constrains('date_begin', 'date_end')
    def _check_closing_date(self):
        for event in self:
            if event.date_end < event.date_begin:
                raise ValidationError(_('The closing date cannot be earlier than the beginning date.'))

    @api.model
    def _read_group_stage_ids(self, stages, domain, order):
        return self.env['event.stage'].search([])

    @api.model_create_multi
    def create(self, vals_list):
        events = super(EventEvent, self).create(vals_list)
        for res in events:
            if res.organizer_id:
                res.message_subscribe([res.organizer_id.id])
        self.env.flush_all()
        return events

    def write(self, vals):
        if 'stage_id' in vals and 'kanban_state' not in vals:
            # reset kanban state when changing stage
            vals['kanban_state'] = 'normal'
        res = super(EventEvent, self).write(vals)
        if vals.get('organizer_id'):
            self.message_subscribe([vals['organizer_id']])
        return res

    def name_get(self):
        """Adds ticket seats availability if requested by context."""
        if not self.env.context.get('name_with_seats_availability'):
            return super().name_get()
        res = []
        for event in self:
            # event or its tickets are sold out
            if event.event_registrations_sold_out:
                name = _('%(event_name)s (Sold out)', event_name=event.name)
            elif event.seats_limited and event.seats_max:
                name = _(
                    '%(event_name)s (%(count)s seats remaining)',
                    event_name=event.name,
                    count=formatLang(self.env, event.seats_available, digits=0),
                )
            else:
                name = event.name
            res.append((event.id, name))
        return res

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        self.ensure_one()
        default = dict(default or {}, name=_("%s (copy)") % (self.name))
        return super(EventEvent, self).copy(default)

    @api.model
    def _get_mail_message_access(self, res_ids, operation, model_name=None):
        if (
            operation == 'create'
            and self.env.user.has_group('event.group_event_registration_desk')
            and (not model_name or model_name == 'event.event')
        ):
            # allow the registration desk users to post messages on Event
            # can not be done with "_mail_post_access" otherwise public user will be
            # able to post on published Event (see website_event)
            return 'read'
        return super(EventEvent, self)._get_mail_message_access(res_ids, operation, model_name)

    def _set_tz_context(self):
        self.ensure_one()
        return self.with_context(tz=self.date_tz or 'UTC')

    def action_set_done(self):
        """
        Action which will move the events
        into the first next (by sequence) stage defined as "Ended"
        (if they are not already in an ended stage)
        """
        first_ended_stage = self.env['event.stage'].search([('pipe_end', '=', True)], limit=1, order='sequence')
        if first_ended_stage:
            self.write({'stage_id': first_ended_stage.id})

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
            cal_event.add('dtstart').value = event.date_begin.astimezone(pytz.timezone(event.date_tz))
            cal_event.add('dtend').value = event.date_end.astimezone(pytz.timezone(event.date_tz))
            cal_event.add('summary').value = event.name
            if event.address_id:
                cal_event.add('location').value = event.address_inline

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

from collections import namedtuple
from datetime import datetime
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, tools
from odoo.tools import exception_to_unicode
from odoo.tools.translate import _
from odoo.exceptions import MissingError, ValidationError


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

    @api.model
    def _selection_template_model(self):
        return [('mail.template', 'Mail')]

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
    template_model_id = fields.Many2one('ir.model', string='Template Model', compute='_compute_template_model_id', compute_sudo=True)
    template_ref = fields.Reference(string='Template', selection='_selection_template_model', required=True)

    @api.depends('notification_type')
    def _compute_template_model_id(self):
        mail_model = self.env['ir.model']._get('mail.template')
        for mail in self:
            mail.template_model_id = mail_model if mail.notification_type == 'mail' else False

    def _prepare_event_mail_values(self):
        self.ensure_one()
        return namedtuple("MailValues", ['notification_type', 'interval_nbr', 'interval_unit', 'interval_type', 'template_ref'])(
            self.notification_type,
            self.interval_nbr,
            self.interval_unit,
            self.interval_type,
            '%s,%i' % (self.template_ref._name, self.template_ref.id)
        )

class EventMailScheduler(models.Model):
    """ Event automated mailing. This model replaces all existing fields and
    configuration allowing to send emails on events since Odoo 9. A cron exists
    that periodically checks for mailing to run. """
    _name = 'event.mail'
    _rec_name = 'event_id'
    _description = 'Event Automated Mailing'

    @api.model
    def _selection_template_model(self):
        return [('mail.template', 'Mail')]

    def _selection_template_model_get_mapping(self):
        return {'mail': 'mail.template'}

    @api.onchange('notification_type')
    def set_template_ref_model(self):
        mail_model = self.env['mail.template']
        if self.notification_type == 'mail':
            record = mail_model.search([('model', '=', 'event.registration')], limit=1)
            self.template_ref = "{},{}".format('mail.template', record.id) if record else False

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
    scheduled_date = fields.Datetime('Schedule Date', compute='_compute_scheduled_date', store=True)
    # contact and status
    mail_registration_ids = fields.One2many(
        'event.mail.registration', 'scheduler_id',
        help='Communication related to event registrations')
    mail_done = fields.Boolean("Sent", copy=False, readonly=True)
    mail_state = fields.Selection(
        [('running', 'Running'), ('scheduled', 'Scheduled'), ('sent', 'Sent')],
        string='Global communication Status', compute='_compute_mail_state')
    mail_count_done = fields.Integer('# Sent', copy=False, readonly=True)
    template_model_id = fields.Many2one('ir.model', string='Template Model', compute='_compute_template_model_id', compute_sudo=True)
    template_ref = fields.Reference(string='Template', selection='_selection_template_model', required=True)

    @api.depends('notification_type')
    def _compute_template_model_id(self):
        mail_model = self.env['ir.model']._get('mail.template')
        for mail in self:
            mail.template_model_id = mail_model if mail.notification_type == 'mail' else False

    @api.depends('event_id.date_begin', 'event_id.date_end', 'interval_type', 'interval_unit', 'interval_nbr')
    def _compute_scheduled_date(self):
        for scheduler in self:
            if scheduler.interval_type == 'after_sub':
                date, sign = scheduler.event_id.create_date, 1
            elif scheduler.interval_type == 'before_event':
                date, sign = scheduler.event_id.date_begin, -1
            else:
                date, sign = scheduler.event_id.date_end, 1

            scheduler.scheduled_date = date.replace(microsecond=0) + _INTERVALS[scheduler.interval_unit](sign * scheduler.interval_nbr) if date else False

    @api.depends('interval_type', 'scheduled_date', 'mail_done')
    def _compute_mail_state(self):
        for scheduler in self:
            # registrations based
            if scheduler.interval_type == 'after_sub':
                scheduler.mail_state = 'running'
            # global event based
            elif scheduler.mail_done:
                scheduler.mail_state = 'sent'
            elif scheduler.scheduled_date:
                scheduler.mail_state = 'scheduled'
            else:
                scheduler.mail_state = 'running'

    @api.constrains('notification_type', 'template_ref')
    def _check_template_ref_model(self):
        model_map = self._selection_template_model_get_mapping()
        for record in self.filtered('template_ref'):
            model = model_map[record.notification_type]
            if record.template_ref._name != model:
                raise ValidationError(_('The template which is referenced should be coming from %(model_name)s model.', model_name=model))

    def execute(self):
        for scheduler in self:
            now = fields.Datetime.now()
            if scheduler.interval_type == 'after_sub':
                new_registrations = scheduler.event_id.registration_ids.filtered_domain(
                    [('state', 'not in', ('cancel', 'draft'))]
                ) - scheduler.mail_registration_ids.registration_id
                scheduler._create_missing_mail_registrations(new_registrations)

                # execute scheduler on registrations
                scheduler.mail_registration_ids.execute()
                total_sent = len(scheduler.mail_registration_ids.filtered(lambda reg: reg.mail_sent))
                scheduler.update({
                    'mail_done': total_sent >= (scheduler.event_id.seats_reserved + scheduler.event_id.seats_used),
                    'mail_count_done': total_sent,
                })
            else:
                # before or after event -> one shot email
                if scheduler.mail_done or scheduler.notification_type != 'mail':
                    continue
                # no template -> ill configured, skip and avoid crash
                if not scheduler.template_ref:
                    continue
                # do not send emails if the mailing was scheduled before the event but the event is over
                if scheduler.scheduled_date <= now and (scheduler.interval_type != 'before_event' or scheduler.event_id.date_end > now):
                    scheduler.event_id.mail_attendees(scheduler.template_ref.id)
                    # Mail is sent to all attendees (unconfirmed as well), so count all attendees
                    scheduler.update({
                        'mail_done': True,
                        'mail_count_done': scheduler.event_id.seats_expected,
                    })
        return True

    def _create_missing_mail_registrations(self, registrations):
        new = []
        for scheduler in self:
            new += [{
                'registration_id': registration.id,
                'scheduler_id': scheduler.id,
            } for registration in registrations]
        if new:
            return self.env['event.mail.registration'].create(new)
        return self.env['event.mail.registration']

    def _prepare_event_mail_values(self):
        self.ensure_one()
        return namedtuple("MailValues", ['notification_type', 'interval_nbr', 'interval_unit', 'interval_type', 'template_ref'])(
            self.notification_type,
            self.interval_nbr,
            self.interval_unit,
            self.interval_type,
            '%s,%i' % (self.template_ref._name, self.template_ref.id)
        )

    @api.model
    def _warn_template_error(self, scheduler, exception):
        # We warn ~ once by hour ~ instead of every 10 min if the interval unit is more than 'hours'.
        if random.random() < 0.1666 or scheduler.interval_unit in ('now', 'hours'):
            ex_s = exception_to_unicode(exception)
            try:
                event, template = scheduler.event_id, scheduler.template_ref
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
        """ Backward compatible method, notably if crons are not updated when
        migrating for some reason. """
        return self.schedule_communications(autocommit=autocommit)

    @api.model
    def schedule_communications(self, autocommit=False):
        schedulers = self.search([
            ('event_id.active', '=', True),
            ('mail_done', '=', False),
            ('scheduled_date', '<=', fields.Datetime.now())
        ])

        for scheduler in schedulers:
            try:
                # Prevent a mega prefetch of the registration ids of all the events of all the schedulers
                self.browse(scheduler.id).execute()
            except Exception as e:
                _logger.exception(e)
                self.env.invalidate_all()
                self._warn_template_error(scheduler, e)
            else:
                if autocommit and not getattr(threading.current_thread(), 'testing', False):
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
        done = self.browse()
        for reg_mail in todo:
            organizer = reg_mail.scheduler_id.event_id.organizer_id
            company = self.env.company
            author = self.env.ref('base.user_root').partner_id
            if organizer.email:
                author = organizer
            elif company.email:
                author = company.partner_id
            elif self.env.user.email:
                author = self.env.user.partner_id

            email_values = {
                'author_id': author.id,
            }
            template = None
            try:
                template = reg_mail.scheduler_id.template_ref.exists()
            except MissingError:
                pass

            if not template:
                _logger.warning("Cannot process ticket %s, because Mail Scheduler %s has reference to non-existent template", reg_mail.registration_id, reg_mail.scheduler_id)
                continue

            if not template.email_from:
                email_values['email_from'] = author.email_formatted
            template.send_mail(reg_mail.registration_id.id, email_values=email_values)
            done |= reg_mail
        done.write({'mail_sent': True})

    @api.depends('registration_id', 'scheduler_id.interval_unit', 'scheduler_id.interval_type')
    def _compute_scheduled_date(self):
        for mail in self:
            if mail.registration_id:
                mail.scheduled_date = mail.registration_id.create_date.replace(microsecond=0) + _INTERVALS[mail.scheduler_id.interval_unit](mail.scheduler_id.interval_nbr)
            else:
                mail.scheduled_date = False

```

## File: models\event_registration.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from dateutil.relativedelta import relativedelta
import pytz

from odoo import _, api, fields, models, SUPERUSER_ID
from odoo.tools import format_date, email_normalize, email_normalize_all
from odoo.exceptions import AccessError, ValidationError

# phone_validation is not officially in the depends of event, but we would like
# to have the formatting available in event, not in event_sms -> do a conditional
# import just to be sure
try:
    from odoo.addons.phone_validation.tools.phone_validation import phone_format
except ImportError:
    def phone_format(number, country_code, country_phone_code, force_format='INTERNATIONAL', raise_exception=True):
        return number


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
    active = fields.Boolean(default=True)
    # utm informations
    utm_campaign_id = fields.Many2one('utm.campaign', 'Campaign',  index=True, ondelete='set null')
    utm_source_id = fields.Many2one('utm.source', 'Source', index=True, ondelete='set null')
    utm_medium_id = fields.Many2one('utm.medium', 'Medium', index=True, ondelete='set null')
    # attendee
    partner_id = fields.Many2one('res.partner', string='Booked by', tracking=1)
    name = fields.Char(
        string='Attendee Name', index='trigram',
        compute='_compute_name', readonly=False, store=True, tracking=10)
    email = fields.Char(string='Email', compute='_compute_email', readonly=False, store=True, tracking=11)
    phone = fields.Char(string='Phone', compute='_compute_phone', readonly=False, store=True, tracking=12)
    mobile = fields.Char(string='Mobile', compute='_compute_mobile', readonly=False, store=True, tracking=13)
    # organization
    date_closed = fields.Datetime(
        string='Attended Date', compute='_compute_date_closed',
        readonly=False, store=True)
    event_begin_date = fields.Datetime(string="Event Start Date", related='event_id.date_begin', readonly=True)
    event_end_date = fields.Datetime(string="Event End Date", related='event_id.date_end', readonly=True)
    event_organizer_id = fields.Many2one(string='Event Organizer', related='event_id.organizer_id', readonly=True)
    event_user_id = fields.Many2one(string='Event Responsible', related='event_id.user_id', readonly=True)
    company_id = fields.Many2one(
        'res.company', string='Company', related='event_id.company_id',
        store=True, readonly=True, states={'draft': [('readonly', False)]})
    state = fields.Selection([
        ('draft', 'Unconfirmed'), ('cancel', 'Cancelled'),
        ('open', 'Confirmed'), ('done', 'Attended')],
        string='Status', default='draft', readonly=True, copy=False, tracking=True)

    def default_get(self, fields):
        ret_vals = super().default_get(fields)
        utm_mixin_fields = ("campaign_id", "medium_id", "source_id")
        utm_fields = ("utm_campaign_id", "utm_medium_id", "utm_source_id")
        if not any(field in utm_fields for field in fields):
            return ret_vals
        utm_mixin_defaults = self.env['utm.mixin'].default_get(utm_mixin_fields)
        for (mixin_field, field) in zip(utm_mixin_fields, utm_fields):
            if field in fields and utm_mixin_defaults.get(mixin_field):
                ret_vals[field] = utm_mixin_defaults[mixin_field]
        return ret_vals

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
                    registration.date_closed = self.env.cr.now()
                else:
                    registration.date_closed = False

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

    @api.onchange('phone', 'event_id', 'partner_id')
    def _onchange_phone_validation(self):
        if self.phone:
            country = self.partner_id.country_id or self.event_id.country_id or self.env.company.country_id
            self.phone = self._phone_format(self.phone, country)

    @api.onchange('mobile', 'event_id', 'partner_id')
    def _onchange_mobile_validation(self):
        if self.mobile:
            country = self.partner_id.country_id or self.event_id.country_id or self.env.company.country_id
            self.mobile = self._phone_format(self.mobile, country)

    # ------------------------------------------------------------
    # CRUD
    # ------------------------------------------------------------

    @api.model_create_multi
    def create(self, vals_list):
        # format numbers: prefetch side records, then try to format according to country
        all_partner_ids = set(values['partner_id'] for values in vals_list if values.get('partner_id'))
        all_event_ids = set(values['event_id'] for values in vals_list if values.get('event_id'))
        for values in vals_list:
            if not values.get('phone') and not values.get('mobile'):
                continue

            related_country = self.env['res.country']
            if values.get('partner_id'):
                related_country = self.env['res.partner'].with_prefetch(all_partner_ids).browse(values['partner_id']).country_id
            if not related_country and values.get('event_id'):
                related_country = self.env['event.event'].with_prefetch(all_event_ids).browse(values['event_id']).country_id
            if not related_country:
                related_country = self.env.company.country_id

            for fname in {'mobile', 'phone'}:
                if values.get(fname):
                    values[fname] = self._phone_format(values[fname], related_country)

        registrations = super(EventRegistration, self).create(vals_list)

        # auto_confirm if possible; if not automatically confirmed, call mail schedulers in case
        # some were created already open
        if registrations._check_auto_confirmation():
            registrations.sudo().action_confirm()
        elif not self.env.context.get('install_mode', False):
            # running the scheduler for demo data can cause an issue where wkhtmltopdf runs during
            # server start and hangs indefinitely, leading to serious crashes
            # we currently avoid this by not running the scheduler, would be best to find the actual
            # reason for this issue and fix it so we can remove this check
            registrations._update_mail_schedulers()
        return registrations

    def write(self, vals):
        confirming = vals.get('state') in {'open', 'done'}
        to_confirm = (self.filtered(lambda registration: registration.state in {'draft', 'cancel'})
                      if confirming else None)
        ret = super(EventRegistration, self).write(vals)
        # As these Event(Ticket) methods are model constraints, it is not necessary to call them
        # explicitly when creating new registrations. However, it is necessary to trigger them here
        # as changes in registration states cannot be used as constraints triggers.
        if confirming:
            to_confirm.event_id._check_seats_availability()
            to_confirm.event_ticket_id._check_seats_availability()

            if not self.env.context.get('install_mode', False):
                # running the scheduler for demo data can cause an issue where wkhtmltopdf runs
                # during server start and hangs indefinitely, leading to serious crashes we
                # currently avoid this by not running the scheduler, would be best to find the
                # actual reason for this issue and fix it so we can remove this check
                to_confirm._update_mail_schedulers()

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

    def toggle_active(self):
        pre_inactive = self - self.filtered(self._active_name)
        super().toggle_active()
        # Necessary triggers as changing registration states cannot be used as triggers for the
        # Event(Ticket) models constraints.
        if pre_inactive:
            pre_inactive.event_id._check_seats_availability()
            pre_inactive.event_ticket_id._check_seats_availability()

    def _check_auto_confirmation(self):
        """ Checks that all registrations are for `auto-confirm` events. """
        return all(event.auto_confirm for event in self.event_id)

    def _phone_format(self, number, country):
        """ Call phone_validation formatting tool function. Returns original
        number in case formatting cannot be done (no country, wrong info, ...) """
        if not number or not country:
            return number
        new_number = phone_format(
            number,
            country.code,
            country.phone_code,
            force_format='E164',
            raise_exception=False,
        )
        return new_number if new_number else number

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

    def action_send_badge_email(self):
        """ Open a window to compose an email, with the template - 'event_badge'
            message loaded by default
        """
        self.ensure_one()
        template = self.env.ref('event.event_registration_mail_template_badge', raise_if_not_found=False)
        compose_form = self.env.ref('mail.email_compose_message_wizard_form')
        ctx = dict(
            default_model='event.registration',
            default_res_id=self.id,
            default_use_template=bool(template),
            default_template_id=template and template.id,
            default_composition_mode='comment',
            default_email_layout_xmlid="mail.mail_notification_light",
            name_with_seats_availability=False,
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

    def _update_mail_schedulers(self):
        """ Update schedulers to set them as running again, and cron to be called
        as soon as possible. """
        open_registrations = self.filtered(lambda registration: registration.state == 'open')
        if not open_registrations:
            return

        onsubscribe_schedulers = self.env['event.mail'].sudo().search([
            ('event_id', 'in', open_registrations.event_id.ids),
            ('interval_type', '=', 'after_sub')
        ])
        if not onsubscribe_schedulers:
            return

        onsubscribe_schedulers.update({'mail_done': False})
        # we could simply call _create_missing_mail_registrations and let cron do their job
        # but it currently leads to several delays. We therefore call execute until
        # cron triggers are correctly used
        onsubscribe_schedulers.with_user(SUPERUSER_ID).execute()

    # ------------------------------------------------------------
    # MAILING / GATEWAY
    # ------------------------------------------------------------

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

    # ------------------------------------------------------------
    # TOOLS
    # ------------------------------------------------------------

    def get_date_range_str(self, lang_code=False):
        self.ensure_one()
        today_tz = pytz.utc.localize(fields.Datetime.now()).astimezone(pytz.timezone(self.event_id.date_tz))
        event_date_tz = pytz.utc.localize(self.event_begin_date).astimezone(pytz.timezone(self.event_id.date_tz))
        diff = (event_date_tz.date() - today_tz.date())
        if diff.days <= 0:
            return _('today')
        elif diff.days == 1:
            return _('tomorrow')
        elif (diff.days < 7):
            return _('in %d days') % (diff.days, )
        elif (diff.days < 14):
            return _('next week')
        elif event_date_tz.month == (today_tz + relativedelta(months=+1)).month:
            return _('next month')
        else:
            return _('on %(date)s', date=format_date(self.env, self.event_begin_date, lang_code=lang_code, date_format='medium'))

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
        'Red Kanban Label', default=lambda s: _('Blocked'), translate=True, prefetch='legend', required=True,
        help='Override the default value displayed for the blocked state for kanban selection.')
    legend_done = fields.Char(
        'Green Kanban Label', default=lambda s: _('Ready for Next Stage'), translate=True, prefetch='legend', required=True,
        help='Override the default value displayed for the done state for kanban selection.')
    legend_normal = fields.Char(
        'Grey Kanban Label', default=lambda s: _('In Progress'), translate=True, prefetch='legend', required=True,
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

    def _default_sequence(self):
        """
        Here we use a _default method instead of ordering on 'sequence, id' to
        prevent adding a new related stored field in the 'event.tag' model that
        would hold the category id.
        """
        return (self.search([], order="sequence desc", limit=1).sequence or 0) + 1

    name = fields.Char("Name", required=True, translate=True)
    sequence = fields.Integer('Sequence', default=_default_sequence)
    tag_ids = fields.One2many('event.tag', 'category_id', string="Tags")


class EventTag(models.Model):
    _name = "event.tag"
    _description = "Event Tag"
    _order = "category_sequence, sequence, id"

    def _default_color(self):
        return randint(1, 11)

    name = fields.Char("Name", required=True, translate=True)
    sequence = fields.Integer('Sequence', default=0)
    category_id = fields.Many2one("event.tag.category", string="Category", required=True, ondelete='cascade')
    category_sequence = fields.Integer(related='category_id.sequence', string='Category Sequence', store=True)
    color = fields.Integer(
        string='Color Index', default=lambda self: self._default_color(),
        help='Tag color. No color means no display in kanban or front-end, to distinguish internal tags from public categorization tags.')

```

## File: models\event_ticket.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError, UserError
from odoo.tools.misc import formatLang


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
    seats_limited = fields.Boolean(string='Limit Attendees', readonly=True, store=True,
                                   compute='_compute_seats_limited')
    seats_max = fields.Integer(
        string='Maximum Attendees',
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
    start_sale_datetime = fields.Datetime(string="Registration Start")
    end_sale_datetime = fields.Datetime(string="Registration End")
    is_launched = fields.Boolean(string='Are sales launched', compute='_compute_is_launched')
    is_expired = fields.Boolean(string='Is Expired', compute='_compute_is_expired')
    sale_available = fields.Boolean(
        string='Is Available', compute='_compute_sale_available', compute_sudo=True,
        help='Whether it is possible to sell these tickets')
    registration_ids = fields.One2many('event.registration', 'event_ticket_id', string='Registrations')
    # seats
    seats_reserved = fields.Integer(string='Reserved Seats', compute='_compute_seats', store=False)
    seats_available = fields.Integer(string='Available Seats', compute='_compute_seats', store=False)
    seats_unconfirmed = fields.Integer(string='Unconfirmed Seats', compute='_compute_seats', store=False)
    seats_used = fields.Integer(string='Used Seats', compute='_compute_seats', store=False)
    is_sold_out = fields.Boolean(
        'Sold Out', compute='_compute_is_sold_out', help='Whether seats are not available for this ticket.')

    @api.depends('end_sale_datetime', 'event_id.date_tz')
    def _compute_is_expired(self):
        for ticket in self:
            ticket = ticket._set_tz_context()
            current_datetime = fields.Datetime.context_timestamp(ticket, fields.Datetime.now())
            if ticket.end_sale_datetime:
                end_sale_datetime = fields.Datetime.context_timestamp(ticket, ticket.end_sale_datetime)
                ticket.is_expired = end_sale_datetime < current_datetime
            else:
                ticket.is_expired = False

    @api.depends('start_sale_datetime', 'event_id.date_tz')
    def _compute_is_launched(self):
        now = fields.Datetime.now()
        for ticket in self:
            if not ticket.start_sale_datetime:
                ticket.is_launched = True
            else:
                ticket = ticket._set_tz_context()
                current_datetime = fields.Datetime.context_timestamp(ticket, now)
                start_sale_datetime = fields.Datetime.context_timestamp(ticket, ticket.start_sale_datetime)
                ticket.is_launched = start_sale_datetime <= current_datetime

    @api.depends('is_expired', 'start_sale_datetime', 'event_id.date_tz', 'seats_available', 'seats_max')
    def _compute_sale_available(self):
        for ticket in self:
            ticket.sale_available = ticket.is_launched and not ticket.is_expired and not ticket.is_sold_out

    @api.depends('seats_max', 'registration_ids.state', 'registration_ids.active')
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
                        WHERE event_ticket_id IN %s AND state IN ('draft', 'open', 'done') AND active = true
                        GROUP BY event_ticket_id, state
                    """
            self.env['event.registration'].flush_model(['event_id', 'event_ticket_id', 'state', 'active'])
            self.env.cr.execute(query, (tuple(self.ids),))
            for event_ticket_id, state, num in self.env.cr.fetchall():
                results.setdefault(event_ticket_id, {})[state_field[state]] = num

        # compute seats_available
        for ticket in self:
            ticket.update(results.get(ticket._origin.id or ticket.id, {}))
            if ticket.seats_max > 0:
                ticket.seats_available = ticket.seats_max - (ticket.seats_reserved + ticket.seats_used)

    @api.depends('seats_limited', 'seats_available')
    def _compute_is_sold_out(self):
        for ticket in self:
            ticket.is_sold_out = ticket.seats_limited and not ticket.seats_available

    @api.constrains('start_sale_datetime', 'end_sale_datetime')
    def _constrains_dates_coherency(self):
        for ticket in self:
            if ticket.start_sale_datetime and ticket.end_sale_datetime and ticket.start_sale_datetime > ticket.end_sale_datetime:
                raise UserError(_('The stop date cannot be earlier than the start date. '
                                  'Please check ticket %(ticket_name)s', ticket_name=ticket.name))

    @api.constrains('registration_ids', 'seats_max')
    def _check_seats_availability(self, minimal_availability=0):
        sold_out_tickets = []
        for ticket in self:
            if ticket.seats_max and ticket.seats_available < minimal_availability:
                sold_out_tickets.append((_(
                    '- the ticket "%(ticket_name)s" (%(event_name)s): Missing %(nb_too_many)i seats.',
                    ticket_name=ticket.name, event_name=ticket.event_id.name, nb_too_many=-ticket.seats_available)))
        if sold_out_tickets:
            raise ValidationError(_('There are not enough seats available for:')
                                  + '\n%s\n' % '\n'.join(sold_out_tickets))

    def name_get(self):
        """Adds ticket seats availability if requested by context."""
        if not self.env.context.get('name_with_seats_availability'):
            return super().name_get()
        res = []
        for ticket in self:
            if not ticket.seats_max:
                name = ticket.name
            elif not ticket.seats_available:
                name = _('%(ticket_name)s (Sold out)', ticket_name=ticket.name)
            else:
                name = _(
                    '%(ticket_name)s (%(count)s seats remaining)',
                    ticket_name=ticket.name,
                    count=formatLang(self.env, ticket.seats_available, digits=0),
                )
            res.append((ticket.id, name))
        return res

    def _get_ticket_multiline_description(self):
        """ Compute a multiline description of this ticket. It is used when ticket
        description are necessary without having to encode it manually, like sales
        information. """
        return '%s\n%s' % (self.display_name, self.event_id.display_name)

    def _set_tz_context(self):
        self.ensure_one()
        return self.with_context(tz=self.event_id.date_tz or 'UTC')

    @api.ondelete(at_uninstall=False)
    def _unlink_except_if_registrations(self):
        if self.registration_ids:
            raise UserError(_(
                "The following tickets cannot be deleted while they have one or more registrations linked to them:\n- %s",
                '\n- '.join(self.mapped('name'))))

```

## File: models\mail_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.osv import expression


class MailTemplate(models.Model):
    _inherit = 'mail.template'

    @api.model
    def _name_search(self, name, args=None, operator='ilike', limit=100, name_get_uid=None):
        """Context-based hack to filter reference field in a m2o search box to emulate a domain the ORM currently does not support.

        As we can not specify a domain on a reference field, we added a context
        key `filter_template_on_event` on the template reference field. If this
        key is set, we add our domain in the `args` in the `_name_search`
        method to filtrate the mail templates.
        """
        if self.env.context.get('filter_template_on_event'):
            args = expression.AND([[('model', '=', 'event.registration')], args])
        return super(MailTemplate, self)._name_search(name, args, operator, limit, name_get_uid)

    def unlink(self):
        res = super().unlink()
        domain = ('template_ref', 'in', [f"{template._name},{template.id}" for template in self])
        self.env['event.mail'].sudo().search([domain]).unlink()
        self.env['event.type.mail'].sudo().search([domain]).unlink()
        return res

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    module_event_sale = fields.Boolean("Tickets")
    module_website_event_meet = fields.Boolean("Discussion Rooms")
    module_website_event_track = fields.Boolean("Tracks and Agenda")
    module_website_event_track_live = fields.Boolean("Live Mode")
    module_website_event_track_quiz = fields.Boolean("Quiz on Tracks")
    module_website_event_exhibitor = fields.Boolean("Advanced Sponsors")
    module_website_event_questions = fields.Boolean("Registration Survey")
    module_event_barcode = fields.Boolean("Barcode")
    module_website_event_sale = fields.Boolean("Online Ticketing")
    module_event_booth = fields.Boolean("Booth Management")

    @api.onchange('module_website_event_track')
    def _onchange_module_website_event_track(self):
        """ Reset sub-modules, otherwise you may have track to False but still
        have track_live or track_quiz to True, meaning track will come back due
        to dependencies of modules. """
        for config in self:
            if not config.module_website_event_track:
                config.module_website_event_track_live = False
                config.module_website_event_track_quiz = False

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    event_count = fields.Integer(
        '# Events', compute='_compute_event_count', groups='event.group_event_registration_desk')

    def _compute_event_count(self):
        self.event_count = 0
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
from . import mail_template
from . import res_config_settings
from . import res_partner

```

## File: report\event_event_reports.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="paperformat_event_foldable_badge" model="report.paperformat">
        <field name="name">Custom Paperformat for the Event Foldable Badge report</field>
        <field name="default" eval="False"/>
        <field name="disable_shrinking" eval="True"/>
        <field name="format">A4</field>
        <field name="page_height">0</field>
        <field name="page_width">0</field>
        <field name="orientation">Portrait</field>
        <field name="margin_top">0</field>
        <field name="margin_bottom">0</field>
        <field name="margin_left">0</field>
        <field name="margin_right">0</field>
        <field name="dpi">96</field>
    </record>

    <record id="paperformat_event_full_page_ticket" model="report.paperformat">
        <field name="name">Custom Paperformat for the Event Full Page Ticket report</field>
        <field name="default" eval="False"/>
        <field name="disable_shrinking" eval="True"/>
        <field name="format">A4</field>
        <field name="page_height">0</field>
        <field name="page_width">0</field>
        <field name="orientation">Portrait</field>
        <field name="margin_top">0</field>
        <field name="margin_bottom">8</field>
        <field name="margin_left">0</field>
        <field name="margin_right">0</field>
        <field name="dpi">96</field>
    </record>

    <!-- The "Full Page Ticket", as opposed to the foldable badge that is meant to be folded and
    only contains the bare minimum (attendee name + barcode), gives all the information of the
    ticket in a portrait A4 format.
    It allows to add some information in the ticket_instructions field and, when printed, functions
    as an "official" ticket that the attendee can show to the registration desk where all
    the information are available (event name, organizer, SO reference, barcode, footer with
    sponsors, ...). -->
    <record id="action_report_event_registration_full_page_ticket" model="ir.actions.report">
        <field name="name">Full Page Ticket</field>
        <field name="model">event.registration</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">event.event_registration_report_template_full_page_ticket</field>
        <field name="report_file">event.event_registration_report_template_full_page_ticket</field>
        <field name="print_report_name">'Full Page Ticket - %s - %s' % ((object.event_id.name or 'Event').replace('/',''), (object.name or '').replace('/',''))</field>
        <field name="binding_model_id" ref="model_event_registration"/>
        <field name="binding_type">report</field>
        <field name="paperformat_id" ref="paperformat_event_full_page_ticket"/>
    </record>

    <record id="action_report_event_event_full_page_ticket" model="ir.actions.report">
        <field name="name">Full Page Ticket Example</field>
        <field name="model">event.event</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">event.event_event_report_template_full_page_ticket</field>
        <field name="report_file">event.event_event_report_template_full_page_ticket</field>
        <field name="print_report_name">'Full Page Ticket - %s' % (object.name or 'Event').replace('/','')</field>
        <field name="binding_model_id" ref="model_event_event"/>
        <field name="binding_type">report</field>
        <field name="paperformat_id" ref="paperformat_event_full_page_ticket"/>
    </record>

    <record id="action_report_event_registration_foldable_badge" model="ir.actions.report">
        <field name="name">Foldable Badge</field>
        <field name="model">event.registration</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">event.event_registration_report_template_foldable_badge</field>
        <field name="report_file">event.event_registration_report_template_foldable_badge</field>
        <field name="print_report_name">'Foldable Badge - %s - %s' % ((object.event_id.name or 'Event').replace('/',''), (object.name or '').replace('/',''))</field>
        <field name="binding_model_id" ref="model_event_registration"/>
        <field name="binding_type">report</field>
        <field name="paperformat_id" ref="paperformat_event_foldable_badge"/>
    </record>

    <record id="action_report_event_event_foldable_badge" model="ir.actions.report">
        <field name="name">Foldable Badge Example</field>
        <field name="model">event.event</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">event.event_event_report_template_foldable_badge</field>
        <field name="report_file">event.event_event_report_template_foldable_badge</field>
        <field name="print_report_name">'Foldable Badge - %s' % (object.name or 'Event').replace('/','')</field>
        <field name="binding_model_id" ref="model_event_event"/>
        <field name="binding_type">report</field>
        <field name="paperformat_id" ref="paperformat_event_foldable_badge"/>
    </record>

</odoo>

```

## File: report\event_event_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- EVENT FOLDABLE BADGE -->

<template id="event_report_template_foldable_badge">
    <div class="o_event_foldable_badge_container">
        <div class="row">
            <!-- Front -->
            <div class="page p-1 col-6 o_event_foldable_badge_top">
                <div class="oe_structure"/>
            </div>
            <!-- Back -->
            <div class="page col-6 o_event_foldable_badge_top o_event_foldable_badge_ticket pt-4">
                <div class="oe_structure"/>
                <div class="o_event_foldable_badge_ticket_wrapper p-2">
                    <div class="o_event_foldable_badge_ticket_wrapper_top page">
                        <h5 class="o_event_foldable_badge_event_name text-odoo fw-bold text-center" t-field="event.name"/>
                        <div class="text-center o_event_foldable_badge_font_small">
                            <span itemprop="startDate" t-field="event.date_begin"
                                t-options='{"widget": "datetime", "date_only": True}'
                                class="fw-bold"/>
                            <span itemprop="startDateTime" t-field="event.date_begin"
                                class="fw-bold"
                                t-options='{"widget": "datetime", "time_only": True, "tz_name": event.date_tz, "hide_seconds": True}'/>
                            <span class="fa fa-arrow-right o_event_foldable_badge_font_faded"/>
                            <span t-if="not event.is_one_day"
                                itemprop="endDate" t-field="event.date_end"
                                t-options='{"widget": "datetime", "date_only": True}'
                                class="fw-bold"/>
                            <span itemprop="endDateTime" t-field="event.date_end"
                                class="fw-bold"
                                t-options='{"widget": "datetime", "time_only": True, "tz_name": event.date_tz, "hide_seconds": True}'/>
                        </div>
                        <div class="o_event_foldable_badge_font_faded o_event_foldable_badge_font_small text-center">
                            (<span itemprop="timezone" t-out="event.date_tz"/>)
                        </div>
                        <div t-if="event.address_id" class="o_event_foldable_badge_font_faded o_event_foldable_badge_font_small text-center">
                            <t t-call="event.event_report_template_formatted_event_address"/>
                        </div>
                        <div class="text-center mt-5 pt-2">
                            <h2 t-if="attendee" t-field="attendee.name"/>
                            <h2 t-elif="not attendee"><span>John Doe</span></h2>
                            <t t-set="first_ticket" t-value="event.event_ticket_ids[0] if event.event_ticket_ids else None"/>
                            <div t-if="attendee" class="o_event_foldable_badge_font_faded" t-field="attendee.event_ticket_id"/>
                            <div t-elif="first_ticket" t-out="first_ticket.name"
                                class="o_event_foldable_badge_font_faded"/>
                        </div>
                    </div>
                    <div class="o_event_foldable_badge_barcode mt-2">
                        <div class="o_event_foldable_badge_barcode_container">
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="row">
            <!-- Inner left -->
            <div class="page o_event_foldable_badge_bottom o_event_foldable_badge_left p-1 px-2 col-6 overflow-hidden">
                <div class="oe_structure"/>
                <div t-field="event.ticket_instructions" class="p-4"></div>
            </div>
            <!-- Inner right -->
            <div class="page o_event_foldable_badge_bottom o_event_foldable_badge_right col-6 text-center">
                <div class="oe_structure"/>
                <div class="h-50 col-12 row m-0">
                    <div class="col-6 h-100 p-0">
                        <div class="o_event_foldable_badge_step fw-bold">1</div>
                        <img src="/event/static/src/img/how_to_fold_1.png" class="w-100 h-100" alt="How to Fold (1)"/>
                    </div>
                    <div class="col-6 h-100 p-0">
                        <div class="o_event_foldable_badge_step fw-bold">2</div>
                        <img src="/event/static/src/img/how_to_fold_2.png" class="w-100 h-100" alt="How to Fold (2)"/>
                    </div>
                </div>
                <div class="h-50 col-12 row m-0">
                    <div class="col-6 h-100 p-0">
                        <div class="o_event_foldable_badge_step fw-bold">3</div>
                        <img src="/event/static/src/img/how_to_fold_3.png" class="w-100 h-100" alt="How to Fold (3)"/>
                    </div>
                    <div class="col-6 h-100 p-0">
                        <div class="o_event_foldable_badge_step fw-bold">4</div>
                        <img src="/event/static/src/img/how_to_fold_4.png" class="w-100 h-100" alt="How to Fold (4)"/>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="event_registration_report_template_foldable_badge">
    <t t-call="web.basic_layout">
        <t t-foreach="docs" t-as="attendee">
            <t t-set="event" t-value="attendee.event_id.with_context(tz=attendee.event_id.date_tz)"/>
            <t t-call="event.event_report_template_foldable_badge"/>
        </t>
    </t>
</template>

<template id="event_event_report_template_foldable_badge">
    <t t-call="web.basic_layout">
        <t t-foreach="docs" t-as="event">
            <t t-set="event" t-value="event.with_context(tz=event.date_tz)"/>
            <t t-call="event.event_report_template_foldable_badge"/>
        </t>
    </t>
</template>

<!-- EVENT FULL PAGE TICKET -->

<template id="event_report_template_full_page_ticket">
    <div class="row page">
        <div class="o_event_full_page_ticket_container page w-100">
            <div class="o_event_full_page_ticket_wrapper">
                <div class="o_event_full_page_ticket_details row">
                    <div class="col-10">
                        <div class="row">
                            <div class="col-8 page">
                                <div class="o_event_full_page_ticket_font_faded">
                                    Event Ticket For
                                </div>
                                <h5 class="o_event_full_page_ticket_event_name text-odoo fw-bold py-0 my-0" t-field="event.name"/>
                                <span itemprop="startDate" t-field="event.date_begin"
                                     t-options='{"widget": "datetime", "date_only": True}'
                                     class="fw-bold"/>
                                <span itemprop="startDateTime" t-field="event.date_begin"
                                    class="fw-bold"
                                    t-options='{"widget": "datetime", "time_only": True, "hide_seconds": True, "tz_name": event.date_tz}'/>
                                <span class="fa fa-arrow-right o_event_full_page_ticket_font_faded"/>
                                <span t-if="not event.is_one_day"
                                    itemprop="endDate" t-field="event.date_end"
                                    t-options='{"widget": "datetime", "date_only": True}'
                                    class="fw-bold"/>
                                <span itemprop="endDateTime" t-field="event.date_end"
                                    class="fw-bold"
                                    t-options='{"widget": "datetime", "time_only": True, "hide_seconds": True, "tz_name": event.date_tz}'/>
                                <span itemprop="timeZone" class="o_event_full_page_ticket_font_faded small">(<t t-out="event.date_tz"/>)</span>
                                <div t-if="event.address_id" class="o_event_full_page_ticket_font_faded">
                                    <t t-call="event.event_report_template_formatted_event_address"/>
                                </div>
                                <br/>
                                <h5 t-if="attendee" t-field="attendee.name"></h5>
                                <h5 t-elif="not attendee"><span>John Doe</span></h5>
                                <t t-set="first_ticket" t-value="event.event_ticket_ids[0] if event.event_ticket_ids else None"/>
                                <div t-if="attendee" class="o_event_full_page_ticket_font_faded" t-field="attendee.event_ticket_id"/>
                                <div t-elif="first_ticket" t-out="first_ticket.name"
                                    class="o_event_full_page_ticket_font_faded"/>
                            </div>
                            <div class="col-4 o_event_full_page_ticket_side_info page">
                                <span t-if="event.organizer_id.image_128">
                                    <img t-att-src="image_data_uri(event.organizer_id.image_128)" class="o_event_full_page_ticket_event_logo mb-2"/>
                                </span>
                                <div t-if="attendee and attendee.partner_id" class="mb-2 o_event_full_page_ticket_side_info_booked_by">
                                    <div class="o_event_full_page_ticket_font_faded o_event_full_page_ticket_small_caps fw-bold">Booked By</div>
                                    <div class="o_event_full_page_ticket_small" t-field="attendee.partner_id"/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="o_event_full_page_ticket_barcode col-2">
                        <div class="o_event_full_page_ticket_barcode_container">
                        </div>
                    </div>
                </div>
            </div>
            <div class="page oe_structure"/>
            <div t-field="event.ticket_instructions" class="o_event_full_page_extra_instructions px-2 pt-3"></div>
        </div>
    </div>
</template>

<template id="event_report_full_page_ticket_layout">
    <!-- Inspired from "external_layout_standard" to get a repeated footer element. -->
    <div class="article"
        t-att-data-oe-model="main_object and main_object._name"
        t-att-data-oe-id="main_object and main_object.id"
        t-att-data-oe-lang="main_object and main_object.env.context.get('lang')">
        <main>
            <t t-out="0"/>
        </main>
    </div>

    <div class="row footer o_event_full_page_ticket_footer d-block">
        <div class="o_event_full_page_ticket_powered_by bg-odoo text-white text-center p-2 w-100">
            <t t-if="event.organizer_id">
                <span class="fw-bold" t-field="event.organizer_id.name"></span>
                <span t-if="event.organizer_id.phone" class="ps-3 fa fa-phone"/>
                <span t-if="event.organizer_id.phone" t-field="event.organizer_id.phone"/>
                <span t-if="event.organizer_id.email_normalized" class="ps-3 fa fa-envelope"/>
                <span t-if="event.organizer_id.email_normalized" t-field="event.organizer_id.email_normalized"/>
                <span t-if="event.organizer_id.website" class="ps-3 fa fa-globe"/>
                <span t-if="event.organizer_id.website" t-field="event.organizer_id.website"/>
            </t>
            <t t-else="">
                <span t-out="event.name"/> <!-- Force some content to avoid messing the layout -->
            </t>
        </div>
    </div>
</template>

<template id="event_registration_report_template_full_page_ticket">
    <t t-foreach="docs" t-as="attendee">
        <t t-call="web.html_container">
            <t t-set="event" t-value="attendee.event_id._set_tz_context()"/>
            <t t-set="main_object" t-value="attendee"/>
            <t t-call="event.event_report_full_page_ticket_layout">
                <t t-call="event.event_report_template_full_page_ticket"/>
            </t>
        </t>
    </t>
</template>

<template id="event_event_report_template_full_page_ticket">
    <t t-foreach="docs" t-as="event">
        <t t-call="web.html_container">
            <t t-set="event" t-value="event._set_tz_context()"/>
            <t t-set="main_object" t-value="event"/>
            <t t-call="event.event_report_full_page_ticket_layout">
                <t t-call="event.event_report_template_full_page_ticket"/>
            </t>
        </t>
    </t>
</template>

<!-- MISC -->

<template id="event_report_template_formatted_event_address">
    <!-- Small utility template to display "Venue" as:
    fa-map-marker PartnerName
    RestOfAddress -->
    <span class="fa fa-map-marker"/>
    <t t-if="event.address_id.contact_address.strip()">
        <t t-set="address_bits" t-value="event.address_id.contact_address.split('\n')"/>
        <t t-if="address_bits">
            <t t-set="partner_name" t-value="address_bits[0]"/>
            <span t-out="partner_name"/>
        </t>
        <t t-if="len(address_bits) > 1">
            <br/>
        </t>
        <t t-set="remaining_bits" t-value="address_bits[1:]"/>
        <t t-foreach="remaining_bits" t-as="address_bit">
            <t t-if="address_bit and address_bit.strip()">
                <span t-out="address_bit"/>
            </t>
        </t>
    </t>
    <t t-else="" t-out="event.address_id.name"/>
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

        <record id="group_event_registration_desk" model="res.groups">
            <field name="name">Registration Desk</field>
            <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
            <field name="category_id" ref="base.module_category_marketing_events"/>
        </record>

        <record id="group_event_user" model="res.groups">
            <field name="name">User</field>
            <field name="implied_ids" eval="[(4, ref('group_event_registration_desk'))]"/>
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
access_event_type_registration,event.type.registration,model_event_type,event.group_event_registration_desk,1,0,0,0
access_event_type_manager,event.type.manager,model_event_type,event.group_event_manager,1,1,1,1
access_event_type_ticket_registration,event.type.ticket.registration,model_event_type_ticket,event.group_event_registration_desk,1,0,0,0
access_event_type_ticket_manager,event.type.ticket.manager,model_event_type_ticket,event.group_event_manager,1,1,1,1
access_event_event_registration,event.event.registration,model_event_event,event.group_event_registration_desk,1,0,0,0
access_event_event_user,event.event.user,model_event_event,event.group_event_user,1,1,1,0
access_event_event_manager,event.event.manager,model_event_event,event.group_event_manager,1,1,1,1
access_event_event_ticket,event.event.ticket,model_event_event_ticket,,0,0,0,0
access_event_event_ticket_registration,event.event.ticket.registration,model_event_event_ticket,event.group_event_registration_desk,1,0,0,0
access_event_event_ticket_user,event.event.ticket.user,model_event_event_ticket,event.group_event_user,1,1,1,1
access_event_registration,event.registration,model_event_registration,,0,0,0,0
access_event_registration_registration,event.registration.registration,model_event_registration,event.group_event_registration_desk,1,1,1,0
access_event_registration_manager,event.registration.manager,model_event_registration,event.group_event_manager,1,1,1,1
access_event_mail_registration,event.mail.registration,model_event_mail,event.group_event_registration_desk,1,0,0,0
access_event_mail_user,event.mail.user,model_event_mail,event.group_event_user,1,1,1,1
access_event_mail_registration_registration,event.mail.registration.registration,model_event_mail_registration,event.group_event_registration_desk,1,0,0,0
access_event_mail_registration_manager,event.mail.registration.manager,model_event_mail_registration,event.group_event_manager,1,1,1,1
access_event_type_mail_registration,event.type.mail.registration,model_event_type_mail,event.group_event_registration_desk,1,0,0,0
access_event_type_mail_manager,event.type.mail.manager,model_event_type_mail,event.group_event_manager,1,1,1,1
access_event_stage_registration,event.stage.registration,model_event_stage,event.group_event_registration_desk,1,0,0,0
access_event_stage_manager,event.stage.manager,model_event_stage,event.group_event_manager,1,1,1,1
access_event_tag_category_registration,event.tag.category.registration,model_event_tag_category,event.group_event_registration_desk,1,0,0,0
access_event_tag_category_user,event.tag.category.user,model_event_tag_category,event.group_event_user,1,1,1,1
access_event_tag,event.tag,model_event_tag,,0,0,0,0
access_event_tag_registration,event.tag.user,model_event_tag,event.group_event_registration_desk,1,0,0,0
access_event_tag_user,event.tag.user,model_event_tag,event.group_event_user,1,1,1,0
access_event_tag_manager,event.tag.manager,model_event_tag,event.group_event_manager,1,1,1,1

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

## File: static\src\icon_selection_field\icon_selection_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { _lt } from "@web/core/l10n/translation";
import { standardFieldProps } from "@web/views/fields/standard_field_props";

const { Component } = owl;

export class IconSelectionField extends Component {
    get icon() {
        return this.props.icons[this.props.value];
    }
    get title() {
        return this.props.value.charAt(0).toUpperCase() + this.props.value.slice(1);
    }
}
IconSelectionField.template = "event.IconSelectionField";
IconSelectionField.props = {
    ...standardFieldProps,
    icons: Object,
};

IconSelectionField.displayName = _lt("Icon Selection");
IconSelectionField.supportedTypes = ["char", "text", "selection"];

IconSelectionField.extractProps = ({ attrs }) => ({
    icons: attrs.options,
});

registry.category("fields").add("event_icon_selection", IconSelectionField);

```

## File: static\src\icon_selection_field\icon_selection_field.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates xml:space="preserve">

    <t t-name="event.IconSelectionField" owl="1">
        <i t-att-class="icon" t-att-data-tooltip="title"/>
    </t>

</templates>

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

const {_t} = require('web.core');
const {Markup} = require('web.utils');

var tour = require('web_tour.tour');
var EventAdditionalTourSteps = require('event.event_steps');

tour.register('event_tour', {
    url: '/web',
    rainbowManMessage: _t("Great! Now all you have to do is wait for your attendees to show up!"),
    sequence: 210,
}, [tour.stepUtils.showAppsMenuItem(), {
    trigger: '.o_app[data-menu-xmlid="event.event_main_menu"]',
    content: Markup(_t("Ready to <b>organize events</b> in a few minutes? Let's get started!")),
    position: 'bottom',
    edition: 'enterprise',
}, {
    trigger: '.o_app[data-menu-xmlid="event.event_main_menu"]',
    content: Markup(_t("Ready to <b>organize events</b> in a few minutes? Let's get started!")),
    edition: 'community',
}, {
    trigger: '.o-kanban-button-new',
    extra_trigger: '.o_event_kanban_view',
    content: Markup(_t("Let's create your first <b>event</b>.")),
    position: 'bottom',
    width: 175,
}, {
    trigger: '.o_event_form_view input[id="name"]',
    content: Markup(_t("This is the <b>name</b> your guests will see when registering.")),
    run: 'text Odoo Experience 2020',
}, {
    trigger: '.o_event_form_view div[name="date_end"]',
    content: _t("Open date range picker. Pick a Start date for your event"),
    run: function () {
        $('input[id="date_begin"]').val('09/30/2020 08:00:00').change();
        $('input[id="date_end"]').val('10/02/2020 23:00:00').change();
        $('.o_event_form_view input[id="date_end"]').click();
    },
}, {
    content: _t("Apply change."),
    trigger: '.daterangepicker .applyBtn',
    in_modal: false,
}, {
    trigger: '.o_event_form_view div[name="event_ticket_ids"] .o_field_x2many_list_row_add a',
    content: Markup(_t("Ticket types allow you to distinguish your attendees. Let's <b>create</b> a new one.")),
}, tour.stepUtils.autoExpandMoreButtons(),
...new EventAdditionalTourSteps()._get_website_event_steps(), {
    trigger: '.o_event_form_view div[name="stage_id"]',
    content: _t("Now that your event is ready, click here to move it to another stage."),
    position: 'bottom',
}, {
    trigger: 'ol.breadcrumb li.breadcrumb-item:first',
    extra_trigger: '.o_event_form_view div[name="stage_id"]',
    content: Markup(_t("Use the <b>breadcrumbs</b> to go back to your kanban overview.")),
    position: 'bottom',
    run: 'click',
}].filter(Boolean));

});

```

## File: views\event_event_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>

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
                        <button name="%(event.event_registration_action_stats_from_event)d"
                                type="action" class="oe_stat_button" icon="fa-line-chart">
                            <span class="o_stat_text">
                                Registration statistics
                            </span>
                        </button>
                        <button name="%(event.act_event_registration_from_event)d"
                                type="action"
                                context="{'search_default_expected': True}"
                                class="oe_stat_button"
                                icon="fa-users"
                                help="Total Registrations for this Event">
                            <field name="seats_expected" widget="statinfo" string="Attendees"/>
                        </button>
                    </div>
                    <field name="active" invisible="1"/>
                    <field name="company_id" invisible="1"/>
                    <field name="legend_blocked" invisible="1"/>
                    <field name="legend_normal" invisible="1"/>
                    <field name="legend_done" invisible="1"/>
                    <widget name="web_ribbon" text="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <field name="kanban_state" widget="state_selection" class="ms-auto float-end"/>
                    <div class="oe_title">
                        <label for="name" string="Event Name"/>
                        <h1><field class="text-break" name="name" placeholder="e.g. Conference for Architects"/></h1>
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
                        <group name="right_event_details">
                            <field name="organizer_id"/>
                            <field name="user_id" domain="[('share', '=', False)]"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                            <field name="address_id"
                                context="{'show_address': 1}"
                                options='{"always_reload": True}'/>
                            <label for="seats_limited" string="Limit Registrations"/>
                            <div>
                                <field name="seats_limited"/>
                                <span attrs="{'invisible': [('seats_limited', '=', False)], 'required': [('seats_limited', '=', False)]}">to <field name="seats_max" class="oe_inline o_input_9ch"/> Confirmed Attendees</span>
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
                                    <field name="notification_type"/>
                                    <field name="template_model_id" invisible="1"/>
                                    <field name="template_ref" options="{'hide_model': True, 'no_quick_create': True}" context="{'filter_template_on_event': True, 'default_model': 'event.registration'}"/>
                                    <field name="interval_nbr" attrs="{'readonly':[('interval_unit','=','now')]}"/>
                                    <field name="interval_unit"/>
                                    <field name="interval_type"/>
                                    <field name="scheduled_date" groups="base.group_no_one"/>
                                    <field name="mail_count_done"/>
                                    <field name="mail_state" widget="event_icon_selection" string=" " nolabel="1"
                                        options="{'sent': 'fa fa-check', 'scheduled': 'fa fa-hourglass-half', 'running': 'fa fa-cogs'}"/>
                                </tree>
                            </field>
                        </page>
                        <page string="Notes" name="event_notes">
                            <group>
                                <label for="note" string="Note" />
                                <br />
                                <field nolabel="1" colspan="2" name="note" 
                                    placeholder="Add some internal notes (to do lists, contact info, ...)" />
                                <label for="ticket_instructions" string="Ticket Instructions" />
                                <br />
                                <field nolabel="1" colspan="2" name="ticket_instructions" 
                                    placeholder="e.g. How to get to your event, door closing time, ..." />
                            </group>
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
                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                <field name="seats_expected" string="Expected Attendees" sum="Total" readonly="1"/>
                <field name="seats_used" sum="Total" readonly="1"/>
                <field name="seats_max" string="Maximum Seats" sum="Total" readonly="1" optional="hide"/>
                <field name="seats_reserved" sum="Total" readonly="1" optional="hide"/>
                <field name="seats_unconfirmed" string="Unconfirmed Seats" sum="Total" readonly="1" optional="hide"/>
                <field name="stage_id" readonly="1"/>
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
                    <field name="name" placeholder="e.g. Conference for Architects"/>
                    <label for="date_begin" string="Date"/>
                    <div class="o_row">
                        <field name="date_begin" widget="daterange" options="{'related_end_date': 'date_end'}"/>
                        <i class="fa fa-long-arrow-right mx-2" aria-label="Arrow icon" title="Arrow"/>
                        <field name="date_end" widget="daterange" options="{'related_start_date': 'date_begin'}"/>
                    </div>
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
                                <div class="col-3 text-bg-primary p-2 text-center d-flex flex-column justify-content-center">
                                    <div t-esc="luxon.DateTime.fromISO(record.date_begin.raw_value).toFormat('d')" class="o_event_fontsize_20"/>
                                    <div>
                                        <t t-esc="luxon.DateTime.fromISO(record.date_begin.raw_value).toFormat('MMM yyyy')"/>
                                    </div>
                                    <div><t t-esc="luxon.DateTime.fromISO(record.date_begin.raw_value).toFormat('t')"/></div>
                                        <div t-if="record.date_begin.raw_value !== record.date_end.raw_value">
                                            <i class="fa fa-arrow-right me-2 o_event_fontsize_09" title="End date"/>
                                            <t t-esc="luxon.DateTime.fromISO(record.date_end.raw_value).toFormat('d MMM')"/>
                                        </div>
                                </div>
                                <div class="col-9 py-2 px-3 d-flex flex-column justify-content-between pt-3">
                                    <div>
                                        <div class="o_kanban_record_title o_text_overflow" t-att-title="record.name.value">
                                            <field name="name"/>
                                        </div>
                                        <div t-if="record.address_id.value" class="d-flex">
                                            <i class="fa fa-map-marker mt-1 me-1" title="Location"/>
                                            <span t-esc="record.address_id.value"/>
                                        </div>
                                    </div>
                                    <h5 class="o_event_fontsize_11 p-0">
                                        <a name="%(event_registration_action_stats_from_event)d"
                                           type="action"
                                           context="{'search_default_expected': True}">
                                            <t t-esc="record.seats_expected.raw_value"/> Expected attendees
                                        </a>
                                        <t t-set="total_seats" t-value="record.seats_reserved.raw_value + record.seats_used.raw_value"/>
                                        <div  class="pt-2 pt-md-0" t-if="total_seats > 0 and ! record.auto_confirm.raw_value"><br/>
                                            <a class="ps-2" name="%(event_registration_action_stats_from_event)d" type="action" context="{'search_default_confirmed': True}">
                                                <i class="fa fa-level-up fa-rotate-90" title="Confirmed"/><span class="ps-2"><t t-esc="total_seats"/> Confirmed</span>
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
                <field name="user_id" avatar_field="avatar_128"/>
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
                <field name="address_search"/>
                <field name="stage_id"/>
                <filter string="My Events" name="myevents" help="My Events" domain="[('user_id', '=', uid)]"/>
                <separator/>
                <filter string="Upcoming/Running" name="upcoming"
                    domain="[('date_end', '&gt;=', datetime.datetime.combine(context_today(), datetime.time(0,0,0)))]" help="Upcoming events from today" />
                <separator/>
                <filter string="Start Date" name="start_date" date="date_begin"/>
                <separator/>
                <filter string="Archived" name="filter_inactive" domain="[('active', '=', False)]"/>
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

    <!-- EVENT.EVENT VIEWS -->

    <record model="ir.actions.act_window" id="action_event_view">
       <field name="name">Events</field>
       <field name="type">ir.actions.act_window</field>
       <field name="res_model">event.event</field>
       <field name="view_mode">kanban,calendar,tree,form,pivot,graph</field>
       <field name="search_view_id" ref="view_event_search"/>
       <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create an Event
          </p><p>
            Schedule and organize your events: handle registrations, send automated confirmation emails, sell tickets, etc.
          </p>
        </field>
    </record>

    <record id="event.menu_event_event" model="ir.ui.menu">
        <field name="action" ref="event.action_event_view"/>
    </record>

</data></odoo>

```

## File: views\event_mail_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>

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
                            <field name="notification_type"/>
                            <field name="template_ref" options="{'hide_model': True, 'no_quick_create': True}" context="{'filter_template_on_event': True, 'default_model': 'event.registration'}"/>
                            <field name="mail_state"/>
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
                <field name="notification_type"/>
                <field name="template_ref" options="{'hide_model': True, 'no_quick_create': True}" context="{'filter_template_on_event': True, 'default_model': 'event.registration'}"/>
                <field name="scheduled_date"/>
                <field name="mail_count_done"/>
                <field name="mail_state" widget="event_icon_selection" string=" " nolabel="1"
                    options="{'sent': 'fa fa-check', 'scheduled': 'fa fa-hourglass-half', 'running': 'fa fa-cogs'}"/>
            </tree>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_event_mail">
        <field name="name">Events Mail Schedulers</field>
        <field name="res_model">event.mail</field>
        <field name="context">{'create': False}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Nothing Scheduled yet!
            </p><p>
                Under this technical menu you will find all scheduled communication related to your events.
            </p>
        </field>
    </record>

    <record id="menu_event_mail_schedulers" model="ir.ui.menu">
        <field name="action" ref="event.action_event_mail"/>
    </record>
</data></odoo>

```

## File: views\event_menu_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>

    <!-- MAIN MENU -->
    <menuitem name="Events"
        id="event_main_menu"
        sequence="125"
        groups="event.group_event_registration_desk"
        web_icon="event,static/description/icon.svg"/>

    <!-- HEADER: EVENTS -->
    <menuitem name="Events"
        id="menu_event_event"
        sequence="1"
        parent="event.event_main_menu"
        groups="event.group_event_registration_desk"/>

    <!-- HEADER: REPORTING -->
    <menuitem name="Reporting"
        id="menu_reporting_events"
        sequence="50"
        parent="event_main_menu"
        groups="event.group_event_user"/>

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
        sequence="3"
        parent="menu_event_configuration"/>

</data></odoo>

```

## File: views\event_registration_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>

    <!-- EVENT.REGISTRATION VIEWS -->
    <record model="ir.ui.view" id="view_event_registration_tree">
        <field name="name">event.registration.tree</field>
        <field name="model">event.registration</field>
        <field name="arch" type="xml">
            <tree string="Registration" multi_edit="1" sample="1"
                  expand="1" default_order="create_date desc"
                  class="o_event_registration_view_tree">
                <field name="active" invisible="1"/>
                <field name="create_date" optional="show" string="Registration Date"/>
                <field name="name"/>
                <field name="partner_id" optional="hide"/>
                <field name="email" optional="show"/>
                <field name="phone" optional="show"/>
                <field name="mobile" optional="hide"/>
                <field name="event_id" invisible="context.get('default_event_id')"/>
                <field name="event_ticket_id" domain="[('event_id', '=', event_id)]"/>
                <field name="activity_ids" widget="list_activity"/>
                <field name="state" decoration-info="state in ('draft', 'open')"
                       decoration-success="state == 'done'"
                       decoration-muted="state == 'cancel'" widget="badge"/>
                <field name="company_id" groups="base.group_multi_company" optional="hide"/>
                <field name="message_needaction" invisible="1"/>
                <button name="action_confirm" string="Confirm" type="object" icon="fa-check"
                        attrs="{'invisible': ['|', ('active', '=', False), ('state', '!=', 'draft')]}"/>
                <button name="action_set_done" string="Mark as Attending" type="object" icon="fa-level-down"
                        attrs="{'invisible': ['|', ('active', '=', False), ('state', '!=', 'open')]}"/>
                <button name="action_cancel" string="Cancel" type="object"
                        class="o_btn_cancel_registration" icon="fa-times"
                        attrs="{'invisible': ['|', ('active', '=', False), '&amp;', ('state', '!=', 'open'), ('state', '!=', 'draft')]}"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </tree>
        </field>
    </record>

    <record model="ir.ui.view" id="view_event_registration_form">
        <field name="name">event.registration.form</field>
        <field name="model">event.registration</field>
        <field name="arch" type="xml">
            <form string="Event Registration">
                <field name="active" invisible="1"/>
                <header>
                    <button name="action_send_badge_email" string="Send by Email" type="object" class="oe_highlight"
                            attrs="{'invisible': ['|', ('active', '=', False), '&amp;', ('state', '!=', 'open'), ('state', '!=', 'done')]}"/>
                    <button name="action_confirm" string="Confirm" type="object" class="oe_highlight"
                            attrs="{'invisible': ['|', ('active', '=', False), ('state', '!=', 'draft')]}"/>
                    <button name="action_set_done" string="Attended" type="object" class="oe_highlight"
                            attrs="{'invisible': ['|', ('active', '=', False), ('state', '!=', 'open')]}"/>
                    <button name="action_set_draft" string="Set To Unconfirmed" type="object"
                            attrs="{'invisible': ['|', ('active', '=', False), '&amp;', ('state', '!=', 'cancel'), ('state', '!=', 'done')]}"/>
                    <button name="action_cancel" string="Cancel Registration" type="object"
                            attrs="{'invisible': ['|', ('active', '=', False), '&amp;', ('state', '!=', 'open'), ('state', '!=', 'draft')]}"/>
                    <field name="state" nolabel="1" colspan="2" widget="statusbar" statusbar_visible="draft,open,done"/>
                </header>
                <sheet string="Registration">
                    <div class="oe_button_box" name="button_box"/>
                    <widget name="web_ribbon" text="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <group>
                        <group string="Attendee" name="attendee">
                            <field class="o_text_overflow" name="name"/>
                            <field name="email"/>
                            <field name="phone" class="o_force_ltr" widget="phone" options="{'enable_sms': false}"/>
                            <field name="mobile" class="o_force_ltr" widget="phone"/>
                        </group>
                        <group string="Event Information" name="event">
                            <field class="text-break" name="event_id" attrs="{'readonly': [('state', '!=', 'draft')]}"
                                   context="{'name_with_seats_availability': True}" options="{'no_create': True}"/>
                            <field name="event_ticket_id" attrs="{'invisible': [('event_id', '=', False)]}"
                                   context="{'name_with_seats_availability': True}" options="{'no_open': True, 'no_create': True}"
                                   domain="[('event_id', '=', event_id)]"/>
                            <field name="partner_id"/>
                            <field name="create_date" string="Registration Date" groups="base.group_no_one"/>
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
                <field name="active" invisible="1"/>
                <templates>
                    <t t-name="event_attendees_kanban_icons_desktop">
                        <div class="d-none d-md-block h-100">
                            <div id="event_attendees_kanban_icons_desktop" class="h-100 float-end p-2 d-flex align-items-end flex-column">
                                <t t-if="record.active.raw_value">
                                    <a class="btn btn-md btn-primary" string="Confirm Registration" name="action_confirm" type="object" states="draft" role="button">
                                        <i class="fa fa-check" role="img" aria-label="Confirm button" title="Confirm Registration"/>
                                    </a>
                                    <a class="btn btn-md btn-primary" string="Confirm Attendance" name="action_set_done" type="object" states="open" role="button">
                                        <i class="fa fa-user-plus" role="img" aria-label="Attended button" title="Confirm Attendance"/>
                                    </a>
                                    <span class="text-muted" states="done">Attended</span>
                                    <span class="text-muted" states="cancel">Canceled</span>
                                </t>
                            </div>
                        </div>
                    </t>
                    <t t-name="event_attendees_kanban_icons_mobile">
                        <div id="event_attendees_kanban_icons_mobile" class="d-md-none h-100 ps-4">
                            <t t-if="record.active.raw_value">
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
                            </t>
                        </div>
                    </t>
                    <t t-name="kanban-box">
                        <div t-attf-class="#{!record.active.raw_value ? 'oe_kanban_card_ribbon' : ''} oe_kanban_global_click o_event_registration_kanban container-fluid p-0">
                            <div t-if="!record.active.raw_value" class="ribbon ribbon-top-right">
                                <span class="bg-danger">Archived</span>
                            </div>
                            <div class="row h-100">
                                <div class="col-9 pe-0">
                                    <div class="oe_kanban_content h-100">
                                        <div class="o_kanban_record_body pt-1 ps-2 h-100 d-flex flex-column">
                                            <b class="o_kanban_record_title"><field name="name"/></b>
                                            <field class="o_text_overflow" name="event_id" invisible="context.get('default_event_id')" />
                                            <span class="o_text_overflow" attrs="{'invisible': [('partner_id', '=', False)]}">Booked by <field name="partner_id" /></span>
                                            <div id="event_ticket_id" class="o_field_many2manytags o_field_widget d-flex mt-auto">
                                                <t t-if="record.event_ticket_id.raw_value">
                                                    <div t-attf-class="badge rounded-pill o_tag_color_#{(record.event_ticket_id.raw_value % 11) + 1}" >
                                                        <b><span class="o_badge_text o_text_overflow"><t t-out="record.event_ticket_id.value"/></span></b>
                                                    </div>
                                                </t>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                                <div id="event_attendees_kanban_icons" class="col-3 ps-0">
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
            <pivot string="Registration" display_quantity="1" sample="1">
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
                <field name="event_id"/>
                <field name="event_user_id" string="Responsible"/>
                <field name="event_organizer_id" string="Organizer"/>
                <field name="name" string="Participant" filter_domain="['|', ('name', 'ilike', self), ('email', 'ilike', self)]"/>
                <filter string="Ongoing Events" name="filter_is_ongoing" domain="[('event_id.is_ongoing', '=', True)]"/>
                <filter string="Expected" name="expected" domain="[('state', 'in', ['draft', 'open', 'done'])]"/>
                <separator/>
                <filter string="Unconfirmed" name="unconfirmed" domain="[('state', '=', 'draft')]"/>
                <filter string="Confirmed" name="confirmed" domain="[('state', '=', 'open')]"/>
                <filter string="Attended" name="attended" domain="[('state', '=', 'done')]"/>
                <separator/>
                <filter string="Registration Date" name="filter_create_date" date="create_date"/>
                <filter string="Event Start Date" name="filter_event_begin_date" date="event_begin_date"/>
                <filter string="Attended Date" name="filter_date_closed" date="date_closed"/>
                <field name="partner_id"/>
                <field name="company_id"/>
                <separator/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                <filter string="Last 30 days" name="filter_last_month_creation" domain="[('create_date','&gt;', (context_today() - datetime.timedelta(days=30)).strftime('%Y-%m-%d'))]"/>
                <separator/>
                <filter string="Archived" name="filter_inactive" domain="[('active', '=', False)]"/>
                <group expand="0" string="Group By">
                    <filter string="Partner" name="partner" domain="[]" context="{'group_by':'partner_id'}"/>
                    <filter string="Event" name="group_event" domain="[]" context="{'group_by':'event_id'}"/>
                    <filter string="Ticket Type" name ="group_event_ticket_id" domain="[]" context="{'group_by': 'event_ticket_id'}"/>
                    <filter string="Status" name="status" domain="[]" context="{'group_by':'state'}"/>
                    <filter string="Registration Date" name="group_by_create_date_week" domain="[]" context="{'group_by': 'create_date:week'}"
                            invisible="context.get('registration_view_hide_group_by_create_date_week')"/>
                    <filter string="Registration Date" name="group_by_create_date_day" domain="[]" context="{'group_by': 'create_date:day'}"
                            invisible="not context.get('registration_view_hide_group_by_create_date_week')"/>
                    <filter string="Campaign" name="group_by_utm_campaign_id" domain="[]"
                            context="{'group_by': 'utm_campaign_id'}"/>
                    <filter string="Medium" name="group_by_utm_medium_id" domain="[]"
                            context="{'group_by': 'utm_medium_id'}"/>
                    <filter string="Source" name="group_by_utm_source_id" domain="[]"
                            context="{'group_by': 'utm_source_id'}"/>
               </group>
            </search>
        </field>
    </record>

    <!-- Search view typically used when coming from a specific event, meaning we don't need event-related fields -->
    <record id="event_registration_view_search_event_specific" model="ir.ui.view">
        <field name="name">event.registration.view.search.event.specific</field>
        <field name="model">event.registration</field>
        <field name="inherit_id" ref="view_registration_search"/>
        <field name="mode">primary</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <xpath expr="//search/field[@name='event_id']" position="replace"/>
            <xpath expr="//search/field[@name='event_user_id']" position="replace"/>
            <xpath expr="//search/field[@name='event_organizer_id']" position="replace"/>
            <xpath expr="//search/filter[@name='filter_is_ongoing']" position="replace"/>
            <xpath expr="//search/group/filter[@name='group_event']" position="replace"/>
        </field>
    </record>

    <!-- EVENT.REGISTRATION ACTIONS -->
    <record id="act_event_registration_from_event" model="ir.actions.act_window">
        <field name="res_model">event.registration</field>
        <field name="name">Attendees</field>
        <field name="view_mode">tree,kanban,form,calendar,graph</field>
        <field name="domain">[('event_id', '=', active_id)]</field>
        <field name="context">{'default_event_id': active_id, 'name_with_seats_availability': True}</field>
        <field name="search_view_id" ref="event_registration_view_search_event_specific"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No Attendees yet!
            </p><p>
                Wait until Attendees register to your Event or create their registrations manually.
            </p>
        </field>
    </record>

    <!-- We need kanban view to be displayed first while coming from registration desk, so we have created
     new action and changed the view sequence. -->
    <record id="event_registration_action_kanban" model="ir.actions.act_window">
        <field name="res_model">event.registration</field>
        <field name="name">Attendees</field>
        <field name="view_mode">kanban,tree,form,calendar,graph</field>
        <field name="domain">[('event_id', '=', active_id)]</field>
        <field name="context">{'default_event_id': active_id}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No Attendees yet!
            </p><p>
                Wait until Attendees register to your Event or create their registrations manually.
            </p>
        </field>
    </record>

    <record id="event_registration_action" model="ir.actions.act_window">
        <field name="res_model">event.registration</field>
        <field name="name">Attendees</field>
        <field name="view_mode">kanban,tree,form,calendar,graph</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No Attendees expected yet!
            </p><p>
                Wait until Attendees register to your Event or create their registrations manually.
            </p>
        </field>
    </record>

    <record id="event_registration_action_tree" model="ir.actions.act_window">
       <field name="name">Event registrations</field>
       <field name="type">ir.actions.act_window</field>
       <field name="res_model">event.registration</field>
       <field name="view_mode">tree,kanban,form,calendar,graph</field>
    </record>

    <record id="action_registration" model="ir.actions.act_window">
        <field name="name">Attendees</field>
        <field name="res_model">event.registration</field>
        <field name="domain"></field>
        <field name="view_mode">graph,pivot,kanban,tree,form</field>
        <field name="context">{
                'search_default_filter_last_month_creation': 1,
                'search_default_status': 2,
                'search_default_group_by_create_date_day': 3,
                'search_default_group_event': 1,
                'registration_view_hide_group_by_create_date_week': 1,
            }
        </field>
        <field name="search_view_id" ref="view_registration_search"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No Attendees yet!
            </p><p>
                From this dashboard you can report, analyze and detect trends regarding your event registrations.
            </p>
        </field>
    </record>

    <record id="event_registration_action_stats_from_event" model="ir.actions.act_window">
        <field name="name">Registration statistics</field>
        <field name="res_model">event.registration</field>
        <field name="view_mode">graph,pivot,kanban,tree,form</field>
        <field name="domain">[('event_id', '=', active_id)]</field>
        <field name="context">{
                'default_event_id': active_id,
                'search_default_group_by_create_date_day': 1,
                'search_default_status': 2,
                'registration_view_hide_group_by_create_date_week': 1,
            }
        </field>
        <field name="search_view_id" ref="event_registration_view_search_event_specific"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No Attendees yet!
            </p><p>
                From this dashboard you can report, analyze and detect trends regarding your event registrations.
            </p>
        </field>
    </record>

    <menuitem name="Attendees"
        id="menu_action_registration"
        parent="event.menu_reporting_events"
        sequence="4"
        action="action_registration"
        groups="event.group_event_user"/>
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
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create an Event Stage
            </p><p>
                Event stages are used to track the progress of an Event from its origin until its conclusion.
            </p>
        </field>
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
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create an Event Tag Category
                </p><p>
                    Use Event Tag Categories to classify and organize your event tags.
                </p>
            </field>
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
        </data>
    <template id="event_default_descripton" name="Event default description">
        <section class="s_text_block">
            <h5>Join us for this 24 hours Event</h5>
            <p>Every year we invite our community, partners and end-users to come and meet us! It's the ideal event to get together and present new features, roadmap of future versions, achievements of the software, workshops, training sessions, etc...
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
                <field name="start_sale_datetime" optional="show"/>
                <field name="end_sale_datetime" optional="show"/>
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
                            <field name="start_sale_datetime"/>
                            <field name="end_sale_datetime"/>
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
                                    <strong><t t-out="record.name.value"/></strong>
                                </div>
                            </div>
                            <div><i>
                            <t t-out="record.seats_reserved.value"/> reserved + <t t-out="record.seats_reserved.value"/> unconfirmed
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
                        <label for="name" string="Ticket Type"/>
                        <h1><field name="name" placeholder="e.g. VIP Ticket"/></h1>
                    </div>
                    <group>
                        <group>
                            <field name="event_id"/>
                            <field name="seats_limited"/>
                            <field name="seats_available"/>
                            <field name="start_sale_datetime"/>
                            <field name="end_sale_datetime"/>
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

## File: views\event_type_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>

    <!-- EVENT.TYPE VIEWS -->
    <record model="ir.ui.view" id="view_event_type_form">
        <field name="name">event.type.form</field>
        <field name="model">event.type</field>
        <field name="arch" type="xml">
            <form string="Event Category">
               <sheet>
                    <div class="oe_title" name="event_type_title">
                        <label for="name" string="Event Template"/>
                        <h1><field name="name" placeholder="e.g. Online Conferences" class="mb-2"/></h1>
                    </div>
                   <group>
                       <group>
                           <field name="default_timezone" class="w-100"/>
                           <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_quick_create': True}"/>
                       </group>
                       <group>
                           <label for="has_seats_limitation" string="Limit Registrations"/>
                           <div>
                               <field name="has_seats_limitation"/>
                               <span attrs="{'invisible': [('has_seats_limitation', '=', False)], 'required': [('has_seats_limitation', '=', False)]}">
                                   to <field name="seats_max" class="oe_inline o_input_9ch"/>
                                   Confirmed Attendees
                               </span>
                           </div>
                           <label for="auto_confirm" string="Autoconfirmation"/>
                           <field name="auto_confirm" nolabel="1"/>
                       </group>
                   </group>

                   <notebook>
                       <page string="Tickets">
                           <field name="event_type_ticket_ids"
                                  class="w-100"
                                  context="{
                                     'tree_view_ref': 'event.event_type_ticket_view_tree_from_type',
                                     'form_view_ref': 'event.event_type_ticket_view_form_from_type'
                                  }"
                           />
                       </page>
                       <page string="Communication" name="event_type_communication">
                           <field name="event_type_mail_ids" class="w-100">
                               <tree string="Communication" editable="bottom">
                                   <field name="notification_type"/>
                                   <field name="template_model_id" invisible="1"/>
                                   <field name="template_ref" options="{'model_field': 'template_model_id', 'no_quick_create': True}" context="{'filter_template_on_event': True, 'default_model': 'event.registration'}"/>
                                   <field name="interval_nbr" attrs="{'readonly':[('interval_unit', '=', 'now')]}"/>
                                   <field name="interval_unit"/>
                                    <field name="interval_type"/>
                               </tree>
                           </field>
                       </page>
                       <page string="Notes">
                           <group>
                                <label for="note" string="Note" />
                                <br />
                                <field nolabel="1" colspan="2" name="note" 
                                    placeholder="Add some internal notes (to do lists, contact info, ...)" />
                                <label for="ticket_instructions" string="Ticket Instructions" />
                                <br />
                                <field nolabel="1" colspan="2" name="ticket_instructions" 
                                    placeholder="e.g. How to get to your event, door closing time, ..." />
                            </group>
                       </page>
                   </notebook>
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
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create an Event Template
            </p><p>
                Event Templates combine configurations you use often and are
                usually based on the types of events you organize (e.g. "Workshop",
                "Roadshow", "Online Webinar", etc).
            </p>
        </field>
    </record>

    <record id="menu_event_type" model="ir.ui.menu">
        <field name="action" ref="event.action_event_type"/>
    </record>
</data></odoo>

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
                                        <field name="module_website_event_exhibitor"/>
                                    </div>
                                    <div class="o_setting_right_pane">
                                        <label string="Online Exhibitors" for="module_website_event_exhibitor"/>
                                        <div class="text-muted">
                                            Display Sponsors and Exhibitors on your event pages
                                        </div>
                                    </div>
                                </div>
                                <div class="mt-3">
                                    <div class="o_setting_left_pane">
                                        <field name="module_event_booth"/>
                                    </div>
                                    <div class="o_setting_right_pane">
                                        <label string="Booth Management" for="module_event_booth"/>
                                        <div class="text-muted">
                                            Create Booths and manage their reservations
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

