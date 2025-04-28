# Odoo Module: website_event

Category: Website/Website

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

{
    'name': 'Events',
    'category': 'Website/Website',
    'sequence': 166,
    'summary': 'Publish events, sell tickets',
    'website': 'https://www.odoo.com/page/events',
    'description': "",
    'depends': ['website', 'website_partner', 'website_mail', 'event'],
    'data': [
        'data/event_data.xml',
        'views/res_config_settings_views.xml',
        'views/event_snippets.xml',
        'views/event_templates.xml',
        'views/event_views.xml',
        'security/ir.model.access.csv',
        'security/event_security.xml',
    ],
    'demo': [
        'data/event_demo.xml'
    ],
    'application': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-

import babel.dates
import re
import werkzeug
from werkzeug.datastructures import OrderedMultiDict

from datetime import datetime, timedelta
from dateutil.relativedelta import relativedelta

from odoo import fields, http, _
from odoo.addons.http_routing.models.ir_http import slug
from odoo.addons.website.controllers.main import QueryURL
from odoo.http import request
from odoo.tools.misc import get_lang


class WebsiteEventController(http.Controller):

    def sitemap_event(env, rule, qs):
        if not qs or qs.lower() in '/events':
            yield {'loc': '/events'}

    @http.route(['/event', '/event/page/<int:page>', '/events', '/events/page/<int:page>'], type='http', auth="public", website=True, sitemap=sitemap_event)
    def events(self, page=1, **searches):
        Event = request.env['event.event']
        EventType = request.env['event.type']

        searches.setdefault('search', '')
        searches.setdefault('date', 'all')
        searches.setdefault('type', 'all')
        searches.setdefault('country', 'all')

        website = request.website

        def sdn(date):
            return fields.Datetime.to_string(date.replace(hour=23, minute=59, second=59))

        def sd(date):
            return fields.Datetime.to_string(date)
        today = datetime.today()
        dates = [
            ['all', _('Next Events'), [("date_end", ">", sd(today))], 0],
            ['today', _('Today'), [
                ("date_end", ">", sd(today)),
                ("date_begin", "<", sdn(today))],
                0],
            ['week', _('This Week'), [
                ("date_end", ">=", sd(today + relativedelta(days=-today.weekday()))),
                ("date_begin", "<", sdn(today + relativedelta(days=6-today.weekday())))],
                0],
            ['nextweek', _('Next Week'), [
                ("date_end", ">=", sd(today + relativedelta(days=7-today.weekday()))),
                ("date_begin", "<", sdn(today + relativedelta(days=13-today.weekday())))],
                0],
            ['month', _('This month'), [
                ("date_end", ">=", sd(today.replace(day=1))),
                ("date_begin", "<", (today.replace(day=1) + relativedelta(months=1)).strftime('%Y-%m-%d 00:00:00'))],
                0],
            ['nextmonth', _('Next month'), [
                ("date_end", ">=", sd(today.replace(day=1) + relativedelta(months=1))),
                ("date_begin", "<", (today.replace(day=1) + relativedelta(months=2)).strftime('%Y-%m-%d 00:00:00'))],
                0],
            ['old', _('Past Events'), [
                ("date_end", "<", today.strftime('%Y-%m-%d 00:00:00'))],
                0],
        ]

        # search domains
        domain_search = {'website_specific': website.website_domain()}

        if searches['search']:
            domain_search['search'] = [('name', 'ilike', searches['search'])]

        current_date = None
        current_type = None
        current_country = None
        for date in dates:
            if searches["date"] == date[0]:
                domain_search["date"] = date[2]
                if date[0] != 'all':
                    current_date = date[1]

        if searches["type"] != 'all':
            current_type = EventType.browse(int(searches['type']))
            domain_search["type"] = [("event_type_id", "=", int(searches["type"]))]

        if searches["country"] != 'all' and searches["country"] != 'online':
            current_country = request.env['res.country'].browse(int(searches['country']))
            domain_search["country"] = ['|', ("country_id", "=", int(searches["country"])), ("country_id", "=", False)]
        elif searches["country"] == 'online':
            domain_search["country"] = [("country_id", "=", False)]

        def dom_without(without):
            domain = [('state', "in", ['draft', 'confirm', 'done'])]
            for key, search in domain_search.items():
                if key != without:
                    domain += search
            return domain

        # count by domains without self search
        for date in dates:
            if date[0] != 'old':
                date[3] = Event.search_count(dom_without('date') + date[2])

        domain = dom_without('type')
        types = Event.read_group(domain, ["id", "event_type_id"], groupby=["event_type_id"], orderby="event_type_id")
        types.insert(0, {
            'event_type_id_count': sum([int(type['event_type_id_count']) for type in types]),
            'event_type_id': ("all", _("All Categories"))
        })

        domain = dom_without('country')
        countries = Event.read_group(domain, ["id", "country_id"], groupby="country_id", orderby="country_id")
        countries.insert(0, {
            'country_id_count': sum([int(country['country_id_count']) for country in countries]),
            'country_id': ("all", _("All Countries"))
        })

        step = 12  # Number of events per page
        event_count = Event.search_count(dom_without("none"))
        pager = website.pager(
            url="/event",
            url_args=searches,
            total=event_count,
            page=page,
            step=step,
            scope=5)

        order = 'date_begin'
        if searches.get('date', 'all') == 'old':
            order = 'date_begin desc'
        if searches["country"] != 'all':   # if we are looking for a specific country
            order = 'is_online, ' + order  # show physical events first
        order = 'is_published desc, ' + order
        events = Event.search(dom_without("none"), limit=step, offset=pager['offset'], order=order)

        keep = QueryURL('/event', **{key: value for key, value in searches.items() if (key == 'search' or value != 'all')})

        values = {
            'current_date': current_date,
            'current_country': current_country,
            'current_type': current_type,
            'event_ids': events,  # event_ids used in website_event_track so we keep name as it is
            'dates': dates,
            'types': types,
            'countries': countries,
            'pager': pager,
            'searches': searches,
            'keep': keep,
        }

        if searches['date'] == 'old':
            # the only way to display this content is to set date=old so it must be canonical
            values['canonical_params'] = OrderedMultiDict([('date', 'old')])

        return request.render("website_event.index", values)

    @http.route(['''/event/<model("event.event", "[('website_id', 'in', (False, current_website_id))]"):event>/page/<path:page>'''], type='http', auth="public", website=True, sitemap=False)
    def event_page(self, event, page, **post):
        if not event.can_access_from_current_website():
            raise werkzeug.exceptions.NotFound()

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

    @http.route(['''/event/<model("event.event", "[('website_id', 'in', (False, current_website_id))]"):event>'''], type='http', auth="public", website=True)
    def event(self, event, **post):
        if not event.can_access_from_current_website():
            raise werkzeug.exceptions.NotFound()

        if event.menu_id and event.menu_id.child_id:
            target_url = event.menu_id.child_id[0].url
        else:
            target_url = '/event/%s/register' % str(event.id)
        if post.get('enable_editor') == '1':
            target_url += '?enable_editor=1'
        return request.redirect(target_url)

    @http.route(['''/event/<model("event.event", "[('website_id', 'in', (False, current_website_id))]"):event>/register'''], type='http', auth="public", website=True, sitemap=False)
    def event_register(self, event, **post):
        if not event.can_access_from_current_website():
            raise werkzeug.exceptions.NotFound()

        urls = event._get_event_resource_urls()
        values = {
            'event': event,
            'main_object': event,
            'range': range,
            'registrable': event.sudo()._is_event_registrable(),
            'google_url': urls.get('google_url'),
            'iCal_url': urls.get('iCal_url'),
        }
        return request.render("website_event.event_description_full", values)

    @http.route('/event/add_event', type='json', auth="user", methods=['POST'], website=True)
    def add_event(self, event_name="New Event", **kwargs):
        event = self._add_event(event_name, request.context)
        return "/event/%s/register?enable_editor=1" % slug(event)

    def _add_event(self, event_name=None, context=None, **kwargs):
        if not event_name:
            event_name = _("New Event")
        date_begin = datetime.today() + timedelta(days=(14))
        vals = {
            'name': event_name,
            'date_begin': fields.Date.to_string(date_begin),
            'date_end': fields.Date.to_string((date_begin + timedelta(days=(1)))),
            'seats_available': 1000,
            'website_id': request.website.id,
        }
        return request.env['event.event'].with_context(context or {}).create(vals)

    def get_formated_date(self, event):
        start_date = fields.Datetime.from_string(event.date_begin).date()
        end_date = fields.Datetime.from_string(event.date_end).date()
        month = babel.dates.get_month_names('abbreviated', locale=get_lang(event.env).code)[start_date.month]
        return ('%s %s%s') % (month, start_date.strftime("%e"), (end_date != start_date and ("-" + end_date.strftime("%e")) or ""))

    @http.route('/event/get_country_event_list', type='json', auth='public', website=True)
    def get_country_events(self, **post):
        Event = request.env['event.event']
        country_code = request.session['geoip'].get('country_code')
        result = {'events': [], 'country': False}
        events = None
        domain = request.website.website_domain()
        if country_code:
            country = request.env['res.country'].search([('code', '=', country_code)], limit=1)
            events = Event.search(domain + ['|', ('address_id', '=', None), ('country_id.code', '=', country_code), ('date_begin', '>=', '%s 00:00:00' % fields.Date.today()), ('state', '=', 'confirm')], order="date_begin")
        if not events:
            events = Event.search(domain + [('date_begin', '>=', '%s 00:00:00' % fields.Date.today()), ('state', '=', 'confirm')], order="date_begin")
        for event in events:
            if country_code and event.country_id.code == country_code:
                result['country'] = country
            result['events'].append({
                "date": self.get_formated_date(event),
                "event": event,
                "url": event.website_url})
        return request.env['ir.ui.view'].render_template("website_event.country_events_list", result)

    def _process_tickets_details(self, data):
        nb_register = int(data.get('nb_register-0', 0))
        if nb_register:
            return [{'id': 0, 'name': 'Registration', 'quantity': nb_register, 'price': 0}]
        return []

    @http.route(['/event/<model("event.event"):event>/registration/new'], type='json', auth="public", methods=['POST'], website=True)
    def registration_new(self, event, **post):
        tickets = self._process_tickets_details(post)
        availability_check = True
        if event.seats_availability == 'limited':
            ordered_seats = 0
            for ticket in tickets:
                ordered_seats += ticket['quantity']
            if event.seats_available < ordered_seats:
                availability_check = False
        if not tickets:
            return False
        return request.env['ir.ui.view'].render_template("website_event.registration_attendee_details", {'tickets': tickets, 'event': event, 'availability_check': availability_check})

    def _process_registration_details(self, details):
        ''' Process data posted from the attendee details form. '''
        registrations = {}
        global_values = {}
        for key, value in details.items():
            counter, field_name = key.split('-', 1)
            if counter == '0':
                global_values[field_name] = value
            else:
                registrations.setdefault(counter, dict())[field_name] = value
        for key, value in global_values.items():
            for registration in registrations.values():
                registration[key] = value
        return list(registrations.values())

    @http.route(['''/event/<model("event.event", "[('website_id', 'in', (False, current_website_id))]"):event>/registration/confirm'''], type='http', auth="public", methods=['POST'], website=True)
    def registration_confirm(self, event, **post):
        if not event.can_access_from_current_website():
            raise werkzeug.exceptions.NotFound()

        Attendees = request.env['event.registration']
        registrations = self._process_registration_details(post)

        for registration in registrations:
            registration['event_id'] = event
            Attendees += Attendees.sudo().create(
                Attendees._prepare_attendee_values(registration))

        urls = event._get_event_resource_urls()
        return request.render("website_event.registration_complete", {
            'attendees': Attendees.sudo(),
            'event': event,
            'google_url': urls.get('google_url'),
            'iCal_url': urls.get('iCal_url')
        })

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main

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
        <field name="is_published">True</field>
        <field name="twitter_hashtag">DesignFair</field>
        <field name="subtitle">Get Inspired • Stay Connected • Have Fun</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_0.jpg')", "background-color": "oe_black", "resize_class": "o_record_has_cover cover_mid", "opacity": "0.4"}</field>
        <field name="description"><![CDATA[
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
                    <p class="mb-0">If you wish to make a presentation, please send your topic proposal as soon as possible for approval to Mr. Famke Jenssens at ngh (a) yourcompany (dot) com. The presentations should be, for example, a presentation of a community module, a case study, methodology feedback, technical, etc. Each presentation must be in English.</em></p>
                </div>
                <p class="mb-3">For any additional information, please contact us at <a href="mailto:events@yourcompany.com">events@yourcompany.com</a>.</p>
                <div class="bg-light rounded-right border-left border-secondary p-3 mb-5" style="border-left-width: 3px !important;">
                    <p class="mb-0">OpenElec Applications reserves the right to cancel, re-name or re-locate the event or change the dates on which it is held.</p>
                </div>
            </div>
        ]]></field>
    </record>

    <record id="event.event_1" model="event.event">
        <field name="website_menu">True</field>
        <field name="website_published">True</field>
        <field name="subtitle">The Great Reno Balloon Race is the world's largest free hot-air ballooning event.</field>
        <field name="twitter_hashtag">RenoRace</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_1.jpg')", "background-color": "oe_black", "resize_class": "o_record_has_cover cover_mid", "opacity": "0.4"}</field>
    </record>

    <record id="event.event_2" model="event.event">
        <field name="website_published">True</field>
        <field name="subtitle">Enhance your architectural business and improve professional skills.</field>
        <field name="twitter_hashtag">odoo</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_2.jpg')", "background-color": "oe_black", "resize_class": "o_record_has_cover cover_mid", "opacity": "0.4"}</field>
        <field name="description"><![CDATA[
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
        ]]></field>
    </record>

    <record id="event.event_3" model="event.event">
        <field name="website_published">True</field>
        <field name="subtitle">Experience live music, local food and beverages.</field>
        <field name="twitter_hashtag">odoo</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_3.jpg')", "background-color": "oe_black", "resize_class": "o_record_has_cover cover_mid", "opacity": "0.4"}</field>
    </record>

    <record id="event.event_4" model="event.event">
        <field name="website_published">True</field>
        <field name="subtitle">Discover how to grow a sustainable business with our experts.</field>
        <field name="twitter_hashtag">odoo</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_4.jpg')", "background-color": "oe_black", "resize_class": "o_record_has_cover cover_mid", "opacity": "0.4"}</field>
    </record>

    <record id="event.event_5" model="event.event">
        <field name="website_published">True</field>
        <field name="subtitle">Bring your outdoor field hockey season to the next level by taking the field at this 9th annual Field Hockey tournament.</field>
        <field name="twitter_hashtag">odoo</field>
        <field name="cover_properties">{"background-image": "url('/website_event/static/src/img/event_cover_5.jpg')", "background-color": "oe_black", "resize_class": "o_record_has_cover cover_mid", "opacity": "0.4"}</field>
    </record>

    <record id="event.event_6" model="event.event">
        <field name="website_published">False</field>
        <field name="twitter_hashtag">odoo</field>
        <field name="cover_properties">{"background-image": "none", "background-color": "secondary", "opacity": ""}</field>
    </record>

    <record id="base.res_partner_1" model="res.partner">
        <field name="is_published">True</field>
    </record>

    <record id="base.res_partner_2" model="res.partner">
        <field name="is_published">True</field>
    </record>

    <record id="base.res_partner_3" model="res.partner">
        <field name="is_published">True</field>
    </record>

    <record id="base.res_partner_4" model="res.partner">
        <field name="is_published">True</field>
    </record>

    <record id="event.event_2" model="event.event">
        <field name="is_published">True</field>
        <field name="twitter_hashtag">odoo</field>
    </record>

    <record id="base.res_partner_address_4" model="res.partner">
        <field name="is_published">True</field>
    </record>
</odoo>

```

## File: models\event.py

```python
# -*- coding: utf-8 -*-

import pytz
import werkzeug
import json

from odoo import api, fields, models, _
from odoo.addons.http_routing.models.ir_http import slug
from odoo.exceptions import UserError

GOOGLE_CALENDAR_URL = 'https://www.google.com/calendar/render?'


class EventType(models.Model):
    _name = 'event.type'
    _inherit = ['event.type']

    website_menu = fields.Boolean(
        'Display a dedicated menu on Website')


class Event(models.Model):
    _name = 'event.event'
    _inherit = ['event.event', 'website.seo.metadata', 'website.published.multi.mixin']

    website_published = fields.Boolean(tracking=True)

    subtitle = fields.Char('Event Subtitle', translate=True)

    is_participating = fields.Boolean("Is Participating", compute="_compute_is_participating")

    cover_properties = fields.Text(
        'Cover Properties',
        default='{"background-image": "none", "background-color": "oe_blue", "opacity": "0.4", "resize_class": "cover_mid"}')

    website_menu = fields.Boolean('Dedicated Menu',
        help="Creates menus Introduction, Location and Register on the page "
             " of the event on the website.", copy=False)
    menu_id = fields.Many2one('website.menu', 'Event Menu', copy=False)

    def _compute_is_participating(self):
        # we don't allow public user to see participating label
        if self.env.user != self.env['website'].get_current_website().user_id:
            email = self.env.user.partner_id.email
            for event in self:
                domain = ['&','&', '|', ('email', '=', email), ('partner_id', '=', self.env.user.partner_id.id),
                          ('event_id', '=', event.id), ('state', '!=', 'cancel')]
                event.is_participating = self.env['event.registration'].sudo().search_count(domain)
        else:
            self.is_participating = False

    @api.depends('name')
    def _compute_website_url(self):
        super(Event, self)._compute_website_url()
        for event in self:
            if event.id:  # avoid to perform a slug on a not yet saved record in case of an onchange.
                event.website_url = '/event/%s' % slug(event)

    @api.onchange('event_type_id')
    def _onchange_type(self):
        super(Event, self)._onchange_type()
        if self.event_type_id:
            self.website_menu = self.event_type_id.website_menu

    def _get_menu_entries(self):
        """ Method returning menu entries to display on the website view of the
        event, possibly depending on some options in inheriting modules. """
        self.ensure_one()
        return [
            (_('Introduction'), False, 'website_event.template_intro'),
            (_('Location'), False, 'website_event.template_location'),
            (_('Register'), '/event/%s/register' % slug(self), False),
        ]

    def _toggle_create_website_menus(self, vals):
        for event in self:
            if 'website_menu' in vals:
                if event.menu_id and not event.website_menu:
                    event.menu_id.unlink()
                elif event.website_menu:
                    if not event.menu_id:
                        root_menu = self.env['website.menu'].create({'name': event.name, 'website_id': event.website_id.id})
                        event.menu_id = root_menu
                    for sequence, (name, url, xml_id) in enumerate(event._get_menu_entries()):
                        event._create_menu(sequence, name, url, xml_id)

    @api.model
    def create(self, vals):
        res = super(Event, self).create(vals)
        res._toggle_create_website_menus(vals)
        return res

    def write(self, vals):
        res = super(Event, self).write(vals)
        self._toggle_create_website_menus(vals)
        return res

    def _create_menu(self, sequence, name, url, xml_id):
        if not url:
            self.env['ir.ui.view'].with_context(_force_unlink=True).search([('name', '=', name + ' ' + self.name)]).unlink()
            newpath = self.env['website'].new_page(name + ' ' + self.name, template=xml_id, ispage=False)['url']
            url = "/event/" + slug(self) + "/page/" + newpath[1:]
        menu = self.env['website.menu'].create({
            'name': name,
            'url': url,
            'parent_id': self.menu_id.id,
            'sequence': sequence,
            'website_id': self.website_id.id,
        })
        return menu

    def google_map_img(self, zoom=8, width=298, height=298):
        self.ensure_one()
        if self.address_id:
            return self.sudo().address_id.google_map_img(zoom=zoom, width=width, height=height)
        return None

    def google_map_link(self, zoom=8):
        self.ensure_one()
        if self.address_id:
            return self.sudo().address_id.google_map_link(zoom=zoom)
        return None

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'is_published' in init_values and self.is_published:
            return self.env.ref('website_event.mt_event_published')
        elif 'is_published' in init_values and not self.is_published:
            return self.env.ref('website_event.mt_event_unpublished')
        return super(Event, self)._track_subtype(init_values)

    def action_open_badge_editor(self):
        """ open the event badge editor : redirect to the report page of event badge report """
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'target': 'new',
            'url': '/report/html/%s/%s?enable_editor' % ('event.event_event_report_template_badge', self.id),
        }

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
        encoded_params = werkzeug.url_encode(params)
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

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import event

