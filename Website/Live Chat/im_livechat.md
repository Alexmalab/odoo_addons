# Odoo Module: im_livechat

Category: Website/Live Chat

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
from . import controllers
from . import models
from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name' : 'Live Chat',
    'version': '1.0',
    'sequence': 170,
    'summary': 'Chat with your website visitors',
    'category': 'Website/Live Chat',
    'complexity': 'easy',
    'website': 'https://www.odoo.com/page/live-chat',
    'description':
        """
Live Chat Support
==========================

Allow to drop instant messaging widgets on any web page that will communicate
with the current server and dispatch visitors request amongst several live
chat operators.
Help your customers with this chat, and analyse their feedback.

        """,
    'data': [
        "data/mail_shortcode_data.xml",
        "data/mail_data.xml",
        "data/im_livechat_channel_data.xml",
        "security/im_livechat_channel_security.xml",
        "security/ir.model.access.csv",
        "views/rating_views.xml",
        "views/mail_channel_views.xml",
        "views/im_livechat_channel_views.xml",
        "views/im_livechat_channel_templates.xml",
        "views/res_users_views.xml",
        "report/im_livechat_report_channel_views.xml",
        "report/im_livechat_report_operator_views.xml"
    ],
    'demo': [
        "data/im_livechat_channel_demo.xml",
        'data/mail_shortcode_demo.xml',
    ],
    'depends': ["mail", "rating"],
    'qweb': ['static/src/xml/*.xml'],
    'installable': True,
    'auto_install': False,
    'application': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64

from odoo import http,tools, _
from odoo.http import request
from odoo.addons.base.models.assetsbundle import AssetsBundle


class LivechatController(http.Controller):

    # Note: the `cors` attribute on many routes is meant to allow the livechat
    # to be embedded in an external website.

    @http.route('/im_livechat/external_lib.<any(css,js):ext>', type='http', auth='public')
    def livechat_lib(self, ext, **kwargs):
        # _get_asset return the bundle html code (script and link list) but we want to use the attachment content
        xmlid = 'im_livechat.external_lib'
        files, remains = request.env["ir.qweb"]._get_asset_content(xmlid, options=request.context)
        asset = AssetsBundle(xmlid, files)

        mock_attachment = getattr(asset, ext)()
        if isinstance(mock_attachment, list):  # suppose that CSS asset will not required to be split in pages
            mock_attachment = mock_attachment[0]
        # can't use /web/content directly because we don't have attachment ids (attachments must be created)
        status, headers, content = request.env['ir.http'].binary_content(id=mock_attachment.id, unique=asset.checksum)
        content_base64 = base64.b64decode(content) if content else ''
        headers.append(('Content-Length', len(content_base64)))
        return request.make_response(content_base64, headers)

    @http.route('/im_livechat/load_templates', type='json', auth='none', cors="*")
    def load_templates(self, **kwargs):
        base_url = request.httprequest.base_url
        templates = [
            'mail/static/src/xml/abstract_thread_window.xml',
            'mail/static/src/xml/discuss.xml',
            'mail/static/src/xml/thread.xml',
            'im_livechat/static/src/xml/im_livechat.xml',
        ]
        return [tools.file_open(tmpl, 'rb').read() for tmpl in templates]

    @http.route('/im_livechat/support/<int:channel_id>', type='http', auth='public')
    def support_page(self, channel_id, **kwargs):
        channel = request.env['im_livechat.channel'].sudo().browse(channel_id)
        return request.render('im_livechat.support_page', {'channel': channel})

    @http.route('/im_livechat/loader/<int:channel_id>', type='http', auth='public')
    def loader(self, channel_id, **kwargs):
        username = kwargs.get("username", _("Visitor"))
        channel = request.env['im_livechat.channel'].sudo().browse(channel_id)
        info = channel.get_livechat_info(username=username)
        return request.render('im_livechat.loader', {'info': info, 'web_session_required': True}, headers=[('Content-Type', 'application/javascript')])

    @http.route('/im_livechat/init', type='json', auth="public", cors="*")
    def livechat_init(self, channel_id):
        available = len(request.env['im_livechat.channel'].sudo().browse(channel_id)._get_available_users())
        rule = {}
        if available:
            # find the country from the request
            country_id = False
            country_code = request.session.geoip and request.session.geoip.get('country_code') or False
            if country_code:
                country_ids = request.env['res.country'].sudo().search([('code', '=', country_code)])
                if country_ids:
                    country_id = country_ids[0].id
            # extract url
            url = request.httprequest.headers.get('Referer')
            # find the first matching rule for the given country and url
            matching_rule = request.env['im_livechat.channel.rule'].sudo().match_rule(channel_id, url, country_id)
            if matching_rule:
                rule = {
                    'action': matching_rule.action,
                    'auto_popup_timer': matching_rule.auto_popup_timer,
                    'regex_url': matching_rule.regex_url,
                }
        return {
            'available_for_me': available and (not rule or rule['action'] != 'hide_button'),
            'rule': rule,
        }

    @http.route('/im_livechat/get_session', type="json", auth='public', cors="*")
    def get_session(self, channel_id, anonymous_name, previous_operator_id=None, **kwargs):
        user_id = None
        country_id = None
        # if the user is identifiy (eg: portal user on the frontend), don't use the anonymous name. The user will be added to session.
        if request.session.uid:
            user_id = request.env.user.id
            country_id = request.env.user.country_id.id
        else:
            # if geoip, add the country name to the anonymous name
            if request.session.geoip:
                # get the country of the anonymous person, if any
                country_code = request.session.geoip.get('country_code', "")
                country = request.env['res.country'].sudo().search([('code', '=', country_code)], limit=1) if country_code else None
                if country:
                    anonymous_name = "%s (%s)" % (anonymous_name, country.name)
                    country_id = country.id

        if previous_operator_id:
            previous_operator_id = int(previous_operator_id)

        return request.env["im_livechat.channel"].with_context(lang=False).sudo().browse(channel_id)._open_livechat_mail_channel(anonymous_name, previous_operator_id, user_id, country_id)

    @http.route('/im_livechat/feedback', type='json', auth='public', cors="*")
    def feedback(self, uuid, rate, reason=None, **kwargs):
        Channel = request.env['mail.channel']
        channel = Channel.sudo().search([('uuid', '=', uuid)], limit=1)
        if channel:
            # limit the creation : only ONE rating per session
            values = {
                'rating': rate,
                'consumed': True,
                'feedback': reason,
            }
            if not channel.rating_ids:
                res_model_id = request.env['ir.model'].sudo().search([('model', '=', channel._name)], limit=1).id
                values.update({
                    'res_id': channel.id,
                    'res_model_id': res_model_id,
                })
                # find the partner (operator)
                if channel.channel_partner_ids:
                    values['rated_partner_id'] = channel.channel_partner_ids[0] and channel.channel_partner_ids[0].id or False
                # if logged in user, set its partner on rating
                values['partner_id'] = request.env.user.partner_id.id if request.session.uid else False
                # create the rating
                rating = request.env['rating.rating'].sudo().create(values)
            else:
                rating = channel.rating_ids[0]
                rating.write(values)
            return rating.id
        return False

    @http.route('/im_livechat/history', type="json", auth="public", cors="*")
    def history_pages(self, pid, channel_uuid, page_history=None):
        partner_ids = (pid, request.env.user.partner_id.id)
        channel = request.env['mail.channel'].sudo().search([('uuid', '=', channel_uuid), ('channel_partner_ids', 'in', partner_ids)])
        if channel:
            channel._send_history_message(pid, page_history)
        return True

    @http.route('/im_livechat/notify_typing', type='json', auth='public', cors="*")
    def notify_typing(self, uuid, is_typing):
        """ Broadcast the typing notification of the website user to other channel members
            :param uuid: (string) the UUID of the livechat channel
            :param is_typing: (boolean) tells whether the website user is typing or not.
        """
        Channel = request.env['mail.channel']
        channel = Channel.sudo().search([('uuid', '=', uuid)], limit=1)
        channel.notify_typing(is_typing=is_typing, is_website_user=True)

    @http.route('/im_livechat/email_livechat_transcript', type='json', auth='public', cors="*")
    def email_livechat_transcript(self, uuid, email):
        channel = request.env['mail.channel'].sudo().search([('uuid', '=', uuid)], limit=1)
        if channel:
            channel._email_livechat_transcript(email)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
from . import main

```

## File: data\im_livechat_channel_data.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">
        <record id="im_livechat_channel_data" model="im_livechat.channel">
            <field name="name">YourWebsite.com</field>
            <field name="default_message">Hello, how may I help you?</field>
        </record>
    </data>
</odoo>

```

## File: data\im_livechat_channel_demo.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record id="im_livechat_channel_rule_demo" model="im_livechat.channel.rule">
            <field name="regex_url">/im_livechat/</field>
            <field name="action">auto_popup</field>
            <field name="auto_popup_timer">3</field>
            <field name="channel_id" ref="im_livechat_channel_data" />
        </record>

        <record id="im_livechat.im_livechat_group_user" model="res.groups">
            <field name="users" eval="[(4, ref('base.user_demo'))]"/>
        </record>

        <record id="im_livechat_channel_data" model="im_livechat.channel">
             <field name="user_ids" eval="[(4, ref('base.user_admin'))]"/>
        </record>
    </data>
</odoo>

```

## File: data\mail_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <template id="livechat_email_template">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 24px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="100%" style="background-color: white; padding: 0; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Livechat Conversation</span><br/>
                    <span style="font-size: 20px; font-weight: bold;">
                        <t t-esc="company.name"/>
                    </span>
                </td><td valign="middle" align="right">
                    <img t-att-src="'/logo.png?company=%s' % company.id" style="padding: 0px; margin: 0px; height: 48px;" t-att-alt="'%s' % company.name"/>
                </td></tr>
                <tr><td colspan="2" style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin:4px 0px 32px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- CONTENT -->
    <t t-set="top" t-value="'border-top: thin solid #dee2e6;'" />
    <t t-set="bottom" t-value="'border-bottom: thin solid #dee2e6;'" />
    <t t-set="right" t-value="'border-right: thin solid #dee2e6;'" />
    <t t-set="left" t-value="'border-left: thin solid #dee2e6;'" />
    <tr>
        <td style="padding: 0 50px;">
            <div style="font-size: 13px; padding: 10px 0;">
                <span>Hello,</span><br />Here's a copy of your conversation with
                <span t-field="channel.livechat_operator_id.name"/>, on the
                <span t-field="channel.livechat_channel_id.create_date"/>
            </div>
            <table cellspacing="0" cellpadding="0" style="width:100%; border-collapse: collapse;">
                <t t-foreach="channel.message_ids.sorted(key=lambda m: m.date)" t-as="message" >
                    <t t-set="author_name" t-value="message.author_id.name if message.author_id else 'You'" />
                    <tr>
                        <td valign="top" align="center" rowspan="2" t-att-style="'width: 70px;' + top + bottom + left">
                            <t t-if="message.author_avatar">
                                <img  t-attf-alt="{{author_name}}" style="width: 64px; height: 64px; object-fit: cover;" t-attf-src="data:image/png;base64,{{message.author_avatar}}"/>
                            </t>
                            <t t-else="">
                                <img  t-attf-alt="{{author_name}}" style="width: 64px; height: 64px; object-fit: cover;" src="/mail/static/src/img/smiley/avatar.jpg"/>
                            </t>
                        </td>
                        <td t-att-style="'padding-left: 5px; margin: 0px;' + top">
                            <strong t-esc="author_name"/>
                        </td>
                        <td  t-att-style="'font-size: 13px; padding: 5px;' + top + right" align="right"><span t-field="message.date"/></td>
                    </tr>
                    <tr>
                        <td valign="top" colspan="2" t-att-style="'padding-left: 5px;' + bottom + right">
                            <span t-field="message.body" style="font-size: 13px;"/>
                        </td>
                    </tr>
                </t>
            </table>
            <div style="font-size: 13px; padding: 30px 0;">
                <span>Best regards,</span><br /><br />
                <span t-field="company.name"/>
            </div>
        </td>
    </tr>
    <!-- FOOTER -->
    <tr>
        <td align="center" style="min-width: 590px; padding: 0 8px 0 8px; font-size:11px;">
            <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 4px 0px;"/>
            <b t-esc="company.name"/><br/>
            <div style="color: #999999;">
                <t t-esc="company.phone"/>
                <t t-if="company.email"> |
                    <a t-att-href="'mailto:%s' % company.email" style="text-decoration:none; color: #999999;"><t t-esc="company.email"/></a>
                </t>
                <t t-if="company.website"> |
                    <a t-att-href="'%s' % company.website" style="text-decoration:none; color: #999999;">
                        <t t-esc="company.website"/>
                    </a>
                </t>
            </div>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- POWERED BY -->
<tr><td align="center" style="min-width: 590px;">
        Powered by <a target="_blank" href="https://www.odoo.com?utm_source=db&amp;utm_medium=email" style="color: #875A7B;">Odoo</a>
</td></tr>
</table>
        </template>
    </data>
</odoo>

```

## File: data\mail_shortcode_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <record id="mail_shortcode_data_hello" model="mail.shortcode">
        <field name="source">hello</field>
        <field name="substitution">Hello. How may I help you?</field>
    </record>
</data></odoo>
```

## File: data\mail_shortcode_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="mail_canned_response_bye" model="mail.shortcode">
            <field name="source">bye</field>
            <field name="substitution">Thanks for your feedback. Good bye!</field>
        </record>

    </data>
</odoo>

```

## File: models\im_livechat_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64
import random
import re

from odoo import api, fields, models, modules, _


