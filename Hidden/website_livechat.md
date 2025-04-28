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
    'assets': {
        'mail.assets_discuss_public': [
            'website_livechat/static/src/components/*/*',
            'website_livechat/static/src/models/*/*.js',
        ],
        'web.assets_frontend': [
            'mail/static/src/js/utils.js',
            'im_livechat/static/src/legacy/public_livechat.js',
            'website_livechat/static/src/legacy/public_livechat.js',
            'im_livechat/static/src/legacy/public_livechat.scss',
            'website_livechat/static/src/legacy/public_livechat.scss',
        ],
        'website.assets_editor': [
            'website_livechat/static/src/js/**/*',
        ],
        'web.assets_backend': [
            'website_livechat/static/src/components/*/*.js',
            'website_livechat/static/src/components/*/*.scss',
            'website_livechat/static/src/models/*/*.js',
        ],
        'web.assets_tests': [
            'website_livechat/static/tests/tours/**/*',
        ],
        'web.qunit_suite_tests': [
            'website_livechat/static/src/components/*/tests/*.js',
            'website_livechat/static/src/models/*/tests/*.js',
            'website_livechat/static/tests/helpers/mock_models.js',
            'website_livechat/static/tests/helpers/mock_server.js',
        ],
        'web.assets_qweb': [
            'website_livechat/static/src/components/*/*.xml',
        ],
    },
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
            ('res_model', '=', 'mail.channel'), ('res_id', 'in', channel.sudo().channel_ids.ids),
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

    @http.route('/im_livechat/get_session', type="json", auth='public', cors="*")
    def get_session(self, channel_id, anonymous_name, previous_operator_id=None, **kwargs):
        """ Override to use visitor name instead of 'Visitor' whenever a visitor start a livechat session. """
        visitor_sudo = request.env['website.visitor']._get_visitor_from_request()
        if visitor_sudo:
            anonymous_name = visitor_sudo.with_context(lang=visitor_sudo.lang_id.code).display_name
        return super(WebsiteLivechat, self).get_session(channel_id, anonymous_name, previous_operator_id=previous_operator_id, **kwargs)

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
from . import main
from . import test

