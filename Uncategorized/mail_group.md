# Odoo Module: mail_group

Category: Uncategorized

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Mail Group",
    'summary': "Manage your mailing lists",
    'description': """
Manage your mailing lists from Odoo.
    """,
    'version': '1.1',
    'depends': [
        'mail',
        'portal',
    ],
    'data': [
        'data/ir_cron_data.xml',
        'data/mail_templates.xml',
        'data/mail_template_data.xml',
        'data/res_groups.xml',
        'security/ir.model.access.csv',
        'security/mail_group_security.xml',
        'wizard/mail_group_message_reject_views.xml',
        'views/mail_group_member_views.xml',
        'views/mail_group_message_views.xml',
        'views/mail_group_moderation_views.xml',
        'views/mail_group_views.xml',
        'views/mail_group_menus.xml',
        'views/portal_templates.xml',
    ],
    'demo': [
        'data/mail_group_demo.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'mail_group/static/src/css/mail_group.scss',
            'mail_group/static/src/js/*',
        ],
        'web.assets_backend': [
            'mail_group/static/src/css/mail_group_backend.scss',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import babel.dates
import werkzeug

from odoo import http, fields, tools, models
from odoo.addons.http_routing.models.ir_http import slug
from odoo.addons.portal.controllers.portal import pager as portal_pager
from odoo.exceptions import AccessError
from odoo.http import request, Response
from odoo.osv import expression
from odoo.tools import consteq
from odoo.tools.misc import get_lang


class PortalMailGroup(http.Controller):
    _thread_per_page = 20
    _replies_per_page = 5

    def _get_website_domain(self):
        # Base group domain in addition to the security access rules
        # Do not show rejected message on the portal view even for admin
        return [('moderation_status', '!=', 'rejected')]

    def _get_archives(self, group_id):
        """Return the different date range and message count for the group messages."""
        domain = expression.AND([self._get_website_domain(), [('mail_group_id', '=', group_id)]])
        results = request.env['mail.group.message']._read_group(
            domain,
            groupby=['create_date:month'],
            aggregates=['__count'],
        )

        date_groups = []

        locale = get_lang(request.env).code
        fmt = models.READ_GROUP_DISPLAY_FORMAT['month']
        interval = models.READ_GROUP_TIME_GRANULARITY['month']
        for start, count in results:
            label = babel.dates.format_datetime(start, format=fmt, locale=locale)
            date_groups.append({
                'date': label,
                'date_begin': fields.Date.to_string(start),
                'date_end': fields.Date.to_string(start + interval),
                'messages_count': count,
            })

        thread_domain = expression.AND([domain, [('group_message_parent_id', '=', False)]])
        threads_count = request.env['mail.group.message'].search_count(thread_domain)

        return {
            'threads_count': threads_count,
            'threads_time_data': date_groups,
        }

    # ------------------------------------------------------------
    # MAIN PAGE
    # ------------------------------------------------------------

    @http.route('/groups', type='http', auth='public', sitemap=True, website=True)
    def groups_index(self, email='', **kw):
        """View of the group lists. Allow the users to subscribe and unsubscribe."""
        if kw.get('group_id') and kw.get('token'):
            group_id = int(kw.get('group_id'))
            token = kw.get('token')
            group = request.env['mail.group'].browse(group_id).exists().sudo()
            if not group:
                raise werkzeug.exceptions.NotFound()

            if token != group._generate_group_access_token():
                raise werkzeug.exceptions.NotFound()

            mail_groups = group

        else:
            mail_groups = request.env['mail.group'].search([]).sudo()

        if not request.env.user._is_public():
            # Force the email if the user is logged
            email_normalized = request.env.user.email_normalized
            partner_id = request.env.user.partner_id.id
        else:
            email_normalized = tools.email_normalize(email)
            partner_id = None

        members_data = mail_groups._find_members(email_normalized, partner_id)

        return request.render('mail_group.mail_groups', {
            'mail_groups': [{
                'group': group,
                'is_member': bool(members_data.get(group.id, False)),
            } for group in mail_groups],
            'email': email_normalized,
            'is_mail_group_manager': request.env.user.has_group('mail_group.group_mail_group_manager'),
        })

    # ------------------------------------------------------------
    # THREAD DISPLAY / MANAGEMENT
    # ------------------------------------------------------------

    @http.route([
        '/groups/<model("mail.group"):group>',
        '/groups/<model("mail.group"):group>/page/<int:page>',
    ], type='http', auth='public', sitemap=True, website=True)
    def group_view_messages(self, group, page=1, mode='thread', date_begin=None, date_end=None, **post):
        GroupMessage = request.env['mail.group.message']

        domain = expression.AND([self._get_website_domain(), [('mail_group_id', '=', group.id)]])
        if mode == 'thread':
            domain = expression.AND([domain, [('group_message_parent_id', '=', False)]])

        if date_begin and date_end:
            domain = expression.AND([domain, [('create_date', '>', date_begin), ('create_date', '<=', date_end)]])

        # SUDO after the search to apply access rules but be able to read attachments
        messages_sudo = GroupMessage.search(
            domain, limit=self._thread_per_page,
            offset=(page - 1) * self._thread_per_page).sudo()

        pager = portal_pager(
            url=f'/groups/{slug(group)}',
            total=GroupMessage.search_count(domain),
            page=page,
            step=self._thread_per_page,
            scope=5,
            url_args={'date_begin': date_begin, 'date_end': date_end, 'mode': mode}
        )

        self._generate_attachments_access_token(messages_sudo)

        return request.render('mail_group.group_messages', {
            'page_name': 'groups',
            'group': group,
            'messages': messages_sudo,
            'archives': self._get_archives(group.id),
            'date_begin': date_begin,
            'date_end': date_end,
            'pager': pager,
            'replies_per_page': self._replies_per_page,
            'mode': mode,
        })

    @http.route('/groups/<model("mail.group"):group>/<model("mail.group.message"):message>',
                type='http', auth='public', sitemap=False, website=True)
    def group_view_message(self, group, message, mode='thread', date_begin=None, date_end=None, **post):
        if group != message.mail_group_id:
            raise werkzeug.exceptions.NotFound()

        GroupMessage = request.env['mail.group.message']
        base_domain = expression.AND([
            self._get_website_domain(),
            [('mail_group_id', '=', group.id),
             ('group_message_parent_id', '=', message.group_message_parent_id.id)],
        ])

        next_message = GroupMessage.search(
            expression.AND([base_domain, [('id', '>', message.id)]]),
            order='id ASC', limit=1)
        prev_message = GroupMessage.search(
            expression.AND([base_domain, [('id', '<', message.id)]]),
            order='id DESC', limit=1)

        message_sudo = message.sudo()
        self._generate_attachments_access_token(message_sudo)

        values = {
            'page_name': 'groups',
            'message': message_sudo,
            'group': group,
            'mode': mode,
            'archives': self._get_archives(group.id),
            'date_begin': date_begin,
            'date_end': date_end,
            'replies_per_page': self._replies_per_page,
            'next_message': next_message,
            'prev_message': prev_message,
        }
        return request.render('mail_group.group_message', values)

    @http.route('/groups/<model("mail.group"):group>/<model("mail.group.message"):message>/get_replies',
                type='json', auth='public', methods=['POST'], website=True)
    def group_message_get_replies(self, group, message, last_displayed_id, **post):
        if group != message.mail_group_id:
            raise werkzeug.exceptions.NotFound()

        replies_domain = expression.AND([
            self._get_website_domain(),
            [('id', '>', int(last_displayed_id)), ('group_message_parent_id', '=', message.id)],
        ])
        # SUDO after the search to apply access rules but be able to read attachments
        replies_sudo = request.env['mail.group.message'].search(replies_domain, limit=self._replies_per_page).sudo()
        message_count = request.env['mail.group.message'].search_count(replies_domain)

        if not replies_sudo:
            return

        message_sudo = message.sudo()

        self._generate_attachments_access_token(message_sudo | replies_sudo)

        values = {
            'group': group,
            'parent_message': message_sudo,
            'messages': replies_sudo,
            'msg_more_count': message_count - self._replies_per_page,
            'replies_per_page': self._replies_per_page,
        }
        return request.env['ir.qweb']._render('mail_group.messages_short', values)

    # ------------------------------------------------------------
    # SUBSCRIPTION
    # ------------------------------------------------------------

    # csrf is disabled here because it will be called by the MUA with unpredictable session at that time
    @http.route('/group/<int:group_id>/unsubscribe_oneclick', website=True, type='http', auth='public',
           methods=['POST'], csrf=False)
    def group_unsubscribe_oneclick(self, group_id, token, email):
        """ Unsubscribe a given user from a given group. One-click unsubscribe
        allow mail user agent to propose a one click button to the user to
        unsubscribe as defined in rfc8058. Only POST method is allowed preventing
        the risk that anti-spam trigger unwanted unsubscribe (scenario explained
        in the same rfc).

        :param int group_id: group ID from which user wants to unsubscribe;
        :param str token: optional access token ensuring security;
        :param email: email to unsubscribe;
        """
        group_sudo = request.env['mail.group'].sudo().browse(group_id).exists()
        # new route parameters
        if group_sudo and token and email:
            correct_token = group_sudo._generate_email_access_token(email)
            if not consteq(correct_token, token):
                raise werkzeug.exceptions.NotFound()
            group_sudo._leave_group(email)
        else:
            raise werkzeug.exceptions.NotFound()
        return Response(status=200)

    @http.route('/group/subscribe', type='json', auth='public', website=True)
    def group_subscribe(self, group_id=0, email=None, token=None, **kw):
        """Subscribe the current logged user or the given email address to the mailing list.

        If the user is logged, the action is automatically done.

        But if the user is not logged (public user) an email will be send with a token
        to confirm the action.

        :param group_id: Id of the group
        :param email: Email to add in the member list
        :param token: An access token to bypass the <mail.group> access rule
        :return:
            'added'
                if the member was added in the mailing list
            'email_sent'
                if we send a confirmation email
            'is_already_member'
                if we try to subscribe but we are already member
        """
        group_sudo, is_member, partner_id = self._group_subscription_get_group(group_id, email, token)

        if is_member:
            return 'is_already_member'

        if not request.env.user._is_public():
            # For logged user, automatically join / leave without sending a confirmation email
            group_sudo._join_group(request.env.user.email, partner_id)
            return 'added'

        # For non-logged user, send an email with a token to confirm the action
        group_sudo._send_subscribe_confirmation_email(email)
        return 'email_sent'

    @http.route('/group/unsubscribe', type='json', auth='public', website=True)
    def group_unsubscribe(self, group_id=0, email=None, token=None, **kw):
        """Unsubscribe the current logged user or the given email address to the mailing list.

        If the user is logged, the action is automatically done.

        But if the user is not logged (public user) an email will be send with a token
        to confirm the action.

        :param group_id: Id of the group
        :param email: Email to add in the member list
        :param token: An access token to bypass the <mail.group> access rule
        :return:
            'removed'
                if the member was removed from the mailing list
            'email_sent'
                if we send a confirmation email
            'is_not_member'
                if we try to unsubscribe but we are not member
        """
        group_sudo, is_member, partner_id = self._group_subscription_get_group(group_id, email, token)

        if not is_member:
            return 'is_not_member'

        if not request.env.user._is_public():
            # For logged user, automatically join / leave without sending a confirmation email
            group_sudo._leave_group(request.env.user.email, partner_id)
            return 'removed'

        # For non-logged user, send an email with a token to confirm the action
        group_sudo._send_unsubscribe_confirmation_email(email)
        return 'email_sent'

    def _group_subscription_get_group(self, group_id, email, token):
        """Check the given token and return,

        :return:
            - The group sudo-ed
            - True if the email is member of the group
            - The partner of the current user
        :raise NotFound: if the given token is not valid
        """
        group = request.env['mail.group'].browse(int(group_id)).exists()
        if not group:
            raise werkzeug.exceptions.NotFound()

        # SUDO to have access to field of the many2one
        group_sudo = group.sudo()

        if token and token != group_sudo._generate_group_access_token():
            raise werkzeug.exceptions.NotFound()

        elif not token:
            try:
                # Check that the current user has access to the group
                group.check_access_rights('read')
                group.check_access_rule('read')
            except AccessError:
                raise werkzeug.exceptions.NotFound()

        partner_id = None
        if not request.env.user._is_public():
            partner_id = request.env.user.partner_id.id

        is_member = bool(group_sudo._find_member(email, partner_id))

        return group_sudo, is_member, partner_id

    @http.route('/group/subscribe-confirm', type='http', auth='public', website=True)
    def group_subscribe_confirm(self, group_id, email, token, **kw):
        """Confirm the subscribe / unsubscribe action which was sent by email."""
        group = self._group_subscription_confirm_get_group(group_id, email, token, 'subscribe')
        if not group:
            return request.render('mail_group.invalid_token_subscription')

        partners = request.env['mail.thread'].sudo()._mail_find_partner_from_emails([email])
        partner_id = partners[0].id if partners else None
        group._join_group(email, partner_id)

        return request.render('mail_group.confirmation_subscription', {
            'group': group,
            'email': email,
            'subscribing': True,
        })

    @http.route('/group/unsubscribe-confirm', type='http', auth='public', website=True)
    def group_unsubscribe_confirm(self, group_id, email, token, **kw):
        """Confirm the subscribe / unsubscribe action which was sent by email."""
        group = self._group_subscription_confirm_get_group(group_id, email, token, 'unsubscribe')
        if not group:
            return request.render('mail_group.invalid_token_subscription')

        group._leave_group(email, all_members=True)

        return request.render('mail_group.confirmation_subscription', {
            'group': group,
            'email': email,
            'subscribing': False,
        })

    def _group_subscription_confirm_get_group(self, group_id, email, token, action):
        """Retrieve the group and check the token use to perform the given action."""
        if not group_id or not email or not token:
            return False
        # Here we can SUDO because the token will be checked
        group = request.env['mail.group'].browse(int(group_id)).exists().sudo()
        if not group:
            raise werkzeug.exceptions.NotFound()

        excepted_token = group._generate_action_token(email, action)
        return group if token == excepted_token else False

    def _generate_attachments_access_token(self, messages):
        for message in messages:
            if message.attachment_ids:
                message.attachment_ids.generate_access_token()
            self._generate_attachments_access_token(message.group_message_child_ids)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\ir_cron_data.xml

```xml
<odoo>
    <data>
        <record id="ir_cron_mail_notify_group_moderators" model="ir.cron">
            <field name="name">Mail List: Notify group moderators</field>
            <field name="model_id" ref="model_mail_group"/>
            <field name="state">code</field>
            <field name="code">model._cron_notify_moderators()</field>
            <field name="user_id" ref="base.user_root"/>
            <field name="interval_number">1</field>
            <field name="interval_type">days</field>
            <field name="numbercall">-1</field>
            <field name="doall" eval="False"/>
            <field name="priority">1000</field>
        </record>
    </data>
</odoo>

```

## File: data\mail_group_demo.xml

```xml
<odoo>
    <data>
        <!-- Group 1 -->
        <record id="mail_group_1" model="mail.group">
            <field name="name">My Company News</field>
            <field name="alias_name">newsletter</field>
            <field name="description">Receive news about "My Company"</field>
            <field name="moderation" eval="True"/>
            <field name="moderator_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="access_mode">groups</field>
            <field name="access_group_id" ref="base.group_user"/>
        </record>
        <!-- Members of group 1 -->
        <record id="mail_group_member_1" model="mail.group.member">
            <field name="partner_id" ref="base.partner_admin"/>
            <field name="email">admin@yourcompany.example.com</field>
            <field name="mail_group_id" ref="mail_group_1"/>
        </record>
        <record id="mail_group_member_2" model="mail.group.member">
            <field name="partner_id" ref="base.partner_demo"/>
            <field name="email">mark.brown23@example.com</field>
            <field name="mail_group_id" ref="mail_group_1"/>
        </record>
        <!-- Attachment of messages -->
        <record id="ir_attachment_mail_group_message_1" model="ir.attachment">
            <field name="datas">U3VwZXIgc2VjcmV0IGF0dGFjaG1lbnQ=</field>
            <field name="name">attachment.txt</field>
        </record>
        <record id="ir_attachment_mail_group_message_2" model="ir.attachment">
            <field name="datas">QnV0IEphdmFzY3JpcHQgc3Vja3M=</field>
            <field name="name">attachment.txt</field>
        </record>
        <!-- Messages of group 1 -->
        <record id="mail_group_message_1" model="mail.group.message">
            <field name="subject">Important Announce</field>
            <field name="body">This weekend, you should all come to our barbecue!</field>
            <field name="email_from">mark.brown23@example.com</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="moderation_status">accepted</field>
            <field name="group_message_parent_id" eval="False"/>
            <field name="mail_group_id" ref="mail_group_1"/>
            <field name="attachment_ids" eval="[(4, ref('ir_attachment_mail_group_message_1'))]"/>
        </record>
        <record id="mail_group_message_1_1" model="mail.group.message">
            <field name="subject">Re: Important Announce</field>
            <field name="body">Will there be vegetarian food?</field>
            <field name="email_from">admin@yourcompany.example.com</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="moderation_status">accepted</field>
            <field name="group_message_parent_id" ref="mail_group_message_1"/>
            <field name="mail_group_id" ref="mail_group_1"/>
        </record>
        <record id="mail_group_message_1_1_1" model="mail.group.message">
            <field name="subject">Re: Important Announce</field>
            <field name="body">Yes of course, and for those who are allergic bring your own food</field>
            <field name="email_from">mark.brown23@example.com</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="moderation_status">accepted</field>
            <field name="group_message_parent_id" ref="mail_group_message_1_1"/>
            <field name="mail_group_id" ref="mail_group_1"/>
        </record>
        <record id="mail_group_message_1_2" model="mail.group.message">
            <field name="subject">Re: Important Announce</field>
            <field name="body">Can I come with my dog?</field>
            <field name="email_from">joel.willis63@example.com</field>
            <field name="moderation_status">accepted</field>
            <field name="group_message_parent_id" ref="mail_group_message_1"/>
            <field name="mail_group_id" ref="mail_group_1"/>
        </record>
        <record id="mail_group_message_1_2_1" model="mail.group.message">
            <field name="subject">Re: Important Announce</field>
            <field name="body">No animals please.</field>
            <field name="email_from">admin@yourcompany.example.com</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="moderation_status">accepted</field>
            <field name="group_message_parent_id" ref="mail_group_message_1_2"/>
            <field name="mail_group_id" ref="mail_group_1"/>
        </record>
        <record id="mail_group_message_2" model="mail.group.message">
            <field name="subject">Recruitment</field>
            <field name="body">We are pleased to announce that we have hired 10 new people this month.</field>
            <field name="email_from">admin@yourcompany.example.com</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="moderation_status">accepted</field>
            <field name="group_message_parent_id" eval="False"/>
            <field name="mail_group_id" ref="mail_group_1"/>
        </record>
        <record id="mail_group_message_3" model="mail.group.message">
            <field name="subject">Relocation</field>
            <field name="body">We are moving our office to Brussels. Take back all your remaining stuff.</field>
            <field name="email_from">admin@yourcompany.example.com</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="moderation_status">accepted</field>
            <field name="group_message_parent_id" eval="False"/>
            <field name="mail_group_id" ref="mail_group_1"/>
        </record>
        <record id="mail_group_message_3_1" model="mail.group.message">
            <field name="subject">Re: Relocation</field>
            <field name="body">Is there a swimming pool in this new office?</field>
            <field name="email_from">joel.willis63@example.com</field>
            <field name="moderation_status">accepted</field>
            <field name="group_message_parent_id" ref="mail_group_message_3"/>
            <field name="mail_group_id" ref="mail_group_1"/>
        </record>
        <record id="mail_group_message_3_1_1" model="mail.group.message">
            <field name="subject">Re: Relocation</field>
            <field name="body">Of course!</field>
            <field name="email_from">admin@yourcompany.example.com</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="moderation_status">accepted</field>
            <field name="group_message_parent_id" ref="mail_group_message_3_1"/>
            <field name="mail_group_id" ref="mail_group_1"/>
        </record>
        <record id="mail_group_message_3_2" model="mail.group.message">
            <field name="subject">Re: Relocation</field>
            <field name="body">What is the deadline for taking back my stuff?</field>
            <field name="email_from">mark.brown23@example.com</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="moderation_status">accepted</field>
            <field name="group_message_parent_id" ref="mail_group_message_3"/>
            <field name="mail_group_id" ref="mail_group_1"/>
        </record>
        <record id="mail_group_message_3_2_1" model="mail.group.message">
            <field name="subject">Re: Relocation</field>
            <field name="body">In 4 weeks.</field>
            <field name="email_from">admin@yourcompany.example.com</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="moderation_status">pending_moderation</field>
            <field name="group_message_parent_id" ref="mail_group_message_3_2"/>
            <field name="mail_group_id" ref="mail_group_1"/>
        </record>
        <record id="mail_group_message_4" model="mail.group.message">
            <field name="subject">I really like CSS &amp; HTML</field>
            <field name="body">
                &lt;p style=&quot;margin:0px; font-size:13px;&quot;&gt;Hi,&lt;/p&gt;&lt;br&gt; &lt;p style=&quot;margin:0px; font-size:13px;&quot;&gt; We &lt;u style=&quot;font-size:14px&quot;&gt;really&lt;/u&gt; like &lt;font style=&quot;color:rgb(57, 123, 33); font-weight:bolder&quot;&gt;colors&lt;/font&gt; and &lt;span style=&quot;font-weight:bolder&quot;&gt;&lt;font style=&quot;color:rgb(255, 0, 0)&quot;&gt;CSS&lt;/font&gt;&lt;/span&gt;. &lt;/p&gt; &lt;p style=&quot;margin:0px; font-size:13px;&quot;&gt;Do you know you can add &lt;/p&gt; &lt;a href=&quot;https://example.com&quot; target=&quot;_blank&quot; class=&quot;btn btn-primary flat btn-sm&quot;&gt;link in email&lt;/a&gt; ? &lt;ol&gt; &lt;li&gt;Also image&lt;/li&gt; &lt;li&gt;List&lt;/li&gt; &lt;li&gt;Any HTML code in the end...&lt;/li&gt; &lt;/ol&gt;

            </field>
            <field name="email_from">admin@yourcompany.example.com</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="moderation_status">pending_moderation</field>
            <field name="group_message_parent_id" eval="False"/>
            <field name="mail_group_id" ref="mail_group_1"/>
            <field name="attachment_ids" eval="[(4, ref('ir_attachment_mail_group_message_2'))]"/>
        </record>
        <!-- Group 2 -->
        <record id="mail_group_2" model="mail.group">
            <field name="name">Public Mailing List</field>
            <field name="alias_name">public_group</field>
            <field name="description">Get the patch notes of our amazing product.</field>
            <field name="moderation" eval="True"/>
            <field name="moderator_ids" eval="[(4, ref('base.user_admin'))]"/>
        </record>
        <!-- Message of Group 2 -->
        <record id="mail_group_message_5" model="mail.group.message">
            <field name="subject">Best patch ever</field>
            <field name="body">In this patch, we have cleaned the CSS!</field>
            <field name="email_from">mark.brown23@example.com</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="moderation_status">accepted</field>
            <field name="group_message_parent_id" eval="False"/>
            <field name="mail_group_id" ref="mail_group_2"/>
        </record>
    </data>
</odoo>

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="mail_group_footer" name="Mail Group: Footer">
            <div id="o_mg_message_footer">
                <p>_______________________________________________</p>
                <p>Mailing-List: <t t-esc="group_url"/></p>
                <p>Post to: <t t-esc="mailto"/></p>
                <p>Unsubscribe: <a t-att-href="unsub_url" t-esc="unsub_label"/></p>
            </div>
        </template>

        <template id="mail_group_notify_moderation">
            <div style="max-width: 600px">
                <p>Hello <t t-esc="moderator.partner_id.name"/>,</p>
                <p>You have messages to moderate, please go for the proceedings.</p>
                <p><a t-attf-href="/web#action=mail_group.mail_group_action&amp;id={{group.id}}&amp;view_type=form" class="o_default_snippet_text">Moderate Messages</a></p>
                <p>Thank you!</p>
            </div>
        </template>
    </data>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mail_template_guidelines" model="mail.template">
        <field name="name">Mail Group: Send Guidelines</field>
        <field name="model_id" ref="mail_group.model_mail_group_member"/>
        <field name="subject">Guidelines of group {{ object.mail_group_id.name }}</field>
        <field name="email_to">{{ object.email }}</field>
        <field name="description">Sent to people who subscribed to a mailing group with group guidelines</field>
        <field name="body_html" type="html">
            <div>
                <p>Hello <t t-out="object.partner_id.name or ''"></t>,</p>
                <p>Please find below the guidelines of the <t t-out="object.mail_group_id.name"></t> mailing list.</p>
                <p><t t-out="object.mail_group_id.moderation_guidelines_msg or ''"></t></p>
            </div>
        </field>
        <field name="lang">{{ object.partner_id.lang }}</field>
        <field name="auto_delete" eval="True"/>
    </record>

    <!-- Confirm subscription email -->
    <record id="mail_template_list_subscribe" model="mail.template">
        <field name="name">Mail Group: Mailing List Subscription</field>
        <field name="model_id" ref="mail_group.model_mail_group"/>
        <field name="subject">Confirm subscription to {{ object.name }}</field>
        <field name="description">Subscription confirmation to a mailing group</field>
        <field name="body_html" type="html">
            <div style="margin: 0px; padding: 0px;">
                Hello,<br/><br/>
                You have requested to be subscribed to the mailing list <strong t-out="object.name or ''"></strong>.
                <br/><br/>
                To confirm, please visit the following link: <strong t-if="ctx.get('token_url')"><a t-att-href="ctx['token_url']"><t t-out="ctx['token_url'] or ''"></t></a></strong>
                <br/><br/>
                If this was a mistake or you did not requested this action, please ignore this message.
            </div>
        </field>
        <field name="auto_delete" eval="True"/>
    </record>

    <!-- Confirm unsubscription email -->
    <record id="mail_template_list_unsubscribe" model="mail.template">
        <field name="name">Mail Group: Mailing List Unsubscription</field>
        <field name="model_id" ref="mail_group.model_mail_group"/>
        <field name="subject">Confirm unsubscription to {{ object.name }}</field>
        <field name="description">Sent to people who unsubscribed from a mailing group</field>
        <field name="body_html" type="html">
            <div style="margin: 0px; padding: 0px;">
                Hello,<br/><br/>
                You have requested to be unsubscribed to the mailing list <strong t-out="object.name or ''"></strong>.
                <br/><br/>
                To confirm, please visit the following link: <strong t-if="ctx.get('token_url')"><a t-att-href="ctx['token_url']"><t t-out="ctx['token_url'] or ''"></t></a></strong>.
                <br/><br/>
                If this was a mistake or you did not requested this action, please ignore this message.
            </div>
        </field>
        <field name="auto_delete" eval="True"/>
    </record>
</odoo>

```

## File: data\res_groups.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="group_mail_group_manager" model="res.groups">
        <field name="name">Mail Group Administrator</field>
        <field name="category_id" ref="base.module_category_usability"/>
    </record>
    <record id="base.group_system" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('mail_group.group_mail_group_manager'))]"/>
    </record>