class ImLivechatChannel(models.Model):
    """ Livechat Channel
        Define a communication channel, which can be accessed with 'script_external' (script tag to put on
        external website), 'script_internal' (code to be integrated with odoo website) or via 'web_page' link.
        It provides rating tools, and access rules for anonymous people.
    """

    _name = 'im_livechat.channel'
    _inherit = ['rating.parent.mixin']
    _description = 'Livechat Channel'
    _rating_satisfaction_days = 7  # include only last 7 days to compute satisfaction

    def _default_image(self):
        image_path = modules.get_module_resource('im_livechat', 'static/src/img', 'default.png')
        return base64.b64encode(open(image_path, 'rb').read())

    def _default_user_ids(self):
        return [(6, 0, [self._uid])]

    # attribute fields
    name = fields.Char('Name', required=True, help="The name of the channel")
    button_text = fields.Char('Text of the Button', default='Have a Question? Chat with us.',
        help="Default text displayed on the Livechat Support Button")
    default_message = fields.Char('Welcome Message', default='How may I help you?',
        help="This is an automated 'welcome' message that your visitor will see when they initiate a new conversation.")
    input_placeholder = fields.Char('Chat Input Placeholder', help='Text that prompts the user to initiate the chat.')

    # computed fields
    web_page = fields.Char('Web Page', compute='_compute_web_page_link', store=False, readonly=True,
        help="URL to a static page where you client can discuss with the operator of the channel.")
    are_you_inside = fields.Boolean(string='Are you inside the matrix?',
        compute='_are_you_inside', store=False, readonly=True)
    script_external = fields.Text('Script (external)', compute='_compute_script_external', store=False, readonly=True)
    nbr_channel = fields.Integer('Number of conversation', compute='_compute_nbr_channel', store=False, readonly=True)

    image_128 = fields.Image("Image", max_width=128, max_height=128, default=_default_image)

    # relationnal fields
    user_ids = fields.Many2many('res.users', 'im_livechat_channel_im_user', 'channel_id', 'user_id', string='Operators', default=_default_user_ids)
    channel_ids = fields.One2many('mail.channel', 'livechat_channel_id', 'Sessions')
    rule_ids = fields.One2many('im_livechat.channel.rule', 'channel_id', 'Rules')

    def _are_you_inside(self):
        for channel in self:
            channel.are_you_inside = bool(self.env.uid in [u.id for u in channel.user_ids])

    def _compute_script_external(self):
        view = self.env['ir.model.data'].get_object('im_livechat', 'external_loader')
        values = {
            "url": self.env['ir.config_parameter'].sudo().get_param('web.base.url'),
            "dbname": self._cr.dbname,
        }
        for record in self:
            values["channel_id"] = record.id
            record.script_external = view.render(values)

    def _compute_web_page_link(self):
        base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        for record in self:
            record.web_page = "%s/im_livechat/support/%i" % (base_url, record.id)

    @api.depends('channel_ids')
    def _compute_nbr_channel(self):
        data = self.env['mail.channel'].read_group([('livechat_channel_id', 'in', self._ids)], ['__count'], ['livechat_channel_id'], lazy=False)
        channel_count = {x['livechat_channel_id'][0]: x['__count'] for x in data}
        for record in self:
            record.nbr_channel = channel_count.get(record.id, 0)

    # --------------------------
    # Action Methods
    # --------------------------
    def action_join(self):
        self.ensure_one()
        return self.write({'user_ids': [(4, self._uid)]})

    def action_quit(self):
        self.ensure_one()
        return self.write({'user_ids': [(3, self._uid)]})

    def action_view_rating(self):
        """ Action to display the rating relative to the channel, so all rating of the
            sessions of the current channel
            :returns : the ir.action 'action_view_rating' with the correct domain
        """
        self.ensure_one()
        action = self.env['ir.actions.act_window'].for_xml_id('im_livechat', 'rating_rating_action_view_livechat_rating')
        action['domain'] = [('parent_res_id', '=', self.id), ('parent_res_model', '=', 'im_livechat.channel')]
        return action

    # --------------------------
    # Channel Methods
    # --------------------------
    def _get_available_users(self):
        """ get available user of a given channel
            :retuns : return the res.users having their im_status online
        """
        self.ensure_one()
        return self.user_ids.filtered(lambda user: user.im_status == 'online')

    def _get_livechat_mail_channel_vals(self, anonymous_name, operator, user_id=None, country_id=None):
        # partner to add to the mail.channel
        operator_partner_id = operator.partner_id.id
        channel_partner_to_add = [(4, operator_partner_id)]
        visitor_user = False
        if user_id:
            visitor_user = self.env['res.users'].browse(user_id)
            if visitor_user and visitor_user.active:  # valid session user (not public)
                channel_partner_to_add.append((4, visitor_user.partner_id.id))
        return {
            'channel_partner_ids': channel_partner_to_add,
            'livechat_operator_id': operator_partner_id,
            'livechat_channel_id': self.id,
            'anonymous_name': False if user_id else anonymous_name,
            'country_id': country_id,
            'channel_type': 'livechat',
            'name': ' '.join([visitor_user.display_name if visitor_user else anonymous_name, operator.livechat_username if operator.livechat_username else operator.name]),
            'public': 'private',
            'email_send': False,
        }

    def _open_livechat_mail_channel(self, anonymous_name, previous_operator_id=None, user_id=None, country_id=None):
        """ Return a mail.channel given a livechat channel. It creates one with a connected operator, or return false otherwise
            :param anonymous_name : the name of the anonymous person of the channel
            :param previous_operator_id : partner_id.id of the previous operator that this visitor had in the past
            :param user_id : the id of the logged in visitor, if any
            :param country_code : the country of the anonymous person of the channel
            :type anonymous_name : str
            :return : channel header
            :rtype : dict

            If this visitor already had an operator within the last 7 days (information stored with the 'im_livechat_previous_operator_pid' cookie),
            the system will first try to assign that operator if he's available (to improve user experience).
        """
        self.ensure_one()
        operator = False
        if previous_operator_id:
            available_users = self._get_available_users()
            # previous_operator_id is the partner_id of the previous operator, need to convert to user
            if previous_operator_id in available_users.mapped('partner_id').ids:
                operator = next(available_user for available_user in available_users if available_user.partner_id.id == previous_operator_id)
        if not operator:
            operator = self._get_random_operator()
        if not operator:
            # no one available
            return False

        # create the session, and add the link with the given channel
        mail_channel_vals = self._get_livechat_mail_channel_vals(anonymous_name, operator, user_id=user_id, country_id=country_id)
        mail_channel = self.env["mail.channel"].with_context(mail_create_nosubscribe=False).sudo().create(mail_channel_vals)
        mail_channel._broadcast([operator.partner_id.id])
        return mail_channel.sudo().channel_info()[0]

    def _get_random_operator(self):
        """ Return a random operator from the available users of the channel that have the lowest number of active livechats.
        A livechat is considered 'active' if it has at least one message within the 30 minutes.

        (Some annoying conversions have to be made on the fly because this model holds 'res.users' as available operators
        and the mail_channel model stores the partner_id of the randomly selected operator)

        :return : user
        :rtype : res.users
        """
        operators = self._get_available_users()
        if len(operators) == 0:
            return False

        self.env.cr.execute("""SELECT COUNT(DISTINCT c.id), c.livechat_operator_id
            FROM mail_channel c
            LEFT OUTER JOIN mail_message_mail_channel_rel r ON c.id = r.mail_channel_id
            LEFT OUTER JOIN mail_message m ON r.mail_message_id = m.id
            WHERE m.create_date > ((now() at time zone 'UTC') - interval '30 minutes')
            AND c.channel_type = 'livechat'
            AND c.livechat_operator_id in %s
            GROUP BY c.livechat_operator_id
            ORDER BY COUNT(DISTINCT c.id) asc""", (tuple(operators.mapped('partner_id').ids),))
        active_channels = self.env.cr.dictfetchall()

        # If inactive operator(s), return one of them
        active_channel_operator_ids = [active_channel['livechat_operator_id'] for active_channel in active_channels]
        inactive_operators = [operator for operator in operators if operator.partner_id.id not in active_channel_operator_ids]
        if inactive_operators:
            return random.choice(inactive_operators)

        # If no inactive operator, active_channels is not empty as len(operators) > 0 (see above).
        # Get the less active operator using the active_channels first element's count (since they are sorted 'ascending')
        lowest_number_of_conversations = active_channels[0]['count']
        less_active_operator = random.choice([
            active_channel['livechat_operator_id'] for active_channel in active_channels
            if active_channel['count'] == lowest_number_of_conversations])

        # convert the selected 'partner_id' to its corresponding res.users
        return next(operator for operator in operators if operator.partner_id.id == less_active_operator)

    def _get_channel_infos(self):
        self.ensure_one()

        return {
            'button_text': self.button_text,
            'input_placeholder': self.input_placeholder,
            'default_message': self.default_message,
            "channel_name": self.name,
            "channel_id": self.id,
        }

    def get_livechat_info(self, username='Visitor'):
        self.ensure_one()

        if username == 'Visitor':
            username = _('Visitor')
        info = {}
        info['available'] = len(self._get_available_users()) > 0
        info['server_url'] = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        if info['available']:
            info['options'] = self._get_channel_infos()
            info['options']["default_username"] = username
        return info


class ImLivechatChannelRule(models.Model):
    """ Channel Rules
        Rules defining access to the channel (countries, and url matching). It also provide the 'auto pop'
        option to open automatically the conversation.
    """

    _name = 'im_livechat.channel.rule'
    _description = 'Livechat Channel Rules'
    _order = 'sequence asc'

    regex_url = fields.Char('URL Regex',
        help="Regular expression specifying the web pages this rule will be applied on.")
    action = fields.Selection([('display_button', 'Display the button'), ('auto_popup', 'Auto popup'), ('hide_button', 'Hide the button')],
        string='Action', required=True, default='display_button',
        help="* 'Display the button' displays the chat button on the pages.\n"\
             "* 'Auto popup' displays the button and automatically open the conversation pane.\n"\
             "* 'Hide the button' hides the chat button on the pages.")
    auto_popup_timer = fields.Integer('Auto popup timer', default=0,
        help="Delay (in seconds) to automatically open the conversation window. Note: the selected action must be 'Auto popup' otherwise this parameter will not be taken into account.")
    channel_id = fields.Many2one('im_livechat.channel', 'Channel',
        help="The channel of the rule")
    country_ids = fields.Many2many('res.country', 'im_livechat_channel_country_rel', 'channel_id', 'country_id', 'Country',
        help="The rule will only be applied for these countries. Example: if you select 'Belgium' and 'United States' and that you set the action to 'Hide Button', the chat button will be hidden on the specified URL from the visitors located in these 2 countries. This feature requires GeoIP installed on your server.")
    sequence = fields.Integer('Matching order', default=10,
        help="Given the order to find a matching rule. If 2 rules are matching for the given url/country, the one with the lowest sequence will be chosen.")

    def match_rule(self, channel_id, url, country_id=False):
        """ determine if a rule of the given channel matches with the given url
            :param channel_id : the identifier of the channel_id
            :param url : the url to match with a rule
            :param country_id : the identifier of the country
            :returns the rule that matches the given condition. False otherwise.
            :rtype : im_livechat.channel.rule
        """
        def _match(rules):
            for rule in rules:
                # url might not be set because it comes from referer, in that
                # case match the first rule with no regex_url
                if re.search(rule.regex_url or '', url or ''):
                    return rule
            return False
        # first, search the country specific rules (the first match is returned)
        if country_id: # don't include the country in the research if geoIP is not installed
            domain = [('country_ids', 'in', [country_id]), ('channel_id', '=', channel_id)]
            rule = _match(self.search(domain))
            if rule:
                return rule
        # second, fallback on the rules without country
        domain = [('country_ids', '=', False), ('channel_id', '=', channel_id)]
        return _match(self.search(domain))

```

## File: models\ir_autovacuum.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, models

class AutoVacuum(models.AbstractModel):
    _inherit = 'ir.autovacuum'

    @api.model
    def power_on(self, *args, **kwargs):
        self.env['mail.channel'].remove_empty_livechat_sessions()
        self.env['mail.channel.partner'].unpin_old_livechat_sessions()
        return super(AutoVacuum, self).power_on(*args, **kwargs)

```

## File: models\mail_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools import html_escape

class ChannelPartner(models.Model):
    _inherit = 'mail.channel.partner'

    @api.model
    def unpin_old_livechat_sessions(self):
        """ Unpin livechat sessions with no activity for at least one day to
            clean the operator's interface """
        self.env.cr.execute("""
            UPDATE mail_channel_partner
            SET is_pinned = false
            WHERE id in (
                SELECT cp.id FROM mail_channel_partner cp
                INNER JOIN mail_channel c on c.id = cp.channel_id
                WHERE c.channel_type = 'livechat' AND cp.is_pinned is true AND
                    cp.write_date < current_timestamp - interval '1 day'
            )
        """)


