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
    'version': '1.3',
    'depends': ['website_event'],
    'data': [
        'security/ir.model.access.csv',
        'security/event_track_security.xml',
        'data/event_data.xml',
        'data/mail_template_data.xml',
        'data/mail_templates.xml',
        'data/mail_message_subtype_data.xml',
        'data/event_track_data.xml',
        'views/event_templates.xml',
        'views/event_track_templates_agenda.xml',
        'views/event_track_templates_list.xml',
        'views/event_track_templates_reminder.xml',
        'views/event_track_templates_page.xml',
        'views/event_track_templates_proposal.xml',
        'views/event_track_views.xml',
        'views/event_track_location_views.xml',
        'views/event_track_tag_views.xml',
        'views/event_track_stage_views.xml',
        'views/event_track_visitor_views.xml',
        'views/event_event_views.xml',
        'views/event_type_views.xml',
        'views/res_config_settings_view.xml',
        'views/website_visitor_views.xml',
        'views/event_menus.xml',
        'views/snippets.xml',
    ],
    'demo': [
        'data/event_demo.xml',
        'data/event_track_location_demo.xml',
        'data/event_track_tag_demo.xml',
        'data/event_track_demo.xml',
        'data/event_track_demo_description.xml',
        'data/event_track_visitor_demo.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'website_event_track/static/src/scss/event_track_templates.scss',
            'website_event_track/static/src/scss/event_track_templates_online.scss',
            'website_event_track/static/src/scss/pwa_frontend.scss',
            'website_event_track/static/src/js/website_event_track.js',
            'website_event_track/static/src/js/website_event_track_proposal_form.js',
            'website_event_track/static/src/js/website_event_track_proposal_form_tags.js',
            'website_event_track/static/src/js/event_track_reminder.js',
            'website_event_track/static/src/js/event_track_timer.js',
            'website_event_track/static/src/js/website_event_pwa_widget.js',
            'website_event_track/static/lib/idb-keyval/idb-keyval.js',
            'website_event_track/static/src/xml/event_track_proposal_templates.xml',
            'website_event_track/static/src/xml/website_event_pwa.xml',
            'website_event_track/static/src/xml/website_event_track_form_tags_wrapper.xml',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.website_event.controllers.main import WebsiteEventController


class EventOnlineController(WebsiteEventController):

    def _get_registration_confirm_values(self, event, attendees_sudo):
        values = super(EventOnlineController, self)._get_registration_confirm_values(event, attendees_sudo)
        values['hide_sponsors'] = True
        return values

```

## File: controllers\event_track.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval
from collections import defaultdict
from datetime import timedelta
from pytz import timezone, utc
from werkzeug.exceptions import Forbidden, NotFound

import babel
import babel.dates
import base64
import json
import operator
import pytz

from odoo import exceptions, http, fields, tools, _
from odoo.http import request
from odoo.osv import expression
from odoo.tools import is_html_empty, plaintext2html
from odoo.tools.misc import babel_locale_parse


class EventTrackController(http.Controller):

    def _get_event_tracks_agenda_domain(self, event):
        """ Base domain for displaying track names (preview). The returned search
        domain will select the tracks that belongs to a track stage that should
        be visible in the agenda (see: 'is_visible_in_agenda'). Published tracks
        are also displayed whatever their stage. """
        agenda_domain = [
            '&',
            ('event_id', '=', event.id),
            '|',
            ('is_published', '=', True),
            ('stage_id.is_visible_in_agenda', '=', True),
        ]
        return agenda_domain

    def _get_event_tracks_domain(self, event):
        """ Base domain for displaying tracks. The returned search domain will
        select the tracks that belongs to a track stage that should be visible
        in the agenda (see: 'is_visible_in_agenda'). When the user is a visitor,
        the domain will contain an additional condition that will remove the
        unpublished tracks from the search results."""
        search_domain_base = self._get_event_tracks_agenda_domain(event)
        if not request.env.user.has_group('event.group_event_registration_desk'):
            search_domain_base = expression.AND([
                search_domain_base,
                [('is_published', '=', True)]
            ])
        return search_domain_base

    # ------------------------------------------------------------
    # TRACK LIST VIEW
    # ------------------------------------------------------------

    @http.route([
        '''/event/<model("event.event"):event>/track''',
        '''/event/<model("event.event"):event>/track/tag/<model("event.track.tag"):tag>'''
    ], type='http', auth="public", website=True, sitemap=False, readonly=True)
    def event_tracks(self, event, tag=None, **searches):
        """ Main route

        :param event: event whose tracks are about to be displayed;
        :param tag: deprecated: search for a specific tag
        :param searches: frontend search dict, containing

          * 'search': search string;
          * 'tags': list of tag IDs for filtering;
        """

        if searches.get('tags', '[]').count(',') > 0 and request.httprequest.method == 'GET' and not searches.get('prevent_redirect'):
            # Previously, the tags were searched using GET, which caused issues with crawlers (too many hits)
            # We replaced those with POST to avoid that, but it's not sufficient as bots "remember" crawled pages for a while
            # This permanent redirect is placed to instruct the bots that this page is no longer valid
            # Note: We allow a single tag to be GET, to keep crawlers & indexes on those pages
            # What we really want to avoid is combinatorial explosions
            # (Tags are formed as a JSON array, so we count ',' to keep it simple)
            # TODO: remove in a few stable versions (v19?), including the "prevent_redirect" param in templates
            slug = request.env['ir.http']._slug
            return request.redirect(f'/event/{slug(event)}/track', code=301)

        return request.render(
            "website_event_track.tracks_session",
            self._event_tracks_get_values(event, tag=tag, **searches)
        )

    def _event_tracks_get_values(self, event, tag=None, **searches):
        # init and process search terms
        searches.setdefault('search', '')
        searches.setdefault('search_wishlist', '')
        searches.setdefault('tags', '')
        search_domain = self._get_event_tracks_agenda_domain(event)

        # search on content
        if searches.get('search'):
            search_domain = expression.AND([
                search_domain,
                [('name', 'ilike', searches['search'])]
            ])

        # search on tags
        search_tags = self._get_search_tags(searches['tags'])
        if not search_tags and tag:  # backward compatibility
            search_tags = tag
        if search_tags:
            # Example: You filter on age: 10-12 and activity: football.
            # Doing it this way allows to only get events who are tagged "age: 10-12" AND "activity: football".
            # Add another tag "age: 12-15" to the search and it would fetch the ones who are tagged:
            # ("age: 10-12" OR "age: 12-15") AND "activity: football
            grouped_tags = dict()
            for search_tag in search_tags:
                grouped_tags.setdefault(search_tag.category_id, list()).append(search_tag)
            search_domain_items = [
                [('tag_ids', 'in', [tag.id for tag in grouped_tags[group]])]
                for group in grouped_tags
            ]
            search_domain = expression.AND([
                search_domain,
                *search_domain_items
            ])

        # fetch data to display with TZ set for both event and tracks
        now_tz = utc.localize(fields.Datetime.now().replace(microsecond=0), is_dst=False).astimezone(timezone(event.date_tz))
        today_tz = now_tz.date()
        event = event.with_context(tz=event.date_tz or 'UTC')
        tracks_sudo = event.env['event.track'].sudo().search(search_domain, order='is_published desc, date asc')
        tag_categories = request.env['event.track.tag.category'].sudo().search([])

        # filter on wishlist (as post processing due to costly search on is_reminder_on)
        if searches.get('search_wishlist'):
            tracks_sudo = tracks_sudo.filtered(lambda track: track.is_reminder_on)

        # organize categories for display: announced, live, soon and day-based
        tracks_announced = tracks_sudo.filtered(lambda track: not track.date)
        tracks_wdate = tracks_sudo - tracks_announced
        date_begin_tz_all = list(set(
            dt.date()
            for dt in self._get_dt_in_event_tz(tracks_wdate.mapped('date'), event)
        ))
        date_begin_tz_all.sort()
        tracks_sudo_live = tracks_wdate.filtered(lambda track: track.is_track_live)
        tracks_sudo_soon = tracks_wdate.filtered(lambda track: not track.is_track_live and track.is_track_soon)
        tracks_by_day = []
        for display_date in date_begin_tz_all:
            matching_tracks = tracks_wdate.filtered(lambda track: self._get_dt_in_event_tz([track.date], event)[0].date() == display_date)
            tracks_by_day.append({'date': display_date, 'name': display_date, 'tracks': matching_tracks})
        if tracks_announced:
            tracks_announced = tracks_announced.sorted('wishlisted_by_default', reverse=True)
            tracks_by_day.append({'date': False, 'name': _('Coming soon'), 'tracks': tracks_announced})

        for tracks_group in tracks_by_day:
            # the tracks group is folded if all tracks are done (and if it's not "today")
            tracks_group['default_collapsed'] = (today_tz != tracks_group['date']) and all(
                track.is_track_done and not track.is_track_live
                for track in tracks_group['tracks']
            )

        # return rendering values
        return {
            # event information
            'event': event,
            'main_object': event,
            # tracks display information
            'tracks': tracks_sudo,
            'tracks_by_day': tracks_by_day,
            'tracks_live': tracks_sudo_live,
            'tracks_soon': tracks_sudo_soon,
            'today_tz': today_tz,
            # search information
            'searches': searches,
            'search_count': len(tracks_sudo),
            'search_key': searches['search'],
            'search_wishlist': searches['search_wishlist'],
            'search_tags': search_tags,
            'tag_categories': tag_categories,
            # environment
            'is_html_empty': is_html_empty,
            'hostname': request.httprequest.host.split(':')[0],
            'is_event_user': request.env.user.has_group('event.group_event_user'),
        }

    # ------------------------------------------------------------
    # AGENDA VIEW
    # ------------------------------------------------------------

    @http.route(['''/event/<model("event.event"):event>/agenda'''], type='http', auth="public", website=True, sitemap=False)
    def event_agenda(self, event, tag=None, **post):
        event = event.with_context(tz=event.date_tz or 'UTC')
        vals = {
            'event': event,
            'main_object': event,
            'tag': tag,
            'is_event_user': request.env.user.has_group('event.group_event_user'),
        }

        vals.update(self._prepare_calendar_values(event))

        return request.render("website_event_track.agenda_online", vals)

    def _prepare_calendar_values(self, event):
        """ This methods slit the day (max end time - min start time) into
        15 minutes time slots. For each time slot, we assign the tracks that
        start at this specific time slot, and we add the number of time slot
        that the track covers (track duration / 15 min). The calendar will be
        divided into rows of 15 min, and the talks will cover the corresponding
        number of rows (15 min slots). """
        event = event.with_context(tz=event.date_tz or 'UTC')
        local_tz = pytz.timezone(event.date_tz or 'UTC')
        lang_code = request.env.context.get('lang')

        base_track_domain = expression.AND([
            self._get_event_tracks_agenda_domain(event),
            [('date', '!=', False)]
        ])
        tracks_sudo = request.env['event.track'].sudo().search(base_track_domain)

        locations = list(set(track.location_id for track in tracks_sudo))
        locations.sort(key=operator.itemgetter('sequence', 'id'))

        # First split day by day (based on start time)
        time_slots_by_tracks = {track: self._split_track_by_days(track, local_tz) for track in tracks_sudo}

        # extract all the tracks time slots
        track_time_slots = set().union(*(time_slot.keys() for time_slot in [time_slots for time_slots in time_slots_by_tracks.values()]))

        # extract unique days
        days = list(set(time_slot.date() for time_slot in track_time_slots))
        days.sort()

        # Create the dict that contains the tracks at the correct time_slots / locations coordinates
        tracks_by_days = dict.fromkeys(days, 0)
        time_slots_by_day = dict((day, dict(start=set(), end=set())) for day in days)
        tracks_by_rounded_times = dict((time_slot, dict((location, {}) for location in locations)) for time_slot in track_time_slots)
        for track, time_slots in time_slots_by_tracks.items():
            start_date = fields.Datetime.from_string(track.date).replace(tzinfo=pytz.utc).astimezone(local_tz)
            end_date = start_date + timedelta(hours=(track.duration or 0.25))

            for time_slot, duration in time_slots.items():
                tracks_by_rounded_times[time_slot][track.location_id][track] = {
                    'rowspan': duration,  # rowspan
                    'start_date': self._get_locale_time(start_date, lang_code),
                    'end_date': self._get_locale_time(end_date, lang_code),
                    'occupied_cells': self._get_occupied_cells(track, duration, locations, local_tz)
                }

                # get all the time slots by day to determine the max duration of a day.
                day = time_slot.date()
                time_slots_by_day[day]['start'].add(time_slot)
                time_slots_by_day[day]['end'].add(time_slot+timedelta(minutes=15*duration))
                tracks_by_days[day] += 1

        # split days into 15 minutes time slots
        global_time_slots_by_day = dict((day, {}) for day in days)
        for day, time_slots in time_slots_by_day.items():
            start_time_slot = min(time_slots['start'])
            end_time_slot = max(time_slots['end'])

            time_slots_count = int(((end_time_slot - start_time_slot).total_seconds() / 3600) * 4)
            current_time_slot = start_time_slot
            for i in range(0, time_slots_count + 1):
                global_time_slots_by_day[day][current_time_slot] = tracks_by_rounded_times.get(current_time_slot, {})
                global_time_slots_by_day[day][current_time_slot]['formatted_time'] = self._get_locale_time(current_time_slot, lang_code)
                current_time_slot = current_time_slot + timedelta(minutes=15)

        # count the number of tracks by days
        tracks_by_days = dict.fromkeys(days, 0)
        locations_by_days = defaultdict(list)
        for track in tracks_sudo:
            track_day = fields.Datetime.from_string(track.date).replace(tzinfo=pytz.utc).astimezone(local_tz).date()
            tracks_by_days[track_day] += 1
            if track.location_id not in locations_by_days[track_day]:
                locations_by_days[track_day].append(track.location_id)

        for used_locations in locations_by_days.values():
            used_locations.sort(key=operator.itemgetter('sequence', 'id'))

        return {
            'days': days,
            'tracks_by_days': tracks_by_days,
            'locations_by_days': locations_by_days,
            'time_slots': global_time_slots_by_day,
            'locations': locations  # TODO: clean me in master, kept for retro-compatibility
        }

    def _get_locale_time(self, dt_time, lang_code):
        """ Get locale time from datetime object

            :param dt_time: datetime object
            :param lang_code: language code (eg. en_US)
        """
        locale = babel_locale_parse(lang_code)
        return babel.dates.format_time(dt_time, format='short', locale=locale)

    def time_slot_rounder(self, time, rounded_minutes):
        """ Rounds to nearest hour by adding a timedelta hour if minute >= rounded_minutes
            E.g. : If rounded_minutes = 15 -> 09:26:00 becomes 09:30:00
                                              09:17:00 becomes 09:15:00
        """
        return (time.replace(second=0, microsecond=0, minute=0, hour=time.hour)
                + timedelta(minutes=rounded_minutes * (time.minute // rounded_minutes)))

    def _split_track_by_days(self, track, local_tz):
        """
        Based on the track start_date and the duration,
        split the track duration into :
            start_time by day : number of time slot (15 minutes) that the track takes on that day.
        E.g. :  start date = 01-01-2000 10:00 PM and duration = 3 hours
                return {
                    01-01-2000 10:00:00 PM: 8 (2 * 4),
                    01-02-2000 00:00:00 AM: 4 (1 * 4)
                }
        Also return a set of all the time slots
        """
        start_date = fields.Datetime.from_string(track.date).replace(tzinfo=pytz.utc).astimezone(local_tz)
        start_datetime = self.time_slot_rounder(start_date, 15)
        end_datetime = self.time_slot_rounder(start_datetime + timedelta(hours=(track.duration or 0.25)), 15)
        time_slots_count = int(((end_datetime - start_datetime).total_seconds() / 3600) * 4)

        time_slots_by_day_start_time = {start_datetime: 0}
        for i in range(0, time_slots_count):
            # If the new time slot is still on the current day
            next_day = (start_datetime + timedelta(days=1)).date()
            if (start_datetime + timedelta(minutes=15*i)).date() <= next_day:
                time_slots_by_day_start_time[start_datetime] += 1
            else:
                start_datetime = next_day.datetime()
                time_slots_by_day_start_time[start_datetime] = 0

        return time_slots_by_day_start_time

    def _get_occupied_cells(self, track, rowspan, locations, local_tz):
        """
        In order to use only once the cells that the tracks will occupy, we need to reserve those cells
        (time_slot, location) coordinate. Those coordinated will be given to the template to avoid adding
        blank cells where already occupied by a track.
        """
        occupied_cells = []

        start_date = fields.Datetime.from_string(track.date).replace(tzinfo=pytz.utc).astimezone(local_tz)
        start_date = self.time_slot_rounder(start_date, 15)
        for i in range(0, rowspan):
            time_slot = start_date + timedelta(minutes=15*i)
            if track.location_id:
                occupied_cells.append((time_slot, track.location_id))
            # when no location, reserve all locations
            else:
                occupied_cells += [(time_slot, location) for location in locations if location]

        return occupied_cells

    # ------------------------------------------------------------
    # TRACK PAGE VIEW
    # ------------------------------------------------------------

    @http.route('''/event/<model("event.event", "[('website_track', '=', True)]"):event>/track/<model("event.track", "[('event_id', '=', event.id)]"):track>''',
                type='http', auth="public", website=True, sitemap=True, readonly=True)
    def event_track_page(self, event, track, **options):
        track = self._fetch_track(track.id, allow_sudo=False)

        return request.render(
            "website_event_track.event_track_main",
            self._event_track_page_get_values(event, track.sudo(), **options)
        )

    def _event_track_page_get_values(self, event, track, **options):
        track = track.sudo()

        option_widescreen = options.get('widescreen', False)
        option_widescreen = bool(option_widescreen) if option_widescreen != '0' else False
        # search for tracks list
        tracks_other = track._get_track_suggestions(
            restrict_domain=self._get_event_tracks_domain(track.event_id),
            limit=10
        )

        return {
            # event information
            'event': event,
            'main_object': track,
            'track': track,
            # sidebar
            'tracks_other': tracks_other,
            # options
            'option_widescreen': option_widescreen,
            # environment
            'is_html_empty': is_html_empty,
            'hostname': request.httprequest.host.split(':')[0],
            'is_event_user': request.env.user.has_group('event.group_event_user'),
            'user_event_manager': request.env.user.has_group('event.group_event_manager'),
        }

    @http.route("/event/track/toggle_reminder", type="json", auth="public", website=True)
    def track_reminder_toggle(self, track_id, set_reminder_on):
        """ Set a reminder a track for current visitor. Track visitor is created or updated
        if it already exists. Exception made if un-favoriting and no track_visitor
        record found (should not happen unless manually done).

        :param boolean set_reminder_on:
          If True, set as a favorite, otherwise un-favorite track;
          If the track is a Key Track (wishlisted_by_default):
            if set_reminder_on = False, blacklist the track_partner
            otherwise, un-blacklist the track_partner
        """
        track = self._fetch_track(track_id, allow_sudo=True)
        force_create = set_reminder_on or track.wishlisted_by_default
        event_track_partner = track._get_event_track_visitors(force_create=force_create)

        if not track.wishlisted_by_default:
            if not event_track_partner or event_track_partner.is_wishlisted == set_reminder_on:  # ignore if new state = old state
                return {'error': 'ignored'}
            event_track_partner.is_wishlisted = set_reminder_on
        else:
            if not event_track_partner or event_track_partner.is_blacklisted != set_reminder_on:  # ignore if new state = old state
                return {'error': 'ignored'}
            event_track_partner.is_blacklisted = not set_reminder_on

        result = {'reminderOn': set_reminder_on}

        return result

    # ------------------------------------------------------------
    # TRACK PROPOSAL
    # ------------------------------------------------------------

    @http.route(['''/event/<model("event.event"):event>/track_proposal'''], type='http', auth="public", website=True, sitemap=False)
    def event_track_proposal(self, event, **post):
        return request.render("website_event_track.event_track_proposal", {'event': event, 'main_object': event})

    @http.route(['''/event/<model("event.event"):event>/track_proposal/post'''], type='http', auth="public", methods=['POST'], website=True)
    def event_track_proposal_post(self, event, **post):
        if not event.can_access_from_current_website():
            return json.dumps({'error': 'forbidden'})

        # Only accept existing tag indices. Use search instead of browse + exists:
        # this prevents users to register colorless tags if not allowed to (ACL).
        input_tag_indices = [int(tag_id) for tag_id in post['tags'].split(',') if tag_id]
        valid_tag_indices = request.env['event.track.tag'].search([('id', 'in', input_tag_indices)]).ids

        contact = request.env['res.partner']
        visitor_partner = request.env['website.visitor']._get_visitor_from_request().partner_id
        # Contact name is required. Therefore, empty contacts are not considered here. At least one of contact_phone
        # and contact_email must be filled. Email is verified. If the post tries to create contact with no valid entry,
        # raise exception. If normalized email is the same as logged partner, use its partner_id on track instead.
        # This prevents contact duplication. Otherwise, create new contact with contact additional info of post.
        if post.get('add_contact_information'):
            valid_contact_email = tools.email_normalize(post.get('contact_email'))
            # Here, the phone is not formatted. To format it, one needs a country. Based on a country, from geoip for instance.
            # The problem is that one could propose a track in country A with phone number of country B. Validity is therefore
            # quite tricky. We accept any format of contact_phone. Could be improved with select country phone widget.
            if valid_contact_email or post.get('contact_phone'):
                if visitor_partner and valid_contact_email == visitor_partner.email_normalized:
                    contact = visitor_partner
                else:
                    contact = request.env['res.partner'].sudo().create({
                        'email': valid_contact_email,
                        'name': post.get('contact_name'),
                        'phone': post.get('contact_phone'),
                    })
            else:
                return json.dumps({'error': 'invalidFormInputs'})
        # If the speaker email is the same as logged user's, then also uses its partner on track, same as above.
        else:
            valid_speaker_email = tools.email_normalize(post['partner_email'])
            if visitor_partner and valid_speaker_email == visitor_partner.email_normalized:
                contact = visitor_partner

        track = request.env['event.track'].with_context({'mail_create_nosubscribe': True}).sudo().create({
            'name': post['track_name'],
            'partner_id': contact.id,
            'partner_name': post['partner_name'],
            'partner_email': post['partner_email'],
            'partner_phone': post['partner_phone'],
            'partner_function': post['partner_function'],
            'contact_phone': contact.phone,
            'contact_email': contact.email,
            'event_id': event.id,
            'tag_ids': [(6, 0, valid_tag_indices)],
            'description': plaintext2html(post['description']),
            'partner_biography': plaintext2html(post['partner_biography']),
            'user_id': False,
            'image': base64.b64encode(post['image'].read()) if post.get('image') else False,
        })

        if request.env.user != request.website.user_id:
            track.sudo().message_subscribe(partner_ids=request.env.user.partner_id.ids)

        return json.dumps({'success': True})

    # ACL : This route is necessary since rpc search_read method in js is not accessible to all users (e.g. public user).
    @http.route(['''/event/track_tag/search_read'''], type='json', auth="public", website=True)
    def website_event_track_fetch_tags(self, domain, fields):
        return request.env['event.track.tag'].search_read(domain, fields)

    # ------------------------------------------------------------
    # TOOLS
    # ------------------------------------------------------------

    def _fetch_track(self, track_id, allow_sudo=False):
        track = request.env['event.track'].browse(track_id).exists()
        if not track:
            raise NotFound()
        if not track.has_access('read'):
            if not allow_sudo:
                raise Forbidden()
            track = track.sudo()

        event = track.event_id
        # JSON RPC have no website in requests
        if hasattr(request, 'website_id') and not event.can_access_from_current_website():
            raise NotFound()
        if not event.has_access('read'):
            raise Forbidden()

        return track

    def _get_search_tags(self, tag_search):
        # TDE FIXME: make me generic (slides, event, ...)
        try:
            tag_ids = literal_eval(tag_search)
        except Exception:
            tags = request.env['event.track.tag'].sudo()
        else:
            # perform a search to filter on existing / valid tags implicitly
            tags = request.env['event.track.tag'].sudo().search([('id', 'in', tag_ids)])
        return tags

    def _get_dt_in_event_tz(self, datetimes, event):
        tz_name = event.date_tz
        return [
            utc.localize(dt, is_dst=False).astimezone(timezone(tz_name))
            for dt in datetimes
        ]

```

## File: controllers\webmanifest.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo import http
from odoo.http import request
from odoo.tools.misc import file_open
from odoo.tools.translate import _


class TrackManifest(http.Controller):

    @http.route('/event/manifest.webmanifest', type='http', auth='public', methods=['GET'], website=True, sitemap=False, readonly=True)
    def webmanifest(self):
        """ Returns a WebManifest describing the metadata associated with a web application.
        Using this metadata, user agents can provide developers with means to create user 
        experiences that are more comparable to that of a native application.
        """
        website = request.website
        manifest = {
            'name': website.events_app_name,
            'short_name': website.events_app_name,
            'description': _('%s Online Events Application') % website.company_id.name,
            'scope': request.env['ir.http']._url_for('/event'),
            'start_url': request.env['ir.http']._url_for('/event'),
            'display': 'standalone',
            'background_color': '#ffffff',
            'theme_color': '#875A7B',
        }
        icon_sizes = ['192x192', '512x512']
        manifest['icons'] = [{
            'src': website.image_url(website, 'app_icon', size=size),
            'sizes': size,
            'type': 'image/png',
        } for size in icon_sizes]
        body = json.dumps(manifest)
        response = request.make_response(body, [
            ('Content-Type', 'application/manifest+json'),
        ])
        return response

    @http.route('/event/service-worker.js', type='http', auth='public', methods=['GET'], website=True, sitemap=False, readonly=True)
    def service_worker(self):
        """ Returns a ServiceWorker javascript file scoped for website_event
        """
        with file_open('website_event_track/static/src/js/service_worker.js', 'r') as fp:
            body = fp.read()
        js_cdn_url = 'undefined'
        if request.website.cdn_activated:
            cdn_url = request.website.cdn_url.replace('"','%22').replace('\x5c','%5C')
            js_cdn_url = '"%s"' % cdn_url
        body = body.replace('__ODOO_CDN_URL__', js_cdn_url)
        response = request.make_response(body, [
            ('Content-Type', 'text/javascript'),
            ('Service-Worker-Allowed', request.env['ir.http']._url_for('/event')),
        ])
        return response

    @http.route('/event/offline', type='http', auth='public', methods=['GET'], website=True, sitemap=False, readonly=True)
    def offline(self):
        """ Returns the offline page used by the 'website_event' PWA
        """
        return request.render('website_event_track.pwa_offline')

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event
from . import event_track
from . import webmanifest

```

## File: data\event_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Update the event type if exists -->
        <function model="event.type" name="write">
            <value eval="[ref('event.event_type_data_conference', False)]"/>
            <value eval="{'website_menu': True, 'website_track': True, 'website_track_proposal': True}"/>
        </function>
    </data>
</odoo>

```

## File: data\event_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event.event_0" model="event.event">
        <field name="website_menu" eval="True"/>
        <field name="website_track" eval="True"/>
        <field name="website_track_proposal" eval="True"/>
    </record>
    <record id="event.event_7" model="event.event">
        <field name="website_menu" eval="True"/>
        <field name="website_track" eval="True"/>
        <field name="website_track_proposal" eval="True"/>
    </record>
</odoo>

```

## File: data\event_track_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="event_track_stage0" model="event.track.stage">
            <field name="name">Proposal</field>
            <field name="sequence">1</field>
            <field name="color">1</field>
        </record>
        <record id="event_track_stage1" model="event.track.stage">
            <field name="name">Confirmed</field>
            <field name="sequence">2</field>
            <field name="mail_template_id" ref="mail_template_data_track_confirmation"/>
            <field name="color">2</field>
        </record>
        <record id="event_track_stage2" model="event.track.stage">
            <field name="name">Announced</field>
            <field name="sequence">3</field>
            <field name="color">3</field>
            <field name="is_visible_in_agenda" eval="True"/>
        </record>
        <record id="event_track_stage3" model="event.track.stage">
            <field name="name">Published</field>
            <field name="sequence">4</field>
            <field name="color">4</field>
            <field name="is_visible_in_agenda" eval="True"/>
            <field name="is_fully_accessible" eval="True"/>
        </record>
        <record id="event_track_stage4" model="event.track.stage">
            <field name="name">Refused</field>
            <field name="sequence">5</field>
            <field name="color">5</field>
            <field name="fold" eval="True"/>
        </record>
        <record id="event_track_stage5" model="event.track.stage">
            <field name="name">Cancelled</field>
            <field name="sequence">6</field>
            <field name="fold" eval="True"/>
            <field name="is_cancel" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: data\event_track_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_track1" model="event.track">
        <field name="name">How to design a new piece of furniture</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=1, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
        <field name="location_id" ref="website_event_track.event_track_location5"/>
        <field name="duration" eval="1"/>
        <field name="partner_id" ref="base.res_partner_2"/>
        <field name="stage_id" ref="event_track_stage0"/>
        <field name="kanban_state">blocked</field>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track2" model="event.track">
        <field name="name">How to integrate hardware materials in your pieces of furniture</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=2, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
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
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=2, minutes=35)).strftime('%Y-%m-%d %H:%M:%S')"></field>
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
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=3, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
        <field name="location_id" ref="website_event_track.event_track_location5"/>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_2"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track5" model="event.track">
        <field name="name">The new way to promote your creations</field>
        <field name="is_published" eval="False"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=4, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
        <field name="location_id" ref="website_event_track.event_track_location6"/>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_4"/>
        <field name="stage_id" ref="event_track_stage2"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track6" model="event.track">
        <field name="name">Detailed roadmap of our new products</field>
        <field name="is_published" eval="False"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=7, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
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
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=10, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
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
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=1, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
        <field name="location_id" ref="website_event_track.event_track_location7"/>
        <field name="duration" eval="0.5"/>
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
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=3, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
        <field name="location_id" ref="website_event_track.event_track_location7"/>
        <field name="duration" eval="1"/>
        <field name="partner_id" ref="base.res_partner_12"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track10" model="event.track">
        <field name="name">Raising qualitive insights from your customers</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=5, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
        <field name="location_id" ref="website_event_track.event_track_location7"/>
        <field name="duration" eval="0.5"/>
        <field name="stage_id" ref="event_track_stage0"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track11" model="event.track">
        <field name="name">Discover our new design team</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=10, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
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
        <field name="date" eval="(DateTime.now() + timedelta(days=2, hours=1, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
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
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=2, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
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
        <field name="date" eval="(DateTime.now() + timedelta(days=2, hours=3, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
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
        <field name="date" eval="(DateTime.now() + timedelta(days=2, hours=8, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
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
        <field name="date" eval="(DateTime.now() + timedelta(days=2, hours=12, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
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
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=1, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
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
        <field name="date" eval="(DateTime.now() + timedelta(days=2, hours=1, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
        <field name="location_id" ref="website_event_track.event_track_location9"/>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_2"/>
        <field name="stage_id" ref="event_track_stage1"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track19" model="event.track">
        <field name="name">Advanced lead management: tips and tricks from the fields</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=3, hours=1, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
        <field name="location_id" ref="website_event_track.event_track_location9"/>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_4"/>
        <field name="stage_id" ref="event_track_stage1"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track20" model="event.track">
        <field name="name">New Certification Program</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=3, hours=5, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
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
        <field name="date" eval="(DateTime.now() + timedelta(days=4, hours=1, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
        <field name="location_id" ref="website_event_track.event_track_location9"/>
        <field name="duration" eval="0.5"/>
        <field name="stage_id" ref="event_track_stage1"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track22" model="event.track">
        <field name="name">Minimal but efficient design</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=4, hours=4, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
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
        <field name="date" eval="(DateTime.now() + timedelta(days=4, hours=5, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
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
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=1, minutes=25)).strftime('%Y-%m-%d %H:%M:%S')"></field>
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
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=4, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
        <field name="location_id" ref="website_event_track.event_track_location10"/>
        <field name="duration" eval="3.5"/>
        <field name="partner_id" ref="base.res_partner_18"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track27" model="event.track">
        <field name="name">My Company global presentation</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=3, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
        <field name="duration" eval="1"/>
        <field name="partner_id" ref="base.res_partner_1"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track28" model="event.track">
        <field name="name">Status &amp; Strategy</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=1, hours=4, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
        <field name="duration" eval="0.5"/>
        <field name="partner_id" ref="base.res_partner_2"/>
        <field name="stage_id" ref="event_track_stage2"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track29" model="event.track">
        <field name="name">The new marketing strategy</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=2, hours=1, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
        <field name="duration" eval="0.25"/>
        <field name="partner_id" ref="base.res_partner_2"/>
        <field name="stage_id" ref="event_track_stage2"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track30" model="event.track">
        <field name="name">Morning break</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=2, hours=4, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
        <field name="duration" eval="0.25"/>
        <field name="stage_id" ref="event_track_stage1"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="event_track31" model="event.track">
        <field name="name">Lunch</field>
        <field name="is_published" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=2, hours=16, minutes=5)).strftime('%Y-%m-%d %H:%M:%S')"></field>
        <field name="duration" eval="1"/>
        <field name="stage_id" ref="event_track_stage1"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>

    <!-- Tracks of: "OpenWood: Furniture Collection Online Reveal" -->
    <!-- DAY 1 -->
    <record id="event_7_track_1" model="event.track">
        <field name="name">What This Event Is All About</field>
        <field name="color">1</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="wishlisted_by_default" eval="True"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=1)).strftime('%Y-%m-%d 06:00:00')"></field>
        <field name="tag_ids" eval="[
            (4, ref('website_event_track.event_track_tag1')),
            (4, ref('website_event_track.event_track_tag2')),
            (4, ref('website_event_track.event_track_tag3')),
            (4, ref('website_event_track.event_track_tag13'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">2</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_15"/>
        <field name="website_cta" eval="True"/>
        <field name="website_cta_title">Try Now</field>
        <field name="website_cta_url">http://www.example.com</field>
        <field name="website_cta_delay">10</field>
    </record>
    <record id="event_7_track_2" model="event.track">
        <field name="name">First Day Wrapup</field>
        <field name="color">1</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="wishlisted_by_default" eval="True"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=1)).strftime('%Y-%m-%d 16:00:00')"></field>
        <field name="tag_ids" eval="[
            (4, ref('website_event_track.event_track_tag1')),
            (4, ref('website_event_track.event_track_tag2')),
            (4, ref('website_event_track.event_track_tag3')),
            (4, ref('website_event_track.event_track_tag13'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">1</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_28"/>
    </record>
    <!-- Location 1 -->
    <record id="event_7_track_3" model="event.track">
        <field name="name">Easy Way To Build a Wooden House</field>
        <field name="color">2</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_1"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=1)).strftime('%Y-%m-%d 08:30:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag1')), (4, ref('website_event_track.event_track_tag11'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">2.5</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_16"/>
    </record>
    <record id="event_7_track_4" model="event.track">
        <field name="name">Life at Home Around the World: William’s Story</field>
        <field name="color">0</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage4"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_1"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=1)).strftime('%Y-%m-%d 12:00:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag1')), (4, ref('website_event_track.event_track_tag11'))]"/>
        <field name="is_published" eval="False"/>
        <field name="duration">1</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_15"/>
        <field name="website_cta" eval="True"/>
        <field name="website_cta_title">Try Now</field>
        <field name="website_cta_url">http://www.example.com</field>
        <field name="website_cta_delay">10</field>
    </record>
    <record id="event_7_track_5" model="event.track">
        <field name="name">Top 10 Most Expensive Wood in the World</field>
        <field name="color">0</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_1"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=1)).strftime('%Y-%m-%d 14:00:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag3')), (4, ref('website_event_track.event_track_tag11'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">1</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_3"/>
    </record>
    <!-- Location 2 -->
    <record id="event_7_track_6" model="event.track">
        <field name="name">Securing your Lumber during transport</field>
        <field name="color">0</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_2"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=1)).strftime('%Y-%m-%d 08:30:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag3')), (4, ref('website_event_track.event_track_tag12'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">1.5</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_28"/>
    </record>
    <record id="event_7_track_7" model="event.track">
        <field name="name">Woodworking: How I got started!</field>
        <field name="color">3</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_2"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=1)).strftime('%Y-%m-%d 10:00:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag2')), (4, ref('website_event_track.event_track_tag12'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">1.5</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_15"/>
    </record>
    <record id="event_7_track_8" model="event.track">
        <field name="name">Dealing with OpenWood Furniture</field>
        <field name="color">0</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage2"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_2"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=1)).strftime('%Y-%m-%d 12:00:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag1')), (4, ref('website_event_track.event_track_tag12'))]"/>
        <field name="is_published" eval="False"/>
        <field name="duration">2</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_3"/>
    </record>
    <record id="event_7_track_9" model="event.track">
        <field name="name">Kitchens for the Future</field>
        <field name="color">0</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage2"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_2"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=1)).strftime('%Y-%m-%d 14:00:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag1')), (4, ref('website_event_track.event_track_tag12'))]"/>
        <field name="is_published" eval="False"/>
        <field name="duration">1</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_4"/>
    </record>
    <!-- Location 3 -->
    <record id="event_7_track_l3_1" model="event.track">
        <field name="name">Voice from Customer</field>
        <field name="color">5</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage2"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_3"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=1)).strftime('%Y-%m-%d 08:30:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag1')), (4, ref('website_event_track.event_track_tag12'))]"/>
        <field name="is_published" eval="False"/>
        <field name="duration">2</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_31"/>
    </record>
    <record id="event_7_track_l3_2" model="event.track">
        <field name="name">Who's OpenWood anyway?</field>
        <field name="color">0</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="website_event_track.event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_3"/>
        <field name="date" eval="(DateTime.now() - timedelta(days=1)).strftime('%Y-%m-%d 13:00:00')"></field>
        <field name="duration">1.5</field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag12'))]"/>
        <field name="is_published" eval="True"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_18"/>
    </record>

    <!-- DAY 2 -->
    <record id="event_7_track_10" model="event.track">
        <field name="name">Welcome to Day 2</field>
        <field name="color">1</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="wishlisted_by_default" eval="True"/>
        <field name="date" eval="DateTime.now().strftime('%Y-%m-%d 06:00:00')"></field>
        <field name="tag_ids" eval="[
            (4, ref('website_event_track.event_track_tag1')),
            (4, ref('website_event_track.event_track_tag2')),
            (4, ref('website_event_track.event_track_tag3')),
            (4, ref('website_event_track.event_track_tag13'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">1.5</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_28"/>
    </record>
    <record id="event_7_track_11" model="event.track">
        <field name="name">Day 2 Wrapup</field>
        <field name="color">1</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="date" eval="DateTime.now().strftime('%Y-%m-%d 16:00:00')"></field>
        <field name="tag_ids" eval="[
            (4, ref('website_event_track.event_track_tag1')),
            (4, ref('website_event_track.event_track_tag2')),
            (4, ref('website_event_track.event_track_tag3')),
            (4, ref('website_event_track.event_track_tag13'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">1</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_16"/>
    </record>
    <!-- Location 1 -->
    <record id="event_7_track_12" model="event.track">
        <field name="name">Climate positive</field>
        <field name="color">3</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_1"/>
        <field name="date" eval="DateTime.now().strftime('%Y-%m-%d 08:00:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag3')), (4, ref('website_event_track.event_track_tag11'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">3</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_15"/>
    </record>
    <record id="event_7_track_13" model="event.track">
        <field name="name">Log House Building</field>
        <field name="color">0</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="wishlisted_by_default" eval="True"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_1"/>
        <field name="date" eval="DateTime.now().strftime('%Y-%m-%d 12:00:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag3'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">1.5</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_16"/>
    </record>
    <record id="event_7_track_14" model="event.track">
        <field name="name">Building a DIY cabin from the ground up</field>
        <field name="color">0</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_1"/>
        <field name="date" eval="DateTime.now().strftime('%Y-%m-%d 13:30:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag3'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">1.5</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_3"/>
    </record>
    <!-- Location 2 -->
    <record id="event_7_track_15" model="event.track">
        <field name="name">Logs to lumber</field>
        <field name="color">0</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_2"/>
        <field name="date" eval="DateTime.now().strftime('%Y-%m-%d 08:00:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag3')), (4, ref('website_event_track.event_track_tag11'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">1</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_31"/>
    </record>
    <record id="event_7_track_16" model="event.track">
        <field name="name">Pretty. Ugly. Lovely.</field>
        <field name="color">0</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_2"/>
        <field name="date" eval="DateTime.now().strftime('%Y-%m-%d 09:00:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag2')), (4, ref('website_event_track.event_track_tag12'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">2</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_16"/>
    </record>
    <record id="event_7_track_17" model="event.track">
        <field name="name">10 DIY Furniture Ideas For Absolute Beginners</field>
        <field name="color">7</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="wishlisted_by_default" eval="True"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_2"/>
        <field name="date" eval="DateTime.now().strftime('%Y-%m-%d 12:00:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag1')), (4, ref('website_event_track.event_track_tag11'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">1</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_31"/>
    </record>
    <record id="event_7_track_18" model="event.track">
        <field name="name">6 Woodworking tips and tricks for beginners</field>
        <field name="color">0</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_2"/>
        <field name="date" eval="DateTime.now().strftime('%Y-%m-%d 13:00:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag1')), (4, ref('website_event_track.event_track_tag12'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">1</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_17"/>
    </record>
    <record id="event_7_track_19" model="event.track">
        <field name="name">DIY Timber Cladding Project</field>
        <field name="color">7</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_2"/>
        <field name="date" eval="DateTime.now().strftime('%Y-%m-%d 14:00:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag1')), (4, ref('website_event_track.event_track_tag11'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">1</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_3"/>
    </record>
    <!-- Location 3 -->
    <record id="event_7_track_l3_10" model="event.track">
        <field name="name">Live Testimonial</field>
        <field name="color">5</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="website_event_track.event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_3"/>
        <field name="date" eval="DateTime.now() - timedelta(minutes=30)"></field>
        <field name="duration">0.75</field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag1')), (4, ref('website_event_track.event_track_tag12'))]"/>
        <field name="is_published" eval="True"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_31"/>
    </record>
    <record id="event_7_track_l3_11" model="event.track">
        <field name="name">Happy with OpenWood</field>
        <field name="color">0</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="website_event_track.event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_3"/>
        <field name="date" eval="DateTime.now() + timedelta(minutes=15)"></field>
        <field name="duration">0.75</field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag1')), (4, ref('website_event_track.event_track_tag12'))]"/>
        <field name="is_published" eval="True"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_3"/>
    </record>

    <!-- DAY 3 -->
    <record id="event_7_track_20" model="event.track">
        <field name="name">Our Last Day Together!</field>
        <field name="color">1</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=1)).strftime('%Y-%m-%d 06:00:00')"></field>
        <field name="tag_ids" eval="[
            (4, ref('website_event_track.event_track_tag1')),
            (4, ref('website_event_track.event_track_tag2')),
            (4, ref('website_event_track.event_track_tag3')),
            (4, ref('website_event_track.event_track_tag13'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">1.5</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_28"/>
    </record>
    <record id="event_7_track_21" model="event.track">
        <field name="name">Event Wrapup</field>
        <field name="color">1</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=1)).strftime('%Y-%m-%d 15:00:00')"></field>
        <field name="tag_ids" eval="[
            (4, ref('website_event_track.event_track_tag1')),
            (4, ref('website_event_track.event_track_tag2')),
            (4, ref('website_event_track.event_track_tag3')),
            (4, ref('website_event_track.event_track_tag13'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">1.5</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_3"/>
    </record>
    <!-- Location 1 -->
    <record id="event_7_track_22" model="event.track">
        <field name="name">Tools for the Woodworking Beginner</field>
        <field name="color">0</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_1"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=1)).strftime('%Y-%m-%d 08:30:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag3')), (4, ref('website_event_track.event_track_tag11'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">2</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_4"/>
    </record>
    <record id="event_7_track_23" model="event.track">
        <field name="name">Restoring Old Woodworking Tools</field>
        <field name="color">7</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_1"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=1)).strftime('%Y-%m-%d 12:00:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag3')), (4, ref('website_event_track.event_track_tag12'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">1.5</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_4"/>
    </record>
    <!-- Location 2 -->
    <record id="event_7_track_24" model="event.track">
        <field name="name">Old is New</field>
        <field name="color">4</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_2"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=1)).strftime('%Y-%m-%d 09:00:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag2'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">1</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_17"/>
    </record>
    <!-- Location 3 -->
    <record id="event_7_track_25" model="event.track">
        <field name="name">Live Testimonials</field>
        <field name="color">0</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage3"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_3"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=1)).strftime('%Y-%m-%d 07:30:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag1')), (4, ref('website_event_track.event_track_tag2')), (4, ref('website_event_track.event_track_tag12'))]"/>
        <field name="is_published" eval="True"/>
        <field name="duration">3</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_18"/>
    </record>
    <record id="event_7_track_26" model="event.track">
        <field name="name">Less Furniture is More Furniture</field>
        <field name="color">0</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="stage_id" ref="event_track_stage2"/>
        <field name="location_id" ref="website_event_track.event_track_location_online_3"/>
        <field name="date" eval="(DateTime.now() + timedelta(days=1)).strftime('%Y-%m-%d 12:30:00')"></field>
        <field name="tag_ids" eval="[(4, ref('website_event_track.event_track_tag1'))]"/>
        <field name="is_published" eval="False"/>
        <field name="duration">0.75</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_17"/>
    </record>

</odoo>

```

## File: data\event_track_demo_description.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="event_7_track_3" model="event.track">
        <field name="description" type="html">
<p>Considering to build a wooden house? Watch this video to find out more information about a construction process and final result. Step by step simple explanation! Interested?</p>
        </field>
    </record>
    <record id="event_7_track_5" model="event.track">
        <field name="description" type="html">
<p>Top most expensive wood in the world is quite interesting topic and several people may be surprised
    that there are hundreds of wood types exist around the globe following different properties and use.</p>
<p>There are several variants of wood is available in the world but we are talking about most expensive
    ones in the world and keeping to the point we have arranged ten most expensive wood.</p>
        </field>
    </record>
    <record id="event_7_track_6" model="event.track">
        <field name="description" type="html">
<p>Use these simple steps to easily haul LONG lumber in a short box pickup truck.  A dose of carpenter's
    ingenuity along with a couple boards, a sturdy strap and a few screws are all I use to easily haul
    long boards from the lumberyard to the Next Level Carpentry shop or jobsite.</p>
<p>Using a unique wrapping method for a tie down strap (NOT Bungee cords!!!) allows lumber to be
    cinched securely WITHOUT the need to tie and untie tricky or complicated knots.</p>
        </field>
    </record>
    <record id="event_7_track_7" model="event.track">
        <field name="description" type="html">
<p>Probably one of the most asked questions I've gotten is how I got started woodworking! In this video I share with you how/why I started building furniture!</p>
        </field>
    </record>
    <record id="event_7_track_13" model="event.track">
        <field name="description" type="html">
<p>Considering to build a wooden house? Watch this video to find out more information about a construction process and final result. Step by step simple explanation! Interested?</p>
        </field>
    </record>
    <record id="event_7_track_15" model="event.track">
        <field name="description" type="html">
<p>In this video we will see how lumber is made in a sawmill factory.</p>
        </field>
    </record>
    <record id="event_7_track_17" model="event.track">
        <field name="description" type="html">
<p> As you may have heard before, making your own furniture is actually not as difficult or as complicated as you think.
    In fact, some projects are so easy anyone could successfully complete them. For example, making a cute stool out of
    a old tire is a real piece of cake and if you’re in need of a coffee table you can easily put one together using
    wood crates.</p>
<p>There are a lot of ideas worth exploring so start with the 10 DIY furniture ideas for absolute beginners.</p>
        </field>
    </record>
    <record id="event_7_track_18" model="event.track">
        <field name="description" type="html">
<p>In this video, I covered 6 tips and tricks to help out beginners:
    <ul>
        <li>Making a center marking gauge</li>
        <li>Bandy clamp hack</li>
        <li>Right angle clamp jig</li>
        <li>Miter saw tip</li>
        <li>Glue tip</li>
        <li>Dowel Hack</li>
    </ul>
</p>
        </field>
    </record>
    <record id="event_7_track_19" model="event.track">
        <field name="description" type="html">
<p>Link to Q&amp;A here! The time has come to hide those old block walls. Love simple and transformation type projects like this! :)-</p>
        </field>
    </record>
    <record id="event_7_track_23" model="event.track">
        <field name="description" type="html">
