# Odoo Module: website_event

Category: Marketing/Events

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
    'name': 'Events',
    'version': '1.4',
    'category': 'Marketing/Events',
    'sequence': 140,
    'summary': 'Publish events, sell tickets',
    'website': 'https://www.odoo.com/app/events',
    'description': "",
    'depends': [
        'event',
        'website',
        'website_partner',
        'website_mail',
    ],
    'data': [
        'data/event_data.xml',
        'views/res_config_settings_views.xml',
        'views/event_snippets.xml',
        'views/event_templates_list.xml',
        'views/event_templates_page.xml',
        'views/event_templates_page_registration.xml',
        'views/event_templates_page_misc.xml',
        'views/event_templates_widgets.xml',
        'views/website_templates.xml',
        'views/event_event_views.xml',
        'views/event_registration_views.xml',
        'views/event_tag_category_views.xml',
        'views/event_type_views.xml',
        'views/website_event_menu_views.xml',
        'views/website_visitor_views.xml',
        'views/event_menus.xml',
        'security/ir.model.access.csv',
        'security/event_security.xml',
    ],
    'demo': [
        'data/res_partner_demo.xml',
        'data/website_visitor_demo.xml',
        'data/event_demo.xml',
        'data/event_registration_demo.xml',
    ],
    'application': True,
    'assets': {
        'web.assets_common': [
            'website_event/static/src/js/tours/**/*',
        ],
        'web.assets_tests': [
            'website_event/static/tests/**/*',
        ],
        'web.assets_frontend': [
            'website_event/static/src/scss/event_templates_common.scss',
            'website_event/static/src/scss/event_templates_list.scss',
            'website_event/static/src/scss/event_templates_page.scss',
            'website_event/static/src/snippets/s_country_events_list/000.scss',
            'website_event/static/src/js/display_timer_widget.js',
            'website_event/static/src/js/register_toaster_widget.js',
            'website_event/static/src/js/website_geolocation.js',
            'website_event/static/src/js/website_event.js',
            'website_event/static/src/js/website_event_ticket_details.js',
            'website_event/static/src/js/website_event_set_customize_options.js',
        ],
        'website.assets_editor': [
            'website_event/static/src/js/website_event.editor.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\community.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request


class EventCommunityController(http.Controller):

    @http.route('/event/<model("event.event"):event>/community', type="http", auth="public", website=True, sitemap=False)
    def community(self, event, lang=None, **kwargs):
        """ This skeleton route will be overriden in website_event_track_quiz, website_event_meet and website_event_meet_quiz. """
        return request.render('website.page_404')

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-

import babel.dates
import pytz
import re
import werkzeug

from ast import literal_eval
from collections import defaultdict
from datetime import datetime, timedelta
from dateutil.parser import parse
from dateutil.relativedelta import relativedelta
from werkzeug.datastructures import OrderedMultiDict
from werkzeug.exceptions import NotFound

from odoo import fields, http, _
from odoo.addons.http_routing.models.ir_http import slug
from odoo.addons.website.controllers.main import QueryURL
from odoo.http import request
from odoo.osv import expression

from odoo.tools.misc import get_lang
from odoo.exceptions import UserError

class WebsiteEventController(http.Controller):

    def sitemap_event(env, rule, qs):
        if not qs or qs.lower() in '/events':
            yield {'loc': '/events'}

    # ------------------------------------------------------------
    # EVENT LIST
    # ------------------------------------------------------------

    @http.route(['/event', '/event/page/<int:page>', '/events', '/events/page/<int:page>'], type='http', auth="public", website=True, sitemap=sitemap_event)
    def events(self, page=1, **searches):
        Event = request.env['event.event']
        SudoEventType = request.env['event.type'].sudo()

        searches.setdefault('search', '')
        searches.setdefault('date', 'all')
        searches.setdefault('tags', '')
        searches.setdefault('type', 'all')
        searches.setdefault('country', 'all')

        website = request.website

        step = 12  # Number of events per page

        options = {
            'displayDescription': False,
            'displayDetail': False,
            'displayExtraDetail': False,
            'displayExtraLink': False,
            'displayImage': False,
            'allowFuzzy': not searches.get('noFuzzy'),
            'date': searches.get('date'),
            'tags': searches.get('tags'),
            'type': searches.get('type'),
            'country': searches.get('country'),
        }
        order = 'date_begin'
        if searches.get('date', 'all') == 'old':
            order = 'date_begin desc'
        order = 'is_published desc, ' + order + ', id desc'
        search = searches.get('search')
        event_count, details, fuzzy_search_term = website._search_with_fuzzy("events", search,
            limit=page * step, order=order, options=options)
        event_details = details[0]
        events = event_details.get('results', Event)
        events = events[(page - 1) * step:page * step]

        # count by domains without self search
        domain_search = [('name', 'ilike', fuzzy_search_term or searches['search'])] if searches['search'] else []

        no_date_domain = event_details['no_date_domain']
        dates = event_details['dates']
        for date in dates:
            if date[0] != 'old':
                date[3] = Event.search_count(expression.AND(no_date_domain) + domain_search + date[2])

        no_country_domain = event_details['no_country_domain']
        countries = Event.read_group(expression.AND(no_country_domain) + domain_search, ["id", "country_id"],
            groupby="country_id", orderby="country_id")
        countries.insert(0, {
            'country_id_count': sum([int(country['country_id_count']) for country in countries]),
            'country_id': ("all", _("All Countries"))
        })

        search_tags = event_details['search_tags']
        current_date = event_details['current_date']
        current_type = None
        current_country = None

        if searches["type"] != 'all':
            current_type = SudoEventType.browse(int(searches['type']))

        if searches["country"] != 'all' and searches["country"] != 'online':
            current_country = request.env['res.country'].browse(int(searches['country']))

        pager = website.pager(
            url="/event",
            url_args=searches,
            total=event_count,
            page=page,
            step=step,
            scope=5)

        keep = QueryURL('/event', **{key: value for key, value in searches.items() if (key == 'search' or value != 'all')})

        searches['search'] = fuzzy_search_term or search

        values = {
            'current_date': current_date,
            'current_country': current_country,
            'current_type': current_type,
            'event_ids': events,  # event_ids used in website_event_track so we keep name as it is
            'dates': dates,
            'categories': request.env['event.tag.category'].search([('is_published', '=', True)]),
            'countries': countries,
            'pager': pager,
            'searches': searches,
            'search_tags': search_tags,
            'keep': keep,
            'search_count': event_count,
            'original_search': fuzzy_search_term and search,
        }

        if searches['date'] == 'old':
            # the only way to display this content is to set date=old so it must be canonical
            values['canonical_params'] = OrderedMultiDict([('date', 'old')])

        return request.render("website_event.index", values)

    # ------------------------------------------------------------
    # EVENT PAGE
    # ------------------------------------------------------------

    @http.route(['''/event/<model("event.event"):event>/page/<path:page>'''], type='http', auth="public", website=True, sitemap=False)
    def event_page(self, event, page, **post):
        values = {
            'event': event,
        }

        if '.' not in page:
            page = 'website_event.%s' % page

        try:
            # Every event page view should have its own SEO.
            values['seo_object'] = request.website.get_template(page)
            values['main_object'] = event
        except ValueError:
            # page not found
            values['path'] = re.sub(r"^website_event\.", '', page)
            values['from_template'] = 'website_event.default_page'  # .strip('website_event.')
            page = request.website.is_publisher() and 'website.page_404' or 'http_routing.404'

        return request.render(page, values)

    @http.route(['''/event/<model("event.event"):event>'''], type='http', auth="public", website=True, sitemap=True)
    def event(self, event, **post):
        if event.menu_id and event.menu_id.child_id:
            target_url = event.menu_id.child_id[0].url
        else:
            target_url = '/event/%s/register' % str(event.id)
        if post.get('enable_editor') == '1':
            target_url += '?enable_editor=1'
        return request.redirect(target_url)

    @http.route(['''/event/<model("event.event"):event>/register'''], type='http', auth="public", website=True, sitemap=False)
    def event_register(self, event, **post):
        values = self._prepare_event_register_values(event, **post)
        return request.render("website_event.event_description_full", values)

    def _prepare_event_register_values(self, event, **post):
        """Return the require values to render the template."""
        urls = event._get_event_resource_urls()
        return {
            'event': event,
            'main_object': event,
            'range': range,
            'google_url': urls.get('google_url'),
            'iCal_url': urls.get('iCal_url'),
        }

    def _process_tickets_form(self, event, form_details):
        """ Process posted data about ticket order. Generic ticket are supported
        for event without tickets (generic registration).

        :return: list of order per ticket: [{
            'id': if of ticket if any (0 if no ticket),
            'ticket': browse record of ticket if any (None if no ticket),
            'name': ticket name (or generic 'Registration' name if no ticket),
            'quantity': number of registrations for that ticket,
        }, {...}]
        """
        ticket_order = {}
        for key, value in form_details.items():
            registration_items = key.split('nb_register-')
            if len(registration_items) != 2:
                continue
            ticket_order[int(registration_items[1])] = int(value)

        ticket_dict = dict((ticket.id, ticket) for ticket in request.env['event.event.ticket'].sudo().search([
            ('id', 'in', [tid for tid in ticket_order.keys() if tid]),
            ('event_id', '=', event.id)
        ]))

        return [{
            'id': tid if ticket_dict.get(tid) else 0,
            'ticket': ticket_dict.get(tid),
            'name': ticket_dict[tid]['name'] if ticket_dict.get(tid) else _('Registration'),
            'quantity': count,
        } for tid, count in ticket_order.items() if count]

    @http.route(['/event/<model("event.event"):event>/registration/new'], type='json', auth="public", methods=['POST'], website=True)
    def registration_new(self, event, **post):
        tickets = self._process_tickets_form(event, post)
        availability_check = True
        if event.seats_limited:
            ordered_seats = 0
            for ticket in tickets:
                ordered_seats += ticket['quantity']
            if event.seats_available < ordered_seats:
                availability_check = False
        if not tickets:
            return False
        default_first_attendee = {}
        if not request.env.user._is_public():
            default_first_attendee = {
                "name": request.env.user.name,
                "email": request.env.user.email,
                "phone": request.env.user.mobile or request.env.user.phone,
            }
        else:
            visitor = request.env['website.visitor']._get_visitor_from_request()
            if visitor.email:
                default_first_attendee = {
                    "name": visitor.name,
                    "email": visitor.email,
                    "phone": visitor.mobile,
                }
        return request.env['ir.ui.view']._render_template("website_event.registration_attendee_details", {
            'tickets': tickets,
            'event': event,
            'availability_check': availability_check,
            'default_first_attendee': default_first_attendee,
        })

    def _process_attendees_form(self, event, form_details):
        """ Process data posted from the attendee details form.

        :param form_details: posted data from frontend registration form, like
            {'1-name': 'r', '1-email': 'r@r.com', '1-phone': '', '1-event_ticket_id': '1'}
        """
        allowed_fields = request.env['event.registration']._get_website_registration_allowed_fields()
        registration_fields = {key: v for key, v in request.env['event.registration']._fields.items() if key in allowed_fields}
        for ticket_id in list(filter(lambda x: x is not None, [form_details[field] if 'event_ticket_id' in field else None for field in form_details.keys()])):
            if int(ticket_id) not in event.event_ticket_ids.ids and len(event.event_ticket_ids.ids) > 0:
                raise UserError(_("This ticket is not available for sale for this event"))
        registrations = {}
        global_values = {}
        for key, value in form_details.items():
            counter, attr_name = key.split('-', 1)
            field_name = attr_name.split('-')[0]
            if field_name not in registration_fields:
                continue
            elif isinstance(registration_fields[field_name], (fields.Many2one, fields.Integer)):
                value = int(value) or False  # 0 is considered as a void many2one aka False
            else:
                value = value

            if counter == '0':
                global_values[attr_name] = value
            else:
                registrations.setdefault(counter, dict())[attr_name] = value
        for key, value in global_values.items():
            for registration in registrations.values():
                registration[key] = value

        return list(registrations.values())

    def _create_attendees_from_registration_post(self, event, registration_data):
        """ Also try to set a visitor (from request) and
        a partner (if visitor linked to a user for example). Purpose is to gather
        as much informations as possible, notably to ease future communications.
        Also try to update visitor informations based on registration info. """
        visitor_sudo = request.env['website.visitor']._get_visitor_from_request(force_create=True)
        visitor_sudo._update_visitor_last_visit()
        visitor_values = {}

        registrations_to_create = []
        for registration_values in registration_data:
            registration_values['event_id'] = event.id
            if not registration_values.get('partner_id') and visitor_sudo.partner_id:
                registration_values['partner_id'] = visitor_sudo.partner_id.id
            elif not registration_values.get('partner_id'):
                registration_values['partner_id'] = False if request.env.user._is_public() else request.env.user.partner_id.id

            if visitor_sudo:
                # registration may give a name to the visitor, yay
                if registration_values.get('name') and not visitor_sudo.name and not visitor_values.get('name'):
                    visitor_values['name'] = registration_values['name']
                # update registration based on visitor
                registration_values['visitor_id'] = visitor_sudo.id

            registrations_to_create.append(registration_values)

        if visitor_values:
            visitor_sudo.write(visitor_values)

        return request.env['event.registration'].sudo().create(registrations_to_create)

    @http.route(['''/event/<model("event.event"):event>/registration/confirm'''], type='http', auth="public", methods=['POST'], website=True)
    def registration_confirm(self, event, **post):
        registrations = self._process_attendees_form(event, post)
        attendees_sudo = self._create_attendees_from_registration_post(event, registrations)
        visitor_sudo = attendees_sudo.visitor_id

        redirect = request.redirect(('/event/%s/registration/success?' % event.id) + werkzeug.urls.url_encode({'registration_ids': ",".join([str(id) for id in attendees_sudo.ids])}))

        # make sure the vistor's uuid is correctly logged in cookies when disabling "Track Visitor" on all the pages of the event
        # we set visitor_uuid in the cookie to be sure "event_registration_success" can retrieve the visitor
        if request.httprequest.cookies.get('visitor_uuid', '') != visitor_sudo.access_token:
            expiration_date = datetime.now() + timedelta(days=365)
            redirect.set_cookie('visitor_uuid', visitor_sudo.access_token, expires=expiration_date)

        return redirect

    @http.route(['/event/<model("event.event"):event>/registration/success'], type='http', auth="public", methods=['GET'], website=True, sitemap=False)
    def event_registration_success(self, event, registration_ids):
        # fetch the related registrations, make sure they belong to the correct visitor / event pair
        visitor = request.env['website.visitor']._get_visitor_from_request()
        if not visitor:
            raise NotFound()
        attendees_sudo = request.env['event.registration'].sudo().search([
            ('id', 'in', [str(registration_id) for registration_id in registration_ids.split(',')]),
            ('event_id', '=', event.id),
            ('visitor_id', '=', visitor.id),
        ])
        return request.render("website_event.registration_complete",
            self._get_registration_confirm_values(event, attendees_sudo))

    def _get_registration_confirm_values(self, event, attendees_sudo):
        urls = event._get_event_resource_urls()
        return {
            'attendees': attendees_sudo,
            'event': event,
            'google_url': urls.get('google_url'),
            'iCal_url': urls.get('iCal_url')
        }

    # ------------------------------------------------------------
    # EDITOR (NEW EVENT)
    # ------------------------------------------------------------

    @http.route('/event/add_event', type='json', auth="user", methods=['POST'], website=True)
    def add_event(self, name, event_start, event_end, address_values, **kwargs):
        values = self._prepare_event_values(name, event_start, event_end, address_values)
        event = request.env['event.event'].create(values)
        return "/event/%s/register?enable_editor=1" % slug(event)

    def _prepare_event_values(self, name, event_start, event_end, address_values=None):
        """
        Return the values to create a new event.
        event_start,event_date are datetimes in the user tz.
        address_values is used to either choose an existing location or create one as we allow it in the frontend.
        """
        date_begin = parse(event_start).astimezone(pytz.utc).replace(tzinfo=None)
        date_end = parse(event_end).astimezone(pytz.utc).replace(tzinfo=None)
        address_id = request.env['res.partner']
        if address_values:
            (address_pid, address_vals) = int(address_values[0]), address_values[1]
            address_id = address_pid
            if address_pid == 0:
                address_id = request.env['res.partner'].create(address_vals).id
        return {
            'name': name,
            'date_begin': date_begin,
            'date_end': date_end,
            'address_id': address_id,
            'seats_available': 1000,
            'website_id': request.website.id,
            'event_ticket_ids': request.env['event.event.ticket'],
        }

    # ------------------------------------------------------------
    # TOOLS (JSON)
    # ------------------------------------------------------------

    @http.route('/event/get_country_event_list', type='json', auth='public', website=True)
    def get_country_events(self, **post):
        Event = request.env['event.event']
        country_code = request.session['geoip'].get('country_code')
        result = {'events': [], 'country': False}
        events = None
        domain = request.website.website_domain()
        if country_code:
            country = request.env['res.country'].search([('code', '=', country_code)], limit=1)
            events = Event.search(domain + ['|', ('address_id', '=', None), ('country_id.code', '=', country_code), ('date_begin', '>=', '%s 00:00:00' % fields.Date.today())], order="date_begin")
        if not events:
            events = Event.search(domain + [('date_begin', '>=', '%s 00:00:00' % fields.Date.today())], order="date_begin")
        for event in events:
            if country_code and event.country_id.code == country_code:
                result['country'] = country
            result['events'].append({
                "date": self.get_formated_date(event),
                "event": event,
                "url": event.website_url})
        return request.env['ir.ui.view']._render_template("website_event.country_events_list", result)

    # ------------------------------------------------------------
    # TOOLS (HELPERS)
    # ------------------------------------------------------------

    def get_formated_date(self, event):
        start_date = fields.Datetime.from_string(event.date_begin).date()
        end_date = fields.Datetime.from_string(event.date_end).date()
        month = babel.dates.get_month_names('abbreviated', locale=get_lang(event.env).code)[start_date.month]
        return ('%s %s%s') % (month, start_date.strftime("%e"), (end_date != start_date and ("-" + end_date.strftime("%e")) or ""))

    def _extract_searched_event_tags(self, searches):
        tags = request.env['event.tag']
        if searches.get('tags'):
            try:
                tag_ids = literal_eval(searches['tags'])
            except:
                pass
            else:
                # perform a search to filter on existing / valid tags implicitely + apply rules on color
                tags = request.env['event.tag'].search([('id', 'in', tag_ids)])
        return tags

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main
from . import community

```

## File: data\event_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="menu_events" model="website.menu">
            <field name="name">Events</field>
            <field name="url">/event</field>
            <field name="parent_id" ref="website.main_menu"/>
            <field name="sequence" type="int">30</field>
        </record>
        <record id="action_open_website" model="ir.actions.act_url">
            <field name="name">Website Home</field>
            <field name="target">self</field>
            <field name="url">/event</field>
        </record>
        <record id="base.open_menu" model="ir.actions.todo">
            <field name="action_id" ref="action_open_website"/>
            <field name="state">open</field>
        </record>

        <record id="mt_event_published" model="mail.message.subtype">
            <field name="name">Event published</field>
            <field name="res_model">event.event</field>
            <field name="default" eval="False"/>
            <field name="description">Event published</field>
        </record>
        <record id="mt_event_unpublished" model="mail.message.subtype">
            <field name="name">Event unpublished</field>
            <field name="res_model">event.event</field>
            <field name="default" eval="False"/>
            <field name="description">Event unpublished</field>
        </record>

    </data>
</odoo>

```

## File: data\event_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event.event_0" model="event.event">
        <field name="website_published" eval="True"/>
        <field name="subtitle">Get Inspired • Stay Connected • Have Fun</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_0.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
        <field name="description" type="html">
            <div class="oe_structure">
                <h5>Join us for this 3-day Event</h5>
                <p class="lead mb-3">Every year we invite our community, partners and end-users to come and meet us! It's the ideal event to get together and present new features, roadmap of future versions, achievements of the software, workshops, training sessions, etc....</p>
                <p class="mb-3">This event is also an opportunity to showcase our partners' case studies, methodology or developments. Be there and see directly from the source the features of the version 12!
                </p>
                <div class="bg-light rounded-right border-left border-secondary p-3 mb-5" style="border-left-width: 3px !important;">
                    <p class="mb-0"><i class="fa fa-info-circle mr-2"/>This event and all the conferences are in <b>English</b>!</p>
                </div>
                <h5 class="mb-2">What's new?</h5>
                <ul class="mb-5">
                    <li class="mb-2"><b>The Design Fair is preceded by 2 days of Training Sessions for experts!</b><br/> We propose 3 different training sessions, 2 days each.</li>
                    <li class="mb-2"><b>The whole event is open to all public!</b> <br/>We ask a participation fee of 49.50€ for the costs for the 3 days (coffee breaks, catering, drinks and a surprising concert and beer party).<br/> For those who don't want to contribute, there is a free ticket, therefore, catering and access to evening events aren't included.</li>
                    <li class="mb-2"><b>The plenary sessions in the morning will be shorter</b> and we will give more time for thematical meetings, conferences, workshops and tutorial sessions in the afternoon.</li>
                </ul>
                <h5 class="mb-2">Program</h5>
                <p>Conferences, workshops and trainings will be organized in 6 rooms:</p>
                <ul class="mb-5">
                    <li><b>Technical Rooms</b> - One dedicated to advanced Odoo developers, one for new developers.</li>
                    <li><b>Technical Rooms</b> - One dedicated to advanced Odoo developers, one for new developers.</li>
                    <li><b>Business Room</b> - To discuss implementation methodologies, best sales practices, etc.</li>
                    <li><b>Workshop Room</b> - Mainly for developers.</li>
                </ul>
                <div class="bg-light rounded-right border-left border-secondary p-3 mb-3" style="border-left-width: 3px !important;">
                    <p class="mb-0"><em>If you wish to make a presentation, please send your topic proposal as soon as possible for approval to Mr. Famke Jenssens at ngh (a) yourcompany (dot) com. The presentations should be, for example, a presentation of a community module, a case study, methodology feedback, technical, etc. Each presentation must be in English.</em></p>
                </div>
                <p class="mb-3">For any additional information, please contact us at <a href="mailto:events@yourcompany.com">events@yourcompany.com</a>.</p>
                <div class="bg-light rounded-right border-left border-secondary p-3 mb-5" style="border-left-width: 3px !important;">
                    <p class="mb-0">OpenElec Applications reserves the right to cancel, re-name or re-locate the event or change the dates on which it is held.</p>
                </div>
            </div>
        </field>
    </record>

    <record id="event.event_1" model="event.event">
        <field name="website_menu" eval="True"/>
        <field name="website_published" eval="True"/>
        <field name="menu_register_cta" eval="True"/>
        <field name="subtitle">The Great Reno Balloon Race is the world's largest free hot-air ballooning event.</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_1.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
        <field name="description" type="html">
            <div class="oe_structure">
                <h5>Join us for the greatest ballon race of all times !</h5>
                <p class="lead mb-3">The best aeronauts of the world will gather on this event to offer you the most spectacular show.</p>
                <p class="lead mb-3">Around one hundred ballons will simultaneously take flight and turn the sky into a beautiful canvas of colours.</p>
                <p class="lead mb-3">This is the perfect place for spending a nice day with your family, we guarantee you will be leaving with beautiful everlasting memories !</p>
                <p class="mb-3">For any additional information, please contact us at <a href="mailto:events@yourcompany.com">events@yourcompany.com</a>.</p>
                <div class="bg-light rounded-right border-left border-secondary p-3 mb-5" style="border-left-width: 3px !important;">
                    <p class="mb-1">We reserve the right to cancel, re-name or re-locate the event or change the dates on which it is held in case the weather fails us.</p>
                    <p class="mb0">The safety of our attendees and our aeronauts comes first !</p>
                </div>
            </div>
        </field>
    </record>

    <record id="event.event_2" model="event.event">
        <field name="website_published" eval="True"/>
        <field name="subtitle">Enhance your architectural business and improve professional skills.</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_2.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
        <field name="description" type="html">
<div class="oe_structure">
    <h5>Conference for Architects</h5>
    <p class="lead">During this conference, our team will give a detailed overview of our business applications. You’ll know all the benefits of using it.</p>
    <h6>Objectives</h6>
    <p>Having attended this conference, participants should be able to:</p>
    <ul class="mb-4">
        <li>Understand the various modules;</li>
        <li>Functional flow of the main applications;</li>
    </ul>
    <h6>Program</h6>
    <ul class="mb-4">
        <li>Introduction, CRM, Sales Management</li>
        <li>Purchase, Sales &amp; Purchase management, Financial accounting.</li>
        <li>Project management, Human resources, Contract management.</li>
        <li>Warehouse management, Manufacturing (MRP) &amp; Sales, Import/Export.</li>
        <li>Point of Sale (POS), Introduction to report customization.</li>
    </ul>

    <p>For any additional information, please contact us at <a href="mailto:events@odoo.com">events@odoo.com</a></p>

    <section class="s_we_speaker bg-200 p-3 mb-4" itemscope="itemscope" itemtype="http://schema.org/Person" itemprop="performer">
        <span class="badge badge-secondary o_wevent_badge float-right">SPEAKER</span>
        <img src="/mail/static/src/img/odoobot.png" width="70" class="img-fluid rounded-circle float-left mr-3" alt=""/>
        <div class="overflow-hidden">
            <h4 class="mt-3 mb-1" itemprop="name">John DOE</h4>
            <h6 class="mb-4">Company</h6>
            <p>At just 13 years old, John DOE was already starting to develop his first business applications for customers. After mastering civil engineering, he founded TinyERP. This was the first phase of OpenERP which would later became Odoo, the most installed open-source business software worldwide.</p>
        </div>
    </section>

    <div class="alert alert-info">
        <p class="mb-0">Chamber Works reserves the right to cancel, re-name or re-locate the event or change the dates on which it is held.</p>
    </div>
</div>
        </field>
    </record>

    <record id="event.event_3" model="event.event">
        <field name="website_published" eval="True"/>
        <field name="subtitle">Experience live music, local food and beverages.</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_3.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
        <field name="description" type="html">
            <div class="oe_structure">
                <h5>Here it is, the 12th edition of our Live Musical Festival !</h5>
                <p class="lead mb-3">Once again we assembled the most legendary bands in Rock history.</p>
                <p class="lead mb-3">Bands like Bar Fighters, Led Slippers and Link Floyd will offer you the show of the century during our three day event.</p>
                <p class="lead mb-3">This is the perfect place for spending a nice time with your friends while listening to some of the most iconic rock songs of all times!</p>
                <p class="mb-3">For any additional information, please contact us at <a href="mailto:events@yourcompany.com">events@yourcompany.com</a>.</p>
                <div class="bg-light rounded-right border-left border-secondary p-3 mb-5" style="border-left-width: 3px !important;">
                    <p class="mb-1">We reserve the right to cancel, re-name or re-locate the event or change the dates on which it is held in case the weather fails us.</p>
                </div>
            </div>
        </field>
    </record>

    <record id="event.event_4" model="event.event">
        <field name="website_published" eval="True"/>
        <field name="subtitle">Discover how to grow a sustainable business with our experts.</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_4.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
    </record>

    <record id="event.event_5" model="event.event">
        <field name="website_published" eval="True"/>
        <field name="subtitle">Bring your outdoor field hockey season to the next level by taking the field at this 9th annual Field Hockey tournament.</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_5.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
        <field name="description" type="html">
            <div class="oe_structure">
                <h5>Seasoned Hockey Fans and curious people, this tournament is for you !</h5>
                <p class="lead mb-3">The best Hockey teams of the country will compete for the national Hockey trophy.</p>
                <p class="lead mb-3">If you don't know anything about Hockey, this is a great introduction to this wonderful sport as you will will be able to see some training process and also have some time
                to chat with experienced players and trainers once the tournament is over !
                </p>
                <p class="mb-3">For any additional information, please contact us at <a href="mailto:events@yourcompany.com">events@yourcompany.com</a>.</p>
                <div class="bg-light rounded-right border-left border-secondary p-3 mb-5" style="border-left-width: 3px !important;">
                    <p class="mb-1">We reserve the right to cancel, re-name or re-locate the event or change the dates on which it is held in case the weather fails us.</p>
                </div>
            </div>
        </field>
    </record>

    <record id="event.event_6" model="event.event">
        <field name="website_published" eval="False"/>
        <field name="cover_properties">{"background-image": "none", "background-color": "secondary", "opacity": ""}</field>
    </record>

    <record id="event.event_7" model="event.event">
        <field name="website_menu" eval="True"/>
        <field name="website_published" eval="True"/>
        <field name="menu_register_cta" eval="True"/>
        <field name="subtitle">Our newest collection will be revealed online! Interact with us on our live streams!</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_7.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
        <field name="description" type="html">
<div class="oe_structure">
    <h5>The finest OpenWood furnitures are coming to your house in a brand new collection</h5>
    <p>
        And this time, we go fully ONLINE! Meet us in our live streams from the comfort of your house.<br/>
        Special discount codes will be handed out during the various streams, make sure to be there on time.
    </p>
    <p class="mb-3">For any additional information, please contact us at <a href="mailto:events@openwood.example.com">events@penwood.example.com</a>.</p>
    <div class="bg-light rounded-right border-left border-secondary p-3 mb-5" style="border-left-width: 3px !important;">
        <p class="mb-1">This event is fully online and FREE, if you have paid for tickets, you should get a refund.<br/>
        It will require a good Internet connection to get the best video quality.</p>
    </div>
</div>
        </field>
    </record>

</odoo>

```

## File: data\event_registration_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <!-- OpenWood Collection Online Reveal: Gemini (all) -->
    <record id="event.event_registration_7_0" model="event.registration">
        <field name="visitor_id" ref="website_event.website_visitor_event_0"/>
    </record>
    <record id="event.event_registration_7_1" model="event.registration">
        <field name="visitor_id" ref="website_event.website_visitor_event_0"/>
    </record>
    <record id="event.event_registration_7_2" model="event.registration">
        <field name="visitor_id" ref="website_event.website_visitor_event_1"/>
    </record>
    <record id="event.event_registration_7_3" model="event.registration">
        <field name="visitor_id" ref="website_event.website_visitor_event_1"/>
    </record>

</data></odoo>

```

## File: data\res_partner_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="base.res_partner_1" model="res.partner">
        <field name="is_published">True</field>
        <field name="website_short_description">Wood Corner brings honesty and seriousness to wood industry while helping customers deal with trees, flowers and fungi.</field>
        <field name="website_description" type="html">
<div><section class="s_text_image o_colored_level" data-snippet="s_image_text" data-name="Image - Text">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-3 pt8 pb8 o_colored_level">
                <img src="/web/image/website.s_image_text_default_image" class="img img-fluid mx-auto" alt=""/>
            </div>
            <div class="col-lg-9 pt8 pb8 o_colored_level">
                <h2 class="o_default_snippet_text">Happy to be Sponsor</h2>
                <p class="o_default_snippet_text">As a team, we are happy to contribute to this event.</p>
                <p class="o_default_snippet_text">Come see us live, we hope to meet you !</p>
                <p><a href="#" class="btn btn-primary o_default_snippet_text">Discover more</a></p>
            </div>
        </div>
    </div>
</section></div></field>
    </record>

    <record id="base.res_partner_2" model="res.partner">
        <field name="is_published">True</field>
        <field name="website_short_description">Deco Addict brings honesty and seriousness to wood industry while helping customers deal with trees, flowers and fungi.</field>
        <field name="website_description" type="html">
<div><section class="s_text_image o_colored_level" data-snippet="s_image_text" data-name="Image - Text">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-3 pt8 pb8 o_colored_level">
                <img src="/web/image/website.s_image_text_default_image" class="img img-fluid mx-auto" alt=""/>
            </div>
            <div class="col-lg-9 pt8 pb8 o_colored_level">
                <h2 class="o_default_snippet_text">Happy to be Sponsor</h2>
                <p class="o_default_snippet_text">As a team, we are happy to contribute to this event.</p>
                <p class="o_default_snippet_text">Come see us live, we hope to meet you !</p>
                <p><a href="#" class="btn btn-primary o_default_snippet_text">Discover more</a></p>
            </div>
        </div>
    </div>
</section></div></field>
    </record>

    <record id="base.res_partner_3" model="res.partner">
        <field name="is_published">True</field>
        <field name="website_short_description">Gemini Furniture brings honesty and seriousness to wood industry while helping customers deal with trees, flowers and fungi.</field>
        <field name="website_description" type="html">
<div><section class="s_text_image o_colored_level" data-snippet="s_image_text" data-name="Image - Text">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-3 pt8 pb8 o_colored_level">
                <img src="/web/image/website.s_image_text_default_image" class="img img-fluid mx-auto" alt=""/>
            </div>
            <div class="col-lg-9 pt8 pb8 o_colored_level">
                <h2 class="o_default_snippet_text">Happy to be Sponsor</h2>
                <p class="o_default_snippet_text">As a team, we are happy to contribute to this event.</p>
                <p class="o_default_snippet_text">Come see us live, we hope to meet you !</p>
                <p><a href="#" class="btn btn-primary o_default_snippet_text">Discover more</a></p>
            </div>
        </div>
    </div>
</section></div></field>
    </record>

    <record id="base.res_partner_4" model="res.partner">
        <field name="is_published">True</field>
        <field name="website_short_description">Ready Mat brings honesty and seriousness to wood industry while helping customers deal with trees, flowers and fungi.</field>
        <field name="website_description" type="html">
<div><section class="s_text_image o_colored_level" data-snippet="s_image_text" data-name="Image - Text">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-3 pt8 pb8 o_colored_level">
                <img src="/web/image/website.s_image_text_default_image" class="img img-fluid mx-auto" alt=""/>
            </div>
            <div class="col-lg-9 pt8 pb8 o_colored_level">
                <h2 class="o_default_snippet_text">Happy to be Sponsor</h2>
                <p class="o_default_snippet_text">As a team, we are happy to contribute to this event.</p>
                <p class="o_default_snippet_text">Come see us live, we hope to meet you !</p>
                <p><a href="#" class="btn btn-primary o_default_snippet_text">Discover more</a></p>
            </div>
        </div>
    </div>
</section></div></field>
    </record>

    <record id="base.res_partner_10" model="res.partner">
        <field name="is_published">True</field>
        <field name="website_short_description">The Jackson Group brings honesty and seriousness to wood industry while helping customers deal with trees, flowers and fungi.</field>
        <field name="website_description" type="html">
<div><section class="s_text_image o_colored_level" data-snippet="s_image_text" data-name="Image - Text">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-3 pt8 pb8 o_colored_level">
                <img src="/web/image/website.s_image_text_default_image" class="img img-fluid mx-auto" alt=""/>
            </div>
            <div class="col-lg-9 pt8 pb8 o_colored_level">
                <h2 class="o_default_snippet_text">Happy to be Sponsor</h2>
                <p class="o_default_snippet_text">As a team, we are happy to contribute to this event.</p>
                <p class="o_default_snippet_text">Come see us live, we hope to meet you !</p>
                <p><a href="#" class="btn btn-primary o_default_snippet_text">Discover more</a></p>
            </div>
        </div>
    </div>
</section></div></field>
    </record>

    <record id="base.res_partner_12" model="res.partner">
        <field name="is_published">True</field>
        <field name="website_short_description">Azure Interior brings honesty and seriousness to wood industry while helping customers deal with trees, flowers and fungi.</field>
        <field name="website_description" type="html">
<div><section class="s_text_image o_colored_level" data-snippet="s_image_text" data-name="Image - Text">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-3 pt8 pb8 o_colored_level">
                <img src="/web/image/website.s_image_text_default_image" class="img img-fluid mx-auto" alt=""/>
            </div>
            <div class="col-lg-9 pt8 pb8 o_colored_level">
                <h2 class="o_default_snippet_text">Happy to be Sponsor</h2>
                <p class="o_default_snippet_text">As a team, we are happy to contribute to this event.</p>
                <p class="o_default_snippet_text">Come see us live, we hope to meet you !</p>
                <p><a href="#" class="btn btn-primary o_default_snippet_text">Discover more</a></p>
            </div>
        </div>
    </div>
</section></div></field>
    </record>

    <record id="event.res_partner_event_1" model="res.partner">
        <field name="is_published">True</field>
        <field name="website_short_description">Bloem brings honesty and seriousness to wood industry while helping customers deal with trees, flowers and fungi.</field>
        <field name="website_description" type="html">
<div><section class="s_text_image o_colored_level" data-snippet="s_image_text" data-name="Image - Text">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-3 pt8 pb8 o_colored_level">
                <img src="/web/image/website.s_image_text_default_image" class="img img-fluid mx-auto" alt=""/>
            </div>
            <div class="col-lg-9 pt8 pb8 o_colored_level">
                <h2 class="o_default_snippet_text">Happy to be Sponsor</h2>
                <p class="o_default_snippet_text">As a team, we are happy to contribute to this event.</p>
                <p class="o_default_snippet_text">Come see us live, we hope to meet you !</p>
                <p><a href="#" class="btn btn-primary o_default_snippet_text">Discover more</a></p>
            </div>
        </div>
    </div>
</section></div></field>
    </record>

    <record id="event.res_partner_event_2" model="res.partner">
        <field name="is_published">True</field>
        <field name="website_short_description">OpenWood brings honesty and seriousness to wood industry while helping customers deal with trees, flowers and fungi.</field>
        <field name="website_description" type="html">
<div><section class="s_text_image o_colored_level" data-snippet="s_image_text" data-name="Image - Text">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-3 pt8 pb8 o_colored_level">
                <img src="/web/image/website.s_image_text_default_image" class="img img-fluid mx-auto" alt=""/>
            </div>
            <div class="col-lg-9 pt8 pb8 o_colored_level">
                <h2 class="o_default_snippet_text">Happy to be Sponsor</h2>
                <p class="o_default_snippet_text">As a team, we are happy to contribute to this event.</p>
                <p class="o_default_snippet_text">Come see us live, we hope to meet you !</p>
                <p><a href="#" class="btn btn-primary o_default_snippet_text">Discover more</a></p>
            </div>
        </div>
    </div>
</section></div></field>
    </record>

    <record id="event.res_partner_event_3" model="res.partner">
        <field name="is_published">True</field>
        <field name="website_short_description">Tree Dealers brings honesty and seriousness to wood industry while helping customers deal with trees, flowers and fungi.</field>
        <field name="website_description" type="html">
<div><section class="s_text_image o_colored_level" data-snippet="s_image_text" data-name="Image - Text">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-3 pt8 pb8 o_colored_level">
                <img src="/web/image/website.s_image_text_default_image" class="img img-fluid mx-auto" alt=""/>
            </div>
            <div class="col-lg-9 pt8 pb8 o_colored_level">
                <h2 class="o_default_snippet_text">Happy to be Sponsor</h2>
                <p class="o_default_snippet_text">As a team, we are happy to contribute to this event.</p>
                <p class="o_default_snippet_text">Come see us live, we hope to meet you !</p>
                <p><a href="#" class="btn btn-primary o_default_snippet_text">Discover more</a></p>
            </div>
        </div>
    </div>
</section></div></field>
    </record>

    <record id="event.res_partner_event_4" model="res.partner">
        <field name="is_published">True</field>
        <field name="website_short_description">Shangai Pterocarpus Furniture brings honesty and seriousness to wood industry while helping customers deal with trees, flowers and fungi.</field>
        <field name="website_description" type="html">
<div><section class="s_text_image o_colored_level" data-snippet="s_image_text" data-name="Image - Text">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-3 pt8 pb8 o_colored_level">
                <img src="/web/image/website.s_image_text_default_image" class="img img-fluid mx-auto" alt=""/>
            </div>
            <div class="col-lg-9 pt8 pb8 o_colored_level">
                <h2 class="o_default_snippet_text">Happy to be Sponsor</h2>
                <p class="o_default_snippet_text">As a team, we are happy to contribute to this event.</p>
                <p class="o_default_snippet_text">Come see us live, we hope to meet you !</p>
                <p><a href="#" class="btn btn-primary o_default_snippet_text">Discover more</a></p>
            </div>
        </div>
    </div>
</section></div></field>
    </record>

    <!-- 15-16-28: children of 12 (Azure Interior) -->
    <record id="base.res_partner_address_15" model="res.partner">
        <field name="is_published" eval="True"/>
        <field name="website">http://azure.example.com</field>
        <field name="website_description" type="xml">
            <p>
                Brandon works in IT sector <b>since 10 years</b>. He is known
                notably for selling mouse traps. With that trick he cut
                IT budget by almost half within the last 2 years.
            </p>
        </field>
    </record>
    <record id="base.res_partner_address_16" model="res.partner">
        <field name="is_published" eval="True"/>
        <field name="website">http://azure.example.com</field>
        <field name="website_description" type="xml">
            <p>
                Nicole works in IT sector <b>since 20 years</b>. She
                develops software to help develop websites.  She sold her
                first company at 30 years old and manage to grow Azure Interior
                from 1 to 55 employees mostly by reselling services on
                Odoo.
            </p><p>
                Nicole is <b>author of several books</b>, including Amazon best seller
                "How Azure and Odoo will change the business world!".
            </p>
        </field>
    </record>
    <record id="base.res_partner_address_28" model="res.partner">
        <field name="is_published" eval="True"/>
        <field name="website">http://azure.example.com</field>
        <field name="website_description" type="xml">
            <p>
                Colleen Diaz works in IT sector <b>since 10 years</b>. He is known
                notably for selling mouse traps. With that trick he cut
                IT budget by almost half within the last 2 years.
            </p>
        </field>
    </record>

    <!-- 17-18: children of 10 (Jackson Group) -->
    <record id="base.res_partner_address_17" model="res.partner">
        <field name="is_published" eval="True"/>
        <field name="website">http://jackson.group.example.com</field>
        <field name="website_description" type="xml">
            <p>
                Toni Rhodes works in IT sector <b>since 10 years</b>. He is known
                notably for selling mouse traps. With that trick he cut
                IT budget by almost half within the last 2 years.
                Famous Managing Partner.
            </p>
        </field>
    </record>
    <record id="base.res_partner_address_18" model="res.partner">
        <field name="is_published" eval="True"/>
        <field name="website">http://jackson.group.example.com</field>
        <field name="website_description" type="xml">
            <p>
                Gordon Owens works in IT sector <b>since 10 years</b>. He is known
                notably for selling mouse traps. With that trick he cut
                IT budget by almost half within the last 2 years.
                Famous Senior Consultant.
            </p>
        </field>
    </record>

    <!-- 3-4-31: children of 2 (Deco Addict) -->
    <record id="base.res_partner_address_3" model="res.partner">
        <field name="is_published">True</field>
        <field name="website_short_description">Douglas Fletcher is a mighty functional consultant at Deco Addict</field>
        <field name="website_description" type="html">
<p>
    Douglas Fletcher works in IT sector <b>since 10 years</b>. He is known
    notably for selling mouse traps. With that trick he cut
    IT budget by almost half within the last 2 years.
</p></field>
    </record>
    <record id="base.res_partner_address_4" model="res.partner">
        <field name="is_published">True</field>
        <field name="website_short_description">Floyd Steward is a mighty analyst at Deco Addict.</field>
        <field name="website_description" type="html">
<p>
    Floyd Steward works in IT sector <b>since 10 years</b>. He is known
    notably for selling mouse traps. With that trick he cut
    IT budget by almost half within the last 2 years.
</p></field>
    </record>
    <record id="base.res_partner_address_31" model="res.partner">
        <field name="is_published">True</field>
        <field name="website_short_description">Addison Olson is a mighty sales representative at Deco Addict.</field>
        <field name="website_description" type="html">
<p>
    Addison Olson works in IT sector <b>since 10 years</b>. He is known
    notably for selling mouse traps. With that trick he cut
    IT budget by almost half within the last 2 years.
</p></field>
    </record>

</odoo>

```

## File: data\website_visitor_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="website_visitor_event_0" model="website.visitor">
        <field name="name">Edwin Hansen</field>
        <field name="partner_id" ref="base.res_partner_address_5"/>
        <field name="country_id" ref="base.us"/>
    </record>
    <record id="website_visitor_event_1" model="website.visitor">
        <field name="name">Soham Palmer</field>
        <field name="partner_id" ref="base.res_partner_address_11"/>
        <field name="country_id" ref="base.us"/>
    </record>
    <record id="website_visitor_event_2" model="website.visitor">
        <field name="name">Philipe J. Fry</field>
        <field name="partner_id" eval="False"/>
        <field name="country_id" ref="base.be"/>
    </record>
    <record id="website_visitor_event_2_1" model="website.visitor">
        <field name="name">Philipe J. Fry (old)</field>
        <field name="parent_id" ref="website_event.website_visitor_event_2"/>
        <field name="partner_id" eval="False"/>
        <field name="country_id" ref="base.be"/>
    </record>

</odoo>

```

## File: models\event_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval
from collections import defaultdict
from dateutil.relativedelta import relativedelta
import json
import werkzeug.urls

from pytz import utc

from odoo import api, fields, models, _
from odoo.addons.http_routing.models.ir_http import slug
from odoo.osv import expression
from odoo.tools.misc import get_lang, format_date

GOOGLE_CALENDAR_URL = 'https://www.google.com/calendar/render?'


class Event(models.Model):
    _name = 'event.event'
    _inherit = [
        'event.event',
        'website.seo.metadata',
        'website.published.multi.mixin',
        'website.cover_properties.mixin',
        'website.searchable.mixin',
    ]

    def _default_cover_properties(self):
        res = super()._default_cover_properties()
        res.update({
            'background-image': "url('/website_event/static/src/img/event_cover_4.jpg')",
            'opacity': '0.4',
            'resize_class': 'cover_auto'
        })
        return res

    # description
    subtitle = fields.Char('Event Subtitle', translate=True)
    # registration
    is_participating = fields.Boolean("Is Participating", compute="_compute_is_participating")
    # website
    website_published = fields.Boolean(tracking=True)
    website_menu = fields.Boolean(
        string='Website Menu',
        compute='_compute_website_menu', readonly=False, store=True,
        help="Allows to display and manage event-specific menus on website.")
    menu_id = fields.Many2one('website.menu', 'Event Menu', copy=False)
    menu_register_cta = fields.Boolean(
        'Extra Register Button', compute='_compute_menu_register_cta',
        readonly=False, store=True)
    # sub-menus management
    introduction_menu = fields.Boolean(
        "Introduction Menu", compute="_compute_website_menu_data",
        readonly=False, store=True)
    introduction_menu_ids = fields.One2many(
        "website.event.menu", "event_id", string="Introduction Menus",
        domain=[("menu_type", "=", "introduction")])
    location_menu = fields.Boolean(
        "Location Menu", compute="_compute_website_menu_data",
        readonly=False, store=True)
    location_menu_ids = fields.One2many(
        "website.event.menu", "event_id", string="Location Menus",
        domain=[("menu_type", "=", "location")])
    register_menu = fields.Boolean(
        "Register Menu", compute="_compute_website_menu_data",
        readonly=False, store=True)
    register_menu_ids = fields.One2many(
        "website.event.menu", "event_id", string="Register Menus",
        domain=[("menu_type", "=", "register")])
    community_menu = fields.Boolean(
        "Community Menu", compute="_compute_community_menu",
        readonly=False, store=True,
        help="Display community tab on website")
    community_menu_ids = fields.One2many(
        "website.event.menu", "event_id", string="Event Community Menus",
        domain=[("menu_type", "=", "community")])
    # live information
    is_ongoing = fields.Boolean(
        'Is Ongoing', compute='_compute_time_data', search='_search_is_ongoing',
        help="Whether event has begun")
    is_done = fields.Boolean(
        'Is Done', compute='_compute_time_data',
        help="Whether event is finished")
    start_today = fields.Boolean(
        'Start Today', compute='_compute_time_data',
        help="Whether event is going to start today if still not ongoing")
    start_remaining = fields.Integer(
        'Remaining before start', compute='_compute_time_data',
        help="Remaining time before event starts (minutes)")

    def _compute_is_participating(self):
        """Heuristic

          * public, no visitor: not participating as we have no information;
          * public and visitor: check visitor is linked to a registration. As
            visitors are merged on the top parent, current visitor check is
            sufficient even for successive visits;
          * logged, no visitor: check partner is linked to a registration. Do
            not check the email as it is not really secure;
          * logged as visitor: check partner or visitor are linked to a
            registration;
        """
        current_visitor = self.env['website.visitor']._get_visitor_from_request(force_create=False)
        if self.env.user._is_public() and not current_visitor:
            events = self.env['event.event']
        elif self.env.user._is_public():
            events = self.env['event.registration'].sudo().search([
                ('event_id', 'in', self.ids),
                ('state', '!=', 'cancel'),
                ('visitor_id', '=', current_visitor.id),
            ]).event_id
        else:
            if current_visitor:
                domain = [
                    '|',
                    ('partner_id', '=', self.env.user.partner_id.id),
                    ('visitor_id', '=', current_visitor.id)
                ]
            else:
                domain = [('partner_id', '=', self.env.user.partner_id.id)]
            events = self.env['event.registration'].sudo().search(
                expression.AND([
                    domain,
                    ['&', ('event_id', 'in', self.ids), ('state', '!=', 'cancel')]
                ])
            ).event_id

        for event in self:
            event.is_participating = event in events

    @api.depends('event_type_id')
    def _compute_website_menu(self):
        """ Also ensure a value for website_menu as it is a trigger notably for
        track related menus. """
        for event in self:
            if event.event_type_id and event.event_type_id != event._origin.event_type_id:
                event.website_menu = event.event_type_id.website_menu
            elif not event.website_menu:
                event.website_menu = False

    @api.depends("event_type_id", "website_menu", "community_menu")
    def _compute_community_menu(self):
        """ Set False in base module. Sub modules will add their own logic
        (meet or track_quiz). """
        for event in self:
            event.community_menu = False

    @api.depends("website_menu")
    def _compute_website_menu_data(self):
        """ Synchronize with website_menu at change and let people update them
        at will afterwards. """
        for event in self:
            event.introduction_menu = event.website_menu
            event.location_menu = event.website_menu
            event.register_menu = event.website_menu

    @api.depends("event_type_id", "website_menu")
    def _compute_menu_register_cta(self):
        """ At type onchange: synchronize. At website_menu update: synchronize. """
        for event in self:
            if event.event_type_id and event.event_type_id != event._origin.event_type_id:
                event.menu_register_cta = event.event_type_id.menu_register_cta
            elif event.website_menu and (event.website_menu != event._origin.website_menu or not event.menu_register_cta):
                event.menu_register_cta = True
            elif not event.website_menu:
                event.menu_register_cta = False

    @api.depends('date_begin', 'date_end')
    def _compute_time_data(self):
        """ Compute start and remaining time. Do everything in UTC as we compute only
        time deltas here. """
        now_utc = utc.localize(fields.Datetime.now().replace(microsecond=0))
        for event in self:
            date_begin_utc = utc.localize(event.date_begin, is_dst=False)
            date_end_utc = utc.localize(event.date_end, is_dst=False)
            event.is_ongoing = date_begin_utc <= now_utc <= date_end_utc
            event.is_done = now_utc > date_end_utc
            event.start_today = date_begin_utc.date() == now_utc.date()
            if date_begin_utc >= now_utc:
                td = date_begin_utc - now_utc
                event.start_remaining = int(td.total_seconds() / 60)
            else:
                event.start_remaining = 0

    @api.depends('name')
    def _compute_website_url(self):
        super(Event, self)._compute_website_url()
        for event in self:
            if event.id:  # avoid to perform a slug on a not yet saved record in case of an onchange.
                event.website_url = '/event/%s' % slug(event)

    # ------------------------------------------------------------
    # CRUD
    # ------------------------------------------------------------

    @api.model_create_multi
    def create(self, vals_list):
        events = super().create(vals_list)
        events._update_website_menus()
        return events

    def write(self, vals):
        menus_state_by_field = self._split_menus_state_by_field()
        res = super(Event, self).write(vals)
        menus_update_by_field = self._get_menus_update_by_field(menus_state_by_field, force_update=vals.keys())
        self._update_website_menus(menus_update_by_field=menus_update_by_field)
        return res

    # ------------------------------------------------------------
    # WEBSITE MENU MANAGEMENT
    # ------------------------------------------------------------

    def toggle_website_menu(self, val):
        self.website_menu = val

    def _get_menu_update_fields(self):
        """" Return a list of fields triggering a split of menu to activate /
        menu to de-activate. Due to saas-13.3 improvement of menu management
        this is done using side-methods to ease inheritance.

        :return list: list of fields, each of which triggering a menu update
          like website_menu, website_track, ... """
        return ['community_menu', 'introduction_menu', 'location_menu', 'register_menu']

    def _get_menu_type_field_matching(self):
        return {
            'community': 'community_menu',
            'introduction': 'introduction_menu',
            'location': 'location_menu',
            'register': 'register_menu',
        }

    def _split_menus_state_by_field(self):
        """ For each field linked to a menu, get the set of events having this
        menu activated and de-activated. Purpose is to find those whose value
        changed and update the underlying menus.

        :return dict: key = name of field triggering a website menu update, get {
          'activated': subset of self having its menu currently set to True
          'deactivated': subset of self having its menu currently set to False
        } """
        menus_state_by_field = dict()
        for fname in self._get_menu_update_fields():
            activated = self.filtered(lambda event: event[fname])
            menus_state_by_field[fname] = {
                'activated': activated,
                'deactivated': self - activated,
            }
        return menus_state_by_field

    def _get_menus_update_by_field(self, menus_state_by_field, force_update=None):
        """ For each field linked to a menu, get the set of events requiring
        this menu to be activated or de-activated based on previous recorded
        value.

        :param menus_state_by_field: see ``_split_menus_state_by_field``;
        :param force_update: list of field to which we force update of menus. This
          is used notably when a direct write to a stored editable field messes with
          its pre-computed value, notably in a transient mode (aka demo for example);

        :return dict: key = name of field triggering a website menu update, get {
          'activated': subset of self having its menu toggled to True
          'deactivated': subset of self having its menu toggled to False
        } """
        menus_update_by_field = dict()
        for fname in self._get_menu_update_fields():
            if fname in force_update:
                menus_update_by_field[fname] = self
            else:
                menus_update_by_field[fname] = self.env['event.event']
                menus_update_by_field[fname] |= menus_state_by_field[fname]['activated'].filtered(lambda event: not event[fname])
                menus_update_by_field[fname] |= menus_state_by_field[fname]['deactivated'].filtered(lambda event: event[fname])
        return menus_update_by_field

    def _get_website_menu_entries(self):
        """ Method returning menu entries to display on the website view of the
        event, possibly depending on some options in inheriting modules.

        Each menu entry is a tuple containing :
          * name: menu item name
          * url: if set, url to a route (do not use xml_id in that case);
          * xml_id: template linked to the page (do not use url in that case);
          * sequence: specific sequence of menu entry to be set on the menu;
          * menu_type: type of menu entry (used in inheriting modules to ease
            menu management; not used in this module in 13.3 due to technical
            limitations);
        """
        self.ensure_one()
        return [
            (_('Introduction'), False, 'website_event.template_intro', 1, 'introduction'),
            (_('Location'), False, 'website_event.template_location', 50, 'location'),
            (_('Register'), '/event/%s/register' % slug(self), False, 100, 'register'),
            (_('Community'), '/event/%s/community' % slug(self), False, 80, 'community'),
        ]

    def _update_website_menus(self, menus_update_by_field=None):
        """ Synchronize event configuration and its menu entries for frontend.

        :param menus_update_by_field: see ``_get_menus_update_by_field``"""
        for event in self:
            if event.menu_id and not event.website_menu:
                # do not rely on cascade, as it is done in SQL -> not calling override and
                # letting some ir.ui.views in DB
                (event.menu_id + event.menu_id.child_id).sudo().unlink()
            elif event.website_menu and not event.menu_id:
                root_menu = self.env['website.menu'].sudo().create({'name': event.name, 'website_id': event.website_id.id})
                event.menu_id = root_menu
            if event.menu_id and (not menus_update_by_field or event in menus_update_by_field.get('community_menu')):
                event._update_website_menu_entry('community_menu', 'community_menu_ids', 'community')
            if event.menu_id and (not menus_update_by_field or event in menus_update_by_field.get('introduction_menu')):
                event._update_website_menu_entry('introduction_menu', 'introduction_menu_ids', 'introduction')
            if event.menu_id and (not menus_update_by_field or event in menus_update_by_field.get('location_menu')):
                event._update_website_menu_entry('location_menu', 'location_menu_ids', 'location')
            if event.menu_id and (not menus_update_by_field or event in menus_update_by_field.get('register_menu')):
                event._update_website_menu_entry('register_menu', 'register_menu_ids', 'register')

    def _update_website_menu_entry(self, fname_bool, fname_o2m, fmenu_type):
        """ Generic method to create menu entries based on a flag on event. This
        method is a bit obscure, but is due to preparation of adding new menus
        entries and pages for event in a stable version, leading to some constraints
        while developing.

        :param fname_bool: field name (e.g. website_track)
        :param fname_o2m: o2m linking towards website.event.menu matching the
          boolean fields (normally an entry ot website.event.menu with type matching
          the boolean field name)
        :param method_name: method returning menu entries information: url, sequence, ...
        """
        self.ensure_one()
        new_menu = None

        menu_data = [menu_info for menu_info in self._get_website_menu_entries()
                     if menu_info[4] == fmenu_type]
        if self[fname_bool] and not self[fname_o2m]:
            # menus not found but boolean True: get menus to create
            for name, url, xml_id, menu_sequence, menu_type in menu_data:
                new_menu = self._create_menu(menu_sequence, name, url, xml_id, menu_type)
        elif not self[fname_bool]:
            # will cascade delete to the website.event.menu
            self[fname_o2m].mapped('menu_id').sudo().unlink()

        return new_menu

    def _create_menu(self, sequence, name, url, xml_id, menu_type):
        """ Create a new menu for the current event.

        If url: create a website menu. Menu leads directly to the URL that
        should be a valid route.

        If xml_id: create a new page using the qweb template given by its
        xml_id. Take its url back thanks to new_page of website, then link
        it to a menu. Template is duplicated and linked to a new url, meaning
        each menu will have its own copy of the template. This is currently
        limited to two menus: introduction and location.

        :param menu_type: type of menu. Mainly used for inheritance purpose
          allowing more fine-grain tuning of menus.
        """
        self.check_access_rights('write')
        view_id = False
        if not url:
            # add_menu=False, ispage=False -> simply create a new ir.ui.view with name
            # and template
            page_result = self.env['website'].sudo().new_page(
                name=name + ' ' + self.name, template=xml_id,
                add_menu=False, ispage=False)
            url = "/event/" + slug(self) + "/page" + page_result['url']  # url contains starting "/"
            view_id = page_result['view_id']

        website_menu = self.env['website.menu'].sudo().create({
            'name': name,
            'url': url,
            'parent_id': self.menu_id.id,
            'sequence': sequence,
            'website_id': self.website_id.id,
        })
        self.env['website.event.menu'].create({
            'menu_id': website_menu.id,
            'event_id': self.id,
            'menu_type': menu_type,
            'view_id': view_id,
        })
        return website_menu

    # ------------------------------------------------------------
    # TOOLS
    # ------------------------------------------------------------

    def google_map_link(self, zoom=8):
        """ Temporary method for stable """
        return self._google_map_link(zoom=zoom)

    def _google_map_link(self, zoom=8):
        self.ensure_one()
        if self.address_id:
            return self.sudo().address_id.google_map_link(zoom=zoom)
        return None

    def _track_subtype(self, init_values):
        self.ensure_one()
        if init_values.keys() & {'is_published', 'website_published'}:
            if self.is_published:
                return self.env.ref('website_event.mt_event_published', raise_if_not_found=False)
            return self.env.ref('website_event.mt_event_unpublished', raise_if_not_found=False)
        return super(Event, self)._track_subtype(init_values)

    def _get_event_resource_urls(self):
        url_date_start = self.date_begin.strftime('%Y%m%dT%H%M%SZ')
        url_date_stop = self.date_end.strftime('%Y%m%dT%H%M%SZ')
        params = {
            'action': 'TEMPLATE',
            'text': self.name,
            'dates': url_date_start + '/' + url_date_stop,
            'details': self.name,
        }
        if self.address_id:
            params.update(location=self.sudo().address_id.contact_address.replace('\n', ' '))
        encoded_params = werkzeug.urls.url_encode(params)
        google_url = GOOGLE_CALENDAR_URL + encoded_params
        iCal_url = '/event/%d/ics?%s' % (self.id, encoded_params)
        return {'google_url': google_url, 'iCal_url': iCal_url}

    def _default_website_meta(self):
        res = super(Event, self)._default_website_meta()
        event_cover_properties = json.loads(self.cover_properties)
        # background-image might contain single quotes eg `url('/my/url')`
        res['default_opengraph']['og:image'] = res['default_twitter']['twitter:image'] = event_cover_properties.get('background-image', 'none')[4:-1].strip("'")
        res['default_opengraph']['og:title'] = res['default_twitter']['twitter:title'] = self.name
        res['default_opengraph']['og:description'] = res['default_twitter']['twitter:description'] = self.subtitle
        res['default_twitter']['twitter:card'] = 'summary'
        res['default_meta_description'] = self.subtitle
        return res

    def get_backend_menu_id(self):
        return self.env.ref('event.event_main_menu').id

    @api.model
    def _search_build_dates(self):
        today = fields.Datetime.today()

        def sdn(date):
            return fields.Datetime.to_string(date.replace(hour=23, minute=59, second=59))

        def sd(date):
            return fields.Datetime.to_string(date)

        def get_month_filter_domain(filter_name, months_delta):
            first_day_of_the_month = today.replace(day=1)
            filter_string = _('This month') if months_delta == 0 \
                else format_date(self.env, value=today + relativedelta(months=months_delta),
                    date_format='LLLL', lang_code=get_lang(self.env).code).capitalize()
            return [filter_name, filter_string, [
                ("date_end", ">=", sd(first_day_of_the_month + relativedelta(months=months_delta))),
                ("date_begin", "<", sd(first_day_of_the_month + relativedelta(months=months_delta+1)))],
                0]

        return [
            ['all', _('Upcoming Events'), [("date_end", ">", sd(today))], 0],
            ['today', _('Today'), [
                ("date_end", ">", sd(today)),
                ("date_begin", "<", sdn(today))],
                0],
            get_month_filter_domain('month', 0),
            ['old', _('Past Events'), [
                ("date_end", "<", sd(today))],
                0],
        ]

    @api.model
    def _search_get_detail(self, website, order, options):
        with_description = options['displayDescription']
        with_date = options['displayDetail']
        date = options.get('date', 'all')
        country = options.get('country')
        tags = options.get('tags')
        event_type = options.get('type', 'all')

        domain = [website.website_domain()]
        if event_type != 'all':
            domain.append([("event_type_id", "=", int(event_type))])
        search_tags = self.env['event.tag']
        if tags:
            try:
                tag_ids = literal_eval(tags)
            except SyntaxError:
                pass
            else:
                # perform a search to filter on existing / valid tags implicitely + apply rules on color
                search_tags = self.env['event.tag'].search([('id', 'in', tag_ids)])

            # Example: You filter on age: 10-12 and activity: football.
            # Doing it this way allows to only get events who are tagged "age: 10-12" AND "activity: football".
            # Add another tag "age: 12-15" to the search and it would fetch the ones who are tagged:
            # ("age: 10-12" OR "age: 12-15") AND "activity: football
            grouped_tags = defaultdict(list)
            for tag in search_tags:
                grouped_tags[tag.category_id].append(tag)
            for group in grouped_tags:
                domain.append([('tag_ids', 'in', [tag.id for tag in grouped_tags[group]])])

        no_country_domain = domain.copy()
        if country:
            if country == 'online':
                domain.append([("country_id", "=", False)])
            elif country != 'all':
                domain.append(['|', ("country_id", "=", int(country)), ("country_id", "=", False)])

        no_date_domain = domain.copy()
        dates = self._search_build_dates()
        current_date = None
        for date_details in dates:
            if date == date_details[0]:
                domain.append(date_details[2])
                no_country_domain.append(date_details[2])
                if date_details[0] != 'all':
                    current_date = date_details[1]

        search_fields = ['name']
        fetch_fields = ['name', 'website_url']
        mapping = {
            'name': {'name': 'name', 'type': 'text', 'match': True},
            'website_url': {'name': 'website_url', 'type': 'text', 'truncate': False},
        }
        if with_description:
            search_fields.append('subtitle')
            fetch_fields.append('subtitle')
            mapping['description'] = {'name': 'subtitle', 'type': 'text', 'match': True}
        if with_date:
            mapping['detail'] = {'name': 'range', 'type': 'html'}
        return {
            'model': 'event.event',
            'base_domain': domain,
            'search_fields': search_fields,
            'fetch_fields': fetch_fields,
            'mapping': mapping,
            'icon': 'fa-ticket',
            # for website_event main controller:
            'dates': dates,
            'current_date': current_date,
            'search_tags': search_tags,
            'no_date_domain': no_date_domain,
            'no_country_domain': no_country_domain,
        }

    def _search_render_results(self, fetch_fields, mapping, icon, limit):
        with_date = 'detail' in mapping
        results_data = super()._search_render_results(fetch_fields, mapping, icon, limit)
        if with_date:
            for event, data in zip(self, results_data):
                begin = self.env['ir.qweb.field.date'].record_to_html(event, 'date_begin', {})
                end = self.env['ir.qweb.field.date'].record_to_html(event, 'date_end', {})
                data['range'] = '%s🠖%s' % (begin, end) if begin != end else begin
        return results_data

```

## File: models\event_registration.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class EventRegistration(models.Model):
    _name = 'event.registration'
    _inherit = ['event.registration']

    visitor_id = fields.Many2one('website.visitor', string='Visitor', ondelete='set null')

    def _get_website_registration_allowed_fields(self):
        return {'name', 'phone', 'email', 'mobile', 'event_id', 'partner_id', 'event_ticket_id'}

```

## File: models\event_tag_category.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class EventTagCategory(models.Model):
    _name = 'event.tag.category'
    _inherit = ['event.tag.category', 'website.published.mixin']

    def _default_is_published(self):
        return True

```

## File: models\event_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class EventType(models.Model):
    _name = 'event.type'
    _inherit = ['event.type']

    website_menu = fields.Boolean('Display a dedicated menu on Website')
    community_menu = fields.Boolean(
        "Community Menu", compute="_compute_community_menu",
        readonly=False, store=True,
        help="Display community tab on website")
    menu_register_cta = fields.Boolean(
        'Extra Register Button', compute='_compute_menu_register_cta',
        readonly=False, store=True)

    @api.depends('website_menu')
    def _compute_community_menu(self):
        for event_type in self:
            event_type.community_menu = event_type.website_menu

    @api.depends('website_menu')
    def _compute_menu_register_cta(self):
        for event_type in self:
            event_type.menu_register_cta = event_type.website_menu

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from odoo.addons.http_routing.models.ir_http import url_for

class Website(models.Model):
    _inherit = "website"

    def get_suggested_controllers(self):
        suggested_controllers = super(Website, self).get_suggested_controllers()
        suggested_controllers.append((_('Events'), url_for('/event'), 'website_event'))
        return suggested_controllers

    def get_cta_data(self, website_purpose, website_type):
        cta_data = super(Website, self).get_cta_data(website_purpose, website_type)
        if website_purpose == 'sell_more' and website_type == 'event':
            cta_btn_text = _('Next Events')
            return {'cta_btn_text': cta_btn_text, 'cta_btn_href': '/event'}
        return cta_data

    def _search_get_details(self, search_type, order, options):
        result = super()._search_get_details(search_type, order, options)
        if search_type in ['events', 'all']:
            result.append(self.env['event.event']._search_get_detail(self, order, options))
        return result

```

## File: models\website_event_menu.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class EventMenu(models.Model):
    _name = "website.event.menu"
    _description = "Website Event Menu"

    menu_id = fields.Many2one('website.menu', string='Menu', ondelete='cascade')
    event_id = fields.Many2one('event.event', string='Event', ondelete='cascade')
    view_id = fields.Many2one('ir.ui.view', string='View', ondelete='cascade', help='Used when not being an url based menu')
    menu_type = fields.Selection(
        [('community', 'Community Menu'),
         ('introduction', 'Introduction'),
         ('location', 'Location'),
         ('register', 'Register'),
        ], string="Menu Type", required=True)

    def unlink(self):
        self.view_id.sudo().unlink()
        return super(EventMenu, self).unlink()

```

## File: models\website_menu.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class WebsiteMenu(models.Model):
    _inherit = "website.menu"

    def unlink(self):
        """ Override to synchronize event configuration fields with menu deletion. """
        event_updates = {}
        website_event_menus = self.env['website.event.menu'].search([('menu_id', 'in', self.ids)])
        for event_menu in website_event_menus:
            to_update = event_updates.setdefault(event_menu.event_id, list())
            for menu_type, fname in event_menu.event_id._get_menu_type_field_matching().items():
                if event_menu.menu_type == menu_type:
                    to_update.append(fname)

        # manually remove website_event_menus to call their ``unlink`` method. Otherwise
        # super unlinks at db level and skip model-specific behavior.
        website_event_menus.unlink()
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


class WebsiteVisitor(models.Model):
    _name = 'website.visitor'
    _inherit = ['website.visitor']

    parent_id = fields.Many2one(
        'website.visitor', string="Parent", ondelete='set null',
        help="Main identity")
    event_registration_ids = fields.One2many(
        'event.registration', 'visitor_id', string='Event Registrations',
        groups="event.group_event_registration_desk")
    event_registration_count = fields.Integer(
        '# Registrations', compute='_compute_event_registration_count',
        groups="event.group_event_registration_desk")
    event_registered_ids = fields.Many2many(
        'event.event', string="Registered Events",
        compute="_compute_event_registered_ids", compute_sudo=True,
        search="_search_event_registered_ids",
        groups="event.group_event_registration_desk")

    @api.depends('event_registration_ids')
    def _compute_event_registration_count(self):
        if self.ids:
            read_group_res = self.env['event.registration'].read_group(
                [('visitor_id', 'in', self.ids)],
                ['visitor_id'], ['visitor_id'])
            visitor_mapping = dict(
                (item['visitor_id'][0], item['visitor_id_count'])
                for item in read_group_res)
        else:
            visitor_mapping = dict()
        for visitor in self:
            visitor.event_registration_count = visitor_mapping.get(visitor.id) or 0

    @api.depends('event_registration_ids.email', 'event_registration_ids.mobile', 'event_registration_ids.phone')
    def _compute_email_phone(self):
        super(WebsiteVisitor, self)._compute_email_phone()
        self.flush()

        for visitor in self.filtered(lambda visitor: not visitor.email or not visitor.mobile):
            linked_registrations = visitor.event_registration_ids.sorted(lambda reg: (reg.create_date, reg.id), reverse=False)
            if not visitor.email:
                visitor.email = next((reg.email for reg in linked_registrations if reg.email), False)
            if not visitor.mobile:
                visitor.mobile = next((reg.mobile or reg.phone for reg in linked_registrations if reg.mobile or reg.phone), False)

    @api.depends('parent_id', 'event_registration_ids')
    def _compute_event_registered_ids(self):
        # include parent's registrations in a visitor o2m field. We don't add
        # child one as child should not have registrations (moved to the parent)
        for visitor in self:
            all_registrations = visitor.event_registration_ids | visitor.parent_id.event_registration_ids
            visitor.event_registered_ids = all_registrations.mapped('event_id')

    def _search_event_registered_ids(self, operator, operand):
        """ Search visitors with terms on events within their event registrations. E.g. [('event_registered_ids',
        'in', [1, 2])] should return visitors having a registration on events 1, 2 as
        well as their children for notification purpose. """
        if operator == "not in":
            raise NotImplementedError("Unsupported 'Not In' operation on visitors registrations")

        all_registrations = self.env['event.registration'].sudo().search([
            ('event_id', operator, operand)
        ])
        if all_registrations:
            # search children, even archived one, to contact them
            visitors = all_registrations.with_context(active_test=False).mapped('visitor_id')
            children = self.env['website.visitor'].with_context(
                active_test=False
            ).sudo().search([('parent_id', 'in', visitors.ids)])
            visitor_ids = (visitors + children).ids
        else:
            visitor_ids = []

        return [('id', 'in', visitor_ids)]

    def _link_to_partner(self, partner, update_values=None):
        """ Propagate partner update to registration records """
        if partner:
            registration_wo_partner = self.event_registration_ids.filtered(lambda registration: not registration.partner_id)
            if registration_wo_partner:
                registration_wo_partner.partner_id = partner
        super(WebsiteVisitor, self)._link_to_partner(partner, update_values=update_values)

    def _link_to_visitor(self, target, keep_unique=True):
        """ Override linking process to link registrations to the final visitor. """
        self.event_registration_ids.write({'visitor_id': target.id})

        res = super(WebsiteVisitor, self)._link_to_visitor(target, keep_unique=False)

        if keep_unique:
            self.partner_id = False
            self.parent_id = target.id
            self.active = False

        return res

    def _get_visitor_from_request(self, force_create=False):
        """ When fetching visitor, now that duplicates are linked to a main visitor
        instead of unlinked, you may have more collisions issues with cookie being
        set after a de-connection for example.

        In base method, visitor associated to a partner in case of public user is
        not taken into account. It is considered as desynchronized cookie. Here
        we also discard if the visitor has a main visitor whose partner is set
        (aka wrong after logout partner). """
        visitor = super(WebsiteVisitor, self)._get_visitor_from_request(force_create=force_create)

        # also check that visitor parent partner is not different from user's one (indicates duplicate due to invalid or wrong cookie)
        if visitor and visitor.parent_id.partner_id:
            if self.env.user._is_public():
                visitor = self.env['website.visitor'].sudo()
            elif not visitor.partner_id:
                visitor = self.env['website.visitor'].sudo().with_context(active_test=False).search(
                    [('partner_id', '=', self.env.user.partner_id.id)]
                )

        if not visitor and force_create:
            visitor = self._create_visitor()

        return visitor

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_event
from . import event_registration
from . import event_tag_category
from . import event_type
from . import website
from . import website_event_menu
from . import website_menu
from . import website_visitor

```

## File: security\event_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
    <record id="event_event_public" model="ir.rule">
        <field name="name">Event: public/portal: published read</field>
        <field name="model_id" ref="event.model_event_event"/>
        <field name="domain_force">[('website_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

    <record id="ir_rule_event_tag_public" model="ir.rule">
        <field name="name">Event Tag: public/portal: color = published and category = published</field>
        <field name="model_id" ref="event.model_event_tag"/>
        <field name="domain_force">[('category_id.website_published', '=', True), ('color', '!=', False), ('color', '!=', 0)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

    <record id="ir_rule_event_event_ticket_public" model="ir.rule">
        <field name="name">Event Ticket: public/portal: published read</field>
        <field name="model_id" ref="event.model_event_event_ticket"/>
        <field name="domain_force">[('event_id.website_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>
    </data>

    <record id="event.group_event_manager" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('website.group_website_publisher'))]"/>
    </record>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_event_event,event.event,model_event_event,,1,0,0,0
access_event_event_ticket,event.event.ticket,event.model_event_event_ticket,,1,0,0,0
access_event_type,event.type,event.model_event_type,,0,0,0,0
access_event_tag_category,event.tag.category,event.model_event_tag_category,,1,0,0,0
access_event_tag,event.tag,event.model_event_tag,,1,0,0,0
access_website_event_menu,website.event.menu,model_website_event_menu,,1,0,0,0
access_website_event_menu_user,website.event.menu.user,model_website_event_menu,event.group_event_user,1,1,1,1
access_website_visitor_user,website.visitor.user,model_website_visitor,event.group_event_user,1,1,0,0

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#269396"/><stop offset="100%" stop-color="#218689"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M43.126 69H4c-2 0-4-.146-4-4.078v-23.83l15-16.955 8-4.078 8 2.039L39 17l26 23.451L43.126 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M32.51 22.527l-6.99 7.529.047-.334c.058-.081.058-.189 0-.323-.1-.215-.238-.3-.411-.258.043-.086.072-.211.086-.376.015-.165.022-.262.022-.29.043-.187.13-.352.26-.495a4.13 4.13 0 0 0 .357-.474c.05-.086.054-.129.01-.129.506.058.867-.021 1.084-.236.072-.072.155-.194.248-.366.094-.172.17-.294.228-.366.13-.086.23-.125.303-.118a.974.974 0 0 1 .314.118c.137.072.242.108.314.108.202.014.314-.065.335-.237a.465.465 0 0 0-.162-.43c.173.014.195-.108.065-.366a.823.823 0 0 0-.173-.193c-.173-.058-.368-.022-.585.107-.115.058-.1.115.044.172-.015-.014-.083.061-.206.226-.123.165-.242.29-.357.377-.116.086-.231.05-.347-.108a1.73 1.73 0 0 1-.119-.29c-.065-.18-.133-.276-.206-.29-.115 0-.23.107-.346.322.043-.115-.036-.222-.238-.323a1.564 1.564 0 0 0-.52-.172c.274-.172.217-.366-.173-.58-.101-.058-.249-.094-.444-.108-.195-.015-.335.014-.422.086a.577.577 0 0 0-.12.247c-.006.065.03.122.11.172.078.05.154.09.226.119.073.028.156.057.25.086.093.028.155.05.183.064.203.144.26.244.174.301-.03.015-.09.04-.184.076-.094.035-.177.068-.25.096-.072.03-.115.058-.13.086-.043.058-.043.158 0 .302.044.143.03.243-.043.3-.072-.07-.137-.196-.194-.376a1.367 1.367 0 0 0-.152-.355c.101.13-.08.172-.541.13l-.217-.022c-.058 0-.173.014-.346.043a1.619 1.619 0 0 1-.444.021.403.403 0 0 1-.292-.172c-.058-.114-.058-.258 0-.43.014-.057.043-.072.086-.043a3.238 3.238 0 0 1-.238-.204 2.11 2.11 0 0 0-.216-.183 12.32 12.32 0 0 0-2.036.882.536.536 0 0 0 .26-.022c.072-.028.166-.075.282-.14.115-.064.187-.103.216-.118.49-.2.794-.25.91-.15l.108-.108c.202.23.346.409.433.538-.101-.057-.318-.065-.65-.022-.288.086-.447.172-.476.259.101.172.137.3.108.387a3.375 3.375 0 0 1-.249-.215c-.108-.1-.213-.18-.314-.237a.868.868 0 0 0-.324-.108c-.231 0-.39.008-.477.022a13.627 13.627 0 0 0-5.088 4.776c.101.1.188.157.26.172.058.014.094.079.108.193a.82.82 0 0 0 .054.237c.022.043.105.022.25-.064.13.114.15.25.064.408.015-.014.332.18.953.581.274.244.426.394.455.452.043.158-.03.287-.217.387a1.103 1.103 0 0 0-.195-.194c-.115-.1-.18-.129-.195-.086-.043.072-.04.205.011.398.05.194.127.284.228.27-.101 0-.17.114-.206.344-.036.229-.054.484-.054.763 0 .28-.007.448-.022.506l.043.021c-.043.172-.003.42.12.742.122.323.277.463.465.42-.188.043-.043.351.433.925.087.115.144.18.173.194.044.028.13.082.26.16.13.08.238.151.325.216a.92.92 0 0 1 .216.226c.058.072.13.233.217.484.086.25.187.42.303.505-.029.087.04.23.206.43.166.202.241.366.227.496a.106.106 0 0 0-.054.021.106.106 0 0 1-.054.022c.043.1.155.2.335.3.18.101.293.194.336.28.014.043.029.115.043.216.015.1.036.179.065.236.029.058.087.072.173.043.03-.287-.144-.731-.52-1.334-.216-.358-.338-.566-.367-.623a1.231 1.231 0 0 1-.12-.334 1.633 1.633 0 0 0-.097-.312c.029 0 .072.01.13.032.058.022.12.047.184.076.065.028.12.057.162.086.044.028.058.05.044.064-.044.1-.03.226.043.377.072.15.159.283.26.398a29.568 29.568 0 0 0 .628.688c.086.086.187.226.303.42.115.193.115.29 0 .29.13 0 .274.072.433.215.159.144.281.287.368.43.072.115.13.302.173.56.043.258.08.43.108.516.03.1.09.197.184.29.094.094.185.162.271.205l.347.172.281.15c.072.03.206.105.4.226a2.8 2.8 0 0 0 .466.248c.144.057.26.086.346.086.087 0 .192-.018.314-.054.123-.036.22-.06.293-.075.216-.029.425.079.628.322.202.244.353.395.454.452.52.273.917.352 1.191.237-.029.014-.025.068.01.161.037.093.095.205.174.334a21.069 21.069 0 0 0 .314.494c.072.086.202.194.39.323.187.13.317.237.39.323.086-.058.137-.122.151-.194-.043.115.007.258.152.43.144.173.274.244.39.216.201-.043.302-.273.302-.689-.447.215-.8.086-1.06-.387a.418.418 0 0 0-.055-.118 1.367 1.367 0 0 1-.14-.366.303.303 0 0 1 0-.161c.014-.043.05-.065.108-.065.13 0 .202-.025.216-.075.015-.05 0-.14-.043-.27a4.316 4.316 0 0 1-.086-.279c-.015-.115-.094-.258-.239-.43a5.739 5.739 0 0 1-.26-.323c-.072.13-.187.187-.346.172-.159-.014-.274-.079-.346-.193a.59.59 0 0 1-.033.118.509.509 0 0 0-.032.14c-.188 0-.296-.007-.325-.022.014-.043.032-.168.054-.376s.047-.37.076-.484c.014-.057.054-.144.119-.258l.069-.125 3.432 3.26a.426.426 0 0 0-.07.07 1.065 1.065 0 0 0-.13.258c-.043.115-.079.194-.107.237-.03-.058-.112-.104-.25-.14-.137-.036-.205-.075-.205-.118.029.143.058.394.086.753.03.358.065.63.109.817.1.445.014.789-.26 1.033-.39.358-.6.645-.628.86-.058.316.029.502.26.56 0 .1-.058.247-.173.44-.116.194-.166.348-.152.463 0 .086.014.2.043.344 1.916-.332 3.649-1.013 5.199-2.042l2.057 1.954c-.42.297-.858.578-1.312.841-2.548 1.477-5.33 2.216-8.347 2.216-3.017 0-5.799-.739-8.346-2.216a16.5 16.5 0 0 1-6.052-6.013c-1.487-2.53-2.23-5.295-2.23-8.293 0-2.997.743-5.761 2.23-8.293a16.5 16.5 0 0 1 6.052-6.012c2.547-1.478 5.33-2.216 8.346-2.216 2.071 0 4.032.348 5.882 1.044zm-8.39 17.386l2.321 2.205a.67.67 0 0 0-.224.318 4.44 4.44 0 0 0-.065.226.726.726 0 0 1-.108.247.538.538 0 0 1-.195.15c-.101.044-.275.058-.52.044-.245-.014-.419-.05-.52-.108-.187-.114-.35-.322-.487-.624-.137-.3-.205-.566-.205-.796 0-.143.018-.333.054-.57.036-.236.057-.415.065-.537.004-.082-.035-.257-.12-.527l.004-.028zm17.614-12.441l14.71 13.245-7.995 8.879-14.71-13.245 7.995-8.88zm14.245 20.962a2.978 2.978 0 0 0 .208 4.206l-3.997 4.44a2.978 2.978 0 0 1-4.205.233L25.92 37.445a2.978 2.978 0 0 1-.208-4.206l3.997-4.44a2.978 2.978 0 0 0 4.205-.233 2.978 2.978 0 0 0-.207-4.206l3.997-4.44a2.978 2.978 0 0 1 4.205-.233l22.065 19.868a2.978 2.978 0 0 1 .208 4.206l-3.997 4.44a2.978 2.978 0 0 0-4.206.233zm2.233-6.774a1.489 1.489 0 0 0-.104-2.103L42.662 25.65a1.489 1.489 0 0 0-2.102.116l-8.661 9.62a1.489 1.489 0 0 0 .104 2.102l15.445 13.908c.61.548 1.55.496 2.103-.117l8.66-9.619z" opacity=".3"/><path fill="#FFF" d="M32.51 20.527l-6.99 7.529.047-.334c.058-.081.058-.189 0-.323-.1-.215-.238-.3-.411-.258.043-.086.072-.211.086-.376.015-.165.022-.262.022-.29.043-.187.13-.352.26-.495a4.13 4.13 0 0 0 .357-.474c.05-.086.054-.129.01-.129.506.058.867-.021 1.084-.236.072-.072.155-.194.248-.366.094-.172.17-.294.228-.366.13-.086.23-.125.303-.118a.974.974 0 0 1 .314.118c.137.072.242.108.314.108.202.014.314-.065.335-.237a.465.465 0 0 0-.162-.43c.173.014.195-.108.065-.366a.823.823 0 0 0-.173-.193c-.173-.058-.368-.022-.585.107-.115.058-.1.115.044.172-.015-.014-.083.061-.206.226-.123.165-.242.29-.357.377-.116.086-.231.05-.347-.108a1.73 1.73 0 0 1-.119-.29c-.065-.18-.133-.276-.206-.29-.115 0-.23.107-.346.322.043-.115-.036-.222-.238-.323a1.564 1.564 0 0 0-.52-.172c.274-.172.217-.366-.173-.58-.101-.058-.249-.094-.444-.108-.195-.015-.335.014-.422.086a.577.577 0 0 0-.12.247c-.006.065.03.122.11.172.078.05.154.09.226.119.073.028.156.057.25.086.093.028.155.05.183.064.203.144.26.244.174.301-.03.015-.09.04-.184.076-.094.035-.177.068-.25.096-.072.03-.115.058-.13.086-.043.058-.043.158 0 .302.044.143.03.243-.043.3-.072-.07-.137-.196-.194-.376a1.367 1.367 0 0 0-.152-.355c.101.13-.08.172-.541.13l-.217-.022c-.058 0-.173.014-.346.043a1.619 1.619 0 0 1-.444.021.403.403 0 0 1-.292-.172c-.058-.114-.058-.258 0-.43.014-.057.043-.072.086-.043a3.238 3.238 0 0 1-.238-.204 2.11 2.11 0 0 0-.216-.183 12.32 12.32 0 0 0-2.036.882.536.536 0 0 0 .26-.022c.072-.028.166-.075.282-.14.115-.064.187-.103.216-.118.49-.2.794-.25.91-.15l.108-.108c.202.23.346.409.433.538-.101-.057-.318-.065-.65-.022-.288.086-.447.172-.476.259.101.172.137.3.108.387a3.375 3.375 0 0 1-.249-.215c-.108-.1-.213-.18-.314-.237a.868.868 0 0 0-.324-.108c-.231 0-.39.008-.477.022a13.627 13.627 0 0 0-5.088 4.776c.101.1.188.157.26.172.058.014.094.079.108.193a.82.82 0 0 0 .054.237c.022.043.105.022.25-.064.13.114.15.25.064.408.015-.014.332.18.953.581.274.244.426.394.455.452.043.158-.03.287-.217.387a1.103 1.103 0 0 0-.195-.194c-.115-.1-.18-.129-.195-.086-.043.072-.04.205.011.398.05.194.127.284.228.27-.101 0-.17.114-.206.344-.036.229-.054.484-.054.763 0 .28-.007.448-.022.506l.043.021c-.043.172-.003.42.12.742.122.323.277.463.465.42-.188.043-.043.351.433.925.087.115.144.18.173.194.044.028.13.082.26.16.13.08.238.151.325.216a.92.92 0 0 1 .216.226c.058.072.13.233.217.484.086.25.187.42.303.505-.029.087.04.23.206.43.166.202.241.366.227.496a.106.106 0 0 0-.054.021.106.106 0 0 1-.054.022c.043.1.155.2.335.3.18.101.293.194.336.28.014.043.029.115.043.216.015.1.036.179.065.236.029.058.087.072.173.043.03-.287-.144-.731-.52-1.334-.216-.358-.338-.566-.367-.623a1.231 1.231 0 0 1-.12-.334 1.633 1.633 0 0 0-.097-.312c.029 0 .072.01.13.032.058.022.12.047.184.076.065.028.12.057.162.086.044.028.058.05.044.064-.044.1-.03.226.043.377.072.15.159.283.26.398a29.568 29.568 0 0 0 .628.688c.086.086.187.226.303.42.115.193.115.29 0 .29.13 0 .274.072.433.215.159.144.281.287.368.43.072.115.13.302.173.56.043.258.08.43.108.516.03.1.09.197.184.29.094.094.185.162.271.205l.347.172.281.15c.072.03.206.105.4.226a2.8 2.8 0 0 0 .466.248c.144.057.26.086.346.086.087 0 .192-.018.314-.054.123-.036.22-.06.293-.075.216-.029.425.079.628.322.202.244.353.395.454.452.52.273.917.352 1.191.237-.029.014-.025.068.01.161.037.093.095.205.174.334a21.069 21.069 0 0 0 .314.494c.072.086.202.194.39.323.187.13.317.237.39.323.086-.058.137-.122.151-.194-.043.115.007.258.152.43.144.173.274.244.39.216.201-.043.302-.273.302-.689-.447.215-.8.086-1.06-.387a.418.418 0 0 0-.055-.118 1.367 1.367 0 0 1-.14-.366.303.303 0 0 1 0-.161c.014-.043.05-.065.108-.065.13 0 .202-.025.216-.075.015-.05 0-.14-.043-.27a4.316 4.316 0 0 1-.086-.279c-.015-.115-.094-.258-.239-.43a5.739 5.739 0 0 1-.26-.323c-.072.13-.187.187-.346.172-.159-.014-.274-.079-.346-.193a.59.59 0 0 1-.033.118.509.509 0 0 0-.032.14c-.188 0-.296-.007-.325-.022.014-.043.032-.168.054-.376s.047-.37.076-.484c.014-.057.054-.144.119-.258l.069-.125 3.432 3.26a.426.426 0 0 0-.07.07 1.065 1.065 0 0 0-.13.258c-.043.115-.079.194-.107.237-.03-.058-.112-.104-.25-.14-.137-.036-.205-.075-.205-.118.029.143.058.394.086.753.03.358.065.63.109.817.1.445.014.789-.26 1.033-.39.358-.6.645-.628.86-.058.316.029.502.26.56 0 .1-.058.247-.173.44-.116.194-.166.348-.152.463 0 .086.014.2.043.344 1.916-.332 3.649-1.013 5.199-2.042l2.057 1.954c-.42.297-.858.578-1.312.841-2.548 1.477-5.33 2.216-8.347 2.216-3.017 0-5.799-.739-8.346-2.216a16.5 16.5 0 0 1-6.052-6.013c-1.487-2.53-2.23-5.295-2.23-8.293 0-2.997.743-5.761 2.23-8.293a16.5 16.5 0 0 1 6.052-6.012c2.547-1.478 5.33-2.216 8.346-2.216 2.071 0 4.032.348 5.882 1.044zm-8.39 17.386l2.321 2.205a.67.67 0 0 0-.224.318 4.44 4.44 0 0 0-.065.226.726.726 0 0 1-.108.247.538.538 0 0 1-.195.15c-.101.044-.275.058-.52.044-.245-.014-.419-.05-.52-.108-.187-.114-.35-.322-.487-.624-.137-.3-.205-.566-.205-.796 0-.143.018-.333.054-.57.036-.236.057-.415.065-.537.004-.082-.035-.257-.12-.527l.004-.028zm17.614-12.441l14.71 13.245-7.995 8.879-14.71-13.245 7.995-8.88zm14.245 20.962a2.978 2.978 0 0 0 .208 4.206l-3.997 4.44a2.978 2.978 0 0 1-4.205.233L25.92 35.445a2.978 2.978 0 0 1-.208-4.206l3.997-4.44a2.978 2.978 0 0 0 4.205-.233 2.978 2.978 0 0 0-.207-4.206l3.997-4.44a2.978 2.978 0 0 1 4.205-.233l22.065 19.868a2.978 2.978 0 0 1 .208 4.206l-3.997 4.44a2.978 2.978 0 0 0-4.206.233zm2.233-6.774a1.489 1.489 0 0 0-.104-2.103L42.662 23.65a1.489 1.489 0 0 0-2.102.116l-8.661 9.62a1.489 1.489 0 0 0 .104 2.102l15.445 13.908c.61.548 1.55.496 2.103-.117l8.66-9.619z"/></g></g></svg>
```

## File: static\src\img\snippets_thumbs\s_country_events.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <rect id="path-1" width="28" height="4" x="0" y="0"/>
    <filter id="filter-2" width="103.6%" height="150%" x="-1.8%" y="-12.5%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <rect id="path-3" width="28" height="4" x="0" y="5"/>
    <filter id="filter-4" width="103.6%" height="150%" x="-1.8%" y="-12.5%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <rect id="path-5" width="28" height="4" x="0" y="10"/>
    <filter id="filter-6" width="103.6%" height="150%" x="-1.8%" y="-12.5%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <linearGradient id="linearGradient-7" x1="50%" x2="50%" y1="0%" y2="100%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <path id="path-8" d="M7.429 3.279L9.72 5.571l-4.15 4.15-2.29-2.292 4.15-4.15zm-1.531 7.102l4.483-4.483a.446.446 0 0 0 .138-.327.446.446 0 0 0-.138-.326L7.755 2.619a.443.443 0 0 0-.326-.13.443.443 0 0 0-.327.13L2.62 7.102a.446.446 0 0 0-.138.327c0 .125.046.234.138.326l2.626 2.626c.087.087.196.13.326.13.131 0 .24-.043.327-.13zm6.45-4.621l-6.58 6.587a.893.893 0 0 1-.657.269.893.893 0 0 1-.657-.269l-.914-.914c.271-.27.406-.6.406-.987 0-.386-.135-.715-.406-.986-.27-.271-.6-.406-.986-.406-.387 0-.716.135-.987.406L.66 8.546a.893.893 0 0 1-.268-.657c0-.259.09-.477.268-.656L7.24.66a.893.893 0 0 1 .656-.268c.26 0 .478.09.657.268l.907.907c-.271.27-.406.6-.406.987 0 .386.135.715.406.986.27.271.6.406.986.406.387 0 .716-.135.987-.406l.914.907a.893.893 0 0 1 .269.657c0 .258-.09.477-.269.656z"/>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_country_events">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(18 23)">
        <g class="group_2" transform="translate(18)">
          <g class="rectangle">
            <use fill="#000" filter="url(#filter-2)" xlink:href="#path-1"/>
            <use fill="#FFF" fill-opacity=".95" xlink:href="#path-1"/>
          </g>
          <g class="rectangle">
            <use fill="#000" filter="url(#filter-4)" xlink:href="#path-3"/>
            <use fill="#FFF" fill-opacity=".95" xlink:href="#path-3"/>
          </g>
          <g class="rectangle">
            <use fill="#000" filter="url(#filter-6)" xlink:href="#path-5"/>
            <use fill="#FFF" fill-opacity=".95" xlink:href="#path-5"/>
          </g>
        </g>
        <mask id="mask-9" fill="#fff">
          <use xlink:href="#path-8"/>
        </mask>
        <use fill="url(#linearGradient-7)" class="ticket" xlink:href="#path-8"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_speaker_bio.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <rect id="path-1" width="28" height="2" x="0" y="0"/>
    <filter id="filter-2" width="103.6%" height="200%" x="-1.8%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <rect id="path-3" width="21" height="1" x="0" y="5"/>
    <filter id="filter-4" width="104.8%" height="300%" x="-2.4%" y="-50%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <rect id="path-5" width="24" height="1" x="0" y="8"/>
    <filter id="filter-6" width="104.2%" height="300%" x="-2.1%" y="-50%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <rect id="path-7" width="17" height="1" x="0" y="11"/>
    <filter id="filter-8" width="105.9%" height="300%" x="-2.9%" y="-50%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <linearGradient id="linearGradient-9" x1="50%" x2="50%" y1="0%" y2="100%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <path id="path-10" d="M9.758 10.149c0 .527-.15.98-.453 1.357-.302.377-.666.565-1.092.565H2.018c-.426 0-.79-.188-1.092-.565-.302-.378-.453-.83-.453-1.357 0-.411.02-.8.061-1.164.041-.365.118-.733.229-1.103.111-.37.253-.687.424-.95a2.03 2.03 0 0 1 .682-.646c.283-.167.608-.25.976-.25.633.619 1.39.928 2.27.928.88 0 1.638-.31 2.271-.928.368 0 .693.083.976.25.283.167.51.382.682.646.171.263.313.58.424.95.111.37.188.738.229 1.103.04.365.061.753.061 1.164zM7.901 3.714c0 .77-.272 1.426-.816 1.97-.544.544-1.2.816-1.97.816a2.684 2.684 0 0 1-1.97-.816 2.684 2.684 0 0 1-.815-1.97c0-.769.272-1.425.816-1.97A2.684 2.684 0 0 1 5.116.93c.768 0 1.425.272 1.97.816.543.544.815 1.2.815 1.97z"/>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_speaker_bio">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(21 24)">
        <g class="group_2" transform="translate(13 1)">
          <g class="rectangle">
            <use fill="#000" filter="url(#filter-2)" xlink:href="#path-1"/>
            <use fill="#FFF" fill-opacity=".95" xlink:href="#path-1"/>
          </g>
          <g class="rectangle">
            <use fill="#000" filter="url(#filter-4)" xlink:href="#path-3"/>
            <use fill="#FFF" fill-opacity=".348" xlink:href="#path-3"/>
          </g>
          <g class="rectangle">
            <use fill="#000" filter="url(#filter-6)" xlink:href="#path-5"/>
            <use fill="#FFF" fill-opacity=".348" xlink:href="#path-5"/>
          </g>
          <g class="rectangle">
            <use fill="#000" filter="url(#filter-8)" xlink:href="#path-7"/>
            <use fill="#FFF" fill-opacity=".348" xlink:href="#path-7"/>
          </g>
        </g>
        <mask id="mask-11" fill="#fff">
          <use xlink:href="#path-10"/>
        </mask>
        <use fill="url(#linearGradient-9)" class="user" xlink:href="#path-10"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\js\display_timer_widget.js

```javascript
odoo.define('website_event.display_timer_widget', function (require) {
'use strict';

var core = require('web.core');
var _t = core._t;
var publicWidget = require('web.public.widget');

publicWidget.registry.displayTimerWidget = publicWidget.Widget.extend({
    selector: '.o_display_timer',

    /**
     * This widget allows to display a dom element at the end of a certain time laps.
     * There are 2 timers available:
     *   - The main-timer: display the DOM element (using the displayClass) at the end of this timer.
     *   - The pre-timer: additional timer to display the main-timer. This pre-timer can be invisible or visible,
     *                    depending of the startCountdownDisplay option. Once the pre-timer is over,
                          the main-timer is displayed.
     * @override
     */
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self.options = self.$target.data();
            self.preCountdownDisplay = self.options["preCountdownDisplay"];
            self.preCountdownTime = self.options["preCountdownTime"];
            self.preCountdownText = self.options["preCountdownText"];

            self.mainCountdownTime = self.options["mainCountdownTime"];
            self.mainCountdownText = self.options["mainCountdownText"];
            self.mainCountdownDisplay = self.options["mainCountdownDisplay"];

            self.displayClass = self.options["displayClass"];

            if (self.preCountdownDisplay) {
                $(self.$el).parent().removeClass('d-none');
            }

            self._checkTimer();
            self.interval = setInterval(function () { self._checkTimer(); }, 1000);
        });
    },

    /**
     * This method removes 1 second to the current timer (pre-timer or main-timer)
     * and call the method to update the DOM, unless main-timer is over. In that last case,
     * the DOM element to show is displayed.
     *
     * @private
     */
    _checkTimer: function () {
        var now = new Date();

        var remainingPreSeconds = this.preCountdownTime - (now.getTime()/1000);
        if (remainingPreSeconds <= 1) {
            this.$('.o_countdown_text').text(this.mainCountdownText);
            if (this.mainCountdownDisplay) {
                $(this.$el).parent().removeClass('d-none');
            }
            var remainingMainSeconds = this.mainCountdownTime - (now.getTime()/1000);
            if (remainingMainSeconds <= 1) {
                clearInterval(this.interval);
                $(this.displayClass).removeClass('d-none');
                $(this.$el).parent().addClass('d-none');
            } else {
                this._updateCountdown(remainingMainSeconds);
            }
        } else {
            this._updateCountdown(remainingPreSeconds);
        }
    },

    /**
     * This method update the DOM to display the remaining time.
     * from seconds, the method extract the number of days, hours, minutes and seconds and
     * override the different DOM elements values.
     *
     * @private
     */
    _updateCountdown: function (remainingTime) {
        var remainingSeconds = remainingTime;
        var days = Math.floor(remainingSeconds / 86400);

        remainingSeconds = remainingSeconds % 86400;
        var hours = Math.floor(remainingSeconds / 3600);

        remainingSeconds = remainingSeconds % 3600;
        var minutes = Math.floor(remainingSeconds / 60);

        remainingSeconds = Math.floor(remainingSeconds % 60);

        this.$("span.o_timer_days").text(days);
        this.$("span.o_timer_hours").text(this._zeroPad(hours, 2));
        this.$("span.o_timer_minutes").text(this._zeroPad(minutes, 2));
        this.$("span.o_timer_seconds").text(this._zeroPad(remainingSeconds, 2));
    },

    /**
     * Small tool to add leading zéros to the given number, in function of the needed number of leading zéros.
     *
     * @private
     */
    _zeroPad: function (num, places) {
      var zero = places - num.toString().length + 1;
      return new Array(+(zero > 0 && zero)).join("0") + num;
    },

});

return publicWidget.registry.countdownWidget;

});

```

## File: static\src\js\register_toaster_widget.js

```javascript
odoo.define('website_event.register_toaster_widget', function (require) {
'use strict';

let core = require('web.core');
const {Markup} = require('web.utils');
let _t = core._t;
let publicWidget = require('web.public.widget');

publicWidget.registry.RegisterToasterWidget = publicWidget.Widget.extend({
    selector: '.o_wevent_register_toaster',

    /**
     * This widget allows to display a toast message on the page.
     *
     * @override
     */
    start: function () {
        const message = this.$el.data('message');
        if (message && message.length) {
            this.displayNotification({
                title: _t("Register"),
                message: Markup(message),
                type: 'info',
            });
        }
        return this._super.apply(this, arguments);
    },
});

return publicWidget.registry.RegisterToasterWidget;

});

```

## File: static\src\js\website_event.editor.js

```javascript
odoo.define('website_event.editor', function (require) {
"use strict";

var core = require('web.core');
var Dialog = require('web.Dialog');
var time = require('web.time');
var WebsiteNewMenu = require('website.newMenu');
var _t = core._t;

var EventCreateDialog = Dialog.extend({
    xmlDependencies: Dialog.prototype.xmlDependencies.concat(['/website_event/static/src/xml/event_create.xml']),
    template: 'website_event.event_create_dialog',
    events: _.extend({}, Dialog.prototype.events, {
        'change select[name="event_location"]': '_onLocationChanged',
        'click .input-group-append': '_onDatetimeClicked',
    }),
    jsLibs: [
        '/web/static/lib/daterangepicker/daterangepicker.js',
        '/web/static/src/legacy/js/libs/daterangepicker.js',
    ],
    

    /**
     * @override
     * @param {Object} parent
     * @param {Object} options
     */
    init: function (parent, options) {
        options = _.defaults(options || {}, {
            title: _t("New Event"),
            size: 'medium',
            buttons: [
                {
                    text: _t("Create"),
                    classes: 'btn-primary',
                    click: this._onClickCreate.bind(this),
                },
                {
                    text: _t("Discard"),
                    close: true
                },
            ]
        });
        this.eventStart = moment();
        this.eventEnd = moment().add(1, "d");
        this._super(parent, options);
    },

    /**
     * @override
     */
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(() => {
            // stat / end datetimes widgets
            self.$('input.daterange-input').each(function () {
                self._initDateRangePicker($(this));
            });
            // address select2 configuration
            self.$('input.o_wevent_js_address_id').select2(self._getSelect2AddressConfig());
        });
    },

    /**
     * @param {*} errors
     */
    _display_errors: function (errors) {
        this.$("p.text-danger").toggleClass('d-none', true);
        for (let i = 0; i < errors.length; i++) {
            this.$("#" + errors[i] + " p.text-danger").toggleClass('d-none', false);
        }
    },

    _getSelect2AddressConfig: function () {
        var self = this;
        return {
            allowClear: true,
            formatNoMatches: false,
            multiple: false,
            selection_data: false,
            width: '100%',

            createSearchChoice: function (name, data) {
                var addedPartner = $(this.opts.element).select2('data');
                if (_.filter(_.union(addedPartner, data), function (partner) {
                    return partner.text.toLowerCase().localeCompare(name.toLowerCase()) === 0;
                }).length === 0) {
                    return {
                        create: true,
                        id: _.uniqueId('address_'),
                        name: name,
                        text: _.str.sprintf(_t('Create "%s"'), name),
                    };
                }
            },
            fill_data: function (query, data) {
                var that = this;
                var partners = {results: []};
                _.each(data, function (partner) {
                    var contact_address = partner.contact_address.replace(/[\n]+/g, '').trim();
                    if (contact_address && that.matcher(query.term, contact_address)) {
                        partners.results.push({
                            id: partner.id,
                            text: contact_address,
                        });
                    }
                });
                query.callback(partners);
            },
            formatSelection: function (data) {
                if (data.name) {
                    data.text = data.name;
                }
                return data.text;
            },
            query: function (query) {
                var that = this;
                // fetch data only once and store it
                if (!this.selection_data) {
                    self._rpc({
                        model: 'res.partner',
                        method: 'search_read',
                        fields: ['contact_address'],
                        domain: [],
                    }).then(function (data) {
                        that.fill_data(query, data);
                        that.selection_data = data;
                    });
                } else {
                    this.fill_data(query, this.selection_data);
                }
            }
        }
    },

    _getSelect2AddressValues: function () {
        var select2Value = this.$('input.o_wevent_js_address_id').select2('data');
        if (select2Value) {
            if (select2Value.create) {
                return [0, {'name': select2Value.name}];
            } else {
                return [select2Value.id, {}];
            }
        }
        return [];
    },

    /**
     *
     * @param {*} $dateGroup
     */
    _initDateRangePicker: function ($dateGroup) {
        var minDate = moment().subtract(1, "d");
        var maxDate = moment().add(200, "y");
        var self = this;
        var textDirection = _t.database.parameters.direction;

        $dateGroup.daterangepicker({
            // dates
            endDate: this.eventEnd,
            maxDate: maxDate,
            minDate: minDate,
            startDate: this.eventStart,
            // display
            locale: {
                direction: textDirection,
                format: time.getLangDatetimeFormat().replace(':ss', ''),
                applyLabel: _t('Apply'),
                cancelLabel: _t('Cancel'),
                weekLabel: 'W',
                customRangeLabel: _t('Custom Range'),
                daysOfWeek: moment.weekdaysMin(),
                monthNames: moment.monthsShort(),
                firstDay: moment.localeData().firstDayOfWeek()
            },
            opens: 'left',
            timePicker: true,
            timePicker24Hour: true,
            viewDate: moment(new Date()).hours(minDate.hours()).minutes(minDate.minutes()).seconds(minDate.seconds()).milliseconds(minDate.milliseconds()),
        }, function (start, end, label) {
            self.eventStart = start;
            self.eventEnd = end;
        });
    },
    /**
     * @private
     */
    _prepareFormValues: function () {
        return {
            address_values: this._getSelect2AddressValues(),
            event_start: this.eventStart,
            event_end: this.eventEnd,
            location: this.$('select[name=event_location]').val(),
            name: this.$('input[name=name]').val().trim(),
        };
    },
    /**
     * @private
     * @param {*} values
     */
    _validateForm: function (values) {
        var errors = [];
        if (!values.name){
            errors.push('name');
        }
        if (values.location === 'on_site' && !values.address_values.length) {
            errors.push('address');
        }
        if (!values.event_start || !values.event_end) {
            errors.push('event_dates');
        }
        return errors;
    },

    /**
     * @private
     */
    _submitForm: function () {
        var self = this;
        var eventValues = this._prepareFormValues();
        var errors = this._validateForm(eventValues)
        console.log(eventValues, errors);
        if (errors.length) {
            this._display_errors(errors);
            return;
        }
        return this._rpc({
            route: '/event/add_event',
            params: eventValues,
        }).then((url) => {
            window.location.href = url;
            return new Promise(function () {});
        });
    },

    /**
     * @private
     */
    _onClickCreate: function () {
        return this._submitForm();
    },

    /**
     * @private
     * @param {*} ev
     */
    _onLocationChanged: function (ev) {
        this.$('.show_visibility_address').toggleClass('d-none', ev.target.value === 'online');
    },
});

WebsiteNewMenu.include({
    actions: _.extend({}, WebsiteNewMenu.prototype.actions || {}, {
        new_event: '_createNewEvent',
    }),

    //--------------------------------------------------------------------------
    // Actions
    //--------------------------------------------------------------------------

    /**
     * Asks the user information about a new event to create, then creates it
     * and redirects the user to this new event.
     *
     * @private
     * @returns {Promise} Unresolved if there is a redirection
     */
    _createNewEvent: function () {
        var self = this;
        var def = new Promise(function (resolve) {
            var dialog = new EventCreateDialog(self, {});
            dialog.open();
            dialog.on('closed', self, resolve);
        });
        return def;
    },
});
});

```

## File: static\src\js\website_event.js

```javascript
odoo.define('website_event.website_event', function (require) {

var ajax = require('web.ajax');
var core = require('web.core');
var Widget = require('web.Widget');
var publicWidget = require('web.public.widget');

var _t = core._t;

// Catch registration form event, because of JS for attendee details
var EventRegistrationForm = Widget.extend({

    /**
     * @override
     */
    start: function () {
        var self = this;
        var res = this._super.apply(this, arguments).then(function () {
            $('#registration_form .a-submit')
                .off('click')
                .click(function (ev) {
                    self.on_click(ev);
                })
                .prop('disabled', false);
        });
        return res;
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    on_click: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        var $form = $(ev.currentTarget).closest('form');
        var $button = $(ev.currentTarget).closest('[type="submit"]');
        var post = {};
        $('#registration_form table').siblings('.alert').remove();
        $('#registration_form select').each(function () {
            post[$(this).attr('name')] = $(this).val();
        });
        var tickets_ordered = _.some(_.map(post, function (value, key) { return parseInt(value); }));
        if (!tickets_ordered) {
            $('<div class="alert alert-info"/>')
                .text(_t('Please select at least one ticket.'))
                .insertAfter('#registration_form table');
            return new Promise(function () {});
        } else {
            $button.attr('disabled', true);
            var action = $form.data('action') || $form.attr('action');
            return ajax.jsonRpc(action, 'call', post).then(function (modal) {
                var $modal = $(modal);
                $modal.modal({backdrop: 'static', keyboard: false});
                $modal.find('.modal-body > div').removeClass('container'); // retrocompatibility - REMOVE ME in master / saas-19
                $modal.appendTo('body').modal();
                $modal.on('click', '.js_goto_event', function () {
                    $modal.modal('hide');
                    $button.prop('disabled', false);
                });
                $modal.on('click', '.close', function () {
                    $button.prop('disabled', false);
                });
            });
        }
    },
});

publicWidget.registry.EventRegistrationFormInstance = publicWidget.Widget.extend({
    selector: '#registration_form',

    /**
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);
        this.instance = new EventRegistrationForm(this);
        return Promise.all([def, this.instance.attachTo(this.$el)]);
    },
    /**
     * @override
     */
    destroy: function () {
        this.instance.setElement(null);
        this._super.apply(this, arguments);
        this.instance.setElement(this.$el);
    },
});

return EventRegistrationForm;
});

```

## File: static\src\js\website_event_set_customize_options.js

```javascript
odoo.define('website_event.set_customize_options', function (require) {
"use strict";

var CustomizeMenu = require('website.customizeMenu');
var publicWidget = require('web.public.widget');

var EventSpecificOptions = publicWidget.Widget.extend({
    template: 'website_event.customize_options',
    xmlDependencies: ['/website_event/static/src/xml/customize_options.xml'],
    events: {
        'change #display-website-menu': '_onDisplaySubmenuChange',
    },

    /**
     * @override
     */
    start: function () {
        this.$submenuInput = this.$('#display-website-menu');
        this.modelName = this._getEventObject().model;
        this.eventId = this._getEventObject().id;
        this._initCheckbox();
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    _onDisplaySubmenuChange: function (ev) {
        var checkboxValue = this.$submenuInput.is(':checked');
        this._toggleSubmenuDisplay(checkboxValue);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _getCheckboxFields: function () {
        return ['website_menu', 'website_url'];
    },

    _getCheckboxFieldMatch: function (checkboxField) {
        if (checkboxField === 'website_menu') {
            return this.$submenuInput;
        }
    },

    _getEventObject: function() {
        var repr = $('html').data('main-object');
        var m = repr.match(/(.+)\((\d+),(.*)\)/);
        return {
            model: m[1],
            id: m[2] | 0,
        };
    },

    _initCheckbox: function () {
        var self = this;
        this._rpc({
            model: this.modelName,
            method: 'read',
            args: [
                [this.eventId],
                this._getCheckboxFields()
            ],
        }).then((data) => {
            self._initCheckboxCallback(data);
        });
    },

    _initCheckboxCallback: function (rpcData) {
        if (rpcData[0]['website_menu']) {
            var submenuInput = this._getCheckboxFieldMatch('website_menu');
            submenuInput.attr('checked', 'checked');
        }
        this.eventUrl = rpcData[0]['website_url'];
    },

    _reloadEventPage: function () {
        window.location = this.eventUrl;
    },

    _toggleSubmenuDisplay: function (val) {
        var self = this;
        this._rpc({
            model: this.modelName,
            method: 'toggle_website_menu',
            args: [[this.eventId], val],
        }).then(function () {
            self._reloadEventPage();
        });
    },

});

CustomizeMenu.include({
    _getEventObject: function() {
        var repr = $('html').data('main-object');
        var m = repr.match(/(.+)\((\d+),(.*)\)/);
        return {
            model: m[1],
            id: m[2] | 0,
        };
    },

    _loadCustomizeOptions: function () {
        var self = this;
        var def = this._super.apply(this, arguments);
        return def.then(function () {
            if (!self.__eventOptionsLoaded && self._getEventObject().model === 'event.event') {
                self.__eventOptionsLoaded = true;
                self.eventOptions = new EventSpecificOptions(self);
                // If this is the first customize menu, add the divider at top
                if (!self.$('.dropdown-divider').length) {
                    self.$('.dropdown-menu').append($('<div/>', {
                        class: 'dropdown-divider',
                        role: 'separator',
                    }));
                }
                self.eventOptions.insertAfter(self.$el.find('.dropdown-divider:first()'));
            }
        });
    },
});

return {
    EventSpecificOptions: EventSpecificOptions,
};

});

```

## File: static\src\js\website_event_ticket_details.js

```javascript
odoo.define('website_event.ticket_details', function (require) {
    var publicWidget = require('web.public.widget');

    publicWidget.registry.ticketDetailsWidget = publicWidget.Widget.extend({
        selector: '.o_wevent_js_ticket_details',
        events: {
            'click .o_wevent_registration_btn': '_onTicketDetailsClick',
            'change .custom-select': '_onTicketQuantityChange'
        },
        start: function (){
            this.foldedByDefault = this.$el.data('foldedByDefault') === 1;
            return this._super.apply(this, arguments);
        },

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        /**
         * @private
         */
        _getTotalTicketCount: function (){
            var ticketCount = 0;
            this.$('.custom-select').each(function (){
                ticketCount += parseInt($(this).val());
            });
            return ticketCount;
        },

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------

        /**
         * When the "Fold Tickets Details" option is active, this will be called each
         * time the user expand or fold the tickets (o_wevent_registration_btn). This
         * allows to show/hide elements depending on the folding state.
         *
         * @private
         * @param {*} ev
         */
        _onTicketDetailsClick: function (ev){
            ev.preventDefault();
            if (this.foldedByDefault){
                let $target = $(ev.currentTarget);
                $target.toggleClass('btn-primary');
                $target.children().toggleClass('d-none');
                $target.siblings('.o_wevent_registration_title, .o_wevent_price_range').toggleClass('d-none');
            }
        },
        /**
         * @private
         */
        _onTicketQuantityChange: function (){
            this.$('button.btn-primary').attr('disabled', this._getTotalTicketCount() === 0);
        }
    });

return publicWidget.registry.ticketDetailsWidget;
});

```

## File: static\src\js\website_geolocation.js

```javascript
odoo.define('website_event.geolocation', function (require) {
'use strict';

var publicWidget = require('web.public.widget');

publicWidget.registry.visitor = publicWidget.Widget.extend({
    selector: ".oe_country_events, .country_events",
    disabledInEditableMode: false,

    /**
     * @override
     */
    start: function () {
        var defs = [this._super.apply(this, arguments)];
        var self = this;
        var $eventList = this.$('.country_events_list');
        this._originalContent = $eventList[0].outerHTML;
        defs.push(this._rpc({route: '/event/get_country_event_list'}).then(function (data) {
            if (data) {
                self._$loadedContent = $(data);
                self._$loadedContent.attr('contentEditable', false);
                $eventList.replaceWith(self._$loadedContent);
            }
        }));
        return Promise.all(defs);
    },
    /**
     * @override
     */
    destroy: function () {
        this._super.apply(this, arguments);
        if (this._$loadedContent) {
            this._$loadedContent.replaceWith(this._originalContent);
        }
    },
});
});

```

## File: static\src\js\tours\event_tour.js

```javascript
odoo.define('website_event.event_steps', function (require) {
"use strict";

const {_t} = require('web.core');
const {Markup} = require('web.utils');

var EventAdditionalTourSteps = require('event.event_steps');

EventAdditionalTourSteps.include({

    init: function() {
        this._super.apply(this, arguments);
    },

    _get_website_event_steps: function () {
        this._super.apply(this, arguments);
        return [{
                trigger: '.o_event_form_view button[name="is_published"]',
                content: Markup(_t("Use this <b>shortcut</b> to easily access your event web page.")),
                position: 'bottom',
            }, {
                trigger: 'li#edit-page-menu a',
                extra_trigger: '.o_wevent_event',
                content: Markup(_t("With the Edit button, you can <b>customize</b> the web page visitors will see when registering.")),
                position: 'bottom',
            }, {
                trigger: 'div[name="Image - Text"] .oe_snippet_thumbnail',
                extra_trigger: '.o_wevent_event',
                content: Markup(_t("<b>Drag and Drop</b> this snippet below the event title.")),
                position: 'bottom',
                run: 'drag_and_drop #o_wevent_event_main_col',
            }, {
                trigger: 'button[data-action="save"]',
                extra_trigger: '.o_wevent_event',
                content: Markup(_t("Don't forget to click <b>save</b> when you're done.")),
                position: 'bottom',
            }, {
                trigger: 'label.js_publish_btn',
                extra_trigger: '.o_wevent_event',
                content: Markup(_t("Looking great! Let's now <b>publish</b> this page so that it becomes <b>visible</b> on your website!")),
                position: 'bottom',
            }, {
                trigger: 'a.css_edit_dynamic',
                extra_trigger: '.js_publish_management[data-object="event.event"] .js_publish_btn .css_unpublish:visible',
                content: _t("Want to change your event configuration? Let's go back to the event form."),
                position: 'bottom',
                run: function (actions) {
                    actions.click('div.dropdown-menu a#edit-in-backend');
                },
            }, {
                trigger: 'a#edit-in-backend',
                extra_trigger: '.o_wevent_event',
                content: _t("This shortcut will bring you right back to the event form."),
                position: 'bottom'
            }];
    }
});

return EventAdditionalTourSteps;

});

```

## File: static\src\xml\customize_options.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

    <t t-name="website_event.customize_options" id="event_options">
        <li class="dropdown-header">Event Specific</li>
        <label class="o_switch dropdown-item m-0" name="display-website-menu" for="display-website-menu">
            <input id="display-website-menu" type="checkbox"/>
            <span/>
            Event Sub-menu
        </label>
    </t>

</templates>

```

## File: static\src\xml\event_create.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="website_event.event_create_dialog">
        <form id="editor_new_event" style="overflow:visible;">
            <div class="form-group row" id="name">
                <label for="name" class="col-md-2 col-form-label">Event Name</label>
                <div class="col-md-10">
                    <input type="text" name="name" class="form-control" required="required"/>
                </div>
                <p class="text-danger mt-1 mb-0 col-md-8 d-none">Please fill in this field</p>
            </div>
            <div class="form-group row">
                <label class="col-form-label col-md-2" for="event_location">Location</label>
                <div class="col-md-10">
                    <select class="form-control" id="event_location" name="event_location">
                        <option value="online">Online</option>
                        <option value="on_site">On Site</option>
                    </select>
                </div>
            </div>
            <div class="form-group row d-none show_visibility_address" id="address">
                <label class="col-form-label col-md-2" for="address_id">Venue</label>
                <div class="col-md-10">
                    <input type="text" class="form-control o_wevent_js_address_id" name="address_id" placeholder="Select Venue"/>
                </div>
                <p class="text-danger mt-1 mb-0 col-md-8 d-none">Please fill in this field</p>
            </div>
            <div class="form-group row d-flex justify-content-between align-items-center" id="event_dates">
                <label class="col-form-label col-md-2" for="event_start_end">Start &#8594; End</label>
                <div class="col-md-10 s_website_form_datetime input-group date o_wevent_form_datetime" data-target-input="nearest">
                    <input type="text" name="event_start_end" class="form-control daterange-input bg-transparent text-dark rounded-0 p-1"/>
                </div>
                <p class="text-danger mt-1 col-md-8 d-none">Please fill in this field</p>
            </div>
        </form>
    </t>
</templates>

```

## File: views\event_event_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="event_event_view_form" model="ir.ui.view">
        <field name="name">event.event.view.form.inherit.website</field>
        <field name="model">event.event</field>
        <field name="priority" eval="5"/>
        <field name="inherit_id" ref="event.view_event_form"/>
        <field name="arch" type="xml">
            <field name="company_id" position="after">
                <field name="website_id" options="{'no_create': True}" domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]" groups="website.group_multi_website"/>
            </field>
            <div name="button_box" position="inside">
                <field name="website_url" invisible="1"/>
                <field name="is_published" widget="website_redirect_button"/>
            </div>
            <xpath expr="//div[hasclass('oe_title')]" position="after">
                <div name="event_menu_configuration" groups="base.group_no_one">
                    <label for="website_menu" string="Website Submenu"/>
                    <field name="website_menu"/>
                    <!-- hidden sub-menus, they are triggered all at once based on "website_menu" -->
                    <field name="introduction_menu" invisible="1"/>
                    <field name="location_menu" invisible="1"/>
                    <field name="register_menu" invisible="1"/>
                    <label for="menu_register_cta" string="Extra Register Button"/>
                    <field name="menu_register_cta"/>
                    <label for="community_menu" string="Community" invisible="1"/>
                    <field name="community_menu" invisible="1"/>
                </div>
            </xpath>
        </field>
    </record>

    <record id="event_event_view_list" model="ir.ui.view">
        <field name="name">event.event.view.list.inherit.website</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="event.view_event_tree"/>
        <field name="arch" type="xml">
            <field name="company_id" position="after">
                <field name="website_id" groups="website.group_multi_website" domain="['|', ('company_id', '=', company_id), ('company_id', '=', False)]" optional="show"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\event_menus.xml

```xml
<?xml version="1.0"?>
<odoo><data>

    <menuitem id="menu_website_event_menu"
        name="Website Menus"
        action="website_event_menu_action"
        parent="event.menu_event_configuration"
        groups="base.group_no_one"
        sequence="99"/>

</data></odoo>

```

## File: views\event_registration_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>
    <record id="event_registration_action_from_visitor" model="ir.actions.act_window">
        <field name="name">Registrations</field>
        <field name="res_model">event.registration</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="domain">[('visitor_id', 'in', [active_id])]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No registration linked to this visitor
            </p>
        </field>
    </record>

    <record id="event_registration_view_form" model="ir.ui.view">
        <field name="name">event.registration.view.form.inherit.online</field>
        <field name="model">event.registration</field>
        <field name="inherit_id" ref="event.view_event_registration_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="visitor_id" groups="base.group_no_one"/>
            </xpath>
        </field>
    </record>

    <record id="event_registration_view_tree" model="ir.ui.view">
        <field name="name">event.registration.view.tree.inherit.online</field>
        <field name="model">event.registration</field>
        <field name="inherit_id" ref="event.view_event_registration_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="visitor_id" optional="hide" groups="base.group_no_one"/>
            </xpath>
        </field>
    </record>
</data></odoo>

```

## File: views\event_snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- Snippet - Country Events - Placeholder -->
<template id="s_country_events" name="Events">
    <div t-attf-class="s_country_events_list oe_country_events #{_classes}">
        <div class="country_events_list">
            <h6 class="o_wevent_sidebar_title">
                <i class="fa fa-globe mr-2"/>Upcoming Events
            </h6>
        </div>
    </div>
</template>

<!-- Snippets and options -->
<template id="snippets" inherit_id="website.snippets">
    <xpath expr="//t[@id='event_local_events_hook']" position="replace">
        <t t-snippet="website_event.s_country_events" t-thumbnail="/website_event/static/src/img/snippets_thumbs/s_country_events.svg"/>
    </xpath>
    <xpath expr="//t[@id='event_speaker_bio_hook']" position="replace">
        <t t-snippet="website_event.s_speaker_bio" t-thumbnail="/website_event/static/src/img/snippets_thumbs/s_speaker_bio.svg"/>
    </xpath>
</template>

<template id="snippet_options" inherit_id="website.snippet_options">
    <xpath expr="//*[@t-set='so_content_addition_selector']" position="inside">, .oe_country_events, .s_speaker_bio</xpath>
</template>

<template id="event_searchbar_input_snippet_options" inherit_id="website.searchbar_input_snippet_options" name="event search bar snippet options">
    <xpath expr="//div[@data-js='SearchBar']/we-select[@data-name='scope_opt']" position="inside">
        <we-button data-set-search-type="events" data-select-data-attribute="events" data-name="search_events_opt" data-form-action="/events">Events</we-button>
    </xpath>
    <xpath expr="//div[@data-js='SearchBar']/we-select[@data-name='order_opt']" position="inside">
        <we-button data-set-order-by="date_begin asc" data-select-data-attribute="date_begin asc" data-dependencies="search_events_opt" data-name="order_date_begin_asc_opt">Date (old to new)</we-button>
        <we-button data-set-order-by="date_end desc" data-select-data-attribute="date_end desc" data-dependencies="search_events_opt" data-name="order_date_end_desc_opt">Date (new to old)</we-button>
    </xpath>
    <xpath expr="//div[@data-js='SearchBar']/div[@data-dependencies='limit_opt']" position="inside">
        <we-checkbox string="Description" data-dependencies="search_events_opt" data-select-data-attribute="true" data-attribute-name="displayDescription"
            data-apply-to=".search-query"/>
        <we-checkbox string="Event Date" data-dependencies="search_events_opt" data-select-data-attribute="true" data-attribute-name="displayDetail"
            data-apply-to=".search-query"/>
    </xpath>
</template>

<!-- Snippet - Speaker Bio -->
<template id="s_speaker_bio" name="Speaker Bio">
    <div class="s_speaker_bio" itemscope="itemscope" itemtype="http://schema.org/Person" itemprop="performer">
        <span class="badge badge-secondary text-uppercase o_wevent_badge">Speaker</span>
        <img src="/website_event/static/src/img/speaker.png" width="70" class="img-fluid rounded-circle float-left mr-3" alt=""/>
        <div class="overflow-hidden">
            <h4 class="mt-3 mb-1" itemprop="name">John DOE</h4>
            <h6 class="mb-4">Company</h6>
            <p>At just 13 years old, John DOE was already starting to develop his first business applications for customers. After mastering civil engineering, he founded TinyERP. This was the first phase of OpenERP which would later became Odoo, the most installed open-source business software worldwide.</p>
        </div>
    </div>
</template>

</odoo>

```

## File: views\event_tag_category_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="event_tag_category_view_form" model="ir.ui.view">
        <field name="name">event.tag.category.view.form.inherit.website</field>
        <field name="model">event.tag.category</field>
        <field name="inherit_id" ref="event.event_tag_category_view_form"/>
        <field name="arch" type="xml">
            <field name="tag_ids" position="before">
                <field name="is_published" string="Show on Website" widget="boolean_toggle"/>
            </field>
        </field>
    </record>

    <record id="event_tag_category_view_tree" model="ir.ui.view">
        <field name="name">event.tag.category.view.tree.inherit.website</field>
        <field name="model">event.tag.category</field>
        <field name="inherit_id" ref="event.event_tag_category_view_tree"/>
        <field name="arch" type="xml">
            <field name="tag_ids" position="after">
                <field name="is_published" string="Show on Website"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\event_templates_list.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- Index -->
<template id="index" name="Events">
    <t t-call="website.layout">
        <div id="wrap" class="o_wevent_index">
            <!-- Options -->
            <t t-set="opt_events_list_cards" t-value="is_view_active('website_event.opt_events_list_cards')"/>
            <t t-set="opt_events_list_columns" t-value="is_view_active('website_event.opt_events_list_columns')"/>
            <!-- Topbar -->
            <t t-call="website_event.index_topbar">
                <t t-set="search" t-value="original_search or search or searches['search']"/>
            </t>
            <!-- Drag/Drop Area -->
            <div id="oe_structure_we_index_1" class="oe_structure oe_empty"/>
            <!-- Content -->
            <div t-attf-class="o_wevent_events_list #{opt_events_list_cards and 'opt_event_list_cards_bg'}">
                <div class="container">
                    <div class="row">
                        <t t-call="website_event.searched_tags"/>
                    </div>
                    <div class="row">
                        <div id="o_wevent_index_main_col" t-attf-class="col-md my-5 #{opt_events_list_columns and 'opt_events_list_columns' or 'opt_events_list_rows'}">
                            <div class="row">
                                <!-- Events List -->
                                <t t-call="website_event.events_list"/>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <!-- Drag/Drop Area -->
            <div id="oe_structure_we_index_2" class="oe_structure oe_empty"/>
        </div>
    </t>
</template>

<!-- Index - OPTION - Sidebar -->
<template id="opt_index_sidebar" inherit_id="website_event.index" active="False" customize_show="True" name="Show Sidebar">
    <xpath expr="//div[@id='o_wevent_index_main_col']" position="after">
        <t t-call="website_event.index_sidebar"/>
    </xpath>
</template>

<!-- Index Topbar -->
<template id="index_topbar" name="Topbar">
    <nav class="navbar navbar-light border-top shadow-sm d-print-none">
        <div class="container">
            <div class="d-flex flex-column flex-sm-row justify-content-between w-100">
                <span class="navbar-brand h4 my-0 mr-auto">Events</span>
                <ul class="o_wevent_index_topbar_filters nav">
                    <t t-foreach="categories" t-as="category">
                        <li t-if="category.is_published and category.tag_ids and any(tag.color for tag in category.tag_ids)" class="nav-item dropdown mr-2 my-1">
                            <a href="#" role="button" class="btn dropdown-toggle" data-toggle="dropdown">
                                <i class="fa fa-folder-open"/>
                                <t t-esc="category.name"/>
                            </a>
                            <div class="dropdown-menu">
                                <t t-foreach="category.tag_ids" t-as="tag">
                                    <a t-if="tag.color"
                                        t-att-href="'/event?%s' % keep_query('*', tags=str((search_tags - tag).ids if tag in search_tags else (tag | search_tags).ids))"
                                        t-attf-class="dropdown-item d-flex align-items-center justify-content-between #{'active' if tag in search_tags else ''}">
                                        <t t-esc="tag.name"/>
                                    </a>
                                </t>
                            </div>
                        </li>
                    </t>
                </ul>
                <div class="d-flex align-items-center flex-wrap pl-sm-3 pr-0">
                    <t t-call="website_event.events_search_box_input"/>
                </div>
            </div>
        </div>
    </nav>
</template>

<template id="searched_tags" name="Searched tags">
    <div class="d-flex align-items-center mt16">
        <t t-foreach="search_tags" t-as="tag">
            <span class="align-items-baseline border d-inline-flex pl-2 rounded ml16 mb-2 bg-white">
                <i class="fa fa-tag mr-2 text-muted"/>
                <t t-esc="tag.display_name"/>
                <a t-att-href="'/event?%s' % keep_query('*', tags=str((search_tags - tag).ids))" class="btn border-0 py-1">&#215;</a>
            </span>
        </t>
    </div>
</template>

<!-- Filter - Date -->
<template id="event_time" inherit_id="website_event.index_topbar" customize_show="True" name="Filter by Date">
    <xpath expr="//ul[hasclass('o_wevent_index_topbar_filters')]" position="inside">
        <li class="nav-item dropdown mr-2 my-1">
            <a href="#" role="button" class="btn dropdown-toggle" data-toggle="dropdown">
                <i class="fa fa-calendar"/>
                <t t-if="current_date" t-esc="current_date"/>
                <t t-else="">Upcoming Events</t>
            </a>
            <div class="dropdown-menu">
                <t t-foreach="dates" t-as="date">
                    <t t-if="date[3] or (date[0] in ('old','all'))">
                        <a t-att-href="keep('/event', date=date[0])" t-attf-class="dropdown-item d-flex align-items-center justify-content-between #{searches.get('date') == date[0] and 'active'}">
                            <t t-esc="date[1]"/>
                            <span t-if="date[3]" t-esc="date[3]" class="badge badge-pill badge-primary ml-3"/>
                        </a>
                    </t>
                </t>
            </div>
        </li>
    </xpath>
</template>

<!-- Filter - Location -->
<template id="event_location" inherit_id="website_event.index_topbar" active="False" customize_show="True" name="Filter by Country">
    <xpath expr="//ul[hasclass('o_wevent_index_topbar_filters')]" position="inside">
        <li class="nav-item dropdown mr-2 my-1">
            <a href="#" role="button" class="btn dropdown-toggle" data-toggle="dropdown">
                <i class="fa fa-map-marker"/>
                <t t-if="searches['country'] == 'online'">
                    <t t-set="online_event_text">Online Events</t>
                    <t t-out="online_event_text"/>
                </t>
                <t t-elif="current_country" t-esc="current_country.name"/>
                <t t-else="">All countries</t>
            </a>
            <div class="dropdown-menu">
                <t t-foreach="countries" t-as="country">
                    <t t-if="country['country_id']">
                        <a t-att-href="keep('/event', country=country['country_id'][0])" t-attf-class="dropdown-item d-flex align-items-center justify-content-between #{searches.get('country') == str(country['country_id'] and country['country_id'][0]) and 'active'}">
                            <t t-esc="country['country_id'][1]"/>
                            <span t-esc="country['country_id_count']" class="badge badge-pill badge-primary ml-auto"/>
                        </a>
                    </t>
                    <t t-else="">
                        <a t-att-href="keep('/event', country='online')" t-attf-class="dropdown-item d-flex align-items-center justify-content-between #{searches.get('country') == 'online' and 'active'}">
                            <span>Online Events</span>
                            <span t-esc="country['country_id_count']" class="badge badge-pill badge-primary ml-3"/>
                        </a>
                    </t>
                </t>
            </div>
        </li>
    </xpath>
</template>

<!-- Index - Events list -->
<template id="events_list" name="Events list">
    <!-- Options -->
    <t t-set="opt_index_sidebar" t-value="is_view_active('website_event.opt_index_sidebar')"/>
    <t t-if="opt_events_list_columns" t-set="opt_event_size" t-value="opt_index_sidebar and 'col-md-6' or 'col-md-6 col-lg-4'"/>
    <t t-else="" t-set="opt_event_size" t-value="opt_index_sidebar and 'col-12' or 'col-xl-10 offset-xl-1'"/>
    <!-- No events -->
    <t t-if="not event_ids">
        <div class="col-12">
            <div class="h2 mb-3">No events found.</div>
            <div class="alert alert-info text-center" groups="event.group_event_user">
                <p class="m-0">Use the top button '<b>+ New</b>' to create an event.</p>
            </div>
        </div>
    </t>
    <!-- Fuzzy search -->
    <div t-if="event_ids and original_search" class="col-12 alert alert-warning mt8">
        No results found for '<span t-esc="original_search"/>'. Showing results for '<span t-esc="searches['search']"/>'.
    </div>
    <!-- List -->
    <div t-foreach="event_ids" t-as="event" t-attf-class=" #{opt_event_size} mb-4">
        <a t-attf-href="/event/#{ slug(event) }/#{(not event.menu_id) and 'register'}" class="text-decoration-none" t-att-data-publish="event.website_published and 'on' or 'off'">
            <article t-attf-class="h-100 #{opt_events_list_cards and 'card border-0 shadow-sm'}" itemscope="itemscope" itemtype="http://schema.org/Event">
                <div class="h-100 row no-gutters">
                    <!-- Header -->
                    <header t-attf-class="overflow-hidden bg-secondary #{opt_events_list_columns and 'col-12' or 'col-sm-4 col-lg-3'} #{(not opt_events_list_cards) and 'shadow'}">
                        <!-- Image + Link -->
                        <div class="d-block h-100 w-100">
                            <t t-call="website.record_cover">
                                <t t-set="_record" t-value="event"/>

                                <!-- Short Date -->
                                <div class="o_wevent_event_date position-absolute bg-white shadow-sm text-dark">
                                    <span t-field="event.with_context(tz=event.date_tz).date_begin" t-options="{'format': 'LLL'}" class="o_wevent_event_month"/>
                                    <span t-field="event.with_context(tz=event.date_tz).date_begin" t-options="{'format': 'dd'}" class="o_wevent_event_day oe_hide_on_date_edit"/>
                                </div>
                                <!-- Participating -->
                                <small t-if="event.is_participating" class="o_wevent_participating bg-success">
                                    <i class="fa fa-check mr-2"/>Registered
                                </small>
                                <!-- Unpublished -->
                                <small t-if="not event.website_published" class="o_wevent_unpublished bg-danger">
                                    <i class="fa fa-ban mr-2"/>Unpublished
                                </small>
                            </t>
                        </div>
                    </header>
                    <div t-att-class="'%s %s' % (
                        opt_events_list_columns and 'col-12' or 'col',
                        opt_events_list_columns and event.event_registrations_open and not event.event_registrations_sold_out and 'h-100' or '')">
                        <!-- Body -->
                        <main t-attf-class="#{opt_events_list_cards and 'card-body' or (opt_events_list_columns and 'py-3' or 'px-4')}">
                            <!-- Title -->
                            <h5 t-attf-class="card-title mt-2 mb-0 text-truncate #{(not event.website_published) and 'text-danger'}">
                                <span t-field="event.name" itemprop="name"/>
                            </h5>
                            <!-- Start Date & Time -->
                            <time itemprop="startDate" t-att-datetime="event.date_begin">
                                <span t-field="event.with_context(tz=event.date_tz).date_begin" t-options="{'date_only': 'true', 'format': 'long'}"/> -
                                <span t-field="event.with_context(tz=event.date_tz).date_begin" t-options="{'time_only': 'true', 'format': 'short'}" class="oe_hide_on_date_edit"/>
                            </time>
                            <!-- Location -->
                            <div itemprop="location" t-field="event.address_id" t-options="{'widget': 'contact', 'fields': ['city'], 'no_marker': 'true'}"/>
                            <div class="mt8">
                                <t t-foreach="event.tag_ids.filtered(lambda tag: tag.category_id.is_published)" t-as="tag">
                                    <span t-if="tag.color"
                                        t-attf-class="badge mr4 #{'badge-primary' if tag in search_tags else 'badge-light'} #{'o_tag_color_%s' % tag.color if tag.color else ''}">
                                        <span t-esc="tag.name"/>
                                    </span>
                                </t>
                            </div>
                        </main>
                    </div>
                    <!-- Footer -->
                    <footer t-if="not event.event_registrations_open or event.event_registrations_sold_out"
                        t-att-class="'small align-self-end w-100 %s %s' % (
                            opt_events_list_cards and 'card-footer' or (not opt_events_list_columns and 'mx-4 mt-auto pt-2') or 'py-2',
                            opt_events_list_cards and 'border-top' or '',
                        )">
                        <span t-if="not event.event_registrations_open" class="text-danger">
                            <t t-if="not event.event_registrations_started">
                                Registrations not yet open
                            </t>
                            <t t-elif="event.event_registrations_sold_out">
                                Sold Out
                            </t>
                            <t t-else="">
                                Registrations Closed
                            </t>
                        </span>
                    </footer>
                </div>
            </article>
        </a>
    </div>
    <!-- Pager -->
    <div class="form-inline justify-content-center my-3">
        <t t-call="website.pager"/>
    </div>
</template>

<template id="opt_events_list_columns" inherit_id="website_event.events_list" active="True" customize_show="True" name="Layout • Columns"/>

<template id="opt_events_list_cards" inherit_id="website_event.events_list" active="True" customize_show="True" name="'Cards' Design"/>

<template id="opt_events_list_categories" inherit_id="website_event.events_list" active="False" customize_show="True" name="Show Templates">
    <xpath expr="//main/*" position="before">
        <span t-if="event.event_type_id" t-attf-href="/event?type=#{event.event_type_id.id}" t-attf-class="badge bg-secondary o_wevent_badge #{opt_events_list_columns and 'o_wevent_badge_event' or 'float-right'}" t-field="event.event_type_id"/>
    </xpath>
</template>

<!-- Index - Sidebar -->
<template id="index_sidebar" name="Sidebar">
    <div id="o_wevent_index_sidebar" class="col-lg-4 ml-lg-3 ml-xl-5 my-5"/>
</template>

<!-- Index - Sidebar - About us -->
<template id="index_sidebar_about_us" inherit_id="website_event.index_sidebar" active="True" customize_show="True" name="About us" priority="20">
    <xpath expr="//div[@id='o_wevent_index_sidebar']" position="inside">
        <div class="o_wevent_sidebar_block">
            <h6 class="o_wevent_sidebar_title">About us</h6>
            <p>Use this paragrah to write a short text about your events or company.</p>
        </div>
        <div id="oe_structure_website_event_about_us_1" class="oe_structure"/>
    </xpath>
</template>

<!-- Index - Sidebar - Follow us -->
<template id="index_sidebar_follow_us" inherit_id="website_event.index_sidebar" active="True" customize_show="True" name="Follow us" priority="30">
    <xpath expr="//div[@id='o_wevent_index_sidebar']" position="inside">
        <div class="o_wevent_sidebar_block">
            <h6 class="o_wevent_sidebar_title">Follow Us</h6>
            <div class="o_wevent_sidebar_social mx-n1">
                <a t-if="website.social_facebook" t-att-href="website.social_facebook" class="o_wevent_social_link"><i class="fa fa-facebook text-facebook" aria-label="Facebook" title="Facebook"/></a>
                <a t-if="website.social_twitter" t-att-href="website.social_twitter" class="o_wevent_social_link"><i class="fa fa-twitter text-twitter" aria-label="Twitter" title="Twitter"/></a>
                <a t-if="website.social_linkedin" t-att-href="website.social_linkedin" class="o_wevent_social_link"><i class="fa fa-linkedin text-linkedin" aria-label="LinkedIn" title="LinkedIn"/></a>
                <a t-if="website.social_youtube" t-att-href="website.social_youtube" class="o_wevent_social_link"><i class="fa fa-youtube-play text-youtube" aria-label="Youtube" title="Youtube"/></a>
                <a t-if="website.social_github" t-att-href="website.social_github" class="o_wevent_social_link"><i class="fa fa-github text-github" aria-label="Github" title="Github"/></a>
                <a t-if="website.social_instagram" t-att-href="website.social_instagram" class="o_wevent_social_link"><i class="fa fa-instagram text-instagram" aria-label="Instagram" title="Instagram"/></a>
            </div>
        </div>
        <div id="oe_structure_website_event_follow_us_1" class="oe_structure"/>
    </xpath>
</template>

<!-- Index - Sidebar - Photos -->
<template id="index_sidebar_photos" inherit_id="website_event.index_sidebar" active="True" customize_show="True" name="Photos" priority="40">
    <xpath expr="//div[@id='o_wevent_index_sidebar']" position="inside">
        <h6 class="o_wevent_sidebar_title">Photos</h6>
        <a href="/event">
            <figure class="o_wevent_sidebar_block o_wevent_sidebar_figure figure">
                <img class="figure-img img-fluid rounded" src="/website_event/static/src/img/event_past_0.jpg" alt=""/>
                <figcaption class="figure-caption text-muted">A past event</figcaption>
            </figure>
        </a>
        <a href="/event">
            <figure class="o_wevent_sidebar_block o_wevent_sidebar_figure figure">
                <img class="figure-img img-fluid rounded" src="/website_event/static/src/img/event_training_0.jpg" alt=""/>
                <figcaption class="figure-caption text-muted">Our Trainings</figcaption>
            </figure>
        </a>
    </xpath>
</template>

<!-- Index - Sidebar - Quotes -->
<template id="index_sidebar_quotes" inherit_id="website_event.index_sidebar" active="True" customize_show="True" name="Quotes" priority="60">
    <xpath expr="//div[@id='o_wevent_index_sidebar']" position="inside">
        <div class="o_wevent_sidebar_block card">
            <div class="card-body">
                <blockquote class="blockquote mb-0">
                    <p><em>Write here a quote from one of your attendees. It gives confidence in your events.</em></p>
                    <footer class="blockquote-footer text-muted">Author</footer>
                </blockquote>
            </div>
        </div>
    </xpath>
</template>

<!-- Index - Sidebar - Snippet - Country Events -->
<template id="index_sidebar_country_event" inherit_id="website_event.index_sidebar" active="False" customize_show="True" name="Country Events" priority="70">
    <xpath expr="//div[@id='o_wevent_index_sidebar']" position="inside">
        <div class="o_wevent_sidebar_block">
            <t t-snippet-call="website_event.s_country_events"/>
        </div>
    </xpath>
</template>

</odoo>

```

## File: views\event_templates_page.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- Event -->
<template id="layout" name="Event">
    <t t-call="website.layout">
        <!-- Options -->
        <t t-set="opt_events_list_categories" t-value="is_view_active('website_event.opt_events_list_categories')"/>
        <div id="wrap" t-attf-class="o_wevent_event js_event #{'o_wevent_hide_sponsors' if hide_sponsors else ''}">
            <t t-if="not event.menu_id">
                <nav class="navbar navbar-light border-top shadow-sm d-print-none">
                    <div class="container align-items-baseline justify-content-start">
                        <a href="/event" class="navbar-brand h4 my-0 mr-0 mr-md-4">
                            <i class="fa fa-long-arrow-left text-primary mr-2"/>
                            <span>All Events</span>
                        </a>
                        <ul class="navbar-nav flex-row ml-md-auto ml-0">
                            <li t-if="opt_events_list_categories" class="nav-item mr-3">
                                <a t-attf-href="/event?type=#{event.event_type_id.id}" t-if="event.event_type_id" class="nav-link">
                                    <i class="fa fa-folder-open text-primary mr-2"/><span t-field="event.event_type_id"/>
                                </a>
                            </li>
                            <li t-if="event.country_id" class="nav-item mr-3">
                                <a t-attf-href="/event?country=#{event.country_id.id}" class="nav-link">
                                    <i class="fa fa-map-marker text-primary mr-2"/><span t-field="event.country_id"/>
                                </a>
                            </li>
                        </ul>
                        <div class="d-flex align-items-centerflex-wrap pl-sm-3 pr-0">
                            <t t-call="website_event.events_search_box_input">
                                <t t-set="_classes" t-valuef="ml-auto"/>
                            </t>
                        </div>
                    </div>
                </nav>
            </t>
            <t t-else="">
                <nav class="navbar navbar-light border-top shadow-sm navbar-expand-md">
                    <div class="container align-items-baseline">
                        <a t-att-href="event.website_url" t-field="event.name" class="navbar-brand h4 my-0 mr-0"/>
                        <button class="navbar-toggler ml-auto" type="button" data-toggle="collapse" data-target="#o_wevent_event_submenu" aria-controls="o_wevent_event_submenu" aria-expanded="false" aria-label="Toggle navigation">
                            <span class="navbar-toggler-icon"></span>
                        </button>
                        <div id="o_wevent_event_submenu" class="collapse navbar-collapse">
                            <ul class="navbar-nav w-100 flex-md-wrap" t-att-data-menu_name="editable and 'Event Menu'" t-att-data-content_menu_id="editable and event.menu_id.id">
                                <t t-foreach="event.menu_id.child_id" t-as="submenu">
                                    <t t-call="website.submenu">
                                        <t t-set="item_class" t-value="'nav-item'"/>
                                        <t t-set="link_class" t-value="'nav-link'"/>
                                    </t>
                                </t>
                            </ul>
                            <!-- Add Register additional CTA button, in addition to menus -->
                            <a t-if="event.menu_register_cta and not event.is_participating"
                                t-att-href="'/event/%s/register' % (slug(event))"
                                class="btn btn-primary ml-auto">
                                Register
                            </a>
                        </div>
                    </div>
                </nav>
            </t>
            <t t-out="0"/>
            <t t-set="editor_sub_message">Following content will appear on all events.</t>
            <div class="oe_structure oe_empty" id="oe_structure_website_event_layout_1" t-att-data-editor-sub-message="editor_sub_message"/>
        </div>
    </t>
</template>

</odoo>

```

## File: views\event_templates_page_misc.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- Multipage event - Default template when creating a new page -->
<template id="default_page">
    <t t-call="website.layout">
        <div class="oe_structure oe_empty"/>
    </t>
</template>

<!-- Multipage event - Default template for the Introduction page -->
<template id="template_intro">
    <t t-call="website_event.layout">
        <div class="oe_structure oe_empty" id="oe_structure_website_event_intro_1"/>
        <section class="s_title pt32 pb32" data-vcss="001" data-snippet="s_title" data-name="Title">
            <div class="container s_allow_columns">
                <h1 style="text-align: center;">
                    <font style="font-size: 62px;" class="o_default_snippet_text">Introduction</font>
                </h1>
            </div>
        </section>
        <div class="oe_structure oe_empty" id="oe_structure_website_event_intro_2"/>
    </t>
</template>

<!-- Multipage event - Default template for the Location page -->
<template id="template_location">
    <t t-call="website_event.layout">
        <div class="oe_structure" id="oe_structure_website_event_location_1"/>
        <section class="pt32 pb32">
            <div class="container">
                <div class="row">
                    <div class="col-12">
                        <h1 class="o_page_header mb-3">Event Location</h1>
                        <h4 class="mb-3" t-field="event.address_id" t-options='{
                            "widget": "contact",
                            "fields": ["name"],
                        }'/>
                        <div class="mb-3" t-field="event.address_id" t-options='{
                            "widget": "contact",
                            "fields": ["address"],
                            "no_marker": True
                        }'/>
                        <div class="mb-3" t-field="event.address_id" t-options='{
                            "widget": "contact",
                            "fields": ["phone", "mobile", "email"],
                            "no_marker": True
                        }'/>
                    </div>
                </div>
            </div>
        </section>
        <div class="oe_structure" id="oe_structure_website_event_location_2"/>
    </t>
</template>

<template id="404" name="Event 404">
    <t t-call="website.layout">
        <div id="wrap">
            <div class="oe_structure oe_empty">
                <div class="container">
                    <h1 class="mt-4">Event not found!</h1>
                    <p>Sorry, the requested event is not available anymore.</p>
                    <p><a t-attf-href="/event">Return to the event list.</a></p>
                </div>
            </div>
        </div>
    </t>
</template>

</odoo>

```

## File: views\event_templates_page_registration.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>


<template id="event_details" name="Event Header">
    <t t-call="website_event.layout">
        <div name="event" itemscope="itemscope" itemtype="http://schema.org/Event">
            <t t-call="website.record_cover">
                <t t-set="_record" t-value="event"/>
                <t t-set="use_filters" t-value="True"/>
                <t t-set="use_size" t-value="True"/>
                <t t-set="use_text_align" t-value="True"/>

                <div class="container d-flex flex-column flex-grow-1 justify-content-around">
                    <div class="o_wevent_event_title">
                        <span t-if="event.is_participating" class="badge badge-success o_wevent_badge"><i class="fa fa-check mr-2"/>Registered</span>
                        <h1 t-field="event.name" class="o_wevent_event_name" itemprop="name" placeholder="Event Title"/>
                        <h2 t-field="event.subtitle" class="o_wevent_event_subtitle" placeholder="Event Subtitle"/>
                    </div>
                </div>
                <div class="container">
                    <t t-call="website_event.registration_template"/>
                </div>
            </t>
            <t t-out="0"/>
        </div>
    </t>
</template>

<!-- Event - Description -->
<template id="event_description_full" name="Event Description" track="1">
    <t t-call="website_event.event_details">
        <section class="bg-200 mt-n5">
            <div class="container overflow-hidden">
                <div class="row no-gutters mt-n4 mb-3">
                    <!-- Description -->
                    <div id="o_wevent_event_main_col" class="col-lg-8 bg-white px-3 pt-5 pb-0 shadow-sm">
                        <span t-field="event.description" itemprop="description"/>
                    </div>
                    <div class="col-lg-4 bg-light shadow-sm d-print-none">
                        <!-- Date & Time -->
                        <div class="o_wevent_sidebar_block">
                            <h6 class="o_wevent_sidebar_title">Date &amp; Time</h6>
                            <div class="d-flex">
                                <h5 t-field="event.with_context(tz=event.date_tz).date_begin" class="my-1 mr-1 oe_hide_on_date_edit" t-options="{'date_only': 'true', 'format': 'EEEE'}"/>
                                <h5 class="my-1 oe_hide_on_date_edit" t-field="event.with_context(tz=event.date_tz).date_begin" t-options="{'date_only': 'true', 'format': 'long'}" itemprop="startDate" t-att-datetime="event.date_begin"/>
                            </div>
                            <t t-if="not event.is_one_day">Start -</t>
                            <span t-field="event.with_context(tz=event.date_tz).date_begin" t-options="{'time_only': 'true', 'format': 'short'}"/>
                            <t t-if="event.is_one_day">
                                <i class="fa fa-long-arrow-right mx-1"/>
                                <span t-field="event.with_context(tz=event.date_tz).date_end" t-options="{'time_only': 'true', 'format': 'short'}"/>
                            </t>
                            <t t-else="">
                                <i class="fa fa-long-arrow-down d-block text-muted mx-3 my-2" style="font-size: 1.5rem"/>
                                <div class="d-flex">
                                    <h5 t-field="event.with_context(tz=event.date_tz).date_end" class="my-1 mr-1 oe_hide_on_date_edit" t-options="{'date_only': 'true', 'format': 'EEEE'}"/>
                                    <h5 class="my-1 oe_hide_on_date_edit" t-field="event.with_context(tz=event.date_tz).date_end" t-options="{'date_only': 'true', 'format': 'long'}"/>
                                </div>
                                <t t-if="not event.is_one_day">End -</t>
                                <span t-field="event.with_context(tz=event.date_tz).date_end" t-options="{'time_only': 'true', 'format': 'short'}"/>
                            </t>
                            <!-- Timezone -->
                            <small t-esc="event.date_tz" class="d-block my-3 text-muted"/>

                            <div class="dropdown">
                                <i class="fa fa-calendar mr-1"/>
                                <a href="#" role="button" data-toggle="dropdown">Add to Calendar</a>
                                <div class="dropdown-menu">
                                    <a t-att-href="iCal_url" class="dropdown-item">iCal/Outlook</a>
                                    <a t-att-href="google_url" class="dropdown-item" target="_blank">Google</a>
                                </div>
                            </div>
                        </div>
                        <!-- Location -->
                        <div t-if="event.address_id" class="o_wevent_sidebar_block">
                            <h6 class="o_wevent_sidebar_title">Location</h6>
                            <h4 t-field="event.address_id" class="" t-options='{
                                "widget": "contact",
                                "fields": ["name"]
                            }'/>
                            <div itemprop="location" class="mb-2" t-field="event.address_id" t-options='{
                                "widget": "contact",
                                "fields": ["address"],
                                "no_marker": True
                            }'/>
                            <div class="mb-3" t-field="event.address_id" t-options='{
                                "widget": "contact",
                                "fields": ["phone", "mobile", "email"]
                            }'/>
                            <i class="fa fa-map-marker fa-fw" role="img"/>
                            <a t-att-href="event._google_map_link()" target="_blank">Get the direction</a>
                        </div>
                        <!-- Organizer -->
                        <div t-if="event.organizer_id" class="o_wevent_sidebar_block">
                            <h6 class="o_wevent_sidebar_title">Organizer</h6>
                            <h4 t-field="event.organizer_id"/>
                            <div itemprop="location" t-field="event.organizer_id" t-options="{'widget': 'contact', 'fields': ['phone', 'mobile', 'email']}"/>
                        </div>
                        <!-- Social -->
                        <div class="o_wevent_sidebar_block">
                            <h6 class="o_wevent_sidebar_title">SHARE</h6>
                            <p class="mb-2">Find out what people see and say about this event, and join the conversation.</p>
                            <t t-snippet-call="website.s_share">
                                <t t-set="_no_title" t-value="True"/>
                                <t t-set="_classes" t-valuef="o_wevent_sidebar_social mx-n1"/>
                                <t t-set="_link_classes" t-valuef="o_wevent_social_link"/>
                            </t>
                        </div>
                    </div>
                </div>
            </div>
        </section>
    </t>