class MailChannel(models.Model):
    """ Chat Session
        Reprensenting a conversation between users.
        It extends the base method for anonymous usage.
    """

    _name = 'mail.channel'
    _inherit = ['mail.channel', 'rating.mixin']

    anonymous_name = fields.Char('Anonymous Name')
    channel_type = fields.Selection(selection_add=[('livechat', 'Livechat Conversation')])
    livechat_channel_id = fields.Many2one('im_livechat.channel', 'Channel')
    livechat_operator_id = fields.Many2one('res.partner', string='Operator', help="""Operator for this specific channel""")
    country_id = fields.Many2one('res.country', string="Country", help="Country of the visitor of the channel")

    _sql_constraints = [('livechat_operator_id', "CHECK((channel_type = 'livechat' and livechat_operator_id is not null) or (channel_type != 'livechat'))",
                         'Livechat Operator ID is required for a channel of type livechat.')]

    def _compute_is_chat(self):
        super(MailChannel, self)._compute_is_chat()
        for record in self:
            if record.channel_type == 'livechat':
                record.is_chat = True

    def _channel_message_notifications(self, message, message_format=False):
        """ When a anonymous user create a mail.channel, the operator is not notify (to avoid massive polling when
            clicking on livechat button). So when the anonymous person is sending its FIRST message, the channel header
            should be added to the notification, since the user cannot be listining to the channel.
        """
        livechat_channels = self.filtered(lambda x: x.channel_type == 'livechat')
        other_channels = self.filtered(lambda x: x.channel_type != 'livechat')
        notifications = super(MailChannel, livechat_channels)._channel_message_notifications(message.with_context(im_livechat_use_username=True)) + \
                        super(MailChannel, other_channels)._channel_message_notifications(message, message_format)
        for channel in self:
            # add uuid for private livechat channels to allow anonymous to listen
            if channel.channel_type == 'livechat' and channel.public == 'private':
                notifications.append([channel.uuid, notifications[0][1]])
        if not message.author_id:
            unpinned_channel_partner = self.mapped('channel_last_seen_partner_ids').filtered(lambda cp: not cp.is_pinned)
            if unpinned_channel_partner:
                unpinned_channel_partner.write({'is_pinned': True})
                notifications = self._channel_channel_notifications(unpinned_channel_partner.mapped('partner_id').ids) + notifications
        return notifications

    def channel_fetch_message(self, last_id=False, limit=20):
        """ Override to add the context of the livechat username."""
        channel = self.with_context(im_livechat_use_username=True) if self.channel_type == 'livechat' else self
        return super(MailChannel, channel).channel_fetch_message(last_id=last_id, limit=limit)

    def channel_info(self, extra_info=False):
        """ Extends the channel header by adding the livechat operator and the 'anonymous' profile
            :rtype : list(dict)
        """
        channel_infos = super(MailChannel, self).channel_info(extra_info)
        channel_infos_dict = dict((c['id'], c) for c in channel_infos)
        for channel in self:
            # add the last message date
            if channel.channel_type == 'livechat':
                # add the operator id
                if channel.livechat_operator_id:
                    res = channel.livechat_operator_id.with_context(im_livechat_use_username=True).name_get()[0]
                    channel_infos_dict[channel.id]['operator_pid'] = (res[0], res[1].replace(',', ''))
                # add the anonymous or partner name
                channel_infos_dict[channel.id]['correspondent_name'] = channel._channel_get_livechat_partner_name()
                last_msg = self.env['mail.message'].search([("channel_ids", "in", [channel.id])], limit=1)
                if last_msg:
                    channel_infos_dict[channel.id]['last_message_date'] = last_msg.date
        return list(channel_infos_dict.values())

    @api.model
    def channel_fetch_slot(self):
        values = super(MailChannel, self).channel_fetch_slot()
        pinned_channels = self.env['mail.channel.partner'].search([('partner_id', '=', self.env.user.partner_id.id), ('is_pinned', '=', True)]).mapped('channel_id')
        values['channel_livechat'] = self.search([('channel_type', '=', 'livechat'), ('id', 'in', pinned_channels.ids)]).channel_info()
        return values

    def _channel_get_livechat_partner_name(self):
        if self.livechat_operator_id in self.channel_partner_ids:
            partners = self.channel_partner_ids - self.livechat_operator_id
            if partners:
                partner_name = False
                for partner in partners:
                    if not partner_name:
                        partner_name = partner.name
                    else:
                        partner_name += ', %s' % partner.name
                    if partner.country_id:
                        partner_name += ' (%s)' % partner.country_id.name
                return partner_name
        if self.anonymous_name:
            return self.anonymous_name
        return _("Visitor")

    @api.model
    def remove_empty_livechat_sessions(self):
        hours = 1  # never remove empty session created within the last hour
        self.env.cr.execute("""
            SELECT id as id
            FROM mail_channel C
            WHERE NOT EXISTS (
                SELECT *
                FROM mail_message_mail_channel_rel R
                WHERE R.mail_channel_id = C.id
            ) AND C.channel_type = 'livechat' AND livechat_channel_id IS NOT NULL AND
                COALESCE(write_date, create_date, (now() at time zone 'UTC'))::timestamp
                < ((now() at time zone 'UTC') - interval %s)""", ("%s hours" % hours,))
        empty_channel_ids = [item['id'] for item in self.env.cr.dictfetchall()]
        self.browse(empty_channel_ids).unlink()

    def _define_command_history(self):
        return {
            'channel_types': ['livechat'],
            'help': _('See 15 last visited pages')
        }

    def _execute_command_history(self, **kwargs):
        notification = []
        notification_values = {
            '_type': 'history_command',
        }
        notification.append([self.uuid, dict(notification_values)])
        return self.env['bus.bus'].sendmany(notification)

    def _send_history_message(self, pid, page_history):
        message_body = _('No history found')
        if page_history:
            html_links = ['<li><a href="%s" target="_blank">%s</a></li>' % (html_escape(page), html_escape(page)) for page in page_history]
            message_body = '<span class="o_mail_notification"><ul>%s</ul></span>' % (''.join(html_links))
        self.env['bus.bus'].sendone((self._cr.dbname, 'res.partner', pid), {
            'body': message_body,
            'channel_ids': self.ids,
            'info': 'transient_message',
        })

    # Rating Mixin

    def _rating_get_parent_field_name(self):
        return 'livechat_channel_id'

    def _email_livechat_transcript(self, email):
        company = self.env.user.company_id
        render_context = {
            "company": company,
            "channel": self,
        }
        template = self.env.ref('im_livechat.livechat_email_template')
        mail_body = template.render(render_context, engine='ir.qweb', minimal_qcontext=True)
        mail_body = self.env['mail.thread']._replace_local_links(mail_body)
        mail = self.env['mail.mail'].create({
            'subject': _('Conversation with %s') % self.livechat_operator_id.name,
            'email_from': self.env.company.email,
            'email_to': email,
            'body_html': mail_body,
        })
        mail.send()

```

## File: models\rating.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class Rating(models.Model):

    _inherit = "rating.rating"

    @api.depends('res_model', 'res_id')
    def _compute_res_name(self):
        for rating in self:
            # cannot change the rec_name of session since it is use to create the bus channel
            # so, need to override this method to set the same alternative rec_name as in reporting
            if rating.res_model == 'mail.channel':
                current_object = self.env[rating.res_model].sudo().browse(rating.res_id)
                rating.res_name = ('%s / %s') % (current_object.livechat_channel_id.name, current_object.id)
            else:
                super(Rating, rating)._compute_res_name()

    def action_open_rated_object(self):
        action = super(Rating, self).action_open_rated_object()
        if self.res_model == 'mail.channel':
            view_id = self.env.ref('im_livechat.mail_channel_view_form').id
            action['views'] = [[view_id, 'form']]
        return action

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api


class Partners(models.Model):
    """ Update of res.partners class
        - override name_get to take into account the livechat username
    """
    _inherit = 'res.partner'

    def name_get(self):
        if self.env.context.get('im_livechat_use_username'):
            # process the ones with livechat username
            users_with_livechatname = self.env['res.users'].search([('partner_id', 'in', self.ids), ('livechat_username', '!=', False)])
            map_with_livechatname = {}
            for user in users_with_livechatname:
                map_with_livechatname[user.partner_id.id] = user.livechat_username

            # process the ones without livecaht username
            partner_without_livechatname = self - users_with_livechatname.mapped('partner_id')
            no_livechatname_name_get = super(Partners, partner_without_livechatname).name_get()
            map_without_livechatname = dict(no_livechatname_name_get)

            # restore order
            result = []
            for partner in self:
                name = map_with_livechatname.get(partner.id)
                if not name:
                    name = map_without_livechatname.get(partner.id)
                result.append((partner.id, name))
        else:
            result = super(Partners, self).name_get()
        return result

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class Users(models.Model):
    """ Update of res.users class
        - add a preference about username for livechat purpose
    """
    _inherit = 'res.users'

    livechat_username = fields.Char("Livechat Username", help="This username will be used as your name in the livechat channels.")

    def __init__(self, pool, cr):
        """ Override of __init__ to add access rights on livechat_username
            Access rights are disabled by default, but allowed
            on some specific fields defined in self.SELF_{READ/WRITE}ABLE_FIELDS.
        """
        init_res = super(Users, self).__init__(pool, cr)
        # duplicate list to avoid modifying the original reference
        type(self).SELF_WRITEABLE_FIELDS = list(self.SELF_WRITEABLE_FIELDS)
        type(self).SELF_WRITEABLE_FIELDS.extend(['livechat_username'])
        # duplicate list to avoid modifying the original reference
        type(self).SELF_READABLE_FIELDS = list(self.SELF_READABLE_FIELDS)
        type(self).SELF_READABLE_FIELDS.extend(['livechat_username'])
        return init_res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*
from . import res_users
from . import res_partner
from . import im_livechat_channel
from . import ir_autovacuum
from . import mail_channel
from . import rating

```

## File: report\im_livechat_report_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools


class ImLivechatReportChannel(models.Model):
    """ Livechat Support Report on the Channels """

    _name = "im_livechat.report.channel"
    _description = "Livechat Support Channel Report"
    _order = 'start_date, technical_name'
    _auto = False

    uuid = fields.Char('UUID', readonly=True)
    channel_id = fields.Many2one('mail.channel', 'Conversation', readonly=True)
    channel_name = fields.Char('Channel Name', readonly=True)
    technical_name = fields.Char('Code', readonly=True)
    livechat_channel_id = fields.Many2one('im_livechat.channel', 'Channel', readonly=True)
    start_date = fields.Datetime('Start Date of session', readonly=True, help="Start date of the conversation")
    start_hour = fields.Char('Start Hour of session', readonly=True, help="Start hour of the conversation")
    day_number = fields.Char('Day Number', readonly=True, help="Day number of the session (1 is Monday, 7 is Sunday)")
    time_to_answer = fields.Float('Time to answer (sec)', digits=(16, 2), readonly=True, group_operator="avg", help="Average time in seconds to give the first answer to the visitor")
    start_date_hour = fields.Char('Hour of start Date of session', readonly=True)
    duration = fields.Float('Average duration', digits=(16, 2), readonly=True, group_operator="avg", help="Duration of the conversation (in seconds)")
    nbr_speaker = fields.Integer('# of speakers', readonly=True, group_operator="avg", help="Number of different speakers")
    nbr_message = fields.Integer('Average message', readonly=True, group_operator="avg", help="Number of message in the conversation")
    is_without_answer = fields.Integer('Session(s) without answer', readonly=True, group_operator="sum",
                                       help="""A session is without answer if the operator did not answer. 
                                       If the visitor is also the operator, the session will always be answered.""")
    days_of_activity = fields.Integer('Days of activity', group_operator="max", readonly=True, help="Number of days since the first session of the operator")
    is_anonymous = fields.Integer('Is visitor anonymous', readonly=True)
    country_id = fields.Many2one('res.country', 'Country of the visitor', readonly=True)
    is_happy = fields.Integer('Visitor is Happy', readonly=True)
    rating = fields.Integer('Rating', group_operator="avg", readonly=True)
    # TODO DBE : Use Selection field - Need : Pie chart must show labels, not keys.
    rating_text = fields.Char('Satisfaction Rate', readonly=True)
    is_unrated = fields.Integer('Session not rated', readonly=True)
    partner_id = fields.Many2one('res.partner', 'Operator', readonly=True)

    def init(self):
        # Note : start_date_hour must be remove when the read_group will allow grouping on the hour of a datetime. Don't forget to change the view !
        tools.drop_view_if_exists(self.env.cr, 'im_livechat_report_channel')
        self.env.cr.execute("""
            CREATE OR REPLACE VIEW im_livechat_report_channel AS (
                SELECT
                    C.id as id,
                    C.uuid as uuid,
                    C.id as channel_id,
                    C.name as channel_name,
                    CONCAT(L.name, ' / ', C.id) as technical_name,
                    C.livechat_channel_id as livechat_channel_id,
                    C.create_date as start_date,
                    to_char(date_trunc('hour', C.create_date), 'YYYY-MM-DD HH24:MI:SS') as start_date_hour,
                    to_char(date_trunc('hour', C.create_date), 'HH24') as start_hour,
                    extract(dow from  C.create_date) as day_number, 
                    EXTRACT('epoch' FROM MAX(M.create_date) - MIN(M.create_date)) AS duration,
                    EXTRACT('epoch' FROM MIN(MO.create_date) - MIN(M.create_date)) AS time_to_answer,
                    count(distinct C.livechat_operator_id) as nbr_speaker,
                    count(distinct M.id) as nbr_message,
                    CASE 
                        WHEN EXISTS (select distinct M.author_id FROM mail_message M, mail_message_mail_channel_rel R 
                                        WHERE M.author_id=C.livechat_operator_id AND R.mail_channel_id = C.id 
                                        AND R.mail_message_id = M.id and C.livechat_operator_id = M.author_id)
                        THEN 0
                        ELSE 1
                    END as is_without_answer,
                    (DATE_PART('day', date_trunc('day', now()) - date_trunc('day', C.create_date)) + 1) as days_of_activity,
                    CASE
                        WHEN C.anonymous_name IS NULL THEN 0
                        ELSE 1
                    END as is_anonymous,
                    C.country_id,
                    CASE 
                        WHEN rate.rating = 10 THEN 1
                        ELSE 0
                    END as is_happy,
                    Rate.rating as rating,
                    CASE
                        WHEN Rate.rating = 1 THEN 'Unhappy'
                        WHEN Rate.rating = 10 THEN 'Happy'
                        WHEN Rate.rating = 5 THEN 'Neutral'
                        ELSE null
                    END as rating_text,
                    CASE 
                        WHEN rate.rating > 0 THEN 0
                        ELSE 1
                    END as is_unrated,
                    C.livechat_operator_id as partner_id
                FROM mail_channel C
                    JOIN mail_message_mail_channel_rel R ON (C.id = R.mail_channel_id)
                    JOIN mail_message M ON (M.id = R.mail_message_id)
                    JOIN im_livechat_channel L ON (L.id = C.livechat_channel_id)
                    LEFT JOIN mail_message MO ON (R.mail_message_id = MO.id AND MO.author_id = C.livechat_operator_id)
                    LEFT JOIN rating_rating Rate ON (Rate.res_id = C.id and Rate.res_model = 'mail.channel' and Rate.parent_res_model = 'im_livechat.channel')
                    WHERE C.livechat_operator_id is not null
                GROUP BY C.livechat_operator_id, C.id, C.name, C.livechat_channel_id, L.name, C.create_date, C.uuid, Rate.rating
            )
        """)

```

## File: report\im_livechat_report_channel_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>

        <record id="im_livechat_report_channel_view_pivot" model="ir.ui.view">
            <field name="name">im_livechat.report.channel.pivot</field>
            <field name="model">im_livechat.report.channel</field>
            <field name="arch" type="xml">
                <pivot string="Livechat Support Statistics" disable_linking="True">
                    <field name="technical_name" type="row"/>
                    <field name="duration" type="measure"/>
                    <field name="nbr_message" type="measure"/>
                </pivot>
            </field>
        </record>

        <record id="im_livechat_report_channel_view_graph" model="ir.ui.view">
            <field name="name">im_livechat.report.channel.graph</field>
            <field name="model">im_livechat.report.channel</field>
            <field name="arch" type="xml">
                <graph string="Livechat Support Statistics">
                    <field name="technical_name"/>
                    <field name="nbr_message" type="measure"/>
                </graph>
            </field>
        </record>

        <record id="im_livechat_report_channel_view_search" model="ir.ui.view">
            <field name="name">im_livechat.report.channel.search</field>
            <field name="model">im_livechat.report.channel</field>
            <field name="arch" type="xml">
                <search string="Search report">
                    <field name="channel_name"/>
                    <filter name="missed_session" string="Missed sessions" domain="[('nbr_speaker','&lt;=', 1)]"/>
                    <filter name="treated_session" string="Treated sessions" domain="[('nbr_speaker','&gt;', 1)]"/>
                    <filter name="last_24h" string="Last 24h" domain="[('start_date','&gt;', (context_today() - datetime.timedelta(days=1)).strftime('%%Y-%%m-%%d') )]"/>
                    <filter name="start_date_filter" string="This Week" domain="[
                        ('start_date', '>=', (datetime.datetime.combine(context_today() + relativedelta(weeks=-1,days=1,weekday=0), datetime.time(0,0,0)).to_utc()).strftime('%Y-%m-%d %H:%M:%S')),
                        ('start_date', '&lt;', (datetime.datetime.combine(context_today() + relativedelta(days=1,weekday=0), datetime.time(0,0,0)).to_utc()).strftime('%Y-%m-%d %H:%M:%S'))]"/>
                    <group expand="0" string="Group By...">
                        <filter name="group_by_session" string="Code" domain="[]" context="{'group_by':'technical_name'}"/>
                        <filter name="group_by_channel" string="Channel" domain="[]" context="{'group_by':'channel_id'}"/>
                        <filter name="group_by_operator" string="Operator" domain="[('partner_id','!=', False)]" context="{'group_by':'partner_id'}"/>
                        <separator orientation="vertical" />
                        <filter name="group_by_hour" string="Creation date (hour)" domain="[]" context="{'group_by':'start_date_hour'}"/>
                        <filter name="group_by_month" string="Creation date" domain="[]" context="{'group_by':'start_date:month'}" />
                    </group>
                </search>
            </field>
        </record>

        <record id="im_livechat_report_channel_action" model="ir.actions.act_window">
            <field name="name">Session Statistics</field>
            <field name="res_model">im_livechat.report.channel</field>
            <field name="view_mode">graph,pivot</field>
            <field name="context">{"search_default_last_week":1}</field>
            <field name="help">Livechat Support Channel Statistics allows you to easily check and analyse your company livechat session performance. Extract information about the missed sessions, the audiance, the duration of a session, etc.</field>
        </record>

        <menuitem
            id="menu_reporting_livechat_channel"
            name="Session Statistics"
            parent="menu_reporting_livechat"
            sequence="10"
            action="im_livechat_report_channel_action"/>


    </data>
</odoo>

```

