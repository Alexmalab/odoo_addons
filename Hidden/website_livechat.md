# Odoo Module: website_livechat

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': 'Website Live Chat',
    'category': 'Hidden',
    'summary': 'Chat with your website visitors',
    'version': '1.0',
    'description': """
Allow website visitors to chat with the collaborators. This module also brings a feedback tool for the livechat and web pages to display your channel with its ratings on the website.
    """,
    'depends': ['website', 'im_livechat'],
    'installable': True,
    'auto_install': True,
    'data': [
        'views/website_livechat.xml',
        'views/res_config_settings_views.xml',
        'views/im_livechat_chatbot_script_view.xml',
        'views/website_livechat_view.xml',
        'views/website_visitor_views.xml',
        'views/im_livechat_channel_add.xml',
        'security/ir.model.access.csv',
        'security/website_livechat.xml',
        'data/website_livechat_data.xml',
    ],
    'demo': [
        'data/website_livechat_chatbot_demo.xml',
    ],
    'assets': {
        'website.assets_wysiwyg': [
            'website_livechat/static/src/scss/**/*',
        ],
        'website.assets_editor': [
            'website_livechat/static/src/js/**/*',
        ],
        'web.assets_frontend': [
            'website_livechat/static/src/patch/assets_frontend/website.scss',
        ],
        'web.assets_backend': [
            'website_livechat/static/src/**/*',
            ('remove', 'website_livechat/static/src/scss/**/*'),
        ],
        'web.assets_unit_tests': [
            'website_livechat/static/tests/**/*',
            ('remove', 'website_livechat/static/tests/embed/**/*'),
            ('remove', 'website_livechat/static/tests/tours/**/*'),
        ],
        'im_livechat.embed_assets_unit_tests': [
            'website_livechat/static/tests/mock_server/**/*',
            'website_livechat/static/tests/website_livechat_test_helpers.js',
            'website_livechat/static/tests/embed/**/*',
        ],
        'web.assets_tests': [
            'website_livechat/static/tests/tours/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\chatbot.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta
from odoo import http, Command, fields
from odoo.http import request
from odoo.addons.mail.tools.discuss import Store


class WebsiteLivechatChatbotScriptController(http.Controller):
    @http.route('/chatbot/<model("chatbot.script"):chatbot_script>/test',
        type="http", auth="user", website=True)
    def chatbot_test_script(self, chatbot_script):
        """ Custom route allowing to test a chatbot script.
        As we don't have a im_livechat.channel linked to it, we pre-emptively create a discuss.channel
        that will hold the conversation between the bot and the user testing the script. """

        channels = request.env["discuss.channel"].search([
            ["is_member", "=", True],
            ["livechat_active", "=", True],
            ["chatbot_current_step_id.chatbot_script_id", "=", chatbot_script.id],
        ])
        for channel in channels:
            channel._close_livechat_session()

        discuss_channel_values = {
            "channel_member_ids": [
                Command.create(
                    {
                        "partner_id": chatbot_script.operator_partner_id.id,
                        # making sure the unpin_dt is always later than the last_interest_dt
                        # so that the channel is unpinned
                        "unpin_dt": fields.Datetime.now(),
                        "last_interest_dt": fields.Datetime.now() - timedelta(seconds=30),
                    }
                ),
                Command.create(
                    {
                        "partner_id": request.env.user.partner_id.id,
                        "fold_state": "open",
                    }
                ),
            ],
            'livechat_active': True,
            'livechat_operator_id': chatbot_script.operator_partner_id.id,
            'chatbot_current_step_id': chatbot_script._get_welcome_steps()[-1].id,
            'anonymous_name': False,
            'channel_type': 'livechat',
            'name': chatbot_script.title,
        }

        visitor_sudo = request.env['website.visitor']._get_visitor_from_request()
        if visitor_sudo:
            discuss_channel_values['livechat_visitor_id'] = visitor_sudo.id

        discuss_channel = request.env['discuss.channel'].create(discuss_channel_values)
        chatbot_script._post_welcome_steps(discuss_channel)
        store = Store()
        request.env["res.users"]._init_store_data(store)
        return request.render("im_livechat.chatbot_test_script_page", {
            'server_url': chatbot_script.get_base_url(),
            'channel_data': {'id': discuss_channel.id, 'model': 'discuss.channel'},
            'chatbot_data': chatbot_script._format_for_frontend(),
            'storeData': store.get_result(),
        })

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http, _
from odoo.http import request
from odoo.addons.im_livechat.controllers.main import LivechatController


class WebsiteLivechat(LivechatController):

    @http.route('/livechat', type='http', auth="public", website=True, sitemap=True)
    def channel_list(self, **kw):
        # display the list of the channel
        channels = request.env['im_livechat.channel'].search([('website_published', '=', True)])
        values = {
            'channels': channels
        }
        return request.render('website_livechat.channel_list_page', values)

    @http.route('/livechat/channel/<model("im_livechat.channel"):channel>', type='http', auth='public', website=True, sitemap=True)
    def channel_rating(self, channel, **kw):
        # get the last 100 ratings and the repartition per grade
        domain = [
            ('res_model', '=', 'discuss.channel'), ('res_id', 'in', channel.sudo().channel_ids.ids),
            ('consumed', '=', True), ('rating', '>=', 1),
        ]
        ratings = request.env['rating.rating'].sudo().search(domain, order='create_date desc', limit=100)
        repartition = channel.sudo().channel_ids.rating_get_grades(domain=domain)

        # compute percentage
        percentage = dict.fromkeys(['great', 'okay', 'bad'], 0)
        for grade in repartition:
            percentage[grade] = round(repartition[grade] * 100.0 / sum(repartition.values()), 1) if sum(repartition.values()) else 0

        # filter only on the team users that worked on the last 100 ratings and get their detailed stat
        ratings_per_partner = {partner_id: dict(great=0, okay=0, bad=0)
                               for partner_id in ratings.mapped('rated_partner_id.id')}
        total_ratings_per_partner = dict.fromkeys(ratings.mapped('rated_partner_id.id'), 0)
        # keep 10 for backward compatibility
        rating_texts = {10: 'great', 5: 'great', 3: 'okay', 1: 'bad'}

        for rating in ratings:
            partner_id = rating.rated_partner_id.id
            if partner_id:
                ratings_per_partner[partner_id][rating_texts[rating.rating]] += 1
                total_ratings_per_partner[partner_id] += 1

        for partner_id, rating in ratings_per_partner.items():
            for k, v in ratings_per_partner[partner_id].items():
                ratings_per_partner[partner_id][k] = round(100 * v / total_ratings_per_partner[partner_id], 1)

        # the value dict to render the template
        values = {
            'main_object': channel,
            'channel': channel,
            'ratings': ratings,
            'team': channel.sudo().user_ids,
            'percentage': percentage,
            'ratings_per_user': ratings_per_partner
        }
        return request.render("website_livechat.channel_page", values)

    def _get_guest_name(self):
        visitor_sudo = request.env["website.visitor"]._get_visitor_from_request()
        return _('Visitor #%d', visitor_sudo.id) if visitor_sudo else super()._get_guest_name()

    @http.route()
    def get_session(self, channel_id, anonymous_name, previous_operator_id=None, chatbot_script_id=None, persisted=True, **kwargs):
        """ Override to use visitor name instead of 'Visitor' whenever a visitor start a livechat session. """
        visitor_sudo = request.env['website.visitor']._get_visitor_from_request()
        if visitor_sudo:
            anonymous_name = _('Visitor #%s', visitor_sudo.id)
        return super(WebsiteLivechat, self).get_session(channel_id, anonymous_name, previous_operator_id=previous_operator_id, chatbot_script_id=chatbot_script_id, persisted=persisted, **kwargs)

```

## File: controllers\test.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import Controller, request, route


class TestBusController(Controller):
    """
    This controller is only useful for test purpose. Bus is unavailable in test mode, but there is no way to know,
    at client side, if we are running in test mode or not. This route can be called while running tours to mock
    some behaviour in function of the test mode status (activated or not).

    E.g. : To test the livechat and to check there is no duplicates in message displayed in the chatter,
    in test mode, we need to mock a 'message added' notification that is normally triggered by the bus.
    In Normal mode, the bus triggers itself the notification.
    """
    @route('/bus/test_mode_activated', type="json", auth="public")
    def is_test_mode_activated(self):
        return request.registry.in_test_mode()

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import chatbot
from . import main
from . import test

```

## File: data\website_livechat_chatbot_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data noupdate="1">

    <!-- Add a Channel Rule to demonstrate our bot on the '/contactus' page -->

    <record id="website_livechat_channel_rule_chatbot" model="im_livechat.channel.rule" forcecreate='False'>
        <field name="regex_url">/contactus</field>
        <field name="sequence">5</field>
        <field name="action">auto_popup</field>
        <field name="auto_popup_timer">2</field>
        <field name="chatbot_script_id" ref="im_livechat.chatbot_script_welcome_bot"/>
        <field name="channel_id" ref="im_livechat.im_livechat_channel_data"/>
    </record>

</data></odoo>

```

## File: data\website_livechat_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
     <data noupdate="1">
         <record id="website.default_website" model="website" forcecreate="False">
             <field name="channel_id" ref="im_livechat.im_livechat_channel_data"></field>
         </record>
     </data>
</odoo>

```

## File: models\chatbot_script.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ChatbotScript(models.Model):
    _inherit = 'chatbot.script'

    def action_test_script(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'url': '/chatbot/%s/test' % self.id,
            'target': 'self',
        }

```

## File: models\chatbot_script_step.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ChatbotScriptStep(models.Model):
    _inherit = 'chatbot.script.step'

    def _chatbot_prepare_customer_values(self, discuss_channel, create_partner=True, update_partner=True):
        values = super()._chatbot_prepare_customer_values(discuss_channel, create_partner, update_partner)
        # sudo - website.visitor: chat bot can access visitor information
        if visitor_sudo := discuss_channel.livechat_visitor_id.sudo():
            if not values.get('email') and visitor_sudo.email:
                values['email'] = visitor_sudo.email
            if not values.get('phone') and visitor_sudo.mobile:
                values['phone'] = visitor_sudo.mobile
            values['country'] = {'id': visitor_sudo.country_id} if visitor_sudo.country_id else False

        return values

```

## File: models\discuss_channel.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.addons.mail.tools.discuss import Store
from odoo.exceptions import AccessError


class DiscussChannel(models.Model):
    _inherit = 'discuss.channel'

    livechat_visitor_id = fields.Many2one('website.visitor', string='Visitor', index='btree_not_null')

    def channel_pin(self, pinned=False):
        """ Override to clean an empty livechat channel.
         This is typically called when the operator send a chat request to a website.visitor
         but don't speak to them and closes the chatter.
         This allows operators to send the visitor a new chat request.
         If active empty livechat channel,
         delete discuss_channel as not useful to keep empty chat
         """
        super().channel_pin(pinned=pinned)
        if self.livechat_active and not self.message_ids:
            self.sudo().unlink()

    def _to_store(self, store: Store):
        """
        Override to add visitor information on the mail channel infos.
        This will be used to display a banner with visitor informations
        at the top of the livechat channel discussion view in discuss module.
        """
        super()._to_store(store)
        for channel in self.filtered('livechat_visitor_id'):
            channel_info = {
                "requested_by_operator": channel.create_uid in channel.livechat_operator_id.user_ids
            }
            visitor = channel.livechat_visitor_id
            try:
                country_id = visitor.partner_id.country_id or visitor.country_id
                channel_info['visitor'] = {
                    'name': visitor.partner_id.name or visitor.partner_id.display_name or visitor.display_name or _("Visitor #%(id)d.", id=visitor.id),
                    'country': {'id': country_id.id, 'code': country_id.code.lower()} if country_id else False,
                    'id': visitor.id,
                    'is_connected': visitor.is_connected,
                    'history': self.sudo()._get_visitor_history(visitor),
                    'website_name': visitor.website_id.name,
                    'lang_name': visitor.lang_id.name,
                    'partner_id': visitor.partner_id.id,
                    'type': "visitor",
                }
            except AccessError:
                pass
            store.add(channel, channel_info)

    def _get_visitor_history(self, visitor):
        """
        Prepare history string to render it in the visitor info div on discuss livechat channel view.
        :param visitor: website.visitor of the channel
        :return: arrow separated string containing navigation history information
        """
        recent_history = self.env['website.track'].search([('page_id', '!=', False), ('visitor_id', '=', visitor.id)], limit=3)
        return ' → '.join(visit.page_id.name + ' (' + visit.visit_datetime.strftime('%H:%M') + ')' for visit in reversed(recent_history))

    def _get_visitor_leave_message(self, operator=False, cancel=False):
        if not cancel:
            if self.livechat_visitor_id.id:
                return _("Visitor #%(id)d left the conversation.", id=self.livechat_visitor_id.id)
            return _("Visitor left the conversation.")
        return _(
            "%(visitor)s started a conversation with %(operator)s.\nThe chat request has been cancelled",
            visitor=self.livechat_visitor_id.display_name or _("The visitor"),
            operator=operator or _("an operator"),
        )

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, **kwargs):
        """Override to mark the visitor as still connected.
        If the message sent is not from the operator (so if it's the visitor or
        odoobot sending closing chat notification, the visitor last action date is updated."""
        message = super().message_post(**kwargs)
        message_author_id = message.author_id
        visitor = self.livechat_visitor_id
        if len(self) == 1 and visitor and message_author_id != self.livechat_operator_id:
            # sudo: website.visitor: updating data of a specific visitor
            visitor.sudo()._update_visitor_last_visit()
        return message

```

## File: models\im_livechat.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields
from odoo.tools.translate import html_translate


class ImLivechatChannel(models.Model):

    _name = 'im_livechat.channel'
    _inherit = ['im_livechat.channel', 'website.published.mixin']

    def _compute_website_url(self):
        super(ImLivechatChannel, self)._compute_website_url()
        for channel in self:
            channel.website_url = "/livechat/channel/%s" % (self.env['ir.http']._slug(channel),)

    website_description = fields.Html(
        "Website description", default=False, translate=html_translate,
        sanitize_overridable=True,
        sanitize_attributes=False, sanitize_form=False,
        help="Description of the channel displayed on the website page")

```

## File: models\im_livechat_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _


class ImLivechatChannel(models.Model):
    _inherit = 'im_livechat.channel'

    def _get_livechat_discuss_channel_vals(self, anonymous_name, previous_operator_id=None, chatbot_script=None, user_id=None, country_id=None, lang=None):
        discuss_channel_vals = super(ImLivechatChannel, self)._get_livechat_discuss_channel_vals(
            anonymous_name, previous_operator_id, chatbot_script, user_id=user_id, country_id=country_id, lang=lang
        )
        if not discuss_channel_vals:
            return False
        visitor_sudo = self.env['website.visitor']._get_visitor_from_request()
        if visitor_sudo:
            discuss_channel_vals['livechat_visitor_id'] = visitor_sudo.id
            # As chat requested by the visitor, delete the chat requested by an operator if any to avoid conflicts between two flows
            # TODO DBE : Move this into the proper method (open or init mail channel)
            chat_request_channel = self.env['discuss.channel'].sudo().search([('livechat_visitor_id', '=', visitor_sudo.id), ('livechat_active', '=', True)])
            for discuss_channel in chat_request_channel:
                operator = discuss_channel.livechat_operator_id
                operator_name = operator.user_livechat_username or operator.name
                discuss_channel._close_livechat_session(cancel=True, operator=operator_name)

        return discuss_channel_vals

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    @classmethod
    def _get_translation_frontend_modules_name(cls):
        mods = super(IrHttp, cls)._get_translation_frontend_modules_name()
        return mods + ['im_livechat']

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    channel_id = fields.Many2one('im_livechat.channel', string='Website Live Channel', related='website_id.channel_id', readonly=False)

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _, Command
from odoo.addons.mail.models.discuss.mail_guest import add_guest_to_context


class Website(models.Model):

    _inherit = "website"

    channel_id = fields.Many2one('im_livechat.channel', string='Website Live Chat Channel')

    @add_guest_to_context
    def _get_livechat_channel_info(self):
        """ Get the livechat info dict (button text, channel name, ...) for the livechat channel of
            the current website.
        """
        self.ensure_one()
        if self.channel_id:
            livechat_info = self.channel_id.sudo().get_livechat_info()
            if livechat_info['available']:
                livechat_request_session = self._get_livechat_request_session()
                if livechat_request_session:
                    livechat_info['options']['force_thread'] = livechat_request_session
            return livechat_info
        return {}

    def _get_livechat_request_session(self):
        """
        Check if there is an opened chat request for the website livechat channel and the current visitor (from request).
        If so, prepare the livechat session information that will be stored in visitor's cookies
        and used by livechat widget to directly open this session instead of allowing the visitor to
        initiate a new livechat session.
        :param {int} channel_id: channel
        :return: {dict} livechat request session information
        """
        visitor = self.env['website.visitor']._get_visitor_from_request()
        chat_request_session = {}
        if visitor:
            # get active chat_request linked to visitor
            chat_request_channel = self.env['discuss.channel'].sudo().search([
                ("channel_type", "=", "livechat"),
                ('livechat_visitor_id', '=', visitor.id),
                ('livechat_channel_id', '=', self.channel_id.id),
                ('livechat_active', '=', True),
                ('has_message', '=', True)
            ], order='create_date desc', limit=1)
            if chat_request_channel:
                if not visitor.partner_id:
                    current_guest = self.env['mail.guest']._get_guest_from_context()
                    channel_guest_member = chat_request_channel.channel_member_ids.filtered(lambda m: m.guest_id)
                    if current_guest and current_guest != channel_guest_member.guest_id:
                        # Channel was created with a guest but the visitor was
                        # linked to another guest in the meantime. We need to
                        # update the channel to link it to the current guest.
                        chat_request_channel.write({'channel_member_ids': [
                            Command.unlink(channel_guest_member.id),
                            Command.create({'guest_id': current_guest.id, 'fold_state': 'open'})
                        ]})
                    if not current_guest and channel_guest_member:
                        channel_guest_member.guest_id._set_auth_cookie()
                        chat_request_channel = chat_request_channel.with_context(guest=channel_guest_member.guest_id.sudo(False))
                if chat_request_channel.is_member:
                    chat_request_session = {
                        "id": chat_request_channel.id,
                        "model": "discuss.channel",
                    }
        return chat_request_session

    def get_suggested_controllers(self):
        suggested_controllers = super(Website, self).get_suggested_controllers()
        suggested_controllers.append((_('Live Support'), self.env['ir.http']._url_for('/livechat'), 'website_livechat'))
        return suggested_controllers

```

## File: models\website_visitor.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, Command, fields, models, _
from odoo.addons.mail.tools.discuss import Store
from odoo.exceptions import UserError
from odoo.http import request
from odoo.tools import get_lang
from odoo.tools.sql import column_exists, create_column


class WebsiteVisitor(models.Model):
    _inherit = 'website.visitor'

    livechat_operator_id = fields.Many2one('res.partner', compute='_compute_livechat_operator_id', store=True, string='Speaking with', index='btree_not_null')
    livechat_operator_name = fields.Char('Operator Name', related="livechat_operator_id.name")
    discuss_channel_ids = fields.One2many('discuss.channel', 'livechat_visitor_id',
                                       string="Visitor's livechat channels", readonly=True)
    session_count = fields.Integer('# Sessions', compute="_compute_session_count")

    def _auto_init(self):
        # Skip the computation of the field `livechat_operator_id` at the module installation
        # We can assume no livechat operator attributed to visitor if it was not installed
        if not column_exists(self.env.cr, "website_visitor", "livechat_operator_id"):
            create_column(self.env.cr, "website_visitor", "livechat_operator_id", "int4")
        return super()._auto_init()

    @api.depends('discuss_channel_ids.livechat_active', 'discuss_channel_ids.livechat_operator_id')
    def _compute_livechat_operator_id(self):
        results = self.env['discuss.channel'].search_read(
            [('livechat_visitor_id', 'in', self.ids), ('livechat_active', '=', True)],
            ['livechat_visitor_id', 'livechat_operator_id']
        )
        visitor_operator_map = {int(result['livechat_visitor_id'][0]): int(result['livechat_operator_id'][0]) for result in results}
        for visitor in self:
            visitor.livechat_operator_id = visitor_operator_map.get(visitor.id, False)

    @api.depends('discuss_channel_ids')
    def _compute_session_count(self):
        sessions = self.env['discuss.channel'].search([('livechat_visitor_id', 'in', self.ids)])
        session_count = dict.fromkeys(self.ids, 0)
        for session in sessions.filtered(lambda c: c.message_ids):
            session_count[session.livechat_visitor_id.id] += 1
        for visitor in self:
            visitor.session_count = session_count.get(visitor.id, 0)

    def action_send_chat_request(self):
        """ Send a chat request to website_visitor(s).
        This creates a chat_request and a discuss_channel with livechat active flag.
        But for the visitor to get the chat request, the operator still has to speak to the visitor.
        The visitor will receive the chat request the next time he navigates to a website page.
        (see _handle_webpage_dispatch for next step)"""
        # check if visitor is available
        unavailable_visitors_count = self.env['discuss.channel'].search_count([('livechat_visitor_id', 'in', self.ids), ('livechat_active', '=', True)])
        if unavailable_visitors_count:
            raise UserError(_('Recipients are not available. Please refresh the page to get latest visitors status.'))
        # check if user is available as operator
        for website in self.mapped('website_id'):
            if not website.channel_id:
                raise UserError(_('No Livechat Channel allows you to send a chat request for website %s.', website.name))
        self.website_id.channel_id.write({'user_ids': [(4, self.env.user.id)]})
        # Create chat_requests and linked discuss_channels
        discuss_channel_vals_list = []
        for visitor in self:
            operator = self.env.user
            country = visitor.country_id
            visitor_name = "Visitor #%d (%s)" % (visitor.id, country.name) if country else f"Visitor #{visitor.id}"
            members_to_add = [Command.link(operator.partner_id.id)]
            if visitor.partner_id:
                members_to_add.append(Command.link(visitor.partner_id.id))
            discuss_channel_vals_list.append({
                'channel_partner_ids': members_to_add,
                'livechat_channel_id': visitor.website_id.channel_id.id,
                'livechat_operator_id': self.env.user.partner_id.id,
                'channel_type': 'livechat',
                'country_id': country.id,
                'anonymous_name': visitor_name,
                'name': ', '.join([visitor_name, operator.livechat_username if operator.livechat_username else operator.name]),
                'livechat_visitor_id': visitor.id,
                'livechat_active': True,
            })
        discuss_channels = self.env['discuss.channel'].create(discuss_channel_vals_list)
        for channel in discuss_channels:
            if not channel.livechat_visitor_id.partner_id:
                # sudo: mail.guest - creating a guest in a dedicated channel created from livechat
                guest = self.env["mail.guest"].sudo().create(
                    {
                        "country_id": country.id,
                        "lang": get_lang(channel.env).code,
                        "name": _("Visitor #%d", channel.livechat_visitor_id.id),
                        "timezone": visitor.timezone,
                    }
                )
                channel.add_members(guest_ids=guest.ids, post_joined_message=False)
        # Open empty chatter to allow the operator to start chatting with
        # the visitor. Also open the visitor's chat window in order for it
        # to be displayed at the next page load.
        channel_members = self.env['discuss.channel.member'].sudo().search([
            ('channel_id', 'in', discuss_channels.ids),
        ])
        channel_members.write({
            'fold_state': 'open',
        })
        operator._bus_send(
            "website_livechat.send_chat_request", Store(discuss_channels).get_result()
        )

    def _merge_visitor(self, target):
        """ Copy sessions of the secondary visitors to the main partner visitor. """
        target.discuss_channel_ids |= self.discuss_channel_ids
        self.discuss_channel_ids.channel_partner_ids = [
            (3, self.env.ref('base.public_partner').id),
            (4, target.partner_id.id),
        ]
        return super()._merge_visitor(target)

    def _upsert_visitor(self, access_token, force_track_values=None):
        visitor_id, upsert = super()._upsert_visitor(access_token, force_track_values=force_track_values)
        if upsert == 'inserted':
            visitor_sudo = self.sudo().browse(visitor_id)
            if discuss_channel_uuid := request.cookies.get("im_livechat_uuid"):
                discuss_channel = request.env["discuss.channel"].sudo().search([("uuid", "=", discuss_channel_uuid)])
                discuss_channel.write({
                    'livechat_visitor_id': visitor_sudo.id,
                    'anonymous_name': "Visitor #%d (%s)" % (visitor_sudo.id, visitor_sudo.country_id.name) if visitor_sudo.country_id else f"Visitor #{visitor_sudo.id}"
                })
        return visitor_id, upsert

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import chatbot_script
from . import chatbot_script_step
from . import im_livechat
from . import im_livechat_channel
from . import ir_http
from . import discuss_channel
from . import res_config_settings
from . import website
from . import website_visitor

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_im_livechat_channel_public_public,im_livechat.channel.public,im_livechat.model_im_livechat_channel,base.group_public,1,0,0,0
access_im_livechat_channel_public_portal,im_livechat.channel.public,im_livechat.model_im_livechat_channel,base.group_portal,1,0,0,0
access_im_livechat_channel_public_employee,im_livechat.channel.public,im_livechat.model_im_livechat_channel,base.group_user,1,0,0,0
access_website_visitor_livechat_users,website.visitor.livechat.users,model_website_visitor,im_livechat.im_livechat_group_user,1,1,0,0
access_website_track_livechat_users,website.track.livechat.users,website.model_website_track,im_livechat.im_livechat_group_user,1,0,0,0

```

## File: security\website_livechat.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

        <record id="im_livechat_channel_rule_public" model="ir.rule">
            <field name="name">website_livechat.channel.public</field>
            <field name="model_id" ref="im_livechat.model_im_livechat_channel"/>
            <field name="domain_force">[('website_published', '=', True)]</field>
            <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_unlink" eval="0"/>
        </record>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M4 26.222C4 27.204 4.796 28 5.778 28H16c6.627 0 12-5.373 12-12S22.627 4 16 4 4 9.373 4 16v10.222Z" fill="#FBB945"/><path d="M46 5.778C46 4.796 45.204 4 44.222 4H34c-6.627 0-12 5.373-12 12s5.373 12 12 12 12-5.373 12-12V5.778Z" fill="#985184"/><path d="M25 23.937c1.867-2.115 3-4.894 3-7.937s-1.133-5.822-3-7.938A11.954 11.954 0 0 0 22 16c0 3.043 1.133 5.822 3 7.937Z" fill="#953B24"/><path d="M4 38.5C4 32.701 8.701 28 14.5 28S25 32.701 25 38.5V46H4v-7.5Z" fill="#FBB945"/><path d="M25 38.5C25 32.701 29.701 28 35.5 28S46 32.701 46 38.5V46H25v-7.5Z" fill="#985184"/></svg>

```

## File: static\src\js\systray_items\new_content.js

```javascript
import { NewContentModal, MODULE_STATUS } from "@website/systray_items/new_content";
import { patch } from "@web/core/utils/patch";

patch(NewContentModal.prototype, {
    setup() {
        super.setup();

        const newChannelElement = this.state.newContentElements.find(
            (element) => element.moduleXmlId === "base.module_website_livechat"
        );
        newChannelElement.createNewContent = () =>
            this.onAddContent("website_livechat.im_livechat_channel_action_add");
        newChannelElement.status = MODULE_STATUS.INSTALLED;
        newChannelElement.model = "im_livechat.channel";
    },
});

```

## File: static\src\web\channel_member_model_patch.js

```javascript
import { ChannelMember } from "@mail/core/common/channel_member_model";
import { patch } from "@web/core/utils/patch";

patch(ChannelMember.prototype, {
    getLangName() {
        if (this.persona.is_public && this.thread.visitor?.langName) {
            return this.thread.visitor.langName;
        }
        return super.getLangName();
    },
});

```

## File: static\src\web\thread_model_patch.js

```javascript
import { Record } from "@mail/core/common/record";
import { Thread } from "@mail/core/common/thread_model";
import { patch } from "@web/core/utils/patch";

patch(Thread.prototype, {
    setup() {
        super.setup(...arguments);
        this.visitor = Record.one("Persona");
        this.visitorPartner = Record.one("Persona");
    },
});

```

## File: static\src\web\thread_patch.js

```javascript
import { Thread } from "@mail/core/common/thread";
import { ImStatus } from "@mail/core/common/im_status";

Object.assign(Thread.components, { ImStatus });

```

## File: static\src\web\thread_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="website_livechat.Thread" t-inherit="mail.Thread" t-inherit-mode="extension">
        <xpath expr="//*[hasclass('o-mail-Thread')]" position="before">
            <t t-set="visitor" t-value="props.thread.visitor"/>
            <div t-if="visitor and !env.inChatWindow" class="o-website_livechat-VisitorBanner py-4 px-2 d-flex border-bottom">
                <div t-if="props.thread.correspondent" class="o-website_livechat-VisitorBanner-sidebar me-2 d-flex justify-content-center">
                    <img class="rounded o-website_livechat-VisitorBanner-avatar o_object_fit_cover" t-att-src="props.thread.correspondent.persona.avatarUrl" alt="Avatar"/>
                </div>
                <div>
                    <div class="d-flex align-items-baseline">
                        <ImStatus t-if="visitor.is_connected" className="'me-1'" persona="visitor"/>
                        <span class="me-2 fw-bolder" t-esc="visitor.name"/>
                        <img t-if="visitor.country" class="me-2 o_country_flag align-self-center" t-att-src="visitor.country.flagUrl" t-att-alt="visitor.country.code or visitor.country.name"/>
                        <span class="me-2">
                            <i class="me-1 fa fa-comment-o" aria-label="Lang"/>
                            <t t-esc="visitor.lang_name"/>
                        </span>
                        <span t-if="visitor.website_name">
                            <i class="me-1 fa fa-globe" aria-label="Website"/>
                            <span t-esc="visitor.website_name"/>
                        </span>
                    </div>
                    <div class="mt-1">
                        <i class="me-1 fa fa-history" aria-label="History"/>
                        <span t-esc="visitor.history"/>
                    </div>
                </div>
            </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\web\website_livechat_notification_handler.js

```javascript
import { registry } from "@web/core/registry";

export const websiteLivechatNotifications = {
    dependencies: ["bus_service", "mail.store"],
    start(env, { bus_service: busService, "mail.store": store }) {
        busService.subscribe("website_livechat.send_chat_request", (payload) => {
            const { Thread } = store.insert(payload);
            for (const thread of Thread) {
                const chatWindow = store.ChatWindow.insert({ thread });
                chatWindow.open();
                chatWindow.focus();
            }
        });
    },
};

registry.category("services").add("website_livechat.notifications", websiteLivechatNotifications);

```

## File: static\src\web\@types\models.d.ts

```ts
declare module "models" {
    export interface Thread {
        visitor: Persona,
        visitorPartner: Persona,
    }
}

```

## File: views\im_livechat_channel_add.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<record id="im_livechat_channel_view_form_add" model="ir.ui.view">
    <field name="name">im_livechat.channel.view.form.add</field>
    <field name="model">im_livechat.channel</field>
    <field name="arch" type="xml">
        <form js_class="website_new_content_form">
            <group>
                <field name="website_url" invisible="1"/>
                <field name="name" string="Channel Name"/>
            </group>
        </form>
    </field>
</record>

<record id="im_livechat_channel_action_add" model="ir.actions.act_window">
    <field name="name">New Channel</field>
    <field name="res_model">im_livechat.channel</field>
    <field name="view_mode">form</field>
    <field name="target">new</field>
    <field name="view_id" ref="im_livechat_channel_view_form_add"/>
</record>

</odoo>

```

## File: views\im_livechat_chatbot_script_view.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="chatbot_script_view_form" model="ir.ui.view">
        <field name="name">chatbot.script.view.form.inherit.website.livechat</field>
        <field name="model">chatbot.script</field>
        <field name="inherit_id" ref="im_livechat.chatbot_script_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <button name="action_test_script" type="object" class="btn btn-secondary" string="Test"
                    invisible="not active or not script_step_ids"/>
            </xpath>
        </field>
    </record>

</data></odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.livechat</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <setting id="livechat" position="inside">
                <div class="content-group mt16">
                    <div class="row">
                        <label class="col-lg-3 o_light_label" string="Channel" for="channel_id"/>
                        <field name="channel_id" class="oe_inline"/>
                    </div>
                </div>
            </setting>
        </field>
    </record>
</odoo>

```

## File: views\website_livechat.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!--
            Integrate Livechat in Common Frontend for Website
            Template registering all the assets required to execute the Livechat from a page containing Odoo
        -->
        <template id="loader" inherit_id="website.layout" name="Livechat : include loader on Website">
            <xpath expr="//head">
                <t t-if="not no_livechat and website and website.channel_id">
                    <script>
                        <t t-call="im_livechat.loader" t-nocache="Should be up-to-date with available operators">
                            <t t-set="info" t-value="website._get_livechat_channel_info()"/>
                        </t>
                    </script>
                </t>
            </xpath>
        </template>

        <!-- Page Layout -->
        <template id="channel_page" name="Livechat Channel Satisfaction Page">
            <t t-call="website.layout">
              <div id="wrap">
                <div class="container">
                    <h1><span>Livechat Channel</span>  <small t-field="channel.name" /></h1>
                    <div t-field="channel.website_description" class="oe_structure mt16" />
                    <div class="row mt32">
                        <div class="col-lg-8">
                            <t t-if="len(ratings) &gt; 0">
                                <div class="row">
                                    <div class="col-lg-12 mb32">
                                        <h3>Statistics</h3>
                                        <div class="row">
                                            <div class="col-lg-4 d-flex justify-content-end flex-column">
                                                <div class="card bg-success text-white" t-attf-style="height: #{160 + int(percentage['great'])}px;">
                                                    <div class="card-body text-center">
                                                        <img src="/rating/static/src/img/rating_5.png" style="height:40px" alt="Happy face"/>
                                                    </div>
                                                    <div class="card-body text-center">
                                                        <h2 style="margin: 0">
                                                            <b>
                                                                <t t-esc="percentage['great']" />
                                                            </b>
                                                            <small>%</small>
                                                        </h2>
                                                    </div>
                                                </div>
                                            </div>
                                            <div class="col-lg-4 d-flex justify-content-end flex-column">
                                                <div class="card bg-warning text-white" t-attf-style="height: #{160 + int(percentage['okay'])}px;">
                                                    <div class="card-body text-center">
                                                        <img src="/rating/static/src/img/rating_3.png" style="height:40px" alt="Neutral face"/>
                                                    </div>
                                                    <div class="card-body text-center">
                                                        <h2 style="margin: 0">
                                                            <b>
                                                                <t t-esc="percentage['okay']" />
                                                            </b>
                                                            <small>%</small>
                                                        </h2>
                                                    </div>
                                                </div>
                                            </div>
                                            <div class="col-lg-4 d-flex justify-content-end flex-column">
                                                <div class="card bg-danger text-white" t-attf-style="height: #{160 + int(percentage['bad'])}px;">
                                                    <div class="card-body text-center">
                                                        <img src="/rating/static/src/img/rating_1.png" style="height:40px" alt="Sad face"/>
                                                    </div>
                                                    <div class="card-body text-center">
                                                        <h2 style="margin: 0">
                                                            <b>
                                                                <t t-esc="percentage['bad']" />
                                                            </b>
                                                            <small>%</small>
                                                        </h2>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                                <div class="row">
                                    <div class="col-lg-12 mb32">
                                        <h3>The <t t-esc="len(ratings)"/> last feedbacks</h3>
                                        <div>
                                            <t t-foreach="ratings" t-as="rating">
                                                <img t-attf-src='/rating/static/src/img/rating_#{int(rating.rating)}.png' t-att-alt="rating.res_name" width="48px" height="48px"/>
                                                <t t-if="(rating_index+1) % 5 == 0">
                                                    <br/>
                                                </t>
                                            </t>
                                        </div>
                                    </div>
                                </div>
                            </t>
                            <t t-if="len(ratings) == 0">
                                <h4 style="text-align:center">There are no ratings for this channel for now.</h4>
                            </t>
                        </div>
                        <div class="col-lg-4 mb32">
                            <h3>The Team</h3>
                            <t t-foreach="team" t-as="user">
                                <div class="d-flex mt-3">
                                    <img t-att-src="image_data_uri(user.avatar_128)" class="o_image_64_cover rounded me-2" t-att-alt="user.livechat_username or user.name"/>
                                    <div class="flex-grow-1">
                                        <h5>
                                            <t t-if="user.livechat_username">
                                                <t t-esc="user.livechat_username"/>
                                            </t>
                                            <t t-else="">
                                                <t t-esc="user.name"/>
                                            </t>
                                        </h5>
                                        <div class="col-lg-12">
                                            <div class="row gx-0">
                                                <t t-if="user.partner_id.id in ratings_per_user">
                                                    <div class="col-lg-4 ps-0 pe-0">
                                                        <img t-attf-src='/rating/static/src/img/rating_5.png' alt="Great" width="16px" height="16px"/>
                                                        <span class="align-middle"><t t-esc="ratings_per_user[user.partner_id.id]['great']"/>%</span>
                                                    </div>
                                                    <div class="col-lg-4 ps-0 pe-0">
                                                        <img t-attf-src='/rating/static/src/img/rating_3.png' alt="Okay" width="16px" height="16px"/>
                                                        <span class="align-middle"><t t-esc="ratings_per_user[user.partner_id.id]['okay']"/>%</span>
                                                    </div>
                                                    <div class="col-lg-4 ps-0 pe-0">
                                                        <img t-attf-src='/rating/static/src/img/rating_1.png' alt="Bad" width="16px" height="16px"/>
                                                        <span class="align-middle"><t t-esc="ratings_per_user[user.partner_id.id]['bad']"/>%</span>
                                                    </div>
                                                </t>
                                                <t t-else="">
                                                    <div class="col-lg-12 ps-0 pe-0 opacity-50">Not rated yet</div>
                                                </t>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </t>
                        </div>
                    </div>
                </div>
              </div>
            </t>
        </template>


        <template id="channel_list_page" name="Livechat Channel List Page">
            <t t-call="website.layout">
                <div id="wrap">
                    <div class="oe_structure" id="oe_structure_website_livechat_channel_list_1"/>
                    <div class="container">
                        <h1 class="pt-3">Livechat Support Channels</h1>
                        <div class="row mt32 mb32">
                            <t t-if="not len(channels)">
                                <div class="col-lg-6 offset-lg-3">
                                    There are no public livechat channels to show.
                                </div>
                            </t>
                            <t t-if="len(channels)">
                                <div class="col-lg-6">
                                    <t t-foreach="channels" t-as="channel">
                                        <div t-attf-class="d-flex#{' mt-3' if channel_index else ''}">
                                            <a t-attf-href="/livechat/channel/#{ slug(channel)}">
                                                <img t-att-src="channel.image_128 and image_data_uri(channel.image_128) or '/web/static/img/placeholder.png'" t-att-alt="channel.name" class="o_image_64_cover"/>
                                            </a>
                                            <div class="flex-grow-1 h-100 my-auto">
                                                <h4><t t-esc="channel.name"/></h4>
                                            </div>
                                        </div>
                                    </t>
                                </div>
                            </t>
                        </div>
                    </div>
                    <div class="oe_structure" id="oe_structure_website_livechat_channel_list_2"/>
                </div>
            </t>
        </template>
    </data>
</odoo>

```

## File: views\website_livechat_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="im_livechat_channel_form_view" model="ir.ui.view">
            <field name="name">im_livechat.channel.form.inherit.website_livechat</field>
            <field name="model">im_livechat.channel</field>
            <field name="inherit_id" ref="im_livechat.im_livechat_channel_view_form"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <field name="is_published" widget="website_redirect_button"/>
                </div>
            </field>
        </record>

    </data>
</odoo>

```

## File: views\website_visitor_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="website_visitor_livechat_session_action" model="ir.actions.act_window">
        <field name="name">Visitor's Sessions</field>
        <field name="res_model">discuss.channel</field>
        <field name="view_mode">list,form</field>
        <field name="view_id" ref="im_livechat.discuss_channel_view_tree"/>
        <field name="domain">[('livechat_visitor_id', '=', active_id), ('has_message', '=', True)]</field>
        <field name="context">{
                'search_default_livechat_visitor_id': [active_id],
                'default_livechat_visitor_id': active_id,
            }</field>
    </record>

    <record id="website_visitor_livechat_session_action_tree" model="ir.actions.act_window.view">
        <field name="sequence">1</field>
        <field name="view_mode">list</field>
        <field name="view_id" ref="im_livechat.discuss_channel_view_tree"/>
        <field name="act_window_id" ref="website_livechat.website_visitor_livechat_session_action"/>
    </record>

    <record id="website_visitor_livechat_session_action_form" model="ir.actions.act_window.view">
        <field name="sequence">2</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="im_livechat.discuss_channel_view_form"/>
        <field name="act_window_id" ref="website_livechat.website_visitor_livechat_session_action"/>
    </record>

    <!-- website visitor views -->
    <record id="website_visitor_view_kanban" model="ir.ui.view">
        <field name="name">website.visitor.view.kanban.inherit.website.livechat</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_kanban"/>
        <field name="arch" type="xml">
            <field name="country_id" position="after">
                <field name="livechat_operator_id"/>
                <field name="session_count"/>
            </field>
            <xpath expr="//div[@name='w_visitor_kanban_actions']" position="before">
                <div t-if="record.session_count.raw_value">Chats<field name="session_count" class="float-end fw-bold"/></div>
                <div t-if="record.livechat_operator_id.raw_value">
                    Speaking With
                    <div name="livechat_operator_id"
                        class="fw-bold float-end d-inline-block">
                        <field name="livechat_operator_id" widget="image" options="{'preview_image': 'avatar_128', 'size': [19, 19]}" class="rounded-circle" alt="Operator Avatar"/>
                        <field name="livechat_operator_name"/>
                    </div>
                </div>
                <t t-else="">
                    <br invisible="livechat_operator_id or not is_connected"/>
                </t>
            </xpath>
            <xpath expr="//div[hasclass('w_visitor_kanban_actions_ungrouped')]" position="before">
                <div class="col-lg col-sm-4 col-6 py-0 my-2">
                    <span t-att-class="record.session_count.raw_value ? 'fw-bold' : 'text-muted'">
                        <field name="session_count"/>
                    </span>
                    <div t-att-class="record.session_count.raw_value ? '' : 'text-muted'">Chats</div>
                </div>
                <div t-if="record.livechat_operator_id.raw_value" class="col-lg col-sm-4 col-6 py-0 my-2">
                    <field name="livechat_operator_id" widget="image" options="{'preview_image': 'avatar_128', 'size': [19, 19]}" class="rounded-circle" alt="Operator Avatar"/>
                    <field name="livechat_operator_name" class="fw-bold"/>
                    <div>Speaking With</div>
                </div>
                <div t-else="" class="col-lg d-none d-lg-block"/>
            </xpath>
            <xpath expr="//div[@name='w_visitor_kanban_actions']" position="inside">
                <button name="action_send_chat_request" type="object"
                        class="btn btn-secondary"
                        invisible="livechat_operator_id or not is_connected">
                        Chat
                </button>
            </xpath>
            <xpath expr="//div[hasclass('w_visitor_kanban_actions_ungrouped')]" position="inside">
                <button name="action_send_chat_request" type="object"
                        class="btn btn-secondary"
                        invisible="livechat_operator_id or not is_connected">
                        Chat
                </button>
            </xpath>
        </field>
    </record>

    <record id="website_visitor_view_form" model="ir.ui.view">
        <field name="name">website.visitor.view.form.inherit.website.livechat</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <button name="action_send_chat_request" string="Send chat request" type="object" class="oe_highlight"
                invisible="livechat_operator_id or not is_connected"/>
            </xpath>
            <xpath expr="//group[@id='general_info']" position="before">
                <group id="livechat_info" invisible="not livechat_operator_id">
                    <field name="livechat_operator_id" widget="many2one_avatar"/>
                </group>
            </xpath>
            <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                <button name="%(website_visitor_livechat_session_action)d" type="action" class="oe_stat_button" icon="fa-comment"
                        invisible="session_count == 0">
                    <field name="session_count" widget="statinfo" string="Chats"/>
                </button>
            </xpath>
        </field>
    </record>

    <record id="website_visitor_view_tree" model="ir.ui.view">
        <field name="name">website.visitor.view.list.inherit.website.livechat</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='email']" position="after">
                <field name="livechat_operator_id" optional="hide"/>
                <button name="action_send_chat_request" string="Chat" type="object" icon="fa-comments"
                        invisible="livechat_operator_id or not is_connected"/>
            </xpath>
        </field>
    </record>

    <record id="website_visitor_view_search" model="ir.ui.view">
        <field name="name">website.visitor.view.search.website.livechat</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='filter_is_connected']" position="after">
                <separator/>
                <filter string="Available" name="filter_not_in_conversation" domain="[('livechat_operator_id', '=', False)]"/>
                <filter string="In Conversation" name="filter_in_conversation" domain="[('livechat_operator_id', '!=', False)]"/>
            </xpath>
        </field>
    </record>

    <record id="website_livechat_send_chat_request_action_server" model="ir.actions.server">
        <field name="name">Send Chat Requests</field>
        <field name="model_id" ref="model_website_visitor"/>
        <field name="binding_model_id" ref="model_website_visitor"/>
        <field name="binding_view_types">list</field>
        <field name="state">code</field>
        <field name="code">
            if records:
                action = records.action_send_chat_request()
        </field>
    </record>

    <menuitem
        id="website_livechat_visitor_menu"
        name="Visitors"
        parent="im_livechat.menu_livechat_root"
        action="website.website_visitors_action"
        groups="im_livechat.im_livechat_group_user"
        sequence="15"/>
</data></odoo>

```

