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
    'depends': [
        'event',
        'website',
        'website_partner',
        'website_mail',
    ],
    'data': [
        'data/event_data.xml',
        'data/website_snippet_data.xml',
        'views/event_snippets.xml',
        'views/snippets/s_events.xml',
        'views/snippets/snippets.xml',
        'views/event_templates_list.xml',
        'views/event_templates_svg.xml',
        'views/event_templates_page.xml',
        'views/event_templates_page_registration.xml',
        'views/event_templates_page_misc.xml',
        'views/event_templates_widgets.xml',
        'views/event_event_views.xml',
        'views/event_registration_views.xml',
        'views/event_question_views.xml',
        'views/event_registration_answer_views.xml',
        'views/event_tag_category_views.xml',
        'views/event_tag_views.xml',
        'views/event_type_views.xml',
        'views/website_event_menu_views.xml',
        'views/website_visitor_views.xml',
        'views/event_menus.xml',
        'views/website_pages_views.xml',
        'views/event_event_add.xml',
        'security/ir.model.access.csv',
        'security/event_security.xml',
    ],
    'demo': [
        'data/res_partner_demo.xml',
        'data/event_demo.xml',
        'data/event_question_demo.xml',
        'data/event_registration_demo.xml',
        'data/event_registration_answer_demo.xml',
    ],
    'application': True,
    'assets': {
        'web.assets_backend': [
            'website_event/static/src/js/tours/**/*',
        ],
        'web.assets_tests': [
            'website_event/static/tests/**/*',
        ],
        'web.assets_frontend': [
            'website_event/static/src/js/tours/**/*',
            'website_event/static/src/scss/event_templates_common.scss',
            'website_event/static/src/scss/event_templates_list.scss',
            'website_event/static/src/scss/event_templates_page.scss',
            'website_event/static/src/js/display_timer_widget.js',
            'website_event/static/src/js/register_toaster_widget.js',
            'website_event/static/src/js/website_event.js',
            'website_event/static/src/js/website_event_ticket_details.js',
        ],
        'website.assets_wysiwyg': [
            '/website_event/static/src/snippets/s_events/options.js',
            'website_event/static/src/snippets/options.js',
        ],
        'website.assets_editor': [
            'website_event/static/src/js/systray_items/*.js',
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
import re
import werkzeug

from ast import literal_eval
from collections import Counter
from werkzeug.datastructures import OrderedMultiDict
from werkzeug.exceptions import NotFound

from odoo import fields, http, _
from odoo.addons.website.controllers.main import QueryURL
from odoo.http import request
from odoo.osv import expression
from odoo.tools.misc import get_lang
from odoo.tools import lazy
from odoo.exceptions import UserError

class WebsiteEventController(http.Controller):

    def sitemap_event(env, rule, qs):
        if not qs or qs.lower() in '/events':
            yield {'loc': '/events'}

    # ------------------------------------------------------------
    # EVENT LIST
    # ------------------------------------------------------------

    def _get_events_search_options(self, **post):
        return {
            'displayDescription': False,
            'displayDetail': False,
            'displayExtraDetail': False,
            'displayExtraLink': False,
            'displayImage': False,
            'allowFuzzy': not post.get('noFuzzy'),
            'date': post.get('date'),
            'tags': post.get('tags'),
            'type': post.get('type'),
            'country': post.get('country'),
        }

    @http.route(['/event', '/event/page/<int:page>', '/events', '/events/page/<int:page>'], type='http', auth="public", website=True, sitemap=sitemap_event)
    def events(self, page=1, **searches):
        Event = request.env['event.event']
        SudoEventType = request.env['event.type'].sudo()

        searches.setdefault('search', '')
        searches.setdefault('date', 'upcoming')
        searches.setdefault('tags', '')
        searches.setdefault('type', 'all')
        searches.setdefault('country', 'all')

        website = request.website

        step = 12  # Number of events per page

        options = self._get_events_search_options(**searches)
        order = 'date_begin'
        if searches.get('date', 'upcoming') == 'old':
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
            if date[0] not in ['all', 'old']:
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

        keep = QueryURL('/event', **{
            key: value for key, value in searches.items() if (
                key == 'search' or
                (value != 'upcoming' if key == 'date' else value != 'all'))
            })

        searches['search'] = fuzzy_search_term or search

        values = {
            'current_date': current_date,
            'current_country': current_country,
            'current_type': current_type,
            'event_ids': events,  # event_ids used in website_event_track so we keep name as it is
            'dates': dates,
            'categories': request.env['event.tag.category'].search([
                ('is_published', '=', True), '|', ('website_id', '=', website.id), ('website_id', '=', False)
            ]),
            'countries': countries,
            'pager': pager,
            'searches': searches,
            'search_tags': search_tags,
            'keep': keep,
            'search_count': event_count,
            'original_search': fuzzy_search_term and search,
            'website': website
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

        view = request.env["website.event.menu"].sudo().search([
            ("event_id", "=", event.id), ("view_id.key", "ilike", page)], limit=1).view_id

        try:
            # Every event page view should have its own SEO.
            page = view.key if view else page
            values['seo_object'] = request.website.get_template(page)
            values['main_object'] = event
        except ValueError:
            # page not found
            values['path'] = re.sub(r"^website_event\.", '', page)
            values['from_template'] = 'website_event.default_page'  # .strip('website_event.')
            page = request.env.user.has_group('website.group_website_designer') and 'website.page_404' or 'http_routing.404'

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
        urls = lazy(event._get_event_resource_urls)
        return {
            'event': event,
            'main_object': event,
            'range': range,
            'google_url': lazy(lambda: urls.get('google_url')),
            'iCal_url': lazy(lambda: urls.get('iCal_url')),
            'registration_error_code': post.get('registration_error_code'),
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
                    "name": visitor.display_name,
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
        Extracts question answers:
        - For both questions asked 'once_per_order' and questions asked to every attendee
        - For questions of type 'simple_choice', extracting the suggested answer id
        - For questions of type 'text_box', extracting the text answer of the attendee.

        :param form_details: posted data from frontend registration form, like
            {'1-name': 'r', '1-email': 'r@r.com', '1-phone': '', '1-event_ticket_id': '1'}
        """
        allowed_fields = request.env['event.registration']._get_website_registration_allowed_fields()
        registration_fields = {key: v for key, v in request.env['event.registration']._fields.items() if key in allowed_fields}
        for ticket_id in list(filter(lambda x: x is not None, [form_details[field] if 'event_ticket_id' in field else None for field in form_details.keys()])):
            if int(ticket_id) not in event.event_ticket_ids.ids and len(event.event_ticket_ids.ids) > 0:
                raise UserError(_("This ticket is not available for sale for this event"))
        registrations = {}
        general_answer_ids = []
        general_identification_answers = {}
        # as we may have several questions populating the same field (e.g: the phone)
        # we use this to hold the fields that have already been handled
        # goal is to use the answer to the first question of every 'type' (aka name / phone / email / company name)
        already_handled_fields_data = {}
        for key, value in form_details.items():
            if not value or '-' not in key:
                continue

            key_values = key.split('-')
            # Special case for handling event_ticket_id data that holds only 2 values
            if len(key_values) == 2:
                registration_index, field_name = key_values
                if field_name not in registration_fields:
                    continue
                registrations.setdefault(registration_index, dict())[field_name] = int(value) or False
                continue

            if len(key_values) != 3:
                continue

            registration_index, question_type, question_id = key_values
            answer_values = None
            if question_type == 'simple_choice':
                answer_values = {
                    'question_id': int(question_id),
                    'value_answer_id': int(value)
                }
            else:
                answer_values = {
                    'question_id': int(question_id),
                    'value_text_box': value
                }

            if answer_values and not int(registration_index):
                general_answer_ids.append((0, 0, answer_values))
            elif answer_values:
                registrations.setdefault(registration_index, dict())\
                    .setdefault('registration_answer_ids', list()).append((0, 0, answer_values))

            if question_type in ('name', 'email', 'phone', 'company_name')\
                and question_type not in already_handled_fields_data.get(registration_index, []):
                if question_type not in registration_fields:
                    continue

                field_name = question_type
                already_handled_fields_data.setdefault(registration_index, list()).append(field_name)

                if not int(registration_index):
                    general_identification_answers[field_name] = value
                else:
                    registrations.setdefault(registration_index, dict())[field_name] = value

        if general_answer_ids:
            for registration in registrations.values():
                registration.setdefault('registration_answer_ids', list()).extend(general_answer_ids)

        if general_identification_answers:
            for registration in registrations.values():
                registration.update(general_identification_answers)

        return list(registrations.values())

    def _create_attendees_from_registration_post(self, event, registration_data):
        """ Also try to set a visitor (from request) and
        a partner (if visitor linked to a user for example). Purpose is to gather
        as much informations as possible, notably to ease future communications.
        Also try to update visitor informations based on registration info. """
        visitor_sudo = request.env['website.visitor']._get_visitor_from_request(force_create=True)

        registrations_to_create = []
        for registration_values in registration_data:
            registration_values['event_id'] = event.id
            if not registration_values.get('partner_id') and visitor_sudo.partner_id:
                registration_values['partner_id'] = visitor_sudo.partner_id.id
            elif not registration_values.get('partner_id'):
                registration_values['partner_id'] = False if request.env.user._is_public() else request.env.user.partner_id.id

            # update registration based on visitor
            registration_values['visitor_id'] = visitor_sudo.id

            registrations_to_create.append(registration_values)

        return request.env['event.registration'].sudo().create(registrations_to_create)

    @http.route(['''/event/<model("event.event"):event>/registration/confirm'''], type='http', auth="public", methods=['POST'], website=True)
    def registration_confirm(self, event, **post):
        """ Check before creating and finalize the creation of the registrations
            that we have enough seats for all selected tickets.
            If we don't, the user is instead redirected to page to register with a
            formatted error message. """
        if not request.env['ir.http']._verify_request_recaptcha_token('website_event_registration'):
            raise UserError(_('Suspicious activity detected by Google reCaptcha.'))
        registrations_data = self._process_attendees_form(event, post)
        registration_tickets = Counter(registration['event_ticket_id'] for registration in registrations_data)
        event_tickets = request.env['event.event.ticket'].browse(list(registration_tickets.keys()))
        if any(event_ticket.seats_limited and event_ticket.seats_available < registration_tickets.get(event_ticket.id) for event_ticket in event_tickets):
            return request.redirect('/event/%s/register?registration_error_code=insufficient_seats' % event.id)
        attendees_sudo = self._create_attendees_from_registration_post(event, registrations_data)

        return request.redirect(('/event/%s/registration/success?' % event.id) + werkzeug.urls.url_encode({'registration_ids': ",".join([str(id) for id in attendees_sudo.ids])}))

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

    <!-- Event Type -->
    <record id="event.event_type_0" model="event.type">
        <field name="question_ids" eval="[(5, 0, 0),
            (0, 0, {'title': 'Name', 'question_type': 'name', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Email', 'question_type': 'email', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Phone', 'question_type': 'phone'})]"/>
    </record>

    <record id="event.event_type_1" model="event.type">
        <field name="question_ids" eval="[(5, 0, 0),
            (0, 0, {'title': 'Name', 'question_type': 'name', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Email', 'question_type': 'email', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Phone', 'question_type': 'phone'})]"/>
    </record>

    <record id="event.event_type_2" model="event.type">
        <field name="question_ids" eval="[(5, 0, 0),
            (0, 0, {'title': 'Name', 'question_type': 'name', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Email', 'question_type': 'email', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Phone', 'question_type': 'phone'})]"/>
    </record>

    <!-- Event Event -->
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
                <div class="text-bg-light border-start border-2 border-secondary p-3">
                    <p class="mb-0"><i class="fa fa-info-circle me-2"/>This event and all the conferences are in <b>English</b>!</p>
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
                <div class="rounded-end border-start border-secondary p-3 mb-3" style="border-start-width: 3px !important;">
                    <p class="mb-0"><em>If you wish to make a presentation, please send your topic proposal as soon as possible for approval to Mr. Famke Jenssens at ngh (a) yourcompany (dot) com. The presentations should be, for example, a presentation of a community module, a case study, methodology feedback, technical, etc. Each presentation must be in English.</em></p>
                </div>
                <p class="mb-3">For any additional information, please contact us at <a href="mailto:events@yourcompany.com">events@yourcompany.com</a>.</p>
                <div class="text-bg-light border-start border-2 border-secondary p-3">
                    <p class="mb-0">OpenElec Applications reserves the right to cancel, re-name or re-locate the event or change the dates on which it is held.</p>
                </div>
            </div>
        </field>
        <field name="question_ids" eval="[(5, 0, 0),
            (0, 0, {'title': 'Name', 'question_type': 'name', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Email', 'question_type': 'email', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Phone', 'question_type': 'phone'})]"/>
    </record>

    <record id="event.event_1" model="event.event">
        <field name="website_menu" eval="True"/>
        <field name="website_published" eval="True"/>
        <field name="menu_register_cta" eval="True"/>
        <field name="subtitle">The Great Reno Balloon Race is the world's largest free hot-air ballooning event.</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_1.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
        <field name="description" type="html">
            <div class="oe_structure">
                <h5>Join us for the greatest ballon race of all times!</h5>
                <p class="lead mb-3">The best aeronauts of the world will gather on this event to offer you the most spectacular show.</p>
                <p class="lead mb-3">Around one hundred ballons will simultaneously take flight and turn the sky into a beautiful canvas of colours.</p>
                <p class="lead mb-3">This is the perfect place for spending a nice day with your family, we guarantee you will be leaving with beautiful everlasting memories!</p>
                <p class="mb-3">For any additional information, please contact us at <a href="mailto:events@yourcompany.com">events@yourcompany.com</a>.</p>
                <div class="text-bg-light border-start border-2 border-secondary p-3">
                    <p class="mb-1">We reserve the right to cancel, re-name or re-locate the event or change the dates on which it is held in case the weather fails us.</p>
                    <p class="mb0">The safety of our attendees and our aeronauts comes first!</p>
                </div>
            </div>
        </field>
        <field name="question_ids" eval="[(5, 0, 0),
            (0, 0, {'title': 'Name', 'question_type': 'name', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Email', 'question_type': 'email', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Phone', 'question_type': 'phone'})]"/>
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

    <section class="s_we_speaker p-3 mb-4" itemscope="itemscope" itemtype="http://schema.org/Person" itemprop="performer">
        <span class="badge text-bg-secondary o_wevent_badge float-end">SPEAKER</span>
        <img src="/mail/static/src/img/odoobot.png" width="70" class="img-fluid rounded-circle float-start me-3" alt=""/>
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
        <field name="question_ids" eval="[(5, 0, 0),
            (0, 0, {'title': 'Name', 'question_type': 'name', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Email', 'question_type': 'email', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Phone', 'question_type': 'phone'})]"/>
    </record>

    <record id="event.event_3" model="event.event">
        <field name="website_published" eval="True"/>
        <field name="subtitle">Experience live music, local food and beverages.</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_3.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
        <field name="description" type="html">
            <div class="oe_structure">
                <h5>Here it is, the 12th edition of our Live Musical Festival!</h5>
                <p class="lead mb-3">Once again we assembled the most legendary bands in Rock history.</p>
                <p class="lead mb-3">Bands like Bar Fighters, Led Slippers and Link Floyd will offer you the show of the century during our three day event.</p>
                <p class="lead mb-3">This is the perfect place for spending a nice time with your friends while listening to some of the most iconic rock songs of all times!</p>
                <p class="mb-3">For any additional information, please contact us at <a href="mailto:events@yourcompany.com">events@yourcompany.com</a>.</p>
                <div class="text-bg-light border-start border-2 border-secondary p-3">
                    <p class="mb-1">We reserve the right to cancel, re-name or re-locate the event or change the dates on which it is held in case the weather fails us.</p>
                </div>
            </div>
        </field>
        <field name="question_ids" eval="[(5, 0, 0),
            (0, 0, {'title': 'Name', 'question_type': 'name', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Email', 'question_type': 'email', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Phone', 'question_type': 'phone'})]"/>
    </record>

    <record id="event.event_4" model="event.event">
        <field name="website_published" eval="True"/>
        <field name="subtitle">Discover how to grow a sustainable business with our experts.</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_4.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
        <field name="question_ids" eval="[(5, 0, 0),
            (0, 0, {'title': 'Name', 'question_type': 'name', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Email', 'question_type': 'email', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Phone', 'question_type': 'phone'})]"/>
    </record>

    <record id="event.event_5" model="event.event">
        <field name="website_published" eval="True"/>
        <field name="subtitle">Bring your outdoor field hockey season to the next level by taking the field at this 9th annual Field Hockey tournament.</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_5.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
        <field name="description" type="html">
            <div class="oe_structure">
                <h5>Seasoned Hockey Fans and curious people, this tournament is for you!</h5>
                <p class="lead mb-3">The best Hockey teams of the country will compete for the national Hockey trophy.</p>
                <p class="lead mb-3">If you don't know anything about Hockey, this is a great introduction to this wonderful sport as you will will be able to see some training process and also have some time
                to chat with experienced players and trainers once the tournament is over!
                </p>
                <p class="mb-3">For any additional information, please contact us at <a href="mailto:events@yourcompany.com">events@yourcompany.com</a>.</p>
                <div class="text-bg-light border-start border-2 border-secondary p-3">
                    <p class="mb-1">We reserve the right to cancel, re-name or re-locate the event or change the dates on which it is held in case the weather fails us.</p>
                </div>
            </div>
        </field>
        <field name="question_ids" eval="[(5, 0, 0),
            (0, 0, {'title': 'Name', 'question_type': 'name', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Email', 'question_type': 'email', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Phone', 'question_type': 'phone'})]"/>
    </record>

    <record id="event.event_6" model="event.event">
        <field name="website_published" eval="False"/>
        <field name="cover_properties">{"background-image": "none", "background-color": "secondary", "opacity": ""}</field>
        <field name="question_ids" eval="[(5, 0, 0),
            (0, 0, {'title': 'Name', 'question_type': 'name', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Email', 'question_type': 'email', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Phone', 'question_type': 'phone'})]"/>
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
    <div class="text-bg-light border-start border-2 border-secondary p-3">
        <p class="mb-1">This event is fully online and FREE, if you have paid for tickets, you should get a refund.<br/>
        It will require a good Internet connection to get the best video quality.</p>
    </div>
</div>
        </field>
        <field name="question_ids" eval="[(5, 0, 0),
            (0, 0, {'title': 'Name', 'question_type': 'name', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Email', 'question_type': 'email', 'is_mandatory_answer': True}),
            (0, 0, {'title': 'Phone', 'question_type': 'phone'})]"/>
    </record>

</odoo>

```

## File: data\event_question_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <!-- EVENT TYPE SPECIFIC -->
    <record id="event_type_data_sports_question_0" model="event.question">
        <field name="title">How did you learn about this event?</field>
        <field name="once_per_order" eval="False"/>
        <field name="event_type_id" ref="event.event_type_2"/>
    </record>
    <record id="event_type_data_sports_question_0_answer_0" model="event.question.answer">
        <field name="name">Social Media</field>
        <field name="sequence">1</field>
        <field name="question_id" ref="website_event.event_type_data_sports_question_0"/>
    </record>
    <record id="event_type_data_sports_question_0_answer_1" model="event.question.answer">
        <field name="name">Blog Post</field>
        <field name="sequence">2</field>
        <field name="question_id" ref="website_event.event_type_data_sports_question_0"/>
    </record>
    <record id="event_type_data_sports_question_0_answer_2" model="event.question.answer">
        <field name="name">Radio Ad</field>
        <field name="sequence">3</field>
        <field name="question_id" ref="website_event.event_type_data_sports_question_0"/>
    </record>

    <!-- EVENT SPECIFIC -->
    <record id="event_0_question_0" model="event.question">
        <field name="title">Meal Type</field>
        <field name="question_type">simple_choice</field>
        <field name="once_per_order" eval="False"/>
        <field name="event_id" ref="event.event_0"/>
    </record>
    <record id="event_0_question_0_answer_0" model="event.question.answer">
        <field name="name">Mixed</field>
        <field name="sequence">1</field>
        <field name="question_id" ref="website_event.event_0_question_0"/>
    </record>
    <record id="event_0_question_0_answer_1" model="event.question.answer">
        <field name="name">Vegetarian</field>
        <field name="sequence">2</field>
        <field name="question_id" ref="website_event.event_0_question_0"/>
    </record>
    <record id="event_0_question_0_answer_2" model="event.question.answer">
        <field name="name">Pastafarian</field>
        <field name="sequence">3</field>
        <field name="question_id" ref="website_event.event_0_question_0"/>
    </record>
    <record id="event_0_question_1" model="event.question">
        <field name="title">Allergies</field>
        <field name="question_type">text_box</field>
        <field name="once_per_order" eval="False"/>
        <field name="event_id" ref="event.event_0"/>
    </record>
    <record id="event_0_question_2" model="event.question">
        <field name="title">How did you learn about this event?</field>
        <field name="question_type">simple_choice</field>
        <field name="once_per_order" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
    </record>
    <record id="event_0_question_2_answer_0" model="event.question.answer">
        <field name="name">Our website</field>
        <field name="sequence">1</field>
        <field name="question_id" ref="website_event.event_0_question_2"/>
    </record>
    <record id="event_0_question_2_answer_1" model="event.question.answer">
        <field name="name">Commercials</field>
        <field name="sequence">2</field>
        <field name="question_id" ref="website_event.event_0_question_2"/>
    </record>
    <record id="event_0_question_2_answer_2" model="event.question.answer">
        <field name="name">A friend</field>
        <field name="sequence">3</field>
        <field name="question_id" ref="website_event.event_0_question_2"/>
    </record>

    <!-- Questions of: "Great Reno Ballon Race" -->
    <record id="event_1_question_0" model="event.question">
        <field name="title">How did you learn about this event?</field>
        <field name="question_type">simple_choice</field>
        <field name="once_per_order" eval="False"/>
        <field name="event_id" ref="event.event_1"/>
    </record>
    <record id="event_1_question_0_answer_0" model="event.question.answer">
        <field name="name">Social Media</field>
        <field name="sequence">1</field>
        <field name="question_id" ref="website_event.event_1_question_0"/>
    </record>
    <record id="event_1_question_0_answer_1" model="event.question.answer">
        <field name="name">Blog Post</field>
        <field name="sequence">2</field>
        <field name="question_id" ref="website_event.event_1_question_0"/>
    </record>
    <record id="event_1_question_0_answer_2" model="event.question.answer">
        <field name="name">Radio Ad</field>
        <field name="sequence">3</field>
        <field name="question_id" ref="website_event.event_1_question_0"/>
    </record>

    <!-- Questions of: "Hockey Tournament" -->
    <record id="event_5_question_0" model="event.question">
        <field name="title">How did you learn about this event?</field>
        <field name="question_type">simple_choice</field>
        <field name="once_per_order" eval="False"/>
        <field name="event_id" ref="event.event_5"/>
    </record>
    <record id="event_5_question_0_answer_0" model="event.question.answer">
        <field name="name">Social Media</field>
        <field name="sequence">1</field>
        <field name="question_id" ref="website_event.event_5_question_0"/>
    </record>
    <record id="event_5_question_0_answer_1" model="event.question.answer">
        <field name="name">Blog Post</field>
        <field name="sequence">2</field>
        <field name="question_id" ref="website_event.event_5_question_0"/>
    </record>
    <record id="event_5_question_0_answer_2" model="event.question.answer">
        <field name="name">Radio Ad</field>
        <field name="sequence">3</field>
        <field name="question_id" ref="website_event.event_5_question_0"/>
    </record>
    <record id="event_5_question_1" model="event.question">
        <field name="title">What's your Hockey level?</field>
        <field name="question_type">text_box</field>
        <field name="once_per_order" eval="False"/>
        <field name="event_id" ref="event.event_5"/>
    </record>

    <!-- Questions of: "OpenWood: Furniture Collection Online Reveal" -->
    <record id="event_7_question_0" model="event.question">
        <field name="title">Which field are you working in</field>
        <field name="question_type">simple_choice</field>
        <field name="once_per_order" eval="False"/>
        <field name="event_id" ref="event.event_7"/>
    </record>
    <record id="event_7_question_0_answer_0" model="event.question.answer">
        <field name="name">Consumers</field>
        <field name="sequence">1</field>
        <field name="question_id" ref="website_event.event_7_question_0"/>
    </record>
    <record id="event_7_question_0_answer_1" model="event.question.answer">
        <field name="name">Sales</field>
        <field name="sequence">2</field>
        <field name="question_id" ref="website_event.event_7_question_0"/>
    </record>
    <record id="event_7_question_0_answer_2" model="event.question.answer">
        <field name="name">Research</field>
        <field name="sequence">3</field>
        <field name="question_id" ref="website_event.event_7_question_0"/>
    </record>
    <record id="event_7_question_1" model="event.question">
        <field name="title">How did you hear about us?</field>
        <field name="question_type">text_box</field>
        <field name="once_per_order" eval="True"/>
        <field name="event_id" ref="event.event_7"/>
    </record>

</data></odoo>

```

## File: data\event_registration_answer_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="event_registration_0_0_registration_answer_0" model="event.registration.answer">
        <field name="value_answer_id" ref="website_event.event_0_question_0_answer_0" />
        <field name="question_id" ref="website_event.event_0_question_0" />
        <field name="registration_id" ref="event.event_registration_0_0" />
    </record>
    <record id="event_registration_0_0_registration_answer_1" model="event.registration.answer">
        <field name="value_text_box">Fish Nuts</field>
        <field name="question_id" ref="website_event.event_0_question_1" />
        <field name="registration_id" ref="event.event_registration_0_0" />
    </record>
    <record id="event_registration_0_0_registration_answer_2" model="event.registration.answer">
        <field name="value_answer_id" ref="website_event.event_0_question_2_answer_0" />
        <field name="question_id" ref="website_event.event_0_question_2" />
        <field name="registration_id" ref="event.event_registration_0_0" />
    </record>
    <record id="event_registration_0_1_registration_answer_0" model="event.registration.answer">
        <field name="value_answer_id" ref="website_event.event_0_question_0_answer_1" />
        <field name="question_id" ref="website_event.event_0_question_0" />
        <field name="registration_id" ref="event.event_registration_0_1" />
    </record>
    <record id="event_registration_0_1_registration_answer_1" model="event.registration.answer">
        <field name="value_answer_id" ref="website_event.event_0_question_2_answer_0" />
        <field name="question_id" ref="website_event.event_0_question_2" />
        <field name="registration_id" ref="event.event_registration_0_1" />
    </record>
    <record id="event_registration_0_2_registration_answer_0" model="event.registration.answer">
        <field name="value_answer_id" ref="website_event.event_0_question_0_answer_2" />
        <field name="question_id" ref="website_event.event_0_question_0" />
        <field name="registration_id" ref="event.event_registration_0_2" />
    </record>
    <record id="event_registration_0_2_registration_answer_1" model="event.registration.answer">
        <field name="value_answer_id" ref="website_event.event_0_question_2_answer_2" />
        <field name="question_id" ref="website_event.event_0_question_2" />
        <field name="registration_id" ref="event.event_registration_0_2" />
    </record>

</data></odoo>

```

## File: data\event_registration_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <!-- OpenWood Collection Online Reveal: Gemini (all) -->
    <record id="event.event_registration_7_0" model="event.registration">
        <field name="visitor_id" ref="website.website_visitor_0"/>
    </record>
    <record id="event.event_registration_7_1" model="event.registration">
        <field name="visitor_id" ref="website.website_visitor_0"/>
    </record>
    <record id="event.event_registration_7_2" model="event.registration">
        <field name="visitor_id" ref="website.website_visitor_1"/>
    </record>
    <record id="event.event_registration_7_3" model="event.registration">
        <field name="visitor_id" ref="website.website_visitor_1"/>
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
                <p class="o_default_snippet_text">Come see us live, we hope to meet you!</p>
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
                <p class="o_default_snippet_text">Come see us live, we hope to meet you!</p>
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
                <p class="o_default_snippet_text">Come see us live, we hope to meet you!</p>
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
                <p class="o_default_snippet_text">Come see us live, we hope to meet you!</p>
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
                <p class="o_default_snippet_text">Come see us live, we hope to meet you!</p>
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
                <p class="o_default_snippet_text">Come see us live, we hope to meet you!</p>
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
                <p class="o_default_snippet_text">Come see us live, we hope to meet you!</p>
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
                <p class="o_default_snippet_text">Come see us live, we hope to meet you!</p>
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
                <p class="o_default_snippet_text">Come see us live, we hope to meet you!</p>
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
                <p class="o_default_snippet_text">Come see us live, we hope to meet you!</p>
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
        <field name="website_description" type="html">
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
        <field name="website_description" type="html">
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
        <field name="website_description" type="html">
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
        <field name="website_description" type="html">
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
        <field name="website_description" type="html">
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

## File: data\website_snippet_data.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="ir_filters_event_list_snippet" model="ir.filters">
        <field name="name">Upcoming Events</field>
        <field name="model_id">event.event</field>
        <field name="user_id" eval="False" />
        <field name="domain">[('date_begin', '&gt;=', context_today())]</field>
        <field name="sort">["date_begin asc"]</field>
    </record>

    <record id="website_snippet_filter_event_list" model="website.snippet.filter">
        <field name="filter_id" ref="website_event.ir_filters_event_list_snippet"/>
        <field name="field_names">name,subtitle</field>
        <field name="limit" eval="16"/>
        <field name="name">Upcoming Events</field>
    </record>

</data></odoo>

```

## File: models\event_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval
from dateutil.relativedelta import relativedelta
import json
import werkzeug.urls

from markupsafe import Markup
from pytz import utc, timezone

from odoo import api, fields, models, _
from odoo.addons.http_routing.models.ir_http import slug
from odoo.exceptions import ValidationError
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

    def _default_question_ids(self):
        return self.env['event.type']._default_question_ids()

    # description
    subtitle = fields.Char('Event Subtitle', translate=True)
    # registration
    is_participating = fields.Boolean("Is Participating", compute="_compute_is_participating")
    # website
    website_published = fields.Boolean(tracking=True)
    website_menu = fields.Boolean(
        string='Website Menu',
        compute='_compute_website_menu', precompute=True, readonly=False, store=True,
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
    address_name = fields.Char(related='address_id.name')
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
        'Is Done', compute='_compute_time_data')
    start_today = fields.Boolean(
        'Start Today', compute='_compute_time_data',
        help="Whether event is going to start today if still not ongoing")
    start_remaining = fields.Integer(
        'Remaining before start', compute='_compute_time_data',
        help="Remaining time before event starts (minutes)")
    # questions
    question_ids = fields.One2many(
        'event.question', 'event_id', 'Questions', copy=True,
        compute='_compute_question_ids', readonly=False, store=True)
    general_question_ids = fields.One2many('event.question', 'event_id', 'General Questions',
                                           domain=[('once_per_order', '=', True)])
    specific_question_ids = fields.One2many('event.question', 'event_id', 'Specific Questions',
                                            domain=[('once_per_order', '=', False)])

    def _compute_is_participating(self):
        """Heuristic

          * public, no visitor: not participating as we have no information;
          * check only confirmed and attended registrations, a draft registration
            does not make the attendee participating;
          * public and visitor: check visitor is linked to a registration. As
            visitors are merged on the top parent, current visitor check is
            sufficient even for successive visits;
          * logged, no visitor: check partner is linked to a registration. Do
            not check the email as it is not really secure;
          * logged as visitor: check partner or visitor are linked to a
            registration;
        """
        current_visitor = self.env['website.visitor']._get_visitor_from_request(force_create=False)
        base_domain = [('event_id', 'in', self.ids), ('state', 'in', ['open', 'done'])]
        if self.env.user._is_public() and not current_visitor:
            events = self.env['event.event']
        elif self.env.user._is_public():
            events = self.env['event.registration'].sudo().search(
                expression.AND([base_domain, [('visitor_id', '=', current_visitor.id)]])
            ).event_id
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
                expression.AND([base_domain, domain])
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

    @api.depends('event_type_id')
    def _compute_question_ids(self):
        """ Update event questions from its event type. Depends are set only on
        event_type_id itself to emulate an onchange. Changing event type content
        itself should not trigger this method.

        When synchronizing questions:

          * lines with no registered answers are removed;
          * type lines are added;
        """
        if self._origin.question_ids:
            # lines to keep: those with already given answers
            questions_tokeep_ids = self.env['event.registration.answer'].search(
                [('question_id', 'in', self._origin.question_ids.ids)]
            ).question_id.ids
        else:
            questions_tokeep_ids = []
        for event in self:
            if not event.event_type_id and not event.question_ids:
                event.question_ids = self._default_question_ids()
                continue

            if questions_tokeep_ids:
                questions_toremove = event._origin.question_ids.filtered(
                    lambda question: question.id not in questions_tokeep_ids)
                command = [(3, question.id) for question in questions_toremove]
            else:
                command = [(5, 0)]
            event.question_ids = command

            # copy questions so changes in the event don't affect the event type
            for question in event.event_type_id.question_ids:
                event.question_ids += question.copy({'event_type_id': False})

    # -------------------------------------------------------------------------
    # CONSTRAINT METHODS
    # -------------------------------------------------------------------------

    @api.constrains('website_id')
    def _check_website_id(self):
        for event in self:
            if event.website_id and event.website_id.company_id != event.company_id:
                raise ValidationError(_("The website must be from the same company as the event."))

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
          boolean fields (normally an entry of website.event.menu with type matching
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
                name=f'{name} {self.name}', template=xml_id,
                add_menu=False, ispage=False)
            view_id = page_result['view_id']
            view = self.env["ir.ui.view"].browse(view_id)
            url = f"/event/{slug(self)}/page/{view.key.split('.')[-1]}"  # url contains starting "/"

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
        url_date_start = self.date_begin.astimezone(timezone(self.date_tz)).strftime('%Y%m%dT%H%M%S')
        url_date_stop = self.date_end.astimezone(timezone(self.date_tz)).strftime('%Y%m%dT%H%M%S')
        params = {
            'action': 'TEMPLATE',
            'text': self.name,
            'dates': f'{url_date_start}/{url_date_stop}',
            'ctz': self.date_tz,
            'details': self.name,
        }
        if self.address_id:
            params.update(location=self.address_inline)
        encoded_params = werkzeug.urls.url_encode(params)
        google_url = GOOGLE_CALENDAR_URL + encoded_params
        iCal_url = f'/event/{self.id:d}/ics?{encoded_params}'
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
            ['upcoming', _('Upcoming Events'), [("date_end", ">", sd(today))], 0],
            ['today', _('Today'), [
                ("date_end", ">", sd(today)),
                ("date_begin", "<", sdn(today))],
                0],
            get_month_filter_domain('month', 0),
            ['old', _('Past Events'), [
                ("date_end", "<", sd(today))],
                0],
            ['all', _('All Events'), [], 0]
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
            for tags in search_tags.grouped('category_id').values():
                domain.append([('tag_ids', 'in', tags.ids)])

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
                if date_details[0] != 'upcoming':
                    current_date = date_details[1]

        search_fields = ['name']
        fetch_fields = ['name', 'website_url', 'address_name']
        mapping = {
            'name': {'name': 'name', 'type': 'text', 'match': True},
            'website_url': {'name': 'website_url', 'type': 'text', 'truncate': False},
            'address_name': {'name': 'address_name', 'type': 'text', 'match': True},
        }
        if with_description:
            search_fields.append('subtitle')
            fetch_fields.append('subtitle')
            mapping['description'] = {'name': 'subtitle', 'type': 'text', 'match': True}
        if with_date:
            mapping['detail'] = {'name': 'range', 'type': 'html'}

        # Bypassing the access rigths of partner to search the address.
        def search_in_address(env, search_term):
            ret = env['event.event'].sudo()._search([
               ('address_search', 'ilike', search_term),
            ])
            return [('id', 'in', ret)]

        return {
            'model': 'event.event',
            'base_domain': domain,
            'search_fields': search_fields,
            'search_extra': search_in_address,
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
                data['range'] = (
                    Markup('{} <i class="fa fa-long-arrow-right"></i> {}').format(begin, end)
                    if begin != end else begin
                )
        return results_data

```

## File: models\event_question.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class EventQuestion(models.Model):
    _name = 'event.question'
    _rec_name = 'title'
    _order = 'sequence,id'
    _description = 'Event Question'

    title = fields.Char(required=True, translate=True)
    question_type = fields.Selection([
        ('simple_choice', 'Selection'),
        ('text_box', 'Text Input'),
        ('name', 'Name'),
        ('email', 'Email'),
        ('phone', 'Phone'),
        ('company_name', 'Company'),
    ], default='simple_choice', string="Question Type", required=True)
    event_type_id = fields.Many2one('event.type', 'Event Type', ondelete='cascade')
    event_id = fields.Many2one('event.event', 'Event', ondelete='cascade')
    answer_ids = fields.One2many('event.question.answer', 'question_id', "Answers", copy=True)
    sequence = fields.Integer(default=10)
    once_per_order = fields.Boolean('Ask once per order',
                                    help="If True, this question will be asked only once and its value will be propagated to every attendees."
                                         "If not it will be asked for every attendee of a reservation.")
    is_mandatory_answer = fields.Boolean('Mandatory Answer')

    @api.constrains('event_type_id', 'event_id')
    def _constrains_event(self):
        if any(question.event_type_id and question.event_id for question in self):
            raise UserError(_("Question cannot be linked to both an Event and an Event Type."))

    def write(self, vals):
        """ We add a check to prevent changing the question_type of a question that already has answers.
        Indeed, it would mess up the event.registration.answer (answer type not matching the question type). """

        if 'question_type' in vals:
            questions_new_type = self.filtered(lambda question: question.question_type != vals['question_type'])
            if questions_new_type:
                answer_count = self.env['event.registration.answer'].search_count([('question_id', 'in', questions_new_type.ids)])
                if answer_count > 0:
                    raise UserError(_("You cannot change the question type of a question that already has answers!"))
        return super(EventQuestion, self).write(vals)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_answered_question(self):
        if self.env['event.registration.answer'].search_count([('question_id', 'in', self.ids)]):
            raise UserError(_('You cannot delete a question that has already been answered by attendees.'))

    def action_view_question_answers(self):
        """ Allow analyzing the attendees answers to event questions in a convenient way:
        - A graph view showing counts of each suggestions for simple_choice questions
          (Along with secondary pivot and tree views)
        - A tree view showing textual answers values for text_box questions. """
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("website_event.action_event_registration_report")
        action['domain'] = [('question_id', '=', self.id)]
        if self.question_type == 'simple_choice':
            action['views'] = [(False, 'graph'), (False, 'pivot'), (False, 'tree')]
        elif self.question_type == 'text_box':
            action['views'] = [(False, 'tree')]
        return action

```

## File: models\event_question_answer.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class EventQuestionAnswer(models.Model):
    """ Contains suggested answers to a 'simple_choice' event.question. """
    _name = 'event.question.answer'
    _order = 'sequence,id'
    _description = 'Event Question Answer'

    name = fields.Char('Answer', required=True, translate=True)
    question_id = fields.Many2one('event.question', required=True, ondelete='cascade')
    sequence = fields.Integer(default=10)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_selected_answer(self):
        if self.env['event.registration.answer'].search_count([('value_answer_id', 'in', self.ids)]):
            raise UserError(_('You cannot delete an answer that has already been selected by attendees.'))

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
    registration_answer_ids = fields.One2many('event.registration.answer', 'registration_id', string='Attendee Answers')
    registration_answer_choice_ids = fields.One2many('event.registration.answer', 'registration_id', string='Attendee Selection Answers',
        domain=[('question_type', '=', 'simple_choice')])

    def _get_website_registration_allowed_fields(self):
        return {'name', 'phone', 'email', 'company_name', 'event_id', 'partner_id', 'event_ticket_id'}

    def _get_registration_summary(self):
        res = super()._get_registration_summary()
        res['registration_answers'] = self.registration_answer_ids.filtered('value_answer_id').mapped('display_name')
        return res

```

## File: models\event_registration_answer.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class EventRegistrationAnswer(models.Model):
    """ Represents the user input answer for a single event.question """
    _name = 'event.registration.answer'
    _description = 'Event Registration Answer'
    _rec_names_search = ['value_answer_id', 'value_text_box']

    question_id = fields.Many2one(
        'event.question', ondelete='restrict', required=True,
        domain="[('event_id', '=', event_id)]")
    registration_id = fields.Many2one('event.registration', required=True, ondelete='cascade')
    partner_id = fields.Many2one('res.partner', related='registration_id.partner_id')
    event_id = fields.Many2one('event.event', related='registration_id.event_id')
    question_type = fields.Selection(related='question_id.question_type')
    value_answer_id = fields.Many2one('event.question.answer', string="Suggested answer")
    value_text_box = fields.Text('Text answer')

    _sql_constraints = [
        ('value_check', "CHECK(value_answer_id IS NOT NULL OR COALESCE(value_text_box, '') <> '')", "There must be a suggested value or a text value.")
    ]

    # for displaying selected answers by attendees in attendees list view
    @api.depends('value_answer_id', 'question_type', 'value_text_box')
    def _compute_display_name(self):
        for reg in self:
            reg.display_name = reg.value_answer_id.name if reg.question_type == "simple_choice" else reg.value_text_box

```

## File: models\event_tag.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class EventTag(models.Model):
    _name = 'event.tag'
    _inherit = ['event.tag', 'website.published.multi.mixin']

    def default_get(self, fields_list):
        result = super().default_get(fields_list)
        if self.env.context.get('default_website_id'):
            result['website_id'] = self.env.context.get('default_website_id')
        return result

```

## File: models\event_tag_category.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class EventTagCategory(models.Model):
    _name = 'event.tag.category'
    _inherit = ['event.tag.category', 'website.published.multi.mixin']

    def _default_is_published(self):
        return True

```

## File: models\event_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class EventType(models.Model):
    _name = 'event.type'
    _inherit = ['event.type']

    def _default_question_ids(self):
        return [
            (0, 0, {'title': _('Name'), 'question_type': 'name', 'is_mandatory_answer': True}),
            (0, 0, {'title': _('Email'), 'question_type': 'email', 'is_mandatory_answer': True}),
            (0, 0, {'title': _('Phone'), 'question_type': 'phone'}),
        ]

    website_menu = fields.Boolean('Display a dedicated menu on Website')
    community_menu = fields.Boolean(
        "Community Menu", compute="_compute_community_menu",
        readonly=False, store=True,
        help="Display community tab on website")
    menu_register_cta = fields.Boolean(
        'Extra Register Button', compute='_compute_menu_register_cta',
        readonly=False, store=True)
    question_ids = fields.One2many(
        'event.question', 'event_type_id', default=_default_question_ids,
        string='Questions', copy=True)

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
    _rec_name = "menu_id"

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

## File: models\website_snippet_filter.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta

from odoo import models, fields, _


class WebsiteSnippetFilter(models.Model):
    _inherit = 'website.snippet.filter'

    def _get_hardcoded_sample(self, model):
        samples = super()._get_hardcoded_sample(model)
        if model._name == 'event.event':
            data = [{
                'cover_properties': '{"background-image": "url(\'/website_event/static/src/img/event_cover_1.jpg\')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}',
                'name': _('Great Reno Ballon Race'),
                'date_begin': fields.Date.today() + timedelta(days=10),
                'date_end': fields.Date.today() + timedelta(days=11),
            }, {
                'cover_properties': '{"background-image": "url(\'/website_event/static/src/img/event_cover_2.jpg\')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}',
                'name': _('Conference For Architects'),
                'date_begin': fields.Date.today(),
                'date_end': fields.Date.today() + timedelta(days=2),
            }, {
                'cover_properties': '{"background-image": "url(\'/website_event/static/src/img/event_cover_3.jpg\')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}',
                'name': _('Live Music Festival'),
                'date_begin': fields.Date.today() + timedelta(weeks=8),
                'date_end': fields.Date.today() + timedelta(weeks=8, days=5),
            }, {
                'cover_properties': '{"background-image": "url(\'/website_event/static/src/img/event_cover_5.jpg\')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}',
                'name': _('Hockey Tournament'),
                'date_begin': fields.Date.today() + timedelta(days=7),
                'date_end': fields.Date.today() + timedelta(days=7),
            }, {
                'cover_properties': '{"background-image": "url(\'/website_event/static/src/img/event_cover_7.jpg\')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}',
                'name': _('OpenWood Collection Online Reveal'),
                'date_begin': fields.Date.today() + timedelta(days=1),
                'date_end': fields.Date.today() + timedelta(days=3),
            }, {
                'cover_properties': '{"background-image": "url(\'/website_event/static/src/img/event_cover_4.jpg\')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}',
                'name': _('Business Workshops'),
                'date_begin': fields.Date.today() + timedelta(days=2),
                'date_end': fields.Date.today() + timedelta(days=4),
            }]
            merged = []
            for index in range(0, max(len(samples), len(data))):
                merged.append({**samples[index % len(samples)], **data[index % len(data)]})
                # merge definitions
            samples = merged
        return samples

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

    @api.depends('partner_id', 'event_registration_ids.name')
    def _compute_display_name(self):
        """ If there is an event registration for an anonymous visitor, use that
        registered attendee name as visitor name. """
        super()._compute_display_name()
        # sudo is needed for `event_registration_ids`
        for visitor in self.sudo().filtered(lambda v: not v.partner_id and v.event_registration_ids):
            visitor.display_name = visitor.event_registration_ids[-1].name

    @api.depends('event_registration_ids')
    def _compute_event_registration_count(self):
        read_group_res = self.env['event.registration']._read_group(
            [('visitor_id', 'in', self.ids)],
            ['visitor_id'], ['__count'])
        visitor_mapping = {visitor.id: count for visitor, count in read_group_res}
        for visitor in self:
            visitor.event_registration_count = visitor_mapping.get(visitor.id, 0)

    @api.depends('event_registration_ids.email', 'event_registration_ids.phone')
    def _compute_email_phone(self):
        super(WebsiteVisitor, self)._compute_email_phone()

        for visitor in self.filtered(lambda visitor: not visitor.email or not visitor.mobile):
            linked_registrations = visitor.event_registration_ids.sorted(lambda reg: (reg.create_date, reg.id), reverse=False)
            if not visitor.email:
                visitor.email = next((reg.email for reg in linked_registrations if reg.email), False)
            if not visitor.mobile:
                visitor.mobile = next((reg.phone for reg in linked_registrations if reg.phone), False)

    @api.depends('event_registration_ids')
    def _compute_event_registered_ids(self):
        # include parent's registrations in a visitor o2m field. We don't add
        # child one as child should not have registrations (moved to the parent)
        for visitor in self:
            all_registrations = visitor.event_registration_ids
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
            visitor_ids = all_registrations.with_context(active_test=False).visitor_id.ids
        else:
            visitor_ids = []

        return [('id', 'in', visitor_ids)]

    def _inactive_visitors_domain(self):
        """ Visitors registered to events are considered always active and should not be deleted. """
        domain = super()._inactive_visitors_domain()
        return expression.AND([domain, [('event_registration_ids', '=', False)]])

    def _merge_visitor(self, target):
        """ Override linking process to link registrations to the final visitor. """
        self.event_registration_ids.visitor_id = target.id
        registration_wo_partner = self.event_registration_ids.filtered(lambda registration: not registration.partner_id)
        if registration_wo_partner:
            registration_wo_partner.partner_id = target.partner_id
        return super()._merge_visitor(target)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_event
from . import event_question
from . import event_question_answer
from . import event_registration
from . import event_registration_answer
from . import event_tag_category
from . import event_tag
from . import event_type
from . import website
from . import website_event_menu
from . import website_menu
from . import website_snippet_filter
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

    <record id="ir_rule_event_question_published" model="ir.rule">
        <field name="name">Event Question: not event groups: event published read</field>
        <field name="model_id" ref="website_event.model_event_question"/>
        <field name="domain_force">[('event_id.is_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

    <record id="ir_rule_event_question_event_user" model="ir.rule">
        <field name="name">Event Question: event user: read all</field>
        <field name="model_id" ref="website_event.model_event_question"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('event.group_event_registration_desk'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

    <record id="ir_rule_event_question_answer_published" model="ir.rule">
        <field name="name">Event Question Answer: not event groups: event published read</field>
        <field name="model_id" ref="website_event.model_event_question_answer"/>
        <field name="domain_force">[('question_id.event_id.is_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

    <record id="ir_rule_event_question_answer_event_user" model="ir.rule">
        <field name="name">Event Question Answer: event user: read all</field>
        <field name="model_id" ref="website_event.model_event_question_answer"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('event.group_event_registration_desk'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>
    </data>

    <record id="event.group_event_manager" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('website.group_website_restricted_editor'))]"/>
    </record>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_event_event_public,event.event,model_event_event,base.group_public,1,0,0,0
access_event_event_portal,event.event,model_event_event,base.group_portal,1,0,0,0
access_event_event_employee,event.event,model_event_event,base.group_user,1,0,0,0
access_event_event_ticket_public,event.event.ticket,event.model_event_event_ticket,base.group_public,1,0,0,0
access_event_event_ticket_portal,event.event.ticket,event.model_event_event_ticket,base.group_portal,1,0,0,0
access_event_event_ticket_employee,event.event.ticket,event.model_event_event_ticket,base.group_user,1,0,0,0
access_event_type,event.type,event.model_event_type,,0,0,0,0
access_event_tag_category_public,event.tag.category,event.model_event_tag_category,base.group_public,1,0,0,0
access_event_tag_category_portal,event.tag.category,event.model_event_tag_category,base.group_portal,1,0,0,0
access_event_tag_category_employee,event.tag.category,event.model_event_tag_category,base.group_user,1,0,0,0
access_event_tag_public,event.tag,event.model_event_tag,base.group_public,1,0,0,0
access_event_tag_portal,event.tag,event.model_event_tag,base.group_portal,1,0,0,0
access_event_tag_employee,event.tag,event.model_event_tag,base.group_user,1,0,0,0
access_website_event_menu_public,website.event.menu,model_website_event_menu,base.group_public,1,0,0,0
access_website_event_menu_portal,website.event.menu,model_website_event_menu,base.group_portal,1,0,0,0
access_website_event_menu_employee,website.event.menu,model_website_event_menu,base.group_user,1,0,0,0
access_website_event_menu_user,website.event.menu.user,model_website_event_menu,event.group_event_user,1,1,1,1
access_website_visitor_user,website.visitor.user,model_website_visitor,event.group_event_registration_desk,1,1,0,0
access_event_question_public,event.question,model_event_question,base.group_public,1,0,0,0
access_event_question_portal,event.question,model_event_question,base.group_portal,1,0,0,0
access_event_question_employee,event.question,model_event_question,base.group_user,1,0,0,0
access_event_question_user,event.question.user,model_event_question,event.group_event_user,1,1,1,1
access_event_question_answer_public,event.question.answer,model_event_question_answer,base.group_public,1,0,0,0
access_event_question_answer_portal,event.question.answer,model_event_question_answer,base.group_portal,1,0,0,0
access_event_question_answer_employee,event.question.answer,model_event_question_answer,base.group_user,1,0,0,0
access_event_question_answer_registration,event.question.answer.registration,model_event_question_answer,event.group_event_registration_desk,1,1,0,0
access_event_question_answer_user,event.question.answer.user,model_event_question_answer,event.group_event_user,1,1,1,1
access_event_registration_answer,event.registration.answer,model_event_registration_answer,event.group_event_registration_desk,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M13.238 11.483a1.927 1.927 0 0 1 .654-2.55c1.403-.912 2.927-1.233 4.181-.613.2.1.376.24.54.39L49.26 37.346c.472.426.741 1.025.741 1.653H28L13.238 11.483Z" fill="#FC868B"/><path d="M50 39c0 1.657-4.925 3-11 3s-11-1.343-11-3 4.925-3 11-3 11 1.343 11 3Z" fill="#F9464C"/><path d="M36.762 11.483a1.927 1.927 0 0 0-.654-2.55c-1.403-.912-2.927-1.233-4.181-.613-.2.1-.376.24-.54.39L.74 37.346A2.226 2.226 0 0 0 0 39h22l14.762-27.517Z" fill="#1AD3BB"/><path d="M31.693 20.93 25 14.677l-6.693 6.255L25 33.407l6.693-12.476Z" fill="#1A6F66"/><path d="M0 39c0 1.657 4.925 3 11 3s11-1.343 11-3-4.925-3-11-3-11 1.343-11 3Z" fill="#03AF89"/></svg>

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
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";

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
            self.options = self.$el.data();
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

export default publicWidget.registry.countdownWidget;

```

## File: static\src\js\register_toaster_widget.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import publicWidget from "@web/legacy/js/public/public_widget";

publicWidget.registry.RegisterToasterWidget = publicWidget.Widget.extend({
    selector: '.o_wevent_register_toaster',
    /**
     * @override
     */
    init() {
        this._super(...arguments);
        this.notification = this.bindService("notification");
    },
    /**
     * This widget allows to display a toast message on the page.
     *
     * @override
     */
    start: function () {
        const message = this.$el.data('message');
        if (message && message.length) {
            this.notification.add(message, {
                title: _t("Register"),
                type: 'info',
            });
        }
        return this._super.apply(this, arguments);
    },
});

export default publicWidget.registry.RegisterToasterWidget;

```

## File: static\src\js\website_event.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import { _t } from "@web/core/l10n/translation";
import { ReCaptcha } from "@google_recaptcha/js/recaptcha";
import { jsonrpc } from "@web/core/network/rpc_service";
import { session } from "@web/session";

// Catch registration form event, because of JS for attendee details
var EventRegistrationForm = publicWidget.Widget.extend({

    /**
     * @constructor
     */
    init: function () {
        this._super(...arguments);
        this._recaptcha = new ReCaptcha();
        this.notification = this.bindService("notification");
        // dynamic get rather than import as we don't depend on this module
        if (session.turnstile_site_key) {
            const { turnStile } = odoo.loader.modules.get("@website_cf_turnstile/js/turnstile");
            this._turnstile = turnStile;
        }
    },

    /**
     * @override
     */
    willStart: async function () {
        this._recaptcha.loadLibs();
        return this._super(...arguments);
    },

    /**
     * @override
     */
    start: function () {
        var self = this;
        const post = this._getPost();
        const noTicketsOrdered = Object.values(post).map((value) => parseInt(value)).every(value => value === 0);
        var res = this._super.apply(this.arguments).then(function () {
            $('#registration_form .a-submit')
                .off('click')
                .click(function (ev) {
                    self.on_click(ev);
                })
                .prop('disabled', noTicketsOrdered);
        });
        return res;
    },

    _getPost: function () {
        var post = {};
        $('#registration_form select').each(function () {
            post[$(this).attr('name')] = $(this).val();
        });
        return post;
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
        const post = this._getPost();
        $button.attr('disabled', true);
        const self = this;
        return jsonrpc($form.attr('action'), post).then(async function (modal) {
            const tokenObj = await self._recaptcha.getToken('website_event_registration');
            if (tokenObj.error) {
                self.notification.add(tokenObj.error, {
                    type: "danger",
                    title: _t("Error"),
                    sticky: true,
                });
                $button.prop('disabled', false);
                return false;
            }
            var $modal = $(modal);
            const form = $modal[0].querySelector("form#attendee_registration");
            self._addTurnstile(form);
            $modal.find('.modal-body > div').removeClass('container'); // retrocompatibility - REMOVE ME in master / saas-19
            $modal.appendTo(document.body);
            const modalBS = new Modal($modal[0], {backdrop: 'static', keyboard: false});
            modalBS.show();
            $modal.appendTo('body').modal('show');
            $modal.on('click', '.js_goto_event', function () {
                $modal.modal('hide');
                $button.prop('disabled', false);
            });
            $modal.on('click', '.btn-close', function () {
                $button.prop('disabled', false);
            });
            $modal.on('submit', 'form', function (ev) {
                const tokenInput = document.createElement('input');
                tokenInput.setAttribute('name', 'recaptcha_token_response');
                tokenInput.setAttribute('type', 'hidden');
                tokenInput.setAttribute('value', tokenObj.token);
                ev.currentTarget.appendChild(tokenInput);
            })
        });
    },

    _addTurnstile: function (form) {
        if (!this._turnstile) {
            return false;
        }

        const turnstileNodes = this._turnstile.addTurnstile("website_event_registration");

        const modalFooter = form.querySelector("div.modal-footer");
        const formButton = form.querySelector("button[type=submit]");

        this._turnstile.addSpinnerNoMangle(formButton);
        turnstileNodes.prependTo(modalFooter);
        this._turnstile.renderTurnstile(turnstileNodes);

        return true;
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

publicWidget.registry.EventPage = publicWidget.Widget.extend({
    selector: '#o_wevent_event_submenu .dropdown-menu a.dropdown-toggle',
    events: {
        'click ': '_onClickSubDropDown',
    },
    _onClickSubDropDown:function(ev){
        ev.stopPropagation()
    }
})

export default EventRegistrationForm;

```

## File: static\src\js\website_event_ticket_details.js

```javascript
/** @odoo-module **/
    import publicWidget from "@web/legacy/js/public/public_widget";

    publicWidget.registry.ticketDetailsWidget = publicWidget.Widget.extend({
        selector: '.o_wevent_js_ticket_details',
        events: {
            'change .form-select': '_onTicketQuantityChange'
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
            this.$('.form-select').each(function (){
                ticketCount += parseInt($(this).val());
            });
            return ticketCount;
        },

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------
        /**
         * @private
         */
        _onTicketQuantityChange: function (){
            this.$('button.btn-primary').attr('disabled', this._getTotalTicketCount() === 0);
        }
    });

export default publicWidget.registry.ticketDetailsWidget;

```

## File: static\src\js\systray_items\new_content.js

```javascript
/** @odoo-module **/

import { NewContentModal, MODULE_STATUS } from '@website/systray_items/new_content';
import { patch } from "@web/core/utils/patch";

patch(NewContentModal.prototype, {
    setup() {
        super.setup();

        const newEventElement = this.state.newContentElements.find(element => element.moduleXmlId === 'base.module_website_event');
        newEventElement.createNewContent = () => this.onAddContent('website_event.event_event_action_add', true);
        newEventElement.status = MODULE_STATUS.INSTALLED;
        newEventElement.model = 'event.event';
    },
});

```

## File: static\src\js\tours\event_tour.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import EventAdditionalTourSteps from "@event/js/tours/event_steps";

import { markup } from "@odoo/owl";
import { patch } from "@web/core/utils/patch";

patch(EventAdditionalTourSteps.prototype, {

    _get_website_event_steps() {
        return [
            ...super._get_website_event_steps(), {
                trigger: '.o_event_form_view button[title="Unpublished"]',
                content: markup(_t("Use this <b>shortcut</b> to easily access your event web page.")),
                position: 'bottom',
            }, {
                trigger: '.o_edit_website_container a',
                content: markup(_t("With the Edit button, you can <b>customize</b> the web page visitors will see when registering.")),
                position: 'bottom',
            }, {
                trigger: '#oe_snippets.o_loaded div[name="Image - Text"] .oe_snippet_thumbnail',
                content: markup(_t("<b>Drag and Drop</b> this snippet below the event title.")),
                position: 'bottom',
                run: 'drag_and_drop_native iframe #o_wevent_event_main_col',
            }, {
                trigger: 'button[data-action="save"]',
                content: markup(_t("Don't forget to click <b>save</b> when you're done.")),
                position: 'bottom',
            }, {
                trigger: '.o_menu_systray_item .o_switch_danger_success',
                extra_trigger: 'iframe body:not(.editor_enable) .o_wevent_event',
                content: markup(_t("Looking great! Let's now <b>publish</b> this page so that it becomes <b>visible</b> on your website!")),
                position: 'bottom',
            }, {
                trigger: '.o_website_edit_in_backend > a',
                extra_trigger: 'iframe .o_wevent_event',
                content: _t("This shortcut will bring you right back to the event form."),
                position: 'bottom'
            }];
    }
});

export default EventAdditionalTourSteps;

```

## File: static\src\snippets\options.js

```javascript
/** @odoo-module **/

import options from '@web_editor/js/editor/snippets.options';

options.registry.WebsiteEvent = options.Class.extend({
    init() {
        this._super(...arguments);
        this.orm = this.bindService("orm");
    },

    /**
     * @override
     */
    async start() {
        const res = await this._super(...arguments);
        this.currentWebsiteUrl = this.ownerDocument.location.pathname;
        this.eventId = this._getEventObjectId();
        // Only need for one RPC request as the option will be destroyed if a
        // change is made.
        const rpcData = await this.orm.read("event.event", [this.eventId], ["website_menu","website_url"]);
        this.data.reload = this.currentWebsiteUrl;
        this.websiteMenu = rpcData[0]['website_menu'];
        this.data.reload = rpcData[0]['website_url'];
        return res;
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * @see this.selectClass for parameters
     */
    displaySubmenu(previewMode, widgetValue, params) {
        return this.orm.call("event.event", "toggle_website_menu", [[this.eventId], widgetValue]);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _computeWidgetState(methodName, params) {
        switch (methodName) {
            case 'displaySubmenu': {
                return this.websiteMenu;
            }
        }
        return this._super(...arguments);
    },
    /**
     * Ensure that we get the event object id as we could be inside a sub-object of the event
     * like an event.track
     * @private
     */
    _getEventObjectId() {
        const objectIds = this.currentWebsiteUrl.match(/\d+(?![-\w])/);
        return parseInt(objectIds[0]) | 0;
    },
});

```

## File: static\src\snippets\s_events\000.js

```javascript
/** @odoo-module **/

import { groupBy } from '@web/core/utils/arrays';
import publicWidget from '@web/legacy/js/public/public_widget';
import DynamicSnippet from '@website/snippets/s_dynamic_snippet/000';

const DynamicSnippetEvents = DynamicSnippet.extend({
    selector: '.s_event_upcoming_snippet',
    disabledInEditableMode: false,

    /**
     * @override
     * @private
     */
    _getSearchDomain: function () {
        let searchDomain = this._super.apply(this, arguments);
        const filterByTagIds = this.$el.get(0).dataset.filterByTagIds;
        if (filterByTagIds) {
            let tagGroupedByCategory = groupBy(JSON.parse(filterByTagIds), 'category_id');
            for (const category in tagGroupedByCategory) {
                searchDomain = searchDomain.concat(
                    [['tag_ids', 'in', tagGroupedByCategory[category].map(e => e.id)]]);
            }
        }
        return searchDomain;
    }

});

publicWidget.registry.events = DynamicSnippetEvents;

```

## File: static\src\snippets\s_events\options.js

```javascript
/** @odoo-module **/

import options from '@web_editor/js/editor/snippets.options';
import dynamicSnippetOptions from '@website/snippets/s_dynamic_snippet/options';

const dynamicSnippetEventOptions = dynamicSnippetOptions.extend({
    /**
     * @override
     */
    init() {
        this._super.apply(this, arguments);
        this.modelNameFilter = 'event.event';
    },

    _setOptionsDefaultValues() {
        this._setOptionValue('numberOfRecords', 4);
        this._super.apply(this, arguments);
    },

});

options.registry.event_upcoming_snippet = dynamicSnippetEventOptions;

```

## File: static\src\snippets\s_searchbar\000.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';

publicWidget.registry.searchBar.include({
    /**
     *
     * @override
     */
    _getFieldsNames() {
        return [...this._super(), 'address_name'];
    }
});

```

## File: static\src\snippets\s_searchbar\000.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates xml:space="preserve">

    <t t-inherit="website.s_searchbar.autocomplete" t-inherit-mode="extension">
        <xpath expr="//div[@class='o_search_result_item_detail px-3']" position="inside">
            <span t-if="result['address_name']" class="small">
                <span class="fa fa-map-marker"/> <t t-out="result['address_name']"/>
            </span>
        </xpath>
    </t>

</templates>


```

## File: views\event_event_add.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<record id="event_event_view_form_add" model="ir.ui.view">
    <field name="name">event.event.view.form.add</field>
    <field name="model">event.event</field>
    <field name="arch" type="xml">
        <form js_class="website_new_content_form">
            <group>
                <field name="website_url" invisible="1"/>
                <field name="company_id" invisible="1"/>
                <field name="name" placeholder="e.g. &quot;Conference for Architects&quot;" string="Event Name"/>
                <field name="address_id" context="{'show_address': 1}"/>
                <field name="date_begin" string="Start &#8594; End" widget="daterange" options="{'end_date_field': 'date_end'}" />
                <field name="date_end" invisible="1" />
            </group>
        </form>
    </field>
</record>

<record id="event_event_action_add" model="ir.actions.act_window">
    <field name="name">New Event</field>
    <field name="res_model">event.event</field>
    <field name="view_mode">form</field>
    <field name="target">new</field>
    <field name="view_id" ref="event_event_view_form_add"/>
    <field name="context">{'default_address_id': False}</field>
</record>

</odoo>

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
            <xpath expr="//group[@name='right_event_details']/field[@name='company_id']" position="after">
                <field name="website_id" invisible="1"/>
                <field name="website_id" options="{'no_create': True}" domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]" groups="website.group_multi_website"/>
            </xpath>
            <xpath expr="//field[@name='tag_ids']" position="attributes">
                <attribute name="context">{'default_website_id': website_id}</attribute>
                <attribute name="groups">website.group_multi_website</attribute>
            </xpath>
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
            <page name="event_notes" position="before">
                <page string="Questions" name="questions">
                    <field name="question_ids" string="Question" nolabel="1">
                        <tree>
                            <field name="sequence" widget="handle" />
                            <field name="title"/>
                            <field name="is_mandatory_answer" string="Mandatory"/>
                            <field name="once_per_order" string="Once per Order"/>
                            <field name="question_type" string="Type" />
                            <field name="answer_ids" widget="many2many_tags"
                                invisible="question_type != 'simple_choice'" />
                            <button name="action_view_question_answers" type="object" class="p-0" icon="fa-bar-chart pe-1" string="Stats"
                                    title="Answer Breakdown" invisible="question_type not in ['simple_choice', 'text_box']"/>
                        </tree>
                        <!-- Need to repeat the whole tree form here to be able to create answers properly
                            Without this, the sub-fields of answer_ids are unknown to the web framework.
                            We need this because we create questions and answers when the event type changes. -->
                        <form string="Question">
                            <sheet>
                                <h1 class="d-flex"><field name="title" placeholder='e.g. "Do you have any diet restrictions?"' class="flex-grow-1"/></h1>
                                <group>
                                    <group>
                                        <field name="is_mandatory_answer"/>
                                        <field name="question_type" widget="radio"/>
                                    </group>
                                    <group>
                                        <field name="once_per_order"/>
                                    </group>
                                </group>
                                <notebook invisible="question_type != 'simple_choice'">
                                    <page string="Answers" name="answers">
                                        <field name="answer_ids">
                                            <tree editable="bottom">
                                                <!-- 'display_name' is necessary for the many2many_tags to work on the event view -->
                                                <field name="display_name" column_invisible="True" />
                                                <field name="sequence" widget="handle" />
                                                <field name="name"/>
                                            </tree>
                                        </field>
                                    </page>
                                </notebook>
                            </sheet>
                        </form>
                    </field>
                </page>
            </page>
              
            <field name="tag_ids" position="attributes">
                <attribute name="domain">['|', ('category_id.website_id', '=', website_id), ('category_id.website_id', '=', False)]</attribute>
                <attribute name="invisible">not website_id</attribute>
            </field>

            <field name="tag_ids" position="after">
                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_quick_create': True}" invisible="website_id"></field>
            </field>
            
        </field>
    </record>

    <record id="event_event_view_list" model="ir.ui.view">
        <field name="name">event.event.view.list.inherit.website</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="event.view_event_tree"/>
        <field name="arch" type="xml">
            <field name="company_id" position="after">
                <field name="company_id" column_invisible="True"/>
                <field name="website_id" groups="website.group_multi_website" domain="['|', ('company_id', '=', company_id), ('company_id', '=', False)]" optional="show"/>
            </field>
        </field>
    </record>

    <record id="event_event_view_search" model="ir.ui.view">
        <field name="name">event.event.search.inherit.website</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="event.view_event_search"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='upcoming']" position="after">
                <separator/>
                <filter string="Published" name="filter_published" domain="[('website_published', '=', True)]"/>
            </xpath>
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

## File: views\event_question_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_question_view_form" model="ir.ui.view">
        <field name="name">event.question.view.form</field>
        <field name="model">event.question</field>
        <field name="arch" type="xml">
            <form string="Question">
                <sheet>
                    <h1 class="d-flex"><field name="title" placeholder='e.g. "Do you have any diet restrictions?"' class="flex-grow-1"/></h1>
                    <group>
                        <group>
                            <field name="is_mandatory_answer"/>
                            <field name="question_type" widget="radio"/>
                        </group>
                        <group>
                            <field name="once_per_order"/>
                        </group>
                    </group>
                    <notebook invisible="question_type != 'simple_choice'">
                        <page string="Answers" name="answers">
                            <field name="answer_ids">
                                <tree editable="bottom">
                                    <!-- 'display_name' is necessary for the many2many_tags to work on the event view -->
                                    <field name="display_name" column_invisible="True" />
                                    <field name="sequence" widget="handle" />
                                    <field name="name"/>
                                </tree>
                            </field>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>
</odoo>

```

## File: views\event_registration_answer_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_registration_answer_view_search" model="ir.ui.view">
        <field name="name">event.registration.answer.view.search</field>
        <field name="model">event.registration.answer</field>
        <field name="arch" type="xml">
            <search>
                <field name="value_text_box" />
                <field name="value_answer_id" />
                <field name="question_id" />
            </search>
        </field>
    </record>

    <record id="event_registration_answer_view_tree" model="ir.ui.view">
        <field name="name">event.registration.answer.view.tree</field>
        <field name="model">event.registration.answer</field>
        <field name="arch" type="xml">
            <tree string="Answer Breakdown" create="0">
                <field name="registration_id" optional="show" />
                <field name="partner_id" optional="hide" />
                <field name="question_id" optional="show" />
                <field name="value_text_box" />
                <field name="value_answer_id" string="Selected answer" />
            </tree>
        </field>
    </record>

    <record id="event_registration_answer_view_graph" model="ir.ui.view">
        <field name="name">event.registration.answer.view.graph</field>
        <field name="model">event.registration.answer</field>
        <field name="arch" type="xml">
            <graph string="Answer Breakdown" sample="1">
                <field name="value_answer_id" />
            </graph>
        </field>
    </record>

    <record id="event_registration_answer_view_pivot" model="ir.ui.view">
        <field name="name">event.registration.answer.view.pivot</field>
        <field name="model">event.registration.answer</field>
        <field name="arch" type="xml">
            <pivot string="Answer Breakdown" sample="1">
                <field name="registration_id" type="row"/>
                <field name="value_answer_id" type="col"/>
            </pivot>
        </field>
    </record>

    <record id="action_event_registration_report" model="ir.actions.act_window">
        <field name="name">Answer Breakdown</field>
        <field name="res_model">event.registration.answer</field>
        <field name="view_mode">tree,graph,pivot</field>
        <field name="search_view_id" ref="event_registration_answer_view_search"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No Answers yet!
            </p><p>
                Come back once you have registrations to overview answers.
            </p>
        </field>
    </record>
</odoo>

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
            <xpath expr="//field[@name='phone']" position="after">
                <field name="visitor_id" groups="base.group_no_one"/>
            </xpath>
            <sheet position="inside">
                <notebook>
                    <page string="Questions" name="questions">
                        <field name="registration_answer_ids" widget="one2many">
                            <tree editable="bottom">
                                <field name="event_id" column_invisible="True" />
                                <field name="question_id" domain="[('event_id', '=', event_id)]" options="{'no_create': True}" />
                                <field name="question_type" string="Type" />
                                <field name="value_answer_id"
                                    invisible="question_type != 'simple_choice'"
                                    domain="[('question_id', '=', question_id)]" options="{'no_create': True}"/>
                                <field name="value_text_box" invisible="question_type == 'simple_choice'" />
                            </tree>
                            <kanban class="o_kanban_mobile" create="false" delete="false">
                                <field name="event_id"/>
                                <field name="question_id"/>
                                <field name="question_type"/>
                                <field name="value_answer_id"/>
                                <field name="value_text_box"/>

                                <templates>
                                    <t t-name="kanban-box">
                                        <div class="d-flex flex-column justify-content-between">
                                            <div class="o_kanban_record_title oe_kanban_details">
                                                <strong>
                                                    <field name="question_id" domain="[('event_id', '=', event_id)]"/>
                                                </strong>
                                            </div>
                                            <field name="value_answer_id"
                                                invisible="question_type != 'simple_choice'"
                                                domain="[('question_id', '=', question_id)]" options="{'no_create': True}"/>
                                            <field name="value_text_box"
                                                invisible="question_type == 'simple_choice'"/>
                                        </div>
                                    </t>
                                </templates>
                            </kanban>
                        </field>
                    </page>
                </notebook>
            </sheet>
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
            <field name="state" position="before">
                <field name="registration_answer_ids" string="Selected Answers" widget="many2many_tags" optional="hide" readonly="1"/>
            </field>
        </field>
    </record>

    <record id="event_registration_view_kanban" model="ir.ui.view">
        <field name="name">event.registration.kanban.inherit.online</field>
        <field name="model">event.registration</field>
        <field name="inherit_id" ref="event.event_registration_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='event_ticket_id']" position="before">
                <field name="registration_answer_choice_ids" class="mt-1" widget="many2many_tags"/>
            </xpath>
        </field>
    </record>

    <record id="event_registration_view_search" model="ir.ui.view">
        <field name="name">event.registration.view.search.inherit.online</field>
        <field name="model">event.registration</field>
        <field name="inherit_id" ref="event.view_registration_search"/>
        <field name="arch" type="xml">
            <field name="partner_id" position="after">
                <field name="registration_answer_ids" string="Selected Answers"/>
            </field>
        </field>
    </record>
</data></odoo>

```

## File: views\event_snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- Snippets and options -->
<template id="snippets" inherit_id="website.snippets">
    <xpath expr="//t[@id='event_speaker_bio_hook']" position="replace">
        <t t-snippet="website_event.s_speaker_bio" string="Speaker Bio" t-thumbnail="/website_event/static/src/img/snippets_thumbs/s_speaker_bio.svg"/>
    </xpath>
</template>

<template id="snippet_options" inherit_id="website.snippet_options">
    <xpath expr="//*[@t-set='so_content_addition_selector']" position="inside" t-translation="off">, .s_speaker_bio</xpath>
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
        <span class="badge text-bg-secondary text-uppercase o_wevent_badge">Speaker</span>
        <img src="/website_event/static/src/img/speaker.png" width="70" class="img-fluid rounded-circle float-start me-3" alt=""/>
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
                <field name="website_id" groups="website.group_multi_website" string="Website" invisible="not is_published"/>
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

## File: views\event_tag_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_tag_view_form_inherit" model="ir.ui.view">
        <field name="name">event.tag.view.form.inherit</field>
        <field name="model">event.tag</field>
        <field name="inherit_id" ref="event.event_tag_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='category_id']" position="before">
                <field name="website_id" invisible="1"/>
            </xpath>

            <xpath expr="//field[@name='category_id']" position="attributes">
                <attribute name="domain">['|', ('website_id', '=', website_id), ('website_id', '=', False)]</attribute>
            </xpath>

        </field>
    </record>

</odoo>

```

## File: views\event_templates_list.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- Index -->
<template id="index" name="Events" track="1">
    <t t-call="website.layout">
        <t t-set="head">
            <meta t-if="search_tags" name="robots" content="none"/>
        </t>

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
            <div class="o_wevent_events_list">
                <div class="container">
                    <t t-call="website_event.searched_tags"/>
                    <div class="row">
                        <div id="o_wevent_index_main_col" t-attf-class="col-md mb-3 #{opt_events_list_columns and 'opt_events_list_columns' or 'opt_events_list_rows'}">
                            <div class="row g-4 g-lg-3 g-xxl-4">
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
<template id="opt_index_sidebar" inherit_id="website_event.index" active="False" name="Show Sidebar">
    <xpath expr="//div[@id='o_wevent_index_main_col']" position="after">
        <t t-call="website_event.index_sidebar"/>
    </xpath>
</template>

<!-- Index Topbar -->
<template id="index_topbar" name="Topbar">
        <div class="container mt-3 mb-4">
            <div class="o_wevent_index_topbar_filters d-flex d-print-none align-items-center justify-content-end flex-wrap gap-2 w-100">
                <h2 class="h4 my-0 me-auto pe-sm-4">Events</h2>
                    <t t-foreach="categories" t-as="category">
                        <div t-if="category.is_published and category.tag_ids and any(tag.color for tag in category.tag_ids)" class="dropdown d-none d-lg-block">
                            <a href="#" role="button" class="btn btn-light dropdown-toggle" data-bs-toggle="dropdown">
                                <t t-out="category.name"/>
                            </a>
                            <div class="dropdown-menu">
                                <t t-foreach="category.tag_ids" t-as="tag">
                                    <a t-if="tag.color"
                                        t-att-href="'/event?%s' % keep_query('*', tags=str((search_tags - tag).ids if tag in search_tags else (tag | search_tags).ids))"
                                        t-attf-class="dropdown-item d-flex align-items-center justify-content-between #{'active' if tag in search_tags else ''}">
                                        <t t-out="tag.name"/>
                                    </a>
                                </t>
                            </div>
                        </div>
                    </t>
                <div class="o_wevent_search d-flex w-100 w-lg-auto">
                    <div class="w-100 flex-grow-1">
                        <t t-call="website_event.events_search_box_input"/>
                    </div>
                    <button class="btn btn-light position-relative ms-2 d-lg-none"
                        data-bs-toggle="offcanvas"
                        data-bs-target="#o_wevent_index_offcanvas">
                        <i class="fa fa-sliders"/>
                    </button>
                </div>
            </div>
            <!-- Off canvas filters on mobile-->
            <div id="o_wevent_index_offcanvas" class="o_website_offcanvas offcanvas offcanvas-end d-lg-none p-0 overflow-visible mw-75">
                <div class="offcanvas-header">
                    <h5 class="offcanvas-title">Filters</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="offcanvas" aria-label="Close"/>
                </div>
                <div class="offcanvas-body p-0">
                    <div class="accordion accordion-flush">
                        <t t-foreach="categories" t-as="category">
                            <div class="accordion-item">
                                <h2 class="accordion-header">
                                    <button class="accordion-button collapsed"
                                        type="button"
                                        data-bs-toggle="collapse"
                                        t-attf-aria-controls="o_wevent_offcanvas_cat_#{category.id}"
                                        t-att-data-bs-target="'.o_wevent_offcanvas_cat_%s' % category.id"
                                        aria-expanded="false">
                                        <t t-out="category.name"/>
                                    </button>
                                </h2>
                                <div t-attf-id="o_wevent_offcanvas_cat_#{category.id}" t-attf-class="o_wevent_offcanvas_cat_#{category.id} accordion-collapse collapse">
                                    <div class="accordion-body pt-0">
                                        <ul class="list-group list-group-flush">
                                            <t t-if="category.is_published and category.tag_ids and any(tag.color for tag in category.tag_ids)" t-foreach="category.tag_ids" t-as="tag">
                                                <li t-if="tag.color" class="list-group-item border-0 px-0">
                                                    <a t-att-href="'/event?%s' % keep_query('*', tags=str((search_tags - tag).ids if tag in search_tags else (tag | search_tags).ids))"
                                                        class="text-reset" t-att-title="tag.name">
                                                        <div class="form-check">
                                                            <input class="form-check-input pe-none" type="checkbox" t-attf-name="#{tag.color}" t-att-checked="tag in search_tags"/>
                                                            <label class="form-check-label" t-attf-for="#{tag.color}" t-out="tag.name"/>
                                                        </div>
                                                    </a>
                                                </li>
                                            </t>
                                        </ul>
                                    </div>
                                </div>
                            </div>
                        </t>
                        <span class="o_wevent_offcanvas_date"/>
                        <span class="o_wevent_offcanvas_country"/>
                    </div>
                </div>
            </div>
        </div>
</template>

<template id="searched_tags" name="Searched tags">
    <div class="d-flex align-items-center">
        <t t-foreach="search_tags" t-as="tag">
            <span t-attf-class="d-inline-flex align-items-baseline rounded border ps-2 me-2 mb-2 #{'o_tag_color_%s' % tag.color if tag.color else ''}">
                <t t-out="tag.display_name"/>
                <a t-att-href="'/event?%s' % keep_query('*', tags=str((search_tags - tag).ids))" class="btn border-0 py-1 text-white">&#215;</a>
            </span>
        </t>
    </div>
</template>

<!-- Filter - Date -->
<template id="event_time" inherit_id="website_event.index_topbar" name="Filter by Date">
    <xpath expr="//div[hasclass('o_wevent_search')]" position="before">
        <div class="dropdown d-none d-lg-block">
            <a href="#" role="button" class="btn btn-light dropdown-toggle" data-bs-toggle="dropdown" title="Filter by Date">
                <t t-if="current_date" t-out="current_date"/>
                <t t-else="">Upcoming Events</t>
            </a>
            <div class="dropdown-menu">
                <t t-foreach="dates" t-as="date">
                    <t t-if="date[3] or (date[0] in ('old','upcoming','all'))">
                        <t t-set="is_active" t-value="searches.get('date') == date[0]"/>
                        <a t-att-href="keep('/event', date=date[0])" t-attf-class="dropdown-item d-flex align-items-center justify-content-between #{is_active and 'active'}">
                            <t t-out="date[1]"/>
                            <span t-if="date[3]" t-out="date[3]" t-attf-class="badge rounded-pill ms-3 #{is_active and 'text-bg-light' or 'text-bg-primary'}"/>
                        </a>
                    </t>
                </t>
            </div>
        </div>
    </xpath>
    <xpath expr="//span[hasclass('o_wevent_offcanvas_date')]" position="replace">
        <div class="accordion-item">
            <h2 class="accordion-header">
                <button class="accordion-button collapsed"
                    type="button"
                    data-bs-toggle="collapse"
                    data-bs-target=".o_wevent_offcanvas_date"
                    aria-expanded="false"
                    aria-controls="o_wevent_offcanvas_date">
                    Date
                </button>
            </h2>
            <div id="o_wevent_offcanvas_date" class="o_wevent_offcanvas_date accordion-collapse collapse" aria-labelledby="offcanvas_date">
                <div class="accordion-body pt-0">
                    <ul class="list-group list-group-flush">
                        <t t-foreach="dates" t-as="date">
                            <li t-if="date[3] or (date[0] in ('old','upcoming','all'))" class="list-group-item px-0 border-0">
                                <t t-set="is_active" t-value="searches.get('date') == date[0]"/>
                                <a t-att-href="keep('/event', date=date[0])" class="d-flex align-items-center justify-content-between text-reset text-decoration-none" t-att-title="date[1]">
                                    <div class="form-check flex-basis-100">
                                        <input class="form-check-input pe-none" type="radio" t-attf-name="#{date[1]}" t-att-checked="is_active"/>
                                        <label class="form-check-label" t-attf-for="#{date[1]}" t-out="date[1]"/>
                                    </div>
                                    <span t-if="date[3]" t-out="date[3]" class="badge rounded-pill text-bg-light"/>
                                </a>
                            </li>
                        </t>
                    </ul>
                </div>
            </div>
        </div>
    </xpath>
</template>

<!-- Filter - Location -->
<template id="event_location" inherit_id="website_event.index_topbar" active="False" name="Filter by Country">
    <xpath expr="//div[hasclass('o_wevent_index_topbar_filters')]/div" position="before">
        <div class="dropdown d-none d-lg-block">
            <a href="#" role="button" class="btn btn-light dropdown-toggle" data-bs-toggle="dropdown" title="Filter by Country">
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
                        <t t-set="is_active" t-value="searches.get('country') == str(country['country_id'] and country['country_id'][0])"/>
                        <a t-att-href="keep('/event', country=country['country_id'][0])" t-attf-class="dropdown-item d-flex align-items-center justify-content-between #{is_active and 'active'}" t-att-title="country['country_id'][1]">
                            <t t-out="country['country_id'][1]"/>
                            <span t-out="country['country_id_count']" t-attf-class="badge rounded-pill ms-3 #{is_active and 'text-bg-light' or 'text-bg-primary'}"/>
                        </a>
                    </t>
                    <t t-else="">
                        <t t-set="is_active" t-value="searches.get('country') == 'online'"/>
                        <a t-att-href="keep('/event', country='online')" t-attf-class="dropdown-item d-flex align-items-center justify-content-between #{is_active and 'active'}" title="Online Events">
                            <span>Online Events</span>
                            <span t-out="country['country_id_count']" t-attf-class="badge rounded-pill ms-3 #{is_active and 'text-bg-light' or 'text-bg-primary'}"/>
                        </a>
                    </t>
                </t>
            </div>
        </div>
    </xpath>
    <xpath expr="//span[hasclass('o_wevent_offcanvas_country')]" position="replace">
        <div class="accordion-item">
            <h2 class="accordion-header">
                <button class="accordion-button collapsed"
                    type="button"
                    data-bs-toggle="collapse"
                    data-bs-target=".o_wevent_offcanvas_country"
                    aria-expanded="false"
                    aria-controls="o_wevent_offcanvas_country">
                    Countries
                </button>
            </h2>
            <div id="o_wevent_offcanvas_country" class="o_wevent_offcanvas_country accordion-collapse collapse" aria-labelledby="offcanvas_country">
                <div class="accordion-body pt-0">
                    <ul class="list-group list-group-flush">
                        <li t-foreach="countries" t-as="country" class="list-group-item px-0 border-0">
                            <t t-if="country['country_id']">
                                <t t-set="is_active" t-value="searches.get('country') == str(country['country_id'] and country['country_id'][0])"/>
                                <a t-att-href="keep('/event', country=country['country_id'][0])" class="d-flex align-items-center justify-content-between text-reset text-decoration-none" t-att-title="country['country_id'][1]">
                                    <div class="form-check flex-basis-100">
                                        <input class="form-check-input pe-none" type="radio" t-attf-name="#{country['country_id'][1]}" t-att-checked="is_active"/>
                                        <label class="form-check-label" t-attf-for="#{country['country_id'][1]}" t-out="country['country_id'][1]"/>
                                    </div>
                                    <span t-out="country['country_id_count']" class="badge rounded-pill text-bg-light ms-auto"/>
                                </a>
                            </t>
                            <t t-else="">
                                <t t-set="is_active" t-value="searches.get('country') == 'online'"/>
                                <a t-att-href="keep('/event', country='online')" class="d-flex align-items-center justify-content-between text-reset text-decoration-none">
                                    <div class="form-check flex-basis-100">
                                        <input class="form-check-input pe-none" type="radio" name="Online Events" t-att-checked="is_active"/>
                                        <label class="form-check-label" for="Online Events">Online Events</label>
                                    </div>
                                    <span t-out="country['country_id_count']" class="badge rounded-pill px-2 text-bg-light ms-auto"/>
                                </a>
                            </t>
                        </li>
                    </ul>
                </div>
            </div>    
        </div>
    </xpath>
</template>

<!-- Index - Events list -->
<template id="events_list" name="Events list">
    <!-- Options -->
    <t t-set="opt_index_sidebar" t-value="is_view_active('website_event.opt_index_sidebar')"/>
    <t t-if="opt_events_list_columns" t-set="opt_event_size" t-value="opt_index_sidebar and 'col-md-6' or 'col-md-6 col-lg-4 col-xl-3'"/>
    <t t-else="" t-set="opt_event_size" t-value="opt_index_sidebar and 'col-12' or 'col-xl-12'"/>
    <!-- No events -->
    <div t-if="not event_ids" class="col-12 text-center">
        <div t-call="website_event.event_empty_events_svg" class="my-4"/>
        <h3>No events scheduled yet</h3>
        <p t-if="searches['search']">We couldn't find any event matching your search for: <strong t-out="searches['search']"/>.</p>
        <p t-elif="searches['tags']">No event matching your search criteria could be found.</p>
        <p t-else="">We couldn't find any event scheduled at this moment.</p>
        <div class="o_not_editable my-3" groups="event.group_event_user">
            <a class="o_wevent_cta btn" target="_blank" href="/web?#model=event.event&amp;view_type=form&amp;action=event.action_event_view">
                <span class="fa fa-plus me-1"/> Create an Event
            </a>
        </div>
    </div>
    <!-- Fuzzy search -->
    <div t-if="event_ids and original_search" class="col-12 alert alert-warning mt8">
        No results found for '<span t-out="original_search"/>'. Showing results for '<span t-out="searches['search']"/>'.
    </div>
    <!-- List -->
    <div t-foreach="event_ids" t-as="event" t-attf-class=" #{opt_event_size}">
        <a t-cache="event if not editable and event.website_published else None" t-attf-href="/event/#{ slug(event) }/#{(not event.menu_id) and 'register'}" class="text-decoration-none text-reset " t-att-data-publish="event.website_published and 'on' or 'off'">
            <article t-attf-class="h-100 #{opt_events_list_cards and 'card'}" itemscope="itemscope" itemtype="http://schema.org/Event">
                <div t-attf-class="h-100 #{opt_events_list_columns and 'd-flex flex-wrap flex-column' or 'row mx-0'}">
                    <!-- Header -->
                    <header t-attf-class="card-header overflow-hidden bg-secondary p-0 border-0 rounded-0 #{opt_events_list_columns and 'col-12' or 'col-4 col-lg-3'} #{(not opt_events_list_cards) and 'shadow-sm'}">
                        <!-- Image + Link -->
                        <div class="d-block h-100 w-100">
                            <t t-call="website.record_cover">
                                <t t-set="_record" t-value="event"/>
                                <!-- Short Date -->
                                <div t-attf-class="o_wevent_event_date position-absolute shadow-sm o_not_editable #{(not opt_events_list_columns) and 'left'} ">
                                    <span t-out="event.date_begin" t-options="{'widget': 'datetime', 'tz_name': event.date_tz, 'format': 'LLL'}" class="o_wevent_event_month"/>
                                    <span t-out="event.date_begin" t-options="{'widget': 'datetime', 'tz_name': event.date_tz, 'format': 'dd'}" class="o_wevent_event_day oe_hide_on_date_edit"/>
                                </div>
                                <!-- Not open -->
                                <span t-if="not event.event_registrations_open and (not opt_events_list_cards or not opt_events_list_columns)" class="position-absolute bottom-0 px-3 py-2 w-100 text-bg-light">
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
                                <!-- Participating -->
                                <small t-if="event.is_participating" class="o_wevent_participating position-absolute bottom-0 px-3 py-2 w-100 text-bg-success">
                                    <i class="fa fa-check me-2"/>Registered
                                </small>
                                <!-- Unpublished -->
                                <small t-if="not event.website_published" class="o_wevent_unpublished position-absolute bottom-0 px-3 py-2 w-100 text-bg-danger">
                                    <i class="fa fa-ban me-2"/>Unpublished
                                </small>
                            </t>
                        </div>
                    </header>
                    
                    <!-- Body -->
                    <main t-attf-class="card-body position-relative d-flex flex-column justify-content-between gap-2 #{opt_events_list_columns and 'col-12 py-3' or 'col-8 col-lg-9 px-4'} #{not opt_events_list_cards and opt_events_list_columns and 'bg-transparent px-0'} #{not opt_events_list_cards and not opt_events_list_columns and 'bg-transparent py-0'}">
                        <div id="event_details">
                            <div class="d-flex flex-wrap gap-1 small">
                                <t t-foreach="event.tag_ids.filtered(lambda tag: tag.category_id.website_id == website or not tag.category_id.website_id)" t-as="tag">
                                    <span t-if="tag.color"
                                        t-attf-class="badge mt-3 mt-sm-0 rounded-pill px-2 p-1 #{'o_tag_color_%s' % tag.color if tag.color else 'text-bg-light'}">
                                        <t t-out="tag.name"/>
                                    </span>
                                </t>
                            </div>
                            <!-- Title -->
                            <h5 t-attf-class="card-title my-2 #{(not event.website_published) and 'text-danger'}">
                                <span t-field="event.name" itemprop="name"/>
                            </h5>
                            <!-- Start Date & Time -->
                            <!-- TODO remove t-out one in master -->
                            <small t-if="False" class="o_not_editable opacity-75" itemprop="description" t-out="event.subtitle">
                            </small>
                            <small class="opacity-75" itemprop="description" t-field="event.subtitle"/>
                        </div>
                        <!-- Location -->
                        <small class="o_not_editable fw-bold" itemprop="location" t-out="event.address_id" t-options="{'widget': 'contact', 'fields': ['city'], 'no_marker': 'true'}"/>
                    </main>
                    
                    <!-- Footer -->
                    <footer t-if="not event.event_registrations_open and opt_events_list_columns and opt_events_list_cards"
                        t-att-class="'small align-self-end w-100 %s %s' % (
                            opt_events_list_cards and 'card-footer' or (not opt_events_list_columns and 'py-2 mt-2') or 'py-2',
                            opt_events_list_cards and 'border-top' or 'px-2',
                        )">
                        <span t-if="not event.event_registrations_open">
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
    <div class="d-flex justify-content-center my-3">
        <t t-call="website.pager"/>
    </div>
</template>

<template id="opt_events_list_columns" inherit_id="website_event.events_list" active="True" name="Layout • Columns"/>

<template id="opt_events_list_cards" inherit_id="website_event.events_list" active="True" name="'Cards' Design"/>

<template id="opt_events_list_categories" inherit_id="website_event.events_list" active="False" name="Show Templates">
    <xpath expr="//main/div[@id='event_details']" position="after">
        <span t-if="event.event_type_id" t-attf-href="/event?type=#{event.event_type_id.id}" t-attf-class="badge text-bg-secondary o_wevent_badge #{opt_events_list_columns and 'o_wevent_badge_event' or 'position-absolute bottom-0 end-0 end-sm-0 start-sm-0'} #{not opt_events_list_columns and opt_events_list_cards and 'me-sm-3 mb-sm-3'}" t-field="event.event_type_id"/>
    </xpath>
</template>

<!-- Index - Sidebar -->
<template id="index_sidebar" name="Sidebar">
    <div id="o_wevent_index_sidebar" class="col-lg-4 ms-lg-3 ps-xxl-5 mb-5"/>
</template>

<!-- Index - Sidebar - About us -->
<template id="index_sidebar_about_us" inherit_id="website_event.index_sidebar" active="True" name="About us" priority="20">
    <xpath expr="//div[@id='o_wevent_index_sidebar']" position="inside">
        <div class="o_wevent_sidebar_block">
            <h6 class="o_wevent_sidebar_title">About us</h6>
            <p>Use this paragraph to write a short text about your events or company.</p>
        </div>
        <div id="oe_structure_website_event_about_us_1" class="oe_structure"/>
    </xpath>
</template>

<!-- Index - Sidebar - Follow us -->
<template id="index_sidebar_follow_us" inherit_id="website_event.index_sidebar" active="False" name="Follow us" priority="30">
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
                <a t-if="website.social_tiktok" t-att-href="website.social_tiktok" class="o_wevent_social_link"><i class="fa fa-tiktok text-tiktok" aria-label="TikTok" title="TikTok"/></a>
            </div>
        </div>
        <div id="oe_structure_website_event_follow_us_1" class="oe_structure"/>
    </xpath>
</template>

<!-- Index - Sidebar - Photos -->
<template id="index_sidebar_photos" inherit_id="website_event.index_sidebar" active="True" name="Photos" priority="40">
    <xpath expr="//div[@id='o_wevent_index_sidebar']" position="inside">
        <h6 class="o_wevent_sidebar_title">Photos</h6>
        <a href="/event">
            <figure class="o_wevent_sidebar_block o_wevent_sidebar_figure figure">
                <img class="figure-img img-fluid rounded oe_unremovable" src="/website_event/static/src/img/event_past_0.jpg" alt=""/>
                <figcaption class="figure-caption oe_unremovable">A past event</figcaption>
            </figure>
        </a>
        <a href="/event">
            <figure class="o_wevent_sidebar_block o_wevent_sidebar_figure figure">
                <img class="figure-img img-fluid rounded oe_unremovable" src="/website_event/static/src/img/event_training_0.jpg" alt=""/>
                <figcaption class="figure-caption oe_unremovable">Our Trainings</figcaption>
            </figure>
        </a>
    </xpath>
</template>

<!-- Index - Sidebar - Quotes -->
<template id="index_sidebar_quotes" inherit_id="website_event.index_sidebar" active="True" name="Quotes" priority="60">
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
        <div id="wrap" t-attf-class="o_wevent_event js_event d-flex flex-column h-100 #{'o_wevent_hide_sponsors' if hide_sponsors else ''}">
            <t t-if="not hide_submenu" t-call="website_event.navbar"/>
            <t t-out="0"/>
            <t t-set="editor_sub_message">Following content will appear on all events.</t>
            <div class="oe_structure oe_empty" id="oe_structure_website_event_layout_1" t-att-data-editor-sub-message="editor_sub_message"/>
        </div>
    </t>
</template>

<template id="navbar" name="Event Navbar">
    <div class="container my-3">
        <a href="/event">
            <i class="fa fa-chevron-left me-2"/>
            <span>Back to events</span> 
        </a>
    </div>
    <div class="mb-3">
        <div class="container d-flex flex-wrap flex-lg-nowrap justify-content-between align-items-center">
            <t t-set="active_submenu" t-value="event.menu_id.child_id.filtered(lambda sm: sm._is_active())"></t>
            <h6 t-field="event.name" t-attf-class="flex-grow-1 #{'my-2' if active_submenu else 'mb-0'}"/>
            <div class="d-flex flex-grow-1 align-items-center justify-content-end gap-2">
                <nav t-if="active_submenu" class="navbar navbar-light navbar-expand-md p-0 d-md-none">
                    <div class="container align-items-baseline">
                        <div id="o_wevent_event_submenu" class="dropdown nav-item">
                            <button class="dropdown-toggle btn btn-light" data-bs-toggle="dropdown">
                                <t t-out="active_submenu.name if len(active_submenu) == 1 else ''"/>
                            </button>
                            <ul class="dropdown-menu flex-md-wrap w-100 overflow-x-hidden" t-att-data-menu_name="editable and 'Event Menu'" t-att-data-content_menu_id="editable and event.menu_id.id">
                                <t t-foreach="event.menu_id.child_id" t-as="submenu">
                                    <t t-call="website.submenu">
                                        <t t-set="link_class" t-value="'dropdown-item'"/>
                                    </t>
                                </t>
                            </ul>
                        </div>
                    </div>
                </nav>
                <!-- Add Register additional CTA button, in addition to menus -->   
                <a t-if="event.menu_register_cta and not event.is_participating"
                    t-att-href="'/event/%s/register' % (slug(event))"
                    t-attf-class="btn btn-primary #{'d-none' if hide_register_cta else ''}">
                    Register
                </a>
            </div>
        </div>
        <nav class="navbar navbar-light navbar-expand-md d-none d-md-block p-0">
            <div class="container align-items-baseline">
                <button class="navbar-toggler ms-auto" type="button" data-bs-toggle="collapse" data-bs-target="#o_wevent_event_submenu" aria-controls="o_wevent_event_submenu" aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div id="o_wevent_event_submenu" class="collapse navbar-collapse">
                    <ul class="navbar-nav flex-md-wrap w-100" t-att-data-menu_name="editable and 'Event Menu'" t-att-data-content_menu_id="editable and event.menu_id.id">
                        <t t-foreach="event.menu_id.child_id" t-as="submenu">
                            <t t-call="website.submenu">
                                <t t-set="item_class" t-value="'nav-item'"/>
                                <t t-set="link_class" t-value="'nav-link'"/>
                            </t>
                        </t>
                    </ul>
                </div>
            </div>
        </nav>
        
    </div>
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
        <div class="oe_structure">
            <section class="s_title pt32 pb32" data-vcss="001" data-snippet="s_title" data-name="Title">
                <div class="container s_allow_columns">
                    <h1 class="display-3 o_default_snippet_text" style="text-align: center;">Introduction</h1>
                </div>
            </section>
        </div>
        <div class="oe_structure oe_empty" id="oe_structure_website_event_intro_2"/>
    </t>
</template>

<!-- Multipage event - Default template for the Location page -->
<template id="template_location">
    <t t-call="website_event.layout">
        <div class="oe_structure" id="oe_structure_website_event_location_1"/>
        <section>
            <div class="container">
                <div class="row">
                    <div class="col-12">
                        <h3 class="mt-3 mb-4">Event Location</h3>
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
        <div class="h-100" name="event" itemscope="itemscope" itemtype="http://schema.org/Event">
            <meta itemprop="startDate" t-attf-content="{{event.date_begin}}Z"/>
            <meta itemprop="endDate" t-attf-content="{{event.date_end}}Z"/>
            <t t-out="0"/>
        </div>
    </t>
</template>

<!-- Event - Description -->
<template id="event_description_full" name="Event Description" track="1">
    <t t-set="hide_submenu" t-value="True"/>
    <t t-set="hide_register_cta" t-value="True"/>
    <t t-call="website_event.event_details">
        <section class="h-100" t-cache="event if not editable and event.website_published else None">
            <div class="container overflow-hidden pb-5">
                <div class="row h-100">
                <div class="col pe-xxl-5">
                    <t t-call="website_event.navbar"/>
                    <!-- Show the button tickets only on mobile -->
                    <div class="container d-lg-none">
                        <t t-call="website_event.registration_template">
                            <t t-set="toast_message" t-value=""/>
                        </t>
                    </div>
                    <!-- Description -->
                    <div id="o_wevent_event_main_col">
                        <t t-call="website.record_cover">
                            <t t-set="_record" t-value="event"/>
                            <t t-set="use_filters" t-value="True"/>
                            <t t-set="use_size" t-value="False"/>
                            <t t-set="use_text_align" t-value="True"/>
                            <div class="container d-flex flex-column flex-grow-1 justify-content-around">
                                <div class="o_wevent_event_title p-3 my-5">
                                    <span t-if="event.is_participating" class="badge text-bg-success o_wevent_badge"><i class="fa fa-check me-2"/>Registered</span>
                                    <h1 t-field="event.name" class="o_wevent_event_name" itemprop="name" placeholder="Event Title"/>
                                    <h2 t-field="event.subtitle" class="o_wevent_event_subtitle" placeholder="Event Subtitle"/>
                                </div>
                            </div>
                        </t>
                        <div class="mt-4" t-field="event.description" itemprop="description"/>
                    </div>
                </div>
                    <!-- Sidebar -->
                    <div class="d-lg-flex justify-content-end col-lg-4 col-xl-3 mb-3 mb-lg-0 d-print-none">
                        <div class="mt-3">
                            <!-- Tickets Desktop -->
                            <div class="container d-none d-lg-block">
                                <t t-call="website_event.registration_template"/>
                            </div>
                            <!-- Date & Time -->
                            <div class="o_wevent_sidebar_block border-bottom pb-3 my-3">
                                <h6 class="o_wevent_sidebar_title">Date &amp; Time</h6>
                                <div class="d-flex">
                                    <h6 t-field="event.date_begin" class="my-1" t-options="{'tz_name': event.date_tz, 'format': 'full', 'date_only': 'true'}" t-att-datetime="event.date_begin"/>
                                </div>
                                <t t-if="not event.is_one_day">Start -</t>
                                <span t-out="event.date_begin" t-options="{'widget': 'datetime', 'tz_name': event.date_tz, 'time_only': 'true', 'format': 'short'}"/>
                                <t t-if="event.is_one_day">
                                    <i class="fa fa-long-arrow-right mx-1"/>
                                </t>
                                <t t-else="">
                                    (<span t-out="event.date_tz"/>)
                                    <i class="fa fa-long-arrow-down d-block text-muted mx-3 my-2" style="font-size: 1.5rem"/>
                                    <div class="d-flex">
                                        <h6 t-field="event.date_end" class="my-1" t-options="{'tz_name': event.date_tz, 'format': 'full', 'date_only': 'true'}"/>
                                    </div>
                                    <t t-if="not event.is_one_day">End -</t>
                                </t>
                                <span t-if="not event.is_one_day" t-out="event.date_end" t-options="{'widget': 'datetime', 'tz_name': event.date_tz, 'time_only': 'true', 'format': 'short'}"/>
                                <span t-else="" t-field="event.date_end" t-options="{'tz_name': event.date_tz, 'time_only': 'true', 'format': 'short'}" t-att-datetime="event.date_end"/>
                                (<span t-out="event.date_tz"/>)

                                <a href="#" role="button" data-bs-toggle="dropdown" class="btn btn-secondary dropdown w-100 mt-3">
                                    <i class="fa fa-calendar me-2"/>Add to Calendar
                                </a>
                                <div class="dropdown-menu">
                                    <a t-att-href="iCal_url" class="dropdown-item">iCal/Outlook</a>
                                    <a t-att-href="google_url" class="dropdown-item" target="_blank">Google</a>
                                </div>
                            </div>
                            <!-- Location -->
                            <div t-if="event.address_id" class="o_wevent_sidebar_block border-bottom pb-3 mb-3">
                                <h6 class="o_wevent_sidebar_title">Location</h6>
                                <h4 t-field="event.address_id" class="font-sans-serif mt-0 mb-1" style="font-size: 1rem" t-options='{
                                    "widget": "contact",
                                    "fields": ["name"]
                                }'/>
                                <div itemprop="location" class="mb-2 small" t-field="event.address_id" t-options='{
                                    "widget": "contact",
                                    "fields": ["address"],
                                    "no_marker": True
                                }'/>
                                <div class="mb-2 small" t-field="event.address_id" t-options='{
                                    "widget": "contact",
                                    "fields": ["phone", "mobile", "email"]
                                }'/>
                                <a t-att-href="event._google_map_link()" target="_blank" class="btn btn-secondary w-100">
                                    <i class="fa fa-map-marker fa-fw" role="img"/>Get the direction
                                </a>
                            </div>
                            <!-- Organizer -->
                            <div t-if="event.organizer_id" class="o_wevent_sidebar_block border-bottom pb-3 mb-3">
                                <h6 class="o_wevent_sidebar_title">Organizer</h6>
                                <h4 t-field="event.organizer_id" class="font-sans-serif mt-0 mb-1" style="font-size: 1rem"/>
                                <div class="small" itemprop="location" t-field="event.organizer_id" t-options="{'widget': 'contact', 'fields': ['phone', 'mobile', 'email']}"/>
                            </div>
                            <!-- Social -->
                            <div class="o_wevent_sidebar_block">
                                <h6 class="o_wevent_sidebar_title">Share</h6>
                                <p>Find out what people see and say about this event, and join the conversation.</p>
                                <t t-snippet-call="website.s_share">
                                    <t t-set="_no_title" t-value="True"/>
                                    <t t-set="_classes" t-valuef="o_wevent_sidebar_social mx-n1"/>
                                    <t t-set="_link_classes" t-valuef="o_wevent_social_link"/>
                                </t>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>
        <t t-call="website_event.modal_ticket_registration"/>
    </t>
</template>

<!-- Event - Registration -->
<template id="registration_template" name="Registration">
    <!-- Show the tickets button only on desktop -->
    <div t-attf-class="d-flex d-lg-block flex-wrap justify-content-between align-items-center {{event.event_registrations_open and 'pb-3 border-bottom' or ''}}">
        <div class="d-flex flex-wrap justify-content-between align-items-center w-100 mb-3">
            <h4 class="mb-0">Tickets</h4>
            <t t-call="website_event.registration_configure_tickets_button" t-if="request.env.user.has_group('event.group_event_manager')">
                <t t-set="linkClasses" t-translation="off">ms-2</t>
            </t>
        </div>
        <button t-if="event.event_registrations_open" type="button" data-bs-toggle="modal" data-bs-target="#modal_ticket_registration" class="btn btn-primary w-100">Register</button>
        <div t-if="registration_error_code == 'insufficient_seats'" class="alert alert-danger my-3" role="alert">
            Registration failed! These tickets are not available anymore.
        </div>
    </div>
    <div t-if="toast_message" class="o_wevent_register_toaster d-none" t-att-data-message="toast_message"/>
    <div t-if="not event.event_registrations_open" class="mb-3">
        <div class="alert alert-info mb-0 d-flex flex-wrap gap-2 justify-content-between align-items-center" role="status">
            <t t-if="not event.event_registrations_started and not event.event_registrations_sold_out">
                <div class="d-flex flex-column">
                    <em class="me-2">Ticket Sales starting on
                        <span class="" t-out="event.start_sale_datetime"
                            t-options="{'widget': 'datetime', 'tz_name': event.date_tz, 'format': 'short'}"/>
                        <span t-out="event.date_tz"/>
                    </em>
                    <t t-call="website_event.registration_configure_tickets_button"
                        t-if="request.env.user.has_group('event.group_event_manager')">
                            <t t-set="linkClasses" t-translation="off">alert-link my-3</t>
                    </t>
                </div>
                <button class="btn btn-danger" disabled="1">Registrations not yet open</button>
            </t>
            <t t-else="">
                <div>
                    <em t-if="event.event_registrations_sold_out">Tickets for this Event are <b>Sold Out</b></em>
                    <em t-else="">Registrations are <b>closed</b></em>
                </div>
                <button class="btn btn-danger py-2 w-100" disabled="1">
                    <span t-if="event.event_registrations_sold_out">Sold Out</span>
                    <span t-else="">Registrations Closed</span>
                </button>
            </t>
        </div>
    </div>
</template>

<template id="registration_attendee_details" name="Registration Attendee Details">
    <div id="modal_attendees_registration" class="modal fade" tabindex="-1" role="dialog">
        <div class="modal-dialog modal-lg" role="document">
            <form id="attendee_registration" t-attf-action="/event/#{slug(event)}/registration/confirm" method="post" class="js_website_submit_form">
                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                <div class="modal-content">
                    <div class="modal-header align-items-center">
                        <h4 class="modal-title">Attendees</h4>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"/>
                    </div>
                    <t t-set="counter" t-value="0"/>
                    <t t-set="input_type_by_question_type" t-value="{'name': 'text', 'email': 'email', 'phone': 'tel', 'company_name': 'text'}"/>
                    <t t-if="availability_check" t-foreach="tickets" t-as="ticket">
                        <t t-foreach="range(1, ticket['quantity'] + 1)" t-as="att_counter" name="attendee_loop">
                            <t t-set="counter" t-value="counter + 1"/>
                            <div class="modal-body">
                                <h5 t-attf-class="mt-1 pb-2 #{'border-bottom' if event.question_ids else ''}">Ticket #<span t-out="counter"/> <small class="text-muted">- <span t-out="ticket['name']"/></small></h5>
                                <div t-if="event.specific_question_ids" class="row">
                                    <t t-foreach="event.specific_question_ids" t-as="question">
                                        <div class="col-lg-6 mt-2">
                                            <t t-call="website_event.registration_event_question">
                                                <t t-set="registration_index" t-value="counter"/>
                                            </t>
                                        </div>
                                    </t>
                                </div>
                                <input class="d-none" type="text" t-attf-name="#{counter}-event_ticket_id" t-attf-value="#{ticket['id']}"/>
                            </div>
                        </t>
                    </t>
                    <div t-if="availability_check and event.general_question_ids" class="modal-body border-top o_wevent_registration_question_global">
                        <div class="row">
                            <t t-foreach="event.general_question_ids" t-as="question">
                                <div class="mt-2" t-att-class="question.question_type=='text_box' and 'col-lg-12' or 'col-lg-6'">
                                    <t t-call="website_event.registration_event_question">
                                        <t t-set="registration_index" t-value="0"/>
                                    </t>
                                </div>
                            </t>
                        </div>
                    </div>
                    <t t-elif="not availability_check">
                        <div class="modal-body border-bottom">
                            <strong> You ordered more tickets than available seats</strong>
                        </div>
                    </t>
                    <div class="modal-footer border-top">
                        <button type="button" class="btn btn-secondary js_goto_event" data-bs-dismiss="modal">Cancel</button>
                        <button type="submit" class="btn btn-primary" t-if="availability_check">Confirm Registration</button>
                    </div>
                </div>
            </form>
        </div>
    </div>
</template>

<template id="registration_event_question" name="Registration Event Question">
    <label t-out="question.title"/>
    <span t-if="question.is_mandatory_answer">*</span>
    <t t-if="question.question_type in ['name', 'email', 'phone', 'company_name']">
        <input class="form-control" t-att-type="input_type_by_question_type[question.question_type]"
               t-attf-name="#{registration_index}-#{question.question_type}-#{question.id}"
               t-att-required="question.is_mandatory_answer" t-att-value="default_first_attendee.get(question.question_type, '') if counter in (0,1) else ''"/>
    </t>
    <t t-elif="question.question_type == 'simple_choice'">
        <select t-attf-name="#{registration_index}-#{question.question_type}-#{question.id}"
                class="form-select" t-att-required="question.is_mandatory_answer">
            <option value=""/>
            <t t-foreach="question.answer_ids" t-as="answer">
                <option t-out="answer.name" t-att-value="answer.id"/>
            </t>
        </select>
    </t>
    <t t-else="">
        <textarea t-attf-name="#{registration_index}-#{question.question_type}-#{question.id}"
                  class="col-lg-12 form-control" t-att-required="question.is_mandatory_answer"/>
    </t>
</template>

<template id="registration_complete" name="Registration Completed">
    <t t-call="website_event.layout">
        <div class="container my-5 o_wereg_confirmed">
            <div class="row mb-3">
                <div class="col-12">
                    <h3>Registration confirmed!</h3>
                    <u><a class="h4 text-primary" t-out="event.name" t-att-href="event.website_url" /></u>
                </div>
            </div>
            <div t-if="attendees" class="row mb-3">
                <div class="col-12 mb-2">
                    <a class="btn btn-primary" title="Download All Tickets" target="_blank"
                        t-attf-href="/event/{{ event.id }}/my_tickets?registration_ids={{ attendees.ids }}&amp;tickets_hash={{ event._get_tickets_access_hash(attendees.ids) }}">
                        Download Tickets <i class="ms-1 fa fa-download"/>
                    </a>
                </div>
            </div>
            <div class="row mb-3 o_wereg_confirmed_attendees">
                <div class="col-md-4 col-xs-12 mt-3" t-foreach="attendees" t-as="attendee">
                    <div class="d-flex flex-column">
                        <span t-if="attendee.name"  class="fw-bold text-truncate">
                            <t t-out="attendee.name"/>
                        </span>
                        <span t-if="attendee.email" class="text-truncate">
                            <i class="fa fa-envelope me-2"/>
                            <t t-out="attendee.email"/>
                        </span>
                        <span t-if="attendee.phone">
                            <i class="fa fa-phone me-2"/>
                            <t t-out="attendee.phone"/>
                        </span>
                        <span>
                            <i class="fa fa-ticket me-2"/>
                            <t t-if="attendee.event_ticket_id">
                                <t t-out="attendee.event_ticket_id.name"/> (Ref: <t t-out="attendee.id"/>)
                            </t>
                            <t t-else="">Ref: <t t-out="attendee.id"/></t>
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
                        <div class="col ps-0">
                            <span itemprop="startDate" t-out="event.date_begin_located"/>
                            (<span t-out="event.date_tz"/>)
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-2 col-md-1">
                            <b>End</b>
                        </div>
                        <div class="col ps-0">
                            <span itemprop="endDate" t-out="event.date_end_located"/>
                            (<span t-out="event.date_tz"/>)
                        </div>
                    </div>
                    <div class="mt-4">
                        <h5 t-field="event.address_id" class="text-secondary fw-bold" t-options='{
                            "widget": "contact",
                            "fields": ["name"]
                            }'/>
                        <a itemprop="location" t-att-href="event.google_map_link()" target="_BLANK" temprop="location" t-field="event.address_id" t-options='{
                            "widget": "contact",
                            "fields": ["address"]
                            }'/>
                        <div itemprop="contactInfo" t-field="event.organizer_id" t-options='{
                            "widget": "contact",
                            "fields": ["phone", "mobile", "email"]
                            }'/>
                    </div>
                    <div id="add_to_calendar" class="mt-4 d-flex flex-column flex-md-row">
                        <a role="button" class="btn btn-primary" t-att-href="iCal_url">
                            <i class="fa fa-fw fa-calendar"/> Add to iCal/Outlook
                        </a>
                        <a role="button" class="btn btn-primary ms-md-2 mt-2 mt-md-0" t-att-href="google_url" target='_blank'>
                            <i class="fa fa-fw fa-calendar"/> Add to Google Calendar
                        </a>
                    </div>
                </div>
            </div>
        </div>
        <input t-if='website.plausible_shared_key' type='hidden' class='js_plausible_push' data-event-name='Lead Generation' t-attf-data-event-params='{"CTA": "Event Registration"}' />
    </t>
</template>

<!-- Button to configure Tickets -->
<template id="registration_configure_tickets_button" name="Registration Configure Ticket Button">
    <a t-attf-class="o_not_editable text-nowrap {{linkClasses or '' }}" t-attf-href="/web#id=#{event.id}&amp;menu_id=#{backend_menu_id}&amp;view_type=form&amp;model=event.event" role="link"  title="Configure event tickets">
        <i class="fa fa-gear me-1" role="img" aria-label="Configure" title="Configure event tickets"/><em>Configure Tickets</em>
    </a>
</template>

<template id="modal_ticket_registration" name="Modal for tickets registration">
    <!-- Modal -->
    <div class="modal fade" id="modal_ticket_registration" data-bs-backdrop="static" data-bs-keyboard="false" tabindex="-1" aria-labelledby="staticBackdropLabel" aria-hidden="true">
        <div class="modal-dialog">
            <div class="modal-content">
            <div class="modal-header">
                <div class="o_wevent_registration_title modal-title fs-5" id="staticBackdropLabel">Tickets</div>
                <div t-if="len(event.event_ticket_ids) &gt; 2" class="o_wevent_price_range ms-2"/>
                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>

            <form t-if="event.event_registrations_open and (not event.event_ticket_ids or any(not ticket.is_expired for ticket in event.event_ticket_ids))"
                id="registration_form"
                t-attf-action="/event/#{slug(event)}/registration/new" method="post"
                itemscope="itemscope" itemprop="offers" itemtype="http://schema.org/AggregateOffer">
                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                <div id="o_wevent_tickets" class="shadow-sm o_wevent_js_ticket_details">
                    <div t-if="registration_error_code == 'insufficient_seats'" class="alert alert-danger" role="alert">
                        <p class="mb-0">
                            Registration failed! These tickets are not available anymore.
                        </p>
                    </div>
                    <t t-set="tickets" t-value="event.event_ticket_ids.filtered(lambda ticket: not ticket.is_expired)"/>
                    <!-- If some tickets expired and there is only one type left, we keep the same layout -->
                    <t t-if="len(event.event_ticket_ids) &gt; 1">
                        <span t-if="not event.event_registrations_open" class="text-danger">
                            <i class="fa fa-ban me-2"/>Sold Out
                        </span>
                        <div id="o_wevent_tickets_collapse" class="modal-body collapse show">
                            <div t-foreach="tickets" t-as="ticket"
                                t-attf-class="d-flex justify-content-between o_wevent_ticket_selector mb-2 pb-2 {{not ticket_last and 'border-bottom' or ''}}"
                                t-att-name="ticket.name">
                                <div itemscope="itemscope" itemtype="http://schema.org/Offer">
                                    <h5 itemprop="name" t-field="ticket.name" class="h6 my-0"/>
                                    <t t-if="ticket.description">
                                        <small t-field="ticket.description" class="text-muted py-2"/>
                                        <br/>
                                    </t>
                                    <small t-if="ticket.end_sale_datetime and ticket.sale_available and not ticket.is_expired"
                                        class="text-muted me-3" itemprop="availabilityEnds">Sales end on
                                        <span itemprop="priceValidUntil" t-out="ticket.end_sale_datetime"
                                        t-options="{'widget': 'datetime', 'tz_name': event.date_tz, 'format': 'short'}"/>
                                        (<span t-out="ticket.event_id.date_tz"/>)
                                    </small>
                                    <small t-if="ticket.start_sale_datetime and not ticket.sale_available and not ticket.is_expired"
                                        class="text-muted me-3" itemprop="availabilityEnds">
                                        Sales start on <span itemprop="priceValidUntil" t-out="ticket.start_sale_datetime"
                                        t-options="{'widget': 'datetime', 'tz_name': event.date_tz, 'format': 'short'}"/>
                                        (<span t-out="ticket.event_id.date_tz"/>)
                                    </small>
                                </div>
                                <div class="d-flex flex-column flex-md-row align-items-center justify-content-between gap-2">
                                    <div class="o_wevent_registration_multi_select flex-md-grow-1 text-end"/>
                                    <div class="ms-auto">
                                        <select t-if="not ticket.is_expired and ticket.sale_available"
                                            t-attf-name="nb_register-#{ticket.id}"
                                            class="w-auto form-select">
                                            <t t-set="seats_max_ticket" t-value="(not ticket.seats_limited or ticket.seats_available &gt; 9) and 10 or ticket.seats_available + 1"/>
                                            <t t-set="seats_max_event" t-value="(not event.seats_limited or event.seats_available &gt; 9) and 10 or event.seats_available + 1"/>
                                            <t t-set="seats_max" t-value="min(seats_max_ticket, seats_max_event)"/>
                                            <t t-foreach="range(0, seats_max)" t-as="nb">
                                                <option t-out="nb" t-att-selected="len(ticket) == 0 and nb == 0 and 'selected'"/>
                                            </t>
                                        </select>
                                        <div t-else="" class="text-danger">
                                            <span t-if="not ticket.sale_available and not ticket.is_expired and ticket.is_launched" >Sold Out</span>
                                            <span t-if="ticket.is_expired">Expired</span>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="modal-footer flex-lg-row border-top">
                            <button type="button" class="btn btn-light" data-bs-dismiss="modal">Close</button>
                            <button type="submit" class="btn btn-primary o_wait_lazy_js a-submit" disabled="" t-attf-id="#{event.id}">
                                Register
                                <t t-if="event.seats_limited and event.seats_max and event.seats_available &lt;= (event.seats_max * 0.2)">
                                    (only <t t-out="event.seats_available"/> available)
                                </t>
                            </button>
                        </div>
                    </t>
                    <div t-else="" class="o_wevent_registration_single">
                        <div class="modal-body row px-3 py-2 mx-0">
                            <div class="col-12 col-md-8 p-0" itemscope="itemscope" itemtype="http://schema.org/Offer">
                                <h5 itemprop="name" class="my-0 pe-3 o_wevent_single_ticket_name">
                                    <span t-if="tickets" t-field="tickets.name"/>
                                    <span t-else="">Registration</span>
                                </h5>
                                <t t-if="tickets.description">
                                    <small t-field="tickets.description" class="text-muted py-2"/>
                                    <br/>
                                </t>
                                <small t-if="tickets.end_sale_datetime and tickets.sale_available and not tickets.is_expired"
                                    class="text-muted" itemprop="availabilityEnds">
                                    Sales end on
                                    <span itemprop="priceValidUntil" t-out="tickets.end_sale_datetime"
                                        t-options="{'widget': 'datetime', 'tz_name': event.date_tz, 'format': 'short'}"/>
                                    (<span t-out="tickets.event_id.date_tz"/>)
                                </small>
                            </div>
                            <div class="col-md-4 d-flex align-items-center justify-content-between p-0">
                                <t t-if="event.event_registrations_open">
                                    <link itemprop="availability" content="http://schema.org/InStock"/>
                                    <div class="o_wevent_registration_single_select w-auto ms-auto">
                                        <select t-att-name="'nb_register-%s' % (tickets.id if tickets else 0)" class="d-inline w-auto form-select">
                                            <t t-set="seats_max_ticket" t-value="(not tickets or not tickets.seats_limited or tickets.seats_available &gt; 9) and 10 or tickets.seats_available + 1"/>
                                            <t t-set="seats_max_event" t-value="(not event.seats_limited or event.seats_available &gt; 9) and 10 or event.seats_available + 1"/>
                                            <t t-set="seats_max" t-value="min(seats_max_ticket, seats_max_event) if tickets else seats_max_event"/>
                                            <t t-foreach="range(0, seats_max)" t-as="nb">
                                                <option t-out="nb" t-att-selected="nb == 1 and 'selected'"/>
                                            </t>
                                        </select>
                                    </div>
                                </t>
                                <t t-else="">
                                    <span itemprop="availability" content="http://schema.org/SoldOut" class="text-danger">
                                        <i class="fa fa-ban me-2"/>Sold Out
                                    </span>
                                </t>
                            </div>
                        </div>
                        <div class="modal-footer flex-lg-row border-top">
                            <button type="button" class="btn btn-light" data-bs-dismiss="modal">Close</button>
                            <button type="submit" class="btn btn-primary o_wait_lazy_js a-submit" t-attf-id="#{event.id}" disabled="disabled">
                                Register
                                <t t-if="event.seats_limited and event.seats_max and event.seats_available &lt;= (event.seats_max * 0.2)">
                                    (only <t t-out="event.seats_available"/> available)
                                </t>
                            </button>
                        </div>
                    </div>
                </div>
            </form>
            </div>
        </div>
    </div>
</template>

</odoo>

```

## File: views\event_templates_svg.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="website_event.event_empty_events_svg">
        <svg width="211" height="179" viewBox="0 0 211 179" fill="none" xmlns="http://www.w3.org/2000/svg">
            <g fill="var(--o-color-1, #000)">
                <path d="M189.68 38.15C189.98 37.78 189.92 37.24 189.55 36.94L185.07 33.34C184.7 33.04 184.16 33.1 183.86 33.47C183.56 33.84 183.62 34.38 183.99 34.68L188.47 38.28C188.84 38.58 189.38 38.52 189.68 38.15Z"/>
                <path d="M175.07 114.26H173.06V112.25C173.06 111.77 172.67 111.39 172.2 111.39C171.73 111.39 171.34 111.78 171.34 112.25V114.26H169.33C168.85 114.26 168.47 114.65 168.47 115.12C168.47 115.59 168.86 115.98 169.33 115.98H171.34V117.99C171.34 118.47 171.73 118.85 172.2 118.85C172.67 118.85 173.06 118.46 173.06 117.99V115.98H175.07C175.55 115.98 175.93 115.59 175.93 115.12C175.93 114.65 175.54 114.26 175.07 114.26Z"/>
                <path d="M6.6 101.9H4.59V99.89C4.59 99.41 4.2 99.03 3.73 99.03C3.26 99.03 2.87 99.42 2.87 99.89V101.9H0.86C0.38 101.9 0 102.29 0 102.76C0 103.23 0.39 103.62 0.86 103.62H2.87V105.63C2.87 106.11 3.26 106.49 3.73 106.49C4.2 106.49 4.59 106.1 4.59 105.63V103.62H6.6C7.08 103.62 7.46 103.23 7.46 102.76C7.46 102.29 7.07 101.9 6.6 101.9Z"/>
                <path d="M136.57 2.87H134.56V0.86C134.56 0.38 134.17 0 133.7 0C133.23 0 132.84 0.39 132.84 0.86V2.87H130.83C130.35 2.87 129.97 3.26 129.97 3.73C129.97 4.2 130.36 4.59 130.83 4.59H132.84V6.6C132.84 7.08 133.23 7.46 133.7 7.46C134.17 7.46 134.56 7.07 134.56 6.6V4.59H136.57C137.05 4.59 137.43 4.2 137.43 3.73C137.43 3.26 137.04 2.87 136.57 2.87Z"/>
                <path d="M16.41 60.22C16.07 60.56 16.07 61.1 16.41 61.44L20.47 65.5C20.81 65.84 21.35 65.84 21.69 65.5C22.03 65.16 22.03 64.62 21.69 64.28L17.63 60.22C17.29 59.88 16.75 59.88 16.41 60.22Z"/>
                <path d="M83.97 6.25C83.63 5.91 83.09 5.91 82.75 6.25L78.69 10.31C78.35 10.65 78.35 11.19 78.69 11.53C79.03 11.87 79.57 11.87 79.91 11.53L83.97 7.47C84.31 7.13 84.31 6.59 83.97 6.25Z"/>
                <path fill-rule="evenodd" clip-rule="evenodd" d="M55.59 45.22C54.64 45.22 53.87 44.45 53.87 43.5C53.87 42.55 54.64 41.78 55.59 41.78C56.54 41.78 57.31 42.55 57.31 43.5C57.31 44.45 56.54 45.22 55.59 45.22ZM55.59 46.37C54 46.37 52.72 45.08 52.72 43.5C52.72 41.92 54.01 40.63 55.59 40.63C57.17 40.63 58.46 41.92 58.46 43.5C58.46 45.08 57.17 46.37 55.59 46.37Z"/>
                <path fill-rule="evenodd" clip-rule="evenodd" d="M208.09 83.22C207.14 83.22 206.37 82.45 206.37 81.5C206.37 80.55 207.14 79.78 208.09 79.78C209.04 79.78 209.81 80.55 209.81 81.5C209.81 82.45 209.04 83.22 208.09 83.22ZM208.09 84.37C206.5 84.37 205.22 83.08 205.22 81.5C205.22 79.92 206.51 78.63 208.09 78.63C209.67 78.63 210.96 79.92 210.96 81.5C210.96 83.08 209.67 84.37 208.09 84.37Z"/>
            </g>
                <path fill="var(--o-color-1, #000)" opacity=".1" d="M105.47 179C149.377 179 184.97 172.732 184.97 165C184.97 157.268 149.377 151 105.47 151C61.5633 151 25.97 157.268 25.97 165C25.97 172.732 61.5633 179 105.47 179Z"/>
                <path fill="var(--o-color-1, #000)" opacity=".5" d="M177.31 57.83C172.39 62.1 164.94 61.57 160.67 56.65C156.4 51.73 156.93 44.28 161.85 40.01H161.87C162.18 39.73 162.21 39.26 161.94 38.95L154.35 30.2C151.05 26.4 145.29 25.99 141.49 29.29L55.29 104.06C51.49 107.36 51.08 113.12 54.38 116.93L62.17 125.91C62.49 126.28 63.04 126.3 63.4 125.98C68.32 121.71 75.77 122.24 80.04 127.16C84.31 132.08 83.78 139.53 78.86 143.8C78.49 144.12 78.44 144.67 78.76 145.03L86.27 153.69C89.57 157.5 95.33 157.9 99.13 154.6L185.32 79.81C189.12 76.51 189.53 70.75 186.23 66.94L178.36 57.87C178.09 57.55 177.61 57.53 177.3 57.81L177.31 57.83Z"/>
            <g fill="var(--o-color-1, #000)">
                <path d="M143.63 54.96L140.73 51.62C139.88 50.64 138.39 50.53 137.41 51.39L95.07 88.13C94.09 88.98 93.98 90.47 94.84 91.45L97.74 94.79C98.59 95.77 100.08 95.88 101.06 95.03L143.4 58.29C144.38 57.44 144.49 55.95 143.63 54.97V54.96Z"/>
                <path d="M134.52 75.16C133.67 74.18 132.18 74.07 131.2 74.93L104.06 98.48C103.08 99.33 102.97 100.82 103.82 101.8C104.67 102.78 106.16 102.89 107.14 102.03L134.28 78.48C135.26 77.63 135.37 76.14 134.52 75.16Z"/>
                <path d="M155.8 68.98C154.95 68 153.46 67.89 152.48 68.75L110.14 105.49C109.16 106.34 109.05 107.83 109.9 108.81C110.75 109.79 112.24 109.9 113.22 109.04L155.56 72.3C156.54 71.45 156.65 69.96 155.8 68.98Z"/>
                <path d="M150.92 82.4L116.23 112.51C115.25 113.36 115.14 114.85 115.99 115.83C116.84 116.81 118.33 116.91 119.31 116.06L154 85.95C154.98 85.1 155.09 83.61 154.24 82.63C153.39 81.65 151.9 81.54 150.92 82.4Z"/>
            </g>
            <g fill="var(--o-color-1, #000)">
                <path d="M110.85 133.81C110.49 133.81 110.14 133.66 109.88 133.37L106.36 129.32C105.89 128.79 105.95 127.97 106.49 127.51C107.02 127.05 107.83 127.1 108.3 127.64L111.82 131.69C112.29 132.22 112.23 133.04 111.69 133.5C111.45 133.71 111.15 133.81 110.85 133.81ZM103.82 125.71C103.46 125.71 103.11 125.56 102.85 125.27L99.33 121.22C98.87 120.69 98.92 119.87 99.46 119.41C99.99 118.94 100.81 119 101.27 119.54L104.79 123.59C105.26 124.12 105.19 124.93 104.66 125.4C104.42 125.61 104.12 125.71 103.82 125.71ZM96.79 117.61C96.43 117.61 96.08 117.46 95.82 117.17L92.3 113.12C91.83 112.59 91.89 111.77 92.43 111.31C92.96 110.84 93.78 110.9 94.24 111.44L97.76 115.49C98.23 116.02 98.17 116.84 97.63 117.3C97.39 117.51 97.09 117.61 96.79 117.61ZM89.76 109.51C89.4 109.51 89.05 109.36 88.79 109.07L85.27 105.02C84.8 104.49 84.86 103.67 85.4 103.21C85.93 102.74 86.75 102.8 87.21 103.34L90.73 107.39C91.2 107.92 91.14 108.74 90.6 109.2C90.36 109.41 90.06 109.51 89.76 109.51ZM82.73 101.41C82.37 101.41 82.02 101.26 81.76 100.97L78.24 96.92C77.77 96.39 77.83 95.57 78.37 95.11C78.9 94.64 79.72 94.7 80.18 95.24L83.7 99.29C84.17 99.82 84.11 100.64 83.57 101.1C83.33 101.31 83.03 101.41 82.73 101.41Z"/>
                <path d="M93.18 158.32C90.21 158.32 87.25 157.08 85.16 154.67L77.65 146.01C77.23 145.52 77.03 144.91 77.08 144.27C77.14 143.65 77.43 143.08 77.9 142.67L77.94 142.64C79.99 140.85 81.24 138.35 81.43 135.62C81.63 132.87 80.74 130.22 78.94 128.14C75.22 123.85 68.7 123.39 64.41 127.11C63.93 127.52 63.33 127.73 62.71 127.7C62.07 127.67 61.48 127.38 61.07 126.89L53.28 117.91C51.42 115.77 50.51 113.03 50.71 110.2C50.91 107.37 52.2 104.79 54.34 102.93L140.5 28.15C144.92 24.32 151.64 24.79 155.47 29.21L163.06 37.96C163.45 38.41 163.64 38.99 163.6 39.59C163.56 40.19 163.28 40.73 162.84 41.12C158.53 44.86 158.07 51.38 161.79 55.67C165.51 59.96 172.03 60.42 176.32 56.7L176.36 56.67C176.8 56.29 177.36 56.11 177.94 56.14C178.55 56.17 179.1 56.45 179.5 56.9L187.37 65.97C189.23 68.11 190.14 70.85 189.94 73.68C189.74 76.51 188.45 79.09 186.31 80.95L100.13 155.74C98.12 157.48 95.65 158.33 93.18 158.33V158.32ZM80.31 144.52L87.42 152.71C90.17 155.89 95 156.23 98.18 153.47L184.36 78.68C185.9 77.34 186.82 75.49 186.97 73.46C187.12 71.43 186.46 69.46 185.12 67.93L177.73 59.42C172.2 63.72 164.19 62.98 159.54 57.63C154.89 52.28 155.3 44.24 160.33 39.37L153.22 31.17C150.47 27.99 145.64 27.65 142.46 30.41L56.27 105.2C54.73 106.54 53.81 108.39 53.66 110.42C53.51 112.45 54.17 114.42 55.51 115.95L62.89 124.46C68.43 120.08 76.51 120.79 81.18 126.17C83.51 128.85 84.65 132.28 84.4 135.82C84.17 139.16 82.71 142.23 80.3 144.52H80.31Z"/>
                <path d="M75.69 93.29C75.33 93.29 74.98 93.14 74.72 92.85L73.04 90.92C72.57 90.39 72.63 89.57 73.17 89.11C73.7 88.65 74.51 88.71 74.98 89.24L76.66 91.18C77.13 91.71 77.07 92.53 76.53 92.99C76.29 93.2 75.99 93.3 75.69 93.3V93.29Z"/>
                <path d="M116.05 139.79C115.69 139.79 115.34 139.64 115.08 139.35L113.4 137.42C112.94 136.89 113 136.07 113.53 135.61C114.06 135.14 114.88 135.2 115.34 135.74L117.02 137.67C117.49 138.2 117.43 139.02 116.89 139.48C116.65 139.69 116.35 139.79 116.05 139.79Z"/>
            </g>
            <path d="M137.79 54.44C133.73 59.54 126.31 60.38 121.21 56.33C116.11 52.27 115.27 44.85 119.32 39.75V39.73C119.59 39.41 119.53 38.94 119.21 38.68L110.14 31.47C106.2 28.34 100.46 28.99 97.33 32.93L26.31 122.23C23.18 126.17 23.83 131.91 27.77 135.04L37.07 142.44C37.45 142.74 37.99 142.67 38.3 142.29C42.36 137.19 49.78 136.35 54.88 140.4C59.98 144.46 60.82 151.88 56.77 156.98C56.47 157.36 56.52 157.91 56.89 158.21L65.86 165.35C69.8 168.48 75.54 167.83 78.67 163.89L149.71 74.59C152.84 70.65 152.19 64.91 148.25 61.78L138.86 54.31C138.54 54.05 138.07 54.11 137.81 54.44H137.79Z" fill="#FFFFFF"/>
            <path fill="var(--o-color-1, #000)" opacity=".3" d="M137.79 54.44C133.73 59.54 126.31 60.38 121.21 56.33C116.11 52.27 115.27 44.85 119.32 39.75V39.73C119.59 39.41 119.53 38.94 119.21 38.68L110.14 31.47C106.2 28.34 100.46 28.99 97.33 32.93L26.31 122.23C23.18 126.17 23.83 131.91 27.77 135.04L37.07 142.44C37.45 142.74 37.99 142.67 38.3 142.29C42.36 137.19 49.78 136.35 54.88 140.4C59.98 144.46 60.82 151.88 56.77 156.98C56.47 157.36 56.52 157.91 56.89 158.21L65.86 165.35C69.8 168.48 75.54 167.83 78.67 163.89L149.71 74.59C152.84 70.65 152.19 64.91 148.25 61.78L138.86 54.31C138.54 54.05 138.07 54.11 137.81 54.44H137.79Z"/>
            <g fill="var(--o-color-1, #000)">
                <path d="M63.6209 99.2776L98.5267 55.4106C99.3361 54.3934 100.812 54.2256 101.829 55.035L105.295 57.7933C106.313 58.6028 106.48 60.0782 105.671 61.0954L70.7651 104.962C69.9557 105.98 68.4802 106.147 67.463 105.338L63.9965 102.58C62.9793 101.77 62.8114 100.295 63.6209 99.2776Z"/>
                <path d="M100.03 79.31C99.01 78.5 97.54 78.67 96.73 79.69L74.36 107.81C73.55 108.83 73.72 110.31 74.74 111.11C75.76 111.92 77.24 111.75 78.04 110.73L100.41 82.61C101.22 81.59 101.05 80.11 100.03 79.31Z"/>
                <path d="M119.82 69.34C118.8 68.53 117.33 68.7 116.52 69.72L81.62 113.59C80.81 114.61 80.98 116.09 82 116.89C83.02 117.69 84.5 117.53 85.3 116.51L120.2 72.64C121.01 71.62 120.84 70.14 119.82 69.34Z"/>
                <path d="M120.78 83.04C119.76 82.23 118.29 82.4 117.48 83.42L88.88 119.37C88.07 120.39 88.24 121.87 89.26 122.67C90.28 123.48 91.75 123.31 92.56 122.29L121.15 86.34C121.96 85.32 121.79 83.84 120.77 83.04H120.78Z"/>
            </g>
            <g fill="var(--o-color-1, #000)">
                <path d="M45.28 107.93C45 107.93 44.72 107.84 44.48 107.65L42.47 106.05C41.91 105.61 41.83 104.8 42.26 104.25C42.7 103.7 43.51 103.61 44.06 104.05L46.07 105.65C46.63 106.09 46.71 106.9 46.28 107.45C46.03 107.77 45.65 107.93 45.28 107.93Z"/>
                <path d="M154.25 67.72C153.93 64.9 152.53 62.38 150.31 60.62L140.92 53.15C140.45 52.78 139.87 52.61 139.28 52.68C138.7 52.75 138.17 53.03 137.8 53.49L137.76 53.53C134.23 57.98 127.73 58.71 123.28 55.18C118.83 51.64 118.09 45.15 121.64 40.68C122.01 40.21 122.17 39.63 122.11 39.03C122.05 38.44 121.75 37.9 121.28 37.53L112.22 30.32C110 28.56 107.23 27.76 104.41 28.08C101.59 28.4 99.07 29.8 97.31 32.02L26.27 121.31C22.63 125.89 23.39 132.58 27.97 136.22L37.27 143.62C37.77 144.02 38.4 144.19 39.03 144.11C39.65 144.03 40.2 143.71 40.57 143.25L40.62 143.19C44.16 138.76 50.64 138.03 55.08 141.56C57.23 143.27 58.59 145.72 58.9 148.46C59.21 151.19 58.44 153.89 56.73 156.04L56.68 156.11C56.31 156.6 56.15 157.2 56.22 157.81C56.29 158.44 56.6 158.99 57.09 159.39L66.06 166.53C67.96 168.04 70.26 168.84 72.65 168.84C73.06 168.84 73.46 168.82 73.87 168.77C76.69 168.45 79.21 167.05 80.98 164.83L152.02 75.53C153.78 73.31 154.58 70.54 154.26 67.72H154.25ZM149.67 73.66L78.63 162.97C77.36 164.56 75.55 165.57 73.53 165.8C71.51 166.03 69.51 165.46 67.92 164.19L59.43 157.44C61.39 154.74 62.25 151.46 61.87 148.14C61.47 144.61 59.72 141.45 56.94 139.24C51.36 134.8 43.29 135.59 38.65 140.91L29.83 133.89C26.54 131.27 25.99 126.47 28.61 123.18L99.65 33.86C100.92 32.27 102.73 31.26 104.75 31.03C106.77 30.8 108.77 31.37 110.36 32.64L118.85 39.4C114.79 45.11 115.87 53.09 121.42 57.5C126.97 61.91 134.98 61.16 139.63 55.93L148.45 62.95C150.04 64.22 151.05 66.03 151.28 68.05C151.51 70.07 150.94 72.07 149.67 73.66Z"/>
                <path d="M58.67 115.67C58.11 115.23 57.31 115.32 56.87 115.88C56.43 116.43 56.52 117.24 57.07 117.68L61.27 121.02C61.51 121.21 61.79 121.3 62.07 121.3C62.45 121.3 62.82 121.13 63.07 120.82C63.51 120.27 63.42 119.46 62.87 119.02L58.67 115.68V115.67Z"/>
                <path d="M50.27 108.99C49.72 108.55 48.91 108.64 48.47 109.19C48.03 109.74 48.12 110.55 48.67 110.99L52.87 114.33C53.11 114.52 53.39 114.61 53.67 114.61C54.04 114.61 54.42 114.44 54.67 114.13C55.11 113.58 55.02 112.77 54.46 112.33L50.26 108.99H50.27Z"/>
                <path d="M67.07 122.34C66.52 121.9 65.71 121.99 65.27 122.55C64.83 123.1 64.92 123.91 65.47 124.35L69.67 127.69C69.91 127.88 70.19 127.97 70.47 127.97C70.84 127.97 71.22 127.8 71.47 127.49C71.91 126.94 71.82 126.13 71.26 125.69L67.06 122.35L67.07 122.34Z"/>
                <path d="M75.47 129.02C74.92 128.58 74.11 128.67 73.67 129.22C73.23 129.77 73.32 130.58 73.88 131.02L78.08 134.36C78.32 134.55 78.6 134.64 78.88 134.64C79.25 134.64 79.63 134.47 79.88 134.16C80.32 133.61 80.23 132.8 79.68 132.36L75.48 129.02H75.47Z"/>
                <path d="M83.87 135.7C83.31 135.26 82.51 135.35 82.07 135.91C81.63 136.47 81.72 137.27 82.28 137.71L86.48 141.05C86.72 141.24 87 141.33 87.28 141.33C87.66 141.33 88.03 141.16 88.28 140.85C88.72 140.3 88.63 139.49 88.07 139.05L83.87 135.71V135.7Z"/>
                <path d="M93.47 146.26C93.19 146.26 92.91 146.17 92.67 145.98L90.66 144.38C90.1 143.94 90.02 143.13 90.46 142.58C90.9 142.03 91.71 141.94 92.26 142.38L94.27 143.98C94.83 144.42 94.91 145.23 94.48 145.78C94.23 146.1 93.85 146.26 93.48 146.26H93.47Z"/>
            </g>
        </svg>
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
        <t t-set="_classes" t-valuef="o_wevent_event_searchbar_form flex-grow-1 #{_classes}"/>
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
        <form t-attf-class="o_wevent_event_searchbar_form o_wait_lazy_js w-100 flex-grow-1 #{_classes}"
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
            <span class="o_countdown_text pe-1" t-out="pre_countdown_text if pre_countdown_text else main_countdown_text if not pre_countdown_display else ''"/>
            <div t-if="int(remaining_time) > 86400"
             class="o_countdown_metric_container"><span class="o_countdown_remaining o_timer_days pe-1">0</span><span class="o_countdown_metric pe-1">days</span></div>
            <div t-if="int(remaining_time) > 3600"
                 class="o_countdown_metric_container"><span class="o_countdown_remaining o_timer_hours">00</span><span class="o_countdown_metric">:</span></div>
            <div class="o_countdown_metric_container"><span class="o_countdown_remaining o_timer_minutes">00</span><span class="o_countdown_metric">:</span></div>
            <div class="o_countdown_metric_container"><span class="o_countdown_remaining o_timer_seconds">00</span><span class="o_countdown_metric"></span></div>
        </span>
    </div>
</template>

<template id="display_timer_alert_widget" name="Display Countdown widget">
    <div class="o_we_track_timer alert alert-warning alert-dismissible fade show d-none mb-2" role="alert" t-att-data-time-to-live="time_to_live">
        Starts <span />
        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
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
                    <span name="menu_register_cta" class="d-inline-block">
                        <label for="menu_register_cta" string="Register Button"/>
                        <field name="menu_register_cta"/>
                    </span>
            </xpath>
            <page name="event_type_communication" position="after">
                <page string="Questions" name="page_questions">
                    <field name="question_ids" class="w-100">
                        <tree sample="1">
                            <field name="title"/>
                            <field name="is_mandatory_answer" string="Mandatory"/>
                            <field name="once_per_order" string="Once per Order"/>
                            <field name="question_type" />
                            <field name="answer_ids" widget="many2many_tags"/>
                        </tree>
                    </field>
                </page>
            </page>
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

## File: views\website_pages_views.xml

```xml
<?xml version="1.0"?>
<odoo>

<record id="event_pages_tree_view" model="ir.ui.view">
    <field name="name">Event Pages Tree</field>
    <field name="model">event.event</field>
    <field name="priority">99</field>
    <field name="mode">primary</field>
    <field name="inherit_id" ref="event.view_event_tree"/>
    <field name="arch" type="xml">
        <xpath expr="//tree" position="attributes">
            <attribute name="js_class">website_pages_list</attribute>
            <attribute name="type">object</attribute>
            <attribute name="action">open_website_url</attribute>
        </xpath>

        <field name="address_id" position="attributes">
            <attribute name="optional">hide</attribute>
        </field>
        <field name="date_end" position="attributes">
            <attribute name="optional">hide</attribute>
        </field>
        <field name="seats_used" position="attributes">
            <attribute name="optional">hide</attribute>
        </field>

        <field name="name" position="after">
            <field name="website_url"/>
        </field>
        <field name="stage_id" position="before">
            <field name="is_seo_optimized"/>
            <field name="is_published"/>

            <field name="website_id" position="move"/>
        </field>
    </field>
</record>

<record id="event_pages_kanban_view" model="ir.ui.view">
    <field name="name">Event Pages Kanban</field>
    <field name="model">event.event</field>
    <field name="priority">99</field>
    <field name="mode">primary</field>
    <field name="inherit_id" ref="event.view_event_kanban"/>
    <field name="arch" type="xml">
        <xpath expr="//kanban" position="attributes">
            <attribute name="js_class">website_pages_kanban</attribute>
            <attribute name="type">object</attribute>
            <attribute name="action">open_website_url</attribute>
        </xpath>
        <xpath expr="//kanban" position="inside">
            <field name="website_url" invisible="1"/>
        </xpath>
        <xpath expr="//div[hasclass('o_kanban_record_title')]" position="inside">
            <div class="text-muted" t-if="record.website_id.value" groups="website.group_multi_website">
                <i class="fa fa-globe me-1" title="Website"/>
                <field name="website_id"/>
            </div>
        </xpath>
        <xpath expr="//div[hasclass('o_kanban_record_bottom')]" position="after">
            <div class="border-top mt-2 pt-2">
                <field name="is_published" widget="boolean_toggle"/>
                <t t-if="record.is_published.raw_value">Published</t>
                <t t-else="">Not Published</t>
            </div>
        </xpath>
    </field>
</record>

<record id="action_event_pages_list" model="ir.actions.act_window">
    <field name="name">Event Pages</field>
    <field name="res_model">event.event</field>
    <field name="view_mode">tree,kanban</field>
    <field name="view_ids" eval="[(5, 0, 0),
        (0, 0, {'view_mode': 'tree', 'view_id': ref('event_pages_tree_view')}),
        (0, 0, {'view_mode': 'kanban', 'view_id': ref('event_pages_kanban_view')}),
    ]"/>
    <field name="context">{'create_action': 'website_event.event_event_action_add'}</field>
</record>

<menuitem id="menu_event_pages"
    parent="website.menu_content"
    sequence="40"
    name="Events"
    action="action_event_pages_list"/>

</odoo>

```

## File: views\website_visitor_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

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
                    invisible="event_registration_count == 0">
                    <field name="event_registration_count" widget="statinfo" string="Registrations"/>
                </button>
            </xpath>
        </field>
    </record>
</data></odoo>

```

## File: views\snippets\snippets.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo>

<template id="remove_external_snippets" inherit_id="website.external_snippets">
    <xpath expr="//t[@t-install='website_event']" position="replace"/>
</template>

<template id="snippets" inherit_id="website.snippets">
    <xpath expr="//t[@id='event_upcoming_snippet_hook']" position="replace">
        <t t-snippet="website_event.s_events" string="Events" t-thumbnail="/website/static/src/img/snippets_thumbs/s_event_upcoming_snippet.svg"/>
    </xpath>
</template>

<template id="snippet_options" inherit_id="website.snippet_options" name="Event snippet options">
    <xpath expr="." position="inside">
        <!-- Events page  -->
        <div data-selector="main:has(.o_wevent_events_list)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Events Page">
            <we-select string="Layout" data-no-preview="true" data-reload="/">
                <we-button data-customize-website-views="website_event.opt_events_list_columns">Grid</we-button>
                <we-button data-customize-website-views="">List</we-button>
            </we-select>
            <we-checkbox string="Card design"
                         data-customize-website-views="website_event.opt_events_list_cards"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Template badge"
                         data-customize-website-views="website_event.opt_events_list_categories"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-row string="Top Bar Filter" class="o_we_full_row">
                <we-button string="Date"
                           data-customize-website-views="website_event.event_time"
                           data-no-preview="true"
                           data-reload="/"/>
                <we-button string="Countries"
                           data-customize-website-views="website_event.event_location"
                           data-no-preview="true"
                           data-reload="/"/>
            </we-row>
            <we-checkbox string="Sidebar"
                         data-name="events_sidebar_opt"
                         data-customize-website-views="website_event.opt_index_sidebar"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="About Us"
                         class="o_we_sublevel_1"
                         data-dependencies="events_sidebar_opt"
                         data-customize-website-views="website_event.index_sidebar_about_us"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Follow Us"
                         class="o_we_sublevel_1"
                         data-dependencies="events_sidebar_opt"
                         data-customize-website-views="website_event.index_sidebar_follow_us"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Photos"
                         class="o_we_sublevel_1"
                         data-dependencies="events_sidebar_opt"
                         data-customize-website-views="website_event.index_sidebar_photos"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Quotes"
                         class="o_we_sublevel_1"
                         data-dependencies="events_sidebar_opt"
                         data-customize-website-views="website_event.index_sidebar_quotes"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
        <!-- Event page  -->
        <div data-js="WebsiteEvent" data-selector="main:has(.o_wevent_event)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Event Page">
            <we-checkbox string="Sub-menu (Specific)"
                         data-display-submenu="true"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
    </xpath>
</template>

<record id="website_event.s_searchbar_000_js" model="ir.asset">
    <field name="name">Searchbar 000 JS</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">website_event/static/src/snippets/s_searchbar/000.js</field>
</record>

<record id="website_event.s_searchbar_000_xml" model="ir.asset">
    <field name="name">Searchbar 000 XML</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">website_event/static/src/snippets/s_searchbar/000.xml</field>
</record>

</odoo>

```

## File: views\snippets\s_events.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo>

<template id="s_events" name="Events">
    <t t-call="website.s_dynamic_snippet_template">
        <t t-set="snippet_name" t-value="'s_events'"/>
        <t t-set="snippet_classes" t-value="'s_event_upcoming_snippet'"/>
    </t>
</template>

<!--TEMPLATES-->

<template id="dynamic_filter_template_event_event_picture" name="Picture Layout" priority="5">
    <figure t-foreach="records" t-as="data" class="w-100 s_events_event">
        <t t-set="record" t-value="data['_record']._set_tz_context()"/>
        <a class="s_events_event_cover position-relative d-flex flex-column shadow-sm overflow-hidden rounded text-decoration-none"
           t-att-href="data['call_to_action_url']">
            <t t-call="website.record_cover">
                <t t-set="_record" t-value="record"/>
                <t t-set="_resize_height" t-value="512"/>
                <t t-set="_resize_width" t-value="512"/>
                <t t-set="use_filters" t-value="True"/>
                <t t-set="additionnal_classes" t-value="'h-100 w-100 bg-600 position-absolute'"/>
            </t>

            <figcaption class="text-center w-100 h-100 px-3 d-flex flex-column flex-grow-1">
                <div t-if="is_sample" class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                <div class="s_events_event_title text-white" t-field="record.name"/>
                <time itemprop="startDate" t-att-datetime="record.date_begin" class="text-white">
                    <span t-out="record.date_begin"
                          t-options="{'widget': 'datetime', 'date_only': 'true', 'format': 'long', 'tz_name': record.date_tz}"/>
                    -
                    <span t-out="record.date_begin"
                          t-options="{'widget': 'datetime', 'time_only': 'true', 'format': 'short', 'tz_name': record.date_tz}"/>
                    (<span t-out="record.date_tz"/>)
                </time>
            </figcaption>
        </a>
    </figure>
</template>

<template id="dynamic_filter_template_event_event_card" name="Card Layout" priority="10">
    <div t-foreach="records" t-as="data" class="pb32 w-100 s_events_event">
        <t t-set="record" t-value="data['_record']._set_tz_context()"/>
        <div class="card shadow-sm">
            <a class="s_events_event_cover" t-att-href="data['call_to_action_url']">
                <t t-call="website.record_cover">
                    <t t-set="_record" t-value="record"/>
                    <t t-set="_resize_height" t-value="512"/>
                    <t t-set="_resize_width" t-value="512"/>

                    <div class="s_events_event_date position-absolute shadow-sm text-dark">
                        <span t-field="record.date_begin" t-options="{'format': 'LLL', 'tz_name': record.date_tz}"
                              class="s_events_event_month"/>
                        <span t-field="record.date_begin" t-options="{'format': 'dd', 'tz_name': record.date_tz}"
                              class="s_events_event_day"/>
                    </div>
                </t>
            </a>
            <div class="card-body p-3">
                <div t-if="is_sample" class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                <h5 class="mb-0 text-truncate" t-field="record.name"/>
                <time itemprop="startDate" t-att-datetime="record.date_begin">
                    <span t-field="record.date_begin"
                          t-options="{'date_only': 'true', 'format': 'long', 'tz_name': record.date_tz}"/>
                    -
                    <span t-field="record.date_begin"
                          t-options="{'time_only': 'true', 'format': 'short', 'tz_name': record.date_tz}"/>
                    (<span t-out="record.date_tz"/>)
                </time>
                <div itemprop="location" t-field="record.address_id"
                     t-options="{'widget': 'contact', 'fields': ['city'], 'no_marker': 'true'}"/>
            </div>
        </div>
    </div>
</template>

<!--SNIPPET OPTIONS -->

<template id="s_events_options" inherit_id="website.snippet_options">
    <xpath expr="." position="inside">
        <t t-call="website_event.s_dynamic_snippet_options_template">
            <t t-set="snippet_name" t-value="'event_upcoming_snippet'"/>
            <t t-set="snippet_selector" t-value="'.s_event_upcoming_snippet'"/>
        </t>
    </xpath>
</template>

<template id="s_dynamic_snippet_options_template" inherit_id="website.s_dynamic_snippet_options_template">
    <xpath expr="//we-select[@data-name='filter_opt']" position="after">
        <we-many2many t-if="snippet_name == 'event_upcoming_snippet'"
                      string="Event Tags"
                      data-name="event_tag_opt"
                      data-model="event.tag"
                      data-fakem2m="true"
                      data-domain='[["category_id.website_published", "=", true], ["color", "not in", ["0", false]]]'
                      data-limit="10"
                      data-attribute-name="filterByTagIds"
                      data-fields="[&quot;category_id&quot;]"
                      data-select-data-attribute=""
                      data-no-preview="true"/>
    </xpath>
</template>

<!--ASSETS-->

<record id="website_event.s_events_000_scss" model="ir.asset">
    <field name="name">Event 000 SCSS</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">website_event/static/src/snippets/s_events/000.scss</field>
</record>

<record id="website_event.s_events_000_js" model="ir.asset">
    <field name="name">Event 000 JS</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">website_event/static/src/snippets/s_events/000.js</field>
</record>

</odoo>

```