## File: report\im_livechat_report_operator.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools


class ImLivechatReportOperator(models.Model):
    """ Livechat Support Report on the Operator """

    _name = "im_livechat.report.operator"
    _description = "Livechat Support Operator Report"
    _order = 'livechat_channel_id, partner_id'
    _auto = False

    partner_id = fields.Many2one('res.partner', 'Operator', readonly=True)
    livechat_channel_id = fields.Many2one('im_livechat.channel', 'Channel', readonly=True)
    nbr_channel = fields.Integer('# of Sessions', readonly=True, group_operator="sum", help="Number of conversation")
    channel_id = fields.Many2one('mail.channel', 'Conversation', readonly=True)
    start_date = fields.Datetime('Start Date of session', readonly=True, help="Start date of the conversation")
    time_to_answer = fields.Float('Time to answer', digits=(16, 2), readonly=True, group_operator="avg", help="Average time to give the first answer to the visitor")
    duration = fields.Float('Average duration', digits=(16, 2), readonly=True, group_operator="avg", help="Duration of the conversation (in seconds)")

    def init(self):
        # Note : start_date_hour must be remove when the read_group will allow grouping on the hour of a datetime. Don't forget to change the view !
        tools.drop_view_if_exists(self.env.cr, 'im_livechat_report_operator')
        self.env.cr.execute("""
            CREATE OR REPLACE VIEW im_livechat_report_operator AS (
                SELECT
                    row_number() OVER () AS id,
                    C.livechat_operator_id AS partner_id,
                    C.livechat_channel_id AS livechat_channel_id,
                    COUNT(DISTINCT C.id) AS nbr_channel,
                    C.id AS channel_id,
                    C.create_date AS start_date,
                    EXTRACT('epoch' FROM MAX(M.create_date) - MIN(M.create_date)) AS duration,
                    EXTRACT('epoch' FROM MIN(MO.create_date) - MIN(M.create_date)) AS time_to_answer
                FROM mail_channel C
                    JOIN mail_message_mail_channel_rel R ON R.mail_channel_id = C.id
                    JOIN mail_message M ON R.mail_message_id = M.id
                    LEFT JOIN mail_message MO ON (R.mail_message_id = MO.id AND MO.author_id = C.livechat_operator_id)
                WHERE C.livechat_channel_id IS NOT NULL
                GROUP BY C.id, C.livechat_operator_id
            )
        """)

```

## File: report\im_livechat_report_operator_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>

        <record id="im_livechat_report_operator_view_pivot" model="ir.ui.view">
            <field name="name">im_livechat.report.operator.pivot</field>
            <field name="model">im_livechat.report.operator</field>
            <field name="arch" type="xml">
                <pivot string="Livechat Support Statistics" disable_linking="True">
                    <field name="partner_id" type="row"/>
                    <field name="duration" type="measure"/>
                    <field name="nbr_channel" type="measure"/>
                </pivot>
            </field>
        </record>

        <record id="im_livechat_report_operator_view_graph" model="ir.ui.view">
            <field name="name">im_livechat.report.operator.graph</field>
            <field name="model">im_livechat.report.operator</field>
            <field name="arch" type="xml">
                <graph string="Livechat Support Statistics">
                    <field name="partner_id"/>
                    <field name="nbr_channel" type="measure"/>
                </graph>
            </field>
        </record>

        <record id="im_livechat_report_operator_view_search" model="ir.ui.view">
            <field name="name">im_livechat.report.operator.search</field>
            <field name="model">im_livechat.report.operator</field>
            <field name="arch" type="xml">
                <search string="Search report">
                    <field name="partner_id"/>
                    <filter name="last_24h" string="Last 24h" domain="[('start_date','&gt;', (context_today() - datetime.timedelta(days=1)).strftime('%%Y-%%m-%%d') )]"/>
                    <filter name="start_date_filter" string="This Week" domain="[
                        ('start_date', '>=', (datetime.datetime.combine(context_today() + relativedelta(weeks=-1,days=1,weekday=0), datetime.time(0,0,0)).to_utc()).strftime('%Y-%m-%d %H:%M:%S')),
                        ('start_date', '&lt;', (datetime.datetime.combine(context_today() + relativedelta(days=1,weekday=0), datetime.time(0,0,0)).to_utc()).strftime('%Y-%m-%d %H:%M:%S'))]"/>
                    <group expand="0" string="Group By...">
                        <filter name="group_by_channel" string="Channel" domain="[]" context="{'group_by':'channel_id'}"/>
                        <filter name="group_by_operator" string="Operator" domain="[('partner_id','!=', False)]" context="{'group_by':'partner_id'}"/>
                        <separator orientation="vertical" />
                        <filter name="group_by_month" string="Creation date" domain="[]" context="{'group_by':'start_date:month'}" />
                    </group>
                </search>
            </field>
        </record>

        <record id="im_livechat_report_operator_action" model="ir.actions.act_window">
            <field name="name">Operator Analysis</field>
            <field name="res_model">im_livechat.report.operator</field>
            <field name="view_mode">graph,pivot</field>
            <field name="context">{"search_default_last_week":1}</field>
            <field name="help">Livechat Support Channel Statistics allows you to easily check and analyse your company livechat session performance. Extract information about the missed sessions, the audiance, the duration of a session, etc.</field>
        </record>

        <menuitem
            id="menu_reporting_livechat_operator"
            name="Operator Analysis"
            parent="menu_reporting_livechat"
            sequence="10"
            action="im_livechat_report_operator_action"/>

    </data>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
from . import im_livechat_report_channel
from . import im_livechat_report_operator

```

## File: security\im_livechat_channel_security.xml

```xml
<?xml version="1.0"?>
<odoo>

        <record id="module_category_im_livechat" model="ir.module.category">
            <field name="name">Live Chat</field>
            <field name="sequence" eval="20" />
        </record>

        <record id="im_livechat_group_user" model="res.groups">
            <field name="name">User</field>
            <field name="category_id" ref="base.module_category_website_live_chat"/>
            <field name="comment">The user will be able to join support channels.</field>
        </record>

        <record id="im_livechat_group_manager" model="res.groups">
            <field name="name">Administrator</field>
            <field name="comment">The user will be able to delete support channels.</field>
            <field name="category_id" ref="base.module_category_website_live_chat"/>
            <field name="implied_ids" eval="[(4, ref('im_livechat.im_livechat_group_user'))]"/>
            <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        </record>

        <record id="im_livechat_rule_manager_read_all_mail_channel" model="ir.rule">
            <field name="name">Livechat: Administrator: read all livechat channel</field>
            <field name="model_id" ref="model_mail_channel"/>
            <field name="groups" eval="[(4, ref('im_livechat_group_manager'))]"/>
            <field name="domain_force">[('channel_type', '=', 'livechat')]</field>
            <field name="perm_read" eval="True"/>
            <field name="perm_create" eval="False"/>
            <field name="perm_write" eval="False"/>
            <field name="perm_unlink" eval="False"/>
        </record>

    <data noupdate="1">
        <record id="base.default_user" model="res.users">
            <field name="groups_id" eval="[(4,ref('im_livechat.im_livechat_group_manager'))]"/>
        </record>
    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_livechat_channel,im_livechat.channel,model_im_livechat_channel,,1,0,0,0
access_livechat_channel_user,im_livechat.channel.user,model_im_livechat_channel,im_livechat_group_user,1,1,1,0
access_livechat_channel_manager,im_livechat.channel.manager,model_im_livechat_channel,im_livechat_group_manager,1,1,1,1
access_livechat_support_report_channel,im_livechat.report.channel,model_im_livechat_report_channel,im_livechat_group_manager,1,1,1,1
access_livechat_support_report_operator,im_livechat.report.operator,model_im_livechat_report_operator,im_livechat_group_manager,1,1,1,1
access_livechat_channel_rule,im_livechat.channel.rule,model_im_livechat_channel_rule,,1,0,0,0
access_livechat_channel_rule_user,im_livechat.channel.rule,model_im_livechat_channel_rule,im_livechat_group_user,1,1,1,0
access_livechat_channel_rule_manager,im_livechat.channel.rule,model_im_livechat_channel_rule,im_livechat_group_manager,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70">
    <defs>
        <path id="icon-a" d="M4,5.35309892e-14 C36.4160122,9.87060235e-15 58.0836068,-3.97961823e-14 65,5.07020818e-14 C69,6.733808e-14 70,1 70,5 C70,43.0488877 70,62.4235458 70,65 C70,69 69,70 65,70 C61,70 9,70 4,70 C1,70 7.10542736e-15,69 7.10542736e-15,65 C7.25721566e-15,62.4676575 3.83358709e-14,41.8005206 3.60818146e-14,5 C-1.13686838e-13,1 1,5.75716207e-14 4,5.35309892e-14 Z"/>
        <linearGradient id="icon-c" x1="100%" x2="0%" y1="0%" y2="100%">
            <stop offset="0%" stop-color="#B06161"/>
            <stop offset="45.785%" stop-color="#984E4E"/>
            <stop offset="100%" stop-color="#7C3838"/>
        </linearGradient>
        <path id="icon-d" d="M43.2723381,47 L4.024685,47 C2.0123425,47 0,45.9810543 0,42.9242174 L1.48938458e-14,18.2012896 L16.225629,0 L25.9851921,2.3438768 L45.134626,15.7216414 L52.320905,8.28006517 L57.3517613,13.3747934 L52.0323386,25.8943714 L59,31.065596 L43.4971227,46.7652986 L43.2723381,47 Z"/>
        <path id="icon-e" d="M29.8148059,45.4374895 C26.7685921,45.4374895 23.8982694,44.8827122 21.3772984,43.9035051 C18.8353437,45.9172903 15.7166184,47.1443958 12.4033677,47.4961458 C12.3794138,47.4986872 12.3553463,47.4999741 12.3312612,47.5000012 C12.0285762,47.5000012 11.7551388,47.2950872 11.6817361,47.0028098 C11.602338,46.6778841 11.8509027,46.4778919 12.0970368,46.2395091 C13.3136913,45.0550598 14.7886327,44.1239231 15.3654843,40.1448333 C13.0451961,37.8929934 11.6666667,35.0822747 11.6666667,32.0312864 C11.6666667,24.6264075 19.792577,18.6250012 29.8148059,18.6250012 C39.8370347,18.6250012 47.962945,24.6263255 47.962945,32.0312864 C47.962864,39.4413333 39.8370347,45.4374895 29.8148059,45.4374895 Z M57.9336461,54.2291067 C56.8039245,53.1522825 55.4343071,52.305802 54.8986939,48.68847 C60.4734134,43.3918762 59.125509,35.8148958 51.8464038,31.7026692 C51.8489153,31.8120989 51.851751,31.9214466 51.851751,32.0312864 C51.851751,42.0795403 41.3531335,49.7823567 28.8220864,49.3580911 C31.9105919,51.8978606 36.4369322,53.500013 41.4813857,53.500013 C44.3100649,53.500013 46.9753298,52.9956848 49.3161967,52.1054817 C51.676589,53.9361731 54.5725135,55.0517161 57.6491903,55.3714739 C57.9559262,55.4038762 58.2457293,55.2096262 58.3192131,54.9230091 C58.3930209,54.6276145 58.1621993,54.4458333 57.9336461,54.2291067 Z"/>
    </defs>
    <g fill="none" fill-rule="evenodd">
        <mask id="icon-b" fill="#fff">
            <use xlink:href="#icon-a"/>
        </mask>
        <g mask="url(#icon-b)">
            <rect width="70" height="70" fill="url(#icon-c)"/>
            <path fill="#FFF" fill-opacity=".383" d="M4,1.8 L65,1.8 C67.6666667,1.8 69.3333333,1.13333333 70,-0.2 C70,2.46666667 70,3.46666667 70,2.8 L1.10547097e-14,2.8 C-1.65952376e-14,3.46666667 -2.9161925e-14,2.46666667 -2.66453526e-14,-0.2 C0.666666667,1.13333333 2,1.8 4,1.8 Z" transform="matrix(1 0 0 -1 0 2.8)"/>
            <g transform="translate(0 22)">
                <use fill="#000" fill-opacity=".151" xlink:href="#icon-d"/>
            </g>
            <path fill="#000" fill-opacity=".383" d="M4,4 L65,4 C67.6666667,4 69.3333333,3 70,1 C70,3.66666667 70,5 70,5 L1.77635684e-15,5 C1.77635684e-15,5 1.77635684e-15,3.66666667 1.77635684e-15,1 C0.666666667,3 2,4 4,4 Z" transform="translate(0 65)"/>
            <use fill="#000" fill-rule="nonzero" opacity=".345" xlink:href="#icon-e"/>
            <path fill="#FFF" fill-rule="nonzero" d="M29.8148059,43.4374895 C26.7685921,43.4374895 23.8982694,42.8827122 21.3772984,41.9035051 C18.8353437,43.9172903 15.7166184,45.1443958 12.4033677,45.4961458 C12.3794138,45.4986872 12.3553463,45.4999741 12.3312612,45.5000012 C12.0285762,45.5000012 11.7551388,45.2950872 11.6817361,45.0028098 C11.602338,44.6778841 11.8509027,44.4778919 12.0970368,44.2395091 C13.3136913,43.0550598 14.7886327,42.1239231 15.3654843,38.1448333 C13.0451961,35.8929934 11.6666667,33.0822747 11.6666667,30.0312864 C11.6666667,22.6264075 19.792577,16.6250012 29.8148059,16.6250012 C39.8370347,16.6250012 47.962945,22.6263255 47.962945,30.0312864 C47.962864,37.4413333 39.8370347,43.4374895 29.8148059,43.4374895 Z M57.9336461,52.2291067 C56.8039245,51.1522825 55.4343071,50.305802 54.8986939,46.68847 C60.4734134,41.3918762 59.125509,33.8148958 51.8464038,29.7026692 C51.8489153,29.8120989 51.851751,29.9214466 51.851751,30.0312864 C51.851751,40.0795403 41.3531335,47.7823567 28.8220864,47.3580911 C31.9105919,49.8978606 36.4369322,51.500013 41.4813857,51.500013 C44.3100649,51.500013 46.9753298,50.9956848 49.3161967,50.1054817 C51.676589,51.9361731 54.5725135,53.0517161 57.6491903,53.3714739 C57.9559262,53.4038762 58.2457293,53.2096262 58.3192131,52.9230091 C58.3930209,52.6276145 58.1621993,52.4458333 57.9336461,52.2291067 Z"/>
        </g>
    </g>
</svg>

```

