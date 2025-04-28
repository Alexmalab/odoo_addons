# Odoo Module: im_livechat

Category: Website/Live Chat

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
from . import controllers
from . import models
from . import report
from . import demo
from . import tools

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
        "data/mail_templates.xml",
        "data/im_livechat_channel_data.xml",
        "data/im_livechat_chatbot_data.xml",
        'data/digest_data.xml',
        'views/chatbot_script_answer_views.xml',
        'views/chatbot_script_step_views.xml',
        'views/chatbot_script_views.xml',
        "views/rating_rating_views.xml",
        "views/discuss_channel_views.xml",
        "views/im_livechat_channel_views.xml",
        "views/im_livechat_channel_templates.xml",
        "views/im_livechat_chatbot_templates.xml",
        "views/res_users_views.xml",
        "views/digest_views.xml",
        "views/webclient_templates.xml",
        "report/im_livechat_report_channel_views.xml",
        "report/im_livechat_report_operator_views.xml"
    ],
    'demo': [
        "demo/im_livechat_channel/im_livechat_channel.xml",
        "demo/im_livechat_channel/im_livechat_session_1.xml",
        "demo/im_livechat_channel/im_livechat_session_2.xml",
        "demo/im_livechat_channel/im_livechat_session_3.xml",
        "demo/im_livechat_channel/im_livechat_session_4.xml",
        "demo/im_livechat_channel/im_livechat_session_5.xml",
        "demo/im_livechat_channel/im_livechat_session_6.xml",
        "demo/im_livechat_channel/im_livechat_session_7.xml",
        "demo/im_livechat_channel/im_livechat_session_8.xml",
        "demo/im_livechat_channel/im_livechat_session_9.xml",
        "demo/im_livechat_channel/im_livechat_session_10.xml",
        "demo/im_livechat_channel/im_livechat_session_11.xml",
    ],
    'depends': ["mail", "rating", "digest", "utm"],
    'installable': True,
    'application': True,
    'assets': {
        'web.assets_frontend': [
            'web/static/src/views/fields/file_handler.*',
            'web/static/src/views/fields/formatters.js',
            ('include', 'im_livechat.assets_embed_core'),
            'im_livechat/static/src/embed/frontend/**/*',
        ],
        'web.assets_backend': [
            'im_livechat/static/src/js/colors_reset_button/*',
            'im_livechat/static/src/js/im_livechat_chatbot_steps_one2many.js',
            'im_livechat/static/src/js/im_livechat_chatbot_script_answers_m2m.js',
            'im_livechat/static/src/views/**/*',
            'im_livechat/static/src/scss/im_livechat_history.scss',
            'im_livechat/static/src/scss/im_livechat_form.scss',
            'im_livechat/static/src/core/common/**/*',
            'im_livechat/static/src/core/public_web/**/*',
            'im_livechat/static/src/core/web/**/*',
        ],
        'web.assets_unit_tests': [
            'im_livechat/static/tests/**/*',
            ('remove', 'im_livechat/static/tests/embed/**/*'),
            ('remove', 'im_livechat/static/tests/tours/**/*'),
        ],
        'im_livechat.qunit_embed_suite': [
            'im_livechat/static/tests/embed/**/*',
        ],
        'web.assets_tests': [
            'im_livechat/static/tests/tours/**/*',
        ],
        'im_livechat.assets_embed_core': [
            ('remove', 'web/static/src/core/browser/title_service.js'),
            'mail/static/src/model/**/*',
            'mail/static/src/core/common/**/*',
            'mail/static/src/discuss/core/common/*',
            'mail/static/src/discuss/call/common/**',
            'mail/static/src/discuss/typing/**/*',
            'mail/static/src/utils/common/**/*',
            ('remove', 'mail/static/src/**/*.dark.scss'),
            "rating/static/src/core/common/**/*",
            'im_livechat/static/src/core/common/**/*',
            'im_livechat/static/src/embed/common/**/*',
        ],
        'im_livechat.assets_embed_external': [
            'im_livechat/static/src/embed/common/scss/bootstrap_overridden.scss',
            ('include', 'web._assets_helpers'),
            ('include', 'web._assets_backend_helpers'),
            'web/static/src/scss/pre_variables.scss',
            'web/static/lib/bootstrap/scss/_variables.scss',
            'web/static/lib/bootstrap/scss/_variables-dark.scss',
            'web/static/lib/bootstrap/scss/_maps.scss',
            ('include', 'web._assets_bootstrap_backend'),
            'web/static/src/scss/bootstrap_overridden.scss',
            'web/static/src/scss/ui.scss',
            'web/static/src/libs/fontawesome/css/font-awesome.css',
            'web/static/lib/odoo_ui_icons/style.css',
            'web/static/src/webclient/webclient.scss',
            ('include', 'web._assets_core'),
            'web/static/src/libs/pdfjs.js',
            'web/static/src/views/fields/formatters.js',
            'web/static/src/views/fields/file_handler.*',
            'web/static/src/scss/mimetypes.scss',
            'bus/static/src/*.js',
            'bus/static/src/services/**/*.js',
            'bus/static/src/workers/websocket_worker.js',
            'bus/static/src/workers/websocket_worker_utils.js',
            ('remove', 'bus/static/src/outdated_page_watcher_service.js'),
            ('remove', 'bus/static/src/services/assets_watchdog_service.js'),
            ('remove', 'bus/static/src/simple_notification_service.js'),
            ('include', 'im_livechat.assets_embed_core'),
            'im_livechat/static/src/embed/external/**/*',
        ],
        'im_livechat.assets_embed_cors': [
            ('include', 'im_livechat.assets_embed_external'),
            'im_livechat/static/src/embed/cors/**/*',
        ],
        'im_livechat.embed_assets_unit_tests_setup': [
            ('include', 'web.assets_unit_tests_setup'),
            ('remove', 'im_livechat/static/**'),
            ('include', 'im_livechat.assets_embed_external'),
            ('remove', 'im_livechat/static/src/embed/external/boot.js'),
            ('remove', 'mail/static/src/discuss/core/web/discuss_core_common_service_patch.js'),
            'web/static/src/core/browser/title_service.js',
            'web/static/tests/web_test_helpers.js',
            'bus/static/tests/bus_test_helpers.js',
            'mail/static/tests/mail_test_helpers.js',
            'mail/static/tests/mail_test_helpers_contains.js',
            'im_livechat/static/tests/livechat_test_helpers.js',
            'bus/static/tests/mock_server/**/*',
            'mail/static/tests/mock_server/**/*',
            'rating/static/tests/mock_server/**/*',
            'im_livechat/static/tests/mock_server/**/*',
            'bus/static/tests/mock_websocket.js',
        ],
        'im_livechat.embed_assets_unit_tests': [
            'web/static/tests/_framework/**/*',
            'im_livechat/static/tests/embed/**/*',
        ],
        "mail.assets_public": [
            "im_livechat/static/src/core/common/**/*",
            "im_livechat/static/src/core/public_web/**/*",
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\attachment.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.exceptions import NotFound

from odoo import _
from odoo.http import route, request
from odoo.addons.mail.controllers.attachment import AttachmentController
from odoo.exceptions import AccessError
from odoo.addons.mail.models.discuss.mail_guest import add_guest_to_context


class LivechatAttachmentController(AttachmentController):
    @route()
    @add_guest_to_context
    def mail_attachment_upload(self, ufile, thread_id, thread_model, is_pending=False, **kwargs):
        thread = request.env[thread_model]._get_thread_with_access(
            int(thread_id), mode=request.env[thread_model]._mail_post_access, **kwargs
        )
        if not thread:
            raise NotFound()
        if (
            thread_model == "discuss.channel"
            and thread.channel_type == "livechat"
            and not thread.livechat_active
            and not request.env.user._is_internal()
        ):
            raise AccessError(_("You are not allowed to upload attachments on this channel."))
        return super().mail_attachment_upload(ufile, thread_id, thread_model, is_pending, **kwargs)

```

## File: controllers\chatbot.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request
from odoo.addons.mail.models.discuss.mail_guest import add_guest_to_context
from odoo.addons.mail.tools.discuss import Store


class LivechatChatbotScriptController(http.Controller):
    @http.route("/chatbot/restart", type="json", auth="public")
    @add_guest_to_context
    def chatbot_restart(self, channel_id, chatbot_script_id):
        discuss_channel = request.env["discuss.channel"].search([("id", "=", channel_id)])
        chatbot = request.env['chatbot.script'].browse(chatbot_script_id)
        if not discuss_channel or not chatbot.exists():
            return None
        chatbot_language = self._get_chatbot_language()
        message = discuss_channel.with_context(lang=chatbot_language)._chatbot_restart(chatbot)
        return Store(message, for_current_user=True).get_result()

    @http.route("/chatbot/answer/save", type="json", auth="public")
    @add_guest_to_context
    def chatbot_save_answer(self, channel_id, message_id, selected_answer_id):
        discuss_channel = request.env["discuss.channel"].search([("id", "=", channel_id)])
        chatbot_message = request.env['chatbot.message'].sudo().search([
            ('mail_message_id', '=', message_id),
            ('discuss_channel_id', '=', discuss_channel.id),
        ], limit=1)
        selected_answer = request.env['chatbot.script.answer'].sudo().browse(selected_answer_id)

        if not discuss_channel or not chatbot_message or not selected_answer.exists():
            return

        if selected_answer in chatbot_message.script_step_id.answer_ids:
            chatbot_message.write({'user_script_answer_id': selected_answer_id})

    @http.route("/chatbot/step/trigger", type="json", auth="public")
    @add_guest_to_context
    def chatbot_trigger_step(self, channel_id, chatbot_script_id=None):
        chatbot_language = self._get_chatbot_language()
        discuss_channel = request.env["discuss.channel"].with_context(lang=chatbot_language).search([("id", "=", channel_id)])
        if not discuss_channel:
            return None

        next_step = False
        # sudo: chatbot.script.step - visitor can access current step of the script
        if current_step := discuss_channel.sudo().chatbot_current_step_id:
            chatbot = current_step.chatbot_script_id
            user_messages = discuss_channel.message_ids.filtered(
                lambda message: message.author_id != chatbot.operator_partner_id
            )
            user_answer = request.env['mail.message'].sudo()
            if user_messages:
                user_answer = user_messages.sorted(lambda message: message.id)[-1]
            next_step = current_step._process_answer(discuss_channel, user_answer.body)
        elif chatbot_script_id:  # when restarting, we don't have a "current step" -> set "next" as first step of the script
            chatbot = request.env['chatbot.script'].sudo().browse(chatbot_script_id).with_context(lang=chatbot_language)
            if chatbot.exists():
                next_step = chatbot.script_step_ids[:1]

        if not next_step:
            # sudo: visitor cannot write on channel otherwise. Just writing a
            # boolean is safe
            discuss_channel.sudo().livechat_active = False
            return None

        posted_message = next_step._process_step(discuss_channel)
        store = Store(posted_message, for_current_user=True)
        store.add(next_step)
        store.add(
            "ChatbotStep",
            {
                "id": (next_step.id, posted_message.id),
                "isLast": next_step._is_last_step(discuss_channel),
                "message": Store.one(posted_message, only_id=True),
                "operatorFound": next_step.step_type == "forward_operator"
                and len(discuss_channel.channel_member_ids) > 2,
                "scriptStep": Store.one(next_step, only_id=True),
            },
        )
        store.add(
            "Chatbot",
            {
                "currentStep": {
                    "id": (next_step.id, discuss_channel.id),
                    "scriptStep": next_step.id,
                    "message": posted_message.id,
                },
                "id": (chatbot.id, discuss_channel.id),
                "script": Store.one(chatbot, only_id=True),
                "thread": Store.one(discuss_channel, only_id=True),
            },
        )
        return store.get_result()

    @http.route("/chatbot/step/validate_email", type="json", auth="public")
    @add_guest_to_context
    def chatbot_validate_email(self, channel_id):
        discuss_channel = request.env["discuss.channel"].search(
            [("id", "=", channel_id)]
        ).with_context(lang=self._get_chatbot_language())
        if not discuss_channel or not discuss_channel.chatbot_current_step_id:
            return None

        # sudo: chatbot.script - visitor can access chatbot script of their channel
        chatbot = discuss_channel.sudo().chatbot_current_step_id.chatbot_script_id
        user_messages = discuss_channel.message_ids.filtered(
            lambda message: message.author_id != chatbot.operator_partner_id
        )

        if user_messages:
            user_answer = user_messages.sorted(lambda message: message.id)[-1]
            result = chatbot._validate_email(user_answer.body, discuss_channel)

            if posted_message := result.pop("posted_message"):
                result["data"] = Store(posted_message, for_current_user=True).get_result()
        return result

    def _get_chatbot_language(self):
        return request.env["chatbot.script"]._get_chatbot_language()

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from markupsafe import Markup
import re
from werkzeug.exceptions import NotFound
from urllib.parse import urlsplit

from odoo import http, tools, _, release
from odoo.exceptions import UserError
from odoo.http import request
from odoo.tools import replace_exceptions
from odoo.addons.base.models.assetsbundle import AssetsBundle
from odoo.addons.mail.models.discuss.mail_guest import add_guest_to_context
from odoo.addons.mail.tools.discuss import Store


class LivechatController(http.Controller):

    # Note: the `cors` attribute on many routes is meant to allow the livechat
    # to be embedded in an external website.

    @http.route('/im_livechat/external_lib.<any(css,js):ext>', type='http', auth='public', cors='*')
    def external_lib(self, ext, **kwargs):
        """ Preserve compatibility with legacy livechat imports. Only
        serves javascript since the css will be fetched by the shadow
        DOM of the livechat to avoid conflicts.
        """
        if ext == 'css':
            raise request.not_found()
        return self.assets_embed(ext, **kwargs)

    @http.route('/im_livechat/assets_embed.<any(css, js):ext>', type='http', auth='public', cors='*')
    def assets_embed(self, ext, **kwargs):
        # If the request comes from a different origin, we must provide the CORS
        # assets to enable the redirection of routes to the CORS controller.
        headers = request.httprequest.headers
        origin_url = urlsplit(headers.get('referer'))
        bundle = 'im_livechat.assets_embed_external'
        if origin_url.netloc != headers.get('host') or origin_url.scheme != request.httprequest.scheme:
            bundle = 'im_livechat.assets_embed_cors'
        asset = request.env["ir.qweb"]._get_asset_bundle(bundle)
        if ext not in ('css', 'js'):
            raise request.not_found()
        stream = request.env['ir.binary']._get_stream_from(getattr(asset, ext)())
        return stream.get_response()

    @http.route('/im_livechat/font-awesome', type='http', auth='none', cors="*")
    def fontawesome(self, **kwargs):
        return http.Stream.from_path('web/static/src/libs/fontawesome/fonts/fontawesome-webfont.woff2').get_response()

    @http.route('/im_livechat/odoo_ui_icons', type='http', auth='none', cors="*")
    def odoo_ui_icons(self, **kwargs):
        return http.Stream.from_path('web/static/lib/odoo_ui_icons/fonts/odoo_ui_icons.woff2').get_response()

    @http.route('/im_livechat/emoji_bundle', type='http', auth='public', cors='*')
    def get_emoji_bundle(self):
        bundle = 'web.assets_emoji'
        asset = request.env["ir.qweb"]._get_asset_bundle(bundle)
        stream = request.env['ir.binary']._get_stream_from(asset.js())
        return stream.get_response()

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

    @http.route('/im_livechat/init', type='json', auth="public")
    @add_guest_to_context
    def livechat_init(self, channel_id):
        operator_available = len(request.env['im_livechat.channel'].sudo().browse(channel_id).available_operator_ids)
        rule = {}
        # find the country from the request
        country_id = False
        if request.geoip.country_code:
            country_id = request.env['res.country'].sudo().search([('code', '=', request.geoip.country_code)], limit=1).id
        # extract url
        url = request.httprequest.headers.get('Referer')
        # find the first matching rule for the given country and url
        if matching_rule := request.env['im_livechat.channel.rule'].sudo().match_rule(channel_id, url, country_id):
            matching_rule = matching_rule.with_context(lang=request.env['chatbot.script']._get_chatbot_language())
            rule = {
                "action": matching_rule.action,
                "auto_popup_timer": matching_rule.auto_popup_timer,
                "regex_url": matching_rule.regex_url,
                "chatbotScript": matching_rule.chatbot_script_id._format_for_frontend()
                if matching_rule.chatbot_script_id
                else None,
            }
        store = Store()
        request.env["res.users"]._init_store_data(store)
        return {
            'available_for_me': bool((rule and rule.get('chatbotScript'))
                                or operator_available and (not rule or rule['action'] != 'hide_button')),
            'rule': rule,
            'storeData': store.get_result(),
        }

    def _get_guest_name(self):
        return _("Visitor")

    @http.route('/im_livechat/get_session', methods=["POST"], type="json", auth='public')
    @add_guest_to_context
    def get_session(self, channel_id, anonymous_name, previous_operator_id=None, chatbot_script_id=None, persisted=True, **kwargs):
        store = Store()
        user_id = None
        country_id = None
        # if the user is identifiy (eg: portal user on the frontend), don't use the anonymous name. The user will be added to session.
        if request.session.uid:
            user_id = request.env.user.id
            country_id = request.env.user.country_id.id
        else:
            # if geoip, add the country name to the anonymous name
            if request.geoip.country_code:
                # get the country of the anonymous person, if any
                country = request.env['res.country'].sudo().search([('code', '=', request.geoip.country_code)], limit=1)
                if country:
                    country_id = country.id

        if previous_operator_id:
            previous_operator_id = int(previous_operator_id)

        chatbot_script = False
        if chatbot_script_id:
            chatbot_script = request.env['chatbot.script'].sudo().with_context(
                lang=request.env["chatbot.script"]._get_chatbot_language()
            ).browse(chatbot_script_id)
        channel_vals = request.env["im_livechat.channel"].with_context(lang=False).sudo().browse(channel_id)._get_livechat_discuss_channel_vals(
            anonymous_name,
            previous_operator_id=previous_operator_id,
            chatbot_script=chatbot_script,
            user_id=user_id,
            country_id=country_id,
            lang=request.cookies.get('frontend_lang')
        )
        if not channel_vals:
            return False
        if not persisted:
            channel_info = {
                "id": -1,  # only one temporary thread at a time, id does not matter.
                "isLoaded": True,
                "name": channel_vals["name"],
                "operator": Store.one(
                    request.env["res.partner"].sudo().browse(channel_vals["livechat_operator_id"]),
                    fields=["avatar_128", "user_livechat_username"],
                ),
                "scrollUnread": False,
                "state": "open",
                "livechat_active": True,
                "channel_type": "livechat",
                "chatbot": (
                    {
                        "script": chatbot_script._format_for_frontend(),
                        "steps": chatbot_script._get_welcome_steps().mapped(
                            lambda s: {"scriptStep": {"id": s.id}}
                        ),
                    }
                    if chatbot_script
                    else None
                ),
            }
            store.add("discuss.channel", channel_info)
        else:
            channel = request.env['discuss.channel'].with_context(
                mail_create_nosubscribe=False,
                lang=request.env['chatbot.script']._get_chatbot_language()
            ).sudo().create(channel_vals)
            if chatbot_script:
                chatbot_script._post_welcome_steps(channel)
            with replace_exceptions(UserError, by=NotFound()):
                # sudo: mail.guest - creating a guest and their member in a dedicated channel created from livechat
                __, guest = channel.sudo()._find_or_create_persona_for_channel(
                    guest_name=self._get_guest_name(),
                    country_code=request.geoip.country_code,
                    timezone=request.env['mail.guest']._get_timezone_from_request(request),
                    post_joined_message=False
                )
            channel = channel.with_context(guest=guest)  # a new guest was possibly created
            channel.channel_member_ids.filtered(lambda m: m.is_self).fold_state = "open"
            if not chatbot_script or chatbot_script.operator_partner_id != channel.livechat_operator_id:
                channel._broadcast([channel.livechat_operator_id.id])
            store.add(channel)
            store.add(channel, {"isLoaded": not chatbot_script, "scrollUnread": False})
            if guest:
                store.add({"guest_token": guest._format_auth_cookie()})
        request.env["res.users"]._init_store_data(store)
        return store.get_result()

    def _post_feedback_message(self, channel, rating, reason):
        reason = Markup("<br>" + re.sub(r'\r\n|\r|\n', "<br>", reason) if reason else "")
        body = Markup(
            """<div class="o_mail_notification o_hide_author">"""
            """%(rating)s: <img class="o_livechat_emoji_rating" src="%(rating_url)s" alt="rating"/>%(reason)s"""
            """</div>"""
        ) % {
            "rating": _("Rating"),
            "rating_url": rating.rating_image_url,
            "reason": reason,
        }
        # sudo: discuss.channel - not necessary for posting, but necessary to update related rating
        channel.sudo().message_post(
            body=body,
            message_type="notification",
            rating_id=rating.id,
            subtype_xmlid="mail.mt_comment",
        )

    @http.route("/im_livechat/feedback", type="json", auth="public")
    @add_guest_to_context
    def feedback(self, channel_id, rate, reason=None, **kwargs):
        if channel := request.env["discuss.channel"].search([("id", "=", channel_id)]):
            # limit the creation : only ONE rating per session
            values = {
                'rating': rate,
                'consumed': True,
                'feedback': reason,
                'is_internal': False,
            }
            # sudo: rating.rating - visitor can access rating to check if
            # feedback was already given
            if not channel.sudo().rating_ids:
                values.update({
                    'res_id': channel.id,
                    'res_model_id': request.env['ir.model']._get_id('discuss.channel'),
                })
                # sudo: res.partner - visitor must find the operator to rate
                if channel.sudo().channel_partner_ids:
                    values['rated_partner_id'] = channel.channel_partner_ids[0].id
                # if logged in user, set its partner on rating
                values['partner_id'] = request.env.user.partner_id.id if request.session.uid else False
                # create the rating
                rating = request.env['rating.rating'].sudo().create(values)
            else:
                rating = channel.rating_ids[0]
                # sudo: rating.rating - guest or portal user can update their livechat rating
                rating.sudo().write(values)
            self._post_feedback_message(channel, rating, reason)
            return rating.id
        return False

    @http.route("/im_livechat/history", type="json", auth="public")
    @add_guest_to_context
    def history_pages(self, pid, channel_id, page_history=None):
        if channel := request.env["discuss.channel"].search([("id", "=", channel_id)]):
            if pid in channel.sudo().channel_member_ids.partner_id.ids:
                request.env["res.partner"].browse(pid)._bus_send_history_message(channel, page_history)

    @http.route("/im_livechat/email_livechat_transcript", type="json", auth="public")
    @add_guest_to_context
    def email_livechat_transcript(self, channel_id, email):
        if channel := request.env["discuss.channel"].search([("id", "=", channel_id)]):
            channel._email_livechat_transcript(email)

    @http.route("/im_livechat/visitor_leave_session", type="json", auth="public")
    @add_guest_to_context
    def visitor_leave_session(self, channel_id):
        """Called when the livechat visitor leaves the conversation.
        This will clean the chat request and warn the operator that the conversation is over.
        This allows also to re-send a new chat request to the visitor, as while the visitor is
        in conversation with an operator, it's not possible to send the visitor a chat request."""
        if channel := request.env["discuss.channel"].search([("id", "=", channel_id)]):
            channel._close_livechat_session()

```

## File: controllers\thread.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request, route
from odoo.addons.mail.controllers import thread


class ThreadController(thread.ThreadController):
    @route()
    def mail_message_post(self, thread_model, thread_id, post_data, context=None, **kwargs):
        if selected_answer_id := kwargs.pop("selected_answer_id", None):
            request.update_context(selected_answer_id=selected_answer_id)
        return super().mail_message_post(thread_model, thread_id, post_data, context, **kwargs)

```

## File: controllers\webclient.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request, route
from odoo.addons.mail.controllers.webclient import WebclientController
from odoo.addons.mail.models.discuss.mail_guest import add_guest_to_context


class WebClient(WebclientController):
    @route("/web/tests/livechat", type="http", auth="user")
    def test_external_livechat(self, **kwargs):
        return request.render(
            "im_livechat.unit_embed_suite",
            {
                "server_url": request.env["ir.config_parameter"].get_base_url(),
                "session_info": {"view_info": request.env["ir.ui.view"].get_view_info()},
            },
        )

    def _process_request_for_internal_user(self, store, **kwargs):
        super()._process_request_for_internal_user(store, **kwargs)
        if kwargs.get("livechat_channels"):
            store.add(
                request.env["im_livechat.channel"].search([]), fields=["are_you_inside", "name"]
            )

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import attachment
from . import chatbot
from . import main
from . import thread
from . import webclient
from . import cors

```

## File: controllers\cors\attachment.py

```python
from odoo.http import route
from odoo.addons.mail.controllers.attachment import AttachmentController
from odoo.addons.im_livechat.tools.misc import force_guest_env


class LivechatAttachmentController(AttachmentController):
    @route("/im_livechat/cors/attachment/upload", auth="public", cors="*", csrf=False)
    def im_livechat_attachment_upload(self, guest_token, ufile, thread_id, thread_model, is_pending=False, **kwargs):
        force_guest_env(guest_token)
        return self.mail_attachment_upload(ufile, thread_id, thread_model, is_pending, **kwargs)

    @route("/im_livechat/cors/attachment/delete", methods=["POST"], type="json", auth="public", cors="*")
    def im_livechat_attachment_delete(self, guest_token, attachment_id, access_token=None):
        force_guest_env(guest_token)
        return self.mail_attachment_delete(attachment_id, access_token)

```

## File: controllers\cors\binary.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo.http import route
from odoo.addons.mail.controllers.discuss.binary import BinaryController
from odoo.addons.im_livechat.tools.misc import downgrade_to_public_user, force_guest_env


class LivechatBinaryController(BinaryController):
    @route(
        "/im_livechat/cors/channel/<int:channel_id>/attachment/<int:attachment_id>",
        methods=["GET"],
        type="http",
        auth="public",
        cors="*",
    )
    def livechat_channel_attachment(self, guest_token, channel_id, attachment_id, download=None, **kwargs):
        force_guest_env(guest_token)
        return self.discuss_channel_attachment(channel_id, attachment_id, download, **kwargs)

    @route(
        [
            "/im_livechat/cors/channel/<int:channel_id>/image/<int:attachment_id>",
            "/im_livechat/cors/channel/<int:channel_id>/image/<int:attachment_id>/<int:width>x<int:height>",
        ],
        methods=["GET"],
        type="http",
        auth="public",
        cors="*"
    )
    def livechat_fetch_image(self, guest_token, channel_id, attachment_id, width=0, height=0, **kwargs):
        force_guest_env(guest_token)
        return self.fetch_image(channel_id, attachment_id, width, height, **kwargs)

    @route(["/im_livechat/cors/web/image"], type='http', auth="public", cors="*")
    # pylint: disable=redefined-builtin,invalid-name
    def livechat_content_image(
        self, model, id, field, unique=False, guest_token=None, access_token=None
    ):
        if guest_token:
            force_guest_env(guest_token)
        else:
            downgrade_to_public_user()
        return self.content_image(
            model=model, id=id, field=field, unique=unique, access_token=access_token
        )

```

## File: controllers\cors\channel.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import route
from odoo.addons.mail.controllers.discuss.channel import ChannelController
from odoo.addons.im_livechat.tools.misc import force_guest_env


class LivechatChannelController(ChannelController):
    @route("/im_livechat/cors/channel/messages", methods=["POST"], type="json", auth="public", cors="*")
    def livechat_channel_messages(self, guest_token, channel_id, before=None, after=None, limit=30, around=None):
        force_guest_env(guest_token)
        return self.discuss_channel_messages(channel_id, before, after, limit, around)

    @route("/im_livechat/cors/channel/mark_as_read", methods=["POST"], type="json", auth="public", cors="*")
    def livechat_channel_mark_as_read(self, guest_token, **kwargs):
        force_guest_env(guest_token)
        return self.discuss_channel_mark_as_read(**kwargs)

    @route("/im_livechat/cors/channel/fold", methods=["POST"], type="json", auth="public", cors="*")
    def livechat_channel_fold(self, guest_token, channel_id, state, state_count):
        force_guest_env(guest_token)
        return self.discuss_channel_fold(channel_id, state, state_count)

    @route("/im_livechat/cors/channel/notify_typing", methods=["POST"], type="json", auth="public", cors="*")
    def livechat_channel_notify_typing(self, guest_token, channel_id, is_typing):
        force_guest_env(guest_token)
        return self.discuss_channel_notify_typing(channel_id, is_typing)

```

## File: controllers\cors\chatbot.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import route
from odoo.addons.im_livechat.controllers.chatbot import LivechatChatbotScriptController
from odoo.addons.im_livechat.tools.misc import force_guest_env


class CorsLivechatChatbotScriptController(LivechatChatbotScriptController):
    @route("/chatbot/cors/restart", type="json", auth="public", cors="*")
    def cors_chatbot_restart(self, guest_token, channel_id, chatbot_script_id):
        force_guest_env(guest_token)
        return self.chatbot_restart(channel_id, chatbot_script_id)

    @route("/chatbot/cors/answer/save", type="json", auth="public", cors="*")
    def cors_chatbot_save_answer(self, guest_token, channel_id, message_id, selected_answer_id):
        force_guest_env(guest_token)
        return self.chatbot_save_answer(channel_id, message_id, selected_answer_id)

    @route("/chatbot/cors/step/trigger", type="json", auth="public", cors="*")
    def cors_chatbot_trigger_step(self, guest_token, channel_id, chatbot_script_id=None):
        force_guest_env(guest_token)
        return self.chatbot_trigger_step(channel_id, chatbot_script_id)

    @route("/chatbot/cors/step/validate_email", type="json", auth="public", cors="*")
    def cors_chatbot_validate_email(self, guest_token, channel_id):
        force_guest_env(guest_token)
        return self.chatbot_validate_email(channel_id)

```

## File: controllers\cors\link_preview.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import route
from odoo.addons.mail.controllers.link_preview import LinkPreviewController
from odoo.addons.im_livechat.tools.misc import force_guest_env


class LivechatLinkPreviewController(LinkPreviewController):
    @route("/im_livechat/cors/link_preview", methods=["POST"], type="json", auth="public", cors="*")
    def livechat_link_preview(self, guest_token, message_id):
        force_guest_env(guest_token)
        self.mail_link_preview(message_id)

    @route("/im_livechat/cors/link_preview/hide", methods=["POST"], type="json", auth="public", cors="*")
    def livechat_link_preview_hide(self, guest_token, link_preview_ids):
        force_guest_env(guest_token)
        self.mail_link_preview_hide(link_preview_ids)

```

## File: controllers\cors\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import route
from odoo.addons.im_livechat.controllers.main import LivechatController
from odoo.addons.im_livechat.tools.misc import force_guest_env


class CorsLivechatController(LivechatController):
    @route("/im_livechat/cors/visitor_leave_session", type="json", auth="public", cors="*")
    def cors_visitor_leave_session(self, guest_token, channel_id):
        force_guest_env(guest_token)
        self.visitor_leave_session(channel_id)

    @route("/im_livechat/cors/feedback", type="json", auth="public", cors="*")
    def cors_feedback(self, guest_token, channel_id, rate, reason=None):
        force_guest_env(guest_token)
        self.feedback(channel_id, rate, reason)

    @route("/im_livechat/cors/history", type="json", auth="public", cors="*")
    def cors_history_pages(self, guest_token, pid, channel_id, page_history=None):
        force_guest_env(guest_token)
        return self.history_pages(pid, channel_id, page_history)

    @route("/im_livechat/cors/email_livechat_transcript", type="json", auth="public", cors="*")
    def cors_email_livechat_transcript(self, guest_token, channel_id, email):
        force_guest_env(guest_token)
        return self.email_livechat_transcript(channel_id, email)

    @route("/im_livechat/cors/get_session", methods=["POST"], type="json", auth="public", cors="*")
    def cors_get_session(
        self, channel_id, anonymous_name, previous_operator_id=None, chatbot_script_id=None, persisted=True, **kwargs
    ):
        force_guest_env(kwargs.get("guest_token", ""), raise_if_not_found=False)
        return self.get_session(
            channel_id, anonymous_name, previous_operator_id, chatbot_script_id, persisted, **kwargs
        )

    @route("/im_livechat/cors/init", type="json", auth="public", cors="*")
    def cors_livechat_init(self, channel_id, guest_token=""):
        force_guest_env(guest_token, raise_if_not_found=False)
        return self.livechat_init(channel_id)

```

## File: controllers\cors\message_reaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import route
from odoo.addons.mail.controllers.message_reaction import MessageReactionController
from odoo.addons.im_livechat.tools.misc import force_guest_env


class LivechatMessageReactionController(MessageReactionController):
    @route("/im_livechat/cors/message/reaction", methods=["POST"], type="json", auth="public", cors="*")
    def livechat_message_reaction(self, guest_token, message_id, content, action, **kwargs):
        force_guest_env(guest_token)
        return self.mail_message_reaction(message_id, content, action, **kwargs)

```

## File: controllers\cors\rtc.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import route
from odoo.addons.mail.controllers.discuss.rtc import RtcController
from odoo.addons.im_livechat.tools.misc import force_guest_env


class LivechatRtcController(RtcController):
    @route("/im_livechat/cors/rtc/channel/join_call", methods=["POST"], type="json", auth="public", cors="*")
    def livechat_channel_call_join(self, guest_token, channel_id, check_rtc_session_ids=None):
        force_guest_env(guest_token)
        return self.channel_call_join(channel_id, check_rtc_session_ids)

    @route("/im_livechat/cors/rtc/channel/leave_call", methods=["POST"], type="json", auth="public", cors="*")
    def livechat_channel_call_leave(self, guest_token, channel_id):
        force_guest_env(guest_token)
        return self.channel_call_leave(channel_id)

    @route("/im_livechat/cors/rtc/session/update_and_broadcast", methods=["POST"], type="json", auth="public", cors="*")
    def livechat_session_update_and_broadcast(self, guest_token, session_id, values):
        force_guest_env(guest_token)
        self.session_update_and_broadcast(session_id, values)

    @route("/im_livechat/cors/rtc/session/notify_call_members", methods=["POST"], type="json", auth="public", cors="*")
    def livechat_session_call_notify(self, guest_token, peer_notifications):
        force_guest_env(guest_token)
        self.session_call_notify(peer_notifications)

    @route("/im_livechat/cors/channel/ping", methods=["POST"], type="json", auth="public", cors="*")
    def livechat_channel_ping(self, guest_token, channel_id, rtc_session_id=None, check_rtc_session_ids=None):
        force_guest_env(guest_token)
        return self.channel_ping(channel_id, rtc_session_id, check_rtc_session_ids)

```

## File: controllers\cors\thread.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import route
from odoo.addons.mail.controllers.thread import ThreadController
from odoo.addons.im_livechat.tools.misc import force_guest_env


class LivechatThreadController(ThreadController):
    @route("/im_livechat/cors/message/post", methods=["POST"], type="json", auth="public", cors="*")
    def livechat_message_post(self, guest_token, thread_model, thread_id, post_data, context=None, **kwargs):
        force_guest_env(guest_token)
        return self.mail_message_post(thread_model, thread_id, post_data, context, **kwargs)

    @route("/im_livechat/cors/message/update_content", methods=["POST"], type="json", auth="public", cors="*")
    def livechat_message_update_content(
        self, guest_token, message_id, body, attachment_ids, attachment_tokens=None, partner_ids=None
    ):
        force_guest_env(guest_token)
        return self.mail_message_update_content(message_id, body, attachment_ids, attachment_tokens, partner_ids)

```

## File: controllers\cors\webclient.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import route
from odoo.addons.mail.controllers.webclient import WebclientController
from odoo.addons.im_livechat.tools.misc import force_guest_env


class WebClient(WebclientController):
    """Override to add CORS support."""

    @route("/im_livechat/cors/action", methods=["POST"], type="json", auth="public", cors="*")
    def livechat_action(self, guest_token, **kwargs):
        force_guest_env(guest_token)
        return self.mail_action(**kwargs)

    @route("/im_livechat/cors/data", methods=["POST"], type="json", auth="public", cors="*", readonly=True)
    def livechat_data(self, guest_token, **kwargs):
        force_guest_env(guest_token)
        return self.mail_data(**kwargs)

```

## File: controllers\cors\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import attachment
from . import binary
from . import chatbot
from . import channel
from . import link_preview
from . import main
from . import message_reaction
from . import rtc
from . import thread
from . import webclient

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
    <img src="https://download.odoocdn.com/digests/im_livechat/static/src/img/milk-canned-responses.gif" width="540" class="illustration_border" />
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
        <field name="message">Welcome to CompanyName! 👋</field>
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
                </td><td valign="middle" align="right" t-if="not company.uses_default_logo">
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
                <!-- sudo: res.users - visitor can access the name of the operator -->
                <span t-esc="channel.livechat_operator_id.sudo().user_livechat_username or channel.livechat_operator_id.sudo().name"/>, on the
                <span t-field="channel.create_date"/>
            </div>
            <table cellspacing="0" cellpadding="0" style="width:100%; border-collapse: collapse;">
                <t t-foreach="channel.message_ids.sorted(key=lambda m: m.date)" t-as="message" >
                    <!-- sudo: res.partner - visitor can access authors of the messages sent on their channel -->
                    <t t-set="author_name"><t t-if="message.author_id" t-esc="message.author_id.sudo().user_livechat_username or message.author_id.sudo().name"></t><t t-else="">You</t></t>
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
    _rec_name = 'discuss_channel_id'

    mail_message_id = fields.Many2one('mail.message', string='Related Mail Message', required=True, ondelete="cascade")
    discuss_channel_id = fields.Many2one('discuss.channel', string='Discussion Channel', required=True, ondelete="cascade")
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

from odoo import api, models, fields
from odoo.http import request
from odoo.tools import email_normalize, get_lang, html2plaintext, is_html_empty, plaintext2html
from odoo.exceptions import ValidationError


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

    @api.constrains("script_step_ids")
    def _check_question_selection(self):
        for step in self.script_step_ids:
            if step.step_type == "question_selection" and not step.answer_ids:
                raise ValidationError(self.env._("Step of type 'Question' must have answers."))

    def _compute_livechat_channel_count(self):
        channels_data = self.env['im_livechat.channel.rule']._read_group(
            [('chatbot_script_id', 'in', self.ids)], ['chatbot_script_id'], ['channel_id:count_distinct'])
        mapped_channels = {chatbot_script.id: count_distinct for chatbot_script, count_distinct in channels_data}
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

    def copy_data(self, default=None):
        vals_list = super().copy_data(default=default)
        return [dict(vals, title=self.env._("%s (copy)", script.title)) for script, vals in zip(self, vals_list)]

    def copy(self, default=None):
        """ Correctly copy the 'triggering_answer_ids' field from the original script_step_ids to the clone.
        This needs to be done in post-processing to make sure we get references to the newly created
        answers from the copy instead of references to the answers of the original.

        This implementation assumes that the order of created steps and answers will be kept between
        the original and the clone, using 'zip()' to match the records between the two. """
        default = default or {}
        new_scripts = super().copy(default=default)
        if 'question_ids' in default:
            return new_scripts

        for old_script, new_script in zip(self, new_scripts):
            original_steps = old_script.script_step_ids.sorted()
            clone_steps = new_script.script_step_ids.sorted()

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
        return new_scripts

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
        the frontend, since those are not inserted into the discuss.channel as actual mail.messages,
        to avoid bloating the channels with bot messages if the end-user never interacts with it. """
        self.ensure_one()

        welcome_steps = self.env['chatbot.script.step']
        for step in self.script_step_ids:
            welcome_steps += step
            if step.step_type != 'text':
                break

        return welcome_steps

    def _post_welcome_steps(self, discuss_channel):
        """ Welcome messages are only posted after the visitor's first interaction with the chatbot.
        See 'chatbot.script#_get_welcome_steps()' for more details.

        Side note: it is important to set the 'chatbot_current_step_id' on each iteration so that
        it's correctly set when going into 'discuss_channel#_message_post_after_hook()'. """

        self.ensure_one()
        posted_messages = self.env['mail.message']

        for welcome_step in self._get_welcome_steps():
            discuss_channel.chatbot_current_step_id = welcome_step.id

            if not is_html_empty(welcome_step.message):
                posted_messages += discuss_channel.with_context(mail_create_nosubscribe=True).message_post(
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
            'id': self.id,
            'name': self.title,
            'partner': {'id': self.operator_partner_id.id, 'type': 'partner', 'name': self.operator_partner_id.name},
            'welcomeSteps': [
                step._format_for_frontend()
                for step in self._get_welcome_steps()
            ]
        }

    def _validate_email(self, email_address, discuss_channel):
        email_address = html2plaintext(email_address)
        email_normalized = email_normalize(email_address)

        posted_message = False
        error_message = False
        if not email_normalized:
            error_message = self.env._(
                "'%(input_email)s' does not look like a valid email. Can you please try again?",
                input_email=email_address
            )
            posted_message = discuss_channel._chatbot_post_message(self, plaintext2html(error_message))

        return {
            'success': bool(email_normalized),
            'posted_message': posted_message,
            'error_message': error_message,
        }

    def _get_chatbot_language(self):
        return get_lang(
            self.env, lang_code=request and request.httprequest.cookies.get("frontend_lang")
        ).code

```

## File: models\chatbot_script_answer.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields
from odoo.osv import expression
from odoo.addons.mail.tools.discuss import Store

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

    @api.depends('script_step_id')
    @api.depends_context('chatbot_script_answer_display_short_name')
    def _compute_display_name(self):
        if self._context.get('chatbot_script_answer_display_short_name'):
            return super()._compute_display_name()

        for answer in self:
            if answer.script_step_id:
                answer_message = answer.script_step_id.message.replace('\n', ' ')
                shortened_message = textwrap.shorten(answer_message, width=26, placeholder=" [...]")
                answer.display_name = f"{shortened_message}: {answer.name}"
            else:
                answer.display_name = answer.name

    @api.model
    def _search_display_name(self, operator, value):
        """
        Search the records whose name or step message are matching the ``name`` pattern.
        The chatbot_script_id is also passed to the context through the custom widget
        ('chatbot_triggering_answers_widget') This allows to only see the question_answer
        from the same chatbot you're configuring.
        """
        domain = []
        if value and operator == 'ilike':
            # search on both name OR step's message (combined with passed args)
            domain = ['|', ('name', operator, value), ('script_step_id.message', operator, value)]

        force_domain_chatbot_script_id = self.env.context.get('force_domain_chatbot_script_id')
        if force_domain_chatbot_script_id:
            domain = expression.AND([domain, [('chatbot_script_id', '=', force_domain_chatbot_script_id)]])

        return domain

    def _to_store(self, store: Store, /, *, fields=None):
        if fields is None:
            fields = ["name", "redirect_link"]
        store.add("chatbot.script.answer", self._read_format(fields, load=False))

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
from odoo.addons.mail.tools.discuss import Store

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

        read_group_results = self.env['chatbot.script.step']._read_group(
            [('chatbot_script_id', 'in', list(vals_by_chatbot_id))],
            ['chatbot_script_id'],
            ['sequence:max'],
        )
        max_sequence_by_chatbot = {
            chatbot_script.id: sequence
            for chatbot_script, sequence in read_group_results
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

    def _chatbot_prepare_customer_values(self, discuss_channel, create_partner=True, update_partner=True):
        """ Common method that allows retreiving default customer values from the discuss.channel
        following a chatbot.script.

        This method will return a dict containing the 'customer' values such as:
        {
            'partner': The created partner (see 'create_partner') or the partner from the
              environment if not public
            'email': The email extracted from the discuss.channel messages
              (see step_type 'question_email')
            'phone': The phone extracted from the discuss.channel messages
              (see step_type 'question_phone')
            'description': A default description containing the "Please contact me on" and "Please
              call me on" with the related email and phone numbers.
              Can be used as a default description to create leads or tickets for example.
        }

        :param record discuss_channel: the discuss.channel holding the visitor's conversation with the bot.
        :param bool create_partner: whether or not to create a res.partner is the current user is public.
          Defaults to True.
        :param bool update_partner: whether or not to set update the email and phone on the res.partner
          from the environment (if not a public user) if those are not set yet. Defaults to True.

        :return dict: a dict containing the customer values."""

        partner = False
        user_inputs = discuss_channel._chatbot_find_customer_values_in_messages({
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

    def _is_last_step(self, discuss_channel=False):
        self.ensure_one()
        discuss_channel = discuss_channel or self.env['discuss.channel']

        # if it's not a question and if there is no next step, then we end the script
        # sudo: chatbot.script.answser - visitor can access their own answers
        if self.step_type != "question_selection" and not self._fetch_next_step(
            discuss_channel.sudo().chatbot_message_ids.user_script_answer_id
        ):
            return True

        return False

    def _process_answer(self, discuss_channel, message_body):
        """ Method called when the user reacts to the current chatbot.script step.
        For most chatbot.script.step#step_types it simply returns the next chatbot.script.step of
        the script (see '_fetch_next_step').

        Some extra processing is done for steps of type 'question_email' and 'question_phone' where
        we store the user raw answer (the mail message HTML body) into the chatbot.message in order
        to be able to recover it later (see '_chatbot_prepare_customer_values').

        :param discuss_channel:
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
                ('discuss_channel_id', '=', discuss_channel.id),
                ('script_step_id', '=', self.id),
            ], limit=1)

            if chatbot_message:
                chatbot_message.write({'user_raw_answer': message_body})
                self.env.flush_all()

        # sudo: chatbot.script.answer - visitor can access their own answer
        return self._fetch_next_step(discuss_channel.sudo().chatbot_message_ids.user_script_answer_id)

    def _process_step(self, discuss_channel):
        """ When we reach a chatbot.step in the script we need to do some processing on behalf of
        the bot. Which is for most chatbot.script.step#step_types just posting the message field.

        Some extra processing may be required for special step types such as 'forward_operator',
        'create_lead', 'create_ticket' (in their related bridge modules).
        Those will have a dedicated processing method with specific docstrings.

        Returns the mail.message posted by the chatbot's operator_partner_id. """

        self.ensure_one()
        # sudo: discuss.channel - updating current step on the channel is allowed
        discuss_channel.sudo().chatbot_current_step_id = self.id

        if self.step_type == 'forward_operator':
            return self._process_step_forward_operator(discuss_channel)
        return discuss_channel._chatbot_post_message(self.chatbot_script_id, plaintext2html(self.message))

    def _process_step_forward_operator(self, discuss_channel):
        """ Special type of step that will add a human operator to the conversation when reached,
        which stops the script and allow the visitor to discuss with a real person.

        In case we don't find any operator (e.g: no-one is available) we don't post any messages.
        The script will continue normally, which allows to add extra steps when it's the case
        (e.g: ask for the visitor's email and create a lead). """

        human_operator = False
        posted_message = self.env["mail.message"]

        if discuss_channel.livechat_channel_id:
            # sudo: res.users - visitor can access operator of their channel
            # sudo: res.lang - visitor can access their own lang
            human_operator = discuss_channel.livechat_channel_id.sudo()._get_operator(
                lang=self.env.context.get("lang"), country_id=discuss_channel.country_id.id
            )

        # handle edge case where we found yourself as available operator -> don't do anything
        # it will act as if no-one is available (which is fine)
        if human_operator and human_operator != self.env.user:
            if self.message:
                # first post the message of the step (if we have one)
                posted_message = discuss_channel._chatbot_post_message(self.chatbot_script_id, plaintext2html(self.message))

            # next, add the human_operator to the channel and post a "Operator joined the channel" notification
            discuss_channel.sudo()._add_members(
                users=human_operator,
                inviting_partner=self.chatbot_script_id.operator_partner_id,
                open_chat_window=True,
            )

            # finally, rename the channel to include the operator's name
            discuss_channel.sudo().name = ' '.join([
                self.env.user.display_name if not self.env.user._is_public() else discuss_channel.anonymous_name,
                human_operator.livechat_username if human_operator.livechat_username else human_operator.name
            ])
            step_message = next((
                # sudo - chatbot.message.id: visitor can access chat bot messages.
                m.mail_message_id for m in discuss_channel.sudo().chatbot_message_ids
                if m.script_step_id == self
                and m.mail_message_id.author_id == self.chatbot_script_id.operator_partner_id
            ), self.env["mail.message"])
            discuss_channel._bus_send_store(
                Store(
                    "ChatbotStep",
                    {
                        "id": (self.id, step_message.id),
                        "scriptStep": self.id,
                        "message": step_message.id,
                        "operatorFound": True,
                    },
                )
            )
            discuss_channel._broadcast(human_operator.partner_id.ids)
            discuss_channel.channel_pin(pinned=True)

        return posted_message

    def _to_store(self, store: Store, /, *, fields=None):
        if fields is None:
            fields = ["answer_ids", "message", "type"]
        for step in self:
            data = step._read_format(
                [f for f in fields if f not in {"answer_ids", "message", "type"}], load=False
            )[0]
            if "answer_ids" in fields:
                data["answers"] = Store.many(step.answer_ids)
            if "message" in fields:
                data["message"] = plaintext2html(step.message) if step.message else False
            if "type" in fields:
                data["type"] = step.step_type
            store.add("chatbot.script.step", data)

    # --------------------------
    # Tooling / Misc
    # --------------------------

    def _format_for_frontend(self):
        """ Small utility method that formats the step into a dict usable by the frontend code. """
        self.ensure_one()

        return {
            'id': self.id,
            'answers': [{
                'id': answer.id,
                "name": answer.name,
                "redirect_link": answer.redirect_link,
            } for answer in self.answer_ids],
            'message': plaintext2html(self.message) if not is_html_empty(self.message) else False,
            'isLast': self._is_last_step(),
            'type': self.step_type
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
        channels = self.env['discuss.channel'].search([('channel_type', '=', 'livechat')])
        start, end, __ = self._get_kpi_compute_parameters()
        domain = [
            ('create_date', '>=', start),
            ('create_date', '<', end),
        ]
        ratings = channels.rating_get_grades(domain)
        self.kpi_livechat_rating_value = (
            ratings['great'] * 100 / sum(ratings.values())
            if sum(ratings.values()) else 0
        )

    def _compute_kpi_livechat_conversations_value(self):
        start, end, __ = self._get_kpi_compute_parameters()
        self.kpi_livechat_conversations_value = self.env['discuss.channel'].search_count([
            ('channel_type', '=', 'livechat'),
            ('create_date', '>=', start), ('create_date', '<', end),
        ])

    def _compute_kpi_livechat_response_value(self):
        start, end, __ = self._get_kpi_compute_parameters()
        response_time = self.env['im_livechat.report.channel'].sudo()._read_group([
            ('start_date', '>=', start),
            ('start_date', '<', end),
        ], [], ['time_to_answer:avg'])
        self.kpi_livechat_response_value = response_time[0][0]

    def _compute_kpis_actions(self, company, user):
        res = super(Digest, self)._compute_kpis_actions(company, user)
        res['kpi_livechat_rating'] = 'im_livechat.rating_rating_action_livechat_report'
        res['kpi_livechat_conversations'] = 'im_livechat.im_livechat_report_operator_action'
        res['kpi_livechat_response'] = 'im_livechat.im_livechat_report_channel_time_to_answer_action'
        return res

```

## File: models\discuss_channel.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.addons.mail.tools.discuss import Store
from odoo.tools import email_normalize, html2plaintext, plaintext2html

from markupsafe import Markup


class DiscussChannel(models.Model):
    """ Chat Session
        Reprensenting a conversation between users.
        It extends the base method for anonymous usage.
    """

    _name = 'discuss.channel'
    _inherit = ['rating.mixin', 'discuss.channel']

    anonymous_name = fields.Char('Anonymous Name')
    channel_type = fields.Selection(selection_add=[('livechat', 'Livechat Conversation')], ondelete={'livechat': 'cascade'})
    duration = fields.Float('Duration', compute='_compute_duration', help='Duration of the session in hours')
    livechat_active = fields.Boolean('Is livechat ongoing?', help='Livechat session is active until visitor leaves the conversation.')
    livechat_channel_id = fields.Many2one('im_livechat.channel', 'Channel', index='btree_not_null')
    livechat_operator_id = fields.Many2one('res.partner', string='Operator', index='btree_not_null')
    chatbot_current_step_id = fields.Many2one('chatbot.script.step', string='Chatbot Current Step')
    chatbot_message_ids = fields.One2many('chatbot.message', 'discuss_channel_id', string='Chatbot Messages')
    country_id = fields.Many2one('res.country', string="Country", help="Country of the visitor of the channel")

    _sql_constraints = [('livechat_operator_id', "CHECK((channel_type = 'livechat' and livechat_operator_id is not null) or (channel_type != 'livechat'))",
                         'Livechat Operator ID is required for a channel of type livechat.')]

    @api.depends('message_ids')
    def _compute_duration(self):
        for record in self:
            start = record.message_ids[-1].date if record.message_ids else record.create_date
            end = record.message_ids[0].date if record.message_ids else fields.Datetime.now()
            record.duration = (end - start).total_seconds() / 3600

    def _to_store(self, store: Store):
        """Extends the channel header by adding the livechat operator and the 'anonymous' profile"""
        super()._to_store(store)
        chatbot_lang = self.env["chatbot.script"]._get_chatbot_language()
        for channel in self:
            channel_info = {}
            if channel.chatbot_current_step_id:
                # sudo: chatbot.script.step - returning the current script/step of the channel
                current_step_sudo = channel.chatbot_current_step_id.sudo().with_context(lang=chatbot_lang)
                chatbot_script = current_step_sudo.chatbot_script_id
                # sudo: channel - accessing chatbot messages to get the current step message
                step_message = next((
                    m.mail_message_id for m in channel.sudo().chatbot_message_ids
                    if m.script_step_id == current_step_sudo
                    and m.mail_message_id.author_id == chatbot_script.operator_partner_id
                ), None) if channel.chatbot_current_step_id.sudo().step_type != 'forward_operator' else None
                current_step = {
                    'scriptStep': current_step_sudo._format_for_frontend(),
                    "message": Store.one_id(step_message),
                    'operatorFound': current_step_sudo.step_type == 'forward_operator' and len(channel.channel_member_ids) > 2,
                }
                channel_info["chatbot"] = {
                    'script': chatbot_script._format_for_frontend(),
                    'steps': [current_step],
                    'currentStep': current_step,
                }
            channel_info['anonymous_name'] = channel.anonymous_name
            channel_info['anonymous_country'] = {
                'code': channel.country_id.code,
                'id': channel.country_id.id,
                'name': channel.country_id.name,
            } if channel.country_id else False
            if channel.channel_type == "livechat":
                channel_info["operator"] = Store.one(
                    channel.livechat_operator_id, fields=["avatar_128", "user_livechat_username"]
                )
            if channel.channel_type == "livechat":
                channel_info["livechat_active"] = channel.livechat_active
                if self.env.user._is_internal():
                    channel_info["livechatChannel"] = Store.one(
                        channel.livechat_channel_id, fields=["name"]
                    )
            store.add(channel, channel_info)

    @api.autovacuum
    def _gc_empty_livechat_sessions(self):
        hours = 1  # never remove empty session created within the last hour
        self.env.cr.execute("""
            SELECT id as id
            FROM discuss_channel C
            WHERE NOT EXISTS (
                SELECT 1
                FROM mail_message M
                WHERE M.res_id = C.id AND m.model = 'discuss.channel'
            ) AND C.channel_type = 'livechat' AND livechat_channel_id IS NOT NULL AND
                COALESCE(write_date, create_date, (now() at time zone 'UTC'))::timestamp
                < ((now() at time zone 'UTC') - interval %s)""", ("%s hours" % hours,))
        empty_channel_ids = [item['id'] for item in self.env.cr.dictfetchall()]
        self.browse(empty_channel_ids).unlink()

    def execute_command_history(self, **kwargs):
        self._bus_send("im_livechat.history_command", {"id": self.id})

    def _get_visitor_leave_message(self, operator=False, cancel=False):
        return _('Visitor left the conversation.')

    def _close_livechat_session(self, **kwargs):
        """ Set deactivate the livechat channel and notify (the operator) the reason of closing the session."""
        self.ensure_one()
        if self.livechat_active:
            member = self.channel_member_ids.filtered(lambda m: m.is_self)
            if member:
                member.fold_state = "closed"
                # sudo: discuss.channel.rtc.session - member of current user can leave call
                member.sudo()._rtc_leave_call()
            # sudo: discuss.channel - visitor left the conversation, state must be updated
            self.sudo().livechat_active = False
            # avoid useless notification if the channel is empty
            if not self.message_ids:
                return
            # Notify that the visitor has left the conversation
            # sudo: mail.message - posting visitor leave message is allowed
            self.sudo().message_post(
                author_id=self.env.ref('base.partner_root').id,
                body=Markup('<div class="o_mail_notification o_hide_author">%s</div>')
                % self._get_visitor_leave_message(**kwargs),
                message_type='notification',
                subtype_xmlid='mail.mt_comment'
            )

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
            # sudo: chatbot.script.step - getting the type of the current step
            lambda m: m.script_step_id.sudo().step_type in step_type_to_field
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
        # sudo: mail.message - chat bot is allowed to post a message which
        # requires reading its partner among other things.
        return self.with_context(mail_create_nosubscribe=True).sudo().message_post(
            author_id=chatbot_script.sudo().operator_partner_id.id,
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
        This method is called just before _notify_thread() method which is calling the _to_store()
        method. We need a 'chatbot.message' record before it happens to correctly display the message.
        It's created only if the mail channel is linked to a chatbot step. We also need to save the
        user answer if the current step is a question selection.
        """
        if self.chatbot_current_step_id:
            selected_answer = (
                self.env["chatbot.script.answer"]
                .browse(self.env.context.get("selected_answer_id"))
                .exists()
            )
            if selected_answer in self.chatbot_current_step_id.answer_ids:
                # sudo - chatbot.message: finding the question message to update the user answer is allowed.
                question_msg = (
                    self.env["chatbot.message"]
                    .sudo()
                    .search(
                        [
                            ("discuss_channel_id", "=", self.id),
                            ("script_step_id", "=", self.chatbot_current_step_id.id),
                        ],
                        order="id DESC",
                        limit=1,
                    )
                )
                question_msg.user_script_answer_id = selected_answer
                if store := self.env.context.get("message_post_store"):
                    store.add(message, for_current_user=True).add(question_msg.mail_message_id)

            self.env["chatbot.message"].sudo().create(
                {
                    "mail_message_id": message.id,
                    "discuss_channel_id": self.id,
                    "script_step_id": self.chatbot_current_step_id.id,
                }
            )

        return super()._message_post_after_hook(message, msg_vals)

    def _chatbot_restart(self, chatbot_script):
        # sudo: discuss.channel - visitor can clear current step to restart the script
        self.sudo().chatbot_current_step_id = False
        # sudo: discuss.channel - visitor can reactivate livechat
        self.sudo().livechat_active = True
        # sudo: chatbot.message - visitor can clear chatbot messages to restart the script
        self.sudo().chatbot_message_ids.unlink()
        return self._chatbot_post_message(
            chatbot_script,
            Markup('<div class="o_mail_notification">%s</div>') % _('Restarting conversation...'),
        )

    def _types_allowing_seen_infos(self):
        return super()._types_allowing_seen_infos() + ["livechat"]

```

## File: models\discuss_channel_member.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, timedelta

from odoo import api, models, fields
from odoo.osv import expression
from odoo.addons.mail.tools.discuss import Store


class ChannelMember(models.Model):
    _inherit = 'discuss.channel.member'

    @api.autovacuum
    def _gc_unpin_livechat_sessions(self):
        """ Unpin read livechat sessions with no activity for at least one day to
            clean the operator's interface """
        members = self.env['discuss.channel.member'].search([
            ('is_pinned', '=', True),
            ('last_seen_dt', '<=', datetime.now() - timedelta(days=1)),
            ('channel_id.channel_type', '=', 'livechat'),
        ])
        sessions_to_be_unpinned = members.filtered(lambda m: m.message_unread_counter == 0)
        sessions_to_be_unpinned.write({'unpin_dt': fields.Datetime.now()})
        for member in sessions_to_be_unpinned:
            member._bus_send("discuss.channel/unpin", {"id": member.channel_id.id})

    def _to_store(self, store: Store, **kwargs):
        super()._to_store(store, **kwargs)
        for member in self.filtered(lambda m: m.channel_id.channel_type == "livechat"):
            # sudo: discuss.channel - reading livechat channel to check whether current member is a bot is allowed
            store.add(
                member,
                {
                    "is_bot": member.partner_id
                    in member.channel_id.sudo().livechat_channel_id.rule_ids.chatbot_script_id.operator_partner_id,
                },
            )

    def _get_store_partner_fields(self, fields):
        self.ensure_one()
        if self.channel_id.channel_type == 'livechat':
            return ["active", "avatar_128", "country", "is_public", "user_livechat_username"]
        return super()._get_store_partner_fields(fields)

    def _get_rtc_invite_members_domain(self, *a, **kw):
        domain = super()._get_rtc_invite_members_domain(*a, **kw)
        chatbot = self.channel_id.chatbot_current_step_id.chatbot_script_id
        if self.channel_id.channel_type == "livechat" and chatbot:
            domain = expression.AND(
                [
                    domain,
                    [("partner_id", "!=", chatbot.operator_partner_id.id)],
                ]
            )
        return domain

```

## File: models\im_livechat_channel.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta
import random
import re

from odoo import api, Command, fields, models, _
from odoo.addons.mail.tools.discuss import Store
from odoo.addons.bus.websocket import WebsocketConnectionHandler


class ImLivechatChannel(models.Model):
    """ Livechat Channel
        Define a communication channel, which can be accessed with 'script_external' (script tag to put on
        external website), 'script_internal' (code to be integrated with odoo website) or via 'web_page' link.
        It provides rating tools, and access rules for anonymous people.
    """

    _name = 'im_livechat.channel'
    _inherit = ['rating.parent.mixin']
    _description = 'Livechat Channel'
    _rating_satisfaction_days = 14  # include only last 14 days to compute satisfaction

    def _default_user_ids(self):
        return [(6, 0, [self._uid])]

    def _default_button_text(self):
        return _('Have a Question? Chat with us.')

    def _default_default_message(self):
        return _('How may I help you?')

    # attribute fields
    name = fields.Char('Channel Name', required=True)
    button_text = fields.Char('Text of the Button', default=_default_button_text,
        help="Default text displayed on the Livechat Support Button", translate=True)
    default_message = fields.Char('Welcome Message', default=_default_default_message,
        help="This is an automated 'welcome' message that your visitor will see when they initiate a new conversation.", translate=True)
    input_placeholder = fields.Char('Chat Input Placeholder', help='Text that prompts the user to initiate the chat.', translate=True)
    header_background_color = fields.Char(default="#875A7B", help="Default background color of the channel header once open")
    title_color = fields.Char(default="#FFFFFF", help="Default title color of the channel once open")
    button_background_color = fields.Char(default="#875A7B", help="Default background color of the Livechat button")
    button_text_color = fields.Char(default="#FFFFFF", help="Default text color of the Livechat button")

    # computed fields
    web_page = fields.Char('Web Page', compute='_compute_web_page_link', store=False, readonly=True,
        help="URL to a static page where you client can discuss with the operator of the channel.")
    are_you_inside = fields.Boolean(string='Are you inside the matrix?',
        compute='_are_you_inside', store=False, readonly=True)
    available_operator_ids = fields.Many2many('res.users', compute='_compute_available_operator_ids')
    script_external = fields.Html('Script (external)', compute='_compute_script_external', store=False, readonly=True, sanitize=False)
    nbr_channel = fields.Integer('Number of conversation', compute='_compute_nbr_channel', store=False, readonly=True)

    image_128 = fields.Image("Image", max_width=128, max_height=128)

    # relationnal fields
    user_ids = fields.Many2many('res.users', 'im_livechat_channel_im_user', 'channel_id', 'user_id', string='Operators', default=_default_user_ids)
    channel_ids = fields.One2many('discuss.channel', 'livechat_channel_id', 'Sessions')
    chatbot_script_count = fields.Integer(string='Number of Chatbot', compute='_compute_chatbot_script_count')
    rule_ids = fields.One2many('im_livechat.channel.rule', 'channel_id', 'Rules')

    def _are_you_inside(self):
        for channel in self:
            channel.are_you_inside = self.env.user in channel.user_ids

    @api.depends('user_ids.im_status')
    def _compute_available_operator_ids(self):
        for record in self:
            record.available_operator_ids = record.user_ids.filtered(lambda user: user._is_user_available())

    @api.depends('rule_ids.chatbot_script_id')
    def _compute_chatbot_script_count(self):
        data = self.env['im_livechat.channel.rule']._read_group(
            [('channel_id', 'in', self.ids)], ['channel_id'], ['chatbot_script_id:count_distinct'])
        mapped_data = {channel.id: count_distinct for channel, count_distinct in data}
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
        data = self.env['discuss.channel']._read_group([
            ('livechat_channel_id', 'in', self.ids),
        ], ['livechat_channel_id'], ['__count'])
        channel_count = {livechat_channel.id: count for livechat_channel, count in data}
        for record in self:
            record.nbr_channel = channel_count.get(record.id, 0)

    # --------------------------
    # Action Methods
    # --------------------------
    def action_join(self):
        self.ensure_one()
        self.user_ids = [Command.link(self.env.user.id)]
        self.env.user._bus_send_store(self, fields=["are_you_inside", "name"])

    def action_quit(self):
        self.ensure_one()
        self.user_ids = [Command.unlink(self.env.user.id)]
        self.env.user._bus_send_store(self, fields=["are_you_inside", "name"])

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
    def _get_livechat_discuss_channel_vals(
        self, anonymous_name, previous_operator_id=None, chatbot_script=None, user_id=None, country_id=None, lang=None
    ):
        user_operator = False
        if chatbot_script:
            if chatbot_script.id not in self.browse(self.ids).mapped('rule_ids.chatbot_script_id.id'):
                return False
        else:
            user_operator = self._get_operator(previous_operator_id=previous_operator_id, lang=lang, country_id=country_id)
            if not user_operator:
                # no one available
                return False
        # partner to add to the discuss.channel
        operator_partner_id = user_operator.partner_id.id if user_operator else chatbot_script.operator_partner_id.id
        members_to_add = [
            Command.create({
                # making sure the unpin_dt is always later than the last_interest_dt
                # so that the channel is always unpinned at first
                'last_interest_dt': fields.Datetime.now() - timedelta(seconds=30),
                'partner_id': operator_partner_id,
                'unpin_dt': fields.Datetime.now(),
            })
        ]
        visitor_user = False
        if user_id:
            visitor_user = self.env['res.users'].browse(user_id)
            if visitor_user and visitor_user.active and user_operator and visitor_user != user_operator:  # valid session user (not public)
                members_to_add.append(Command.create({'partner_id': visitor_user.partner_id.id}))

        if chatbot_script:
            name = chatbot_script.title
        else:
            name = ' '.join([
                visitor_user.display_name if visitor_user else anonymous_name,
                user_operator.livechat_username or user_operator.name
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

    def _get_less_active_operator(self, operator_statuses, operators):
        """ Retrieve the most available operator based on the following criteria:
        - Lowest number of active chats.
        - Not in  a call.
        - If an operator is in a call and has two or more active chats, don't
          give priority over an operator with more conversations who is not in a
          call.

        :param operator_statuses: list of dictionaries containing the operator's
            id, the number of active chats and a boolean indicating if the
            operator is in a call. The list is ordered by the number of active
            chats (ascending) and whether the operator is in a call
            (descending).
        :param operators: recordset of :class:`ResUsers` operators to choose from.
        :return: the :class:`ResUsers` record for the chosen operator
        """
        if not operators:
            return False

        # 1) only consider operators in the list to choose from
        operator_statuses = [
            s for s in operator_statuses if s['livechat_operator_id'] in set(operators.partner_id.ids)
        ]

        # 2) try to select an inactive op, i.e. one w/ no active status (no recent chat)
        active_op_partner_ids = {s['livechat_operator_id'] for s in operator_statuses}
        candidates = operators.filtered(lambda o: o.partner_id.id not in active_op_partner_ids)
        if candidates:
            return random.choice(candidates)

        # 3) otherwise select least active ops, based on status ordering (count + in_call)
        best_status = operator_statuses[0]
        best_status_op_partner_ids = {
            s['livechat_operator_id']
            for s in operator_statuses
            if (s['count'], s['in_call']) == (best_status['count'], best_status['in_call'])
        }
        candidates = operators.filtered(lambda o: o.partner_id.id in best_status_op_partner_ids)
        return random.choice(candidates)

    def _get_operator(self, previous_operator_id=None, lang=None, country_id=None):
        """ Return an operator for a livechat. Try to return the previous
        operator if available. If not, one of the most available operators be
        returned.

        A livechat is considered 'active' if it has at least one message within
        the 30 minutes. This method will try to match the given lang and
        country_id.

        (Some annoying conversions have to be made on the fly because this model
        holds 'res.users' as available operators and the discuss_channel model
        stores the partner_id of the randomly selected operator)

        :param previous_operator_id: id of the previous operator with whom the
            visitor was chatting.
        :param lang: code of the preferred lang of the visitor.
        :param country_id: id of the country of the visitor.
        :return : user
        :rtype : res.users
        """
        if not self.available_operator_ids:
            return False
        # FIXME: remove inactive call sessions so operators no longer in call are available
        # sudo: required to use garbage collecting function.
        self.env["discuss.channel.rtc.session"].sudo()._gc_inactive_sessions()
        self.env.cr.execute("""
            WITH operator_rtc_session AS (
                SELECT COUNT(DISTINCT s.id) as nbr, member.partner_id as partner_id
                  FROM discuss_channel_rtc_session s
                  JOIN discuss_channel_member member ON (member.id = s.channel_member_id)
                  GROUP BY member.partner_id
            )
            SELECT COUNT(DISTINCT c.id), COALESCE(rtc.nbr, 0) > 0 as in_call, c.livechat_operator_id
            FROM discuss_channel c
            LEFT OUTER JOIN mail_message m ON c.id = m.res_id AND m.model = 'discuss.channel'
            LEFT OUTER JOIN operator_rtc_session rtc ON rtc.partner_id = c.livechat_operator_id
            WHERE c.channel_type = 'livechat' AND c.create_date > ((now() at time zone 'UTC') - interval '24 hours')
            AND (
                c.livechat_active IS TRUE
                OR m.create_date > ((now() at time zone 'UTC') - interval '30 minutes')
            )
            AND c.livechat_operator_id in %s
            GROUP BY c.livechat_operator_id, rtc.nbr
            ORDER BY COUNT(DISTINCT c.id) < 2 OR rtc.nbr IS NULL DESC, COUNT(DISTINCT c.id) ASC, rtc.nbr IS NULL DESC""",
            (tuple(self.available_operator_ids.partner_id.ids),)
        )
        operator_statuses = self.env.cr.dictfetchall()
        operator = None
        # Try to match the previous operator
        if previous_operator_id in self.available_operator_ids.partner_id.ids:
            previous_operator_status = next(
                (status for status in operator_statuses if status['livechat_operator_id'] == previous_operator_id),
                None
            )
            if not previous_operator_status or previous_operator_status['count'] < 2 or not previous_operator_status['in_call']:
                previous_operator_user = next(
                    available_user
                    for available_user in self.available_operator_ids
                    if available_user.partner_id.id == previous_operator_id
                )
                return previous_operator_user
        # Try to match an operator with the same main lang as the visitor
        # If no operator with the same lang, try to match an operator with the addition lang
        if lang:
            same_lang_operator_ids = self.available_operator_ids.filtered(lambda operator: operator.partner_id.lang == lang)
            if same_lang_operator_ids:
                operator = self._get_less_active_operator(operator_statuses, same_lang_operator_ids)
            else:
                addition_lang_operator_ids = self.available_operator_ids.filtered(lambda operator: lang in operator.res_users_settings_id.livechat_lang_ids.mapped('code'))
                if addition_lang_operator_ids:
                    operator = self._get_less_active_operator(operator_statuses, addition_lang_operator_ids)
        # Try to match an operator with the same country as the visitor
        if country_id and not operator:
            same_country_operator_ids = self.available_operator_ids.filtered(lambda operator: operator.partner_id.country_id.id == country_id)
            if same_country_operator_ids:
                operator = self._get_less_active_operator(operator_statuses, same_country_operator_ids)
        # Try to get a random operator, regardless of the lang or the country
        if not operator:
            operator = self._get_less_active_operator(operator_statuses, self.available_operator_ids)
        return operator

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
        info['available'] = self.chatbot_script_count or len(self.available_operator_ids) > 0
        info['server_url'] = self.get_base_url()
        info["websocket_worker_version"] = WebsocketConnectionHandler._VERSION
        if info['available']:
            info['options'] = self._get_channel_infos()
            info['options']["default_username"] = username
        return info

    def _to_store(self, store: Store, /, *, fields=None):
        if fields is None:
            fields = []
        store.add(self._name, self._read_format(fields))


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
                if not re.search(rule.regex_url or "", url or ""):
                    continue
                if rule.chatbot_script_id and (
                    not rule.chatbot_script_id.active or not rule.chatbot_script_id.script_step_ids
                ):
                    continue
                if rule.chatbot_only_if_no_operator and rule.channel_id.available_operator_ids:
                    continue
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

## File: models\mail_message.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.addons.mail.tools.discuss import Store


class MailMessage(models.Model):
    _inherit = 'mail.message'

    parent_author_name = fields.Char(compute="_compute_parent_author_name")
    parent_body = fields.Html(compute="_compute_parent_body")

    @api.depends('parent_id')
    def _compute_parent_author_name(self):
        for message in self:
            author = message.parent_id.author_id or message.parent_id.author_guest_id
            message.parent_author_name = author.name if author else False

    @api.depends('parent_id.body')
    def _compute_parent_body(self):
        for message in self:
            message.parent_body = message.parent_id.body if message.parent_id else False

    def _to_store(self, store: Store, **kwargs):
        """If we are currently running a chatbot.script, we include the information about
        the chatbot.message related to this mail.message.
        This allows the frontend display to include the additional features
        (e.g: Show additional buttons with the available answers for this step)."""
        super()._to_store(store, **kwargs)
        channel_messages = self.filtered(lambda message: message.model == "discuss.channel")
        channel_by_message = channel_messages._record_by_message()
        for message in channel_messages.filtered(
            lambda message: channel_by_message[message].channel_type == "livechat"
        ):
            channel = channel_by_message[message]
            # sudo: chatbot.script.step - checking whether the current message is from chatbot
            chatbot = channel.chatbot_current_step_id.sudo().chatbot_script_id.operator_partner_id
            if (channel.chatbot_current_step_id and message.author_id == chatbot):
                chatbot_message = (
                    self.env["chatbot.message"]
                    .sudo()
                    .search([("mail_message_id", "=", message.id)], limit=1)
                )
                if step := chatbot_message.script_step_id:
                    step_data = {
                        "id": (step.id, message.id),
                        "message": Store.one(message, only_id=True),
                        "scriptStep": Store.one(step, only_id=True),
                        "operatorFound": step.step_type == "forward_operator"
                        and len(channel.channel_member_ids) > 2,
                    }
                    if answer := chatbot_message.user_script_answer_id:
                        step_data["selectedAnswer"] = Store.one(answer, only_id=True)
                    store.add("ChatbotStep", step_data)
                    store.add(
                        message,
                        {"chatbotStep": {"scriptStep": step.id, "message": message.id}},
                    )

    def _author_to_store(self, store: Store):
        messages_w_author_channel = self.filtered(
            lambda message: message.author_id
            and message.model == "discuss.channel"
            and message.res_id
        )
        channel_by_message = messages_w_author_channel._record_by_message()
        messages_w_author_livechat = messages_w_author_channel.filtered(
            lambda message: channel_by_message[message].channel_type == "livechat"
        )
        super(MailMessage, self - messages_w_author_livechat)._author_to_store(store)
        for message in messages_w_author_livechat:
            store.add(
                message,
                {
                    "author": Store.one(
                        message.author_id,
                        fields=["avatar_128", "is_company", "user_livechat_username", "user"],
                    ),
                },
            )

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
            if rating.res_model == 'discuss.channel':
                current_object = self.env[rating.res_model].sudo().browse(rating.res_id)
                rating.res_name = ('%s / %s') % (current_object.livechat_channel_id.name, current_object.id)
            else:
                super(Rating, rating)._compute_res_name()

    def action_open_rated_object(self):
        action = super(Rating, self).action_open_rated_object()
        if self.res_model == 'discuss.channel':
            if self.env[self.res_model].browse(self.res_id):
                ctx = self.env.context.copy()
                ctx.update({'active_id': self.res_id})
                return {
                    'type': 'ir.actions.client',
                    'tag': 'mail.action_discuss',
                    'context': ctx,
                }
            view_id = self.env.ref('im_livechat.discuss_channel_view_form').id
            action['views'] = [[view_id, 'form']]
        return action

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from markupsafe import Markup

from odoo import api, models, fields, _
from odoo.addons.mail.tools.discuss import Store


class Partners(models.Model):
    """Update of res.partner class to take into account the livechat username."""
    _inherit = 'res.partner'

    user_livechat_username = fields.Char(compute='_compute_user_livechat_username')

    def _search_for_channel_invite_to_store(self, store: Store, channel):
        super()._search_for_channel_invite_to_store(store, channel)
        if channel.channel_type != "livechat" or not self:
            return
        lang_name_by_code = dict(self.env["res.lang"].get_installed())
        invite_by_self_count_by_partner_id = dict(
            self.env["discuss.channel.member"]._read_group(
                [["create_uid", "=", self.env.user.id], ["partner_id", "in", self.ids]],
                groupby=["partner_id"],
                aggregates=["__count"],
            )
        )
        active_livechat_partners = (
            self.env["im_livechat.channel"].search([]).available_operator_ids.partner_id
        )
        for partner in self:
            store.add(
                partner,
                {
                    "invite_by_self_count": invite_by_self_count_by_partner_id.get(partner, 0),
                    "is_available": partner in active_livechat_partners,
                    "lang_name": lang_name_by_code[partner.lang],
                },
            )

    @api.depends('user_ids.livechat_username')
    def _compute_user_livechat_username(self):
        for partner in self:
            partner.user_livechat_username = next(iter(partner.user_ids.mapped('livechat_username')), False)

    def _to_store(self, store: Store, /, *, fields=None, **kwargs):
        """Override to add name when user_livechat_username is not set."""
        super()._to_store(store, fields=fields, **kwargs)
        if fields and "user_livechat_username" in fields:
            if partners := self.filtered(lambda p: not p.user_livechat_username):
                super(Partners, partners)._to_store(store, fields=["name"])

    def _bus_send_history_message(self, channel, page_history):
        message_body = _("No history found")
        if page_history:
            message_body = Markup("<ul>%s</ul>") % (
                Markup("").join(
                    Markup('<li><a href="%(page)s" target="_blank">%(page)s</a></li>')
                    % {"page": page}
                    for page in page_history
                )
            )
        self._bus_send_transient_message(channel, message_body)

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

    livechat_username = fields.Char(string='Livechat Username', compute='_compute_livechat_username', inverse='_inverse_livechat_username', store=False)
    livechat_lang_ids = fields.Many2many('res.lang', string='Livechat Languages', compute='_compute_livechat_lang_ids', inverse='_inverse_livechat_lang_ids', store=False)
    has_access_livechat = fields.Boolean(compute='_compute_has_access_livechat', string='Has access to Livechat', store=False, readonly=True)

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + ['livechat_username', 'livechat_lang_ids', 'has_access_livechat']

    @property
    def SELF_WRITEABLE_FIELDS(self):
        return super().SELF_WRITEABLE_FIELDS + ['livechat_username', 'livechat_lang_ids']

    @api.depends('res_users_settings_id.livechat_username')
    def _compute_livechat_username(self):
        for user in self:
            user.livechat_username = user.res_users_settings_id.livechat_username

    def _inverse_livechat_username(self):
        for user in self:
            settings = self.env['res.users.settings']._find_or_create_for_user(user)
            settings.livechat_username = user.livechat_username

    @api.depends('res_users_settings_id.livechat_lang_ids')
    def _compute_livechat_lang_ids(self):
        for user in self:
            user.livechat_lang_ids = user.res_users_settings_id.livechat_lang_ids

    def _inverse_livechat_lang_ids(self):
        for user in self:
            settings = self.env['res.users.settings']._find_or_create_for_user(user)
            settings.livechat_lang_ids = user.livechat_lang_ids

    def _compute_has_access_livechat(self):
        for user in self.sudo():
            user.has_access_livechat = user.has_group('im_livechat.im_livechat_group_user')

    def _init_store_data(self, store):
        super()._init_store_data(store)
        store.add({"has_access_livechat": self.env.user.has_access_livechat})

```

## File: models\res_users_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResUsersSettings(models.Model):
    _inherit = 'res.users.settings'

    livechat_username = fields.Char("Livechat Username", help="This username will be used as your name in the livechat channels.")
    livechat_lang_ids = fields.Many2many(comodel_name='res.lang', string='Livechat languages',
                            help="These languages, in addition to your main language, will be used to assign you to Live Chat sessions.")

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
from . import discuss_channel
from . import discuss_channel_member
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
    channel_id = fields.Many2one('discuss.channel', 'Conversation', readonly=True)
    channel_name = fields.Char('Channel Name', readonly=True)
    technical_name = fields.Char('Code', readonly=True)
    livechat_channel_id = fields.Many2one('im_livechat.channel', 'Channel', readonly=True)
    start_date = fields.Datetime('Start Date of session', readonly=True)
    start_hour = fields.Char('Start Hour of session', readonly=True)
    day_number = fields.Char('Day Number', readonly=True, help="1 is Monday, 7 is Sunday")
    time_to_answer = fields.Float('Time to answer (sec)', digits=(16, 2), readonly=True, aggregator="avg", help="Average time in seconds to give the first answer to the visitor")
    start_date_hour = fields.Char('Hour of start Date of session', readonly=True)
    duration = fields.Float('Average duration', digits=(16, 2), readonly=True, aggregator="avg", help="Duration of the conversation (in seconds)")
    nbr_speaker = fields.Integer('# of speakers', readonly=True, aggregator="avg", help="Number of different speakers")
    nbr_message = fields.Integer('Average message', readonly=True, aggregator="avg", help="Number of message in the conversation")
    is_without_answer = fields.Integer('Session(s) without answer', readonly=True, aggregator="sum",
                                       help="""A session is without answer if the operator did not answer. 
                                       If the visitor is also the operator, the session will always be answered.""")
    days_of_activity = fields.Integer('Days of activity', aggregator="max", readonly=True, help="Number of days since the first session of the operator")
    is_anonymous = fields.Integer('Is visitor anonymous', readonly=True)
    country_id = fields.Many2one('res.country', 'Country of the visitor', readonly=True)
    is_happy = fields.Integer('Visitor is Happy', readonly=True)
    rating = fields.Integer('Rating', aggregator="avg", readonly=True)
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
                                        AND M.model = 'discuss.channel'
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
                FROM discuss_channel C
                    JOIN mail_message M ON (M.res_id = C.id AND M.model = 'discuss.channel')
                    JOIN im_livechat_channel L ON (L.id = C.livechat_channel_id)
                    LEFT JOIN mail_message MO ON (MO.res_id = C.id AND MO.model = 'discuss.channel' AND MO.author_id = C.livechat_operator_id)
                    LEFT JOIN rating_rating Rate ON (Rate.res_id = C.id and Rate.res_model = 'discuss.channel' and Rate.parent_res_model = 'im_livechat.channel')
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
                    <field name="duration" type="measure" string="Duration of Session (min)"/>
                    <field name="nbr_message" type="measure" string="Messages per session"/>
                    <field name="is_without_answer" invisible="True"/>
                </pivot>
            </field>
        </record>

        <record id="im_livechat_report_channel_view_graph" model="ir.ui.view">
            <field name="name">im_livechat.report.channel.graph</field>
            <field name="model">im_livechat.report.channel</field>
            <field name="arch" type="xml">
                <graph string="Livechat Support Statistics" sample="1" disable_linking="1">
                    <field name="technical_name"/>
                    <field name="nbr_message" type="measure" string="Messages per session"/>
                    <field name="duration" string="Duration of Session (min)"/>
                    <field name="is_without_answer" invisible="True"/>
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
            <field name="context">{"search_default_last_week":1, "group_by": "start_date:day"}</field>
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
    nbr_channel = fields.Integer('# of Sessions', readonly=True, aggregator="sum")
    channel_id = fields.Many2one('discuss.channel', 'Conversation', readonly=True)
    start_date = fields.Datetime('Start Date of session', readonly=True)
    time_to_answer = fields.Float('Time to answer', digits=(16, 2), readonly=True, aggregator="avg", help="Average time to give the first answer to the visitor")
    duration = fields.Float('Average duration', digits=(16, 2), readonly=True, aggregator="avg", help="Duration of the conversation (in seconds)")
    rating = fields.Float('Average rating', readonly=True, aggregator="avg", help="Average rating given by the visitor")

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
                    C.rating_last_value as rating,
                    EXTRACT('epoch' FROM MAX(M.create_date) - MIN(M.create_date)) AS duration,
                    EXTRACT('epoch' FROM MIN(MO.create_date) - MIN(M.create_date)) AS time_to_answer
                FROM discuss_channel C
                    JOIN mail_message M ON M.res_id = C.id AND M.model = 'discuss.channel'
                    LEFT JOIN mail_message MO ON (MO.res_id = C.id AND MO.model = 'discuss.channel' AND MO.author_id = C.livechat_operator_id)
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
                    <filter name="last_month" string="This Month" invisible="True" domain="[
                        ('start_date', '>=', (datetime.datetime.combine(context_today() + relativedelta(months=-1,days=1,weekday=0), datetime.time(0,0,0)).to_utc()).strftime('%Y-%m-%d %H:%M:%S')),
                        ('start_date', '&lt;', (datetime.datetime.combine(context_today() + relativedelta(days=1,weekday=0), datetime.time(0,0,0)).to_utc()).strftime('%Y-%m-%d %H:%M:%S'))]"/>
                    <separator/>
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
            <field name="context">{"search_default_last_month":1}</field>
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
            <field name="implied_ids" eval="[(4, ref('im_livechat.im_livechat_group_user')), (4, ref('mail.group_mail_canned_response_admin'))]"/>
            <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        </record>

    <data noupdate="1">
        <record id="ir_rule_discuss_channel_group_im_livechat_group_manager" model="ir.rule">
            <field name="name">discuss.channel: livechat manager can read all livechat channels</field>
            <field name="model_id" ref="mail.model_discuss_channel"/>
            <field name="groups" eval="[(4, ref('im_livechat_group_manager'))]"/>
            <field name="domain_force">[('channel_type', '=', 'livechat')]</field>
            <field name="perm_create" eval="False"/>
            <field name="perm_write" eval="False"/>
            <field name="perm_unlink" eval="False"/>
        </record>

        <record id="ir_rule_discuss_channel_member_group_im_livechat_group_manager" model="ir.rule">
            <field name="name">discuss.channel.member: livechat manager can read all livechat channel members and can invite anyone</field>
            <field name="model_id" ref="mail.model_discuss_channel_member"/>
            <field name="groups" eval="[(4, ref('im_livechat_group_manager'))]"/>
            <field name="domain_force">[('channel_id.channel_type', '=', 'livechat')]</field>
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
access_livechat_channel_public,im_livechat.channel,model_im_livechat_channel,base.group_public,1,0,0,0
access_livechat_channel_portal,im_livechat.channel,model_im_livechat_channel,base.group_portal,1,0,0,0
access_livechat_channel_employee,im_livechat.channel,model_im_livechat_channel,base.group_user,1,0,0,0
access_livechat_channel_user,im_livechat.channel.user,model_im_livechat_channel,im_livechat_group_user,1,1,1,0
access_livechat_channel_manager,im_livechat.channel.manager,model_im_livechat_channel,im_livechat_group_manager,1,1,1,1
access_livechat_support_report_channel,im_livechat.report.channel,model_im_livechat_report_channel,im_livechat_group_manager,1,0,0,0
access_livechat_support_report_operator,im_livechat.report.operator,model_im_livechat_report_operator,im_livechat_group_manager,1,0,0,0
access_livechat_channel_rule_public,im_livechat.channel.rule,model_im_livechat_channel_rule,base.group_public,1,0,0,0
access_livechat_channel_rule_portal,im_livechat.channel.rule,model_im_livechat_channel_rule,base.group_portal,1,0,0,0
access_livechat_channel_rule_employee,im_livechat.channel.rule,model_im_livechat_channel_rule,base.group_user,1,0,0,0
access_livechat_channel_rule_user,im_livechat.channel.rule,model_im_livechat_channel_rule,im_livechat_group_user,1,1,1,0
access_livechat_channel_rule_manager,im_livechat.channel.rule,model_im_livechat_channel_rule,im_livechat_group_manager,1,1,1,1
access_chatbot_script_user,chatbot.script.user,model_chatbot_script,im_livechat_group_user,1,1,1,1
access_chatbot_script_step_user,chatbot.script.step.user,model_chatbot_script_step,im_livechat_group_user,1,1,1,1
access_chatbot_script_answer,chatbot.script.answer,model_chatbot_script_answer,,0,0,0,0
access_chatbot_script_answer_user,chatbot.script.answer.user,model_chatbot_script_answer,im_livechat_group_user,1,1,1,1
access_chatbot_message_user,chatbot.script.user,model_chatbot_message,im_livechat_group_user,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M4 26.222C4 27.204 4.796 28 5.778 28H16c6.627 0 12-5.373 12-12S22.627 4 16 4 4 9.373 4 16v10.222Z" fill="#FC868B"/><path d="M46 5.778C46 4.796 45.204 4 44.222 4H34c-6.627 0-12 5.373-12 12s5.373 12 12 12 12-5.373 12-12V5.778Z" fill="#985184"/><path d="M25 23.937c1.867-2.115 3-4.894 3-7.937s-1.133-5.822-3-7.938A11.954 11.954 0 0 0 22 16c0 3.043 1.133 5.822 3 7.937Z" fill="#962B48"/><path d="M4 38.5C4 32.701 8.701 28 14.5 28S25 32.701 25 38.5V46H4v-7.5Z" fill="#FC868B"/><path d="M25 38.5C25 32.701 29.701 28 35.5 28S46 32.701 46 38.5V46H25v-7.5Z" fill="#985184"/></svg>

```

## File: static\src\core\common\channel_member_model_patch.js

```javascript
import { ChannelMember } from "@mail/core/common/channel_member_model";
import { patch } from "@web/core/utils/patch";

patch(ChannelMember.prototype, {
    get name() {
        if (this.thread.channel_type === "livechat" && this.persona.user_livechat_username) {
            return this.persona.user_livechat_username;
        }
        if (this.persona.is_public) {
            return this.thread.anonymous_name;
        }
        return super.name;
    },
});

```

## File: static\src\core\common\livechat_channel_model.js

```javascript
import { Record } from "@mail/core/common/record";

export class LivechatChannel extends Record {
    static _name = "im_livechat.channel";
    static id = "id";

    /** @type {boolean} */
    are_you_inside;
    /** @type {number} */
    id;
    /** @type {string} */
    name;
}
LivechatChannel.register();

```

## File: static\src\core\common\message_patch.js

```javascript
import { Message } from "@mail/core/common/message";

import { patch } from "@web/core/utils/patch";

patch(Message.prototype, {
    get authorName() {
        if (
            this.message.author?.user_livechat_username &&
            this.message.thread?.channel_type === "livechat"
        ) {
            return this.message.author.user_livechat_username;
        }
        return super.authorName;
    },
});

```

## File: static\src\core\common\thread_model_patch.js

```javascript
import { Record } from "@mail/core/common/record";
import { Thread } from "@mail/core/common/thread_model";

import { patch } from "@web/core/utils/patch";

patch(Thread.prototype, {
    setup() {
        super.setup();
        this.operator = Record.one("Persona");
    },
    get autoOpenChatWindowOnNewMessage() {
        return this.channel_type === "livechat" || super.autoOpenChatWindowOnNewMessage;
    },
    get typesAllowingCalls() {
        return super.typesAllowingCalls.concat(["livechat"]);
    },

    get isChatChannel() {
        return this.channel_type === "livechat" || super.isChatChannel;
    },
});

```

## File: static\src\core\common\@types\models.d.ts

```ts
declare module "models" {
    export interface Thread {
        operator: Persona,
    }
}

```

## File: static\src\core\public_web\discuss_app_category_model_patch.js

```javascript
import { patch } from "@web/core/utils/patch";
import { DiscussAppCategory } from "@mail/core/public_web/discuss_app_category_model";
import { Record } from "@mail/core/common/record";
import { compareDatetime } from "@mail/utils/common/misc";

patch(DiscussAppCategory.prototype, {
    setup() {
        super.setup(...arguments);
        this.livechatChannel = Record.one("im_livechat.channel", {
            inverse: "appCategory",
            onDelete() {
                this.delete();
            },
        });
    },
    /**
     * @param {import("models").Thread} t1
     * @param {import("models").Thread} t2
     */
    sortThreads(t1, t2) {
        if (this.livechatChannel || this.eq(this.app?.defaultLivechatCategory)) {
            return compareDatetime(t2.lastInterestDt, t1.lastInterestDt) || t2.id - t1.id;
        }
        return super.sortThreads(t1, t2);
    },
});

```

## File: static\src\core\public_web\discuss_app_model_patch.js

```javascript
import { DiscussApp } from "@mail/core/public_web/discuss_app_model";
import { Record } from "@mail/core/common/record";

import { _t } from "@web/core/l10n/translation";
import { patch } from "@web/core/utils/patch";

patch(DiscussApp.prototype, {
    setup(env) {
        super.setup(env);
        this.defaultLivechatCategory = Record.one("DiscussAppCategory", {
            compute() {
                return {
                    extraClass: "o-mail-DiscussSidebarCategory-livechat",
                    icon: "fa fa-comments",
                    hideWhenEmpty: true,
                    id: `im_livechat.category_default`,
                    name: _t("Livechat"),
                    sequence: 21,
                };
            },
        });
        this.livechats = Record.many("Thread", { inverse: "appAsLivechats" });
    },
});

```

## File: static\src\core\public_web\livechat_channel_model_patch.js

```javascript
import { LivechatChannel } from "@im_livechat/core/common/livechat_channel_model";
import { Record } from "@mail/core/common/record";

import { patch } from "@web/core/utils/patch";

const livechatChannelPatch = {
    setup() {
        super.setup(...arguments);
        this.appCategory = Record.one("DiscussAppCategory", {
            compute() {
                return {
                    extraClass: "o-mail-DiscussSidebarCategory-livechat",
                    hideWhenEmpty: !this.are_you_inside,
                    id: `im_livechat.category_${this.id}`,
                    name: this.name,
                    sequence: 22,
                };
            },
            inverse: "livechatChannel",
        });
        this.threads = Record.many("Thread", { inverse: "livechatChannel" });
    },
};
patch(LivechatChannel.prototype, livechatChannelPatch);

```

## File: static\src\core\public_web\messaging_menu_patch.js

```javascript
import { MessagingMenu } from "@mail/core/public_web/messaging_menu";

import { _t } from "@web/core/l10n/translation";
import { patch } from "@web/core/utils/patch";

patch(MessagingMenu.prototype, {
    /**
     * @override
     */
    get tabs() {
        const items = super.tabs;
        const hasLivechats = Object.values(this.store.Thread.records).some(
            ({ channel_type }) => channel_type === "livechat"
        );
        if (hasLivechats) {
            items.push({
                id: "livechat",
                icon: "fa fa-comments",
                label: _t("Livechat"),
            });
        }
        return items;
    },
});

```

## File: static\src\core\public_web\thread_model_patch.js

```javascript
import { Record } from "@mail/core/common/record";
import { Thread } from "@mail/core/common/thread_model";

import { patch } from "@web/core/utils/patch";

patch(Thread.prototype, {
    setup() {
        super.setup(...arguments);
        this.appAsLivechats = Record.one("DiscussApp", {
            compute() {
                return this.channel_type === "livechat" ? this.store.discuss : null;
            },
        });
        this.livechatChannel = Record.one("im_livechat.channel", { inverse: "threads" });
        this.anonymous_country = Record.one("Country");
    },
    _computeDiscussAppCategory() {
        if (this.channel_type !== "livechat") {
            return super._computeDiscussAppCategory();
        }
        return this.livechatChannel?.appCategory ?? this.appAsLivechats?.defaultLivechatCategory;
    },
    get hasMemberList() {
        return this.channel_type === "livechat" || super.hasMemberList;
    },
    get canLeave() {
        return this.channel_type !== "livechat" && super.canLeave;
    },
    get canUnpin() {
        if (this.channel_type === "livechat") {
            return !this.selfMember || this.selfMember.message_unread_counter === 0;
        }
        return super.canUnpin;
    },

    get correspondents() {
        return super.correspondents.filter((correspondent) => !correspondent.is_bot);
    },

    computeCorrespondent() {
        let correspondent = super.computeCorrespondent();
        if (this.channel_type === "livechat" && !correspondent) {
            // For livechat threads, the correspondent is the first
            // channel member that is not the operator.
            const orderedChannelMembers = [...this.channelMembers].sort((a, b) => a.id - b.id);
            const isFirstMemberOperator = orderedChannelMembers[0]?.persona.eq(this.operator);
            correspondent = isFirstMemberOperator
                ? orderedChannelMembers[1]
                : orderedChannelMembers[0];
        }
        return correspondent;
    },

    get displayName() {
        if (this.channel_type !== "livechat" || !this.correspondent) {
            return super.displayName;
        }
        if (!this.correspondent.persona.is_public && this.correspondent.persona.country) {
            return `${this.correspondent.name} (${this.correspondent.persona.country.name})`;
        }
        if (this.anonymous_country) {
            return `${this.correspondent.name} (${this.anonymous_country.name})`;
        }
        return this.correspondent.name;
    },

    get avatarUrl() {
        if (this.channel_type === "livechat" && this.correspondent) {
            return this.correspondent.persona.avatarUrl;
        }
        return super.avatarUrl;
    },

    /**
     * @override
     * @param {boolean} pushState
     */
    setAsDiscussThread(pushState) {
        super.setAsDiscussThread(pushState);
        if (this.store.env.services.ui.isSmall && this.channel_type === "livechat") {
            this.store.discuss.activeTab = "livechat";
        }
    },
});

```

## File: static\src\core\public_web\@types\models.d.ts

```ts
declare module "models" {
    export interface DiscussApp {
        defaultLivechatCategory: DiscussAppCategory,
        livechats: Thread,
    }
    export interface Thread {
        anonymous_country: Country,
        anonymous_name: String,
        appAsLivechats: DiscussApp,
    }
    export interface Store {
        has_access_livechat: boolean,
    }
    export interface LivechatChannel {
        threads: Thread[],
    }
}

```

## File: static\src\core\web\channel_commands_patch.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";

registry.category("discuss.channel_commands").add("history", {
    channel_types: ["livechat"],
    help: _t("See 15 last visited pages"),
    methodName: "execute_command_history",
});

```

## File: static\src\core\web\channel_invitation_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="discuss.ChannelInvitation" t-inherit-mode="extension">
        <xpath expr="//*[@name='selectablePartnerDetail']" position="replace">
            <t t-if="props.thread.channel_type !== 'livechat' or !selectablePartner.lang_name">$0</t>
            <div t-else="" class="d-flex flex-column flex-grow-1">
                <t>$0</t>
                <span class="mx-2 text-truncate text-start fs-6">
                    <i class="fa fa-comment-o me-1" aria-label="Lang"/>
                    <t t-esc="selectablePartner.lang_name"/>
                </span>
            </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\core\web\channel_member_list_patch.js

```javascript
import { ChannelMemberList } from "@mail/discuss/core/common/channel_member_list";
import { patch } from "@web/core/utils/patch";

patch(ChannelMemberList.prototype, {
    canOpenChatWith(member) {
        return super.canOpenChatWith(member) && !member.persona.is_public && !member.is_bot;
    },
});

```

## File: static\src\core\web\channel_member_list_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="discuss.channel_member" t-inherit-mode="extension">
        <xpath expr="//*[@t-ref='displayName']" position="inside">
                <div t-if="member.thread.channel_type === 'livechat'" class="ms-2 d-flex flex-wrap">
                    <span t-if="member.getLangName()" class="me-2">
                        <i class="fa fa-comment-o me-1" aria-label="Lang"/>
                        <t t-esc="member.getLangName()"/>
                    </span>
                    <span t-if="member.persona?.country or props.thread.anonymous_country">
                        <i class="fa fa-globe me-1" aria-label="country"/>
                        <t t-esc="member.persona?.country?.name ?? props.thread.anonymous_country.name"/>
                    </span>
                </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\core\web\chat_window_patch.js

```javascript
import { ChatWindow } from "@mail/core/common/chat_window";

import { patch } from "@web/core/utils/patch";

patch(ChatWindow.prototype, {
    async close(options) {
        const thread = this.thread;
        await super.close(options);
        if (thread?.channel_type === "livechat") {
            await thread?.isLoadedDeferred;
            if (thread.messages.length === 0) {
                thread.unpin();
            }
        }
    },
});

```

## File: static\src\core\web\command_palette.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-name="im_livechat.LivechatChannelCommand">
        <div class="o_command_default d-flex align-items-center justify-content-between px-4 py-2">
            <i class="me-2" t-att-class="props.iconClass"/>
            <span class="flex-grow-1 text-ellipsis">
                <t t-slot="name"/>
            </span>
        </div>
    </t>
</templates>

```

## File: static\src\core\web\composer_patch.js

```javascript
import { Composer } from "@mail/core/common/composer";
import { _t } from "@web/core/l10n/translation";

import { patch } from "@web/core/utils/patch";

patch(Composer.prototype, {
    onKeydown(ev) {
        super.onKeydown(ev);
        if (
            ev.key === "Tab" &&
            this.thread?.channel_type === "livechat" &&
            !this.props.composer.text
        ) {
            const threadChanged = this.store.goToOldestUnreadLivechatThread();
            if (threadChanged) {
                // prevent chat window from switching to the next thread: as
                // we want to go to the oldest unread thread, not the next
                // one.
                ev.stopPropagation();
            }
        }
    },
    get placeholder() {
        if (
            this.displayNextLivechatHint() &&
            this.props.composer.isFocused &&
            this.env.inChatWindow
        ) {
            return _t("Tab to next livechat");
        }
        return super.placeholder;
    },
    displayNextLivechatHint() {
        return (
            this.thread?.channel_type === "livechat" &&
            this.store.discuss.livechats.some(
                (thread) => thread.notEq(this.thread) && thread.isUnread
            )
        );
    },
});

```

## File: static\src\core\web\composer_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="im_livechat.Composer" t-inherit="mail.Composer" t-inherit-mode="extension">
        <xpath expr="//Typing" position="replace">
            <div class="d-flex justify-content-between align-items-center w-100">
                <div class="d-flex">$0</div>
                <span t-if="displayNextLivechatHint() and !env.inChatWindow" class="text-muted fst-italic form-text">Tab to next livechat</span>
            </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\core\web\discuss_client_action_patch.js

```javascript
import { DiscussClientAction } from "@mail/core/public_web/discuss_client_action";

import { patch } from "@web/core/utils/patch";

patch(DiscussClientAction.prototype, {
    async restoreDiscussThread() {
        if (this.store.has_access_livechat) {
            this.store.livechatChannels.fetch();
        }
        return super.restoreDiscussThread(...arguments);
    },
});

```

## File: static\src\core\web\discuss_sidebar_category_patch.js

```javascript
import { DiscussSidebarCategory } from "@mail/discuss/core/public_web/discuss_sidebar_categories";
import { patch } from "@web/core/utils/patch";

/** @type {import("@mail/discuss/core/public_web/discuss_sidebar_categories").DiscussSidebarCategory} */
const DiscussSidebarCategoryPatch = {
    get actions() {
        const actions = super.actions;
        if (this.store.has_access_livechat && this.category.livechatChannel && this.category.open) {
            actions.push({
                onSelect: () => {
                    if (this.category.livechatChannel.are_you_inside) {
                        this.category.livechatChannel.leave({ notify: false });
                    } else {
                        this.category.livechatChannel.join({ notify: false });
                    }
                },
                label: this.category.livechatChannel.are_you_inside
                    ? this.category.livechatChannel.leaveTitle
                    : this.category.livechatChannel.joinTitle,
                icon: this.category.livechatChannel.are_you_inside
                    ? "fa fa-sign-out text-danger"
                    : "fa fa-sign-in text-success",
            });
        }
        return actions;
    },
};

patch(DiscussSidebarCategory.prototype, DiscussSidebarCategoryPatch);

```

## File: static\src\core\web\livechat_channel_command_provider.js

```javascript
import { Component } from "@odoo/owl";
import { registry } from "@web/core/registry";

class LivechatChannelCommand extends Component {
    static template = "im_livechat.LivechatChannelCommand";
    static props = {
        executeCommand: Function,
        iconClass: String,
        name: String,
        searchValue: String,
        slots: Object,
    };
}

registry.category("command_provider").add("im_livechat.channel_join_leave", {
    /**
     * @param {import("@web/env").OdooEnv} env
     */
    async provide(env) {
        const store = env.services["mail.store"];
        if (!store?.has_access_livechat) {
            return [];
        }
        await store.livechatChannels.fetch();
        const activeChannels = new Set(
            Object.values(store["im_livechat.channel"].records)
                .filter((c) => c.threads.length > 0)
                .map((c) => c.id)
        );
        // Show live chat channels with ongoing conversations first
        return Object.values(store["im_livechat.channel"].records)
            .sort((c) => (activeChannels.has(c.id) ? -1 : 1))
            .map((c) => ({
                action: c.are_you_inside ? c.leave.bind(c) : c.join.bind(c),
                Component: LivechatChannelCommand,
                name: c.are_you_inside ? c.leaveTitle : c.joinTitle,
                props: {
                    iconClass: c.are_you_inside
                        ? "fa fa-sign-out text-danger"
                        : "fa fa-sign-in text-success",
                },
            }));
    },
});

```

## File: static\src\core\web\livechat_channel_model_patch.js

```javascript
import { LivechatChannel } from "@im_livechat/core/common/livechat_channel_model";

import { useSequential } from "@mail/utils/common/hooks";

import { _t } from "@web/core/l10n/translation";
import { patch } from "@web/core/utils/patch";
import { sprintf } from "@web/core/utils/strings";

const sequential = useSequential();

const livechatChannelPatch = {
    async join({ notify = true } = {}) {
        this.are_you_inside = true;
        if (notify) {
            this.store.env.services.notification.add(_t("You joined %s.", this.name), {
                type: "info",
            });
        }
        await sequential(() =>
            this.store.env.services.orm.call("im_livechat.channel", "action_join", [this.id])
        );
    },
    get joinTitle() {
        return sprintf(_t("Join %s"), this.name);
    },
    async leave({ notify = true } = {}) {
        this.are_you_inside = false;
        if (notify) {
            this.store.env.services.notification.add(_t("You left %s.", this.name), {
                type: "info",
            });
        }
        await sequential(() =>
            this.store.env.services.orm.call("im_livechat.channel", "action_quit", [this.id])
        );
    },
    get leaveTitle() {
        return sprintf(_t("Leave %s"), this.name);
    },
};
patch(LivechatChannel.prototype, livechatChannelPatch);

```

## File: static\src\core\web\partner_compare.js

```javascript
import { partnerCompareRegistry } from "@mail/core/common/partner_compare";

partnerCompareRegistry.add(
    "im_livechat.available",
    (p1, p2, { thread }) => {
        if (thread?.channel_type === "livechat" && p1.is_available !== p2.is_available) {
            return p1.is_available ? -1 : 1;
        }
    },
    { sequence: 15 }
);

partnerCompareRegistry.add(
    "im_livechat.invite-count",
    (p1, p2, { thread }) => {
        if (
            thread?.channel_type === "livechat" &&
            p1.invite_by_self_count !== p2.invite_by_self_count
        ) {
            return p2.invite_by_self_count - p1.invite_by_self_count;
        }
    },
    { sequence: 20 }
);

```

## File: static\src\core\web\store_service_patch.js

```javascript
import { Store } from "@mail/core/common/store_service";
import { compareDatetime } from "@mail/utils/common/misc";

import { patch } from "@web/core/utils/patch";

patch(Store.prototype, {
    setup() {
        super.setup(...arguments);
        this.livechatChannels = this.makeCachedFetchData({ livechat_channels: true });
        this.has_access_livechat = false;
    },
    /**
     * @override
     */
    onStarted() {
        super.onStarted(...arguments);
        if (this.discuss.isActive && this.has_access_livechat) {
            this.livechatChannels.fetch();
        }
    },
    /** @returns {boolean} Whether the livechat thread changed. */
    goToOldestUnreadLivechatThread() {
        const [oldestUnreadThread] = this.discuss.livechats
            .filter((thread) => thread.isUnread)
            .sort(
                (t1, t2) => compareDatetime(t1.lastInterestDt, t2.lastInterestDt) || t1.id - t2.id
            );
        if (!oldestUnreadThread) {
            return false;
        }
        if (this.discuss.isActive) {
            oldestUnreadThread.setAsDiscussThread();
            return true;
        }
        const chatWindow = this.ChatWindow.insert({ thread: oldestUnreadThread });
        chatWindow.open();
        chatWindow.focus();
        return true;
    },
    /**
     * @override
     */
    tabToThreadType(tab) {
        const threadTypes = super.tabToThreadType(tab);
        if (tab === "chat" && !this.env.services.ui.isSmall) {
            threadTypes.push("livechat");
        }
        if (tab === "livechat") {
            threadTypes.push("livechat");
        }
        return threadTypes;
    },
});

```

## File: static\src\core\web\suggestion_service_patch.js

```javascript
import { SuggestionService } from "@mail/core/common/suggestion_service";

import { patch } from "@web/core/utils/patch";

patch(SuggestionService.prototype, {
    /** @override */
    getSupportedDelimiters(thread) {
        const res = super.getSupportedDelimiters(...arguments);
        return thread.channel_type === "livechat"
            ? res.filter((delimiter) => delimiter.at(0) !== "#")
            : res;
    },
});

```

## File: static\src\core\web\thread_icon_patch.js

```javascript
import { ThreadIcon } from "@mail/core/common/thread_icon";
import { _t } from "@web/core/l10n/translation";

import { patch } from "@web/core/utils/patch";

patch(ThreadIcon.prototype, {
    get defaultChatIcon() {
        if (this.props.thread.channel_type === "livechat") {
            return { class: "fa fa-comments opacity-75", title: _t("Livechat") };
        }
        return super.defaultChatIcon;
    },
});

```

## File: static\src\embed\common\attachment_upload_service_patch.js

```javascript
import { AttachmentUploadService } from "@mail/core/common/attachment_upload_service";

import { patch } from "@web/core/utils/patch";

patch(AttachmentUploadService.prototype, {
    async upload(thread, composer, file, options) {
        if (thread.channel_type === "livechat") {
            thread = await this.env.services["im_livechat.livechat"].persist();
            composer = thread.composer;
            if (!thread) {
                return;
            }
        }
        return super.upload(thread, composer, file, options);
    },
});

```

## File: static\src\embed\common\autopopup_service.js

```javascript
import { expirableStorage } from "@im_livechat/embed/common/expirable_storage";
import { SESSION_STATE } from "@im_livechat/embed/common/livechat_service";
import { browser } from "@web/core/browser/browser";
import { registry } from "@web/core/registry";

export class AutopopupService {
    static STORAGE_KEY = "im_livechat_auto_popup";

    /**
     * @param {import("@web/env").OdooEnv} env
     * @param {{
     * "im_livechat.chatbot": import("@im_livechat/embed/common/chatbot/chatbot_service").ChatBotService,
     * "im_livechat.initialized": import("@im_livechat/embed/common/livechat_initialized_service").livechatInitializedService,
     * "im_livechat.livechat": import("@im_livechat/embed/common/livechat_service").LivechatService,
     * "mail.store": import("@mail/core/common/store_service").Store,
     * ui: typeof import("@web/core/ui/ui_service").uiService.start,
     * }} services
     */
    constructor(
        env,
        {
            "im_livechat.chatbot": chatbotService,
            "im_livechat.initialized": livechatInitializedService,
            "im_livechat.livechat": livechatService,
            "mail.store": storeService,
            ui,
        }
    ) {
        this.storeService = storeService;
        this.livechatService = livechatService;
        this.chatbotService = chatbotService;
        this.ui = ui;

        livechatInitializedService.ready.then(() => {
            if (this.allowAutoPopup && livechatService.state === SESSION_STATE.NONE) {
                browser.setTimeout(async () => {
                    if (!this.storeService.ChatWindow.get({ thread: livechatService.thread })) {
                        expirableStorage.setItem(AutopopupService.STORAGE_KEY, true);
                        livechatService.open();
                    }
                }, livechatService.rule.auto_popup_timer * 1000);
            }
        });
    }

    get allowAutoPopup() {
        return Boolean(
            !expirableStorage.getItem(AutopopupService.STORAGE_KEY) &&
                !this.ui.isSmall &&
                this.livechatService.rule?.action === "auto_popup" &&
                this.livechatService.available
        );
    }
}

export const autoPopupService = {
    dependencies: [
        "im_livechat.livechat",
        "im_livechat.initialized",
        "im_livechat.chatbot",
        "mail.store",
        "ui",
    ],

    start(env, services) {
        return new AutopopupService(env, services);
    },
};
registry.category("services").add("im_livechat.autopopup", autoPopupService);

```

## File: static\src\embed\common\boot_helpers.js

```javascript
import { url } from "@web/core/utils/urls";

async function loadFont(name, url) {
    await document.fonts.ready;
    if ([...document.fonts].some(({ family }) => family === name)) {
        // Font already loaded.
        return;
    }
    const link = document.createElement("link");
    link.rel = "preload";
    link.as = "font";
    link.href = url;
    link.crossOrigin = "";
    const style = document.createElement("style");
    style.appendChild(
        document.createTextNode(`
            @font-face {
                font-family: ${name};
                src: url('${url}') format('woff2');
                font-weight: normal;
                font-style: normal;
                font-display: block;
            }
        `)
    );
    const loadPromise = new Promise((res, rej) => {
        link.addEventListener("load", res);
        link.addEventListener("error", rej);
    });
    document.head.appendChild(link);
    document.head.appendChild(style);
    return loadPromise;
}

/**
 * @param {HTMLElement} target
 * @returns {HTMLDivElement}
 */
export function makeRoot(target) {
    const root = document.createElement("div");
    root.classList.add("o-livechat-root");
    root.setAttribute("id", `o-livechat-root-${luxon.DateTime.now().ts + Math.random()}`);
    root.style.zIndex = "calc(9e999)";
    root.style.position = "relative";
    target.appendChild(root);
    return root;
}

/**
 * Initialize the livechat container by loading the styles and
 * the fonts.
 *
 * @param {HTMLElement} root
 * @returns {ShadowRoot}
 */
export async function makeShadow(root) {
    const link = document.createElement("link");
    link.rel = "stylesheet";
    link.href = url("/im_livechat/assets_embed.css");
    const stylesLoadedPromise = new Promise((res, rej) => {
        link.addEventListener("load", res);
        link.addEventListener("error", rej);
    });
    const shadow = root.attachShadow({ mode: "open" });
    shadow.appendChild(link);
    await Promise.all([
        stylesLoadedPromise,
        loadFont("FontAwesome", url("/im_livechat/font-awesome")),
        loadFont("odoo_ui_icons", url("/im_livechat/odoo_ui_icons")),
    ]);
    return shadow;
}

```

## File: static\src\embed\common\chat_hub_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.ChatHub" t-inherit-mode="extension">
        <xpath expr="//*[hasclass('o-mail-ChatHub')]" position="attributes">
            <attribute name="part">ChatHub</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\embed\common\chat_window_model_patch.js

```javascript
import { SESSION_STATE } from "@im_livechat/embed/common/livechat_service";
import { ChatWindow } from "@mail/core/common/chat_window_model";
import { patch } from "@web/core/utils/patch";

patch(ChatWindow.prototype, {
    setup() {
        super.setup();
        this.hasFeedbackPanel = false;
    },
    async close() {
        if (this.thread?.channel_type !== "livechat") {
            return super.close(...arguments);
        }
        if (this.store.env.services["im_livechat.livechat"].state === SESSION_STATE.PERSISTED) {
            this.hasFeedbackPanel = true;
            this.open({ notifyState: this.thread?.state !== "open" });
        } else {
            await super.close(...arguments);
            if (this.thread.isTransient) {
                this.thread.delete();
            }
        }
        this.store.env.services["im_livechat.livechat"].leave();
        this.store.env.services["im_livechat.chatbot"].stop();
    },
});

```

## File: static\src\embed\common\chat_window_patch.js

```javascript
import { FeedbackPanel } from "@im_livechat/embed/common/feedback_panel/feedback_panel";
import { CloseConfirmation } from "@im_livechat/embed/common/close_confirmation";

import { ChatWindow } from "@mail/core/common/chat_window";

import { toRaw, useState } from "@odoo/owl";

import { useService } from "@web/core/utils/hooks";
import { patch } from "@web/core/utils/patch";

Object.assign(ChatWindow.components, { FeedbackPanel, CloseConfirmation });

patch(ChatWindow.prototype, {
    setup() {
        super.setup(...arguments);
        this.livechatService = useService("im_livechat.livechat");
        this.chatbotService = useState(useService("im_livechat.chatbot"));
        this.livechatState = useState({ showCloseConfirmation: false });
    },

    async close() {
        const chatWindow = toRaw(this.props.chatWindow);
        if (chatWindow.thread.id > 0 && !this.livechatState.showCloseConfirmation) {
            this.state.actionsDisabled = true;
            this.livechatState.showCloseConfirmation = true;
        } else {
            this.state.actionsDisabled = false;
            await super.close();
        }
    },

    onCloseConfirmationDialog() {
        this.state.actionsDisabled = false;
        this.livechatState.showCloseConfirmation = false;
    },
});

```

## File: static\src\embed\common\chat_window_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.ChatWindow" t-inherit-mode="extension">
        <xpath expr="//*[@name='thread content']" position="replace">
           <FeedbackPanel t-if="props.chatWindow.hasFeedbackPanel" onClickClose="() => this.close()" thread="thread"/>
           <t t-else="">
            <t>$0</t>
            <t t-if="this.livechatState.showCloseConfirmation">
                <CloseConfirmation onCloseConfirmationDialog.bind="onCloseConfirmationDialog" onClickLeaveConversation.bind="close"/>
            </t>
           </t>
        </xpath>
        <xpath expr="//*[@t-ref='needactionCounter']" position="replace">
            <t t-if="!chatbotService.chatbot">$0</t>
        </xpath>
        <xpath expr="//*[hasclass('o-mail-ChatWindow-header')]" position="attributes">
            <attribute name="t-attf-style" add="color: {{ livechatService.options.title_color }}; background-color: {{ livechatService.options.header_background_color }} !important;" separator=" "/>
        </xpath>
        <xpath expr="//Composer" position="replace">
            <t t-if="chatbotService.inputEnabled">$0</t>
            <t t-else="">
                <span class="bg-200 py-1 text-center fst-italic" t-esc="chatbotService.inputDisabledText"/>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\embed\common\close_confirmation.js

```javascript
import { Component } from "@odoo/owl";
import { useAutofocus } from "@web/core/utils/hooks";

export class CloseConfirmation extends Component {
    static template = "im_livechat.CloseConfirmation";
    static props = ["onCloseConfirmationDialog", "onClickLeaveConversation"];

    setup() {
        useAutofocus({ refName: "root" });
    }

    onKeydown(ev) {
        if (ev.key === "Escape") {
            this.props.onCloseConfirmationDialog();
        } else if (ev.key === "Enter") {
            this.props.onClickLeaveConversation();
        }
    }
}

```

## File: static\src\embed\common\close_confirmation.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="im_livechat.CloseConfirmation">
        <div
            class="o-livechat-CloseConfirmation bg-black-50 position-absolute w-100 h-100 d-flex justify-content-center align-items-center"
            t-on-keydown.capture.stop="onKeydown"
            t-on-click.stop="() => this.props.onCloseConfirmationDialog()"
            t-ref="root"
            tabindex="1"
        >
            <div class="o-livechat-CloseConfirmation-dialog rounded bg-white bg-opacity-100 p-3 m-3 d-flex flex-column" t-ref="dialog">
                <div class="d-flex justify-content-end">
                    <button class="o-livechat-CloseConfirmation-close btn-close" t-on-click.stop="() => this.props.onCloseConfirmationDialog()"/>
                </div>
                <span class="pt-2 pb-2">You're about to leave the conversation, proceed?</span>
                <button class="o-livechat-CloseConfirmation-leave btn btn-danger p-2 gap-1" t-on-click.stop="() => this.props.onClickLeaveConversation()"><i class="fa fa-fw fa-sign-out"/>Yes, leave conversation</button>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\embed\common\composer_patch.js

```javascript
import { Composer } from "@mail/core/common/composer";

import { _t } from "@web/core/l10n/translation";
import { patch } from "@web/core/utils/patch";
import { session } from "@web/session";

patch(Composer.prototype, {
    get placeholder() {
        if (this.thread?.channel_type !== "livechat") {
            return super.placeholder;
        }
        return session.livechatData.options.input_placeholder || _t("Say something...");
    },
});

```

## File: static\src\embed\common\disabled_features.js

```javascript
import { threadActionsRegistry } from "@mail/core/common/thread_actions";
import { Thread } from "@mail/core/common/thread_model";

import { patch } from "@web/core/utils/patch";

patch(Thread.prototype, {
    get hasMemberList() {
        return false;
    },
    get hasAttachmentPanel() {
        return this.channel_type !== "livechat" && super.hasAttachmentPanel;
    },
});

const allowedThreadActions = new Set(["fold-chat-window", "close", "restart", "settings"]);
for (const [actionName] of threadActionsRegistry.getEntries()) {
    if (!allowedThreadActions.has(actionName)) {
        threadActionsRegistry.remove(actionName);
    }
}
threadActionsRegistry.addEventListener("UPDATE", ({ detail: { operation, key } }) => {
    if (operation === "add" && !allowedThreadActions.has(key)) {
        threadActionsRegistry.remove(key);
    }
});

```

## File: static\src\embed\common\expirable_storage.js

```javascript
import { browser } from "@web/core/browser/browser";

const BASE_STORAGE_KEY = "EXPIRABLE_STORAGE";
const CLEAR_INTERVAL = 24 * 60 * 60 * 1000; // 24 hours in milliseconds

function cleanupExpirableStorage() {
    const now = Date.now();
    // Next line is for testing compatibility as for..in is not supported by
    // the `MockStorage` class.
    const keys = browser.localStorage.items?.keys() ?? Object.keys(browser.localStorage);
    for (const key of keys) {
        if (key.startsWith(BASE_STORAGE_KEY)) {
            const item = JSON.parse(browser.localStorage.getItem(key));
            if (item.expires && item.expires < now) {
                browser.localStorage.removeItem(key);
            }
        }
    }
}

export const expirableStorage = {
    /** @param {string} key */
    getItem(key) {
        cleanupExpirableStorage();
        const item = browser.localStorage.getItem(`${BASE_STORAGE_KEY}_${key}`);
        if (item) {
            return JSON.parse(item).value;
        }
        return null;
    },
    /**
     * @param {string} key
     * @param {string} value
     * @param {number} ttl Number of seconds after which the item should expire.
     */
    setItem(key, value, ttl) {
        let expires;
        if (ttl) {
            expires = Date.now() + ttl * 1000;
        }
        browser.localStorage.setItem(
            `${BASE_STORAGE_KEY}_${key}`,
            JSON.stringify({ value, expires })
        );
    },
    /** @param {string} key */
    removeItem(key) {
        browser.localStorage.removeItem(`${BASE_STORAGE_KEY}_${key}`);
    },
};

cleanupExpirableStorage();
setInterval(cleanupExpirableStorage, CLEAR_INTERVAL);

```

## File: static\src\embed\common\history_service.js

```javascript
import { expirableStorage } from "@im_livechat/embed/common/expirable_storage";
import { browser } from "@web/core/browser/browser";
import { rpc } from "@web/core/network/rpc";
import { registry } from "@web/core/registry";

export class HistoryService {
    static HISTORY_STORAGE_KEY = "im_livechat_history";
    static HISTORY_LIMIT = 15;

    constructor(env, services) {
        /** @type {ReturnType<typeof import("@bus/services/bus_service").busService.start>} */
        this.busService = services.bus_service;
        /** @type {import("@im_livechat/embed/common/livechat_service").LivechatService} */
        this.livechatService = services["im_livechat.livechat"];
    }

    setup() {
        this.updateHistory();
        this.busService.subscribe("im_livechat.history_command", (payload) => {
            if (payload.id !== this.livechatService.thread?.id) {
                return;
            }
            const data = expirableStorage.getItem(HistoryService.HISTORY_STORAGE_KEY);
            const history = data ? JSON.parse(data) : [];
            rpc("/im_livechat/history", {
                pid: this.livechatService.thread.operator.id,
                channel_id: this.livechatService.thread.id,
                page_history: history,
            });
        });
    }

    updateHistory() {
        const page = browser.location.href.replace(/^.*\/\/[^/]+/, "");
        const pageHistory = expirableStorage.getItem(HistoryService.HISTORY_STORAGE_KEY);
        const urlHistory = pageHistory ? JSON.parse(pageHistory) : [];
        if (!urlHistory.includes(page)) {
            urlHistory.push(page);
            if (urlHistory.length > HistoryService.HISTORY_LIMIT) {
                urlHistory.shift();
            }
            expirableStorage.setItem(
                HistoryService.HISTORY_STORAGE_KEY,
                JSON.stringify(urlHistory),
                60 * 60 * 24 // kept for 1 day
            );
        }
    }
}

export const historyService = {
    dependencies: ["im_livechat.livechat", "bus_service"],
    start(env, services) {
        const history = new HistoryService(env, services);
        history.setup();
    },
};

registry.category("services").add("im_livechat.history_service", historyService);

```

## File: static\src\embed\common\livechat_button.js

```javascript
import { SESSION_STATE } from "@im_livechat/embed/common/livechat_service";
import { Component, useExternalListener, useRef, useState } from "@odoo/owl";

import { useMovable } from "@mail/utils/common/hooks";

import { useService } from "@web/core/utils/hooks";
import { debounce } from "@web/core/utils/timing";

const LIVECHAT_BUTTON_SIZE = 56;

export class LivechatButton extends Component {
    static template = "im_livechat.LivechatButton";
    static props = {};
    static DEBOUNCE_DELAY = 500;

    setup() {
        this.store = useState(useService("mail.store"));
        /** @type {import('@im_livechat/embed/common/livechat_service').LivechatService} */
        this.livechatService = useState(useService("im_livechat.livechat"));
        this.onClick = debounce(this.onClick.bind(this), LivechatButton.DEBOUNCE_DELAY, {
            leading: true,
        });
        this.ref = useRef("button");
        this.size = LIVECHAT_BUTTON_SIZE;
        this.position = useState({
            left: `calc(97% - ${LIVECHAT_BUTTON_SIZE}px)`,
            top: `calc(97% - ${LIVECHAT_BUTTON_SIZE}px)`,
        });
        this.state = useState({
            animateNotification: !(
                this.livechatService.thread || this.livechatService.shouldRestoreSession
            ),
            hasAlreadyMovedOnce: false,
        });
        useMovable({
            cursor: "grabbing",
            ref: this.ref,
            elements: ".o-livechat-LivechatButton",
            onDrop: ({ top, left }) => {
                this.state.hasAlreadyMovedOnce = true;
                this.position.left = `${left}px`;
                this.position.top = `${top}px`;
            },
        });
        useExternalListener(document.body, "scroll", this._onScroll, { capture: true });
    }

    _onScroll(ev) {
        if (!this.ref.el || this.state.hasAlreadyMovedOnce) {
            return;
        }
        const container = ev.target;
        this.position.top =
            container.scrollHeight - container.scrollTop === container.clientHeight
                ? `calc(93% - ${LIVECHAT_BUTTON_SIZE}px)`
                : `calc(97% - ${LIVECHAT_BUTTON_SIZE}px)`;
    }

    onClick() {
        this.state.animateNotification = false;
        this.livechatService.open();
    }

    get isShown() {
        return (
            this.livechatService.initialized &&
            this.livechatService.available &&
            this.livechatService.state === SESSION_STATE.NONE &&
            Object.keys(this.store.ChatWindow.records).length === 0
        );
    }
}

```

## File: static\src\embed\common\livechat_button.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

<t t-name="im_livechat.LivechatButton">
    <button
        part="openChatButton"
        t-if="isShown"
        class="btn o-livechat-LivechatButton d-print-none position-fixed p-3 d-flex justify-content-center align-items-center shadow rounded-circle "
        t-attf-style="color: {{livechatService.options.button_text_color}}; background-color: {{livechatService.options.button_background_color}}; width: {{size}}px; height: {{size}}px; min-width: 56px; top: {{ position.top }}; left: {{ position.left }};"
        t-ref="button"
        t-on-click="onClick"
        title="Drag to Move"
    >
        <i class="fa fa-commenting" style="font-size: 24px;"/>
        <div t-if="livechatService.rule?.action === 'display_button_and_text'" class="o-livechat-LivechatButton-notification text-nowrap position-absolute bg-100 py-2 px-3 rounded" style="max-width: 75vw;" t-att-class="{'o-livechat-LivechatButton-animate': state.animateNotification}">
            <p class="m-0 text-dark text-truncate" t-esc="livechatService.options.button_text"/>
        </div>
    </button>
</t>

</templates>

```

## File: static\src\embed\common\livechat_initialized_service.js

```javascript
import { registry } from "@web/core/registry";
import { Deferred } from "@web/core/utils/concurrency";

export const livechatInitializedService = {
    start() {
        return {
            ready: new Deferred(),
        };
    },
};
registry.category("services").add("im_livechat.initialized", livechatInitializedService);

```

## File: static\src\embed\common\livechat_rule_model.js

```javascript
import { Record } from "@mail/core/common/record";

export class LivechatRule extends Record {
    /** @type {"auto_popup"|"display_button_and_text"|undefined} */
    action;
    /** @type {number} */
    autoPopupTimer;
    /** @type {string} */
    regexURL;
    chatbotScript = Record.one("ChatbotScript");
}
LivechatRule.register();

```

## File: static\src\embed\common\livechat_service.js

```javascript
import { expirableStorage } from "@im_livechat/embed/common/expirable_storage";

import { reactive } from "@odoo/owl";
import { rpc } from "@web/core/network/rpc";

import { cookie } from "@web/core/browser/cookie";
import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { session } from "@web/session";
import { user } from "@web/core/user";

/**
 * @typedef LivechatRule
 * @property {"auto_popup"|"display_button_and_text"|undefined} [action]
 * @property {number?} [auto_popup_timer]
 * @property {import("@im_livechat/embed/common/chatbot/chatbot_model").IChatbot} [chatbot]
 */

export const RATING = Object.freeze({
    GOOD: 5,
    OK: 3,
    BAD: 1,
});

export const SESSION_STATE = Object.freeze({
    NONE: "NONE",
    CREATED: "CREATED",
    PERSISTED: "PERSISTED",
});

export const ODOO_VERSION_KEY = `${location.origin.replace(
    /:\/{0,2}/g,
    "_"
)}_im_livechat.odoo_version`;

const OPERATOR_STORAGE_KEY = "im_livechat_previous_operator";
const GUEST_TOKEN_STORAGE_KEY = "im_livechat_guest_token";
const SAVED_STATE_STORAGE_KEY = "im_livechat.saved_state";
const LIVECHAT_UUID_COOKIE = "im_livechat_uuid";

export function getGuestToken() {
    return expirableStorage.getItem(GUEST_TOKEN_STORAGE_KEY);
}

export class LivechatService {
    /** @type {keyof typeof SESSION_STATE} */
    state = SESSION_STATE.NONE;
    /** @type {LivechatRule} */
    rule;
    initialized = false;
    available = session.livechatData?.isAvailable;
    /** @type {import("models").Thread} */
    thread;
    _onStateChangeCallbacks = {
        [SESSION_STATE.CREATED]: [],
        [SESSION_STATE.PERSISTED]: [],
        [SESSION_STATE.NONE]: [],
    };

    constructor(env, services) {
        this.setup(env, services);
    }

    /**
     * @param {import("@web/env").OdooEnv} env
     * @param {{
     * bus_service: ReturnType<typeof import("@bus/services/bus_service").busService.start>,
     * "mail.store": import("@mail/core/common/store_service").Store
     * }} services
     */
    setup(env, services) {
        this.env = env;
        this.busService = services.bus_service;
        this.notificationService = services.notification;
        this.store = services["mail.store"];
    }

    async initialize() {
        const data =
            this.options?.init ??
            (await rpc("/im_livechat/init", {
                channel_id: this.options.channel_id,
            }));
        this.available = data.available_for_me;
        this.rule = this.store.LivechatRule.insert(data.rule);
        this.store.insert(data.storeData);
        if (this.options?.force_thread) {
            this.state = SESSION_STATE.PERSISTED;
            this.thread = this.store.Thread.insert(this.options.force_thread);
            this._saveLivechatState();
        }
        if (this.savedState) {
            this.state = this.savedState.persisted
                ? SESSION_STATE.PERSISTED
                : SESSION_STATE.CREATED;
        }
        if (this.state === SESSION_STATE.PERSISTED) {
            await this.busService.addChannel(`mail.guest_${this.guestToken}`);
        }
        this.initialized = true;
        this.env.services["im_livechat.initialized"].ready.resolve();
    }

    /**
     * Open a new live chat thread.
     *
     * @returns {Promise<import("models").Thread|undefined>}
     */
    async open() {
        await this._createThread({ persist: false });
        this.thread?.openChatWindow();
    }

    /**
     * Persist the livechat thread if it is not done yet and swap it with the
     * temporary thread.
     *
     * @returns {Promise<import("models").Thread|undefined>}
     */
    async persist() {
        if (this.state === SESSION_STATE.PERSISTED) {
            return this.thread;
        }
        const temporaryThread = this.thread;
        await this._createThread({ persist: true });
        if (temporaryThread) {
            const chatWindow = this.store.ChatWindow.get({ thread: temporaryThread });
            temporaryThread.delete();
            await chatWindow.close();
        }
        if (!this.thread) {
            return;
        }
        this.store.chatHub.opened.add({ thread: this.thread }).autofocus++;
        await this.busService.addChannel(`mail.guest_${this.guestToken}`);
        await this.env.services["mail.store"].initialize();
        return this.thread;
    }

    /**
     * @param {object} param0
     * @param {boolean} param0.notifyServer Whether to call the
     * `visitor_leave_session` route. Note that this route will never be called
     * if the session was not persisted.
     */
    async leave({ notifyServer = true } = {}) {
        try {
            if (this.thread && this.state === SESSION_STATE.PERSISTED && notifyServer) {
                await rpc("/im_livechat/visitor_leave_session", { channel_id: this.thread.id });
            }
        } finally {
            this.thread = undefined;
            expirableStorage.removeItem(SAVED_STATE_STORAGE_KEY);
            cookie.delete(LIVECHAT_UUID_COOKIE);
            this.state = SESSION_STATE.NONE;
            await Promise.all(this._onStateChangeCallbacks[SESSION_STATE.NONE].map((fn) => fn()));
        }
    }

    /**
     * Add a callback to be executed when the livechat service state changes.
     *
     * @param {keyof typeof SESSION_STATE} state
     * @param {Function} callback
     */
    onStateChange(state, callback) {
        this._onStateChangeCallbacks[state].push(callback);
    }

    /**
     * Save the current live chat state. Only save the strict minimum if the
     * thread is persisted.
     *
     * @param {Object} [saveData]
     */
    _saveLivechatState(saveData) {
        const { guest_token } = this.store;
        if (guest_token) {
            expirableStorage.setItem(GUEST_TOKEN_STORAGE_KEY, guest_token);
        }
        const ONE_DAY_TTL = 60 * 60 * 24;
        if (this.thread.uuid) {
            cookie.set(LIVECHAT_UUID_COOKIE, this.thread.uuid, ONE_DAY_TTL);
        }
        const persisted = this.state === SESSION_STATE.PERSISTED;
        expirableStorage.setItem(
            SAVED_STATE_STORAGE_KEY,
            JSON.stringify({
                livechatUserId: this.savedState?.livechatUserId ?? user.userId,
                persisted,
                store: persisted ? { "discuss.channel": [{ id: this.thread.id }] } : saveData,
            }),
            ONE_DAY_TTL
        );
        if (this.thread.operator) {
            expirableStorage.setItem(
                OPERATOR_STORAGE_KEY,
                this.thread.operator.id,
                ONE_DAY_TTL * 7
            );
        }
    }

    /**
     * @param {object} param0
     * @param {boolean} [param0.persist=false]
     * @returns {Promise<import("models").Thread>}
     */
    async _createThread({ persist = false }) {
        const data = await rpc(
            "/im_livechat/get_session",
            {
                channel_id: this.options.channel_id,
                anonymous_name: this.options.default_username ?? _t("Visitor"),
                chatbot_script_id: this.savedState
                    ? this.thread.chatbot?.script.id
                    : this.rule.chatbotScript?.id,
                previous_operator_id: expirableStorage.getItem(OPERATOR_STORAGE_KEY),
                temporary_id: this.thread?.id,
                persisted: persist,
            },
            { shadow: true }
        );
        // clean copy of data for saving in storage, because store insert will add cyclic references
        const saveData = JSON.parse(JSON.stringify(data));
        const { Thread = [] } = this.store.insert(data);
        this.thread = Thread[0];
        if (!this.thread?.operator) {
            this.notificationService.add(_t("No available collaborator, please try again later."));
            this.leave({ notifyServer: false });
            return;
        }
        this.state = persist ? SESSION_STATE.PERSISTED : SESSION_STATE.CREATED;
        this._saveLivechatState(saveData);
        await Promise.all(this._onStateChangeCallbacks[this.state].map((fn) => fn()));
    }

    get options() {
        return session.livechatData?.options ?? {};
    }

    get savedState() {
        return JSON.parse(expirableStorage.getItem(SAVED_STATE_STORAGE_KEY) ?? false);
    }

    /**
     * @returns {string|undefined}
     */
    get guestToken() {
        return getGuestToken();
    }
}

export const livechatService = {
    dependencies: ["bus_service", "im_livechat.initialized", "mail.store", "notification"],
    start(env, services) {
        const livechat = reactive(new LivechatService(env, services));
        (async () => {
            // Live chat state should be deleted if it is linked to another user
            // (log in/out after chat start).
            if ((livechat.savedState?.livechatUserId || false) !== (user.userId || false)) {
                await livechat.leave({ notifyServer: false });
            }
            if (livechat.available) {
                livechat.initialize();
            }
        })();
        return livechat;
    },
};
registry.category("services").add("im_livechat.livechat", livechatService);

```

## File: static\src\embed\common\message_model_patch.js

```javascript
import { Message } from "@mail/core/common/message_model";
import { Record } from "@mail/core/common/record";

import { patch } from "@web/core/utils/patch";
import { SESSION_STATE } from "./livechat_service";

patch(Message.prototype, {
    setup() {
        super.setup();
        this.chatbotStep = Record.one("ChatbotStep", { inverse: "message" });
    },
    canAddReaction(thread) {
        return (
            super.canAddReaction(thread) &&
            (thread?.channel_type !== "livechat" ||
                this.store.env.services["im_livechat.livechat"].state === SESSION_STATE.PERSISTED)
        );
    },
    canReplyTo(thread) {
        return (
            super.canReplyTo(thread) &&
            (thread?.channel_type !== "livechat" ||
                this.store.env.services["im_livechat.chatbot"].inputEnabled)
        );
    },
});

```

## File: static\src\embed\common\message_patch.js

```javascript
import { Message } from "@mail/core/common/message";

import { patch } from "@web/core/utils/patch";
import { url } from "@web/core/utils/urls";

Message.props.push("isTypingMessage?");

patch(Message.prototype, {
    setup() {
        super.setup();
        this.url = url;
    },

    get quickActionCount() {
        return this.props.thread?.channel_type === "livechat" ? 3 : super.quickActionCount;
    },

    /**
     * @param {import("@im_livechat/embed/common/chatbot/chatbot_step_model").StepAnswer} answer
     */
    answerChatbot(answer) {
        return this.props.message.thread.post(answer.name, {}, { selected_answer_id: answer.id });
    },
});

```

## File: static\src\embed\common\message_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.Message" t-inherit-mode="extension">
        <xpath expr="//*[@t-ref='messageContent']" position="replace">
            <div t-if="props.isTypingMessage">
                <img height="30" t-att-src="url('/im_livechat/static/src/img/chatbot_is_typing.gif')"/>
            </div>
            <t t-else="">$0</t>
        </xpath>
        <xpath expr="//*[hasclass('o-mail-Message-textContent')]" position="after">
            <ul class="p-0 m-0" t-if="props.message.chatbotStep?.answers and !props.message.chatbotStep.selectedAnswer">
                <li
                    t-foreach="props.message.chatbotStep?.answers" t-as="answer" t-key="answer.id"
                    t-esc="answer.name" t-on-click="() => this.answerChatbot(answer)"
                    class="btn btn-outline-primary d-block mt-2 py-2"
                />
            </ul>
        </xpath>
        <xpath expr="//*[hasclass('o-mail-NotificationMessage')]" position="attributes">
            <attribute name="t-attf-class" add="{{ props.thread.channel_type === 'livechat' ? 'o-livechat-NoPinMenu' : '' }}" separator=" "/>
        </xpath>
    </t>
</templates>

```

## File: static\src\embed\common\misc.js

```javascript
export function isValidEmail(val) {
    // http://stackoverflow.com/questions/46155/validate-email-address-in-javascript
    const re =
        /^(([^<>()[\].,;:\s@"]+(\.[^<>()[\].,;:\s@"]+)*)|(".+"))@(([^<>()[\].,;:\s@"]+\.)+[^<>()[\].,;:\s@"]{2,})$/i;
    return re.test(val);
}

```

## File: static\src\embed\common\store_service_patch.js

```javascript
import { SESSION_STATE } from "@im_livechat/embed/common/livechat_service";

import { Store, storeService } from "@mail/core/common/store_service";

import { patch } from "@web/core/utils/patch";

storeService.dependencies.push("im_livechat.initialized");

patch(Store.prototype, {
    async initialize() {
        const livechatInitialized = this.env.services["im_livechat.initialized"];
        await livechatInitialized.ready;
        const livechatService = this.env.services["im_livechat.livechat"];
        if (livechatService.state === SESSION_STATE.PERSISTED) {
            await super.initialize();
            livechatService.thread ??= this.store.Thread.get({
                id: livechatService.savedState?.store["discuss.channel"][0].id,
                model: "discuss.channel",
            });
            if (!livechatService.thread) {
                livechatService.leave({ notifyServer: false });
            }
            return;
        }
        if (livechatService.savedState?.store) {
            const { Thread = [] } = this.store.insert(livechatService.savedState.store);
            livechatService.thread = Thread[0];
        }
        this.isReady.resolve();
    },
    get initMessagingParams() {
        const params = super.initMessagingParams;
        params.init_messaging.channel_types = ["livechat"];
        return params;
    },
});

```

## File: static\src\embed\common\thread_actions.js

```javascript
import { SESSION_STATE } from "@im_livechat/embed/common/livechat_service";

import { threadActionsRegistry } from "@mail/core/common/thread_actions";
import "@mail/discuss/call/common/thread_actions";
import { useComponent } from "@odoo/owl";

import { _t } from "@web/core/l10n/translation";
import { useService } from "@web/core/utils/hooks";
import { patch } from "@web/core/utils/patch";

threadActionsRegistry.add("restart", {
    condition(component) {
        return component.chatbotService.canRestart;
    },
    icon: "fa fa-fw fa-refresh",
    name: _t("Restart Conversation"),
    open(component) {
        component.chatbotService.restart();
        component.props.chatWindow.open();
    },
    sequence: 99,
    sequenceQuick: 15,
});

const callSettingsAction = threadActionsRegistry.get("settings");
patch(callSettingsAction, {
    condition(component) {
        if (component.thread?.channel_type !== "livechat") {
            return super.condition(...arguments);
        }
        return (
            component.livechatService.state === SESSION_STATE.PERSISTED &&
            component.rtcService.state.channel?.eq(component.thread)
        );
    },
    setup() {
        super.setup(...arguments);
        const component = useComponent();
        component.livechatService = useService("im_livechat.livechat");
        component.rtcService = useService("discuss.rtc");
    },
});

```

## File: static\src\embed\common\thread_model_patch.js

```javascript
import { Record } from "@mail/core/common/record";
import { Thread } from "@mail/core/common/thread_model";

import { patch } from "@web/core/utils/patch";
import { SESSION_STATE } from "./livechat_service";

patch(Thread.prototype, {
    setup() {
        super.setup();
        this.chatbotTypingMessage = Record.one("Message", {
            compute() {
                if (this.chatbot) {
                    return { id: -0.1 - this.id, thread: this, author: this.operator };
                }
            },
        });
        this.livechatWelcomeMessage = Record.one("Message", {
            compute() {
                if (this.hasWelcomeMessage) {
                    const livechatService = this.store.env.services["im_livechat.livechat"];
                    return {
                        id: -0.2 - this.id,
                        body: livechatService.options.default_message,
                        thread: this,
                        author: this.operator,
                    };
                }
            },
        });
        this.chatbot = Record.one("Chatbot");
        this.livechat_active;
        this._startChatbot = Record.attr(false, {
            compute() {
                return (
                    this.chatbot?.thread?.eq(
                        this.store.env.services["im_livechat.livechat"].thread
                    ) && this.isLoaded
                );
            },
            onUpdate() {
                if (this._startChatbot) {
                    this.store.env.services["im_livechat.chatbot"].start();
                }
            },
        });
        this.requested_by_operator = false;
    },

    get isLastMessageFromCustomer() {
        return this.newestPersistentOfAllMessage?.isSelfAuthored;
    },

    get membersThatCanSeen() {
        return super.membersThatCanSeen.filter((member) => !member.is_bot);
    },

    get avatarUrl() {
        if (this.channel_type === "livechat") {
            return this.operator.avatarUrl;
        }
        return super.avatarUrl;
    },
    get displayName() {
        if (this.channel_type === "livechat" && this.operator) {
            return this.operator.user_livechat_username || this.operator.name;
        }
        return super.displayName;
    },
    get hasWelcomeMessage() {
        return this.channel_type === "livechat" && !this.chatbot && !this.requested_by_operator;
    },
    /** @returns {Promise<import("models").Message} */
    async post() {
        if (
            this.channel_type === "livechat" &&
            this.store.env.services["im_livechat.livechat"].state !== SESSION_STATE.PERSISTED
        ) {
            const thread = await this.store.env.services["im_livechat.livechat"].persist();
            if (!thread) {
                return;
            }
            return thread.post(...arguments);
        }
        const message = await super.post(...arguments);
        this.store.env.services["im_livechat.chatbot"].bus.trigger("MESSAGE_POST", message);
        return message;
    },

    get showUnreadBanner() {
        if (this.chatbot && !this.chatbot.currentStep?.operatorFound) {
            return false;
        }
        return super.showUnreadBanner;
    },
});

```

## File: static\src\embed\common\thread_patch.js

```javascript
import { Thread } from "@mail/core/common/thread";

import { useState } from "@odoo/owl";

import { useService } from "@web/core/utils/hooks";
import { patch } from "@web/core/utils/patch";

patch(Thread.prototype, {
    setup() {
        super.setup();
        this.chatbotService = useState(useService("im_livechat.chatbot"));
    },
});

```

## File: static\src\embed\common\thread_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.Thread" t-inherit-mode="extension">
        <xpath expr="//*[@name='content']" position="before">
            <div t-if="props.thread?.livechatWelcomeMessage" class="bg-100 py-3">
                <Message message="props.thread.livechatWelcomeMessage" hasActions="false" thread="props.thread" className="'px-3'"/>
            </div>
        </xpath>
        <xpath expr="(//div[hasclass('o-mail-Thread')])" position="inside">
            <Message t-if="props.thread.chatbot?.typingMessage" message="props.thread.chatbot.typingMessage" hasActions="false" isInChatWindow="env.inChatWindow" isTypingMessage="true"  thread="props.thread"/>
        </xpath>
        <xpath expr="//*[hasclass('o-mail-Thread-empty')]" position="replace">
            <t t-if="props.thread.channel_type !== 'livechat'">$0</t>
        </xpath>
        <xpath expr="//*[hasclass('o-mail-Thread-newMessage')]" position="replace">
            <t t-if="!chatbotService.chatbot">$0</t>
        </xpath>
    </t>
</templates>

```

## File: static\src\embed\common\@types\models.d.ts

```ts
declare module "models" {
    import { ChatbotScriptStep as ChatbotScriptStepClass } from "@im_livechat/embed/common/chatbot/chatbot_script_step_model";
    import { ChatbotStep as ChatbotStepClass } from "@im_livechat/embed/common/chatbot/chatbot_step_model";
    import { Chatbot as ChatbotClass } from "@im_livechat/embed/common/chatbot/chatbot_model";
    import { ChatbotScriptStepAnswer as ChatbotScriptStepAnswerClass } from "@im_livechat/embed/common/chatbot/chatbot_script_step_answer_model";
    import { ChatbotScript as ChatbotScriptClass } from "@im_livechat/embed/common/chatbot/chatbot_script_model";
    import { LivechatRule as LivechatRuleClass } from "@im_livechat/embed/common/livechat/livechat_rule_model";

    export interface ChatbotScriptStep extends ChatbotScriptStepClass { }
    export interface ChatbotStep extends ChatbotStepClass { }
    export interface Chatbot extends ChatbotClass { }
    export interface ChatbotScriptStepAnswer extends ChatbotScriptStepAnswerClass { }
    export interface ChatbotScript extends ChatbotScriptClass { }
    export interface LivechatRule extends LivechatRuleClass { }

    export interface ChatWindpw {
        hasFeedbackPanel: boolean,
    }

    export interface Message {
        chatbotStep: ChatbotStep,
    }

    export interface Thread {
        livechatWelcomeMessage: Message,
        chatbot: Chatbot,
        requested_by_operator: boolean,
    }

    export interface Models {
        "ChatbotScriptStep": ChatbotScriptStep,
        "ChatbotStep": ChatbotStep,
        "Chatbot": Chatbot,
        "ChatbotScriptStepAnswer": ChatbotScriptStepAnswer,
        "ChatbotScript": ChatbotScript,
        "LivechatRule": LivechatRule,
    }
}

```

## File: static\src\embed\common\chatbot\chatbot_model.js

```javascript
import { AND, Record } from "@mail/core/common/record";
import { rpc } from "@web/core/network/rpc";
import { browser } from "@web/core/browser/browser";
import { debounce } from "@web/core/utils/timing";
import { expirableStorage } from "../expirable_storage";

export class Chatbot extends Record {
    static id = AND("script", "thread");
    static MESSAGE_DELAY = 1500;
    // Time to wait without user input before considering a multi line step as
    // completed.
    static MULTILINE_STEP_DEBOUNCE_DELAY = 10000;

    isTyping = false;
    script = Record.one("ChatbotScript");
    currentStep = Record.one("ChatbotStep");
    steps = Record.many("ChatbotStep");
    thread = Record.one("Thread", {
        inverse: "chatbot",
        onDelete() {
            this.delete();
        },
    });
    typingMessage = Record.one("Message", {
        compute() {
            if (this.isTyping && this.thread) {
                return {
                    id: -0.1 - this.thread.id,
                    thread: this.thread,
                    author: this.script.partner,
                };
            }
        },
    });
    /**
     * @type {(message: import("models").Message) => Promise<void>}
     */
    _processAnswerDebounced = Record.attr(null, {
        compute() {
            return debounce(
                this._processAnswer,
                this.script.isLivechatTourRunning ? 500 : Chatbot.MULTILINE_STEP_DEBOUNCE_DELAY
            );
        },
    });

    /**
     * @param {import("models").Message} message
     */
    async processAnswer(message) {
        if (this.thread.notEq(message.thread) || !this.currentStep?.expectAnswer) {
            return;
        }
        if (this.currentStep.type === "free_input_multi") {
            await this._processAnswerDebounced(message);
        }
        await this._processAnswer(message);
    }

    async triggerNextStep() {
        if (this.currentStep) {
            await this._simulateTyping();
        }
        await this._goToNextStep();
        if (!this.currentStep || this.currentStep.completed || !this.thread) {
            return;
        }
        if (this.thread.isTransient) {
            // Thread is not persisted thus messages do not exist on the server,
            // create them now on the client side.
            this.currentStep.message = this.store.Message.insert(
                {
                    id: this.store.getNextTemporaryId(),
                    author: this.script.partner,
                    body: this.currentStep.scriptStep.message,
                    thread: this.thread,
                },
                { html: true }
            );
        }
        this.thread.messages.add(this.currentStep.message);
    }

    get completed() {
        return (
            (this.currentStep?.isLast &&
                (!this.currentStep.expectAnswer || this.currentStep?.completed)) ||
            this.currentStep?.operatorFound ||
            !this.thread.livechat_active
        );
    }

    /**
     * Go to the next step of the chatbot, fetch it if needed.
     */
    async _goToNextStep() {
        if (!this.thread || this.currentStep?.isLast) {
            return;
        }
        if (this.steps.at(-1)?.eq(this.currentStep)) {
            const storeData = await rpc("/chatbot/step/trigger", {
                channel_id: this.thread.id,
                chatbot_script_id: this.script.id,
            });
            if (!storeData) {
                this.currentStep.isLast = true;
                return;
            }
            const { ChatbotStep: steps } = this.store.insert(storeData, { html: true });
            this.steps.push(steps[0]);
        } else {
            const nextStepIndex = this.steps.lastIndexOf(this.currentStep) + 1;
            this.currentStep = this.steps[nextStepIndex];
        }
    }

    /**
     * Simulate the typing of the chatbot.
     */
    async _simulateTyping() {
        this.isTyping = true;
        await new Promise((res) =>
            setTimeout(() => {
                this.isTyping = false;
                res();
            }, Chatbot.MESSAGE_DELAY)
        );
    }

    async _processAnswer(message) {
        let stepCompleted = true;
        if (this.currentStep.type === "question_email") {
            stepCompleted = await this._processAnswerQuestionEmail();
        } else if (this.currentStep.type === "question_selection") {
            stepCompleted = await this._processAnswerQuestionSelection(message);
        }
        this.currentStep.completed = stepCompleted;
    }

    /**
     * Process the user answer for a question selection step.
     *
     * @param {import("models").Message} message Answer posted by the user.
     * @returns {Promise<boolean>} Whether the script is ready to go to the next step.
     */
    async _processAnswerQuestionSelection(message) {
        const answer = this.currentStep.selectedAnswer;
        if (!answer.redirect_link) {
            return true;
        }
        let isRedirecting = false;
        if (answer.redirect_link && URL.canParse(answer.redirect_link, window.location.href)) {
            const url = new URL(window.location.href);
            const nextURL = new URL(answer.redirect_link, window.location.href);
            isRedirecting = url.pathname !== nextURL.pathname || url.origin !== nextURL.origin;
        }
        const redirects = JSON.parse(
            expirableStorage.getItem("im_livechat.chatbot_redirect") ?? "[]"
        );
        const targetURL = new URL(answer.redirect_link, window.location.origin);
        const redirectionAlreadyDone =
            targetURL.href === location.href || redirects.includes(message.id);
        redirects.push(message.id);
        const ONE_DAY_TTL = 60 * 60 * 24;
        expirableStorage.setItem(
            "im_livechat.chatbot_redirect",
            JSON.stringify([...new Set(redirects)]),
            ONE_DAY_TTL
        );
        if (!redirectionAlreadyDone) {
            browser.location.assign(answer.redirect_link);
        }
        return redirectionAlreadyDone || !isRedirecting;
    }

    /**
     * Process the user answer for a question email step.
     *
     * @returns {Promise<boolean>} Whether the script is ready to go to the next step.
     */
    async _processAnswerQuestionEmail() {
        const { success, data } = await rpc("/chatbot/step/validate_email", {
            channel_id: this.thread.id,
        });
        const { Message: messages = [] } = this.store.insert(data, { html: true });
        const [message] = messages;
        if (message) {
            this.thread.messages.add(message);
        }
        return success;
    }

    /**
     * Restart the chatbot script.
     */
    restart() {
        if (this.currentStep) {
            this.currentStep.isLast = false;
            this.thread.livechat_active = true;
        }
    }
}
Chatbot.register();

```

## File: static\src\embed\common\chatbot\chatbot_script_model.js

```javascript
import { Record } from "@mail/core/common/record";

export class ChatbotScript extends Record {
    static id = "id";

    /** @type {number} */
    id;
    /** @type {string} */
    name;
    isLivechatTourRunning = false;
    partner = Record.one("Persona");
    welcomeSteps = Record.many("chatbot.script.step");
}
ChatbotScript.register();

```

## File: static\src\embed\common\chatbot\chatbot_script_step_answer_model.js

```javascript
import { Record } from "@mail/core/common/record";

export class ChatbotScriptStepAnswer extends Record {
    static id = "id";
    static _name = "chatbot.script.answer";

    /** @type {number} */
    id;
    /** @type {string} */
    label;
    /** @type {string|false} */
    redirect_link;
}
ChatbotScriptStepAnswer.register();

```

## File: static\src\embed\common\chatbot\chatbot_script_step_model.js

```javascript
import { Record } from "@mail/core/common/record";

export class ChatbotScriptStep extends Record {
    static id = "id";
    static _name = "chatbot.script.step";

    /** @type {number} */
    id;
    /** @type {string} */
    message;
    /** @type {"free_input_multi"|"free_input_single"|"question_email"|"question_phone"|"question_selection"|"text"|"forward_operator"} */
    type;
    isLast = false;
    answers = Record.many("chatbot.script.answer");
}
ChatbotScriptStep.register();

```

## File: static\src\embed\common\chatbot\chatbot_service.js

```javascript
import { SESSION_STATE } from "@im_livechat/embed/common/livechat_service";
import { rpc } from "@web/core/network/rpc";

import { EventBus, reactive } from "@odoo/owl";

import { browser } from "@web/core/browser/browser";
import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";

const STEP_DELAY = 500;
export class ChatBotService {
    /** @type {number} */
    nextStepTimeout;

    constructor(env, services) {
        const self = reactive(this);
        self.setup(env, services);
        return self;
    }

    /**
     * @param {import("@web/env").OdooEnv} env
     * @param {{
     * "im_livechat.livechat": import("@im_livechat/embed/common/livechat_service").LivechatService,
     * "mail.store": import("@mail/core/common/store_service").Store,
     * }} services
     */
    setup(env, services) {
        this.env = env;
        this.bus = new EventBus();
        this.livechatService = services["im_livechat.livechat"];
        this.store = services["mail.store"];
        this.livechatService.onStateChange(SESSION_STATE.NONE, () => this.stop());
        this.bus.addEventListener("MESSAGE_POST", async ({ detail: message }) => {
            await this.chatbot?.processAnswer(message);
            if (this.chatbot?.currentStep.completed) {
                this._triggerNextStep();
            }
        });
    }

    /**
     * Start the chatbot script.
     */
    async start() {
        if (this.chatbot.thread.isLastMessageFromCustomer && !this.chatbot.completed) {
            await this.chatbot?.processAnswer(
                this.livechatService.thread.newestPersistentOfAllMessage
            );
        }
        if (!this.chatbot.currentStep?.expectAnswer || this.chatbot.currentStep?.completed) {
            this._triggerNextStep();
        }
    }

    /**
     * Stop the chatbot script.
     */
    stop() {
        clearTimeout(this.nextStepTimeout);
    }

    /**
     * Restart the chatbot script if it was completed and post the
     * restart message.
     */
    async restart() {
        if (!this.chatbot?.completed) {
            return;
        }
        const data = await rpc("/chatbot/restart", {
            channel_id: this.chatbot.thread.id,
            chatbot_script_id: this.chatbot.script.id,
        });
        const { Message: messages = [] } = this.store.insert(data, { html: true });
        if (!this.livechatService.thread) {
            return;
        }
        const [message] = messages;
        this.livechatService.thread.messages.add(message);
        this.chatbot.restart();
        this._triggerNextStep();
    }

    // =============================================================================
    // SCRIPT PROCESSING
    // =============================================================================

    /**
     * Trigger the next step of the script recursivly until the script is
     * completed or the current step expects an answer from the user.
     */
    async _triggerNextStep() {
        if (this.chatbot.completed) {
            return;
        }
        await this.chatbot.triggerNextStep();
        if (!this.chatbot?.currentStep) {
            return;
        }
        if (this.chatbot.currentStep.expectAnswer || this.chatbot.currentStep.isLast) {
            return;
        }
        this.nextStepTimeout = browser.setTimeout(async () => this._triggerNextStep(), STEP_DELAY);
    }

    // =============================================================================
    // GETTERS
    // =============================================================================

    get canRestart() {
        return this.chatbot?.completed && !this.chatbot.currentStep?.operatorFound;
    }

    get inputEnabled() {
        if (!this.chatbot || this.chatbot.currentStep?.operatorFound) {
            return true;
        }
        return (
            !this.chatbot.currentStep?.completed &&
            !this.isTyping &&
            this.chatbot.currentStep?.expectAnswer &&
            this.chatbot.currentStep?.answers.length === 0
        );
    }

    get inputDisabledText() {
        if (this.inputEnabled) {
            return "";
        }
        if (this.chatbot.completed) {
            return _t("Conversation ended...");
        }
        if (
            this.chatbot.currentStep?.type === "question_selection" &&
            !this.chatbot.currentStep.completed
        ) {
            return _t("Select an option above");
        }
        return _t("Say something");
    }

    get chatbot() {
        return this.livechatService.thread?.chatbot;
    }
}

export const chatBotService = {
    dependencies: ["im_livechat.livechat", "mail.store"],
    start(env, services) {
        return new ChatBotService(env, services);
    },
};
registry.category("services").add("im_livechat.chatbot", chatBotService);

```

## File: static\src\embed\common\chatbot\chatbot_step_model.js

```javascript
import { AND, Record } from "@mail/core/common/record";

export class ChatbotStep extends Record {
    static id = AND("scriptStep", "message");

    operatorFound = false;
    scriptStep = Record.one("chatbot.script.step");
    message = Record.one("Message", { inverse: "chatbotStep" });
    answers = Record.many("chatbot.script.answer", {
        compute() {
            return this.scriptStep?.answers;
        },
    });
    selectedAnswer = Record.one("chatbot.script.answer");
    type = Record.attr("", {
        compute() {
            return this.scriptStep?.type;
        },
    });
    isLast = false;

    get expectAnswer() {
        return [
            "free_input_multi",
            "free_input_single",
            "question_selection",
            "question_email",
            "question_phone",
        ].includes(this.type);
    }
}
ChatbotStep.register();

```

## File: static\src\embed\common\feedback_panel\feedback_panel.js

```javascript
import { RATING } from "@im_livechat/embed/common/livechat_service";
import { TranscriptSender } from "@im_livechat/embed/common/feedback_panel/transcript_sender";

import { Component, useState } from "@odoo/owl";

import { useService } from "@web/core/utils/hooks";
import { session } from "@web/session";
import { url } from "@web/core/utils/urls";
import { rpc } from "@web/core/network/rpc";

/**
 * @typedef {Object} Props
 * @property {Function} [onClickClose]
 * @property {import("models").Thread}
 * @extends {Component<Props, Env>}
 */
export class FeedbackPanel extends Component {
    static template = "im_livechat.FeedbackPanel";
    static props = ["onClickClose?", "thread"];
    static components = { TranscriptSender };

    STEP = Object.freeze({
        RATING: "rating",
        THANKS: "thanks",
    });
    RATING = RATING;

    setup() {
        this.session = session;
        this.livechatService = useService("im_livechat.livechat");
        this.state = useState({
            step: this.STEP.RATING,
            rating: null,
            feedback: "",
        });
        this.url = url;
    }

    /**
     * @param {number} rating
     */
    select(rating) {
        this.state.rating = rating;
    }

    onClickSendFeedback() {
        rpc("/im_livechat/feedback", {
            reason: this.state.feedback,
            rate: this.state.rating,
            channel_id: this.props.thread.id,
        });
        this.state.step = this.STEP.THANKS;
    }
}

```

## File: static\src\embed\common\feedback_panel\feedback_panel.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

<t t-name="im_livechat.FeedbackPanel">
<div class="d-flex flex-column bg-view flex-grow-1 p-3">
    <div class="p-2">
        <div class="mb-5">
            <t t-if="state.step === STEP.RATING">
                <p class="text-center fs-6 mb-4">Did we correctly answer your question?</p>
                <div class="d-flex justify-content-center">
                    <img role="button" class="mx-3 opacity-50 opacity-100-hover" t-att-class="{ 'opacity-100': state.rating === RATING.GOOD }" t-att-src="url(`/rating/static/src/img/rating_${RATING.GOOD}.png`)" t-att-alt="RATING.GOOD" t-on-click="() => this.select(RATING.GOOD)"/>
                    <img role="button" class="mx-3 opacity-50 opacity-100-hover" t-att-class="{ 'opacity-100': state.rating === RATING.OK }" t-att-src="url(`/rating/static/src/img/rating_${RATING.OK}.png`)" t-att-alt="RATING.OK"  t-on-click="() => this.select(RATING.OK)"/>
                    <img role="button" class="mx-3 opacity-50 opacity-100-hover" t-att-class="{ 'opacity-100': state.rating === RATING.BAD }" t-att-src="url(`/rating/static/src/img/rating_${RATING.BAD}.png`)" t-att-alt="RATING.BAD" t-on-click="() => this.select(RATING.BAD)"/>
                </div>
            </t>
            <t t-else="">
                <p class="text-center fs-5 fw-bold mb-4">Thank you for your feedback</p>
            </t>
        </div>
        <div t-if="state.rating and state.step === STEP.RATING" class="d-flex flex-column mb-5">
            <textarea t-model="state.feedback" class="form-control my-2" placeholder="Explain your note"/>
            <button class="btn btn-primary align-self-end" t-on-click="onClickSendFeedback">Send</button>
        </div>
        <div class="mb-5">
            <TranscriptSender thread="props.thread"/>
        </div>
        <button class="btn btn-link text-muted text-decoration-underline fw-normal w-100" t-on-click="props.onClickClose">Close conversation</button>
    </div>
</div>
</t>
</templates>

```

## File: static\src\embed\common\feedback_panel\transcript_sender.js

```javascript
import { isValidEmail } from "@im_livechat/embed/common/misc";
import { rpc } from "@web/core/network/rpc";

import { Component, useState } from "@odoo/owl";

import { useService } from "@web/core/utils/hooks";

/**
 * @typedef {Object} Props
 * @property {import("models").Thread}
 * @extends {Component<Props, Env>}
 */
export class TranscriptSender extends Component {
    static template = "im_livechat.TranscriptSender";
    static props = ["thread"];

    STATUS = Object.freeze({
        IDLE: "idle",
        SENDING: "sending",
        SENT: "sent",
        FAILED: "failed",
    });

    setup() {
        this.isValidEmail = isValidEmail;
        this.livechatService = useService("im_livechat.livechat");
        this.state = useState({
            email: "",
            status: this.STATUS.IDLE,
        });
    }

    async onClickSend() {
        this.state.status = this.STATUS.SENDING;
        try {
            await rpc("/im_livechat/email_livechat_transcript", {
                channel_id: this.props.thread.id,
                email: this.state.email,
            });
            this.state.status = this.STATUS.SENT;
        } catch {
            this.state.status = this.STATUS.FAILED;
        }
    }
}

```

## File: static\src\embed\common\feedback_panel\transcript_sender.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

<t t-name="im_livechat.TranscriptSender">
    <div class="form-text">
        <t t-if="state.status === STATUS.SENT">The conversation was sent.</t>
        <t t-elif="state.status === STATUS.FAILED">An error occurred. Please try again.</t>
        <t t-else="">Receive a copy of this conversation.</t>
    </div>
    <div class="input-group has-validation mb-3">
        <input t-model="state.email" t-att-disabled="[STATUS.SENDING, STATUS.SENT].includes(state.status)" type="text" class="form-control" placeholder="mail@example.com"/>
        <button class="btn btn-primary" type="button" data-action="sendTranscript" t-att-disabled="!state.email or !isValidEmail(state.email) or [STATUS.SENDING, STATUS.SENT].includes(state.status)" t-on-click="onClickSend">
            <i class="fa" t-att-class="{
                'fa-spinner fa-spin': state.status === STATUS.SENDING,
                'fa-check': state.status === STATUS.SENT,
                'fa-paper-plane': state.status === STATUS.IDLE,
                'fa-repeat': state.status === STATUS.FAILED,
            }"/>
        </button>
    </div>
</t>
</templates>

```

## File: static\src\embed\cors\attachment_model_patch.js

```javascript
import { Attachment } from "@mail/core/common/attachment_model";
import { patch } from "@web/core/utils/patch";

patch(Attachment.prototype, {
    get urlQueryParams() {
        return {
            ...super.urlQueryParams,
            guest_token: this.store.env.services["im_livechat.livechat"].guestToken,
        };
    },
    get urlRoute() {
        if (!this.access_token && this.thread?.model === "discuss.channel") {
            return this.isImage
                ? `/im_livechat/cors/channel/${this.thread.id}/image/${this.id}`
                : `/im_livechat/cors/channel/${this.thread.id}/attachment/${this.id}`;
        }
        return super.urlRoute;
    },
});

```

## File: static\src\embed\cors\attachment_upload_service_patch.js

```javascript
import { AttachmentUploadService } from "@mail/core/common/attachment_upload_service";

import { patch } from "@web/core/utils/patch";
import { url } from "@web/core/utils/urls";

patch(AttachmentUploadService.prototype, {
    getUploadURL(thread) {
        if (thread.channel_type === "livechat") {
            return url("/im_livechat/cors/attachment/upload");
        }
        return super.getUploadURL(...arguments);
    },

    _buildFormData() {
        const formData = super._buildFormData(...arguments);
        formData.append("guest_token", this.env.services["im_livechat.livechat"].guestToken);
    },
});

```

## File: static\src\embed\cors\boot.js

```javascript
import { getGuestToken } from "@im_livechat/embed/common/livechat_service";
import { livechatRoutingMap } from "@im_livechat/embed/cors/livechat_routing_map";

import { browser } from "@web/core/browser/browser";
import { rpc } from "@web/core/network/rpc";
import { registry } from "@web/core/registry";
import { session } from "@web/session";

(async function boot() {
    const { fetch } = browser;
    browser.fetch = function (url, ...args) {
        if (!url.match(/^(?:https?:)?\/\//)) {
            url = session.origin + url;
        }
        return fetch(url, ...args);
    };

    // Override rpc to forward requests to CORS-allowed routes.
    // The "guest_token" will be appended to the request parameters for authentication.
    const originalRPC = rpc._rpc;
    rpc._rpc = function (route, params, settings) {
        if (route in livechatRoutingMap.content) {
            route = livechatRoutingMap.get(route, route);
            const guestToken = getGuestToken();
            if (guestToken) {
                params = {
                    ...params,
                    guest_token: guestToken,
                };
            }
        }
        if (!route.match(/^(?:https?:)?\/\//)) {
            route = session.origin + route;
        }
        return originalRPC(route, params, settings);
    };
    // Remove the error service: it fails to identify issues within the shadow
    // DOM of the live chat and causes disruption for pages that embed it by
    // displaying pop-ups for errors outside of its scope.
    registry.category("services").remove("error");
})();

```

## File: static\src\embed\cors\livechat_routing_map.js

```javascript
import { registry } from "@web/core/registry";

/**
 * This routing map is used to redirect the requests made by the livechat to
 * dedicated CORS-allowed routes. Every route expected to be called by the
 * livechat should be added here. Note that this will only be used if the
 * livechat is loaded from a different origin than the Odoo server.
 *
 * @see /im_livechat/embed/cors/boot.js
 */
export const livechatRoutingMap = registry.category("discuss.routing_map");

livechatRoutingMap
    .add("/discuss/channel/messages", "/im_livechat/cors/channel/messages")
    .add("/discuss/channel/notify_typing", "/im_livechat/cors/channel/notify_typing")
    .add("/discuss/channel/mark_as_read", "/im_livechat/cors/channel/mark_as_read")
    .add("/discuss/channel/fold", "/im_livechat/cors/channel/fold")
    .add("/mail/attachment/delete", "/im_livechat/cors/attachment/delete")
    .add("/discuss/channel/ping", "/im_livechat/cors/channel/ping")
    .add("/mail/action", "/im_livechat/cors/action")
    .add("/mail/data", "/im_livechat/cors/data")
    .add("/mail/link_preview", "/im_livechat/cors/link_preview")
    .add("/mail/link_preview/hide", "/im_livechat/cors/link_preview/hide")
    .add("/mail/message/post", "/im_livechat/cors/message/post")
    .add("/mail/message/reaction", "/im_livechat/cors/message/reaction")
    .add("/mail/message/update_content", "/im_livechat/cors/message/update_content")
    .add("/mail/rtc/channel/join_call", "/im_livechat/cors/rtc/channel/join_call")
    .add("/mail/rtc/channel/leave_call", "/im_livechat/cors/rtc/channel/leave_call")
    .add(
        "/mail/rtc/session/notify_call_members",
        "/im_livechat/cors/rtc/session/notify_call_members"
    )
    .add(
        "/mail/rtc/session/update_and_broadcast",
        "/im_livechat/cors/rtc/session/update_and_broadcast"
    )
    .add("/im_livechat/visitor_leave_session", "/im_livechat/cors/visitor_leave_session")
    .add("/im_livechat/get_session", "/im_livechat/cors/get_session")
    .add("/im_livechat/init", "/im_livechat/cors/init")
    .add("/im_livechat/feedback", "/im_livechat/cors/feedback")
    .add("/im_livechat/history", "/im_livechat/cors/history")
    .add("/im_livechat/email_livechat_transcript", "/im_livechat/cors/email_livechat_transcript")
    .add("/chatbot/restart", "/chatbot/cors/restart")
    .add("/chatbot/answer/save", "/chatbot/cors/answer/save")
    .add("/chatbot/step/trigger", "/chatbot/cors/step/trigger")
    .add("/chatbot/step/validate_email", "/chatbot/cors/step/validate_email");

```

## File: static\src\embed\cors\persona_model_patch.js

```javascript
import { Persona } from "@mail/core/common/persona_model";
import { assignDefined } from "@mail/utils/common/misc";

import { patch } from "@web/core/utils/patch";
import { url } from "@web/core/utils/urls";

patch(Persona.prototype, {
    get avatarUrl() {
        const params = assignDefined(
            {},
            {
                guest_token: this.store.env.services["im_livechat.livechat"].guestToken,
                unique: this.write_date,
            }
        );
        if (!this.store.self.isInternalUser) {
            params.access_token = this.avatar_128_access_token;
        }
        if (this.type === "partner") {
            return url("/im_livechat/cors/web/image", {
                field: "avatar_128",
                id: this.id,
                model: "res.partner",
                ...params,
            });
        }
        if (this.type === "guest") {
            return url("/im_livechat/cors/web/image", {
                field: "avatar_128",
                id: this.id,
                model: "mail.guest",
                ...params,
            });
        }
        return super.avatarUrl;
    },
});

```

## File: static\src\embed\cors\store_service_patch.js

```javascript
import { Store } from "@mail/core/common/store_service";

import { patch } from "@web/core/utils/patch";
import { url } from "@web/core/utils/urls";

patch(Store.prototype, {
    getAvatarUrl(persona, thread) {
        if (thread?.model === "discuss.channel" && persona.notEq(thread.operator)) {
            const route =
                persona.type === "partner"
                    ? `/im_livechat/cors/channel/${thread.id}/partner/${persona.id}/avatar_128`
                    : `/im_livechat/cors/channel/${thread.id}/guest/${persona.id}/avatar_128`;
            return url(route, {
                guest_token: this.env.services["im_livechat.livechat"].guestToken,
            });
        }
        return super.avatarUrl(...arguments);
    },
});

```

## File: static\src\embed\external\boot.js

```javascript
import { LivechatButton } from "@im_livechat/embed/common/livechat_button";
import { makeShadow, makeRoot } from "@im_livechat/embed/common/boot_helpers";

import { mount, whenReady } from "@odoo/owl";

import { _t } from "@web/core/l10n/translation";
import { getTemplate } from "@web/core/templates";
import { MainComponentsContainer } from "@web/core/main_components_container";
import { Deferred } from "@web/core/utils/concurrency";
import { registry } from "@web/core/registry";
import { makeEnv, startServices } from "@web/env";
import { session } from "@web/session";

odoo.livechatReady = new Deferred();

(async function boot() {
    session.origin = session.livechatData.serverUrl;
    await whenReady();
    const mainComponentsRegistry = registry.category("main_components");
    mainComponentsRegistry.add("LivechatRoot", { Component: LivechatButton });
    const env = Object.assign(makeEnv(), { embedLivechat: true });
    await startServices(env);
    odoo.isReady = true;
    const target = await makeShadow(makeRoot(document.body));
    await mount(MainComponentsContainer, target, {
        env,
        getTemplate,
        translateFn: _t,
        dev: env.debug,
    });
    odoo.livechatReady.resolve();
})();

```

## File: static\src\embed\external\bus_parameters_service_patch.js

```javascript
import { busParametersService } from "@bus/bus_parameters_service";

import { patch } from "@web/core/utils/patch";
import { session } from "@web/session";

patch(busParametersService, {
    start() {
        return {
            ...super.start(...arguments),
            serverURL: session.livechatData.serverUrl.replace(/\/+$/, ""),
        };
    },
});

```

## File: static\src\embed\external\emoji_loader_patch.js

```javascript
import { loader } from "@web/core/emoji_picker/emoji_picker";

import { loadJS } from "@web/core/assets";
import { patch } from "@web/core/utils/patch";
import { url } from "@web/core/utils/urls";
import { session } from "@web/session";

patch(loader, {
    loadEmoji: () =>
        loadJS(
            url("/im_livechat/emoji_bundle", undefined, {
                origin: session.livechatData.serverUrl,
            })
        ),
});

```

## File: static\src\embed\frontend\boot_service.js

```javascript
import { makeRoot, makeShadow } from "@im_livechat/embed/common/boot_helpers";
import { LivechatRoot } from "@im_livechat/embed/frontend/livechat_root";
import { _t } from "@web/core/l10n/translation";
import { App } from "@odoo/owl";

import { getTemplate } from "@web/core/templates";
import { registry } from "@web/core/registry";
import { session } from "@web/session";

export const livechatBootService = {
    dependencies: ["mail.store"],

    /**
     * To be overriden in tests.
     */
    getTarget() {
        return document.body;
    },

    start(env) {
        if (!session.livechatData?.isAvailable) {
            return;
        }
        const target = this.getTarget();
        const root = makeRoot(target);
        makeShadow(root).then((shadow) => {
            new App(LivechatRoot, {
                env,
                getTemplate,
                translatableAttributes: ["data-tooltip"],
                translateFn: _t,
                dev: env.debug,
            }).mount(shadow);
        });
    },
};
registry.category("services").add("im_livechat.boot", livechatBootService);

```

## File: static\src\embed\frontend\chat_hub_service_patch.js

```javascript
import { chatHubService } from "@mail/core/common/chat_hub";
import { patch } from "@web/core/utils/patch";

patch(chatHubService, {
    // When LiveChat is active, chat hub is added to the live chat root
    // component, see: livechat_root.js, boot_service.js.
    start() {},
});

```

## File: static\src\embed\frontend\livechat_root.js

```javascript
import { LivechatButton } from "@im_livechat/embed/common/livechat_button";

import { ChatHub } from "@mail/core/common/chat_hub";

import { Component, useSubEnv, xml } from "@odoo/owl";

import { useService } from "@web/core/utils/hooks";
// overlay inside shadow so that the styles are dicted by the shadow dom
import { OverlayContainer } from "@web/core/overlay/overlay_container";

export class LivechatRoot extends Component {
    static template = xml`
        <ChatHub/>
        <LivechatButton/>
        <OverlayContainer overlays="overlayService.overlays"/>
    `;
    static components = { ChatHub, LivechatButton, OverlayContainer };
    static props = {};

    setup() {
        useSubEnv({ embedLivechat: true });
        this.overlayService = useService("overlay");
    }
}

```

## File: static\src\js\ajax_external.js

```javascript
import { assets } from "@web/core/assets";

/**
 * This file should be used in the context of an external widget loading (e.g: live chat in a non-Odoo website)
 * It overrides the 'loadJS' method that is supposed to load additional scripts, based on a relative URL (e.g: '/web/webclient/locale/en_US')
 * As we're not in an Odoo website context, the calls will not work, and we avoid a 404 request.
 */
assets.loadJS = function (url) {
    console.log("Tried to load the following script on an external website: " + url);
};

```

## File: static\src\js\im_livechat_chatbot_script_answers_m2m.js

```javascript
import { registry } from "@web/core/registry";
import { user } from "@web/core/user";
import {
    Many2ManyTagsField,
    many2ManyTagsField,
} from "@web/views/fields/many2many_tags/many2many_tags_field";

const fieldRegistry = registry.category("fields");

export class ChatbotScriptTriggeringAnswersMany2Many extends Many2ManyTagsField {
    /**
     * Force the chatbot script ID we are currently editing into the context.
     * This allows to filter triggering question answers on steps of this script.
     */
    setup() {
        super.setup();

        if (this.props.record.model.root.resId) {
            user.updateContext({
                force_domain_chatbot_script_id: this.props.record.model.root.resId,
            });
        }
    }
}

export const chatbotScriptTriggeringAnswersMany2Many = {
    ...many2ManyTagsField,
    component: ChatbotScriptTriggeringAnswersMany2Many,
};

fieldRegistry.add("chatbot_triggering_answers_widget", chatbotScriptTriggeringAnswersMany2Many);

```

## File: static\src\js\im_livechat_chatbot_steps_one2many.js

```javascript
import { registry } from "@web/core/registry";
import { useX2ManyCrud, useOpenX2ManyRecord } from "@web/views/fields/relational_utils";
import { X2ManyField, x2ManyField } from "@web/views/fields/x2many/x2many_field";
import { ListRenderer } from "@web/views/list/list_renderer";

const fieldRegistry = registry.category("fields");

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
    static components = {
        ...X2ManyField.components,
        ListRenderer: ChatbotStepsOne2manyRenderer,
    };
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

        const { saveRecord, updateRecord } = useX2ManyCrud(() => this.list, this.isMany2Many);

        const openRecord = useOpenX2ManyRecord({
            resModel: this.list.resModel,
            activeField: this.activeField,
            activeActions: this.activeActions,
            getList: () => this.list,
            saveRecord: async (record) => {
                await saveRecord(record);
                await this.props.record.save();
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
}

export const chatbotStepsOne2many = {
    ...x2ManyField,
    component: ChatbotStepsOne2many,
};

fieldRegistry.add("chatbot_steps_one2many", chatbotStepsOne2many);

```

## File: static\src\js\colors_reset_button\colors_reset_button.js

```javascript
import { registry } from "@web/core/registry";
import { standardWidgetProps } from "@web/views/widgets/standard_widget_props";
import { Component } from "@odoo/owl";

export class ColorsResetButton extends Component {
    static template = `im_livechat.ColorsResetButton`;
    static props = {
        ...standardWidgetProps,
        default_colors: { type: Object },
    };

    onColorsResetButtonClick() {
        this.props.record.update(this.props.default_colors);
    }
}

export const colorsResetButton = {
    component: ColorsResetButton,
    extractProps: ({ options }) => ({
        // Note: `options` should have `default_colors`. It's specified when using the widget.
        default_colors: options.default_colors,
    }),
};
registry.category("view_widgets").add("colors_reset_button", colorsResetButton);

```

## File: static\src\js\colors_reset_button\colors_reset_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="im_livechat.ColorsResetButton">
        <button class="btn btn-link oe_edit_only" t-on-click="onColorsResetButtonClick" aria-label="Reset to default colors" title="Reset to default colors">
            <span class="fa fa-refresh mb-4"/>
        </button>
    </t>

</templates>

```

## File: static\src\views\livechat_view_controller_mixin.js

```javascript
import { useState } from "@odoo/owl";

import { useService } from "@web/core/utils/hooks";
import { _t } from "@web/core/l10n/translation";

export const LivechatViewControllerMixin = (ViewController) =>
    class extends ViewController {
        setup() {
            super.setup(...arguments);
            this.store = useState(useService("mail.store"));
            this.ui = useState(useService("ui"));
        }

        async openRecord(record) {
            if (!this.ui.isSmall) {
                return this.actionService.doAction("mail.action_discuss", {
                    name: _t("Discuss"),
                    additionalContext: { active_id: record.resId },
                });
            }
            const thread = await this.store.Thread.getOrFetch({
                model: "discuss.channel",
                id: record.resId,
            });
            if (thread) {
                return thread.open();
            }
            return super.openRecord(record);
        }
    };

```

## File: static\src\views\discuss_channel_kanban\discuss_channel_kanban_view.js

```javascript
import { KanbanController } from "@web/views/kanban/kanban_controller";
import { kanbanView } from "@web/views/kanban/kanban_view";
import { registry } from "@web/core/registry";
import { LivechatViewControllerMixin } from "../livechat_view_controller_mixin";

class DiscussChannelKanbanController extends LivechatViewControllerMixin(KanbanController) {}

const discussChannelKanbanView = {
    ...kanbanView,
    Controller: DiscussChannelKanbanController,
};

registry.category("views").add("im_livechat.discuss_channel_kanban", discussChannelKanbanView);

```

## File: static\src\views\discuss_channel_list\discuss_channel_list_view.js

```javascript
import { ListController } from "@web/views/list/list_controller";
import { LivechatViewControllerMixin } from "../livechat_view_controller_mixin";
import { listView } from "@web/views/list/list_view";
import { registry } from "@web/core/registry";

class DiscussChannelListController extends LivechatViewControllerMixin(ListController) {}

const discussChannelListView = {
    ...listView,
    Controller: DiscussChannelListController,
};

registry.category("views").add("im_livechat.discuss_channel_list", discussChannelListView);

```

## File: static\src\views\livechat_channel_kanban\livechat_channel_kanban_menu.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-name="im_livechat.KanbanRecordMenu" t-inherit="web.KanbanRecordMenu">
        <xpath expr="//Dropdown" position="attributes">
            <attribute name="position">'bottom-start'</attribute>
            <attribute name="menuClass" add="` rounded-0 ${props.record.data.available_operator_ids.count > 0 ? 'o-livechat-ChannelKanban-highlighted': ''}`" separator="+"/>
        </xpath>
        <xpath expr="//Dropdown/button" position="attributes">
            <attribute name="class" add="bg-transparent px-2" separator=" "/>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\livechat_channel_kanban\livechat_channel_kanban_record.js

```javascript
import { KanbanRecord } from "@web/views/kanban/kanban_record";

export class LivechatChannelKanbanRecord extends KanbanRecord {}
LivechatChannelKanbanRecord.menuTemplate = "im_livechat.KanbanRecordMenu";

```

## File: static\src\views\livechat_channel_kanban\livechat_channel_kanban_renderer.js

```javascript
import { KanbanRenderer } from "@web/views/kanban/kanban_renderer";
import { LivechatChannelKanbanRecord } from "./livechat_channel_kanban_record";

export class LivechatChannelKanbanRenderer extends KanbanRenderer {
    static components = {
        ...KanbanRenderer.components,
        KanbanRecord: LivechatChannelKanbanRecord,
    };
}

```

## File: static\src\views\livechat_channel_kanban\livechat_channel_kanban_view.js

```javascript
import { LivechatChannelKanbanRenderer } from "@im_livechat/views/livechat_channel_kanban/livechat_channel_kanban_renderer";

import { registry } from "@web/core/registry";
import { kanbanView } from "@web/views/kanban/kanban_view";

const livechatChannelKanbanView = {
    ...kanbanView,
    Renderer: LivechatChannelKanbanRenderer,
};

registry.category("views").add("im_livechat.livechat_channel_kanban", livechatChannelKanbanView);

```

## File: tools\misc.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request
from werkzeug.exceptions import NotFound

def downgrade_to_public_user():
    """Replace the request user by the public one. All the cookies are removed
    in order to ensure that the no user-specific data is kept in the request."""
    public_user = request.env.ref("base.public_user")
    request.update_env(user=public_user)
    request.cookies = {}


def force_guest_env(guest_token, raise_if_not_found=True):
    """Retrieve the guest from the given token and add it to the context.
    The request user is then replaced by the public one.

    :param str guest_token:
    :param bool raise_if_not_found: whether to raise if the guest cannot be
        found from the token
    :raise NotFound: if the guest cannot be found from the token and the
        ``raise_if_not_found`` parameter is set to ``True``
    """
    downgrade_to_public_user()
    guest = request.env["mail.guest"]._get_guest_from_token(guest_token)
    if guest:
        request.update_context(guest=guest)
    elif raise_if_not_found:
        raise NotFound()

```

## File: tools\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import misc

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
        <field name="name">chatbot.script.answer.view.list</field>
        <field name="model">chatbot.script.answer</field>
        <field name="arch" type="xml">
            <list editable="bottom">
                <field name="sequence" widget="handle"/>
                <field name="script_step_id"/>
                <field name="name"/>
                <field name="redirect_link" string="Optional Link"/>
            </list>
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
                <div class="alert alert-info text-center mb-0" role="alert" invisible="not is_forward_operator_child">
                    <span>Reminder: This step will only be played if no operator is available.</span>
                </div>
                <div class="alert alert-info text-center mb-0" role="alert" invisible="step_type != 'forward_operator'">
                    <span>Tip: Plan further steps for the Bot in case no operator is available.</span>
                </div>
                <sheet>
                    <group>
                        <group>
                            <field name="sequence" invisible="1"/>
                            <field name="message" widget="text_emojis" placeholder="e.g. 'How can I help you?'"
                                required="step_type != 'forward_operator'"/>
                            <field name="chatbot_script_id" invisible="1"/>
                            <field name="step_type"/>
                            <field name="triggering_answer_ids" widget="chatbot_triggering_answers_widget"
                                    options="{'no_create': True}">
                                <list>
                                    <!-- added only to correctly fetch the display_name for the tag display -->
                                    <field name="display_name" column_invisible="True"/>
                                </list>
                            </field>
                        </group>
                        <group>
                            <field name="answer_ids" invisible="step_type != 'question_selection'" nolabel="1">
                                <list editable="bottom">
                                    <field name="sequence" widget="handle"/>
                                    <field name="display_name" column_invisible="True"/>
                                    <field name="name"/>
                                    <field name="redirect_link" string="Optional Link"/>
                                </list>
                            </field>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="chatbot_script_step_view_tree" model="ir.ui.view">
        <field name="name">chatbot.script.step.view.list</field>
        <field name="model">chatbot.script.step</field>
        <field name="arch" type="xml">
            <list default_order="sequence asc">
                <field name="sequence" widget="handle"/>
                <field name="message"/>
                <field name="step_type"/>
                <field name="answer_ids" widget="many2many_tags"/>
                <field name="triggering_answer_ids" widget="many2many_tags"/>
            </list>
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
                    invisible="first_step_warning != 'first_step_operator'">
                    <span>Tip: At least one interaction (Question, Email, ...) is needed before the Bot can perform more complex actions (Forward to an Operator, ...). </span>
                    <span>Use Channel Rules if you want the Bot to interact with visitors only when no operator is available.</span>
                </div>
                <div class="alert alert-info text-center" role="alert"
                    invisible="first_step_warning != 'first_step_invalid'">
                    <span>Tip: At least one interaction (Question, Email, ...) is needed before the Bot can perform more complex actions (Forward to an Operator, ...).</span>
                </div>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_view_livechat_channels" type="object" class="oe_stat_button"
                                icon="fa-comments" invisible="livechat_channel_count == 0">
                            <field name="livechat_channel_count" string="Channels" widget="statinfo"/>
                        </button>
                    </div>
                    <field name="active" invisible="1"/>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <field name="image_1920" widget="image" class="oe_avatar" options="{'preview_image': 'image_128'}"/>
                    <div class="oe_title">
                        <label for="title" string="Chatbot Name"/>
                        <h1><field name="title" default_focus="1" placeholder='e.g. "Meeting Scheduler Bot"'/></h1>
                    </div>
                    <notebook>
                        <page string="Script" name="page_script">
                            <field name="script_step_ids" widget="chatbot_steps_one2many" nolabel="1"
                                context="{'chatbot_script_answer_display_short_name': 1}"/>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="chatbot_script_view_tree" model="ir.ui.view">
        <field name="name">chatbot.script.view.list</field>
        <field name="model">chatbot.script</field>
        <field name="arch" type="xml">
            <list sample="1">
                <field name="title"/>
            </list>
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
        <field name="view_mode">list,form</field>
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

## File: views\discuss_channel_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>

        <record id="discuss_channel_view_search" model="ir.ui.view">
            <field name="name">discuss.channel.search</field>
            <field name="model">discuss.channel</field>
            <field name="arch" type="xml">
                <search string="Search history">
                    <field name="name" string="Participant"/>
                    <filter name="filter_my_sessions" domain="[('is_member', '=', True)]" string="My Sessions"/>
                    <separator/>
                    <filter name="filter_session_date" date="create_date" string="Session Date"/>
                    <separator/>
                    <filter name="filter_session_bad_rating" domain="[('rating_ids', '!=', False), ('rating_avg', '&lt;', 2.5)]" string="Bad Ratings"/>
                    <filter name="filter_session_good_rating" domain="[('rating_ids', '!=', False), ('rating_avg', '&gt;=', 2.5)]" string="Good Ratings"/>
                    <filter name="fiter_session_unrated" domain="[('rating_ids', '=', False)]" string="Unrated"/>
                    <group expand="0" string="Group By...">
                        <filter name="group_by_channel" string="Channel" domain="[]" context="{'group_by':'livechat_channel_id'}"/>
                        <separator orientation="vertical"/>
                        <filter name="group_by_month" string="Session Date" domain="[]" context="{'group_by':'create_date:month'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="discuss_channel_view_tree" model="ir.ui.view">
            <field name="name">discuss.channel.list</field>
            <field name="model">discuss.channel</field>
            <field name="arch" type="xml">
                <list js_class="im_livechat.discuss_channel_list" string="History" create="false" default_order="create_date desc">
                    <field name="is_member" column_invisible="True"/>
                    <field name="create_date" string="Session Date"/>
                    <field name="name" string="Participants"/>
                    <field name="country_id"/>
                    <field name="duration" widget="float_time" options="{'displaySeconds': True}"/>
                    <field name="message_ids" string="# Messages"/>
                    <field name="rating_last_image" string="Rating" widget="image" options='{"size": [20, 20]}' class="bg-view"/>
                </list>
            </field>
        </record>

        <record id="discuss_channel_view_kanban" model="ir.ui.view">
            <field name="name">discuss.channel.kanban</field>
            <field name="model">discuss.channel</field>
            <field name="arch" type="xml">
                <kanban js_class="im_livechat.discuss_channel_kanban" class="o_kanban_mobile" sample="1" quick_create="false">
                    <templates>
                        <t t-name="card">
                            <div class="d-flex">
                                <div class="d-flex flex-column">
                                    <field class="fw-bolder" name="name"/>
                                    <span class="fw-bold">Duration: <field class="d-inline fw-normal" name="duration" widget="float_time" options="{'displaySeconds': True}"/></span>
                                    <span class="fw-bold">Date: <field name="create_date" class="fw-normal"/></span>
                                </div>
                                <field name="rating_last_image" string="Rating" widget="image" options='{"size": [40, 40]}' class="ms-auto"/>
                            </div>
                            <footer class="pt-0">
                                <t t-if="record.country_id.raw_value">
                                    <span class="fw-bold">Country: </span><field string="Country" name="country_id"/>
                                </t>
                                <field name="livechat_operator_id" widget="many2one_avatar_user" class="ms-auto me-1"/>
                            </footer>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="discuss_channel_view_form" model="ir.ui.view">
            <field name="name">discuss.channel.form</field>
            <field name="model">discuss.channel</field>
            <field name="arch" type="xml">
                <form string="Session Form" create="false" edit="false">
                    <sheet>
                        <div style="width:50%" class="float-end">
                            <field name="rating_last_image" widget="image" class="float-end bg-view" readonly="1" nolabel="1"/>
                            <field name="rating_last_feedback" nolabel="1"/>
                        </div>
                        <div style="width:50%" class="float-start">
                            <group>
                                <field name="name" string="Participants"/>
                                <field name="create_date" readonly="1" string="Session Date"/>
                            </group>
                        </div>
                    </sheet>
                </form>
            </field>
        </record>


        <record id="discuss_channel_action" model="ir.actions.act_window">
            <field name="name">History</field>
            <field name="res_model">discuss.channel</field>
            <field name="view_mode">kanban,list,form</field>
            <field name="search_view_id" ref="im_livechat.discuss_channel_view_search"/>
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
        <record id="discuss_channel_action_kanban" model="ir.actions.act_window.view">
            <field name="sequence">1</field>
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="im_livechat.discuss_channel_view_kanban"/>
            <field name="act_window_id" ref="im_livechat.discuss_channel_action"/>
        </record>

        <record id="discuss_channel_action_tree" model="ir.actions.act_window.view">
            <field name="sequence">2</field>
            <field name="view_mode">list</field>
            <field name="view_id" ref="im_livechat.discuss_channel_view_tree"/>
            <field name="act_window_id" ref="im_livechat.discuss_channel_action"/>
        </record>

        <record id="discuss_channel_action_form" model="ir.actions.act_window.view">
            <field name="sequence">3</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="im_livechat.discuss_channel_view_form"/>
            <field name="act_window_id" ref="im_livechat.discuss_channel_action"/>
        </record>

        <record id="discuss_channel_action_from_livechat_channel" model="ir.actions.act_window">
            <field name="name">Sessions</field>
            <field name="res_model">discuss.channel</field>
            <field name="view_mode">kanban,list,form</field>
            <field name="domain">[('livechat_channel_id', 'in', [active_id])]</field>
            <field name="context">{
                'search_default_livechat_channel_id': [active_id],
                'default_livechat_channel_id': active_id,
            }</field>
            <field name="search_view_id" ref="discuss_channel_view_search"/>
        </record>
        <record id="discuss_channel_action_livechat_kanban" model="ir.actions.act_window.view">
            <field name="sequence">1</field>
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="im_livechat.discuss_channel_view_kanban"/>
            <field name="act_window_id" ref="im_livechat.discuss_channel_action_from_livechat_channel"/>
        </record>
        <record id="discuss_channel_action_livechat_tree" model="ir.actions.act_window.view">
            <field name="sequence">2</field>
            <field name="view_mode">list</field>
            <field name="view_id" ref="im_livechat.discuss_channel_view_tree"/>
            <field name="act_window_id" ref="im_livechat.discuss_channel_action_from_livechat_channel"/>
        </record>

        <record id="discuss_channel_action_livechat_form" model="ir.actions.act_window.view">
            <field name="sequence">3</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="im_livechat.discuss_channel_view_form"/>
            <field name="act_window_id" ref="im_livechat.discuss_channel_action_from_livechat_channel"/>
        </record>

    </data>
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

                    <script>
                        window.odoo ??= {};
                        window.odoo.csrf_token = "<t t-out="request.csrf_token(None)"/>";
                    </script>
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
            <!-- the loader -->
            <script defer="defer" t-attf-src="{{url}}/im_livechat/loader/{{channel_id}}" type="text/javascript"/>
            <!-- js of all the required lib (internal and external) -->
            <script defer="defer" t-attf-src="{{url}}/im_livechat/assets_embed.js" type="text/javascript" />
        </template>

        <!-- the js code to initialize the LiveSupport object -->
        <template id="loader" name="Livechat : Javascript appending the livechat button">
            <t t-translation="off">
                if (!window.odoo) {
                    window.odoo = {};
                }
                odoo.__session_info__ = odoo.__session_info__ || {
                    websocket_worker_version: <t t-out="json.dumps(info.get('websocket_worker_version', ''))"/>
                }
                odoo.__session_info__ = Object.assign(odoo.__session_info__, {
                    livechatData: {
                        isAvailable: <t t-out="'true' if info['available'] else 'false'"/>,
                        serverUrl: "<t t-out="info['server_url']"/>",
                        options: <t t-out="json.dumps(info.get('options', {}))"/>,
                    },
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
            <field name="path">livechat</field>
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
                <kanban js_class="im_livechat.livechat_channel_kanban" action="im_livechat.discuss_channel_action_from_livechat_channel" type="action">
                    <field name="are_you_inside"/>
                    <field name="rating_count"/>
                    <templates>
                        <t t-name="menu">
                            <div class="container">
                                <a type="object" name="action_view_rating" class="dropdown-item" role="menuitem">Ratings</a>
                                <a type="open" class="dropdown-item" role="menuitem">Configure Channel</a>
                            </div>
                        </t>
                        <t t-name="card" class="p-0 row g-0">
                            <aside t-if="record.image_128.raw_value" class="ps-4 col-3" t-att-class="{'o-livechat-ChannelKanban-highlighted': record.available_operator_ids.raw_value.length > 0}">
                                <field  name="image_128" widget="image" options="{'img_class': 'object-fit-contain'}" alt="Channel"/>
                            </aside>
                            <main class="pe-4 py-2 ps-2 col" t-att-class="{'o-livechat-ChannelKanban-highlighted': record.available_operator_ids.raw_value.length > 0, 'ps-4': !record.image_128.raw_value}">
                                <div class="d-flex justify-content-between">
                                    <div>
                                        <field name="name" class="fs-4" style="word-wrap: break-word;"/>
                                        <p class="fst-italic fs-5"><field name="nbr_channel"/> Sessions</p>
                                    </div>
                                    <div>
                                        <button t-if="record.are_you_inside.raw_value" name="action_quit" type="object" class="btn btn-primary">Leave</button>
                                        <button t-else="" name="action_join" type="object" class="btn btn-secondary">Join</button>
                                    </div>
                                </div>
                                <footer class="pt-0">
                                    <field name="available_operator_ids" widget="many2many_avatar_user" readonly="True"/>
                                    <a t-if="record.rating_count.raw_value > 0" name="action_view_rating" class="ms-auto fs-6" type="object" tabindex="10">
                                        <i class="fa fa-smile-o text-success" title="Percentage of happy ratings" role="img" aria-label="Happy face"/> <field name="rating_percentage_satisfaction"/>%
                                    </a>
                                </footer>
                            </main>
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
                        <button type="object" name="action_join" class="oe_highlight" string="Join Channel" invisible="are_you_inside"/>
                        <button type="object" name="action_quit" class="btn btn-primary" string="Leave Channel" invisible="not are_you_inside"/>
                        <field name="are_you_inside" invisible="1"/>
                    </header>
                    <sheet>
                        <field name="rating_count" invisible="1"/>
                        <div class="oe_button_box" name="button_box">
                            <button class="oe_stat_button" type="object" name="action_view_chatbot_scripts" icon="fa-android"
                                invisible="chatbot_script_count == 0">
                                <field string="Chatbots" name="chatbot_script_count" widget="statinfo"/>
                            </button>
                            <button class="oe_stat_button" type="action" invisible="nbr_channel == 0" name="%(discuss_channel_action_from_livechat_channel)d" icon="fa-comments">
                                <field string="Sessions" name="nbr_channel" widget="statinfo"/>
                            </button>
                            <button name="action_view_rating" invisible="rating_count == 0" class="oe_stat_button" type="object" icon="fa-smile-o">
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
                                            <templates>
                                                <t t-name="card" class="flex-row">
                                                    <aside>
                                                        <field name="avatar_1024" widget="image" alt="User"/>
                                                    </aside>
                                                    <main class="ps-1">
                                                        <div class="d-flex justify-content-between align-items-baseline">
                                                            <field name="name" class="fw-bold fs-5"/>
                                                            <a class="btn p-0 opacity-75 opacity-100-hover" role="button" groups="im_livechat.im_livechat_group_manager" type="delete">
                                                                <i title="Remove operator" class="fa fa-fw fa-lg fa-close"/>
                                                            </a>
                                                        </div>
                                                        <field name="livechat_username" string="Online Chat Name"/>
                                                    </main>
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
                                        <field name="button_text" string="Notification text" help="Text to display on the notification"/>
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
                                <div class="alert alert-warning mt4 mb16" role="alert" invisible="web_page">
                                    Save your Channel to get your configuration widget.
                                </div>
                                <div invisible="not web_page">
                                    <p>
                                        Learn how to display a live chat bubble on your website and start chatting with visitors in a few easy steps.
                                    </p>
                                    <separator string="FOR WEBSITES BUILT WITH ODOO" />
                                    <ol>
                                        <li>Go to <strong>Website</strong> > <strong>Configuration</strong> > <strong>Settings</strong> > <strong>Email</strong> &amp; <strong>Marketing.</strong></li>
                                        <li>Turn on the <strong>Live Chat</strong> feature.</li>
                                        <li>Choose the live chat channel you want to display on your website.</li>
                                    </ol>
                                    <separator string="FOR OTHER WEBSITES" />
                                    <p>
                                        Add the following code to the &lt;head&gt; section of your website:
                                    </p>
                                    <field class="text-muted" name="script_external" readonly="1" widget="CopyClipboardChar"/>
                                    <br/><br/>
                                    <p>
                                        Alternatively, share this URL with your customers or suppliers via email:
                                    </p>
                                    <field class="text-muted" name="web_page" readonly="1" widget="CopyClipboardChar"/>
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
            <field name="name">im.livechat.channel.rule.list</field>
            <field name="model">im_livechat.channel.rule</field>
            <field name="arch" type="xml">
                <list string="Rules">
                    <field name="sequence" widget="handle"/>
                    <field name="regex_url"/>
                    <field name="action"/>
                    <field name="country_ids" widget="many2many_tags"/>
                </list>
            </field>
        </record>

        <record id="im_livechat_channel_rule_view_kanban" model="ir.ui.view">
            <field name="name">im_livechat.channel.rule.kanban</field>
            <field name="model">im_livechat.channel.rule</field>
            <field name="arch" type="xml">
                <kanban>
                    <templates>
                        <t t-name="card">
                            <field name="action"/>
                            <field name="regex_url" />
                            <field name="country_ids" widget="many2many_tags" />
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="im_livechat_channel_rule_view_form" model="ir.ui.view">
            <field name="name">im_livechat.channel.rule.form</field>
            <field name="model">im_livechat.channel.rule</field>
            <field name="arch" type="xml">
                <form string="Channel Rule" class="o_livechat_rules_form">
                    <sheet>
                        <group>
                            <field name="action" widget="radio"/>
                            <label for="chatbot_script_id" string="Chatbot" invisible="action == 'hide_button'"/>
                            <div invisible="action == 'hide_button'">
                                <field name="chatbot_script_id" class="oe_inline" style="width: 60% !important;"
                                       options="{'no_create': True, 'no_open': True}"/>
                            </div>
                            <label for="chatbot_only_if_no_operator" class="oe_inline" invisible="not chatbot_script_id" string="Enabled only if no operator"/>
                            <div class="oe_inline" invisible="not chatbot_script_id">
                                <field name="chatbot_only_if_no_operator"/>
                            </div>
                            <field name="regex_url" placeholder="e.g. /contactus"/>
                            <label for="auto_popup_timer" class="oe_inline" invisible="action != 'auto_popup'"/>
                            <div class="oe_inline" invisible="action != 'auto_popup'">
                                <field name="auto_popup_timer" class="oe_inline"/> seconds
                            </div>
                            <field name="country_ids" widget="many2many_tags" options="{'no_open': True, 'no_create': True}"/>
                        </group>
                    </sheet>
                </form>
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
            action="discuss_channel_action"
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
            action="mail.mail_canned_response_action"
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
            <t t-call="web.frontend_layout">
                <t t-set="no_livechat" t-value="True"/>
                <t t-set="head">
                    <script>
                        <t t-call="im_livechat.loader">
                            <t t-set="info" t-value="{ 'available': True, 'options': { 'init': { 'available_for_me': True, 'rule': { 'chatbot': chatbot_data }, 'storeData': storeData }, 'isTestChatbot': True, 'force_thread': channel_data }, 'server_url': server_url }"/>
                        </t>
                    </script>
                </t>
                <t t-set="title" t-value="chatbot_data['name']"/>

                <div id="wrap">
                    <div groups="im_livechat.im_livechat_group_user" t-ignore="true"
                            class="alert alert-info alert-dismissible rounded-0 fade show d-print-none css_editable_mode_hidden mb-0">
                        <div t-ignore="true" class="text-center">
                            <a t-attf-href="/odoo/action-im_livechat.chatbot_script_action/#{chatbot_data['id']}">
                                <span>You are currently testing</span>
                                <span t-out="chatbot_data['name']"/>
                                <i class="oi oi-fw oi-arrow-right"/>Back to edit mode
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
        <field name="view_mode">kanban,list,graph,pivot,form</field>
        <field name="domain">[('parent_res_model', '=', 'im_livechat.channel'), ('consumed','=',True)]</field>
        <field name="search_view_id" ref="rating_rating_view_search_livechat"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                There is no rating for this channel at the moment
            </p>
        </field>
        <field name="context">{'search_default_rating_last_7_days': 1}</field>
    </record>

    <record id="im_livechat.rating_rating_view_kanban" model="ir.ui.view">
        <field name="name">im_livechat.rating.rating.view.kanban</field>
        <field name="model">rating.rating</field>
        <field name="inherit_id" ref="rating.rating_rating_view_kanban"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//kanban" position="attributes">
                <attribute name="type">object</attribute>
                <attribute name="action">action_open_rated_object</attribute>
            </xpath>
            <xpath expr="//*[@name='action_open_rated_object']" position="attributes">
                <attribute name="style">pointer-events: none;</attribute>
            </xpath>
        </field>
    </record>

    <record id="im_livechat.rating_rating_view_tree" model="ir.ui.view">
        <field name="name">im_livechat.rating.rating.view.list</field>
        <field name="model">rating.rating</field>
        <field name="inherit_id" ref="rating.rating_rating_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//list" position="attributes">
                <attribute name="type">object</attribute>
                <attribute name="action">action_open_rated_object</attribute>
            </xpath>
        </field>
    </record>

    <record id="rating_rating_action_livechat_view_kanban" model="ir.actions.act_window.view">
        <field name="sequence" eval="1"/>
        <field name="view_mode">kanban</field>
        <field name="act_window_id" ref="rating_rating_action_livechat"/>
        <field name="view_id" ref="im_livechat.rating_rating_view_kanban"/>
    </record>

    <record id="rating_rating_action_livechat_view_tree" model="ir.actions.act_window.view">
        <field name="sequence" eval="1"/>
        <field name="view_mode">list</field>
        <field name="act_window_id" ref="rating_rating_action_livechat"/>
        <field name="view_id" ref="im_livechat.rating_rating_view_tree"/>
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
        <field name="view_mode">kanban,list,pivot,graph,form</field>
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
                    <field name="has_access_livechat" invisible="1"/>
                    <field name="livechat_username" string="Online Chat Name"
                        invisible="not has_access_livechat"/>
                    <field name="livechat_lang_ids" string="Online Chat Language"
                        invisible="not has_access_livechat"
                        options="{'no_create': True, 'no_edit': True, 'no_quick_create': True}"
                        widget="many2many_tags"/>
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
                        <field name="has_access_livechat" invisible="1"/>
                        <group name="livechat" string="Livechat"
                            invisible="not has_access_livechat">
                            <field name="livechat_username"/>
                            <field name="livechat_lang_ids" string="Online Chat Language"
                                options="{'no_create': True, 'no_edit': True, 'no_quick_create': True}"
                                widget="many2many_tags"/>
                        </group>
                    </xpath>
            </field>
        </record>

    </data>
</odoo>

```

## File: views\webclient_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="im_livechat.unit_embed_suite">
        <t t-call="web.layout">
            <t t-set="html_data" t-value="{'style': 'height: 100%;'}"/>
            <t t-set="title">Livechat External Tests</t>
            <t t-set="head">
                <meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no"/>

                <script type="text/javascript">
                    odoo.__session_info__ = <t t-out="json.dumps(session_info)"/>;
                </script>

                <t t-call-assets="im_livechat.embed_assets_unit_tests_setup" />
                <t t-call-assets="im_livechat.embed_assets_unit_tests" />
            </t>
        </t>
    </template>
</odoo>

```

