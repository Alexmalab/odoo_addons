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
        "data/mail_templates.xml",
        "data/im_livechat_channel_data.xml",
        "data/im_livechat_chatbot_data.xml",
        'data/digest_data.xml',
        'views/chatbot_script_answer_views.xml',
        'views/chatbot_script_step_views.xml',
        'views/chatbot_script_views.xml',
        "views/rating_rating_views.xml",
        "views/mail_channel_views.xml",
        "views/im_livechat_channel_views.xml",
        "views/im_livechat_channel_templates.xml",
        "views/im_livechat_chatbot_templates.xml",
        "views/res_users_views.xml",
        "views/digest_views.xml",
        "report/im_livechat_report_channel_views.xml",
        "report/im_livechat_report_operator_views.xml"
    ],
    'demo': [
        "data/im_livechat_channel_demo.xml",
        'data/mail_shortcode_demo.xml',
    ],
    'depends': ["mail", "rating", "digest", "utm"],
    'installable': True,
    'application': True,
    'assets': {
        'mail.assets_discuss_public': [
            'im_livechat/static/src/components/*/*',
        ],
        'web.assets_frontend': [
            ('include', 'im_livechat.assets_public_livechat'),
            'im_livechat/static/src/public/main.js',
            'im_livechat/static/src/services/*.js',
            'im_livechat/static/src/legacy/public_livechat_chatbot.js',
            'im_livechat/static/src/legacy/public_livechat.scss',
            'im_livechat/static/src/legacy/public_livechat_chatbot.scss',
            'mail/static/src/utils/*.js',
            'mail/static/src/js/utils.js',
            'mail/static/src/component_hooks/*.js',
            'mail/static/src/services/messaging_service.js',
        ],
        'web.assets_backend': [
            'im_livechat/static/src/js/colors_reset_button/*',
            'im_livechat/static/src/js/im_livechat_chatbot_steps_one2many.js',
            'im_livechat/static/src/js/im_livechat_chatbot_script_answers_m2m.js',
            'im_livechat/static/src/components/*/*.js',
            'im_livechat/static/src/scss/im_livechat_history.scss',
            'im_livechat/static/src/scss/im_livechat_form.scss',
            'im_livechat/static/src/components/*/*.xml',
        ],
        'web.tests_assets': [
            'im_livechat/static/tests/helpers/**/*.js',
        ],
        'web.qunit_suite_tests': [
            'im_livechat/static/tests/qunit_suite_tests/components/**/*.js',
        ],
        'web.assets_tests': [
            'im_livechat/static/tests/tours/**/*',
        ],
        'mail.assets_messaging': [
            'im_livechat/static/src/models/*.js',
        ],
        'im_livechat.assets_public_livechat': [
            ('include', 'mail.assets_core_messaging'),
            'im_livechat/static/src/legacy/models/*',
            'im_livechat/static/src/legacy/widgets/*',
            'im_livechat/static/src/legacy/widgets/*/*',
            'im_livechat/static/src/public_models/*.js',
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
            'web/static/src/legacy/js/promise_extension.js',
            'web/static/src/boot.js',
            'web/static/lib/owl/owl.js',
            'web/static/lib/owl/odoo_module.js',
            'web/static/src/owl2_compatibility/*.js',
            'web/static/src/legacy/legacy_component.js',
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
            'web/static/src/core/assets.js',
            'web/static/src/core/notifications/notification.js',
            'web/static/src/core/notifications/notification_container.js',
            'web/static/src/core/notifications/notification_service.js',
            'web/static/src/core/registry.js',
            'web/static/src/core/transition.js',
            'web/static/src/core/ui/block_ui.js',
            'web/static/src/core/ui/ui_service.js',
            'web/static/src/core/user_service.js',
            'web/static/src/core/utils/components.js',
            'web/static/src/core/utils/functions.js',
            'web/static/src/core/utils/hooks.js',
            'web/static/src/core/utils/numbers.js',
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
            'web/static/src/legacy/js/core/minimal_dom.js',
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
            'bus/static/src/im_status_service.js',
            'bus/static/src/multi_tab_service.js',
            'bus/static/src/services/bus_service.js',
            'bus/static/src/services/legacy/make_bus_service_to_legacy_env.js',
            'bus/static/src/workers/websocket_worker.js',
            'bus/static/src/workers/websocket_worker_utils.js',
            'mail/static/src/js/utils.js',
            'im_livechat/static/src/legacy/public_livechat_chatbot.js',

            ('include', 'web._assets_helpers'),

            'web/static/src/scss/pre_variables.scss',
            'web/static/lib/bootstrap/scss/_variables.scss',
            'im_livechat/static/src/scss/im_livechat_bootstrap.scss',
            'im_livechat/static/src/legacy/public_livechat.scss',
            'im_livechat/static/src/legacy/public_livechat_chatbot.scss',


            'web/static/src/core/utils/transitions.scss',

            'mail/static/src/utils/*.js',
            'mail/static/src/js/emojis.js',
            'mail/static/src/component_hooks/*.js',
            ('include', 'im_livechat.assets_public_livechat'),
            'mail/static/src/services/messaging_service.js',
            # Framework JS
            'bus/static/src/*.js',
            'bus/static/src/services/presence_service.js',
            'web/static/lib/luxon/luxon.js',
            'web/static/src/core/**/*',
            # FIXME: debug menu currently depends on webclient, once it doesn't we don't need to remove the contents of the debug folder
            ('remove', 'web/static/src/core/debug/**/*'),
            'web/static/src/env.js',
            'web/static/src/legacy/js/core/dialog.js',
            'web/static/src/legacy/js/core/owl_dialog.js',
            'web/static/src/legacy/js/core/misc.js',
            'web/static/src/legacy/js/fields/field_utils.js',

            'im_livechat/static/src/public/*.js',
            'im_livechat/static/src/services/*.js',
        ]
    },
    'license': 'LGPL-3',
}

```

## File: controllers\chatbot.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request
from odoo.tools import get_lang, is_html_empty, plaintext2html


class LivechatChatbotScriptController(http.Controller):
    @http.route('/chatbot/restart', type="json", auth="public", cors="*")
    def chatbot_restart(self, channel_uuid, chatbot_script_id):
        chatbot_language = self._get_chatbot_language()
        mail_channel = request.env['mail.channel'].sudo().with_context(lang=chatbot_language).search([('uuid', '=', channel_uuid)], limit=1)
        chatbot = request.env['chatbot.script'].browse(chatbot_script_id)
        if not mail_channel or not chatbot.exists():
            return None

        return mail_channel._chatbot_restart(chatbot).message_format()[0]

    @http.route('/chatbot/post_welcome_steps', type="json", auth="public", cors="*")
    def chatbot_post_welcome_steps(self, channel_uuid, chatbot_script_id):
        mail_channel = request.env['mail.channel'].sudo().search([('uuid', '=', channel_uuid)], limit=1)
        chatbot_language = self._get_chatbot_language()
        chatbot = request.env['chatbot.script'].sudo().with_context(lang=chatbot_language).browse(chatbot_script_id)
        if not mail_channel or not chatbot.exists():
            return None

        return chatbot._post_welcome_steps(mail_channel).message_format()

    @http.route('/chatbot/answer/save', type="json", auth="public", cors="*")
    def chatbot_save_answer(self, channel_uuid, message_id, selected_answer_id):
        mail_channel = request.env['mail.channel'].sudo().search([('uuid', '=', channel_uuid)], limit=1)
        chatbot_message = request.env['chatbot.message'].sudo().search([
            ('mail_message_id', '=', message_id),
            ('mail_channel_id', '=', mail_channel.id),
        ], limit=1)
        selected_answer = request.env['chatbot.script.answer'].sudo().browse(selected_answer_id)

        if not mail_channel or not chatbot_message or not selected_answer.exists():
            return

        if selected_answer in chatbot_message.script_step_id.answer_ids:
            chatbot_message.write({'user_script_answer_id': selected_answer_id})

    @http.route('/chatbot/step/trigger', type="json", auth="public", cors="*")
    def chatbot_trigger_step(self, channel_uuid, chatbot_script_id=None):
        chatbot_language = self._get_chatbot_language()
        mail_channel = request.env['mail.channel'].sudo().with_context(lang=chatbot_language).search([('uuid', '=', channel_uuid)], limit=1)
        if not mail_channel:
            return None

        next_step = False
        if mail_channel.chatbot_current_step_id:
            chatbot = mail_channel.chatbot_current_step_id.chatbot_script_id
            user_messages = mail_channel.message_ids.filtered(
                lambda message: message.author_id != chatbot.operator_partner_id
            )
            user_answer = request.env['mail.message'].sudo()
            if user_messages:
                user_answer = user_messages.sorted(lambda message: message.id)[-1]
            next_step = mail_channel.chatbot_current_step_id._process_answer(mail_channel, user_answer.body)
        elif chatbot_script_id:  # when restarting, we don't have a "current step" -> set "next" as first step of the script
            chatbot = request.env['chatbot.script'].sudo().with_context(lang=chatbot_language).browse(chatbot_script_id)
            if chatbot.exists():
                next_step = chatbot.script_step_ids[:1]

        if not next_step:
            return None

        posted_message = next_step._process_step(mail_channel)
        return {
            'chatbot_posted_message': posted_message.message_format()[0] if posted_message else None,
            'chatbot_step': {
                'chatbot_operator_found': next_step.step_type == 'forward_operator' and len(
                    mail_channel.channel_member_ids) > 2,
                'chatbot_script_step_id': next_step.id,
                'chatbot_step_answers': [{
                    'id': answer.id,
                    'label': answer.name,
                    'redirect_link': answer.redirect_link,
                } for answer in next_step.answer_ids],
                'chatbot_step_is_last': next_step._is_last_step(mail_channel),
                'chatbot_step_message': plaintext2html(next_step.message) if not is_html_empty(next_step.message) else False,
                'chatbot_step_type': next_step.step_type,
            }
        }

    @http.route('/chatbot/step/validate_email', type="json", auth="public", cors="*")
    def chatbot_validate_email(self, channel_uuid):
        mail_channel = request.env['mail.channel'].sudo().search([('uuid', '=', channel_uuid)], limit=1)
        if not mail_channel or not mail_channel.chatbot_current_step_id:
            return None

        chatbot = mail_channel.chatbot_current_step_id.chatbot_script_id
        user_messages = mail_channel.message_ids.filtered(
            lambda message: message.author_id != chatbot.operator_partner_id
        )

        if user_messages:
            user_answer = user_messages.sorted(lambda message: message.id)[-1]
            result = chatbot._validate_email(user_answer.body, mail_channel)

            if result['posted_message']:
                result['posted_message'] = result['posted_message'].message_format()[0]

        return result

    def _get_chatbot_language(self):
        return request.httprequest.cookies.get('frontend_lang', request.env.user.lang or get_lang(request.env).code)

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.exceptions import NotFound

from odoo import http, tools, _
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

        stream = request.env['ir.binary']._get_stream_from(mock_attachment)
        return stream.get_response()

    @http.route('/im_livechat/load_templates', type='json', auth='none', cors="*")
    def load_templates(self, **kwargs):
        templates = self._livechat_templates_get()
        return [tools.file_open(tmpl, 'rb').read() for tmpl in templates]

    def _livechat_templates_get(self):
        return [
            'im_livechat/static/src/legacy/widgets/feedback/feedback.xml',
            'im_livechat/static/src/legacy/widgets/public_livechat_window/public_livechat_window.xml',
            'im_livechat/static/src/legacy/widgets/public_livechat_view/public_livechat_view.xml',
            'im_livechat/static/src/legacy/public_livechat_chatbot.xml',
        ]

    @http.route('/im_livechat/support/<int:channel_id>', type='http', auth='public')
    def support_page(self, channel_id, **kwargs):
        channel = request.env['im_livechat.channel'].sudo().browse(channel_id)
        return request.render('im_livechat.support_page', {'channel': channel})

    @http.route('/im_livechat/loader/<int:channel_id>', type='http', auth='public')
    def loader(self, channel_id, **kwargs):
        username = kwargs.get("username", _("Visitor"))
        channel = request.env['im_livechat.channel'].sudo().browse(channel_id)
        info = channel.get_livechat_info(username=username)
        return request.render('im_livechat.loader', {'info': info}, headers=[('Content-Type', 'application/javascript')])

    @http.route('/im_livechat/init', type='json', auth="public", cors="*")
    def livechat_init(self, channel_id):
        operator_available = len(request.env['im_livechat.channel'].sudo().browse(channel_id)._get_available_users())
        rule = {}
        # find the country from the request
        country_id = False
        country_code = request.geoip.get('country_code')
        if country_code:
            country_id = request.env['res.country'].sudo().search([('code', '=', country_code)], limit=1).id
        # extract url
        url = request.httprequest.headers.get('Referer')
        # find the first matching rule for the given country and url
        matching_rule = request.env['im_livechat.channel.rule'].sudo().match_rule(channel_id, url, country_id)
        if matching_rule and (not matching_rule.chatbot_script_id or matching_rule.chatbot_script_id.script_step_ids):
            frontend_lang = request.httprequest.cookies.get('frontend_lang', request.env.user.lang or 'en_US')
            matching_rule = matching_rule.with_context(lang=frontend_lang)
            rule = {
                'action': matching_rule.action,
                'auto_popup_timer': matching_rule.auto_popup_timer,
                'regex_url': matching_rule.regex_url,
            }
            if matching_rule.chatbot_script_id.active and (not matching_rule.chatbot_only_if_no_operator or
               (not operator_available and matching_rule.chatbot_only_if_no_operator)) and matching_rule.chatbot_script_id.script_step_ids:
                chatbot_script = matching_rule.chatbot_script_id
                rule.update({'chatbot': chatbot_script._format_for_frontend()})
        return {
            'available_for_me': (rule and rule.get('chatbot'))
                                or operator_available and (not rule or rule['action'] != 'hide_button'),
            'rule': rule,
        }

    @http.route('/im_livechat/operator/<int:operator_id>/avatar',
        type='http', auth="public", cors="*")
    def livechat_operator_get_avatar(self, operator_id):
        """ Custom route allowing to retrieve an operator's avatar.

        This is done to bypass more complicated rules, notably 'website_published' when the website
        module is installed.

        Here, we assume that if you are a member of at least one im_livechat.channel, then it's ok
        to make your avatar publicly available.

        We also make the chatbot operator avatars publicly available. """

        is_livechat_member = False
        operator = request.env['res.partner'].sudo().browse(operator_id)
        if operator.exists():
            is_livechat_member = bool(request.env['im_livechat.channel'].sudo().search_count([
                ('user_ids', 'in', operator.user_ids.ids)
            ]))

        if not is_livechat_member:
            # we don't put chatbot operators as livechat members (because we don't have a user_id for them)
            is_livechat_member = bool(request.env['chatbot.script'].sudo().search_count([
                ('operator_partner_id', 'in', operator.ids)
            ]))

        return request.env['ir.binary']._get_image_stream_from(
            operator if is_livechat_member else request.env['res.partner'],
            field_name='avatar_128',
            placeholder='mail/static/src/img/smiley/avatar.jpg',
        ).get_response()

    @http.route('/im_livechat/get_session', type="json", auth='public', cors="*")
    def get_session(self, channel_id, anonymous_name, previous_operator_id=None, chatbot_script_id=None, persisted=True, **kwargs):
        user_id = None
        country_id = None
        # if the user is identifiy (eg: portal user on the frontend), don't use the anonymous name. The user will be added to session.
        if request.session.uid:
            user_id = request.env.user.id
            country_id = request.env.user.country_id.id
        else:
            # if geoip, add the country name to the anonymous name
            if request.geoip:
                # get the country of the anonymous person, if any
                country_code = request.geoip.get('country_code', "")
                country = request.env['res.country'].sudo().search([('code', '=', country_code)], limit=1) if country_code else None
                if country:
                    country_id = country.id

        if previous_operator_id:
            previous_operator_id = int(previous_operator_id)

        chatbot_script = False
        if chatbot_script_id:
            frontend_lang = request.httprequest.cookies.get('frontend_lang', request.env.user.lang or 'en_US')
            chatbot_script = request.env['chatbot.script'].sudo().with_context(lang=frontend_lang).browse(chatbot_script_id)

        return request.env["im_livechat.channel"].with_context(lang=False).sudo().browse(channel_id)._open_livechat_mail_channel(
            anonymous_name,
            previous_operator_id=previous_operator_id,
            chatbot_script=chatbot_script,
            user_id=user_id,
            country_id=country_id,
            persisted=persisted
        )

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
        channel = request.env['mail.channel'].sudo().search([('uuid', '=', uuid)])
        if not channel:
            raise NotFound()
        channel_member = channel.env['mail.channel.member'].search([('channel_id', '=', channel.id), ('partner_id', '=', request.env.user.partner_id.id)])
        if not channel_member:
            raise NotFound()
        channel_member._notify_typing(is_typing=is_typing)

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import chatbot
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
    <img src="https://download.odoocdn.com/digests/im_livechat/static/src/img/canned-responses.gif" class="illustration_border" />
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
            <field name="access_token">LIVECHAT_1</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_0"/>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_admin"/>
            <field eval="False" name="partner_id"/>
            <field eval="True" name="consumed"/>
        </record>
        <function model="mail.channel" name="rating_apply"
            eval="([ref('im_livechat.im_livechat_mail_channel_data_0')], 5, 'LIVECHAT_1', None, 'Good Job')"/>
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
            <field name="access_token">LIVECHAT_2</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_1"/>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_demo"/>
            <field eval="False" name="partner_id"/>
            <field eval="True" name="consumed"/>
        </record>
        <function model="mail.channel" name="rating_apply"
            eval="([ref('im_livechat.im_livechat_mail_channel_data_1')], 5, 'LIVECHAT_2', None, 'Super Job')"/>
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
            <field name="access_token">LIVECHAT_3</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_2"/>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_admin"/>
            <field name="partner_id" ref="base.partner_demo_portal"/>
            <field eval="True" name="consumed"/>
        </record>
        <function model="mail.channel" name="rating_apply"
            eval="([ref('im_livechat.im_livechat_mail_channel_data_2')], 5, 'LIVECHAT_3', None, 'Mega Job')"/>

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
            <field name="access_token">LIVECHAT_4</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_3"/>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_demo"/>
            <field name="partner_id" ref="base.partner_demo_portal"/>
            <field eval="True" name="consumed"/>
        </record>
        <function model="mail.channel" name="rating_apply"
            eval="([ref('im_livechat.im_livechat_mail_channel_data_3')], 5, 'LIVECHAT_4', None, 'Good Job')"/>

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
            <field name="access_token">LIVECHAT_5</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_4"/>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_admin"/>
            <field eval="False" name="partner_id"/>
            <field eval="True" name="consumed"/>
        </record>
        <function model="mail.channel" name="rating_apply"
            eval="([ref('im_livechat.im_livechat_mail_channel_data_4')], 5, 'LIVECHAT_5', None, 'Super Job')"/>
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
            <field name="access_token">LIVECHAT_6</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_5"/>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_admin"/>
            <field eval="False" name="partner_id"/>
            <field eval="True" name="consumed"/>
        </record>
        <function model="mail.channel" name="rating_apply"
            eval="([ref('im_livechat.im_livechat_mail_channel_data_5')], 5, 'LIVECHAT_6', None, 'Good Job')"/>
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
            <field name="body">Yes, of course, you can find it here: https://www.odoo.com/documentation/16.0/</field>
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
            <field name="access_token">LIVECHAT_7</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_6"/>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_admin"/>
            <field name="partner_id" ref="base.partner_demo_portal"/>
            <field eval="True" name="consumed"/>
        </record>
        <function model="mail.channel" name="rating_apply"
            eval="([ref('im_livechat.im_livechat_mail_channel_data_6')], 5, 'LIVECHAT_7', None, 'Super Job')"/>
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
            <field name="access_token">LIVECHAT_8</field>
            <field name="res_id" ref="im_livechat.im_livechat_mail_channel_data_7"/>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_demo"/>
            <field eval="False" name="partner_id"/>
            <field eval="True" name="consumed"/>
        </record>
        <function model="mail.channel" name="rating_apply"
            eval="([ref('im_livechat.im_livechat_mail_channel_data_7')], 5, 'LIVECHAT_8', None, 'Super Job')"/>
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
            <field name="access_token">LIVECHAT_9</field>
            <field name="res_model_id" ref="mail.model_mail_channel"/>
            <field name="rated_partner_id" ref="base.partner_admin"/>
            <field name="partner_id" ref="base.partner_admin"/>
            <field name="create_date" eval="datetime.now() - timedelta(days=1, seconds=-53)"/>
            <field name="res_id" ref="im_livechat.mail_channel_livechat_1"/>
        </record>
        <function model="mail.channel" name="rating_apply"
            eval="([ref('im_livechat.mail_channel_livechat_1')], 5, 'LIVECHAT_9', None, 'Super Job')"/>

    </data>
</odoo>

```

## File: data\im_livechat_chatbot_data.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data noupdate="1">

    <!--
        This provides a working chatbot for people to work with.
        It's placed into 'data' to give them a starting point.
        From that record, they can duplicate / adapt / delete / ...
    -->

    <record id="chatbot_script_welcome_bot" model="chatbot.script">
        <field name="title">Welcome Bot</field>
        <field name="image_1920" type="base64" file="mail/static/src/img/odoobot.png"/>
    </record>

    <record id="chatbot_script_welcome_step_welcome" model="chatbot.script.step">
        <field name="message">Welcome to CompanyName ! 👋</field>
        <field name="sequence">1</field>
        <field name="step_type">text</field>
        <field name="chatbot_script_id" ref="chatbot_script_welcome_bot"/>
    </record>

    <record id="chatbot_script_welcome_step_dispatch" model="chatbot.script.step">
        <field name="message">What are you looking for?</field>
        <field name="sequence">2</field>
        <field name="step_type">question_selection</field>
        <field name="chatbot_script_id" ref="chatbot_script_welcome_bot"/>
    </record>

    <record id="chatbot_script_welcome_step_dispatch_answer_pricing" model="chatbot.script.answer">
        <field name="name">I have a pricing question</field>
        <field name="sequence">1</field>
        <field name="script_step_id" ref="chatbot_script_welcome_step_dispatch"/>
    </record>

    <record id="chatbot_script_welcome_step_dispatch_answer_documentation" model="chatbot.script.answer">
        <field name="name">I am looking for your documentation</field>
        <field name="redirect_link">/</field>
        <field name="sequence">2</field>
        <field name="script_step_id" ref="chatbot_script_welcome_step_dispatch"/>
    </record>

    <record id="chatbot_script_welcome_step_dispatch_answer_just_looking" model="chatbot.script.answer">
        <field name="name">I am just looking around</field>
        <field name="sequence">3</field>
        <field name="script_step_id" ref="chatbot_script_welcome_step_dispatch"/>
    </record>

    <record id="chatbot_script_welcome_step_pricing" model="chatbot.script.step">
        <field name="message">Hmmm, let me check if I can find someone that could help you with that...</field>
        <field name="sequence">3</field>
        <field name="step_type">text</field>
        <field name="chatbot_script_id" ref="chatbot_script_welcome_bot"/>
        <field name="triggering_answer_ids" eval="[(4, ref('chatbot_script_welcome_step_dispatch_answer_pricing'))]"/>
    </record>

    <record id="chatbot_script_welcome_step_pricing_forward_operator" model="chatbot.script.step">
        <field name="sequence">4</field>
        <field name="step_type">forward_operator</field>
        <field name="chatbot_script_id" ref="chatbot_script_welcome_bot"/>
        <field name="triggering_answer_ids" eval="[(4, ref('chatbot_script_welcome_step_dispatch_answer_pricing'))]"/>
    </record>

    <record id="chatbot_script_welcome_step_pricing_noone_available" model="chatbot.script.step">
        <field name="message">Hu-ho, it looks like none of our operators are available 🙁</field>
        <field name="sequence">5</field>
        <field name="step_type">text</field>
        <field name="chatbot_script_id" ref="chatbot_script_welcome_bot"/>
        <field name="triggering_answer_ids" eval="[(4, ref('chatbot_script_welcome_step_dispatch_answer_pricing'))]"/>
    </record>

    <record id="chatbot_script_welcome_step_pricing_email" model="chatbot.script.step">
        <field name="message">Would you mind leaving your email address so that we can reach you back?</field>
        <field name="sequence">6</field>
        <field name="step_type">question_email</field>
        <field name="chatbot_script_id" ref="chatbot_script_welcome_bot"/>
        <field name="triggering_answer_ids" eval="[(4, ref('chatbot_script_welcome_step_dispatch_answer_pricing'))]"/>
    </record>

    <record id="chatbot_script_welcome_step_documentation_redirect" model="chatbot.script.step">
        <field name="message">And tadaaaa here you go! 🌟</field>
        <field name="sequence">7</field>
        <field name="step_type">text</field>
        <field name="chatbot_script_id" ref="chatbot_script_welcome_bot"/>
        <field name="triggering_answer_ids" eval="[(4, ref('chatbot_script_welcome_step_dispatch_answer_documentation'))]"/>
    </record>

    <record id="chatbot_script_welcome_step_documentation_exit" model="chatbot.script.step">
        <field name="message">If you need anything else, feel free to get back in touch</field>
        <field name="sequence">8</field>
        <field name="step_type">text</field>
        <field name="chatbot_script_id" ref="chatbot_script_welcome_bot"/>
        <field name="triggering_answer_ids" eval="[(4, ref('chatbot_script_welcome_step_dispatch_answer_documentation'))]"/>
    </record>

    <record id="chatbot_script_welcome_step_just_looking" model="chatbot.script.step">
        <field name="message">Please do! If there is anything we can help with, let us know</field>
        <field name="sequence">9</field>
        <field name="step_type">text</field>
        <field name="chatbot_script_id" ref="chatbot_script_welcome_bot"/>
        <field name="triggering_answer_ids" eval="[(4, ref('chatbot_script_welcome_step_dispatch_answer_just_looking'))]"/>
    </record>

</data></odoo>

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

## File: data\mail_templates.xml

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

## File: models\chatbot_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class ChatbotMailMessage(models.Model):
    """ Chatbot Mail Message
        We create a new model to store the related step to a mail.message and the user's answer.
        We do this in a new model to avoid bloating the 'mail.message' model.
    """

    _name = 'chatbot.message'
    _description = 'Chatbot Message'
    _order = 'create_date desc, id desc'
    _rec_name = 'mail_channel_id'

    mail_message_id = fields.Many2one('mail.message', string='Related Mail Message', required=True, ondelete="cascade")
    mail_channel_id = fields.Many2one('mail.channel', string='Discussion Channel', required=True, ondelete="cascade")
    script_step_id = fields.Many2one(
        'chatbot.script.step', string='Chatbot Step', required=True, ondelete='cascade')
    user_script_answer_id = fields.Many2one('chatbot.script.answer', string="User's answer", ondelete="set null")
    user_raw_answer = fields.Html(string="User's raw answer")

    _sql_constraints = [
        ('_unique_mail_message_id', 'unique (mail_message_id)',
         "A mail.message can only be linked to a single chatbot message"),
    ]