## File: static\src\js\ajax_external.js

```javascript
odoo.define('web.ajax_external', function (require) {
"use strict";

var ajax = require('web.ajax');

/**
  * This file should be used in the context of an external widget loading (e.g: live chat in a non-Odoo website)
  * It overrides the 'loadJS' method that is supposed to load additional scripts, based on a relative URL (e.g: '/web/webclient/locale/en_US')
  * As we're not in an Odoo website context, the calls will not work, and we avoid a 404 request.
  */
ajax.loadJS = function (url) {
    console.warn('Tried to load the following script on an external website: ' + url);
};
});

```

## File: static\src\js\im_livechat.js

```javascript
odoo.define('im_livechat.im_livechat', function (require) {
"use strict";

require('bus.BusService');
var concurrency = require('web.concurrency');
var config = require('web.config');
var core = require('web.core');
var session = require('web.session');
var time = require('web.time');
var utils = require('web.utils');
var Widget = require('web.Widget');

var WebsiteLivechat = require('im_livechat.model.WebsiteLivechat');
var WebsiteLivechatMessage = require('im_livechat.model.WebsiteLivechatMessage');
var WebsiteLivechatWindow = require('im_livechat.WebsiteLivechatWindow');

var _t = core._t;
var QWeb = core.qweb;

// Constants
var LIVECHAT_COOKIE_HISTORY = 'im_livechat_history';
var HISTORY_LIMIT = 15;

var RATING_TO_EMOJI = {
    "10":"😊",
    "5":"😐",
    "1":"😞"
};

// History tracking
var page = window.location.href.replace(/^.*\/\/[^/]+/, '');
var pageHistory = utils.get_cookie(LIVECHAT_COOKIE_HISTORY);
var urlHistory = [];
if (pageHistory) {
    urlHistory = JSON.parse(pageHistory) || [];
}
if (!_.contains(urlHistory, page)) {
    urlHistory.push(page);
    while (urlHistory.length > HISTORY_LIMIT) {
        urlHistory.shift();
    }
    utils.set_cookie(LIVECHAT_COOKIE_HISTORY, JSON.stringify(urlHistory), 60*60*24); // 1 day cookie
}

var LivechatButton = Widget.extend({
    custom_events: {
        'close_chat_window': '_onCloseChatWindow',
        'post_message_chat_window': '_onPostMessageChatWindow',
        'save_chat_window': '_onSaveChatWindow',
        'updated_typing_partners': '_onUpdatedTypingPartners',
        'updated_unread_counter': '_onUpdatedUnreadCounter',
    },
    events: {
        'click': '_openChat',
        'click .o_livechat_hide': '_hideChat',
    },
    template: 'im_livechat.OpenChatButton',
    init: function (parent, serverURL, options) {
        this._super(parent);
        this.options = _.defaults(options || {}, {
            input_placeholder: _t("Ask something ..."),
            default_username: _t("Visitor"),
            button_text: _t("Chat with one of our collaborators"),
            default_message: _t("How may I help you?"),
        });

        this._history = null;
        // livechat model
        this._livechat = null;
        // livechat window
        this._chatWindow = null;
        this._messages = [];
        this._serverURL = serverURL;
    },
    willStart: function () {
        var self = this;
        var cookie = utils.get_cookie('im_livechat_session');
        var ready;
        if (!cookie) {
            ready = session.rpc('/im_livechat/init', {channel_id: this.options.channel_id})
                .then(function (result) {
                    if (!result.available_for_me) {
                        return Promise.reject();
                    }
                    self._rule = result.rule;
                });
        } else {
            var channel = JSON.parse(cookie);
            ready = session.rpc('/mail/chat_history', {uuid: channel.uuid, limit: 100})
                .then(function (history) {
                    self._history = history;
                });
        }
        return ready.then(this._loadQWebTemplate.bind(this));
    },
    start: function () {
        if (this._history) {
            _.each(this._history.reverse(), this._addMessage.bind(this));
            this._openChat();
        } else if (!config.device.isMobile && this._rule.action === 'auto_popup') {
            var autoPopupCookie = utils.get_cookie('im_livechat_auto_popup');
            if (!autoPopupCookie || JSON.parse(autoPopupCookie)) {
                this._autoPopupTimeout =
                    setTimeout(this._openChat.bind(this), this._rule.auto_popup_timer*1000);
            }
        }
        this.call('bus_service', 'onNotification', this, this._onNotification);
        return this._super();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------


    /**
     * @private
     * @param {Object} data
     * @param {Object} [options={}]
     */
    _addMessage: function (data, options) {
        options = _.extend({}, this.options, options, {
            serverURL: this._serverURL,
        });
        var message = new WebsiteLivechatMessage(this, data, options);

        var hasAlreadyMessage = _.some(this._messages, function (msg) {
            return message.getID() === msg.getID();
        });
        if (hasAlreadyMessage) {
            return;
        }

        if (this._livechat) {
            this._livechat.addMessage(message);
        }

        if (options && options.prepend) {
            this._messages.unshift(message);
        } else {
            this._messages.push(message);
        }
    },
    /**
     * @private
     */
    _askFeedback: function () {
        this._chatWindow.$('.o_thread_composer input').prop('disabled', true);

        var feedback = new Feedback(this, this._livechat);
        this._chatWindow.replaceContentWith(feedback);

        feedback.on('send_message', this, this._sendMessage);
        feedback.on('feedback_sent', this, this._closeChat);
    },
    /**
     * @private
     */
    _hideChat: function (ev) {
        ev.stopPropagation();
        this.$el.hide();
    },
    /**
     * @private
     */
    _closeChat: function () {
        this._chatWindow.destroy();
        utils.set_cookie('im_livechat_session', "", -1); // remove cookie
    },
    /**
     * @private
     * @param {Array} notification
     */
    _handleNotification: function  (notification){
        if (this._livechat && (notification[0] === this._livechat.getUUID())) {
            if (notification[1]._type === 'history_command') { // history request
                var cookie = utils.get_cookie(LIVECHAT_COOKIE_HISTORY);
                var history = cookie ? JSON.parse(cookie) : [];
                session.rpc('/im_livechat/history', {
                    pid: this._livechat.getOperatorPID()[0],
                    channel_uuid: this._livechat.getUUID(),
                    page_history: history,
                });
            } else if (notification[1].info === 'typing_status') {
                var isWebsiteUser = notification[1].is_website_user;
                if (isWebsiteUser) {
                    return; // do not handle typing status notification of myself
                }
                var partnerID = notification[1].partner_id;
                if (notification[1].is_typing) {
                    this._livechat.registerTyping({ partnerID: partnerID });
                } else {
                    this._livechat.unregisterTyping({ partnerID: partnerID });
                }
            } else { // normal message
                // If message from notif is already in chatter messages, stop handling
                if (this._messages.some(message => message.getID() === notification[1].id)) {
                    this._livechat.unregisterTyping({ partnerID: notification[1].author_id[0] });
                    return;
                }
                this._addMessage(notification[1]);
                this._renderMessages();
                if (this._chatWindow.isFolded() || !this._chatWindow.isAtBottom()) {
                    this._livechat.incrementUnreadCounter();
                }
            }
        }
    },
    /**
     * @private
     */
    _loadQWebTemplate: function () {
        return session.rpc('/im_livechat/load_templates').then(function (templates) {
            _.each(templates, function (template) {
                QWeb.add_template(template);
            });
        });
    },
    /**
     * @private
     */
    _openChat: _.debounce(function () {
        if (this._openingChat) {
            return;
        }
        var self = this;
        var cookie = utils.get_cookie('im_livechat_session');
        var def;
        this._openingChat = true;
        clearTimeout(this._autoPopupTimeout);
        if (cookie) {
            def = Promise.resolve(JSON.parse(cookie));
        } else {
            this._messages = []; // re-initialize messages cache
            def = session.rpc('/im_livechat/get_session', {
                channel_id : this.options.channel_id,
                anonymous_name : this.options.default_username,
                previous_operator_id: this._get_previous_operator_id(),
            }, {shadow: true});
        }
        def.then(function (livechatData) {
            if (!livechatData || !livechatData.operator_pid) {
                alert(_t("None of our collaborators seem to be available, please try again later."));
            } else {
                self._livechat = new WebsiteLivechat({
                    parent: self,
                    data: livechatData
                });
                return self._openChatWindow().then(function () {
                    if (!self._history) {
                        self._sendWelcomeMessage();
                    }
                    self._renderMessages();
                    self.call('bus_service', 'addChannel', self._livechat.getUUID());
                    self.call('bus_service', 'startPolling');

                    utils.set_cookie('im_livechat_session', utils.unaccent(JSON.stringify(self._livechat.toData())), 60*60);
                    utils.set_cookie('im_livechat_auto_popup', JSON.stringify(false), 60*60);
                    if (livechatData.operator_pid[0]) {
                        // livechatData.operator_pid contains a tuple (id, name)
                        // we are only interested in the id
                        var operatorPidId = livechatData.operator_pid[0];
                        var oneWeek = 7*24*60*60;
                        utils.set_cookie('im_livechat_previous_operator_pid', operatorPidId, oneWeek);
                    }
                });
            }
        }).then(function () {
            self._openingChat = false;
        }).guardedCatch(function() {
            self._openingChat = false;
        });
    }, 200, true),
    /**
     * Will try to get a previous operator for this visitor.
     * If the visitor already had visitor A, it's better for his user experience
     * to get operator A again.
     *
     * The information is stored in the 'im_livechat_previous_operator_pid' cookie.
     *
     * @private
     * @return {integer} operator_id.partner_id.id if the cookie is set
     */
    _get_previous_operator_id: function () {
        var cookie = utils.get_cookie('im_livechat_previous_operator_pid');
        if (cookie) {
            return cookie;
        }

        return null;
    },
    /**
     * @private
     * @return {Promise}
     */
    _openChatWindow: function () {
        var self = this;
        var options = {
            displayStars: false,
            placeholder: this.options.input_placeholder || "",
        };
        this._chatWindow = new WebsiteLivechatWindow(this, this._livechat, options);
        return this._chatWindow.appendTo($('body')).then(function () {
            var cssProps = {bottom: 0};
            cssProps[_t.database.parameters.direction === 'rtl' ? 'left' : 'right'] = 0;
            self._chatWindow.$el.css(cssProps);
            self.$el.hide();
        });
    },
    /**
     * @private
     */
    _renderMessages: function () {
        var shouldScroll = !this._chatWindow.isFolded() && this._chatWindow.isAtBottom();
        this._livechat.setMessages(this._messages);
        this._chatWindow.render();
        if (shouldScroll) {
            this._chatWindow.scrollToBottom();
        }
    },
    /**
     * @private
     * @param {Object} message
     * @return {Promise}
     */
    _sendMessage: function (message) {
        var self = this;
        return session
            .rpc('/mail/chat_post', {uuid: this._livechat.getUUID(), message_content: message.content})
            .then(function () {
                self._chatWindow.scrollToBottom();
            });
    },
    /**
     * @private
     */
    _sendWelcomeMessage: function () {
        if (this.options.default_message) {
            this._addMessage({
                id: '_welcome',
                attachment_ids: [],
                author_id: this._livechat.getOperatorPID(),
                body: this.options.default_message,
                channel_ids: [this._livechat.getID()],
                date: time.datetime_to_str(new Date()),
                tracking_value_ids: [],
            }, {prepend: true});
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {OdooEvent} ev
     */
    _onCloseChatWindow: function (ev) {
        ev.stopPropagation();
        var isComposerDisabled = this._chatWindow.$('.o_thread_composer input').prop('disabled');
        var shouldAskFeedback = !isComposerDisabled && _.find(this._messages, function (message) {
            return message.getID() !== '_welcome';
        });
        if (shouldAskFeedback) {
            this._chatWindow.toggleFold(false);
            this._askFeedback();
        } else {
            this._closeChat();
        }
    },
    /**
     * @private
     * @param {Array[]} notifications
     */
    _onNotification: function (notifications) {
        var self = this;
        _.each(notifications, function (notification) {
            self._handleNotification(notification);
        });
    },
    /**
     * @private
     * @param {OdooEvent} ev
     * @param {Object} ev.data.messageData
     */
    _onPostMessageChatWindow: function (ev) {
        ev.stopPropagation();
        var self = this;
        var messageData = ev.data.messageData;
        this._sendMessage(messageData).guardedCatch(function (reason) {
            reason.event.preventDefault();
            return self._sendMessage(messageData); // try again just in case
        });
    },
    /**
     * @private
     * @param {OdooEvent} ev
     */
    _onSaveChatWindow: function (ev) {
        ev.stopPropagation();
        utils.set_cookie('im_livechat_session', utils.unaccent(JSON.stringify(this._livechat.toData())), 60*60);
    },
    /**
     * @private
     * @param {OdooEvent} ev
     */
    _onUpdatedTypingPartners: function (ev) {
        ev.stopPropagation();
        this._chatWindow.renderHeader();
    },
    /**
     * @private
     * @param {OdooEvent} ev
     */
    _onUpdatedUnreadCounter: function (ev) {
        ev.stopPropagation();
        this._chatWindow.renderHeader();
    },
});

/*
 * Rating for Livechat
 *
 * This widget displays the 3 rating smileys, and a textarea to add a reason
 * (only for red smiley), and sends the user feedback to the server.
 */
var Feedback = Widget.extend({
    template: 'im_livechat.FeedBack',

    events: {
        'click .o_livechat_rating_choices img': '_onClickSmiley',
        'click .o_livechat_no_feedback span': '_onClickNoFeedback',
        'click .o_rating_submit_button': '_onClickSend',
        'click .o_email_chat_button': '_onEmailChat',
        'click .o_livechat_email_error .alert-link': '_onTryAgain',
    },

    /**
     * @param {?} parent
     * @param {im_livechat.model.WebsiteLivechat} livechat
     */
    init: function (parent, livechat) {
        this._super(parent);
        this._livechat = livechat;
        this.server_origin = session.origin;
        this.rating = undefined;
        this.dp = new concurrency.DropPrevious();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Object} options
     */
    _sendFeedback: function (reason) {
        var self = this;
        var args = {
            uuid: this._livechat.getUUID(),
            rate: this.rating,
            reason: reason,
        };
        this.dp.add(session.rpc('/im_livechat/feedback', args)).then(function () {
            var emoji = RATING_TO_EMOJI[self.rating] || "??" ;
            var content = _.str.sprintf(_t("Rating: %s"), emoji);
            if (reason) {
                content += " \n" + reason;
            }
            self.trigger('send_message', { content: content });
        });
    },
    /**
    * @private
    */
    _showThanksMessage: function () {
        this.$('.o_livechat_rating_box').empty().append($('<div />', {
            text: _t('Thank you for your feedback'),
            class: 'text-muted'
        }));
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onClickNoFeedback: function () {
        this.trigger('feedback_sent'); // will close the chat
    },
    /**
     * @private
     */
    _onClickSend: function () {
        this.$('.o_livechat_rating_reason').hide();
        this._showThanksMessage();
        if (_.isNumber(this.rating)) {
            this._sendFeedback(this.$('textarea').val());
        }
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickSmiley: function (ev) {
        this.rating = parseInt($(ev.currentTarget).data('value'));
        this.$('.o_livechat_rating_choices img').removeClass('selected');
        this.$('.o_livechat_rating_choices img[data-value="'+this.rating+'"]').addClass('selected');

        // only display textearea if bad smiley selected
        if (this.rating !== 10) {
            this.$('.o_livechat_rating_reason').show();
        } else {
            this.$('.o_livechat_rating_reason').hide();
            this._showThanksMessage();
            this._sendFeedback();
        }
    },
    /**
    * @private
    */
    _onEmailChat: function () {
        var self = this;
        var $email = this.$('#o_email');

        if (utils.is_email($email.val())) {
            $email.removeAttr('title').removeClass('is-invalid').prop('disabled', true);
            this.$('.o_email_chat_button').prop('disabled', true);
            this._rpc({
                route: '/im_livechat/email_livechat_transcript',
                params: {
                    uuid: this._livechat.getUUID(),
                    email: $email.val(),
                }
            }).then(function () {
                self.$('.o_livechat_email').html($('<strong />', { text: _t('Conversation Sent') }));
            }).guardedCatch(function () {
                self.$('.o_livechat_email').hide();
                self.$('.o_livechat_email_error').show();
            });
        } else {
            $email.addClass('is-invalid').prop('title', _t('Invalid email address'));
        }
    },
    /**
    * @private
    */
    _onTryAgain: function () {
        this.$('#o_email').prop('disabled', false);
        this.$('.o_email_chat_button').prop('disabled', false);
        this.$('.o_livechat_email_error').hide();
        this.$('.o_livechat_email').show();
    },
});

return {
    LivechatButton: LivechatButton,
    Feedback: Feedback,
};

});

```