```

## File: security\event_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_event_public" model="ir.rule">
        <field name="name">event: Public</field>
        <field name="model_id" ref="event.model_event_event"/>
        <field name="domain_force">[('website_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

    <record id="event.group_event_manager" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('website.group_website_publisher'))]"/>
    </record>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_event_event_public,event.event.public,event.model_event_event,base.group_public,1,0,0,0
access_event_event_portal,event.event.portal,event.model_event_event,base.group_portal,1,0,0,0
access_event_type_public,event.type.public,event.model_event_type,,1,0,0,0
```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#269396"/><stop offset="100%" stop-color="#218689"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M43.126 69H4c-2 0-4-.146-4-4.078v-23.83l15-16.955 8-4.078 8 2.039L39 17l26 23.451L43.126 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M32.51 22.527l-6.99 7.529.047-.334c.058-.081.058-.189 0-.323-.1-.215-.238-.3-.411-.258.043-.086.072-.211.086-.376.015-.165.022-.262.022-.29.043-.187.13-.352.26-.495a4.13 4.13 0 0 0 .357-.474c.05-.086.054-.129.01-.129.506.058.867-.021 1.084-.236.072-.072.155-.194.248-.366.094-.172.17-.294.228-.366.13-.086.23-.125.303-.118a.974.974 0 0 1 .314.118c.137.072.242.108.314.108.202.014.314-.065.335-.237a.465.465 0 0 0-.162-.43c.173.014.195-.108.065-.366a.823.823 0 0 0-.173-.193c-.173-.058-.368-.022-.585.107-.115.058-.1.115.044.172-.015-.014-.083.061-.206.226-.123.165-.242.29-.357.377-.116.086-.231.05-.347-.108a1.73 1.73 0 0 1-.119-.29c-.065-.18-.133-.276-.206-.29-.115 0-.23.107-.346.322.043-.115-.036-.222-.238-.323a1.564 1.564 0 0 0-.52-.172c.274-.172.217-.366-.173-.58-.101-.058-.249-.094-.444-.108-.195-.015-.335.014-.422.086a.577.577 0 0 0-.12.247c-.006.065.03.122.11.172.078.05.154.09.226.119.073.028.156.057.25.086.093.028.155.05.183.064.203.144.26.244.174.301-.03.015-.09.04-.184.076-.094.035-.177.068-.25.096-.072.03-.115.058-.13.086-.043.058-.043.158 0 .302.044.143.03.243-.043.3-.072-.07-.137-.196-.194-.376a1.367 1.367 0 0 0-.152-.355c.101.13-.08.172-.541.13l-.217-.022c-.058 0-.173.014-.346.043a1.619 1.619 0 0 1-.444.021.403.403 0 0 1-.292-.172c-.058-.114-.058-.258 0-.43.014-.057.043-.072.086-.043a3.238 3.238 0 0 1-.238-.204 2.11 2.11 0 0 0-.216-.183 12.32 12.32 0 0 0-2.036.882.536.536 0 0 0 .26-.022c.072-.028.166-.075.282-.14.115-.064.187-.103.216-.118.49-.2.794-.25.91-.15l.108-.108c.202.23.346.409.433.538-.101-.057-.318-.065-.65-.022-.288.086-.447.172-.476.259.101.172.137.3.108.387a3.375 3.375 0 0 1-.249-.215c-.108-.1-.213-.18-.314-.237a.868.868 0 0 0-.324-.108c-.231 0-.39.008-.477.022a13.627 13.627 0 0 0-5.088 4.776c.101.1.188.157.26.172.058.014.094.079.108.193a.82.82 0 0 0 .054.237c.022.043.105.022.25-.064.13.114.15.25.064.408.015-.014.332.18.953.581.274.244.426.394.455.452.043.158-.03.287-.217.387a1.103 1.103 0 0 0-.195-.194c-.115-.1-.18-.129-.195-.086-.043.072-.04.205.011.398.05.194.127.284.228.27-.101 0-.17.114-.206.344-.036.229-.054.484-.054.763 0 .28-.007.448-.022.506l.043.021c-.043.172-.003.42.12.742.122.323.277.463.465.42-.188.043-.043.351.433.925.087.115.144.18.173.194.044.028.13.082.26.16.13.08.238.151.325.216a.92.92 0 0 1 .216.226c.058.072.13.233.217.484.086.25.187.42.303.505-.029.087.04.23.206.43.166.202.241.366.227.496a.106.106 0 0 0-.054.021.106.106 0 0 1-.054.022c.043.1.155.2.335.3.18.101.293.194.336.28.014.043.029.115.043.216.015.1.036.179.065.236.029.058.087.072.173.043.03-.287-.144-.731-.52-1.334-.216-.358-.338-.566-.367-.623a1.231 1.231 0 0 1-.12-.334 1.633 1.633 0 0 0-.097-.312c.029 0 .072.01.13.032.058.022.12.047.184.076.065.028.12.057.162.086.044.028.058.05.044.064-.044.1-.03.226.043.377.072.15.159.283.26.398a29.568 29.568 0 0 0 .628.688c.086.086.187.226.303.42.115.193.115.29 0 .29.13 0 .274.072.433.215.159.144.281.287.368.43.072.115.13.302.173.56.043.258.08.43.108.516.03.1.09.197.184.29.094.094.185.162.271.205l.347.172.281.15c.072.03.206.105.4.226a2.8 2.8 0 0 0 .466.248c.144.057.26.086.346.086.087 0 .192-.018.314-.054.123-.036.22-.06.293-.075.216-.029.425.079.628.322.202.244.353.395.454.452.52.273.917.352 1.191.237-.029.014-.025.068.01.161.037.093.095.205.174.334a21.069 21.069 0 0 0 .314.494c.072.086.202.194.39.323.187.13.317.237.39.323.086-.058.137-.122.151-.194-.043.115.007.258.152.43.144.173.274.244.39.216.201-.043.302-.273.302-.689-.447.215-.8.086-1.06-.387a.418.418 0 0 0-.055-.118 1.367 1.367 0 0 1-.14-.366.303.303 0 0 1 0-.161c.014-.043.05-.065.108-.065.13 0 .202-.025.216-.075.015-.05 0-.14-.043-.27a4.316 4.316 0 0 1-.086-.279c-.015-.115-.094-.258-.239-.43a5.739 5.739 0 0 1-.26-.323c-.072.13-.187.187-.346.172-.159-.014-.274-.079-.346-.193a.59.59 0 0 1-.033.118.509.509 0 0 0-.032.14c-.188 0-.296-.007-.325-.022.014-.043.032-.168.054-.376s.047-.37.076-.484c.014-.057.054-.144.119-.258l.069-.125 3.432 3.26a.426.426 0 0 0-.07.07 1.065 1.065 0 0 0-.13.258c-.043.115-.079.194-.107.237-.03-.058-.112-.104-.25-.14-.137-.036-.205-.075-.205-.118.029.143.058.394.086.753.03.358.065.63.109.817.1.445.014.789-.26 1.033-.39.358-.6.645-.628.86-.058.316.029.502.26.56 0 .1-.058.247-.173.44-.116.194-.166.348-.152.463 0 .086.014.2.043.344 1.916-.332 3.649-1.013 5.199-2.042l2.057 1.954c-.42.297-.858.578-1.312.841-2.548 1.477-5.33 2.216-8.347 2.216-3.017 0-5.799-.739-8.346-2.216a16.5 16.5 0 0 1-6.052-6.013c-1.487-2.53-2.23-5.295-2.23-8.293 0-2.997.743-5.761 2.23-8.293a16.5 16.5 0 0 1 6.052-6.012c2.547-1.478 5.33-2.216 8.346-2.216 2.071 0 4.032.348 5.882 1.044zm-8.39 17.386l2.321 2.205a.67.67 0 0 0-.224.318 4.44 4.44 0 0 0-.065.226.726.726 0 0 1-.108.247.538.538 0 0 1-.195.15c-.101.044-.275.058-.52.044-.245-.014-.419-.05-.52-.108-.187-.114-.35-.322-.487-.624-.137-.3-.205-.566-.205-.796 0-.143.018-.333.054-.57.036-.236.057-.415.065-.537.004-.082-.035-.257-.12-.527l.004-.028zm17.614-12.441l14.71 13.245-7.995 8.879-14.71-13.245 7.995-8.88zm14.245 20.962a2.978 2.978 0 0 0 .208 4.206l-3.997 4.44a2.978 2.978 0 0 1-4.205.233L25.92 37.445a2.978 2.978 0 0 1-.208-4.206l3.997-4.44a2.978 2.978 0 0 0 4.205-.233 2.978 2.978 0 0 0-.207-4.206l3.997-4.44a2.978 2.978 0 0 1 4.205-.233l22.065 19.868a2.978 2.978 0 0 1 .208 4.206l-3.997 4.44a2.978 2.978 0 0 0-4.206.233zm2.233-6.774a1.489 1.489 0 0 0-.104-2.103L42.662 25.65a1.489 1.489 0 0 0-2.102.116l-8.661 9.62a1.489 1.489 0 0 0 .104 2.102l15.445 13.908c.61.548 1.55.496 2.103-.117l8.66-9.619z" opacity=".3"/><path fill="#FFF" d="M32.51 20.527l-6.99 7.529.047-.334c.058-.081.058-.189 0-.323-.1-.215-.238-.3-.411-.258.043-.086.072-.211.086-.376.015-.165.022-.262.022-.29.043-.187.13-.352.26-.495a4.13 4.13 0 0 0 .357-.474c.05-.086.054-.129.01-.129.506.058.867-.021 1.084-.236.072-.072.155-.194.248-.366.094-.172.17-.294.228-.366.13-.086.23-.125.303-.118a.974.974 0 0 1 .314.118c.137.072.242.108.314.108.202.014.314-.065.335-.237a.465.465 0 0 0-.162-.43c.173.014.195-.108.065-.366a.823.823 0 0 0-.173-.193c-.173-.058-.368-.022-.585.107-.115.058-.1.115.044.172-.015-.014-.083.061-.206.226-.123.165-.242.29-.357.377-.116.086-.231.05-.347-.108a1.73 1.73 0 0 1-.119-.29c-.065-.18-.133-.276-.206-.29-.115 0-.23.107-.346.322.043-.115-.036-.222-.238-.323a1.564 1.564 0 0 0-.52-.172c.274-.172.217-.366-.173-.58-.101-.058-.249-.094-.444-.108-.195-.015-.335.014-.422.086a.577.577 0 0 0-.12.247c-.006.065.03.122.11.172.078.05.154.09.226.119.073.028.156.057.25.086.093.028.155.05.183.064.203.144.26.244.174.301-.03.015-.09.04-.184.076-.094.035-.177.068-.25.096-.072.03-.115.058-.13.086-.043.058-.043.158 0 .302.044.143.03.243-.043.3-.072-.07-.137-.196-.194-.376a1.367 1.367 0 0 0-.152-.355c.101.13-.08.172-.541.13l-.217-.022c-.058 0-.173.014-.346.043a1.619 1.619 0 0 1-.444.021.403.403 0 0 1-.292-.172c-.058-.114-.058-.258 0-.43.014-.057.043-.072.086-.043a3.238 3.238 0 0 1-.238-.204 2.11 2.11 0 0 0-.216-.183 12.32 12.32 0 0 0-2.036.882.536.536 0 0 0 .26-.022c.072-.028.166-.075.282-.14.115-.064.187-.103.216-.118.49-.2.794-.25.91-.15l.108-.108c.202.23.346.409.433.538-.101-.057-.318-.065-.65-.022-.288.086-.447.172-.476.259.101.172.137.3.108.387a3.375 3.375 0 0 1-.249-.215c-.108-.1-.213-.18-.314-.237a.868.868 0 0 0-.324-.108c-.231 0-.39.008-.477.022a13.627 13.627 0 0 0-5.088 4.776c.101.1.188.157.26.172.058.014.094.079.108.193a.82.82 0 0 0 .054.237c.022.043.105.022.25-.064.13.114.15.25.064.408.015-.014.332.18.953.581.274.244.426.394.455.452.043.158-.03.287-.217.387a1.103 1.103 0 0 0-.195-.194c-.115-.1-.18-.129-.195-.086-.043.072-.04.205.011.398.05.194.127.284.228.27-.101 0-.17.114-.206.344-.036.229-.054.484-.054.763 0 .28-.007.448-.022.506l.043.021c-.043.172-.003.42.12.742.122.323.277.463.465.42-.188.043-.043.351.433.925.087.115.144.18.173.194.044.028.13.082.26.16.13.08.238.151.325.216a.92.92 0 0 1 .216.226c.058.072.13.233.217.484.086.25.187.42.303.505-.029.087.04.23.206.43.166.202.241.366.227.496a.106.106 0 0 0-.054.021.106.106 0 0 1-.054.022c.043.1.155.2.335.3.18.101.293.194.336.28.014.043.029.115.043.216.015.1.036.179.065.236.029.058.087.072.173.043.03-.287-.144-.731-.52-1.334-.216-.358-.338-.566-.367-.623a1.231 1.231 0 0 1-.12-.334 1.633 1.633 0 0 0-.097-.312c.029 0 .072.01.13.032.058.022.12.047.184.076.065.028.12.057.162.086.044.028.058.05.044.064-.044.1-.03.226.043.377.072.15.159.283.26.398a29.568 29.568 0 0 0 .628.688c.086.086.187.226.303.42.115.193.115.29 0 .29.13 0 .274.072.433.215.159.144.281.287.368.43.072.115.13.302.173.56.043.258.08.43.108.516.03.1.09.197.184.29.094.094.185.162.271.205l.347.172.281.15c.072.03.206.105.4.226a2.8 2.8 0 0 0 .466.248c.144.057.26.086.346.086.087 0 .192-.018.314-.054.123-.036.22-.06.293-.075.216-.029.425.079.628.322.202.244.353.395.454.452.52.273.917.352 1.191.237-.029.014-.025.068.01.161.037.093.095.205.174.334a21.069 21.069 0 0 0 .314.494c.072.086.202.194.39.323.187.13.317.237.39.323.086-.058.137-.122.151-.194-.043.115.007.258.152.43.144.173.274.244.39.216.201-.043.302-.273.302-.689-.447.215-.8.086-1.06-.387a.418.418 0 0 0-.055-.118 1.367 1.367 0 0 1-.14-.366.303.303 0 0 1 0-.161c.014-.043.05-.065.108-.065.13 0 .202-.025.216-.075.015-.05 0-.14-.043-.27a4.316 4.316 0 0 1-.086-.279c-.015-.115-.094-.258-.239-.43a5.739 5.739 0 0 1-.26-.323c-.072.13-.187.187-.346.172-.159-.014-.274-.079-.346-.193a.59.59 0 0 1-.033.118.509.509 0 0 0-.032.14c-.188 0-.296-.007-.325-.022.014-.043.032-.168.054-.376s.047-.37.076-.484c.014-.057.054-.144.119-.258l.069-.125 3.432 3.26a.426.426 0 0 0-.07.07 1.065 1.065 0 0 0-.13.258c-.043.115-.079.194-.107.237-.03-.058-.112-.104-.25-.14-.137-.036-.205-.075-.205-.118.029.143.058.394.086.753.03.358.065.63.109.817.1.445.014.789-.26 1.033-.39.358-.6.645-.628.86-.058.316.029.502.26.56 0 .1-.058.247-.173.44-.116.194-.166.348-.152.463 0 .086.014.2.043.344 1.916-.332 3.649-1.013 5.199-2.042l2.057 1.954c-.42.297-.858.578-1.312.841-2.548 1.477-5.33 2.216-8.347 2.216-3.017 0-5.799-.739-8.346-2.216a16.5 16.5 0 0 1-6.052-6.013c-1.487-2.53-2.23-5.295-2.23-8.293 0-2.997.743-5.761 2.23-8.293a16.5 16.5 0 0 1 6.052-6.012c2.547-1.478 5.33-2.216 8.346-2.216 2.071 0 4.032.348 5.882 1.044zm-8.39 17.386l2.321 2.205a.67.67 0 0 0-.224.318 4.44 4.44 0 0 0-.065.226.726.726 0 0 1-.108.247.538.538 0 0 1-.195.15c-.101.044-.275.058-.52.044-.245-.014-.419-.05-.52-.108-.187-.114-.35-.322-.487-.624-.137-.3-.205-.566-.205-.796 0-.143.018-.333.054-.57.036-.236.057-.415.065-.537.004-.082-.035-.257-.12-.527l.004-.028zm17.614-12.441l14.71 13.245-7.995 8.879-14.71-13.245 7.995-8.88zm14.245 20.962a2.978 2.978 0 0 0 .208 4.206l-3.997 4.44a2.978 2.978 0 0 1-4.205.233L25.92 35.445a2.978 2.978 0 0 1-.208-4.206l3.997-4.44a2.978 2.978 0 0 0 4.205-.233 2.978 2.978 0 0 0-.207-4.206l3.997-4.44a2.978 2.978 0 0 1 4.205-.233l22.065 19.868a2.978 2.978 0 0 1 .208 4.206l-3.997 4.44a2.978 2.978 0 0 0-4.206.233zm2.233-6.774a1.489 1.489 0 0 0-.104-2.103L42.662 23.65a1.489 1.489 0 0 0-2.102.116l-8.661 9.62a1.489 1.489 0 0 0 .104 2.102l15.445 13.908c.61.548 1.55.496 2.103-.117l8.66-9.619z"/></g></g></svg>
```

## File: static\src\js\website_event.editor.js

```javascript
odoo.define('website_event.editor', function (require) {
"use strict";

var core = require('web.core');
var wUtils = require('website.utils');
var WebsiteNewMenu = require('website.newMenu');

var _t = core._t;

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
        return wUtils.prompt({
            id: "editor_new_event",
            window_title: _t("New Event"),
            input: _t("Event Name"),
        }).then(function (result) {
            var eventName = result.val;
            if (!eventName) {
                return;
            }
            return self._rpc({
                route: '/event/add_event',
                params: {
                    event_name: eventName,
                },
            }).then(function (url) {
                window.location.href = url;
                return new Promise(function () {});
            });
        });
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
    events: {
        'click .o_wevent_registration_btn': '_onRegistrationBtnClick',
    },

    /**
     * @override
     */
    start: function () {
        var self = this;
        var res = this._super.apply(this.arguments).then(function () {
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
            return ajax.jsonRpc($form.attr('action'), 'call', post).then(function (modal) {
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
    /**
     * @private
     * @param {Event} ev
     */
    _onRegistrationBtnClick: function (ev) {
        var $btn = $(ev.currentTarget);
        $btn.toggleClass('btn-primary text-left pl-0');
        $btn.siblings().toggleClass('d-none');
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
                self._$loadedContent.addClass('o_temp_auto_element');
                self._$loadedContent.attr('data-temp-auto-element-original-content', self._originalContent);

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

## File: static\src\js\tours\website_event.js

```javascript
odoo.define("website_event.tour", function (require) {
    "use strict";

    var core = require("web.core");
    var tour = require("web_tour.tour");

    var _t = core._t;

    tour.register("event", {
        url: "/",
    }, [tour.STEPS.WEBSITE_NEW_PAGE, {
        trigger: "a[data-action=new_event]",
        content: _t("Click here to create a new event."),
        position: "bottom",
    }, {
        trigger: '.modal-dialog #editor_new_event input[type=text]',
        content: _t("Create a name for your new event and click <em>\"Continue\"</em>. e.g: Technical Training"),
        position: "right",
    }, {
        trigger: '.modal-footer button.btn-primary.btn-continue',
        extra_trigger: '#editor_new_event input[type=text][value!=""]',
        content: _t("Click <em>Continue</em> to create the event."),
        position: "right",
    }, {
        trigger: "#snippet_structure .oe_snippet:eq(2) .oe_snippet_thumbnail",
        content: _t("Drag this block and drop it in your page."),
        position: "bottom",
        run: "drag_and_drop",
    }, {
        trigger: "button[data-action=save]",
        content: _t("Once you click on save, your event is updated."),
        position: "bottom",
        extra_trigger: ".o_dirty",
    }, {
        trigger: ".js_publish_management .js_publish_btn",
        extra_trigger: "body:not(.editor_enable)",
        content: _t("Click to publish your event."),
        position: "top",
    }, {
        trigger: ".css_edit_dynamic",
        extra_trigger: ".js_publish_management .js_publish_btn .css_unpublish:visible",
        content: _t("Click here to customize your event further."),
        position: "bottom",
    }]);
});

```

## File: views\event_snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- Snippet - Country Events - Placeholder -->
<template id="s_country_events" name="Country Events">
    <div t-attf-class="s_country_events_list oe_country_events #{_classes}">
        <div class="country_events_list">
            <h6 class="o_wevent_sidebar_title">
                <i class="fa fa-globe mr-2"/>Upcoming Events
            </h6>
        </div>
    </div>
</template>

<!-- Snippet - Country Events - List -->
<template id="country_events_list" name="Country Events List">
    <div class="country_events_list">
        <t t-if="events">
            <h6 class="o_wevent_sidebar_title">
                <t t-if="country">
                    <i class="fa fa-flag mr-2"/>Events: <span t-esc="country.name"/>
                    <img class="img-fluid" t-att-src="website.image_url(country, 'image')" alt=""/>
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

<!-- Replace snippet demo with actual snippet and options -->
<template id="remove_external_snippets" inherit_id="website.external_snippets">
    <xpath expr="//t[@t-install='website_event']" position="replace"/>
</template>

<template id="snippets" inherit_id="website.snippets">
    <xpath expr="//div[@id='snippet_content']//t[@t-snippet][last()]" position="after">
        <t t-snippet="website_event.s_country_events" t-thumbnail="/website_event/static/src/img/world_map.jpg"/>
        <t t-snippet="website_event.s_speaker_bio" t-thumbnail="/website_event/static/src/img/s_country_events.png"/>
    </xpath>
</template>

<template id="snippet_options" inherit_id="website.snippet_options">
    <xpath expr="//div[@id='so_content_addition']" position="attributes">
        <attribute name="data-selector" add=".oe_country_events, .s_speaker_bio" separator=","/>
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

## File: views\event_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="assets_frontend" inherit_id="website.assets_frontend" name="Event Assets Frontend">
    <xpath expr="//link[last()]" position="after">
        <link rel="stylesheet" type="text/scss" href="/website_event/static/src/scss/website_event.scss"/>
    </xpath>
    <xpath expr="//script[last()]" position="after">
        <script type="text/javascript" src="/website_event/static/src/js/website_geolocation.js"></script>
        <script type="text/javascript" src="/website_event/static/src/js/website_event.js"></script>
    </xpath>
</template>

<template id="assets_editor" inherit_id="website.assets_editor" name="Event Assets Editor">
    <xpath expr="." position="inside">
        <script type="text/javascript" src="/website_event/static/src/js/website_event.editor.js"></script>
        <script type="text/javascript" src="/website_event/static/src/js/tours/website_event.js"></script>
    </xpath>
</template>

<!-- Index -->
<template id="index" name="Events">
    <t t-call="website.layout">
        <div id="wrap" class="o_wevent_index">
            <!-- Options -->
            <t t-set="opt_events_list_cards" t-value="request.website.viewref('website_event.opt_events_list_cards').active"/>
            <t t-set="opt_events_list_columns" t-value="request.website.viewref('website_event.opt_events_list_columns').active"/>
            <!-- Topbar -->
            <t t-call="website_event.index_topbar"/>
            <!-- Drag/Drop Area -->
            <div id="oe_structure_we_index_1" class="oe_structure"/>
            <!-- Content -->
            <div t-attf-class="o_wevent_events_list #{opt_events_list_cards and 'opt_event_list_cards_bg'}">
                <div class="container">
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
            <div id="oe_structure_we_index_2" class="oe_structure"/>
        </div>
    </t>
</template>

<!-- Index - OPTION - Sidebar -->
<template id="opt_index_sidebar" inherit_id="website_event.index" active="True" customize_show="True" name="Show Sidebar">
    <xpath expr="//div[@id='o_wevent_index_main_col']" position="after">
        <t t-call="website_event.index_sidebar"/>
    </xpath>
</template>

<!-- Index Topbar -->
<template id="index_topbar" name="Topbar">
    <nav class="navbar navbar-light border-top shadow-sm d-print-none">
        <div class="container">
            <span class="navbar-brand h4 my-0 mr-4">Events</span>
            <ul class="o_wevent_index_topbar_filters nav mr-n2"/>
            <t t-call="website_event.events_search_box">
                <t t-set="_classes" t-value="'ml-auto pl-lg-3'"/>
                <t t-set="_searches" t-value="searches"/>
            </t>
        </div>
    </nav>
</template>

<!-- Filter - Date -->
<template id="event_time" inherit_id="website_event.index_topbar" customize_show="True" name="Filter by Date">
    <xpath expr="//ul[hasclass('o_wevent_index_topbar_filters')]" position="inside">
        <li class="nav-item dropdown mr-2 my-1 my-lg-0">
            <a href="#" role="button" class="btn dropdown-toggle" data-toggle="dropdown">
                <i class="fa fa-calendar"/>
                <t t-if="current_date" t-esc="current_date"/>
                <t t-else="">Next Events</t>
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

<!-- Filter - Category -->
<template id="event_category" inherit_id="website_event.index_topbar" active="True" customize_show="True" name="Filter by Category">
    <xpath expr="//ul[hasclass('o_wevent_index_topbar_filters')]" position="inside">
        <li class="nav-item dropdown mr-2 my-1 my-lg-0">
            <a href="#" role="button" class="btn dropdown-toggle" data-toggle="dropdown">
                <i class="fa fa-folder-open"/>
                <t t-if="current_type" t-esc="current_type.name"/>
                <t t-else="">All Categories</t>
            </a>
            <div class="dropdown-menu">
                <t t-foreach="types" t-as="type">
                    <t t-if="type['event_type_id']">
                        <a t-att-href="keep('/event', type=type['event_type_id'][0])" t-attf-class="dropdown-item d-flex align-items-center justify-content-between #{searches.get('type') == str(type['event_type_id'] and type['event_type_id'][0]) and 'active'}">
                            <t t-esc="type['event_type_id'][1]"/>
                            <span t-esc="type['event_type_id_count']" class="badge badge-pill badge-primary ml-3"/>
                        </a>
                    </t>
                </t>
            </div>
        </li>
    </xpath>
</template>

<!-- Filter - Location -->
<template id="event_location" inherit_id="website_event.index_topbar" active="True" customize_show="True" name="Filter by Country">
    <xpath expr="//ul[hasclass('o_wevent_index_topbar_filters')]" position="inside">
        <li class="nav-item dropdown mr-2 my-1 my-lg-0">
            <a href="#" role="button" class="btn dropdown-toggle" data-toggle="dropdown">
                <i class="fa fa-map-marker"/>
                <t t-if="current_country" t-esc="current_country.name"/>
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
                        <a t-att-href="keep('/event', country='online')" t-attf-class="dropdown-item d-flex align-items-center #{searches.get('country') == 'online' and 'active'}">
                            <span>Online Events</span>
                            <span t-esc="country['country_id_count']" class="badge badge-pill badge-primary ml-3"/>
                        </a>
                    </t>
                </t>
            </div>
        </li>
    </xpath>
</template>

<!-- Search Box -->
<template id="events_search_box" inherit_id="website.website_search_box" primary="True">
    <xpath expr="//div[@role='search']" position="replace">
        <form t-attf-class="o_wevent_event_searchbar_form o_wait_lazy_js my-1 my-lg-0 #{_classes}"
              t-att-action="action if action else '/event'" method="get">
            <t t-set="search" t-value="search or _searches and _searches['search']"/>
            <t>$0</t>
            <t t-foreach="_searches" t-as="search">
                <input t-if="search != 'search' and search_value != 'all'" type="hidden"
                    t-att-name="search" t-att-value="search_value"/>
            </t>
            <t t-raw="0"/>
        </form>
    </xpath>
</template>

<!-- Index - Events list -->
<template id="events_list" name="Events list">
    <!-- Options -->
    <t t-set="opt_index_sidebar" t-value="request.website.viewref('website_event.opt_index_sidebar').active"/>
    <t t-if="opt_events_list_columns" t-set="opt_event_size" t-value="opt_index_sidebar and 'col-md-6' or 'col-md-6 col-lg-4'"/>
    <t t-else="" t-set="opt_event_size" t-value="opt_index_sidebar and 'col-12' or 'col-xl-10 offset-xl-1'"/>
    <!-- No events -->
    <t t-if="not event_ids">
        <div class="col-12">
            <div class="h2 mb-3">No events found.</div>
            <div class="alert alert-info text-center" groups="event.group_event_manager">
                <p class="m-0">Use the top button '<b>+ New</b>' to create an event.</p>
            </div>
        </div>
    </t>
    <!-- List -->
    <div t-foreach="event_ids" t-as="event" t-attf-class=" #{opt_event_size} mb-4">
        <article t-attf-class="#{opt_events_list_cards and 'card border-0 shadow-sm'}" itemscope="itemscope" itemtype="http://schema.org/Event">
            <div class="row no-gutters">
                <!-- Header -->
                <header t-attf-class="overflow-hidden bg-secondary #{opt_events_list_columns and 'col-12 rounded-top' or 'col-sm-4 col-lg-3 rounded-left'} #{(not opt_events_list_cards) and 'rounded shadow'} #{(not opt_events_list_cards and not opt_events_list_columns) and 'rounded-top'}">
                    <!-- Image + Link -->
                    <a t-attf-href="/event/#{ slug(event) }/#{(not event.menu_id) and 'register'}" class="d-block h-100 w-100">
                        <t t-call="website.record_cover">
                            <t t-set="_record" t-value="event"/>

                            <!-- Short Date -->
                            <div class="o_wevent_event_date position-absolute bg-white shadow-sm">
                                <span t-field="event.with_context(tz=event.date_tz).date_begin" t-options="{'format': 'LLL'}" class="o_wevent_event_month"/>
                                <span t-field="event.with_context(tz=event.date_tz).date_begin" t-options="{'format': 'dd'}" class="o_wevent_event_day"/>
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
                    </a>
                </header>
                <div t-attf-class="#{opt_events_list_columns and 'col-12' or 'col'}">
                    <!-- Body -->
                    <main t-attf-class="#{opt_events_list_cards and 'card-body' or (opt_events_list_columns and 'py-3' or 'px-4')}">
                        <!-- Title -->
                        <h5 t-attf-class="card-title mt-2 mb-0 text-truncate #{(not event.website_published) and 'text-danger'}">
                            <a t-attf-href="/event/#{ slug(event) }/#{(not event.menu_id) and 'register'}" class="text-reset text-decoration-none" itemprop="url">
                                <span t-field="event.name" itemprop="name"/>
                            </a>
                        </h5>
                        <!-- Organizer -->
                        <div t-if="event.organizer_id" class="mb-3">
                            <small class="text-muted text-truncate">Organizer: <span t-field="event.organizer_id" itemprop="organizer"/></small>
                        </div>
                        <!-- Location -->
                        <div t-if="event.is_online">Online</div>
                        <div t-else="" itemprop="location" t-field="event.address_id" t-options="{'widget': 'contact', 'fields': ['city'], 'no_marker': 'true'}"/>
                        <!-- Start Date & Time -->
                        <time itemprop="startDate" t-att-datetime="event.date_begin">
                            <span t-field="event.with_context(tz=event.date_tz).date_begin" t-options="{'date_only': 'true', 'format': 'long'}"/> -
                            <span t-field="event.with_context(tz=event.date_tz).date_begin" t-options="{'time_only': 'true', 'format': 'short'}"/>
                        </time>
                    </main>
                    <!-- Footer -->
                    <footer t-attf-class="small #{opt_events_list_cards and 'card-footer' or ((not opt_events_list_columns) and 'border-top mx-4 mt-auto pt-2' or 'border-top py-2')}">
                        <div itemprop="price"><span content="0" class="font-weight-bold text-uppercase"></span></div>
                    </footer>
                </div>
            </div>
        </article>
    </div>
    <!-- Pager -->
    <div class="form-inline justify-content-center my-3">
        <t t-call="website.pager"/>
    </div>
</template>

<template id="opt_events_list_columns" inherit_id="website_event.events_list" active="True" customize_show="True" name="Layout • Columns"/>

<template id="opt_events_list_cards" inherit_id="website_event.events_list" active="True" customize_show="True" name="'Cards' Design"/>

<template id="opt_events_list_categories" inherit_id="website_event.events_list" active="True" customize_show="True" name="Show categories">
    <xpath expr="//main/*" position="before">
        <a t-if="event.event_type_id" t-attf-href="/event?type=#{event.event_type_id.id}" t-attf-class="badge bg-secondary o_wevent_badge #{opt_events_list_columns and 'o_wevent_badge_event' or 'float-right'}" t-field="event.event_type_id"/>
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
                <figcaption class="figure-caption">A past event</figcaption>
            </figure>
        </a>
        <a href="/event">
            <figure class="o_wevent_sidebar_block o_wevent_sidebar_figure figure">
                <img class="figure-img img-fluid rounded" src="/website_event/static/src/img/event_training_0.jpg" alt=""/>
                <figcaption class="figure-caption">Our Trainings</figcaption>
            </figure>
        </a>
    </xpath>
</template>

<!-- Index - Sidebar - Categories -->
<template id="index_sidebar_categories" inherit_id="website_event.index_sidebar" active="True" customize_show="True" name="Categories" priority="50">
    <xpath expr="//div[@id='o_wevent_index_sidebar']" position="inside">
        <div class="o_wevent_sidebar_block">
            <h6 class="o_wevent_sidebar_title">Categories</h6>
            <t t-foreach="types" t-as="type">
                <t t-if="type['event_type_id']">
                    <a t-att-href="keep('/event', type=type['event_type_id'][0])" class="badge badge-secondary o_wevent_badge mb-2"><t t-esc="type['event_type_id'][1]"/></a>
                </t>
            </t>
        </div>
    </xpath>
</template>

<!-- Index - Sidebar - Quotes -->
<template id="index_sidebar_quotes" inherit_id="website_event.index_sidebar" active="True" customize_show="True" name="Quotes" priority="60">
    <xpath expr="//div[@id='o_wevent_index_sidebar']" position="inside">
        <div class="o_wevent_sidebar_block card">
            <div class="card-body">
                <blockquote class="blockquote mb-0">
                    <p><em>Write here a quote from one of your attendees. It gives confidence in your events.</em></p>
                    <footer class="blockquote-footer">Author</footer>
                </blockquote>
            </div>
        </div>
    </xpath>
</template>

<!-- Index - Sidebar - Snippet - Country Events -->
<template id="index_sidebar_country_event" inherit_id="website_event.index_sidebar" active="True" customize_show="True" name="Country Events" priority="70">
    <xpath expr="//div[@id='o_wevent_index_sidebar']" position="inside">
        <div class="o_wevent_sidebar_block">
            <t t-call="website_event.s_country_events"/>
        </div>
    </xpath>
</template>

<!-- Event -->
<template id="layout" name="Event">
    <t t-call="website.layout">
        <div id="wrap" class="o_wevent_event js_event">
            <t t-if="not event.menu_id">
                <nav class="navbar navbar-light border-top shadow-sm d-print-none">
                    <div class="container">
                        <a href="/event" class="navbar-brand h4 my-0 mr-4">
                            <i class="fa fa-long-arrow-left text-primary mr-2"/>All Events
                        </a>
                        <ul class="navbar-nav flex-row">
                            <li class="nav-item mr-3">
                                <a t-attf-href="/event?type=#{event.event_type_id.id}" t-if="event.event_type_id" class="nav-link">
                                    <i class="fa fa-folder-open text-primary mr-2"/><t t-esc="event.event_type_id.name"/>
                                </a>
                            </li>
                            <li class="nav-item mr-3">
                                <a t-if="event.country_id" t-attf-href="/event?country=#{event.country_id.id}" class="nav-link">
                                    <i class="fa fa-map-marker text-primary mr-2"/><t t-esc="event.country_id.name"/>
                                </a>
                            </li>
                        </ul>
                        <t t-call="website_event.events_search_box">
                            <t t-set="_classes" t-value="'ml-auto'"/>
                            <t t-set="_searches" t-value="searches"/>
                        </t>
                    </div>
                </nav>
            </t>
            <t t-else="">
                <nav class="navbar navbar-light border-top shadow-sm navbar-expand-md">
                    <div class="container">
                        <a href="#" t-field="event.name" class="navbar-brand h4 my-0 mr-4"/>
                        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#o_wevent_event_submenu" aria-controls="o_wevent_event_submenu" aria-expanded="false" aria-label="Toggle navigation">
                            <span class="navbar-toggler-icon"></span>
                        </button>
                        <div id="o_wevent_event_submenu" class="collapse navbar-collapse">
                            <ul class="navbar-nav w-100" t-att-data-menu_name="editable and 'Event Menu'" t-att-data-content_menu_id="editable and event.menu_id.id">
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
            </t>
            <t t-raw="0"/>
            <div class="oe_structure" id="oe_structure_website_event_layout_1" t-att-data-editor-sub-message="'Following content will appear on all events.'"/>
        </div>
    </t>
</template>

<template id="event_details" name="Event Header">
    <t t-call="website_event.layout">
        <div name="event" itemscope="itemscope" itemtype="http://schema.org/Event">
            <t t-call="website.record_cover">
                <t t-set="_record" t-value="event"/>
                <t t-set="use_filters" t-value="True"/>
                <t t-set="use_size" t-value="True"/>
                <t t-set="use_text_size" t-value="True"/>
                <t t-set="use_text_align" t-value="True"/>

                <div class="container d-flex flex-column flex-grow-1 justify-content-around">
                    <div class="o_wevent_event_title">
                        <span t-if="event.is_participating" class="badge badge-success o_wevent_badge"><i class="fa fa-check mr-2"/>Registered</span>
                        <h1 t-field="event.name" class="o_wevent_event_name" itemprop="name" placeholder="Event Title"/>
                        <h2 t-field="event.subtitle" class="o_wevent_event_subtitle" placeholder="Event Subtitle"/>
                    </div>
                </div>
                <div class="container mb-5">
                    <t t-call="website_event.registration_template"/>
                </div>
            </t>
            <t t-raw="0"/>
        </div>
    </t>
</template>

<!-- Event - Description -->
<template id="event_description_full" name="Event Description">
    <t t-call="website_event.event_details">
        <section class="bg-200">
            <div class="container">
                <div class="row no-gutters mt-n5 pb-5">
                    <!-- Description -->
                    <div id="o_wevent_event_main_col" class="col-lg-8 bg-white rounded-left p-5 shadow-sm">
                        <span t-field="event.description" itemprop="description"/>
                    </div>
                    <div class="col-lg-4 bg-light rounded-right shadow-sm d-print-none">
                        <!-- Date & Time -->
                        <div class="o_wevent_sidebar_block">
                            <h6 class="o_wevent_sidebar_title">Date &amp; Time</h6>
                            <span t-field="event.with_context(tz=event.date_tz).date_begin" t-options="{'date_only': 'true', 'format': 'EEEE'}"/>
                            <h4 class="my-1" t-field="event.with_context(tz=event.date_tz).date_begin" t-options="{'date_only': 'true', 'format': 'long'}" itemprop="startDate" t-att-datetime="event.date_begin"/>
                            <t t-if="not event.is_one_day">Start -</t>
                            <span t-field="event.with_context(tz=event.date_tz).date_begin" t-options="{'time_only': 'true', 'format': 'short'}"/>
                            <t t-if="event.is_one_day">
                                <i class="fa fa-long-arrow-right mx-1"/>
                                <span t-field="event.with_context(tz=event.date_tz).date_end" t-options="{'time_only': 'true', 'format': 'short'}"/>
                            </t>
                            <t t-else="">
                                <i class="fa fa-long-arrow-down d-block text-muted mx-3 my-2" style="font-size: 1.5rem"/>
                                <span t-field="event.with_context(tz=event.date_tz).date_end" t-options="{'date_only': 'true', 'format': 'EEEE'}"/>
                                <h4 class="my-1" t-field="event.with_context(tz=event.date_tz).date_end" t-options="{'date_only': 'true', 'format': 'long'}"/>
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
                        <t t-if="not event.is_online">
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
                                <a t-att-href="event.google_map_link()" target="_blank">Get the direction</a>
                            </div>
                        </t>
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
                            <p t-if="event.twitter_hashtag" class="font-weight-bold">Hashtag: <a t-att-href="'http://twitter.com/search?q=%23'+event.twitter_hashtag" target="_blank">#<span t-field="event.twitter_hashtag"/></a></p>
                            <t t-call="website.s_share">
                                <t t-set="_no_title" t-value="True"/>
                                <t t-set="_classes">o_wevent_sidebar_social mx-n1</t>
                                <t t-set="_link_classes">o_wevent_social_link</t>
                            </t>
                        </div>
                    </div>
                </div>
                <ul class="list-unstyled" id="comment">
                    <li t-foreach="event.website_message_ids" t-as="comment" class="media mt-3">
                        <div class="media-body">
                            <t t-call="website.publish_management">
                                <t t-set="object" t-value="comment"/>
                                <t t-set="publish_edit" t-value="True"/>
                            </t>
                            <t t-raw="comment.body"/>
                            <small class="float-right muted text-right">
                                <div t-field="comment.author_id"/>
                                <div t-field="comment.date" t-options='{"hide_seconds":"True"}'/>
                            </small>
                        </div>
                    </li>
                </ul>
            </div>
        </section>
    </t>
</template>

<!-- Event - Registration -->
<template id="registration_template">
    <t t-set="tickets_available" t-value="event.seats_available or event.seats_availability == 'unlimited'"/>
    <t t-set="buy" t-value="tickets_available and event.state == 'confirm'"/>
    <form t-if="buy" id="registration_form" t-attf-action="/event/#{slug(event)}/registration/new" method="post" itemscope="itemscope" itemprop="offers" itemtype="http://schema.org/AggregateOffer" class="mb-5">
        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
        <div id="o_wevent_tickets" class="bg-white rounded shadow-sm">
            <t t-id="tickets" t-call="website_event.ticket">
                <t t-set="name">Registration</t>
                <t t-set="price">
                    <span content="0" itemprop="price" class="font-weight-bold text-uppercase">Free</span>
                </t>
                <t t-if="event.date_begin">
                    <t t-set="registration_end">
                        End on <span t-field="event.with_context(tz=event.date_tz).date_begin" t-options="{'date_only': 'true', 'format': 'medium'}"/>
                    </t>
                </t>
                <t t-set="quantity">
                    <select name="nb_register-0" class="custom-select w-auto">
                        <t t-foreach="range(0, (event.seats_availability == 'unlimited' or event.seats_available &gt; 9) and 10 or event.seats_available+1)" t-as="nb">
                            <option t-esc="nb" t-att-selected="nb == 1 and 'selected'"/>
                        </t>
                    </select>
                </t>
            </t>
        </div>
    </form>
    <div t-if="not buy" class="alert alert-info mb-5" role="status">
        <span t-if="event.state == 'draft'" itemprop="availability" content="http://schema.org/OutOfStock">
            <b>Event registration not yet started.</b>
        </span>
        <span t-if="event.state != 'draft'" itemprop="availability" content="http://schema.org/Discontinued">
            <b>Event registration is closed.</b>
        </span>
        <a t-if="request.env.user.has_group('event.group_event_manager')" t-attf-href="/web#id=#{event.id}&amp;view_type=form&amp;model=event.event" class="float-right">
            <i class="fa fa-pencil mr-2" role="img" aria-label="Create" title="Create"/><em>Configure Event Registration</em>
        </a>
    </div>
</template>

<template id="ticket" name="Ticket offer template">
    <div class="row p-2 pl-3">
        <div class="col-lg-8 d-flex flex-columns align-items-center" itemscope="itemscope" itemtype="http://schema.org/Offer">
            <h6 t-raw="name" itemprop="name" class="my-0 pr-3"/>
            <div class="border-left border-right px-3"><t t-raw="price"/></div>
            <small t-raw="registration_end" class="text-muted ml-3" itemprop="availabilityEnds"/>
            <div class="ml-auto">
                <t t-if="tickets_available">
                    <span class="font-weight-bold align-middle pr-2">Qty</span>
                    <link itemprop="availability" content="http://schema.org/InStock"/>
                    <t t-raw="quantity"/>
                </t>
                <t t-else="">
                    <span itemprop="availability" content="http://schema.org/SoldOut" class="text-danger">
                        <i class="fa fa-ban mr-2"/>Sold Out
                    </span>
                </t>
            </div>
        </div>
        <div class="col-lg-4 pt-3 pt-lg-0 pl-2 pl-lg-0">
            <button type="submit" class="btn btn-primary o_wait_lazy_js btn-block a-submit" t-attf-id="#{event.id}" disabled="disabled">Register</button>
        </div>
    </div>
    <div t-if="description" class="row mx-0">
        <div class="col-12 bg-200 rounded-bottom">
            <p itemprop="description" t-raw="description" class="small my-2"/>
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
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span>&amp;times;</span></button>
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
                                        <input class="form-control" type="text" t-attf-name="#{counter}-name" required="This field is required"/>
                                    </div>
                                    <div class="col-lg my-2">
                                        <label>Email</label>
                                        <input class="form-control" type="email" t-attf-name="#{counter}-email" required="This field is required"/>
                                    </div>
                                    <div class="col-lg my-2">
                                        <label>Phone <small>(Optional)</small></label>
                                        <input class="form-control" type="tel" t-attf-name="#{counter}-phone"/>
                                    </div>
                                    <input class="d-none" type="text" t-attf-name="#{counter}-ticket_id" t-attf-value="#{ticket['id']}"/>
                                </div>
                            </div>
                        </t>
                        <t t-set="counter_type" t-value="counter_type + 1"/>
                    </t>
                    <t t-if="not availability_check">
                        <strong> You ordered more tickets than available seats</strong>
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
        <div class="container my-5">
            <div class="row">
                <div class="col-12">
                    <h2>Registration confirmed!</h2>
                    <table class="table table-striped my-4">
                        <thead class="bg-secondary">
                            <tr>
                                <th>Name</th>
                                <th>E-mail</th>
                                <th>Phone</th>
                                <th>Reference</th>
                            </tr>
                        </thead>
                        <tbody>
                            <t t-foreach="attendees" t-as="attendee">
                                <tr>
                                    <td><t t-if="attendee.name" t-esc="attendee.name"/><t t-else="">N/A</t></td>
                                    <td><t t-if="attendee.email" t-esc="attendee.email"/><t t-else="">N/A</t></td>
                                    <td><t t-if="attendee.phone" t-esc="attendee.phone"/><t t-else="">N/A</t></td>
                                    <td><t t-esc="attendee.id"/></td>
                                </tr>
                            </t>
                        </tbody>
                    </table>
                </div>
                <div class="col-lg-8">
                    <h3><a t-attf-href="/event/#{slug(event)}"><t t-esc="event.name"/></a></h3>
                    <p><b>Start</b> <span itemprop="startDate" t-esc="event.date_begin_located"/><br/> <b>End</b> <span itemprop="endDate" t-esc="event.date_end_located"/></p>
                    <div id="add_to_calendar" class="mt-4">
                        <a role="button" class="btn btn-primary" t-att-href="iCal_url">
                            <i class="fa fa-fw fa-calendar"/> Add to iCal/Outlook
                        </a>
                        <a role="button" class="btn btn-primary" t-att-href="google_url" target='_blank'>
                            <i class="fa fa-fw fa-calendar"/> Add to Google Calendar
                        </a>
                    </div>
                </div>
                <div class="col-lg-4">
                    <h4 t-field="event.address_id" class="text-secondary font-weight-bold" t-options='{
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
            </div>
        </div>
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

<!-- Multipage event - Default template when creating a new page -->
<template id="default_page">
    <t t-call="website.layout">
        <div class="oe_structure oe_empty"/>
    </t>
</template>

<!-- Multipage event - Default template for the Introduction page -->
<template id="template_intro">
    <t t-call="website_event.layout">
        <section>
            <div class="container">
                <div class="row">
                    <div class="col-lg-12 s_title pt48 pb48" data-name="Title">
                        <h1>Introduction</h1>
                    </div>
                </div>
            </div>
        </section>
        <div class="oe_structure oe_empty" id="oe_structure_website_event_intro_1"/>
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

<!-- User Navbar -->
<template id="user_navbar_inherit_website_event" inherit_id="website.user_navbar">
    <xpath expr="//div[@id='o_new_content_menu_choices']//div[@name='module_website_event']" position="attributes">
        <attribute name="name"/>
        <attribute name="t-att-data-module-id"/>
        <attribute name="t-att-data-module-shortdesc"/>
        <attribute name="groups">event.group_event_manager</attribute>
    </xpath>
</template>

<!-- User Navbar - Edit Options -->
<template id="event_edit_options" inherit_id="website.user_navbar" name="Edit Event Options">
    <xpath expr="//li[@id='edit-page-menu']" position="after">
        <t t-if="main_object._name == 'event.event'" t-set="action" t-value="'event.action_event_view'"/>
    </xpath>
</template>

</odoo>

```

## File: views\event_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="event_type_view_form_inherit_website" model="ir.ui.view">
        <field name="name">event.type.view.form.inherit.website</field>
        <field name="model">event.type</field>
        <field name="inherit_id" ref="event.view_event_type_form"/>
        <field name="arch" type="xml">
            <div name="event_type_visibility_seats" position="after">
                <div class="col-12 col-lg-6 o_setting_box" name="event_type_visibility_website">
                    <div class="o_setting_left_pane">
                        <field name="website_menu"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="website_menu"/>
                        <div class="row">
                            <div class="col-lg-8 mt16 text-muted">
                                Check this option to have menus for your event on the
                                website: registrations, schedule, map, ...
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </field>
    </record>

    <record id="event_event_view_form_inherit_website" model="ir.ui.view">
        <field name="name">event.event.view.form.inherit.website</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="event.view_event_form"/>
        <field name="arch" type="xml">
            <field name="organizer_id" position="before">
                <field name="website_id" options="{'no_create': True}" domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]" groups="website.group_multi_website"/>
            </field>
            <div name="button_box" position="inside">
                <field name="website_url" invisible="1"/>
                <field name="is_published" widget="website_redirect_button"/>
            </div>
            <label for="is_online" position="after">
                <field name="website_menu"/>
                <label for="website_menu" string="Website Menu"/>
            </label>
            <xpath expr="//button[@name='button_done']" position="before">
                <button name="action_open_badge_editor"
                    type="object"
                    states="confirm"
                    string="Preview Badges"
                    class="oe_highlight"/>
                <button name="action_open_badge_editor"
                    type="object"
                    states="done"
                    string="Preview Badges"/>
            </xpath>
        </field>
    </record>

    <record id="event_event_view_list_inherit_website" model="ir.ui.view">
        <field name="name">event.event.view.list.inherit.website</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="event.view_event_tree"/>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="website_id" groups="website.group_multi_website" domain="['|', ('company_id', '=', company_id), ('company_id', '=', False)]"/>
            </field>
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