</template>

<!-- Event - Registration -->
<template id="registration_template" name="Registration">
    <div t-if="toast_message" class="o_wevent_register_toaster d-none" t-att-data-message="toast_message"/>
    <div t-if="not event.event_registrations_open" class="bg-white mb-5">
        <div class="alert alert-info mb-0 d-flex justify-content-between align-items-center" role="status">
            <t t-if="not event.event_registrations_started">
                <div class="col-md-8 d-flex">
                    <em class="mr-2">Ticket Sales starting on
                        <span class="" t-esc="event.start_sale_datetime"
                            t-options="{'widget': 'datetime', 'tz_name': event.date_tz, 'format': 'short'}"/>
                        <span t-esc="event.date_tz"/>
                    </em>
                    <t t-call="website_event.registration_configure_tickets_button"
                        t-if="request.env.user.has_group('event.group_event_manager')"/>
                </div>
                <button class="btn btn-danger ml-2 col-md-4"  disabled="1">Registrations not yet open</button>
            </t>
            <t t-else="">
                <div class="col-md-8">
                    <em t-if="event.event_registrations_sold_out">Tickets for this Event are <b>Sold Out</b></em>
                    <em t-else="">Registrations are <b>closed</b></em>
                </div>
                <button class="btn btn-danger ml-3 col-md-4 py-2" disabled="1">
                    <span t-if="event.event_registrations_sold_out">Sold Out</span>
                    <span t-else="">Registrations Closed</span>
                </button>
            </t>
        </div>
    </div>
    <form t-if="event.event_registrations_open and (not event.event_ticket_ids or any(not ticket.is_expired for ticket in event.event_ticket_ids))"
        id="registration_form"
        class="mb-5"
        t-attf-data-action="/event/#{slug(event)}/registration/new" action="javascript:void(0)"
        itemscope="itemscope" itemprop="offers" itemtype="http://schema.org/AggregateOffer">
        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
        <div id="o_wevent_tickets" class="bg-white shadow-sm o_wevent_js_ticket_details" data-folded-by-default="0">
            <t t-set="tickets" t-value="event.event_ticket_ids.filtered(lambda ticket: not ticket.is_expired)"/>
            <!-- If some tickets expired and there is only one type left, we keep the same layout -->
            <t t-if="len(event.event_ticket_ids) &gt; 1">
                <div class="d-flex align-items-center py-2 pl-3 pr-2 border-bottom">
                    <span class="py-2 o_wevent_registration_title text-left">Tickets</span>
                    <div class="o_wevent_price_range d-none"/>
                    <t t-call="website_event.registration_configure_tickets_button"
                        t-if="request.env.user.has_group('event.group_event_manager')"/>
                    <span t-if="not event.event_registrations_open" class="text-danger">
                        <i class="fa fa-ban mr-2"/>Sold Out
                    </span>
                    <a href="#" role="button" class="o_wevent_registration_btn d-none" data-target="#o_wevent_tickets_collapse">
                        <span>Tickets</span>
                        <span class="btn p-0 close d-none">×</span>
                    </a>
                </div>
                <div id="o_wevent_tickets_collapse" class="collapse show">
                    <div t-foreach="tickets" t-as="ticket" class="row px-3 py-3 mx-0 bg-light border-bottom">
                        <div class="col-md-8 col-xs-12 p-0" itemscope="itemscope" itemtype="http://schema.org/Offer">
                            <h5 itemprop="name" t-field="ticket.name" class="my-0"/>
                            <t t-if="ticket.description">
                                <small t-field="ticket.description" class="text-muted py-2"/>
                                <br/>
                            </t>
                            <small t-if="ticket.end_sale_datetime and ticket.sale_available and not ticket.is_expired"
                                   class="text-muted mr-3" itemprop="availabilityEnds">Sales end on
                                <span itemprop="priceValidUntil" t-esc="ticket.end_sale_datetime"
                                t-options="{'widget': 'datetime', 'tz_name': event.date_tz, 'format': 'short'}"/>
                                <span t-esc="ticket.event_id.date_tz"/>
                            </small>
                            <small t-if="ticket.start_sale_datetime and not ticket.sale_available and not ticket.is_expired"
                                   class="text-muted mr-3" itemprop="availabilityEnds">
                                Sales start on <span itemprop="priceValidUntil" t-esc="ticket.start_sale_datetime"
                                t-options="{'widget': 'datetime', 'tz_name': event.date_tz, 'format': 'short'}"/>
                                <span t-esc="ticket.event_id.date_tz"/>
                            </small>
                        </div>
                        <div class="col-md-4 col-xs-12 p-0 d-flex align-items-center justify-content-between">
                            <div class="o_wevent_registration_multi_select"/>
                            <div class="w-auto ml-auto">
                                <select t-if="not ticket.is_expired and ticket.sale_available"
                                    t-attf-name="nb_register-#{ticket.id}"
                                    class="custom-select">
                                    <t t-set="seats_max_ticket" t-value="(not ticket.seats_limited or ticket.seats_available &gt; 9) and 10 or ticket.seats_available + 1"/>
                                    <t t-set="seats_max_event" t-value="(not event.seats_limited or event.seats_available &gt; 9) and 10 or event.seats_available + 1"/>
                                    <t t-set="seats_max" t-value="min(seats_max_ticket, seats_max_event)"/>
                                    <t t-foreach="range(0, seats_max)" t-as="nb">
                                        <option t-esc="nb" t-att-selected="len(ticket) == 0 and nb == 0 and 'selected'"/>
                                    </t>
                                </select>
                                <div t-else="" class="text-danger">
                                    <span t-if="not ticket.sale_available and not ticket.is_expired and ticket.is_launched()" >Sold Out</span>
                                    <span t-if="ticket.is_expired">Expired</span>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="row no-gutters">
                        <div class="col-md-4 offset-md-8 py-2 pl-md-0 pr-md-2">
                            <button type="submit" class="btn btn-primary o_wait_lazy_js btn-block a-submit" disabled="" t-attf-id="#{event.id}">
                                Register
                                <t t-if="event.seats_limited and event.seats_max and event.seats_available &lt;= (event.seats_max * 0.2)">
                                    (only <t t-esc="event.seats_available"/> available)
                                </t>
                            </button>
                        </div>
                    </div>
                </div>
            </t>
            <div t-else="" class="o_wevent_registration_single">
                <div class="row p-2 pl-3">
                    <div class="col-lg-8 d-flex flex-column flex-lg-row align-items-start align-items-lg-center" itemscope="itemscope" itemtype="http://schema.org/Offer">
                        <h6 itemprop="name" class="my-0 pr-3 border-right text-dark o_wevent_single_ticket_name">
                            <span t-if="tickets" t-field="tickets.name"/>
                            <span t-else="">Registration</span>
                        </h6>
                        <t t-if="tickets.description">
                            <small t-field="tickets.description" class="text-muted py-2"/>
                            <br/>
                        </t>
                        <small t-if="tickets.end_sale_datetime and tickets.sale_available and not tickets.is_expired" class="text-muted mr-3" itemprop="availabilityEnds">Sales end on
                            <span itemprop="priceValidUntil" t-esc="tickets.end_sale_datetime"
                                t-options="{'widget': 'datetime', 'tz_name': event.date_tz, 'format': 'short'}"/>
                            <span t-esc="tickets.event_id.date_tz"/>
                        </small>
                        <t t-call="website_event.registration_configure_tickets_button"
                        t-if="request.env.user.has_group('event.group_event_manager')"/>
                        <div class="ml-auto o_wevent_nowrap">
                            <t t-if="event.event_registrations_open">
                                <span class="text-dark font-weight-bold align-middle px-2">Qty</span>
                                <link itemprop="availability" content="http://schema.org/InStock"/>
                                <select t-att-name="'nb_register-%s' % (tickets.id if tickets else 0)" class="w-auto custom-select">
                                    <t t-set="seats_max_ticket" t-value="(not tickets or not tickets.seats_limited or tickets.seats_available &gt; 9) and 10 or tickets.seats_available + 1"/>
                                    <t t-set="seats_max_event" t-value="(not event.seats_limited or event.seats_available &gt; 9) and 10 or event.seats_available + 1"/>
                                    <t t-set="seats_max" t-value="min(seats_max_ticket, seats_max_event) if tickets else seats_max_event"/>
                                    <t t-foreach="range(0, seats_max)" t-as="nb">
                                        <option t-esc="nb" t-att-selected="nb == 1 and 'selected'"/>
                                    </t>
                                </select>
                            </t>
                            <t t-else="">
                                <span itemprop="availability" content="http://schema.org/SoldOut" class="text-danger">
                                    <i class="fa fa-ban mr-2"/>Sold Out
                                </span>
                            </t>
                        </div>
                    </div>
                    <div class="col-lg-4 pt-3 pt-lg-0 pl-2 pl-lg-0">
                        <button type="submit" class="btn btn-primary o_wait_lazy_js btn-block a-submit" t-attf-id="#{event.id}" disabled="disabled">
                            Register
                            <t t-if="event.seats_limited and event.seats_max and event.seats_available &lt;= (event.seats_max * 0.2)">
                                (only <t t-esc="event.seats_available"/> available)
                            </t>
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </form>
</template>