</odoo>

```

## File: models\mail_group.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging
import lxml

from ast import literal_eval
from datetime import datetime
from dateutil import relativedelta
from markupsafe import Markup
from werkzeug import urls

from odoo import _, api, fields, models, tools
from odoo.addons.http_routing.models.ir_http import slug
from odoo.addons.mail.tools.alias_error import AliasError
from odoo.exceptions import ValidationError, UserError
from odoo.osv import expression
from odoo.tools import email_normalize, hmac, generate_tracking_message_id

_logger = logging.getLogger(__name__)

# TODO remove me master
GROUP_SEND_BATCH_SIZE = 500


class MailGroup(models.Model):
    """This model represents a mailing list.

    Users send emails to an alias to create new group messages or reply to existing
    group messages. Moderation can be activated on groups. In that case email have to
    be validated or rejected.
    """
    _name = 'mail.group'
    _description = 'Mail Group'
    # TDE CHECK: use blaclist mixin
    _inherit = ['mail.alias.mixin']
    _order = 'create_date DESC, id DESC'

    @api.model
    def default_get(self, fields):
        res = super().default_get(fields)
        if 'alias_contact' in fields and not res.get('alias_contact'):
            res['alias_contact'] = 'everyone' if res.get('access_mode') == 'public' else 'followers'
        return res

    active = fields.Boolean('Active', default=True)
    name = fields.Char('Name', required=True, translate=True)
    description = fields.Text('Description')
    image_128 = fields.Image('Image', max_width=128, max_height=128)
    # Messages
    mail_group_message_ids = fields.One2many('mail.group.message', 'mail_group_id', string='Pending Messages')
    mail_group_message_last_month_count = fields.Integer('Messages Per Month', compute='_compute_mail_group_message_last_month_count')
    mail_group_message_count = fields.Integer('Messages Count', help='Number of message in this group', compute='_compute_mail_group_message_count')
    mail_group_message_moderation_count = fields.Integer('Pending Messages Count', help='Messages that need an action', compute='_compute_mail_group_message_moderation_count')
    # Members
    is_member = fields.Boolean('Is Member', compute='_compute_is_member')
    member_ids = fields.One2many('mail.group.member', 'mail_group_id', string='Members')
    member_partner_ids = fields.Many2many('res.partner', string='Partners Member', compute='_compute_member_partner_ids', search='_search_member_partner_ids')
    member_count = fields.Integer('Members Count', compute='_compute_member_count')
    # Moderation
    is_moderator = fields.Boolean(string='Moderator', help='Current user is a moderator of the group', compute='_compute_is_moderator')
    moderation = fields.Boolean(string='Moderate this group')
    moderation_rule_count = fields.Integer(string='Moderated emails count', compute='_compute_moderation_rule_count')
    moderation_rule_ids = fields.One2many('mail.group.moderation', 'mail_group_id', string='Moderated Emails')
    moderator_ids = fields.Many2many('res.users', 'mail_group_moderator_rel', string='Moderators',
                                     domain=lambda self: [('groups_id', 'in', self.env.ref('base.group_user').id)])
    moderation_notify = fields.Boolean(
        string='Automatic notification',
        help='People receive an automatic notification about their message being waiting for moderation.')
    moderation_notify_msg = fields.Html(string='Notification message')
    moderation_guidelines = fields.Boolean(
        string='Send guidelines to new subscribers',
        help='Newcomers on this moderated group will automatically receive the guidelines.')
    moderation_guidelines_msg = fields.Html(string='Guidelines')
    # ACLs
    access_mode = fields.Selection([
        ('public', 'Everyone'),
        ('members', 'Members only'),
        ('groups', 'Selected group of users'),
        ], string='Privacy', required=True, default='public')
    access_group_id = fields.Many2one('res.groups', string='Authorized Group',
                                      default=lambda self: self.env.ref('base.group_user'))
    # UI
    can_manage_group = fields.Boolean('Can Manage', help='Can manage the members', compute='_compute_can_manage_group')

    @api.depends('mail_group_message_ids.create_date', 'mail_group_message_ids.moderation_status')
    def _compute_mail_group_message_last_month_count(self):
        month_date = datetime.today() - relativedelta.relativedelta(months=1)
        messages_data = self.env['mail.group.message']._read_group([
            ('mail_group_id', 'in', self.ids),
            ('create_date', '>=', fields.Datetime.to_string(month_date)),
            ('moderation_status', '=', 'accepted'),
        ], ['mail_group_id'], ['__count'])

        # { mail_discusison_id: number_of_mail_group_message_last_month_count }
        messages_data = {
            mail_group.id: count
            for mail_group, count in messages_data
        }

        for group in self:
            group.mail_group_message_last_month_count = messages_data.get(group.id, 0)

    @api.depends('mail_group_message_ids')
    def _compute_mail_group_message_count(self):
        if not self:
            self.mail_group_message_count = 0
            return

        results = self.env['mail.group.message']._read_group(
            [('mail_group_id', 'in', self.ids)],
            ['mail_group_id'],
            ['__count'],
        )
        result_per_group = {
            mail_group.id: count
            for mail_group, count in results
        }
        for group in self:
            group.mail_group_message_count = result_per_group.get(group.id, 0)

    @api.depends('mail_group_message_ids.moderation_status')
    def _compute_mail_group_message_moderation_count(self):
        results = self.env['mail.group.message']._read_group(
            [('mail_group_id', 'in', self.ids), ('moderation_status', '=', 'pending_moderation')],
            ['mail_group_id'],
            ['__count'],
        )
        result_per_group = {
            mail_group.id: count
            for mail_group, count in results
        }

        for group in self:
            group.mail_group_message_moderation_count = result_per_group.get(group.id, 0)

    @api.depends('member_ids')
    def _compute_member_count(self):
        for group in self:
            group.member_count = len(group.member_ids)

    @api.depends_context('uid')
    def _compute_is_member(self):
        if not self or self.env.user._is_public():
            self.is_member = False
            return

        # SUDO to bypass the ACL rules
        members = self.env['mail.group.member'].sudo().search([
            ('partner_id', '=', self.env.user.partner_id.id),
            ('mail_group_id', 'in', self.ids),
        ])
        is_member = {member.mail_group_id.id: True for member in members}

        for group in self:
            group.is_member = is_member.get(group.id, False)

    @api.depends('member_ids')
    def _compute_member_partner_ids(self):
        for group in self:
            group.member_partner_ids = group.member_ids.partner_id

    def _search_member_partner_ids(self, operator, operand):
        return [(
            'member_ids',
            'in',
            self.env['mail.group.member'].sudo()._search([
                ('partner_id', operator, operand)
            ])
        )]

    @api.depends('moderator_ids')
    @api.depends_context('uid')
    def _compute_is_moderator(self):
        for group in self:
            group.is_moderator = self.env.user.id in group.moderator_ids.ids

    @api.depends('moderation_rule_ids')
    def _compute_moderation_rule_count(self):
        for group in self:
            group.moderation_rule_count = len(group.moderation_rule_ids)

    @api.depends('is_moderator')
    @api.depends_context('uid')
    def _compute_can_manage_group(self):
        is_admin = self.env.user.has_group('mail_group.group_mail_group_manager') or self.env.su
        for group in self:
            group.can_manage_group = is_admin or group.is_moderator

    @api.onchange('access_mode')
    def _onchange_access_mode(self):
        if self.access_mode == 'public':
            self.alias_contact = 'everyone'
        else:
            self.alias_contact = 'followers'

    @api.onchange('moderation')
    def _onchange_moderation(self):
        if self.moderation and self.env.user not in self.moderator_ids:
            self.moderator_ids |= self.env.user

    # CONSTRAINTS

    @api.constrains('moderator_ids')
    def _check_moderator_email(self):
        if any(not moderator.email for group in self for moderator in group.moderator_ids):
            raise ValidationError(_('Moderators must have an email address.'))

    @api.constrains('moderation_notify', 'moderation_notify_msg')
    def _check_moderation_notify(self):
        if any(group.moderation_notify and not group.moderation_notify_msg for group in self):
            raise ValidationError(_('The notification message is missing.'))

    @api.constrains('moderation_guidelines', 'moderation_guidelines_msg')
    def _check_moderation_guidelines(self):
        if any(group.moderation_guidelines and not group.moderation_guidelines_msg for group in self):
            raise ValidationError(_('The guidelines description is missing.'))

    @api.constrains('moderator_ids', 'moderation')
    def _check_moderator_existence(self):
        if any(not group.moderator_ids for group in self if group.moderation):
            raise ValidationError(_('Moderated group must have moderators.'))

    @api.constrains('access_mode', 'access_group_id')
    def _check_access_mode(self):
        if any(group.access_mode == 'groups' and not group.access_group_id for group in self):
            raise ValidationError(_('The "Authorized Group" is missing.'))

    def _alias_get_creation_values(self):
        """Return the default values for the automatically created alias."""
        values = super(MailGroup, self)._alias_get_creation_values()
        values['alias_model_id'] = self.env['ir.model']._get('mail.group').id
        values['alias_force_thread_id'] = self.id
        values['alias_defaults'] = literal_eval(self.alias_defaults or '{}')
        return values

    # ------------------------------------------------------------
    # MAILING
    # ------------------------------------------------------------

    def _alias_get_error(self, message, message_dict, alias):
        """ Checks for access errors related to sending email to the mailing list.
        Returns None if the mailing list is public or if no error cases are detected. """
        self.ensure_one()

        # Error Case: Selected group of users, but no user found for that email
        email = email_normalize(message_dict.get('email_from', ''))
        email_has_access = self.search_count([('id', '=', self.id), ('access_group_id.users.email_normalized', '=', email)])
        if self.access_mode == 'groups' and not email_has_access:
            return AliasError('error_mail_group_members_restricted',
                                  _('Only selected groups of users can send email to the mailing list.'))

        # Error Case: Access for members, but no member found for that email
        elif self.access_mode == 'members' and not self._find_member(message_dict.get('email_from')):
            return AliasError('error_mail_group_members_restricted',
                                  _('Only members can send email to the mailing list.'))

        return None

    @api.model
    def message_new(self, msg_dict, custom_values=None):
        """Add the method to make the mail gateway flow work with this model."""
        return

    @api.model
    def message_update(self, msg_dict, update_vals=None):
        """Add the method to make the mail gateway flow work with this model."""
        return

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, body='', subject=None, email_from=None, author_id=None, **kwargs):
        """ Custom posting process. This model does not inherit from ``mail.thread``
        but uses the mail gateway so few methods should be defined.

        This custom posting process works as follow

          * create a ``mail.message`` based on incoming email;
          * create linked ``mail.group.message`` that encapsulates message in a
            format used in mail groups;
          * apply moderation rules;

        :return message: newly-created mail.message
        """
        self.ensure_one()
        # First create the <mail.message>
        Mailthread = self.env['mail.thread']
        values = dict((key, val) for key, val in kwargs.items() if key in self.env['mail.message']._fields)
        author_id, email_from = Mailthread._message_compute_author(author_id, email_from, raise_on_email=True)

        values.update({
            'author_id': author_id,
            # sanitize then make valid Markup, notably for '_process_attachments_for_post'
            'body': Markup(self._clean_email_body(body)),
            'email_from': email_from,
            'model': self._name,
            'partner_ids': [],
            'res_id': self.id,
            'subject': subject,
        })

        # Force the "reply-to" to make the mail group flow work
        values['reply_to'] = self.env['mail.message']._get_reply_to(values)

        # ensure message ID so that replies go to the right thread
        if not values.get('message_id'):
            values['message_id'] = generate_tracking_message_id('%s-mail.group' % self.id)

        values.update(Mailthread._process_attachments_for_post(
            kwargs.get('attachments') or [],
            kwargs.get('attachment_ids') or [],
            values
        ))

        mail_message = Mailthread._message_create([values])

        # Find the <mail.group.message> parent
        group_message_parent_id = False
        if mail_message.parent_id:
            group_message_parent = self.env['mail.group.message'].search(
                [('mail_message_id', '=', mail_message.parent_id.id)])
            group_message_parent_id = group_message_parent.id if group_message_parent else False

        moderation_status = 'pending_moderation' if self.moderation else 'accepted'

        # Create the group message associated
        group_message = self.env['mail.group.message'].create({
            'mail_group_id': self.id,
            'mail_message_id': mail_message.id,
            'moderation_status': moderation_status,
            'group_message_parent_id': group_message_parent_id,
        })

        # Check the moderation rule to determine if we should accept or reject the email
        email_normalized = email_normalize(email_from)
        moderation_rule = self.env['mail.group.moderation'].search([
            ('mail_group_id', '=', self.id),
            ('email', '=', email_normalized),
        ], limit=1)

        if not self.moderation:
            self._notify_members(group_message)

        elif moderation_rule and moderation_rule.status == 'allow':
            group_message.action_moderate_accept()

        elif moderation_rule and moderation_rule.status == 'ban':
            group_message.action_moderate_reject()

        elif self.moderation_notify:
            self.env['mail.mail'].sudo().create({
                'author_id': self.env.user.partner_id.id,
                'auto_delete': True,
                'body_html': group_message.mail_group_id.moderation_notify_msg,
                'email_from': self.env.user.company_id.catchall_formatted or self.env.user.company_id.email_formatted,
                'email_to': email_from,
                'subject': 'Re: %s' % (subject or ''),
                'state': 'outgoing'
            })

        return mail_message

    def action_send_guidelines(self, members=None):
        """ Send guidelines to given members. """
        self.ensure_one()

        if not self.env.is_admin() and not self.is_moderator:
            raise UserError(_('Only an administrator or a moderator can send guidelines to group members.'))

        if not self.moderation_guidelines_msg:
            raise UserError(_('The guidelines description is empty.'))

        template = self.env.ref('mail_group.mail_template_guidelines', raise_if_not_found=False)
        if not template:
            raise UserError(_('Template "mail_group.mail_template_guidelines" was not found. No email has been sent. Please contact an administrator to fix this issue.'))

        banned_emails = self.env['mail.group.moderation'].sudo().search([
            ('status', '=', 'ban'),
            ('mail_group_id', '=', self.id),
        ]).mapped('email')

        if members is None:
            members = self.member_ids
        members = members.filtered(lambda member: member.email_normalized not in banned_emails)

        for member in members:
            company = member.partner_id.company_id or self.env.company
            template.send_mail(
                member.id,
                email_values={
                    'author_id': self.env.user.partner_id.id,
                    'email_from': company.email_formatted or company.catchall_formatted,
                    'reply_to': company.email_formatted or company.catchall_formatted,
                },
            )

        _logger.info('Send guidelines to %i members', len(members))

    def _notify_members(self, message):
        """Send the given message to all members of the mail group (except the author)."""
        self.ensure_one()

        if message.mail_group_id != self:
            raise UserError(_('The group of the message do not match.'))

        if not message.mail_message_id.reply_to:
            _logger.error('The alias or the catchall domain is missing, group might not work properly.')

        base_url = self.get_base_url()
        body = self.env['mail.render.mixin']._replace_local_links(message.body)

        # Email added in a dict to be sure to send only once the email to each address
        member_emails = {
            email_normalize(member.email): member.email
            for member in self.member_ids
        }

        batch_size = int(self.env['ir.config_parameter'].sudo().get_param('mail.session.batch.size', GROUP_SEND_BATCH_SIZE))
        for batch_email_member in tools.split_every(batch_size, member_emails.items()):
            mail_values = []
            for email_member_normalized, email_member in batch_email_member:
                if email_member_normalized == message.email_from_normalized:
                    # Do not send the email to their author
                    continue

                # SMTP headers related to the subscription
                email_url_encoded = urls.url_quote(email_member)
                unsubscribe_url = self._get_email_unsubscribe_url(email_member_normalized)

                headers = {
                    ** self._notify_by_email_get_headers(),
                    'List-Archive': f'<{base_url}/groups/{slug(self)}>',
                    'List-Subscribe': f'<{base_url}/groups?email={email_url_encoded}>',
                    'List-Unsubscribe': f'<{unsubscribe_url}>',
                    'List-Unsubscribe-Post': 'List-Unsubscribe=One-Click',
                    'Precedence': 'list',
                    'X-Auto-Response-Suppress': 'OOF',  # avoid out-of-office replies from MS Exchange
                }
                if self.alias_email:
                    headers.update({
                        'List-Id': f'<{self.alias_email}>',
                        'List-Post': f'<mailto:{self.alias_email}>',
                        'X-Forge-To': f'"{self.name}" <{self.alias_email}>',
                    })

                if message.mail_message_id.parent_id:
                    headers['In-Reply-To'] = message.mail_message_id.parent_id.message_id

                # Add the footer (member specific) in the body
                template_values = {
                    'mailto': f'{self.alias_email}',
                    'group_url': f'{base_url}/groups/{slug(self)}',
                    'unsub_label': f'{base_url}/groups?unsubscribe',
                    'unsub_url':  unsubscribe_url,
                }
                footer = self.env['ir.qweb']._render('mail_group.mail_group_footer', template_values, minimal_qcontext=True)
                member_body = tools.append_content_to_html(body, footer, plaintext=False)

                mail_values.append({
                    'auto_delete': True,
                    'attachment_ids': message.attachment_ids.ids,
                    'body_html': member_body,
                    'email_from': message.email_from,
                    'email_to': email_member,
                    'headers': json.dumps(headers),
                    'mail_message_id': message.mail_message_id.id,
                    'message_id': message.mail_message_id.message_id,
                    'model': 'mail.group',
                    'reply_to': message.mail_message_id.reply_to,
                    'res_id': self.id,
                    'subject': message.subject,
                })

            if mail_values:
                self.env['mail.mail'].sudo().create(mail_values)

    @api.model
    def _cron_notify_moderators(self):
        moderated_groups = self.env['mail.group'].search([('moderation', '=', True)])
        return moderated_groups._notify_moderators()

    def _notify_moderators(self):
        """Push a notification (Inbox / Email) to the moderators whose an action is waiting."""
        template = self.env.ref('mail_group.mail_group_notify_moderation', raise_if_not_found=False)
        if not template:
            _logger.warning('Template "mail_group.mail_group_notify_moderation" was not found. Cannot send reminder notifications.')
            return

        results = self.env['mail.group.message']._read_group(
            [('mail_group_id', 'in', self.ids), ('moderation_status', '=', 'pending_moderation')],
            ['mail_group_id'],
        )
        groups = self.browse([mail_group.id for [mail_group] in results])

        for group in groups:
            moderators_to_notify = group.moderator_ids
            MailThread = self.env['mail.thread']
            for moderator in moderators_to_notify:
                body = self.env['ir.qweb']._render('mail_group.mail_group_notify_moderation', {
                    'moderator': moderator,
                    'group': group,
                    }, minimal_qcontext=True)
                email_from = moderator.company_id.catchall_formatted or moderator.company_id.email_formatted
                MailThread.message_notify(
                    partner_ids=moderator.partner_id.ids,
                    subject=_('Messages are pending moderation'),
                    body=body,
                    email_from=email_from,
                    model='mail.group',
                    notify_author=True,
                    res_id=group.id,
                )

    @api.model
    def _clean_email_body(self, body_html):
        """When we receive an email, we want to clean it before storing it in the database."""
        tree = lxml.html.fromstring(body_html or '')
        # Remove the mailing footer
        xpath_footer = ".//div[contains(@id, 'o_mg_message_footer')]"
        for parent_footer in tree.xpath(xpath_footer + "/.."):
            for footer in parent_footer.xpath(xpath_footer):
                parent_footer.remove(footer)

        return lxml.etree.tostring(tree, encoding='utf-8').decode()

    # ------------------------------------------------------------
    # MEMBERSHIP
    # ------------------------------------------------------------

    def action_join(self):
        self.check_access_rights('read')
        self.check_access_rule('read')
        partner = self.env.user.partner_id
        self.sudo()._join_group(partner.email, partner.id)

        _logger.info('"%s" (#%s) joined mail.group "%s" (#%s)', partner.name, partner.id, self.name, self.id)

    def action_leave(self):
        self.check_access_rights('read')
        self.check_access_rule('read')
        partner = self.env.user.partner_id
        self.sudo()._leave_group(partner.email, partner.id)

        _logger.info('"%s" (#%s) leaved mail.group "%s" (#%s)', partner.name, partner.id, self.name, self.id)

    def _join_group(self, email, partner_id=None):
        self.ensure_one()

        if partner_id:
            partner = self.env['res.partner'].browse(partner_id).exists()
            if not partner:
                raise ValidationError(_('The partner can not be found.'))
            email = partner.email

        existing_member = self._find_member(email, partner_id)
        if existing_member:
            # Update the information of the partner to force the synchronization
            # If one the value is not up to date (e.g. if our email is subscribed
            # but our partner was not set)
            existing_member.write({
                'email': email,
                'partner_id': partner_id,
            })
            return

        member = self.env['mail.group.member'].create({
            'partner_id': partner_id,
            'email': email,
            'mail_group_id': self.id,
        })

        if self.moderation_guidelines:
            # Automatically send the guidelines to the new member
            self.action_send_guidelines(member)

    def _leave_group(self, email, partner_id=None, all_members=False):
        """Remove the given email / partner from the group.

        If the "all_members" parameter is set to True, remove all members with the given
        email address (multiple members might have the same email address).

        Otherwise, remove the most appropriate.
        """
        self.ensure_one()
        if all_members and not partner_id:
            self.env['mail.group.member'].search([
                ('mail_group_id', '=', self.id),
                ('email_normalized', '=', email_normalize(email)),
            ]).unlink()
        else:
            member = self._find_member(email, partner_id)
            if member:
                member.unlink()

    def _send_subscribe_confirmation_email(self, email):
        """Send an email to the given address to subscribe / unsubscribe to the mailing list."""
        self.ensure_one()
        confirm_action_url = self._generate_action_url(email, 'subscribe')

        template = self.env.ref('mail_group.mail_template_list_subscribe')
        template.with_context(token_url=confirm_action_url).send_mail(
            self.id,
            email_layout_xmlid='mail.mail_notification_light',
            email_values={
                'author_id': self.create_uid.partner_id.id,
                'auto_delete': True,
                'email_from': self.env.company.email_formatted,
                'email_to': email,
                'message_type': 'user_notification',
            },
            force_send=True,
        )
        _logger.info('Subscription email sent to %s.', email)

    def _send_unsubscribe_confirmation_email(self, email):
        """Send an email to the given address to subscribe / unsubscribe to the mailing list."""
        self.ensure_one()
        confirm_action_url = self._generate_action_url(email, 'unsubscribe')

        template = self.env.ref('mail_group.mail_template_list_unsubscribe')
        template.with_context(token_url=confirm_action_url).send_mail(
            self.id,
            email_layout_xmlid='mail.mail_notification_light',
            email_values={
                'author_id': self.create_uid.partner_id.id,
                'auto_delete': True,
                'email_from': self.env.company.email_formatted,
                'email_to': email,
                'message_type': 'user_notification',
            },
            force_send=True,
        )
        _logger.info('Unsubscription email sent to %s.', email)

    def _generate_action_url(self, email, action):
        """Generate the confirmation URL to subscribe / unsubscribe from the mailing list."""
        if action not in ['subscribe', 'unsubscribe']:
            raise ValueError(_('Invalid action for URL generation (%s)', action))
        self.ensure_one()

        confirm_action_url = '/group/%s-confirm?%s' % (
            action,
            urls.url_encode({
                'group_id': self.id,
                'email': email,
                'token': self._generate_action_token(email, action),
            })
        )
        base_url = self.get_base_url()
        confirm_action_url = urls.url_join(base_url, confirm_action_url)
        return confirm_action_url

    def _generate_action_token(self, email, action):
        """Generate an action token to be able to subscribe / unsubscribe from the mailing list."""
        if action not in ['subscribe', 'unsubscribe']:
            raise ValueError(_('Invalid action for URL generation (%s)', action))
        self.ensure_one()

        email_normalized = email_normalize(email)
        if not email_normalized:
            raise UserError(_('Email %s is invalid', email))

        data = (self.id, email_normalized, action)
        return hmac(self.env(su=True), 'mail_group-email-subscription', data)

    def _generate_email_access_token(self, email):
        """Generate an action token to be able to unsubscribe from the mailing
        list, while hashing the target email to avoid spoofind other emails.

        :param str email: email included in hash, should be normalized
        """
        return tools.hmac(self.env(su=True), 'mail_group-access-token-portal-email', (self.id, email))

    def _generate_group_access_token(self):
        """Generate an action token to be able to subscribe / unsubscribe from the mailing list."""
        self.ensure_one()
        return hmac(self.env(su=True), 'mail_group-access-token-portal', self.id)

    def _get_email_unsubscribe_url(self, email_to):
        params = urls.url_encode({
            'email': email_to,
            'token': self._generate_email_access_token(email_to),
        })
        return urls.url_join(
            self.get_base_url(),
            f'group/{self.id}/unsubscribe_oneclick?{params}'
        )

    def _find_member(self, email, partner_id=None):
        """Return the <mail.group.member> corresponding to the given email address."""
        self.ensure_one()

        result = self._find_members(email, partner_id)
        return result.get(self.id)

    def _find_members(self, email, partner_id):
        """Get all the members record corresponding to the email / partner_id.

        Can be called in batch and return a dictionary
            {'group_id': <mail.group.member>}

        Multiple members might have the same email address, but with different partner
        because there's no unique constraint on the email field of the <res.partner>
        model.

        When a partner is given for the search, return in priority
        - The member whose partner match the given partner
        - The member without partner but whose email match the given email

        When no partner is given for the search, return in priority
        - A member whose email match the given email and has no partner
        - A member whose email match the given email and has partner
        """
        order = 'partner_id ASC'
        if not email_normalize(email):
            # empty email should match nobody
            return {}

        domain = [('email_normalized', '=', email_normalize(email))]
        if partner_id:
            domain = expression.OR([
                expression.AND([
                    [('partner_id', '=', False)],
                    domain,
                ]),
                [('partner_id', '=', partner_id)],
            ])
            order = 'partner_id DESC'

        domain = expression.AND([domain, [('mail_group_id', 'in', self.ids)]])
        members_data = self.env['mail.group.member'].sudo().search(domain, order=order)
        return {
            member.mail_group_id.id: member
            for member in members_data
        }

```

