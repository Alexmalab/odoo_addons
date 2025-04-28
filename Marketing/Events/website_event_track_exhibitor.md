# Odoo Module: website_event_track_exhibitor

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
    'name': 'Event Exhibitors',
    'category': 'Marketing/Events',
    'sequence': 1004,
    'version': '1.0',
    'summary': 'Event: upgrade sponsors to exhibitors',
    'website': 'https://www.odoo.com/page/events',
    'description': "",
    'depends': [
        'website_event_track',
        'website_jitsi',
    ],
    'data': [
        'security/ir.model.access.csv',
        'security/security.xml',
        'views/assets.xml',
        'views/event_sponsor_views.xml',
        'views/event_event_views.xml',
        'views/event_exhibitor_templates_list.xml',
        'views/event_exhibitor_templates_page.xml',
        'views/event_type_views.xml',
    ],
    'demo': [
        'data/event_demo.xml',
        'data/event_sponsor_demo.xml',
    ],
    'application': False,
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\exhibitor.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval
from random import randint, sample
from werkzeug.exceptions import NotFound, Forbidden

from odoo import exceptions, http
from odoo.addons.website_event_track.controllers.event_track import EventTrackController
from odoo.http import request
from odoo.osv import expression
from odoo.tools import format_duration


class ExhibitorController(EventTrackController):

    def _get_event_sponsors_base_domain(self, event):
        search_domain_base = [
            ('event_id', '=', event.id),
            ('is_exhibitor', '=', True),
        ]
        if not request.env.user.has_group('event.group_event_user'):
            search_domain_base = expression.AND([search_domain_base, [('is_published', '=', True)]])
        return search_domain_base

    # ------------------------------------------------------------
    # MAIN PAGE
    # ------------------------------------------------------------

    @http.route(['/event/<model("event.event"):event>/exhibitors'], type='http', auth="public", website=True, sitemap=False)
    def event_exhibitors(self, event, **searches):
        if not event.can_access_from_current_website():
            raise NotFound()

        return request.render(
            "website_event_track_exhibitor.event_exhibitors",
            self._event_exhibitors_get_values(event, **searches)
        )

    def _event_exhibitors_get_values(self, event, **searches):
        # init and process search terms
        searches.setdefault('search', '')
        searches.setdefault('countries', '')
        searches.setdefault('sponsorships', '')
        search_domain_base = self._get_event_sponsors_base_domain(event)
        search_domain = search_domain_base

        # search on content
        if searches.get('search'):
            search_domain = expression.AND([
                search_domain,
                ['|', ('name', 'ilike', searches['search']), ('website_description', 'ilike', searches['search'])]
            ])

        # search on countries
        search_countries = self._get_search_countries(searches['countries'])
        if search_countries:
            search_domain = expression.AND([
                search_domain,
                [('partner_id.country_id', 'in', search_countries.ids)]
            ])

        # search on sponsor types
        search_sponsorships = self._get_search_sponsorships(searches['sponsorships'])
        if search_sponsorships:
            search_domain = expression.AND([
                search_domain,
                [('sponsor_type_id', 'in', search_sponsorships.ids)]
            ])

        # fetch data to display; use sudo to allow reading partner info, be sure domain is correct
        event = event.with_context(tz=event.date_tz or 'UTC')
        sponsors = request.env['event.sponsor'].sudo().search(search_domain)
        sponsors_all = request.env['event.sponsor'].sudo().search(search_domain_base)
        sponsor_types = sponsors_all.mapped('sponsor_type_id')
        sponsor_countries = sponsors_all.mapped('partner_id.country_id').sorted('name')
        # organize sponsors into categories to help display
        sponsor_categories = dict()
        for sponsor in sponsors:
            if not sponsor_categories.get(sponsor.sponsor_type_id):
                sponsor_categories[sponsor.sponsor_type_id] = request.env['event.sponsor'].sudo()
            sponsor_categories[sponsor.sponsor_type_id] |= sponsor
        sponsor_categories = [
            dict({
                'sponsorship': sponsor_category,
                'sponsors': sample(sponsors, len(sponsors)),
            }) for sponsor_category, sponsors in sponsor_categories.items()]

        # return rendering values
        return {
            # event information
            'event': event,
            'main_object': event,
            'sponsor_categories': sponsor_categories,
            'hide_sponsors': True,
            # search information
            'searches': searches,
            'search_key': searches['search'],
            'search_countries': search_countries,
            'search_sponsorships': search_sponsorships,
            'sponsor_types': sponsor_types,
            'sponsor_countries': sponsor_countries,
            # environment
            'hostname': request.httprequest.host.split(':')[0],
            'user_event_manager': request.env.user.has_group('event.group_event_manager'),
        }

    # ------------------------------------------------------------
    # FRONTEND FORM
    # ------------------------------------------------------------

    @http.route(['''/event/<model("event.event", "[('exhibitor_menu', '=', True)]"):event>/exhibitor/<model("event.sponsor", "[('event_id', '=', event.id)]"):sponsor>'''],
                type='http', auth="public", website=True, sitemap=True)
    def event_exhibitor(self, event, sponsor, **options):
        if not event.can_access_from_current_website():
            raise NotFound()

        try:
            sponsor.check_access_rule('read')
        except exceptions.AccessError:
            raise Forbidden()
        sponsor = sponsor.sudo()

        if 'widescreen' not in options and sponsor.chat_room_id and sponsor.is_in_opening_hours:
            options['widescreen'] = True

        return request.render(
            "website_event_track_exhibitor.event_exhibitor_main",
            self._event_exhibitor_get_values(event, sponsor, **options)
        )

    def _event_exhibitor_get_values(self, event, sponsor, **options):
        # search for exhibitor list
        search_domain_base = self._get_event_sponsors_base_domain(event)
        search_domain_base = expression.AND([
            search_domain_base,
            [('id', '!=', sponsor.id)]
        ])
        sponsors_other = request.env['event.sponsor'].sudo().search(search_domain_base)
        current_country = sponsor.partner_id.country_id

        sponsors_other = sponsors_other.sorted(key=lambda sponsor: (
            sponsor.is_in_opening_hours,
            sponsor.partner_id.country_id == current_country,
            -1 * sponsor.sponsor_type_id.sequence,
            randint(0, 20)
        ), reverse=True)

        option_widescreen = options.get('widescreen', False)
        option_widescreen = bool(option_widescreen) if option_widescreen != '0' else False

        return {
            # event information
            'event': event,
            'main_object': sponsor,
            'sponsor': sponsor,
            'hide_sponsors': True,
            # sidebar
            'sponsors_other': sponsors_other[:30],
            # options
            'option_widescreen': option_widescreen,
            'option_can_edit': request.env.user.has_group('event.group_event_manager'),
            # environment
            'hostname': request.httprequest.host.split(':')[0],
            'user_event_manager': request.env.user.has_group('event.group_event_manager'),
        }

    # ------------------------------------------------------------
    # BUSINESS / MISC
    # ------------------------------------------------------------

    @http.route('/event_sponsor/<int:sponsor_id>/read', type='json', auth='public', website=True)
    def event_sponsor_read(self, sponsor_id):
        """ Marshmalling data for "event not started / sponsor not available" modal """
        sponsor = request.env['event.sponsor'].browse(sponsor_id)
        sponsor_data = sponsor.read([
            'name', 'subtitle',
            'url', 'email', 'phone',
            'website_description', 'website_image_url',
            'hour_from', 'hour_to', 'is_in_opening_hours',
            'event_date_tz', 'country_flag_url',
        ])[0]
        if sponsor.country_id:
            sponsor_data['country_name'] = sponsor.country_id.name
            sponsor_data['country_id'] = sponsor.country_id.id
        else:
            sponsor_data['country_name'] = False
            sponsor_data['country_id'] = False
        if sponsor.sponsor_type_id:
            sponsor_data['sponsor_type_name'] = sponsor.sponsor_type_id.name
            sponsor_data['sponsor_type_id'] = sponsor.sponsor_type_id.id
        else:
            sponsor_data['sponsor_type_name'] = False
            sponsor_data['sponsor_type_id'] = False
        sponsor_data['event_name'] = sponsor.event_id.name
        sponsor_data['event_is_ongoing'] = sponsor.event_id.is_ongoing
        sponsor_data['event_is_done'] = sponsor.event_id.is_done
        sponsor_data['event_start_today'] = sponsor.event_id.start_today
        sponsor_data['event_start_remaining'] = sponsor.event_id.start_remaining
        sponsor_data['event_date_begin_located'] = sponsor.event_id.date_begin_located
        sponsor_data['event_date_end_located'] = sponsor.event_id.date_end_located
        sponsor_data['hour_from_str'] = format_duration(sponsor_data['hour_from'])
        sponsor_data['hour_to_str'] = format_duration(sponsor_data['hour_to'])

        return sponsor_data

    # ------------------------------------------------------------
    # TOOLS
    # ------------------------------------------------------------

    def _get_search_countries(self, country_search):
        # TDE FIXME: make me generic (slides, event, ...)
        try:
            country_ids = literal_eval(country_search)
        except Exception:
            countries = request.env['res.country'].sudo()
        else:
            # perform a search to filter on existing / valid tags implicitly
            countries = request.env['res.country'].sudo().search([('id', 'in', country_ids)])
        return countries

    def _get_search_sponsorships(self, sponsorship_search):
        # TDE FIXME: make me generic (slides, event, ...)
        try:
            sponsorship_ids = literal_eval(sponsorship_search)
        except Exception:
            sponsorships = request.env['event.sponsor.type'].sudo()
        else:
            # perform a search to filter on existing / valid tags implicitly
            sponsorships = request.env['event.sponsor.type'].sudo().search([('id', 'in', sponsorship_ids)])
        return sponsorships