<template id="registration_attendee_details" name="Registration Attendee Details">
    <div id="modal_attendees_registration" class="modal fade" tabindex="-1" role="dialog">
        <div class="modal-dialog modal-lg" role="document">
            <form id="attendee_registration" t-attf-action="/event/#{slug(event)}/registration/confirm" method="post" class="js_website_submit_form">
                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                <div class="modal-content">
                    <div class="modal-header align-items-center">
                        <h4 class="modal-title">Attendees</h4>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span>×</span></button>
                    </div>
                    <t t-set="counter_type" t-value="1"/>
                    <t t-set="counter" t-value="0"/>
                    <t t-foreach="tickets" t-as="ticket" t-if="availability_check">
                        <t t-foreach="range(1, ticket['quantity'] + 1)" t-as="att_counter" name="attendee_loop">
                            <t t-set="counter" t-value="counter + 1"/>
                            <div class="modal-body bg-light border-bottom">
                                <h5 class="mt-1 pb-2 border-bottom">Ticket #<span t-esc="counter"/> <small class="text-muted">- <span t-esc="ticket['name']"/></small></h5>
                                <div class="row">
                                    <div class="col-lg my-2">
                                        <label>Name</label>
                                        <input class="form-control" type="text" t-attf-name="#{counter}-name" required="This field is required"
                                            t-att-value="default_first_attendee.get('name', '') if counter == 1 else ''"/>
                                    </div>
                                    <div class="col-lg my-2">
                                        <label>Email</label>
                                        <input class="form-control" type="email" t-attf-name="#{counter}-email" required="This field is required"
                                            t-att-value="default_first_attendee.get('email', '') if counter == 1 else ''"/>
                                    </div>
                                    <div class="col-lg my-2">
                                        <label>Phone <small>(Optional)</small></label>
                                        <input class="form-control" type="tel" t-attf-name="#{counter}-phone"
                                            t-att-value="default_first_attendee.get('phone', '') if counter == 1 else ''"/>
                                    </div>
                                    <input class="d-none" type="text" t-attf-name="#{counter}-event_ticket_id" t-attf-value="#{ticket['id']}"/>
                                </div>
                            </div>
                        </t>
                        <t t-set="counter_type" t-value="counter_type + 1"/>
                    </t>
                    <t t-if="not availability_check">
                        <div class="modal-body bg-light border-bottom">
                            <strong> You ordered more tickets than available seats</strong>
                        </div>
                    </t>
                    <div class="modal-footer border-0 justify-content-between">
                        <button type="button" class="btn btn-secondary js_goto_event" data-dismiss="modal">Cancel</button>
                        <button type="submit" class="btn btn-primary" t-if="availability_check">Continue</button>
                    </div>
                </div>
            </form>
        </div>
    </div>
