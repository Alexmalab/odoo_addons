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
    'name': 'Live Chat',
    'version': '1.0',
    'sequence': 210,
    'summary': 'Chat with your website visitors',
    'category': 'Website/Live Chat',
    'complexity': 'easy',
    'website': 'https://www.odoo.com/app/live-chat',
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
        "security/im_livechat_channel_security.xml",
        "security/ir.model.access.csv",
        "data/mail_shortcode_data.xml",
        "data/mail_data.xml",
        "data/im_livechat_channel_data.xml",
        'data/digest_data.xml',
        "views/rating_views.xml",
        "views/mail_channel_views.xml",
        "views/im_livechat_channel_views.xml",
        "views/im_livechat_channel_templates.xml",
        "views/res_users_views.xml",
        "views/digest_views.xml",
        "report/im_livechat_report_channel_views.xml",
        "report/im_livechat_report_operator_views.xml"
    ],
    'demo': [
        "data/im_livechat_channel_demo.xml",
        'data/mail_shortcode_demo.xml',
    ],
    'depends': ["mail", "rating", "digest"],
    'installable': True,
    'auto_install': False,
    'application': True,
    'assets': {
        'mail.assets_discuss_public': [
            'im_livechat/static/src/components/*/*',
            'im_livechat/static/src/models/*/*.js',
        ],
        'web.assets_backend': [
            'im_livechat/static/src/js/im_livechat_channel_form_view.js',
            'im_livechat/static/src/js/im_livechat_channel_form_controller.js',
            'im_livechat/static/src/components/*/*.js',
            'im_livechat/static/src/models/*/*.js',
            'im_livechat/static/src/scss/im_livechat_history.scss',
            'im_livechat/static/src/scss/im_livechat_form.scss',
        ],
        'web.qunit_suite_tests': [
            'im_livechat/static/src/components/*/tests/*.js',
            'im_livechat/static/src/legacy/public_livechat.js',
            'im_livechat/static/tests/helpers/mock_models.js',
            'im_livechat/static/tests/helpers/mock_server.js',
        ],
        'web.assets_qweb': [
            'im_livechat/static/src/components/*/*.xml',
        ],
        # Bundle of External Librairies of the Livechat (Odoo + required modules)
        'im_livechat.external_lib': [
            # Momentjs
            'web/static/lib/moment/moment.js',
            'web/static/lib/luxon/luxon.js',
            # Odoo minimal lib
            'web/static/lib/underscore/underscore.js',
            'web/static/lib/underscore.string/lib/underscore.string.js',
            # jQuery
            'web/static/lib/jquery/jquery.js',
            'web/static/lib/jquery.ui/jquery-ui.js',
            'web/static/lib/jquery/jquery.browser.js',
            'web/static/lib/jquery.ba-bbq/jquery.ba-bbq.js',
            # Qweb2 lib
            'web/static/lib/qweb/qweb2.js',
            # Odoo JS Framework
            'web/static/lib/owl/owl.js',
            'web/static/src/legacy/js/promise_extension.js',
            'web/static/src/boot.js',
            'web/static/src/core/assets.js',
            'web/static/src/core/browser/browser.js',
            'web/static/src/core/browser/feature_detection.js',
            'web/static/src/core/dialog/dialog.js',
            'web/static/src/core/errors/error_dialogs.js',
            'web/static/src/core/effects/**/*.js',
            'web/static/src/core/hotkeys/hotkey_service.js',
            'web/static/src/core/hotkeys/hotkey_hook.js',
            'web/static/src/core/l10n/dates.js',
            'web/static/src/core/l10n/localization.js',
            'web/static/src/core/l10n/localization_service.js',
            'web/static/src/core/l10n/translation.js',
            'web/static/src/core/main_components_container.js',
            'web/static/src/core/network/rpc_service.js',
            'web/static/src/core/notifications/notification.js',
            'web/static/src/core/notifications/notification_container.js',
            'web/static/src/core/notifications/notification_service.js',
            'web/static/src/core/registry.js',
            'web/static/src/core/ui/block_ui.js',
            'web/static/src/core/ui/ui_service.js',
            'web/static/src/core/user_service.js',
            'web/static/src/core/utils/components.js',
            'web/static/src/core/utils/functions.js',
            'web/static/src/core/utils/hooks.js',
            'web/static/src/core/utils/strings.js',
            'web/static/src/core/utils/timing.js',
            'web/static/src/core/utils/ui.js',
            'web/static/src/env.js',
            'web/static/src/legacy/utils.js',
            'web/static/src/legacy/js/owl_compatibility.js',
            'web/static/src/legacy/js/libs/download.js',
            'web/static/src/legacy/js/libs/content-disposition.js',
            'web/static/src/legacy/js/libs/pdfjs.js',
            'web/static/src/legacy/js/services/config.js',
            'web/static/src/legacy/js/core/abstract_service.js',
            'web/static/src/legacy/js/core/class.js',
            'web/static/src/legacy/js/core/collections.js',
            'web/static/src/legacy/js/core/translation.js',
            'web/static/src/legacy/js/core/ajax.js',
            'im_livechat/static/src/js/ajax_external.js',
            'web/static/src/legacy/js/core/time.js',
            'web/static/src/legacy/js/core/mixins.js',
            'web/static/src/legacy/js/core/service_mixins.js',
            'web/static/src/legacy/js/core/rpc.js',
            'web/static/src/legacy/js/core/widget.js',
            'web/static/src/legacy/js/core/registry.js',
            'web/static/src/session.js',
            'web/static/src/legacy/js/core/session.js',
            'web/static/src/legacy/js/core/concurrency.js',
            'web/static/src/legacy/js/core/cookie_utils.js',
            'web/static/src/legacy/js/core/utils.js',
            'web/static/src/legacy/js/core/dom.js',
            'web/static/src/legacy/js/core/qweb.js',
            'web/static/src/legacy/js/core/bus.js',
            'web/static/src/legacy/js/services/core.js',
            'web/static/src/legacy/js/core/local_storage.js',
            'web/static/src/legacy/js/core/ram_storage.js',
            'web/static/src/legacy/js/core/abstract_storage_service.js',
            'web/static/src/legacy/js/common_env.js',
            'web/static/src/legacy/js/public/lazyloader.js',
            'web/static/src/legacy/js/public/public_env.js',
            'web/static/src/legacy/js/public/public_root.js',
            'web/static/src/legacy/js/public/public_root_instance.js',
            'web/static/src/legacy/js/public/public_widget.js',
            'web/static/src/legacy/js/services/ajax_service.js',
            'web/static/src/legacy/js/services/local_storage_service.js',
            # Bus, Mail, Livechat
            'bus/static/src/js/longpolling_bus.js',
            'bus/static/src/js/crosstab_bus.js',
            'bus/static/src/js/services/bus_service.js',
            'mail/static/src/js/utils.js',
            'im_livechat/static/src/legacy/public_livechat.js',

            ('include', 'web._assets_helpers'),

            'web/static/lib/bootstrap/scss/_variables.scss',
            'im_livechat/static/src/scss/im_livechat_bootstrap.scss',
            'im_livechat/static/src/legacy/public_livechat.scss',
        ]
    },
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
        bundle = 'im_livechat.external_lib'
        files, _ = request.env["ir.qweb"]._get_asset_content(bundle)
        asset = AssetsBundle(bundle, files)

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
            'im_livechat/static/src/legacy/public_livechat.xml',
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
            country_code = request.session.geoip.get('country_code') if request.session.geoip else False
            if country_code:
                country_id = request.env['res.country'].sudo().search([('code', '=', country_code)], limit=1).id
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
        channel = request.env['mail.channel'].sudo().search([('uuid', '=', uuid)], limit=1)
        if channel:
            # limit the creation : only ONE rating per session
            values = {
                'rating': rate,
                'consumed': True,
                'feedback': reason,
                'is_internal': False,
            }
            if not channel.rating_ids:
                values.update({
                    'res_id': channel.id,
                    'res_model_id': request.env['ir.model']._get_id('mail.channel'),
                })
                # find the partner (operator)
                if channel.channel_partner_ids:
                    values['rated_partner_id'] = channel.channel_partner_ids[0].id
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
        channel = request.env['mail.channel'].sudo().search([('uuid', '=', uuid)], limit=1)
        channel.notify_typing(is_typing=is_typing)

    @http.route('/im_livechat/email_livechat_transcript', type='json', auth='public', cors="*")
    def email_livechat_transcript(self, uuid, email):
        channel = request.env['mail.channel'].sudo().search([
            ('channel_type', '=', 'livechat'),
            ('uuid', '=', uuid)], limit=1)
        if channel:
            channel._email_livechat_transcript(email)

    @http.route('/im_livechat/visitor_leave_session', type='json', auth="public")
    def visitor_leave_session(self, uuid):
        """ Called when the livechat visitor leaves the conversation.
         This will clean the chat request and warn the operator that the conversation is over.
         This allows also to re-send a new chat request to the visitor, as while the visitor is
         in conversation with an operator, it's not possible to send the visitor a chat request."""
        mail_channel = request.env['mail.channel'].sudo().search([('uuid', '=', uuid)])
        if mail_channel:
            mail_channel._close_livechat_session()

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
from . import main

```

## File: data\digest_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data noupdate="1">
        <record id="digest.digest_digest_default" model="digest.digest">
            <field name="kpi_livechat_rating">True</field>
            <field name="kpi_livechat_conversations">True</field>
            <field name="kpi_livechat_response">True</field>
        </record>
    </data>

    <data>
        <record id="digest_tip_im_livechat_0" model="digest.tip">
            <field name="name">Tip: Use canned responses to chat faster</field>
            <field name="sequence">2900</field>
            <field name="group_id" ref="im_livechat.im_livechat_group_manager" />
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Use canned responses to chat faster</p>
    <p class="tip_content">Use canned responses to define templates of messages in the livechat app. To load a canned response, start your sentence with ':' and select the template.</p>
    <img src="/im_livechat/static/src/img/canned-responses.gif" class="illustration_border" />
</div>
            </field>
        </record>
    </data>
</odoo>

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
            <field name="channel_id" ref="im_livechat_channel_data"/>
        </record>
        <record id="im_livechat.im_livechat_group_user" model="res.groups">
            <field eval="[(4, ref('base.user_demo'))]" name="users"/>
        </record>
        <record id="im_livechat_channel_data" model="im_livechat.channel">
            <field eval="[(4, ref('base.user_admin'))]" name="user_ids"/>
        </record>
        <record id="im_livechat_channel_data" model="im_livechat.channel">
            <field eval="[(4, ref('base.user_demo'))]" name="user_ids"/>
        </record>
        <!-- Session 0 -->
        <record id="im_livechat_mail_channel_data_0" model="mail.channel">
            <field name="channel_type">livechat</field>
            <field name="livechat_channel_id" ref="im_livechat_channel_data"/>
            <field name="livechat_operator_id" ref="base.partner_admin"/>
            <field name="name">Visitor #234, Mitchell Admin</field>
            <field name="anonymous_name">Visitor #234, Mitchell Admin</field>
        </record>
        <record id="im_livechat_rating_1" model="rating.rating">
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_0"/>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_admin"/>
            <field eval="False" name="partner_id"/>
            <field eval="True" name="consumed"/>
        </record>
        <function eval="([ref('im_livechat.im_livechat_mail_channel_data_0')], 5)" model="mail.channel" name="rating_apply"/>
        <record id="im_livechat_mail_message_5_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_0"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="body">You're welcome, have a nice day!</field>
            <field eval="DateTime.today() + relativedelta(months=-1, days=-0, minutes=5)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_4_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_0"/>
            <field name="message_type">email</field>
            <field eval="False" name="author_id"/>
            <field name="body">Great! Thanks for the info</field>
            <field eval="DateTime.today() + relativedelta(months=-1, days=-0, minutes=4)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_3_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_0"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="body">Yes, you can use our Timesheets application and Awesome Timesheets to record your time efficiently!</field>
            <field eval="DateTime.today() + relativedelta(months=-1, days=-0, minutes=3)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_2_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_0"/>
            <field name="message_type">email</field>
            <field eval="False" name="author_id"/>
            <field name="body">I'm looking for an application to record my timesheet, any tips?</field>
            <field eval="DateTime.today() + relativedelta(months=-1, days=-0, minutes=2)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_1_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_0"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="body">Hello, how may I help you?</field>
            <field eval="DateTime.today() + relativedelta(months=-1, days=-0, minutes=1)" name="date"/>
        </record>
        <!-- Session 1 -->
        <record id="im_livechat_mail_channel_data_1" model="mail.channel">
            <field name="channel_type">livechat</field>
            <field name="livechat_channel_id" ref="im_livechat_channel_data"/>
            <field name="livechat_operator_id" ref="base.partner_demo"/>
            <field name="name">Visitor #323, Marc Demo</field>
            <field name="anonymous_name">Visitor #323, Marc Demo</field>
        </record>
        <record id="im_livechat_rating_2" model="rating.rating">
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_1"/>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_demo"/>
            <field eval="False" name="partner_id"/>
            <field eval="True" name="consumed"/>
        </record>
        <function eval="([ref('im_livechat.im_livechat_mail_channel_data_1')], 5)" model="mail.channel" name="rating_apply"/>
        <record id="im_livechat_mail_message_10_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_1"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="body">You're welcome, enjoy Odoo!</field>
            <field eval="DateTime.today() + relativedelta(months=-1, days=-1, minutes=10)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_9_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_1"/>
            <field name="message_type">email</field>
            <field eval="False" name="author_id"/>
            <field name="body">Awesome, thanks!</field>
            <field eval="DateTime.today() + relativedelta(months=-1, days=-1, minutes=9)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_8_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_1"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="body">Yes, we just released a new application called Social Marketing that should fit your needs! Check it out :)</field>
            <field eval="DateTime.today() + relativedelta(months=-1, days=-1, minutes=8)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_7_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_1"/>
            <field name="message_type">email</field>
            <field eval="False" name="author_id"/>
            <field name="body">I was wondering if Odoo has an application to easily manage social media for my business..</field>
            <field eval="DateTime.today() + relativedelta(months=-1, days=-1, minutes=7)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_6_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_1"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="body">Hello, how may I help you?</field>
            <field eval="DateTime.today() + relativedelta(months=-1, days=-1, minutes=6)" name="date"/>
        </record>
        <!-- Session 2 -->
        <record id="im_livechat_mail_channel_data_2" model="mail.channel">
            <field name="channel_type">livechat</field>
            <field name="livechat_channel_id" ref="im_livechat_channel_data"/>
            <field name="livechat_operator_id" ref="base.partner_admin"/>
            <field name="name">Joel Willis, Mitchell Admin</field>
            <field name="anonymous_name">Joel Willis, Mitchell Admin</field>
        </record>
        <record id="im_livechat_rating_3" model="rating.rating">
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_2"/>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_admin"/>
            <field name="partner_id" ref="base.partner_demo_portal"/>
            <field eval="True" name="consumed"/>
        </record>
        <function eval="([ref('im_livechat.im_livechat_mail_channel_data_2')], 5)" model="mail.channel" name="rating_apply"/>
        <record id="im_livechat_mail_message_14_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_2"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_demo_portal"/>
            <field name="body">Oh :(</field>
            <field eval="DateTime.today() + relativedelta(months=-1, days=-2, minutes=14)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_13_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_2"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="body">Nope, sorry to disappoint :(</field>
            <field eval="DateTime.today() + relativedelta(months=-1, days=-2, minutes=13)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_12_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_2"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_demo_portal"/>
            <field name="body">Hello, are you single?</field>
            <field eval="DateTime.today() + relativedelta(months=-1, days=-2, minutes=12)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_11_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_2"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="body">Hello, how may I help you?</field>
            <field eval="DateTime.today() + relativedelta(months=-1, days=-2, minutes=11)" name="date"/>
        </record>
        <!-- Session 3 -->
        <record id="im_livechat_mail_channel_data_3" model="mail.channel">
            <field name="channel_type">livechat</field>
            <field name="livechat_channel_id" ref="im_livechat_channel_data"/>
            <field name="livechat_operator_id" ref="base.partner_demo"/>
            <field name="name">Joel Willis, Marc Demo</field>
            <field name="anonymous_name">Joel Willis, Marc Demo</field>
        </record>
        <record id="im_livechat_rating_4" model="rating.rating">
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_3"/>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_demo"/>
            <field name="partner_id" ref="base.partner_demo_portal"/>
            <field eval="True" name="consumed"/>
        </record>
        <function eval="([ref('im_livechat.im_livechat_mail_channel_data_3')], 5)" model="mail.channel" name="rating_apply"/>
        <record id="im_livechat_mail_message_17_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_3"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_demo_portal"/>
            <field name="body">Thanks for the info, I'll look into it!</field>
            <field eval="DateTime.today() + relativedelta(months=-2, days=-3, minutes=17)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_16_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_3"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="body">Hello Joel Willis, you're at the right place! You can customize Odoo using our Studio application in just a few clicks.</field>
            <field eval="DateTime.today() + relativedelta(months=-2, days=-3, minutes=16)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_15_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_3"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_demo_portal"/>
            <field name="body">Hello, I'm looking for a software that can be easily updated with my needs.</field>
            <field eval="DateTime.today() + relativedelta(months=-2, days=-3, minutes=15)" name="date"/>
        </record>
        <!-- Session 4 -->
        <record id="im_livechat_mail_channel_data_4" model="mail.channel">
            <field name="channel_type">livechat</field>
            <field name="livechat_channel_id" ref="im_livechat_channel_data"/>
            <field name="livechat_operator_id" ref="base.partner_admin"/>
            <field name="name">Visitor #532, Mitchell Admin</field>
            <field name="anonymous_name">Visitor #532, Mitchell Admin</field>
        </record>
        <record id="im_livechat_rating_5" model="rating.rating">
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_4"/>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_admin"/>
            <field eval="False" name="partner_id"/>
            <field eval="True" name="consumed"/>
        </record>
        <function eval="([ref('im_livechat.im_livechat_mail_channel_data_4')], 1)" model="mail.channel" name="rating_apply"/>
        <record id="im_livechat_mail_message_21_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_4"/>
            <field name="message_type">email</field>
            <field eval="False" name="author_id"/>
            <field name="body">Ok.. Will do, thanks</field>
            <field eval="DateTime.today() + relativedelta(months=-2, days=-4, minutes=21)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_20_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_4"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="body">Hi, if you need help with your database, feel free to contact our support via http://www.odoo.com/help</field>
            <field eval="DateTime.today() + relativedelta(months=-2, days=-4, minutes=20)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_19_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_4"/>
            <field name="message_type">email</field>
            <field eval="False" name="author_id"/>
            <field name="body">Hello, it seems that I can't log in to my database. Can you help?</field>
            <field eval="DateTime.today() + relativedelta(months=-2, days=-4, minutes=19)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_18_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_4"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="body">Hello, how may I help you?</field>
            <field eval="DateTime.today() + relativedelta(months=-2, days=-4, minutes=18)" name="date"/>
        </record>
        <!-- Session 5 -->
        <record id="im_livechat_mail_channel_data_5" model="mail.channel">
            <field name="channel_type">livechat</field>
            <field name="livechat_channel_id" ref="im_livechat_channel_data"/>
            <field name="livechat_operator_id" ref="base.partner_admin"/>
            <field name="name">Visitor #649, Mitchell Admin</field>
            <field name="anonymous_name">Visitor #649, Mitchell Admin</field>
        </record>
        <record id="im_livechat_rating_6" model="rating.rating">
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_5"/>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_admin"/>
            <field eval="False" name="partner_id"/>
            <field eval="True" name="consumed"/>
        </record>
        <function eval="([ref('im_livechat.im_livechat_mail_channel_data_5')], 5)" model="mail.channel" name="rating_apply"/>
        <record id="im_livechat_mail_message_25_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_5"/>
            <field name="message_type">email</field>
            <field eval="False" name="author_id"/>
            <field name="body">Thanks!</field>
            <field eval="DateTime.today() + relativedelta(months=-3, days=-5, minutes=25)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_24_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_5"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="body">Yes, of course, you can find it here: https://www.odoo.com/documentation/user/14.0/</field>
            <field eval="DateTime.today() + relativedelta(months=-3, days=-5, minutes=24)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_23_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_5"/>
            <field name="message_type">email</field>
            <field eval="False" name="author_id"/>
            <field name="body">Hello, I'm a bit lost in the Invetory module, is there some documentation I could find?</field>
            <field eval="DateTime.today() + relativedelta(months=-3, days=-5, minutes=23)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_22_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_5"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="body">Hello, how may I help you?</field>
            <field eval="DateTime.today() + relativedelta(months=-3, days=-5, minutes=22)" name="date"/>
        </record>
        <!-- Session 6 -->
        <record id="im_livechat_mail_channel_data_6" model="mail.channel">
            <field name="channel_type">livechat</field>
            <field name="livechat_channel_id" ref="im_livechat_channel_data"/>
            <field name="livechat_operator_id" ref="base.partner_admin"/>
            <field name="name">Joel Willis, Mitchell Admin</field>
            <field name="anonymous_name">Joel Willis, Mitchell Admin</field>
        </record>
        <record id="im_livechat_rating_7" model="rating.rating">
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_6"/>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_admin"/>
            <field name="partner_id" ref="base.partner_demo_portal"/>
            <field eval="True" name="consumed"/>
        </record>
        <function eval="([ref('im_livechat.im_livechat_mail_channel_data_6')], 5)" model="mail.channel" name="rating_apply"/>
        <record id="im_livechat_mail_message_29_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_6"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_demo_portal"/>
            <field name="body">Good to hear, thanks!</field>
            <field eval="DateTime.today() + relativedelta(months=-3, days=-6, minutes=29)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_28_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_6"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="body">Joel Willis, you'll need our Inventory and Sales application to do so. You can try them for 15 days, FOR FREE :)</field>
            <field eval="DateTime.today() + relativedelta(months=-3, days=-6, minutes=28)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_27_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_6"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_demo_portal"/>
            <field name="body">Hi, I need a software to easily manage my stock, and generate sales orders.</field>
            <field eval="DateTime.today() + relativedelta(months=-3, days=-6, minutes=27)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_26_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_6"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="body">Hello, how may I help you?</field>
            <field eval="DateTime.today() + relativedelta(months=-3, days=-6, minutes=26)" name="date"/>
        </record>
        <!-- Session 7 -->
        <record id="im_livechat_mail_channel_data_7" model="mail.channel">
            <field name="channel_type">livechat</field>
            <field name="livechat_channel_id" ref="im_livechat_channel_data"/>
            <field name="livechat_operator_id" ref="base.partner_demo"/>
            <field name="name">Visitor #722, Marc Demo</field>
            <field name="anonymous_name">Visitor #722, Marc Demo</field>
        </record>
        <record id="im_livechat_rating_8" model="rating.rating">
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_7"/>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_demo"/>
            <field eval="False" name="partner_id"/>
            <field eval="True" name="consumed"/>
        </record>
        <function eval="([ref('im_livechat.im_livechat_mail_channel_data_7')], 5)" model="mail.channel" name="rating_apply"/>
        <record id="im_livechat_mail_message_33_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_7"/>
            <field name="message_type">email</field>
            <field eval="False" name="author_id"/>
            <field name="body">I'm great, thanks for asking!</field>
            <field eval="DateTime.today() + relativedelta(months=-3, days=-7, minutes=33)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_32_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_7"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="body">I'm fine, and you?</field>
            <field eval="DateTime.today() + relativedelta(months=-3, days=-7, minutes=32)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_31_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_7"/>
            <field name="message_type">email</field>
            <field eval="False" name="author_id"/>
            <field name="body">Heeeey Marc, how are you?</field>
            <field eval="DateTime.today() + relativedelta(months=-3, days=-7, minutes=31)" name="date"/>
        </record>
        <record id="im_livechat_mail_message_30_data" model="mail.message">
            <field name="model">mail.channel</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_7"/>
            <field name="message_type">email</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="body">Hello, how may I help you?</field>
            <field eval="DateTime.today() + relativedelta(months=-3, days=-7, minutes=30)" name="date"/>
        </record>

        <record id="mail_channel_livechat_1" model="mail.channel">
            <field name="name">Visitor, Mitchell Admin</field>
            <field name="livechat_operator_id" ref="base.partner_admin"/>
            <field name="livechat_channel_id" ref="im_livechat.im_livechat_channel_data"/>
            <field name="create_date" eval="datetime.now() - timedelta(days=1)"/>
            <field name="channel_type">livechat</field>
        </record>

        <record id="mail_message_livechat_1" model="mail.message">
            <field name="author_id" eval="False"/>
            <field name="record_name">Visitor</field>
            <field name="date" eval="datetime.now() - timedelta(days=1)"/>
            <field name="create_date" eval="datetime.now() - timedelta(days=1)"/>
            <field name="body">Hi</field>
            <field name="res_id" ref="im_livechat.mail_channel_livechat_1"/>
            <field name="model">mail.channel</field>
        </record>
        <record id="mail_message_livechat_2" model="mail.message">
            <field name="author_id" ref="base.partner_admin"/>
            <field name="date" eval="datetime.now() - timedelta(days=1, seconds=-15)"/>
            <field name="create_date" eval="datetime.now() - timedelta(days=1, seconds=-15)"/>
            <field name="body">Hello, how may I help you?</field>
            <field name="res_id" ref="im_livechat.mail_channel_livechat_1"/>
            <field name="model">mail.channel</field>
        </record>
        <record id="mail_message_livechat_3" model="mail.message">
            <field name="author_id" eval="False"/>
            <field name="record_name">Visitor</field>
            <field name="date" eval="datetime.now() - timedelta(days=1, seconds=-25)"/>
            <field name="create_date" eval="datetime.now() - timedelta(days=1, seconds=-25)"/>
            <field name="body">I would like to know more about the CRM application</field>
            <field name="res_id" ref="im_livechat.mail_channel_livechat_1"/>
            <field name="model">mail.channel</field>
        </record>
        <record id="mail_message_livechat_4" model="mail.message">
            <field name="author_id" ref="base.partner_admin"/>
            <field name="date" eval="datetime.now() - timedelta(days=1, seconds=-33)"/>
            <field name="create_date" eval="datetime.now() - timedelta(days=1, seconds=-33)"/>
            <field name="body">The CRM application helps you to track leads, close opportunities and get accurate forecasts. You can test it for free on our website.</field>
            <field name="res_id" ref="im_livechat.mail_channel_livechat_1"/>
            <field name="model">mail.channel</field>
        </record>
        <record id="mail_message_livechat_5" model="mail.message">
            <field name="author_id" eval="False"/>
            <field name="record_name">Visitor</field>
            <field name="date" eval="datetime.now() - timedelta(days=1, seconds=-42)"/>
            <field name="create_date" eval="datetime.now() - timedelta(days=1, seconds=-42)"/>
            <field name="body">Great, thanks!</field>
            <field name="res_id" ref="im_livechat.mail_channel_livechat_1"/>
            <field name="model">mail.channel</field>
        </record>
        <record id="mail_message_livechat_6" model="mail.message">
            <field name="author_id" eval="False"/>
            <field name="record_name">Visitor</field>
            <field name="date" eval="datetime.now() - timedelta(days=1, seconds=-53)"/>
            <field name="create_date" eval="datetime.now() - timedelta(days=1, seconds=-53)"/>
            <field name="body">Rating: :-)</field>
            <field name="res_id" ref="im_livechat.mail_channel_livechat_1"/>
            <field name="model">mail.channel</field>
        </record>
        <record id="rating_rating_livechat_1" model="rating.rating">
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_admin"/>
            <field name="partner_id" ref="base.partner_admin"/>
            <field name="create_date" eval="datetime.now() - timedelta(days=1, seconds=-53)"/>
            <field name="res_id" ref="im_livechat.mail_channel_livechat_1"/>
            <field name="rating" eval="5"/>
            <field name="consumed" eval="True"/>
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
                <span t-esc="channel.livechat_operator_id.user_livechat_username or channel.livechat_operator_id.name"/>, on the
                <span t-field="channel.livechat_channel_id.create_date"/>
            </div>
            <table cellspacing="0" cellpadding="0" style="width:100%; border-collapse: collapse;">
                <t t-foreach="channel.message_ids.sorted(key=lambda m: m.date)" t-as="message" >
                    <t t-set="author_name"><t t-if="message.author_id" t-esc="message.author_id.user_livechat_username or message.author_id.name"></t><t t-else="">You</t></t>
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

## File: models\digest.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Digest(models.Model):
    _inherit = 'digest.digest'

    kpi_livechat_rating = fields.Boolean('% of Happiness')
    kpi_livechat_rating_value = fields.Float(digits=(16, 2), compute='_compute_kpi_livechat_rating_value')
    kpi_livechat_conversations = fields.Boolean('Conversations handled')
    kpi_livechat_conversations_value = fields.Integer(compute='_compute_kpi_livechat_conversations_value')
    kpi_livechat_response = fields.Boolean('Time to answer(sec)', help="Time to answer the user in second.")
    kpi_livechat_response_value = fields.Float(digits=(16, 2), compute='_compute_kpi_livechat_response_value')

    def _compute_kpi_livechat_rating_value(self):
        channels = self.env['mail.channel'].search([('livechat_operator_id', '=', self.env.user.partner_id.id)])
        for record in self:
            start, end, company = record._get_kpi_compute_parameters()
            domain = [
                ('create_date', '>=', start), ('create_date', '<', end),
                ('rated_partner_id', '=', self.env.user.partner_id.id)
            ]
            ratings = channels.rating_get_grades(domain)
            record.kpi_livechat_rating_value = ratings['great'] * 100 / sum(ratings.values()) if sum(ratings.values()) else 0

    def _compute_kpi_livechat_conversations_value(self):
        for record in self:
            start, end, company = record._get_kpi_compute_parameters()
            record.kpi_livechat_conversations_value = self.env['mail.channel'].search_count([
                ('channel_type', '=', 'livechat'),
                ('livechat_operator_id', '=', self.env.user.partner_id.id),
                ('create_date', '>=', start), ('create_date', '<', end)
            ])

    def _compute_kpi_livechat_response_value(self):
        for record in self:
            start, end, company = record._get_kpi_compute_parameters()
            response_time = self.env['im_livechat.report.operator'].sudo().read_group([
                ('start_date', '>=', start), ('start_date', '<', end),
                ('partner_id', '=', self.env.user.partner_id.id)], ['partner_id', 'time_to_answer'], ['partner_id'])
            record.kpi_livechat_response_value = sum(
                response['time_to_answer']
                for response in response_time
                if response['time_to_answer'] > 0
            )

    def _compute_kpis_actions(self, company, user):
        res = super(Digest, self)._compute_kpis_actions(company, user)
        res['kpi_livechat_rating'] = 'im_livechat.rating_rating_action_livechat_report'
        res['kpi_livechat_conversations'] = 'im_livechat.im_livechat_report_operator_action'
        res['kpi_livechat_response'] = 'im_livechat.im_livechat_report_channel_time_to_answer_action'
        return res

```