```

## File: data\website_livechat_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
     <data noupdate="1">
         <record id="website.default_website" model="website">
             <field name="channel_id" ref="im_livechat.im_livechat_channel_data"></field>
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

    website_description = fields.Html("Website description", default=False, help="Description of the channel displayed on the website page", sanitize_attributes=False, translate=html_translate, sanitize_form=False)

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
            mail_channel_vals['livechat_visitor_id'] = visitor_sudo.id
            if not user_id:
                mail_channel_vals['anonymous_name'] = visitor_sudo.display_name + (' (%s)' % visitor_sudo.country_id.name if visitor_sudo.country_id else '')
            # As chat requested by the visitor, delete the chat requested by an operator if any to avoid conflicts between two flows
            # TODO DBE : Move this into the proper method (open or init mail channel)
            chat_request_channel = self.env['mail.channel'].sudo().search([('livechat_visitor_id', '=', visitor_sudo.id), ('livechat_active', '=', True)])
            for mail_channel in chat_request_channel:
                mail_channel._close_livechat_session(cancel=True, operator=operator.name)

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
from odoo.exceptions import AccessError


class MailChannel(models.Model):
    _inherit = 'mail.channel'

    livechat_visitor_id = fields.Many2one('website.visitor', string='Visitor')

    def _execute_channel_pin(self, pinned=False):
        """ Override to clean an empty livechat channel.
         This is typically called when the operator send a chat request to a website.visitor
         but don't speak to him and closes the chatter.
         This allows operators to send the visitor a new chat request.
         If active empty livechat channel,
         delete mail_channel as not useful to keep empty chat
         """
        super(MailChannel, self)._execute_channel_pin(pinned)
        if self.livechat_active and not self.message_ids:
            self.sudo().unlink()

    def channel_info(self):
        """
        Override to add visitor information on the mail channel infos.
        This will be used to display a banner with visitor informations
        at the top of the livechat channel discussion view in discuss module.
        """
        channel_infos = super().channel_info()
        channel_infos_dict = dict((c['id'], c) for c in channel_infos)
        for channel in self.filtered('livechat_visitor_id'):
            visitor = channel.livechat_visitor_id
            try:
                channel_infos_dict[channel.id]['visitor'] = {
                    'display_name': visitor.display_name,
                    'country_code': visitor.country_id.code.lower() if visitor.country_id else False,
                    'country_id': visitor.country_id.id,
                    'id': visitor.id,
                    'is_connected': visitor.is_connected,
                    'history': self.sudo()._get_visitor_history(visitor),
                    'website_name': visitor.website_id.name,
                    'lang_name': visitor.lang_id.name,
                    'partner_id': visitor.partner_id.id,
                }
            except AccessError:
                pass
        return list(channel_infos_dict.values())

    def _get_visitor_history(self, visitor):
        """
        Prepare history string to render it in the visitor info div on discuss livechat channel view.
        :param visitor: website.visitor of the channel
        :return: arrow separated string containing navigation history information
        """
        recent_history = self.env['website.track'].search([('page_id', '!=', False), ('visitor_id', '=', visitor.id)], limit=3)
        return ' → '.join(visit.page_id.name + ' (' + visit.visit_datetime.strftime('%H:%M') + ')' for visit in reversed(recent_history))

    def _get_visitor_leave_message(self, operator=False, cancel=False):
        name = _('The visitor') if not self.livechat_visitor_id else self.livechat_visitor_id.display_name
        if cancel:
            message = _("""%s has started a conversation with %s. 
                        The chat request has been canceled.""") % (name, operator or _('an operator'))
        else:
            message = _('%s has left the conversation.', name)

        return message

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

from odoo import fields, models, _
from odoo.addons.http_routing.models.ir_http import url_for


class Website(models.Model):

    _inherit = "website"

    channel_id = fields.Many2one('im_livechat.channel', string='Website Live Chat Channel')

    def get_livechat_channel_info(self):
        """ Get the livechat info dict (button text, channel name, ...) for the livechat channel of
            the current website.
        """
        self.ensure_one()
        if self.channel_id:
            livechat_info = self.channel_id.sudo().get_livechat_info()
            if livechat_info['available']:
                livechat_request_session = self._get_livechat_request_session()
                if livechat_request_session:
                    livechat_info['options']['chat_request_session'] = livechat_request_session
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
        if visitor:
            # get active chat_request linked to visitor
            chat_request_channel = self.env['mail.channel'].sudo().search([
                ('livechat_visitor_id', '=', visitor.id),
                ('livechat_channel_id', '=', self.channel_id.id),
                ('livechat_active', '=', True),
                ('has_message', '=', True)
            ], order='create_date desc', limit=1)
            if chat_request_channel:
                return {
                    "folded": False,
                    "id": chat_request_channel.id,
                    "operator_pid": [
                        chat_request_channel.livechat_operator_id.id,
                        chat_request_channel.livechat_operator_id.user_livechat_username or chat_request_channel.livechat_operator_id.display_name,
                        chat_request_channel.livechat_operator_id.user_livechat_username,
                    ],
                    "name": chat_request_channel.name,
                    "uuid": chat_request_channel.uuid,
                    "type": "chat_request"
                }
        return {}

    def get_suggested_controllers(self):
        suggested_controllers = super(Website, self).get_suggested_controllers()
        suggested_controllers.append((_('Live Support'), url_for('/livechat'), 'website_livechat'))
        return suggested_controllers

```

## File: models\website_visitor.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, timedelta
import json

from odoo import models, api, fields, _
from odoo.exceptions import UserError
from odoo.http import request
from odoo.tools.sql import column_exists, create_column