</template>

<template id="registration_complete" name="Registration Completed">
    <t t-call="website_event.layout">
        <div class="container my-5 o_wereg_confirmed">
            <div class="row mb-3">
                <div class="col-12">
                    <h3>Registration confirmed!</h3>
                    <span class="h4 text-muted" t-esc="event.name"/>
                </div>
            </div>
            <div class="row mb-3 o_wereg_confirmed_attendees">
                <div class="col-md-4 col-xs-12 mt-3" t-foreach="attendees" t-as="attendee">
                    <div class="d-flex flex-column">
                        <span class="font-weight-bold text-truncate">
                            <t t-if="attendee.name" t-esc="attendee.name"/>
                            <t t-else="">N/A</t>
                        </span>
                        <span class="text-truncate">
                            <i class="fa fa-envelope mr-2   "></i>
                            <t t-if="attendee.email" t-esc="attendee.email"/>
                            <t t-else="">N/A</t>
                        </span>
                        <span t-if="attendee.phone">
                            <i class="fa fa-phone mr-2"></i>
                            <t t-esc="attendee.phone"/>
                        </span>
                        <span>
                            <i class="fa fa-ticket mr-2"></i>
                            <t t-if="attendee.event_ticket_id">
                                <t t-esc="attendee.event_ticket_id.name"/> (Ref: <t t-esc="attendee.id"/>)
                            </t>
                            <t t-else="">Ref: <t t-esc="attendee.id"/></t>
                        </span>
                    </div>
                </div>
            </div>
            <div class="row mb-3">
                <div class="col">
                    <div class="row">
                        <div class="col-2 col-md-1">
                            <b>Start</b>
                        </div>
                        <div class="col pl-0">
                            <span itemprop="startDate" t-esc="event.date_begin_located"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-2 col-md-1">
                            <b>End</b>
                        </div>
                        <div class="col pl-0">
                            <span itemprop="endDate" t-esc="event.date_end_located"/>
                        </div>
                    </div>
                    <div class="mt-4">
                        <h5 t-field="event.address_id" class="text-secondary font-weight-bold" t-options='{
                            "widget": "contact",
                            "fields": ["name"]
                            }'/>
                        <a itemprop="location" t-att-href="event.google_map_link()" target="_BLANK" temprop="location" t-field="event.address_id" t-options='{
                            "widget": "contact",
                            "fields": ["address"]
                            }'/>
                        <div itemprop="location" t-field="event.address_id" t-options='{
                            "widget": "contact",
                            "fields": ["phone", "mobile", "email"]
                            }'/>
                    </div>
                    <div id="add_to_calendar" class="mt-4 d-flex flex-column flex-md-row">
                        <a role="button" class="btn btn-primary" t-att-href="iCal_url">
                            <i class="fa fa-fw fa-calendar"/> Add to iCal/Outlook
                        </a>
                        <a role="button" class="btn btn-primary ml-md-2 mt-2 mt-md-0" t-att-href="google_url" target='_blank'>
                            <i class="fa fa-fw fa-calendar"/> Add to Google Calendar
                        </a>
                    </div>
                </div>
            </div>
        </div>
    </t>
