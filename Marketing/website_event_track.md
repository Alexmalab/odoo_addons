# Odoo Module: website_event_track

Category: Marketing

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
    'name': 'Advanced Events',
    'category': 'Marketing',
    'summary': 'Sponsors, Tracks, Agenda, Event News',
    'version': '1.0',
    'description': "",
    'depends': ['website_event'],
    'data': [
        'security/ir.model.access.csv',
        'security/event_track_security.xml',
        'data/event_data.xml',
        'data/event_track_data.xml',
        'views/event_track_templates.xml',
        'views/event_track_views.xml',
        'views/event_views.xml',
    ],
    'demo': [
        'data/event_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import babel
import babel.dates
import collections
import datetime
import pytz
from werkzeug.exceptions import NotFound

from odoo import fields, http
from odoo.http import request
from odoo.tools import html_escape as escape, html2plaintext
from odoo.tools.misc import babel_locale_parse


class WebsiteEventTrackController(http.Controller):

    @http.route(['''/event/<model("event.event", "[('website_id', 'in', (False, current_website_id))]"):event>/track/<model("event.track", "[('event_id','=',event[0])]"):track>'''], type='http', auth="public", website=True)
    def event_track_view(self, event, track, **post):
        if not event.can_access_from_current_website():
            raise NotFound()

        track = track.sudo().with_context(tz=event.date_tz or 'UTC')
        values = {'track': track, 'event': track.event_id, 'main_object': track}
        return request.render("website_event_track.track_view", values)

    def _get_locale_time(self, dt_time, lang_code):
        """ Get locale time from datetime object

            :param dt_time: datetime object
            :param lang_code: language code (eg. en_US)
        """
        locale = babel_locale_parse(lang_code)
        return babel.dates.format_time(dt_time, format='short', locale=locale)

    def _prepare_calendar(self, event, event_track_ids):
        local_tz = pytz.timezone(event.date_tz or 'UTC')
        lang_code = request.env.context.get('lang')
        locations = {}                  # { location: [track, start_date, end_date, rowspan]}
        dates = []                      # [ (date, {}) ]
        for track in event_track_ids:
            locations.setdefault(track.location_id or False, [])

        forcetr = True
        for track in event_track_ids:
            start_date = fields.Datetime.from_string(track.date).replace(tzinfo=pytz.utc).astimezone(local_tz)
            end_date = start_date + datetime.timedelta(hours=(track.duration or 0.5))
            location = track.location_id or False
            locations.setdefault(location, [])

            # New TR, align all events
            if forcetr or (start_date>dates[-1][0]) or not location:
                formatted_time = self._get_locale_time(start_date, lang_code)
                dates.append((start_date, {}, bool(location), formatted_time))
                for loc in list(locations):
                    if locations[loc] and (locations[loc][-1][2] > start_date):
                        locations[loc][-1][3] += 1
                    elif not locations[loc] or locations[loc][-1][2] <= start_date:
                        locations[loc].append([False, locations[loc] and locations[loc][-1][2] or dates[0][0], start_date, 1])
                        dates[-1][1][loc] = locations[loc][-1]
                forcetr = not bool(location)

            # Add event
            if locations[location] and locations[location][-1][1] > start_date:
                locations[location][-1][3] -= 1
            locations[location].append([track, start_date, end_date, 1])
            dates[-1][1][location] = locations[location][-1]
            locations = collections.OrderedDict(sorted(locations.items(), key=lambda t: t[0].id if t[0] else 0))
        return {
            'locations': locations,
            'dates': dates
        }

    @http.route(['''/event/<model("event.event", "[('website_id', 'in', (False, current_website_id))]"):event>/agenda'''], type='http', auth="public", website=True, sitemap=False)
    def event_agenda(self, event, tag=None, **post):
        if not event.can_access_from_current_website():
            raise NotFound()

        event = event.with_context(tz=event.date_tz or 'UTC')
        local_tz = pytz.timezone(event.date_tz or 'UTC')
        days_tracks = collections.defaultdict(lambda: [])
        for track in event.track_ids.sorted(lambda track: (bool(track.date), track.date, bool(track.location_id))):
            if not track.date:
                continue
            date = fields.Datetime.from_string(track.date).replace(tzinfo=pytz.utc).astimezone(local_tz)
            days_tracks[str(date)[:10]].append(track)

        days = {}
        tracks_by_days = {}
        for day, tracks in days_tracks.items():
            tracks_by_days[day] = tracks
            days[day] = self._prepare_calendar(event, tracks)

        return request.render("website_event_track.agenda", {
            'event': event,
            'days': days,
            'tracks_by_days': tracks_by_days,
            'tag': tag
        })

    @http.route([
        '''/event/<model("event.event", "[('website_id', 'in', (False, current_website_id))]"):event>/track''',
        '''/event/<model("event.event", "[('website_id', 'in', (False, current_website_id))]"):event>/track/tag/<model("event.track.tag"):tag>'''
    ], type='http', auth="public", website=True, sitemap=False)
    def event_tracks(self, event, tag=None, **post):
        if not event.can_access_from_current_website() or (tag and tag.color == 0):
            raise NotFound()

        event = event.with_context(tz=event.date_tz or 'UTC')
        searches = {}
        if tag:
            searches.update(tag=tag.id)
            tracks = event.track_ids.filtered(lambda track: tag in track.tag_ids)
        else:
            tracks = event.track_ids

        values = {
            'event': event,
            'main_object': event,
            'tracks': tracks,
            'tags': event.tracks_tag_ids,
            'searches': searches,
            'html2plaintext': html2plaintext
        }
        return request.render("website_event_track.tracks", values)

    @http.route(['''/event/<model("event.event", "[('website_id', 'in', (False, current_website_id))]"):event>/track_proposal'''], type='http', auth="public", website=True, sitemap=False)
    def event_track_proposal(self, event, **post):
        if not event.can_access_from_current_website():
            raise NotFound()

        return request.render("website_event_track.event_track_proposal", {'event': event})

    @http.route(['''/event/<model("event.event", "[('website_id', 'in', (False, current_website_id))]"):event>/track_proposal/post'''], type='http', auth="public", methods=['POST'], website=True)
    def event_track_proposal_post(self, event, **post):
        if not event.can_access_from_current_website():
            raise NotFound()

        tags = []
        for tag in event.allowed_track_tag_ids:
            if post.get('tag_' + str(tag.id)):
                tags.append(tag.id)

        track = request.env['event.track'].sudo().create({
            'name': post['track_name'],
            'partner_name': post['partner_name'],
            'partner_email': post['email_from'],
            'partner_phone': post['phone'],
            'partner_biography': escape(post['biography']),
            'event_id': event.id,
            'tag_ids': [(6, 0, tags)],
            'user_id': False,
            'description': escape(post['description'])
        })
        if request.env.user != request.website.user_id:
            track.sudo().message_subscribe(partner_ids=request.env.user.partner_id.ids)
        else:
            partner = request.env['res.partner'].sudo().search([('email', '=', post['email_from'])])
            if partner:
                track.sudo().message_subscribe(partner_ids=partner.ids)
        return request.render("website_event_track.event_track_proposal_success", {'track': track, 'event': event})

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
        <record id="event_type_data_tracks" model="event.type">
            <field name="name">Conference</field>
            <field name="auto_confirm" eval="True"/>
            <field name="use_mail_schedule" eval="True"/>
            <field name="website_menu" eval="True"/>
            <field name="website_track" eval="True"/>
            <field name="website_track_proposal" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: data\event_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="event_track_tag1" model="event.track.tag">
        <field name="name">Technical</field>
    </record>
    <record id="event_track_tag2" model="event.track.tag">
        <field name="name">Business</field>
    </record>
    <record id="event_track_tag3" model="event.track.tag">
        <field name="name">Lightning Talks</field>
    </record>
    <record id="event_track_tag4" model="event.track.tag">
        <field name="name">Round Table</field>
    </record>
    <!--
        This should be done at the end so that the menu is complete
    -->
    <record id="event.event_0" model="event.event">
        <field name="website_menu">True</field>
        <field name="website_track">True</field>
        <field name="website_track_proposal">True</field>
    </record>

    <!-- Sponsorships -->

    <record id="event_sponsor_0" model="event.sponsor">
        <field name="event_id" ref="event.event_0"/>
        <field name="sponsor_type_id" ref="event_sponsor_type1"/>
        <field name="partner_id" ref="base.res_partner_2"/>
        <field name="url">http://odoo.com</field>
    </record>

    <record id="event_sponsor_1" model="event.sponsor">
        <field name="event_id" ref="event.event_0"/>
        <field name="sponsor_type_id" ref="event_sponsor_type2"/>
        <field name="partner_id" ref="base.res_partner_12"/>
        <field name="url">http://odoo.com</field>
    </record>

    <record id="event_sponsor_2" model="event.sponsor">
        <field name="event_id" ref="event.event_0"/>
        <field name="sponsor_type_id" ref="event_sponsor_type2"/>
        <field name="partner_id" ref="base.res_partner_3"/>
    </record>

    <record id="event_sponsor_3" model="event.sponsor">
        <field name="event_id" ref="event.event_0"/>
        <field name="sponsor_type_id" ref="event_sponsor_type3"/>
        <field name="partner_id" ref="base.res_partner_4"/>
        <field name="url">http://odoo.com</field>
    </record>

    <!-- Tracks -->
    <record id="base.res_partner_address_16" model="res.partner">
        <field name="website">http://facebook.com/odoo</field>
        <field name="website_description" type="xml">
            <p>
                Ayaan has in the IT sector <b>since 20 years</b>. He
                develops software to help develop websites.  He sold his
                first company at 30 years old and manage to grow OpenCorp
                from 1 to 55 employees mostly by reselling services on
                Odoo.
            </p><p>
                Ayaan is the <b>author of several books</b>, including Amazon best seller
                "How Odoo will change the business world!".
            </p>
        </field>
    </record>

    <record id="event_track_location5" model="event.track.location">
        <field name="name">Le Foyer du lac</field>
    </record>
    <record id="event_track_location6" model="event.track.location">
        <field name="name">Theatre</field>
    </record>
    <record id="event_track_location7" model="event.track.location">
        <field name="name">Lauzelle</field>
    </record>
    <record id="event_track_location8" model="event.track.location">
        <field name="name">Foyer Royal</field>
    </record>
    <record id="event_track_location9" model="event.track.location">
        <field name="name">Biereau</field>
    </record>
    <record id="event_track_location10" model="event.track.location">
        <field name="name">Bruyère</field>
    </record>
    <record id="event_track1" model="event.track">
        <field name="name">How to design a new piece of furniture</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 06:00:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location5"/>
        <field name="duration" eval="1"/>
        <field name="partner_id" ref="base.res_partner_2"/>
        <field name="color">3</field>
        <field name="stage_id" ref="event_track_stage0"/>
        <field name="kanban_state">blocked</field>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track2" model="event.track">
        <field name="name">How to integrate hardware materials in your pieces of furniture</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 08:30:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location5"/>
        <field name="duration" eval="0.25"/>
        <field name="partner_id" ref="base.res_partner_3"/>
        <field name="stage_id" ref="event_track_stage1"/>
        <field name="kanban_state">done</field>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track3" model="event.track">
        <field name="name">Portfolio presentation</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 10:30:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location5"/>
        <field name="duration" eval="0.3"/>
        <field name="partner_id" ref="base.res_partner_4"/>
        <field name="stage_id" ref="event_track_stage1"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track4" model="event.track">
        <field name="name">How to develop automated processes</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 09:00:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location5"/>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_2"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track5" model="event.track">
        <field name="name">The new way to promote your creations</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 06:00:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location6"/>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_4"/>
        <field name="stage_id" ref="event_track_stage2"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track6" model="event.track">
        <field name="name">Detailed roadmap of our new products</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 06:30:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location6"/>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_3"/>
        <field name="stage_id" ref="event_track_stage2"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track7" model="event.track">
        <field name="name">A technical explanation of how to use computer design apps</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 08:30:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location6"/>
        <field name="duration" eval="1"/>
        <field name="partner_id" ref="base.res_partner_1"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track8" model="event.track">
        <field name="name">How to optimize your sales, from leads to sales orders</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 06:00:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location7"/>
        <field name="duration" eval="0.5"/>
        <field name="color">2</field>
        <field name="partner_id" ref="base.res_partner_4"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="kanban_state">blocked</field>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track9" model="event.track">
        <field name="name">How to improve your quality processes</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 08:30:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location7"/>
        <field name="duration" eval="1"/>
        <field name="color">2</field>
        <field name="partner_id" ref="base.res_partner_12"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track10" model="event.track">
        <field name="name">Raising qualitive insights from your customers</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 06:30:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location7"/>
        <field name="duration" eval="0.5"/>
        <field name="color">5</field>
        <field name="stage_id" ref="event_track_stage0"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track11" model="event.track">
        <field name="name">Discover our new design team</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 10:30:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location7"/>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_4"/>
        <field name="stage_id" ref="event_track_stage1"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track12" model="event.track">
        <field name="name">Latest trends</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 11:00:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location7"/>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_2"/>
        <field name="stage_id" ref="event_track_stage2"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track13" model="event.track">
        <field name="name">Advanced reporting</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 06:30:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location8"/>
        <field name="duration" eval="0.25"/>
        <field name="partner_id" ref="base.res_partner_12"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track14" model="event.track">
        <field name="name">Partnership programs</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 07:00:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location8"/>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_10"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track15" model="event.track">
        <field name="name">How to communicate with your community</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 10:30:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location8"/>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_3"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track16" model="event.track">
        <field name="name">How to follow us on the social media</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 11:00:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location8"/>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_12"/>
        <field name="stage_id" ref="event_track_stage0"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track17" model="event.track">
        <field name="name">The new marketing strategy</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 06:00:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location9"/>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_10"/>
        <field name="stage_id" ref="event_track_stage0"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track18" model="event.track">
        <field name="name">How to build your marketing strategy within a competitive environment</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 08:30:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location9"/>
        <field name="duration" eval="0.5"/>
        <field name="color">5</field>
        <field name="partner_id" ref="base.res_partner_2"/>
        <field name="stage_id" ref="event_track_stage1"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track19" model="event.track">
        <field name="name">Advanced lead management : tips and tricks from the fields</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 09:00:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location9"/>
        <field name="duration" eval="0.5"/>
        <field name="color">5</field>
        <field name="partner_id" ref="base.res_partner_4"/>
        <field name="stage_id" ref="event_track_stage1"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track20" model="event.track">
        <field name="name">New Certification Program</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 10:00:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location9"/>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_3"/>
        <field name="stage_id" ref="event_track_stage1"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track21" model="event.track">
        <field name="name">House of World Cultures</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 10:30:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location9"/>
        <field name="duration" eval="0.5"/>
        <field name="color">7</field>
        <field name="stage_id" ref="event_track_stage1"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track22" model="event.track">
        <field name="name">Minimal but efficient design</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 11:00:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location9"/>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_3"/>
        <field name="stage_id" ref="event_track_stage0"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track23" model="event.track">
        <field name="name">Key Success factors selling our furniture</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 07:00:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location9"/>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_1"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track24" model="event.track">
        <field name="name">Design contest (entire day)</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 06:00:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location10"/>
        <field name="duration" eval="1.5"/>
        <field name="partner_id" ref="base.res_partner_18"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track25" model="event.track">
        <field name="name">Design contest (entire afternoon)</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 08:30:00')"></field>
        <field name="location_id" ref="website_event_track.event_track_location10"/>
        <field name="duration" eval="2.5"/>
        <field name="partner_id" ref="base.res_partner_18"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track27" model="event.track">
        <field name="name">My Company global presentation</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 04:00:00')"></field>
        <field name="duration" eval="1"/>
        <field name="partner_id" ref="base.res_partner_1"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="color">3</field>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track28" model="event.track">
        <field name="name">Status &amp; Strategy</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 05:00:00')"></field>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_2"/>
        <field name="stage_id" ref="event_track_stage2"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track29" model="event.track">
        <field name="name">The new marketing strategy</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 05:30:00')"></field>
        <field name="duration" eval="0.25"/>
        <field name="partner_id" ref="base.res_partner_2"/>
        <field name="stage_id" ref="event_track_stage2"/>
        <field name="color">6</field>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track30" model="event.track">
        <field name="name">Morning break</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 05:45:00')"></field>
        <field name="duration" eval="0.25"/>
        <field name="stage_id" ref="event_track_stage1"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track31" model="event.track">
        <field name="name">Lunch</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="time.strftime(str(DateTime.today().year) + '-06-04 07:30:00')"></field>
        <field name="duration" eval="1"/>
        <field name="stage_id" ref="event_track_stage1"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>

</odoo>

```

## File: data\event_track_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Email templates -->
        <record id="mail_template_data_track_confirmation" model="mail.template">
            <field name="name">Track: Confirmation</field>
            <field name="model_id" ref="website_event_track.model_event_track"/>
            <field name="subject">Confirmation of ${object.name}</field>
            <field name="use_default_to" eval="True"/>
            <field name="body_html" type="xml">
<div>
    Dear ${object.partner_name or ''}<br/>
    We are pleased to inform you that your proposal ${object.name} has been accepted and confirmed for the event ${object.event_id.name}.
    <br/>
    You will find more details here:
    <div style="margin: 16px 0px 16px 0px;">
        <a href="/event/${object.event_id.id}/track/${object.id}"
                style="padding: 8px 16px 8px 16px; font-size: 14px; color: #FFFFFF; text-decoration: none !important; background-color: #875A7B; border: 0px solid #875A7B; border-radius:3px">
            View Talk
        </a>
    </div>
    <br/><br/>
    Thank you,
</div>
            </field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="user_signature" eval="True"/>
            <field name="auto_delete" eval="True"/>
        </record>

        <record id="event_sponsor_type1" model="event.sponsor.type">
            <field name="name">Bronze</field>
            <field name="sequence">3</field>
        </record>
        <record id="event_sponsor_type2" model="event.sponsor.type">
            <field name="name">Silver</field>
            <field name="sequence">2</field>
        </record>
        <record id="event_sponsor_type3" model="event.sponsor.type">
            <field name="name">Gold</field>
            <field name="sequence">1</field>
        </record>

        <record id="event_track_stage0" model="event.track.stage">
            <field name="name">Proposal</field>
            <field name="sequence">1</field>
        </record>
        <record id="event_track_stage1" model="event.track.stage">
            <field name="name">Confirmed</field>
            <field name="sequence">2</field>
            <field name="mail_template_id" ref="mail_template_data_track_confirmation"/>
        </record>
        <record id="event_track_stage2" model="event.track.stage">
            <field name="name">Announced</field>
            <field name="sequence">3</field>
        </record>
        <record id="event_track_stage3" model="event.track.stage">
            <field name="name">Published</field>
            <field name="sequence">4</field>
            <field name="is_done" eval="True"/>
        </record>
        <record id="event_track_stage4" model="event.track.stage">
            <field name="name">Refused</field>
            <field name="sequence">5</field>
            <field name="fold" eval="True"/>
        </record>
        <record id="event_track_stage5" model="event.track.stage">
            <field name="name">Cancelled</field>
            <field name="sequence">6</field>
            <field name="fold" eval="True"/>
            <field name="is_cancel" eval="True"/>
        </record>

        <!-- Event-related subtypes for new track / Chatter -->
        <record id="mt_event_track" model="mail.message.subtype">
            <field name="name">New Track</field>
            <field name="res_model">event.event</field>
            <field name="description">New Track Created</field>
            <field name="default" eval="False"/>
        </record>

        <!-- Track subtypes -->
        <record id="mt_track_blocked" model="mail.message.subtype">
            <field name="name">Track Blocked</field>
            <field name="res_model">event.track</field>
            <field name="default" eval="False"/>
            <field name="internal" eval="True"/>
            <field name="description">Track blocked</field>
        </record>
        <record id="mt_track_ready" model="mail.message.subtype">
            <field name="name">Track Ready</field>
            <field name="res_model">event.track</field>
            <field name="default" eval="True"/>
            <field name="internal" eval="True"/>
            <field name="description">Track Ready for Next Stage</field>
        </record>
    </data>
</odoo>

```

## File: models\event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.addons.http_routing.models.ir_http import slug


class EventType(models.Model):
    _inherit = 'event.type'

    website_track = fields.Boolean('Tracks on Website')
    website_track_proposal = fields.Boolean('Tracks Proposals on Website')

    @api.onchange('website_menu')
    def _onchange_website_menu(self):
        if not self.website_menu:
            self.website_track = False
            self.website_track_proposal = False


class EventMenu(models.Model):
    _name = "website.event.menu"
    _description = "Website Event Menu"

    menu_id = fields.Many2one('website.menu', string='Menu', ondelete='cascade')
    event_id = fields.Many2one('event.event', string='Event', ondelete='cascade')
    menu_type = fields.Selection([('track', 'Event Tracks Menus'), ('track_proposal', 'Event Proposals Menus')])


class Event(models.Model):
    _inherit = "event.event"

    track_ids = fields.One2many('event.track', 'event_id', 'Tracks')
    track_count = fields.Integer('Track Count', compute='_compute_track_count')

    sponsor_ids = fields.One2many('event.sponsor', 'event_id', 'Sponsors')
    sponsor_count = fields.Integer('Sponsor Count', compute='_compute_sponsor_count')

    website_track = fields.Boolean('Tracks on Website')
    website_track_proposal = fields.Boolean('Proposals on Website')

    track_menu_ids = fields.One2many('website.event.menu', 'event_id', string='Event Tracks Menus', domain=[('menu_type', '=', 'track')])
    track_proposal_menu_ids = fields.One2many('website.event.menu', 'event_id', string='Event Proposals Menus', domain=[('menu_type', '=', 'track_proposal')])

    allowed_track_tag_ids = fields.Many2many('event.track.tag', relation='event_allowed_track_tags_rel', string='Available Track Tags')
    tracks_tag_ids = fields.Many2many(
        'event.track.tag', relation='event_track_tags_rel', string='Track Tags',
        compute='_compute_tracks_tag_ids', store=True)

    def _compute_track_count(self):
        data = self.env['event.track'].read_group([('stage_id.is_cancel', '!=', True)], ['event_id'], ['event_id'])
        result = dict((data['event_id'][0], data['event_id_count']) for data in data)
        for event in self:
            event.track_count = result.get(event.id, 0)

    def _compute_sponsor_count(self):
        data = self.env['event.sponsor'].read_group([], ['event_id'], ['event_id'])
        result = dict((data['event_id'][0], data['event_id_count']) for data in data)
        for event in self:
            event.sponsor_count = result.get(event.id, 0)

    def _toggle_create_website_menus(self, vals):
        super(Event, self)._toggle_create_website_menus(vals)
        for event in self:
            if 'website_track' in vals:
                if vals['website_track']:
                    for sequence, (name, url, xml_id, menu_type) in enumerate(event._get_track_menu_entries()):
                        menu = super(Event, event)._create_menu(sequence, name, url, xml_id)
                        event.env['website.event.menu'].create({
                            'menu_id': menu.id,
                            'event_id': event.id,
                            'menu_type': menu_type,
                        })
                else:
                    event.track_menu_ids.mapped('menu_id').unlink()
            if 'website_track_proposal' in vals:
                if vals['website_track_proposal']:
                    for sequence, (name, url, xml_id, menu_type) in enumerate(event._get_track_proposal_menu_entries()):
                        menu = super(Event, event)._create_menu(sequence, name, url, xml_id)
                        event.env['website.event.menu'].create({
                            'menu_id': menu.id,
                            'event_id': event.id,
                            'menu_type': menu_type,
                        })
                else:
                    event.track_proposal_menu_ids.mapped('menu_id').unlink()

    def _get_track_menu_entries(self):
        self.ensure_one()
        res = [
            (_('Talks'), '/event/%s/track' % slug(self), False, 'track'),
            (_('Agenda'), '/event/%s/agenda' % slug(self), False, 'track')]
        return res

    def _get_track_proposal_menu_entries(self):
        self.ensure_one()
        res = [(_('Talk Proposals'), '/event/%s/track_proposal' % slug(self), False, 'track_proposal')]
        return res

    @api.depends('track_ids.tag_ids', 'track_ids.tag_ids.color')
    def _compute_tracks_tag_ids(self):
        for event in self:
            event.tracks_tag_ids = event.track_ids.mapped('tag_ids').filtered(lambda tag: tag.color != 0).ids

    @api.onchange('event_type_id')
    def _onchange_type(self):
        super(Event, self)._onchange_type()
        if self.event_type_id and self.website_menu:
            self.website_track = self.event_type_id.website_track
            self.website_track_proposal = self.event_type_id.website_track_proposal

    @api.onchange('website_menu')
    def _onchange_website_menu(self):
        if not self.website_menu:
            self.website_track = False
            self.website_track_proposal = False

    @api.onchange('website_track')
    def _onchange_website_track(self):
        if not self.website_track:
            self.website_track_proposal = False

    @api.onchange('website_track_proposal')
    def _onchange_website_track_proposal(self):
        if self.website_track_proposal:
            self.website_track = True

```

## File: models\event_track.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.tools.translate import _, html_translate
from odoo.addons.http_routing.models.ir_http import slug
from datetime import timedelta


class TrackTag(models.Model):
    _name = "event.track.tag"
    _description = 'Event Track Tag'
    _order = 'name'

    name = fields.Char('Tag Name', required=True)
    track_ids = fields.Many2many('event.track', string='Tracks')
    color = fields.Integer(string='Color Index', help="Note that colorless tags won't be available on the website.")

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Tag name already exists !"),
    ]


class TrackLocation(models.Model):
    _name = "event.track.location"
    _description = 'Event Track Location'

    name = fields.Char('Location')


class TrackStage(models.Model):
    _name = 'event.track.stage'
    _description = 'Event Track Stage'
    _order = 'sequence, id'

    name = fields.Char(string='Stage Name', required=True, translate=True)
    sequence = fields.Integer(string='Sequence', default=1)
    mail_template_id = fields.Many2one(
        'mail.template', string='Email Template',
        domain=[('model', '=', 'event.track')],
        help="If set an email will be sent to the customer when the track reaches this step.")
    fold = fields.Boolean(
        string='Folded in Kanban',
        help='This stage is folded in the kanban view when there are no records in that stage to display.')
    is_done = fields.Boolean(string='Accepted Stage')
    is_cancel = fields.Boolean(string='Canceled Stage')


class Track(models.Model):
    _name = "event.track"
    _description = 'Event Track'
    _order = 'priority, date'
    _inherit = ['mail.thread', 'mail.activity.mixin', 'website.seo.metadata', 'website.published.mixin']

    @api.model
    def _get_default_stage_id(self):
        return self.env['event.track.stage'].search([], limit=1).id

    name = fields.Char('Title', required=True, translate=True)
    active = fields.Boolean(default=True)
    user_id = fields.Many2one('res.users', 'Responsible', tracking=True, default=lambda self: self.env.user)
    company_id = fields.Many2one('res.company', related='event_id.company_id')
    partner_id = fields.Many2one('res.partner', 'Speaker')
    partner_name = fields.Char('Name')
    partner_email = fields.Char('Email')
    partner_phone = fields.Char('Phone')
    partner_biography = fields.Html('Biography')
    tag_ids = fields.Many2many('event.track.tag', string='Tags')
    stage_id = fields.Many2one(
        'event.track.stage', string='Stage', ondelete='restrict',
        index=True, copy=False, default=_get_default_stage_id,
        group_expand='_read_group_stage_ids',
        required=True, tracking=True)
    kanban_state = fields.Selection([
        ('normal', 'Grey'),
        ('done', 'Green'),
        ('blocked', 'Red')], string='Kanban State',
        copy=False, default='normal', required=True, tracking=True,
        help="A track's kanban state indicates special situations affecting it:\n"
             " * Grey is the default situation\n"
             " * Red indicates something is preventing the progress of this track\n"
             " * Green indicates the track is ready to be pulled to the next stage")
    description = fields.Html(translate=html_translate, sanitize_attributes=False)
    date = fields.Datetime('Track Date')
    date_end = fields.Datetime('Track End Date', compute='_compute_end_date', store=True)
    duration = fields.Float('Duration', default=1.5, help="Track duration in hours.")
    location_id = fields.Many2one('event.track.location', 'Room')
    event_id = fields.Many2one('event.event', 'Event', required=True)
    color = fields.Integer('Color Index')
    priority = fields.Selection([
        ('0', 'Low'), ('1', 'Medium'),
        ('2', 'High'), ('3', 'Highest')],
        'Priority', required=True, default='1')
    image = fields.Image("Image", related='partner_id.image_128', store=True, readonly=False)

    @api.depends('name')
    def _compute_website_url(self):
        super(Track, self)._compute_website_url()
        for track in self:
            if track.id:
                track.website_url = '/event/%s/track/%s' % (slug(track.event_id), slug(track))

    @api.onchange('partner_id')
    def _onchange_partner_id(self):
        if self.partner_id:
            self.partner_name = self.partner_id.name
            self.partner_email = self.partner_id.email
            self.partner_phone = self.partner_id.phone
            self.partner_biography = self.partner_id.website_description

    @api.depends('date', 'duration')
    def _compute_end_date(self):
        for track in self:
            if track.date:
                delta = timedelta(minutes=60 * track.duration)
                track.date_end = track.date + delta
            else:
                track.date_end = False

    @api.model
    def create(self, vals):
        track = super(Track, self).create(vals)

        track.event_id.message_post_with_view(
            'website_event_track.event_track_template_new',
            values={'track': track},
            subject=track.name,
            subtype_id=self.env.ref('website_event_track.mt_event_track').id,
        )

        return track

    def write(self, vals):
        if 'stage_id' in vals and 'kanban_state' not in vals:
            vals['kanban_state'] = 'normal'
        res = super(Track, self).write(vals)
        if vals.get('partner_id'):
            self.message_subscribe([vals['partner_id']])
        return res

    @api.model
    def _read_group_stage_ids(self, stages, domain, order):
        """ Always display all stages """
        return stages.search([], order=order)

    def _track_template(self, changes):
        res = super(Track, self)._track_template(changes)
        track = self[0]
        if 'stage_id' in changes and track.stage_id.mail_template_id:
            res['stage_id'] = (track.stage_id.mail_template_id, {
                'composition_mode': 'comment',
                'auto_delete_message': True,
                'subtype_id': self.env['ir.model.data'].xmlid_to_res_id('mail.mt_note'),
                'email_layout_xmlid': 'mail.mail_notification_light'
            })
        return res

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'kanban_state' in init_values and self.kanban_state == 'blocked':
            return self.env.ref('website_event_track.mt_track_blocked')
        elif 'kanban_state' in init_values and self.kanban_state == 'done':
            return self.env.ref('website_event_track.mt_track_ready')
        return super(Track, self)._track_subtype(init_values)

    def _message_get_suggested_recipients(self):
        recipients = super(Track, self)._message_get_suggested_recipients()
        for track in self:
            if track.partner_email and track.partner_email != track.partner_id.email:
                track._message_add_suggested_recipient(recipients, email=track.partner_email, reason=_('Speaker Email'))
        return recipients

    def _message_post_after_hook(self, message, msg_vals):
        if self.partner_email and not self.partner_id:
            # we consider that posting a message with a specified recipient (not a follower, a specific one)
            # on a document without customer means that it was created through the chatter using
            # suggested recipients. This heuristic allows to avoid ugly hacks in JS.
            new_partner = message.partner_ids.filtered(lambda partner: partner.email == self.partner_email)
            if new_partner:
                self.search([
                    ('partner_id', '=', False),
                    ('partner_email', '=', new_partner.email),
                    ('stage_id.is_cancel', '=', False),
                ]).write({'partner_id': new_partner.id})
        return super(Track, self)._message_post_after_hook(message, msg_vals)

    def open_track_speakers_list(self):
        return {
            'name': _('Speakers'),
            'domain': [('id', 'in', self.mapped('partner_id').ids)],
            'view_mode': 'kanban,form',
            'res_model': 'res.partner',
            'view_id': False,
            'type': 'ir.actions.act_window',
        }


class SponsorType(models.Model):
    _name = "event.sponsor.type"
    _description = 'Event Sponsor Type'
    _order = "sequence"

    name = fields.Char('Sponsor Type', required=True, translate=True)
    sequence = fields.Integer('Sequence')


class Sponsor(models.Model):
    _name = "event.sponsor"
    _description = 'Event Sponsor'
    _order = "sequence"

    event_id = fields.Many2one('event.event', 'Event', required=True)
    sponsor_type_id = fields.Many2one('event.sponsor.type', 'Sponsoring Type', required=True)
    partner_id = fields.Many2one('res.partner', 'Sponsor/Customer', required=True)
    url = fields.Char('Sponsor Website')
    sequence = fields.Integer('Sequence', store=True, related='sponsor_type_id.sequence', readonly=False)
    image_128 = fields.Image(string="Logo", related='partner_id.image_128', store=True, readonly=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_track
from . import event

```

## File: security\event_track_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="event_track_public" model="ir.rule">
            <field name="name">event tracks: Public</field>
            <field name="model_id" ref="website_event_track.model_event_track"/>
            <field name="domain_force">[('website_published', '=', True)]</field>
            <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="False"/>
            <field name="perm_create" eval="False"/>
            <field name="perm_unlink" eval="False"/>
        </record>

    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_event_location_manager,event.track.location manager,model_event_track_location,event.group_event_manager,1,1,1,1
access_event_track_manager,event.track manager,model_event_track,event.group_event_manager,1,1,1,1
access_event_track_tag_public,event.track.tag public,model_event_track_tag,,1,0,0,0
access_event_track_tag_manager,event.track.tag manager,model_event_track_tag,event.group_event_manager,1,1,1,1
access_event_track_location_public,event.track.location public,model_event_track_location,,1,0,0,0
access_event_track_location_manager,event.track.location manager,model_event_track_location,event.group_event_manager,1,1,1,1
access_event_track__public,event.track.public,model_event_track,,1,0,0,0
access_event_track_sponsor_type_manager,event.track.sponsor.type.public,model_event_sponsor_type,event.group_event_manager,1,1,1,1
access_event_track_sponsor_type_public,event.track.sponsor.type.public,model_event_sponsor_type,,1,0,0,0
access_event_track_sponsor_manager,event.track.sponsor.type.public,model_event_sponsor,event.group_event_manager,1,1,1,1
access_event_track_sponsor_public,event.track.sponsor.public,model_event_sponsor,,1,0,0,0
access_event_track_stage_all,event.track.stage.all,model_event_track_stage,,1,0,0,0
access_event_track_stage_event_manager,event.track.stage.event.manager,model_event_track_stage,event.group_event_manager,1,1,1,1
access_website_event_menu_public,website.event.menu,model_website_event_menu,,1,0,0,0
access_website_event_menu_manager,website.event.menu,model_website_event_menu,event.group_event_manager,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#CD7690"/><stop offset="100%" stop-color="#CA5377"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M34.79 69H4c-2 0-4-.146-4-4.075v-20.97l25-23.86 9-2.038 10 2.037L45 15h12v8.15l-6 5.095 2 8.151-1 7.132-3 5.095L34.79 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M24.762 52.559V50c-3.65-2.952-4.016-8.81-1.095-17.571l9.767 1.485A79.883 79.883 0 0 1 33 32l10-3c3.333 10 3.667 15.667 1 17l1.4 4.898c3.439-3.111 5.6-7.61 5.6-12.612 0-9.39-7.611-17-17-17s-17 7.61-17 17c0 5.982 3.09 11.243 7.762 14.273zm2.566 1.367A16.945 16.945 0 0 0 34 55.286c3.28 0 6.343-.93 8.94-2.538L41 47c-2.868-.574-5.242-4.355-7.123-11.344-.816 9.87-2.759 15-5.83 15.392l-.72 2.878zm15.289-32.579a8.215 8.215 0 1 1 8.68 9.066A18.93 18.93 0 0 1 53 38.286c0 10.493-8.507 19-19 19s-19-8.507-19-19c0-10.494 8.507-19 19-19 3.102 0 6.03.743 8.617 2.061zM49.786 18v3H47v2h2.786v3H52v-3h3v-2h-3v-3h-2.214z" opacity=".3"/><path fill="#FFF" d="M24.762 50.559V48c-3.65-2.952-4.016-8.81-1.095-17.571l9.767 1.485A79.883 79.883 0 0 1 33 30l10-3c3.333 10 3.667 15.667 1 17l1.4 4.898c3.439-3.111 5.6-7.61 5.6-12.612 0-9.39-7.611-17-17-17s-17 7.61-17 17c0 5.982 3.09 11.243 7.762 14.273zm2.566 1.367A16.945 16.945 0 0 0 34 53.286c3.28 0 6.343-.93 8.94-2.538L41 45c-2.868-.574-5.242-4.355-7.123-11.344-.816 9.87-2.759 15-5.83 15.392l-.72 2.878zm15.289-32.579a8.215 8.215 0 1 1 8.68 9.066A18.93 18.93 0 0 1 53 36.286c0 10.493-8.507 19-19 19s-19-8.507-19-19c0-10.494 8.507-19 19-19 3.102 0 6.03.743 8.617 2.061zM49.786 16v3H47v2h2.786v3H52v-3h3v-2h-3v-3h-2.214z"/></g></g></svg>
```

## File: static\description\index.html

```html
<section class="oe_container">
    <div class="oe_row oe_spaced">
        <div class="oe_span12">
            <h2 class="oe_slogan">Organize Events, Trainings &amp; Webinars</h2>
            <h3 class="oe_slogan">Schedule, Promote, Sell, Organize</h3>
        </div>
        <div class="oe_span6">
            <div class="oe_demo oe_picture oe_screenshot">
                <img src="event.png">
            </div>
        </div>
        <div class="oe_span6">
            <p class='oe_mt32'>
                Get extra features per event; multiple pages, sponsors,
                multiple talks, talk proposal form, agenda, event-related news,
                documents (slides of presentations), event-specific menus.
            </p>
        </div>
    </div>
</section>

<section class="oe_container oe_dark">
    <div class="oe_row">
        <h2 class="oe_slogan">Organize Your Tracks</h2>
        <h3 class="oe_slogan">From the talk proposal to the publication</h3>
        <div class="oe_span6">
            <p class='oe_mt32'>
                Add a talk proposal form on your events to allow visitors to
                submit talks and speakers. Organize the validation process
                of every talk, and schedule easily.
            </p><p>
                Odoo's unique frontend and backend integration makes
                organization and publication so easy. Easily design beautiful
                speaker biographies and talks description.
            </p>
        </div>
        <div class="oe_span6">
            <div class="oe_bg_img oe_centered">
                <img class="oe_picture oe_screenshot" src="tracks.png">
            </div>
        </div>
    </div>
</section>

<section class="oe_container">
    <div class="oe_row">
        <h2 class="oe_slogan">Agenda and List of Talks</h2>
        <h3 class="oe_slogan">A strong user interface</h3>
        <div class="oe_span6">
            <img class="oe_picture oe_screenshot" src="tracks.png">
        </div>
        <div class="oe_span6">
           <p class='oe_mt32'>
             Get a beautiful agenda for each event published automatically on
             your website.  Allow your visitors to easily search and browse
             talks, filter by tags, locations or speakers.
           </p>
        </div>
   </div>
</section>

<section class="oe_container oe_dark">
    <div class="oe_row">
        <h2 class="oe_slogan">Manage Sponsors</h2>
        <h3 class="oe_slogan">Sell sponsorship, promote your sponsors</h3>
        <div class="oe_span6">
          <p class='oe_mt32'>
            Add sponsors to your events and publish sponsors per level (e.g.
            bronze, silver, gold) on the bottom of every page of the event.
          </p><p>
            Sell sponsorship packages online through the Odoo eCommerce for
            a full sales cycle integration.
          </p>
        </div>
        <div class="oe_span6">
            <img class="oe_picture oe_screenshot" src="sponsor.png">
        </div>
   </div>
</section>

<section class="oe_container">
    <div class="oe_row">
        <h2 class="oe_slogan">Communicate Efficiently</h2>
        <h3 class="oe_slogan">Activate a blog for some events</h3>
        <div class="oe_span6">
            <img class="oe_picture oe_screenshot" src="blog.png">
        </div>
        <div class="oe_span6">
           <p class='oe_mt32'>
             You can activate a blog for each event allowing you to communicate
             on specific events. Visitors can subscribe to news to get informed.
           </p>
        </div>
   </div>
</section>



```

## File: static\src\js\website_event_track.js

```javascript
odoo.define('website_event_track.website_event_track', function (require) {
'use strict';

var publicWidget = require('web.public.widget');

publicWidget.registry.websiteEventTrack = publicWidget.Widget.extend({
    selector: '.o_wevent_event',
    events: {
        'input #event_track_search': '_onEventTrackSearchInput',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onEventTrackSearchInput: function (ev) {
        ev.preventDefault();

        var text = $(ev.currentTarget).val();
        var filter = _.str.sprintf(':containsLike(%s)', text);

        $('#search_summary').removeClass('invisible');
        var $tracks = $('.event_track');
        $('#search_number').text($tracks.filter(filter).length);
        $tracks.removeClass('invisible').not(filter).addClass('invisible');
    },
});
});

```

## File: views\event_track_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="assets_frontend" inherit_id="website.assets_frontend" name="Website Event Track Assets">
    <xpath expr="." position="inside">
        <link rel="stylesheet" href="/website_event_track/static/src/css/website_event_track.css"/>
        <script type="text/javascript" src="/website_event_track/static/src/js/website_event_track.js"></script>
    </xpath>
</template>

<template name="Sponsors" id="event_sponsor" customize_show="True" inherit_id="website_event.layout">
    <xpath expr="//div[@id='wrap']" position="inside">
        <div class="container mt32 mb16 d-print-none" t-if="event.sponsor_ids">
            <section>
                <h2 class="text-center mb32">Our Sponsors</h2>
            </section>
            <div class="row">
                <div t-attf-class="col-sm-6 col-lg-#{(len(event.sponsor_ids) > 6) and 2 or (12 // len(event.sponsor_ids))} text-center mb16 oe_sponsor" t-foreach="event.sponsor_ids" t-as="sponsor">
                    <t t-if="sponsor.url">
                        <a t-att-href="sponsor.url" style="position: relative; display: inline-block;" class="bg-white">
                            <span t-field="sponsor.image_128"
                                t-options='{"widget": "image", "class": "shadow"}'/>
                            <div class="ribbon-wrapper">
                                <div t-field="sponsor.sponsor_type_id" t-attf-class="ribbon ribbon_#{sponsor.sponsor_type_id.name}"/>
                            </div>
                        </a>
                    </t>
                    <t t-if="not sponsor.url">
                        <span style="position: relative; display: inline-block;" class="bg-white">
                            <span t-field="sponsor.image_128"
                                t-options='{"widget": "image", "class": "shadow"}'/>
                            <div class="ribbon-wrapper">
                                <div t-field="sponsor.sponsor_type_id" t-attf-class="ribbon ribbon_#{sponsor.sponsor_type_id.name}"/>
                            </div>
                        </span>
                    </t>
                </div>
            </div>
        </div>
    </xpath>
</template>

<template id="agenda">
    <t t-call="website_event.layout">
        <div class="oe_structure" id="oe_structure_website_event_track_agenda_1"/>
        <section class="container pt48">
            <h1 t-field="event.name"/>
            <div class="form-inline float-right">
                <label class="invisible text-muted" id="search_summary"><span id="search_number">0</span> Found </label>
                <input type="text" class="form-control" placeholder="Filter Tracks..." id="event_track_search"/>
            </div>
        </section>
        <t t-set="dayslist" t-value="list(days.keys())"/>
        <t t-set="dayslist2" t-value="dayslist.sort()"/> <!-- display days in the right order -->
        <section class="container" t-foreach="dayslist" t-as="day">
            <t t-set="locations" t-value="days[day]['locations']"/>
            <t t-set="dates" t-value="days[day]['dates']"/>
            <h3 class="o_page_header mt0">
                <span t-field="tracks_by_days[day][0].date" t-options="{'widget': 'date'}"/>
                <small><t t-esc="len(tracks_by_days[day])"/> talks</small>
            </h3>
            <table id="table_search" class="table table-bordered table-sm">
            <tr>
                <th/>
                <t t-foreach="locations.keys()" t-as="location">
                    <th t-if="location" class="active">
                        <span t-esc="location and location.name or 'Unknown'"/>
                    </th>
                </t>
            </tr>
            <tr t-foreach="dates" t-as="dt">
                <td class="active">
                    <b t-esc="dt[-1]"/>
                </td>
                <t t-if="dt[2]"> <!-- Not a multi location -->
                    <t t-foreach="locations" t-as="location">
                        <t t-if="location and dt[1].get(location, False)">
                            <t t-set="track" t-value="dt[1][location][0]"/>
                            <t t-if="track">
                                <td t-att-rowspan="dt[1][location][3]" t-attf-class="event_color_#{track.color} #{track and 'event_track' or ''}">
                                    <t t-if="track">
                                        <a t-attf-href="/event/#{ slug(event) }/track/#{ slug(track) }">
                                            <span t-esc="track and track.name"/>
                                            <span t-if="not track.website_published" class="badge badge-warning">unpublished</span>
                                        </a>
                                        <div class="text-muted">
                                            <small t-if="track.partner_id" t-esc="track.partner_id.sudo().name"/>
                                            <small t-if="not track.partner_id" t-esc="track.partner_name"/>
                                        </div>
                                    </t>
                                </td>
                            </t>
                            <td t-if="not track" t-att-rowspan="dt[1][location][3]" class="event_track"/>
                        </t>
                    </t>
                </t><t t-if="not dt[2]">
                    <t t-set="track" t-value="dt[1][False][0]"/>
                    <td t-att-colspan="len(locations)-1" t-attf-class="text-center event_color_#{track.color} #{track and 'event_track' or ''}">
                        <a t-attf-href="/event/#{ slug(event) }/track/#{ slug(track) }">
                            <span t-esc="track.name"/>
                        </a>
                        <div class="text-muted">
                            <small t-if="track.partner_id" t-esc="track.partner_id.sudo().name"/>
                            <small t-if="not track.partner_id" t-esc="track.partner_name"/>
                        </div>
                    </td>
                </t>
            </tr>
            </table>
        </section>
        <div class="oe_structure" id="oe_structure_website_event_track_agenda_2"/>
    </t>
</template>

<template id="tracks">
    <t t-call="website_event.layout">
    <div class="oe_structure" id="oe_structure_website_event_track_tracks_1"/>
        <section class="pt48 pb48">
            <div class="container">
                <div class="row">
                    <div id="left_column">
                    </div>
                    <div class="col-lg-9">
                        <div class="alert alert-warning" t-if="not len(tracks)" role="alert">
                            No tracks found!
                        </div>
                        <div class="row mb-2" t-foreach="tracks" t-as="track">
                            <div class="col-md-2">
                                <t t-if="track.image">
                                    <span t-field="track.image"
                                        t-options='{"widget": "image", "class": "rounded-circle"}'/>
                                </t>
                            </div>
                            <div class="col-md-10">
                                <h3 class="mt0 mb0">
                                    <a t-attf-href="/event/#{ slug(event) }/track/#{ slug(track) }" class="text-secondary"><span t-field="track.name"> </span></a>
                                    <small t-if="not track.website_published" class="badge badge-danger">unpublished</small>
                                </h3>
                                <ul class="list-inline mb0">
                                    <li t-if="track.partner_id" class="list-inline-item">
                                        <i class="fa fa-user text-primary"/>
                                        <t t-esc="track.partner_id.sudo().name"/>
                                    </li>
                                    <li t-if="track.date" class="list-inline-item">
                                        <i class="fa fa-calendar text-primary"/>
                                        <span t-field="track.date" t-options='{"hide_seconds":"True"}'/>
                                    </li>
                                    <li class="list-inline-item" t-if="track.location_id">
                                        <i class="fa fa-map-marker"/>
                                        <span t-field="track.location_id"/>
                                    </li>
                                </ul>
                                <ul class="list-inline">
                                    <li t-foreach="track.tag_ids" t-as="tag_id" class="list-inline-item fa fa-tags" t-if="tag_id.color != 0">
                                        <a t-attf-href="/event/#{ slug(event) }/track/tag/#{ slug(tag_id) }">
                                            <i class="fa fa-tags"/>
                                            <span t-field="tag_id.name"/>
                                        </a>
                                    </li>
                                </ul>
                                <p class="mt8"><t t-esc="html2plaintext(track.description or '')[0:500]"/><t t-if="track.description">...</t></p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>
        <div class="oe_structure" id="oe_structure_website_event_track_tracks_2"/>
    </t>
</template>

<template id="tracks_filter" inherit_id="website_event_track.tracks" customize_show="True" name="Filter on Tags">
    <xpath expr="//div[@id='left_column']" position="inside">
        <ul class="nav nav-pills flex-column">
            <li class="nav-item"><a t-attf-href="/event/#{ slug(event) }/track" t-attf-class="nav-link #{'' if searches.get('tag') else ' active'}">All Tags</a></li>
            <t t-foreach="tags" t-as="tag">
                <li class="nav-item">
                    <a t-attf-href="/event/#{ slug(event) }/track/tag/#{ slug(tag) }" t-attf-class="nav-link#{searches.get('tag') == tag.id and ' active' or ''}">
                        <t t-esc="tag.name"/>
                    </a>
                </li>
            </t>
        </ul>
    </xpath>
    <xpath expr="//div[@id='left_column']" position="attributes">
        <attribute name="class">col-lg-3 css_no_print</attribute>
    </xpath>
</template>


<template id="track_edit_options" inherit_id="website.user_navbar" name="Edit Track Options">
    <xpath expr="//li[@id='edit-page-menu']" position="after">
        <t t-if="main_object._name == 'event.track'" t-set="action" t-value="'website_event_track.action_event_track'" />
    </xpath>
</template>
<template id="track_view">
    <t t-call="website_event.layout">
        <section class="pt48">
            <div class="container">
                <div class="row">
                    <div class="col-12">
                        <h1 t-field="track.name"/>
                        <small>Proposed by
                            <span t-if="track.partner_id" t-field="track.partner_id.sudo().name"/>
                            <span t-if="not track.partner_id" t-field="track.partner_name"/>
                        </small>
                        <ul t-if="track.tag_ids" class="text-center text-muted list-inline">
                            <li t-foreach="track.tag_ids" t-as="tag_id" class="list-inline-item" t-if="tag_id.color != 0">
                                <span class="fa fa-tags"></span>
                                <a t-attf-href="/event/#{ slug(event) }/track/tag/#{ slug(tag_id) }">
                                    <span t-field="tag_id.name"/>
                                </a>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
        </section>
        <section class="pt32">
            <div class="container">
                <div class="row">
                    <div class="col-lg-8">
                        <div t-field="track.description"/>
                        <h3>About The Speaker</h3>
                        <div t-field="track.partner_biography"/>
                        <div t-if="track.partner_id" class="row mt32">
                            <div class="col-md-2">
                                <span t-field="track.partner_id.sudo().image_128" t-options='{"widget": "image", "class": "rounded-circle"}'/>
                            </div>
                            <div class="col-md-10">
                                <h4 t-field="track.partner_id.name" class="mb4"/>
                                <div class="mb16" t-if="track.partner_id.website">
                                    <i class="fa fa-home mr-2"/><a t-att-href="track.partner_id.website"><span t-field="track.partner_id.website"/></a>
                                </div>
                                <div t-field="track.partner_id.website_description"/>
                            </div>
                        </div>
                    </div>
                    <div class="col-lg-4" id="right_column">
                        <div class="card">
                            <h4 class="card-header">Practical Info</h4>
                            <div class="card-body">
                                <b>Date</b><br/>
                                <span t-field="track.date" t-options='{"hide_seconds":"True"}'/>
                                <t t-if="event.date_tz">(<span t-field="event.date_tz"/>)</t>
                                <br/>
                                <b>Duration</b><br/>
                                <span t-field="track.duration" t-options='{"widget": "duration", "unit": "hour", "round": "minute"}'/><br/>
                                <b>Location</b><br/>
                                <span t-field="track.location_id"/><br/>
                            </div>
                        </div>

                        <div class="card" t-if="False">
                            <h4 class="card-header">Documents</h4>
                            <div class="card-body">
                                Put here the list of documents, like slides of
                                the presentations. Remove the above t-if when
                                it's implemented.
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>
    </t>
</template>

<template id="event_track_social" name="Social Widgets" inherit_id="website_event_track.track_view" active="False" customize_show="True">
    <xpath expr="//div[@id='right_column']" position="inside">
        <div class="card">
            <h4 class="card-header">Social Stream</h4>
            <div class="card-body">
                <t t-call="website_mail.follow"><t t-set="object" t-value="event"/></t>
                <div t-if="event.twitter_hashtag" class="mt16">
                    <p><strong>Participate on Twitter</strong></p>
                    <p class="text-muted">
                        Find out what people see and say about this event,
                        and join the conversation.
                    </p>
                    <p><strong>Use this tag:
                        <a t-att-href="'http://twitter.com/search?q=#'+event.twitter_hashtag" class="badge badge-primary">#<span t-field="event.twitter_hashtag"/></a>
                    </strong></p>
                </div>
            </div>
        </div>
    </xpath>
</template>

<template id="event_track_proposal">
    <t t-call="website_event.layout">
        <div class="oe_structure" id="oe_structure_website_event_track_proposal_1"/>
        <div class="container">
            <section class="pt48">
                <h1 class="o_page_header">Call for Proposals</h1>
                <h2 class="text-center text-secondary font-weight-bold my-4" t-esc="event.name"/>
            </section>
            <section id="forms" t-if="not event.website_track_proposal">
                <h1>Proposals are closed!</h1>
                <p>
                    This event does not accept proposals.
                </p>
            </section>
            <section class="row">
                <div class="col-lg-9">
                    <div class="oe_structure">
                        <section>
                            <h3 class="o_page_header mt16">
                                Introduction
                            </h3>
                            <p>
                                We will accept a broad range of
                                presentations, from reports on academic and
                                commercial projects to tutorials and case
                                studies. As long as the presentation is
                                interesting and potentially useful to the
                                audience, it will be considered for
                                inclusion in the programme.
                            </p>
                        </section>
                        <section class="mt-5">
                            <h3 class="o_page_header">Application</h3>
                            <p>
                                Fill this form to propose your talk.
                            </p>
                            <div class="alert alert-info" role="alert">
                                <i class="fa fa-info-circle"/>
                                You can add multiple speakers by separating names, emails and phones with commas.
                            </div>
                        </section>
                    </div>
                    <section id="forms" t-if="event.website_track_proposal">
                        <form class="mt32 js_website_submit_form" t-attf-action="/event/#{event.id}/track_proposal/post" method="post" enctype="multipart/form-data">
                            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                            <div class="form-group row">
                                <label class="col-lg-3 col-md-4 col-form-label" for="track_name">Talk Title</label>
                                <div class="col-lg-7 col-md-8">
                                    <input type="text" class="form-control" name="track_name" required="True"/>
                                </div>
                            </div>
                            <div class="form-group row">
                                <label class="col-lg-3 col-md-4 col-form-label" for="description">Talk Introduction</label>
                                <div class="col-lg-7 col-md-8">
                                    <textarea  class="form-control" name="description" style="min-height: 120px"/>
                                </div>
                            </div>
                            <div class="form-group row" t-if="len(event.allowed_track_tag_ids)">
                                <label class="col-lg-3 col-md-4 col-form-label" for="phone">Tags</label>
                                <div class="col-lg-9 col-md-8">
                                    <div class="row">
                                        <label class="col-md-4" t-foreach="event.allowed_track_tag_ids" t-as="tag">
                                            <input type="checkbox" value="1" t-attf-name="tag_#{str(tag.id)}"/>
                                            <t t-esc="tag.name"/>
                                        </label>
                                    </div>
                                </div>
                            </div>
                            <div class="form-group row">
                                <label class="col-lg-3 col-md-4 col-form-label" for="partner_name">Speaker(s) Name</label>
                                <div class="col-lg-7 col-md-8">
                                    <input type="text" class="form-control" name="partner_name" required="True"/>
                                </div>
                            </div>
                            <div class="form-group row">
                                <label class="col-lg-3 col-md-4 col-form-label" for="email_from">Speaker(s) Email</label>
                                <div class="col-lg-7 col-md-8">
                                    <input type="email" class="form-control" name="email_from" required="True" multiple="multiple"/>
                                </div>
                            </div>
                            <div class="form-group row">
                                <label class="col-lg-3 col-md-4 col-form-label" for="phone">Speaker(s) Phone</label>
                                <div class="col-lg-7 col-md-8">
                                    <input type="text" class="form-control" name="phone" required="True"/>
                                </div>
                            </div>
                            <div class="form-group row">
                                <label class="col-lg-3 col-md-4 col-form-label" for="biography">Speaker(s) Biography</label>
                                <div class="col-lg-7 col-md-8">
                                    <textarea  class="form-control" name="biography" rows="5"/>
                                </div>
                            </div>
                            <div class="form-group row o_form_buttons">
                                <div class="offset-lg-3 offset-md-4 col-md-8 col-lg-7">
                                    <button type="submit" class="btn btn-primary">Submit Proposal</button>
                                </div>
                            </div>
                        </form>
                    </section>
                    <div class="oe_structure" id="oe_structure_website_event_track_proposal_2"/>
                </div>
                <div class="col-lg-3">
                    <div class="card mb-3 bg-secondary">
                        <h4 class="card-header">Talks Types</h4>
                        <div class="card-body">
                            <ul class="list-unstyled">
                                <li>
                                    <strong>Regular Talks</strong>. These are standard talks with slides,
                                    alocated in slots of 60 minutes.
                                </li><li>
                                    <strong>Lightning Talks</strong>. These are 30 minutes talks on many
                                    different topics. Most topics are accepted in lightning talks.
                                </li>
                            </ul>
                        </div>
                    </div>
                    <div class="card bg-secondary">
                        <h4 class="card-header">Submission Agreement</h4>
                        <div class="card-body">
                            <p>
                            We require speakers to accept an agreement in which they commit to:
                            </p>
                            <ul class="list-unstyled">
                                <li>
                                    Timely release of presentation material (slides),
                                    for publishing on our website.
                                </li><li>
                                    Allow video and audio recording of their
                                    presentation, for publishing on our website.
                                </li>
                            </ul>
                        </div>
                    </div>
                </div>
            </section>
        </div>
        <div class="oe_structure" id="oe_structure_website_event_track_proposal_3"/>
    </t>
</template>

<template id="event_track_proposal_success">
    <t t-call="website_event.event_details">
        <p class="text-center mt-2">
            Thank you for your proposal.
        </p><p class="text-center">
            We will evaluate your proposition and get back to you shortly.
        </p>
    </t>
</template>

<!-- Chatter templates -->
<template id="event_track_template_new">
    <p>New track proposal <a href="#" t-att-data-oe-model="track._name" t-att-data-oe-id="track.id"> <t t-esc="track.name"/></a></p>
    <ul>
        <li><b>Proposed By</b>: <t t-esc="track.partner_name or track.partner_email"/></li>
        <li><b>Mail</b>: <a t-attf-href="mailto:#{track.partner_email}"><t t-esc="track.partner_email"/></a></li>
        <li><b>Phone</b>: <t t-esc="track.partner_phone"/></li>
        <li><b>Speakers Biography</b>: <t t-esc="track.partner_biography"/></li>
        <li><b>Talk Introduction</b>: <t t-esc="track.description"/></li>
    </ul>
</template>

</odoo>

```

## File: views\event_track_views.xml

```xml
<?xml version="1.0"?>
<odoo>

        <!-- Event Tracks -->
        <record model="ir.ui.view" id="view_event_track_kanban">
            <field name="name">event.track.kanban</field>
            <field name="model">event.track</field>
            <field name="arch" type="xml">
                <kanban default_group_by="stage_id">
                    <templates>
                        <field name="color"/>
                        <field name="partner_id"/>
                        <field name="stage_id"/>
                        <field name="website_url"/>
                        <field name="activity_ids"/>
                        <field name="activity_state"/>
                        <progressbar field="kanban_state" colors='{"done": "success", "blocked": "danger"}'/>
                        <t t-name="kanban-box">
                            <div t-attf-class="{{!selection_mode ? 'oe_kanban_color_' + kanban_getcolor(record.color.raw_value) : ''}} oe_kanban_card oe_kanban_global_click">
                                <div class="o_dropdown_kanban dropdown" groups="base.group_user">

                                    <a role="button" class="dropdown-toggle o-no-caret btn" data-toggle="dropdown" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                                        <span class="fa fa-ellipsis-v"/>
                                    </a>
                                    <div class="dropdown-menu" role="menu">
                                        <a role="menuitem" t-att-href="record.website_url.value" class="dropdown-item">View Track</a>
                                        <t t-if="widget.editable"><a role="menuitem" type="edit" class="dropdown-item">Edit Track</a></t>
                                        <t t-if="widget.deletable"><a role="menuitem" type="delete" class="dropdown-item">Delete</a></t>
                                        <ul class="oe_kanban_colorpicker" data-field="color"/>
                                    </div>
                                </div>
                                <div class="oe_kanban_content">
                                    <div class="o_kanban_record_top">
                                        <h4 class="o_kanban_record_title"><field name="name"/></h4>
                                    </div>
                                    <div class="o_kanban_record_body">
                                        <t t-if="duration"><field name="duration" widget="float_time"/> hours</t>
                                        <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                    </div>
                                    <div class="o_kanban_record_bottom">
                                        <div class="oe_kanban_bottom_left">
                                            <field name="priority" widget="priority"/>
                                            <field name="activity_ids" widget="kanban_activity"/>
                                        </div>
                                        <div class="oe_kanban_bottom_right">
                                            <field name="kanban_state" widget="state_selection" groups="base.group_user"/>
                                            <img t-att-src="kanban_image('res.partner', 'image_128', record.partner_id.raw_value)"
                                                t-att-title="record.partner_id.value" t-att-alt="record.partner_id.value"
                                                class="oe_kanban_avatar"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="view_event_track_calendar" model="ir.ui.view">
            <field name="name">event.track.calendar</field>
            <field name="model">event.track</field>
            <field eval="2" name="priority"/>
            <field name="arch" type="xml">
                <calendar date_start="date" date_delay="duration" string="Event Tracks" color="location_id" event_limit="5">
                    <field name="location_id"/>
                    <field name="event_id"/>
                    <field name="partner_id" avatar_field="image_128"/>
                    <field name="user_id" avatar_field="image_128"/>
                </calendar>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_track_search">
            <field name="name">event.track.search</field>
            <field name="model">event.track</field>
            <field name="arch" type="xml">
                <search string="Event Tracks">
                    <field name="name"/>
                    <field name="event_id"/>
                    <field name="location_id"/>
                    <field name="stage_id"/>
                    <field name="partner_id"/>
                    <filter string="My Tracks" name="my_tracks" domain="[('user_id', '=', uid)]"/>
                    <separator/>
                    <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction', '=', True)]"/>
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
                        <filter string="Stage" name="stage" context="{'group_by': 'stage_id'}"/>
                        <filter string="Date" name="date" context="{'group_by': 'date'}"/>
                        <filter string="Event" name="event" context="{'group_by': 'event_id'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_track_form">
            <field name="name">event.track.form</field>
            <field name="model">event.track</field>
            <field name="arch" type="xml">
                <form string="Event Track">
                    <header>
                        <field name="stage_id" widget="statusbar" options="{'clickable': '1'}"/>
                    </header>
                    <sheet string="Track">
                        <div class="oe_button_box" name="button_box">
                            <field name="website_url" invisible="1"/>
                            <field name="is_published" widget="website_redirect_button"/>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <field name="kanban_state" widget="state_selection" class="float-right"/>
                        <div class="oe_title">
                            <label for="name" class="oe_edit_only"/>
                            <h1>
                                <field name="name" placeholder="e.g. Inspiring Business Talk"/>
                            </h1>
                        </div>
                        <group>
                            <group>
                                <field name="date"/>
                                <field name="location_id"/>
                                <label for="duration"/>
                                <div class="o_row">
                                    <field name="duration" widget="float_time"/>
                                    <span>hours</span>
                                </div>
                                <field name="active" invisible="1"/>
                            </group>
                            <group>
                                <field name="company_id" invisible="1"/>
                                <field name="user_id"/>
                                <field name="event_id"/>
                                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                            </group>
                        </group>
                        <notebook>
                            <page string="Speakers">
                                <group>
                                    <field name="partner_id" domain="['|', ('company_id', '=', company_id), ('company_id', '=', False)]"/>
                                    <field name="partner_name"/>
                                    <field name="partner_email"/>
                                    <field name="partner_phone" class="o_force_ltr"/>
                                    <field name="partner_biography"/>
                                </group>
                            </page>
                            <page string="Description">
                                <field name="description"/>
                            </page>
                        </notebook>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" widget="mail_followers"/>
                        <field name="activity_ids" widget="mail_activity"/>
                        <field name="message_ids" widget="mail_thread" options="{'post_refresh': 'recipients'}"/>
                    </div>
                </form>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_track_tree">
            <field name="name">event.track.tree</field>
            <field name="model">event.track</field>
            <field name="arch" type="xml">
                <tree string="Event Track">
                    <field name="name"/>
                    <field name="active" invisible="1"/>
                    <field name="event_id"/>
                    <field name="activity_exception_decoration" widget="activity_exception"/>
                </tree>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_track_graph">
            <field name="name">event.track.graph</field>
            <field name="model">event.track</field>
            <field name="arch" type="xml">
                <graph string="Tracks" type="bar" orientation="horizontal">
                    <field name="location_id"/>
                    <field name="duration" operator="+"/>
                </graph>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_event_track">
            <field name="name">Event Tracks</field>
            <field name="res_model">event.track</field>
            <field name="view_mode">kanban,tree,form,calendar,graph,activity</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                  Add a new track
                </p><p>
                  Tracks define the schedule of your event. These can be a talk, a round table, a meeting, etc.
                </p>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_event_track_from_event">
            <field name="res_model">event.track</field>
            <field name="name">Event Tracks</field>
            <field name="view_mode">kanban,tree,form,calendar,graph,activity</field>
            <field name="context">{'search_default_event_id': active_id, 'default_event_id': active_id, 'group_by': 'stage_id'}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Add a new track
              </p><p>
                Tracks define the schedule of your event. These can be a talk, a round table, a meeting, etc.
              </p>
            </field>
        </record>

        <menuitem
            id="menu_event_track"
            name="Event Tracks"
            action="action_event_track"
            parent="event.event_main_menu"
            groups="base.group_no_one"/>

        <!-- EVENTS/CONFIGURATION/EVENT locations -->
        <record model="ir.ui.view" id="view_event_location_form">
            <field name="name">Event Locations</field>
            <field name="model">event.track.location</field>
            <field name="arch" type="xml">
                <form string="Event Location">
                    <sheet>
                        <group>
                            <field name="name"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_location_tree">
            <field name="name">Event Location</field>
            <field name="model">event.track.location</field>
            <field name="arch" type="xml">
                <tree string="Event Location" editable="bottom">
                    <field name="name" required="True"/>
                </tree>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_event_track_location">
            <field name="name">Event Locations</field>
            <field name="res_model">event.track.location</field>
        </record>

        <menuitem
            id="menu_event_track_location"
            name="Locations"
            action="action_event_track_location"
            parent="event.menu_event_configuration"/>

        <!-- EVENTS/CONFIGURATION/EVENT Sponsor Types -->
        <record model="ir.ui.view" id="view_event_sponsor_type_form">
            <field name="name">Sponsor Types</field>
            <field name="model">event.sponsor.type</field>
            <field name="arch" type="xml">
                <form string="Event Sponsor Types">
                    <sheet>
                        <group>
                            <field name="sequence"/>
                            <field name="name"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_sponsor_type_tree">
            <field name="name">Sponsor Types</field>
            <field name="model">event.sponsor.type</field>
            <field name="arch" type="xml">
                <tree string="Event Sponsor Type" editable="bottom">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                </tree>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_event_sponsor_type">
            <field name="name">Sponsor Types</field>
            <field name="res_model">event.sponsor.type</field>
        </record>

        <menuitem
            id="menu_event_sponsor_type"
            action="action_event_sponsor_type"
            parent="event.menu_event_configuration"
            groups="base.group_no_one"/>

        <!-- EVENT.SPONSOR VIEWS -->
        <record model="ir.ui.view" id="view_event_sponsor_tree">
            <field name="name">event.sponsor.tree</field>
            <field name="model">event.sponsor</field>
            <field name="arch" type="xml">
                <tree editable="bottom">
                    <field name="partner_id"/>
                    <field name="url"/>
                    <field name="sponsor_type_id"/>
                </tree>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_sponsor_search">
            <field name="name">event.sponsor.search</field>
            <field name="model">event.sponsor</field>
            <field name="arch" type="xml">
                <search string="Event Sponsors">
                    <field name="partner_id"/>
                    <field name="event_id"/>
                </search>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_event_sponsor_from_event">
            <field name="name">Event Tracks</field>
            <field name="res_model">event.sponsor</field>
            <field name="view_mode">tree,form</field>
            <field name="context">{'search_default_event_id': active_id, 'default_event_id': active_id}</field>
        </record>

        <!-- EVENTS/CONFIGURATION/EVENT Tags -->
        <record model="ir.ui.view" id="view_event_track_tag_form">
            <field name="name">Track Tags</field>
            <field name="model">event.track.tag</field>
            <field name="arch" type="xml">
                <form string="Event Track Tag">
                    <sheet>
                        <group>
                            <field name="name"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record model="ir.ui.view" id="view_event_track_tag_tree">
            <field name="name">Tracks Tag</field>
            <field name="model">event.track.tag</field>
            <field name="arch" type="xml">
                <tree string="Event Track Tag" editable="bottom">
                    <field name="name"/>
                </tree>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_event_track_tag">
            <field name="name">Track Tags</field>
            <field name="res_model">event.track.tag</field>
        </record>

        <menuitem
            id="menu_event_track_tag"
            action="action_event_track_tag"
            parent="event.menu_event_configuration"
            groups="base.group_no_one"/>

        <!-- EVENTS TRACK STAGES-->
        <record id="event_track_stage_view_search" model="ir.ui.view">
            <field name="name">event.track.stage.view.search</field>
            <field name="model">event.track.stage</field>
            <field name="arch" type="xml">
                <search string="Track Stage">
                   <field name="name" string="Track Stages"/>
                </search>
            </field>
        </record>

        <record id="event_track_stage_view_form" model="ir.ui.view">
            <field name="name">event.track.stage.view.form</field>
            <field name="model">event.track.stage</field>
            <field name="arch" type="xml">
                <form string="Track Stage">
                    <sheet>
                        <group>
                            <group>
                                <field name="name"/>
                                <field name="mail_template_id"/>
                                <field name="sequence" groups="base.group_no_one"/>
                            </group>
                            <group>
                                <field name="fold"/>
                                <field name="is_done"/>
                                <field name="is_cancel"/>
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="event_track_stage_view_tree" model="ir.ui.view">
            <field name="name">event.track.stage.view.tree</field>
            <field name="model">event.track.stage</field>
            <field name="arch" type="xml">
                <tree string="Track Stage">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                    <field name="fold"/>
                </tree>
            </field>
        </record>

        <record id="view_event_track_stage_kanban" model="ir.ui.view">
            <field name="name">event.track.stage.kanban</field>
            <field name="model">event.track.stage</field>
            <field name="arch" type="xml">
                <kanban>
                    <field name="name"/>
                    <field name="fold"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_global_click">
                                <div>
                                    <strong class="o_kanban_record_title"><field name="name"/></strong>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="event_track_stage_action" model="ir.actions.act_window">
            <field name="name">Track Stages</field>
            <field name="res_model">event.track.stage</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="view_id" ref="event_track_stage_view_tree"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Add a new stage in the task pipeline
              </p><p>
                Define the steps that will be used in the event from the
                creation of the track, up to the closing of the track.
                You will use these stages in order to track the progress in
                solving an event track.
              </p>
            </field>
        </record>

        <menuitem name="Track Stages"
            id="event_track_stage_menu"
            action="event_track_stage_action"
            parent="event.menu_event_configuration"
            groups="base.group_no_one"/>

</odoo>

```

## File: views\event_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="event_type_view_form_inherit_track" model="ir.ui.view">
        <field name="name">event.type.view.form.inherit.track</field>
        <field name="model">event.type</field>
        <field name="inherit_id" ref="website_event.event_type_view_form_inherit_website"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='event_type_visibility_website']/div[hasclass('o_setting_right_pane')]" position='inside'>
                <div class="row mt16" attrs="{'invisible': [('website_menu', '=', False)]}">
                    <label class="col-lg-4" for="website_track"/> <field name="website_track"/>
                </div>
                <div class="row mt16" attrs="{'invisible': [('website_menu', '=', False)]}">
                    <label class="col-lg-4" for="website_track_proposal"/> <field name="website_track_proposal"/>
                </div>
            </xpath>
        </field>
    </record>

        <!-- Events Organisation/CONFIGURATION/EVENTS -->
        <record id="view_event_form" model="ir.ui.view">
            <field name="name">Event Tracks</field>
            <field name="inherit_id" ref="website_event.event_event_view_form_inherit_website"/>
            <field name="model">event.event</field>
            <field name="arch" type="xml">
                <field name="is_published" position="before">
                    <button name="%(action_event_track_from_event)d"
                            type="action"
                            class="oe_stat_button"
                            icon="fa-inbox">
                        <field name="track_count" string="Tracks" widget="statinfo"/>
                    </button>
                    <button name="%(action_event_sponsor_from_event)d"
                            type="action"
                            class="oe_stat_button"
                            icon="fa-users">
                        <field name="sponsor_count" string="Sponsors" widget="statinfo"/>
                    </button>
                </field>
                <label for="website_menu" position="after">
                    <field name="website_track" attrs="{'invisible': [('website_menu', '=', False)]}"/>
                    <label for="website_track" string="Tracks on Website" attrs="{'invisible': [('website_menu', '=', False)]}"/>
                    <field name="website_track_proposal" attrs="{'invisible': [('website_menu', '=', False)]}"/>
                    <label for="website_track_proposal" string="Track Proposals on Website" attrs="{'invisible': [('website_menu', '=', False)]}"/>
                </label>
            </field>
        </record>
</odoo>

```