## File: models\mail_group_member.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, fields, models
from odoo.tools import email_normalize

_logger = logging.getLogger(__name__)


class MailGroupMember(models.Model):
    """Models a group member that can be either an email address either a full partner."""
    _name = 'mail.group.member'
    _description = 'Mailing List Member'
    _rec_name = 'email'

    email = fields.Char(string='Email', compute='_compute_email', readonly=False, store=True)
    email_normalized = fields.Char(
        string='Normalized Email', compute='_compute_email_normalized',
        index=True, store=True)
    mail_group_id = fields.Many2one('mail.group', string='Group', required=True, ondelete='cascade')
    partner_id = fields.Many2one('res.partner', 'Partner', ondelete='cascade')

    _sql_constraints = [(
        'unique_partner',
        'UNIQUE(partner_id, mail_group_id)',
        'This partner is already subscribed to the group',
    )]

    @api.depends('partner_id.email')
    def _compute_email(self):
        for member in self:
            if member.partner_id:
                member.email = member.partner_id.email
            elif not member.email:
                member.email = False

    @api.depends('email')
    def _compute_email_normalized(self):
        for moderation in self:
            moderation.email_normalized = email_normalize(moderation.email)

```

## File: models\mail_group_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import _, api, fields, models
from odoo.exceptions import AccessError, UserError
from odoo.osv import expression
from odoo.tools import email_normalize, append_content_to_html, ustr

_logger = logging.getLogger(__name__)


class MailGroupMessage(models.Model):
    """Emails belonging to a discussion group.

    Those are build on <mail.message> with additional information related to specific
    features of <mail.group> like better parent / children management and moderation.
    """
    _name = 'mail.group.message'
    _description = 'Mailing List Message'
    _rec_name = 'subject'
    _order = 'create_date DESC'
    _primary_email = 'email_from'

    # <mail.message> fields, can not be done with inherits because it will impact
    # the performance of the <mail.message> model (different cache, so the ORM will need
    # to do one more SQL query to be able to update the <mail.group.message> cache)
    attachment_ids = fields.Many2many(related='mail_message_id.attachment_ids', readonly=False)
    author_id = fields.Many2one(related='mail_message_id.author_id', readonly=False)
    email_from = fields.Char(related='mail_message_id.email_from', readonly=False)
    email_from_normalized = fields.Char('Normalized From', compute='_compute_email_from_normalized', store=True)
    body = fields.Html(related='mail_message_id.body', readonly=False)
    subject = fields.Char(related='mail_message_id.subject', readonly=False)
    # Thread
    mail_group_id = fields.Many2one(
        'mail.group', string='Group',
        required=True, ondelete='cascade')
    mail_message_id = fields.Many2one('mail.message', 'Mail Message', required=True, ondelete='cascade', index=True, copy=False)
    # Parent and children
    group_message_parent_id = fields.Many2one(
        'mail.group.message', string='Parent', store=True)
    group_message_child_ids = fields.One2many('mail.group.message', 'group_message_parent_id', string='Children')
    # Moderation
    author_moderation = fields.Selection([('ban', 'Banned'), ('allow', 'Whitelisted')], string='Author Moderation Status',
                                         compute='_compute_author_moderation')
    is_group_moderated = fields.Boolean('Is Group Moderated', related='mail_group_id.moderation')
    moderation_status = fields.Selection(
        [('pending_moderation', 'Pending Moderation'),
         ('accepted', 'Accepted'),
         ('rejected', 'Rejected')],
        string='Status', index=True, copy=False,
        required=True, default='pending_moderation')
    moderator_id = fields.Many2one('res.users', string='Moderated By')
    create_date = fields.Datetime(string='Posted')

    @api.depends('email_from')
    def _compute_email_from_normalized(self):
        for message in self:
            message.email_from_normalized = email_normalize(message.email_from)

    @api.depends('email_from_normalized', 'mail_group_id')
    def _compute_author_moderation(self):
        moderations = self.env['mail.group.moderation'].search([
            ('mail_group_id', 'in', self.mail_group_id.ids),
        ])
        all_emails = set(self.mapped('email_from_normalized'))
        moderations = {
            (moderation.mail_group_id, moderation.email): moderation.status
            for moderation in moderations
            if moderation.email in all_emails
        }
        for message in self:
            message.author_moderation = moderations.get((message.mail_group_id, message.email_from_normalized), False)

    @api.constrains('mail_message_id')
    def _constrains_mail_message_id(self):
        for message in self:
            if message.mail_message_id.model != 'mail.group':
                raise AccessError(_(
                    'Group message can only be linked to mail group. Current model is %s.',
                    message.mail_message_id.model,
                ))
            if message.mail_message_id.res_id != message.mail_group_id.id:
                raise AccessError(_('The record of the message should be the group.'))

    @api.model_create_multi
    def create(self, values_list):
        for vals in values_list:
            if not vals.get('mail_message_id'):
                vals.update({
                    'res_id': vals.get('mail_group_id'),
                    'model': 'mail.group',
                })
                vals['mail_message_id'] = self.env['mail.message'].sudo().create({
                    field: vals.pop(field)
                    for field in self.env['mail.message']._fields
                    if field in vals
                    and field in self.env['mail.thread']._get_message_create_valid_field_names()
                }).id
        return super(MailGroupMessage, self).create(values_list)

    def copy(self, default=None):
        default = dict(default or {})
        default['mail_message_id'] = self.mail_message_id.copy().id
        return super(MailGroupMessage, self).copy(default)

    # --------------------------------------------------
    # MODERATION API
    # --------------------------------------------------

    def action_moderate_accept(self):
        """Accept the incoming email.

        Will send the incoming email to all members of the group.
        """
        self._assert_moderable()
        self.write({
            'moderation_status': 'accepted',
            'moderator_id': self.env.uid,
        })

        # Send the email to the members of the group
        for message in self:
            message.mail_group_id._notify_members(message)

    def action_moderate_reject_with_comment(self, reject_subject, reject_comment):
        self._assert_moderable()
        if reject_subject or reject_comment:
            self._moderate_send_reject_email(reject_subject, reject_comment)
        self.action_moderate_reject()

    def action_moderate_reject(self):
        self._assert_moderable()
        self.write({
            'moderation_status': 'rejected',
            'moderator_id': self.env.uid,
        })

    def action_moderate_allow(self):
        self._create_moderation_rule('allow')

        # Accept all emails of the same authors
        same_author = self._get_pending_same_author_same_group()
        same_author.action_moderate_accept()

    def action_moderate_ban(self):
        self._create_moderation_rule('ban')

        # Reject all emails of the same author
        same_author = self._get_pending_same_author_same_group()
        same_author.action_moderate_reject()

    def action_moderate_ban_with_comment(self, ban_subject, ban_comment):
        self._create_moderation_rule('ban')

        if ban_subject or ban_comment:
            self._moderate_send_reject_email(ban_subject, ban_comment)

        # Reject all emails of the same author
        same_author = self._get_pending_same_author_same_group()
        same_author.action_moderate_reject()

    def _get_pending_same_author_same_group(self):
        """Return the pending messages of the same authors in the same groups."""
        return self.search(
            expression.AND([
                expression.OR([
                    [
                        ('mail_group_id', '=', message.mail_group_id.id),
                        ('email_from_normalized', '=', message.email_from_normalized),
                    ] for message in self
                ]),
                [('moderation_status', '=', 'pending_moderation')],
            ])
        )

    def _create_moderation_rule(self, status):
        """Create a moderation rule <mail.group.moderation> with the given status.

        Update existing moderation rule for the same email address if found,
        otherwise create a new rule.
        """
        if status not in ('ban', 'allow'):
            raise ValueError(_('Wrong status (%s)', status))

        for message in self:
            if not email_normalize(message.email_from):
                raise UserError(_('The email "%s" is not valid.', message.email_from))

        existing_moderation = self.env['mail.group.moderation'].search(
            expression.OR([
                [
                    ('email', '=', email_normalize(message.email_from)),
                    ('mail_group_id', '=', message.mail_group_id.id)
                ] for message in self
            ])
        )
        existing_moderation.status = status

        # Add the value in a set to create only 1 moderation rule per (email_normalized, group)
        moderation_to_create = {
            (email_normalize(message.email_from), message.mail_group_id.id)
            for message in self
            if email_normalize(message.email_from) not in existing_moderation.mapped('email')
        }

        self.env['mail.group.moderation'].create([
            {
                'email': email,
                'mail_group_id': mail_group_id,
                'status': status,
            } for email, mail_group_id in moderation_to_create])

    def _assert_moderable(self):
        """Raise an error if one of the current message can not be moderated.

        A <mail.group.message> can only be moderated
        if it's moderation status is "pending_moderation".
        """
        non_moderable_messages = self.filtered_domain([
            ('moderation_status', '!=', 'pending_moderation'),
        ])
        if non_moderable_messages:
            if len(self) == 1:
                raise UserError(_('This message can not be moderated'))
            raise UserError(_(
                'Those messages can not be moderated: %s.',
                ', '.join(non_moderable_messages.mapped('subject')),
            ))

    def _moderate_send_reject_email(self, subject, comment):
        for message in self:
            if not message.email_from:
                continue

            body_html = append_content_to_html('<div>%s</div>' % ustr(comment), message.body, plaintext=False)
            body_html = self.env['mail.render.mixin']._replace_local_links(body_html)
            self.env['mail.mail'].sudo().create({
                'author_id': self.env.user.partner_id.id,
                'auto_delete': True,
                'body_html': body_html,
                'email_from': self.env.user.email_formatted or self.env.company.catchall_formatted,
                'email_to': message.email_from,
                'references': message.mail_message_id.message_id,
                'subject': subject,
                'state': 'outgoing',
            })

```