</template>

<!-- Button to configure Tickets -->
<template id="registration_configure_tickets_button" name="Registration Configure Ticket Button">
    <div class="o_not_editable mx-2" role="link">
        <a class="text-nowrap" t-attf-href="/web#id=#{event.id}&amp;menu_id=#{backend_menu_id}&amp;view_type=form&amp;model=event.event">
            <i class="fa fa-gear mr-1" role="img" aria-label="Configure" title="Configure event tickets"/><em>Configure Tickets</em>
        </a>
    </div>
</template>

<!-- This template is called whenever the "Fold Ticket Details" option is checked on the website (Customize > Fold Tickets Details) -->
<template id="fold_register_details" inherit_id="website_event.registration_template" active="False" customize_show="True" name="Fold ticket details">
    <xpath expr="//div[@id='o_wevent_tickets']" position="attributes">
        <attribute name="data-folded-by-default">1</attribute>
    </xpath>
    <xpath expr="//div[@id='o_wevent_tickets_collapse']" position="attributes">
        <attribute name="class">collapse</attribute>
    </xpath>
    <xpath expr="//span[hasclass('o_wevent_registration_title')]" position="attributes">
        <attribute name="class" add="d-none" separator=" "/>
    </xpath>
    <xpath expr="//div[hasclass('o_wevent_price_range')]" position="attributes">
        <attribute name="class">o_wevent_price_range</attribute>
    </xpath>
    <xpath expr="//a[hasclass('o_wevent_registration_btn')]" position="attributes">
        <attribute
            name="class">btn btn-primary o_wevent_registration_btn collapsed ml-auto</attribute>
        <attribute name="data-toggle">collapse</attribute>
    </xpath>