class WebsiteVisitor(models.Model):
    _inherit = 'website.visitor'

    livechat_operator_id = fields.Many2one('res.partner', compute='_compute_livechat_operator_id', store=True, string='Speaking with')
    livechat_operator_name = fields.Char('Operator Name', related="livechat_operator_id.name")
    mail_channel_ids = fields.One2many('mail.channel', 'livechat_visitor_id',
                                       string="Visitor's livechat channels", readonly=True)
    session_count = fields.Integer('# Sessions', compute="_compute_session_count")

    def _auto_init(self):
        # Skip the computation of the field `livechat_operator_id` at the module installation
        # We can assume no livechat operator attributed to visitor if it was not installed
        if not column_exists(self.env.cr, "website_visitor", "livechat_operator_id"):
            create_column(self.env.cr, "website_visitor", "livechat_operator_id", "int4")
        return super()._auto_init()

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
        sessions = self.env['mail.channel'].search([('livechat_visitor_id', 'in', self.ids)])
        session_count = dict.fromkeys(self.ids, 0)
        for session in sessions.filtered(lambda c: c.message_ids):
            session_count[session.livechat_visitor_id.id] += 1
        for visitor in self:
            visitor.session_count = session_count.get(visitor.id, 0)

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
                raise UserError(_('No Livechat Channel allows you to send a chat request for website %s.', website.name))
        self.website_id.channel_id.write({'user_ids': [(4, self.env.user.id)]})
        # Create chat_requests and linked mail_channels
        mail_channel_vals_list = []
        for visitor in self:
            operator = self.env.user
            country = visitor.country_id
            visitor_name = "%s (%s)" % (visitor.display_name, country.name) if country else visitor.display_name
            channel_partner_to_add = [(4, operator.partner_id.id)]
            if visitor.partner_id:
                channel_partner_to_add.append((4, visitor.partner_id.id))
            else:
                channel_partner_to_add.append((4, self.env.ref('base.public_partner').id))
            mail_channel_vals_list.append({
                'channel_partner_ids': channel_partner_to_add,
                'livechat_channel_id': visitor.website_id.channel_id.id,
                'livechat_operator_id': self.env.user.partner_id.id,
                'channel_type': 'livechat',
                'public': 'private',
                'country_id': country.id,
                'anonymous_name': visitor_name,
                'name': ', '.join([visitor_name, operator.livechat_username if operator.livechat_username else operator.name]),
                'livechat_visitor_id': visitor.id,
                'livechat_active': True,
            })
        if mail_channel_vals_list:
            mail_channels = self.env['mail.channel'].create(mail_channel_vals_list)
            # Open empty chatter to allow the operator to start chatting with the visitor.
            channel_members = self.env['mail.channel.partner'].sudo().search([
                ('partner_id', '=', self.env.user.partner_id.id),
                ('channel_id', 'in', mail_channels.ids),
            ])
            channel_members.write({
                'fold_state': 'open',
                'is_minimized': True,
            })
            mail_channels_info = mail_channels.channel_info()
            notifications = []
            for mail_channel_info in mail_channels_info:
                notifications.append([operator.partner_id, 'website_livechat.send_chat_request', mail_channel_info])
            self.env['bus.bus']._sendmany(notifications)

    def _link_to_visitor(self, target, keep_unique=True):
        """ Copy sessions of the secondary visitors to the main partner visitor. """
        if target.partner_id:
            target.mail_channel_ids |= self.mail_channel_ids
        super(WebsiteVisitor, self)._link_to_visitor(target, keep_unique=keep_unique)

    def _link_to_partner(self, partner, update_values=None):
        """ Adapt partner in members of related livechats """
        if partner:
            self.mail_channel_ids.channel_partner_ids = [
                (3, self.env.ref('base.public_partner').id),
                (4, partner.id),
            ]
        super(WebsiteVisitor, self)._link_to_partner(partner, update_values=update_values)

    def _create_visitor(self):
        visitor = super(WebsiteVisitor, self)._create_visitor()
        mail_channel_uuid = json.loads(request.httprequest.cookies.get('im_livechat_session', '{}')).get('uuid')
        if mail_channel_uuid:
            mail_channel = request.env["mail.channel"].sudo().search([("uuid", "=", mail_channel_uuid)])
            mail_channel.write({
                'livechat_visitor_id': visitor.id,
                'anonymous_name': visitor.display_name
            })
        return visitor

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
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#CD7690"/><stop offset="100%" stop-color="#CA5377"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M42.103 69H4c-2 0-4-.145-4-4.07V41.395L20 19.14C23.333 15.07 28.333 11 35 11s11.667 7.801 17 17.298v8.14h2V48.65l-2.635 1.09L51 54.754 42.103 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M35.096 13C25.103 13 18 21.103 18 31.096V35c-1.333 0-2 .667-2 2v8.171C16 48.51 18.662 51 22 51h2c1.333 0 2-.667 2-2V37c0-1.333-.667-2-2-2h-2.979v-3.904c0-7.781 6.294-14.075 14.075-14.075C42.878 17.021 49 23.315 49 31.096V35h-3c-1.333 0-2 .667-2 2v12c0 1.333.667 2 2 2h3c.667 1.333.667 2.667 0 4h-9c0-.667-.328-1-.984-1H36c-.667 0-1 .333-1 1v1c.064.667.398 1 1 1h13c2 0 3-2 3-6 2 0 3-2 3-6v-8c0-1.333-1-2-3-2v-3.904C52 21.103 45.09 13 35.096 13z" opacity=".3"/><path fill="#FFF" d="M35.096 11C25.103 11 18 19.103 18 29.096V33c-1.333 0-2 .667-2 2v8.171C16 46.51 18.662 49 22 49h2c1.333 0 2-.667 2-2V35c0-1.333-.667-2-2-2h-2.979v-3.904c0-7.781 6.294-14.075 14.075-14.075C42.878 15.021 49 21.315 49 29.096V33h-3c-1.333 0-2 .667-2 2v12c0 1.333.667 2 2 2h3c.667 1.333.667 2.667 0 4h-9c0-.667-.328-1-.984-1H36c-.667 0-1 .333-1 1v1c.064.667.398 1 1 1h13c2 0 3-2 3-6 2 0 3-2 3-6v-8c0-1.333-1-2-3-2v-3.904C52 19.103 45.09 11 35.096 11z"/></g></g></svg>
```

## File: static\src\components\thread_view\thread_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.ThreadView" t-inherit-mode="extension">
        <xpath expr="//t[@t-if='threadView.topbar']" position="after">
            <t t-if="threadView.hasVisitorBanner">
                <VisitorBanner
                    class="border-bottom"
                    visitorLocalId="threadView.thread.visitor.localId"
                />
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\components\visitor_banner\visitor_banner.js

```javascript
/** @odoo-module **/