## File: models\mail_group_moderation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _
from odoo.exceptions import UserError
from odoo.tools import email_normalize


class MailGroupModeration(models.Model):
    """Represent the moderation rules for an email address in a group."""
    _name = 'mail.group.moderation'
    _description = 'Mailing List black/white list'

    email = fields.Char(string='Email', required=True)
    status = fields.Selection(
        [('allow', 'Always Allow'), ('ban', 'Permanent Ban')],
        string='Status', required=True, default='ban')
    mail_group_id = fields.Many2one('mail.group', string='Group', required=True, ondelete='cascade')

    _sql_constraints = [(
        'mail_group_email_uniq',
        'UNIQUE(mail_group_id, email)',
        'You can create only one rule for a given email address in a group.',
    )]

    @api.model_create_multi
    def create(self, vals_list):
        for values in vals_list:
            email_normalized = email_normalize(values.get('email'))
            if not email_normalized:
                raise UserError(_('Invalid email address %r', values.get('email')))
            values['email'] = email_normalized
        return super(MailGroupModeration, self).create(vals_list)

    def write(self, values):
        if 'email' in values:
            email_normalized = email_normalize(values['email'])
            if not email_normalized:
                raise UserError(_('Invalid email address %r', values.get('email')))
            values['email'] = email_normalized
        return super(MailGroupModeration, self).write(values)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mail_group