</template>

</odoo>

```

## File: views\event_templates_widgets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- Search Box -->
<template id="events_search_box_input" name="Events search box">
    <t t-call="website.website_search_box_input">
        <t t-set="_classes" t-valuef="o_wevent_event_searchbar_form w-100 my-1 my-lg-0 #{_classes}"/>
        <t t-set="search_type" t-valuef="events"/>
        <t t-set="action" t-valuef="/event"/>
        <t t-set="display_description" t-valuef="true"/>
        <t t-set="display_detail" t-valuef="false"/>
        <t t-set="search" t-value="search or searches and searches['search']"/>
        <t t-set="placeholder">Search an event...</t>
        <t t-foreach="searches" t-as="item">
            <input t-if="item != 'search' and item_value != 'all'" type="hidden"
                t-att-name="item" t-att-value="item_value"/>
        </t>
        <t t-out="0"/>
    </t>
</template>

<template id="events_search_box" inherit_id="website.website_search_box" primary="True">
    <xpath expr="//div[@role='search']" position="replace">
        <form t-attf-class="o_wevent_event_searchbar_form o_wait_lazy_js w-100 my-1 my-lg-0 #{_classes}"
              t-att-action="action if action else '/event'" method="get">
            <t t-set="search" t-value="search or _searches and _searches['search']"/>
            <t t-set="placeholder" t-value="placeholder or _placeholder"/>
            <t>$0</t>
            <t t-foreach="_searches" t-as="search">
                <input t-if="search != 'search' and search_value != 'all'" type="hidden"
                    t-att-name="search" t-att-value="search_value"/>
            </t>
            <t t-out="0"/>
        </form>
    </xpath>
</template>

<!-- Snippet - Country Events - List -->
<template id="country_events_list" name="Country Events List">
    <div class="country_events_list">
        <t t-if="events">
            <h6 class="o_wevent_sidebar_title">
                <t t-if="country">
                    <i class="fa fa-flag mr-2"/>Events: <span t-esc="country.name"/>
                    <img class="img-fluid" t-att-src="country.image_url" alt=""/>
                </t>
                <t t-else="">
                    <i class="fa fa-globe mr-2"/>Upcoming Events
                </t>
            </h6>
            <ul class="list-group mb-3">
                <li t-foreach="events[:5]" t-as="event_dict" class="list-group-item d-flex justify-content-between">
                    <a t-att-href="event_dict['url']">
                        <i t-if="not event_dict['event'].website_published" class="fa fa-ban text-danger mr-1" role="img" aria-label="Unpublished" title="Unpublished"/>
                        <span t-esc="event_dict['event'].name" t-attf-class="#{(not event_dict['event'].website_published) and 'text-danger' or ''}"/>
                    </a>
                    <span t-esc="event_dict['date']"/>
                </li>
            </ul>
            <div t-if="len(events) &gt; 5">
                <t t-if="country">
                    <a t-attf-href="/event?country=#{country.id}" class="small"><b>See all events from <span t-esc="country.name"/></b></a>
                </t>
                <t t-else="">
                    <a href="/event" class="small"><b>View all</b></a>
                </t>
            </div>
        </t>
    </div>
</template>

<!-- Timer widget -->
<template id="display_timer_widget" name="Display Timer Widget">
    <t t-set="pre_countdown_display" t-value="bool(pre_countdown_text) or pre_countdown_display"/>
    <t t-set="pre_countdown_time" t-value="datetime.datetime.now().timestamp() + int(pre_remaining_time)"/>

    <div class="o_display_timer"
         t-att-data-display-class="display_class"
         t-att-data-main-countdown-time="datetime.datetime.now().timestamp() + int(main_remaining_time)"
         t-att-data-main-countdown-text="main_countdown_text"
         t-att-data-main-countdown-display="main_countdown_display"
         t-att-data-pre-countdown-time="pre_countdown_time"
         t-att-data-pre-countdown-display="pre_countdown_display"
         t-att-data-pre-countdown-text="pre_countdown_text">
        <t t-set="remaining_time" t-value="pre_remaining_time if pre_remaining_time else main_remaining_time"/>
        <span class="o_display_timer_countdown d-flex justify-content-center">
            <span class="o_countdown_text pr-1" t-esc="pre_countdown_text if pre_countdown_text else main_countdown_text if not pre_countdown_display else ''"/>
            <div t-if="int(remaining_time) > 86400"
             class="o_countdown_metric_container"><span class="o_countdown_remaining o_timer_days pr-1">0</span><span class="o_countdown_metric pr-1">days</span></div>
            <div t-if="int(remaining_time) > 3600"
                 class="o_countdown_metric_container"><span class="o_countdown_remaining o_timer_hours">00</span><span class="o_countdown_metric">:</span></div>
            <div class="o_countdown_metric_container"><span class="o_countdown_remaining o_timer_minutes">00</span><span class="o_countdown_metric">:</span></div>
            <div class="o_countdown_metric_container"><span class="o_countdown_remaining o_timer_seconds">00</span><span class="o_countdown_metric"></span></div>
        </span>
    </div>
</template>

<template id="display_timer_alert_widget" name="Display Countdown widget">
    <div class="o_we_track_timer alert alert-warning alert-dismissible fade show d-none mx-3 mt-3 mb-0" role="alert" t-att-data-time-to-live="time_to_live">
        Starts <span />
        <button type="button" class="close" data-dismiss="alert" aria-label="Close">
            <i class="fa fa-times"/>
        </button>
    </div>
</template>

</odoo>

```

