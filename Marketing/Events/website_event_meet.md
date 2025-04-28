# Odoo Module: website_event_meet

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
    'name': 'Event Meeting / Rooms',
    'category': 'Marketing/Events',
    'sequence': 1002,
    'version': '1.0',
    'summary': 'Event: meeting and chat rooms',
    'website': 'https://www.odoo.com/app/events',
    'depends': [
        'website_event_jitsi',
    ],
    'demo': ['data/website_event_meet_demo.xml'],
    'data': [
        'security/ir.model.access.csv',
        'security/security.xml',
        'views/event_meet_templates_list.xml',
        'views/event_meet_templates_page.xml',
        'views/event_meeting_room_views.xml',
        'views/event_event_views.xml',
        'views/event_type_views.xml',
        'views/snippets.xml',
    ],
    'installable': True,
    'assets': {
        'web.assets_frontend': [
            'website_event_meet/static/src/scss/event_meet_templates.scss',
            'website_event_meet/static/src/js/website_event_meeting_room.js',
            'website_event_meet/static/src/js/website_event_create_meeting_room_button.js',
            'website_event_meet/static/src/xml/website_event_meeting_room.xml',
        ],
        'website.assets_wysiwyg': [
            'website_event_meet/static/src/js/snippets/options.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\community.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from werkzeug.exceptions import Forbidden, NotFound

from odoo import exceptions, http
from odoo.http import request
from odoo.addons.http_routing.models.ir_http import slug
from odoo.addons.website_event.controllers.community import EventCommunityController
from odoo.osv import expression

_logger = logging.getLogger(__name__)


class WebsiteEventMeetController(EventCommunityController):

    def _get_event_rooms_base_domain(self, event):
        search_domain_base = [('event_id', '=', event.id)]
        if not request.env.user.has_group('event.group_event_registration_desk'):
            search_domain_base = expression.AND([search_domain_base, [('is_published', '=', True)]])
        return search_domain_base

    def _sort_event_rooms(self, room):
        return (room.website_published, room.is_pinned, room.room_last_activity, room.id)

    # ------------------------------------------------------------
    # MAIN PAGE
    # ------------------------------------------------------------

    @http.route(["/event/<model('event.event'):event>/community"], type="http",
                auth="public", website=True, sitemap=True)
    def community(self, event, page=1, lang=None, **kwargs):
        """Display the meeting rooms of the event on the frontend side.

        :param event: event for which we display the meeting rooms
        :param lang: lang id used to perform a search
        """
        return request.render(
            "website_event_meet.event_meet",
            self._event_meeting_rooms_get_values(event, lang=lang)
        )

    def _event_meeting_rooms_get_values(self, event, lang=None):
        search_domain = self._get_event_rooms_base_domain(event)
        meeting_rooms_all = request.env['event.meeting.room'].sudo().search(search_domain)
        if lang:
            search_domain = expression.AND([
                search_domain,
                [('room_lang_id', '=', int(lang))]
            ])
        meeting_rooms = request.env['event.meeting.room'].sudo().search(search_domain)
        meeting_rooms = meeting_rooms.sorted(self._sort_event_rooms, reverse=True)

        is_event_user = request.env.user.has_group("event.group_event_registration_desk")
        if not is_event_user:
            meeting_rooms = meeting_rooms.filtered(lambda m: not m.room_is_full)

        visitor = request.env['website.visitor']._get_visitor_from_request()

        return {
            # event information
            "event": event,
            'main_object': event,
            # rooms
            "meeting_rooms": meeting_rooms,
            "current_lang": request.env["res.lang"].browse(int(lang)) if lang else False,
            "available_languages": meeting_rooms_all.mapped("room_lang_id"),
            "default_lang_code": request.context.get('lang', request.env.user.lang),
            "default_username": visitor.display_name if visitor else None,
            # environment
            "is_event_user": is_event_user,
        }

    @http.route("/event/<model('event.event'):event>/meeting_room_create",
                type="http", auth="public", methods=["POST"], website=True)
    def create_meeting_room(self, event, **post):
        if not event or (not event.is_published and not request.env.user.user_has_groups('base.group_user')) or not event.meeting_room_allow_creation:
            raise Forbidden()

        name = post.get("name")
        summary = post.get("summary")
        target_audience = post.get("audience")
        lang_code = post.get("lang_code")
        max_capacity = post.get("capacity")

        # get the record to be sure they really exist
        lang = request.env["res.lang"].search([("code", "=", lang_code)], limit=1)

        if not lang or max_capacity == "no_limit":
            raise Forbidden()

        meeting_room = request.env["event.meeting.room"].sudo().create({
            "name": name,
            "summary": summary,
            "target_audience": target_audience,
            "is_pinned": False,
            "event_id": event.id,
            "room_lang_id": lang.id,
            "room_max_capacity": max_capacity,
            "is_published": True,
        })
        _logger.info("New meeting room (%s) created by %s (uid %s)" % (name, request.httprequest.remote_addr, request.env.uid))

        return request.redirect(f"/event/{slug(event)}/meeting_room/{slug(meeting_room)}")

    @http.route(["/event/active_langs"], type="json", auth="public")
    def active_langs(self):
        return request.env["res.lang"].sudo().get_installed()

    # ------------------------------------------------------------
    # ROOM PAGE VIEW
    # ------------------------------------------------------------

    @http.route('''/event/<model('event.event', "[('community_menu', '=', True)]"):event>/meeting_room/<model("event.meeting.room","[('event_id','=',event.id)]"):meeting_room>''',
                type="http", auth="public", website=True, sitemap=True)
    def event_meeting_room_page(self, event, meeting_room, **post):
        """Display the meeting room frontend view.

        :param event: Event for which we display the meeting rooms
        :param meeting_room: Meeting Room to display
        """
        if meeting_room not in event.sudo().meeting_room_ids:
            raise NotFound()

        try:
            meeting_room.check_access_rule('read')
        except exceptions.AccessError:
            raise Forbidden()

        meeting_room = meeting_room.sudo()

        return request.render(
            "website_event_meet.event_meet_main",
            self._event_meeting_room_page_get_values(event, meeting_room),
        )

    def _event_meeting_room_page_get_values(self, event, meeting_room):
        # search for meeting room list. Set a limit to 6 because it is better than 5 or 7
        meeting_rooms_other = request.env['event.meeting.room'].sudo().search([
            ('event_id', '=', event.id), ('id', '!=', meeting_room.id), ('is_published', '=', True),
        ], limit=6)

        if not request.env.user.has_group("event.group_event_manager"):
            # only the event manager can see meeting rooms which are full
            meeting_rooms_other = meeting_rooms_other.filtered(lambda m: not m.room_is_full)

        meeting_rooms_other = meeting_rooms_other.sorted(self._sort_event_rooms, reverse=True)

        return {
            # event information
            'event': event,
            'main_object': meeting_room,
            'meeting_room': meeting_room,
            # sidebar
            'meeting_rooms_other': meeting_rooms_other,
            # options
            'option_widescreen': True,
            'is_event_user': request.env.user.has_group('event.group_event_registration_desk'),
        }

```

## File: controllers\website_event_main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from babel.dates import format_datetime

from odoo import _
from odoo.http import request
from odoo.addons.website_event.controllers import main


class WebsiteEventController(main.WebsiteEventController):
    def _prepare_event_register_values(self, event, **post):
        values = super(WebsiteEventController, self)._prepare_event_register_values(event, **post)

        if "from_room_id" in post and not event.is_ongoing:
            meeting_room = request.env["event.meeting.room"].browse(int(post["from_room_id"])).sudo().exists()
            if meeting_room and meeting_room.is_published:
                date_begin = format_datetime(event.date_begin, format="medium", tzinfo=event.date_tz)

                values["toast_message"] = (
                    _('The event %s starts on %s (%s). \nJoin us there to chat about "%s" !')
                    % (event.name, date_begin, event.date_tz, meeting_room.name)
                )

        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import community
from . import website_event_main

```

## File: data\website_event_meet_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="event_meeting_room_0" model="event.meeting.room">
            <field name="name">Best wood for furniture</field>
            <field name="summary">Let's talk about wood types for furniture</field>
            <field name="target_audience">wood expert(s)</field>
            <field name="is_pinned" eval="True"/>
            <field name="website_published" eval="True"/>
            <field name="event_id" ref="event.event_7"/>
            <field name="room_lang_id" ref="base.lang_en"/>
            <field name="room_max_capacity">12</field>
            <field name="room_participant_count">9</field>
        </record>
        <record id="event_meeting_room_1" model="event.meeting.room">
            <field name="name">Reducing the ecological footprint with wood ?</field>
            <field name="summary">Share your tips to reduce your ecological footprint using wood.</field>
            <field name="target_audience">ecologist(s)</field>
            <field name="is_pinned" eval="True"/>
            <field name="website_published" eval="True"/>
            <field name="event_id" ref="event.event_7"/>
            <field name="room_lang_id" ref="base.lang_en"/>
            <field name="room_max_capacity">8</field>
            <field name="room_participant_count">8</field>
        </record>
        <record id="event_meeting_room_2" model="event.meeting.room">
            <field name="name">Vos meubles préférés ?</field>
            <field name="summary">Venez partager vos meubles préférés et l'utilisation que vous en faites.</field>
            <field name="target_audience">client(s)</field>
            <field name="is_pinned" eval="True"/>
            <field name="website_published" eval="True"/>
            <field name="event_id" ref="event.event_7"/>
            <field name="room_lang_id" ref="base.lang_fr"/>
            <field name="room_max_capacity">8</field>
            <field name="room_participant_count">3</field>
        </record>
        <record id="event.event_7" model="event.event">
            <field name="meeting_room_allow_creation" eval="True"/>
            <field name="community_menu" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: models\event_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Event(models.Model):
    _inherit = "event.event"

    meeting_room_ids = fields.One2many("event.meeting.room", "event_id", string="Meeting rooms")
    meeting_room_count = fields.Integer("Room count", compute="_compute_meeting_room_count")
    meeting_room_allow_creation = fields.Boolean(
        "Allow Room Creation", compute="_compute_meeting_room_allow_creation",
        readonly=False, store=True,
        help="Let Visitors Create Rooms")

    @api.depends("event_type_id", "website_menu", "community_menu")
    def _compute_community_menu(self):
        """ At type onchange: synchronize. At website_menu update: synchronize. """
        for event in self:
            if event.event_type_id and event.event_type_id != event._origin.event_type_id:
                event.community_menu = event.event_type_id.community_menu
            elif event.website_menu and (event.website_menu != event._origin.website_menu or not event.community_menu):
                event.community_menu = True
            elif not event.website_menu:
                event.community_menu = False

    @api.depends("meeting_room_ids")
    def _compute_meeting_room_count(self):
        meeting_room_count = self.env["event.meeting.room"].sudo()._read_group(
            domain=[("event_id", "in", self.ids)],
            fields=["id:count"],
            groupby=["event_id"],
        )

        meeting_room_count = {
            result["event_id"][0]: result["event_id_count"]
            for result in meeting_room_count
        }

        for event in self:
            event.meeting_room_count = meeting_room_count.get(event.id, 0)

    @api.depends("event_type_id", "community_menu", "meeting_room_allow_creation")
    def _compute_meeting_room_allow_creation(self):
        for event in self:
            if event.event_type_id and event.event_type_id != event._origin.event_type_id:
                event.meeting_room_allow_creation = event.event_type_id.meeting_room_allow_creation
            elif event.community_menu and event.community_menu != event._origin.community_menu:
                event.meeting_room_allow_creation = True
            elif not event.community_menu or not event.meeting_room_allow_creation:
                event.meeting_room_allow_creation = False

```

## File: models\event_meeting_room.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime

from odoo import api, fields, models
from odoo.addons.http_routing.models.ir_http import slug


class EventMeetingRoom(models.Model):
    _name = "event.meeting.room"
    _description = "Event Meeting Room"
    _order = "is_pinned DESC, id"
    _inherit = [
        'chat.room.mixin',
        'website.published.mixin',
    ]

    _DELAY_CLEAN = datetime.timedelta(hours=4)

    name = fields.Char("Topic", required=True, translate=True)
    active = fields.Boolean('Active', default=True)
    is_published = fields.Boolean(copy=True)  # make the inherited field copyable
    event_id = fields.Many2one("event.event", string="Event", required=True, ondelete="cascade")
    is_pinned = fields.Boolean("Is Pinned")
    chat_room_id = fields.Many2one("chat.room", required=True, ondelete="restrict")
    room_max_capacity = fields.Selection(default="8", copy=True)
    summary = fields.Char("Summary", translate=True)
    target_audience = fields.Char("Audience", translate=True)

    @api.depends('name', 'event_id.name')
    def _compute_website_url(self):
        super(EventMeetingRoom, self)._compute_website_url()
        for meeting_room in self:
            if meeting_room.id:
                base_url = meeting_room.event_id.get_base_url()
                meeting_room.website_url = '%s/event/%s/meeting_room/%s' % (base_url, slug(meeting_room.event_id), slug(meeting_room))

    @api.model_create_multi
    def create(self, values_list):
        for values in values_list:
            if not values.get("chat_room_id") and not values.get('room_name'):
                values['room_name'] = 'odoo-room-%s' % (values['name'])
        return super(EventMeetingRoom, self).create(values_list)

    @api.autovacuum
    def _archive_meeting_rooms(self):
        """Archive all non-pinned room with 0 participant if nobody has joined it for a moment."""
        self.sudo().search([
            ("is_pinned", "=", False),
            ("active", "=", True),
            ("room_participant_count", "=", 0),
            ("room_last_activity", "<", fields.Datetime.now() - self._DELAY_CLEAN),
        ]).active = False

    def open_website_url(self):
        """ Overridden to use a relative URL instead of an absolute when website_id is False. """
        if self.event_id.website_id:
            return super().open_website_url()
        return self.env['website'].get_client_action(f'/event/{slug(self.event_id)}/meeting_room/{slug(self)}')

```

## File: models\event_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class EventType(models.Model):
    _inherit = "event.type"

    meeting_room_allow_creation = fields.Boolean(
        "Allow Room Creation", compute='_compute_meeting_room_allow_creation',
        readonly=False, store=True,
        help="Let Visitors Create Rooms")

    @api.depends('community_menu')
    def _compute_meeting_room_allow_creation(self):
        for event_type in self:
            event_type.meeting_room_allow_creation = event_type.community_menu

```

## File: models\website_event_menu.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class EventMenu(models.Model):
    _inherit = "website.event.menu"

    menu_type = fields.Selection(
        selection_add=[("meeting_room", "Event Meeting Room Menus")],
        ondelete={'meeting_room': 'cascade'})

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_event
from . import event_type
from . import event_meeting_room
from . import website_event_menu

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_event_meeting_room,event.meeting.room,model_event_meeting_room,,1,0,0,0
access_event_meeting_room_user,event.meeting.room.user,model_event_meeting_room,event.group_event_user,1,1,1,1
access_chat_room_registration,chat.room.registration,website_jitsi.model_chat_room,event.group_event_registration_desk,1,0,0,0
access_chat_room_user,chat.room.user,website_jitsi.model_chat_room,event.group_event_user,1,1,1,1

```

## File: security\security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="event_meeting_room_rule_share" model="ir.rule">
        <field name="name">Event Meeting Room: public/portal published only</field>
        <field name="model_id" ref="website_event_meet.model_event_meeting_room"/>
        <field name="domain_force">[('website_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

</odoo>

```

## File: static\src\js\website_event_create_meeting_room_button.js

```javascript
odoo.define('website_event_meet.website_event_create_room_button', function (require) {
'use strict';

const publicWidget = require('web.public.widget');
const core = require('web.core');
const QWeb = core.qweb;

publicWidget.registry.websiteEventCreateMeetingRoom = publicWidget.Widget.extend({
    selector: '.o_wevent_create_room_button',
    events: {
        'click': '_onClickCreate',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    _onClickCreate: async function () {
        if (!this.$createModal) {
            const langs = await this._rpc({
                route: "/event/active_langs",
            });

            this.$createModal = $(QWeb.render(
                'event_meet_create_room_modal',
                {
                    csrf_token: odoo.csrf_token,
                    eventId: this.$el.data("eventId"),
                    defaultLangCode: this.$el.data("defaultLangCode"),
                    langs: langs,
                }
            ));

            this.$createModal.appendTo(this.$el.parentNode);
        }

        this.$createModal.modal('show');
    },

    //--------------------------------------------------------------------------
    // Override
    //--------------------------------------------------------------------------

    /**
     * Remove the create modal from the DOM, to avoid issue when editing the template
     * with the website editor.
     *
     * @override
     */
    destroy: function () {
        $('.o_wevent_create_meeting_room_modal').remove();
        this._super.apply(this, arguments);
    },
});

return publicWidget.registry.websiteEventMeetingRoom;

});

```

## File: static\src\js\website_event_meeting_room.js

```javascript
odoo.define('website_event_meet.website_event_meet_meeting_room', function (require) {
'use strict';

const publicWidget = require('web.public.widget');
const core = require('web.core');
const Dialog = require('web.Dialog');
const _t = core._t;

publicWidget.registry.websiteEventMeetingRoom = publicWidget.Widget.extend({
    selector: '.o_wevent_meeting_room_card',
    events: {
        'click .o_wevent_meeting_room_delete': '_onDeleteClick',
        'click .o_wevent_meeting_room_duplicate': '_onDuplicateClick',
        'click .o_wevent_meeting_room_is_pinned': '_onPinClick',
    },

    start: function () {
        this._super.apply(this, arguments);
        this.meetingRoomId = parseInt(this.$el.data('meeting-room-id'));
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
      * Delete the meeting room.
      *
      * @private
      */
    _onDeleteClick: async function (event) {
        event.preventDefault();
        event.stopPropagation();

        Dialog.confirm(
            this,
            _t("Are you sure you want to close this room ?"),
            {
                confirm_callback: async () => {
                    await this._rpc({
                        model: 'event.meeting.room',
                        method: 'write',
                        args: [this.meetingRoomId, {is_published: false}],
                        context: this.context,
                    });

                    // remove the element so we do not need to refresh the page
                    this.$el.remove();
                }
            },
        );
    },

    /**
      * Duplicate the room.
      *
      * @private
      */
    _onDuplicateClick: function (event) {
        event.preventDefault();
        event.stopPropagation();
        Dialog.confirm(
            this,
            _t("Are you sure you want to duplicate this room ?"),
            {
                confirm_callback: async () => {
                    await this._rpc({
                        model: 'event.meeting.room',
                        method: 'copy',
                        args: [this.meetingRoomId],
                        context: this.context,
                    });

                    window.location.reload();
                }
            },
        );
    },

    /**
      * Pin/unpin the room.
      *
      * @private
      */
    _onPinClick: async function (event) {
        event.preventDefault();
        event.stopPropagation();

        const pinnedButtonClass = "o_wevent_meeting_room_pinned";
        const isPinned = event.currentTarget.classList.contains(pinnedButtonClass);

        await this._rpc({
            model: 'event.meeting.room',
            method: 'write',
            args: [this.meetingRoomId, {is_pinned: !isPinned}],
            context: this.context,
        });

        // TDE FIXME: addclass ?
        if (isPinned) {
            event.currentTarget.classList.remove(pinnedButtonClass);
        } else {
            event.currentTarget.classList.add(pinnedButtonClass);
        }
    }
});

return publicWidget.registry.websiteEventMeetingRoom;

});

```

## File: static\src\js\snippets\options.js

```javascript
/** @odoo-module **/

import options from 'web_editor.snippets.options';

options.registry.WebsiteEvent.include({

    /**
     * @override
     */
    async start() {
        const res = await this._super(...arguments);
        const rpcData = await this._rpc({
            model: 'event.event',
            method: 'read',
            args: [
                [this.eventId],
                ['meeting_room_allow_creation'],
            ],
        });
        this.meetingRoomAllowCreation = rpcData[0]['meeting_room_allow_creation'];
        return res;
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * @see this.selectClass for parameters
     */
    allowRoomCreation(previewMode, widgetValue, params) {
        return this._rpc({
            model: 'event.event',
            method: 'write',
            args: [[this.eventId], {
                meeting_room_allow_creation: widgetValue
            }],
        // TODO: Remove the request_save in master, it's already done by the
        // data-page-options set to true in the template.
        }).then(() => this.trigger_up('request_save', {reload: true, optionSelector: this.data.selector}));
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    async _computeWidgetState(methodName, params) {
        switch (methodName) {
            case 'allowRoomCreation': {
                return this.meetingRoomAllowCreation;
            }
        }
        return this._super(...arguments);
    },
});

```

## File: static\src\xml\website_event_meeting_room.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-name="event_meet_create_room_modal">
        <div class="modal fade o_wevent_create_meeting_room_modal" role="dialog">
            <div class="modal-dialog">
                <div class="modal-content border-0">
                    <main class="modal-body p-3">
                        <form t-attf-action="/event/#{eventId}/meeting_room_create"
                            id="o_wevent_create_meeting_room_form"
                            class="text-dark"
                            method="POST">
                            <h2>Launch a new topic</h2>
                            <p>Be sure you are ready to spend at least 10 minutes in the room if you want to initiate a new topic.</p>
                            <div class="row m-2">
                                <label class="col-4 mt-2 text-start">Room Topic</label>
                                <input class="form-control col-8" maxlength="50" name="name" placeholder="e.g. Finance" required="1"/>
                            </div>
                            <div class="row m-2">
                                <label class="col-4 mt-2 text-start">Short Summary</label>
                                <input class="form-control col-8" maxlength="200" name="summary" placeholder="e.g. Let's talk about Corporate Finance" required="1"/>
                            </div>
                            <div class="row m-2">
                                <label class="col-4 mt-2 text-start">Target People</label>
                                <input class="form-control col-8" maxlength="30" name="audience" placeholder="e.g. Accountants"/>
                            </div>
                            <div class="row m-2">
                                <label class="col-4 mt-2 text-start">Language</label>
                                <select class="o_wevent_create_meeting_room_lang form-select col-8" name="lang_code" required="1">
                                    <option t-foreach="langs" t-as="language" t-att-value="language[0]" t-out="language[1]"
                                        t-att-selected="language[0] == defaultLangCode and 'selected' or None"/>
                                </select>
                            </div>
                            <div class="row m-2">
                                <label class="col-4 mt-2 text-start">Capacity</label>
                                <select class="form-select col-8" name="capacity" required="1">
                                    <option value="4">4</option>
                                    <option selected="selected" value="8">8</option>
                                    <option value="12">12</option>
                                    <option value="16">16</option>
                                    <option value="20">20</option>
                                </select>
                            </div>
                            <input name="event" t-att-value="event" type="hidden"/>
                            <input name="csrf_token" t-att-value="csrf_token" type="hidden"/>
                        </form>
                    </main>
                    <footer class="modal-footer justify-content-start">
                        <button class="btn btn-primary" form="o_wevent_create_meeting_room_form" type="submit">Create</button>
                        <button class="btn bg-white text-primary" role="button" data-bs-dismiss="modal">Discard</button>
                    </footer>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: views\event_event_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="event_event_view_form" model="ir.ui.view">
        <field name="name">event.event.view.form.inherit.meet</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="website_event.event_event_view_form"/>
        <field name="priority" eval="10"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='is_published']" position="before">
                <button class="oe_stat_button" context="{'default_event_id': active_id, 'search_default_event_id': active_id}" icon="fa-comments-o" name="%(event_meeting_room_action)d" type="action">
                    <field name="meeting_room_count" string="Rooms" widget="statinfo"/>
                </button>
            </xpath>
            <xpath expr="//label[@for='community_menu']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//field[@name='community_menu']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//field[@name='community_menu']" position="after">
                <field name="meeting_room_allow_creation" invisible="1"/>
            </xpath>
        </field>
    </record>

</data></odoo>


```

## File: views\event_meeting_room_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="website_event_meet.event_meeting_room_action" model="ir.actions.act_window">
        <field name="name">Meeting Room</field>
        <field name="res_model">event.meeting.room</field>
        <field name="view_mode">tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Room
            </p>
            <p>
                Rooms allow your event attendees to meet up and chat on different topics.
            </p>
        </field>
    </record>
    <record id="event_meeting_room_view_search" model="ir.ui.view">
        <field name="name">event.meeting.room.search</field>
        <field name="model">event.meeting.room</field>
        <field name="arch" type="xml">
            <search string="Meeting Room">
                <field name="event_id"/>
            </search>
        </field>
    </record>
    <record id="event_meeting_room_view_form" model="ir.ui.view">
        <field name="name">event.meeting.room.form</field>
        <field name="model">event.meeting.room</field>
        <field name="arch" type="xml">
            <form string="Meeting Room">
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <field name="website_url" invisible="1"/>
                        <field name="is_published" widget="website_redirect_button"/>
                    </div>
                    <label for="name"/>
                    <h1>
                        <field name="name" placeholder="e.g. Finance"/>
                    </h1>
                    <group>
                        <group>
                            <field name="event_id"/>
                            <field name="summary" placeholder="e.g. Let's talk about Corporate Finance"/>
                            <field name="target_audience" placeholder="e.g. Accountants"/>
                            <field name="is_pinned"/>
                        </group>
                        <group>
                            <field name="chat_room_id" required="0"/>
                            <field name="room_participant_count" readonly="1"/>
                            <field name="room_max_capacity" widget="radio" options="{'horizontal': true}"/>
                            <field name="room_lang_id" options="{'no_create': True}"/>
                        </group>
                    </group>
                    <notebook>
                        <page name="Reporting" string="Reporting">
                            <group>
                                <field name="room_last_activity"/>
                                <field name="room_max_participant_reached"/>
                            </group>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>
    <record id="event_meeting_room_view_tree" model="ir.ui.view">
        <field name="name">event.meeting.room.tree</field>
        <field name="model">event.meeting.room</field>
        <field name="arch" type="xml">
            <tree string="Meeting Room" multi_edit="1" sample="1">
                <field name="name"/>
                <field name="summary" optional="hide"/>
                <field name="target_audience"/>
                <field name="is_published"/>
                <field name="is_pinned"/>
                <field name="room_is_full" readonly="1"/>
                <field name="room_participant_count" readonly="1" sum="Total Participant Count" />
                <field name="room_max_capacity"/>
                <field name="room_lang_id"/>
            </tree>
        </field>
    </record>
    <record id="event_meeting_room_view_search" model="ir.ui.view">
        <field name="name">event.meeting.room.search</field>
        <field name="model">event.meeting.room</field>
        <field name="arch" type="xml">
            <search>
                <field name="name" string="Topic"/>
                <field name="summary" string="Summary"/>
                <field name="target_audience" string="Audience"/>
                <field name="event_id" string="Event"/>
                <filter domain="[('is_published', '=', False)]" name="filter_unpublished" string="Unpublished"/>
                <group expand="0" string="Group By">
                    <filter string="Event" name="groupby_event" domain="[]" context="{'group_by': 'event_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="action_meeting_room_from_event" model="ir.actions.act_window">
        <field name="res_model">event.meeting.room</field>
        <field name="name">Event Rooms</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{'search_default_event_id': active_id, 'default_event_id': active_id}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Room
            </p>
            <p>
                Rooms allow your event attendees to meet up and chat on different topics.
            </p>
        </field>
    </record>

</odoo>

```

## File: views\event_meet_templates_list.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="event_meet" name="Meeting Rooms">
    <t t-call="website_event.layout">
        <div class="o_wevent_online o_wemeet_index">
            <!-- Drag/Drop Area -->
            <div id="oe_structure_website_event_location_1" class="oe_structure"/>
            <!-- Content -->
            <div class="o_wemeet_container container">
                <div class="row mb-5 mx-0">
                    <t t-call="website_event_meet.community_main"/>
                    <t t-call="website_event_meet.community_aside"/>
                </div>
            </div>
            <!-- Drag/Drop Area -->
            <div id="oe_structure_website_event_location_2" class="oe_structure mb-5"/>
        </div>
    </t>
</template>

<!-- ============================================================ -->
<!-- CONTENT: MAIN TEMPLATES -->
<!-- ============================================================ -->

<!-- Meeting Rooms Main Display -->
<template id="community_main" name="Meeting Rooms: Main Display">
    <div class="col-12 col-md-8 ps-0 pe-0 pe-md-3 mt-3">
        <h2 class="d-flex flex-row justify-content-between">
            <span>Join a room</span>
            <div class="dropdown">
                <a class="dropdown-toggle o-no-caret btn btn-outline-secondary py-0 px-1" title="Languages Menu"
                    aria-label="Dropdown menu" data-bs-display="static" data-bs-toggle="dropdown" href="#" role="button">
                    <span t-out="current_lang.name if current_lang else 'All Languages'"/> ▼</a>
                <div class="dropdown-menu" role="menu">
                    <a class="dropdown-item" role="menuitem" t-attf-href="/event/#{slug(event)}/community">All Languages
                       </a>
                    <a class="dropdown-item" role="menuitem" t-as="language" t-attf-href="/event/#{slug(event)}/community?lang=#{language.id}" t-out="language.name" t-foreach="available_languages"/>
                </div>
            </div>
        </h2>
        <hr class="mt-2 mb-3"/>
        <p class="mt-">Choose a topic that interests you and start talking with the community. <br/> Don't forget to setup your camera and microphone.</p>
        <div class="d-flex flex-column justify-content-start align-items-start">
            <t t-as="meeting_room" t-call="website_event_meet.meeting_room_card" t-foreach="meeting_rooms">
                <t t-set="meeting_room" t-value="meeting_room"/>
                <t t-set="opened" t-value="int(meeting_room.id == open_room_id)"/>
            </t>
            <div t-if="not meeting_rooms" class="m-auto text-center text-muted">
                <h3 class="mt8">No Room Open</h3>
                <div groups="event.group_event_user">
                    <a target="_blank" t-att-href="'/web?#action=website_event_meet.action_meeting_room_from_event&amp;active_id=%s' % event.id">
                        <p>Create one to get conversations going</p>
                    </a>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="meeting_room_card" name="Meeting Room Card">
    <div class="modal o_join_later_modal fixed-top" t-attf-id="o_join_later_modal_#{meeting_room.id}" tabindex="-1" role="dialog" style="top: 0">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="mt-4 col-12 alert alert-warning text-center" role="alert">
                    <nav class="navbar navbar-default">
                        <div class="container-fluid">
                            <div class="navbar-header">
                                <div class="o_wevent_meeting_room_card_menu"></div>
                            </div>
                        </div>
                    </nav>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                    <t t-if="not event.is_done">
                        <span>This room is not open right now!</span><br />
                        Join us here on the
                        <strong itemprop="startDate" t-field="event.date_begin" t-options="{'tz_name': event.date_tz, 'format': 'medium'}"/>
                        <strong>(<t t-out="event.date_tz"/>)</strong>
                        to have a chat with us!
                    </t>
                    <t t-else="">
                        Event <span t-out="event.name" class="fw-bold"/> is over.
                        <br/>
                        <span>Join us next time to chat about <b t-out="meeting_room.name"/>!</span>
                    </t>
                </div>

                <div class="modal-body row">
                    <div class="col-3">
                        <div class="w-100" t-attf-style="background-image: #{json.loads(event.cover_properties).get('background-image')}; min-height: 5rem; background-size: cover;"/>
                    </div>
                    <div class="col">
                        <h5 t-out="meeting_room.name"/>
                        <div class="text-muted mb-2"><i class="fa fa-globe"/> <span t-out="meeting_room.room_lang_id.name"/></div>
                        <span t-if="meeting_room.summary" t-field="meeting_room.summary"/>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Action to do when clicking on the card -->
    <t t-if="event.is_ongoing or is_event_user">
        <!--During the event or if event manager-->
        <t t-set="meeting_room_href" t-value="'/event/' + slug(event) + '/meeting_room/' + slug(meeting_room)"/>
    </t>
    <t t-elif="not event.is_ongoing and not event.is_participating">
        <!--Pre-event, if not registered yet-->
        <t t-set="meeting_room_href" t-value="'/event/' + slug(event) + '/register?from_room_id=%i' % meeting_room.id"/>
    </t>
    <t t-else="">
        <!--Pre-event, if registered but event not stared yet-->
        <t t-set="meeting_room_href" t-value="'#'"/>
        <t t-set="meeting_room_data_toggle" t-value="'modal'"/>
        <t t-set="meeting_room_data_target" t-value="'#o_join_later_modal_%i' % meeting_room.id"/>
    </t>

    <a t-if="is_event_user or not meeting_room.room_is_full"
        class="card o_wevent_meeting_room_card w-100 my-2 d-block text-decoration-none rounded-0"
        t-att-data-meeting-room-id="meeting_room.id"
        t-att-data-open-room="opened"
        t-att-data-is-event-manager="int(is_event_user)"
        t-att-href="meeting_room_href"
        t-att-data-bs-toggle="meeting_room_data_toggle"
        t-att-data-bs-target="meeting_room_data_target">
        <div class="text-decoration-none w-100 h-100 p-3" t-att-data-publish="meeting_room.website_published and 'on' or 'off'">
            <div class="o_wevent_meeting_room_corner_ribbon" t-if="meeting_room.room_is_full">Full</div>
            <div class="d-flex flex-column">
                <div class="d-flex flex-row">
                    <h4 t-att-class="'text-break text-uppercase %s' % ('w-75' if meeting_room.website_published else 'w-50')" t-out="meeting_room.name"/>
                    <div t-if="not meeting_room.is_published" class="w-25">
                        <span class="badge text-bg-danger">Unpublished</span>
                    </div>
                </div>
                <span class="text-muted" t-field="meeting_room.summary"/>
                <div class="d-flex flex-row justify-content-between align-items-center">
                    <span class="h6 m-0">
                        <span t-out="meeting_room.room_participant_count"/>&amp;nbsp;
                        <span t-if="meeting_room.target_audience" class="text-uppercase" t-field="meeting_room.target_audience"/>
                        <span t-else="" class="text-uppercase">participant(s)</span>
                    </span>
                    <div class="d-inline border py-1 px-2 text-bg-secondary" t-out="meeting_room.room_lang_id.name"/>
                </div>
            </div>
        </div>
        <div t-if="is_event_user" class="position-absolute o_wevent_meeting_room_manager_menu d-flex justify-content-end flex-column flex-md-row">
            <button t-attf-class="o_wevent_meeting_room_is_pinned btn #{'o_wevent_meeting_room_pinned' if meeting_room.is_pinned else ''}">
                <i class="fa fa-thumb-tack"/>
            </button>
            <div class="dropdown dropstart">
                <button class="btn" data-bs-toggle="dropdown"><i class="fa fa-ellipsis-v px-1"/></button>
                <div class="dropdown-menu">
                    <button class="dropdown-item btn o_wevent_meeting_room_duplicate" type="button">Duplicate</button>
                    <button class="dropdown-item btn o_wevent_meeting_room_delete" type="button">Close</button>
                </div>
            </div>
        </div>
    </a>
</template>

<!-- ============================================================ -->
<!-- ASIDE: CREATE A ROOM -->
<!-- ============================================================ -->

<template id="community_aside" name="Community: Aside">
    <div class="col-md-4 p-0 mt-3 o_wevent_community_aside">
        <div class="d-none d-md-block mb-3" t-if="event.meeting_room_allow_creation">
            <h2>Start a topic</h2>
            <hr class="mt-2 mb-3"/>
            <p>Want to create your own discussion room ?</p>
            <a href="#" role="button"
                class="btn btn-primary o_wevent_create_room_button"
                t-if="event.is_ongoing or event.start_today or is_event_user"
                t-att-data-event-id="event.id"
                t-att-data-default-lang-code="default_lang_code">
                <span>Create a Room</span>
            </a>
            <div t-else="" class="d-flex flex-column">
                <button disabled="disabled" class="btn btn-primary align-self-start">Create a Room</button>
                Room creation will be available when event starts at 
                <span>
                    <span class="fw-bold" t-field="event.with_context(tz=event.date_tz).date_begin"
                        t-options="{'format': 'medium'}"/>
                    <span class="small">(<t t-out="event.date_tz"/>)</span>
                </span>
            </div>
        </div>
    </div>
</template>

</odoo>

```

## File: views\event_meet_templates_page.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="event_meet_main" name="Event Meet">
    <t t-set="no_header" t-value="option_widescreen"/>
    <t t-set="no_footer" t-value="option_widescreen"/>
    <t t-call="website_event.layout">
        <div class="o_wevent_online o_wemeet_index">
            <!-- Options -->
            <t t-set="option_widescreen" t-value="option_widescreen or False"/>
            <!-- Drag/Drop Area -->
            <div id="oe_structure_wemeet_index_1" class="oe_structure"/>
            <!-- Content -->
            <div t-att-class="'o_wevent_online_page_container %s' % ('container pb-3' if not option_widescreen else 'pb-3')">
                <div t-att-class="'row mb-5 mx-0 %s' % ('justify-content-center' if not meeting_rooms_other else '')">
                    <t t-if="meeting_rooms_other">
                        <t t-call="website_event_meet.meeting_room_aside"/>
                    </t>
                    <t t-call="website_event_meet.meeting_room_main"/>
                </div>
            </div>
            <!-- Drag/Drop Area -->
            <div id="oe_structure_wemeet_index_2" class="oe_structure"/>
        </div>
    </t>
</template>

<!-- ============================================================ -->
<!-- CONTENT: MAIN TEMPLATES -->
<!-- ============================================================ -->

<template id="meeting_room_main" name="Meeting Room: Main Content">
    <div t-att-class="'col-12 o_wemeet_room_main o_wevent_theme_bg_base border mt-3 p-0 %s' % ('col-md-9 col-lg-10' if option_widescreen else 'col-md-8 col-lg-9')">
        <!-- EVENT NOT STARTED ALERTS -->
        <t t-if="not meeting_room.event_id.is_ongoing">
            <div t-if="meeting_room.event_id.is_done" class="alert alert-warning text-center">
                The event <span t-out="meeting_room.event_id.name" class="fw-bold"/> is over.
                <br/>
                <span>Join us next time to chat about <b t-out="meeting_room.name"/>!</span>
            </div>
            <div t-else="" class="alert alert-warning text-center">
                The event <span t-out="meeting_room.event_id.name" class="fw-bold"/>
                <span t-if="meeting_room.event_id.start_today">
                    starts in
                    <span t-if="meeting_room.event_id.start_remaining &gt;= 1" t-out="meeting_room.event_id.start_remaining"
                        t-options="{'widget': 'duration', 'digital': False, 'unit': 'minute', 'round': 'minute'}"/>
                    <t t-else="">
                        a few seconds
                    </t>.
                </span>
                <span class="my-0" t-else="meeting_room.event_id.start_today">
                    starts on
                    <span t-field="meeting_room.event_id.with_context(tz=meeting_room.event_id.date_tz).date_begin"
                        t-options="{'format': 'medium'}"/> (<t t-out="meeting_room.event_id.date_tz"/>).
                </span>
                <br/>
                <span>Join us there to chat about <b t-out="meeting_room.name"/> !</span>
            </div>
        </t>
        <!-- ROOM CONTENT -->
        <div class="d-flex flex-column">
            <div t-if="meeting_room.room_is_full and not is_event_user" class="alert alert-warning text-center">
                <span>Oops! This room is full !</span><br />Come back later to have a chat with us!
            </div>
            <div t-else="" class="d-flex flex-column">
                <div id="o_wemeet_jitsi_iframe">
                    <div class="o_wjitsi_chat_room_loading position-absolute w-100 text-center text-muted">
                        <i class="fa fa-spin fa-circle-o-notch me-3"/>
                        <span>Loading your room...</span>
                    </div>
                </div>
                <div class="d-flex flex-row-reverse">
                    <t t-call="website_jitsi.chat_room_join_button">
                        <t t-set="_classes" t-value="'d-none'"/>
                        <t t-set="room_name" t-value="meeting_room.room_name"/>
                        <t t-set="chat_room_id" t-value="meeting_room.chat_room_id.id"/>
                        <t t-set="auto_open" t-value="1"/>
                        <t t-set="attach_to" t-value="'#o_wemeet_jitsi_iframe'"/>
                        <t t-set="max_capacity" t-value="meeting_room.room_max_capacity"/>
                        <t t-set="check_full" t-value="int(not is_event_user)"/>
                        <t t-set="default_username" t-value="default_username"/>
                        <t t-set="jitsi_server_domain" t-value="meeting_room.chat_room_id.jitsi_server_domain"/>
                    </t>
                </div>
            </div>
        </div>
        <!-- ROOM DESCRIPTION -->
        <div class="mx-3">
            <div class="h5 my-3" t-out="meeting_room.name"/>
            <div t-if="meeting_room.room_participant_count > 0" class="text-muted mb-3">
                A chat among <t t-out="meeting_room.room_participant_count"/> <span class="fw-bold" t-out="meeting_room.target_audience"/>
            </div>
        </div>
        <hr class="my-0"/>
        <div t-field="meeting_room.summary" class="my-2 mx-3 oe_no_empty"/>
     </div>
</template>

<!-- ============================================================ -->
<!-- ASIDE: CONTROL PANEL -->
<!-- ============================================================ -->

<template id="meeting_room_aside" name="Meeting Room: Aside">
    <div t-att-class="'col-12 mt-3 ps-0 pe-0 border o_wevent_online_page_aside %s' % ('col-md-3 col-lg-2' if option_widescreen else 'col-md-4 col-lg-3')">
        <div class="o_wevent_online_page_aside_content">
            <div class="d-flex align-items-center justify-content-between me-2">
                <span class="h5 m-3">Other Rooms</span>
                <a href="#collapse_meet_room_aside" data-bs-toggle="collapse" class="d-lg-none p-2 text-decoration-none o_wevent_online_page_aside_collapse collapsed">
                    <i class="fa fa-chevron-down d-lg-none"/>
                </a>
            </div>
            <ul id="collapse_meet_room_aside" class="list-unstyled collapse d-lg-block mb-0">
                <li t-foreach="meeting_rooms_other" t-as="meeting_room_other">
                    <a class="d-block w-100 h-100 px-2 pt-2 pb-1 text-decoration-none"
                        t-att-href="'/event/%s/meeting_room/%s' % (slug(event), slug(meeting_room_other))">
                        <div class="flex-grow-1 mw-100">
                            <div class="text-truncate" t-out="meeting_room_other.name"/>
                            <span class="text-muted" t-out="meeting_room_other.summary"></span>
                            <div class="d-flex justify-content-between align-items-center">
                                <div class="text-muted">
                                    <b>&amp;#9900;&amp;nbsp;</b>
                                    <small><t t-out="meeting_room_other.room_participant_count"/> <t t-out="meeting_room_other.target_audience"/></small>
                                </div>
                                <small t-if="meeting_room_other.room_lang_id" class="text-muted"><i class="fa fa-globe"/> <t t-out="meeting_room_other.room_lang_id.name"/></small>
                            </div>
                        </div>
                    </a>
                </li>
            </ul>
        </div>
    </div>
</template>

</odoo>

```

## File: views\event_type_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>
    <record id="event_type_view_form" model="ir.ui.view">
        <field name="name">event.type.view.form.inherit.meet</field>
        <field name="model">event.type</field>
        <field name="inherit_id" ref="website_event.event_type_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='community_menu']" position='after'>
                <span attrs="{'invisible': [('community_menu', '=', False)]}">
                    <label for="meeting_room_allow_creation"/>
                    <field name="meeting_room_allow_creation"/>
                </span>
            </xpath>
            <xpath expr="//span[@name='community_menu']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
        </field>
    </record>

</data></odoo>

```

## File: views\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="snippet_options" inherit_id="website.snippet_options" name="Event Meet Snippet Options">
    <xpath expr="." position="inside">
        <div data-js="WebsiteEvent" data-selector="main:has(.o_wemeet_container)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Event Page">
            <we-checkbox string="Room Creation (Specific)"
                         data-allow-room-creation="true"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
    </xpath>
</template>

</odoo>

```

