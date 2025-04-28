# Odoo Module: website_jitsi

Category: Hidden

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
    "name": "Website Jitsi",
    'category': 'Hidden',
    'version': '1.0',
    "summary": "Create Jitsi room on website.",
    'website': 'https://www.odoo.com/app/events',
    "description": "Create Jitsi room on website.",
    "depends": [
        "website"
    ],
    "data": [
        'views/chat_room_templates.xml',
        'views/chat_room_views.xml',
        'views/res_config_settings.xml',
        'security/ir.model.access.csv',
    ],
    'application': False,
    'assets': {
        'web.assets_frontend': [
            'website_jitsi/static/src/css/chat_room.css',
            'website_jitsi/static/src/js/chat_room.js',
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

from odoo import http
from odoo.http import request


class WebsiteJitsiController(http.Controller):

    @http.route(["/jitsi/update_status"], type="json", auth="public")
    def jitsi_update_status(self, room_name, participant_count, joined):
        """ Update room status: participant count, max reached

        Use the SQL keywords "FOR UPDATE SKIP LOCKED" in order to skip if the row
        is locked (instead of raising an exception, wait for a moment and retry).
        This endpoint may be called multiple times and we don't care having small
        errors in participant count compared to performance issues.

        :raise ValueError: wrong participant count
        :raise NotFound: wrong room name
        """
        if participant_count < 0:
            raise ValueError()

        chat_room = self._chat_room_exists(room_name)
        if not chat_room:
            raise NotFound()

        request.env.cr.execute(
            """
            WITH req AS (
                SELECT id
                  FROM chat_room
                  -- Can not update the chat room if we do not have its name
                 WHERE name = %s
                   FOR UPDATE SKIP LOCKED
            )
            UPDATE chat_room AS wcr
               SET participant_count = %s,
                   last_activity = NOW(),
                   max_participant_reached = GREATEST(max_participant_reached, %s)
              FROM req
             WHERE wcr.id = req.id;
            """,
            [room_name, participant_count, participant_count]
        )

    @http.route(["/jitsi/is_full"], type="json", auth="public")
    def jitsi_is_full(self, room_name):
        return self._chat_room_exists(room_name).is_full

    # ------------------------------------------------------------
    # TOOLS
    # ------------------------------------------------------------

    def _chat_room_exists(self, room_name):
        return request.env["chat.room"].sudo().search([("name", "=", room_name)], limit=1)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\chat_room.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from uuid import uuid4

from odoo import api, fields, models


class ChatRoom(models.Model):
    """ Store all useful information to manage chat room (currently limited
    to Jitsi). This model embeds all information about the chat room. We do not
    store them in the related mixin (see chat.room.mixin) to avoid to add too
    many fields on the models which want to use the chat room mixin as the
    behavior can be optional in those models.

    The participant count is automatically updated thanks to the chat room widget
    to avoid having a costly computed field with a members model.
    """
    _name = "chat.room"
    _description = "Chat Room"

    def _default_name(self, objname='room'):
        return "odoo-%s-%s" % (objname, str(uuid4())[:8])

    name = fields.Char(
        "Room Name", required=True, copy=False,
        default=lambda self: self._default_name())
    is_full = fields.Boolean("Full", compute="_compute_is_full")
    jitsi_server_domain = fields.Char(
        'Jitsi Server Domain', compute='_compute_jitsi_server_domain',
        help='The Jitsi server domain can be customized through the settings to use a different server than the default "meet.jit.si"')
    lang_id = fields.Many2one(
        "res.lang", "Language",
        default=lambda self: self.env["res.lang"].search([("code", "=", self.env.user.lang)], limit=1))
    max_capacity = fields.Selection(
        [("4", "4"), ("8", "8"), ("12", "12"), ("16", "16"),
         ("20", "20"), ("no_limit", "No limit")], string="Max capacity",
        default="8", required=True)
    participant_count = fields.Integer("Participant count", default=0, copy=False)
    # reporting fields
    last_activity = fields.Datetime(
        "Last Activity", copy=False, readonly=True,
        default=lambda self: fields.Datetime.now())
    max_participant_reached = fields.Integer(
        "Max participant reached", copy=False, readonly=True,
        help="Maximum number of participant reached in the room at the same time")

    @api.depends("max_capacity", "participant_count")
    def _compute_is_full(self):
        for room in self:
            if room.max_capacity == "no_limit":
                room.is_full = False
            else:
                room.is_full = room.participant_count >= int(room.max_capacity)

    def _compute_jitsi_server_domain(self):
        jitsi_server_domain = self.env['ir.config_parameter'].sudo().get_param(
            'website_jitsi.jitsi_server_domain', 'meet.jit.si')

        for room in self:
            room.jitsi_server_domain = jitsi_server_domain

```

## File: models\chat_room_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo import api, fields, models
from odoo.tools import remove_accents

class ChatRoomMixin(models.AbstractModel):
    """Add the chat room configuration (`chat.room`) on the needed models.

    The chat room configuration contains all information about the room. So, we store
    all the chat room logic at the same place, for all models.
    Embed chat room related fields prefixed with `room_`.
    """
    _name = "chat.room.mixin"
    _description = "Chat Room Mixin"
    ROOM_CONFIG_FIELDS = [
        ('room_name', 'name'),
        ('room_lang_id', 'lang_id'),
        ('room_max_capacity', 'max_capacity'),
        ('room_participant_count', 'participant_count')
    ]

    chat_room_id = fields.Many2one("chat.room", "Chat Room", readonly=True, copy=False, ondelete="set null")
    # chat room related fields
    room_name = fields.Char("Room Name", related="chat_room_id.name")
    room_is_full = fields.Boolean("Room Is Full", related="chat_room_id.is_full")
    room_lang_id = fields.Many2one("res.lang", "Language", related="chat_room_id.lang_id", readonly=False)
    room_max_capacity = fields.Selection(string="Max capacity", related="chat_room_id.max_capacity", readonly=False, required=True)
    room_participant_count = fields.Integer("Participant count", related="chat_room_id.participant_count", readonly=False)
    room_last_activity = fields.Datetime("Last activity", related="chat_room_id.last_activity")
    room_max_participant_reached = fields.Integer("Peak participants", related="chat_room_id.max_participant_reached")

    @api.model_create_multi
    def create(self, values_list):
        for values in values_list:
            if any(values.get(fmatch[0]) for fmatch in self.ROOM_CONFIG_FIELDS) and not values.get('chat_room_id'):
                if values.get('room_name'):
                    values['room_name'] = self._jitsi_sanitize_name(values['room_name'])
                room_values = dict((fmatch[1], values[fmatch[0]]) for fmatch in self.ROOM_CONFIG_FIELDS if values.get(fmatch[0]))
                values['chat_room_id'] = self.env['chat.room'].create(room_values).id
        return super(ChatRoomMixin, self).create(values_list)

    def write(self, values):
        if any(values.get(fmatch[0]) for fmatch in self.ROOM_CONFIG_FIELDS):
            if values.get('room_name'):
                values['room_name'] = self._jitsi_sanitize_name(values['room_name'])
            for document in self.filtered(lambda doc: not doc.chat_room_id):
                room_values = dict((fmatch[1], values[fmatch[0]]) for fmatch in self.ROOM_CONFIG_FIELDS if values.get(fmatch[0]))
                document.chat_room_id = self.env['chat.room'].create(room_values).id
        return super(ChatRoomMixin, self).write(values)

    def copy_data(self, default=None):
        if default is None:
            default = {}
        if self.chat_room_id:
            chat_room_default = {}
            if 'room_name' not in default:
                chat_room_default['name'] = self._jitsi_sanitize_name(self.chat_room_id.name)
            default['chat_room_id'] = self.chat_room_id.copy(default=chat_room_default).id
        return super(ChatRoomMixin, self).copy_data(default=default)

    def unlink(self):
        rooms = self.chat_room_id
        res = super(ChatRoomMixin, self).unlink()
        rooms.unlink()
        return res

    def _jitsi_sanitize_name(self, name):
        sanitized = re.sub(r'[^\w+.]+', '-', remove_accents(name).lower())
        counter, sanitized_suffixed = 1, sanitized
        existing = self.env['chat.room'].search([('name', '=like', '%s%%' % sanitized)]).mapped('name')
        while sanitized_suffixed in existing:
            sanitized_suffixed = '%s-%d' % (sanitized, counter)
            counter += 1
        return sanitized_suffixed

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    jitsi_server_domain = fields.Char(
        'Jitsi Server Domain', default='meet.jit.si', config_parameter='website_jitsi.jitsi_server_domain',
        help='The Jitsi server domain can be customized through the settings to use a different server than the default "meet.jit.si"')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import chat_room
from . import chat_room_mixin
from . import res_config_settings

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_chat_room_all,access.chat.room.all,model_chat_room,,0,0,0,0
access_chat_room_user,access.chat.room.user,model_chat_room,base.group_user,1,0,0,0
access_chat_room_system,access.chat.room.system,model_chat_room,base.group_system,1,1,1,1

```

## File: static\src\js\chat_room.js

```javascript
odoo.define('website_jitsi.chat_room', function (require) {
'use strict';

const config = require("web.config");
const core = require('web.core');
const Dialog = require('web.Dialog');
const publicWidget = require('web.public.widget');
const QWeb = core.qweb;
const _t = core._t;

publicWidget.registry.ChatRoom = publicWidget.Widget.extend({
    selector: '.o_wjitsi_room_widget',
    xmlDependencies: ['/website_jitsi/static/src/xml/chat_room_modal.xml'],
    events: {
        'click .o_wjitsi_room_link': '_onChatRoomClick',
    },

    /**
      * Manage the chat room (Jitsi), update the participant count...
      *
      * The widget takes some options
      * - 'room-name', the name of the Jitsi room
      * - 'chat-room-id', the ID of the `chat.room` record
      * - 'auto-open', the chat room will be automatically opened when the page is loaded
      * - 'check-full', check if the chat room is full before joining
      * - 'attach-to', a JQuery selector of the element on which we will add the Jitsi
      *                iframe. If nothing is specified, it will open a modal instead.
      * - 'default-username': the username to use in the chat room
      * - 'jitsi-server': the domain name of the Jitsi server to use
      */
    start: async function () {
        await this._super.apply(this, arguments);
        this.roomName = this.$el.data('room-name');
        this.chatRoomId = parseInt(this.$el.data('chat-room-id'));
        // automatically open the current room
        this.autoOpen = parseInt(this.$el.data('auto-open') || 0);
        // before joining, perform a RPC call to verify that the chat room is not full
        this.checkFull = parseInt(this.$el.data('check-full') || 0);
        // query selector of the element on which we attach the Jitsi iframe
        // if not defined, the widget will pop in a modal instead
        this.attachTo = this.$el.data('attach-to') || false;
        // default username for jitsi
        this.defaultUsername = this.$el.data('default-username') || false;

        this.jitsiServer = this.$el.data('jitsi-server') || 'meet.jit.si';

        this.maxCapacity = parseInt(this.$el.data('max-capacity')) || Infinity;

        if (this.autoOpen) {
            await this._onChatRoomClick();
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
      * Click on a chat room to join it.
      *
      * @private
      */
    _onChatRoomClick: async function () {
        if (this.checkFull) {
            // maybe we didn't refresh the page for a while and so we might join a room
            // which is full, so we perform a RPC call to verify that we can really join
            let isChatRoomFull = await this._rpc({
                route: '/jitsi/is_full',
                params: {
                    room_name: this.roomName,
                },
            });

            if (isChatRoomFull) {
                window.location.reload();
                return;
            }
        }

        if (await this._openMobileApplication(this.roomName)) {
            // we opened the mobile application
            return;
        }

        await this._loadJisti();

        if (this.attachTo) {
            // attach the Jitsi iframe on the given parent node
            let $parentNode = $(this.attachTo);
            $parentNode.find("iframe").trigger("empty");
            $parentNode.empty();

            await this._joinJitsiRoom($parentNode);
        } else {
            // create a model and append the Jitsi iframe in it
            let $jitsiModal = $(QWeb.render('chat_room_modal', {}));
            $("body").append($jitsiModal);
            $jitsiModal.modal('show');

            let jitsiRoom = await this._joinJitsiRoom($jitsiModal.find('.modal-body'));

            // close the modal when hanging up
            jitsiRoom.addEventListener('videoConferenceLeft', async () => {
                $('.o_wjitsi_room_modal').modal('hide');
            });

            // when the modal is closed, delete the Jitsi room object and clear the DOM
            $jitsiModal.on('hidden.bs.modal', async () => {
                jitsiRoom.dispose();
                $(".o_wjitsi_room_modal").remove();
            });
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
      * Jitsi do not provide an REST API to get the number of participant in a room.
      * The only way to get the number of participant is to be in the room and to use
      * the Javascript API. So, to update the participant count on the server side,
      * the participant have to send the count in RPC...
      *
      * When leaving a room, the event "participantLeft" is called for the current user
      * once per participant in the room (like if all other participants were leaving the
      * room and then the current user himself).
      *
      * "participantLeft" is called only one time for the other participant who are still
      * in the room.
      *
      * We can not ask the user who is leaving the room to update the participant count
      * because user might close their browser tab without hanging up (and so without
      * triggering the event "videoConferenceLeft"). So, we wait for a moment (because the
      * event "participantLeft" is called many time for the participant who is leaving)
      * and the first participant send the new participant count (so we avoid spamming the
      * server with HTTP requests).
      *
      * We use "setTimout" to send maximum one HTTP request per interval, even if multiple
      * participants join/leave at the same time in the defined interval.
      *
      * Update on the 29 June 2020
      *
      * @private
      * @param {jQuery} $jitsiModal, jQuery modal element in which we add the Jitsi room
      * @returns {JitsiRoom} the newly created Jitsi room
      */
    _joinJitsiRoom: async function ($parentNode) {
        let jitsiRoom = await this._createJitsiRoom(this.roomName, $parentNode);

        if (this.defaultUsername) {
            jitsiRoom.executeCommand("displayName", this.defaultUsername);
        }

        let timeoutCall = null;
        const updateParticipantCount = (joined) => {
            this.allParticipantIds = Object.keys(jitsiRoom._participants).sort();
            // if we reached the maximum capacity, update immediately the participant count
            const timeoutTime = this.allParticipantIds.length >= this.maxCapacity ? 0 : 2000;

            // we clear the old timeout to be sure to call it only once each 2 seconds
            // (so if 2 participants join/leave in this interval, we will perform only
            // one HTTP request for both).
            clearTimeout(timeoutCall);
            timeoutCall = setTimeout(() => {
                this.allParticipantIds = Object.keys(jitsiRoom._participants).sort();
                if (this.participantId === this.allParticipantIds[0]) {
                    // only the first participant of the room send the new participant
                    // count so we avoid to send to many HTTP requests
                    this._updateParticipantCount(this.allParticipantIds.length, joined);
                }
            }, timeoutTime);
        };

        jitsiRoom.addEventListener('participantJoined', () => updateParticipantCount(true));
        jitsiRoom.addEventListener('participantLeft', () => updateParticipantCount(false));

        // update the participant count when joining the room
        jitsiRoom.addEventListener('videoConferenceJoined', async (event) => {
            this.participantId = event.id;
            updateParticipantCount(true);
            $('.o_wjitsi_chat_room_loading').addClass('d-none');

            // recheck if the room is not full
            if (this.checkFull && this.allParticipantIds.length > this.maxCapacity) {
                clearTimeout(timeoutCall);
                jitsiRoom.executeCommand('hangup');
                window.location.reload();
            }
        });

        // update the participant count when using the "Leave" button
        jitsiRoom.addEventListener('videoConferenceLeft', async (event) => {
            this.allParticipantIds = Object.keys(jitsiRoom._participants)
            if (!this.allParticipantIds.length) {
                // bypass the checks and timer of updateParticipantCount
                this._updateParticipantCount(this.allParticipantIds.length, false);
            }
        });

        return jitsiRoom;
    },

    /**
      * Perform an HTTP request to update the participant count on the server side.
      *
      * @private
      * @param {integer} count, current number of participant in the room
      * @param {boolean} joined, true if someone joined the room
      */
    _updateParticipantCount: async function (count, joined) {
        await this._rpc({
            route: '/jitsi/update_status',
            params: {
                room_name: this.roomName,
                participant_count: count,
                joined: joined,
            },
        });
    },


    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
      * Redirect on the Jitsi mobile application if we are on mobile.
      *
      * @private
      * @param {string} roomName
      * @returns {boolean} true is we were redirected to the mobile application
      */
    _openMobileApplication: async function (roomName) {
        if (config.device.isMobile) {
            // we are on mobile, open the room in the application
            window.location = `intent://${this.jitsiServer}/${roomName}#Intent;scheme=org.jitsi.meet;package=org.jitsi.meet;end`;
            return true;
        }
        return false;
    },

    /**
      * Create a Jitsi room on the given DOM element.
      *
      * @private
      * @param {string} roomName
      * @param {jQuery} $parentNode
      * @returns {JitsiRoom} the newly created Jitsi room
      */
    _createJitsiRoom: async function (roomName, $parentNode) {
      await this._loadJisti();
        const options = {
            roomName: roomName,
            width: "100%",
            height: "100%",
            parentNode: $parentNode[0],
            configOverwrite: {disableDeepLinking: true},
        };
        return new window.JitsiMeetExternalAPI(this.jitsiServer, options);
    },

    /**
      * Load the Jitsi external library if necessary.
      *
      * @private
      */
    _loadJisti: async function () {
      if (!window.JitsiMeetExternalAPI) {
          await $.ajax({
              url: `https://${this.jitsiServer}/external_api.js`,
              dataType: "script",
          });
      }
    },
});