## File: models\im_livechat_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64
import random
import re

from odoo import api, Command, fields, models, modules, _


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

    def _default_button_text(self):
        return _('Have a Question? Chat with us.')

    def _default_default_message(self):
        return _('How may I help you?')

    # attribute fields
    name = fields.Char('Name', required=True, help="The name of the channel")
    button_text = fields.Char('Text of the Button', default=_default_button_text, translate=True,
        help="Default text displayed on the Livechat Support Button")
    default_message = fields.Char('Welcome Message', default=_default_default_message, translate=True,
        help="This is an automated 'welcome' message that your visitor will see when they initiate a new conversation.")
    input_placeholder = fields.Char('Chat Input Placeholder', translate=True, help='Text that prompts the user to initiate the chat.')
    header_background_color = fields.Char(default="#875A7B", help="Default background color of the channel header once open")
    title_color = fields.Char(default="#FFFFFF", help="Default title color of the channel once open")
    button_background_color = fields.Char(default="#878787", help="Default background color of the Livechat button")
    button_text_color = fields.Char(default="#FFFFFF", help="Default text color of the Livechat button")

    # computed fields
    web_page = fields.Char('Web Page', compute='_compute_web_page_link', store=False, readonly=True,
        help="URL to a static page where you client can discuss with the operator of the channel.")
    are_you_inside = fields.Boolean(string='Are you inside the matrix?',
        compute='_are_you_inside', store=False, readonly=True)
    script_external = fields.Html('Script (external)', compute='_compute_script_external', store=False, readonly=True, sanitize=False)
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
        view = self.env.ref('im_livechat.external_loader')
        values = {
            "dbname": self._cr.dbname,
        }
        for record in self:
            values["channel_id"] = record.id
            values["url"] = record.get_base_url()
            record.script_external = view._render(values) if record.id else False

    def _compute_web_page_link(self):
        for record in self:
            record.web_page = "%s/im_livechat/support/%i" % (record.get_base_url(), record.id) if record.id else False

    @api.depends('channel_ids')
    def _compute_nbr_channel(self):
        data = self.env['mail.channel'].read_group([
            ('livechat_channel_id', 'in', self._ids),
            ('has_message', '=', True)], ['__count'], ['livechat_channel_id'], lazy=False)
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
            :returns : the ir.action 'action_view_rating' with the correct context
        """
        self.ensure_one()
        action = self.env['ir.actions.act_window']._for_xml_id('im_livechat.rating_rating_action_livechat')
        action['context'] = {'search_default_parent_res_name': self.name}
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
        channel_partner_to_add = [Command.create({'partner_id': operator_partner_id, 'is_pinned': False})]
        visitor_user = False
        if user_id:
            visitor_user = self.env['res.users'].browse(user_id)
            if visitor_user and visitor_user.active and visitor_user != operator:  # valid session user (not public)
                channel_partner_to_add.append(Command.create({'partner_id': visitor_user.partner_id.id}))
        return {
            'channel_last_seen_partner_ids': channel_partner_to_add,
            'livechat_active': True,
            'livechat_operator_id': operator_partner_id,
            'livechat_channel_id': self.id,
            'anonymous_name': False if user_id else anonymous_name,
            'country_id': country_id,
            'channel_type': 'livechat',
            'name': ' '.join([visitor_user.display_name if visitor_user else anonymous_name, operator.livechat_username if operator.livechat_username else operator.name]),
            'public': 'private',
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
            LEFT OUTER JOIN mail_message m ON c.id = m.res_id AND m.model = 'mail.channel'
            WHERE c.channel_type = 'livechat'
            AND c.livechat_operator_id in %s
            AND m.create_date > ((now() at time zone 'UTC') - interval '30 minutes')
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
            'header_background_color': self.header_background_color,
            'button_background_color': self.button_background_color,
            'title_color': self.title_color,
            'button_text_color': self.button_text_color,
            'button_text': self.button_text,
            'input_placeholder': self.input_placeholder,
            'default_message': self.default_message,
            "channel_name": self.name,
            "channel_id": self.id,
        }

    def get_livechat_info(self, username=None):
        self.ensure_one()

        if username is None:
            username = _('Visitor')
        info = {}
        info['available'] = len(self._get_available_users()) > 0
        info['server_url'] = self.get_base_url()
        if info['available']:
            info['options'] = self._get_channel_infos()
            info['options']['current_partner_id'] = self.env.user.partner_id.id
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

## File: models\mail_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools import html_escape


class MailChannel(models.Model):
    """ Chat Session
        Reprensenting a conversation between users.
        It extends the base method for anonymous usage.
    """

    _name = 'mail.channel'
    _inherit = ['mail.channel', 'rating.mixin']

    anonymous_name = fields.Char('Anonymous Name')
    channel_type = fields.Selection(selection_add=[('livechat', 'Livechat Conversation')])
    livechat_active = fields.Boolean('Is livechat ongoing?', help='Livechat session is active until visitor leave the conversation.')
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
        notifications = super()._channel_message_notifications(message=message, message_format=message_format)
        for channel in self:
            # add uuid for private livechat channels to allow anonymous to listen
            if channel.channel_type == 'livechat' and channel.public == 'private':
                notifications.append([channel.uuid, 'mail.channel/new_message', notifications[0][2]])
        if not message.author_id:
            unpinned_channel_partner = self.channel_last_seen_partner_ids.filtered(lambda cp: not cp.is_pinned)
            if unpinned_channel_partner:
                unpinned_channel_partner.write({'is_pinned': True})
                notifications = self._channel_channel_notifications(unpinned_channel_partner.mapped('partner_id').ids) + notifications
        return notifications

    def channel_info(self):
        """ Extends the channel header by adding the livechat operator and the 'anonymous' profile
            :rtype : list(dict)
        """
        channel_infos = super().channel_info()
        channel_infos_dict = dict((c['id'], c) for c in channel_infos)
        for channel in self:
            # add the last message date
            if channel.channel_type == 'livechat':
                # add the operator id
                if channel.livechat_operator_id:
                    display_name = channel.livechat_operator_id.user_livechat_username or channel.livechat_operator_id.display_name
                    channel_infos_dict[channel.id]['operator_pid'] = (channel.livechat_operator_id.id, display_name.replace(',', ''))
                # add the anonymous or partner name
                channel_infos_dict[channel.id]['livechat_visitor'] = channel._channel_get_livechat_visitor_info()
        return list(channel_infos_dict.values())

    def _channel_info_format_member(self, partner, partner_info):
        """Override to remove sensitive information in livechat."""
        if self.channel_type == 'livechat':
            return {
                'active': partner.active,
                'id': partner.id,
                'name': partner.user_livechat_username or partner.name,  # for API compatibility in stable
                'email': False,  # for API compatibility in stable
                'im_status': False,  # for API compatibility in stable
                'livechat_username': partner.user_livechat_username,
            }
        return super()._channel_info_format_member(partner=partner, partner_info=partner_info)

    def _notify_typing_partner_data(self):
        """Override to remove name and return livechat username if applicable."""
        data = super()._notify_typing_partner_data()
        if self.channel_type == 'livechat' and self.env.user.partner_id.user_livechat_username:
            data['partner_name'] = self.env.user.partner_id.user_livechat_username  # for API compatibility in stable
            data['livechat_username'] = self.env.user.partner_id.user_livechat_username
        return data

    def _channel_get_livechat_visitor_info(self):
        self.ensure_one()
        # remove active test to ensure public partner is taken into account
        channel_partner_ids = self.with_context(active_test=False).channel_partner_ids
        partners = channel_partner_ids - self.livechat_operator_id
        if not partners:
            # operator probably testing the livechat with his own user
            partners = channel_partner_ids
        first_partner = partners and partners[0]
        if first_partner and (not first_partner.user_ids or not any(user._is_public() for user in first_partner.user_ids)):
            # legit non-public partner
            return {
                'country': first_partner.country_id.name_get()[0] if first_partner.country_id else False,
                'id': first_partner.id,
                'name': first_partner.name,
            }
        return {
            'country': self.country_id.name_get()[0] if self.country_id else False,
            'id': False,
            'name': self.anonymous_name or _("Visitor"),
        }

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

    @api.autovacuum
    def _gc_empty_livechat_sessions(self):
        hours = 1  # never remove empty session created within the last hour
        self.env.cr.execute("""
            SELECT id as id
            FROM mail_channel C
            WHERE NOT EXISTS (
                SELECT *
                FROM mail_message M
                WHERE M.res_id = C.id AND m.model = 'mail.channel'
            ) AND C.channel_type = 'livechat' AND livechat_channel_id IS NOT NULL AND
                COALESCE(write_date, create_date, (now() at time zone 'UTC'))::timestamp
                < ((now() at time zone 'UTC') - interval %s)""", ("%s hours" % hours,))
        empty_channel_ids = [item['id'] for item in self.env.cr.dictfetchall()]
        self.browse(empty_channel_ids).unlink()

    def _execute_command_help_message_extra(self):
        msg = super(MailChannel, self)._execute_command_help_message_extra()
        return msg + _("Type <b>:shortcut</b> to insert a canned response in your message.<br>")

    def execute_command_history(self, **kwargs):
        self.env['bus.bus']._sendone(self.uuid, 'im_livechat.history_command', {'id': self.id})

    def _send_history_message(self, pid, page_history):
        message_body = _('No history found')
        if page_history:
            html_links = ['<li><a href="%s" target="_blank">%s</a></li>' % (html_escape(page), html_escape(page)) for page in page_history]
            message_body = '<ul>%s</ul>' % (''.join(html_links))
        self._send_transient_message(self.env['res.partner'].browse(pid), message_body)

    def _message_update_content_after_hook(self, message):
        self.ensure_one()
        if self.channel_type == 'livechat':
            self.env['bus.bus']._sendone(self.uuid, 'mail.message/insert', {
                'id': message.id,
                'body': message.body,
            })
        super()._message_update_content_after_hook(message=message)

    def _get_visitor_leave_message(self, operator=False, cancel=False):
        return _('Visitor has left the conversation.')

    def _close_livechat_session(self, **kwargs):
        """ Set deactivate the livechat channel and notify (the operator) the reason of closing the session."""
        self.ensure_one()
        if self.livechat_active:
            self.livechat_active = False
            # avoid useless notification if the channel is empty
            if not self.message_ids:
                return
            # Notify that the visitor has left the conversation
            self.message_post(author_id=self.env.ref('base.partner_root').id,
                              body=self._get_visitor_leave_message(**kwargs), message_type='comment', subtype_xmlid='mail.mt_comment')

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
        mail_body = template._render(render_context, engine='ir.qweb', minimal_qcontext=True)
        mail_body = self.env['mail.render.mixin']._replace_local_links(mail_body)
        mail = self.env['mail.mail'].sudo().create({
            'subject': _('Conversation with %s', self.livechat_operator_id.user_livechat_username or self.livechat_operator_id.name),
            'email_from': company.catchall_formatted or company.email_formatted,
            'author_id': self.env.user.partner_id.id,
            'email_to': email,
            'body_html': mail_body,
        })
        mail.send()

```

## File: models\mail_channel_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ChannelPartner(models.Model):
    _inherit = 'mail.channel.partner'

    @api.autovacuum
    def _gc_unpin_livechat_sessions(self):
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

```

## File: models\mail_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MailMessage(models.Model):
    _inherit = 'mail.message'

    def _message_format(self, fnames, format_reply=True):
        """Override to remove email_from and to return the livechat username if applicable.
        A third param is added to the author_id tuple in this case to be able to differentiate it
        from the normal name in client code."""
        vals_list = super()._message_format(fnames=fnames, format_reply=format_reply)
        for vals in vals_list:
            message_sudo = self.browse(vals['id']).sudo().with_prefetch(self.ids)
            if message_sudo.model == 'mail.channel' and self.env['mail.channel'].browse(message_sudo.res_id).channel_type == 'livechat':
                vals.pop('email_from')
                if message_sudo.author_id.user_livechat_username:
                    vals['author_id'] = (message_sudo.author_id.id, message_sudo.author_id.user_livechat_username, message_sudo.author_id.user_livechat_username)
        return vals_list

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

from odoo import api, models, fields


class Partners(models.Model):
    """Update of res.partner class to take into account the livechat username."""
    _inherit = 'res.partner'

    user_livechat_username = fields.Char(compute='_compute_user_livechat_username')

    def _get_channels_as_member(self):
        channels = super()._get_channels_as_member()
        channels |= self.env['mail.channel'].search([
            ('channel_type', '=', 'livechat'),
            ('channel_last_seen_partner_ids', 'in', self.env['mail.channel.partner'].sudo()._search([
                ('partner_id', '=', self.id),
                ('is_pinned', '=', True),
            ])),
        ])
        return channels

    @api.depends('user_ids.livechat_username')
    def _compute_user_livechat_username(self):
        for partner in self:
            partner.user_livechat_username = next(iter(partner.user_ids.mapped('livechat_username')), False)

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

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + ['livechat_username']

    @property
    def SELF_WRITEABLE_FIELDS(self):
        return super().SELF_WRITEABLE_FIELDS + ['livechat_username']

```

## File: models\res_users_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResUsersSettings(models.Model):
    _inherit = 'res.users.settings'

    is_discuss_sidebar_category_livechat_open = fields.Boolean("Is category livechat open", default=True)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*

from . import res_users
from . import res_partner
from . import im_livechat_channel
from . import mail_channel
from . import mail_channel_partner
from . import mail_message
from . import res_users_settings
from . import rating
from . import digest

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
                        WHEN EXISTS (select distinct M.author_id FROM mail_message M
                                        WHERE M.author_id=C.livechat_operator_id
                                        AND M.res_id = C.id
                                        AND M.model = 'mail.channel'
                                        AND C.livechat_operator_id = M.author_id)
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
                        WHEN rate.rating = 5 THEN 1
                        ELSE 0
                    END as is_happy,
                    Rate.rating as rating,
                    CASE
                        WHEN Rate.rating = 1 THEN 'Unhappy'
                        WHEN Rate.rating = 5 THEN 'Happy'
                        WHEN Rate.rating = 3 THEN 'Neutral'
                        ELSE null
                    END as rating_text,
                    CASE 
                        WHEN rate.rating > 0 THEN 0
                        ELSE 1
                    END as is_unrated,
                    C.livechat_operator_id as partner_id
                FROM mail_channel C
                    JOIN mail_message M ON (M.res_id = C.id AND M.model = 'mail.channel')
                    JOIN im_livechat_channel L ON (L.id = C.livechat_channel_id)
                    LEFT JOIN mail_message MO ON (MO.res_id = C.id AND MO.model = 'mail.channel' AND MO.author_id = C.livechat_operator_id)
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
                <pivot string="Livechat Support Statistics" disable_linking="1" sample="1">
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
                <graph string="Livechat Support Statistics" sample="1" disable_linking="1">
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
                    <separator/>
                    <filter name="filter_start_date" date="start_date"/>
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

        <record id="im_livechat_report_channel_time_to_answer_action" model="ir.actions.act_window">
            <field name="name">Session Statistics</field>
            <field name="res_model">im_livechat.report.channel</field>
            <field name="view_mode">graph,pivot</field>
            <field name="context">{"graph_measure": "time_to_answer", "search_default_last_week":1}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No data yet!
                </p>
            </field>
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
                    JOIN mail_message M ON M.res_id = C.id AND M.model = 'mail.channel'
                    LEFT JOIN mail_message MO ON (MO.res_id = C.id AND MO.model = 'mail.channel' AND MO.author_id = C.livechat_operator_id)
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
                <pivot string="Livechat Support Statistics" disable_linking="1" sample="1">
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
                <graph string="Livechat Support Statistics" sample="1" disable_linking="1">
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
                    <separator/>
                    <filter name="filter_start_date" date="start_date"/>
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

    <data noupdate="1">
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

## File: static\src\components\composer\composer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-inherit="mail.Composer" t-inherit-mode="extension">
        <xpath expr="//*[hasclass('o_Composer_buttonAttachment')]" position="replace">
            <t t-if="!composerView.composer.activeThread or composerView.composer.activeThread.channel_type !== 'livechat'">$0</t>
        </xpath>
    </t>

</templates>

```

## File: static\src\components\discuss\discuss.js

```javascript
/** @odoo-module **/

import { Discuss } from '@mail/components/discuss/discuss';

import { patch } from 'web.utils';

const components = { Discuss };

patch(components.Discuss.prototype, 'im_livechat/static/src/components/discuss/discuss.js', {

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    mobileNavbarTabs(...args) {
        return [...this._super(...args), {
            icon: 'fa fa-comments',
            id: 'livechat',
            label: this.env._t("Livechat"),
        }];
    }

});

```

## File: static\src\components\discuss_sidebar\discuss_sidebar.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.DiscussSidebar" t-inherit-mode="extension">
        <xpath expr="//DiscussSidebarCategory[contains(@class, 'o_DiscussSidebar_categoryChat')]" position="before">
            <t t-set="categoryLivechat" t-value="messaging.discuss.categoryLivechat"/>
            <t t-if="categoryLivechat and categoryLivechat.categoryItems.length">
                <DiscussSidebarCategory
                    class="o_DiscussSidebar_category o_DiscussSidebar_categoryLivechat"
                    categoryLocalId="categoryLivechat.localId"
                />
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\components\notification_list\notification_list.js

```javascript
/** @odoo-module **/

import { NotificationList } from '@mail/components/notification_list/notification_list';
import { patch } from 'web.utils';

const components = { NotificationList };

components.NotificationList._allowedFilters.push('livechat');

patch(components.NotificationList.prototype, 'im_livechat/static/src/components/notification_list/notification_list.js', {

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Override to include livechat channels.
     *
     * @override
     */
    _getThreads(props) {
        if (props.filter === 'livechat') {
            return this.messaging.models['mail.thread'].all(thread =>
                thread.channel_type === 'livechat' &&
                thread.isPinned &&
                thread.model === 'mail.channel'
            );
        }
        return this._super(...arguments);
    },

});

```

## File: static\src\components\thread_icon\thread_icon.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.ThreadIcon" t-inherit-mode="extension">
        <xpath expr="//*[@name='rootCondition']" position="inside">
            <t t-elif="thread.channel_type === 'livechat'">
                <t t-if="thread.orderedOtherTypingMembers.length > 0">
                    <ThreadTypingIcon
                        class="o_ThreadIcon_typing"
                        animation="'pulse'"
                        title="thread.typingStatusText"
                    />
                </t>
                <t t-else="">
                    <div class="fa fa-comments" title="Livechat"/>
                </t>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\components\thread_needaction_preview\thread_needaction_preview.js

```javascript
/** @odoo-module **/

import { ThreadNeedactionPreview } from '@mail/components/thread_needaction_preview/thread_needaction_preview';

import { patch } from 'web.utils';

const components = { ThreadNeedactionPreview };

patch(components.ThreadNeedactionPreview.prototype, 'thread_needaction_preview', {

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    image(...args) {
        if (this.thread.channel_type === 'livechat') {
            return '/mail/static/src/img/smiley/avatar.jpg';
        }
        return this._super(...args);
    }

});

```

## File: static\src\components\thread_preview\thread_preview.js

```javascript
/** @odoo-module **/

import { ThreadPreview } from '@mail/components/thread_preview/thread_preview';

import { patch } from 'web.utils';

const components = { ThreadPreview };

patch(components.ThreadPreview.prototype, 'im_livechat/static/src/components/thread_preview/thread_preview.js', {

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    image(...args) {
        if (this.thread.channel_type === 'livechat') {
            return '/mail/static/src/img/smiley/avatar.jpg';
        }
        return this._super(...args);
    }

});

```

## File: static\src\js\ajax_external.js

```javascript
/** @odoo-module **/

import ajax from 'web.ajax';

/**
  * This file should be used in the context of an external widget loading (e.g: live chat in a non-Odoo website)
  * It overrides the 'loadJS' method that is supposed to load additional scripts, based on a relative URL (e.g: '/web/webclient/locale/en_US')
  * As we're not in an Odoo website context, the calls will not work, and we avoid a 404 request.
  */
ajax.loadJS = function (url) {
    console.log('Tried to load the following script on an external website: ' + url);
};

```

## File: static\src\js\im_livechat_channel_form_controller.js

```javascript
/** @odoo-module **/

import FormController from 'web.FormController';

const ImLivechatChannelFormController = FormController.extend({
    events: Object.assign({}, FormController.prototype.events, {
        'click .o_im_livechat_channel_form_button_colors_reset_button': '_onClickLivechatButtonColorsResetButton',
        'click .o_im_livechat_channel_form_chat_window_colors_reset_button': '_onClickLivechatChatWindowColorsResetButton',
    }),

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Object} colorValues
     */
    async _updateColors(colorValues) {
        for (const name in colorValues) {
            this.$(`[name="${name}"] .o_field_color`).css('background-color', colorValues[name]);
        }
        const result = await this.model.notifyChanges(this.handle, colorValues);
        this._updateRendererState(this.model.get(this.handle), { fieldNames: result });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    async _onClickLivechatButtonColorsResetButton() {
        await this._updateColors({
            button_background_color: "#878787",
            button_text_color: "#FFFFFF",
        });
    },
    /**
     * @private
     */
    async _onClickLivechatChatWindowColorsResetButton() {
        await this._updateColors({
            header_background_color: "#875A7B",
            title_color: "#FFFFFF",
        });
    },
});

export default ImLivechatChannelFormController;

```

## File: static\src\js\im_livechat_channel_form_view.js

```javascript
/** @odoo-module **/

import ImLivechatChannelFormController from '@im_livechat/js/im_livechat_channel_form_controller';

import FormView from 'web.FormView';
import viewRegistry from 'web.view_registry';

const ImLivechatChannelFormView = FormView.extend({
    config: Object.assign({}, FormView.prototype.config, {
        Controller: ImLivechatChannelFormController,
    }),
});

viewRegistry.add('im_livechat_channel_form_view_js', ImLivechatChannelFormView);

export default ImLivechatChannelFormView;

```

## File: static\src\legacy\public_livechat.js

```javascript
odoo.define('im_livechat.legacy.im_livechat.im_livechat', function (require) {
"use strict";

require('bus.BusService');
var concurrency = require('web.concurrency');
var config = require('web.config');
var core = require('web.core');
var session = require('web.session');
var time = require('web.time');
var utils = require('web.utils');
var Widget = require('web.Widget');

var WebsiteLivechat = require('im_livechat.legacy.im_livechat.model.WebsiteLivechat');
var WebsiteLivechatMessage = require('im_livechat.legacy.im_livechat.model.WebsiteLivechatMessage');
var WebsiteLivechatWindow = require('im_livechat.legacy.im_livechat.WebsiteLivechatWindow');

var _t = core._t;
var QWeb = core.qweb;

// Constants
var LIVECHAT_COOKIE_HISTORY = 'im_livechat_history';
var HISTORY_LIMIT = 15;

var RATING_TO_EMOJI = {
    "5": "😊",
    "3": "😐",
    "1": "😞"
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
    utils.set_cookie(LIVECHAT_COOKIE_HISTORY, JSON.stringify(urlHistory), 60 * 60 * 24); // 1 day cookie
}

var LivechatButton = Widget.extend({
    className: 'openerp o_livechat_button d-print-none',
    custom_events: {
        'close_chat_window': '_onCloseChatWindow',
        'post_message_chat_window': '_onPostMessageChatWindow',
        'save_chat_window': '_onSaveChatWindow',
        'updated_typing_partners': '_onUpdatedTypingPartners',
        'updated_unread_counter': '_onUpdatedUnreadCounter',
    },
    events: {
        'click': '_openChat'
    },
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
    async willStart() {
        var cookie = utils.get_cookie('im_livechat_session');
        if (cookie) {
            var channel = JSON.parse(cookie);
            this._history = await session.rpc('/mail/chat_history', {uuid: channel.uuid, limit: 100});
            this._history.reverse().forEach(message => { message.body = utils.Markup(message.body); });
        } else {
            const result = await session.rpc('/im_livechat/init', {channel_id: this.options.channel_id});
            if (!result.available_for_me) {
                return Promise.reject();
            }
            this._rule = result.rule;
        }
        const proms = [this._loadQWebTemplate()];
        // Translations are already loaded in Odoo frontend. Only load them if
        // we are in an external page.
        if (!session.is_frontend) {
            proms.push(session.load_translations(["im_livechat"]));
        }
        return Promise.all(proms);
    },
    start: function () {
        this.$el.text(this.options.button_text);
        if (this._history) {
            this._history.forEach(m => this._addMessage(m));
            this._openChat();
        } else if (!config.device.isMobile && this._rule.action === 'auto_popup') {
            var autoPopupCookie = utils.get_cookie('im_livechat_auto_popup');
            if (!autoPopupCookie || JSON.parse(autoPopupCookie)) {
                this._autoPopupTimeout =
                    setTimeout(this._openChat.bind(this), this._rule.auto_popup_timer * 1000);
            }
        }
        this.call('bus_service', 'onNotification', this, this._onNotification);
        if (this.options.button_background_color) {
            this.$el.css('background-color', this.options.button_background_color);
        }
        if (this.options.button_text_color) {
            this.$el.css('color', this.options.button_text_color);
        }

        // If website_event_track installed, put the livechat banner above the PWA banner.
        var pwaBannerHeight = $('.o_pwa_install_banner').outerHeight(true);
        if (pwaBannerHeight) {
            this.$el.css('bottom', pwaBannerHeight + 'px');
        }

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
    _closeChat: function () {
        this._chatWindow.destroy();
        utils.set_cookie('im_livechat_session', "", -1); // remove cookie
    },
    /**
     * @private
     * @param {Object} notification
     * @param {Object} notification.payload
     * @param {string} notification.type
     */
    _handleNotification: function ({ payload, type }) {
        switch (type) {
            case 'im_livechat.history_command': {
                if (payload.id !== this._livechat._id) {
                    return;
                }
                const cookie = utils.get_cookie(LIVECHAT_COOKIE_HISTORY);
                const history = cookie ? JSON.parse(cookie) : [];
                session.rpc('/im_livechat/history', {
                    pid: this._livechat.getOperatorPID()[0],
                    channel_uuid: this._livechat.getUUID(),
                    page_history: history,
                });
                return;
            }
            case 'mail.channel.partner/typing_status': {
                if (payload.channel_id !== this._livechat._id) {
                    return;
                }
                const partnerID = payload.partner_id;
                if (partnerID === this.options.current_partner_id) {
                    // ignore typing display of current partner.
                    return;
                }
                if (payload.is_typing) {
                    this._livechat.registerTyping({ partnerID });
                } else {
                    this._livechat.unregisterTyping({ partnerID });
                }
                return;
            }
            case 'mail.channel/new_message': {
                if (payload.id !== this._livechat._id) {
                    return;
                }
                const notificationData = payload.message;
                // If message from notif is already in chatter messages, stop handling
                if (this._messages.some(message => message.getID() === notificationData.id)) {
                    return;
                }
                notificationData.body = utils.Markup(notificationData.body);
                this._addMessage(notificationData);
                if (this._chatWindow.isFolded() || !this._chatWindow.isAtBottom()) {
                    this._livechat.incrementUnreadCounter();
                }
                this._renderMessages();
                return;
            }
            case 'mail.message/insert': {
                const message = this._messages.find(message => message._id === payload.id);
                if (!message) {
                    return;
                }
                message._body = utils.Markup(payload.body);
                this._renderMessages()
                return;
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
                channel_id: this.options.channel_id,
                anonymous_name: this.options.default_username,
                previous_operator_id: this._get_previous_operator_id(),
            }, { shadow: true });
        }
        def.then(function (livechatData) {
            if (!livechatData || !livechatData.operator_pid) {
                try {
                    self.displayNotification({
                        message: _t("No available collaborator, please try again later."),
                        sticky: true,
                    });
                } catch (err) {
                    /**
                     * Failure in displaying notification happens when
                     * notification service doesn't exist, which is the case in
                     * external lib. We don't want notifications in external
                     * lib at the moment because they use bootstrap toast and
                     * we don't want to include boostrap in external lib.
                     */
                    console.warn(_t("No available collaborator, please try again later."));
                }
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

                    utils.set_cookie('im_livechat_session', utils.unaccent(JSON.stringify(self._livechat.toData()), true), 60 * 60);
                    utils.set_cookie('im_livechat_auto_popup', JSON.stringify(false), 60 * 60);
                    if (livechatData.operator_pid[0]) {
                        // livechatData.operator_pid contains a tuple (id, name)
                        // we are only interested in the id
                        var operatorPidId = livechatData.operator_pid[0];
                        var oneWeek = 7 * 24 * 60 * 60;
                        utils.set_cookie('im_livechat_previous_operator_pid', operatorPidId, oneWeek);
                    }
                });
            }
        }).then(function () {
            self._openingChat = false;
        }).guardedCatch(function () {
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
            headerBackgroundColor: this.options.header_background_color,
            placeholder: this.options.input_placeholder || "",
            titleColor: this.options.title_color,
        };
        this._chatWindow = new WebsiteLivechatWindow(this, this._livechat, options);
        return this._chatWindow.appendTo($('body')).then(function () {
            var cssProps = { bottom: 0 };
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
        this._livechat._notifyMyselfTyping({ typing: false });
        return session
            .rpc('/mail/chat_post', { uuid: this._livechat.getUUID(), message_content: message.content })
            .then(function (messageId) {
                if (!messageId) {
                    try {
                        self.displayNotification({
                            message: _t("Session expired... Please refresh and try again."),
                            sticky: true,
                        });
                    } catch (err) {
                        /**
                         * Failure in displaying notification happens when
                         * notification service doesn't exist, which is the case
                         * in external lib. We don't want notifications in
                         * external lib at the moment because they use bootstrap
                         * toast and we don't want to include boostrap in
                         * external lib.
                         */
                        console.warn(_t("Session expired... Please refresh and try again."));
                    }
                    self._closeChat();
                }
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
                date: time.datetime_to_str(new Date()),
                model: "mail.channel",
                res_id: this._livechat.getID(),
            }, { prepend: true });
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
        this._visitorLeaveSession();
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
        utils.set_cookie('im_livechat_session', utils.unaccent(JSON.stringify(this._livechat.toData()), true), 60 * 60);
    },
    /**
     * @private
     * @param {OdooEvent} ev
     */
    _onUpdatedTypingPartners(ev) {
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
    /**
     * @private
     * Called when the visitor leaves the livechat chatter the first time (first click on X button)
     * this will deactivate the mail_channel, notify operator that visitor has left the channel.
     */
    _visitorLeaveSession: function () {
        var cookie = utils.get_cookie('im_livechat_session');
        if (cookie) {
            var channel = JSON.parse(cookie);
            session.rpc('/im_livechat/visitor_leave_session', {uuid: channel.uuid});
            utils.set_cookie('im_livechat_session', "", -1); // remove cookie
        }
    },
});

/*
 * Rating for Livechat
 *
 * This widget displays the 3 rating smileys, and a textarea to add a reason
 * (only for red smiley), and sends the user feedback to the server.
 */
var Feedback = Widget.extend({
    template: 'im_livechat.legacy.im_livechat.FeedBack',

    events: {
        'click .o_livechat_rating_choices img': '_onClickSmiley',
        'click .o_livechat_no_feedback span': '_onClickNoFeedback',
        'click .o_rating_submit_button': '_onClickSend',
        'click .o_email_chat_button': '_onEmailChat',
        'click .o_livechat_email_error .alert-link': '_onTryAgain',
    },

    /**
     * @param {?} parent
     * @param {im_livechat.legacy.im_livechat.model.WebsiteLivechat} livechat
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
        this.dp.add(session.rpc('/im_livechat/feedback', args)).then(function (response) {
            var emoji = RATING_TO_EMOJI[self.rating] || "??";
            if (!reason) {
                var content = _.str.sprintf(_t("Rating: %s"), emoji);
            }
            else {
                var content = "Rating reason: \n" + reason;
            }
            self.trigger('send_message', { content: content, isFeedback: true });
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
        this.$('.o_livechat_rating_choices img[data-value="' + this.rating + '"]').addClass('selected');

        // only display textearea if bad smiley selected
        if (this.rating !== 5) {
            this._sendFeedback();
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

odoo.define('im_livechat.legacy.im_livechat.model.WebsiteLivechat', function (require) {
"use strict";

var AbstractThread = require('im_livechat.legacy.mail.model.AbstractThread');
var ThreadTypingMixin = require('im_livechat.legacy.mail.model.ThreadTypingMixin');

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
     * @param {im_livechat.legacy.im_livechat.im_livechat:LivechatButton} params.parent
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
     * @returns {im_livechat.legacy.im_livechat.model.WebsiteLivechatMessage[]}
     */
    getMessages: function () {
        // ignore removed messages
        return this._messages.filter(message => !message.isEmpty());
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
     * @param {im_livechat.legacy.im_livechat.model.WebsiteLivechatMessage[]} messages
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

odoo.define('im_livechat.legacy.im_livechat.model.WebsiteLivechatMessage', function (require) {
"use strict";

var AbstractMessage = require('im_livechat.legacy.mail.model.AbstractMessage');

/**
 * This is a message that is handled by im_livechat, without making use of the
 * mail.Manager. The purpose of this is to make im_livechat compatible with
 * mail.widget.Thread.
 *
 * @see im_livechat.legacy.mail.model.AbstractMessage for more information.
 */
var WebsiteLivechatMessage = AbstractMessage.extend({

    /**
     * @param {im_livechat.legacy.im_livechat.im_livechat.LivechatButton} parent
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
            source += '/web/image/res.partner/' + this.getAuthorID() + '/avatar_128';
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

odoo.define('im_livechat.legacy.im_livechat.WebsiteLivechatWindow', function (require) {
"use strict";

var AbstractThreadWindow = require('im_livechat.legacy.mail.AbstractThreadWindow');

/**
 * This is the widget that represent windows of livechat in the frontend.
 *
 * @see im_livechat.legacy.mail.AbstractThreadWindow for more information
 */
var LivechatWindow = AbstractThreadWindow.extend({
    events: _.extend(AbstractThreadWindow.prototype.events, {
        'input .o_composer_text_field': '_onInput',
    }),
    /**
     * @override
     * @param {im_livechat.legacy.im_livechat.im_livechat:LivechatButton} parent
     * @param {im_livechat.legacy.im_livechat.model.WebsiteLivechat} thread
     * @param {Object} [options={}]
     * @param {string} [options.headerBackgroundColor]
     * @param {string} [options.titleColor]
     */
    init(parent, thread, options = {}) {
        this._super.apply(this, arguments);
        this._thread = thread;
    },
    /**
     * @override
     * @return {Promise}
     */
    async start() {
        await this._super(...arguments);
        if (this.options.headerBackgroundColor) {
            this.$('.o_thread_window_header').css('background-color', this.options.headerBackgroundColor);
        }
        if (this.options.titleColor) {
            this.$('.o_thread_window_header').css('color', this.options.titleColor);
        }
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

odoo.define('im_livechat.legacy.mail.model.AbstractThread', function (require) {
"use strict";

var Class = require('web.Class');
var Mixins = require('web.mixins');

/**
 * Abstract thread is the super class of all threads, either backend threads
 * (which are compatible with mail service) or website livechats.
 *
 * Abstract threads contain abstract messages
 */
var AbstractThread = Class.extend(Mixins.EventDispatcherMixin, {
    /**
     * @param {Object} params
     * @param {Object} params.data
     * @param {integer|string} params.data.id the ID of this thread
     * @param {string} params.data.name the name of this thread
     * @param {string} [params.data.status=''] the status of this thread
     * @param {Object} params.parent Object with the event-dispatcher mixin
     *   (@see {web.mixins.EventDispatcherMixin})
     */
    init: function (params) {
        Mixins.EventDispatcherMixin.init.call(this, arguments);
        this.setParent(params.parent);

        this._folded = false; // threads are unfolded by default
        this._id = params.data.id;
        this._name = params.data.name;
        this._status = params.data.status || '';
        this._unreadCounter = 0; // amount of messages not yet been read
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Add a message to this thread.
     *
     * @param {im_livechat.legacy.mail.model.AbstractMessage} message
     */
    addMessage: function (message) {
        this._addMessage.apply(this, arguments);
        this.trigger('message_added', message);
    },
    /**
     * Updates the folded state of the thread
     *
     * @param {boolean} folded
     */
    fold: function (folded) {
        this._folded = folded;
    },
    /**
     * Get the ID of this thread
     *
     * @returns {integer|string}
     */
    getID: function () {
        return this._id;
    },
    /**
     * @abstract
     * @returns {im_livechat.legacy.mail.model.AbstractMessage[]}
     */
    getMessages: function () {},
    /**
     * Get the name of this thread. If the name of the thread has been created
     * by the user from an input, it may be escaped.
     *
     * @returns {string}
     */
    getName: function () {
        return this._name;
    },
    /**
     * Get the status of the thread (e.g. 'online', 'offline', etc.)
     *
     * @returns {string}
     */
    getStatus: function () {
        return this._status;
    },
    /**
     * Returns the title to display in thread window's headers.
     *
     * @returns {string} the name of the thread by default (see @getName)
     */
    getTitle: function () {
        return this.getName();
    },
    getType: function () {},
    /**
     * @returns {integer}
     */
    getUnreadCounter: function () {
        return this._unreadCounter;
    },
    /**
     * @returns {boolean}
     */
    hasMessages: function () {
        return !_.isEmpty(this.getMessages());
    },
    /**
     * States whether this thread is compatible with the 'seen' feature.
     * By default, threads do not have thsi feature active.
     * @see {im_livechat.legacy.mail.model.ThreadSeenMixin} to enable this feature on a thread.
     *
     * @returns {boolean}
     */
    hasSeenFeature: function () {
        return false;
    },
    /**
     * States whether this thread is folded or not.
     *
     * @return {boolean}
     */
    isFolded: function () {
        return this._folded;
    },
    /**
     * Mark the thread as read, which resets the unread counter to 0. This is
     * only performed if the unread counter is not 0.
     *
     * @returns {Promise}
     */
    markAsRead: function () {
        if (this._unreadCounter > 0) {
            return this._markAsRead();
        }
        return Promise.resolve();
    },
    /**
     * Post a message on this thread
     *
     * @returns {Promise} resolved with the message object to be sent to the
     *   server
     */
    postMessage: function () {
        return this._postMessage.apply(this, arguments)
                                .then(this.trigger.bind(this, 'message_posted'));
    },
    /**
     * Resets the unread counter of this thread to 0.
     */
    resetUnreadCounter: function () {
        this._unreadCounter = 0;
        this._warnUpdatedUnreadCounter();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Add a message to this thread.
     *
     * @abstract
     * @private
     * @param {im_livechat.legacy.mail.model.AbstractMessage} message
     */
    _addMessage: function (message) {},
    /**
     * Increments the unread counter of this thread by 1 unit.
     *
     * @private
     */
    _incrementUnreadCounter: function () {
        this._unreadCounter++;
    },
    /**
     * Mark the thread as read
     *
     * @private
     * @returns {Promise}
     */
    _markAsRead: function () {
        this.resetUnreadCounter();
        return Promise.resolve();
    },
    /**
     * Post a message on this thread
     *
     * @abstract
     * @private
     * @returns {Promise} resolved with the message object to be sent to the
     *   server
     */
    _postMessage: function () {
        return Promise.resolve();
    },
    /**
     * Warn views (e.g. discuss app, thread window, etc.) to update visually
     * the unread counter of this thread.
     *
     * @abstract
     * @private
     */
    _warnUpdatedUnreadCounter: function () {},
});

return AbstractThread;

});

odoo.define('im_livechat.legacy.mail.model.ThreadTypingMixin', function (require) {
"use strict";

var CCThrottleFunction = require('im_livechat.legacy.mail.model.CCThrottleFunction');
var Timer = require('im_livechat.legacy.mail.model.Timer');
var Timers = require('im_livechat.legacy.mail.model.Timers');

var core = require('web.core');

var _t = core._t;

/**
 * Mixin for enabling the "is typing..." notification on a type of thread.
 */
var ThreadTypingMixin = {
    // Default partner infos
    _DEFAULT_TYPING_PARTNER_ID: '_default',
    _DEFAULT_TYPING_PARTNER_NAME: 'Someone',

    /**
     * Initialize the internal data for typing feature on threads.
     *
     * Also listens on some internal events of the thread:
     *
     * - 'message_added': when a message is added, remove the author in the
     *     typing partners.
     * - 'message_posted': when a message is posted, let the user have the
     *     possibility to immediately notify if he types something right away,
     *     instead of waiting for a throttle behaviour.
     */
    init: function () {
        // Store the last "myself typing" status that has been sent to the
        // server. This is useful in order to not notify the same typing
        // status multiple times.
        this._lastNotifiedMyselfTyping = false;

        // Timer of current user that is typing a very long text. When the
        // receivers do not receive any typing notification for a long time,
        // they assume that the related partner is no longer typing
        // something (e.g. they have closed the browser tab).
        // This is a timer to let others know that we are still typing
        // something, so that they do not assume we stopped typing
        // something.
        this._myselfLongTypingTimer = new Timer({
            duration: 50 * 1000,
            onTimeout: this._onMyselfLongTypingTimeout.bind(this),
        });

        // Timer of current user that was currently typing something, but
        // there is no change on the input for several time. This is used
        // in order to automatically notify other users that we have stopped
        // typing something, due to making no changes on the composer for
        // some time.
        this._myselfTypingInactivityTimer = new Timer({
            duration: 5 * 1000,
            onTimeout: this._onMyselfTypingInactivityTimeout.bind(this),
        });

        // Timers of users currently typing in the thread. This is useful
        // in order to automatically unregister typing users when we do not
        // receive any typing notification after a long time. Timers are
        // internally indexed by partnerID. The current user is ignored in
        // this list of timers.
        this._othersTypingTimers = new Timers({
            duration: 60 * 1000,
            onTimeout: this._onOthersTypingTimeout.bind(this),
        });

        // Clearable and cancellable throttled version of the
        // `doNotifyMyselfTyping` method. (basically `notifyMyselfTyping`
        // with slight pre- and post-processing)
        // @see {mail.model.ResetableThrottleFunction}
        // This is useful when the user posts a message and types something
        // else: he must notify immediately that he is typing something,
        // instead of waiting for the throttle internal timer.
        this._throttleNotifyMyselfTyping = CCThrottleFunction({
            duration: 2.5 * 1000,
            func: this._onNotifyMyselfTyping.bind(this),
        });

        // This is used to track the order of registered partners typing
        // something, in order to display the oldest typing partners.
        this._typingPartnerIDs = [];

        this.on('message_added', this, this._onTypingMessageAdded);
        this.on('message_posted', this, this._onTypingMessagePosted);
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Get the text to display when some partners are typing something on the
     * thread:
     *
     * - single typing partner:
     *
     *   A is typing...
     *
     * - two typing partners:
     *
     *   A and B are typing...
     *
     * - three or more typing partners:
     *
     *   A, B and more are typing...
     *
     * The choice of the members name for display is not random: it displays
     * the user that have been typing for the longest time. Also, this function
     * is hard-coded to display at most 2 partners. This limitation comes from
     * how translation works in Odoo, for which unevaluated string cannot be
     * translated.
     *
     * @returns {string} list of members that are typing something on the thread
     *   (excluding the current user).
     */
    getTypingMembersToText: function () {
        var typingPartnerIDs = this._typingPartnerIDs;
        var typingMembers = _.filter(this._members, function (member) {
            return _.contains(typingPartnerIDs, member.id);
        });
        var sortedTypingMembers = _.sortBy(typingMembers, function (member) {
            return _.indexOf(typingPartnerIDs, member.id);
        });
        var displayableTypingMembers = sortedTypingMembers.slice(0, 3);

        if (displayableTypingMembers.length === 0) {
            return '';
        } else if (displayableTypingMembers.length === 1) {
            return _.str.sprintf(_t("%s is typing..."), displayableTypingMembers[0].name);
        } else if (displayableTypingMembers.length === 2) {
            return _.str.sprintf(_t("%s and %s are typing..."),
                                    displayableTypingMembers[0].name,
                                    displayableTypingMembers[1].name);
        } else {
            return _.str.sprintf(_t("%s, %s and more are typing..."),
                                    displayableTypingMembers[0].name,
                                    displayableTypingMembers[1].name);
        }
    },
    /**
     * Threads with this mixin have the typing notification feature
     *
     * @returns {boolean}
     */
    hasTypingNotification: function () {
        return true;
    },
    /**
     * Tells if someone other than current user is typing something on this
     * thread.
     *
     * @returns {boolean}
     */
    isSomeoneTyping: function () {
        return !(_.isEmpty(this._typingPartnerIDs));
    },
    /**
     * Register someone that is currently typing something in this thread.
     * If this is the current user that is typing something, don't do anything
     * (we do not have to display anything)
     *
     * This method is ignored if we try to register the current user.
     *
     * @param {Object} params
     * @param {integer} params.partnerID ID of the partner linked to the user
     *   currently typing something on the thread.
     */
    registerTyping: function (params) {
        if (this._isTypingMyselfInfo(params)) {
            return;
        }
        var partnerID = params.partnerID;
        this._othersTypingTimers.registerTimer({
            timeoutCallbackArguments: [partnerID],
            timerID: partnerID,
        });
        if (_.contains(this._typingPartnerIDs, partnerID)) {
            return;
        }
        this._typingPartnerIDs.push(partnerID);
        this._warnUpdatedTypingPartners();
    },
    /**
     * This method must be called when the user starts or stops typing something
     * in the composer of the thread.
     *
     * @param {Object} params
     * @param {boolean} params.typing tell whether the current is typing or not.
     */
    setMyselfTyping: function (params) {
        var typing = params.typing;
        if (this._lastNotifiedMyselfTyping === typing) {
            this._throttleNotifyMyselfTyping.cancel();
        } else {
            this._throttleNotifyMyselfTyping(params);
        }

        if (typing) {
            this._myselfTypingInactivityTimer.reset();
        } else {
            this._myselfTypingInactivityTimer.clear();
        }
    },
    /**
     * Unregister someone from currently typing something in this thread.
     *
     * @param {Object} params
     * @param {integer} params.partnerID ID of the partner related to the user
     *   that is currently typing something
     */
    unregisterTyping: function (params) {
        var partnerID = params.partnerID;
        this._othersTypingTimers.unregisterTimer({ timerID: partnerID });
        if (!_.contains(this._typingPartnerIDs, partnerID)) {
            return;
        }
        this._typingPartnerIDs = _.reject(this._typingPartnerIDs, function (id) {
            return id === partnerID;
        });
        this._warnUpdatedTypingPartners();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Tells whether the provided information on a partner is related to the
     * current user or not.
     *
     * @abstract
     * @private
     * @param {Object} params
     * @param {integer} params.partner ID of partner to check
     */
    _isTypingMyselfInfo: function (params) {
        return false;
    },
    /**
     * Notify to the server that the current user either starts or stops typing
     * something.
     *
     * @abstract
     * @private
     * @param {Object} params
     * @param {boolean} params.typing whether we are typing something or not
     * @returns {Promise} resolved if the server is notified, rejected
     *   otherwise
     */
    _notifyMyselfTyping: function (params) {
        return Promise.resolve();
    },
    /**
     * Warn views that the list of users that are currently typing on this
     * thread has been updated.
     *
     * @abstract
     * @private
     */
    _warnUpdatedTypingPartners: function () {},

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called when current user is typing something for a long time. In order
     * to not let other users assume that we are no longer typing something, we
     * must notify again that we are typing something.
     *
     * @private
     */
    _onMyselfLongTypingTimeout: function () {
        this._throttleNotifyMyselfTyping.clear();
        this._throttleNotifyMyselfTyping({ typing: true });
    },
    /**
     * Called when current user has something typed in the composer, but is
     * inactive for some time. In this case, he automatically notifies that he
     * is no longer typing something
     *
     * @private
     */
    _onMyselfTypingInactivityTimeout: function () {
        this._throttleNotifyMyselfTyping.clear();
        this._throttleNotifyMyselfTyping({ typing: false });
    },
    /**
     * Called by throttled version of notify myself typing
     *
     * Notify to the server that the current user either starts or stops typing
     * something. Remember last notified stuff from the server, and update
     * related typing timers.
     *
     * @private
     * @param {Object} params
     * @param {boolean} params.typing whether we are typing something or not.
     */
    _onNotifyMyselfTyping: function (params) {
        var typing = params.typing;
        this._lastNotifiedMyselfTyping = typing;
        this._notifyMyselfTyping(params);
        if (typing) {
            this._myselfLongTypingTimer.reset();
        } else {
            this._myselfLongTypingTimer.clear();
        }
    },
    /**
     * Called when current user do not receive a typing notification of someone
     * else typing for a long time. In this case, we assume that this person is
     * no longer typing something.
     *
     * @private
     * @param {integer} partnerID partnerID of the person we assume he is no
     *   longer typing something.
     */
    _onOthersTypingTimeout: function (partnerID) {
        this.unregisterTyping({ partnerID: partnerID });
    },
    /**
     * Called when a new message is added to the thread
     * On receiving a message from a typing partner, unregister this partner
     * from typing partners (otherwise, it will still display it until timeout).
     *
     * @private
     * @param {mail.model.AbstractMessage} message
     */
    _onTypingMessageAdded: function (message) {
        var partnerID = message.hasAuthor() ?
                        message.getAuthorID() :
                        this._DEFAULT_TYPING_PARTNER_ID;
        this.unregisterTyping({ partnerID: partnerID });
    },
    /**
     * Called when current user has posted a message on this thread.
     *
     * The current user receives the possibility to immediately notify the
     * other users if he is typing something else.
     *
     * Refresh the context for the current user to notify that he starts or
     * stops typing something. In other words, when this function is called and
     * then the current user types something, it immediately notifies the
     * server as if it is the first time he is typing something.
     *
     * @private
     */
    _onTypingMessagePosted: function () {
        this._lastNotifiedMyselfTyping = false;
        this._throttleNotifyMyselfTyping.clear();
        this._myselfLongTypingTimer.clear();
        this._myselfTypingInactivityTimer.clear();
    },
};

return ThreadTypingMixin;

});

odoo.define('im_livechat.legacy.mail.model.AbstractMessage', function (require) {
"use strict";

var mailUtils = require('@mail/js/utils');

var Class = require('web.Class');
var core = require('web.core');
var session = require('web.session');
var time = require('web.time');

var _t = core._t;

/**
 * This is an abstract class for modeling messages in JS.
 * The purpose of this interface is to make im_livechat compatible with
 * mail.widget.Thread, as this widget was designed to work with messages that
 * are instances of mail.model.Messages.
 *
 * Ideally, im_livechat should also handle mail.model.Message, but this is not
 * feasible for the moment, as mail.model.Message requires mail.Manager to work,
 * and this module should not leak outside of the backend, hence the use of
 * mail.model.AbstractMessage as a work-around.
 */
var AbstractMessage = Class.extend({

    /**
     * @param {Widget} parent
     * @param {Object} data
     * @param {Array} [data.attachment_ids=[]]
     * @param {Array} [data.author_id]
     * @param {string} [data.body = ""]
     * @param {string} [data.date] the server-format date time of the message.
     *   If not provided, use current date time for this message.
     * @param {integer} data.id
     * @param {boolean} [data.is_discussion = false]
     * @param {boolean} [data.is_notification = false]
     * @param {string} [data.message_type = undefined]
     */
    init: function (parent, data) {
        this._attachmentIDs = data.attachment_ids || [];
        this._body = data.body || "";
        // by default: current datetime
        this._date = data.date ? moment(time.str_to_datetime(data.date)) : moment();
        this._id = data.id;
        this._isDiscussion = data.is_discussion;
        this._isNotification = data.is_notification;
        this._serverAuthorID = data.author_id;
        this._type = data.message_type || undefined;

        this._processAttachmentURL();
        this._attachmentIDs.forEach(function (attachment) {
            attachment.filename = attachment.filename || attachment.name || _t("unnamed");
        });
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Get the list of files attached to this message.
     * Note that attachments are stored with server-format
     *
     * @return {Object[]}
     */
    getAttachments: function () {
        return this._attachmentIDs;
    },
    /**
     * Get the server ID (number) of the author of this message
     * If there are no author, return -1;
     *
     * @return {integer}
     */
    getAuthorID: function () {
        if (!this.hasAuthor()) {
            return -1;
        }
        return this._serverAuthorID[0];
    },
    /**
     * Threads do not have an im status by default
     *
     * @return {undefined}
     */
    getAuthorImStatus: function () {
        return undefined;
    },
    /**
     * Get the relative url of the avatar to display next to the message
     *
     * @abstract
     * @return {string}
     */
    getAvatarSource: function () {
        if (this.hasAuthor()) {
            return '/web/image/res.partner/' + this.getAuthorID() + '/avatar_128';
        }
    },
    /**
     * Get the body content of this message
     *
     * @return {string}
     */
    getBody: function () {
        return this._body;
    },
    /**
     * @return {moment}
     */
    getDate: function () {
        return this._date;
    },
    /**
     * Get the date day of this message
     *
     * @return {string}
     */
    getDateDay: function () {
        var date = this.getDate().format('YYYY-MM-DD');
        if (date === moment().format('YYYY-MM-DD')) {
            return _t("Today");
        } else if (date === moment().subtract(1, 'days').format('YYYY-MM-DD')) {
            return _t("Yesterday");
        }
        return this.getDate().format('LL');
    },
    /**
     * Get the name of the author, if there is an author of this message
     * If there are no author of this message, returns 'null'
     *
     * @return {string}
     */
    getDisplayedAuthor: function () {
        return this.hasAuthor() ? this._getAuthorName() : null;
    },
    /**
     * Get the server ID (number) of this message
     *
     * @override
     * @return {integer}
     */
    getID: function () {
        return this._id;
    },
    /**
     * Get the list of images attached to this message.
     * Note that attachments are stored with server-format
     *
     * @return {Object[]}
     */
    getImageAttachments: function () {
        return _.filter(this.getAttachments(), function (attachment) {
            return attachment.mimetype && attachment.mimetype.split('/')[0] === 'image';
        });
    },
    /**
     * Get the list of non-images attached to this message.
     * Note that attachments are stored with server-format
     *
     * @return {Object[]}
     */
    getNonImageAttachments: function () {
        return _.difference(this.getAttachments(), this.getImageAttachments());
    },
    /**
     * Gets the class to use as the notification icon.
     *
     * @returns {string}
     */
    getNotificationIcon() {
        if (!this.hasNotificationsError()) {
            return 'fa fa-envelope-o';
        }
        return 'fa fa-envelope';
    },
    /**
     * Gets the list of notifications of this message, in no specific order.
     * By default messages do not have notifications.
     *
     * @returns {Object[]}
     */
    getNotifications() {
        return [];
    },
    /**
     * Gets the text to display next to the notification icon.
     *
     * @returns {string}
     */
    getNotificationText() {
        return '';
    },
    /**
     * Get the time elapsed between sent message and now
     *
     * @return {string}
     */
    getTimeElapsed: function () {
        return mailUtils.timeFromNow(this.getDate());
    },
    /**
     * Get the type of message (e.g. 'comment', 'email', 'notification', ...)
     * By default, messages are of type 'undefined'
     *
     * @override
     * @return {string|undefined}
     */
    getType: function () {
        return this._type;
    },
    /**
     * State whether this message contains some attachments.
     *
     * @override
     * @return {boolean}
     */
    hasAttachments: function () {
        return this.getAttachments().length > 0;
    },
    /**
     * State whether this message has an author
     *
     * @return {boolean}
     */
    hasAuthor: function () {
        return !!(this._serverAuthorID && this._serverAuthorID[0]);
    },
    /**
     * State whether this message has an email of its sender.
     * By default, messages do not have any email of its sender.
     *
     * @return {string}
     */
    hasEmailFrom: function () {
        return false;
    },
    /**
     * State whether this image contains images attachments
     *
     * @return {boolean}
     */
    hasImageAttachments: function () {
        return _.some(this.getAttachments(), function (attachment) {
            return attachment.mimetype && attachment.mimetype.split('/')[0] === 'image';
        });
    },
    /**
     * State whether this image contains non-images attachments
     *
     * @return {boolean}
     */
    hasNonImageAttachments: function () {
        return _.some(this.getAttachments(), function (attachment) {
            return !(attachment.mimetype && attachment.mimetype.split('/')[0] === 'image');
        });
    },
    /**
     * States whether this message has some notifications.
     *
     * @returns {boolean}
     */
    hasNotifications() {
        return this.getNotifications().length > 0;
    },
    /**
     * States whether this message has notifications that are in error.
     *
     * @returns {boolean}
     */
    hasNotificationsError() {
        return this.getNotifications().some(notif =>
            notif.notification_status === 'exception' ||
            notif.notification_status === 'bounce'
        );
    },
    /**
     * State whether this message originates from a channel.
     * By default, messages do not originate from a channel.
     *
     * @override
     * @return {boolean}
     */
    originatesFromChannel: function () {
        return false;
    },
    /**
     * State whether this message has a subject
     * By default, messages do not have any subject.
     *
     * @return {boolean}
     */
    hasSubject: function () {
        return false;
    },
    /**
     * State whether this message is empty
     *
     * @return {boolean}
     */
    isEmpty: function () {
        return !this.hasTrackingValues() &&
        !this.hasAttachments() &&
        !this.getBody();
    },
    /**
     * By default, messages do not have any subtype description
     *
     * @return {boolean}
     */
    hasSubtypeDescription: function () {
        return false;
    },
    /**
     * State whether this message contains some tracking values
     * By default, messages do not have any tracking values.
     *
     * @return {boolean}
     */
    hasTrackingValues: function () {
        return false;
    },
    /**
     * State whether this message is a discussion
     *
     * @return {boolean}
     */
    isDiscussion: function () {
        return this._isDiscussion;
    },
    /**
     * State whether this message is linked to a document thread
     * By default, messages are not linked to a document thread.
     *
     * @return {boolean}
     */
    isLinkedToDocumentThread: function () {
        return false;
    },
    /**
     * State whether this message is needaction
     * By default, messages are not needaction.
     *
     * @return {boolean}
     */
    isNeedaction: function () {
        return false;
    },
    /**
     * State whether this message is a note (i.e. a message from "Log note")
     *
     * @return {boolean}
     */
    isNote: function () {
        return this._isNote;
    },
    /**
     * State whether this message is a notification
     *
     * User notifications are defined as either
     *      - notes
     *      - pushed to user Inbox or email through classic notification process
     *      - not linked to any document, meaning model and res_id are void
     *
     * This is useful in order to display white background for user
     * notifications in chatter
     *
     * @returns {boolean}
     */
    isNotification: function () {
        return this._isNotification;
    },
    /**
     * State whether this message is starred
     * By default, messages are not starred.
     *
     * @return {boolean}
     */
    isStarred: function () {
        return false;
    },
    /**
     * State whether this message is a system notification
     * By default, messages are not system notifications
     *
     * @override
     * @return {boolean}
     */
    isSystemNotification: function () {
        return false;
    },
    /**
     * States whether the current message needs moderation in general.
     * By default, messages do not require any moderation.
     *
     * @returns {boolean}
     */
    needsModeration: function () {
        return false;
    },
    /**
     * @params {integer[]} attachmentIDs
     */
    removeAttachments: function (attachmentIDs) {
        this._attachmentIDs = _.reject(this._attachmentIDs, function (attachment) {
            return _.contains(attachmentIDs, attachment.id);
        });
    },
    /**
     * State whether this message should redirect to the author
     * when clicking on the author of this message.
     *
     * Do not redirect on author clicked of self-posted messages.
     *
     * @return {boolean}
     */
    shouldRedirectToAuthor: function () {
        return !this._isMyselfAuthor();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Get the name of the author of this message.
     * If there are no author of this messages, returns '' (empty string).
     *
     * @private
     * @returns {string}
     */
    _getAuthorName: function () {
        if (!this.hasAuthor()) {
            return "";
        }
        return this._serverAuthorID[1];
    },
    /**
     * State whether the current user is the author of this message
     *
     * @private
     * @return {boolean}
     */
    _isMyselfAuthor: function () {
        return this.hasAuthor() && (this.getAuthorID() === session.partner_id);
    },
    /**
     * Compute url of attachments of this message
     *
     * @private
     */
    _processAttachmentURL: function () {
        _.each(this.getAttachments(), function (attachment) {
            attachment.url = '/web/content/' + attachment.id + '?download=true';
        });
    },

});

return AbstractMessage;

});

odoo.define('im_livechat.legacy.mail.AbstractThreadWindow', function (require) {
"use strict";

var ThreadWidget = require('im_livechat.legacy.mail.widget.Thread');

var config = require('web.config');
var core = require('web.core');
var Widget = require('web.Widget');

var QWeb = core.qweb;
var _t = core._t;

/**
 * This is an abstract widget for rendering thread windows.
 * AbstractThreadWindow is kept for legacy reasons.
 */
var AbstractThreadWindow = Widget.extend({
    template: 'im_livechat.legacy.mail.AbstractThreadWindow',
    custom_events: {
        document_viewer_closed: '_onDocumentViewerClose',
    },
    events: {
        'click .o_thread_window_close': '_onClickClose',
        'click .o_thread_window_title': '_onClickFold',
        'click .o_composer_text_field': '_onComposerClick',
        'click .o_mail_thread': '_onThreadWindowClicked',
        'keydown .o_composer_text_field': '_onKeydown',
        'keypress .o_composer_text_field': '_onKeypress',
    },
    FOLD_ANIMATION_DURATION: 200, // duration in ms for (un)fold transition
    HEIGHT_OPEN: '400px', // height in px of thread window when open
    HEIGHT_FOLDED: '34px', // height, in px, of thread window when folded
    /**
     * Children of this class must make use of `thread`, which is an object that
     * represent the thread that is linked to this thread window.
     *
     * If no thread is provided, this will represent the "blank" thread window.
     *
     * @abstract
     * @param {Widget} parent
     * @param {im_livechat.legacy.mail.model.AbstractThread} [thread=null] the thread that this
     *   thread window is linked to. If not set, it is the "blank" thread
     *   window.
     * @param {Object} [options={}]
     * @param {im_livechat.legacy.mail.model.AbstractThread} [options.thread]
     */
    init: function (parent, thread, options) {
        this._super(parent);

        this.options = _.defaults(options || {}, {
            autofocus: true,
            displayStars: true,
            displayReplyIcons: false,
            displayNotificationIcons: false,
            placeholder: _t("Say something"),
        });

        this._hidden = false;
        this._thread = thread || null;

        this._debouncedOnScroll = _.debounce(this._onScroll.bind(this), 100);

        if (!this.hasThread()) {
            // internal fold state of thread window without any thread
            this._folded = false;
        }
    },
    start: function () {
        var self = this;
        this.$input = this.$('.o_composer_text_field');
        this.$header = this.$('.o_thread_window_header');
        var options = {
            displayMarkAsRead: false,
            displayStars: this.options.displayStars,
        };
        if (this._thread && this._thread._type === 'document_thread') {
            options.displayDocumentLinks = false;
        }
        this._threadWidget = new ThreadWidget(this, options);

        // animate the (un)folding of thread windows
        this.$el.css({ transition: 'height ' + this.FOLD_ANIMATION_DURATION + 'ms linear' });
        if (this.isFolded()) {
            this.$el.css('height', this.HEIGHT_FOLDED);
        } else if (this.options.autofocus) {
            this._focusInput();
        }
        if (!config.device.isMobile) {
            var margin_dir = _t.database.parameters.direction === "rtl" ? "margin-left" : "margin-right";
            this.$el.css(margin_dir, $.position.scrollbarWidth());
        }
        var def = this._threadWidget.replace(this.$('.o_thread_window_content')).then(function () {
            self._threadWidget.$el.on('scroll', self, self._debouncedOnScroll);
        });
        return Promise.all([this._super(), def]);
    },
    /**
     * @override
     */
    do_hide: function () {
        this._hidden = true;
        this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    do_show: function () {
        this._hidden = false;
        this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    do_toggle: function (display) {
        this._hidden = _.isBoolean(display) ? !display : !this._hidden;
        this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Close this window
     *
     * @abstract
     */
    close: function () {},
    /**
     * Get the ID of the thread window, which is equivalent to the ID of the
     * thread related to this window
     *
     * @returns {integer|string}
     */
    getID: function () {
        return this._getThreadID();
    },
    /**
     * @returns {mail.model.Thread|undefined}
     */
    getThread: function () {
        if (!this.hasThread()) {
            return undefined;
        }
        return this._thread;
    },
    /**
     * Get the status of the thread, such as the im status of a DM chat
     * ('online', 'offline', etc.). If this window has no thread, returns
     * `undefined`.
     *
     * @returns {string|undefined}
     */
    getThreadStatus: function () {
        if (!this.hasThread()) {
            return undefined;
        }
        return this._thread.getStatus();
    },
    /**
     * Get the title of the thread window, which usually contains the name of
     * the thread.
     *
     * @returns {string}
     */
    getTitle: function () {
        if (!this.hasThread()) {
            return _t("Undefined");
        }
        return this._thread.getTitle();
    },
    /**
     * Get the unread counter of the related thread. If there are no thread
     * linked to this window, returns 0.
     *
     * @returns {integer}
     */
    getUnreadCounter: function () {
        if (!this.hasThread()) {
            return 0;
        }
        return this._thread.getUnreadCounter();
    },
    /**
     * States whether this thread window is related to a thread or not.
     *
     * This is useful in order to provide specific behaviour for thread windows
     * without any thread, e.g. let them open a thread from this "blank" thread
     * window.
     *
     * @returns {boolean}
     */
    hasThread: function () {
        return !! this._thread;
    },
    /**
     * Tells whether the bottom of the thread in the thread window is visible
     * or not.
     *
     * @returns {boolean}
     */
    isAtBottom: function () {
        return this._threadWidget.isAtBottom();
    },
    /**
     * State whether the related thread is folded or not. If there are no
     * thread related to this window, it means this is the "blank" thread
     * window, therefore we use the internal folded state.
     *
     * @returns {boolean}
     */
    isFolded: function () {
        if (!this.hasThread()) {
            return this._folded;
        }
        return this._thread.isFolded();
    },
    /**
     * States whether the current environment is in mobile or not. This is
     * useful in order to customize the template rendering for mobile view.
     *
     * @returns {boolean}
     */
    isMobile: function () {
        return config.device.isMobile;
    },
    /**
     * States whether the thread window is hidden or not.
     *
     * @returns {boolean}
     */
    isHidden: function () {
        return this._hidden;
    },
    /**
     * States whether the input of the thread window should be displayed or not.
     * By default, any thread window with a thread needs a composer.
     *
     * @returns {boolean}
     */
    needsComposer: function () {
        return this.hasThread();
    },
    /**
     * Render the thread window
     */
    render: function () {
        this.renderHeader();
        if (this.hasThread()) {
            this._threadWidget.render(this._thread, { displayLoadMore: false });
        }
    },
    /**
     * Render the header of this thread window.
     * This is useful when some information on the header have be updated such
     * as the status or the title of the thread that have changed.
     *
     * @private
     */
    renderHeader: function () {
        var options = this._getHeaderRenderingOptions();
        this.$header.html(
            QWeb.render('im_livechat.legacy.mail.AbstractThreadWindow.HeaderContent', options));
    },
    /**
     * Scroll to the bottom of the thread in the thread window
     */
    scrollToBottom: function () {
        this._threadWidget.scrollToBottom();
    },
    /**
     * Toggle the fold state of this thread window. Also update the fold state
     * of the thread model. If the boolean parameter `folded` is provided, it
     * folds/unfolds the window when it is set/unset.
     *
     * @param {boolean} [folded] if not a boolean, toggle the fold state.
     *   Otherwise, fold/unfold the window if set/unset.
     */
    toggleFold: function (folded) {
        if (!_.isBoolean(folded)) {
            folded = !this.isFolded();
        }
        this._updateThreadFoldState(folded);
    },
    /**
     * Update the visual state of the window so that it matched the internal
     * fold state. This is useful in case the related thread has its fold state
     * that has been changed.
     */
    updateVisualFoldState: function () {
        if (!this.isFolded()) {
            this._threadWidget.scrollToBottom();
            if (this.options.autofocus) {
                this._focusInput();
            }
        }
        var height = this.isFolded() ? this.HEIGHT_FOLDED : this.HEIGHT_OPEN;
        this.$el.css({ height: height });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Set the focus on the composer of the thread window. This operation is
     * ignored in mobile context.
     *
     * @private
     * Set the focus on the input of the window
     */
    _focusInput: function () {
        if (
            config.device.touch &&
            config.device.size_class <= config.device.SIZES.SM
        ) {
            return;
        }
        this.$input.focus();
    },
    /**
     * Returns the options used by the rendering of the window's header
     *
     * @private
     * @returns {Object}
     */
    _getHeaderRenderingOptions: function () {
        return {
            status: this.getThreadStatus(),
            thread: this.getThread(),
            title: this.getTitle(),
            unreadCounter: this.getUnreadCounter(),
            widget: this,
        };
    },
    /**
     * Get the ID of the related thread.
     * If this window is not related to a thread, it means this is the "blank"
     * thread window, therefore it returns "_blank" as its ID.
     *
     * @private
     * @returns {integer|string} the threadID, or '_blank' for the window that
     *   is not related to any thread.
     */
    _getThreadID: function () {
        if (!this.hasThread()) {
            return '_blank';
        }
        return this._thread.getID();
    },
    /**
     * Tells whether there is focus on this thread. Note that a thread that has
     * the focus means the input has focus.
     *
     * @private
     * @returns {boolean}
     */
    _hasFocus: function () {
        return this.$input.is(':focus');
    },
    /**
     * Post a message on this thread window, and auto-scroll to the bottom of
     * the thread.
     *
     * @private
     * @param {Object} messageData
     */
    _postMessage: function (messageData) {
        var self = this;
        if (!this.hasThread()) {
            return;
        }
        this._thread.postMessage(messageData)
            .then(function () {
                self._threadWidget.scrollToBottom();
            });
    },
    /**
     * Update the fold state of the thread.
     *
     * This function is called when toggling the fold state of this window.
     * If there is no thread linked to this window, it means this is the
     * "blank" thread window, therefore we use the internal state 'folded'
     *
     * @private
     * @param {boolean} folded
     */
    _updateThreadFoldState: function (folded) {
        if (this.hasThread()) {
            this._thread.fold(folded);
        } else {
            this._folded = folded;
            this.updateVisualFoldState();
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Close the thread window.
     * Mark the thread as read if the thread window was open.
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onClickClose: function (ev) {
        ev.stopPropagation();
        ev.preventDefault();
        if (
            this.hasThread() &&
            this._thread.getUnreadCounter() > 0 &&
            !this.isFolded()
        ) {
            this._thread.markAsRead();
        }
        this.close();
    },
    /**
     * Fold/unfold the thread window.
     * Also mark the thread as read.
     *
     * @private
     */
    _onClickFold: function () {
        if (!config.device.isMobile) {
            this.toggleFold();
        }
    },
    /**
     * Called when the composer is clicked -> forces focus on input even if
     * jquery's blockUI is enabled.
     *
     * @private
     * @param {Event} ev
     */
    _onComposerClick: function (ev) {
        if ($(ev.target).closest('a, button').length) {
            return;
        }
        this._focusInput();
    },
    /**
     * @private
     */
    _onDocumentViewerClose: function () {
        this._focusInput();
    },
    /**
     * Called when typing something on the composer of this thread window.
     *
     * @private
     * @param {KeyboardEvent} ev
     */
    _onKeydown: function (ev) {
        ev.stopPropagation(); // to prevent jquery's blockUI to cancel event
        // ENTER key (avoid requiring jquery ui for external livechat)
        if (ev.which === 13) {
            var content = _.str.trim(this.$input.val());
            var messageData = {
                content: content,
                attachment_ids: [],
                partner_ids: [],
            };
            this.$input.val('');
            if (content) {
                this._postMessage(messageData);
            }
        }
    },
    /**
     * @private
     * @param {KeyboardEvent} ev
     */
    _onKeypress: function (ev) {
        ev.stopPropagation(); // to prevent jquery's blockUI to cancel event
    },
    /**
     * @private
     */
    _onScroll: function () {
        if (this.hasThread() && this.isAtBottom()) {
            this._thread.markAsRead();
        }
    },
    /**
     * When a thread window is clicked on, we want to give the focus to the main
     * input. An exception is made when the user is selecting something.
     *
     * @private
     */
    _onThreadWindowClicked: function () {
        var selectObj = window.getSelection();
        if (selectObj.anchorOffset === selectObj.focusOffset) {
            this.$input.focus();
        }
    },
});

return AbstractThreadWindow;

});

odoo.define('im_livechat.legacy.mail.model.CCThrottleFunctionObject', function (require) {
"use strict";

var Class = require('web.Class');

/**
 * This object models the behaviour of the clearable and cancellable (CC)
 * throttle version of a provided function.
 */
var CCThrottleFunctionObject = Class.extend({

    /**
     * @param {Object} params
     * @param {integer} params.duration duration of the 'cooldown' phase, i.e.
     *   the minimum duration between the most recent function call that has
     *   been made and the following function call.
     * @param {function} params.func provided function for making the CC
     *   throttled version.
     */
    init: function (params) {
        this._arguments = undefined;
        this._cooldownTimeout = undefined;
        this._duration = params.duration;
        this._func = params.func;
        this._shouldCallFunctionAfterCD = false;
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Cancel any buffered function call, but keep the cooldown phase running.
     */
    cancel: function () {
        this._arguments = undefined;
        this._shouldCallFunctionAfterCD = false;
    },
    /**
     * Clear the internal throttle timer, so that the following function call
     * is immediate. For instance, if there is a cooldown stage, it is aborted.
     */
    clear: function () {
        if (this._cooldownTimeout) {
            clearTimeout(this._cooldownTimeout);
            this._onCooldownTimeout();
        }
    },
    /**
     * Called when there is a call to the function. This function is throttled,
     * so the time it is called depends on whether the "cooldown stage" occurs
     * or not:
     *
     * - no cooldown stage: function is called immediately, and it starts
     *      the cooldown stage when successful.
     * - in cooldown stage: function is called when the cooldown stage has
     *      ended from timeout.
     *
     * Note that after the cooldown stage, only the last attempted function
     * call will be considered.
     */
    do: function () {
        this._arguments = Array.prototype.slice.call(arguments);
        if (this._cooldownTimeout === undefined) {
            this._callFunction();
        } else {
            this._shouldCallFunctionAfterCD = true;
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Immediately calls the function with arguments of last buffered function
     * call. It initiates the cooldown stage after this function call.
     *
     * @private
     */
    _callFunction: function () {
        this._func.apply(null, this._arguments);
        this._cooldown();
    },
    /**
     * Called when the function has been successfully called. The following
     * calls to the function with this object should suffer a "cooldown stage",
     * which prevents the function from being called until this stage has ended.
     *
     * @private
     */
    _cooldown: function () {
        this.cancel();
        this._cooldownTimeout = setTimeout(
            this._onCooldownTimeout.bind(this),
            this._duration
        );
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called when the cooldown stage ended from timeout. Calls the function if
     * a function call was buffered.
     *
     * @private
     */
    _onCooldownTimeout: function () {
        if (this._shouldCallFunctionAfterCD) {
            this._callFunction();
        } else {
            this._cooldownTimeout = undefined;
        }
    },
});

return CCThrottleFunctionObject;

});

odoo.define('im_livechat.legacy.mail.model.CCThrottleFunction', function (require) {
"use strict";

var CCThrottleFunctionObject = require('im_livechat.legacy.mail.model.CCThrottleFunctionObject');

/**
 * A function that creates a cancellable and clearable (CC) throttle version
 * of a provided function.
 *
 * This throttle mechanism allows calling a function at most once during a
 * certain period:
 *
 * - When a function call is made, it enters a 'cooldown' phase, in which any
 *     attempt to call the function is buffered until the cooldown phase ends.
 * - At most 1 function call can be buffered during the cooldown phase, and the
 *     latest one in this phase will be considered at its end.
 * - When a cooldown phase ends, any buffered function call will be performed
 *     and another cooldown phase will follow up.
 *
 * This throttle version has the following interesting properties:
 *
 * - cancellable: it allows removing a buffered function call during the
 *     cooldown phase, but it keeps the cooldown phase running.
 * - clearable: it allows to clear the internal clock of the throttled function,
 *     so that any cooldown phase is immediately ending.
 *
 * @param {Object} params
 * @param {integer} params.duration a duration for the throttled behaviour,
 *   in milli-seconds.
 * @param {function} params.func the function to throttle
 * @returns {function} the cancellable and clearable throttle version of the
 *   provided function in argument.
 */
var CCThrottleFunction = function (params) {
    var duration = params.duration;
    var func = params.func;

    var throttleFunctionObject = new CCThrottleFunctionObject({
        duration: duration,
        func: func,
    });

    var callable = function () {
        return throttleFunctionObject.do.apply(throttleFunctionObject, arguments);
    };
    callable.cancel = function () {
        throttleFunctionObject.cancel();
    };
    callable.clear = function () {
        throttleFunctionObject.clear();
    };

    return callable;
};

return CCThrottleFunction;

});

odoo.define('im_livechat.legacy.mail.model.Timer', function (require) {
"use strict";

var Class = require('web.Class');

/**
 * This class creates a timer which, when times out, calls a function.
 */
var Timer = Class.extend({

    /**
     * Instantiate a new timer. Note that the timer is not started on
     * initialization (@see start method).
     *
     * @param {Object} params
     * @param {number} params.duration duration of timer before timeout in
     *   milli-seconds.
     * @param {function} params.onTimeout function that is called when the
     *   timer times out.
     */
    init: function (params) {
        this._duration = params.duration;
        this._timeout = undefined;
        this._timeoutCallback = params.onTimeout;
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Clears the countdown of the timer.
     */
    clear: function () {
        clearTimeout(this._timeout);
    },
    /**
     * Resets the timer, i.e. resets its duration.
     */
    reset: function () {
        this.clear();
        this.start();
    },
    /**
     * Starts the timer, i.e. after a certain duration, it times out and calls
     * a function back.
     */
    start: function () {
        this._timeout = setTimeout(this._onTimeout.bind(this), this._duration);
    },

    //--------------------------------------------------------------------------
    // Handler
    //--------------------------------------------------------------------------

    /**
     * Called when the timer times out, calls back a function on timeout.
     *
     * @private
     */
    _onTimeout: function () {
        this._timeoutCallback();
    },

});

return Timer;

});

odoo.define('im_livechat.legacy.mail.model.Timers', function (require) {
"use strict";

var Timer = require('im_livechat.legacy.mail.model.Timer');

var Class = require('web.Class');

/**
 * This class lists several timers that use a same callback and duration.
 */
var Timers = Class.extend({

    /**
     * Instantiate a new list of timers
     *
     * @param {Object} params
     * @param {integer} params.duration duration of the underlying timers from
     *   start to timeout, in milli-seconds.
     * @param {function} params.onTimeout a function to call back for underlying
     *   timers on timeout.
     */
    init: function (params) {
        this._duration = params.duration;
        this._timeoutCallback = params.onTimeout;
        this._timers = {};
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Register a timer with ID `timerID` to start.
     *
     * - an already registered timer with this ID is reset.
     * - (optional) can provide a list of arguments that is passed to the
     *   function callback when timer times out.
     *
     * @param {Object} params
     * @param {Array} [params.timeoutCallbackArguments]
     * @param {integer} params.timerID
     */
    registerTimer: function (params) {
        var timerID = params.timerID;
        if (this._timers[timerID]) {
            this._timers[timerID].clear();
        }
        var timerParams = {
            duration: this._duration,
            onTimeout: this._timeoutCallback,
        };
        if ('timeoutCallbackArguments' in params) {
            timerParams.onTimeout = this._timeoutCallback.bind.apply(
                this._timeoutCallback,
                [null].concat(params.timeoutCallbackArguments)
            );
        } else {
            timerParams.onTimeout = this._timeoutCallback;
        }
        this._timers[timerID] = new Timer(timerParams);
        this._timers[timerID].start();
    },
    /**
     * Unregister a timer with ID `timerID`. The unregistered timer is aborted
     * and will not time out.
     *
     * @param {Object} params
     * @param {integer} params.timerID
     */
    unregisterTimer: function (params) {
        var timerID = params.timerID;
        if (this._timers[timerID]) {
            this._timers[timerID].clear();
            delete this._timers[timerID];
        }
    },

});

return Timers;

});

odoo.define('im_livechat.legacy.mail.widget.Thread', function (require) {
"use strict";

var DocumentViewer = require('im_livechat.legacy.mail.DocumentViewer');
var mailUtils = require('@mail/js/utils');

var core = require('web.core');
var time = require('web.time');
var Widget = require('web.Widget');

var QWeb = core.qweb;
var _lt = core._lt;

var ORDER = {
    ASC: 1, // visually, ascending order of message IDs (from top to bottom)
    DESC: -1, // visually, descending order of message IDs (from top to bottom)
};

var READ_MORE = _lt("read more");
var READ_LESS = _lt("read less");

/**
 * This is a generic widget to render a thread.
 * Any thread that extends mail.model.AbstractThread can be used with this
 * widget.
 */
var ThreadWidget = Widget.extend({
    className: 'o_mail_thread',

    events: {
        'click a': '_onClickRedirect',
        'click img': '_onClickRedirect',
        'click strong': '_onClickRedirect',
        'click .o_thread_show_more': '_onClickShowMore',
        'click .o_attachment_download': '_onAttachmentDownload',
        'click .o_attachment_view': '_onAttachmentView',
        'click .o_attachment_delete_cross': '_onDeleteAttachment',
        'click .o_thread_message_needaction': '_onClickMessageNeedaction',
        'click .o_thread_message_star': '_onClickMessageStar',
        'click .o_thread_message_reply': '_onClickMessageReply',
        'click .oe_mail_expand': '_onClickMailExpand',
        'click .o_thread_message': '_onClickMessage',
        'click': '_onClick',
        'click .o_thread_message_notification_error': '_onClickMessageNotificationError',
        'click .o_thread_message_moderation': '_onClickMessageModeration',
        'change .moderation_checkbox': '_onChangeModerationCheckbox',
    },

    /**
     * @override
     * @param {widget} parent
     * @param {Object} options
     */
    init: function (parent, options) {
        this._super.apply(this, arguments);
        this.attachments = [];
        // options when the thread is enabled (e.g. can send message,
        // interact on messages, etc.)
        this._enabledOptions = _.defaults(options || {}, {
            displayOrder: ORDER.ASC,
            displayMarkAsRead: true,
            displayModerationCommands: false,
            displayStars: true,
            displayDocumentLinks: true,
            displayAvatars: true,
            squashCloseMessages: true,
            displayNotificationIcons: true,
            displayReplyIcons: false,
            loadMoreOnScroll: false,
            hasMessageAttachmentDeletable: false,
        });
        // options when the thread is disabled
        this._disabledOptions = {
            displayOrder: this._enabledOptions.displayOrder,
            displayMarkAsRead: false,
            displayModerationCommands: false,
            displayStars: false,
            displayDocumentLinks: false,
            displayAvatars: this._enabledOptions.displayAvatars,
            squashCloseMessages: false,
            displayNotificationIcons: false,
            displayReplyIcons: false,
            loadMoreOnScroll: this._enabledOptions.loadMoreOnScroll,
            hasMessageAttachmentDeletable: false,
        };
        this._selectedMessageID = null;
        this._currentThreadID = null;
        this._messageMailPopover = null;
        this._messageSeenPopover = null;
        // used to track popover IDs to destroy on re-rendering of popovers
        this._openedSeenPopoverIDs = [];
    },
    /**
     * The message mail popover may still be shown at this moment. If we do not
     * remove it, it stays visible on the page until a page reload.
     *
     * @override
     */
    destroy: function () {
        clearInterval(this._updateTimestampsInterval);
        if (this._messageMailPopover) {
            this._messageMailPopover.popover('hide');
        }
        if (this._messageSeenPopover) {
            this._messageSeenPopover.popover('hide');
        }
        this._destroyOpenSeenPopoverIDs();
        this._super();
    },
    /**
     * @param {im_livechat.legacy.mail.model.AbstractThread} thread the thread to render.
     * @param {Object} [options]
     * @param {integer} [options.displayOrder=ORDER.ASC] order of displaying
     *    messages in the thread:
     *      - ORDER.ASC: last message is at the bottom of the thread
     *      - ORDER.DESC: last message is at the top of the thread
     * @param {boolean} [options.displayLoadMore]
     * @param {Array} [options.domain=[]] the domain for the messages in the
     *    thread.
     * @param {boolean} [options.isCreateMode]
     * @param {boolean} [options.scrollToBottom=false]
     * @param {boolean} [options.squashCloseMessages]
     */
    render: function (thread, options) {
        var self = this;

        var shouldScrollToBottomAfterRendering = false;
        if (this._currentThreadID === thread.getID() && this.isAtBottom()) {
            shouldScrollToBottomAfterRendering = true;
        }
        this._currentThreadID = thread.getID();

        // copy so that reverse do not alter order in the thread object
        var messages = _.clone(thread.getMessages());

        var modeOptions = options.isCreateMode ? this._disabledOptions :
                                                    this._enabledOptions;

        // attachments ordered by messages order (increasing ID)
        this.attachments = _.uniq(_.flatten(_.map(messages, function (message) {
            return message.getAttachments();
        })));

        options = _.extend({}, modeOptions, options, {
            selectedMessageID: this._selectedMessageID,
        });

        // dict where key is message ID, and value is whether it should display
        // the author of message or not visually
        var displayAuthorMessages = {};

        // Hide avatar and info of a message if that message and the previous
        // one are both comments wrote by the same author at the same minute
        // and in the same document (users can now post message in documents
        // directly from a channel that follows it)
        var prevMessage;
        _.each(messages, function (message) {
            if (
                // is first message of thread
                !prevMessage ||
                // more than 1 min. elasped
                (Math.abs(message.getDate().diff(prevMessage.getDate())) > 60000) ||
                prevMessage.getType() !== 'comment' ||
                message.getType() !== 'comment' ||
                // from a different author
                (prevMessage.getAuthorID() !== message.getAuthorID()) ||
                (
                    // messages are linked to a document thread
                    (
                        prevMessage.isLinkedToDocumentThread() &&
                        message.isLinkedToDocumentThread()
                    ) &&
                    (
                        // are from different documents
                        prevMessage.getDocumentModel() !== message.getDocumentModel() ||
                        prevMessage.getDocumentID() !== message.getDocumentID()
                    )
                )
            ) {
                displayAuthorMessages[message.getID()] = true;
            } else {
                displayAuthorMessages[message.getID()] = !options.squashCloseMessages;
            }
            prevMessage = message;
        });

        if (modeOptions.displayOrder === ORDER.DESC) {
            messages.reverse();
        }

        this.$el.html(QWeb.render('im_livechat.legacy.mail.widget.Thread', {
            thread: thread,
            displayAuthorMessages: displayAuthorMessages,
            options: options,
            ORDER: ORDER,
            dateFormat: time.getLangDatetimeFormat(),
        }));

        _.each(messages, function (message) {
            var $message = self.$('.o_thread_message[data-message-id="' + message.getID() + '"]');
            $message.find('.o_mail_timestamp').data('date', message.getDate());

            self._insertReadMore($message);
        });

        if (shouldScrollToBottomAfterRendering) {
            this.scrollToBottom();
        }

        if (!this._updateTimestampsInterval) {
            this.updateTimestampsInterval = setInterval(function () {
                self._updateTimestamps();
            }, 1000 * 60);
        }

        this._renderMessageNotificationPopover(messages);
        if (thread.hasSeenFeature()) {
            this._renderMessageSeenPopover(thread, messages);
        }
    },

    /**
     * Render thread widget when loading, i.e. when messaging is not yet ready.
     * @see /mail/init_messaging
     */
    renderLoading: function () {
        this.$el.html(QWeb.render('im_livechat.legacy.mail.widget.ThreadLoading'));
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    getScrolltop: function () {
        return this.$el.scrollTop();
    },
    /**
     * State whether the bottom of the thread is visible or not,
     * with a tolerance of 5 pixels
     *
     * @return {boolean}
     */
    isAtBottom: function () {
        var fullHeight = this.el.scrollHeight;
        var topHiddenHeight = this.$el.scrollTop();
        var visibleHeight = this.$el.outerHeight();
        var bottomHiddenHeight = fullHeight - topHiddenHeight - visibleHeight;
        return bottomHiddenHeight < 5;
    },
    /**
     * Removes a message and re-renders the thread
     *
     * @param {integer} [messageID] the id of the removed message
     * @param {mail.model.AbstractThread} thread the thread which contains
     *   updated list of messages (so it does not contain any message with ID
     *   `messageID`).
     * @param {Object} [options] options for the thread rendering
     */
    removeMessageAndRender: function (messageID, thread, options) {
        var self = this;
        this._currentThreadID = thread.getID();
        return new Promise(function (resolve, reject) {
            self.$('.o_thread_message[data-message-id="' + messageID + '"]')
            .fadeOut({
                done: function () {
                    if (self._currentThreadID === thread.getID()) {
                        self.render(thread, options);
                    }
                    resolve();
                },
                duration: 200,
            });
        });
    },
    /**
     * Scroll to the bottom of the thread
     */
    scrollToBottom: function () {
        this.$el.scrollTop(this.el.scrollHeight);
    },
    /**
     * Scrolls the thread to a given message
     *
     * @param {integer} options.msgID the ID of the message to scroll to
     * @param {integer} [options.duration]
     * @param {boolean} [options.onlyIfNecessary]
     */
    scrollToMessage: function (options) {
        var $target = this.$('.o_thread_message[data-message-id="' + options.messageID + '"]');
        if (options.onlyIfNecessary) {
            var delta = $target.parent().height() - $target.height();
            var offset = delta < 0 ?
                            0 :
                            delta - ($target.offset().top - $target.offsetParent().offset().top);
            offset = - Math.min(offset, 0);
            this.$el.scrollTo("+=" + offset + "px", options.duration);
        } else if ($target.length) {
            this.$el.scrollTo($target);
        }
    },
    /**
     * Scroll to the specific position in pixel
     *
     * If no position is provided, scroll to the bottom of the thread
     *
     * @param {integer} [position] distance from top to position in pixels.
     *    If not provided, scroll to the bottom.
     */
    scrollToPosition: function (position) {
        if (position) {
            this.$el.scrollTop(position);
        } else {
            this.scrollToBottom();
        }
    },
    /**
     * Toggle all the moderation checkboxes in the thread
     *
     * @param {boolean} checked if true, check the boxes,
     *      otherwise uncheck them.
     */
    toggleModerationCheckboxes: function (checked) {
        this.$('.moderation_checkbox').prop('checked', checked);
    },
    /**
     * Unselect the selected message
     */
    unselectMessage: function () {
        this.$('.o_thread_message').removeClass('o_thread_selected_message');
        this._selectedMessageID = null;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _destroyOpenSeenPopoverIDs: function () {
        _.each(this._openedSeenPopoverIDs, function (popoverID) {
            $('#' + popoverID).remove();
        });
        this._openedSeenPopoverIDs = [];
    },
    /**
     * Modifies $element to add the 'read more/read less' functionality
     * All element nodes with 'data-o-mail-quote' attribute are concerned.
     * All text nodes after a ``#stopSpelling`` element are concerned.
     * Those text nodes need to be wrapped in a span (toggle functionality).
     * All consecutive elements are joined in one 'read more/read less'.
     *
     * @private
     * @param {jQuery} $element
     */
    _insertReadMore: function ($element) {
        var self = this;

        var groups = [];
        var readMoreNodes;

        // nodeType 1: element_node
        // nodeType 3: text_node
        var $children = $element.contents()
            .filter(function () {
                return this.nodeType === 1 ||
                        this.nodeType === 3 &&
                        this.nodeValue.trim();
            });

        _.each($children, function (child) {
            var $child = $(child);

            // Hide Text nodes if "stopSpelling"
            if (
                child.nodeType === 3 &&
                $child.prevAll('[id*="stopSpelling"]').length > 0
            ) {
                // Convert Text nodes to Element nodes
                $child = $('<span>', {
                    text: child.textContent,
                    'data-o-mail-quote': '1',
                });
                child.parentNode.replaceChild($child[0], child);
            }

            // Create array for each 'read more' with nodes to toggle
            if (
                $child.attr('data-o-mail-quote') ||
                (
                    $child.get(0).nodeName === 'BR' &&
                    $child.prev('[data-o-mail-quote="1"]').length > 0
                )
            ) {
                if (!readMoreNodes) {
                    readMoreNodes = [];
                    groups.push(readMoreNodes);
                }
                $child.hide();
                readMoreNodes.push($child);
            } else {
                readMoreNodes = undefined;
                self._insertReadMore($child);
            }
        });

        _.each(groups, function (group) {
            // Insert link just before the first node
            var $readMore = $('<a>', {
                class: 'o_mail_read_more',
                href: '#',
                text: READ_MORE,
            }).insertBefore(group[0]);

            // Toggle All next nodes
            var isReadMore = true;
            $readMore.click(function (e) {
                e.preventDefault();
                isReadMore = !isReadMore;
                _.each(group, function ($child) {
                    $child.hide();
                    $child.toggle(!isReadMore);
                });
                $readMore.text(isReadMore ? READ_MORE : READ_LESS);
            });
        });
    },
    /**
    * @private
    * @param {MouseEvent} ev
    */
    _onDeleteAttachment: function (ev) {
        ev.stopPropagation();
        var $target = $(ev.currentTarget);
        this.trigger_up('delete_attachment', {
            attachmentId: $target.data('id'),
            attachmentName: $target.data('name')
        });
        },
    /**
     * @private
     * @param {Object} options
     * @param {integer} [options.channelID]
     * @param {string} options.model
     * @param {integer} options.id
     */
    _redirect: _.debounce(function (options) {
        if ('channelID' in options) {
            this.trigger('redirect_to_channel', options.channelID);
        } else {
            this.trigger('redirect', options.model, options.id);
        }
    }, 500, true),
    /**
     * Render the popover when mouse-hovering on the notification icon of a
     * message in the thread.
     * There is at most one such popover at any given time.
     *
     * @private
     * @param {im_livechat.legacy.mail.model.AbstractMessage[]} messages list of messages in the
     *   rendered thread, for which popover on mouseover interaction is
     *   permitted.
     */
    _renderMessageNotificationPopover(messages) {
        if (this._messageMailPopover) {
            this._messageMailPopover.popover('hide');
        }
        if (!this.$('.o_thread_tooltip').length) {
            return;
        }
        this._messageMailPopover = this.$('.o_thread_tooltip').popover({
            html: true,
            boundary: 'viewport',
            placement: 'auto',
            trigger: 'hover',
            offset: '0, 1',
            content: function () {
                var messageID = $(this).data('message-id');
                var message = _.find(messages, function (message) {
                    return message.getID() === messageID;
                });
                return QWeb.render('im_livechat.legacy.mail.widget.Thread.Message.MailTooltip', {
                    notifications: message.getNotifications(),
                });
            },
        });
    },
    /**
     * Render the popover when mouse hovering on the seen icon of a message
     * in the thread. Only seen icons in non-squashed message have popover,
     * because squashed messages hides this icon on message mouseover.
     *
     * @private
     * @param {im_livechat.legacy.mail.model.AbstractThread} thread with thread seen mixin,
     *   @see {im_livechat.legacy.mail.model.ThreadSeenMixin}
     * @param {im_livechat.legacy.mail.model.Message[]} messages list of messages in the
     *   rendered thread.
     */
    _renderMessageSeenPopover: function (thread, messages) {
        var self = this;
        this._destroyOpenSeenPopoverIDs();
        if (this._messageSeenPopover) {
            this._messageSeenPopover.popover('hide');
        }
        if (!this.$('.o_thread_message_core .o_mail_thread_message_seen_icon').length) {
            return;
        }
        this._messageSeenPopover = this.$('.o_thread_message_core .o_mail_thread_message_seen_icon').popover({
            html: true,
            boundary: 'viewport',
            placement: 'auto',
            trigger: 'hover',
            offset: '0, 1',
            content: function () {
                var $this = $(this);
                self._openedSeenPopoverIDs.push($this.attr('aria-describedby'));
                var messageID = $this.data('message-id');
                var message = _.find(messages, function (message) {
                    return message.getID() === messageID;
                });
                return QWeb.render('im_livechat.legacy.mail.widget.Thread.Message.SeenIconPopoverContent', {
                    thread: thread,
                    message: message,
                });
            },
        });
    },
    /**
     * @private
     */
    _updateTimestamps: function () {
        var isAtBottom = this.isAtBottom();
        this.$('.o_mail_timestamp').each(function () {
            var date = $(this).data('date');
            $(this).html(mailUtils.timeFromNow(date));
        });
        if (isAtBottom && !this.isAtBottom()) {
            this.scrollToBottom();
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {MouseEvent} event
     */
    _onAttachmentDownload: function (event) {
        event.stopPropagation();
    },
    /**
     * @private
     * @param {MouseEvent} event
     */
    _onAttachmentView: function (event) {
        event.stopPropagation();
        var activeAttachmentID = $(event.currentTarget).data('id');
        if (activeAttachmentID) {
            var attachmentViewer = new DocumentViewer(this, this.attachments, activeAttachmentID);
            attachmentViewer.appendTo($('body'));
        }
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onChangeModerationCheckbox: function (ev) {
        this.trigger_up('update_moderation_buttons');
    },
    /**
     * @private
     */
    _onClick: function () {
        if (this._selectedMessageID) {
            this.unselectMessage();
            this.trigger('unselect_message');
        }
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickMailExpand: function (ev) {
        ev.preventDefault();
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickMessage: function (ev) {
        $(ev.currentTarget).toggleClass('o_thread_selected_message');
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickMessageNeedaction: function (ev) {
        var messageID = $(ev.currentTarget).data('message-id');
        this.trigger('mark_as_read', messageID);
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickMessageNotificationError(ev) {
        const messageID = $(ev.currentTarget).data('message-id');
        this.do_action('mail.mail_resend_message_action', {
            additional_context: {
                mail_message_to_resend: messageID,
            }
        });
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickMessageReply: function (ev) {
        this._selectedMessageID = $(ev.currentTarget).data('message-id');
        this.$('.o_thread_message').removeClass('o_thread_selected_message');
        this.$('.o_thread_message[data-message-id="' + this._selectedMessageID + '"]')
            .addClass('o_thread_selected_message');
        this.trigger('select_message', this._selectedMessageID);
        ev.stopPropagation();
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickMessageStar: function (ev) {
        var messageID = $(ev.currentTarget).data('message-id');
        this.trigger('toggle_star_status', messageID);
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickMessageModeration: function (ev) {
        var $button = $(ev.currentTarget);
        var messageID = $button.data('message-id');
        var decision = $button.data('decision');
        this.trigger_up('message_moderation', {
            messageID: messageID,
            decision: decision,
        });
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickRedirect: function (ev) {
        // ignore inherited branding
        if ($(ev.target).data('oe-field') !== undefined) {
            return;
        }
        var id = $(ev.target).data('oe-id');
        if (id) {
            ev.preventDefault();
            var model = $(ev.target).data('oe-model');
            var options;
            if (model && (model !== 'mail.channel')) {
                options = {
                    model: model,
                    id: id
                };
            } else {
                options = { channelID: id };
            }
            this._redirect(options);
        }
    },
    /**
     * @private
     */
    _onClickShowMore: function () {
        this.trigger('load_more_messages');
    },
});

ThreadWidget.ORDER = ORDER;

return ThreadWidget;

});

odoo.define('im_livechat.legacy.mail.DocumentViewer', function (require) {
"use strict";

var core = require('web.core');
var Widget = require('web.Widget');
var { hidePDFJSButtons } = require('@web/legacy/js/libs/pdfjs');

var QWeb = core.qweb;

var SCROLL_ZOOM_STEP = 0.1;
var ZOOM_STEP = 0.5;

var DocumentViewer = Widget.extend({
    template: "im_livechat.legacy.mail.DocumentViewer",
    events: {
        'click .o_download_btn': '_onDownload',
        'click .o_viewer_img': '_onImageClicked',
        'click .o_viewer_video': '_onVideoClicked',
        'click .move_next': '_onNext',
        'click .move_previous': '_onPrevious',
        'click .o_rotate': '_onRotate',
        'click .o_zoom_in': '_onZoomIn',
        'click .o_zoom_out': '_onZoomOut',
        'click .o_zoom_reset': '_onZoomReset',
        'click .o_close_btn, .o_viewer_img_wrapper': '_onClose',
        'click .o_print_btn': '_onPrint',
        'DOMMouseScroll .o_viewer_content': '_onScroll', // Firefox
        'mousewheel .o_viewer_content': '_onScroll', // Chrome, Safari, IE
        'keydown': '_onKeydown',
        'keyup': '_onKeyUp',
        'mousedown .o_viewer_img': '_onStartDrag',
        'mousemove .o_viewer_content': '_onDrag',
        'mouseup .o_viewer_content': '_onEndDrag'
    },
    /**
     * The documentViewer takes an array of objects describing attachments in
     * argument, and the ID of an active attachment (the one to display first).
     * Documents that are not of type image or video are filtered out.
     *
     * @override
     * @param {Array<Object>} attachments list of attachments
     * @param {integer} activeAttachmentID
     */
    init: function (parent, attachments, activeAttachmentID) {
        this._super.apply(this, arguments);
        this.attachment = _.filter(attachments, function (attachment) {
            var match = attachment.type === 'url' ? attachment.url.match("(youtu|.png|.jpg|.gif)") : attachment.mimetype.match("(image|video|application/pdf|text)");
            if (match) {
                attachment.fileType = match[1];
                if (match[1].match("(.png|.jpg|.gif)")) {
                    attachment.fileType = 'image';
                }
                if (match[1] === 'youtu') {
                    var youtube_array = attachment.url.split('/');
                    var youtube_token = youtube_array[youtube_array.length - 1];
                    if (youtube_token.indexOf('watch') !== -1) {
                        youtube_token = youtube_token.split('v=')[1];
                        var amp = youtube_token.indexOf('&');
                        if (amp !== -1) {
                            youtube_token = youtube_token.substring(0, amp);
                        }
                    }
                    attachment.youtube = youtube_token;
                }
                return true;
            }
        });
        this.activeAttachment = _.findWhere(attachments, { id: activeAttachmentID });
        this.modelName = 'ir.attachment';
        this._reset();
    },
    /**
     * Open a modal displaying the active attachment
     * @override
     */
    start: function () {
        this.$el.modal('show');
        this.$el.on('hidden.bs.modal', _.bind(this._onDestroy, this));
        this.$('.o_viewer_img').on("load", _.bind(this._onImageLoaded, this));
        this.$('[data-toggle="tooltip"]').tooltip({ delay: 0 });
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    destroy: function () {
        if (this.isDestroyed()) {
            return;
        }
        this.trigger_up('document_viewer_closed');
        this.$el.modal('hide');
        this.$el.remove();
        this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //---------------------------------------------------------------------------

    /**
     * @private
     */
    _next: function () {
        var index = _.findIndex(this.attachment, this.activeAttachment);
        index = (index + 1) % this.attachment.length;
        this.activeAttachment = this.attachment[index];
        this._updateContent();
    },
    /**
     * @private
     */
    _previous: function () {
        var index = _.findIndex(this.attachment, this.activeAttachment);
        index = index === 0 ? this.attachment.length - 1 : index - 1;
        this.activeAttachment = this.attachment[index];
        this._updateContent();
    },
    /**
     * @private
     */
    _reset: function () {
        this.scale = 1;
        this.dragStartX = this.dragstopX = 0;
        this.dragStartY = this.dragstopY = 0;
    },
    /**
     * Render the active attachment
     *
     * @private
     */
    _updateContent: function () {
        this.$('.o_viewer_content').html(QWeb.render('im_livechat.legacy.mail.DocumentViewer.Content', {
            widget: this
        }));
        if (this.activeAttachment.fileType === 'application/pdf') {
            hidePDFJSButtons(this.$('.o_viewer_content')[0]);
        }
        this.$('.o_viewer_img').on("load", _.bind(this._onImageLoaded, this));
        this.$('[data-toggle="tooltip"]').tooltip({ delay: 0 });
        this._reset();
    },
    /**
     * Get CSS transform property based on scale and angle
     *
     * @private
     * @param {float} scale
     * @param {float} angle
     */
    _getTransform: function (scale, angle) {
        return 'scale3d(' + scale + ', ' + scale + ', 1) rotate(' + angle + 'deg)';
    },
    /**
     * Rotate image clockwise by provided angle
     *
     * @private
     * @param {float} angle
     */
    _rotate: function (angle) {
        this._reset();
        var new_angle = (this.angle || 0) + angle;
        this.$('.o_viewer_img').css('transform', this._getTransform(this.scale, new_angle));
        this.$('.o_viewer_img').css('max-width', new_angle % 180 !== 0 ? $(document).height() : '100%');
        this.$('.o_viewer_img').css('max-height', new_angle % 180 !== 0 ? $(document).width() : '100%');
        this.angle = new_angle;
    },
    /**
     * Zoom in/out image by provided scale
     *
     * @private
     * @param {integer} scale
     */
    _zoom: function (scale) {
        if (scale > 0.5) {
            this.$('.o_viewer_img').css('transform', this._getTransform(scale, this.angle || 0));
            this.scale = scale;
        }
        this.$('.o_zoom_reset').add('.o_zoom_out').toggleClass('disabled', scale === 1);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {MouseEvent} e
     */
    _onClose: function (e) {
        e.preventDefault();
        this.destroy();
    },
    /**
     * When popup close complete destroyed modal even DOM footprint too
     *
     * @private
     */
    _onDestroy: function () {
        this.destroy();
    },
    /**
     * @private
     * @param {MouseEvent} e
     */
    _onDownload: function (e) {
        e.preventDefault();
        window.location = '/web/content/' + this.modelName + '/' + this.activeAttachment.id + '/' + 'datas' + '?download=true';
    },
    /**
     * @private
     * @param {MouseEvent} e
     */
    _onDrag: function (e) {
        e.preventDefault();
        if (this.enableDrag) {
            var $image = this.$('.o_viewer_img');
            var $zoomer = this.$('.o_viewer_zoomer');
            var top = $image.prop('offsetHeight') * this.scale > $zoomer.height() ? e.clientY - this.dragStartY : 0;
            var left = $image.prop('offsetWidth') * this.scale > $zoomer.width() ? e.clientX - this.dragStartX : 0;
            $zoomer.css("transform", "translate3d(" + left + "px, " + top + "px, 0)");
            $image.css('cursor', 'move');
        }
    },
    /**
     * @private
     * @param {MouseEvent} e
     */
    _onEndDrag: function (e) {
        e.preventDefault();
        if (this.enableDrag) {
            this.enableDrag = false;
            this.dragstopX = e.clientX - this.dragStartX;
            this.dragstopY = e.clientY - this.dragStartY;
            this.$('.o_viewer_img').css('cursor', '');
        }
    },
    /**
     * On click of image do not close modal so stop event propagation
     *
     * @private
     * @param {MouseEvent} e
     */
    _onImageClicked: function (e) {
        e.stopPropagation();
    },
    /**
     * Remove loading indicator when image loaded
     * @private
     */
    _onImageLoaded: function () {
        this.$('.o_loading_img').hide();
    },
    /**
     * Move next previous attachment on keyboard right left key
     *
     * @private
     * @param {KeyEvent} e
     */
    _onKeydown: function (e) {
        switch (e.which) {
            case $.ui.keyCode.RIGHT:
                e.preventDefault();
                this._next();
                break;
            case $.ui.keyCode.LEFT:
                e.preventDefault();
                this._previous();
                break;
        }
    },
    /**
     * Close popup on ESCAPE keyup
     *
     * @private
     * @param {KeyEvent} e
     */
    _onKeyUp: function (e) {
        switch (e.which) {
            case $.ui.keyCode.ESCAPE:
                e.preventDefault();
                this._onClose(e);
                break;
        }
    },
    /**
     * @private
     * @param {MouseEvent} e
     */
    _onNext: function (e) {
        e.preventDefault();
        this._next();
    },
    /**
     * @private
     * @param {MouseEvent} e
     */
    _onPrevious: function (e) {
        e.preventDefault();
        this._previous();
    },
    /**
     * @private
     * @param {MouseEvent} e
     */
    _onPrint: function (e) {
        e.preventDefault();
        var src = this.$('.o_viewer_img').prop('src');
        var script = QWeb.render('im_livechat.legacy.mail.PrintImage', {
            src: src
        });
        var printWindow = window.open('about:blank', "_new");
        printWindow.document.open();
        printWindow.document.write(script);
        printWindow.document.close();
    },
    /**
     * Zoom image on scroll
     *
     * @private
     * @param {MouseEvent} e
     */
    _onScroll: function (e) {
        var scale;
        if (e.originalEvent.wheelDelta > 0 || e.originalEvent.detail < 0) {
            scale = this.scale + SCROLL_ZOOM_STEP;
            this._zoom(scale);
        } else {
            scale = this.scale - SCROLL_ZOOM_STEP;
            this._zoom(scale);
        }
    },
    /**
     * @private
     * @param {MouseEvent} e
     */
    _onStartDrag: function (e) {
        e.preventDefault();
        this.enableDrag = true;
        this.dragStartX = e.clientX - (this.dragstopX || 0);
        this.dragStartY = e.clientY - (this.dragstopY || 0);
    },
    /**
     * On click of video do not close modal so stop event propagation
     * and provide play/pause the video instead of quitting it
     *
     * @private
     * @param {MouseEvent} e
     */
    _onVideoClicked: function (e) {
        e.stopPropagation();
        var videoElement = e.target;
        if (videoElement.paused) {
            videoElement.play();
        } else {
            videoElement.pause();
        }
    },
    /**
     * @private
     * @param {MouseEvent} e
     */
    _onRotate: function (e) {
        e.preventDefault();
        this._rotate(90);
    },
    /**
     * @private
     * @param {MouseEvent} e
     */
    _onZoomIn: function (e) {
        e.preventDefault();
        var scale = this.scale + ZOOM_STEP;
        this._zoom(scale);
    },
    /**
     * @private
     * @param {MouseEvent} e
     */
    _onZoomOut: function (e) {
        e.preventDefault();
        var scale = this.scale - ZOOM_STEP;
        this._zoom(scale);
    },
    /**
     * @private
     * @param {MouseEvent} e
     */
    _onZoomReset: function (e) {
        e.preventDefault();
        this.$('.o_viewer_zoomer').css("transform", "");
        this._zoom(1);
    },
});
return DocumentViewer;
});


```

## File: static\src\legacy\public_livechat.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates xml:space="preserve">

    <t t-name="im_livechat.legacy.im_livechat.FeedBack">
        <div class="o_livechat_rating text-center">
            <div class="o_livechat_rating_box">
                <div class="o_livechat_rating_feedback_text">
                    Did we correctly answer your question ?
                </div>
                <div class="o_livechat_rating_choices">
                    <img t-att-src="widget.server_origin + '/rating/static/src/img/rating_5.png'" alt="Good" data-value="5"/>
                    <img t-att-src="widget.server_origin + '/rating/static/src/img/rating_3.png'" alt="OK" data-value="3"/>
                    <img t-att-src="widget.server_origin + '/rating/static/src/img/rating_1.png'" alt="Bad" data-value="1" />
                </div>
            </div>
            <div class="o_livechat_rating_reason">
                <textarea id="reason" placeholder="Explain your note"></textarea>
                <div class="o_livechat_rating_reason_button">
                    <button type="button" class="btn btn-primary btn-sm o_rating_submit_button">Send</button>
                </div>
            </div>
            <div class="o_livechat_email text-left">
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

    <!--
        @param {im_livechat.legacy.mail.AbstractThreadWindow} widget
    -->
    <t t-name="im_livechat.legacy.mail.AbstractThreadWindow">
        <div class="o_thread_window o_in_home_menu" t-att-data-thread-id="widget.getID()">
            <div class="o_thread_window_header">
                <t t-call="im_livechat.legacy.mail.AbstractThreadWindow.HeaderContent">
                    <t t-set="status" t-value="widget.getThreadStatus()"/>
                    <t t-set="title" t-value="widget.getTitle()"/>
                    <t t-set="unreadCounter" t-value="widget.getUnreadCounter()"/>
                    <t t-set="thread" t-value="widget.getThread()"/>
                </t>
            </div>
            <div class="o_thread_window_content">
            </div>
            <div t-if="widget.needsComposer()" class="o_thread_composer o_chat_mini_composer">
                <input class="o_composer_text_field" t-att-placeholder="widget.options.placeholder"/>
            </div>
        </div>
    </t>

    <!--
        @param {string} [status] e.g. 'online', 'offline'.
        @param {string} title the title of the thread window, e.g. the record
          name of the document.
        @param {integer} [unreadCounter] the number of unread messages on the
          thread.
        @param {im_livechat.legacy.mail.model.Thread|undefined} thread
        @param {Object} widget
        @param {function} widget.isMobile function without any param that
          states whether it should render for desktop or mobile screen.
    -->
    <t t-name="im_livechat.legacy.mail.AbstractThreadWindow.HeaderContent">
        <span t-if="widget.isMobile()">
            <a href="#" class="o_thread_window_close fa fa-1x fa-arrow-left" aria-label="Close chat window" title="Close chat window"/>
        </span>
        <span class="o_thread_window_title">
            <t t-esc="title"/>
            <span t-if="unreadCounter"> (<t t-esc="unreadCounter"/>)</span>
            <t t-if="thread and thread.hasTypingNotification() and thread.isSomeoneTyping()" t-call="im_livechat.legacy.mail.ThreadTypingIcon"/>
        </span>
        <span t-if="!widget.isMobile()" class="o_thread_window_buttons">
            <a href="#" class="o_thread_window_close fa fa-close"/>
        </span>
    </t>

    <!--
        @param {string} status
        @param {integer|undefined} [partnerID]
    -->
    <t t-name="im_livechat.legacy.mail.UserStatus">
        <span t-att-class="partnerID ? 'o_updatable_im_status' : ''" t-att-data-partner-id="partnerID">
            <i t-if="status == 'online'" class="o_mail_user_status o_user_online fa fa-circle" title="Online" role="img" aria-label="User is online"/>
            <i t-if="status == 'away'" class="o_mail_user_status o_user_idle fa fa-circle" title="Idle" role="img" aria-label="User is idle"/>
            <i t-if="status == 'offline'" class="o_mail_user_status fa fa-circle-o" title="Offline" role="img" aria-label="User is offline"/>
        </span>
    </t>

    <!--
        @param {mail.model.AbstractThread} thread
        @param {Object} options
        @param {boolean} [options.displayEmptyThread]
        @param {boolean} [options.displayModerationCommands]
        @param {boolean} [options.displayNoMatchFound]
        @param {Array} [options.domain=[]] the domain to restrict messages on the thread.
    -->
    <t t-name="im_livechat.legacy.mail.widget.Thread">
        <t t-if="thread.hasMessages({ 'domain': options.domain || [] })">
            <t t-call="im_livechat.legacy.mail.widget.Thread.Content"/>
        </t>
    </t>

    <!-- Rendering of thread when messaging not yet ready -->
    <div t-name="im_livechat.legacy.mail.widget.ThreadLoading" class="o_mail_thread_loading">
        <i class="o_mail_thread_loading_icon fa fa-circle-o-notch fa-spin"/>
        <span>Please wait...</span>
    </div>

    <!--
        @param {im_livechat.legacy.mail.DocumentViewer} widget
    -->
    <t t-name="im_livechat.legacy.mail.DocumentViewer.Content">
        <div class="o_viewer_content">
            <t t-set="model" t-value="widget.modelName"/>
            <div class="o_viewer-header">
                <span class="o_image_caption">
                    <i class="fa fa-picture-o mr8" t-if="widget.activeAttachment.fileType == 'image'" role="img" aria-label="Image" title="Image"/>
                    <i class="fa fa-file-text mr8" t-if="widget.activeAttachment.fileType == 'application/pdf'" role="img" aria-label="PDF file" title="PDF file"/>
                    <i class="fa fa-video-camera mr8" t-if="widget.activeAttachment.fileType == 'video'" role="img" aria-label="Video" title="Video"/>
                    <t t-esc="widget.activeAttachment.name"/>
                    <a role="button" href="#" class="o_download_btn ml8 small" data-toggle="tooltip" data-placement="right" title="Download"><i class="fa fa-fw fa-download" role="img" aria-label="Download"/></a>
                </span>
                <a role="button" class="o_close_btn float-right" href="#" aria-label="Close" title="Close">×</a>
            </div>
            <div class="o_viewer_img_wrapper">
                <div class="o_viewer_zoomer">
                    <t t-if="widget.activeAttachment.fileType === 'image'">
                        <div class="o_loading_img text-center">
                            <i class="fa fa-circle-o-notch fa-spin text-gray-light fa-3x fa-fw" role="img" aria-label="Loading" title="Loading"/>
                        </div>
                        <t t-set="unique" t-value="widget.activeAttachment.checksum ? widget.activeAttachment.checksum.slice(-8) : ''"/>
                        <img class="o_viewer_img" t-attf-src="/web/image/#{widget.activeAttachment.id}?unique=#{unique}&amp;model=#{model}" alt="Viewer"/>
                    </t>
                    <iframe t-if="widget.activeAttachment.fileType == 'application/pdf'" class="mt32 o_viewer_pdf"  t-attf-src="/web/static/lib/pdfjs/web/viewer.html?file=/web/content/#{widget.activeAttachment.id}?model%3D#{model}" />
                    <iframe t-if="(widget.activeAttachment.fileType || '').indexOf('text') !== -1" class="mt32 o_viewer_text" t-attf-src="/web/content/#{widget.activeAttachment.id}?model=#{model}" />
                    <iframe t-if="widget.activeAttachment.fileType == 'youtu'" class="mt32 o_viewer_text"  allow="autoplay; encrypted-media" width="560" height="315" t-attf-src="https://www.youtube.com/embed/#{widget.activeAttachment.youtube}"/>
                    <video t-if="widget.activeAttachment.fileType == 'video'" class="o_viewer_video" controls="controls">
                        <source t-attf-src="/web/image/#{widget.activeAttachment.id}?model=#{model}" t-att-data-type="widget.activeAttachment.mimetype"/>
                    </video>
                </div>
            </div>
            <div t-if="widget.activeAttachment.fileType == 'image'" class="o_viewer_toolbar btn-toolbar" role="toolbar">
                <div class="btn-group" role="group">
                    <a role="button" href="#" class="o_viewer_toolbar_btn btn o_zoom_in" data-toggle="tooltip" title="Zoom In"><i class="fa fa-fw fa-plus" role="img" aria-label="Zoom In"/></a>
                    <a role="button" href="#" class="o_viewer_toolbar_btn btn o_zoom_reset disabled" data-toggle="tooltip" title="Reset Zoom"><i class="fa fa-fw fa-search" role="img" aria-label="Reset Zoom"/></a>
                    <a role="button" href="#" class="o_viewer_toolbar_btn btn o_zoom_out disabled" data-toggle="tooltip" title="Zoom Out"><i class="fa fa-fw fa-minus" role="img" aria-label="Zoom Out"/></a>
                </div>
                <div class="btn-group" role="group">
                    <a role="button" href="#" class="o_viewer_toolbar_btn btn o_rotate" data-toggle="tooltip" title="Rotate"><i class="fa fa-fw fa-repeat" role="img" aria-label="Rotate"/></a>
                </div>
                <div class="btn-group" role="group">
                    <a role="button" href="#" class="o_viewer_toolbar_btn btn o_print_btn" data-toggle="tooltip" title="Print"><i class="fa fa-fw fa-print" role="img" aria-label="Print"/></a>
                    <a role="button" href="#" class="o_viewer_toolbar_btn btn o_download_btn" data-toggle="tooltip" title="Download"><i class="fa fa-fw fa-download" role="img" aria-label="Download"/></a>
                </div>
            </div>
        </div>
    </t>

    <!--
        @param {im_livechat.legacy.mail.DocumentViewer} widget
    -->
    <t t-name="im_livechat.legacy.mail.DocumentViewer">
        <div class="modal o_modal_fullscreen" tabindex="-1" data-keyboard="false" role="dialog">
            <t class="o_document_viewer_content_call" t-call="im_livechat.legacy.mail.DocumentViewer.Content"/>

            <t t-if="widget.attachment.length !== 1">
                <a class="arrow arrow-left move_previous" href="#">
                    <span class="fa fa-chevron-left" role="img" aria-label="Previous" title="Previous"/>
                </a>
                <a class="arrow arrow-right move_next" href="#">
                    <span class="fa fa-chevron-right" role="img" aria-label="Next" title="Next"/>
                </a>
            </t>
        </div>
    </t>

    <!--
        @param {string} src
    -->
    <t t-name="im_livechat.legacy.mail.PrintImage">
        <html>
            <head>
                <script>
                    function onload_img() {
                        setTimeout('print_img()', 10);
                    }
                    function print_img() {
                        window.print();
                        window.close();
                    }
                </script>
            </head>
            <body onload='onload_img()'>
                <img t-att-src='src' alt=""/>
            </body>
        </html>
    </t>

    <!--
        @param {mail.model.AbstractThread} thread
        @param {Object} options
        @param {integer} [options.displayOrder] 1 or -1 ascending (respectively, descending) order for
          the thread messages (from top to bottom)
        @param {Array} [options.domain=[]] the domain to restrict messages on the thread.
        @param {Object} ORDER
        @param {integer} ORDER.ASC=1 messages are ordered by ascending order of IDs, (from top to bottom)
        @param {integer} ORDER.DESC=-1 messages are ordered by descending IDs, (from top to bottom)

                    _____________            _____________
                   |             |          |             |
                   |  message 1  |          |  message n  |
                   |  message 2  |          |  ...        |
                   |  ...        |          |  message 2  |
                   |  message n  |          |  message 1  |
                   |_____________|          |_____________|

        ORDER:           ASC                     DESC

    -->
    <t t-name="im_livechat.legacy.mail.widget.Thread.Content">
        <t t-set="messages" t-value="thread.getMessages({ 'domain': options.domain || [] })"/>
        <t t-if="options.displayOrder === ORDER.ASC" t-call="im_livechat.legacy.mail.widget.Thread.Content.ASC"/>
        <t t-else="" t-call="im_livechat.legacy.mail.widget.Thread.Content.DESC"/>
    </t>

    <!--
        @param {mail.model.AbstractThread} thread
        @param {Object} options
        @param {boolean} [options.displayBottomThreadFreeSpace=false]
        @param {boolean} [options.displayLoadMore=false]

                     _____________
                    |             |
                    |  message 1  |
                    |  message 2  |
                    |  ...        |
                    |  message n  |
                    |_____________|

                      ASC Order
    -->
    <t t-name="im_livechat.legacy.mail.widget.Thread.Content.ASC">
        <div class="o_mail_thread_content">
            <t t-if="options.displayLoadMore" t-call="im_livechat.legacy.mail.widget.Thread.LoadMore"/>
            <t t-call="im_livechat.legacy.mail.widget.Thread.Messages"/>
            <t t-if="options.displayBottomThreadFreeSpace">
                <div class="o_thread_bottom_free_space"/>
            </t>
        </div>
    </t>

    <!--
        @param {mail.model.AbstractThread} thread
        @param {Object} options
        @param {boolean} [options.displayLoadMore=false]
        @param {string|integer} [options.messagesSeparatorPosition] 'top' or
            message ID, the separator is placed just after this message.

                     _____________
                    |             |
                    |  message n  |
                    |  ...        |
                    |  message 2  |
                    |  message 1  |
                    |_____________|

                      DESC Order

    -->
    <t t-name="im_livechat.legacy.mail.widget.Thread.Content.DESC">
        <div class="o_mail_thread_content">
            <t t-if="options.messagesSeparatorPosition == 'top'" t-call="im_livechat.legacy.mail.MessagesSeparator"/>
            <t t-set="messages" t-value="messages.slice().reverse()"/>
            <t t-call="im_livechat.legacy.mail.widget.Thread.Messages"/>
            <t t-if="options.displayLoadMore" t-call="im_livechat.legacy.mail.widget.Thread.LoadMore"/>
        </div>
    </t>

    <!--
        @param {mail.model.AbstractMessage[]} messages messages are ordered based
          on desired display order
    -->
    <t t-name="im_livechat.legacy.mail.widget.Thread.Messages">
        <t t-set="current_day" t-value="0"/>
        <t t-foreach="messages" t-as="message">
            <div t-if="current_day !== message.getDateDay()" class="o_thread_date_separator">
                <span class="o_thread_date">
                    <t t-esc="message.getDateDay()"/>
                </span>
                <t t-set="current_day" t-value="message.getDateDay()"/>
            </div>

            <t t-call="im_livechat.legacy.mail.widget.Thread.Message"/>
        </t>
    </t>

    <!--
        @param {mail.model.AbstractThread} thread
        @param {string} dateFormat
        @param {Object} options
        @param {mail.model.AbstractMessage} message
        @param {Object} options
        @param {boolean} [options.displayAvatars]
        @param {boolean} [options.displayDocumentLinks]
        @param {boolean} [options.displayNotificationIcons]
        @param {boolean} [options.displayMarkAsRead]
        @param {boolean} [options.displayModerationCommands] when set, display the moderation commands on
          the message. This includes the moderation checkboxes (needs a control panel such as in Discuss app).
        @param {boolean} [options.displayReplyIcons]
        @param {boolean} [options.displayStars]
        @param {boolean} [options.displaySubjectsOnMessages]
        @param {boolean} options.hasMessageAttachmentDeletable
        @param {string|integer} [options.messagesSeparatorPosition] 'top' or
            message ID, the separator is placed just after this message.
        @param {integer} [options.selectedMessageID]
    -->
    <t t-name="im_livechat.legacy.mail.widget.Thread.Message">
        <div t-if="!message.isEmpty()" t-att-class="'o_thread_message ' + (message.getID() === options.selectedMessageID ? 'o_thread_selected_message ' : ' ') + (message.isDiscussion() or message.isNotification() ? ' o_mail_discussion ' : ' o_mail_not_discussion ')" t-att-data-message-id="message.getID()">
            <div t-if="options.displayAvatars" class="o_thread_message_sidebar">
                <t t-if="message.hasAuthor()">
                    <div t-if="displayAuthorMessages[message.getID()]" class="o_thread_message_sidebar_image">
                        <img
                            alt=""
                            t-att-src="message.getAvatarSource()"
                            data-oe-model="res.partner"
                            t-att-data-oe-id="message.shouldRedirectToAuthor() ? message.getAuthorID() : ''"
                            t-attf-class="o_thread_message_avatar rounded-circle #{message.shouldRedirectToAuthor() ? 'o_mail_redirect' : ''}"/>
                        <t t-call="im_livechat.legacy.mail.UserStatus">
                            <t t-set="status" t-value="message.getAuthorImStatus()"/>
                            <t t-set="partnerID" t-value="message.getAuthorID()"/>
                        </t>
                    </div>
                </t>
                <t t-else="">
                    <img t-if="displayAuthorMessages[message.getID()]"
                        alt=""
                        t-att-src="message.getAvatarSource()"
                        class="o_thread_message_avatar rounded-circle"/>
                </t>
                <span t-if="!displayAuthorMessages[message.getID()]" t-att-title="message.getDate().format(dateFormat)" class="o_thread_message_side_date">
                    <t t-esc="message.getDate().format('hh:mm')"/>
                </span>
                <i t-if="!displayAuthorMessages[message.getID()] and options.displayStars and message.getType() !== 'notification'"
                    t-att-class="'fa o_thread_message_star o_thread_icon ' + (message.isStarred() ? 'fa-star' : 'fa-star-o')"
                    t-att-data-message-id="message.getID()" title="Mark as Todo" role="img" aria-label="Mark as todo"/>
                <t t-if="!displayAuthorMessages[message.getID()] and thread.hasSeenFeature()" t-call="im_livechat.legacy.mail.widget.Thread.Message.SeenIcon"/>
            </div>
            <div class="o_thread_message_core">
                <p t-if="displayAuthorMessages[message.getID()]" class="o_mail_info text-muted">
                    <t t-if="message.isNote()">
                        Note by
                    </t>

                    <strong t-if="message.hasAuthor()"
                            data-oe-model="res.partner" t-att-data-oe-id="message.shouldRedirectToAuthor() ? message.getAuthorID() : ''"
                            t-attf-class="o_thread_author #{message.shouldRedirectToAuthor() ? 'o_mail_redirect' : ''}">
                        <t t-esc="message.getDisplayedAuthor()"/>
                    </strong>
                    <strong t-elif="message.hasEmailFrom()">
                        <a class="text-muted" t-attf-href="mailto:#{message.getEmailFrom()}?subject=Re: #{message.hasSubject() ? message.getSubject() : ''}">
                            <t t-esc="message.getEmailFrom()"/>
                        </a>
                    </strong>
                    <strong t-else="" class="o_thread_author">
                        <t t-esc="message.getDisplayedAuthor()"/>
                    </strong>

                    - <small class="o_mail_timestamp" t-att-title="message.getDate().format(dateFormat)"><t t-esc="message.getTimeElapsed()"/></small>
                    <t t-if="message.isLinkedToDocumentThread() and options.displayDocumentLinks">
                        <small>on</small> <a t-att-href="message.getURL()" t-att-data-oe-model="message.getDocumentModel()" t-att-data-oe-id="message.getDocumentID()" class="o_document_link"><t t-esc="message.getDocumentName()"/></a>
                    </t>
                    <t t-if="message.originatesFromChannel() and (message.getOriginChannelID() !== thread.getID())">
                        (<small>from</small> <a t-att-data-oe-id="message.getOriginChannelID()" href="#">#<t t-esc="message.getOriginChannelName()"/></a>)
                    </t>
                    <span t-if="options.displayNotificationIcons and message.hasNotifications()" class="o_thread_tooltip_container">
                        <span name="notification_icon" t-attf-class="d-inline-flex align-items-center o_thread_tooltip o_thread_message_notification {{ message.hasNotificationsError() ? 'o_thread_message_notification_error' : '' }}" t-att-data-message-id="message.getID()" t-att-data-message-type="message.getType()">
                            <i t-att-class="message.getNotificationIcon()"/>
                            <small t-if="message.getNotificationText()" t-esc="message.getNotificationText()" class="font-weight-bold ml-1"/>
                        </span>
                    </span>
                    <span t-attf-class="o_thread_icons">
                        <t t-if="thread.hasSeenFeature()" t-call="im_livechat.legacy.mail.widget.Thread.Message.SeenIcon"/>
                       <i t-if="message.isLinkedToDocumentThread() and options.displayReplyIcons"
                           class="fa fa-reply o_thread_icon o_thread_message_reply"
                           t-att-data-message-id="message.getID()" title="Reply" role="img" aria-label="Reply"/>
                        <i t-if="message.isNeedaction() and options.displayMarkAsRead"
                           class="fa fa-check o_thread_icon o_thread_message_needaction"
                           t-att-data-message-id="message.getID()" title="Mark as Read" role="img" aria-label="Mark as Read"/>
                    </span>
                </p>
                <div class="o_thread_message_content">
                    <t t-out="message.getBody()"/>
                    <t t-if="message.hasTrackingValues()">
                        <t t-if="message.hasSubtypeDescription()">
                            <p><t t-esc="message.getSubtypeDescription()"/></p>
                        </t>
                        <t t-call="im_livechat.legacy.mail.widget.Thread.MessageTracking"/>
                    </t>
                    <t t-if="message.hasAttachments()">
                        <div t-if="message.hasImageAttachments()" class="o_attachments_previews">
                            <t t-foreach="message.getImageAttachments()" t-as="attachment">
                                <t t-call="im_livechat.legacy.mail.AttachmentPreview">
                                    <t t-set="isDeletable" t-value="options.hasMessageAttachmentDeletable"/>
                                </t>
                            </t>
                        </div>
                        <div t-if="message.hasNonImageAttachments()" class="o_attachments_list">
                            <t t-foreach="message.getNonImageAttachments()" t-as="attachment">
                                <t t-call="im_livechat.legacy.mail.Attachment">
                                    <t t-set="isDeletable" t-value="options.hasMessageAttachmentDeletable"/>
                                </t>
                            </t>
                        </div>
                    </t>
                </div>
            </div>
        </div>
        <t t-if="options.messagesSeparatorPosition == message.getID()">
            <t t-call="im_livechat.legacy.mail.MessagesSeparator"/>
        </t>
    </t>

    <!--
        Display the seen icon of a message in the thread.
        It shows the fetch 'check' before the seen 'check'. We change the order in the template
        so that fetch 'check' is on top of 'seen' check (z-index does not work with absolute positioning)

        @param {mail.model.Thread} thread
        @param {mail.model.Message} message
    -->
    <t t-name="im_livechat.legacy.mail.widget.Thread.Message.SeenIcon">
        <t t-if="message.isMyselfAuthor() and message.getID() >= thread.getLastMessageIDSeenByEveryone()">
            <span t-attf-class="o_mail_thread_message_seen_icon #{thread.hasEveryoneSeen(message) ? 'o_all_seen' : ''}" t-att-data-message-id="message.getID()">
                <t t-if="thread.hasSomeoneFetched(message)">
                    <i class="fa fa-check"/>
                </t>
                <t t-if="thread.hasSomeoneSeen(message)">
                    <i class="fa fa-check"/>
                </t>
            </span>
        </t>
    </t>

    <!--
        Display the popover content when clicking the seen icion of a message in the thread.

        List the members that have received the messages, and the members that have seen it.

        @param {mail.model.Thread} thread
        @param {mail.model.Message} message
    -->
    <t t-name="im_livechat.legacy.mail.widget.Thread.Message.SeenIconPopoverContent">
        <div class="o_mail_thread_message_seen_icon_content">
            <t t-if="thread.hasEveryoneSeen(message)">
                <p>Seen by Everyone</p>
            </t>
            <t t-else="">
                <t t-if="thread.hasSomeoneSeen(message)">
                    <t t-set="seen_members" t-value="thread.getSeenMembers(message)"/>
                    Seen by:
                    <ul>
                        <li t-foreach="seen_members" t-as="member">
                            <t t-esc="member.name"/> (<t t-esc="member.email"/>)
                        </li>
                    </ul>
                </t>
                <t t-if="thread.hasSomeoneFetched(message)">
                    <t t-if="thread.hasEveryoneFetched(message)">
                        <p>Received by Everyone</p>
                    </t>
                    <t t-else="">
                        <t t-set="fetched_members" t-value="thread.getFetchedNotSeenMembers(message)"/>
                        <t t-if="!_.isEmpty(fetched_members)">
                            Received by:
                            <ul>
                                <li t-foreach="fetched_members" t-as="member">
                                    <t t-esc="member.name"/> (<t t-esc="member.email"/>)
                                </li>
                            </ul>
                        </t>
                    </t>
                </t>
            </t>
        </div>
    </t>

    <!--
        @param {Object[]} notifications: list of notifications
           A notification is an object with at least the following keys:
           {notification_status, partner_name}
    -->
    <t t-name="im_livechat.legacy.mail.widget.Thread.Message.MailTooltip">
        <div t-foreach="notifications" t-as="notification">
            <span name="notification_status" class="d-inline-block text-center o_thread_tooltip_icon">
                <i t-if="notification.notification_status === 'sent'" class='fa fa-check' title="Sent" role="img" aria-label="Sent"/>
                <i t-if="notification.notification_status === 'bounce'" class='fa fa-exclamation text-danger' title="Bounced" role="img" aria-label="Bounced"/>
                <i t-if="notification.notification_status === 'exception'" class='fa fa-exclamation text-danger' title="Error" role="img" aria-label="Error"/>
                <i t-if="notification.notification_status === 'ready'" class='fa fa-send-o' title="Ready" role="img" aria-label="Ready"/>
                <i t-if="notification.notification_status === 'canceled'" class='fa fa-trash-o' title="Canceled" role="img" aria-label="Canceled"/>
            </span>
            <span name="partner_name" t-esc="notification.partner_name"/>
        </div>
    </t>

    <t t-name="im_livechat.legacy.mail.MessagesSeparator">
        <div class="o_thread_new_messages_separator">
            <span class="o_thread_separator_label">New messages</span>
        </div>
    </t>

    <!--
        @param {mail.model.Message} message
    -->
    <t t-name="im_livechat.legacy.mail.widget.Thread.MessageTracking">
        <ul class="o_mail_thread_message_tracking">
            <t t-foreach='message.getTrackingValues()' t-as='value'>
                <li>
                    <t t-esc="value.changed_field"/>:
                    <t t-if="value.old_value">
                        <span> <t t-esc="value.old_value || ((value.field_type !== 'boolean') and '')"/> </span>
                        <span t-if="value.old_value !== value.new_value" class="fa fa-long-arrow-right" role="img" aria-label="Changed" title="Changed"/>
                    </t>
                    <span t-if="value.old_value !== value.new_value">
                        <t t-esc="value.new_value || ((value.field_type !== 'boolean') and '')"/>
                    </span>
                </li>
            </t>
        </ul>
    </t>

    <!--
        @param {Array} attachments
    -->
    <t t-name="im_livechat.legacy.mail.composer.Attachments">
        <div t-if="attachments.length > 0" class="o_attachments o_attachments_list">
            <t t-foreach="attachments" t-as='attachment'>
                <t t-call="im_livechat.legacy.mail.Attachment">
                     <t t-set="editable" t-value="true"/>
                </t>
            </t>
        </div>
    </t>

    <!--
        @param {Object} attachment
        @param {integer} attachment.id
        @param {string} attachment.name
        @param {string} attachment.url
        @param {boolean} [isDeletable=false]
    -->
    <t t-name="im_livechat.legacy.mail.AttachmentPreview">
        <div class="o_attachment" t-att-title="attachment.name">
            <div class="o_attachment_wrap">
                <div class="o_image_box">
                    <div class="o_attachment_image" t-attf-style="background-image:url('/web/image/#{attachment.id}/160x160/?crop=true')"/>
                    <div t-attf-class="o_image_overlay o_attachment_view"  t-att-data-id="attachment.id">
                        <span t-if="isDeletable" class="fa fa-times o_attachment_delete_cross" t-att-title="'Delete ' + attachment.name" t-att-data-id="attachment.id" t-att-data-name="attachment.name"/>
                        <span class="o_attachment_title text-white"><t t-esc="attachment.name"/></span>
                        <a class="o_attachment_download" t-att-href='attachment.url'>
                            <i t-attf-class="fa fa-download text-white" t-att-title="'Download ' + attachment.name" role="img" aria-label="Download"></i>
                        </a>
                    </div>
                </div>
            </div>
        </div>
    </t>

    <!--
        @param {Object} attachment
        @param {string} attachment.filename
        @param {integer} attachment.id
        @param {string} [attachment.mimetype]
        @param {string} attachment.name
        @param {boolean} attachment.upload
        @param {string} attachment.url
        @param {boolean} [editable=false] if set, it means the attachment is rendered in the composer.
          Some changes are required in that case, such as "delete" button is not visible (pretty unlink is used instead).
        @param {boolean} [isDeletable=false]
    -->
    <t t-name="im_livechat.legacy.mail.Attachment">
        <t t-set="type" t-value="attachment.mimetype and attachment.mimetype.split('/').shift()"/>
        <div t-attf-class="o_attachment #{ editable ? 'o_attachment_editable' : '' } #{attachment.upload ? 'o_attachment_uploading' : ''}" t-att-title="attachment.name">
            <div class="o_attachment_wrap">
                <span t-if="!editable and isDeletable" class="fa fa-times o_attachment_delete_cross" t-att-title="'Delete ' + attachment.name" t-att-data-id="attachment.id" t-att-data-name="attachment.name"/>
                <t t-set="has_preview" t-value="type == 'image' or type == 'video' or attachment.mimetype == 'application/pdf'"/>
                <t t-set="ext" t-value="attachment.filename.split('.').pop()"/>

                <div t-attf-class="o_image_box float-left #{has_preview ? 'o_attachment_view' : ''}" t-att-data-id="attachment.id">
                    <div t-if="has_preview"
                         class="o_image o_hover"
                         t-att-style="type == 'image' ? 'background-image:url(/web/image/' + attachment.id + '/38x38/?crop=true' : '' "
                         t-att-data-mimetype="attachment.mimetype">
                    </div>
                    <a t-elif="!editable" t-att-href='attachment.url' t-att-title="'Download ' + attachment.name" aria-label="Download">
                        <span class="o_image o_hover" t-att-data-mimetype="attachment.mimetype" t-att-data-ext="ext"/>
                    </a>
                    <span t-else="" class="o_image" t-att-data-mimetype="attachment.mimetype" t-att-data-ext="ext" role="img" aria-label="Document not downloadable"/>
                </div>

                <div class="caption">
                    <span t-if="has_preview or editable" t-attf-class="ml4 #{has_preview? 'o_attachment_view' : ''}" t-att-data-id="attachment.id"><t t-esc='attachment.name'/></span>
                    <a t-else="" class="ml4" t-att-href="attachment.url" t-att-title="'Download ' + attachment.name"><t t-esc='attachment.name'/></a>
                </div>
                <div t-if="editable" class="caption small">
                    <b t-attf-class="ml4 small text-uppercase #{has_preview? 'o_attachment_view' : ''}" t-att-data-id="attachment.id"><t t-esc="ext"/></b>
                    <div class="progress o_attachment_progress_bar">
                        <div class="progress-bar progress-bar-striped active" style="width: 100%">Uploading</div>
                    </div>
                </div>
                <div t-if="!editable" class="caption small">
                    <b t-if="has_preview" class="ml4 small text-uppercase o_attachment_view" t-att-data-id="attachment.id"><t t-esc="ext"/></b>
                    <a t-else="" class="ml4 small text-uppercase" t-att-href="attachment.url" t-att-title="'Download ' + attachment.name"><b><t t-esc='ext'/></b></a>
                    <a class="ml4 o_attachment_download float-right" t-att-title="'Download ' + attachment.name" t-att-href='attachment.url'><i t-attf-class="fa fa-download" role="img" aria-label="Download"/></a>
                </div>
                <div t-if="editable" class="o_attachment_uploaded"><i class="text-success fa fa-check" role="img" aria-label="Uploaded" title="Uploaded"/></div>
                <div t-if="editable" class="o_attachment_delete" t-att-data-id="attachment.id"><span class="text-white" role="img" aria-label="Delete" title="Delete">×</span></div>
            </div>
        </div>
    </t>

    <!--
        @param {Object} options
        @param {boolean} [options.loadMoreOnScroll]
    -->
     <t t-name="im_livechat.legacy.mail.widget.Thread.LoadMore">
        <div class="o_thread_show_more">
            <t t-if="options.loadMoreOnScroll">
                <span><i class="fa fa-circle-o-notch fa-spin" role="img" aria-label="Please wait" title="Please wait"/> Loading older messages... </span>
            </t>
            <t t-else="">
                <button class="btn btn-link">-------- Show older messages --------</button>
            </t>
        </div>
    </t>

    <!--
        @param {mail.model.Thread} thread with typing feature
    -->
    <t t-name="im_livechat.legacy.mail.ThreadTypingIcon">
        <span class="o_mail_thread_typing_icon" t-att-title="thread.getTypingMembersToText()">
            <span class="o_mail_thread_typing_icon_dot"/>
            <span class="o_mail_thread_typing_icon_dot"/>
            <span class="o_mail_thread_typing_icon_dot"/>
        </span>
    </t>

</templates>

```

## File: static\src\models\chat_window\chat_window.js

```javascript
/** @odoo-module **/

import { registerInstancePatchModel } from '@mail/model/model_core';

registerInstancePatchModel('mail.chat_window', 'im_livechat/static/src/models/chat_window/chat_window.js', {

    /**
     * @override
     */
    close({ notifyServer } = {}) {
        if (
            this.thread &&
            this.thread.model === 'mail.channel' &&
            this.thread.channel_type === 'livechat' &&
            this.thread.cache.isLoaded &&
            this.thread.messages.length === 0
        ) {
            notifyServer = true;
            this.thread.unpin();
        }
        this._super({ notifyServer });
    }
});

```

## File: static\src\models\composer_view\composer_view.js

```javascript
/** @odoo-module **/

import { registerInstancePatchModel } from '@mail/model/model_core';

registerInstancePatchModel('mail.composer_view', 'im_livechat/static/src/models/composer_view/composer_view.js', {

    /**
     * @override
     */
    _computeHasDropZone() {
        if (this.composer.thread && this.composer.thread.channel_type === 'livechat') {
            return false;
        }
        return this._super();
    },
});

```

## File: static\src\models\discuss\discuss.js

```javascript
/** @odoo-module **/

import { registerFieldPatchModel, registerInstancePatchModel } from '@mail/model/model_core';
import { one2one } from '@mail/model/model_field';

registerInstancePatchModel('mail.discuss', 'im_livechat/static/src/models/discuss/discuss.js', {

    /**
     * @override
     */
    onInputQuickSearch(value) {
        if (!this.sidebarQuickSearchValue) {
            this.categoryLivechat.open();
        }
        return this._super(value);
    },
});

registerFieldPatchModel('mail.discuss', 'im_livechat/static/src/models/discuss/discuss.js', {
    /**
     * Discuss sidebar category for `livechat` channel threads.
     */
    categoryLivechat: one2one('mail.discuss_sidebar_category', {
        inverse: 'discussAsLivechat',
        isCausal: true,
    }),
});

```

## File: static\src\models\discuss_sidebar_category\discuss_sidebar_category.js

```javascript
/** @odoo-module **/

import { registerFieldPatchModel, registerIdentifyingFieldsPatch } from '@mail/model/model_core';
import { one2one } from '@mail/model/model_field';

registerFieldPatchModel('mail.discuss_sidebar_category', 'im_livechat', {
    discussAsLivechat: one2one('mail.discuss', {
        inverse: 'categoryLivechat',
        readonly: true,
    }),
});

registerIdentifyingFieldsPatch('mail.discuss_sidebar_category', 'im_livechat', identifyingFields => {
    identifyingFields[0].push('discussAsLivechat');
});

```

## File: static\src\models\discuss_sidebar_category_item\discuss_sidebar_category_item.js

```javascript
/** @odoo-module **/

import { registerInstancePatchModel } from '@mail/model/model_core';

registerInstancePatchModel('mail.discuss_sidebar_category_item', 'im_livechat/static/src/models/discuss_sidebar_category_item/discuss_sidebar_category_item.js', {

    //----------------------------------------------------------------------
    // Private
    //----------------------------------------------------------------------

    /**
     * @override
     */
    _computeAvatarUrl() {
        if (this.channelType === 'livechat') {
            if (this.channel.correspondent && this.channel.correspondent.id > 0) {
                return this.channel.correspondent.avatarUrl;
            }
        }
        return this._super();
    },
    /**
     * @override
     */
    _computeCategoryCounterContribution() {
        switch (this.channel.channel_type) {
            case 'livechat':
                return this.channel.localMessageUnreadCounter > 0 ? 1 : 0;
        }
        return this._super();
    },
    /**
     * @override
     */
    _computeCounter() {
        if (this.channelType === 'livechat') {
            return this.channel.localMessageUnreadCounter;
        }
        return this._super();
    },

    /**
     * @override
     */
    _computeHasUnpinCommand() {
        if (this.channelType === 'livechat') {
            return !this.channel.localMessageUnreadCounter;
        }
        return this._super();
    },

    /**
     * @override
     */
    _computeHasThreadIcon() {
        if (this.channelType === 'livechat') {
            return false;
        }
        return this._super();
    },

});

```

## File: static\src\models\message\message.js

```javascript
/** @odoo-module **/

import {
    registerClassPatchModel,
    registerInstancePatchModel,
} from '@mail/model/model_core';

registerClassPatchModel('mail.message', 'im_livechat/static/src/models/message/message.js', {
    /**
     * @override
     */
    convertData(data) {
        const data2 = this._super(data);
        if ('author_id' in data) {
            if (data.author_id[2]) {
                // flux specific for livechat, a 3rd param is livechat_username
                // and means 2nd param (display_name) should be ignored
                data2.author = [
                    ['insert-and-replace', {
                        id: data.author_id[0],
                        livechat_username: data.author_id[2],
                    }],
                ];
            }
        }
        return data2;
    },
});
registerInstancePatchModel('mail.message', 'im_livechat/static/src/models/message/message.js', {

    //----------------------------------------------------------------------
    // Private
    //----------------------------------------------------------------------

    /**
     * @override
     */
    _computeHasReactionIcon() {
        if (this.originThread && this.originThread.channel_type === 'livechat') {
            return false;
        }
        return this._super();
    },
});

```

## File: static\src\models\message_action_list\message_action_list.js

```javascript
/** @odoo-module **/

import { registerInstancePatchModel } from '@mail/model/model_core';

registerInstancePatchModel('mail.message_action_list', 'im_livechat/static/src/models/message_action_list/message_action_list.js', {

    //----------------------------------------------------------------------
    // Private
    //----------------------------------------------------------------------

    /**
     * @override
     */
    _computeHasReplyIcon() {
        if (
            this.message &&
            this.message.originThread &&
            this.message.originThread.channel_type === 'livechat'
        ) {
            return false;
        }
        return this._super();
    }
});

```

## File: static\src\models\messaging_initializer\messaging_initializer.js

```javascript
/** @odoo-module **/

import { registerInstancePatchModel } from '@mail/model/model_core';
import { insert, insertAndReplace } from '@mail/model/model_field_command';

registerInstancePatchModel('mail.messaging_initializer', 'im_livechat/static/src/models/messaging_initializer/messaging_initializer.js', {
    /**
     * @override
     * @param {Object} resUsersSettings
     * @param {boolean} resUsersSettings.is_discuss_sidebar_category_livechat_open
     */
    _initResUsersSettings({ is_discuss_sidebar_category_livechat_open }) {
        this.messaging.discuss.update({
            categoryLivechat: insertAndReplace({
                isServerOpen: is_discuss_sidebar_category_livechat_open,
                name: this.env._t("Livechat"),
                serverStateKey: 'is_discuss_sidebar_category_livechat_open',
                sortComputeMethod: 'last_action',
                supportedChannelTypes: ['livechat'],
            }),
        });
        this._super(...arguments);
    },

    /**
     * @override
     * @param {Object[]} [param0.channel_livechat=[]]
     */
    _initCommands() {
        this._super();
        this.messaging.update({
            commands: insert({
                channel_types: ['livechat'],
                help: this.env._t("See 15 last visited pages"),
                methodName: 'execute_command_history',
                name: "history",
            }),
        });
    },
});

```

## File: static\src\models\messaging_notification_handler\messaging_notification_handler.js

```javascript
/** @odoo-module **/

import { registerInstancePatchModel } from '@mail/model/model_core';

registerInstancePatchModel('mail.messaging_notification_handler', 'im_livechat/static/src/models/messaging_notification_handler/messaging_notification_handler.js', {

    //----------------------------------------------------------------------
    // Private
    //----------------------------------------------------------------------

    /**
     * @override
     * @param {object} settings
     * @param {boolean} [settings.is_discuss_sidebar_category_livechat_open]
    */
    _handleNotificationResUsersSettings(settings) {
        if ('is_discuss_sidebar_category_livechat_open' in settings) {
            this.messaging.discuss.categoryLivechat.update({
                isServerOpen: settings.is_discuss_sidebar_category_livechat_open,
            });
        }
        this._super(settings);
    },

    /**
     * @override
     */
    _handleNotificationChannelPartnerTypingStatus({ channel_id, is_typing, livechat_username, partner_id, partner_name }) {
        const channel = this.messaging.models['mail.thread'].findFromIdentifyingData({
            id: channel_id,
            model: 'mail.channel',
        });
        if (!channel) {
            return;
        }
        let partnerId;
        let partnerName;
        if (this.messaging.publicPartners.some(publicPartner => publicPartner.id === partner_id)) {
            // Some shenanigans that this is a typing notification
            // from public partner.
            partnerId = channel.correspondent.id;
            partnerName = channel.correspondent.name;
        } else {
            partnerId = partner_id;
            partnerName = partner_name;
        }
        const data = {
            channel_id,
            is_typing,
            partner_id: partnerId,
            partner_name: partnerName,
        };
        if (livechat_username) {
            // flux specific, livechat_username is returned instead of name for livechat channels
            delete data.partner_name; // value still present for API compatibility in stable
            this.models['mail.partner'].insert({
                id: partnerId,
                livechat_username: livechat_username,
            });
        }
        this._super(data);
    },
});

```

## File: static\src\models\partner\partner.js

```javascript
/** @odoo-module **/

import { registerClassPatchModel, registerFieldPatchModel } from '@mail/model/model_core';
import { attr } from '@mail/model/model_field';

let nextPublicId = -1;

registerClassPatchModel('mail.partner', 'im_livechat/static/src/models/partner/partner.js', {

    //----------------------------------------------------------------------
    // Public
    //----------------------------------------------------------------------

    convertData(data) {
        const data2 = this._super(data);
        if ('livechat_username' in data) {
            // flux specific, if livechat username is present it means `name`,
            // `email` and `im_status` contain `false` even though their value
            // might actually exist. Remove them from data2 to avoid overwriting
            // existing value (that could be known through other means).
            delete data2.name;
            delete data2.email;
            delete data2.im_status;
            data2.livechat_username = data.livechat_username;
        }
        return data2;
    },
    getNextPublicId() {
        const id = nextPublicId;
        nextPublicId -= 1;
        return id;
    },
});

registerFieldPatchModel('mail.partner', 'im_livechat/static/src/models/partner/partner.js', {
    /**
     * States the specific name of this partner in the context of livechat.
     * Either a string or undefined.
     */
    livechat_username: attr(),
});

```

## File: static\src\models\thread\thread.js

```javascript
/** @odoo-module **/

import {
    registerClassPatchModel,
    registerInstancePatchModel,
} from '@mail/model/model_core';
import { insert, link, unlink } from '@mail/model/model_field_command';

registerClassPatchModel('mail.thread', 'im_livechat/static/src/models/thread/thread.js', {

    //----------------------------------------------------------------------
    // Public
    //----------------------------------------------------------------------

    /**
     * @override
     */
    convertData(data) {
        const data2 = this._super(data);
        if ('livechat_visitor' in data && data.livechat_visitor) {
            if (!data2.members) {
                data2.members = [];
            }
            // `livechat_visitor` without `id` is the anonymous visitor.
            if (!data.livechat_visitor.id) {
                /**
                 * Create partner derived from public partner and replace the
                 * public partner.
                 *
                 * Indeed the anonymous visitor is registered as a member of the
                 * channel as the public partner in the database to avoid
                 * polluting the contact list with many temporary partners.
                 *
                 * But the issue with public partner is that it is the same
                 * record for every livechat, whereas every correspondent should
                 * actually have its own visitor name, typing status, etc.
                 *
                 * Due to JS being temporary by nature there is no such notion
                 * of polluting the database, it is therefore acceptable and
                 * easier to handle one temporary partner per channel.
                 */
                data2.members.push(unlink(this.messaging.publicPartners));
                const partner = this.messaging.models['mail.partner'].create(
                    Object.assign(
                        this.messaging.models['mail.partner'].convertData(data.livechat_visitor),
                        { id: this.messaging.models['mail.partner'].getNextPublicId() }
                    )
                );
                data2.members.push(link(partner));
                data2.correspondent = link(partner);
            } else {
                const partnerData = this.messaging.models['mail.partner'].convertData(data.livechat_visitor);
                data2.members.push(insert(partnerData));
                data2.correspondent = insert(partnerData);
            }
        }
        return data2;
    },
});

registerInstancePatchModel('mail.thread', 'im_livechat/static/src/models/thread/thread.js', {
    //----------------------------------------------------------------------
    // Public
    //----------------------------------------------------------------------

    /**
     * @override
     */
    getMemberName(partner) {
        if (this.channel_type === 'livechat' && partner.livechat_username) {
            return partner.livechat_username;
        }
        return this._super(partner);
    },

    //----------------------------------------------------------------------
    // Private
    //----------------------------------------------------------------------

    /**
     * @override
     */
    _computeCorrespondent() {
        if (this.channel_type === 'livechat') {
            // livechat correspondent never change: always the public member.
            return;
        }
        return this._super();
    },
    /**
     * @override
     */
    _computeDisplayName() {
        if (this.channel_type === 'livechat' && this.correspondent) {
            if (this.correspondent.country) {
                return `${this.correspondent.nameOrDisplayName} (${this.correspondent.country.name})`;
            }
            return this.correspondent.nameOrDisplayName;
        }
        return this._super();
    },
    /**
     * @override
     */
    _computeHasInviteFeature() {
        if (this.channel_type === 'livechat') {
            return true;
        }
        return this._super();
    },
    /**
     * @override
     */
    _computeHasMemberListFeature() {
        if (this.channel_type === 'livechat') {
            return true;
        }
        return this._super();
    },
    /**
     * @override
     */
    _computeIsChatChannel() {
        return this.channel_type === 'livechat' || this._super();
    },
    /**
     * @override
     */
    _getDiscussSidebarCategory() {
        switch (this.channel_type) {
            case 'livechat':
                return this.messaging.discuss.categoryLivechat;
        }
        return this._super();
    }
});

```

## File: views\digest_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="im_livechat.digest_digest_view_form_inherit" model="ir.ui.view">
        <field name="name">im.livechat.digest.digest.view.form.inherit</field>
        <field name="model">digest.digest</field>
        <field name="inherit_id" ref="digest.digest_digest_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//group[@name='kpi_general']" position="after">
                <group name="kpi_im_livechat" string="Live Chat">
                    <field name="kpi_livechat_rating"/>
                    <field name="kpi_livechat_conversations"/>
                    <field name="kpi_livechat_response"/>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\im_livechat_channel_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <!--
            Integrate Livechat Conversation in the Discuss
        -->
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
                    <t t-out="channel.script_external"/>

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
            <link t-attf-href="{{url}}/im_livechat/external_lib.css" rel="stylesheet"/>
            <!-- js of all the required lib (internal and external) -->
            <script t-attf-src="{{url}}/im_livechat/external_lib.js" type="text/javascript" />
            <!-- the loader -->
            <script t-attf-src="{{url}}/im_livechat/loader/{{channel_id}}" type="text/javascript"/>
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
                        var im_livechat = require('im_livechat.legacy.im_livechat.im_livechat');
                        var button = new im_livechat.LivechatButton(
                            rootWidget,
                            "<t t-esc="info['server_url']"/>",
                            <t t-out="json.dumps(info.get('options', {}))"/>
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
                    <field name="rating_count"/>
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
                                            <div t-if="record.rating_count.raw_value &gt; 0" class="float-right">
                                                <a name="action_view_rating" type="object" tabindex="10">
                                                    <i class="fa fa-smile-o text-success" title="Percentage of happy ratings" role="img" aria-label="Happy face"/>
                                                    <t t-esc="record.rating_percentage_satisfaction.raw_value"/>%
                                               </a>
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
                <form js_class="im_livechat_channel_form_view_js">
                    <header>
                        <button type="object" name="action_join" class="oe_highlight" string="Join Channel" attrs='{"invisible": [["are_you_inside", "=", True]]}'/>
                        <button type="object" name="action_quit" string="Leave Channel" attrs='{"invisible": [["are_you_inside", "=", False]]}'/>
                        <field name="are_you_inside" invisible="1"/>
                    </header>
                    <sheet>
                        <field name="rating_count" invisible="1"/>
                        <div class="oe_button_box" name="button_box">
                            <button class="oe_stat_button" type="action" attrs="{'invisible':[('nbr_channel','=', 0)]}" name="%(mail_channel_action_from_livechat_channel)d" icon="fa-comments">
                                <field string="Sessions" name="nbr_channel" widget="statinfo"/>
                            </button>
                            <button name="action_view_rating" attrs="{'invisible':[('rating_count', '=', 0)]}" class="oe_stat_button" type="object" icon="fa-smile-o">
                                <field string="% Happy" name="rating_percentage_satisfaction" widget="statinfo"/>
                            </button>
                        </div>
                        <field name="image_128" widget="image" class="oe_avatar"/>
                        <div class="oe_title">
                            <label for="name"/>
                            <h1>
                                <field name="name" placeholder="e.g. YourWebsite.com"/>
                            </h1>
                        </div>
                        <notebook>
                            <page string="Operators" name="operators">
                                <group>
                                    <field name="user_ids" nolabel="1" colspan="2" domain="[['groups_id', 'not in', %(base.group_portal)d]]">
                                        <kanban>
                                            <field name="id"/>
                                            <field name="name"/>
                                            <templates>
                                                <t t-name="kanban-box">
                                                    <div class="oe_kanban_global_click">
                                                        <div class="o_kanban_image">
                                                            <img t-att-src="kanban_image('res.users', 'avatar_1024', record.id.raw_value)" alt="User"/>
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
                            <page string="Options" name="options">
                                <group>
                                    <group string="Livechat Button">
                                        <field name="button_text"/>
                                        <label for="button_background_color" string="Livechat Button Color" />
                                        <div class="o_livechat_layout_colors">
                                            <field name="button_background_color" widget="color" class="mb-4 o_im_livechat_field_widget_color"/>
                                            <field name="button_text_color" widget="color" class="mb-4 o_im_livechat_field_widget_color"/>
                                            <button class="btn btn-link oe_edit_only o_im_livechat_channel_form_button_colors_reset_button" aria-label="Reset to default colors" title="Reset to default colors">
                                                <span class="fa fa-refresh mb-4"/>
                                            </button>
                                        </div>
                                    </group>
                                    <group string="Livechat Window">
                                        <field name="default_message" placeholder="e.g. Hello, how may I help you?"/>
                                        <field name="input_placeholder"/>
                                        <label for="header_background_color" string="Channel Header Color" />
                                        <div class="o_livechat_layout_colors">
                                            <field name="header_background_color" widget="color" class="mb-4 o_im_livechat_field_widget_color"/>
                                            <field name="title_color" widget="color" class="mb-4 o_im_livechat_field_widget_color"/>
                                            <button class="btn btn-link oe_edit_only o_im_livechat_channel_form_chat_window_colors_reset_button" aria-label="Reset to default colors" title="Reset to default colors">
                                                <span class="fa fa-refresh mb-4"/>
                                            </button>
                                        </div>
                                    </group>
                                </group>
                            </page>
                            <page string="Channel Rules" name="channel_rules">
                                <span class="text-muted">Define rules for your live support channel. You can apply an action for the given URL, and per country.<br />To identify the country, GeoIP must be installed on your server, otherwise, the countries of the rule will not be taken into account.</span>
                                <group>
                                    <field name="rule_ids" nolabel="1"/>
                                </group>
                            </page>
                            <page string="Widget" name="configuration_widget">
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
                                        <p>For websites built with the Odoo CMS, go to Website > Configuration > Settings and select the Website Live Chat Channel you want to add to your website.</p>
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
                            <field name="regex_url" placeholder="e.g. /contactus"/>
                            <label for="auto_popup_timer" class="oe_inline" attrs="{'invisible': [('action', '!=', 'auto_popup')]}"/>
                            <div class="oe_inline" attrs="{'invisible': [('action', '!=', 'auto_popup')]}">
                                <field name="auto_popup_timer" class="oe_inline"/> seconds
                            </div>
                            <field name="country_ids" widget="many2many_tags" options="{'no_open': True, 'no_create': True}"/>
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
            sequence="240"/>

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
                    <field name="message_ids" string="# Messages"/>
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
            <field name="domain">[('livechat_channel_id', 'in', [active_id]), ('has_message', '=', True)]</field>
            <field name="context">{
                'search_default_livechat_channel_id': [active_id],
                'default_livechat_channel_id': active_id,
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

    <record id="rating_rating_view_form_livechat" model="ir.ui.view">
        <field name="name">rating.rating.form.livechat</field>
        <field name="model">rating.rating</field>
        <field name="inherit_id" ref="rating.rating_rating_view_form"/>
        <field name="priority">32</field>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='rating_image_container']" position="replace">
                <field name="rating_text" string="Rating"/>
            </xpath>
        </field>
    </record>

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
                <filter string="Livechat Channel" name="groupby_livechat_channel" context="{'group_by': 'parent_res_name'}"/>
            </xpath>
            <xpath expr="/search" position="inside">
                <filter string="This Week" name="creation_date_filter" domain="[
                ('create_date', '>=', (datetime.datetime.combine(context_today() + relativedelta(weeks=-1,days=1,weekday=0), datetime.time(0,0,0)).to_utc()).strftime('%Y-%m-%d %H:%M:%S')),
                ('create_date', '&lt;', (datetime.datetime.combine(context_today() + relativedelta(days=1,weekday=0), datetime.time(0,0,0)).to_utc()).strftime('%Y-%m-%d %H:%M:%S'))]"/>
            </xpath>
        </field>
    </record>

    <record id="rating_rating_action_livechat" model="ir.actions.act_window">
        <field name="name">Ratings for livechat channel</field>
        <field name="res_model">rating.rating</field>
        <field name="view_mode">kanban,tree,graph,pivot,form</field>
        <field name="domain">[('parent_res_model', '=', 'im_livechat.channel'), ('consumed','=',True)]</field>
        <field name="search_view_id" ref="rating_rating_view_search_livechat"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                There is no rating for this channel at the moment
            </p>
        </field>
        <field name="context">{'search_default_rating_last_7_days': 1}</field>
    </record>

    <record id="rating_rating_action_livechat_view_kanban" model="ir.actions.act_window.view">
        <field name="sequence" eval="1"/>
        <field name="view_mode">kanban</field>
        <field name="act_window_id" ref="rating_rating_action_livechat"/>
        <field name="view_id" ref="rating.rating_rating_view_kanban"/>
    </record>

    <record id="rating_rating_action_livechat_view_form" model="ir.actions.act_window.view">
        <field name="sequence">5</field>
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="rating_rating_action_livechat"/>
        <field name="view_id" ref="rating_rating_view_form_livechat"/>
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

    <record id="rating_rating_action_livechat_report_view_kanban" model="ir.actions.act_window.view">
        <field name="sequence">1</field>
        <field name="view_mode">kanban</field>
        <field name="act_window_id" ref="rating_rating_action_livechat_report"/>
        <field name="view_id" ref="rating.rating_rating_view_kanban"/>
    </record>

    <record id="rating_rating_action_livechat_report_view_form" model="ir.actions.act_window.view">
        <field name="sequence">5</field>
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="rating_rating_action_livechat_report"/>
        <field name="view_id" ref="rating_rating_view_form_livechat"/>
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