from . import mail_group_member
from . import mail_group_message
from . import mail_group_moderation

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_mail_group_all_public,access_mail_group_all,model_mail_group,base.group_public,1,0,0,0
access_mail_group_all_portal,access_mail_group_all,model_mail_group,base.group_portal,1,0,0,0
access_mail_group_user,access_mail_group_user,model_mail_group,base.group_user,1,1,1,1
access_mail_group_member,access_mail_group_member,model_mail_group_member,base.group_user,1,1,1,1
access_mail_group_message_all_public,access_mail_group_message_all,model_mail_group_message,base.group_public,1,0,0,0
access_mail_group_message_all_portal,access_mail_group_message_all,model_mail_group_message,base.group_portal,1,0,0,0
access_mail_group_message_user,access_mail_group_message_user,model_mail_group_message,base.group_user,1,1,1,1
access_mail_group_moderation,access_mail_group_moderation,model_mail_group_moderation,base.group_user,1,1,1,1
access_mail_group_message_reject,access_mail_group_message_reject,model_mail_group_message_reject,base.group_user,1,1,1,1

```

## File: security\mail_group_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="mail_group_rule_read_all" model="ir.rule">
        <field name="name">Mail Group: Access only public and joined groups</field>
        <field name="model_id" ref="model_mail_group"/>
        <field name="domain_force">[
            '|',
            '|',
            '|',
                ('moderator_ids', 'in', user.id),
                ('access_mode', '=', 'public'),
                '&amp;',
                    ('access_mode', '=', 'groups'),
                    ('access_group_id', 'in', [g.id for g in user.groups_id]),
                '&amp;',
                    ('access_mode', '=', 'members'),
                    ('member_partner_ids', 'in', [user.partner_id.id]),
            ]
        </field>
        <field name="groups" eval="[(4, ref('base.group_user')), (4, ref('base.group_portal')), (4, ref('base.group_public'))]"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>
    <record id="mail_group_rule_write_all" model="ir.rule">
        <field name="name">Mail Group: Moderator have write access on their group</field>
        <field name="model_id" ref="model_mail_group"/>
        <field name="domain_force">[('moderator_ids', 'in', user.id)]</field>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_read" eval="False"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_unlink" eval="True"/>
    </record>
    <record id="mail_group_rule_administrator" model="ir.rule">
        <field name="name">Mail Group: Administrator have access to all mail group</field>
        <field name="model_id" ref="model_mail_group"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('mail_group.group_mail_group_manager'))]"/>
        <field name="perm_create" eval="True"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_unlink" eval="True"/>
    </record>
    <record id="mail_group_message_rule_public" model="ir.rule">
        <field name="name">Mail Group Message: Only accepted message are accessible</field>
        <field name="model_id" ref="model_mail_group_message"/>
        <field name="domain_force">[
            '&amp;',
                ('moderation_status', '=', 'accepted'),
                '|',
                '|',
                '|',
                    ('mail_group_id.moderator_ids', 'in', user.id),
                    ('mail_group_id.access_mode', '=', 'public'),
                    '&amp;',
                        ('mail_group_id.access_mode', '=', 'groups'),
                        ('mail_group_id.access_group_id', 'in', [g.id for g in user.groups_id]),
                    '&amp;',
                        ('mail_group_id.access_mode', '=', 'members'),
                        ('mail_group_id.member_partner_ids', 'in', [user.partner_id.id]),
            ]
        </field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
        <field name="perm_create" eval="True"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_unlink" eval="True"/>
    </record>
    <record id="mail_group_message_rule_user" model="ir.rule">
        <field name="name">Mail Group Message: Non-accepted messages are accessible only by moderators</field>
        <field name="model_id" ref="model_mail_group_message"/>
        <field name="domain_force">[
                '&amp;',
                    '|',
                        ('moderation_status', '=', 'accepted'),
                        ('mail_group_id.moderator_ids', 'in', user.id),
                    '|',
                    '|',
                    '|',
                        ('mail_group_id.moderator_ids', 'in', user.id),
                        ('mail_group_id.access_mode', '=', 'public'),
                        '&amp;',
                            ('mail_group_id.access_mode', '=', 'groups'),
                            ('mail_group_id.access_group_id', 'in', [g.id for g in user.groups_id]),
                        '&amp;',
                            ('mail_group_id.access_mode', '=', 'members'),
                            ('mail_group_id.member_partner_ids', 'in', [user.partner_id.id]),
            ]
        </field>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
        <field name="perm_create" eval="True"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_unlink" eval="True"/>
    </record>
    <record id="mail_group_message_rule_administrator" model="ir.rule">
        <field name="name">Mail Group Message: Administrator have access to all messages</field>
        <field name="model_id" ref="model_mail_group_message"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('mail_group.group_mail_group_manager'))]"/>
        <field name="perm_create" eval="True"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_unlink" eval="True"/>
    </record>
    <record id="mail_group_member_rule_user" model="ir.rule">
        <field name="name">Mail Group Member: Members are accessible only by moderators</field>
        <field name="model_id" ref="model_mail_group_member"/>
        <field name="domain_force">[('mail_group_id.moderator_ids', 'in', user.id)]</field>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
        <field name="perm_create" eval="True"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_unlink" eval="True"/>
    </record>
    <record id="mail_group_member_rule_administrator" model="ir.rule">
        <field name="name">Mail Group Member: Administrator have access to all members</field>
        <field name="model_id" ref="model_mail_group_member"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('mail_group.group_mail_group_manager'))]"/>
        <field name="perm_create" eval="True"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_unlink" eval="True"/>
    </record>
    <record id="mail_group_moderation_rule_user" model="ir.rule">
        <field name="name">Mail Group Moderation: Moderation rules are accessible only by moderators</field>
        <field name="model_id" ref="model_mail_group_moderation"/>
        <field name="domain_force">[('mail_group_id.moderator_ids', 'in', user.id)]</field>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
        <field name="perm_create" eval="True"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_unlink" eval="True"/>
    </record>
    <record id="mail_group_moderation_rule_administrator" model="ir.rule">
        <field name="name">Mail Group Moderation: Administrator have access to all moderation rules</field>
        <field name="model_id" ref="model_mail_group_moderation"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('mail_group.group_mail_group_manager'))]"/>
        <field name="perm_create" eval="True"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_unlink" eval="True"/>
    </record>
</odoo>

```