<p> Restoring old woodworking tools</p>
        </field>
    </record>
</odoo>

```

## File: data\event_track_location_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

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

    <record id="event_track_location_online_1" model="event.track.location">
        <field name="name">RD and Sales</field>
    </record>
    <record id="event_track_location_online_2" model="event.track.location">
        <field name="name">Furniture</field>
    </record>
    <record id="event_track_location_online_3" model="event.track.location">
        <field name="name">Live with Customers</field>
    </record>

</odoo>

```

## File: data\event_track_tag_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_track_tag_category_1" model="event.track.tag.category">
        <field name="name">Audience</field>
        <field name="sequence">10</field>
    </record>
    <record id="event_track_tag_category_2" model="event.track.tag.category">
        <field name="name">Format</field>
        <field name="sequence">20</field>
    </record>

    <record id="event_track_tag1" model="event.track.tag">
        <field name="name">Consumers</field>
        <field name="color">1</field>
        <field name="category_id" ref="event_track_tag_category_1"/>
        <field name="sequence">1</field>
    </record>
    <record id="event_track_tag2" model="event.track.tag">
        <field name="name">Sales</field>
        <field name="color">2</field>
        <field name="category_id" ref="event_track_tag_category_1"/>
        <field name="sequence">2</field>
    </record>
    <record id="event_track_tag3" model="event.track.tag">
        <field name="name">Research</field>
        <field name="color">3</field>
        <field name="category_id" ref="event_track_tag_category_1"/>
        <field name="sequence">3</field>
    </record>
    <record id="event_track_tag11" model="event.track.tag">
        <field name="name">Lightning Talks</field>
        <field name="color">4</field>
        <field name="category_id" ref="event_track_tag_category_2"/>
        <field name="sequence">11</field>
    </record>
    <record id="event_track_tag12" model="event.track.tag">
        <field name="name">Round Table</field>
        <field name="color">5</field>
        <field name="category_id" ref="event_track_tag_category_2"/>
        <field name="sequence">12</field>
    </record>
    <record id="event_track_tag13" model="event.track.tag">
        <field name="name">Keynote</field>
        <field name="color">6</field>
        <field name="category_id" ref="event_track_tag_category_2"/>
        <field name="sequence">13</field>
    </record>

</odoo>

```

## File: data\event_track_visitor_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="event_7_track_2_visitor_0" model="event.track.visitor">
        <field name="track_id" ref="website_event_track.event_7_track_2"/>
        <field name="visitor_id" ref="website.website_visitor_0"/>
        <field name="is_wishlisted" eval="True"/>
    </record>
    <record id="event_7_track_2_visitor_2" model="event.track.visitor">
        <field name="track_id" ref="website_event_track.event_7_track_2"/>
        <field name="visitor_id" ref="website.website_visitor_2"/>
        <field name="is_wishlisted" eval="True"/>
    </record>

</data></odoo>

```

## File: data\mail_message_subtype_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!-- Event-related subtypes for new track / Chatter -->
    <record id="mt_event_track" model="mail.message.subtype">
        <field name="name">New Track</field>
        <field name="res_model">event.event</field>
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
</data></odoo>

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- Chatter templates -->
<template id="event_track_template_new">
    <p>
        <span>New track proposal</span>
        <a href="#" t-att-data-oe-model="track._name" t-att-data-oe-id="track.id" t-out="track.name"/>
    </p>
    <ul>
        <!-- "contact" information (the ones from partner_id) take precedence over public information -->
        <t t-set="speaker_name" t-value="track.partner_id.name or track.partner_name or track.partner_email" />
        <li t-if="speaker_name">
            <b>Proposed By</b>: <span t-out="speaker_name"/>
        </li>
        <t t-set="speaker_email" t-value="track.contact_email or track.partner_email" />
        <li t-if="speaker_email">
            <b>Mail</b>: <a t-attf-href="mailto:#{speaker_email}" t-out="speaker_email"></a>
        </li>
        <li t-if="track.contact_phone or track.partner_phone">
            <b>Phone</b>: <span t-out="track.contact_phone or track.partner_phone"/>
        </li>
        <li t-if="not is_html_empty(track.partner_biography)">
            <b>Speaker Biography</b>: <div t-field="track.partner_biography"/>
        </li>
        <li t-if="not is_html_empty(track.description)">
            <b>Talk Introduction</b>: <div t-field="track.description"/>
        </li>
    </ul>
</template>

</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <record id="mail_template_data_track_confirmation" model="mail.template">
        <field name="name">Event: Track Confirmation</field>
        <field name="model_id" ref="website_event_track.model_event_track"/>
        <field name="subject">Confirmation of {{ object.name }}</field>
        <field name="email_from">{{ (object.event_id.organizer_id.email_formatted or object.event_id.company_id.email_formatted or user.email_formatted or '') }}</field>
        <field name="use_default_to" eval="True"/>
        <field name="description">Sent to speakers whose track proposal is accepted (set the template on the right stage)</field>
        <field name="body_html" type="html">