## File: views\event_type_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="event_type_view_form" model="ir.ui.view">
        <field name="name">event.type.view.form.inherit.website</field>
        <field name="model">event.type</field>
        <field name="inherit_id" ref="event.view_event_type_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_title')]" position="after">
                    <span name="website_menu">
                        <label for="website_menu" string="Website Submenu"/>
                        <field name="website_menu"/>
                    </span>
                    <span name="community_menu" invisible="1">
                        <label for="community_menu" string="Community"/>
                        <field name="community_menu"/>
                    </span>
                    <span name="menu_register_cta">
                        <label for="menu_register_cta" string="Register Button"/>
                        <field name="menu_register_cta"/>
                    </span>
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
        <field name="name">res.config.settings.view.form.inherit.website.event</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="event.res_config_settings_view_form"/>
        <field name="arch" type="xml"> 
            <div name="event_settings_website" position="after">
                <div class="col-12 col-lg-6 o_setting_box">
                    <div class="o_setting_left_pane">
                        <field name="module_website_event_questions"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label string="Questions" for="module_website_event_questions"/>
                        <div class="text-muted">
                            Ask questions to attendees when registering online
                        </div>
                    </div>
                </div>
            </div>
        </field>
    </record>

</odoo>

```

## File: views\website_event_menu_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>

    <record id="website_event_menu_view_search" model="ir.ui.view">
        <field name="name">website.event.menu.view.search</field>
        <field name="model">website.event.menu</field>
        <field name="arch" type="xml">
            <search string="Website Event Menus">
                <field name="menu_id"/>
                <field name="event_id"/>
                <field name="menu_type"/>
                <field name="view_id"/>
            </search>
        </field>
    </record>

    <record id="website_event_menu_view_form" model="ir.ui.view">
        <field name="name">website.event.menu.view.form</field>
        <field name="model">website.event.menu</field>
        <field name="arch" type="xml">
            <form string="Website Event Menu">
                <sheet>
                    <group>
                        <field name="menu_id"/>
                        <field name="event_id"/>
                        <field name="menu_type"/>
                        <field name="view_id"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="website_event_menu_view_tree" model="ir.ui.view">
        <field name="name">website.event.menu.view.tree</field>
        <field name="model">website.event.menu</field>
        <field name="arch" type="xml">
            <tree string="Website Event Menus">
                <field name="menu_id"/>
                <field name="event_id"/>
                <field name="menu_type"/>
                <field name="view_id"/>
            </tree>
        </field>
    </record>

    <record id="website_event_menu_action" model="ir.actions.act_window">
        <field name="name">Menus</field>
        <field name="res_model">website.event.menu</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{'create': False}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No Website Menu Items yet!
            </p><p>
                This technical menu displays all event sub-menu items.
            </p>
        </field>
    </record>

</data>
</odoo>

```