```

## File: controllers\website_event_main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from babel.dates import format_datetime

from odoo import _
from odoo.http import request
from odoo.addons.website_event.controllers.main import WebsiteEventController


class WebsiteEventController(WebsiteEventController):
    def _prepare_event_register_values(self, event, **post):
        values = super(WebsiteEventController, self)._prepare_event_register_values(event, **post)

        if "from_sponsor_id" in post and not event.is_ongoing:
            sponsor = request.env["event.sponsor"].browse(int(post["from_sponsor_id"])).exists()
            if sponsor:
                date_begin = format_datetime(event.with_context(tz=event.date_tz).date_begin, format="medium")

                values["toast_message"] = (
                    _('The event %s starts on %s (%s). \nJoin us there to meet %s !')
                    % (event.name, date_begin, event.date_tz, sponsor.partner_name)
                )

        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import exhibitor
from . import website_event_main

```

## File: data\event_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="event.event_7" model="event.event">
        <field name="exhibitor_menu" eval="True"/>
    </record>

</odoo>

```

## File: data\event_sponsor_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="website_event_track.event_7_sponsor_0" model="event.sponsor">
        <field name="is_published" eval="True"/>
        <field name="subtitle">Catchy sentence !</field>
        <field name="is_exhibitor" eval="False"/>
    </record>
    <record id="website_event_track.event_7_sponsor_1" model="event.sponsor">
        <field name="is_published" eval="True"/>
        <field name="subtitle">You won't believe this sponsor</field>
        <field name="is_exhibitor" eval="False"/>
    </record>
    <record id="website_event_track.event_7_sponsor_2" model="event.sponsor">
        <field name="is_published" eval="True"/>
        <field name="subtitle">Honestly we are good</field>
        <field name="is_exhibitor" eval="True"/>
        <field name="room_name">gemini-furniture</field>
        <field name="room_max_capacity">12</field>
    </record>
    <record id="website_event_track.event_7_sponsor_3" model="event.sponsor">
        <field name="is_published" eval="True"/>
        <field name="subtitle">Catchy sentence !</field>
        <field name="is_exhibitor" eval="True"/>
        <field name="room_name">ready-mat</field>
        <field name="room_max_capacity">12</field>
    </record>
    <record id="website_event_track.event_7_sponsor_4" model="event.sponsor">
        <field name="is_published" eval="True"/>
        <field name="subtitle">You won't believe this sponsor</field>
        <field name="is_exhibitor" eval="True"/>
        <field name="room_name">the-jackson-group</field>
        <field name="room_max_capacity">12</field>
    </record>
    <record id="website_event_track.event_7_sponsor_5" model="event.sponsor">
        <field name="is_published" eval="True"/>
        <field name="subtitle">Honestly we are good</field>
        <field name="is_exhibitor" eval="True"/>
        <field name="room_name">azure-interior</field>
        <field name="room_max_capacity">12</field>
    </record>
    <record id="website_event_track.event_7_sponsor_6" model="event.sponsor">
        <field name="is_published" eval="True"/>
        <field name="subtitle">You won't believe this sponsor</field>
        <field name="is_exhibitor" eval="True"/>
        <field name="room_name">bloem-flower</field>
        <field name="room_max_capacity">12</field>
    </record>
    <record id="website_event_track.event_7_sponsor_7" model="event.sponsor">
        <field name="is_published" eval="True"/>
        <field name="subtitle">Honestly we are good</field>
        <field name="is_exhibitor" eval="True"/>
        <field name="room_name">open-wood</field>
        <field name="room_max_capacity">12</field>
    </record>
    <record id="website_event_track.event_7_sponsor_8" model="event.sponsor">
        <field name="is_published" eval="True"/>
        <field name="subtitle">You won't believe this sponsor</field>
        <field name="is_exhibitor" eval="True"/>
        <field name="room_name">tree-dealers</field>
        <field name="room_max_capacity">12</field>
    </record>
    <record id="website_event_track.event_7_sponsor_9" model="event.sponsor">
        <field name="is_published" eval="True"/>
        <field name="subtitle">Honestly we are good</field>
        <field name="is_exhibitor" eval="True"/>
        <field name="room_name">pterocarpus</field>
        <field name="room_max_capacity">12</field>
    </record>

</odoo>

```

## File: models\event_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.addons.http_routing.models.ir_http import slug