<div>
    Dear <t t-out="object.partner_id.name or object.partner_name or ''">Brandon Freeman</t><br/>
    We are pleased to inform you that your proposal <t t-out="object.name or ''">What This Event Is All About</t> has been accepted and confirmed for the event <t t-out="object.event_id.name or ''">OpenWood Collection Online Reveal</t>.
    <br/>
    You will find more details here:
    <div style="margin: 16px 0px 16px 0px;">
        <a t-attf-href="/event/{{ object.event_id.id }}/track/{{ object.id }}"
                style="padding: 8px 16px 8px 16px; font-size: 14px; color: #FFFFFF; text-decoration: none !important; background-color: #875A7B; border: 0px solid #875A7B; border-radius:3px">
            View Talk
        </a>
    </div>
    <br/><br/>
    Thank you,
    <t t-if="user.signature">
        <br />
        <t t-out="user.signature or ''">--<br/>Mitchell Admin</t>
    </t>
</div></field>
        <field name="lang">{{ object.partner_id.lang }}</field>
        <field name="auto_delete" eval="True"/>
    </record>
</data></odoo>

```

## File: models\event_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class Event(models.Model):
    _inherit = "event.event"

    track_ids = fields.One2many('event.track', 'event_id', 'Tracks')
    track_count = fields.Integer('Track Count', compute='_compute_track_count')
    website_track = fields.Boolean(
        'Tracks on Website', compute='_compute_website_track',
        readonly=False, store=True)
    website_track_proposal = fields.Boolean(
        'Proposals on Website', compute='_compute_website_track_proposal',
        readonly=False, store=True)
    track_menu_ids = fields.One2many('website.event.menu', 'event_id', string='Event Tracks Menus', domain=[('menu_type', '=', 'track')])
    track_proposal_menu_ids = fields.One2many('website.event.menu', 'event_id', string='Event Proposals Menus', domain=[('menu_type', '=', 'track_proposal')])
    allowed_track_tag_ids = fields.Many2many('event.track.tag', relation='event_allowed_track_tags_rel', string='Available Track Tags')
    tracks_tag_ids = fields.Many2many(
        'event.track.tag', relation='event_track_tags_rel', string='Track Tags',
        compute='_compute_tracks_tag_ids', store=True)

    def _compute_track_count(self):
        data = self.env['event.track']._read_group([('stage_id.is_cancel', '!=', True)], ['event_id'], ['__count'])
        result = {event.id: count for event, count in data}
        for event in self:
            event.track_count = result.get(event.id, 0)

    @api.depends('event_type_id', 'website_menu')
    def _compute_website_track(self):
        """ Propagate event_type configuration (only at change); otherwise propagate
        website_menu updated value. Also force True is track_proposal changes. """
        for event in self:
            if event.event_type_id and event.event_type_id != event._origin.event_type_id:
                event.website_track = event.event_type_id.website_track
            elif event.website_menu and (event.website_menu != event._origin.website_menu or not event.website_track):
                event.website_track = True
            elif not event.website_menu:
                event.website_track = False

    @api.depends('event_type_id', 'website_track')
    def _compute_website_track_proposal(self):
        """ Propagate event_type configuration (only at change); otherwise propagate
        website_track updated value (both together True or False at update). """
        for event in self:
            if event.event_type_id and event.event_type_id != event._origin.event_type_id:
                event.website_track_proposal = event.event_type_id.website_track_proposal
            elif event.website_track != event._origin.website_track or not event.website_track or not event.website_track_proposal:
                event.website_track_proposal = event.website_track

    @api.depends('track_ids.tag_ids', 'track_ids.tag_ids.color')
    def _compute_tracks_tag_ids(self):
        for event in self:
            event.tracks_tag_ids = event.track_ids.mapped('tag_ids').filtered(lambda tag: tag.color != 0).ids

    # ------------------------------------------------------------
    # WEBSITE MENU MANAGEMENT
    # ------------------------------------------------------------

    def toggle_website_track(self, val):
        self.website_track = val

    def toggle_website_track_proposal(self, val):
        self.website_track_proposal = val

    def _get_menu_update_fields(self):
        return super(Event, self)._get_menu_update_fields() + ['website_track', 'website_track_proposal']

    def _update_website_menus(self, menus_update_by_field=None):
        super(Event, self)._update_website_menus(menus_update_by_field=menus_update_by_field)
        for event in self:
            if event.menu_id and (not menus_update_by_field or event in menus_update_by_field.get('website_track')):
                event._update_website_menu_entry('website_track', 'track_menu_ids', 'track')
            if event.menu_id and (not menus_update_by_field or event in menus_update_by_field.get('website_track_proposal')):
                event._update_website_menu_entry('website_track_proposal', 'track_proposal_menu_ids', 'track_proposal')

    def _get_menu_type_field_matching(self):
        res = super(Event, self)._get_menu_type_field_matching()
        res['track_proposal'] = 'website_track_proposal'
        return res

    def _get_website_menu_entries(self):
        self.ensure_one()
        return super(Event, self)._get_website_menu_entries() + [
            (_('Talks'), '/event/%s/track' % self.env['ir.http']._slug(self), False, 10, 'track'),
            (_('Agenda'), '/event/%s/agenda' % self.env['ir.http']._slug(self), False, 70, 'track'),
            (_('Talk Proposals'), '/event/%s/track_proposal' % self.env['ir.http']._slug(self), False, 15, 'track_proposal')
        ]

```

## File: models\event_track.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta
from pytz import utc
from random import randint

from odoo import api, fields, models, tools
from odoo.osv import expression
from odoo.tools.mail import is_html_empty
from odoo.tools.translate import _, html_translate


class Track(models.Model):
    _name = "event.track"
    _description = 'Event Track'
    _order = 'priority, date'
    _inherit = ['mail.thread', 'mail.activity.mixin', 'website.seo.metadata', 'website.published.mixin']

    @api.model
    def _get_default_stage_id(self):
        return self.env['event.track.stage'].search([], limit=1).id

    # description
    name = fields.Char('Title', required=True, translate=True)
    event_id = fields.Many2one('event.event', 'Event', required=True)
    active = fields.Boolean(default=True)
    user_id = fields.Many2one('res.users', 'Responsible', tracking=True, default=lambda self: self.env.user)
    company_id = fields.Many2one('res.company', related='event_id.company_id')
    tag_ids = fields.Many2many('event.track.tag', string='Tags')
    description = fields.Html(translate=html_translate, sanitize_attributes=False, sanitize_form=False)
    color = fields.Integer('Agenda Color')
    priority = fields.Selection([
        ('0', 'Low'), ('1', 'Medium'),
        ('2', 'High'), ('3', 'Highest')],
        'Priority', required=True, default='1')
    # management
    stage_id = fields.Many2one(
        'event.track.stage', string='Stage', ondelete='restrict',
        index=True, copy=False, default=_get_default_stage_id,
        group_expand='_read_group_expand_full',  # Always display all stages
        required=True, tracking=True)
    legend_blocked = fields.Char(related='stage_id.legend_blocked',
        string='Kanban Blocked Explanation', readonly=True)
    legend_done = fields.Char(related='stage_id.legend_done',
        string='Kanban Valid Explanation', readonly=True)
    legend_normal = fields.Char(related='stage_id.legend_normal',
        string='Kanban Ongoing Explanation', readonly=True)
    kanban_state = fields.Selection([
        ('normal', 'Grey'),
        ('done', 'Green'),
        ('blocked', 'Red')], string='Kanban State',
        copy=False, default='normal', required=True,
        help="A track's kanban state indicates special situations affecting it:\n"
             " * Grey is the default situation\n"
             " * Red indicates something is preventing the progress of this track\n"
             " * Green indicates the track is ready to be pulled to the next stage")
    kanban_state_label = fields.Char(
        string='Kanban State Label', compute='_compute_kanban_state_label', store=True,
        tracking=True)
    partner_id = fields.Many2one('res.partner', 'Contact')
    # speaker information
    partner_name = fields.Char(
        string='Name', compute='_compute_partner_name',
        readonly=False, store=True, tracking=10)
    partner_email = fields.Char(
        string='Email', compute='_compute_partner_email',
        readonly=False, store=True, tracking=20)
    partner_phone = fields.Char(
        string='Phone', compute='_compute_partner_phone',
        readonly=False, store=True, tracking=30)
    partner_biography = fields.Html(
        string='Biography', compute='_compute_partner_biography',
        sanitize_attributes=False,
        readonly=False, store=True)
    partner_function = fields.Char(
        'Job Position', compute='_compute_partner_function',
        store=True, readonly=False)
    partner_company_name = fields.Char(
        'Company Name', compute='_compute_partner_company_name',
        readonly=False, store=True)
    partner_tag_line = fields.Char(
        'Tag Line', compute='_compute_partner_tag_line',
        help='Description of the partner (name, function and company name)')
    image = fields.Image(
        string="Speaker Photo", compute="_compute_partner_image",
        readonly=False, store=True,
        max_width=256, max_height=256)
    # contact information
    contact_email = fields.Char(
        string='Contact Email', compute='_compute_contact_email',
        readonly=False, store=True, tracking=20)
    contact_phone = fields.Char(
        string='Contact Phone', compute='_compute_contact_phone',
        readonly=False, store=True, tracking=30)
    location_id = fields.Many2one('event.track.location', 'Location')
    # time information
    date = fields.Datetime('Track Date')
    date_end = fields.Datetime('Track End Date', compute='_compute_end_date', store=True)
    duration = fields.Float('Duration', default=0.5)
    is_track_live = fields.Boolean(
        'Is Track Live', compute='_compute_track_time_data')
    is_track_soon = fields.Boolean(
        'Is Track Soon', compute='_compute_track_time_data')
    is_track_today = fields.Boolean(
        'Is Track Today', compute='_compute_track_time_data')
    is_track_upcoming = fields.Boolean(
        'Is Track Upcoming', compute='_compute_track_time_data')
    is_track_done = fields.Boolean(
        'Is Track Done', compute='_compute_track_time_data')
    track_start_remaining = fields.Integer(
        'Minutes before track starts', compute='_compute_track_time_data',
        help="Remaining time before track starts (seconds)")
    track_start_relative = fields.Integer(
        'Minutes compare to track start', compute='_compute_track_time_data',
        help="Relative time compared to track start (seconds)")
    # frontend description
    website_image = fields.Image(string="Website Image", max_width=1024, max_height=1024)
    website_image_url = fields.Char(
        string='Image URL', compute='_compute_website_image_url',
        compute_sudo=True, store=False)
    # wishlist / visitors management
    event_track_visitor_ids = fields.One2many(
        'event.track.visitor', 'track_id', string="Track Visitors",
        groups="event.group_event_user")
    is_reminder_on = fields.Boolean('Is Reminder On', compute='_compute_is_reminder_on')
    wishlist_visitor_ids = fields.Many2many(
        'website.visitor', string="Visitor Wishlist",
        compute="_compute_wishlist_visitor_ids", compute_sudo=True,
        search="_search_wishlist_visitor_ids",
        groups="event.group_event_user")
    wishlist_visitor_count = fields.Integer(
        string="# Wishlisted",
        compute="_compute_wishlist_visitor_ids", compute_sudo=True,
        groups="event.group_event_user")
    wishlisted_by_default = fields.Boolean(
        string='Always Wishlisted',
        help="""If set, the talk will be set as favorite for each attendee registered to the event.""")
    # Call to action
    website_cta = fields.Boolean('Magic Button',
                                 help="Display a Call to Action button to your Attendees while they watch your Track.")
    website_cta_title = fields.Char('Button Title')
    website_cta_url = fields.Char('Button Target URL')
    website_cta_delay = fields.Integer('Show Button')
    # time information for CTA
    is_website_cta_live = fields.Boolean(
        'Is CTA Live', compute='_compute_cta_time_data',
        help="CTA button is available")
    website_cta_start_remaining = fields.Integer(
        'Minutes before CTA starts', compute='_compute_cta_time_data',
        help="Remaining time before CTA starts (seconds)")

    @api.depends('name')
    def _compute_website_url(self):
        super(Track, self)._compute_website_url()
        for track in self:
            if track.id:
                track.website_url = '/event/%s/track/%s' % (self.env['ir.http']._slug(track.event_id), self.env['ir.http']._slug(track))

    # STAGES

    @api.depends('stage_id', 'kanban_state')
    def _compute_kanban_state_label(self):
        for track in self:
            if track.kanban_state == 'normal':
                track.kanban_state_label = track.stage_id.legend_normal
            elif track.kanban_state == 'blocked':
                track.kanban_state_label = track.stage_id.legend_blocked
            else:
                track.kanban_state_label = track.stage_id.legend_done

    # SPEAKER

    @api.depends('partner_id')
    def _compute_partner_name(self):
        for track in self:
            if track.partner_id and not track.partner_name:
                track.partner_name = track.partner_id.name

    @api.depends('partner_id')
    def _compute_partner_email(self):
        for track in self:
            if track.partner_id and not track.partner_email:
                track.partner_email = track.partner_id.email

    @api.depends('partner_id')
    def _compute_partner_phone(self):
        for track in self:
            if track.partner_id and not track.partner_phone:
                track.partner_phone = track.partner_id.phone

    @api.depends('partner_id')
    def _compute_partner_biography(self):
        for track in self:
            if not track.partner_biography:
                track.partner_biography = track.partner_id.website_description
            elif track.partner_id and is_html_empty(track.partner_biography) and \
                not is_html_empty(track.partner_id.website_description):
                track.partner_biography = track.partner_id.website_description

    @api.depends('partner_id')
    def _compute_partner_function(self):
        for track in self:
            if track.partner_id and not track.partner_function:
                track.partner_function = track.partner_id.function

    @api.depends('partner_id', 'partner_id.company_type')
    def _compute_partner_company_name(self):
        for track in self:
            if track.partner_id.company_type == 'company':
                track.partner_company_name = track.partner_id.name
            elif not track.partner_company_name:
                track.partner_company_name = track.partner_id.parent_id.name

    @api.depends('partner_name', 'partner_function', 'partner_company_name')
    def _compute_partner_tag_line(self):
        for track in self:
            if not track.partner_name:
                track.partner_tag_line = False
                continue

            tag_line = track.partner_name
            if track.partner_function:
                if track.partner_company_name:
                    tag_line = _('%(name)s, %(function)s at %(company)s',
                                 name=track.partner_name,
                                 function=track.partner_function,
                                 company=track.partner_company_name
                                )
                else:
                    tag_line = '%s, %s' % (track.partner_name, track.partner_function)
            elif track.partner_company_name:
                tag_line = _('%(name)s from %(company)s',
                             name=tag_line,
                             company=track.partner_company_name
                            )
            track.partner_tag_line = tag_line

    @api.depends('partner_id')
    def _compute_partner_image(self):
        for track in self:
            if not track.image:
                track.image = track.partner_id.image_256

    # CONTACT

    @api.depends('partner_id', 'partner_id.email')
    def _compute_contact_email(self):
        for track in self:
            if track.partner_id:
                track.contact_email = track.partner_id.email

    @api.depends('partner_id', 'partner_id.phone')
    def _compute_contact_phone(self):
        for track in self:
            if track.partner_id:
                track.contact_phone = track.partner_id.phone

    # TIME

    @api.depends('date', 'duration')
    def _compute_end_date(self):
        for track in self:
            if track.date:
                delta = timedelta(minutes=60 * track.duration)
                track.date_end = track.date + delta
            else:
                track.date_end = False


    # FRONTEND DESCRIPTION

    @api.depends('image', 'partner_id.image_256')
    def _compute_website_image_url(self):
        for track in self:
            if track.website_image:
                track.website_image_url = self.env['website'].image_url(track, 'website_image', size=1024)
            else:
                track.website_image_url = '/website_event_track/static/src/img/event_track_default_%d.jpeg' % (track.id % 2)

    # WISHLIST / VISITOR MANAGEMENT

    @api.depends('wishlisted_by_default', 'event_track_visitor_ids.visitor_id',
                 'event_track_visitor_ids.partner_id', 'event_track_visitor_ids.is_wishlisted',
                 'event_track_visitor_ids.is_blacklisted')
    @api.depends_context('uid')
    def _compute_is_reminder_on(self):
        current_visitor = self.env['website.visitor']._get_visitor_from_request()
        if self.env.user._is_public() and not current_visitor:
            for track in self:
                track.is_reminder_on = track.wishlisted_by_default
        else:
            if self.env.user._is_public():
                domain = [('visitor_id', '=', current_visitor.id)]
            elif current_visitor:
                domain = [
                    '|',
                    ('partner_id', '=', self.env.user.partner_id.id),
                    ('visitor_id', '=', current_visitor.id)
                ]
            else:
                domain = [('partner_id', '=', self.env.user.partner_id.id)]

            event_track_visitors = self.env['event.track.visitor'].sudo().search_read(
                expression.AND([
                    domain,
                    [('track_id', 'in', self.ids)]
                ]), fields=['track_id', 'is_wishlisted', 'is_blacklisted']
            )

            wishlist_map = {
                track_visitor['track_id'][0]: {
                    'is_wishlisted': track_visitor['is_wishlisted'],
                    'is_blacklisted': track_visitor['is_blacklisted']
                } for track_visitor in event_track_visitors
            }
            for track in self:
                if wishlist_map.get(track.id):
                    track.is_reminder_on = wishlist_map.get(track.id)['is_wishlisted'] or (track.wishlisted_by_default and not wishlist_map[track.id]['is_blacklisted'])
                else:
                    track.is_reminder_on = track.wishlisted_by_default

    @api.depends('event_track_visitor_ids.visitor_id', 'event_track_visitor_ids.is_wishlisted')
    def _compute_wishlist_visitor_ids(self):
        results = self.env['event.track.visitor']._read_group(
            [('track_id', 'in', self.ids), ('is_wishlisted', '=', True)],
            ['track_id'],
            ['visitor_id:array_agg'],
        )
        visitor_ids_map = {track.id: visitor_ids for track, visitor_ids in results}
        for track in self:
            track.wishlist_visitor_ids = visitor_ids_map.get(track.id, [])
            track.wishlist_visitor_count = len(visitor_ids_map.get(track.id, []))

    def _search_wishlist_visitor_ids(self, operator, operand):
        if operator == "not in":
            raise NotImplementedError("Unsupported 'Not In' operation on track wishlist visitors")

        track_visitors = self.env['event.track.visitor'].sudo().search([
            ('visitor_id', operator, operand),
            ('is_wishlisted', '=', True)
        ])
        return [('id', 'in', track_visitors.track_id.ids)]

    # TIME

    @api.depends('date', 'date_end')
    def _compute_track_time_data(self):
        """ Compute start and remaining time for track itself. Do everything in
        UTC as we compute only time deltas here. """
        now_utc = utc.localize(fields.Datetime.now().replace(microsecond=0))
        for track in self:
            if not track.date:
                track.is_track_live = track.is_track_soon = track.is_track_today = track.is_track_upcoming = track.is_track_done = False
                track.track_start_relative = track.track_start_remaining = 0
                continue
            date_begin_utc = utc.localize(track.date, is_dst=False)
            date_end_utc = utc.localize(track.date_end, is_dst=False)
            track.is_track_live = date_begin_utc <= now_utc < date_end_utc
            track.is_track_soon = (date_begin_utc - now_utc).total_seconds() < 30*60 if date_begin_utc > now_utc else False
            track.is_track_today = date_begin_utc.date() == now_utc.date()
            track.is_track_upcoming = date_begin_utc > now_utc
            track.is_track_done = date_end_utc <= now_utc
            if date_begin_utc >= now_utc:
                track.track_start_relative = int((date_begin_utc - now_utc).total_seconds())
                track.track_start_remaining = track.track_start_relative
            else:
                track.track_start_relative = int((now_utc - date_begin_utc).total_seconds())
                track.track_start_remaining = 0

    @api.depends('date', 'date_end', 'website_cta', 'website_cta_delay')
    def _compute_cta_time_data(self):
        """ Compute start and remaining time for track itself. Do everything in
        UTC as we compute only time deltas here. """
        now_utc = utc.localize(fields.Datetime.now().replace(microsecond=0))
        for track in self:
            if not track.website_cta:
                track.is_website_cta_live = track.website_cta_start_remaining = False
                continue

            date_begin_utc = utc.localize(track.date, is_dst=False) + timedelta(minutes=track.website_cta_delay or 0)
            date_end_utc = utc.localize(track.date_end, is_dst=False)
            track.is_website_cta_live = date_begin_utc <= now_utc <= date_end_utc
            if date_begin_utc >= now_utc:
                td = date_begin_utc - now_utc
                track.website_cta_start_remaining = int(td.total_seconds())
            else:
                track.website_cta_start_remaining = 0

    # ------------------------------------------------------------
    # CRUD
    # ------------------------------------------------------------

    @api.model_create_multi
    def create(self, vals_list):
        for values in vals_list:
            if values.get('website_cta_url'):
                values['website_cta_url'] = self.env['res.partner']._clean_website(values['website_cta_url'])

        tracks = super(Track, self).create(vals_list)

        post_values = {} if self.env.user.email else {'email_from': self.env.company.catchall_formatted}
        for track in tracks:
            track.event_id.message_post_with_source(
                'website_event_track.event_track_template_new',
                render_values={
                    'track': track,
                    'is_html_empty': is_html_empty,
                },
                subtype_xmlid='website_event_track.mt_event_track',
                **post_values,
            )
            track._synchronize_with_stage(track.stage_id)

        return tracks

    def write(self, vals):
        if vals.get('website_cta_url'):
            vals['website_cta_url'] = self.env['res.partner']._clean_website(vals['website_cta_url'])
        if 'stage_id' in vals and 'kanban_state' not in vals:
            vals['kanban_state'] = 'normal'
        if vals.get('stage_id'):
            stage = self.env['event.track.stage'].browse(vals['stage_id'])
            self._synchronize_with_stage(stage)
        res = super(Track, self).write(vals)
        return res

    def _synchronize_with_stage(self, stage):
        if stage.is_fully_accessible:
            self.is_published = True
        elif stage.is_cancel:
            self.is_published = False

    # ------------------------------------------------------------
    # MESSAGING
    # ------------------------------------------------------------

    def _message_get_default_recipients(self):
        return {
            track.id: {
                'partner_ids': [],
                'email_to': ','.join(tools.email_normalize_all(track.contact_email or track.partner_email)) or track.contact_email or track.partner_email,
                'email_cc': False
            } for track in self
        }

    def _message_get_suggested_recipients(self):
        recipients = super()._message_get_suggested_recipients()
        if self.partner_id:
            if self.partner_id not in recipients:
                self._message_add_suggested_recipient(recipients, partner=self.partner_id, reason=_('Contact'))
        else:
            #  Priority: contact information then speaker information
            if self.contact_email and self.contact_email != self.partner_id.email:
                self._message_add_suggested_recipient(recipients, email=self.contact_email, reason=_('Contact Email'))
            if not self.contact_email and self.partner_email and self.partner_email != self.partner_id.email:
                self._message_add_suggested_recipient(recipients, email=self.partner_email, reason=_('Speaker Email'))
        return recipients

    def _message_post_after_hook(self, message, msg_vals):
        #  OVERRIDE
        #  If no partner is set on track when sending a message, then we create one from suggested contact selected.
        #  If one or more have been created from chatter (Suggested Recipients) we search for the expected one and write the partner_id on track.
        if msg_vals.get('partner_ids') and not self.partner_id:
            #  Contact(s) created from chatter set on track : we verify if at least one is the expected contact
            #  linked to the track. (created from contact_email if any, then partner_email if any)
            main_email = self.contact_email or self.partner_email
            main_email_normalized = tools.email_normalize(main_email)
            new_partner = message.partner_ids.filtered(
                lambda partner: partner.email == main_email or (main_email_normalized and partner.email_normalized == main_email_normalized)
            )
            if new_partner:
                mail_email_fname = 'contact_email' if self.contact_email else 'partner_email'
                if new_partner[0].email_normalized:
                    email_domain = (mail_email_fname, 'in', [new_partner[0].email, new_partner[0].email_normalized])
                else:
                    email_domain = (mail_email_fname, '=', new_partner[0].email)
                self.search([
                    ('partner_id', '=', False), email_domain, ('stage_id.is_cancel', '=', False),
                ]).write({'partner_id': new_partner[0].id})
        return super(Track, self)._message_post_after_hook(message, msg_vals)

    def _track_template(self, changes):
        res = super(Track, self)._track_template(changes)
        track = self[0]
        if 'stage_id' in changes and track.stage_id.mail_template_id:
            res['stage_id'] = (track.stage_id.mail_template_id, {
                'auto_delete_keep_log': False,
                'composition_mode': 'comment',
                'email_layout_xmlid': 'mail.mail_notification_light',
                'subtype_id': self.env['ir.model.data']._xmlid_to_res_id('mail.mt_note'),
            })
        return res

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'kanban_state' in init_values and self.kanban_state == 'blocked':
            return self.env.ref('website_event_track.mt_track_blocked')
        elif 'kanban_state' in init_values and self.kanban_state == 'done':
            return self.env.ref('website_event_track.mt_track_ready')
        return super(Track, self)._track_subtype(init_values)

    # ------------------------------------------------------------
    # ACTION
    # ------------------------------------------------------------

    def open_track_speakers_list(self):
        return {
            'name': _('Speakers'),
            'domain': [('id', 'in', self.mapped('partner_id').ids)],
            'view_mode': 'kanban,form',
            'res_model': 'res.partner',
            'view_id': False,
            'type': 'ir.actions.act_window',
        }

    def get_backend_menu_id(self):
        return self.env.ref('event.event_main_menu').id

    # ------------------------------------------------------------
    # TOOLS
    # ------------------------------------------------------------

    def _get_event_track_visitors(self, force_create=False):
        self.ensure_one()

        force_visitor_create = self.env.user._is_public()
        visitor_sudo = self.env['website.visitor']._get_visitor_from_request(force_create=force_visitor_create)
        if visitor_sudo:
            visitor_sudo._update_visitor_last_visit()

        if self.env.user._is_public():
            domain = [('visitor_id', '=', visitor_sudo.id)]
        elif visitor_sudo:
            domain = [
                '|',
                ('partner_id', '=', self.env.user.partner_id.id),
                ('visitor_id', '=', visitor_sudo.id)
            ]
        else:
            domain = [('partner_id', '=', self.env.user.partner_id.id)]

        track_visitors = self.env['event.track.visitor'].sudo().search(
            expression.AND([domain, [('track_id', 'in', self.ids)]])
        )
        missing = self - track_visitors.track_id
        if missing and force_create:
            track_visitors += self.env['event.track.visitor'].sudo().create([{
                'visitor_id': visitor_sudo.id,
                'partner_id': self.env.user.partner_id.id if not self.env.user._is_public() else False,
                'track_id': track.id,
            } for track in missing])

        return track_visitors

    def _get_track_suggestions(self, restrict_domain=None, limit=None):
        """ Returns the next tracks suggested after going to the current one
        given by self. Tracks always belong to the same event.

        Heuristic is

          * live first;
          * then ordered by start date, finished being sent to the end;
          * wishlisted (manually or by default);
          * tag matching with current track;
          * location matching with current track;
          * finally a random to have an "equivalent wave" randomly given;

        :param restrict_domain: an additional domain to restrict candidates;
        :param limit: number of tracks to return;
        """
        self.ensure_one()

        base_domain = [
            '&',
            ('event_id', '=', self.event_id.id),
            ('id', '!=', self.id),
        ]
        if restrict_domain:
            base_domain = expression.AND([
                base_domain,
                restrict_domain
            ])

        track_candidates = self.search(base_domain, limit=None, order='date asc')
        if not track_candidates:
            return track_candidates

        track_candidates = track_candidates.sorted(
            lambda track:
                (track.is_published,
                 track.track_start_remaining == 0  # First get the tracks that started less than 10 minutes ago ...
                 and track.track_start_relative < (10 * 60)
                 and not track.is_track_done,  # ... AND not finished
                 track.track_start_remaining > 0,  # Then the one that will begin later (the sooner come first)
                 -1 * track.track_start_remaining,
                 track.is_reminder_on,
                 not track.wishlisted_by_default,
                 len(track.tag_ids & self.tag_ids),
                 track.location_id == self.location_id,
                 randint(0, 20),
                ), reverse=True
        )

        return track_candidates[:limit]