```

## File: models\chatbot_script.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models, fields
from odoo.tools import email_normalize, html2plaintext, is_html_empty, plaintext2html


class ChatbotScript(models.Model):
    _name = 'chatbot.script'
    _description = 'Chatbot Script'
    _inherit = ['image.mixin', 'utm.source.mixin']
    _rec_name = 'title'
    _order = 'title, id'

    # we keep a separate field for UI since name is manipulated by 'utm.source.mixin'
    title = fields.Char('Title', required=True, translate=True, default="Chatbot")
    active = fields.Boolean(default=True)
    image_1920 = fields.Image(related='operator_partner_id.image_1920', readonly=False)

    script_step_ids = fields.One2many('chatbot.script.step', 'chatbot_script_id',
        copy=True, string='Script Steps')
    operator_partner_id = fields.Many2one('res.partner', string='Bot Operator',
        ondelete='restrict', required=True, copy=False)
    livechat_channel_count = fields.Integer(string='Livechat Channel Count', compute='_compute_livechat_channel_count')
    first_step_warning = fields.Selection([
        ('first_step_operator', 'First Step Operator'),
        ('first_step_invalid', 'First Step Invalid'),
    ], compute="_compute_first_step_warning")

    def _compute_livechat_channel_count(self):
        channels_data = self.env['im_livechat.channel.rule'].read_group(
            [('chatbot_script_id', 'in', self.ids)], ['channel_id:count_distinct'], ['chatbot_script_id'])
        mapped_channels = {channel['chatbot_script_id'][0]: channel['channel_id'] for channel in channels_data}
        for script in self:
            script.livechat_channel_count = mapped_channels.get(script.id, 0)

    @api.depends('script_step_ids.step_type')
    def _compute_first_step_warning(self):
        for script in self:
            allowed_first_step_types = [
                'question_selection',
                'question_email',
                'question_phone',
                'free_input_single',
                'free_input_multi',
            ]
            welcome_steps = script.script_step_ids and script._get_welcome_steps()
            if welcome_steps and welcome_steps[-1].step_type == 'forward_operator':
                script.first_step_warning = 'first_step_operator'
            elif welcome_steps and welcome_steps[-1].step_type not in allowed_first_step_types:
                script.first_step_warning = 'first_step_invalid'
            else:
                script.first_step_warning = False

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        """ Correctly copy the 'triggering_answer_ids' field from the original script_step_ids to the clone.
        This needs to be done in post-processing to make sure we get references to the newly created
        answers from the copy instead of references to the answers of the original.

        This implementation assumes that the order of created steps and answers will be kept between
        the original and the clone, using 'zip()' to match the records between the two. """

        default = default or {}
        default['title'] = self.title + _(' (copy)')

        clone_chatbot_script = super().copy(default=default)
        if 'question_ids' in default:
            return clone_chatbot_script

        original_steps = self.script_step_ids.sorted()
        clone_steps = clone_chatbot_script.script_step_ids.sorted()

        answers_map = {}
        for clone_step, original_step in zip(clone_steps, original_steps):
            for clone_answer, original_answer in zip(clone_step.answer_ids.sorted(), original_step.answer_ids.sorted()):
                answers_map[original_answer] = clone_answer

        for clone_step, original_step in zip(clone_steps, original_steps):
            clone_step.write({
                'triggering_answer_ids': [
                    (4, answer.id)
                    for answer in [
                        answers_map[original_answer]
                        for original_answer
                        in original_step.triggering_answer_ids
                    ]
                ]
            })

        return clone_chatbot_script

    @api.model_create_multi
    def create(self, vals_list):
        operator_partners_values = [{
            'name': vals['title'],
            'image_1920': vals.get('image_1920', False),
            'active': False,
        } for vals in vals_list if 'operator_partner_id' not in vals and 'title' in vals]

        operator_partners = self.env['res.partner'].create(operator_partners_values)

        for vals, partner in zip(
            [vals for vals in vals_list if 'operator_partner_id' not in vals and 'title' in vals],
            operator_partners
        ):
            vals['operator_partner_id'] = partner.id

        return super().create(vals_list)

    def write(self, vals):
        res = super().write(vals)

        if 'title' in vals:
            self.operator_partner_id.write({'name': vals['title']})

        return res

    def _get_welcome_steps(self):
        """ Returns a sub-set of script_step_ids that only contains the "welcoming steps".
        We consider those as all the steps the bot will say before expecting a first answer from
        the end user.

        Example 1:
        - step 1 (question_selection): What do you want to do? - Create a Lead, -Create a Ticket
        - step 2 (text): Thank you for visiting our website!
        -> The welcoming steps will only contain step 1, since directly after that we expect an
        input from the user

        Example 2:
        - step 1 (text): Hello! I'm a bot!
        - step 2 (text): I am here to help lost users.
        - step 3 (question_selection): What do you want to do? - Create a Lead, -Create a Ticket
        - step 4 (text): Thank you for visiting our website!
        -> The welcoming steps will contain steps 1, 2 and 3.
        Meaning the bot will have a small monologue with himself before expecting an input from the
        end user.

        This is important because we need to display those welcoming steps in a special fashion on
        the frontend, since those are not inserted into the mail.channel as actual mail.messages,
        to avoid bloating the channels with bot messages if the end-user never interacts with it. """
        self.ensure_one()

        welcome_steps = self.env['chatbot.script.step']
        for step in self.script_step_ids:
            welcome_steps += step
            if step.step_type != 'text':
                break

        return welcome_steps

    def _post_welcome_steps(self, mail_channel):
        """ Welcome messages are only posted after the visitor's first interaction with the chatbot.
        See 'chatbot.script#_get_welcome_steps()' for more details.

        Side note: it is important to set the 'chatbot_current_step_id' on each iteration so that
        it's correctly set when going into 'mail_channel#_message_post_after_hook()'. """

        self.ensure_one()
        posted_messages = self.env['mail.message']

        for welcome_step in self._get_welcome_steps():
            mail_channel.chatbot_current_step_id = welcome_step.id

            if not is_html_empty(welcome_step.message):
                posted_messages += mail_channel.with_context(mail_create_nosubscribe=True).message_post(
                    author_id=self.operator_partner_id.id,
                    body=plaintext2html(welcome_step.message),
                    message_type='comment',
                    subtype_xmlid='mail.mt_comment',
                )

        return posted_messages

    def action_view_livechat_channels(self):
        self.ensure_one()
        action = self.env['ir.actions.act_window']._for_xml_id('im_livechat.im_livechat_channel_action')
        action['domain'] = [('rule_ids.chatbot_script_id', 'in', self.ids)]
        return action

    # --------------------------
    # Tooling / Misc
    # --------------------------

    def _format_for_frontend(self):
        """ Small utility method that formats the script into a dict usable by the frontend code. """
        self.ensure_one()

        return {
            'chatbot_script_id': self.id,
            'chatbot_name': self.title,
            'chatbot_operator_partner_id': self.operator_partner_id.id,
            'chatbot_welcome_steps': [
                step._format_for_frontend()
                for step in self._get_welcome_steps()
            ]
        }

    def _validate_email(self, email_address, mail_channel):
        email_address = html2plaintext(email_address)
        email_normalized = email_normalize(email_address)

        posted_message = False
        error_message = False
        if not email_normalized:
            error_message = _(
                "'%(input_email)s' does not look like a valid email. Can you please try again?",
                input_email=email_address
            )
            posted_message = mail_channel._chatbot_post_message(self, plaintext2html(error_message))

        return {
            'success': bool(email_normalized),
            'posted_message': posted_message,
            'error_message': error_message,
        }

```

## File: models\chatbot_script_answer.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields
from odoo.osv import expression

import textwrap


class ChatbotScriptAnswer(models.Model):
    _name = 'chatbot.script.answer'
    _description = 'Chatbot Script Answer'
    _order = 'script_step_id, sequence, id'

    name = fields.Char(string='Answer', required=True, translate=True)
    sequence = fields.Integer(string='Sequence', default=1)
    redirect_link = fields.Char('Redirect Link',
        help="The visitor will be redirected to this link upon clicking the option "
             "(note that the script will end if the link is external to the livechat website).")
    script_step_id = fields.Many2one(
        'chatbot.script.step', string='Script Step', required=True, ondelete='cascade')
    chatbot_script_id = fields.Many2one(related='script_step_id.chatbot_script_id')

    def name_get(self):
        if self._context.get('chatbot_script_answer_display_short_name'):
            return super().name_get()

        result = []
        for answer in self:
            answer_message = answer.script_step_id.message.replace('\n', ' ')
            shortened_message = textwrap.shorten(answer_message, width=26, placeholder=" [...]")

            result.append((
                answer.id,
                "%s: %s" % (shortened_message, answer.name)
            ))

        return result

    @api.model
    def _name_search(self, name='', args=None, operator='ilike', limit=100, name_get_uid=None):
        """
        Search the records whose name or step message are matching the ``name`` pattern.
        The chatbot_script_id is also passed to the context through the custom widget
        ('chatbot_triggering_answers_widget') This allows to only see the question_answer
        from the same chatbot you're configuring.
        """
        force_domain_chatbot_script_id = self.env.context.get('force_domain_chatbot_script_id')

        if name and operator == 'ilike':
            if not args:
                args = []

            # search on both name OR step's message (combined with passed args)
            name_domain = [('name', operator, name)]
            step_domain = [('script_step_id.message', operator, name)]
            domain = expression.AND([args, expression.OR([name_domain, step_domain])])

        else:
            domain = args or []

        if force_domain_chatbot_script_id:
            domain = expression.AND([domain, [('chatbot_script_id', '=', force_domain_chatbot_script_id)]])

        return self._search(domain, limit=limit, access_rights_uid=name_get_uid)

```

## File: models\chatbot_script_step.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models, fields
from odoo.exceptions import ValidationError
from odoo.fields import Command
from odoo.osv import expression
from odoo.tools import html2plaintext, is_html_empty, email_normalize, plaintext2html

from collections import defaultdict
from markupsafe import Markup


class ChatbotScriptStep(models.Model):
    _name = 'chatbot.script.step'
    _description = 'Chatbot Script Step'
    _order = 'sequence, id'
    _rec_name = 'message'

    message = fields.Text(string='Message', translate=True)
    sequence = fields.Integer(string='Sequence')
    chatbot_script_id = fields.Many2one(
        'chatbot.script', string='Chatbot', required=True, ondelete='cascade')
    step_type = fields.Selection([
        ('text', 'Text'),
        ('question_selection', 'Question'),
        ('question_email', 'Email'),
        ('question_phone', 'Phone'),
        ('forward_operator', 'Forward to Operator'),
        ('free_input_single', 'Free Input'),
        ('free_input_multi', 'Free Input (Multi-Line)'),
    ], default='text', required=True)
    # answers
    answer_ids = fields.One2many(
        'chatbot.script.answer', 'script_step_id',
        copy=True, string='Answers')
    triggering_answer_ids = fields.Many2many(
        'chatbot.script.answer', domain="[('script_step_id.sequence', '<', sequence)]",
        compute='_compute_triggering_answer_ids', readonly=False, store=True,
        copy=False,  # copied manually, see chatbot.script#copy
        string='Only If', help='Show this step only if all of these answers have been selected.')
    # forward-operator specifics
    is_forward_operator_child = fields.Boolean(compute='_compute_is_forward_operator_child')

    @api.depends('sequence')
    def _compute_triggering_answer_ids(self):
        for step in self.filtered('triggering_answer_ids'):
            update_command = [Command.unlink(answer.id) for answer in step.triggering_answer_ids
                                if answer.script_step_id.sequence >= step.sequence]
            if update_command:
                step.triggering_answer_ids = update_command

    @api.depends('sequence', 'triggering_answer_ids', 'chatbot_script_id.script_step_ids.triggering_answer_ids',
                 'chatbot_script_id.script_step_ids.answer_ids', 'chatbot_script_id.script_step_ids.sequence')
    def _compute_is_forward_operator_child(self):
        parent_steps_by_chatbot = {}
        for chatbot in self.chatbot_script_id:
            parent_steps_by_chatbot[chatbot.id] = chatbot.script_step_ids.filtered(
                lambda step: step.step_type in ['forward_operator', 'question_selection']
            ).sorted(lambda s: s.sequence, reverse=True)
        for step in self:
            parent_steps = parent_steps_by_chatbot[step.chatbot_script_id.id].filtered(
                lambda s: s.sequence < step.sequence
            )
            parent = step
            while True:
                parent = parent._get_parent_step(parent_steps)
                if not parent or parent.step_type == 'forward_operator':
                    break
            step.is_forward_operator_child = parent and parent.step_type == 'forward_operator'

    @api.model_create_multi
    def create(self, vals_list):
        """ Ensure we correctly assign sequences when creating steps.
        Indeed, sequences are very important within the script, and will break the whole flow if
        not correctly defined.

        This override will group created steps by chatbot_id and increment the sequence accordingly.
        It will also look for an existing step for that chatbot and resume from the highest sequence.

        This cannot be done in a default_value for the sequence field as we cannot search by
        runbot_id.
        It is also safer and more efficient to do it here (we can batch everything).

        It is still possible to manually pass the 'sequence' in the values, which will take priority. """

        vals_by_chatbot_id = {}
        for vals in vals_list:
            chatbot_id = vals.get('chatbot_script_id')
            if chatbot_id:
                step_values = vals_by_chatbot_id.get(chatbot_id, [])
                step_values.append(vals)
                vals_by_chatbot_id[chatbot_id] = step_values

        max_sequence_by_chatbot = {}
        if vals_by_chatbot_id:
            read_group_results = self.env['chatbot.script.step'].read_group(
                [('chatbot_script_id', 'in', list(vals_by_chatbot_id.keys()))],
                ['sequence:max'],
                ['chatbot_script_id']
            )

            max_sequence_by_chatbot = {
                read_group_result['chatbot_script_id'][0]: read_group_result['sequence']
                for read_group_result in read_group_results
            }

        for chatbot_id, step_vals in vals_by_chatbot_id.items():
            current_sequence = 0
            if chatbot_id in max_sequence_by_chatbot:
                current_sequence = max_sequence_by_chatbot[chatbot_id] + 1

            for vals in step_vals:
                if 'sequence' in vals:
                    current_sequence = vals.get('sequence')
                else:
                    vals['sequence'] = current_sequence
                    current_sequence += 1

        return super().create(vals_list)

    # --------------------------
    # Business Methods
    # --------------------------

    def _chatbot_prepare_customer_values(self, mail_channel, create_partner=True, update_partner=True):
        """ Common method that allows retreiving default customer values from the mail.channel
        following a chatbot.script.

        This method will return a dict containing the 'customer' values such as:
        {
            'partner': The created partner (see 'create_partner') or the partner from the
              environment if not public
            'email': The email extracted from the mail.channel messages
              (see step_type 'question_email')
            'phone': The phone extracted from the mail.channel messages
              (see step_type 'question_phone')
            'description': A default description containing the "Please contact me on" and "Please
              call me on" with the related email and phone numbers.
              Can be used as a default description to create leads or tickets for example.
        }

        :param record mail_channel: the mail.channel holding the visitor's conversation with the bot.
        :param bool create_partner: whether or not to create a res.partner is the current user is public.
          Defaults to True.
        :param bool update_partner: whether or not to set update the email and phone on the res.partner
          from the environment (if not a public user) if those are not set yet. Defaults to True.

        :return dict: a dict containing the customer values."""

        partner = False
        user_inputs = mail_channel._chatbot_find_customer_values_in_messages({
            'question_email': 'email',
            'question_phone': 'phone',
        })
        input_email = user_inputs.get('email', False)
        input_phone = user_inputs.get('phone', False)

        if self.env.user._is_public() and create_partner:
            partner = self.env['res.partner'].create({
                'name': input_email,
                'email': input_email,
                'phone': input_phone,
            })
        elif not self.env.user._is_public():
            partner = self.env.user.partner_id
            if update_partner:
                # update email/phone value from partner if not set
                update_values = {}
                if input_email and not partner.email:
                    update_values['email'] = input_email
                if input_phone and not partner.phone:
                    update_values['phone'] = input_phone
                if update_values:
                    partner.write(update_values)

        description = Markup('')
        if input_email:
            description += Markup('%s<strong>%s</strong><br>') % (_('Please contact me on: '), input_email)
        if input_phone:
            description += Markup('%s<strong>%s</strong><br>') % (_('Please call me on: '), input_phone)
        if description:
            description += Markup('<br>')

        return {
            'partner': partner,
            'email': input_email,
            'phone': input_phone,
            'description': description,
        }

    def _fetch_next_step(self, selected_answer_ids):
        """ Fetch the next step depending on the user's selected answers.
            If a step contains multiple triggering answers from the same step the condition between
            them must be a 'OR'. If is contains multiple triggering answers from different steps the
            condition between them must be a 'AND'.

            e.g:

            STEP 1 : A B
            STEP 2 : C D
            STEP 3 : E
            STEP 4 ONLY IF A B C E

            Scenario 1 (A C E):

            A in (A B) -> OK
            C in (C)   -> OK
            E in (E)   -> OK

            -> OK

            Scenario 2 (B D E):

            B in (A B) -> OK
            D in (C)   -> NOK
            E in (E)   -> OK

            -> NOK
        """
        self.ensure_one()
        domain = [('chatbot_script_id', '=', self.chatbot_script_id.id), ('sequence', '>', self.sequence)]
        if selected_answer_ids:
            domain = expression.AND([domain, [
                '|',
                ('triggering_answer_ids', '=', False),
                ('triggering_answer_ids', 'in', selected_answer_ids.ids)]])
        steps = self.env['chatbot.script.step'].search(domain)
        for step in steps:
            if not step.triggering_answer_ids:
                return step
            answers_by_step = defaultdict(list)
            for answer in step.triggering_answer_ids:
                answers_by_step[answer.script_step_id.id].append(answer)
            if all(any(answer in step_triggering_answers for answer in selected_answer_ids)
                   for step_triggering_answers in answers_by_step.values()):
                return step
        return self.env['chatbot.script.step']

    def _get_parent_step(self, all_parent_steps):
        """ Returns the first preceding step that matches either the triggering answers
         or the possible answers the user can select """
        self.ensure_one()

        if not self.chatbot_script_id.ids:
            return self.env['chatbot.script.step']

        for step in all_parent_steps:
            if step.sequence >= self.sequence:
                continue
            if self.triggering_answer_ids:
                if not (all(answer in self.triggering_answer_ids for answer in step.triggering_answer_ids) or
                        any(answer in self.triggering_answer_ids for answer in step.answer_ids)):
                    continue
            elif step.triggering_answer_ids:
                continue
            return step
        return self.env['chatbot.script.step']

    def _is_last_step(self, mail_channel=False):
        self.ensure_one()
        mail_channel = mail_channel or self.env['mail.channel']

        # if it's not a question and if there is no next step, then we end the script
        if self.step_type != 'question_selection' and not self._fetch_next_step(
           mail_channel.chatbot_message_ids.user_script_answer_id):
            return True

        return False

    def _process_answer(self, mail_channel, message_body):
        """ Method called when the user reacts to the current chatbot.script step.
        For most chatbot.script.step#step_types it simply returns the next chatbot.script.step of
        the script (see '_fetch_next_step').

        Some extra processing is done for steps of type 'question_email' and 'question_phone' where
        we store the user raw answer (the mail message HTML body) into the chatbot.message in order
        to be able to recover it later (see '_chatbot_prepare_customer_values').

        :param mail_channel:
        :param message_body:
        :return: script step to display next
        :rtype: 'chatbot.script.step' """

        self.ensure_one()

        user_text_answer = html2plaintext(message_body)
        if self.step_type == 'question_email' and not email_normalize(user_text_answer):
            # if this error is raised, display an error message but do not go to next step
            raise ValidationError(_('"%s" is not a valid email.', user_text_answer))

        if self.step_type in ['question_email', 'question_phone']:
            chatbot_message = self.env['chatbot.message'].search([
                ('mail_channel_id', '=', mail_channel.id),
                ('script_step_id', '=', self.id),
            ], limit=1)

            if chatbot_message:
                chatbot_message.write({'user_raw_answer': message_body})
                self.env.flush_all()

        return self._fetch_next_step(mail_channel.chatbot_message_ids.user_script_answer_id)

    def _process_step(self, mail_channel):
        """ When we reach a chatbot.step in the script we need to do some processing on behalf of
        the bot. Which is for most chatbot.script.step#step_types just posting the message field.

        Some extra processing may be required for special step types such as 'forward_operator',
        'create_lead', 'create_ticket' (in their related bridge modules).
        Those will have a dedicated processing method with specific docstrings.

        Returns the mail.message posted by the chatbot's operator_partner_id. """

        self.ensure_one()
        # We change the current step to the new step
        mail_channel.chatbot_current_step_id = self.id

        if self.step_type == 'forward_operator':
            return self._process_step_forward_operator(mail_channel)

        return mail_channel._chatbot_post_message(self.chatbot_script_id, plaintext2html(self.message))

    def _process_step_forward_operator(self, mail_channel):
        """ Special type of step that will add a human operator to the conversation when reached,
        which stops the script and allow the visitor to discuss with a real person.

        In case we don't find any operator (e.g: no-one is available) we don't post any messages.
        The script will continue normally, which allows to add extra steps when it's the case
        (e.g: ask for the visitor's email and create a lead). """

        human_operator = False
        posted_message = False

        if mail_channel.livechat_channel_id:
            human_operator = mail_channel.livechat_channel_id._get_random_operator()

        # handle edge case where we found yourself as available operator -> don't do anything
        # it will act as if no-one is available (which is fine)
        if human_operator and human_operator != self.env.user:
            mail_channel.sudo().add_members(
                human_operator.partner_id.ids,
                open_chat_window=True,
                post_joined_message=False)

            # rename the channel to include the operator's name
            mail_channel.sudo().name = ' '.join([
                self.env.user.display_name if not self.env.user._is_public() else mail_channel.anonymous_name,
                human_operator.livechat_username if human_operator.livechat_username else human_operator.name
            ])

            if self.message:
                # first post the message of the step (if we have one)
                posted_message = mail_channel._chatbot_post_message(self.chatbot_script_id, plaintext2html(self.message))

            # then post a small custom 'Operator has joined' notification
            mail_channel._chatbot_post_message(
                self.chatbot_script_id,
                Markup('<div class="o_mail_notification">%s</div>') % _('%s has joined', human_operator.partner_id.name))

            mail_channel._broadcast(human_operator.partner_id.ids)
            mail_channel.channel_pin(pinned=True)

        return posted_message

    # --------------------------
    # Tooling / Misc
    # --------------------------

    def _format_for_frontend(self):
        """ Small utility method that formats the step into a dict usable by the frontend code. """
        self.ensure_one()

        return {
            'chatbot_script_step_id': self.id,
            'chatbot_step_answers': [{
                'id': answer.id,
                'label': answer.name,
                'redirect_link': answer.redirect_link,
            } for answer in self.answer_ids],
            'chatbot_step_message': plaintext2html(self.message) if not is_html_empty(self.message) else False,
            'chatbot_step_is_last': self._is_last_step(),
            'chatbot_step_type': self.step_type
        }

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
    kpi_livechat_response = fields.Boolean('Time to answer (sec)')
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
            response_time = self.env['im_livechat.report.operator'].sudo()._read_group([
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
    name = fields.Char('Channel Name', required=True)
    button_text = fields.Char('Text of the Button', default=_default_button_text,
        help="Default text displayed on the Livechat Support Button")
    default_message = fields.Char('Welcome Message', default=_default_default_message,
        help="This is an automated 'welcome' message that your visitor will see when they initiate a new conversation.")
    input_placeholder = fields.Char('Chat Input Placeholder', help='Text that prompts the user to initiate the chat.')
    header_background_color = fields.Char(default="#875A7B", help="Default background color of the channel header once open")
    title_color = fields.Char(default="#FFFFFF", help="Default title color of the channel once open")
    button_background_color = fields.Char(default="#875A7B", help="Default background color of the Livechat button")
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
    chatbot_script_count = fields.Integer(string='Number of Chatbot', compute='_compute_chatbot_script_count')
    rule_ids = fields.One2many('im_livechat.channel.rule', 'channel_id', 'Rules')

    def _are_you_inside(self):
        for channel in self:
            channel.are_you_inside = bool(self.env.uid in [u.id for u in channel.user_ids])

    @api.depends('rule_ids.chatbot_script_id')
    def _compute_chatbot_script_count(self):
        data = self.env['im_livechat.channel.rule'].read_group(
            [('channel_id', 'in', self.ids)], ['chatbot_script_id:count_distinct'], ['channel_id'])
        mapped_data = {rule['channel_id'][0]: rule['chatbot_script_id'] for rule in data}
        for channel in self:
            channel.chatbot_script_count = mapped_data.get(channel.id, 0)

    def _compute_script_external(self):
        values = {
            "dbname": self._cr.dbname,
        }
        for record in self:
            values["channel_id"] = record.id
            values["url"] = record.get_base_url()
            record.script_external = self.env['ir.qweb']._render('im_livechat.external_loader', values) if record.id else False

    def _compute_web_page_link(self):
        for record in self:
            record.web_page = "%s/im_livechat/support/%i" % (record.get_base_url(), record.id) if record.id else False

    @api.depends('channel_ids')
    def _compute_nbr_channel(self):
        data = self.env['mail.channel']._read_group([
            ('livechat_channel_id', 'in', self.ids)
        ], ['__count'], ['livechat_channel_id'], lazy=False)
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

    def action_view_chatbot_scripts(self):
        action = self.env['ir.actions.act_window']._for_xml_id('im_livechat.chatbot_script_action')
        chatbot_script_ids = self.env['im_livechat.channel.rule'].search(
            [('channel_id', 'in', self.ids)]).mapped('chatbot_script_id')
        if len(chatbot_script_ids) == 1:
            action['res_id'] = chatbot_script_ids.id
            action['view_mode'] = 'form'
            action['views'] = [(False, 'form')]
        else:
            action['domain'] = [('id', 'in', chatbot_script_ids.ids)]
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

    def _get_livechat_mail_channel_vals(self, anonymous_name, operator=None, chatbot_script=None, user_id=None, country_id=None):
        # partner to add to the mail.channel
        operator_partner_id = operator.partner_id.id if operator else chatbot_script.operator_partner_id.id
        members_to_add = [Command.create({'partner_id': operator_partner_id, 'is_pinned': False})]
        visitor_user = False
        if user_id:
            visitor_user = self.env['res.users'].browse(user_id)
            if visitor_user and visitor_user.active and operator and visitor_user != operator:  # valid session user (not public)
                members_to_add.append(Command.create({'partner_id': visitor_user.partner_id.id}))

        if chatbot_script:
            name = chatbot_script.title
        else:
            name = ' '.join([
                visitor_user.display_name if visitor_user else anonymous_name,
                operator.livechat_username if operator.livechat_username else operator.name
            ])

        return {
            'channel_member_ids': members_to_add,
            'livechat_active': True,
            'livechat_operator_id': operator_partner_id,
            'livechat_channel_id': self.id,
            'chatbot_current_step_id': chatbot_script._get_welcome_steps()[-1].id if chatbot_script else False,
            'anonymous_name': False if user_id else anonymous_name,
            'country_id': country_id,
            'channel_type': 'livechat',
            'name': name,
        }

    def _open_livechat_mail_channel(self, anonymous_name, previous_operator_id=None, chatbot_script=None, user_id=None, country_id=None, persisted=True):
        """ Return a livechat session. If the session is persisted, creates a mail.channel record with a connected operator or with Odoobot as
            an operator if a chatbot has been configured, or return false otherwise
            :param anonymous_name : the name of the anonymous person of the session
            :param previous_operator_id : partner_id.id of the previous operator that this visitor had in the past
            :param chatbot_script : chatbot script if there is one configured
            :param user_id : the id of the logged in visitor, if any
            :param country_code : the country of the anonymous person of the session
            :param persisted: whether or not the session should be persisted
            :type anonymous_name : str
            :return : channel header
            :rtype : dict

            If this visitor already had an operator within the last 7 days (information stored with the 'im_livechat_previous_operator_pid' cookie),
            the system will first try to assign that operator if he's available (to improve user experience).
        """
        self.ensure_one()
        user_operator = False
        if chatbot_script:
            if chatbot_script.id not in self.env['im_livechat.channel.rule'].search(
                    [('channel_id', 'in', self.ids)]).mapped('chatbot_script_id').ids:
                return False
        elif previous_operator_id:
            available_users = self._get_available_users()
            # previous_operator_id is the partner_id of the previous operator, need to convert to user
            if previous_operator_id in available_users.mapped('partner_id').ids:
                user_operator = next(available_user for available_user in available_users if available_user.partner_id.id == previous_operator_id)
        if not user_operator and not chatbot_script:
            user_operator = self._get_random_operator()
        if not user_operator and not chatbot_script:
            # no one available
            return False
        mail_channel_vals = self._get_livechat_mail_channel_vals(anonymous_name, user_operator, chatbot_script, user_id=user_id, country_id=country_id)
        if persisted:
            # create the session, and add the link with the given channel
            mail_channel = self.env["mail.channel"].with_context(mail_create_nosubscribe=False).sudo().create(mail_channel_vals)
            if user_operator:
                mail_channel._broadcast([user_operator.partner_id.id])
            return mail_channel.sudo().channel_info()[0]
        else:
            operator_partner_id = user_operator.partner_id if user_operator else chatbot_script.operator_partner_id
            display_name = operator_partner_id.user_livechat_username or operator_partner_id.display_name
            return {
                'name': mail_channel_vals['name'],
                'chatbot_current_step_id': mail_channel_vals['chatbot_current_step_id'],
                'state': 'open',
                'operator_pid': (operator_partner_id.id, display_name.replace(',', '')),
                'chatbot_script_id': chatbot_script.id if chatbot_script else None
            }

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
        info['available'] = self.chatbot_script_count or len(self._get_available_users()) > 0
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
    action = fields.Selection([
        ('display_button', 'Show'),
        ('display_button_and_text', 'Show with notification'),
        ('auto_popup', 'Open automatically'),
        ('hide_button', 'Hide')], string='Live Chat Button', required=True, default='display_button',
        help="* 'Show' displays the chat button on the pages.\n"\
             "* 'Show with notification' is 'Show' in addition to a floating text just next to the button.\n"\
             "* 'Open automatically' displays the button and automatically opens the conversation pane.\n"\
             "* 'Hide' hides the chat button on the pages.\n")
    auto_popup_timer = fields.Integer('Open automatically timer', default=0,
        help="Delay (in seconds) to automatically open the conversation window. Note: the selected action must be 'Open automatically' otherwise this parameter will not be taken into account.")
    chatbot_script_id = fields.Many2one('chatbot.script', string='Chatbot')
    chatbot_only_if_no_operator = fields.Boolean(
        string='Enabled only if no operator', help='Enable the bot only if there is no operator available')
    channel_id = fields.Many2one('im_livechat.channel', 'Channel',
        help="The channel of the rule")
    country_ids = fields.Many2many('res.country', 'im_livechat_channel_country_rel', 'channel_id', 'country_id', 'Country',
        help="The rule will only be applied for these countries. Example: if you select 'Belgium' and 'United States' and that you set the action to 'Hide', the chat button will be hidden on the specified URL from the visitors located in these 2 countries. This feature requires GeoIP installed on your server.")
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
from odoo.tools import email_normalize, html_escape, html2plaintext, plaintext2html

from markupsafe import Markup


class MailChannel(models.Model):
    """ Chat Session
        Reprensenting a conversation between users.
        It extends the base method for anonymous usage.
    """

    _name = 'mail.channel'
    _inherit = ['mail.channel', 'rating.mixin']

    anonymous_name = fields.Char('Anonymous Name')
    channel_type = fields.Selection(selection_add=[('livechat', 'Livechat Conversation')], ondelete={'livechat': 'cascade'})
    livechat_active = fields.Boolean('Is livechat ongoing?', help='Livechat session is active until visitor leaves the conversation.')
    livechat_channel_id = fields.Many2one('im_livechat.channel', 'Channel', index='btree_not_null')
    livechat_operator_id = fields.Many2one('res.partner', string='Operator')
    chatbot_current_step_id = fields.Many2one('chatbot.script.step', string='Chatbot Current Step')
    chatbot_message_ids = fields.One2many('chatbot.message', 'mail_channel_id', string='Chatbot Messages')
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
            # add uuid to allow anonymous to listen
            if channel.channel_type == 'livechat':
                notifications.append([channel.uuid, 'mail.channel/new_message', notifications[0][2]])
        if not message.author_id:
            unpinned_members = self.channel_member_ids.filtered(lambda member: not member.is_pinned)
            if unpinned_members:
                unpinned_members.write({'is_pinned': True})
                notifications = self._channel_channel_notifications(unpinned_members.partner_id.ids) + notifications
        return notifications

    def channel_info(self):
        """ Extends the channel header by adding the livechat operator and the 'anonymous' profile
            :rtype : list(dict)
        """
        channel_infos = super().channel_info()
        channel_infos_dict = dict((c['id'], c) for c in channel_infos)
        for channel in self:
            channel_infos_dict[channel.id]['channel']['anonymous_name'] = channel.anonymous_name
            channel_infos_dict[channel.id]['channel']['anonymous_country'] = {
                'code': channel.country_id.code,
                'id': channel.country_id.id,
                'name': channel.country_id.name,
            } if channel.country_id else [('clear',)]
            if channel.livechat_operator_id:
                display_name = channel.livechat_operator_id.user_livechat_username or channel.livechat_operator_id.display_name
                channel_infos_dict[channel.id]['operator_pid'] = (channel.livechat_operator_id.id, display_name.replace(',', ''))
        return list(channel_infos_dict.values())

    @api.autovacuum
    def _gc_empty_livechat_sessions(self):
        hours = 1  # never remove empty session created within the last hour
        self.env.cr.execute("""
            SELECT id as id
            FROM mail_channel C
            WHERE NOT EXISTS (
                SELECT 1
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
        return super()._message_update_content_after_hook(message=message)

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
        mail_body = self.env['ir.qweb']._render('im_livechat.livechat_email_template', render_context, minimal_qcontext=True)
        mail_body = self.env['mail.render.mixin']._replace_local_links(mail_body)
        mail = self.env['mail.mail'].sudo().create({
            'subject': _('Conversation with %s', self.livechat_operator_id.user_livechat_username or self.livechat_operator_id.name),
            'email_from': company.catchall_formatted or company.email_formatted,
            'author_id': self.env.user.partner_id.id,
            'email_to': email,
            'body_html': mail_body,
        })
        mail.send()

    def _get_channel_history(self):
        """
        Converting message body back to plaintext for correct data formatting in HTML field.
        """
        return Markup('').join(
            Markup('%s: %s<br/>') % (message.author_id.name or self.anonymous_name, html2plaintext(message.body))
            for message in self.message_ids.sorted('id')
        )

    # =======================
    # Chatbot
    # =======================

    def _chatbot_find_customer_values_in_messages(self, step_type_to_field):
        """
        Look for user's input in the channel's messages based on a dictionary
        mapping the step_type to the field name of the model it will be used on.

        :param dict step_type_to_field: a dict of step types to customer fields
            to fill, like : {'question_email': 'email_from', 'question_phone': 'mobile'}
        """
        values = {}
        filtered_message_ids = self.chatbot_message_ids.filtered(
            lambda m: m.script_step_id.step_type in step_type_to_field.keys()
        )
        for message_id in filtered_message_ids:
            field_name = step_type_to_field[message_id.script_step_id.step_type]
            if not values.get(field_name):
                values[field_name] = html2plaintext(message_id.user_raw_answer or '')

        return values

    def _chatbot_post_message(self, chatbot_script, body):
        """ Small helper to post a message as the chatbot operator

        :param record chatbot_script
        :param string body: message HTML body """

        return self.with_context(mail_create_nosubscribe=True).message_post(
            author_id=chatbot_script.operator_partner_id.id,
            body=body,
            message_type='comment',
            subtype_xmlid='mail.mt_comment',
        )

    def _chatbot_validate_email(self, email_address, chatbot_script):
        email_address = html2plaintext(email_address)
        email_normalized = email_normalize(email_address)

        posted_message = False
        error_message = False
        if not email_normalized:
            error_message = _(
                "'%(input_email)s' does not look like a valid email. Can you please try again?",
                input_email=email_address
            )
            posted_message = self._chatbot_post_message(chatbot_script, plaintext2html(error_message))

        return {
            'success': bool(email_normalized),
            'posted_message': posted_message,
            'error_message': error_message,
        }

    def _message_post_after_hook(self, message, msg_vals):
        """
        This method is called just before _notify_thread() method which is calling the _message_format()
        method. We need a 'chatbot.message' record before it happens to correctly display the message.
        It's created only if the mail channel is linked to a chatbot step.
        """
        if self.chatbot_current_step_id:
            self.env['chatbot.message'].sudo().create({
                'mail_message_id': message.id,
                'mail_channel_id': self.id,
                'script_step_id': self.chatbot_current_step_id.id,
            })
        return super(MailChannel, self)._message_post_after_hook(message, msg_vals)

    def _chatbot_restart(self, chatbot_script):
        self.write({
            'chatbot_current_step_id': False
        })

        self.chatbot_message_ids.unlink()

        return self._chatbot_post_message(
            chatbot_script,
            '<div class="o_mail_notification">%s</div>' % _('Restarting conversation...'))

```

