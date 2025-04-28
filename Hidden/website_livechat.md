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
    'application': False,
    'auto_install': True,
    'data': [
        'views/website_livechat.xml',
        'views/res_config_settings_views.xml',
        'views/website_livechat_view.xml',
        'views/website_visitor_views.xml',
        'security/ir.model.access.csv',
        'security/website_livechat.xml',
        'data/website_livechat_data.xml',
    ],
    'qweb': [
        'static/src/xml/thread.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http, _
from odoo.http import request
from odoo.addons.im_livechat.controllers.main import LivechatController


class WebsiteLivechat(LivechatController):

    @http.route('/livechat', type='http', auth="public", website=True)
    def channel_list(self, **kw):
        # display the list of the channel
        channels = request.env['im_livechat.channel'].search([('website_published', '=', True)])
        values = {
            'channels': channels
        }
        return request.render('website_livechat.channel_list_page', values)


    @http.route('/livechat/channel/<model("im_livechat.channel"):channel>', type='http', auth='public', website=True)
    def channel_rating(self, channel, **kw):
        # get the last 100 ratings and the repartition per grade
        domain = [
            ('res_model', '=', 'mail.channel'), ('res_id', 'in', channel.sudo().channel_ids.ids),
            ('consumed', '=', True), ('rating', '>=', 1),
        ]
        ratings = request.env['rating.rating'].search(domain, order='create_date desc', limit=100)
        repartition = channel.sudo().channel_ids.rating_get_grades(domain=domain)

        # compute percentage
        percentage = dict.fromkeys(['great', 'okay', 'bad'], 0)
        for grade in repartition:
            percentage[grade] = round(repartition[grade] * 100.0 / sum(repartition.values()), 1) if sum(repartition.values()) else 0

        # filter only on the team users that worked on the last 100 ratings and get their detailed stat
        ratings_per_partner = {partner_id: dict(great=0, okay=0, bad=0)
                               for partner_id in ratings.mapped('rated_partner_id.id')}
        total_ratings_per_partner = dict.fromkeys(ratings.mapped('rated_partner_id.id'), 0)
        rating_texts = {10: 'great', 5: 'okay', 1: 'bad'}

        for rating in ratings:
            partner_id = rating.rated_partner_id.id
            ratings_per_partner[partner_id][rating_texts[rating.rating]] += 1
            total_ratings_per_partner[partner_id] += 1

        for partner_id, rating in ratings_per_partner.items():
            for k, v in ratings_per_partner[partner_id].items():
                ratings_per_partner[partner_id][k] = round(100 * v / total_ratings_per_partner[partner_id], 1)

        # the value dict to render the template
        values = {
            'channel': channel,
            'ratings': ratings,
            'team': channel.sudo().user_ids,
            'percentage': percentage,
            'ratings_per_user': ratings_per_partner
        }
        return request.render("website_livechat.channel_page", values)

    @http.route('/im_livechat/get_session', type="json", auth='public', cors="*")
    def get_session(self, channel_id, anonymous_name, previous_operator_id=None, **kwargs):
        """ Override to use visitor name instead of 'Visitor' whenever a visitor start a livechat session. """
        visitor_sudo = request.env['website.visitor']._get_visitor_from_request()
        if visitor_sudo:
            anonymous_name = visitor_sudo.with_context(lang=visitor_sudo.lang_id.code).display_name
        return super(WebsiteLivechat, self).get_session(channel_id, anonymous_name, previous_operator_id=previous_operator_id, **kwargs)

    @http.route('/im_livechat/visitor_leave_session', type='json', auth="public")
    def visitor_leave_session(self, uuid):
        """ Called when the livechat visitor leaves the conversation.
         This will clean the chat request and warn the operator that the conversation is over.
         This allows also to re-send a new chat request to the visitor, as while the visitor is
         in conversation with an operator, it's not possible to send the visitor a chat request."""
        mail_channel = request.env['mail.channel'].sudo().search([('uuid', '=', uuid)])
        if mail_channel:
            mail_channel.close_livechat_request_session()

    @http.route('/im_livechat/close_empty_livechat', type='json', auth="public")
    def close_empty_livechat(self, uuid):
        """ Called when an operator send a chat request to a visitor but does not speak to him and closes
        the chatter. (when the operator does not complete the 'send chat request' flow in other terms)
        This will clean the chat request and allows operators to send the visitor a new chat request."""
        mail_channel = request.env['mail.channel'].sudo().search([('uuid', '=', uuid)])
        if mail_channel:
            mail_channel.channel_pin(uuid, False)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
from . import main

```

## File: data\website_livechat_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
     <data noupdate="1">
         <record id="website.default_website" model="website">
             <field name="channel_id" ref="im_livechat.im_livechat_channel_data"></field>
         </record>

         <record id="menu_livechat" model="website.menu">
            <field name="name">Live Support</field>
            <field name="url">/livechat</field>
            <field name="parent_id" ref="website.main_menu"/>
            <field name="sequence">55</field>
        </record>
     </data>
</odoo>

```

## File: models\im_livechat.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields
from odoo.addons.http_routing.models.ir_http import slug
from odoo.tools.translate import html_translate


class ImLivechatChannel(models.Model):

    _name = 'im_livechat.channel'
    _inherit = ['im_livechat.channel', 'website.published.mixin']

    def _compute_website_url(self):
        super(ImLivechatChannel, self)._compute_website_url()
        for channel in self:
            channel.website_url = "/livechat/channel/%s" % (slug(channel),)

    website_description = fields.Html("Website description", default=False, help="Description of the channel displayed on the website page", sanitize_attributes=False, translate=html_translate)

```

## File: models\im_livechat_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _


class ImLivechatChannel(models.Model):
    _inherit = 'im_livechat.channel'

    def _get_livechat_mail_channel_vals(self, anonymous_name, operator, user_id=None, country_id=None):
        mail_channel_vals = super(ImLivechatChannel, self)._get_livechat_mail_channel_vals(anonymous_name, operator, user_id=user_id, country_id=country_id)
        visitor_sudo = self.env['website.visitor']._get_visitor_from_request()
        if visitor_sudo:
            mail_channel_vals.update({
                'livechat_visitor_id': visitor_sudo.id,
                'livechat_active': True
            })
            if not user_id:
                mail_channel_vals['anonymous_name'] = visitor_sudo.display_name + (' (%s)' % visitor_sudo.country_id.name if visitor_sudo.country_id else '')
            # As chat requested by the visitor, delete the chat requested by an operator if any to avoid conflicts between two flows
            chat_request_channel = self.env['mail.channel'].sudo().search([('livechat_visitor_id', '=', visitor_sudo.id), ('livechat_active', '=', True)])
            for mail_channel in chat_request_channel:
                mail_channel.close_livechat_request_session(type='cancel', speaking_with=operator.name)

        return mail_channel_vals

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

## File: models\mail_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class MailChannel(models.Model):
    _inherit = 'mail.channel'

    livechat_visitor_id = fields.Many2one('website.visitor', string='Visitor')
    livechat_active = fields.Boolean('Is livechat ongoing?', help='Livechat session is not considered as active if the visitor left the conversation.')

    def _execute_channel_pin(self, pinned=False):
        """ Override to clean an empty livechat channel.
         This is typically called when the operator send a chat request to a website.visitor
         but don't speak to him and closes the chatter.
         This allows operators to send the visitor a new chat request.
         If active empty livechat channel,
         delete mail_channel as not useful to keep empty chat
         """
        super(MailChannel, self)._execute_channel_pin(pinned)
        if self.livechat_active and not self.channel_message_ids:
            self.unlink()

    def channel_info(self, extra_info=False):
        """
        Override to add visitor information on the mail channel infos.
        This will be used to display a banner with visitor informations
        at the top of the livechat channel discussion view in discuss module.
        """
        channel_infos = super(MailChannel, self).channel_info(extra_info)
        channel_infos_dict = dict((c['id'], c) for c in channel_infos)
        for channel in self:
            visitor = channel.livechat_visitor_id
            if visitor:
                channel_infos_dict[channel.id]['visitor'] = {
                    'name': visitor.display_name,
                    'country_code': visitor.country_id.code.lower() if visitor.country_id else False,
                    'is_connected': visitor.is_connected,
                    'history': self._get_visitor_history(visitor),
                    'website': visitor.website_id.name,
                    'lang': visitor.lang_id.name,
                    'partner_id': visitor.partner_id.id,
                }
        return list(channel_infos_dict.values())

    def _get_visitor_history(self, visitor):
        """
        Prepare history string to render it in the visitor info div on discuss livechat channel view.
        :param visitor: website.visitor of the channel
        :return: arrow separated string containing navigation history information
        """
        recent_history = self.env['website.track'].search([('page_id', '!=', False), ('visitor_id', '=', visitor.id)], limit=3)
        return ' → '.join(visit.page_id.name + ' (' + visit.visit_datetime.strftime('%H:%M') + ')' for visit in reversed(recent_history))

    def close_livechat_request_session(self, type='leave', **kwargs):
        """ Set deactivate the livechat channel and notify (the operator) the reason of closing the session."""
        self.ensure_one()
        if self.livechat_active:
            self.livechat_active = False
            # avoid useless notification if the channel is empty
            if not self.channel_message_ids:
                return
            # Notify that the visitor has left the conversation
            name = _('The visitor') if not self.livechat_visitor_id else self.livechat_visitor_id.display_name
            if type == 'cancel':
                message = _('has started a conversation with %s. The chat request has been canceled.') % kwargs.get('speaking_with', 'an operator')
            else:
                message = _('has left the conversation.')
            leave_message = '%s %s' % (name, message)
            self.message_post(author_id=self.env.ref('base.user_root').sudo().partner_id.id,
                              body=leave_message, message_type='comment', subtype='mt_comment')

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, **kwargs):
        """Override to mark the visitor as still connected.
        If the message sent is not from the operator (so if it's the visitor or
        odoobot sending closing chat notification, the visitor last action date is updated."""
        message = super(MailChannel, self).message_post(**kwargs)
        message_author_id = message.author_id
        visitor = self.livechat_visitor_id
        if len(self) == 1 and visitor and message_author_id != self.livechat_operator_id:
            visitor._update_visitor_last_visit()
        return message

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

from odoo import api, fields, models


class Website(models.Model):

    _inherit = "website"

    channel_id = fields.Many2one('im_livechat.channel', string='Website Live Chat Channel')

    def get_livechat_channel_info(self):
        """ Get the livechat info dict (button text, channel name, ...) for the livechat channel of
            the current website.
        """
        self.ensure_one()
        if self.channel_id:
            return self.channel_id.sudo().get_livechat_info()
        return {}

```

## File: models\website_visitor.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, timedelta
import json

from odoo import models, api, fields, _
from odoo.exceptions import UserError


class WebsiteVisitor(models.Model):
    _inherit = 'website.visitor'

    livechat_operator_id = fields.Many2one('res.partner', compute='_compute_livechat_operator_id', store=True, string='Speaking with')
    livechat_operator_name = fields.Char('Operator Name', related="livechat_operator_id.name")
    mail_channel_ids = fields.One2many('mail.channel', 'livechat_visitor_id',
                                       string="Visitor's livechat channels", readonly=True)
    session_count = fields.Integer('# Sessions', compute="_compute_session_count")

    @api.depends('mail_channel_ids.livechat_active', 'mail_channel_ids.livechat_operator_id')
    def _compute_livechat_operator_id(self):
        results = self.env['mail.channel'].search_read(
            [('livechat_visitor_id', 'in', self.ids), ('livechat_active', '=', True)],
            ['livechat_visitor_id', 'livechat_operator_id']
        )
        visitor_operator_map = {int(result['livechat_visitor_id'][0]): int(result['livechat_operator_id'][0]) for result in results}
        for visitor in self:
            visitor.livechat_operator_id = visitor_operator_map.get(visitor.id, False)

    @api.depends('mail_channel_ids')
    def _compute_session_count(self):
        sessions = self.env['mail.channel'].read_group([('livechat_visitor_id', 'in', self.ids)], ['livechat_visitor_id'], ['livechat_visitor_id'])
        sessions_count = {session['livechat_visitor_id'][0]: session['livechat_visitor_id_count'] for session in sessions}
        for visitor in self:
            visitor.session_count = sessions_count.get(visitor.id, 0)

    def action_send_chat_request(self):
        """ Send a chat request to website_visitor(s).
        This creates a chat_request and a mail_channel with livechat active flag.
        But for the visitor to get the chat request, the operator still has to speak to the visitor.
        The visitor will receive the chat request the next time he navigates to a website page.
        (see _handle_webpage_dispatch for next step)"""
        # check if visitor is available
        unavailable_visitors_count = self.env['mail.channel'].search_count([('livechat_visitor_id', 'in', self.ids), ('livechat_active', '=', True)])
        if unavailable_visitors_count:
            raise UserError(_('Recipients are not available. Please refresh the page to get latest visitors status.'))
        # check if user is available as operator
        for website in self.mapped('website_id'):
            if not website.channel_id:
                raise UserError(_('No Livechat Channel allows you to send a chat request for website %s.' % website.name))
        self.website_id.channel_id.write({'user_ids': [(4, self.env.user.id)]})
        # Create chat_requests and linked mail_channels
        mail_channel_vals_list = []
        for visitor in self:
            operator = self.env.user
            country = visitor.country_id
            visitor_name = "%s (%s)" % (visitor.display_name, country.name) if country else visitor.display_name
            mail_channel_vals_list.append({
                'channel_partner_ids':  [(4, operator.partner_id.id)],
                'livechat_channel_id': visitor.website_id.channel_id.id,
                'livechat_operator_id': self.env.user.partner_id.id,
                'channel_type': 'livechat',
                'public': 'private',
                'email_send': False,
                'country_id': country.id,
                'anonymous_name': visitor_name,
                'name': ', '.join([visitor_name, operator.livechat_username if operator.livechat_username else operator.name]),
                'livechat_visitor_id': visitor.id,
                'livechat_active': True,
            })
        if mail_channel_vals_list:
            mail_channels = self.env['mail.channel'].create(mail_channel_vals_list)
            # Open empty chatter to allow the operator to start chatting with the visitor.
            mail_channels_info = mail_channels.channel_info('channel_minimize')
            for mail_channel_info in mail_channels_info:
                self.env['bus.bus'].sendone((self._cr.dbname, 'res.partner', operator.partner_id.id), mail_channel_info)

    def _handle_website_page_visit(self, response, website_page, visitor_sudo):
        """ Called when the visitor navigates to a website page.
         This checks if there is a chat request for the visitor.
         It will set the livechat session cookie of the visitor with the mail channel information
         to make the usual livechat mechanism do the rest.
         (opening the chatter if a livechat session exist for the visitor)
         This will only happen if the mail channel linked to the chat request already has a message.
         So that empty livechat channel won't pop up at client side. """
        super(WebsiteVisitor, self)._handle_website_page_visit(response, website_page, visitor_sudo)
        visitor_id = visitor_sudo.id or self.env['website.visitor']._get_visitor_from_request().id
        if visitor_id:
            # get active chat_request linked to visitor
            chat_request_channel = self.env['mail.channel'].sudo().search([('livechat_visitor_id', '=', visitor_id), ('livechat_active', '=', True)], order='create_date desc', limit=1)
            if chat_request_channel and chat_request_channel.channel_message_ids:
                livechat_session = json.dumps({
                    "folded": False,
                    "id": chat_request_channel.id,
                    "message_unread_counter": 0,
                    "operator_pid": [
                        chat_request_channel.livechat_operator_id.id,
                        chat_request_channel.livechat_operator_id.display_name
                    ],
                    "name": chat_request_channel.name,
                    "uuid": chat_request_channel.uuid,
                    "type": "chat_request"
                })
                expiration_date = datetime.now() + timedelta(days=100 * 365)  # never expire
                response.set_cookie('im_livechat_session', livechat_session, expires=expiration_date.timestamp())

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import im_livechat
from . import im_livechat_channel
from . import ir_http
from . import mail_channel
from . import res_config_settings
from . import website
from . import website_visitor

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_im_livechat_channel_public,im_livechat.channel.public,im_livechat.model_im_livechat_channel,,1,0,0,0
access_website_visitor_livechat_users,website.visitor.livechat.users,model_website_visitor,im_livechat.im_livechat_group_user,1,1,0,0
access_website_track_livechat_users,website.track.livechat.users,website.model_website_track,im_livechat.im_livechat_group_user,1,0,0,0

```

## File: security\website_livechat.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

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

    </data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#CD7690"/><stop offset="100%" stop-color="#CA5377"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M42.103 69H4c-2 0-4-.145-4-4.07V41.395L20 19.14C23.333 15.07 28.333 11 35 11s11.667 7.801 17 17.298v8.14h2V48.65l-2.635 1.09L51 54.754 42.103 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M35.096 13C25.103 13 18 21.103 18 31.096V35c-1.333 0-2 .667-2 2v8.171C16 48.51 18.662 51 22 51h2c1.333 0 2-.667 2-2V37c0-1.333-.667-2-2-2h-2.979v-3.904c0-7.781 6.294-14.075 14.075-14.075C42.878 17.021 49 23.315 49 31.096V35h-3c-1.333 0-2 .667-2 2v12c0 1.333.667 2 2 2h3c.667 1.333.667 2.667 0 4h-9c0-.667-.328-1-.984-1H36c-.667 0-1 .333-1 1v1c.064.667.398 1 1 1h13c2 0 3-2 3-6 2 0 3-2 3-6v-8c0-1.333-1-2-3-2v-3.904C52 21.103 45.09 13 35.096 13z" opacity=".3"/><path fill="#FFF" d="M35.096 11C25.103 11 18 19.103 18 29.096V33c-1.333 0-2 .667-2 2v8.171C16 46.51 18.662 49 22 49h2c1.333 0 2-.667 2-2V35c0-1.333-.667-2-2-2h-2.979v-3.904c0-7.781 6.294-14.075 14.075-14.075C42.878 15.021 49 21.315 49 29.096V33h-3c-1.333 0-2 .667-2 2v12c0 1.333.667 2 2 2h3c.667 1.333.667 2.667 0 4h-9c0-.667-.328-1-.984-1H36c-.667 0-1 .333-1 1v1c.064.667.398 1 1 1h13c2 0 3-2 3-6 2 0 3-2 3-6v-8c0-1.333-1-2-3-2v-3.904C52 19.103 45.09 11 35.096 11z"/></g></g></svg>
```

## File: static\src\js\channel.js

```javascript
odoo.define('website.livechat.mail.model.Channel', function (require) {
"use strict";

var Channel = require('mail.model.Channel');
var session = require('web.session');

/**
 * This class represent channels in JS. In this context, the word channel
 * has the same meaning of channel on the server, meaning that direct messages
 * (DM) and livechats are also channels.
 *
 * Any piece of code in JS that make use of channels must ideally interact with
 * such objects, instead of direct data from the server.
 */
Channel.include({
    /**
     * @override
     * Add the visitor if is set on the channel
     * @param {Object} params
     * @param {Object} params.data
     * @param {string} params.data.visitor
     */
    init: function (params) {
        var self = this;
        var data = params.data;
        this._visitor = data.visitor;
        this._super.apply(this, arguments);
    },

});

return Channel;

});

```

## File: static\src\js\im_livechat.js

```javascript
odoo.define('website_livechat.livechat_request', function (require) {
"use strict";

var utils = require('web.utils');
var session = require('web.session');
var LivechatButton = require('im_livechat.im_livechat').LivechatButton;


LivechatButton.include({

    /**
     * @override
     * This will will correctly format the livechat session cookie
     * that comes from server side (and that is not properly formatted)
     * This is used for chat request mechanism, when an operator send a chat request
     * from backend to a website visitor.
     */
    willStart: function () {
        var self = this;
        var cookie = utils.get_cookie('im_livechat_session');
        var ready;
        if (cookie) {
            var cleanedLivechatSessionCookie = this.decode_server_cookie(cookie);
            utils.set_cookie('im_livechat_session', cleanedLivechatSessionCookie, 60*60);
        }
        return this._super();
    },

    /**
     * @override
     * Called when the visitor closes the livechat chatter
     * (no matter the way : close, send feedback, ..)
     * this will deactivate the mail_channel, clean the chat request if any
     * and allow the operators to send the visitor a new chat request
     */
    _closeChat: function () {
        var self = this;
        var cookie = utils.get_cookie('im_livechat_session');
        if (cookie) {
            var channel = JSON.parse(cookie);
            var ready = session.rpc('/im_livechat/visitor_leave_session', {uuid: channel.uuid});
            ready.then(self._super());
        }
        else {
            this._super();
        }
    },

    /**
    * Utils to correctly re-encode json string sent by server.
    * Copied from StackOverflow.
    */
    decode_server_cookie: function (val) {
        if (val.indexOf('\\') === -1) {
            return val;  // not encoded
        }
        val = val.slice(1, -1).replace(/\\"/g, '"');
        val = val.replace(/\\(\d{3})/g, function(match, octal) {
            return String.fromCharCode(parseInt(octal, 8));
        });
        return val.replace(/\\\\/g, '\\');
    },
});

return {
    LivechatButton: LivechatButton,
};

});

```

## File: static\src\js\thread_window.js

```javascript
odoo.define('website_livechat.ThreadWindow', function (require) {
"use strict";

var ThreadWindow = require('mail.ThreadWindow');
var session = require('web.session');

/**
 * This is the main widget for rendering small windows for mail.model.Thread.
 * Almost all instances of this class are linked to a thread. The sole
 * exception is the "blank" thread window. This window let us open another
 * thread window, using this "blank" thread window.
 */
ThreadWindow.include({
    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    close: function () {
        var self = this;
        if (this.hasThread() && this._thread._type === "livechat" && this._threadWidget._messages.length == 0) {
            session.rpc('/im_livechat/close_empty_livechat', {uuid: this._thread._uuid});
        }
        else {
            this._super();
        }
    },
});

return ThreadWindow;

});

```

## File: static\src\js\website_livechat.editor.js

```javascript
odoo.define('website_livechat.editor', function (require) {
'use strict';

var core = require('web.core');
var wUtils = require('website.utils');
var WebsiteNewMenu = require('website.newMenu');

var _t = core._t;

WebsiteNewMenu.include({
    actions: _.extend({}, WebsiteNewMenu.prototype.actions || {}, {
        new_channel: '_createNewChannel',
    }),

    //--------------------------------------------------------------------------
    // Actions
    //--------------------------------------------------------------------------

    /**
     * Asks the user information about a new channel to create, then creates it
     * and redirects the user to this new channel.
     *
     * @private
     * @returns {Promise} Unresolved if there is a redirection
     */
    _createNewChannel: function () {
        var self = this;
        return wUtils.prompt({
            window_title: _t("New Channel"),
            input: _t("Name"),
        }).then(function (result) {
            var name = result.val;
            if (!name) {
                return;
            }
            return self._rpc({
                model: 'im_livechat.channel',
                method: 'create_and_get_website_url',
                args: [[]],
                kwargs: {
                    name: name,
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

## File: static\src\xml\thread.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-extend="mail.widget.Thread.Content.ASC">
        <t t-jquery=".o_mail_thread_content" t-operation="replace">
            <t t-call="mail.widget.Thread.website.visitor.banner"/>
            <div class="o_mail_thread_content o_sub_w_visitor_info">
                <t t-if="options.displayLoadMore" t-call="mail.widget.Thread.LoadMore"/>
                <t t-call="mail.widget.Thread.Messages"/>
                <t t-if="options.displayBottomThreadFreeSpace">
                    <div class="o_thread_bottom_free_space"/>
                </t>
            </div>
        </t>
    </t>

    <t t-extend="mail.widget.Thread.Content.DESC">
        <t t-jquery=".o_mail_thread_content" t-operation="replace">
            <t t-call="mail.widget.Thread.website.visitor.banner"/>
            <div class="o_mail_thread_content">
                <t t-if="options.messagesSeparatorPosition == 'top'" t-call="mail.MessagesSeparator"/>
                <t t-set="messages" t-value="messages.slice().reverse()"/>
                <t t-call="mail.widget.Thread.Messages"/>
                <t t-if="options.displayLoadMore" t-call="mail.widget.Thread.LoadMore"/>
            </div>
        </t>
    </t>

    <t t-name="mail.widget.Thread.website.visitor.banner">
        <t t-set="visitor" t-value="thread._visitor"/>
        <t t-if="visitor">
            <div class="o_w_visitor_info o_thread_message o_mail_discussion d-flex">
                <div class="o_thread_message_sidebar align-self-center">
                    <div class="o_thread_message_sidebar_image">
                        <img alt=""
                             t-att-src="visitor.partner_id ? '/web/image/res.partner/' + visitor.partner_id + '/image_128' : '/mail/static/src/img/smiley/avatar.jpg'"
                             class="o_thread_message_avatar rounded-circle"/>
                        <span>
                            <i t-if="visitor.is_connected" aria-label="Visitor is connected" class="o_w_visitor_status o_w_visitor_connected fa fa-circle" role="img" title="Online"></i>
                            <i t-else="" aria-label="Visitor is offline" class="o_w_visitor_status o_w_visitor_disconnected fa fa-circle" role="img" title="Offline"></i>
                        </span>
                    </div>
                </div>
                <div class="o_thread_message_core pt-2 align-self-center">
                    <div class="d-flex">
                        <img t-if="visitor.country_code" t-attf-src='base/static/img/country_flags/#{visitor.country_code}.png' class="o_country_flag mr-2 mb-1"/>
                        <strong class="o_thread_author" t-esc="visitor.name"/>
                        <span><i class="fa fa-comment-o ml-4 mr-2" aria-label="Lang"/><t t-esc="visitor.lang"/></span>
                        <span t-if="visitor.website"><i class="fa fa-globe ml-4 mr-2" aria-label="Website"/><span t-esc="visitor.website"/></span>
                    </div>
                    <div class="o_thread_message_content">
                        <div class="d-flex">
                            <span><i class="fa fa-history mr-2" aria-label="History"/></span>
                            <p t-esc="visitor.history"/>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </t>
</templates>

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
            <div id="google_maps_setting" position="after">
                <div class="col-12 col-lg-6 o_setting_box" id="live_chat_install_setting">
                    <div class="o_setting_right_pane">
                        <span class="o_form_label">Live Chat</span>
                        <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                        <div class="text-muted">
                            Live chat channel of your website
                        </div>
                        <div class="content-group mt16">
                            <div class="row">
                                <label class="col-lg-3 o_light_label" string="Channel" for="channel_id"/>
                                <field name="channel_id" class="oe_inline"/>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </field>
    </record>

</odoo>

```

## File: views\website_livechat.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="assets_backend" inherit_id="website.assets_backend" name="website_livechat assets backend">
            <xpath expr="." position="inside">
                <!-- Visitor Chat Request -->
                <script type="text/javascript" src="/website_livechat/static/src/js/thread_window.js"></script>
                <script type="text/javascript" src="/website_livechat/static/src/js/channel.js"></script>
                <link rel="stylesheet" type="text/scss" href="/website_livechat/static/src/scss/mail_thread_visitor_info.scss"/>
            </xpath>
        </template>

        <!--
            Integrate Livechat in Common Frontend for Website
            Template registering all the assets required to execute the Livechat from a page containing Odoo
        -->
        <template id="assets_frontend" name="im_livechat assets frontend" inherit_id="website.assets_frontend">
            <xpath expr="." position="inside">
                    <!-- thread window -->
                    <script type="text/javascript" src="/mail/static/src/js/thread_windows/abstract_thread_window.js"></script>
                    <script type="text/javascript" src="/im_livechat/static/src/js/website_livechat_window.js"></script>

                <script type="text/javascript" src="/bus/static/src/js/longpolling_bus.js"></script>
                <script type="text/javascript" src="/bus/static/src/js/crosstab_bus.js"></script>
                <script type="text/javascript" src="/bus/static/src/js/services/bus_service.js"></script>

                <script type="text/javascript" src="/mail/static/src/js/document_viewer.js"></script>
                <script type="text/javascript" src="/mail/static/src/js/thread_widget.js"></script>
                <script type="text/javascript" src="/mail/static/src/js/utils.js"></script>
                <script type="text/javascript" src="/im_livechat/static/src/js/im_livechat.js"></script>
                <!-- Models -->
                    <!-- threads -->
                    <script type="text/javascript" src="/mail/static/src/js/models/threads/abstract_thread.js"></script>
                    <script type="text/javascript" src="/im_livechat/static/src/js/models/website_livechat.js"></script>
                    <script type="text/javascript" src="/mail/static/src/js/models/threads/mixins/thread_typing_mixin.js"></script>
                    <!-- messages -->
                    <script type="text/javascript" src="/mail/static/src/js/models/messages/abstract_message.js"></script>
                    <script type="text/javascript" src="/im_livechat/static/src/js/models/website_livechat_message.js"></script>
                    <!-- utils -->
                    <script type="text/javascript" src="/mail/static/src/js/models/utils/timer.js"></script>
                    <script type="text/javascript" src="/mail/static/src/js/models/utils/timers.js"></script>
                    <script type="text/javascript" src="/mail/static/src/js/models/utils/cc_throttle_function.js"></script>
                <!--Chat Request-->
                <script type="text/javascript" src="/website_livechat/static/src/js/im_livechat.js"></script>
                <!--Stylesheets-->
                <link rel="stylesheet" type="text/scss" href="/mail/static/src/scss/abstract_thread_window.scss"></link>
                <link rel="stylesheet" type="text/scss" href="/mail/static/src/scss/thread.scss"></link>
                <link rel="stylesheet" type="text/scss" href="/im_livechat/static/src/scss/im_livechat.scss"/>
            </xpath>
        </template>

        <template id="assets_editor_inherit_website_livechat" inherit_id="website.assets_editor" name="website_livechat Assets Editor">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/website_livechat/static/src/js/website_livechat.editor.js"></script>
            </xpath>
        </template>

        <template id="loader" inherit_id="website.layout" name="Livechat : include loader on Website">
            <xpath expr="//div[@id='wrapwrap']" position="after">
                <t t-if="website and website.channel_id">
                    <script>
                        <t t-call="im_livechat.loader">
                            <t t-set="info" t-value="website.get_livechat_channel_info()"/>
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
                    <!-- published button -->
                    <t t-call="website.publish_management">
                        <t t-set="object" t-value="channel"/>
                        <t t-set="publish_edit" t-value="True"/>
                    </t>

                    <h1><span>Livechat Channel</span>  <small t-field="channel.name" /></h1>
                    <div t-field="channel.website_description" class="oe_structure mt16" />
                    <div class="row mt32">
                        <div class="col-lg-8">
                            <t t-if="len(ratings) &gt; 0">
                                <div class="row">
                                    <div class="col-lg-12 mb32">
                                        <h3>Statistics</h3>
                                        <div class="row">
                                            <div class="col-lg-5">
                                                <div class="card bg-success text-white" t-attf-style="height: #{160 + int(percentage['great'])}px;">
                                                    <div class="card-body text-center">
                                                        <img src="/rating/static/src/img/rating_10.png" style="height:40px" alt="Happy face"/>
                                                    </div>
                                                    <div class="card-body text-center">
                                                        <h2 style="margin: 0">
                                                            <b style="font-size: 30px">
                                                                <t t-esc="percentage['great']" />
                                                            </b>
                                                            <small>%</small>
                                                        </h2>
                                                    </div>
                                                </div>
                                            </div>
                                            <div class="col-lg-4">
                                                <div class="card bg-warning text-white" t-attf-style="height: #{160 + int(percentage['okay'])}px;">
                                                    <div class="card-body text-center">
                                                        <img src="/rating/static/src/img/rating_5.png" style="height:40px" alt="Neutral face"/>
                                                    </div>
                                                    <div class="card-body text-center">
                                                        <h2 style="margin: 0">
                                                            <b style="font-size: 30px">
                                                                <t t-esc="percentage['okay']" />
                                                            </b>
                                                            <small>%</small>
                                                        </h2>
                                                    </div>
                                                </div>
                                            </div>
                                            <div class="col-lg-3">
                                                <div class="card bg-danger text-white" t-attf-style="height: #{160 + int(percentage['bad'])}px;">
                                                    <div class="card-body text-center">
                                                        <img src="/rating/static/src/img/rating_1.png" style="height:40px" alt="Sad face"/>
                                                    </div>
                                                    <div class="card-body text-center">
                                                        <h2 style="margin: 0">
                                                            <b style="font-size: 30px">
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
                                        <div style="text-align:center">
                                            <t t-foreach="ratings" t-as="rating">
                                                <img t-attf-src='/rating/static/src/img/rating_#{int(rating.rating)}.png' t-att-alt="rating.res_name" width="48px" height="48px"/>
                                                <t t-if="(rating_index+1) % 10 == 0">
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
                                <div class="media mt-3">
                                    <img t-if="user.image_128" t-att-src="image_data_uri(user.image_128)" class="o_image_64_cover rounded o_livechat_operator_avatar" t-att-alt="user.livechat_username or user.name"/>
                                    <div class="media-body">
                                        <h5>
                                            <t t-if="user.livechat_username">
                                                <t t-esc="user.livechat_username"/>
                                            </t>
                                            <t t-else="">
                                                <t t-esc="user.name"/>
                                            </t>
                                        </h5>
                                        <div class="col-lg-12">
                                            <div class="row">
                                                <t t-if="user.partner_id.id in ratings_per_user">
                                                    <div class="col-lg-4 pl-0 pr-0">
                                                        <img t-attf-src='/rating/static/src/img/rating_10.png' alt="Great" width="16px" height="16px"/>
                                                        <span class="align-middle"><t t-esc="ratings_per_user[user.partner_id.id]['great']"/>%</span>
                                                    </div>
                                                    <div class="col-lg-4 pl-0 pr-0">
                                                        <img t-attf-src='/rating/static/src/img/rating_5.png' alt="Okay" width="16px" height="16px"/>
                                                        <span class="align-middle"><t t-esc="ratings_per_user[user.partner_id.id]['okay']"/>%</span>
                                                    </div>
                                                    <div class="col-lg-4 pl-0 pr-0">
                                                        <img t-attf-src='/rating/static/src/img/rating_1.png' alt="Bad" width="16px" height="16px"/>
                                                        <span class="align-middle"><t t-esc="ratings_per_user[user.partner_id.id]['bad']"/>%</span>
                                                    </div>
                                                </t>
                                                <t t-else="">
                                                    <div class="col-lg-12 pl-0 pr-0 o_livechat_no_rating">Not rated yet</div>
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
                        <h1>Livechat Support Channels</h1>
                        <div class="row mt32 mb32">
                            <t t-if="not len(channels)">
                                <div class="col-lg-6 offset-lg-3">
                                    There are no public livechat channels to show.
                                </div>
                            </t>
                            <t t-if="len(channels)">
                                <div class="col-lg-6 offset-lg-3">
                                    <t t-foreach="channels" t-as="channel">
                                        <div t-attf-class="media#{' mt-3' if channel_index else ''}">
                                            <a t-attf-href="/livechat/channel/#{ slug(channel)}">
                                                <img t-att-src="channel.image_128 and image_data_uri(channel.image_128) or '/web/static/src/img/placeholder.png'" t-att-alt="channel.name" class="o_image_64_cover"/>
                                            </a>
                                            <div class="media-body">
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

        <!-- User Navbar -->
        <template id="user_navbar_inherit_website_livechat" inherit_id="website.user_navbar">
            <xpath expr="//div[@id='o_new_content_menu_choices']//div[@name='module_website_livechat']" position="attributes">
                <attribute name="name"/>
                <attribute name="t-att-data-module-id"/>
                <attribute name="t-att-data-module-shortdesc"/>
                <attribute name="groups">im_livechat.im_livechat_group_manager</attribute>
            </xpath>
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
        <field name="res_model">mail.channel</field>
        <field name="view_mode">tree,form</field>
        <field name="view_id" ref="im_livechat.mail_channel_view_tree"/>
        <field name="domain">[('livechat_visitor_id', '=', active_id)]</field>
    </record>

    <record id="website_visitor_livechat_session_action_tree" model="ir.actions.act_window.view">
        <field name="sequence">1</field>
        <field name="view_mode">tree</field>
        <field name="view_id" ref="im_livechat.mail_channel_view_tree"/>
        <field name="act_window_id" ref="website_livechat.website_visitor_livechat_session_action"/>
    </record>

    <record id="website_visitor_livechat_session_action_form" model="ir.actions.act_window.view">
        <field name="sequence">2</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="im_livechat.mail_channel_view_form"/>
        <field name="act_window_id" ref="website_livechat.website_visitor_livechat_session_action"/>
    </record>

    <!-- website visitor views -->
    <record id="website_visitor_view_kanban" model="ir.ui.view">
        <field name="name">website.visitor.view.kanban.inherit.website.livechat</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_kanban"/>
        <field name="arch" type="xml">
            <field name="page_ids" position="after">
                <field name="livechat_operator_id"/>
            </field>
            <xpath expr="//div[hasclass('w_visitor_kanban_actions')]" position="before">
                <div t-if="record.livechat_operator_id.raw_value">
                    <span class="fa fa-comments mr-2"/>Speaking With <span class="font-weight-bold"><field name="livechat_operator_name"/></span>
                </div>
                <t t-else="">
                    <br attrs="{'invisible': ['|', ('livechat_operator_id', '!=', False), ('is_connected', '=', False)]}"/>
                </t>
            </xpath>
            <xpath expr="//div[hasclass('w_visitor_kanban_actions_ungrouped')]" position="before">
                <div class="col" >
                    <b>
                        <field name="livechat_operator_name" t-if="record.livechat_operator_id.raw_value"/>
                        <span t-else="">-</span>
                    </b>
                    <div>Speaking With</div>
                </div>
            </xpath>
            <xpath expr="//div[hasclass('w_visitor_kanban_actions')]" position="inside">
                <button name="action_send_chat_request" type="object"
                        class="btn btn-secondary"
                        attrs="{'invisible': ['|', ('livechat_operator_id', '!=', False), ('is_connected', '=', False)]}">
                        Chat
                </button>
            </xpath>
            <xpath expr="//div[hasclass('w_visitor_kanban_actions_ungrouped')]" position="inside">
                <button name="action_send_chat_request" type="object"
                        class="btn btn-secondary border"
                        attrs="{'invisible': ['|', ('livechat_operator_id', '!=', False), ('is_connected', '=', False)]}">
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
                attrs="{'invisible': ['|', ('livechat_operator_id', '!=', False), ('is_connected', '=', False)]}"/>
            </xpath>
            <xpath expr="//group[@id='general_info']" position="before">
                <group id="livechat_info">
                    <field name="livechat_operator_id"/>
                </group>
            </xpath>
            <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                <button name="%(website_visitor_livechat_session_action)d" type="action" class="oe_stat_button" icon="fa-comment"
                        attrs="{'invisible': [('session_count', '=', 0)]}">
                    <field name="session_count" widget="statinfo" string="Sessions"/>
                </button>
            </xpath>
        </field>
    </record>

    <record id="website_visitor_view_tree" model="ir.ui.view">
        <field name="name">website.visitor.view.tree.inherit.website.livechat</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='is_connected']" position="after">
                <field name="livechat_operator_id"/>
                <button name="action_send_chat_request" string="Send chat request" type="object" icon="fa-comments"
                        attrs="{'invisible': ['|', ('livechat_operator_id', '!=', False), ('is_connected', '=', False)]}"/>
            </xpath>
        </field>
    </record>

    <record id="website_visitor_view_search" model="ir.ui.view">
        <field name="name">website.visitor.view.search.website.livechat</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//search" position="inside">
                <filter string="Busy" name="in_conversation" domain="[('livechat_operator_id', '!=', False)]"/>
                <filter string="Available" name="not_in_conversation" domain="[('livechat_operator_id', '=', False)]"/>
            </xpath>
        </field>
    </record>

    <record id="website_livechat_send_chat_request_action_server" model="ir.actions.server">
        <field name="name">Send Chat Requests</field>
        <field name="type">ir.actions.server</field>
        <field name="model_id" ref="model_website_visitor"/>
        <field name="binding_model_id" ref="model_website_visitor"/>
        <field name="binding_view_types">list</field>
        <field name="state">code</field>
        <field name="code">
            if records:
                action = records.action_send_chat_request()
        </field>
    </record>

    <record id="website.website_visitors_action" model="ir.actions.act_window">
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
              Wait for visitors to come to your website to see their history.</p>
            <p>Interact with them by sending them messages.</p>
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