```

## File: models\event_track_location.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class TrackLocation(models.Model):
    _name = "event.track.location"
    _description = 'Event Track Location'
    _order = 'sequence, id'

    name = fields.Char('Location', required=True)
    sequence = fields.Integer(default=10, help='Define the order in which the location will appear on "Agenda" page')

```

## File: models\event_track_stage.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


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
    # legends
    color = fields.Integer(string='Color')
    description = fields.Text(string='Description', translate=True)
    legend_blocked = fields.Char('Red Kanban Label', default=lambda s: s.env._('Blocked'), translate=True)
    legend_done = fields.Char('Green Kanban Label', default=lambda s: s.env._('Ready for Next Stage'), translate=True)
    legend_normal = fields.Char('Grey Kanban Label', default=lambda s: s.env._('In Progress'), translate=True)
    # pipe
    fold = fields.Boolean(
        string='Folded in Kanban',
        help='This stage is folded in the kanban view when there are no records in that stage to display.')
    is_visible_in_agenda = fields.Boolean(
        string='Visible in agenda', compute='_compute_is_visible_in_agenda', store=True,
        help='If checked, the related tracks will be visible in the frontend.')
    is_fully_accessible = fields.Boolean(
        string='Fully accessible', compute='_compute_is_fully_accessible', store=True,
        help='If checked, automatically publish tracks so that access links to customers are provided.')
    is_cancel = fields.Boolean(string='Cancelled Stage')

    @api.depends('is_cancel', 'is_fully_accessible')
    def _compute_is_visible_in_agenda(self):
        for record in self:
            if record.is_cancel:
                record.is_visible_in_agenda = False
            elif record.is_fully_accessible:
                record.is_visible_in_agenda = True

    @api.depends('is_cancel', 'is_visible_in_agenda')
    def _compute_is_fully_accessible(self):
        for record in self:
            if record.is_cancel or not record.is_visible_in_agenda:
                record.is_fully_accessible = False

```

## File: models\event_track_tag.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint

from odoo import fields, models


class TrackTag(models.Model):
    _name = "event.track.tag"
    _description = 'Event Track Tag'
    _order = "category_id, sequence, name"

    def _default_color(self):
        return randint(1, 11)

    name = fields.Char('Tag Name', required=True)
    track_ids = fields.Many2many('event.track', string='Tracks')
    color = fields.Integer(
        string='Color Index', default=lambda self: self._default_color(),
        help="Note that colorless tags won't be available on the website.")
    sequence = fields.Integer('Sequence', default=10)
    category_id = fields.Many2one('event.track.tag.category', string="Category", ondelete="set null")

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Tag name already exists!"),
    ]

```

## File: models\event_track_tag_category.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class TrackTagCategory(models.Model):
    _name = "event.track.tag.category"
    _description = 'Event Track Tag Category'
    _order = "sequence"

    name = fields.Char("Name", required=True, translate=True)
    sequence = fields.Integer('Sequence', default=10)
    tag_ids = fields.One2many('event.track.tag', 'category_id', string="Tags")

```

## File: models\event_track_visitor.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class TrackVisitor(models.Model):
    """ Table linking track and visitors. """
    _name = 'event.track.visitor'
    _description = 'Track / Visitor Link'
    _table = 'event_track_visitor'
    _rec_name = 'track_id'
    _order = 'track_id'

    partner_id = fields.Many2one(
        'res.partner', string='Partner', compute='_compute_partner_id',
        index=True, ondelete='set null', readonly=False, store=True)
    visitor_id = fields.Many2one(
        'website.visitor', string='Visitor', index=True, ondelete='cascade')
    track_id = fields.Many2one(
        'event.track', string='Track',
        index=True, required=True, ondelete='cascade')
    is_wishlisted = fields.Boolean(string="Is Wishlisted")
    is_blacklisted = fields.Boolean(string="Is reminder off", help="As key track cannot be un-favorited, this field store the partner choice to remove the reminder for key tracks.")

    @api.depends('visitor_id')
    def _compute_partner_id(self):
        for track_visitor in self:
            if track_visitor.visitor_id.partner_id and not track_visitor.partner_id:
                track_visitor.partner_id = track_visitor.visitor_id.partner_id
            elif not track_visitor.partner_id:
                track_visitor.partner_id = False

```

## File: models\event_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class EventType(models.Model):
    _inherit = 'event.type'

    website_track = fields.Boolean(
        string='Tracks on Website', compute='_compute_website_track_menu_data',
        readonly=False, store=True)
    website_track_proposal = fields.Boolean(
        string='Tracks Proposals on Website', compute='_compute_website_track_menu_data',
        readonly=False, store=True)

    @api.depends('website_menu')
    def _compute_website_track_menu_data(self):
        """ Simply activate or de-activate all menus at once. """
        for event_type in self:
            event_type.website_track = event_type.website_menu
            event_type.website_track_proposal = event_type.website_menu

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    events_app_name = fields.Char('Events App Name', related='website_id.events_app_name', readonly=False)

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


import base64

from odoo import api, fields, models
from odoo.exceptions import ValidationError
from odoo.tools.image import ImageProcess
from odoo.tools.translate import _


class Website(models.Model):
    _inherit = "website"

    app_icon = fields.Image(
        string='Website App Icon',
        compute='_compute_app_icon',
        store=True,
        readonly=True,
        help='This field holds the image used as mobile app icon on the website (PNG format).')
    events_app_name = fields.Char(
        string='Events App Name',
        compute='_compute_events_app_name',
        store=True,
        readonly=False,
        help="This fields holds the Event's Progressive Web App name.")

    @api.depends('name')
    def _compute_events_app_name(self):
        for website in self:
            if not website.events_app_name:
                website.events_app_name = _('%s Events') % website.name

    @api.constrains('events_app_name')
    def _check_events_app_name(self):
        for website in self:
            if not website.events_app_name:
                raise ValidationError(_('"Events App Name" field is required.'))

    @api.depends('favicon')
    def _compute_app_icon(self):
        """ Computes a squared image based on the favicon to be used as mobile webapp icon.
            App Icon should be in PNG format and size of at least 512x512.

            If the favicon is an SVG image, it will be skipped and the app_icon will be set to False.

        """
        for website in self:
            image = ImageProcess(base64.b64decode(website.favicon)) if website.favicon else None
            if not (image and image.image):
                website.app_icon = False
                continue
            w, h = image.image.size
            square_size = w if w > h else h
            image.crop_resize(square_size, square_size)
            image.image = image.image.resize((512, 512))
            image.operationsCount += 1
            website.app_icon = base64.b64encode(image.image_quality(output_format='PNG'))

```

## File: models\website_event_menu.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class EventMenu(models.Model):
    _inherit = "website.event.menu"

    menu_type = fields.Selection(
        selection_add=[('track', 'Event Tracks Menus'), ('track_proposal', 'Event Proposals Menus')],
        ondelete={'track': 'cascade', 'track_proposal': 'cascade'})

```

## File: models\website_menu.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class WebsiteMenu(models.Model):
    _inherit = "website.menu"

    def unlink(self):
        """ Override to synchronize event configuration fields with menu deletion.
        This should be cleaned in upcoming versions. """    
        event_updates = {}
        website_event_menus = self.env['website.event.menu'].search([('menu_id', 'in', self.ids)])
        for event_menu in website_event_menus:
            to_update = event_updates.setdefault(event_menu.event_id, list())
            # specifically check for /track in menu URL; to avoid unchecking track field when removing
            # agenda page that has also menu_type='track'
            if event_menu.menu_type == 'track' and '/track' in event_menu.menu_id.url:
                to_update.append('website_track')

        # call super that resumes the unlink of menus entries (including website event menus)
        res = super(WebsiteMenu, self).unlink()

        # update events
        for event, to_update in event_updates.items():
            if to_update:
                event.write(dict((fname, False) for fname in to_update))

        return res

```

## File: models\website_visitor.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.osv import expression


class WebsiteVisitor(models.Model):
    _name = 'website.visitor'
    _inherit = ['website.visitor']

    event_track_visitor_ids = fields.One2many(
        'event.track.visitor', 'visitor_id', string="Track Visitors",
        groups='event.group_event_user')
    event_track_wishlisted_ids = fields.Many2many(
        'event.track', string="Wishlisted Tracks",
        compute="_compute_event_track_wishlisted_ids", compute_sudo=True,
        search="_search_event_track_wishlisted_ids",
        groups="event.group_event_user")
    event_track_wishlisted_count = fields.Integer(
        string="# Wishlisted",
        compute="_compute_event_track_wishlisted_ids", compute_sudo=True,
        groups='event.group_event_user')

    @api.depends('event_track_visitor_ids.track_id', 'event_track_visitor_ids.is_wishlisted')
    def _compute_event_track_wishlisted_ids(self):
        results = self.env['event.track.visitor']._read_group(
            [('visitor_id', 'in', self.ids), ('is_wishlisted', '=', True)],
            ['visitor_id'],
            ['track_id:array_agg'],
        )
        track_ids_map = {visitor.id: track_ids for visitor, track_ids in results}
        for visitor in self:
            visitor.event_track_wishlisted_ids = track_ids_map.get(visitor.id, [])
            visitor.event_track_wishlisted_count = len(visitor.event_track_wishlisted_ids)

    def _search_event_track_wishlisted_ids(self, operator, operand):
        """ Search visitors with terms on wishlisted tracks. E.g. [('event_track_wishlisted_ids',
        'in', [1, 2])] should return visitors having wishlisted tracks 1, 2. """
        if operator == "not in":
            raise NotImplementedError("Unsupported 'Not In' operation on track wishlist visitors")

        track_visitors = self.env['event.track.visitor'].sudo().search([
            ('track_id', operator, operand),
            ('is_wishlisted', '=', True)
        ])

        return [('id', 'in', track_visitors.visitor_id.ids)]

    def _inactive_visitors_domain(self):
        """ Visitors registered to push subscriptions are considered always active and should not be
        deleted. """
        domain = super()._inactive_visitors_domain()
        return expression.AND([domain, [('event_track_visitor_ids', '=', False)]])

    def _merge_visitor(self, target):
        """ Override linking process to link wishlist to the final visitor. """
        self.event_track_visitor_ids.visitor_id = target.id
        track_visitor_wo_partner = self.event_track_visitor_ids.filtered(lambda track_visitor: not track_visitor.partner_id)
        if track_visitor_wo_partner:
            track_visitor_wo_partner.partner_id = target.partner_id
        return super()._merge_visitor(target)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_event
from . import event_track
from . import event_track_location
from . import event_track_stage
from . import event_track_tag
from . import event_track_tag_category
from . import event_track_visitor
from . import event_type
from . import res_config_settings
from . import website
from . import website_event_menu
from . import website_menu
from . import website_visitor

```