## File: static\src\js\im_livechat_backend.js

```javascript
odoo.define('im_livechat.Discuss', function (require) {
"use strict";

var Discuss = require('mail.Discuss');

Discuss.include({
    /**
     * Override to sort livechats by last message's date
     *
     * @override
     * @private
     * @param {mail.model.Channel[]} channels
     * @returns {mail.model.Channel[]}
     */
    _sortChannels: function (channels) {
        var partition = _.partition(channels, function (channel) {
            return channel.getType() === 'livechat';
        });
        partition[0].sort(function (c1, c2) {
            if (!c1.hasMessages()) {
                return -1;
            } else if (!c2.hasMessages()) {
                return 1;
            }
            return c2.getLastMessage().getDate().diff(c1.getLastMessage().getDate());
        });
        channels = partition[0].concat(partition[1]);
        return this._super(channels);
    },
});

});

```

## File: static\src\js\website_livechat_window.js

```javascript
odoo.define('im_livechat.WebsiteLivechatWindow', function (require) {
"use strict";

var AbstractThreadWindow = require('mail.AbstractThreadWindow');

/**
 * This is the widget that represent windows of livechat in the frontend.
 *
 * @see mail.AbstractThreadWindow for more information
 */
var LivechatWindow = AbstractThreadWindow.extend({
    events: _.extend(AbstractThreadWindow.prototype.events, {
        'input .o_composer_text_field': '_onInput',
    }),
    /**
     * @override
     * @param {im_livechat.im_livechat.LivechatButton} parent
     * @param {im_livechat.model.WebsiteLivechat} thread
     */
    init: function (parent, thread) {
        this._super.apply(this, arguments);

        this._thread = thread;
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    close: function () {
        this.trigger_up('close_chat_window');
    },
    /**
     * Replace the thread content with provided new content
     *
     * @param {$.Element} $element
     */
    replaceContentWith: function ($element) {
        $element.replace(this._threadWidget.$el);
    },
    /**
     * Warn the parent widget (LivechatButton)
     *
     * @override
     * @param {boolean} folded
     */
    toggleFold: function () {
        this._super.apply(this, arguments);
        this.trigger_up('save_chat_window');
        this.updateVisualFoldState();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     * @private
     * @param {Object} messageData
     */
    _postMessage: function (messageData) {
        this.trigger_up('post_message_chat_window', { messageData: messageData });
        this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called when the input in the composer changes
     *
     * @private
     */
    _onInput: function () {
        if (this.hasThread() && this._thread.hasTypingNotification()) {
            var isTyping = this.$input.val().length > 0;
            this._thread.setMyselfTyping({ typing: isTyping });
        }
    },
});

return LivechatWindow;

});

```

## File: static\src\js\models\website_livechat.js

```javascript
odoo.define('im_livechat.model.WebsiteLivechat', function (require) {
"use strict";

var AbstractThread = require('mail.model.AbstractThread');
var ThreadTypingMixin = require('mail.model.ThreadTypingMixin');

var session = require('web.session');

/**
 * Thread model that represents a livechat on the website-side. This livechat
 * is not linked to the mail service.
 */
var WebsiteLivechat = AbstractThread.extend(ThreadTypingMixin, {

    /**
     * @override
     * @private
     * @param {Object} params
     * @param {Object} params.data
     * @param {boolean} [params.data.folded] states whether the livechat is
     *   folded or not. It is considered only if this is defined and it is a
     *   boolean.
     * @param {integer} params.data.id the ID of this livechat.
     * @param {integer} [params.data.message_unread_counter] the unread counter
     *   of this livechat.
     * @param {Array} params.data.operator_pid
     * @param {string} params.data.name the name of this livechat.
     * @param {string} [params.data.state] if 'folded', the livechat is folded.
     *   This is ignored if `folded` is provided and is a boolean value.
     * @param {string} params.data.uuid the UUID of this livechat.
     * @param {im_livechat.im_livechat.LivechatButton} params.parent
     */
    init: function (params) {
        this._super.apply(this, arguments);
        ThreadTypingMixin.init.call(this, arguments);

        this._members = [];
        this._operatorPID = params.data.operator_pid;
        this._uuid = params.data.uuid;

        if (params.data.message_unread_counter !== undefined) {
            this._unreadCounter = params.data.message_unread_counter;
        }

        if (_.isBoolean(params.data.folded)) {
            this._folded = params.data.folded;
        } else {
            this._folded = params.data.state === 'folded';
        }

        // Necessary for thread typing mixin to display is typing notification
        // bar text (at least, for the operator in the members).
        this._members.push({
            id: this._operatorPID[0],
            name: this._operatorPID[1]
        });
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     * @returns {im_livechat.model.WebsiteLivechatMessage[]}
     */
    getMessages: function () {
        return this._messages;
    },
    /**
     * @returns {Array}
     */
    getOperatorPID: function () {
        return this._operatorPID;
    },
    /**
     * @returns {string}
     */
    getUUID: function () {
        return this._uuid;
    },
    /**
     * Increments the unread counter of this livechat by 1 unit.
     *
     * Note: this public method makes sense because the management of messages
     * for website livechat is external. This method should be dropped when
     * this class handles messages by itself.
     */
    incrementUnreadCounter: function () {
        this._incrementUnreadCounter();
    },
    /**
     * AKU: hack for the moment
     *
     * @param {im_livechat.model.WebsiteLivechatMessage[]} messages
     */
    setMessages: function (messages) {
        this._messages = messages;
    },
    /**
     * @returns {Object}
     */
    toData: function () {
        return {
            folded: this.isFolded(),
            id: this.getID(),
            message_unread_counter: this.getUnreadCounter(),
            operator_pid: this.getOperatorPID(),
            name: this.getName(),
            uuid: this.getUUID(),
        };
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override {mail.model.ThreadTypingMixin}
     * @private
     * @param {Object} params
     * @param {boolean} params.isWebsiteUser
     * @returns {boolean}
     */
    _isTypingMyselfInfo: function (params) {
        return params.isWebsiteUser;
    },
    /**
     * @override {mail.model.ThreadTypingMixin}
     * @private
     * @param {Object} params
     * @param {boolean} params.typing
     * @returns {Promise}
     */
    _notifyMyselfTyping: function (params) {
        return session.rpc('/im_livechat/notify_typing', {
            uuid: this.getUUID(),
            is_typing: params.typing,
        }, { shadow: true });
    },
    /**
     * Warn views that the list of users that are currently typing on this
     * livechat has been updated.
     *
     * @override {mail.model.ThreadTypingMixin}
     * @private
     */
    _warnUpdatedTypingPartners: function () {
        this.trigger_up('updated_typing_partners');
    },
    /**
     * Warn that the unread counter has been updated on this livechat
     *
     * @override
     * @private
     */
    _warnUpdatedUnreadCounter: function () {
        this.trigger_up('updated_unread_counter');
    },

    //--------------------------------------------------------------------------
    // Handler
    //--------------------------------------------------------------------------

    /**
     * Override so that it only unregister typing operators.
     *
     * Note that in the frontend, there is no way to identify a message that is
     * from the current user, because there is no partner ID in the session and
     * a message with an author sets the partner ID of the author.
     *
     * @override {mail.model.ThreadTypingMixin}
     * @private
     * @param {mail.model.AbstractMessage} message
     */
    _onTypingMessageAdded: function (message) {
        var operatorID = this.getOperatorPID()[0];
        if (message.hasAuthor() && message.getAuthorID() === operatorID) {
            this.unregisterTyping({ partnerID: operatorID });
        }
    },
});

return WebsiteLivechat;

});

```

## File: static\src\js\models\website_livechat_message.js

```javascript
odoo.define('im_livechat.model.WebsiteLivechatMessage', function (require) {
"use strict";

var AbstractMessage = require('mail.model.AbstractMessage');

/**
 * This is a message that is handled by im_livechat, without making use of the
 * mail.Manager. The purpose of this is to make im_livechat compatible with
 * mail.widget.Thread.
 *
 * @see mail.model.AbstractMessage for more information.
 */
var WebsiteLivechatMessage =  AbstractMessage.extend({

    /**
     * @param {im_livechat.im_livechat.LivechatButton} parent
     * @param {Object} data
     * @param {Object} options
     * @param {string} options.default_username
     * @param {string} options.serverURL
     */
    init: function (parent, data, options) {
        this._super.apply(this, arguments);

        this._defaultUsername = options.default_username;
        this._serverURL = options.serverURL;
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Get the relative url of the avatar to display next to the message
     *
     * @override
     * @return {string}
     */
    getAvatarSource: function () {
        var source = this._serverURL;
        if (this.hasAuthor()) {
            source += '/web/partner_image/' + this.getAuthorID();
        } else {
            source += '/mail/static/src/img/smiley/avatar.jpg';
        }
        return source;
    },
    /**
     * Get the text to display for the author of the message
     *
     * Rule of precedence for the displayed author::
     *
     *      author name > default usernane
     *
     * @override
     * @return {string}
     */
    getDisplayedAuthor: function () {
        return this._super.apply(this, arguments) || this._defaultUsername;
    },

});

return WebsiteLivechatMessage;

});

```

## File: static\src\xml\im_livechat.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates xml:space="preserve">

    <t t-name="im_livechat.FeedBack">
        <div class="o_livechat_rating text-center">
            <div class="o_livechat_rating_box">
                <div class="o_livechat_rating_feedback_text">
                    Did we correctly answer your question ?
                </div>
                <div class="o_livechat_rating_choices">
                    <img t-att-src="widget.server_origin + '/rating/static/src/img/rating_10.png'" alt="Good" data-value="10"/>
                    <img t-att-src="widget.server_origin + '/rating/static/src/img/rating_5.png'" alt="OK" data-value="5"/>
                    <img t-att-src="widget.server_origin + '/rating/static/src/img/rating_1.png'" alt="Bad" data-value="1" />
                </div>
            </div>
            <div class="o_livechat_rating_reason">
                <textarea id="reason" placeholder="Explain your note"></textarea>
                <div class="o_livechat_rating_reason_button">
                    <input type="button" class="btn btn-primary btn-sm o_rating_submit_button" value="Send" />
                </div>
            </div>
            <div class="o_livechat_email">
                <span class="text-muted">Receive a copy of this conversation</span>
                <div class="input-group">
                    <input id="o_email" type="text" class="form-control" placeholder="mail@example.com"/>
                    <button type="button" class="o_email_chat_button btn btn-primary rounded-0">
                        <i class="fa fa-paper-plane"/>
                    </button>
                </div>
            </div>
            <div class="alert alert-danger px-0 o_livechat_email_error" style="display: none;" role="alert">
                Oops! Something went wrong.<br />Please check your internet connection.<br />
                <a href="#" class="alert-link">Try again</a>
            </div>
            <div class="o_livechat_no_feedback text-muted">
                <span>Close conversation</span>
            </div>
        </div>
    </t>

    <t t-name="im_livechat.OpenChatButton">
        <div class="openerp o_livechat_button d-print-none row">
            <div class="o_livechat_open">
                <t t-esc="widget.options.button_text" />
            </div>
            <button type="button" class="close o_livechat_hide mx-2 d-md-none" data-dismiss="alert" aria-label="Close"><span title="Close" class="fa fa-times"></span></button>
        </div>
    </t>
</templates>

```

## File: static\src\xml\im_livechat_backend.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<template>

    <t t-extend="mail.discuss.SidebarChannels">
        <t t-jquery="t[t-call='mail.discuss.SidebarItems']:last" t-operation="after">
            <t t-set="type" t-value="'livechat'"/>
            <t t-set="disableAddThread" t-value="true"/>
            <t t-call="mail.discuss.SidebarTitle">
                <t t-set="title">Livechat</t>
                <t t-set="icon" t-value="fa-comments"/>
            </t>
            <t t-call="mail.discuss.SidebarItems"/>
        </t>
    </t>

    <!-- Mobile templates -->
    <t t-extend="mail.discuss_mobile">
        <t t-jquery=".o_mail_mobile_tabs" t-operation="append">
            <div class="o_mail_mobile_tab" data-type="livechat">
                <span class="fa fa-comments"/>
                <span class="o_tab_title">Livechat</span>
            </div>
        </t>
    </t>

</template>

```