class EventEvent(models.Model):
    _inherit = "event.event"

    exhibitor_menu = fields.Boolean(
        string='Showcase Exhibitors', compute='_compute_exhibitor_menu',
        readonly=False, store=True)
    exhibitor_menu_ids = fields.One2many(
        'website.event.menu', 'event_id', string='Exhibitors Menus',
        domain=[('menu_type', '=', 'exhibitor')])

    @api.depends('event_type_id', 'website_menu', 'exhibitor_menu')
    def _compute_exhibitor_menu(self):
        for event in self:
            if event.event_type_id and event.event_type_id != event._origin.event_type_id:
                event.exhibitor_menu = event.event_type_id.exhibitor_menu
            elif event.website_menu and (event.website_menu != event._origin.website_menu or not event.exhibitor_menu):
                event.exhibitor_menu = True
            elif not event.website_menu:
                event.exhibitor_menu = False

    # ------------------------------------------------------------
    # WEBSITE MENU MANAGEMENT
    # ------------------------------------------------------------

    def toggle_exhibitor_menu(self, val):
        self.exhibitor_menu = val

    def _get_menu_update_fields(self):
        return super(EventEvent, self)._get_menu_update_fields() + ['exhibitor_menu']

    def _update_website_menus(self, menus_update_by_field=None):
        super(EventEvent, self)._update_website_menus(menus_update_by_field=menus_update_by_field)
        for event in self:
            if event.menu_id and (not menus_update_by_field or event in menus_update_by_field.get('exhibitor_menu')):
                event._update_website_menu_entry('exhibitor_menu', 'exhibitor_menu_ids', '_get_exhibitor_menu_entries')

    def _get_menu_type_field_matching(self):
        res = super(EventEvent, self)._get_menu_type_field_matching()
        res['exhibitor'] = 'exhibitor_menu'
        return res

    def _get_exhibitor_menu_entries(self):
        self.ensure_one()
        return [(_('Exhibitors'), '/event/%s/exhibitors' % slug(self), False, 60, 'exhibitor')]

```

## File: models\event_sponsor.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, timedelta
from pytz import timezone, utc

from odoo import api, fields, models
from odoo.addons.http_routing.models.ir_http import slug
from odoo.addons.resource.models.resource import float_to_time
from odoo.tools import is_html_empty
from odoo.tools.translate import html_translate


class EventSponsor(models.Model):
    _name = 'event.sponsor'
    _inherit = [
        'event.sponsor',
        'website.published.mixin',
        'chat.room.mixin',
    ]
    _rec_name = 'name'
    _order = 'sponsor_type_id, sequence'

    # description
    subtitle = fields.Char('Slogan', help='Catchy marketing sentence for promote')
    is_exhibitor = fields.Boolean("Exhibitor's Chat")
    website_description = fields.Html(
        'Description', compute='_compute_website_description',
        sanitize_attributes=False, sanitize_form=True, translate=html_translate,
        readonly=False, store=True)
    # live mode
    hour_from = fields.Float('Opening hour', default=8.0)
    hour_to = fields.Float('End hour', default=18.0)
    event_date_tz = fields.Selection(string='Timezone', related='event_id.date_tz', readonly=True)
    is_in_opening_hours = fields.Boolean(
        'Within opening hours', compute='_compute_is_in_opening_hours')
    # chat room
    chat_room_id = fields.Many2one(readonly=False)
    room_name = fields.Char(readonly=False)
    # country information (related to ease frontend templates)
    country_id = fields.Many2one(
        'res.country', string='Country',
        related='partner_id.country_id', readonly=True)
    country_flag_url = fields.Char(
        string='Country Flag',
        compute='_compute_country_flag_url', compute_sudo=True)

    @api.onchange('is_exhibitor')
    def _onchange_is_exhibitor(self):
        """ Keep an explicit onchange to allow configuration of room names, even
        if this field is normally a related on chat_room_id.name. It is not a real
        computed field, an onchange used in form view is sufficient. """
        for sponsor in self:
            if sponsor.is_exhibitor and not sponsor.room_name:
                if sponsor.name:
                    room_name = "odoo-exhibitor-%s" % sponsor.name
                else:
                    room_name = self.env['chat.room']._default_name(objname='exhibitor')
                sponsor.room_name = self._jitsi_sanitize_name(room_name)
            if sponsor.is_exhibitor and not sponsor.room_max_capacity:
                sponsor.room_max_capacity = '8'

    @api.depends('partner_id')
    def _compute_website_description(self):
        for sponsor in self:
            if is_html_empty(sponsor.website_description):
                sponsor.website_description = sponsor.partner_id.website_description

    @api.depends('event_id.is_ongoing', 'hour_from', 'hour_to', 'event_id.date_begin', 'event_id.date_end')
    def _compute_is_in_opening_hours(self):
        """ Opening hours: hour_from and hour_to are given within event TZ or UTC.
        Now() must therefore be computed based on that TZ. """
        for sponsor in self:
            if not sponsor.event_id.is_ongoing:
                sponsor.is_in_opening_hours = False
            elif sponsor.hour_from is False or sponsor.hour_to is False:
                sponsor.is_in_opening_hours = True
            else:
                event_tz = timezone(sponsor.event_id.date_tz)
                # localize now, begin and end datetimes in event tz
                dt_begin = sponsor.event_id.date_begin.astimezone(event_tz)
                dt_end = sponsor.event_id.date_end.astimezone(event_tz)
                now_utc = utc.localize(fields.Datetime.now().replace(microsecond=0))
                now_tz = now_utc.astimezone(event_tz)

                # compute opening hours
                opening_from_tz = event_tz.localize(datetime.combine(now_tz.date(), float_to_time(sponsor.hour_from)))
                opening_to_tz = event_tz.localize(datetime.combine(now_tz.date(), float_to_time(sponsor.hour_to)))
                if sponsor.hour_to == 0:
                    # when closing 'at midnight', we consider it's at midnight the next day
                    opening_to_tz = opening_to_tz + timedelta(days=1)

                opening_from = max([dt_begin, opening_from_tz])
                opening_to = min([dt_end, opening_to_tz])

                sponsor.is_in_opening_hours = opening_from <= now_tz < opening_to

    @api.depends('partner_id.country_id.image_url')
    def _compute_country_flag_url(self):
        for sponsor in self:
            if sponsor.partner_id.country_id:
                sponsor.country_flag_url = sponsor.partner_id.country_id.image_url
            else:
                sponsor.country_flag_url = False

    # ------------------------------------------------------------
    # MIXINS
    # ---------------------------------------------------------

    @api.depends('name', 'event_id.name')
    def _compute_website_url(self):
        super(EventSponsor, self)._compute_website_url()
        for sponsor in self:
            if sponsor.id:  # avoid to perform a slug on a not yet saved record in case of an onchange.
                base_url = sponsor.event_id.get_base_url()
                sponsor.website_url = '%s/event/%s/exhibitor/%s' % (base_url, slug(sponsor.event_id), slug(sponsor))

    # ------------------------------------------------------------
    # CRUD
    # ------------------------------------------------------------

    @api.model_create_multi
    def create(self, values_list):
        for values in values_list:
            if values.get('is_exhibitor') and not values.get('room_name'):
                exhibitor_name = values['name'] if values.get('name') else self.env['res.partner'].browse(values['partner_id']).name
                name = 'odoo-exhibitor-%s' % exhibitor_name or 'sponsor'
                values['room_name'] = name
        return super(EventSponsor, self).create(values_list)

    def write(self, values):
        toupdate = self.env['event.sponsor']
        if values.get('is_exhibitor') and not values.get('chat_room_id') and not values.get('room_name'):
            toupdate = self.filtered(lambda exhibitor: not exhibitor.chat_room_id)
            # go into sequential update in order to create a custom room name for each sponsor
            for exhibitor in toupdate:
                values['room_name'] = 'odoo-exhibitor-%s' % exhibitor.name
                super(EventSponsor, exhibitor).write(values)
        return super(EventSponsor, self - toupdate).write(values)

    # ------------------------------------------------------------
    # ACTIONS
    # ---------------------------------------------------------

    def get_backend_menu_id(self):
        return self.env.ref('event.event_main_menu').id

```