return publicWidget.registry.ChatRoom;

});

```

## File: static\src\xml\chat_room_modal.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-name="chat_room_modal">
        <div class="o_wjitsi_room_modal modal fade" role="dialog">
            <div class="modal-dialog">
                <div class="modal-content border-0">
                    <main class="modal-body p-0">
                        <div class="o_wjitsi_chat_room_loading position-absolute w-100 text-center text-muted">
                            <i class="fa fa-spin fa-circle-o-notch mr-3"/>
                            <span>Loading your room...</span>
                        </div>
                    </main>
                    <footer class="modal-footer">
                        <button class="btn btn-primary" data-dismiss="modal" type="button">Close</button>
                    </footer>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: views\chat_room_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <template id="chat_room_join_button">
        <div t-att-class="'o_wjitsi_room_widget %s' % (_classes or '')"
            t-att-data-chat-room-id="chat_room_id"
            t-att-data-room-name="room_name"
            t-att-data-auto-open="auto_open"
            t-att-data-check-full="check_full"
            t-att-data-attach-to="attach_to"
            t-att-data-default-username="default_username"
            t-att-data-max-capacity="max_capacity"
            t-att-data-jitsi-server="jitsi_server_domain">
            <button class="o_wjitsi_room_link btn btn-primary">Join the room</button>
        </div>
    </template>

</data></odoo>

```