import { registerMessagingComponent } from '@mail/utils/messaging_component';

const { Component } = owl;

class VisitorBanner extends Component {

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @returns {website_livechat.visitor}
     */
    get visitor() {
        return this.messaging && this.messaging.models['website_livechat.visitor'].get(this.props.visitorLocalId);
    }

}

Object.assign(VisitorBanner, {
    props: {
        visitorLocalId: String,
    },
    template: 'website_livechat.VisitorBanner',
});

registerMessagingComponent(VisitorBanner);

export default VisitorBanner;

```

## File: static\src\components\visitor_banner\visitor_banner.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="website_livechat.VisitorBanner" owl="1">
        <div class="o_VisitorBanner">
            <div class="o_VisitorBanner_sidebar">
                <div class="o_VisitorBanner_avatarContainer">
                    <img class="o_VisitorBanner_avatar rounded-circle" t-att-src="visitor.avatarUrl" alt="Avatar"/>
                    <t t-if="visitor.is_connected">
                        <i class="o_VisitorBanner_onlineStatusIcon fa fa-circle" title="Online" role="img" aria-label="Visitor is online"/>
                    </t>
                </div>
            </div>
            <div class="o_VisitorBanner_content">
                <t t-if="visitor.country">
                    <img class="o_VisitorBanner_country o_country_flag" t-att-src="visitor.country.flagUrl" t-att-alt="visitor.country.code or visitor.country.name"/>
                </t>
                <span class="o_VisitorBanner_visitor" t-esc="visitor.nameOrDisplayName"/>
                <span class="o_VisitorBanner_language">
                    <i class="o_VisitorBanner_languageIcon fa fa-comment-o" aria-label="Lang"/>
                    <t t-esc="visitor.lang_name"/>
                </span>
                <t t-if="visitor.website_name">
                    <span class="o_VisitorBanner_website">
                        <i class="o_VisitorBanner_websiteIcon fa fa-globe" aria-label="Website"/>
                        <span t-esc="visitor.website_name"/>
                    </span>
                </t>
                <div class="o_VisitorBanner_history">
                    <i class="o_VisitorBanner_historyIcon fa fa-history" aria-label="History"/>
                    <span t-esc="visitor.history"/>
                </div>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\js\website_livechat.editor.js

```javascript
/** @odoo-module **/