## File: models\mail_channel_member.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, timedelta

from odoo import api, models


class ChannelMember(models.Model):
    _inherit = 'mail.channel.member'

    @api.autovacuum
    def _gc_unpin_livechat_sessions(self):
        """ Unpin read livechat sessions with no activity for at least one day to
            clean the operator's interface """
        members = self.env['mail.channel.member'].search([
            ('is_pinned', '=', True),
            ('last_seen_dt', '<=', datetime.now() - timedelta(days=1)),
            ('channel_id.channel_type', '=', 'livechat'),
        ])
        sessions_to_be_unpinned = members.filtered(lambda m: m.message_unread_counter == 0)
        sessions_to_be_unpinned.write({'is_pinned': False})
        self.env['bus.bus']._sendmany([(member.partner_id, 'mail.channel/unpin', {'id': member.channel_id.id}) for member in sessions_to_be_unpinned])

    def _get_partner_data(self, fields=None):
        if self.channel_id.channel_type == 'livechat':
            data = {
                'active': self.partner_id.active,
                'id': self.partner_id.id,
                'is_public': self.partner_id.is_public,
            }
            if self.partner_id.user_livechat_username:
                data['user_livechat_username'] = self.partner_id.user_livechat_username
            else:
                data['name'] = self.partner_id.name
            if not self.partner_id.is_public:
                data['country'] = {
                    'code': self.partner_id.country_id.code,
                    'id': self.partner_id.country_id.id,
                    'name': self.partner_id.country_id.name,
                } if self.partner_id.country_id else [('clear',)]
            return data
        return super()._get_partner_data(fields=fields)

```

## File: models\mail_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MailMessage(models.Model):
    _inherit = 'mail.message'

    def _message_format(self, fnames, format_reply=True, legacy=False):
        """Override to remove email_from and to return the livechat username if applicable.
        A third param is added to the author_id tuple in this case to be able to differentiate it
        from the normal name in client code.

        In addition, if we are currently running a chatbot.script, we include the information about
        the chatbot.message related to this mail.message.
        This allows the frontend display to include the additional features
        (e.g: Show additional buttons with the available answers for this step). """

        vals_list = super()._message_format(fnames=fnames, format_reply=format_reply, legacy=legacy)
        for vals in vals_list:
            message_sudo = self.browse(vals['id']).sudo().with_prefetch(self.ids)
            mail_channel = self.env['mail.channel'].browse(message_sudo.res_id) if message_sudo.model == 'mail.channel' else self.env['mail.channel']
            if mail_channel.channel_type == 'livechat':
                if message_sudo.author_id:
                    vals.pop('email_from')
                if message_sudo.author_id.user_livechat_username:
                    vals['author'] = {
                        'id': message_sudo.author_id.id,
                        'user_livechat_username': message_sudo.author_id.user_livechat_username,
                    }
                # sudo: chatbot.script.step - members of a channel can access the current chatbot step
                if mail_channel.chatbot_current_step_id \
                        and message_sudo.author_id == mail_channel.chatbot_current_step_id.sudo().chatbot_script_id.operator_partner_id:
                    chatbot_message_id = self.env['chatbot.message'].sudo().search([
                        ('mail_message_id', '=', message_sudo.id)], limit=1)
                    if chatbot_message_id.script_step_id:
                        vals['chatbot_script_step_id'] = chatbot_message_id.script_step_id.id
                        if chatbot_message_id.script_step_id.step_type == 'question_selection':
                            vals['chatbot_step_answers'] = [{
                                'id': answer.id,
                                'label': answer.name,
                                'redirect_link': answer.redirect_link,
                            } for answer in chatbot_message_id.script_step_id.answer_ids]
                    if chatbot_message_id.user_script_answer_id:
                        vals['chatbot_selected_answer_id'] = chatbot_message_id.user_script_answer_id.id
        return vals_list

```