## File: views\chat_room_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="chat_room_view_search" model="ir.ui.view">
        <field name="name">chat.room.search</field>
        <field name="model">chat.room</field>
        <field name="arch" type="xml">
            <search string="Chat Room">
                <field name="name"/>
            </search>
        </field>
    </record>

    <record id="chat_room_view_form" model="ir.ui.view">
        <field name="name">chat.room.form</field>
        <field name="model">chat.room</field>
        <field name="arch" type="xml">
            <form string="Chat Room">
                <sheet>
                    <label for="name"/>
                    <h1>
                        <field name="name"/>
                    </h1>
                    <group>
                        <group>
                            <field name="is_full"/>
                            <field name="lang_id" options="{'no_create': True}"/>
                            <field name="participant_count"/>
                            <field name="max_capacity"/>
                        </group>
                    </group>
                    <notebook>
                        <page name="Reporting" string="Reporting">
                            <group>
                                <field name="last_activity"/>
                                <field name="max_participant_reached"/>
                            </group>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="chat_room_view_tree" model="ir.ui.view">
        <field name="name">chat.room.tree</field>
        <field name="model">chat.room</field>
        <field name="arch" type="xml">
            <tree string="Chat Room">
                <field name="name"/>
                <field name="is_full"/>
                <field name="lang_id"/>
                <field name="participant_count"/>
                <field name="max_capacity"/>
            </tree>
        </field>
    </record>

    <record id="chat_room_action" model="ir.actions.act_window">
        <field name="name">Chat Rooms</field>
        <field name="res_model">chat.room</field>
        <field name="view_mode">tree,form</field>
    </record>

    <menuitem name="Chat Rooms"
        id="chat_room_menu"
        sequence="50"
        action="chat_room_action"
        parent="website.menu_website_global_configuration"
        groups="base.group_no_one"/>
</odoo>

```

## File: views\res_config_settings.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.jitsi</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website.res_config_settings_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//div[@id='google_maps_setting']" position="after">
                <div class="col-12 col-lg-6 o_setting_box">
                    <div class="o_setting_right_pane">
                        <label for="jitsi_server_domain"/>
                        <field name="jitsi_server_domain" placeholder="meet.jit.si"/>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