## File: models\event_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class EventType(models.Model):
    _inherit = "event.type"

    exhibitor_menu = fields.Boolean(
        string='Showcase Exhibitors', compute='_compute_exhibitor_menu',
        readonly=False, store=True,
        help='Display exhibitors on website')

    @api.depends('website_menu')
    def _compute_exhibitor_menu(self):
        for event_type in self:
            event_type.exhibitor_menu = event_type.website_menu

```

## File: models\website_event_menu.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class EventMenu(models.Model):
    _inherit = "website.event.menu"

    menu_type = fields.Selection(
        selection_add=[('exhibitor', 'Exhibitors Menus')],
        ondelete={'exhibitor': 'cascade'})

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_event
from . import event_sponsor
from . import event_type
from . import website_event_menu

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
chat_room_access_event_manager,chat.room.access.event.manager,website_jitsi.model_chat_room,event.group_event_manager,1,1,1,1

```

## File: security\security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="event_sponsor_rule_share" model="ir.rule">
        <field name="name">Event Sponsor: public/portal published only</field>
        <field name="model_id" ref="website_event_track.model_event_sponsor"/>
        <field name="domain_force">[('website_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

</odoo>

```

## File: static\src\js\event_exhibitor_connect.js

```javascript
odoo.define('website_event_track_exhibitor.event_exhibitor_connect', function (require) {
'use strict';

var Dialog = require('web.Dialog');
var publicWidget = require('web.public.widget');

var ExhibitorConnectClosedDialog = Dialog.extend({
    events: _.extend({}, Dialog.prototype.events, {
        'click .o_wesponsor_js_connect_modal_contry': '_onClickCountryFlag',
    }),
    template: 'exhibitor.connect.closed.modal',

    /**
     * @override
     * @param {Object} parent;
     * @param {Object} options holding a sponsorData obj with required values to
     *   display (see .xml for details);
     */
    init: function (parent, options) {
        options = _.defaults(options || {}, {
            size: 'medium',
            renderHeader: false,
            renderFooter: false,
            backdrop: true,
        });
        this.sponsorId = options.sponsorId;
        this._super(parent, options);
    },

    /**
     * @override
     * Wait for fetching sponsor data;
     */
    willStart: function () {
        return Promise.all([
            this._super(...arguments),
            this._fetchSponsor()
        ]);
    },

    //---------------------------------------------------------------------
    // Private
    //---------------------------------------------------------------------

    /**
     * @private
     * @returns {Promise<*>} promise after fetching sponsor data, given its
     *   sponsorId. Necessary to render template content;
     */
    _fetchSponsor: function () {
        let self = this;
        let rpcPromise = this._rpc({
            route: `/event_sponsor/${this.sponsorId}/read`,
        }).then(function (readData) {
            self.sponsorData = readData;
            return Promise.resolve();
        });
        return rpcPromise;
    },
});


publicWidget.registry.eventExhibitorConnect = publicWidget.Widget.extend({
    selector: '.o_wesponsor_js_connect',
    xmlDependencies: ['/website_event_track_exhibitor/static/src/xml/event_exhibitor_connect.xml'],

    /**
     * @override
     * @public
     */
    init: function () {
        this._super(...arguments);
        this._onConnectClick = _.debounce(this._onConnectClick, 500, true);
    },

    /**
     * @override
     * @public
     */
    start: function () {
        var self = this;
        return this._super(...arguments).then(function () {
            self.eventIsOngoing = self.$el.data('eventIsOngoing') || false;
            self.sponsorIsOngoing = self.$el.data('sponsorIsOngoing') || false;
            self.isParticipating = self.$el.data('isParticipating') || false;
            self.userEventManager = self.$el.data('userEventManager') || false;
            self.$el.on('click', self._onConnectClick.bind(self));
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //-------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     * On click, if sponsor is not within opening hours, display a modal instead
     * of redirecting on the sponsor view;
     */
    _onConnectClick: function (ev) {
        ev.stopPropagation();
        ev.preventDefault();

        if (this.userEventManager) {
            document.location = this.$el.data('sponsorUrl');
        } else if (!this.eventIsOngoing && !this.isParticipating) {
            document.location = this.$el.data('registerUrl');
        } else if (!this.eventIsOngoing || ! this.sponsorIsOngoing) {
            return this._openClosedDialog();
        } else {
            document.location = this.$el.data('sponsorUrl');
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _openClosedDialog: function ($element) {
        const sponsorId = this.$el.data('sponsorId');
        return new ExhibitorConnectClosedDialog(
            this, {
                sponsorId: sponsorId,
            }
        ).open();
    },

});


return {
    ExhibitorConnectClosedDialog: ExhibitorConnectClosedDialog,
    eventExhibitorConnect: publicWidget.registry.eventExhibitorConnect,
};

});

```

## File: static\src\xml\event_exhibitor_connect.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="exhibitor.connect.closed.modal">
        <div class="o_wesponsor_js_connect_modal_main container">
            <div class="row mt-2">
                <t t-if="! widget.sponsorData.event_is_ongoing">
                    <div class="col-12 alert alert-warning text-center" role="alert"
                        t-if="widget.sponsorData.event_is_done">
                        Event <span t-esc="widget.sponsorData.event_name" class="font-weight-bold"/> is over.

                        <br/>
                        <span>Join us next time to meet <b t-esc="widget.sponsorData.name"/>!</span>
                    </div>
                    <div class="col-12 alert alert-warning text-center" role="alert"
                        t-else="">
                        <span t-esc="widget.sponsorData.name" class="font-weight-bold"/> is not available right now.<br />
                        Event <span t-esc="widget.sponsorData.event_name" class="font-weight-bold"/>
                        <span t-if="widget.sponsorData.event_start_today">
                            starts in
                            <span t-esc="widget.sponsorData.event_start_remaining"/> minutes
                        </span>
                        <span class="my-0" t-else="">
                            starts on <span t-esc="widget.sponsorData.event_date_begin_located"/>
                        </span>
                    </div>
                </t>
                <div class="col-12 alert alert-warning text-center" role="alert"
                    t-else="">
                    <span t-esc="widget.sponsorData.name" class="font-weight-bold"/> is not available right now.<br />
                    Come back between
                    <strong>
                        <t t-esc="widget.sponsorData.hour_from_str"/>
                        -
                        <t t-esc="widget.sponsorData.hour_to_str"/>
                    </strong> (<span t-esc="widget.sponsorData.event_date_tz"/>)
                    to meet them !
                </div>
                <div class="col-2">
                    <img class="img" style="max-width: 100%;"
                        t-att-src="widget.sponsorData.website_image_url"
                        t-att-alt="widget.sponsorData.name"/>
                </div>
                <div class="col-10">
                    <div class="d-flex align-items-top mb-3">
                        <div class="d-flex flex-column">
                            <div class="d-flex align-items-center">
                                <h4 t-esc="widget.sponsorData.name" class="mb4"/>
                                <span class="badge badge-primary ml-2"
                                    t-esc="widget.sponsorData.sponsor_type_name"/>
                            </div>
                            <span class="text-muted" t-if="widget.sponsorData.subtitle" t-esc="widget.sponsorData.subtitle"/>
                            <span t-if="widget.sponsorData.url">
                                <i class="fa fa-home mr-2"/><a t-att-href="widget.sponsorData.url"><span t-esc="widget.sponsorData.url"/></a>
                            </span>
                            <span t-if="widget.sponsorData.email">
                                <i class="fa fa-envelope mr-2"/><a t-att-mailto="widget.sponsorData.email"><span t-esc="widget.sponsorData.email"/></a>
                            </span>
                            <span t-if="widget.sponsorData.phone">
                                <i class="fa fa-phone mr-2"/><span t-esc="widget.sponsorData.phone"/>
                            </span>
                        </div>
                        <img t-if="widget.sponsorData.country_flag_url"
                            class="img ml-auto"
                            style="max-height: 36px;"
                            t-att-src="widget.sponsorData.country_flag_url"
                            t-att-alt="widget.sponsorData.country_name"/>
                    </div>
                </div>
                <span class="col-12" t-if="widget.sponsorData.website_description" t-raw="widget.sponsorData.website_description"/>
            </div>
            <div>
            </div>
        </div>
    </t>

</templates>

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="assets_frontend" inherit_id="website.assets_frontend" name="Event Exhibitor Assets Frontend">
    <xpath expr="//link[last()]" position="after">
        <link rel="stylesheet" type="text/scss" href="/website_event_track_exhibitor/static/src/scss/event_exhibitor_templates.scss"/>
    </xpath>
    <xpath expr="//script[last()]" position="after">
        <script type="text/javascript" src="/website_event_track_exhibitor/static/src/js/event_exhibitor_connect.js"></script>
    </xpath>
</template>

</odoo>

```

## File: views\event_event_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_event_view_form" model="ir.ui.view">
        <field name="name">event.event.view.form.inherit.exhibitor</field>
        <field name="inherit_id" ref="website_event_track.event_event_view_form"/>
        <field name="model">event.event</field>
        <field name="priority" eval="3"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='website_track_proposal']" position="after">
                <label for="exhibitor_menu"/>
                <field name="exhibitor_menu"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\event_exhibitor_templates_list.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="event_exhibitors" name="Event Exhibitors">
    <t t-call="website_event.layout">
        <div class="o_wevent_online o_wevent_online_bg o_wesponsor_index">
            <!-- Topbar -->
            <t t-call="website_event_track_exhibitor.exhibitors_topbar"/>
            <!-- Drag/Drop Area -->
            <div id="oe_structure_wesponsor_index_1" class="oe_structure"/>
            <!-- Content -->
            <div class="o_wesponsor_container container">
                <div class="row">
                    <t t-call="website_event_track_exhibitor.exhibitors_search"/>
                </div>
                <div class="row">
                    <t t-call="website_event_track_exhibitor.exhibitors_main"/>
                </div>
            </div>
            <!-- Drag/Drop Area -->
            <div id="oe_structure_wesponsor_index_2" class="oe_structure mb-5"/>
        </div>
    </t>
</template>

<!-- ============================================================ -->
<!-- TOPBAR: BASE NAVIGATION -->
<!-- ============================================================ -->

<!-- TOPBAR: BASE NAVIGATION -->

<!-- Main topbar -->
<template id="exhibitors_topbar"
    name="Exhibitor Tools">
    <nav class="navbar navbar-light border-top shadow-sm d-print-none">
        <div class="container">
            <div class="d-flex flex-column flex-sm-row justify-content-between w-100">
                <ul class="o_wesponsor_topbar_filters o_wevent_index_topbar_filters nav"/>
                <div class="d-flex align-items-center flex-wrap pl-sm-3 pr-0">
                    <t t-call="website_event.events_search_box">
                        <t t-set="_searches" t-value="searches"/>
                        <t t-set="action" t-value="'/event/%s/exhibitors' % (slug(event))"/>
                        <t t-set="_placeholder" t-value="'Search an exhibitor ...'"/>
                    </t>
                </div>
            </div>
        </div>
    </nav>
</template>

<!-- Topbar: optional country filters -->
<template id="exhibitors_topbar_country"
    inherit_id="website_event_track_exhibitor.exhibitors_topbar"
    name="Filter by Country"
    active="True"
    customize_show="True">
    <xpath expr="//ul[hasclass('o_wesponsor_topbar_filters')]" position="inside">
        <li class="nav-item dropdown mr-2 my-1">
            <a href="#" role="button" class="btn dropdown-toggle" data-toggle="dropdown">
                <i class="fa fa-folder-open"/>
                By Country
            </a>
            <div class="dropdown-menu">
                <a t-att-href="'/event/%s/exhibitors?%s' % (slug(event), keep_query('*', countries=''))"
                    t-attf-class="dropdown-item d-flex align-items-center justify-content-between #{'active' if not search_countries else ''}">
                    All Countries
                </a>
                <t t-foreach="sponsor_countries" t-as="sponsor_country">
                    <a t-att-href="'/event/%s/exhibitors?%s' % (
                            slug(event),
                            keep_query('*', countries=str((search_countries - sponsor_country).ids if sponsor_country in search_countries else (sponsor_country | search_countries).ids))
                        )"
                        t-attf-class="dropdown-item d-flex align-items-center justify-content-between #{'active' if sponsor_country in search_countries else ''}">
                        <t t-esc="sponsor_country.name"/>
                    </a>
                </t>
            </div>
        </li>
    </xpath>
</template>

<!-- Topbar: optional sponsorship filters -->
<template id="exhibitors_topbar_sponsorship"
    inherit_id="website_event_track_exhibitor.exhibitors_topbar"
    name="Filter by Sponsorship"
    active="True"
    customize_show="True">
    <xpath expr="//ul[hasclass('o_wesponsor_topbar_filters')]" position="inside">
        <li class="nav-item dropdown mr-2 my-1">
            <a href="#" role="button" class="btn dropdown-toggle" data-toggle="dropdown">
                <i class="fa fa-folder-open"/>
                By Sponsorship
            </a>
            <div class="dropdown-menu">
                <a t-att-href="'/event/%s/exhibitors?%s' % (slug(event), keep_query('*', sponsorships=''))"
                    t-attf-class="dropdown-item d-flex align-items-center justify-content-between #{'active' if not search_sponsorships else ''}">
                    All Sponsorships
                </a>
                <t t-foreach="sponsor_types" t-as="sponsor_type">
                    <a t-att-href="'/event/%s/exhibitors?%s' % (
                            slug(event),
                            keep_query('*', sponsorships=str((search_sponsorships - sponsor_type).ids if sponsor_type in search_sponsorships else (sponsor_type | search_sponsorships).ids))
                        )"
                        t-attf-class="dropdown-item d-flex align-items-center justify-content-between #{'active' if sponsor_type in search_sponsorships else ''}">
                        <t t-esc="sponsor_type.name"/>
                    </a>
                </t>
            </div>
        </li>
    </xpath>
</template>

<!-- ============================================================ -->
<!-- CONTENT: MAIN TEMPLATES -->
<!-- ============================================================ -->

<!-- Exhibitors Main Display -->
<template id="exhibitors_main" name="Exhibitors: Main Display">
    <!-- No exhibitors -->
    <t t-if="not sponsor_categories">
        <div class="col-12">
            <div class="h2 mb-3">No exhibitor found.</div>
            <div t-if="search_key" class="alert alert-info text-center">
                <p class="m-0">We did not find any exhibitor matching your <strong t-esc="search_key"/> search.</p>
            </div>
            <div t-else="" class="alert alert-info text-center" groups="event.group_event_manager">
                <p class="m-0">Add some exhibitors to get started !</p>
            </div>
        </div>
    </t>
    <!-- Cards -->
    <div class="col-12" t-call="website_event_track_exhibitor.exhibitors_display_cards"/>
</template>

<!-- Exhibitors: Cards-based display -->
<template id="exhibitors_display_cards" name="Exhibitors Cards">
    <div t-foreach="sponsor_categories" t-as="sponsor_category" class="row mb-3">
        <div class="col-12">
            <h2 class="m-0" t-esc="sponsor_category['sponsorship'].name"/>
            <hr class="mt-2 pb-1 mb-1"/>
        </div>
        <div t-foreach="sponsor_category['sponsors']" t-as="sponsor" class="col-md-6 col-lg-3 mb-4">
            <t t-call="website_event_track_exhibitor.exhibitor_card"/>
        </div>
    </div>
</template>

<!-- ============================================================ -->
<!-- TOOL TEMPLATES -->
<!-- ============================================================ -->

<template id="exhibitor_card" name="Exhibitor Card">
    <article t-att-class="'h-100 card border-0 shadow-sm o_wesponsor_card %s' % ('o_wesponsor_card_unpublished' if not sponsor.is_published else '')"
        itemscope="itemscope" itemtype="http://schema.org/Event">
        <div class="h-100 row no-gutters">
            <t t-set="sponsor_image_url" t-value="sponsor.website_image_url"/>
            <header t-att-class="'overflow-hidden col-12 rounded-top position-static %s' % ('bg-secondary' if not sponsor_image_url else '')">

                <small t-if="not sponsor.is_published" class="o_wesponsor_card_header_badge bg-danger">
                    <i class="fa fa-ban mr-2"/>Unpublished
                </small>

                <div t-att-data-sponsor-url="sponsor.website_url" class="o_wesponsor_js_connect"
                     t-attf-data-register-url="/event/#{slug(event)}/register?from_sponsor_id=#{sponsor.id}"
                     t-att-data-is-participating="event.is_participating"
                     t-att-data-sponsor-id="sponsor.id"
                     t-att-data-event-is-ongoing="sponsor.event_id.is_ongoing"
                     t-att-data-sponsor-is-ongoing="sponsor.is_in_opening_hours"
                     t-att-data-user-event-manager="user_event_manager">
                    <div t-if="sponsor_image_url" class="card-img-top position-static o_wesponsor_bg_image"
                        t-attf-style="padding-top: 50%; background-image: url(#{sponsor_image_url});">
                        <div class="o_wesponsor_connect_button">
                            <a href="#" class="btn btn-primary h3">
                                <t t-if="not user_event_manager and not sponsor.event_id.is_ongoing and not event.is_participating">
                                    Register
                                </t>
                                <t t-else="">Connect</t>
                            </a>
                        </div>
                        <img class="position-absolute mr-2 mt-2"
                            style="right: 0; top: 0; max-height: 20px;"
                            t-if="sponsor.partner_id.country_id"
                            t-att-src="sponsor.partner_id.country_id.image_url"
                            t-att-alt="sponsor.partner_id.country_id.name"/>
                    </div>
                    <div t-else="" class="o_wesponsor_gradient card-img-top position-relative"
                        style="padding-top: 50%">
                        <div class="o_wesponsor_connect_button">
                            <a href="#" class="btn btn-primary h3">
                                <t t-if="not user_event_manager and not sponsor.event_id.is_ongoing and not event.is_participating">
                                    Register
                                </t>
                                <t t-else="">Connect</t>
                            </a>
                        </div>
                    </div>
                </div>
            </header>
            <div class="col-12">
                <main class="card-body h-100">
                    <!-- Title -->
                    <h5 class="card-title mt-0 mb-0 text-truncate d-flex align-items-top">
                        <span t-field="sponsor.name" itemprop="name"/>
                        <span t-if="sponsor.is_in_opening_hours and sponsor.chat_room_id"
                            class="ml-auto badge badge-danger">Live
                        </span>
                    </h5>
                    <!-- Catchy sentence -->
                    <span class="text-muted" t-esc="sponsor.subtitle"/>
                </main>
            </div>
        </div>
    </article>
</template>

<!-- Searched terms -->
<template id="exhibitors_search" name="Exhibitors: search terms">
    <div class="d-flex align-items-center mb-3">
        <t t-foreach="search_countries" t-as="country">
            <span class="align-items-baseline border d-inline-flex pl-2 mt-3 rounded ml16 mb-2 bg-white">
                <i class="fa fa-tag mr-2 text-muted"/>
                <t t-esc="country.display_name"/>
                <a t-att-href="'/event/%s/exhibitors?%s' % (
                    slug(event),
                    keep_query('*', countries=str((search_countries - country).ids)))"
                    class="btn border-0 py-1">&#215;</a>
            </span>
        </t>
        <t t-foreach="search_sponsorships" t-as="sponsorship">
            <span class="align-items-baseline border d-inline-flex pl-2 mt-3 rounded ml16 mb-2 bg-white">
                <i class="fa fa-tag mr-2 text-muted"/>
                <t t-esc="sponsorship.display_name"/>
                <a t-att-href="'/event/%s/exhibitors?%s' % (
                    slug(event),
                    keep_query('*', sponsorships=str((search_sponsorships - sponsorship).ids)))"
                    class="btn border-0 py-1">&#215;</a>
            </span>
        </t>
    </div>
</template>

</odoo>

```

## File: views\event_exhibitor_templates_page.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="event_exhibitor_main" name="Event Exhibitor">
    <t t-set="no_header" t-value="option_widescreen"/>
    <t t-set="no_footer" t-value="option_widescreen"/>
    <t t-call="website_event.layout">
        <div class="o_wevent_online o_wevent_online_bg o_wesponsor_index">
            <!-- Options -->
            <t t-set="option_widescreen" t-value="option_widescreen or False"/>
            <!-- Drag/Drop Area -->
            <div id="oe_structure_wesponsor_index_1" class="oe_structure"/>
            <!-- Content -->
            <div t-att-class="'o_wevent_online_page_container %s' % ('container pb-3' if not option_widescreen else 'pb-3')">
                <div class="row mb-5 mx-0">
                    <t t-call="website_event_track_exhibitor.exhibitor_aside"/>
                    <t t-call="website_event_track_exhibitor.exhibitor_main"/>
                </div>
            </div>
            <!-- Drag/Drop Area -->
            <div id="oe_structure_wesponsor_index_2" class="oe_structure"/>
        </div>
    </t>
</template>

<!-- ============================================================ -->
<!-- CONTENT: MAIN TEMPLATES -->
<!-- ============================================================ -->

<template id="exhibitor_main" name="Exhibitor: Main Content">
    <div t-att-class="'col-12 o_wevent_online_page_main o_wesponsor_exhibitor_main bg-white p-0 %s' % ('col-md-9 col-lg-10' if option_widescreen else 'col-md-8 col-lg-9')">
        <!-- EVENT NOT STARTED ALERTS -->
        <t t-if="not sponsor.event_id.is_ongoing">
            <div t-if="sponsor.event_id.is_done" class="alert alert-warning rounded-0 text-center" role="alert">
                Event <span t-esc="sponsor.event_id.name" class="font-weight-bold"/> is over.
                <br/>
                <span>Join us next time to meet <b t-esc="sponsor.partner_name"/>!</span>
            </div>
            <div t-else="" class="alert alert-warning rounded-0 text-center" role="alert">
                Event <span t-esc="sponsor.event_id.name" class="font-weight-bold"/>
                <span t-if="sponsor.event_id.start_today">
                    starts in
                    <span t-esc="sponsor.event_id.start_remaining"
                        t-options="{'widget': 'duration', 'digital': True, 'unit': 'minute', 'round': 'minute'}"/>.
                </span>
                <span class="my-0" t-else="">
                    starts on
                    <span t-field="sponsor.event_id.with_context(tz=sponsor.event_id.date_tz).date_begin"
                        t-options="{'format': 'medium'}"/> (<t t-esc="sponsor.event_id.date_tz"/>).
                </span>
                <br/>
                <span>Join us there to meet <b t-esc="sponsor.partner_name"/> !</span>
            </div>
        </t>
        <!-- SPONSOR JITSI + CLOSED/FULL ALERTS -->
        <div t-if="sponsor.event_id.is_ongoing and sponsor.chat_room_id" class="d-flex flex-column">
            <t t-if="not sponsor.is_in_opening_hours">
                <div class="col-12 alert alert-warning rounded-0 text-center" role="alert">
                    <span>Oops! This room is currently closed</span><br />
                    Come back between
                    <strong>
                        <t t-esc="sponsor.hour_from" t-options="{'widget': 'float_time'}"/>
                        -
                        <t t-esc="sponsor.hour_to" t-options="{'widget': 'float_time'}"/>
                    </strong> (<span t-esc="sponsor.event_date_tz"/>)
                    to meet them !
                </div>
            </t>
            <t t-elif="sponsor.room_is_full">
                <div class="col-12 alert alert-warning rounded-0 text-center" role="alert">
                    <span>Oops! This room is full</span><br />Come back later to have a chat with us!
                </div>
            </t>
            <t t-else="">
                <div id="o_wsponsor_jitsi_iframe"/>
                <div class="d-flex flex-row-reverse">
                    <t t-call="website_jitsi.chat_room_join_button">
                        <t t-set="_classes" t-value="'d-none'"/>
                        <t t-set="room_name" t-value="sponsor.room_name"/>
                        <t t-set="chat_room_id" t-value="sponsor.chat_room_id.id"/>
                        <t t-set="auto_open" t-value="1"/>
                        <t t-set="attach_to" t-value="'#o_wsponsor_jitsi_iframe'"/>
                        <t t-set="max_capacity" t-value="sponsor.room_max_capacity"/>
                        <t t-set="check_full" t-value="int(not option_can_edit)"/>
                        <t t-set="jitsi_server_domain" t-value="sponsor.chat_room_id.jitsi_server_domain"/>
                    </t>
                </div>
            </t>
        </div>
        <!-- SPONSOR DESCRIPTION -->
        <span class="h5 d-inline-block mx-3 mt-3 mb-0">About <t t-esc="sponsor.name"/></span>
        <hr class="mt-2 mb-0"/>
        <div class="ml-3">
            <div class="float-left pt-3">
                <span t-if="sponsor.image_512" t-field="sponsor.image_512" class="o_wevent_online_page_avatar"
                    t-options="{'widget': 'image', 'max-width': '96'}"/>
                <span t-elif="sponsor.partner_id.image_512" t-field="sponsor.partner_id.image_512" class="o_wevent_online_page_avatar"
                    t-options="{'widget': 'image', 'max-width': '96'}"/>
            </div>
            <div class="o_wevent_sponsor px-3 pt-3 d-flex flex-row justify-content-between position-relative">
                <div class="d-flex flex-column">
                    <div class="d-flex align-items-center">
                        <span t-field="sponsor.name" class="h4 mb-0"/>
                        <span t-if="sponsor.sponsor_type_id.display_ribbon_style and sponsor.sponsor_type_id.display_ribbon_style != 'no_ribbon'"
                              t-field="sponsor.sponsor_type_id" t-attf-class="o_ribbon o_ribbon_right ribbon_#{sponsor.sponsor_type_id.display_ribbon_style}"/>
                    </div>
                    <span t-field="sponsor.subtitle" class="text-muted mb-3"/>
                    <div t-if="sponsor.url" class="d-flex text-break align-items-baseline">
                        <i class="fa fa-home mr-2"/><a t-att-href="sponsor.url"><span t-field="sponsor.url"/></a>
                    </div>
                    <div t-elif="sponsor.partner_id.website" class="d-flex text-break align-items-baseline">
                        <i class="fa fa-home mr-2"/><a t-att-href="sponsor.partner_id.website"><span t-field="sponsor.partner_id.website"/></a>
                    </div>
                    <div t-if="sponsor.email" class="d-flex text-break align-items-baseline">
                        <i class="fa fa-envelope mr-2"/><a t-att-mailto="sponsor.email"><span t-field="sponsor.email"/></a>
                    </div>
                    <div t-if="sponsor.phone" class="d-flex text-break align-items-baseline">
                        <i class="fa fa-phone mr-2"/><span t-field="sponsor.phone"/>
                    </div>
                </div>
                <a t-if="sponsor.partner_id.country_id"
                    t-att-href="'/event/%s/exhibitors?countries=%s' % (slug(sponsor.event_id), [sponsor.partner_id.country_id.id])"
                    t-attf-class="text-right d-none d-md-block #{'mr-5' if sponsor.sponsor_type_id.display_ribbon_style and sponsor.sponsor_type_id.display_ribbon_style != 'no_ribbon' else ''}">
                    <img class="img"
                        style="max-height: 36px;"
                        t-att-src="sponsor.partner_id.country_id.image_url"
                        t-att-alt="sponsor.partner_id.country_id.name"/>
                </a>
            </div>
        </div>
        <div t-field="sponsor.website_description" class="my-2 mx-3 oe_no_empty"/>
    </div>
</template>

<!-- ============================================================ -->
<!-- ASIDE: CONTROL PANEL -->
<!-- ============================================================ -->

<template id="exhibitor_aside" name="Exhibitor: Aside">
    <div t-att-class="'col-12 pl-0 pr-0 pr-md-3 o_wevent_online_page_aside o_wesponsor_exhibitor_aside %s' % ('col-md-3 col-lg-2' if option_widescreen else 'col-md-4 col-lg-3')">
        <div class="bg-white o_wevent_online_page_aside_content">
            <div class="d-flex align-items-center justify-content-between mx-2">
                <span class="h5 mb-0 pt-0 pt-md-3 pb-0 pb-md-2">Other exhibitors</span>
                <a href="#collapse_exhibitor_aside" data-toggle="collapse"
                   class="d-lg-none p-2 text-decoration-none o_wevent_online_page_aside_collapse collapsed">
                    <i class="fa fa-chevron-down d-lg-none"/>
                </a>
            </div>
            <ul id="collapse_exhibitor_aside" class="list-unstyled collapse d-lg-block mb-0">
                <li t-foreach="sponsors_other" t-as="sponsor_other">
                    <a class="d-flex w-100 h-100 px-2 pt-2 pb-1 text-decoration-none"
                        t-att-href="sponsor_other.website_url">
                        <div class="d-flex flex-column align-items-center">
                            <img t-if="sponsor_other.partner_id.country_id"
                            class="mr-2 mb-1 o_wesponsor_aside_logo"
                            t-att-src="sponsor_other.partner_id.country_id.image_url"
                            t-att-alt="sponsor_other.partner_id.country_id.name"/>
                            <span t-if="sponsor_other.sponsor_type_id.display_ribbon_style not in [False, 'no_ribbon']"
                                t-att-class="'badge badge-light mr-2 ribbon_%s' % sponsor_other.sponsor_type_id.display_ribbon_style"
                                t-esc="sponsor_other.sponsor_type_id.name"/>
                            <span t-else="" class="badge badge-light mr-2"
                                t-esc="sponsor_other.sponsor_type_id.name"/>
                        </div>
                        <div class="flex-grow-1 overflow-auto">
                            <span class="d-flex align-items-baseline o_wesponsor_sponsor_name">
                                <span class="d-inline-block text-truncate" t-esc="sponsor_other.name"/>

                            </span>

                            <small class="text-muted" t-esc="sponsor_other.subtitle"/>
                        </div>
                    </a>
                </li>
            </ul>
        </div>
    </div>
</template>

<!-- ============================================================ -->
<!-- MISC / MENUS -->
<!-- ============================================================ -->

<!-- User Navbar - Edit Options -->
<template id="event_sponsor_edit_options" inherit_id="website.user_navbar" name="Edit Event Sponsor Options">
    <xpath expr="//li[@id='edit-page-menu']" position="after">
        <t t-if="main_object._name == 'event.sponsor'"
            t-set="action" t-value="'website_event_track_exhibitor.event_sponsor_action'"/>
    </xpath>
</template>

</odoo>

```

## File: views\event_sponsor_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="event_sponsor_view_search" model="ir.ui.view">
        <field name="name">event.sponsor.view.search.inherit.exhibitor</field>
        <field name="model">event.sponsor</field>
        <field name="inherit_id" ref="website_event_track.event_sponsor_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='archived']" position="after">
                <separator/>
                <filter string="Exhibitor" name="filter_is_exhibitor" domain="[('is_exhibitor', '=', True)]"/>
            </xpath>
        </field>
    </record>

    <record id="event_sponsor_view_tree" model="ir.ui.view">
        <field name="name">event.sponsor.view.tree.inherit.exhibitor</field>
        <field name="model">event.sponsor</field>
        <field name="inherit_id" ref="website_event_track.event_sponsor_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='sponsor_type_id']" position="after">
                <field name="is_published" optional="show"/>
                <field name="is_exhibitor"/>
            </xpath>
        </field>
    </record>

    <record id="event_sponsor_view_kanban" model="ir.ui.view">
        <field name="name">event.sponsor.view.kanban.inherit.kanban</field>
        <field name="model">event.sponsor</field>
        <field name="inherit_id" ref="website_event_track.event_sponsor_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="replace">
                <field name="name"/>
            </xpath>
        </field>
    </record>

    <record id="event_sponsor_view_form" model="ir.ui.view">
        <field name="name">event.sponsor.view.form.inherit.exhibitor</field>
        <field name="model">event.sponsor</field>
        <field name="inherit_id" ref="website_event_track.event_sponsor_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//widget[@name='web_ribbon']" position="before">
                <div class="oe_button_box" name="button_box">
                    <field name="website_url" invisible="1"/>
                    <field name="is_published" widget="website_redirect_button"/>
                </div>
            </xpath>
            <xpath expr="//h1" position="after">
                <div class="oe_title">
                    <label for="subtitle" class="oe_edit_only"/>
                    <field name="subtitle" placeholder="e.g. : Your best choice for your home"/>
                </div>
            </xpath>
            <xpath expr="//field[@name='sponsor_type_id']" position="after">
                <field name="is_exhibitor"/>
            </xpath>
            <xpath expr="//field[@name='mobile']" position="after">
                <label for="hour_from" string="Opening Hours"/>
                <div class="o_row">
                    <field name="hour_from" widget="float_time" nolabel="1" class="oe_inline"/>
                    <i class="fa fa-long-arrow-right mx-2" aria-label="Arrow icon" title="Arrow"/>
                    <field name="hour_to" widget="float_time" nolabel="1" class="oe_inline"/>
                    <field name="event_date_tz" nolabel="1" class="oe_inline"/>
                </div>
            </xpath>
            <xpath expr="//sheet/group" position="after">
                <notebook>
                    <page string="Description">
                        <group>
                            <field name="website_description" nolabel="1"/>
                        </group>
                    </page>
                    <page string="Exhibitor" attrs="{'invisible': [('is_exhibitor', '=', False)]}">
                        <group>
                            <group>
                                <field name="room_name" attrs="{'required': [('is_exhibitor', '=', True)]}" string="Jitsi Name"/>
                                <field name="room_lang_id"/>
                                <field name="room_max_capacity" attrs="{'required': [('is_exhibitor', '=', True)]}"/>
                                <field name="chat_room_id" groups="base.group_no_one"/>
                            </group>
                        </group>
                    </page>
                </notebook>
            </xpath>
        </field>
    </record>

    <record id="event_sponsor_action" model="ir.actions.act_window">
        <field name="name">Event Sponsors</field>
        <field name="res_model">event.sponsor</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="help" type="html">
<p class="o_view_nocontent_smiling_face">
    Create a Sponsor / Exhibitor
</p><p>
    Sponsors might be advertised on your event pages footer.<br />
    Exhibitors might have a dedicated page with chat room for people to connect with them.
</p>
        </field>
    </record>

    <record id="website_event_track.event_sponsor_action_from_event" model="ir.actions.act_window">
        <field name="help" type="html">
<p class="o_view_nocontent_smiling_face">
    Create a Sponsor / Exhibitor
</p><p>
    Sponsors might be advertised on your event pages footer.<br />
    Exhibitors might have a dedicated page with chat room for people to connect with them.
</p>
        </field>
    </record>

</odoo>

```

## File: views\event_type_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_type_view_form" model="ir.ui.view">
        <field name="name">event.type.view.form.inherit.exhibitor</field>
        <field name="model">event.type</field>
        <field name="inherit_id" ref="website_event.event_type_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='website_menu']" position='after'>
                <div class="row mt16" attrs="{'invisible': [('website_menu', '=', False)]}">
                    <label class="col-lg-4" for="exhibitor_menu"/> <field name="exhibitor_menu"/>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

