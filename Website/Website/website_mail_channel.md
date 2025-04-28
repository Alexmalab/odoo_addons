# Odoo Module: website_mail_channel

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Website Mail Channels',
    'category': 'Website/Website',
    'summary': 'Allow visitors to join public mail channels',
    'description': """
Visitors can join public mail channels managed in the Discuss app in order to get regular updates or reach out with your community.
    """,
    'depends': ['website_mail'],
    'data': [
        'data/mail_template_data.xml',
        'views/assets.xml',
        'views/snippets/s_channel.xml',
        'views/snippets/snippets.xml',
        'views/website_mail_channel_templates.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import werkzeug

from datetime import datetime
from dateutil import relativedelta

from odoo import http, fields, tools, _
from odoo.http import request
from odoo.addons.http_routing.models.ir_http import slug


class MailGroup(http.Controller):
    _thread_per_page = 20
    _replies_per_page = 10

    def _get_archives(self, group_id):
        MailMessage = request.env['mail.message']
        # use sudo to avoid side-effects due to custom ACLs
        groups = MailMessage.sudo()._read_group_raw(
            [('model', '=', 'mail.channel'), ('res_id', '=', group_id), ('message_type', '!=', 'notification')],
            ['subject', 'date'],
            groupby=["date"], orderby="date desc")
        for group in groups:
            (r, label) = group['date']
            start, end = r.split('/')
            group['date'] = label
            group['date_begin'] = self._to_date(start)
            group['date_end'] = self._to_date(end)
        return groups

    def _to_date(self, dt):
        """ date is (of course) a datetime so start and end are datetime
        strings, but we just want date strings
        """
        return (datetime
            .strptime(dt, tools.DEFAULT_SERVER_DATETIME_FORMAT)
            .date() # may be unnecessary?
            .strftime(tools.DEFAULT_SERVER_DATE_FORMAT))

    @http.route("/groups", type='http', auth="public", website=True, sitemap=True)
    def view(self, **post):
        groups = request.env['mail.channel'].search([('alias_id.alias_name', '!=', False)])

        # compute statistics
        month_date = datetime.today() - relativedelta.relativedelta(months=1)
        # use sudo to avoid side-effects due to custom ACLs
        messages = request.env['mail.message'].sudo().read_group([
            ('model', '=', 'mail.channel'),
            ('date', '>=', fields.Datetime.to_string(month_date)),
            ('message_type', '!=', 'notification'),
            ('res_id', 'in', groups.ids),
        ], ['res_id'], ['res_id'])
        message_data = dict((message['res_id'], message['res_id_count']) for message in messages)

        group_data = dict(
            (group.id, {'monthly_message_nbr': message_data.get(group.id, 0),
                        'members_count': len(group.channel_partner_ids)})
            for group in groups.sudo())
        return request.render('website_mail_channel.mail_channels', {'groups': groups, 'group_data': group_data})

    @http.route(["/groups/is_member"], type='json', auth="public", website=True)
    def is_member(self, channel_id=0, **kw):
        """ Determine if the current user is member of the given channel_id
            :param channel_id : the channel_id to check
        """
        current_user = request.env.user
        session_partner_id = request.session.get('partner_id')
        public_user = request.website.user_id
        partner = None
        # find the current partner
        if current_user != public_user:
            partner = current_user.partner_id
        elif session_partner_id:
            partner = request.env['res.partner'].sudo().browse(session_partner_id)

        values = {
            'is_user': current_user != public_user,
            'email': partner.email if partner else "",
            'is_member': False,
            'alias_name': False,
        }
        # check if the current partner is member or not
        channel = request.env['mail.channel'].browse(int(channel_id))
        if channel.exists() and partner is not None:
            values['is_member'] = bool(partner in channel.sudo().channel_partner_ids)
        return values

    @http.route(["/groups/subscription"], type='json', auth="public", website=True)
    def subscription(self, channel_id=0, subscription="on", email='', **kw):
        """ Subscribe to a mailing list : this will create a partner with its email address (if public user not
            registered yet) and add it as channel member
            :param channel_id : the channel id to join/quit
            :param subscription : 'on' to unsubscribe the user, 'off' to subscribe
        """
        unsubscribe = subscription == 'on'
        channel = request.env['mail.channel'].browse(int(channel_id))
        if not channel.exists():
            return False

        partner_ids = []

        # search partner_id
        if request.env.user != request.website.user_id:
            # connected users are directly (un)subscribed
            partner_ids = request.env.user.partner_id.ids

            # add or remove channel members
            if unsubscribe:
                channel.check_access_rule('read')
                channel.sudo().write({'channel_partner_ids': [(3, partner_id) for partner_id in partner_ids]})
                return "off"
            else:  # add partner to the channel
                request.session['partner_id'] = partner_ids[0]
                channel.check_access_rule('read')
                channel.sudo().write({'channel_partner_ids': [(4, partner_id) for partner_id in partner_ids]})
            return "on"

        else:
            # public users will recieve confirmation email
            partner_ids = [p.id for p in request.env['mail.thread'].sudo()._mail_find_partner_from_emails([email], records=channel.sudo()) if p]
            if not partner_ids or not partner_ids[0]:
                name = email.split('@')[0]
                partner_ids = [request.env['res.partner'].sudo().create({'name': name, 'email': email}).id]

            channel.sudo()._send_confirmation_email(partner_ids, unsubscribe)
            return "email"

    @http.route([
        '''/groups/<model('mail.channel', "[('channel_type', '=', 'channel')]"):group>''',
        '''/groups/<model('mail.channel'):group>/page/<int:page>'''
    ], type='http', auth="public", website=True, sitemap=True)
    def thread_headers(self, group, page=1, mode='thread', date_begin=None, date_end=None, **post):
        if group.channel_type != 'channel':
            raise werkzeug.exceptions.NotFound()

        Message = request.env['mail.message']

        domain = [('model', '=', 'mail.channel'), ('res_id', '=', group.id), ('message_type', '!=', 'notification')]
        if mode == 'thread':
            domain += [('parent_id', '=', False)]
        if date_begin and date_end:
            domain += [('date', '>=', date_begin), ('date', '<=', date_end)]

        pager = request.website.pager(
            url='/groups/%s' % slug(group),
            total=Message.search_count(domain),
            page=page,
            step=self._thread_per_page,
            url_args={'mode': mode, 'date_begin': date_begin or '', 'date_end': date_end or ''},
        )
        messages = Message.search(domain, limit=self._thread_per_page, offset=pager['offset'])
        values = {
            'messages': messages,
            'group': group,
            'pager': pager,
            'mode': mode,
            'archives': self._get_archives(group.id),
            'date_begin': date_begin,
            'date_end': date_end,
            'replies_per_page': self._replies_per_page,
        }
        return request.render('website_mail_channel.group_messages', values)

    @http.route([
        '''/groups/<model('mail.channel', "[('channel_type', '=', 'channel')]"):group>/<model('mail.message', "[('model','=','mail.channel'), ('res_id','=',group.id)]"):message>''',
    ], type='http', auth="public", website=True, sitemap=True)
    def thread_discussion(self, group, message, mode='thread', date_begin=None, date_end=None, **post):
        if group.channel_type != 'channel':
            raise werkzeug.exceptions.NotFound()

        Message = request.env['mail.message']
        if mode == 'thread':
            base_domain = [('model', '=', 'mail.channel'), ('res_id', '=', group.id), ('parent_id', '=', message.parent_id and message.parent_id.id or False)]
        else:
            base_domain = [('model', '=', 'mail.channel'), ('res_id', '=', group.id)]
        next_message = Message.search(base_domain + [('date', '<', message.date)], order="date DESC", limit=1) or None
        prev_message = Message.search(base_domain + [('date', '>', message.date)], order="date", limit=1) or None
        values = {
            'message': message,
            'group': group,
            'mode': mode,
            'archives': self._get_archives(group.id),
            'date_begin': date_begin,
            'date_end': date_end,
            'replies_per_page': self._replies_per_page,
            'next_message': next_message,
            'prev_message': prev_message,
        }
        return request.render('website_mail_channel.group_message', values)

    @http.route(
        '''/groups/<model('mail.channel', "[('channel_type', '=', 'channel')]"):group>/<model('mail.message', "[('model','=','mail.channel'), ('res_id','=',group.id)]"):message>/get_replies''',
        type='json', auth="public", methods=['POST'], website=True)
    def render_messages(self, group, message, **post):
        if group.channel_type != 'channel':
            return False

        last_displayed_id = post.get('last_displayed_id')
        if not last_displayed_id:
            return False

        replies_domain = [('id', '<', int(last_displayed_id)), ('parent_id', '=', message.id)]
        messages = request.env['mail.message'].search(replies_domain, limit=self._replies_per_page)
        message_count = request.env['mail.message'].search_count(replies_domain)
        values = {
            'group': group,
            'thread_header': message,
            'messages': messages,
            'msg_more_count': message_count - self._replies_per_page,
            'replies_per_page': self._replies_per_page,
        }
        return request.env.ref('website_mail_channel.messages_short')._render(values, engine='ir.qweb')

    @http.route("/groups/<int:group_id>/get_alias_info", type='json', auth='public', website=True)
    def get_alias_info(self, group_id, **post):
        group = request.env['mail.channel'].search([('id', '=', group_id)])
        if not group:  # doesn't exist or doesn't have the right to access it
            return {}

        return {
            'alias_name': group.alias_id and group.alias_id.alias_name and group.alias_id.alias_domain and '%s@%s' % (group.alias_id.alias_name, group.alias_id.alias_domain) or False
        }

    @http.route("/groups/subscribe/<model('mail.channel'):channel>/<int:partner_id>/<string:token>", type='http', auth='public', website=True)
    def confirm_subscribe(self, channel, partner_id, token, **kw):
        subscriber = request.env['mail.channel.partner'].search([('channel_id', '=', channel.id), ('partner_id', '=', partner_id)])
        if subscriber:
            # already registered, maybe clicked twice
            return request.render('website_mail_channel.invalid_token_subscription')

        subscriber_token = channel._generate_action_token(partner_id, action='subscribe')
        if token != subscriber_token:
            return request.render('website_mail_channel.invalid_token_subscription')

        # add partner
        channel.sudo().write({'channel_partner_ids': [(4, partner_id)]})

        return request.render("website_mail_channel.confirmation_subscription", {'subscribing': True})

    @http.route("/groups/unsubscribe/<model('mail.channel'):channel>/<int:partner_id>/<string:token>", type='http', auth='public', website=True)
    def confirm_unsubscribe(self, channel, partner_id, token, **kw):
        subscriber = request.env['mail.channel.partner'].search([('channel_id', '=', channel.id), ('partner_id', '=', partner_id)])
        if not subscriber:
            partner = request.env['res.partner'].browse(partner_id).sudo().exists()
            # FIXME: remove try/except in master
            try:
                response = request.render(
                    'website_mail_channel.not_subscribed',
                    {'partner_id': partner})
                # make sure the rendering (and thus error if template is
                # missing) happens inside the try block
                response.flatten()
                return response
            except ValueError:
                return _("The address %s is already unsubscribed or was never subscribed to any mailing list") % (
                    partner.email
                )

        subscriber_token = channel._generate_action_token(partner_id, action='unsubscribe')
        if token != subscriber_token:
            return request.render('website_mail_channel.invalid_token_subscription')

        # remove partner
        channel.sudo().write({'channel_partner_ids': [(3, partner_id)]})

        return request.render("website_mail_channel.confirmation_subscription", {'subscribing': False})

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import main

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <data>
        <record id="mail_template_list_subscribe" model="mail.template">
            <field name="name">Channel: Mailing list subscription</field>
            <field name="model_id" ref="mail.model_mail_channel"/>
            <field name="subject">Confirm subscription to ${object.name}</field>
            <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 16px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Your Channel</span><br/>
                    <span style="font-size: 20px; font-weight: bold;">${object.name}</span>
                </td><td valign="middle" align="right">
                    <img src="/logo.png?company=${user.company_id.id}" style="padding: 0px; margin: 0px; height: auto; width: 80px;" alt="${user.company_id.name}"/>
                </td></tr>
                <tr><td colspan="2" style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin:16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- CONTENT -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="top" style="font-size: 13px;">
                    <div style="margin: 0px; padding: 0px;">
                        Hello,<br/><br/>
                        You have requested to be subscribed to the mailing list <strong>${object.name}</strong>.
                        <br/><br/>
                        To confirm, please visit the following link: <strong><a href="${ctx['token_url']}">${ctx['token_url']}</a></strong>
                        <br/><br/>
                        If this was a mistake or you did not requested this action, please ignore this message.
                        % if user.signature
                            <br/>
                            ${user.signature | safe}
                        % endif
                    </div>
                </td></tr>
                <tr><td style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- FOOTER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; font-size: 11px; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle" align="left">
                        ${user.company_id.name}
                </td></tr>
                <tr><td valign="middle" align="left" style="opacity: 0.7;">
                    % if user.company_id.phone
                        ${user.company_id.phone} |
                    %endif
                    % if user.company_id.email
                        <a href="'mailto:%s' % ${user.company_id.email}" style="text-decoration:none; color: #454748;">${user.company_id.email}</a> |
                    % endif
                    % if user.company_id.website
                        <a href="'%s' % ${user.company_id.website}" style="text-decoration:none; color: #454748;">${user.company_id.website}
                        </a>
                    % endif
                </td></tr>
            </table>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- POWERED BY -->
<tr><td align="center" style="min-width: 590px;">
    <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
      <tr><td style="text-align: center; font-size: 13px;">
        Powered by <a target="_blank" href="https://www.odoo.com?utm_source=db&amp;utm_medium=mail" style="color: #875A7B;">Odoo</a>
      </td></tr>
    </table>
</td></tr>
</table>
            </field>
            <field name="auto_delete" eval="True"/>
        </record>

        <record id="mail_template_list_unsubscribe" model="mail.template">
            <field name="name">Channel: Mailing list unsubscription</field>
            <field name="model_id" ref="mail.model_mail_channel"/>
            <field name="subject">Confirm unsubscription to ${object.name}</field>
            <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 16px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Your Channel</span><br/>
                    <span style="font-size: 20px; font-weight: bold;">${object.name}</span>
                </td><td valign="middle" align="right">
                    <img src="/logo.png?company=${user.company_id.id}" style="padding: 0px; margin: 0px; height: auto; width: 80px;" alt="${user.company_id.name}"/>
                </td></tr>
                <tr><td colspan="2" style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin:16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- CONTENT -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="top" style="font-size: 13px;">
                    <div style="margin: 0px; padding: 0px;">
                        Hello,<br/><br/>
                        You have requested to be unsubscribed to the mailing list <strong>${object.name}</strong>.
                        <br/><br/>
                        To confirm, please visit the following link: <strong><a href="${ctx['token_url']}">${ctx['token_url']}</a></strong>.
                        <br/><br/>
                        If this was a mistake or you did not requested this action, please ignore this message.
                        % if user.signature:
                            <br/>
                            ${user.signature | safe}
                        % endif
                    </div>
                </td></tr>
                <tr><td style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- FOOTER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; font-size: 11px; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle" align="left">
                        ${user.company_id.name}
                </td></tr>
                <tr><td valign="middle" align="left" style="opacity: 0.7;">
                    % if user.company_id.phone
                        ${user.company_id.phone} |
                    %endif
                    % if user.company_id.email
                        <a href="'mailto:%s' % ${user.company_id.email}" style="text-decoration:none; color: #454748;">${user.company_id.email}</a> |
                    % endif
                    % if user.company_id.website
                        <a href="'%s' % ${user.company_id.website}" style="text-decoration:none; color: #454748;">${user.company_id.website}
                        </a>
                    % endif
                </td></tr>
            </table>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- POWERED BY -->
<tr><td align="center" style="min-width: 590px;">
    <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
      <tr><td style="text-align: center; font-size: 13px;">
        Powered by <a target="_blank" href="https://www.odoo.com?utm_source=db&amp;utm_medium=mail" style="color: #875A7B;">Odoo</a>
      </td></tr>
    </table>
</td></tr>
</table>
            </field>
            <field name="auto_delete" eval="True"/>
        </record>
    </data>

</odoo>

```

## File: models\mail_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib
import hmac

from werkzeug import urls

from odoo import models
from odoo.addons.http_routing.models.ir_http import slug


class MailGroup(models.Model):
    _inherit = 'mail.channel'

    def _notify_email_header_dict(self):
        headers = super(MailGroup, self)._notify_email_header_dict()
        base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        headers['List-Archive'] = '<%s/groups/%s>' % (base_url, slug(self))
        headers['List-Subscribe'] = '<%s/groups>' % (base_url)
        headers['List-Unsubscribe'] = '<%s/groups?unsubscribe>' % (base_url,)
        return headers

    def _send_confirmation_email(self, partner_ids, unsubscribe=False):
        website = self.env['website'].get_current_website()
        base_url = website.get_base_url()

        route = "/groups/%(action)s/%(channel)s/%(partner)s/%(token)s"
        if unsubscribe:
            template = self.env.ref('website_mail_channel.mail_template_list_unsubscribe')
            action = 'unsubscribe'
        else:
            template = self.env.ref('website_mail_channel.mail_template_list_subscribe')
            action = 'subscribe'

        for partner_id in partner_ids:
            # generate a new token per subscriber
            token = self._generate_action_token(partner_id, action=action)

            token_url = urls.url_join(base_url, route % {
                'action': action,
                'channel': self.id,
                'partner': partner_id,
                'token': token,
            })
            template.with_context(token_url=token_url).send_mail(
                self.id,
                force_send=True,
                email_values={
                    'recipient_ids': [(4, partner_id)],
                    'email_from': website.company_id.email,
                }
            )

        return True

    def _generate_action_token(self, partner_id, action='unsubscribe'):
        self.ensure_one()
        secret = self.env['ir.config_parameter'].sudo().get_param('database.secret')
        data = '$'.join([
                str(self.id),
                str(partner_id),
                action])
        return hmac.new(secret.encode('utf-8'), data.encode('utf-8'), hashlib.md5).hexdigest()

```

## File: models\mail_mail.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, tools, _
from odoo.addons.http_routing.models.ir_http import slug


class MailMail(models.Model):
    _inherit = 'mail.mail'

    def _send_prepare_body(self):
        """ Short-circuit parent method for mail groups, replace the default
            footer with one appropriate for mailing-lists."""
        if self.model == 'mail.channel' and self.res_id:
            # no super() call on purpose, no private links that could be quoted!
            channel = self.env['mail.channel'].browse(self.res_id)
            base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
            vals = {
                'maillist': _('Mailing-List'),
                'post_to': _('Post to'),
                'unsub': _('Unsubscribe'),
                'mailto': 'mailto:%s@%s' % (channel.alias_name, channel.alias_domain),
                'group_url': '%s/groups/%s' % (base_url, slug(channel)),
                'unsub_url': '%s/groups?unsubscribe' % (base_url,),
            }
            footer = """_______________________________________________
                        %(maillist)s: %(group_url)s
                        %(post_to)s: %(mailto)s
                        %(unsub)s: %(unsub_url)s
                    """ % vals
            body = tools.append_content_to_html(self.body, footer, container_tag='div')
            return body
        else:
            return super(MailMail, self)._send_prepare_body()

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
        suggested_controllers.append((_('Mailing Lists'), url_for('/groups'), 'website_mail_channel'))
        return suggested_controllers

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import mail_channel
from . import mail_mail
from . import website

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#CD7690"/><stop offset="100%" stop-color="#CA5377"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M42.474 69H4c-2 0-4-.146-4-4.075V39.178L21 15h30l-2 6.113h3l-3 5.095h6l-5 10.188 4.213 1.034L53 54.736 42.474 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M15 36.646h12.813l2.562 3.416h10.25l3.417-3.416H56l-2.563 19.646H17.563L15 36.646zm.854-10.25h39.292v7.687h-5.98v-3.416H20.98v3.416h-5.125v-7.687zm2.563-5.125h34.166v3.416H18.417v-3.416zM20.979 17h29.896v2.563H20.979V17z" opacity=".3"/><path fill="#FFF" d="M15 34.646h12.813l2.562 3.416h10.25l3.417-3.416H56l-2.563 19.646H17.563L15 34.646zm.854-10.25h39.292v7.687h-5.98v-3.416H20.98v3.416h-5.125v-7.687zm2.563-5.125h34.166v3.416H18.417v-3.416zM20.979 15h29.896v2.563H20.979V15z"/></g></g></svg>
```

## File: static\src\js\website_mail_channel.js

```javascript
odoo.define('website_mail_channel', function (require) {
'use strict';

var publicWidget = require('web.public.widget');

publicWidget.registry.websiteMailChannel = publicWidget.Widget.extend({
    selector: '#wrapwrap',
    events: {
        'click .o_mg_link_hide': '_onHideLinkClick',
        'click .o_mg_link_show': '_onShowLinkClick',
        'click button.o_mg_read_more': '_onReadMoreClick',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onHideLinkClick: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        var $link = $(ev.currentTarget);
        var $container = $link.parents('div').first();
        $container.find('.o_mg_link_hide').first().hide();
        $container.find('.o_mg_link_show').first().show();
        $container.find('.o_mg_link_content').first().show();
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onShowLinkClick: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        var $link = $(ev.currentTarget);
        var $container = $link.parents('div').first();
        $container.find('.o_mg_link_hide').first().show();
        $container.find('.o_mg_link_show').first().hide();
        $container.find('.o_mg_link_content').first().hide();
    },
    /**
     * @private
     * @param {Event} ev
     */
     _onReadMoreClick: function (ev) {
        var $link = $(ev.target);
        this._rpc({
            route: $link.data('href'),
            params: {
                last_displayed_id: $link.data('msg-id'),
            },
        }).then(function (data) {
            if (!data) {
                return;
            }
            var $threadContainer = $link.parents('.o_mg_replies').first().find('ul.list-unstyled');
            if ($threadContainer) {
                var $lastMsg = $threadContainer.find('li.media').last();
                $(data).find('li.media').insertAfter($lastMsg);
                $(data).find('.o_mg_read_more').parent().appendTo($threadContainer);
            }
            var $showMore = $link.parent();
            $showMore.remove();
            return;
        });
     },
});
});

```

## File: static\src\snippets\s_channel\000.js

```javascript
odoo.define('website_mail_channel.s_channel', function (require) {
'use strict';

var publicWidget = require('web.public.widget');

publicWidget.registry.Channel = publicWidget.Widget.extend({
    selector: '.s_channel',
    disabledInEditableMode: false,
    read_events: {
        'click .js_follow_btn, .js_unfollow_btn': '_onFollowUnFollowBtnClick',
        'click .js_follow_btn': '_onFollowBtnClick',
    },

    /**
     * @override
     */
    start: function () {
        var self = this;
        this.is_user = false;
        var unsubscribePage = window.location.search.slice(1).split('&').indexOf("unsubscribe") >= 0;

        var always = function (data) {
            self.is_user = data.is_user;
            self.email = data.email;
            self.$target.find('.js_mg_link').attr('href', '/groups/' + self.$target.data('id'));
            if (unsubscribePage && self.is_user) {
                self.$target.find(".js_mg_follow_form").remove();
            }
            self._toggleSubscription(data.is_member ? 'on' : 'off', data.email);
            self.$target.removeClass('d-none');
        };

        this._rpc({
            route: '/groups/is_member',
            params: {
                model: this.$target.data('object'),
                channel_id: this.$target.data('id'),
                get_alias_info: true,
            },
        }).then(always).guardedCatch(always);

        // not if editable mode to allow designer to edit alert field
        if (!this.editableMode) {
            this.$('> .alert').addClass('d-none');
            this.$('> .input-group-append.d-none').removeClass('d-none');
        }
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    destroy: function () {
        this.el.classList.add('d-none');
        this._super(...arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _getAliasInfo: function () {
        var self = this;
        if (! this.$target.data('id')) {
            return Promise.resolve();
        }
        return this._rpc({route: '/groups/' + this.$target.data('id') + '/get_alias_info'}).then(function (data) {
            if (data.alias_name) {
                self.$target.find('.js_mg_email').attr('href', 'mailto:' + data.alias_name);
                self.$target.find('.js_mg_email').removeClass('d-none');
            } else {
                self.$target.find('.js_mg_email').addClass('d-none');
            }
        });
    },
    /**
     * @private
     */
    _toggleSubscription: function (follow, email) {
        // .js_mg_follow_form contains subscribe form
        // .js_mg_details contains send/archives/unsubscribe links
        // .js_mg_confirmation contains message warning has been sent
        var aliasDone = this._getAliasInfo();
        if (follow === "on") {
            // user is connected and can unsubscribe
            this.$target.find(".js_mg_follow_form").addClass('d-none');
            this.$target.find(".js_mg_details").removeClass('d-none');
            this.$target.find(".js_mg_confirmation").addClass('d-none');
        } else if (follow === "off") {
            // user is connected and can subscribe
            this.$target.find(".js_mg_follow_form").removeClass('d-none');
            this.$target.find(".js_mg_details").addClass('d-none');
            this.$target.find(".js_mg_confirmation").addClass('d-none');
        } else if (follow === "email") {
            // a confirmation email has been sent
            this.$target.find(".js_mg_follow_form").addClass('d-none');
            this.$target.find(".js_mg_details").addClass('d-none');
            this.$target.find(".js_mg_confirmation").removeClass('d-none');
        } else {
            console.error("Unknown subscription action", follow);
        }
        this.$target.find('input.js_follow_email')
            .val(email ? email : "")
            .attr("disabled", follow === "on" || (email.length && this.is_user) ? "disabled" : false);
        this.$target.attr("data-follow", follow);
        return Promise.resolve(aliasDone);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onFollowBtnClick: function (ev) {
        if ($(ev.currentTarget).closest('.js_mg_follow_form').length) {
            this.$('.js_follow_email').val($(ev.currentTarget).closest('.js_mg_follow_form').find('.js_follow_email').val());
        }
    },
    /**
     * @private
     */
    _onFollowUnFollowBtnClick: function (ev) {
        ev.preventDefault();
        var self = this;
        var $email = this.$target.find(".js_follow_email");

        if ($email.length && !$email.val().match(/.+@.+/)) {
            this.$target.addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
            return false;
        }
        this.$target.removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');

        var subscriptionAction = this.$target.attr("data-follow") || "off";
        if (window.location.search.slice(1).split('&').indexOf("unsubscribe") >= 0) {
            // force unsubscribe mode via URI /groups?unsubscribe
            subscriptionAction = 'on';
        }
        this._rpc({
            route: '/groups/subscription',
            params: {
                'channel_id': +this.$target.data('id'),
                'object': this.$target.data('object'),
                'subscription': subscriptionAction,
                'email': $email.length ? $email.val() : false,
            },
        }).then(function (follow) {
            self._toggleSubscription(follow, self.email);
        });
    },
});
});

```

## File: static\src\snippets\s_channel\options.js

```javascript
odoo.define('website_mail_channel.s_channel_options', function (require) {
'use strict';

var core = require('web.core');
var options = require('web_editor.snippets.options');
var wUtils = require('website.utils');

var _t = core._t;

options.registry.Channel = options.Class.extend({
    /**
     * @override
     */
    async start() {
        await this._super(...arguments);
        this.publicChannels = await this._getPublicChannels();
    },
    /**
     * If we have already created channels => select the first one
     * else => modal prompt (create a new channel)
     *
     * @override
     */
    onBuilt() {
        if (this.publicChannels.length) {
            this.$target[0].dataset.id = this.publicChannels[0][0];
        } else {
            const widget = this._requestUserValueWidgets('create_mail_channel_opt')[0];
            widget.$el.click();
        }
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Creates a new mail.channel through a modal prompt.
     *
     * @see this.selectClass for parameters
     */
    createChannel: function (previewMode, widgetValue, params) {
        var self = this;
        return wUtils.prompt({
            id: "editor_new_mail_channel_subscribe",
            window_title: _t("New Mail Channel"),
            input: _t("Name"),
        }).then(function (result) {
            var name = result.val;
            if (!name) {
                return;
            }
            return self._rpc({
                model: 'mail.channel',
                method: 'create',
                args: [{
                    name: name,
                    public: 'public',
                }],
            }).then(function (id) {
                self.$target.attr("data-id", id);
                return self._rerenderXML();
            });
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _renderCustomXML(uiFragment) {
        // TODO remove this part in master 
        const createChannelEl = uiFragment.querySelector('we-button[data-create-channel]');
        createChannelEl.dataset.name = 'create_mail_channel_opt';

        return this._getPublicChannels().then(channels => {
            const menuEl = uiFragment.querySelector('.select_discussion_list');
            for (const channel of channels) {
                const el = document.createElement('we-button');
                el.dataset.selectDataAttribute = channel[0];
                el.textContent = channel[1];
                menuEl.appendChild(el);
            }
        });
    },
    /**
     * @private
     * @return {Promise}
     */
    _getPublicChannels() {
        return this._rpc({
            model: 'mail.channel',
            method: 'name_search',
            args: ['', [['public', '=', 'public']]],
        });
    },
});
});

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="assets_frontend" inherit_id="website.assets_frontend">
    <xpath expr="//link[last()]" position="after">
        <link rel="stylesheet" href="/website_mail_channel/static/src/css/website_mail_channel.css"/>
    </xpath>
    <xpath expr="//script[last()]" position="after">
        <script type="text/javascript" src="/website_mail_channel/static/src/js/website_mail_channel.js"/>
    </xpath>
</template>

<template id="assets_wysiwyg" inherit_id="website.assets_wysiwyg" name="Website Mail Channel Editor Assets">
    <xpath expr="//script[last()]" position="after">
        <script type="text/javascript" src="/website_mail_channel/static/src/snippets/s_channel/options.js"/>
    </xpath>
</template>

</odoo>

```

## File: views\website_mail_channel_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="mail_channels" name="Mailing Lists">
    <t t-call="website.layout">
        <div id="wrap" class="oe_structure oe_empty">
            <section class="bg-primary jumbotron mt0 mb0">
                <div class="container">
                    <h1>Stay in touch with our Community</h1>
                    <p>Alone we can do so little, together we can do so much</p>
                </div>
            </section>
        </div>
        <div class="container mt32">
            <div t-if="'unsubscribe' in request.params" class="offset-lg-9 col-lg-3 alert alert-info" role="status">
               <h3>Need to unsubscribe? It's right here! <span class="fa fa-2x fa-arrow-down float-right" role="img" aria-label="" title="Read this !"></span></h3>
            </div>
            <div class="row mt8" t-foreach="groups" t-as="group">
                <div class="col-lg-3">
                    <img t-att-src="website.image_url(group, 'image_128')" class="o_image_64_cover float-left" alt="Group"/>
                    <strong><a t-attf-href="/groups/#{ slug(group) }" t-esc="group.name"/></strong><br />
                    <t t-if="group.alias_id and group.alias_id.alias_name and group.alias_id.alias_domain">
                        <i class='fa fa-envelope-o' role="img" aria-label="Alias" title="Alias"/>
                        <a t-attf-href="mailto:#{group.alias_id.alias_name}@#{group.alias_id.alias_domain}"><span t-field="group.alias_id"/></a>
                    </t>
                </div>
                <div class="col-lg-4">
                    <div t-esc="group.description" class="text-muted"/>
                </div>
                <div class="col-lg-2">
                    <i class='fa fa-fw fa-user' role="img" aria-label="Recipients" title="Recipients"/> <t t-esc="group_data[group.id]['members_count']"/> members<br />
                    <i class='fa fa-fw fa-envelope-o' role="img" aria-label="Traffic" title="Traffic"/> <t t-raw="group_data[group.id]['monthly_message_nbr']"/> messages / month
                </div>
                <div class="col-lg-3">
                    <!--<t t-call="website_mail.follow"><t t-set="object" t-value="group"/></t>-->

                    <div class="s_channel"
                              t-att-data-id="group.id"
                              data-object="mail.channel"
                              t-att-data-follow="'on' if 'unsubscribe' in request.params else 'off'"
                              data-snippet="s_channel">
                        <div class="input-group js_mg_follow_form">
                            <input
                                  type="email"
                                  name="email"
                                  class="js_follow_email form-control"
                                  placeholder="your email..."/>
                            <div t-if="'unsubscribe' not in request.params" class="input-group-append">
                               <button href="#" class="btn btn-primary js_follow_btn">Subscribe</button>
                            </div>
                            <div t-if="'unsubscribe' in request.params" class="input-group-append">
                               <button href="#" class="btn btn-primary js_follow_btn">Unsubscribe</button>
                            </div>
                        </div>
                        <p class="js_mg_details d-none">
                            <span class="js_mg_email d-none"><a href="#" class="js_mg_email"><i class="fa fa-envelope-o"/> send mail</a> - </span>
                            <a href="#" class="js_mg_link"><i class="fa fa-file-o"/> archives</a> -
                            <a role="button" href="#" class="js_unfollow_btn"><i class="fa fa-times"/> unsubscribe</a>
                        </p>
                    </div>


                </div>
            </div>
        </div>
    </t>
</template>

<template id="group_messages" name="Message Threads">
    <t t-call="website.layout">
        <section class="container">
            <div class="mt8">
                <ol class="breadcrumb float-left">
                    <li class="breadcrumb-item"><a href="/groups">Mailing Lists</a></li>
                    <li class="breadcrumb-item">
                        <a t-attf-href="/groups/#{slug(group)}?#{mode and 'mode=%s' % mode or ''}#{date_begin and '&amp;date_begin=%s' % date_begin or ''}#{date_end and '&amp;date_end=%s' % date_end or ''}"><t t-esc="group.name"/></a>
                    </li>
                </ol>
            </div>
            <div class="row">
                <div class="col-lg-12">
                    <h1 class="text-center">
                        <t t-esc="group.name"/> mailing list archives
                    </h1><h4 class="text-center text-muted" t-if="group.alias_id and group.alias_id.alias_name and group.alias_id.alias_domain">
                        <i class='fa fa-envelope-o' role="img" aria-label="Alias" title="Alias"/>
                        <a t-attf-href="mailto:#{group.alias_id.alias_name}@#{group.alias_id.alias_domain}"><span t-field="group.alias_id"/></a>
                    </h4>
                </div>
                <div class="col-lg-3">
                    <h2>Archives</h2>
                    <ul class="nav nav-pills flex-column" id="group_mode">
                        <li class="nav-item">
                            <a t-attf-href="/groups/#{ slug(group) }?mode=thread" t-attf-class="nav-link#{mode=='thread' and ' active' or ''}">By thread</a>
                        </li>
                        <li class="nav-item">
                            <a t-attf-href="/groups/#{ slug(group) }?mode=date" t-attf-class="nav-link#{mode=='date' and not date_begin and ' active' or ''}">By date</a>
                            <ul class="nav nav-pills flex-column" style="margin-left: 8px;">
                                <t t-foreach="archives" t-as="month_archive">
                                <li class="nav-item">
                                    <a t-ignore="True" t-attf-href="/groups/#{ slug(group) }?mode=date&amp;date_begin=#{ month_archive['date_begin'] }&amp;date_end=#{month_archive['date_end']}"
                                        t-attf-class="nav-link#{month_archive['date_begin'] == date_begin and ' active' or ''}">
                                        <t t-esc="month_archive['date']"/>
                                        <span class="float-right badge badge-pill" t-esc="month_archive['date_count']"/>
                                    </a>
                                </li>
                                </t>
                            </ul>
                        </li>
                    </ul>
                </div>
                <div class="col-lg-9">
                    <div>
                        <t t-call="website.pager"/>
                    </div>
                    <t t-call="website_mail_channel.messages_short">
                        <t t-set="messages" t-value="messages"/>
                        <t t-set="msg_more_count" t-value="0"/>
                        <t t-set="thread_header" t-value="None"/>
                    </t>
                    <div>
                        <t t-call="website.pager"/>
                    </div>
                </div>
            </div>
        </section>
    </t>
</template>

<template id="group_message">
    <t t-call="website.layout">
        <t t-set="additional_title"><t t-esc="message.description"/></t>
        <section class="container">
            <div class="row mt8">
                <ol class="breadcrumb float-left">
                    <li class="breadcrumb-item"><a href="/groups">Mailing Lists</a></li>
                    <li class="breadcrumb-item">
                        <a t-attf-href="/groups/#{slug(group)}?#{mode and 'mode=%s' % mode or ''}#{date_begin and '&amp;date_begin=%s' % date_begin or ''}#{date_end and '&amp;date_end=%s' % date_end or ''}"><t t-esc="group.name"/></a>
                    </li>
                    <li t-if="message" class="breadcrumb-item active"><t t-esc="message.description"/></li>
                </ol>
            </div>
            <div class="row">
                <h1 class="text-center">
                    <t t-esc="group.name"/> mailing list archives
                </h1><h4 class="text-center text-muted" t-if="group.alias_id and group.alias_id.alias_name and group.alias_id.alias_domain">
                    <i class='fa fa-envelope-o' role="img" aria-label="Alias" title="Alias"/>
                    <a t-attf-href="mailto:#{group.alias_id.alias_name}@#{group.alias_id.alias_domain}"><span t-field="group.alias_id"/></a>
                </h4>
            </div>
            <div class="row">
                <div class="col-lg-3">
                    <h4>Browse archives</h4>
                    <ul class="nav nav-pills flex-column" id="group_mode">
                        <li class="nav-item">
                            <a t-attf-href="/groups/#{ slug(group) }?mode=thread" t-attf-class="nav-link#{mode=='thread' and ' active' or ''}">By thread</a>
                        </li>
                        <li class="nav-item">
                            <a t-attf-href="/groups/#{ slug(group) }?mode=date" t-attf-class="nav-link#{mode=='date' and not date_begin and ' active' or ''}">By date</a>
                            <ul class="nav nav-pills flex-column" style="margin-left: 8px;">
                                <t t-foreach="archives" t-as="month_archive">
                                <li class="nav-item">
                                    <a t-ignore="True" t-attf-href="/groups/#{ slug(group) }?mode=date&amp;date_begin=#{ month_archive['date_begin'] }&amp;date_end=#{month_archive['date_end']}"
                                        t-attf-class="nav-link#{month_archive['date_begin'] == date_begin and ' active' or ''}">
                                        <t t-esc="month_archive['date']"/>
                                        <span class="float-right badge badge-pill" t-esc="month_archive['date_count']"/>
                                    </a>
                                </li>
                                </t>
                            </ul>
                        </li>
                    </ul>
                </div>
                <div class="col-lg-9">
                    <div class="row">
                        <h4 class="col-lg-6">
                            <t t-if="prev_message"><a t-attf-href='/groups/#{slug(group)}/#{slug(prev_message)}?#{mode and "mode=%s" % mode or ""}'>
                                <i class="fa fa-arrow-left" role="img" aria-label="Previous message" title="Previous message"/> <t t-esc="prev_message.description"/>
                            </a></t>
                        </h4>
                        <h4 class="col-lg-6">
                            <t t-if="next_message"><a class="float-right" t-attf-href='/groups/#{slug(group)}/#{slug(next_message)}?#{mode and "mode=%s" % mode or ""}'>
                                <t t-esc="next_message.description"/> <i class="fa fa-arrow-right" role="img" aria-label="Next message" title="Next message"/>
                            </a></t>
                        </h4>
                    </div>
                    <div class="media">
                        <img class="rounded mt0 o_image_40_cover"
                            t-att-src="website.image_url(message, 'author_avatar')"  alt="Avatar"/>
                        <div class="media-body">
                            <h4 t-esc="message.description"/>
                            <small>
                                by
                                <t t-if="message.author_id">
                                    <span t-field="message.author_id" style="display: inline-block;" t-options='{
                                        "widget": "contact",
                                        "fields": ["name"]
                                    }'/>
                                </t>
                                <t t-if="not message.author_id"><t t-esc="message.email_from"/></t>
                                - <i class="fa fa-calendar" role="img" aria-label="Date" title="Date"/> <span t-field="message.date"/>
                            </small>
                            <div t-raw="message.body"/>

                            <div>
                                <p t-if="message.attachment_ids" class="mt8">
                                    <a href="#" class="o_mg_link_hide">
                                        <i class="fa fa-chevron-right" role="img" aria-label="Hide attachments" title="Hide attachments"/> <t t-raw="len(message.attachment_ids)"/> attachments
                                    </a>
                                    <a href="#" class="o_mg_link_show">
                                        <i class="fa fa-chevron-down" role="img" aria-label="Show attachments" title="Show attachments"/> <t t-raw="len(message.attachment_ids)"/> attachments
                                    </a>
                                </p>
                                <div class="o_mg_link_content">
                                    <div class="col-lg-2 col-md-3 text-center" t-foreach='message.attachment_ids' t-as='attachment'>
                                        <a t-attf-href="/web/content/#{attachment.id}?download=true" target="_blank">
                                            <div class='oe_attachment_embedded o_image' t-att-title="attachment.name" t-att-data-mimetype="attachment.mimetype" t-attf-data-src="/web/image/#{attachment.id}/100x80"/>
                                            <div class='oe_attachment_name'><t t-raw='attachment.name' /></div>
                                        </a>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div t-if="message.child_ids" class="o_mg_replies">
                        <h4 class="o_page_header">Follow-Ups</h4>
                        <t t-call="website_mail_channel.messages_short">
                            <t t-set="messages" t-value="message.child_ids[:replies_per_page]"/>
                            <t t-set="msg_more_count" t-value="len(message.child_ids) - replies_per_page"/>
                            <t t-set="thread_header" t-value="message"/>
                        </t>
                    </div>
                    <div t-if="message.parent_id">
                        <h4 class="o_page_header">Reference</h4>
                        <t t-call="website_mail_channel.messages_short">
                            <t t-set="messages" t-value="[message.parent_id]"/>
                        </t>
                    </div>
                </div>
            </div>
        </section>
    </t>
</template>

<template id="messages_short">
    <div>
        <ul class="list-unstyled">
            <li t-foreach="messages" t-as="thread" class="media mt-3">
                <img class="rounded mt-0 o_image_40_cover" alt="Avatar"
                    t-att-src="website.image_url(thread, 'author_avatar')"/>
                <div class="media-body">
                    <h4>
                        <a t-attf-href="/groups/#{slug(group)}/#{slug(thread)}?mode=#{mode}&amp;date_begin=#{date_begin}&amp;date_end=#{date_end}" t-esc="thread.description"/>
                    </h4>
                    <small>
                        by
                        <t t-if="thread.author_id">
                            <span t-field="thread.author_id" style="display: inline-block;" t-options='{
                                "widget": "contact",
                                "fields": ["name"]
                            }'/>
                        </t>
                        <t t-if="not thread.author_id"><t t-esc="thread.email_from"/></t>
                        - <i class="fa fa-calendar" role="img" aria-label="Date" title="Date"/> <span t-field="thread.date"/>
                        - <i class="fa fa-paperclip" role="img" aria-label="Attachments" title="Attachments"/> <t t-esc="len(thread.attachment_ids)"/>
                    </small>
                    <p t-if="thread.child_ids" class="mt8">
                        <a href="#" class="o_mg_link_hide">
                            <i class="fa fa-chevron-right" role="img" aria-label="Hide replies" title="Hide replies"/> <t t-raw="len(thread.child_ids)"/> replies
                        </a>
                        <a href="#" class="o_mg_link_show">
                            <i class="fa fa-chevron-down" role="img" aria-label="Show replies" title="Show replies"/> <t t-raw="len(thread.child_ids)"/> replies
                        </a>
                    </p>
                    <div class="o_mg_link_content o_mg_replies">
                        <t t-call="website_mail_channel.messages_short">
                            <t t-set="messages" t-value="thread.child_ids[:replies_per_page]"/>
                            <t t-set="msg_more_count" t-value="len(thread.child_ids) - replies_per_page"/>
                            <t t-set="thread_header" t-value="thread"/>
                        </t>
                    </div>
                </div>
            </li>
        </ul>
        <p t-if="messages and (msg_more_count or 0) > 0 and thread_header">
            <button class="fa btn-link o_mg_read_more"
                t-attf-data-href="/groups/#{slug(group)}/#{slug(thread_header)}/get_replies"
                t-attf-data-msg-id="#{messages[-1].id}">
                show <t t-esc="msg_more_count"/> more replies
            </button>
        </p>
    </div>
</template>


<template id="confirmation_subscription" name="Mailing List Confirmation">
    <t t-call="website.layout">
        <div id="wrap" class="oe_structure oe_empty">
            <div class="container">
                <p>
                    You have been correctly
                    <t t-if="subscribing">subscribed</t>
                    <t t-if="not subscribing">unsubscribed</t>
                    to the mailing list.
                </p>
            </div>
        </div>
    </t>
</template>

<template id="invalid_token_subscription" name="Invalid Token Submitted">
    <t t-call="website.layout">
        <div id="wrap" class="oe_structure oe_empty">
            <div class="container">
                <p>
                    Invalid or expired confirmation link.
                </p>
            </div>
        </div>
    </t>
</template>

<template id="not_subscribed" name="Email address was not subscribed">
    <t t-call="website.layout">
        <div id="wrap" class="oe_structure oe_empty">
            <div class="container">
                <p>
                    The address <t t-esc="partner_id.email"/> is already
                    unsubscribed or was never subscribed to the mailing
                    list, you may want to check that the address was
                    correct.
                </p>
            </div>
        </div>
    </t>
</template>

</odoo>

```

## File: views\snippets\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="remove_external_snippets" inherit_id="website.external_snippets">
    <xpath expr="//t[@t-install='website_mail_channel']" position="replace"/>
</template>

<template id="snippets" inherit_id="website.snippets" name="Snippet Subscribe">
    <xpath expr="//t[@id='mail_channel_discussion_group_hook']" position="replace">
        <t t-snippet="website_mail_channel.s_channel" t-thumbnail="/website/static/src/img/snippets_thumbs/s_channel.svg"/>
    </xpath>
</template>

</odoo>

```

## File: views\snippets\s_channel.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_channel" name="Discussion Group">
    <div class="s_channel"
         data-id="0" data-object="mail.channel" data-follow="off">
        <div class="input-group js_mg_follow_form">
            <input type="email" name="email" placeholder="your email..."
                   class="js_follow_email form-control"/>
            <span class="input-group-append">
                <button href="#" class="btn btn-primary js_follow_btn">Subscribe</button>
            </span>
        </div>
        <p class="js_mg_details d-none">
            <span class="js_mg_email d-none"><a href="#" class="js_mg_email"><i class="fa fa-envelope-o"/> send mail</a> - </span>
            <a href="#" class="js_mg_link"><i class="fa fa-file-o"/> archives</a> -
            <a role="button" href="#" class="js_unfollow_btn"><i class="fa fa-times"/> unsubscribe</a>
        </p>
        <p class="js_mg_confirmation d-none">
            a confirmation email has been sent.
        </p>
    </div>
</template>

<template id="s_channel_options" inherit_id="website.snippet_options">
    <xpath expr="." position="inside">
        <div data-js='Channel'
             data-selector=".s_channel"
             data-drop-near="p, h1, h2, h3, blockquote, .card">
            <we-row>
                <we-select class="select_discussion_list" data-attribute-name="id" data-no-preview="true">
                    <!-- 'we-button' added programmatically with DB data -->
                </we-select>
                <we-button class="fa fa-fw fa-plus" title="Create a public discussion group in your backend"
                           data-create-channel="" data-no-preview="true" data-name="create_mail_channel_opt"/>
            </we-row>
        </div>
    </xpath>
</template>

<template id="assets_snippet_s_channel_js_000" inherit_id="website.assets_frontend">
    <xpath expr="//script[last()]" position="after">
        <script type="text/javascript" src="/website_mail_channel/static/src/snippets/s_channel/000.js"/>
    </xpath>
</template>

</odoo>

```