import { _t } from 'web.core';
import wUtils from 'website.utils';
import WebsiteNewMenu from 'website.newMenu';

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

```

## File: static\src\legacy\public_livechat.js

```javascript
odoo.define('website_livechat.legacy.website_livechat.livechat_request', function (require) {
"use strict";

var utils = require('web.utils');
var LivechatButton = require('im_livechat.legacy.im_livechat.im_livechat').LivechatButton;


LivechatButton.include({
    className: `${LivechatButton.prototype.className} o_bottom_fixed_element`,

    /**
     * @override
     * Check if a chat request is opened for this visitor
     * if yes, replace the session cookie and start the conversation immediately.
     * Do this before calling super to have everything ready before executing existing start logic.
     * This is used for chat request mechanism, when an operator send a chat request
     * from backend to a website visitor.
     */
    willStart: function () {
        if (this.options.chat_request_session) {
            utils.set_cookie('im_livechat_session', JSON.stringify(this.options.chat_request_session), 60*60);
        }
        return this._super();
    },
    /**
     * @override
     */
     start() {
        // We trigger a resize to launch the event that checks if this element hides
        // a button when the page is loaded.
        $(window).trigger('resize');
        return this._super(...arguments);
    },
});

return {
    LivechatButton: LivechatButton,
};

});

```

## File: static\src\models\messaging_notification_handler\messaging_notification_handler.js

```javascript
/** @odoo-module **/

import { registerInstancePatchModel } from '@mail/model/model_core';

registerInstancePatchModel('mail.messaging_notification_handler', 'website_livechat/static/src/models/messaging_notification_handler/messaging_notification_handler.js', {

    //----------------------------------------------------------------------
    // Private
    //----------------------------------------------------------------------

    /**
     * @override
     */
    async _handleNotification(message) {
        if (message.type === 'website_livechat.send_chat_request') {
            const convertedData = this.messaging.models['mail.thread'].convertData(
                Object.assign({ model: 'mail.channel' }, message.payload)
            );
            this.messaging.models['mail.thread'].insert(convertedData);
            const channel = this.messaging.models['mail.thread'].findFromIdentifyingData({
                id: message.payload.id,
                model: 'mail.channel',
            });
            this.messaging.chatWindowManager.openThread(channel, {
                makeActive: true,
            });
        }
        return this._super(message);
    },
});

```

## File: static\src\models\thread\thread.js

```javascript
/** @odoo-module **/

import {
    registerClassPatchModel,
    registerFieldPatchModel,
} from '@mail/model/model_core';
import { many2one } from '@mail/model/model_field';
import { insert, unlink } from '@mail/model/model_field_command';

registerClassPatchModel('mail.thread', 'website_livechat/static/src/models/thread/thread.js', {

    //----------------------------------------------------------------------
    // Public
    //----------------------------------------------------------------------

    /**
     * @override
     */
    convertData(data) {
        const data2 = this._super(data);
        if ('visitor' in data) {
            if (data.visitor) {
                data2.visitor = insert(this.messaging.models['website_livechat.visitor'].convertData(data.visitor));
            } else {
                data2.visitor = unlink();
            }
        }
        return data2;
    },

});

registerFieldPatchModel('mail.thread', 'website_livechat/static/src/models/thread/thread.js', {
    /**
     * Visitor connected to the livechat.
     */
    visitor: many2one('website_livechat.visitor', {
        inverse: 'threads',
    }),
});

```

## File: static\src\models\thread_view\thread_view.js

```javascript
/** @odoo-module **/

import {
    registerFieldPatchModel,
    registerInstancePatchModel,
} from '@mail/model/model_core';
import { attr } from '@mail/model/model_field';

registerInstancePatchModel('mail.thread_view', 'website_livechat/static/src/models/thread_view/thread_view.js', {

    //----------------------------------------------------------------------
    // Private
    //----------------------------------------------------------------------

    /**
     * @private
     * @returns {boolean}
     */
    _computeHasVisitorBanner() {
        return Boolean(this.thread && this.thread.visitor && this.threadViewer && this.threadViewer.discuss);
    },
});

registerFieldPatchModel('mail.thread_view', 'website_livechat/static/src/models/thread_view/thread_view.js', {
    /**
     * Determines whether visitor banner should be displayed.
     */
    hasVisitorBanner: attr({
        compute: '_computeHasVisitorBanner',
    }),
});