## File: security\event_track_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="event_track_public" model="ir.rule">
        <field name="name">Event Tracks: public/portal: published</field>
        <field name="model_id" ref="website_event_track.model_event_track"/>
        <field name="domain_force">[('website_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

    <record id="ir_rule_event_track_tag_public" model="ir.rule">
        <field name="name">Event Track Tag: public/portal: color = published</field>
        <field name="model_id" ref="website_event_track.model_event_track_tag"/>
        <field name="domain_force">['&amp;', ('color', '!=', False), ('color', '!=', 0)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_event_track_public,event.track,model_event_track,base.group_public,1,0,0,0
access_event_track_portal,event.track,model_event_track,base.group_portal,1,0,0,0
access_event_track_employee,event.track,model_event_track,base.group_user,1,0,0,0
access_event_track_user,event.track.user,model_event_track,event.group_event_user,1,1,1,0
access_event_track_manager,event.track.manager,model_event_track,event.group_event_manager,1,1,1,1
access_event_track_tag_public,event.track.tag,model_event_track_tag,base.group_public,1,0,0,0
access_event_track_tag_portal,event.track.tag,model_event_track_tag,base.group_portal,1,0,0,0
access_event_track_tag_employee,event.track.tag,model_event_track_tag,base.group_user,1,0,0,0
access_event_track_tag_user,event.track.tag.user,model_event_track_tag,event.group_event_user,1,1,1,0
access_event_track_tag_manager,event.track.tag.manager,model_event_track_tag,event.group_event_manager,1,1,1,1
access_event_track_location_user,event.track.location.user,model_event_track_location,event.group_event_user,1,1,1,0
access_event_track_location_manager,event.track.location.manager,model_event_track_location,event.group_event_manager,1,1,1,1
access_event_track_stage_public,event.track.stage,model_event_track_stage,base.group_public,1,0,0,0
access_event_track_stage_portal,event.track.stage,model_event_track_stage,base.group_portal,1,0,0,0
access_event_track_stage_employee,event.track.stage,model_event_track_stage,base.group_user,1,0,0,0
access_event_track_stage_manager,event.track.stage.manager,model_event_track_stage,event.group_event_manager,1,1,1,1
access_event_track_visitor,event.track.visitor,model_event_track_visitor,,0,0,0,0
access_event_track_visitor_manager,event.track.visitor.manager,model_event_track_visitor,event.group_event_manager,1,1,1,1
access_event_track_tag_category_user,event.track.tag.category.user,model_event_track_tag_category,event.group_event_user,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M13.238 11.483a1.927 1.927 0 0 1 .654-2.55c1.403-.912 2.927-1.233 4.181-.613.2.1.376.24.54.39L49.26 37.346c.472.426.741 1.025.741 1.653H28L13.238 11.483Z" fill="#FBB945"/><path d="M50 39c0 1.657-4.925 3-11 3s-11-1.343-11-3 4.925-3 11-3 11 1.343 11 3Z" fill="#F78613"/><path d="M36.762 11.483a1.927 1.927 0 0 0-.654-2.55c-1.403-.912-2.927-1.233-4.181-.613-.2.1-.376.24-.54.39L.74 37.346A2.226 2.226 0 0 0 0 39h22l14.762-27.517Z" fill="#985184"/><path d="M31.693 20.93 25 14.677l-6.693 6.255L25 33.407l6.693-12.476Z" fill="#953B24"/><path d="M0 39c0 1.657 4.925 3 11 3s11-1.343 11-3-4.925-3-11-3-11 1.343-11 3Z" fill="#712258"/></svg>

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

## File: static\lib\idb-keyval\idb-keyval.js

```javascript
//     idb-keyval.js 3.2.0
//     https://github.com/jakearchibald/idb-keyval
//     Copyright 2016, Jake Archibald
//     Licensed under the Apache License, Version 2.0

var idbKeyval = (function (exports) {
    'use strict';
    
    class Store {
        constructor(dbName = 'keyval-store', storeName = 'keyval') {
            this.storeName = storeName;
            this._dbp = new Promise((resolve, reject) => {
                const openreq = indexedDB.open(dbName, 1);
                openreq.onerror = () => reject(openreq.error);
                openreq.onsuccess = () => resolve(openreq.result);
                // First time setup: create an empty object store
                openreq.onupgradeneeded = () => {
                    openreq.result.createObjectStore(storeName);
                };
            });
        }
        _withIDBStore(type, callback) {
            return this._dbp.then(db => new Promise((resolve, reject) => {
                const transaction = db.transaction(this.storeName, type);
                transaction.oncomplete = () => resolve();
                transaction.onabort = transaction.onerror = () => reject(transaction.error);
                callback(transaction.objectStore(this.storeName));
            }));
        }
    }
    let store;
    function getDefaultStore() {
        if (!store)
            store = new Store();
        return store;
    }
    function get(key, store = getDefaultStore()) {
        let req;
        return store._withIDBStore('readonly', store => {
            req = store.get(key);
        }).then(() => req.result);
    }
    function set(key, value, store = getDefaultStore()) {
        return store._withIDBStore('readwrite', store => {
            store.put(value, key);
        });
    }
    function del(key, store = getDefaultStore()) {
        return store._withIDBStore('readwrite', store => {
            store.delete(key);
        });
    }
    function clear(store = getDefaultStore()) {
        return store._withIDBStore('readwrite', store => {
            store.clear();
        });
    }
    function keys(store = getDefaultStore()) {
        const keys = [];
        return store._withIDBStore('readonly', store => {
            // This would be store.getAllKeys(), but it isn't supported by Edge or Safari.
            // And openKeyCursor isn't supported by Safari.
            (store.openKeyCursor || store.openCursor).call(store).onsuccess = function () {
                if (!this.result)
                    return;
                keys.push(this.result.key);
                this.result.continue();
            };
        }).then(() => keys);
    }
    
    exports.Store = Store;
    exports.get = get;
    exports.set = set;
    exports.del = del;
    exports.clear = clear;
    exports.keys = keys;
    
    return exports;
    
    }({}));

```

## File: static\src\js\event_track_reminder.js

```javascript
/** @odoo-module **/

import { debounce } from "@web/core/utils/timing";
import publicWidget from "@web/legacy/js/public/public_widget";
import { _t } from "@web/core/l10n/translation";
import { rpc } from "@web/core/network/rpc";
import { Component } from "@odoo/owl";

publicWidget.registry.websiteEventTrackReminder = publicWidget.Widget.extend({
    selector: '.o_wetrack_js_reminder',
    events: {
        'click': '_onReminderToggleClick',
    },

    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this._onReminderToggleClick = debounce(this._onReminderToggleClick, 500, true);
        this.notification = this.bindService("notification");
    },

    //--------------------------------------------------------------------------
    // Handlers
    //-------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onReminderToggleClick: function (ev) {
        ev.stopPropagation();
        ev.preventDefault();
        var self = this;
        var $trackLink = $(ev.currentTarget).find('i');

        if (this.reminderOn === undefined) {
            this.reminderOn = $trackLink.data('reminderOn');
        }

        var reminderOnValue = !this.reminderOn;

        rpc('/event/track/toggle_reminder', {
            track_id: $trackLink.data('trackId'),
            set_reminder_on: reminderOnValue,
        }).then(function (result) {
            if (result.error && result.error === 'ignored') {
                self.notification.add(_t('Talk already in your Favorites'), {
                    type: 'info',
                    title: _t('Error'),
                });
            } else {
                self.reminderOn = reminderOnValue;
                var reminderText = self.reminderOn ? _t('Favorite On') : _t('Set Favorite');
                self.$('.o_wetrack_js_reminder_text').text(reminderText);
                self._updateDisplay();
                var message = self.reminderOn ? _t('Talk added to your Favorites') : _t('Talk removed from your Favorites');
                self.notification.add(message, {
                    type: 'info',
                });
                if (self.reminderOn) {
                    Component.env.bus.trigger('open_notification_request', [
                        'add_track_to_favorite',
                        {
                            title: _t('Allow push notifications?'),
                            body: _t('You have to enable push notifications to get reminders for your favorite tracks.'),
                            delay: 0
                        },
                    ]);
                }
            }
        });
    },

    _updateDisplay: function () {
        var $trackLink = this.$el.find('i');
        if (this.reminderOn) {
            $trackLink.addClass('fa-bell').removeClass('fa-bell-o');
            $trackLink.attr('title', _t('Favorite On'));
        } else {
            $trackLink.addClass('fa-bell-o').removeClass('fa-bell');
            $trackLink.attr('title', _t('Set Favorite'));
        }
    },

});

export default publicWidget.registry.websiteEventTrackReminder;

```

## File: static\src\js\event_track_timer.js

```javascript
/** @odoo-module **/

import { formatDuration } from "@web/core/l10n/dates";
import publicWidget from "@web/legacy/js/public/public_widget";
import { _t } from "@web/core/l10n/translation";
const { DateTime } = luxon;

/*
 * Simple implementation of a timer widget that uses a "time to live" configuration
 * value to countdown seconds on a target element.
 * Will be used to visually countdown the time before a talk starts.
 * When the timer reaches 0, the element destroys itself.
 */
publicWidget.registry.websiteEventTrackTimer = publicWidget.Widget.extend({

    selector: '.o_we_track_timer',
    events: {
        'click .close': '_onCloseClick',
    },

    /**
     * @override
     */
    start: function () {
        return this._super.apply(this, arguments).then(() => {
            let timeToLive = this.$el.data('time-to-live');
            let deadline = DateTime.now().plus({ seconds: timeToLive });
            let remainingMs = deadline.diff(DateTime.now()).as("milliseconds");
            if (remainingMs > 0) {
                this._updateTimerDisplay(remainingMs);
                this.$el.removeClass('d-none');
                this.deadline = deadline;
                this.timer = setInterval(this._refreshTimer.bind(this), 1000);
            } else {
                this.destroy();
            }
        });
    },

    /**
     * @override
     */
    destroy: function () {
        this.$el.parent().remove();
        clearInterval(this.timer);
        this._super(...arguments);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    _onCloseClick: function () {
        this.destroy();
    },

    /**
     * The function will trigger an update if the timer has not yet expired.
     * Otherwise, the component will be destroyed.
     */
    _refreshTimer: function () {
        let remainingMs = this.deadline.diffNow().as("milliseconds");
        if (remainingMs > 0) {
            this._updateTimerDisplay(remainingMs);
        } else {
            this.destroy();
        }
    },

    /**
     * The function will have the responsibility to update the text indicating
     * the time remaining before the counter expires. The function will use
     * Luxon to transform the remaining time in more a human friendly format
     * Example: "in 32 minutes", "in 17 hours", etc.
     * @param {integer} remainingMs - Time remaining before the counter expires (in ms).
     */
    _updateTimerDisplay: function (remainingMs) {
        let $timerTextEl = this.$el.find('span');
        const humanDuration = formatDuration(remainingMs / 1000, true);
        const str = _t("in %s", humanDuration);
        if (str !== $timerTextEl.text()) {
            $timerTextEl.text(str);
        }
    },
});

export default publicWidget.registry.websiteEventTrackTimer;

```

## File: static\src\js\service_worker.js

```javascript
/* eslint-env serviceworker */
/* eslint-disable no-restricted-globals */
/* global idbKeyval */
importScripts("/website_event_track/static/lib/idb-keyval/idb-keyval.js");

const PREFIX = "odoo-event";
const SYNCABLE_ROUTES = ["/event/track/toggle_reminder"];
const CACHABLE_ROUTES = ["/web/webclient/version_info"];
const MAX_CACHE_SIZE = 512 * 1024 * 1024; // 500 MB
const MAX_CACHE_QUOTA = 0.5;
// eslint-disable-next-line no-undef
const CDN_URL = __ODOO_CDN_URL__; // {string|undefined} the cdn_url configured for the website if activated

const { Store, set, get } = idbKeyval;
const pendingRequestsQueueName = `${PREFIX}-pending-requests`;
const cacheName = `${PREFIX}-cache`;
const syncStore = new Store(`${PREFIX}-sync-db`, `${PREFIX}-sync-store`);
const cacheStore = new Store(`${PREFIX}-cache-db`, `${PREFIX}-cache-store`);
const offlineRoute = `${self.registration.scope}/offline`;
const scopeURL = new URL(self.registration.scope);
const cdnURL = CDN_URL ? (CDN_URL.startsWith("http") ? new URL(CDN_URL) : new URL(`http:${CDN_URL}`)) : undefined;

/**
 *
 * @param {string} url
 * @returns {string}
 */
const urlPathname = (url) => new URL(url).pathname;

/**
 *
 * @param {Array} whitelist
 * @returns {Function}
 */
const canHandleRoutes = (whitelist) => (url) => whitelist.includes(urlPathname(url));

/**
 *
 * @param {Request} request
 * @returns {boolean}
 */
const isGET = (request) => request.method === "GET";

/**
 *
 * @returns {Function}
 */
const isSyncableURL = canHandleRoutes(SYNCABLE_ROUTES);

/**
 *
 * @returns {Function}
 */
const isCachableURL = canHandleRoutes(CACHABLE_ROUTES);

/**
 *
 * @returns {boolean} true if navigator has a quota we can read and we reached it
 */
const isCacheFull = async () => {
    if (!("storage" in navigator && "estimate" in navigator.storage)) {
        return false;
    }
    try {
        const { usage, quota } = await navigator.storage.estimate();
        return usage / quota > MAX_CACHE_QUOTA || usage > MAX_CACHE_SIZE;
    } catch (error) {
        console.error(`call to storage.estimate failed`, error);
        return false;
    }
};

/**
 *
 * @return {Promise}
 */
const fetchToCacheOfflinePage = () => caches.open(cacheName).then((cache) => cache.add(offlineRoute));

/**
 *
 * @param {Request} req
 * @returns {Promise<Object>}
 */
const serializeRequest = async (req) => ({
    url: req.url,
    method: req.method,
    headers: Object.fromEntries(req.headers.entries()),
    body: await req.text(),
    mode: req.mode,
    credentials: req.credentials,
    cache: req.cache,
    redirect: req.redirect,
    referrer: req.referrer,
    integrity: req.integrity,
});

/**
 *
 * @param {Object} requestData
 * @returns {Request}
 */
const deserializeRequest = (requestData) => {
    const { url } = requestData;
    delete requestData.url;
    return new Request(url, requestData);
};

/**
 *
 * @param {Response} res
 * @returns {Promise<Object>}
 */
const serializeResponse = async (res) => ({
    body: await res.text(),
    status: res.status,
    statusText: res.statusText,
    headers: Object.fromEntries(res.headers.entries()),
});

/**
 *
 * @param {Object} responseData
 * @returns {Response}
 */
const deserializeResponse = (responseData) => {
    const { body } = responseData;
    delete responseData.body;
    return new Response(body, responseData);
};

/**
 *
 * @param {Object} serializedRequest
 * @returns {string}
 */
const buildCacheKey = ({ url, body: { method, params } }) =>
    JSON.stringify({
        url,
        method,
        params,
    });

/**
 *
 * @returns {int}
 */
const uniqueRequestId = () => Math.floor(Math.random() * 1000 * 1000 * 1000);

/**
 *
 * @returns {Response}
 */
const buildEmptyResponse = () => new Response(JSON.stringify({ jsonrpc: "2.0", id: uniqueRequestId(), result: {} }));

/**
 *
 * @param {Request} request
 * @param {Response} response
 * @returns {Promise}
 */
const cacheRequest = async (request, response) => {
    // only attempts to cache local or cdn delivered urls
    const url = new URL(request.url);
    if (url.hostname !== scopeURL.hostname && (!cdnURL || url.hostname !== cdnURL.hostname)) {
        console.error(`ignoring cache for ${request.url} => ${url.hostname}, local: ${scopeURL.hostname}, cdn: ${cdnURL ? cdnURL.hostname : cdnURL}`);
        return;
    }

    // don't even attempt to cache:
    //  - error pages (why cache that?)
    //  - non-"basic" response types, which include tracker 1-time opaque requests
    //    that are consuming cache space for no reason (namely due to padding MBs accounted for
    //    each opaque request)
    if (!response || !response.ok || response.type !== "basic") {
        console.error(`ignoring cache for ${request.url} => ${response.type}, mode: ${request.mode}, cache: ${request.cache}`);
        return;
    }

    // never blow up cache quota, as it will break things, and the space
    // is shared with cookies and localStorage
    if (await isCacheFull()) {
        // TODO: clear some part of the cache to free older/less-relevant content
        console.log("Cache full, not caching!");
        return;
    }

    console.log(`grant cache for ${request.url} => ${response.type}, mode: ${request.mode}, cache: ${request.cache},
                    isGet: ${isGET(request)}, isCachable: ${isCachableURL(request.url)}`);
    if (isGET(request)) {
        const cache = await caches.open(cacheName);
        await cache.put(request, response.clone());
    } else if (isCachableURL(request.url)) {
        const serializedRequest = await serializeRequest(request);
        const serializedResponse = await serializeResponse(response.clone());
        await set(buildCacheKey(serializedRequest), serializedResponse, cacheStore);
    }
};

/**
 *
 * @param {Request} request
 * @returns {boolean}
 */
const isCachableRequest = (request) => isGET(request) || isCachableURL(request.url);

/**
 *
 * @param request
 * @param requestError
 * @return {boolean}
 */
const isOfflineDocumentRequest = (request, requestError) =>
    request && requestError && requestError.message === 'Failed to fetch' && (
        (isGET(request) && request.mode === 'navigate' && request.destination === 'document') ||
        // request.mode = navigate isn't supported in all browsers => check for http header accept:text/html
        (request.method === 'GET' && request.headers.get('accept').includes('text/html'))
    );

/**
 *
 * @param {Request} request
 * @returns {Promise<Response|null>}
 */
const matchCache = async (request) => {
    if (isGET(request)) {
        const cache = await caches.open(cacheName);
        const response = await cache.match(request.url);
        if (response) {
            return deserializeResponse(await serializeResponse(response.clone()));
        }
    }
    if (isCachableURL(request.url)) {
        const serializedRequest = await serializeRequest(request);
        const cachedResponse = await get(buildCacheKey(serializedRequest), cacheStore);
        if (cachedResponse) {
            return deserializeResponse(cachedResponse);
        }
    }
    return null;
};

/**
 *
 * @param {Request} request
 * @param {object} [options]
 * @param {Boolean} [options.disableTracking] whether adding a header preventing the server to track the request
 * @returns {Promise<Response>}
 */
const processFetchRequest = async (request, options) => {
    const requestCopy = request.clone();
    let response;
    try {
        if (options && options.disableTracking) {
            response = await fetch(request, { headers: { 'X-Disable-Tracking': '1' } });
        } else {
            response = await fetch(request);
        }
        await cacheRequest(request, response);
    } catch (requestError) {
        if (isCachableRequest(requestCopy)) {
            try {
                response = await matchCache(requestCopy);
            } catch (err) {
                console.warn("An error occurs when reading the cache request", err);
            }
        } else if (isSyncableURL(requestCopy.url)) {
            const pendingRequests = (await get(pendingRequestsQueueName, syncStore)) || [];
            const serializedRequest = await serializeRequest(requestCopy);
            await set(pendingRequestsQueueName, [...pendingRequests, serializedRequest], syncStore);
            if (self.registration.sync) {
                await self.registration.sync.register(pendingRequestsQueueName).catch((err) => {
                    console.warn("Cannot use BackgroundSync", err);
                    throw requestError;
                });
            }
            return buildEmptyResponse();
        } else {
            console.warn(`Offline ${requestCopy.method} request currently not supported`, requestCopy);
        }

        if (!response) {
            if (isOfflineDocumentRequest(request, requestError)) {
                const cache = await caches.open(cacheName);
                return await cache.match(offlineRoute);
            }
            throw requestError;
        }
    }
    return response;
};

/**
 *
 * @returns {Promise}
 */
const processPendingRequests = async () => {
    const pendingRequests = (await get(pendingRequestsQueueName, syncStore)) || [];
    if (!pendingRequests.length) {
        console.info("Nothing to sync!");
        return;
    }
    let pendingRequest;
    while ((pendingRequest = pendingRequests.shift())) {
        const request = deserializeRequest(pendingRequest);
        await fetch(request);
        await set(pendingRequestsQueueName, pendingRequests, syncStore);
    }
};

/**
 * Add given urls to the Cache, skipping the ones already present
 * @param {Array<string>} urls
 */
const prefetchUrls = async (urls = []) => {
    const cache = await caches.open(cacheName);
    const uniqUrls = new Set(urls);
    for (let url of uniqUrls) {
        if (await cache.match(url)) {
            continue;
        }
        try {
            await processFetchRequest(new Request(url), { disableTracking: true });
        } catch (error) {
            console.error(`fail to prefetch ${url} : ${error}`);
        }
    }
};

/**
 * Handle the message sent to the Worker (using the postMessage() method).
 * The message is defined by the name of the action to perform and its associated parameters (optional).
 *
 * Actions:
 * - prefetch-pages: add {Array} urls with their "alternative url" to the Cache (if not already present).
 * - prefetch-assets: add {Array} urls to the Cache (if not already present).
 *
 * @param {Object} data
 * @param {string} data.action action's name
 * @param {*} data.* action's parameter(s)
 * @returns {Promise}
 */
const processMessage = (data) => {
    const { action } = data;
    switch (action) {
        case "prefetch-pages":
            const { urls: pagesUrls } = data;
            // To prevent redirection cached by the browser (cf. 301 Permanently Moved) from breaking the offline cache
            // we also add alternative urls with the following rule:
            // * if original url has a trailing "/", adds url with striped trailing "/"
            // * if original url doesn't end with "/", adds url without the trailing "/"
            const maybeRedirectedUrl = pagesUrls.map((url) => (url.endsWith("/") ? url.slice(0, -1) : url));
            return prefetchUrls([...pagesUrls, ...maybeRedirectedUrl]);
        case "prefetch-assets":
            const { urls: assetsUrls } = data;
            return prefetchUrls(assetsUrls);
    }
    throw new Error(`Action '${action}' not found.`);
};

self.addEventListener("fetch", (event) => {
    event.respondWith(processFetchRequest(event.request));
});

self.addEventListener("sync", (event) => {
    console.info(`Syncing pending requests...`);
    if (event.tag === pendingRequestsQueueName) {
        event.waitUntil(processPendingRequests());
    }
});

self.addEventListener("message", (event) => {
    event.waitUntil(processMessage(event.data));
});

// Precache static resources here. Like offline page
self.addEventListener('install', (event) => {
    event.waitUntil(fetchToCacheOfflinePage());
});

```

## File: static\src\js\website_event_pwa_widget.js

```javascript
/** @odoo-module **/

    /*
     * The "deferredPrompt" Promise will resolve only if the "beforeinstallprompt" event
     * has been triggered. It allows to register this listener as soon as possible
     * to avoid missed-events (as the browser can trigger it very early in the page lifecycle).
     */
    var deferredPrompt = new Promise(function (resolve, reject) {
        if ("serviceWorker" in navigator) {
            window.addEventListener("beforeinstallprompt", function (ev) {
                ev.preventDefault();
                resolve(ev);
            });
        } else {
            console.log("ServiceWorker not supported");
        }
    });

    import publicWidget from "@web/legacy/js/public/public_widget";
    import { utils as uiUtils } from "@web/core/ui/ui_service";

    var PWAInstallBanner = publicWidget.Widget.extend({
        template: "pwa_install_banner",
        events: {
            "click .o_btn_install": "_onClickInstall",
            "click .o_btn_close": "_onClickClose",
        },

        /**
         * @private
         */
        _onClickClose: function () {
            this.trigger_up("prompt_close_bar");
        },

        /**
         * @private
         */
        _onClickInstall: function () {
            this.trigger_up("prompt_install");
        },
    });

    publicWidget.registry.WebsiteEventPWAWidget = publicWidget.Widget.extend({
        selector: "#wrapwrap.event",
        custom_events: {
            prompt_install: "_onPromptInstall",
            prompt_close_bar: "_onPromptCloseBar",
        },

        /**
         *
         * @override
         */
        start: function () {
            var self = this;
            return this._super.apply(this, arguments)
                .then(this._registerServiceWorker.bind(this))
                .then(function () {
                    // Don't wait for the prompt's Promise as it may never resolve.
                    deferredPrompt.then(self._showInstallBanner.bind(self));
                })
                .then(this._prefetch.bind(this));
        },

        /**
         *
         * @override
         */
        destroy: function () {
            this._super.apply(this, arguments);
        },

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        /**
         * Returns the PWA's scope
         *
         * Note: this method performs a matching to handle URLs with the language prefix.
         *       Typically this prefix is in the form of "en" or "en_US" but it can also be
         *       any string using the customization options in the Website's settings.
         * @private
         * @returns {String}
         */
        _getScope: function () {
            var matches = window.location.pathname.match(/^(\/(?:event|[^/]+\/event))\/?/);
            if (matches && matches[1]) {
                return matches[1];
            }
            return "/event";
        },

        /**
         * @private
         */
        _hideInstallBanner: function () {
            this.installBanner ? this.installBanner.destroy() : undefined;
            $(".o_livechat_button").css("bottom", "0");
        },

        /**
         * Parse the current page for first-level children pages and ask the ServiceWorker
         * to already fetch them to populate the cache.
         * @private
         */
        _prefetch: function () {
            if (!("serviceWorker" in navigator)) {
                return;
            }
            var assetsUrls = Array.from(document.querySelectorAll('link[rel="stylesheet"], script[src]')).map(function (el) {
                return el.href || el.src;
            });
            navigator.serviceWorker.ready.then(function (registration) {
                registration.active.postMessage({
                    action: "prefetch-assets",
                    urls: assetsUrls,
                });
            }).catch(function (error) {
                console.error("Service worker ready failed, error:", error);
            });
            var currentPageUrl = window.location.href;
            var childrenPagesUrls = Array.from(document.querySelectorAll('a[href^="' + this._getScope() + '/"]')).map(function (el) {
                return el.href;
            });
            navigator.serviceWorker.ready.then(function (registration) {
                registration.active.postMessage({
                    action: "prefetch-pages",
                    urls: childrenPagesUrls.concat(currentPageUrl),
                });
            }).catch(function (error) {
                console.error("Service worker ready failed, error:", error);
            });
        },

        /**
         *
         * @private
         */
        _registerServiceWorker: function () {
            if (!("serviceWorker" in navigator)) {
                return;
            }
            var scope = this._getScope();
            return navigator.serviceWorker
                .register(scope + "/service-worker.js", { scope: scope })
                .catch(function (error) {
                    console.error("Service worker registration failed, error:", error);
                });
        },

        /**
         * @private
         */
        _showInstallBanner: function () {
            if (!uiUtils.isSmall()) {
                return;
            }
            var self = this;
            this.installBanner = new PWAInstallBanner(this);
            this.installBanner.appendTo(this.$el).then(function () {
                // If Livechat available, It should be placed above the PWA banner.
                var height = self.$(".o_pwa_install_banner").outerHeight(true);
                $(".o_livechat_button").css("bottom", height + "px");
            });
        },

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------

        /**
         * @private
         * @param ev {Event}
         */
        _onPromptCloseBar: function (ev) {
            ev.stopPropagation();
            this._hideInstallBanner();
        },
        /**
         * @private
         * @param ev {Event}
         */
        _onPromptInstall: function (ev) {
            ev.stopPropagation();
            this._hideInstallBanner();
            deferredPrompt.then(function (prompt) {
                    prompt.prompt();
                    prompt.userChoice.then(function (choiceResult) {
                        if (choiceResult.outcome === "accepted") {
                            console.log("User accepted the install prompt");
                        } else {
                            console.log("User dismissed the install prompt");
                        }
                    });
                });
        },
    });

    export default {
        PWAInstallBanner: PWAInstallBanner,
        WebsiteEventPWAWidget: publicWidget.registry.WebsiteEventPWAWidget,
    };

```

## File: static\src\js\website_event_track.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import { debounce } from "@web/core/utils/timing";

publicWidget.registry.websiteEventTrack = publicWidget.Widget.extend({
    selector: '.o_wevent_event',
    events: {
        'input #event_track_search': '_onEventTrackSearchInput',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    start: function () {
        this._super.apply(this, arguments).then(() => {
            this.$el.find('[data-bs-toggle="popover"]').popover();

            this.agendas = Array.from(this.target.getElementsByClassName('o_we_online_agenda'));

            if (this.agendas.length > 0) {
                this._checkAgendasOverflow(this.agendas);
                this.agendaScroller = this.$el[0].querySelector('.o_we_agenda_horizontal_scroller_container');
                this.agendaScrollerElement = this.agendaScroller.querySelector('.o_we_agenda_horizontal_scroller');

                this.agendas.forEach(agenda => {
                    agenda.addEventListener('scroll', event => {
                        this._onAgendaScroll(agenda, event);
                        if(this.agendaScroller && this.visibleAgenda) {
                            this.agendaScroller.scrollLeft = this.visibleAgenda.scrollLeft;
                        }
                    });
                });

                if (this.agendaScroller) {
                    this._updateAgendaScroll = debounce(this._updateAgendaScroll, 50);
                    if (document.querySelector('#wrapwrap.event')) {
                        window.addEventListener('scroll',
                            this._updateAgendaScroll.bind(this)
                        );
                    }

                    window.addEventListener('resize', () => {
                        this._updateAgendaScroll();
                    });

                    this.agendaScroller.addEventListener('scroll', () => {
                        if (this.visibleAgenda) {
                            this.visibleAgenda.scrollLeft = this.agendaScroller.scrollLeft;
                        }
                    });

                    this._updateAgendaScroll();
                }
            }
        });
    },

    /**
     * Dynamic horizontal scrollbar.
     * It's meant the show up as a sticky scrollbar at the bottom of the screen, to allow scrolling
     * the agenda horizontally even if you've not reached the bottom of the agenda container.
     * Makes the user experience much smoother.
     *
     * Technically, the code checks "what is the last agenda on the screen" and enables our sticky
     * scrollbar based on that.
     */
    _updateAgendaScroll() {
        // reverse the agendas, we always want the last agenda "on screen" to be the scrolled one
        this.visibleAgenda = this.agendas.toReversed().find((el) => {
            const rect = el.getBoundingClientRect();
            let containerOffset = {
                top: rect.top + window.scrollY + 30,  // some offset for a better experience
                bottom: rect.bottom + window.scrollY
            };
            let windowOffset = {
                top: window.scrollY,
                bottom: window.scrollY + window.innerHeight
            };

            // if the top of the container if visible but NOT the bottom
            return (containerOffset.top < windowOffset.bottom) &&
                !(containerOffset.bottom < windowOffset.bottom);
        });

        if (this.visibleAgenda && this.visibleAgenda.classList.contains('o_we_online_agenda_has_content_hidden')) {
            // need to account for vertical scrollbar width
            const verticalScrollbarWidth = window.innerWidth - document.documentElement.clientWidth;

            this.agendaScroller.classList.remove('d-none');
            this.agendaScrollerElement.style.width = (this.visibleAgenda.scrollWidth + verticalScrollbarWidth) + 'px';
        } else {
            this.agendaScroller.classList.add('d-none');
        }
    },

    /**
     * @private
     * @param {Object} agendas
     */
    _checkAgendasOverflow: function (agendas) {
        agendas.forEach(agendaEl => {
            const hasScroll = agendaEl.querySelector('table').clientWidth > agendaEl.clientWidth;

            agendaEl.classList.toggle('o_we_online_agenda_has_scroll', hasScroll);
            agendaEl.classList.toggle('o_we_online_agenda_has_content_hidden', hasScroll);
        });
    },

    /**
     * @private
     * @param {Object} agendaEl
     * @param {Event} event
     */
    _onAgendaScroll: function (agendaEl, event) {
        const tableEl = agendaEl.querySelector('table');
        const gutter = 4; // = map-get($spacers, 1)
        const gap = tableEl.clientWidth - agendaEl.clientWidth - gutter;

        agendaEl.classList.add('o_we_online_agenda_is_scrolling');
        agendaEl.classList.toggle('o_we_online_agenda_has_content_hidden', gap > Math.ceil(agendaEl.scrollLeft));

        requestAnimationFrame(() => {
            setTimeout(() => {
                agendaEl.classList.remove('o_we_online_agenda_is_scrolling');
            }, 200);
        });
    },

    /**
     * @private
     * @param {Event} ev
     */
    _onEventTrackSearchInput: function (ev) {
        ev.preventDefault();
        var text = $(ev.currentTarget).val();
        var $tracks = $('.event_track');

        //check if the user is performing a search; i.e., text is not empty
        if (text) {
            function filterTracks(index, element) {
                //when filtering elements only check the text content
                return this.textContent.toLowerCase().includes(text.toLowerCase());
            }
            $('#search_summary').removeClass('invisible');
            $('#search_number').text($tracks.filter(filterTracks).length);

            $tracks.removeClass('invisible').not(filterTracks).addClass('invisible');
        } else {
            //if no search is being performed; hide the result count text
            $('#search_summary').addClass('invisible');
            $tracks.removeClass('invisible')
        }
    },
});

```

## File: static\src\js\website_event_track_proposal_form.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import { _t } from "@web/core/l10n/translation";
import { renderToElement } from "@web/core/utils/render";
import { scrollTo } from "@web_editor/js/common/scrolling";

publicWidget.registry.websiteEventTrackProposalForm = publicWidget.Widget.extend({
    selector: '.o_website_event_track_proposal_form',
    events: {
        'click .o_wetrack_add_contact_information_checkbox': '_onAdvancedContactToggle',
        'input input[name="partner_name"]': '_onPartnerNameInput',
        'click .o_wetrack_proposal_submit_button': '_onProposalFormSubmit',
    },

    /**
     * @override
     */
    init: function () {
        this._super(...arguments);
        this.useAdvancedContact = false;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Evaluate and return validity of form input fields:
     * - 1) error 'invalidFormInputs' : Invalid ones are marked as is-invalid and o_wetrack_input_error.
     * - 2) error 'noContactMean' : Contact mean fields are marked as is-invalid and contact
     * section as o_wetrack_no_contact_mean_error if none of them is filled.
     *
     * @private
     * @returns {Boolean} - True if no error remain, false otherwise
     */
    _isFormValid: function () {
        var formErrors = [];

        // 1) Valid Form Inputs
        this.$('.form-control').each(function () {
            var $formControl = $(this);
            // Validate current input, if not SelectMenu field.
            var inputs = $formControl.not(".o_wetrack_select_tags");
            var invalidInputs = inputs.toArray().filter(function (input) {
                return !input.checkValidity();
            });

            $formControl.removeClass('o_wetrack_input_error is-invalid');
            if (invalidInputs.length) {
                $formControl.addClass('o_wetrack_input_error is-invalid');
                formErrors.push('invalidFormInputs');
            }
        });

        // 2) Advanced Contact Must Have a Contact Mean
        if (this.useAdvancedContact) {
            var hasContactMean = this.$('.o_wetrack_contact_phone_input').val() ||
                this.$('.o_wetrack_contact_email_input').val();
            if (!hasContactMean) {
                this.$('.o_wetrack_contact_information').addClass('o_wetrack_no_contact_mean_error');
                this.$('.o_wetrack_contact_mean').addClass('is-invalid');
                formErrors.push('noContactMean');
            } else {
                this.$('.o_wetrack_contact_information').removeClass('o_wetrack_no_contact_mean_error');
                this.$('.o_wetrack_contact_mean:not(".o_wetrack_input_error")').removeClass('is-invalid');
            }
        }

        // Form Validity and Error Display
        this._updateErrorDisplay(formErrors);
        return formErrors.length === 0;
    },

    /**
     * If there are still errors in form, display the error section and
     * compose the error message accordingly.
     *
     * @private
     * @param {Array} errors - Names of errors still present in form.
     */
    _updateErrorDisplay: function (errors) {

        this.$('.o_wetrack_proposal_error_section').toggleClass('d-none', !errors.length);

        var errorMessages = [];
        var $errorElement = this.$('.o_wetrack_proposal_error_message');

        if (errors.includes('invalidFormInputs')) {
            errorMessages.push(_t('Please fill out the form correctly.'));
        }

        if (errors.includes('noContactMean')) {
            errorMessages.push(_t('Please enter either a contact email address or a contact phone number.'));
        }

        if (errors.includes('forbidden')) {
            errorMessages.push(_t('You cannot access this page.'));
        }

        $errorElement.text(errorMessages.join(' ')).change();
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Display / Hide Additional Contact Information section when toggling
     * the checkbox on the form o_wetrack_add_contact_information_checkbox.
     * Also empty the email to prevent hidden email format error.
     *
     * @private
     * @param {Event} ev
     */
    _onAdvancedContactToggle: function (ev) {
        this.useAdvancedContact = !this.useAdvancedContact;
        var $contactName = this.$(".o_wetrack_contact_name_input")[0];
        var $advancedInformation = this.$('.o_wetrack_contact_information');

        if (this.useAdvancedContact) {
            $advancedInformation.removeClass('d-none');
            $contactName.setAttribute("required", "True");
        } else {
            this.$('.o_wetrack_contact_email_input').val('').change();
            $advancedInformation.addClass('d-none');
            $contactName.removeAttribute("required");
        }
    },

    /**
     * Propagates the new input on speaker name to contact name, as long as the latter
     * is the start of partner name. Otherwise, do not modify existing contact name.
     *
     * @private
     * @param {Event} ev
     */
    _onPartnerNameInput: function (ev) {
        var partnerNameText = $(ev.currentTarget).val();
        var contactNameText = this.$(".o_wetrack_contact_name_input").val();
        if (partnerNameText.startsWith(contactNameText)) {
            this.$(".o_wetrack_contact_name_input").val(partnerNameText).change();
        }
    },

    /**
     * Submits the form if no errors are present in the form after validation.
     *
     * If the submission succeeds, we replace the form with a template containing a small success
     * message.
     *
     * Then we scroll to the position of the success message so that the user can see it.
     * To do that we have to compute the position of the beginning of the element, relatively to its
     * position and the amount already scrolled, then subtract the floating header menu.
     *
     * @private
     * @param {Event} ev
     */
    _onProposalFormSubmit: async function (ev) {
        ev.preventDefault();
        ev.stopPropagation();

        // Prevent further clicking
        this.$el.find('.o_wetrack_proposal_submit_button')
            .addClass('disabled')
            .attr('disabled', 'disabled');

        // Submission of the form if no errors remain
        if (this._isFormValid()) {
            const formData = new FormData(this.$el[0]);

            const response = await $.ajax({
                url: `/event/${encodeURIComponent(this.$el.data('eventId'))}/track_proposal/post`,
                data: formData,
                processData: false,
                contentType: false,
                type: 'POST'
            });

            const jsonResponse = response && JSON.parse(response);
            if (jsonResponse.success) {
                // TODO we really should not remove the whole widget element
                // like that + probably restore the widget before edit mode etc.
                const parentEl = this.el.parentNode;
                this.$el.replaceWith($(renderToElement('event_track_proposal_success')));
                scrollTo(parentEl, { extraOffset: 20, duration: 50 });
            } else if (jsonResponse.error) {
                this._updateErrorDisplay([jsonResponse.error]);
            }
        }

        // Restore button
        this.$el.find('.o_wetrack_proposal_submit_button')
            .removeAttr('disabled')
            .removeClass('disabled');
    },
});

export default publicWidget.registry.websiteEventTrackProposalForm;

```

## File: static\src\js\website_event_track_proposal_form_tags.js

```javascript
import { Component, useState } from "@odoo/owl";
import { _t } from "@web/core/l10n/translation";
import { rpc } from "@web/core/network/rpc";
import { SelectMenu } from "@web/core/select_menu/select_menu";
import publicWidget from "@web/legacy/js/public/public_widget";
import { attachComponent } from "@web_editor/js/core/owl_utils";

class WebsiteEventTrackProposalFormTagsWrapper extends Component {
    static template = "website_event_track.WebsiteEventTrackProposalFormTagsWrapper";
    static components = { SelectMenu };
    static props = {
        placeholder: { optional: true, type: String },
        defaultChoices: { optional: true, type: Array },
    };

    setup() {
        this.state = useState({
            ...this.props,
            value: [],
        });
    }
    onSelect(item) {
        this.state.value = item;
    }
}

publicWidget.registry.websiteEventTrackProposalFormTags = publicWidget.Widget.extend({
    selector: '.o_website_event_track_proposal_form_tags',

    async willStart() {
        const choices = await rpc("/event/track_tag/search_read", {
            fields: ["id", "name", "category_id"],
            domain: [["color", "!=", 0]],
        });
        this.choices = choices.map(({ id, category_id, name }) => {
            return {
                value: id,
                label: category_id ? `${category_id[1]} : ${name}` : name,
            };
        });
    },

    async start() {
        await this._super(...arguments);
        await attachComponent(this, this.el, WebsiteEventTrackProposalFormTagsWrapper, {
            defaultChoices: this.choices || [],
            placeholder: _t("Select Categories"),
        });
    },
});

export default publicWidget.registry.websiteEventTrackProposalFormTags;

```

## File: static\src\xml\event_track_proposal_templates.xml

```xml
<templates>

    <t t-name="event_track_proposal_success">
        <section class="my-5">
            <h3 class="o_page_header">Application</h3>
            <p>Thank you for your proposal.</p>
            <p>We will evaluate your proposition and get back to you shortly.</p>
        </section>
    </t>

</templates>

```

## File: static\src\xml\website_event_pwa.xml

```xml
<templates>

    <t t-name="pwa_install_banner">
        <div class="alert alert-default o_pwa_install_banner">
            <button class="btn o_btn_close">x</button>
            <strong>Install Application</strong>
            <button class="btn btn-primary o_btn_install">Install</button>
        </div>
    </t>

</templates>

```

## File: static\src\xml\website_event_track_form_tags_wrapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<t t-name="website_event_track.WebsiteEventTrackProposalFormTagsWrapper">
    <input name="tags" type="hidden" class="form-control o_wetrack_select_tags d-none" t-att-value="state.value"/>
    <SelectMenu
        choices="state.defaultChoices"
        multiSelect="true"
        searchable="true"
        onSelect.bind="onSelect"
        value="state.value"
        placeholder="state.placeholder"/>
</t>

</templates>

```

## File: views\event_event_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="event_event_view_form" model="ir.ui.view">
        <field name="name">event.event.view.from.inherit.track</field>
        <field name="inherit_id" ref="website_event.event_event_view_form"/>
        <field name="model">event.event</field>
        <field name="priority" eval="3"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='website_url']" position="before">
                <button name="%(action_event_track_from_event)d"
                        type="action"
                        class="oe_stat_button"
                        icon="fa-microphone">
                    <field name="track_count" string="Tracks" widget="statinfo"/>
                </button>
            </xpath>
            <xpath expr="//field[@name='website_menu']" position="after">
                <label for="website_track" string="Showcase Tracks"/>
                <field name="website_track"/>
                <label for="website_track_proposal" string="Allow Track Proposals"/>
                <field name="website_track_proposal"/>
            </xpath>
        </field>
    </record>

    <record id="event_event_view_list" model="ir.ui.view">
        <field name="name">event.event.view.list.inherit.website.event.track</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="event.view_event_tree"/>
        <field name="arch" type="xml">
            <field name="stage_id" position="after">
                <field name="track_count" readonly="1" optional="hide"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\event_menus.xml

```xml
<?xml version="1.0"?>
<odoo><data>

    <!-- MAIN ITEMS -->
    <menuitem id="menu_event_track"
        name="Tracks"
        sequence="40"
        action="action_event_track"
        parent="event.event_main_menu"
        groups="base.group_no_one"/>

    <!-- CONFIGURATION -->
    <menuitem id="event_track_stage_menu"
        name="Track Stages"
        action="event_track_stage_action"
        parent="event.menu_event_configuration"
        groups="base.group_no_one"
        sequence="30"/>

    <menuitem id="menu_event_track_location"
        name="Track Locations"
        action="action_event_track_location"
        parent="event.menu_event_configuration"
        sequence="32"/>

    <menuitem id="event_track_tag_category_menu"
        name="Track Tag Categories"
        action="event_track_tag_category_action"
        parent="event.menu_event_configuration"
        groups="base.group_no_one"
        sequence="33"/>

    <menuitem
        id="menu_event_track_tag"
        name="Track Tags"
        action="action_event_track_tag"
        parent="event.menu_event_configuration"
        groups="base.group_no_one"
        sequence="34"/>

    <menuitem id="event_track_visitor_menu"
        name="Track Visitors"
        sequence="38"
        action="event_track_visitor_action"
        parent="event.menu_event_configuration"
        groups="base.group_no_one"/>

</data></odoo>

```

## File: views\event_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- Add a shortcut to favorites / tracks after registration -->
<template id="registration_complete" inherit_id="website_event.registration_complete">
    <xpath expr="//div[hasclass('row')][last()]" position="after">
        <div t-if="event.website_track" class="row mt-5 mb256">
            <div class="col-12">
                <h3>Book your seats to the best talks</h3>
                <p>Get prepared and
                    <a t-att-href="'/event/%s/track' % (slug(event))">register to your favorites talks now.</a>
                </p>
            </div>
        </div>
    </xpath>
</template>

<template id="event_confirmation_end_page_hook" inherit_id="website_event.event_confirmation_end_page_hook">
    <xpath expr="//div[hasclass('o_wevent_confirmation_end_page_hook')]" position="inside">
        <div t-if="event.website_track" class="row mt-5 mb-3">
            <div class="col-12">
                <h3>Book your seats to the best talks</h3>
                <p>Get prepared and
                    <a t-att-href="'/event/%s/track' % (slug(event))">register to your favorites talks now.</a>
                </p>
            </div>
        </div>
    </xpath>
</template>

<template id="layout" inherit_id="website_event.layout">
    <xpath expr='//t[@t-call="website.layout"]' position="inside">
        <t t-set="pageName" t-value="'event'"/>
        <t t-set="head">
            <t t-out="head"/>
            <t t-call="website_event_track.pwa_manifest"/>
        </t>
    </xpath>
</template>

<template id="index" inherit_id="website_event.index">
    <xpath expr='//t[@t-call="website.layout"]' position="inside">
        <t t-set="pageName" t-value="'event'"/>
        <t t-set="head">
            <t t-out="head"/>
            <t t-call="website_event_track.pwa_manifest"/>
        </t>
    </xpath>
</template>

<template id="pwa_manifest">
    <link rel="manifest" href="/event/manifest.webmanifest" crossorigin="use-credentials"/>
    <link rel="apple-touch-icon" t-att-href="website.image_url(website, 'app_icon', size='192x192')"/>
    <meta name="theme-color" content="#875A7B"/>
</template>

<template id="pwa_offline" inherit_id="website_event.index" primary="True">
    <xpath expr='//div[@id="wrap"]' position="replace">
        <div id="wrap" class="o_wevent_index">
            <div class="container">
                <div class="row">
                    <div class="col-12 card-body">
                        <div class="h2 mb-3">You're offline!</div>
                        <div class="alert alert-info text-center">
                            <span class="fa-stack fa-4x">
                                <i class="fa fa-wifi fa-stack-1x"></i>
                                <i class="fa fa-ban fa-stack-2x text-muted"></i>
                            </span>
                            <p>This page hasn't been saved for offline reading yet.<br/>Please check your network connection.</p>
                            <p>
                                <a t-att-href="url_for('/event')" class="btn btn-primary btn-block">Home page</a>
                                <button onclick="history.back();" class="btn btn-secondary btn-block">Previous page</button>
                            </p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <script type="text/javascript">
            window.addEventListener('online', function(e) {
                console.log('Go back online');
                location.reload();
            });
        </script>
    </xpath>
</template>

</odoo>

```

## File: views\event_track_location_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <!-- EVENTS/CONFIGURATION/EVENT locations -->
    <record model="ir.ui.view" id="view_event_location_form">
        <field name="name">Event Locations</field>
        <field name="model">event.track.location</field>
        <field name="arch" type="xml">
            <form string="Event Locations">
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="sequence"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record model="ir.ui.view" id="view_event_location_tree">
        <field name="name">Event Location</field>
        <field name="model">event.track.location</field>
        <field name="arch" type="xml">
            <list string="Event Location" editable="bottom">
                <field name="sequence" widget="handle"/>
                <field name="name" placeholder='e.g. "Main Conference Room"'/>
            </list>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_event_track_location">
        <field name="name">Event Locations</field>
        <field name="res_model">event.track.location</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Track Location
            </p><p>
                Manage from here the places where you organize your tracks (e.g. Rooms, Channels, ...).
            </p>
        </field>
    </record>

</odoo>

```

## File: views\event_track_stage_views.xml

```xml
<?xml version="1.0"?>
<odoo>

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
                            <field name="is_visible_in_agenda" readonly="is_cancel"/>
                            <field name="is_fully_accessible" readonly="is_cancel"/>
                            <field name="is_cancel"/>
                        </group>
                        <group>
                            <field name="fold"/>
                            <field name="color" widget="color_picker"/>
                        </group>
                    </group>
                    <group string="Stage Description and Tooltips">
                        <p class="text-muted" colspan="2">
                            Define labels explaining kanban state management.
                        </p>
                        <label for="legend_normal" string=" " class="o_status"
                            title="Task in progress. Click to block or set as done."
                            aria-label="Task in progress. Click to block or set as done." role="img"/>
                        <field name="legend_normal" nolabel="1"/>
                        <label for="legend_blocked" string=" " class="o_status o_status_red"
                            title="Task is blocked. Click to unblock or set as done."
                            aria-label="Task is blocked. Click to unblock or set as done." role="img"/>
                        <field name="legend_blocked" nolabel="1"/>
                        <label for="legend_done" string=" " class="o_status o_status_green"
                            title="This step is done. Click to block or set in progress."
                            aria-label="This step is done. Click to block or set in progress." role="img"/>
                        <field name="legend_done" nolabel="1"/>
                        <p class="text-muted" colspan="2">
                            Add a description to help your coworkers understand the meaning and purpose of the stage.
                        </p>
                        <field name="description" placeholder="Add a description..." nolabel="1" colspan="2"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="event_track_stage_view_tree" model="ir.ui.view">
        <field name="name">event.track.stage.view.list</field>
        <field name="model">event.track.stage</field>
        <field name="arch" type="xml">
            <list string="Track Stage">
                <field name="sequence" widget="handle" groups="event.group_event_manager"/>
                <field name="name"/>
                <field name="is_visible_in_agenda"/>
                <field name="is_fully_accessible"/>
                <field name="is_cancel"/>
                <field name="fold"/>
            </list>
        </field>
    </record>

    <record id="view_event_track_stage_kanban" model="ir.ui.view">
        <field name="name">event.track.stage.kanban</field>
        <field name="model">event.track.stage</field>
        <field name="arch" type="xml">
            <kanban>
                <templates>
                    <t t-name="card">
                        <field name="name" class="fw-medium fs-5"/>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="event_track_stage_action" model="ir.actions.act_window">
        <field name="name">Track Stages</field>
        <field name="res_model">event.track.stage</field>
        <field name="view_mode">list,kanban,form</field>
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

</odoo>

```

## File: views\event_track_tag_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="event_track_tag_category_view_form" model="ir.ui.view">
        <field name="name">event.track.tag.category.view.form</field>
        <field name="model">event.track.tag.category</field>
        <field name="arch" type="xml">
            <form string="Track Tag Category">
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="tag_ids" context="{'default_category_id': id}">
                            <list string="Tags" editable="bottom">
                                <field name="sequence" widget="handle"/>
                                <field name="name"/>
                                <field name="color" widget="color_picker"/>
                            </list>
                        </field>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="event_track_tag_category_view_list" model="ir.ui.view">
        <field name="name">event.track.tag.category.view.list</field>
        <field name="model">event.track.tag.category</field>
        <field name="arch" type="xml">
            <list string="Track Tags Category">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
            </list>
        </field>
    </record>

    <record id="event_track_tag_category_action" model="ir.actions.act_window">
        <field name="name">Track Tag Categories</field>
        <field name="res_model">event.track.tag.category</field>
        <field name="view_mode">list,form</field>
    </record>

    <record model="ir.ui.view" id="view_event_track_tag_form">
        <field name="name">Track Tags</field>
        <field name="model">event.track.tag</field>
        <field name="arch" type="xml">
            <form string="Event Track Tag">
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="color" widget="color_picker"/>
                    <field name="category_id"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record model="ir.ui.view" id="view_event_track_tag_tree">
        <field name="name">Tracks Tag</field>
        <field name="model">event.track.tag</field>
        <field name="arch" type="xml">
            <list string="Event Track Tag" editable="bottom">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="category_id"/>
                <field name="color" widget="color_picker"/>
            </list>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_event_track_tag">
        <field name="name">Track Tags</field>
        <field name="res_model">event.track.tag</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Track Tag
            </p><p>
                Add tags to your tracks to help your attendees browse your event web pages.
            </p>
        </field>
    </record>

</odoo>

```

## File: views\event_track_templates_agenda.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- Revamped agenda : Will need to replace agenda_online from website_event_track in master -->
<template id="agenda_online" name="Track Online: Agenda">
    <t t-call="website_event.layout">
        <!-- No tracks -->
        <div t-if="not tracks_by_days" class="container">
            <div class="row">
                <div class="col-12 text-center">
                    <div t-call="website_event.event_empty_events_svg" class="my-4"/>
                    <h2 class="mb-2">No track found.</h2>
                    <p t-if="search_key">We could not find any track matching your search for: <strong t-out="search_key"/>.</p>
                    <p t-else="">We could not find any track at this moment.</p>
                    <div class="o_not_editable my-3" groups="event.group_event_user">
                        <a class="o_wevent_cta btn" target="_blank" t-att-href="'/odoo/%s/action-website_event_track.action_event_track_from_event' % event.id">
                            <span class="fa fa-plus me-1"/> Schedule Tracks
                        </a>
                    </div>
                </div>
            </div>
        </div>

        <t t-else="">
            <div class="container">
                <!-- Options -->
                <t t-set="option_track_wishlist" t-value="not event.is_done and is_view_active('website_event_track.agenda_topbar_wishlist')"/>
                <!-- Topbar -->
                <t t-call="website_event_track.agenda_topbar"/>
            </div>
            <!-- Drag/Drop Area -->
            <t t-set="editor_message">Available on all event Agenda pages</t>
            <div class="oe_structure oe_empty" id="oe_structure_website_event_track_agenda_1" t-att-data-editor-message="editor_message"/>

            <!-- Content -->
            <div class="o_wevent_online o_weagenda_index mb-3">
                <t t-call="website_event_track.agenda_main"/>
            </div>
            <!-- Drag/Drop Area -->
            <div class="oe_structure oe_empty" id="oe_structure_website_event_track_agenda_2" t-att-data-editor-message="editor_message"/>
        </t>
    </t>
</template>

<!-- ============================================================ -->
<!-- TOPBAR: BASE NAVIGATION -->
<!-- ============================================================ -->

<!-- Main topbar -->
<template id="agenda_topbar" name="Agenda Tools">
    <div class="o_weagenda_topbar_filters o_wevent_index_topbar_filters d-print-none d-flex flex-wrap align-items-center justify-content-end gap-2 w-100 mt-1 mt-lg-3 mb-3">
        <h3 class="my-0 me-auto pe-sm-4">Agenda</h3>
        <label class="invisible text-muted me-2" id="search_summary"><span id="search_number" class="me-1">0</span>Results</label>
        <div class="o_wevent_search d-flex w-100 w-lg-auto">
            <div class="input-group">
                <input type="text" class="form-control search-query oe_search_box border-0 text-bg-light" placeholder="Filter Tracks" id="event_track_search"/>
                <div class="btn oe_search_button btn-light">
                    <i class="fa fa-filter" role="img"/>
                </div>
            </div>
        </div>
    </div>
</template>

<!-- Option: Tracks display: optional favorites -->
<template id="agenda_topbar_wishlist"
    inherit_id="website_event_track.agenda_topbar"
    name="Allow Wishlists"
    active="True">
    <xpath expr="//div[hasclass('o_weagenda_topbar_filters')]" position="inside">
    </xpath>
</template>

<!-- ============================================================ -->
<!-- CONTENT: MAIN TEMPLATES -->
<!-- ============================================================ -->

<!-- Agenda Main Display -->
<template id="agenda_main" name="Tracks: Main Display">
    <t t-set="event_has_complex_day" t-value="false"/>
    <t t-foreach="days" t-as="day">
        <t t-if="len(locations_by_days[day]) &gt; 5" t-set="event_has_complex_day" t-value="true"/>
    </t>

    <section t-foreach="days" t-as="day">
        <!-- DAY HEADER -->
        <div t-attf-class="{{event_has_complex_day and 'container-fluid' or 'container-xl'}}">
            <hr t-attf-class="mb-2 {{not day_first and 'mt-5'}}"/>
            <div class="o_we_track_day_header d-flex justify-content-between align-items-start pt-2 mb-3 w-100">
                <time class="d-flex flex-grow-1 align-items-end gap-2" t-att-datetime="day">
                    <span class="o_wevent_day_header_number lh-1" t-out="day" t-options="{'widget': 'date', 'format': 'dd'}"/>
                    <div class="small">
                        <div class="lh-1 fw-bold" t-out="day" t-options="{'widget': 'date', 'format': 'MMMM YYYY'}"/>
                        <span class="lh-1" t-out="day" t-options="{'widget': 'date', 'format': 'EEEE '}"/>
                        <span class="d-none d-md-inline small lh-1 text-muted">(timezone: <t t-out="event.date_tz"/>)</span>
                    </div>
                </time>
                <small><b t-out="tracks_by_days[day]"/> <span class="text-muted"> tracks</span></small>
            </div>
        </div>

        <t t-set="locations" t-value="locations_by_days[day]"/>
        <!-- Day Agenda -->
        <div t-attf-class="{{event_has_complex_day and 'container-fluid' or 'container-xl'}} pe-0">
            <div class="o_we_online_agenda mb-3">
                <table id="table_search" class="table table-sm border-0 h-100">
                    <!--Header-->
                    <tr>
                        <th class="border-0 position-sticky"/>
                        <t t-foreach="locations" t-as="location">
                            <th t-if="location" class="o_we_online_agenda_header_location active text-center">
                                <t t-set="unknown_location_default_text">Unknown</t>
                                <span t-out="location and location.name or unknown_location_default_text"/>
                            </th>
                        </t>
                    </tr>

                    <!-- Time Slots -->
                    <t t-set="used_cells" t-value="[]"/>
                    <t t-foreach="time_slots[day]" t-as="time_slot">
                        <t t-set="is_round_hour" t-value="time_slot == time_slot.replace(minute=0)"/>
                        <t t-set="is_half_hour" t-value="time_slot == time_slot.replace(minute=30)"/>

                        <tr t-att-class="'%s' % ('active' if is_round_hour else '')">
                            <td class="active bg-body">
                                <b t-if="is_round_hour" t-out="time_slots[day][time_slot]['formatted_time']"/>
                            </td>

                            <t t-foreach="locations" t-as="location">
                                <t t-set="tracks" t-value="time_slots[day][time_slot].get(location, {})"/>
                                <t t-if="tracks">
                                    <t t-foreach="tracks" t-as="track">
                                        <t t-set="_classes"
                                            t-value="'text-center %s %s %s' % (
                                                'o_color_%s' % (track.color or 0),
                                                'event_track h-100' if track else '',
                                                'o_location_size_%d' % len(locations),
                                            )"/>
                                        <t t-if="track.location_id and track.location_id == location">
                                            <td t-att-rowspan="tracks[track]['rowspan']"
                                                t-att-class="_classes">
                                                <t t-call="website_event_track.agenda_main_track"/>
                                            </td>
                                        </t>
                                        <t t-else="">
                                            <td t-att-colspan="len(locations)-1"
                                                t-att-rowspan="tracks[track]['rowspan']"
                                                t-att-class="_classes">
                                                <t t-call="website_event_track.agenda_main_track"/>
                                            </td>
                                        </t>
                                        <t t-set="used_cells" t-value="used_cells + tracks[track]['occupied_cells']"/>
                                    </t>
                                </t>
                                <t t-elif="location and (time_slot, location) not in used_cells">
                                    <td t-att-rowspan="1"
                                        t-att-class="'o_location_size_%s %s' % (len(locations),
                                            'o_we_agenda_time_slot_half' if is_half_hour else
                                            'o_we_agenda_time_slot_main' if is_round_hour else
                                            ''
                                        )"><div/></td>
                                </t>
                            </t>
                        </tr>
                    </t>
                </table>
            </div>
            <span class="o_we_online_agenda_overlay py-5 px-3 pe-none z-1"/>
        </div>
    </section>

    <div class="position-sticky bottom-0 w-100 overflow-x-scroll o_we_agenda_horizontal_scroller_container">
        <div class="o_we_agenda_horizontal_scroller"/>
    </div>
</template>

<template id="agenda_main_track" name="Track Agenda: Track">
    <div class="d-flex flex-column h-100" t-att-data-publish="track.website_published and 'on' or 'off'">
        <div class="o_we_agenda_card_content position-relative d-flex flex-column justify-content-center my-1">
            <div class="o_we_agenda_card_title mb-1">
                <t t-if="track.website_published or is_event_user">
                    <a t-att-href="'/event/%s/track/%s' % (slug(event), slug(track))" class="text-reset fw-bold stretched-link">
                        <t t-out="track.name"/>
                    </a>
                </t>
                <t t-else="">
                    <span class="fw-bold">
                        <t t-out="track.name"/>
                    </span>
                </t>
            </div>
            <div class="opacity-75 text-center" t-if="track.partner_tag_line">
                <small t-out="track.partner_tag_line"/>
            </div>
        </div>
        <div class="o_we_agenda_card_filter_badges d-flex justify-content-center align-items-end gap-1 pb-1">
            <div>
                <t t-foreach="track.tag_ids" t-as="tag">
                    <span t-if="tag.color" t-att-title="tag.name"
                        t-attf-class="me-1 mt-1 badge o_badge_clickable #{'o_color_'+str(tag.color)}" t-out="tag.name"
                        onclick="
                        var value = this.title;
                        var target = $('#event_track_search');
                        if (target.val() == value) { target.val(''); } else { target.val(value); }
                        target.trigger('input');
                        "
                    />
                </t>
            </div>
            <div class="d-flex justify-content-end flex-wrap-reverse ms-auto align-items-center gap-1 o_weagenda_track_badges">
                <small t-if="track.is_track_live and not track.is_track_done and track.website_published"
                    class="mx-1 badge text-bg-danger rounded-1">Live
                </small>
                <small t-if="not track.website_published and is_event_user"
                    title="Unpublished"
                    class="ms-1 mt-1 mt-md-0 badge text-bg-danger o_wevent_online_badge_unpublished">Unpublished</small>
                <span t-if="option_track_wishlist">
                    <t t-call="website_event_track.track_widget_reminder">
                        <t t-set="reminder_light" t-value="False"/>
                        <t t-set="reminder_small" t-value="True"/>
                        <t t-set="light_theme" t-value="False"/>
                    </t>
                </span>
            </div>
        </div>
    </div>
</template>

</odoo>

```

## File: views\event_track_templates_list.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="tracks_session" name="Event Tracks">
    <t t-call="website_event.layout">
        <div class="o_wevent_online o_wesession_index container">
            <!-- Options -->
            <t t-set="option_track_wishlist" t-value="not event.is_done and is_view_active('website_event_track.session_topbar_wishlist')"/>
            <!-- Topbar -->
            <t t-call="website_event_track.session_topbar"/>
            <!-- Drag/Drop Area -->
            <div id="oe_structure_wesession_index_1" class="oe_structure"/>
            <!-- Content -->
            <div class="o_wesession_container">
                <t t-call="website_event_track.tracks_search"/>
                <t t-call="website_event_track.tracks_main"/>
            </div>
            <!-- Drag/Drop Area -->
            <div id="oe_structure_wesession_index_2" class="oe_structure mb-5"/>
        </div>
    </t>
</template>

<!-- ============================================================ -->
<!-- TOPBAR: BASE NAVIGATION -->
<!-- ============================================================ -->

<!-- Main topbar -->
<template id="session_topbar" name="Tracks Tools">
    <div>
        <div class="o_wesession_topbar_filters o_wevent_index_topbar_filters d-print-none d-flex flex-wrap align-items-center justify-content-end gap-2 w-100 mt-1 mt-lg-3 mb-3">
            <h3 class="my-0 me-auto pe-sm-4">Talks</h3>
            <!-- Optional wishlist filter -->
            <div t-if="option_track_wishlist" class="dropdown d-none d-lg-block">
                <a href="#" role="button" class="btn btn-light dropdown-toggle" data-bs-toggle="dropdown">
                    Favorites
                </a>
                <div class="dropdown-menu">
                    <a t-att-href="'/event/%s/track?%s' % (
                            slug(event),
                            keep_query('*', search_wishlist='')
                        )"
                        class="dropdown-item d-flex align-items-center justify-content-between">
                        All Talks
                    </a>
                    <a t-att-href="'/event/%s/track?%s' % (
                            slug(event),
                            keep_query('*', search_wishlist='1')
                        )"
                        t-attf-class="dropdown-item d-flex align-items-center justify-content-between #{'active' if search_wishlist else ''}">
                        Favorites
                    </a>
                </div>
            </div>
            <div class="o_wevent_search d-flex w-100 w-lg-auto">
                <t t-set="event_search_placeholder">Search a talk ...</t>
                <t t-call="website_event.events_search_box">
                    <t t-set="_searches" t-value="searches"/>
                    <t t-set="action" t-value="'/event/%s/track' % (slug(event))"/>
                    <t t-set="_placeholder" t-value="event_search_placeholder"/>
                </t>
                <button class="btn btn-light position-relative ms-2 d-lg-none"
                    data-bs-toggle="offcanvas"
                    data-bs-target="#o_wevent_talk_offcanvas">
                    <i class="fa fa-sliders"/>
                </button>
            </div>
        </div>
        <!-- Off canvas filters on mobile-->
        <div id="o_wevent_talk_offcanvas" class="o_website_offcanvas offcanvas offcanvas-end d-lg-none p-0 overflow-visible mw-75">
            <div class="offcanvas-header">
                <h5 class="offcanvas-title">Filters</h5>
                <button type="button" class="btn-close" data-bs-dismiss="offcanvas" aria-label="Close"/>
            </div>
            <div class="offcanvas-body p-0">
                <div class="accordion accordion-flush">
                    <div t-if="option_track_wishlist" class="accordion-item">
                        <h2 class="accordion-header">
                            <button class="accordion-button collapsed"
                                type="button"
                                data-bs-toggle="collapse"
                                data-bs-target=".o_wevent_offcanvas_fav"
                                aria-expanded="false"
                                aria-controls="o_wevent_offcanvas_fav">
                                Favorites
                            </button>
                        </h2>
                        <div class="o_wevent_offcanvas_fav accordion-collapse collapse">
                            <div class="accordion-body pt-0">
                                <ul class="list-group list-group-flush">
                                    <!-- Optional wishlist filter -->
                                    <li class="list-group-item border-0 px-0">
                                        <a t-att-href="'/event/%s/track?%s' % (
                                                slug(event),
                                                keep_query('*', search_wishlist='')
                                            )"
                                            class="d-flex align-items-center justify-content-between text-reset">
                                            All Talks
                                        </a>
                                    </li>
                                    <li class="list-group-item border-0 px-0">
                                        <a t-att-href="'/event/%s/track?%s' % (
                                                slug(event),
                                                keep_query('*', search_wishlist='1')
                                            )"
                                            t-attf-class="d-flex align-items-center justify-content-between text-reset #{'active' if search_wishlist else ''}">
                                            Favorites
                                        </a>
                                    </li>
                                </ul>
                            </div>
                        </div>
                    </div>
                    <t t-foreach="tag_categories" t-as="tag_category">
                    <div class="accordion-item">
                        <h2 class="accordion-header">
                            <button class="accordion-button collapsed"
                                type="button"
                                data-bs-toggle="collapse"
                                t-att-data-bs-target="'.o_wevent_offcanvas_tag_%s' % tag_category.name"
                                aria-expanded="false">
                                <t t-out="tag_category.name"/>
                            </button>
                        </h2>
                        <div t-attf-class="o_wevent_offcanvas_tag_#{tag_category.name} accordion-collapse collapse">
                            <div class="accordion-body pt-0">
                                <ul class="list-group list-group-flush">
                                    <t t-if="tag_category.tag_ids and any(tag.color for tag in tag_category.tag_ids)" t-foreach="tag_category.tag_ids" t-as="tag">
                                        <li class="list-group-item border-0 px-0">
                                            <span t-if="tag.color"
                                                t-att-data-post="'/event/%s/track?%s' % (
                                                    slug(event),
                                                    keep_query('*', tags=str((search_tags - tag).ids if tag in search_tags else (tag | search_tags).ids), prevent_redirect=True)
                                                )"
                                                t-attf-class="post_link cursot-pointer d-flex align-items-center justify-content-between text-reset #{'active' if tag in search_tags else ''}">
                                                <t t-out="tag.name"/>
                                            </span>
                                        </li>
                                    </t>
                                </ul>
                            </div>
                        </div>
                    </div>
                    </t>
                </div>
            </div>
        </div>
    </div>
</template>

<!-- Option: Topbar: optional tags filters -->
<template id="session_topbar_tag"
    inherit_id="website_event_track.session_topbar"
    name="Filter by Tags"
    active="True">
    <xpath expr="//div[hasclass('o_wevent_search')]" position="before">
        <t t-foreach="tag_categories" t-as="tag_category">
            <div t-if="tag_category.tag_ids and any(tag.color for tag in tag_category.tag_ids)" class="dropdown d-none d-lg-block my-1">
                <a href="#" role="button" class="btn btn-light dropdown-toggle" data-bs-toggle="dropdown">
                    <t t-out="tag_category.name"/>
                </a>
                <div class="dropdown-menu">
                    <t t-foreach="tag_category.tag_ids" t-as="tag">
                        <span t-att-data-post="'/event/%s/track?%s' % (
                                slug(event),
                                keep_query('*', tags=str((search_tags - tag).ids if tag in search_tags else (tag | search_tags).ids), prevent_redirect=True)
                            )"
                            t-if="tag.color"
                            t-attf-class="post_link cursor-pointer dropdown-item d-flex align-items-center justify-content-between #{'active' if tag in search_tags else ''}">
                            <t t-out="tag.name"/>
                        </span>
                    </t>
                </div>
            </div>
        </t>
    </xpath>
</template>

<!-- Option: Tracks display: optional wishlist -->
<template id="session_topbar_wishlist"
    inherit_id="website_event_track.session_topbar"
    name="Allow Wishlists"
    active="True">
    <xpath expr="//div[hasclass('o_wesession_topbar_filters')]" position="inside">
    </xpath>
</template>

<!-- ============================================================ -->
<!-- CONTENT: MAIN TEMPLATES -->
<!-- ============================================================ -->

<!-- Tracks Main Display -->
<template id="tracks_main" name="Tracks: Main Display">
    <!-- No tracks -->
    <div t-if="not tracks" class="col-12 text-center">
        <div t-call="website_event.event_empty_events_svg" class="my-4"/>
        <h4>No track found.</h4>
        <p t-if="search_key">We could not find any track matching your search for: <strong t-out="search_key"/>.</p>
        <p t-else="">We could not find any track at this moment.</p>
        <div class="o_not_editable my-3" groups="event.group_event_user">
            <a class="o_wevent_cta btn" target="_blank" t-att-href="'/odoo/%s/action-website_event_track.action_event_track_from_event' % event.id">
                <span class="fa fa-plus me-1"/> Schedule Tracks
            </a>
        </div>
    </div>
    <!-- Cards -->
    <div class="col-12" t-call="website_event_track.tracks_display_cards"/>
    <!-- List -->
    <div class="col-12" t-call="website_event_track.tracks_display_list"/>
</template>

<!-- Tracks: Cards-based display -->
<template id="tracks_display_cards" name="Tracks: Cards Display">
    <div class="row mb-3" t-if="tracks_live">
        <div class="col-12">
            <h4>Live Now</h4>
        </div>
        <t t-call="website_event_track.track_cards_section">
            <t t-set="tracks" t-value="tracks_live"/>
        </t>
    </div>
    <div class="row mb-3" t-if="tracks_soon">
        <div class="col-12">
            <h4>Coming soon ...</h4>
        </div>
        <t t-call="website_event_track.track_cards_section">
            <t t-set="tracks" t-value="tracks_soon"/>
        </t>
    </div>
</template>

<!-- Tracks: List-based display -->
<template id="tracks_display_list" name="Tracks: List Display">
    <div t-if="tracks" class="pt24">
        <h5>Book your talks</h5>
        <p>Plan your experience by adding your favorites talks to your wishlist</p>
    </div>
    <div t-if="tracks" class="o_wesession_list mb-3">
        <ul class="list-unstyled">
            <li class="border-bottom" t-foreach="tracks_by_day" t-as="tracks_info">
                <t t-set="tracks_date" t-value="tracks_info['date']"/>
                <t t-set="tracks_header_name" t-value="tracks_info['name']"/>
                <t t-set="tracks" t-value="tracks_info['tracks']"/>
                <!-- DAY HEADER -->
                <div class="o_we_track_day_header d-flex align-items-center position-relative py-3">
                   <time class="d-flex flex-grow-1 align-items-end gap-2" t-if="tracks_date" t-att-datetime="tracks_date">
                        <span class="o_wevent_day_header_number lh-1" t-out="tracks_date" t-options="{'widget': 'date', 'format': 'dd'}"/>
                        <div class="small">
                            <div class="lh-1 fw-bold" t-out="tracks_date" t-options="{'widget': 'date', 'format': 'MMMM YYYY'}"/>
                            <span class="lh-1" t-out="tracks_date" t-options="{'widget': 'date', 'format': 'EEEE '}"/>
                            <span class="d-none d-md-inline small lh-1 text-muted">(timezone: <t t-out="event.date_tz"/>)</span>
                        </div>
                    </time>
                    <div class="d-flex flex-grow-1" t-elif="tracks_header_name">
                        <span class="h4 m-0"
                            t-out="tracks_header_name"/>
                    </div>
                    <a t-attf-class="o_wevent_collapse_link my-auto align-self-start stretched-link text-reset {{ 'collapsed' if tracks_info['default_collapsed'] else '' }}"
                        t-attf-href="#collapse_session_list_{{ tracks_info_index }}"
                        t-attf-aria-controls="collapse_session_list_{{ tracks_info_index }}"
                        t-att-aria-expanded="'false' if tracks_info['default_collapsed'] else 'true'"
                        data-bs-toggle="collapse">
                        <i class="oi oi-chevron-down"/>
                    </a>
                    <hr class="mt-2 mb-2"/>
                </div>
                <!-- DAY TRACKS LIST -->
                <div t-attf-class="collapse {{ '' if tracks_info['default_collapsed'] else 'show' }}"
                    t-attf-id="collapse_session_list_{{ tracks_info_index }}">
                    <div t-foreach="tracks" t-as="track"
                        t-att-class="'o_wesession_list_item p-3 o_color_%d' % (track.color)">
                        <div class="row g-0">
                            <div class="col-12 d-flex justify-content-between">
                                <!-- Hour: Live > Remaining > Hour: desktop only -->
                                <div class="d-flex align-items-center gap-1 flex-wrap">
                                    <div t-if="tracks_date and today_tz &lt;= tracks_date"
                                        >
                                        <span t-if="track.is_track_live and not track.is_track_done"
                                            class="alert alert-danger px-2 py-1">Live</span>
                                        <span class="fw-bold" t-elif="not track.is_track_done and track.is_track_soon">
                                            <span t-out="track.track_start_remaining"
                                                t-options="{'widget': 'duration', 'digital': False, 'format': 'narrow',
                                                            'add_direction': True, 'unit': 'second', 'round': 'minute'}"/>
                                        </span>
                                        <span class="fw-bold" t-elif="not track.is_track_done and not track.is_track_soon"
                                            t-out="track.date"
                                            t-options="{'widget': 'datetime', 'time_only': True, 'format': 'short', 'tz_name': event.date_tz}"/>
                                        <span t-else="" class="alert alert-dark px-2 py-1 small">Finished</span>
                                    </div>
                                    <span t-if="tracks_date and today_tz &lt;= tracks_date and track.location_id">-</span>
                                    <div t-if="track.location_id">
                                        <i class="fa fa-map-marker" role="img"/> <span t-field="track.location_id"/>
                                    </div>
                                </div>
                                <!-- reminder -->
                                <div t-if="not event.is_done and (not track.date or today_tz &lt;= tracks_date) and option_track_wishlist"
                                    class="d-none d-md-block py-1">
                                    <t t-call="website_event_track.track_widget_reminder">
                                        <t t-set="reminder_small" t-value="False"/>
                                        <t t-set="reminder_light" t-value="False"/>
                                    </t>
                                </div>
                            </div>
                            <!-- Main column: name, speaker -->
                            <div class="col-12">
                                <span class="fw-bold mb0">
                                    <a t-if="track.website_published or is_event_user"
                                        class="text-reset me-2"
                                        t-att-href="track.website_url">
                                        <span t-field="track.name"/>
                                    </a>
                                    <t t-else="">
                                        <span class="text-reset me-2" t-field="track.name"/>
                                    </t>
                                </span>
                                <span t-if="not track.website_published and is_event_user" class="alert alert-danger px-2 py-1 align-top small">
                                    Unpublished
                                </span>
                            </div>

                            <div class="col-12 d-flex justify-content-between align-items-end">
                                <div class="d-flex align-items-center w-100 gap-2 text-muted">
                                    <span class="text-muted" t-out="track.partner_tag_line"/>
                                    <t t-if="tracks_date and today_tz &lt;= tracks_date">
                                        <t t-if="track.duration and not track.is_track_done and not track.is_track_done">
                                            <span class="d-none d-md-block">&amp;bull;</span>
                                            <span class="d-none d-md-block"
                                                t-out="track.duration"
                                                t-options="{'widget': 'duration', 'digital': False, 'format': 'short', 'unit': 'hour', 'round': 'minute'}"/>
                                        </t>
                                    </t>
                                </div>
                                <!-- Aside column: tags -->
                                <div class="d-none d-md-flex justify-content-end align-items-center w-auto gap-2">
                                    <!-- Tags: desktop only -->
                                    <t t-foreach="track.tag_ids" t-as="tag">
                                        <t t-if="tag.color" t-call="website_event_track.track_tag_badge_link"/>
                                    </t>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </li>
        </ul>
    </div>
</template>

<!-- ============================================================ -->
<!-- TOOL TEMPLATES -->
<!-- ============================================================ -->

<template id="track_cards_section" name="Track Cards">
    <div t-foreach="tracks" t-as="track" class="col-md-6 col-lg-4 col-xl-3 mb-4">
        <t t-if="track.website_published or is_event_user">
            <a t-att-href="'/event/%s/track/%s' % (slug(track.event_id), slug(track))" class="text-decoration-none text-reset">
                <t t-call="website_event_track.track_card"/>
            </a>
        </t>
        <t t-else="">
            <t t-call="website_event_track.track_card"/>
        </t>
    </div>
</template>

<template id="track_card" name="Track Card">
    <article t-att-class="'card o_wesession_track_card h-100 %s' % ('o_wesession_track_card_unpublished' if (not track.website_published and is_event_user) else '')">
        <div class="h-100 row g-0">
            <header class="overflow-hidden bg-secondary col-12">
                <small t-if="not track.website_published and is_event_user" class="o_wesession_track_card_header_badge bg-danger">
                    <i class="fa fa-ban me-2"/>Unpublished
                </small>

                <div t-if="track.website_image_url" class="card-img-top"
                    t-attf-style="padding-top: 50%; background-image: url(#{track.website_image_url}); background-size: cover; background-position:center">
                    <span t-if="option_track_wishlist and not track.is_track_live" class="position-absolute h3 mt-2 me-2 text-white" style="right: 0; top: 0;">
                        <t t-call="website_event_track.track_widget_reminder">
                            <t t-set="reminder_light" t-value="True"/>
                        </t>
                    </span>
                </div>
                <div t-else="" class="o_wesession_gradient card-img-top position-relative"
                    style="padding-top: 50%;">
                    <span t-if="option_track_wishlist" class="position-absolute h3 mt-2 me-2" style="right: 0; top: 0;">
                        <t t-call="website_event_track.track_widget_reminder">
                            <t t-set="reminder_light" t-value="True"/>
                        </t>
                    </span>
                    <i class="fa fa-glass fa-2x mx-2 mb-3 position-absolute text-white-75" style="right:0; bottom: 0;"/>
                </div>
            </header>
            <div class="col-12">
                <main class="card-body">
                    <!-- Tags> -->
                    <div class="mb-2 d-flex flex-wrap gap-1">
                        <t t-foreach="track.tag_ids" t-as="tag">
                            <t t-if="tag.color" t-call="website_event_track.track_tag_badge_info"/>
                        </t>
                    </div>
                    <!-- Title -->
                    <h5 class="card-title mb-0 text-truncate">
                        <span t-field="track.name"/>
                    </h5>
                </main>
            </div>
            <!-- Footer -->
            <footer class="small align-self-end w-100 p-3">
                <div class="d-flex justify-content-between align-items-center">
                    <!-- Speaker -->
                    <span class="opacity-75 text-truncate" t-field="track.partner_name"/>
                    <!-- Starts -->
                    <span class="opacity-75 ms-auto">
                        <t t-if="track.track_start_remaining &gt;= 60">In
                            <span class="ms-auto" t-field="track.track_start_remaining"
                                t-options="{'widget': 'duration', 'digital': False, 'format': 'short', 'unit': 'second', 'round': 'minute'}"/>
                        </t>
                        <t t-elif="track.track_start_relative &gt;= 60">
                            <span class="ms-auto" t-field="track.track_start_relative"
                                t-options="{'widget': 'duration', 'digital': False, 'format': 'short', 'unit': 'second', 'round': 'minute'}"/>
                            ago
                        </t>
                        <t t-else="">
                            Starting now!
                        </t>
                    </span>
                </div>
            </footer>
        </div>
    </article>
</template>

<!-- Searched tags -->
<template id="tracks_search" name="Tracks: search tags">
    <div t-if="search_wishlist or search_tags" class="d-flex flex-wrap align-items-center my-3 gap-2">
        <span t-if="search_wishlist"
            class="align-items-baseline border d-inline-flex rounded">
            <i class="fa fa-bell mx-2 text-muted"/> Favorite Talks
            <a t-att-href="'/event/%s/track?%s' % (slug(event), keep_query('*', search_wishlist=''))"
                class="btn border-0 py-1">
                &#215;
            </a>
        </span>

        <t t-foreach="search_tags" t-as="tag">
            <span class="align-items-baseline border d-inline-flex rounded">
                <i class="fa fa-tag mx-2 text-muted"/>
                <t t-out="tag.display_name"/>
                <span t-att-data-post="'/event/%s/track?%s' % (slug(event), keep_query('*', tags=str((search_tags - tag).ids), prevent_redirect=True))"
                    class="post_link cursor-pointer btn border-0 py-1">
                    &#215;
                </span>
            </span>
        </t>
    </div>
</template>

<!-- ============================================================ -->
<!-- MISC TOOLS -->
<!-- ============================================================ -->

<template id="track_tag_badge_link" name="Track: Tag Badge Link">
    <span t-if="search_tags"
        t-att-data-post="'/event/%s/track?%s'% (
            slug(event),
            keep_query('*', tags=str((search_tags - tag).ids if tag in search_tags else (tag | search_tags).ids), prevent_redirect=True)
        )"
        t-att-class="'post_link cursor-pointer badge rounded-pill text-decoration-none %s' % ('text-bg-primary opacity-75-hover' if tag in search_tags else 'o_badge o_badge_clickable o_color_0')"
        t-out="tag.name"/>
    <span t-else=""
        t-att-data-post="'/event/%s/track?%s'% (
            slug(event),
            keep_query('*', tags=str(tag.ids), prevent_redirect=True)
        )"
        t-att-class="'post_link cursor-pointer badge rounded-pill text-decoration-none o_badge o_badge_clickable o_color_%s' % (tag.color)"
        t-out="tag.name"/>
</template>

<template id="track_tag_badge_info" name="Track: Tag Badge Info">
    <span t-if="search_tags"
        t-att-class="'badge %s' % ('text-bg-primary' if tag in search_tags else 'o_color_0')"
        t-out="tag.name"/>
    <span t-else=""
        t-att-class="'badge rounded-pill o_color_%s' % (tag.color)"
        t-out="tag.name"/>
</template>

</odoo>

```

## File: views\event_track_templates_page.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="event_track_main" name="Event Exhibitor">
    <t t-call="website_event.layout">
        <t t-set="navbar__back_url" t-value="'/event/%s/track' % (slug(event))"/>
        <t t-set="navbar__back_title">Back to All Talks</t>
        <t t-set="navbar__back_text">All Talks</t>

        <div class="o_wevent_online o_wesession_index">
            <!-- Options -->
            <t t-set="option_widescreen" t-value="option_widescreen or False"/>
            <t t-set="option_track_wishlist" t-value="not event.is_done and is_view_active('website_event_track.session_topbar_wishlist')"/>
            <!-- Drag/Drop Area -->
            <div id="oe_structure_wesession_track_index_1" class="oe_structure"/>
            <!-- Content -->
            <div t-att-class="'o_wevent_online_page_container py-3 %s' % ('container' if not option_widescreen else 'container-fluid')">
                <div t-att-class="'row mb-5 mx-0 %s' % ('justify-content-center' if not tracks_other else '')">
                    <t t-if="tracks_other">
                        <t t-call="website_event_track.event_track_aside"/>
                    </t>
                    <t t-call="website_event_track.event_track_content"/>
                </div>
            </div>
            <!-- Drag/Drop Area -->
            <div id="oe_structure_wesession_track_index_2" class="oe_structure"/>
        </div>
    </t>
</template>

<!-- ============================================================ -->
<!-- CONTENT: MAIN TEMPLATES -->
<!-- ============================================================ -->

<template id="event_track_content_partner_info">
    <div class="mb-1 align-items-baseline text-break" t-if="track.partner_function">
        <i class="fa fa-briefcase me-2"/><span t-out="track.partner_function"/>
        <t t-if="track.partner_company_name">
            <span> at </span><span t-out="track.partner_company_name"/>
        </t>
    </div>
    <div class="mb-1 align-items-baseline text-break" t-if="track.partner_id.website">
        <i class="fa fa-globe me-2"/><a t-att-href="track.partner_id.website"><span t-field="track.partner_id.website"/></a>
    </div>
    <div class="mb-1 align-items-baseline text-break" t-if="track.partner_email">
        <i class="fa fa-envelope me-2"/><a t-att-mailto="track.partner_email"><span t-field="track.partner_email"/></a>
    </div>
    <div class="mb-1 align-items-baseline text-break" t-if="track.partner_phone">
        <i class="fa fa-phone me-2"/><span t-field="track.partner_phone"/>
    </div>
</template>

<template id="event_track_content" name="Track: Main Description">
    <div name="o_wesession_track_main" t-attf-class="o_wesession_track_main col-12 p-0 ps-md-3 ps-lg-4 {{'col-md-9 col-xl-10' if option_widescreen else 'col-md-8 col-xl-9 ps-xl-5'}}">
        <!-- LIVE INFORMATIONS -->
        <div t-if="not track.event_id.is_ongoing and track.event_id.start_remaining" class="pt-3">
            <div class="mx-3 alert alert-warning">
                Event <span t-out="track.event_id.name" class="fw-bold"/>
                <span t-if="track.event_id.start_today">
                    starts in
                    <span t-if="track.event_id.start_remaining &gt;= 1" t-out="track.event_id.start_remaining"
                        t-options="{'widget': 'duration', 'digital': False, 'unit': 'minute', 'round': 'minute'}"/>
                    <t t-else="">
                        a few seconds
                    </t>.
                </span>
                <span t-else="">
                    starts on
                    <span t-field="track.event_id.with_context(tz=track.event_id.date_tz).date_begin"
                        t-options="{'format': 'medium', 'tz_name': track.event_id.date_tz, 'hide_seconds': 'True'}"/>
                    (<span t-out="track.event_id.date_tz"/>)
                </span>
            </div>
        </div>
        <div t-elif="track.track_start_remaining">
            <t t-call="website_event.display_timer_alert_widget">
                <t t-set="time_to_live" t-value="track.track_start_remaining"/>
            </t>
        </div>
        <!-- TRACK DESCRIPTION -->
        <div class="o_wesession_track_main_description overflow-auto">
            <div class="pb-3 border-bottom">
                <div class="d-flex flex-wrap justify-content-between align-items-center gap-2 mb-2">
                    <span class="h4 mb-0" t-out="track.name"/>
                    <div class="d-flex flex-wrap gap-1">
                        <t t-foreach="track.tag_ids" t-as="tag">
                            <span t-if="tag.color"
                                t-att-class="'badge o_color_%s' % (tag.color)"
                                t-out="tag.name"/>
                        </t>
                    </div>
                </div>
                <div class="d-flex flex-wrap justify-content-between align-items-center">
                    <div class="text-muted">
                        <t t-if="track.location_id">
                            <strong>Location:</strong> <span t-field="track.location_id"/>
                            <span> - </span>
                        </t>
                        <t t-if="track.date">
                            <span t-field="track.date"
                                t-options='{"hide_seconds":"True", "format": "short", "tz_name": event.date_tz}'/>
                            -
                            <span t-field="track.date_end"
                                t-options='{"hide_seconds":"True", "format": "short", "tz_name": event.date_tz}'/>
                            <t t-if="event.date_tz">
                                (<span t-out="event.date_tz"/>)
                            </t>
                        </t>
                        <t t-if="track.duration">
                            (<span t-field="track.duration"
                                t-options='{"widget": "duration", "unit": "hour", "round": "minute"}'/>)
                        </t>
                    </div>
                    <div class="o_we_track_action_buttons d-flex justify-content-md-end align-items-center flex-wrap">
                        <div class="o_we_track_reminder_button my-1">
                            <div t-if="option_track_wishlist and not track.is_track_done"
                                t-call="website_event_track.track_widget_reminder">
                                <t t-set="reminder_small" t-value="False"/>
                                <t t-set="reminder_light" t-value="False"/>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- ABOUT AUTHOR -->
            <t t-set="track_fields" t-value="['image', 'partner_name', 'partner_function', 'partner_email', 'partner_phone']"/>
            <!-- Avoid rendering a section for track information if we don't have anything to display -->
            <t t-if="any(track[track_field] for track_field in track_fields) or track.partner_id.website or not is_html_empty(track.partner_biography)">
                <div>
                    <div class="mt-2 mb-2 d-flex">
                        <div t-if="track.image" t-field="track.image"
                             t-options="{'widget': 'image', 'class': 'rounded-circle me-2', 'max_width': '96'}"/>
                        <div class="d-none d-md-flex flex-grow-1 flex-column ps-2 pe-0 pe-md-2">
                            <div t-if="track.partner_name" t-field="track.partner_name" class="mb-2 fw-bold"/>
                            <t t-call="website_event_track.event_track_content_partner_info"/>
                        </div>
                    </div>
                    <div class="d-flex d-md-none flex-grow-1 flex-column mt-3 mb-3 ps-2 pe-0 pe-md-2">
                        <div t-if="track.partner_name" t-field="track.partner_name" class="mb-2 fw-bold"/>
                        <t t-call="website_event_track.event_track_content_partner_info"/>
                    </div>
                    <div t-if="not is_html_empty(track.partner_biography)" t-field="track.partner_biography"
                         class="oe_no_empty"/>
                    <hr t-if="not is_html_empty(track.description)" class="mt-2 pb-1 mb-1"/>
                </div>
            </t>

            <!-- Description -->
            <div t-if="not is_html_empty(track.description)" t-field="track.description" class="my-2 oe_no_empty"/>
            <t t-elif="env.user.has_group('event.group_event_user')">
                <div t-field="track.description" class="my-2"
                     placeholder="e.g. &quot;This talk will be about...&quot;"/>
                <div class="alert alert-info mt-3 o_wesession_track_main_empty_descr_warning">
                    This track does not have a description.
                </div>
            </t>
        </div>
    </div>
</template>

<!-- ============================================================ -->
<!-- ASIDE: CONTROL PANEL -->
<!-- ============================================================ -->

<template id="event_track_aside" name="Track: Aside">
    <div name="o_wevent_online_page_aside" t-attf-class="o_wevent_online_page_aside o_wesession_track_aside col-12 ps-0 pe-0 mb-3 border {{'col-md-3 col-xl-2' if option_widescreen else 'col-md-4 col-xl-3'}}">
        <div class="o_wevent_online_page_aside_content">
            <div class="mx-2" t-if="track.date and track.website_cta">
                <t t-set="cta_coundown" t-value="bool(track.website_cta_start_remaining)"/>
                <div t-if="cta_coundown" class="text-center w-100 btn btn-primary d-none">
                    <t t-call="website_event.display_timer_widget">
                        <t t-set="pre_remaining_time" t-value="int(track.track_start_remaining)"/>
                        <!--<t t-set="pre_countdown_text" t-value="'Talk starts in'"/>-->
                        <t t-set="pre_countdown_display" t-value="bool(False)"/>
                        <t t-set="main_remaining_time" t-value="int(track.website_cta_start_remaining)"/>
                        <!--<t t-set="main_countdown_text" t-value="'Magic happens in'"/>-->
                        <t t-set="main_countdown_display" t-value="bool(False)"/>
                        <t t-set="display_class" t-value="'.o_event_cta_action'"/>
                    </t>
                </div>
                <div t-att-class="'o_event_cta_action %s' % ('d-none' if cta_coundown else '')">
                    <a t-att-href="track.website_cta_url"
                        target="_blank"
                        class="btn btn-primary w-100 my-3">
                        <span t-out="track.website_cta_title"/>
                    </a>
                </div>
            </div>
            <div class="position-relative d-flex align-items-center justify-content-between py-2 bg-light">
                <ul class="nav nav-tabs o_wesession_track_aside_nav d-flex border-0" role="tablist">
                    <li class="nav-item flex-grow-1">
                        <a href="#track_list" aria-controls="track_list" class="nav-link active" role="tab" data-bs-toggle="tab">
                            Talks
                        </a>
                    </li>
                </ul>
                <a href="#collapse_track_aside" data-bs-toggle="collapse" class="o_wevent_collapse_link stretched-link d-md-none px-3 text-decoration-none o_wevent_online_page_aside_collapse collapsed">
                    <i class="oi oi-chevron-down d-md-none"/>
                </a>
            </div>
            <div id="collapse_track_aside"
                class="tab-content collapse d-md-block o_wesession_track_aside_tabs">
                <div class="tab-pane fade show active" id="track_list" role="tabpanel">
                    <ul class="list-group list-group-flush">
                        <li t-foreach="tracks_other" t-as="track_other" class="list-group-item list-group-item-action">
                            <a t-if="is_event_user or track_other.is_published"
                                t-att-data-publish="track_other.website_published and 'on' or 'off'"
                                class="d-flex flex-column flex-wrap flex-md-nowrap text-decoration-none text-reset"
                                t-att-href="track_other.website_url">
                                <t t-call="website_event_track.event_track_aside_other_track"/>
                            </a>
                            <div t-else="" class="text-muted px-2 pt-2 pb-1">
                                <div t-att-data-publish="track_other.website_published and 'on' or 'off'">
                                    <t t-call="website_event_track.event_track_aside_other_track"/>
                                </div>
                            </div>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="event_track_aside_other_track">
    <span t-out="track_other.name" class="fw-bold"/>
    <div class="d-flex flex-column flex-lg-row flex-wrap justify-content-between align-items-start mt-1">
        <small class="opacity-75" t-out="track_other.partner_name"/>
        <div class="d-inline-block o_wesession_track_aside_info text-truncate">
            <small t-if="not track_other.website_published and user_event_manager"
                class="badge text-bg-danger">Unpublished</small>
            <small t-if="track_other.is_track_live and not track_other.is_track_done"
                class="badge text-bg-danger">Live</small>
            <small t-elif="track_other.is_track_done"
                class="badge text-bg-info">Done</small>
            <small t-elif="track_other.is_track_today and track_other.track_start_remaining"
                class="badge text-bg-info">
                <span t-out="track_other.track_start_remaining"
                    t-options="{'widget': 'duration', 'digital': False, 'add_direction': True,
                                'unit': 'second', 'round': 'minute', 'format': 'narrow'}"/>
            </small>
            <div t-elif="track_other.date" class="badge text-bg-info">
                <span t-out="track_other.date" t-options="{'widget': 'datetime', 'tz_name': track_other.event_id.date_tz, 'format': 'MMM. dd'}"/>
            </div>
        </div>
    </div>
</template>

</odoo>

```

## File: views\event_track_templates_proposal.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="event_track_proposal">
    <t t-call="website_event.layout">
        <t t-set="hide_submenu" t-value="True"/>
        <t t-set="editor_message">DROP BUILDING BLOCKS HERE TO MAKE THEM AVAILABLE ACROSS ALL PROPOSAL PAGES OF ALL EVENTS</t>
        <div class="oe_structure" id="oe_structure_website_event_track_proposal_1" t-att-data-editor-message="editor_message"/>
        <div class="container">
            <t t-call="website_event.navbar"/>

            <div class="row mb-3">
                <div t-att-class="event.website_track_proposal and 'col-lg-8 col-xl-9 pe-xxl-5' or 'col-12'">
                    <!-- Default Title (editable) -->
                    <div class="oe_structure">
                        <section class="pt24 pb24">
                            <h3 class="my-0">Call for Proposals</h3>
                        </section>
                    </div>

                    <!-- "Proposal are CLOSED" content -->
                    <div t-if="not event.website_track_proposal" class="oe_structure">
                        <section class="s_text_block pb40" data-snippet="s_text_block" data-name="Text">
                            <div class="container s_allow_columns">
                                <h4>Proposals are closed.</h4>
                                <p> We're sorry, this event does not accept proposals anymore </p>
                            </div>
                        </section>
                    </div>

                    <!-- "Proposal are ALLOWED" content -->
                    <div t-if="event.website_track_proposal" class="oe_structure">
                        <section>
                            <h4 class="mt-0">
                                Introduction
                            </h4>
                            <p class="lead">
                                We will accept a broad range of
                                presentations, from reports on academic and
                                commercial projects to tutorials and case
                                studies. As long as the presentation is
                                interesting and potentially useful to the
                                audience, it will be considered for
                                inclusion in the programme.
                            </p>
                        </section>
                    </div>

                    <section t-if="event.website_track_proposal" id="forms">
                        <form class="js_website_submit_form o_website_event_track_proposal_form row mt-3" t-att-data-event-id="event.id" enctype="multipart/form-data">
                            <input name="csrf_token" type="hidden" t-att-value="request.csrf_token()"/>
                            <div id="track_intro" class="col-12">
                                <hr/>
                                <h5 class="mt-0 mb-1">Talk Intro</h5>
                                <div class="form-text text-muted mb16">
                                    What is your talk about?
                                </div>
                                <div class="row mb-3">
                                    <label class="col-form-label col-sm-auto" for="track_name" style="width: 200px" >
                                        <span class="o_wetrack_proposal_label_content">Talk Title</span>
                                        <span>*</span>
                                    </label>
                                    <div class="col-sm">
                                        <input name="track_name" type="text" class="form-control form-field" required="True"/>
                                    </div>
                                </div>
                                <div class="row mb-3">
                                    <label class="col-form-label col-sm-auto" for="description" style="width: 200px" >
                                        <span class="o_wetrack_proposal_label_content">Talk Introduction</span>
                                        <span>*</span>
                                    </label>
                                    <div class="col-sm">
                                        <textarea name="description" class="form-control" rows="4" required="True"/>
                                    </div>
                                </div>
                                <div class="row mb-3">
                                    <label class="col-form-label col-sm-auto" for="tags" style="width: 200px">Categories</label>
                                    <div class="col-sm o_website_event_track_proposal_form_tags"/>
                                </div>
                            </div>

                            <div id="speaker_profile" class="col-12">
                                <hr/>
                                <h5 class="mt-0 mb-1">Speaker Profile</h5>
                                <div class="form-text text-muted mb16">
                                    Who will give this talk? We will show this to attendees to showcase your talk.
                                </div>
                                <div class="row mb-3">
                                    <label class="col-form-label col-sm-auto" for="partner_name" style="width: 200px">Name</label>
                                    <div class="col-sm"><input name="partner_name" type="text" class="form-control"/></div>
                                </div>
                                <div class="row mb-3">
                                    <label class="col-form-label col-sm-auto" for="partner_email" style="width: 200px">Email</label>
                                    <div class="col-sm"><input name="partner_email" type="email" class="form-control" /></div>
                                </div>
                                <div class="row mb-3">
                                    <label class="col-form-label col-sm-auto" for="partner_phone" style="width: 200px">Phone</label>
                                    <div class="col-sm"><input name="partner_phone" type="text" class="form-control"/></div>
                                </div>
                                <div class="row mb-3">
                                    <label class="col-form-label col-sm-auto" for="image" style="width: 200px">Picture</label><br/>
                                    <div class="col-sm"><input name="image" class="form-control" type="file" accept="image/*" style="width: 100%"/></div>
                                </div>
                                <div class="row mb-3">
                                    <label class="col-form-label col-sm-auto" for="partner_function" style="width: 200px">Job Title</label>
                                    <div class="col-sm"><input name="partner_function" type="text" class="form-control"/></div>
                                </div>
                                <div class="row mb-3">
                                    <label class="col-form-label col-sm-auto" for="partner_biography" style="width: 200px" >Biography</label>
                                    <div class="col-sm"><textarea name="partner_biography" rows="4" class="form-control"/></div>
                                </div>
                            </div>
                            <t t-call="website_event_track.event_track_proposal_contact_details"/>
                            <div class="o_form_buttons col-12 d-flex align-items-center pt16 pb48">
                                <button type="submit" class="btn btn-primary o_wetrack_proposal_submit_button">Submit Proposal</button>
                                <span class="o_wetrack_proposal_error_section text-danger d-none ml8">
                                    <i class="fa fa-close mr4" role="img" aria-label="Error" title="Error"/>
                                    <span class="o_wetrack_proposal_error_message"/>
                                </span>
                            </div>
                        </form>
                    </section>
                    <div class="oe_structure" id="oe_structure_website_event_track_proposal_2" t-att-data-editor-message="editor_message"/>
                </div> <!-- Close main column -->

                <!-- Sidebar: visible if proposals are allowed only -->
                <aside t-if="event.website_track_proposal" class="col-lg-4 col-xl-3 mt-3 mt-lg-0 pt-3">
                    <h5 class="o_page_header">Talks Types</h5>
                    <h6>Regular Talks</h6>
                    <p>
                        These are standard talks with slides, alocated in slots of 60 minutes.
                    </p>
                    <h6>Lightning Talks</h6>
                    <p>
                        These are 30 minutes talks on many different topics. Most topics are accepted in lightning talks.
                    </p>
                    <br/>
                    <h5 class="o_page_header">Submission Agreement</h5>
                    <p>
                    We require speakers to accept an agreement in which they commit to:
                    </p>
                    <ul>
                        <li>
                            Timely release of presentation material (slides),
                            for publishing on our website.
                        </li>
                        <li>
                            Allow video and audio recording of their
                            presentation, for publishing on our website.
                        </li>
                    </ul>
                </aside>
            </div>
        </div>
        <div class="oe_structure" id="oe_structure_website_event_track_proposal_3" t-att-data-editor-message="editor_message"/>
    </t>
</template>

<template id="event_track_proposal_contact_details">
    <div id="event_track_proposal_additional_contact_section" class="col-12">
        <div class="mb-3">
            <input name="add_contact_information" type= "checkbox" class="o_wetrack_add_contact_information_checkbox"/>
            <label class="fw-normal" for="add_contact_information">Contact me through a different email/phone</label>
        </div>
        <div class="o_wetrack_contact_information d-none">
            <hr/>
            <h5 class="mt-0 mb-1">Contact Information</h5>
            <div class="form-text text-muted mb16">
                How can our team get in touch with you?
            </div> 
            <div class="row mb-3">
                <label class="col-form-label col-sm-auto" style="width: 200px">
                    <span for="contact_name">Name</span>
                    <span>*</span>
                </label>
                <div class="col-sm"><input name="contact_name" type="text" class="form-control o_wetrack_contact_name_input"/></div>
            </div>
            <div class="row mb-3">
                <label class="col-form-label col-sm-auto" for="contact_email" style="width: 200px" >Email</label>
                <div class="col-sm"><input name="contact_email" type="email" class="form-control o_wetrack_contact_mean o_wetrack_contact_email_input"/></div>
            </div>
            <div class="row mb-3">
                <label class="col-form-label col-sm-auto" for="contact_phone" style="width: 200px" >Phone</label>
                <div class="col-sm"><input name="contact_phone" type="text" class="form-control o_wetrack_contact_mean o_wetrack_contact_phone_input"/></div>
            </div>
        </div> 
    </div>
</template>

</odoo>

```

## File: views\event_track_templates_reminder.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- Options
    * reminder_light: no text displayed
    * reminder_small: text displayed as small
    * light_theme: bell is white/gold if set; otherwse bell is gray / white
 -->
<template id="track_widget_reminder">
    <t t-set="_btn_style" t-value="'btn-link' if reminder_light else (track.is_reminder_on and '') or ''"/>
    <t t-set="_btn_size" t-value="'btn-sm' if reminder_small else ''"/>
    <div t-att-class="'o_wetrack_js_reminder %s %s' % (_btn_style, _btn_size)" role="button">
        <t t-if="track.is_reminder_on" t-set="title">Favorite On</t>
        <t t-else="track.is_reminder_on" t-set="title">Set Favorite</t>
        <i t-att-class="'fa fa-bell%s %s' % ('' if track.is_reminder_on else '-o', 'text-white' if reminder_light and not light_theme else 'text-dark')"
           t-att-data-track-id="track.id"
           t-att-title="title"
           t-att-data-is-reminder-light="reminder_light"
           t-att-data-reminder-on="bool(track.is_reminder_on)"></i>
    </div>
</template>

</odoo>

```

## File: views\event_track_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="website_visitor_action_from_track" model="ir.actions.act_window">
        <field name="name">Visitors Wishlist</field>
        <field name="res_model">website.visitor</field>
        <field name="view_mode">kanban,list,form,graph</field>
        <field name="domain">[('event_track_wishlisted_ids', 'in', [active_id])]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Wait for visitors to add this track to their list of favorites
            </p>
        </field>
    </record>

    <record id="event_track_view_form_quick_create" model="ir.ui.view">
        <field name="name">event.track.view.form.quick.create</field>
        <field name="model">event.track</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <form>
                <group>
                    <field name="event_id" invisible="context.get('default_event_id', False)"/>
                    <field name="name" placeholder="e.g. Inspiring Business Talk"/>
                    <field name="date"/>
                </group>
            </form>
        </field>
    </record>

    <record model="ir.ui.view" id="view_event_track_kanban">
        <field name="name">event.track.kanban</field>
        <field name="model">event.track</field>
        <field name="arch" type="xml">
            <kanban highlight_color="color" default_group_by="stage_id" quick_create_view="website_event_track.event_track_view_form_quick_create">
                <field name="website_url"/>
                <field name="legend_blocked"/>
                <field name="legend_normal"/>
                <field name="legend_done"/>
                <templates>
                    <progressbar field="kanban_state" colors='{"done": "success", "blocked": "danger"}'/>
                    <t t-name="menu" groups="base.group_user">
                        <a role="menuitem" t-att-href="record.website_url.value" class="dropdown-item">View Track</a>
                        <t t-if="widget.editable"><a role="menuitem" type="open" class="dropdown-item">Edit Track</a></t>
                        <t t-if="widget.deletable"><a role="menuitem" type="delete" class="dropdown-item">Delete</a></t>
                        <field name="color" widget="kanban_color_picker"/>
                    </t>
                    <t t-name="card">
                        <field name="name" class="fw-medium fs-5"/>
                        <div t-if="duration" class="d-flex"><field name="duration" widget="float_time"/>hours</div>
                        <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                        <footer class="pt-0">
                            <div class="d-flex align-items-center">
                                <field name="priority" widget="priority"/>
                                <field name="activity_ids" widget="kanban_activity" class="ms-1"/>
                            </div>
                            <div class="d-flex ms-auto align-items-center">
                                <field name="kanban_state" widget="state_selection" groups="base.group_user"/>
                                <div t-att-title="record.partner_id.value">
                                    <field name="partner_id" widget="image" options="{'preview_image': 'avatar_128'}" class="o_avatar rounded ms-1"/>
                                </div>
                            </div>
                        </footer>
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
                <field name="location_id" filters="1"/>
                <field name="event_id"/>
                <field name="partner_id" avatar_field="avatar_128"/>
                <field name="user_id" avatar_field="avatar_128"/>
            </calendar>
        </field>
    </record>

    <record model="ir.ui.view" id="view_event_track_search">
        <field name="name">event.track.search</field>
        <field name="model">event.track</field>
        <field name="arch" type="xml">
            <search string="Event Tracks">
                <field name="name"/>
                <field name="tag_ids"/>
                <field name="partner_id" string="Speaker" filter_domain="[
                    '|', '|', '|', '|',
                    ('partner_id', 'ilike', self), ('partner_name', 'ilike', self),
                    ('contact_email', 'ilike', self), ('partner_email', 'ilike', self), ('partner_company_name', 'ilike', self)]"/>
                <field name="location_id"/>
                <field name="event_id"/>
                <field name="stage_id"/>
                <filter string="My Tracks" name="my_tracks" domain="[('user_id', '=', uid)]"/>
                <separator/>
                <filter string="Published" name="filter_published" domain="[('website_published', '=', True)]"/>
                <separator/>
                <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction', '=', True)]" groups="mail.group_mail_notification_type_inbox"/>
                <separator/>
                <filter string="Always Wishlisted" name="filter_wishlisted_by_default" domain="[('wishlisted_by_default', '=', True)]"/>
                <separator/>
                <filter name="filter_date" date="date"/>
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
                    <filter string="Stage" name="stage" context="{'group_by': 'stage_id'}"/>
                    <filter string="Date" name="date" context="{'group_by': 'date'}"/>
                    <filter string="Event" name="event" context="{'group_by': 'event_id'}"/>
                    <filter string="Location" name="location" context="{'group_by': 'location_id'}"/>
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
                        <field name="wishlist_visitor_count" invisible="1"/>
                        <button name="%(website_event_track.website_visitor_action_from_track)d"
                            type="action" class="oe_stat_button" icon="fa-bell"
                            groups="event.group_event_user" invisible="wishlist_visitor_count == 0">
                            <field name="wishlist_visitor_count" string="Wishlisted By" widget="statinfo"/>
                        </button>
                        <field name="is_published" widget="website_redirect_button"/>
                    </div>
                    <field name="legend_blocked" invisible="1"/>
                    <field name="legend_normal" invisible="1"/>
                    <field name="legend_done" invisible="1"/>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <div class="d-flex gap-4 mb-3">
                        <div class="flex-grow-1">
                            <label for="name"/>
                            <h1>
                                <field name="name" placeholder="e.g. Inspiring Business Talk" class="w-sm-75"/>
                            </h1>
                        </div>
                        <field name="website_image" widget="image" class="oe_avatar"/>
                        <field name="kanban_state" widget="state_selection"/>
                    </div>
                    <group>
                        <group>
                            <field name="date"/>
                            <field name="location_id"/>
                            <field name="duration" widget="float_time"/>
                            <field name="active" invisible="1"/>
                            <field name="wishlisted_by_default"/>
                        </group>
                        <group>
                            <field name="company_id" invisible="1"/>
                            <field name="user_id" widget="many2one_avatar_user" domain="[('share', '=', False)]"/>
                            <field name="event_id"/>
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                            <field name="color" widget="color_picker"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Speaker" name="speaker">
                            <group string="Contact Details" class="display-flex justify-content-between">
                                <group>
                                    <field name="partner_id" context="{'default_phone': contact_phone, 'default_email': contact_email}"/>
                                    <field name="contact_email" readonly="partner_id"/>
                                    <field name="contact_phone" class="o_force_ltr" readonly="partner_id"/>
                                </group>
                            </group>
                            <group string="Speaker Bio" class="display-flex justify-content-between">
                                <group>
                                    <field name="partner_name"/>
                                    <field name="partner_email"/>
                                    <field name="partner_phone" class="o_force_ltr"/>
                                    <field name="partner_function"/>
                                    <field name="partner_company_name"/>
                                </group>
                                <group>
                                    <field name="image" nolabel="1" widget="image" class="oe_avatar"/>
                                </group>
                            </group>
                            <group>
                                <field name="partner_biography" string="Biography" placeholder='e.g. "John Doe was born in..."'/>
                            </group>
                        </page>
                        <page string="Description" name="description">
                            <field name="description" placeholder='e.g. "This talk will be about..."'/>
                        </page>
                        <page string="Interactivity" name="interactivity">
                            <group>
                                <group name="event_track_cta_group">
                                    <field name="website_cta"/>
                                    <field name="website_cta_title" placeholder="e.g. Get Yours Now!"
                                        invisible="not website_cta"
                                        required="website_cta"/>
                                    <field name="website_cta_url" placeholder="e.g. http://www.example.com"
                                        invisible="not website_cta"
                                        required="website_cta"/>
                                    <label for="website_cta_delay"
                                        invisible="not website_cta"/>
                                    <div invisible="not website_cta">
                                        <field name="website_cta_delay" class="oe_inline"
                                            required="website_cta"/> minutes after Track start
                                    </div>
                                </group>
                            </group>
                        </page>
                    </notebook>
                </sheet>
                <chatter reload_on_post="True"/>
            </form>
        </field>
    </record>

    <record model="ir.ui.view" id="view_event_track_tree">
        <field name="name">event.track.list</field>
        <field name="model">event.track</field>
        <field name="arch" type="xml">
            <list string="Event Track" multi_edit="1">
                <field name="name"/>
                <field name="active" column_invisible="True"/>
                <field name="partner_name"/>
                <field name="partner_id" optional="hide"/>
                <field name="partner_email"/>
                <field name="partner_phone"/>
                <field name="event_id" column_invisible="context.get('default_event_id')"/>
                <field name="wishlisted_by_default" optional="hide"/>
                <field name="wishlist_visitor_count" optional="hide"/>
                <field name="stage_id"/>
                <field name="color" widget="color_picker" optional="hide"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </list>
        </field>
    </record>

    <record model="ir.ui.view" id="view_event_track_graph">
        <field name="name">event.track.graph</field>
        <field name="model">event.track</field>
        <field name="arch" type="xml">
            <graph string="Tracks" sample="1">
                <field name="location_id"/>
                <field name="duration" operator="+"/>
                <field name="website_cta_delay" invisible="1"/>
                <field name="color" invisible="1"/>
            </graph>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_event_track">
        <field name="name">Event Tracks</field>
        <field name="res_model">event.track</field>
        <field name="view_mode">kanban,list,form,calendar,graph,activity</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
              Create a Track
            </p><p>
              Tracks define your event schedule. They can be talks, workshops or any similar activity.
            </p>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_event_track_from_event">
        <field name="res_model">event.track</field>
        <field name="name">Event Tracks</field>
        <field name="view_mode">kanban,list,form,calendar,graph,activity</field>
        <field name="context">{'search_default_event_id': active_id, 'default_event_id': active_id}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a Track
          </p><p>
            Tracks define your event schedule. They can be talks, workshops or any similar activity.
          </p>
        </field>
    </record>

    <record id="event_track_action_from_visitor" model="ir.actions.act_window">
        <field name="name">Wishlisted Tracks</field>
        <field name="res_model">event.track</field>
        <field name="view_mode">kanban,list,form</field>
        <field name="domain">[('wishlist_visitor_ids', 'in', [active_id])]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No track favorited by this visitor
            </p>
        </field>
    </record>

</odoo>

```

## File: views\event_track_visitor_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>

    <record id="event_track_visitor_view_search" model="ir.ui.view" >
        <field name="name">event.track.visitor.view.search</field>
        <field name="model">event.track.visitor</field>
        <field name="arch" type="xml">
            <search string="Track Visitors">
                <field name="track_id"/>
                <field name="visitor_id"/>
                <field name="partner_id"/>
                <field name="is_wishlisted"/>
                <group string="Group By" expand="0">
                    <filter string="Track" name="groupby_track_id" context="{'group_by': 'track_id'}"/>
                    <filter string="Visitor" name="groupby_visitor_id" context="{'group_by': 'visitor_id'}"/>
                    <filter string="Customer" name="groupby_partner_id" context="{'group_by': 'partner_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="event_track_visitor_view_form" model="ir.ui.view">
        <field name="name">event.track.visitor.view.form</field>
        <field name="model">event.track.visitor</field>
        <field name="arch" type="xml">
            <form string="Track Visitor">
                <sheet string="Track Visitor">
                    <group>
                        <group>
                            <field name="track_id"/>
                            <field name="is_wishlisted"/>
                        </group>
                        <group>
                            <field name="visitor_id"/>
                            <field name="partner_id"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="event_track_visitor_view_list" model="ir.ui.view">
        <field name="name">event.track.visitor.view.list</field>
        <field name="model">event.track.visitor</field>
        <field name="arch" type="xml">
            <list string="Track Visitors">
                <field name="track_id"/>
                <field name="visitor_id"/>
                <field name="partner_id"/>
                <field name="is_wishlisted"/>
            </list>
        </field>
    </record>

    <record id="event_track_visitor_action" model="ir.actions.act_window">
        <field name="name">Track Visitors</field>
        <field name="res_model">event.track.visitor</field>
        <field name="view_mode">list,form</field>
        <field name="context">{'create': False}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
              No Track Visitors yet!
            </p><p>
              Track Visitors store statistics on your events, including how many times tracks have been wishlisted.
            </p><p>
              They will be created automatically once attendees start browsing your events.
            </p>
        </field>
    </record>

</data></odoo>

```

## File: views\event_type_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="event_type_view_form_inherit_track" model="ir.ui.view">
        <field name="name">event.type.view.form.inherit.track</field>
        <field name="model">event.type</field>
        <field name="inherit_id" ref="website_event.event_type_view_form"/>
        <field name="arch" type="xml">
                <xpath expr="//span[@name='website_menu']" position='after'>
                <span>
                    <label for="website_track" string="Tracks Menu Item"/>
                    <field name="website_track"/>
                </span>
                <span name="website_track_proposal">
                    <label for="website_track_proposal" string="Track Proposals Menu Item"/>
                    <field name="website_track_proposal"/>
                </span>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\res_config_settings_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="20"/>
        <field name="inherit_id" ref="website.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <!-- TODO TDE there are two div with website_settings id in the website settings view -->
            <xpath expr="//block[@id='website_settings']" position="inside">
                <setting id="events_app_setting" groups="base.group_no_one">
                    <label for="events_app_name" string="Events PWA"/>
                    <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                    <div class="text-muted">
                        Name of your website's Events Progressive Web Application
                    </div>
                    <div class="content-group">
                        <div class="row mt16">
                            <label class="col-lg-3 o_light_label" string="Name" for="events_app_name"/>
                            <field name="events_app_name" required="website_id"/>
                        </div>
                    </div>
                </setting>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="snippet_options" inherit_id="website.snippet_options" name="Event track snippet options">
    <xpath expr="." position="inside">
        <div data-selector="main:has(.o_weagenda_topbar_filters)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Event Page">
            <we-checkbox string="Wishlists"
                         data-customize-website-views="website_event_track.agenda_topbar_wishlist"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
        <div data-selector="main:has(.o_wesession_index)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Event Page">
            <we-checkbox string="Wishlists"
                         data-customize-website-views="website_event_track.session_topbar_wishlist"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Filter by Tags"
                         data-customize-website-views="website_event_track.session_topbar_tag"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
    </xpath>
</template>

</odoo>

```

## File: views\website_visitor_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="website_visitor_view_tree" model="ir.ui.view">
        <field name="name">website.visitor.view.list.inherit.event.track</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website_event.website_visitor_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='event_registration_count']" position="after">
                <field name="event_track_wishlisted_count" optional="hide"/>
            </xpath>
        </field>
    </record>

    <record id="website_visitor_view_form" model="ir.ui.view">
        <field name="name">website.visitor.view.form.inherit.event.track</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website_event.website_visitor_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@id='w_visitor_visit_counter']" position="before">
                <button name="%(website_event_track.event_track_action_from_visitor)d"
                    type="action"
                    class="oe_stat_button" icon="fa-ticket"
                    groups="event.group_event_manager"
                    invisible="event_track_wishlisted_count == 0">
                    <field name="event_track_wishlisted_count" widget="statinfo" string="Tracks"/>
                </button>
            </xpath>
        </field>
    </record>
</data></odoo>

```