## File: views\website_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- User Navbar -->
<template id="user_navbar_inherit_website_event" inherit_id="website.user_navbar">
    <xpath expr="//div[@id='o_new_content_menu_choices']//div[@name='module_website_event']" position="attributes">
        <attribute name="name"/>
        <attribute name="t-att-data-module-id"/>
        <attribute name="t-att-data-module-shortdesc"/>
        <attribute name="t-if">is_designer</attribute>
    </xpath>
</template>

<!-- User Navbar - Edit Options -->
<template id="event_edit_options" inherit_id="website.user_navbar" name="Edit Event Options">
    <xpath expr="//div[@id='edit-page-menu']" position="after">
        <t t-if="main_object._name == 'event.event'" t-set="action" t-value="'event.action_event_view'"/>
    </xpath>
</template>

</odoo>

```

## File: views\website_visitor_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

   <record id="website_visitor_view_search" model="ir.ui.view">
        <field name="name">website.visitor.view.search.inherit.event</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='group_by_lang']" position="after">
                <filter string="Main Contact" name="group_by_parent_id" groups="base.group_no_one" context="{'group_by': 'parent_id'}"/>
            </xpath>
        </field>
    </record>

    <record id="website_visitor_view_tree" model="ir.ui.view">
        <field name="name">website.visitor.view.tree.inherit.event</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='page_ids']" position="after">
                <field name="event_registration_count" optional="hide"/>
            </xpath>
        </field>
    </record>

    <record id="website_visitor_view_form" model="ir.ui.view">
        <field name="name">website.visitor.view.form.inherit.event</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@id='w_visitor_visit_counter']" position="before">
                <button name="%(website_event.event_registration_action_from_visitor)d"
                    type="action"
                    class="oe_stat_button" icon="fa-ticket"
                    groups="event.group_event_user"
                    attrs="{'invisible': [('event_registration_count', '=', 0)]}">
                    <field name="event_registration_count" widget="statinfo" string="Registrations"/>
                </button>
            </xpath>
            <xpath expr="//field[@name='page_ids']" position="after">
                <field name="parent_id" groups="base.group_no_one"/>
            </xpath>
        </field>
    </record>
</data></odoo>

```