```

## File: static\src\models\visitor\visitor.js

```javascript
/** @odoo-module **/

import { registerNewModel } from '@mail/model/model_core';
import { attr, many2one, one2many } from '@mail/model/model_field';
import { insert, link, unlink } from '@mail/model/model_field_command';

function factory(dependencies) {

    class Visitor extends dependencies['mail.model'] {
        //----------------------------------------------------------------------
        // Public
        //----------------------------------------------------------------------

        /**
         * @override
         */
        static convertData(data) {
            const data2 = {};
            if ('country_id' in data) {
                if (data.country_id) {
                    data2.country = insert({
                        id: data.country_id,
                        code: data.country_code,
                    });
                } else {
                    data2.country = unlink();
                }
            }
            if ('history' in data) {
                data2.history = data.history;
            }
            if ('id' in data) {
                data2.id = data.id;
            }
            if ('is_connected' in data) {
                data2.is_connected = data.is_connected;
            }
            if ('lang_name' in data) {
                data2.lang_name = data.lang_name;
            }
            if ('display_name' in data) {
                data2.display_name = data.display_name;
            }
            if ('partner_id' in data) {
                if (data.partner_id) {
                    data2.partner = insert({ id: data.partner_id });
                } else {
                    data2.partner = unlink();
                }
            }
            if ('website_name' in data) {
                data2.website_name = data.website_name;
            }
            return data2;
        }

        //----------------------------------------------------------------------
        // Private
        //----------------------------------------------------------------------

        /**
         * @private
         * @returns {string}
         */
        _computeAvatarUrl() {
            if (!this.partner) {
                return '/mail/static/src/img/smiley/avatar.jpg';
            }
            return this.partner.avatarUrl;
        }

        /**
         * @private
         * @returns {mail.country}
         */
        _computeCountry() {
            if (this.partner && this.partner.country) {
                return link(this.partner.country);
            }
            if (this.country) {
                return link(this.country);
            }
            return unlink();
        }

        /**
         * @private
         * @returns {string}
         */
        _computeNameOrDisplayName() {
            if (this.partner) {
                return this.partner.nameOrDisplayName;
            }
            return this.display_name;
        }
    }

    Visitor.fields = {
        /**
         * Url to the avatar of the visitor.
         */
        avatarUrl: attr({
            compute: '_computeAvatarUrl',
        }),
        /**
         * Country of the visitor.
         */
        country: many2one('mail.country', {
            compute: '_computeCountry',
        }),
        /**
         * Display name of the visitor.
         */
        display_name: attr(),
        /**
         * Browsing history of the visitor as a string.
         */
        history: attr(),
        /**
         * States the id of this visitor.
         */
        id: attr({
            readonly: true,
            required: true,
        }),
        /**
         * Determine whether the visitor is connected or not.
         */
        is_connected: attr(),
        /**
         * Name of the language of the visitor. (Ex: "English")
         */
        lang_name: attr(),
        nameOrDisplayName: attr({
            compute: '_computeNameOrDisplayName',
        }),
        /**
         * Partner linked to this visitor, if any.
         */
        partner: many2one('mail.partner'),
        /**
         * Threads with this visitor as member
         */
        threads: one2many('mail.thread', {
            inverse: 'visitor',
        }),
        /**
         * Name of the website on which the visitor is connected. (Ex: "Website 1")
         */
        website_name: attr(),
    };
    Visitor.identifyingFields = ['id'];
    Visitor.modelName = 'website_livechat.visitor';

    return Visitor;
}