## File: models\rating_rating.py

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
            ('channel_member_ids', 'in', self.env['mail.channel.member'].sudo()._search([
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

from . import chatbot_message
from . import chatbot_script
from . import chatbot_script_answer
from . import chatbot_script_step
from . import res_users
from . import res_partner
from . import im_livechat_channel
from . import mail_channel
from . import mail_channel_member
from . import mail_message
from . import res_users_settings
from . import rating_rating
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
    start_date = fields.Datetime('Start Date of session', readonly=True)
    start_hour = fields.Char('Start Hour of session', readonly=True)
    day_number = fields.Char('Day Number', readonly=True, help="1 is Monday, 7 is Sunday")
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
                    <filter name="last_24h" string="Last 24h" domain="[('start_date','&gt;', (context_today() - datetime.timedelta(days=1)).strftime('%Y-%m-%d') )]"/>
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
            <field name="help">Livechat Support Channel Statistics allows you to easily check and analyse your company livechat session performance. Extract information about the missed sessions, the audience, the duration of a session, etc.</field>
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
    nbr_channel = fields.Integer('# of Sessions', readonly=True, group_operator="sum")
    channel_id = fields.Many2one('mail.channel', 'Conversation', readonly=True)
    start_date = fields.Datetime('Start Date of session', readonly=True)
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
                    <filter name="last_24h" string="Last 24h" domain="[('start_date','&gt;', (context_today() - datetime.timedelta(days=1)).strftime('%Y-%m-%d') )]"/>
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
            <field name="help">Livechat Support Channel Statistics allows you to easily check and analyse your company livechat session performance. Extract information about the missed sessions, the audience, the duration of a session, etc.</field>
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
access_chatbot_script,chatbot.script,model_chatbot_script,,1,0,0,0
access_chatbot_script_user,chatbot.script.user,model_chatbot_script,im_livechat_group_user,1,1,1,1
access_chatbot_script_step,chatbot.script.step,model_chatbot_script_step,,0,0,0,0
access_chatbot_script_step_user,chatbot.script.step.user,model_chatbot_script_step,im_livechat_group_user,1,1,1,1
access_chatbot_script_answer,chatbot.script.answer,model_chatbot_script_answer,,0,0,0,0
access_chatbot_script_answer_user,chatbot.script.answer.user,model_chatbot_script_answer,im_livechat_group_user,1,1,1,1
access_chatbot_message_all,chatbot.message,model_chatbot_message,,0,0,0,0
access_chatbot_message_user,chatbot.script.user,model_chatbot_message,im_livechat_group_user,1,1,1,1

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
            <t t-if="!composerView.composer.activeThread or !composerView.composer.activeThread.channel or composerView.composer.activeThread.channel.channel_type !== 'livechat'">$0</t>
        </xpath>
    </t>

</templates>

```

## File: static\src\components\discuss_sidebar\discuss_sidebar.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.DiscussSidebar" t-inherit-mode="extension">
        <xpath expr="//*[@name='beforeCategoryChat']" position="before">
            <t t-set="categoryLivechat" t-value="discussView.discuss.categoryLivechat"/>
            <t t-if="categoryLivechat and categoryLivechat.categoryItems.length">
                <DiscussSidebarCategory
                    className="'o_DiscussSidebar_category o_DiscussSidebar_categoryLivechat'"
                    record="categoryLivechat"
                />
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\components\thread_icon\thread_icon.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.ThreadIcon" t-inherit-mode="extension">
        <xpath expr="//*[@name='root']" position="inside">
            <t t-elif="thread.channel and thread.channel.channel_type === 'livechat'">
                <t t-if="thread.orderedOtherTypingMembers.length > 0">
                    <ThreadTypingIcon
                        className="'o_ThreadIcon_typing'"
                        animation="'pulse'"
                        title="thread.typingStatusText"
                    />
                </t>
                <t t-else="">
                    <div class="fa fa-fw fa-comments" title="Livechat"/>
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
        if (this.threadNeedactionPreviewView.thread.channel && this.threadNeedactionPreviewView.thread.channel.channel_type === 'livechat') {
            return '/mail/static/src/img/smiley/avatar.jpg';
        }
        return this._super(...args);
    }

});

```

## File: static\src\js\ajax_external.js

```javascript
/** @odoo-module **/

import { assets } from "@web/core/assets";

/**
  * This file should be used in the context of an external widget loading (e.g: live chat in a non-Odoo website)
  * It overrides the 'loadJS' method that is supposed to load additional scripts, based on a relative URL (e.g: '/web/webclient/locale/en_US')
  * As we're not in an Odoo website context, the calls will not work, and we avoid a 404 request.
  */
assets.loadJS = function (url) {
    console.log('Tried to load the following script on an external website: ' + url);
};

```

## File: static\src\js\im_livechat_chatbot_script_answers_m2m.js

```javascript
/** @odoo-module **/


import { registry } from "@web/core/registry";
import { Many2ManyTagsField } from "@web/views/fields/many2many_tags/many2many_tags_field";

const fieldRegistry = registry.category("fields");

export class ChatbotScriptTriggeringAnswersMany2Many extends Many2ManyTagsField {
    /**
     * Force the chatbot script ID we are currently editing into the context.
     * This allows to filter triggering question answers on steps of this script.
     */
    setup() {
        super.setup();

        if (this.props.record.model.root.data.id) {
            this.env.services.user.updateContext({
                force_domain_chatbot_script_id: this.props.record.model.root.data.id
            });
        }
    }
};

fieldRegistry.add("chatbot_triggering_answers_widget", ChatbotScriptTriggeringAnswersMany2Many);

```

## File: static\src\js\im_livechat_chatbot_steps_one2many.js

```javascript
/** @odoo-module */

import { ListRenderer } from "@web/views/list/list_renderer";
import { registry } from "@web/core/registry";
import { patch } from '@web/core/utils/patch';
import { useX2ManyCrud, useOpenX2ManyRecord, X2ManyFieldDialog } from "@web/views/fields/relational_utils";
import { X2ManyField } from "@web/views/fields/x2many/x2many_field";

const fieldRegistry = registry.category("fields");

patch(X2ManyFieldDialog.prototype, 'chatbot_script_step_sequence', {
    /**
     * Dirty patching of the 'X2ManyFieldDialog'.
     * It is done to force the "save and new" to close the dialog first, and then click again on
     * the "Add a line" link.
     * 
     * This is the only way (or at least the least complicated) to correctly compute the sequence
     * field, which is crucial when creating chatbot.steps, as they depend on each other.
     * 
     */
    async save({ saveAndNew }) {
        if (this.record.resModel !== 'chatbot.script.step') {
            return this._super(...arguments);
        }

        if (await this.record.checkValidity()) {
            this.record = (await this.props.save(this.record, { saveAndNew })) || this.record;
        } else {
            return false;
        }

        this.props.close();

        if (saveAndNew) {
            document.querySelector('.o_field_x2many_list_row_add a').click();
        }

        return true;
    }
});

export class ChatbotStepsOne2manyRenderer extends ListRenderer {
    /**
     * Small override to force column entries to be non-sortable.
     * Indeed, we want to force it being sorted on "sequence" at all times.
     */
    setup() {
        super.setup();

        for (const [, properties] of Object.entries(this.fields)) {
            properties.sortable = false;
        }
    }
}

export class ChatbotStepsOne2many extends X2ManyField {
    /**
     * Overrides the "openRecord" method to overload the save.
     *
     * Every time we save a sub-chatbot.script.step, we want to save the whole chatbot.script record
     * and form view.
     *
     * This allows the end-user to easily chain steps, otherwise he would have to save the
     * enclosing form view in-between each step addition.
     */
    setup() {
        super.setup();

        const { saveRecord, updateRecord } = useX2ManyCrud(
            () => this.list,
            this.isMany2Many
        );

        const openRecord = useOpenX2ManyRecord({
            resModel: this.list.resModel,
            activeField: this.activeField,
            activeActions: this.activeActions,
            getList: () => this.list,
            saveRecord: async (record) => {
                await saveRecord(record);
                await this.props.record.save({stayInEdition: true});
            },
            updateRecord: updateRecord,
        });

        this._openRecord = (params) => {
            const activeElement = document.activeElement;
            openRecord({
                ...params,
                onClose: () => {
                    if (activeElement) {
                        activeElement.focus();
                    }
                },
            });
        };
    }
};

fieldRegistry.add("chatbot_steps_one2many", ChatbotStepsOne2many);

ChatbotStepsOne2many.components = {
    ...X2ManyField.components,
    ListRenderer: ChatbotStepsOne2manyRenderer
};

```

## File: static\src\js\colors_reset_button\colors_reset_button.js

```javascript
/** @odoo-module **/

import { registry } from '@web/core/registry';
import { standardWidgetProps } from "@web/views/widgets/standard_widget_props";

const { Component } = owl;

export class ColorsResetButton extends Component {
    onColorsResetButtonClick() {
        this.props.record.update(this.props.default_colors);
    }
}
ColorsResetButton.template = `im_livechat.ColorsResetButton`;
ColorsResetButton.props = {
    ...standardWidgetProps,
    default_colors: { type: Object },
};
ColorsResetButton.extractProps = ({ attrs }) => {
    // Note: `options` should have `default_colors`. It's specified when using the widget.
    return attrs.options;
};

registry.category('view_widgets').add('colors_reset_button', ColorsResetButton);

```

## File: static\src\js\colors_reset_button\colors_reset_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="im_livechat.ColorsResetButton" owl="1">
        <button class="btn btn-link oe_edit_only" t-on-click="onColorsResetButtonClick" aria-label="Reset to default colors" title="Reset to default colors">
            <span class="fa fa-refresh mb-4"/>
        </button>
    </t>

</templates>

```

## File: static\src\legacy\public_livechat_chatbot.js

```javascript
/** @odoo-module **/

import session from 'web.session';
import time from 'web.time';
import utils from 'web.utils';

import LivechatButton from '@im_livechat/legacy/widgets/livechat_button';

/**
 * Override of the LivechatButton to include chatbot capabilities.
 * Main changes / hooking points are:
 * - Show a custom welcome message that is in fact the first message of the chatbot script
 * - When messages are rendered, add click handles to chatbot options
 * - When the user picks an option or answers to the chatbot, display a "chatbot is typing..."
 *   message for a couple seconds and then trigger the next step of the script
 */
 LivechatButton.include({

     //--------------------------------------------------------------------------
     // Private - LiveChat Overrides
     //--------------------------------------------------------------------------

    /**
     * @private
     * @override
     */
    _prepareGetSessionParameters() {
        const parameters = this._super(...arguments);

        const { publicLivechat } = this.messaging.publicLivechatGlobal;
        if (publicLivechat && publicLivechat.isTemporary && !publicLivechat.data.chatbot_script_id) {
            return parameters;
        } else if (publicLivechat && publicLivechat.data.chatbot_script_id) {
            parameters.chatbot_script_id = publicLivechat.data.chatbot_script_id;
        } else if (this.messaging.publicLivechatGlobal.chatbot.isActive) {
            parameters.chatbot_script_id = this.messaging.publicLivechatGlobal.chatbot.scriptId;
        }

        return parameters;
    },
    /**
     * Small override to handle chatbot welcome message(s).
     * @private
     */
    _sendWelcomeMessage() {
        if (this.messaging.publicLivechatGlobal.chatbot.isActive) {
            this._sendWelcomeChatbotMessage(
                0,
                this.messaging.publicLivechatGlobal.chatbot.state === 'welcome' ? 0 : this.messaging.publicLivechatGlobal.chatbot.messageDelay,
            );
        } else {
            this._super(...arguments);
        }
    },
    /**
     * The bot can say multiple messages in quick succession as "welcome messages".
     * (See chatbot.script#_get_welcome_steps() for more details).
     *
     * It is important that those messages are sent as "welcome messages", meaning manually added
     * within the template, without creating actual mail.messages in the mail.channel.
     *
     * Indeed, if the end-user never interacts with the bot, those empty mail.channels are deleted
     * by a garbage collector mechanism.
     *
     * About "welcomeMessageDelay":
     *
     * The first time we open the chat, we want to bot to slowly input those messages in one at a
     * time, with pauses during which the end-user sees ("The bot is typing...").
     *
     * However, if the user navigates within the website (meaning he has an opened mail.channel),
     * then we input all the welcome messages at once without pauses, to avoid having that annoying
     * slow effect on every page / refresh.
     *
     * @private
     */
    _sendWelcomeChatbotMessage(stepIndex, welcomeMessageDelay) {
        const chatbotStep = this.messaging.publicLivechatGlobal.chatbot.welcomeSteps[stepIndex];
        this.messaging.publicLivechatGlobal.chatbot.update({ currentStep: { data: chatbotStep } });

        if (chatbotStep.chatbot_step_message) {
            this.messaging.publicLivechatGlobal.livechatButtonView.addMessage({
                id: '_welcome_' + stepIndex,
                is_discussion: true, // important for css style -> we only want white background for chatbot
                author: (
                    this.messaging.publicLivechatGlobal.publicLivechat.operator
                    ? {
                        id: this.messaging.publicLivechatGlobal.publicLivechat.operator.id,
                        name: this.messaging.publicLivechatGlobal.publicLivechat.operator.name,
                    }
                    : [['clear']]
                ),
                body: utils.Markup(chatbotStep.chatbot_step_message),
                chatbot_script_step_id: chatbotStep.chatbot_script_step_id,
                chatbot_step_answers: chatbotStep.chatbot_step_answers,
                date: time.datetime_to_str(new Date()),
                model: "mail.channel",
                message_type: "comment",
                res_id: this.messaging.publicLivechatGlobal.publicLivechat.id,
            });
        }

        if (stepIndex + 1 < this.messaging.publicLivechatGlobal.chatbot.welcomeSteps.length) {
            if (welcomeMessageDelay !== 0) {
                this.messaging.publicLivechatGlobal.chatbot.setIsTyping(true);
            }

            this.messaging.publicLivechatGlobal.chatbot.update({
                welcomeMessageTimeout: setTimeout(() => {
                    if (!this.messaging.publicLivechatGlobal.chatWindow || !this.messaging.publicLivechatGlobal.chatWindow.exists()) {
                        return;
                    }
                    this._sendWelcomeChatbotMessage(stepIndex + 1, welcomeMessageDelay);
                    this.messaging.publicLivechatGlobal.chatWindow.renderMessages();
                }, welcomeMessageDelay),
            });
        } else {
            if (this.messaging.publicLivechatGlobal.chatbot.currentStep.data.chatbot_step_type === 'forward_operator') {
                // special case when the last welcome message is a forward to an operator
                // we need to save the welcome messages before continuing the script
                // indeed, if there are no operator available, the script will continue
                // with steps that are NOT included in the welcome messages
                // (hence why we need to have those welcome messages posted BEFORE that)
                this.messaging.publicLivechatGlobal.chatbot.postWelcomeMessages();
            }

            // we are done posting welcome messages, let's start the actual script
            this.messaging.publicLivechatGlobal.chatbot.processStep();
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Saves the selected chatbot.script.answer onto our chatbot.message.
     * Will update the state of the related message (in this.messaging.publicLivechatGlobal.messages) to set the selected option
     * as well which will in turn adapt the display to not show options anymore.
     *
     * This method also handles an optional redirection link placed on the chatbot.script.answer and
     * will make sure to properly save the selected choice before redirecting.
     *
     * @param {MouseEvent} ev
     * @private
     */
    async _onChatbotOptionClicked(ev) {
        ev.stopPropagation();

        const $target = $(ev.currentTarget);
        const stepId = $target.closest('ul').data('chatbotStepId');
        const selectedAnswer = $target.data('chatbotStepAnswerId');

        const redirectLink = $target.data('chatbotStepRedirectLink');
        let isRedirecting = false;
        if (redirectLink && URL.canParse(redirectLink, window.location.href)) {
            const url = new URL(window.location.href);
            const nextURL = new URL(redirectLink, window.location.href);
            isRedirecting = url.pathname !== nextURL.pathname || url.origin !== nextURL.origin;
        }
        this.messaging.publicLivechatGlobal.chatbot.update({ isRedirecting });

        await this.messaging.publicLivechatGlobal.livechatButtonView.sendMessage({
            content: $target.text().trim(),
        });

        let stepMessage = null;
        for (const message of this.messaging.publicLivechatGlobal.messages) {
            // we do NOT want to use a 'find' here because we want the LAST message that respects
            // this condition.
            // indeed, if you restart the script, you can have multiple messages with the same step id,
            // but here we only care about the very last one (the current step of the script)
            // reversing the this.messages variable seems like a bad idea because it could have
            // bad implications for other flows (as the reverse is in-place, not in a copy)
            if (message.widget.getChatbotStepId() === stepId) {
                stepMessage = message;
            }
        }
        const messageId = stepMessage.id;
        stepMessage.widget.setChatbotStepAnswerId(selectedAnswer);
        this.messaging.publicLivechatGlobal.chatbot.currentStep.data.chatbot_selected_answer_id = selectedAnswer;
        this.messaging.publicLivechatGlobal.chatWindow.renderMessages();
        this.messaging.publicLivechatGlobal.chatbot.saveSession();

        const saveAnswerPromise = session.rpc('/chatbot/answer/save', {
            channel_uuid: this.messaging.publicLivechatGlobal.publicLivechat.uuid,
            message_id: messageId,
            selected_answer_id: selectedAnswer,
        });

        if (redirectLink) {
            await saveAnswerPromise;  // ensure answer is saved before redirecting
            window.location = redirectLink;
        }
    },
});

export default LivechatButton;

```

## File: static\src\legacy\public_livechat_chatbot.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates xml:space="preserve">
    <!--
    =============================================
    Chatbot overrides to base livechat templates
    =============================================
     -->

    <!-- Extend the base livechat window to include a div allowing to restart the script at the end -->
    <t t-extend="im_livechat.legacy.PublicLivechatWindow">
        <t t-jquery='div.o_chat_mini_composer' t-operation="after">
            <div class="o_livechat_chatbot_end bg-200 fst-italic text-center border" style="display: none;">
                <span>Conversation ended...</span>
                <a href="#" class="o_livechat_chatbot_restart">Restart</a>
            </div>
        </t>
    </t>

    <!-- Extend the base livechat window header to include a button allowing to restart the script -->
    <t t-extend="im_livechat.legacy.PublicLivechatWindow.HeaderContent">
        <t t-jquery="span.o_thread_window_title" t-operation="after">
            <a t-if="widget.isMobile() &amp;&amp; widget.messaging.publicLivechatGlobal.chatbot.data &amp;&amp; widget.messaging.publicLivechatGlobal.chatbot.hasRestartButton"
                href="#" class="o_livechat_chatbot_main_restart fa fa-1x fa-refresh"
                title="Restart Conversation"/>
        </t>
        <t t-jquery="span.o_thread_window_buttons a.o_thread_window_close" t-operation="before">
            <a t-if="widget.messaging.publicLivechatGlobal.chatbot.data &amp;&amp; widget.messaging.publicLivechatGlobal.chatbot.hasRestartButton"
                href="#" class="o_livechat_chatbot_main_restart fa fa-refresh"
                title="Restart Conversation"/>
        </t>
    </t>

    <!-- Extend the base livechat message to allow displaying options for the user to select -->
    <t t-extend="im_livechat.legacy.mail.widget.Thread.Message">
        <t t-jquery='div.o_thread_message_content' t-operation="append">
            <ul t-if="message.getChatbotStepAnswers() &amp;&amp; message.getChatbotStepAnswers().length !== 0 &amp;&amp; !message.getChatbotStepAnswerId()"
                class="o_livechat_chatbot_options"
                t-att-data-message-id="message.getID()"
                t-att-data-chatbot-step-id="message.getChatbotStepId()">
                <t t-foreach="message.getChatbotStepAnswers()" t-as="stepAnswer">
                    <li t-att-data-chatbot-step-answer-id="stepAnswer.id"
                        t-att-data-chatbot-step-redirect-link="stepAnswer.redirect_link"
                        class="o_livechat_chatbot_stepAnswer d-inline-block border border-primary rounded p-2 me-3 mb-1 fw-bold">
                        <t t-out="stepAnswer.label"/>
                    </li>
                    <br/>
                </t>
            </ul>
        </t>
    </t>

    <!--
    =============================================
    Tooling / Utils
    =============================================
     -->

    <!--
        This small template simulates a fake message from the bot.
        The goal is to have something like:
        [image] The bot
                . . .

        With a small animation on the dots to make them bounce.
        This fake message is then removed when the chat window is refreshed with the real message.
    -->
    <t t-name="im_livechat.legacy.chatbot.is_typing_message">
        <div class="o_thread_message">
            <div class="o_thread_message_sidebar">
                <img alt="chatbot_image" t-att-src="chatbotImageSrc"
                    class="o_thread_message_avatar rounded-circle"/>
            </div>
            <div class="o_thread_message_core">
                <p class="o_mail_info text-muted">
                    <strong class="o_thread_author" t-out="chatbotName"/>
                </p>
                <div class="o_thread_message_content o_PublicLivechatMessage_content">
                    <img class="o_livechat_chatbot_typing"
                        t-att-src="chatbotIsTypingImageSrc"
                        width="30" alt="is typing"/>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\legacy\models\cc_throttle_function.js

```javascript
/** @odoo-module **/

import CCThrottleFunctionObject from '@im_livechat/legacy/models/cc_throttle_function_object';

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
const CCThrottleFunction = function (params) {
    const duration = params.duration;
    const func = params.func;

    const throttleFunctionObject = new CCThrottleFunctionObject({
        duration,
        func,
    });

    const callable = function () {
        return throttleFunctionObject.do(...arguments);
    };
    callable.cancel = function () {
        throttleFunctionObject.cancel();
    };
    callable.clear = function () {
        throttleFunctionObject.clear();
    };

    return callable;
};

export default CCThrottleFunction;

```

## File: static\src\legacy\models\cc_throttle_function_object.js

```javascript
/** @odoo-module **/

import Class from 'web.Class';

/**
 * This object models the behaviour of the clearable and cancellable (CC)
 * throttle version of a provided function.
 */
const CCThrottleFunctionObject = Class.extend({

    /**
     * @param {Object} params
     * @param {integer} params.duration duration of the 'cooldown' phase, i.e.
     *   the minimum duration between the most recent function call that has
     *   been made and the following function call.
     * @param {function} params.func provided function for making the CC
     *   throttled version.
     */
    init(params) {
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
    cancel() {
        this._arguments = undefined;
        this._shouldCallFunctionAfterCD = false;
    },
    /**
     * Clear the internal throttle timer, so that the following function call
     * is immediate. For instance, if there is a cooldown stage, it is aborted.
     */
    clear() {
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
    do() {
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
    _callFunction() {
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
    _cooldown() {
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
    _onCooldownTimeout() {
        if (this._shouldCallFunctionAfterCD) {
            this._callFunction();
        } else {
            this._cooldownTimeout = undefined;
        }
    },
});

export default CCThrottleFunctionObject;

```

## File: static\src\legacy\models\public_livechat.js

```javascript
/** @odoo-module **/

import CCThrottleFunction from '@im_livechat/legacy/models/cc_throttle_function';
import Timer from '@im_livechat/legacy/models/timer';
import Timers from '@im_livechat/legacy/models/timers';

import Class from 'web.Class';
import { _t } from 'web.core';
import session from 'web.session';
import Mixins from 'web.mixins';
import { sprintf } from 'web.utils';

/**
 * Thread model that represents a livechat on the website-side. This livechat
 * is not linked to the mail service.
 */
const PublicLivechat = Class.extend(Mixins.EventDispatcherMixin, {
    /**
     * @private
     * @param {Messaging} messaging
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
     * @param {string} [params.data.status=''] the status of this thread
     * @param {string} params.data.uuid the UUID of this livechat.
     * @param {@im_livechat/legacy/widgets/livechat_button} params.parent
     */
    init(messaging, params) {
        Mixins.EventDispatcherMixin.init.call(this, arguments);
        this.setParent(params.parent);
        this.messaging = messaging;

        /**
         * Initialize the internal data for typing feature on threads.
         */

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

        if (params.data.message_unread_counter !== undefined) {
            this.messaging.publicLivechatGlobal.publicLivechat.update({
                unreadCounter: params.data.message_unread_counter
            });
        }

        if (_.isBoolean(params.data.folded)) {
            this.messaging.publicLivechatGlobal.publicLivechat.update({ isFolded: params.data.folded });
        } else {
            this.messaging.publicLivechatGlobal.publicLivechat.update({ isFolded: params.data.state === 'folded' });
        }
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Called when a new message is added to the thread
     * On receiving a message from a typing partner, unregister this partner
     * from typing partners (otherwise, it will still display it until timeout).
     *
     * Note that it only unregister typing operators.
     *
     * Note that in the frontend, there is no way to identify a message that is
     * from the current user, because there is no partner ID in the session and
     * a message with an author sets the partner ID of the author.
     *
     * @param {@im_livechat/legacy/models/public_livechat_message} message
     */
    addMessage(message) {
        const operatorID = this.messaging.publicLivechatGlobal.publicLivechat.operator.id;
        if (message.hasAuthor() && message.getAuthorID() === operatorID) {
            this.unregisterTyping({ partnerID: operatorID });
        }
    },
    /**
     * @override
     * @returns {@im_livechat/legacy/models/public_livechat_message[]}
     */
     getMessages() {
        // ignore removed messages
        return this.messaging.publicLivechatGlobal.messages.filter(message => !message.widget.isEmpty()).map(message => message.widget);
    },
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
    getTypingMembersToText() {
        const typingPartnerIDs = this._typingPartnerIDs;
        const typingMembers = (
            this.messaging.publicLivechatGlobal.publicLivechat.operator && this._typingPartnerIDs.includes(this.messaging.publicLivechatGlobal.publicLivechat.operator.id)
            ? [this.messaging.publicLivechatGlobal.publicLivechat.operator]
            : []
        );
        const sortedTypingMembers = _.sortBy(typingMembers, function (member) {
            return _.indexOf(typingPartnerIDs, member.id);
        });
        const displayableTypingMembers = sortedTypingMembers.slice(0, 3);

        if (displayableTypingMembers.length === 0) {
            return '';
        } else if (displayableTypingMembers.length === 1) {
            return sprintf(_t("%s is typing..."), displayableTypingMembers[0].name);
        } else if (displayableTypingMembers.length === 2) {
            return sprintf(_t("%s and %s are typing..."),
                                    displayableTypingMembers[0].name,
                                    displayableTypingMembers[1].name);
        } else {
            return sprintf(_t("%s, %s and more are typing..."),
                                    displayableTypingMembers[0].name,
                                    displayableTypingMembers[1].name);
        }
    },
    /**
     * @returns {boolean}
     */
    hasMessages() {
        return !_.isEmpty(this.getMessages());
    },
    /**
     * Tells if someone other than current user is typing something on this
     * thread.
     *
     * @returns {boolean}
     */
    isSomeoneTyping() {
        return !(_.isEmpty(this._typingPartnerIDs));
    },
    /**
     * Mark the thread as read, which resets the unread counter to 0. This is
     * only performed if the unread counter is not 0.
     *
     * @returns {Promise}
     */
    markAsRead() {
        if (this.messaging.publicLivechatGlobal.publicLivechat.unreadCounter > 0) {
            this.messaging.publicLivechatGlobal.publicLivechat.update({ unreadCounter: 0 });
            this.messaging.publicLivechatGlobal.chatWindow.widget.renderHeader();
            return Promise.resolve();
        }
        return Promise.resolve();
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
     */
    async postMessage() {
        this._lastNotifiedMyselfTyping = false;
        this._throttleNotifyMyselfTyping.clear();
        this._myselfLongTypingTimer.clear();
        this._myselfTypingInactivityTimer.clear();
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
    registerTyping(params) {
        if (params.isWebsiteUser) {
            return;
        }
        const partnerID = params.partnerID;
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
    setMyselfTyping(params) {
        const typing = params.typing;
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
     * @returns {Object}
     */
    toData() {
        return {
            visitor_uid: this.messaging.publicLivechatGlobal.getVisitorUserId(),
            chatbot_script_id: this.messaging.publicLivechatGlobal.publicLivechat.data.chatbot_script_id,
            folded: this.messaging.publicLivechatGlobal.publicLivechat.isFolded,
            id: this.messaging.publicLivechatGlobal.publicLivechat.id,
            message_unread_counter: this.messaging.publicLivechatGlobal.publicLivechat.unreadCounter,
            operator_pid: (
                this.messaging.publicLivechatGlobal.publicLivechat.operator
                ? [
                    this.messaging.publicLivechatGlobal.publicLivechat.operator.id,
                    this.messaging.publicLivechatGlobal.publicLivechat.operator.name,
                ]
                : []
            ),
            name: this.messaging.publicLivechatGlobal.publicLivechat.name,
            uuid: this.messaging.publicLivechatGlobal.publicLivechat.uuid,
        };
    },
    /**
     * Unregister someone from currently typing something in this thread.
     *
     * @param {Object} params
     * @param {integer} params.partnerID ID of the partner related to the user
     *   that is currently typing something
     */
    unregisterTyping(params) {
        const partnerID = params.partnerID;
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
     * Notify to the server that the current user either starts or stops typing
     * something.
     *
     * @private
     * @param {Object} params
     * @param {boolean} params.typing whether we are typing something or not
     * @returns {Promise} resolved if the server is notified, rejected
     *   otherwise
     */
    _notifyMyselfTyping(params) {
        if (this.messaging.publicLivechatGlobal.publicLivechat.isTemporary) {
            // channel is not created yet, it will be when first message is
            // sent. Until then, do not notify visitor is typing.
            return;
        }
        return session.rpc('/im_livechat/notify_typing', {
            uuid: this.messaging.publicLivechatGlobal.publicLivechat.uuid,
            is_typing: params.typing,
        }, { shadow: true });
    },
    /**
     * Warn views that the list of users that are currently typing on this
     * livechat has been updated.
     *
     * @private
     */
    _warnUpdatedTypingPartners() {
        this.messaging.publicLivechatGlobal.chatWindow.widget.renderHeader();
    },

    //--------------------------------------------------------------------------
    // Handler
    //--------------------------------------------------------------------------

    /**
     * Called when current user is typing something for a long time. In order
     * to not let other users assume that we are no longer typing something, we
     * must notify again that we are typing something.
     *
     * @private
     */
    _onMyselfLongTypingTimeout() {
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
    _onMyselfTypingInactivityTimeout() {
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
    _onNotifyMyselfTyping(params) {
        const typing = params.typing;
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
    _onOthersTypingTimeout(partnerID) {
        this.unregisterTyping({ partnerID });
    },
});

export default PublicLivechat;

```

## File: static\src\legacy\models\public_livechat_message.js

```javascript
/** @odoo-module **/

import * as mailUtils from '@mail/js/utils';

import Class from 'web.Class';
import { _t } from 'web.core';
import session from 'web.session';
import time from 'web.time';

/**
 * This is a message that is handled by im_livechat, without making use of the
 * mail.Manager. The purpose of this is to make im_livechat compatible with
 * mail.widget.Thread.
 *
 * @see @im_livechat/legacy/models/public_livechat_message for more information.
 */
const PublicLivechatMessage = Class.extend({

    /**
     * @param {@im_livechat/legacy/widgets/livechat_button} parent
     * @param {Messaging} messaging
     * @param {Object} data
     * @param {Object|Array} [data.author]
     * @param {string} [data.body = ""]
     * @param {string} [data.date] the server-format date time of the message.
     *   If not provided, use current date time for this message.
     * @param {integer} data.id
     * @param {boolean} [data.is_discussion = false]
     * @param {boolean} [data.is_notification = false]
     * @param {string} [data.message_type = undefined]
     */
    init(parent, messaging, data) {
        this.messaging = messaging;
        this._body = data.body || "";
        // by default: current datetime
        this._date = data.date ? moment(time.str_to_datetime(data.date)) : moment();
        this._id = data.id;
        this._isDiscussion = data.is_discussion;
        this._isNotification = data.is_notification;
        this._serverAuthor = data.author;
        this._type = data.message_type || undefined;

        this._defaultUsername = this.messaging.publicLivechatGlobal.options.default_username;
        this._serverURL = this.messaging.publicLivechatGlobal.serverUrl;

        if (this.messaging.publicLivechatGlobal.chatbot.isActive) {
            this._chatbotStepId = data.chatbot_script_step_id;
            this._chatbotStepAnswers = data.chatbot_step_answers;
            this._chatbotStepAnswerId = data.chatbot_selected_answer_id;
        }
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Get the server ID (number) of the author of this message
     * If there are no author, return -1;
     *
     * @return {integer}
     */
    getAuthorID() {
        if (!this.hasAuthor()) {
            return -1;
        }
        return this._serverAuthor.id;
    },
    /**
     * Get the relative url of the avatar to display next to the message
     *
     * @return {string}
     */
    getAvatarSource() {
        let source = this._serverURL;
        if (this.isOperatorTheAuthor()) {
            source += `/im_livechat/operator/${this.getAuthorID()}/avatar`;
        } else if (this.hasAuthor() && session.user_id) {
            source += `/web/image/res.partner/${this.getAuthorID()}/avatar_128`;
        } else {
            source += '/mail/static/src/img/smiley/avatar.jpg';
        }
        return source;
    },
    /**
     * Get the body content of this message
     *
     * @return {string}
     */
    getBody() {
        return this._body;
    },
    /**
     * @return {string}
     */
    getChatbotStepId() {
        return this._chatbotStepId;
    },
    /**
     * @return {string}
     */
    getChatbotStepAnswers() {
        return this._chatbotStepAnswers;
    },
    /**
     * @return {string}
     */
    getChatbotStepAnswerId() {
        return this._chatbotStepAnswerId;
    },
    /**
     * @return {moment}
     */
    getDate() {
        return this._date;
    },
    /**
     * Get the date day of this message
     *
     * @return {string}
     */
    getDateDay() {
        const date = this.getDate().format('YYYY-MM-DD');
        if (date === moment().format('YYYY-MM-DD')) {
            return _t("Today");
        } else if (date === moment().subtract(1, 'days').format('YYYY-MM-DD')) {
            return _t("Yesterday");
        }
        return this.getDate().format('LL');
    },
    /**
     * Get the text to display for the author of the message
     *
     * Rule of precedence for the displayed author::
     *
     *      author name > default usernane
     *
     * @return {string}
     */
    getDisplayedAuthor() {
        return (this.hasAuthor() ? this._getAuthorName() : null) || this._defaultUsername;
    },
    /**
     * Get the server ID (number) of this message
     *
     * @return {integer}
     */
    getID() {
        return this._id;
    },
    /**
     * Get the time elapsed between sent message and now
     *
     * @return {string}
     */
    getTimeElapsed() {
        return mailUtils.timeFromNow(this.getDate());
    },
    /**
     * Get the type of message (e.g. 'comment', 'email', 'notification', ...)
     * By default, messages are of type 'undefined'
     *
     * @return {string|undefined}
     */
    getType() {
        return this._type;
    },
    /**
     * State whether this message has an author
     *
     * @return {boolean}
     */
    hasAuthor() {
        return Boolean(this._serverAuthor && this._serverAuthor.id);
    },
    /**
     * State whether this message is empty
     *
     * @return {boolean}
     */
    isEmpty() {
        return !this.getBody();
    },
    /**
     * State whether this message is a discussion
     *
     * @return {boolean}
     */
    isDiscussion() {
        return this._isDiscussion;
    },
    /**
     * State whether this message is a note (i.e. a message from "Log note")
     *
     * @return {boolean}
     */
    isNote() {
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
    isNotification() {
        return this._isNotification;
    },
    setChatbotStepAnswerId(chatbotStepAnswerId) {
        this._chatbotStepAnswerId = chatbotStepAnswerId;
    },
    /**
     * State whether this message should redirect to the author
     * when clicking on the author of this message.
     *
     * Do not redirect on author clicked of self-posted messages.
     *
     * @return {boolean}
     */
    shouldRedirectToAuthor() {
        return !this._isMyselfAuthor();
    },

    isVisitorTheAuthor() {
        return !this.hasAuthor() || this._isMyselfAuthor();
    },

    isOperatorTheAuthor() {
        return this.hasAuthor() && !this._isMyselfAuthor();
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
    _getAuthorName() {
        if (!this.hasAuthor()) {
            return "";
        }
        return this._serverAuthor.name || this._serverAuthor.user_livechat_username;
    },
    /**
     * State whether the current user is the author of this message
     *
     * @private
     * @return {boolean}
     */
    _isMyselfAuthor() {
        return this.hasAuthor() && (this.getAuthorID() === this.messaging.publicLivechatGlobal.options.current_partner_id);
    },

});

export default PublicLivechatMessage;

```

## File: static\src\legacy\models\timer.js

```javascript
/** @odoo-module **/

import Class from 'web.Class';

/**
 * This class creates a timer which, when times out, calls a function.
 */
const Timer = Class.extend({

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
    init(params) {
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
    clear() {
        clearTimeout(this._timeout);
    },
    /**
     * Resets the timer, i.e. resets its duration.
     */
    reset() {
        this.clear();
        this.start();
    },
    /**
     * Starts the timer, i.e. after a certain duration, it times out and calls
     * a function back.
     */
    start() {
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
    _onTimeout() {
        this._timeoutCallback();
    },

});

export default Timer;

```

## File: static\src\legacy\models\timers.js

```javascript
/** @odoo-module **/

import Timer from '@im_livechat/legacy/models/timer';

import Class from 'web.Class';

/**
 * This class lists several timers that use a same callback and duration.
 */
const Timers = Class.extend({

    /**
     * Instantiate a new list of timers
     *
     * @param {Object} params
     * @param {integer} params.duration duration of the underlying timers from
     *   start to timeout, in milli-seconds.
     * @param {function} params.onTimeout a function to call back for underlying
     *   timers on timeout.
     */
    init(params) {
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
     registerTimer(params) {
        const timerID = params.timerID;
        if (this._timers[timerID]) {
            this._timers[timerID].clear();
        }
        const timerParams = {
            duration: this._duration,
            onTimeout: this._timeoutCallback,
        };
        if ('timeoutCallbackArguments' in params) {
            timerParams.onTimeout = this._timeoutCallback.bind.apply(
                this._timeoutCallback,
                [null, ...params.timeoutCallbackArguments]
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
     unregisterTimer(params) {
        const timerID = params.timerID;
        if (this._timers[timerID]) {
            this._timers[timerID].clear();
            delete this._timers[timerID];
        }
    },

});

export default Timers;

```

## File: static\src\legacy\widgets\livechat_button.js

```javascript
/** @odoo-module **/

import time from 'web.time';
import {getCookie} from 'web.utils.cookies';
import Widget from 'web.Widget';

const LivechatButton = Widget.extend({
    className: 'openerp o_livechat_button d-print-none',
    events: {
        'click': '_onClick'
    },
    init(parent, messaging) {
        this._super(parent);
        this.messaging = messaging;
    },
    start() {
        this.messaging.publicLivechatGlobal.livechatButtonView.start();
        return this._super();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

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
     _get_previous_operator_id() {
        const cookie = getCookie('im_livechat_previous_operator_pid');
        if (cookie) {
            return cookie;
        }

        return null;
    },
    /**
     * @private
     */
    _prepareGetSessionParameters() {
        return {
            channel_id: this.messaging.publicLivechatGlobal.channelId,
            anonymous_name: this.messaging.publicLivechatGlobal.livechatButtonView.defaultUsername,
            previous_operator_id: this._get_previous_operator_id(),
        };
    },
    /**
     * @private
     */
    _sendWelcomeMessage() {
        if (this.messaging.publicLivechatGlobal.livechatButtonView.defaultMessage) {
            this.messaging.publicLivechatGlobal.livechatButtonView.addMessage({
                id: '_welcome',
                author: {
                    id: this.messaging.publicLivechatGlobal.publicLivechat.operator.id,
                    name: this.messaging.publicLivechatGlobal.publicLivechat.operator.name,
                },
                body: this.messaging.publicLivechatGlobal.livechatButtonView.defaultMessage,
                date: time.datetime_to_str(new Date()),
                model: "mail.channel",
                res_id: this.messaging.publicLivechatGlobal.publicLivechat.id,
            }, { prepend: true });
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onClick() {
        this.messaging.publicLivechatGlobal.livechatButtonView.openChat();
    },
});

export default LivechatButton;

```

## File: static\src\legacy\widgets\feedback\feedback.js

```javascript
/** @odoo-module **/

import concurrency from 'web.concurrency';
import core from 'web.core';
import session from 'web.session';
import utils from 'web.utils';
import Widget from 'web.Widget';

const _t = core._t;
/*
 * Rating for Livechat
 *
 * This widget displays the 3 rating smileys, and a textarea to add a reason
 * (only for red smiley), and sends the user feedback to the server.
 */
const Feedback = Widget.extend({
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
     * @param {Messaging} messaging
     * @param {@im_livechat/legacy/models/public_livechat} livechat
     */
    init(parent, messaging, livechat) {
        this._super(parent);
        this.messaging = messaging;
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
     _sendFeedback(reason) {
        const args = {
            uuid: this.messaging.publicLivechatGlobal.publicLivechat.uuid,
            rate: this.rating,
            reason,
        };
        this.dp.add(session.rpc('/im_livechat/feedback', args)).then((response) => {
            const emoji = this.messaging.publicLivechatGlobal.RATING_TO_EMOJI[this.rating] || "??";
            let content;
            if (!reason) {
                content = utils.sprintf(_t("Rating: %s"), emoji);
            }
            else {
                content = "Rating reason: \n" + reason;
            }
            this.trigger('send_message', { content, isFeedback: true });
        });
    },
    /**
    * @private
    */
    _showThanksMessage() {
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
    _onClickNoFeedback() {
        this.trigger('feedback_sent'); // will close the chat
    },
    /**
     * @private
     */
    _onClickSend() {
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
    _onClickSmiley(ev) {
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
    _onEmailChat() {
        const $email = this.$('#o_email');

        if (utils.is_email($email.val())) {
            $email.removeAttr('title').removeClass('is-invalid').prop('disabled', true);
            this.$('.o_email_chat_button').prop('disabled', true);
            this._rpc({
                route: '/im_livechat/email_livechat_transcript',
                params: {
                    uuid: this.messaging.publicLivechatGlobal.publicLivechat.uuid,
                    email: $email.val(),
                }
            }).then(() => {
                this.$('o_livechat_email_sentLabel').show();
                this.$('o_livechat_email_receiveCopyLabel').hide();
                this.$('o_livechat_email_receiveCopyForm').hide();
            }).guardedCatch(() => {
                this.$('.o_livechat_email').hide();
                this.$('.o_livechat_email_error').show();
            });
        } else {
            $email.addClass('is-invalid').prop('title', _t('Invalid email address'));
        }
    },
    /**
    * @private
    */
    _onTryAgain() {
        this.$('#o_email').prop('disabled', false);
        this.$('.o_email_chat_button').prop('disabled', false);
        this.$('.o_livechat_email_error').hide();
        this.$('.o_livechat_email').show();
    },
});

export default Feedback;

```

## File: static\src\legacy\widgets\feedback\feedback.xml

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
            <div class="o_livechat_email text-start">
                <strong class="o_livechat_email_sentLabel" style="display: none;">Conversation Sent</strong>
                <span class="o_livechat_email_receiveCopyLabel text-muted">Receive a copy of this conversation</span>
                <div class="o_livechat_email_receiveCopyForm input-group">
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
</templates>

```

## File: static\src\legacy\widgets\public_livechat_view\public_livechat_view.js

```javascript
/** @odoo-module **/

import * as mailUtils from '@mail/js/utils';

import core from 'web.core';
import time from 'web.time';
import Widget from 'web.Widget';

const QWeb = core.qweb;
const _lt = core._lt;

const ORDER = {
    ASC: 1, // visually, ascending order of message IDs (from top to bottom)
    DESC: -1, // visually, descending order of message IDs (from top to bottom)
};

const READ_MORE = _lt("read more");
const READ_LESS = _lt("read less");

/**
 * This is a generic widget to render a thread.
 * Any thread that extends mail.model.AbstractThread can be used with this
 * widget.
 */
const PublicLivechatView = Widget.extend({
    className: 'o_mail_thread',

    events: {
        'click a': '_onClickRedirect',
        'click img': '_onClickRedirect',
        'click strong': '_onClickRedirect',
        'click .o_thread_show_more': '_onClickShowMore',
        'click .o_thread_message_needaction': '_onClickMessageNeedaction',
        'click .o_thread_message_star': '_onClickMessageStar',
        'click .o_thread_message_reply': '_onClickMessageReply',
        'click .oe_mail_expand': '_onClickMailExpand',
        'click .o_thread_message': '_onClickMessage',
        'click': '_onClick',
        'click .o_thread_message_notification_error': '_onClickMessageNotificationError',
    },

    /**
     * @override
     * @param {widget} parent
     * @param {Messaging} messaging
     * @param {Object} options
     */
    init(parent, messaging, options) {
        this._super(...arguments);
        this.messaging = messaging;
        // options when the thread is enabled (e.g. can send message,
        // interact on messages, etc.)
        this._enabledOptions = _.defaults(options || {}, {
            displayOrder: ORDER.ASC,
            displayMarkAsRead: true,
            displayDocumentLinks: true,
            displayAvatars: true,
            squashCloseMessages: true,
            loadMoreOnScroll: false,
        });
        this._selectedMessageID = null;
        this._currentThreadID = null;
    },
    /**
     * The message mail popover may still be shown at this moment. If we do not
     * remove it, it stays visible on the page until a page reload.
     *
     * @override
     */
    destroy() {
        clearInterval(this._updateTimestampsInterval);
        this._super();
    },
    /**
     * @param {Object} [options]
     * @param {integer} [options.displayOrder=ORDER.ASC] order of displaying
     *    messages in the thread:
     *      - ORDER.ASC: last message is at the bottom of the thread
     *      - ORDER.DESC: last message is at the top of the thread
     * @param {boolean} [options.displayLoadMore]
     * @param {Array} [options.domain=[]] the domain for the messages in the
     *    thread.
     * @param {boolean} [options.scrollToBottom=false]
     * @param {boolean} [options.squashCloseMessages]
     */
    render(options) {
        let shouldScrollToBottomAfterRendering = false;
        if (this._currentThreadID === this.messaging.publicLivechatGlobal.publicLivechat.id && this.isAtBottom()) {
            shouldScrollToBottomAfterRendering = true;
        }
        this._currentThreadID = this.messaging.publicLivechatGlobal.publicLivechat.id;

        // copy so that reverse do not alter order in the thread object
        const messages = _.clone(this.messaging.publicLivechatGlobal.publicLivechat.widget.getMessages());

        const modeOptions = this._enabledOptions;

        options = Object.assign({}, modeOptions, options, {
            selectedMessageID: this._selectedMessageID,
        });

        // dict where key is message ID, and value is whether it should display
        // the author of message or not visually
        const displayAuthorMessages = {};

        // Hide avatar and info of a message if that message and the previous
        // one are both comments wrote by the same author at the same minute
        // and in the same document (users can now post message in documents
        // directly from a channel that follows it)
        let prevMessage;
        for (let message of messages) {
            if (
                // is first message of thread
                !prevMessage ||
                // more than 1 min. elasped
                (Math.abs(message.getDate().diff(prevMessage.getDate())) > 60000) ||
                prevMessage.getType() !== 'comment' ||
                message.getType() !== 'comment' ||
                // from a different author
                prevMessage.getAuthorID() !== message.getAuthorID()
            ) {
                displayAuthorMessages[message.getID()] = true;
            } else {
                displayAuthorMessages[message.getID()] = !options.squashCloseMessages;
            }
            prevMessage = message;
        }

        if (modeOptions.displayOrder === ORDER.DESC) {
            messages.reverse();
        }

        this.$el.html(QWeb.render('im_livechat.legacy.mail.widget.Thread', {
            displayAuthorMessages,
            options,
            ORDER,
            dateFormat: time.getLangDatetimeFormat(),
            widget: this,
        }));

        for (let message of messages) {
            const $message = this.$('.o_thread_message[data-message-id="' + message.getID() + '"]');
            $message.find('.o_mail_timestamp').data('date', message.getDate());

            this._insertReadMore($message);
        }

        if (shouldScrollToBottomAfterRendering) {
            this.scrollToBottom();
        }

        if (!this._updateTimestampsInterval) {
            this.updateTimestampsInterval = setInterval(() => {
                this._updateTimestamps();
            }, 1000 * 60);
        }
    },

    /**
     * Render thread widget when loading, i.e. when messaging is not yet ready.
     * @see /mail/init_messaging
     */
    renderLoading() {
        this.$el.html(QWeb.render('im_livechat.legacy.mail.widget.ThreadLoading'));
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    getScrolltop() {
        return this.$el.scrollTop();
    },
    /**
     * State whether the bottom of the thread is visible or not,
     * with a tolerance of 5 pixels
     *
     * @return {boolean}
     */
     isAtBottom() {
        const fullHeight = this.el.scrollHeight;
        const topHiddenHeight = this.$el.scrollTop();
        const visibleHeight = this.$el.outerHeight();
        const bottomHiddenHeight = fullHeight - topHiddenHeight - visibleHeight;
        return bottomHiddenHeight < 5;
    },
    /**
     * Scroll to the bottom of the thread
     */
    scrollToBottom() {
        this.$el.scrollTop(this.el.scrollHeight);
    },
    /**
     * Scrolls the thread to a given message
     *
     * @param {integer} options.msgID the ID of the message to scroll to
     * @param {integer} [options.duration]
     * @param {boolean} [options.onlyIfNecessary]
     */
     scrollToMessage(options) {
        const $target = this.$('.o_thread_message[data-message-id="' + options.messageID + '"]');
        if (options.onlyIfNecessary) {
            const delta = $target.parent().height() - $target.height();
            let offset = delta < 0 ?
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
    scrollToPosition(position) {
        if (position) {
            this.$el.scrollTop(position);
        } else {
            this.scrollToBottom();
        }
    },
    /**
     * Unselect the selected message
     */
    unselectMessage() {
        this.$('.o_thread_message').removeClass('o_thread_selected_message');
        this._selectedMessageID = null;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

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
     _insertReadMore($element) {

        const groups = [];
        let readMoreNodes;

        // nodeType 1: element_node
        // nodeType 3: text_node
        const $children = $element.contents()
            .filter(function () {
                return this.nodeType === 1 ||
                        this.nodeType === 3 &&
                        this.nodeValue.trim();
            });

            for (let child of $children) {
            let $child = $(child);

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
                this._insertReadMore($child);
            }
        }

        for (let group of groups) {
            // Insert link just before the first node
            const $readMore = $('<a>', {
                class: 'o_mail_read_more',
                href: '#',
                text: READ_MORE,
            }).insertBefore(group[0]);

            // Toggle All next nodes
            let isReadMore = true;
            $readMore.click(function (e) {
                e.preventDefault();
                isReadMore = !isReadMore;
                for (let $child of group) {
                    $child.hide();
                    $child.toggle(!isReadMore);
                }
                $readMore.text(isReadMore ? READ_MORE : READ_LESS);
            });
        }
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
     * @private
     */
     _updateTimestamps() {
        const isAtBottom = this.isAtBottom();
        this.$('.o_mail_timestamp').each(function () {
            const date = $(this).data('date');
            $(this).text(mailUtils.timeFromNow(date));
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
     */
    _onClick() {
        if (this._selectedMessageID) {
            this.unselectMessage();
            this.trigger('unselect_message');
        }
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickMailExpand(ev) {
        ev.preventDefault();
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickMessage(ev) {
        $(ev.currentTarget).toggleClass('o_thread_selected_message');
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
     _onClickMessageNeedaction(ev) {
        const messageID = $(ev.currentTarget).data('message-id');
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
    _onClickMessageReply(ev) {
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
     _onClickMessageStar(ev) {
        const messageID = $(ev.currentTarget).data('message-id');
        this.trigger('toggle_star_status', messageID);
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickRedirect(ev) {
        // ignore inherited branding
        if ($(ev.target).data('oe-field') !== undefined) {
            return;
        }
        const id = $(ev.target).data('oe-id');
        if (id) {
            ev.preventDefault();
            const model = $(ev.target).data('oe-model');
            let options;
            if (model && (model !== 'mail.channel')) {
                options = {
                    model,
                    id
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
    _onClickShowMore() {
        this.trigger('load_more_messages');
    },
});

PublicLivechatView.ORDER = ORDER;

export default PublicLivechatView;

```

## File: static\src\legacy\widgets\public_livechat_view\public_livechat_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates xml:space="preserve">
    <!--
        @param {mail.model.AbstractThread} thread
        @param {Object} options
        @param {boolean} [options.displayEmptyThread]
        @param {boolean} [options.displayNoMatchFound]
        @param {Array} [options.domain=[]] the domain to restrict messages on the thread.
    -->
    <t t-name="im_livechat.legacy.mail.widget.Thread">
        <t t-if="widget.messaging.publicLivechatGlobal.publicLivechat.widget.hasMessages()">
            <t t-call="im_livechat.legacy.mail.widget.Thread.Content"/>
        </t>
    </t>

    <!-- Rendering of thread when messaging not yet ready -->
    <div t-name="im_livechat.legacy.mail.widget.ThreadLoading" class="o_mail_thread_loading">
        <i class="o_mail_thread_loading_icon fa fa-circle-o-notch fa-spin"/>
        <span>Please wait...</span>
    </div>
    
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
        <t t-set="messages" t-value="widget.messaging.publicLivechatGlobal.publicLivechat.widget.getMessages({ 'domain': options.domain || [] })"/>
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
        @param {boolean} [options.displayMarkAsRead]
        @param {boolean} [options.displaySubjectsOnMessages]
        @param {string|integer} [options.messagesSeparatorPosition] 'top' or
            message ID, the separator is placed just after this message.
        @param {integer} [options.selectedMessageID]
    -->
    <t t-name="im_livechat.legacy.mail.widget.Thread.Message">
        <div t-if="!message.isEmpty()" t-att-class="'o_thread_message o_PublicLivechatMessage ' + (message.getID() === options.selectedMessageID ? 'o_thread_selected_message ' : ' ') + (message.isDiscussion() or message.isNotification() ? ' o_mail_discussion ' : ' o_mail_not_discussion ') + (message.isVisitorTheAuthor() ? 'o-isVisitorTheAuthor' : '')"
            t-att-data-message-id="message.getID()">
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
            </div>
            <div class="o_thread_message_core">
                <p t-if="displayAuthorMessages[message.getID()]" t-attf-class="o_mail_info text-muted o_PublicLivechatMessage_header {{ message.isVisitorTheAuthor() ? 'o-isVisitorTheAuthor' : '' }}">
                    <t t-if="message.isNote()">
                        Note by
                    </t>
                    <t t-if="!message.isVisitorTheAuthor()">
                        <strong t-if="message.hasAuthor()"
                                data-oe-model="res.partner" t-att-data-oe-id="message.shouldRedirectToAuthor() ? message.getAuthorID() : ''"
                                t-attf-class="o_thread_author o_PublicLivechatMessage_headerAuthor #{message.shouldRedirectToAuthor() ? 'o_mail_redirect' : ''}">
                            <t t-esc="message.getDisplayedAuthor()"/>
                        </strong>
                        <strong t-else="" class="o_thread_author o_PublicLivechatMessage_headerAuthor">
                            <t t-esc="message.getDisplayedAuthor()"/>
                        </strong>
                    </t>

                    <span t-if="!message.isVisitorTheAuthor()" class="o_PublicLivechatMessage_headerDatePrefix">-</span> <small class="o_mail_timestamp o_PublicLivechatMessage_headerDate" t-att-title="message.getDate().format(dateFormat)"><t t-esc="message.getTimeElapsed()"/></small>
                    <span t-attf-class="o_thread_icons">
                    </span>
                </p>
                <div t-att-data-message-id="message.getID()" t-attf-class="o_PublicLivechatMessage_bubbleWrap {{ message.isVisitorTheAuthor() ? 'o-isVisitorTheAuthor' : '' }} {{ message.isOperatorTheAuthor() ? 'o-isOperatorTheAuthor' : '' }}">
                    <div t-attf-class="o_PublicLivechatMessage_bubble {{ message.isVisitorTheAuthor() ? 'o-isVisitorTheAuthor' : '' }} {{ message.isOperatorTheAuthor() ? 'o-isOperatorTheAuthor' : '' }} {{ !message.isEmpty() ? 'o-isContentNonEmpty' : '' }}">
                        <div t-attf-class="o_PublicLivechatMessage_background {{ message.isVisitorTheAuthor() ? 'o-isVisitorTheAuthor' : '' }} {{ message.isOperatorTheAuthor() ? 'o-isOperatorTheAuthor' : '' }}"/>
                        <div class="o_thread_message_content o_PublicLivechatMessage_content">
                            <t t-out="message.getBody()"/>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <t t-if="options.messagesSeparatorPosition == message.getID()">
            <t t-call="im_livechat.legacy.mail.MessagesSeparator"/>
        </t>
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
        @param {string} status
        @param {integer|undefined} [partnerID]
    -->
    <t t-name="im_livechat.legacy.mail.UserStatus">
        <span t-att-class="partnerID ? 'o_updatable_im_status' : ''" t-att-data-partner-id="partnerID"/>
    </t>
    
    <t t-name="im_livechat.legacy.mail.MessagesSeparator">
        <div class="o_thread_new_messages_separator">
            <span class="o_thread_separator_label">New messages</span>
        </div>
    </t>
</templates>
```

## File: static\src\legacy\widgets\public_livechat_window\public_livechat_window.js

```javascript
/** @odoo-module **/

import config from 'web.config';
import { _t, qweb } from 'web.core';
import Widget from 'web.Widget';

import {unaccent} from 'web.utils';
import {setCookie} from 'web.utils.cookies';

/**
 * This is the widget that represent windows of livechat in the frontend.
 *
 * @see @im_livechat/legacy/widgets/public_livechat_window/public_livechat_window for more information
 */
const PublicLivechatWindow = Widget.extend({
    FOLD_ANIMATION_DURATION: 200, // duration in ms for (un)fold transition
    HEIGHT_OPEN: '400px', // height in px of thread window when open
    HEIGHT_FOLDED: '34px', // height, in px, of thread window when folded
    template: 'im_livechat.legacy.PublicLivechatWindow',
    events: {
        'click .o_thread_window_close': '_onClickClose',
        'click .o_thread_window_header': '_onClickFold',
        'click .o_composer_text_field': '_onComposerClick',
        'click .o_mail_thread': '_onThreadWindowClicked',
        'keydown .o_composer_text_field': '_onKeydown',
        'keypress .o_composer_text_field': '_onKeypress',
        'input .o_composer_text_field': '_onInput',
    },
    /**
     * @param {Widget} parent
     * @param {Messaging} messaging
     * @param {@im_livechat/legacy/models/public_livechat} thread
     */
    init(parent, messaging, thread) {
        this._super(parent);
        this.messaging = messaging;

        this._debouncedOnScroll = _.debounce(this._onScroll.bind(this), 100);
    },
    /**
     * @override
     * @return {Promise}
     */
    async start() {
        this.$input = this.$('.o_composer_text_field');
        this.$header = this.$('.o_thread_window_header');

        // animate the (un)folding of thread windows
        this.$el.css({ transition: 'height ' + this.FOLD_ANIMATION_DURATION + 'ms linear' });
        if (this.messaging.publicLivechatGlobal.publicLivechat.isFolded) {
            this.$el.css('height', this.HEIGHT_FOLDED);
        } else {
            this._focusInput();
        }
        const def = this.messaging.publicLivechatGlobal.chatWindow.publicLivechatView.widget.replace(this.$('.o_thread_window_content')).then(() => {
            this.messaging.publicLivechatGlobal.chatWindow.publicLivechatView.widget.$el.on('scroll', this, this._debouncedOnScroll);
        });
        await Promise.all([this._super(), def]);
        if (this.messaging.publicLivechatGlobal.livechatButtonView.headerBackgroundColor) {
            this.$('.o_thread_window_header').css('background-color', this.messaging.publicLivechatGlobal.livechatButtonView.headerBackgroundColor);
        }
        if (this.messaging.publicLivechatGlobal.livechatButtonView.titleColor) {
            this.$('.o_thread_window_header').css('color', this.messaging.publicLivechatGlobal.livechatButtonView.titleColor);
        }
    },


    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    close() {
        const isComposerDisabled = this.messaging.publicLivechatGlobal.chatWindow.widget.$('.o_thread_composer input').prop('disabled');
        const shouldAskFeedback = !isComposerDisabled && this.messaging.publicLivechatGlobal.messages.find(function (message) {
            return message.id !== '_welcome';
        });
        if (shouldAskFeedback) {
            this.messaging.publicLivechatGlobal.chatWindow.widget.toggleFold(false);
            this.messaging.publicLivechatGlobal.livechatButtonView.askFeedback();
        } else {
            this.messaging.publicLivechatGlobal.livechatButtonView.closeChat();
        }
        this.messaging.publicLivechatGlobal.leaveSession();
    },
    /**
     * States whether the current environment is in mobile or not. This is
     * useful in order to customize the template rendering for mobile view.
     *
     * @returns {boolean}
     */
    isMobile() {
        return config.device.isMobile;
    },
    /**
     * Render the thread window
     */
    render() {
        this.renderHeader();
        this.messaging.publicLivechatGlobal.chatWindow.publicLivechatView.widget.render({ displayLoadMore: false });
    },
    /**
     * Render the header of this thread window.
     * This is useful when some information on the header have be updated such
     * as the status or the title of the thread that have changed.
     *
     * @private
     */
    renderHeader() {
        this.$header.html(qweb.render('im_livechat.legacy.PublicLivechatWindow.HeaderContent', { widget: this }));
    },

    /**
     * Render the chat window itself.
     */
    renderChatWindow() {
        this.renderElement();
        this.adjustPosition();
    },

    /**
     * Compute position of this chat window and apply corresponding styles to
     * the underlying widget.
     */
    adjustPosition() {
        const cssProps = { bottom: 0 };
        cssProps[this.messaging.locale.textDirection === 'rtl' ? 'left' : 'right'] = 0;
        if (!config.device.isMobile) {
            const margin_dir = _t.database.parameters.direction === "rtl" ? "margin-left" : "margin-right";
            cssProps[margin_dir] = $.position.scrollbarWidth();
        }
        this.$el.css(cssProps);
    },

    /**
     * Replace the thread content with provided new content
     *
     * @param {$.Element} $element
     */
    replaceContentWith($element) {
        $element.replace(this.messaging.publicLivechatGlobal.chatWindow.publicLivechatView.widget.$el);
    },
    /**
     * Toggle the fold state of this thread window. Also update the fold state
     * of the thread model. If the boolean parameter `folded` is provided, it
     * folds/unfolds the window when it is set/unset.
     *
     * Warn the parent widget (LivechatButton)
     *
     * @param {boolean} [folded] if not a boolean, toggle the fold state.
     *   Otherwise, fold/unfold the window if set/unset.
     */
    toggleFold(folded) {
        if (!_.isBoolean(folded)) {
            folded = !this.messaging.publicLivechatGlobal.publicLivechat.isFolded;
        }
        this.messaging.publicLivechatGlobal.publicLivechat.update({ isFolded: folded });
        if (this.messaging.publicLivechatGlobal.publicLivechat.operator) {
            setCookie('im_livechat_session', unaccent(JSON.stringify(this.messaging.publicLivechatGlobal.publicLivechat.widget.toData()), true), 60 * 60, 'required');
        }
        this.updateVisualFoldState();
    },
    /**
     * Update the visual state of the window so that it matched the internal
     * fold state. This is useful in case the related thread has its fold state
     * that has been changed.
     */
    updateVisualFoldState() {
        if (!this.messaging.publicLivechatGlobal.publicLivechat.isFolded) {
            this.messaging.publicLivechatGlobal.chatWindow.publicLivechatView.widget.scrollToBottom();
            this._focusInput();
        }
        const height = this.messaging.publicLivechatGlobal.publicLivechat.isFolded ? this.HEIGHT_FOLDED : this.HEIGHT_OPEN;
        this.$el.css({ height });
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
    _focusInput() {
        if (
            config.device.touch &&
            config.device.size_class <= config.device.SIZES.SM
        ) {
            return;
        }
        this.$input.focus();
    },
    /**
     * Tells whether there is focus on this thread. Note that a thread that has
     * the focus means the input has focus.
     *
     * @private
     * @returns {boolean}
     */
    _hasFocus() {
        return this.$input.is(':focus');
    },
    /**
     * Post a message on this thread window, and auto-scroll to the bottom of
     * the thread.
     *
     * @private
     * @param {Object} messageData
     */
    async _postMessage(messageData) {
        try {
            await this.messaging.publicLivechatGlobal.livechatButtonView.sendMessage(messageData);
        } catch (_err) {
            await this.messaging.publicLivechatGlobal.livechatButtonView.sendMessage(messageData); // try again just in case
        }
        if (!this.messaging.publicLivechatGlobal.publicLivechat.operator) {
            return;
        }
        this.messaging.publicLivechatGlobal.publicLivechat.widget.postMessage(messageData)
            .then(() => {
                this.messaging.publicLivechatGlobal.chatWindow.publicLivechatView.widget.scrollToBottom();
            });
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
    _onClickClose(ev) {
        ev.stopPropagation();
        ev.preventDefault();
        if (
            this.messaging.publicLivechatGlobal.publicLivechat.unreadCounter > 0 &&
            !this.messaging.publicLivechatGlobal.publicLivechat.isFolded
        ) {
            this.messaging.publicLivechatGlobal.publicLivechat.widget.markAsRead();
        }
        this.close();
    },
    /**
     * Fold/unfold the thread window.
     * Also mark the thread as read.
     *
     * @private
     */
    _onClickFold() {
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
    _onComposerClick(ev) {
        if ($(ev.target).closest('a, button').length) {
            return;
        }
        this._focusInput();
    },
    /**
     * Called when the input in the composer changes
     *
     * @private
     */
    _onInput() {
        const isTyping = this.$input.val().length > 0;
        this.messaging.publicLivechatGlobal.publicLivechat.widget.setMyselfTyping({ typing: isTyping });
    },
    /**
     * Called when typing something on the composer of this thread window.
     *
     * @private
     * @param {KeyboardEvent} ev
     */
    _onKeydown(ev) {
        ev.stopPropagation(); // to prevent jquery's blockUI to cancel event
        // ENTER key (avoid requiring jquery ui for external livechat)
        if (ev.which === 13) {
            const content = _.str.trim(this.$input.val());
            const messageData = {
                content,
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
    _onKeypress(ev) {
        ev.stopPropagation(); // to prevent jquery's blockUI to cancel event
    },
    /**
     * @private
     */
    _onScroll() {
        if (
            !this.messaging.exists() ||
            !this.messaging.publicLivechatGlobal ||
            !this.messaging.publicLivechatGlobal.chatWindow
        ) {
            return;
        }
        if (this.messaging.publicLivechatGlobal.chatWindow.publicLivechatView.widget.isAtBottom()) {
            this.messaging.publicLivechatGlobal.publicLivechat.widget.markAsRead();
        }
    },
    /**
     * When a thread window is clicked on, we want to give the focus to the main
     * input. An exception is made when the user is selecting something.
     *
     * @private
     */
    _onThreadWindowClicked() {
        const selectObj = window.getSelection();
        if (selectObj.anchorOffset === selectObj.focusOffset) {
            this.$input.focus();
        }
    },
});

export default PublicLivechatWindow;

```

## File: static\src\legacy\widgets\public_livechat_window\public_livechat_window.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <!--
        @param {im_livechat.legacy.PublicLivechatWindow} widget
    -->
    <t t-name="im_livechat.legacy.PublicLivechatWindow">
        <div class="o_thread_window o_in_home_menu"
            t-att-data-thread-id="widget.messaging.publicLivechatGlobal.publicLivechat.id"
            t-att-data-thread-model='widget.messaging.publicLivechatGlobal.publicLivechat.model'
        >
            <div class="o_thread_window_header">
                <t t-call="im_livechat.legacy.PublicLivechatWindow.HeaderContent">
                    <t t-set="status" t-value="widget.messaging.publicLivechatGlobal.publicLivechat.status"/>
                    <t t-set="title" t-value="widget.messaging.publicLivechatGlobal.publicLivechat.name"/>
                    <t t-set="unreadCounter" t-value="widget.messaging.publicLivechatGlobal.publicLivechat.unreadCounter"/>
                    <t t-set="thread" t-value="widget.messaging.publicLivechatGlobal.publicLivechat.widget"/>
                </t>
            </div>
            <t t-if="widget.messaging.publicLivechatGlobal.publicLivechat.operator">
                <div class="o_thread_window_content">
                </div>
                <div class="o_thread_composer o_chat_mini_composer">
                    <input class="o_composer_text_field o_PublicLivechatWindow_composer" t-att-placeholder="widget.messaging.publicLivechatGlobal.chatWindow.inputPlaceholder"/>
                </div>
            </t>
            <div t-else="" class="d-flex justify-content-center align-items-center flex-grow-1">
                <p class="text-500">No operator available</p>
            </div>
        </div>
    </t>

    <!--
        @param {im_livechat/legacy/widgets/public_livechat_window/public_livechat_window} widget
    -->
    <t t-name="im_livechat.legacy.PublicLivechatWindow.HeaderContent">
        <span class="o_thread_window_title">
            <t t-esc="widget.messaging.publicLivechatGlobal.publicLivechat.name"/>
            <span t-if="widget.messaging.publicLivechatGlobal.publicLivechat.unreadCounter"> (<t t-esc="widget.messaging.publicLivechatGlobal.publicLivechat.unreadCounter"/>)</span>
            <t t-if="widget.messaging.publicLivechatGlobal.publicLivechat.widget.isSomeoneTyping()" t-call="im_livechat.legacy.mail.ThreadTypingIcon"/>
        </span>
        <span class="o_thread_window_buttons">
            <a href="#" class="o_thread_window_close fa fa-close"/>
        </span>
    </t>

    <!--
        @param {mail.model.Thread} thread with typing feature
    -->
    <t t-name="im_livechat.legacy.mail.ThreadTypingIcon">
        <span class="o_mail_thread_typing_icon" t-att-title="widget.messaging.publicLivechatGlobal.publicLivechat.widget.getTypingMembersToText()">
            <span class="o_mail_thread_typing_icon_dot"/>
            <span class="o_mail_thread_typing_icon_dot"/>
            <span class="o_mail_thread_typing_icon_dot"/>
        </span>
    </t>

</templates>

```

## File: static\src\models\channel.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { attr, one } from '@mail/model/model_field';

registerPatch({
    name: 'Channel',
    fields: {
        anonymous_country: one('Country'),
        anonymous_name: attr(),
        discussSidebarCategory: {
            compute() {
                if (this.channel_type === 'livechat') {
                    return this.messaging.discuss.categoryLivechat;
                }
                return this._super();
            },
        },
        displayName: {
            compute() {
                if (!this.thread) {
                    return;
                }
                if (this.channel_type === 'livechat' && this.correspondent) {
                    if (!this.correspondent.is_public && this.correspondent.country) {
                        return `${this.thread.getMemberName(this.correspondent.persona)} (${this.correspondent.country.name})`;
                    }
                    if (this.anonymous_country) {
                        return `${this.thread.getMemberName(this.correspondent.persona)} (${this.anonymous_country.name})`;
                    }
                    return this.thread.getMemberName(this.correspondent.persona);
                }
                return this._super();
            },
        },
    },
});

```

## File: static\src\models\channel_preview_view.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';

registerPatch({
    name: 'ChannelPreviewView',
    fields: {
        imageUrl: {
            compute() {
                if (this.channel.channel_type === 'livechat') {
                    return '/mail/static/src/img/smiley/avatar.jpg';
                }
                return this._super();
            },
        },
    },
});

```

## File: static\src\models\chat_window.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';

registerPatch({
    name: 'ChatWindow',
    recordMethods: {
        /**
         * @override
         */
        close({ notifyServer } = {}) {
            if (
                this.thread &&
                this.thread.channel &&
                this.thread.channel.channel_type === 'livechat' &&
                this.thread.cache.isLoaded &&
                this.thread.messages.length === 0
            ) {
                notifyServer = true;
                this.thread.unpin();
            }
            this._super({ notifyServer });
        },
    },
});

```

## File: static\src\models\composer_view.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { clear } from '@mail/model/model_field_command';
import '@mail/models/composer_view';

registerPatch({
    name: 'ComposerView',
    fields: {
        dropZoneView: {
            compute() {
                if (this.composer.thread && this.composer.thread.channel && this.composer.thread.channel.channel_type === 'livechat') {
                    return clear();
                }
                return this._super();
            },
        },
    },
});

```

## File: static\src\models\discuss.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { one } from '@mail/model/model_field';

registerPatch({
    name: 'Discuss',
    recordMethods: {
        /**
         * @override
         */
        onInputQuickSearch(value) {
            if (!this.sidebarQuickSearchValue) {
                this.categoryLivechat.open();
            }
            return this._super(value);
        },
    },
    fields: {
        /**
         * Discuss sidebar category for `livechat` channel threads.
         */
        categoryLivechat: one('DiscussSidebarCategory', {
            default: {},
            inverse: 'discussAsLivechat',
        }),
    },
});

```

## File: static\src\models\discuss_sidebar_category.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { one } from '@mail/model/model_field';
import { clear } from '@mail/model/model_field_command';

registerPatch({
    name: 'DiscussSidebarCategory',
    fields: {
        categoryItemsOrderedByLastAction: {
            compute() {
                if (this.discussAsLivechat) {
                    return this.categoryItems;
                }
                return this._super();
            },
        },
        discussAsLivechat: one('Discuss', {
            identifying: true,
            inverse: 'categoryLivechat',
        }),
        isServerOpen: {
            compute() {
                // there is no server state for non-users (guests)
                if (!this.messaging.currentUser) {
                    return clear();
                }
                if (!this.messaging.currentUser.res_users_settings_id) {
                    return clear();
                }
                if (this.discussAsLivechat) {
                    return this.messaging.currentUser.res_users_settings_id.is_discuss_sidebar_category_livechat_open;
                }
                return this._super();
            },
        },
        name: {
            compute() {
                if (this.discussAsLivechat) {
                    return this.env._t("Livechat");
                }
                return this._super();
            },
        },
        orderedCategoryItems: {
            compute() {
                if (this.discussAsLivechat) {
                    return this.categoryItemsOrderedByLastAction;
                }
                return this._super();
            },
        },
        serverStateKey: {
            compute() {
                if (this.discussAsLivechat) {
                    return 'is_discuss_sidebar_category_livechat_open';
                }
                return this._super();
            },
        },
        supportedChannelTypes: {
            compute() {
                if (this.discussAsLivechat) {
                    return ['livechat'];
                }
                return this._super();
            },
        },
    },
});

```

## File: static\src\models\discuss_sidebar_category_item.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { clear } from '@mail/model/model_field_command';

registerPatch({
    name: 'DiscussSidebarCategoryItem',
    fields: {
        avatarUrl: {
            compute() {
                if (this.channel.channel_type === 'livechat') {
                    if (this.channel.correspondent && !this.channel.correspondent.is_public) {
                        return this.channel.correspondent.avatarUrl;
                    }
                }
                return this._super();
            },
        },
        categoryCounterContribution: {
            compute() {
                if (this.channel.channel_type === 'livechat') {
                    return this.channel.localMessageUnreadCounter > 0 ? 1 : 0;
                }
                return this._super();
            },
        },
        counter: {
            compute() {
                if (this.channel.channel_type === 'livechat') {
                    return this.channel.localMessageUnreadCounter;
                }
                return this._super();
            },
        },
        hasThreadIcon: {
            compute() {
                if (this.channel.channel_type === 'livechat') {
                    return clear();
                }
                return this._super();
            },
        },
        hasUnpinCommand: {
            compute() {
                if (this.channel.channel_type === 'livechat') {
                    return !this.channel.localMessageUnreadCounter;
                }
                return this._super();
            },
        },
    },
});

```

## File: static\src\models\message.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';

registerPatch({
    name: 'Message',
    fields: {
        hasReactionIcon: {
            compute() {
                if (this.originThread && this.originThread.channel && this.originThread.channel.channel_type === 'livechat') {
                    return false;
                }
                return this._super();
            },
        },
    },
});

```

## File: static\src\models\message_action_list.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { clear } from '@mail/model/model_field_command';

registerPatch({
    name: 'MessageActionList',
    fields: {
        actionReplyTo: {
            compute() {
                if (
                    this.message &&
                    this.message.originThread &&
                    this.message.originThread.channel &&
                    this.message.originThread.channel.channel_type === 'livechat'
                ) {
                    return clear();
                }
                return this._super();
            }
        },
    },
});

```

## File: static\src\models\message_view.js

```javascript
/** @odoo-module **/

import { registerPatch } from "@mail/model/model_core";

registerPatch({
    name: "MessageView",
    fields: {
        hasAuthorClickable: {
            compute() {
                if (
                    this.message &&
                    this.message.originThread &&
                    this.message.originThread.channel &&
                    this.message.originThread.channel.channel_type === "livechat"
                ) {
                    return this.message.author === this.message.originThread.channel.correspondent;
                }
                return this._super();
            },
        },
    },
});

```

## File: static\src\models\messaging.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { many } from '@mail/model/model_field';

registerPatch({
    name: 'Messaging',
    fields: {
        /**
         * All pinned livechats that are known.
         */
        pinnedLivechats: many('Thread', {
            inverse: 'messagingAsPinnedLivechat',
            readonly: true,
        }),
    },
});

```

## File: static\src\models\messaging_initializer.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { insert } from '@mail/model/model_field_command';

registerPatch({
    name: 'MessagingInitializer',
    recordMethods: {
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
    },
});

```

## File: static\src\models\mobile_messaging_navbar_view.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';

registerPatch({
    name: 'MobileMessagingNavbarView',
    fields: {
        tabs: {
            compute() {
                const res = this._super();
                if (this.messaging.pinnedLivechats.length > 0) {
                    return [...res, {
                        icon: 'fa fa-comments',
                        id: 'livechat',
                        label: this.env._t("Livechat"),
                    }];
                }
                return res;
            },
        },
    },
});

```

## File: static\src\models\notification_list_view.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';

registerPatch({
    name: 'NotificationListView',
    fields: {
        filteredChannels: {
            compute() {
                if (this.filter === 'livechat') {
                    return this.messaging.models['Channel'].all(channel =>
                        channel.channel_type === 'livechat' &&
                        channel.thread.isPinned
                    );
                }
                return this._super();
            },
        },
    },
});

```

## File: static\src\models\partner.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { attr } from '@mail/model/model_field';

registerPatch({
    name: 'Partner',
    fields: {
        /**
         * States the specific name of this partner in the context of livechat.
         * Either a string or undefined.
         */
        user_livechat_username: attr(),
    },
});

```

## File: static\src\models\res_users_settings.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { attr } from '@mail/model/model_field';

registerPatch({
    name: 'res.users.settings',
    fields: {
        is_discuss_sidebar_category_livechat_open: attr({
            default: true,
        }),
    },
});

```

## File: static\src\models\thread.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { one } from '@mail/model/model_field';
import { clear } from '@mail/model/model_field_command';

registerPatch({
    name: 'Thread',
    recordMethods: {
        /**
         * @override
         */
        getMemberName(persona) {
            if (this.channel && this.channel.channel_type === 'livechat' && persona.partner && persona.partner.user_livechat_username) {
                return persona.partner.user_livechat_username;
            }
            if (this.channel && this.channel.channel_type === 'livechat' && persona.partner && persona.partner.is_public && this.channel.anonymous_name) {
                return this.channel.anonymous_name;
            }
            return this._super(persona);
        },
    },
    fields: {
        hasInviteFeature: {
            compute() {
                if (this.channel && this.channel.channel_type === 'livechat') {
                    return true;
                }
                return this._super();
            },
        },
        hasMemberListFeature: {
            compute() {
                if (this.channel && this.channel.channel_type === 'livechat') {
                    return true;
                }
                return this._super();
            },
        },
        isChatChannel: {
            compute() {
                if (this.channel && this.channel.channel_type === 'livechat') {
                    return true;
                }
                return this._super();
            },
        },
        /**
         * If set, current thread is a livechat.
         */
        messagingAsPinnedLivechat: one('Messaging', {
            compute() {
                if (!this.messaging || !this.channel || this.channel.channel_type !== 'livechat' || !this.isPinned) {
                    return clear();
                }
                return this.messaging;
            },
            inverse: 'pinnedLivechats',
        }),
    },
});

```

## File: static\src\public\main.js

```javascript
/** @odoo-module **/

import { publicLivechatService } from '@im_livechat/services/public_livechat_service';
import { isAvailable, options, serverUrl } from 'im_livechat.loaderData';

import { messagingService } from '@mail/services/messaging_service';
import { makeMessagingToLegacyEnv } from '@mail/utils/make_messaging_to_legacy_env';

import { registry } from '@web/core/registry';

const messagingValuesService = {
    start() {
        return {
            publicLivechatGlobal: { isAvailable, options, serverUrl },
        };
    }
};

const serviceRegistry = registry.category('services');
serviceRegistry.add('messaging', messagingService);
serviceRegistry.add('messagingValues', messagingValuesService);
serviceRegistry.add('public_livechat_service', publicLivechatService);

registry.category('wowlToLegacyServiceMappers').add('make_messaging_to_legacy_env', makeMessagingToLegacyEnv);

```

## File: static\src\public\session.js

```javascript
odoo.define('web.session', function (require) {

    const Session = require('web.Session');
    const { serverUrl } = require('im_livechat.loaderData');

    return new Session(undefined, serverUrl, { use_cors: true });
});

```

## File: static\src\public_models\chatbot.js

```javascript
/** @odoo-module **/

import { registerModel } from '@mail/model/model_core';
import { attr, one } from '@mail/model/model_field';
import { clear, increment } from '@mail/model/model_field_command';

import { qweb } from 'web.core';
import { Markup } from 'web.utils';

registerModel({
    name: 'Chatbot',
    recordMethods: {
        /**
         * Add message posted by the bot into the conversation.
         * This allows not having to wait for the bus (since we run checks based on messages in the
         * conversation, having the result be there immediately eases the process).
         *
         * It also helps while running test tours since those don't have the bus enabled.
         */
        addMessage(message, options) {
            message.body = Markup(message.body);
            this.messaging.publicLivechatGlobal.livechatButtonView.addMessage(message, options);
            if (this.messaging.publicLivechatGlobal.publicLivechat.isFolded || !this.messaging.publicLivechatGlobal.chatWindow.publicLivechatView.widget.isAtBottom()) {
                this.messaging.publicLivechatGlobal.publicLivechat.update({ unreadCounter: increment() });
            }

            if (!options || !options.skipRenderMessages) {
                this.messaging.publicLivechatGlobal.chatWindow.renderMessages();
            }
        },
        /**
         * Once the script ends, adds a visual element at the end of the chat window allowing to restart
         * the whole script.
         */
        endScript() {
            if (
                this.currentStep &&
                this.currentStep.data &&
                this.currentStep.data.conversation_closed
            ) {
                // don't touch anything if the user has closed the conversation, let the chat window
                // handle the display
                return;
            }
            this.messaging.publicLivechatGlobal.chatWindow.widget.$('.o_composer_text_field').addClass('d-none');
            this.messaging.publicLivechatGlobal.chatWindow.widget.$('.o_livechat_chatbot_main_restart').show();
            this.messaging.publicLivechatGlobal.chatWindow.widget.$('.o_livechat_chatbot_end').show();
            this.messaging.publicLivechatGlobal.chatWindow.widget.$('.o_livechat_chatbot_restart').one('click', this.messaging.publicLivechatGlobal.livechatButtonView.onChatbotRestartScript);
        },
        onKeydownInput() {
            if (
                this.currentStep &&
                this.currentStep.data &&
                this.currentStep.data.chatbot_step_type === 'free_input_multi'
            ) {
                this.debouncedAwaitUserInput();
            }
        },
        /**
         * When the user first interacts with the bot, we want to make sure to actually post the welcome
         * messages into the conversation.
         *
         * Indeed, before that, they are 'virtual' messages that are not tied to mail.messages, see
         * #_sendWelcomeChatbotMessage() for more information.
         *
         * Posting them as real messages allows to have a cleaner model and conversation, that will be
         * kept intact when changing page on the website.
         *
         * It also allows tying any first response / question_selection choice to a chatbot.message
         * that has a linked mail.message.
         */
        async postWelcomeMessages() {
            const welcomeMessages = this.messaging.publicLivechatGlobal.welcomeMessages;

            if (welcomeMessages.length === 0) {
                // we already posted the welcome messages, nothing to do
                return;
            }

            const postedWelcomeMessages = await this.messaging.rpc({
                route: '/chatbot/post_welcome_steps',
                params: {
                    channel_uuid: this.messaging.publicLivechatGlobal.publicLivechat.uuid,
                    chatbot_script_id: this.scriptId,
                },
            });

            const welcomeMessagesIds = welcomeMessages.map(welcomeMessage => welcomeMessage.id);
            this.messaging.publicLivechatGlobal.update({
                messages: this.messaging.publicLivechatGlobal.messages.filter((message) => {
                    !welcomeMessagesIds.includes(message.id);
                }),
            });

            postedWelcomeMessages.reverse();
            postedWelcomeMessages.forEach((message) => {
                this.addMessage(message, {
                    prepend: true,
                    skipRenderMessages: true,
                });
            });

            this.messaging.publicLivechatGlobal.chatWindow.renderMessages();
        },
        /**
         * Processes the step, depending on the current state of the script and the author of the last
         * message that was typed into the conversation.
         *
         * This is a rather complicated process since we have many potential states to handle.
         * Here are the detailed possible outcomes:
         *
         * - Check if the script is finished, and if so end it.
         *
         * - If a human operator has taken over the conversation
         *   -> enable the input and let the operator handle the visitor.
         *
         * - If the received step is of type expecting an input from the user
         *   - the last message if from the user (he has already answered)
         *     -> trigger the next step
         *   - otherwise
         *     -> enable the input and let the user type
         *
         * - Otherwise
         *   - if the step is of type 'question_selection' and we are still waiting for the user to
         *     select one of the options
         *     -> don't do anything, wait for the user to click one of the options
         *   - otherwise
         *     -> trigger the next step
         */
        processStep() {
            if (this.shouldEndScript) {
                this.endScript();
            } else if (
                this.currentStep.data.chatbot_step_type === 'forward_operator' &&
                this.currentStep.data.chatbot_operator_found
            ) {
                this.messaging.publicLivechatGlobal.chatWindow.enableInput();
            } else if (this.isExpectingUserInput) {
                if (this.messaging.publicLivechatGlobal.isLastMessageFromCustomer) {
                    // user has already typed a message in -> trigger next step
                    this.setIsTyping();
                    this.update({
                        nextStepTimeout: setTimeout(
                            this.triggerNextStep,
                            this.messageDelay,
                        ),
                    });
                } else {
                    this.messaging.publicLivechatGlobal.chatWindow.enableInput();
                }
            } else {
                let triggerNextStep = true;
                if (this.currentStep.data.chatbot_step_type === 'question_selection') {
                    if (!this.messaging.publicLivechatGlobal.isLastMessageFromCustomer) {
                        // if there is no last message or if the last message is from the bot
                        // -> don't trigger the next step, we are waiting for the user to pick an option
                        triggerNextStep = false;
                    }
                }

                if (triggerNextStep) {
                    let nextStepDelay = this.messageDelay;
                    if (this.messaging.publicLivechatGlobal.chatWindow.widget.$('.o_livechat_chatbot_typing').length !== 0) {
                        // special case where we already have a "is typing" message displayed
                        // can happen when the previous step did not trigger any message posted from the bot
                        // e.g: previous step was "forward_operator" and no-one is available
                        // -> in that case, don't wait and trigger the next step immediately
                        nextStepDelay = 0;
                    } else {
                        this.setIsTyping();
                    }

                    this.update({
                        nextStepTimeout: setTimeout(
                            this.triggerNextStep,
                            nextStepDelay,
                        ),
                    });
                }
            }

            if (!this.hasRestartButton) {
                this.messaging.publicLivechatGlobal.chatWindow.widget.$('.o_livechat_chatbot_main_restart').hide();
            }
        },
        /**
         * See 'Chatbot/saveSession'.
         *
         * We retrieve the livechat uuid from the session cookie since the livechat Widget is not yet
         * initialized when we restore the chatbot state.
         *
         * We also clear any older keys that store a previously saved chatbot session.
         * (In that case we clear the actual browser's local storage, we don't use the localStorage
         * object as it does not allow browsing existing keys, see 'local_storage.js'.)
         */
        restoreSession() {
            const browserLocalStorage = window.localStorage;
            if (browserLocalStorage && browserLocalStorage.length) {
                for (let i = 0; i < browserLocalStorage.length; i++) {
                    const key = browserLocalStorage.key(i);
                    if (key.startsWith('im_livechat.chatbot.state.uuid_') && key !== this.sessionCookieKey) {
                        browserLocalStorage.removeItem(key);
                    }
                }
            }
            const chatbotState = localStorage.getItem(this.sessionCookieKey);
            if (chatbotState) {
                this.update({ currentStep: { data: this.localStorageState._chatbotCurrentStep } });
            }
        },
        /**
         * Register current chatbot step state into localStorage to be able to resume if the visitor
         * goes to another website page or if he refreshes his page.
         *
         * (Will not work if the visitor switches browser but his livechat session will not be restored
         *  anyway in that case, since it's stored into a cookie).
         */
        saveSession() {
            localStorage.setItem('im_livechat.chatbot.state.uuid_' + this.messaging.publicLivechatGlobal.publicLivechat.uuid, JSON.stringify({
                '_chatbot': this.data,
                '_chatbotCurrentStep': this.currentStep.data,
            }));
        },
        /**
         * Adds a small "is typing" animation into the chat window.
         *
         * @param {boolean} [isWelcomeMessage=false]
         */
        setIsTyping(isWelcomeMessage = false) {
            if (this.messaging.publicLivechatGlobal.livechatButtonView.isTypingTimeout) {
                clearTimeout(this.messaging.publicLivechatGlobal.livechatButtonView.isTypingTimeout);
            }
            this.messaging.publicLivechatGlobal.chatWindow.disableInput('');
            this.messaging.publicLivechatGlobal.livechatButtonView.update({
                isTypingTimeout: setTimeout(
                    () => {
                        if (!this.messaging.publicLivechatGlobal.chatWindow || !this.messaging.publicLivechatGlobal.chatWindow.exists()) {
                            return;
                        }
                        this.messaging.publicLivechatGlobal.chatWindow.widget.$('.o_mail_thread_content').append(
                            $(qweb.render('im_livechat.legacy.chatbot.is_typing_message', {
                                'chatbotImageSrc': this.messaging.publicLivechatGlobal.serverUrl + `/im_livechat/operator/${
                                    this.messaging.publicLivechatGlobal.publicLivechat.operator.id
                                }/avatar`,
                                'chatbotIsTypingImageSrc': this.messaging.publicLivechatGlobal.serverUrl + '/im_livechat/static/src/img/chatbot_is_typing.gif',
                                'chatbotName': this.name,
                                'isWelcomeMessage': isWelcomeMessage,
                            }))
                        );
                        this.messaging.publicLivechatGlobal.chatWindow.publicLivechatView.widget.scrollToBottom();
                    },
                    this.messageDelay / 3,
                ),
            });
        },
        /**
         * Triggers the next step of the script by calling the associated route.
         * This will receive the next step and call step processing.
         */
        async triggerNextStep() {
            if (!this.messaging.publicLivechatGlobal.chatWindow || !this.messaging.publicLivechatGlobal.chatWindow.exists()) {
                return;
            }
            let triggerNextStep = true;
            if (
                this.currentStep &&
                this.currentStep.data &&
                this.currentStep.data.chatbot_step_type === 'question_email'
            ) {
                triggerNextStep = await this.validateEmail();
            }

            if (!triggerNextStep) {
                return;
            }

            const nextStep = await this.messaging.rpc({
                route: '/chatbot/step/trigger',
                params: {
                    channel_uuid: this.messaging.publicLivechatGlobal.publicLivechat.uuid,
                    chatbot_script_id: this.scriptId,
                },
            });

            if (nextStep) {
                if (nextStep.chatbot_posted_message) {
                    this.addMessage(nextStep.chatbot_posted_message);
                }

                this.update({ currentStep: { data: nextStep.chatbot_step } });

                this.processStep();
            } else {
                // did not find next step -> end the script
                this.currentStep.data.chatbot_step_is_last = true;
                this.messaging.publicLivechatGlobal.chatWindow.renderMessages();
                this.endScript();
            }

            this.saveSession();

            return nextStep;
        },
        /**
         * A special case is handled for email steps, where we first validate the email (server side)
         * and we allow the user to try again in case the format is incorrect.
         *
         * The validation is made server-side to have the same test when we validate here and when we
         * register the answer, but also to easily post a message as the bot ("Sorry, try again...").
         *
         * Returns a boolean stating whether the email was valid or not.
         */
        async validateEmail() {
            let emailValidResult = await this.messaging.rpc({
                route: '/chatbot/step/validate_email',
                params: { channel_uuid: this.messaging.publicLivechatGlobal.publicLivechat.uuid },
            });

            if (emailValidResult.success) {
                this.currentStep.data.is_email_valid = true;
                this.saveSession();

                return true;
            } else {
                // email is not valid, let the user try again
                this.messaging.publicLivechatGlobal.chatWindow.enableInput();
                if (emailValidResult.posted_message) {
                    this.addMessage(emailValidResult.posted_message);
                }

                return false;
            }
        },
        /**
         * This method will be transformed into a 'debounced' version (see init).
         *
         * The purpose is to handle steps of type 'free_input_multi', that will let the user type in
         * multiple lines of text before the bot goes to the next step.
         *
         * Every time a 'keydown' is detected into the input, or every time a message is sent, we call
         * this debounced method, which will give the user about 10 seconds to type more text before
         * the next step is triggered.
         *
         * First we check if the last message was sent by the user, to make sure we always let him type
         * at least one message before moving on.
         */
        awaitUserInput() {
            if (this.messaging.publicLivechatGlobal.isLastMessageFromCustomer) {
                if (this.shouldEndScript) {
                    this.endScript();
                } else {
                    this.setIsTyping();
                    this.update({
                        nextStepTimeout: setTimeout(
                            this.triggerNextStep,
                            this.messageDelay,
                        )
                    });
                }
            }
        },
    },
    fields: {
        awaitUserInputDebounceTime: attr({
            compute() {
                return 10000;
            },
        }),
        data: attr({
            compute() {
                if (this.messaging.publicLivechatGlobal.isTestChatbot) {
                    return this.messaging.publicLivechatGlobal.testChatbotData;
                }
                if (this.state === 'init') {
                    return this.messaging.publicLivechatGlobal.rule.chatbot;
                }
                if (this.state === 'welcome') {
                    return this.messaging.publicLivechatGlobal.livechatInit.rule.chatbot;
                }
                if (
                    this.state === 'restore_session' &&
                    this.localStorageState
                ) {
                    return this.localStorageState._chatbot;
                }
                return clear();
            },
        }),
        currentStep: one('ChatbotStep', {
            inverse: 'chabotOwner',
        }),
        debouncedAwaitUserInput: attr({
            compute() {
                // debounced to let the user type several sentences, see 'Chatbot/awaitUserInput' for details
                return _.debounce(
                    this.awaitUserInput,
                    this.awaitUserInputDebounceTime,
                );
            },
        }),
        hasRestartButton: attr({
            /**
             * Will display a "Restart script" button in the conversation toolbar.
             *
             * Side-case: if the conversation has been forwarded to a human operator, we don't want to
             * display that restart button.
             */
            compute() {
                const { publicLivechat } = this.messaging.publicLivechatGlobal;
                if (publicLivechat && !publicLivechat.operator) {
                    return false;
                }
                if (
                    !this.messaging.publicLivechatGlobal.publicLivechat ||
                    !this.messaging.publicLivechatGlobal.publicLivechat.uuid
                ) {
                    return false;
                }
                if (publicLivechat && !publicLivechat.data.chatbot_script_id) {
                    return false;
                }
                return Boolean(
                    !this.currentStep ||
                    (
                        this.currentStep.data.chatbot_step_type !== 'forward_operator' ||
                        !this.currentStep.data.chatbot_operator_found
                    )
                );
            },
            default: false,
        }),
        isActive: attr({
            compute() {
                if (this.messaging.publicLivechatGlobal.isTestChatbot) {
                    return true;
                }
                if (this.messaging.publicLivechatGlobal.rule && this.messaging.publicLivechatGlobal.rule.chatbot) {
                    return true;
                }
                if (this.messaging.publicLivechatGlobal.livechatInit && this.messaging.publicLivechatGlobal.livechatInit.rule.chatbot) {
                    return true;
                }
                if (this.state === 'welcome') {
                    return true;
                }
                if (this.localStorageState) {
                    return true;
                }
                return clear();
            },
            default: false,
        }),
        isExpectingUserInput: attr({
            compute() {
                if (!this.currentStep) {
                    return clear();
                }
                return [
                    'question_phone',
                    'question_email',
                    'free_input_single',
                    'free_input_multi',
                ].includes(this.currentStep.data.chatbot_step_type);
            },
            default: false,
        }),
        isRedirecting: attr({
            default: false,
        }),
        lastWelcomeStep: attr({
            compute() {
                if (!this.welcomeSteps) {
                    return clear();
                }
                return this.welcomeSteps[this.welcomeSteps.length - 1];
            },
        }),
        localStorageState: attr({
            compute() {
                if (!this.messaging.publicLivechatGlobal.sessionCookie) {
                    return clear();
                }
                const data = localStorage.getItem(this.sessionCookieKey);
                if (!data) {
                    return clear();
                }
                return JSON.parse(data);
            },
        }),
        name: attr({
            compute() {
                if (!this.data) {
                    return clear();
                }
                return this.data.name;
            },
        }),
        nextStepTimeout: attr(),
        messageDelay: attr({
            compute() {
                return clear();
            },
            default: 3500, // in milliseconds
        }),
        publicLivechatGlobalOwner: one('PublicLivechatGlobal', {
            identifying: true,
            inverse: 'chatbot',
        }),
        scriptId: attr({
            compute() {
                if (!this.data) {
                    return clear();
                }
                return this.data.chatbot_script_id;
            },
        }),
        serverUrl: attr(),
        sessionCookieKey: attr({
            compute() {
                if (!this.messaging.publicLivechatGlobal.sessionCookie) {
                    return clear();
                }
                return 'im_livechat.chatbot.state.uuid_' + JSON.parse(this.messaging.publicLivechatGlobal.sessionCookie).uuid;
            },
        }),
        shouldEndScript: attr({
            /**
             * Compute method that checks if the script should be ended or not.
             * If the user has closed the conversation -> script has ended.
             *
             * Otherwise, there are 2 use cases where we want to end the script:
             *
             * If the current step is the last one AND the conversation was not taken over by a human operator
             *   1. AND we expect a user input (or we are on a selection)
             *       AND the user has already answered
             *   2. AND we don't expect a user input
             */
            compute() {
                if (!this.currentStep) {
                    return clear();
                }
                if (this.currentStep.data.conversation_closed) {
                    return true;
                }
                if (this.currentStep.data.chatbot_step_is_last &&
                    (this.currentStep.data.chatbot_step_type !== 'forward_operator' ||
                    !this.currentStep.data.chatbot_operator_found)
                ) {
                    if (this.currentStep.data.chatbot_step_type === 'question_email'
                        && !this.currentStep.data.is_email_valid
                    ) {
                        // email is not (yet) valid, let the user answer / try again
                        return false;
                    } else if (
                        (this.isExpectingUserInput ||
                        this.currentStep.data.chatbot_step_type === 'question_selection') &&
                        this.messaging.publicLivechatGlobal.messages.length !== 0
                    ) {
                        if (this.messaging.publicLivechatGlobal.lastMessage.authorId !== this.messaging.publicLivechatGlobal.publicLivechat.operator.id) {
                            // we are on the last step of the script, expect a user input and the user has
                            // already answered
                            // -> end the script
                            return true;
                        }
                    } else if (!this.isExpectingUserInput) {
                        // we are on the last step of the script and we do not expect a user input
                        // -> end the script
                        return true;
                    }
                }
                return false;
            },
            default: false,
        }),
        state: attr({
            compute() {
                if (this.messaging.publicLivechatGlobal.rule && !!this.messaging.publicLivechatGlobal.rule.chatbot) {
                    return 'init';
                }
                if (this.messaging.publicLivechatGlobal.livechatInit && this.messaging.publicLivechatGlobal.livechatInit.rule.chatbot) {
                    return 'welcome';
                }
                if (
                    !this.messaging.publicLivechatGlobal.rule &&
                    this.messaging.publicLivechatGlobal.history !== null &&
                    this.messaging.publicLivechatGlobal.history.length !== 0 &&
                    this.sessionCookieKey &&
                    localStorage.getItem(this.sessionCookieKey)
                ) {
                    return 'restore_session';
                }
                return clear();
            },
        }),
        welcomeMessageTimeout: attr(),
        welcomeSteps: attr({
            compute() {
                if (!this.data) {
                    return clear();
                }
                return this.data.chatbot_welcome_steps;
            },
        }),
    },
});

```

## File: static\src\public_models\chatbot_step.js

```javascript
/** @odoo-module **/

import { registerModel } from '@mail/model/model_core';
import { attr, one } from '@mail/model/model_field';

registerModel({
    name: 'ChatbotStep',
    fields: {
        chabotOwner: one('Chatbot', {
            identifying: true,
            inverse: 'currentStep',
        }),
        data: attr(),
    },
});

```

## File: static\src\public_models\livechat_button_view.js

```javascript
/** @odoo-module **/

import { registerModel } from '@mail/model/model_core';
import { attr, one } from '@mail/model/model_field';
import { clear } from '@mail/model/model_field_command';

import {getCookie, deleteCookie} from 'web.utils.cookies';

registerModel({
    name: 'LivechatButtonView',
    lifecycleHooks: {
        _created() {
            this.update({ widget: this.env.services.public_livechat_service.mountLivechatButton() });
        },
        _willDelete() {
            this.widget.destroy();
        },
    },
    recordMethods: {
        /**
         * @param {Object} data
         * @param {Object} [options={}]
         */
        addMessage(data, options) {
            const hasAlreadyMessage = _.some(this.messaging.publicLivechatGlobal.messages, function (msg) {
                return data.id === msg.id;
            });
            if (hasAlreadyMessage) {
                return;
            }
            const message = this.messaging.models['PublicLivechatMessage'].insert({
                data,
                id: data.id,
            });

            if (this.messaging.publicLivechatGlobal.publicLivechat && this.messaging.publicLivechatGlobal.publicLivechat.widget) {
                this.messaging.publicLivechatGlobal.publicLivechat.widget.addMessage(message.widget);
            }

            if (options && options.prepend) {
                this.messaging.publicLivechatGlobal.update({
                    messages: [message, ...this.messaging.publicLivechatGlobal.messages],
                });
            } else {
                this.messaging.publicLivechatGlobal.update({
                    messages: [...this.messaging.publicLivechatGlobal.messages, message],
                });
            }
        },
        askFeedback() {
            this.messaging.publicLivechatGlobal.chatWindow.widget.$('.o_thread_composer input').prop('disabled', true);
            this.messaging.publicLivechatGlobal.update({ feedbackView: {} });
            /**
             * When we enter the "ask feedback" process of the chat, we hide some elements that become
             * unnecessary and irrelevant (restart / end messages, any text field values, ...).
             */
            if (
                this.messaging.publicLivechatGlobal.chatbot.currentStep &&
                this.messaging.publicLivechatGlobal.chatbot.currentStep.data
            ) {
                this.messaging.publicLivechatGlobal.chatbot.currentStep.data.conversation_closed = true;
                this.messaging.publicLivechatGlobal.chatbot.saveSession();
            }
            this.messaging.publicLivechatGlobal.chatWindow.widget.$('.o_livechat_chatbot_main_restart').hide();
            this.messaging.publicLivechatGlobal.chatWindow.widget.$('.o_livechat_chatbot_end').hide();
            this.messaging.publicLivechatGlobal.chatWindow.widget.$('.o_composer_text_field')
                .removeClass('d-none')
                .val('');
        },
        /**
         * Restart the script and then trigger the "next step" (which will be the first of the script
         * in this case).
         */
        async onChatbotRestartScript(ev) {
            this.messaging.publicLivechatGlobal.chatWindow.widget.$('.o_composer_text_field').removeClass('d-none');
            this.messaging.publicLivechatGlobal.chatWindow.widget.$('.o_livechat_chatbot_end').hide();

            if (this.messaging.publicLivechatGlobal.chatbot.nextStepTimeout) {
                clearTimeout(this.messaging.publicLivechatGlobal.chatbot.nextStepTimeout);
            }

            if (this.messaging.publicLivechatGlobal.chatbot.welcomeMessageTimeout) {
                clearTimeout(this.messaging.publicLivechatGlobal.chatbot.welcomeMessageTimeout);
            }
            if (this.messaging.publicLivechatGlobal.publicLivechat.uuid) {
                const postedMessage = await this.messaging.rpc({
                    route: '/chatbot/restart',
                    params: {
                        channel_uuid: this.messaging.publicLivechatGlobal.publicLivechat.uuid,
                        chatbot_script_id: this.messaging.publicLivechatGlobal.chatbot.scriptId,
                    },
                });

                if (postedMessage) {
                    this.messaging.publicLivechatGlobal.chatbot.addMessage(postedMessage);
                }
            }

            this.messaging.publicLivechatGlobal.chatbot.update({ currentStep: clear() });
            this.messaging.publicLivechatGlobal.chatbot.setIsTyping();
            this.messaging.publicLivechatGlobal.chatbot.update({
                nextStepTimeout: setTimeout(
                    this.messaging.publicLivechatGlobal.chatbot.triggerNextStep,
                    this.messaging.publicLivechatGlobal.chatbot.messageDelay,
                ),
            });
        },
        closeChat() {
            this.messaging.publicLivechatGlobal.update({ chatWindow: clear() });
            deleteCookie('im_livechat_session');
        },
        openChat() {
            if (this.isOpenChatDebounced) {
                this.openChatDebounced();
            } else {
                this._openChat();
            }
        },
        async openChatWindow() {
            this.messaging.publicLivechatGlobal.update({ chatWindow: {} });
            await this.messaging.publicLivechatGlobal.chatWindow.widget.appendTo($('body'));
            this.messaging.publicLivechatGlobal.chatWindow.widget.adjustPosition();
            this.widget.$el.hide();
            this._openChatWindowChatbot();
        },
        /**
         * @param {Object} message
         */
        async sendMessage(message) {
            if (this.messaging.publicLivechatGlobal.publicLivechat.isTemporary) {
                await this.messaging.publicLivechatGlobal.publicLivechat.createLivechatChannel();
                if (!this.messaging.publicLivechatGlobal.publicLivechat.operator) {
                    return;
                }
            }
            await this._sendMessageChatbotBefore();
            await this._sendMessage(message);
            this._sendMessageChatbotAfter();
        },
        async start() {
            if (!this.messaging.publicLivechatGlobal.hasWebsiteLivechatFeature) {
                this.widget.$el.text(this.buttonText);
            }
            this.update({ isWidgetMounted: true });
            if (this.messaging.publicLivechatGlobal.history) {
                for (const m of this.messaging.publicLivechatGlobal.history) {
                    this.addMessage(m);
                }
                await this.openChat();
            } else if (!this.messaging.device.isSmall && this.messaging.publicLivechatGlobal.rule.action === 'auto_popup') {
                const autoPopupCookie = getCookie('im_livechat_auto_popup');
                if (!autoPopupCookie || JSON.parse(autoPopupCookie)) {
                    this.update({
                        autoOpenChatTimeout: setTimeout(
                            this.openChat,
                            this.messaging.publicLivechatGlobal.rule.auto_popup_timer * 1000,
                        ),
                    });
                }
            }
            if (this.buttonBackgroundColor) {
                this.widget.$el.css('background-color', this.buttonBackgroundColor);
            }
            if (this.buttonTextColor) {
                this.widget.$el.css('color', this.buttonTextColor);
            }
            // If website_event_track installed, put the livechat banner above the PWA banner.
            const pwaBannerHeight = $('.o_pwa_install_banner').outerHeight(true);
            if (pwaBannerHeight) {
                this.widget.$el.css('bottom', pwaBannerHeight + 'px');
            }
        },
        /**
         * @private
         */
        async _openChat() {
            if (this.isOpeningChat) {
                return;
            }
            const cookie = decodeURIComponent(getCookie('im_livechat_session'));
            let def;
            this.update({ isOpeningChat: true });
            clearTimeout(this.autoOpenChatTimeout);
            if (cookie) {
                def = Promise.resolve(JSON.parse(cookie));
            } else {
                // re-initialize messages cache
                this.messaging.publicLivechatGlobal.update({ messages: clear() });
                def = this.messaging.rpc({
                    route: '/im_livechat/get_session',
                    params: {
                        ...this.widget._prepareGetSessionParameters(),
                        persisted: false,
                    },
                }, { silent: true });
            }
            def.then((livechatData) => {
                if (!livechatData || !livechatData.operator_pid) {
                    try {
                        this.widget.displayNotification({
                            message: this.env._t("No available collaborator, please try again later."),
                            sticky: true,
                        });
                    } catch (_err) {
                        /**
                         * Failure in displaying notification happens when
                         * notification service doesn't exist, which is the case in
                         * external lib. We don't want notifications in external
                         * lib at the moment because they use bootstrap toast and
                         * we don't want to include boostrap in external lib.
                         */
                        console.warn(this.env._t("No available collaborator, please try again later."));
                    }
                } else {
                    this.messaging.publicLivechatGlobal.update({
                        publicLivechat: { data: livechatData },
                    });
                    return this.openChatWindow().then(() => {
                        if (!this.messaging.publicLivechatGlobal.history) {
                            this.widget._sendWelcomeMessage();
                        }
                        this.messaging.publicLivechatGlobal.chatWindow.renderMessages();
                        this.messaging.publicLivechatGlobal.publicLivechat.updateSessionCookie();
                    });
                }
            }).then(() => {
                this.update({ isOpeningChat: false });
            }).guardedCatch(() => {
                this.update({ isOpeningChat: false });
            });
        },
        /**
         * Resuming the chatbot script if we are currently running one.
         *
         * In addition, we register a resize event on the window object to scroll messages to bottom.
         * This is done especially for mobile (Android) where the keyboard opens upon focusing the input
         * field and shrinks the whole window size.
         * Scrolling to the bottom allows the user to see the last messages properly when that happens.
         *
         * @private
         * @override
         */
        _openChatWindowChatbot() {
            window.addEventListener('resize', () => {
                if (this.messaging.publicLivechatGlobal.chatWindow) {
                    this.messaging.publicLivechatGlobal.chatWindow.publicLivechatView.widget.scrollToBottom();
                }
            });

            if (
                this.messaging.publicLivechatGlobal.chatbot.currentStep &&
                this.messaging.publicLivechatGlobal.chatbot.currentStep.data &&
                this.messaging.publicLivechatGlobal.messages &&
                this.messaging.publicLivechatGlobal.messages.length !== 0
            ) {
                this.messaging.publicLivechatGlobal.chatbot.processStep();
            }
        },
        /**
         * @private
         * @param {Object} message
         */
        async _sendMessage(message) {
            this.messaging.publicLivechatGlobal.publicLivechat.widget._notifyMyselfTyping({ typing: false });
            const messageId = await this.messaging.rpc({
                route: '/mail/chat_post',
                params: { uuid: this.messaging.publicLivechatGlobal.publicLivechat.uuid, message_content: message.content },
            });
            if (!messageId) {
                try {
                    this.widget.displayNotification({
                        message: this.env._t("Session expired... Please refresh and try again."),
                        sticky: true,
                    });
                } catch (_err) {
                    /**
                     * Failure in displaying notification happens when
                     * notification service doesn't exist, which is the case
                     * in external lib. We don't want notifications in
                     * external lib at the moment because they use bootstrap
                     * toast and we don't want to include boostrap in
                     * external lib.
                     */
                    console.warn(this.env._t("Session expired... Please refresh and try again."));
                }
                this.closeChat();
            }
            this.messaging.publicLivechatGlobal.chatWindow.publicLivechatView.widget.scrollToBottom();
        },
        /**
         * @private
         */
        _sendMessageChatbotAfter() {
            if (this.messaging.publicLivechatGlobal.chatbot.isRedirecting) {
                return;
            }
            if (
                this.messaging.publicLivechatGlobal.chatbot.isActive &&
                this.messaging.publicLivechatGlobal.chatbot.currentStep &&
                this.messaging.publicLivechatGlobal.chatbot.currentStep.data
            ) {
                if (
                    this.messaging.publicLivechatGlobal.chatbot.currentStep.data.chatbot_step_type === 'forward_operator' &&
                    this.messaging.publicLivechatGlobal.chatbot.currentStep.data.chatbot_operator_found
                ) {
                    return; // operator has taken over the conversation, let them speak
                } else if (this.messaging.publicLivechatGlobal.chatbot.currentStep.data.chatbot_step_type === 'free_input_multi') {
                    this.messaging.publicLivechatGlobal.chatbot.debouncedAwaitUserInput();
                } else if (!this.messaging.publicLivechatGlobal.chatbot.shouldEndScript) {
                    this.messaging.publicLivechatGlobal.chatbot.setIsTyping();
                    this.messaging.publicLivechatGlobal.chatbot.update({
                        nextStepTimeout: setTimeout(
                            this.messaging.publicLivechatGlobal.chatbot.triggerNextStep,
                            this.messaging.publicLivechatGlobal.chatbot.messageDelay,
                        ),
                    });
                } else {
                    this.messaging.publicLivechatGlobal.chatbot.endScript();
                }
                this.messaging.publicLivechatGlobal.chatbot.saveSession();
            }
        },
        /**
         * When the Customer sends a message, we need to act depending on our current state:
         * - If the conversation has been forwarded to an operator
         *   Then there is nothing to do, we let them speak
         * - If we are currently on a 'free_input_multi' step
         *   Await more user input (see #Chatbot/awaitUserInput for details)
         * - Otherwise we continue the script or end it if it's the last step
         *
         * We also save the current session state.
         * Important as this may be the very first interaction with the bot, we need to save right away
         * to correctly handle any page redirection / page refresh.
         *
         * Special side case: if we are currently redirecting to another page (see '_onChatbotOptionClicked')
         * we shortcut the process as we are currently moving to a different URL.
         * The script will be resumed on the new page (if in the same website domain).
         *
         * @private
         */
        async _sendMessageChatbotBefore() {
            if (
                this.messaging.publicLivechatGlobal.chatbot.isActive &&
                this.messaging.publicLivechatGlobal.chatbot.currentStep &&
                this.messaging.publicLivechatGlobal.chatbot.currentStep.data
            ) {
                await this.messaging.publicLivechatGlobal.chatbot.postWelcomeMessages();
            }
        },
    },
    fields: {
        autoOpenChatTimeout: attr(),
        buttonBackgroundColor: attr({
            compute() {
                return this.messaging.publicLivechatGlobal.options.button_background_color;
            },
        }),
        buttonText: attr({
            compute() {
                if (this.messaging.publicLivechatGlobal.options.button_text) {
                    return this.messaging.publicLivechatGlobal.options.button_text;
                }
                return this.env._t("Chat with one of our collaborators");
            },
        }),
        buttonTextColor: attr({
            compute() {
                return this.messaging.publicLivechatGlobal.options.button_text_color;
            },
        }),
        chatbotNextStepTimeout: attr(),
        chatbotWelcomeMessageTimeout: attr(),
        currentPartnerId: attr({
            compute() {
                if (!this.messaging.publicLivechatGlobal.isAvailable) {
                    return clear();
                }
                return this.messaging.publicLivechatGlobal.options.current_partner_id;
            },
        }),
        defaultMessage: attr({
            compute() {
                if (this.messaging.publicLivechatGlobal.options.default_message) {
                    return this.messaging.publicLivechatGlobal.options.default_message;
                }
                return this.env._t("How may I help you?");
            },
        }),
        defaultUsername: attr({
            compute() {
                if (this.messaging.publicLivechatGlobal.options.default_username) {
                    return this.messaging.publicLivechatGlobal.options.default_username;
                }
                return this.env._t("Visitor");
            },
        }),
        headerBackgroundColor: attr({
            compute() {
                return this.messaging.publicLivechatGlobal.options.header_background_color;
            },
        }),
        inputPlaceholder: attr({
            compute() {
                if (this.messaging.publicLivechatGlobal.chatbot.isActive) {
                    // void the default livechat placeholder in the user input
                    // as we use it for specific things (e.g: showing "please select an option above")
                    return clear();
                }
                if (this.messaging.publicLivechatGlobal.options.input_placeholder) {
                    return this.messaging.publicLivechatGlobal.options.input_placeholder;
                }
                return this.env._t("Ask something ...");
            },
            default: '',
        }),
        isOpenChatDebounced: attr({
            compute() {
                return clear();
            },
            default: true,
        }),
        isOpeningChat: attr({
            default: false,
        }),
        isTypingTimeout: attr(),
        isWidgetMounted: attr({
            default: false,
        }),
        openChatDebounced: attr({
            compute() {
                return _.debounce(this._openChat, 200, true);
            },
        }),
        publicLivechatGlobalOwner: one('PublicLivechatGlobal', {
            identifying: true,
            inverse: 'livechatButtonView',
        }),
        serverUrl: attr({
            compute() {
                return this.messaging.publicLivechatGlobal.serverUrl;
            },
        }),
        titleColor: attr({
            compute() {
                return this.messaging.publicLivechatGlobal.options.title_color;
            },
        }),
        widget: attr(),
    },
});

```

## File: static\src\public_models\livechat_operator.js

```javascript
/** @odoo-module **/

import { registerModel } from '@mail/model/model_core';
import { attr } from '@mail/model/model_field';

registerModel({
    name: 'LivechatOperator',
    fields: {
        id: attr({
            identifying: true,
        }),
        name: attr(),
    },
});

```

## File: static\src\public_models\messaging.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { one } from '@mail/model/model_field';

registerPatch({
    name: 'Messaging',
    fields: {
        publicLivechatGlobal: one('PublicLivechatGlobal', {
            isCausal: true,
        }),
    },
});

```

## File: static\src\public_models\public_livechat.js

```javascript
/** @odoo-module **/

import PublicLivechat from '@im_livechat/legacy/models/public_livechat';

import { registerModel } from '@mail/model/model_core';
import { attr, one } from '@mail/model/model_field';
import { clear } from '@mail/model/model_field_command';

import { deleteCookie, setCookie } from 'web.utils.cookies';

registerModel({
    name: 'PublicLivechat',
    lifecycleHooks: {
        _created() {
            this.update({
                widget: new PublicLivechat(this.messaging, {
                    parent: this.publicLivechatGlobalOwner.livechatButtonView.widget,
                    data: this.data,
                }),
            });
        },
        _willDelete() {
            this.widget.destroy();
        },
    },
    recordMethods: {
        async createLivechatChannel() {
            const livechatData = await this.messaging.rpc({
                route: "/im_livechat/get_session",
                params: this.messaging.publicLivechatGlobal.livechatButtonView.widget._prepareGetSessionParameters(),
            });
            if (!livechatData || !livechatData.operator_pid) {
                this.update({ data: clear() });
                deleteCookie("im_livechat_session");
                this.messaging.publicLivechatGlobal.chatWindow.widget.renderChatWindow();
            } else {
                this.update({ data: livechatData });
                this.widget.data = livechatData;
                this.updateSessionCookie();
            }
        },
        updateSessionCookie() {
            deleteCookie("im_livechat_session");
            setCookie(
                "im_livechat_session",
                encodeURIComponent(JSON.stringify(this.widget.toData()), true),
                60 * 60,
                "required"
            );
            setCookie("im_livechat_auto_popup", JSON.stringify(false), 60 * 60, "optional");
            if (this.operator) {
                const operatorPidId = this.operator.id;
                const oneWeek = 7 * 24 * 60 * 60;
                setCookie("im_livechat_previous_operator_pid", operatorPidId, oneWeek, "optional");
            }
        },
    },
    fields: {
        data: attr(),
        id: attr({
            compute() {
                if (!this.data) {
                    return clear();
                }
                return this.data.id;
            },
        }),
        isFolded: attr({
            default: false,
        }),
        isTemporary: attr({
            compute() {
                if (!this.data || !this.data.id) {
                    return true;
                }
                return false;
            },
        }),
        publicLivechatGlobalOwner: one('PublicLivechatGlobal', {
            identifying: true,
            inverse: 'publicLivechat',
        }),
        name: attr({
            compute() {
                if (!this.data || !this.operator) {
                    return clear();
                }
                return this.data.name;
            },
        }),
        operator: one('LivechatOperator', {
            compute() {
                if (!this.data) {
                    return clear();
                }
                if (!this.data.operator_pid) {
                    return clear();
                }
                if (!this.data.operator_pid[0]) {
                    return clear();
                }
                return {
                    id: this.data.operator_pid[0],
                    name: this.data.operator_pid[1],
                };
            },
        }),
        status: attr({
            compute() {
                if (!this.data) {
                    return clear();
                }
                return this.data.status || '';
            },
        }),
        // amount of messages that have not yet been read on this chat
        unreadCounter: attr({
            default: 0,
        }),
        uuid: attr({
            compute() {
                if (!this.data) {
                    return clear();
                }
                return this.data.uuid;
            },
        }),
        widget: attr(),
    },
});

```

## File: static\src\public_models\public_livechat_feedback_view.js

```javascript
/** @odoo-module **/

import Feedback from '@im_livechat/legacy/widgets/feedback/feedback';

import { registerModel } from '@mail/model/model_core';
import { attr, one } from '@mail/model/model_field';

registerModel({
    name: 'PublicLivechatFeedbackView',
    lifecycleHooks: {
        _created() {
            this.update({
                widget: new Feedback(
                    this.messaging.publicLivechatGlobal.livechatButtonView.widget,
                    this.messaging,
                    this.messaging.publicLivechatGlobal.publicLivechat.widget,
                ),
            });
            this.messaging.publicLivechatGlobal.chatWindow.widget.replaceContentWith(this.widget);
            this.widget.on('feedback_sent', null, this._onFeedbackSent);
            this.widget.on('send_message', null, this._onSendMessage);
        },
        _willDelete() {
            this.widget.destroy();
        },
    },
    recordMethods: {
        _onFeedbackSent() {
            this.messaging.publicLivechatGlobal.livechatButtonView.closeChat();
        },
        _onSendMessage(...args) {
            this.messaging.publicLivechatGlobal.livechatButtonView.sendMessage(...args);
        },
    },
    fields: {
        publicLivechatGlobalOwner: one('PublicLivechatGlobal', {
            identifying: true,
            inverse: 'feedbackView',
        }),
        widget: attr(),
    },
});

```

## File: static\src\public_models\public_livechat_global.js

```javascript
/** @odoo-module **/

import { registerModel } from '@mail/model/model_core';
import { attr, many, one } from '@mail/model/model_field';
import { clear } from '@mail/model/model_field_command';

import { session } from "@web/session";
import legacySession from "web.session";

import { qweb } from 'web.core';
import { Markup } from 'web.utils';
import {getCookie, setCookie, deleteCookie} from 'web.utils.cookies';

registerModel({
    name: 'PublicLivechatGlobal',
    lifecycleHooks: {
        _created() {
            // History tracking
            const page = window.location.href.replace(/^.*\/\/[^/]+/, '');
            const pageHistory = getCookie(this.LIVECHAT_COOKIE_HISTORY);
            let urlHistory = [];
            if (pageHistory) {
                urlHistory = JSON.parse(pageHistory) || [];
            }
            if (!_.contains(urlHistory, page)) {
                urlHistory.push(page);
                while (urlHistory.length > this.HISTORY_LIMIT) {
                    urlHistory.shift();
                }
                setCookie(this.LIVECHAT_COOKIE_HISTORY, JSON.stringify(urlHistory), 60 * 60 * 24, 'optional'); // 1 day cookie
            }
            if (this.isAvailable) {
                this.willStart();
            }
        },
    },
    recordMethods: {
        async loadQWebTemplate() {
            const templates = await this.messaging.rpc({ route: '/im_livechat/load_templates' });
            for (const template of templates) {
                qweb.add_template(template);
            }
            this.update({ hasLoadedQWebTemplate: true });
        },
        async willStart() {
            await this._willStart();
            await this._willStartChatbot();
        },
        async _willStart() {
            const strCookie = decodeURIComponent(getCookie('im_livechat_session'));
            let isSessionCookieAvailable = Boolean(strCookie);
            let cookie = JSON.parse(strCookie || '{}');
            if (isSessionCookieAvailable && (cookie.visitor_uid !== session.user_id || !cookie.id)) {
                this.leaveSession();
                isSessionCookieAvailable = false;
                cookie = {};
            }
            if (cookie.id) {
                const history = await this.messaging.rpc({
                    route: '/mail/chat_history',
                    params: { uuid: cookie.uuid, limit: 100 },
                });
                history.reverse();
                this.update({ history });
                for (const message of this.history) {
                    message.body = Markup(message.body);
                }
                this.update({ isAvailableForMe: true });
            } else {
                const result = await this.messaging.rpc({
                    route: '/im_livechat/init',
                    params: { channel_id: this.channelId },
                });
                if (result.available_for_me) {
                    this.update({ isAvailableForMe: true });
                }
                this.update({ rule: result.rule });
            }
            const proms = [this.loadQWebTemplate()];
            if (!session.is_frontend) {
                proms.push(legacySession.load_translations(["im_livechat"]));
            }
            return Promise.all(proms);
        },
        /**
         * This override handles the following use cases:
         *
         * - If the chat is started for the first time (first visit of a visitor)
         *   We register the chatbot configuration and the rest of the behavior is triggered by various
         *   method overrides ('sendWelcomeMessage', 'sendMessage', ...)
         *
         * - If the chat has been started before, but the user did not interact with the bot
         *   In addition, we fetch the configuration (with a '/init' call), to see if we have a bot
         *   configured.
         *   Indeed we want to trigger the bot script on every page where the associated rule is matched.
         *
         * - If we have a non-empty chat history, resume the chat script where the end-user left it by
         *   fetching the necessary information from the local storage.
         *
         * @override
         */
        async _willStartChatbot() {
            if (this.rule) {
                // noop
            } else if (this.history !== null && this.history.length === 0) {
                this.update({
                    livechatInit: await this.messaging.rpc({
                        route: '/im_livechat/init',
                        params: { channel_id: this.channelId },
                    }),
                });
            } else if (this.history !== null && this.history.length !== 0) {
                const sessionCookie = decodeURIComponent(getCookie('im_livechat_session'));
                if (sessionCookie) {
                    this.update({ sessionCookie });
                }
            }

            if (this.chatbot.state === 'init') {
                // we landed on a website page where a channel rule is configured to run a chatbot.script
                // -> initialize necessary state
                if (this.rule.chatbot_welcome_steps && this.rule.chatbot_welcome_steps.length !== 0) {
                    this.chatbot.update({
                        currentStep: {
                            data: this.chatbot.lastWelcomeStep,
                        },
                    });
                }
            } else if (this.chatbot.state === 'welcome') {
                // we landed on a website page and a chatbot script was initialized on a previous one
                // however the end-user did not interact with the bot ( :( )
                // -> remove cookie to force opening the popup again
                // -> initialize necessary state
                // -> batch welcome message (see '_sendWelcomeChatbotMessage')
                deleteCookie('im_livechat_auto_popup');
                this.update({ history: clear() });
                this.update({ rule: this.livechatInit.rule });
            } else if (this.chatbot.state === 'restore_session') {
                // we landed on a website page and a chatbot script is currently running
                // -> restore the user's session (see 'Chatbot/restoreSession')
                this.chatbot.restoreSession();
            }
        },

        getVisitorUserId() {
            const cookie = JSON.parse(decodeURIComponent(getCookie("im_livechat_session")) || "{}");
            if ("visitor_uid" in cookie) {
                return cookie.visitor_uid;
            }
            return session.user_id;
        },

        /**
         * Called when the visitor leaves the livechat chatter the first time (first click on X button)
         * this will deactivate the mail_channel, notify operator that visitor has left the channel.
         */
        leaveSession() {
            const cookie = decodeURIComponent(getCookie('im_livechat_session'));
            if (cookie) {
                const channel = JSON.parse(cookie);
                if (channel.uuid) {
                    this.messaging.rpc({ route: '/im_livechat/visitor_leave_session', params: { uuid: channel.uuid } });
                }
                deleteCookie('im_livechat_session');
            }
        },
    },
    fields: {
        HISTORY_LIMIT: attr({
            default: 15,
        }),
        LIVECHAT_COOKIE_HISTORY: attr({
            default: 'im_livechat_history',
        }),
        RATING_TO_EMOJI: attr({
            default: {
                5: "😊",
                3: "😐",
                1: "😞",
            },
        }),
        channelId: attr({
            compute() {
                return this.options.channel_id;
            },
        }),
        chatbot: one('Chatbot', {
            default: {},
            inverse: 'publicLivechatGlobalOwner',
        }),
        chatWindow: one('PublicLivechatWindow', {
            inverse: 'publicLivechatGlobalOwner',
        }),
        feedbackView: one('PublicLivechatFeedbackView', {
            inverse: 'publicLivechatGlobalOwner',
        }),
        hasLoadedQWebTemplate: attr({
            default: false,
        }),
        hasWebsiteLivechatFeature: attr({
            compute() {
                return false;
            },
        }),
        history: attr({
            default: null,
        }),
        isAvailable: attr({
            default: false,
        }),
        isAvailableForMe: attr({
            default: false,
        }),
        isLastMessageFromCustomer: attr({
            /**
             * Compares the last message of the conversation to this livechat's operator id.
             */
            compute() {
                if (!this.lastMessage) {
                    return clear();
                }
                if (!this.publicLivechat) {
                    return clear();
                }
                if (!this.publicLivechat.operator) {
                    return clear();
                }
                return this.lastMessage.authorId !== this.publicLivechat.operator.id;
            },
            default: false,
        }),
        isTestChatbot: attr({
            compute() {
                if (!this.options) {
                    return clear();
                }
                return Boolean(this.options.isTestChatbot);
            },
            default: false,
        }),
        lastMessage: one('PublicLivechatMessage', {
            compute() {
                if (this.messages.length === 0) {
                    return clear();
                }
                return this.messages[this.messages.length - 1];
            },
        }),
        livechatButtonView: one('LivechatButtonView', {
            compute() {
                if (this.isAvailable && (this.isAvailableForMe || this.isTestChatbot) && this.hasLoadedQWebTemplate && this.env.services.public_livechat_service) {
                    return {};
                }
                return clear();
            },
            inverse: 'publicLivechatGlobalOwner',
        }),
        livechatInit: attr(),
        messages: many('PublicLivechatMessage'),
        notificationHandler: one('PublicLivechatGlobalNotificationHandler', {
            inverse: 'publicLivechatGlobalOwner',
            compute() {
                if (this.publicLivechat && !this.publicLivechat.isTemporary) {
                    return {};
                }
                return clear();
            }
        }),
        options: attr({
            default: {},
        }),
        publicLivechat: one('PublicLivechat', {
            inverse: 'publicLivechatGlobalOwner',
        }),
        rule: attr(),
        serverUrl: attr({
            default: '',
        }),
        sessionCookie: attr(),
        testChatbotData: attr({
            compute() {
                if (!this.options) {
                    return clear();
                }
                return this.options.testChatbotData;
            },
        }),
        welcomeMessages: many('PublicLivechatMessage', {
            compute() {
                return this.messages.filter((message) => {
                    return message.id && typeof message.id === 'string' && message.id.startsWith('_welcome_');
                });
            },
        }),
    },
});

```

## File: static\src\public_models\public_livechat_global_notification_handler.js

```javascript
/** @odoo-module **/

import { registerModel } from '@mail/model/model_core';
import { one } from '@mail/model/model_field';
import { increment } from '@mail/model/model_field_command';

import session from 'web.session';
import utils from 'web.utils';
import {getCookie} from 'web.utils.cookies';

registerModel({
    name: 'PublicLivechatGlobalNotificationHandler',
    lifecycleHooks: {
        _created() {
            this.env.services['bus_service'].addChannel(this.messaging.publicLivechatGlobal.publicLivechat.uuid);
            this.env.services['bus_service'].addEventListener('notification', this._onNotification);
        },
    },
    recordMethods: {
        /**
         * @private
         * @param {Object} notification
         * @param {Object} notification.payload
         * @param {string} notification.type
         */
        _handleNotification({ payload, type }) {
            switch (type) {
                case 'im_livechat.history_command': {
                    if (payload.id !== this.messaging.publicLivechatGlobal.publicLivechat.id) {
                        return;
                    }
                    const cookie = getCookie(this.messaging.publicLivechatGlobal.LIVECHAT_COOKIE_HISTORY);
                    const history = cookie ? JSON.parse(cookie) : [];
                    session.rpc('/im_livechat/history', {
                        pid: this.messaging.publicLivechatGlobal.publicLivechat.operator.id,
                        channel_uuid: this.messaging.publicLivechatGlobal.publicLivechat.uuid,
                        page_history: history,
                    });
                    return;
                }
                case 'mail.channel.member/typing_status': {
                    if (!this.messaging.publicLivechatGlobal.chatWindow || !this.messaging.publicLivechatGlobal.chatWindow.exists()) {
                        return;
                    }
                    const channelMemberData = payload;
                    if (channelMemberData.channel.id !== this.messaging.publicLivechatGlobal.publicLivechat.id) {
                        return;
                    }
                    if (!channelMemberData.persona.partner) {
                        return;
                    }
                    if (channelMemberData.persona.partner.id === this.messaging.publicLivechatGlobal.livechatButtonView.currentPartnerId) {
                        // ignore typing display of current partner.
                        return;
                    }
                    if (channelMemberData.isTyping) {
                        this.messaging.publicLivechatGlobal.publicLivechat.widget.registerTyping({ partnerID: channelMemberData.persona.partner.id });
                    } else {
                        this.messaging.publicLivechatGlobal.publicLivechat.widget.unregisterTyping({ partnerID: channelMemberData.persona.partner.id });
                    }
                    return;
                }
                case 'mail.channel/new_message': {
                    if (!this.messaging.publicLivechatGlobal.chatWindow || !this.messaging.publicLivechatGlobal.chatWindow.exists()) {
                        return;
                    }
                    if (payload.id !== this.messaging.publicLivechatGlobal.publicLivechat.id) {
                        return;
                    }
                    const notificationData = payload.message;
                    // If message from notif is already in chatter messages, stop handling
                    if (this.messaging.publicLivechatGlobal.messages.some(message => message.id === notificationData.id)) {
                        return;
                    }
                    notificationData.body = utils.Markup(notificationData.body);
                    this.messaging.publicLivechatGlobal.livechatButtonView.addMessage(notificationData);
                    if (this.messaging.publicLivechatGlobal.publicLivechat.isFolded || !this.messaging.publicLivechatGlobal.chatWindow.publicLivechatView.widget.isAtBottom()) {
                        this.messaging.publicLivechatGlobal.publicLivechat.update({ unreadCounter: increment() });
                    }
                    this.messaging.publicLivechatGlobal.chatWindow.renderMessages();
                    return;
                }
                case 'mail.message/insert': {
                    if (!this.messaging.publicLivechatGlobal.chatWindow || !this.messaging.publicLivechatGlobal.chatWindow.exists()) {
                        return;
                    }
                    const message = this.messaging.publicLivechatGlobal.messages.find(message => message.id === payload.id);
                    if (!message) {
                        return;
                    }
                    message.widget._body = utils.Markup(payload.body);
                    this.messaging.publicLivechatGlobal.chatWindow.renderMessages();
                    return;
                }
            }
        },
        /**
         * @private
         * @param {CustomEvent} ev
         * @param {Array[]} [ev.detail] Notifications coming from the bus.
         */
        _onNotification({ detail: notifications }) {
            for (const notification of notifications) {
                this._handleNotification(notification);
            }
        },
    },
    fields: {
        publicLivechatGlobalOwner: one('PublicLivechatGlobal', {
            identifying: true,
            inverse: 'notificationHandler',
        }),
    },
});

```

## File: static\src\public_models\public_livechat_message.js

```javascript
/** @odoo-module **/

import { registerModel } from '@mail/model/model_core';
import { attr } from '@mail/model/model_field';
import { clear } from '@mail/model/model_field_command';

import PublicLivechatMessage from '@im_livechat/legacy/models/public_livechat_message';

registerModel({
    name: 'PublicLivechatMessage',
    lifecycleHooks: {
        _created() {
            this.update({ widget: new PublicLivechatMessage(this.messaging.publicLivechatGlobal.livechatButtonView.widget, this.messaging, this.data) });
        },
        _willDelete() {
            this.widget.destroy();
        },
    },
    fields: {
        authorId: attr({
            compute() {
                if (this.data.author && this.data.author.id) {
                    return this.data.author.id;
                }
                return clear();
            },
        }),
        data: attr(),
        id: attr({
            identifying: true,
        }),
        widget: attr(),
    },
});

```

## File: static\src\public_models\public_livechat_view.js

```javascript
/** @odoo-module **/

import PublicLivechatView from '@im_livechat/legacy/widgets/public_livechat_view/public_livechat_view';

import { registerModel } from '@mail/model/model_core';
import { attr, one } from '@mail/model/model_field';

registerModel({
    name: 'PublicLivechatView',
    lifecycleHooks: {
        _created() {
            this.update({
                widget: new PublicLivechatView(this, this.messaging, { displayMarkAsRead: false }),
            });
        },
        _willDelete() {
            this.widget.destroy();
        },
    },
    fields: {
        publicLivechatWindowOwner: one('PublicLivechatWindow', {
            identifying: true,
            inverse: 'publicLivechatView',
        }),
        widget: attr(),
    },
});

```

## File: static\src\public_models\public_livechat_window.js

```javascript
/** @odoo-module **/

import PublicLivechatWindow from '@im_livechat/legacy/widgets/public_livechat_window/public_livechat_window';

import { registerModel } from '@mail/model/model_core';
import { attr, one } from '@mail/model/model_field';

registerModel({
    name: 'PublicLivechatWindow',
    lifecycleHooks: {
        _created() {
            this.update({
                widget: new PublicLivechatWindow(
                    this.messaging.publicLivechatGlobal.livechatButtonView.widget,
                    this.messaging,
                    this.messaging.publicLivechatGlobal.publicLivechat.widget,
                ),
            });
        },
        _willDelete() {
            this.widget.destroy();
        },
    },
    recordMethods: {
        enableInput() {
            const $composerTextField = this.widget.$('.o_composer_text_field');
            $composerTextField
                .prop('disabled', false)
                .removeClass('text-center fst-italic bg-200')
                .val('')
                .focus();

            $composerTextField.off('keydown', this.messaging.publicLivechatGlobal.chatbot.onKeydownInput);
            if (this.messaging.publicLivechatGlobal.chatbot.currentStep.data.chatbot_step_type === 'free_input_multi') {
                $composerTextField.on('keydown', this.messaging.publicLivechatGlobal.chatbot.onKeydownInput);
            }
        },
        /**
         * Disable the input allowing the user to type.
         * This is typically used when we want to force him to click on one of the chatbot options.
         *
         * @private
         */
        disableInput(disableText) {
            this.widget.$('.o_composer_text_field')
                .prop('disabled', true)
                .addClass('text-center fst-italic bg-200')
                .val(disableText);
        },
        renderMessages() {
            const shouldScroll = !this.isFolded && this.publicLivechatView.widget.isAtBottom();
            this.widget.render();
            if (shouldScroll) {
                this.publicLivechatView.widget.scrollToBottom();
            }
            const self = this;

            this.widget.$('.o_thread_message:last .o_livechat_chatbot_options li').each(function () {
                $(this).on('click', self.messaging.publicLivechatGlobal.livechatButtonView.widget._onChatbotOptionClicked.bind(self.messaging.publicLivechatGlobal.livechatButtonView.widget));
            });

            this.widget.$('.o_livechat_chatbot_main_restart').on('click', (ev) => {
                ev.stopPropagation(); // prevent fold behaviour
                this.messaging.publicLivechatGlobal.livechatButtonView.onChatbotRestartScript(ev);
            });

            if (this.messaging.publicLivechatGlobal.messages.length !== 0) {
                const lastMessage = this.messaging.publicLivechatGlobal.lastMessage;
                const stepAnswers = lastMessage.widget.getChatbotStepAnswers();
                if (stepAnswers && stepAnswers.length !== 0 && !lastMessage.widget.getChatbotStepAnswerId()) {
                    this.disableInput(this.env._t("Select an option above"));
                }
            }
        },
    },
    fields: {
        inputPlaceholder: attr({
            compute() {
                if (this.messaging.publicLivechatGlobal.livechatButtonView.inputPlaceholder) {
                    return this.messaging.publicLivechatGlobal.livechatButtonView.inputPlaceholder;
                }
                return this.env._t("Say something");
            },
        }),
        publicLivechatGlobalOwner: one('PublicLivechatGlobal', {
            identifying: true,
            inverse: 'chatWindow',
        }),
        publicLivechatView: one('PublicLivechatView', {
            default: {},
            inverse: 'publicLivechatWindowOwner',
        }),
        widget: attr(),
    },
});

```

## File: static\src\services\public_livechat_service.js

```javascript
/** @odoo-module **/

import LivechatButton from '@im_livechat/legacy/widgets/livechat_button';

import rootWidget from 'root.widget';

import {getCookie, deleteCookie} from 'web.utils.cookies';

export const publicLivechatService = {
    dependencies: ['messaging'],
    async start(env, { messaging: messagingService }) {
        const messaging = await messagingService.get();
        try {
            JSON.parse(decodeURIComponent(getCookie('im_livechat_session')));
        } catch {
            // Cookies are not supposed to contain non-ASCII characters.
            // However, some were set in the past. Let's clean them up.
            deleteCookie('im_livechat_session');
        }
        return {
            mountLivechatButton() {
                const livechatButton = new LivechatButton(rootWidget, messaging);
                livechatButton.appendTo(document.body).catch(error => {
                    console.info("Can't load 'LivechatButton' because:", error);
                });
                return livechatButton;
            },
        };
    },
};

```

## File: views\chatbot_script_answer_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="chatbot_script_answer_view_form" model="ir.ui.view">
        <field name="name">chatbot.script.answer.view.form</field>
        <field name="model">chatbot.script.answer</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="script_step_id" invisible="1"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="chatbot_script_answer_view_tree" model="ir.ui.view">
        <field name="name">chatbot.script.answer.view.tree</field>
        <field name="model">chatbot.script.answer</field>
        <field name="arch" type="xml">
            <tree editable="bottom">
                <field name="sequence" widget="handle"/>
                <field name="script_step_id"/>
                <field name="name"/>
                <field name="redirect_link" string="Optional Link"/>
            </tree>
        </field>
    </record>

</data></odoo>

```

## File: views\chatbot_script_step_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="chatbot_script_step_view_form" model="ir.ui.view">
        <field name="name">chatbot.script.step.view.form</field>
        <field name="model">chatbot.script.step</field>
        <field name="arch" type="xml">
            <form disable_autofocus="1">
                <field name="is_forward_operator_child" invisible="1"/>
                <div class="alert alert-info text-center mb-0" role="alert" attrs="{'invisible': [('is_forward_operator_child', '=', False)]}">
                    <span>Reminder: This step will only be played if no operator is available.</span>
                </div>
                <div class="alert alert-info text-center mb-0" role="alert" attrs="{'invisible': [('step_type', '!=', 'forward_operator')]}">
                    <span>Tip: Plan further steps for the Bot in case no operator is available.</span>
                </div>
                <sheet>
                    <group>
                        <group>
                            <field name="sequence" invisible="1"/>
                            <field name="message" widget="text_emojis" placeholder="e.g. 'How can I help you?'"
                                attrs="{'required': [('step_type', '!=', 'forward_operator')]}"/>
                            <field name="chatbot_script_id" invisible="1"/>
                            <field name="step_type"/>
                            <field name="triggering_answer_ids" widget="chatbot_triggering_answers_widget"
                                    options="{'no_create': True}">
                                <tree>
                                    <!-- added only to correctly fetch the display_name for the tag display -->
                                    <field name="display_name" invisible="1"/>
                                </tree>
                            </field>
                        </group>
                        <group>
                            <field name="answer_ids" attrs="{'invisible': [('step_type', '!=', 'question_selection')]}" nolabel="1" colspan="2">
                                <tree editable="bottom">
                                    <field name="sequence" widget="handle"/>
                                    <field name="display_name" invisible="1"/>
                                    <field name="name"/>
                                    <field name="redirect_link" string="Optional Link"/>
                                </tree>
                            </field>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="chatbot_script_step_view_tree" model="ir.ui.view">
        <field name="name">chatbot.script.step.view.tree</field>
        <field name="model">chatbot.script.step</field>
        <field name="arch" type="xml">
            <tree default_order="sequence asc">
                <field name="sequence" widget="handle"/>
                <field name="message"/>
                <field name="step_type"/>
                <field name="answer_ids" widget="many2many_tags"/>
                <field name="triggering_answer_ids" widget="many2many_tags"/>
            </tree>
        </field>
    </record>

</data></odoo>

```

## File: views\chatbot_script_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="chatbot_script_view_form" model="ir.ui.view">
        <field name="name">chatbot.script.view.form</field>
        <field name="model">chatbot.script</field>
        <field name="arch" type="xml">
            <form>
                <header>
                </header>
                <field name="first_step_warning" invisible="1"/>
                <div class="alert alert-info text-center" role="alert"
                    attrs="{'invisible': [('first_step_warning', '!=', 'first_step_operator')]}">
                    <span>Tip: At least one interaction (Question, Email, ...) is needed before the Bot can perform more complex actions (Forward to an Operator, ...). </span>
                    <span>Use Channel Rules if you want the Bot to interact with visitors only when no operator is available.</span>
                </div>
                <div class="alert alert-info text-center" role="alert"
                    attrs="{'invisible': [('first_step_warning', '!=', 'first_step_invalid')]}">
                    <span>Tip: At least one interaction (Question, Email, ...) is needed before the Bot can perform more complex actions (Forward to an Operator, ...).</span>
                </div>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_view_livechat_channels" type="object" class="oe_stat_button"
                                icon="fa-comments" attrs="{'invisible': [('livechat_channel_count', '=', 0)]}">
                            <field name="livechat_channel_count" string="Channels" widget="statinfo"/>
                        </button>
                    </div>
                    <field name="active" invisible="1"/>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <field name="image_1920" widget="image" class="oe_avatar" options="{'preview_image': 'image_128'}"/>
                    <div class="oe_title">
                        <label for="title" string="Chatbot Name"/>
                        <h1><field name="title" default_focus="1" placeholder='e.g. "Meeting Scheduler Bot"'/></h1>
                    </div>
                    <notebook>
                        <page string="Script">
                            <field name="script_step_ids" widget="chatbot_steps_one2many" nolabel="1"
                                context="{'chatbot_script_answer_display_short_name': 1}"/>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="chatbot_script_view_tree" model="ir.ui.view">
        <field name="name">chatbot.script.view.tree</field>
        <field name="model">chatbot.script</field>
        <field name="arch" type="xml">
            <tree sample="1">
                <field name="title"/>
            </tree>
        </field>
    </record>

    <record id="chatbot_script_view_search" model="ir.ui.view">
        <field name="name">chatbot.script.view.search</field>
        <field name="model">chatbot.script</field>
        <field name="arch" type="xml">
            <search>
                <field name="title" string="Name" filter_domain="[('title', 'ilike', self)]"/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="chatbot_script_action" model="ir.actions.act_window">
        <field name="name">Chatbot</field>
        <field name="res_model">chatbot.script</field>
        <field name="view_mode">tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Chatbot
            </p><p>
                You can create a new Chatbot with a defined script to speak to your website visitors.
            </p>
        </field>
    </record>

</data></odoo>

```

## File: views\digest_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="im_livechat.digest_digest_view_form_inherit" model="ir.ui.view">
        <field name="name">im.livechat.digest.digest.view.form.inherit</field>
        <field name="model">digest.digest</field>
        <field name="priority">80</field>
        <field name="inherit_id" ref="digest.digest_digest_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//group[@name='kpis']/group[last()]" position="before">
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
                    odoo.define('im_livechat.loaderData', function() {
                        return {
                            isAvailable: <t t-out="'true' if info['available'] else 'false'"/>,
                            serverUrl: "<t t-out="info['server_url']"/>",
                            options: <t t-out="json.dumps(info.get('options', {}))"/>,
                        };
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
                                    <div class="float-end">
                                        <button t-if="record.are_you_inside.raw_value" name="action_quit" type="object" class="btn btn-primary">Leave</button>
                                        <button t-if="!record.are_you_inside.raw_value" name="action_join" type="object" class="btn btn-primary">Join</button>
                                    </div>
                                    <strong class="o_kanban_record_title" style="word-wrap: break-word;"><field name="name"/></strong>
                                    <div>
                                        <div>
                                            <i class="fa fa-user" role="img" aria-label="User" title="User"></i> <t t-esc="(record.user_ids.raw_value || []).length"/> Operators
                                            <br/>
                                            <i class="fa fa-comments" role="img" aria-label="Comments" title="Comments"></i> <t t-esc="record.nbr_channel.raw_value"/> Sessions
                                            <div t-if="record.rating_count.raw_value &gt; 0" class="float-end">
                                                <a name="action_view_rating" type="object" tabindex="10">
                                                    <i class="fa fa-smile-o text-success" title="Percentage of happy ratings" role="img" aria-label="Happy face"/> <t t-esc="record.rating_percentage_satisfaction.raw_value"/>%
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
                <form>
                    <header>
                        <button type="object" name="action_join" class="oe_highlight" string="Join Channel" attrs='{"invisible": [["are_you_inside", "=", True]]}'/>
                        <button type="object" name="action_quit" string="Leave Channel" attrs='{"invisible": [["are_you_inside", "=", False]]}'/>
                        <field name="are_you_inside" invisible="1"/>
                    </header>
                    <sheet>
                        <field name="rating_count" invisible="1"/>
                        <div class="oe_button_box" name="button_box">
                            <button class="oe_stat_button" type="object" name="action_view_chatbot_scripts" icon="fa-android"
                                attrs="{'invisible': [('chatbot_script_count', '=', 0)]}">
                                <field string="Chatbots" name="chatbot_script_count" widget="statinfo"/>
                            </button>
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
                                    <p class="text-muted" colspan="2">
                                        Operators that do not show any activity In Odoo for more than 30 minutes will be considered as disconnected.
                                    </p>
                            </page>
                            <page string="Options" name="options">
                                <group>
                                    <group string="Livechat Button">
                                        <field name="button_text"/>
                                        <label for="button_background_color" string="Livechat Button Color" />
                                        <div class="o_livechat_layout_colors d-flex align-items-center align-middle">
                                            <field name="button_background_color" widget="color" class="mb-4 w-auto o_im_livechat_field_widget_color"/>
                                            <field name="button_text_color" widget="color" class="mb-4 w-auto o_im_livechat_field_widget_color"/>
                                            <widget name="colors_reset_button" options="{'default_colors': {'button_background_color': '#878787', 'button_text_color': '#FFFFFF'}}" />
                                        </div>
                                    </group>
                                    <group string="Livechat Window">
                                        <field name="default_message" placeholder="e.g. Hello, how may I help you?"/>
                                        <field name="input_placeholder"/>
                                        <label for="header_background_color" string="Channel Header Color" />
                                        <div class="o_livechat_layout_colors d-flex align-items-center align-middle">
                                            <field name="header_background_color" widget="color" class="mb-4 w-auto o_im_livechat_field_widget_color"/>
                                            <field name="title_color" widget="color" class="mb-4 w-auto o_im_livechat_field_widget_color"/>
                                            <widget name="colors_reset_button" options="{'default_colors': {'header_background_color': '#875A7B', 'title_color': '#FFFFFF'}}" />
                                        </div>
                                    </group>
                                </group>
                            </page>
                            <page string="Channel Rules" name="channel_rules">
                                <field name="rule_ids" colspan="2"/>
                                <div class="text-muted" colspan="2">Define rules for your live support channel. You can apply an action for the given URL, and per country.<br />To identify the country, GeoIP must be installed on your server, otherwise, the countries of the rule will not be taken into account.</div>
                            </page>
                            <page string="Widget" name="configuration_widget">
                                <div class="alert alert-warning mt4 mb16" role="alert" attrs='{"invisible": [["web_page", "!=", False]]}'>
                                    Save your Channel to get your configuration widget.
                                </div>
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
                    <field name="country_ids" widget="many2many_tags"/>
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
                            <label for="chatbot_script_id" string="Chatbot" attrs="{'invisible': [('action', '=', 'hide_button')]}"/>
                            <div attrs="{'invisible': [('action', '=', 'hide_button')]}">
                                <field name="chatbot_script_id" class="oe_inline" style="width: 60% !important;"
                                       options="{'no_create': True, 'no_open': True}"/>
                            </div>
                            <label for="chatbot_only_if_no_operator" class="oe_inline" attrs="{'invisible': [('chatbot_script_id', '=', False)]}" string="Enabled only if no operator"/>
                            <div class="oe_inline" attrs="{'invisible': [('chatbot_script_id', '=', False)]}">
                                <field name="chatbot_only_if_no_operator"/>
                            </div>
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
            web_icon="im_livechat,static/description/icon.svg"
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

        <menuitem
            id="chatbot_config"
            name="Chatbots"
            parent="livechat_config"
            action="chatbot_script_action"
            groups="im_livechat_group_user"
            sequence="20"/>

    </data>
</odoo>

```

## File: views\im_livechat_chatbot_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="chatbot_test_script_page">
            <script>
                <t t-call="im_livechat.loader">
                    <t t-set="info" t-value="{ 'available': True, 'options': { 'isTestChatbot': True, 'testChatbotChannelData': channel_data, 'testChatbotData': chatbot_data }, 'server_url': server_url }"/>
                </t>
            </script>
            <t t-call="web.frontend_layout">
                <t t-set="no_livechat" t-value="True"/>
                <t t-set="title" t-value="chatbot_data['chatbot_name']"/>

                <div id="wrap">
                    <div groups="im_livechat.im_livechat_group_user" t-ignore="true"
                            class="alert alert-info alert-dismissible rounded-0 fade show d-print-none css_editable_mode_hidden mb-0">
                        <div t-ignore="true" class="text-center">
                            <a t-attf-href="/web#view_type=form&amp;model=chatbot.script&amp;id=#{chatbot_data['chatbot_script_id']}&amp;action=im_livechat.chatbot_script_action">
                                <span>You are currently testing</span>
                                <span t-out="chatbot_data['chatbot_name']"/>
                                <i class="fa fa-fw fa-arrow-right"/>Back to edit mode
                            </a>
                        </div>
                        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                    </div>
                </div>
            </t>
        </template>
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
                    <field name="rating_last_image" string="Rating" widget="image" options='{"size": [20, 20]}' class="bg-view"/>
                </tree>
            </field>
        </record>

        <record id="mail_channel_view_form" model="ir.ui.view">
            <field name="name">mail.channel.form</field>
            <field name="model">mail.channel</field>
            <field name="arch" type="xml">
                <form string="Session Form" create="false" edit="false">
                    <sheet>
                        <div style="width:50%" class="float-end">
                            <field name="rating_last_image" widget="image" class="float-end bg-view" readonly="1" nolabel="1"/>
                            <field name="rating_last_feedback" nolabel="1"/>
                        </div>
                        <div style="width:50%" class="float-start">
                            <group>
                                <field name="name" string="Attendees"/>
                                <field name="create_date" readonly="1" string="Session Date"/>
                            </group>
                        </div>

                        <group string="History" class="o_history_container">
                            <div class="o_history_kanban_container w-100 p-3" colspan="2">
                                <div class="o_history_kanban_sub_container">
                                    <field name="message_ids" mode="kanban">
                                        <kanban default_order="create_date DESC">
                                            <field name="author_id"/>
                                            <field name="body"/>
                                            <field name="create_date"/>
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
                                                            <div class="float-end"><p><field name="date"/></p></div>
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

## File: views\rating_rating_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="rating_rating_view_search_livechat" model="ir.ui.view">
        <field name="name">rating.rating.search.livechat</field>
        <field name="model">rating.rating</field>
        <field name="inherit_id" ref="rating.rating_rating_view_search"/>
        <field name="mode">primary</field>
        <field name="priority">64</field>
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
        <field name="view_id" ref="rating.rating_rating_view_form_text"/>
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
        <field name="view_id" ref="rating.rating_rating_view_form_text"/>
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
                    <field name="livechat_username" string="Online Chat Name" readonly="0"
                        attrs="{'invisible': [('share', '=', True)]}"/>
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
                        <group name="livechat" string="Livechat"
                            attrs="{'invisible': [('share', '=', True)]}">
                            <field name="livechat_username"/>
                        </group>
                    </xpath>
            </field>
        </record>

    </data>
</odoo>

```

