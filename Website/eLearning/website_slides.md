# Odoo Module: website_slides

Category: Website/eLearning

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
{
    'name': 'eLearning',
    'version': '2.7',
    'sequence': 125,
    'summary': 'Manage and publish an eLearning platform',
    'website': 'https://www.odoo.com/app/elearning',
    'category': 'Website/eLearning',
    'description': """
Create Online Courses
=====================

Featuring

 * Integrated course and lesson management
 * Fullscreen navigation
 * Support Youtube videos, Google documents, PDF, images, articles
 * Test knowledge with quizzes
 * Filter and Tag
 * Statistics
""",
    'depends': [
        'portal_rating',
        'website',
        'website_mail',
        'website_profile',
    ],
    'data': [
        'security/website_slides_security.xml',
        'security/ir.model.access.csv',
        'views/gamification_karma_tracking_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
        'views/rating_rating_views.xml',
        'views/slide_embed_views.xml',
        'views/slide_question_views.xml',
        'views/slide_slide_partner_views.xml',
        'views/slide_slide_views.xml',
        'views/slide_channel_partner_views.xml',
        'views/slide_channel_views.xml',
        'views/slide_channel_tag_views.xml',
        'views/slide_snippets.xml',
        'views/website_slides_menu_views.xml',
        'views/website_slides_templates_homepage.xml',
        'views/website_slides_templates_course.xml',
        'views/website_slides_templates_lesson.xml',
        'views/website_slides_templates_lesson_fullscreen.xml',
        'views/website_slides_templates_lesson_embed.xml',
        'views/website_slides_templates_profile.xml',
        'views/website_slides_templates_utils.xml',
        'views/website_pages_views.xml',
        'views/slide_channel_add.xml',
        'wizard/slide_channel_invite_views.xml',
        'data/gamification_data.xml',
        'data/mail_activity_type_data.xml',
        'data/mail_message_subtype_data.xml',
        'data/mail_template_data.xml',
        'data/mail_templates.xml',
        'data/slide_data.xml',
        'data/website_data.xml',
        'data/slides_tour.xml',
    ],
    'demo': [
        'data/res_users_demo.xml',
        'data/slide_channel_tag_demo.xml',
        'data/slide_channel_demo.xml',
        'data/slide_slide_demo.xml',
        'data/slide_user_demo.xml',
        'data/slide_user_gamification_demo.xml',
    ],
    'installable': True,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'website_slides/static/src/activity/**/*',
            'website_slides/static/src/slide_category_one2many_field.js',
            'website_slides/static/src/slide_category_list_renderer.js',
            'website_slides/static/src/scss/slide_views.scss',
            'website_slides/static/src/js/tours/slides_tour.js',
            'website_slides/static/src/js/components/**/*.js',
            'website_slides/static/src/views/**/*.js',
            'website_slides/static/src/views/**/*.xml',
        ],
        'web.assets_frontend': [
            'website_slides/static/src/scss/website_slides.scss',
            'website_slides/static/src/scss/website_slides_profile.scss',
            'website_slides/static/src/scss/slides_slide_fullscreen.scss',
            'website_slides/static/src/js/slides.js',
            'website_slides/static/src/js/slides_share.js',
            'website_slides/static/src/js/slides_upload.js',
            'website_slides/static/src/js/slides_category_add.js',
            'website_slides/static/src/js/slides_category_delete.js',
            'website_slides/static/src/js/slides_slide_archive.js',
            'website_slides/static/src/js/slides_slide_toggle_is_preview.js',
            'website_slides/static/src/js/slides_slide_like.js',
            'website_slides/static/src/js/slides_course_page.js',
            'website_slides/static/src/js/slides_course_slides_list.js',
            'website_slides/static/src/js/slides_course_fullscreen_player.js',
            'website_slides/static/src/js/slides_course_join.js',
            'website_slides/static/src/js/slides_course_enroll_email.js',
            'website_slides/static/src/js/slides_course_prerequisite.js',
            'website_slides/static/src/js/slides_course_quiz.js',
            'website_slides/static/src/js/slides_course_quiz_question_form.js',
            'website_slides/static/src/js/slides_course_tag_add.js',
            'website_slides/static/src/js/slides_course_unsubscribe.js',
            'website_slides/static/src/js/snippets.animation.js',
            'website_slides/static/src/js/portal_rating_composer.js',
            'website_slides/static/src/xml/website_slides_sidebar.xml',
            'website_slides/static/src/xml/website_slides_fullscreen.xml',
            'website_slides/static/src/xml/slide_management.xml',
            'website_slides/static/src/xml/slide_course_join.xml',
            'website_slides/static/src/xml/slide_course_prerequisite.xml',
            'website_slides/static/src/xml/slide_quiz_create.xml',
            'website_slides/static/src/xml/slide_quiz.xml',
            'website_slides/static/src/js/public/**/*',
        ],
        'website.assets_editor': [
            'website_slides/static/src/js/systray_items/*.js',
        ],
        'web.assets_tests': [
            'website_slides/static/tests/tours/*.js',
        ],
        'website_slides.slide_embed_assets': [
            # TODO this bundle now includes 'assets_common' files directly, but
            # most of these files are useless in this context, clean this up.
            ('include', 'web._assets_helpers'),
            ('include', 'web._assets_frontend_helpers'),

            'web/static/src/scss/pre_variables.scss',
            'web/static/lib/bootstrap/scss/_variables.scss',
            'web/static/lib/bootstrap/scss/_variables-dark.scss',
            'web/static/lib/bootstrap/scss/_maps.scss',

            ('include', 'web._assets_bootstrap_frontend'),

            'web/static/src/libs/fontawesome/css/font-awesome.css',
            'web/static/lib/odoo_ui_icons/*',
            'web/static/src/webclient/navbar/navbar.scss',
            'web/static/src/scss/animation.scss',
            'web/static/src/scss/fontawesome_overridden.scss',
            'web/static/src/scss/mimetypes.scss',
            'web/static/src/scss/ui.scss',
            'web/static/src/core/colorpicker/colorpicker.scss',
            'web/static/src/views/fields/translation_dialog.scss',
            'web/static/src/views/fields/signature/signature_field.scss',
            'web/static/src/legacy/scss/ui.scss',
            'website/static/src/libs/zoomodoo/zoomodoo.scss',

            'web/static/src/module_loader.js',
            'web/static/src/session.js',

            'web/static/lib/luxon/luxon.js',
            'web/static/lib/owl/owl.js',
            'web/static/lib/owl/odoo_module.js',
            'web/static/lib/jquery/jquery.js',
            'web/static/lib/popper/popper.js',
            'web/static/lib/bootstrap/js/dist/util/index.js',
            'web/static/lib/bootstrap/js/dist/dom/data.js',
            'web/static/lib/bootstrap/js/dist/dom/event-handler.js',
            'web/static/lib/bootstrap/js/dist/dom/manipulator.js',
            'web/static/lib/bootstrap/js/dist/dom/selector-engine.js',
            'web/static/lib/bootstrap/js/dist/util/config.js',
            'web/static/lib/bootstrap/js/dist/util/component-functions.js',
            'web/static/lib/bootstrap/js/dist/util/backdrop.js',
            'web/static/lib/bootstrap/js/dist/util/focustrap.js',
            'web/static/lib/bootstrap/js/dist/util/sanitizer.js',
            'web/static/lib/bootstrap/js/dist/util/scrollbar.js',
            'web/static/lib/bootstrap/js/dist/util/swipe.js',
            'web/static/lib/bootstrap/js/dist/util/template-factory.js',
            'web/static/lib/bootstrap/js/dist/base-component.js',
            'web/static/lib/bootstrap/js/dist/alert.js',
            'web/static/lib/bootstrap/js/dist/button.js',
            'web/static/lib/bootstrap/js/dist/carousel.js',
            'web/static/lib/bootstrap/js/dist/collapse.js',
            'web/static/lib/bootstrap/js/dist/dropdown.js',
            'web/static/lib/bootstrap/js/dist/modal.js',
            'web/static/lib/bootstrap/js/dist/offcanvas.js',
            'web/static/lib/bootstrap/js/dist/tooltip.js',
            'web/static/lib/bootstrap/js/dist/popover.js',
            'web/static/lib/bootstrap/js/dist/scrollspy.js',
            'web/static/lib/bootstrap/js/dist/tab.js',
            'web/static/lib/bootstrap/js/dist/toast.js',
            'web/static/src/libs/bootstrap.js',
            'web/static/src/legacy/js/libs/jquery.js',
            'website/static/src/libs/zoomodoo/zoomodoo.js',
            'web/static/src/core/**/*.js',
            'web/static/src/env.js',
            'web/static/src/libs/pdfjs.js',
            ('remove', 'web/static/src/core/emoji_picker/emoji_data.js'),

            'website_slides/static/src/scss/website_slides.scss',
            'website_slides/static/lib/pdfslidesviewer/PDFSlidesViewer.js',
            'website_slides/static/src/js/slides_embed.js',
        ],
        'web.qunit_suite_tests': [
            'website_slides/static/tests/legacy/**/*',
        ],
        'web.assets_unit_tests': [
            'website_slides/static/tests/**/*',
            ('remove', 'website_slides/static/tests/legacy/**/*'),
            ('remove', 'website_slides/static/tests/tours/**/*'),
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\mail.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.exceptions import NotFound, Forbidden

from odoo import http
from odoo.http import request
from odoo.addons.portal.controllers.mail import PortalChatter
from odoo.tools import plaintext2html, html2plaintext


class SlidesPortalChatter(PortalChatter):

    def _portal_post_has_content(self, thread_model, thread_id, message, attachment_ids=None, **kw):
        """ Relax constraint on slide model: having a rating value is sufficient
        to consider we have a content. """
        if thread_model == 'slide.channel' and kw.get('rating_value'):
            return True
        return super()._portal_post_has_content(thread_model, thread_id, message, attachment_ids=attachment_ids, **kw)

    @http.route([
        '/slides/mail/update_comment',
        ], type='json', auth="user", methods=['POST'])
    def mail_update_message(self, thread_model, thread_id, message_id, post_data, **post):
        # keep this mechanism intern to slide currently (saas 12.5) as it is
        # considered experimental
        if thread_model != 'slide.channel':
            raise Forbidden()
        thread_id = int(thread_id)
        attachment_ids = post_data.get('attachment_ids', [])

        self._portal_post_check_attachments(attachment_ids, post.get('attachment_tokens', []))

        pid = int(post['pid']) if post.get('pid') else False
        channel = request.env["slide.channel"]._get_thread_with_access(
            thread_id,
            request.env["slide.channel"]._mail_post_access,
            token=post.get("token"),
            hash=post.get("hash"),
            pid=pid,
        )
        if not channel:
            raise Forbidden()
        # fetch and update mail.message
        message_id = int(message_id)
        message_body = plaintext2html(post_data.get('body', ''))
        subtype_comment_id = request.env['ir.model.data']._xmlid_to_res_id('mail.mt_comment')
        domain = [
            ('model', '=', thread_model),
            ('res_id', '=', thread_id),
            ('subtype_id', '=', subtype_comment_id),
            ('author_id', '=', request.env.user.partner_id.id),
            ('message_type', '=', 'comment'),
            ('id', '=', message_id)
        ]  # restrict to the given message_id
        message = request.env['mail.message'].search(domain, limit=1)
        if not message:
            raise NotFound()
        message.sudo().write({
            'body': message_body,
            'attachment_ids': [(4, aid) for aid in attachment_ids],
        })

        # update rating
        if post_data.get('rating_value'):
            domain = [('res_model', '=', thread_model), ('res_id', '=', thread_id), ('message_id', '=', message.id)]
            rating = request.env['rating.rating'].sudo().search(domain, order='write_date DESC', limit=1)
            rating.write({
                'rating': float(post_data['rating_value']),
                'feedback': html2plaintext(message.body),
            })
        return {
            'default_message_id': message.id,
            'default_message': html2plaintext(message.body),
            'default_rating_value': message.rating_value,
            'rating_avg': channel.rating_avg,
            'rating_count': channel.rating_count,
            'default_attachment_ids': message.attachment_ids.sudo().read(['id', 'name', 'mimetype', 'file_size', 'access_token']),
            'force_submit_url': '/slides/mail/update_comment',
        }

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval
from dateutil.relativedelta import relativedelta

import base64
import json
import logging
import math
import werkzeug

from odoo import fields, http, tools, _
from odoo.addons.website.controllers.main import QueryURL
from odoo.addons.website.models.ir_http import sitemap_qs2dom
from odoo.addons.website_profile.controllers.main import WebsiteProfile
from odoo.exceptions import AccessError, ValidationError, UserError, MissingError
from odoo.http import request, Response
from odoo.osv import expression
from odoo.tools import consteq, email_split

_logger = logging.getLogger(__name__)


def handle_wslide_error(exception):
    if isinstance(exception, AccessError):
        return request.redirect("/slides?invite_error=no_rights", 302)


class WebsiteSlides(WebsiteProfile):
    _slides_per_page = 12
    _slides_per_aside = 20
    _slides_per_category = 3
    _channel_order_by_criterion = {
        'vote': 'total_votes desc',
        'view': 'total_views desc',
        'date': 'create_date desc',
    }

    def sitemap_slide(env, rule, qs):
        Channel = env['slide.channel']
        dom = sitemap_qs2dom(qs=qs, route='/slides/', field=Channel._rec_name)
        dom += env['website'].get_current_website().website_domain()
        for channel in Channel.search(dom):
            loc = '/slides/%s' % env['ir.http']._slug(channel)
            if not qs or qs.lower() in loc:
                yield {'loc': loc}

    def _slide_render_context_base(self):
        return {
            # current user info
            'user': request.env.user,
            'is_public_user': request.website.is_public_user(),
            # tools
            '_slugify_tags': self._slugify_tags,
        }

    # SLIDE UTILITIES
    # --------------------------------------------------

    def _fetch_slide(self, slide_id):
        slide = request.env['slide.slide'].browse(int(slide_id)).exists()
        if not slide:
            return {'error': 'slide_wrong'}
        if not slide.has_access('read'):
            return {'error': 'slide_access'}
        return {'slide': slide}

    def _set_viewed_slide(self, slide, quiz_attempts_inc=False):
        if not slide.channel_id.is_member:
            if not isinstance(request.session.get('viewed_slides'), dict):
                # Compatibility layer with Odoo 15.0,
                # where `viewed_slides` are stored as `list` in sessions.
                # For performance concerns, `viewed_slides` is changed to a dict,
                # but sessions coming from Odoo 15.0 after an upgrade should still be compatible.
                # This compatibility layer regarding `viewed_slides` must remain from Odoo 16.0 and above,
                # as this is possible to do a jump of multiple versions in one go,
                # and carry the sessions with the upgrade.
                # e.g. upgrade from Odoo 15.0 to 18.0.
                request.session.viewed_slides = dict.fromkeys(request.session.get('viewed_slides', []), 1)
            viewed_slides = request.session['viewed_slides']
            # Convert `slide.id` to string is necessary because of the JSON format of the session
            slide_id = str(slide.id)
            if slide_id not in viewed_slides:
                if tools.sql.increment_fields_skiplock(slide, 'public_views', 'total_views'):
                    viewed_slides[slide_id] = 1
                    request.session.touch()
        else:
            slide.action_set_viewed(quiz_attempts_inc=quiz_attempts_inc)
        return True

    def _slide_mark_completed(self, slide):
        # quiz use their specific mechanism to be marked as done
        if slide.slide_category == 'quiz' or slide.question_ids:
            raise UserError(_("Slide with questions must be marked as done when submitting all good answers "))
        if not slide.can_self_mark_completed:
            raise werkzeug.exceptions.Forbidden(_("This slide can not be marked as completed."))
        slide.action_mark_completed()

    def _slide_mark_uncompleted(self, slide):
        if not slide.can_self_mark_uncompleted:
            raise werkzeug.exceptions.Forbidden(_("This slide can not be marked as uncompleted."))
        slide.action_mark_uncompleted()

    def _get_slide_detail(self, slide):
        base_domain = self._get_channel_slides_base_domain(slide.channel_id)
        category_data = slide.channel_id._get_categorized_slides(
            base_domain,
            order=request.env['slide.slide']._order_by_strategy['sequence'],
            force_void=True
        )

        if slide.channel_id.channel_type == 'documentation':
            most_viewed_slides = request.env['slide.slide'].search(base_domain, limit=self._slides_per_aside, order='total_views desc')
            related_domain = expression.AND([base_domain, [('category_id', '=', slide.category_id.id)]])
            related_slides = request.env['slide.slide'].search(related_domain, limit=self._slides_per_aside)
        else:
            most_viewed_slides, related_slides = request.env['slide.slide'], request.env['slide.slide']

        channel_slides_ids = slide.channel_id.slide_content_ids.ids
        slide_index = channel_slides_ids.index(slide.id)
        previous_slide = slide.channel_id.slide_content_ids[slide_index-1] if slide_index > 0 else None
        next_slide = slide.channel_id.slide_content_ids[slide_index+1] if slide_index < len(channel_slides_ids) - 1 else None

        render_values = self._slide_render_context_base()
        render_values.update({
            # slide
            'slide': slide,
            'main_object': slide,
            'most_viewed_slides': most_viewed_slides,
            'related_slides': related_slides,
            'previous_slide': previous_slide,
            'next_slide': next_slide,
            'category_data': category_data,
            # rating and comments
            'comments': slide.website_message_ids or [],
        })

        # allow rating and comments
        if slide.channel_id.allow_comment:
            render_values.update({
                'message_post_pid': request.env.user.partner_id.id,
            })

        return render_values

    def _get_slide_quiz_partner_info(self, slide, quiz_done=False):
        return slide._compute_quiz_info(request.env.user.partner_id, quiz_done=quiz_done)[slide.id]

    def _get_slide_quiz_data(self, slide):
        is_designer = request.env.user.has_group('website.group_website_designer')
        slides_resources = slide.slide_resource_ids if slide.channel_id.is_member else []
        values = {
            'slide_description': slide.description,
            'slide_questions': [{
                'answer_ids': [{
                    'comment': answer.comment if is_designer else None,
                    'id': answer.id,
                    'is_correct': answer.is_correct if slide.user_has_completed or is_designer else None,
                    'text_value': answer.text_value,
                } for answer in question.sudo().answer_ids],
                'id': question.id,
                'question': question.question,
            } for question in slide.question_ids],
            'slide_resource_ids': [{
                'display_name' : resource.display_name,
                'download_url': resource.download_url,
                'id': resource.id,
                'link': resource.link,
                'resource_type': resource.resource_type,
            } for resource in slides_resources]
        }
        if 'slide_answer_quiz' in request.session:
            slide_answer_quiz = json.loads(request.session['slide_answer_quiz'])
            if str(slide.id) in slide_answer_quiz:
                values['session_answers'] = slide_answer_quiz[str(slide.id)]
        values.update(self._get_slide_quiz_partner_info(slide))
        return values

    def _get_new_slide_category_values(self, channel, name):
        return {
            'name': name,
            'channel_id': channel.id,
            'is_category': True,
            'is_published': True,
            'sequence': channel.slide_ids[-1]['sequence'] + 1 if channel.slide_ids else 1,
        }

    # CHANNEL UTILITIES
    # --------------------------------------------------

    def _get_channel_slides_base_domain(self, channel):
        """ base domain when fetching slide list data related to a given channel

         * website related domain, and restricted to the channel and is not a
           category slide (behavior is different from classic slide);
         * if publisher: everything is ok;
         * if not publisher but has user: either slide is published, either
           current user is the one that uploaded it;
         * if not publisher and public: published;
        """
        base_domain = expression.AND([request.website.website_domain(), ['&', ('channel_id', '=', channel.id), ('is_category', '=', False)]])
        if not channel.can_publish:
            if request.website.is_public_user():
                base_domain = expression.AND([base_domain, [('website_published', '=', True)]])
            else:
                base_domain = expression.AND([base_domain, ['|', ('website_published', '=', True), ('user_id', '=', request.env.user.id)]])
        return base_domain

    def _get_channel_progress(self, channel, include_quiz=False):
        """ Replacement to user_progress. Both may exist in some transient state. """
        slides = request.env['slide.slide'].sudo().search([('channel_id', '=', channel.id)])
        channel_progress = dict((sid, dict()) for sid in slides.ids)
        if not request.env.user._is_public() and channel.is_member:
            slide_partners = request.env['slide.slide.partner'].sudo().search([
                ('channel_id', '=', channel.id),
                ('partner_id', '=', request.env.user.partner_id.id),
                ('slide_id', 'in', slides.ids)
            ])
            for slide_partner in slide_partners:
                channel_progress[slide_partner.slide_id.id].update(slide_partner.read()[0])
                if slide_partner.slide_id.question_ids:
                    gains = [slide_partner.slide_id.quiz_first_attempt_reward,
                             slide_partner.slide_id.quiz_second_attempt_reward,
                             slide_partner.slide_id.quiz_third_attempt_reward,
                             slide_partner.slide_id.quiz_fourth_attempt_reward]
                    channel_progress[slide_partner.slide_id.id]['quiz_gain'] = gains[slide_partner.quiz_attempts_count] if slide_partner.quiz_attempts_count < len(gains) else gains[-1]

        if include_quiz:
            quiz_info = slides._compute_quiz_info(request.env.user.partner_id, quiz_done=False)
            for slide_id, slide_info in quiz_info.items():
                channel_progress[slide_id].update(slide_info)

        return channel_progress

    def _channel_remove_session_answers(self, channel, slide=False):
        """ Will remove the answers saved in the session for a specific channel / slide. """

        if 'slide_answer_quiz' not in request.session:
            return

        slides_domain = [('channel_id', '=', channel.id)]
        if slide:
            slides_domain = expression.AND([slides_domain, [('id', '=', slide.id)]])
        slides = request.env['slide.slide'].search(slides_domain)

        session_slide_answer_quiz = json.loads(request.session['slide_answer_quiz'])
        for slide_id in slides.ids:
            session_slide_answer_quiz.pop(str(slide_id), None)
        request.session['slide_answer_quiz'] = json.dumps(session_slide_answer_quiz)

    def _prepare_collapsed_categories(self, categories_values, slide, next_category_to_open):
        """ Collapse the category if:
            - there is no category (the slides are uncategorized)
            - the category contains the current slide
            - the category is ongoing (has at least one slide completed but not all of its slides)
            - the category is the next one to be opened because the current one has just been completed
        """
        if request.env.user._is_public() or not slide.channel_id.is_member:
            return categories_values
        for category_dict in categories_values:
            category = category_dict.get('category')
            if not category or slide in category.slide_ids or category == next_category_to_open:
                category_dict['is_collapsed'] = True
            else:
                # collapse if category is ongoing
                slides_completion = category.slide_ids.mapped('user_has_completed')
                category_dict['is_collapsed'] = any(slides_completion) and not all(slides_completion)
        return categories_values

    # TAG UTILITIES
    # --------------------------------------------------

    def _slugify_tags(self, tag_ids, toggle_tag_id=None):
        """ Prepares a comma separated slugified tags for the sake of readable
        URLs.

        :param toggle_tag_id: add the tag being clicked (current_tag) to the already
          selected tags (tag_ids) as well as in URL; if tag is already selected
          by the user it is removed from the selected tags (and so from the URL);
        """
        tag_ids = list(tag_ids)  # required to avoid using the same list
        if toggle_tag_id and toggle_tag_id in tag_ids:
            tag_ids.remove(toggle_tag_id)
        elif toggle_tag_id:
            tag_ids.append(toggle_tag_id)
        return ','.join(request.env['ir.http']._slug(tag) for tag in request.env['slide.channel.tag'].browse(tag_ids))

    def _channel_search_tags_ids(self, search_tags):
        """ Input: %5B4%5D """
        ChannelTag = request.env['slide.channel.tag']
        try:
            tag_ids = literal_eval(search_tags or '')
        except Exception:
            return ChannelTag
        # perform a search to filter on existing / valid tags implicitly
        return ChannelTag.search([('id', 'in', tag_ids)]) if tag_ids else ChannelTag

    def _channel_search_tags_slug(self, search_tags):
        """ Input: hotels-1,adventure-2 """
        ChannelTag = request.env['slide.channel.tag']
        try:
            tag_ids = list(filter(None, [request.env['ir.http']._unslug(tag)[1] for tag in (search_tags or '').split(',')]))
        except Exception:
            return ChannelTag
        # perform a search to filter on existing / valid tags implicitly
        return ChannelTag.search([('id', 'in', tag_ids)]) if tag_ids else ChannelTag

    def _create_or_get_channel_tag(self, tag_id, group_id):
        if not tag_id:
            return request.env['slide.channel.tag']
        # handle creation of new channel tag
        if tag_id[0] == 0:
            group_id = self._create_or_get_channel_tag_group_id(group_id)
            if not group_id:
                return {'error': _('Missing "Tag Group" for creating a new "Tag".')}

            return request.env['slide.channel.tag'].create({
                'name': tag_id[1]['name'],
                'group_id': group_id,
            })
        return request.env['slide.channel.tag'].browse(tag_id[0])

    def _create_or_get_channel_tag_group_id(self, group_id):
        if not group_id:
            return False
        # handle creation of new channel tag group
        if group_id[0] == 0:
            return request.env['slide.channel.tag.group'].create({
                'name': group_id[1]['name'],
            }).id
        # use existing channel tag group
        return group_id[0]

    # --------------------------------------------------
    # SLIDE.CHANNEL MAIN / SEARCH
    # --------------------------------------------------

    @http.route('/slides', type='http', auth="public", website=True, sitemap=True, readonly=True)
    def slides_channel_home(self, **post):
        """ Home page for eLearning platform. Is mainly a container page, does not allow search / filter. """
        channels_all = tools.lazy(lambda: request.env['slide.channel'].search(request.website.website_domain()))
        if not request.env.user._is_public():
            #If a course is completed, we don't want to see it in first position but in last
            channels_my = tools.lazy(lambda: channels_all.filtered(lambda channel: channel.is_member).sorted(lambda channel: 0 if channel.completed else channel.completion, reverse=True)[:3])
        else:
            channels_my = request.env['slide.channel']
        channels_popular = tools.lazy(lambda: channels_all.sorted('total_votes', reverse=True)[:3])
        channels_newest = tools.lazy(lambda: channels_all.sorted('create_date', reverse=True)[:3])

        achievements = tools.lazy(lambda: request.env['gamification.badge.user'].sudo().search([('badge_id.is_published', '=', True)], limit=5))
        if request.env.user._is_public():
            challenges = None
            challenges_done = None
        else:
            challenges = tools.lazy(lambda: request.env['gamification.challenge'].sudo().search([
                ('challenge_category', '=', 'slides'),
                ('reward_id.is_published', '=', True)
            ], order='id asc', limit=5))
            challenges_done = tools.lazy(lambda: request.env['gamification.badge.user'].sudo().search([
                ('challenge_id', 'in', challenges.ids),
                ('user_id', '=', request.env.user.id),
                ('badge_id.is_published', '=', True)
            ]).mapped('challenge_id'))

        users = tools.lazy(lambda: request.env['res.users'].sudo().search([
            ('karma', '>', 0),
            ('website_published', '=', True)], limit=5, order='karma desc'))

        render_values = self._slide_render_context_base()
        render_values.update(self._prepare_user_values(**post))
        render_values.update({
            'channels_my': channels_my,
            'channels_popular': channels_popular,
            'channels_newest': channels_newest,
            'achievements': achievements,
            'users': users,
            'top3_users': tools.lazy(self._get_top3_users),
            'challenges': challenges,
            'challenges_done': challenges_done,
            'search_tags': request.env['slide.channel.tag'],
            'slide_query_url': QueryURL('/slides/all', ['tag']),
            'slugify_tags': self._slugify_tags,
        })

        return request.render('website_slides.courses_home', render_values)

    def _get_slide_channel_search_options(self, my=None, slug_tags=None, slide_category=None, **post):
        return {
            'displayDescription': True,
            'displayDetail': False,
            'displayExtraDetail': False,
            'displayExtraLink': False,
            'displayImage': False,
            'allowFuzzy': not post.get('noFuzzy'),
            'my': my,
            'tag': slug_tags or post.get('tag'),
            'slide_category': slide_category,
        }

    @http.route(['/slides/all', '/slides/all/tag/<string:slug_tags>'], type='http', auth="public", website=True, sitemap=True, readonly=True)
    def slides_channel_all(self, slide_category=None, slug_tags=None, my=False, **post):
        if slug_tags and slug_tags.count(',') > 0 and request.httprequest.method == 'GET' and not post.get('prevent_redirect'):
            # Previously, the tags were searched using GET, which caused issues with crawlers (too many hits)
            # We replaced those with POST to avoid that, but it's not sufficient as bots "remember" crawled pages for a while
            # This permanent redirect is placed to instruct the bots that this page is no longer valid
            # TODO: remove in a few stable versions (v19?), including the "prevent_redirect" param in templates
            # Note: We allow a single tag to be GET, to keep crawlers & indexes on those pages
            # What we really want to avoid is combinatorial explosions
            return request.redirect('/slides/all', code=301)

        render_values = self.slides_channel_all_values(slide_category=slide_category, slug_tags=slug_tags, my=my, **post)
        return request.render('website_slides.courses_all', render_values)

    def slides_channel_all_values(self, slide_category=None, slug_tags=None, my=False, **post):
        """ Home page displaying a list of courses displayed according to some
        criterion and search terms.

          :param string slide_category: if provided, filter the course to contain at
           least one slide of type 'slide_category'. Used notably to display courses
           with certifications;
          :param string slug_tags: if provided, filter the slide.channels having
            the tag(s) (in comma separated slugified form);
          :param bool my: if provided, filter the slide.channels for which the
           current user is a member of
          :param dict post: post parameters, including

           * ``search``: filter on course description / name;
        """
        options = self._get_slide_channel_search_options(
            my=my,
            slug_tags=slug_tags,
            slide_category=slide_category,
            **post
        )
        search = post.get('search')
        order = self._channel_order_by_criterion.get(post.get('sorting'))
        search_count, details, fuzzy_search_term = request.website._search_with_fuzzy("slide_channels_only", search,
            limit=1000, order=order, options=options)
        channels = details[0].get('results', request.env['slide.channel'])

        tag_groups = request.env['slide.channel.tag.group'].search(
            ['&', ('tag_ids', '!=', False), ('website_published', '=', True)])
        if slug_tags:
            search_tags = self._channel_search_tags_slug(slug_tags)
        elif post.get('tags'):
            search_tags = self._channel_search_tags_ids(post['tags'])
        else:
            search_tags = request.env['slide.channel.tag']

        render_values = self._slide_render_context_base()
        render_values.update(self._prepare_user_values(**post))
        render_values.update({
            'channels': channels,
            'tag_groups': tag_groups,
            'search_term': fuzzy_search_term or search,
            'original_search': fuzzy_search_term and search,
            'search_slide_category': slide_category,
            'search_my': my,
            'search_tags': search_tags,
            'search_count': search_count,
            'top3_users': self._get_top3_users(),
            'slugify_tags': self._slugify_tags,
            'slide_query_url': QueryURL('/slides/all', ['tag']),
        })

        return render_values

    def _prepare_additional_channel_values(self, values, **kwargs):
        return values

    def _get_top3_users(self):
        return request.env['res.users'].sudo().search_read([
            ('karma', '>', 0),
            ('website_published', '=', True)], ['id'], limit=3, order='karma desc')

    def _get_user_slide_authorization(self, slide_id):
        """ Get authorization status for the current user to access the given slide along with some data.
        :return: Dict in the form:
        {
            'status': authorized|not_found|not_authorized,
            'slide': the slide corresponding to the slide_id (only if status != 'not_found')
            'channel_id': id of the channel containing the slide (only if status != 'not_found')
        }
        """
        status = 'authorized'
        try:
            slide = request.env['slide.slide'].browse(slide_id)
            slide.check_access('read')
        except (AccessError, MissingError):
            try:
                slide = request.env['slide.slide'].sudo().browse([slide_id])
            except MissingError:
                return {'status': 'not_found'}
            status = 'not_authorized'
        return {'status': status, 'slide': slide, 'channel_id': slide.sudo().channel_id.id}

    @http.route([
        '/slides/<int:channel_id>',
        '/slides/<int:channel_id>/category/<int:category_id>',
        '/slides/<int:channel_id>/category/<int:category_id>/page/<int:page>',
        '/slides/<model("slide.channel"):channel>',
        '/slides/<model("slide.channel"):channel>/page/<int:page>',
        '/slides/<model("slide.channel"):channel>/tag/<model("slide.tag"):tag>',
        '/slides/<model("slide.channel"):channel>/tag/<model("slide.tag"):tag>/page/<int:page>',
        '/slides/<model("slide.channel"):channel>/category/<model("slide.slide"):category>',
        '/slides/<model("slide.channel"):channel>/category/<model("slide.slide"):category>/page/<int:page>',
    ], type='http', auth="public", website=True, sitemap=sitemap_slide, handle_params_access_error=handle_wslide_error, readonly=True)
    def channel(self, channel=False, channel_id=False, category=None, category_id=False, tag=None, page=1, slide_category=None, uncategorized=False, sorting=None, search=None, **kw):
        """ Will return the rendered page of a course, with optional parameters allowing customization:

        :param channel: slide.channel to be rendered.
        :param channel_id: id of the rendered channel. (*)
        :param category: slide.slide (should be a category). Filter contents to those
            below this category (= section).
        :param category_id: id of the desired slide.slide category. (*)
        :param tag: slide.tag used to filter contents.
        :param slide_category: one of the values of linked selection field.
            Filter to this category of slides (video, article...)
        :param uncategorized: To set to True to access all slides outside of any slide.slide category.
        :param sorting: string defining the way to sort contents. ('most_voted', ...)
        :param search: string of the user search in the search bar.
        :param kw.invite_partner_id: id of the invited partner. (**)
        :param kw.invite_hash: string hash based on course and partner. (**)

        (*) Should be used for preview of invited attendees only. A 403 error could occur when using
            channel and category, if their access to models is denied. The generic shared course link
            uses channel_id as well, for the same reason.
        (**) Those are used to check and give invited attendees the access to the course and
            allow them browing its list of contents.
        """
        invite_partner_id = int(kw['invite_partner_id']) if kw.get('invite_partner_id') else False
        invite_hash = kw.get('invite_hash')
        valid_invite_values = {}

        # Invitation data processing
        if request.website.is_public_user() and invite_partner_id and invite_hash and channel_id and not channel:
            valid_invite_values = self._get_channel_values_from_invite(channel_id, invite_hash, invite_partner_id)
            if valid_invite_values.get('invite_preview'):
                channel = valid_invite_values.get('invite_channel')
                valid_invite_values['pager_args'] = {
                    'invite_hash': invite_hash,
                    'invite_partner_id': invite_partner_id
                }

        if channel_id < 0:
            # the string part of the channel "slugification" can be blank
            # meaning it can be "/slides/taking-care-of-trees-2" OR just "/slides/-2" if the first part is blank
            # as we use a IntConverter on the route definition, this will pick up a negative ID
            # (the IntConverter is necessary as we want a custom page in case the user can't access the course)
            channel_id = abs(channel_id)

        # Check access rights
        if channel_id and not channel:
            channel = request.env['slide.channel'].browse(channel_id).exists()
            if not channel:
                return self._redirect_to_slides_main('no_channel')
        if not channel.has_access('read'):
            return self._redirect_to_slides_main('no_rights')

        if category_id and not category:
            category = channel.slide_category_ids.filtered(lambda category: category.id == category_id)

        domain = self._get_channel_slides_base_domain(channel)
        pager_url = "/slides/%s" % (channel.id)
        pager_args = valid_invite_values.get('pager_args', {})
        slide_categories = dict(request.env['slide.slide']._fields['slide_category']._description_selection(request.env))

        if search:
            domain += [
                '|', '|',
                ('name', 'ilike', search),
                ('description', 'ilike', search),
                ('html_content', 'ilike', search)]
            pager_args['search'] = search
        else:
            if category:
                domain += [('category_id', '=', category.id)]
                pager_url += "/category/%s" % category.id
            elif tag:
                domain += [('tag_ids', '=', tag.id)]
                pager_url += "/tag/%s" % tag.id
            if uncategorized:
                domain += [('category_id', '=', False)]
                pager_args['uncategorized'] = 1
            elif slide_category:
                domain += [('slide_category', '=', slide_category)]
                pager_url += "?slide_category=%s" % slide_category

        # sorting criterion
        if channel.channel_type == 'documentation':
            default_sorting = 'latest' if channel.promote_strategy in ['specific', 'none', False] else channel.promote_strategy
            actual_sorting = sorting if sorting and sorting in request.env['slide.slide']._order_by_strategy else default_sorting
        else:
            actual_sorting = 'sequence'
        order = request.env['slide.slide']._order_by_strategy[actual_sorting]
        pager_args['sorting'] = actual_sorting

        slide_count = request.env['slide.slide'].sudo().search_count(domain)
        page_count = math.ceil(slide_count / self._slides_per_page)
        pager = request.website.pager(url=pager_url, total=slide_count, page=page,
                                      step=self._slides_per_page, url_args=pager_args,
                                      scope=page_count if page_count < self._pager_max_pages else self._pager_max_pages)

        query_string = None
        if category:
            query_string = "?search_category=%s" % category.id
        elif tag:
            query_string = "?search_tag=%s" % tag.id
        elif slide_category:
            query_string = "?search_slide_category=%s" % slide_category
        elif uncategorized:
            query_string = "?search_uncategorized=1"

        errors = {'access_error': False}
        if request.params.get('access_error') == 'course_content' and request.params.get('access_error_slide_id'):
            # Access are re-verified to support use case where the user refresh the page after an update of their access
            user_slide_authorization = self._get_user_slide_authorization(int(request.params.get('access_error_slide_id')))
            if user_slide_authorization['status'] == 'not_authorized':
                errors.update({
                    'access_error': 'course_content',
                    'access_error_content_name': request.params.get('access_error_slide_name'),
                })

        render_values = self._slide_render_context_base()
        render_values.update({
            'channel': channel,
            'main_object': channel,
            'active_tab': kw.get('active_tab', 'home'),
            # search
            'search_category': category,
            'search_tag': tag,
            'search_slide_category': slide_category,
            'search_uncategorized': uncategorized,
            'query_string': query_string,
            'slide_categories': slide_categories,
            'sorting': actual_sorting,
            'search': search,
            # display data
            'pager': pager,
            'slide_count': slide_count,
            # display upload modal
            'enable_slide_upload': kw.get('enable_slide_upload', False),
            # invitation data
            'invite_hash': invite_hash,
            'invite_partner_id': invite_partner_id,
            'invite_preview': valid_invite_values.get('invite_preview'),
            'is_partner_without_user': valid_invite_values.get('is_partner_without_user'),
            ** errors,
            ** self._slide_channel_prepare_review_values(channel),
        })

        # fetch slides and handle uncategorized slides; done as sudo because we want to display all
        # of them but unreachable ones won't be clickable (+ slide controller will crash anyway)
        # documentation mode may display less slides than content by category but overhead of
        # computation is reasonable
        if channel.promote_strategy == 'specific':
            render_values['slide_promoted'] = channel.sudo().promoted_slide_id
        else:
            render_values['slide_promoted'] = request.env['slide.slide'].sudo().search(domain, limit=1, order=order)

        limit_category_data = False
        if channel.channel_type == 'documentation':
            if category or uncategorized:
                limit_category_data = self._slides_per_page
            else:
                limit_category_data = self._slides_per_category

        render_values['category_data'] = channel._get_categorized_slides(
            domain, order,
            force_void=not category,
            limit=limit_category_data,
            offset=pager['offset'])
        render_values['channel_progress'] = self._get_channel_progress(channel, include_quiz=True)

        # for sys admins: prepare data to install directly modules from eLearning when
        # uploading slides. Currently supporting only survey, because why not.
        if request.env.user.has_group('base.group_system'):
            module = request.env.ref('base.module_survey')
            if module.state != 'installed':
                render_values['modules_to_install'] = json.dumps([{
                    'id': module.id,
                    'name': module.shortdesc,
                    'motivational': _('Want to test and certify your students?'),
                    'default_slide_category': 'certification',
                }])

        render_values = self._prepare_additional_channel_values(render_values, **kw)
        return request.render('website_slides.course_main', render_values)

    @staticmethod
    def _get_channel_values_from_invite(channel_id, invite_hash, invite_partner_id):
        """ Check identification parameters and returns values used to give access to signed out invited members.
        The course is returned as sudo to allow them seeing a preview of the course even if visibility if not public.
        Returns dict of values or containing 'invite_error' and a value corresponding to the error. See _get_invite_error_msg."""
        channel_sudo = request.env['slide.channel'].browse(channel_id).exists().sudo()
        partner_sudo = request.env['res.partner'].browse(invite_partner_id).exists().sudo()
        if not partner_sudo or not channel_sudo.is_published:
            return {'invite_error': 'no_partner' if not partner_sudo else 'no_channel' if not channel_sudo else 'no_rights'}

        channel_partner_sudo = channel_sudo.channel_partner_all_ids.filtered(lambda cp: cp.partner_id.id == invite_partner_id)
        if not channel_partner_sudo:
            return {'invite_error': 'expired'}
        if not consteq(channel_partner_sudo._get_invitation_hash(), invite_hash):
            return {'invite_error': 'hash_fail'}

        if channel_partner_sudo.member_status == 'invited':
            if not channel_partner_sudo.last_invitation_date or \
               channel_partner_sudo.last_invitation_date + relativedelta(months=3) < fields.Datetime.now():
                return {'invite_error': 'expired'}

        return {
            'invite_channel': channel_sudo,
            'invite_channel_partner': channel_partner_sudo,
            'invite_preview': True,
            'is_partner_without_user': not partner_sudo.user_ids,
            'invite_partner': partner_sudo
        }

    # SLIDE.CHANNEL UTILS
    # --------------------------------------------------

    @staticmethod
    def _redirect_to_slides_main(invite_error=''):
        return request.redirect(f"/slides?invite_error={invite_error}" if invite_error else "/slides")

    @staticmethod
    def _redirect_to_channel(channel):
        return request.redirect(f"/slides/{request.env['ir.http']._slug(channel)}")

    def _slide_channel_prepare_review_values(self, channel):
        values = {
            'rating_avg': channel.rating_avg,
            'rating_count': channel.rating_count,
        }

        if not request.env.user._is_public():
            subtype_comment_id = request.env['ir.model.data']._xmlid_to_res_id('mail.mt_comment')
            last_message = request.env['mail.message'].search([
                ('model', '=', channel._name),
                ('res_id', '=', channel.id),
                ('author_id', '=', request.env.user.partner_id.id),
                ('message_type', '=', 'comment'),
                ('subtype_id', '=', subtype_comment_id)
            ], order='write_date DESC', limit=1)

            if last_message:
                last_message_values = last_message.read(['body', 'rating_value', 'attachment_ids'])[0]
                last_message_attachment_ids = last_message_values.pop('attachment_ids', [])
                if last_message_attachment_ids:
                    # use sudo as portal user cannot read access_token, necessary for updating attachments
                    # through frontend chatter -> access is already granted and limited to current user message
                    last_message_attachment_ids = json.dumps(
                        request.env['ir.attachment'].sudo().browse(last_message_attachment_ids).read(
                            ['id', 'name', 'mimetype', 'file_size', 'access_token']
                        )
                    )
            else:
                last_message_values = {}
                last_message_attachment_ids = []

            values.update({
                'last_message_id': last_message_values.get('id'),
                'last_message': tools.html2plaintext(last_message_values.get('body', '')),
                'last_rating_value': last_message_values.get('rating_value'),
                'last_message_attachment_ids': last_message_attachment_ids,
            })
            if channel.can_review:
                values.update({
                    'message_post_hash': channel._sign_token(request.env.user.partner_id.id),
                    'message_post_pid': request.env.user.partner_id.id,
                })

        return values

    @http.route('/slides/<int:channel_id>/invite', type='http', auth='public', website=True, sitemap=False)
    def slide_channel_invite(self, channel_id, invite_partner_id, invite_hash):
        """ This route is included in the invitation link in email to join / check out the course. It is
        the main entry point on the attendee's side when sharing or inviting them. As rule of thumb, this will
        redirect to the course if the rights are given, and to the main /slides page with appropriate error
        message otherwise. (See _get_invite_error_msg method)

        It acts as a redirector:
            - Returns error if parameters are not valid or if expired invitation.
            - If a user is logged, verify the link is for this user. Redirects according to Acl's.
            - If no user is logged:
                - Redirects to login / signup if the partner is enrolled.
                - Redirects to the course with invite parameters. They will be able to browse a course preview
                before logging in / signing up, as prompted in an information banner.

        :param channel_id: The id of the course the user is invited to. Do not use <model> in the route instead,
            otherwise an error 403 could be returned if the (public) user has no access to the record.
        :param invite_partner_id: The id of the invited partner.
        :param invite_hash: The invitation hash that allows a direct access to channel_id, even if not connected.
        """
        channel = request.env['slide.channel'].browse(int(channel_id)).exists()
        if not channel:
            return self._redirect_to_slides_main('no_channel')

        # --- Compute rights of current user
        has_rights = channel.has_access('read')

        invite_values = self._get_channel_values_from_invite(channel_id, invite_hash, int(invite_partner_id))
        if invite_values.get('invite_error'):
            return self._redirect_to_channel(channel) if has_rights else self._redirect_to_slides_main(invite_values.get('invite_error'))

        invite_partner = invite_values.get('invite_partner')
        invite_channel_partner = invite_values.get('invite_channel_partner')

        # --- A user is logged
        if not request.website.is_public_user():
            if request.env.user.partner_id.id != invite_partner.id:
                return self._redirect_to_slides_main('partner_fail')
            return self._redirect_to_channel(channel) if has_rights else self._redirect_to_slides_main('no_rights')

        redirect_url = f'/slides/{channel_id}'

        # --- No user is logged.
        if invite_channel_partner.member_status != 'invited':
            # Enrolled partner. Access to the course but needs to log in / sign up.
            if invite_values.get('is_partner_without_user'):
                invite_partner.signup_prepare()
                signup_url = invite_partner._get_signup_url_for_action(url=redirect_url)[invite_partner.id]
                return request.redirect(signup_url)
            else:
                return request.redirect(f'/web/login?redirect={redirect_url}&auth_login={invite_partner.user_ids[0].login}')
        # Pending invitation. A banner will allow partner to login / signup on the course page.
        return request.redirect(f'{redirect_url}?invite_partner_id={invite_partner_id}&invite_hash={invite_hash}')

    @http.route(['/slides/<int:channel_id>/identify'], type='http', auth='public', website=True, sitemap=False)
    def slide_channel_identify_from_invite(self, channel_id, invite_partner_id, invite_hash):
        """ This route redirects invited partners when they click on the login / signup button, when they are
        asked to login / signup as invited to a course as public user on the course page preview. """
        if not request.website.is_public_user():
            return self._redirect_to_slides_main('identify_fail')

        invite_partner_id = int(invite_partner_id)
        invite_values = self._get_channel_values_from_invite(channel_id, invite_hash, invite_partner_id)
        if invite_values.get('invite_preview'):
            partner_sudo = invite_values.get('invite_partner')
            if invite_values.get('is_partner_without_user'):
                partner_sudo.signup_prepare()
                return request.redirect(partner_sudo._get_signup_url_for_action(url=f'/slides/{channel_id}')[partner_sudo.id])
            else:
                return request.redirect(f'/web/login?redirect=/slides/{channel_id}&auth_login={partner_sudo.user_ids[0].login}')
        return self._redirect_to_slides_main('identify_fail')

    @http.route(['/slides/channel/join'], type='json', auth='public', website=True)
    def slide_channel_join(self, channel_id):
        if request.website.is_public_user():
            return {
                'error': 'public_user',
                'error_signup_allowed': request.env['res.users'].sudo()._get_signup_invitation_scope() == 'b2c',
            }
        channel = request.env['slide.channel'].browse(channel_id)
        if channel.is_member_invited and channel.enroll == 'invite':
            success = channel.sudo()._action_add_members(request.env.user.partner_id)
        else:
            success = channel._action_add_members(request.env.user.partner_id)
        return {'error': 'join_done'} if not success else success

    @http.route(['/slides/channel/leave'], type='json', auth='user', website=True)
    def slide_channel_leave(self, channel_id):
        channel = request.env['slide.channel'].browse(channel_id)
        channel._remove_membership(request.env.user.partner_id.ids)
        self._channel_remove_session_answers(channel)
        return True

    @http.route(['/slides/channel/tag/search_read'], type='json', auth='user', methods=['POST'], website=True)
    def slide_channel_tag_search_read(self, fields, domain):
        can_create = request.env['slide.channel.tag'].has_access('create')
        return {
            'read_results': request.env['slide.channel.tag'].search_read(domain, fields),
            'can_create': can_create,
        }

    @http.route(['/slides/channel/tag/group/search_read'], type='json', auth='user', methods=['POST'], website=True)
    def slide_channel_tag_group_search_read(self, fields, domain):
        can_create = request.env['slide.channel.tag.group'].has_access('create')
        return {
            'read_results': request.env['slide.channel.tag.group'].search_read(domain, fields),
            'can_create': can_create,
        }

    @http.route('/slides/channel/tag/add', type='json', auth='user', methods=['POST'], website=True)
    def slide_channel_tag_add(self, channel_id, tag_id=None, group_id=None):
        """ Adds a slide channel tag to the specified slide channel.

        :param integer channel_id: Channel ID
        :param list tag_id: Channel Tag ID as first value of list. If id=0, then this is a new tag to
                            generate and expects a second list value of the name of the new tag.
        :param list group_id: Channel Tag Group ID as first value of list. If id=0, then this is a new
                              tag group to generate and expects a second list value of the name of the
                              new tag group. This value is required for when a new tag is being created.

        tag_id and group_id values are provided by a SelectMenu OWL component. Default "None" values
        allow for graceful failures in exceptional cases when values are not provided.

        :return: channel's course page
        """

        # handle exception during addition of course tag and send error notification to the client
        # otherwise client slide create dialog box continue processing even server fail to create a slide
        try:
            channel = request.env['slide.channel'].browse(int(channel_id))
            can_upload = channel.can_upload
            can_publish = channel.can_publish
        except UserError as e:
            _logger.error(e)
            return {'error': e.args[0]}
        else:
            if not can_upload or not can_publish:
                return {'error': _('You cannot add tags to this course.')}

        tag = self._create_or_get_channel_tag(tag_id, group_id)
        tag.write({'channel_ids': [(4, channel.id, 0)]})

        return {'url': "/slides/%s" % (request.env['ir.http']._slug(channel))}

    @http.route(['/slides/channel/send_share_email'], type='json', auth='user', website=True)
    def slide_channel_send_share_email(self, channel_id, emails):
        if not email_split(emails):
            return False
        channel = request.env['slide.channel'].browse(int(channel_id))
        channel._send_share_email(emails)
        return True

    @http.route(['/slides/channel/subscribe'], type='json', auth='user', website=True)
    def slide_channel_subscribe(self, channel_id):
        # Presentation Published subtype
        subtype = request.env.ref("website_slides.mt_channel_slide_published", raise_if_not_found=False)
        if subtype:
            return request.env['slide.channel'].browse(channel_id).message_subscribe(
                partner_ids=[request.env.user.partner_id.id], subtype_ids=subtype.ids)
        return True

    @http.route(['/slides/channel/unsubscribe'], type='json', auth='user', website=True)
    def slide_channel_unsubscribe(self, channel_id):
        request.env['slide.channel'].browse(channel_id).message_unsubscribe(partner_ids=[request.env.user.partner_id.id])
        return True

    # --------------------------------------------------
    # SLIDE.SLIDE MAIN / SEARCH
    # --------------------------------------------------

    @http.route('/slides/slide/<model("slide.slide"):slide>', type='http', auth="public",
                website=True, sitemap=True, handle_params_access_error=handle_wslide_error)
    def slide_view(self, slide, **kwargs):
        if not slide.channel_id.can_access_from_current_website() or not slide.active:
            raise werkzeug.exceptions.NotFound()
        # redirection to channel's homepage for category slides
        if slide.is_category:
            return request.redirect(slide.channel_id.website_url)

        if slide.can_self_mark_completed and not slide.user_has_completed \
           and slide.channel_id.channel_type == 'training' and slide.slide_category != 'video':
            self._slide_mark_completed(slide)
            next_category_to_open = slide._get_next_category()
        else:
            self._set_viewed_slide(slide)
            next_category_to_open = False

        values = self._get_slide_detail(slide)
        # quiz-specific: update with karma and quiz information
        if slide.question_ids:
            values.update(self._get_slide_quiz_data(slide))
        # sidebar: update with user channel progress
        values['channel_progress'] = self._get_channel_progress(slide.channel_id, include_quiz=True)
        # sidebar: auto-collapsed the categories depending on conditions
        values['category_data'] = self._prepare_collapsed_categories(values['category_data'], slide, next_category_to_open)

        # Allows to have breadcrumb for the previously used filter
        values.update({
            'search_category': slide.category_id if kwargs.get('search_category') else None,
            'search_tag': request.env['slide.tag'].browse(int(kwargs.get('search_tag'))) if kwargs.get('search_tag') else None,
            'slide_categories': dict(request.env['slide.slide']._fields['slide_category']._description_selection(request.env)) if kwargs.get('search_slide_category') else None,
            'search_slide_category': kwargs.get('search_slide_category'),
            'search_uncategorized': kwargs.get('search_uncategorized'),
        })

        values['channel'] = slide.channel_id
        values = self._prepare_additional_channel_values(values, **kwargs)
        values['signup_allowed'] = request.env['res.users'].sudo()._get_signup_invitation_scope() == 'b2c'

        if kwargs.get('fullscreen') == '1':
            values.update(self._slide_channel_prepare_review_values(slide.channel_id))
            return request.render("website_slides.slide_fullscreen", values)

        values.pop('channel', None)
        return request.render("website_slides.slide_main", values)

    @http.route('/slides/slide/<int:slide_id>/share', type='http', auth="public", website=True, sitemap=False)
    def slide_shared_view(self, slide_id, **kwargs):
        user_slide_authorization = self._get_user_slide_authorization(slide_id)
        status = user_slide_authorization['status']
        if status == 'not_found':
            raise werkzeug.exceptions.NotFound()

        if status == 'authorized':
            return request.redirect(
                '%s?%s' % (user_slide_authorization['slide'].website_url, werkzeug.urls.url_encode(kwargs)))

        channel_id = user_slide_authorization['channel_id']
        return request.redirect('/slides/%s?%s' % (channel_id, werkzeug.urls.url_encode({
            'access_error': 'course_content',
            'access_error_slide_id': slide_id,
            'access_error_slide_name': user_slide_authorization['slide'].name,
        })))

    @http.route('/slides/slide/<model("slide.slide"):slide>/pdf_content',
                type='http', auth="public", website=True, sitemap=False, handle_params_access_error=handle_wslide_error)
    def slide_get_pdf_content(self, slide):
        response = Response()
        response.data = slide.binary_content and base64.b64decode(slide.binary_content) or b''
        response.mimetype = 'application/pdf'
        return response

    @http.route('/slides/slide/<int:slide_id>/get_image', type='http', auth="public", website=True, sitemap=False)
    def slide_get_image(self, slide_id, field='image_128', width=0, height=0, crop=False):
        # Protect infographics by limiting access to 256px (large) images
        if field not in ('image_128', 'image_256', 'image_512', 'image_1024', 'image_1920'):
            return werkzeug.exceptions.Forbidden()

        slide = request.env['slide.slide'].sudo().browse(slide_id).exists()
        if not slide:
            raise werkzeug.exceptions.NotFound()

        return request.env['ir.binary']._get_image_stream_from(
            slide, field, width=int(width), height=int(height), crop=int(crop)
        ).get_response()

    # SLIDE.SLIDE UTILS
    # --------------------------------------------------

    @http.route('/slides/slide/get_html_content', type="json", auth="public", website=True)
    def get_html_content(self, slide_id):
        fetch_res = self._fetch_slide(slide_id)
        if fetch_res.get('error'):
            return fetch_res
        return {
            'html_content': request.env['ir.qweb.field.html'].record_to_html(fetch_res['slide'], 'html_content', {'template_options': {}})
        }

    @http.route('/slides/slide/<model("slide.slide"):slide>/set_completed',
                website=True, type="http", auth="user", handle_params_access_error=handle_wslide_error)
    def slide_set_completed_and_redirect(self, slide, next_slide_id=None):
        self._slide_mark_completed(slide)
        next_slide = None
        if next_slide_id:
            next_slide = self._fetch_slide(next_slide_id).get('slide', None)
        return request.redirect("/slides/slide/%s" % (request.env['ir.http']._slug(next_slide) if next_slide else request.env['ir.http']._slug(slide)))

    @http.route('/slides/slide/set_completed', website=True, type="json", auth="public")
    def slide_set_completed(self, slide_id):
        if request.website.is_public_user():
            return {'error': 'public_user'}
        fetch_res = self._fetch_slide(slide_id)
        if fetch_res.get('error'):
            return fetch_res
        self._slide_mark_completed(fetch_res['slide'])
        next_category = fetch_res['slide']._get_next_category()
        return {
            'channel_completion': fetch_res['slide'].channel_id.completion,
            'next_category_id': next_category.id if next_category else False,
        }

    @http.route('/slides/slide/<model("slide.slide"):slide>/set_uncompleted',
                website=True, type='http', auth='user', handle_params_access_error=handle_wslide_error)
    def slide_set_uncompleted_and_redirect(self, slide):
        self._slide_mark_uncompleted(slide)
        return request.redirect(f'/slides/slide/{request.env["ir.http"]._slug(slide)}')

    @http.route('/slides/slide/set_uncompleted', website=True, type='json', auth='public')
    def slide_set_uncompleted(self, slide_id):
        if request.website.is_public_user():
            return {'error': 'public_user'}
        fetch_res = self._fetch_slide(slide_id)
        if fetch_res.get('error'):
            return fetch_res
        self._slide_mark_uncompleted(fetch_res['slide'])
        return {
            'channel_completion': fetch_res['slide'].channel_id.completion,
            'next_category_id': False,
        }

    @http.route('/slides/slide/like', type='json', auth="public", website=True)
    def slide_like(self, slide_id, upvote):
        if request.website.is_public_user():
            return {'error': 'public_user', 'error_signup_allowed': request.env['res.users'].sudo()._get_signup_invitation_scope() == 'b2c'}
        # check slide access
        fetch_res = self._fetch_slide(slide_id)
        if fetch_res.get('error'):
            return fetch_res
        # check slide operation
        slide = fetch_res['slide']
        if not slide.channel_id.is_member:
            return {'error': 'channel_membership_required'}
        if not slide.channel_id.allow_comment:
            return {'error': 'channel_comment_disabled'}
        if not slide.channel_id.can_vote:
            return {'error': 'channel_karma_required'}
        if upvote:
            slide.action_like()
        else:
            slide.action_dislike()
        # for large number of likes/dislikes, format them so they don't break the UI
        # first display is done using a widget but this route updated the UI directly
        # hence calling format_decimalized_number
        return {
            'user_vote': slide.user_vote,
            'likes': tools.misc.format_decimalized_number(slide.likes),
            'dislikes': tools.misc.format_decimalized_number(slide.dislikes),
        }

    @http.route('/slides/slide/archive', type='json', auth='user', website=True)
    def slide_archive(self, slide_id):
        """ This route allows channel publishers to archive slides.
        It has to be done in sudo mode since only restricted_editors can write on slides in ACLs """
        slide = request.env['slide.slide'].browse(int(slide_id))
        if slide.channel_id.can_publish:
            slide.sudo().active = False
            return True

        return False

    @http.route('/slides/slide/toggle_is_preview', type='json', auth='user', website=True)
    def slide_preview(self, slide_id):
        slide = request.env['slide.slide'].browse(int(slide_id))
        if slide.channel_id.can_publish:
            slide.is_preview = not slide.is_preview
        return slide.is_preview

    @http.route(['/slides/slide/send_share_email'], type='json', auth='user', website=True)
    def slide_send_share_email(self, slide_id, emails, fullscreen=False):
        if not email_split(emails):
            return False
        slide = request.env['slide.slide'].browse(int(slide_id))
        slide._send_share_email(emails, fullscreen)
        return True

    # --------------------------------------------------
    # TAGS SECTION
    # --------------------------------------------------

    @http.route('/slide_channel_tag/add', type='json', auth='user', methods=['POST'], website=True)
    def slide_channel_tag_create_or_get(self, tag_id, group_id):
        tag = self._create_or_get_channel_tag(tag_id, group_id)
        return {'tag_id': tag.id}

    # --------------------------------------------------
    # QUIZ SECTION
    # --------------------------------------------------

    @http.route('/slides/slide/quiz/question_add_or_update', type='json', methods=['POST'], auth='user', website=True)
    def slide_quiz_question_add_or_update(self, slide_id, question, sequence, answer_ids, existing_question_id=None):
        """ Add a new question to an existing slide. Completed field of slide.partner
        link is set to False to make sure that the creator can take the quiz again.

        An optional question_id to udpate can be given. In this case question is
        deleted first before creating a new one to simplify management.

        :param integer slide_id: Slide ID
        :param string question: Question Title
        :param integer sequence: Question Sequence
        :param array answer_ids: Array containing all the answers :
                [
                    'sequence': Answer Sequence (Integer),
                    'text_value': Answer Title (String),
                    'is_correct': Answer Is Correct (Boolean)
                ]
        :param integer existing_question_id: question ID if this is an update

        :return: rendered question template
        """

        new_question_values = {
            'sequence': sequence,
            'question': question,
            'slide_id': slide_id,
            'answer_ids': [(0, 0, {
                'sequence': answer['sequence'],
                'text_value': answer['text_value'],
                'is_correct': answer['is_correct'],
                'comment': answer['comment']
            }) for answer in answer_ids]
        }

        try:
            # Attempt to create the question and validate the fields.
            # We want to return the error to have a nice display instead of the default mechanism
            # of exception handling that shows sticky toasters.
            # (Use a 'new' and not a create to avoid having to rollback anything if an error is
            # raised)
            slide_question = request.env['slide.question'].new(new_question_values)
            slide_question._validate_fields(new_question_values.keys())
        except ValidationError as e:
            return {'error': e.args[0]}

        fetch_res = self._fetch_slide(slide_id)
        if fetch_res.get('error'):
            return fetch_res
        slide = fetch_res['slide']
        if existing_question_id:
            request.env['slide.question'].search([
                ('slide_id', '=', slide.id),
                ('id', '=', int(existing_question_id))
            ]).unlink()

        request.env['slide.slide.partner'].search([
            ('slide_id', '=', slide_id),
            ('partner_id', '=', request.env.user.partner_id.id)
        ]).write({'completed': False})

        slide_question = request.env['slide.question'].create(new_question_values)
        return request.env['ir.qweb']._render('website_slides.lesson_content_quiz_question', {
            'slide': slide,
            'question': slide_question,
        })

    @http.route('/slides/slide/quiz/get', type="json", auth="public", website=True)
    def slide_quiz_get(self, slide_id):
        fetch_res = self._fetch_slide(slide_id)
        if fetch_res.get('error'):
            return fetch_res
        slide = fetch_res['slide']
        return self._get_slide_quiz_data(slide)

    @http.route('/slides/slide/quiz/reset', type="json", auth="user", website=True)
    def slide_quiz_reset(self, slide_id):
        fetch_res = self._fetch_slide(slide_id)
        if fetch_res.get('error'):
            return fetch_res
        request.env['slide.slide.partner'].search([
            ('slide_id', '=', fetch_res['slide'].id),
            ('partner_id', '=', request.env.user.partner_id.id)
        ]).write({'completed': False, 'quiz_attempts_count': 0})

    @http.route('/slides/slide/quiz/submit', type="json", auth="public", website=True)
    def slide_quiz_submit(self, slide_id, answer_ids):
        if request.website.is_public_user():
            return {'error': 'public_user'}
        fetch_res = self._fetch_slide(slide_id)
        if fetch_res.get('error'):
            return fetch_res
        slide = fetch_res['slide']

        if slide.user_has_completed:
            self._channel_remove_session_answers(slide.channel_id, slide)
            return {'error': 'slide_quiz_done'}

        all_questions = request.env['slide.question'].sudo().search([('slide_id', '=', slide.id)])

        user_answers = request.env['slide.answer'].sudo().search([('id', 'in', answer_ids)])
        if user_answers.mapped('question_id') != all_questions:
            return {'error': 'slide_quiz_incomplete'}

        user_bad_answers = user_answers.filtered(lambda answer: not answer.is_correct)

        self._set_viewed_slide(slide, quiz_attempts_inc=True)
        quiz_info = self._get_slide_quiz_partner_info(slide, quiz_done=True)

        rank_progress = {}
        if not user_bad_answers:
            rank_progress['previous_rank'] = self._get_rank_values(request.env.user)
            slide._action_mark_completed()
            rank_progress['new_rank'] = self._get_rank_values(request.env.user)
            rank_progress.update({
                'description': request.env.user.rank_id.description,
                'last_rank': not request.env.user._get_next_rank(),
                'level_up': rank_progress['previous_rank']['lower_bound'] != rank_progress['new_rank']['lower_bound']
            })
        self._channel_remove_session_answers(slide.channel_id, slide)
        return {
            'answers': {
                answer.question_id.id: {
                    'is_correct': answer.is_correct,
                    'comment': answer.comment
                } for answer in user_answers
            },
            'completed': slide.user_has_completed,
            'channel_completion': slide.channel_id.completion,
            'quizKarmaWon': quiz_info['quiz_karma_won'],
            'quizKarmaGain': quiz_info['quiz_karma_gain'],
            'quizAttemptsCount': quiz_info['quiz_attempts_count'],
            'rankProgress': rank_progress,
        }

    @http.route(['/slides/slide/quiz/save_to_session'], type='json', auth='public', website=True)
    def slide_quiz_save_to_session(self, quiz_answers):
        session_slide_answer_quiz = json.loads(request.session.get('slide_answer_quiz', '{}'))
        slide_id = quiz_answers['slide_id']
        session_slide_answer_quiz[str(slide_id)] = quiz_answers['slide_answers']
        request.session['slide_answer_quiz'] = json.dumps(session_slide_answer_quiz)

    def _get_rank_values(self, user):
        lower_bound = user.rank_id.karma_min or 0
        next_rank = user._get_next_rank()
        upper_bound = next_rank.karma_min
        progress = 100
        if next_rank and (upper_bound - lower_bound) != 0:
            progress = 100 * ((user.karma - lower_bound) / (upper_bound - lower_bound))
        return {
            'lower_bound': lower_bound,
            'upper_bound': upper_bound,
            'karma': user.karma,
            'motivational': next_rank.description_motivational,
            'progress': progress
        }
    # --------------------------------------------------
    # CATEGORY MANAGEMENT
    # --------------------------------------------------

    @http.route(['/slides/category/search_read'], type='json', auth='user', methods=['POST'], website=True)
    def slide_category_search_read(self, fields, domain):
        category_slide_domain = domain if domain else []
        category_slide_domain = expression.AND([category_slide_domain, [('is_category', '=', True)]])
        can_create = request.env['slide.slide'].has_access('create')
        return {
            'read_results': request.env['slide.slide'].search_read(category_slide_domain, fields),
            'can_create': can_create,
        }

    @http.route('/slides/category/add', type="http", website=True, auth="user", methods=['POST'])
    def slide_category_add(self, channel_id, name):
        """ Adds a category to the specified channel. Slide is added at the end
        of slide list based on sequence. """
        channel = request.env['slide.channel'].browse(int(channel_id))
        if not channel.can_upload or not channel.can_publish:
            raise werkzeug.exceptions.NotFound()

        request.env['slide.slide'].create(self._get_new_slide_category_values(channel, name))

        return request.redirect("/slides/%s" % (request.env['ir.http']._slug(channel)))

    # --------------------------------------------------
    # SLIDE.UPLOAD
    # --------------------------------------------------

    @http.route(['/slides/prepare_preview'], type='json', auth='user', methods=['POST'], website=True)
    def prepare_preview(self, channel_id, slide_category, url=None):
        """ Will attempt to fetch external metadata for this slide from the correct
        source (YouTube, Google Drive, ...).

        To take advantage of the slide business method, we create a temporary slide record before
        fetching the metadata.
        This allows a lot of code simplification, since we use "new", it will not created anything
        in database. """

        if not url:
            return {}

        Slide = request.env['slide.slide']

        additional_values = {}
        if slide_category == 'video':
            identical_video = request.env['slide.slide']
            existing_videos = Slide.search([
                ('channel_id', '=', int(channel_id)),
                ('slide_category', '=', 'video')
            ])

            slide = Slide.new({
                'channel_id': int(channel_id),
                'name': 'memory_record_for_computed_fields',
                'slide_category': 'video',
                'url': url
            })

            if not slide.video_source_type:
                slide.unlink()
                return {'error': _("Could not find your video. Please check if your link is correct and if the video can be accessed.")}

            if slide.video_source_type == 'youtube':
                identical_video = existing_videos.filtered(
                    lambda existing_video: slide.youtube_id == existing_video.youtube_id)
            elif slide.video_source_type == 'google_drive':
                identical_video = existing_videos.filtered(
                    lambda existing_video: slide.google_drive_id == existing_video.google_drive_id)
            elif slide.video_source_type == 'vimeo':
                identical_video = existing_videos.filtered(
                    lambda existing_video: slide.vimeo_id == existing_video.vimeo_id)
            if identical_video:
                identical_video_name = identical_video[0].name
                additional_values['info'] = _('This video already exists in this channel on the following content: %s', identical_video_name)
        elif slide_category in ['document', 'infographic']:
            slide = Slide.new({
                'channel_id': int(channel_id),
                'name': 'memory_record_for_computed_fields',
                'slide_category': slide_category,
                'source_type': 'external',
                'url': url
            })

            if not slide.google_drive_id:
                return {'error': _('Please enter valid Google Drive Link')}

        slide_values, error = slide._fetch_external_metadata(image_url_only=True)
        if error:
            return {'error': error}

        if additional_values:
            slide_values.update(additional_values)

        return slide_values

    @http.route(['/slides/add_slide'], type='json', auth='user', methods=['POST'], website=True)
    def create_slide(self, *args, **post):
        # check the size only when we upload a file.
        if post.get('binary_content'):
            file_size = len(post['binary_content']) * 3 / 4  # base64
            if (file_size / 1024.0 / 1024.0) > 25:
                return {'error': _('File is too big. File size cannot exceed 25MB')}

        values = dict((fname, post[fname]) for fname in self._get_valid_slide_post_values() if post.get(fname))

        # handle exception during creation of slide and sent error notification to the client
        # otherwise client slide create dialog box continue processing even server fail to create a slide
        try:
            channel = request.env['slide.channel'].browse(values['channel_id'])
            can_upload = channel.can_upload
            can_publish = channel.can_publish
        except UserError as e:
            _logger.error(e)
            return {'error': e.args[0]}
        else:
            if not can_upload:
                return {'error': _('You cannot upload on this channel.')}

        if post.get('duration'):
            # minutes to hours conversion
            values['completion_time'] = int(post['duration']) / 60

        category = False
        # handle creation of new categories on the fly
        if post.get('category_id'):
            category_id = post['category_id'][0]
            if category_id == 0:
                category = request.env['slide.slide'].create(self._get_new_slide_category_values(channel, post['category_id'][1]['name']))
                values['sequence'] = category.sequence + 1
            else:
                category = request.env['slide.slide'].browse(category_id)
                values.update({
                    'sequence': request.env['slide.slide'].browse(post['category_id'][0]).sequence + 1
                })

        # create slide itself
        try:
            values['user_id'] = request.env.uid
            slide = request.env['slide.slide'].sudo().create(values)
        except UserError as e:
            _logger.error(e)
            return {'error': e.args[0]}
        except Exception as e:
            _logger.error(e)
            return {'error': _('Internal server error, please try again later or contact administrator.\nHere is the error message: %s', e)}

        # ensure correct ordering by re sequencing slides in front-end (backend should be ok thanks to list view)
        channel._resequence_slides(slide, force_category=category)

        redirect_url = "/slides/slide/%s" % (slide.id)
        if slide.slide_category == 'article':
            redirect_url = request.env["website"].get_client_action_url(redirect_url, True)
        elif slide.slide_category == 'quiz':
            redirect_url += "?quiz_quick_create"
        elif channel.channel_type == "training":
            redirect_url = "/slides/%s" % (request.env['ir.http']._slug(channel))
        return {
            'url': redirect_url,
            'channel_type': channel.channel_type,
            'slide_id': slide.id,
            'category_id': slide.category_id
        }

    def _get_valid_slide_post_values(self):
        return ['name', 'url', 'video_url', 'document_google_url', 'image_google_url', 'tag_ids', 'slide_category', 'channel_id',
            'is_preview', 'binary_content', 'description', 'image_1920', 'is_published', 'source_type']

    @http.route(['/slides/tag/search_read'], type='json', auth='user', methods=['POST'], website=True)
    def slide_tag_search_read(self, fields, domain):
        can_create = request.env['slide.tag'].has_access('create')
        return {
            'read_results': request.env['slide.tag'].search_read(domain, fields),
            'can_create': can_create,
        }

    # --------------------------------------------------
    # EMBED IN THIRD PARTY WEBSITES
    # --------------------------------------------------

    @http.route('/slides/embed/<int:slide_id>', type='http', auth='public', website=True, sitemap=False)
    def slides_embed(self, slide_id, page="1", **kw):
        return self._slide_embed(slide_id, page=page, is_external_embed=False, **kw)

    @http.route('/slides/embed_external/<int:slide_id>', type='http', auth='public', website=True, sitemap=False)
    def slides_embed_external(self, slide_id, page="1", **kw):
        return self._slide_embed(slide_id, page=page, is_external_embed=True, **kw)

    def _slide_embed(self, slide_id, page="1", is_external_embed=False, **kw):
        """ Note : don't use the 'model' in the route (use 'slide_id'), otherwise if public cannot
        access the embedded slide, the error will be the website.403 page instead of the one of the
        website_slides.embed_slide.

        Do not forget the rendering here will be displayed in the embedded iframe

        Try accessing slide, and display to corresponding template.

        When the content is embedded *externally*, meaning on a third party website, we do some
        additional steps like displaying sharing controls and also updating some KPIs. """

        try:
            slide = request.env['slide.slide'].browse(slide_id)
            if not slide.exists() or not slide.sudo().active:
                raise werkzeug.exceptions.NotFound()

            referer_url = request.httprequest.headers.get('Referer', '')
            if is_external_embed:
                slide.sudo()._embed_increment(referer_url)

            values = self._get_slide_detail(slide)
            values['page'] = page
            values['is_external_embed'] = is_external_embed
            self._set_viewed_slide(slide)
            return request.render('website_slides.embed_slide', values)
        except AccessError: # TODO : please, make it clean one day, or find another secure way to detect
                            # if the slide can be embedded, and properly display the error message.
            return request.render('website_slides.embed_slide_forbidden', {})

    # --------------------------------------------------
    # PROFILE
    # --------------------------------------------------

    def _prepare_user_values(self, **kwargs):
        values = super(WebsiteSlides, self)._prepare_user_values(**kwargs)
        invite_error_msg = self._get_invite_error_msg(kwargs.get('invite_error'))
        if invite_error_msg:
            values['invite_error_msg'] = invite_error_msg

        channel = self._get_channels(**kwargs)
        if channel:
            values['channel'] = channel
        return values

    def _get_channels(self, **kwargs):
        channels = []
        if kwargs.get('channel'):
            channels = kwargs['channel']
        elif kwargs.get('channel_id'):
            channels = tools.lazy(lambda: request.env['slide.channel'].browse(int(kwargs['channel_id'])))
        return channels

    @staticmethod
    def _get_invite_error_msg(invite_error):
        return {
            'expired': _('This invitation link has expired.'),
            'hash_fail': _('This invitation link has an invalid hash.'),
            'identify_fail': _('This identification link does not seem to be valid.'),
            'no_channel': _('This course does not exist.'),
            'no_partner': _('The contact associated with this invitation does not seem to be valid.'),
            'no_rights': _('You do not have permission to access this course.'),
            'partner_fail': _('This invitation link is not for this contact.'),
        }.get(invite_error, '')

    def _prepare_user_slides_profile(self, user):
        courses = request.env['slide.channel.partner'].sudo().search([('partner_id', '=', user.partner_id.id), ('member_status', '!=', 'invited')])
        courses_completed = courses.filtered(lambda c: c.member_status == 'completed')
        courses_ongoing = courses - courses_completed
        values = {
            'uid': request.env.user.id,
            'user': user,
            'main_object': user,
            'courses_completed': courses_completed,
            'courses_ongoing': courses_ongoing,
            'is_profile_page': True,
            'badge_category': 'slides',
            'my_profile': request.env.user.id == user.id,
        }
        return values

    def _prepare_user_profile_values(self, user, **post):
        values = super(WebsiteSlides, self)._prepare_user_profile_values(user, **post)
        if post.get('channel_id'):
            values.update({'edit_button_url_param': 'channel_id=' + str(post['channel_id'])})
        channels = self._get_channels(**post)
        if not channels:
            channels = request.env['slide.channel'].search([])
        values.update(self._prepare_user_values(channel=channels[0] if len(channels) == 1 else True, **post))
        values.update(self._prepare_user_slides_profile(user))
        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main
from . import mail

```

## File: data\gamification_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Get started: register to the platform -->
    <record id="badge_data_register" model="gamification.badge">
        <field name="name">Get started</field>
        <field name="description">Register to the platform</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/standard_badge_bronze.svg"/>
        <field name="is_published" eval="True"/>
        <field name="level">bronze</field>
        <field name="rule_auth">nobody</field>
    </record>
    <record id="badge_data_register_goal" model="gamification.goal.definition">
        <field name="name">Get started</field>
        <field name="description">Register to the platform</field>
        <field name="computation_mode">count</field>
        <field name="display_mode">boolean</field>
        <field name="model_id" ref="base.model_res_users"/>
        <field name="condition">higher</field>
        <field name="domain">[
            ('active', '!=', False),
            ('karma', '>', 0),
        ]</field>
        <field name="batch_mode">True</field>
        <field name="batch_distinctive_field" ref="base.field_res_users__id"/>
        <field name="batch_user_expression">user.id</field>
    </record>
    <record id="badge_data_register_challenge" model="gamification.challenge">
        <field name="name">Register to the platform</field>
        <field name="challenge_category">slides</field>
        <field name="period">once</field>
        <field name="visibility_mode">personal</field>
        <field name="report_message_frequency">never</field>
        <field name="reward_id" ref="badge_data_register"/>
        <field name="reward_realtime">True</field>
        <field name="user_domain">[('karma', '>', 0)]</field>
        <field name="state">inprogress</field>
    </record>
    <record id="badge_data_register_challenge_line_0" model="gamification.challenge.line">
        <field name="definition_id" ref="badge_data_register_goal"/>
        <field name="challenge_id" ref="badge_data_register_challenge"/>
        <field name="target_goal">1</field>
    </record>

    <!-- Know yourself: complete your profile -->
    <record id="badge_data_profile" model="gamification.badge">
        <field name="name">Know yourself</field>
        <field name="description">Complete your profile</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/standard_badge_bronze.svg"/>
        <field name="is_published" eval="True"/>
        <field name="level">bronze</field>
        <field name="rule_auth">nobody</field>
    </record>
    <record id="badge_data_profile_goal" model="gamification.goal.definition">
        <field name="name">Know yourself</field>
        <field name="description">Complete your profile</field>
        <field name="computation_mode">count</field>
        <field name="display_mode">boolean</field>
        <field name="model_id" ref="base.model_res_users"/>
        <field name="condition">higher</field>
        <field name="domain">[
            ('partner_id.country_id', '!=', False),
            ('partner_id.city', '!=', False),
            ('partner_id.email', '!=', False)
        ]</field>
        <field name="batch_mode">True</field>
        <field name="batch_distinctive_field" ref="base.field_res_users__id"/>
        <field name="batch_user_expression">user.id</field>
    </record>
    <record id="badge_data_profile_challenge" model="gamification.challenge">
        <field name="name">Complete your profile</field>
        <field name="challenge_category">slides</field>
        <field name="period">once</field>
        <field name="visibility_mode">personal</field>
        <field name="report_message_frequency">never</field>
        <field name="reward_id" ref="badge_data_profile"/>
        <field name="reward_realtime">True</field>
        <field name="user_domain">[('karma', '>', 0)]</field>
        <field name="state">inprogress</field>
    </record>
    <record id="badge_data_profile_challenge_line_0" model="gamification.challenge.line">
        <field name="definition_id" ref="badge_data_profile_goal"/>
        <field name="challenge_id" ref="badge_data_profile_challenge"/>
        <field name="target_goal">1</field>
    </record>

    <!-- Power User: complete a course -->
    <record id="badge_data_course" model="gamification.badge">
        <field name="name">Power User</field>
        <field name="description">Complete a course</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/standard_badge_silver.svg"/>
        <field name="is_published" eval="True"/>
        <field name="level">silver</field>
        <field name="rule_auth">nobody</field>
    </record>
    <record id="badge_data_course_goal" model="gamification.goal.definition">
        <field name="name">Power User</field>
        <field name="description">Complete a course</field>
        <field name="computation_mode">count</field>
        <field name="display_mode">boolean</field>
        <field name="model_id" ref="website_slides.model_slide_channel_partner"/>
        <field name="condition">higher</field>
        <field name="domain">[
            ('member_status', '=', 'completed')
        ]</field>
        <field name="batch_mode">True</field>
        <field name="batch_distinctive_field" ref="website_slides.field_slide_channel_partner__partner_id"/>
        <field name="batch_user_expression">user.partner_id.id</field>
    </record>
    <record id="badge_data_course_challenge" model="gamification.challenge">
        <field name="name">Complete a course</field>
        <field name="challenge_category">slides</field>
        <field name="period">once</field>
        <field name="visibility_mode">personal</field>
        <field name="report_message_frequency">never</field>
        <field name="reward_id" ref="badge_data_course"/>
        <field name="reward_realtime">True</field>
        <field name="user_domain">[('karma', '>', 0)]</field>
        <field name="state">inprogress</field>
    </record>
    <record id="badge_data_course_challenge_line_0" model="gamification.challenge.line">
        <field name="definition_id" ref="badge_data_course_goal"/>
        <field name="challenge_id" ref="badge_data_course_challenge"/>
        <field name="target_goal">1</field>
    </record>

    <!-- Certified Knowledge: get a certification -->
    <record id="badge_data_certification" model="gamification.badge">
        <field name="name">Certified Knowledge</field>
        <field name="description">Get a certification</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/standard_badge_gold.svg"/>
        <field name="is_published" eval="False"/>
        <field name="level">gold</field>
        <field name="rule_auth">nobody</field>
    </record>
    <record id="badge_data_certification_goal" model="gamification.goal.definition">
        <field name="name">Certified Knowledge</field>
        <field name="description">Get a certification</field>
        <field name="computation_mode">count</field>
        <field name="display_mode">boolean</field>
        <field name="model_id" ref="website_slides.model_slide_slide_partner"/>
        <field name="condition">higher</field>
        <field name="domain">[
            ('completed', '=', True),
            (0, '=', 1)
        ]</field>
        <field name="batch_mode">True</field>
        <field name="batch_distinctive_field" ref="website_slides.field_slide_slide_partner__partner_id"/>
        <field name="batch_user_expression">user.partner_id.id</field>
    </record>
    <record id="badge_data_certification_challenge" model="gamification.challenge">
        <field name="name">Get a certification</field>
        <field name="challenge_category">slides</field>
        <field name="period">once</field>
        <field name="visibility_mode">personal</field>
        <field name="report_message_frequency">never</field>
        <field name="reward_id" ref="badge_data_certification"/>
        <field name="reward_realtime">True</field>
        <field name="user_domain">[('karma', '>', 0)]</field>
        <field name="state">inprogress</field>
    </record>
    <record id="badge_data_certification_challenge_line_0" model="gamification.challenge.line">
        <field name="definition_id" ref="badge_data_certification_goal"/>
        <field name="challenge_id" ref="badge_data_certification_challenge"/>
        <field name="target_goal">1</field>
    </record>

    <!-- Community hero: reach 2000 XP -->
    <record id="badge_data_karma" model="gamification.badge">
        <field name="name">Community hero</field>
        <field name="description">Reach 2000 XP</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/standard_badge_gold.svg"/>
        <field name="is_published" eval="True"/>
        <field name="level">gold</field>
        <field name="rule_auth">nobody</field>
    </record>
    <record id="badge_data_karma_goal" model="gamification.goal.definition">
        <field name="name">Community hero</field>
        <field name="description">Reach 2000 XP</field>
        <field name="computation_mode">count</field>
        <field name="display_mode">boolean</field>
        <field name="model_id" ref="base.model_res_users"/>
        <field name="condition">higher</field>
        <field name="domain">[
            ('karma', '>=', 2000)
        ]</field>
        <field name="batch_mode">True</field>
        <field name="batch_distinctive_field" ref="base.field_res_users__id"/>
        <field name="batch_user_expression">user.id</field>
    </record>
    <record id="badge_data_karma_challenge" model="gamification.challenge">
        <field name="name">Reach 2000 XP</field>
        <field name="challenge_category">slides</field>
        <field name="period">once</field>
        <field name="visibility_mode">personal</field>
        <field name="report_message_frequency">never</field>
        <field name="reward_id" ref="badge_data_karma"/>
        <field name="reward_realtime">True</field>
        <field name="user_domain">[('karma', '>', 0)]</field>
        <field name="state">inprogress</field>
    </record>
    <record id="badge_data_karma_challenge_line_0" model="gamification.challenge.line">
        <field name="definition_id" ref="badge_data_karma_goal"/>
        <field name="challenge_id" ref="badge_data_karma_challenge"/>
        <field name="target_goal">1</field>
    </record>
</odoo>

```

## File: data\mail_activity_type_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <record id="mail_activity_data_access_request" model="mail.activity.type">
        <field name="name">Access Request</field>
        <field name="icon">fa-check-circle</field>
        <field name="sequence">50</field>
        <field name="res_model">slide.channel</field>
    </record>
</data></odoo>

```

## File: data\mail_message_subtype_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!-- Channel subtypes -->
    <record id="mt_channel_slide_published" model="mail.message.subtype">
        <field name="name">Presentation Published</field>
        <field name="res_model">slide.channel</field>
        <field name="default" eval="True"/>
        <field name="description">Presentation Published</field>
    </record>
</data></odoo>

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
        <!-- QWeb templates -->
        <!-- Note: mail_notification_channel_invite: record should be a slide.channel.partner record -->
        <template id="mail_notification_channel_invite">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 24px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="100%" style="background-color: white; padding: 0; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Your <t t-esc="model_description or 'document'"/></span><br/>
                    <span style="font-size: 20px; font-weight: bold;">
                        <t t-esc="message.record_name and message.record_name.replace('/','-') or ''"/>
                    </span>
                </td><td valign="middle" align="right" t-if="company and not company.uses_default_logo">
                    <img t-att-src="'/logo.png?company=%s' % company.id" style="padding: 0px; margin: 0px; height: 48px;" t-att-alt="'%s' % company.name"/>
                </td></tr>
                <tr><td colspan="2" style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin:4px 0px 32px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- CONTENT -->
    <tr>
        <td style="min-width: 590px;">
            <t t-out="message.body"/>
            <div style="margin: 32px 0px 32px 0px; text-align: center;">
                <a t-att-href="record.invitation_link"
                    style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                    <t t-if="enroll_mode"> Click here to start the course </t> <t t-else=""> Click here to get started </t>
                </a>
            </div>
            <div style="margin: 0px; padding: 0px; font-size:13px;">
                Enjoy this exclusive content!
            </div>
            <div>&amp;nbsp;</div>
            <div t-if="signature" style="font-size: 13px;">
                <div t-out="signature"/>
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
</data></odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
         <record id="slide_template_published" model="mail.template">
            <field name="name">Elearning: New Course Content Notification</field>
            <field name="model_id" ref="model_slide_slide"/>
            <field name="subject">New {{ object.slide_category }} published on {{ object.channel_id.name }}</field>
            <field name="description">Sent to attendees when new course is published</field>
            <field name="body_html" type="html">
                <div style="margin: 0px; padding: 0px;">
                    <p style="margin: 0px; padding: 0px; font-size: 13px;">
                        Hello<br/><br/>
                        There is something new in the course <strong t-out="object.channel_id.name or ''">Trees, Wood and Gardens</strong> you are following:<br/><br/>
                        <center><strong t-out="object.name or ''">Trees</strong></center>
                        <t t-if="object.image_1024">
                            <div style="padding: 16px 8px 16px 8px; text-align: center;">
                                <a t-att-href="object.website_share_url">
                                <img t-att-alt="object.name" t-attf-src="{{ ctx.get('base_url') }}/web/image/slide.slide/{{ object.id }}/image_1024" style="height:auto; width:150px; margin: 16px;"/>
                            </a>
                        </div>
                        </t>
                        <div style="padding: 16px 8px 16px 8px; text-align: center;">
                            <a t-att-href="object.website_share_url"
                                style="background-color: #875a7b; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px;">View content</a>
                        </div>
                        Enjoy this exclusive content!
                        <t t-if="user.signature">
                            <br />
                            <t t-out="user.signature or ''">--<br/>Mitchell Admin</t>
                        </t>
                    </p>
                </div>
            </field>
            <field name="auto_delete" eval="True"/>
        </record>

        <record id="slide_template_shared" model="mail.template">
            <field name="name">Elearning: Course Share</field>
            <field name="model_id" ref="model_slide_slide"/>
            <field name="subject">{{ user.name }} shared a {{ object.slide_category }} with you!</field>
            <field name="email_from">{{ user.email_formatted }}</field>
            <field name="email_to">{{ ctx.get('email', '') }}</field>
            <field name="description">Sent when attendees share the course by email</field>
            <field name="body_html" type="html">
                <div style="margin: 0px; padding: 0px;">
                    <p style="margin: 0px; padding: 0px; font-size: 13px;">
                        Hello<br/><br/>
                        <t t-out="user.name or ''">Mitchell Admin</t> shared the <t t-out="object.slide_category or ''">document</t> <strong t-out="object.name or ''">Trees</strong> with you!
                        <div style="margin: 16px 8px 16px 8px; text-align: center;">
                            <a t-att-href="(object.website_share_url + '?fullscreen=1') if ctx.get('fullscreen') else object.website_share_url">
                                <img t-att-alt="object.name" t-attf-src="{{ ctx.get('base_url') }}/web/image/slide.slide/{{ object.id }}/image_1024" style="height:auto; width:150px; margin: 16px;"/>
                            </a>
                        </div>
                        <div style="padding: 16px 8px 16px 8px; text-align: center;">
                            <a t-att-href="(object.website_share_url + '?fullscreen=1') if ctx.get('fullscreen') else object.website_share_url"
                                style="background-color: #875a7b; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px;">View <strong t-out="object.name or ''">Trees</strong></a>
                        </div>
                        <t t-if="user.signature">
                            <br />
                            <t t-out="user.signature or ''">--<br/>Mitchell Admin</t>
                        </t>
                    </p>
                </div>
            </field>
            <field name="auto_delete" eval="True"/>
        </record>

        <!-- Completed Channel Message -->
        <record id="mail_template_channel_completed" model="mail.template">
            <field name="name">Elearning: Completed Course</field>
            <field name="model_id" ref="model_slide_channel_partner"/>
            <field name="subject">Congratulations! You completed {{ object.channel_id.name }}</field>
            <field name="email_from">{{ (object.channel_id.user_id.email_formatted or object.channel_id.user_id.company_id.catchall_formatted) }}</field>
            <field name="partner_to">{{ object.partner_id.id }}</field>
            <field name="description">Sent to attendees once they've completed the course</field>
            <field name="body_html" type="html">
                <div style="margin: 0px; padding: 0px;">
                    <div style="margin: 0px; padding: 0px; font-size: 13px;">
                        <p style="margin: 0px;">Hello <t t-out="object.partner_id.name or ''">Brandon Freeman</t>,</p><br/>
                        <p><b>Congratulations!</b></p>
                        <p>You've completed the course <b t-out="object.channel_id.name or ''">Basics of Gardening</b></p>
                        <p>Check out the other available courses.</p><br/>

                        <div style="padding: 16px 8px 16px 8px; text-align: center;">
                            <a href="/slides/all" style="background-color: #875a7b; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px;">
                                Explore courses
                            </a>
                        </div>
                        Enjoy this exclusive content!
                        <t t-if="object.channel_id.user_id.signature">
                            <br />
                            <t t-out="object.channel_id.user_id.signature or ''">--<br/>Mitchell Admin</t>
                        </t>
                    </div>
                </div>
            </field>
            <field name="auto_delete" eval="True"/>
            <field name="lang">{{ object.partner_id.lang }}</field>
        </record>

        <record id="mail_template_channel_shared" model="mail.template">
            <field name="name">Channel Shared</field>
            <field name="model_id" ref="model_slide_channel"/>
            <field name="subject">{{ user.name }} shared a Course</field>
            <field name="email_from">{{ user.email_formatted }}</field>
            <field name="body_html" type="html">
                <div style="margin: 0px; padding: 0px;">
                    <p style="margin: 0px; padding: 0px; font-size: 13px;">
                        Hello<br/><br/>
                        <t t-out="user.name or ''">Mitchell Admin</t> shared the <strong t-out="object.name or ''">document</strong> with you!
                        <div style="margin: 16px 8px 16px 8px; text-align: center;">
                            <a t-att-href="object.website_url">
                                <img t-att-alt="object.name"
                                    t-attf-src="{{ ctx.get('base_url') }}/web/image/slide.channel/{{ object.id }}/image_256"
                                    style="height:auto; width:150px; margin: 16px;"/>
                            </a>
                        </div>
                        <div style="padding: 16px 8px 16px 8px; text-align: center;">
                            <a t-att-href="object.website_url"
                                style="background-color: #875a7b; padding: 8px 16px 8px 16px;
                                text-decoration: none; color: #fff; border-radius: 5px;">
                                View <strong t-out="object.name or ''">Document</strong></a>
                        </div>
                        <t t-if="user.signature">
                            <br />
                            <t t-out="user.signature or ''">--<br/>Mitchell Admin</t>
                        </t>
                    </p>
                </div>
            </field>
            <field name="auto_delete" eval="True"/>
        </record>

        <!-- Slide channel invite feature -->
        <record id="mail_template_slide_channel_enroll" model="mail.template">
            <field name="name">Elearning: Add Attendees to Course</field>
            <field name="model_id" ref="model_slide_channel_partner" />
            <field name="subject">You have been invited to join {{ object.channel_id.name }}</field>
            <field name="email_from">{{ user.email_formatted }}</field>
            <field name="use_default_to" eval="True"/>
            <field name="description">Sent to attendees when they are added to a course</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px; font-size: 13px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        Hello<br/><br/>
        You have been enrolled to a new course: <t t-out="object.channel_id.name or ''">Basics of Gardening</t>.
    </p>
</div>
            </field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <!-- Slide channel sharing feature -->
        <record id="mail_template_slide_channel_invite" model="mail.template">
            <field name="name">Elearning: Promotional Course Invitation</field>
            <field name="model_id" ref="model_slide_channel_partner" />
            <field name="subject">You have been invited to check out {{ object.channel_id.name }}</field>
            <field name="email_from">{{ user.email_formatted }}</field>
            <field name="use_default_to" eval="True"/>
            <field name="description">Sent to potential attendees to check out the course.</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px; font-size: 13px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        Hello<br/><br/>
        You have been invited to check out this course: <t t-out="object.channel_id.name or ''">Basics of Gardening</t>.
    </p>
</div>
            </field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: data\res_users_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="website_slides.group_website_slides_officer" model="res.groups">
            <field name="users" eval="[(4, ref('base.user_demo'))]"/>
        </record>

        <record id="website_slides.group_website_slides_manager" model="res.groups">
            <field name="users" eval="[(4, ref('base.user_admin')), (4, ref('base.user_root'))]"/>
        </record>
    </data>
</odoo>

```

## File: data\slides_tour.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="slides_tour" model="web_tour.tour">
        <field name="name">slides_tour</field>
        <field name="url"></field>
    </record>
</odoo>

```

## File: data\slide_channel_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <!-- This channel will gain a forum -->
    <record id="slide_channel_demo_0_gard_0" model="slide.channel">
        <field name="name">Basics of Gardening</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="enroll">public</field>
        <field name="channel_type">training</field>
        <field name="allow_comment" eval="True"/>
        <field name="promote_strategy">most_voted</field>
        <field name="is_published" eval="True"/>
        <field name="tag_ids" eval="[(5, 0),
                                     (4, ref('website_slides.slide_channel_tag_level_basic')),
                                     (4, ref('website_slides.slide_channel_tag_role_gardener')),
                                     (4, ref('website_slides.slide_channel_tag_other_0')),
                                     (4, ref('website_slides.slide_channel_tag_other_2'))]"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/channel_demo_gardening.jpg"/>
        <field name="description">Learn the basics of gardening!</field>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=8)"/>
    </record>

    <!-- This channel will be set on payment -->
    <record id="slide_channel_demo_1_gard1" model="slide.channel">
        <field name="name">Taking care of Trees</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="enroll">public</field>
        <field name="channel_type">training</field>
        <field name="allow_comment" eval="True"/>
        <field name="promote_strategy">latest</field>
        <field name="is_published" eval="True"/>
        <field name="tag_ids" eval="[(5, 0),
                                     (4, ref('website_slides.slide_channel_tag_level_intermediate')),
                                     (4, ref('website_slides.slide_channel_tag_role_gardener')),
                                     (4, ref('website_slides.slide_channel_tag_other_0'))]"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/channel_demo_gardening_2.jpg"/>
        <field name="description">Learn how to take care of your favorite trees. Learn when to plant, how to manage potted trees, ...</field>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=7)"/>
    </record>

    <!-- This channel will gain a forum -->
    <record id="slide_channel_demo_2_gard2" model="slide.channel">
        <field name="name">Trees, Wood and Gardens</field>
        <field name="user_id" ref="base.user_demo"/>
        <field name="prerequisite_channel_ids" eval="[(5, 0),
                                                      (4, ref('website_slides.slide_channel_demo_0_gard_0')),
                                                      (4, ref('website_slides.slide_channel_demo_1_gard1'))]"/>
        <field name="enroll">public</field>
        <field name="channel_type">documentation</field>
        <field name="allow_comment" eval="True"/>
        <field name="promote_strategy">most_viewed</field>
        <field name="is_published" eval="True"/>
        <field name="tag_ids" eval="[(5, 0),
                                     (4, ref('website_slides.slide_channel_tag_level_intermediate')),
                                     (4, ref('website_slides.slide_channel_tag_role_gardener')),
                                     (4, ref('website_slides.slide_channel_tag_role_carpenter')),
                                     (4, ref('website_slides.slide_channel_tag_other_0')),
                                     (4, ref('website_slides.slide_channel_tag_other_2'))]"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/channel_demo_gardening_3.jpg"/>
        <field name="description">A lot of nice documentation: trees, wood, gardens. A gold mine for references.</field>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=6)"/>
    </record>

    <record id="slide_channel_demo_3_furn0" model="slide.channel">
        <field name="name">Choose your wood!</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="enroll">invite</field>
        <field name="visibility">members</field>
        <field name="channel_type">training</field>
        <field name="allow_comment" eval="True"/>
        <field name="promote_strategy">latest</field>
        <field name="is_published" eval="True"/>
        <field name="tag_ids" eval="[(5, 0),
                                     (4, ref('website_slides.slide_channel_tag_level_basic')),
                                     (4, ref('website_slides.slide_channel_tag_role_gardener')),
                                     (4, ref('website_slides.slide_channel_tag_role_carpenter')),
                                     (4, ref('website_slides.slide_channel_tag_role_furniture')),
                                     (4, ref('website_slides.slide_channel_tag_other_2'))]"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/channel_demo_tree_1.jpg"/>
        <field name="description">Knowing which kind of wood to use depending on your application is important. In this course you
will learn the basics of wood characteristics.</field>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=5)"/>
    </record>

    <record id="slide_channel_demo_4_furn1" model="slide.channel">
        <field name="name">Furniture Technical Specifications</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="enroll">invite</field>
        <field name="channel_type">documentation</field>
        <field name="allow_comment" eval="True"/>
        <field name="promote_strategy">most_voted</field>
        <field name="is_published" eval="True"/>
        <field name="tag_ids" eval="[(5, 0),
                                     (4, ref('website_slides.slide_channel_tag_level_intermediate')),
                                     (4, ref('website_slides.slide_channel_tag_role_furniture'))]"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/channel_demo_furniture.jpg"/>
        <field name="description">If you are looking for technical specifications, have a look at this documentation.</field>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=4)"/>
    </record>

    <!-- This channel will gain a certification slide -->
    <record id="slide_channel_demo_5_furn2" model="slide.channel">
        <field name="name">Basics of Furniture Creation</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="enroll">invite</field>
        <field name="visibility">connected</field>
        <field name="channel_type">training</field>
        <field name="allow_comment" eval="True"/>
        <field name="promote_strategy">latest</field>
        <field name="is_published" eval="True"/>
        <field name="tag_ids" eval="[(5, 0),
                                     (4, ref('website_slides.slide_channel_tag_level_intermediate')),
                                     (4, ref('website_slides.slide_channel_tag_role_furniture')),
                                     (4, ref('website_slides.slide_channel_tag_other_0')),
                                     (4, ref('website_slides.slide_channel_tag_other_1'))]"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/channel_demo_furniture_2.jpg"/>
        <field name="description">All you need to know about furniture creation.</field>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=3)"/>
    </record>

    <!-- This channel will be set on payment -->
    <!-- This channel will gain a certification slide -->
    <record id="slide_channel_demo_6_furn3" model="slide.channel">
        <field name="name">DIY Furniture</field>
        <field name="user_id" ref="base.user_demo"/>
        <field name="enroll">invite</field>
        <field name="channel_type">training</field>
        <field name="allow_comment" eval="True"/>
        <field name="promote_strategy">most_voted</field>
        <field name="is_published" eval="True"/>
        <field name="tag_ids" eval="[(5, 0),
                                 (4, ref('website_slides.slide_channel_tag_level_advanced')),
                                 (4, ref('website_slides.slide_channel_tag_role_carpenter')),
                                 (4, ref('website_slides.slide_channel_tag_role_furniture')),
                                 (4, ref('website_slides.slide_channel_tag_other_1'))]"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/channel_demo_furniture_3.jpg"/>
        <field name="description">So much amazing certification.</field>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=2)"/>
    </record>

    <record id="slide_tag_demo_cheatsheet" model="slide.tag">
        <field name="name">CheatSheet</field>
    </record>
    <record id="slide_tag_demo_theory" model="slide.tag">
        <field name="name">Theory</field>
    </record>
    <record id="slide_tag_demo_exercises" model="slide.tag">
        <field name="name">Exercises</field>
    </record>
    <record id="slide_tag_demo_tools" model="slide.tag">
        <field name="name">Tools</field>
    </record>
    <record id="slide_tag_demo_colorful" model="slide.tag">
        <field name="name">Colorful</field>
    </record>
    <record id="slide_tag_demo_howto" model="slide.tag">
        <field name="name">HowTo</field>
    </record>

</data></odoo>

```

## File: data\slide_channel_tag_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <record id="slide_channel_tag_group_role" model="slide.channel.tag.group">
        <field name="name">Your Role</field>
        <field name="is_published" eval="True"/>
    </record>
    <record id="slide_channel_tag_role_gardener" model="slide.channel.tag">
        <field name="name">Gardener</field>
        <field name="color">10</field>
        <field name="group_id" ref="website_slides.slide_channel_tag_group_role"/>
    </record>
    <record id="slide_channel_tag_role_carpenter" model="slide.channel.tag">
        <field name="name">Carpenter</field>
        <field name="color">11</field>
        <field name="group_id" ref="website_slides.slide_channel_tag_group_role"/>
    </record>
    <record id="slide_channel_tag_role_furniture" model="slide.channel.tag">
        <field name="name">Furniture Designer</field>
        <field name="color">3</field>
        <field name="group_id" ref="website_slides.slide_channel_tag_group_role"/>
    </record>

    <record id="slide_channel_tag_other_0" model="slide.channel.tag">
        <field name="name">Quiz</field>
        <field name="group_id" ref="website_slides.slide_channel_tag_group_data_other"/>
    </record>
    <record id="slide_channel_tag_other_1" model="slide.channel.tag">
        <field name="name">Certification</field>
        <field name="color">7</field>
        <field name="group_id" ref="website_slides.slide_channel_tag_group_data_other"/>
    </record>
    <record id="slide_channel_tag_other_2" model="slide.channel.tag">
        <field name="name">Dog Friendly</field>
        <field name="group_id" ref="website_slides.slide_channel_tag_group_data_other"/>
    </record>
</data></odoo>

```

## File: data\slide_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="slide_channel_tag_group_data_other" model="slide.channel.tag.group">
            <field name="name">Tags</field>
            <field name="sequence">20</field>
            <field name="is_published" eval="True"/>
        </record>
        <record id="slide_channel_tag_group_level" model="slide.channel.tag.group">
            <field name="name">Your Level</field>
            <field name="is_published" eval="True"/>
        </record>
        <record id="slide_channel_tag_level_basic" model="slide.channel.tag">
            <field name="name">Basic</field>
            <field name="color">10</field>
            <field name="group_id" ref="website_slides.slide_channel_tag_group_level"/>
        </record>
        <record id="slide_channel_tag_level_intermediate" model="slide.channel.tag">
            <field name="name">Intermediate</field>
            <field name="color">3</field>
            <field name="group_id" ref="website_slides.slide_channel_tag_group_level"/>
        </record>
        <record id="slide_channel_tag_level_advanced" model="slide.channel.tag">
            <field name="name">Advanced</field>
            <field name="color">1</field>
            <field name="group_id" ref="website_slides.slide_channel_tag_group_level"/>
        </record>
    </data>
</odoo>

```

## File: data\slide_slide_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <!-- CHANNEL 0: Basics of Gardening -->
    <!-- ================================================== -->
    <record id="slide_slide_demo_0_0" model="slide.slide">
        <field name="name">Gardening: The Know-How</field>
        <field name="sequence">1</field>
        <field name="binary_content" type="base64" file="website_slides/static/src/img/presentation.pdf"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/channel-training-default.jpg"/>
        <field name="slide_category">document</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="True"/>
        <field name="public_views">10</field>
        <field name="completion_time">2.5</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_tools')), (4, ref('website_slides.slide_tag_demo_howto'))]"/>
        <field name="description">A summary of know-how: how and what. All the basics for this course about gardening.</field>
    </record>
        <!--RESOURCE-->
        <record id="slide_slide_demo_0_0_resource_0" model="slide.slide.resource">
            <field name="resource_type">file</field>
            <field name="name">Document</field>
            <field name="data" type="base64" file="website_slides/static/src/img/document.png"/>
            <field name="slide_id" ref="slide_slide_demo_0_0"/>
        </record>
    <record id="slide_slide_demo_0_1" model="slide.slide">
        <field name="name">Home Gardening</field>
        <field name="sequence">2</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_gardening_1.jpg"/>
        <field name="slide_category">infographic</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">5</field>
        <field name="completion_time">1</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_colorful')), (4, ref('website_slides.slide_tag_demo_theory'))]"/>
        <field name="description">Interesting information about home gardening. Keep it close!</field>
    </record>
    <record id="slide_slide_demo_0_2" model="slide.slide">
        <field name="name">Mighty Carrots</field>
        <field name="sequence">3</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_gardening_2.jpg"/>
        <field name="slide_category">infographic</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">2</field>
        <field name="completion_time">2</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_colorful')), (4, ref('website_slides.slide_tag_demo_theory'))]"/>
        <field name="description">You won't believe those facts about carrots.</field>
    </record>
    <record id="slide_slide_demo_0_3" model="slide.slide">
        <field name="name">How to Grow and Harvest The Best Strawberries | Basics</field>
        <field name="sequence">4</field>
        <field name="binary_content" type="base64" file="website_slides/static/src/img/presentation.pdf"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_thumb_l0JZ25VvbwE.jpg"/>
        <field name="slide_category">document</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">0</field>
        <field name="completion_time">1</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_howto'))]"/>
        <field name="description">Here is How to get the Sweetest Strawberries you ever tasted!</field>
    </record>
    <record id="slide_slide_demo_0_4" model="slide.slide">
        <field name="name">Test your knowledge</field>
        <field name="sequence">5</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_owl.jpg"/>
        <field name="slide_category">quiz</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="True"/>
        <field name="public_views">0</field>
        <field name="completion_time">1</field>
        <field name="description">Show your newly mastered knowledge!</field>
    </record>
        <record id="slide_slide_demo_0_4_question_0" model="slide.question">
            <field name="question">What is a strawberry?</field>
            <field name="sequence">1</field>
            <field name="slide_id" ref="slide_slide_demo_0_4"/>
        </record>
        <record id="slide_slide_demo_0_4_question_0_0" model="slide.answer">
            <field name="text_value">A fruit</field>
            <field name="sequence">1</field>
            <field name="is_correct" eval="True"/>
            <field name="comment">Correct! A strawberry is a fruit because it's the product of a tree.</field>
            <field name="question_id" ref="slide_slide_demo_0_4_question_0"/>
        </record>
        <record id="slide_slide_demo_0_4_question_0_1" model="slide.answer">
            <field name="text_value">A vegetable</field>
            <field name="sequence">2</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect! A strawberry is not a vegetable.</field>
            <field name="question_id" ref="slide_slide_demo_0_4_question_0"/>
        </record>
        <record id="slide_slide_demo_0_4_question_0_2" model="slide.answer">
            <field name="text_value">A table</field>
            <field name="sequence">3</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect! A table is a piece of furniture.</field>
            <field name="question_id" ref="slide_slide_demo_0_4_question_0"/>
        </record>
        <record id="slide_slide_demo_0_4_question_1" model="slide.question">
            <field name="question">What is the best tool to dig a hole for your plants?</field>
            <field name="sequence">2</field>
            <field name="slide_id" ref="slide_slide_demo_0_4"/>
        </record>
        <record id="slide_slide_demo_0_4_question_1_0" model="slide.answer">
            <field name="text_value">A shovel</field>
            <field name="sequence">1</field>
            <field name="is_correct" eval="True"/>
            <field name="comment">Correct! A shovel is the perfect tool to dig a hole.</field>
            <field name="question_id" ref="slide_slide_demo_0_4_question_1"/>
        </record>
        <record id="slide_slide_demo_0_4_question_1_1" model="slide.answer">
            <field name="text_value">A spoon</field>
            <field name="sequence">2</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect! Good luck digging a hole with a spoon...</field>
            <field name="question_id" ref="slide_slide_demo_0_4_question_1"/>
        </record>

    <!-- CHANNEL 1: Taking care of Trees -->
    <!-- ================================================== -->

    <!--                    Categories                      -->
    <record id="slide_category_demo_1_0" model="slide.slide">
        <field name="name">Interesting Facts</field>
        <field name="is_category" eval="True"/>
        <field name="channel_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="sequence">0</field>
    </record>
    <record id="slide_category_demo_1_1" model="slide.slide">
        <field name="name">Methods</field>
        <field name="is_category" eval="True"/>
        <field name="channel_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="sequence">4</field>
    </record>

    <!--                    Slides                      -->
    <record id="slide_slide_demo_1_0" model="slide.slide">
        <field name="name">List Infographic</field>
        <field name="sequence">1</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_tree_infographic_1.jpg"/>
        <field name="slide_category">infographic</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="True"/>
        <field name="public_views">5</field>
        <field name="completion_time">0.5</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_colorful'))]"/>
        <field name="description">Just some basics Tree Infographic.</field>
    </record>
    <record id="slide_slide_demo_1_1" model="slide.slide">
        <field name="name">Interesting List Facts</field>
        <field name="sequence">2</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_tree_infographic_2.jpg"/>
        <field name="slide_category">infographic</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">5</field>
        <field name="completion_time">1.5</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_colorful'))]"/>
        <field name="description">Just some basics Interesting Tree Facts.</field>
    </record>
    <record id="slide_slide_demo_1_2" model="slide.slide">
        <field name="name">Energy Efficiency Facts</field>
        <field name="sequence">3</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_tree_infographic_3.jpg"/>
        <field name="slide_category">infographic</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">10</field>
        <field name="completion_time">1</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_colorful')), (4, ref('website_slides.slide_tag_demo_theory'))]"/>
        <field name="description">Just some basics Energy Efficiency Facts.</field>
    </record>
        <record id="slide_slide_demo_1_2_link_0" model="slide.slide.resource">
            <field name="resource_type">url</field>
            <field name="name">Energy Efficient Link 1</field>
            <field name="link">http://www.example.com</field>
            <field name="slide_id" ref="slide_slide_demo_1_2"/>
        </record>
        <record id="slide_slide_demo_1_2_link_1" model="slide.slide.resource">
            <field name="resource_type">url</field>
            <field name="name">Energy Efficient Link 2</field>
            <field name="link">http://www.example.com</field>
            <field name="slide_id" ref="slide_slide_demo_1_2"/>
        </record>
        <!--RESOURCE-->
        <record id="slide_slide_demo_1_2_resource_0" model="slide.slide.resource">
            <field name="resource_type">file</field>
            <field name="name">Presentation</field>
            <field name="data" type="base64" file="website_slides/static/src/img/presentation.pdf"/>
            <field name="slide_id" ref="slide_slide_demo_1_2"/>
        </record>
    <record id="slide_slide_demo_1_3" model="slide.slide">
        <field name="name">How to plant a potted.list</field>
        <field name="sequence">5</field>
        <field name="url">https://www.youtube.com/watch?v=QYmgrw0PgLU</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_thumb_QYmgrw0PgLU.jpg"/>
        <field name="slide_category">video</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">0</field>
        <field name="completion_time">1</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_howto'))]"/>
        <field name="description">Jim and Todd plant a potted tree for a customer of Knecht's Nurseries and Landscaping. Narrated by Leif Knecht, owner.</field>
    </record>
    <record id="slide_slide_demo_1_4" model="slide.slide">
        <field name="name">A little chat with Harry Potted</field>
        <field name="sequence">6</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_tree_img_1.jpg"/>
        <field name="slide_category">article</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="html_content" type="html">
<section class="s_cover parallax bg-black-50 pt16 pb16" data-scroll-background-ratio="0" style="background-image: none;" data-snippet="s_cover">
    <span class="s_parallax_bg oe_img_bg" style="background-image: url('/website_slides/static/src/img/slide_demo_tree_img_1.jpg'); background-position: 50% 0;"></span>
    <div class="o_we_bg_filter bg-black-50"/>
    <div class="container">
        <div class="row s_nb_column_fixed">
            <div class="col-lg-12">
                <h1 class="o_default_snippet_text display-3" style="text-align: center;">Catchy Headline</h1>
                <p class="lead o_default_snippet_text" style="text-align: center;">Write one or two paragraphs describing your product, services or a specific feature.<br/> To be successful your content needs to be useful to your readers.</p>
                <p>
                    <a href="/contactus" class="btn btn-primary rounded-circle o_default_snippet_text">Contact us</a>
                </p>
            </div>
        </div>
    </div>
</section>
<section class="s_text_image pt80 pb80" data-snippet="s_text_image">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-6 pt16 pb16">
                <img src="/website_slides/static/src/img/slide_demo_tree_img_1.jpg" class="img img-fluid mx-auto rounded" alt="Odoo • Image and Text"/>
            </div>
            <div class="col-lg-6 pt16 pb16">
                <h2 class="o_default_snippet_text h3-fs">Section Subtitle</h2>
                <p class="o_default_snippet_text">Write one or two paragraphs describing your product or services. <br/>To be successful your content needs to be useful to your readers.</p>
                <p class="o_default_snippet_text">Start with the customer – find out what they want and give it to them.</p>
                <p class="o_default_snippet_text"><a href="#" class="btn btn-outline-primary">Discover more</a></p>
            </div>
        </div>
    </div>
</section></field>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="True"/>
        <field name="public_views">5</field>
        <field name="completion_time">1</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_colorful'))]"/>
        <field name="description">We had a little chat with Harry Potted, sure he had interesting things to say!</field>
    </record>
        <record id="slide_slide_demo_1_4_link_0" model="slide.slide.resource">
            <field name="resource_type">url</field>
            <field name="name">Know More Link 1</field>
            <field name="link">http://www.example.com</field>
            <field name="slide_id" ref="slide_slide_demo_1_4"/>
        </record>
        <record id="slide_slide_demo_1_4_link_1" model="slide.slide.resource">
            <field name="resource_type">url</field>
            <field name="name">Know More Link 2</field>
            <field name="link">http://www.example.com</field>
            <field name="slide_id" ref="slide_slide_demo_1_4"/>
        </record>
        <record id="slide_slide_demo_1_4_question_0" model="slide.question">
            <field name="question">Do you think Harry Potted has a good name?</field>
            <field name="sequence">1</field>
            <field name="slide_id" ref="slide_slide_demo_1_4"/>
        </record>
        <record id="slide_slide_demo_1_4_question_0_0" model="slide.answer">
            <field name="text_value">Yes</field>
            <field name="sequence">1</field>
            <field name="is_correct" eval="True"/>
            <field name="comment">Correct!</field>
            <field name="question_id" ref="slide_slide_demo_1_4_question_0"/>
        </record>
        <record id="slide_slide_demo_1_4_question_0_1" model="slide.answer">
            <field name="text_value">No</field>
            <field name="sequence">2</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect!</field>
            <field name="question_id" ref="slide_slide_demo_1_4_question_0"/>
        </record>
        <record id="slide_slide_demo_1_4_question_1" model="slide.question">
            <field name="question">Did you read the whole article?</field>
            <field name="sequence">2</field>
            <field name="slide_id" ref="slide_slide_demo_1_4"/>
        </record>
        <record id="slide_slide_demo_1_4_question_1_0" model="slide.answer">
            <field name="text_value">Yes</field>
            <field name="sequence">1</field>
            <field name="is_correct" eval="True"/>
            <field name="comment">Correct! Congratulations you have time to loose</field>
            <field name="question_id" ref="slide_slide_demo_1_4_question_1"/>
        </record>
        <record id="slide_slide_demo_1_4_question_1_1" model="slide.answer">
            <field name="text_value">No</field>
            <field name="sequence">2</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect! You really should read it.</field>
            <field name="question_id" ref="slide_slide_demo_1_4_question_1"/>
        </record>
        <record id="slide_slide_demo_1_4_question_1_2" model="slide.answer">
            <field name="text_value">What was the question again?</field>
            <field name="sequence">3</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect! Seriously?</field>
            <field name="question_id" ref="slide_slide_demo_1_4_question_1"/>
        </record>
    <record id="slide_slide_demo_1_5" model="slide.slide">
        <field name="name">3 Main Methodologies</field>
        <field name="sequence">6</field>
        <field name="binary_content" type="base64" file="website_slides/static/src/img/presentation.pdf"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/channel-training-default.jpg"/>
        <field name="slide_category">document</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">10</field>
        <field name="completion_time">2.5</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_howto'))]"/>
        <field name="description">A summary of know-how: how and what.</field>
    </record>
    <record id="slide_slide_demo_1_6" model="slide.slide">
        <field name="name">How to Grow and Harvest The Best Strawberries | Gardening Tips and Tricks</field>
        <field name="sequence">7</field>
        <field name="url">https://www.youtube.com/watch?v=l0JZ25VvbwE</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_thumb_l0JZ25VvbwE.jpg"/>
        <field name="slide_category">video</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">0</field>
        <field name="completion_time">1</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_howto'))]"/>
        <field name="description">Here is How to get the Sweetest Strawberries you ever tasted!</field>
    </record>

    <!-- CHANNEL 2: Trees, Wood and Garden -->
    <!-- ================================================== -->

    <!--                    Categories                      -->
    <record id="slide_category_demo_2_0" model="slide.slide">
        <field name="name">Trees</field>
        <field name="is_category" eval="True"/>
        <field name="channel_id" ref="website_slides.slide_channel_demo_2_gard2"/>
        <field name="sequence">0</field>
    </record>
    <record id="slide_category_demo_2_1" model="slide.slide">
        <field name="name">Wood</field>
        <field name="is_category" eval="True"/>
        <field name="channel_id" ref="website_slides.slide_channel_demo_2_gard2"/>
        <field name="sequence">4</field>
    </record>

    <!--                    Slides                      -->
    <!-- Category: Trees -->
    <record id="slide_slide_demo_2_0" model="slide.slide">
        <field name="name">Main Trees Categories</field>
        <field name="sequence">1</field>
        <field name="binary_content" type="base64" file="website_slides/static/src/img/presentation.pdf"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_tree_img_2.jpg"/>
        <field name="slide_category">document</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_2_gard2"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="True"/>
        <field name="public_views">0</field>
        <field name="completion_time">1</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_howto'))]"/>
        <field name="quiz_first_attempt_reward">100</field>
        <field name="quiz_second_attempt_reward">75</field>
        <field name="quiz_third_attempt_reward">50</field>
        <field name="quiz_fourth_attempt_reward">25</field>
        <field name="description">A summary of know-how: what are the main trees categories and how to differentiate them.</field>
    </record>
        <!-- LINKS -->
        <record id="slide_slide_demo_2_0_link_0" model="slide.slide.resource">
            <field name="resource_type">url</field>
            <field name="name">Trees Classification Link</field>
            <field name="link">http://www.example.com</field>
            <field name="slide_id" ref="slide_slide_demo_2_0"/>
        </record>
        <record id="slide_slide_demo_2_0_link_1" model="slide.slide.resource">
            <field name="resource_type">url</field>
            <field name="name">Main types of trees Link</field>
            <field name="link">http://www.example.com</field>
            <field name="slide_id" ref="slide_slide_demo_2_0"/>
        </record>
        <!--RESOURCE-->
        <record id="slide_slide_demo_2_0_resource_0" model="slide.slide.resource">
            <field name="resource_type">file</field>
            <field name="name">List image</field>
            <field name="data" type="base64" file="website_slides/static/src/img/slide_demo_tree_img_2.jpg"/>
            <field name="slide_id" ref="slide_slide_demo_2_0"/>
        </record>
        <!-- QUIZZ -->
        <record id="slide_slide_demo_2_0_question_0" model="slide.question">
            <field name="question">Do you make beams out of lemon trees?</field>
            <field name="sequence">1</field>
            <field name="slide_id" ref="slide_slide_demo_2_0"/>
        </record>
        <record id="slide_slide_demo_2_0_question_0_0" model="slide.answer">
            <field name="text_value">Yes</field>
            <field name="sequence">1</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect!</field>
            <field name="question_id" ref="slide_slide_demo_2_0_question_0"/>
        </record>
        <record id="slide_slide_demo_2_0_question_0_1" model="slide.answer">
            <field name="text_value">No</field>
            <field name="sequence">2</field>
            <field name="is_correct" eval="True"/>
            <field name="comment">Correct!</field>
            <field name="question_id" ref="slide_slide_demo_2_0_question_0"/>
        </record>
        <record id="slide_slide_demo_2_0_question_1" model="slide.question">
            <field name="question">Do you make lemons out of beams?</field>
            <field name="sequence">2</field>
            <field name="slide_id" ref="slide_slide_demo_2_0"/>
        </record>
        <record id="slide_slide_demo_2_0_question_1_0" model="slide.answer">
            <field name="text_value">Yes</field>
            <field name="sequence">1</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect!</field>
            <field name="question_id" ref="slide_slide_demo_2_0_question_1"/>
        </record>
        <record id="slide_slide_demo_2_0_question_1_1" model="slide.answer">
            <field name="text_value">No</field>
            <field name="sequence">2</field>
            <field name="is_correct" eval="True"/>
            <field name="comment">Correct!</field>
            <field name="question_id" ref="slide_slide_demo_2_0_question_1"/>
        </record>
        <record id="slide_slide_demo_2_0_question_1_2" model="slide.answer">
            <field name="text_value">And also bananas</field>
            <field name="sequence">3</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect! of course not ...</field>
            <field name="question_id" ref="slide_slide_demo_2_0_question_1"/>
        </record>
    <record id="slide_slide_demo_2_1" model="slide.slide">
        <field name="name">A Mighty Forest from Ages</field>
        <field name="sequence">2</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_tree_img_3.jpg"/>
        <field name="slide_category">article</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_2_gard2"/>
        <field name="html_content" type="html">
<section class="s_cover parallax bg-black-50 pt16 pb16" data-scroll-background-ratio="0" style="background-image: none;" data-snippet="s_cover">
    <span class="s_parallax_bg oe_img_bg" style="background-image: url('/website_slides/static/src/img/slide_demo_tree_img_3.jpg'); background-position: 50% 0;"></span>
    <div class="o_we_bg_filter bg-black-50"/>
    <div class="container">
        <div class="row s_nb_column_fixed">
            <div class="col-lg-12">
                <h1 class="o_default_snippet_text display-3" style="text-align: center;">Catchy Headline</h1>
                <p class="lead o_default_snippet_text" style="text-align: center;">Write one or two paragraphs describing your product, services or a specific feature.<br/> To be successful your content needs to be useful to your readers.</p>
                <p class="o_default_snippet_text">
                    <a href="/contactus" class="btn btn-primary rounded-circle">Contact us</a>
                </p>
            </div>
        </div>
    </div>
</section>
<section class="s_text_image pt80 pb80" data-snippet="s_text_image">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-6 pt16 pb16">
                <img src="/website_slides/static/src/img/slide_demo_tree_img_3.jpg" class="img img-fluid mx-auto rounded" alt="Odoo • Image and Text"/>
            </div>
            <div class="col-lg-6 pt16 pb16">
                <h2 class="o_default_snippet_text h3-fs">Section Subtitle</h2>
                <p class="o_default_snippet_text">Write one or two paragraphs describing your product or services. <br/>To be successful your content needs to be useful to your readers.</p>
                <p class="o_default_snippet_text">Start with the customer – find out what they want and give it to them.</p>
                <p class="o_default_snippet_text">
                    <a href="#" class="btn btn-outline-primary">Discover more</a>
                </p>
            </div>
        </div>
    </div>
</section></field>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">0</field>
        <field name="completion_time">2</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_theory'))]"/>
        <field name="description">Mighty forest just don't appear in a few weeks. Learn how time made our forests mighty and mysterious.</field>
    </record>
    <record id="slide_slide_demo_2_2" model="slide.slide">
        <field name="name">List planting in hanging bottles on wall</field>
        <field name="sequence">3</field>
        <field name="url">https://www.youtube.com/watch?v=ebBez6bcSEc</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_thumb_ebBez6bcSEc.jpg"/>
        <field name="slide_category">video</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_2_gard2"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">0</field>
        <field name="completion_time">1</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_howto'))]"/>
        <field name="description">How to wall decorating by tree planting in hanging plastic bottles.</field>
    </record>
    <!-- Category: Wood -->
    <record id="slide_slide_demo_2_3" model="slide.slide">
        <field name="name">Wood Characteristics</field>
        <field name="sequence">5</field>
        <field name="binary_content" type="base64" file="website_slides/static/src/img/presentation.pdf"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/channel-documentation-default.jpg"/>
        <field name="slide_category">document</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_2_gard2"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">0</field>
        <field name="completion_time">2</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_cheatsheet'))]"/>
        <field name="quiz_first_attempt_reward">100</field>
        <field name="quiz_second_attempt_reward">75</field>
        <field name="quiz_third_attempt_reward">50</field>
        <field name="quiz_fourth_attempt_reward">25</field>
        <field name="description">Knowing wood characteristics is a requirement in order to know which kind of wood to use in a given situation.</field>
    </record>


    <!-- CHANNEL 3: Choose your wood!           -->
    <!-- ======================================= -->

    <!--                 Categories              -->
    <record id="slide_category_demo_3_0" model="slide.slide">
        <field name="name">Working with Wood</field>
        <field name="is_category" eval="True"/>
        <field name="channel_id" ref="website_slides.slide_channel_demo_3_furn0"/>
        <field name="sequence">1</field>
    </record>

    <!--                    Slides                      -->
    <record id="slide_slide_demo_3_0" model="slide.slide">
        <field name="name">Comparing Hardness of Wood Species</field>
        <field name="sequence">2</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_wood_infographic_1.jpg"/>
        <field name="slide_category">infographic</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_3_furn0"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="True"/>
        <field name="public_views">10</field>
        <field name="completion_time">12</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_colorful'))]"/>
        <field name="description">Comparing Hardness of Wood Species</field>
    </record>
    <record id="slide_slide_demo_3_1" model="slide.slide">
        <field name="name">Wood Bending With Steam Box</field>
        <field name="sequence">3</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_thumb_PYr1rK8pS30.jpg"/>
        <field name="url">https://www.youtube.com/watch?v=PYr1rK8pS30</field>
        <field name="slide_category">video</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_3_furn0"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">10</field>
        <field name="completion_time">3</field>
        <field name="description">Watching the master(s) at work</field>
    </record>

    <!-- CHANNEL 4: Furniture Technical Specifications -->
    <!-- ======================================= -->

    <!--                 Categories              -->
    <record id="slide_category_demo_4_0" model="slide.slide">
        <field name="name">Documents</field>
        <field name="is_category" eval="True"/>
        <field name="channel_id" ref="website_slides.slide_channel_demo_4_furn1"/>
        <field name="sequence">0</field>
    </record>
    <record id="slide_category_demo_4_1" model="slide.slide">
        <field name="name">Technical Drawings</field>
        <field name="is_category" eval="True"/>
        <field name="channel_id" ref="website_slides.slide_channel_demo_4_furn1"/>
        <field name="sequence">10</field>
    </record>

    <!--                    Slides                      -->
    <record id="slide_slide_demo_4_0" model="slide.slide">
        <field name="name">Foreword</field>
        <field name="sequence">1</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_furniture_2.jpg"/>
        <field name="slide_category">article</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_4_furn1"/>
        <field name="html_content" type="html">
<section class="s_cover parallax bg-black-50 pt16 pb16" data-scroll-background-ratio="0" style="background-image: none;" data-snippet="s_cover">
    <span class="s_parallax_bg oe_img_bg" style="background-image: url('/website_slides/static/src/img/slide_demo_tree_img_3.jpg'); background-position: 50% 0;"></span>
    <div class="o_we_bg_filter bg-black-50"/>
    <div class="container">
        <div class="row s_nb_column_fixed">
            <div class="col-lg-12">
                <h1 class="o_default_snippet_text display-3" style="text-align: center;">Foreword</h1>
                <p class="lead o_default_snippet_text" style="text-align: center;">Write one or two paragraphs describing your product, services or a specific feature.<br/> To be successful your content needs to be useful to your readers.</p>
                <p>
                    <a href="/contactus" class="btn btn-primary rounded-circle o_default_snippet_text">Contact us</a>
                </p>
            </div>
        </div>
    </div>
</section>
<section class="s_text_image pt80 pb80" data-snippet="s_text_image">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-6 pt16 pb16">
                <img src="/website_slides/static/src/img/slide_demo_tree_img_1.jpg" class="img img-fluid mx-auto rounded" alt="Odoo • Image and Text"/>
            </div>
            <div class="col-lg-6 pt16 pb16">
                <h2 class="o_default_snippet_text h3-fs">Section Subtitle</h2>
                <p class="o_default_snippet_text">Write one or two paragraphs describing your product or services. <br/>To be successful your content needs to be useful to your readers.</p>
                <p class="o_default_snippet_text">Start with the customer – find out what they want and give it to them.</p>
                <p class="o_default_snippet_text"><a href="#" class="btn btn-outline-primary">Discover more</a></p>
            </div>
        </div>
    </div>
</section></field>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=6)"/>
        <field name="is_preview" eval="True"/>
        <field name="public_views">10</field>
        <field name="completion_time">1</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_howto'))]"/>
        <field name="description">Foreword for this documentation: how to use it, main attention points</field>
    </record>
    <record id="slide_slide_demo_4_1" model="slide.slide">
        <field name="name">Wood Types</field>
        <field name="sequence">2</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_thumb_bvSe6r5BpaY.jpg"/>
        <field name="url">https://www.youtube.com/watch?v=bvSe6r5BpaY</field>
        <field name="slide_category">video</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_4_furn1"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=3)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">0</field>
        <field name="completion_time">0.5</field>
        <field name="description">Which wood type is best for my solid wood furniture? That's the question we help you answer in this video!</field>
    </record>
    <!-- Catgory: technical drawings -->
    <record id="slide_slide_demo_4_10" model="slide.slide">
        <field name="name">Drawing 1</field>
        <field name="sequence">11</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_furniture_img.jpg"/>
        <field name="slide_category">infographic</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_4_furn1"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=4)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">2</field>
        <field name="completion_time">2</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_theory'))]"/>
        <field name="description">Technical drawing</field>
    </record>
    <record id="slide_slide_demo_4_11" model="slide.slide">
        <field name="name">Drawing 2</field>
        <field name="sequence">12</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_furniture_img_1.jpg"/>
        <field name="slide_category">infographic</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_4_furn1"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=4)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">2</field>
        <field name="completion_time">2</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_theory'))]"/>
        <field name="description">Technical drawing</field>
    </record>
    <record id="slide_slide_demo_4_12" model="slide.slide">
        <field name="name">Presentation</field>
        <field name="sequence">13</field>
        <field name="binary_content" type="base64" file="website_slides/static/src/img/presentation.pdf"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/channel-documentation-default.jpg"/>
        <field name="slide_category">document</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_4_furn1"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=4)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">0</field>
        <field name="completion_time">3</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_howto'))]"/>
        <field name="description">GLork</field>
    </record>

    <!-- CHANNEL 5: Basics of Furniture Creation -->
    <!-- ======================================= -->

    <!--                 Categories              -->
    <record id="slide_category_demo_5_0" model="slide.slide">
        <field name="name">Tools and Methods</field>
        <field name="is_category" eval="True"/>
        <field name="channel_id" ref="website_slides.slide_channel_demo_5_furn2"/>
        <field name="sequence">0</field>
    </record>

    <record id="slide_category_demo_5_1" model="slide.slide">
        <field name="name">Hand on!</field>
        <field name="is_category" eval="True"/>
        <field name="channel_id" ref="website_slides.slide_channel_demo_5_furn2"/>
        <field name="sequence">3</field>
    </record>

    <record id="slide_category_demo_5_2" model="slide.slide">
        <field name="name">Test Yourself</field>
        <field name="is_category" eval="True"/>
        <field name="channel_id" ref="website_slides.slide_channel_demo_5_furn2"/>
        <field name="sequence">5</field>
    </record>

    <!--                    Slides                     -->
    <record id="slide_slide_demo_5_0" model="slide.slide">
        <field name="name">Unforgettable Tools</field>
        <field name="sequence">1</field>
        <field name="binary_content" type="base64" file="website_slides/static/src/img/presentation.pdf"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/channel-training-default.jpg"/>
        <field name="slide_category">document</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_5_furn2"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="True"/>
        <field name="public_views">10</field>
        <field name="completion_time">2.5</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_tools'))]"/>
        <field name="description">Tools you will need to complete this course.</field>
    </record>
        <record id="slide_slide_demo_5_0_link_0" model="slide.slide.resource">
            <field name="resource_type">url</field>
            <field name="name">Example Link 1</field>
            <field name="link">http://www.example.com</field>
            <field name="slide_id" ref="slide_slide_demo_5_0"/>
        </record>
        <record id="slide_slide_demo_5_0_link_1" model="slide.slide.resource">
            <field name="resource_type">url</field>
            <field name="name">Example Link 2</field>
            <field name="link">http://www.example.com</field>
            <field name="slide_id" ref="slide_slide_demo_5_0"/>
        </record>
    <record id="slide_slide_demo_5_1" model="slide.slide">
        <field name="name">How to find quality wood</field>
        <field name="sequence">2</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_thumb_5WMqwTnZ-qs.jpg"/>
        <field name="url">https://www.youtube.com/watch?v=5WMqwTnZ-qs</field>
        <field name="slide_category">video</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_5_furn2"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">0</field>
        <field name="completion_time">3</field>
        <field name="description">Learn to identify quality wood in order to create solid furnitures.</field>
    </record>
     <record id="slide_slide_demo_5_2" model="slide.slide">
        <field name="name">How To Build a HIGH QUALITY Dining Table with LIMITED TOOLS</field>
        <field name="sequence">4</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_thumb_grrXe1QZNzQ.jpg"/>
        <field name="url">https://www.youtube.com/watch?v=grrXe1QZNzQ</field>
        <field name="slide_category">video</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_5_furn2"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="True"/>
        <field name="public_views">5</field>
        <field name="completion_time">3</field>
        <field name="description">From a piece of wood to a fully functional furniture, step by step.</field>
    </record>
    <record id="slide_slide_demo_5_3" model="slide.slide">
        <field name="name">Test your knowledge!</field>
        <field name="sequence">6</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_owl.jpg"/>
        <field name="slide_category">quiz</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_5_furn2"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">0</field>
        <field name="completion_time">0.5</field>
        <field name="description">Test your knowledge!</field>
    </record>
        <record id="slide_slide_demo_5_3_question_0" model="slide.question">
            <field name="question">Do you want to reply correctly?</field>
            <field name="sequence">1</field>
            <field name="slide_id" ref="slide_slide_demo_5_3"/>
        </record>
        <record id="slide_slide_demo_5_3_question_0_0" model="slide.answer">
            <field name="text_value">Yes</field>
            <field name="sequence">1</field>
            <field name="is_correct" eval="True"/>
            <field name="comment">Correct! You did it!</field>
            <field name="question_id" ref="slide_slide_demo_5_3_question_0"/>
        </record>
        <record id="slide_slide_demo_5_3_question_0_1" model="slide.answer">
            <field name="text_value">No</field>
            <field name="sequence">2</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect! You better think twice...</field>
            <field name="question_id" ref="slide_slide_demo_5_3_question_0"/>
        </record>

    <!-- CHANNEL 6: DIY Furniture -->
    <!-- ======================================= -->

</data></odoo>

```

## File: data\slide_user_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!-- CHANNEL 0: Basics of Gardening -->
    <!-- ================================================== -->
    <!-- Note that admin is already member due to auto subscribe -->
    <record id="slide_channel_0_partner_demo" model="slide.channel.partner">
        <field name="channel_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="partner_id" ref="base.partner_demo"/>
    </record>
    <record id="slide_channel_0_partner_demo_portal" model="slide.channel.partner">
        <field name="channel_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="partner_id" ref="base.partner_demo_portal"/>
    </record>
    <record id="slide_slide_0_0_partner_admin" model="slide.slide.partner">
        <field name="slide_id" ref="website_slides.slide_slide_demo_0_0"/>
        <field name="partner_id" ref="base.partner_admin"/>
        <field name="completed" eval="True"/>
        <field name="vote">1</field>
    </record>
    <record id="slide_slide_0_1_partner_admin" model="slide.slide.partner">
        <field name="slide_id" ref="website_slides.slide_slide_demo_0_1"/>
        <field name="partner_id" ref="base.partner_admin"/>
        <field name="completed" eval="True"/>
        <field name="vote">1</field>
    </record>
    <record id="slide_channel_0_partner_demo" model="slide.channel.partner">
        <field name="channel_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="partner_id" ref="base.partner_demo"/>
    </record>
    <record id="slide_slide_0_0_partner_demo" model="slide.slide.partner">
        <field name="slide_id" ref="website_slides.slide_slide_demo_0_0"/>
        <field name="partner_id" ref="base.partner_demo"/>
        <field name="vote">1</field>
    </record>
    <record id="slide_slide_0_1_partner_demo" model="slide.slide.partner">
        <field name="slide_id" ref="website_slides.slide_slide_demo_0_1"/>
        <field name="partner_id" ref="base.partner_demo"/>
        <field name="vote">1</field>
    </record>
    <record id="slide_channel_0_partner_demo_portal" model="slide.channel.partner">
        <field name="channel_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="partner_id" ref="base.partner_demo_portal"/>
    </record>
    <record id="slide_slide_0_0_partner_demo_portal" model="slide.slide.partner">
        <field name="slide_id" ref="website_slides.slide_slide_demo_0_0"/>
        <field name="partner_id" ref="base.partner_demo_portal"/>
        <field name="vote">1</field>
    </record>

    <record id="message_channel_0_admin" model="mail.message">
        <field name="model">slide.channel</field>
        <field name="res_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="message_type">comment</field>
        <field name="author_id" ref="base.partner_admin"/>
        <field name="body" type="html"><div>I fear beginners could be lost... Isn't it a bit harsh for a "basics" course?</div></field>
    </record>
    <record id="rating_channel_0_admin" model="rating.rating">
        <field name="res_model_id" ref="website_slides.model_slide_channel"/>
        <field name="res_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="partner_id" ref="base.partner_admin"/>
        <field name="consumed" eval="True"/>
        <field name="feedback">I fear beginners could be lost... Isn't it a bit harsh for a "basics" course?</field>
        <field name="rating">3</field>
        <field name="message_id" ref="website_slides.message_channel_0_admin"/>
    </record>
    <record id="message_channel_0_demo" model="mail.message">
        <field name="model">slide.channel</field>
        <field name="res_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="message_type">comment</field>
        <field name="author_id" ref="base.partner_demo"/>
        <field name="body" type="html"><div>Back to basics and interesting. Just WOW!</div></field>
    </record>
    <record id="rating_channel_0_demo" model="rating.rating">
        <field name="res_model_id" ref="website_slides.model_slide_channel"/>
        <field name="res_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="partner_id" ref="base.partner_demo"/>
        <field name="consumed" eval="True"/>
        <field name="rating">5</field>
        <field name="feedback">Back to basics and interesting. Just WOW!</field>
        <field name="message_id" ref="website_slides.message_channel_0_demo"/>
    </record>

    <function model="slide.channel" name="message_subscribe"
            eval="[ref('website_slides.slide_channel_demo_0_gard_0')], [ref('base.partner_admin'), ref('base.partner_demo')]"/>

    <!-- CHANNEL 1: Taking care of Trees -->
    <!-- ================================================== -->
    <!-- Note that admin is already member due to auto subscribe -->
    <record id="slide_channel_1_partner_demo" model="slide.channel.partner">
        <field name="channel_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="partner_id" ref="base.partner_demo"/>
    </record>
    <record id="slide_channel_1_partner_demo_portal" model="slide.channel.partner">
        <field name="channel_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="partner_id" ref="base.partner_demo_portal"/>
    </record>
    <record id="slide_slide_1_0_partner_admin" model="slide.slide.partner">
    	<field name="slide_id" ref="website_slides.slide_slide_demo_1_0"/>
    	<field name="partner_id" ref="base.partner_admin"/>
    	<field name="completed" eval="True"/>
    </record>
    <record id="slide_slide_1_1_partner_admin" model="slide.slide.partner">
    	<field name="slide_id" ref="website_slides.slide_slide_demo_1_1"/>
    	<field name="partner_id" ref="base.partner_admin"/>
    	<field name="completed" eval="True"/>
    </record>
    <record id="slide_slide_1_2_partner_admin" model="slide.slide.partner">
        <field name="slide_id" ref="website_slides.slide_slide_demo_1_2"/>
        <field name="partner_id" ref="base.partner_admin"/>
        <field name="vote">1</field>
        <field name="completed" eval="True"/>
    </record>
    <record id="slide_slide_1_3_partner_admin" model="slide.slide.partner">
        <field name="slide_id" ref="website_slides.slide_slide_demo_1_3"/>
        <field name="partner_id" ref="base.partner_admin"/>
        <field name="vote">1</field>
        <field name="completed" eval="True"/>
    </record>
    <record id="slide_channel_1_partner_demo" model="slide.channel.partner">
        <field name="channel_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="partner_id" ref="base.partner_demo"/>
    </record>
    <record id="slide_slide_1_0_partner_demo" model="slide.slide.partner">
        <field name="slide_id" ref="website_slides.slide_slide_demo_1_0"/>
        <field name="partner_id" ref="base.partner_demo"/>
        <field name="vote">1</field>
        <field name="completed" eval="True"/>
    </record>
    <record id="slide_slide_1_1_partner_demo" model="slide.slide.partner">
        <field name="slide_id" ref="website_slides.slide_slide_demo_1_1"/>
        <field name="partner_id" ref="base.partner_demo"/>
        <field name="vote">1</field>
    </record>
    <record id="slide_channel_1_partner_demo_portal" model="slide.channel.partner">
        <field name="channel_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="partner_id" ref="base.partner_demo_portal"/>
    </record>
    <record id="slide_slide_1_0_partner_demo" model="slide.slide.partner">
        <field name="slide_id" ref="website_slides.slide_slide_demo_1_0"/>
        <field name="partner_id" ref="base.partner_demo_portal"/>
        <field name="vote">1</field>
        <field name="completed" eval="True"/>
    </record>

    <record id="message_channel_1_admin" model="mail.message">
        <field name="model">slide.channel</field>
        <field name="res_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="message_type">comment</field>
        <field name="author_id" ref="base.partner_admin"/>
        <field name="body" type="html"><div>Very good course.</div></field>
    </record>
    <record id="rating_channel_1_admin" model="rating.rating">
        <field name="res_model_id" ref="website_slides.model_slide_channel"/>
        <field name="res_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="partner_id" ref="base.partner_admin"/>
        <field name="consumed" eval="True"/>
        <field name="feedback">Very good course.</field>
        <field name="rating">5</field>
        <field name="message_id" ref="website_slides.message_channel_1_admin"/>
    </record>
    <record id="message_channel_1_demo" model="mail.message">
        <field name="model">slide.channel</field>
        <field name="res_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="message_type">comment</field>
        <field name="author_id" ref="base.partner_demo"/>
        <field name="body" type="html"><div>Interesting!</div></field>
    </record>
    <record id="rating_channel_1_demo" model="rating.rating">
        <field name="res_model_id" ref="website_slides.model_slide_channel"/>
        <field name="res_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="partner_id" ref="base.partner_demo"/>
        <field name="consumed" eval="True"/>
        <field name="rating">4</field>
        <field name="feedback">Interesting!</field>
        <field name="message_id" ref="website_slides.message_channel_1_demo"/>
    </record>
    <record id="message_channel_1_portal" model="mail.message">
        <field name="model">slide.channel</field>
        <field name="res_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="message_type">comment</field>
        <field name="author_id" ref="base.partner_demo_portal"/>
        <field name="body" type="html"><div>Interesting. Could be great to include more examples.</div></field>
    </record>
    <record id="rating_channel_1_portal" model="rating.rating">
        <field name="res_model_id" ref="website_slides.model_slide_channel"/>
        <field name="res_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="partner_id" ref="base.partner_demo_portal"/>
        <field name="consumed" eval="True"/>
        <field name="rating">3</field>
        <field name="feedback">Interesting. Could be great to include more examples.</field>
        <field name="message_id" ref="website_slides.message_channel_1_portal"/>
        <field name="publisher_comment">Thanks for the feedback! We just updated first lessons to better include newcomers.</field>
        <field name="publisher_id" ref="base.partner_admin"/>
        <field name="publisher_datetime" eval="(DateTime.today() - relativedelta(days=1)).strftime('%Y-%m-%d %H:%M')"/>
    </record>

    <function model="slide.channel" name="message_subscribe"
            eval="[ref('website_slides.slide_channel_demo_1_gard1')], [ref('base.partner_admin'), ref('base.partner_demo')]"/>


    <!-- CHANNEL 2: Trees, Wood and Garden -->
    <!-- ================================================== -->
    <record id="slide_channel_2_partner_admin" model="slide.channel.partner">
        <field name="channel_id" ref="website_slides.slide_channel_demo_2_gard2"/>
        <field name="partner_id" ref="base.partner_admin"/>
    </record>
    <record id="slide_slide_2_0_partner_admin" model="slide.slide.partner">
        <field name="slide_id" ref="website_slides.slide_slide_demo_2_0"/>
        <field name="partner_id" ref="base.partner_admin"/>
        <field name="vote">1</field>
        <field name="quiz_attempts_count">1</field>
        <field name="completed" eval="True"/>
    </record>
    <record id="slide_slide_2_1_partner_admin" model="slide.slide.partner">
        <field name="slide_id" ref="website_slides.slide_slide_demo_2_1"/>
        <field name="partner_id" ref="base.partner_admin"/>
        <field name="vote">1</field>
    </record>
    <!-- note that partner demo is already member of slide_channel_demo_2_gard2 due to auto subscribe -->
    <record id="slide_slide_2_0_partner_demo" model="slide.slide.partner">
        <field name="slide_id" ref="website_slides.slide_slide_demo_2_0"/>
        <field name="partner_id" ref="base.partner_demo"/>
        <field name="vote">1</field>
        <field name="quiz_attempts_count">2</field>
    </record>

    <function model="slide.channel" name="message_subscribe"
            eval="[ref('website_slides.slide_channel_demo_2_gard2')], [ref('base.partner_admin'), ref('base.partner_demo')]"/>

</data></odoo>

```

## File: data\slide_user_gamification_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Completion of Quiz and Karma Generation -->
        <record id="slide_slide_1_4_partner_admin" model="slide.slide.partner">
            <field name="slide_id" ref="website_slides.slide_slide_demo_1_4"/>
            <field name="partner_id" ref="base.partner_admin"/>
            <field name="quiz_attempts_count">1</field>
            <field name="completed" eval="True"/>
        </record>
        <function model="res.users" name="_add_karma">
            <value eval="[ref('base.user_admin')]"/>
            <value eval="20"/>
            <value model="slide.slide" eval="obj().env.ref('website_slides.slide_slide_demo_1_4')"/>
            <value>Quiz completed</value>
        </function>
        <!-- Completion of Course (Karma Generation Triggered in Python) -->
        <record id="slide_slide_1_3_partner_admin" model="slide.slide.partner">
            <field name="slide_id" ref="website_slides.slide_slide_demo_1_3"/>
            <field name="partner_id" ref="base.partner_admin"/>
            <field name="completed" eval="True"/>
        </record>
        <record id="slide_slide_1_5_partner_admin" model="slide.slide.partner">
            <field name="slide_id" ref="website_slides.slide_slide_demo_1_5"/>
            <field name="partner_id" ref="base.partner_admin"/>
            <field name="completed" eval="True"/>
        </record>
        <!-- Demo join and leave a course -->
        <function model="res.users" name="_add_karma">
            <value eval="[ref('base.user_demo')]"/>
            <value eval="30"/>
            <value model="slide.channel" eval="obj().env.ref('website_slides.slide_channel_demo_1_gard1')"/>
            <value>Course Completed</value>
        </function>
        <function model="res.users" name="_add_karma">
            <value eval="[ref('base.user_demo')]"/>
            <value eval="-30"/>
            <value model="slide.channel" eval="obj().env.ref('website_slides.slide_channel_demo_1_gard1')"/>
            <value>Membership Removed</value>
        </function>
    </data>
</odoo>

```

## File: data\website_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="website.default_website" model="website">
            <field name="website_slide_google_app_key">AIzaSyDOWlmDW-7DbLmOR9ZsT5AOEXf4n6TFwQA</field>
        </record>

        <record id="website_menu_slides" model="website.menu">
            <field name="name">Courses</field>
            <field name="url">/slides</field>
            <field name="parent_id" ref="website.main_menu"/>
            <field name="sequence" type="int">50</field>
        </record>
    </data>
</odoo>

```

## File: models\gamification_challenge.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class Challenge(models.Model):
    _inherit = 'gamification.challenge'

    challenge_category = fields.Selection(selection_add=[
        ('slides', 'Website / Slides')
    ], ondelete={'slides': 'set default'})

```

## File: models\gamification_karma_tracking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models


class KarmaTracking(models.Model):
    _inherit = 'gamification.karma.tracking'

    def _get_origin_selection_values(self):
        return (
            super(KarmaTracking, self)._get_origin_selection_values()
            + [('slide.slide', _('Course Quiz')), ('slide.channel', self.env['ir.model']._get('slide.channel').display_name)]
        )

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = "res.config.settings"

    website_slide_google_app_key = fields.Char(related='website_id.website_slide_google_app_key', readonly=False)
    module_website_sale_slides = fields.Boolean(string="Sell on eCommerce")
    module_website_slides_forum = fields.Boolean(string="Forum")
    module_website_slides_survey = fields.Boolean(string="Certifications")
    module_mass_mailing_slides = fields.Boolean(string="Mailing")

```

## File: models\res_groups.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class UserGroup(models.Model):
    _inherit = 'res.groups'

    def write(self, vals):
        """ Automatically subscribe new users to linked slide channels """
        write_res = super(UserGroup, self).write(vals)
        if vals.get('users'):
            # TDE FIXME: maybe directly check users and subscribe them
            self.env['slide.channel'].sudo().search([('enroll_group_ids', 'in', self._ids)])._add_groups_members()
        return write_res

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.osv import expression


class ResPartner(models.Model):
    _inherit = 'res.partner'

    slide_channel_ids = fields.Many2many(
        'slide.channel', string='eLearning Courses',
        compute='_compute_slide_channel_values',
        search='_search_slide_channel_ids',
        groups="website_slides.group_website_slides_officer")
    slide_channel_completed_ids = fields.One2many(
        'slide.channel', string='Completed Courses',
        compute='_compute_slide_channel_values',
        search='_search_slide_channel_completed_ids',
        groups="website_slides.group_website_slides_officer")
    slide_channel_count = fields.Integer(
        'Course Count', compute='_compute_slide_channel_values',
        groups="website_slides.group_website_slides_officer")
    slide_channel_company_count = fields.Integer(
        'Company Course Count', compute='_compute_slide_channel_company_count',
        groups="website_slides.group_website_slides_officer")

    def _compute_slide_channel_values(self):
        data = {
            (partner.id, member_status): channel_ids
            for partner, member_status, channel_ids in self.env['slide.channel.partner'].sudo()._read_group(
                domain=[('partner_id', 'in', self.ids), ('member_status', '!=', 'invited')],
                groupby=['partner_id', 'member_status'],
                aggregates=['channel_id:array_agg']
            )
        }

        for partner in self:
            slide_channel_ids = data.get((partner.id, 'joined'), []) + data.get((partner.id, 'ongoing'), []) + data.get((partner.id, 'completed'), [])
            partner.slide_channel_ids = slide_channel_ids
            partner.slide_channel_completed_ids = self.env['slide.channel'].browse(data.get((partner.id, 'completed'), []))
            partner.slide_channel_count = len(slide_channel_ids)

    def _search_slide_channel_completed_ids(self, operator, value):
        cp_done = self.env['slide.channel.partner'].sudo().search([
            ('channel_id', operator, value),
            ('member_status', '=', 'completed')
        ])
        return [('id', 'in', cp_done.partner_id.ids)]

    def _search_slide_channel_ids(self, operator, value):
        cp_enrolled = self.env['slide.channel.partner'].search([
            ('channel_id', operator, value),
            ('member_status', '!=', 'invited')
        ])
        return [('id', 'in', cp_enrolled.partner_id.ids)]

    @api.depends('is_company', 'child_ids.slide_channel_count')
    def _compute_slide_channel_company_count(self):
        for partner in self:
            if partner.is_company:
                partner.slide_channel_company_count = self.env['slide.channel'].sudo().search_count(
                    [('partner_ids', 'in', partner.child_ids.ids)]
                )
            else:
                partner.slide_channel_company_count = 0

    def action_view_courses(self):
        """ View partners courses. In singleton mode, return courses followed
        by all its contacts (if company) or by themselves (if not a company).
        Otherwise simply set a domain on required partners. The courses to which
        the partner(s) is not enrolled (e.g. invited) are not shown. """
        action = self.env["ir.actions.actions"]._for_xml_id("website_slides.slide_channel_partner_action")
        action['display_name'] = _('Courses')
        action['domain'] = [('member_status', '!=', 'invited')]
        if len(self) == 1 and self.is_company:
            action['domain'] = expression.AND([action['domain'], [('partner_id', 'in', self.child_ids.ids)]])
        elif len(self) == 1:
            action['context'] = {'search_default_partner_id': self.id}
        else:
            action['domain'] = expression.AND([action['domain'], [('partner_id', 'in', self.ids)]])
        return action

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _


class Users(models.Model):
    _inherit = 'res.users'

    @api.model_create_multi
    def create(self, vals_list):
        """ Trigger automatic subscription based on user groups """
        users = super(Users, self).create(vals_list)
        for user in users:
            self.env['slide.channel'].sudo().search([
                ('enroll_group_ids', 'in', user.groups_id.ids)
            ])._action_add_members(user.partner_id)
        return users

    def write(self, vals):
        """ Trigger automatic subscription based on updated user groups """
        res = super(Users, self).write(vals)
        sanitized_vals = self._remove_reified_groups(vals)
        if sanitized_vals.get('groups_id'):
            added_group_ids = [command[1] for command in sanitized_vals['groups_id'] if command[0] == 4]
            added_group_ids += [id for command in sanitized_vals['groups_id'] if command[0] == 6 for id in command[2]]
            self.env['slide.channel'].sudo().search([('enroll_group_ids', 'in', added_group_ids)])._action_add_members(self.mapped('partner_id'))
        return res

    def get_gamification_redirection_data(self):
        res = super(Users, self).get_gamification_redirection_data()
        res.append({
            'url': '/slides',
            'label': _('See our eLearning')
        })
        return res

```

## File: models\slide_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast
import logging
import uuid

from collections import defaultdict
from dateutil.relativedelta import relativedelta
from markupsafe import Markup

from odoo import api, fields, models, tools, _
from odoo.exceptions import AccessError, UserError, ValidationError
from odoo.osv import expression
from odoo.tools import is_html_empty

_logger = logging.getLogger(__name__)


class ChannelUsersRelation(models.Model):
    _name = 'slide.channel.partner'
    _description = 'Channel / Partners (Members)'
    _table = 'slide_channel_partner'
    _rec_name = 'partner_id'

    active = fields.Boolean(string='Active', default=True)
    channel_id = fields.Many2one('slide.channel', string='Course', index=True, required=True, ondelete='cascade')
    member_status = fields.Selection([
        ('invited', 'Invite Sent'),
        ('joined', 'Joined'),
        ('ongoing', 'Ongoing'),
        ('completed', 'Finished')],
        string='Attendee Status', readonly=True, required=True, default='joined')
    completion = fields.Integer('% Completed Contents', default=0, aggregator="avg")
    completed_slides_count = fields.Integer('# Completed Contents', default=0)
    partner_id = fields.Many2one('res.partner', index=True, required=True, ondelete='cascade')
    partner_email = fields.Char(related='partner_id.email', readonly=True)
    # channel-related information (for UX purpose)
    channel_user_id = fields.Many2one('res.users', string='Responsible', related='channel_id.user_id')
    channel_type = fields.Selection(related='channel_id.channel_type')
    channel_visibility = fields.Selection(related='channel_id.visibility')
    channel_enroll = fields.Selection(related='channel_id.enroll')
    channel_website_id = fields.Many2one('website', string='Website', related='channel_id.website_id')
    next_slide_id = fields.Many2one('slide.slide', string='Next Lesson', compute='_compute_next_slide_id')

    # Invitation
    invitation_link = fields.Char('Invitation Link', compute="_compute_invitation_link")
    last_invitation_date = fields.Datetime('Last Invitation Date')

    _sql_constraints = [
        ('channel_partner_uniq',
         'unique(channel_id, partner_id)',
         'A partner membership to a channel must be unique!'
        ),
        ('check_completion',
         'check(completion >= 0 and completion <= 100)',
         'The completion of a channel is a percentage and should be between 0% and 100.'
        )
    ]

    @api.depends('channel_id', 'partner_id')
    def _compute_invitation_link(self):
        ''' This sets the url used as hyperlink in the channel invitation email in template mail_notification_channel_invite.
        The partner_id is given in the url, as well as a hash based on the partner and channel id. '''
        for record in self:
            invitation_hash = record._get_invitation_hash()
            record.invitation_link = f'{record.channel_id.get_base_url()}/slides/{record.channel_id.id}/invite?invite_partner_id={record.partner_id.id}&invite_hash={invitation_hash}'

    def _compute_next_slide_id(self):
        self.env['slide.channel.partner'].flush_model()
        self.env['slide.slide'].flush_model()
        self.env['slide.slide.partner'].flush_model()
        query = """
            SELECT DISTINCT ON (SCP.id)
                SCP.id AS id,
                SS.id AS slide_id
            FROM slide_channel_partner SCP
            JOIN slide_slide SS
                ON SS.channel_id = SCP.channel_id
                AND SS.is_published = TRUE
                AND SS.active = TRUE
                AND SS.is_category = FALSE
                AND NOT EXISTS (
                    SELECT 1
                      FROM slide_slide_partner
                     WHERE slide_id = SS.id
                       AND partner_id = SCP.partner_id
                       AND completed = TRUE
                )
            WHERE SCP.id IN %s
            ORDER BY SCP.id, SS.sequence, SS.id
        """
        self.env.cr.execute(query, [tuple(self.ids)])
        next_slide_per_membership = {
            line['id']: line['slide_id']
            for line in self.env.cr.dictfetchall()
        }

        for membership in self:
            membership.next_slide_id = next_slide_per_membership.get(membership.id, False)

    def _recompute_completion(self):
        """ This method computes the completion and member_status of attendees that are neither
            'invited' nor 'completed'. Indeed, once completed, membership should remain so.
            We do not do any update on the 'invited' records.
            One should first set member_status to 'joined' before recomputing those values
            when enrolling an invited or archived attendee.
            It takes into account the previous completion value to add or remove karma for
            completing the course to the attendee (see _post_completion_update_hook)
        """
        read_group_res = self.env['slide.slide.partner'].sudo()._read_group(
            ['&', '&', ('channel_id', 'in', self.mapped('channel_id').ids),
             ('partner_id', 'in', self.mapped('partner_id').ids),
             ('completed', '=', True),
             ('slide_id.is_published', '=', True),
             ('slide_id.active', '=', True)],
            ['channel_id', 'partner_id'],
            aggregates=['__count'])
        mapped_data = {
            (channel.id, partner.id): count
            for channel, partner, count in read_group_res
        }

        completed_records = self.env['slide.channel.partner']
        uncompleted_records = self.env['slide.channel.partner']
        for record in self:
            if record.member_status in ('completed', 'invited'):
                continue
            was_finished = record.completion == 100
            record.completed_slides_count = mapped_data.get((record.channel_id.id, record.partner_id.id), 0)
            record.completion = round(100.0 * record.completed_slides_count / (record.channel_id.total_slides or 1))

            if not record.channel_id.active:
                continue
            elif not was_finished and record.channel_id.total_slides and record.completed_slides_count >= record.channel_id.total_slides:
                completed_records += record
            elif was_finished and record.completed_slides_count < record.channel_id.total_slides:
                uncompleted_records += record

            if record.completion == 100:
                record.member_status = 'completed'
            elif record.completion == 0:
                record.member_status = 'joined'
            else:
                record.member_status = 'ongoing'

        if completed_records:
            completed_records._post_completion_update_hook(completed=True)
            completed_records._send_completed_mail()

        if uncompleted_records:
            uncompleted_records._post_completion_update_hook(completed=False)

    def unlink(self):
        """
        Override unlink method :
        Remove attendee from a channel, then also remove slide.slide.partner related to.
        """
        if self:
            # find all slide link to the channel and the partner
            removed_slide_partner_domain = expression.OR([
                [('partner_id', '=', channel_partner.partner_id.id),
                 ('slide_id', 'in', channel_partner.channel_id.slide_ids.ids)]
                for channel_partner in self
            ])
            self.env['slide.slide.partner'].search(removed_slide_partner_domain).unlink()
        return super(ChannelUsersRelation, self).unlink()

    def _get_invitation_hash(self):
        """ Returns the invitation hash of the attendee, used to access courses as invited / joined. """
        self.ensure_one()
        token = (self.partner_id.id, self.channel_id.id)
        return tools.hmac(self.env(su=True), 'website_slides-channel-invite', token)

    def _post_completion_update_hook(self, completed=True):
        """ Post hook of _recompute_completion. Adds or removes
        karma given for completing the course.

        :param completed:
            True if course is completed.
            False if we remove an existing course completion.
        """
        for channel, memberships in self.grouped("channel_id").items():

            karma = channel.karma_gen_channel_finish
            if karma <= 0:
                continue

            karma_per_users = {}
            for user in memberships.sudo().partner_id.user_ids:
                karma_per_users[user] = {
                    'gain': karma if completed else karma * -1,
                    'source': channel,
                    'reason': _('Course Finished') if completed else _('Course Set Uncompleted'),
                }

            self.env['res.users']._add_karma_batch(karma_per_users)

    def _send_completed_mail(self):
        """ Send an email to the attendee when they have successfully completed a course. """
        template_to_records = dict()
        for record in self:
            template = record.channel_id.completed_template_id
            if template:
                template_to_records.setdefault(template, self.env['slide.channel.partner'])
                template_to_records[template] += record

        record_email_values = {}
        for template, records in template_to_records.items():
            record_values = template._generate_template(
                records.ids,
                ['attachment_ids',
                 'body_html',
                 'email_cc',
                 'email_from',
                 'email_to',
                 'mail_server_id',
                 'model',
                 'partner_to',
                 'reply_to',
                 'report_template_ids',
                 'res_id',
                 'scheduled_date',
                 'subject',
                ]
            )
            for res_id, values in record_values.items():
                # attachments specific not supported currently, only attachment_ids
                values.pop('attachments', False)
                values['body'] = values.get('body_html')  # keep body copy in chatter
                record_email_values[res_id] = values

        mail_mail_values = []
        for record in self:
            email_values = record_email_values.get(record.id)

            if not email_values or not email_values.get('partner_ids'):
                continue

            email_values.update(
                author_id=record.channel_id.user_id.partner_id.id or self.env.company.partner_id.id,
                auto_delete=True,
                recipient_ids=[(4, pid) for pid in email_values['partner_ids']],
            )
            email_values['body_html'] = template._render_encapsulate(
                'mail.mail_notification_light', email_values['body_html'],
                add_context={
                    'message': self.env['mail.message'].sudo().new(dict(body=email_values['body_html'], record_name=record.channel_id.name)),
                    'model_description': _('Completed Course')  # tde fixme: translate into partner lang
                }
            )
            mail_mail_values.append(email_values)

        if mail_mail_values:
            self.env['mail.mail'].sudo().create(mail_mail_values)

    @api.autovacuum
    def _gc_slide_channel_partner(self):
        ''' The invitations of 'invited' attendees are only valid for 3 months. Remove outdated invitations
        with no completion. A missing last_invitation_date is also considered as expired.'''
        limit_dt = fields.Datetime.subtract(fields.Datetime.now(), months=3)
        expired_invitations = self.env['slide.channel.partner'].with_context(active_test=False).search([
            ('member_status', '=', 'invited'),
            ('completion', '=', 0),
            '|',
            ('last_invitation_date', '=', False),
            '&',
            ('last_invitation_date', '!=', False),
            ('last_invitation_date', '<', limit_dt),
        ])
        expired_invitations.unlink()


class Channel(models.Model):
    """ A channel is a container of slides. """
    _name = 'slide.channel'
    _description = 'Course'
    _inherit = [
        'rating.mixin',
        'mail.activity.mixin',
        'image.mixin',
        'website.cover_properties.mixin',
        'website.seo.metadata',
        'website.published.multi.mixin',
        'website.searchable.mixin',
    ]
    _order = 'sequence, id'
    _partner_unfollow_enabled = True

    def _default_cover_properties(self):
        """ Cover properties defaults are overridden to keep a consistent look for the slides
        channels headers across Odoo versions (pre-customization, with purple gradient fitting the
        homepage images, etc). Furthermore, as adding padding to the cover would not look great,
        its height is set to fit to content (snippet option to change this also disabled on the view)."""
        res = super()._default_cover_properties()
        res.update({
            "background_color_class": "o_cc3",
            'background_color_style': (
                'background-color: rgba(0, 0, 0, 0); '
                'background-image: linear-gradient(120deg, #875A7B, #78516F);'
            ),
            'opacity': '0',
            'resize_class': 'cover_auto'
        })
        return res

    def _default_access_token(self):
        return str(uuid.uuid4())

    def _get_default_enroll_msg(self):
        return _('Contact Responsible')

    # description
    name = fields.Char('Name', translate=True, required=True)
    active = fields.Boolean(default=True, tracking=100)
    description = fields.Html('Description', translate=True, sanitize_attributes=False, sanitize_form=False, help="The description that is displayed on top of the course page, just below the title")
    description_short = fields.Html('Short Description', translate=True, sanitize_attributes=False, sanitize_form=False, help="The description that is displayed on the course card")
    description_html = fields.Html('Detailed Description', translate=tools.html_translate, sanitize_attributes=False, sanitize_form=False)
    channel_type = fields.Selection([
        ('training', 'Training'), ('documentation', 'Documentation')],
        string="Course type", default="training", required=True)
    sequence = fields.Integer(default=10)
    user_id = fields.Many2one('res.users', string='Responsible', default=lambda self: self.env.uid)
    color = fields.Integer('Color Index', default=0, help='Used to decorate kanban view')
    tag_ids = fields.Many2many(
        'slide.channel.tag', 'slide_channel_tag_rel', 'channel_id', 'tag_id',
        string='Tags', help='Used to categorize and filter displayed channels/courses')
    # slides: promote, statistics
    slide_ids = fields.One2many('slide.slide', 'channel_id', string="Slides and categories", copy=True)
    slide_content_ids = fields.One2many('slide.slide', string='Content', compute="_compute_category_and_slide_ids")
    slide_category_ids = fields.One2many('slide.slide', string='Categories', compute="_compute_category_and_slide_ids")
    slide_last_update = fields.Date('Last Update', compute='_compute_slide_last_update', store=True)
    slide_partner_ids = fields.One2many(
        'slide.slide.partner', 'channel_id', string="Slide User Data",
        copy=False, groups='website_slides.group_website_slides_officer')
    promote_strategy = fields.Selection([
        ('latest', 'Latest Created'),
        ('most_voted', 'Most Voted'),
        ('most_viewed', 'Most Viewed'),
        ('specific', 'Select Manually'),
        ('none', 'None')],
        string="Featured Content", default='latest', required=False,
        help='Defines the content that will be promoted on the course home page',
        copy=False,
    )
    promoted_slide_id = fields.Many2one('slide.slide', string='Promoted Slide', copy=False)
    access_token = fields.Char("Security Token", copy=False, default=_default_access_token)
    nbr_document = fields.Integer('Documents', compute='_compute_slides_statistics', store=True)
    nbr_video = fields.Integer('Videos', compute='_compute_slides_statistics', store=True)
    nbr_infographic = fields.Integer('Infographics', compute='_compute_slides_statistics', store=True)
    nbr_article = fields.Integer("Articles", compute='_compute_slides_statistics', store=True)
    nbr_quiz = fields.Integer("Number of Quizs", compute='_compute_slides_statistics', store=True)
    total_slides = fields.Integer('Number of Contents', compute='_compute_slides_statistics', store=True)
    total_views = fields.Integer('Visits', compute='_compute_slides_statistics', store=True)
    total_votes = fields.Integer('Votes', compute='_compute_slides_statistics', store=True)
    total_time = fields.Float('Duration', compute='_compute_slides_statistics', digits=(10, 2), store=True)
    rating_avg_stars = fields.Float("Rating Average (Stars)", compute='_compute_rating_stats', digits=(16, 1), compute_sudo=True)
    # configuration
    allow_comment = fields.Boolean(
        "Allow rating on Course", default=True,
        help="Allow Attendees to like and comment your content and to submit reviews on your course.")
    publish_template_id = fields.Many2one(
        'mail.template', string='New Content Notification',
        help="Defines the email your Attendees will receive each time you upload new content.",
        default=lambda self: self.env['ir.model.data']._xmlid_to_res_id('website_slides.slide_template_published'),
        domain=[('model', '=', 'slide.slide')])
    share_channel_template_id = fields.Many2one(
        'mail.template', string='Channel Share Template',
        help='Email template used when sharing a channel',
        default=lambda self: self.env['ir.model.data']._xmlid_to_res_id('website_slides.mail_template_channel_shared'))
    share_slide_template_id = fields.Many2one(
        'mail.template', string='Share Template',
        help="Email template used when sharing a slide",
        default=lambda self: self.env['ir.model.data']._xmlid_to_res_id('website_slides.slide_template_shared'))
    completed_template_id = fields.Many2one(
        'mail.template', string='Completion Notification', help="Defines the email your Attendees will receive once they reach the end of your course.",
        default=lambda self: self.env['ir.model.data']._xmlid_to_res_id('website_slides.mail_template_channel_completed'),
        domain=[('model', '=', 'slide.channel.partner')])
    enroll = fields.Selection([
        ('public', 'Open'), ('invite', 'On Invitation')],
        compute='_compute_enroll', store=True, readonly=False,
        default='public', string='Enroll Policy', required=True,
        help='Defines how people can enroll to your Course.', copy=False)
    enroll_msg = fields.Html(
        'Enroll Message', help="Message explaining the enroll process",
        default=_get_default_enroll_msg, translate=tools.html_translate, sanitize_attributes=False)
    enroll_group_ids = fields.Many2many('res.groups', string='Auto Enroll Groups', help="Members of those groups are automatically added as members of the channel.")
    visibility = fields.Selection([
        ('public', 'Everyone'),
        ('connected', 'Signed In'),
        ('members', 'Course Attendees')
    ], default='public', string='Show Course To', required=True,
        help='Defines who can access your courses and their content.')
    upload_group_ids = fields.Many2many(
        'res.groups', 'rel_upload_groups', 'channel_id', 'group_id', string='Upload Groups',
        help="Group of users allowed to publish contents on a documentation course.")
    website_default_background_image_url = fields.Char('Background image URL', compute='_compute_website_default_background_image_url')
    # membership
    channel_partner_ids = fields.One2many(
        'slide.channel.partner', 'channel_id', string='Enrolled Attendees Information',
        groups='website_slides.group_website_slides_officer', domain=[('member_status', '!=', 'invited')])
    channel_partner_all_ids = fields.One2many(
        'slide.channel.partner', 'channel_id', string='All Attendees Information',
        groups='website_slides.group_website_slides_officer')
    members_count = fields.Integer('# Enrolled Attendees', compute='_compute_members_counts')
    members_all_count = fields.Integer('# Enrolled or Invited Attendees', compute='_compute_members_counts')
    members_engaged_count = fields.Integer(
        '# Active Attendees', help="Active attendees include both 'joined' and 'ongoing' attendees.",
        compute='_compute_members_counts')
    members_completed_count = fields.Integer('# Completed Attendees', compute='_compute_members_counts')
    members_invited_count = fields.Integer('# Invited Attendees', compute='_compute_members_counts')
    # partner_ids is implemented as compute/search instead of specifying the relation table
    # directly because we want to exclude active=False records on the joining table
    partner_ids = fields.Many2many(
        'res.partner', string='Attendees', help="Enrolled partners in the course",
        compute="_compute_partners", search="_search_partner_ids")
    # not stored access fields, depending on each user
    completed = fields.Boolean('Done', compute='_compute_user_statistics', compute_sudo=False)
    completion = fields.Integer('Completion', compute='_compute_user_statistics', compute_sudo=False)
    can_upload = fields.Boolean('Can Upload', compute='_compute_can_upload', compute_sudo=False)
    has_requested_access = fields.Boolean(string='Access Requested', compute='_compute_has_requested_access', compute_sudo=False)
    is_member = fields.Boolean(
        string='Is Enrolled Attendee', help='Is the attendee actively enrolled.',
        compute='_compute_membership_values', search="_search_is_member")
    is_member_invited = fields.Boolean(
        string='Is Invited Attendee', help='Is the invitation for this attendee pending.',
        compute='_compute_membership_values', search="_search_is_member_invited")
    partner_has_new_content = fields.Boolean(compute='_compute_partner_has_new_content', compute_sudo=False)
    # karma generation
    karma_gen_channel_rank = fields.Integer(string='Course ranked', default=5)
    karma_gen_channel_finish = fields.Integer(string='Course finished', default=10)
    # Karma based actions
    karma_review = fields.Integer('Add Review', default=10, help="Karma needed to add a review on the course")
    karma_slide_comment = fields.Integer('Add Comment', default=3, help="Karma needed to add a comment on a slide of this course")
    karma_slide_vote = fields.Integer('Vote', default=3, help="Karma needed to like/dislike a slide of this course.")
    can_review = fields.Boolean('Can Review', compute='_compute_action_rights', compute_sudo=False)
    can_comment = fields.Boolean('Can Comment', compute='_compute_action_rights', compute_sudo=False)
    can_vote = fields.Boolean('Can Vote', compute='_compute_action_rights', compute_sudo=False)
    # prerequisite settings
    prerequisite_channel_ids = fields.Many2many(
        'slide.channel', 'slide_channel_prerequisite_slide_channel_rel', 'channel_id', 'prerequisite_channel_id',
        string='Prerequisites', help='Prerequisite courses to complete before accessing this one.',
        domain="[('id', '!=', id), ('visibility', '=', visibility), ('website_published', '=', website_published)]")
    prerequisite_of_channel_ids = fields.Many2many(
        'slide.channel', 'slide_channel_prerequisite_slide_channel_rel', 'prerequisite_channel_id', 'channel_id',
        string='Prerequisite Of', help='Courses that have this course as prerequisite.')
    prerequisite_user_has_completed = fields.Boolean(
        'Has Completed Prerequisite', compute='_compute_prerequisite_user_has_completed')

    _sql_constraints = [
        (
            "check_enroll",
            "CHECK(visibility != 'members' OR enroll = 'invite')",
            "The Enroll Policy should be set to 'On Invitation' when visibility is set to 'Course Attendees'"
        ),
    ]

    @api.depends('visibility')
    def _compute_enroll(self):
        self.filtered(lambda channel: channel.visibility == 'members').enroll = 'invite'

    @api.depends('channel_partner_all_ids', 'channel_partner_all_ids.member_status', 'channel_partner_all_ids.active')
    def _compute_partners(self):
        data = {
            slide_channel: partner_ids
            for slide_channel, partner_ids in self.env['slide.channel.partner'].sudo()._read_group(
                [('channel_id', 'in', self.ids), ('member_status', '!=', 'invited')],
                ['channel_id'],
                aggregates=['partner_id:array_agg']
            )
        }
        for slide_channel in self:
            slide_channel.partner_ids = data.get(slide_channel, [])

    def _search_partner_ids(self, operator, value):
        if isinstance(value, int) and operator == 'in':
            value = [value]
        return [(
            'channel_partner_ids', '=', self.env['slide.channel.partner'].sudo()._search(
                [('partner_id', operator, value),
                 ('active', '=', True),
                 ('member_status', '!=', 'invited')],
            )
        )]

    @api.depends('slide_ids.is_published')
    def _compute_slide_last_update(self):
        for record in self:
            record.slide_last_update = fields.Date.today()

    @api.depends('channel_partner_all_ids.channel_id', 'channel_partner_all_ids.member_status')
    def _compute_members_counts(self):
        read_group_res = self.env['slide.channel.partner'].sudo()._read_group(
            domain=[('channel_id', 'in', self.ids)],
            groupby=['channel_id', 'member_status'],
            aggregates=['__count']
        )
        data = {(channel.id, member_status): count for channel, member_status, count in read_group_res}
        for channel in self:
            channel.members_invited_count = data.get((channel.id, 'invited'), 0)
            channel.members_engaged_count = data.get((channel.id, 'joined'), 0) + data.get((channel.id, 'ongoing'), 0)
            channel.members_completed_count = data.get((channel.id, 'completed'), 0)
            channel.members_all_count = channel.members_invited_count + channel.members_engaged_count + channel.members_completed_count
            channel.members_count = channel.members_engaged_count + channel.members_completed_count

    @api.depends('activity_ids.request_partner_id')
    @api.depends_context('uid')
    @api.model
    def _compute_has_requested_access(self):
        requested_cids = self.sudo().activity_search(
            ['website_slides.mail_activity_data_access_request'],
            additional_domain=[('request_partner_id', '=', self.env.user.partner_id.id)]
        ).mapped('res_id')
        for channel in self:
            channel.has_requested_access = channel.id in requested_cids

    @api.depends('channel_partner_all_ids.partner_id', 'channel_partner_all_ids.member_status', 'channel_partner_all_ids.active')
    @api.depends_context('uid')
    def _compute_membership_values(self):
        if self.env.user._is_public():
            self.is_member = False
            self.is_member_invited = False
            return
        data = {
            member_status: channel_ids
            for member_status, channel_ids in self.env['slide.channel.partner'].sudo()._read_group(
                [('partner_id', '=', self.env.user.partner_id.id), ('channel_id', 'in', self.ids), ('active', '=', True)],
                ['member_status'], ['channel_id:array_agg']
            )
        }
        active_channels_ids = data.get('joined', []) + data.get('ongoing', []) + data.get('completed', [])
        invitation_pending_channels_ids = data.get('invited', [])
        for channel in self:
            channel.is_member = channel.id in active_channels_ids
            channel.is_member_invited = channel.id in invitation_pending_channels_ids

    def _search_is_member(self, operator, value):
        if operator not in ['=', '!='] or not isinstance(value, bool):
            raise NotImplementedError(_('Operation not supported'))
        check_has_access = operator == '=' and value or operator == '!=' and not value
        return [('id', 'in' if check_has_access else 'not in', self._search_is_member_channel_ids())]

    def _search_is_member_invited(self, operator, value):
        if operator not in ['=', '!='] or not isinstance(value, bool):
            raise NotImplementedError(_('Operation not supported'))
        check_has_access = operator == '=' and value or operator == '!=' and not value
        return [('id', 'in' if check_has_access else 'not in', self._search_is_member_channel_ids(invited=True))]

    def _search_is_member_channel_ids(self, invited=False):
        return self.env['slide.channel.partner'].sudo()._read_group(
            [('partner_id', '=', self.env.user.partner_id.id), ('member_status', '=' if invited else '!=', 'invited'), ('active', '=', True)],
            aggregates=['channel_id:array_agg']
        )[0][0]

    @api.depends('slide_ids.is_category')
    def _compute_category_and_slide_ids(self):
        for channel in self:
            channel.slide_category_ids = channel.slide_ids.filtered(lambda slide: slide.is_category)
            channel.slide_content_ids = channel.slide_ids - channel.slide_category_ids

    @api.depends('slide_ids.slide_category', 'slide_ids.is_published', 'slide_ids.completion_time',
                 'slide_ids.likes', 'slide_ids.dislikes', 'slide_ids.total_views', 'slide_ids.is_category', 'slide_ids.active')
    def _compute_slides_statistics(self):
        default_vals = dict(total_views=0, total_votes=0, total_time=0, total_slides=0)
        keys = ['nbr_%s' % slide_category for slide_category in self.env['slide.slide']._fields['slide_category'].get_values(self.env)]
        default_vals.update(dict((key, 0) for key in keys))

        result = dict((cid, dict(default_vals)) for cid in self.ids)
        read_group_res = self.env['slide.slide']._read_group(
            [('active', '=', True), ('is_published', '=', True), ('channel_id', 'in', self.ids), ('is_category', '=', False)],
            ['channel_id', 'slide_category'],
            aggregates=['__count', 'likes:sum', 'dislikes:sum', 'total_views:sum', 'completion_time:sum'])
        for channel, slide_category, count, likes_sum, dislikes_sum, total_views_sum, completion_time_sum in read_group_res:
            channel_dict = result[channel.id]
            channel_dict['total_votes'] += likes_sum
            channel_dict['total_votes'] -= dislikes_sum
            channel_dict['total_views'] += total_views_sum
            channel_dict['total_time'] += completion_time_sum
            if slide_category:
                channel_dict[f'nbr_{slide_category}'] = count
                channel_dict['total_slides'] += count

        for record in self:
            record.update(result.get(record.id, default_vals))

    def _compute_rating_stats(self):
        super(Channel, self)._compute_rating_stats()
        for record in self:
            record.rating_avg_stars = record.rating_avg

    @api.depends('slide_partner_ids', 'slide_partner_ids.completed', 'total_slides')
    @api.depends_context('uid')
    def _compute_user_statistics(self):
        current_user_info = self.env['slide.channel.partner'].sudo().search(
            [('channel_id', 'in', self.ids), ('partner_id', '=', self.env.user.partner_id.id)]
        )
        mapped_data = dict((info.channel_id.id, (info.member_status == 'completed', info.completed_slides_count)) for info in current_user_info)
        for record in self:
            completed, completed_slides_count = mapped_data.get(record.id, (False, 0))
            record.completed = completed
            record.completion = 100.0 if completed else round(100.0 * completed_slides_count / (record.total_slides or 1))

    @api.depends('upload_group_ids', 'user_id')
    @api.depends_context('uid')
    def _compute_can_upload(self):
        for record in self:
            if record.user_id == self.env.user:
                record.can_upload = True
            elif record.upload_group_ids:
                record.can_upload = bool(record.upload_group_ids & self.env.user.groups_id)
            else:
                record.can_upload = self.env.user.has_group('website_slides.group_website_slides_manager')

    @api.depends('channel_type', 'user_id', 'can_upload')
    @api.depends_context('uid')
    def _compute_can_publish(self):
        """ For channels of type 'training', only the responsible (see user_id field) can publish slides.
        The 'sudo' user needs to be handled because they are the one used for uploads done on the front-end when the
        logged in user is not publisher but fulfills the upload_group_ids condition. Invited attendees can
        preview the course as public and sudo. Prevent them from uploading."""
        for record in self:
            if not record.can_upload:
                record.can_publish = False
            elif record.user_id == self.env.user:
                record.can_publish = True
            else:
                record.can_publish = self.env.user.has_group('website_slides.group_website_slides_manager')

    @api.model
    def _get_can_publish_error_message(self):
        return _("Publishing is restricted to the responsible of training courses or members of the publisher group for documentation courses")

    @api.depends('slide_partner_ids')
    @api.depends_context('uid')
    def _compute_partner_has_new_content(self):
        new_published_slides = self.env['slide.slide'].sudo().search([
            ('is_published', '=', True),
            ('date_published', '>', fields.Datetime.now() - relativedelta(days=7)),
            ('channel_id', 'in', self.ids),
            ('is_category', '=', False)
        ])
        slide_partner_completed = self.env['slide.slide.partner'].sudo().search([
            ('channel_id', 'in', self.ids),
            ('partner_id', '=', self.env.user.partner_id.id),
            ('slide_id', 'in', new_published_slides.ids),
            ('completed', '=', True)
        ]).mapped('slide_id')
        for channel in self:
            new_slides = new_published_slides.filtered(lambda slide: slide.channel_id == channel)
            channel.partner_has_new_content = any(slide not in slide_partner_completed for slide in new_slides)

    @api.depends('channel_type')
    def _compute_website_default_background_image_url(self):
        for channel in self:
            channel.website_default_background_image_url = f'website_slides/static/src/img/channel-{channel.channel_type}-default.jpg'

    @api.depends('name', 'website_id.domain')
    def _compute_website_url(self):
        super(Channel, self)._compute_website_url()
        for channel in self:
            if channel.id:  # avoid to perform a slug on a not yet saved record in case of an onchange.
                base_url = channel.get_base_url()
                channel.website_url = '%s/slides/%s' % (base_url, self.env['ir.http']._slug(channel))

    @api.depends('can_publish', 'is_member', 'karma_review', 'karma_slide_comment', 'karma_slide_vote')
    @api.depends_context('uid')
    def _compute_action_rights(self):
        user_karma = self.env.user.karma
        for channel in self:
            if channel.can_publish:
                channel.can_vote = channel.can_comment = channel.can_review = True
            elif not channel.is_member:
                channel.can_vote = channel.can_comment = channel.can_review = False
            else:
                channel.can_review = user_karma >= channel.karma_review
                channel.can_comment = user_karma >= channel.karma_slide_comment
                channel.can_vote = user_karma >= channel.karma_slide_vote

    ######################
    # Prerequisite Compute
    ######################

    @api.depends('prerequisite_channel_ids', 'channel_partner_ids.member_status')
    @api.depends_context('uid')
    def _compute_prerequisite_user_has_completed(self):
        completed_prerequisite_channels = self.env['slide.channel.partner'].sudo().search([
            ('partner_id', '=', self.env.user.partner_id.id),
            ('channel_id', 'in', self.prerequisite_channel_ids.ids),
            ('member_status', '=', 'completed'),
        ]).mapped('channel_id')
        for channel in self:
            channel.prerequisite_user_has_completed = all(
                channel in completed_prerequisite_channels for channel in channel.prerequisite_channel_ids)

    # ---------------------------------------------------------
    # ORM Overrides
    # ---------------------------------------------------------

    def _init_column(self, column_name):
        """ Initialize the value of the given column for existing rows.
            Overridden here because we need to generate different access tokens
            and by default _init_column calls the default method once and applies
            it for every record.
        """
        if column_name != 'access_token':
            super(Channel, self)._init_column(column_name)
        else:
            query = """
                UPDATE %(table_name)s
                SET access_token = md5(md5(random()::varchar || id::varchar) || clock_timestamp()::varchar)::uuid::varchar
                WHERE access_token IS NULL
            """ % {'table_name': self._table}
            self.env.cr.execute(query)

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            # Ensure creator is member of its channel it is easier for them to manage it (unless it is odoobot)
            if not vals.get('channel_partner_ids') and not self.env.is_superuser():
                vals['channel_partner_ids'] = [(0, 0, {
                    'partner_id': self.env.user.partner_id.id
                })]
            if not is_html_empty(vals.get('description')) and is_html_empty(vals.get('description_short')):
                vals['description_short'] = vals['description']

        channels = super(Channel, self.with_context(mail_create_nosubscribe=True)).create(vals_list)

        for channel in channels:
            if channel.user_id:
                channel._action_add_members(channel.user_id.partner_id)
            if channel.enroll_group_ids:
                channel._add_groups_members()

        return channels

    def copy_data(self, default=None):
        default = dict(default or {})
        vals_list = super().copy_data(default=default)
        for channel, vals in zip(self, vals_list):
            if 'name' not in default:
                vals['name'] = f"{channel.name} ({_('copy')})"
            if 'enroll' not in default and channel.visibility == "members":
                vals['enroll'] = 'invite'
        return vals_list

    def write(self, vals):
        # If description_short wasn't manually modified, there is an implicit link between this field and description.
        if not is_html_empty(vals.get('description')) and is_html_empty(vals.get('description_short')) and self.description == self.description_short:
            vals['description_short'] = vals.get('description')

        res = super(Channel, self).write(vals)

        if vals.get('user_id'):
            self._action_add_members(self.env['res.users'].sudo().browse(vals['user_id']).partner_id)
            self.activity_reschedule(['website_slides.mail_activity_data_access_request'], new_user_id=vals.get('user_id'))
        if 'enroll_group_ids' in vals:
            self._add_groups_members()

        return res

    def unlink(self):
        """" Necessary override to avoid cache issues in the ORM.
        This signals the ORM to remove slides first to avoid having the SQL cascade the deletion,
        which attempts to recompute slide statistics of removed slides and creates a cache failure.

        Indeed, slides statistics are computed using a read_group which will try to flush the records
        first and fail with a "Could not find all values of slide.slide.category_id to flush them".
        (Fix suggested by the ORM team).

        (See '_compute_slides_statistics' and '_compute_category_completion_time'). """

        self.slide_ids.unlink()
        return super().unlink()

    def toggle_active(self):
        """ Archiving/unarchiving a channel does it on its slides, too.
        1. When archiving
        We want to be archiving the channel FIRST.
        So that when slides are archived and the recompute is triggered,
        it does not try to mark the channel as "completed".
        That happens because it counts slide_done / slide_total, but slide_total
        will be 0 since all the slides for the course have been archived as well.

        2. When un-archiving
        We want to archive the channel LAST.
        So that when it recomputes stats for the channel and completion, it correctly
        counts the slides_total by counting slides that are already un-archived. """

        to_archive = self.filtered(lambda channel: channel.active)
        to_activate = self.filtered(lambda channel: not channel.active)
        if to_archive:
            super(Channel, to_archive).toggle_active()
            to_archive.is_published = False
            to_archive.mapped('slide_ids').action_archive()
        if to_activate:
            to_activate.with_context(active_test=False).mapped('slide_ids').action_unarchive()
            super(Channel, to_activate).toggle_active()

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, *, parent_id=False, subtype_id=False, **kwargs):
        """ Temporary workaround to avoid spam. If someone replies on a channel
        through the 'Presentation Published' email, it should be considered as a
        note as we don't want all channel followers to be notified of this answer.
        Also make sure that only one review can be posted per course."""
        self.ensure_one()
        if kwargs.get('message_type') == 'comment' and not self.can_review:
            raise AccessError(_('Not enough karma to review'))
        if parent_id:
            parent_message = self.env['mail.message'].sudo().browse(parent_id)
            if parent_message.subtype_id and parent_message.subtype_id == self.env.ref('website_slides.mt_channel_slide_published'):
                subtype_id = self.env.ref('mail.mt_note').id
        message = super().message_post(parent_id=parent_id, subtype_id=subtype_id, **kwargs)
        if self.env.user._is_internal() and not message.rating_value:
            return message
        if message.subtype_id == self.env.ref("mail.mt_comment"):
            domain = [
                ("res_id", "=", self.id),
                ("author_id", "=", message.author_id.id),
                ("model", "=", "slide.channel"),
                ("subtype_id", "=", self.env.ref("mail.mt_comment").id),
            ]
            if self.env["mail.message"].search_count(domain, limit=2) > 1:
                raise ValidationError(_("Only a single review can be posted per course."))
        if message.rating_value and message.is_current_user_or_guest_author:
            self.env.user._add_karma(self.karma_gen_channel_rank, self, _("Course Ranked"))
        return message

    # ---------------------------------------------------------
    # Business / Actions
    # ---------------------------------------------------------

    def action_redirect_to_members(self, status_filter=''):
        """ Redirects to attendees of the course. If status_filter is set to 'invited' /
        'engaged' ('joined' + 'ongoing') / 'completed', attendees are filtered accordingly."""
        action_ctx = {}
        action = self.env["ir.actions.actions"]._for_xml_id("website_slides.slide_channel_partner_action")
        if status_filter == 'engaged':
            action_ctx['search_default_filter_joined'] = 1
            action_ctx['search_default_filter_ongoing'] = 1
        elif status_filter:
            action_ctx[f'search_default_filter_{status_filter}'] = 1
        action['domain'] = [('channel_id', 'in', self.ids)]
        action['sample'] = 1
        if status_filter == 'completed':
            help_message = {
                'header_message': _("No Attendee has completed this course yet!"),
                'body_message': ""
            }
        else:
            help_message = {
                'header_message': _("No Attendees Yet!"),
                'body_message': _("From here you'll be able to monitor attendees and to track their progress.")
            }
        action['help'] = Markup("""<p class="o_view_nocontent_smiling_face">%(header_message)s</p><p>%(body_message)s</p>""") % help_message
        if len(self) == 1:
            action['display_name'] = _('Attendees of %s', self.name)
            action_ctx['default_channel_id'] = self.id
        action['context'] = action_ctx
        return action

    def action_redirect_to_engaged_members(self):
        return self.action_redirect_to_members('engaged')

    def action_redirect_to_completed_members(self):
        return self.action_redirect_to_members('completed')

    def action_redirect_to_invited_members(self):
        return self.action_redirect_to_members('invited')

    def action_channel_enroll(self):
        template = self.env.ref('website_slides.mail_template_slide_channel_enroll', raise_if_not_found=False)
        return self._action_channel_open_invite_wizard(template, enroll_mode=True)

    def action_channel_invite(self):
        template = self.env.ref('website_slides.mail_template_slide_channel_invite', raise_if_not_found=False)
        return self._action_channel_open_invite_wizard(template)

    def _action_channel_open_invite_wizard(self, mail_template, enroll_mode=False):
        """ Open the invitation wizard to invite and add attendees to the course(s) in self.

        :param mail_template: mail.template used in the invite wizard.
        :param enroll_mode: true if we want to enroll the attendees invited through the wizard.
            False otherwise, adding them as 'invited', e.g. when using "Invite" action."""
        course_name = self.name if len(self) == 1 else ''
        local_context = dict(
            self.env.context,
            default_channel_id=self.id if len(self) == 1 else False,
            default_email_layout_xmlid='website_slides.mail_notification_channel_invite',
            default_enroll_mode=enroll_mode,
            default_template_id=mail_template and mail_template.id or False,
            default_use_template=bool(mail_template),
        )
        if enroll_mode:
            name = _('Enroll Attendees to %(course_name)s', course_name=course_name or _('a course'))
        else:
            name = _('Invite Attendees to %(course_name)s', course_name=course_name or _('a course'))

        return {
            'type': 'ir.actions.act_window',
            'views': [[False, 'form']],
            'res_model': 'slide.channel.invite',
            'target': 'new',
            'context': local_context,
            'name': name,
        }

    def _action_add_members(self, target_partners, member_status='joined', raise_on_access=False):
        """ Adds the target_partners as attendees of the channel(s).
            Partners are added as follows, depending on the value of member_status:
            1) (Default) 'joined'. The partners will be added as enrolled attendees. This will make the content
                (slides) of the channel available to that partner. This can also happen when an invited attendee
                enrolls themself. The attendees are also subscribed to the chatter of the channel.
                :return: the union of previous partners re-enrolling, new attendees and invited ones enrolling.
            2) 'invited' : This is used when inviting partners. The partners are added as invited attendees
                This will make the channel accessible but not the slides until they enroll themselves.
                :return: returns the union of new records and the ones unarchived.
        """
        SlideChannelPartnerSudo = self.env['slide.channel.partner'].sudo()
        allowed_channels = self._filter_add_members(target_partners, raise_on_access=raise_on_access)
        if not allowed_channels or not target_partners:
            return SlideChannelPartnerSudo

        existing_channel_partners = self.env['slide.channel.partner'].with_context(active_test=False).sudo().search([
            ('channel_id', 'in', allowed_channels.ids),
            ('partner_id', 'in', target_partners.ids)
        ])

        # Unarchive existing channel partners, recomputing their completion and updating member_status
        archived_channel_partners = existing_channel_partners.filtered(lambda channel_partner: not channel_partner.active)
        to_unarchived = SlideChannelPartnerSudo
        if archived_channel_partners:
            archived_channel_partners.action_unarchive()
            to_unarchived = archived_channel_partners
            # Update member_status (and completion if enrolling)
            to_unarchived.member_status = member_status
            if member_status == 'joined':
                to_unarchived._recompute_completion()

        existing_channel_partners_map = defaultdict(lambda: self.env['slide.channel.partner'])
        for channel_partner in existing_channel_partners:
            existing_channel_partners_map[channel_partner.channel_id] += channel_partner

        # Invited partners confirming their invitation by enrolling, or upgraded to 'joined'.
        to_update_as_joined = SlideChannelPartnerSudo
        to_create_channel_partners_values = []

        for channel in allowed_channels:
            channel_partners = existing_channel_partners_map[channel]
            if member_status == 'joined':
                to_update_as_joined += channel_partners.filtered(lambda cp: cp.member_status == 'invited')
            for partner in target_partners - channel_partners.partner_id:
                to_create_channel_partners_values.append(dict(channel_id=channel.id, partner_id=partner.id, member_status=member_status))

        new_slide_channel_partners = SlideChannelPartnerSudo.create(to_create_channel_partners_values)
        to_update_as_joined.member_status = 'joined'
        to_update_as_joined._recompute_completion()

        # All fragments are in sudo.
        result_channel_partners = to_unarchived + to_update_as_joined + new_slide_channel_partners

        # Subscribe partners joining the course to the chatter.
        if member_status == 'joined':
            result_channel_partners_map = defaultdict(list)
            for channel_partner in result_channel_partners:
                result_channel_partners_map[channel_partner.channel_id].append(channel_partner.partner_id.id)
            for channel, partner_ids in result_channel_partners_map.items():
                channel.message_subscribe(
                    partner_ids=partner_ids,
                    subtype_ids=[self.env.ref('website_slides.mt_channel_slide_published').id]
                )
        return result_channel_partners

    def _filter_add_members(self, target_partners, raise_on_access=False):
        allowed = self.filtered(lambda channel: channel.enroll == 'public')
        on_invite = self.filtered(lambda channel: channel.enroll == 'invite')
        if on_invite:
            if on_invite.has_access('write'):
                allowed |= on_invite
            elif raise_on_access:
                raise AccessError(_('You are not allowed to add members to this course. Please contact the course responsible or an administrator.'))
        return allowed

    def _add_groups_members(self):
        for channel in self:
            channel._action_add_members(channel.mapped('enroll_group_ids.users.partner_id'))

    def _get_earned_karma(self, partner_ids):
        """ Compute the number of karma earned by partners on a channel
        Warning: this count will not be accurate if the configuration has been
        modified after the completion of a course!
        """
        total_karma = defaultdict(list)

        slide_completed = self.env['slide.slide.partner'].sudo().search([
            ('partner_id', 'in', partner_ids),
            ('channel_id', 'in', self.ids),
            ('completed', '=', True),
            ('quiz_attempts_count', '>', 0)
        ])
        for partner_slide in slide_completed:
            slide = partner_slide.slide_id
            if not slide.question_ids:
                continue
            gains = [
                slide.quiz_first_attempt_reward,
                slide.quiz_second_attempt_reward,
                slide.quiz_third_attempt_reward,
                slide.quiz_fourth_attempt_reward,
            ]
            attempts = min(partner_slide.quiz_attempts_count, len(gains))
            total_karma[partner_slide.partner_id.id].append({
                'karma': gains[attempts - 1],
                'channel_id': slide.channel_id,
            })

        channel_completed = self.env['slide.channel.partner'].sudo().search([
            ('partner_id', 'in', partner_ids),
            ('channel_id', 'in', self.ids),
            ('member_status', '=', 'completed')
        ])
        for partner_channel in channel_completed:
            channel = partner_channel.channel_id
            total_karma[partner_channel.partner_id.id].append({
                'karma': channel.karma_gen_channel_finish,
                'channel_id': channel,
            })

        return total_karma

    def _remove_membership(self, partner_ids):
        """ Karma earned during course progress is kept upon membership removal.
        This is done because re-joining the course will not allow you to gain the karma again,
        as we keep your progress """
        if not partner_ids:
            raise ValueError("Do not use this method with an empty partner_id recordset")

        removed_channel_partner_domain = expression.OR([
            [('partner_id', 'in', partner_ids),
             ('channel_id', '=', channel.id)]
            for channel in self
        ])

        self.message_unsubscribe(partner_ids=partner_ids)
        if self:
            removed_channel_partner = self.env['slide.channel.partner'].sudo().search(removed_channel_partner_domain)
            if removed_channel_partner:
                removed_channel_partner.action_archive()

    def _send_share_email(self, emails):
        """ Share channel through emails."""
        courses_without_templates = self.filtered(lambda channel: not channel.share_channel_template_id)
        if courses_without_templates:
            raise UserError(_('Impossible to send emails. Select a "Channel Share Template" for courses %(course_names)s first',
                                 course_names=', '.join(courses_without_templates.mapped('name'))))
        mail_ids = []
        for record in self:
            template = record.share_channel_template_id.with_context(
                user=self.env.user,
                email=emails,
                base_url=record.get_base_url(),
            )
            email_values = {'email_to': emails}
            if self.env.user._is_portal():
                template = template.sudo()
                email_values['email_from'] = self.env.company.catchall_formatted or self.env.company.email_formatted

            mail_ids.append(template.send_mail(record.id, email_layout_xmlid='mail.mail_notification_light', email_values=email_values))
        return mail_ids

    def action_view_slides(self):
        action = self.env["ir.actions.actions"]._for_xml_id("website_slides.slide_slide_action")
        action['context'] = {
            'search_default_published': 1,
            'default_channel_id': self.id
        }
        action['domain'] = [('channel_id', "=", self.id), ('is_category', '=', False)]
        return action

    def action_view_ratings(self):
        action = self.env["ir.actions.actions"]._for_xml_id("website_slides.rating_rating_action_slide_channel")
        action['name'] = _('Rating of %s', self.name)
        action['domain'] = expression.AND([ast.literal_eval(action.get('domain', '[]')), [('res_id', 'in', self.ids)]])
        return action

    def action_request_access(self):
        """ Request access to the channel. Returns a dict with keys being either 'error'
        (specific error raised) or 'done' (request done or not). """
        if self.env.user._is_public():
            return {'error': _('You have to sign in before')}
        if not self.is_published:
            return {'error': _('Course not published yet')}
        if self.is_member:
            return {'error': _('Already member')}
        if self.enroll == 'invite':
            activities = self.sudo()._action_request_access(self.env.user.partner_id)
            if activities:
                return {'done': True}
            return {'error': _('Already Requested')}
        return {'done': False}

    def action_grant_access(self, partner_id):
        partner = self.env['res.partner'].browse(partner_id).exists()
        if partner:
            if self._action_add_members(partner):
                self.activity_search(
                    ['website_slides.mail_activity_data_access_request'],
                    user_id=self.user_id.id, additional_domain=[('request_partner_id', '=', partner.id)]
                ).action_feedback(feedback=_('Access Granted'))

    def action_refuse_access(self, partner_id):
        partner = self.env['res.partner'].browse(partner_id).exists()
        if partner:
            self.activity_search(
                ['website_slides.mail_activity_data_access_request'],
                user_id=self.user_id.id, additional_domain=[('request_partner_id', '=', partner.id)]
            ).action_feedback(feedback=_('Access Refused'))

    # ---------------------------------------------------------
    # Mailing Mixin API
    # ---------------------------------------------------------

    def _rating_domain(self):
        """ Only take the published rating into account to compute avg and count """
        domain = super(Channel, self)._rating_domain()
        return expression.AND([domain, [('is_internal', '=', False)]])

    def _action_request_access(self, partner):
        activities = self.env['mail.activity']
        requested_cids = self.sudo().activity_search(
            ['website_slides.mail_activity_data_access_request'],
            additional_domain=[('request_partner_id', '=', partner.id)]
        ).mapped('res_id')
        for channel in self:
            if channel.id not in requested_cids and channel.user_id:
                activities += channel.activity_schedule(
                    'website_slides.mail_activity_data_access_request',
                    note=_('<b>%s</b> is requesting access to this course.', partner.name),
                    user_id=channel.user_id.id,
                    request_partner_id=partner.id
                )
        return activities

    # ---------------------------------------------------------
    # Data / Misc
    # ---------------------------------------------------------

    def _get_categorized_slides(self, base_domain, order, force_void=True, limit=False, offset=False):
        """ Return an ordered structure of slides by categories within a given
        base_domain that must fulfill slides. As a course structure is based on
        its slides sequences, uncategorized slides must have the lowest sequences.

        Example
          * category 1 (sequence 1), category 2 (sequence 3)
          * slide 1 (sequence 0), slide 2 (sequence 2)
          * course structure is: slide 1, category 1, slide 2, category 2
            * slide 1 is uncategorized,
            * category 1 has one slide : Slide 2
            * category 2 is empty.

        Backend and frontend ordering is the same, uncategorized first. It
        eases resequencing based on DOM / displayed order, notably when
        drag n drop is involved. """
        self.ensure_one()
        all_categories = self.env['slide.slide'].sudo().search([('channel_id', '=', self.id), ('is_category', '=', True)])
        all_slides = self.env['slide.slide'].sudo().search(base_domain, order=order)
        category_data = []

        # Prepare all categories by natural order
        for category in all_categories:
            category_slides = all_slides.filtered(lambda slide: slide.category_id == category)
            if not category_slides and not force_void:
                continue
            category_data.append({
                'category': category, 'id': category.id,
                'name': category.name, 'slug_name': self.env['ir.http']._slug(category),
                'total_slides': len(category_slides),
                'slides': category_slides[(offset or 0):(limit + offset or len(category_slides))],
            })

        # Add uncategorized slides in first position
        uncategorized_slides = all_slides.filtered(lambda slide: not slide.category_id)
        if uncategorized_slides or force_void:
            category_data.insert(0, {
                'category': False, 'id': False,
                'name': _('Uncategorized'), 'slug_name': _('Uncategorized'),
                'total_slides': len(uncategorized_slides),
                'slides': uncategorized_slides[(offset or 0):(offset + limit or len(uncategorized_slides))],
            })

        return category_data

    def _move_category_slides(self, category, new_category):
        if not category.slide_ids:
            return
        truncated_slide_ids = [slide_id for slide_id in self.slide_ids.ids if slide_id not in category.slide_ids.ids]
        if new_category:
            place_idx = truncated_slide_ids.index(new_category.id)
            ordered_slide_ids = truncated_slide_ids[:place_idx] + category.slide_ids.ids + truncated_slide_ids[place_idx]
        else:
            ordered_slide_ids = category.slide_ids.ids + truncated_slide_ids
        for index, slide_id in enumerate(ordered_slide_ids):
            self.env['slide.slide'].browse([slide_id]).sequence = index + 1

    def _resequence_slides(self, slide, force_category=False):
        ids_to_resequence = self.slide_ids.ids
        index_of_added_slide = ids_to_resequence.index(slide.id)
        next_category_id = None
        if self.slide_category_ids:
            force_category_id = force_category.id if force_category else slide.category_id.id
            index_of_category = self.slide_category_ids.ids.index(force_category_id) if force_category_id else None
            if index_of_category is None:
                next_category_id = self.slide_category_ids.ids[0]
            elif index_of_category < len(self.slide_category_ids.ids) - 1:
                next_category_id = self.slide_category_ids.ids[index_of_category + 1]

        if next_category_id:
            added_slide_id = ids_to_resequence.pop(index_of_added_slide)
            index_of_next_category = ids_to_resequence.index(next_category_id)
            ids_to_resequence.insert(index_of_next_category, added_slide_id)
            for i, record in enumerate(self.env['slide.slide'].browse(ids_to_resequence)):
                record.write({'sequence': i + 1})  # start at 1 to make people scream
        else:
            slide.write({
                'sequence': self.env['slide.slide'].browse(ids_to_resequence[-1]).sequence + 1
            })

    def get_backend_menu_id(self):
        return self.env.ref('website_slides.website_slides_menu_root').id

    @api.model
    def _search_get_detail(self, website, order, options):
        with_description = options['displayDescription']
        with_date = options['displayDetail']
        my = options.get('my')
        search_tags = options.get('tag')
        slide_category = options.get('slide_category')
        domain = [website.website_domain()]
        if my:
            domain.append([('is_member', '=', True)])
        if search_tags:
            ChannelTag = self.env['slide.channel.tag']
            try:
                tag_ids = list(filter(None, [self.env['ir.http']._unslug(tag)[1] for tag in search_tags.split(',')]))
                tags = ChannelTag.search([('id', 'in', tag_ids)]) if tag_ids else ChannelTag
            except Exception:
                tags = ChannelTag
            # Group by group_id
            # OR inside a group, AND between groups.
            for tags in tags.grouped('group_id').values():
                domain.append([('tag_ids', 'in', tags.ids)])
        if slide_category and 'nbr_%s' % slide_category in self:
            domain.append([('nbr_%s' % slide_category, '>', 0)])
        search_fields = ['name']
        fetch_fields = ['name', 'website_url']
        mapping = {
            'name': {'name': 'name', 'type': 'text', 'match': True},
            'website_url': {'name': 'website_url', 'type': 'text', 'truncate': False},
        }
        if with_description:
            search_fields.append('description_short')
            fetch_fields.append('description_short')
            mapping['description'] = {'name': 'description_short', 'type': 'text', 'html': True, 'match': True}
        if with_date:
            fetch_fields.append('slide_last_update')
            mapping['detail'] = {'name': 'slide_last_update', 'type': 'date'}
        return {
            'model': 'slide.channel',
            'base_domain': domain,
            'search_fields': search_fields,
            'fetch_fields': fetch_fields,
            'mapping': mapping,
            'icon': 'fa-graduation-cap',
        }

    def _get_placeholder_filename(self, field):
        image_fields = ['image_%s' % size for size in [1920, 1024, 512, 256, 128]]
        if field in image_fields:
            return self.website_default_background_image_url
        return super()._get_placeholder_filename(field)

    def open_website_url(self):
        """ Overridden to use a relative URL instead of an absolute when website_id is False. """
        if self.website_id:
            return super().open_website_url()
        return self.env['website'].get_client_action(f'/slides/{self.env["ir.http"]._slug(self)}')

    def _mail_get_partner_fields(self, introspect_fields=False):
        return []

```

## File: models\slide_channel_tag.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint

from odoo import fields, models


class SlideChannelTagGroup(models.Model):
    _name = 'slide.channel.tag.group'
    _description = 'Channel/Course Groups'
    _inherit = 'website.published.mixin'
    _order = 'sequence asc'

    name = fields.Char('Group Name', required=True, translate=True)
    sequence = fields.Integer('Sequence', default=10, index=True, required=True)
    tag_ids = fields.One2many('slide.channel.tag', 'group_id', string='Tags')

    def _default_is_published(self):
        return True


class SlideChannelTag(models.Model):
    _name = 'slide.channel.tag'
    _description = 'Channel/Course Tag'
    _order = 'group_sequence asc, sequence asc'

    name = fields.Char('Name', required=True, translate=True)
    sequence = fields.Integer('Sequence', default=10, index=True, required=True)
    group_id = fields.Many2one('slide.channel.tag.group', string='Group', index=True, required=True, ondelete="cascade")
    group_sequence = fields.Integer(
        'Group sequence', related='group_id.sequence',
        index=True, readonly=True, store=True)
    channel_ids = fields.Many2many('slide.channel', 'slide_channel_tag_rel', 'tag_id', 'channel_id', string='Channels')
    color = fields.Integer(
        string='Color Index', default=lambda self: randint(1, 11),
        help="Tag color used in both backend and website. No color means no display in kanban or front-end, to distinguish internal tags from public categorization tags")

```

## File: models\slide_embed.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models


class EmbeddedSlide(models.Model):
    """ Embedding in third party websites. Track view count, generate statistics. """
    _name = 'slide.embed'
    _description = 'Embedded Slides View Counter'
    _rec_name = 'website_name'

    slide_id = fields.Many2one(
        'slide.slide', string="Presentation",
        required=True, index=True, ondelete='cascade')
    url = fields.Char('Third Party Website URL')
    website_name = fields.Char('Website', compute='_compute_website_name')
    count_views = fields.Integer('# Views', default=1)

    @api.depends('url')
    def _compute_website_name(self):
        for slide_embed in self:
            slide_embed.website_name = slide_embed.url or _('Unknown Website')

```

## File: models\slide_question.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class SlideQuestion(models.Model):
    _name = "slide.question"
    _rec_name = "question"
    _description = "Content Quiz Question"
    _order = "sequence"

    sequence = fields.Integer("Sequence")
    question = fields.Char("Question Name", required=True, translate=True)
    slide_id = fields.Many2one('slide.slide', string="Content", required=True, ondelete='cascade')
    answer_ids = fields.One2many('slide.answer', 'question_id', string="Answer", copy=True)
    answers_validation_error = fields.Char("Error on Answers", compute='_compute_answers_validation_error')
    # statistics
    attempts_count = fields.Integer(compute='_compute_statistics', groups='website_slides.group_website_slides_officer')
    attempts_avg = fields.Float(compute="_compute_statistics", digits=(6, 2), groups='website_slides.group_website_slides_officer')
    done_count = fields.Integer(compute="_compute_statistics", groups='website_slides.group_website_slides_officer')

    @api.constrains('answer_ids')
    def _check_answers_integrity(self):
        questions_to_fix = [
            f'- {question.slide_id.name}: {question.question}'
            for question in self
            if question.answers_validation_error
        ]
        if questions_to_fix:
            raise ValidationError(_(
                'All questions must have at least one correct answer and one incorrect answer: \n%s\n',
                '\n'.join(questions_to_fix)))

    @api.depends('slide_id')
    def _compute_statistics(self):
        slide_partners = self.env['slide.slide.partner'].sudo().search([('slide_id', 'in', self.slide_id.ids)])
        slide_stats = dict((s.slide_id.id, dict({'attempts_count': 0, 'attempts_unique': 0, 'done_count': 0})) for s in slide_partners)

        for slide_partner in slide_partners:
            slide_stats[slide_partner.slide_id.id]['attempts_count'] += slide_partner.quiz_attempts_count
            slide_stats[slide_partner.slide_id.id]['attempts_unique'] += 1
            if slide_partner.completed:
                slide_stats[slide_partner.slide_id.id]['done_count'] += 1

        for question in self:
            stats = slide_stats.get(question.slide_id.id)
            question.attempts_count = stats.get('attempts_count', 0) if stats else 0
            question.attempts_avg = stats.get('attempts_count', 0) / stats.get('attempts_unique', 1) if stats else 0
            question.done_count = stats.get('done_count', 0) if stats else 0

    @api.depends('answer_ids', 'answer_ids.is_correct')
    def _compute_answers_validation_error(self):
        for question in self:
            correct = question.answer_ids.filtered('is_correct')
            question.answers_validation_error = _(
                'This question must have at least one correct answer and one incorrect answer.'
            ) if not correct or correct == question.answer_ids else ''

class SlideAnswer(models.Model):
    _name = "slide.answer"
    _rec_name = "text_value"
    _description = "Slide Question's Answer"
    _order = 'question_id, sequence, id'

    sequence = fields.Integer("Sequence")
    question_id = fields.Many2one('slide.question', string="Question", required=True, ondelete='cascade')
    text_value = fields.Char("Answer", required=True, translate=True)
    is_correct = fields.Boolean("Is correct answer")
    comment = fields.Text("Comment", translate=True, help='This comment will be displayed to the user if they select this answer')

```

## File: models\slide_slide.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import datetime
import io
import logging
import re
import requests

from dateutil.relativedelta import relativedelta
from markupsafe import Markup
from werkzeug import urls

from odoo import api, fields, models, _
from odoo.exceptions import RedirectWarning, UserError, AccessError
from odoo.http import request
from odoo.tools import html2plaintext, sql
from odoo.tools.pdf import PdfFileReader

_logger = logging.getLogger(__name__)


class SlidePartnerRelation(models.Model):
    _name = 'slide.slide.partner'
    _description = 'Slide / Partner decorated m2m'
    _table = 'slide_slide_partner'
    _rec_name = 'partner_id'

    slide_id = fields.Many2one('slide.slide', string="Content", ondelete="cascade", index=True, required=True)
    slide_category = fields.Selection(related='slide_id.slide_category')
    channel_id = fields.Many2one(
        'slide.channel', string="Channel",
        related="slide_id.channel_id", store=True, index=True, ondelete='cascade')
    partner_id = fields.Many2one('res.partner', index=True, required=True, ondelete='cascade')
    vote = fields.Integer('Vote', default=0)
    completed = fields.Boolean('Completed')
    quiz_attempts_count = fields.Integer('Quiz attempts count', default=0)

    _sql_constraints = [
        ('slide_partner_uniq',
         'unique(slide_id, partner_id)',
         'A partner membership to a slide must be unique!'
        ),
        ('check_vote',
         'CHECK(vote IN (-1, 0, 1))',
         'The vote must be 1, 0 or -1.'
        ),
    ]

    @api.model_create_multi
    def create(self, vals_list):
        res = super().create(vals_list)
        completed = res.filtered('completed')
        if completed:
            completed._recompute_completion()
        return res

    def write(self, values):
        slides_completion_to_recompute = self.env['slide.slide.partner']
        if 'completed' in values:
            slides_completion_to_recompute = self.filtered(
                lambda slide_partner: slide_partner.completed != values['completed'])

        res = super(SlidePartnerRelation, self).write(values)

        if slides_completion_to_recompute:
            slides_completion_to_recompute._recompute_completion()

        return res

    def _recompute_completion(self):
        self.env['slide.channel.partner'].search([
            ('channel_id', 'in', self.channel_id.ids),
            ('partner_id', 'in', self.partner_id.ids),
            ('member_status', 'not in', ('completed', 'invited'))
        ])._recompute_completion()


class SlideTag(models.Model):
    """ Tag to search slides across channels. """
    _name = 'slide.tag'
    _description = 'Slide Tag'

    name = fields.Char('Name', required=True, translate=True)

    _sql_constraints = [
        ('slide_tag_unique', 'UNIQUE(name)', 'A tag must be unique!'),
    ]


class Slide(models.Model):
    _name = 'slide.slide'
    _inherit = [
        'mail.thread',
        'image.mixin',
        'website.seo.metadata',
        'website.published.mixin',
        'website.searchable.mixin',
    ]
    _description = 'Slides'
    _mail_post_access = 'read'
    _order_by_strategy = {
        'sequence': 'sequence asc, id asc',
        'most_viewed': 'total_views desc',
        'most_voted': 'likes desc',
        'latest': 'date_published desc',
    }
    _order = 'sequence asc, is_category asc, id asc'
    _partner_unfollow_enabled = True

    YOUTUBE_VIDEO_ID_REGEX = r'^(?:(?:https?:)?//)?(?:www\.|m\.)?(?:youtu\.be/|youtube(-nocookie)?\.com/(?:embed/|v/|shorts/|live/|watch\?v=|watch\?.+&v=))((?:\w|-){11})\S*$'
    GOOGLE_DRIVE_DOCUMENT_ID_REGEX = r'(^https:\/\/docs.google.com|^https:\/\/drive.google.com).*\/d\/([^\/]*)'
    VIMEO_VIDEO_ID_REGEX = r'\/\/(player.)?vimeo.com\/(?:[a-z]*\/)*([0-9]{6,11})\/?([0-9a-z]{6,11})?[?]?.*'

    # description
    name = fields.Char('Title', required=True, translate=True)
    image_1920 = fields.Image(compute="_compute_image_1920", store=True, readonly=False)  # image.mixin override
    active = fields.Boolean(default=True, tracking=100)
    sequence = fields.Integer('Sequence', default=0)
    user_id = fields.Many2one('res.users', string='Uploaded by', default=lambda self: self.env.uid)
    description = fields.Html('Description', translate=True, sanitize_attributes=False, sanitize_overridable=True)
    channel_id = fields.Many2one('slide.channel', string="Course", required=True, ondelete='cascade')
    tag_ids = fields.Many2many('slide.tag', 'rel_slide_tag', 'slide_id', 'tag_id', string='Tags')
    is_preview = fields.Boolean('Allow Preview', default=False, help="The course is accessible by anyone : the users don't need to join the channel to access the content of the course.")
    is_new_slide = fields.Boolean('Is New Slide', compute='_compute_is_new_slide')
    completion_time = fields.Float('Duration', digits=(10, 4), compute='_compute_category_completion_time', recursive=True, readonly=False, store=True)
    # Categories
    is_category = fields.Boolean('Is a category', default=False)
    category_id = fields.Many2one('slide.slide', string="Section", compute="_compute_category_id", store=True)
    slide_ids = fields.One2many('slide.slide', "category_id", string="Content")
    # subscribers
    partner_ids = fields.Many2many('res.partner', 'slide_slide_partner', 'slide_id', 'partner_id',
                                   string='Subscribers', groups='website_slides.group_website_slides_officer', copy=False)
    slide_partner_ids = fields.One2many('slide.slide.partner', 'slide_id', string='Subscribers information', groups='website_slides.group_website_slides_officer', copy=False)
    user_membership_id = fields.Many2one(
        'slide.slide.partner', string="Subscriber information",
        compute='_compute_user_membership_id', compute_sudo=False,
        help="Subscriber information for the current logged in user")
    # current user membership
    user_vote = fields.Integer('User vote', compute='_compute_user_membership_id', compute_sudo=False)
    user_has_completed = fields.Boolean('Is Member', compute='_compute_user_membership_id', compute_sudo=False)
    user_has_completed_category = fields.Boolean('Is Category Completed', compute='_compute_category_completed')
    # Quiz related fields
    question_ids = fields.One2many("slide.question", "slide_id", string="Questions", copy=True)
    questions_count = fields.Integer(string="Numbers of Questions", compute='_compute_questions_count')
    quiz_first_attempt_reward = fields.Integer("Reward: first attempt", default=10)
    quiz_second_attempt_reward = fields.Integer("Reward: second attempt", default=7)
    quiz_third_attempt_reward = fields.Integer("Reward: third attempt", default=5,)
    quiz_fourth_attempt_reward = fields.Integer("Reward: every attempt after the third try", default=2)
    # content
    can_self_mark_completed = fields.Boolean('Can Mark Completed', compute='_compute_mark_complete_actions',
        help='The slide can be marked as completed even without opening it')
    can_self_mark_uncompleted = fields.Boolean('Can Mark Uncompleted', compute='_compute_mark_complete_actions',
        help='The slide can be marked as not completed and the progression')
    slide_category = fields.Selection([
        ('infographic', 'Image'),
        ('article', 'Article'),
        ('document', 'Document'),
        ('video', 'Video'),
        ('quiz', "Quiz")],
        string='Category', required=True,
        default='document')
    source_type = fields.Selection([
        ('local_file', 'Upload from Device'),
        ('external', 'Retrieve from Google Drive')],
        default='local_file', required=True)
    # generic
    url = fields.Char('External URL', help="URL of the Google Drive file or URL of the YouTube video")
    binary_content = fields.Binary('File', attachment=True)
    slide_resource_ids = fields.One2many('slide.slide.resource', 'slide_id', string="Additional Resource for this slide", copy=True)
    slide_resource_downloadable = fields.Boolean('Allow Download', default=False, help="Allow the user to download the content of the slide.")
    # google
    google_drive_id = fields.Char('Google Drive ID of the external URL', compute='_compute_google_drive_id')
    # content - webpage
    html_content = fields.Html(
        "HTML Content", translate=True,
        sanitize_attributes=False, sanitize_form=False, sanitize_overridable=True,
        help="Custom HTML content for slides of category 'Article'.")
    # content - images
    image_binary_content = fields.Binary('Image Content', related='binary_content', readonly=False) # Used to filter file input to images only
    image_google_url = fields.Char('Image Link', related='url', readonly=False,
        help="Link of the image (we currently only support Google Drive as source)")
    # content - documents
    slide_icon_class = fields.Char('Slide Icon fa-class', compute='_compute_slide_icon_class')
    slide_type = fields.Selection([
        ('image', 'Image'),
        ('article', 'Article'),
        ('quiz', 'Quiz'),
        ('pdf', 'PDF'),
        ('sheet', 'Sheet (Excel, Google Sheet, ...)'),
        ('doc', 'Document (Word, Google Doc, ...)'),
        ('slides', 'Slides (PowerPoint, Google Slides, ...)'),
        ('youtube_video', 'YouTube Video'),
        ('google_drive_video', 'Google Drive Video'),
        ('vimeo_video', 'Vimeo Video')],
        string="Slide Type", compute='_compute_slide_type', store=True, readonly=False,
        help="Subtype of the slide category, allows more precision on the actual file type / source type.")
    document_google_url = fields.Char('Document Link', related='url', readonly=False,
        help="Link of the document (we currently only support Google Drive as source)")
    document_binary_content = fields.Binary('PDF Content', related='binary_content', readonly=False) # Used to filter file input to PDF only
    # content - videos
    video_url = fields.Char('Video Link', related='url', readonly=False,
        help="Link of the video (we support YouTube, Google Drive and Vimeo as sources)")
    video_source_type = fields.Selection([
        ('youtube', 'YouTube'),
        ('google_drive', 'Google Drive'),
        ('vimeo', 'Vimeo')],
        string='Video Source', compute="_compute_video_source_type")
    youtube_id = fields.Char('Video YouTube ID', compute='_compute_youtube_id')
    vimeo_id = fields.Char('Video Vimeo ID', compute='_compute_vimeo_id')
    # website
    website_id = fields.Many2one(related='channel_id.website_id', readonly=True)
    date_published = fields.Datetime('Publish Date', readonly=True, tracking=False, copy=False)
    likes = fields.Integer('Likes', compute='_compute_like_info', store=True, compute_sudo=False)
    dislikes = fields.Integer('Dislikes', compute='_compute_like_info', store=True, compute_sudo=False)
    embed_code = fields.Html('Embed Code', readonly=True, compute='_compute_embed_code', sanitize=False)
    embed_code_external = fields.Html('External Embed Code', readonly=True, compute='_compute_embed_code', sanitize=False,
        help="Same as 'Embed Code' but used to embed the content on an external website.")
    website_share_url = fields.Char('Share URL', compute='_compute_website_share_url')
    # views
    embed_ids = fields.One2many('slide.embed', 'slide_id', string="External Slide Embeds")
    embed_count = fields.Integer('# of Embeds', compute='_compute_embed_counts')
    slide_views = fields.Integer('# of Website Views', store=True, compute="_compute_slide_views")
    public_views = fields.Integer('# of Public Views', copy=False, default=0, readonly=True)
    total_views = fields.Integer("# Total Views", default="0", compute='_compute_total', store=True)
    # comments
    comments_count = fields.Integer('Number of comments', compute="_compute_comments_count")
    # channel
    channel_type = fields.Selection(related="channel_id.channel_type", string="Channel type")
    channel_allow_comment = fields.Boolean(related="channel_id.allow_comment", string="Allows comment")
    # Statistics in case the slide is a category
    nbr_document = fields.Integer("Number of Documents", compute='_compute_slides_statistics', store=True)
    nbr_video = fields.Integer("Number of Videos", compute='_compute_slides_statistics', store=True)
    nbr_infographic = fields.Integer("Number of Images", compute='_compute_slides_statistics', store=True)
    nbr_article = fields.Integer("Number of Articles", compute='_compute_slides_statistics', store=True)
    nbr_quiz = fields.Integer("Number of Quizs", compute="_compute_slides_statistics", store=True)
    total_slides = fields.Integer(compute='_compute_slides_statistics', store=True)
    is_published = fields.Boolean(tracking=1)
    website_published = fields.Boolean(tracking=False)

    _sql_constraints = [
        ('exclusion_html_content_and_url', "CHECK(html_content IS NULL OR url IS NULL)", "A slide is either filled with a url or HTML content. Not both.")
    ]

    @api.depends('slide_category', 'source_type', 'image_binary_content')
    def _compute_image_1920(self):
        for slide in self:
            if slide.slide_category == 'infographic' and slide.source_type == 'local_file' and slide.image_binary_content:
                slide.image_1920 = slide.image_binary_content
            elif not slide.image_1920:
                slide.image_1920 = False

    @api.depends('date_published', 'is_published')
    def _compute_is_new_slide(self):
        for slide in self:
            slide.is_new_slide = slide.date_published > fields.Datetime.now() - relativedelta(days=7) if slide.is_published else False

    def _get_placeholder_filename(self, field):
        return self.channel_id._get_placeholder_filename(field)

    @api.depends('channel_id.slide_ids.is_category', 'channel_id.slide_ids.sequence', 'channel_id.slide_ids.slide_ids')
    def _compute_category_id(self):
        """ Will take all the slides of the channel for which the index is higher
        than the index of this category and lower than the index of the next category.

        Lists are manually sorted because when adding a new browse record order
        will not be correct as the added slide would actually end up at the
        first place no matter its sequence."""
        self.category_id = False  # initialize whatever the state

        channel_slides = {}
        for slide in self:
            if slide.channel_id.id not in channel_slides:
                channel_slides[slide.channel_id.id] = slide.channel_id.slide_ids

        for cid, slides in channel_slides.items():
            current_category = self.env['slide.slide']
            slide_list = list(slides)
            slide_list.sort(key=lambda s: (s.sequence, not s.is_category))
            for slide in slide_list:
                if slide.is_category:
                    current_category = slide
                elif slide.category_id != current_category:
                    slide.category_id = current_category.id

    @api.depends('slide_category', 'question_ids', 'channel_id.is_member')
    @api.depends_context('uid')
    def _compute_mark_complete_actions(self):
        """Determine if the slide can be marked as (un)completed.

        We can't mark a slide with questions as completed manually because we need to
        complete the quiz first. But we can mark as uncompleted a slide with questions,
        and the answers will be reset, the karma removed, etc (see mark_uncompleted).
        """
        for slide in self:
            slide.can_self_mark_uncompleted = slide.website_published and slide.channel_id.is_member
            slide.can_self_mark_completed = (
                slide.website_published
                and slide.channel_id.is_member
                and slide.slide_category != 'quiz'
                and not slide.question_ids
            )

    @api.depends('question_ids')
    def _compute_questions_count(self):
        for slide in self:
            slide.questions_count = len(slide.question_ids)

    @api.depends('website_message_ids.res_id', 'website_message_ids.model', 'website_message_ids.message_type')
    def _compute_comments_count(self):
        for slide in self:
            slide.comments_count = len(slide.website_message_ids)

    @api.depends('slide_views', 'public_views')
    def _compute_total(self):
        for record in self:
            record.total_views = record.slide_views + record.public_views

    @api.depends('slide_partner_ids.vote')
    def _compute_like_info(self):
        rg_data = self.env['slide.slide.partner'].sudo()._read_group(
            [('slide_id', 'in', self.ids), ('vote', 'in', (-1, 1))],
            ['slide_id', 'vote'], ['__count'],
        )
        mapped_data = {
            (slide.id, vote): count
            for slide, vote, count in rg_data
        }

        for slide in self:
            slide.likes = mapped_data.get((slide.id, 1), 0)
            slide.dislikes = mapped_data.get((slide.id, -1), 0)

    @api.depends('slide_partner_ids.slide_id')
    def _compute_slide_views(self):
        # TODO awa: tried compute_sudo, for some reason it doesn't work in here...
        read_group_res = self.env['slide.slide.partner'].sudo()._read_group(
            [('slide_id', 'in', self.ids)],
            ['slide_id'],
            aggregates=['__count'],
        )
        mapped_data = {slide.id: count for slide, count in read_group_res}
        for slide in self:
            slide.slide_views = mapped_data.get(slide.id, 0)

    @api.depends('embed_ids.slide_id')
    def _compute_embed_counts(self):
        read_group_res = self.env['slide.embed']._read_group(
            [('slide_id', 'in', self.ids)],
            ['slide_id'],
            ['count_views:sum'],
        )
        mapped_data = {
            slide.id: count_views_sum
            for slide, count_views_sum in read_group_res
        }

        for slide in self:
            slide.embed_count = mapped_data.get(slide.id, 0)

    @api.depends('slide_ids.sequence', 'slide_ids.active', 'slide_ids.slide_category', 'slide_ids.is_published', 'slide_ids.is_category')
    def _compute_slides_statistics(self):
        # Do not use dict.fromkeys(self.ids, dict()) otherwise it will use the same dictionnary for all keys.
        # Therefore, when updating the dict of one key, it updates the dict of all keys.
        keys = ['nbr_%s' % slide_category for slide_category in self.env['slide.slide']._fields['slide_category'].get_values(self.env)]
        default_vals = dict((key, 0) for key in keys + ['total_slides'])

        res = self.env['slide.slide']._read_group(
            [('is_published', '=', True), ('category_id', 'in', self.filtered('is_category').ids), ('is_category', '=', False)],
            ['category_id', 'slide_category'], ['__count'])

        result = {category_id: dict(default_vals) for category_id in self.ids}
        for category, slide_category, count in res:
            result[category.id][f'nbr_{slide_category}'] = count
            result[category.id]['total_slides'] += count

        for record in self:
            record.update(result.get(record._origin.id, default_vals))

    @api.depends('category_id', 'category_id.slide_ids', 'category_id.slide_ids.user_has_completed')
    def _compute_category_completed(self):
        for slide in self:
            if not slide.category_id:
                slide.user_has_completed_category = False
            else:
                slide.user_has_completed_category = all(slide.category_id.slide_ids.mapped('user_has_completed'))

    @api.depends('slide_ids.sequence', 'slide_ids.active', 'slide_ids.completion_time', 'slide_ids.is_published', 'slide_ids.is_category')
    def _compute_category_completion_time(self):
        # We don't use read_group() function, otherwise we will have issue with flushing the
        # data as completion_time is recursive and when it'll try to flush data before it is calculated
        for category in self.filtered(lambda slide: slide.is_category):
            filtered_slides = category.slide_ids.filtered(lambda slide: slide.is_published)
            category.completion_time = sum(filtered_slides.mapped("completion_time"))

    @api.depends('slide_type')
    def _compute_slide_icon_class(self):
        icon_per_slide_type = {
            'image': 'fa-file-picture-o',
            'article': 'fa-file-text-o',
            'quiz': 'fa-question-circle-o',
            'pdf': 'fa-file-pdf-o',
            'sheet': 'fa-file-excel-o',
            'doc': 'fa-file-word-o',
            'slides': 'fa-file-powerpoint-o',
            'youtube_video': 'fa-youtube-play',
            'google_drive_video': 'fa-play-circle-o',
            'vimeo_video': 'fa-vimeo',
        }
        for slide in self:
            slide.slide_icon_class = icon_per_slide_type.get(slide.slide_type, 'fa-file-o')

    @api.depends('slide_category', 'source_type', 'video_source_type')
    def _compute_slide_type(self):
        """ For 'local content' or specific slide categories, the slide type is directly derived
        from the slide category.

        For external content, the slide type is determined from the metadata and the mime_type.
        (See #_fetch_google_drive_metadata() for more details)."""

        for slide in self:
            if slide.slide_category == 'document':
                if slide.source_type == 'local_file':
                    slide.slide_type = 'pdf'
                elif slide.slide_type not in ['pdf', 'sheet', 'doc', 'slides']:
                    slide.slide_type = False
            elif slide.slide_category == 'infographic':
                slide.slide_type = 'image'
            elif slide.slide_category == 'article':
                slide.slide_type = 'article'
            elif slide.slide_category == 'quiz':
                slide.slide_type = 'quiz'
            elif slide.slide_category == 'video' and slide.video_source_type == 'youtube':
                slide.slide_type = 'youtube_video'
            elif slide.slide_category == 'video' and slide.video_source_type == 'google_drive':
                slide.slide_type = 'google_drive_video'
            elif slide.slide_category == 'video' and slide.video_source_type == 'vimeo':
                slide.slide_type = 'vimeo_video'
            else:
                slide.slide_type = False

    @api.depends('slide_partner_ids.partner_id', 'slide_partner_ids.vote', 'slide_partner_ids.completed')
    @api.depends('uid')
    def _compute_user_membership_id(self):
        slide_partners = self.env['slide.slide.partner'].sudo().search([
            ('slide_id', 'in', self.ids),
            ('partner_id', '=', self.env.user.partner_id.id),
        ])

        for record in self:
            record.user_membership_id = next(
                (slide_partner for slide_partner in slide_partners if slide_partner.slide_id == record),
                self.env['slide.slide.partner']
            )
            record.user_vote = record.user_membership_id.vote
            record.user_has_completed = record.user_membership_id.completed

    @api.depends('slide_category', 'google_drive_id', 'video_source_type', 'youtube_id')
    def _compute_embed_code(self):
        request_base_url = request.httprequest.url_root if request else False
        for slide in self:
            base_url = request_base_url or slide.get_base_url()
            if base_url[-1] == '/':
                base_url = base_url[:-1]

            embed_code = False
            embed_code_external = False
            if slide.slide_category == 'video':
                if slide.video_source_type == 'youtube':
                    query_params = urls.url_parse(slide.video_url).query
                    query_params = query_params + '&theme=light' if query_params else 'theme=light'
                    embed_code = Markup('<iframe src="//www.youtube-nocookie.com/embed/%s?%s" allowFullScreen="true" frameborder="0" aria-label="%s"></iframe>') % (slide.youtube_id, query_params, _('YouTube'))
                elif slide.video_source_type == 'google_drive':
                    embed_code = Markup('<iframe src="//drive.google.com/file/d/%s/preview" allowFullScreen="true" frameborder="0" aria-label="%s"></iframe>') % (slide.google_drive_id, _('Google Drive'))
                elif slide.video_source_type == 'vimeo':
                    if '/' in slide.vimeo_id:
                        # in case of privacy 'with URL only', vimeo adds a token after the video ID
                        # the embed url needs to receive that token as a "h" parameter
                        [vimeo_id, vimeo_token] = slide.vimeo_id.split('/')
                        embed_code = Markup("""
                            <iframe src="https://player.vimeo.com/video/%s?h=%s&badge=0&amp;autopause=0&amp;player_id=0"
                                frameborder="0" allow="autoplay; fullscreen; picture-in-picture" allowfullscreen aria-label="%s"></iframe>""") % (
                                vimeo_id, vimeo_token, _('Vimeo'))
                    else:
                        embed_code = Markup("""
                            <iframe src="https://player.vimeo.com/video/%s?badge=0&amp;autopause=0&amp;player_id=0"
                                frameborder="0" allow="autoplay; fullscreen; picture-in-picture" allowfullscreen aria-label="%s"></iframe>""") % (slide.vimeo_id, _('Vimeo'))
            elif slide.slide_category in ['infographic', 'document'] and slide.source_type == 'external' and slide.google_drive_id:
                embed_code = Markup('<iframe src="//drive.google.com/file/d/%s/preview" allowFullScreen="true" frameborder="0" aria-label="%s"></iframe>') % (slide.google_drive_id, _('Google Drive'))
            elif slide.slide_category == 'document' and slide.source_type == 'local_file':
                slide_url = base_url + self.env['ir.http']._url_for('/slides/embed/%s?page=1' % slide.id)
                slide_url_external = base_url + self.env['ir.http']._url_for('/slides/embed_external/%s?page=1' % slide.id)
                base_embed_code = Markup('<iframe src="%s" class="o_wslides_iframe_viewer" allowFullScreen="true" height="%s" width="%s" frameborder="0" aria-label="%s"></iframe>')
                iframe_aria_label = _('Embed code')
                embed_code = base_embed_code % (slide_url, 315, 420, iframe_aria_label)
                embed_code_external = base_embed_code % (slide_url_external, 315, 420, iframe_aria_label)

            slide.embed_code = embed_code
            slide.embed_code_external = embed_code_external or embed_code

    @api.depends('video_url')
    def _compute_video_source_type(self):
        for slide in self:
            video_source_type = False
            youtube_match = re.match(self.YOUTUBE_VIDEO_ID_REGEX, slide.video_url) if slide.video_url else False
            if youtube_match and len(youtube_match.groups()) == 2 and len(youtube_match.group(2)) == 11:
                video_source_type = 'youtube'
            if slide.video_url and not video_source_type and re.match(self.GOOGLE_DRIVE_DOCUMENT_ID_REGEX, slide.video_url):
                video_source_type = 'google_drive'
            vimeo_match = re.search(self.VIMEO_VIDEO_ID_REGEX, slide.video_url) if slide.video_url else False
            if not video_source_type and vimeo_match and len(vimeo_match.groups()) == 3:
                video_source_type = 'vimeo'

            slide.video_source_type = video_source_type

    @api.depends('video_url', 'video_source_type')
    def _compute_youtube_id(self):
        for slide in self:
            if slide.video_url and slide.video_source_type == 'youtube':
                match = re.match(self.YOUTUBE_VIDEO_ID_REGEX, slide.video_url)
                if match and len(match.groups()) == 2 and len(match.group(2)) == 11:
                    slide.youtube_id = match.group(2)
                else:
                    slide.youtube_id = False
            else:
                slide.youtube_id = False

    @api.depends('video_url', 'video_source_type')
    def _compute_vimeo_id(self):
        for slide in self:
            if slide.video_url and slide.video_source_type == 'vimeo':
                match = re.search(self.VIMEO_VIDEO_ID_REGEX, slide.video_url)
                if match and len(match.groups()) == 3:
                    if match.group(3):
                        # in case of privacy 'with URL only', vimeo adds a token after the video ID
                        # the share url is then 'vimeo_id/token'
                        # the token will be captured in the third group of the regex (if any)
                        slide.vimeo_id = '%s/%s' % (match.group(2), match.group(3))
                    else:
                        # regular video, we just capture the vimeo_id
                        slide.vimeo_id = match.group(2)
            else:
                slide.vimeo_id = False

    @api.depends('url', 'document_google_url', 'image_google_url', 'video_url')
    def _compute_google_drive_id(self):
        """ Extracts the Google Drive ID from the url based on the slide category. """

        for slide in self:
            url = slide.url or slide.document_google_url or slide.image_google_url or slide.video_url
            google_drive_id = False
            if url:
                match = re.match(self.GOOGLE_DRIVE_DOCUMENT_ID_REGEX, url)
                if match and len(match.groups()) == 2:
                    google_drive_id = match.group(2)

            slide.google_drive_id = google_drive_id

    @api.onchange('url', 'document_google_url', 'image_google_url', 'video_url')
    def _on_change_url(self):
        """ Keeping a 'onchange' because we want this behavior for the frontend.
        Changing the document / video external URL will populate some metadata on the form view.
        We only populate the field that are empty to avoid overriding user assigned values.
        The slide metadata are also fetched in create / write overrides to ensure consistency. """

        self.ensure_one()
        if self.url or self.document_google_url or self.image_google_url or self.video_url:
            slide_metadata, _error = self._fetch_external_metadata()
            if slide_metadata:
                self.update({
                    key: value
                    for key, value in slide_metadata.items()
                    if not self[key]
                })

    @api.onchange('document_binary_content')
    def _on_change_document_binary_content(self):
        if self.slide_category == 'document' and self.source_type == 'local_file' and self.document_binary_content:
            completion_time = self._get_completion_time_pdf(base64.b64decode(self.document_binary_content))
            if completion_time:
                self.completion_time = completion_time

    @api.onchange('slide_category')
    def _on_change_slide_category(self):
        """ Prevents mis-match when ones uploads an image and then a pdf without saving the form. """
        if self.slide_category != 'infographic' and self.image_binary_content:
            self.image_binary_content = False
        elif self.slide_category != 'document' and self.document_binary_content:
            self.document_binary_content = False

    @api.depends('name', 'channel_id.website_id.domain')
    def _compute_website_url(self):
        super(Slide, self)._compute_website_url()
        for slide in self:
            if slide.id:  # avoid to perform a slug on a not yet saved record in case of an onchange.
                base_url = slide.channel_id.get_base_url()
                slide.website_url = '%s/slides/slide/%s' % (base_url, self.env['ir.http']._slug(slide))

    @api.depends('is_published')
    def _compute_website_share_url(self):
        self.website_share_url = False
        for slide in self:
            if slide.id:  # ensure we can build the URL
                base_url = slide.channel_id.get_base_url()
                slide.website_share_url = '%s/slides/slide/%s/share' % (base_url, slide.id)

    @api.depends('channel_id.can_publish')
    def _compute_can_publish(self):
        for record in self:
            record.can_publish = record.channel_id.can_publish

    @api.model
    def _get_can_publish_error_message(self):
        return _("Publishing is restricted to the responsible of training courses or members of the publisher group for documentation courses")

    # ---------------------------------------------------------
    # ORM Overrides
    # ---------------------------------------------------------

    @api.model_create_multi
    def create(self, vals_list):
        channel_ids = [vals['channel_id'] for vals in vals_list]
        can_publish_channel_ids = self.env['slide.channel'].browse(channel_ids).filtered(lambda c: c.can_publish).ids
        for vals in vals_list:
            # Do not publish slide if user has not publisher rights
            if vals['channel_id'] not in can_publish_channel_ids:
                # 'website_published' is handled by mixin
                vals['date_published'] = False

            if vals.get('is_category'):
                vals['is_preview'] = True
                vals['is_published'] = True
            if vals.get('is_published') and not vals.get('date_published'):
                vals['date_published'] = datetime.datetime.now()

        slides = super().create(vals_list)

        for slide, vals in zip(slides, vals_list):
            # avoid fetching external metadata when installing the module (i.e. for demo data)
            # we also support a context key if you don't want to fetch the metadata when creating a slide
            if any(vals.get(url_param) for url_param in ['url', 'video_url', 'document_google_url', 'image_google_url']) \
               and not self.env.context.get('install_mode') \
               and not self.env.context.get('website_slides_skip_fetch_metadata'):
                slide_metadata, _error = slide._fetch_external_metadata()
                if slide_metadata:
                    # only update keys that are not set in the incoming vals
                    slide.update({key: value for key, value in slide_metadata.items() if key not in vals.keys()})

            if 'completion_time' not in vals:
                slide._on_change_document_binary_content()

            if slide.is_published and not slide.is_category:
                slide._post_publication()
                slide.channel_id.channel_partner_ids._recompute_completion()
        return slides

    def write(self, values):
        if values.get('is_category'):
            values['is_preview'] = True
            values['is_published'] = True

        # if the slide type is changed, remove incompatible url or html_content
        # done here to satisfy the SQL constraint
        # using a stored-computed field in place does not work
        if 'slide_category' in values:
            if values['slide_category'] == 'article':
                values = {'url': False, **values}
            elif values['slide_category'] != 'article':
                values = {'html_content': False, **values}

        res = super(Slide, self).write(values)
        if values.get('is_published'):
            self.date_published = datetime.datetime.now()
            self._post_publication()

        # avoid fetching external metadata when installing the module (i.e. for demo data)
        # we also support a context key if you don't want to fetch the metadata when modifying a slide
        if any(values.get(url_param) for url_param in ['url', 'video_url', 'document_google_url', 'image_google_url']) \
           and not self.env.context.get('install_mode') \
           and not self.env.context.get('website_slides_skip_fetch_metadata'):
            slide_metadata, _error = self._fetch_external_metadata()
            if slide_metadata:
                # only update keys that are not set in the incoming values and for which we don't have a value yet
                self.update({
                    key: value
                    for key, value in slide_metadata.items()
                    if key not in values.keys() and not any(slide[key] for slide in self)
                })

        if 'is_published' in values or 'active' in values:
            # recompute the completion for all partners of the channel
            self.channel_id.channel_partner_ids._recompute_completion()

        return res

    def copy_data(self, default=None):
        """Sets the sequence to zero so that it always lands at the beginning
        of the newly selected course as an uncategorized slide"""
        default = dict(default or {})
        if 'slide.channel' not in self._context.get('__copy_data_seen', {}) and 'sequence' not in default:
            default['sequence'] = 0
        return super().copy_data(default=default)

    def unlink(self):
        for category in self.filtered(lambda slide: slide.is_category):
            category.channel_id._move_category_slides(category, False)
        channel_partner_ids = self.channel_id.channel_partner_ids
        res = super(Slide, self).unlink()
        channel_partner_ids._recompute_completion()
        return res

    def toggle_active(self):
        # archiving/unarchiving a channel does it on its slides, too
        to_archive = self.filtered(lambda slide: slide.active)
        res = super(Slide, self).toggle_active()
        if to_archive:
            to_archive.filtered(lambda slide: not slide.is_category).is_published = False
        return res

    # ---------------------------------------------------------
    # Mail/Rating
    # ---------------------------------------------------------

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, *, message_type='notification', **kwargs):
        self.ensure_one()
        if message_type == 'comment' and not self.channel_id.can_comment:  # user comments have a restriction on karma
            raise AccessError(_('Not enough karma to comment'))
        return super(Slide, self).message_post(message_type=message_type, **kwargs)

    def _get_access_action(self, access_uid=None, force_website=False):
        """ Instead of the classic form view, redirect to website if it is published. """
        self.ensure_one()
        if force_website or self.website_published:
            return {
                'type': 'ir.actions.act_url',
                'url': '%s' % self.website_url,
                'target': 'self',
                'target_type': 'public',
                'res_id': self.id,
            }
        return super(Slide, self)._get_access_action(access_uid=access_uid, force_website=force_website)

    def _notify_get_recipients_groups(self, message, model_description, msg_vals=None):
        """ Add access button to everyone if the document is active. """
        groups = super()._notify_get_recipients_groups(
            message, model_description, msg_vals=msg_vals
        )
        if not self:
            return groups

        self.ensure_one()
        if self.website_published:
            for _group_name, _group_method, group_data in groups:
                group_data['has_button_access'] = True

        return groups

    # ---------------------------------------------------------
    # Business Methods
    # ---------------------------------------------------------

    def _embed_increment(self, url):
        """ Increment the view count of the record we have based on the passed url.
        If the url is empty, which typically happens if the browser does not pass the 'referer'
        header properly, then we increment the entry that has 'False' as url value. """

        self.ensure_one()

        url_entry = url
        if not urls.url_parse(url).netloc:
            url_entry = False

        embed_entry = self.env['slide.embed'].search([
            ('url', '=', url_entry),
            ('slide_id', '=', self.id)
        ], limit=1)

        if embed_entry:
            embed_entry.count_views += 1
        else:
            embed_entry = self.env['slide.embed'].create({
                'slide_id': self.id,
                'url': url_entry,
            })

        return embed_entry

    def _post_publication(self):
        for slide in self.filtered(lambda slide: slide.website_published and slide.channel_id.publish_template_id):
            publish_template = slide.channel_id.publish_template_id
            html_body = publish_template.with_context(base_url=slide.get_base_url())._render_field('body_html', slide.ids)[slide.id]
            subject = publish_template._render_field('subject', slide.ids)[slide.id]
            # We want to use the 'reply_to' of the template if set. However, `mail.message` will check
            # if the key 'reply_to' is in the kwargs before calling _get_reply_to. If the value is
            # falsy, we don't include it in the 'message_post' call.
            kwargs = {}
            reply_to = publish_template._render_field('reply_to', slide.ids)[slide.id]
            if reply_to:
                kwargs['reply_to'] = reply_to
            slide.channel_id.with_context(mail_create_nosubscribe=True).message_post(
                subject=subject,
                body=html_body,
                subtype_xmlid='website_slides.mt_channel_slide_published',
                email_layout_xmlid='mail.mail_notification_light',
                **kwargs,
            )
        return True

    def _generate_signed_token(self, partner_id):
        """ Lazy generate the acces_token and return it signed by the given partner_id
            :rtype tuple (string, int)
            :return (signed_token, partner_id)
        """
        if not self.access_token:
            self.write({'access_token': self._default_access_token()})
        return self._sign_token(partner_id)

    def _send_share_email(self, email, fullscreen):
        courses_without_templates = self.channel_id.filtered(lambda channel: not channel.share_slide_template_id)
        if courses_without_templates:
            raise UserError(_('Impossible to send emails. Select a "Share Template" for courses %(course_names)s first',
                                 course_names=', '.join(courses_without_templates.mapped('name'))))
        mail_ids = []
        for record in self:
            template = record.channel_id.share_slide_template_id.with_context(
                user=self.env.user,
                email=email,
                base_url=record.get_base_url(),
                fullscreen=fullscreen
            )
            email_values = {'email_to': email}
            if self.env.user._is_portal():
                template = template.sudo()
                email_values['email_from'] = self.env.company.catchall_formatted or self.env.company.email_formatted

            mail_ids.append(template.send_mail(record.id, email_layout_xmlid='mail.mail_notification_light', email_values=email_values))
        return mail_ids

    def action_like(self):
        self.check_access('read')
        return self._action_vote(upvote=True)

    def action_dislike(self):
        self.check_access('read')
        return self._action_vote(upvote=False)

    def _action_vote(self, upvote=True):
        """ Private implementation of voting. It does not check for any real access
        rights; public methods should grant access before calling this method.

          :param upvote: if True, is a like; if False, is a dislike
        """
        self_sudo = self.sudo()
        SlidePartnerSudo = self.env['slide.slide.partner'].sudo()
        slide_partners = SlidePartnerSudo.search([
            ('slide_id', 'in', self.ids),
            ('partner_id', '=', self.env.user.partner_id.id)
        ])
        slide_id = slide_partners.mapped('slide_id')
        new_slides = self_sudo - slide_id

        for slide_partner in slide_partners:
            if upvote:
                slide_partner.vote = 0 if slide_partner.vote == 1 else 1
            else:
                slide_partner.vote = 0 if slide_partner.vote == -1 else -1

        for new_slide in new_slides:
            new_vote = 1 if upvote else -1
            new_slide.write({
                'slide_partner_ids': [(0, 0, {'vote': new_vote, 'partner_id': self.env.user.partner_id.id})]
            })

    def action_set_viewed(self, quiz_attempts_inc=False):
        if any(not slide.channel_id.is_member for slide in self):
            raise UserError(_('You cannot mark a slide as viewed if you are not among its members.'))

        return bool(self._action_set_viewed(self.env.user.partner_id, quiz_attempts_inc=quiz_attempts_inc))

    def _action_set_viewed(self, target_partner, quiz_attempts_inc=False):
        self_sudo = self.sudo()
        SlidePartnerSudo = self.env['slide.slide.partner'].sudo()
        existing_sudo = SlidePartnerSudo.search([
            ('slide_id', 'in', self.ids),
            ('partner_id', '=', target_partner.id)
        ])
        if quiz_attempts_inc and existing_sudo:
            sql.increment_fields_skiplock(existing_sudo, 'quiz_attempts_count')
            existing_sudo.invalidate_recordset(['quiz_attempts_count'])

        new_slides = self_sudo - existing_sudo.mapped('slide_id')
        return SlidePartnerSudo.create([{
            'slide_id': new_slide.id,
            'channel_id': new_slide.channel_id.id,
            'partner_id': target_partner.id,
            'quiz_attempts_count': 1 if quiz_attempts_inc else 0,
            'vote': 0} for new_slide in new_slides])

    def action_mark_completed(self):
        if any(not slide.can_self_mark_completed for slide in self):
            raise UserError(_('You cannot mark a slide as completed if you are not among its members.'))

        return self._action_mark_completed()

    def _action_mark_completed(self):
        uncompleted_slides = self.filtered(lambda slide: not slide.user_has_completed)

        target_partner = self.env.user.partner_id
        uncompleted_slides._action_set_quiz_done()
        SlidePartnerSudo = self.env['slide.slide.partner'].sudo()
        existing_sudo = SlidePartnerSudo.search([
            ('slide_id', 'in', uncompleted_slides.ids),
            ('partner_id', '=', target_partner.id)
        ])
        existing_sudo.write({'completed': True})

        new_slides = uncompleted_slides.sudo() - existing_sudo.mapped('slide_id')
        SlidePartnerSudo.create([{
            'slide_id': new_slide.id,
            'channel_id': new_slide.channel_id.id,
            'partner_id': target_partner.id,
            'vote': 0,
            'completed': True} for new_slide in new_slides])

    def action_mark_uncompleted(self):
        if any(not slide.can_self_mark_uncompleted for slide in self):
            raise UserError(_('You cannot mark a slide as uncompleted if you are not among its members.'))

        completed_slides = self.filtered(lambda slide: slide.user_has_completed)

        # Remove the Karma point gained
        completed_slides._action_set_quiz_done(completed=False)

        self.env['slide.slide.partner'].sudo().search([
            ('slide_id', 'in', completed_slides.ids),
            ('partner_id', '=', self.env.user.partner_id.id),
        ]).completed = False

    def _action_set_quiz_done(self, completed=True):
        """Add or remove karma point related to the quiz.

        :param completed:
            True if the quiz will be marked as completed (karma will be increased)
            If set to False, we will remove the karma instead of increasing it,
            so that the user can take the quiz multiple times but not gain karma infinitely
        """
        if any(not slide.channel_id.is_member or not slide.website_published for slide in self):
            raise UserError(
                _('You cannot mark a slide quiz as completed if you are not among its members or it is unpublished.') if completed
                else _('You cannot mark a slide quiz as not completed if you are not among its members or it is unpublished.')
            )

        points = 0
        for slide in self:
            user_membership_sudo = slide.user_membership_id.sudo()
            if not user_membership_sudo \
               or user_membership_sudo.completed == completed \
               or not user_membership_sudo.quiz_attempts_count \
               or not slide.question_ids:
                continue

            gains = [slide.quiz_first_attempt_reward,
                     slide.quiz_second_attempt_reward,
                     slide.quiz_third_attempt_reward,
                     slide.quiz_fourth_attempt_reward]
            points = gains[min(user_membership_sudo.quiz_attempts_count, len(gains)) - 1]
            if points:
                if completed:
                    reason = _('Quiz Completed')
                else:
                    points *= -1
                    reason = _('Quiz Set Uncompleted')
                self.env.user.sudo()._add_karma(points, slide, reason)

        return True

    def action_view_embeds(self):
        self.ensure_one()

        action = self.env["ir.actions.actions"]._for_xml_id("website_slides.slide_embed_action")
        action['context'] = {'search_default_slide_id': self.id}
        return action

    def _compute_quiz_info(self, target_partner, quiz_done=False):
        result = dict.fromkeys(self.ids, False)
        slide_partners = self.env['slide.slide.partner'].sudo().search([
            ('slide_id', 'in', self.ids),
            ('partner_id', '=', target_partner.id)
        ])
        slide_partners_map = dict((sp.slide_id.id, sp) for sp in slide_partners)
        for slide in self:
            if not slide.question_ids:
                gains = [0]
            else:
                gains = [slide.quiz_first_attempt_reward,
                         slide.quiz_second_attempt_reward,
                         slide.quiz_third_attempt_reward,
                         slide.quiz_fourth_attempt_reward]
            result[slide.id] = {
                'quiz_karma_max': gains[0],  # what could be gained if succeed at first try
                'quiz_karma_gain': gains[0],  # what would be gained at next test
                'quiz_karma_won': 0,  # what has been gained
                'quiz_attempts_count': 0,  # number of attempts
            }
            slide_partner = slide_partners_map.get(slide.id)
            if slide.question_ids and slide_partner and slide_partner.quiz_attempts_count:
                result[slide.id]['quiz_karma_gain'] = gains[slide_partner.quiz_attempts_count] if slide_partner.quiz_attempts_count < len(gains) else gains[-1]
                result[slide.id]['quiz_attempts_count'] = slide_partner.quiz_attempts_count
                if quiz_done or slide_partner.completed:
                    result[slide.id]['quiz_karma_won'] = gains[slide_partner.quiz_attempts_count-1] if slide_partner.quiz_attempts_count < len(gains) else gains[-1]
        return result

    # --------------------------------------------------
    # Parsing methods
    # --------------------------------------------------

    def _fetch_external_metadata(self, image_url_only=False):
        self.ensure_one()

        slide_metadata = {}
        error = False
        if self.slide_category == 'video' and self.video_source_type == 'youtube':
            slide_metadata, error = self._fetch_youtube_metadata(image_url_only)
        elif self.slide_category == 'video' and self.video_source_type == 'google_drive':
            slide_metadata, error = self._fetch_google_drive_metadata(image_url_only)
        elif self.slide_category == 'video' and self.video_source_type == 'vimeo':
            slide_metadata, error = self._fetch_vimeo_metadata(image_url_only)
        elif self.slide_category in ['document', 'infographic'] and self.source_type == 'external':
            # external documents & google drive videos share the same method currently
            slide_metadata, error = self._fetch_google_drive_metadata(image_url_only)

        return slide_metadata, error

    def _fetch_youtube_metadata(self, image_url_only=False):
        """ Fetches video metadata from the YouTube API.

        Returns a dict containing video metadata with the following keys (matching slide.slide fields):
        - 'name' matching the video title
        - 'description' matching the video description
        - 'image_1920' binary data of the video thumbnail
          OR 'image_url' containing an external link to the thumbnail when 'image_url_only' param is True
        - 'completion_time' matching the video duration
          The received duration is under a special format (e.g: PT1M21S15, meaning 1h 21m 15s).

        :param image_url_only: if True, will return 'image_url' instead of binary data
          Typically used when displaying a slide preview to the end user.
        :return a tuple (values, error) containing the values of the slide and a potential error
          (e.g: 'Video could not be found') """

        self.ensure_one()
        google_app_key = self.env['website'].get_current_website().sudo().website_slide_google_app_key
        error_message = False
        try:
            response = requests.get(
                'https://www.googleapis.com/youtube/v3/videos',
                timeout=3,
                params={
                    'fields': 'items(id,snippet,contentDetails)',
                    'id': self.youtube_id,
                    'key': google_app_key,
                    'part': 'snippet,contentDetails'
                }
            )
            response.raise_for_status()
        except requests.exceptions.HTTPError as e:
            error_message = e.response.content
            if 'application/json' in e.response.headers.get('content-type'):
                json_response = e.response.json()
                if json_response.get('error', {}).get('code') == 404:
                    return {}, _('Your video could not be found on YouTube, please check the link and/or privacy settings')
        except requests.exceptions.ConnectionError as e:
            error_message = str(e)

        if not error_message:
            response = response.json()
            if response.get('error'):
                error_message = response.get('error', {}).get('errors', [{}])[0].get('reason')

            if not response.get('items'):
                error_message = _('Your video could not be found on YouTube, please check the link and/or privacy settings')

        if error_message:
            _logger.warning('Could not fetch YouTube metadata: %s', error_message)
            return {}, error_message

        slide_metadata = {'slide_type': 'youtube_video'}
        youtube_values = response.get('items')[0]
        youtube_duration = youtube_values.get('contentDetails', {}).get('duration')
        if youtube_duration:
            parsed_duration = re.search(r'^PT(?:(\d+)H)?(?:(\d+)M)?(?:(\d+)S)?$', youtube_duration)
            if parsed_duration:
                slide_metadata['completion_time'] = (int(parsed_duration.group(1) or 0)) + \
                                                    (int(parsed_duration.group(2) or 0) / 60) + \
                                                    (round(int(parsed_duration.group(3) or 0) /60) / 60)

        if youtube_values.get('snippet'):
            snippet = youtube_values['snippet']
            slide_metadata.update({
                'name': snippet['title'],
                'description': snippet['description'],
            })

            thumbnail_url = snippet['thumbnails']['high']['url']
            if image_url_only:
                slide_metadata['image_url'] = thumbnail_url
            else:
                slide_metadata['image_1920'] = base64.b64encode(
                    requests.get(thumbnail_url, timeout=3).content
                )

        return slide_metadata, None

    def _fetch_google_drive_metadata(self, image_url_only=False):
        """ Fetches document / video metadata from the Google Drive API.

        Returns a dict containing metadata with the following keys (matching slide.slide fields):
        - 'name' matching the external file title
        - 'image_1920' binary data of the file thumbnail
          OR 'image_url' containing an external link to the thumbnail when 'image_url_only' param is True
        - 'completion_time' which is computed for 2 types of files:
          - pdf files where we download the content and then use slide.slide#_get_completion_time_pdf()
          - videos where we use the 'videoMediaMetadata' to extract the 'durationMillis'

        :param image_url_only: if True, will return 'image_url' instead of binary data
          Typically used when displaying a slide preview to the end user.
        :return a tuple (values, error) containing the values of the slide and a potential error
          (e.g: 'File could not be found') """

        params = {}
        params['projection'] = 'BASIC'
        if 'google.drive.config' in self.env:
            access_token = False
            try:
                access_token = self.env['google.drive.config'].get_access_token()
            except (RedirectWarning, UserError):
                pass  # ignore and use the 'key' fallback

            if access_token:
                params['access_token'] = access_token

        if not params.get('access_token'):
            params['key'] = self.env['website'].get_current_website().sudo().website_slide_google_app_key

        error_message = False
        try:
            response = requests.get(
                'https://www.googleapis.com/drive/v2/files/%s' % self.google_drive_id,
                timeout=3,
                params=params
            )
            response.raise_for_status()
        except requests.exceptions.HTTPError as e:
            error_message = e.response.content
            if 'application/json' in e.response.headers.get('content-type'):
                json_response = e.response.json()
                if json_response.get('error', {}).get('code') == 404:
                    # in case we don't find the file on GDrive, we want to give some feedback to our user
                    return {}, _('Your file could not be found on Google Drive, please check the link and/or privacy settings')
        except requests.exceptions.ConnectionError as e:
            error_message = str(e)

        if not error_message:
            response = response.json()
            if response.get('error'):
                error_message = response.get('error', {}).get('errors', [{}])[0].get('reason')

        if error_message:
            _logger.warning('Could not fetch Google Drive metadata: %s', error_message)
            return {}, error_message

        google_drive_values = response
        slide_metadata = {
            'name': google_drive_values.get('title')
        }

        if google_drive_values.get('thumbnailLink'):
            # small trick, we remove '=s220' to get a higher definition
            thumbnail_url = google_drive_values['thumbnailLink'].replace('=s220', '')
            if image_url_only:
                slide_metadata['image_url'] = thumbnail_url
            else:
                slide_metadata['image_1920'] = base64.b64encode(
                    requests.get(thumbnail_url, timeout=3).content
                )

        if self.slide_category == 'document':
            sheet_mimetypes = [
                'application/vnd.ms-excel',
                'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
                'application/vnd.oasis.opendocument.spreadsheet',
                'application/vnd.google-apps.spreadsheet'
            ]

            doc_mimetypes = [
                'application/msword',
                'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
                'application/vnd.oasis.opendocument.text',
                'application/vnd.google-apps.document'
            ]

            slides_mimetypes = [
                'application/vnd.ms-powerpoint',
                'application/vnd.openxmlformats-officedocument.presentationml.presentation',
                'application/vnd.oasis.opendocument.presentation',
                'application/vnd.google-apps.presentation'
            ]

            mime_type = google_drive_values.get('mimeType')
            if mime_type == 'application/pdf':
                slide_metadata['slide_type'] = 'pdf'
                if google_drive_values.get('downloadUrl'):
                    # attempt to download PDF content to extract a completion_time based on the number of pages
                    try:
                        pdf_response = requests.get(google_drive_values.get('downloadUrl'), timeout=5)
                        completion_time = self._get_completion_time_pdf(pdf_response.content)
                        if completion_time:
                            slide_metadata['completion_time'] = completion_time
                    except Exception:
                        pass  # fail silently as this is nice to have
            elif mime_type in sheet_mimetypes:
                slide_metadata['slide_type'] = 'sheet'
            elif mime_type in doc_mimetypes:
                slide_metadata['slide_type'] = 'doc'
            elif mime_type in slides_mimetypes:
                slide_metadata['slide_type'] = 'slides'
            elif mime_type and mime_type.startswith('image/'):
                # image and videos should be input using another "slide_category" but let's be nice and
                # assign them a matching slide_type
                slide_metadata['slide_type'] = 'image'
            elif mime_type and mime_type.startswith('video/'):
                slide_metadata['slide_type'] = 'google_drive_video'

        elif self.slide_category == 'video':
            completion_time = round(float(
                google_drive_values.get('videoMediaMetadata', {}).get('durationMillis', 0)
                ) / (60 * 1000)) / 60  # millis to hours conversion rounded to the minute
            if completion_time:
                slide_metadata['completion_time'] = completion_time

        return slide_metadata, None

    def _fetch_vimeo_metadata(self, image_url_only=False):
        """ Fetches video metadata from the Vimeo API.
        See https://developer.vimeo.com/api/oembed/showcases for more information.

        Returns a dict containing video metadata with the following keys (matching slide.slide fields):
        - 'name' matching the video title
        - 'description' matching the video description
        - 'image_1920' binary data of the video thumbnail
          OR 'image_url' containing an external link to the thumbnail when 'fetch_image' param is False
        - 'completion_time' matching the video duration

        :param image_url_only: if False, will return 'image_url' instead of binary data
          Typically used when displaying a slide preview to the end user.
        :return a tuple (values, error) containing the values of the slide and a potential error
          (e.g: 'Video could not be found') """

        self.ensure_one()
        error_message = False
        try:
            response = requests.get(
                'https://vimeo.com/api/oembed.json?%s' % urls.url_encode({'url': self.video_url}),
                timeout=3
            )
            response.raise_for_status()
        except requests.exceptions.HTTPError as e:
            error_message = e.response.content
            if e.response.status_code == 404:
                return {}, _('Your video could not be found on Vimeo, please check the link and/or privacy settings')
        except requests.exceptions.ConnectionError as e:
            error_message = str(e)

        if not error_message and 'application/json' in response.headers.get('content-type'):
            response = response.json()
            if response.get('error'):
                error_message = response.get('error', {}).get('errors', [{}])[0].get('reason')

            if not response:
                error_message = _('Please enter a valid Vimeo video link')

        if error_message:
            _logger.warning('Could not fetch Vimeo metadata: %s', error_message)
            return {}, error_message

        vimeo_values = response
        slide_metadata = {'slide_type': 'vimeo_video'}

        if vimeo_values.get('title'):
            slide_metadata['name'] = vimeo_values.get('title')

        if vimeo_values.get('description'):
            slide_metadata['description'] = vimeo_values.get('description')

        if vimeo_values.get('duration'):
            # seconds to hours conversion
            slide_metadata['completion_time'] = round(vimeo_values.get('duration') / 60) / 60

        thumbnail_url = vimeo_values.get('thumbnail_url')
        if thumbnail_url:
            if image_url_only:
                slide_metadata['image_url'] = thumbnail_url
            else:
                slide_metadata['image_1920'] = base64.b64encode(
                    requests.get(thumbnail_url, timeout=3).content
                )

        return slide_metadata, None

    def _default_website_meta(self):
        res = super(Slide, self)._default_website_meta()
        res['default_opengraph']['og:title'] = res['default_twitter']['twitter:title'] = self.name
        res['default_opengraph']['og:description'] = res['default_twitter']['twitter:description'] = html2plaintext(self.description)
        res['default_opengraph']['og:image'] = res['default_twitter']['twitter:image'] = self.env['website'].image_url(self, 'image_1024')
        res['default_meta_description'] = html2plaintext(self.description)
        return res

    # ---------------------------------------------------------
    # Data / Misc
    # ---------------------------------------------------------

    def _get_completion_time_pdf(self, data_bytes):
        """ For PDFs, we assume that it takes 5 minutes to read a page.
        This method receives the data of the PDF as bytes. """

        if data_bytes.startswith(b'%PDF-'):
            try:
                pdf = PdfFileReader(io.BytesIO(data_bytes), overwriteWarnings=False)
                return (5 * len(pdf.pages)) / 60
            except Exception:
                pass  # as this is a nice to have, fail silently

        return False

    def _get_next_category(self):
        channel_category_ids = self.channel_id.slide_category_ids.ids
        if not channel_category_ids:
            return self.env['slide.slide']
        # If current slide is uncategorized and all the channel uncategorized slides are completed, return the first category
        if not self.category_id and all(self.channel_id.slide_ids.filtered(
            lambda s: not s.is_category and not s.category_id).mapped('user_has_completed')):
            return self.env['slide.slide'].browse(channel_category_ids[0])
        # If current category is completed and current category is not the last one, get next category
        elif self.user_has_completed_category and self.category_id.id in channel_category_ids and self.category_id.id != channel_category_ids[-1]:
            index_current_category = channel_category_ids.index(self.category_id.id)
            return self.env['slide.slide'].browse(channel_category_ids[index_current_category+1])
        return self.env['slide.slide']

    def get_backend_menu_id(self):
        return self.env.ref('website_slides.website_slides_menu_root').id

    @api.model
    def _search_get_detail(self, website, order, options):
        with_description = options['displayDescription']
        search_fields = ['name']
        fetch_fields = ['id', 'name']
        mapping = {
            'name': {'name': 'name', 'type': 'text', 'match': True},
            'website_url': {'name': 'url', 'type': 'text', 'truncate': False},
            'extra_link': {'name': 'course', 'type': 'text'},
            'extra_link_url': {'name': 'course_url', 'type': 'text', 'truncate': False},
        }
        if with_description:
            search_fields.append('description')
            fetch_fields.append('description')
            mapping['description'] = {'name': 'description', 'type': 'text', 'html': True, 'match': True}
        return {
            'model': 'slide.slide',
            'base_domain': [website.website_domain()],
            'search_fields': search_fields,
            'fetch_fields': fetch_fields,
            'mapping': mapping,
            'icon': 'fa-shopping-cart',
            'order': 'name desc, id desc' if 'name desc' in order else 'name asc, id desc',
        }

    def _search_render_results(self, fetch_fields, mapping, icon, limit):
        icon_per_category = {
            'infographic': 'fa-file-picture-o',
            'article': 'fa-file-text',
            'presentation': 'fa-file-pdf-o',
            'document': 'fa-file-pdf-o',
            'video': 'fa-play-circle',
            'quiz': 'fa-question-circle',
            'link': 'fa-file-code-o', # appears in template "slide_icon"
        }
        results_data = super()._search_render_results(fetch_fields, mapping, icon, limit)
        for slide, data in zip(self, results_data):
            data['_fa'] = icon_per_category.get(slide.slide_category, 'fa-file-pdf-o')
            data['url'] = slide.website_url
            data['course'] = _('Course: %s', slide.channel_id.name)
            data['course_url'] = slide.channel_id.website_url
        return results_data

    def open_website_url(self):
        """ Overridden to use a relative URL instead of an absolute when website_id is False. """
        if self.website_id:
            return super().open_website_url()
        return self.env['website'].get_client_action(f'/slides/slide/{self.env["ir.http"]._slug(self)}')

    def _mail_get_partner_fields(self, introspect_fields=False):
        return []

```

## File: models\slide_slide_resource.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.urls import url_encode

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo.tools.mimetypes import get_extension


class SlideResource(models.Model):
    _name = 'slide.slide.resource'
    _description = "Additional resource for a particular slide"
    _order = "sequence, id"

    slide_id = fields.Many2one('slide.slide', required=True, ondelete='cascade')
    resource_type = fields.Selection([('file', 'File'), ('url', 'Link')], required=True)
    name = fields.Char('Name', compute="_compute_name", readonly=False, store=True)
    data = fields.Binary('Resource', compute='_compute_reset_resources', store=True, readonly=False)
    file_name = fields.Char(store=True)
    link = fields.Char('Link', compute='_compute_reset_resources', store=True, readonly=False)
    download_url = fields.Char('Download URL', compute='_compute_download_url')
    sequence = fields.Integer(string="Sequence")

    _sql_constraints = [
        ('check_url', "CHECK (resource_type != 'url' OR link IS NOT NULL)", 'A resource of type url must contain a link.'),
        ('check_file_type', "CHECK (resource_type != 'file' OR link IS NULL)", 'A resource of type file cannot contain a link.'),
    ]

    @api.depends('resource_type')
    def _compute_reset_resources(self):
        for resource in self:
            if resource.resource_type == 'file':
                resource.link = False
                resource.data = resource.data
            else:
                resource.data = False
                resource.link = resource.link

    @api.depends('file_name', 'resource_type', 'data', 'link')
    def _compute_name(self):
        for resource in self:
            to_update = not resource.name or resource.name == _("Resource")
            if to_update:
                new_name = _("Resource")
                if resource.resource_type == 'file' and (resource.data or resource.file_name):
                    new_name = resource.file_name
                elif resource.resource_type == 'url':
                    new_name = resource.link
                resource.name = new_name

    @api.depends('name', 'file_name')
    def _compute_download_url(self):
        for resource in self:
            extension = get_extension(resource.file_name) if resource.file_name else ''
            if not resource.name:
                resource.download_url = False
                continue
            file_name = resource.name if resource.name.endswith(extension) else resource.name + extension
            resource.download_url = f'/web/content/slide.slide.resource/{resource.id}/data?' + url_encode({
                'download': 'true',
                'filename': file_name,
            })

    @api.constrains('data')
    def _check_link_type(self):
        for record in self:
            if record.resource_type != 'file' and record.data:
                raise ValidationError(_("Resource %(resource_name)s is a link and should not contain a data file", resource_name=record.name))

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class Website(models.Model):
    _inherit = "website"

    website_slide_google_app_key = fields.Char('Google Doc Key', groups='base.group_system')

    def get_suggested_controllers(self):
        suggested_controllers = super(Website, self).get_suggested_controllers()
        suggested_controllers.append((_('Courses'), self.env['ir.http']._url_for('/slides'), 'website_slides'))
        return suggested_controllers

    def _search_get_details(self, search_type, order, options):
        result = super()._search_get_details(search_type, order, options)
        if search_type in ['slides', 'slide_channels_only', 'all']:
            result.append(self.env['slide.channel']._search_get_detail(self, order, options))
        if search_type in ['slides', 'slides_only', 'all']:
            result.append(self.env['slide.slide']._search_get_detail(self, order, options))
        return result

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import gamification_challenge
from . import gamification_karma_tracking
from . import slide_slide
from . import slide_question
from . import slide_embed
from . import slide_channel
from . import slide_channel_tag
from . import slide_slide_resource
from . import res_config_settings
from . import website
from . import res_users
from . import res_groups
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_slide_slide_public,slide.slide.all,model_slide_slide,base.group_public,1,0,0,0
access_slide_slide_portal,slide.slide.all,model_slide_slide,base.group_portal,1,0,0,0
access_slide_slide_employee,slide.slide.all,model_slide_slide,base.group_user,1,0,0,0
access_slide_slide_officer,slide.slide.officer,model_slide_slide,website_slides.group_website_slides_officer,1,1,1,0
access_slide_slide_manager,slide.slide.manager,model_slide_slide,website_slides.group_website_slides_manager,1,1,1,1
access_slide_slide_partner_all,slide.slide.partner.all,model_slide_slide_partner,,0,0,0,0
access_slide_slide_partner_system,slide.slide.partner.system,model_slide_slide_partner,website_slides.group_website_slides_officer,1,1,1,1
access_slide_question_public,slide.question.all,model_slide_question,base.group_public,1,0,0,0
access_slide_question_portal,slide.question.all,model_slide_question,base.group_portal,1,0,0,0
access_slide_question_employee,slide.question.all,model_slide_question,base.group_user,1,0,0,0
access_slide_question_officer,slide.question.officer,model_slide_question,website_slides.group_website_slides_officer,1,1,1,1
access_slide_answer_all,slide.answer.all,model_slide_answer,,0,0,0,0
access_slide_answer_officer,slide.answer.officer,model_slide_answer,website_slides.group_website_slides_officer,1,1,1,1
access_slide_tag_public,slide.tag.all,model_slide_tag,base.group_public,1,0,0,0
access_slide_tag_portal,slide.tag.all,model_slide_tag,base.group_portal,1,0,0,0
access_slide_tag_employee,slide.tag.all,model_slide_tag,base.group_user,1,0,0,0
access_slide_tag_officer,slide.tag.officer,model_slide_tag,website_slides.group_website_slides_officer,1,1,1,1
access_slide_channel_tag_public,slide.channel.tag.all,model_slide_channel_tag,base.group_public,1,0,0,0
access_slide_channel_tag_portal,slide.channel.tag.all,model_slide_channel_tag,base.group_portal,1,0,0,0
access_slide_channel_tag_employee,slide.channel.tag.all,model_slide_channel_tag,base.group_user,1,0,0,0
access_slide_channel_tag_user,slide.channel.tag.user,model_slide_channel_tag,website_slides.group_website_slides_officer,1,1,1,1
access_slide_channel_tag_group_public,slide.channel.tag.group.all,model_slide_channel_tag_group,base.group_public,1,0,0,0
access_slide_channel_tag_group_portal,slide.channel.tag.group.all,model_slide_channel_tag_group,base.group_portal,1,0,0,0
access_slide_channel_tag_group_employee,slide.channel.tag.group.all,model_slide_channel_tag_group,base.group_user,1,0,0,0
access_slide_channel_tag_group_user,slide.channel.tag.group.user,model_slide_channel_tag_group,website_slides.group_website_slides_officer,1,1,1,1
access_slide_channel_public,slide.channel.all,model_slide_channel,base.group_public,1,0,0,0
access_slide_channel_portal,slide.channel.all,model_slide_channel,base.group_portal,1,0,0,0
access_slide_channel_employee,slide.channel.all,model_slide_channel,base.group_user,1,0,0,0
access_slide_channel_officer,slide.channel.officer,model_slide_channel,website_slides.group_website_slides_officer,1,1,1,0
access_slide_channel_manager,slide.channel.manager,model_slide_channel,website_slides.group_website_slides_manager,1,1,1,1
access_slide_channel_partners_all,slide.channel.users.all,model_slide_channel_partner,,0,0,0,0
access_slide_channel_partners_system,slide.channel.users.system,model_slide_channel_partner,website_slides.group_website_slides_officer,1,1,1,1
access_slide_embed_public,slide.embed.all,model_slide_embed,base.group_public,1,0,0,0
access_slide_embed_portal,slide.embed.all,model_slide_embed,base.group_portal,1,0,0,0
access_slide_embed_user,slide.embed.user,model_slide_embed,base.group_user,1,1,1,1
access_slide_slide_resource_all,slide.slide.resource.all,model_slide_slide_resource,,0,0,0,0
access_slide_slide_resource_public,slide.slide.resource.public,model_slide_slide_resource,base.group_public,0,0,0,0
access_slide_slide_resource_portal,slide.slide.resource.portal,model_slide_slide_resource,base.group_portal,1,0,0,0
access_slide_slide_resource_internal,slide.slide.resource.internal,model_slide_slide_resource,base.group_user,1,0,0,0
access_slide_slide_resource_publisher,slide.slide.resource.publisher,model_slide_slide_resource,website_slides.group_website_slides_officer,1,1,1,1
access_slide_channel_invite,access.slide.channel.invite,model_slide_channel_invite,base.group_user,1,1,1,0

```

## File: security\website_slides_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record model="ir.module.category" id="base.module_category_website_elearning">
            <field name="sequence">21</field>
        </record>

        <record id="group_website_slides_officer" model="res.groups">
            <field name="name">Officer</field>
            <field name="category_id" ref="base.module_category_website_elearning"/>
            <field name="implied_ids" eval="[(4, ref('website.group_website_restricted_editor'))]"/>
        </record>

        <record id="group_website_slides_manager" model="res.groups">
            <field name="name">Manager</field>
            <field name="category_id" ref="base.module_category_website_elearning"/>
            <field name="implied_ids" eval="[(4, ref('group_website_slides_officer'))]"/>
            <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        </record>

        <record id="base.default_user" model="res.users">
            <field name="groups_id" eval="[(4,ref('group_website_slides_manager'))]"/>
        </record>

        <data noupdate="1">
        <!-- CHANNEL -->
        <record id="rule_slide_channel_global" model="ir.rule">
            <field name="name">Channel: always visible (sub rules exist)</field>
            <field name="model_id" ref="model_slide_channel"/>
            <field name="domain_force">[(1, '=', 1)]</field>
        </record>

        <record id="rule_slide_channel_visibility_public_user" model="ir.rule">
            <field name="name">Channel: public: restricted to public and published</field>
            <field name="model_id" ref="model_slide_channel"/>
            <field name="groups" eval="[(4, ref('base.group_public'))]"/>
            <field name="domain_force">[('website_published', '=', True), ('visibility', '=', 'public')]</field>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>

        <record id="rule_slide_channel_visibility_signed_in_user" model="ir.rule">
            <field name="name">Channel: portal/user: restricted to published, public or (invited) attendee, connected user</field>
            <field name="model_id" ref="model_slide_channel"/>
            <field name="groups" eval="[(4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
            <field name="domain_force">[
                '&amp;',
                    ('website_published', '=', True),
                    '|',
                        ('visibility', 'in', ('public', 'connected')),
                        '|',
                            ('is_member_invited', '=', True),
                            ('is_member', '=', True),
                ]
            </field>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>

        <record id="rule_slide_channel_officer_r" model="ir.rule">
            <field name="name">Channel: officer: read all</field>
            <field name="model_id" ref="model_slide_channel"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_website_slides_officer'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>
        <record id="rule_slide_channel_officer_cw" model="ir.rule">
            <field name="name">Channel: officer: create/write own only</field>
            <field name="model_id" ref="model_slide_channel"/>
            <field name="domain_force">[('user_id', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('group_website_slides_officer'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="0"/>
            <field name="perm_create" eval="1"/>
        </record>

        <record id="rule_slide_channel_manager" model="ir.rule">
            <field name="name">Channel: manager: crud all</field>
            <field name="model_id" ref="model_slide_channel"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_website_slides_manager'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>

        <record id="rule_slide_channel_tag_public" model="ir.rule">
            <field name="name">Channel Tag: public/portal: color = published</field>
            <field name="model_id" ref="model_slide_channel_tag"/>
            <field name="domain_force">['&amp;', ('color', '!=', False), ('color', '!=', 0)]</field>
            <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="False"/>
            <field name="perm_create" eval="False"/>
            <field name="perm_unlink" eval="False"/>
        </record>

        <!-- SLIDE -->
        <record id="rule_slide_slide_global" model="ir.rule">
            <field name="name">Slide: always visible (sub rules exist)</field>
            <field name="model_id" ref="model_slide_slide"/>
            <field name="domain_force">[(1, '=', 1)]</field>
        </record>

        <record id="rule_slide_slide_public_user" model="ir.rule">
            <field name="name">Slide: public: restricted to published or public channel &amp; (category or previewable)</field>
            <field name="model_id" ref="model_slide_slide"/>
            <field name="groups" eval="[(4, ref('base.group_public'))]"/>
            <field name="domain_force">[
                    ('channel_id.website_published', '=', True),
                    ('website_published', '=', True),
                    ('channel_id.visibility', '=', 'public'),
                    '|',
                        ('is_category','=', True),
                        ('is_preview', '=', True),
                ]
            </field>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>

        <record id="rule_slide_slide_signed_in_user" model="ir.rule">
            <field name="name">Slide: portal/user: restricted to published and connected user, (invited) attendee if course visible to attendees only</field>
            <field name="model_id" ref="model_slide_slide"/>
            <field name="groups" eval="[(4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
            <field name="domain_force">[
                '&amp;',
                    '|',
                        ('user_id', '=', user.id),
                        '&amp;',
                            ('website_published', '=', True),
                            ('channel_id.website_published', '=', True),
                    '|',
                        '&amp;',
                            '|',
                                ('channel_id.visibility', 'in', ('public','connected')),
                                ('channel_id.is_member_invited', '=', True),
                            '|',
                                ('is_category', '=', True),
                                ('is_preview', '=', True),
                        ('channel_id.is_member', '=', True),
                ]
            </field>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>

        <record id="rule_slide_slide_officer_r" model="ir.rule">
            <field name="name">Slide: officer: read all</field>
            <field name="model_id" ref="model_slide_slide"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_website_slides_officer'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>

        <record id="rule_slide_slide_officer_cw" model="ir.rule">
            <field name="name">Slide: officer: create/write own only</field>
            <field name="model_id" ref="model_slide_slide"/>
            <field name="domain_force">[('channel_id.user_id', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('group_website_slides_officer'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="0"/>
            <field name="perm_create" eval="1"/>
        </record>

        <record id="rule_slide_slide_manager" model="ir.rule">
            <field name="name">Slide: manager: crud all</field>
            <field name="model_id" ref="model_slide_slide"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_website_slides_manager'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>

        <!-- CHANNEL PARTNER -->
        <record id="rule_slide_channel_partner_officer" model="ir.rule">
            <field name="name">Channel Partner: officer: create/write/unlink own only</field>
            <field name="model_id" ref="model_slide_channel_partner"/>
            <field name="domain_force">[('channel_id.user_id', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('group_website_slides_officer'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="0"/>
            <field name="perm_create" eval="1"/>
        </record>

        <record id="rule_slide_channel_partner_manager" model="ir.rule">
            <field name="name">Channel Partner: manager: crud all</field>
            <field name="model_id" ref="model_slide_channel_partner"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_website_slides_manager'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>

        <!-- SLIDE PARTNER -->
        <record id="rule_slide_slide_partner_officer" model="ir.rule">
            <field name="name">Slide Partner: officer: create/write/unlink own only</field>
            <field name="model_id" ref="model_slide_slide_partner"/>
            <field name="domain_force">[('channel_id.user_id', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('group_website_slides_officer'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="0"/>
            <field name="perm_create" eval="1"/>
        </record>

        <record id="rule_slide_slide_partner_manager" model="ir.rule">
            <field name="name">Slide Partner: manager: crud all</field>
            <field name="model_id" ref="model_slide_slide_partner"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_website_slides_manager'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>

        <!--SLIDE RESOURCE-->
        <record id="rule_slide_slide_resource_downloadable" model="ir.rule">
            <field name="name">Resource: read restricted to channel members and channel responsible</field>
            <field name="model_id" ref="model_slide_slide_resource"/>
            <field name="domain_force">[('slide_id.channel_id.is_member', '=', True)]</field>
            <field name="groups" eval="[(4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="False"/>
            <field name="perm_create" eval="False"/>
            <field name="perm_unlink" eval="False"/>
        </record>

        <record id="rule_slide_slide_resource_officer_read" model="ir.rule">
            <field name="name">Resource: officer: read all</field>
            <field name="model_id" ref="model_slide_slide_resource"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_website_slides_officer'))]"/>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="False"/>
            <field name="perm_create" eval="False"/>
            <field name="perm_unlink" eval="False"/>
        </record>

        <record id="rule_slide_slide_resource_officer_crud" model="ir.rule">
            <field name="name">Resource: officer: crud own only</field>
            <field name="model_id" ref="model_slide_slide_resource"/>
            <field name="domain_force">[('slide_id.channel_id.user_id', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('group_website_slides_officer'))]"/>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="True"/>
            <field name="perm_create" eval="True"/>
            <field name="perm_unlink" eval="True"/>
        </record>

        <record id="rule_slide_slide_resource_downloadable_manager" model="ir.rule">
            <field name="name">Resource: manager: crud all</field>
            <field name="model_id" ref="model_slide_slide_resource"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_website_slides_manager'))]"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_create" eval="1"/>
            <field name="perm_unlink" eval="1"/>
        </record>
    </data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M41.413 29.673c0 9.018-7.348 16.328-16.412 16.328-9.065 0-16.413-7.31-16.413-16.328 0-9.017 7.348-16.327 16.413-16.327 9.064 0 16.412 7.31 16.412 16.327Z" fill="#1AD3BB"/><path d="M22.637 5.551a5.337 5.337 0 0 1 4.726 0L50 16.921 27.363 28.288a5.337 5.337 0 0 1-4.726 0L0 16.92 22.637 5.551Z" fill="#985184"/><path d="m39.574 22.156-12.211 6.132a5.337 5.337 0 0 1-4.725 0l-12.212-6.133A16.421 16.421 0 0 1 25 13.344a16.42 16.42 0 0 1 14.574 8.812Z" fill="#005E7A"/></svg>

```

## File: static\lib\pdfslidesviewer\PDFSlidesViewer.js

```javascript
/**
    Homemade helper for browsing PDF document from page to page.
    This is hightly inspired from https://github.com/mozilla/pdf.js/blob/master/examples/learning/prevnext.html
    This lib requires PDF JS. It simply uses PDFjs and its promises.
    DOC : http://mozilla.github.io/pdf.js/api/draft/api.js.html
*/

// !!!!!!!!! use globalThis.pdfjsLib and not pdfjsLib

globalThis.PDFSlidesViewer = (function(){
    function PDFSlidesViewer(pdf_url, $canvas) {
        // pdf variables
        this.pdf = null;
        this.pdf_url = pdf_url;
        this.pdf_page_total = 0;
        this.pdf_page_current = 1; // default is the first page
        this.pdf_zoom = 1; // 1 = scale to fit to available space
        // promise business
        this.pageRendering = false;
        this.pageNumPending = null;
        //canvas
        this.canvas = $canvas;
        this.canvas_context = $canvas.getContext('2d');
    }

    /**
     * Load the PDF document
     */
    PDFSlidesViewer.prototype.loadDocument = async function() {
        const file_content = await globalThis.pdfjsLib.getDocument(this.pdf_url).promise;
        this.pdf = file_content;
        this.pdf_page_total = file_content.numPages;
        return file_content;
    };

    /**
     * Get page info from document, resize canvas accordingly, and render page.
     * @param page_number : Page number.
     */
    PDFSlidesViewer.prototype.renderPage = function(page_number) {
        var self = this;
        this.pageRendering = true;
        return this.pdf.getPage(page_number).then(function(page) {
            // Each PDF page has its own viewport which defines the size in pixels and initial rotation.
            // We provide the scale at which to render it (relative to the natural size of the document)
            var scale = self.getScaleToFit(page) * self.pdf_zoom;
            var viewport = page.getViewport({ scale: scale });
            // important to match, otherwise the browser will scale the rendered output and it will be ugly
            self.canvas.height = viewport.height;
            self.canvas.width = viewport.width;
            // Render PDF page into canvas context
            var renderContext = {
                canvasContext: self.canvas_context,
                viewport: viewport
            };
            var renderTask = page.render(renderContext);
            // Wait for rendering to finish
            return renderTask.promise.then(function () {
                self.pageRendering = false;
                if (self.pdf_zoom === 1 && scale > self.getScaleToFit(page)) {
                    // if the scale has changed (because we just added scrollbars) and we no longer fit the space
                    return self.renderPage(page_number);
                }
                if (self.pageNumPending !== null) {
                    // New page rendering is pending
                    self.renderPage(self.pageNumPending);
                    self.pageNumPending = null;
                }
                self.pdf_page_current = page_number;
                return page_number;
            });
        });
    };

    /**
     * If another page rendering in progress, waits until the rendering is
     * finised. Otherwise, executes rendering immediately.
     */
    PDFSlidesViewer.prototype.queueRenderPage = function(num) {
        if(this.pageRendering) {
            this.pageNumPending = num; // the queue is only the last elem
            return Promise.resolve(num);
        } else {
            return this.renderPage(num);
        }
    }

    /**
     * Displays previous page.
     */
    PDFSlidesViewer.prototype.previousPage = function() {
        if (this.pdf_page_current <= 1) {
          return Promise.resolve(false);
        }
        this.pdf_page_current--;
        return this.queueRenderPage(this.pdf_page_current);
    };

    /**
     * Displays next page.
     */
    PDFSlidesViewer.prototype.nextPage = function() {
        if (this.pdf_page_current >= this.pdf_page_total) {
            return Promise.resolve(false);
        }
        this.pdf_page_current++;
        return this.queueRenderPage(this.pdf_page_current);
    };

    /*
     * Calculate a scale to fit the document on the available space.
     */
    PDFSlidesViewer.prototype.getScaleToFit = function(page) {
        var maxWidth = this.canvas.parentNode.clientWidth;
        var maxHeight = this.canvas.parentNode.clientHeight;
        var hScale = maxWidth / page.view[2];
        var vScale = maxHeight / page.view[3];
        return Math.min(hScale, vScale);
    };

    /**
     * Displays the given page.
     */
    PDFSlidesViewer.prototype.changePage = function(num){
        if(1 <= num <= this.pdf_page_total){
            this.pdf_page_current = num;
            return this.queueRenderPage(num);
        }
        return Promise.resolve(false);
    }

    /**
     * Displays first page.
     */
    PDFSlidesViewer.prototype.firstPage = function(){
        this.pdf_page_current = 1;
        return this.queueRenderPage(1);
    }

    /**
     * Displays last page.
     */
    PDFSlidesViewer.prototype.lastPage = function(){
        this.pdf_page_current = this.pdf_page_total;
        return this.queueRenderPage(this.pdf_page_total);
    }

    PDFSlidesViewer.prototype.toggleFullScreenFooter = function(){
        if(document.fullscreenElement || document.mozFullScreenElement || document.webkitFullscreenElement || document.msFullscreenElement) {
            var $navBarFooter = $('div#PDFViewer div.oe_slides_panel_footer').parent();
            $navBarFooter.toggleClass('oe_show_footer');
            $navBarFooter.toggle();
        }
    }

    PDFSlidesViewer.prototype.toggleFullScreen = function(){
        // The canvas and the navigation bar needs to be fullscreened
        var el = this.canvas.parentNode.parentNode;

        var isFullscreenAvailable = document.fullscreenEnabled || document.mozFullScreenEnabled || document.webkitFullscreenEnabled || document.msFullscreenEnabled || false;
        if(isFullscreenAvailable){ // Full screen supported
            // get the actual element in FullScreen mode (Null if no element)
            var fullscreenElement = document.fullscreenElement || document.mozFullScreenElement || document.webkitFullscreenElement || document.msFullscreenElement;

            if (fullscreenElement) { // Exit the full screen mode
                if (document.exitFullscreen) {
                    // W3C standard
                    document.exitFullscreen();
                } else if (document.mozCancelFullScreen) {
                    // Firefox 10+, Firefox for Android
                    document.mozCancelFullScreen();
                } else if (document.webkitExitFullscreen) {
                    // Chrome 20+, Safari 6+, Opera 15+, Chrome for Android, Opera Mobile 16+
                    document.webkitExitFullscreen();
                } else if (document.webkitCancelFullScreen) {
                    // Chrome 15+, Safari 5.1+
                    document.webkitCancelFullScreen();
                } else if (document.msExitFullscreen) {
                    // IE 11+
                    document.msExitFullscreen();
                }
            }else { // Request to put the 'el' element in FullScreen mode
                if (el.requestFullscreen) {
                    // W3C standard
                    el.requestFullscreen();
                } else if (el.mozRequestFullScreen) {
                    // Firefox 10+, Firefox for Android
                    el.mozRequestFullScreen();
                } else if (el.msRequestFullscreen) {
                    // IE 11+
                    el.msRequestFullscreen();
                } else if (el.webkitRequestFullscreen) {
                    if (navigator.userAgent.indexOf('Safari') != -1 && navigator.userAgent.indexOf('Chrome') == -1) {
                        // Safari 6+
                        el.webkitRequestFullscreen();
                    } else {
                        // Chrome 20+, Opera 15+, Chrome for Android, Opera Mobile 16+
                        el.webkitRequestFullscreen(Element.ALLOW_KEYBOARD_INPUT);
                    }
                } else if (el.webkitRequestFullScreen) {
                    if (navigator.userAgent.indexOf('Safari') != -1 && navigator.userAgent.indexOf('Chrome') == -1) {
                        // Safari 5.1+
                        el.webkitRequestFullScreen();
                    } else {
                        // Chrome 15+
                        el.webkitRequestFullScreen(Element.ALLOW_KEYBOARD_INPUT);
                    }
                }
            }
        }else{
            // Full screen not supported by the browser
            console.error("ERROR : full screen not supported by web browser");
        }
    }
    return PDFSlidesViewer;
})();

```

## File: static\src\slide_category_list_renderer.js

```javascript
/** @odoo-module */

import { makeContext } from "@web/core/context";
import { ListRenderer } from "@web/views/list/list_renderer";
import { useEffect } from "@odoo/owl";

export class SlideCategoryListRenderer extends ListRenderer {
    setup() {
        super.setup();

        this.discriminant = "is_category";
        this.titleField = "name";

        useEffect(
            (table) => {
                if (table) {
                    table.classList.add("o_section_list_view");
                }
            },
            () => [this.tableRef.el]
        );
    }

    add(params) {
        let editable = false;
        if (params.context && !this.env.isSmall) {
            const evaluatedContext = makeContext([params.context]);
            if (evaluatedContext[`default_${this.discriminant}`]) {
                editable = this.props.editable;
            }
        }
        super.add({ ...params, editable });
    }

    getColumns(record) {
        const columns = super.getColumns(record);
        if (this.isSection(record)) {
            return this.getSectionColumns(columns);
        }
        return columns;
    }

    getRowClass(record) {
        const classNames = super.getRowClass(record).split(" ");
        if (this.isSection(record)) {
            classNames.push(`o_is_section`, `fw-bold`);
        }
        return classNames.join(" ");
    }

    getSectionColumns(columns) {
        const sectionColumns = columns.filter((col) => col.widget === "handle");
        const colspan = columns.length - sectionColumns.length;
        const titleCol = columns.find(
            (col) => col.type === "field" && col.name === this.titleField
        );
        sectionColumns.push({ ...titleCol, colspan });
        return sectionColumns;
    }

    isInlineEditable(record) {
        return this.isSection(record) && this.props.editable;
    }

    isSection(record) {
        return record.data[this.discriminant];
    }

    onCellKeydownEditMode(hotkey) {
        switch (hotkey) {
            case "enter":
            case "tab":
            case "shift+tab": {
                this.props.list.leaveEditMode();
                return true;
            }
        }
        return super.onCellKeydownEditMode(...arguments);
    }
}

```

## File: static\src\slide_category_one2many_field.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { SlideCategoryListRenderer } from "./slide_category_list_renderer";
import { X2ManyField, x2ManyField } from "@web/views/fields/x2many/x2many_field";

class SlideCategoryOneToManyField extends X2ManyField {
    static components = {
        ...X2ManyField.components,
        ListRenderer: SlideCategoryListRenderer,
    };
    static defaultProps = {
        ...X2ManyField.defaultProps,
        editable: "bottom",
    };
    setup() {
        super.setup();
        this.canOpenRecord = true;
    }
}

registry.category("fields").add("slide_category_one2many", {
    ...x2ManyField,
    component: SlideCategoryOneToManyField,
    additionalClasses: [...x2ManyField.additionalClasses || [], "o_field_one2many"],
});

```

## File: static\src\activity\activity_patch.js

```javascript
/** @odoo-module **/

import { Activity } from "@mail/core/web/activity";

import { patch } from "@web/core/utils/patch";

/** @type {import("@mail/core/web/activity").Activity } */
const ActivityPatch = {
    async onGrantAccess() {
        await this.env.services.orm.call(
            "slide.channel",
            "action_grant_access",
            [[this.props.activity.res_id]],
            { partner_id: this.props.activity.request_partner_id }
        );
        this.props.activity.remove();
        this.props.reloadParentView();
    },
    async onRefuseAccess() {
        await this.env.services.orm.call(
            "slide.channel",
            "action_refuse_access",
            [[this.props.activity.res_id]],
            { partner_id: this.props.activity.request_partner_id }
        );
        this.props.activity.remove();
        this.props.reloadParentView();
    },
};

patch(Activity.prototype, ActivityPatch);

```

## File: static\src\activity\activity_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-inherit="mail.Activity" t-inherit-mode="extension">
        <xpath expr="//t[@name='tools']" position="replace">
            <t t-if="props.activity.request_partner_id and props.activity.res_model === 'slide.channel'">
                <button class="btn btn-link" t-on-click="onGrantAccess">
                    <i class="fa fa-check"/> Grant Access
                </button>
                <button class="btn btn-link" t-on-click="onRefuseAccess">
                    <i class="fa fa-times"/> Refuse Access
                </button>
            </t>
            <t t-else="">$0</t>
        </xpath>
    </t>
</templates>

```

## File: static\src\img\banner_default.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="1280" height="515"><defs><path id="a" d="M0 0L1280 0 1280 515 0 515z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><use fill="#845978" xlink:href="#a"/><g mask="url(#b)"><g transform="translate(-64 -902)"><path fill="#FFF" opacity=".105" d="M2443 1200.003L1425.898 1227.592 2428.994 1053.537 2410.807 959.354 1413.838 1188.559 2367.991 814.755 2332.04 726.066 1402.709 1149.229 2264.239 596.223 2212.162 516.325 1376.929 1117.888 2118.458 399.473 2052.002 331.594 1351.903 1085.917 1940.74 238.14 1862.693 184.686 1316.365 1067.012 1732.352 113.368 1645.54 76.631 1281.269 1047.257 1507.742 33.798 1415.612 14.927 1241.359 1043.705 1268.482 0 1174.518 0 1201.53 1039.145 1031.165 14.307 938.981 32.888 1163.316 1051.484 797.46 76.631 710.65 113.368 1124.834 1062.875 583.567 182.638 505.363 235.852 1094.152 1089.198 390.988 331.587 324.544 399.471 1062.838 1114.751 233.088 513.158 180.759 592.892 1044.335 1151.069 110.958 726.063 75.002 814.755 1025.021 1186.927 33.079 955.542 14.605 1049.674 1021.586 1227.721 0 1200.003 0 1295.992 1017.114 1268.403 14.001 1442.465 32.191 1536.639 1029.174 1307.438 75.002 1681.24 110.958 1769.932 1040.3 1346.768 178.761 1899.772 230.838 1979.679 1066.076 1378.114 324.544 2096.524 390.983 2164.406 1091.094 1410.09 502.255 2257.855 580.3 2311.319 1126.649 1428.964 710.641 2382.632 797.453 2419.374 1161.731 1448.736 935.256 2462.199 1027.383 2481.071 1201.646 1452.3 1174.518 2496 1268.482 2496 1241.475 1456.834 1411.837 2481.691 1504.019 2463.109 1279.686 1444.516 1645.54 2419.374 1732.352 2382.632 1318.176 1433.141 1859.435 2313.36 1937.649 2260.148 1348.874 1406.818 2052.009 2164.406 2118.458 2096.524 1380.157 1381.246 2209.92 1982.84 2262.241 1903.103 1398.639 1344.912 2332.047 1769.932 2367.991 1681.24 1417.991 1309.061 2409.923 1540.453 2428.386 1446.317 1421.502 1268.286 2443 1295.992 2443 1200.003"/><path fill="#845978" d="M64 1417L1344 1417 1344 1248 64 1248z"/><g transform="translate(892 1246)" fill="#573F51"><path d="M354.867 14.066L362.3 28 86.043 91.596 84.19 74.089z"/><path d="M248.749 6.147L253.38 18.931 28.168 71.1 30.604 58.435z"/><path d="M294.541 7.005L311.054 0 324.033 8.712 58.485 79.222 52.043 70.679z"/><path d="M172.008 16.138L209.092 6.884 213.083 12.127 0.734 61.158 0.891 56.487z"/></g><path fill="#573F51" d="M1111.493 1204.155c-.021.535-.06 1.102-.117 1.69-.356 3.73-1.408 8.343-3.014 11.624-2.639 5.398-4.017 11.736-3.912 16.782.105 5.044.703 9.657 1.787 13.639 1.082 3.979 2.8 6.916.164 9.283-1.315 1.183-2.172 1.28-3.133 1.056-.962-.225-2.029-.772-3.763-.88-3.468-.212-3.412.795-6.057 2.963-2.643 2.166-7.56 1.829-11.07.81-3.514-1.02-2.025-2.47-1.05-3.375 1.15-1.077 8.05-5.494 9.373-6.577 1.324-1.084 2.84-1.774 4.922-4.717 1.52-2.144 1.396-9.167 1.696-9.991.3-.824.118-26.067.225-27.286.004-.05.008-.118.011-.197.087-1.89.015-11.867-.166-15.148-.189-3.424-1.874-21.513-1.859-24.342.016-2.49-.223-10.77-.078-12.715.449-5.982 1.684-12.607 2.079-14.625-4.06-5.473-20.5-23.557-17.062-29.655 1.181-2.097 4.16-6.867 10.993-14.678 1.111-1.27 5.622-8.67 6.564-10.051 3.358-4.921 6.404-7.743 9.178-8.714.107-.066.203-.118.287-.154.1-.043.197-.088.292-.135.95-.765 2.077-1.477 2.687-1.967.56-.45.956-1.005 1.227-1.616a27.455 27.455 0 0 1-.397-.353c-3.066-1.651-6.327-5.703-7.704-11.32-1.135-4.632-1.509-9.206-.696-12.845.076-.969.141-1.94.123-2.693-.153-.132-.3-.27-.44-.414a4.451 4.451 0 0 1-1.253-2.742c-.086-1.08.41-1.714-.018-2.698-.936-2.12-.973-2.25.426-4.104 3.676-4.872 11.968-6.568 17.706-5.853 7.174.893 14.118 3.489 20.54 6.725 3.652 1.84 7.251 3.79 10.968 5.497 3.375 1.55 6.992 3.113 10.735 3.438 4.154.362 8.352-.28 12.49.392 1.773.287 4.26.443 5.61 1.78.247.247 2.205 3.21 1.81 3.409.094-.23-.967-1.403-1.137-1.597-.788-.9-1.703-1.352-2.893-1.395-2.524-.103-5.085.247-7.543.8.274.097 5.012 2.04 4.89 2.328-.026-.35-4.149-.697-4.446-.721a26.884 26.884 0 0 0-7.93.544c2.858.066 5.61.68 8.367 1.354-1.572.144-3.148.282-4.71.535-2.477.392-4.769 1.304-7.147 2.055 1.694.207 3.297-.054 4.943-.397 2.87-.585 5.825-.892 8.754-.665 2.58.206 5.575.828 7.278 2.95.675.842 1.216 1.858 1.36 2.94.042.318-.558 2.82-.905 2.474.293.189.435-1.687.407-1.808-.198-.881-.796-1.653-1.454-2.248-1.619-1.46-3.95-1.475-6.002-1.233-3.993.473-7.308 2.489-11.08 3.576a17.61 17.61 0 0 1-1.43.348c.369.618.762 1.225 1.192 1.814 3.905 5.34 10.308 2.235 15.31.52 5.26-1.803 12.645-2.675 17.262.952 1.685 1.324 2.916 3.496 3.463 5.545.46 1.726.58 3.588.226 5.343-.363 1.792-1.099 2.206-.193 3.969 1.318 2.584 3.388 4.936 5.84 6.492 1.452.92 3.156 1.594 4.903 1.485.444-.025 4.764-1.718 4.345-2.093.185.127-2.114 2.112-2.303 2.225-1.719 1.036-3.787 1.18-5.717.757-3.102-.674-6.367-2.076-9.164-3.541 1.545 2.146 2.88 4.442 4.263 6.694-1.37-1.242-2.724-2.503-4.113-3.723-1.867-1.632-3.826-3.274-6.185-4.136-2.203.983-4.22 2.36-6.147 3.799 3.977-.252 8.627.185 11.438 3.366 1.346 1.499 2.244 3.439 2.053 5.485-.036.397-1.543 4.917-1.985 4.59 1.155.42.785-5.933.5-6.49-1.256-2.448-3.925-3.62-6.564-3.715-4.79-.176-9.434 1.916-13.8 3.646-6.352 2.513-12.744 3.538-18.602 1.868 3.06 6.546 5.709 13.409 11.322 18.436 8.568 7.246 20.031 9.339 30.543 10.576 13.703 1.516 26.903 2.15 37.383 11.108 7.138 5.962 19.081 8.482 28.586 7.96-12.672.697-17.935 12.337-26.638 19.17-8.704 6.835-20.646 4.315-31.13 3.53-13.225-1.088-15.62 13.118-20.58 22.017-4.578 7.515-14.337 11.684-22.559 10.776-7.242-1.04-15.38-7.999-23.157-10.374.444 2.193.875 4.066.977 4.355.174.5 2.238 4.612 4.258 9.212l.151-.157.121.274c.48 1.099.92 2.14 1.308 3.094 1.502 3.69 2.926 9.864 1.196 18.319a212.782 212.782 0 0 1-.93 4.289c-1.299 5.803-2.16 9.635-1.35 17.602.223 2.225.77 4.009 1.255 5.586.917 2.99 1.712 5.576-.135 9.723-2.092 4.689-2.836 6.772-5.12 6.898-.625.032-1.266-.082-1.962-.356-2.96-1.16-4.164-8.26-2.866-12.488.598-1.944 1.118-3.414 1.575-4.71.498-1.413.892-2.528 1.21-3.745.581-2.25-2.86-21.918-3.369-24.276-.162-.752-.338-2.483-.558-4.675-.236-2.334-.528-5.24-.895-7.723l-.104-.697.247.29c-.31-2.02-.674-3.757-1.11-4.73-1.467-3.28-2.739-3-6.226-12.887-.306-.868-.602-1.722-.89-2.562-5.065 1.12-9.121 2.356-12.317 3.4-1.002.327-2.797 1.098-4.632 1.72.236 1.513.475 2.763.686 3.499.238.828.321 2.097.262 3.61z"/><path fill="#573F51" d="M1257.956 1185.867l.35-.214.211.39c.262.487.546.89.8 1.245.206.29.399.563.555.844l.061.109c4.155 7.467 5.06 15.14 5.086 20.264.037 7.473-2.373 17.109-3.969 23.485l-.231.919c.202 1.712.469 4.139.776 7.794.258 3.046.818 5.549 1.314 7.759 1.109 4.94 1.986 8.84-1.051 15.342-3.33 7.135-5.332 10.32-8.382 10.32h-.002c-.76 0-1.582-.191-2.584-.607-4.362-1.797-6.106-12.685-4.048-18.994 1.303-3.988 1.936-6.68 2.496-9.053.288-1.222.561-2.376.895-3.579.397-1.433.012-5.182-1.146-11.138-2.149-6.859-4.144-17.743-5.215-23.593-.146-.795-.275-1.5-.386-2.09-.369-1.975-.591-3.935-.808-5.828-.284-2.498-.554-4.857-1.118-6.696l-.311-1.014.518.295a8.435 8.435 0 0 0-.539-1.199c-1.739-3.133-2.525-1.848-3.936-6.913-1.073-3.85-2.386-7.25-4.824-11.594-.45 1.492-.972 2.944-1.467 4.45-.956 2.897-.981 5.362.018 8.193.372 1.055.661 2.155.879 3.283l.396-.17.092.522c.666 3.78.638 8.161-.082 13.025-1.258 8.369-5.06 15.851-9.181 23.415-.456.834-.956 1.661-1.483 2.538-2.069 3.434-4.209 6.985-3.53 10.831.24 1.357.664 2.715 1.117 4.154.708 2.268 1.443 4.614 1.383 6.941-.068 2.557-2.373 5.529-5.246 5.529-.293 0-.586-.031-.875-.094a32.806 32.806 0 0 1-1.978-.539c-1.333-.391-2.711-.799-4.04-.799-.814 0-1.54.15-2.22.461-.771.349-1.437.971-2.081 1.573-.201.191-.401.376-.606.556-1.635 1.443-3.043 2.228-4.708 2.621a15.96 15.96 0 0 1-3.687.409c-2.217 0-4.606-.39-7.102-1.161-.991-.309-3.054-.945-3.027-2.516.017-1.011.875-1.777 1.501-2.338l.329-.292c1.471-1.282 3.194-2.342 4.865-3.369.629-.389 1.286-.792 1.901-1.193 1.976-1.282 4.056-2.706 6.548-4.474 2.504-1.779 4.154-3.616 5.045-5.621.362-.812.772-1.619 1.167-2.398.774-1.529 1.576-3.109 2.029-4.762 1.025-3.75 1.81-15.59 1.818-15.697.195-5.522.458-11.423 1.046-17.151l.019-.185.15-.11c.1-.075.202-.149.305-.223.154-1.448.33-2.892.533-4.327.564-3.995 1.788-8.564-.075-12.392-1.48-3.041-1.218-6.035-1.48-9.355-.567-7.134-3.397-13.732-4.369-20.789a129.162 129.162 0 0 1-1.115-21.947c.093-2.66.242-5.345.652-7.965-.542-.86-1.107-1.598-1.697-2.116-2.767-2.43-16.811-11.084-19.283-15.707-2.477-4.623-.7-13.566.509-18.364 1.178-4.672 3.27-4.536 3.799-9.789.528-5.253 6.764-14.781 6.764-14.781s3.495-9.884 9.583-12.833c2.455-1.957 4.892-2.262 6.785-2.144.402-.1 4.41-1.094 8.661-2.19l-.045-.326c-2.84-.43-1.969-1.46-1.969-1.46s-.765.524-1.376.311c-1.565-.542-2.594-1.837-2.938-2.946-.345-1.108-.325-9.416-.89-10.669-.565-1.253-2.065-3.684-1.708-4.71.36-1.027 1.268-2.701 1.075-3.081-.194-.379-1.532-3.146-.991-4.745.196-.577.913-1.815 1.77-3.183-.175.187-.275.3-.275.3s.021-1.98 1.6-5.357c1.581-3.377 4.995-6.182 4.995-6.182l-.223 2.941s.534-2.55 3.965-3.455c3.43-.905 5.793.203 5.793.203l-1.948 1.433s3.144-1.532 6.542-1.134c3.399.396 6.636 3.149 6.636 3.149l-1.811.264s1.814 1.754 4.033 3.082c2.22 1.327 2.577 2.319 2.577 2.319s-1.825.38-2.071 2.489c-.247 2.109-.764 8.577-1.637 10.968-.663 1.813-1.795 3.57-2.304 4.313l.286.762c-1.129 2.61.276 4.743-.122 7.572a2.276 2.276 0 0 1-.34.877c13.797-2.203 30.236-1.437 38.839 1.84 16.726 6.432 26.376 23.159 45.675 23.802 19.301.644 38.601-12.223 58.543-7.076 5.79 1.287 9.649 4.504 13.511 8.363 5.146 4.503 10.291 9.006 16.084 12.223-12.225 4.504-21.873 2.573-23.805 18.657-1.285 13.509-1.93 26.375-17.369 29.592-13.511 3.217-27.663-3.86-39.887 4.504-7.72 5.146-7.077 14.797-11.58 21.872-4.502 9.651-10.936 16.083-21.871 14.797-12.226-1.287-21.873-9.65-34.098-9.007-2.497 0-4.996.607-7.493 1.819.9 8.704 1.49 16.902 1.077 20.069-.535 4.103-.024 6.368.676 7.846zm-47.861-77.259c1.04-3.92-.19-10.75-.803-14.726-.442-2.847-1.741-5.627-2.935-8.179-2.549 5.476-6.527 10.322-5.576 13.589 1.11 3.814 5.099 5.725 8.48 11.07.24-.634.691-1.929.691-1.929l.143.175z"/><g transform="translate(873 1139)"><path fill="#418ECA" d="M25.659 35.361c-3.172-2.336-6.91-.743-11.387 5.818-.707 1.039-4.095 6.596-4.929 7.549-5.133 5.867-7.37 9.449-8.257 11.024-2.707 4.801 10.995 19.498 13.189 22.801 2.192 3.301 1.362-6.213 1.362-6.213S7.703 63.739 7.749 63.136c.139-1.771 8.088-8.693 8.088-8.693s14.449-15.673 9.822-19.082"/><path fill="#1A1919" d="M12.857 51.647c-.474-1.979-.629-3.155.447-5.129.995-1.787 1.971-3.936 3.437-5.476 1.325-1.397 3.519-3.25 3.992-5.325 2.244-1.478 3.091-1.706 4.926-.356 4.627 3.409-9.822 19.082-9.822 19.082s-.308.237-1.477 1.316c-.536-1.429-1.226-2.819-1.503-4.112"/><path fill="#F1DCBC" d="M7.677 67.01c.265.114.596.29.919.272 0-.835-.057-1.567.078-2.329 2.082 3.634 6.963 11.387 6.963 11.387s.83 9.514-1.362 6.213c-1.503-2.263-8.412-9.878-11.739-15.992.605-.009 1.199-.003 1.761-.01 1.161.001 2.266.133 3.38.459"/><path fill="#2D2118" d="M67.078 66.508c-.148-.429-.288-.86-.423-1.295.256.374.301.868.423 1.295-.095-.271-.065-.227 0 0zm4.192-7.726c-.312 1.108-.841 2.211-1.687 3.014.877-3.611 1.518-6.704-.744-9.975 2.497 1.554 3.199 4.218 2.431 6.961-.235.835.232-.83 0 0zm-4.727 5.777c-1.107-1.913-1.362-4.535-2.186-6.615-.729-1.845-1.721-3.524-3.028-5.019 2.897.461 5.47 2.197 5.433 5.41-.026 2.201-.433 4.31-.067 6.509a19.071 19.071 0 0 0-.152-.285zM19.642 7.837c-5.382 8.241-4.964 16.814.79 24.746 1.901 4.55 4.693 8.136 8.709 11.016 2.418 1.72 2.927 4.571 3.602 7.322 1.965 8.011 8.398 11.746 16.182 13.188 3.469.641 7.258 1.163 10.237 3.186 1.639 1.113 2.871 2.924 2.693 4.983-.04.467-2.34 4.658-2.906 3.93.15.384 2.912-1.891 3.093-2.13.938-1.226 1.142-2.818.883-4.309-.52-3.145-3.305-5.278-5.936-6.704 1.801-.147 3.633-.22 5.427.033 1.158 1.491 1.749 3.317 2.285 5.1.399 1.331.757 2.674 1.133 4.01.018-1.985.084-3.978-.043-5.96 1.195 2.048 2.714 4.243 4.419 5.909 1.059 1.039 2.433 1.773 3.94 1.799.165.003 2.42-.342 2.354-.498.116.407-3.308-.239-3.6-.401-1.156-.627-1.972-1.736-2.529-2.901-.938-1.968-1.317-4.29-1.126-6.46.127-1.484.762-1.453 1.707-2.45.924-.975 1.591-2.208 1.986-3.489.47-1.523.555-3.395.009-4.91-1.49-4.15-6.537-6.541-10.605-7.492-3.865-.904-9.179-1.482-9.533-6.438-.265-3.698.89-7.276-.058-10.944-1.556-6.014-6.959-7.692-12.174-9.532-1.925-.677-3.754-1.549-5.235-2.977-1.689-1.636-1.027-5.038-1.737-7.147-.761-2.268-2.502-3.616-4.715-4.386-.508-.177-6.552-1.424-6.553-.103-.284.896-1.934 2.851-2.699 4.009-.848 1.298.856-1.295 0 0z"/><path fill="#F1DCBC" d="M31.302 24.242s.144 2.631 1.084 4.209c.939 1.576 5.973 4.308 5.973 4.308s-8.682 7.089-9.748 7.458c-1.065.368-3.41.033-4.292-.462-.88-.494-1.908-3.696-1.85-4.009.062-.315 1.136-1.925 1.472-2.874.336-.95.238-6.918.238-6.918l7.123-1.712"/><path fill="#C8A67A" d="M27.466 32.44c-1.221.265-2.352-.03-3.377-.598.155-1.998.09-5.888.09-5.888l7.123-1.712s.144 2.619 1.078 4.198c-1.212 1.75-2.845 3.554-4.914 4"/><path fill="#418ECA" d="M52.384 76.226c-.717-1.746-2.198-21.275-2.647-24.105-.475-3-1.764-5.256-2.042-6.095-.279-.838-.416-1.917-.386-2.772.032-.854-.726-6.617-3.544-9.424-1.527-1.519-4.439-1.695-5.757-1.7-1.32-.005-3.031.637-4.788 1.815-2.395 1.606-8.073 6.813-8.966 6.086-.893-.727-.697-5.476-1.035-6-.341-.524-1.003.133-1.837.489-.834.356-3.258 2.817-3.902 3.783-.39.583-.845 1.989-1.181 3.305-.949 1.267-1.916 2.872-2.225 3.489-.559 1.117-1.23 3.015-.252 5.289.979 2.274 2.491 4.363 3.246 5.406.079.111.179.241.288.38.183 1.025.346 2.078.509 3.211.377 2.617-.365 4.675-.652 6.475-.29 1.8-1.27 6.198-1.752 8.706 0 0 23.626 3.676 22.547 2.343-1.521-1.87-2.272-4.221-3.175-5.982-1.638-3.2-.891-6.997-.824-8.552.07-1.556 1.284-3.484 2.005-4.455.723-.971 2.262-5.428 2.692-6.073.429-.645 1.076 1.212 1.697 2.651 2.231 5.159 5.655 20.019 5.667 24.018.002.836.199 2.199.814 4.919.598 2.649 2.132 11.967 2.466 13.809.334 1.844-.053 1.865-.637 3.915-.586 2.05-.368 3.201-.459 4.368-.09 1.169.142 2.555.222 4.022.059 1.07-.04 1.492-.077 2.029.374-.035.749-.063 1.123-.098.18-.714-.036-1.908-.036-1.908l.035-.777c.035-.778.771-1.516.771-1.516s.843 2.667.877 3.286a4.56 4.56 0 0 1-.051.757c.57-.054 1.14-.113 1.71-.17.051-.312.085-.632.105-.994.077-1.4-.681-9.843-.62-11.552.062-1.711.832-16.989.917-18.235.088-1.247-.131-2.398-.846-4.143z"/><path fill="#F1DCBC" d="M13.275 114.357c.248 2.93.465 5.505.518 6.485.142 2.57.195 10.608.117 11.522-.08.915 10.276-.407 10.454-3.326.08-1.332.027-2.455-.175-3.157-.551-1.919-1.355-8.497-1.409-11.782-3.166.125-6.334.209-9.505.258zm32.218-.586c-.014-.246-.036-.633-.067-1.125-3.666.317-7.336.602-11.008.84.807 2.436 1.666 5.042 2.654 7.846 2.619 7.427 3.574 7.218 4.677 9.679.352.787.643 2.24.886 3.907.143.979 1.293 3.566 3.45 7.762 3.916-6.327 5.657-10.023 5.223-11.089a95.252 95.252 0 0 0-.948-2.244c-1.548-3.546-3.162-6.761-3.297-7.145-.165-.464-1.457-6.384-1.57-8.431z"/><path fill="#252A1D" d="M46.24 162.453c.456-1.76-2.155-16.589-2.526-18.303-.254-1.175-.565-5.746-1.09-9.302 1.91 2.246 3.142 4.89 3.142 4.89s.696-3.069 1.62-5.936c.631-1.954 2.083-3.652 2.94-4.533.334.764.669 1.546.982 2.322 1.111 2.729 2.197 7.29.892 13.669-1.306 6.379-2.464 9.117-1.716 16.484.49 4.86 2.943 6.733.853 11.422-2.09 4.687-3.008 5.666-5.125 4.834-2.117-.828-2.998-6.143-2.061-9.191.936-3.048 1.636-4.597 2.089-6.356m-44.51 6.389c.866-.809 6.047-4.125 7.04-4.938.995-.815 2.133-1.332 3.698-3.543 1.14-1.611 1.047-6.884 1.273-7.503.224-.62.089-19.579.169-20.494.003-.037.006-.088.008-.147a16.36 16.36 0 0 1 3.131-1.887c2.284-1.059 4.805-1.744 7.337-1.736-.117 2.953-.955 7.146-2.35 9.998-1.983 4.055-3.017 8.815-2.938 12.605.077 3.788.527 7.253 1.341 10.242.813 2.989 2.103 5.196.124 6.974-1.976 1.775-2.577.294-5.18.132-2.605-.16-2.563.598-4.549 2.226-1.984 1.625-5.679 1.373-8.314.608-2.639-.767-1.521-1.855-.79-2.537"/><path fill="#1A1919" d="M38.727 49.838c.449.402.692 1.411.731 2.504-.284-.531-.544-.81-.752-.497-.309.463-1.191 2.895-1.929 4.589.11-.501.173-1.069.172-1.725-.054-1.02-1.073-7.101 1.778-4.871"/><path fill="#F1DCBC" d="M21.655 8.25c4.388-1.947 12.618 2.256 12.52 10.33-.1 8.073-5.219 10.99-6.67 11.38-1.454.391-7.342-5.104-8.449-9.697-1.105-4.594-1.795-10.064 2.599-12.013"/><path fill="#50341D" d="M23.135.769c-1.631.621-2.748 2.046-2.998 3.769-.12.797-.689 1.004-1.064 1.724a3.338 3.338 0 0 0-.298 2.244c.571 2.631 3.126 4.794 5.193 6.362 2.318 1.757 4.762 3.332 7.077 5.095 1.198.912 2.495 1.752 3.588 2.787 1.112 1.048 1.803 2.418 2.41 3.802 1.484 3.374 2.887 6.494 5.852 8.851 1.346 1.07 2.862 1.8 4.174 2.894a12.895 12.895 0 0 0 4.708 2.483c2.835.813 5.748.852 8.477 2.144 1.402.665 2.881 1.604 3.329 3.18.181.641.253 1.37.027 2.011-.029.087-.869 1.224-.98.987.083.36 1.463-.994 1.563-1.212.341-.746.402-1.609.307-2.415-.236-2.029-1.894-3.62-3.454-4.78-1.773-1.313-3.776-2.296-5.836-3.069-1.184-.438-2.308-.912-3.303-1.72 1.813.471 3.635.805 5.368 1.544 1.095.463 2.153 1.004 3.211 1.539-1.486-1.529-2.993-3.018-4.785-4.198a20.193 20.193 0 0 1 5.264 2.815c.179.135 2.663 1.999 2.54 2.233.194-.134-2.047-3.261-2.183-3.432 1.785.628 3.555 1.427 5.121 2.501.738.501 1.14 1.155 1.283 2.041.031.192.239 1.36.086 1.47.331.032.268-2.636.208-2.891-.325-1.39-1.844-2.481-2.858-3.371-2.366-2.078-5.292-3.345-7.792-5.233-2.252-1.699-3.93-4.136-5.459-6.469-1.685-2.569-3.198-5.247-4.788-7.874-2.795-4.622-6.177-9.043-10.386-12.473C33.37 1.364 27.417-.864 23.135.769c-.943.36.948-.361 0 0"/><path fill="#343534" d="M11.419 107.536c0 3.597-.359 12.226-.359 12.226s9.709 2.878 19.777 3.238c10.069.359 17.98-3.598 19.418-4.316 1.439-.72 6.114-1.079 6.114-1.079S41.625 82.724 37.31 75.172c-4.315-7.552-21.935-.719-21.935-.719s-3.956 29.486-3.956 33.083z"/><path fill="#CBCCCB" d="M35.787 113.862L65.21 113.862 65.21 91.636 35.787 91.636z"/><path fill="#FFFFFE" d="M53.356 102.432a3.386 3.386 0 1 1-6.775 0 3.386 3.386 0 0 1 6.775 0"/><path fill="#F1DCBC" d="M56.411 114.063c-.102-.647-1.022-4.939-1.601-6.029-.579-1.09-2.213-5.415-2.316-6.301a6.177 6.177 0 0 0-.081-.502c-.072-1.187-.117-2.162-.1-2.627.056-1.573.712-14.603.884-17.671-1.449.638-2.606 1.769-2.948 3.462-.025.119-.162.146-.251.081-1.181-.862-2.301-1.782-3.46-2.641.098.475.213 1.002.346 1.598.598 2.649 2.132 11.967 2.466 13.809.334 1.844-.053 1.865-.637 3.915a10.41 10.41 0 0 0-.38 2.055l-.028.02s-1.124 5.755-1.159 6.744c-.034.988.103 2.794-.102 3.44-.204.647-.953 2.248-.953 2.248s1.226.408 1.669-.102c.442-.511 1.635-2.419 1.635-2.589 0-.17.238 2.112.238 2.112s.238 1.123.851.987c.614-.135.818-.987 1.397-.987.579 0 .238.681.783.681s.817-.919 1.26-.988c.443-.068.409.545 1.124.477.716-.068 1.465-.546 1.363-1.192"/></g><g transform="translate(925 1113)"><path fill="#907E63" d="M43.628 20.934s2.299 6.159 2.791 7.309c.493 1.149 1.971 4.106 2.3 4.27.328.164.083-.411 2.956-.247 2.874.165 2.793 3.203 2.793 3.203s-17.247 12.424-22.502 12.26c-5.257-.165-14.432-9.027-14.432-9.027s2.408-1.697 5.036-2.026c2.627-.328 7.508-1.207 7.508-1.207l-.82-5.995 14.37-8.54"/><path fill="#605442" d="M44.112 22.228l-.484-1.294-14.37 8.54.82 5.995.024.655c.666.174 1.467 0 2.132-.092.548-.075 1.096-.154 1.647-.16 3.391-.034 3.815-2.246 5.641-3.714.738-.595 1.789-1.177 2.504-1.841.698-.646 1.863-1.468 1.995-2.402.298-2.125-.756-3.727.091-5.687"/><path fill="#907E63" d="M23.137 12.53c-.406 1.201.598 3.28.743 3.564.146.285-.536 1.543-.805 2.314-.269.77.857 2.596 1.281 3.537.425.941.411 7.181.669 8.013.259.834 1.031 1.806 2.206 2.213.46.16 1.035-.233 1.035-.233s-.779.92 2.134 1.171 4.239.024 4.958-.784a13.006 13.006 0 0 1 3.252-2.652c1.532-.891 3.177-1.235 4.235-3.428 1.059-2.195.244-5.143.316-5.759.072-.615.908-3.193 1.026-4.962.117-1.77.029-9.982-4.898-10.736-4.928-.754-8.752-2.182-11.997 1.184-.347.361-3.747 5.358-4.155 6.558z"/><path fill="#252A1D" d="M10.417 216.905a11.96 11.96 0 0 0 2.741-.509c1.224-.385 2.236-1.049 3.383-2.22.142-.147.282-.296.423-.448.45-.487.914-.99 1.472-1.293a3.933 3.933 0 0 1 1.637-.468c.996-.072 2.05.157 3.07.377.499.11 1.016.22 1.512.296.22.031.441.039.661.023 2.152-.156 3.716-2.509 3.626-4.429-.082-1.747-.762-3.463-1.416-5.124-.417-1.051-.81-2.047-1.064-3.048-.719-2.846.689-5.624 2.051-8.308.347-.687.676-1.331.972-1.982 2.673-5.892 5.112-11.703 5.595-18.042.273-3.683.054-6.965-.652-9.759l-.097-.387-.354.183c-1.25.648-2.542 1.271-3.791 1.876-2.015.974-4.099 1.981-6.081 3.12-1.893 1.087-3.476 2.185-4.841 3.359l-.106.09-.003.14c-.127 4.324-.002 8.758.154 12.904.001.081.061 8.992-.502 11.858-.249 1.263-.763 2.49-1.259 3.679a37.83 37.83 0 0 0-.742 1.858c-.558 1.549-1.695 3.019-3.473 4.488a140.341 140.341 0 0 1-4.659 3.709c-.438.336-.908.671-1.358.999-1.195.86-2.429 1.749-3.46 2.788l-.002.001-.228.237c-.438.454-1.041 1.074-.997 1.832.066 1.178 1.646 1.541 2.406 1.72 1.912.439 3.723.601 5.382.48m48.342-17.409c-.061-.729-.12-1.391-.177-1.998-2.779.008-5.558.027-8.333.166.123 1.295.118 2.214-.023 2.725a69.888 69.888 0 0 0-.673 2.688c-.419 1.782-.896 3.802-1.874 6.799-1.546 4.737-.235 12.915 3.039 14.265.754.312 1.371.455 1.942.455h.001c2.291 0 3.795-2.391 6.296-7.75 2.28-4.883 1.622-7.814.789-11.523-.372-1.659-.794-3.54-.987-5.827"/><path fill="#1A1919" d="M43.173 6.685C41.498 1.614 35.196-.77 29.067 1.234c-6.128 2.004-9.714 7.787-8.059 12.795 5.94-5.694 14.013-8.316 22.165-7.344"/><path fill="#1A1919" d="M42.825 26.504c5.014-1.74 7.194-8.587 4.866-15.295C45.364 4.504 39.413.477 34.398 2.216l8.427 24.288"/><path fill="#907E63" d="M10.651 78.683a45.745 45.745 0 0 1 1.863-3.532c1.434 1.934 3.564 3.845 5.364 7.127 2.666 4.863 2.646 2.235 6.408 2.956 3.761.723 2.648 2.609 4.168 4.851 1.519 2.242 2.614 3.583 2.169 5.555-.446 1.982-2.352 3.947-6.213 4.788-2.96.642-4.507.99-6.024-.878-1.517-1.866-3.068-8.237-5.338-10.474l-2.663-10.294c.093.056.223-.01.266-.099"/><path fill="#343534" d="M19.27 83.782s-2.21 36.436-1.916 49.688c.269 12.047-2.613 44.024-2.613 44.024l-3.945 27.935 21.639.606s3.332-31.497 3.661-37.41c.329-5.914 3.075-39.73 3.075-39.73s4.058 45.376 4.387 47.675c.329 2.299 3.09 30.951 3.09 30.951l7.213 3.394 12.941-1.697s-1.899-43.82-2.885-49.732c-.985-5.914-2.495-21.418-3.48-27.332-.985-5.912-7.228-41.395-7.228-41.395l-1.971-3.614-31.968-3.363"/><path fill="#D89A13" d="M52.236 79.356c.742-4.081 5.936-21.521 5.936-21.521l12.653 14.101-15.621 17.439 3.339 8.163s24.155-19.666 25.639-23.005c1.485-3.34-15.62-33.394-18.959-36.734-3.34-3.34-10.532-5.159-13.8-5.362-3.269-.204-20.337 12.783-20.337 12.783l-2.339-9.926s-9.352.124-13.183 3.244c-2.459 2.001-7.112 10.393-9.338 15.958-2.227 5.566-9.018 15.987-4.937 20.811 4.082 4.824 13.789 17.201 13.789 17.201A639.882 639.882 0 0 0 18.968 87l-.34-5.241c-2.716-2.696-6.805-7.234-6.805-10.195 0-4.452 5.33-12.244 5.33-12.244l1.984 30.162 32.376-.816s-.019-5.228.723-9.31z"/><g transform="rotate(24 -140.5 137.11)"><path fill="#CBCCCB" d="M0.909 25.869L33.909 25.869 33.909 0.869 0.909 0.869z"/><path fill="#FFFFFE" d="M21.663 12.71a4 4 0 1 1-8.001 0 4 4 0 0 1 8 0"/></g><path fill="#907E63" d="M63.861 83.712l-7.227 3.596c-2.666 4.862-2.646 2.234-6.407 2.955-3.762.722-2.648 2.609-4.168 4.851-1.52 2.241-2.615 3.582-2.17 5.555.447 1.982 2.352 3.947 6.213 4.788 2.961.643 4.508.991 6.024-.878 1.517-1.866 3.068-8.238 5.338-10.474l2.663-10.294c-.093.057-.223-.01-.266-.099"/><path fill="#D89A13" d="M63.687 63.981l7.138 7.955-15.621 17.439 3.339 8.163s24.155-19.666 25.639-23.005c.76-1.711-3.358-10.434-8.005-18.985"/><path fill="#907E63" d="M39.404 18.504c.021 2.062.83 3.728 1.805 3.718.975-.011 1.748-1.692 1.727-3.755-.022-2.063-.829-3.728-1.804-3.717-.976.009-1.75 1.69-1.728 3.754"/><path fill="#583F07" d="M27.989 38.209L31.086 45.22 30.232 36.567 27.989 38.209"/><path fill="#886210" d="M29.74 33.119L26.456 34.268 24.54 42.862 30.232 36.567 29.74 33.119"/><path fill="#886210" d="M47.748 30.82L39.921 38.757 48.623 36.403 49.5 40.618 51.196 32.024 47.748 30.82"/></g></g></g></g></svg>
```

## File: static\src\img\banner_default_all.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="1280" height="515"><defs><path id="a" d="M0 0L1280 0 1280 515 0 515z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><use fill="#845978" xlink:href="#a"/><g mask="url(#b)"><g transform="translate(-238 -978)"><path fill="#FFF" opacity=".101" d="M2560 1257.215L1494.187 1286.119 2545.323 1103.766 2526.265 1005.092 1481.549 1245.226 2481.399 853.6 2443.725 760.682 1469.888 1204.02 2372.678 624.649 2318.107 540.942 1442.872 1171.185 2219.915 418.519 2150.277 347.403 1416.649 1137.689 2033.686 249.494 1951.901 193.491 1379.408 1117.883 1815.318 118.773 1724.348 80.284 1342.631 1097.186 1579.95 35.41 1483.409 15.639 1300.81 1093.465 1329.232 0 1230.768 0 1259.073 1088.687 1080.55 14.989 983.951 34.456 1219.029 1101.615 835.652 80.284 744.685 118.773 1178.704 1113.549 611.515 191.345 529.566 247.096 1146.553 1141.127 409.713 347.396 340.088 418.516 1113.739 1167.899 244.251 537.623 189.416 621.159 1094.35 1205.947 116.272 760.679 78.594 853.6 1074.111 1243.515 34.664 1001.099 15.304 1099.718 1070.511 1286.254 0 1257.215 0 1357.78 1065.826 1328.876 14.672 1511.237 33.732 1609.9 1078.463 1369.772 78.594 1761.395 116.272 1854.316 1090.122 1410.977 187.322 1990.346 241.893 2074.063 1117.133 1443.818 340.088 2196.479 409.708 2267.597 1143.349 1477.318 526.309 2365.501 608.092 2421.514 1180.607 1497.092 744.675 2496.227 835.644 2534.721 1217.369 1517.806 980.047 2579.588 1076.586 2599.359 1259.195 1521.54 1230.768 2615 1329.232 2615 1300.932 1526.29 1479.453 2600.009 1576.049 2580.541 1340.973 1513.385 1724.348 2534.721 1815.318 2496.227 1381.305 1501.468 1948.487 2423.652 2030.447 2367.904 1413.474 1473.89 2150.284 2267.597 2219.915 2196.479 1446.256 1447.099 2315.757 2077.374 2370.584 1993.836 1465.623 1409.033 2443.733 1854.316 2481.399 1761.395 1485.901 1371.472 2525.339 1613.896 2544.686 1515.272 1489.581 1328.754 2560 1357.78 2560 1257.215"/><path fill="#845978" d="M238 1530L1649 1530 1649 1288 238 1288z"/><path fill="#62495B" d="M1184.337 1239.582c.847 2.898-.292 11.332-2.838 16.44-2.61 5.243-3.973 11.4-3.87 16.3.104 4.9.696 9.381 1.768 13.249.61 2.202 1.424 4.075 1.53 5.704.116 1.291-.188 2.43-1.356 3.457-2.603 2.295-3.393.38-6.824.172-3.432-.206-3.377.77-5.994 2.872-2.615 2.1-7.481 1.773-10.954.786-1.726-.491-2.233-1.085-2.187-1.666-.052-.639.623-1.273 1.146-1.75 1.139-1.047 7.964-5.337 9.272-6.39 1.31-1.052 2.808-1.722 4.869-4.58 1.502-2.083 1.38-8.905 1.677-9.706.296-.8.117-25.32.223-26.503.103-1.182.032-11.58-.154-14.905-.187-3.326-1.853-20.897-1.838-23.644.015-2.42-.22-10.46-.078-12.351.41-5.36 1.482-11.253 1.951-13.672-4.047-5.29-17.535-20.276-17.058-27.208-.057-.897.08-1.685.456-2.335 1.178-2.043 4.149-6.69 10.967-14.3 1.108-1.237 5.609-8.446 6.549-9.792 2.508-3.59 4.842-6.03 7.015-7.427-.027.117-.055.234-.085.351.4-.253.795-.47 1.184-.652.237-.186.453-.34.638-.455.473-.493 1.117-.989 1.75-1.443.481-1.24 1.321-2.015 2.509-2.517-3.11-1.568-6.466-5.624-7.866-11.291-.793-3.207-1.213-6.385-1.116-9.238a36.606 36.606 0 0 1-.004-1.584c.05-2.044-.037 2.043 0 0 .03-1.213.25-3.094.303-4.57a7.24 7.24 0 0 1-.374-.35 4.327 4.327 0 0 1-1.252-2.69c-.086-1.057.41-1.68-.018-2.645-.936-2.078-.973-2.206.426-4.024 3.676-4.777 11.967-6.44 17.705-5.739 7.173.876 14.116 3.421 20.537 6.594 3.653 1.805 7.251 3.718 10.968 5.39 3.375 1.521 6.992 3.053 10.734 3.371 4.154.355 8.352-.273 12.49.385 1.772.281 4.26.434 5.608 1.746.248.241 2.206 3.148 1.81 3.342.095-.226-.966-1.376-1.136-1.566-.788-.882-1.703-1.326-2.893-1.368-2.524-.101-5.084.242-7.542.785.274.095 5.011 2 4.888 2.282-.025-.343-4.147-.683-4.444-.707a27.4 27.4 0 0 0-7.93.534c2.858.064 5.61.666 8.367 1.327-1.572.141-3.148.277-4.711.525-2.476.384-4.768 1.278-7.145 2.015 1.693.203 3.296-.053 4.942-.39 2.869-.573 5.825-.874 8.754-.652 2.58.202 5.574.812 7.277 2.893.675.826 1.216 1.822 1.359 2.883.022.163-.131.914-.333 1.544 4.53-.863 9.602-.703 13.088 2.02 1.676 1.31 2.9 3.46 3.444 5.487.458 1.708.577 3.55.225 5.286-.36 1.774-1.093 2.183-.192 3.927 1.311 2.557 3.37 4.884 5.808 6.424 1.445.911 3.14 1.577 4.877 1.47.442-.026 4.739-1.7 4.322-2.072.184.126-2.103 2.09-2.29 2.202-1.71 1.025-3.767 1.167-5.687.749-3.085-.667-6.333-2.054-9.115-3.504 1.537 2.123 2.864 4.395 4.24 6.623-1.362-1.229-2.709-2.476-4.09-3.683-1.857-1.615-3.806-3.24-6.152-4.093-2.191.973-4.198 2.336-6.114 3.76 3.956-.25 8.58.182 11.377 3.33 1.338 1.483 2.231 3.402 2.041 5.427-.035.392-1.534 4.865-1.974 4.541 1.149.416.78-5.87.497-6.42-1.249-2.423-3.903-3.582-6.528-3.677-4.764-.174-9.383 1.896-13.727 3.607-6.39 2.517-12.823 3.525-18.704 1.79 2.999 6.302 5.64 12.88 11.16 17.713 8.531 7.055 19.945 9.093 30.411 10.297 13.644 1.476 26.787 2.094 37.221 10.814 7.107 5.805 18.999 8.258 28.462 7.75-12.617.678-17.857 12.01-26.522 18.664-8.667 6.653-20.557 4.2-30.995 3.437-13.168-1.06-15.554 12.77-20.491 21.434-4.558 7.316-14.275 11.375-22.461 10.491-7.107-.997-15.08-7.592-22.722-9.996.467 2.288.937 4.31 1.045 4.606.212.6 3.277 6.553 5.59 12.14 1.463 3.533 2.892 9.43 1.173 17.681-1.72 8.25-3.245 11.791-2.26 21.321.641 6.24 3.83 8.669 1.184 14.64-.103.294-.219.596-.349.907-1.936 4.633-2.625 6.691-4.74 6.816-.578.031-1.172-.081-1.816-.352-2.175-.91-3.326-5.513-3.108-9.464.019-1.23.178-2.415.49-3.412.688-2.203 1.28-3.8 1.771-5.177.302-.93.556-1.757.77-2.639.54-2.223-2.647-21.654-3.117-23.984-.05-.245-.1-.596-.154-1.034-.475-3.534-1.13-13.188-2.435-16.05-1.452-3.185-2.71-2.913-6.159-12.517-.37-1.031-.726-2.042-1.071-3.033-5.187 1.108-9.326 2.34-12.572 3.377-.905.288-2.457.932-4.1 1.502.263 1.753.538 3.221.777 4.037zm-11.709-65.5c1.087-5.093 2.296-10.76 2.52-12.129.378-2.328 1.356-4.99.86-8.373-.388-2.641-.758-4.978-1.322-7.285-.747.847-1.202 1.338-1.202 1.338s-9.15 7.925-10.484 10.847c.444 1.05 2.883 5.021 5.29 8.849 1.714 2.697 3.396 5.3 4.338 6.753zm61.355-68.14c.228.352.467.7.718 1.041 3.884 5.285 10.252 2.212 15.227.516 1.19-.406 2.49-.765 3.843-1.034.062-.433.087-.88.074-.937-.198-.864-.796-1.621-1.454-2.205-1.619-1.431-3.95-1.446-6.002-1.208-3.992.463-7.307 2.44-11.08 3.506-.437.124-.88.23-1.326.32z"/><path fill="#62495B" d="M1331.97 1238.626l.367-.223.221.408c.275.51.574.931.84 1.303.216.303.42.59.583.883l.064.114c4.362 7.816 5.312 15.846 5.339 21.21.039 7.821-2.491 17.906-4.167 24.58l-.242.962c.212 1.792.492 4.332.815 8.157.27 3.188.858 5.808 1.379 8.121 1.164 5.17 2.085 9.252-1.103 16.058-3.496 7.467-5.598 10.801-8.8 10.801h-.002c-.797 0-1.66-.2-2.712-.635-4.58-1.881-6.41-13.277-4.25-19.88 1.368-4.174 2.033-6.992 2.62-9.475.303-1.28.59-2.487.94-3.746.417-1.5.013-5.424-1.203-11.658-2.256-7.179-4.35-18.57-5.474-24.693-.154-.832-.289-1.57-.406-2.188-.387-2.067-.62-4.118-.848-6.1-.298-2.614-.581-5.083-1.173-7.008l-.327-1.061.544.309a8.815 8.815 0 0 0-.566-1.255c-1.825-3.28-2.65-1.935-4.132-7.236-1.126-4.03-2.505-7.587-5.064-12.134-.473 1.561-1.02 3.08-1.54 4.657-1.003 3.032-1.03 5.612.02 8.575.39 1.104.693 2.256.921 3.436l.417-.178.096.547c.7 3.956.67 8.541-.086 13.632-1.32 8.76-5.312 16.59-9.638 24.507-.478.873-1.003 1.739-1.557 2.657-2.172 3.594-4.418 7.31-3.705 11.336.252 1.42.697 2.841 1.172 4.348.744 2.373 1.515 4.829 1.452 7.264-.071 2.677-2.49 5.787-5.507 5.787-.307 0-.615-.032-.918-.098a34.52 34.52 0 0 1-2.077-.564c-1.4-.41-2.846-.837-4.24-.837-.855 0-1.618.157-2.331.483-.81.365-1.509 1.016-2.185 1.646-.21.2-.42.394-.636.582-1.716 1.51-3.194 2.332-4.942 2.743a16.802 16.802 0 0 1-3.87.429c-2.328 0-4.836-.409-7.456-1.216-1.04-.323-3.206-.989-3.178-2.633.018-1.058.919-1.86 1.576-2.447l.345-.306c1.544-1.341 3.353-2.45 5.107-3.526.66-.407 1.35-.829 1.996-1.248 2.074-1.342 4.258-2.833 6.874-4.683 2.628-1.862 4.36-3.785 5.296-5.883.38-.85.81-1.695 1.225-2.51.812-1.6 1.654-3.254 2.13-4.984 1.076-3.925 1.9-16.317 1.908-16.43.205-5.779.481-11.955 1.098-17.95l.02-.194.158-.115c.106-.078.212-.156.32-.233.162-1.516.347-3.027.56-4.53.592-4.18 1.876-8.963-.08-12.97-1.553-3.182-1.278-6.316-1.553-9.79-.595-7.467-3.566-14.373-4.586-21.76a134.789 134.789 0 0 1-1.17-22.97c.096-2.784.253-5.595.683-8.337-.568-.9-1.161-1.672-1.78-2.214-2.905-2.543-17.648-11.6-20.243-16.44-2.6-4.838-.735-14.198.534-19.22 1.237-4.89 3.433-4.748 3.988-10.246.554-5.498 7.1-15.47 7.1-15.47s3.67-10.345 10.06-13.431c2.578-2.049 5.136-2.368 7.123-2.245.422-.104 4.63-1.145 9.092-2.29l-.047-.343c-2.981-.45-2.067-1.528-2.067-1.528s-.803.548-1.445.326c-1.642-.568-2.723-1.923-3.084-3.084-.362-1.16-.341-9.855-.934-11.167-.593-1.311-2.168-3.855-1.793-4.93.378-1.074 1.331-2.826 1.128-3.224-.203-.397-1.608-3.293-1.04-4.966.206-.605.958-1.9 1.858-3.332-.183.196-.289.314-.289.314s.023-2.072 1.68-5.607c1.66-3.534 5.244-6.47 5.244-6.47l-.234 3.078s.56-2.669 4.162-3.616c3.6-.947 6.081.213 6.081.213l-2.045 1.5s3.3-1.604 6.868-1.187c3.568.414 6.966 3.295 6.966 3.295l-1.901.277s1.904 1.836 4.233 3.226c2.33 1.388 2.706 2.427 2.706 2.427s-1.916.397-2.174 2.605c-.26 2.207-.802 8.977-1.719 11.48-.695 1.896-1.884 3.735-2.418 4.514l.3.797c-1.185 2.732.29 4.964-.128 7.925a2.378 2.378 0 0 1-.358.918c14.484-2.305 31.741-1.504 40.772 1.926 17.559 6.732 27.689 24.24 47.948 24.912 20.262.674 40.522-12.793 61.456-7.406 6.078 1.347 10.13 4.714 14.184 8.753 5.402 4.713 10.803 9.426 16.884 12.793-12.833 4.714-22.961 2.693-24.99 19.527-1.348 14.14-2.026 27.606-18.233 30.973-14.183 3.367-29.04-4.04-41.872 4.714-8.104 5.386-7.429 15.487-12.156 22.892-4.726 10.1-11.48 16.833-22.96 15.487-12.834-1.347-22.96-10.1-35.794-9.427-2.62 0-5.245.635-7.866 1.903.945 9.11 1.565 17.69 1.13 21.006-.56 4.294-.024 6.665.71 8.211zm-50.243-80.861c1.092-4.104-.2-11.253-.843-15.414-.464-2.979-1.828-5.89-3.08-8.56-2.677 5.731-6.853 10.803-5.855 14.223 1.166 3.992 5.353 5.992 8.902 11.585.251-.663.726-2.018.726-2.018l.15.184z"/></g></g></g></svg>
```

## File: static\src\img\banner_heroes_default.svg

```svg
<svg height="749" viewBox="0 0 1920 749" width="1920" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"><defs><path id="a" d="m0 0h1920v749h-1920z"/><mask id="b" fill="#fff"><use fill="#fff" fill-rule="evenodd" xlink:href="#a"/></mask></defs><g fill="none" fill-rule="evenodd"><use fill="#36576b" fill-rule="nonzero" xlink:href="#a"/><path d="m3341 938.466-1537.519 41.694 1516.347-263.036-27.493-142.331-1507.085 346.379 1442.362-564.899-54.347-134.029-1404.837 639.492 1302.345-835.712-78.722-120.744-1262.595 909.092 1120.945-1085.68-100.459-102.581-1058.316 1139.946 890.125-1281.175-117.982-80.781-825.865 1333.387 628.834-1441.164-131.231-55.518-550.657 1466.828 342.352-1531.557-139.269-28.519-263.413 1554.707 41.001-1577.265h-142.042l40.832 1570.374-257.534-1548.753-139.351 28.08 339.119 1539.321-553.052-1473.216-131.227 55.518 626.107 1434.912-818.215-1330.23-118.218 80.418 890.052 1289.592-1062.949-1144.916-100.44 102.588 1116.053 1080.944-1254.306-909.138-79.103 120.495 1305.438 843.526-1410.953-642.275-54.355 134.033 1436.113 562.432-1499.485-349.672-27.928 142.252 1522.219 269.069-1544.296-41.888v145.061l1537.537-41.694-1516.372 263.046 27.497 142.318 1507.106-346.373-1442.391 564.896 54.355 134.033 1404.855-639.493-1302.36 835.709 78.722 120.758 1262.602-909.096-1120.948 1085.674 100.433 102.584 1058.334-1139.935-890.128 1281.157 117.978 80.795 825.897-1333.43-628.866 1441.2 131.231 55.525 550.668-1466.845-342.356 1531.564 139.266 28.519 263.427-1554.697-41.008 1577.258h142.042l-40.825-1570.407 257.53 1548.783 139.348-28.081-339.116-1539.317 553.049 1473.223 131.231-55.525-626.097-1434.887 818.204 1330.202 118.233-80.415-890.031-1289.567 1062.906 1144.88 100.448-102.584-1116.064-1080.941 1254.323 909.139 79.093-120.499-1305.478-843.548 1411.001 642.297 54.336-134.033-1436.084-562.443 1499.471 349.683 27.91-142.26-1522.072-269.043 1544.163 41.87z" fill="#fff" mask="url(#b)" opacity=".043"/></g></svg>
```

## File: static\src\img\quiz_modal_success.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="377" height="503"><defs><filter id="a" width="218.6%" height="210.3%" x="-59.3%" y="-55.1%" filterUnits="objectBoundingBox"><feOffset in="SourceAlpha" result="shadowOffsetOuter1"/><feGaussianBlur in="shadowOffsetOuter1" result="shadowBlurOuter1" stdDeviation="110"/><feColorMatrix in="shadowBlurOuter1" result="shadowMatrixOuter1" values="0 0 0 0 1 0 0 0 0 1 0 0 0 0 1 0 0 0 0.126174607 0"/><feMerge><feMergeNode in="shadowMatrixOuter1"/><feMergeNode in="SourceGraphic"/></feMerge></filter></defs><g fill="none" fill-rule="evenodd" filter="url(#a)" transform="scale(-1 1) rotate(10 -170.596 -2124.816)"><path fill="#DD9E63" d="M200.228 136.773l-.97 5.966 3.6 8.464-13.154-2.359c-2.309 4.07 1.061 8.372 10.107 12.904 3.324 1.295 5.078 1.156 5.262-.416l2.078 4.44.415.415 18.523 10.223 9.554 2.914 30.74 8.88-5.507-18.27c-3.046.832-8.4 1.063-16.062.693l14.124-7.353-.696.752.835-.613 1.246-.833 4.016-2.914-23.955-42.317-9.14-9.296-4.292-3.053-30.602 24.42 3.878 7.353z"/><path fill="#17252E" d="M342.943 366.508l-.139-.14c0 .092-.046.186-.14.279l-.976-.14h-.976l.28-.278L333 364c7.53 15.138 12.41 28.836 14.641 41.096 3.08 16.986 4.64 32.406 4.683 46.262l12.708-2.261-6.75-38.29-15.339-43.882v-.417zM108 100.758c-.647-.648-1.551-1.84-2.105-2.394-15.432-15.187-39.272-33.43-71.52-54.727V31.969l-3.189-15.278L16.494.022C9.287-.533 3.788 9.374 0 29.746c8.87 9.816 16.217 17.317 22.038 22.502 11.459 25.743 25.202 49.275 41.004 69.368 17.28 9.26 32.644 3.938 44.01-16.803"/><path fill="#321714" d="M116 148l14-1.82-5.142-12.18c.094 5.133-2.857 9.8-8.858 14"/><path fill="#5E2A1C" d="M344 289L363 301 357.396 289.061z"/><path fill="#17252E" d="M322.087 175.43a298.543 298.543 0 0 0-9.12-14.039 5.557 5.557 0 0 0-.828-1.391c.184 1.112.46 2.179.829 3.198a223.398 223.398 0 0 1 1.935 10.842c1.013 7.875 1.243 15.567.69 23.073-.276 3.15-.69 6.301-1.244 9.452a2.631 2.631 0 0 0-.138.834V208.094c9.12 40.495 8.382 70.982-2.211 91.462 4.422-2.688 8.291-5.607 11.606-8.757 9.95-8.897 16.489 15.66 19.62 73.67 1.197-3.8 2.212-7.553 3.04-11.26.46.092.967.231 1.52.417 23.303 7.91 44.755 13.517 64.357 16.82l-8.682-49.268c-12.076-6.763-24.923-13.368-38.542-19.815l-19.481-12.093 12.02.416 38.134-2.78 19.758 5.56c6.448-6.486 9.533-12.973 9.257-19.459l-13.954-12.232-11.883 8.201c-14.554-8.248-26.943-14.734-37.167-19.46-5.16-2.317-9.764-4.216-13.817-5.699a46.205 46.205 0 0 0-5.25-1.529c1.105-13.437-1.98-28.494-9.258-45.175-3.04-6.95-6.77-14.177-11.191-21.684z"/><path fill="#8C633C" d="M244.453 170.75c7.31.398 13.64.315 16.547-.578l-1.538-7.023-.793.595.661-.744-14.877 7.75z"/><path fill="#A92121" d="M264.154 160.259l-5.154 3.75c10.533-.759 21.498-.795 29.741.725l2.748.57c7.42 1.615 14.42 4.35 19.825 7.864.915-3.23 1.726-7.252 2.275-10.862-.51-.818-1.121-1.772-1.49-2.846l-3.573-.854c-4.763-1.236-10.442-1.758-17.037-1.568h-2.748c-7.053.474-15.52 1.321-24.587 3.22zm-60.15 11.836l7.495 9.23 1.103.28c5.707 1.302 11.713 3.23 17.788 5.277 2.669.836 5.337 1.768 8.007 2.791 1.749.745 3.506 1.512 5.347 2.257 6.074 2.605 12.212 5.3 18.563 8.558L267 203l-31.093-24.578-9.525-2.931-18.086-9.073-.414-.418-7.606.512-.276-.14v.698l4.004 4.746v.279zM156.477 221c-14.296 20.087-29.667 62.398-46.114 126.935-.934 5.486-1.854 10.52-3.162 15.077 4.111-12.926 13.295-30.181 27.404-51.756 14.577-23.62 22.526-52.454 24.395-86.49L156.477 221z"/><path fill="#6E1414" d="M360.542 411.783c11.138-9.89 24.829-10.215 41.072-.978 6.243 3.059 12.472 4.846 18.745 6.193l-7.12-40.378c-15.604-2.143-39.017-5.35-70.239-9.62l15.315 44.084 2.227.7zm-101.268-247.55l1.587 6.787 5.177 16.603-30.038-8.49 30.464 23.778c5.042 2.546 10.172 5.998 15.4 9.089 2.428-1.727 5.329-4.394 7.57-6.304a63.564 63.564 0 0 1 2.801-2.319c2.522-2.362 4.762-4.727 6.723-7.091 6.07-7.182 10.52-15.695 13.042-24.15-5.51-3.365-12.046-5.82-19.61-7.365-.933-.182-1.867-.363-2.8-.546-8.404-1.454-17.991-1.486-28.729-.76l-1.587.767zm-100.6 61.223c-1.857 33.97-10.02 62.742-24.495 86.317-14.01 21.533-23.056 38.75-27.138 51.65v.278c-.093.093-.186.28-.28.558a2.65 2.65 0 0 0-.138.834c-.185.28-.279.604-.279.975v.14c-3.99 13.922-9.834 22.832-17.535 26.73L87 395.863c6.587-.28 13.035-.743 19.344-1.394 17.722-1.855 34.7-5.475 50.938-10.859 2.876-.741 5.66-1.392 8.35-1.948 2.876-.465 5.614-.928 8.21-1.393 11.32-1.763 20.831-1.95 28.532-.557v.975l.278-.975c1.114.28 2.273.557 3.479.836 8.072 2.134 13.731 6.31 16.98 12.53 1.669 3.248 4.174 5.801 7.515 7.657 2.319 1.299 5.01 2.367 8.072 3.202 1.113.186 2.273.418 3.478.696.558 0 1.207.046 1.95.139.649.093 1.391.232 2.225.417 4.547.28 9.79.14 15.727-.417h.836c11.226-1.206 20.226-.65 27 1.672.741.185 1.437.37 2.086.556l-13.639-25.617c-29.844-51.171-45.756-78.088-47.735-80.748-12.805-16.985-31.732-38.054-56.783-63.207-2.69-2.598-5.428-5.243-8.211-7.935a249.14 249.14 0 0 0-8.628-8.493l1.67 4.456z"/><path fill="#1A1B19" d="M201.018 137.549L197.107 130.186 227.973 105.73 232.302 108.787 233 107.537 221.827 96.838 207.861 86 183 120.597 188.168 127.962 183.42 136.437 190.403 149.638 203.671 152 200.04 143.523 201.018 137.549"/><path fill="#BE242B" d="M88.809 392.938c-1.866 1.273-4.675 2.335-6.809 3.062 1.422-.091 3.578-.137 5-.137"/><path fill="#2A4454" d="M291.425 408.806c6.33 16.017 16.288 32.033 29.255 48.05l32.599-5.735c-.067-13.728-1.613-28.99-4.637-45.788-2.208-12.226-7.042-25.886-14.5-40.983-43.822-26.488-63.385-20.792-56.941 17.088L290.734 407c.277.648.506 1.25.691 1.806zm-125.91-267.711c-12.163-6.753-25.257-11.886-39.278-15.4-2.508-6.937-8.59-15.168-18.247-24.695a64.781 64.781 0 0 0-1.81 3.607c-11.422 20.717-25.897 26.102-43.262 16.852 7.428 9.342 15.45 18.48 23.9 26.71a179.683 179.683 0 0 0 8.775 8.186 377.875 377.875 0 0 0 9.333 7.908l1.253 1.387c18.387 15.17 37.37 29.475 58.543 39.279a218.728 218.728 0 0 0 8.218 3.469c18.015 7.492 37.752 11.267 57.624 14.967l8.079 1.388c1.764.368 3.575.647 5.432.831 1.021.185 1.997.324 2.925.416l-34.266-44.534-1.113-.277-7.522-8.88v-.277l-4.04-4.717v-.693c-8.263-7.954-17.039-14.845-26.325-20.671a105.437 105.437 0 0 0-8.218-4.856z"/><path fill="#192A35" d="M130 146.18L116 148c6-4.2 8.952-8.867 8.858-14L130 146.18"/><path fill="#36576B" d="M156.23 221.4c2.962 2.785 6.15 5.811 8.926 8.689 2.778 2.691 5.509 5.337 8.193 7.935 24.997 25.157 43.882 46.229 56.658 63.217a270.764 270.764 0 0 1 8.054 11.417c.864 1.361 14.056 24.475 39.576 69.342-6.48-37.966 12.313-43.675 56.38-17.127l7.083 2.228.693.279.973.139c.093-.093.138-.187.138-.278l.14.139v.417l70.341 9.637-1.334-7.557c-19.514-3.647-40.868-9.586-64.172-17.47a12.468 12.468 0 0 0-1.527-.417 136.644 136.644 0 0 1-3.055 11.278c-3.148-58.11-9.002-82.357-19-73.446-3.333 3.157-7.221 6.08-11.666 8.773 10.647-20.515 11.388-51.055 2.223-91.621v-.14-.556c0-.278.045-.557.138-.835.557-3.157.973-6.313 1.25-9.469.556-7.52.325-15.224-.693-23.113-.557-3.621-1.219-6.936-1.96-10.555-.555 3.527-1.281 6.563-2.206 9.719-2.5 8.633-6.759 16.616-12.776 23.95-1.945 2.413-4.166 4.827-6.666 7.24a63.466 63.466 0 0 0-2.777 2.367 101.299 101.299 0 0 1-6.943 5.57 260.687 260.687 0 0 0-15.276-8.633c-1.573-.836-3.148-1.67-4.721-2.507-6.388-3.249-12.637-6.173-18.747-8.771a323.35 323.35 0 0 1-5.416-2.229 137.827 137.827 0 0 0-8.054-2.784c-6.11-2.042-12.035-3.714-17.775-5.013l34.161 44.696a36.576 36.576 0 0 1-2.916-.418 50.824 50.824 0 0 1-5.416-.834c-2.685-.465-5.37-.93-8.054-1.393-19.812-3.714-38.697-9.329-56.658-16.848a217.273 217.273 0 0 1-8.193-3.482c-21.107-9.84-40.825-22.37-59.156-37.595l50.23 56.06z"/></g></svg>
```

## File: static\src\img\standard_badge_bronze.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="300" height="300"><g fill="none"><circle cx="150" cy="150" r="150" fill="#FFF"/><path fill="#C37933" d="M150 300C67.157 300 0 232.843 0 150S67.157 0 150 0s150 67.157 150 150-67.157 150-150 150zm0-9.375c77.665 0 140.625-62.96 140.625-140.625 0-77.665-62.96-140.625-140.625-140.625C72.335 9.375 9.375 72.335 9.375 150c0 77.665 62.96 140.625 140.625 140.625zm0-14.063C80.101 276.563 23.437 219.9 23.437 150S80.102 23.437 150 23.437 276.563 80.102 276.563 150 219.899 276.563 150 276.563zm56.757-126.57l12.882-12.602c1.867-1.743 2.49-3.922 1.867-6.536-.747-2.551-2.365-4.139-4.854-4.76l-17.55-4.482 4.947-17.365c.747-2.552.156-4.73-1.773-6.535-1.805-1.93-3.983-2.52-6.535-1.774l-17.363 4.948-4.48-17.551c-.623-2.552-2.21-4.14-4.761-4.761-2.552-.685-4.73-.094-6.535 1.773L150 93.324l-12.602-12.977c-1.805-1.93-3.983-2.52-6.535-1.773-2.551.622-4.138 2.209-4.76 4.76l-4.481 17.552-17.363-4.948c-2.552-.747-4.73-.155-6.535 1.774-1.929 1.805-2.52 3.983-1.773 6.535l4.947 17.365-17.55 4.481c-2.489.623-4.107 2.21-4.854 4.761-.622 2.614 0 4.793 1.867 6.536l12.882 12.603-12.882 12.603c-1.867 1.743-2.49 3.921-1.867 6.535.747 2.552 2.365 4.14 4.854 4.762l17.55 4.481-4.947 17.365c-.747 2.552-.156 4.73 1.773 6.535 1.805 1.93 3.983 2.52 6.535 1.774l17.363-4.948 4.48 17.551c.623 2.552 2.21 4.17 4.761 4.855 2.614.622 4.792 0 6.535-1.867L150 206.755l12.602 12.884c1.245 1.369 2.832 2.053 4.761 2.053.436 0 1.027-.062 1.774-.186 2.551-.747 4.138-2.365 4.76-4.855l4.481-17.551 17.363 4.948c2.552.747 4.73.155 6.535-1.774 1.929-1.805 2.52-3.983 1.773-6.535l-4.947-17.365 17.55-4.481c2.489-.623 4.107-2.21 4.854-4.762.622-2.614 0-4.792-1.867-6.535l-12.882-12.603z"/></g></svg>
```

## File: static\src\img\standard_badge_gold.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="300" height="300"><g fill="none"><circle cx="150" cy="150" r="150" fill="#FFF"/><path fill="#E2BE00" d="M150 300C67.157 300 0 232.843 0 150S67.157 0 150 0s150 67.157 150 150-67.157 150-150 150zm0-9.375c77.665 0 140.625-62.96 140.625-140.625 0-77.665-62.96-140.625-140.625-140.625C72.335 9.375 9.375 72.335 9.375 150c0 77.665 62.96 140.625 140.625 140.625zm0-14.063C80.101 276.563 23.437 219.9 23.437 150S80.102 23.437 150 23.437 276.563 80.102 276.563 150 219.899 276.563 150 276.563zm56.757-126.57l12.882-12.602c1.867-1.743 2.49-3.922 1.867-6.536-.747-2.551-2.365-4.139-4.854-4.76l-17.55-4.482 4.947-17.365c.747-2.552.156-4.73-1.773-6.535-1.805-1.93-3.983-2.52-6.535-1.774l-17.363 4.948-4.48-17.551c-.623-2.552-2.21-4.14-4.761-4.761-2.552-.685-4.73-.094-6.535 1.773L150 93.324l-12.602-12.977c-1.805-1.93-3.983-2.52-6.535-1.773-2.551.622-4.138 2.209-4.76 4.76l-4.481 17.552-17.363-4.948c-2.552-.747-4.73-.155-6.535 1.774-1.929 1.805-2.52 3.983-1.773 6.535l4.947 17.365-17.55 4.481c-2.489.623-4.107 2.21-4.854 4.761-.622 2.614 0 4.793 1.867 6.536l12.882 12.603-12.882 12.603c-1.867 1.743-2.49 3.921-1.867 6.535.747 2.552 2.365 4.14 4.854 4.762l17.55 4.481-4.947 17.365c-.747 2.552-.156 4.73 1.773 6.535 1.805 1.93 3.983 2.52 6.535 1.774l17.363-4.948 4.48 17.551c.623 2.552 2.21 4.17 4.761 4.855 2.614.622 4.792 0 6.535-1.867L150 206.755l12.602 12.884c1.245 1.369 2.832 2.053 4.761 2.053.436 0 1.027-.062 1.774-.186 2.551-.747 4.138-2.365 4.76-4.855l4.481-17.551 17.363 4.948c2.552.747 4.73.155 6.535-1.774 1.929-1.805 2.52-3.983 1.773-6.535l-4.947-17.365 17.55-4.481c2.489-.623 4.107-2.21 4.854-4.762.622-2.614 0-4.792-1.867-6.535l-12.882-12.603z"/></g></svg>
```

## File: static\src\img\standard_badge_silver.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="300" height="300"><g fill="none"><circle cx="150" cy="150" r="150" fill="#FFF"/><path fill="#838997" d="M150 300C67.157 300 0 232.843 0 150S67.157 0 150 0s150 67.157 150 150-67.157 150-150 150zm0-9.375c77.665 0 140.625-62.96 140.625-140.625 0-77.665-62.96-140.625-140.625-140.625C72.335 9.375 9.375 72.335 9.375 150c0 77.665 62.96 140.625 140.625 140.625zm0-14.063C80.101 276.563 23.437 219.9 23.437 150S80.102 23.437 150 23.437 276.563 80.102 276.563 150 219.899 276.563 150 276.563zm56.757-126.57l12.882-12.602c1.867-1.743 2.49-3.922 1.867-6.536-.747-2.551-2.365-4.139-4.854-4.76l-17.55-4.482 4.947-17.365c.747-2.552.156-4.73-1.773-6.535-1.805-1.93-3.983-2.52-6.535-1.774l-17.363 4.948-4.48-17.551c-.623-2.552-2.21-4.14-4.761-4.761-2.552-.685-4.73-.094-6.535 1.773L150 93.324l-12.602-12.977c-1.805-1.93-3.983-2.52-6.535-1.773-2.551.622-4.138 2.209-4.76 4.76l-4.481 17.552-17.363-4.948c-2.552-.747-4.73-.155-6.535 1.774-1.929 1.805-2.52 3.983-1.773 6.535l4.947 17.365-17.55 4.481c-2.489.623-4.107 2.21-4.854 4.761-.622 2.614 0 4.793 1.867 6.536l12.882 12.603-12.882 12.603c-1.867 1.743-2.49 3.921-1.867 6.535.747 2.552 2.365 4.14 4.854 4.762l17.55 4.481-4.947 17.365c-.747 2.552-.156 4.73 1.773 6.535 1.805 1.93 3.983 2.52 6.535 1.774l17.363-4.948 4.48 17.551c.623 2.552 2.21 4.17 4.761 4.855 2.614.622 4.792 0 6.535-1.867L150 206.755l12.602 12.884c1.245 1.369 2.832 2.053 4.761 2.053.436 0 1.027-.062 1.774-.186 2.551-.747 4.138-2.365 4.76-4.855l4.481-17.551 17.363 4.948c2.552.747 4.73.155 6.535-1.774 1.929-1.805 2.52-3.983 1.773-6.535l-4.947-17.365 17.55-4.481c2.489-.623 4.107-2.21 4.854-4.762.622-2.614 0-4.792-1.867-6.535l-12.882-12.603z"/></g></svg>
```

## File: static\src\js\portal_rating_composer.js

```javascript
import RatingPopupComposer from "@portal_rating/js/portal_rating_composer";

RatingPopupComposer.include({
    _update_options: function (data) {
        this._super(...arguments);
        this.options.force_submit_url =
            data.force_submit_url ||
            (this.options.default_message_id && "/slides/mail/update_comment");
    },
});

```

## File: static\src\js\slides.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';
import { deserializeDateTime } from "@web/core/l10n/dates";

publicWidget.registry.websiteSlides = publicWidget.Widget.extend({
    selector: '#wrapwrap',

    /**
     * @override
     * @param {Object} parent
     */
    start: function (parent) {
        var defs = [this._super.apply(this, arguments)];

        $("timeago.timeago").toArray().forEach((el) => {
            var datetime = $(el).attr('datetime');
            var datetimeObj = deserializeDateTime(datetime);
            // if presentation 7 days, 24 hours, 60 min, 60 second, 1000 millis old(one week)
            // then return fix formate string else timeago
            var displayStr = '';
            if (datetimeObj && new Date().getTime() - datetimeObj.valueOf() > 7 * 24 * 60 * 60 * 1000) {
                displayStr = datetimeObj.toFormat('DD');
            } else {
                displayStr = datetimeObj.toRelative();
            }
            $(el).text(displayStr);
        });

        return Promise.all(defs);
    },
});

export default publicWidget.registry.websiteSlides;

```

## File: static\src\js\slides_category_add.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import publicWidget from '@web/legacy/js/public/public_widget';
import { CategoryAddDialog } from "@website_slides/js/public/components/category_add_dialog/category_add_dialog";

publicWidget.registry.websiteSlidesCategoryAdd = publicWidget.Widget.extend({
    selector: '.o_wslides_js_slide_section_add',
    events: {
        'click': '_onAddSectionClick',
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _openDialog: function (channelId) {
        this.call("dialog", "add", CategoryAddDialog, {
            title: _t("Add a section"),
            confirmLabel: _t("Save"),
            confirm: ({ formEl }) => {
                if (!formEl.checkValidity()) {
                    return false;
                }
                formEl.classList.add("was-validated");
                formEl.submit();
                return true;
            },
            cancelLabel: _t("Back"),
            cancel: () => {},
            channelId,
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onAddSectionClick: function (ev) {
        ev.preventDefault();
        this._openDialog($(ev.currentTarget).attr('channel_id'));
    },
});

export default {
    websiteSlidesCategoryAdd: publicWidget.registry.websiteSlidesCategoryAdd
};

```

## File: static\src\js\slides_category_delete.js

```javascript
/** @odoo-module **/

import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import publicWidget from "@web/legacy/js/public/public_widget";
import { _t } from "@web/core/l10n/translation";

publicWidget.registry.websiteSlidesCategoryDelete = publicWidget.Widget.extend({
    selector: ".o_wslides_js_category_delete",
    events: {
        click: "_onClickDeleteCateogry",
    },

    init() {
        this._super(...arguments);
        this.orm = this.bindService("orm");
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickDeleteCateogry(ev) {
        const categoryId = parseInt(ev.currentTarget.dataset.categoryId);
        this.call("dialog", "add", ConfirmationDialog, {
            title: _t("Delete Category"),
            body: _t("Are you sure you want to delete this category?"),
            confirmLabel: _t("Delete"),
            confirm: async () => {
                /**
                 * Calls 'unlink' method on slides.slide to delete the category and
                 * reloads page after deletion to re-arrange the content on UI
                 */
                await this.orm.unlink("slide.slide", [categoryId]);
                window.location.reload();
            },
            cancel: () => {},
        });
    },
});

export default {
    websiteSlidesCategoryDelete: publicWidget.registry.websiteSlidesCategoryDelete,
};

```

## File: static\src\js\slides_course_enroll_email.js

```javascript
/** @odoo-module **/

import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { _t } from "@web/core/l10n/translation";
import { escape } from "@web/core/utils/strings";
import publicWidget from "@web/legacy/js/public/public_widget";

export const WebsiteSlidesEnroll = publicWidget.Widget.extend({
    selector: "#wrapwrap",
    events: {
        "click .o_wslides_js_channel_enroll": "_onSendRequestClick",
    },
    init() {
        this._super(...arguments);
        this.orm = this.bindService("orm");
    },
    async _onSendRequestClick(ev) {
        ev.preventDefault();
        const clickedEl = ev.currentTarget;
        const channelId = parseInt(clickedEl.dataset.channelId);
        await new Promise((resolve) =>
            this.call("dialog", "add", ConfirmationDialog, {
                confirm: resolve,
                title: _t("Request Access."),
                body: _t("Do you want to request access to this course?"),
                confirmLabel: _t("Yes"),
                cancel: () => {}, // show cancel button
            })
        );
        const { error, done } = await this.orm.call(
            "slide.channel",
            "action_request_access",
            [channelId],
        );
        const $alert = $(clickedEl.closest(".alert"));
        const message = done ? _t("Request sent!") : error || _t("Unknown error, try again.");
        $alert.replaceWith(`
            <div class="alert alert-${done ? "success" : "danger"}" role="alert">
                <strong>${escape(message)}</strong>
            </div>`);
    },
});

publicWidget.registry.WebsiteSlidesEnroll = WebsiteSlidesEnroll;

```

## File: static\src\js\slides_course_fullscreen_player.js

```javascript
/** @odoo-module **/

/* global YT, Vimeo */

    import publicWidget from '@web/legacy/js/public/public_widget';
    import { renderToElement } from "@web/core/utils/render";
    import { session } from "@web/session";
    import { Quiz } from '@website_slides/js/slides_course_quiz';
    import { SlideCoursePage } from '@website_slides/js/slides_course_page';
    import { unhideConditionalElements } from '@website/js/content/inject_dom';
    import { SlideShareDialog } from './public/components/slide_share_dialog/slide_share_dialog';
    import '@website_slides/js/slides_course_join';
    import { SIZES, utils as uiUtils } from "@web/core/ui/ui_service";
    import { rpc } from "@web/core/network/rpc";

    import { markup } from "@odoo/owl";

    /**
     * Helper: Get the slide dict matching the given criteria
     *
     * @private
     * @param {Array<Object>} slideList List of dict reprensenting a slide
     * @param {[string] : any} matcher
     */
    var findSlide = function (slideList, matcher) {
        return slideList.find((slide) => {
            return Object.keys(matcher).every((key) => matcher[key] === slide[key]);
        });
    };

    /**
     * This widget is responsible of display Youtube Player
     *
     * The widget will trigger an event `change_slide` when the video is at
     * its end, and `slide_completed` when the player is at 30 sec before the
     * end of the video (30 sec before is considered as completed).
     */
    var VideoPlayerYouTube = publicWidget.Widget.extend({
        template: 'website.slides.fullscreen.video.youtube',
        youtubeUrl: 'https://www.youtube.com/iframe_api',

        init: function (parent, slide) {
            this.slide = slide;
            return this._super.apply(this, arguments);
        },
        start: function (){
            var self = this;
            return Promise.all([this._super.apply(this, arguments), this._loadYoutubeAPI()]).then(function() {
                self._setupYoutubePlayer();
            });
        },
        _loadYoutubeAPI: function () {
            var self = this;
            var prom = new Promise(function (resolve, reject) {
                if ($(document).find('script[src="' + self.youtubeUrl + '"]').length === 0) {
                    var $youtubeElement = $('<script/>', {src: self.youtubeUrl});
                    $(document.head).append($youtubeElement);

                    // function called when the Youtube asset is loaded
                    // see https://developers.google.com/youtube/iframe_api_reference#Requirements
                    window.onYouTubeIframeAPIReady = function () {
                        resolve();
                    };
                } else {
                    resolve();
                }
            });
            return prom;
        },
        /**
         * Links the youtube api to the iframe present in the template
         *
         * @private
         */
        _setupYoutubePlayer: function (){
            this.player = new YT.Player('youtube-player' + this.slide.id, {
                playerVars: {
                    'autoplay': 1,
                    'origin': window.location.origin
                },
                events: {
                    'onStateChange': this._onPlayerStateChange.bind(this)
                }
            });
        },
        /**
         * Specific method of the youtube api.
         * Whenever the player starts playing/pausing/buffering/..., a setinterval is created.
         * This setinterval is used to check te user's progress in the video.
         * Once the user reaches a particular time in the video (30s before end), the slide will be considered as completed
         * if the video doesn't have a mini-quiz.
         * This method also allows to automatically go to the next slide (or the quiz associated to the current
         * video) once the video is over
         *
         * @private
         * @param {*} event
         */
        _onPlayerStateChange: function (event){
            var self = this;

            if (self.slide.completed) {
                return;
            }

            if (event.data !== YT.PlayerState.ENDED) {
                if (!event.target.getCurrentTime) {
                    return;
                }

                if (self.tid) {
                    clearInterval(self.tid);
                }

                self.currentVideoTime = event.target.getCurrentTime();
                self.totalVideoTime = event.target.getDuration();
                self.tid = setInterval(function (){
                    self.currentVideoTime += 1;
                    if (self.totalVideoTime && self.currentVideoTime > self.totalVideoTime - 30){
                        clearInterval(self.tid);
                        if (self.slide.isMember && !self.slide.hasQuestion && !self.slide.completed){
                            self.trigger_up('slide_mark_completed', self.slide);
                        }
                    }
                }, 1000);
            } else {
                if (self.tid) {
                    clearInterval(self.tid);
                }
                this.player = undefined;
                if (this.slide.hasNext) {
                    this.trigger_up('slide_go_next');
                }
            }
        },
    });

    /**
     * This widget is responsible of loading the Vimeo video.
     *
     * Similarly to the YouTube implementation, the widget will trigger an event `change_slide` when
     * the video is at its end, and `slide_completed` when the player is at 30 sec before the end of
     * the video (30 sec before is considered as completed).
     *
     * See https://developer.vimeo.com/player/sdk/reference for all the API documentation.
     */
    var VideoPlayerVimeo = publicWidget.Widget.extend({
        template: 'website.slides.fullscreen.video.vimeo',
        vimeoScriptUrl: 'https://player.vimeo.com/api/player.js',

        init: function (parent, slide) {
            this.slide = slide;
            return this._super.apply(this, arguments);
        },

        /**
         * Loads the Vimeo JS API that allows interfacing with the iframe viewer.
         * (We only load the API if not already loaded).
         *
         * @returns {Promise}
         */
        willStart: function () {
            var self = this;
            var vimeoAPIPromise = new Promise(function (resolve, reject) {
                if ($(document).find('script[src="' + self.vimeoScriptUrl + '"]').length === 0) {
                    $.ajax({
                        url: self.vimeoScriptUrl,
                        dataType: 'script',
                        success: function () {resolve();}
                    });
                } else {
                    resolve();
                }
            });

            return Promise.all([this._super.apply(this, arguments), vimeoAPIPromise]);
        },

        start: function () {
            return this._super.apply(arguments).then(this._setupVideoPlayer.bind(this));
        },

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        /**
         * Instantiate the Vimeo player and register the various events.
         */
        _setupVideoPlayer: async function () {
            this.player = new Vimeo.Player(this.$('iframe')[0]);
            this.videoDuration = await this.player.getDuration();
            this.player.on('timeupdate', this._onVideoTimeUpdate.bind(this));
            this.player.on('ended', this._onVideoEnded.bind(this));
        },

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------

        /**
         * When the player triggers the 'ended' event, we go to the next slide if there is one.
         *
         * See https://developer.vimeo.com/player/sdk/reference#ended for more information
         */
        _onVideoEnded: function () {
            if (this.slide.hasNext) {
                this.trigger_up('slide_go_next', this.slide);
            }
        },

        /**
         * Every time the video changes position, both while viewing and also when seeking manually,
         * Vimeo triggers this handy 'timeupdate' event.
         * We use it to set the slide as completed as soon as we reach the end (30 last seconds).
         *
         * See https://developer.vimeo.com/player/sdk/reference#timeupdate for more information
         *
         * @param {Object} eventData the 'timeupdate' event data
         */
         _onVideoTimeUpdate: async function (eventData) {
            if (eventData.seconds > (this.videoDuration - 30)) {
                if (this.slide.isMember && !this.slide.hasQuestion && !this.slide.completed){
                    this.trigger_up('slide_mark_completed', this.slide);
                }
            }
        }
    });


    /**
     * This widget is responsible of navigation for one slide to another:
     *  - by clicking on any slide list entry
     *  - by mouse click (next / prev)
     *  - by recieving the order to go to prev/next slide (`goPrevious` and `goNext` public methods)
     *
     * The widget will trigger an event `change_slide` with
     * the `slideId` and `isMiniQuiz` as data.
     */
    var Sidebar = publicWidget.Widget.extend({
        events: {
            'click .o_wslides_fs_sidebar_list_item .o_wslides_fs_slide_name': '_onClickTab',
        },
        init: function (parent, slideList, defaultSlide) {
            var result = this._super.apply(this, arguments);
            this.slideEntries = slideList;
            this._slideEntry = defaultSlide;
            return result;
        },
        start: function () {
            var self = this;
            return this._super.apply(this, arguments).then(function () {
                $(document).keydown(self._onKeyDown.bind(self));
            });
        },
        destroy: function () {
            $(document).unbind('keydown', this._onKeyDown.bind(this));
            return this._super.apply(this, arguments);
        },
        //--------------------------------------------------------------------------
        // Public
        //--------------------------------------------------------------------------
        /**
         * Change the current slide with the next one (if there is one).
         *
         * @public
         */
        goNext: function () {
            var currentIndex = this._getCurrentIndex();
            if (currentIndex < this.slideEntries.length-1) {
                this._updateSlideEntry(this.slideEntries[currentIndex + 1]);
            }
        },
        /**
         * Change the current slide with the previous one (if there is one).
         *
         * @public
         */
        goPrevious: function () {
            var currentIndex = this._getCurrentIndex();
            if (currentIndex >= 1) {
                this._updateSlideEntry(this.slideEntries[currentIndex - 1]);
            }
        },

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------
        /**
         * Get the index of the current slide entry (slide and/or quiz)
         */
        _getCurrentIndex: function () {
            const slide = this._slideEntry;
            var currentIndex = this.slideEntries.findIndex(entry =>{
                return entry.id === slide.id && entry.isQuiz === slide.isQuiz;
            });
            return currentIndex;
        },
        //--------------------------------------------------------------------------
        // Handler
        //--------------------------------------------------------------------------
        /**
         * Handler called whenever the user clicks on a sub-quiz which is linked to a slide.
         * This does NOT handle the case of a slide of category "quiz".
         * By going through this handler, the widget will be able to determine that it has to render
         * the associated quiz and not the main content.
         *
         * @private
         * @param {*} ev
         */
        _onClickMiniQuiz: function (ev) {
            var slideID = parseInt($(ev.currentTarget).data().slide_id);
            this._updateSlideEntry({
                slideID: slideID,
                isMiniQuiz: true
            });
            this.trigger_up('change_slide', this._slideEntry);
        },
        /**
         * Handler called when the user clicks on a normal slide tab
         *
         * @private
         * @param {*} ev
         */
        _onClickTab: function (ev) {
            ev.stopPropagation();
            const $elem = $(ev.currentTarget).closest('.o_wslides_fs_sidebar_list_item');
            if ($elem.data('canAccess') === 'True') {
                var isQuiz = $elem.data('isQuiz');
                var slideID = parseInt($elem.data('id'));
                var slide = findSlide(this.slideEntries, {id: slideID, isQuiz: isQuiz});
                this._updateSlideEntry(slide);
            }
        },
        /**
         * Actively changes the active tab in the sidebar so that it corresponds
         * the slide currently displayed
         *
         * @private
         * @param {Object} slide
         */
        _updateSlideEntry: function (slide) {
            if (this._slideEntry === slide) {
                return;
            }
            this._slideEntry = slide;
            this.$('.o_wslides_fs_sidebar_list_item.active').removeClass('active');
            var selector = '.o_wslides_fs_sidebar_list_item[data-id='+slide.id+'][data-is-quiz!="1"]';

            this.$(selector).addClass('active');
            this.trigger_up('change_slide', this._slideEntry);
        },

        /**
         * Binds left and right arrow to allow the user to navigate between slides
         *
         * @param {*} ev
         * @private
         */
        _onKeyDown: function (ev){
            switch (ev.key){
                case "ArrowLeft":
                    this.goPrevious();
                    break;
                case "ArrowRight":
                    this.goNext();
                    break;
            }
        },
    });

    /**
     * This widget's purpose is to show content of a course, naviguating through contents
     * and correclty display it. It also handle slide completion, course progress, ...
     *
     * This widget is rendered sever side, and attached to the existing DOM.
     */
    var Fullscreen = SlideCoursePage.extend({
        events: Object.assign({}, SlideCoursePage.prototype.events, {
            'click .o_wslides_fs_toggle_sidebar': '_onClickToggleSidebar',
            'click .o_wslides_fs_share': '_onClickShareSlide',
        }),
        custom_events: Object.assign({}, SlideCoursePage.prototype.custom_events, {
            'change_slide': '_onChangeSlideRequest',
            'slide_go_next': '_onSlideGoToNext',
        }),
        /**
        * @override
        * @param {Object} el
        * @param {Object} slides Contains the list of all slides of the course
        * @param {integer} defaultSlideId Contains the ID of the slide requested by the user
        */
        init: function (parent, slides, defaultSlideId, channelData) {
            var result = this._super.apply(this,arguments);
            this.initialSlideID = defaultSlideId;
            this.slides = this._preprocessSlideData(slides);
            this.channel = channelData;
            var slide;
            const urlParams = new URL(window.location).searchParams;
            if (defaultSlideId) {
                slide = findSlide(this.slides, {id: defaultSlideId, isQuiz: String(urlParams.get("quiz")) === "1" });
            } else {
                slide = this.slides[0];
            }

            this._slideValue = slide;

            this.sidebar = new Sidebar(this, this.slides, slide);
            return result;
        },
        /**
         * @override
         */
        start: function () {
            var self = this;
            this._toggleSidebar();
            const backendNavEl = document.querySelector('.o_frontend_to_backend_nav');
            if (backendNavEl) {
                backendNavEl.remove();
            }
            return this._super.apply(this, arguments).then(function () {
                return self._onChangeSlide(); // trigger manually once DOM ready, since slide content is not rendered server side
            });
        },
        /**
         * Extended to attach sub widget to sub DOM. This might be experimental but
         * seems working fine.
         *
         * @override
         */
        attachTo: function (){
            var defs = [this._super.apply(this, arguments)];
            defs.push(this.sidebar.attachTo(this.$('.o_wslides_fs_sidebar')));
            return $.when.apply($, defs);
        },
        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------
        /**
         * Fetches content with an rpc call for slides of category "article"
         *
         * @private
         */
        _fetchHtmlContent: function () {
            const currentSlide = this._slideValue;
            return rpc("/slides/slide/get_html_content", {
                'slide_id': currentSlide.id
            }).then(function (data){
                if (data.html_content) {
                    currentSlide.htmlContent = data.html_content;
                }
            });
        },
        /**
        * Fetches slide content depending on its category.
        * If the slide doesn't need to fetch any content, return a resolved deferred
        *
        * @private
        */
        _fetchSlideContent: function () {
            const slide = this._slideValue;
            if (slide.category === 'article' && !slide.isQuiz) {
                return this._fetchHtmlContent();
            }
            return Promise.resolve();
        },
        getDocumentMaxPage() {
            const iframe = document.querySelector("iframe.o_wslides_iframe_viewer");
            const iframeDocument = iframe.contentWindow.document;
            return parseInt(iframeDocument.querySelector("#page_count").innerText);
        },
        /**
         * Extend the slide data list to add informations about rendering method, and other
         * specific values according to their slide_category.
         */
        _preprocessSlideData: function (slidesDataList) {
            slidesDataList.forEach(function (slideData, index) {
                // compute hasNext slide
                slideData.hasNext = index < slidesDataList.length-1;
                // compute embed url
                if (slideData.category === 'video' && slideData.videoSourceType !== 'vimeo') {
                    slideData.embedCode = $(slideData.embedCode).attr('src') || ""; // embedCode contains an iframe tag, where src attribute is the url (youtube or embed document from odoo)
                    var separator = slideData.embedCode.indexOf("?") !== -1 ? "&" : "?";
                    var scheme = slideData.embedCode.indexOf('//') === 0 ? 'https:' : '';
                    var params = { rel: 0, enablejsapi: 1, origin: window.location.origin };
                    if (slideData.embedCode.indexOf("//drive.google.com") === -1) {
                        params.autoplay = 1;
                    }
                    slideData.embedUrl = slideData.embedCode ? scheme + slideData.embedCode + separator + $.param(params) : "";
                } else if (slideData.category === 'video' && slideData.videoSourceType === 'vimeo') {
                    slideData.embedCode = markup(slideData.embedCode);
                } else if (slideData.category === 'infographic') {
                    slideData.embedUrl = `/web/image/slide.slide/${encodeURIComponent(slideData.id)}/image_1024`;
                } else if (slideData.category === 'document') {
                    slideData.embedUrl = $(slideData.embedCode).attr('src');
                }
                // fill empty property to allow searching on it with list.filter(matcher)
                slideData.isQuiz = !!slideData.isQuiz;
                slideData.hasQuestion = !!slideData.hasQuestion;
                // technical settings for the Fullscreen to work
                var autoSetDone = false;
                if (!slideData.hasQuestion) {
                    if (['infographic', 'document', 'article'].includes(slideData.category)) {
                        autoSetDone = true;  // images, documents (local + external) and articles are marked as completed when opened
                    } else if (slideData.category === 'video' && slideData.videoSourceType === 'google_drive') {
                        autoSetDone = true;  // google drive videos do not benefit from the YouTube integration and are marked as completed when opened
                    }
                }
                slideData._autoSetDone = autoSetDone;
            });
            return slidesDataList;
        },
        /**
         * Changes the url whenever the user changes slides.
         * This allows the user to refresh the page and stay on the right slide
         *
         * @private
         */
        _pushUrlState: function () {
            var urlParts = window.location.pathname.split('/');
            urlParts[urlParts.length - 1] = this._slideValue.slug;
            var url =  urlParts.join('/');
            this.$('.o_wslides_fs_exit_fullscreen').attr('href', url);
            var params = {'fullscreen': 1 };
            if (this._slideValue.isQuiz) {
                params.quiz = 1;
            }
            var fullscreenUrl = `${url}?${$.param(params)}`;
            history.pushState(null, '', fullscreenUrl);
        },
        /**
         * Render the current slide content using specific mecanism according to slide category:
         * - simply append content (for article)
         * - template rendering (for image, document, ....)
         * - using a sub widget (quiz and video)
         *
         * @private
         * @returns Deferred
         */
        _renderSlide: async function () {
            // Avoid concurrent execution of the slide rendering as it writes the content at the same place anyway.
            if (this._renderSlideRunning) { return; }
            this._renderSlideRunning = true;
            try {
                const slide = this._slideValue;
                var $content = this.$('.o_wslides_fs_content');
                $content.empty();
                if (this.websiteAnimateWidget) {
                    this.websiteAnimateWidget.destroy()
                    this.websiteAnimateWidget = null;
                }

                // display quiz slide, or quiz attached to a slide
                if (slide.category === 'quiz' || slide.isQuiz) {
                    $content.addClass('bg-white');
                    var QuizWidget = new Quiz(this, slide, this.channel);
                    return await QuizWidget.appendTo($content);
                }

                // render slide content
                if (['document', 'infographic'].includes(slide.category)) {
                    $content.empty().append(renderToElement('website.slides.fullscreen.content', {widget: this}));
                } else if (slide.category === 'video' && slide.videoSourceType === 'youtube') {
                    this.videoPlayer = new VideoPlayerYouTube(this, slide);
                    return await this.videoPlayer.appendTo($content);
                } else if (slide.category === 'video' && slide.videoSourceType === 'vimeo') {
                    this.videoPlayer = new VideoPlayerVimeo(this, slide);
                    return await this.videoPlayer.appendTo($content);
                } else if (slide.category === 'video' && slide.videoSourceType === 'google_drive') {
                    $content.empty().append(renderToElement('website.slides.fullscreen.video.google_drive', {widget: this}));
                } else if (slide.category === 'article'){
                    this.websiteAnimateWidget = new publicWidget.registry.WebsiteAnimate();
                    var $wpContainer = $('<div>').addClass('o_wslide_fs_article_content bg-white block w-100 overflow-auto p-3');
                    $wpContainer.html(slide.htmlContent);
                    $content.append($wpContainer);
                    this.trigger_up('widgets_start_request', {
                        $target: $content,
                    });
                    this.websiteAnimateWidget.attachTo($wpContainer);
                }
                unhideConditionalElements();
            } finally {
                this._renderSlideRunning = false;
            }
        },
        /**
         * @private
         */
        _updateSlideValue: function (slide) {
            if (this._slideValue === slide) {
                return;
            }
            this._slideValue = slide;
            this._onChangeSlide();
        },

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------
        /**
         * Triggered whenever the user changes slides.
         * When the current slide is changed, widget will be automatically updated
         * and allowed to: fetch the content if needed, render it, update the url,
         * and set slide as "completed" according to its category requirements. In
         * mobile case (i.e. limited screensize), sidebar will be toggled since
         * sidebar will block most or all of new slide visibility.
         *
         * @private
         */
        _onChangeSlide: function () {
            var self = this;
            const slide = this._slideValue;
            self._pushUrlState();
            return this._fetchSlideContent().then(function() { // render content
                var websiteName = document.title.split(" | ")[1]; // get the website name from title
                document.title =  (websiteName) ? slide.name + ' | ' + websiteName : slide.name;
                if  (uiUtils.getSize() < SIZES.MD) {
                    self._toggleSidebar(); // hide sidebar when small device screen
                }
                return self._renderSlide();
            }).then(function() {
                if (slide._autoSetDone && !session.is_website_user) {  // no useless RPC call
                    if (slide.category === 'document') {
                        // only set the slide as completed after iFrame is loaded to avoid concurrent execution with 'embedUrl' controller
                        self.el.querySelector('iframe.o_wslides_iframe_viewer').addEventListener('load', () => self._toggleSlideCompleted(slide));
                    } else {
                           return self._toggleSlideCompleted(slide);
                    }
                }
            });
        },
        /**
         * Changes current slide when receiving custom event `change_slide` with
         * its id and if it's its quizz or not we need to display.
         *
         * @private
         */
        _onChangeSlideRequest: function (ev) {
            var slideData = ev.data;
            var newSlide = findSlide(this.slides, {
                id: slideData.id,
                isQuiz: slideData.isQuiz || false,
            });
            this._updateSlideValue(newSlide);
        },
        /**
         * After a slide has been marked as completed / uncompleted, update the state
         * of this widget and reload the slide if needed (e.g. to re-show the questions
         * of a quiz).
         *
         * We might need to set multiple slide as completed, because of "isQuiz"
         * set to True / False
         *
         * @private
         * @param {Object} slide: slide to set as completed
         * @param {Boolean} completed: true to mark the slide as completed
         *     false to mark the slide as not completed
         */
        _toggleSlideCompleted: async function (slide, completed = true) {
            await this._super(...arguments);

            const fsSlides = this.slides.filter(_slide => _slide.id === slide.id);

            fsSlides.forEach(slide => slide.completed = completed);

            const currentSlide = this._slideValue;
            if (currentSlide.id === slide.id) {
                currentSlide.completed = completed;
                this._updateSlideValue(currentSlide);

                if ((currentSlide.hasQuestion || currentSlide.type === 'quiz') && !completed) {
                    // Reload the quiz
                    this._renderSlide();
                }
            }
        },
        /**
         * Go the next slide
         *
         * @private
         */
        _onSlideGoToNext: function (ev) {
            this.sidebar.goNext();
        },
        /**
         * Called when the sidebar toggle is clicked -> toggles the sidebar visibility.
         *
         * @private
         */
        _onClickToggleSidebar: function (ev){
            ev.preventDefault();
            this._toggleSidebar();
        },

        _onClickShareSlide: function (ev) {
            const slide = this._slideValue;
            this.call("dialog", "add", SlideShareDialog, {
                category: slide.category,
                documentMaxPage: slide.category == 'document' && this.getDocumentMaxPage(),
                emailSharing: slide.emailSharing === 'True',
                embedCode: slide.embedCode || '',
                id: slide.id,
                isFullscreen: true,
                name: slide.name,
                url: slide.websiteShareUrl,
            });
        },

        /**
         * Toggles sidebar visibility.
         *
         * @private
         */
        _toggleSidebar: function () {
            this.$('.o_wslides_fs_sidebar').toggleClass('o_wslides_fs_sidebar_hidden');
            this.$('.o_wslides_fs_toggle_sidebar').toggleClass('active');
        },
    });

    publicWidget.registry.websiteSlidesFullscreenPlayer = publicWidget.Widget.extend({
        selector: '.o_wslides_fs_main',
        start: function (){
            var proms = [this._super.apply(this, arguments)];
            var fullscreen = new Fullscreen(this, this._getSlides(), this._getCurrentSlideID(), this._extractChannelData());
            proms.push(fullscreen.attachTo(".o_wslides_fs_main"));
            // To prevent double scrollbar due to footer overflow
            document.querySelector('.o_footer')?.classList.add('d-none');
            return proms;
        },
        _extractChannelData: function (){
            return this.$el.data();
        },
        _getCurrentSlideID: function (){
            return parseInt(this.$('.o_wslides_fs_sidebar_list_item.active').data('id'));
        },
        /**
         * @private
         * Creates slides objects from every slide-list-cells attributes
         */
        _getSlides: function (){
            var $slides = this.$('.o_wslides_fs_sidebar_list_item[data-can-access="True"]');
            var slideList = [];
            $slides.each(function () {
                var slideData = $(this).data();
                slideList.push(slideData);
            });
            return slideList;
        },
    });

    export default Fullscreen;

```

## File: static\src\js\slides_course_join.js

```javascript
/** @odoo-module **/

import { sprintf } from '@web/core/utils/strings';
import { renderToElement } from "@web/core/utils/render";
import publicWidget from '@web/legacy/js/public/public_widget';
import { _t } from "@web/core/l10n/translation";
import { rpc } from "@web/core/network/rpc";

var CourseJoinWidget = publicWidget.Widget.extend({
    template: 'slide.course.join',
    events: {
        'click .o_wslides_js_course_join_link': '_onClickJoin',
    },

    /**
     *
     * Overridden to add options parameters.
     *
     * @param {Object} parent
     * @param {Object} options
     * @param {Object} options.channel slide.channel information
     * @param {boolean} options.isMember whether current user is enrolled
     * @param {boolean} options.isMemberOrInvited whether current user is at least invited
     * @param {string} options.inviteHash hash of the invited attendee. Needed to grant
     *   access to a course preview / to identify.
     * @param {integer} options.invitePartnerId id of partner of invited attendee if any.
     *   Also needed to access course preview / to identify.
     * @param {boolean} options.invitePreview whether the course is rendered as a preview.
     *   This is true when an invited attendee is on the course while unlogged.
     * @param {boolean} options.isPartnerWithoutUser whether invited partner has users. Used
     *   to redirect properly to sign up / log in.
     * @param {string} [options.joinMessage] the message to use for the simple join case
     *   when the course is free and the user is logged in, defaults to "Join this Course".
     * @param {Promise} [options.beforeJoin] a promise to execute before we redirect to
     *   another url within the join process (login / buy course / ...)
     * @param {function} [options.afterJoin] a callback function called after the user has
     *   joined the course
     */
    init: function (parent, options) {
        this._super.apply(this, arguments);
        this.channel = options.channel;
        this.isMember = options.isMember;
        this.isMemberOrInvited = options.isMemberOrInvited;
        this.inviteHash = options.inviteHash;
        this.invitePartnerId = options.invitePartnerId;
        this.invitePreview = options.invitePreview;
        this.isPartnerWithoutUser = options.isPartnerWithoutUser;
        this.publicUser = options.publicUser;
        this.joinMessage = options.joinMessage || _t('Join this Course');
        this.beforeJoin = options.beforeJoin || function () {return Promise.resolve();};
        this.afterJoin = options.afterJoin || function () {document.location.reload();};
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickJoin: function (ev) {
        ev.preventDefault();

        if (this.invitePreview || (this.channel.channelEnroll === 'invite' && this.isMemberOrInvited)) {
            this.joinChannel(this.channel.channelId);
            return;
        }

        if (this.channel.channelEnroll !== 'invite') {
            if (this.publicUser) {
                this.beforeJoin().then(this._redirectToLogin.bind(this));
            } else if (!this.isMember) {
                this.joinChannel(this.channel.channelId);
            }
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Builds a login page that then redirects to this slide page, or the channel if the course
     * is not configured as public enroll type.
     *
     * @private
     */
    _redirectToLogin: function () {
        var url;
        if (this.channel.channelEnroll === 'public') {
            url = window.location.pathname;
            if (document.location.href.indexOf("fullscreen") !== -1) {
                url += '?fullscreen=1';
            }
        } else {
            url = `/slides/${encodeURIComponent(this.channel.channelId)}`;
        }
        document.location = sprintf('/web/login?redirect=%s', encodeURIComponent(url));
    },

    /**
     * @private
     * @param {Object} $el
     * @param {String} message
     */
    _popoverAlert: function ($el, message) {
        $el.popover({
            trigger: 'focus',
            delay: {'hide': 300},
            placement: 'bottom',
            container: 'body',
            html: true,
            content: function () {
                return message;
            }
        }).popover('show');
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------
    /**
     * @public
     * @param {integer} channelId
     */
    joinChannel: function (channelId) {
        var self = this;
        rpc('/slides/channel/join', {
            channel_id: channelId,
        }).then(function (data) {
            if (!data.error) {
                self.afterJoin();
            } else {
                if (data.error === 'public_user') {
                    const popupContent = renderToElement('slide.course.join.popupContent', {
                        channelId: channelId,
                        courseUrl: encodeURIComponent(document.URL),
                        errorSignupAllowed: data.error_signup_allowed,
                        widget: self,
                    });
                    self._popoverAlert(self.$el, popupContent);
                } else if (data.error === 'join_done') {
                    self._popoverAlert(self.$el, _t('You have already joined this channel'));
                } else {
                    self._popoverAlert(self.$el, _t('Unknown error'));
                }
            }
        });
    },
});

publicWidget.registry.websiteSlidesCourseJoin = publicWidget.Widget.extend({
    selector: '.o_wslides_js_course_join_link',

    /**
     * @override
     * @param {Object} parent
     */
    start: function () {
        var self = this;
        var proms = [this._super.apply(this, arguments)];
        var data = self.$el.data();
        var options = {
            channel: {
                channelEnroll: data.channelEnroll,
                channelId: data.channelId
            },
            inviteHash: data.inviteHash,
            invitePartnerId: data.invitePartnerId,
            invitePreview: data.invitePreview,
            isMemberOrInvited: data.isMemberOrInvited,
            isPartnerWithoutUser: data.isPartnerWithoutUser
        };
        $('.o_wslides_js_course_join').each(function () {
            proms.push(new CourseJoinWidget(self, options).attachTo($(this)));
        });
        return Promise.all(proms);
    },
});

export default {
    courseJoinWidget: CourseJoinWidget,
    websiteSlidesCourseJoin: publicWidget.registry.websiteSlidesCourseJoin
};

```

## File: static\src\js\slides_course_page.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';
import { session } from "@web/session";
import { renderToElement } from "@web/core/utils/render";
import { rpc } from "@web/core/network/rpc";

/**
 * Global widget for both fullscreen view and non-fullscreen view of a slide course.
 * Contains general methods to update the UI elements (progress bar, sidebar...) as well
 * as method to mark the slide as completed / uncompleted.
 */
export const SlideCoursePage = publicWidget.Widget.extend({
    events: {
        'click button.o_wslides_button_complete': '_onClickComplete',
    },

    custom_events: {
        'slide_completed': '_onSlideCompleted',
        'slide_mark_completed': '_onSlideMarkCompleted',
    },

    /**
     * Collapse the next category when the current one has just been completed
     */
    collapseNextCategory: function (nextCategoryId) {
        const categorySection = document.getElementById(`category-collapse-${nextCategoryId}`);
        if (categorySection?.getAttribute('aria-expanded') === 'false') {
            categorySection.setAttribute('aria-expanded', true);
            document.querySelector(`ul[id=collapse-${nextCategoryId}]`).classList.add('show');
        }
    },

    /**
     * Greens up the bullet when the slide is completed
     *
     * @public
     * @param {Object} slide
     * @param {Boolean} completed
     */
    toggleCompletionButton: function (slide, completed = true) {
        const $button = this.$(`.o_wslides_sidebar_done_button[data-id="${slide.id}"]`);

        if (!$button.length) {
            return;
        }

        const newButton = renderToElement('website.slides.sidebar.done.button', {
            slideId: slide.id,
            uncompletedIcon: $button.data('uncompletedIcon') ?? 'fa-circle-thin',
            slideCompleted: completed ? 1 : 0,
            canSelfMarkUncompleted: slide.canSelfMarkUncompleted,
            canSelfMarkCompleted: slide.canSelfMarkCompleted,
            isMember: slide.isMember,
        });
        $button.replaceWith(newButton);
    },

    /**
     * Updates the progressbar whenever a lesson is completed
     *
     * @public
     * @param {Integer} channelCompletion
     */
    updateProgressbar: function (channelCompletion) {
        const completion = Math.min(100, channelCompletion);

        const $completed = $('.o_wslides_channel_completion_completed');
        const $progressbar = $('.o_wslides_channel_completion_progressbar');

        if (completion < 100) {
            // Hide the "Completed" text and show the progress bar
            $completed.addClass('d-none');
            $progressbar.removeClass('d-none').addClass('d-flex');
        } else {
            // Hide the progress bar and show the "Completed" text
            $completed.removeClass('d-none');
            $progressbar.addClass('d-none').removeClass('d-flex');
        }

        $progressbar.find('.progress-bar').css('width', `${completion}%`);
        $progressbar.find('.o_wslides_progress_percentage').text(completion);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Once the completion conditions are filled,
     * rpc call to set the relation between the slide and the user as "completed"
     *
     * @private
     * @param {Object} slide: slide to set as completed
     * @param {Boolean} completed: true to mark the slide as completed
     *     false to mark the slide as not completed
     */
    _toggleSlideCompleted: async function (slide, completed = true) {
        if (!!slide.completed === !!completed || !slide.isMember || !slide.canSelfMarkCompleted) {
            // no useless RPC call
            return;
        }

        const data = await rpc(
            `/slides/slide/${completed ? 'set_completed' : 'set_uncompleted'}`,
            {slide_id: slide.id},
        );

        this.toggleCompletionButton(slide, completed);
        this.updateProgressbar(data.channel_completion);
        if (data.next_category_id) {
            this.collapseNextCategory(data.next_category_id);
        }
    },
    /**
     * Retrieve the slide data corresponding to the slide id given in argument.
     * This method used the "slide_sidebar_done_button" template.
     *
     * @private
     * @param {Integer} slideId
     */
    _getSlide: function (slideId) {
        return $(`.o_wslides_sidebar_done_button[data-id="${slideId}"]`).data();
    },

    //--------------------------------------------------------------------------
    // Handler
    //--------------------------------------------------------------------------
    /**
     * We clicked on the "done" button.
     * It will make a RPC call to update the slide state and update the UI.
     *
     * @private
     * @param {Event} ev
     */
    _onClickComplete: function (ev) {
        ev.stopPropagation();
        ev.preventDefault();

        const $button = $(ev.currentTarget).closest('.o_wslides_sidebar_done_button');

        const slideData = $button.data();
        const isCompleted = Boolean(slideData.completed);

        this._toggleSlideCompleted(slideData, !isCompleted);
    },

    /**
     * The slide has been completed, update the UI
     *
     * @private
     * @param {Event} ev
     */
    _onSlideCompleted: function (ev) {
        const slideId = ev.data.slideId;
        const completed = ev.data.completed;
        const slide = this._getSlide(slideId);
        if (slide) {
            // Just joined the course (e.g. When "Submit & Join" action), update the UI
            this.toggleCompletionButton(slide, completed);
        }
        this.updateProgressbar(ev.data.channelCompletion);
    },

    /**
     * Make a RPC call to complete the slide then update the UI
     *
     * @private
     * @param {Event} ev
     */
    _onSlideMarkCompleted: function (ev) {
        if (!session.is_website_user) { // no useless RPC call
            const slide = this._getSlide(ev.data.id);
            this._toggleSlideCompleted(slide, true);
        }
    }
});

```

## File: static\src\js\slides_course_prerequisite.js

```javascript
/** @odoo-module **/

import { renderToElement } from "@web/core/utils/render";
import publicWidget from '@web/legacy/js/public/public_widget';

publicWidget.registry.websiteSlidesCoursePrerequisite = publicWidget.Widget.extend({
    selector: '.o_wslides_js_prerequisite_course',

    async start() {
        await this._super(...arguments);
        const channels = this.$el.data('channels');
        this.$el.popover({
            trigger: 'focus',
            placement: 'bottom',
            container: 'body',
            html: true,
            content: renderToElement('slide.course.prerequisite', {channels: channels}),
        });
    },
});

export default {
    websiteSlidesCoursePrerequisite: publicWidget.registry.websiteSlidesCoursePrerequisite
};

```

## File: static\src\js\slides_course_quiz.js

```javascript
/** @odoo-module **/

    import publicWidget from '@web/legacy/js/public/public_widget';
    import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
    import { renderToElement } from "@web/core/utils/render";
    import { escape } from "@web/core/utils/strings";
    import { session } from "@web/session";
    import CourseJoin from '@website_slides/js/slides_course_join';
    import QuestionFormWidget from '@website_slides/js/slides_course_quiz_question_form';
    import { SlideCoursePage } from '@website_slides/js/slides_course_page';
    import { rpc } from "@web/core/network/rpc";
    import { SlideQuizFinishDialog } from "@website_slides/js/public/components/slide_quiz_finish_dialog/slide_quiz_finish_dialog";
    import { user } from "@web/core/user";

    import { _t } from "@web/core/l10n/translation";

    import { markup } from "@odoo/owl";

    const CourseJoinWidget = CourseJoin.courseJoinWidget;

    /**
     * This widget is responsible of displaying quiz questions and propositions. Submitting the quiz will fetch the
     * correction and decorate the answers according to the result. Error message or modal can be displayed.
     *
     * This widget can be attached to DOM rendered server-side by `website_slides.slide_category_quiz` or
     * used client side (Fullscreen).
     *
     * Triggered events are :
     * - slide_go_next: need to go to the next slide, when quiz is done. Event data contains the current slide id.
     * - quiz_completed: when the quiz is passed and completed by the user. Event data contains current slide data.
     */
    var Quiz = publicWidget.Widget.extend({
        template: 'slide.slide.quiz',
        events: {
            "click .o_wslides_quiz_answer": '_onAnswerClick',
            "click .o_wslides_js_lesson_quiz_submit": '_submitQuiz',
            "click .o_wslides_quiz_continue": '_onClickNext',
            "click .o_wslides_js_lesson_quiz_reset": '_onClickReset',
            'click .o_wslides_js_quiz_add': '_onCreateQuizClick',
            'click .o_wslides_js_quiz_edit_question': '_onEditQuestionClick',
            'click .o_wslides_js_quiz_delete_question': '_onDeleteQuestionClick',
        },

        custom_events: {
            display_created_question: '_displayCreatedQuestion',
            display_updated_question: '_displayUpdatedQuestion',
            reset_display: '_resetDisplay',
            delete_question: '_deleteQuestion',
        },

        /**
        * @override
        * @param {Object} parent
        * @param {Object} slide_data holding all the classic slide information
        * @param {Object} quiz_data : optional quiz data to display. If not given, will be fetched. (questions and answers).
        */
        init: function (parent, slide_data, channel_data, quiz_data) {
            this._super.apply(this, arguments);
            this.slide = Object.assign({
                id: 0,
                name: '',
                hasNext: false,
                completed: false,
                isMember: false,
                isMemberOrInvited: false,
            }, slide_data);
            this.quiz = quiz_data || false;
            if (this.quiz) {
                this.quiz.questionsCount = quiz_data.questions.length;
            }
            this.isMember = slide_data.isMember || false;
            this.isMemberOrInvited = slide_data.isMemberOrInvited || false;
            this.publicUser = session.is_website_user;
            this.userId = user.userId;
            this.redirectURL = encodeURIComponent(document.URL);
            this.channel = channel_data;

            this.orm = this.bindService("orm");
        },

        /**
         * @override
         */
        willStart: function () {
            var defs = [this._super.apply(this, arguments)];
            if (!this.quiz) {
                defs.push(this._fetchQuiz());
            }
            return Promise.all(defs);
        },

        /**
         * Overridden to add custom rendering behavior upon start of the widget.
         *
         * If the user has answered the quiz before having joined the course, we check
         * their answers (saved into their session) here as well.
         *
         * @override
         */
        start: function () {
            var self = this;
            return this._super.apply(this, arguments).then(function ()  {
                self._renderValidationInfo();
                self._bindSortable();
                self._checkLocationHref();
                if (!self.isMember) {
                    self._renderJoinWidget();
                } else if (self.slide.sessionAnswers) {
                    self._applySessionAnswers();
                    self._submitQuiz();
                }
            });
        },

        destroy() {
            this._unbindSortable();
            return this._super(...arguments);
        },

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        _showErrorMessage: function (errorCode) {
            var message = _t('There was an error validating this quiz.');
            if (errorCode === 'slide_quiz_incomplete') {
                message = _t('All questions must be answered!');
            } else if (errorCode === 'slide_quiz_done') {
                message = _t('This quiz is already done. Retaking it is not possible.');
            } else if (errorCode === 'public_user') {
                message = _t('You must be logged to submit the quiz.');
            }

            this.$('.o_wslides_js_quiz_submit_error')
                .removeClass('d-none')
                .find('.o_wslides_js_quiz_submit_error_text')
                .text(message);
        },

        _hideErrorMessage: function () {
            this.$('.o_wslides_js_quiz_submit_error')
                .addClass('d-none');
        },

        /**
         * Allows to reorder the questions
         * @private
         */
        _bindSortable: function () {
            this.bindedSortable = this.call(
                "sortable",
                "create",
                {
                    ref: { el: this.el },
                    handle: ".o_wslides_js_quiz_sequence_handler",
                    elements: ".o_wslides_js_lesson_quiz_question",
                    onDrop: this._reorderQuestions.bind(this),
                    clone: false,
                    placeholderClasses: ['o_wslides_js_quiz_sequence_highlight', 'position-relative', 'my-3'],
                    applyChangeOnDrop: true
                },
            ).enable();
        },

        _unbindSortable: function () {
            this.bindedSortable?.cleanup();
        },

        /**
         * Get all the questions ID from the displayed Quiz
         * @returns {Array}
         * @private
         */
        _getQuestionsIds: function () {
            return this.$('.o_wslides_js_lesson_quiz_question').map(function () {
                return $(this).data('question-id');
            }).get();
        },

        /**
         * Modify visually the sequence of all the questions after
         * calling the _reorderQuestions RPC call.
         * @private
         */
        _modifyQuestionsSequence: function () {
            this.$('.o_wslides_js_lesson_quiz_question').each(function (index, question) {
                $(question).find('span.o_wslides_quiz_question_sequence').text(index + 1);
            });
        },

        /**
         * RPC call to resequence all the questions. It is called
         * after modifying the sequence of a question and also after
         * deleting a question.
         * @private
         */
        _reorderQuestions: function () {
            rpc('/web/dataset/resequence', {
                model: "slide.question",
                ids: this._getQuestionsIds()
            }).then(this._modifyQuestionsSequence.bind(this))
        },
        /*
         * @private
         * Fetch the quiz for a particular slide
         */
        _fetchQuiz: function () {
            var self = this;
            return rpc('/slides/slide/quiz/get', {
                'slide_id': self.slide.id,
            }).then(function (quiz_data) {
                self.slide.sessionAnswers = quiz_data.session_answers;
                self.quiz = {
                    description_safe: quiz_data.slide_description ? markup(quiz_data.slide_description) : '',
                    questions: quiz_data.slide_questions || [],
                    questionsCount: quiz_data.slide_questions.length,
                    quizAttemptsCount: quiz_data.quiz_attempts_count || 0,
                    quizKarmaGain: quiz_data.quiz_karma_gain || 0,
                    quizKarmaWon: quiz_data.quiz_karma_won || 0,
                    slideResources: quiz_data.slide_resource_ids || [],
                };
            });
        },

        /**
         * Hide the edit and delete button and also the handler
         * to resequence the question
         * @private
         */
        _hideEditOptions: function () {
            this.$('.o_wslides_js_lesson_quiz_question .o_wslides_js_quiz_edit_del,' +
                   ' .o_wslides_js_lesson_quiz_question .o_wslides_js_quiz_sequence_handler').addClass('d-none');
        },

        /**
         * @private
         * Decorate the answers according to state
         */
        _disableAnswers: function () {
            var self = this;
            this.$('.o_wslides_js_lesson_quiz_question').addClass('completed-disabled');
            this.$('input[type=radio]').each(function () {
                $(this).prop('disabled', self.slide.completed);
            });
        },

        /**
         * Decorate the answer inputs according to the correction and adds the answer comment if
         * any.
         *
         * @private
         */
        _renderAnswersHighlightingAndComments: function () {
            var self = this;
            this.$('.o_wslides_js_lesson_quiz_question').each(function () {
                var $question = $(this);
                var questionId = $question.data('questionId');
                var isCorrect = self.quiz.answers[questionId].is_correct;
                $question.find('a.o_wslides_quiz_answer').each(function () {
                    var $answer = $(this);
                    $answer.find('i.fa').addClass('d-none');
                    if ($answer.find('input[type=radio]')[0].checked) {
                        if (isCorrect) {
                            $answer.removeClass('list-group-item-danger').addClass('list-group-item-success');
                            $answer.find('i.fa-check-circle').removeClass('d-none');
                        } else {
                            $answer.removeClass('list-group-item-success').addClass('list-group-item-danger');
                            $answer.find('i.fa-times-circle').removeClass('d-none');
                            $answer.find('label input').prop('checked', false);
                        }
                    } else {
                        $answer.removeClass('list-group-item-danger list-group-item-success');
                        $answer.find('i.fa-circle').removeClass('d-none');
                    }
                });
                var comment = self.quiz.answers[questionId].comment;
                if (comment) {
                    $question.find('.o_wslides_quiz_answer_info').removeClass('d-none');
                    $question.find('.o_wslides_quiz_answer_comment').text(comment);
                }
            });
        },

        /**
         * Will check if we have answers coming from the session and re-apply them.
         */
        _applySessionAnswers: function () {
            if (!this.slide.sessionAnswers || this.slide.sessionAnswers.length === 0) {
                return;
            }

            var self = this;
            this.$('.o_wslides_js_lesson_quiz_question').each(function () {
                var $question = $(this);
                $question.find('a.o_wslides_quiz_answer').each(function () {
                    var $answer = $(this);
                    if (!$answer.find('input[type=radio]')[0].checked &&
                        self.slide.sessionAnswers.includes($answer.data('answerId'))) {
                        $answer.find('input[type=radio]').prop('checked', true);
                    }
                });
            });

            // reset answers coming from the session
            this.slide.sessionAnswers = false;
        },

        /*
         * @private
         * Update validation box (karma, buttons) according to widget state
         */
        _renderValidationInfo: function () {
            var $validationElem = this.$('.o_wslides_js_lesson_quiz_validation');
            $validationElem.empty().append(
                renderToElement('slide.slide.quiz.validation', {'widget': this})
            );
        },
        /*
        * Toggle additional resource info box
        *
        * @private
        * @param {Boolean} show - Whether show or hide the information
        */
        _toggleAdditionalResourceInfo: function(show) {
            const resourceInfo = document.getElementsByClassName('o_wslides_js_lesson_quiz_resource_info')[0];
            resourceInfo && (show ? resourceInfo.classList.remove('d-none') : resourceInfo.classList.add('d-none'));
        },
        /**
         * Renders the button to join a course.
         * If the user is logged in, the course is public, and the user has previously tried to
         * submit answers, we automatically attempt to join the course.
         *
         * @private
         */
        _renderJoinWidget: function () {
            var $widgetLocation = this.$(".o_wslides_join_course_widget");
            if ($widgetLocation.length !== 0) {
                var courseJoinWidget = new CourseJoinWidget(this, {
                    isQuiz: true,
                    channel: this.channel,
                    isMember: this.isMember,
                    isMemberOrInvited: this.isMemberOrInvited,
                    publicUser: this.publicUser,
                    beforeJoin: this._saveQuizAnswersToSession.bind(this),
                    afterJoin: this._afterJoin.bind(this),
                    joinMessage: _t('Join & Submit'),
                });

                courseJoinWidget.appendTo($widgetLocation);
                if (!this.publicUser && courseJoinWidget.channel.channelEnroll === 'public' && this.slide.sessionAnswers) {
                    courseJoinWidget.joinChannel(this.channel.channelId);
                }
            }
        },

        /**
         * Get the quiz answers filled in by the User
         *
         * @private
         */
        _getQuizAnswers: function () {
            return this.$('input[type=radio]:checked').map(function (index, element) {
                return parseInt($(element).val());
            }).get();
        },

        /**
         * Submit a quiz and get the correction. It will display messages
         * according to quiz result.
         *
         * @private
         */
         async _submitQuiz() {
            const data = await rpc('/slides/slide/quiz/submit', {
                slide_id: this.slide.id,
                answer_ids: this._getQuizAnswers(),
            });
            if (data.error) {
                this._showErrorMessage(data.error);
                return;
            } else {
                this._hideErrorMessage();
            }
            Object.assign(this.quiz, data);
            const {rankProgress, completed, channel_completion: completion} = this.quiz;
            // three of the rankProgress properties are HTML messages, mark if set
            if ('description' in rankProgress) {
                rankProgress['description'] = markup(rankProgress['description'] || '');
                rankProgress['previous_rank']['motivational'] =
                    markup(rankProgress['previous_rank']['motivational'] || '');
                rankProgress['new_rank']['motivational'] =
                    markup(rankProgress['new_rank']['motivational'] || '');
            }
            if (completed) {
                this._disableAnswers();
                this.call("dialog", "add", SlideQuizFinishDialog, {
                    quiz: this.quiz,
                    hasNext: this.slide.hasNext,
                    onClickNext: (ev) => this._onClickNext(ev),
                    userId: this.userId,
                });
                this.slide.completed = true;
                this.trigger_up('slide_completed', {
                    slideId: this.slide.id,
                    channelCompletion: completion,
                    completed: true,
                });
            }
            this._hideEditOptions();
            this._renderAnswersHighlightingAndComments();
            this._renderValidationInfo();
            this._toggleAdditionalResourceInfo(!completed);
        },

        /**
         * Get all the question information after clicking on
         * the edit button
         * @param $elem
         * @returns {{id: *, sequence: number, text: *, answers: Array}}
         * @private
         */
        _getQuestionDetails: function ($elem) {
            var answers = [];
            $elem.find('.o_wslides_quiz_answer').each(function () {
                answers.push({
                    'id': $(this).data('answerId'),
                    'text_value': $(this).data('text'),
                    'is_correct': $(this).data('isCorrect'),
                    'comment': $(this).data('comment')
                });
            });
            return {
                'id': $elem.data('questionId'),
                'sequence': parseInt($elem.find('.o_wslides_quiz_question_sequence').text()),
                'text': $elem.data('title'),
                'answers': answers,
            };
        },

        /**
         * If the slides has been called with the Add Quiz button on the slide list
         * it goes straight to the 'Add Quiz' button and clicks on it.
         * @private
         */
        _checkLocationHref: function () {
            if (window.location.href.includes('quiz_quick_create') && this.quiz.questionsCount === 0) {
                this._onCreateQuizClick();
            }
        },

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------

        /**
         * When clicking on an answer, this one should be marked as "checked".
         *
         * @private
         * @param OdooEvent ev
         */
        _onAnswerClick: function (ev) {
            ev.preventDefault();
            if (!this.slide.completed) {
                $(ev.currentTarget).find('input[type=radio]').prop('checked', true);
            }
        },

        /**
         * Triggering a event to switch to next slide
         *
         * @private
         * @param OdooEvent ev
         */
        _onClickNext: function (ev) {
            if (this.slide.hasNext) {
                this.trigger_up('slide_go_next');
            }
        },

        /**
         * Resets the completion of the slide so the user can take
         * the quiz again
         *
         * @private
         */
        _onClickReset: function () {
            rpc('/slides/slide/quiz/reset', {
                slide_id: this.slide.id
            }).then(function () {
                window.location.reload();
            });
        },
        /**
         * Saves the answers from the user and redirect the user to the
         * specified url
         *
         * @private
         */
        _saveQuizAnswersToSession: function () {
            this._hideErrorMessage();

            return rpc('/slides/slide/quiz/save_to_session', {
                'quiz_answers': {'slide_id': this.slide.id, 'slide_answers': this._getQuizAnswers()},
            });
        },
        /**
        * After joining the course, we save the questions in the session
        * and reload the page to update the view.
        *
        * @private
        */
       _afterJoin: function () {
            this._saveQuizAnswersToSession().then(() => {
                window.location.reload();
            });
       },

        /**
         * When clicking on 'Add a Question' or 'Add Quiz' it
         * initialize a new QuestionFormWidget to input the new
         * question.
         * @private
         */
        _onCreateQuizClick: function () {
            var $elem = this.$('.o_wslides_js_lesson_quiz_new_question');
            this.$('.o_wslides_js_quiz_add').addClass('d-none');
            new QuestionFormWidget(this, {
                slideId: this.slide.id,
                sequence: this.quiz.questionsCount + 1
            }).appendTo($elem);
        },

        /**
         * When clicking on the edit button of a question it
         * initialize a new QuestionFormWidget with the existing
         * question as inputs.
         * @param ev
         * @private
         */
        _onEditQuestionClick: function (ev) {
            var $editedQuestion = $(ev.currentTarget).closest('.o_wslides_js_lesson_quiz_question');
            var question = this._getQuestionDetails($editedQuestion);
            new QuestionFormWidget(this, {
                editedQuestion: $editedQuestion,
                question: question,
                slideId: this.slide.id,
                sequence: question.sequence,
                update: true
            }).insertAfter($editedQuestion);
            $editedQuestion.hide();
        },

        /**
         * When clicking on the delete button of a question it toggles a modal
         * to confirm the deletion. When confirming it sends an RPC request to
         * delete the Question and triggers an event to delete it from the UI.
         * @param ev
         * @private
         */
        _onDeleteQuestionClick: function (ev) {
            const question = ev.currentTarget.closest('.o_wslides_js_lesson_quiz_question');
            const questionId = parseInt(question.dataset.questionId);
            this.call('dialog', 'add', ConfirmationDialog, {
                title: _t('Delete Question'),
                body: markup(_t('Are you sure you want to delete this question "<strong>%s</strong>"?', escape(question.dataset.title))),
                cancel: () => {
                },
                cancelLabel: _t('No'),
                confirm: async () => {
                    await this.orm.unlink('slide.question', [questionId]);
                    this.trigger_up('delete_question', { questionId });
                },
                confirmLabel: _t('Yes'),
            });
        },

        /**
         * Displays the created Question at the correct place (after the last question or
         * at the first place if there is no questions yet) It also displays the 'Add Question'
         * button or open a new QuestionFormWidget if the user wants to immediately add another one.
         *
         * @param event
         * @private
         */
        _displayCreatedQuestion: function (event) {
            var $lastQuestion = this.$('.o_wslides_js_lesson_quiz_question:last');
            if ($lastQuestion.length !== 0) {
                $lastQuestion.after(event.data.newQuestionRenderedTemplate);
            } else {
                this.$el.prepend(event.data.newQuestionRenderedTemplate);
            }
            this.quiz.questionsCount++;
            event.data.questionFormWidget.destroy();
            this.$('.o_wslides_js_quiz_add_question').removeClass('d-none');
        },

        /**
         * Replace the edited question by the new question and destroy
         * the QuestionFormWidget.
         * @param event
         * @private
         */
        _displayUpdatedQuestion: function (event) {
            var questionFormWidget = event.data.questionFormWidget;
            event.data.$editedQuestion.replaceWith(event.data.newQuestionRenderedTemplate);
            questionFormWidget.destroy();
        },

        /**
         * If the user cancels the creation or update of a Question it resets the display
         * of the updated Question or it displays back the buttons.
         *
         * @param event
         * @private
         */
        _resetDisplay: function (event) {
            var questionFormWidget = event.data.questionFormWidget;
            if (questionFormWidget.update) {
                questionFormWidget.$editedQuestion.show();
            } else {
                if (this.quiz.questionsCount > 0) {
                    this.$('.o_wslides_js_quiz_add_question').removeClass('d-none');
                } else {
                    this.$('.o_wslides_js_quiz_add_quiz').removeClass('d-none');
                }
            }
            questionFormWidget.destroy();
        },

        /**
         * After deletion of a Question the display is refreshed with the removal of the Question
         * the reordering of all the remaining Questions and the change of the new Question sequence
         * if the QuestionFormWidget is initialized.
         *
         * @param event
         * @private
         */
        _deleteQuestion: function (event) {
            var questionId = event.data.questionId;
            this.$('.o_wslides_js_lesson_quiz_question[data-question-id=' + questionId + ']').remove();
            this.quiz.questionsCount--;
            this._reorderQuestions();
            var $newQuestionSequence = this.$('.o_wslides_js_lesson_quiz_new_question .o_wslides_quiz_question_sequence');
            $newQuestionSequence.text(parseInt($newQuestionSequence.text()) - 1);
            if (this.quiz.questionsCount === 0 && !this.$('.o_wsildes_quiz_question_input').length) {
                this.$('.o_wslides_js_quiz_add_quiz').removeClass('d-none');
                this.$('.o_wslides_js_quiz_add_question').addClass('d-none');
                this.$('.o_wslides_js_lesson_quiz_validation').addClass('d-none');
            }
        },
    });

    publicWidget.registry.websiteSlidesQuizNoFullscreen = SlideCoursePage.extend({
        selector: '.o_wslides_lesson_main', // selector of complete page, as we need slide content and aside content table
        custom_events: Object.assign({}, SlideCoursePage.prototype.custom_events, {
            slide_go_next: '_onQuizNextSlide',
        }),

        //----------------------------------------------------------------------
        // Public
        //----------------------------------------------------------------------

        /**
         * @override
         * @param {Object} parent
         */
        start: function () {
            const ret = this._super(...arguments);

            const $quiz = this.$('.o_wslides_js_lesson_quiz');
            if ($quiz.length) {
                const slideData = $quiz.data();
                const channelData = this._extractChannelData(slideData);
                slideData.quizData = {
                    questions: this._extractQuestionsAndAnswers(),
                    sessionAnswers: slideData.sessionAnswers || [],
                    quizKarmaMax: slideData.quizKarmaMax,
                    quizKarmaWon: slideData.quizKarmaWon || 0,
                    quizKarmaGain: slideData.quizKarmaGain,
                    quizAttemptsCount: slideData.quizAttemptsCount,
                };

                this.quiz = new Quiz(this, slideData, channelData, slideData.quizData);
                this.quiz.attachTo($quiz);
            } else {
                this.quiz = null;
            }
            return ret;
        },

        //----------------------------------------------------------------------
        // Handlers
        //---------------------------------------------------------------------
        _onQuizNextSlide: function () {
            var url = this.$('.o_wslides_js_lesson_quiz').data('next-slide-url');
            window.location.replace(url);
        },

        //----------------------------------------------------------------------
        // Private
        //---------------------------------------------------------------------

        /**
         * Get the slide data from the elements in the DOM.
         *
         * We need this overwrite because a documentation in non-fullscreen view
         * doesn't have the standard "done" button and so in that case the slide
         * data can not be retrieved.
         *
         * @override
         * @param {Integer} slideId
         */
        _getSlide: function (slideId) {
            const slide = this._super(...arguments);
            if (slide) {
                return slide;
            }
            // A quiz in a documentation on non fullscreen view
            return $(`.o_wslides_js_lesson_quiz[data-id="${slideId}"]`).data();
        },

        /**
         * After a slide has been marked as completed / uncompleted, update the state
         * of this widget and reload the slide if needed (e.g. to re-show the questions
         * of a quiz).
         *
         * @override
         * @param {Object} slide
         * @param {Boolean} completed
         */
        toggleCompletionButton: function (slide, completed = true) {
            this._super(...arguments);

            if (this.quiz && this.quiz.slide.id === slide.id && !completed && this.quiz.quiz.questionsCount) {
                // The quiz has been marked as "Not Done", re-load the questions
                this.quiz.quiz.answers = null;
                this.quiz.quiz.sessionAnswers = null;
                this.quiz.slide.completed = false;
                this.quiz._fetchQuiz().then(() => {
                    this.quiz.renderElement();
                    this.quiz._renderValidationInfo();
                });

            }

            // The quiz has been submitted in a documentation and in non fullscreen view,
            // should update the button "Mark Done" to "Mark To Do"
            const $doneButton = $('.o_wslides_done_button');
            if ($doneButton.length && completed) {
                $doneButton
                    .removeClass('o_wslides_done_button disabled btn-primary text-white')
                    .addClass('o_wslides_undone_button btn-light')
                    .text(_t('Mark To Do'))
                    .removeAttr('title')
                    .removeAttr('aria-disabled')
                    .attr('href', `/slides/slide/${encodeURIComponent(slide.id)}/set_uncompleted`);
            }
        },

        _extractChannelData: function (slideData) {
            return {
                channelId: slideData.channelId,
                channelEnroll: slideData.channelEnroll,
                channelRequestedAccess: slideData.channelRequestedAccess || false,
                signupAllowed: slideData.signupAllowed
            };
        },

        /**
         * Extract data from exiting DOM rendered server-side, to have the list of questions with their
         * relative answers.
         * This method should return the same format as /slide/quiz/get controller.
         *
         * @return {Array<Object>} list of questions with answers
         */
        _extractQuestionsAndAnswers: function () {
            var questions = [];
            this.$('.o_wslides_js_lesson_quiz_question').each(function () {
                var $question = $(this);
                var answers = [];
                $question.find('.o_wslides_quiz_answer').each(function () {
                    var $answer = $(this);
                    answers.push({
                        id: $answer.data('answerId'),
                        text: $answer.data('text'),
                    });
                });
                questions.push({
                    id: $question.data('questionId'),
                    title: $question.data('title'),
                    answer_ids: answers,
                });
            });
            return questions;
        },
    });

    export var Quiz = Quiz;
    export const websiteSlidesQuizNoFullscreen = publicWidget.registry.websiteSlidesQuizNoFullscreen;

```

## File: static\src\js\slides_course_quiz_question_form.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';
import { _t } from "@web/core/l10n/translation";
import { renderToElement } from "@web/core/utils/render";
import { rpc } from "@web/core/network/rpc";

/**
 * This Widget is responsible of displaying the question inputs when adding a new question or when updating an
 * existing one. When validating the question it makes an RPC call to the server and trigger an event for
 * displaying the question by the Quiz widget.
 */
var QuestionFormWidget = publicWidget.Widget.extend({
    template: 'slide.quiz.question.input',
    events: {
        'click .o_wslides_js_quiz_validate_question': '_validateQuestion',
        'click .o_wslides_js_quiz_cancel_question': '_cancelValidation',
        'click .o_wslides_js_quiz_comment_answer': '_toggleAnswerLineComment',
        'click .o_wslides_js_quiz_add_answer': '_addAnswerLine',
        'click .o_wslides_js_quiz_remove_answer': '_removeAnswerLine',
        'click .o_wslides_js_quiz_remove_answer_comment': '_removeAnswerLineComment',
        'change .o_wslides_js_quiz_answer_comment > input[type=text]': '_onCommentChanged'
    },

    /**
     * @override
     * @param parent
     * @param options
     */
    init: function (parent, options) {
        this.$editedQuestion = options.editedQuestion;
        this.question = options.question || {};
        this.update = options.update;
        this.sequence = options.sequence;
        this.slideId = options.slideId;
        this._super.apply(this, arguments);
    },

    /**
     * @override
     * @returns {*}
     */
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self.$('.o_wslides_quiz_question input').focus();
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     *
     * @param commentInput
     * @private
     */
    _onCommentChanged: function (event) {
        var input = event.currentTarget;
        var commentIcon = $(input).closest('.o_wslides_js_quiz_answer').find('.o_wslides_js_quiz_comment_answer');
        if (input.value.trim() !== '') {
            commentIcon.addClass('text-primary');
            commentIcon.removeClass('text-muted');
        } else {
            commentIcon.addClass('text-muted');
            commentIcon.removeClass('text-primary');
        }
    },

    /**
     * Toggle the input for commenting the answer line which will be
     * seen by the frontend user when submitting the quiz.
     * @param ev
     * @private
     */
    _toggleAnswerLineComment: function (ev) {
        var commentLine = $(ev.currentTarget).closest('.o_wslides_js_quiz_answer').find('.o_wslides_js_quiz_answer_comment').toggleClass('d-none');
        commentLine.find('input[type=text]').focus();
    },

    /**
     * Adds a new answer line after the element the user clicked on
     * e.g. If there is 3 answer lines and the user click on the add
     *      answer button on the second line, the new answer line will
     *      display between the second and the third line.
     * @param ev
     * @private
     */
    _addAnswerLine: function (ev) {
        $(ev.currentTarget).closest('.o_wslides_js_quiz_answer').after(renderToElement('slide.quiz.answer.line'));
    },

    /**
     * Removes an answer line. Can't remove the last answer line.
     * @param ev
     * @private
     */
    _removeAnswerLine: function (ev) {
        if (this.$('.o_wslides_js_quiz_answer').length > 1) {
            $(ev.currentTarget).closest('.o_wslides_js_quiz_answer').remove();
        }
    },

    /**
     *
     * @param ev
     * @private
     */
    _removeAnswerLineComment: function (ev) {
        var commentLine = $(ev.currentTarget).closest('.o_wslides_js_quiz_answer_comment').addClass('d-none');
        commentLine.find('input[type=text]').val('').change();
    },

    /**
     * Handler when user click on 'Save' or 'Update' buttons.
     * @param ev
     * @private
     */
    _validateQuestion: function (ev) {
        this._createOrUpdateQuestion({
            update: $(ev.currentTarget).hasClass('o_wslides_js_quiz_update'),
        });
    },

    /**
     * Handler when user click on the 'Cancel' button.
     * Calls a method from slides_course_quiz.js widget
     * which will handle the reset of the question display.
     * @private
     */
    _cancelValidation: function () {
        this.trigger_up('reset_display', {
            questionFormWidget: this,
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * RPC call to create or update a question.
     * Triggers method from slides_course_quiz.js to
     * correctly display the question.
     * @param options
     * @private
     */
    _createOrUpdateQuestion: async function (options) {
        var $form = this.$('form');

        if (this._isValidForm($form)) {
            var values = this._serializeForm($form);
            var renderedQuestion = await rpc('/slides/slide/quiz/question_add_or_update', values);

            if (typeof renderedQuestion === 'object' && renderedQuestion.error) {
                this.$('.o_wslides_js_quiz_validation_error')
                    .removeClass('d-none')
                    .find('.o_wslides_js_quiz_validation_error_text')
                    .text(renderedQuestion.error);
            } else if (options.update) {
                this.$('.o_wslides_js_quiz_validation_error').addClass('d-none');
                this.trigger_up('display_updated_question', {
                    newQuestionRenderedTemplate: renderedQuestion,
                    $editedQuestion: this.$editedQuestion,
                    questionFormWidget: this,
                });
            } else {
                this.$('.o_wslides_js_quiz_validation_error').addClass('d-none');
                this.trigger_up('display_created_question', {
                    newQuestionRenderedTemplate: renderedQuestion,
                    questionFormWidget: this
                });
            }
        } else {
            this.$('.o_wslides_js_quiz_validation_error')
                .removeClass('d-none')
                .find('.o_wslides_js_quiz_validation_error_text')
                .text(_t('Please fill in the question'));
            this.$('.o_wslides_quiz_question input').focus();
        }
    },

    /**
     * Check if the Question has been filled up
     * @param $form
     * @returns {boolean}
     * @private
     */
    _isValidForm: function($form) {
        return $form.find('.o_wslides_quiz_question input[type=text]').val().trim() !== "";
    },

    /**
     * Serialize the form into a JSON object to send it
     * to the server through a RPC call.
     * @param $form
     * @returns {{id: *, sequence: *, question: *, slide_id: *, answer_ids: Array}}
     * @private
     */
    _serializeForm: function ($form) {
        var answers = [];
        var sequence = 1;
        $form.find('.o_wslides_js_quiz_answer').each(function () {
            var value = $(this).find('.o_wslides_js_quiz_answer_value').val();
            if (value.trim() !== "") {
                var answer = {
                    'sequence': sequence++,
                    'text_value': value,
                    'is_correct': $(this).find('input[type=radio]').prop('checked') === true,
                    'comment': $(this).find('.o_wslides_js_quiz_answer_comment input[type=text]').val().trim()
                };
                answers.push(answer);
            }
        });
        return {
            'existing_question_id': this.$el.data('id'),
            'sequence': this.sequence,
            'question': $form.find('.o_wslides_quiz_question input[type=text]').val(),
            'slide_id': this.slideId,
            'answer_ids': answers
        };
    },

});

export default QuestionFormWidget;

```

## File: static\src\js\slides_course_slides_list.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';
import { _t } from "@web/core/l10n/translation";
import { rpc } from "@web/core/network/rpc";
import { SlideCoursePage } from '@website_slides/js/slides_course_page';

publicWidget.registry.websiteSlidesCourseSlidesList = SlideCoursePage.extend({
    selector: '.o_wslides_slides_list',

    start: function () {
        this._super.apply(this,arguments);

        this.channelId = this.$el.data('channelId');
        this.bindedSortable = [];

        this._updateHref();
        this._bindSortable();
    },

    destroy() {
        this._unbindSortable();
        return this._super(...arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------,

    /**
     * Bind the sortable jQuery widget to both
     * - course sections
     * - course slides
     *
     * @private
     */
    _bindSortable: function () {
        const sortableBaseParam = {
            clone: false,
            placeholderClasses: ['o_wslides_slides_list_slide_hilight', 'position-relative', 'mb-1'],
            onDrop: this._reorderSlides.bind(this),
            applyChangeOnDrop: true
        };

        const container = this.el.querySelector('ul.o_wslides_js_slides_list_container');
        this.bindedSortable.push(this.call(
            "sortable",
            "create",
            {
                ...sortableBaseParam,
                ref: { el: container },
                elements: ".o_wslides_slide_list_category",
                handle: ".o_wslides_slide_list_category_header .o_wslides_slides_list_drag",
                sortableId: "category",
            },
        ).enable());

        this.bindedSortable.push(this.call(
            "sortable",
            "create",
            {
                ...sortableBaseParam,
                ref: { el: container },
                elements: ".o_wslides_slides_list_slide:not(.o_wslides_js_slides_list_empty):not(.o_not_editable)",
                handle: ".o_wslides_slides_list_drag",
                connectGroups: true,
                groups: ".o_wslides_js_slides_list_container ul",
                sortableId: "list",
            },
        ).enable());
    },

    _unbindSortable: function () {
        this.bindedSortable.forEach(sortable => sortable.cleanup());
    },

    /**
     * This method will check that a section is empty/not empty
     * when the slides are reordered and show/hide the
     * "Empty category" placeholder.
     *
     * @private
     */
    _checkForEmptySections: function (){
        this.$('.o_wslides_slide_list_category').each(function (){
            var $categoryHeader = $(this).find('.o_wslides_slide_list_category_header');
            var categorySlideCount = $(this).find('.o_wslides_slides_list_slide:not(.o_not_editable)').length;
            var $emptyFlagContainer = $categoryHeader.find('.o_wslides_slides_list_drag').first();
            var $emptyFlag = $emptyFlagContainer.find('small');
            if (categorySlideCount === 0 && $emptyFlag.length === 0){
                $emptyFlagContainer.append($('<small>', {
                    'class': "ms-1 text-muted fw-bold",
                    text: _t("(empty)")
                }));
            } else if (categorySlideCount > 0 && $emptyFlag.length > 0){
                $emptyFlag.remove();
            }
        });
    },

    _getSlides: function (){
        var categories = [];
        this.$('.o_wslides_js_list_item').each(function (){
            categories.push(parseInt($(this).data('slideId')));
        });
        return categories;
    },
    _reorderSlides: function (){
        var self = this;
        rpc('/web/dataset/resequence', {
            model: "slide.slide",
            ids: self._getSlides(),
        }).then(function (res) {
            self._checkForEmptySections();
        });
    },

    /**
     * Change links href to fullscreen mode for SEO.
     *
     * Specifications demand that links are generated (xml) without the "fullscreen"
     * parameter for SEO purposes.
     *
     * This method then adds the parameter as soon as the page is loaded.
     *
     * @private
     */
    _updateHref: function () {
        this.$(".o_wslides_js_slides_list_slide_link").each(function (){
            var href = $(this).attr('href');
            var operator = href.indexOf('?') !== -1 ? '&' : '?';
            $(this).attr('href', href + operator + "fullscreen=1");
        });
    }
});

export default publicWidget.registry.websiteSlidesCourseSlidesList;

```

## File: static\src\js\slides_course_tag_add.js

```javascript
/** @odoo-module **/

import { CourseTagAddDialog } from "@website_slides/js/public/components/course_tag_add_dialog/course_tag_add_dialog";
import publicWidget from '@web/legacy/js/public/public_widget';

publicWidget.registry.websiteSlidesTag = publicWidget.Widget.extend({
    selector: '.o_wslides_js_channel_tag_add',
    events: {
        'click': '_onAddTagClick',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onAddTagClick: function (ev) {
        ev.preventDefault();
        const channelTagIds = ev.currentTarget.dataset.channelTagIds;
        this.call("dialog", "add", CourseTagAddDialog, {
            channelId: parseInt(ev.currentTarget.dataset.channelId, 10),
            tagIds: channelTagIds ? JSON.parse(channelTagIds) : [],
        });
    },
});

export default {
    websiteSlidesTag: publicWidget.registry.websiteSlidesTag,
};

```

## File: static\src\js\slides_course_unsubscribe.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';
import { SlideUnsubscribeDialog } from "./public/components/slide_unsubscribe_dialog/slide_unsubscribe_dialog";

publicWidget.registry.websiteSlidesUnsubscribe = publicWidget.Widget.extend({
    selector: '.o_wslides_js_channel_unsubscribe',
    events: {
        'click': '_onUnsubscribeClick',
    },
    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _openDialog: function ($element) {
        var data = $element.data();
        this.call("dialog", "add", SlideUnsubscribeDialog, data);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onUnsubscribeClick: function (ev) {
        ev.preventDefault();
        this._openDialog($(ev.currentTarget));
    },
});

export default {
    websiteSlidesUnsubscribe: publicWidget.registry.websiteSlidesUnsubscribe
};

```

## File: static\src\js\slides_embed.js

```javascript
// @odoo-module ignore
/**
 * This is a minimal version of the PDFViewer widget.
 * It is NOT use in the website_slides module, but it is called when embedding
 * a slide/video/document. This code can depend on pdf.js, JQuery and Bootstrap
 * (see website_slides.slide_embed_assets bundle, in website_slides_embed.xml)
 */
$(function () {

    function debounce(func, timeout = 300){
        let timer;
        return (...args) => {
          clearTimeout(timer);
          timer = setTimeout(() => { func.apply(this, args); }, timeout);
        };
    }

    if ($('#PDFViewer') && $('#PDFViewerCanvas')) { // check if presentation only
        var MIN_ZOOM=1, MAX_ZOOM=10, ZOOM_INCREMENT=.5;

        // define embedded viewer (minimal object of the website.slide.PDFViewer widget)
        var EmbeddedViewer = function ($viewer) {
            var self = this;
            this.viewer = $viewer;
            this.slide_url = $viewer.find('#PDFSlideViewer').data('slideurl');
            this.slide_id = $viewer.find('#PDFSlideViewer').data('slideid');
            this.defaultpage = parseInt($viewer.find('#PDFSlideViewer').data('defaultpage'));
            this.canvas = $viewer.find('canvas')[0];

            this.pdf_viewer = new globalThis.PDFSlidesViewer(this.slide_url, this.canvas);
            this.hasSuggestions = !!this.$(".oe_slides_suggestion_media").length;
            this.pdf_viewer.loadDocument().then(function () {
                self.on_loaded_file();
            });
        };
        EmbeddedViewer.prototype.__proto__ = {
            // jquery inside the object (like Widget)
            $: function (selector) {
                return this.viewer.find($(selector));
            },
            // post process action (called in '.then()')
            on_loaded_file: function () {
                this.$('canvas').show();
                this.$('#page_count').text(this.pdf_viewer.pdf_page_total);
                this.$('#PDFViewerLoader').hide();
                // init first page to display
                var initpage = this.defaultpage;
                var pageNum = (initpage > 0 && initpage <= this.pdf_viewer.pdf_page_total) ? initpage : 1;
                this.render_page(pageNum);
            },
            on_rendered_page: function (pageNumber) {
                if (pageNumber) {
                    this.$('#page_number').val(pageNumber);
                    this.navUpdate(pageNumber);
                }
            },
            on_resize: function() {
                this.render_page(this.pdf_viewer.pdf_page_current);
            },
            // page switching
            render_page: function (pageNumber) {
                this.pdf_viewer.queueRenderPage(pageNumber).then(this.on_rendered_page.bind(this));
                this.navUpdate(pageNumber);
            },
            change_page: function () {
                var pageAsked = parseInt(this.$('#page_number').val(), 10);
                if (1 <= pageAsked && pageAsked <= this.pdf_viewer.pdf_page_total) {
                    this.pdf_viewer.changePage(pageAsked).then(this.on_rendered_page.bind(this));
                    this.navUpdate(pageAsked);
                } else {
                    // if page number out of range, reset the page_counter to the actual page
                    this.$('#page_number').val(this.pdf_viewer.pdf_page_current);
                }
            },
            next: function () {
                if (
                    this.pdf_viewer.pdf_page_current >=
                    this.pdf_viewer.pdf_page_total + this.hasSuggestions
                ) {
                    return;
                }

                var self = this;
                this.pdf_viewer.nextPage().then(function (pageNum) {
                    if (pageNum) {
                        self.on_rendered_page(pageNum);
                    } else {
                        if (self.pdf_viewer.pdf) { // avoid display suggestion when pdf is not loaded yet
                            self.display_suggested_slides();
                        }
                    }
                });
            },
            previous: function () {
                const slideSuggestOverlay = this.$("#slide_suggest");
                if (!slideSuggestOverlay.hasClass('d-none')) {
                    // Hide suggested slide overlay before changing page nb.
                    slideSuggestOverlay.addClass('d-none');
                    this.$("#next").removeClass("disabled");
                    if (this.pdf_viewer.pdf_page_total <= 1) {
                        this.$("#previous, #first").addClass("disabled");
                    }
                    return;
                }
                var self = this;
                this.pdf_viewer.previousPage().then(function (pageNum) {
                    if (pageNum) {
                        self.on_rendered_page(pageNum);
                    }
                    slideSuggestOverlay.addClass('d-none');
                });
            },
            first: function () {
                var self = this;
                this.pdf_viewer.firstPage().then(function (pageNum) {
                    self.on_rendered_page(pageNum);
                    self.$("#slide_suggest").addClass('d-none');
                });
            },
            last: function () {
                var self = this;
                this.pdf_viewer.lastPage().then(function (pageNum) {
                    self.on_rendered_page(pageNum);
                    self.$("#slide_suggest").addClass('d-none');
                });
            },
            zoomIn: function() {
                if(this.pdf_viewer.pdf_zoom < MAX_ZOOM) {
                    this.pdf_viewer.pdf_zoom += ZOOM_INCREMENT;
                    this.render_page(this.pdf_viewer.pdf_page_current);
                }
            },
            zoomOut: function() {
                if(this.pdf_viewer.pdf_zoom > MIN_ZOOM) {
                    this.pdf_viewer.pdf_zoom -= ZOOM_INCREMENT;
                    this.render_page(this.pdf_viewer.pdf_page_current);
                }
            },
            navUpdate: function (pageNum) {
                const pagesCount = this.pdf_viewer.pdf_page_total + this.hasSuggestions;
                this.$("#first").toggleClass("disabled", pagesCount < 2 || pageNum < 2);
                this.$("#last").toggleClass(
                    "disabled",
                    pagesCount < 2 || pageNum >= this.pdf_viewer.pdf_page_total
                );
                this.$("#next").toggleClass("disabled", pageNum >= pagesCount);
                this.$("#previous").toggleClass("disabled", pageNum <= 1);
                this.$("#zoomout").toggleClass("disabled", this.pdf_viewer.pdf_zoom <= MIN_ZOOM);
                this.$("#zoomin").toggleClass("disabled", this.pdf_viewer.pdf_zoom >= MAX_ZOOM);
            },
            // full screen mode
            fullscreen: function () {
                this.pdf_viewer.toggleFullScreen();
            },
            fullScreenFooter: function (ev) {
                if (ev.target.id === "PDFViewerCanvas") {
                    this.pdf_viewer.toggleFullScreenFooter();
                }
            },
            // display suggestion displayed after last slide
            display_suggested_slides: function () {
                this.$("#slide_suggest").removeClass("d-none");
                this.$("#next, #last").addClass("disabled");
                this.$("#previous, #first").removeClass("disabled");
            },
        };

        // embedded pdf viewer
        var embeddedViewer = new EmbeddedViewer($('#PDFViewer'));

        // bind the actions
        $('#previous').on('click', function () {
            embeddedViewer.previous();
        });
        $('#next').on('click', function () {
            embeddedViewer.next();
        });
        $('#first').on('click', function () {
            embeddedViewer.first();
        });
        $('#last').on('click', function () {
            embeddedViewer.last();
        });
        $('#zoomin').on('click', function () {
            embeddedViewer.zoomIn();
        });
        $('#zoomout').on('click', function () {
            embeddedViewer.zoomOut();
        });
        $('#page_number').on('change', function () {
            embeddedViewer.change_page();
        });
        $('#fullscreen').on('click', function () {
            embeddedViewer.fullscreen();
        });
        $('#PDFViewer').on('click', function (ev) {
            embeddedViewer.fullScreenFooter(ev);
        });
        $('#PDFViewer').on('wheel', function (ev) {
            if (ev.metaKey || ev.ctrlKey) {
                if (ev.originalEvent.deltaY > 0) {
                    embeddedViewer.zoomOut();
                } else if(ev.originalEvent.deltaY < 0) {
                    embeddedViewer.zoomIn();
                }
                return false;
            }
        });
        $(window).on("resize", debounce(() => {
            embeddedViewer.on_resize();
        }, 500));

        // switching slide with keyboard
        $(document).keydown(function (ev) {
            if (ev.key === "ArrowLeft" || ev.key === "ArrowUp") {
                embeddedViewer.previous();
            }
            if (ev.key === "ArrowRight" || ev.key === "ArrowDown") {
                embeddedViewer.next();
            }
        });

        // display the option panels
        $('.oe_slide_js_embed_option_link').on('click', function (ev) {
            ev.preventDefault();
            var toggleDiv = $(this).data('slide-option-id');
            $('.oe_slide_embed_option').not(toggleDiv).each(function () {
                $(this).hide();
            });
            $(toggleDiv).slideToggle();
        });

        // animation for the suggested slides
        $('.oe_slides_suggestion_media').hover(
            function () {
                $(this).find('.oe_slides_suggestion_caption').stop().slideDown(250);
            },
            function () {
                $(this).find('.oe_slides_suggestion_caption').stop().slideUp(250);
            }
        );

        // To avoid create a dependancy to openerpframework.js, we use JQuery AJAX to post data instead of ajax.jsonRpc
        $('.oe_slide_js_share_email button').on('click', function () {
            var widget = $('.oe_slide_js_share_email');
            var input = widget.find('input');
            var slideID = widget.find('button').data('slide-id');
            if (input.val()) {
                widget.removeClass('o_has_error').find('.form-control, .form-select').removeClass('is-invalid');
                $.ajax({
                    type: "POST",
                    dataType: 'json',
                    url: '/slides/slide/send_share_email',
                    contentType: "application/json; charset=utf-8",
                    data: JSON.stringify({'jsonrpc': "2.0", 'method': "call", "params": {'slide_id': slideID, 'emails': input.val()}}),
                    success: function (action) {
                        if (action.result) {
                            widget.find('.alert-info').removeClass('d-none');
                            widget.find('.input-group').addClass('d-none');
                        } else {
                            widget.find('.alert-warning').removeClass('d-none');
                            widget.find('.input-group').addClass('d-none');
                            widget.addClass('o_has_error').find('.form-control, .form-select').addClass('is-invalid');
                            input.focus();
                        }
                    },
                });
            } else {
                widget.find('.alert-warning').removeClass('d-none');
                widget.find('.input-group').addClass('d-none');
                widget.addClass('o_has_error').find('.form-control, .form-select').addClass('is-invalid');
                input.focus();
            }
        });
    }
});

```

## File: static\src\js\slides_share.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';
import { SlideShareDialog } from './public/components/slide_share_dialog/slide_share_dialog';
import { browser } from '@web/core/browser/browser';


publicWidget.registry.websiteSlidesShare = publicWidget.Widget.extend({
    selector: '#wrapwrap',
    events: {
        'click .o_wslides_share': '_onClickShareSlide',
    },

    getDocumentMaxPage() {
        const iframe = document.querySelector("iframe.o_wslides_iframe_viewer");
        const iframeDocument = iframe.contentWindow.document;
        return parseInt(iframeDocument.querySelector("#page_count").innerText);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    _onClickShareSlide: function (ev) {
        ev.stopPropagation();
        ev.preventDefault();
        const data = ev.currentTarget.dataset;
        this.call("dialog", "add", SlideShareDialog, {
            category: data.category,
            documentMaxPage: data.category == 'document' && this.getDocumentMaxPage(),
            emailSharing: data.emailSharing === 'True',
            embedCode: data.embedCode,
            id: parseInt(data.id),
            isChannel: data.isChannel === 'True',
            name: data.name,
            url: data.url,
        });
    },
});

publicWidget.registry.websiteSlidesEmbedShare = publicWidget.Widget.extend({
    selector: '.oe_slide_js_embed_code_widget',
    events: {
        'click .o_embed_clipboard_button': '_onShareLinkCopy',
    },

    _onShareLinkCopy: async function (ev) {
        ev.preventDefault();
        const $clipboardBtn = $(ev.currentTarget);
        $clipboardBtn.tooltip({title: "Copied!", trigger: "manual", placement: "bottom"});
        var share_embed_el = this.$('#wslides_share_embed_id_' + $clipboardBtn[0].id.split('id_')[1]);
        await browser.navigator.clipboard.writeText(share_embed_el.val() || '');
        $clipboardBtn.tooltip('show');
        setTimeout(function () {
            $clipboardBtn.tooltip("hide");
        }, 800);
    },
});

export const WebsiteSlidesShare = publicWidget.registry.websiteSlidesShare;
export const WebsiteSlidesEmbedShare = publicWidget.registry.websiteSlidesEmbedShare;

```

## File: static\src\js\slides_slide_archive.js

```javascript
/** @odoo-module **/

import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { _t } from "@web/core/l10n/translation";
import publicWidget from "@web/legacy/js/public/public_widget";
import { rpc } from "@web/core/network/rpc";

publicWidget.registry.websiteSlidesSlideArchive = publicWidget.Widget.extend({
    selector: ".o_wslides_js_slide_archive",
    events: {
        click: "_onArchiveSlideClick",
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _openDialog: function ($slideTarget) {
        const slideId = $slideTarget.data("slideId");
        this.call("dialog", "add", ConfirmationDialog, {
            title: _t("Archive Content"),
            body: _t("Are you sure you want to archive this content?"),
            confirmLabel: _t("Archive"),
            confirm: async () => {
                /**
                 * Calls 'archive' on slide controller and then visually removes the slide dom element
                 */
                const isArchived = await rpc("/slides/slide/archive", {
                    slide_id: slideId,
                });
                if (isArchived) {
                    $slideTarget.closest(".o_wslides_slides_list_slide").remove();
                    $(".o_wslides_slide_list_category").each(function () {
                        var $categoryHeader = $(this).find(".o_wslides_slide_list_category_header");
                        var categorySlideCount = $(this).find(
                            ".o_wslides_slides_list_slide:not(.o_not_editable)"
                        ).length;
                        var $emptyFlagContainer = $categoryHeader
                            .find(".o_wslides_slides_list_drag")
                            .first();
                        var $emptyFlag = $emptyFlagContainer.find("small");
                        if (categorySlideCount === 0 && $emptyFlag.length === 0) {
                            $emptyFlagContainer.append(
                                $("<small>", {
                                    class: "ms-1 text-muted fw-bold",
                                    text: _t("(empty)"),
                                })
                            );
                        }
                    });
                }
            },
            cancel: () => {},
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onArchiveSlideClick: function (ev) {
        ev.preventDefault();
        var $slideTarget = $(ev.currentTarget);
        this._openDialog($slideTarget);
    },
});

export default {
    websiteSlidesSlideArchive: publicWidget.registry.websiteSlidesSlideArchive,
};

```

## File: static\src\js\slides_slide_like.js

```javascript
/** @odoo-module **/

import { sprintf } from '@web/core/utils/strings';
import { _t } from "@web/core/l10n/translation";
import publicWidget from '@web/legacy/js/public/public_widget';
import { rpc } from "@web/core/network/rpc";
import '@website_slides/js/slides';

var SlideLikeWidget = publicWidget.Widget.extend({
    events: {
        'click .o_wslides_js_slide_like_up': '_onClickUp',
        'click .o_wslides_js_slide_like_down': '_onClickDown',
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Object} $el
     * @param {String} message
     */
    _popoverAlert: function ($el, message) {
        $el.popover({
            trigger: 'focus',
            delay: {'hide': 300},
            placement: 'bottom',
            container: 'body',
            html: true,
            content: function () {
                return message;
            }
        }).popover('show');
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onClick: function (slideId, voteType) {
        var self = this;
        rpc('/slides/slide/like', {
            slide_id: slideId,
            upvote: voteType === 'like',
        }).then(function (data) {
            if (! data.error) {
                const $likesBtn = self.$('span.o_wslides_js_slide_like_up');
                const $likesIcon = $likesBtn.find('i.fa');
                const $dislikesBtn = self.$('span.o_wslides_js_slide_like_down');
                const $dislikesIcon = $dislikesBtn.find('i.fa');

                // update 'thumbs-up' button with latest state
                $likesBtn.data('user-vote', data.user_vote);
                $likesBtn.find('span').text(data.likes);
                $likesIcon.toggleClass("fa-thumbs-up", data.user_vote === 1);
                $likesIcon.toggleClass("fa-thumbs-o-up", data.user_vote !== 1);
                // update 'thumbs-down' button with latest state
                $dislikesBtn.data('user-vote', data.user_vote);
                $dislikesBtn.find('span').text(data.dislikes);
                $dislikesIcon.toggleClass("fa-thumbs-down", data.user_vote === -1);
                $dislikesIcon.toggleClass("fa-thumbs-o-down", data.user_vote !== -1);
            } else {
                if (data.error === 'public_user') {
                    const message = data.error_signup_allowed ?
                        _t('Please <a href="/web/login?redirect=%(url)s">login</a> or <a href="/web/signup?redirect=%(url)s">create an account</a> to vote for this lesson') :
                        _t('Please <a href="/web/login?redirect=%(url)s">login</a> to vote for this lesson');
                    self._popoverAlert(self.$el, sprintf(message, { url: encodeURIComponent(document.URL) }));
                } else if (data.error === 'slide_access') {
                    self._popoverAlert(self.$el, _t('You don\'t have access to this lesson'));
                } else if (data.error === 'channel_membership_required') {
                    self._popoverAlert(self.$el, _t('You must be member of this course to vote'));
                } else if (data.error === 'channel_comment_disabled') {
                    self._popoverAlert(self.$el, _t('Votes and comments are disabled for this course'));
                } else if (data.error === 'channel_karma_required') {
                    self._popoverAlert(self.$el, _t('You don\'t have enough karma to vote'));
                } else {
                    self._popoverAlert(self.$el, _t('Unknown error'));
                }
            }
        });
    },

    _onClickUp: function (ev) {
        var slideId = $(ev.currentTarget).data('slide-id');
        return this._onClick(slideId, 'like');
    },

    _onClickDown: function (ev) {
        var slideId = $(ev.currentTarget).data('slide-id');
        return this._onClick(slideId, 'dislike');
    },
});

publicWidget.registry.websiteSlidesSlideLike = publicWidget.Widget.extend({
    selector: '#wrapwrap',

    /**
     * @override
     * @param {Object} parent
     */
    start: function () {
        var self = this;
        var defs = [this._super.apply(this, arguments)];
        $('.o_wslides_js_slide_like').each(function () {
            defs.push(new SlideLikeWidget(self).attachTo($(this)));
        });
        return Promise.all(defs);
    },
});

export default {
    slideLikeWidget: SlideLikeWidget,
    websiteSlidesSlideLike: publicWidget.registry.websiteSlidesSlideLike
};

```

## File: static\src\js\slides_slide_toggle_is_preview.js

```javascript
/** @odoo-module **/

    import publicWidget from '@web/legacy/js/public/public_widget';
    import { rpc } from "@web/core/network/rpc";

    publicWidget.registry.websiteSlidesSlideToggleIsPreview = publicWidget.Widget.extend({
        selector: '.o_wslides_js_slide_toggle_is_preview',
        events: {
            'click': '_onPreviewSlideClick',
        },

        _toggleSlidePreview: function($slideTarget) {
            rpc('/slides/slide/toggle_is_preview', {
                slide_id: $slideTarget.data('slideId')
            }).then(function (isPreview) {
                if (isPreview) {
                    $slideTarget.removeClass('text-bg-light badge-hide border');
                    $slideTarget.addClass('text-bg-success');
                } else {
                    $slideTarget.removeClass('text-bg-success');
                    $slideTarget.addClass('text-bg-light badge-hide border');
                }
            });
        },

        _onPreviewSlideClick: function (ev) {
            ev.preventDefault();
            this._toggleSlidePreview($(ev.currentTarget));
        },
    });

    export default {
        websiteSlidesSlideToggleIsPreview: publicWidget.registry.websiteSlidesSlideToggleIsPreview
    };

```

## File: static\src\js\slides_upload.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';
import { SlideUploadDialog } from "@website_slides/js/public/components/slide_upload_dialog/slide_upload_dialog";

publicWidget.registry.websiteSlidesUpload = publicWidget.Widget.extend({
    selector: '.o_wslides_js_slide_upload',
    events: {
        'click': '_onUploadClick',
    },

    /**
     * Automatically opens the upload dialog if requested from query string.
     * If openModal is defined ( === '' ), opens the category selection dialog.
     * If openModal is a category name, opens the category's upload dialog.
     *
     * @override
     */
    start: function () {
        if ('openModal' in this.$el.data()) {
            this._openDialog(this.$el);
            this.$el.data('openModal', false);
        }
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _openDialog: function ($element) {
        const dataset = $element.data();
        this.call("dialog", "add", SlideUploadDialog, {
            categoryId: dataset.categoryId,
            channelId: dataset.channelId,
            canPublish: dataset.canPublish === "True",
            canUpload: dataset.canUpload === "True",
            modulesToInstall: dataset.modulesToInstall || [],
            openModal: dataset.openModal,
        });
    },
    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onUploadClick: function (ev) {
        ev.preventDefault();
        this._openDialog($(ev.currentTarget));
    },
});

export default {
    websiteSlidesUpload: publicWidget.registry.websiteSlidesUpload
};

```

## File: static\src\js\snippets.animation.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';
import '@website/js/content/snippets.animation';

publicWidget.registry.WebsiteAnimate.include({
    /**
     * @override
     * @todo This should be avoided: the natural scrollbar of the browser should
     * always be preferred. Indeed, moving the main scroll of the page to a
     * different location causes a lot of issues. See 189a7c96e6e26825dc05c0c64
     * for more information (improvement of 18.0 for general scrolling behaviors
     * in all website pages). E.g. issue in eLearning: go to an article in full
     * screen mode, try to use the up/down arrow keys to scroll: it does not
     * work (you first have to focus the article which should not be needed as
     * it is the only main scrollable element of the page).
     */
    findScrollingElement() {
        const articleContent = document.querySelector('.o_wslide_fs_article_content');
        return articleContent ? $(articleContent) : this._super(...arguments);
    }
});

```

## File: static\src\js\components\editor.js

```javascript
/** @odoo-module **/

import { WebsiteEditorComponent } from '@website/components/editor/editor';
import { WebsiteTranslator } from '@website/components/translator/translator';
import { patch } from "@web/core/utils/patch";

patch(WebsiteEditorComponent.prototype, {
    /**
     * @override
     */
    publicRootReady() {
        const { pathname, search } = this.websiteService.contentWindow.location;
        if (pathname.includes('slides') && search.includes('fullscreen=1')) {
            this.websiteContext.edition = false;
            this.websiteService.goToWebsite({path: `${pathname}?fullscreen=0`, edition: true});
        } else {
            super.publicRootReady(...arguments);
        }
    }
});

patch(WebsiteTranslator.prototype, {
    /**
     * When editing translations of a slide in fullscreen mode: force fullscreen off.
     * Indeed, the fullscreen layout is not fit for content edition.
     * @override
     */
    publicRootReady() {
        const { pathname, search, hash } = this.websiteService.contentWindow.location;
        if (pathname.includes('slides') && search.includes('fullscreen=1')) {
            const searchParams = new URLSearchParams(search);
            searchParams.set('edit_translations', '1');
            searchParams.set('fullscreen', '0');
            this.websiteService.goToWebsite({
                path: encodeURI(pathname + `?${searchParams.toString() + hash}`),
                translation: true
            });
        } else {
            super.publicRootReady(...arguments);
        }
    }
});

```

## File: static\src\js\public\components\category_add_dialog\category_add_dialog.js

```javascript
/** @odoo-module **/

import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { useAutofocus } from "@web/core/utils/hooks";

export class CategoryAddDialog extends ConfirmationDialog {
    static template = "website_slides.CategoryAddDialog";
    static props = {
        ...ConfirmationDialog.props,
        channelId: String,
    };

    setup() {
        super.setup();
        this.inputRef = useAutofocus();
        this.csrf_token = odoo.csrf_token;
        this.lastInputValue;
    }

    _confirm() {
        this.execButton(() => {
            if (this.inputRef.el.value === this.lastInputValue) {
                return;
            }
            this.lastInputValue = this.inputRef.el.value;
            return this.props.confirm({ formEl: this.modalRef.el.querySelector("form") });
        });
    }
}

```

## File: static\src\js\public\components\category_add_dialog\category_add_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

  <t t-name="website_slides.CategoryAddDialog" t-inherit="web.ConfirmationDialog">
    <xpath expr="//p[hasclass('text-prewrap')]" position="replace">
        <div>
            <form t-on-submit.prevent="_confirm" action="/slides/category/add" method="POST" id="slide_category_add_form">
                <input type="hidden" name="csrf_token" t-att-value="csrf_token"/>
                <input type="hidden" name="channel_id" t-att-value="props.channelId"/>
                <div class="mb-3 row">
                    <label for="section_name" class="col-sm-3 col-form-label">Section name</label>
                    <div class="col-sm-9">
                        <input t-ref="autofocus" type="text" autocomplete="off" class="form-control" name="name" id="section_name" required="required" placeholder='e.g. "Introduction"'/>
                    </div>
                </div>
            </form>
        </div>
    </xpath>
  </t>

</templates>

```

## File: static\src\js\public\components\course_tag_add_dialog\course_tag_add_dialog.js

```javascript
import { Component, useState } from "@odoo/owl";
import { Dialog } from "@web/core/dialog/dialog";
import { DropdownItem } from "@web/core/dropdown/dropdown_item";
import { SelectMenu } from "@web/core/select_menu/select_menu";
import { _t } from "@web/core/l10n/translation";
import { uniqueId } from "@web/core/utils/functions";
import { rpc } from "@web/core/network/rpc";

export class CourseTagAddDialog extends Component {
    static components = { Dialog, DropdownItem, SelectMenu };
    static props = {
        channelId: { type: Number, optional: true },
        defaultTag: { type: String, optional: true },
        tagIds: Array,
        close: Function,
    };
    static template = "website_slides.CourseTagAddDialog";

    async setup() {
        super.setup();
        this.choices = useState({
            tagIds: [],
            tagGroupIds: [],
            tagId: null,
            tagGroupId: null,
        });
        this.state = useState({
            showTagGroup: false,
            canCreateTagGroup: false,
            canCreateTag: false,
            alertMsg: "",
        });
        this.validation = useState({
            tagIsValid: undefined,
            tagGroupIsValid: undefined,
        });
        const [tags, groups] = await Promise.all([
            this._fetchChoices("tag", [
                ["id", "not in", this.props.tagIds],
                ["color", "!=", 0],
            ]),
            this._fetchChoices("tag/group"),
        ]);
        this.choices.tagIds = tags.choices;
        this.state.canCreateTag = tags.can_create;
        this.choices.tagGroupIds = groups.choices;
        this.state.canCreateTagGroup = groups.can_create;

        if (this.props.defaultTag) {
            // Note: when a default tag is passed to the props we want the tag SelectMenu to behave
            // like a 'readonly' selectMenu dropdown (can see the options but cannot change the selection)
            this.createChoice(this.props.defaultTag);
            this.state.canCreateTag = false;
        }
    }

    get displayTagValue() {
        return this.choices.tagId
            ? this.choices.tagIds.find((t) => t.value === this.choices.tagId).label
            : _t("Select or create a tag");
    }

    get displayTagGroupValue() {
        return this.choices.tagGroupId
            ? this.choices.tagGroupIds.find((t) => t.value === this.choices.tagGroupId).label
            : _t("Select or create a tag group");
    }

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    onClickFormSubmit() {
        this.state.alertMsg = "";
        if (!this._formValidate()) {
            return;
        }
        const values = this._getSelectMenuValues();
        if (this.props.defaultTag && !this.channelId) {
            this._createNewTag(values);
        } else {
            this._addTagToChannel(values);
        }
    }

    /**
     * Create a new choice for a given select menu (type) and select it.
     * Also display tag group select
     * @param {String} label
     * @param {String} type
     */
    createChoice(label, type = "tag") {
        const tempId = uniqueId("temp");
        this.choices[`${type}Ids`].push({ value: tempId, label: label });
        this.choices[`${type}Id`] = tempId;
        this.state.showTagGroup = true;
    }

    /**
     * Set the tagId value and displays the tagGroup Select Menu when appropriate
     * @param {*} value
     */
    onTagSelect(value) {
        if (!this.props.defaultTag) {
            this.choices.tagId = value;
            this.state.showTagGroup = this._toCreate(value) ? true : false;
        }
    }

    /**
     * Set the tagGroupId
     * @param {*} value
     */
    onTagGroupSelect(value) {
        this.choices.tagGroupId = value;
    }

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Object} values
     */
    async _addTagToChannel(values) {
        const data = await rpc("/slides/channel/tag/add", {
            channel_id: this.props.channelId,
            ...values,
        });

        if (data.error) {
            this.state.alertMsg = data.error;
        } else {
            window.location.reload();
        }
    }

    /**
     * @private
     * @param {Object} values
     */
    async _createNewTag(values) {
        const data = await rpc("/slide_channel_tag/add", values);

        if (data.error) {
            this.state.alertMsg = data.error;
        } else {
            this.props.close();
        }
    }

    /**
     * @private
     * @returns Boolean
     */
    _formValidate() {
        for (const key in this.validation) {
            this.validation[key] = undefined;
        }
        if (!this.choices.tagId) {
            this.validation.tagIsValid = false;
            return false;
        }
        this.validation.tagIsValid = true;
        if (this.state.showTagGroup) {
            if (!this.choices.tagGroupId) {
                this.validation.tagGroupIsValid = false;
                return false;
            }
            this.validation.tagGroupIsValid = true;
        }
        return true;
    }

    /**
     * @private
     * @param {String} type
     * @param {Array} domain
     * @param {Array} fields
     * @returns {Object} result
     */
    async _fetchChoices(type, domain = [], fields = ["name"]) {
        const { read_results, can_create } = await rpc(`/slides/channel/${type}/search_read`, {
            fields,
            domain,
        });

        const choices = read_results.map((choice) => {
            return { value: choice.id, label: choice.name };
        });
        return { choices, can_create };
    }

    /**
     * Get value for tagId and [when appropriate] tagGroupId to send to server
     * @private
     */
    _getSelectMenuValues() {
        const tag = this.choices.tagIds.find((c) => c.value === this.choices.tagId);
        if (!tag) {
            return {};
        }
        if (!this._toCreate(tag.value)) {
            // existing tag
            return { tag_id: [tag.value] };
        }
        const group = this.choices.tagGroupIds.find((c) => c.value === this.choices.tagGroupId);
        if (!group) {
            return {};
        }
        return {
            tag_id: [0, { name: tag.label }],
            group_id: this._toCreate(group.value) ? [0, { name: group.label }] : [group.value],
        };
    }

    /**
     * @private
     * @param {*} value
     * @returns Boolean
     */
    _toCreate(value) {
        return typeof value === "string" && value.startsWith("temp");
    }
}

```

## File: static\src\js\public\components\course_tag_add_dialog\course_tag_add_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

  <t t-name="website_slides.CourseTagAddDialog">
    <Dialog size="'md'" title.translate="Add Tag">
        <t t-set-slot="footer">
            <a role="button" class="btn btn-primary me-1" t-on-click.prevent="onClickFormSubmit">
                <span>Add</span>
            </a>
            <button type="button" class="btn btn-secondary" t-on-click="props.close">
                <span>Back</span>
            </button>
        </t>
        <div class="mb-3">
            <div t-if="state.alertMsg" t-out="state.alertMsg" class="alert alert-warning"/>
            <div class="mb-3" t-att-class="{'form-control is-valid': validation.tagIsValid, 'form-control is-invalid': validation.tagIsValid === false}">
                <label class="col-form-label">Tag</label>
                <SelectMenu
                    choices="choices.tagIds"
                    onSelect.bind="onTagSelect"
                    required="true" 
                    togglerClass="'text-dark pe-1'"
                    value="choices.tagId"
                >
                    <t t-out="displayTagValue"/>
                    <t t-set-slot="bottomArea" t-slot-scope="select">
                        <DropdownItem
                            t-if="select.data.searchValue and state.canCreateTag"
                            class="'btn text-primary'"
                            onSelected="() => this.createChoice(select.data.searchValue)"
                        >
                            Create this tag "<i t-out="select.data.searchValue" />"
                        </DropdownItem>
                    </t>
                </SelectMenu>
            </div>
            <div t-if="state.showTagGroup" class="mb-3" t-att-class="{'form-control is-valid': validation.tagGroupIsValid, 'form-control is-invalid': validation.tagGroupIsValid === false}">
                <label class="col-form-label">Tag Group</label>
                <SelectMenu 
                    choices="choices.tagGroupIds"
                    onSelect.bind="onTagGroupSelect"
                    required="true"
                    togglerClass="'text-dark pe-1'"
                    value="choices.tagGroupId"
                >
                    <t t-out="displayTagGroupValue"/>
                    <t t-set-slot="bottomArea" t-slot-scope="select">
                        <DropdownItem
                            t-if="select.data.searchValue and state.canCreateTagGroup"
                            class="'btn text-primary'"
                            onSelected="() => this.createChoice(select.data.searchValue, 'tagGroup')"
                        >
                            Create this tag group"<i t-out="select.data.searchValue" />"
                        </DropdownItem>
                    </t>
                </SelectMenu>
            </div>
        </div>
    </Dialog>
  </t>

</templates>

```

## File: static\src\js\public\components\slide_quiz_finish_dialog\slide_quiz_finish_dialog.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { browser } from "@web/core/browser/browser";
import { Dialog } from "@web/core/dialog/dialog";
import { Component, onMounted, useState } from "@odoo/owl";
import { SlideXPProgressBar } from "@website_slides/js/public/components/slide_quiz_finish_dialog/slide_xp_progress_bar";

export class SlideQuizFinishDialog extends Component {
    static components = { Dialog, SlideXPProgressBar };
    static props = {
        close: Function,
        hasNext: Boolean,
        onClickNext: Function,
        quiz: Object,
        userId: Number,
    };
    static template = "website_slides.SlideQuizFinishDialog";

    setup() {
        super.setup();
        this.state = useState({
            animateKarmaGain: false,
            fadeRankMotivational: false,
            hideDismissBtns: true,
            showRankMotivational: false,
        });
        this.title = this.props.quiz.rankProgress.level_up ? _t("Level up!") : _t("Amazing!");
        onMounted(() => this.animateText());
    }

    //--------------------------------
    // Handler
    //--------------------------------

    onClickNext() {
        this.props.onClickNext();
        this.props.close();
    }

    //--------------------------------
    // Business methods
    //--------------------------------

    /**
     * Handles the animation of the different text such as the karma gain
     * and the motivational message when the user levels up.
     * @public
     */
    animateText() {
        browser.setTimeout(() => {
            this.state.animateKarmaGain = true;
            this.state.hideDismissBtns = false;
        }, 800);

        if (this.props.quiz.rankProgress.level_up) {
            browser.setTimeout(() => {
                this.state.fadeRankMotivational = true;
                browser.setTimeout(() => {
                    this.state.showRankMotivational = true;
                }, 800);
            }, 800);
        }
    }
}

```

## File: static\src\js\public\components\slide_quiz_finish_dialog\slide_quiz_finish_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

    <t t-name="website_slides.SlideQuizFinishDialog" owl="1">
        <Dialog
            bodyClass="'d-flex p-0'"
            contentClass="'o_wslides_quiz_modal shadow-lg'"
            footer="false"
            header="false"
            size="'md'"
            technical="false"
        >
            <a role="button" class="o_wslides_quiz_modal_close_btn btn-close position-absolute" aria-label="Close" t-on-click.prevent="() => this.props.close()"/>
            <div class="o_wslides_quiz_success_image d-none d-md-flex flex-shrink-0">
                <img class="o_wslides_quiz_modal_hero" src="/website_slides/static/src/img/quiz_modal_success.svg" alt="Triumphant hero"/>
            </div>
            <div class="d-flex flex-column flex-grow-1 justify-content-between ps-md-5 p-3 overflow-visible">
                <div>
                    <h1 class="mt-3 display-4 fw-bold" t-out="title"/>
                    <div class="pb-3">
                        <h4 class="pb-2 d-flex fade" t-att-class="state.animateKarmaGain ? 'show in': ''">
                            <t t-if="props.quiz.quizKarmaWon > 0">
                                You gained <span class="badge text-bg-success text-white fw-bold ms-2 me-1"><t t-out="props.quiz.quizKarmaWon"/> XP</span>!
                            </t>
                            <t t-else="">You did it!</t>
                        </h4>
                        <div class="mt-5 mb-4">
                            <SlideXPProgressBar
                                previousRank="props.quiz.rankProgress.previous_rank"
                                newRank="props.quiz.rankProgress.new_rank"
                                levelUp="props.quiz.rankProgress.level_up"
                            />
                        </div>
                    </div>
                    <div class="pb-3 o_wslides_quiz_modal_rank_motivational" t-att-class="{'fade': state.fadeRankMotivational, 'show in': state.showRankMotivational}">
                        <t t-if="props.quiz.rankProgress.last_rank" t-out="props.quiz.rankProgress.description"/>
                        <t t-else="" t-out="props.quiz.rankProgress.new_rank.motivational"/>
                    </div>
                </div>
                <div t-attf-class="o_wslides_quiz_modal_dismiss align-self-end ${this.state.hideDismissBtns ? 'd-none': ''}">
                    <a t-if="props.quiz.rankProgress.level_up" type="button" target="_blank" t-attf-href="/profile/user/#{props.userId}" class="btn btn-light border me-1">
                        Check Profile
                    </a>
                    <button t-if="props.hasNext" type="button" class="btn btn-light border" t-on-click="onClickNext">
                        Next <i class="oi oi-chevron-right"/>
                    </button>
                    <a t-else="" type="button" href="/slides" class="btn btn-light border">End course</a>
                </div>
            </div>
        </Dialog>
    </t>

</templates>

```

## File: static\src\js\public\components\slide_quiz_finish_dialog\slide_xp_progress_bar.js

```javascript
/** @odoo-module **/

import { browser } from "@web/core/browser/browser";
import { Component, onMounted, useState } from "@odoo/owl";

export class SlideXPProgressBar extends Component {
    static props = {
        previousRank: Object,
        newRank: Object,
        levelUp: Boolean,
    };
    static template = "website_slides.SlideXPProgressBar";

    setup() {
        super.setup();
        this.state = useState({
            hideRankBounds: true,
            rankLowerBound: this.props.previousRank.lower_bound,
            rankProgressPercentage: this.props.previousRank.progress,
            userKarma: this.props.previousRank.karma,
            rankUpperBound: this.props.previousRank.upper_bound,
        });
        onMounted(() => {
            this.animateProgressBar();
        });
    }

    //--------------------------------
    // Business methods
    //--------------------------------

    /**
     * Handles the animation of the karma gain in the following steps:
     * 1. Animate the tooltip text to increment smoothly from the old
     *    karma value to the new karma value.
     * 2a. The user doesn't level up
     *    I.   When the user doesn't level up the progress bar simply goes
     *         from the old karma value to the new karma value.
     * 2b. The user levels up
     *    I.   The first step makes the progress bar go from the old karma
     *         value to 100%.
     *    II.  The second step makes the progress bar go from 100% to 0%.
     *    III. The third and final step makes the progress bar go from 0%
     *         to the new karma value. It also changes the lower and upper
     *         bound to match the new rank.
     * @public
     */
    animateProgressBar() {
        // tooltip: karma incrementation
        const duration = this.props.levelUp ? 1700 : 800;
        const startTime = Date.now();

        const animateKarma = () => {
            const progress = (Date.now() - startTime) / duration;
            if (progress >= 1) {
                this.state.userKarma = this.props.newRank.karma;
            } else {
                this.state.userKarma = Math.ceil(
                    this.props.previousRank.karma +
                        (this.props.newRank.karma - this.props.previousRank.karma) * progress
                );
                browser.requestAnimationFrame(animateKarma);
            }
        };

        // progress bar and tooltip animations
        this.state.hideRankBounds = false;
        browser.requestAnimationFrame(animateKarma);
        this.state.rankProgressPercentage = this.props.newRank.progress;

        if (this.props.levelUp) {
            browser.setTimeout(() => {
                this.state.rankLowerBound = this.props.newRank.lower_bound;
                this.state.rankUpperBound = this.props.newRank.upper_bound;
            }, 800);
        }
    }
}

```

## File: static\src\js\public\components\slide_quiz_finish_dialog\slide_xp_progress_bar.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

    <t t-name="website_slides.SlideXPProgressBar" owl="1">
        <div class="progress">
            <div class="progress-bar" t-att-class="{'level-up': props.levelUp}" role="progressbar"
                 t-att-aria-valuenow="props.previousRank.progress" aria-valuemin="0" aria-valuemax="100" aria-label="Progress bar"
                 t-attf-style="width: #{state.rankProgressPercentage}%"/>
        </div>
        <small class="float-start text-primary fw-bold" t-att-class="{'d-none': state.hideRankBounds}"
               t-out="state.rankLowerBound"/>
        <small t-if="state.rankUpperBound" class="float-end fw-bold"
               t-att-class="{'d-none': state.hideRankBounds}" t-out="state.rankUpperBound"/>
        <div class="o_wlides_xp_progressbar_tooltip tooltip fade show bs-tooltip-top position-relative" t-att-class="{'level-up': props.levelUp}" role="tooltip"
             t-attf-style="left: #{state.rankProgressPercentage}%">
            <div class="tooltip-arrow"/>
            <div class="tooltip-inner d-flex justify-content-center w-100" t-out="state.userKarma"/>
        </div>
    </t>

</templates>

```

## File: static\src\js\public\components\slide_share_dialog\email_sharing_input.js

```javascript
/** @odoo-module **/

import { rpc } from "@web/core/network/rpc";
import { session } from "@web/session";
import { useService } from "@web/core/utils/hooks";
import { _t } from "@web/core/l10n/translation";

import { Component, useRef, useState } from "@odoo/owl";

export class EmailSharingInput extends Component {
    static template = "website_slides.EmailSharingInput";
    static props = {
        id: { type: Number },
        isChannel: { type: Boolean, optional: true },
        isFullscreen: { type: Boolean, optional: true },
        category: { type: String, optional: true },
    };

    setup() {
        this.notification = useService("notification");
        this.input = useRef("input");
        this.isWebsiteUser = session.is_website_user;
        this.state = useState({
            isDone: false,
            isInvalid: false,
        });
    }

    onKeyPress(event) {
        if (event.key === "Enter") {
            event.preventDefault();
            this.onShareByEmailClick();
        }
    }

    async onShareByEmailClick() {
        const emails = this.input.el.value;
        if (emails) {
            const type = this.props.isChannel ? "channel" : "slide";
            const done = await rpc(`/slides/${type}/send_share_email`, {
                emails: emails,
                fullscreen: this.props.isFullscreen,
                [`${type}_id`]: this.props.id,
            });
            this.state.isDone = done;
            if (done) {
                return;
            }
        }
        this.setInvalid();
    }

    setInvalid() {
        this.state.isInvalid = true;
        this.notification.add(_t("Please enter valid email(s)"), { type: "danger" });
        this.input.el.focus();
    }
}

```

## File: static\src\js\public\components\slide_share_dialog\email_sharing_input.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="website_slides.EmailSharingInput">
        <div class="o_wslides_js_share_email">
            <div t-if="!this.state.isDone and !this.isWebsiteUser" class="input-group">
                <input t-ref="input" type="text" placeholder="friend1@email.com, friend2@email.com"
                       class="form-control" t-att-class="{'is-invalid': this.state.isInvalid}"
                       t-on-keypress="onKeyPress"/>
                <button type="button" class="btn btn-primary" t-on-click.stop="onShareByEmailClick">
                    <i class="fa fa-envelope-o mx-1"/>Send Email
                </button>
            </div>
            <div t-if="this.state.isDone or this.isWebsiteUser" class="alert alert-info" role="alert">
                <span t-if="this.state.isDone">
                    <strong>Sharing is caring!</strong> Email(s) sent.
                </span>
                <span t-if="this.isWebsiteUser">
                    Please <a t-attf-href="/odoo?redirect={{ window.location.href }}" class="fw-bold"> login </a>
                    to share this <span t-esc="this.props.category">course</span> by email.
                </span>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\js\public\components\slide_share_dialog\slide_share_dialog.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { browser } from "@web/core/browser/browser";
import { CopyButton } from "@web/core/copy_button/copy_button";
import { Dialog } from "@web/core/dialog/dialog";
import { EmailSharingInput } from "./email_sharing_input";

import { Component, useRef } from "@odoo/owl";

export class SlideShareDialog extends Component {
    static template = "website_slides.SlideShareDialog";
    static components = { Dialog, CopyButton, EmailSharingInput };
    static props = {
        category: { type: String, optional: true },
        close: { type: Function },
        documentMaxPage: { type: Number, optional: true },
        emailSharing: { type: Boolean, optional: true },
        embedCode: { type: String, optional: true },
        id: { type: Number },
        isChannel: { type: Boolean, optional: true },
        isFullscreen: { type: Boolean, optional: true },
        name: { type: String },
        url: { type: String },
    };

    setup() {
        this.codeInput = useRef("codeInput");
        this.copyUrlText = _t("Copy Link");
        this.copyEmbedCodeText = _t("Copy Embed Code");
        this.successText = _t("Copied");
    }

    onSocialShareClick(url) {
        browser.open(url, "Share Dialog", "width=626,height=436");
    }

    onPageChange(event) {
        const page = event.currentTarget.value;
        const newEmbedCodeValue = this.codeInput.el.value.replace(/(page=).*?([^\d]+)/, "$1" + page + "$2");
        this.codeInput.el.value = newEmbedCodeValue;
    }
}

```

## File: static\src\js\public\components\slide_share_dialog\slide_share_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

    <t t-name="website_slides.SlideShareDialog">
        <t t-if="this.props.isChannel" t-set="title">Share this Course</t>
        <t t-else="" t-set="title">Share this Content</t>
        <Dialog size="'md'" title="title">
            <div class="row">
                <div class="col-12">
                    <h5 class="mt-0 mb-2">Share Link</h5>
                    <div class="input-group">
                        <input type="text" class="form-control text-center" t-att-value="this.props.url"
                               readonly="readonly" onClick="this.select();"/>
                        <CopyButton content="this.props.url" copyText="copyUrlText" className="'btn-primary'" successText="successText"/>
                    </div>
                </div>
                <div class="col-12 mt-4">
                    <h5 class="mt-0 mb-2">Share on Social Media</h5>
                    <div class="btn-group" role="group">
                        <div class="s_share">
                            <a class="btn border bg-white" aria-label="Share on Facebook" title="Share on Facebook"
                               t-on-click.prevent="() => onSocialShareClick(`https://www.facebook.com/sharer/sharer.php?u=${props.url}`)">
                                <i class="fa fa-facebook-square fa-fw"/>
                            </a>
                            <a class="btn border bg-white" aria-label="Share on X" title="Share on X"
                               t-on-click.prevent="() => onSocialShareClick(`https://twitter.com/intent/tweet?text=${props.name}&amp;url=${props.url}`)">
                                <i class="fa fa-twitter fa-fw"/>
                            </a>
                            <a class="btn border bg-white" aria-label="Share on LinkedIn" title="Share on LinkedIn"
                               t-on-click.prevent="() => onSocialShareClick(`http://www.linkedin.com/sharing/share-offsite/?url=${props.url}`)">
                                <i class="fa fa-linkedin fa-fw"/>
                            </a>
                            <a class="btn border bg-white" aria-label="Share on Whatsapp" title="Share on Whatsapp"
                               t-on-click.prevent="() => onSocialShareClick(`https://wa.me/?text=${window.location.href}`)">
                                <i class="fa fa-whatsapp fa-fw"/>
                            </a>
                            <a class=" btn border bg-white" aria-label="Share on Pinterest" title="Share on Pinterest"
                               t-on-click.prevent="() => onSocialShareClick(`http://pinterest.com/pin/create/button/?url=${window.location.href}`)">
                                <i class="fa fa-pinterest fa-fw"/>
                            </a>
                        </div>
                    </div>
                </div>
                <div class="col-12" t-if="this.props.emailSharing">
                    <h5 class="mt-4">Share by Email</h5>
                    <EmailSharingInput id="this.props.id" category="this.props.category"
                                       isFullscreen="this.props.isFullscreen" isChannel="this.props.isChannel"/>
                </div>
                <div class="col-12 o_wslides_embed_code" t-if="this.props.embedCode">
                    <h5 class="mt-4">Embed in another Website</h5>
                    <div class="input-group">
                        <textarea t-ref="codeInput" class="form-control" t-att-value="this.props.embedCode"
                                  readonly="readonly" onClick="this.select();"/>
                        <CopyButton content="this.props.embedCode" copyText="copyEmbedCodeText"
                                    successText="successText"/>
                    </div>
                    <div t-if="this.props.category == 'document' and this.props.documentMaxPage > 1" class="input-group mt-4">
                        <span class="input-group-text">Start at Page</span>
                        <input type="number" class="form-control" t-on-change="onPageChange"
                               value="1" min="1" t-att-max="this.props.documentMaxPage"/>
                    </div>
                </div>
            </div>

            <t t-set-slot="footer">
                <button class="btn btn-primary" t-on-click="() => this.props.close()">Close</button>
            </t>
        </Dialog>
    </t>

</templates>

```

## File: static\src\js\public\components\slide_unsubscribe_dialog\slide_unsubscribe_dialog.js

```javascript
/** @odoo-module **/

import { Component, useState } from "@odoo/owl";
import { CheckBox } from "@web/core/checkbox/checkbox";
import { Dialog } from "@web/core/dialog/dialog";
import { _t } from "@web/core/l10n/translation";
import { rpc } from "@web/core/network/rpc";

export class SlideUnsubscribeDialog extends Component {
    static template = "website_slides.SlideUnsubscribeDialog";
    static components = { CheckBox, Dialog };
    static props = {
        channelId: Number,
        isFollower: { type: String, optional: true },
        visibility: String,
        enroll: { type: String, optional: true },
        close: Function,
    };

    setup() {
        this.state = useState({
            buttonDisabled: false,
        });
        this.channelID = parseInt(this.props.channelId, 10);
        this.isFollower = this.props.isFollower === "True";
        this.updateState("subscription");
        this.isChecked = this.isFollower;
    }

    updateState(mode) {
        if (mode === "subscription") {
            this.state.title = this.isFollower ? _t("Subscribe") : _t("Notifications");
            this.state.mode = "subscription";
        } else if (mode === "leave") {
            this.state.title = _t("Leave the course");
            this.state.mode = "leave";
        }
    }

    onChangeCheckbox(isChecked) {
        this.isChecked = isChecked;
    }

    onClickLeaveCourse() {
        this.updateState("leave");
    }

    onClickLeaveCourseCancel() {
        this.updateState("subscription");
    }

    async onClickLeaveCourseSubmit() {
        if (this.state.buttonDisabled) {
            return;
        }
        this.state.buttonDisabled = true;

        await rpc("/slides/channel/leave", { channel_id: this.channelID });
        if (this.props.visibility === "public" || this.props.visibility === "connected") {
            window.location.reload();
        } else {
            window.location.href = "/slides";
        }
    }

    async onClickSubscriptionSubmit() {
        if (this.state.buttonDisabled) {
            return;
        }
        this.state.buttonDisabled = true;

        if (this.isFollower === this.isChecked) {
            this.props.close();
        } else {
            await rpc(`/slides/channel/${this.isChecked ? "subscribe" : "unsubscribe"}`, {
                channel_id: this.channelID,
            });
            window.location.reload();
        }
    }
}

```

## File: static\src\js\public\components\slide_unsubscribe_dialog\slide_unsubscribe_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

  <t t-name="website_slides.SlideUnsubscribeDialog">
    <Dialog size="'md'" title="state.title">
      <div>
        <t t-if="state.mode === 'subscription'">
            <form class="clearfix">
                <div class="controls mt8">
                    <CheckBox value="isChecked" name="subscribed" onChange="(isChecked) => this.onChangeCheckbox(isChecked)">
                        Be notified when a new content is added.
                    </CheckBox>
                </div>
            </form>
        </t>
        <t t-if="state.mode === 'leave'">
            <p>Do you really want to leave the course?</p>
            <p>All progress will be lost until you rejoin this course.</p>
        </t>
      </div>

      <t t-set-slot="footer">
        <t t-if="state.mode === 'subscription'">
            <button class="btn btn-primary" t-att-disabled="state.buttonDisabled" t-on-click="() => this.onClickSubscriptionSubmit()">Save</button>
            <button class="btn" t-att-disabled="state.buttonDisabled" t-on-click="() => this.props.close()">Discard</button>
            <button class="btn btn-danger ms-auto" t-att-disabled="state.buttonDisabled" t-on-click="() => this.onClickLeaveCourse()">or Leave the course</button>
        </t>
        <t t-if="state.mode === 'leave'">
            <button class="btn btn-danger" t-att-disabled="state.buttonDisabled" t-on-click="() => this.onClickLeaveCourseSubmit()">Leave the course</button>
            <button class="btn" t-att-disabled="state.buttonDisabled" t-on-click="() => this.onClickLeaveCourseCancel()">Discard</button>
        </t>
      </t>
    </Dialog>
  </t>

</templates>

```

## File: static\src\js\public\components\slide_upload_dialog\slide_install_module.js

```javascript
/** @odoo-module **/

import { Component, useState } from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";
import { redirect } from "@web/core/utils/urls";
import { _t } from "@web/core/l10n/translation";

export class SlideInstallModule extends Component {
    static components = {};
    static props = {
        moduleData: {
            name: String,
            id: Number,
            default_slide_category: { type: String, optional: true },
        },
    };
    static template = "website_slides.SlideInstallModule";

    setup() {
        this.orm = useService("orm");
        this.state = useState({
            status: "start", // "failure", "installing"
            message: _t('Do you want to install "%s"?', this.props.moduleData.name),
        });
    }

    async installModule() {
        if (this.state.status === "installing") {
            return;
        }
        this.state.status = "installing";
        this.state.message = _t('Installing "%s"...', this.props.moduleData.name);
        try {
            await this.orm.call("ir.module.module", "button_immediate_install", [
                [this.props.moduleData.id],
            ]);
        } catch {
            this.state.hasFailed = "failure";
            this.state.message = _t('Failed to install "%s"', this.props.moduleData.name);
            return;
        }
        let redirectUrl = window.location.origin + window.location.pathname;
        if (this.props.moduleData.default_slide_category) {
            redirectUrl += "?enable_slide_upload=" + this.props.moduleData.default_slide_category;
        }
        redirect(redirectUrl);
    }
}

```

## File: static\src\js\public\components\slide_upload_dialog\slide_install_module.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="website_slides.SlideInstallModule">
        <span
            t-if="state.status === 'installing'"
            class="spinner-border spinner-border-sm me-1"
        />
        <t t-out="state.message"/>
        <t t-portal="'#o_w_slide_upload_btns'">
            <button
                class="btn btn-primary"
                t-att-disabled="state.status === 'installing'"
                t-on-click="installModule"
            >
                <span t-if="state.status === 'failure'">Retry</span>
                <span t-else="">Install</span>
            </button>
        </t>
    </t>
</templates>

```

## File: static\src\js\public\components\slide_upload_dialog\slide_upload_category.js

```javascript
/** @odoo-module **/

import { Component, onMounted, onWillStart, useState } from "@odoo/owl";
import { getDataURLFromFile } from "@web/core/utils/urls";
import { rpc } from "@web/core/network/rpc";
import { uniqueId } from "@web/core/utils/functions";
import { DropdownItem } from "@web/core/dropdown/dropdown_item";
import { SelectMenu } from "@web/core/select_menu/select_menu";
import { _t } from "@web/core/l10n/translation";
import { SlideUploadSourceTypes } from "./slide_upload_source_types";
import { SlideUploadSelectTags } from "./slide_upload_select_tags";

export class SlideUploadCategory extends Component {
    static components = { DropdownItem, SelectMenu, SlideUploadSelectTags, SlideUploadSourceTypes };
    static props = {
        alertMsg: { type: String, optional: true },
        channelId: Number,
        categoryId: { type: String, optional: true },
        slideCategory: String,
        canPublish: Boolean,
        canUpload: Boolean,
        upload: Function,
        slots: Object,
    };
    static sourceSettings = {
        document: {
            sourceTypeLabel: _t("Document Source"),
            selectFileLabel: _t("Choose a PDF"),
            acceptedFiles: "application/pdf",
            urlInputLabel: _t("Document Link"),
            urlInputName: "document_google_url",
        },
        infographic: {
            sourceTypeLabel: _t("Image Source"),
            selectFileLabel: _t("Choose an Image"),
            acceptedFiles: "image/*",
            urlInputLabel: _t("Image Link"),
            urlInputName: "image_google_url",
        },
        video: {
            urlInputLabel: _t("Video Link"),
            urlInputName: "video_url",
        },
    };
    static template = "website_slides.SlideUploadCategory";

    setup() {
        this.sourceSettings = SlideUploadCategory.sourceSettings;
        this.state = useState({
            alert: {
                class: "",
                message: "",
                show: false,
            },
            form: {
                duration: null,
                isLoading: false,
                isLocalSource: true,
                slideImage: "/website_slides/static/src/img/document.png",
                slideName: "",
                wasValidated: false,
                url: "",
            },
            preview: {
                show: false,
                hideSlideVideoTitle: true,
                videoTitle: "",
            },
            choices: {
                categories: [],
                categoryId: "",
                tags: [],
                tagIds: [],
            },
        });
        this.canSubmitForm = false;
        this.defaultCategoryId = parseInt(this.props.categoryId, 10);
        this.file = {};
        this.isValidUrl = true;

        onWillStart(async () => {
            const categories = await this._fetch_choices("category", [
                ["channel_id", "=", this.props.channelId],
            ]);
            this.state.choices.categories = categories;
            this.state.choices.categoryId = this._getDefaultCategoryId();
            const tags = await this._fetch_choices("tag");
            this.state.choices.tags = tags;
        });

        onMounted(() => {
            if (this.props.alertMsg) {
                this._alertDisplay(this.props.alertMsg);
            }
        });
    }

    /**
     * To figure when to propose users to create a new category or tag
     */
    choiceExists(input, choices) {
        return choices.some((choice) => input.toLowerCase() === choice.label.toLowerCase());
    }

    get displayCategoryValue() {
        return this.state.choices.categoryId
            ? this.state.choices.categories.find((c) => c.value === this.state.choices.categoryId)
                  .label
            : _t("Select or create a category");
    }

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    // Category and tag SelectMenus

    onCategorySelect(value) {
        this.state.choices.categoryId = value;
    }

    onClickCreateCategoryBtn(categoryName) {
        const tempId = uniqueId("temp");
        this.state.choices.categories.push({ value: tempId, label: categoryName });
        this.state.choices.categoryId = tempId;
    }

    onTagsSelect(values) {
        this.state.choices.tagIds = values;
    }

    onClickCreateTagBtn(tagName) {
        const tempId = uniqueId("temp");
        this.state.choices.tags.push({ value: tempId, label: tagName });
        this.state.choices.tagIds.push(tempId);
    }

    // Form

    async onChangeFileInput(ev) {
        this._alertRemove();

        const preventOnchange = ev.currentTarget.dataset.preventOnchange;

        const file = ev.target.files[0];
        if (!file) {
            this.state.form.slideImage = "/website_slides/static/src/img/document.png";
            this.state.preview.show = false;
            return;
        }
        const isImage = /^image\/.*/.test(file.type);
        let loaded = false;
        this.file.name = file.name;
        this.file.type = file.type;
        if (!isImage && this.file.type !== "application/pdf") {
            this._alertDisplay(_t("Invalid file type. Please select pdf or image file"));
            this._fileReset();
            this.state.preview.show = false;
            return;
        }
        if (file.size > 25 * 1024 * 1024) {
            this._alertDisplay(_t("File is too big. File size cannot exceed 25MB"));
            this._fileReset();
            this.state.preview.show = false;
            return;
        }

        if (file.type !== "application/pdf") {
            const dataURL = await getDataURLFromFile(file);
            if (isImage) {
                this.state.form.slideImage = dataURL;
            }
            this.file.data = dataURL.split(",", 2)[1];
            this.state.preview.show = true;
        } else {
            this.canSubmitForm = false;
            const dataURL = await getDataURLFromFile(file);
            this.file.data = dataURL.split(",", 2)[1];
            /**
             * The following line fixes pdfjsLib 'Util' global variable.
             * This is (most likely) related to #32181 which lazy loads most assets.
             * See commit 3716a9b
             */
            window.Util = window.pdfjsLib.Util;
            // pdf is stored in file.data in base64 and converted in binary (atob) to generate the preview
            const pdfTask = window.pdfjsLib.getDocument({ data: atob(this.file.data) });
            pdfTask.onPassword = () => {
                this._alertDisplay(_t("You can not upload password protected file."));
                this._fileReset();
                this.canSubmitForm = true;
            };
            const pdf = await pdfTask.promise;

            this.state.form.duration = (pdf.numPages || 0) * 5;
            const page = await pdf.getPage(1);
            const viewport = page.getViewport({ scale: 1 });
            const canvas = document.getElementById("data_canvas");
            const context = canvas.getContext("2d");
            canvas.height = viewport.height;
            canvas.width = viewport.width;
            // Render PDF page into canvas context
            await page.render({
                canvasContext: context,
                viewport: viewport,
            }).promise;
            this.state.form.slideImage = canvas.toDataURL();
            if (loaded) {
                this.canSubmitForm = true;
            }
            loaded = true;
            this.state.preview.show = true;
        }

        if (!preventOnchange) {
            const input = file.name;
            const inputVal = input.substr(0, input.lastIndexOf(".")) || input;
            if (this.state.form.slideName === "") {
                this.state.form.slideName = inputVal;
            }
        }
    }

    /**
     * When the URL changes for slides of categories infographic, document and video, we attempt to fetch
     * some metadata on YouTube / Google Drive (such as a name, a title, a duration, ...).
     */
    async onChangeUrl(url) {
        this._alertRemove();
        this.isValidUrl = false;
        this.canSubmitForm = false;
        this.state.form.isLoading = true;
        this.state.form.url = url;
        const data = await rpc("/slides/prepare_preview/", {
            url: this.state.form.url,
            slide_category: this.props.slideCategory,
            channel_id: this.props.channelId,
        });
        this.canSubmitForm = true;
        if (data.error) {
            this._alertDisplay(data.error);
            this.state.preview.show = false;
        } else {
            if (data.info) {
                this._alertDisplay(data.info, "alert-info");
            } else {
                this._alertRemove();
            }

            this.isValidUrl = true;

            if (data.name) {
                this.state.form.slideName = data.name;
                this.state.preview.videoTitle = data.name;
                this.state.preview.hideSlideVideoTitle = false;
            } else {
                this.state.preview.hideSlideVideoTitle = true;
            }

            if (data.completion_time) {
                // hours to minutes conversion
                this.state.form.duration = Math.round(data.completion_time * 60);
            }
            if (data.image_url) {
                this.state.form.slideImage = data.image_url;
            }

            if (!data.name && !data.image_url) {
                this.state.preview.show = false;
            } else {
                this.state.preview.show = true;
            }
        }

        this.state.form.isLoading = false;
    }

    async onClickFormSubmit(forcePublished) {
        if (!this._formValidate()) {
            return;
        }
        const values = await this._formValidateGetValues(forcePublished);
        this.props.upload(values, this.props.slideCategory);
    }

    /**
     * When the user selects 'local_file' or 'external' as source type, we display the 'upload'
     * field or the 'document_google_url' / 'image_google_url' fields respectively.
     */
    onClickSourceType(isLocalSource) {
        this.state.form.isLocalSource = isLocalSource;
    }

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    // Alert messages

    /**
     * @param {string} message
     */
    _alertDisplay(message, alertClass = "alert-warning") {
        this.state.alert.message = message;
        this.state.alert.class = alertClass;
        this.state.alert.show = true;
    }

    _alertRemove() {
        this.state.alert.show = false;
        this.state.alert.message = "";
        this.state.alert.class = "";
    }

    // Category and tag SelectMenus

    /**
     * Get value for category_id and tag_ids (ORM cmd) to send to server
     */
    _getSelectMenuValues() {
        const result = {};
        // tags
        if (this.state.choices.tagIds.length > 0) {
            const tags = Object.fromEntries(
                this.state.choices.tags.map((tag) => [tag.value, tag.label])
            );
            result.tag_ids = this.state.choices.tagIds.map((tagId) =>
                this._toCreate(tagId) ? [0, 0, { name: tags[tagId] }] : [4, tagId]
            );
        }
        // category
        if (!this.defaultCategoryId) {
            if (this._toCreate(this.state.choices.categoryId)) {
                const category = this.state.choices.categories.find(
                    (cat) => cat.value === this.state.choices.categoryId
                );
                result.category_id = [0, { name: category.label }];
            } else {
                const categoryId = this.state.choices.categoryId || this._getDefaultCategoryId();
                result.category_id = [categoryId];
            }
        } else {
            result.category_id = [this.defaultCategoryId];
        }
        return result;
    }

    /**
     * Returns the id of the last section of the channel or null (no sections)
     * @returns {Number|Null}
     */
    _getDefaultCategoryId() {
        return this.state.choices.categories.length > 0
            ? this.state.choices.categories[this.state.choices.categories.length - 1].value
            : null;
    }

    /**
     * Fetch available course categories and tags
     */
    async _fetch_choices(type, domain = [], fields = ["name"]) {
        const results = await rpc(`/slides/${type}/search_read`, { fields, domain });

        return results.read_results.map((choice) => {
            return { value: choice.id, label: choice.name };
        });
    }

    /**
     * Check whether it is a new category/tag or not
     */
    _toCreate(value) {
        return typeof value === "string" && value.startsWith("temp");
    }

    // Form

    _fileReset() {
        document.getElementById("upload").value = "";
        this.file.name = false;
    }

    _formValidate() {
        this.state.form.wasValidated = true;
        return (
            document.querySelector("#o_w_slide_upload_category_form").checkValidity() &&
            this.isValidUrl
        );
    }

    /**
     * Extract values to submit from form, force the slide_category according to
     * filled values.
     * @param {boolean} forcePublished
     */
    async _formValidateGetValues(forcePublished) {
        let sourceType = "local_file";
        if (this.props.slideCategory === "video") {
            sourceType = "external"; // force external for videos
        } else {
            sourceType = this.state.form.isLocalSource ? "local_file" : "external";
        }
        const values = Object.assign(
            {
                channel_id: this.props.channelId,
                document_google_url: this.state.form.url,
                duration: this.state.form.duration,
                image_google_url: this.state.form.url,
                is_published: forcePublished,
                name: this.state.form.slideName,
                slide_category: this.props.slideCategory,
                source_type: sourceType,
                video_url: this.state.form.url,
            },
            this._getSelectMenuValues()
        ); // add tags and category

        if (this.file.type === "application/pdf") {
            Object.assign(values, {
                image_1920: document.getElementById("data_canvas").toDataURL().split(",")[1],
                slide_category: "document",
                binary_content: this.file.data,
            });
        } else if (/^image\/.*/.test(this.file.type)) {
            Object.assign(values, {
                slide_category: "infographic",
                binary_content: this.file.data,
            });
        }
        return values;
    }
}

```

## File: static\src\js\public\components\slide_upload_dialog\slide_upload_category.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="website_slides.SlideUploadCategory" owl="1">
        <div>
            <div t-if="state.alert.show" t-attf-class="alert {{state.alert.class}}"
                role="alert" t-out="state.alert.message" />
            <form
                id="o_w_slide_upload_category_form"
                t-att-class="{'clearfix': true, 'was-validated': state.form.wasValidated}"
                t-on-submit.prevent="(ev) => ev.preventDefault()"
            >
                <div class="row">
                    <!-- Left column: slide upload form -->
                    <div class="col-md-6">
                        <SlideUploadSourceTypes
                            t-if="props.slideCategory in sourceSettings"
                            attributes="sourceSettings[props.slideCategory]"
                            isLocalSource="state.form.isLocalSource"
                            onClickSourceType.bind="onClickSourceType"
                            onChangeFileInput.bind="onChangeFileInput"
                            onChangeUrl.bind="onChangeUrl"
                        />
                        <canvas id="data_canvas" class="d-none"/>
                        <t t-call="website_slides.UploadFormCommonInputs"/>
                    </div>
                    <!-- Right column: preview and tutorial -->
                    <div class="col-md-6">
                        <div class="img-thumbnail h-100">
                            <div
                                t-if="state.preview.show"
                                class="o_slide_preview h-100 flex-column justify-content-center"
                            >
                                <img
                                    referrerPolicy="no-referrer"
                                    t-att-src="state.form.slideImage"
                                    id="slide-image"
                                    title="Content Preview"
                                    alt="Content Preview"
                                    class="img-fluid"
                                />
                                <div
                                    t-att-class="{'d-none': state.preview.hideSlideVideoTitle, 'mt-1': true}"
                                    t-out="state.preview.videoTitle"
                                />
                            </div>
                            <div t-else="" class="p-3">
                                <t t-slot="tutorial"/>
                            </div>
                        </div>
                    </div>
                </div>
            </form>
        </div>
        <t t-if="state.form.isLoading" t-call="website_slides.UploadDialogLoading"/>
        <t t-call="website_slides.UploadBtns"/>
    </t>

    <!-- Slide Upload Category common part templates -->

    <t t-name="website_slides.UploadBtns">
        <div t-portal="'#o_w_slide_upload_btns'">
            <button
                t-if="props.canPublish"
                class="btn btn-primary o_w_slide_upload_published"
                t-on-click="() => this.onClickFormSubmit(true)"
            >
                <span>Save and Publish</span>
            </button>
            <button
                t-if="props.canUpload"
                t-att-class="{ 'btn': true, 'btn-primary': !props.canPublish }"
                t-on-click="() => this.onClickFormSubmit(false)"
            >
                <span>Save</span>
            </button>
        </div>
    </t>

    <t t-name="website_slides.UploadFormCommonInputs">
        <div class="mb-3">
            <label for="name" class="col-form-label">Title</label>
            <input
                t-model="state.form.slideName"
                id="name"
                name="name"
                placeholder="Title"
                class="form-control"
                required="required"
                autocomplete="off"
            />
        </div>
        <div t-if="!state.defaultCategoryID" class="mb-3">
            <label for="category_id" class="col-form-label">Section</label>
            <SelectMenu
                choices="state.choices.categories"
                value="state.choices.categoryId"
                onSelect.bind="onCategorySelect"
                togglerClass="'text-dark pe-1'"
            >
                <t t-out="displayCategoryValue"/>
                <t t-set-slot="bottomArea" t-slot-scope="select">
                    <DropdownItem
                            t-if="select.data.searchValue and !this.choiceExists(select.data.searchValue, select.data.choices)"
                            class="'btn text-primary'"
                            onSelected="() => this.onClickCreateCategoryBtn(select.data.searchValue)"
                        >
                            Create New Category "<i t-out="select.data.searchValue"/>"
                    </DropdownItem>
                </t>
            </SelectMenu>
        </div>
        <div class="mb-3">
            <label for="tag_ids" class="col-form-label">Tags</label>
            <SlideUploadSelectTags
                choices="state.choices.tags"
                multiSelect="true"
                value="state.choices.tagIds"
                onSelect.bind="onTagsSelect"
                togglerClass="'text-dark pe-1'"
            >
                <t t-set-slot="bottomArea" t-slot-scope="select">
                    <DropdownItem
                            t-if="select.data.searchValue and !this.choiceExists(select.data.searchValue, select.data.choices)"
                            class="'btn text-primary'"
                            onSelected="() => this.onClickCreateTagBtn(select.data.searchValue)"
                        >
                            Create New Tag "<i t-out="select.data.searchValue"/>"
                    </DropdownItem>
                </t>
            </SlideUploadSelectTags>
        </div>
        <div class="mb-3">
            <label for="duration" class="col-form-label">Estimated Completion Time</label>
            <div class="input-group">
                <input
                    t-model="state.form.duration"
                    type="number"
                    id="duration"
                    min="0"
                    name="duration"
                    placeholder='e.g. "15"'
                    class="form-control"
                    autocomplete="off"
                />
                <span class="input-group-text">Minutes</span>
            </div>
        </div>
    </t>

    <t t-name="website_slides.UploadDialogLoading">
        <div
            t-portal="'.o_w_slide_upload_dialog_content'"
            class="o_wslides_slide_upload_loading position-absolute h-100 w-100 text-center d-flex flex-column justify-content-center"
        >
            <h2 class="text-white">
                <span class="fa fa-spinner fa-pulse"/>
                <span>Loading content...</span>
            </h2>
        </div>
    </t>

</templates>

```

## File: static\src\js\public\components\slide_upload_dialog\slide_upload_dialog.js

```javascript
/** @odoo-module **/

import { Component, onMounted, useState } from "@odoo/owl";
import { Dialog } from "@web/core/dialog/dialog";
import { DropdownItem } from "@web/core/dropdown/dropdown_item";
import { redirect } from "@web/core/utils/urls";
import { SelectMenu } from "@web/core/select_menu/select_menu";
import { ModuleToInstallIcon, SlideCategoryIcon } from "./slide_upload_dialog_select";
import { SlideInstallModule } from "./slide_install_module";
import { SlideUploadCategory } from "./slide_upload_category";
import { _t } from "@web/core/l10n/translation";
import { useService } from "@web/core/utils/hooks";
import { rpc } from "@web/core/network/rpc";

export class SlideUploadDialog extends Component {
    static baseSettings = {
        modulesToInstallMsg: "",
        page: "select",
        size: "md",
        alertMsg: "",
        title: _t("Add Content"),
        installModuleData: null,
    };
    static categoryData = {
        document: { icon: "fa-file-pdf-o", label: _t("Document") },
        infographic: { icon: "fa-file-image-o", label: _t("Image") },
        article: { icon: "fa-file-text", label: _t("Article") },
        video: { icon: "fa-file-video-o", label: _t("Video") },
        quiz: { icon: "fa-question-circle", label: _t("Quiz") },
    };
    static components = {
        Dialog,
        DropdownItem,
        ModuleToInstallIcon,
        SelectMenu,
        SlideCategoryIcon,
        SlideUploadCategory,
        SlideInstallModule,
    };
    static pagesTemplates = {
        article: "website_slides.SlideCategoryTutorial.Article",
        document: "website_slides.SlideCategoryTutorial.Document",
        infographic: "website_slides.SlideCategoryTutorial.Infographic",
        select: "website_slides.SlideUploadDialogSelect",
        install_module: "website_slides.UploadDialogInstallModule",
        upload: "website_slides.UploadInProgressDialog",
        video: "website_slides.SlideCategoryTutorial.Video",
        quiz: "website_slides.SlideCategoryTutorial.Quiz",
    };
    static props = {
        canPublish: Boolean,
        canUpload: Boolean,
        categoryId: { type: String, optional: true },
        channelId: Number,
        close: Function,
        modulesToInstall: { type: Array, optional: true },
        openModal: { type: String, optional: true },
    };
    static template = "website_slides.SlideUploadDialog";

    setup() {
        this.defaultCategoryID = parseInt(this.props.categoryId, 10);
        this.modulesToInstallStatus = null;
        this.dialog = useService("dialog");
        this.orm = useService("orm");
        this.pagesTemplates = this.constructor.pagesTemplates;
        this.slideCategoryData = this.constructor.categoryData;
        this.state = useState({ ...this.constructor.baseSettings });
        onMounted(() => {
            if (this.props.openModal && this.props.openModal in this.slideCategoryData) {
                // Sets the appropriate category's upload template if one has to be opened on load.
                this.onClickSlideCategoryIcon(this.props.openModal);
            }
        });
    }

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    onClickSlideCategoryIcon(slideCategory) {
        this.state.page = slideCategory;
        this.state.size = "lg";
    }

    onClickInstallModuleIcon(moduleId) {
        this.state.page = "install_module";
        this.state.installModuleData = this.props.modulesToInstall.find((m) => m.id === moduleId);
        this.state.size = "md";
    }

    onClickGoBack() {
        Object.assign(this.state, SlideUploadDialog.baseSettings);
    }

    /**
     * Show the upload page while processing new slide submission
     */
    async uploadSlide(formValues, previousPage) {
        this.state.page = "upload";
        this.state.size = "md";
        const data = await rpc("/slides/add_slide", formValues);
        if (data.error) {
            this.state.page = previousPage;
            this.state.size = "lg";
            this.state.alertMsg = data.error;
            return;
        }
        if (data.url.includes("enable_editor")) {
            // If we need to enter edit mode, it should be done to the top
            // window so that we end up refreshing the backend client action
            // in edit mode.
            const { origin, pathname } = window.top.location;
            const url = new URL(data.url, `${origin}${pathname}`);
            if (url.origin === origin) {
                window.top.location = url.href;
            }
        } else {
            redirect(data.url);
        }
    }
}

```

## File: static\src\js\public\components\slide_upload_dialog\slide_upload_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="website_slides.SlideUploadDialog" owl="1">
        <Dialog size="state.size" title="state.title" contentClass="'o_w_slide_upload_dialog_content'">
            <t t-set-slot="footer">
                <button t-if="state.page ==='select'" type="button" class="btn" t-on-click="() => this.props.close()">
                    <span>Cancel</span>
                </button>
                <t t-elif="!['select', 'upload'].includes(state.page)">
                    <div id="o_w_slide_upload_btns"/>
                    <button class="btn" t-on-click="onClickGoBack">
                        <span>Back</span>
                    </button>
                </t>
            </t>
            <div class="o_w_slide_upload_modal_container">
                <SlideUploadCategory
                    t-if="Object.keys(slideCategoryData).includes(state.page)"
                    alertMsg="state.alertMsg"
                    categoryId="props.categoryId"
                    channelId="props.channelId"
                    slideCategory="state.page"
                    canPublish="props.canPublish"
                    canUpload="props.canUpload"
                    upload.bind="uploadSlide"
                >
                    <t t-set-slot="tutorial">
                        <t t-call="{{pagesTemplates[state.page]}}"/>
                    </t>
                </SlideUploadCategory>
                <t t-else="">
                    <t t-call="{{pagesTemplates[state.page]}}"/>
                </t>
            </div>
        </Dialog>
    </t>

    <!-- Slide Category templates -->
    <t t-name="website_slides.SlideCategoryTutorial.Document">
        <div class="h5">How do I add new content?</div>
        <div>
            <span>You can either upload a file from your computer or insert a Google Drive link.</span><br/>
        </div>
        <div class="h5 mt-3">What types of documents do we support?</div>
        <div>
            <span>When using local files, we only support PDF files.</span><br/>
            <span>If you want to use other types of files, you may want to use an external source (Google Drive) instead.</span>
        </div>
        <div class="h5 mt-3">How to upload your PowerPoint Presentations or Word Documents?</div>
        <div>
            <span>Save your presentations or documents as PDF files and upload them.</span>
        </div>
        <div>
            Through Google Drive, we support most common types of documents.
            Including regular documents (Google Doc, .docx), Sheets (Google Sheet, .xlsx), PowerPoints, ...
        </div>
        <div class="h5 mt-3">How to use Google Drive?</div>
        <div>
            <span>First, upload the file on your Google Drive account.</span><br/>
            <span>Then, go into the file permissions and set it as "Anyone with the link".</span><br/>
            <span>The Google Drive link to use here can be obtained by clicking the "Share" button in the Google interface.</span><br/>
            <span>It should look similar to
            <span class="fst-italic">https://drive.google.com/file/d/ABC/view?usp=sharing</span></span>
        </div>
    </t>

    <t t-name="website_slides.SlideCategoryTutorial.Infographic">
        <div class="h5">How do I add new content?</div>
        <div>
            <span>You can either upload a file from your computer or insert a Google Drive link.</span><br/>
            <span>The Google Drive link can be obtained by using the 'share' button in the Google interface.</span><br/>
            <span>It should look similar to
            <span class="fst-italic">https://drive.google.com/file/d/ABC/view?usp=sharing</span></span>
        </div>
    </t>

    <t t-name="website_slides.SlideCategoryTutorial.Article">
        <div class="h5">How to create a Lesson as an Article?</div>
        <div>First, create your lesson, then edit it with the website builder. You'll be able to drop building blocks on your page and edit them.</div>
    </t>

    <t t-name="website_slides.SlideCategoryTutorial.Video">
        <div class="p-3">
            <div class="h5">How to upload your videos?</div>
            <div class="h6">On YouTube</div>
            <div>First, upload your videos on YouTube and mark them as <strong>unlisted</strong>. This way, they will be secured.</div>
            <div>What does <strong>unlisted</strong> means? The YouTube "unlisted" means it is a video which can be viewed only by the users with the link to it. Your video will never come up in the search results nor on your channel.</div>
            <div><a href="https://support.google.com/youtube/answer/157177" target="_blank" >Change video privacy settings</a></div>
            <br/>
            <div class="h6">On Vimeo</div>
            <div>
                <span>First, upload your videos on Vimeo and mark them as <strong>Private</strong>. This way, they will be secured.</span><br/>
                <span>What does <strong>Private</strong> mean? The Vimeo "Private" privacy setting means it is a video which can be viewed only by the users with the link to it.
                Your video will never come up in the search results nor on your channel.</span><br/>
                <span><a href="https://vimeo.zendesk.com/hc/en-us/articles/224819527-Changing-the-privacy-settings-of-your-videos" target="_blank" >Change video privacy settings</a></span><br/><br/>
                <span>The video link to input here can be obtained by using the 'share' button in the Vimeo interface.</span><br/>
                <span>It should look similar to </span>
                <span class="fst-italic">https://vimeo.com/558907333/30da9ff3d8</span>
                <span> for 'Private' videos and similar to</span>
                <span class="fst-italic">https://vimeo.com/558907555</span>
                <span> for public ones.</span>
            </div>
            <br/>
            <div class="h6">On Google Drive</div>
            <div>
                <span>The Google Drive link can be obtained by using the 'share' button in the Google interface.</span><br/>
                <span>It should look similar to
                <span class="fst-italic">https://drive.google.com/file/d/ABC/view?usp=sharing</span></span>
            </div>
        </div>
    </t>

    <t t-name="website_slides.SlideCategoryTutorial.Quiz">
        <div class="h5">Test your students with small Quizzes</div>
        <div>With Quizzes you can keep your students focused and motivated by answering some questions and gaining some karma points</div>
        <img src="/website_slides/static/src/img/onboarding-quiz.png" title="Quiz Demo Data" class="img-fluid"/>
    </t>

    <!-- Misc -->
    
    <t t-name="website_slides.UploadInProgressDialog">
        <div class="text-center" role="status">
            <div class="fa-3x">
                <i class="fa fa-circle-o-notch fa-spin"></i>
            </div>
            <h4>Uploading document ...</h4>
        </div>
    </t>

    <t t-name="website_slides.UploadDialogInstallModule">
        <SlideInstallModule moduleData="this.state.installModuleData"/>
    </t>

</templates>

```

## File: static\src\js\public\components\slide_upload_dialog\slide_upload_dialog_select.js

```javascript
/** @odoo-module **/

import { Component } from "@odoo/owl";

export class ModuleToInstallIcon extends Component {
    static template = "website_slides.ModuleToInstallIcon";
    static props = {
        title: String,
        moduleId: Number,
        motivational: String,
        onClickInstallModuleIcon: Function,
    };
}

export class SlideCategoryIcon extends Component {
    static template = "website_slides.SlideCategoryIcon";
    static props = {
        slideCategory: String,
        categoryData: {
            type: Object,
            shape: {
                icon: String,
                label: String,
            },
        },
        onClickSlideCategoryIcon: Function,
    };
}

```

## File: static\src\js\public\components\slide_upload_dialog\slide_upload_dialog_select.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <!-- Slide Category Selection templates -->

    <t t-name="website_slides.SlideUploadDialogSelect">
        <div class="row p-1 mt-4 mb-2">
            <div
                t-foreach="slideCategoryData"
                t-as="slide_category"
                t-key="slide_category_index"
                class="col-6 col-md-4"
            >
                <SlideCategoryIcon
                    slideCategory="slide_category"
                    categoryData="slideCategoryData[slide_category]"
                    onClickSlideCategoryIcon.bind="onClickSlideCategoryIcon"
                />
            </div>
        </div>
        <ModuleToInstallIcon
            t-if="props.modulesToInstall.length"
            t-foreach="props.modulesToInstall"
            t-as="module_info"
            t-key="module_info_index"
            title="module_info['name']"
            moduleId="module_info['id']"
            motivational="module_info['motivational']"
            onClickInstallModuleIcon.bind="onClickInstallModuleIcon"
        />
    </t>

    <t t-name="website_slides.SlideCategoryIcon" owl="1">
        <a
            href="#"
            t-on-click="() => props.onClickSlideCategoryIcon(props.slideCategory)"
            t-att-data-slide-category="props.slideCategory"
            class="content-type d-flex flex-column align-items-center mb-4 btn rounded border text-600 p-3"
        >
            <i t-attf-class="fa #{props.categoryData.icon} mb-2 fa-3x"/>
            <t t-out="props.categoryData.label"/>
        </a>
    </t>

    <t t-name="website_slides.ModuleToInstallIcon" owl="1">
        <a class="w-100 text-center mb-4 btn rounded border text-600 p-3"
            href="#" t-on-click="() => props.onClickInstallModuleIcon(props.moduleId)"
            t-att-title="props.title"
            t-att-data-module-id="props.moduleId">
            <i class="fa fa-trophy"/> <t t-out="props.motivational"/><span class="text-primary"> Install the <t t-out="props.title"/> app.</span>
        </a>
    </t>
</templates>

```

## File: static\src\js\public\components\slide_upload_dialog\slide_upload_select_tags.js

```javascript
/** @odoo-module **/

import { SelectMenu } from "@web/core/select_menu/select_menu";

export class SlideUploadSelectTags extends SelectMenu {}
SlideUploadSelectTags.template = "website_slides.SlideUploadSelectTags";

```

## File: static\src\js\public\components\slide_upload_dialog\slide_upload_select_tags.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="website_slides.SlideUploadSelectTags" t-inherit="web.SelectMenu" t-inherit-mode="primary">
        <xpath expr="//TagsList" position="after">
            <t t-if="props.value.length === 0">Select or create tags</t>
        </xpath>
    </t>
</templates>

```

## File: static\src\js\public\components\slide_upload_dialog\slide_upload_source_types.js

```javascript
/** @odoo-module **/

import { Component, useState } from "@odoo/owl";

export class SlideUploadSourceTypes extends Component {
    static props = {
        attributes: {
            type: Object,
            shape: {
                sourceTypeLabel: { type: String, optional: true },
                selectFileLabel: { type: String, optional: true },
                acceptedFiles: { type: String, optional: true },
                urlInputLabel: String,
                urlInputName: String,
            },
        },
        isLocalSource: Boolean,
        onClickSourceType: Function,
        onChangeFileInput: Function,
        onChangeUrl: Function,
    };
    static template = "website_slides.SlideUploadSourceTypes";

    setup() {
        this.state = useState({ url: "" });
    }
}

```

## File: static\src\js\public\components\slide_upload_dialog\slide_upload_source_types.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="website_slides.SlideUploadSourceTypes" owl="1">
        <div t-if="props.attributes.urlInputName === 'video_url'" class="mb-3">
            <label
                t-att-for="props.attributes.urlInputName"
                class="col-form-label"
                t-out="props.attributes.urlInputLabel"
            />
            <input
                t-model="state.url"
                t-att-id="props.attributes.urlInputName"
                t-att-name="props.attributes.urlInputName"
                class="form-control"
                autocomplete="off"
                placeholder='e.g "https://www.youtube.com/watch?v=ebBez6bcSEc"'
                required="required"
                t-on-change="() => props.onChangeUrl(state.url)"
            />
        </div>
        <t t-else="">
            <div class="mb-3">
                <label
                    for="source_type"
                    class="col-form-label"
                    t-out="props.attributes.sourceTypeLabel"
                /><br/>
                <div class="form-check ms-2">
                    <input
                        class="form-check-input"
                        type="radio"
                        name="source_type"
                        id="source_type_local_file"
                        data-value="local_file"
                        checked="checked"
                        t-on-click="() => props.onClickSourceType(true)"
                    />
                    <label
                        class="form-check-label fw-normal"
                        for="source_type_local_file"
                    >
                        Upload from Device
                    </label>
                </div>
                <div class="form-check ms-2">
                    <input
                        class="form-check-input"
                        type="radio"
                        name="source_type"
                        id="source_type_external"
                        data-value="external"
                        t-on-click="() => props.onClickSourceType(false)"
                    />
                    <label
                        class="form-check-label fw-normal"
                        for="source_type_external"
                    >
                        Retrieve from Google Drive
                    </label>
                </div>
            </div>
            <div t-if="props.isLocalSource" class="mb-3">
                <label
                    for="upload"
                    class="col-form-label"
                    t-out="props.attributes.selectFileLabel"
                />
                <input
                    id="upload"
                    name="file"
                    class="form-control h-100"
                    t-att-accept="props.attributes.acceptedFiles"
                    type="file"
                    required="required"
                    t-on-change.prevent="(ev) => props.onChangeFileInput(ev)"
                />
            </div>
            <div t-else="" class="mb-3">
                <label
                    t-att-for="props.attributes.urlInputName"
                    class="col-form-label"
                    t-out="props.attributes.urlInputLabel"
                />
                <input
                    t-model="state.url"
                    t-att-id="props.attributes.urlInputName"
                    t-att-name="props.attributes.urlInputName"
                    class="form-control h-100"
                    autocomplete="off"
                    placeholder='e.g "https://drive.google.com/file/..."'
                    required="required"
                    t-on-change="() => props.onChangeUrl(state.url)"
                />
            </div>
        </t>
    </t>
</templates>

```

## File: static\src\js\systray_items\new_content.js

```javascript
/** @odoo-module **/

import { NewContentModal, MODULE_STATUS } from '@website/systray_items/new_content';
import { patch } from "@web/core/utils/patch";

patch(NewContentModal.prototype, {
    setup() {
        super.setup();

        const newSlidesChannelElement = this.state.newContentElements.find(element => element.moduleXmlId === 'base.module_website_slides');
        newSlidesChannelElement.createNewContent = () => this.onAddContent('website_slides.slide_channel_action_add');
        newSlidesChannelElement.status = MODULE_STATUS.INSTALLED;
        newSlidesChannelElement.model = 'slide.channel';
    },
});

```

## File: static\src\js\tours\slides_tour.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registerWebsitePreviewTour } from '@website/js/tours/tour_utils';

import { markup } from "@odoo/owl";

registerWebsitePreviewTour('slides_tour', {
    url: '/slides',
}, () => [{
    trigger: "body:not(.editor_has_snippets) .o_new_content_container > a",
    content: markup(_t("Welcome on your course's home page. It's still empty for now. Click on \"<b>New</b>\" to write your first course.")),
    tooltipPosition: 'bottom',
    run: "click",
}, {
    trigger: 'a[data-module-xml-id="base.module_website_slides"]',
    content: markup(_t("Select <b>Course</b> to create it and manage it.")),
    tooltipPosition: 'bottom',
    run: "click",
}, {
    trigger: 'input#name_0',
    content: markup(_t("Give your course an engaging <b>Title</b>.")),
    tooltipPosition: 'bottom',
    run: "edit My New Course",
}, {
    trigger: 'div[name="description"] div[contenteditable=true]',
    content: markup(_t("Give your course a helpful <b>Description</b>.")),
    tooltipPosition: 'bottom',
    run: "edit This course is for advanced users.",
}, {
    trigger: 'button.btn-primary',
    content: markup(_t("Click on the <b>Save</b> button to create your first course.")),
    run: "click",
}, {
    trigger: ':iframe .o_wslides_js_slide_section_add',
    content: markup(_t("Congratulations, your course has been created, but there isn't any content yet. First, let's add a <b>Section</b> to give your course a structure.")),
    tooltipPosition: 'bottom',
    run: "click",
}, {
    trigger: ':iframe #section_name',
    content: markup(_t("A good course has a structure. Pick a name for your first <b>Section</b>.")),
    tooltipPosition: 'bottom',
    run: "click",
}, {
    trigger: ':iframe button.btn-primary:contains("Save")',
    content: markup(_t("Click <b>Save</b> to create it.")),
    tooltipPosition: 'bottom',
    run: "click",
}, {
    trigger: '.o_iframe:iframe a.btn-primary.o_wslides_js_slide_upload',
    content: markup(_t("Your first section is created, now it's time to add lessons to your course. Click on <b>Add Content</b> to upload a document, create an article or link a video.")),
    tooltipPosition: 'bottom',
    run: "click",
}, {
    trigger: ':iframe a[data-slide-category="document"]',
    content: markup(_t("First, let's add a <b>Document</b>. It has to be a .pdf file.")),
    tooltipPosition: 'bottom',
    run: "click",
}, {
    trigger: ':iframe input#upload',
    content: markup(_t("Choose a <b>File</b> on your computer.")),
    run: "click",
}, {
    trigger: ':iframe input#name',
    content: markup(_t("The <b>Title</b> of your lesson is autocompleted but you can change it if you want.</br>A <b>Preview</b> of your file is available on the right side of the screen.")),
    run: "click",
}, {
    trigger: ':iframe input#duration',
    content: markup(_t("The <b>Duration</b> of the lesson is based on the number of pages of your document. You can change this number if your attendees will need more time to assimilate the content.")),
    run: "click",
}, {
    trigger: ':iframe button.o_w_slide_upload_published',
    content: markup(_t("<b>Save & Publish</b> your lesson to make it available to your attendees.")),
    tooltipPosition: 'bottom',
    run: "click",
}, {
    trigger: '.o_iframe:iframe span.badge:contains("New")',
    content: markup(_t("Congratulations! Your first lesson is available. Let's see the options available here. The tag \"<b>New</b>\" indicates that this lesson was created less than 7 days ago.")),
    tooltipPosition: 'bottom',
    run: "click",
}, {
    trigger: ':iframe a[name="o_wslides_list_slide_add_quizz"]',
    content: markup(_t("If you want to be sure that attendees have understood and memorized the content, you can add a Quiz on the lesson. Click on <b>Add Quiz</b>.")),
    run: "click",
}, {
    trigger: ':iframe input[name="question-name"]',
    content: markup(_t("Enter your <b>Question</b>. Be clear and concise.")),
    tooltipPosition: 'left',
    run: "click",
}, {
    trigger: ':iframe input.o_wslides_js_quiz_answer_value',
    content: markup(_t("Enter at least two possible <b>Answers</b>.")),
    tooltipPosition: 'left',
    run: "click",
}, {
    trigger: ':iframe a.o_wslides_js_quiz_is_correct',
    content: markup(_t("Mark the correct answer by checking the <b>correct</b> mark.")),
    tooltipPosition: 'right',
    run: "click",
}, {
    trigger: ':iframe i.o_wslides_js_quiz_comment_answer:last',
    content: markup(_t("You can add <b>comments</b> on answers. This will be visible with the results if the user select this answer.")),
    tooltipPosition: 'right',
    run: "click",
}, {
    trigger: ':iframe a.o_wslides_js_quiz_validate_question',
    content: markup(_t("<b>Save</b> your question.")),
    tooltipPosition: 'left',
    run: "click",
}, {
    trigger: '.o_iframe:iframe li.breadcrumb-item:nth-child(2)',
    content: markup(_t("Click on your <b>Course</b> to go back to the table of content.")),
    tooltipPosition: 'top',
    run: "click",
}, {
    trigger: '.o_menu_systray_item.o_website_publish_container a',
    content: markup(_t("Once you're done, don't forget to <b>Publish</b> your course.")),
    tooltipPosition: 'bottom',
    run: "click",
}, {
    trigger: ':iframe a.o_wslides_js_slides_list_slide_link',
    content: markup(_t("Congratulations, you've created your first course.<br/>Click on the title of this content to see it in fullscreen mode.")),
    tooltipPosition: 'bottom',
    run: "click",
}, {
    trigger: ':iframe .o_wslides_fs_toggle_sidebar',
    content: markup(_t("Finally you can click here to enjoy your content in fullscreen")),
    tooltipPosition: 'bottom',
    run: "click",
}]);

```

## File: static\src\views\slide_channel_partner_list\slide_channel_partner_list_controller.js

```javascript
/** @odoo-module **/

import { ListController } from '@web/views/list/list_controller';
import { useService } from '@web/core/utils/hooks';


export default class SlideChannelPartnerListController extends ListController {
    static template = "website_slides.SlideChannelPartnerListView";
    setup() {
        super.setup();
        this.action = useService('action');
        this.orm = useService('orm');
        this.channelId = this.props.context.default_channel_id || false;
    }

    /**
     * Method opening the wizard to enroll new slide channel partners.
     * Reloads the model afterwards to see new attendees.
     * 
     * @private
     */
    async _openEnrollWizard() {
        const action = await this.orm.call(
            'slide.channel',
            'action_channel_enroll',
            [this.channelId]
        );
        this.action.doAction(action, {
            onClose: async () => {
                await this.model.load();
                this.model.useSampleModel = false;
                this.render(true);
            }
        });
    }
}

```

## File: static\src\views\slide_channel_partner_list\slide_channel_partner_list_view.js

```javascript
/** @odoo-module **/

import { listView } from '@web/views/list/list_view';
import { registry } from '@web/core/registry';

import SlideChannelPartnerListController from './slide_channel_partner_list_controller.js';

export const SlideChannelPartnerListView = {
    ...listView,
    Controller: SlideChannelPartnerListController,
};

registry.category('views').add('slide_channel_partner_enroll_tree', SlideChannelPartnerListView);

```

## File: static\src\views\slide_channel_partner_list\slide_channel_partner_list_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates xml:space="preserve">

    <t t-inherit="web.ListView" t-inherit-mode="primary" t-name="website_slides.SlideChannelPartnerListView">
        <xpath expr="//t[@t-foreach='archInfo.headerButtons']" position="after">
            <button t-if="channelId" type="button" class="btn btn-primary" t-on-click="_openEnrollWizard">
                New
            </button>
        </xpath>
    </t>

</templates>

```

## File: static\src\xml\slide_course_join.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="slide.course.join">
        <div>
            <a role="button"
                class="btn btn-primary o_wslides_js_course_join_link text-uppercase fw-bold"
                title="Join the Course" aria-label="Join the Course"
                href="#">
                <t t-if="widget.channel.channelEnroll == 'public'" t-esc="widget.joinMessage"/>
                <t t-if="widget.channel.channelEnroll == 'invite' and widget.isMemberOrInvited" t-esc="widget.joinMessage"/>
            </a>
        </div>
    </t>

    <t t-name="slide.course.join.popupContent">
        <div t-if="widget.invitePreview">
            Please <a t-attf-href="/slides/#{channelId}/identify?invite_partner_id=#{widget.invitePartnerId}&amp;invite_hash=#{widget.inviteHash}">
            <t t-if="widget.isPartnerWithoutUser">create an account</t><t t-else="">login</t>
            </a> to join this course
        </div>
        <div t-else="">
            <t t-if="errorSignupAllowed">
                Please <a t-attf-href="/web/login?redirect=#{courseUrl}">login</a> or <a t-attf-href="/web/signup?redirect=#{courseUrl}">create an account</a> to join this course
            </t>
            <t t-else="">
                Please <a t-attf-href="/web/login?redirect=#{courseUrl}">login</a> to join this course
            </t>
        </div>
    </t>
</templates>

```

## File: static\src\xml\slide_course_prerequisite.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="slide.course.prerequisite">
        <ul class="m-0 ps-3">
            <li t-foreach="channels" t-as="channel" t-key="channel.course_id">
                <a t-attf-href="/slides/#{channel.course_id}" t-out="channel.course_name"/>
            </li>
        </ul>
    </t>
</templates>

```

## File: static\src\xml\slide_management.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="slides.slide.archive">
        <div>
            <p>Are you sure you want to archive this content?</p>
        </div>
    </t>

    <t t-name="slides.category.delete">
        <div>
            <p>Are you sure you want to delete this category?</p>
        </div>
    </t>

</templates>

```

## File: static\src\xml\slide_quiz.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="slide.slide.quiz">
        <div class="o_wslides_fs_quiz_container o_wslides_wrap h-100 w-100 overflow-auto pb-5">
            <div class="container">
                <div t-if="widget.quiz.description_safe" t-out="widget.quiz.description_safe" class="mt-3" />
                <div t-foreach="widget.quiz.questions" t-as="question" t-key="question_index"
                     t-attf-class="o_wslides_js_lesson_quiz_question mt-5 mb-4 #{widget.slide.completed ? 'completed-disabled' : ''}"
                     t-att-data-question-id="question.id" t-att-data-title="question.question">
                    <div class="h4">
                        <small class="text-muted"><span t-esc="question_index+1"/>. </small> <span t-esc="question.question"/>
                    </div>
                    <div class="list-group">
                        <t t-foreach="question.answer_ids" t-as="answer" t-key="answer_index">
                            <a t-att-data-answer-id="answer.id" href="#"
                                t-att-data-text="answer.text_value"
                                t-attf-class="o_wslides_quiz_answer list-group-item d-flex align-items-center list-group-item-action #{widget.slide.completed  &amp;&amp; answer.is_correct ? 'list-group-item-success' : '' }">

                                <label class="my-0 d-flex align-items-center justify-content-center me-2">
                                    <input type="radio"
                                        t-att-name="question.id"
                                        t-att-value="answer.id"
                                        class="d-none"/>
                                    <i t-att-class="'fa fa-circle text-400' + (!(widget.slide.completed &amp;&amp; answer.is_correct) ? '' : ' d-none')"></i>
                                    <i class="fa fa-times-circle text-danger d-none"></i>
                                    <i t-att-class="'fa fa-check-circle text-success' + (widget.slide.completed &amp;&amp; answer.is_correct ? '' :  ' d-none')"></i>
                                </label>
                                <span t-esc="answer.text_value"/>
                            </a>
                            <t t-if="widget.slide.completed and answer.is_correct" t-set="correct_answer_comment" t-value="answer.comment"/>
                        </t>
                        <div t-attf-class="o_wslides_quiz_answer_info list-group-item list-group-item-info #{correct_answer_comment ? '' : 'd-none'}">
                            <i class="fa fa-info-circle"/>
                            <span class="o_wslides_quiz_answer_comment ms-2">
                                <t t-if="correct_answer_comment" t-out="correct_answer_comment"/>
                            </span>
                        </div>
                    </div>
                </div>
                <div t-if="!widget.slide.completed" class="o_wslides_js_lesson_quiz_validation pt-3"/>
                <div t-else="" class="row">
                    <div class="o_wslides_js_lesson_quiz_validation col py-2 bg-100 mb-2 border-bottom"/>
                </div>
            </div>
        </div>
    </t>

    <t t-name="slide.slide.quiz.validation">
        <div id="validation">
            <div t-if="!widget.isMember">
                <div class="o_wslides_join_course alert alert-info d-flex align-items-center justify-content-between">
                    <div t-if="widget.channel.channelEnroll == 'invite' and !widget.isMemberOrInvited">
                        <b>This course is private.
                            <span t-if="widget.publicUser">
                                Please
                                <a t-att-href="'/web/login?redirect=' + widget.redirectURL" class="fw-bold">
                                    sign in
                                </a>
                                to enroll.
                            </span>
                            <a t-else="" href="#" class="fw-bold o_wslides_js_channel_enroll"
                               t-att-data-channel-id="widget.channel.channelId">
                                <span t-if="widget.channel.channelRequestedAccess" class="text-success">
                                    Responsible already contacted.
                                </span>
                                <span t-else="">
                                    Contact the responsible to enroll.
                                </span>
                            </a>
                        </b>
                        <span class="my-0 h4">
                            <span title="Succeed and gain karma" aria-label="Succeed and gain karma" class="badge bg-warning fw-bold ms-3">
                                + <t t-esc="widget.quiz.quizKarmaGain"/> XP
                            </span>
                        </span>
                    </div>
                    <div t-else="" class="w-100">
                        <b class="h5 mb-0 o_wslides_quiz_join_course_message">
                            <span t-if="widget.channel.channelEnroll == 'public' or (widget.isMemberOrInvited and widget.channel.channelEnroll == 'invite')">
                                <t t-if="widget.publicUser">
                                    Sign in and join the course to verify your answers!
                                </t>
                                <t t-else="">
                                    Join the course to take the quiz and verify your answers!
                                </t>
                            </span>
                        </b>
                        <span class="my-0 h4">
                            <span title="Succeed and gain karma" aria-label="Succeed and gain karma" class="badge bg-warning fw-bold ms-3">
                                + <t t-esc="widget.quiz.quizKarmaGain"/> XP
                            </span>
                        </span>
                        <div class="o_wslides_join_course_widget float-end"/>
                    </div>
                </div>
                <span t-if="widget.publicUser &amp;&amp; widget.channel.signupAllowed" class="d-block mt-2">
                    <span>Don't have an account?</span>
                    <a class="fw-bold" t-att-href="'/web/signup?redirect=' + widget.redirectURL">Sign Up!</a>
                </span>
            </div>
            <div t-else="" class="d-md-flex align-items-center justify-content-between">
                <div t-att-class="'d-flex align-items-center' + (widget.slide.completed ? ' alert alert-success my-0 py-1 px-3' : '')">
                    <button t-if="! widget.slide.completed" role="button" title="Check answers"
                        class="btn btn-primary text-uppercase fw-bold o_wslides_js_lesson_quiz_submit">Check your answers</button>
                    <b t-else="" class="my-0 h5">Done!</b>
                    <span class="my-0 h5" style="line-height: 1">
                        <span role="button" title="Succeed and gain karma" class="badge bg-warning fw-bold ms-3">
                            + <t t-if="!widget.slide.completed" t-esc="widget.quiz.quizKarmaGain"/><t t-else="" t-esc="widget.quiz.quizKarmaWon"/> XP
                        </span>
                    </span>
                    <div class="ms-3 d-none text-danger o_wslides_js_quiz_submit_error">
                        <i class="fa fa-close me-1"/>
                        <span class="o_wslides_js_quiz_submit_error_text"></span>
                    </div>
                </div>
                <div class="ms-auto mt-3 mt-md-0">
                    <button t-if="widget.quiz.quizAttemptsCount > 0 &amp;&amp; widget.slide.channelCanUpload" class="btn btn-light border o_wslides_js_lesson_quiz_reset me-1">
                        Reset
                    </button>
                    <button t-if="widget.slide.completed &amp;&amp; widget.slide.hasNext" class="btn btn-primary o_wslides_quiz_continue">
                        Continue <i class="oi oi-chevron-right ms-1"/>
                    </button>
                </div>
            </div>
            <div class="mt-4 o_wslides_js_lesson_quiz_resource_info d-none">
                <div t-if="widget.quiz.slideResources?.length > 0">
                    <span>Need help? Review related content:</span>
                    <ul>
                        <li t-foreach="widget.quiz.slideResources" t-as="resource" t-key="resource_index">
                            <a t-att-href="resource.link or resource.download_url" t-esc="resource.display_name"/>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\xml\slide_quiz_create.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="slide.quiz.question.input">
        <div t-attf-class="o_wsildes_quiz_question_input mt-3 #{!widget.update ? 'col' : ''}" t-att-data-id="widget.question.id || ''">
            <form class="mb-3">
                <div class="o_wslides_quiz_question row align-items-center me-0 mb-2">
                    <div class="input-group">
                        <span class="input-group-text o_wslides_quiz_question_sequence"><t t-esc="widget.sequence"/></span>
                        <input type="text" name="question-name" class="form-control col-11" placeholder='e.g. "Which animal cannot fly?"'
                            t-att-value="widget.question.text"/>
                    </div>
                </div>
                <div class="text-muted mb-2">
                    <span>Select the correct answer below:</span>
                </div>
                <div class="list-group">
                    <t t-if="widget.question.answers" >
                        <t t-foreach="widget.question.answers" t-as="answer" t-key="answer_index">
                            <t t-call="slide.quiz.answer.line"/>
                        </t>
                    </t>
                    <t t-else="" >
                        <t t-foreach="['A giraffe', 'A bird', 'A fly']" t-as="placeholder" t-key="placeholder_index">
                            <t t-call="slide.quiz.answer.line"/>
                        </t>
                    </t>
                </div>
            </form>
            <div>
                <t t-if="widget.update" t-call="slide.quiz.update.buttons"/>
                <t t-else="" t-call="slide.quiz.create.buttons"/>
                <div class="ms-2 d-none text-danger o_wslides_js_quiz_validation_error">
                    <i class="fa fa-close me-1"/>
                    <span class="o_wslides_js_quiz_validation_error_text"></span>
                </div>
            </div>
        </div>
    </t>

    <t t-name="slide.quiz.answer.line">
        <div class="o_wslides_js_quiz_answer row align-items-center mb-1" t-attf-data-answer-id="#{answer ? answer.id : ''}" >
            <div class="col ms-3 ms-md-5">
                <div class="row align-items-center">
                    <div class="col-9 p-0">
                        <div class="input-group">
                            <input type="text" class="o_wslides_js_quiz_answer_value form-control" t-attf-placeholder='e.g. "{{placeholder || "another animal"}}"' t-attf-value="#{answer ? answer.text_value : ''}"/>
                            <div class="input-group-text">
                                <a class="o_wslides_js_quiz_is_correct" title="This is the correct answer">
                                    <label class="my-0">
                                        <input t-if="answer and answer.is_correct" class="d-none" type="radio" name="radio" checked="true" />
                                        <input t-else="" class="d-none" type="radio" name="radio" />
                                        <i class="o_wslides_js_quiz_icon fa fa-lg fa-check-circle-o text-muted" />
                                    </label>
                                </a>
                            </div>
                        </div>
                    </div>
                    <i t-attf-class="o_wslides_js_quiz_icon o_wslides_js_quiz_comment_answer fa fa-lg fa-info-circle col-auto p-md-2 py-2 ps-2 pe-1 #{answer &amp;&amp; answer.comment ? 'text-primary' : 'text-muted'}" title="Add comment on this answer" />
                    <i class="o_wslides_js_quiz_icon o_wslides_js_quiz_add_answer fa fa-lg fa-plus-circle col-auto p-md-2 py-2 px-1 text-muted" title="Add an answer below this one" />
                    <i class="o_wslides_js_quiz_icon o_wslides_js_quiz_remove_answer fa fa-lg fa-trash-o col-auto p-md-2 py-2 px-1 text-muted" title="Remove this answer" />
                </div>
                <div class="o_wslides_js_quiz_answer_comment row align-items-center d-none">
                    <div class="col-8 offset-1 p-0">
                        <input type="text" class="form-control mt-1" placeholder="This is the correct answer, congratulations" t-attf-value="#{answer ? answer.comment : ''}"/>
                    </div>
                    <i class="o_wslides_js_quiz_icon o_wslides_js_quiz_remove_answer_comment fa fa-lg fa-trash-o p-2 text-muted col-auto" title="Remove the answer comment" />
                </div>
            </div>
        </div>
    </t>

    <t t-name="slide.quiz.create.buttons">
        <a class="o_wslides_js_quiz_validate_question btn btn-primary text-white border me-1" role="button">
            <span>Save</span>
        </a>
        <a class="o_wslides_js_quiz_cancel_question btn btn-light border" role="button">
            <span>Cancel</span>
        </a>
    </t>

    <t t-name="slide.quiz.update.buttons">
        <a class="o_wslides_js_quiz_validate_question o_wslides_js_quiz_update btn btn-primary text-white border me-1" role="button">
            <span>Update</span>
        </a>
        <a class="o_wslides_js_quiz_cancel_question btn btn-light border" role="button">
            <span>Cancel</span>
        </a>
    </t>

</templates>

```

## File: static\src\xml\website_slides_fullscreen.xml

```xml
<templates id="template" xml:space="preserve">

    <t t-name="website.slides.fullscreen.content">
        <t t-if="widget._slideValue.category === 'document'">
            <div class="ratio h-100">
                <iframe t-att-src="widget._slideValue.embedUrl" class="o_wslides_iframe_viewer" allowFullScreen="true" frameborder="0" aria-label="Slides"/>
            </div>
        </t>
        <t t-if="widget._slideValue.category === 'infographic'">
            <div class="o_wslides_fs_player w-100 h-100 overflow-auto d-flex align-items-start justify-content-center">
                <img t-att-src="'/web/image/slide.slide/'+ widget._slideValue.id +'/image_1024'" class="img-fluid position-relative m-auto" alt="Slide image"/>
            </div>
        </t>
    </t>

    <t t-name="website.slides.fullscreen.video.google_drive">
        <div class="player ratio ratio-16x9 embed-responsive-item h-100">
            <iframe t-att-src="widget._slideValue.embedUrl" allowFullScreen="true" frameborder="0" autoplay="1" allow="autoplay"></iframe>
        </div>
    </t>

    <t t-name="website.slides.fullscreen.video.youtube">
        <div class="player ratio ratio-16x9 embed-responsive-item h-100">
            <iframe t-att-id="'youtube-player' + widget.slide.id" t-att-src="widget.slide.embedUrl" allowFullScreen="true" frameborder="0" enablejsapi="1" autoplay="1" allow="autoplay"></iframe>
        </div>
    </t>

    <t t-name="website.slides.fullscreen.video.vimeo">
        <div class="player ratio ratio-16x9 embed-responsive-item h-100">
            <t t-out="widget.slide.embedCode"/>
        </div>
    </t>

</templates>

```

## File: static\src\xml\website_slides_sidebar.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <!-- JS Equivalent of the Python template "slide_sidebar_done_button" -->
    <t t-name="website.slides.sidebar.done.button">
        <div class="o_wslides_sidebar_done_button"
            t-att-data-id="slideId"
            t-att-data-uncompleted-icon="uncompletedIcon"
            t-att-data-completed="slideCompleted"
            t-att-data-can-self-mark-completed="canSelfMarkCompleted"
            t-att-data-can-self-mark-uncompleted="canSelfMarkUncompleted"
            t-att-data-is-member="isMember"> <!-- The template is only used for members -->
            <button class="o_wslides_button_complete btn btn-sm" t-if="slideCompleted and canSelfMarkUncompleted or !slideCompleted and canSelfMarkCompleted">
                <i t-if="slideCompleted" class="o_wslides_slide_completed fa fa-check-circle fa-fw text-success fa-lg" t-att-data-slide-id="slideId" title="Mark as not done"/>
                <i t-else="" t-attf-class="fa #{uncompletedIcon or 'fa-circle-thin'} fa-fw fa-lg" t-att-data-slide-id="slideId" title="Mark as done"/>
            </button>
            <button class="o_wslides_button_complete btn btn-sm border-0" t-else="" disabled="1">
                <i t-if="slideCompleted" class="o_wslides_slide_completed fa fa-check fa-fw text-success fa-lg" t-att-data-slide-id="slideId" title="Can not be marked as not done"/>
                <i t-else="" t-attf-class="fa #{uncompletedIcon or 'fa-circle-thin'} fa-fw fa-lg" t-att-data-slide-id="slideId" title="Can not be marked as done"/>
            </button>
        </div>
    </t>
</templates>

```

## File: views\gamification_karma_tracking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="gamification_karma_tracking_view_search" model="ir.ui.view">
        <field name="name">gamification.karma.tracking.view.search.inherit.website.slides</field>
        <field name="model">gamification.karma.tracking</field>
        <field name="inherit_id" ref="gamification.gamification_karma_tracking_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='filter_res_users']" position='after'>
                <filter string="Course" name="filter_slide_channel"
                    domain="[('origin_ref', 'ilike', 'slide.channel,')]"/>
                <filter string="Quiz" name="filter_slide_slide"
                    domain="[('origin_ref', 'ilike', 'slide.slide,')]"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\rating_rating_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="rating_rating_view_search_slide_channel" model="ir.ui.view">
        <field name="name">rating.rating.view.search.slides</field>
        <field name="model">rating.rating</field>
        <field name="priority">64</field>
        <field name="inherit_id" ref="rating.rating_rating_view_search"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='rating_text']" position="after">
                <filter string="Course" name="groupby_course" context="{'group_by': 'res_name'}"/>
            </xpath>
            <filter name="filter_create_date" position="attributes">
                <attribute name="default_period">custom_create_date_last_30_days</attribute>
            </filter>
            <xpath expr="//filter[@name='my_ratings']" position="replace"/>
            <xpath expr="//filter[@name='responsible']" position="replace"/>
            <xpath expr="//filter[@name='resource']" position="replace"/>
        </field>
    </record>

    <record id="rating_rating_view_graph_slide_channel" model="ir.ui.view">
        <field name="name">rating.rating.view.graph.slides</field>
        <field name="model">rating.rating</field>
        <field name="priority">64</field>
        <field name="arch" type="xml">
            <graph string="Reviews" type="bar" sample="1">
                <field name="res_name" invisible="1"/>
                <field name="rating" type="measure"/>
                <field name="res_id" invisible="1"/>
                <field name="parent_res_id" invisible="1"/>
            </graph>
        </field>
    </record>

    <record id="rating_rating_view_pivot_slide_channel" model="ir.ui.view">
        <field name="name">rating.rating.view.pivot.slides</field>
        <field name="model">rating.rating</field>
        <field name="priority">64</field>
        <field name="arch" type="xml">
            <pivot sample="1">
                <field name="res_name" type="row"/>
                <field name="rating_text" type="col"/>
                <field name="rating" type="measure"/>
            </pivot>
        </field>
    </record>

    <record id="rating_rating_view_tree_slide_channel" model="ir.ui.view">
        <field name="name">rating.rating.view.list.slides</field>
        <field name="model">rating.rating</field>
        <field name="priority">64</field>
        <field name="arch" type="xml">
            <list create="0">
                <field name="create_date" string="Review Date"/>
                <field name="partner_id"/>
                <field name="res_name" string="Course"/>
                <field name="rating" string="Score"/>
                <field name="feedback"/>
            </list>
        </field>
    </record>

    <record id="rating_rating_action_slide_channel" model="ir.actions.act_window">
        <field name="name">Reviews</field>
        <field name="res_model">rating.rating</field>
        <field name="domain">[('consumed', '=', True), ('res_model', '=', 'slide.channel')]</field>
        <field name="context">{}</field>
        <field name="view_mode">kanban,list,graph,pivot,form</field>
        <field name="search_view_id" ref="rating_rating_view_search_slide_channel"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No Reviews yet!
            </p>
            <p>Come back later to check the feedbacks given by your Attendees.</p>
        </field>
    </record>

    <record id="rating_rating_view_form_slides" model="ir.ui.view">
        <field name="name">rating.rating.view.form.slides</field>
        <field name="model">rating.rating</field>
        <field name="priority">64</field>
        <field name="active" eval="True"/>
        <field name="arch" type="xml">
            <form string="Ratings" create="false">
                <sheet>
                    <group>
                        <group>
                            <field name="partner_id"/>
                            <field name="resource_ref" string="Course"/>
                            <field name="create_date"/>
                        </group>
                        <group>
                            <field name="rating"/>
                            <field name="is_internal"/>
                        </group>
                    </group>
                    <group class="mw-100" invisible="not feedback">
                        <field name="feedback"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="rating_rating_action_slide_channel_view_kanban" model="ir.actions.act_window.view">
        <field name="act_window_id" ref="rating_rating_action_slide_channel"/>
        <field name="sequence">1</field>
        <field name="view_mode">kanban</field>
        <field name="view_id" ref="rating.rating_rating_view_kanban_stars"/>
    </record>

    <record id="rating_rating_action_slide_channel_view_graph" model="ir.actions.act_window.view">
        <field name="act_window_id" ref="rating_rating_action_slide_channel"/>
        <field name="sequence">2</field>
        <field name="view_mode">graph</field>
        <field name="view_id" ref="rating_rating_view_graph_slide_channel"/>
    </record>
    <record id="rating_rating_action_slide_channel_view_pivot" model="ir.actions.act_window.view">
        <field name="act_window_id" ref="rating_rating_action_slide_channel"/>
        <field name="sequence">3</field>
        <field name="view_mode">pivot</field>
        <field name="view_id" ref="rating_rating_view_pivot_slide_channel"/>
    </record>
    <record id="rating_rating_action_slide_channel_view_tree" model="ir.actions.act_window.view">
        <field name="act_window_id" ref="rating_rating_action_slide_channel"/>
        <field name="sequence">4</field>
        <field name="view_mode">list</field>
        <field name="view_id" ref="rating_rating_view_tree_slide_channel"/>
    </record>
    <record id="rating_rating_action_slide_channel_view_form" model="ir.actions.act_window.view">
        <field name="act_window_id" ref="rating_rating_action_slide_channel"/>
        <field name="sequence">5</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="rating_rating_view_form_slides"/>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.slides</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//setting[@id='website_marketing_automation']" position="after">
                <setting id="slides_install_setting" groups="website_slides.group_website_slides_manager">
                    <span class="o_form_label">Slides</span>
                    <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                    <div class="text-muted">
                        Google Drive API Key
                    </div>
                    <div class="content-group">
                        <div class="row mt16">
                            <label for="website_slide_google_app_key" class="col-lg-3 o_light_label" string="API Key"/>
                            <field name="website_slide_google_app_key" class="oe_inline"/>
                        </div>
                        <div class="oe_link">
                            <a href="https://console.developers.google.com/flows/enableapi?apiid=drive,youtube"><span class="oi oi-arrow-right"/>
                                Create a Google Project and Get a Key
                            </a>
                        </div>
                    </div>
                </setting>
            </xpath>
            <xpath expr="//form" position="inside">
                <app data-string="eLearning" string="eLearning" name="website_slides" groups="website_slides.group_website_slides_manager">
                    <block title="eLearning" id="website_slides_selection_settings">
                        <setting id="website_slide_install_website_slides_survey" colspan="4" help="Evaluate the knowledge of your Attendees and certify their skills.">
                            <field name="module_website_slides_survey"/>
                        </setting>
                        <setting id="website_slides_install_sale_slides" colspan="4" string="Paid Courses" help="Sell access to your courses on your website and track revenues.">
                            <field name="module_website_sale_slides"/>
                        </setting>
                        <setting id="website_slides_install_mass_mailing_slides" colspan="4" help="Update all your Attendees at once through mass mailings.">
                            <field name="module_mass_mailing_slides"/>
                        </setting>
                        <setting id="website_slide_install_website_slides_forum" colspan="4" help="Create a community and let Attendees answer each others' questions.">
                            <field name="module_website_slides_forum"/>
                        </setting>
                    </block>
                </app>
            </xpath>
        </field>
    </record>

    <record id="website_slides_action_settings" model="ir.actions.act_window">
        <field name="name">Settings</field>
        <field name="res_model">res.config.settings</field>
        <field name="view_mode">form</field>
        <field name="target">inline</field>
        <field name="context">{'module': 'website_slides', 'bin_size': False}</field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="res_partner_view_form" model="ir.ui.view">
        <field name="name">res.partner.view.form.inherit.slides</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button class="oe_stat_button" type="object"
                    icon="fa-graduation-cap" name="action_view_courses"
                    groups="website_slides.group_website_slides_officer"
                    invisible="slide_channel_count == 0 or is_company">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value"><field name="slide_channel_count"/></span>
                        <span class="o_stat_text">Courses</span>
                    </div>
                </button>
                <button class="oe_stat_button" type="object"
                    icon="fa-graduation-cap" name="action_view_courses"
                    groups="website_slides.group_website_slides_officer"
                    invisible="slide_channel_company_count == 0 or not is_company">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value"><field name="slide_channel_company_count" /></span>
                        <span class="o_stat_text">Courses</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>

</data></odoo>

```

## File: views\slide_channel_add.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<record id="slide_channel_view_form_add" model="ir.ui.view">
    <field name="name">slide.channel.view.form.add</field>
    <field name="model">slide.channel</field>
    <field name="arch" type="xml">
        <form js_class="website_new_content_form">
            <div class="oe_title">
                <label for="name" string="Course Title"/>
                <h1><field name="name" placeholder="e.g. Computer Science for kids" class="w-100"/></h1>
            </div>
            <group>
                <field name="website_url" invisible="1"/>
                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" placeholder="Tags"/>
                <field name="channel_type" widget="image_radio" options="{'images': ['/website_slides/static/src/img/channel-training-layout.png', '/website_slides/static/src/img/channel-documentation-layout.png']}" string="Choose a layout"/>
                <field name="description" placeholder="Common tasks for a computer scientist is asking the right questions and answering questions..." class="mb-3"/>
                <field name="allow_comment" string="Allow Rating"/>
            </group>
        </form>
    </field>
</record>

<record id="slide_channel_action_add" model="ir.actions.act_window">
    <field name="name">New Course</field>
    <field name="res_model">slide.channel</field>
    <field name="view_mode">form</field>
    <field name="target">new</field>
    <field name="view_id" ref="slide_channel_view_form_add"/>
</record>

</odoo>

```

## File: views\slide_channel_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="slide_channel_partner_view_search" model="ir.ui.view">
            <field name="name">slide.channel.partner.search</field>
            <field name="model">slide.channel.partner</field>
            <field name="arch" type="xml">
                <search string="Course Member">
                    <field name="partner_id"/>
                    <field name="partner_email"/>
                    <field name="channel_id" string="Course"/>
                     <separator/>
                    <filter string="Archived" name="filter_archived" domain="[('active', '=', False)]"/>
                    <separator/>
                    <filter string="Invite Sent" name="filter_invited" domain="[('member_status', '=', 'invited')]"/>
                    <filter string="Joined" name="filter_joined" domain="[('member_status', '=', 'joined')]"/>
                    <filter string="Ongoing" name="filter_ongoing" domain="[('member_status', '=', 'ongoing')]"/>
                    <filter string="Completed" name="filter_completed" domain="[('member_status', '=', 'completed')]"/>
                    <group expand="0" string="Group By">
                        <filter string="Course" name="groupby_channel_id" context="{'group_by': 'channel_id'}"/>
                        <filter string="Status" name="groupby_member_status" context="{'group_by': 'member_status'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="slide_channel_partner_view_tree" model="ir.ui.view">
            <field name="name">slide.channel.partner.list</field>
            <field name="model">slide.channel.partner</field>
            <field name="arch" type="xml">
                <list string="Attendees" js_class="slide_channel_partner_enroll_tree" create="0" sample="1">
                    <field name="channel_id" string="Course Name"/>
                    <field name="partner_id"/>
                    <field name="partner_email" string="Email"/>
                    <field name="create_date" string="Added On"/>
                    <field name="write_date" string="Last Action On"/>
                    <field name="last_invitation_date" string="Last Invitation" optional="hide"/>
                    <field name="member_status" string="Status" widget="badge"
                        decoration-success="member_status == 'completed'"
                        decoration-info="member_status == 'ongoing'"
                        decoration-warning="member_status == 'joined'"
                        decoration-muted="member_status == 'invited'"/>
                    <field name="completion" string="Progress" widget="progressbar"/>
                    <field name="next_slide_id"/>
                    <field name="channel_user_id" widget="many2one_avatar_user" optional="hide"/>
                    <field name="channel_type" optional="hide"/>
                    <field name="channel_visibility" optional="hide"/>
                    <field name="channel_enroll" widget="badge"
                        decoration-success="channel_enroll == 'public'"
                        decoration-info="channel_enroll == 'invite'"
                        decoration-warning="channel_enroll == 'payment'"
                        optional="hide"/>
                    <field name="channel_website_id" groups="website.group_multi_website" optional="hide"/>
                    <field name="active" column_invisible="True"/>
                    <button name="action_archive" title="Archive" icon="fa-times" type="object"
                        invisible="not active"/>
                    <button name="action_unarchive" title="Unarchive" icon="fa-undo" type="object"
                        invisible="active"/>
                </list>
            </field>
        </record>

        <record id="slide_channel_partner_view_kanban" model="ir.ui.view">
            <field name="name">slide.channel.partner.view.kanban</field>
            <field name="model">slide.channel.partner</field>
            <field name="arch" type="xml">
                <kanban can_open="0" string="Attendees" class="o_slide_attendee_kanban">
                    <templates>
                        <t t-name="card">
                            <field name="channel_id" class="fw-bolder fs-5"/>
                            <field name="partner_id"/>
                            <footer class="mt-2">
                                <field name="completion" widget="progressbar"/>
                                <field name="channel_user_id" widget="many2one_avatar_user" class="ms-auto"/>
                            </footer>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="slide_channel_partner_view_graph" model="ir.ui.view">
            <field name="name">slide.channel.partner.view.graph</field>
            <field name="model">slide.channel.partner</field>
            <field name="arch" type="xml">
                <graph string="Attendees" stacked="0" sample="1"/>
            </field>
        </record>

        <record id="slide_channel_partner_view_pivot" model="ir.ui.view">
            <field name="name">slide.channel.partner.view.pivot</field>
            <field name="model">slide.channel.partner</field>
            <field name="arch" type="xml">
                <pivot string="Attendees" sample="1">
                    <field name="completion" invisible="1"/>
                </pivot>
            </field>
        </record>

        <record id="slide_channel_partner_action" model="ir.actions.act_window">
            <field name="name">Attendees</field>
            <field name="res_model">slide.channel.partner</field>
            <field name="view_mode">list,kanban</field>
            <field name="search_view_id" ref="website_slides.slide_channel_partner_view_search"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    <strong>No Attendees Yet!</strong>
                </p>
                <p>
                    From here you'll be able to monitor attendees and to track their progress.
                </p>
            </field>
        </record>

        <record id="slide_channel_partner_action_report" model="ir.actions.act_window">
            <field name="name">Attendees</field>
            <field name="res_model">slide.channel.partner</field>
            <field name="view_mode">graph,pivot,list,kanban</field>
            <field name="search_view_id" ref="website_slides.slide_channel_partner_view_search"/>
            <field name="context">{'search_default_groupby_member_status': 1}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    <strong>No Attendees Yet!</strong>
                </p>
                <p>
                    From here you'll be able to monitor attendees and to track their progress.
                </p>
            </field>
        </record>

    </data>
</odoo>

```

## File: views\slide_channel_tag_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <!-- SLIDE.CHANNEL.TAG -->
    <record id="slide_channel_tag_view_search" model="ir.ui.view">
        <field name="name">slide.channel.tag.view.search</field>
        <field name="model">slide.channel.tag</field>
        <field name="arch" type="xml">
            <search string="Course Tags">
                <field name="name"/>
                <field name="group_id"/>
            </search>
        </field>
    </record>

    <record id="slide_channel_tag_view_form" model="ir.ui.view">
        <field name="name">slide.channel.tag.view.form</field>
        <field name="model">slide.channel.tag</field>
        <field name="arch" type="xml">
            <form string="Course Tag">
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="group_id"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="slide_channel_tag_view_tree" model="ir.ui.view">
        <field name="name">slide.channel.tag.view.list</field>
        <field name="model">slide.channel.tag</field>
        <field name="arch" type="xml">
            <list string="Course Tags" editable="top">
                <field name="sequence" widget="handle"/>
                <field name="group_sequence" column_invisible="True"/>
                <field name="name"/>
                <field name="group_id"/>
            </list>
        </field>
    </record>

    <record id="slide_channel_tag_action" model="ir.actions.act_window">
        <field name="name">Course Tags</field>
        <field name="res_model">slide.channel.tag</field>
        <field name="view_mode">list,form</field>
    </record>

    <!-- SLIDE.CHANNEL.TAG.GROUP -->
    <record id="slide_channel_tag_group_view_search" model="ir.ui.view">
        <field name="name">slide.channel.tag.group.view.search</field>
        <field name="model">slide.channel.tag.group</field>
        <field name="arch" type="xml">
            <search string="Course Tag Groups">
                <field name="name"/>
                <filter string="Has Menu Entry" name="filter_is_published" domain="[('is_published', '=', True)]"/>
            </search>
        </field>
    </record>

    <record id="slide_channel_tag_group_view_form" model="ir.ui.view">
        <field name="name">slide.channel.tag.group.view.form</field>
        <field name="model">slide.channel.tag.group</field>
        <field name="arch" type="xml">
            <form string="Course Tag Group">
                <sheet>
                    <div class="oe_title">
                        <label for="name" string="Course Group Name"/>
                        <h1><field name="name" default_focus="1" placeholder="e.g. Your Level"/></h1>
                    </div>
                    <group>
                        <field name="is_published" string="Menu Entry"/>
                        <field name="tag_ids" nolabel="1" colspan="2">
                            <list editable="bottom">
                                <field name="sequence" widget="handle"/>
                                <field name="group_sequence" column_invisible="True"/>
                                <field name="name" string="Tag Name"/>
                                <field name="color" string="Color" widget="color_picker"/>
                                <control>
                                    <create string="Add a tag"/>
                                </control>
                            </list>
                        </field>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="slide_channel_tag_group_view_tree" model="ir.ui.view">
        <field name="name">slide.channel.tag.group.view.list</field>
        <field name="model">slide.channel.tag.group</field>
        <field name="arch" type="xml">
            <list string="Course Tag Groups">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="is_published" string="Menu Entry"/>
                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
            </list>
        </field>
    </record>

    <record id="slide_channel_tag_group_action" model="ir.actions.act_window">
        <field name="name">Course Groups</field>
        <field name="res_model">slide.channel.tag.group</field>
        <field name="view_mode">list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Course Group
            </p>
            <p>
                Use Course Groups to classify and organize your Courses.
            </p>
        </field>
    </record>

    </data>
</odoo>

```

## File: views\slide_channel_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- SLIDE.CHANNEL VIEWS -->
        <record model="ir.ui.view" id="view_slide_channel_form">
            <field name="name">slide.channel.view.form</field>
            <field name="model">slide.channel</field>
            <field name="arch" type="xml">
                <form string="Channels">
                    <header>
                        <button name="action_channel_enroll" string="Add Attendees" type="object" class="oe_highlight"/>
                        <button name="action_channel_invite" string="Invite" type="object" class="oe_highlight btn btn-secondary"/>
                    </header>
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button icon="fa-eye"
                                class="oe_stat_button"
                                groups="website_slides.group_website_slides_officer"
                                invisible="total_views == 0">
                                <field name="total_views" widget="statinfo" string="Visits"/>
                            </button>
                            <button name="action_view_slides"
                                type="object"
                                icon="fa-files-o"
                                class="oe_stat_button"
                                groups="website_slides.group_website_slides_officer">
                                <field name="total_slides" widget="statinfo" string="Published Contents"/>
                            </button>
                            <button name="action_redirect_to_completed_members"
                                type="object"
                                icon="fa-flag-checkered"
                                class="oe_stat_button"
                                groups="website_slides.group_website_slides_officer">
                                <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_value"><field name="members_completed_count" nolabel="1"/></span>
                                    <span name="members_completed_count_label" class="o_stat_text">Finished</span>
                                </div>
                            </button>
                            <button name="action_redirect_to_members"
                                type="object"
                                icon="fa-graduation-cap"
                                class="oe_stat_button"
                                groups="website_slides.group_website_slides_officer">
                                <div class="o_stat_info">
                                    <span class="o_stat_value">
                                        <field name="members_all_count" nolabel="1"/>
                                    </span>
                                    <span class="o_stat_text">Attendees</span>
                                </div>
                            </button>
                             <button name="action_view_ratings"
                                type="object"
                                icon="fa-star-half-o"
                                class="oe_stat_button"
                                groups="website_slides.group_website_slides_officer"
                                invisible="not allow_comment">
                                <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_value"><field name="rating_avg_stars" nolabel="1"/>/5</span>
                                    <span name="rating_count_label" class="o_stat_text"><field name="rating_count" nolabel="1"/> Reviews</span>
                                </div>
                            </button>
                            <field name="is_published" widget="website_redirect_button"/>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                        <field name="image_1920" widget="image" class="oe_avatar" options="{'preview_image': 'image_128'}"/>
                        <div class="oe_title">
                            <label for="name" string="Course Title"/>
                            <h1><field name="name" options="{'line_breaks': False}" widget="text" default_focus="1" placeholder='e.g. "Computer Science for kids"'/></h1>
                        </div>
                        <div>
                            <field name="active" invisible="1"/>
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" placeholder="Tags"/>
                        </div>
                        <notebook colspan="4">
                            <page name="content" string="Content">
                                <field name="slide_ids" string="Content" colspan="4" nolabel="1" widget="slide_category_one2many" mode="list,kanban" context="{'default_channel_id': id, 'form_view_ref' : 'website_slides.view_slide_slide_form_wo_channel_id'}">
                                     <list decoration-bf="is_category" editable="bottom">
                                        <field name="sequence" widget="handle"/>
                                        <field name="name"/>
                                        <field name="slide_category" invisible="slide_category == 'category'"/>
                                        <field name="completion_time" invisible="slide_category == 'category'" string="Duration" widget="float_time"/>
                                        <field name="total_views" invisible="slide_category == 'category'"/>
                                        <field name="is_preview" string="Preview"/>
                                        <field name="is_published" string="Published"/>
                                        <field name="is_category" column_invisible="True"/>
                                        <control>
                                            <create name="add_slide_section" string="Add Section" context="{'default_is_category': True}"/>
                                            <create name="add_slide_lesson" string="Add Content"/>
                                        </control>
                                    </list>
                                </field>
                            </page>
                            <page name="description" string="Description" >
                                <field name="description" widget="html" nolabel="1" placeholder="Common tasks for a computer scientist is asking the right questions and answering questions. In this course, you'll study those topics with activities about mathematics, science and logic."/>
                            </page>
                            <page name="options" string="Options">
                                <group>
                                    <group name="course" string="Course">
                                        <field name="user_id" domain="[('share', '=', False)]" widget="many2one_avatar_user"/>
                                        <field name="website_id" options="{'no_create': True}" groups="website.group_multi_website"/>
                                    </group>
                                    <group name="access_rights" string="Access Rights">
                                        <field name="website_published" invisible="1"/>
                                        <field name="prerequisite_channel_ids" widget="many2many_tags"/>
                                        <field name="prerequisite_of_channel_ids" widget="many2many_tags"
                                               readonly="1"
                                               invisible="not prerequisite_of_channel_ids"/>
                                        <field name="visibility" widget="radio" options="{'horizontal': true}"/>
                                        <field name="enroll" widget="radio" options="{'horizontal': true}" invisible="visibility == 'members'"/>
                                        <field name="upload_group_ids" widget="many2many_tags" groups="base.group_no_one"/>
                                        <field name="enroll_group_ids" widget="many2many_tags" groups="base.group_no_one"/>
                                        <field name="enroll_msg" invisible="enroll != 'invite' or visibility == 'members'"/>
                                    </group>
                                </group>
                                <group>
                                    <group name="communication" string="Communication">
                                        <field string="Allow Reviews" name="allow_comment"/>
                                        <field name="share_slide_template_id" domain="[('model','=','slide.slide')]" groups="base.group_no_one"/>
                                        <field name="share_channel_template_id" domain="[('model', '=', 'slide.channel')]" groups="base.group_no_one"/>
                                        <field name="publish_template_id" placeholder="No Notification"/>
                                        <field name="completed_template_id" placeholder="No Notification"/>
                                    </group>
                                    <group name="display" string="Display">
                                        <field string="Type" name="channel_type" widget="radio" options="{'horizontal': true}"/>
                                        <field name="promote_strategy" widget="selection"
                                            invisible="channel_type == 'training'"/>
                                        <field name="promoted_slide_id"
                                               invisible="channel_type == 'training' or promote_strategy != 'specific'"
                                               required="channel_type != 'training' and promote_strategy == 'specific'"
                                               string="Content"
                                               domain="[('channel_id', '=', id), ('is_category', '=', False)]"/>
                                    </group>
                                </group>
                            </page>
                            <page string="Karma" name="karma_rules">
                                <group>
                                    <group string="Rewards">
                                        <field name="karma_gen_channel_rank" string="Review Course"/>
                                        <field name="karma_gen_channel_finish" string="Finish Course"/>
                                    </group>
                                    <group string="Access Rights" invisible="not allow_comment">
                                        <field name="karma_review" invisible="not allow_comment"/>
                                        <field name="karma_slide_comment" invisible="not allow_comment"/>
                                        <field name="karma_slide_vote" invisible="not allow_comment"/>
                                    </group>
                                </group>
                            </page>
                        </notebook>
                    </sheet>
                    <chatter/>
                </form>
            </field>
        </record>


        <record id="slide_channel_view_tree" model="ir.ui.view">
            <field name="name">slide.channel.view.list</field>
            <field name="model">slide.channel</field>
            <field name="arch" type="xml">
                <list string="Courses" sample="1" multi_edit="1">
                    <field name="sequence" widget="handle"/>
                    <field name="name" readonly="1"/>
                    <field name="user_id" widget="many2one_avatar_user"/>
                    <field name="website_id" groups="website.group_multi_website"/>
                    <field name="channel_type" string="Course Type"/>
                    <field name="visibility"/>
                    <field name="enroll" widget="badge" decoration-success="enroll == 'public'" decoration-info="enroll == 'invite'" decoration-warning="enroll == 'payment'" optional="hide"/>
                    <field name="is_published" string="Published"/>
                    <field name="active" column_invisible="True"/>
                </list>
            </field>
        </record>

        <record id="slide_channel_view_tree_report" model="ir.ui.view">
            <field name="name">slide.channel.view.list.report</field>
            <field name="model">slide.channel</field>
            <field name="priority">20</field>
            <field name="arch" type="xml">
                <list string="Courses" create="false" default_order="total_views desc" sample="1">
                    <field name="name"/>
                    <field name="user_id" widget="many2one_avatar_user"/>
                    <field name="total_views" string="# Views"/>
                    <field name="rating_avg_stars" string="Average Review"/>
                    <field name="total_time" string="Total Duration" widget="float_time" sum="Total Duration"/>
                    <field name="members_count" string="# Attendees" sum="Total Attendees"/>
                    <field name="members_completed_count" string="# Completed" sum="Total Completed"/>
                </list>
            </field>
        </record>

        <record id="slide_channel_view_search" model="ir.ui.view">
            <field name="name">slide.channel.view.search</field>
            <field name="model">slide.channel</field>
            <field name="arch" type="xml">
                <search string="Courses">
                    <field name="name" string="Course"/>
                    <field name="user_id" string="Responsible"/>
                    <field name="tag_ids" string="Tags"/>
                    <field name="slide_ids" string="Contents"/>
                    <filter string="Published" name="filter_published" domain="[('is_published', '=', True)]"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                </search>
            </field>
        </record>

        <record id="slide_channel_view_graph" model="ir.ui.view">
            <field name="name">slide.channel.view.graph</field>
            <field name="model">slide.channel</field>
            <field name="arch" type="xml">
                <graph string="Courses" sample="1">
                    <field name="name"/>
                    <field name="total_views" type="measure"/>
                    <field name="karma_slide_comment" invisible="1"/>
                    <field name="karma_review" invisible="1"/>
                    <field name="color" invisible="1"/>
                    <field name="karma_gen_channel_finish" invisible="1"/>
                    <field name="karma_gen_channel_rank" invisible="1"/>
                    <field name="sequence" invisible="1"/>
                </graph>
            </field>
        </record>

        <record id="slide_channel_view_pivot" model="ir.ui.view">
            <field name="name">slide.channel.view.pivot</field>
            <field name="model">slide.channel</field>
            <field name="arch" type="xml">
                <pivot string="Pivot" default_order="create_date desc" sample="1">
                    <field type="row" name="name" />
                    <field type="measure" name="total_views"/>
                    <field name="karma_slide_comment" invisible="1"/>
                    <field name="karma_review" invisible="1"/>
                    <field name="color" invisible="1"/>
                    <field name="karma_gen_channel_finish" invisible="1"/>
                    <field name="karma_gen_channel_rank" invisible="1"/>
                    <field name="sequence" invisible="1"/>
                </pivot>
            </field>
        </record>

        <record id="slide_channel_view_kanban" model="ir.ui.view">
            <field name="name">slide.channel.view.kanban</field>
            <field name="model">slide.channel</field>
            <field name="arch" type="xml">
                <kanban highlight_color="color" string="eLearning Overview" class="o_slide_kanban o_slide_channel_kanban" edit="false" sample="1">
                    <field name="website_published"/>
                    <templates>
                        <t t-name="menu">
                            <div role="menuitem" aria-haspopup="true">
                                <field name="color" widget="kanban_color_picker"/>
                            </div>
                            <div class="o_kanban_slides_card_manage_pane">
                                <t t-if="widget.deletable">
                                    <a role="menuitem" type="delete" class="dropdown-item">Delete</a>
                                </t>
                                <a role="menuitem" type="open" class="dropdown-item">Edit</a>
                                <a role="menuitem" name="action_channel_enroll" class="dropdown-item" type="object">Add Attendees</a>
                                <a role="menuitem" name="action_channel_invite" class="dropdown-item" type="object">Invite</a>
                            </div>
                        </t>
                        <t t-name="card">
                            <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                            <widget name="web_ribbon" title="Published" bg_color="text-bg-success" invisible="not website_published or not active"/>
                            <field name="name" class="fw-bold fs-4 me-auto ms-1"/>
                            <field t-if="record.tag_ids" name="tag_ids" widget="many2many_tags"  options="{'color_field': 'color'}" class="mb-4 w-75 ms-1"/>
                            <div class="row g-0 mb16 mt-1 ms-2 me-2">
                                <div name="card_primary_left" class="col-6">
                                    <button class="btn btn-primary" name="open_website_url" type="object">View course</button>
                                </div>
                                <div class="col-6">
                                    <div class="d-flex">
                                        <label for="total_views" class="mb0 me-auto">Views</label>
                                        <field class="text-nowrap" name="total_views"/>
                                    </div>
                                    <div class="d-flex" name="info_total_slides">
                                        <label for="total_slides" class="mb0 me-auto">Contents</label>
                                        <field class="text-nowrap" name="total_slides"/>
                                    </div>
                                    <div class="d-flex" name="info_total_time">
                                        <label for="total_time" class="mb0 me-auto">Duration</label>
                                        <field class="text-nowrap" name="total_time" widget="float_time"/>
                                    </div>
                                    <div class="d-flex" name="info_avg_rating" t-if="record.rating_count.raw_value">
                                        <a name="action_view_ratings" type="object" class="me-auto"><field name="rating_count"/> Reviews</a>
                                        <span class="text-nowrap"><field name="rating_avg_stars"/> / 5</span>
                                    </div>
                                </div>
                            </div>
                            <div name="card_content" class="row mt-auto">
                                <a name="action_redirect_to_invited_members" type="object" class="d-flex flex-column align-items-center col-3 border-end">
                                    <field name="members_invited_count" class="fw-bold"/>
                                    <span class="text-muted text-truncate mw-100" title="Invited">Invited</span>
                                </a>
                                <a name="action_redirect_to_engaged_members" type="object" class="d-flex flex-column align-items-center col-3 border-end">
                                    <field name="members_engaged_count" class="fw-bold"/>
                                    <span class="text-muted text-truncate mw-100" title="Ongoing">Ongoing</span>
                                </a>
                                <a name="action_redirect_to_completed_members" type="object" class="d-flex flex-column align-items-center col-3 border-end">
                                    <field name="members_completed_count" class="fw-bold"/>
                                    <span name="done_members_count_label" class="text-muted text-truncate mw-100" title="Finished">Finished</span>
                                </a>
                                <a name="action_redirect_to_members" type="object" class="d-flex flex-column align-items-center col-3">
                                    <field name="members_all_count" class="fw-bold"/>
                                    <span class="text-muted text-truncate mw-100" title="Total">Total</span>
                                </a>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="slide_channel_action_overview" model="ir.actions.act_window">
            <field name="name">All Courses</field>
            <field name="path">e-learning</field>
            <field name="res_model">slide.channel</field>
            <field name="view_mode">kanban,list,form</field>
            <field name="view_id" ref="slide_channel_view_kanban"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    <strong>Create a course</strong>
                </p>
                <p>
                    Your eLearning platform starts here!<br/>
                    Upload content, set up rewards, manage attendees...
                </p>
            </field>
        </record>

        <record id="slide_channel_action_report" model="ir.actions.act_window">
            <field name="name">Courses</field>
            <field name="res_model">slide.channel</field>
            <field name="view_mode">list,graph,pivot,form</field>
            <field name="view_id" ref="slide_channel_view_tree_report"/>
            <field name="context">{"search_default_filter_published":1}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    <strong>Create a course</strong>
                </p>
                <p>
                    Your eLearning platform starts here!<br/>
                    Upload content, set up rewards, manage attendees...
                </p>
            </field>
        </record>

        <record id="slide_channel_action_report_view_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="1"/>
            <field name="view_mode">list</field>
            <field name="view_id" ref="slide_channel_view_tree_report"/>
            <field name="act_window_id" ref="slide_channel_action_report"/>
        </record>
        <record id="slide_channel_action_report_view_graph" model="ir.actions.act_window.view">
            <field name="sequence" eval="2"/>
            <field name="view_mode">graph</field>
            <field name="view_id" ref="slide_channel_view_graph"/>
            <field name="act_window_id" ref="slide_channel_action_report"/>
        </record>
        <record id="slide_channel_action_report_view_pivot" model="ir.actions.act_window.view">
            <field name="sequence" eval="3"/>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="slide_channel_view_pivot"/>
            <field name="act_window_id" ref="slide_channel_action_report"/>
        </record>
        <record id="slide_channel_action_report_view_form" model="ir.actions.act_window.view">
            <field name="sequence" eval="4"/>
            <field name="view_mode">form</field>
            <field name="view_id" ref="view_slide_channel_form"/>
            <field name="act_window_id" ref="slide_channel_action_report"/>
        </record>

    </data>
</odoo>

```

## File: views\slide_embed_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="slide_embed_view_tree" model="ir.ui.view">
            <field name="name">slide.embed.view.list</field>
            <field name="model">slide.embed</field>
            <field name="arch" type="xml">
                <list string="Embed Views" create="0" edit="0">
                    <field name="website_name" string="External Website"/>
                    <field name="count_views"/>
                    <field name="slide_id" string="Content" optional="hide"/>
                </list>
            </field>
        </record>

        <record id="slide_embed_view_search" model="ir.ui.view">
            <field name="name">slide.embed.view.search</field>
            <field name="model">slide.embed</field>
            <field name="arch" type="xml">
                <search>
                    <field name="slide_id" string="Content"/>
                </search>
            </field>
        </record>

        <record id="slide_embed_action" model="ir.actions.act_window">
            <field name="name">Embed Views</field>
            <field name="res_model">slide.embed</field>
            <field name="view_mode">list,search</field>
        </record>
    </data>
</odoo>

```

## File: views\slide_question_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="slide_question_view_form" model="ir.ui.view">
        <field name="name">slide.question.view.form</field>
        <field name="model">slide.question</field>
        <field name="arch" type="xml">
            <form string="Quiz">
                <div invisible="answers_validation_error == ''">
                    <div class="alert alert-info" role="alert" aria-label="Validation error">
                        <i class="fa fa-info-circle" aria-hidden="true"/>
                        <field name="answers_validation_error" class="ms-2" readonly="1"/>
                    </div>
                </div>
                <sheet>
                    <div class="oe_title">
                        <label for="question" string="Question Name"/>
                        <h1><field options="{'line_breaks': False}" widget="text" name="question" default_focus="1" placeholder="e.g. What powers a computer?"/></h1>
                    </div>
                    <field name="answer_ids">
                        <list editable="bottom" create="true" delete="true">
                            <field name="display_name" column_invisible="True"/>
                            <field name="text_value"/>
                            <field name="is_correct"/>
                            <field name="comment"/>
                        </list>
                    </field>
                </sheet>
            </form>
        </field>
    </record>

    <record id="slide_question_view_tree" model="ir.ui.view">
        <field name="name">slide.question.view.list</field>
        <field name="model">slide.question</field>
        <field name="arch" type="xml">
            <list string="Quizzes">
                <field name="sequence" widget="handle"/>
                <field name="question"/>
                <field name="slide_id"/>
            </list>
        </field>
    </record>

    <record id="slide_question_view_tree_report" model="ir.ui.view">
        <field name="name">slide.question.view.list.report</field>
        <field name="model">slide.question</field>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <list string="Quizzes" create="0">
                <field name="sequence" widget="handle"/>
                <field name="question"/>
                <field name="slide_id"/>
                <field name="attempts_count"/>
                <field name="attempts_avg"/>
                <field name="done_count"/>
            </list>
        </field>
    </record>

    <record id="slide_question_view_search" model="ir.ui.view">
        <field name="name">slide.question.view.search</field>
        <field name="model">slide.question</field>
        <field name="arch" type="xml">
            <search string="Quizzes">
                <field name="question"/>
                <field name="slide_id"/>
            </search>
        </field>
    </record>

    <record id="slide_question_action_report" model="ir.actions.act_window">
        <field name="name">Quizzes</field>
        <field name="res_model">slide.question</field>
        <field name="view_mode">list,graph,pivot,form</field>
        <field name="view_id" ref="slide_question_view_tree_report"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No Quiz data yet!
            </p>
            <p>
                Come back later to oversee how well your Attendees are doing.
            </p>
        </field>
    </record>
</odoo>

```

## File: views\slide_slide_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="slide_slide_partner_view_search" model="ir.ui.view">
        <field name="name">slide.slide.partner.view.search</field>
        <field name="model">slide.slide.partner</field>
        <field name="arch" type="xml">
            <search string="Attendees">
                <field name="partner_id"/>
                <field name="slide_id"/>
                <field name="channel_id"/>
                 <separator/>
                <filter string="Completed" name="filter_completed" domain="[('completed', '=', True)]"/>
                <group expand="0" string="Group By">
                    <filter string="Content" name="groupby_slide_id" context="{'group_by': 'slide_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="slide_slide_partner_view_tree" model="ir.ui.view">
        <field name="name">slide.slide.partner.view.list</field>
        <field name="model">slide.slide.partner</field>
        <field name="arch" type="xml">
            <list string="Attendees" create="0" delete="0">
                <field name="create_date" string="Accessed on"/>
                <field name="partner_id"/>
                <field name="slide_id" optional="hide"/>
                <field name="channel_id" optional="hide"/>
                <field name="completed" readonly="1"/>
                <field name="quiz_attempts_count" readonly="1" string="# Quizz Attempts"
                    sum="# Total Attempts"/>
                <field name="vote" readonly="1"
                    sum="# Likes"/>
            </list>
        </field>
    </record>

    <record id="slide_slide_partner_view_form" model="ir.ui.view">
        <field name="name">slide.slide.partner.view.form</field>
        <field name="model">slide.slide.partner</field>
        <field name="arch" type="xml">
            <form string="Attendee" create="0">
                <sheet>
                    <group col="2" name="main_content">
                        <group>
                            <field name="partner_id"/>
                            <field name="slide_id"/>
                            <field name="slide_category" invisible="1"/>
                            <field name="channel_id"/>
                        </group>
                        <group>
                            <field name="completed" readonly="1"/>
                            <field name="quiz_attempts_count" readonly="1"/>
                            <field name="vote" readonly="1"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="slide_slide_partner_action_from_slide" model="ir.actions.act_window">
        <field name="name">Attendees</field>
        <field name="res_model">slide.slide.partner</field>
        <field name="view_mode">list,form,kanban</field>
        <field name="search_view_id" ref="website_slides.slide_slide_partner_view_search"/>
        <field name="context">{'default_slide_id': active_id}</field>
        <field name="domain">[('slide_id', '=', active_id)]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                <strong>No Attendee Yet!</strong>
            </p>
            <p>
                From here you'll be able to monitor attendees and to track their progress.
            </p>
        </field>
    </record>

</data></odoo>

```

## File: views\slide_slide_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- SLIDE.TAG -->
        <record id="view_slide_tag_form" model="ir.ui.view">
            <field name="name">slide.tag.form</field>
            <field name="model">slide.tag</field>
            <field name="arch" type="xml">
                <form string="Tag">
                    <sheet>
                        <group>
                            <field name="name"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="view_slide_tag_tree" model="ir.ui.view">
            <field name="name">slide.tag.list</field>
            <field name="model">slide.tag</field>
            <field name="arch" type="xml">
                <list string="Tags" editable="bottom">
                    <field name="name" placeholder="e.g 'HowTo'"/>
                </list>
            </field>
        </record>

        <record id="action_slide_tag" model="ir.actions.act_window">
            <field name="name">Content Tags</field>
            <field name="res_model">slide.tag</field>
            <field name="view_mode">list,form</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a Content Tag
                </p>
                <p>
                    Use Content Tags to classify your Content.
                </p>
            </field>
        </record>

        <!-- SLIDE.SLIDE -->
        <record id="view_slide_slide_form" model="ir.ui.view">
            <field name="name">slide.slide.form</field>
            <field name="model">slide.slide</field>
            <field name="arch" type="xml">
                <form string="Lesson">
                    <sheet>
                        <field name="channel_type" invisible="1" readonly="1"/>
                        <field name="channel_allow_comment" invisible="1" readonly="1"/>
                        <div class="oe_button_box" name="button_box">
                            <button name="%(slide_slide_partner_action_from_slide)d"
                                    class="oe_stat_button" type="action" icon="fa-graduation-cap"
                                    invisible="slide_views == 0">
                                <field name="slide_views" widget="statinfo" string="Attendees"/>
                            </button>
                            <button disabled="1" icon="fa-thumbs-up" class="oe_stat_button"
                                invisible="channel_type == 'training' or likes == 0">
                                <field class="ms-1" name="likes" widget="statinfo" string="Likes"/>
                             </button>
                             <button disabled="1" icon="fa-thumbs-down" class="oe_stat_button"
                                invisible="channel_type == 'training' or dislikes == 0">
                                <field class="ms-1" name="dislikes" widget="statinfo" string="Dislikes"/>
                             </button>
                             <button disabled="1" icon="fa-comments" class="oe_stat_button"
                                 invisible="not channel_allow_comment or comments_count == 0">
                                <field class="ms-1" name="comments_count" widget="statinfo" string="Comments"/>
                            </button>
                            <button name="action_view_embeds" class="oe_stat_button" type="object" icon="fa-share-alt"
                                invisible="embed_count == 0">
                                <div class="o_stat_info">
                                    <span class="o_stat_value"><field name="embed_count"/></span>
                                    <span class="o_stat_text">Embed Views</span>
                                </div>
                            </button>
                            <field name="is_published" widget="website_redirect_button"
                                   invisible="is_category or not channel_id"/>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                        <field name="image_1920" widget="image" class="oe_avatar" options='{"preview_image": "image_256"}'
                            invisible="is_category"/>
                        <div class="oe_title pe-xl-0">
                            <div>
                                <label for="name" string="Content Title"/>
                            </div>
                            <h1>
                                <field name="name" default_focus="1" placeholder="e.g. Setting up your computer" class="me-0"/>
                                <field name="is_category" invisible="1"/>
                            </h1>
                            <field name="tag_ids" invisible="is_category" widget="many2many_tags" placeholder="Tags..."/>
                        </div>
                        <notebook invisible="is_category">
                            <page name="document" string="Document">
                                <group>
                                    <group name="lesson_details">
                                        <field name="active" invisible="1"/>
                                        <field name="channel_id"/>
                                        <field name="slide_category" string="Content Type"/>
                                        <field name="slide_type" invisible="1"/>
                                        <div class="text-muted" colspan="2" invisible="slide_category != 'quiz'">
                                            You can add questions to this quiz in the 'Quiz' tab.
                                        </div>
                                        <label for="source_type" string="" invisible="slide_category not in ['infographic', 'document']"/>
                                        <field name="source_type" widget="radio" nolabel="1" invisible="slide_category not in ['infographic', 'document']" />
                                        <field name="video_url" invisible="slide_category != 'video'" readonly="slide_category != 'video'" required="slide_category == 'video'"
                                            placeholder='e.g "www.youtube.com/watch?v=ebBez6bcSEc"'
                                            widget="url"/>
                                        <field name="document_google_url" invisible="source_type != 'external' or slide_category != 'document'" readonly="source_type != 'external' or slide_category != 'document'"
                                            placeholder='e.g "https://drive.google.com/file/..."'
                                            widget="url"/>
                                        <field name="image_google_url" invisible="source_type != 'external' or slide_category != 'infographic'" readonly="source_type != 'external' or slide_category != 'infographic'"
                                            placeholder='e.g "https://drive.google.com/file/..."'
                                            widget="url"/>
                                        <field name="document_binary_content" string="" options="{'accepted_file_extensions': '.pdf'}"
                                            invisible="source_type == 'external' or slide_category != 'document'"
                                            readonly="source_type == 'external' or slide_category != 'document'"/>
                                        <field name="image_binary_content" string="" options="{'accepted_file_extensions': 'image/*'}"
                                            invisible="source_type == 'external' or slide_category != 'infographic'"
                                            readonly="source_type == 'external' or slide_category != 'infographic'"/>
                                    </group>
                                    <group name="related_details">
                                        <field name="user_id" string="Responsible" domain="[('share', '=', False)]" widget="many2one_avatar"/>
                                        <label for="completion_time"/>
                                        <div>
                                            <field name="completion_time" widget="float_time" class="oe_inline"/>
                                            <span> hours</span>
                                        </div>
                                        <field name="slide_resource_downloadable" invisible="slide_category != 'document' or source_type != 'local_file'"/>
                                        <field name="date_published" string="Published Date" invisible="not date_published" groups="base.group_no_one"/>
                                        <field name="is_preview"/>
                                        <field name="public_views"/>
                                        <field name="total_views"/>
                                    </group>
                                </group>
                            </page>
                            <page name="description" string="Description">
                                <field name="description" options="{'embedded_components': false}" placeholder="e.g. In this video, we'll give you the keys on how Odoo can help you to grow your business. At the end, we'll propose you a quiz to test your knowledge."/>
                            </page>
                            <page string="Additional Resources" name="external_links" >
                                <group>
                                    <field name="slide_resource_ids" widget="one2many" nolabel="1">
                                        <list editable="top">
                                            <field name="sequence" widget="handle"/>
                                            <field name="resource_type"/>
                                            <field name="name" required="1"/>
                                            <field name="file_name" column_invisible="True"/>
                                            <field name="data" readonly="resource_type == 'url'" filename="file_name"/>
                                            <field name="link" string="Link"
                                                readonly="resource_type == 'file'"
                                                required="resource_type == 'url'"/>
                                        </list>
                                    </field>
                                </group>
                            </page>
                            <page name="quiz" string="Quiz">
                                <group name="quiz_details">
                                    <group name="quiz_rewards" string="Points Rewards">
                                        <group>
                                            <field string="First Try" name="quiz_first_attempt_reward"/>
                                            <field string="Second Try" name="quiz_second_attempt_reward"/>
                                            <field string="Third Try" name="quiz_third_attempt_reward"/>
                                            <field string="Fourth Try &amp; More" name="quiz_fourth_attempt_reward"/>
                                        </group>
                                    </group>
                                    <group name="questions" string="Questions">
                                        <field name="question_ids" nolabel="1">
                                            <list>
                                                <field name="sequence" widget="handle"/>
                                                <field name="question" string="Question"/>
                                                <field name="answer_ids" string="Answers" widget="many2many_tags"/>
                                            </list>
                                        </field>
                                    </group>
                                </group>
                            </page>
                        </notebook>
                    </sheet>
                    <chatter/>
                </form>
            </field>
        </record>

        <record id="view_slide_slide_form_wo_channel_id" model="ir.ui.view">
            <field name="name">slide.slide.form.wo.channel_id</field>
            <field name="model">slide.slide</field>
            <field name="inherit_id" ref="view_slide_slide_form"/>
            <field name="priority" eval="50"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <field name="channel_id" position="attributes">
                    <attribute name="invisible">1</attribute>
                    <attribute name="required">0</attribute>
                </field>
            </field>
        </record>

        <record id="slide_slide_view_kanban" model="ir.ui.view">
            <field name="name">slide.slide.view.kanban</field>
            <field name="model">slide.slide</field>
            <field name="arch" type="xml">
                <kanban edit="false" group_create="0"
                    records_draggable="0"
                    class="o_slide_kanban"
                    sample="1">
                    <templates>
                        <t t-name="card" class="flex-row">
                            <aside class="o_kanban_aside_full">
                                <t t-if="record.image_128.raw_value">
                                    <div class="o_kanban_image_fill position-relative w-100">
                                        <field name="image_128" class="h-100" widget="image" options="{'img_class': 'object-fit-cover'}"/>
                                        <field name="channel_id" class="o_website_slides_inner_image position-absolute bottom-0 end-0 bg-light" widget="image" options="{'preview_image': 'image_128', 'img_class': 'object-fit-contain'}"/>
                                    </div>
                                </t>
                                <t t-else="">
                                    <img src="/website_slides/static/src/img/channel-training-default.jpg" class="w-100" options="{'img_class': 'object-fit-cover'}" alt="Default training image"/>
                                </t>
                            </aside>
                            <main>
                                <field name="name" class="fw-bolder fs-5"/>
                                <field name="channel_id" class="text-mutex"/>
                                <field name="tag_ids" widget="many2many_tags" class="mb-2"/>
                                <footer class="mt-auto d-flex justify-content-between align-items-end pt-0">
                                    <span>
                                        <t t-if="record.slide_category.raw_value == 'infographic'">
                                            <i class="fa fa-file-image-o me-2" aria-label="Infographic" role="img" title="Infographic"/>
                                        </t>
                                        <t t-elif="record.slide_category.raw_value == 'article'">
                                            <i class="fa fa-file-code-o me-2" aria-label="article" role="img" title="Article"/>
                                        </t>
                                        <t t-elif="record.slide_category.raw_value == 'video'">
                                            <i class="fa fa-file-video-o me-2" aria-label="Video" role="img" title="Video"/>
                                        </t>
                                        <t t-elif="record.slide_category.raw_value == 'quiz'">
                                            <i class="fa fa-flag me-2" aria-label="Quiz" role="img" title="Quiz"/>
                                        </t>
                                        <t t-else=""><i class="fa fa-file-pdf-o me-2" aria-label="Document" role="img" title="Document"/></t>
                                        <field name="slide_category"/>
                                    </span>
                                    <span class="d-flex align-items-center">
                                        <i class="fa fa-clock-o me-2" aria-label="Duration" role="img" title="Duration"/><field name="completion_time" widget="float_time"/>
                                    </span>
                                    <span>
                                        <i class="fa fa-question me-2" aria-label="Number of Questions" role="img" title="Number of Questions"/><field name="questions_count"/>
                                    </span>
                                    <span>
                                        <i class="fa fa-eye me-2" aria-label="Views" role="img" title="Views"/><field name="total_views"/>
                                    </span>
                                    <field name="user_id" widget="many2one_avatar_user"/>
                                </footer>
                            </main>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="view_slide_slide_tree" model="ir.ui.view">
            <field name="name">slide.slide.list</field>
            <field name="model">slide.slide</field>
            <field name="arch" type="xml">
                <list string="Contents" sample="1" multi_edit="1">
                    <field name="name" readonly="1"/>
                    <field name="channel_id" readonly="1"/>
                    <field name="category_id" readonly="1" optional="hide"/>
                    <field name="user_id" widget="many2one_avatar_user"/>
                    <field name="is_published"/>
                    <field name="date_published" readonly="1"/>
                    <field name="completion_time" sum="Total" readonly="1" widget="float_time"/>
                </list>
            </field>
        </record>

        <record id="slide_slide_view_tree_report" model="ir.ui.view">
            <field name="name">slide.slide.view.list.report</field>
            <field name="model">slide.slide</field>
            <field name="priority" eval="20"/>
            <field name="arch" type="xml">
                <list string="Contents" sample="1">
                    <field name="name"/>
                    <field name="user_id" widget="many2one_avatar_user"/>
                    <field name="channel_id"/>
                    <field name="category_id" optional="hide"/>
                    <field name="date_published"/>
                    <field name="total_views" string="# Views" sum="Total Views"/>
                    <field name="questions_count" string="# Questions" sum="Total Questions"/>
                    <field name="completion_time" sum="Total Duration" widget="float_time"/>
                </list>
            </field>
        </record>

        <record id="view_slide_slide_search" model="ir.ui.view">
            <field name="name">slide.slide.filter</field>
            <field name="model">slide.slide</field>
            <field name="arch" type="xml">
                <search string="Search Contents">
                    <field name="name"/>
                    <field name="channel_id"/>
                    <field name="user_id"/>
                    <field name="tag_ids"/>
                    <filter name="filter_user_id_uid" string="My Content" domain="[('user_id', '=', uid)]"/>
                    <separator/>
                    <filter name="published" string="Published" domain="[('is_published', '=', True)]"/>
                    <filter name="not_published" string="Waiting for validation" domain="[('is_published', '=', False)]"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                    <group expand="0" string="Group By">
                        <filter string="Course" name="groupby_channel" domain="[]" context="{'group_by': 'channel_id'}"/>
                        <filter string="Category" name="groupby_category" domain="[]" context="{'group_by': 'category_id'}"/>
                        <filter string="Type" name="groupby_type" domain="[]" context="{'group_by': 'slide_category'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="slide_slide_view_graph" model="ir.ui.view">
            <field name="name">slide.slide.view.graph</field>
            <field name="model">slide.slide</field>
            <field name="arch" type="xml">
                <graph string="Graph of Contents" stacked="0" sample="1">
                    <field name="channel_id"/>
                    <field name="slide_category"/>
                    <field name="total_views" type="measure"/>
                    <field name="quiz_first_attempt_reward" invisible="1"/>
                    <field name="quiz_second_attempt_reward" invisible="1"/>
                    <field name="quiz_third_attempt_reward" invisible="1"/>
                    <field name="quiz_fourth_attempt_reward" invisible="1"/>
                    <field name="sequence" invisible="1"/>
                </graph>
            </field>
        </record>

        <record id="slide_slide_view_pivot" model="ir.ui.view">
            <field name="name">slide.slide.view.pivot</field>
            <field name="model">slide.slide</field>
            <field name="arch" type="xml">
                <pivot sample="1">
                    <field name="channel_id" type="row"/>
                    <field name="total_views" type="measure"/>
                </pivot>
            </field>
        </record>

        <record id="slide_slide_action" model="ir.actions.act_window">
            <field name="name">Contents</field>
            <field name="res_model">slide.slide</field>
            <field name="view_mode">kanban,list,form</field>
            <field name="context">{'search_default_own_publications':True}</field>
            <field name="domain">[('is_category', '=', False)]</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Add Content
                </p>
                <p>
                    Content are the lessons that compose a course
                    <br/>and can be of different types (presentations, documents, videos, ...).
                </p>
            </field>
        </record>

        <record id="slide_slide_action_report" model="ir.actions.act_window">
            <field name="name">Contents</field>
            <field name="res_model">slide.slide</field>
            <field name="view_mode">graph,list,form,pivot</field>
            <field name="context">{"search_default_published": 1}</field>
            <field name="domain">[('is_category', '=', False)]</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No data yet!
                </p><p>
                    Create new content for your eLearning
                </p>
            </field>
        </record>

        <record id="slide_slide_action_report_view_graph" model="ir.actions.act_window.view">
            <field name="sequence" eval="0"/>
            <field name="view_mode">graph</field>
            <field name="view_id" ref="slide_slide_view_graph"/>
            <field name="act_window_id" ref="slide_slide_action_report"/>
        </record>
        <record id="slide_slide_action_report_view_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="1"/>
            <field name="view_mode">list</field>
            <field name="view_id" ref="slide_slide_view_tree_report"/>
            <field name="act_window_id" ref="slide_slide_action_report"/>
        </record>
        <record id="slide_slide_action_report_view_form" model="ir.actions.act_window.view">
            <field name="sequence" eval="2"/>
            <field name="view_mode">form</field>
            <field name="view_id" ref="view_slide_slide_form"/>
            <field name="act_window_id" ref="slide_slide_action_report"/>
        </record>
        <record id="slide_slide_action_report_view_pivot" model="ir.actions.act_window.view">
            <field name="sequence" eval="3"/>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="slide_slide_view_pivot"/>
            <field name="act_window_id" ref="slide_slide_action_report"/>
        </record>
    </data>
</odoo>

```

## File: views\slide_snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- Snippets and options -->
<template id="slide_searchbar_input_snippet_options" inherit_id="website.searchbar_input_snippet_options" name="slide search bar snippet options">
    <xpath expr="//div[@data-js='SearchBar']/we-select[@data-name='scope_opt']" position="inside">
        <we-button data-set-search-type="slides" data-select-data-attribute="slides" data-name="search_slides_opt" data-form-action="/slides/all">Courses</we-button>
    </xpath>
    <xpath expr="//div[@data-js='SearchBar']/we-select[@data-name='order_opt']" position="inside">
        <we-button data-set-order-by="slide_last_update asc" data-select-data-attribute="slide_last_update asc" data-dependencies="search_slides_opt" data-name="order_slide_last_update_asc_opt">Date (old to new)</we-button>
        <we-button data-set-order-by="slide_last_update desc" data-select-data-attribute="slide_last_update desc" data-dependencies="search_slides_opt" data-name="order_slide_last_update_desc_opt">Date (new to old)</we-button>
    </xpath>
    <xpath expr="//div[@data-js='SearchBar']/div[@data-dependencies='limit_opt']" position="inside">
        <we-checkbox string="Description" data-dependencies="search_slides_opt" data-select-data-attribute="true" data-attribute-name="displayDescription"
            data-apply-to=".search-query"/>
        <we-checkbox string="Publication Date" data-dependencies="search_slides_opt" data-select-data-attribute="true" data-attribute-name="displayDetail"
            data-apply-to=".search-query"/>
    </xpath>
</template>

<template id="snippet_options" inherit_id="website.snippet_options" name="Slides Snippet Options">
    <xpath expr="." position="inside">
        <div data-selector="main:has(.o_wslides_home_main)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Courses Page">
            <we-checkbox string="New Content Ribbon"
                         data-customize-website-views="website_slides.course_card_information_arrow"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
        <div data-selector="main:has(.o_wslides_home_aside_loggedin)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Courses Page">
            <we-checkbox string="Achievements"
                         data-customize-website-views="website_slides.toggle_latest_achievements"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Leaderboard"
                         data-customize-website-views="website_slides.toggle_leaderboard"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
    </xpath>
</template>

</odoo>

```

## File: views\website_pages_views.xml

```xml
<?xml version="1.0"?>
<odoo>

<record id="slide_channel_pages_tree_view" model="ir.ui.view">
    <field name="name">Course Pages List</field>
    <field name="model">slide.channel</field>
    <field name="priority">99</field>
    <field name="mode">primary</field>
    <field name="inherit_id" ref="slide_channel_view_tree"/>
    <field name="arch" type="xml">
        <xpath expr="//list" position="attributes">
            <attribute name="js_class">website_pages_list</attribute>
            <attribute name="type">object</attribute>
            <attribute name="action">open_website_url</attribute>
        </xpath>

        <field name="name" position="after">
            <field name="website_url"/>
        </field>
        <xpath expr="//list">
            <field name="is_seo_optimized"/>
            <field name="is_published"/>

            <field name="website_id" position="move"/>
        </xpath>
    </field>
</record>

<record id="slide_channel_pages_kanban_view" model="ir.ui.view">
    <field name="name">Course Pages Kanban</field>
    <field name="model">slide.channel</field>
    <field name="priority">99</field>
    <field name="mode">primary</field>
    <field name="inherit_id" ref="slide_channel_view_kanban"/>
    <field name="arch" type="xml">
        <xpath expr="//kanban" position="attributes">
            <attribute name="js_class">website_pages_kanban</attribute>
            <attribute name="type">object</attribute>
            <attribute name="action">open_website_url</attribute>
        </xpath>
        <xpath expr="//kanban" position="inside">
            <field name="website_url" invisible="1"/>
        </xpath>
        <xpath expr="//div[@name='card_primary_left']" position="replace">
            <div class="col-6 text-primary" t-if="record.website_id.value" groups="website.group_multi_website">
                <i class="fa fa-globe me-1" title="Website"/>
                <field name="website_id"/>
            </div>
        </xpath>
        <xpath expr="//div[@name='card_content']" position="after">
            <div class="d-flex border-top mt-2 pt-2">
                <field name="is_published" widget="boolean_toggle"/>
                <t t-if="record.is_published.raw_value">Published</t>
                <t t-else="">Not Published</t>
            </div>
        </xpath>
    </field>
</record>

<record id="action_slide_channel_pages_list" model="ir.actions.act_window">
    <field name="name">Course Pages</field>
    <field name="res_model">slide.channel</field>
    <field name="view_mode">list,kanban,form</field>
    <field name="view_ids" eval="[(5, 0, 0),
        (0, 0, {'view_mode': 'list', 'view_id': ref('slide_channel_pages_tree_view')}),
        (0, 0, {'view_mode': 'kanban', 'view_id': ref('slide_channel_pages_kanban_view')}),
    ]"/>
    <field name="context">{'create_action': 'website_slides.slide_channel_action_add'}</field>
</record>

<menuitem id="menu_slide_channel_pages"
    parent="website.menu_content"
    sequence="50"
    name="Courses"
    action="action_slide_channel_pages_list"/>

</odoo>

```

## File: views\website_slides_menu_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem name="eLearning"
        id="website_slides_menu_root"
        web_icon="website_slides,static/description/icon.png"
        groups="website_slides.group_website_slides_officer"
        action="slide_channel_action_overview"
        sequence="100"/>

    <!-- Main top menu elements -->
    <menuitem name="Courses"
        id="website_slides_menu_courses"
        parent="website_slides_menu_root"
        sequence="1"/>
    <menuitem name="Reporting"
        id="website_slides_menu_report"
        parent="website_slides_menu_root"
        groups="website_slides.group_website_slides_manager"
        sequence="9"/>
    <menuitem name="Configuration"
        id="website_slides_menu_configuration"
        parent="website_slides_menu_root"
        sequence="99"/>
    
    <!-- Courses sub-menu -->
    <menuitem name="Courses"
        id="website_slides_menu_courses_courses"
        parent="website_slides_menu_courses"
        sequence="1"
        action="slide_channel_action_overview"/>
    <menuitem name="Contents"
        id="website_slides_menu_courses_content"
        parent="website_slides_menu_courses"
        sequence="2"
        action="slide_slide_action"/>

    <!-- Reporting sub-menu -->
    <menuitem name="Courses"
        id="website_slides_menu_report_courses"
        parent="website_slides_menu_report"
        sequence="1"
        action="slide_channel_action_report"/>
    <menuitem name="Contents"
        id="website_slides_menu_report_contents"
        parent="website_slides_menu_report"
        sequence="2"
        action="slide_slide_action_report"/>
    <menuitem name="Attendees"
        id="website_slides_menu_report_attendees"
        parent="website_slides_menu_report"
        sequence="5"
        action="slide_channel_partner_action_report"/>
    <menuitem name="Reviews"
        id="website_slides_menu_report_reviews"
        parent="website_slides_menu_report"
        sequence="10"
        action="rating_rating_action_slide_channel"/>
    <menuitem name="Quizzes"
        id="website_slides_menu_report_quizzes"
        parent="website_slides_menu_report"
        sequence="15"
        action="slide_question_action_report"/>

    <!-- Settings sub-menu -->
    <menuitem name="Settings"
        id="website_slides_menu_config_settings"
        parent="website_slides_menu_configuration"
        sequence="1"
        action="website_slides_action_settings"
        groups="base.group_system"/>
    <menuitem name="Course Groups"
        id="website_slides_menu_config_course_groups"
        parent="website_slides_menu_configuration"
        sequence="2"
        action="slide_channel_tag_group_action"/>
    <menuitem name="Content Tags"
        id="website_slides_menu_config_content_tags"
        parent="website_slides_menu_configuration"
        sequence="3"
        action="action_slide_tag"/>

</odoo>

```

## File: views\website_slides_templates_course.xml

```xml
<?xml version="1.0" ?>
<odoo><data>

<!-- Channels sub-template: header -->
<template id="course_nav" name="Course Navigation Header">
    <div class="o_wslides_course_nav">
        <div class="container">
            <div class="row align-items-center justify-content-between">
                <!-- Desktop Mode -->
                <nav aria-label="breadcrumb" class="col-md-8 d-none d-md-flex">
                    <ol class="breadcrumb flex-nowrap bg-transparent mb-0 ps-0 py-0 overflow-hidden">
                        <li class="breadcrumb-item flex-shrink-0">
                            <a href="/slides" title="Courses">Courses</a>
                        </li>
                        <t t-set="breadcrumb_class" t-value="'breadcrumb-item text-truncate %s' % ('fw-bold' if not slide else '')" />
                        <li t-att-class="'breadcrumb-item text-truncate %s' % ('fw-bold' if not search_category and not search_tag and not search_slide_category and not slide else '')">
                            <a t-if="invite_preview" t-attf-href="/slides/#{channel.id}?#{keep_query('invite_hash', 'invite_partner_id')}" class="text-truncate d-block"><span t-esc="channel.name" t-att-title="channel.name"/></a>
                            <a t-else="" t-att-href="'/slides/%s' % slug(channel)" class="text-truncate d-block"><span t-esc="channel.name" t-att-title="channel.name"/></a>
                        </li>
                        <li t-att-class="breadcrumb_class" t-if="search_category">
                            <a t-if="invite_preview" t-attf-href="/slides/#{channel.id}/category/#{search_category.id}?#{keep_query('invite_hash', 'invite_partner_id')}" aria-current='page'><span t-esc="search_category.name" t-att-title="search_category.name"/></a>
                            <a t-else="" t-attf-href="/slides/#{slug(channel)}/category/#{slug(search_category)}" aria-current='page'><span t-esc="search_category.name" t-att-title="search_category.name"/></a>
                        </li>
                        <li t-att-class="breadcrumb_class" t-if="search_tag">
                            <a t-att-href="'/slides/%s/tag/%s' % (slug(channel), slug(search_tag))" aria-current='page'><span t-esc="search_tag.name" t-att-title="search_tag.name"/></a>
                        </li>
                        <li t-att-class="breadcrumb_class" t-if="search_uncategorized">
                            <a t-if="invite_preview" t-attf-href="/slides/#{channel.id}?uncategorized=1&amp;#{keep_query('invite_hash', 'invite_partner_id')}" aria-current='page' title="Uncategorized">Uncategorized</a>
                            <a t-else="" t-attf-href="/slides/#{slug(channel)}?uncategorized=1" aria-current='page' title="Uncategorized">Uncategorized</a>
                        </li>
                        <li t-att-class="breadcrumb_class" t-if="search_slide_category">
                            <a t-att-href="'/slides/%s?slide_category=%s' % (slug(channel), search_slide_category)" aria-current='page'>
                                <span t-esc="slide_categories.get('search_slide_category', '-')" t-att-title="slide_categories.get('search_slide_category', '-')"/>
                            </a>
                        </li>
                        <li t-if="slide" class="breadcrumb-item text-truncate fw-bold">
                            <a t-att-href="'/slides/slide/%s' % slug(slide)"><span t-esc="slide.name" t-att-title="slide.name"/></a>
                        </li>
                    </ol>
                </nav>

                <div class="col-md-4 d-none d-md-flex flex-row align-items-center justify-content-end">
                    <!-- search -->
                    <t t-call="website.website_search_box_input">
                        <t t-set="_classes" t-valuef="o_wslides_course_nav_search ms-1 position-relative"/>
                        <t t-set="_input_classes" t-valuef="border-0 rounded-0 bg-transparent"/>
                        <t t-set="_submit_classes" t-valuef="btn-link rounded-0 pe-1"/>
                        <t t-set="search_type" t-valuef="slides"/>
                        <t t-set="action" t-valuef="/slides/all"/>
                        <t t-set="display_description" t-valuef="true"/>
                        <t t-set="display_detail" t-valuef="false"/>
                        <t t-set="placeholder">Search courses</t>
                        <t t-set="search" t-value="search_term"/>
                    </t>
                </div>

                <!-- Mobile Mode -->
                <div class="col d-md-none py-1">
                    <div class="btn-group w-100 position-relative" role="group" aria-label="Mobile sub-nav">
                        <div class="btn-group w-100">
                            <a class="btn bg-black-25 text-white dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="false">Nav</a>

                            <div class="dropdown-menu">
                                <a class="dropdown-item" href="/slides">Home</a>
                                <t t-set="dropdown_class" t-value="'dropdown-item %s' % ('active' if not slide else '')"/>
                                <a t-attf-class="dropdown-item #{'active' if not search_category and not search_tag and not search_slide_category else ''}"
                                    t-attf-href="/slides/#{channel.id if invite_preview else slug(channel)}?#{keep_query('invite_hash', 'invite_partner_id') if invite_preview else ''}">
                                    &#9492;<span class="ms-1" t-esc="channel.name"/>
                                </a>
                                <a t-att-class="dropdown_class" aria-current="page" t-if="search_category"
                                    t-attf-href="/slides/#{channel.id if invite_preview else slug(channel)}/category/#{search_category.id if invite_preview else slug(search_category)}?#{keep_query('invite_hash', 'invite_partner_id') if invite_preview else ''}">
                                    &#9492;<span class="ms-1" t-esc="search_category.name"/>
                                </a>
                                <a t-att-class="dropdown_class" aria-current="page" t-if="search_tag" t-att-href="'/slides/%s/tag/%s' % (slug(channel), slug(search_tag))">
                                    &#9492;<span class="ms-1" t-esc="search_tag.name"/>
                                </a>
                                <a t-att-class="dropdown_class" aria-current="page" t-if="search_uncategorized"
                                    t-attf-href="/slides/#{channel.id if invite_preview else slug(channel)}?uncategorized=1&amp;#{keep_query('invite_hash', 'invite_partner_id') if invite_preview else ''}">
                                    &#9492;<span class="ms-1">Uncategorized</span>
                                </a>
                                <a t-att-class="dropdown_class" aria-current="page" t-if="search_slide_category" t-att-href="'/slides/%s?slide_category=%s' % (slug(channel), search_slide_category)">
                                    &#9492;<span class="ms-1" t-esc="slide_categories.get('search_slide_category', '-')"/>
                                </a>
                                 <a t-if="slide" class="dropdown-item active" t-att-href="'/slides/slide/%s' % (slug(slide))">
                                    &#9492;<span class="ms-1" t-esc="slide.name"/>
                                </a>
                            </div>
                        </div>

                        <div class="btn-group ms-1 position-static">
                            <a class="btn bg-black-25 text-white dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="false" aria-label="Search"><i class="fa fa-search"></i></a>
                            <div class="dropdown-menu dropdown-menu-end w-100" style="right: 10px;">
                                <t t-call="website.website_search_box_input">
                                    <t t-set="_classes" t-valuef="px-3"/>
                                    <t t-set="search_type" t-valuef="slides"/>
                                    <t t-set="action" t-value="'/slides/%s' % slug(channel)"/>
                                    <t t-set="display_description" t-valuef="true"/>
                                    <t t-set="display_detail" t-valuef="false"/>
                                    <t t-set="placeholder">Search courses</t>
                                    <t t-set="search" t-value="search_term"/>
                                </t>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>


<!-- Channel main template -->
<template id='course_main' name="Course Main" track="1">
    <t t-set="head">
        <script type="module" src="/web/static/lib/pdfjs/build/pdf.js"/>
        <script type="module" src="/web/static/lib/pdfjs/build/pdf.worker.js"/>
        <script type="text/javascript" src="/website_slides/static/lib/pdfslidesviewer/PDFSlidesViewer.js"></script>
    </t>
    <t t-set="body_classname" t-value="'o_wslides_body'"/>
    <t t-call="website.layout">
        <div id="wrap" t-attf-class="wrap mt-0">
            <div class="oe_structure oe_empty" t-call="website.record_cover">
                <t t-set="_record" t-value="channel"/>
                <t t-set="use_filters" t-value="True"/>
                <t t-set="use_size" t-value="True"/>
                <t t-set="use_text_align" t-value="True"/>
                <div t-attf-class="o_wslides_course_header position-relative pb-md-0 pt-2 pt-md-5 #{'pb-3' if channel.channel_type == 'training' else 'o_wslides_course_doc_header pb-5'}">
                    <t t-call="website_slides.course_nav"/>
                    <div class="container mt-5 mt-md-3 mt-xl-4">
                        <div class="row align-items-end align-items-md-stretch">
                            <!-- ==== Header Left ==== -->
                            <div class="col-12 col-md-4 col-lg-3 position-relative">
                                <div class="d-flex align-items-end justify-content-around h-100">
                                    <div t-field="channel.image_1920" t-options="{'widget': 'image', 'class' : 'o_wslides_course_pict d-inline-block mb-2 mt-3 my-md-0', 'preview_image': 'image_1024'}" class="h-100"/>
                                </div>
                            </div>

                            <!-- ==== Header Right ==== -->
                            <div class="col-12 col-md-8 col-lg-9 d-flex flex-column">
                                <div class="d-flex flex-column">
                                    <h1 t-field="channel.name"/>
                                    <div class="mb-0 mb-xl-3" t-field="channel.description"/>
                                </div>
                                <div class="d-flex flex-column justify-content-center h5 flex-grow-1 mb-md-5 o_not_editable" t-if="channel.allow_comment">
                                    <t t-call="portal_rating.rating_stars_static_popup_composer">
                                        <t t-set="rating_avg" t-value="rating_avg"/>
                                        <t t-set="rating_count" t-value="rating_count"/>
                                        <t t-set="object" t-value="channel"/>
                                        <t t-set="token" t-value="channel.access_token"/>
                                        <t t-set="hash" t-value="message_post_hash"/>
                                        <t t-set="pid" t-value="message_post_pid"/>
                                        <t t-set="default_message" t-value="last_message"/>
                                        <t t-set="default_message_id" t-value="last_message_id"/>
                                        <t t-set="default_rating_value" t-value="last_rating_value"/>
                                        <t t-set="default_attachment_ids" t-value="last_message_attachment_ids"/>
                                        <t t-set="force_submit_url" t-value="'/slides/mail/update_comment' if last_message_id else False"/>
                                        <t t-set="rate_with_void_content" t-value="True"/>
                                        <t t-set="disable_composer" t-value="not channel.can_review"/>
                                        <t t-set="_link_btn_classes" t-value="'btn-sm btn-link mx-3 o_wslides_header_text'"/>
                                        <t t-set="_text_classes" t-value="'css_editable_mode_hidden'"/>
                                    </t>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <div class="o_wslides_course_main">
                <t t-set="channel_frontend_tags" t-value="channel.tag_ids.filtered(lambda tag: tag.color)"/>
                <div class="container mb-5">
                    <div class="row">
                        <!-- Sidebar -->
                        <div class="col-12 col-md-4 col-lg-3 mt-3 mt-md-0">
                            <t t-call="website_slides.course_sidebar"/>
                        </div>
                        <div class="col-12 col-md-8 col-lg-9 position-relative">
                            <ul class="nav nav-tabs o_wslides_nav_tabs flex-nowrap" role="tablist" id="profile_extra_info_tablist">
                                <li class="nav-item" role="presentation">
                                    <a t-att-class="'nav-link %s' % ('active' if active_tab == 'home' else '')"
                                        id="home-tab" data-bs-toggle="pill" href="#home" role="tab" aria-controls="home"
                                        t-att-aria-selected="'true' if active_tab == 'home' else 'false'">
                                        <i class="fa fa-home"/> Course
                                    </a>
                                </li>
                                <li t-if="channel.allow_comment" class="nav-item o_wslides_course_header_nav_review" role="presentation">
                                    <a t-att-class="'nav-link %s' % ('active' if active_tab == 'review' else '')"
                                        id="review-tab" data-bs-toggle="pill" href="#review" role="tab" aria-controls="review"
                                        t-att-aria-selected="'true' if active_tab == 'review' else 'false'">
                                        Reviews<t t-if="rating_count"> (<t t-esc="rating_count"/>)</t>
                                    </a>
                                </li>
                            </ul>

                            <div class="tab-content py-4 o_wslides_tabs_content mb-4" id="courseMainTabContent">
                                <div t-att-class="'tab-pane fade %s' % ('show active' if active_tab == 'home' else '')" id="home" role="tabpanel" aria-labelledby="home-tab">
                                    <!-- ==== Error notification ==== -->
                                    <div t-if="access_error == 'course_content'" class="alert alert-danger alert-dismissable mb-1" role="alert">
                                        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                                        You need to join this course to access &quot;<t t-out="access_error_content_name"/>&quot;.
                                    </div>
                                    <!-- ==== Invitation Banner ==== -->
                                    <div t-if="invite_preview" class="o_wslides_identification_banner alert alert-success mb-4 p-2 px-3" role="alert">
                                        You have been invited to this course.
                                        <a class="o_underline" t-attf-href="/slides/#{channel.id}/identify?#{keep_query('invite_partner_id', 'invite_hash')}">
                                            <t t-if="is_partner_without_user">Sign up</t>
                                            <t t-else="">Log in</t>
                                        </a> to browse preview content and enroll.
                                    </div>
                                    <div t-if="channel.channel_type == 'training' and (channel_frontend_tags or channel.can_upload)"
                                         class="mb-1 pt-1">
                                        <t t-if="channel_frontend_tags">
                                            <t t-foreach="channel_frontend_tags" t-as="channel_tag">
                                                <span t-attf-class="badge o_wslides_channel_tag #{'o_color_'+str(channel_tag.color)}" t-esc="channel_tag.name"/>
                                            </t>
                                        </t>
                                        <a t-if="channel.can_upload"
                                            class="o_wslides_js_channel_tag_add badge text-bg-primary fw-normal m-1 o_not_editable"
                                            role="button"
                                            aria-label="Add Tag"
                                            href="#"
                                            t-att-data-channel-id="channel.id"
                                            t-att-data-channel-tag-ids="channel.tag_ids.ids">
                                            <span>Add Tag</span>
                                         </a>
                                    </div>
                                    <t t-if="channel.channel_type == 'training'" t-call="website_slides.course_slides_list"/>
                                    <t t-else="" t-call="website_slides.course_slides_cards"/>
                                </div>
                                <div t-if="channel.allow_comment" t-att-class="'tab-pane fade %s' % ('show active' if active_tab == 'review' else '')" id="review" role="tabpanel" aria-labelledby="review-tab">
                                    <t t-call="portal.message_thread">
                                        <t t-set="object" t-value="channel"/>
                                        <t t-set="hash" t-value="message_post_hash"/>
                                        <t t-set="pid" t-value="message_post_pid"/>
                                        <t t-set="display_rating" t-value="True"/>
                                        <t t-set="disable_composer" t-value="True"/>
                                    </t>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </t>
</template>

<template id="course_sidebar" name="Course Sidebar (infos, CTA)">
    <!-- Channel sidebar (aka general information + CTAs) -->
    <div class="o_wslides_course_sidebar bg-white px-3 py-2 py-md-3 mb-3 mb-md-5">

        <div class="o_wslides_sidebar_top d-flex justify-content-between">
            <t t-call="website_slides.course_join"/>
            <button t-attf-class="btn d-md-none bg-white ms-1 border #{'alert' if channel.is_member else ''} #{'align-self-start' if channel.is_member or channel.enroll == 'invite' else 'align-self-end'}" type="button" data-bs-toggle="collapse" data-bs-target="#o_wslides_sidebar_collapse" aria-expanded="false" aria-controls="o_wslides_sidebar_collapse">More info</button>
        </div>

        <div id="o_wslides_sidebar_collapse" class="collapse d-md-block">
            <table class="table table-sm mt-3">
                <tr t-if="channel.user_id">
                    <th class="border-top-0">Responsible</th>
                    <td class="border-top-0 text-break"><span t-esc="channel.user_id.display_name"/></td>
                </tr>
                <tr>
                    <th class="border-top-0">Last Update</th>
                    <td class="border-top-0"><t t-esc="channel.slide_last_update" t-options="{'widget': 'date'}"/></td>
                </tr>
                <tr t-if="channel.total_time">
                    <th class="border-top-0">Completion Time</th>
                    <td class="border-top-0"><t class="fw-bold" t-esc="channel.total_time" t-options="{'widget': 'duration', 'unit': 'hour', 'round': 'minute'}"/></td>
                </tr>
                <tr>
                    <th>Members</th>
                    <td><t t-esc="channel.members_count"/></td>
                </tr>
            </table>

            <div class="mt-3 d-grid o_not_editable">
                <button role="button" class="o_wslides_share btn btn-link" title="Share Channel"
                        aria-label="Share Channel" t-att-data-id="channel.id" t-att-data-name="channel.name"
                        t-att-data-url="channel.website_url" data-is-channel="True"
                        t-att-data-email-sharing="bool(channel.share_channel_template_id)">
                    <i class="fa fa-share-alt"/> Share
                </button>
            </div>
        </div>
    </div>
</template>

<template id="course_join">
    <div class="o_wslides_js_course_join flex-grow-1 d-grid">
        <t t-if="(invite_preview and channel.enroll in ['public', 'invite']) or (not channel.is_member and (channel.enroll == 'public' or (channel.is_member_invited and channel.enroll == 'invite')))">
           <div t-if="not invite_preview and channel.prerequisite_channel_ids and not channel.prerequisite_user_has_completed" class="text-center">
                <div class="alert my-0 bg-100">
                    <h6>
                        <i class="fa fa-lock"/>
                        <span>Course Locked</span>
                    </h6>
                    <div>
                        <small>
                            Finish
                            <a t-if="len(channel.prerequisite_channel_ids) == 1"
                                t-attf-href="/slides/{{channel.prerequisite_channel_ids[0].id}}"
                                t-out="channel.prerequisite_channel_ids[0].name"/>
                            <a t-else="" href="#" class="o_wslides_js_prerequisite_course"
                                t-att-data-channels="json.dumps(
                                    [{'course_id': course.id, 'course_name': course.name}
                                    for course in channel.prerequisite_channel_ids]
                                )">
                                courses
                            </a>
                            to unlock
                        </small>
                        <small>
                            or
                            <a role="button" class="o_wslides_js_course_join_link"
                                title="Start Course" aria-label="Start Course" href="#"
                                t-att-data-channel-id="channel.id"
                                t-att-data-channel-enroll="channel.enroll"
                                t-att-data-invite-hash="invite_hash"
                                t-att-data-invite-partner-id="invite_partner_id"
                                t-att-data-invite-preview="invite_preview"
                                t-att-data-is-member-or-invited="channel.is_member or channel.is_member_invited"
                                t-att-data-is-partner-without-user="is_partner_without_user">
                                <t t-if="channel.channel_type == 'documentation'">start</t>
                                <t t-else="">join</t>
                            </a> anyway
                        </small>
                    </div>
                </div>
            </div>
            <a t-else="" role="button" class="btn btn-primary btn-block o_wslides_js_course_join_link"
               title="Start Course" aria-label="Start Course" href="#"
               t-att-data-channel-id="channel.id"
               t-att-data-channel-enroll="channel.enroll"
               t-att-data-invite-hash="invite_hash"
               t-att-data-invite-partner-id="invite_partner_id"
               t-att-data-invite-preview="invite_preview"
               t-att-data-is-member-or-invited="channel.is_member or channel.is_member_invited"
               t-att-data-is-partner-without-user="is_partner_without_user">
                <span class="cta-title text_small_caps">
                    <t t-if="channel.channel_type == 'documentation'">Start this Course</t>
                    <t t-else="">Join this Course</t>
                </span>
            </a>
        </t>
        <div t-elif="not invite_preview and channel.enroll == 'invite' and not channel.is_member and not channel.is_member_invited" class="text-center">
            <div t-if="channel.prerequisite_channel_ids and not channel.prerequisite_user_has_completed" class="text-center">
                <div class="alert my-0 bg-100">
                    <h6>
                        <i class="fa fa-lock"/>
                        <span>Course Locked</span>
                    </h6>
                    <div>
                        <small>
                            Finish
                            <a t-if="len(channel.prerequisite_channel_ids) == 1"
                                t-attf-href="/slides/{{channel.prerequisite_channel_ids[0].id}}"
                                t-out="channel.prerequisite_channel_ids[0].name"/>
                            <a t-else="" href="#" class="o_wslides_js_prerequisite_course"
                                t-att-data-channels="json.dumps(
                                    [{'course_id': course.id, 'course_name': course.name}
                                    for course in channel.prerequisite_channel_ids]
                                )">
                                courses
                            </a>
                            to unlock
                        </small>
                        <small t-if="is_public_user">
                            or <a t-att-href="'/web/login?redirect=/slides/%s' % (slug(channel))">sign in</a> to request access
                        </small>
                        <small t-else="">
                            or <a href="#" class="o_wslides_js_channel_enroll" t-att-data-channel-id="channel.id">request</a> direct access
                        </small>
                    </div>
                </div>
            </div>
            <div t-else="" t-attf-class="alert my-0 bg-100 p-2 #{'o_wslides_js_channel_enroll' if not is_public_user and channel.user_id else ''}"
                t-att-data-channel-id="channel.id">
                Private Course
                <div t-if="is_public_user">
                    <small>
                        Please <a t-att-href="'/web/login?redirect=/slides/%s' % (slug(channel))">sign in</a> to contact responsible
                    </small>
                </div>
                <div t-elif="channel.has_requested_access">
                    <small class="text-success">
                        Request already sent
                    </small>
                </div>
                <div t-else="" class="o_wslides_enroll_msg">
                    <small>
                        <div t-field="channel.enroll_msg"/>
                    </small>
                </div>
            </div>
        </div>
        <t t-elif="channel.is_member">
            <button class="d-flex align-items-center alert my-0 px-2 px-xl-3 bg-100 w-100 o_wslides_js_channel_unsubscribe"
                    t-att-data-channel-id="channel.id"
                    t-att-data-is-follower="channel.message_is_follower"
                    t-att-data-enroll="channel.enroll"
                    t-att-data-visibility="channel.visibility">
                <t t-call="website_slides.slides_misc_user_image">
                    <t t-set="img_class" t-value="'rounded-circle me-1'"/>
                    <t t-set="img_style" t-value="'width: 1.4em; height: 1.4em;'"/>
                </t>
                <h6 class="d-flex flex-grow-1 my-0">You're enrolled</h6>
                <i class="fa fa-check"/>
                <i class="fa fa-times"/>
            </button>
            <div class="d-flex align-items-center pt-3">
                <span t-attf-class="o_wslides_channel_completion_completed badge text-bg-success mx-auto #{'d-none' if not channel.completed else ''}">
                    <i class="fa fa-check"/> Completed
                </span>
                <div t-attf-class="o_wslides_channel_completion_progressbar #{'d-none' if channel.completed else 'd-flex'} w-100 align-items-center">
                    <div class="progress flex-grow-1 bg-black-50" style="height: 6px;">
                        <div class="progress-bar" role="progressbar" t-attf-style="width: #{channel.completion}%" t-att-aria-valuenow="channel.completion" aria-valuemin="0" aria-valuemax="100" aria-label="Progress bar"></div>
                    </div>
                    <div class="ms-3 small">
                        <span class="o_wslides_progress_percentage" t-esc="channel.completion"/> %
                    </div>
                </div>
            </div>
        </t>
    </div>
</template>

<template id="course_slides_list" name="Training Course content: list">
    <div class="mb-5 o_wslides_slides_list" t-att-data-channel-id="channel.id">

        <ul class="o_wslides_js_slides_list_container list-unstyled">
            <t t-set="j" t-value="0"/>
            <t t-foreach="category_data" t-as="category">
                <t t-set="category_id" t-value="category['id'] if category['id'] else None"/>

                <li t-if="category['total_slides'] or channel.can_publish" t-att-class="'o_wslides_slide_list_category o_wslides_js_list_item mb-2' if category_id else 'mt-1'" t-att-data-slide-id="category_id" t-att-data-category-id="category_id">
                    <div t-att-data-category-id="category_id"
                         t-att-class="'o_wslides_slide_list_category_header position-relative d-flex justify-content-between align-items-center mt8 %s %s' % ('bg-white shadow-sm border-bottom-0' if category_id else 'border-0', 'o_wslides_js_category py-0' if channel.can_upload else 'py-2')">
                        <div t-att-class="'d-flex align-items-center me-auto ps-3 %s' % ('o_wslides_slides_list_drag' if channel.can_publish else '')">
                            <div t-if="channel.can_publish and category_id" class="o_wslides_slides_list_drag py-2 pe-3">
                                <i class="fa fa-bars"/>
                            </div>
                            <span t-if="category_id" t-field="category['category'].name"/>
                            <small t-if="not category['total_slides'] and category_id" class="ms-1 text-muted"><b>(empty)</b></small>
                        </div>
                        <t t-if="category_id">
                            <div class="d-flex o_not_editable">
                                <a  t-if="channel.can_publish"
                                    class="o_text_link text-danger o_wslides_js_category_delete px-3 py-2"
                                    role="button"
                                    aria-label="Delete Category"
                                    href="#"
                                    t-att-data-category-id="category_id">
                                    <i class="fa fa-trash"/>
                                </a>
                            </div>
                            <div class="d-flex ms-2 o_not_editable">
                                <span t-field="category['category'].total_slides"/><span class="ms-1">Lessons</span>
                                <span class="ms-1">&#183;</span>
                                <span class="ms-1" t-field="category['category'].completion_time" t-options="{'widget': 'duration', 'format' : 'short', 'unit': 'hour', 'round': 'minute'}"/>
                            </div>
                            <div class="d-flex border-start ms-2 o_not_editable">
                                <a  t-if="channel.can_upload"
                                    class="o_text_link o_wslides_js_slide_upload px-3 py-2"
                                    role="button"
                                    aria-label="Add Content"
                                    href="#"
                                    t-att-data-modules-to-install="modules_to_install"
                                    t-att-data-channel-id="channel.id"
                                    t-att-data-category-id="category_id"
                                    t-att-data-can-upload="channel.can_upload"
                                    t-att-data-can-publish="channel.can_publish">
                                    <i class="fa fa-plus me-1"/> <span class="d-none d-md-inline-block">Add Content</span>
                                </a>
                            </div>
                        </t>
                    </div>
                    <ul t-att-data-category-id="category_id" class="list-unstyled pb-1 border-top">
                        <li class="o_wslides_slides_list_slide o_not_editable border-0"/>
                        <li class="o_wslides_js_slides_list_empty border-0"/>

                        <t t-foreach="category['slides']" t-as="slide">
                            <t t-call="website_slides.course_slides_list_slide" />
                            <t t-set="j" t-value="j+1"/>
                        </t>
                    </ul>
                </li>
            </t>
        </ul>
        <div t-if="channel.can_upload" class="o_wslides_content_actions o_not_editable btn-group">
            <a  class="o_wslides_js_slide_upload me-1 border btn btn-primary"
                role="button"
                aria-label="Add Content"
                href="#"
                t-att-data-open-modal="enable_slide_upload"
                t-att-data-modules-to-install="modules_to_install"
                t-att-data-channel-id="channel.id"
                t-att-data-can-upload="channel.can_upload"
                t-att-data-can-publish="channel.can_publish"><i class="fa fa-plus me-1"/><span>Add Content</span></a>
            <a class="o_wslides_js_slide_section_add border btn btn-light bg-white" t-attf-channel_id="#{channel.id}"
                href="#" role="button"
                groups="website_slides.group_website_slides_officer"><i class="fa fa-folder-o me-1"/><span>Add Section</span></a>
        </div>
        <t t-if="not channel.slide_ids and channel.can_publish" t-call="website_slides.course_slides_list_sample"/>
        <t t-elif="slide_count == 0 and not channel.can_publish" t-call="website_slides.course_slides_list_placeholder"/>
    </div>
    <div t-field="channel.description_html"/>
</template>

<template id="course_slides_list_placeholder" name="Course Placeholder Content">
    <div class="my-5 d-flex">
        <div class="mx-auto">
            <img class="mx-auto mb-3" src="/web/static/img/smiling_face.svg"/>
            <div class="text-muted fw-bold">No lessons are available yet.</div>
        </div>
    </div>
</template>

<template id="course_slides_list_sample" name="Course Sample Content">
    <ul class="list-unstyled mt-3" style="opacity: 50%;">
        <li class="o_wslides_slide_list_category mb-2">
            <div class="o_wslides_slide_list_category_header position-relative d-flex justify-content-between align-items-center mt8 bg-white shadow-sm border-bottom-0 py-2">
                <div class="d-flex align-items-center ps-3">
                    <span class="text-muted">Common tasks for a computer scientist</span>
                </div>
            </div>
            <ul class="list-unstyled pb-1 border-top card">
                <div class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                <li class="o_wslides_slides_list_slide bg-white-50 border-top-0 d-flex align-items-center ps-2 py-1 pe-2">
                    <i class="fa fa-file-text py-2 mx-2"/>
                    <div class="text-truncate me-auto">
                        <span>Asking Question</span>
                    </div>
                    <div class="d-flex flex-row">
                        <span class="badge fw-bold m-1 text-bg-warning">
                            <i class="fa fa-fw fa-flag"/> 10 xp
                        </span>
                    </div>
                </li>
                <li class="o_wslides_slides_list_slide bg-white-50 border-top-0 d-flex align-items-center ps-2 py-1 pe-2">
                    <i class="fa fa-question-circle py-2 mx-2"/>
                    <div class="text-truncate me-auto">
                        <span>Asking the right question</span>
                    </div>
                </li>
                <li class="o_wslides_slides_list_slide bg-white-50 border-top-0 d-flex align-items-center ps-2 py-1 pe-2">
                    <i class="fa fa-file-pdf-o py-2 mx-2"/>
                    <div class="text-truncate me-auto">
                        <span>Answering Questions</span>
                    </div>
                    <div class="d-flex flex-row">
                        <span class="badge text-bg-info badge-arrow-right fw-normal m-1">New</span>
                    </div>
                </li>
            </ul>
        </li>
        <li class="o_wslides_slide_list_category mb-2">
            <div class="o_wslides_slide_list_category_header position-relative d-flex justify-content-between align-items-center mt8 bg-white shadow-sm border-bottom-0 py-2">
                <div class="d-flex align-items-center ps-3">
                    <span class="text-muted">Parts of computer science</span>
                </div>
            </div>
            <ul class="list-unstyled pb-1 border-top card">
                <div class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                <li class="o_wslides_slides_list_slide bg-white-50 border-top-0 d-flex align-items-center ps-2 py-1 pe-2">
                    <i class="fa fa-file-pdf-o py-2 mx-2"/>
                    <div class="text-truncate me-auto">
                        <span>Mathematics</span>
                    </div>
                </li>
                <li class="o_wslides_slides_list_slide bg-white-50 border-top-0 d-flex align-items-center ps-2 py-1 pe-2">
                    <i class="fa fa-file-pdf-o py-2 mx-2"/>
                    <div class="text-truncate me-auto">
                        <span>Science</span>
                    </div>
                </li>
                <li class="o_wslides_slides_list_slide bg-white-50 border-top-0 d-flex align-items-center ps-2 py-1 pe-2">
                    <i class="fa fa-play py-2 mx-2"/>
                    <div class="text-truncate me-auto">
                        <span>Logic</span>
                    </div>
                    <div class="d-flex flex-row">
                        <span class="badge text-bg-success fw-normal m-1">Preview</span>
                    </div>
                </li>
            </ul>
        </li>
    </ul>
</template>

<template id="course_slides_list_slide" name="Slide template for a training channel">
    <li t-att-index="j" t-att-data-slide-id="slide.id" t-att-data-category-id="category_id" t-attf-class="o_wslides_slides_list_slide o_wslides_js_list_item bg-white-50 border-top-0 d-flex align-items-center ps-2 #{'py-1 pe-2' if not channel.can_upload else ''}">
        <div t-if="channel.can_publish" class=" o_wslides_slides_list_drag m-2 o_not_editable">
            <i class="fa fa-sort"></i>
        </div>
        <i t-attf-class="fa #{slide.slide_icon_class} py-2 mx-2"/>
        <div class="text-truncate me-auto">
            <a t-if="not invite_preview and (slide.is_preview or channel.is_member or channel.can_publish)" class="o_wslides_js_slides_list_slide_link" t-attf-href="/slides/slide/#{slug(slide)}">
                <span t-field="slide.name"/>
            </a>
            <span t-else="">
                <span t-esc="slide.name"/>
            </span>
        </div>

        <div class="d-flex flex-row o_not_editable align-items-center">
            <a name="o_wslides_list_slide_add_quizz" t-if="channel.can_upload and not slide.question_ids" t-attf-href="/slides/slide/#{slug(slide)}?quiz_quick_create" aria-label="Add quiz">
                <span class="badge text-bg-primary badge-hide fw-normal m-1">Add Quiz</span>
            </a>
            <a t-if="channel.can_upload" href="#" name="o_wslides_slide_toggle_is_preview" aria-label="Preview">
                <span t-att-data-slide-id="slide.id" t-attf-class="o_wslides_js_slide_toggle_is_preview badge #{'text-bg-success' if slide.is_preview else 'text-bg-primary badge-hide'} fw-normal m-1"><span>Preview</span></span>
            </a>
            <t t-elif="slide.is_preview and not channel.is_member">
                <span class="badge text-bg-success fw-normal m-1"><span>Preview</span></span>
            </t>
            <span t-if="slide.is_new_slide and not channel_progress[slide.id].get('completed')" class="badge text-bg-info badge-arrow-right fw-normal m-1">
                New
            </span>
            <span t-if="slide.question_ids" t-att-class="'badge fw-bold m-1 %s' % ('text-bg-success' if channel_progress[slide.id].get('completed') else 'text-bg-warning')">
                <i t-attf-class="fa fa-fw #{'fa-check' if channel_progress[slide.id].get('completed') else 'fa-flag'}"/>
                <t t-esc="channel_progress[slide.id].get('quiz_karma_won', 0) if channel_progress[slide.id].get('completed') else channel_progress[slide.id].get('quiz_karma_gain', 0)"/> xp
            </span>
            <span class="badge text-bg-danger fw-normal m-1" t-if="not slide.website_published">Unpublished</span>
        </div>

        <div t-if="channel.is_member or channel.can_publish" class="pt-2 pb-2 border-start ms-2 me-2 ps-2 d-flex flex-row align-items-center o_wslides_slides_list_slide_controls o_not_editable">
            <t t-call="website_slides.slide_sidebar_done_button">
                <t t-set="is_member" t-value="channel.is_member"/>
                <t t-set="slide_completed" t-value="channel_progress[slide.id].get('completed')"/>
                <t t-set="use_slide_icon" t-value="False"/>
            </t>
            <span t-if="channel.can_publish" class="d-none d-md-flex">
                <a t-if="slide.slide_category == 'article'" class="mx-2 o_text_link text-primary o_not_editable" target="_blank" t-attf-href="/slides/slide/#{slug(slide)}?enable_editor=1" title="Edit"><span class="fa fa-pencil"/></a>
                <a t-else="" class="mx-2 o_text_link text-primary o_not_editable" target="_blank" t-attf-href="/odoo/slide.slide/{{slide.id}}" title="Edit in backend"><span class="fa fa-pencil"/></a>
                <a href="#" t-att-data-slide-id="slide.id" class="o_text_link text-danger mx-2 o_wslides_js_slide_archive o_not_editable" title="Delete"><span class="fa fa-trash"/></a>
            </span>
        </div>
    </li>
</template>

<!-- ======= Documentation Course content: cards / categories=======  -->
<template id="course_promoted_slide" name="Documentation Course content: promoted slide">
    <div class="o_wslides_promoted_slide">
        <div t-if="not search and not search_slide_category and slide_promoted" class="container py-1 mb-2">
            <div class="card flex-column flex-lg-row">
                <a t-if="not invite_preview and (slide_promoted.is_preview or channel.is_member or is_slides_publisher)" t-attf-href="/slides/slide/#{slug(slide_promoted)}#{query_string}" class="w-100 w-lg-50 flex-shrink-0 rounded">
                    <div t-field="slide_promoted.image_1920" t-options="{'widget': 'image', 'style': 'height:100%', 'preview_image': 'image_512'}" class="h-100"/>
                </a>
                <div t-else="" class="w-100 w-lg-50 flex-shrink-0 rounded">
                    <div t-field="slide_promoted.channel_id.image_1920" t-options="{'widget': 'image', 'style': 'height:100%', 'preview_image': 'image_512'}" class="h-100"/>
                </div>

                <div class="card-body">
                    <a t-if="not invite_preview and (slide_promoted.is_preview or channel.is_member or is_slides_publisher)"
                    t-attf-href="/slides/slide/#{slug(slide_promoted)}#{query_string}"
                    class="h4 d-block pb-2 border-bottom" t-att-title="slide_promoted.name" t-field="slide_promoted.name"/>
                    <h4 t-else="" class="text-muted pb-2 border-bottom" t-field="slide_promoted.name"/>
                    <div class="o_wslides_desc_truncate_10 mt-3" t-field="slide_promoted.description"/>
                    <div t-if="slide_promoted.tag_ids" class="mt-2 pt-1">
                        <t t-foreach="slide_promoted.tag_ids" t-as="tag">
                            <h4 t-if="invite_preview" class="badge text-bg-info" t-esc="tag.name"/>
                            <a t-else="" t-attf-href="/slides/#{slug(slide_promoted.channel_id)}/tag/#{slug(tag)}" class="badge text-bg-primary" t-esc="tag.name"/>
                        </t>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="course_slides_cards" name="Documentation Course content: cards / categories">
    <div t-if="not invite_preview" class="o_wslides_lesson_nav mb-4">
        <div class="container">
            <div class="row">
                <nav class="navbar navbar-expand-lg navbar-light bg-transparent col">
                    <a class="navbar-brand d-lg-none" href="#">Filter &amp; order</a>

                    <div class="ms-auto d-lg-none" t-if="search_slide_category or search">
                        <a t-att-href="'/slides/%s' % (slug(channel))" class="btn btn-info me-3">
                            <i class="fa fa-eraser me-1"/>Clear filters
                        </a>
                    </div>

                    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
                        <span class="navbar-toggler-icon"></span>
                    </button>

                    <div class="collapse navbar-collapse row" id="navbarSupportedContent">
                        <div class="col-12">
                            <ul class="navbar-nav me-lg-auto align-items-lg-center">

                                <t t-set="slide_category_keys" t-value="slide_categories.keys()"/>
                                <t t-foreach="slide_category_keys" t-as="slide_category_key">
                                    <t t-if="search_category">
                                        <li t-if="search_category['nbr_%s' % slide_category_key] > 0" class="nav-item">
                                            <a t-att-href="'/slides/%s/category/%s?%s' % (slug(channel), slug(search_category), keep_query(slide_category=slide_category_key))"
                                               t-att-class="'nav-link d-flex align-items-center justify-content-between me-1 %s' % ('active' if search_slide_category == slide_category_key else '')">
                                               <t t-esc="slide_categories[slide_category_key]"/>
                                               <span t-attf-class="badge ms-1 #{'text-bg-info' if search_slide_category == slide_category_key else 'bg-400'}" t-esc="search_category['nbr_%s' % slide_category_key]"/>
                                            </a>
                                        </li>
                                    </t>
                                    <t t-else="">
                                        <li t-if="channel['nbr_%s' % slide_category_key] > 0" class="nav-item">
                                            <a t-att-href="'/slides/%s?%s' % (slug(channel), keep_query(slide_category=slide_category_key))"
                                               t-att-class="'nav-link d-flex align-items-center justify-content-between me-1 %s' % ('active' if search_slide_category == slide_category_key else '')">
                                               <t t-esc="slide_categories[slide_category_key]"/>
                                               <span t-attf-class="badge ms-1 #{'text-bg-info' if search_slide_category == slide_category_key else 'bg-400'}" t-esc="channel['nbr_%s' % slide_category_key]"/>
                                            </a>
                                        </li>
                                    </t>
                                </t>
                            </ul>
                        </div>

                        <div class="col-12 d-flex align-items-start">
                            <ul class="navbar-nav me-auto">
                                <li class="nav-item dropdown ms-lg-auto">
                                    <a class="nav-link dropdown-toggle dropdown-toggle align-items-center d-flex" type="button" id="slidesChannelDropdownSort"
                                       data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="false" href="#">
                                        <b>Order by</b>
                                        <span class="d-none d-xl-inline">:
                                            <t t-if="sorting == 'most_voted'">Most Voted</t>
                                            <t t-elif="sorting == 'most_viewed'">Most Viewed</t>
                                            <t t-else="">Newest</t>
                                        </span>
                                    </a>
                                    <div class="dropdown-menu" aria-labelledby="slidesChannelDropdownSort" role="menu">
                                        <h6 class="dropdown-header">Sort by</h6>
                                        <a role="menuitem" t-att-href="'/slides/%s?%s' % (slug(channel), keep_query('slide_category', sorting='latest'))"
                                           t-att-class="'dropdown-item %s' % ('active' if sorting and sorting == 'latest' else '')">Newest</a>
                                        <a role="menuitem" t-att-href="'/slides/%s?%s' % (slug(channel), keep_query('slide_category', sorting='most_voted'))"
                                           t-att-class="'dropdown-item %s' % ('active' if sorting and sorting == 'most_voted' else '')">Most Voted</a>
                                        <a role="menuitem" t-att-href="'/slides/%s?%s' % (slug(channel), keep_query('slide_category', sorting='most_viewed'))"
                                           t-att-class="'dropdown-item %s' % ('active' if sorting and sorting == 'most_viewed' else '')">Most Viewed</a>
                                    </div>
                                </li>
                            </ul>

                            <div class="me-3 d-none d-lg-inline-block">
                                <a t-if="search_slide_category or search" t-att-href="'/slides/%s' % (slug(channel))" class="btn btn-sm btn-info ms-1">
                                    <i class="fa fa-eraser me-1"/>Clear filters
                                </a>
                            </div>

                            <form t-attf-action="/slides/#{slug(channel)}" role="search" method="get" class="my-2 my-lg-0">
                                <div class="input-group position-relative">
                                    <input type="text" class="form-control border" name="search" placeholder="Search in content" t-att-value="search"/>
                                    <button class="btn border" type="submit" aria-label="Search" title="Search">
                                        <i class="fa fa-search"/>
                                    </button>
                                </div>
                            </form>
                        </div>
                    </div>
                </nav>
            </div>
        </div>
    </div>

    <div class="container">
        <div class="row">
            <div class="mb-2 pt-1 text-start col">
                <t t-if="channel_frontend_tags">
                    <t t-foreach="channel_frontend_tags" t-as="channel_tag">
                        <span t-attf-class="badge o_wslides_channel_tag #{'o_color_'+str(channel_tag.color)}" t-esc="channel_tag.name"/>
                    </t>
                </t>
                <a t-if="channel.can_upload"
                    class="o_wslides_js_channel_tag_add badge text-bg-primary fw-normal m-1"
                    role="button"
                    aria-label="Add Tag"
                    href="#"
                    t-att-data-channel-id="channel.id"
                    t-att-data-channel-tag-ids="channel.tag_ids.ids">
                    <span>Add Tag</span>
                </a>
            </div>

            <div t-if="channel.can_upload" class="text-end pb-2 col-auto">
                <a class="btn btn-primary py-1 o_wslides_js_slide_upload"
                    title="Upload Document" role="button"
                    href="#"
                    t-att-data-channel-id="channel.id"
                    t-att-data-can-upload="channel.can_upload"
                    t-att-data-can-publish="channel.can_publish">
                    <i class="fa fa-cloud-upload me-1"/>Add Content
                </a>
                <a class="btn btn-secondary py-1 o_wslides_js_slide_section_add"
                    title="Add Section" role="button"
                    href="#"
                    t-att-channel_id="channel.id">
                    <i class="fa fa-folder-o me-1"/>Add a section
                </a>
            </div>
        </div>
    </div>
    <!-- Featured lesson  -->
    <t t-if="channel.promote_strategy != 'none'">
        <t t-call="website_slides.course_promoted_slide"/>
    </t>

    <div class="container py-2">
        <t t-if="search">
            <t t-set="search_results_number" t-value="0"/>
            <t t-foreach="category_data" t-as="category">
                <t t-set="search_results_number" t-value="search_results_number + category['total_slides']"/>
            </t>
            <div t-if="search_results_number == 0" class="alert alert-info mt-4 mb-5 text-center">
                No content was found using your search <span class="fw-bold" t-esc="search"/>.
            </div>
        </t>

        <t t-foreach="category_data" t-as="category">
            <div class="mb-2" t-if="(category['slides'] or channel.can_publish) and (search_category and search_category.id == category['id'] or not search_category)">
                <t t-set="is_empty_editable" t-value="not category['slides'] and channel.can_publish"/>
                <div class="d-flex align-items-center justify-content-between border-bottom pb-2 mb-3" t-if="category['id'] and query_string != '?search_uncategorized=1'">
                    <h5 t-attf-class="m-0 #{'text-muted' if is_empty_editable else ''}"><t t-esc="category['name']"/></h5>
                    <a t-if="category['id'] and not is_empty_editable" t-attf-href="/slides/#{channel.id}/category/#{category['id']}?#{keep_query('invite_hash', 'invite_partner_id') if invite_preview else ''}">View all</a>
                </div>
                <div class="d-flex align-items-center justify-content-between border-bottom pb-2 mb-3" t-if="not category['id'] and len(category['slides']) > 0">
                    <h5 t-if="len(category_data) > 1" t-attf-class="m-0 #{'text-muted' if is_empty_editable else ''}"><t t-esc="category['name']"/></h5>
                    <a t-attf-href="/slides/#{channel.id}?uncategorized=1&amp;#{keep_query('invite_hash', 'invite_partner_id') if invite_preview else ''}">View all</a>
                </div>
                <div class="row mx-n2">
                    <t t-foreach="category['slides']" t-as="slide">
                        <div class="col-12 col-sm-6 col-lg-4 px-2 d-flex" t-call="website_slides.lesson_card"/>
                    </t>
                </div>
            </div>
        </t>

        <div class="row">
            <div class="col" t-field="channel.description_html"/>
        </div>
    </div>
    <t t-if="search_category or search_uncategorized">
        <div class="d-flex justify-content-center pb-5">
            <t t-call="website_profile.pager_nobox"></t>
        </div>
    </t>
</template>

<template id='lesson_card' name="Lesson Card">
    <div class="card w-100 o_wslides_lesson_card mb-4">
        <t t-if="slide.is_new_slide and not channel_progress[slide.id].get('completed')" t-call="website_slides.course_card_information"/>
        <t t-set="can_access" t-value="not invite_preview and (slide.is_preview or channel.is_member or channel.can_publish)"/>
        <a t-if="can_access" t-attf-href="/slides/slide/#{slug(slide)}#{query_string}" t-title="slide.name" style="height:150px">
            <div t-field="slide.image_1920" t-options="{'widget': 'image', 'preview_image': 'image_512'}" class="o_wslides_background_image h-100"/>
        </a>
        <div t-else="" class="o_wslides_background_image" style="height:150px">
            <div t-field="slide.channel_id.image_1920" t-options="{'widget': 'image', 'preview_image': 'image_512'}" class="o_wslides_background_image h-100"/>
        </div>
        <i t-if="channel_progress[slide.id].get('completed')" class="position-absolute py-1 px-2 h5 fa fa-check-circle text-primary" style="right:0; top:0;"/>

        <div class="card-body d-flex flex-column px-3">
            <a t-if="can_access" class="card-title h5 o_wslides_desc_truncate_2" t-attf-href="/slides/slide/#{slug(slide)}#{query_string}" t-esc="slide.name"/>
            <span t-else="" class="card-title h5 o_wslides_desc_truncate_2 text-muted" t-esc="slide.name"/>
            <div class="text-muted o_wslides_desc_truncate_2 mb-2" t-if="slide.is_preview or (not slide.is_published and user.has_group('website_slides.group_website_slides_officer'))">
                <span t-if="slide.is_preview" class="badge text-bg-info">Preview</span>
                <span t-if="not slide.is_published and channel.can_publish" class="badge text-bg-danger">Unpublished</span>
            </div>
            <div class="card-text mb-auto">
                <div class="o_wslides_desc_truncate_3 fw-light oe_no_empty" t-field="slide.description"/>
            </div>
            <div class="text-muted o_wslides_desc_truncate_2 my-2">
                <t t-foreach="slide.tag_ids" t-as="tag">
                    <h4 t-if="invite_preview" class="badge text-bg-info" t-esc="tag.name"/>
                    <a t-else="" t-attf-href="/slides/#{slug(slide.channel_id)}/tag/#{slug(tag)}" class="badge text-bg-primary" t-esc="tag.name"/>
                </t>
            </div>
            <span t-if="channel.is_member and channel_progress[slide.id].get('completed')" class="badge text-bg-success align-self-start"><i class="fa fa-check me-1"/>Completed</span>
        </div>
        <div class="card-footer bg-white text-600">
            <div class="d-flex align-items-center small">
                <span class="fw-bold me-auto" t-field="slide.completion_time" t-options='{"widget": "float_time"}'/>
                <div class="o_wslides_js_slide_like">
                    <span t-attf-class="d-inline-block text-center o_wslides_js_slide_like_up #{'disabled' if not channel.can_vote else ''}" tabindex="0" data-bs-toggle="popover" t-att-data-slide-id="slide.id" t-att-data-user-vote="slide.user_vote">
                        <i t-attf-class="fa fa-1x #{'fa-thumbs-up' if slide.user_vote == 1 else 'fa-thumbs-o-up'}" role="img" aria-label="Likes" title="Like"></i>
                        <span t-field="slide.likes" t-options="{'format_decimalized_number': True}"/>
                    </span>
                    <span t-attf-class="d-inline-block text-center ms-1 o_wslides_js_slide_like_down #{'disabled' if not channel.can_vote else ''}" tabindex="0" data-bs-toggle="popover" t-att-data-slide-id="slide.id" t-att-data-user-vote="slide.user_vote">
                        <i t-attf-class="fa fa-1x #{'fa-thumbs-down' if slide.user_vote == -1 else 'fa-thumbs-o-down'}" role="img" aria-label="Dislikes" title="Dislike"></i>
                        <span t-field="slide.dislikes" t-options="{'format_decimalized_number': True}"/>
                    </span>
                </div>
            </div>
        </div>
    </div>
</template>

</data></odoo>

```

## File: views\website_slides_templates_homepage.xml

```xml
<?xml version="1.0" ?>
<odoo><data>

<!-- Channels home template -->
<template id='courses_home' name="Odoo Courses Homepage">
    <t t-set="body_classname" t-value="'o_wslides_body'"/>
    <t t-call="website.layout">
        <div id="wrap" class="wrap o_wslides_wrap">
            <div class="oe_structure oe_empty">
            <section class="s_banner overflow-hidden" style="background-color:(0, 0, 0, 0); background-image: url(&quot;/website_slides/static/src/img/banner_default.svg&quot;); background-size: cover; background-position: 55% 65%" data-snippet="s_banner">
                <div class="container align-items-center d-flex mb-5 mt-lg-5 pt-lg-4 pb-lg-1">
                    <div class="text-white">
                        <h1 class="display-3 mb-0">Reach new heights</h1>
                        <h2 class="mb-4">Start your online course today!</h2>
                        <div class="row mt-1 mb-3">
                            <div class="col">
                                <p>Skill up and have an impact! Your business career starts here.<br/>Time to start a course.</p>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
            </div>
            <div class="container mt16 o_wslides_home_nav position-relative">
                <nav class="navbar navbar-expand-lg navbar-light shadow-sm">
                    <t t-call="website.website_search_box_input">
                        <t t-set="_form_classes" t-valuef="o_wslides_nav_navbar_right order-lg-3"/>
                        <t t-set="search_type" t-valuef="slides"/>
                        <t t-set="action" t-valuef="/slides/all"/>
                        <t t-set="display_description" t-valuef="true"/>
                        <t t-set="display_detail" t-valuef="false"/>
                        <t t-set="placeholder">Search courses</t>
                        <input type="hidden" name="prevent_redirect" value="True"/>
                    </t>
                    <button class="navbar-toggler px-2 order-1" type="button"
                        data-bs-toggle="collapse" data-bs-target="#navbarSlidesHomepage"
                        aria-controls="navbarSlidesHomepage" aria-expanded="false" aria-label="Toggle navigation">
                        <span class="navbar-toggler-icon"/>
                    </button>
                    <div class="collapse navbar-collapse order-2" id="navbarSlidesHomepage">
                        <div class="navbar-nav pt-3 pt-lg-0">
                            <a class="nav-link nav-link me-md-2 o_wslides_home_all_slides" href="/slides/all"><i class="fa fa-graduation-cap me-1"/>All courses</a>
                        </div>
                    </div>
                </nav>
                <div class="o_wprofile_email_validation_container">
                    <t t-call="website_profile.email_validation_banner">
                        <t t-set="redirect_url" t-value="'/slides'"/>
                        <t t-set="send_alert_classes" t-value="'alert alert-danger alert-dismissable mt-4 mb-0'"/>
                        <t t-set="done_alert_classes" t-value="'alert alert-success alert-dismissable mt-4 mb-0'"/>
                        <t t-set="additional_validation_email_message"> and join this Community</t>
                        <t t-set="additional_validated_email_message"> You may now participate in our eLearning.</t>
                    </t>
                </div>
            </div>

            <div class="container o_wslides_home_main">
                <div class="row">
                    <t t-set="has_side_column" t-value="is_view_active('website_slides.toggle_leaderboard')"/>
                    <t t-if="is_public_user">
                        <div t-if="has_side_column" class="col-lg-3 order-3 order-lg-2">
                            <div class="row">
                                <div class="col-12 col-md-5 col-lg-12">
                                    <div class="ps-md-5 ps-lg-0">
                                        <t t-call="website_slides.slides_home_users_small"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </t>
                    <div t-else="" class="col-lg-3 order-lg-2">
                        <t t-set="has_side_column" t-value="True"/>
                        <div class="o_wslides_home_aside_loggedin card p-3 p-lg-0 mb-4">
                            <div class="o_wslides_home_aside_title">
                                <div class="d-flex align-items-center">
                                    <t t-call="website_slides.slides_misc_user_image">
                                        <t t-set="img_class" t-value="'rounded-circle me-1'"/>
                                        <t t-set="img_style" t-value="'width: 22px; height: 22px;'"/>
                                    </t>
                                    <h5 t-esc="user.name" class="d-flex flex-grow-1 mb-0"/>
                                    <a class="d-none d-lg-block" t-att-href="'/profile/user/%s' % user.id">View</a>
                                    <a class="d-lg-none btn btn-sm bg-white border" href="#" data-bs-toggle="collapse" data-bs-target="#o_wslides_home_aside_content">More info</a>
                                </div>
                                <hr class="d-none d-lg-block mt-2 mb-2 mb-1"/>
                            </div>
                            <div id="o_wslides_home_aside_content" class="collapse d-lg-block">
                                <div class="row g-0 mb-5 mt-3 mt-lg-0">
                                    <div class="col-12 col-sm-6 col-lg-12">
                                        <t t-call="website_slides.slides_home_user_profile_small"/>
                                    </div>
                                    <div class="col-12 col-sm-6 col-lg-12 ps-md-5 ps-lg-0 mt-lg-4">
                                        <t t-call="website_slides.slides_home_user_achievements_small"/>
                                    </div>
                                    <div class="col-12 col-md-7 col-lg-12 ps-md-5 ps-lg-0 mt-lg-4 mb-3">
                                        <t t-call="website_slides.slides_home_achievements_small"/>
                                    </div>
                                    <div class="col-12 col-sm-6 col-lg-12 ps-md-5 ps-lg-0 mt-lg-4">
                                        <t t-call="website_slides.slides_home_users_small"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div t-att-class="'col-lg-9 pe-lg-5 order-lg-1' if has_side_column else 'col-lg pr-lg'">
                        <div t-if="invite_error_msg" role="alert" class="o_not_editable alert alert-danger text-center" t-esc="invite_error_msg"/>
                        <div class="o_wslides_home_content_section mb-3"
                            t-if="not channels_popular">
                            <p class="h2">No Course created yet.</p>
                            <p groups="website_slides.group_website_slides_officer">Click on "New" in the top-right corner to write your first course.</p>
                        </div>
                        <t t-if="channels_my">
                            <t t-set="void_count" t-value="3 - len(channels_my[:3])"/>
                            <div class="o_wslides_home_content_section mb-3">
                                <div class="row o_wslides_home_content_section_title align-items-center">
                                    <div class="col">
                                        <a href="/slides/all?my=1" class="float-end">View all</a>
                                        <h5 class="m-0">My courses</h5>
                                        <hr class="mt-2 mb-2"/>
                                    </div>
                                </div>
                                <div class="row mx-n2 mt8">
                                    <t t-foreach="channels_my[:3]" t-as="channel">
                                        <div class="col-md-4 col-sm-6 px-2 col-xs-12 d-flex">
                                            <t t-call="website_slides.course_card"/>
                                        </div>
                                    </t>
                                </div>
                            </div>
                        </t>
                        <div class="o_wslides_home_content_section mb-3"
                            t-if="channels_popular">
                            <div class="row o_wslides_home_content_section_title align-items-center">
                                <div class="col">
                                    <a href="slides/all" class="float-end">View all</a>
                                    <h5 class="m-0">Most popular courses</h5>
                                    <hr class="mt-2 mb-2"/>
                                </div>
                            </div>
                            <div class="row mx-n2 mt8">
                                <t t-foreach="channels_popular[:3]" t-as="channel">
                                    <div class="col-md-4 col-sm-6 px-2 col-xs-12 d-flex">
                                        <t t-call="website_slides.course_card"/>
                                    </div>
                                </t>
                            </div>
                        </div>
                        <div class="o_wslides_home_content_section mb-3"
                            t-if="channels_newest">
                            <div class="row o_wslides_home_content_section_title align-items-center">
                                <div class="col">
                                    <a href="slides/all" class="float-end">View all</a>
                                    <h5 class="m-0">Newest courses</h5>
                                    <hr class="mt-2 mb-2"/>
                                </div>
                            </div>
                            <div class="row mx-n2 mt8">
                                <t t-foreach="channels_newest[:3]" t-as="channel">
                                    <div class="col-md-4 col-sm-6 px-2 col-xs-12 d-flex">
                                        <t t-call="website_slides.course_card"/>
                                    </div>
                                </t>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <t t-call="website_slides.courses_footer"></t>
        </div>
    </t>
</template>

<!-- Channel all/main template -->
<template id='courses_all' name="Odoo All Courses">
    <t t-set="body_classname" t-value="'o_wslides_body'"/>
    <t t-call="website.layout">
        <div id="wrap" class="wrap o_wslides_wrap">
            <!-- Repeat structure for every section to allow customization through website editor. !-->
            <div class="oe_structure oe_empty" t-if="search_my">
                <section class="s_banner" data-snippet="s_banner"
                         style="background-color:(0, 0, 0, 0); background-image: url(&quot;/website_slides/static/src/img/banner_default_all.svg&quot;); background-size: cover; background-position: 80% 20%">
                    <div class="container py-5"><h1 class="display-3 mb-0 text-white">My Courses</h1></div>
                </section>
            </div>
            <div class="oe_structure oe_empty" t-elif="search_slide_category == 'certification'">
               <section class="s_banner" data-snippet="s_banner"
                        style="background-color:(0, 0, 0, 0); background-image: url(&quot;/website_slides/static/src/img/banner_default_all.svg&quot;); background-size: cover; background-position: 80% 20%">
                    <div class="container py-5"><h1 class="display-3 mb-0 text-white">Certifications</h1></div>
                </section>
            </div>
            <div class="oe_structure oe_empty" t-else="">
                <section class="s_banner" data-snippet="s_banner"
                         style="background-color:(0, 0, 0, 0); background-image: url(&quot;/website_slides/static/src/img/banner_default_all.svg&quot;); background-size: cover; background-position: 80% 20%">
                    <div class="container py-5"><h1 class="display-3 mb-0 text-white">All Courses</h1></div>
                </section>
            </div>
            <div class="container mt16 o_wslides_home_nav position-relative">
                <!-- Navbar dynamically composed using displayed channel tag groups. -->
                <nav class="navbar navbar-expand-md navbar-light shadow-sm ps-0">
                    <div class="navbar-nav border-end">
                        <a class="nav-link nav-item px-3" href="/slides"><i class="oi oi-chevron-left"/></a>
                    </div>
                    <!-- Clear filtering (mobile)-->
                    <div class="text-nowrap ms-auto d-md-none" t-if="search_slide_category or search_my or search_tags">
                        <a href="/slides/all" class="btn btn-info me-2" role="button" title="Clear filters">
                            <i class="fa fa-eraser"/> Clear filters
                        </a>
                    </div>
                    <t t-else="" t-call="website.website_search_box_input">
                        <!-- Search box (mobile)-->
                        <t t-set="_form_classes" t-valuef="o_wslides_nav_navbar_right d-md-none"/>
                        <t t-set="search_type" t-valuef="slides"/>
                        <!-- No action: remain on same URL -->
                        <t t-set="display_description" t-valuef="true"/>
                        <t t-set="display_detail" t-valuef="false"/>
                        <t t-set="placeholder">Search courses</t>
                        <t t-set="search" t-value="original_search or search_term"/>
                        <input t-if="search_my" type="hidden" name="my" t-att-value="1"/>
                        <input t-if="search_slide_category" type="hidden" name="slide_category" t-att-value="search_slide_category" />
                        <input type="hidden" name="prevent_redirect" value="True"/>
                    </t>
                    <button class="navbar-toggler px-1" type="button"
                        data-bs-toggle="collapse" data-bs-target="#navbarTagGroups"
                        aria-controls="navbarTagGroups" aria-expanded="false" aria-label="Toggle navigation">
                        <span class="navbar-toggler-icon small"/>
                    </button>
                    <div class="collapse navbar-collapse" id="navbarTagGroups">
                        <t t-set="search_tag_groups" t-value="search_tags.mapped('group_id')"/>
                        <ul class="navbar-nav flex-grow-1">
                            <t t-foreach="tag_groups" t-as="tag_group">
                                <t t-set="group_frontend_tags" t-value="tag_group.tag_ids.filtered(lambda tag: tag.color)"/>
                                <li class="nav-item dropdown ml16" t-if="group_frontend_tags">
                                    <a t-att-class="'nav-link dropdown-toggle %s' % ('active' if tag_group in search_tag_groups else '')"
                                        href="/slides/all"
                                        t-att-data-bs-target="'#navToogleTagGroup%s' % tag_group.id"
                                        role="button" data-bs-toggle="dropdown"
                                        aria-haspopup="true" aria-expanded="false"
                                        t-esc="tag_group.name"/>
                                    <div class="dropdown-menu" t-att-id="'navToogleTagGroup%s' % tag_group.id">
                                        <t t-foreach="group_frontend_tags" t-as="tag">
                                            <span t-att-class="'post_link cursor-pointer dropdown-item %s' % ('active' if tag in search_tags else '')"
                                                t-att-data-post="slide_query_url(tag=slugify_tags(search_tags.ids, toggle_tag_id=tag.id), my=search_my, search=search_term, slide_category=search_slide_category, prevent_redirect=True)"
                                                t-esc="tag.name"/>
                                        </t>
                                    </div>
                                </li>
                            </t>
                        </ul>
                        <!-- Clear filtering (desktop)-->
                        <div class="ms-auto d-none d-md-flex" t-if="search_slide_category or search_my or search_tags">
                            <a href="/slides/all" class="btn btn-info text-nowrap me-2" role="button" title="Clear filters">
                                <i class="fa fa-eraser"/> Clear filters
                            </a>
                        </div>
                        <!-- Search box (desktop) -->
                        <t t-call="website.website_search_box_input">
                            <t t-set="_form_classes" t-valuef="o_wslides_nav_navbar_right d-none d-md-flex"/>
                            <t t-set="search_type" t-valuef="slides"/>
                            <!-- No action: remain on same URL -->
                            <t t-set="display_description" t-valuef="true"/>
                            <t t-set="display_detail" t-valuef="false"/>
                            <t t-set="placeholder">Search courses</t>
                            <t t-set="search" t-value="original_search or search_term"/>
                            <input t-if="search_my" type="hidden" name="my" t-att-value="1"/>
                            <input t-if="search_slide_category" type="hidden" name="slide_category" t-att-value="search_slide_category" />
                            <input type="hidden" name="prevent_redirect" value="True"/>
                        </t>
                    </div>
                </nav>
                <div class="o_wprofile_email_validation_container mb16 mt16">
                    <t t-call="website_profile.email_validation_banner">
                        <t t-set="redirect_url" t-value="'/slides'"/>
                        <t t-set="additional_validation_email_message"> and join this Community</t>
                        <t t-set="additional_validated_email_message"> You may now participate in our eLearning.</t>
                    </t>
                </div>
                <!-- Display tags -->
                <t t-if="search_my">
                      <span class="align-items-baseline border d-inline-flex ps-2 rounded mb-2">
                      <i class="fa fa-tag me-2 text-muted"/>
                      My Courses
                      <span t-att-data-post="slide_query_url(tag=slugify_tags(search_tags.ids), search=search_term, prevent_redirect=True)"
                         class="post_link cursor-pointer btn border-0 py-1">&#215;</span>
                    </span>
                </t>
                <t t-if="search_term">
                      <span class="align-items-baseline border d-inline-flex ps-2 rounded mb-2">
                      <i class="fa fa-tag me-2 text-muted"/>
                      <t t-esc="search_term"/>
                      <span t-att-data-post="slide_query_url(tag=slugify_tags(search_tags.ids), my=search_my, slide_category=search_slide_category, prevent_redirect=True)"
                         class="post_link cursor-pointer btn border-0 py-1">&#215;</span>
                    </span>
                </t>
                <t t-foreach="search_tags" t-as="tag">
                    <span class="align-items-baseline border d-inline-flex ps-2 rounded mb-2">
                        <i class="fa fa-tag me-2 text-muted"/>
                        <t t-esc="tag.display_name"/>
                        <span t-att-data-post='slide_query_url(tag=slugify_tags(search_tags.ids, tag.id), my=search_my, search=search_term, slide_category=search_slide_category, prevent_redirect=True)'
                            class="post_link cursor-pointer btn border-0 py-1">&#215;</span>
                    </span>
                </t>
            </div>
            <div class="container o_wslides_home_main pb-5">
                <div t-if="not channels and not search_term and not search_slide_category and not search_my and not search_tags">
                    <p class="h2">No Course created yet.</p>
                    <p groups="website_slides.group_website_slides_officer">Click on "New" in the top-right corner to write your first course.</p>
                </div>
                <div t-elif="search_term and not channels" class="alert alert-info mb-5">
                    No course was found matching your search <code><t t-esc="search_term"/></code>.
                </div>
                <div t-elif="not channels" class="alert alert-info mb-5">
                    No course was found matching your search.
                </div>
                <t t-else="">
                    <div t-if="original_search" class="alert alert-warning mb-5">
                        No results found for '<span t-esc="original_search"/>'. Showing results for '<span t-esc="search_term"/>'.
                    </div>
                    <div class="row mx-n2">
                        <t t-foreach="channels" t-as="channel">
                            <div class="col-12 col-sm-6 col-md-4 col-lg-3 px-2 d-flex">
                                <t t-call="website_slides.course_card"/>
                            </div>
                        </t>
                    </div>
                </t>
            </div>

            <t t-call="website_slides.courses_footer"></t>
        </div>
    </t>
</template>

<template id='courses_footer'>
    <section class="s_banner">
        <div class="oe_structure oe_empty" id="oe_structure_website_slides_course_footer_1"/>
    </section>
</template>

<template id='course_card' name="Course Card">
    <div t-attf-class="card w-100 o_wslides_course_card mb-4 #{'o_wslides_course_unpublished' if not channel.is_published else ''}" t-cache="channel if is_public_user and not search_tags else None">
        <t t-set="channel_frontend_tags" t-value="channel.tag_ids.filtered(lambda tag: tag.color)"/>
        <a t-attf-href="/slides/#{slug(channel)}" t-title="channel.name" style="height:120px">
            <div t-field="channel.image_1920" t-options="{'widget': 'image', 'preview_image': 'image_512'}" class="o_wslides_background_image h-100">
                <t t-if="channel.partner_has_new_content" t-call="website_slides.course_card_information"/>
            </div>
        </a>
        <div class="card-body p-3">
            <a class="card-title h5 mb-2 o_wslides_desc_truncate_2" t-attf-href="/slides/#{slug(channel)}" t-field="channel.name"/>
            <span t-if="not channel.is_published" class="badge text-bg-danger">Unpublished</span>
            <div class="card-text d-flex flex-column flex-grow-1 mt-1">
                <div class="fw-light o_wslides_desc_truncate_3" t-field="channel.description_short"/>
                <div t-if="channel_frontend_tags" class="mt-auto pt-1 o_wslides_desc_truncate_2_badges">
                    <t t-foreach="channel_frontend_tags" t-as="tag">
                        <t t-if="search_tags">
                            <span t-att-data-post="slide_query_url(tag=slugify_tags(search_tags.ids, toggle_tag_id=tag.id), my=search_my, search=search_term, slide_category=search_slide_category, prevent_redirect=True)"
                                t-attf-class="post_link cursor-pointer badge o_badge_clickable #{'o_color_'+str(tag.color) if tag in search_tags else 'o_wslides_channel_tag o_color_0'}" t-esc="tag.name"/>
                        </t>
                        <t t-else="">
                            <span t-att-data-post="slide_query_url(tag=slugify_tags(search_tags.ids, toggle_tag_id=tag.id), my=search_my, search=search_term, slide_category=search_slide_category, prevent_redirect=True)"
                                t-attf-class="post_link cursor-pointer badge o_badge_clickable o_wslides_channel_tag #{'o_color_'+str(tag.color)}" t-esc="tag.name"/>
                        </t>
                    </t>
                </div>
            </div>
        </div>
        <div class="card-footer bg-white text-600 px-3">
            <div class="d-flex justify-content-between align-items-center">
                <small t-if="channel.total_time" class="fw-bold" t-esc="channel.total_time" t-options="{'widget': 'duration', 'unit': 'hour', 'round': 'minute'}"/>
                <div class="d-flex flex-grow-1 justify-content-end">
                    <t t-if="channel.is_member and channel.completed">
                        <span class="badge text-bg-success pull-right"><i class="fa fa-check"/> Completed</span>
                    </t>
                    <div t-elif="channel.is_member and channel.channel_type != 'documentation'" class="progress w-50" style="height: 6px">
                        <div class="progress-bar" role="progressbar" t-att-aria-valuenow="channel.completion" aria-valuemin="0" aria-valuemax="100" t-attf-style="width:#{channel.completion}%;" aria-label="Progress bar"/>
                    </div>
                    <small t-else=""><b t-esc="channel.total_slides"/> steps</small>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="course_card_information" name='Course Information'>
    <t id="course_card_information_content">
    </t>
</template>

<template id="course_card_information_arrow" inherit_id="website_slides.course_card_information"
    active="True" name='New Content Ribbon'>
    <xpath expr="//t[@id='course_card_information_content']" position="inside">
        <span class="o_wslides_arrow">New Content</span>
    </xpath>
</template>

<template id='slides_home_achievements_small' name="Users">
    <t class="o_wslides_home_aside">
    </t>
</template>

<template id="toggle_latest_achievements" inherit_id="website_slides.slides_home_achievements_small" active="True" name='Display Achievements'>
    <xpath expr="//t[hasclass('o_wslides_home_aside')]" position="inside">
        <div t-if="achievements">
            <div class="row o_wslides_home_aside_title">
                <div class="col">
                    <h5 class="m-0">Latest achievements</h5>
                    <hr class="mt-2"/>
                </div>
            </div>
            <div class="row">
                <div class="col">
                    <t t-foreach="achievements" t-as="achievement">
                        <t t-call="website_slides.achievement_card"/>
                    </t>
                </div>
            </div>
        </div>
    </xpath>
</template>

<template id='achievement_card' name="Achivement Card">
    <div class="d-flex g-0 mt8 align-items-center">
        <t t-call="website_slides.slides_misc_user_image">
            <t t-set="user" t-value="achievement.user_id"/>
        </t>
        <div style="line-height: 1.3">
            <span class="fw-bold" t-esc="achievement.user_id.name"/> achieved <span class="fw-bold" t-esc="achievement.badge_id.name"/>
        </div>
    </div>
</template>

<template id='slides_home_users_small' name="Users">
    <div class="o_wslides_home_aside">
    </div>
</template>

<template id="toggle_leaderboard" inherit_id="website_slides.slides_home_users_small" active="True" name='Display Leaderboard'>
    <xpath expr="//div[hasclass('o_wslides_home_aside')]" position="inside">
        <div class="row o_wslides_home_aside_title">
            <div class="col">
                <a t-if="users" href="/profile/users" class="float-end">View all</a>
                <h5 class="m-0">Leaderboard</h5>
                <hr class="mt-2 mb-2"/>
            </div>
        </div>
        <div class="row">
            <t t-if="users">
                <div class="col">
                    <t t-set="counter" t-value="1"/>
                    <t t-foreach="users" t-as="user">
                        <t t-call="website_slides.user_quickkarma_card"/>
                        <t t-set="counter" t-value="counter + 1"/>
                    </t>
                </div>
            </t>
            <t t-else=""><p class="col mt8">No leaderboard currently :(</p></t>
        </div>
    </xpath>
</template>

<template id='user_quickkarma_card' name="User QuickKarma Card">
    <div class="d-flex mb-3 align-items-center">
        <b class="me-2 text-muted" t-esc="counter"/>
        <t t-call="website_slides.slides_misc_user_image"/>
        <div style="line-height:1.3">
            <span class="fw-bold" t-esc="user.name"/>
            <div class="d-flex align-items-center">
                <t t-esc="user.rank_id.name"/>
                <span class="text-500 mx-2">&#8226;</span>
                <span class="badge text-bg-success"><t t-esc="user.karma"/> xp</span>
            </div>
        </div>
    </div>
</template>

<template id='slides_home_user_profile_small' name="User Profile">
    <div class="o_wslides_home_aside">
        <div t-if="user.rank_id" class="d-flex align-items-center">
            <span class="fw-bold text-muted me-2">Current rank:</span>
            <img t-att-src="website.image_url(user.rank_id, 'image_128')" width="16" height="16" alt="" class="o_object_fit_cover me-1"/>
            <a href="/profile/ranks_badges" t-field="user.rank_id"/>
        </div>
        <t t-set="next_rank_id" t-value="user._get_next_rank()"/>
        <div t-if="next_rank_id" class="fw-bold text-muted mt-1">Next rank:</div>
        <t t-if="next_rank_id or user.rank_id" t-call="website_profile.profile_next_rank_card">
            <t t-set="bg_class" t-valuef="bg-200"/>
            <t t-set="img_max_width" t-value="'50%'"/>
        </t>
        <div t-if="next_rank_id" t-field="next_rank_id.description_motivational"/>
        <div t-else="">Congratulations, you have reached the last rank!</div>
    </div>
</template>

<template id='slides_home_user_achievements_small' name="User Achievements">
    <div class="o_wslides_home_aside flex-grow-1">
        <div class="row o_wslides_home_aside_title">
            <div class="col">
                <a href="/profile/ranks_badges?badge_category=slides" class="float-end">View all</a>
                <h5 class="m-0">Badges</h5>
                <hr class="mt-2 mt-2"/>
            </div>
        </div>
        <t t-foreach="challenges" t-as="challenge">
            <t t-set="challenge_done" t-value="challenge in challenges_done if challenges_done else False"/>
            <div t-attf-class="d-flex mb-3 align-items-center #{'o_wslides_entry_muted' if not challenge_done else ''}">
                <div t-if="challenge.reward_id.image_1920" t-field="challenge.reward_id.image_1920"
                    t-options="{'widget': 'image', 'preview_image': 'image_128', 'class': 'me-2', 'style': 'max-height: 36px'}"/>
                <img t-else="" t-attf-src="'/website_profile/static/src/img/badge_%s.svg' % (challenge.reward_id.level)" t-att-alt="challenge.reward_id.name" style="max-height: 36px" class="me-2"/>
                <div class="flex-grow-1">
                    <b class="text_small_caps" t-esc="challenge.reward_id.name"/><br/>
                    <span class="text-muted" t-esc="challenge.reward_id.description"/>
                </div>
                <i t-if="challenge_done" class="fa fa-check h5 text-success" aria-label="Done" title="Done" role="img"></i>
            </div>
        </t>
    </div>
</template>

<template id='slides_misc_user_image' name="User Avatar">
    <img t-attf-class="o_avatar {{img_class or 'rounded-circle float-start'}}"
        t-att-style="img_style if img_style else 'width: 32px; height: 32px;'"
        t-att-src="'/profile/avatar/%s?field=avatar_128' % user.id"
        t-att-alt="user.name"/>
</template>
</data></odoo>

```

## File: views\website_slides_templates_lesson.xml

```xml
<?xml version="1.0" ?>
<odoo><data>

<!-- Slide: main template: detailed view -->
<template id="slide_main" name="Slide Detailed View" track="1">
    <t t-set="body_classname" t-value="'o_wslides_body'"/>
    <t t-call="website.layout">
        <div id="wrap" class="wrap o_wslides_wrap">
            <t t-call="website.record_cover">
                <t t-set="_record" t-value="slide.channel_id"/>
                <t t-set="use_filters" t-value="True"/>
                <t t-set="use_size" t-value="True"/>
                <t t-set="use_text_align" t-value="True"/>

                <div class="o_wslides_lesson_header position-relative pb-0 pt-2 pt-md-5">
                    <t t-call="website_slides.course_nav">
                        <t t-set="channel" t-value="slide.channel_id"/>
                    </t>
                    <div class="container o_wslides_lesson_header_container mt-5 mt-md-3 mt-xl-4">
                        <div class="row align-items-md-stretch">
                            <div t-attf-class="col-12 col-sm-9 d-flex flex-column #{'col-lg-6 offset-lg-3' if slide.channel_id.channel_type == 'training' else ''}">
                                <h2 class="fw-medium w-100 text-truncate overflow-hidden">
                                    <a t-att-href="'/slides/%s' % (slug(slide.channel_id))" class="text-white text-decoration-none" t-field="slide.channel_id.name"/>
                                </h2>
                                <div t-if="slide.channel_id.channel_type == 'documentation'" class="mb-3 small">
                                    <span class="fw-normal">Last update:</span>
                                    <t t-esc="slide.date_published" t-options="{'widget': 'date'}"/>
                                </div>
                                <div t-else="" t-attf-class="o_wslides_channel_completion_progressbar #{'d-none' if slide.channel_id.completed else 'd-flex'} align-items-center pb-3">
                                    <div class="progress w-50 bg-black-25" style="height: 10px;">
                                        <div class="progress-bar rounded-start bg-info" role="progressbar" aria-label="Progress bar"
                                            t-att-aria-valuenow="slide.channel_id.completion" aria-valuemin="0" aria-valuemax="100"
                                            t-attf-style="width: #{slide.channel_id.completion}%;">
                                        </div>
                                    </div>
                                    <i class="fa fa-trophy m-0 ms-2 p-0 text-black-50"></i>
                                    <small class="ms-2 text-white-50"><span class="o_wslides_progress_percentage" t-esc="slide.channel_id.completion"/> %</small>
                                </div>
                            </div>
                            <div t-attf-class="o_wslides_channel_completion_completed col-12 col-sm-3 #{'d-none' if not slide.channel_id.completed else ''}">
                                <h2>
                                    <small><span class="badge text-bg-success fw-normal"><i class="fa fa-check"/> Completed</span></small>
                                </h2>
                            </div>
                        </div>
                    </div>
                </div>
            </t>
            <div class="container o_wslides_lesson_main">
                <div class="row">
                    <t t-set="can_access_channel" t-value="slide.channel_id.is_member or slide.channel_id.can_publish"/>
                    <div t-attf-class="o_wslides_lesson_aside col-lg-3 #{'order-2' if slide.channel_id.channel_type == 'documentation' else ''}">
                        <t t-if="slide.channel_id.channel_type == 'training'" t-call="website_slides.slide_aside_training"/>
                        <t t-if="slide.channel_id.channel_type == 'documentation'" t-call="website_slides.slide_aside_documentation"/>
                    </div>
                    <div t-attf-class="o_wslides_lesson_content col-lg-9 #{'order-1' if slide.channel_id.channel_type == 'documentation' else ''}">
                        <t t-call="website_slides.slide_content_detailed"/>
                    </div>
                </div>
            </div>
        </div>
    </t>
</template>

<!-- Slide: sidebar documentation mode -->
<template id="slide_aside_documentation" name="Slide: Sidebar in Documentation">
    <div class="o_wslides_lesson_aside_doc position-relative bg-white pb-1 my-3 border-bottom">
        <ul class="nav nav-tabs nav-fill" role="tablist">
            <li class="nav-item" role="presentation"><a aria-controls="related" href="#related" class="nav-link rounded-0 border-top-0 border-start-0 py-2 active" data-bs-toggle="tab" role="tab">Related</a></li>
            <li class="nav-item" role="presentation"><a aria-controls="most_viewed" href="#most_viewed" class="nav-link rounded-0 border-top-0 border-end-0 py-2" data-bs-toggle="tab" role="tab">Most Viewed</a></li>
        </ul>
        <div class="tab-content">
            <div role="tabpanel" id="related" class="tab-pane active bg-100">
                <ul class="list-group list-group-flush">
                    <t t-set="related_slides_list" t-value="list(related_slides)"/>
                    <t t-if="not related_slides_list">
                        No presentation available.
                    </t>
                    <t t-else="" t-foreach="related_slides_list" t-as="aside_slide">
                        <t t-call="website_slides.slide_aside_card"/>
                    </t>
                </ul>
            </div>
            <div role="tabpanel" id="most_viewed" class="tab-pane bg-100">
                <ul class="list-group list-group-flush">
                    <t t-set="most_viewed_slides_list" t-value="list(most_viewed_slides)"/>
                    <t t-if="not list(most_viewed_slides_list)">
                        No presentation available.
                    </t>
                    <t t-else="" t-foreach="most_viewed_slides_list" t-as="aside_slide">
                        <t t-call="website_slides.slide_aside_card"/>
                    </t>
                </ul>
            </div>
        </div>
    </div>
</template>

<!-- Slide sub-template: display an item in a list of related slides (Related, Most Viewed, ...) -->
<template id="slide_aside_card" name="Related Slide">
    <a class="list-group-item list-group-item-action d-flex align-items-start px-2" t-att-href="'/slides/slide/%s' % (slug(aside_slide))">
        <div class="me-1 border o_wslides_background_image_aside_card" t-attf-style="background-image: url(#{website.image_url(aside_slide, 'image_256')});"/>
        <div class="overflow-hidden d-flex flex-column justify-content-start">
            <h6 t-esc="aside_slide.name" class="o_wslides_desc_truncate_2 mb-1" style="line-height: 1.15"/>
            <small class="text-600">
                <t t-esc="aside_slide.total_views"/> Views &#8226; <timeago class="timeago" t-att-datetime="aside_slide.create_date"></timeago>
            </small>
        </div>
    </a>
</template>

<!-- Slide: sidebar training mode -->
<template id="slide_aside_training" name="Slide: Sidebar in Training">
    <div class="o_wslides_lesson_aside_list position-relative bg-white border-bottom mt-4">
        <div class="bg-100 text-1000 h6 my-0 text-decoration-none border-bottom d-flex align-items-center justify-content-between">
            <span class="p-2">Course content</span>
            <a href="#collapse_slide_aside" data-bs-toggle="collapse" class="d-lg-none p-2 text-decoration-none o_wslides_lesson_aside_collapse">
                <i class="oi oi-chevron-down d-lg-none"/>
            </a>
        </div>
        <ul id="collapse_slide_aside" class="list-unstyled my-0 pb-3 collapse d-lg-block">
            <t t-set="i" t-value="0"/>
            <t t-if="category.get('slides')" t-foreach="category_data" t-as="category">
                <t t-call="website_slides.slide_aside_training_category">
                    <t t-set="category_slide_ids" t-value="category['slides']"/>
                </t>
            </t>
        </ul>
    </div>
</template>

<template id="slide_aside_training_category" name="Category item for the slide detailed view list">
    <t t-set="category_collapsed" t-value="category and category.get('is_collapsed')"/>
    <t t-if="category" t-set="category" t-value="category.get('category')"/>
    <li class="o_wslides_fs_sidebar_section mt-2">
        <a t-attf-href="#collapse-#{category.id if category else 0}" t-attf-id="category-collapse-#{category.id if category else 0}"
            data-bs-toggle="collapse" role="button" t-att-aria-expanded="'true' if category_collapsed else 'false'"
            class="o_wslides_lesson_aside_list_link ps-2 text-600 text-uppercase text-decoration-none p-1 small d-flex"
            t-att-aria-controls="('collapse-%s') % (category.id if category else 0)">
            <t t-if="category">
                <b t-field="category.name"/>
            </t>
            <t t-else="">
                <b>Uncategorized</b>
            </t>
            <div class="flex-grow-1"/>
            <i class="fa fa-fw fa-caret-left" role="img"/>
            <i class="fa fa-fw fa-caret-down" role="img"/>
        </a>
        <ul t-attf-class="collapse #{'show' if category_collapsed else ''} p-0 m-0 list-unstyled" t-att-id="('collapse-%s') % (category.id if category else 0)" >
            <t t-set="is_member" t-value="slide.channel_id.is_member"/>
            <t t-set="can_access_channel" t-value="is_member or slide.channel_id.can_publish"/>
            <t t-foreach="category_slide_ids" t-as="aside_slide">
                <t t-set="slide_completed" t-value="channel_progress[aside_slide.id].get('completed')"/>
                <t t-set="can_access" t-value="aside_slide.is_preview or can_access_channel"/>
                <li class="p-0 pb-1">
                    <div t-att-class="'o_wslides_lesson_aside_list_link d-flex p-1 %s%s' % (('bg-100 active' if aside_slide == slide else ''), 'text-muted' if not can_access else '')"
                        t-att-data-id="slide.id"
                        t-att-data-completed="slide_completed">
                        <t t-call="website_slides.slide_sidebar_done_button">
                            <t t-set="slide" t-value="aside_slide"/>
                            <t t-set="slide_completed" t-value="channel_progress[aside_slide.id].get('completed')"/>
                            <t t-set="use_slide_icon" t-value="True"/>
                        </t>
                        <a t-att-href="'/slides/slide/%s' % (slug(aside_slide)) if can_access else '#'"
                            t-attf-class="d-flex text-decoration-none mw-100 overflow-hidden #{'text-muted' if not can_access else ''}">
                            <div class="o_wslides_lesson_link_name text-truncate" t-att-title="aside_slide.name">
                                <span t-esc="aside_slide.name"/>
                                <span class="align-items-end" t-if="aside_slide.question_ids">
                                    <span t-att-class="'badge rounded-pill %s' % ('text-bg-success' if channel_progress[aside_slide.id].get('completed') else 'text-bg-info')">
                                        <t t-esc="channel_progress[aside_slide.id].get('quiz_karma_won') if channel_progress[aside_slide.id].get('completed') else channel_progress[aside_slide.id].get('quiz_karma_gain')"/> xp
                                    </span>
                                </span>
                            </div>
                        </a>
                    </div>
                    <ul t-if="aside_slide.sudo().slide_resource_ids or aside_slide.question_ids" class="o_wslides_lesson_aside_list_links list-group mb-1 list-unstyled fw-light">
                        <t t-if="can_access_channel" t-foreach="aside_slide.slide_resource_ids" t-as="resource">
                           <li class="ps-3">
                                <a t-if="resource.resource_type == 'url'" t-att-href="resource.link" target="new" class="text-decoration-none small">
                                    <i class="fa fa-link"/><span t-field="resource.name"/>
                                </a>
                                <a t-else="" t-att-href="resource.download_url" class="text-decoration-none small">
                                    <i class="fa fa-download"/><span t-field="resource.name"/>
                                </a>
                            </li>
                        </t>
                        <div t-else="" class="o_wslides_js_course_join o_wslides_no_access ps-3">
                            <li t-if="aside_slide.channel_id.enroll == 'public' or (aside_slide.channel_id.enroll == 'invite' and aside_slide.channel_id.is_member_invited)"
                                class="text-decoration-none small">
                                <i class="fa fa-download"/>
                                <t t-call="website_slides.join_course_link">
                                    <t t-set="for_resources" t-value="1"/>
                                </t>
                            </li>
                        </div>
                        <li t-if="aside_slide.question_ids and aside_slide.slide_category != 'quiz'" class="ps-3">
                            <a t-if="can_access" t-att-href="'/slides/slide/%s#lessonQuiz' % (slug(aside_slide))"
                                class="o_wslides_lesson_aside_list_link text-decoration-none small text-600">
                                <i class="fa fa-flag text-warning"/> Quiz
                            </a>
                            <span t-else="" class="o_wslides_lesson_aside_list_link text-decoration-none small text-600 text-muted">
                                <i class="fa fa-flag text-warning"/> Quiz
                            </span>
                        </li>
                    </ul>
                </li>
            </t>
        </ul>
    </li>
</template>

<!-- Slide: all its content, not fullscreen mode -->
<template id="slide_content_detailed" name="Slide: Detailed Content">
    <div class="d-flex flex-wrap align-items-start my-3 w-100">
        <t t-set="slide_completed" t-value="channel_progress[slide.id].get('completed')"/>
        <div class="col-12 col-md order-2 order-md-1 d-flex">
            <div class="d-flex align-items-start overflow-hidden">
                <h1 class="h4 my-0 d-flex flex_row overflow-hidden">
                    <i t-attf-class="fa #{slide.slide_icon_class} me-2"/>
                    <span class="text-truncate" t-field="slide.name"/>
                </h1>
            </div>
        </div>
        <div class="col-12 col-md order-1 order-md-2 text-nowrap flex-grow-0 d-flex flex-wrap flex-md-nowrap justify-content-center justify-content-md-end align-items-center mb-3 mb-md-0">
            <t t-set="quiz_karma_won" t-value="channel_progress[slide.id].get('quiz_karma_won', 0)"/>
            <t t-set="quiz_karma_gain" t-value="channel_progress[slide.id].get('quiz_karma_gain', 0)"/>
            <span t-if="slide.question_ids and (slide_completed or quiz_karma_gain)" style="flex-basis: 100%"
                t-attf-class="mx-2 my-1 badge #{'text-bg-success' if slide_completed else 'text-bg-info'}">
                <span t-if="slide_completed">
                    <i class="fa fa-check-circle"/>
                    <t t-if="quiz_karma_won">
                        <t t-esc="quiz_karma_won" />
                        <span>XP</span>
                    </t>
                </span>
                <t t-else="">
                    <span t-esc="quiz_karma_gain"/>
                    <span>XP</span>
                </t>
            </span>
            <div class="btn-group flex-grow-1 flex-sm-0 my-1" role="group" aria-label="Lesson Nav">
                <a t-attf-class="o_wslides_nav_button btn btn-light border my-auto #{'disabled' if not previous_slide else ''} me-2"
                    role="button" t-att-aria-disabled="'true' if not previous_slide else None" aria-label="Previous"
                    t-att-href="'/slides/slide/%s' % (slug(previous_slide)) if previous_slide else '#'">
                    <i class="oi oi-chevron-left me-2"></i> <span class="d-none d-sm-inline-block">Prev</span>
                </a>
                <t t-if="slide.channel_id.channel_type == 'documentation' and slide.channel_id.is_member">
                    <t t-set="is_quiz" t-value="slide.slide_category == 'quiz' or slide.question_ids"/>
                    <a t-if="slide_completed and slide.can_self_mark_uncompleted" role="button"
                        class="o_wslides_undone_button btn btn-light border me-2"
                        t-attf-href="/slides/slide/#{slide.id}/set_uncompleted">
                        Mark To Do
                    </a>
                    <a t-elif="not slide_completed and is_quiz" role="button"
                        class="o_wslides_done_button btn btn-primary border text-white me-2"
                        href="#quiz_container">
                        Take Quiz
                    </a>
                    <a t-else="not slide_completed and slide.can_self_mark_completed" role="button"
                        class="o_wslides_done_button btn btn-primary border text-white me-2"
                        t-attf-href="/slides/slide/#{slide.id}/set_completed?next_slide_id=#{next_slide.id if next_slide else ''}">
                        Mark Done
                    </a>
                </t>
                <div t-if="slide.channel_id.channel_type == 'documentation' and not slide.channel_id.is_member" class="me-2">
                    <t t-call="website_slides.course_join">
                        <t t-set="channel" t-value="slide.channel_id"/>
                    </t>
                </div>
                <a t-attf-class="o_wslides_nav_button btn btn-light border my-auto #{'disabled' if not next_slide else ''}"
                    role="button" t-att-aria-disabled="'true' if not next_slide else None" aria-label="Next"
                    t-att-href="'/slides/slide/%s' % (slug(next_slide)) if next_slide else '#'">
                    <span class="d-none d-sm-inline-block">Next</span> <i class="oi oi-chevron-right ms-2"></i>
                </a>
            </div>
            <a class="btn btn-light border ms-2 my-1" role="button" t-att-href="'/slides/slide/%s?fullscreen=1' % (slug(slide))" aria-label="Fullscreen">
                <i class="fa fa-desktop me-xl-2 my-1"/>
                <span class="d-none d-xl-inline-block">Fullscreen</span>
            </a>
            <a class="o_wslides_share btn btn-light border ms-2 my-1" role="button" t-att-data-name="slide.name"
               t-att-data-id="slide.id" t-att-data-url="slide.website_share_url" t-att-data-category="slide.slide_category"
               t-att-data-email-sharing="bool(slide.channel_id.share_slide_template_id)"
               t-att-data-embed-code="slide.embed_code_external if slide.slide_category in ['video', 'document'] else False"
               aria-label="Share">
                <i class="fa fa-share-alt me-xl-2 my-1"/>
                <span class="d-none d-xl-inline-block">Share</span>
            </a>
        </div>
    </div>
    <div t-if="slide.tag_ids" class="pb-2">
        <t t-foreach="slide.tag_ids" t-as="tag">
            <a t-att-href="'/slides/%s/tag/%s' % (slug(slide.channel_id), slug(tag))" class="badge text-bg-info" t-esc="tag.name"/>
        </t>
    </div>
    <t t-set="editor_message">BUILDING BLOCKS DROPPED HERE WILL BE SHOWN ACROSS ALL LESSONS</t>
    <div class="oe_structure oe_empty" id="oe_structure_website_slides_lesson_top_1" t-att-data-editor-message="editor_message"/>
    <div t-if="slide.slide_category == 'infographic'" class="o_wslides_lesson_content_type" t-field='slide.image_1920' t-options="{'widget': 'image', 'style': 'width: 100%;'}"/>
    <div t-else="" class="o_wslides_lesson_content_type">
        <div t-if="slide.slide_category == 'document'" class="ratio ratio-4x3 embed-responsive-item mb8" style="height: 600px;">
            <t t-out="slide.embed_code"/>
        </div>
        <div t-if="slide.slide_category == 'video'" class="ratio ratio-16x9 embed-responsive-item mb8">
            <t t-out="slide.embed_code"/>
        </div>
        <div t-if="slide.slide_category == 'article'">
            <div t-if="is_html_empty(slide.html_content)" class="alert alert-info o_not_editable">
                Click on the "Edit" button in the top corner of the screen to edit your slide content.
            </div>
            <div class="bg-white p-3">
                <div t-field="slide.html_content" placeholder="Add your content here..."/>
            </div>
        </div>
    </div>

    <div class="mb-5 position-relative">
        <ul class="nav nav-tabs o_wslides_lesson_nav" role="tablist">
            <li class="nav-item" role="presentation">
                <a href="#about" aria-controls="about" t-att-class="'nav-link active' if not comments else 'nav-link'" role="tab" data-bs-toggle="tab">
                    <i class="fa fa-home"></i> About
                </a>
            </li>
            <li class="nav-item" role="presentation">
                <a href="#discuss" aria-controls="discuss" t-att-class="'nav-link active' if comments else 'nav-link'" role="tab" data-bs-toggle="tab">
                    <i class="fa fa-comments"></i> Comments (<span t-esc="slide.comments_count"/>)
                </a>
            </li>
            <li class="nav-item" role="presentation" groups="base.group_user">
                <a href="#statistic" aria-controls="statistic" class="nav-link" role="tab" data-bs-toggle="tab">
                    <i class="fa fa-bar-chart"></i> Statistics
                </a>
            </li>
        </ul>
        <div class="tab-content mt-3">
            <div role="tabpanel" t-att-class="not comments and 'tab-pane fade in show active' or 'tab-pane fade'" id="about">
                <div t-field="slide.description"/>
                <div t-if="slide.channel_id.allow_comment" class="d-flex">
                    <span class="text-muted fw-bold me-3">Rating</span>
                    <div class="text-muted border-start ps-3">
                        <div class="o_wslides_js_slide_like me-2">
                            <span t-attf-class="o_wslides_js_slide_like_up #{'disabled' if not slide.channel_id.can_vote else ''}" tabindex="0" data-bs-toggle="popover" t-att-data-slide-id="slide.id" t-att-data-user-vote="slide.user_vote">
                                <i t-attf-class="fa fa-1x #{'fa-thumbs-up' if slide.user_vote == 1 else 'fa-thumbs-o-up'}" role="img" aria-label="Likes" title="Like"/>
                                <span t-field="slide.likes" t-options="{'format_decimalized_number': True}"/>
                            </span>
                            <span t-attf-class="o_wslides_js_slide_like_down ms-3 #{'disabled' if not slide.channel_id.can_vote else ''}" tabindex="0" data-bs-toggle="popover" t-att-data-slide-id="slide.id" t-att-data-user-vote="slide.user_vote">
                                <i t-attf-class="fa fa-1x #{'fa-thumbs-down' if slide.user_vote == -1 else 'fa-thumbs-o-down'}" role="img" aria-label="Dislikes" title="Dislike"/>
                                <span t-field="slide.dislikes" t-options="{'format_decimalized_number': True}"/>
                            </span>
                        </div>
                    </div>
                </div>
            </div>
            <div role="tabpanel" t-att-class="comments and 'tab-pane fade in show active' or 'tab-pane fade'" id="discuss">
                <t t-set="enable_slide_comments" t-value="0"/>
                <p t-if="not (slide.channel_id.allow_comment and slide.channel_id.channel_type == 'training')">
                    Commenting is not enabled on this course.
                </p>
                <t t-elif="not slide.comments_count">
                    <p t-if="not can_access_channel">
                        There are no comments for now.
                        <t t-if="slide.channel_id.enroll != 'invite'">
                            <div class="o_wslides_js_course_join o_wslides_no_access_comments d-inline">
                                <t t-if="slide.channel_id.enroll == 'public'" t-call="website_slides.join_course_link"/>
                            </div>
                            to be the first to leave a comment.
                        </t>
                    </p>
                    <p t-elif="not slide.channel_id.can_comment">
                        There are no comments for now. Earn more Karma to be the first to leave a comment.
                    </p>
                    <t t-else="" t-set="enable_slide_comments" t-value="1"/>
                </t>
                <t t-else="">
                    <t t-set="enable_slide_comments" t-value="1"/>
                    <t t-if="not slide.channel_id.can_comment and can_access_channel"><p>Earn more Karma to leave a comment.</p></t>
                </t>
                <t t-if="enable_slide_comments" t-call="portal.message_thread">
                    <t t-set="object" t-value="slide"/>
                    <t t-set="disable_composer" t-value="not (slide.channel_id.can_comment and slide.channel_id.allow_comment)"/>
                    <t t-set="display_rating" t-value="False"/>
                </t>
            </div>
            <div role="tabpanel" class="tab-pane fade" groups="base.group_user" id="statistic" t-att-slide-url="slide.website_url">
                <div class="row">
                    <div class="col-md-6">
                        <table class="table table-sm">
                            <tbody>
                                <tr>
                                    <th colspan="2" class="border-top-0">Views</th>
                                </tr>
                                <tr class="border-top-0">
                                    <th class="border-top-0"><span t-esc="slide.total_views"/></th>
                                    <td class="border-top-0 w-100">Total Views</td>
                                </tr>
                                <tr>
                                    <th><span t-esc="slide.slide_views"/></th>
                                    <td>Members Views</td>
                                </tr>
                                <tr>
                                    <th><span t-esc="slide.public_views"/></th>
                                    <td>Public Views</td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div class="o_wslides_js_quiz_container" t-att-data-slide-id="slide.id" id="quiz_container">
        <div class="row" t-if="slide.slide_category != 'certification'">
            <t t-if="slide.question_ids">
                <t t-call="website_slides.lesson_content_quiz"/>
            </t>
            <div t-else="" class="o_wslides_js_lesson_quiz col" t-att-data-id="slide.id">
                <t t-if="slide.channel_id.can_upload" t-call="website_slides.lesson_content_quiz_add_buttons"/>
            </div>
        </div>
    </div>
    <div class="mt-3 mb-3">
        <t t-if="slide.sudo().slide_resource_ids">
            <t t-set="can_access_channel" t-value="slide.channel_id.is_member or slide.channel_id.can_publish"/>
            <t t-if="can_access_channel">
                <t t-set="links" t-value="slide.slide_resource_ids.filtered(lambda res: res.resource_type == 'url')"/>
                <div class="row mb-4 mt-4" t-if="links">
                    <span class="text-muted fw-bold col-4 col-md-3">External sources</span>
                    <div class="text-muted me-auto border-start ps-3 col-8 col-md-9">
                        <t t-foreach="links" t-as="link">
                            <a t-att-href="link.link" t-esc="link.name"/><br />
                        </t>
                    </div>
                </div>
                <t t-set="files" t-value="slide.slide_resource_ids.filtered(lambda res: res.resource_type == 'file' and res.data)"/>
                <div class="row mb-4 o_wslides_js_course_join" t-if="files">
                    <span class="text-muted fw-bold col-4 col-md-3">
                        Additional Resources
                    </span>
                    <div class="text-muted me-auto border-start ps-3 col-8 col-md-9">
                        <t t-foreach="slide.slide_resource_ids" t-as="resource" t-if="resource.resource_type == 'file' and resource.data">
                            <a t-att-href="resource.download_url" t-esc="resource.name"/><br />
                        </t>
                    </div>
                </div>
            </t>
            <t t-else="">
                <span t-if="slide.is_preview" class="text-muted fw-bold me-3">
                    Additional Resources
                </span>
                <div class="o_wslides_js_course_join o_wslides_no_access">
                    <div t-if="slide.channel_id.enroll == 'invite' and not slide.channel_id.is_member_invited">
                        <span>Content only accessible to course attendees.</span>
                    </div>
                    <div t-else="" class="text-muted me-auto border-start ps-3">
                        <t t-call="website_slides.join_course_link">
                            <t t-set="for_resources" t-value="1"/>
                        </t>
                    </div>
                </div>
            </t>
        </t>
    </div>
</template>

<!-- Slide sub-tempalte: render a quiz serverside. Should be sync with JS qweb template "slide.slide.quiz" -->
<template id="lesson_content_quiz" name="Lesson: Quiz specific content">
    <t t-set="slide_completed" t-value="channel_progress[slide.id].get('completed')"/>
    <div class="o_wslides_js_lesson_quiz col" id="lessonQuiz"
        t-att-data-id="slide.id"
        t-att-data-name="slide.name"
        t-att-data-slide-category="slide.slide_category"
        t-att-data-is-member="slide.channel_id.is_member"
        t-att-data-is-member-or-invited="slide.channel_id.is_member or slide.channel_id.is_member_invited"
        t-att-data-can-self-mark-completed="slide.can_self_mark_completed"
        t-att-data-can-self-mark-uncompleted="slide.can_self_mark_uncompleted"
        t-att-data-completed="slide_completed"
        t-att-data-quiz-attempts-count="quiz_attempts_count"
        t-att-data-quiz-karma-max="quiz_karma_max"
        t-att-data-quiz-karma-gain="quiz_karma_gain"
        t-att-data-quiz-karma-won="quiz_karma_won"
        t-att-data-has-next="1 if next_slide else 0"
        t-att-data-next-slide-url="'/slides/slide/%s' % (slug(next_slide)) if next_slide else None"
        t-att-data-channel-id="slide.channel_id.id"
        t-att-data-channel-enroll="slide.channel_id.enroll"
        t-att-data-channel-requested-access="slide.channel_id.has_requested_access"
        t-att-data-channel-can-upload="slide.channel_id.can_upload"
        t-att-data-signup-allowed="signup_allowed"
        t-att-data-session-answers="session_answers">
        <t t-foreach="slide_questions" t-as="question">
            <t t-call="website_slides.lesson_content_quiz_question"/>
        </t>
        <t t-if="slide.channel_id.can_upload" t-call="website_slides.lesson_content_quiz_add_buttons"/>
        <div class="o_wslides_js_lesson_quiz_validation pt-3"/>
    </div>
</template>

<template id="lesson_content_quiz_question" name="Lesson: Quiz question template">
    <div t-att-class="'o_wslides_js_lesson_quiz_question mt-3 %s' % ('completed-disabled' if slide_completed else ('disabled' if not (slide.channel_id.is_member or slide.is_preview) else ''))"
         t-att-data-question-id="question['id']" t-att-data-title="question['question']" >
        <div class="row d-flex mb-2 mx-0">
            <div class="h4">
                <small class="text-muted">
                    <i class="o_wslides_js_quiz_icon o_wslides_js_quiz_sequence_handler fa fa-bars me-1 text-muted" t-if="slide.channel_id.can_upload and not slide_completed" />
                    <t t-if="question_index != NoneType"><span class="o_wslides_quiz_question_sequence" t-esc="question_index+1"/>.</t>
                    <t t-else=""><span class="o_wslides_quiz_question_sequence" t-esc="question['sequence']"/>.</t>
                </small>
                <span t-esc="question['question']"/>
            </div>
            <div class="ms-auto o_wslides_js_quiz_edit_del" t-if="slide.channel_id.can_upload and not slide_completed" >
                <i class="o_wslides_js_quiz_icon o_wslides_js_quiz_edit_question fa fa-pencil-square-o p-1 text-muted"></i>
                <i class="o_wslides_js_quiz_icon o_wslides_js_quiz_delete_question fa fa-trash p-1 text-muted"></i>
            </div>
        </div>
        <div class="list-group">
            <t t-foreach="question['answer_ids']" t-as="answer">
                <a t-att-data-answer-id="answer['id']" href="#"
                    t-att-data-text="answer['text_value']" t-att-data-is-correct="answer['is_correct']" t-att-data-comment="answer['comment']"
                    t-att-class="'o_wslides_quiz_answer list-group-item list-group-item-action d-flex align-items-center %s' % ('list-group-item-success' if slide_completed and answer['is_correct'] else '')">
                    <label class="my-0 d-flex align-items-center justify-content-center me-2">
                        <input type="radio"
                            t-att-name="question['id']"
                            t-att-value="answer['id']"
                            class="d-none"
                            t-att-disabled="True if not slide.channel_id.is_member or slide_completed else ''"/>
                        <i t-att-class="'fa fa-circle text-400 %s' % ('d-none' if slide_completed and answer['is_correct'] else '')"/>
                        <i class="fa fa-times-circle text-danger d-none"></i>
                        <i t-att-class="'fa fa-check-circle text-success %s' % ('d-none' if not (slide_completed and answer['is_correct']) else '')"></i>
                    </label>
                    <span t-esc="answer['text_value']"/>
                </a>
                <t t-if="slide_completed and answer['is_correct']" t-set="correct_answer_comment" t-value="answer['comment']"/>
            </t>
            <div t-attf-class="o_wslides_quiz_answer_info list-group-item list-group-item-info #{'' if correct_answer_comment else 'd-none'}">
                <i class="fa fa-info-circle"/>
                <span class="o_wslides_quiz_answer_comment ms-1">
                    <t t-if="correct_answer_comment" t-out="correct_answer_comment"/>
                </span>
            </div>
        </div>
    </div>
</template>

<template id="lesson_content_quiz_add_buttons" name="Lesson: Quiz Add Buttons template">
    <div class="o_wslides_js_lesson_quiz_new_question mt-3">
        <a t-attf-class="o_wslides_js_quiz_add o_wslides_js_quiz_add_quiz btn btn-light border #{'d-none ' if slide.question_ids else ''}" role="button">
            <i class="fa fa-plus me-2"/>
            <span>Add Quiz</span>
        </a>
        <a t-attf-class="o_wslides_js_quiz_add o_wslides_js_quiz_add_question btn btn-light border ms-3 #{'' if slide.question_ids else 'd-none '}" role="button">
            <i class="fa fa-plus me-2"/>
            <span>Add Question</span>
        </a>
    </div>
</template>

</data></odoo>

```

## File: views\website_slides_templates_lesson_embed.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <data>
        <!--
            This template is the PDF Viewer. It will mostly rendered throught an iFrame.
            The js file to bind event is slides_embed.js
        -->
        <template id="embed_slide" name="Embedded Slide Page">
            <html>
                <head>
                    <title><t t-esc="slide.name"/></title>
                    <t t-call-assets="website_slides.slide_embed_assets"/>
                    <script type="module" src="/web/static/lib/pdfjs/build/pdf.js"/>
                    <script type="module" src="/web/static/lib/pdfjs/build/pdf.worker.js"/>
                </head>
                <body>
                    <div id="PDFViewer" class="o_wslides_fs_pdf_viewer d-flex flex-column h-100">
                        <!-- PDF Viewer Header : contains the name, and the share links -->
                        <div t-if="is_external_embed" class="oe_slides_share_bar">
                            <div class="container-fluid">
                                <div class="row align-items-center">
                                    <div class="col">
                                        <div class="oe_slides_ellipsis">
                                            <a target="_new" t-att-href="slide.website_url">
                                                <span  t-esc="slide.name" t-att-title="slide.name"/>
                                            </a>
                                        </div>
                                    </div>
                                    <div class="col flex-grow-0 d-flex text-nowrap small">
                                        <a class="oe_slide_js_embed_option_link" href="#" data-bs-toggle="modal" t-attf-data-bs-target="#slideShareModal_{{slide.id}}">
                                            <i class="fa fa-share-alt" aria-label="Share" title="Share"/>
                                            Share
                                        </a>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <!--  PDF Viewer Body : contains the canvas, the loader, or the image-->
                        <div id="PDFSlideViewer" class="d-flex align-items-start position-relative flex-grow-1 overflow-auto" style="height: 0;"
                                t-attf-data-slideid="#{slide.id}"
                                t-attf-data-slideurl="/slides/slide/#{slug(slide)}/pdf_content"
                                t-attf-data-downloadable="#{False}"
                                t-att-data-defaultpage="page">
                            <t t-if="is_external_embed">
                                <t t-call="website_slides.slide_share_modal">
                                    <t t-set="record" t-value="slide"/>
                                    <t t-set="email_sharing" t-value="slide.channel_id.share_slide_template_id"/>
                                    <t t-set="website_share_url" t-value="slide.website_share_url"/>
                                    <t t-set="include_embed" t-value="True"/>
                                    <t t-set="embed_hide_starting_page" t-value="True"/>
                                </t>
                            </t>
                            <div id="slide_suggest" class="oe_slide_embed_option bg-300 container-fluid overflow-auto d-none">
                                <div class="row">
                                    <t t-foreach="related_slides" t-as="suggest_slide">
                                        <div class="col-6 col-md-4 col-lg-3 oe_slides_suggestion_media">
                                            <div class="card mb-3">
                                                <a t-att-href="suggest_slide.website_url" target="_new" class="card-img-top ratio ratio-16x9">
                                                    <img t-att-src="website.image_url(suggest_slide, 'image_1024')" class="card-img-top embed-responsive-item" t-att-alt="suggest_slide.name"/>
                                                </a>
                                                <div class="card-body">
                                                    <h6 class="card-title">
                                                        <a t-att-href="suggest_slide.website_url" target="_new">
                                                            <t t-esc="suggest_slide.name"/>
                                                        </a>
                                                    </h6>
                                                    <div class="oe_slides_suggestion_caption"/>
                                                </div>
                                            </div>
                                        </div>
                                    </t>
                                </div>
                            </div>
                            <t t-if="slide.slide_category == 'document'">
                                <div id="PDFViewerLoader" class="oe_slides_loader mt-3 mx-2 w-100">
                                    <div class="toast show mx-auto">
                                        <div class="toast-header">
                                            <i class="fa fa-circle-o-notch fa-spin me-2"/><b>Loading...</b>
                                        </div>
                                        <div class="toast-body p-0">
                                            <img class="img-fluid w-100" t-att-src="website.image_url(slide, 'image_256')"/>
                                        </div>
                                    </div>
                                </div>
                                <canvas id="PDFViewerCanvas" class="mx-auto" style="display: none;"></canvas>
                            </t>
                            <t t-if="slide.slide_category == 'infographic'">
                                <img t-att-src="website.image_url(slide, 'image_1024')" class="img-fluid" style="width: 100%" alt="Slide image"/>
                            </t>
                        </div>
                        <!-- Fixed bottom navbar -->
                        <div id="PDFViewerNav" class="pt-2 pb-2 bg-light text-white" role="navigation" t-if="slide.slide_category == 'document'">
                            <div class="container-fluid oe_slides_panel_footer">
                                <div class="row align-items-center">
                                    <div class="col-5 col-sm-3 d-flex align-items-center">
                                        <div class="input-group input-group-sm flex-nowrap" style="max-width: 100px">
                                            <input type="number" class="form-control text-center" id="page_number" style="min-width: 60px"/>
                                            <span class="input-group-text" id="page_count"/>
                                        </div>
                                        <span id="zoomout" class="d-inline ms-2 me-2" title="Zoom out" aria-label="Zoom out" role="button">
                                            <i class="fa fa-search-minus" />
                                        </span>
                                        <span id="zoomin" class="d-inline" title="Zoom in" aria-label="Zoom in" role="button">
                                            <i class="fa fa-search-plus" />
                                        </span>
                                    </div>
                                    <div class="col text-center o_slide_navigation_buttons">
                                        <span id="first" class="me-1 me-sm-2" title="First slide" aria-label="First slide" role="button"><i class="fa fa-step-backward"/></span>
                                        <span id="previous" class="mx-1 mx-sm-2" title="Previous slide" aria-label="Previous slide" role="button"><i class="fa fa-arrow-circle-left"/></span>
                                        <span id="next" class="mx-1 mx-sm-2" title="Next slide" aria-label="Next slide" role="button"><i class="fa fa-arrow-circle-right"/></span>
                                        <span id="last" class="mx-1 mx-sm-2" title="Last slide" aria-label="Last slide" role="button"><i class="fa fa-step-forward"/></span>
                                        <a t-if="slide.slide_resource_downloadable" id="download" t-attf-href="/web/content/slide.slide/#{slide.id}/binary_content?download=true"
                                           class="ms-1 ms-sm-2" title="Download Content" aria-label="Download" role="button">
                                            <i class="fa fa-download" />
                                        </a>
                                    </div>
                                    <div class="col-2 col-sm-3 text-end flex-grow-0">
                                        <span id="fullscreen" class="ms-1 ms-sm-2"
                                           title="View fullscreen" aria-label="Fullscreen" role="button">
                                            <i class="fa fa-arrows-alt"/>
                                        </span>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </body>
            </html>
        </template>

        <!--
            Template render isntead of Embedded Slide, it this one is forbidden.
            So, it will mostly be rendered throught an iFrame
        -->
        <template id="embed_slide_forbidden" name="Forbidden Embedded Slide">
            <html>
                <head>
                    <t t-call-assets="website_slides.slide_embed_assets" t-js="false"/>
                </head>
                <body>
                    <div class="slide-private-view">
                        <h3 style="border-bottom: 1px solid !important;padding-bottom: 10px;"><i class="fa fa-exclamation-triangle" role="img" aria-label="Attention" title="Attention"></i> This document is private.</h3>
                    </div>
                </body>
            </html>
        </template>
    </data>
</odoo>

```

## File: views\website_slides_templates_lesson_fullscreen.xml

```xml
<?xml version="1.0" ?>
<odoo><data>

<!-- Slide template for the fullscreen mode -->
<template id="slide_fullscreen" name="Fullscreen">
    <t t-set="head">
        <link rel="canonical" t-att-href="slide.website_url" />
    </t>
    <t t-call="website.layout">
        <div class="o_wslides_fs_main d-flex flex-column"
            t-att-data-channel-id="slide.channel_id.id"
            t-att-data-channel-enroll="slide.channel_id.enroll"
            t-att-data-signup-allowed="signup_allowed"
            t-att-data-session-answers="session_answers">

            <div class="o_wslides_slide_fs_header d-flex flex-shrink-0 text-white">
                <div class="d-flex">
                    <a class="o_wslides_fs_toggle_sidebar d-flex align-items-center px-3" href="#" title="Lessons">
                        <i class="fa fa-bars"/><span class="d-none d-md-inline-block ms-1">Lessons</span>
                    </a>
                    <a class="o_wslides_fs_review d-flex align-items-center" title="Reviews" t-if="slide.channel_id.allow_comment">
                        <t t-call="portal_rating.rating_stars_static_popup_composer">
                            <t t-set="rating_avg" t-value="rating_avg"/>
                            <t t-set="rating_total" t-value="rating_count"/>
                            <t t-set="object" t-value="channel"/>
                            <t t-set="token" t-value="channel.access_token"/>
                            <t t-set="hash" t-value="message_post_hash"/>
                            <t t-set="pid" t-value="message_post_pid"/>
                            <t t-set="default_message" t-value="last_message"/>
                            <t t-set="default_message_id" t-value="last_message_id"/>
                            <t t-set="default_rating_value" t-value="last_rating_value"/>
                            <t t-set="default_attachment_ids" t-value="last_message_attachment_ids"/>
                            <t t-set="force_submit_url" t-value="'/slides/mail/update_comment' if last_message_id else False"/>
                            <t t-set="disable_composer" t-value="not channel.can_review"/>
                            <t t-set="_link_btn_classes" t-value="'d-inline-block text-white fw-light shadow-none'"/>
                            <t t-set="icon" t-value="'fa fa-pencil'"/>
                            <t t-set="_text_classes" t-value="'d-none d-md-inline-block'"/>
                            <t t-set="hide_rating_avg" t-value="True"/>
                            <t t-set="is_fullscreen" t-value="True"/>
                        </t>
                    </a>
                </div>
                <div class="d-flex ms-auto">
                    <a class="o_wslides_fs_share d-flex align-items-center px-3" href="#" title="Share">
                        <i class="fa fa-share-alt"/>
                        <span class="d-none d-md-inline-block ms-2">Share</span>
                    </a>
                    <a class="d-flex align-items-center px-3 o_wslides_fs_exit_fullscreen" t-attf-href="/slides/slide/#{slug(slide)}" title="Exit Fullscreen">
                        <i class="fa fa-sign-out"/><span class="d-none d-md-inline-block ms-1">Exit Fullscreen</span>
                    </a>
                    <a class="d-flex align-items-center px-3" t-attf-href="/slides/#{slug(slide.channel_id)}" title="Back to course">
                        <i class="fa fa-home"/><span class="d-none d-md-inline-block ms-1">Back to course</span>
                    </a>
                </div>
            </div>

            <div class="o_wslides_fs_container d-flex position-relative overflow-hidden flex-grow-1">
                <div class="o_wslides_fs_content align-items-stretch justify-content-center d-flex flex-grow-1 order-2"></div>

                <div class="o_wslides_fs_sidebar o_wslides_fs_sidebar_hidden text-white flex-shrink-0 order-1">
                    <div class="o_wslides_fs_sidebar_content d-flex flex-column px-3 pt-3 h-100">
                        <div class="o_wslides_fs_sidebar_header mb-3">
                            <a class="h5 d-block mb-1" t-attf-href="/slides/#{slug(slide.channel_id)}">
                                <span t-field="slide.channel_id.name"/>
                            </a>
                            <div t-if="not is_public_user">
                                <span t-attf-class="o_wslides_channel_completion_completed badge text-bg-success #{'d-none' if not slide.channel_id.completed else ''}">
                                    <i class="fa fa-check"/> Completed
                                </span>
                                <div t-attf-class="o_wslides_channel_completion_progressbar #{'d-none' if slide.channel_id.completed else 'd-flex'} w-100 align-items-center">
                                    <div class="progress flex-grow-1 bg-black-50" style="height: 6px;">
                                        <div class="progress-bar" role="progressbar" t-attf-style="width: #{slide.channel_id.completion}%" t-att-aria-valuenow="slide.channel_id.completion" aria-valuemin="0" aria-valuemax="100" aria-label="Progress bar"></div>
                                    </div>
                                    <div class="ms-3 small">
                                        <span class="o_wslides_progress_percentage" t-esc="slide.channel_id.completion"/> %
                                    </div>
                                </div>
                            </div>
                        </div>
                        <ul class="mx-n3 list-unstyled my-0 pb-2 overflow-auto">
                            <t t-foreach="category_data" t-as="category">
                                <t t-if="category.get('slides')">
                                    <t t-call="website_slides.slide_fullscreen_sidebar_category">
                                        <t t-set="slides" t-value="category['slides']"/>
                                        <t t-set="current_slide" t-value="slide"/>
                                    </t>
                                </t>
                            </t>
                        </ul>
                    </div>
                    <a href="#" class="o_wslides_fs_toggle_sidebar d-md-none bg-black-50"/>
                </div>
            </div>
        </div>
    </t>
</template>


<template id="slide_fullscreen_sidebar_category" name="Slides category template for fullscreen view side bar">
    <t t-set="category_collapsed" t-value="category and category.get('is_collapsed')"/>
    <t t-if="category" t-set="category" t-value="category.get('category')"/>
    <li class="o_wslides_fs_sidebar_section py-2 px-3">
        <a t-if="category" class="text-uppercase text-500 py-1 small d-flex" t-attf-id="category-collapse-#{category.id if category else 0}"
            data-bs-toggle="collapse" role="button" t-att-aria-expanded="'true' if category_collapsed else 'false'"
            t-attf-href="#collapse-#{category.id if category else 0}" t-attf-aria-controls="collapse-#{category.id if category else 0}">
            <b t-field="category.name"/>
            <div class="flex-grow-1"/>
            <i class="fa fa-fw fa-caret-left" role="img"/>
            <i class="fa fa-fw fa-caret-down" role="img"/>
        </a>
        <ul t-attf-class="o_wslides_fs_sidebar_section_slides position-relative px-0 pb-1 my-0 mx-n3 collapse #{'show' if category_collapsed else ''}"
            t-attf-id="collapse-#{category.id if category else 0}">
            <t t-set="is_member" t-value="current_slide.channel_id.is_member"/>
            <t t-set="can_access_channel" t-value="is_member or current_slide.channel_id.can_publish"/>
            <t t-foreach="slides" t-as="slide">
                <t t-set="slide_completed" t-value="channel_progress[slide.id].get('completed')"/>
                <t t-set="use_slide_icon" t-value="True"/>
                <t t-set="can_access" t-value="can_access_channel or slide.is_preview"/>
                <t t-set="is_member" t-value="current_slide.channel_id.is_member"/>
                <t t-set="is_member_or_invited" t-value="is_member or current_slide.channel_id.is_member_invited"/>
                <li t-attf-class="o_wslides_fs_sidebar_list_item d-flex py-1 #{'active' if slide.id == current_slide.id else ''}"
                    t-att-data-id="slide.id"
                    t-att-data-can-access="can_access"
                    t-att-data-name="slide.name"
                    t-att-data-category="slide.slide_category"
                    t-att-data-video-source-type="slide.video_source_type"
                    t-att-data-slug="slug(slide)"
                    t-att-data-has-question="1 if slide.question_ids else 0"
                    t-att-data-is-quiz="0"
                    t-att-data-completed="slide_completed"
                    t-att-data-embed-code="slide.embed_code if slide.slide_category in ['video', 'document', 'infographic'] else False"
                    t-att-data-can-self-mark-completed="slide.can_self_mark_completed"
                    t-att-data-can-self-mark-uncompleted="slide.can_self_mark_uncompleted"
                    t-att-data-is-member="is_member"
                    t-att-data-is-member-or-invited="is_member_or_invited"
                    t-att-data-session-answers="session_answers"
                    t-att-data-website-share-url="slide.website_share_url"
                    t-att-data-email-sharing="bool(slide.channel_id.share_slide_template_id)">
                    <div class="ms-2 o_wslides_sidebar_content overflow-hidden">
                        <a t-if="can_access" class="d-block" href="#">
                            <div class="d-flex">
                                <t t-if="is_member" t-call="website_slides.slide_sidebar_done_button"/>
                                <i t-else="" t-attf-class="fa #{slide.slide_icon_class} me-2"/>
                                <div class="o_wslides_fs_slide_name text-truncate" t-esc="slide.name"/>
                            </div>
                        </a>
                        <span t-else="" class="d-block" href="#">
                            <div class="d-flex">
                                <t t-if="is_member" t-call="website_slides.slide_sidebar_done_button"/>
                                <i t-else="" t-attf-class="fa #{slide.slide_icon_class} me-2 text-600"/>
                                <div class="o_wslides_fs_slide_name text-600 text-truncate" t-esc="slide.name"/>
                            </div>
                        </span>
                        <ul class="list-unstyled w-100 small fw-light" t-if="slide.sudo().slide_resource_ids or (slide.question_ids and not slide.slide_category =='quiz')" >
                            <t t-if="can_access_channel" t-foreach="slide.slide_resource_ids" t-as="resource">
                                <li class="ps-1 mb-1">
                                    <a t-if="resource.resource_type == 'url'" class="o_wslides_fs_slide_link" t-att-href="resource.link" target="_blank">
                                        <i class="fa fa-link me-2"/><span t-esc="resource.name"/>
                                    </a>
                                    <a t-else="" class="o_wslides_fs_slide_link ps-0" t-att-href="resource.download_url">
                                        <i class="fa fa-download me-2"/><span t-esc="resource.name"/>
                                    </a>
                                </li>
                            </t>
                            <div t-else="" class="o_wslides_js_course_join o_wslides_no_access ps-0">
                                <li t-if="slide.channel_id.enroll == 'public' or (slide.channel_id.enroll == 'invite' and slide.channel_id.is_member_invited)"
                                    class="o_wslides_fs_slide_link mb-1">
                                    <i class="fa fa-download me-1"/>
                                    <t t-call="website_slides.join_course_link">
                                        <t t-set="for_resources" t-value="1"/>
                                    </t>
                                </li>
                            </div>
                            <li class="o_wslides_fs_sidebar_list_item ps-0 mb-1" t-if="slide.question_ids and not slide.slide_category == 'quiz'"
                                t-att-data-id="slide.id"
                                t-att-data-can-access="can_access"
                                t-att-data-video-source-type="slide.video_source_type"
                                t-att-data-name="slide.name"
                                t-att-data-category="slide.slide_category"
                                t-att-data-slug="slug(slide)"
                                t-att-data-has-question="1 if slide.question_ids else 0"
                                t-att-data-is-quiz="1"
                                t-att-data-completed="slide_completed"
                                t-att-data-can-self-mark-completed="slide.can_self_mark_completed"
                                t-att-data-can-self-mark-uncompleted="slide.can_self_mark_uncompleted"
                                t-att-data-is-member="is_member"
                                t-att-data-is-member-or-invited="is_member_or_invited"
                                t-att-data-session-answers="session_answers"
                                t-att-data-website-share-url="slide.website_share_url">
                                <a t-if="can_access" class="o_wslides_fs_slide_quiz o_wslides_fs_slide_name" href="#" t-att-index="i">
                                    <i class="fa fa-flag-checkered text-warning"/>Quiz
                                </a>
                                <span t-else="" class="text-600">
                                    <i class="fa fa-flag-checkered text-warning"/>Quiz
                                </span>
                            </li>
                        </ul>
                    </div>
                </li>
            </t>
        </ul>
    </li>
</template>


</data></odoo>

```

## File: views\website_slides_templates_profile.xml

```xml
<?xml version="1.0" ?>
<odoo><data>
    <!--Access Denied - Profile Page-->
    <template id="profile_access_denied" inherit_id="website_profile.profile_access_denied">
        <xpath expr="//div[@id='profile_access_denied_return_link_container']" position="inside">
            <t t-if="request.params.get('channel_id')">
                <p><a t-attf-href="/slides/course-#{request.params.get('channel_id')}">Return to the course.</a></p>
            </t>
        </xpath>
    </template>

    <template id="user_profile_content" inherit_id="website_profile.user_profile_content">
        <xpath expr="//div[@id='profile_about_badge']" position="before">
            <t t-if="channel">
                <div class="mb32">
                    <h5 class="border-bottom pb-1">Completed Courses</h5>
                    <t t-if="courses_completed" t-call="website_slides.display_course">
                        <t t-set="courses" t-value="courses_completed"></t>
                    </t>
                    <div t-elif="request.env.user == user" class="text-muted d-inline-block">
                        Go through all its content to see a Course in this section. <br />
                        <a href="/slides/" class="btn-link">
                            <i class="fa fa-arrow-right"></i> Start Learning
                        </a>
                    </div>
                    <div t-else="" class="text-muted d-inline-block">No completed courses yet!</div>
                    <div t-if="request.env.user != user" class="text-end d-inline-block pull-right">
                        <a href="/slides/all" class="btn btn-link btn-sm"><i class="oi oi-arrow-right me-1"/>All Courses</a>
                    </div>
                </div>
                <div class="mb32">
                    <h5 class="border-bottom pb-1">Ongoing Courses</h5>
                    <t t-if="courses_ongoing" t-call="website_slides.display_course">
                        <t t-set="courses" t-value="courses_ongoing"></t>
                    </t>
                     <p t-elif="request.env.user == user" class="text-muted">
                        All the courses you attend will appear here. <br />
                        <a href="/slides/" class="btn-link">
                            <i class="fa fa-arrow-right"></i> Start Learning
                        </a>
                    </p>
                    <p t-else="" class="text-muted">No ongoing courses yet!</p>
                </div>
            </t>
        </xpath>
    </template>

    <template id="display_course">
        <div class="row">
            <div class="col-12 col-lg-6" t-foreach="courses" t-as="course">
                <div class="card mb-2">
                    <div class="card-body o_wprofile_slides_course_card_body p-0 d-flex"
                        t-attf-onclick="location.href='/slides/#{slug(course.channel_id)}';">
                        <div t-field="course.channel_id.image_1920" t-options="{'widget': 'image', 'class':'o_wslides_course_card_image', 'preview_image': 'image_512'}" class="rounded-start"/>
                        <div class="p-2 w-100">
                            <h5 class="mt-0 mb-1" t-field="course.channel_id.name"/>

                            <div class="overflow-hidden mb-1" style="height:24px">
                                <t t-foreach="course.channel_id.tag_ids.filtered(lambda tag: tag.color)" t-as="tag">
                                    <a t-att-href="'/slides/all/tag/%s' % slug(tag)" onclick="event.stopPropagation()" t-attf-class="badge o_wslides_channel_tag post_link #{'o_color_'+str(tag.color)}" t-esc="tag.name"/>
                                </t>
                            </div>

                            <div class="d-flex align-items-center">
                                <div class="progress flex-grow-1" style="height:0.5em">
                                    <div class="progress-bar bg-primary" t-att-style="'width: '+ str(course.completion)+'%'"/>
                                </div>
                                <small class="fw-bold ps-2"><span t-esc="course.completion"/> %</small>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </template>
</data></odoo>

```

## File: views\website_slides_templates_utils.xml

```xml
<?xml version="1.0" ?>
<odoo><data>

<!-- Share on social networks -->
<template id='slide_share_social' name="Slides Media Share">
    <t t-if="not website_share_url">
        <t t-set="website_share_url"
           t-value="record.website_share_url if 'website_share_url' in record else record.website_url"/>
    </t>
    <div class="btn-group" role="group">
        <div class="s_share">
            <a t-attf-href="https://www.facebook.com/sharer/sharer.php?u=#{website_share_url}" class="btn border bg-white o_wslides_js_social_share"
                social-key="facebook" aria-label="Share on Facebook" title="Share on Facebook" target="_blank">
                <i class="fa fa-facebook-square fa-fw"/>
            </a>
            <a t-attf-href="https://twitter.com/intent/tweet?text=#{record.name}&amp;url=#{website_share_url}" class="btn border bg-white o_wslides_js_social_share"
                social-key="twitter" aria-label="Share on X" title="Share on X" target="_blank">
                <i class="fa fa-twitter fa-fw"/>
            </a>
            <a t-attf-href="http://www.linkedin.com/sharing/share-offsite/?url=#{website_share_url}" class="btn border bg-white o_wslides_js_social_share"
                social-key="linkedin" aria-label="Share on LinkedIn" title="Share on LinkedIn" target="_blank">
                <i class="fa fa-linkedin fa-fw"/>
            </a>
            <a t-attf-href="https://wa.me/?text=#{website_share_url}" class="btn border bg-white o_wslides_js_social_share"
                social-key="whatsapp" aria-label="Share on Whatsapp" title="Share on Whatsapp" target="_blank">
                <i class="fa fa-whatsapp fa-fw"/>
            </a>
            <a t-attf-href="http://pinterest.com/pin/create/button/?url=#{website_share_url}"
                social-key="pinterest"
                class="btn border bg-white o_wslides_js_social_share"
                aria-label="Share on Pinterest" title="Share on Pinterest">
                <i class="fa fa-pinterest fa-fw"/>
            </a>
        </div>
    </div>
</template>

<!-- Slide sub-template: share: send by email -->
<template id='slide_social_email' name="Share by Email">
    <h5 class="mt-4">Share by Email</h5>
    <div t-if="not is_public_user">
        <div class="oe_slide_js_share_email">
            <div class="input-group">
                <input type="text" class="form-control" placeholder="your-friend@domain.com, your-friend2@domain.com"/>
                <button class="btn btn-primary" type="button"
                    data-loading-text="Sending..."
                    t-attf-data-slide-id="#{record.id if record._name == 'slide.slide' else False}"
                    t-attf-data-channel-id="#{record.id if record._name == 'slide.channel' else False}"
                    style="border-top-end-radius: 4px;border-bottom-end-radius: 4px;">
                    <i class="fa fa-envelope"/> Send Email
                </button>
            </div>
            <div class="alert alert-info d-none" role="alert">
                <strong>Sharing is caring!</strong> Email(s) sent.
            </div>
            <div class="alert alert-warning d-none" role="alert">Please enter valid email(s)</div>
        </div>
    </div>
    <div t-if="is_public_user" class="alert alert-info d-inline-block">
        <p class="mb-0">Please <a t-attf-href="/odoo?redirect=#{request.httprequest.url}" class="fw-bold"> login </a> to share this
        <span t-field="record.slide_category" t-if="record._name == 'slide.slide'"/>
        <span t-field="record.name" t-if="record._name == 'slide.channel'"/> by email.</p>
    </div>
</template>

<!-- Slide sub-template: share: embed in your website -->
<template id="slide_social_embed" name="Share on Your Website">
    <div class="oe_slide_js_embed_code_widget mt-4">
        <h5 class="mt0">Embed in another Website</h5>
        <div>
            <textarea class="form-control slide_embed_code" readonly="readonly" onClick="this.select();" t-attf-id="wslides_share_embed_id_{{record.id}}">
                <!-- Use the external embedding URL here, see python controller '_slide_embed' method for more details. -->
                <t t-esc="slide.embed_code_external"/>
            </textarea>
            <button t-att-id="'share_embed_clipboard_button_id_%s' % record.id" class="btn btn-sm btn-primary o_embed_clipboard_button float-end mt-1 p-2" >
                <i class="fa fa-clipboard"/> Copy Embed Code
            </button>
        </div>
        <div t-if="slide.slide_category == 'document' and not embed_hide_starting_page" class="input-group mt-5">
            <span class="input-group-text">Start at Page</span>
            <input type="number" value="1" class="form-control"/>
        </div>
    </div>
</template>

<!-- Share: social media -->
<template id='slide_share_modal'>
    <div class="modal fade" t-att-id="'slideShareModal_%s' % record.id" tabindex="-1" role="dialog" aria-labelledby="slideShareModalLabel" aria-hidden="true">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <t t-call="website_slides.slide_share_modal_header"/>
                <t t-call="website_slides.slide_share_modal_body"/>
            </div>
        </div>
    </div>
</template>

<template id="slide_share_modal_header">
    <div class="modal-header">
        <h5 class="modal-title" id="slideShareModalLabel">
            <t t-if="slide_share_modal_title" t-out="slide_share_modal_title"/>
            <t t-else="">Share This Content</t>
        </h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
    </div>
</template>

<template id="slide_share_modal_body">
    <div class="modal-body">
        <t t-call="website_slides.slide_share_link"/>
        <h5 class="mt-3">Share on Social Media</h5>
        <t t-call="website_slides.slide_share_social"/>
        <t t-if="email_sharing" t-call="website_slides.slide_social_email"/>
        <t t-if="include_embed" t-call="website_slides.slide_social_embed"/>
    </div>
</template>

<template id="slide_share_link">
    <t t-if="not website_share_url">
        <t t-set="website_share_url"
           t-value="record.website_share_url if 'website_share_url' in record else record.website_url"/>
    </t>
    <h5>Share Link</h5>
    <div class="input-group">
        <input type="text" t-attf-id="wslides_share_link_id_{{record.id}}"
            class="form-control o_wslides_js_share_link text-center"
            readonly="readonly" onclick="this.select();"
            t-att-value="website_share_url"/>
        <button t-att-id="'share_link_clipboard_button_id_%s' % record.id" class="btn btn-sm btn-primary o_clipboard_button" >
            <i class="fa fa-clipboard"/> Copy Link
        </button>
    </div>
</template>

<template id="join_course_link" name="Join Course Link">
    <a class="o_wslides_js_course_join_link" href="#" t-att-data-channel-enroll="slide.channel_id.enroll"
       t-att-data-channel-id="slide.channel_id.id">Join this Course</a><t t-if="for_resources"> to access resources</t>
</template>

<!-- Python equivalent of the JS template "website.slides.sidebar.done.button" -->
<template id="slide_sidebar_done_button" name="Sidebar Done Button">
    <t t-set="uncompleted_icon" t-value="use_slide_icon and slide.slide_icon_class or 'fa-circle-thin'"/>
    <div t-if="is_member"
        class="o_wslides_sidebar_done_button align-self-start"
        t-att-data-id="slide.id"
        t-att-data-uncompleted-icon="uncompleted_icon"
        t-att-data-completed="slide_completed"
        t-att-data-can-self-mark-completed="slide.can_self_mark_completed"
        t-att-data-can-self-mark-uncompleted="slide.can_self_mark_uncompleted"
        t-att-data-is-member="is_member">
        <button class="o_wslides_button_complete btn btn-sm" t-if="slide_completed and slide.can_self_mark_uncompleted or not slide_completed and slide.can_self_mark_completed">
            <i t-if="slide_completed" class="o_wslides_slide_completed fa fa-check-circle fa-fw text-success fa-lg" t-att-data-slide-id="slide.id" title="Mark as not done"/>
            <i t-else="" t-attf-class="fa #{uncompleted_icon} fa-fw fa-lg" t-att-data-slide-id="slide.id" title="Mark as done"/>
        </button>
        <button class="o_wslides_button_complete o_wslides_button_uncompleted btn btn-sm border-0" t-else="" disabled="1">
            <i t-if="slide_completed" class="o_wslides_slide_completed fa fa-check fa-fw text-success fa-lg" t-att-data-slide-id="slide.id" title="Can not be marked as not done"/>
            <i t-else="" t-attf-class="fa #{uncompleted_icon} fa-fw fa-lg" t-att-data-slide-id="slide.id" title="Can not be marked as done"/>
        </button>
    </div>
</template>

</data></odoo>

```

## File: wizard\slide_channel_invite.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import re

from odoo import api, fields, models, _
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)

emails_split = re.compile(r"[;,\n\r]+")


class SlideChannelInvite(models.TransientModel):
    _name = 'slide.channel.invite'
    _inherit = 'mail.composer.mixin'
    _description = 'Channel Invitation Wizard'

    # composer content
    attachment_ids = fields.Many2many('ir.attachment', string='Attachments')
    send_email = fields.Boolean('Send Email', compute="_compute_send_email", readonly=False, store=True)
    # recipients
    partner_ids = fields.Many2many('res.partner', string='Recipients')
    # slide channel
    channel_id = fields.Many2one('slide.channel', string='Course', required=True)
    channel_invite_url = fields.Char('Course Link', compute='_compute_channel_invite_url')
    channel_visibility = fields.Selection(related='channel_id.visibility')
    channel_published = fields.Boolean(related='channel_id.is_published')
    # membership
    enroll_mode = fields.Boolean(
        'Enroll partners', readonly=True,
        help='Whether invited partners will be added as enrolled. Otherwise, they will be added as invited.')

    @api.depends('channel_id')
    def _compute_channel_invite_url(self):
        for invite in self:
            channel = invite.channel_id
            invite.channel_invite_url = f'{channel.get_base_url()}/slides/{channel.id}'

    # Overrides of mail.composer.mixin
    @api.depends('channel_id')  # fake trigger otherwise not computed in new mode
    def _compute_render_model(self):
        self.render_model = 'slide.channel.partner'

    @api.depends('channel_id', 'enroll_mode')
    def _compute_send_email(self):
        self.send_email = self.channel_visibility != 'public' or self.enroll_mode

    def action_invite(self):
        """ Process the wizard content and proceed with sending the related email(s),
            rendering any template patterns on the fly if needed. This method is used both
            to add members as 'joined' (when adding attendees) and as 'invited' (on invitation),
            depending on the value of enroll_mode. Archived members can be invited or enrolled.
            They will become 'invited', or another status if enrolled depending on their progress.
            Invited members can be reinvited, or enrolled depending on enroll_mode. """
        self.ensure_one()

        if not self.send_email:
            return
        if not self.env.user.email:
            raise UserError(_("Unable to post message, please configure the sender's email address."))
        if not self.partner_ids:
            raise UserError(_("Please select at least one recipient."))

        mail_values = []
        attendees_to_reinvite = self.env['slide.channel.partner'].search([
            ('member_status', '=', 'invited'),
            ('channel_id', '=', self.channel_id.id),
            ('partner_id', 'in', self.partner_ids.ids)
        ]) if not self.enroll_mode else self.env['slide.channel.partner']

        channel_partners = self.channel_id._action_add_members(
            self.partner_ids - attendees_to_reinvite.partner_id,
            member_status='joined' if self.enroll_mode else 'invited',
            raise_on_access=True
        )
        if not self.enroll_mode:
            (attendees_to_reinvite | channel_partners).last_invitation_date = fields.Datetime.now()

        for channel_partner in (attendees_to_reinvite | channel_partners):
            mail_values.append(self._prepare_mail_values(channel_partner))
        self.env['mail.mail'].sudo().create(mail_values)

        return {'type': 'ir.actions.act_window_close'}

    def _prepare_mail_values(self, slide_channel_partner):
        """ Create mail specific for recipient """
        lang = self._render_lang(slide_channel_partner.ids)[slide_channel_partner.id]
        subject = self._render_field('subject', slide_channel_partner.ids, set_lang=lang)[slide_channel_partner.id]
        body = self._render_field('body', slide_channel_partner.ids, set_lang=lang)[slide_channel_partner.id]
        # post the message
        mail_values = {
            'attachment_ids': [(4, att.id) for att in self.attachment_ids],
            'author_id': self.env.user.partner_id.id,
            'auto_delete': self.template_id.auto_delete if self.template_id else True,
            'body_html': body,
            'email_from': self.env.user.email_formatted,
            'model': None,
            'recipient_ids': [(4, slide_channel_partner.partner_id.id)],
            'res_id': None,
            'subject': subject,
        }

        # optional support of default_email_layout_xmlid in context
        email_layout_xmlid = self.env.context.get('default_email_layout_xmlid', self.env.context.get('notif_layout'))
        if email_layout_xmlid:
            # could be great to use ``_notify_by_email_prepare_rendering_context`` someday
            template_ctx = {
                'message': self.env['mail.message'].sudo().new({'body': mail_values['body_html'], 'record_name': self.channel_id.name}),
                'model_description': self.env['ir.model']._get('slide.channel').display_name,
                'record': slide_channel_partner,
                'company': self.env.company,
                'signature': self.channel_id.user_id.signature,
            }
            body = self.env['ir.qweb']._render(email_layout_xmlid, template_ctx, engine='ir.qweb', minimal_qcontext=True, raise_if_not_found=False, lang=lang)
            if body:
                mail_values['body_html'] = self.env['mail.render.mixin']._replace_local_links(body)
            else:
                _logger.warning('QWeb template %s not found when sending slide channel mails. Sending without layout.', email_layout_xmlid)

        return mail_values

```

## File: wizard\slide_channel_invite_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="slide_channel_invite_view_form" model="ir.ui.view">
            <field name="name">slide.channel.invite.view.form</field>
            <field name="model">slide.channel.invite</field>
            <field name="arch" type="xml">
                <form string="Compose Email" class="o_mail_composer_form">
                    <div class="alert alert-warning text-center" role="alert" invisible="channel_published or not channel_id">
                        This course is not published. Attendees may not be able to access its contents.
                    </div>
                    <sheet>
                        <field name="channel_id" invisible="1"/>
                        <field name="channel_published" invisible="1"/>
                        <field name="channel_visibility" invisible="1"/>
                        <field name="enroll_mode" invisible="1"/>
                        <group col="1">
                            <group col="2" invisible="enroll_mode or channel_visibility != 'public'">
                                <field name="channel_invite_url" readonly="1" widget="CopyClipboardChar"/>
                                <field name="send_email" widget="boolean_toggle" options="{'autosave': False}"/>
                            </group>
                            <group col="2" invisible="not send_email">
                                <field name="partner_ids"
                                    widget="many2many_tags_email"
                                    placeholder="Add contacts..."
                                    required="send_email"
                                    options="{'no_quick_create': True}"
                                    context="{'show_email': True, 'form_view_ref': 'base.view_partner_simple_form'}"/>
                            </group>
                            <group col="2" invisible="not send_email">
                                <field name="lang" invisible="1"/>
                                <field name="render_model" invisible="1"/>
                                <field name="subject" placeholder="Subject..."/>
                            </group>
                            <field name="can_edit_body" invisible="1"/>
                            <field name="body" class="oe-bordered-editor" widget="html_mail" invisible="not send_email" readonly="not can_edit_body" force_save="1"/>
                            <group invisible="not send_email">
                                <group>
                                    <field name="attachment_ids" widget="many2many_binary"/>
                                </group>
                                <group>
                                    <field name="template_id" label="Use template" context="{'default_model': 'slide.channel.partner'}"/>
                                </group>
                            </group>
                        </group>
                    </sheet>
                    <footer>
                        <button string="Send" invisible="not send_email" name="action_invite" type="object" class="btn-primary" data-hotkey="q"/>
                        <button string="Close" invisible="send_email" class="btn-secondary" special="cancel" data-hotkey="x"/>
                        <button string="Cancel" invisible="not send_email" class="btn-secondary" special="cancel" data-hotkey="x"/>
                    </footer>
                </form>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import slide_channel_invite

```