## File: static\src\js\mail_group.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import { _t } from "@web/core/l10n/translation";

publicWidget.registry.MailGroup = publicWidget.Widget.extend({
    selector: '.o_mail_group',
    events: {
        'click .o_mg_subscribe_btn': '_onSubscribeBtnClick',
    },

    init() {
        this._super(...arguments);
        this.rpc = this.bindService("rpc");
    },

    /**
     * @override
     */
    start: function () {
        this.mailgroupId = this.$el.data('id');
        this.isMember = this.$el.data('isMember') || false;
        const searchParams = (new URL(document.location.href)).searchParams;
        this.token = searchParams.get('token');
        this.forceUnsubscribe = searchParams.has('unsubscribe');
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onSubscribeBtnClick: async function (ev) {
        ev.preventDefault();
        const $email = this.$el.find(".o_mg_subscribe_email");
        const email = $email.val();

        if (!email.match(/.+@.+/)) {
            this.$el.addClass('o_has_error').find('.form-control, .form-select').addClass('is-invalid');
            return false;
        }

        this.$el.removeClass('o_has_error').find('.form-control, .form-select').removeClass('is-invalid');

        const action = (this.isMember || this.forceUnsubscribe) ? 'unsubscribe' : 'subscribe';

        const response = await this.rpc('/group/' + action, {
            'group_id': this.mailgroupId,
            'email': email,
            'token': this.token,
        });

        this.$el.find('.o_mg_alert').remove();

        if (response === 'added') {
            this.isMember = true;
            this.$el.find('.o_mg_subscribe_btn').text(_t('Unsubscribe')).removeClass('btn-primary').addClass('btn-outline-primary');
        } else if (response === 'removed') {
            this.isMember = false;
            this.$el.find('.o_mg_subscribe_btn').text(_t('Subscribe')).removeClass('btn-outline-primary').addClass('btn-primary');
        } else if (response === 'email_sent') {
            // The confirmation email has been sent
            this.$el.html(
                $('<div class="o_mg_alert alert alert-success" role="alert"/>')
                .text(_t('An email with instructions has been sent.'))
            );
        } else if (response === 'is_already_member') {
            this.isMember = true;
            this.$el.find('.o_mg_subscribe_btn').text(_t('Unsubscribe')).removeClass('btn-primary').addClass('btn-outline-primary');
            this.$el.find('.o_mg_subscribe_form').before(
                $('<div class="o_mg_alert alert alert-warning" role="alert"/>')
                .text(_t('This email is already subscribed.'))
            );
        } else if (response === 'is_not_member') {
            if (!this.forceUnsubscribe) {
                this.isMember = false;
                this.$el.find('.o_mg_subscribe_btn').text(_t('Subscribe'));
            }
            this.$el.find('.o_mg_subscribe_form').before(
                $('<div class="o_mg_alert alert alert-warning" role="alert"/>')
                .text(_t('This email is not subscribed.'))
            );
        }

    },
});

export default publicWidget.registry.MailGroup;

```

## File: static\src\js\mail_group_message.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";

publicWidget.registry.MailGroupMessage = publicWidget.Widget.extend({
    selector: '.o_mg_message',
    events: {
        'click .o_mg_link_hide': '_onHideLinkClick',
        'click .o_mg_link_show': '_onShowLinkClick',
        'click button.o_mg_read_more': '_onReadMoreClick',
    },

    init() {
        this._super(...arguments);
        this.rpc = this.bindService("rpc");
    },

    /**
     * @override
     */
    start: function () {
        // By default hide the mention of the previous email for which we reply
        // And add a button "Read more" to show the mention of the parent email
        const body = this.$el.find('.card-body').first();
        const quoted = body.find('*[data-o-mail-quote]');
        const readMore = $('<button class="btn btn-light btn-sm ms-1"/>').text('. . .');
        quoted.first().before(readMore);
        readMore.on('click', () => {
            quoted.toggleClass('visible');
        });

        return this._super.apply(this, arguments);
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
        const $link = $(ev.currentTarget);
        const $container = $link.closest('.o_mg_link_parent');
        $container.find('.o_mg_link_hide').first().addClass('d-none');
        $container.find('.o_mg_link_show').first().removeClass('d-none');
        $container.find('.o_mg_link_content').first().removeClass('d-none');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onShowLinkClick: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        const $link = $(ev.currentTarget);
        const $container = $link.closest('.o_mg_link_parent');
        $container.find('.o_mg_link_hide').first().removeClass('d-none');
        $container.find('.o_mg_link_show').first().addClass('d-none');
        $container.find('.o_mg_link_content').first().addClass('d-none');
    },
    /**
     * @private
     * @param {Event} ev
     */
     _onReadMoreClick: function (ev) {
        const $link = $(ev.target);
        this.rpc($link.data('href'), {
            last_displayed_id: $link.data('last-displayed-id'),
        }).then(function (data) {
            if (!data) {
                return;
            }
            const $threadContainer = $link.parents('.o_mg_replies').first().find('ul.list-unstyled').first();
            if ($threadContainer) {
                const $data = $(data);
                const $lastMsg = $threadContainer.children('li.media').last();
                const $newMessages = $data.find('ul.list-unstyled').first().children('li.media');
                $newMessages.insertAfter($lastMsg);
                $data.find('.o_mg_read_more').parent().appendTo($threadContainer);
            }
            const $showMore = $link.parent();
            $showMore.remove();
        });
     },
});

```

## File: views\mail_group_member_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mail_group_member_view_tree" model="ir.ui.view">
        <field name="name">mail.group.member.view.tree</field>
        <field name="model">mail.group.member</field>
        <field name="arch" type="xml">
            <tree editable="top" sample="1">
                <field name="email"
                    force_save="1"
                    required="1"
                    readonly="partner_id"/>
                <field name="email_normalized" column_invisible="True" force_save="1"/>
                <field name="partner_id"/>
                <field name="mail_group_id"/>
            </tree>
        </field>
    </record>
    <record id="mail_group_member_view_search" model="ir.ui.view">
        <field name="name">mail.group.member.view.search</field>
        <field name="model">mail.group.member</field>
        <field name="arch" type="xml">
            <search string="Search Mail Group Member">
                <field name="email" required="1" />
                <field name="partner_id"/>
                <field name="mail_group_id"/>
            </search>
        </field>
    </record>
    <record id="mail_group_member_action" model="ir.actions.act_window">
        <field name="name">Members</field>
        <field name="res_model">mail.group.member</field>
        <field name="view_mode">tree</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">No Members in this list yet!</p>
            <p>Let people subscribe to your list online or manually add them here.</p>
        </field>
    </record>
</odoo>

```

## File: views\mail_group_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="mail_group_menu"
        name="Mail Groups"
        action="mail_group_action"
        parent="mail.mail_menu_technical"
        sequence="50"/>
    <menuitem id="mail_group_moderation_menu"
        name="Moderation Rules"
        action="mail_group_moderation_action"
        parent="mail.mail_menu_technical"
        sequence="51"/>
</odoo>

```

## File: views\mail_group_message_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mail_group_message_view_list" model="ir.ui.view">
        <field name="name">mail.group.message.view.list</field>
        <field name="model">mail.group.message</field>
        <field name="arch" type="xml">
            <tree sample="1">
                <field name="create_date"/>
                <field name="author_id"/>
                <field name="email_from"/>
                <field name="subject"/>
                <field name="mail_group_id"/>
                <field name="moderation_status"/>
                <field name="is_group_moderated" column_invisible="True"/>
                <button name="action_moderate_accept" string="Accept" title="Accept"
                    type="object" class="btn btn-primary"
                    invisible="moderation_status != 'pending_moderation' or not is_group_moderated"/>
                <button name="%(mail_group_message_reject_action)d"
                    string="Reject" title="Remove message with explanation"
                    type="action" class="btn btn-secondary"
                    invisible="moderation_status != 'pending_moderation' or not is_group_moderated"
                    context="{'default_mail_group_message_id': id, 'default_action': 'reject'}" />
                <button name="action_moderate_allow" string="Whitelist"
                    title="Add this email address to white list of people and accept all pending messages from the same author."
                    type="object" class="btn btn-secondary"
                    invisible="moderation_status != 'pending_moderation' or not is_group_moderated"/>
                <button name="%(mail_group_message_reject_action)d"
                    string="Ban" title="Ban this email address and reject all pending messages from the same author and send an email to the author"
                    type="action" class="btn btn-secondary"
                    invisible="moderation_status != 'pending_moderation' or not is_group_moderated"
                    context="{'default_mail_group_message_id': id, 'default_action': 'ban'}" />
                <button name="action_moderate_accept" string="Send" title="Send"
                    type="object" class="btn btn-primary"
                    invisible="moderation_status != 'pending_moderation' or is_group_moderated"/>
            </tree>
        </field>
    </record>
    <record id="mail_group_message_view_form" model="ir.ui.view">
        <field name="name">mail.group.message.view.form</field>
        <field name="model">mail.group.message</field>
        <field name="arch" type="xml">
            <form string="Group Message" class="o_mail_group_message_form">
                <header>
                    <button name="action_moderate_accept" string="Accept" title="Accept"
                        type="object" class="btn btn-primary"
                        invisible="moderation_status != 'pending_moderation' or not is_group_moderated"/>
                    <button name="%(mail_group_message_reject_action)d"
                        string="Reject" title="Remove message with explanation"
                        type="action" class="btn btn-secondary"
                        invisible="moderation_status != 'pending_moderation' or not is_group_moderated"
                        context="{'default_mail_group_message_id': id, 'default_action': 'reject'}" />
                    <button name="action_moderate_allow" string="Whitelist"
                        title="Add this email address to white list of people and accept all pending messages from the same author."
                        type="object" class="btn btn-secondary"
                        invisible="not is_group_moderated"/>
                    <button name="%(mail_group_message_reject_action)d"
                        string="Ban" title="Ban this email address and reject all pending messages from the same author and send an email to the author"
                        type="action" class="btn btn-secondary"
                        invisible="moderation_status != 'pending_moderation' or not is_group_moderated"
                        context="{'default_mail_group_message_id': id, 'default_action': 'ban'}" />
                    <button name="action_moderate_accept" string="Send" title="Send"
                        type="object" class="btn btn-primary"
                        invisible="moderation_status != 'pending_moderation' or is_group_moderated"/>
                </header>
                <sheet>
                    <widget name="web_ribbon" title="Rejected" bg_color="text-bg-danger"
                        invisible="moderation_status != 'rejected'"/>
                    <widget name="web_ribbon" title="Accepted" bg_color="text-bg-success"
                        invisible="moderation_status != 'accepted'"/>
                    <group>
                        <field name="mail_message_id" invisible="1"/>
                        <field name="moderation_status" invisible="1"/>
                        <field name="is_group_moderated" invisible="1"/>
                        <field name="subject"/>
                        <field name="author_id"/>
                        <label for="email_from" string="From"/>
                        <div>
                            <field name="email_from" nolabel="1"/>
                            <span class="ms-2 badge text-bg-success" invisible="author_moderation != 'allow'">Whitelisted</span>
                            <span class="ms-2 badge text-bg-danger" invisible="author_moderation != 'ban'">Banned</span>
                            <field name="author_moderation" invisible="1"/>
                        </div>
                        <field name="mail_group_id"/>
                        <field name="create_date"/>
                        <field name="attachment_ids" widget="many2many_binary"/>
                        <field name="body" options="{'style-inline': true}"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
    <record id="mail_group_message_view_search" model="ir.ui.view">
        <field name="name">mail.group.message.view.search</field>
        <field name="model">mail.group.message</field>
        <field name="arch" type="xml">
            <search string="Search Group Message">
                <field name="mail_group_id"/>
                <field name="email_from"/>
                <field name="author_id"/>
                <field name="moderation_status"/>
                <separator/>
                <group expand="0" string="Group By">
                    <filter string="group" name="group_by_group" context="{'group_by': 'mail_group_id'}"/>
                </group>
            </search>
        </field>
    </record>
    <record id="mail_group_message_action" model="ir.actions.act_window">
        <field name="name">Messages</field>
        <field name="res_model">mail.group.message</field>
        <field name="view_mode">tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">No Messages in this list yet!</p>
            <p>When people send an email to the alias of the list, they will appear here.</p>
        </field>
    </record>
</odoo>

```

## File: views\mail_group_moderation_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="mail_group_moderation_view_tree" model="ir.ui.view">
        <field name="name">mail.group.moderation.view.tree</field>
        <field name="model">mail.group.moderation</field>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <tree string="Moderation Lists" editable="bottom" sample="1">
                <field name="mail_group_id"/>
                <field name="email"/>
                <field name="status"/>
            </tree>
        </field>
    </record>
    <record id="mail_group_moderation_view_search" model="ir.ui.view">
        <field name="name">mail.group.moderation.view.search</field>
        <field name="model">mail.group.moderation</field>
        <field name="priority">25</field>
        <field name="arch" type="xml">
            <search string="Search Moderation List">
                <field name="mail_group_id"/>
                <field name="email"/>
                <field name="status"/>
                <filter string="Is Banned"
                        name="status_ban" help="Banned Emails"
                        domain="[('status', '=', 'ban')]"/>
                <separator/>
                <filter string="Is Allowed"
                        name="status_allow" help="Allowed Emails"
                        domain="[('status', '=', 'allow')]"/>
                <filter name="group_by_status" string="Status" domain="[]" context="{'group_by':'status'}"/>
            </search>
        </field>
    </record>
    <record id="mail_group_moderation_action" model="ir.actions.act_window">
        <field name="name">Moderation</field>
        <field name="res_model">mail.group.moderation</field>
        <field name="view_mode">tree,form</field>
        <field name="search_view_id" ref="mail_group_moderation_view_search"/>
    </record>
</odoo>

```

## File: views\mail_group_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="mail_group_view_list" model="ir.ui.view">
        <field name="name">mail.group.view.list</field>
        <field name="model">mail.group</field>
        <field name="arch" type="xml">
            <tree sample="1">
                <field name="name"/>
                <field name="alias_email" string="Alias"/>
                <field name="moderation" string="Moderated"/>
                <field name="member_count" string="Members"/>
            </tree>
        </field>
    </record>
    <record id="mail_group_view_kanban" model="ir.ui.view">
        <field name="name">mail.group.view.kanban</field>
        <field name="model">mail.group</field>
        <field name="arch" type="xml">
            <kanban string="Mail Groups" sample="1">
                <field name="id"/>
                <field name="description"/>
                <field name="is_member"/>
                <field name="member_count"/>
                <templates>
                    <t t-name="kanban-description">
                        <div class="oe_group_description" t-if="record.description.raw_value">
                            <field name="description"/>
                        </div>
                    </t>
                    <t t-name="kanban-box">
                        <div class="oe_module_vignette oe_kanban_global_click">
                            <img t-att-src="kanban_image('mail.group', 'image_128', record.id.raw_value)" class="oe_module_icon" alt="Group"/>
                            <div class="oe_module_desc">
                                <h4 class="o_kanban_record_title">#<field name="name"/></h4>
                                <p class="o_mg_description text-truncate text-nowrap" t-esc="record.description.raw_value or ''"/>
                                <span>
                                    <t t-esc="record.member_count.raw_value"/>
                                    <t t-if="record.member_count.raw_value > 1">Members</t>
                                    <t t-else="">Member</t>
                                </span>
                                <field name="is_member" invisible="1"/>
                                <button type="object" invisible="is_member" class="btn btn-primary float-end" name="action_join">Join</button>
                                <button type="object" invisible="not is_member" class="btn btn-secondary float-end" name="action_leave">Leave</button>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>
    <record id="mail_group_view_form" model="ir.ui.view">
        <field name="name">mail.group.view.form</field>
        <field name="model">mail.group</field>
        <field name="arch" type="xml">
            <form string="Mail Group">
                <header>
                    <button type="object" invisible="is_member"
                        class="btn btn-primary" name="action_join" string="Join"/>
                    <button type="object" invisible="not is_member"
                        class="btn btn-secondary" name="action_leave" string="Leave"/>
                    <button name="action_send_guidelines" type="object" class="btn btn-secondary" string="Send Guidelines"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box" groups="base.group_user" invisible="not can_manage_group">
                        <button name="%(mail_group.mail_group_member_action)d"
                                type="action"
                                context="{'search_default_mail_group_id': id}"
                                class="oe_stat_button"
                                icon="fa-users"
                                help="Members of this group">
                            <field name="member_count" widget="statinfo" string="Members"/>
                        </button>
                       <button name="%(mail_group.mail_group_message_action)d"
                                type="action"
                                context="{'search_default_mail_group_id': id}"
                                class="oe_stat_button"
                                icon="fa-envelope"
                                help="All messages of this group">
                            <field name="mail_group_message_count" widget="statinfo" string="Emails"/>
                        </button>
                        <button name="%(mail_group.mail_group_message_action)d"
                                type="action"
                                context="{'search_default_mail_group_id': id, 'search_default_moderation_status': 'pending_moderation'}"
                                class="oe_stat_button"
                                icon="fa-commenting-o"
                                help="Emails waiting an action for this group"
                                invisible="not moderation">
                            <field name="mail_group_message_moderation_count" widget="statinfo" string="To Review"/>
                        </button>
                        <button name="%(mail_group.mail_group_moderation_action)d"
                                type="action"
                                context="{'search_default_mail_group_id': id}"
                                invisible="not moderation"
                                class="oe_stat_button"
                                icon="fa-gavel"
                                help="Moderated emails in this group">
                            <field name="moderation_rule_count" widget="statinfo" string="Moderations"/>
                        </button>
                    </div>
                    <field name="image_128" widget="image" class="oe_avatar" options="{'size': [90, 90]}"/>
                    <div class="oe_title">
                        <label for="name" string="Group Name"/>
                        <h1>
                            <field name="name" class="oe_inline" default_focus="1" placeholder='e.g. "Newsletter"'/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="active" invisible="1"/>
                            <label for="alias_name" string="Email Alias"/>
                            <div class="oe_inline" name="alias_def">
                                <field name="alias_id" class="oe_read_only oe_inline" string="Email Alias" required="0"/>
                                <div class="oe_inline" name="edit_alias" style="display: inline;">
                                    <div class="oe_edit_only" dir="ltr">
                                        <field name="alias_name" class="oe_inline"/>@
                                        <field name="alias_domain_id" class="oe_inline" placeholder="e.g. domain.com"
                                               options="{'no_create': True, 'no_open': True}"/>
                                    </div>
                                    <button icon="oi-arrow-right" type="action" name="%(base_setup.action_general_configuration)d"
                                            string="Choose or configure a custom domain" class="p-0 btn-link"
                                            invisible="alias_domain_id"/>
                                </div>
                            </div>
                            <field name="description"/>
                            <field name="moderation"/>
                            <td class="o_td_label">
                                <label for="moderator_ids" string="Moderators" invisible="not moderation"/>
                                <label for="moderator_ids" string="Responsible Users" invisible="moderation"/>
                            </td>
                            <field name="moderator_ids" widget="many2many_tags" class="oe_inline" nolabel="1"/>
                            <field name="is_moderator" invisible="1"/>
                            <field name="can_manage_group" invisible="1"/>
                            <field name="is_member" invisible="1"/>
                        </group>
                    </group>
                    <notebook>
                        <page name="privacy" string="Privacy">
                            <group>
                                <field name="access_mode" widget="radio"/>
                                <field name="access_group_id" invisible="access_mode != 'groups'" required="access_mode == 'groups'"/>
                                <field name="alias_contact" invisible="1"/>
                            </group>
                        </page>
                        <page name="moderation" string="Notify Members" invisible="not moderation">
                            <group>
                                <field name="moderation_notify"/>
                                <field invisible="not moderation_notify" required="moderation_notify" name="moderation_notify_msg"/>
                            </group>
                        </page>
                        <page name="guidelines" string="Guidelines">
                            <group>
                                <field name="moderation_guidelines"/>
                                <field required="moderation_guidelines" name="moderation_guidelines_msg"/>
                            </group>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>
    <record id="mail_group_view_search" model="ir.ui.view">
        <field name="name">mail.group.view.search</field>
        <field name="model">mail.group</field>
        <field name="arch" type="xml">
            <search string="Search Mail group">
                <field name="name"/>
                <field name="alias_email"/>
                <separator/>
                <filter string="Archived" name="archived" domain="[('active', '=', False)]"/>
                <filter string="Moderated" name="moderation" domain="[('moderation', '=', True)]"/>
                <group expand="0" string="Group By">
                    <filter string="Moderation" name="Moderation" context="{'group_by':'moderation'}"/>
                </group>
            </search>
        </field>
    </record>
    <record id="mail_group_action" model="ir.actions.act_window">
        <field name="name">Mail Groups</field>
        <field name="res_model">mail.group</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">Create a Mail Group</p>
            <p>Mailing groups are communities that like to discuss a specific topic together.</p>
        </field>
    </record>
</odoo>

```

## File: views\portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!--
        Templates used for the portal view.
    -->

    <template id="mail_groups" name="Mailing Lists">
        <t t-call="portal.portal_layout">
            <div class="oe_structure oe_empty">
                <section class="o_mg_page_description mt0 mb0">
                    <div class="o_we_bg_filter" style="background-color: rgba(1, 126, 132, 0.8) !important">
                        <!-- Color filter effect that can be edited by the website editor -->
                    </div>
                    <div class="container w-100 h-100 text-light p-5">
                        <h1 class="mt-5">Stay in touch with our Community</h1>
                        <p class="mb-5">Alone we can do so little, together we can do so much</p>
                    </div>
                </section>
            </div>
            <div t-if="mail_groups" class="container mt32">
                <div t-if="'unsubscribe' in request.params" class="offset-lg-9 col-lg-3 alert alert-info" role="status">
                    <h5>Need to unsubscribe? <br/>It's right here! <span class="oi fa-2x oi-arrow-down float-end" role="img" aria-label="" title="Read this !"/></h5>
                </div>
                <div class="row mb-4" t-foreach="mail_groups" t-as="group_data">
                    <t t-set="group" t-value="group_data['group']"/>
                    <t t-set="is_member" t-value="group_data['is_member']"/>
                    <div class="col-lg-5">
                        <img t-if="group.image_128" t-attf-src="/web/image/mail.group/#{group.id}/image_128" class="o_image_64_cover float-start me-3" alt="Group"/>
                        <div t-else="" class="o_image_64_cover float-start me-3 d-lg-block d-md-none"/>
                        <div class="d-flex flex-row">
                            <strong><a t-attf-href="/groups/#{ slug(group) }" t-esc="group.name"/></strong>
                            <div t-if="group.alias_email"
                                class="d-flex align-items-center ms-3">
                                <i class="fa fa-envelope-o me-1" role="img" aria-label="Alias" title="Alias"/>
                                <a class="text-break" t-attf-href="mailto:#{group.alias_email}" t-field="group.alias_email"/>
                            </div>
                        </div>
                        <div t-field="group.description" class="text-muted d-flex"/>
                    </div>
                    <div class="col-lg-3">
                        <i class="fa fa-fw fa-user" role="img" aria-label="Recipients" title="Recipients"/> <t t-esc="group.member_count"/> members<br />
                        <i class="fa fa-fw fa-envelope-o" role="img" aria-label="Traffic" title="Traffic"/> <t t-esc="group.mail_group_message_last_month_count"/> messages / month
                    </div>
                    <div class="col-lg-4">
                        <t t-set="force_unsubscribe" t-value="'unsubscribe' in request.params"/>
                        <div class="o_mail_group" t-att-data-id="group.id" t-att-data-is-member="force_unsubscribe or is_member">
                            <div class="input-group o_mg_subscribe_form">
                                <input type="email" name="email" class="o_mg_subscribe_email form-control" t-att-value="email" placeholder="your email..." t-att-readonly="int(bool(email))"/>
                               <button t-if="force_unsubscribe or is_member" href="#" class="btn btn-outline-primary o_mg_subscribe_btn">Unsubscribe</button>
                               <button t-else="" href="#" class="btn btn-primary o_mg_subscribe_btn">Subscribe</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div t-else="" class="container mt32">
                <div class="alert alert-primary text-center">
                    <span>No Mail Group yet.</span>
                    <br/>
                    <a t-if="is_mail_group_manager" class="btn btn-link"
                        href="/web#action=mail_group.mail_group_action&amp;model=mail.group&amp;view_type=form">
                        Create a new group
                    </a>
                </div>
            </div>
        </t>
    </template>

    <template id="group_messages" name="Message Threads">
        <t t-call="portal.portal_layout">
            <section class="container">
                <div class="row w-100 mt-4">
                    <div class="col-lg-3">
                        <t t-call="mail_group.group_archive_menu"/>
                    </div>
                    <div class="col-lg-9">
                        <div>
                            <t t-call="mail_group.group_name"/>
                        </div>
                        <div>
                            <t t-call="portal.pager"/>
                        </div>
                        <t t-call="mail_group.messages_short">
                            <t t-set="messages" t-value="messages"/>
                            <t t-set="msg_more_count" t-value="0"/>
                            <t t-set="parent_message" t-value="None"/>
                        </t>
                        <div>
                            <t t-call="portal.pager"/>
                        </div>
                    </div>
                </div>
            </section>
        </t>
    </template>

    <template id="group_message">
        <t t-call="portal.portal_layout">
        <t t-set="additional_title"><t t-esc="message.subject"/></t>
            <section class="container">
                <div class="row w-100 mt-4">
                    <div class="col-lg-3">
                        <t t-call="mail_group.group_archive_menu"/>
                    </div>
                    <div class="col-lg-9 o_mg_message">
                        <div>
                            <t t-call="mail_group.group_name"/>
                        </div>
                        <div class="row">
                            <h4 t-if="prev_message" t-attf-class="{{'col-lg-6' if next_message else ''}}">
                                <a t-attf-href="/groups/#{slug(group)}/#{slug(prev_message)}?#{mode and 'mode=%s' % mode or ''}">
                                    <i class="oi oi-arrow-left" role="img" aria-label="Previous message" title="Previous message"/> <t t-esc="prev_message.subject"/>
                                </a>
                            </h4>
                            <h4 t-if="next_message" t-attf-class="{{'col-lg-6' if prev_message else ''}} text-end">
                                <a t-attf-href="/groups/#{slug(group)}/#{slug(next_message)}?#{mode and 'mode=%s' % mode or ''}">
                                    <t t-esc="next_message.subject"/> <i class="oi oi-arrow-right" role="img" aria-label="Next message" title="Next message"/>
                                </a>
                            </h4>
                        </div>
                        <div class="d-flex">
                            <div class="flex-grow-1">
                                <div class="card">
                                    <div class="card-body">
                                        <h4 class="card-title d-flex flex-row justify-content-start align-items-center">
                                            <img t-if="message.author_id.active and message.author_id.image_128" t-att-src="image_data_uri(message.author_id.image_128)" class="rounded o_image_40_cover me-2"/>
                                            <span t-esc="message.subject"/>
                                        </h4>
                                        <div class="card-text overflow-hidden" t-field="message.body"/>
                                        <t t-call="mail_group.message_footer"/>
                                        <t t-call="mail_group.message_attachments"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <t t-set="group_message_child_ids" t-value="message.group_message_child_ids.filtered(lambda m: m.moderation_status == 'accepted')"/>
                        <div t-if="group_message_child_ids" class="o_mg_replies mt-3">
                            <h4 class="o_page_header">Follow-Ups</h4>
                            <t t-call="mail_group.messages_short">
                                <t t-set="messages" t-value="group_message_child_ids[:replies_per_page]"/>
                                <t t-set="msg_more_count" t-value="len(group_message_child_ids) - replies_per_page"/>
                                <t t-set="parent_message" t-value="message"/>
                            </t>
                        </div>
                        <div t-if="message.group_message_parent_id and message.group_message_parent_id.moderation_status == 'accepted'">
                            <h4 class="o_page_header">Reference</h4>
                            <t t-call="mail_group.messages_short">
                                <t t-set="messages" t-value="[message.group_message_parent_id]"/>
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
                <li t-foreach="messages" t-as="message" class="d-flex mt-3">
                    <div class="flex-grow-1 o_mg_message mw-100">
                        <div class="card">
                            <div class="o_mg_ribbon" t-if="message.moderation_status == 'pending_moderation'"><span class="bg-warning">Pending</span></div>
                            <div class="card-body">
                                <h5 class="card-title d-flex flex-row justify-content-start align-items-center">
                                    <img t-if="message.author_id.active and message.author_id.image_128" t-att-src="image_data_uri(message.author_id.image_128)" class="rounded o_image_40_cover me-2"/>
                                    <a t-attf-href="/groups/#{slug(group)}/#{slug(message)}?mode=#{mode}&amp;date_begin=#{date_begin}&amp;date_end=#{date_end}" t-esc="message.subject"/>
                                </h5>
                                <div class="card-text overflow-hidden" t-field="message.body"/>
                                <t t-call="mail_group.message_footer"/>
                                <t t-call="mail_group.message_attachments"/>
                            </div>
                        </div>
                        <t t-set="group_message_child_ids" t-value="message.group_message_child_ids.filtered(lambda m: m.moderation_status == 'accepted')"/>
                        <div class="o_mg_link_parent" t-if="group_message_child_ids and (not mode or mode == 'thread')">
                            <p class="mt8">
                                <a href="#" class="o_mg_link_hide">
                                    <i class="oi oi-chevron-right" role="img" aria-label="Hide replies" title="Hide replies"/> <t t-esc="len(group_message_child_ids)"/> replies
                                </a>
                                <a href="#" class="o_mg_link_show d-none">
                                    <i class="oi oi-chevron-down" role="img" aria-label="Show replies" title="Show replies"/> <t t-esc="len(group_message_child_ids)"/> replies
                                </a>
                            </p>
                            <div class="o_mg_link_content o_mg_replies ms-5 d-none">
                                <t t-call="mail_group.messages_short">
                                    <t t-set="messages" t-value="group_message_child_ids[:replies_per_page]"/>
                                    <t t-set="msg_more_count" t-value="len(group_message_child_ids) - replies_per_page"/>
                                    <t t-set="parent_message" t-value="message"/>
                                </t>
                            </div>
                        </div>
                    </div>
                </li>
            </ul>
            <p t-if="messages and (msg_more_count or 0) > 0 and parent_message">
                <button class="btn btn-link o_mg_read_more"
                    t-attf-data-href="/groups/#{slug(group)}/#{slug(parent_message)}/get_replies"
                    t-attf-data-last-displayed-id="#{messages[-1].id}">
                    <t t-esc="msg_more_count"/> more replies
                </button>
            </p>
        </div>
    </template>

    <template id="message_attachments">
        <div class="o_mg_link_parent">
            <t t-set="attachments" t-value="message.attachment_ids"/>
            <p t-if="attachments" class="mt8">
                <a href="#" class="o_mg_link_hide">
                    <i class="oi oi-chevron-right" role="img" aria-label="Hide attachments" title="Hide attachments"/> <t t-esc="len(attachments)"/> attachments
                </a>
                <a href="#" class="o_mg_link_show d-none">
                    <i class="oi oi-chevron-down" role="img" aria-label="Show attachments" title="Show attachments"/> <t t-esc="len(attachments)"/> attachments
                </a>
            </p>
            <div class="o_mg_link_content d-none row justify-content-center">
                <div class="mx-4 my-3 text-center o_mg_attachment" t-foreach="attachments" t-as="attachment">
                    <t t-if="attachment.access_token">
                        <a t-attf-href="/web/content/#{attachment.id}?download=true&amp;access_token=#{attachment.access_token}" target="_blank">
                            <div class="oe_attachment_embedded o_image" t-att-title="attachment.name" t-att-data-mimetype="attachment.mimetype" t-attf-data-src="/web/image/#{attachment.id}/100x80?access_token=#{attachment.access_token}"/>
                            <div class="oe_attachment_name"><t t-esc="attachment.name" /></div>
                        </a>
                    </t>
                    <t t-else="">
                        <a t-attf-href="/web/content/#{attachment.id}?download=true" target="_blank">
                            <div class="oe_attachment_embedded o_image" t-att-title="attachment.name" t-att-data-mimetype="attachment.mimetype" t-attf-data-src="/web/image/#{attachment.id}/100x80"/>
                            <div class="oe_attachment_name"><t t-esc="attachment.name" /></div>
                        </a>
                    </t>
                </div>
            </div>
        </div>
    </template>

    <template id="message_footer" name="Message Footer">
        <hr/>
        <small class="d-inline-block mt-1">
            <span>
                by
                <t t-if="message.author_id" t-esc="message.author_id.name"/>
                <t t-else="" t-esc="message.email_from"/>
            </span>
            <span class="mx-2">-</span>
            <span>
                <i class="fa fa-calendar" role="img" aria-label="Date" title="Date"/>
                <span t-field="message.create_date" t-options="{'widget': 'datetime', 'format': 'hh:mm - d MMM Y'}"/>
            </span>
        </small>
    </template>

    <template id="group_archive_menu">
        <h2>Archives</h2>
        <ul class="nav nav-pills flex-column" id="group_mode">
            <li class="nav-item">
                <a t-attf-href="/groups/#{ slug(group) }?mode=thread" t-attf-class="d-flex align-items-center justify-content-between nav-link#{mode=='thread' and ' active' or ''}">
                    <span>By thread</span>
                    <span class="float-end badge rounded-pill" t-if="archives['threads_count']" t-esc="archives['threads_count']"/>
                </a>
            </li>
            <li class="nav-item">
                <a t-attf-href="/groups/#{ slug(group) }?mode=date" t-attf-class="nav-link#{mode=='date' and not date_begin and ' active' or ''}">By date</a>
                <ul class="nav nav-pills flex-column" style="margin-left: 8px;">
                    <t t-foreach="archives['threads_time_data']" t-as="month_archive">
                    <li class="nav-item">
                        <a t-ignore="True" t-attf-href="/groups/#{ slug(group) }?mode=date&amp;date_begin=#{ month_archive['date_begin'] }&amp;date_end=#{month_archive['date_end']}"
                            t-attf-class="d-flex align-items-center justify-content-between nav-link#{month_archive['date_begin'] == date_begin and ' active' or ''}">
                            <t t-esc="month_archive['date']"/>
                            <span class="float-end badge rounded-pill" t-esc="month_archive['messages_count']"/>
                        </a>
                    </li>
                    </t>
                </ul>
            </li>
        </ul>
    </template>

    <template id="group_name">
        <h1 class="text-center" t-esc="group.name"/>
        <h4 class="text-center text-muted" t-if="group.alias_email">
            <i class="fa fa-envelope-o" role="img" aria-label="Alias" title="Alias"/>
            <a class="text-break" t-attf-href="mailto:#{group.alias_email}" t-field="group.alias_email"/>
        </h4>
    </template>

    <template id="portal_breadcrumbs_group" name="Portal layout : mail group menu entry" inherit_id="portal.portal_breadcrumbs" priority="15">
        <xpath expr="//ol[hasclass('o_portal_submenu')]" position="inside">
            <t t-if="page_name == 'groups'">
                <li class="breadcrumb-item"><a href="/groups">Mailing Lists</a></li>
                <li class="breadcrumb-item">
                    <a t-attf-href="/groups/#{slug(group)}?#{mode and 'mode=%s' % mode or ''}#{date_begin and '&amp;date_begin=%s' % date_begin or ''}#{date_end and '&amp;date_end=%s' % date_end or ''}"><t t-esc="group.name"/></a>
                </li>
                <li t-if="message" class="breadcrumb-item active"><t t-esc="message.subject"/></li>
            </t>
        </xpath>
    </template>

    <template id="confirmation_subscription" name="Mailing List Confirmation">
        <div id="wrap" class="oe_structure oe_empty">
            <t t-call-assets="web.assets_frontend" t-js="false"/>
            <div class="container alert alert-success mt-5">
                The email <strong t-esc="email"/> has been
                <t t-if="subscribing">subscribed to</t>
                <t t-if="not subscribing">unsubscribed from</t>
                the list <strong t-esc="group.name"/>.
                <t t-if="subscribing">
                    <br/> You'll be notified as soon as some new content is posted.
                </t>
            </div>
        </div>
    </template>

    <template id="invalid_token_subscription" name="Invalid Token Submitted">
        <div id="wrap" class="oe_structure oe_empty">
            <t t-call-assets="web.assets_frontend" t-js="false"/>
            <div class="container alert alert-danger mt-5">
                Invalid or expired confirmation link.
            </div>
        </div>
    </template>

</odoo>

```

## File: wizard\mail_group_message_reject.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools, _


class MailGroupMessageReject(models.TransientModel):
    _name = 'mail.group.message.reject'
    _description = 'Reject Group Message'

    subject = fields.Char('Subject', store=True, readonly=False, compute='_compute_subject')
    body = fields.Html('Contents', default='', sanitize_style=True)
    email_from_normalized = fields.Char('Email From', related='mail_group_message_id.email_from_normalized')
    mail_group_message_id = fields.Many2one('mail.group.message', string="Message", required=True, readonly=True)
    action = fields.Selection([('reject', 'Reject'), ('ban', 'Ban')], string='Action', required=True)

    send_email = fields.Boolean('Send Email', help='Send an email to the author of the message', compute='_compute_send_email')

    @api.depends('mail_group_message_id')
    def _compute_subject(self):
        for wizard in self:
            wizard.subject = _('Re: %s', wizard.mail_group_message_id.subject or '')

    @api.depends('body')
    def _compute_send_email(self):
        for wizard in self:
            wizard.send_email = not tools.is_html_empty(wizard.body)

    def action_send_mail(self):
        self.ensure_one()

        # Reject
        if self.action == 'reject' and self.send_email:
            self.mail_group_message_id.action_moderate_reject_with_comment(self.subject, self.body)
        elif self.action == 'reject' and not self.send_email:
            self.mail_group_message_id.action_moderate_reject()

        # Ban
        elif self.action == 'ban' and self.send_email:
            self.mail_group_message_id.action_moderate_ban_with_comment(self.subject, self.body)
        elif self.action == 'ban' and not self.send_email:
            self.mail_group_message_id.action_moderate_ban()

```

## File: wizard\mail_group_message_reject_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mail_group_message_reject_form" model="ir.ui.view">
        <field name="name">mail.group.message.reject.form</field>
        <field name="model">mail.group.message.reject</field>
        <field name="arch" type="xml">
            <form string="Reject">
                <div class="alert alert-warning" role="alert" invisible="action != 'reject'">
                    Reject the message<span invisible="not send_email"> and send an email to the author (<field name="email_from_normalized"/>)</span>.
                </div>
                <div class="alert alert-warning" role="alert" invisible="action != 'ban'">
                    Ban the author of the message (<field name="email_from_normalized"/>) <span invisible="not send_email">and send them an email</span>.
                </div>
                <group>
                    <field name="send_email" invisible="1"/>
                    <field name="mail_group_message_id" invisible="1"/>
                    <field name="subject"/>
                    <field name="body"/>
                    <field name="action" invisible="1"/>
                </group>
                <footer>
                    <button string="Reject Silently" name="action_send_mail" type="object" class="btn-primary"
                        invisible="action != 'reject' or send_email"/>
                    <button string="Send &amp; Reject" name="action_send_mail" type="object" class="btn-primary"
                        invisible="action != 'reject' or not send_email"/>
                    <button string="Ban" name="action_send_mail" type="object" class="btn-primary"
                        invisible="action != 'ban' or send_email"/>
                    <button string="Send &amp; Ban" name="action_send_mail" type="object" class="btn-primary"
                        invisible="action != 'ban' or not send_email"/>
                    <button string="Discard" class="btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="mail_group_message_reject_action" model="ir.actions.act_window">
        <field name="name">Message Rejection Explanation</field>
        <field name="res_model">mail.group.message.reject</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mail_group_message_reject

```