registerNewModel('website_livechat.visitor', factory);

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
        <!--
            Integrate Livechat in Common Frontend for Website
            Template registering all the assets required to execute the Livechat from a page containing Odoo
        -->
        <template id="loader" inherit_id="website.layout" name="Livechat : include loader on Website">
            <xpath expr="//div[@id='wrapwrap']" position="after">
                <t t-if="website and website.channel_id and not no_livechat">
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
                                                            <b style="font-size: 30px">
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
                                                            <b style="font-size: 30px">
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
                                <div class="media mt-3">
                                    <img t-att-src="image_data_uri(user.avatar_128)" class="o_image_64_cover rounded o_livechat_operator_avatar" t-att-alt="user.livechat_username or user.name"/>
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
                                                        <img t-attf-src='/rating/static/src/img/rating_5.png' alt="Great" width="16px" height="16px"/>
                                                        <span class="align-middle"><t t-esc="ratings_per_user[user.partner_id.id]['great']"/>%</span>
                                                    </div>
                                                    <div class="col-lg-4 pl-0 pr-0">
                                                        <img t-attf-src='/rating/static/src/img/rating_3.png' alt="Okay" width="16px" height="16px"/>
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
                                        <div t-attf-class="media#{' mt-3' if channel_index else ''}">
                                            <a t-attf-href="/livechat/channel/#{ slug(channel)}">
                                                <img t-att-src="channel.image_128 and image_data_uri(channel.image_128) or '/web/static/img/placeholder.png'" t-att-alt="channel.name" class="o_image_64_cover"/>
                                            </a>
                                            <div class="media-body h-100 my-auto">
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
            <xpath expr="//div[@id='o_new_content_menu_choices']//div[@name='module_website_livechat']" position="replace">
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
        <field name="domain">[('livechat_visitor_id', '=', active_id), ('has_message', '=', True)]</field>
        <field name="context">{
                'search_default_livechat_visitor_id': [active_id],
                'default_livechat_visitor_id': active_id,
            }</field>
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
                <div>Chats<span class="float-right font-weight-bold"><field name="session_count"/></span></div>
                <div t-if="record.livechat_operator_id.raw_value">
                    Speaking With
                    <div name="livechat_operator_id"
                        class="font-weight-bold float-right d-inline-block">
                        <img class="oe_avatar rounded-circle" width="19" height="19"
                            t-attf-src="/web/image/res.partner/#{record.livechat_operator_id.raw_value}/avatar_128"
                            alt="Operator Avatar"/>
                        <field name="livechat_operator_name"/>
                    </div>
                </div>
                <t t-else="">
                    <br attrs="{'invisible': ['|', ('livechat_operator_id', '!=', False), ('is_connected', '=', False)]}"/>
                </t>
            </xpath>
            <xpath expr="//div[hasclass('w_visitor_kanban_actions_ungrouped')]" position="before">
                <div class="col mx-2">
                    <b>
                        <field name="session_count"/>
                    </b>
                    <div>Chats</div>
                </div>
                <div class="col mx-2">
                    <div t-if="record.livechat_operator_id.raw_value" class="font-weight-bold">
                        <img name="livechat_operator_id"
                            class="oe_avatar rounded-circle" width="19" height="19"
                            t-attf-src="/web/image/res.partner/#{record.livechat_operator_id.raw_value}/avatar_128"
                            alt="Operator Avatar"/>
                        <field name="livechat_operator_name"/>
                    </div>
                    <div t-if="record.livechat_operator_id.raw_value">Speaking With</div>
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
                <group id="livechat_info" attrs="{'invisible': [('livechat_operator_id', '=', False)]}">
                    <field name="livechat_operator_id" widget="many2one_avatar"/>
                </group>
            </xpath>
            <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                <button name="%(website_visitor_livechat_session_action)d" type="action" class="oe_stat_button" icon="fa-comment"
                        attrs="{'invisible': [('session_count', '=', 0)]}">
                    <field name="session_count" widget="statinfo" string="Chats"/>
                </button>
            </xpath>
        </field>
    </record>

    <record id="website_visitor_view_tree" model="ir.ui.view">
        <field name="name">website.visitor.view.tree.inherit.website.livechat</field>
        <field name="model">website.visitor</field>
        <field name="inherit_id" ref="website.website_visitor_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='email']" position="after">
                <field name="livechat_operator_id" optional="hide"/>
                <button name="action_send_chat_request" string="Chat" type="object" icon="fa-comments"
                        attrs="{'invisible': ['|', ('livechat_operator_id', '!=', False), ('is_connected', '=', False)]}"/>
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

    <menuitem
        id="website_livechat_visitor_menu"
        name="Visitors"
        parent="im_livechat.menu_livechat_root"
        action="website.website_visitors_action"
        groups="im_livechat.im_livechat_group_user"
        sequence="15"/>
</data></odoo>

```