## File: views\im_livechat_channel_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <!--
            Integrate Livechat Conversation in the Discuss
        -->
        <template id="assets_backend" name="im_livechat assets backend" inherit_id="web.assets_backend">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/im_livechat/static/src/js/im_livechat_backend.js"></script>
                <link rel="stylesheet" type="text/scss" href="/im_livechat/static/src/scss/im_livechat_history.scss"/>
                <link rel="stylesheet" type="text/scss" href="/im_livechat/static/src/scss/im_livechat_form.scss"/>
            </xpath>
        </template>

        <template id="qunit_suite" name="livechat_tests" inherit_id="web.qunit_suite">
            <xpath expr="//t[@t-set='head']" position="inside">
                <script type="text/javascript" src="/im_livechat/static/src/js/website_livechat_window.js"></script>
                <script type="text/javascript" src="/im_livechat/static/src/js/models/website_livechat.js"></script>
                <script type="text/javascript" src="/im_livechat/static/src/js/models/website_livechat_message.js"></script>

                <script type="text/javascript" src="/im_livechat/static/tests/discuss_livechat_tests.js"></script>
                <script type="text/javascript" src="/im_livechat/static/tests/website_livechat_window_tests.js"></script>
            </xpath>
        </template>

        <!--
            Template rendering the external HTML support page
        -->
        <template id="support_page" name="Livechat : Support Page">
            &lt;!DOCTYPE html&gt;
            <html style="height: 100%">
                <head>
                    <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
                    <meta http-equiv="content-type" content="text/html; charset=utf-8" />
                    <title><t t-esc="channel_name"/> Livechat Support Page</title>

                    <!-- Call the external Bundle to render the css, js, and js loader tags -->
                    <t t-raw="channel.script_external"/>

                    <style type="text/css">
                        body {
                            height: 100%;
                            font-size: 16px;
                            font-weight: 400;
                            font-family: "Lato", "Lucida Grande", "Helvetica neue", "Helvetica", "Verdana", "Arial", sans-serif;
                            overflow: hidden;
                            overflow-y: auto;
                            display: block;
                            margin: 0;
                            padding: 0;
                            border: none;
                            width: 100%;
                            height: 100%;
                            background: #C9C8E0;
                            background-image: -webkit-linear-gradient(top, #7c7bad, #ddddee);
                            background-image: -moz-linear-gradient(top, #7c7bad, #ddddee);
                            background-image: -ms-linear-gradient(top, #7c7bad, #ddddee);
                            background-image: -o-linear-gradient(top, #7c7bad, #ddddee);
                            background-image: linear-gradient(to bottom, #7c7bad, #ddddee);
                            filter: progid:DXImageTransform.Microsoft.gradient( startColorstr='#7c7bad', endColorstr='#ddddee',GradientType=0 );
                            -webkit-background-size: cover;
                            -moz-background-size: cover;
                            -o-background-size: cover;
                            background-size: cover;
                            background-repeat: no-repeat;
                            background-attachment: fixed;
                        }
                        .main {
                            position: absolute;
                            opacity: 0;
                            top: 50%;
                            width: 100%;
                            margin-top: -150px;
                            color: white;
                            text-shadow: 0 1px 0 rgba(34, 52, 72, 0.2);
                            text-align: center;
                        }
                        .main h1 {
                            font-size: 54px;
                        }
                        .main div {
                            font-style: italic;
                        }
                    </style>
                </head>

                <body>
                     <div class="main" style="opacity: 1;">
                        <h1 class="channel_name"><t t-esc="channel.name"/></h1>
                        <div>Website Live Chat Powered by <strong>Odoo</strong>.</div>
                    </div>
                </body>
            </html>
        </template>

        <!--
            Template rendering all the scripts required to execute the Livechat from an external page (which not contain Odoo)
        -->
        <template id="external_loader" name="Livechat : external_script field of livechat channel">
            <!-- css style -->
            <link t-att-href="'%s/im_livechat/external_lib.css' % (url)" rel="stylesheet"/>
            <!-- js of all the required lib (internal and external) -->
            <script t-att-src="'%s/im_livechat/external_lib.js' % (url)" type="text/javascript" />
            <!-- the loader -->
            <script t-att-src="'%s/im_livechat/loader/%i' % (url, channel_id)" type="text/javascript"/>
        </template>


        <!--
            Bundle of External Librairies of the Livechat (Odoo + required modules)
        -->
        <template id="external_lib" name="Livechat : External Librairies of the Livechat, required to make it work">
            <!-- Momentjs -->
            <script type="text/javascript" src="/web/static/lib/moment/moment.js"></script>
            <!-- Odoo minimal lib -->
            <script type="text/javascript" src="/web/static/lib/underscore/underscore.js"></script>
            <script type="text/javascript" src="/web/static/lib/underscore.string/lib/underscore.string.js"></script>
            <!-- jQuery -->
            <script type="text/javascript" src="/web/static/lib/jquery/jquery.js"></script>
            <script type="text/javascript" src="/web/static/lib/jquery.ui/jquery-ui.js"></script>
            <script type="text/javascript" src="/web/static/lib/jquery/jquery.browser.js"></script>
            <script type="text/javascript" src="/web/static/lib/jquery.ba-bbq/jquery.ba-bbq.js"></script>
            <!-- Qweb2 lib -->
            <script type="text/javascript" src="/web/static/lib/qweb/qweb2.js"></script>
            <!-- Odoo JS Framework -->
            <script type="text/javascript" src="/web/static/src/js/promise_extension.js"></script>
            <script type="text/javascript" src="/web/static/src/js/boot.js"></script>
            <script type="text/javascript" src="/web/static/src/js/libs/download.js"></script>
            <script type="text/javascript" src="/web/static/src/js/libs/content-disposition.js"></script>
            <script type="text/javascript" src="/web/static/src/js/services/config.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/abstract_service.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/class.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/collections.js"/>
            <script type="text/javascript" src="/web/static/src/js/core/translation.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/ajax.js"></script>
            <script type="text/javascript" src="/im_livechat/static/src/js/ajax_external.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/time.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/mixins.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/service_mixins.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/rpc.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/widget.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/registry.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/session.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/concurrency.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/utils.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/dom.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/qweb.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/bus.js"></script>
            <script type="text/javascript" src="/web/static/src/js/services/core.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/local_storage.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/ram_storage.js"></script>
            <script type="text/javascript" src="/web/static/src/js/core/abstract_storage_service.js"></script>
            <script type="text/javascript" src="/web/static/src/js/public/lazyloader.js"/>
            <script type="text/javascript" src="/web/static/src/js/public/public_root.js"/>
            <script type="text/javascript" src="/web/static/src/js/public/public_root_instance.js"/>
            <script type="text/javascript" src="/web/static/src/js/public/public_widget.js"/>
            <script type="text/javascript" src="/web/static/src/js/services/ajax_service.js"></script>
            <script type="text/javascript" src="/web/static/src/js/services/local_storage_service.js"></script>
            <!-- Bus, Mail, Livechat -->
                <!-- thread windows -->
                <script type="text/javascript" src="/mail/static/src/js/thread_windows/abstract_thread_window.js"></script>
                <script type="text/javascript" src="/im_livechat/static/src/js/website_livechat_window.js"></script>
                <script type="text/javascript" src="/mail/static/src/js/thread_widget.js"></script>

            <script type="text/javascript" src="/bus/static/src/js/longpolling_bus.js"></script>
            <script type="text/javascript" src="/bus/static/src/js/crosstab_bus.js"></script>
            <script type="text/javascript" src="/bus/static/src/js/services/bus_service.js"></script>

            <script type="text/javascript" src="/mail/static/src/js/utils.js"></script>
            <script type="text/javascript" src="/mail/static/src/js/document_viewer.js"></script>
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
            <t t-call="web._assets_helpers"/>
            <link rel="stylesheet" type="text/scss" href="/im_livechat/static/src/scss/im_livechat_bootstrap.scss"/>
            <link rel="stylesheet" type="text/scss" href="/mail/static/src/scss/abstract_thread_window.scss"></link>
            <link rel="stylesheet" type="text/scss" href="/mail/static/src/scss/thread.scss"></link>
            <link rel="stylesheet" type="text/scss" href="/im_livechat/static/src/scss/im_livechat.scss"/>
        </template>

        <!-- the js code to initialize the LiveSupport object -->
        <template id="loader" name="Livechat : Javascript appending the livechat button">
            <t t-translation="off">
                window.addEventListener('load', function () {
                    <t t-if="web_session_required">
                    odoo.define('web.session', function (require) {
                        var Session = require('web.Session');
                        var modules = odoo._modules;
                        return new Session(undefined, "<t t-esc="info['server_url']"/>", {modules:modules, use_cors: true});
                    });
                    </t>

                    odoo.define('im_livechat.livesupport', function (require) {
            <t t-if="info['available']" t-translation="off">
                        var rootWidget = require('root.widget');
                        var im_livechat = require('im_livechat.im_livechat');
                        var button = new im_livechat.LivechatButton(
                            rootWidget,
                            "<t t-esc="info['server_url']"/>",
                            <t t-raw="json.dumps(info.get('options', {}))"/>
                        );
                        button.appendTo(document.body);
                        window.livechat_button = button;
            </t>
                    });
                });
            </t>
        </template>


    </data>
</odoo>

```

## File: views\im_livechat_channel_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>

        <record id="im_livechat_channel_action" model="ir.actions.act_window">
            <field name="name">Website Live Chat Channels</field>
            <field name="res_model">im_livechat.channel</field>
            <field name="view_mode">kanban,form</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Define a new website live chat channel
              </p><p>
                You can create channels for each website on which you want
                to integrate the website live chat widget, allowing your website
                visitors to talk in real time with your operators.
              </p>
            </field>
        </record>

        <record id="im_livechat_channel_view_kanban" model="ir.ui.view">
            <field name="name">im_livechat.channel.kanban</field>
            <field name="model">im_livechat.channel</field>
            <field name="arch" type="xml">
                <kanban>
                    <field name="id"/>
                    <field name="name"/>
                    <field name="web_page" widget="url"/>
                    <field name="are_you_inside"/>
                    <field name="user_ids"/>
                    <field name="nbr_channel"/>
                    <field name="rating_percentage_satisfaction"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div class="oe_kanban_global_click">
                                <div class="o_kanban_image">
                                    <img t-att-src="kanban_image('im_livechat.channel', 'image_128', record.id.raw_value)" class="img-fluid" alt="Channel"/>
                                </div>
                                <div class="oe_kanban_details">
                                    <div class="float-right">
                                        <button t-if="record.are_you_inside.raw_value" name="action_quit" type="object" class="btn btn-primary">Leave</button>
                                        <button t-if="!record.are_you_inside.raw_value" name="action_join" type="object" class="btn btn-primary">Join</button>
                                    </div>
                                    <strong class="o_kanban_record_title" style="word-wrap: break-word;"><field name="name"/></strong>
                                    <div>
                                        <div>
                                            <i class="fa fa-user" role="img" aria-label="User" title="User"></i>
                                            <t t-esc="(record.user_ids.raw_value || []).length"/> Operators
                                            <br/>
                                            <i class="fa fa-comments" role="img" aria-label="Comments" title="Comments"></i>
                                            <t t-esc="record.nbr_channel.raw_value"/> Sessions
                                             <div class="float-right">
                                                <t t-if="record.rating_percentage_satisfaction.raw_value &gt; 0">
                                                    <a type="action" name="%(rating_rating_action_view_livechat_rating)d" tabindex="10">
                                                        <i class="fa fa-smile-o text-success" t-if="record.rating_percentage_satisfaction.raw_value &gt;= 70" title="Rating: Great" role="img" aria-label="Happy face"/>
                                                        <i class="fa fa-meh-o text-warning" t-if="record.rating_percentage_satisfaction.raw_value &gt; 30 and record.rating_percentage_satisfaction.raw_value &lt; 70" title="Rating: Okay" role="img" aria-label="Neutral face"/>
                                                        <i class="fa fa-frown-o text-danger" t-if="record.rating_percentage_satisfaction.raw_value &lt;= 30" title="Rating: Bad" role="img" aria-label="Sad face"/>
                                                       <t t-esc="record.rating_percentage_satisfaction.raw_value"/>%
                                                   </a>
                                                </t>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="im_livechat_channel_view_form" model="ir.ui.view">
            <field name="name">im_livechat.channel.form</field>
            <field name="model">im_livechat.channel</field>
            <field name="arch" type="xml">
                <form>
                    <header>
                        <button type="object" name="action_join" class="oe_highlight" string="Join Channel" attrs='{"invisible": [["are_you_inside", "=", True]]}'/>
                        <button type="object" name="action_quit" string="Leave Channel" attrs='{"invisible": [["are_you_inside", "=", False]]}'/>
                        <field name="are_you_inside" invisible="1"/>
                    </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button class="oe_stat_button" type="action" attrs="{'invisible':[('nbr_channel','=', 0)]}" name="%(mail_channel_action_from_livechat_channel)d" icon="fa-comments">
                            <field string="Sessions" name="nbr_channel" widget="statinfo"/>
                        </button>
                        <button name="action_view_rating" attrs="{'invisible':[('rating_percentage_satisfaction','=', -1)]}" class="oe_stat_button" type="object" icon="fa-smile-o">
                            <field string="% Happy" name="rating_percentage_satisfaction" widget="statinfo"/>
                        </button>
                    </div>
                    <field name="image_128" widget="image" class="oe_avatar"/>
                    <div class="oe_title">
                        <label for="name" class="oe_edit_only"/>
                        <h1>
                            <field name="name" placeholder="e.g. YourWebsite.com"/>
                        </h1>
                    </div>
                    <notebook>
                        <page string="Operators">
                            <group>
                                <field name="user_ids" nolabel="1" colspan="2">
                                    <kanban>
                                        <field name="id"/>
                                        <field name="name"/>
                                        <templates>
                                            <t t-name="kanban-box">
                                                <div class="oe_kanban_global_click">
                                                    <div class="o_kanban_image">
                                                        <img t-att-src="kanban_image('res.users', 'image_1024', record.id.raw_value)" alt="User"/>
                                                    </div>
                                                    <div class="o_kanban_details">
                                                        <h4 class="o_kanban_record_title"><field name="name"/></h4>
                                                    </div>
                                                </div>
                                            </t>
                                        </templates>
                                    </kanban>
                                </field>
                                <p class="text-muted" style="padding-left: 8px">
                                    Operators that do not show any activity In Odoo for more than 30 minutes will be considered as disconnected.
                                </p>
                            </group>
                        </page>
                        <page string="Options">
                            <group>
                                <field name="button_text"/>
                                <field name="default_message" placeholder="e.g. Hello, how may I help you?"/>
                                <field name="input_placeholder"/>
                            </group>
                        </page>
                        <page string="Channel Rules">
                            <span class="text-muted">Define rules for your live support channel. You can apply an action for the given URL, and per country.<br />To identify the country, GeoIP must be installed on your server, otherwise, the countries of the rule will not be taken into account.</span>
                            <group>
                                <field name="rule_ids" nolabel="1"/>
                            </group>
                        </page>
                        <page string="Widget">
                            <group attrs='{"invisible": [["web_page", "!=", False]]}'>
                                <div class="alert alert-warning mt4 mb16" role="alert">
                                Save your Channel to get your configuration widget.
                                </div>
                            </group>
                            <group>
                                <div attrs='{"invisible": [["web_page", "=", False]]}'>
                                    <separator string="How to use the Website Live Chat widget?"/>
                                    <p>
                                        Copy and paste this code into your website, within the &lt;head&gt; tag:
                                    </p>
                                    <field name="script_external" readonly="1" widget="CopyClipboardText"/>
                                    <p>
                                        or copy this url and send it by email to your customers or suppliers:
                                    </p>
                                    <field name="web_page" readonly="1" widget="CopyClipboardChar"/>
                                    <p>For websites built with the Odoo CMS, go to Website > Configuration > Settings and select the Website Live Chat Channel you want to add on your website.</p>
                                </div>
                            </group>
                        </page>
                    </notebook>
                </sheet>
                </form>
            </field>
        </record>

        <record id="im_livechat_channel_view_search" model="ir.ui.view">
            <field name="name">im.livechat.channel.view.search</field>
            <field name="model">im_livechat.channel</field>
            <field name="arch" type="xml">
                <search string="LiveChat Channel Search">
                    <field name="name" string="Channel"/>
                </search>
            </field>
        </record>

        <!-- im_livechat.channel.rule -->
        <record id="im_livechat_channel_rule_view_tree" model="ir.ui.view">
            <field name="name">im.livechat.channel.rule.tree</field>
            <field name="model">im_livechat.channel.rule</field>
            <field name="arch" type="xml">
                <tree string="Rules">
                    <field name="sequence" widget="handle"/>
                    <field name="regex_url"/>
                    <field name="action"/>
                    <field name="country_ids"/>
                </tree>
            </field>
        </record>

        <record id="im_livechat_channel_rule_view_kanban" model="ir.ui.view">
            <field name="name">im_livechat.channel.rule.kanban</field>
            <field name="model">im_livechat.channel.rule</field>
            <field name="arch" type="xml">
                <kanban>
                    <field name="regex_url"/>
                    <field name="action"/>
                    <field name="country_ids"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div class="oe_kanban_global_click">
                                <div><field name="action"/></div>
                                <field name="regex_url" />
                                <field name="country_ids" widget="many2many_tags" />
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="im_livechat_channel_rule_view_form" model="ir.ui.view">
            <field name="name">im_livechat.channel.rule.form</field>
            <field name="model">im_livechat.channel.rule</field>
            <field name="type">form</field>
            <field name="arch" type="xml">
                <form string="Channel Rule" class="o_livechat_rules_form">
                    <sheet>
                        <group>
                            <field name="action" widget="radio"/>
                            <field name="regex_url" placeholder="e.g. /page/contactus"/>
                            <label for="auto_popup_timer" class="oe_inline" attrs="{'invisible': [('action', '!=', 'auto_popup')]}"/>
                            <div class="oe_inline" attrs="{'invisible': [('action', '!=', 'auto_popup')]}">
                                <field name="auto_popup_timer" class="oe_inline"/> seconds
                            </div>
                            <field name="country_ids" widget="many2many_tags"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <!-- Canned responses -->
        <record id="im_livechat_canned_response_view_tree" model="ir.ui.view">
            <field name="name">im_livechat.canned_response.tree</field>
            <field name="model">mail.shortcode</field>
            <field name="arch" type="xml">
                <tree editable="bottom">
                    <field name="source"/>
                    <field name="substitution"/>
                </tree>
            </field>
        </record>

        <record id="im_livechat_canned_response_action" model="ir.actions.act_window">
            <field name="name">Canned Responses</field>
            <field name="res_model">mail.shortcode</field>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="im_livechat_canned_response_view_tree"/>
            <field name="domain">[]</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new canned response
              </p><p>
                Canned responses allow you to insert prewritten responses in
                your messages by typing <i>:shortcut</i>. The shortcut is
                replaced directly in your message, so that you can still edit
                it before sending.
              </p>
            </field>
        </record>

        <!-- Menu items -->
        <menuitem
            id="menu_livechat_root"
            name="Live Chat"
            web_icon="im_livechat,static/description/icon.png"
            groups="im_livechat_group_user"
            sequence="205"/>

        <menuitem
            id="support_channels"
            name="Channels"
            parent="menu_livechat_root"
            action="im_livechat_channel_action"
            groups="im_livechat_group_user"
            sequence="5"/>

        <menuitem
            id="menu_reporting_livechat"
            name="Report"
            parent="menu_livechat_root"
            sequence="50"
            groups="im_livechat_group_manager"/>


        <menuitem
            id="session_history"
            name="Sessions History"
            parent="menu_reporting_livechat"
            action="mail_channel_action"
            groups="im_livechat_group_user"
            sequence="5"/>

        <menuitem id="rating_rating_menu_livechat"
            name="Customer Ratings"
            action="rating_rating_action_livechat_report"
            parent="menu_reporting_livechat"
            sequence="40"/>

        <menuitem
            id="livechat_config"
            name="Configuration"
            parent="menu_livechat_root"
            sequence="55"/>

        <menuitem
            id="canned_responses"
            name="Canned Responses"
            parent="livechat_config"
            action="im_livechat_canned_response_action"
            groups="im_livechat_group_user"
            sequence="15"/>

    </data>
</odoo>

```

## File: views\mail_channel_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>

        <record id="mail_channel_view_search" model="ir.ui.view">
            <field name="name">mail.channel.search</field>
            <field name="model">mail.channel</field>
            <field name="arch" type="xml">
                <search string="Search history">
                    <field name="name"/>
                    <filter string="With Messages" name="session_not_empty" domain="[('channel_message_ids', '!=', False)]"/>
                    <filter string="Empty Sessions" name="session_empty" domain="[('channel_message_ids', '=', False)]"/>
                    <group expand="0" string="Group By...">
                        <filter name="group_by_channel" string="Channel" domain="[]" context="{'group_by':'livechat_channel_id'}"/>
                        <separator orientation="vertical"/>
                        <filter name="group_by_month" string="Creation Date" domain="[]" context="{'group_by':'create_date:month'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="mail_channel_view_tree" model="ir.ui.view">
            <field name="name">mail.channel.tree</field>
            <field name="model">mail.channel</field>
            <field name="arch" type="xml">
                <tree string="History" create="false" default_order="create_date desc">
                    <field name="create_date" string="Session Date"/>
                    <field name="name" string="Attendees"/>
                    <field name="channel_message_ids" string="# Messages"/>
                    <field name="rating_last_image" string="Rating" widget="image" options='{"size": [20, 20]}'/>
                </tree>
            </field>
        </record>

        <record id="mail_channel_view_form" model="ir.ui.view">
            <field name="name">mail.channel.form</field>
            <field name="model">mail.channel</field>
            <field name="arch" type="xml">
                <form string="Session Form" create="false" edit="false">
                    <sheet>
                        <div style="width:50%" class="float-right">
                            <field name="rating_last_image" widget="image" class="float-right" readonly="1" nolabel="1"/>
                            <field name="rating_last_feedback" nolabel="1"/>
                        </div>
                        <div style="width:50%" class="float-left">
                            <group>
                                <field name="name" string="Attendees"/>
                                <field name="create_date" readonly="1" string="Session Date"/>
                            </group>
                        </div>

                        <group string="History" class="o_history_container">
                            <div class="o_history_kanban_container">
                                <div class="o_history_kanban_sub_container">
                                    <field name="message_ids" mode="kanban">
                                        <kanban default_order="create_date DESC">
                                            <field name="author_id"/>
                                            <field name="body"/>
                                            <field name="create_date" groups="base.group_no_one"/>
                                            <field name="id"/>
                                            <field name="author_avatar"/>
                                            <templates>
                                                <t t-name="kanban-box">
                                                    <div class="oe_module_vignette">
                                                        <div class="o_kanban_image">
                                                            <div>
                                                                 <t t-if="record.author_avatar.raw_value">
                                                                    <img t-att-src="kanban_image('mail.message', 'author_avatar', record.id.raw_value)" alt="Avatar" class="o_image_64_cover rounded-circle"/>
                                                                 </t>
                                                                 <t t-else=""><img alt="Anonymous" src="/mail/static/src/img/smiley/avatar.jpg" class="o_image_64_cover rounded-circle"/></t>
                                                            </div>
                                                        </div>
                                                        <div class="oe_module_desc">
                                                            <div class="float-right"><p><field name="date"/></p></div>
                                                            <div>
                                                                <p><strong>
                                                                    <t t-if="record.author_id.raw_value"><field name="author_id"/></t>
                                                                    <t t-else="">Anonymous</t>
                                                                </strong></p>
                                                                <p>
                                                                    <t t-if="record.body.raw_value"><field name="body" widget="html"/><br/></t>
                                                                </p>
                                                            </div>
                                                        </div>

                                                    </div>
                                                </t>
                                            </templates>
                                         </kanban>
                                    </field>
                                </div>
                            </div>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>


        <record id="mail_channel_action" model="ir.actions.act_window">
            <field name="name">History</field>
            <field name="res_model">mail.channel</field>
            <field name="view_mode">tree,form</field>
            <field name="search_view_id" ref="im_livechat.mail_channel_view_search"/>
            <field name="domain">[('livechat_channel_id', '!=', None)]</field>
            <field name="context">{'search_default_session_not_empty': 1}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_empty_folder">
                    Your chatter history is empty
                </p><p>
                    Create a channel and start chatting to fill up your history.
                </p>
            </field>
        </record>
        <record id="mail_channel_action_tree" model="ir.actions.act_window.view">
            <field name="sequence">1</field>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="im_livechat.mail_channel_view_tree"/>
            <field name="act_window_id" ref="im_livechat.mail_channel_action"/>
        </record>

        <record id="mail_channel_action_form" model="ir.actions.act_window.view">
            <field name="sequence">2</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="im_livechat.mail_channel_view_form"/>
            <field name="act_window_id" ref="im_livechat.mail_channel_action"/>
        </record>


        <record id="mail_channel_action_from_livechat_channel" model="ir.actions.act_window">
            <field name="name">Sessions</field>
            <field name="res_model">mail.channel</field>
            <field name="view_mode">tree,form</field>
            <field name="domain">[('livechat_channel_id', 'in', [active_id])]</field>
            <field name="context">{
                'search_default_livechat_channel_id': [active_id],
                'default_livechat_channel_id': active_id,
                'search_default_session_not_empty': 1,
            }</field>
            <field name="search_view_id" ref="mail_channel_view_search"/>
        </record>
        <record id="mail_channel_action_livechat_tree" model="ir.actions.act_window.view">
            <field name="sequence">1</field>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="im_livechat.mail_channel_view_tree"/>
            <field name="act_window_id" ref="im_livechat.mail_channel_action_from_livechat_channel"/>
        </record>

        <record id="mail_channel_action_livechat_form" model="ir.actions.act_window.view">
            <field name="sequence">2</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="im_livechat.mail_channel_view_form"/>
            <field name="act_window_id" ref="im_livechat.mail_channel_action_from_livechat_channel"/>
        </record>


    </data>
</odoo>

```

## File: views\rating_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="rating_rating_view_search_livechat" model="ir.ui.view">
        <field name="name">rating.rating.search.livechat</field>
        <field name="model">rating.rating</field>
        <field name="inherit_id" ref="rating.rating_rating_view_search"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='resource']" position="replace">
                <filter string="Code" name="resource" context="{'group_by':'res_name'}"/>
            </xpath>
            <xpath expr="//filter[@name='resource']" position="after">
                <field name="parent_res_name"/>
                <filter string="Livechat Channel" name="groupby_livechat_channel" context="{'group_by': 'parent_res_name'}"/>
            </xpath>
            <xpath expr="/search" position="inside">
                <filter string="This Week" name="creation_date_filter" domain="[
                ('create_date', '>=', (datetime.datetime.combine(context_today() + relativedelta(weeks=-1,days=1,weekday=0), datetime.time(0,0,0)).to_utc()).strftime('%Y-%m-%d %H:%M:%S')),
                ('create_date', '&lt;', (datetime.datetime.combine(context_today() + relativedelta(days=1,weekday=0), datetime.time(0,0,0)).to_utc()).strftime('%Y-%m-%d %H:%M:%S'))]"/>
            </xpath>
        </field>
    </record>

    <record id="rating_rating_action_view_livechat_rating" model="ir.actions.act_window">
        <field name="name">Ratings for livechat channel</field>
        <field name="res_model">rating.rating</field>
        <field name="view_mode">kanban,tree,graph,pivot,form</field>
        <field name="domain">[('parent_res_model','=','im_livechat.channel'), ('consumed','=',True)]</field>
        <field name="search_view_id" ref="rating_rating_view_search_livechat"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                There is no rating for this channel at the moment
            </p>
        </field>
        <field name="context">{'search_default_rating_last_7_days': 1}</field>
    </record>

    <record id="rating_rating_action_livechat_report" model="ir.actions.act_window">
        <field name="name">Customer Ratings</field>
        <field name="res_model">rating.rating</field>
        <field name="view_mode">kanban,tree,pivot,graph,form</field>
        <field name="domain">[('parent_res_model','=','im_livechat.channel'), ('consumed', '=', True)]</field>
        <field name="search_view_id" ref="rating_rating_view_search_livechat"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No customer ratings on live chat session yet
            </p>
        </field>
    </record>

</odoo>

```

## File: views\res_users_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>

        <!-- Update Preferences form !-->
        <record id="res_users_form_view_simple_modif" model="ir.ui.view">
            <field name="name">res.users.preferences.form.im_livechat</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="base.view_users_form_simple_modif"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='tz']" position="after">
                    <field name="livechat_username" string="Online Chat Name" readonly="0"/>
                </xpath>
            </field>
        </record>

        <!-- Update user form !-->
        <record id="res_users_form_view" model="ir.ui.view">
            <field name="name">res.users.form.im_livechat</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="base.view_users_form"/>
            <field name="arch" type="xml">
                    <xpath expr="//group[@name='messaging']" position="after">
                        <group name="livechat" string="Livechat">
                            <field name="livechat_username"/>
                        </group>
                    </xpath>
            </field>
        </record>

    </data>
</odoo>

```

