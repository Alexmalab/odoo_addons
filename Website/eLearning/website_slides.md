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
    'version': '2.2',
    'sequence': 125,
    'summary': 'Manage and publish an eLearning platform',
    'website': 'https://www.odoo.com/page/slides',
    'category': 'Website/eLearning',
    'description': """
Create Online Courses
=====================

Featuring

 * Integrated course and lesson management
 * Fullscreen navigation
 * Support Youtube videos, Google documents, PDF, images, web pages
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
        'views/assets.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
        'views/rating_rating_views.xml',
        'views/slide_question_views.xml',
        'views/slide_slide_views.xml',
        'views/slide_channel_partner_views.xml',
        'views/slide_channel_views.xml',
        'views/slide_channel_tag_views.xml',
        'views/website_slides_menu_views.xml',
        'views/website_slides_templates_homepage.xml',
        'views/website_slides_templates_course.xml',
        'views/website_slides_templates_lesson.xml',
        'views/website_slides_templates_lesson_fullscreen.xml',
        'views/website_slides_templates_lesson_embed.xml',
        'views/website_slides_templates_profile.xml',
        'views/website_slides_templates_utils.xml',
        'wizard/slide_channel_invite_views.xml',
        'data/gamification_data.xml',
        'data/mail_data.xml',
        'data/mail_activity_data.xml',
        'data/slide_data.xml',
        'data/website_data.xml',
    ],
    'demo': [
        'data/res_users_demo.xml',
        'data/slide_channel_tag_demo.xml',
        'data/slide_channel_demo.xml',
        'data/slide_slide_demo.xml',
        'data/slide_user_demo.xml',
    ],
    'qweb': [
        'static/src/xml/activity.xml',
    ],
    'installable': True,
    'application': True,
    'license': 'LGPL-3',
}

```

## File: controllers\mail.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import werkzeug

from werkzeug.exceptions import NotFound, Forbidden

from odoo import http
from odoo.http import request
from odoo.addons.portal.controllers.mail import _check_special_access, PortalChatter
from odoo.tools import plaintext2html, html2plaintext


class SlidesPortalChatter(PortalChatter):

    @http.route(['/mail/chatter_post'], type='http', methods=['POST'], auth='public', website=True)
    def portal_chatter_post(self, res_model, res_id, message, **kw):
        result = super(SlidesPortalChatter, self).portal_chatter_post(res_model, res_id, message, **kw)
        if res_model == 'slide.channel':
            rating_value = kw.get('rating_value', False)
            slide_channel = request.env[res_model].sudo().browse(int(res_id))
            if rating_value and slide_channel and request.env.user.partner_id.id == int(kw.get('pid')):
                # apply karma gain rule only once
                request.env.user.add_karma(slide_channel.karma_gen_channel_rank)
        return result

    @http.route([
        '/slides/mail/update_comment',
        '/mail/chatter_update',
        ], type='http', auth="user", methods=['POST'])
    def mail_update_message(self, res_model, res_id, message, message_id, redirect=None, attachment_ids='', attachment_tokens='', **post):
        # keep this mechanism intern to slide currently (saas 12.5) as it is
        # considered experimental
        if res_model != 'slide.channel':
            raise Forbidden()
        res_id = int(res_id)

        attachment_ids = [int(attachment_id) for attachment_id in attachment_ids.split(',') if attachment_id]
        attachment_tokens = [attachment_token for attachment_token in attachment_tokens.split(',') if attachment_token]
        self._portal_post_check_attachments(attachment_ids, attachment_tokens)

        pid = int(post['pid']) if post.get('pid') else False
        if not _check_special_access(res_model, res_id, token=post.get('token'), _hash=post.get('hash'), pid=pid):
            raise Forbidden()

        # fetch and update mail.message
        message_id = int(message_id)
        message_body = plaintext2html(message)
        subtype_comment_id = request.env['ir.model.data'].xmlid_to_res_id('mail.mt_comment')
        domain = [
            ('model', '=', res_model),
            ('res_id', '=', res_id),
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
        if post.get('rating_value'):
            domain = [('res_model', '=', res_model), ('res_id', '=', res_id), ('message_id', '=', message.id)]
            rating = request.env['rating.rating'].sudo().search(domain, order='write_date DESC', limit=1)
            rating.write({
                'rating': float(post['rating_value']),
                'feedback': html2plaintext(message.body),
            })

        # redirect to specified or referrer or simply channel page as fallback
        redirect_url = redirect or (request.httprequest.referrer and request.httprequest.referrer + '#review') or '/slides/%s' % res_id
        return werkzeug.utils.redirect(redirect_url, 302)

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import json
import logging
import werkzeug
import math

from ast import literal_eval
from collections import defaultdict

from odoo import http, tools, _
from odoo.addons.http_routing.models.ir_http import slug
from odoo.addons.website_profile.controllers.main import WebsiteProfile
from odoo.addons.website.models.ir_http import sitemap_qs2dom
from odoo.exceptions import AccessError, UserError
from odoo.http import request
from odoo.osv import expression

_logger = logging.getLogger(__name__)


class WebsiteSlides(WebsiteProfile):
    _slides_per_page = 12
    _slides_per_aside = 20
    _slides_per_category = 4
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
            loc = '/slides/%s' % slug(channel)
            if not qs or qs.lower() in loc:
                yield {'loc': loc}

    # SLIDE UTILITIES
    # --------------------------------------------------

    def _fetch_slide(self, slide_id):
        slide = request.env['slide.slide'].browse(int(slide_id)).exists()
        if not slide:
            return {'error': 'slide_wrong'}
        try:
            slide.check_access_rights('read')
            slide.check_access_rule('read')
        except AccessError:
            return {'error': 'slide_access'}
        return {'slide': slide}

    def _set_viewed_slide(self, slide, quiz_attempts_inc=False):
        if request.env.user._is_public() or not slide.website_published or not slide.channel_id.is_member:
            viewed_slides = request.session.setdefault('viewed_slides', list())
            if slide.id not in viewed_slides:
                if tools.sql.increment_field_skiplock(slide, 'public_views'):
                    viewed_slides.append(slide.id)
                    request.session['viewed_slides'] = viewed_slides
        else:
            slide.action_set_viewed(quiz_attempts_inc=quiz_attempts_inc)
        return True

    def _set_completed_slide(self, slide):
        # quiz use their specific mechanism to be marked as done
        if slide.slide_type == 'quiz' or slide.question_ids:
            raise UserError(_("Slide with questions must be marked as done when submitting all good answers "))
        if slide.website_published and slide.channel_id.is_member:
            slide.action_set_completed()
        return True

    def _get_slide_detail(self, slide):
        base_domain = self._get_channel_slides_base_domain(slide.channel_id)
        if slide.channel_id.channel_type == 'documentation':
            related_domain = expression.AND([base_domain, [('category_id', '=', slide.category_id.id)]])

            most_viewed_slides = request.env['slide.slide'].search(base_domain, limit=self._slides_per_aside, order='total_views desc')
            related_slides = request.env['slide.slide'].search(related_domain, limit=self._slides_per_aside)
            category_data = []
            uncategorized_slides = request.env['slide.slide']
        else:
            most_viewed_slides, related_slides = request.env['slide.slide'], request.env['slide.slide']
            category_data = slide.channel_id._get_categorized_slides(
                base_domain, order=request.env['slide.slide']._order_by_strategy['sequence'],
                force_void=True)
            # temporarily kept for fullscreen, to remove asap
            uncategorized_domain = expression.AND([base_domain, [('channel_id', '=', slide.channel_id.id), ('category_id', '=', False)]])
            uncategorized_slides = request.env['slide.slide'].search(uncategorized_domain)

        channel_slides_ids = slide.channel_id.slide_content_ids.ids
        slide_index = channel_slides_ids.index(slide.id)
        previous_slide = slide.channel_id.slide_content_ids[slide_index-1] if slide_index > 0 else None
        next_slide = slide.channel_id.slide_content_ids[slide_index+1] if slide_index < len(channel_slides_ids) - 1 else None

        values = {
            # slide
            'slide': slide,
            'main_object': slide,
            'most_viewed_slides': most_viewed_slides,
            'related_slides': related_slides,
            'previous_slide': previous_slide,
            'next_slide': next_slide,
            'uncategorized_slides': uncategorized_slides,
            'category_data': category_data,
            # user
            'user': request.env.user,
            'is_public_user': request.website.is_public_user(),
            # rating and comments
            'comments': slide.website_message_ids or [],
        }

        # allow rating and comments
        if slide.channel_id.allow_comment:
            values.update({
                'message_post_pid': request.env.user.partner_id.id,
            })

        return values

    def _get_slide_quiz_partner_info(self, slide, quiz_done=False):
        return slide._compute_quiz_info(request.env.user.partner_id, quiz_done=quiz_done)[slide.id]

    def _get_slide_quiz_data(self, slide):
        slide_completed = slide.user_membership_id.sudo().completed
        values = {
            'slide_questions': [{
                'id': question.id,
                'question': question.question,
                'answer_ids': [{
                    'id': answer.id,
                    'text_value': answer.text_value,
                    'is_correct': answer.is_correct if slide_completed or request.website.is_publisher() else None,
                    'comment': answer.comment if request.website.is_publisher() else None
                } for answer in question.sudo().answer_ids],
            } for question in slide.question_ids]
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

    def _extract_channel_tag_search(self, **post):
        tags = request.env['slide.channel.tag']
        if post.get('tags'):
            try:
                tag_ids = literal_eval(post['tags'])
            except:
                pass
            else:
                # perform a search to filter on existing / valid tags implicitely
                tags = request.env['slide.channel.tag'].search([('id', 'in', tag_ids)])
        return tags

    def _build_channel_domain(self, base_domain, slide_type=None, my=False, **post):
        search_term = post.get('search')
        tags = self._extract_channel_tag_search(**post)

        domain = base_domain
        if search_term:
            domain = expression.AND([
                domain,
                ['|', ('name', 'ilike', search_term), ('description', 'ilike', search_term)]])

        if tags:
            # Group by group_id
            grouped_tags = defaultdict(list)
            for tag in tags:
                grouped_tags[tag.group_id].append(tag)

            # OR inside a group, AND between groups.
            group_domain_list = []
            for group in grouped_tags:
                group_domain_list.append([('tag_ids', 'in', [tag.id for tag in grouped_tags[group]])])

            domain = expression.AND([domain, *group_domain_list])

        if slide_type and 'nbr_%s' % slide_type in request.env['slide.channel']:
            domain = expression.AND([domain, [('nbr_%s' % slide_type, '>', 0)]])

        if my:
            domain = expression.AND([domain, [('partner_ids', '=', request.env.user.partner_id.id)]])
        return domain

    def _channel_remove_session_answers(self, channel, slide=False):
        """ Will remove the answers saved in the session for a specific channel / slide. """

        if 'slide_answer_quiz' not in request.session:
            return

        slides_domain = [('channel_id', '=', channel.id)]
        if slide:
            slides_domain = expression.AND([slides_domain, [('id', '=', slide.id)]])
        slides = request.env['slide.slide'].search_read(slides_domain, ['id'])

        session_slide_answer_quiz = json.loads(request.session['slide_answer_quiz'])
        for slide in slides:
            session_slide_answer_quiz.pop(str(slide['id']), None)
        request.session['slide_answer_quiz'] = json.dumps(session_slide_answer_quiz)

    # TAG UTILITIES
    # --------------------------------------------------

    def _create_or_get_channel_tag(self, tag_id, group_id):
        if not tag_id:
            return request.env['slide.channel.tag']
        # handle creation of new channel tag
        if tag_id[0] == 0:
            group_id = self._create_or_get_channel_tag_group(group_id)
            if not group_id:
                return {'error': _('Missing "Tag Group" for creating a new "Tag".')}

            new_tag = request.env['slide.channel.tag'].create({
                'name': tag_id[1]['name'],
                'group_id': group_id,
            })
            return new_tag
        return request.env['slide.channel.tag'].browse(tag_id[0])

    def _create_or_get_channel_tag_group(self, group_id):
        if not group_id:
            return False
        # handle creation of new channel tag group
        if group_id[0] == 0:
            tag_group = request.env['slide.channel.tag.group'].create({
                'name': group_id[1]['name'],
            })
            group_id = tag_group.id
        # use existing channel tag group
        return group_id[0]

    # --------------------------------------------------
    # SLIDE.CHANNEL MAIN / SEARCH
    # --------------------------------------------------

    @http.route('/slides', type='http', auth="public", website=True, sitemap=True)
    def slides_channel_home(self, **post):
        """ Home page for eLearning platform. Is mainly a container page, does not allow search / filter. """
        domain = request.website.website_domain()
        channels_all = request.env['slide.channel'].search(domain)
        if not request.env.user._is_public():
            #If a course is completed, we don't want to see it in first position but in last
            channels_my = channels_all.filtered(lambda channel: channel.is_member).sorted(lambda channel: 0 if channel.completed else channel.completion, reverse=True)[:3]
        else:
            channels_my = request.env['slide.channel']
        channels_popular = channels_all.sorted('total_votes', reverse=True)[:3]
        channels_newest = channels_all.sorted('create_date', reverse=True)[:3]

        achievements = request.env['gamification.badge.user'].sudo().search([('badge_id.is_published', '=', True)], limit=5)
        if request.env.user._is_public():
            challenges = None
            challenges_done = None
        else:
            challenges = request.env['gamification.challenge'].sudo().search([
                ('challenge_category', '=', 'slides'),
                ('reward_id.is_published', '=', True)
            ], order='id asc', limit=5)
            challenges_done = request.env['gamification.badge.user'].sudo().search([
                ('challenge_id', 'in', challenges.ids),
                ('user_id', '=', request.env.user.id),
                ('badge_id.is_published', '=', True)
            ]).mapped('challenge_id')

        users = request.env['res.users'].sudo().search([
            ('karma', '>', 0),
            ('website_published', '=', True)], limit=5, order='karma desc')

        values = self._prepare_user_values(**post)
        values.update({
            'channels_my': channels_my,
            'channels_popular': channels_popular,
            'channels_newest': channels_newest,
            'achievements': achievements,
            'users': users,
            'top3_users': self._get_top3_users(),
            'challenges': challenges,
            'challenges_done': challenges_done,
            'search_tags': request.env['slide.channel.tag']
        })

        return request.render('website_slides.courses_home', values)

    @http.route('/slides/all', type='http', auth="public", website=True, sitemap=True)
    def slides_channel_all(self, slide_type=None, my=False, **post):
        """ Home page displaying a list of courses displayed according to some
        criterion and search terms.

          :param string slide_type: if provided, filter the course to contain at
           least one slide of type 'slide_type'. Used notably to display courses
           with certifications;
          :param bool my: if provided, filter the slide.channels for which the
           current user is a member of
          :param dict post: post parameters, including

           * ``search``: filter on course description / name;
           * ``channel_tag_id``: filter on courses containing this tag;
           * ``channel_tag_group_id_<id>``: filter on courses containing this tag
             in the tag group given by <id> (used in navigation based on tag group);
        """
        domain = request.website.website_domain()
        domain = self._build_channel_domain(domain, slide_type=slide_type, my=my, **post)

        order = self._channel_order_by_criterion.get(post.get('sorting'))

        channels = request.env['slide.channel'].search(domain, order=order)
        # channels_layouted = list(itertools.zip_longest(*[iter(channels)] * 4, fillvalue=None))

        tag_groups = request.env['slide.channel.tag.group'].search(
            ['&', ('tag_ids', '!=', False), ('website_published', '=', True)])
        search_tags = self._extract_channel_tag_search(**post)

        values = self._prepare_user_values(**post)
        values.update({
            'channels': channels,
            'tag_groups': tag_groups,
            'search_term': post.get('search'),
            'search_slide_type': slide_type,
            'search_my': my,
            'search_tags': search_tags,
            'search_channel_tag_id': post.get('channel_tag_id'),
            'top3_users': self._get_top3_users(),
        })

        return request.render('website_slides.courses_all', values)

    def _prepare_additional_channel_values(self, values, **kwargs):
        return values

    def _get_top3_users(self):
        return request.env['res.users'].sudo().search_read([
            ('karma', '>', 0),
            ('website_published', '=', True),
            ('image_1920', '!=', False)], ['id'], limit=3, order='karma desc')

    @http.route([
        '/slides/<model("slide.channel"):channel>',
        '/slides/<model("slide.channel"):channel>/page/<int:page>',
        '/slides/<model("slide.channel"):channel>/tag/<model("slide.tag"):tag>',
        '/slides/<model("slide.channel"):channel>/tag/<model("slide.tag"):tag>/page/<int:page>',
        '/slides/<model("slide.channel"):channel>/category/<model("slide.slide"):category>',
        '/slides/<model("slide.channel"):channel>/category/<model("slide.slide"):category>/page/<int:page>',
    ], type='http', auth="public", website=True, sitemap=sitemap_slide)
    def channel(self, channel, category=None, tag=None, page=1, slide_type=None, uncategorized=False, sorting=None, search=None, **kw):
        """
        Will return all necessary data to display the requested slide_channel along with a possible category.
        """
        if not channel.can_access_from_current_website():
            raise werkzeug.exceptions.NotFound()

        domain = self._get_channel_slides_base_domain(channel)

        pager_url = "/slides/%s" % (channel.id)
        pager_args = {}
        slide_types = dict(request.env['slide.slide']._fields['slide_type']._description_selection(request.env))

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
                domain += [('tag_ids.id', '=', tag.id)]
                pager_url += "/tag/%s" % tag.id
            if uncategorized:
                domain += [('category_id', '=', False)]
                pager_args['uncategorized'] = 1
            elif slide_type:
                domain += [('slide_type', '=', slide_type)]
                pager_url += "?slide_type=%s" % slide_type

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
        elif slide_type:
            query_string = "?search_slide_type=%s" % slide_type
        elif uncategorized:
            query_string = "?search_uncategorized=1"

        values = {
            'channel': channel,
            'main_object': channel,
            'active_tab': kw.get('active_tab', 'home'),
            # search
            'search_category': category,
            'search_tag': tag,
            'search_slide_type': slide_type,
            'search_uncategorized': uncategorized,
            'query_string': query_string,
            'slide_types': slide_types,
            'sorting': actual_sorting,
            'search': search,
            # chatter
            'rating_avg': channel.rating_avg,
            'rating_count': channel.rating_count,
            # display data
            'user': request.env.user,
            'pager': pager,
            'is_public_user': request.website.is_public_user(),
            # display upload modal
            'enable_slide_upload': 'enable_slide_upload' in kw,
        }
        if not request.env.user._is_public():
            subtype_comment_id = request.env['ir.model.data'].xmlid_to_res_id('mail.mt_comment')
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

        # fetch slides and handle uncategorized slides; done as sudo because we want to display all
        # of them but unreachable ones won't be clickable (+ slide controller will crash anyway)
        # documentation mode may display less slides than content by category but overhead of
        # computation is reasonable
        if channel.promote_strategy == 'specific':
            values['slide_promoted'] = channel.sudo().promoted_slide_id
        else:
            values['slide_promoted'] = request.env['slide.slide'].sudo().search(domain, limit=1, order=order)

        limit_category_data = False
        if channel.channel_type == 'documentation':
            if category or uncategorized:
                limit_category_data = self._slides_per_page
            else:
                limit_category_data = self._slides_per_category

        values['category_data'] = channel._get_categorized_slides(
            domain, order,
            force_void=not category,
            limit=limit_category_data,
            offset=pager['offset'])
        values['channel_progress'] = self._get_channel_progress(channel, include_quiz=True)

        # for sys admins: prepare data to install directly modules from eLearning when
        # uploading slides. Currently supporting only survey, because why not.
        if request.env.user.has_group('base.group_system'):
            module = request.env.ref('base.module_survey')
            if module.state != 'installed':
                values['modules_to_install'] = [{
                    'id': module.id,
                    'name': module.shortdesc,
                    'motivational': _('Evaluate and certify your students.'),
                }]

        values = self._prepare_additional_channel_values(values, **kw)
        return request.render('website_slides.course_main', values)

    # SLIDE.CHANNEL UTILS
    # --------------------------------------------------

    @http.route('/slides/channel/add', type='http', auth='user', methods=['POST'], website=True)
    def slide_channel_create(self, *args, **kw):
        channel = request.env['slide.channel'].create(self._slide_channel_prepare_values(**kw))
        return werkzeug.utils.redirect("/slides/%s" % (slug(channel)))

    def _slide_channel_prepare_values(self, **kw):
        # `tag_ids` is a string representing a list of int with coma. i.e.: '2,5,7'
        # We don't want to allow user to create tags and tag groups on the fly.
        tag_ids = []
        if kw.get('tag_ids'):
            tag_ids = [int(item) for item in kw['tag_ids'].split(',')]

        return {
            'name': kw['name'],
            'description': kw.get('description'),
            'channel_type': kw.get('channel_type', 'documentation'),
            'user_id': request.env.user.id,
            'tag_ids': [(6, 0, tag_ids)],
            'allow_comment': bool(kw.get('allow_comment')),
        }

    @http.route('/slides/channel/enroll', type='http', auth='public', website=True)
    def slide_channel_join_http(self, channel_id):
        # TDE FIXME: why 2 routes ?
        if not request.website.is_public_user():
            channel = request.env['slide.channel'].browse(int(channel_id))
            channel.action_add_member()
        return werkzeug.utils.redirect("/slides/%s" % (slug(channel)))

    @http.route(['/slides/channel/join'], type='json', auth='public', website=True)
    def slide_channel_join(self, channel_id):
        if request.website.is_public_user():
            return {'error': 'public_user', 'error_signup_allowed': request.env['res.users'].sudo()._get_signup_invitation_scope() == 'b2c'}
        success = request.env['slide.channel'].browse(channel_id).action_add_member()
        if not success:
            return {'error': 'join_done'}
        return success

    @http.route(['/slides/channel/leave'], type='json', auth='user', website=True)
    def slide_channel_leave(self, channel_id):
        channel = request.env['slide.channel'].browse(channel_id)
        channel._remove_membership(request.env.user.partner_id.ids)
        self._channel_remove_session_answers(channel)
        return True

    @http.route(['/slides/channel/tag/search_read'], type='json', auth='user', methods=['POST'], website=True)
    def slide_channel_tag_search_read(self, fields, domain):
        can_create = request.env['slide.channel.tag'].check_access_rights('create', raise_exception=False)
        return {
            'read_results': request.env['slide.channel.tag'].search_read(domain, fields),
            'can_create': can_create,
        }

    @http.route(['/slides/channel/tag/group/search_read'], type='json', auth='user', methods=['POST'], website=True)
    def slide_channel_tag_group_search_read(self, fields, domain):
        can_create = request.env['slide.channel.tag.group'].check_access_rights('create', raise_exception=False)
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

        tag_id and group_id values are provided by a Select2. Default "None" values allow for
        graceful failures in exceptional cases when values are not provided.

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

        return {'url': "/slides/%s" % (slug(channel))}

    @http.route(['/slides/channel/subscribe'], type='json', auth='user', website=True)
    def slide_channel_subscribe(self, channel_id):
        return request.env['slide.channel'].browse(channel_id).message_subscribe(partner_ids=[request.env.user.partner_id.id])

    @http.route(['/slides/channel/unsubscribe'], type='json', auth='user', website=True)
    def slide_channel_unsubscribe(self, channel_id):
        request.env['slide.channel'].browse(channel_id).message_unsubscribe(partner_ids=[request.env.user.partner_id.id])
        return True

    # --------------------------------------------------
    # SLIDE.SLIDE MAIN / SEARCH
    # --------------------------------------------------

    @http.route('''/slides/slide/<model("slide.slide"):slide>''', type='http', auth="public", website=True, sitemap=True)
    def slide_view(self, slide, **kwargs):
        if not slide.channel_id.can_access_from_current_website() or not slide.active:
            raise werkzeug.exceptions.NotFound()
        # redirection to channel's homepage for category slides
        if slide.is_category:
            return werkzeug.utils.redirect(slide.channel_id.website_url)
        self._set_viewed_slide(slide)

        values = self._get_slide_detail(slide)
        # quiz-specific: update with karma and quiz information
        if slide.question_ids:
            values.update(self._get_slide_quiz_data(slide))
        # sidebar: update with user channel progress
        values['channel_progress'] = self._get_channel_progress(slide.channel_id, include_quiz=True)

        # Allows to have breadcrumb for the previously used filter
        values.update({
            'search_category': slide.category_id if kwargs.get('search_category') else None,
            'search_tag': request.env['slide.tag'].browse(int(kwargs.get('search_tag'))) if kwargs.get('search_tag') else None,
            'slide_types': dict(request.env['slide.slide']._fields['slide_type']._description_selection(request.env)) if kwargs.get('search_slide_type') else None,
            'search_slide_type': kwargs.get('search_slide_type'),
            'search_uncategorized': kwargs.get('search_uncategorized')
        })

        values['channel'] = slide.channel_id
        values = self._prepare_additional_channel_values(values, **kwargs)
        values.pop('channel', None)

        values['signup_allowed'] = request.env['res.users'].sudo()._get_signup_invitation_scope() == 'b2c'

        if kwargs.get('fullscreen') == '1':
            return request.render("website_slides.slide_fullscreen", values)
        return request.render("website_slides.slide_main", values)

    @http.route('''/slides/slide/<model("slide.slide"):slide>/pdf_content''',
                type='http', auth="public", website=True, sitemap=False)
    def slide_get_pdf_content(self, slide):
        response = werkzeug.wrappers.Response()
        response.data = slide.datas and base64.b64decode(slide.datas) or b''
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

        status, headers, image_base64 = request.env['ir.http'].sudo().binary_content(
            model='slide.slide', id=slide.id, field=field,
            default_mimetype='image/png')
        if status == 301:
            return request.env['ir.http']._response_by_status(status, headers, image_base64)
        if status == 304:
            return werkzeug.wrappers.Response(status=304)

        if not image_base64:
            image_base64 = self._get_default_avatar()
            if not (width or height):
                width, height = tools.image_guess_size_from_field_name(field)

        image_base64 = tools.image_process(image_base64, size=(int(width), int(height)), crop=crop)

        content = base64.b64decode(image_base64)
        headers = http.set_safe_image_headers(headers, content)
        response = request.make_response(content, headers)
        response.status_code = status
        return response

    # SLIDE.SLIDE UTILS
    # --------------------------------------------------

    @http.route('/slides/slide/get_html_content', type="json", auth="public", website=True)
    def get_html_content(self, slide_id):
        fetch_res = self._fetch_slide(slide_id)
        if fetch_res.get('error'):
            return fetch_res
        return {
            'html_content': fetch_res['slide'].html_content
        }

    @http.route('/slides/slide/<model("slide.slide"):slide>/set_completed', website=True, type="http", auth="user")
    def slide_set_completed_and_redirect(self, slide, next_slide_id=None):
        self._set_completed_slide(slide)
        next_slide = None
        if next_slide_id:
            next_slide = self._fetch_slide(next_slide_id).get('slide', None)
        return werkzeug.utils.redirect("/slides/slide/%s" % (slug(next_slide) if next_slide else slug(slide)))

    @http.route('/slides/slide/set_completed', website=True, type="json", auth="public")
    def slide_set_completed(self, slide_id):
        if request.website.is_public_user():
            return {'error': 'public_user'}
        fetch_res = self._fetch_slide(slide_id)
        if fetch_res.get('error'):
            return fetch_res
        self._set_completed_slide(fetch_res['slide'])
        return {
            'channel_completion': fetch_res['slide'].channel_id.completion
        }

    @http.route('/slides/slide/like', type='json', auth="public", website=True)
    def slide_like(self, slide_id, upvote):
        if request.website.is_public_user():
            return {'error': 'public_user', 'error_signup_allowed': request.env['res.users'].sudo()._get_signup_invitation_scope() == 'b2c'}
        slide_partners = request.env['slide.slide.partner'].sudo().search([
            ('slide_id', '=', slide_id),
            ('partner_id', '=', request.env.user.partner_id.id)
        ])
        if (upvote and slide_partners.vote == 1) or (not upvote and slide_partners.vote == -1):
            return {'error': 'vote_done'}
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
        slide.invalidate_cache()
        return slide.read(['likes', 'dislikes', 'user_vote'])[0]

    @http.route('/slides/slide/archive', type='json', auth='user', website=True)
    def slide_archive(self, slide_id):
        """ This route allows channel publishers to archive slides.
        It has to be done in sudo mode since only website_publishers can write on slides in ACLs """
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
    def slide_send_share_email(self, slide_id, email, fullscreen=False):
        slide = request.env['slide.slide'].browse(int(slide_id))
        result = slide._send_share_email(email, fullscreen)
        return result

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

        slide_question = request.env['slide.question'].create({
            'sequence': sequence,
            'question': question,
            'slide_id': slide_id,
            'answer_ids': [(0, 0, {
                'sequence': answer['sequence'],
                'text_value': answer['text_value'],
                'is_correct': answer['is_correct'],
                'comment': answer['comment']
            }) for answer in answer_ids]
        })
        return request.env.ref('website_slides.lesson_content_quiz_question')._render({
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

        if slide.user_membership_id.sudo().completed:
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
            slide._action_set_quiz_done()
            slide.action_set_completed()
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
            'completed': slide.user_membership_id.sudo().completed,
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
        can_create = request.env['slide.slide'].check_access_rights('create', raise_exception=False)
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

        return werkzeug.utils.redirect("/slides/%s" % (slug(channel)))

    # --------------------------------------------------
    # SLIDE.UPLOAD
    # --------------------------------------------------

    @http.route(['/slides/prepare_preview'], type='json', auth='user', methods=['POST'], website=True)
    def prepare_preview(self, **data):
        Slide = request.env['slide.slide']
        unused, document_id = Slide._find_document_data_from_url(data['url'])
        preview = {}
        if not document_id:
            preview['error'] = _('Please enter valid youtube or google doc url')
            return preview
        existing_slide = Slide.search([('channel_id', '=', int(data['channel_id'])), ('document_id', '=', document_id)], limit=1)
        if existing_slide:
            preview['error'] = _('This video already exists in this channel on the following slide: %s', existing_slide.name)
            return preview
        values = Slide._parse_document_url(data['url'], only_preview_fields=True)
        if values.get('error'):
            preview['error'] = values['error']
            return preview
        return values

    @http.route(['/slides/add_slide'], type='json', auth='user', methods=['POST'], website=True)
    def create_slide(self, *args, **post):
        # check the size only when we upload a file.
        if post.get('datas'):
            file_size = len(post['datas']) * 3 / 4  # base64
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
            values['is_published'] = values.get('is_published', False) and can_publish
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
        if channel.channel_type == "training" and not slide.slide_type == "webpage":
            redirect_url = "/slides/%s" % (slug(channel))
        if slide.slide_type == 'webpage':
            redirect_url += "?enable_editor=1"
        return {
            'url': redirect_url,
            'channel_type': channel.channel_type,
            'slide_id': slide.id,
            'category_id': slide.category_id
        }

    def _get_valid_slide_post_values(self):
        return ['name', 'url', 'tag_ids', 'slide_type', 'channel_id', 'is_preview',
                'mime_type', 'datas', 'description', 'image_1920', 'is_published']

    @http.route(['/slides/tag/search_read'], type='json', auth='user', methods=['POST'], website=True)
    def slide_tag_search_read(self, fields, domain):
        can_create = request.env['slide.tag'].check_access_rights('create', raise_exception=False)
        return {
            'read_results': request.env['slide.tag'].search_read(domain, fields),
            'can_create': can_create,
        }

    # --------------------------------------------------
    # EMBED IN THIRD PARTY WEBSITES
    # --------------------------------------------------

    @http.route('/slides/embed/<int:slide_id>', type='http', auth='public', website=True, sitemap=False)
    def slides_embed(self, slide_id, page="1", **kw):
        # Note : don't use the 'model' in the route (use 'slide_id'), otherwise if public cannot access the embedded
        # slide, the error will be the website.403 page instead of the one of the website_slides.embed_slide.
        # Do not forget the rendering here will be displayed in the embedded iframe

        # determine if it is embedded from external web page
        referrer_url = request.httprequest.headers.get('Referer', '')
        base_url = request.env['ir.config_parameter'].sudo().get_param('web.base.url')
        is_embedded = referrer_url and not bool(base_url in referrer_url) or False
        # try accessing slide, and display to corresponding template
        try:
            slide = request.env['slide.slide'].browse(slide_id)
            if not slide.active:
                raise werkzeug.exceptions.NotFound()
            if is_embedded:
                request.env['slide.embed'].sudo()._add_embed_url(slide.id, referrer_url)
            values = self._get_slide_detail(slide)
            values['page'] = page
            values['is_embedded'] = is_embedded
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
        channel = self._get_channels(**kwargs)
        if channel:
            values['channel'] = channel
        return values

    def _get_channels(self, **kwargs):
        channels = []
        if kwargs.get('channel'):
            channels = kwargs['channel']
        elif kwargs.get('channel_id'):
            channels = request.env['slide.channel'].browse(int(kwargs['channel_id']))
        return channels

    def _prepare_user_slides_profile(self, user):
        courses = request.env['slide.channel.partner'].sudo().search([('partner_id', '=', user.partner_id.id)])
        courses_completed = courses.filtered(lambda c: c.completed)
        courses_ongoing = courses - courses_completed
        values = {
            'uid': request.env.user.id,
            'user': user,
            'main_object': user,
            'courses_completed': courses_completed,
            'courses_ongoing': courses_ongoing,
            'is_profile_page': True,
            'badge_category': 'slides',
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
            ('completed', '=', True)
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

## File: data\mail_activity_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mail_activity_data_access_request" model="mail.activity.type">
            <field name="name">Access Request</field>
            <field name="icon">fa-check-circle</field>
            <field name="sequence">50</field>
            <field name="res_model_id" ref="website_slides.model_slide_channel"/>
        </record>
    </data>
</odoo>

```

## File: data\mail_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
         <record id="slide_template_published" model="mail.template">
            <field name="name">Slide Published</field>
            <field name="model_id" ref="model_slide_slide"/>
            <field name="subject">New ${object.slide_type} published on ${object.channel_id.name}</field>
            <field name="body_html" type="html">
                <div style="margin: 0px; padding: 0px;">
                    <p style="margin: 0px; padding: 0px; font-size: 13px;">
                        Hello<br/><br/>
                        There is something new in the course <strong>${object.channel_id.name}</strong> you are following:<br/><br/>
                        <center><strong>${object.name}</strong></center>
                        % if object.image_1024
                        <div style="margin: 16px 8px 16px 8px; text-align: center;">
                            <a href="${object.website_url}">
                                <img alt="${object.name}" src="${ctx['base_url']}/web/image/slide.slide/${object.id}/image_1024" style="height:auto; width:150px; margin: 16px;"/>
                            </a>
                        </div>
                        % endif
                        <div style="margin: 16px 8px 16px 8px; text-align: center;">
                            <a href="${object.website_url}"
                                style="background-color: #875a7b; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px;">View content</a>
                        </div>
                        Enjoy this exclusive content!
                        % if user.signature
                            <br />
                            ${user.signature | safe}
                        % endif
                    </p>
                </div>
            </field>
            <field name="auto_delete" eval="True"/>
        </record>

        <record id="slide_template_shared" model="mail.template">
            <field name="name">Slide Shared</field>
            <field name="model_id" ref="model_slide_slide"/>
            <field name="subject">${user.name} shared a ${object.slide_type} with you!</field>
            <field name="email_from">${user.email_formatted | safe}</field>
            <field name="email_to">${ctx.get('email', '')}</field>
            <field name="body_html" type="html">
                <div style="margin: 0px; padding: 0px;">
                    <p style="margin: 0px; padding: 0px; font-size: 13px;">
                        Hello<br/><br/>
                        ${user.name} shared the ${object.slide_type} <strong>${object.name}</strong> with you!
                        <div style="margin: 16px 8px 16px 8px; text-align: center;">
                            <a href="${(object.website_url + '?fullscreen=1') if ctx['fullscreen'] else object.website_url | safe}">
                                <img alt="${object.name}" src="${ctx['base_url']}/web/image/slide.slide/${object.id}/image_1024" style="height:auto; width:150px; margin: 16px;"/>
                            </a>
                        </div>
                        <div style="margin: 16px 8px 16px 8px; text-align: center;">
                            <a href="${(object.website_url + '?fullscreen=1') if ctx['fullscreen'] else object.website_url | safe}"
                                style="background-color: #875a7b; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px;">View <strong>${object.name}</strong></a>
                        </div>
                        % if user.signature
                            <br />
                            ${user.signature | safe}
                        % endif
                    </p>
                </div>
            </field>
            <field name="auto_delete" eval="True"/>
        </record>

        <!-- Channel subtypes -->
        <record id="mt_channel_slide_published" model="mail.message.subtype">
            <field name="name">Presentation Published</field>
            <field name="res_model">slide.channel</field>
            <field name="default" eval="True"/>
            <field name="description">Presentation Published</field>
        </record>

        <!-- Slide channel invite feature -->
        <record id="mail_template_slide_channel_invite" model="mail.template">
            <field name="name">Channel: Invite by email</field>
            <field name="model_id" ref="model_slide_channel_partner" />
            <field name="subject">You have been invited to join ${object.channel_id.name}</field>
            <field name="use_default_to" eval="True"/>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px; font-size: 13px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        Hello<br/><br/>
        You have been invited to join a new course: ${object.channel_id.name}.
    </p>
</div>
            </field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="auto_delete" eval="True"/>
        </record>

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
                </td><td valign="middle" align="right">
                    <img t-att-src="'/logo.png?company=%s' % (company.id or 0)" style="padding: 0px; margin: 0px; height: 48px;" t-att-alt="'%s' % company.name"/>
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
            <t t-raw="message.body"/>
            <div style="margin: 32px 0px 32px 0px; text-align: center;">
                <a t-att-href="record.channel_id.website_url"
                    style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                    Click here to start the course
                </a>
            </div>
            <div style="margin: 0px; padding: 0px; font-size:13px;">
                Enjoy this exclusive content !
            </div>
            <div>&amp;nbsp;</div>
            <div t-if="signature" style="font-size: 13px;">
                <div t-raw="signature"/>
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
        <field name="description">Learn the basics of gardening !</field>
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
        <field name="name">Choose your wood !</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="enroll">public</field>
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
        <field name="datas" type="base64" file="website_slides/static/src/img/presentation.pdf"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/channel-training-default.jpg"/>
        <field name="slide_type">presentation</field>
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
            <field name="name">Document</field>
            <field name="data" type="base64" file="website_slides/static/src/img/document.png"/>
            <field name="slide_id" ref="slide_slide_demo_0_0"/>
        </record>
    <record id="slide_slide_demo_0_1" model="slide.slide">
        <field name="name">Home Gardening</field>
        <field name="sequence">2</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_gardening_1.jpg"/>
        <field name="slide_type">infographic</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">5</field>
        <field name="completion_time">1</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_colorful')), (4, ref('website_slides.slide_tag_demo_theory'))]"/>
        <field name="description">Interesting information about home gardening. Keep it close !</field>
    </record>
    <record id="slide_slide_demo_0_2" model="slide.slide">
        <field name="name">Mighty Carrots</field>
        <field name="sequence">3</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_gardening_2.jpg"/>
        <field name="slide_type">infographic</field>
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
        <field name="datas" type="base64" file="website_slides/static/src/img/presentation.pdf"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_thumb_l0JZ25VvbwE.jpg"/>
        <field name="slide_type">document</field>
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
        <field name="slide_type">quiz</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="True"/>
        <field name="public_views">0</field>
        <field name="completion_time">1</field>
        <field name="description">Show your newly mastered knowledge !</field>
    </record>
        <record id="slide_slide_demo_0_4_question_0" model="slide.question">
            <field name="question">What is a strawberry ?</field>
            <field name="sequence">1</field>
            <field name="slide_id" ref="slide_slide_demo_0_4"/>
        </record>
        <record id="slide_slide_demo_0_4_question_0_0" model="slide.answer">
            <field name="text_value">A fruit</field>
            <field name="sequence">1</field>
            <field name="is_correct" eval="True"/>
            <field name="comment">Correct ! A strawberry is a fruit because it's the product of a tree.</field>
            <field name="question_id" ref="slide_slide_demo_0_4_question_0"/>
        </record>
        <record id="slide_slide_demo_0_4_question_0_1" model="slide.answer">
            <field name="text_value">A vegetable</field>
            <field name="sequence">2</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect ! A strawberry is not a vegetable.</field>
            <field name="question_id" ref="slide_slide_demo_0_4_question_0"/>
        </record>
        <record id="slide_slide_demo_0_4_question_0_2" model="slide.answer">
            <field name="text_value">A table</field>
            <field name="sequence">3</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect ! A table is a piece of furniture.</field>
            <field name="question_id" ref="slide_slide_demo_0_4_question_0"/>
        </record>
        <record id="slide_slide_demo_0_4_question_1" model="slide.question">
            <field name="question">What is the best tool to dig a hole for your plants ?</field>
            <field name="sequence">2</field>
            <field name="slide_id" ref="slide_slide_demo_0_4"/>
        </record>
        <record id="slide_slide_demo_0_4_question_1_0" model="slide.answer">
            <field name="text_value">A shovel</field>
            <field name="sequence">1</field>
            <field name="is_correct" eval="True"/>
            <field name="comment">Correct ! A shovel is the perfect tool to dig a hole.</field>
            <field name="question_id" ref="slide_slide_demo_0_4_question_1"/>
        </record>
        <record id="slide_slide_demo_0_4_question_1_1" model="slide.answer">
            <field name="text_value">A spoon</field>
            <field name="sequence">2</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect ! Good luck digging a hole with a spoon...</field>
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
        <field name="name">Tree Infographic</field>
        <field name="sequence">1</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_tree_infographic_1.jpg"/>
        <field name="slide_type">infographic</field>
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
        <field name="name">Interesting Tree Facts</field>
        <field name="sequence">2</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_tree_infographic_2.jpg"/>
        <field name="slide_type">infographic</field>
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
        <field name="slide_type">infographic</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">10</field>
        <field name="completion_time">1</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_colorful')), (4, ref('website_slides.slide_tag_demo_theory'))]"/>
        <field name="description">Just some basics Energy Efficiency Facts.</field>
    </record>
        <record id="slide_slide_demo_1_2_link_0" model="slide.slide.link">
            <field name="name">Energy Efficient Link 1</field>
            <field name="link">http://www.example.com</field>
            <field name="slide_id" ref="slide_slide_demo_1_2"/>
        </record>
        <record id="slide_slide_demo_1_2_link_1" model="slide.slide.link">
            <field name="name">Energy Efficient Link 2</field>
            <field name="link">http://www.example.com</field>
            <field name="slide_id" ref="slide_slide_demo_1_2"/>
        </record>
        <!--RESOURCE-->
        <record id="slide_slide_demo_1_2_resource_0" model="slide.slide.resource">
            <field name="name">Presentation</field>
            <field name="data" type="base64" file="website_slides/static/src/img/presentation.pdf"/>
            <field name="slide_id" ref="slide_slide_demo_1_2"/>
        </record>
    <record id="slide_slide_demo_1_3" model="slide.slide">
        <field name="name">How to plant a potted tree</field>
        <field name="sequence">5</field>
        <field name="url">https://www.youtube.com/watch?v=QYmgrw0PgLU</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_thumb_QYmgrw0PgLU.jpg"/>
        <field name="document_id">QYmgrw0PgLU</field>
        <field name="slide_type">video</field>
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
        <field name="slide_type">webpage</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="html_content" type="html">
<section class="s_cover parallax bg-black-50 pt16 pb16" data-scroll-background-ratio="0" style="background-image: none;" data-snippet="s_cover">
    <span class="s_parallax_bg oe_img_bg" style="background-image: url('/website_slides/static/src/img/slide_demo_tree_img_1.jpg'); background-position: 50% 0;"></span>
    <div class="o_we_bg_filter bg-black-50"/>
    <div class="container">
        <div class="row s_nb_column_fixed">
            <div class="col-lg-12">
                <h1 class="o_default_snippet_text" style="font-size: 62px; text-align: center;">Catchy Headline</h1>
                <p class="lead o_default_snippet_text" style="text-align: center;">Write one or two paragraphs describing your product, services or a specific feature.<br/> To be successful your content needs to be useful to your readers.</p>
                <p>
                    <a href="/contactus" class="btn btn-primary rounded-circle o_default_snippet_text">Contact us</a>
                </p>
            </div>
        </div>
    </div>
</section>
<section class="s_text_image pt32 pb32" data-snippet="s_text_image">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-6 pt16 pb16">
                <img src="/website_slides/static/src/img/slide_demo_tree_img_1.jpg" class="img img-fluid mx-auto" alt="Odoo • Image and Text"/>
            </div>
            <div class="col-lg-6 pt16 pb16">
                <h2 class="o_default_snippet_text">Section Subtitle</h2>
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
        <field name="description">We had a little chat with Harry Potted, sure he had interesting things to say !</field>
    </record>
        <record id="slide_slide_demo_1_4_link_0" model="slide.slide.link">
            <field name="name">Know More Link 1</field>
            <field name="link">http://www.example.com</field>
            <field name="slide_id" ref="slide_slide_demo_1_4"/>
        </record>
        <record id="slide_slide_demo_1_4_link_1" model="slide.slide.link">
            <field name="name">Know More Link 2</field>
            <field name="link">http://www.example.com</field>
            <field name="slide_id" ref="slide_slide_demo_1_4"/>
        </record>
        <record id="slide_slide_demo_1_4_question_0" model="slide.question">
            <field name="question">Do you think Harry Potted has a good name ?</field>
            <field name="sequence">1</field>
            <field name="slide_id" ref="slide_slide_demo_1_4"/>
        </record>
        <record id="slide_slide_demo_1_4_question_0_0" model="slide.answer">
            <field name="text_value">Yes</field>
            <field name="sequence">1</field>
            <field name="is_correct" eval="True"/>
            <field name="comment">Correct !</field>
            <field name="question_id" ref="slide_slide_demo_1_4_question_0"/>
        </record>
        <record id="slide_slide_demo_1_4_question_0_1" model="slide.answer">
            <field name="text_value">No</field>
            <field name="sequence">2</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect !</field>
            <field name="question_id" ref="slide_slide_demo_1_4_question_0"/>
        </record>
        <record id="slide_slide_demo_1_4_question_1" model="slide.question">
            <field name="question">Did you read the whole article ?</field>
            <field name="sequence">2</field>
            <field name="slide_id" ref="slide_slide_demo_1_4"/>
        </record>
        <record id="slide_slide_demo_1_4_question_1_0" model="slide.answer">
            <field name="text_value">Yes</field>
            <field name="sequence">1</field>
            <field name="is_correct" eval="True"/>
            <field name="comment">Correct ! Congratulations you have time to loose</field>
            <field name="question_id" ref="slide_slide_demo_1_4_question_1"/>
        </record>
        <record id="slide_slide_demo_1_4_question_1_1" model="slide.answer">
            <field name="text_value">No</field>
            <field name="sequence">2</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect ! You really should read it.</field>
            <field name="question_id" ref="slide_slide_demo_1_4_question_1"/>
        </record>
        <record id="slide_slide_demo_1_4_question_1_2" model="slide.answer">
            <field name="text_value">What was the question again ?</field>
            <field name="sequence">3</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect ! Seriously ?</field>
            <field name="question_id" ref="slide_slide_demo_1_4_question_1"/>
        </record>
    <record id="slide_slide_demo_1_5" model="slide.slide">
        <field name="name">3 Main Methodologies</field>
        <field name="sequence">6</field>
        <field name="datas" type="base64" file="website_slides/static/src/img/presentation.pdf"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/channel-training-default.jpg"/>
        <field name="slide_type">presentation</field>
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
        <field name="document_id">l0JZ25VvbwE</field>
        <field name="slide_type">video</field>
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
        <field name="datas" type="base64" file="website_slides/static/src/img/presentation.pdf"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_tree_img_2.jpg"/>
        <field name="slide_type">presentation</field>
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
        <record id="slide_slide_demo_2_0_link_0" model="slide.slide.link">
            <field name="name">Trees Classification Link</field>
            <field name="link">http://www.example.com</field>
            <field name="slide_id" ref="slide_slide_demo_2_0"/>
        </record>
        <record id="slide_slide_demo_2_0_link_1" model="slide.slide.link">
            <field name="name">Main types of trees Link</field>
            <field name="link">http://www.example.com</field>
            <field name="slide_id" ref="slide_slide_demo_2_0"/>
        </record>
        <!--RESOURCE-->
        <record id="slide_slide_demo_2_0_resource_0" model="slide.slide.resource">
            <field name="name">Tree image</field>
            <field name="data" type="base64" file="website_slides/static/src/img/slide_demo_tree_img_2.jpg"/>
            <field name="slide_id" ref="slide_slide_demo_2_0"/>
        </record>
        <!-- QUIZZ -->
        <record id="slide_slide_demo_2_0_question_0" model="slide.question">
            <field name="question">Do you make beams out of lemon trees ?</field>
            <field name="sequence">1</field>
            <field name="slide_id" ref="slide_slide_demo_2_0"/>
        </record>
        <record id="slide_slide_demo_2_0_question_0_0" model="slide.answer">
            <field name="text_value">Yes</field>
            <field name="sequence">1</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect !</field>
            <field name="question_id" ref="slide_slide_demo_2_0_question_0"/>
        </record>
        <record id="slide_slide_demo_2_0_question_0_1" model="slide.answer">
            <field name="text_value">No</field>
            <field name="sequence">2</field>
            <field name="is_correct" eval="True"/>
            <field name="comment">Correct !</field>
            <field name="question_id" ref="slide_slide_demo_2_0_question_0"/>
        </record>
        <record id="slide_slide_demo_2_0_question_1" model="slide.question">
            <field name="question">Do you make lemons out of beams ?</field>
            <field name="sequence">2</field>
            <field name="slide_id" ref="slide_slide_demo_2_0"/>
        </record>
        <record id="slide_slide_demo_2_0_question_1_0" model="slide.answer">
            <field name="text_value">Yes</field>
            <field name="sequence">1</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect !</field>
            <field name="question_id" ref="slide_slide_demo_2_0_question_1"/>
        </record>
        <record id="slide_slide_demo_2_0_question_1_1" model="slide.answer">
            <field name="text_value">No</field>
            <field name="sequence">2</field>
            <field name="is_correct" eval="True"/>
            <field name="comment">Correct !</field>
            <field name="question_id" ref="slide_slide_demo_2_0_question_1"/>
        </record>
        <record id="slide_slide_demo_2_0_question_1_2" model="slide.answer">
            <field name="text_value">And also bananas</field>
            <field name="sequence">3</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect ! of course not ...</field>
            <field name="question_id" ref="slide_slide_demo_2_0_question_1"/>
        </record>
    <record id="slide_slide_demo_2_1" model="slide.slide">
        <field name="name">A Mighty Forest from Ages</field>
        <field name="sequence">2</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_tree_img_3.jpg"/>
        <field name="slide_type">webpage</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_2_gard2"/>
        <field name="html_content" type="html">
<section class="s_cover parallax bg-black-50 pt16 pb16" data-scroll-background-ratio="0" style="background-image: none;" data-snippet="s_cover">
    <span class="s_parallax_bg oe_img_bg" style="background-image: url('/website_slides/static/src/img/slide_demo_tree_img_3.jpg'); background-position: 50% 0;"></span>
    <div class="o_we_bg_filter bg-black-50"/>
    <div class="container">
        <div class="row s_nb_column_fixed">
            <div class="col-lg-12">
                <h1 class="o_default_snippet_text" style="font-size: 62px; text-align: center;">Catchy Headline</h1>
                <p class="lead o_default_snippet_text" style="text-align: center;">Write one or two paragraphs describing your product, services or a specific feature.<br/> To be successful your content needs to be useful to your readers.</p>
                <p class="o_default_snippet_text">
                    <a href="/contactus" class="btn btn-primary rounded-circle">Contact us</a>
                </p>
            </div>
        </div>
    </div>
</section>
<section class="s_text_image pt32 pb32" data-snippet="s_text_image">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-6 pt16 pb16">
                <img src="/website_slides/static/src/img/slide_demo_tree_img_3.jpg" class="img img-fluid mx-auto" alt="Odoo • Image and Text"/>
            </div>
            <div class="col-lg-6 pt16 pb16">
                <h2 class="o_default_snippet_text">Section Subtitle</h2>
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
        <field name="name">Tree planting in hanging bottles on wall</field>
        <field name="sequence">3</field>
        <field name="url">https://www.youtube.com/watch?v=ebBez6bcSEc</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_thumb_ebBez6bcSEc.jpg"/>
        <field name="document_id">ebBez6bcSEc</field>
        <field name="slide_type">video</field>
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
        <field name="datas" type="base64" file="website_slides/static/src/img/presentation.pdf"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/channel-documentation-default.jpg"/>
        <field name="slide_type">presentation</field>
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


    <!-- CHANNEL 3: Choose your wood !           -->
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
        <field name="slide_type">infographic</field>
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
        <field name="document_id">PYr1rK8pS30</field>
        <field name="slide_type">video</field>
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
        <field name="name">Introduction</field>
        <field name="is_category" eval="True"/>
        <field name="channel_id" ref="website_slides.slide_channel_demo_4_furn1"/>
        <field name="sequence">0</field>
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
        <field name="name">Hand on !</field>
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
        <field name="datas" type="base64" file="website_slides/static/src/img/presentation.pdf"/>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/channel-training-default.jpg"/>
        <field name="slide_type">presentation</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_5_furn2"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="True"/>
        <field name="public_views">10</field>
        <field name="completion_time">2.5</field>
        <field name="tag_ids" eval="[(4, ref('website_slides.slide_tag_demo_tools'))]"/>
        <field name="description">Tools you will need to complete this course.</field>
    </record>
        <record id="slide_slide_demo_5_0_link_0" model="slide.slide.link">
            <field name="name">Example Link 1</field>
            <field name="link">http://www.example.com</field>
            <field name="slide_id" ref="slide_slide_demo_5_0"/>
        </record>
        <record id="slide_slide_demo_5_0_link_1" model="slide.slide.link">
            <field name="name">Example Link 2</field>
            <field name="link">http://www.example.com</field>
            <field name="slide_id" ref="slide_slide_demo_5_0"/>
        </record>
    <record id="slide_slide_demo_5_1" model="slide.slide">
        <field name="name">How to find quality wood</field>
        <field name="sequence">2</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_thumb_5WMqwTnZ-qs.jpg"/>
        <field name="url">https://www.youtube.com/watch?v=5WMqwTnZ-qs</field>
        <field name="document_id">5WMqwTnZ-qs</field>
        <field name="slide_type">video</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_5_furn2"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">0</field>
        <field name="completion_time">3</field>
        <field name="description">Learn to identify quality wood in order to create solid furnitures.</field>
    </record>
     <record id="slide_slide_demo_5_2" model="slide.slide">
        <field name="name">How to create your own piece of furniture</field>
        <field name="sequence">4</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_thumb_ptjeDDoURL8.jpg"/>
        <field name="url">https://www.youtube.com/watch?v=ptjeDDoURL8</field>
        <field name="document_id">ptjeDDoURL8</field>
        <field name="slide_type">video</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_5_furn2"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="True"/>
        <field name="public_views">5</field>
        <field name="completion_time">3</field>
        <field name="description">From a piece of wood to a fully functional furniture, step by step.</field>
    </record>
    <record id="slide_slide_demo_5_3" model="slide.slide">
        <field name="name">Test your knowledge !</field>
        <field name="sequence">6</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_owl.jpg"/>
        <field name="slide_type">quiz</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_5_furn2"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="public_views">0</field>
        <field name="completion_time">0.5</field>
        <field name="description">Test your knowledge !</field>
    </record>
        <record id="slide_slide_demo_5_3_question_0" model="slide.question">
            <field name="question">Do you want to reply correctly ?</field>
            <field name="sequence">1</field>
            <field name="slide_id" ref="slide_slide_demo_5_3"/>
        </record>
        <record id="slide_slide_demo_5_3_question_0_0" model="slide.answer">
            <field name="text_value">Yes</field>
            <field name="sequence">1</field>
            <field name="is_correct" eval="True"/>
            <field name="comment">Correct ! You did it !</field>
            <field name="question_id" ref="slide_slide_demo_5_3_question_0"/>
        </record>
        <record id="slide_slide_demo_5_3_question_0_1" model="slide.answer">
            <field name="text_value">No</field>
            <field name="sequence">2</field>
            <field name="is_correct" eval="False"/>
            <field name="comment">Incorrect ! You better think twice...</field>
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
        <field name="body" type="html"><div>I fear beginners could be lost... Isn't it a bit harsh for a "basics" course ?</div></field>
    </record>
    <record id="rating_channel_0_admin" model="rating.rating">
        <field name="res_model_id" ref="website_slides.model_slide_channel"/>
        <field name="res_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="partner_id" ref="base.partner_admin"/>
        <field name="consumed" eval="True"/>
        <field name="feedback">I fear beginners could be lost... Isn't it a bit harsh for a "basics" course ?</field>
        <field name="rating">3</field>
        <field name="message_id" ref="website_slides.message_channel_0_admin"/>
    </record>
    <record id="message_channel_0_demo" model="mail.message">
        <field name="model">slide.channel</field>
        <field name="res_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="message_type">comment</field>
        <field name="author_id" ref="base.partner_demo"/>
        <field name="body" type="html"><div>Back to basics and interesting. Just WOW !</div></field>
    </record>
    <record id="rating_channel_0_demo" model="rating.rating">
        <field name="res_model_id" ref="website_slides.model_slide_channel"/>
        <field name="res_id" ref="website_slides.slide_channel_demo_0_gard_0"/>
        <field name="partner_id" ref="base.partner_demo"/>
        <field name="consumed" eval="True"/>
        <field name="rating">5</field>
        <field name="feedback">Back to basics and interesting. Just WOW !</field>
        <field name="message_id" ref="website_slides.message_channel_0_demo"/>
    </record>

    <function model="slide.channel" name="message_subscribe"
            eval="[ref('website_slides.slide_channel_demo_0_gard_0')], [ref('base.partner_admin'), ref('base.partner_demo'), ref('base.partner_demo_portal')]"/>

    <!-- CHANNEL 1: Taking care of Trees -->
    <!-- ================================================== -->
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
        <field name="body" type="html"><div>Interesting !</div></field>
    </record>
    <record id="rating_channel_1_demo" model="rating.rating">
        <field name="res_model_id" ref="website_slides.model_slide_channel"/>
        <field name="res_id" ref="website_slides.slide_channel_demo_1_gard1"/>
        <field name="partner_id" ref="base.partner_demo"/>
        <field name="consumed" eval="True"/>
        <field name="rating">4</field>
        <field name="feedback">Interesting !</field>
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
            eval="[ref('website_slides.slide_channel_demo_1_gard1')], [ref('base.partner_admin'), ref('base.partner_demo'), ref('base.partner_demo_portal')]"/>


    <!-- CHANNEL 2: Trees, Wood and Garden -->
    <!-- ================================================== -->
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
    <record id="slide_channel_2_partner_demo" model="slide.channel.partner">
        <field name="channel_id" ref="website_slides.slide_channel_demo_2_gard2"/>
        <field name="partner_id" ref="base.partner_demo"/>
    </record>
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

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Http(models.AbstractModel):
    _inherit = 'ir.http'

    def binary_content(self, xmlid=None, model='ir.attachment', id=None, field='datas',
                       unique=False, filename=None, filename_field='name', download=False,
                       mimetype=None, default_mimetype='application/octet-stream',
                       access_token=None):
        obj = None
        if xmlid:
            obj = self._xmlid_to_obj(self.env, xmlid)
            if obj and obj._name != 'slide.slide':
                obj = None
        elif id and model == 'slide.slide':
            obj = self.env[model].browse(int(id))
        if obj:
            obj.check_access_rights('read')
            obj.check_access_rule('read')
        return super(Http, self).binary_content(
            xmlid=xmlid, model=model, id=id, field=field, unique=unique, filename=filename,
            filename_field=filename_field, download=download, mimetype=mimetype,
            default_mimetype=default_mimetype, access_token=access_token)

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


class ResPartner(models.Model):
    _inherit = 'res.partner'

    slide_channel_ids = fields.Many2many(
        'slide.channel', 'slide_channel_partner', 'partner_id', 'channel_id',
        string='eLearning Courses', copy=False)
    slide_channel_count = fields.Integer('Course Count', compute='_compute_slide_channel_count')
    slide_channel_company_count = fields.Integer('Company Course Count', compute='_compute_slide_channel_company_count')

    @api.depends('is_company')
    def _compute_slide_channel_count(self):
        read_group_res = self.env['slide.channel.partner'].sudo().read_group(
            [('partner_id', 'in', self.ids)],
            ['partner_id'], 'partner_id'
        )
        data = dict((res['partner_id'][0], res['partner_id_count']) for res in read_group_res)
        for partner in self:
            partner.slide_channel_count = data.get(partner.id, 0)

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
        action = self.env["ir.actions.actions"]._for_xml_id("website_slides.slide_channel_action_overview")
        action['name'] = _('Followed Courses')
        action['domain'] = ['|', ('partner_ids', 'in', self.ids), ('partner_ids', 'in', self.child_ids.ids)]
        return action

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _


class Users(models.Model):
    _inherit = 'res.users'

    @api.model
    def create(self, values):
        """ Trigger automatic subscription based on user groups """
        user = super(Users, self).create(values)
        self.env['slide.channel'].sudo().search([('enroll_group_ids', 'in', user.groups_id.ids)])._action_add_members(user.partner_id)
        return user

    def write(self, vals):
        """ Trigger automatic subscription based on updated user groups """
        res = super(Users, self).write(vals)
        if vals.get('groups_id'):
            added_group_ids = [command[1] for command in vals['groups_id'] if command[0] == 4]
            added_group_ids += [id for command in vals['groups_id'] if command[0] == 6 for id in command[2]]
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

import uuid
from collections import defaultdict

from dateutil.relativedelta import relativedelta
import ast

from odoo import api, fields, models, tools, _
from odoo.addons.http_routing.models.ir_http import slug
from odoo.exceptions import AccessError
from odoo.osv import expression


class ChannelUsersRelation(models.Model):
    _name = 'slide.channel.partner'
    _description = 'Channel / Partners (Members)'
    _table = 'slide_channel_partner'

    channel_id = fields.Many2one('slide.channel', index=True, required=True, ondelete='cascade')
    completed = fields.Boolean('Is Completed', help='Channel validated, even if slides / lessons are added once done.')
    completion = fields.Integer('% Completed Slides')
    completed_slides_count = fields.Integer('# Completed Slides')
    partner_id = fields.Many2one('res.partner', index=True, required=True, ondelete='cascade')
    partner_email = fields.Char(related='partner_id.email', readonly=True)

    def _recompute_completion(self):
        read_group_res = self.env['slide.slide.partner'].sudo().read_group(
            ['&', '&', ('channel_id', 'in', self.mapped('channel_id').ids),
             ('partner_id', 'in', self.mapped('partner_id').ids),
             ('completed', '=', True),
             ('slide_id.is_published', '=', True),
             ('slide_id.active', '=', True)],
            ['channel_id', 'partner_id'],
            groupby=['channel_id', 'partner_id'], lazy=False)
        mapped_data = dict()
        for item in read_group_res:
            mapped_data.setdefault(item['channel_id'][0], dict())
            mapped_data[item['channel_id'][0]][item['partner_id'][0]] = item['__count']

        partner_karma = dict.fromkeys(self.mapped('partner_id').ids, 0)
        for record in self:
            record.completed_slides_count = mapped_data.get(record.channel_id.id, dict()).get(record.partner_id.id, 0)
            record.completion = 100.0 if record.completed else round(100.0 * record.completed_slides_count / (record.channel_id.total_slides or 1))
            if not record.completed and record.channel_id.active and record.completed_slides_count >= record.channel_id.total_slides:
                record.completed = True
                partner_karma[record.partner_id.id] += record.channel_id.karma_gen_channel_finish

        partner_karma = {partner_id: karma_to_add
                         for partner_id, karma_to_add in partner_karma.items() if karma_to_add > 0}

        if partner_karma:
            users = self.env['res.users'].sudo().search([('partner_id', 'in', list(partner_karma.keys()))])
            for user in users:
                users.add_karma(partner_karma[user.partner_id.id])

    def unlink(self):
        """
        Override unlink method :
        Remove attendee from a channel, then also remove slide.slide.partner related to.
        """
        removed_slide_partner_domain = []
        for channel_partner in self:
            # find all slide link to the channel and the partner
            removed_slide_partner_domain = expression.OR([
                removed_slide_partner_domain,
                [('partner_id', '=', channel_partner.partner_id.id),
                 ('slide_id', 'in', channel_partner.channel_id.slide_ids.ids)]
            ])
        if removed_slide_partner_domain:
            self.env['slide.slide.partner'].search(removed_slide_partner_domain).unlink()
        return super(ChannelUsersRelation, self).unlink()


class Channel(models.Model):
    """ A channel is a container of slides. """
    _name = 'slide.channel'
    _description = 'Course'
    _inherit = [
        'mail.thread', 'rating.mixin',
        'mail.activity.mixin',
        'image.mixin',
        'website.seo.metadata', 'website.published.multi.mixin']
    _order = 'sequence, id'

    def _default_access_token(self):
        return str(uuid.uuid4())

    def _get_default_enroll_msg(self):
        return _('Contact Responsible')

    # description
    name = fields.Char('Name', translate=True, required=True)
    active = fields.Boolean(default=True, tracking=100)
    description = fields.Text('Description', translate=True, help="The description that is displayed on top of the course page, just below the title")
    description_short = fields.Text('Short Description', translate=True, help="The description that is displayed on the course card")
    description_html = fields.Html('Detailed Description', translate=tools.html_translate, sanitize_attributes=False, sanitize_form=False)
    channel_type = fields.Selection([
        ('training', 'Training'), ('documentation', 'Documentation')],
        string="Course type", default="training", required=True)
    sequence = fields.Integer(default=10, help='Display order')
    user_id = fields.Many2one('res.users', string='Responsible', default=lambda self: self.env.uid)
    color = fields.Integer('Color Index', default=0, help='Used to decorate kanban view')
    tag_ids = fields.Many2many(
        'slide.channel.tag', 'slide_channel_tag_rel', 'channel_id', 'tag_id',
        string='Tags', help='Used to categorize and filter displayed channels/courses')
    # slides: promote, statistics
    slide_ids = fields.One2many('slide.slide', 'channel_id', string="Slides and categories")
    slide_content_ids = fields.One2many('slide.slide', string='Slides', compute="_compute_category_and_slide_ids")
    slide_category_ids = fields.One2many('slide.slide', string='Categories', compute="_compute_category_and_slide_ids")
    slide_last_update = fields.Date('Last Update', compute='_compute_slide_last_update', store=True)
    slide_partner_ids = fields.One2many(
        'slide.slide.partner', 'channel_id', string="Slide User Data",
        copy=False, groups='website_slides.group_website_slides_officer')
    promote_strategy = fields.Selection([
        ('latest', 'Latest Published'),
        ('most_voted', 'Most Voted'),
        ('most_viewed', 'Most Viewed'),
        ('specific', 'Specific'),
        ('none', 'None')],
        string="Promoted Content", default='latest', required=False,
        help='Depending the promote strategy, a slide will appear on the top of the course\'s page :\n'
             ' * Latest Published : the slide created last.\n'
             ' * Most Voted : the slide which has to most votes.\n'
             ' * Most Viewed ; the slide which has been viewed the most.\n'
             ' * Specific : You choose the slide to appear.\n'
             ' * None : No slides will be shown.\n')
    promoted_slide_id = fields.Many2one('slide.slide', string='Promoted Slide')
    access_token = fields.Char("Security Token", copy=False, default=_default_access_token)
    nbr_presentation = fields.Integer('Presentations', compute='_compute_slides_statistics', store=True)
    nbr_document = fields.Integer('Documents', compute='_compute_slides_statistics', store=True)
    nbr_video = fields.Integer('Videos', compute='_compute_slides_statistics', store=True)
    nbr_infographic = fields.Integer('Infographics', compute='_compute_slides_statistics', store=True)
    nbr_webpage = fields.Integer("Webpages", compute='_compute_slides_statistics', store=True)
    nbr_quiz = fields.Integer("Number of Quizs", compute='_compute_slides_statistics', store=True)
    total_slides = fields.Integer('Content', compute='_compute_slides_statistics', store=True)
    total_views = fields.Integer('Visits', compute='_compute_slides_statistics', store=True)
    total_votes = fields.Integer('Votes', compute='_compute_slides_statistics', store=True)
    total_time = fields.Float('Duration', compute='_compute_slides_statistics', digits=(10, 2), store=True)
    rating_avg_stars = fields.Float("Rating Average (Stars)", compute='_compute_rating_stats', digits=(16, 1), compute_sudo=True)
    # configuration
    allow_comment = fields.Boolean(
        "Allow rating on Course", default=True,
        help="If checked it allows members to either:\n"
             " * like content and post comments on documentation course;\n"
             " * post comment and review on training course;")
    publish_template_id = fields.Many2one(
        'mail.template', string='New Content Email',
        help="Email template to send slide publication through email",
        default=lambda self: self.env['ir.model.data'].xmlid_to_res_id('website_slides.slide_template_published'))
    share_template_id = fields.Many2one(
        'mail.template', string='Share Template',
        help="Email template used when sharing a slide",
        default=lambda self: self.env['ir.model.data'].xmlid_to_res_id('website_slides.slide_template_shared'))
    enroll = fields.Selection([
        ('public', 'Public'), ('invite', 'On Invitation')],
        default='public', string='Enroll Policy', required=True,
        help='Condition to enroll: everyone, on invite, on payment (sale bridge).')
    enroll_msg = fields.Html(
        'Enroll Message', help="Message explaining the enroll process",
        default=_get_default_enroll_msg, translate=tools.html_translate, sanitize_attributes=False)
    enroll_group_ids = fields.Many2many('res.groups', string='Auto Enroll Groups', help="Members of those groups are automatically added as members of the channel.")
    visibility = fields.Selection([
        ('public', 'Public'), ('members', 'Members Only')],
        default='public', string='Visibility', required=True,
        help='Applied directly as ACLs. Allow to hide channels and their content for non members.')
    partner_ids = fields.Many2many(
        'res.partner', 'slide_channel_partner', 'channel_id', 'partner_id',
        string='Members', help="All members of the channel.", context={'active_test': False}, copy=False, depends=['channel_partner_ids'])
    members_count = fields.Integer('Attendees count', compute='_compute_members_count')
    members_done_count = fields.Integer('Attendees Done Count', compute='_compute_members_done_count')
    has_requested_access = fields.Boolean(string='Access Requested', compute='_compute_has_requested_access', compute_sudo=False)
    is_member = fields.Boolean(string='Is Member', compute='_compute_is_member', compute_sudo=False)
    channel_partner_ids = fields.One2many('slide.channel.partner', 'channel_id', string='Members Information', groups='website_slides.group_website_slides_officer', depends=['partner_ids'])
    upload_group_ids = fields.Many2many(
        'res.groups', 'rel_upload_groups', 'channel_id', 'group_id', string='Upload Groups',
        help="Group of users allowed to publish contents on a documentation course.")
    # not stored access fields, depending on each user
    completed = fields.Boolean('Done', compute='_compute_user_statistics', compute_sudo=False)
    completion = fields.Integer('Completion', compute='_compute_user_statistics', compute_sudo=False)
    can_upload = fields.Boolean('Can Upload', compute='_compute_can_upload', compute_sudo=False)
    partner_has_new_content = fields.Boolean(compute='_compute_partner_has_new_content', compute_sudo=False)
    # karma generation
    karma_gen_slide_vote = fields.Integer(string='Lesson voted', default=1)
    karma_gen_channel_rank = fields.Integer(string='Course ranked', default=5)
    karma_gen_channel_finish = fields.Integer(string='Course finished', default=10)
    # Karma based actions
    karma_review = fields.Integer('Add Review', default=10, help="Karma needed to add a review on the course")
    karma_slide_comment = fields.Integer('Add Comment', default=3, help="Karma needed to add a comment on a slide of this course")
    karma_slide_vote = fields.Integer('Vote', default=3, help="Karma needed to like/dislike a slide of this course.")
    can_review = fields.Boolean('Can Review', compute='_compute_action_rights', compute_sudo=False)
    can_comment = fields.Boolean('Can Comment', compute='_compute_action_rights', compute_sudo=False)
    can_vote = fields.Boolean('Can Vote', compute='_compute_action_rights', compute_sudo=False)

    @api.depends('slide_ids.is_published')
    def _compute_slide_last_update(self):
        for record in self:
            record.slide_last_update = fields.Date.today()

    @api.depends('channel_partner_ids.channel_id')
    def _compute_members_count(self):
        read_group_res = self.env['slide.channel.partner'].sudo().read_group([('channel_id', 'in', self.ids)], ['channel_id'], 'channel_id')
        data = dict((res['channel_id'][0], res['channel_id_count']) for res in read_group_res)
        for channel in self:
            channel.members_count = data.get(channel.id, 0)

    @api.depends('channel_partner_ids.channel_id', 'channel_partner_ids.completed')
    def _compute_members_done_count(self):
        read_group_res = self.env['slide.channel.partner'].sudo().read_group(['&', ('channel_id', 'in', self.ids), ('completed', '=', True)], ['channel_id'], 'channel_id')
        data = dict((res['channel_id'][0], res['channel_id_count']) for res in read_group_res)
        for channel in self:
            channel.members_done_count = data.get(channel.id, 0)

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

    @api.depends('channel_partner_ids.partner_id')
    @api.depends_context('uid')
    @api.model
    def _compute_is_member(self):
        channel_partners = self.env['slide.channel.partner'].sudo().search([
            ('channel_id', 'in', self.ids),
        ])
        result = dict()
        for cp in channel_partners:
            result.setdefault(cp.channel_id.id, []).append(cp.partner_id.id)
        for channel in self:
            channel.is_member = channel.is_member = self.env.user.partner_id.id in result.get(channel.id, [])

    @api.depends('slide_ids.is_category')
    def _compute_category_and_slide_ids(self):
        for channel in self:
            channel.slide_category_ids = channel.slide_ids.filtered(lambda slide: slide.is_category)
            channel.slide_content_ids = channel.slide_ids - channel.slide_category_ids

    @api.depends('slide_ids.slide_type', 'slide_ids.is_published', 'slide_ids.completion_time',
                 'slide_ids.likes', 'slide_ids.dislikes', 'slide_ids.total_views', 'slide_ids.is_category', 'slide_ids.active')
    def _compute_slides_statistics(self):
        default_vals = dict(total_views=0, total_votes=0, total_time=0, total_slides=0)
        keys = ['nbr_%s' % slide_type for slide_type in self.env['slide.slide']._fields['slide_type'].get_values(self.env)]
        default_vals.update(dict((key, 0) for key in keys))

        result = dict((cid, dict(default_vals)) for cid in self.ids)
        read_group_res = self.env['slide.slide'].read_group(
            [('active', '=', True), ('is_published', '=', True), ('channel_id', 'in', self.ids), ('is_category', '=', False)],
            ['channel_id', 'slide_type', 'likes', 'dislikes', 'total_views', 'completion_time'],
            groupby=['channel_id', 'slide_type'],
            lazy=False)
        for res_group in read_group_res:
            cid = res_group['channel_id'][0]
            result[cid]['total_views'] += res_group.get('total_views', 0)
            result[cid]['total_votes'] += res_group.get('likes', 0)
            result[cid]['total_votes'] -= res_group.get('dislikes', 0)
            result[cid]['total_time'] += res_group.get('completion_time', 0)

        type_stats = self._compute_slides_statistics_type(read_group_res)
        for cid, cdata in type_stats.items():
            result[cid].update(cdata)

        for record in self:
            record.update(result.get(record.id, default_vals))

    def _compute_slides_statistics_type(self, read_group_res):
        """ Compute statistics based on all existing slide types """
        slide_types = self.env['slide.slide']._fields['slide_type'].get_values(self.env)
        keys = ['nbr_%s' % slide_type for slide_type in slide_types]
        result = dict((cid, dict((key, 0) for key in keys + ['total_slides'])) for cid in self.ids)
        for res_group in read_group_res:
            cid = res_group['channel_id'][0]
            slide_type = res_group.get('slide_type')
            if slide_type:
                slide_type_count = res_group.get('__count', 0)
                result[cid]['nbr_%s' % slide_type] = slide_type_count
                result[cid]['total_slides'] += slide_type_count
        return result

    def _compute_rating_stats(self):
        super(Channel, self)._compute_rating_stats()
        for record in self:
            record.rating_avg_stars = record.rating_avg

    @api.depends('slide_partner_ids', 'total_slides')
    @api.depends_context('uid')
    def _compute_user_statistics(self):
        current_user_info = self.env['slide.channel.partner'].sudo().search(
            [('channel_id', 'in', self.ids), ('partner_id', '=', self.env.user.partner_id.id)]
        )
        mapped_data = dict((info.channel_id.id, (info.completed, info.completed_slides_count)) for info in current_user_info)
        for record in self:
            completed, completed_slides_count = mapped_data.get(record.id, (False, 0))
            record.completed = completed
            record.completion = 100.0 if completed else round(100.0 * completed_slides_count / (record.total_slides or 1))

    @api.depends('upload_group_ids', 'user_id')
    @api.depends_context('uid')
    def _compute_can_upload(self):
        for record in self:
            if record.user_id == self.env.user or self.env.is_superuser():
                record.can_upload = True
            elif record.upload_group_ids:
                record.can_upload = bool(record.upload_group_ids & self.env.user.groups_id)
            else:
                record.can_upload = self.env.user.has_group('website_slides.group_website_slides_manager')

    @api.depends('channel_type', 'user_id', 'can_upload')
    @api.depends_context('uid')
    def _compute_can_publish(self):
        """ For channels of type 'training', only the responsible (see user_id field) can publish slides.
        The 'sudo' user needs to be handled because he's the one used for uploads done on the front-end when the
        logged in user is not publisher but fulfills the upload_group_ids condition. """
        for record in self:
            if not record.can_upload:
                record.can_publish = False
            elif record.user_id == self.env.user or self.env.is_superuser():
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

    @api.depends('name', 'website_id.domain')
    def _compute_website_url(self):
        super(Channel, self)._compute_website_url()
        for channel in self:
            if channel.id:  # avoid to perform a slug on a not yet saved record in case of an onchange.
                base_url = channel.get_base_url()
                channel.website_url = '%s/slides/%s' % (base_url, slug(channel))

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

    @api.model
    def create(self, vals):
        # Ensure creator is member of its channel it is easier for him to manage it (unless it is odoobot)
        if not vals.get('channel_partner_ids') and not self.env.is_superuser():
            vals['channel_partner_ids'] = [(0, 0, {
                'partner_id': self.env.user.partner_id.id
            })]
        if vals.get('description') and not vals.get('description_short'):
            vals['description_short'] = vals['description']
        channel = super(Channel, self.with_context(mail_create_nosubscribe=True)).create(vals)

        if channel.user_id:
            channel._action_add_members(channel.user_id.partner_id)
        if 'enroll_group_ids' in vals:
            channel._add_groups_members()

        return channel

    def write(self, vals):
        # If description_short wasn't manually modified, there is an implicit link between this field and description.
        if vals.get('description') and not vals.get('description_short') and self.description == self.description_short:
            vals['description_short'] = vals.get('description')

        res = super(Channel, self).write(vals)

        if vals.get('user_id'):
            self._action_add_members(self.env['res.users'].sudo().browse(vals['user_id']).partner_id)
            self.activity_reschedule(['website_slides.mail_activity_data_access_request'], new_user_id=vals.get('user_id'))
        if 'enroll_group_ids' in vals:
            self._add_groups_members()

        return res

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
        note as we don't want all channel followers to be notified of this answer. """
        self.ensure_one()
        if kwargs.get('message_type') == 'comment' and not self.can_review:
            raise AccessError(_('Not enough karma to review'))
        if parent_id:
            parent_message = self.env['mail.message'].sudo().browse(parent_id)
            if parent_message.subtype_id and parent_message.subtype_id == self.env.ref('website_slides.mt_channel_slide_published'):
                subtype_id = self.env.ref('mail.mt_note').id
        return super(Channel, self).message_post(parent_id=parent_id, subtype_id=subtype_id, **kwargs)

    # ---------------------------------------------------------
    # Business / Actions
    # ---------------------------------------------------------

    def action_redirect_to_members(self, state=None):
        action = self.env["ir.actions.actions"]._for_xml_id("website_slides.slide_channel_partner_action")
        action['domain'] = [('channel_id', 'in', self.ids)]
        if len(self) == 1:
            action['display_name'] = _('Attendees of %s', self.name)
            action['context'] = {'active_test': False, 'default_channel_id': self.id}
        if state:
            action['domain'] += [('completed', '=', state == 'completed')]
        return action

    def action_redirect_to_running_members(self):
        return self.action_redirect_to_members('running')

    def action_redirect_to_done_members(self):
        return self.action_redirect_to_members('completed')

    def action_channel_invite(self):
        self.ensure_one()
        template = self.env.ref('website_slides.mail_template_slide_channel_invite', raise_if_not_found=False)

        local_context = dict(
            self.env.context,
            default_channel_id=self.id,
            default_use_template=bool(template),
            default_template_id=template and template.id or False,
            notif_layout='website_slides.mail_notification_channel_invite',
        )
        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'slide.channel.invite',
            'target': 'new',
            'context': local_context,
        }

    def action_add_member(self, **member_values):
        """ Adds the logged in user in the channel members.
        (see '_action_add_members' for more info)

        Returns True if added successfully, False otherwise."""
        return bool(self._action_add_members(self.env.user.partner_id, **member_values))

    def _action_add_members(self, target_partners, **member_values):
        """ Add the target_partner as a member of the channel (to its slide.channel.partner).
        This will make the content (slides) of the channel available to that partner.

        Returns the added 'slide.channel.partner's (! as sudo !)
        """
        to_join = self._filter_add_members(target_partners, **member_values)
        if to_join:
            existing = self.env['slide.channel.partner'].sudo().search([
                ('channel_id', 'in', self.ids),
                ('partner_id', 'in', target_partners.ids)
            ])
            existing_map = dict((cid, list()) for cid in self.ids)
            for item in existing:
                existing_map[item.channel_id.id].append(item.partner_id.id)

            to_create_values = [
                dict(channel_id=channel.id, partner_id=partner.id, **member_values)
                for channel in to_join
                for partner in target_partners if partner.id not in existing_map[channel.id]
            ]
            slide_partners_sudo = self.env['slide.channel.partner'].sudo().create(to_create_values)
            to_join.message_subscribe(partner_ids=target_partners.ids, subtype_ids=[self.env.ref('website_slides.mt_channel_slide_published').id])
            return slide_partners_sudo
        return self.env['slide.channel.partner'].sudo()

    def _filter_add_members(self, target_partners, **member_values):
        allowed = self.filtered(lambda channel: channel.enroll == 'public')
        on_invite = self.filtered(lambda channel: channel.enroll == 'invite')
        if on_invite:
            try:
                on_invite.check_access_rights('write')
                on_invite.check_access_rule('write')
            except:
                pass
            else:
                allowed |= on_invite
        return allowed

    def _add_groups_members(self):
        for channel in self:
            channel._action_add_members(channel.mapped('enroll_group_ids.users.partner_id'))

    def _get_earned_karma(self, partner_ids):
        """ Compute the number of karma earned by partners on a channel
        Warning: this count will not be accurate if the configuration has been
        modified after the completion of a course!
        """
        total_karma = defaultdict(int)

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
            gains = [slide.quiz_first_attempt_reward,
                     slide.quiz_second_attempt_reward,
                     slide.quiz_third_attempt_reward,
                     slide.quiz_fourth_attempt_reward]
            attempts = min(partner_slide.quiz_attempts_count - 1, 3)
            total_karma[partner_slide.partner_id.id] += gains[attempts]

        channel_completed = self.env['slide.channel.partner'].sudo().search([
            ('partner_id', 'in', partner_ids),
            ('channel_id', 'in', self.ids),
            ('completed', '=', True)
        ])
        for partner_channel in channel_completed:
            channel = partner_channel.channel_id
            total_karma[partner_channel.partner_id.id] += channel.karma_gen_channel_finish

        return total_karma

    def _remove_membership(self, partner_ids):
        """ Unlink (!!!) the relationships between the passed partner_ids
        and the channels and their slides (done in the unlink of slide.channel.partner model).
        Remove earned karma when completed quizz """
        if not partner_ids:
            raise ValueError("Do not use this method with an empty partner_id recordset")

        earned_karma = self._get_earned_karma(partner_ids)
        users = self.env['res.users'].sudo().search([
            ('partner_id', 'in', list(earned_karma)),
        ])
        for user in users:
            if earned_karma[user.partner_id.id]:
                user.add_karma(-1 * earned_karma[user.partner_id.id])

        removed_channel_partner_domain = []
        for channel in self:
            removed_channel_partner_domain = expression.OR([
                removed_channel_partner_domain,
                [('partner_id', 'in', partner_ids),
                 ('channel_id', '=', channel.id)]
            ])
        self.message_unsubscribe(partner_ids=partner_ids)

        if removed_channel_partner_domain:
            self.env['slide.channel.partner'].sudo().search(removed_channel_partner_domain).unlink()

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
        action['name'] = _('Rating of %s') % (self.name)
        action['domain'] = expression.AND([ast.literal_eval(action.get('domain', '[]')), [('res_id', 'in', self.ids)]])
        return action

    def action_request_access(self):
        """ Request access to the channel. Returns a dict with keys being either 'error'
        (specific error raised) or 'done' (request done or not). """
        if self.env.user.has_group('base.group_public'):
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
            if channel.id not in requested_cids:
                activities += channel.activity_schedule(
                    'website_slides.mail_activity_data_access_request',
                    note=_('<b>%s</b> is requesting access to this course.') % partner.name,
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
                'name': category.name, 'slug_name': slug(category),
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

```

## File: models\slide_channel_tag.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

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
    group_id = fields.Many2one('slide.channel.tag.group', string='Group', index=True, required=True)
    group_sequence = fields.Integer(
        'Group sequence', related='group_id.sequence',
        index=True, readonly=True, store=True)
    channel_ids = fields.Many2many('slide.channel', 'slide_channel_tag_rel', 'tag_id', 'channel_id', string='Channels')
    color = fields.Integer(string='Color Index', help="Color to apply to this tag (including in website).")

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
    slide_id = fields.Many2one('slide.slide', string="Content", required=True)
    answer_ids = fields.One2many('slide.answer', 'question_id', string="Answer")
    # statistics
    attempts_count = fields.Integer(compute='_compute_statistics', groups='website_slides.group_website_slides_officer')
    attempts_avg = fields.Float(compute="_compute_statistics", digits=(6, 2), groups='website_slides.group_website_slides_officer')
    done_count = fields.Integer(compute="_compute_statistics", groups='website_slides.group_website_slides_officer')

    @api.constrains('answer_ids')
    def _check_answers_integrity(self):
        for question in self:
            if len(question.answer_ids.filtered(lambda answer: answer.is_correct)) != 1:
                raise ValidationError(_('Question "%s" must have 1 correct answer', question.question))
            if len(question.answer_ids) < 2:
                raise ValidationError(_('Question "%s" must have 1 correct answer and at least 1 incorrect answer', question.question))

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


class SlideAnswer(models.Model):
    _name = "slide.answer"
    _rec_name = "text_value"
    _description = "Slide Question's Answer"
    _order = 'question_id, sequence'

    sequence = fields.Integer("Sequence")
    question_id = fields.Many2one('slide.question', string="Question", required=True, ondelete='cascade')
    text_value = fields.Char("Answer", required=True, translate=True)
    is_correct = fields.Boolean("Is correct answer")
    comment = fields.Text("Comment", translate=True, help='This comment will be displayed to the user if he selects this answer')

```

## File: models\slide_slide.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import datetime
import io
import re
import requests
import PyPDF2
import json

from dateutil.relativedelta import relativedelta
from PIL import Image
from werkzeug import urls

from odoo import api, fields, models, _
from odoo.addons.http_routing.models.ir_http import slug
from odoo.exceptions import UserError, AccessError
from odoo.http import request
from odoo.addons.http_routing.models.ir_http import url_for
from odoo.tools import sql


class SlidePartnerRelation(models.Model):
    _name = 'slide.slide.partner'
    _description = 'Slide / Partner decorated m2m'
    _table = 'slide_slide_partner'

    slide_id = fields.Many2one('slide.slide', ondelete="cascade", index=True, required=True)
    channel_id = fields.Many2one(
        'slide.channel', string="Channel",
        related="slide_id.channel_id", store=True, index=True, ondelete='cascade')
    partner_id = fields.Many2one('res.partner', index=True, required=True, ondelete='cascade')
    vote = fields.Integer('Vote', default=0)
    completed = fields.Boolean('Completed')
    quiz_attempts_count = fields.Integer('Quiz attempts count', default=0)

    def create(self, values):
        res = super(SlidePartnerRelation, self).create(values)
        completed = res.filtered('completed')
        if completed:
            completed._set_completed_callback()
        return res

    def write(self, values):
        res = super(SlidePartnerRelation, self).write(values)
        if values.get('completed'):
            self._set_completed_callback()
        return res

    def _set_completed_callback(self):
        self.env['slide.channel.partner'].search([
            ('channel_id', 'in', self.channel_id.ids),
            ('partner_id', 'in', self.partner_id.ids),
        ])._recompute_completion()


class SlideLink(models.Model):
    _name = 'slide.slide.link'
    _description = "External URL for a particular slide"

    slide_id = fields.Many2one('slide.slide', required=True, ondelete='cascade')
    name = fields.Char('Title', required=True)
    link = fields.Char('Link', required=True)


class SlideResource(models.Model):
    _name = 'slide.slide.resource'
    _description = "Additional resource for a particular slide"

    slide_id = fields.Many2one('slide.slide', required=True, ondelete='cascade')
    name = fields.Char('Name', required=True)
    data = fields.Binary('Resource')


class EmbeddedSlide(models.Model):
    """ Embedding in third party websites. Track view count, generate statistics. """
    _name = 'slide.embed'
    _description = 'Embedded Slides View Counter'
    _rec_name = 'slide_id'

    slide_id = fields.Many2one('slide.slide', string="Presentation", required=True, index=True)
    url = fields.Char('Third Party Website URL', required=True)
    count_views = fields.Integer('# Views', default=1)

    def _add_embed_url(self, slide_id, url):
        baseurl = urls.url_parse(url).netloc
        if not baseurl:
            return 0
        embeds = self.search([('url', '=', baseurl), ('slide_id', '=', int(slide_id))], limit=1)
        if embeds:
            embeds.count_views += 1
        else:
            embeds = self.create({
                'slide_id': slide_id,
                'url': baseurl,
            })
        return embeds.count_views


class SlideTag(models.Model):
    """ Tag to search slides accross channels. """
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
        'website.seo.metadata', 'website.published.mixin']
    _description = 'Slides'
    _mail_post_access = 'read'
    _order_by_strategy = {
        'sequence': 'sequence asc, id asc',
        'most_viewed': 'total_views desc',
        'most_voted': 'likes desc',
        'latest': 'date_published desc',
    }
    _order = 'sequence asc, is_category asc, id asc'

    # description
    name = fields.Char('Title', required=True, translate=True)
    active = fields.Boolean(default=True, tracking=100)
    sequence = fields.Integer('Sequence', default=0)
    user_id = fields.Many2one('res.users', string='Uploaded by', default=lambda self: self.env.uid)
    description = fields.Text('Description', translate=True)
    channel_id = fields.Many2one('slide.channel', string="Course", required=True)
    tag_ids = fields.Many2many('slide.tag', 'rel_slide_tag', 'slide_id', 'tag_id', string='Tags')
    is_preview = fields.Boolean('Allow Preview', default=False, help="The course is accessible by anyone : the users don't need to join the channel to access the content of the course.")
    is_new_slide = fields.Boolean('Is New Slide', compute='_compute_is_new_slide')
    completion_time = fields.Float('Duration', digits=(10, 4), help="The estimated completion time for this slide")
    # Categories
    is_category = fields.Boolean('Is a category', default=False)
    category_id = fields.Many2one('slide.slide', string="Section", compute="_compute_category_id", store=True)
    slide_ids = fields.One2many('slide.slide', "category_id", string="Slides")
    # subscribers
    partner_ids = fields.Many2many('res.partner', 'slide_slide_partner', 'slide_id', 'partner_id',
                                   string='Subscribers', groups='website_slides.group_website_slides_officer', copy=False)
    slide_partner_ids = fields.One2many('slide.slide.partner', 'slide_id', string='Subscribers information', groups='website_slides.group_website_slides_officer', copy=False)
    user_membership_id = fields.Many2one(
        'slide.slide.partner', string="Subscriber information",
        compute='_compute_user_membership_id', compute_sudo=False,
        help="Subscriber information for the current logged in user")
    # Quiz related fields
    question_ids = fields.One2many("slide.question", "slide_id", string="Questions")
    questions_count = fields.Integer(string="Numbers of Questions", compute='_compute_questions_count')
    quiz_first_attempt_reward = fields.Integer("Reward: first attempt", default=10)
    quiz_second_attempt_reward = fields.Integer("Reward: second attempt", default=7)
    quiz_third_attempt_reward = fields.Integer("Reward: third attempt", default=5,)
    quiz_fourth_attempt_reward = fields.Integer("Reward: every attempt after the third try", default=2)
    # content
    slide_type = fields.Selection([
        ('infographic', 'Infographic'),
        ('webpage', 'Web Page'),
        ('presentation', 'Presentation'),
        ('document', 'Document'),
        ('video', 'Video'),
        ('quiz', "Quiz")],
        string='Type', required=True,
        default='document',
        help="The document type will be set automatically based on the document URL and properties (e.g. height and width for presentation and document).")
    datas = fields.Binary('Content', attachment=True)
    url = fields.Char('Document URL', help="Youtube or Google Document URL")
    document_id = fields.Char('Document ID', help="Youtube or Google Document ID")
    link_ids = fields.One2many('slide.slide.link', 'slide_id', string="External URL for this slide")
    slide_resource_ids = fields.One2many('slide.slide.resource', 'slide_id', string="Additional Resource for this slide")
    slide_resource_downloadable = fields.Boolean('Allow Download', default=True, help="Allow the user to download the content of the slide.")
    mime_type = fields.Char('Mime-type')
    html_content = fields.Html("HTML Content", help="Custom HTML content for slides of type 'Web Page'.", translate=True, sanitize_attributes=False, sanitize_form=False)
    # website
    website_id = fields.Many2one(related='channel_id.website_id', readonly=True)
    date_published = fields.Datetime('Publish Date', readonly=True, tracking=1)
    likes = fields.Integer('Likes', compute='_compute_like_info', store=True, compute_sudo=False)
    dislikes = fields.Integer('Dislikes', compute='_compute_like_info', store=True, compute_sudo=False)
    user_vote = fields.Integer('User vote', compute='_compute_user_membership_id', compute_sudo=False)
    embed_code = fields.Text('Embed Code', readonly=True, compute='_compute_embed_code')
    # views
    embedcount_ids = fields.One2many('slide.embed', 'slide_id', string="Embed Count")
    slide_views = fields.Integer('# of Website Views', store=True, compute="_compute_slide_views")
    public_views = fields.Integer('# of Public Views', copy=False)
    total_views = fields.Integer("Views", default="0", compute='_compute_total', store=True)
    # comments
    comments_count = fields.Integer('Number of comments', compute="_compute_comments_count")
    # channel
    channel_type = fields.Selection(related="channel_id.channel_type", string="Channel type")
    channel_allow_comment = fields.Boolean(related="channel_id.allow_comment", string="Allows comment")
    # Statistics in case the slide is a category
    nbr_presentation = fields.Integer("Number of Presentations", compute='_compute_slides_statistics', store=True)
    nbr_document = fields.Integer("Number of Documents", compute='_compute_slides_statistics', store=True)
    nbr_video = fields.Integer("Number of Videos", compute='_compute_slides_statistics', store=True)
    nbr_infographic = fields.Integer("Number of Infographics", compute='_compute_slides_statistics', store=True)
    nbr_webpage = fields.Integer("Number of Webpages", compute='_compute_slides_statistics', store=True)
    nbr_quiz = fields.Integer("Number of Quizs", compute="_compute_slides_statistics", store=True)
    total_slides = fields.Integer(compute='_compute_slides_statistics', store=True)

    _sql_constraints = [
        ('exclusion_html_content_and_url', "CHECK(html_content IS NULL OR url IS NULL)", "A slide is either filled with a document url or HTML content. Not both.")
    ]

    @api.depends('date_published', 'is_published')
    def _compute_is_new_slide(self):
        for slide in self:
            slide.is_new_slide = slide.date_published > fields.Datetime.now() - relativedelta(days=7) if slide.is_published else False

    @api.depends('channel_id.slide_ids.is_category', 'channel_id.slide_ids.sequence')
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

    @api.depends('question_ids')
    def _compute_questions_count(self):
        for slide in self:
            slide.questions_count = len(slide.question_ids)

    def _has_additional_resources(self):
        """Sudo required for public user to know if the course has additional
        resources that they will be able to access once a member."""
        self.ensure_one()
        return bool(self.sudo().slide_resource_ids)

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
        if not self.ids:
            self.update({'likes': 0, 'dislikes': 0})
            return

        rg_data_like = self.env['slide.slide.partner'].sudo().read_group(
            [('slide_id', 'in', self.ids), ('vote', '=', 1)],
            ['slide_id'], ['slide_id']
        )
        rg_data_dislike = self.env['slide.slide.partner'].sudo().read_group(
            [('slide_id', 'in', self.ids), ('vote', '=', -1)],
            ['slide_id'], ['slide_id']
        )
        mapped_data_like = dict(
            (rg_data['slide_id'][0], rg_data['slide_id_count'])
            for rg_data in rg_data_like
        )
        mapped_data_dislike = dict(
            (rg_data['slide_id'][0], rg_data['slide_id_count'])
            for rg_data in rg_data_dislike
        )

        for slide in self:
            slide.likes = mapped_data_like.get(slide.id, 0)
            slide.dislikes = mapped_data_dislike.get(slide.id, 0)

    @api.depends('slide_partner_ids.vote')
    @api.depends_context('uid')
    def _compute_user_info(self):
        """ Deprecated. Now computed directly by _compute_user_membership_id
        for user_vote and _compute_like_info for likes / dislikes. Remove me in
        master. """
        default_stats = {'likes': 0, 'dislikes': 0, 'user_vote': False}

        if not self.ids:
            self.update(default_stats)
            return

        slide_data = dict.fromkeys(self.ids, default_stats)
        slide_partners = self.env['slide.slide.partner'].sudo().search([
            ('slide_id', 'in', self.ids)
        ])

        for slide_partner in slide_partners:
            if slide_partner.vote == 1:
                slide_data[slide_partner.slide_id.id]['likes'] += 1
                if slide_partner.partner_id == self.env.user.partner_id:
                    slide_data[slide_partner.slide_id.id]['user_vote'] = 1
            elif slide_partner.vote == -1:
                slide_data[slide_partner.slide_id.id]['dislikes'] += 1
                if slide_partner.partner_id == self.env.user.partner_id:
                    slide_data[slide_partner.slide_id.id]['user_vote'] = -1

        for slide in self:
            slide.update(slide_data[slide.id])

    @api.depends('slide_partner_ids.slide_id')
    def _compute_slide_views(self):
        # TODO awa: tried compute_sudo, for some reason it doesn't work in here...
        read_group_res = self.env['slide.slide.partner'].sudo().read_group(
            [('slide_id', 'in', self.ids)],
            ['slide_id'],
            groupby=['slide_id']
        )
        mapped_data = dict((res['slide_id'][0], res['slide_id_count']) for res in read_group_res)
        for slide in self:
            slide.slide_views = mapped_data.get(slide.id, 0)

    @api.depends('slide_ids.sequence', 'slide_ids.slide_type', 'slide_ids.is_published', 'slide_ids.is_category')
    def _compute_slides_statistics(self):
        # Do not use dict.fromkeys(self.ids, dict()) otherwise it will use the same dictionnary for all keys.
        # Therefore, when updating the dict of one key, it updates the dict of all keys.
        keys = ['nbr_%s' % slide_type for slide_type in self.env['slide.slide']._fields['slide_type'].get_values(self.env)]
        default_vals = dict((key, 0) for key in keys + ['total_slides'])

        res = self.env['slide.slide'].read_group(
            [('is_published', '=', True), ('category_id', 'in', self.ids), ('is_category', '=', False)],
            ['category_id', 'slide_type'], ['category_id', 'slide_type'],
            lazy=False)

        type_stats = self._compute_slides_statistics_type(res)

        for record in self:
            record.update(type_stats.get(record._origin.id, default_vals))

    def _compute_slides_statistics_type(self, read_group_res):
        """ Compute statistics based on all existing slide types """
        slide_types = self.env['slide.slide']._fields['slide_type'].get_values(self.env)
        keys = ['nbr_%s' % slide_type for slide_type in slide_types]
        result = dict((cid, dict((key, 0) for key in keys + ['total_slides'])) for cid in self.ids)
        for res_group in read_group_res:
            cid = res_group['category_id'][0]
            slide_type = res_group.get('slide_type')
            if slide_type:
                slide_type_count = res_group.get('__count', 0)
                result[cid]['nbr_%s' % slide_type] = slide_type_count
                result[cid]['total_slides'] += slide_type_count
        return result

    @api.depends('slide_partner_ids.partner_id', 'slide_partner_ids.vote')
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

    @api.depends('document_id', 'slide_type', 'mime_type')
    def _compute_embed_code(self):
        base_url = request and request.httprequest.url_root or self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        if base_url[-1] == '/':
            base_url = base_url[:-1]
        for record in self:
            if record.datas and (not record.document_id or record.slide_type in ['document', 'presentation']):
                slide_url = base_url + url_for('/slides/embed/%s?page=1' % record.id)
                record.embed_code = '<iframe src="%s" class="o_wslides_iframe_viewer" allowFullScreen="true" height="%s" width="%s" frameborder="0"></iframe>' % (slide_url, 315, 420)
            elif record.slide_type == 'video' and record.document_id:
                if not record.mime_type:
                    # embed youtube video
                    query = urls.url_parse(record.url).query
                    query = query + '&theme=light' if query else 'theme=light'
                    record.embed_code = '<iframe src="//www.youtube-nocookie.com/embed/%s?%s" allowFullScreen="true" frameborder="0"></iframe>' % (record.document_id, query)
                else:
                    # embed google doc video
                    record.embed_code = '<iframe src="//drive.google.com/file/d/%s/preview" allowFullScreen="true" frameborder="0"></iframe>' % (record.document_id)
            else:
                record.embed_code = False

    @api.onchange('url')
    def _on_change_url(self):
        self.ensure_one()
        if self.url:
            res = self._parse_document_url(self.url)
            if res.get('error'):
                raise UserError(res.get('error'))
            values = res['values']
            if not values.get('document_id'):
                raise UserError(_('Please enter valid Youtube or Google Doc URL'))
            for key, value in values.items():
                self[key] = value

    @api.onchange('datas')
    def _on_change_datas(self):
        """ For PDFs, we assume that it takes 5 minutes to read a page.
            If the selected file is not a PDF, it is an image (You can
            only upload PDF or Image file) then the slide_type is changed
            into infographic and the uploaded dataS is transfered to the
            image field. (It avoids the infinite loading in PDF viewer)"""
        if self.datas:
            data = base64.b64decode(self.datas)
            if data.startswith(b'%PDF-'):
                pdf = PyPDF2.PdfFileReader(io.BytesIO(data), overwriteWarnings=False, strict=False)
                try:
                    pdf.getNumPages()
                except PyPDF2.utils.PdfReadError:
                    return
                self.completion_time = (5 * len(pdf.pages)) / 60
            else:
                self.slide_type = 'infographic'
                self.image_1920 = self.datas
                self.datas = None

    @api.depends('name', 'channel_id.website_id.domain')
    def _compute_website_url(self):
        # TDE FIXME: clena this link.tracker strange stuff
        super(Slide, self)._compute_website_url()
        for slide in self:
            if slide.id:  # avoid to perform a slug on a not yet saved record in case of an onchange.
                base_url = slide.channel_id.get_base_url()
                # link_tracker is not in dependencies, so use it to shorten url only if installed.
                if self.env.registry.get('link.tracker'):
                    url = self.env['link.tracker'].sudo().create({
                        'url': '%s/slides/slide/%s' % (base_url, slug(slide)),
                        'title': slide.name,
                    }).short_url
                else:
                    url = '%s/slides/slide/%s' % (base_url, slug(slide))
                slide.website_url = url

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

    @api.model
    def create(self, values):
        # Do not publish slide if user has not publisher rights
        channel = self.env['slide.channel'].browse(values['channel_id'])
        if not channel.can_publish:
            # 'website_published' is handled by mixin
            values['date_published'] = False

        if values.get('slide_type') == 'infographic' and not values.get('image_1920'):
            values['image_1920'] = values['datas']
        if values.get('is_category'):
            values['is_preview'] = True
            values['is_published'] = True
        if values.get('is_published') and not values.get('date_published'):
            values['date_published'] = datetime.datetime.now()
        if values.get('url') and not values.get('document_id'):
            doc_data = self._parse_document_url(values['url']).get('values', dict())
            for key, value in doc_data.items():
                values.setdefault(key, value)

        slide = super(Slide, self).create(values)

        if slide.is_published and not slide.is_category:
            slide._post_publication()
        return slide

    def write(self, values):
        if values.get('url') and values['url'] != self.url:
            doc_data = self._parse_document_url(values['url']).get('values', dict())
            for key, value in doc_data.items():
                values.setdefault(key, value)
        if values.get('is_category'):
            values['is_preview'] = True
            values['is_published'] = True

        res = super(Slide, self).write(values)
        if values.get('is_published'):
            self.date_published = datetime.datetime.now()
            self._post_publication()

        if 'is_published' in values or 'active' in values:
            # if the slide is published/unpublished, recompute the completion for the partners
            self.slide_partner_ids._set_completed_callback()

        return res

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        """Sets the sequence to zero so that it always lands at the beginning
        of the newly selected course as an uncategorized slide"""
        rec = super(Slide, self).copy(default)
        rec.sequence = 0
        return rec

    def unlink(self):
        if self.question_ids and self.channel_id.channel_partner_ids:
            raise UserError(_("People already took this quiz. To keep course progression it should not be deleted."))
        for category in self.filtered(lambda slide: slide.is_category):
            category.channel_id._move_category_slides(category, False)
        super(Slide, self).unlink()

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

    def get_access_action(self, access_uid=None):
        """ Instead of the classic form view, redirect to website if it is published. """
        self.ensure_one()
        if self.website_published:
            return {
                'type': 'ir.actions.act_url',
                'url': '%s' % self.website_url,
                'target': 'self',
                'target_type': 'public',
                'res_id': self.id,
            }
        return super(Slide, self).get_access_action(access_uid)

    def _notify_get_groups(self, msg_vals=None):
        """ Add access button to everyone if the document is active. """
        groups = super(Slide, self)._notify_get_groups(msg_vals=msg_vals)

        if self.website_published:
            for group_name, group_method, group_data in groups:
                group_data['has_button_access'] = True

        return groups

    # ---------------------------------------------------------
    # Business Methods
    # ---------------------------------------------------------

    def _post_publication(self):
        base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        for slide in self.filtered(lambda slide: slide.website_published and slide.channel_id.publish_template_id):
            publish_template = slide.channel_id.publish_template_id
            html_body = publish_template.with_context(base_url=base_url)._render_field('body_html', slide.ids)[slide.id]
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
        # TDE FIXME: template to check
        mail_ids = []
        base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        for record in self:
            template = record.channel_id.share_template_id.with_context(
                user=self.env.user,
                email=email,
                base_url=base_url,
                fullscreen=fullscreen
            )
            email_values = {'email_to': email}
            if self.env.user.has_group('base.group_portal'):
                template = template.sudo()
                email_values['email_from'] = self.env.company.catchall_formatted or self.env.company.email_formatted

            mail_ids.append(template.send_mail(record.id, notif_layout='mail.mail_notification_light', email_values=email_values))
        return mail_ids

    def action_like(self):
        self.check_access_rights('read')
        self.check_access_rule('read')
        return self._action_vote(upvote=True)

    def action_dislike(self):
        self.check_access_rights('read')
        self.check_access_rule('read')
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
        channel = slide_id.channel_id
        karma_to_add = 0

        for slide_partner in slide_partners:
            if upvote:
                new_vote = 0 if slide_partner.vote == -1 else 1
                if slide_partner.vote != 1:
                    karma_to_add += channel.karma_gen_slide_vote
            else:
                new_vote = 0 if slide_partner.vote == 1 else -1
                if slide_partner.vote != -1:
                    karma_to_add -= channel.karma_gen_slide_vote
            slide_partner.vote = new_vote

        for new_slide in new_slides:
            new_vote = 1 if upvote else -1
            new_slide.write({
                'slide_partner_ids': [(0, 0, {'vote': new_vote, 'partner_id': self.env.user.partner_id.id})]
            })
            karma_to_add += new_slide.channel_id.karma_gen_slide_vote * (1 if upvote else -1)

        if karma_to_add:
            self.env.user.add_karma(karma_to_add)

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
            sql.increment_field_skiplock(existing_sudo, 'quiz_attempts_count')
            SlidePartnerSudo.invalidate_cache(fnames=['quiz_attempts_count'], ids=existing_sudo.ids)

        new_slides = self_sudo - existing_sudo.mapped('slide_id')
        return SlidePartnerSudo.create([{
            'slide_id': new_slide.id,
            'channel_id': new_slide.channel_id.id,
            'partner_id': target_partner.id,
            'quiz_attempts_count': 1 if quiz_attempts_inc else 0,
            'vote': 0} for new_slide in new_slides])

    def action_set_completed(self):
        if any(not slide.channel_id.is_member for slide in self):
            raise UserError(_('You cannot mark a slide as completed if you are not among its members.'))

        return self._action_set_completed(self.env.user.partner_id)

    def _action_set_completed(self, target_partner):
        self_sudo = self.sudo()
        SlidePartnerSudo = self.env['slide.slide.partner'].sudo()
        existing_sudo = SlidePartnerSudo.search([
            ('slide_id', 'in', self.ids),
            ('partner_id', '=', target_partner.id)
        ])
        existing_sudo.write({'completed': True})

        new_slides = self_sudo - existing_sudo.mapped('slide_id')
        SlidePartnerSudo.create([{
            'slide_id': new_slide.id,
            'channel_id': new_slide.channel_id.id,
            'partner_id': target_partner.id,
            'vote': 0,
            'completed': True} for new_slide in new_slides])

        return True

    def _action_set_quiz_done(self):
        if any(not slide.channel_id.is_member for slide in self):
            raise UserError(_('You cannot mark a slide quiz as completed if you are not among its members.'))

        points = 0
        for slide in self:
            user_membership_sudo = slide.user_membership_id.sudo()
            if not user_membership_sudo or user_membership_sudo.completed or not user_membership_sudo.quiz_attempts_count:
                continue

            gains = [slide.quiz_first_attempt_reward,
                     slide.quiz_second_attempt_reward,
                     slide.quiz_third_attempt_reward,
                     slide.quiz_fourth_attempt_reward]
            points += gains[user_membership_sudo.quiz_attempts_count - 1] if user_membership_sudo.quiz_attempts_count <= len(gains) else gains[-1]

        return self.env.user.sudo().add_karma(points)

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

    @api.model
    def _fetch_data(self, base_url, params, content_type=False):
        result = {'values': dict()}
        try:
            response = requests.get(base_url, timeout=3, params=params)
            response.raise_for_status()
            if content_type == 'json':
                result['values'] = response.json()
            elif content_type in ('image', 'pdf'):
                result['values'] = base64.b64encode(response.content)
            else:
                result['values'] = response.content
        except requests.exceptions.HTTPError as e:
            result['error'] = e.response.content
        except requests.exceptions.ConnectionError as e:
            result['error'] = str(e)
        return result

    def _find_document_data_from_url(self, url):
        url_obj = urls.url_parse(url)
        if url_obj.ascii_host == 'youtu.be':
            return ('youtube', url_obj.path[1:] if url_obj.path else False)
        elif url_obj.ascii_host in ('youtube.com', 'www.youtube.com', 'm.youtube.com', 'www.youtube-nocookie.com'):
            v_query_value = url_obj.decode_query().get('v')
            if v_query_value:
                return ('youtube', v_query_value)
            split_path = url_obj.path.split('/')
            if len(split_path) >= 3 and split_path[1] in ('v', 'embed'):
                return ('youtube', split_path[2])

        expr = re.compile(r'(^https:\/\/docs.google.com|^https:\/\/drive.google.com).*\/d\/([^\/]*)')
        arg = expr.match(url)
        document_id = arg and arg.group(2) or False
        if document_id:
            return ('google', document_id)

        return (None, False)

    def _parse_document_url(self, url, only_preview_fields=False):
        document_source, document_id = self._find_document_data_from_url(url)
        if document_source and hasattr(self, '_parse_%s_document' % document_source):
            return getattr(self, '_parse_%s_document' % document_source)(document_id, only_preview_fields)
        return {'error': _('Unknown document')}

    def _parse_youtube_document(self, document_id, only_preview_fields):
        """ If we receive a duration (YT video), we use it to determine the slide duration.
        The received duration is under a special format (e.g: PT1M21S15, meaning 1h 21m 15s). """

        key = self.env['website'].get_current_website().sudo().website_slide_google_app_key
        fetch_res = self._fetch_data('https://www.googleapis.com/youtube/v3/videos', {'id': document_id, 'key': key, 'part': 'snippet,contentDetails', 'fields': 'items(id,snippet,contentDetails)'}, 'json')
        if fetch_res.get('error'):
            return {'error': self._extract_google_error_message(fetch_res.get('error'))}

        values = {'slide_type': 'video', 'document_id': document_id}
        items = fetch_res['values'].get('items')
        if not items:
            return {'error': _('Please enter valid Youtube or Google Doc URL')}
        youtube_values = items[0]

        youtube_duration = youtube_values.get('contentDetails', {}).get('duration')
        if youtube_duration:
            parsed_duration = re.search(r'^PT(?:(\d+)H)?(?:(\d+)M)?(?:(\d+)S)?$', youtube_duration)
            if parsed_duration:
                values['completion_time'] = (int(parsed_duration.group(1) or 0)) + \
                                            (int(parsed_duration.group(2) or 0) / 60) + \
                                            (int(parsed_duration.group(3) or 0) / 3600)

        if youtube_values.get('snippet'):
            snippet = youtube_values['snippet']
            if only_preview_fields:
                values.update({
                    'url_src': snippet['thumbnails']['high']['url'],
                    'title': snippet['title'],
                    'description': snippet['description']
                })

                return values

            values.update({
                'name': snippet['title'],
                'image_1920': self._fetch_data(snippet['thumbnails']['high']['url'], {}, 'image')['values'],
                'description': snippet['description'],
                'mime_type': False,
            })
        return {'values': values}

    def _extract_google_error_message(self, error):
        """
        See here for Google error format
        https://developers.google.com/drive/api/v3/handle-errors
        """
        try:
            error = json.loads(error)
            error = (error.get('error', {}).get('errors', []) or [{}])[0].get('reason')
        except json.decoder.JSONDecodeError:
            error = str(error)

        if error == 'keyInvalid':
            return _('Your Google API key is invalid, please update it in your settings.\nSettings > Website > Features > API Key')

        return _('Could not fetch data from url. Document or access right not available:\n%s', error)

    @api.model
    def _parse_google_document(self, document_id, only_preview_fields):
        def get_slide_type(vals):
            # TDE FIXME: WTF ??
            slide_type = 'presentation'
            if vals.get('image_1920'):
                image = Image.open(io.BytesIO(base64.b64decode(vals['image_1920'])))
                width, height = image.size
                if height > width:
                    return 'document'
            return slide_type

        # Google drive doesn't use a simple API key to access the data, but requires an access
        # token. However, this token is generated in module google_drive, which is not in the
        # dependencies of website_slides. We still keep the 'key' parameter just in case, but that
        # is probably useless.
        params = {}
        params['projection'] = 'BASIC'
        if 'google.drive.config' in self.env:
            access_token = self.env['google.drive.config'].get_access_token()
            if access_token:
                params['access_token'] = access_token
        if not params.get('access_token'):
            params['key'] = self.env['website'].get_current_website().sudo().website_slide_google_app_key

        fetch_res = self._fetch_data('https://www.googleapis.com/drive/v2/files/%s' % document_id, params, "json")
        if fetch_res.get('error'):
            return {'error': self._extract_google_error_message(fetch_res.get('error'))}

        google_values = fetch_res['values']
        if only_preview_fields:
            return {
                'url_src': google_values['thumbnailLink'],
                'title': google_values['title'],
            }

        values = {
            'name': google_values['title'],
            'image_1920': self._fetch_data(google_values['thumbnailLink'].replace('=s220', ''), {}, 'image')['values'],
            'mime_type': google_values['mimeType'],
            'document_id': document_id,
        }
        if google_values['mimeType'].startswith('video/'):
            values['slide_type'] = 'video'
        elif google_values['mimeType'].startswith('image/'):
            values['datas'] = values['image_1920']
            values['slide_type'] = 'infographic'
        elif google_values['mimeType'].startswith('application/vnd.google-apps'):
            values['slide_type'] = get_slide_type(values)
            if 'exportLinks' in google_values:
                values['datas'] = self._fetch_data(google_values['exportLinks']['application/pdf'], params, 'pdf')['values']
        elif google_values['mimeType'] == 'application/pdf':
            # TODO: Google Drive PDF document doesn't provide plain text transcript
            values['datas'] = self._fetch_data(google_values['webContentLink'], {}, 'pdf')['values']
            values['slide_type'] = get_slide_type(values)

        return {'values': values}

    def _default_website_meta(self):
        res = super(Slide, self)._default_website_meta()
        res['default_opengraph']['og:title'] = res['default_twitter']['twitter:title'] = self.name
        res['default_opengraph']['og:description'] = res['default_twitter']['twitter:description'] = self.description
        res['default_opengraph']['og:image'] = res['default_twitter']['twitter:image'] = self.env['website'].image_url(self, 'image_1024')
        res['default_meta_description'] = self.description
        return res

    # ---------------------------------------------------------
    # Data / Misc
    # ---------------------------------------------------------

    def get_backend_menu_id(self):
        return self.env.ref('website_slides.website_slides_menu_root').id

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.addons.http_routing.models.ir_http import url_for


class Website(models.Model):
    _inherit = "website"

    website_slide_google_app_key = fields.Char('Google Doc Key', groups='base.group_system')

    def get_suggested_controllers(self):
        suggested_controllers = super(Website, self).get_suggested_controllers()
        suggested_controllers.append((_('Courses'), url_for('/slides'), 'website_slides'))
        return suggested_controllers

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import gamification_challenge
from . import slide_slide
from . import slide_question
from . import slide_channel
from . import slide_channel_tag
from . import res_config_settings
from . import website
from . import res_users
from . import res_groups
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_slide_slide_all,slide.slide.all,model_slide_slide,,1,0,0,0
access_slide_slide_officer,slide.slide.officer,model_slide_slide,website_slides.group_website_slides_officer,1,1,1,0
access_slide_slide_manager,slide.slide.manager,model_slide_slide,website_slides.group_website_slides_manager,1,1,1,1
access_slide_slide_partner_all,slide.slide.partner.all,model_slide_slide_partner,,0,0,0,0
access_slide_slide_partner_system,slide.slide.partner.system,model_slide_slide_partner,website_slides.group_website_slides_officer,1,1,1,1
access_slide_question_all,slide.question.all,model_slide_question,,1,0,0,0
access_slide_question_officer,slide.question.officer,model_slide_question,website_slides.group_website_slides_officer,1,1,1,1
access_slide_answer_all,slide.answer.all,model_slide_answer,,0,0,0,0
access_slide_answer_officer,slide.answer.officer,model_slide_answer,website_slides.group_website_slides_officer,1,1,1,1
access_slide_tag_all,slide.tag.all,model_slide_tag,,1,0,0,0
access_slide_tag_officer,slide.tag.officer,model_slide_tag,website_slides.group_website_slides_officer,1,1,1,1
access_slide_channel_tag_all,slide.channel.tag.all,model_slide_channel_tag,,1,0,0,0
access_slide_channel_tag_user,slide.channel.tag.user,model_slide_channel_tag,website_slides.group_website_slides_officer,1,1,1,1
access_slide_channel_tag_group_all,slide.channel.tag.group.all,model_slide_channel_tag_group,,1,0,0,0
access_slide_channel_tag_group_user,slide.channel.tag.group.user,model_slide_channel_tag_group,website_slides.group_website_slides_officer,1,1,1,1
access_slide_channel_all,slide.channel.all,model_slide_channel,,1,0,0,0
access_slide_channel_officer,slide.channel.officer,model_slide_channel,website_slides.group_website_slides_officer,1,1,1,0
access_slide_channel_manager,slide.channel.manager,model_slide_channel,website_slides.group_website_slides_manager,1,1,1,1
access_slide_channel_partners_all,slide.channel.users.all,model_slide_channel_partner,,0,0,0,0
access_slide_channel_partners_system,slide.channel.users.system,model_slide_channel_partner,website_slides.group_website_slides_officer,1,1,1,1
access_slide_embed_all,slide.embed.all,model_slide_embed,,1,0,0,0
access_slide_embed_user,slide.embed.user,model_slide_embed,base.group_user,1,1,1,1
access_slide_slide_link_all,slide.slide.link.all,model_slide_slide_link,,1,0,0,0
access_slide_slide_link_officer,slide.slide.link.officer,model_slide_slide_link,website_slides.group_website_slides_officer,1,1,1,1
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
            <field name="implied_ids" eval="[(4, ref('website.group_website_publisher'))]"/>
        </record>

        <record id="group_website_slides_manager" model="res.groups">
            <field name="name">Manager</field>
            <field name="category_id" ref="base.module_category_website_elearning"/>
            <field name="implied_ids" eval="[(4, ref('group_website_slides_officer'))]"/>
        </record>

        <record id="base.default_user" model="res.users">
            <field name="groups_id" eval="[(4,ref('group_website_slides_manager'))]"/>
        </record>

        <record id="base.group_system" model="res.groups">
            <field name="implied_ids" eval="[(4, ref('group_website_slides_manager'))]"/>
        </record>

        <data noupdate="1">
        <!-- CHANNEL -->
        <record id="rule_slide_channel_global" model="ir.rule">
            <field name="name">Channel: always visible (sub rules exist)</field>
            <field name="model_id" ref="model_slide_channel"/>
            <field name="domain_force">[(1, '=', 1)]</field>
        </record>

        <record id="rule_slide_channel_not_website" model="ir.rule">
            <field name="name">Channel: public/portal/user: restricted to published and (public or member only)</field>
            <field name="model_id" ref="model_slide_channel"/>
            <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
            <field name="domain_force">['&amp;', ('website_published', '=', True), '|', ('visibility', '=', 'public'), ('partner_ids', '=', user.partner_id.id)]</field>
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

        <!-- SLIDE -->
        <record id="rule_slide_slide_global" model="ir.rule">
            <field name="name">Slide: always visible (sub rules exist)</field>
            <field name="model_id" ref="model_slide_slide"/>
            <field name="domain_force">[(1, '=', 1)]</field>
        </record>

        <record id="rule_slide_slide_not_website" model="ir.rule">
            <field name="name">Slide: public/portal/user: restricted to published or uploaded by user, and either channel member or public channel &amp; (category or previewable)</field>
            <field name="model_id" ref="model_slide_slide"/>
            <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
            <field name="domain_force">['&amp;',
    '|',
        '&amp;', ('channel_id.visibility', '=', 'public'), '|', ('is_category','=', True), ('is_preview', '=', True),
        ('channel_id.partner_ids', '=', user.partner_id.id),
    '&amp;', ('channel_id.website_published', '=', True), '|', ('user_id', '=', user.id), ('website_published', '=', True)]</field>
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
            <field name="name">Resource: read restricted to channel members</field>
            <field name="model_id" ref="model_slide_slide_resource"/>
            <field name="domain_force">[('slide_id.channel_id.partner_ids', '=', user.partner_id.id)]</field>
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

        <record id="rule_slide_slide_resource_manager" model="ir.rule">
            <field name="name">Resource: manager: crud all</field>
            <field name="model_id" ref="model_slide_slide_resource"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_website_slides_manager'))]"/>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="True"/>
            <field name="perm_create" eval="True"/>
            <field name="perm_unlink" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#B06161"/><stop offset="45.785%" stop-color="#984E4E"/><stop offset="100%" stop-color="#7C3838"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M44.5 69H4c-2 0-4-1-4-4V37.061L13.363 19H40l3 6-4.495 6.88L53.393 35l-5.208 6.243 10.564 8.238L44.5 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M33.875 44H15a2 2 0 0 1-2-2V22a2 2 0 0 1 2-2h26a2 2 0 0 1 2 2v4.865a7.499 7.499 0 0 0-4 6.635c0 1.946.74 3.718 1.956 5.051a4.099 4.099 0 0 0-.387-.021c-2.049 0-1.256-.22-3.073-2.05-3.187 1.066-6.105 4.876-6.105 5.55.625.317 1.453.973 2.484 1.97zM43 43.132c2.348-1.51 8.09-1.51 10.439 0 1.565 1.007 3.392 3.775 5.48 8.304-1.549 1.77-2.593 2.525-3.132 2.265-1.044-1.51-1.826-2.768-2.348-3.775v5.284c-3.653 1.007-7.306 1.007-10.96 0v-5.284C38.16 45.976 35.334 43.666 34 43c0-.573 0-1 1-2 .667-.667 1.667-1.333 3-2 1.768 3.761 3.434 5.139 5 4.132zm4.74.868l-1.13 4.578 1.695 2.29L50 48.578 48.87 44h-1.13zM22 51l6-7 6 7h-2l-4-5-4 5h-2zm6-31v-2 2zm19.5 21a6.5 6.5 0 1 1 0-13 6.5 6.5 0 0 1 0 13z" opacity=".3"/><path fill="#FFF" d="M33.875 42H15a2 2 0 0 1-2-2V20a2 2 0 0 1 2-2h26a2 2 0 0 1 2 2v4.865a7.499 7.499 0 0 0-4 6.635c0 1.946.74 3.718 1.956 5.051a4.099 4.099 0 0 0-.387-.021c-2.049 0-1.256-.22-3.073-2.05-3.187 1.066-6.105 4.876-6.105 5.55.625.317 1.453.973 2.484 1.97zM43 41.132c2.348-1.51 8.09-1.51 10.439 0 1.565 1.007 3.392 3.775 5.48 8.304-1.549 1.77-2.593 2.525-3.132 2.265-1.044-1.51-1.826-2.768-2.348-3.775v5.284c-3.653 1.007-7.306 1.007-10.96 0v-5.284C38.16 43.976 35.334 41.666 34 41c0-.573 0-1 1-2 .667-.667 1.667-1.333 3-2 1.768 3.761 3.434 5.139 5 4.132zm4.74.868l-1.13 4.578 1.695 2.29L50 46.578 48.87 42h-1.13zM22 49l6-7 6 7h-2l-4-5-4 5h-2zm6-31v-2 2zm19.5 21a6.5 6.5 0 1 1 0-13 6.5 6.5 0 0 1 0 13z"/></g></g></svg>
```

## File: static\lib\pdfslidesviewer\PDFSlidesViewer.js

```javascript
/**
    Homemade helper for browsing PDF document from page to page.
    This is hightly inspired from https://github.com/mozilla/pdf.js/blob/master/examples/learning/prevnext.html
    This lib requires PDF JS. It simply uses PDFjs and its promises.
    DOC : http://mozilla.github.io/pdf.js/api/draft/api.js.html
*/

// !!!!!!!!! use window.pdfjsLib and not pdfjsLib

var PDFSlidesViewer = (function(){
    function PDFSlidesViewer(pdf_url, $canvas, disableWorker){
        // pdf variables
        this.pdf = null;
        this.pdf_url = pdf_url || false;
        this.pdf_page_total = 0;
        this.pdf_page_current = 1; // default is the first page
        this.pdf_zoom = 1; // 1 = scale to fit to available space
        // promise business
        this.pageRendering = false;
        this.pageNumPending = null;
        //canvas
        this.canvas = $canvas;
        this.canvas_context = $canvas.getContext('2d');
        // PDF JS business
        /**
         * Disable the web worker and run all code on the main thread. This will happen
         * automatically if the browser doesn't support workers or sending typed arrays
         * to workers.
         * @var {boolean}
         *
         * disableWorker should be 'true' if the document came from another origin than the
         * page (typically the 'embed case').
         * @see http://en.wikipedia.org/wiki/Cross-origin_resource_sharing.
         * this is equivalent to the use_cors option in openerpframework.js
         */
    };

    /**
     * Load the PDF document
     * @param (optional) url : the url of the document to load
     */
    PDFSlidesViewer.prototype.loadDocument = function(url) {
        var self = this;
        var pdf_url = url || this.pdf_url;
        return window.pdfjsLib.getDocument(pdf_url).then(function (file_content) {
            self.pdf = file_content;
            self.pdf_page_total = file_content.numPages;
            return file_content;
        });
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

## File: static\src\components\activity\activity_tests.js

```javascript
odoo.define('website_slides/static/src/tests/activity_tests.js', function (require) {
'use strict';

const components = {
    Activity: require('mail/static/src/components/activity/activity.js'),
};

const {
    afterEach,
    beforeEach,
    createRootComponent,
    start,
} = require('mail/static/src/utils/test_utils.js');

QUnit.module('website_slides', {}, function () {
QUnit.module('components', {}, function () {
QUnit.module('activity', {}, function () {
QUnit.module('activity_tests.js', {
    beforeEach() {
        beforeEach(this);

        this.createActivityComponent = async activity => {
            await createRootComponent(this, components.Activity, {
                props: { activityLocalId: activity.localId },
                target: this.widget.el,
            });
        };

        this.start = async params => {
            const { env, widget } = await start(Object.assign({}, params, {
                data: this.data,
            }));
            this.env = env;
            this.widget = widget;
        };
    },
    afterEach() {
        afterEach(this);
    },
});

QUnit.test('grant course access', async function (assert) {
    assert.expect(8);

    await this.start({
        async mockRPC(route, args) {
            if (args.method === 'action_grant_access') {
                assert.strictEqual(args.args.length, 1);
                assert.strictEqual(args.args[0].length, 1);
                assert.strictEqual(args.args[0][0], 100);
                assert.strictEqual(args.kwargs.partner_id, 5);
                assert.step('access_grant');
            }
            return this._super(...arguments);
        },
    });
    const activity = this.env.models['mail.activity'].create({
        id: 100,
        canWrite: true,
        thread: [['insert', {
            id: 100,
            model: 'slide.channel',
        }]],
        requestingPartner: [['insert', {
            id: 5,
            displayName: "Pauvre pomme",
        }]],
        type: [['insert', {
            id: 1,
            displayName: "Access Request",
        }]],
    });
    await this.createActivityComponent(activity);

    assert.containsOnce(document.body, '.o_Activity', "should have activity component");
    assert.containsOnce(document.body, '.o_Activity_grantAccessButton', "should have grant access button");

    document.querySelector('.o_Activity_grantAccessButton').click();
    assert.verifySteps(['access_grant'], "Grant button should trigger the right rpc call");
});

QUnit.test('refuse course access', async function (assert) {
    assert.expect(8);

    await this.start({
        async mockRPC(route, args) {
            if (args.method === 'action_refuse_access') {
                assert.strictEqual(args.args.length, 1);
                assert.strictEqual(args.args[0].length, 1);
                assert.strictEqual(args.args[0][0], 100);
                assert.strictEqual(args.kwargs.partner_id, 5);
                assert.step('access_refuse');
            }
            return this._super(...arguments);
        },
    });
    const activity = this.env.models['mail.activity'].create({
        id: 100,
        canWrite: true,
        thread: [['insert', {
            id: 100,
            model: 'slide.channel',
        }]],
        requestingPartner: [['insert', {
            id: 5,
            displayName: "Pauvre pomme",
        }]],
        type: [['insert', {
            id: 1,
            displayName: "Access Request",
        }]],
    });
    await this.createActivityComponent(activity);

    assert.containsOnce(document.body, '.o_Activity', "should have activity component");
    assert.containsOnce(document.body, '.o_Activity_refuseAccessButton', "should have refuse access button");

    document.querySelector('.o_Activity_refuseAccessButton').click();
    assert.verifySteps(['access_refuse'], "refuse button should trigger the right rpc call");
});

});
});
});

});

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

## File: static\src\js\activity.js

```javascript
odoo.define('website_slides.Activity', function (require) {
"use strict";

var field_registry = require('web.field_registry');

require('mail.Activity');

var KanbanActivity = field_registry.get('kanban_activity');

function applyInclude(Activity) {
    Activity.include({
        events: _.extend({}, Activity.prototype.events, {
            'click .o_activity_action_grant_access': '_onGrantAccess',
            'click .o_activity_action_refuse_access': '_onRefuseAccess',
        }),

        _onGrantAccess: function (event) {
            var self = this;
            var partnerId = $(event.currentTarget).data('partner-id');
            this._rpc({
                model: 'slide.channel',
                method: 'action_grant_access',
                args: [this.res_id, partnerId],
            }).then(function (result) {
                self.trigger_up('reload');
            });
        },

        _onRefuseAccess: function (event) {
            var self = this;
            var partnerId = $(event.currentTarget).data('partner-id');
            this._rpc({
                model: 'slide.channel',
                method: 'action_refuse_access',
                args: [this.res_id, partnerId],
            }).then(function () {
                self.trigger_up('reload');
            });
        },
    });
}

applyInclude(KanbanActivity);

});

odoo.define('website_slides/static/src/components/activity/activity.js', function (require) {
'use strict';

const components = {
    Activity: require('mail/static/src/components/activity/activity.js'),
};
const { patch } = require('web.utils');

patch(components.Activity, 'website_slides/static/src/components/activity/activity.js', {

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    async _onGrantAccess(ev) {
        await this.env.services.rpc({
            model: 'slide.channel',
            method: 'action_grant_access',
            args: [[this.activity.thread.id]],
            kwargs: { partner_id: this.activity.requestingPartner.id },
        });
        this.trigger('reload');
    },
    /**
     * @private
     */
    async _onRefuseAccess(ev) {
        await this.env.services.rpc({
            model: 'slide.channel',
            method: 'action_refuse_access',
            args: [[this.activity.thread.id]],
            kwargs: { partner_id: this.activity.requestingPartner.id },
        });
        this.trigger('reload');
    },
});

});

```

## File: static\src\js\rating_field_backend.js

```javascript
odoo.define('website_slides.ratingField', function (require) {
"use strict";

var basicFields = require('web.basic_fields');
var fieldRegistry = require('web.field_registry');

var core = require('web.core');

var QWeb = core.qweb;

var FieldFloatRating = basicFields.FieldFloat.extend({
    xmlDependencies: !basicFields.FieldFloat.prototype.xmlDependencies ?
        ['/portal_rating/static/src/xml/portal_tools.xml'] : basicFields.FieldFloat.prototype.xmlDependencies.concat(
            ['/portal_rating/static/src/xml/portal_tools.xml']
        ),
    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     * @private
     */
    _render: function () {
        var self = this;

        return Promise.resolve(this._super()).then(function () {
            self.$el.html(QWeb.render('portal_rating.rating_stars_static', {
                'val': self.value / 2,
                'inline_mode': true
            }));
        });
    },
});

fieldRegistry.add('field_float_rating', FieldFloatRating);

return {
    FieldFloatRating: FieldFloatRating,
};

});

```

## File: static\src\js\slides.js

```javascript
odoo.define('website_slides.slides', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var time = require('web.time');

publicWidget.registry.websiteSlides = publicWidget.Widget.extend({
    selector: '#wrapwrap',

    /**
     * @override
     * @param {Object} parent
     */
    start: function (parent) {
        var defs = [this._super.apply(this, arguments)];

        _.each($("timeago.timeago"), function (el) {
            var datetime = $(el).attr('datetime');
            var datetimeObj = time.str_to_datetime(datetime);
            // if presentation 7 days, 24 hours, 60 min, 60 second, 1000 millis old(one week)
            // then return fix formate string else timeago
            var displayStr = '';
            if (datetimeObj && new Date().getTime() - datetimeObj.getTime() > 7 * 24 * 60 * 60 * 1000) {
                displayStr = moment(datetimeObj).format('ll');
            } else {
                displayStr = moment(datetimeObj).fromNow();
            }
            $(el).text(displayStr);
        });

        return Promise.all(defs);
    },
});

return publicWidget.registry.websiteSlides;

});

//==============================================================================

odoo.define('website_slides.slides_embed', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
require('website_slides.slides');

var SlideSocialEmbed = publicWidget.Widget.extend({
    events: {
        'change input': '_onChangePage',
    },
    /**
     * @constructor
     * @param {Object} parent
     * @param {Number} maxPage
     */
    init: function (parent, maxPage) {
        this._super.apply(this, arguments);
        this.max_page = maxPage || false;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Number} page
     */
    _updateEmbeddedCode: function (page) {
        var $embedInput = this.$('.slide_embed_code');
        var newCode = $embedInput.val().replace(/(page=).*?([^\d]+)/, '$1' + page + '$2');
        $embedInput.val(newCode);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Object} ev
     */
    _onChangePage: function (ev) {
        ev.preventDefault();
        var input = this.$('input');
        var page = parseInt(input.val());
        if (this.max_page && !(page > 0 && page <= this.max_page)) {
            page = 1;
        }
        this._updateEmbeddedCode(page);
    },
});

publicWidget.registry.websiteSlidesEmbed = publicWidget.Widget.extend({
    selector: '#wrapwrap',

    /**
     * @override
     * @param {Object} parent
     */
    start: function (parent) {
        var defs = [this._super.apply(this, arguments)];
        $('iframe.o_wslides_iframe_viewer').on('ready', this._onIframeViewerReady.bind(this));
        return Promise.all(defs);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onIframeViewerReady: function (ev) {
        // TODO : make it work. For now, once the iframe is loaded, the value of #page_count is
        // still now set (the pdf is still loading)
        var $iframe = $(ev.currentTarget);
        var maxPage = $iframe.contents().find('#page_count').val();
        new SlideSocialEmbed(this, maxPage).attachTo($('.oe_slide_js_embed_code_widget'));
    },
});

});

```

## File: static\src\js\slides_category_add.js

```javascript
odoo.define('website_slides.category.add', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var Dialog = require('web.Dialog');
var core = require('web.core');
var _t = core._t;

var CategoryAddDialog = Dialog.extend({
    template: 'slides.category.add',

    /**
     * @override
     */
    init: function (parent, options) {
        options = _.defaults(options || {}, {
            title: _t('Add a section'),
            size: 'medium',
            buttons: [{
                text: _t('Save'),
                classes: 'btn-primary',
                click: this._onClickFormSubmit.bind(this)
            }, {
                text: _t('Discard'),
                close: true
            }]
        });

        this.channelId = options.channelId;
        this._super(parent, options);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _formValidate: function ($form) {
        $form.addClass('was-validated');
        return $form[0].checkValidity();
    },

    _onClickFormSubmit: function (ev) {
        var $form = this.$('#slide_category_add_form');
        if (this._formValidate($form)) {
            $form.submit();
        }
    },
});

publicWidget.registry.websiteSlidesCategoryAdd = publicWidget.Widget.extend({
    selector: '.o_wslides_js_slide_section_add',
    xmlDependencies: ['/website_slides/static/src/xml/slide_management.xml'],
    events: {
        'click': '_onAddSectionClick',
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _openDialog: function (channelId) {
        new CategoryAddDialog(this, {channelId: channelId}).open();
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

return {
    categoryAddDialog: CategoryAddDialog,
    websiteSlidesCategoryAdd: publicWidget.registry.websiteSlidesCategoryAdd
};

});

```

## File: static\src\js\slides_course_enroll_email.js

```javascript
odoo.define('website_slides.course.enroll', function (require) {
'use strict';

var core = require('web.core');
var Dialog = require('web.Dialog');
var publicWidget = require('web.public.widget');
var _t = core._t;

var SlideEnrollDialog = Dialog.extend({
    template: 'slide.course.join.request',

    init: function (parent, options, modalOptions) {
        modalOptions = _.defaults(modalOptions || {}, {
            title: _t('Request Access.'),
            size: 'medium',
            buttons: [{
                text: _t('Yes'),
                classes: 'btn-primary',
                click: this._onSendRequest.bind(this)
            }, {
                text: _t('Cancel'),
                close: true
            }]
        });
        this.$element = options.$element;
        this.channelId = options.channelId;
        this._super(parent, modalOptions);
    },

    _onSendRequest: function () {
        var self = this;
        this._rpc({
            model: 'slide.channel',
            method: 'action_request_access',
            args: [self.channelId]
        }).then(function (result) {
            if (result.error) {
                self.$element.replaceWith('<div class="alert alert-danger" role="alert"><strong>' + result.error + '</strong></div>');
            } else if (result.done) {
                self.$element.replaceWith('<div class="alert alert-success" role="alert"><strong>' + _t('Request sent !') + '</strong></div>');
            } else {
                self.$element.replaceWith('<div class="alert alert-danger" role="alert"><strong>' + _t('Unknown error, try again.') + '</strong></div>');
            }
            self.close();
        });
    }
    
});

publicWidget.registry.websiteSlidesEnroll = publicWidget.Widget.extend({
    selector: '.o_wslides_js_channel_enroll',
    xmlDependencies: ['/website_slides/static/src/xml/slide_course_join.xml'],
    events: {
        'click': '_onSendRequestClick',
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------
    
    _openDialog: function (channelId) {
        new SlideEnrollDialog(this, {
            channelId: channelId,
            $element: this.$el
        }).open();
    },
    
    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------
    
    _onSendRequestClick: function (ev) {
        ev.preventDefault();
        this._openDialog($(ev.currentTarget).data('channelId'));
    }
});

return {
    slideEnrollDialog: SlideEnrollDialog,
    websiteSlidesEnroll: publicWidget.registry.websiteSlidesEnroll
};

});

```

## File: static\src\js\slides_course_fullscreen_player.js

```javascript
var onYouTubeIframeAPIReady = undefined;

odoo.define('website_slides.fullscreen', function (require) {
    'use strict';

    var publicWidget = require('web.public.widget');
    var core = require('web.core');
    var config = require('web.config');
    var QWeb = core.qweb;
    var _t = core._t;

    var session = require('web.session');

    var Quiz = require('website_slides.quiz').Quiz;

    var Dialog = require('web.Dialog');

    require('website_slides.course.join.widget');

    /**
     * Helper: Get the slide dict matching the given criteria
     *
     * @private
     * @param {Array<Object>} slideList List of dict reprensenting a slide
     * @param {Object} matcher (see https://underscorejs.org/#matcher)
     */
    var findSlide = function (slideList, matcher) {
        var slideMatch = _.matcher(matcher);
        return _.find(slideList, slideMatch);
    };

    /**
     * This widget is responsible of display Youtube Player
     *
     * The widget will trigger an event `change_slide` when the video is at
     * its end, and `slide_completed` when the player is at 30 sec before the
     * end of the video (30 sec before is considered as completed).
     */
    var VideoPlayer = publicWidget.Widget.extend({
        template: 'website.slides.fullscreen.video',
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
                    onYouTubeIframeAPIReady = function () {
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
                        if (!self.slide.hasQuestion && !self.slide.completed){
                            self.trigger_up('slide_to_complete', self.slide);
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
            "click .o_wslides_fs_sidebar_list_item": '_onClickTab',
        },
        init: function (parent, slideList, defaultSlide) {
            var result = this._super.apply(this, arguments);
            this.slideEntries = slideList;
            this.set('slideEntry', defaultSlide);
            return result;
        },
        start: function (){
            var self = this;
            this.on('change:slideEntry', this, this._onChangeCurrentSlide);
            return this._super.apply(this, arguments).then(function (){
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
                this.set('slideEntry', this.slideEntries[currentIndex+1]);
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
                this.set('slideEntry', this.slideEntries[currentIndex-1]);
            }
        },
        /**
         * Greens up the bullet when the slide is completed
         *
         * @public
         * @param {Integer} slideId
         */
        setSlideCompleted: function (slideId) {
            var $elem = this.$('.fa-circle-thin[data-slide-id="'+slideId+'"]');
            $elem.removeClass('fa-circle-thin').addClass('fa-check text-success o_wslides_slide_completed');
        },
        /**
         * Updates the progressbar whenever a lesson is completed
         *
         * @public
         * @param {*} channelCompletion
         */
        updateProgressbar: function (channelCompletion) {
            var completion = Math.min(100, channelCompletion);
            this.$('.progress-bar').css('width', completion + "%" );
            this.$('.o_wslides_progress_percentage').text(completion);
        },

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------
        /**
         * Get the index of the current slide entry (slide and/or quiz)
         */
        _getCurrentIndex: function () {
            var slide = this.get('slideEntry');
            var currentIndex = _.findIndex(this.slideEntries, function (entry) {
                return entry.id === slide.id && entry.isQuiz === slide.isQuiz;
            });
            return currentIndex;
        },
        //--------------------------------------------------------------------------
        // Handler
        //--------------------------------------------------------------------------
        /**
         * Handler called whenever the user clicks on a sub-quiz which is linked to a slide.
         * This does NOT handle the case of a slide of type "quiz".
         * By going through this handler, the widget will be able to determine that it has to render
         * the associated quiz and not the main content.
         *
         * @private
         * @param {*} ev
         */
        _onClickMiniQuiz: function (ev){
            var slideID = parseInt($(ev.currentTarget).data().slide_id);
            this.set('slideEntry',{
                slideID: slideID,
                isMiniQuiz: true
            });
            this.trigger_up('change_slide', this.get('slideEntry'));
        },
        /**
         * Handler called when the user clicks on a normal slide tab
         *
         * @private
         * @param {*} ev
         */
        _onClickTab: function (ev) {
            ev.stopPropagation();
            var $elem = $(ev.currentTarget);
            if ($elem.data('canAccess') === 'True') {
                var isQuiz = $elem.data('isQuiz');
                var slideID = parseInt($elem.data('id'));
                var slide = findSlide(this.slideEntries, {id: slideID, isQuiz: isQuiz});
                this.set('slideEntry', slide);
            }
        },
        /**
         * Actively changes the active tab in the sidebar so that it corresponds
         * the slide currently displayed
         *
         * @private
         */
        _onChangeCurrentSlide: function () {
            var slide = this.get('slideEntry');
            this.$('.o_wslides_fs_sidebar_list_item.active').removeClass('active');
            var selector = '.o_wslides_fs_sidebar_list_item[data-id='+slide.id+'][data-is-quiz!="1"]';

            this.$(selector).addClass('active');
            this.trigger_up('change_slide', this.get('slideEntry'));
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

    var ShareDialog = Dialog.extend({
        template: 'website.slide.share.modal',
        events: {
            'click .o_wslides_js_share_email button': '_onShareByEmailClick',
            'click a.o_wslides_js_social_share': '_onSlidesSocialShare',
            'click .o_clipboard_button': '_onShareLinkCopy',
        },

        init: function (parent, options, slide) {
            options = _.defaults(options || {}, {
                title: "Share",
                buttons: [{text: "Cancel", close: true}],
                size: 'medium',
            });
            this._super(parent, options);
            this.slide = slide;
            this.session = session;
        },

        _onShareByEmailClick: function() {
            var form = this.$('.o_wslides_js_share_email');
            var input = form.find('input');
            var slideID = form.find('button').data('slide-id');
            if (input.val() && input[0].checkValidity()) {
                form.removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');
                this._rpc({
                    route: '/slides/slide/send_share_email',
                    params: {
                        slide_id: slideID,
                        email: input.val(),
                        fullscreen: true
                    },
                }).then(function () {
                    form.html('<div class="alert alert-info" role="alert">' + _t('<strong>Thank you!</strong> Mail has been sent.') + '</div>');
                });
            } else {
                form.addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
                input.focus();
            }
        },

        _onSlidesSocialShare: function (ev) {
            ev.preventDefault();
            var popUpURL = $(ev.currentTarget).attr('href');
            window.open(popUpURL, 'Share Dialog', 'width=626,height=436');
        },

        _onShareLinkCopy: function (ev) {
            ev.preventDefault();
            var $clipboardBtn = this.$('.o_clipboard_button');
            $clipboardBtn.tooltip({title: "Copied !", trigger: "manual", placement: "bottom"});
            var self = this;
            var clipboard = new ClipboardJS('.o_clipboard_button', {
                target: function () {
                    return self.$('.o_wslides_js_share_link')[0];
                },
                container: this.el
            });
            clipboard.on('success', function () {
                clipboard.destroy();
                $clipboardBtn.tooltip('show');
                _.delay(function () {
                    $clipboardBtn.tooltip("hide");
                }, 800);
            });
            clipboard.on('error', function (e) {
                clipboard.destroy();
            })
        },

    });

    var ShareButton = publicWidget.Widget.extend({
        events: {
            "click .o_wslides_fs_share": '_onClickShareSlide'
        },

        init: function (el, slide) {
            var result = this._super.apply(this, arguments);
            this.slide = slide;
            return result;
        },

        _openDialog: function() {
            return new ShareDialog(this, {}, this.slide).open();
        },

        _onClickShareSlide: function (ev) {
            ev.preventDefault();
            this._openDialog();
        },

        _onChangeSlide: function (currentSlide) {
            this.slide = currentSlide;
        }

    });

    /**
     * This widget's purpose is to show content of a course, naviguating through contents
     * and correclty display it. It also handle slide completion, course progress, ...
     *
     * This widget is rendered sever side, and attached to the existing DOM.
     */
    var Fullscreen = publicWidget.Widget.extend({
        events: {
            "click .o_wslides_fs_toggle_sidebar": '_onClickToggleSidebar',
        },
        custom_events: {
            'change_slide': '_onChangeSlideRequest',
            'slide_to_complete': '_onSlideToComplete',
            'slide_completed': '_onSlideCompleted',
            'slide_go_next': '_onSlideGoToNext',
        },
        /**
        * @override
        * @param {Object} el
        * @param {Object} slides Contains the list of all slides of the course
        * @param {integer} defaultSlideId Contains the ID of the slide requested by the user
        */
        init: function (parent, slides, defaultSlideId, channelData){
            var result = this._super.apply(this,arguments);
            this.initialSlideID = defaultSlideId;
            this.slides = this._preprocessSlideData(slides);
            this.channel = channelData;
            var slide;
            var urlParams = $.deparam.querystring();
            if (defaultSlideId) {
                slide = findSlide(this.slides, {id: defaultSlideId, isQuiz: urlParams.quiz === "1" });
            } else {
                slide = this.slides[0];
            }

            this.set('slide', slide);

            this.sidebar = new Sidebar(this, this.slides, slide);
            this.shareButton = new ShareButton(this, slide);
            return result;
        },
        /**
         * @override
         */
        start: function (){
            var self = this;
            this.on('change:slide', this, this._onChangeSlide);
            this._toggleSidebar();
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
            defs.push(this.shareButton.attachTo(this.$('.o_wslides_slide_fs_header')));
            return $.when.apply($, defs);
        },
        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------
        /**
         * Fetches content with an rpc call for slides of type "webpage"
         *
         * @private
         */
        _fetchHtmlContent: function (){
            var self = this;
            var currentSlide = this.get('slide');
            return self._rpc({
                route:"/slides/slide/get_html_content",
                params: {
                    'slide_id': currentSlide.id
                }
            }).then(function (data){
                if (data.html_content) {
                    currentSlide.htmlContent = data.html_content;
                }
            });
        },
        /**
        * Fetches slide content depending on its type.
        * If the slide doesn't need to fetch any content, return a resolved deferred
        *
        * @private
        */
        _fetchSlideContent: function (){
            var slide = this.get('slide');
            if (slide.type === 'webpage' && !slide.isQuiz) {
                return this._fetchHtmlContent();
            }
            return Promise.resolve();
        },
        _markAsCompleted: function (slideId, completion) {
            var slide = findSlide(this.slides, {id: slideId});
            slide.completed = true;
            this.sidebar.setSlideCompleted(slide.id);
            this.sidebar.updateProgressbar(completion);
        },
        /**
         * Extend the slide data list to add informations about rendering method, and other
         * specific values according to their slide_type.
         */
        _preprocessSlideData: function (slidesDataList) {
            slidesDataList.forEach(function (slideData, index) {
                // compute hasNext slide
                slideData.hasNext = index < slidesDataList.length-1;
                // compute embed url
                if (slideData.type === 'video') {
                    slideData.embedCode = $(slideData.embedCode).attr('src') || ""; // embedCode contains an iframe tag, where src attribute is the url (youtube or embed document from odoo)
                    var separator = slideData.embedCode.indexOf("?") !== -1 ? "&" : "?";
                    var scheme = slideData.embedCode.indexOf('//') === 0 ? 'https:' : '';
                    var params = { rel: 0, enablejsapi: 1, origin: window.location.origin };
                    if (slideData.embedCode.indexOf("//drive.google.com") === -1) {
                        params.autoplay = 1;
                    }
                    slideData.embedUrl = slideData.embedCode ? scheme + slideData.embedCode + separator + $.param(params) : "";
                } else if (slideData.type === 'infographic') {
                    slideData.embedUrl = _.str.sprintf('/web/image/slide.slide/%s/image_1024', slideData.id);
                } else if (_.contains(['document', 'presentation'], slideData.type)) {
                    slideData.embedUrl = $(slideData.embedCode).attr('src');
                }
                // fill empty property to allow searching on it with _.filter(list, matcher)
                slideData.isQuiz = !!slideData.isQuiz;
                slideData.hasQuestion = !!slideData.hasQuestion;
                // technical settings for the Fullscreen to work
                slideData._autoSetDone = _.contains(['infographic', 'presentation', 'document', 'webpage'], slideData.type) && !slideData.hasQuestion;
            });
            return slidesDataList;
        },
        /**
         * Changes the url whenever the user changes slides.
         * This allows the user to refresh the page and stay on the right slide
         *
         * @private
         */
        _pushUrlState: function (){
            var urlParts = window.location.pathname.split('/');
            urlParts[urlParts.length-1] = this.get('slide').slug;
            var url =  urlParts.join('/');
            this.$('.o_wslides_fs_exit_fullscreen').attr('href', url);
            var params = {'fullscreen': 1 };
            if (this.get('slide').isQuiz){
                params.quiz = 1;
            }
            var fullscreenUrl = _.str.sprintf('%s?%s', url, $.param(params));
            history.pushState(null, '', fullscreenUrl);
        },
        /**
         * Render the current slide content using specific mecanism according to slide type:
         * - simply append content (for webpage)
         * - template rendering (for image, document, ....)
         * - using a sub widget (quiz and video)
         *
         * @private
         * @returns Deferred
         */
        _renderSlide: function () {
            var slide = this.get('slide');
            var $content = this.$('.o_wslides_fs_content');
            $content.empty();

            // display quiz slide, or quiz attached to a slide
            if (slide.type === 'quiz' || slide.isQuiz) {
                $content.addClass('bg-white');
                var QuizWidget = new Quiz(this, slide, this.channel);
                return QuizWidget.appendTo($content);
            }

            // render slide content
            if (_.contains(['document', 'presentation', 'infographic'], slide.type)) {
                $content.html(QWeb.render('website.slides.fullscreen.content', {widget: this}));
            } else if (slide.type === 'video') {
                this.videoPlayer = new VideoPlayer(this, slide);
                return this.videoPlayer.appendTo($content);
            } else if (slide.type === 'webpage'){
                var $wpContainer = $('<div>').addClass('o_wslide_fs_webpage_content bg-white block w-100 overflow-auto');
                $(slide.htmlContent).appendTo($wpContainer);
                $content.append($wpContainer);
                this.trigger_up('widgets_start_request', {
                    $target: $content,
                });
            }
            return Promise.resolve();
        },
        /**
         * Once the completion conditions are filled,
         * rpc call to set the the relation between the slide and the user as "completed"
         *
         * @private
         * @param {Integer} slideId: the id of slide to set as completed
         */
        _setCompleted: function (slideId){
            var self = this;
            var slide = findSlide(this.slides, {id: slideId});
            if (!slide.completed) {  // no useless RPC call
                return this._rpc({
                    route: '/slides/slide/set_completed',
                    params: {
                        slide_id: slide.id,
                    }
                }).then(function (data){
                    self._markAsCompleted(slideId, data.channel_completion);
                    return Promise.resolve();
                });
            }
            return Promise.resolve();
        },
        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------
        /**
         * Triggered whenever the user changes slides.
         * When the current slide is changed, widget will be automatically updated
         * and allowed to: fetch the content if needed, render it, update the url,
         * and set slide as "completed" according to its type requirements. In
         * mobile case (i.e. limited screensize), sidebar will be toggled since 
         * sidebar will block most or all of new slide visibility.
         *
         * @private
         */
        _onChangeSlide: function () {
            var self = this;
            var slide = this.get('slide');
            self._pushUrlState();
            return this._fetchSlideContent().then(function() { // render content
                var websiteName = document.title.split(" | ")[1]; // get the website name from title
                document.title =  (websiteName) ? slide.name + ' | ' + websiteName : slide.name;
                if  (config.device.size_class < config.device.SIZES.MD) {
                    self._toggleSidebar(); // hide sidebar when small device screen
                }
                return self._renderSlide();
            }).then(function() {
                if (slide._autoSetDone && !session.is_website_user) {  // no useless RPC call
                    if (['document', 'presentation'].includes(slide.type)) {
                        // only set the slide as completed after iFrame is loaded to avoid concurrent execution with 'embedUrl' controller
                        self.el.querySelector('iframe.o_wslides_iframe_viewer').addEventListener('load', () => self._setCompleted(slide.id));
                    } else {
                           return self._setCompleted(slide.id);
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
        _onChangeSlideRequest: function (ev){
            var slideData = ev.data;
            var newSlide = findSlide(this.slides, {
                id: slideData.id,
                isQuiz: slideData.isQuiz || false,
            });
            this.set('slide', newSlide);
            this.shareButton._onChangeSlide(newSlide);
        },
        /**
         * Triggered when subwidget has mark the slide as done, and the UI need to be adapted.
         *
         * @private
         */
        _onSlideCompleted: function (ev) {
            var slide = ev.data.slide;
            var completion = ev.data.completion;
            this._markAsCompleted(slide.id, completion);
        },
        /**
         * Triggered when sub widget business is done and that slide
         * can now be marked as done.
         *
         * @private
         */
        _onSlideToComplete: function (ev) {
            if (!session.is_website_user) {  // no useless RPC call
                var slideId = ev.data.id;
                this._setCompleted(slideId);
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
        xmlDependencies: ['/website_slides/static/src/xml/website_slides_fullscreen.xml', '/website_slides/static/src/xml/website_slides_share.xml'],
        start: function (){
            var self = this;
            var proms = [this._super.apply(this, arguments)];
            var fullscreen = new Fullscreen(this, this._getSlides(), this._getCurrentSlideID(), this._extractChannelData());
            proms.push(fullscreen.attachTo(".o_wslides_fs_main"));
            return Promise.all(proms).then(function () {
                $('#edit-page-menu a[data-action="edit"]').on('click', self._onWebEditorClick.bind(self));
            });
        },

        /**
         * The web editor does not work well with the e-learning fullscreen view.
         * It actually completely closes the fullscreen view and opens the edition on a blank page.
         *
         * To avoid this, we intercept the click on the 'edit' button and redirect to the
         * non-fullscreen view of this slide with the editor enabled, which is more suited to edit
         * in-place anyway.
         *
         * @param {MouseEvent} e
         */
        _onWebEditorClick: function (e) {
            e.preventDefault();
            e.stopPropagation();

            window.location = `${window.location.pathname}?fullscreen=0&enable_editor=1`;
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

    return Fullscreen;
});

```

## File: static\src\js\slides_course_join.js

```javascript
odoo.define('website_slides.course.join.widget', function (require) {
'use strict';

var core = require('web.core');
var publicWidget = require('web.public.widget');

var _t = core._t;

var CourseJoinWidget = publicWidget.Widget.extend({
    template: 'slide.course.join',
    xmlDependencies: ['/website_slides/static/src/xml/slide_course_join.xml'],
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
     * @param {boolean} options.isMember whether current user is member or not
     * @param {boolean} options.publicUser whether current user is public or not
     * @param {string} [options.joinMessage] the message to use for the simple join case
     *   when the course if free and the user is logged in, defaults to "Join Course".
     * @param {Promise} [options.beforeJoin] a promise to execute before we redirect to
     *   another url within the join process (login / buy course / ...)
     * @param {function} [options.afterJoin] a callback function called after the user has
     *   joined the course
     */
    init: function (parent, options) {
        this._super.apply(this, arguments);
        this.channel = options.channel;
        this.isMember = options.isMember;
        this.publicUser = options.publicUser;
        this.joinMessage = options.joinMessage || _t('Join Course');
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

        if (this.channel.channelEnroll !== 'invite') {
            if (this.publicUser) {
                this.beforeJoin().then(this._redirectToLogin.bind(this));
            } else if (!this.isMember && this.channel.channelEnroll === 'public') {
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
            url = `/slides/${this.channel.channelId}`;
        }
        document.location = _.str.sprintf('/web/login?redirect=%s', encodeURIComponent(url));
    },

    /**
     * @private
     * @param {Object} $el
     * @param {String} message
     */
    _popoverAlert: function ($el, message) {
        $el.popover({
            trigger: 'focus',
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
        this._rpc({
            route: '/slides/channel/join',
            params: {
                channel_id: channelId,
            },
        }).then(function (data) {
            if (!data.error) {
                self.afterJoin();
            } else {
                if (data.error === 'public_user') {
                    var message = _t('Please <a href="/web/login?redirect=%s">login</a> to join this course');
                    var signupAllowed = data.error_signup_allowed || false;
                    if (signupAllowed) {
                        message = _t('Please <a href="/web/signup?redirect=%s">create an account</a> to join this course');
                    }
                    self._popoverAlert(self.$el, _.str.sprintf(message, encodeURIComponent(document.URL)));
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
        var options = {channel: {channelEnroll: data.channelEnroll, channelId: data.channelId}};
        $('.o_wslides_js_course_join').each(function () {
            proms.push(new CourseJoinWidget(self, options).attachTo($(this)));
        });
        return Promise.all(proms);
    },
});

return {
    courseJoinWidget: CourseJoinWidget,
    websiteSlidesCourseJoin: publicWidget.registry.websiteSlidesCourseJoin
};

});

```

## File: static\src\js\slides_course_quiz.js

```javascript
odoo.define('website_slides.quiz', function (require) {
    'use strict';

    var publicWidget = require('web.public.widget');
    var Dialog = require('web.Dialog');
    var core = require('web.core');
    var session = require('web.session');

    var CourseJoinWidget = require('website_slides.course.join.widget').courseJoinWidget;
    var QuestionFormWidget = require('website_slides.quiz.question.form');
    var SlideQuizFinishModal = require('website_slides.quiz.finish');

    var SlideEnrollDialog = require('website_slides.course.enroll').slideEnrollDialog;

    var QWeb = core.qweb;
    var _t = core._t;

    /**
     * This widget is responsible of displaying quiz questions and propositions. Submitting the quiz will fetch the
     * correction and decorate the answers according to the result. Error message or modal can be displayed.
     *
     * This widget can be attached to DOM rendered server-side by `website_slides.slide_type_quiz` or
     * used client side (Fullscreen).
     *
     * Triggered events are :
     * - slide_go_next: need to go to the next slide, when quiz is done. Event data contains the current slide id.
     * - quiz_completed: when the quiz is passed and completed by the user. Event data contains current slide data.
     */
    var Quiz = publicWidget.Widget.extend({
        template: 'slide.slide.quiz',
        xmlDependencies: [
            '/website_slides/static/src/xml/slide_quiz.xml',
            '/website_slides/static/src/xml/slide_course_join.xml'
        ],
        events: {
            "click .o_wslides_quiz_answer": '_onAnswerClick',
            "click .o_wslides_js_lesson_quiz_submit": '_submitQuiz',
            "click .o_wslides_quiz_modal_btn": '_onClickNext',
            "click .o_wslides_quiz_continue": '_onClickNext',
            "click .o_wslides_js_lesson_quiz_reset": '_onClickReset',
            'click .o_wslides_js_quiz_add': '_onCreateQuizClick',
            'click .o_wslides_js_quiz_edit_question': '_onEditQuestionClick',
            'click .o_wslides_js_quiz_delete_question': '_onDeleteQuestionClick',
            'click .o_wslides_js_channel_enroll': '_onSendRequestToResponsibleClick',
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
            this.slide = _.defaults(slide_data, {
                id: 0,
                name: '',
                hasNext: false,
                completed: false,
                isMember: false,
            });
            this.quiz = quiz_data || false;
            if (this.quiz) {
                this.quiz.questionsCount = quiz_data.questions.length;
            }
            this.isMember = slide_data.isMember || false;
            this.publicUser = session.is_website_user;
            this.userId = session.user_id;
            this.redirectURL = encodeURIComponent(document.URL);
            this.channel = channel_data;
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
         * his answers (saved into his session) here as well.
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

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        _alertShow: function (alertCode) {
            var message = _t('There was an error validating this quiz.');
            if (alertCode === 'slide_quiz_incomplete') {
                message = _t('All questions must be answered !');
            } else if (alertCode === 'slide_quiz_done') {
                message = _t('This quiz is already done. Retaking it is not possible.');
            } else if (alertCode === 'public_user') {
                message = _t('You must be logged to submit the quiz.');
            }

            this.displayNotification({
                type: 'warning',
                message: message,
                sticky: true
            });
        },

        /**
         * Allows to reorder the questions
         * @private
         */
        _bindSortable: function () {
            this.$el.sortable({
                handle: '.o_wslides_js_quiz_sequence_handler',
                items: '.o_wslides_js_lesson_quiz_question',
                stop: this._reorderQuestions.bind(this),
                placeholder: 'o_wslides_js_quiz_sequence_highlight position-relative my-3'
            });
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
            this._rpc({
                route: '/web/dataset/resequence',
                params: {
                    model: "slide.question",
                    ids: this._getQuestionsIds()
                }
            }).then(this._modifyQuestionsSequence.bind(this))
        },
        /*
         * @private
         * Fetch the quiz for a particular slide
         */
        _fetchQuiz: function () {
            var self = this;
            return self._rpc({
                route:'/slides/slide/quiz/get',
                params: {
                    'slide_id': self.slide.id,
                }
            }).then(function (quiz_data) {
                self.quiz = {
                    questions: quiz_data.slide_questions || [],
                    questionsCount: quiz_data.slide_questions.length,
                    quizAttemptsCount: quiz_data.quiz_attempts_count || 0,
                    quizKarmaGain: quiz_data.quiz_karma_gain || 0,
                    quizKarmaWon: quiz_data.quiz_karma_won || 0,
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
                        _.contains(self.slide.sessionAnswers, $answer.data('answerId'))) {
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
            $validationElem.html(
                QWeb.render('slide.slide.quiz.validation', {'widget': this})
            );
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
        _submitQuiz: function () {
            var self = this;

            return this._rpc({
                route: '/slides/slide/quiz/submit',
                params: {
                    slide_id: self.slide.id,
                    answer_ids: this._getQuizAnswers(),
                }
            }).then(function (data) {
                if (data.error) {
                    self._alertShow(data.error);
                } else {
                    self.quiz = _.extend(self.quiz, data);
                    if (data.completed) {
                        self._disableAnswers();
                        new SlideQuizFinishModal(self, {
                            quiz: self.quiz,
                            hasNext: self.slide.hasNext,
                            userId: self.userId
                        }).open();
                        self.slide.completed = true;
                        self.trigger_up('slide_completed', {slide: self.slide, completion: data.channel_completion});
                    }
                    self._hideEditOptions();
                    self._renderAnswersHighlightingAndComments();
                    self._renderValidationInfo();
                }
            });
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
            if (window.location.href.includes('quiz_quick_create')) {
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
            this._rpc({
                route: '/slides/slide/quiz/reset',
                params: {
                    slide_id: this.slide.id
                }
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
            var quizAnswers = this._getQuizAnswers();
            if (quizAnswers.length === this.quiz.questions.length) {
                return this._rpc({
                    route: '/slides/slide/quiz/save_to_session',
                    params: {
                        'quiz_answers': {'slide_id': this.slide.id, 'slide_answers': quizAnswers},
                    }
                });
            } else {
                this._alertShow('slide_quiz_incomplete');
                return Promise.reject('The quiz is incomplete');
            }
        },
        /**
        * After joining the course, we immediately submit the quiz and get the correction.
        * This allows a smooth onboarding when the user is logged in and the course is public.
        *
        * @private
        */
       _afterJoin: function () {
            this.isMember = true;
            this._renderValidationInfo();
            this._applySessionAnswers();
            this._submitQuiz();
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
         * When clicking on the delete button of a question it
         * toggles a modal to confirm the deletion
         * @param ev
         * @private
         */
        _onDeleteQuestionClick: function (ev) {
            var question = $(ev.currentTarget).closest('.o_wslides_js_lesson_quiz_question');
            new ConfirmationDialog(this, {
                questionId: question.data('questionId'),
                questionTitle: question.data('title')
            }).open();
        },

        /**
         * Handler for the contact responsible link below a Quiz
         * @param ev
         * @private
         */
        _onSendRequestToResponsibleClick: function(ev) {
            ev.preventDefault();
            var channelId = $(ev.currentTarget).data('channelId');
            new SlideEnrollDialog(this, {
                channelId: channelId,
                $element: $(ev.currentTarget).closest('.alert.alert-info')
            }).open();
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

    /**
     * Dialog box shown when clicking the deletion button on a Question.
     * When confirming it sends a RPC request to delete the Question.
     */
    var ConfirmationDialog = Dialog.extend({
        template: 'slide.quiz.confirm.deletion',
        xmlDependencies: Dialog.prototype.xmlDependencies.concat(
            ['/website_slides/static/src/xml/slide_quiz_create.xml']
        ),

        /**
         * @override
         * @param parent
         * @param options
         */
        init: function (parent, options) {
            options = _.defaults(options || {}, {
                title: _t('Delete Question'),
                buttons: [
                    { text: _t('Yes'), classes: 'btn-primary', click: this._onConfirmClick },
                    { text: _t('No'), close: true}
                ],
                size: 'medium'
            });
            this.questionId = options.questionId;
            this.questionTitle = options.questionTitle;
            this._super.apply(this, arguments);
        },

        /**
         * Handler when the user confirm the deletion by clicking on 'Yes'
         * it sends a RPC request to the server and triggers an event to
         * visually delete the question.
         * @private
         */
        _onConfirmClick: function () {
            var self = this;
            this._rpc({
                model: 'slide.question',
                method: 'unlink',
                args: [this.questionId],
            }).then(function () {
                self.trigger_up('delete_question', { questionId: self.questionId });
                self.close();
            });
        }
    });

    publicWidget.registry.websiteSlidesQuizNoFullscreen = publicWidget.Widget.extend({
        selector: '.o_wslides_lesson_main', // selector of complete page, as we need slide content and aside content table
        custom_events: {
            slide_go_next: '_onQuizNextSlide',
            slide_completed: '_onQuizCompleted',
        },

        //----------------------------------------------------------------------
        // Public
        //----------------------------------------------------------------------

        /**
         * @override
         * @param {Object} parent
         */
        start: function () {
            var self = this;
            this.quizWidgets = [];
            var defs = [this._super.apply(this, arguments)];
            this.$('.o_wslides_js_lesson_quiz').each(function () {
                var slideData = $(this).data();
                var channelData = self._extractChannelData(slideData);
                slideData.quizData = {
                    questions: self._extractQuestionsAndAnswers(),
                    sessionAnswers: slideData.sessionAnswers || [],
                    quizKarmaMax: slideData.quizKarmaMax,
                    quizKarmaWon: slideData.quizKarmaWon || 0,
                    quizKarmaGain: slideData.quizKarmaGain,
                    quizAttemptsCount: slideData.quizAttemptsCount,
                };
                defs.push(new Quiz(self, slideData, channelData, slideData.quizData).attachTo($(this)));
            });
            return Promise.all(defs);
        },

        //----------------------------------------------------------------------
        // Handlers
        //---------------------------------------------------------------------
        _onQuizCompleted: function (ev) {
            var slide = ev.data.slide;
            var completion = ev.data.completion;
            this.$('#o_wslides_lesson_aside_slide_check_' + slide.id).addClass('text-success fa-check').removeClass('text-600 fa-circle-o');
            // need to use global selector as progress bar is outside this animation widget scope
            $('.o_wslides_lesson_header .progress-bar').css('width', completion + "%");
            $('.o_wslides_lesson_header .progress span').text(_.str.sprintf("%s %%", completion));
        },
        _onQuizNextSlide: function () {
            var url = this.$('.o_wslides_js_lesson_quiz').data('next-slide-url');
            window.location.replace(url);
        },

        //----------------------------------------------------------------------
        // Private
        //---------------------------------------------------------------------

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

    return {
        Quiz: Quiz,
        ConfirmationDialog: ConfirmationDialog,
        websiteSlidesQuizNoFullscreen: publicWidget.registry.websiteSlidesQuizNoFullscreen
    };
});

```

## File: static\src\js\slides_course_quiz_finish.js

```javascript
odoo.define('website_slides.quiz.finish', function (require) {
'use strict';

var Dialog = require('web.Dialog');
var core = require('web.core');
var _t = core._t;

/**
 * This modal is used when the user finishes the quiz.
 * It handles the animation of karma gain and leveling up by animating
 * the progress bar and the text.
 */
var SlideQuizFinishModal = Dialog.extend({
    template: 'slide.slide.quiz.finish',
    events: {
        "click .o_wslides_quiz_modal_btn": '_onClickNext',
    },

    init: function(parent, options) {
        var self = this;
        this.quiz = options.quiz;
        this.hasNext = options.hasNext;
        this.userId = options.userId;
        options = _.defaults(options || {}, {
            size: 'medium',
            dialogClass: 'd-flex p-0',
            technical: false,
            renderHeader: false,
            renderFooter: false
        });
        this._super.apply(this, arguments);
        this.opened(function () {
            self._animateProgressBar();
            self._animateText();
        })
    },

    start: function() {
        var self = this;
        this._super.apply(this, arguments).then(function () {
            self.$modal.addClass('o_wslides_quiz_modal pt-5');
            self.$modal.find('.modal-dialog').addClass('mt-5');
            self.$modal.find('.modal-content').addClass('shadow-lg');
        });
    },

    //--------------------------------
    // Handlers
    //--------------------------------

    _onClickNext: function() {
        this.trigger_up('slide_go_next');
        this.destroy();
    },

    //--------------------------------
    // Private
    //--------------------------------

    /**
     * Handles the animation of the karma gain in the following steps:
     * 1. Initiate the tooltip which will display the actual Karma
     *    over the progress bar.
     * 2. Animate the tooltip text to increment smoothly from the old
     *    karma value to the new karma value and updates it to make it
     *    move as the progress bar moves.
     * 3a. The user doesn't level up
     *    I.   When the user doesn't level up the progress bar simply goes
     *         from the old karma value to the new karma value.
     * 3b. The user levels up
     *    I.   The first step makes the progress bar go from the old karma
     *         value to 100%.
     *    II.  The second step makes the progress bar go from 100% to 0%.
     *    III. The third and final step makes the progress bar go from 0%
     *         to the new karma value. It also changes the lower and upper
     *         bound to match the new rank.
     * @param $modal
     * @param rankProgress
     * @private
     */
    _animateProgressBar: function () {
        var self = this;
        this.$('[data-toggle="tooltip"]').tooltip({
            trigger: 'manual',
            container: '.progress-bar-tooltip',
        }).tooltip('show');

        this.$('.tooltip-inner')
            .prop('karma', this.quiz.rankProgress.previous_rank.karma)
            .animate({
                karma: this.quiz.rankProgress.new_rank.karma
            }, {
                duration: this.quiz.rankProgress.level_up ? 1700 : 800,
                step: function (newKarma) {
                    self.$('.tooltip-inner').text(Math.ceil(newKarma));
                    self.$('[data-toggle="tooltip"]').tooltip('update');
                }
            }
        );

        var $progressBar = this.$('.progress-bar');
        if (this.quiz.rankProgress.level_up) {
            this.$('.o_wslides_quiz_modal_title').text(_t('Level up!'));
            $progressBar.css('width', '100%');
            _.delay(function () {
                self.$('.o_wslides_quiz_modal_rank_lower_bound')
                    .text(self.quiz.rankProgress.new_rank.lower_bound);
                self.$('.o_wslides_quiz_modal_rank_upper_bound')
                    .text(self.quiz.rankProgress.new_rank.upper_bound || "");

                // we need to use _.delay to force DOM re-rendering between 0 and new percentage
                _.delay(function () {
                    $progressBar.addClass('no-transition').width('0%');
                }, 1);
                _.delay(function () {
                    $progressBar
                        .removeClass('no-transition')
                        .width(self.quiz.rankProgress.new_rank.progress + '%');
                }, 100);
            }, 800);
        } else {
            $progressBar.css('width', this.quiz.rankProgress.new_rank.progress + '%');
        }
    },

    /**
     * Handles the animation of the different text such as the karma gain
     * and the motivational message when the user levels up.
     * @private
     */
    _animateText: function () {
        var self = this;
       _.delay(function () {
            self.$('h4.o_wslides_quiz_modal_xp_gained').addClass('show in');
            self.$('.o_wslides_quiz_modal_dismiss').removeClass('d-none');
        }, 800);

        if (this.quiz.rankProgress.level_up) {
            _.delay(function () {
                self.$('.o_wslides_quiz_modal_rank_motivational').addClass('fade');
                _.delay(function () {
                    self.$('.o_wslides_quiz_modal_rank_motivational').html(
                        self.quiz.rankProgress.last_rank ?
                            self.quiz.rankProgress.description :
                            self.quiz.rankProgress.new_rank.motivational
                    );
                    self.$('.o_wslides_quiz_modal_rank_motivational').addClass('show in');
                }, 800);
            }, 800);
        }
    },

});

return SlideQuizFinishModal;

});

```

## File: static\src\js\slides_course_quiz_question_form.js

```javascript
odoo.define('website_slides.quiz.question.form', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var core = require('web.core');

var QWeb = core.qweb;
var _t = core._t;

/**
 * This Widget is responsible of displaying the question inputs when adding a new question or when updating an
 * existing one. When validating the question it makes an RPC call to the server and trigger an event for
 * displaying the question by the Quiz widget.
 */
var QuestionFormWidget = publicWidget.Widget.extend({
    template: 'slide.quiz.question.input',
    xmlDependencies: ['/website_slides/static/src/xml/slide_quiz_create.xml'],
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
        $(ev.currentTarget).closest('.o_wslides_js_quiz_answer').after(QWeb.render('slide.quiz.answer.line'));
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
    _createOrUpdateQuestion: function (options) {
        var self = this;
        var $form = this.$('form');
        if (this._isValidForm($form)) {
            var values = this._serializeForm($form);
            this._rpc({
                route: '/slides/slide/quiz/question_add_or_update',
                params: values
            }).then(function (renderedQuestion) {
                if (options.update) {
                    self.trigger_up('display_updated_question', {
                        newQuestionRenderedTemplate: renderedQuestion,
                        $editedQuestion: self.$editedQuestion,
                        questionFormWidget: self,
                    });
                } else {
                    self.trigger_up('display_created_question', {
                        newQuestionRenderedTemplate: renderedQuestion,
                        questionFormWidget: self
                    });
                }
            });
        } else {
            this.displayNotification({
                type: 'warning',
                message: _t('Please fill in the question'),
                sticky: true
            });
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
                    'comment': $(this).find('.o_wslides_js_quiz_answer_comment > input[type=text]').val().trim()
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

return QuestionFormWidget;
});

```

## File: static\src\js\slides_course_slides_list.js

```javascript
odoo.define('website_slides.course.slides.list', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var core = require('web.core');
var _t = core._t;

publicWidget.registry.websiteSlidesCourseSlidesList = publicWidget.Widget.extend({
    selector: '.o_wslides_slides_list',
    xmlDependencies: ['/website_slides/static/src/xml/website_slides_upload.xml'],

    start: function () {
        this._super.apply(this,arguments);

        this.channelId = this.$el.data('channelId');

        this._updateHref();
        this._bindSortable();
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
        this.$('ul.o_wslides_js_slides_list_container').sortable({
            handle: '.o_wslides_slides_list_drag',
            stop: this._reorderSlides.bind(this),
            items: '.o_wslides_slide_list_category',
            placeholder: 'o_wslides_slides_list_slide_hilight position-relative mb-1'
        });

        this.$('.o_wslides_js_slides_list_container ul').sortable({
            handle: '.o_wslides_slides_list_drag',
            connectWith: '.o_wslides_js_slides_list_container ul',
            stop: this._reorderSlides.bind(this),
            items: '.o_wslides_slides_list_slide:not(.o_wslides_js_slides_list_empty)',
            placeholder: 'o_wslides_slides_list_slide_hilight position-relative mb-1'
        });
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
                    'class': "ml-1 text-muted font-weight-bold",
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
        self._rpc({
            route: '/web/dataset/resequence',
            params: {
                model: "slide.slide",
                ids: self._getSlides()
            }
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

return publicWidget.registry.websiteSlidesCourseSlidesList;

});

```

## File: static\src\js\slides_course_tag_add.js

```javascript
odoo.define('website_slides.channel_tag.add', function (require) {
'use strict';

var core = require('web.core');
var Dialog = require('web.Dialog');
var publicWidget = require('web.public.widget');

var _t = core._t;

var TagCourseDialog = Dialog.extend({
    template: 'website.slides.tag.add',
    events: _.extend({}, Dialog.prototype.events, {
        'change input#tag_id' : '_onChangeTag',
    }),

    /**
    * @override
    * @param {Object} parent
    * @param {Object} options holding channelId
    *      
    */
    init: function (parent, options) {
        options = _.defaults(options || {}, {
            title: _t("Add a tag"),
            size: 'medium',
            buttons: [{
                text: _t("Add"),
                classes: 'btn-primary',
                click: this._onClickFormSubmit.bind(this)
            }, {
                text: _t("Discard"),
                click: this._onClickClose.bind(this)
            }]
        });

        this.channelID = parseInt(options.channelId, 10);
        this.tagIds = options.channelTagIds || [];
        // Open with a tag name as default
        this.defaultTag = options.defaultTag;
        this._super(parent, options);
    },
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self._bindSelect2Dropdown();
            self._hideTagGroup();
            if (self.defaultTag) {
                self._setDefaultSelection();
            }
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * 'Tag' and 'Tag Group' management for select2
     *
     * @private
     */
    _bindSelect2Dropdown: function () {
        var self = this;
        this.$('#tag_id').select2(this._select2Wrapper(_t('Tag'),
            function () {
                return self._rpc({
                    route: '/slides/channel/tag/search_read',
                    params: {
                        fields: ['name'],
                        domain: [['id','not in',self.tagIds]],
                    }
                });
            })
        );
        this.$('#tag_group_id').select2(this._select2Wrapper(_t('Tag Group (required for new tags)'),
            function () {
                return self._rpc({
                    route: '/slides/channel/tag/group/search_read',
                    params: {
                        fields: ['name'],
                        domain: [],
                    }
                });
            })
        );
    },

    /**
     * Wrapper for select2 load data from server at once and store it.
     *
     * @private
     * @param {String} Placeholder for element.
     * @param {Function} Function to fetch data from remote location should return a Promise
     * resolved data should be array of object with id and name. eg. [{'id': id, 'name': 'text'}, ...]
     * @param {String} [nameKey='name'] (optional) the name key of the returned record
     *   ('name' if not provided)
     * @returns {Object} select2 wrapper object
    */
    _select2Wrapper: function (tag, fetchFNC, nameKey) {
        nameKey = nameKey || 'name';

        var values = {
            width: '100%',
            placeholder: tag,
            allowClear: true,
            formatNoMatches: false,
            selection_data: false,
            fetch_rpc_fnc: fetchFNC,
            formatSelection: function (data, container, fmt) {
                if (data.tag) {
                    data.text = data.tag;
                }
                return fmt(data.text);
            },
            createSearchChoice: function (term, data) {
                var addedTags = $(this.opts.element).select2('data');
                if (_.filter(_.union(addedTags, data), function (tag) {
                    return tag.text.toLowerCase().localeCompare(term.toLowerCase()) === 0;
                }).length === 0) {
                    if (this.opts.can_create) {
                        return {
                            id: _.uniqueId('tag_'),
                            create: true,
                            tag: term,
                            text: _.str.sprintf(_t("Create new %s '%s'"), tag, term),
                        };
                    } else {
                        return undefined;
                    }
                }
            },
            fill_data: function (query, data) {
                var that = this,
                    tags = {results: []};
                _.each(data, function (obj) {
                    if (that.matcher(query.term, obj[nameKey])) {
                        tags.results.push({id: obj.id, text: obj[nameKey]});
                    }
                });
                query.callback(tags);
            },
            query: function (query) {
                var that = this;
                // fetch data only once and store it
                if (!this.selection_data) {
                    this.fetch_rpc_fnc().then(function (data) {
                        that.can_create = data.can_create;
                        that.fill_data(query, data.read_results);
                        that.selection_data = data.read_results;
                    });
                } else {
                    this.fill_data(query, this.selection_data);
                }
            }
        };
        return values;
    },

    _setDefaultSelection: function () {
        this.$('#tag_id').select2('data', {id: _.uniqueId('tag_'), text: this.defaultTag, create: true}, true);
        this.$('#tag_id').select2('readonly', true);
    },

    /**
     * Get value for tag_id and [when appropriate] tag_group_id to send to server
     *
     * @private
     */
    _getSelect2DropdownValues: function () {
        var result = {};
        var tag = this.$('#tag_id').select2('data');
        if (tag) {
            if (tag.create) {
                // new tag
                var group = this.$('#tag_group_id').select2('data');
                if(group) {
                    result['tag_id'] = [0, {'name': tag.text}]
                    if (group.create) {
                        // new tag group
                        result['group_id'] = [0, {'name': group.text}];
                    } else {
                        result['group_id'] = [group.id];
                    }
                }
            } else {
                result['tag_id'] = [tag.id];
            }
        }
        return result;
    },

    /**
     * Select2 fields makes the "required" input hidden on the interface.
     * Therefore we need to make a method to visually provide this requirement
     * feedback to users. "tag group" field should only need this when a new tag
     * is created.
     *
     * @private
     */
    _formValidate: function ($form) {
        $form.addClass('was-validated');
        var result = $form[0].checkValidity();
        
        var $tagInput = this.$('#tag_id');
        if ($tagInput.length !== 0){
            var $tagSelect2Container = $tagInput
                .closest('.form-group')
                .find('.select2-container');
            $tagSelect2Container.removeClass('is-invalid is-valid');
            if ($tagInput.is(':invalid')) {
                $tagSelect2Container.addClass('is-invalid');
            } else if ($tagInput.is(':valid')) {
                $tagSelect2Container.addClass('is-valid');
                var $tagGroupInput = this.$('#tag_group_id');
                if ($tagGroupInput.length !== 0){
                    var $tagGroupSelect2Container = $tagGroupInput
                        .closest('.form-group')
                        .find('.select2-container');
                    if ($tagGroupInput.is(':invalid')) {
                        $tagGroupSelect2Container.addClass('is-invalid');
                    } else if ($tagGroupInput.is(':valid')) {
                        $tagGroupSelect2Container.addClass('is-valid');
                    }
                }
            }
        }
        return result;
    },

    _alertDisplay: function (message) {
        this._alertRemove();
        $('<div/>', {
            "class": 'alert alert-warning',
            role: 'alert'
        }).text(message).insertBefore(this.$('form'));
    },
    _alertRemove: function () {
        this.$('.alert-warning').remove();
    },
    
    /**
     * When the user IS NOT creating a new tag, this function hides the group tag field
     * and makes it not required. Since the select2 field makes an extra container, this
     * needs to be hidden along with the group tag input field and its label.
     *
     * @private
     */
    _hideTagGroup: function () {
        var $tag_group_id = this.$('#tag_group_id');
        var $tagGroupSelect2Container = $tag_group_id.closest('.form-group');
        $tagGroupSelect2Container.hide();
        $tag_group_id.removeAttr("required");
        $tag_group_id.select2("val", "");
    },

    /**
     * When the user IS creating a new tag, this function shows the field and
     * makes it required. Since the select2 field makes an extra container, this
     * needs to be shown along with the group input field and its label.
     *
     * @private
     */
    _showTagGroup: function () {
        var $tag_group_id = this.$('#tag_group_id');
        var $tagGroupSelect2Container = $tag_group_id.closest('.form-group');
        $tagGroupSelect2Container.show();
        $tag_group_id.attr("required", "required");
    },

    //--------------------------------------------------------------------------
    // Handler
    //--------------------------------------------------------------------------

    _onClickFormSubmit: function () {
        if (this.defaultTag && !this.channelID) {
            this._createNewTag();
        } else {
            this._addTagToChannel();
        }
    },

    _addTagToChannel: function () {
        var self = this;
        var $form = this.$('#slides_channel_tag_add_form');
        if (this._formValidate($form)) {
            var values = this._getSelect2DropdownValues();
            return this._rpc({
                route: '/slides/channel/tag/add',
                params: {'channel_id': this.channelID,
                         'tag_id': values.tag_id,
                         'group_id': values.group_id},
            }).then(function (data) {
                if (data.error) {
                    self._alertDisplay(data.error);
                } else {
                    window.location = data.url;
                }
            });
        }
    },

    _createNewTag: function () {
        var self = this;
        var $form = this.$('#slides_channel_tag_add_form');
        this.$('#tag_id').select2('readonly', false);
        var valid = this._formValidate($form);
        this.$('#tag_id').select2('readonly', true);
        if (valid) {
            var values = this._getSelect2DropdownValues();
            return this._rpc({
                route: '/slide_channel_tag/add',
                params: {
                    'tag_id': values.tag_id,
                    'group_id': values.group_id
                },
            }).then(function (data) {
                self.trigger_up('tag_refresh', { tag_id: data.tag_id });
                self.close();
            });
        }
    },

    _onClickClose: function () {
        if (this.defaultTag && !this.channelID) {
            this.trigger_up('tag_remove_new');
        }
        this.close();
    },

    _onChangeTag: function (ev) {
        var self = this;
        var tag = $(ev.currentTarget).select2('data');
        if (tag && tag.create) {
            self._showTagGroup();
        } else {
            self._hideTagGroup();
        }
    },
});

publicWidget.registry.websiteSlidesTag = publicWidget.Widget.extend({
    selector: '.o_wslides_js_channel_tag_add',
    xmlDependencies: ['/website_slides/static/src/xml/website_slides_channel_tag.xml'],
    events: {
        'click': '_onAddTagClick',
    },


    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _openDialog: function ($element) {
        var data = $element.data();
        return new TagCourseDialog(this, data).open();
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
        this._openDialog($(ev.currentTarget));
    },
});

return {
    TagCourseDialog: TagCourseDialog,
    websiteSlidesTag: publicWidget.registry.websiteSlidesTag
};

});

```

## File: static\src\js\slides_course_unsubscribe.js

```javascript
odoo.define('website_slides.unsubscribe_modal', function (require) {
'use strict';

var core = require('web.core');
var Dialog = require('web.Dialog');
var publicWidget = require('web.public.widget');
var utils = require('web.utils');

var QWeb = core.qweb;
var _t = core._t;

var SlideUnsubscribeDialog = Dialog.extend({
    template: 'slides.course.unsubscribe.modal',
    _texts: {
        titleSubscribe: _t("Subscribe"),
        titleUnsubscribe: _t("Notifications"),
        titleLeaveCourse: _t("Leave the course")
    },

    /**
     * @override
     * @param {Object} parent
     * @param {Object} options
     */
    init: function (parent, options) {
        options = _.defaults(options || {}, {
            title: options.isFollower === 'True' ? this._texts.titleSubscribe : this._texts.titleUnsubscribe,
            size: 'medium',
        });
        this._super(parent, options);

        this.set('state', '_subscription');
        this.on('change:state', this, this._onChangeType);

        this.channelID = parseInt(options.channelId, 10);
        this.isFollower = options.isFollower === 'True';
        this.enroll = options.enroll;
    },
    /**
     * @override
     */
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self.$('input#subscribed').prop('checked', self.isFollower);
            self._resetModal();
        });
    },

    getSubscriptionState: function () {
        return this.$('input#subscribed').prop('checked');
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------
    _getModalButtons: function () {
        var btnList = [];
        var state = this.get('state');
        if (state === '_subscription') {
            btnList.push({text: _t("Save"), classes: "btn-primary", click: this._onClickSubscriptionSubmit.bind(this)});
            btnList.push({text: _t("Discard"), close: true});
            btnList.push({text: _t("or Leave the course"), classes: "btn-danger ml-auto", click: this._onClickLeaveCourse.bind(this)});
        } else if (state === '_leave') {
            btnList.push({text: _t("Leave the course"), classes: "btn-danger", click: this._onClickLeaveCourseSubmit.bind(this)});
            btnList.push({text: _t("Discard"), click: this._onClickLeaveCourseCancel.bind(this)});
        }
        return btnList;
    },

    /**
     * @private
     */
    _resetModal: function () {
        var state = this.get('state');
        if (state === '_subscription') {
            this.set_title(this.isFollower ? this._texts.titleUnsubscribe : this._texts.titleSubscribe);
            this.$('input#subscribed').prop('checked', this.isFollower);
        }
        else if (state === '_leave') {
            this.set_title(this._texts.titleLeaveCourse);
        }
        this.set_buttons(this._getModalButtons());
    },

    //--------------------------------------------------------------------------
    // Handler
    //--------------------------------------------------------------------------
    _onClickLeaveCourse: function () {
        this.set('state', '_leave');
    },

    _onClickLeaveCourseCancel: function () {
        this.set('state', '_subscription');
    },

    _onClickLeaveCourseSubmit: function () {
        this._rpc({
            route: '/slides/channel/leave',
            params: {channel_id: this.channelID},
        }).then(function () {
            window.location.reload();
        });
    },

    _onClickSubscriptionSubmit: function () {
        if (this.isFollower === this.getSubscriptionState()) {
            this.destroy();
            return;
        }
        this._rpc({
            route: this.getSubscriptionState() ? '/slides/channel/subscribe' : '/slides/channel/unsubscribe',
            params: {channel_id: this.channelID},
        }).then(function () {
            window.location.reload();
        });
    },

    _onChangeType: function () {
        var currentType = this.get('state');
        var tmpl;
        if (currentType === '_subscription') {
            tmpl = 'slides.course.unsubscribe.modal.subscription';
        } else if (currentType === '_leave') {
            tmpl = 'slides.course.unsubscribe.modal.leave';
        }
        this.$('.o_w_slide_unsubscribe_modal_container').empty();
        this.$('.o_w_slide_unsubscribe_modal_container').append(QWeb.render(tmpl, {widget: this}));

        this._resetModal();
    },
});

publicWidget.registry.websiteSlidesUnsubscribe = publicWidget.Widget.extend({
    selector: '.o_wslides_js_channel_unsubscribe',
    xmlDependencies: ['/website_slides/static/src/xml/website_slides_unsubscribe.xml'],
    events: {
        'click': '_onUnsubscribeClick',
    },
    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _openDialog: function ($element) {
        var data = $element.data();
        return new SlideUnsubscribeDialog(this, data).open();
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

return {
    SlideUnsubscribeDialog: SlideUnsubscribeDialog,
    websiteSlidesUnsubscribe: publicWidget.registry.websiteSlidesUnsubscribe
};

});

```

## File: static\src\js\slides_embed.js

```javascript
/**
 * This is a minimal version of the PDFViewer widget.
 * It is NOT use in the website_slides module, but it is called when embedding
 * a slide/video/document. This code can depend on pdf.js, JQuery and Bootstrap
 * (see website_slides.slide_embed_assets bundle, in website_slides_embed.xml)
 */
$(function () {

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

            this.pdf_viewer = new PDFSlidesViewer(this.slide_url, this.canvas, true);
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
                if (this.pdf_viewer.pdf_page_total > 1) {
                    this.$('.o_slide_navigation_buttons').removeClass('hide');
                }
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
                    this.$('#next, #last').removeClass('disabled');
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
                this.$('#first').toggleClass('disabled', pageNum < 3 );
                this.$('#previous').toggleClass('disabled', pageNum < 2 );
                this.$('#next, #last').removeClass('disabled');
                this.$('#zoomout').toggleClass('disabled', this.pdf_viewer.pdf_zoom <= MIN_ZOOM);
                this.$('#zoomin').toggleClass('disabled', this.pdf_viewer.pdf_zoom >= MAX_ZOOM);
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
                this.$("#slide_suggest").removeClass('d-none');
                this.$('#next, #last').addClass('disabled');
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
        $(window).on('resize', _.debounce(function() {
            embeddedViewer.on_resize();
        }, 500));

        // switching slide with keyboard
        $(document).keydown(function (ev) {
            if (ev.keyCode === 37 || ev.keyCode === 38) {
                embeddedViewer.previous();
            }
            if (ev.keyCode === 39 || ev.keyCode === 40) {
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

        // embed widget page selector
        $('.oe_slide_js_embed_code_widget input').on('change', function () {
            var page = parseInt($(this).val());
            if (!(page > 0 && page <= embeddedViewer.pdf_viewer.pdf_page_total)) {
                page = 1;
            }
            var actualCode = embeddedViewer.$('.slide_embed_code').val();
            var newCode = actualCode.replace(/(page=).*?([^\d]+)/, '$1' + page + '$2');
            embeddedViewer.$('.slide_embed_code').val(newCode);
        });

        // To avoid create a dependancy to openerpframework.js, we use JQuery AJAX to post data instead of ajax.jsonRpc
        $('.oe_slide_js_share_email button').on('click', function () {
            var widget = $('.oe_slide_js_share_email');
            var input = widget.find('input');
            var slideID = widget.find('button').data('slide-id');
            if (input.val() && input[0].checkValidity()) {
                widget.removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');
                $.ajax({
                    type: "POST",
                    dataType: 'json',
                    url: '/slides/slide/send_share_email',
                    contentType: "application/json; charset=utf-8",
                    data: JSON.stringify({'jsonrpc': "2.0", 'method': "call", "params": {'slide_id': slideID, 'email': input.val()}}),
                    success: function () {
                        widget.html($('<div class="alert alert-info" role="alert"><strong>Thank you!</strong> Mail has been sent.</div>'));
                    },
                    error: function (data) {
                        console.error("ERROR ", data);
                    },
                });
            } else {
                widget.addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
                input.focus();
            }
        });
    }
});

```

## File: static\src\js\slides_share.js

```javascript
odoo.define('website_slides.slides_share', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
require('website_slides.slides');
var core = require('web.core');
var _t = core._t;

var ShareMail = publicWidget.Widget.extend({
    events: {
        'click button': '_sendMail',
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _sendMail: function () {
        var self = this;
        var input = this.$('input');
        var slideID = this.$('button').data('slide-id');
        if (input.val() && input[0].checkValidity()) {
            this.$el.removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');
            this._rpc({
                route: '/slides/slide/send_share_email',
                params: {
                    slide_id: slideID,
                    email: input.val(),
                },
            }).then(function () {
                self.$el.html($('<div class="alert alert-info" role="alert">' + _t('<strong>Thank you!</strong> Mail has been sent.') + '</div>'));
            });
        } else {
            this.$el.addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
            input.focus();
        }
    },
});

publicWidget.registry.websiteSlidesShare = publicWidget.Widget.extend({
    selector: '#wrapwrap',
    events: {
        'click a.o_wslides_js_social_share': '_onSlidesSocialShare',
        'click .o_clipboard_button': '_onShareLinkCopy',
    },

    /**
     * @override
     * @param {Object} parent
     */
    start: function (parent) {
        var defs = [this._super.apply(this, arguments)];
        defs.push(new ShareMail(this).attachTo($('.oe_slide_js_share_email')));

        return Promise.all(defs);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @override
     * @param {Object} ev
     */
    _onSlidesSocialShare: function (ev) {
        ev.preventDefault();
        var popUpURL = $(ev.currentTarget).attr('href');
        var popUp = window.open(popUpURL, 'Share Dialog', 'width=626,height=436');
        $(window).on('focus', function () {
            if (popUp.closed) {
                $(window).off('focus');
            }
        });
    },

    _onShareLinkCopy: function (ev) {
        ev.preventDefault();
        var $clipboardBtn = $(ev.currentTarget);
        $clipboardBtn.tooltip({title: "Copied !", trigger: "manual", placement: "bottom"});
        var self = this;
        var clipboard = new ClipboardJS('#' + $clipboardBtn[0].id, {
            target: function () {
                var share_link_el = self.$('#wslides_share_link_id_' + $clipboardBtn[0].id.split('id_')[1]);
                return share_link_el[0];
            },
            container: this.el
        });
        clipboard.on('success', function () {
            clipboard.destroy();
            $clipboardBtn.tooltip('show');
            _.delay(function () {
                $clipboardBtn.tooltip("hide");
            }, 800);
        });
        clipboard.on('error', function (e) {
            console.log(e);
            clipboard.destroy();
        })
    },
});
});

```

## File: static\src\js\slides_slide_archive.js

```javascript
odoo.define('website_slides.slide.archive', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var Dialog = require('web.Dialog');
var core = require('web.core');
var _t = core._t;

var SlideArchiveDialog = Dialog.extend({
    template: 'slides.slide.archive',

    /**
     * @override
     */
    init: function (parent, options) {
        options = _.defaults(options || {}, {
            title: _t('Archive Slide'),
            size: 'medium',
            buttons: [{
                text: _t('Archive'),
                classes: 'btn-primary',
                click: this._onClickArchive.bind(this)
            }, {
                text: _t('Cancel'),
                close: true
            }]
        });

        this.$slideTarget = options.slideTarget;
        this.slideId = this.$slideTarget.data('slideId');
        this._super(parent, options);
    },
    _checkForEmptySections: function (){
        $('.o_wslides_slide_list_category').each(function (){
            var $categoryHeader = $(this).find('.o_wslides_slide_list_category_header');
            var categorySlideCount = $(this).find('.o_wslides_slides_list_slide:not(.o_not_editable)').length;
            var $emptyFlagContainer = $categoryHeader.find('.o_wslides_slides_list_drag').first();
            var $emptyFlag = $emptyFlagContainer.find('small');
            if (categorySlideCount === 0 && $emptyFlag.length === 0){
                $emptyFlagContainer.append($('<small>', {
                    'class': "ml-1 text-muted font-weight-bold",
                    text: _t("(empty)")
                }));
            }
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Calls 'archive' on slide controller and then visually removes the slide dom element
     */
    _onClickArchive: function () {
        var self = this;

        this._rpc({
            route: '/slides/slide/archive',
            params: {
                slide_id: this.slideId
            },
        }).then(function (isArchived) {
            if (isArchived){
                self.$slideTarget.closest('.o_wslides_slides_list_slide').remove();
                self._checkForEmptySections();
            }
            self.close();
        });
    }
});

publicWidget.registry.websiteSlidesSlideArchive = publicWidget.Widget.extend({
    selector: '.o_wslides_js_slide_archive',
    xmlDependencies: ['/website_slides/static/src/xml/slide_management.xml'],
    events: {
        'click': '_onArchiveSlideClick',
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _openDialog: function ($slideTarget) {
        new SlideArchiveDialog(this, {slideTarget: $slideTarget}).open();
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

return {
    slideArchiveDialog: SlideArchiveDialog,
    websiteSlidesSlideArchive: publicWidget.registry.websiteSlidesSlideArchive
};

});

```

## File: static\src\js\slides_slide_like.js

```javascript
odoo.define('website_slides.slides.slide.like', function (require) {
'use strict';

var core = require('web.core');
var publicWidget = require('web.public.widget');
require('website_slides.slides');

var _t = core._t;

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
        this._rpc({
            route: '/slides/slide/like',
            params: {
                slide_id: slideId,
                upvote: voteType === 'like',
            },
        }).then(function (data) {
            if (! data.error) {
                self.$el.find('span.o_wslides_js_slide_like_up span').text(data.likes);
                self.$el.find('span.o_wslides_js_slide_like_down span').text(data.dislikes);
            } else {
                if (data.error === 'public_user') {
                    var message = _t('Please <a href="/web/login?redirect=%s">login</a> to vote this lesson');
                    var signupAllowed = data.error_signup_allowed || false;
                    if (signupAllowed) {
                        message = _t('Please <a href="/web/signup?redirect=%s">create an account</a> to vote this lesson');
                    }
                    self._popoverAlert(self.$el, _.str.sprintf(message, encodeURIComponent(document.URL)));
                } else if (data.error === 'vote_done') {
                    self._popoverAlert(self.$el, _t('You have already voted for this lesson'));
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

return {
    slideLikeWidget: SlideLikeWidget,
    websiteSlidesSlideLike: publicWidget.registry.websiteSlidesSlideLike
};

});

```

## File: static\src\js\slides_slide_toggle_is_preview.js

```javascript
odoo.define('website_slides.slide.preview', function (require) {
    'use strict';

    var publicWidget = require('web.public.widget');

    publicWidget.registry.websiteSlidesSlideToggleIsPreview = publicWidget.Widget.extend({
        selector: '.o_wslides_js_slide_toggle_is_preview',
        xmlDependencies: ['/website_slides/static/src/xml/slide_management.xml'],
        events: {
            'click': '_onPreviewSlideClick',
        },

        _toggleSlidePreview: function($slideTarget) {
            this._rpc({
                route: '/slides/slide/toggle_is_preview',
                params: {
                    slide_id: $slideTarget.data('slideId')
                },
            }).then(function (isPreview) {
                if (isPreview) {
                    $slideTarget.removeClass('badge-light badge-hide border');
                    $slideTarget.addClass('badge-success');
                } else {
                    $slideTarget.removeClass('badge-success');
                    $slideTarget.addClass('badge-light badge-hide border');
                }
            });
        },

        _onPreviewSlideClick: function (ev) {
            ev.preventDefault();
            this._toggleSlidePreview($(ev.currentTarget));
        },
    });

    return {
        websiteSlidesSlideToggleIsPreview: publicWidget.registry.websiteSlidesSlideToggleIsPreview
    };

});

```

## File: static\src\js\slides_upload.js

```javascript
odoo.define('website_slides.upload_modal', function (require) {
'use strict';

var core = require('web.core');
var Dialog = require('web.Dialog');
var publicWidget = require('web.public.widget');
var utils = require('web.utils');
var wUtils = require('website.utils');

var QWeb = core.qweb;
var _t = core._t;

var SlideUploadDialog = Dialog.extend({
    template: 'website.slide.upload.modal',
    events: _.extend({}, Dialog.prototype.events, {
        'click .o_wslides_js_upload_install_button': '_onClickInstallModule',
        'click .o_wslides_select_type': '_onClickSlideTypeIcon',
        'change input#upload': '_onChangeSlideUpload',
        'change input#url': '_onChangeSlideUrl',
    }),

    /**
     * @override
     * @param {Object} parent
     * @param {Object} options holding channelId and optionally upload and publish control parameters
     * @param {Object} options.modulesToInstall: list of additional modules to
     *      install {id: module ID, name: module short description}
     */
    init: function (parent, options) {
        options = _.defaults(options || {}, {
            title: _t("Upload a document"),
            size: 'medium',
        });
        this._super(parent, options);
        this._setup();

        this.channelID = parseInt(options.channelId, 10);
        this.defaultCategoryID = parseInt(options.categoryId,10);
        this.canUpload = options.canUpload === 'True';
        this.canPublish = options.canPublish === 'True';
        this.modulesToInstall = options.modulesToInstall ? JSON.parse(options.modulesToInstall.replace(/'/g, '"')) : null;
        this.modulesToInstallStatus = null;

        this.set('state', '_select');
        this.on('change:state', this, this._onChangeType);
        this.set('can_submit_form', false);
        this.on('change:can_submit_form', this, this._onChangeCanSubmitForm);

        this.file = {};
        this.isValidUrl = true;
    },
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self._resetModalButton();
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {string} message
     */
    _alertDisplay: function (message) {
        this._alertRemove();
        $('<div/>', {
            "class": 'alert alert-warning',
            id: 'upload-alert',
            role: 'alert'
        }).text(message).insertBefore(this.$('form'));
    },
    _alertRemove: function () {
        this.$('#upload-alert').remove();
    },
    /**
     * Section and tags management from select2
     *
     * @private
     */
    _bindSelect2Dropdown: function () {
        var self = this;
        this.$('#category_id').select2(this._select2Wrapper(_t('Section'), false,
            function () {
                return self._rpc({
                    route: '/slides/category/search_read',
                    params: {
                        fields: ['name'],
                        domain: [['channel_id', '=', self.channelID]],
                    }
                });
            })
        );
        this.$('#tag_ids').select2(this._select2Wrapper(_t('Tags'), true, function () {
            return self._rpc({
                route: '/slides/tag/search_read',
                params: {
                    fields: ['name'],
                    domain: [],
                }
            });
        }));
    },
    _fetchUrlPreview: function (url) {
        return this._rpc({
            route: '/slides/prepare_preview/',
            params: {
                'url': url,
                'channel_id': this.channelID
            },
        });
    },
    _formSetFieldValue: function (fieldId, value) {
        this.$('form').find('#'+fieldId).val(value);
    },
    _formGetFieldValue: function (fieldId) {
        return this.$('#'+fieldId).val();
    },
    _formValidate: function () {
        var form = this.$("form");
        form.addClass('was-validated');
        return form[0].checkValidity() && this.isValidUrl;
    },
    /**
     * Extract values to submit from form, force the slide_type according to
     * filled values.
     *
     * @private
     */
    _formValidateGetValues: function (forcePublished) {
        var canvas = this.$('#data_canvas')[0];
        var values = _.extend({
            'channel_id': this.channelID,
            'name': this._formGetFieldValue('name'),
            'url': this._formGetFieldValue('url'),
            'description': this._formGetFieldValue('description'),
            'duration': this._formGetFieldValue('duration'),
            'is_published': forcePublished,
        }, this._getSelect2DropdownValues()); // add tags and category

        // default slide_type (for webpage for instance)
        if (_.contains(this.slide_type_data), this.get('state')) {
            values['slide_type'] = this.get('state');
        }

        if (this.file.type === 'application/pdf') {
            _.extend(values, {
                'image_1920': canvas.toDataURL().split(',')[1],
                'slide_type': canvas.height > canvas.width ? 'document' : 'presentation',
                'mime_type': this.file.type,
                'datas': this.file.data
            });
        } else if (values['slide_type'] === 'webpage') {
            let fileData = this.file.type === 'image/svg+xml' ? this.__svgToPNG() : this.file.data;
            if (fileData instanceof Promise) {
                const fileDataProm = fileData;
                this.__svgLoadingExec = () => fileDataProm.then(result => values['image_1920'] = result);
                fileData = this._svgToPng();
            }
            _.extend(values, {
                'mime_type': 'text/html',
                'image_1920': fileData,
            });
        } else if (/^image\/.*/.test(this.file.type)) {
            let fileData = this.file.type === 'image/svg+xml' ? this.__svgToPNG() : this.file.data;
            let fileDataProm;
            if (fileData instanceof Promise) {
                fileDataProm = fileData;
                fileData = this._svgToPng();
            }
            if (values['slide_type'] === 'presentation') {
                if (fileDataProm) {
                    this.__svgLoadingExec = () => fileDataProm.then(result => values['datas'] = result);
                }
                _.extend(values, {
                    'slide_type': 'infographic',
                    'mime_type': this.file.type === 'image/svg+xml' ? 'image/png' : this.file.type,
                    'datas': fileData,
                });
            } else {
                if (fileDataProm) {
                    this.__svgLoadingExec = () => fileDataProm.then(result => values['image_1920'] = result);
                }
                _.extend(values, {
                    'image_1920': fileData,
                });
            }
        }
        return values;
    },
    /**
     * @private
     */
    _fileReset: function () {
        var control = this.$('#upload');
        control.replaceWith(control = control.clone(true));
        this.file.name = false;
    },

    _getModalButtons: function () {
        var btnList = [];
        var state = this.get('state');
        if (state === '_select') {
            btnList.push({text: _t("Cancel"), classes: 'o_w_slide_cancel', close: true});
        } else if (state === '_import') {
            if (! this.modulesToInstallStatus.installing) {
                btnList.push({text: this.modulesToInstallStatus.failed ? _t("Retry") : _t("Install"), classes: 'btn-primary', click: this._onClickInstallModuleConfirm.bind(this)});
            }
            btnList.push({text: _t("Discard"), classes: 'o_w_slide_go_back', click: this._onClickGoBack.bind(this)});
        } else if (state !== '_upload') { // no button when uploading
            if (this.canUpload) {
                if (this.canPublish) {
                    btnList.push({text: _t("Save & Publish"), classes: 'btn-primary o_w_slide_upload o_w_slide_upload_published', click: this._onClickFormSubmit.bind(this)});
                    btnList.push({text: _t("Save"), classes: 'o_w_slide_upload', click: this._onClickFormSubmit.bind(this)});
                } else {
                    btnList.push({text: _t("Save"), classes: 'btn-primary o_w_slide_upload', click: this._onClickFormSubmit.bind(this)});
                }
            }
            btnList.push({text: _t("Discard"), classes: 'o_w_slide_go_back', click: this._onClickGoBack.bind(this)});
        }
        return btnList;
    },
    /**
     * Get value for category_id and tag_ids (ORM cmd) to send to server
     *
     * @private
     */
    _getSelect2DropdownValues: function () {
        var result = {};
        var self = this;
        // tags
        var tagValues = [];
        _.each(this.$('#tag_ids').select2('data'), function (val) {
            if (val.create) {
                tagValues.push([0, 0, {'name': val.text}]);
            } else {
                tagValues.push([4, val.id]);
            }
        });
        if (tagValues) {
            result['tag_ids'] = tagValues;
        }
        // category
        if (!self.defaultCategoryID) {
            var categoryValue = this.$('#category_id').select2('data');
            if (categoryValue && categoryValue.create) {
                result['category_id'] = [0, {'name': categoryValue.text}];
            } else if (categoryValue) {
                result['category_id'] = [categoryValue.id];
                this.categoryID = categoryValue.id;
            }
        } else {
            result['category_id'] = [self.defaultCategoryID];
            this.categoryID = self.defaultCategoryID;
        }
        return result;
    },
    /**
     * Reset the footer buttons, according to current state of modal
     *
     * @private
     */
    _resetModalButton: function () {
        this.set_buttons(this._getModalButtons());
    },
    /**
     * Wrapper for select2 load data from server at once and store it.
     *
     * @private
     * @param {String} Placeholder for element.
     * @param {bool}  true for multiple selection box, false for single selection
     * @param {Function} Function to fetch data from remote location should return a Promise
     * resolved data should be array of object with id and name. eg. [{'id': id, 'name': 'text'}, ...]
     * @param {String} [nameKey='name'] (optional) the name key of the returned record
     *   ('name' if not provided)
     * @returns {Object} select2 wrapper object
    */
    _select2Wrapper: function (tag, multi, fetchFNC, nameKey) {
        nameKey = nameKey || 'name';

        var values = {
            width: '100%',
            placeholder: tag,
            allowClear: true,
            formatNoMatches: false,
            selection_data: false,
            fetch_rpc_fnc: fetchFNC,
            formatSelection: function (data, container, fmt) {
                if (data.tag) {
                    data.text = data.tag;
                }
                return fmt(data.text);
            },
            createSearchChoice: function (term, data) {
                var addedTags = $(this.opts.element).select2('data');
                if (_.filter(_.union(addedTags, data), function (tag) {
                    return tag.text.toLowerCase().localeCompare(term.toLowerCase()) === 0;
                }).length === 0) {
                    if (this.opts.can_create) {
                        return {
                            id: _.uniqueId('tag_'),
                            create: true,
                            tag: term,
                            text: _.str.sprintf(_t("Create new %s '%s'"), tag, term),
                        };
                    } else {
                        return undefined;
                    }
                }
            },
            fill_data: function (query, data) {
                var self = this,
                    tags = {results: []};
                _.each(data, function (obj) {
                    if (self.matcher(query.term, obj[nameKey])) {
                        tags.results.push({id: obj.id, text: obj[nameKey]});
                    }
                });
                query.callback(tags);
            },
            query: function (query) {
                var self = this;
                // fetch data only once and store it
                if (!this.selection_data) {
                    this.fetch_rpc_fnc().then(function (data) {
                        self.can_create = data.can_create;
                        self.fill_data(query, data.read_results);
                        self.selection_data = data.read_results;
                    });
                } else {
                    this.fill_data(query, this.selection_data);
                }
            }
        };

        if (multi) {
            values['multiple'] = true;
        }

        return values;
    },
    /**
     * Init the data relative to the support slide type to upload
     *
     * @private
     */
    _setup: function () {
        this.slide_type_data = {
            presentation: {
                icon: 'fa-file-pdf-o',
                label: _t('Presentation'),
                template: 'website.slide.upload.modal.presentation',
            },
            webpage: {
                icon: 'fa-file-text',
                label: _t('Web Page'),
                template: 'website.slide.upload.modal.webpage',
            },
            video: {
                icon: 'fa-video-camera',
                label: _t('Video'),
                template: 'website.slide.upload.modal.video',
            },
            quiz: {
                icon: 'fa-question-circle',
                label: _t('Quiz'),
                template: 'website.slide.upload.quiz'
            }
        };
    },
    /**
     * Show the preview
     * @private
     */
    _showPreviewColumn: function () {
        this.$('.o_slide_tutorial').addClass('d-none');
        this.$('.o_slide_preview').removeClass('d-none');
    },
    /**
     * Hide the preview
     * @private
     */
    _hidePreviewColumn: function () {
        this.$('.o_slide_tutorial').removeClass('d-none');
        this.$('.o_slide_preview').addClass('d-none');
    },
    /**
     * @private
     */
    // TODO: Remove this part, as now SVG support in image resize tools is included
    //Python PIL does not support SVG, so converting SVG to PNG
    _svgToPng: function () {
        return this.__svgToPNG(true);
    },
    /**
     * Bug fixed version of the original _svgToPng, it can return a Promise
     *
     * @returns {Promise<string>|string}
     */
    __svgToPNG: function (noAsync = false) {
        const imgEl = this.$el.find('img#slide-image')[0];
        const result = wUtils.svgToPNG(imgEl, noAsync);
        if (typeof(result) === 'string') {
            return result.split(',')[1];
        }
        return result.then(png => png.split(',')[1]);
    },

    //--------------------------------------------------------------------------
    // Handler
    //--------------------------------------------------------------------------

    _onChangeType: function () {
        var currentType = this.get('state');
        var tmpl;
        this.$modal.find('.modal-dialog').removeClass('modal-lg');
        if (currentType === '_select') {
            tmpl = 'website.slide.upload.modal.select';
        } else if (currentType === '_upload') {
            tmpl = 'website.slide.upload.modal.uploading';
        } else if (currentType === '_import') {
            tmpl = 'website.slide.upload.modal.import';
        } else {
            tmpl = this.slide_type_data[currentType]['template'];
            this.$modal.find('.modal-dialog').addClass('modal-lg');
        }
        this.$('.o_w_slide_upload_modal_container').empty();
        this.$('.o_w_slide_upload_modal_container').append(QWeb.render(tmpl, {widget: this}));

        this._resetModalButton();

        if (currentType === '_import') {
            this.set_title(_t("New Certification"));
        } else {
            this.set_title(_t("Upload a document"));
        }
    },
    _onChangeCanSubmitForm: function (ev) {
        if (this.get('can_submit_form')) {
            this.$('.o_w_slide_upload').button('reset');
        } else {
            this.$('.o_w_slide_upload').button('loading');
        }
    },
    _onChangeSlideUpload: function (ev) {
        var self = this;
        this._alertRemove();

        var $input = $(ev.currentTarget);
        var preventOnchange = $input.data('preventOnchange');
        var $preview = self.$('#slide-image');

        var file = ev.target.files[0];
        if (!file) {
            this.$('#slide-image').attr('src', '/website_slides/static/src/img/document.png');
            this._hidePreviewColumn();
            return;
        }
        var isImage = /^image\/.*/.test(file.type);
        var loaded = false;
        this.file.name = file.name;
        this.file.type = file.type;
        if (!(isImage || this.file.type === 'application/pdf')) {
            this._alertDisplay(_t("Invalid file type. Please select pdf or image file"));
            this._fileReset();
            this._hidePreviewColumn();
            return;
        }
        if (file.size / 1024 / 1024 > 25) {
            this._alertDisplay(_t("File is too big. File size cannot exceed 25MB"));
            this._fileReset();
            this._hidePreviewColumn();
            return;
        }

        utils.getDataURLFromFile(file).then(function (buffer) {
            if (isImage) {
                $preview.attr('src', buffer);
            }
            buffer = buffer.split(',')[1];
            self.file.data = buffer;
            self._showPreviewColumn();
        });

        if (file.type === 'application/pdf') {
            var ArrayReader = new FileReader();
            this.set('can_submit_form', false);
            // file read as ArrayBuffer for pdfjsLib get_Document API
            ArrayReader.readAsArrayBuffer(file);
            ArrayReader.onload = function (evt) {
                var buffer = evt.target.result;
                var passwordNeeded = function () {
                    self._alertDisplay(_t("You can not upload password protected file."));
                    self._fileReset();
                    self.set('can_submit_form', true);
                };
                /**
                 * The following line fixes pdfjsLib 'Util' global variable.
                 * This is (most likely) related to #32181 which lazy loads most assets.
                 *
                 * That caused an issue where the global 'Util' variable from pdfjsLib can be
                 * (depending of which libraries load first) overridden by the global 'Util'
                 * variable of bootstrap.
                 * (See 'lib/bootstrap/js/util.js' and 'web/static/lib/pdfjs/build/pdfjs.js')
                 *
                 * This commit ensures that the global 'Util' variable is set to the one of pdfjsLib
                 * right before it's used.
                 *
                 * Eventually, we should update or get rid of one of the two libraries since they're
                 * not compatible together, or make a wrapper that makes them compatible.
                 * In the mean time, this small fix allows not refactoring all of this and can not
                 * cause much harm.
                 */
                Util = window.pdfjsLib.Util;
                window.pdfjsLib.getDocument(new Uint8Array(buffer), null, passwordNeeded).then(function getPdf(pdf) {
                    self._formSetFieldValue('duration', (pdf._pdfInfo.numPages || 0) * 5);
                    pdf.getPage(1).then(function getFirstPage(page) {
                        var scale = 1;
                        var viewport = page.getViewport(scale);
                        var canvas = document.getElementById('data_canvas');
                        var context = canvas.getContext('2d');
                        canvas.height = viewport.height;
                        canvas.width = viewport.width;
                        // Render PDF page into canvas context
                        page.render({
                            canvasContext: context,
                            viewport: viewport
                        }).then(function () {
                            var imageData = self.$('#data_canvas')[0].toDataURL();
                            $preview.attr('src', imageData);
                            if (loaded) {
                                self.set('can_submit_form', true);
                            }
                            loaded = true;
                            self._showPreviewColumn();
                        });
                    });
                });
            };
        }

        if (!preventOnchange) {
            var input = file.name;
            var inputVal = input.substr(0, input.lastIndexOf('.')) || input;
            if (this._formGetFieldValue('name') === "") {
                this._formSetFieldValue('name', inputVal);
            }
        }
    },
    _onChangeSlideUrl: function (ev) {
        var self = this;
        var url = $(ev.target).val();
        this._alertRemove();
        this.isValidUrl = false;
        this.set('can_submit_form', false);
        this._fetchUrlPreview(url).then(function (data) {
            self.set('can_submit_form', true);
            if (data.error) {
                self._alertDisplay(data.error);
            } else {
                if (data.completion_time) {
                    // hours to minutes conversion
                    self._formSetFieldValue('duration', Math.round(data.completion_time * 60));
                }
                self.$('#slide-image').attr('src', data.url_src);
                self._formSetFieldValue('name', data.title);
                self._formSetFieldValue('description', data.description);

                self.isValidUrl = true;
                self._showPreviewColumn();
            }
        });
    },

    _onClickInstallModule: function (ev) {
        var $btn = $(ev.currentTarget);
        var moduleId = $btn.data('moduleId');
        if (this.modulesToInstallStatus) {
            this.set('state', '_import');
            if (this.modulesToInstallStatus.installing) {
                this.$('#o_wslides_install_module_text')
                    .text(_.str.sprintf(_t('Already installing "%s".'), this.modulesToInstallStatus.name));
            } else if (this.modulesToInstallStatus.failed) {
                this.$('#o_wslides_install_module_text')
                    .text(_.str.sprintf(_t('Failed to install "%s".'), this.modulesToInstallStatus.name));
            }
        } else {
            this.modulesToInstallStatus = _.extend({}, _.find(this.modulesToInstall, function (item) { return item.id === moduleId; }));
            this.set('state', '_import');
            this.$('#o_wslides_install_module_text')
                .text(_.str.sprintf(_t('Do you want to install the "%s" app?'), this.modulesToInstallStatus.name));
        }
    },

    _onClickInstallModuleConfirm: function () {
        var self = this;
        var $el = this.$('#o_wslides_install_module_text');
        $el.text(_.str.sprintf(_t('Installing "%s".'), this.modulesToInstallStatus.name));
        this.modulesToInstallStatus.installing = true;
        this._resetModalButton();
        this._rpc({
            model: 'ir.module.module',
            method: 'button_immediate_install',
            args: [[this.modulesToInstallStatus.id]],
        }).then(function () {
            window.location.href = window.location.origin + window.location.pathname + '?enable_slide_upload';
        }, function () {
            $el.text(_.str.sprintf(_t('Failed to install "%s".'), self.modulesToInstallStatus.name));
            self.modulesToInstallStatus.installing = false;
            self.modulesToInstallStatus.failed = true;
            self._resetModalButton();
        });
    },

    _onClickGoBack: function () {
        this.set('state', '_select');
        this.isValidUrl = true;
        if (this.modulesToInstallStatus && !this.modulesToInstallStatus.installing) {
            this.modulesToInstallStatus = null;
        }
    },

    _onClickFormSubmit: function (ev) {
        var self = this;
        var $btn = $(ev.currentTarget);
        if (this._formValidate()) {
            var values = this._formValidateGetValues($btn.hasClass('o_w_slide_upload_published')); // get info before changing state
            var oldType = this.get('state');
            this.set('state', '_upload');

            return new Promise(async resolve => {
                if (this.__svgLoadingExec) {
                    await this.__svgLoadingExec();
                }
                delete this.__svgLoadingExec;
                resolve();
            }).then(() => {
                return this._rpc({
                    route: '/slides/add_slide',
                    params: values,
                });
            }).then(function (data) {
                self._onFormSubmitDone(data, oldType);
            });
        }
    },

    _onFormSubmitDone: function (data, oldType) {
        if (data.error) {
            this.set('state', oldType);
            this._alertDisplay(data.error);
        } else {
            window.location = data.url;
        }
    },

    _onClickSlideTypeIcon: function (ev) {
        var $elem = this.$(ev.currentTarget);
        var slideType = $elem.data('slideType');
        this.set('state', slideType);

        this._bindSelect2Dropdown();  // rebind select2 at each modal body rendering
    },
});

publicWidget.registry.websiteSlidesUpload = publicWidget.Widget.extend({
    selector: '.o_wslides_js_slide_upload',
    xmlDependencies: ['/website_slides/static/src/xml/website_slides_upload.xml'],
    events: {
        'click': '_onUploadClick',
    },

    /**
     * @override
     */
    start: function () {
        // Automatically open the upload dialog if requested from query string
        if (this.$el.attr('data-open-modal')) {
            this.$el.removeAttr('data-open-modal');
            this._openDialog(this.$el);
        }
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _openDialog: function ($element) {
        var data = $element.data();
        return new SlideUploadDialog(this, data).open();
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

return {
    SlideUploadDialog: SlideUploadDialog,
    websiteSlidesUpload: publicWidget.registry.websiteSlidesUpload
};

});

```

## File: static\src\js\slide_category_one2many.js

```javascript
odoo.define('survey.slide_category_one2many', function (require){
"use strict";

var Context = require('web.Context');
var FieldOne2Many = require('web.relational_fields').FieldOne2Many;
var FieldRegistry = require('web.field_registry');
var ListRenderer = require('web.ListRenderer');
var config = require('web.config');

var SectionListRenderer = ListRenderer.extend({
    init: function (parent, state, params) {
        this.sectionFieldName = "is_category";
        this._super.apply(this, arguments);
    },
    _checkIfRecordIsSection: function (id){
        var record = this._findRecordById(id);
        return record && record.data[this.sectionFieldName];
    },
    _findRecordById: function (id){
        return _.find(this.state.data, function (record){
            return record.id === id;
        });
    },
    /**
     * Allows to hide specific field in case the record is a section
     * and, in this case, makes the 'title' field take the space of all the other
     * fields
     * @private
     * @override
     * @param {*} record
     * @param {*} node
     * @param {*} index
     * @param {*} options
     */
    _renderBodyCell: function (record, node, index, options){
        var $cell = this._super.apply(this, arguments);

        var isSection = record.data[this.sectionFieldName];

        if (isSection){
            if (node.attrs.widget === "handle"){
                return $cell;
            } else if (node.attrs.name === "name"){
                var nbrColumns = this._getNumberOfCols();
                if (this.handleField){
                    nbrColumns--;
                }
                if (this.addTrashIcon){
                    nbrColumns--;
                }
                $cell.attr('colspan', nbrColumns);
            } else {
                $cell.removeClass('o_invisible_modifier');
                return $cell.addClass('o_hidden');
            }
        }
        return $cell;
    },
    /**
     * Adds specific classes to rows that are sections
     * to apply custom css on them
     * @private
     * @override
     * @param {*} record
     * @param {*} index
     */
    _renderRow: function (record, index){
        var $row = this._super.apply(this, arguments);
        if (record.data[this.sectionFieldName]) {
            $row.addClass("o_is_section");
        }
        return $row;
    },
    /**
     * Adding this class after the view is rendered allows
     * us to limit the custom css scope to this particular case
     * and no other
     * @private
     * @override
     */
    _renderView: function (){
        var def = this._super.apply(this, arguments);
        var self = this;
        return def.then(function () {
            self.$('table.o_list_table').addClass('o_section_list_view');
        });
    },
    // Handlers
    /**
     * Overriden to allow different behaviours depending on
     * the row the user clicked on.
     * If the row is a section: edit inline
     * else use a normal modal
     * @private
     * @override
     * @param {*} ev
     */
    _onRowClicked: function (ev){
        var parent = this.getParent();
        var recordId = $(ev.currentTarget).data('id');
        var is_section = this._checkIfRecordIsSection(recordId);
        if (is_section && parent.mode === "edit"){
            this.editable = "bottom";
        } else {
            this.editable = null;
        }
        this._super.apply(this, arguments);
    },
    /**
     * Overriden to allow different behaviours depending on
     * the cell the user clicked on.
     * If the cell is part of a section: edit inline
     * else use a normal edit modal
     * @private
     * @override
     * @param {*} ev
     */
    _onCellClick: function (ev){
        var parent = this.getParent();
        var recordId = $(ev.currentTarget.parentElement).data('id');
        var is_section = this._checkIfRecordIsSection(recordId);
        if (is_section && parent.mode === "edit"){
            this.editable = "bottom";
        } else {
            this.editable = null;
            this.unselectRow();
        }
        this._super.apply(this, arguments);
    },
    /**
     * In this case, navigating in the list caused issues.
     * For example, editing a section then pressing enter would trigger
     * the inline edition of the next element in the list. Which is not desired
     * if the next element ends up being a question and not a section
     * @override
     * @param {*} ev
     */
    _onNavigationMove: function (ev){
        this.unselectRow();
    },
});

var SectionFieldOne2Many = FieldOne2Many.extend({
    init: function (parent, name, record, options){
        this._super.apply(this, arguments);
        this.sectionFieldName = "is_category";
        this.rendered = false;
    },
    /**
     * Overriden to use our custom renderer
     * @private
     * @override
     */
    _getRenderer: function (){
        if (this.view.arch.tag === 'tree'){
            return SectionListRenderer;
        }
        return this._super.apply(this, arguments);
    },
    /**
     * Overriden to allow different behaviours depending on
     * the object we want to add. Adding a section would be done inline
     * while adding a question would render a modal.
     * @private
     * @override
     * @param {*} ev
     */
    _onAddRecord: function (ev) {
        this.editable = null;
        if (!config.device.isMobile){
            var context_str = ev.data.context && ev.data.context[0];
            var context = new Context(context_str).eval();
            if (context['default_' + this.sectionFieldName]){
                this.editable = "bottom";
            }
        }
        this._super.apply(this, arguments);
    },
});

FieldRegistry.add('slide_category_one2many', SectionFieldOne2Many);
});
```

## File: static\src\js\website_slides.editor.js

```javascript
odoo.define('website_slides.editor', function (require) {
"use strict";

var core = require('web.core');
var Dialog = require('web.Dialog');
var QWeb = core.qweb;
var WebsiteNewMenu = require('website.newMenu');
var TagCourseDialog = require('website_slides.channel_tag.add').TagCourseDialog;
var wUtils = require('website.utils');

var _t = core._t;


var ChannelCreateDialog = Dialog.extend({
    template: 'website.slide.channel.create',
    xmlDependencies: Dialog.prototype.xmlDependencies.concat(
        ['/website_slides/static/src/xml/website_slides_channel.xml',
         '/website_slides/static/src/xml/website_slides_channel_tag.xml']
    ),
    events: _.extend({}, Dialog.prototype.events, {
        'change input#tag_ids' : '_onChangeTag',
    }),
    custom_events: _.extend({}, Dialog.prototype.custom_events, {
        'tag_refresh': '_onTagRefresh',
        'tag_remove_new': '_onTagRemoveNew',
    }),
    /**
     * @override
     * @param {Object} parent
     * @param {Object} options
     */
    init: function (parent, options) {
        options = _.defaults(options || {}, {
            title: _t("New Course"),
            size: 'medium',
            buttons: [{
                text: _t("Create"),
                classes: 'btn-primary',
                click: this._onClickFormSubmit.bind(this)
            }, {
                text: _t("Discard"),
                close: true
            },]
        });
        this._super(parent, options);
    },
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            var $input = self.$('#tag_ids');
            $input.select2({
                width: '100%',
                allowClear: true,
                formatNoMatches: false,
                multiple: true,
                selection_data: false,
                formatSelection: function (data, container, fmt) {
                    if (data.tag) {
                        data.text = data.tag;
                    }
                    return fmt(data.text);
                },
                createSearchChoice: function(term, data) {
                    var addedTags = $(this.opts.element).select2('data');
                    if (_.filter(_.union(addedTags, data), function (tag) {
                        return tag.text.toLowerCase().localeCompare(term.toLowerCase()) === 0;
                    }).length === 0) {
                        if (this.opts.can_create) {
                            return {
                                id: _.uniqueId('tag_'),
                                create: true,
                                tag: term,
                                text: _.str.sprintf(_t("Create new Tag '%s'"), term),
                            };
                        } else {
                            return undefined;
                        }
                    }
                },
                fill_data: function (query, data) {
                    var that = this,
                        tags = {results: []};
                    _.each(data, function (obj) {
                        if (that.matcher(query.term, obj.name)) {
                            tags.results.push({id: obj.id, text: obj.name});
                        }
                    });
                    query.callback(tags);
                },
                query: function (query) {
                    var that = this;
                    // fetch data only once and store it
                    if (!this.selection_data) {
                        self._rpc({
                            route: '/slides/channel/tag/search_read',
                            params: {
                                fields: ['name'],
                                domain: [],
                            }
                        }).then(function (data) {
                            that.can_create = data.can_create;
                                that.fill_data(query, data.read_results);
                                that.selection_data = data.read_results;
                        });
                    } else {
                        this.fill_data(query, this.selection_data);
                    }
                }
            });
        });
    },
    _onClickFormSubmit: function (ev) {
        var $form = this.$("#slide_channel_add_form");
        var $title = this.$("#title");
        if (!$title[0].value){
            $title.addClass('border-danger');
            this.$("#title-required").removeClass('d-none');
        } else {
            $form.submit();
        }
    },
    _onChangeTag: function (ev) {
        var self = this;
        var tags = $(ev.currentTarget).select2('data');
        tags.forEach(function (element) {
            if (element.create) {
                new TagCourseDialog(self, { defaultTag: element.text }).open();
            }
        });
    },
    /**
     * Replace the new tag ID by its real ID
     * @param ev
     * @private
     */
    _onTagRefresh: function (ev) {
        var $tag_ids = $('#tag_ids');
        var tags = $tag_ids.select2('data');
        tags.forEach(function (element) {
            if (element.create) {
                element.id = ev.data.tag_id;
                element.create = false;
            }
        });
        $tag_ids.select2('data', tags);
        // Set selection_data to false to force tag reload
        $tag_ids.data('select2').opts.selection_data = false;
    },
    /**
     * Remove the created tag if the user clicks on 'Discard' on the create tag Dialog
     * @private
     */
    _onTagRemoveNew: function () {
        var tags = $('#tag_ids').select2('data');
        tags = tags.filter(function (value) {
            return !value.create;
        });
        $('#tag_ids').select2('data', tags);
    },
});

WebsiteNewMenu.include({
    actions: _.extend({}, WebsiteNewMenu.prototype.actions || {}, {
        new_slide_channel: '_createNewSlideChannel',
    }),

    //--------------------------------------------------------------------------
    // Actions
    //--------------------------------------------------------------------------

    /**
     * Displays the popup to create a new slide channel,
     * and redirects the user to this channel.
     *
     * @private
     * @returns {Promise} Unresolved if there is a redirection
     */
     _createNewSlideChannel: function () {
        var self = this;
        var def = new Promise(function (resolve) {
            var dialog = new ChannelCreateDialog(self, {});
            dialog.open();
            dialog.on('closed', self, resolve);
        });
        return def;
     },
});
});

```

## File: static\src\js\tours\slides_tour.js

```javascript
odoo.define('website_slides.slides_tour', function (require) {
"use strict";

var core = require('web.core');
var _t = core._t;

var tour = require('web_tour.tour');

tour.register('slides_tour', {
    url: '/slides',
}, [{
    trigger: "body:has(#o_new_content_menu_choices.o_hidden) #new-content-menu > a",
    content: _t("Welcome on your course's home page. It's still empty for now. Click on \"<b>New</b>\" to write your first course."),
    consumeVisibleOnly: true,
    position: 'bottom',
}, {
    trigger: 'a[data-action="new_slide_channel"]',
    content: _t("Select <b>Course</b> to create it and manage it."),
    position: 'bottom',
    width: 210,
}, {
    trigger: 'input[name="name"]',
    content: _t("Give your course an engaging <b>Title</b>."),
    position: 'bottom',
    width: 280,
    run: 'text My New Course',
}, {
    trigger: 'textarea[name="description"]',
    content: _t("Give your course a helpful <b>Description</b>."),
    position: 'bottom',
    width: 300,
    run: 'text This course is for advanced users.',
}, {
    trigger: 'button.btn-primary',
    content: _t("Click on the <b>Create</b> button to create your first course."),
}, {
    trigger: '.o_wslides_js_slide_section_add',
    content: _t("Congratulations, your course has been created, but there isn't any content yet. First, let's add a <b>Section</b> to give your course a structure."),
    position: 'bottom',
}, {
    trigger: 'button.btn-primary',
    content: _t("A good course has a structure. Pick a name for your first section and click <b>Save</b> to create it."),
    position: 'bottom',
    width: 260,
}, {
    trigger: 'a.btn-primary.o_wslides_js_slide_upload',
    content: _t("Your first section is created, now it's time to add lessons to your course. Click on <b>Add Content</b> to upload a document, create a web page or link a video."),
    position: 'bottom',
}, {
    trigger: 'a[data-slide-type="presentation"]',
    content: _t("First, let's add a <b>Presentation</b>. It can be a .pdf or an image."),
    position: 'bottom',
}, {
    trigger: 'input#upload',
    content: _t("Choose a <b>File</b> on your computer."),
}, {
    trigger: 'input#name',
    content: _t("The <b>Title</b> of your lesson is autocompleted but you can change it if you want.</br>A <b>Preview</b> of your file is available on the right side of the screen."),
}, {
    trigger: 'input#duration',
    content: _t("The <b>Duration</b> of the lesson is based on the number of pages of your document. You can change this number if your attendees will need more time to assimilate the content."),
}, {
    trigger: 'button.o_w_slide_upload_published',
    content: _t("<b>Save & Publish</b> your lesson to make it available to your attendees."),
    position: 'bottom',
    width: 285,
}, {
    trigger: 'span.badge-info:contains("New")',
    content: _t("Congratulations! Your first lesson is available. Let's see the options available here. The tag \"<b>New</b>\" indicates that this lesson was created less than 7 days ago."),
    position: 'bottom',
}, {
    trigger: 'a[name="o_wslides_list_slide_add_quizz"]',
    extra_trigger: '.o_wslides_slides_list_slide:hover',
    content: _t("If you want to be sure that attendees have understood and memorized the content, you can add a Quiz on the lesson. Click on <b>Add Quiz</b>."),
}, {
    trigger: 'input[name="question-name"]',
    content: _t("Enter your <b>Question</b>. Be clear and concise."),
    position: 'left',
    width: 330,
}, {
    trigger: 'input.o_wslides_js_quiz_answer_value',
    content: _t("Enter at least two possible <b>Answers</b>."),
    position: 'left',
    width: 290,
}, {
    trigger: 'a.o_wslides_js_quiz_is_correct',
    content: _t("Mark the correct answer by checking the <b>correct</b> mark."),
    position: 'right',
    width: 230,
}, {
    trigger: 'i.o_wslides_js_quiz_comment_answer:last',
    content: _t("You can add <b>comments</b> on answers. This will be visible with the results if the user select this answer."),
    position: 'right',

}, {
    trigger: 'a.o_wslides_js_quiz_validate_question',
    content: _t("<b>Save</b> your question."),
    position: 'left',
    width: 170,
}, {
    trigger: 'li.breadcrumb-item:nth-child(2)',
    content: _t("Click on your <b>Course</b> to go back to the table of content."),
    position: 'top',
}, {
    trigger: 'label.js_publish_btn',
    content: _t("Once you're done, don't forget to <b>Publish</b> your course."),
    position: 'bottom',
}, {
    trigger: 'a.o_wslides_js_slides_list_slide_link',
    content: _t("Congratulations, you've created your first course.<br/>Click on the title of this content to see it in fullscreen mode."),
    position: 'bottom',
}]);

});

```

## File: static\src\xml\activity.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-extend="mail.activity_items">
        <t t-jquery=".o_thread_message .o_thread_message_core .o_thread_message_tools" t-operation="replace">
            <t t-if="activity.activity_type_id[1] == 'Access Request'">
                <t t-if="activity.user_id[0] === uid">
                    <a role="button" class="btn btn-link btn-success text-muted text-success o_activity_link o_activity_action_grant_access" t-att-data-partner-id="activity.request_partner_id[0]">
                        <i class="fa fa-check"/> Grant Access
                    </a>
                    <a role="button" class="btn btn-link btn-danger text-muted text-danger o_activity_link o_activity_action_refuse_access" t-att-data-partner-id="activity.request_partner_id[0]">
                        <i class="fa fa-times"/> Refuse Access
                    </a>
                </t>
            </t>
            <t t-else="">
                <div class="o_thread_message_tools btn-group">
                    <t t-call="mail.activity_thread_message_tools"/>
                </div>
            </t>
        </t>
    </t>

    <t t-inherit="mail.Activity" t-inherit-mode="extension">
        <xpath expr="//*[@name='tools']" position="replace">
            <t t-if="activity.requestingPartner and activity.thread.model === 'slide.channel'">
                <div class="o_Activity_tools">
                    <button class="o_Activity_toolButton o_Activity_grantAccessButton btn btn-link" t-on-click="_onGrantAccess">
                        <i class="fa fa-check"/> Grant Access
                    </button>
                    <button class="o_Activity_toolButton o_Activity_refuseAccessButton btn btn-link" t-on-click="_onRefuseAccess">
                        <i class="fa fa-times"/> Refuse Access
                    </button>
                </div>
            </t>
            <t t-else="">$0</t>
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
                class="btn btn-primary o_wslides_js_course_join_link text-uppercase font-weight-bold"
                title="Join the Course" aria-label="Join the Course"
                href="#">
                <t t-if="widget.channel.channelEnroll == 'public'">
                    <t t-if="widget.publicUser">
                        Sign in
                    </t>
                    <t t-else="" t-esc="widget.joinMessage" />
                </t>
            </a>
        </div>
    </t>

    <t t-name="slide.course.join.request">
        <div>
            <p>Do you want to request access to this course ?</p>
        </div>
    </t>
</templates>

```

## File: static\src\xml\slide_management.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="slides.slide.archive">
        <div>
            <p>Are you sure you want to archive this slide ?</p>
        </div>
    </t>

     <t t-name="slides.category.add">
        <div>
            <form action="/slides/category/add" method="POST" id="slide_category_add_form">
                <input type="hidden" name="csrf_token" t-att-value="csrf_token"/>
                <input type="hidden" name="channel_id" t-att-value="widget.channelId"/>
                <div class="form-group row">
                    <label for="section_name" class="col-sm-3 col-form-label">Section name</label>
                    <div class="col-sm-9">
                        <input type="text" class="form-control" name="name" id="section_name" required="required" placeholder="e.g. Introduction"/>
                    </div>
                </div>
            </form>
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

                <div t-foreach="widget.quiz.questions" t-as="question"
                     t-attf-class="o_wslides_js_lesson_quiz_question mt-3 mb-4 #{widget.slide.completed ? 'completed-disabled' : ''}"
                     t-att-data-question-id="question.id" t-att-data-title="question.question">
                    <div class="h4">
                        <small class="text-muted"><span t-esc="question_index+1"/>. </small> <span t-esc="question.question"/>
                    </div>
                    <div class="list-group">
                        <t t-foreach="question.answer_ids" t-as="answer">
                            <a t-att-data-answer-id="answer.id" href="#"
                                t-att-data-text="answer.text_value"
                                t-attf-class="o_wslides_quiz_answer list-group-item d-flex align-items-center list-group-item-action #{widget.slide.completed  &amp;&amp; answer.is_correct ? 'list-group-item-success' : '' }">

                                <label class="my-0 d-flex align-items-center justify-content-center mr-2">
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
                        </t>
                        <div class="o_wslides_quiz_answer_info list-group-item list-group-item-info d-none">
                            <i class="fa fa-info-circle"/>
                            <span class="o_wslides_quiz_answer_comment"/>
                        </div>
                    </div>
                </div>
                <div t-if="!widget.slide.completed" class="o_wslides_js_lesson_quiz_validation border-top pt-3"/>
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
                    <div t-if="widget.channel.channelEnroll == 'invite'">
                        <b>This course is private.
                            <span t-if="widget.publicUser">
                                Please
                                <a t-att-href="'/web/login?redirect=' + widget.redirectURL" class="font-weight-bold">
                                    sign in
                                </a>
                                to enroll.
                            </span>
                            <a t-else="" href="#" class="font-weight-bold o_wslides_js_channel_enroll"
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
                            <span title="Succeed and gain karma" aria-label="Succeed and gain karma" class="badge badge-pill badge-warning text-white font-weight-bold ml-3 px-2 py-1">
                                + <t t-esc="widget.quiz.quizKarmaGain"/> XP
                            </span>
                        </span>
                    </div>
                    <div t-else="" class="w-100">
                        <b class="h5 mb-0 o_wslides_quiz_join_course_message">
                            <span t-if="widget.channel.channelEnroll == 'public'">
                                <t t-if="widget.publicUser">
                                    Sign in and join the course to verify your answers!
                                </t>
                                <t t-else="">
                                    Join the course to take the quiz and verify your answers!
                                </t>
                            </span>
                        </b>
                        <span class="my-0 h4">
                            <span title="Succeed and gain karma" aria-label="Succeed and gain karma" class="badge badge-pill badge-warning text-white font-weight-bold ml-3 px-2 py-1">
                                + <t t-esc="widget.quiz.quizKarmaGain"/> XP
                            </span>
                        </span>
                        <div class="o_wslides_join_course_widget float-right"/>
                    </div>
                </div>
                <span t-if="widget.publicUser &amp;&amp; widget.channel.signupAllowed" class="d-block mt-2">
                    <span>Don't have an account ?</span>
                    <a class="font-weight-bold" t-att-href="'/web/signup?redirect=' + widget.redirectURL">Sign Up !</a>
                </span>
            </div>
            <div t-else="" class="d-md-flex align-items-center justify-content-between">
                <div t-att-class="'d-flex align-items-center' + (widget.slide.completed ? ' alert alert-success my-0 py-1 px-3' : '')">
                    <button t-if="! widget.slide.completed" role="button" title="Check answers" aria-label="Check answers"
                        class="btn btn-primary text-uppercase font-weight-bold o_wslides_js_lesson_quiz_submit">Check your answers</button>
                    <b t-else="" class="my-0 h5">Done !</b>
                    <span class="my-0 h5" style="line-height: 1">
                        <span role="button" title="Succeed and gain karma" aria-label="Succeed and gain karma" class="badge badge-pill badge-warning text-white font-weight-bold ml-3 px-2">
                            + <t t-if="!widget.slide.completed" t-esc="widget.quiz.quizKarmaGain"/><t t-else="" t-esc="widget.quiz.quizKarmaWon"/> XP
                        </span>
                    </span>
                </div>
                <div class="ml-auto mt-3 mt-md-0">
                    <button t-if="widget.quiz.quizAttemptsCount > 0 &amp;&amp; widget.slide.channelCanUpload" class="btn btn-light border o_wslides_js_lesson_quiz_reset">
                        Reset
                    </button>
                    <button t-if="widget.slide.completed &amp;&amp; widget.slide.hasNext" class="btn btn-primary o_wslides_quiz_continue">
                        Continue <i class="fa fa-chevron-right ml-1"/>
                    </button>
                </div>
            </div>
        </div>
    </t>

    <t t-name="slide.slide.quiz.finish">
        <div>
            <button type="button" class="o_wslides_quiz_modal_close_btn close position-absolute" data-dismiss="modal" aria-label="Close">&#215;</button>
            <div class="o_wslides_gradient d-none d-md-flex flex-shrink-0">
                <img class="o_wslides_quiz_modal_hero" src="/website_slides/static/src/img/quiz_modal_success.svg" alt=""/>
            </div>
            <div class="d-flex flex-column flex-grow-1 justify-content-between pl-md-5 p-3 overflow-auto">
                <div>
                    <h1 class="o_wslides_quiz_modal_title mt-3 display-4 font-weight-bold">Amazing!</h1>
                    <div class="pb-3">
                        <h4 class="o_wslides_quiz_modal_xp_gained pb-2 d-flex fade">You gained <span class="badge badge-pill badge-success text-white font-weight-bold ml-2 mr-1"><t t-esc="widget.quiz.quizKarmaWon"/> XP</span> !</h4>
                        <div class="mt-5 mb-4">
                            <div class="progress">
                                <div class="progress-bar" role="progressbar" t-att-aria-valuenow="widget.quiz.rankProgress.previous_rank.progress" aria-valuemin="0" aria-valuemax="100"
                                    t-attf-style="width: #{widget.quiz.rankProgress.previous_rank.progress}%"/>
                                <div class="progress-bar-tooltip" data-toggle="tooltip" data-placement="top" t-att-title="widget.quiz.rankProgress.new_rank.karma" />
                            </div>
                            <small class="float-left text-primary font-weight-bold o_wslides_quiz_modal_rank_lower_bound">
                                <t t-esc="widget.quiz.rankProgress.previous_rank.lower_bound"/>
                            </small>
                            <small t-if="widget.quiz.rankProgress.previous_rank.upper_bound" class="float-right font-weight-bold o_wslides_quiz_modal_rank_upper_bound">
                                <t t-esc="widget.quiz.rankProgress.previous_rank.upper_bound"/>
                            </small>
                        </div>
                    </div>
                    <div class="pb-3 o_wslides_quiz_modal_rank_motivational">
                        <t t-set="showLastRankDescription" t-value="widget.quiz.rankProgress.last_rank &amp;&amp; !widget.quiz.rankProgress.level_up" />
                        <t t-raw="showLastRankDescription ? widget.quiz.rankProgress.description : widget.quiz.rankProgress.previous_rank.motivational" />
                    </div>
                </div>
                <div class="o_wslides_quiz_modal_dismiss align-self-end d-none">
                    <t t-if="widget.quiz.rankProgress.level_up">
                        <a type="button" target="_blank" t-attf-href="/profile/user/#{widget.userId}" class="btn btn-light border">Check Profile</a>
                    </t>
                    <t t-if="widget.hasNext">
                        <button type="button" class="btn btn-light border o_wslides_quiz_modal_btn">Next <i class="fa fa-chevron-right"/></button>
                    </t>
                    <t t-else="">
                        <a type="button" href="/slides" class="btn btn-light border">End course</a>
                    </t>
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
                <div class="o_wslides_quiz_question row align-items-center mr-0 mb-2">
                    <div class="input-group ml-3">
                        <div class="input-group-prepend">
                            <span class="input-group-text o_wslides_quiz_question_sequence"><t t-esc="widget.sequence"/></span>
                        </div>
                        <input type="text" name="question-name" class="form-control col-11" placeholder="Enter your question"
                            t-att-value="widget.question.text"/>
                    </div>
                </div>
                <div class="text-muted mb-2">
                    <span>Select the correct answer below :</span>
                </div>
                <div class="list-group">
                    <t t-if="widget.question.answers" >
                        <t t-foreach="widget.question.answers" t-as="answer" >
                            <t t-call="slide.quiz.answer.line"/>
                        </t>
                    </t>
                    <t t-else="" >
                        <t t-foreach="[1, 2, 3]">
                            <t t-call="slide.quiz.answer.line" />
                        </t>
                    </t>
                </div>
            </form>
            <t t-if="widget.update" t-call="slide.quiz.update.buttons"/>
            <t t-else="" t-call="slide.quiz.create.buttons"/>
        </div>
    </t>

    <t t-name="slide.quiz.answer.line">
        <div class="o_wslides_js_quiz_answer row align-items-center mb-1" t-attf-data-answer-id="#{answer ? answer.id : ''}" >
            <div class="col ml-3 ml-md-5">
                <div class="row align-items-center">
                    <div class="input-group col-9 p-0">
                        <input type="text" class="o_wslides_js_quiz_answer_value form-control" placeholder="Enter your answer" t-attf-value="#{answer ? answer.text_value : ''}"/>
                        <div class="input-group-append">
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
                    <i t-attf-class="o_wslides_js_quiz_icon o_wslides_js_quiz_comment_answer fa fa-lg fa-info-circle p-md-2 py-2 pl-2 pr-1 #{answer &amp;&amp; answer.comment ? 'text-primary' : 'text-muted'}" title="Add comment on this answer" />
                    <i class="o_wslides_js_quiz_icon o_wslides_js_quiz_add_answer fa fa-lg fa-plus-circle p-md-2 py-2 px-1 text-muted" title="Add an answer below this one" />
                    <i class="o_wslides_js_quiz_icon o_wslides_js_quiz_remove_answer fa fa-lg fa-trash-o p-md-2 py-2 px-1 text-muted" title="Remove this answer" />
                </div>
                <div class="o_wslides_js_quiz_answer_comment row align-items-center d-none">
                    <input type="text" class="form-control col-8 offset-1 mt-1" placeholder="This is the correct answer, congratulations"
                        t-attf-value="#{answer ? answer.comment : ''}" />
                    <i class="o_wslides_js_quiz_icon o_wslides_js_quiz_remove_answer_comment fa fa-lg fa-trash-o p-2 text-muted" title="Remove the answer comment" />
                </div>
            </div>
        </div>
    </t>

    <t t-name="slide.quiz.create.buttons">
        <div>
            <a class="o_wslides_js_quiz_validate_question btn btn-primary text-white border" role="button">
                <span>Save</span>
            </a>
            <a class="o_wslides_js_quiz_cancel_question btn btn-light border" role="button">
                <span>Cancel</span>
            </a>
        </div>
    </t>

    <t t-name="slide.quiz.update.buttons">
        <a class="o_wslides_js_quiz_validate_question o_wslides_js_quiz_update btn btn-primary text-white border" role="button">
            <span>Update</span>
        </a>
        <a class="o_wslides_js_quiz_cancel_question btn btn-light border" role="button">
            <span>Cancel</span>
        </a>
    </t>

    <t t-name="slide.quiz.confirm.deletion">
        <div>Are you sure you want to delete this question : <strong t-esc="widget.questionTitle"/> ?</div>
    </t>

</templates>

```

## File: static\src\xml\website_slides_channel.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates xml:space="preserve">

    <t t-name="website.slide.channel.create">
        <div>
            <form action="/slides/channel/add" method="POST" id="slide_channel_add_form">
                <input type="hidden" name="csrf_token" t-att-value="csrf_token"/>
                <div class="form-group">
                    <label for="title" class="col-form-label">Title</label>
                    <input type="text" class="form-control" name="name" id="title" placeholder="Computer Science for kids" required="1"/>
                    <p id="title-required" class="text-danger mt-1 mb-0 d-none">Please fill in this field</p>
                </div>
                <div class="form-group">
                    <label for="tag_ids" class="col-form-label">Tags</label>
                    <input type="text" class="form-control" name="tag_ids" id="tag_ids" placeholder="Tags"/>
                </div>
                <label for="channel_type">Choose a layout</label>
                <div class="form-row">
                    <div class="form-group col-6">
                        <div class="form-check px-0">
                            <input class="form-check-input d-none" type="radio" name="channel_type" id="channel_type1" value="training" checked="checked"/>
                            <label for="channel_type1">
                                <img class="w-100" src="/website_slides/static/src/img/channel-training-layout.png" alt="Training Layout"/>
                            </label>
                        </div>
                    </div>
                    <div class="form-group col-6">
                        <div class="form-check px-0">
                            <input class="form-check-input d-none" type="radio" name="channel_type" id="channel_type2" value="documentation"/>
                            <label for="channel_type2">
                                <img class="w-100" src="/website_slides/static/src/img/channel-documentation-layout.png" alt="Documentation Layout"/>
                            </label>
                        </div>
                    </div>
                </div>
                <div class="form-group">
                    <label for="title">Description</label>
                    <textarea rows="2" class="form-control" name="description" id="description"
                              placeholder="Common tasks for a computer scientist is asking the right questions and answering
                              questions. In this course, you'll study those topics with activities about mathematics, science and logic." />
                </div>
                <div class="form-group">
                    <label id="communication-label">Review</label>
                    <div class="o_wslide_channel_communication_type">
                        <div class="form-check">
                            <input class="form-check-input" type="checkbox" id="allow_comment" name="allow_comment" checked="checked"/>
                            <span class="form-check-label" for="allow_comment">Allow students to review your course</span>
                        </div>
                    </div>
                </div>
            </form>
        </div>
    </t>

</templates>

```

## File: static\src\xml\website_slides_channel_tag.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates xml:space="preserve">

    <t t-name="website.slides.tag.add">
        <div class="form-group">
            <form action="/slides/channel/tag/add" method="POST" id="slides_channel_tag_add_form">
                <div class="form-group">
                    <label for="tag_id" class="col-form-label">Tag</label>
                    <input class="form-control" id="tag_id" required="required"/>
                </div>
                <div class="form-group">
                    <label id="tag_group_label" for="tag_group_id" class="col-form-label">Tag Group</label>
                    <input class="form-control" id="tag_group_id"/>
                </div>
            </form>
        </div>
    </t>

</templates>

```

## File: static\src\xml\website_slides_fullscreen.xml

```xml
<templates id="template" xml:space="preserve">

    <t t-name="website.slides.fullscreen.content">
        <t t-if="_.contains(['document', 'presentation'], widget.get('slide').type)">
            <div class="embed-responsive h-100">
                <iframe t-att-src="widget.get('slide').embedUrl" class="o_wslides_iframe_viewer" allowFullScreen="true" frameborder="0"/>
            </div>
        </t>
        <t t-if="widget.get('slide').type === 'infographic'">
            <div class="o_wslides_fs_player w-100 h-100 overflow-auto d-flex align-items-start justify-content-center">
                <img t-att-src="'/web/image/slide.slide/'+ widget.get('slide').id +'/image_1024'" class="img-fluid position-relative m-auto" alt="Slide image"/>
            </div>
        </t>
    </t>

    <t t-name="website.slides.fullscreen.video">
        <div class="player embed-responsive embed-responsive-16by9 embed-responsive-item h-100">
            <iframe t-att-id="'youtube-player' + widget.slide.id" t-att-src="widget.slide.embedUrl" allowFullScreen="true" frameborder="0" enablejsapi="1" autoplay="1" allow="autoplay"></iframe>
        </div>
    </t>

</templates>

```

## File: static\src\xml\website_slides_share.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="website.slide.share.modal">
        <div>
            <t t-call="website.slide.share.socialmedia"/>
        </div>
    </t>

    <t t-name="website.slide.share.socialmedia">
        <div class="row">
            <div class="col-12 col-lg-6 mb-4">
                <h5 class="mt-0 mb-2">Share on Social Networks</h5>
                <div class="btn-group" role="group">
                    <a t-attf-href="https://www.facebook.com/sharer/sharer.php?u=#{window.location.href}" class="btn border bg-white o_wslides_js_social_share" social-key="facebook" aria-label="Share on Facebook" title="Share on Facebook"><i class="fa fa-facebook-square fa-fw"/></a>
                    <a t-attf-href="https://twitter.com/intent/tweet?text=#{widget.slide.name}&amp;url=#{window.location.href}" class="btn border bg-white o_wslides_js_social_share"  social-key="twitter" aria-label="Share on Twitter" title="Share on Twitter"><i class="fa fa-twitter fa-fw"/></a>
                    <a t-attf-href="http://www.linkedin.com/sharing/share-offsite/?url=#{window.location.href}" social-key="linkedin" class="btn border bg-white o_wslides_js_social_share" aria-label="Share on LinkedIn" title="Share on LinkedIn"><i class="fa fa-linkedin fa-fw"/></a>
                </div>
            </div>
            <div class="col-12 col-lg-6">
                <h5 class="mt-0 mb-2">Share Link</h5>
                <div class="input-group">
                    <input type="text" class="form-control o_wslides_js_share_link" t-att-value="window.location.href" readonly="readonly" onClick="this.select();" />
                    <div class="input-group-append">
                        <button class="btn btn-sm btn-primary o_clipboard_button" style="border-top-right-radius: 4px;border-bottom-right-radius: 4px;" >
                            <span class="fa fa-clipboard"> Copy Link</span>
                        </button>
                    </div>
                </div>
            </div>
            <div t-attf-class="col-12 col-lg-6">
                <t t-call="website.slide.share.email"/>
            </div>
        </div>
    </t>

    <t t-name="website.slide.share.email">
        <h5 class="mt-4">Share by mail</h5>
        <div t-if="!widget.session.is_website_user" class="form-inline">
            <form class="form-group o_wslides_js_share_email" role="form">
                <div class="input-group">
                    <input type="email" class="form-control" placeholder="your-friend@domain.com"/>
                    <span class="input-group-append">
                        <button class="btn btn-primary" type="button"
                            data-loading-text="Sending..."
                            t-attf-data-slide-id="#{widget.slide.id}"
                            style="border-top-right-radius: 4px;border-bottom-right-radius: 4px;">
                            <i class="fa fa-envelope-o"/> Send Email
                        </button>
                    </span>
                </div>
            </form>
        </div>
        <div t-if="widget.session.is_website_user" class="alert alert-info d-inline-block">
            <p class="mb-0">Please <a t-attf-href="/web?redirect=#{window.location.href}" class="font-weight-bold"> login </a> to share this <t t-esc="widget.slide.type"/> by email.</p>
        </div>
    </t>

</templates>
```

## File: static\src\xml\website_slides_unsubscribe.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="slides.course.unsubscribe.modal">
        <div>
            <div class="o_w_slide_unsubscribe_modal_container">
                <t t-call="slides.course.unsubscribe.modal.subscription"/>
            </div>
        </div>
    </t>

    <t t-name="slides.course.unsubscribe.modal.subscription">
        <form class="clearfix">
            <div class="form-group row">
                <div class="controls mt8 ml-3">
                    <input id="subscribed" name="subscribed" type="checkbox"/>
                    <label for="subscribed" class="col-form-label font-weight-normal">Be notified when a new content is added.</label>
                </div>
            </div>
        </form>
    </t>

    <t t-name="slides.course.unsubscribe.modal.leave">
        <p>Do you really want to leave the course?</p>
        <p>All completed classes and earned karma will be lost.</p>
    </t>

</templates>

```

## File: static\src\xml\website_slides_upload.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="website.slide.upload.modal">
        <div>
            <div class="o_w_slide_upload_modal_container">
                <t t-call="website.slide.upload.modal.select"/>
            </div>
        </div>
    </t>

    <!--
        Slide Type Selection template
    -->
    <t t-name="website.slide.upload.modal.select">
        <div class="row p-1 mt-4 mb-2">
            <div t-foreach="widget.slide_type_data" t-as="slide_type" class="col-6 col-md-3">
                <t t-set="type_data" t-value="widget.slide_type_data[slide_type]"/>

                <a href="#" t-att-data-slide-type="slide_type"
                    class="content-type d-flex flex-column align-items-center mb-4 o_wslides_select_type btn rounded border text-600 p-3">
                    <i t-attf-class="fa #{type_data['icon']} mb-2 fa-3x"/>
                    <t t-esc="type_data['label']"/>
                </a>
            </div>
        </div>
        <t t-if="widget.modulesToInstall">
            <t t-foreach="widget.modulesToInstall" t-as="module_info">
                <a class="o_wslides_js_upload_install_button w-100 text-center mb-4 btn rounded border text-600 p-3"
                    href="#" t-att-title="module_info['name']"
                    t-att-data-module-id="module_info['id']">
                    <i class="fa fa-trophy"></i> <t t-esc="module_info['motivational']"/>
                </a>
            </t>
        </t>
    </t>

    <!--
        Uploading template
    -->
    <t t-name="website.slide.upload.modal.uploading">
        <div class="text-center" role="status">
            <div class="fa-3x">
                <i class="fa fa-spinner fa-pulse"></i>
            </div>
            <h4>Uploading document ...</h4>
        </div>
    </t>

    <!--
        Import module template
    -->
    <t t-name="website.slide.upload.modal.import">
        <p id="o_wslides_install_module_text"/>
    </t>

    <!--
        Slide Type common form part template
    -->
    <t t-name="website.slide.upload.modal.common">
        <div class="form-group">
            <label for="name" class="col-form-label">Title</label>
            <input id="name" name="name" placeholder="Title" class="form-control" required="required"/>
        </div>
        <div t-if="!widget.defaultCategoryID" class="form-group">
            <label for="category_id" class="col-form-label">Section</label>
            <input class="form-control" id="category_id"/>
        </div>
        <div class="form-group">
            <label for="tag_ids" class="col-form-label">Tags</label>
            <input id="tag_ids" name="tag_ids" type="hidden"/>
        </div>
        <div class="form-group">
            <label for="duration" class="col-form-label">Duration</label>
            <div class="input-group">
                <input type="number" id="duration" min="0" name="duration" placeholder="Estimated slide completion time" class="form-control"/>
                    <div class="input-group-prepend">
                    <span class="input-group-text">Minutes</span>
                </div>
            </div>
        </div>
    </t>

    <!--
        Slide Type templates
    -->
    <t t-name="website.slide.upload.modal.presentation">
        <div>
            <form class="clearfix">
                <div class="row">
                    <div id="o_wslides_js_slide_upload_left_column" class="col-md-6">
                        <div class="form-group">
                            <label for="upload" class="col-form-label">Choose a PDF or an Image</label>
                            <input id="upload" name="file" class="form-control h-100" accept="image/*,application/pdf" type="file" required="required"/>
                        </div>
                        <canvas id="data_canvas" class="d-none"></canvas>
                        <t t-call="website.slide.upload.modal.common"/>
                    </div>
                    <div id="o_wslides_js_slide_upload_preview_column" class="col-md-6">
                        <div class="img-thumbnail h-100">
                            <div class="o_slide_tutorial p-3">
                                <div class="h5">How to upload your PowerPoint Presentations or Word Documents?</div>
                                <div class="mx-3 my-4">Save your presentations or documents as PDF files and upload them.</div>
                                <div class="alert alert-warning" role="alert">
                                    <i class="fa fa-info-circle pr-2"/>
                                    Only JPG, PNG, PDF, files types are supported
                                </div>
                            </div>
                            <div class="o_slide_preview d-none">
                                <img src="/website_slides/static/src/img/document.png" id="slide-image" title="Content Preview" alt="Content Preview" class="img-fluid"/>
                            </div>
                        </div>
                    </div>
                </div>
            </form>
        </div>
    </t>

    <t t-name="website.slide.upload.modal.webpage">
        <div>
            <form class="clearfix">
                <div class="row">
                    <div id="o_wslides_js_slide_upload_left_column" class="col-md-6">
                        <canvas id="data_canvas" class="d-none"></canvas>
                        <t t-call="website.slide.upload.modal.common"/>
                    </div>
                    <div id="o_wslides_js_slide_upload_preview_column" class="col-md-6">
                        <div class="img-thumbnail h-100">
                            <div class="o_slide_tutorial p-3">
                                <div class="h5">How to create a Lesson as a Web Page?</div>
                                <div class="mx-3 my-4">First, create your lesson, then edit it with the website builder. You'll be able to drop building blocks on your page and edit them.</div>
                            </div>
                        </div>
                    </div>
                </div>
            </form>
        </div>
    </t>

    <t t-name="website.slide.upload.modal.video">
        <div>
            <form class="clearfix">
                <div class="row">
                    <div id="o_wslides_js_slide_upload_left_column" class="col-md-6">
                        <div class="form-group">
                            <label for="url" class="col-form-label">Youtube Link</label>
                            <input id="url" name="url" class="form-control" placeholder="Youtube Video URL" required="required"/>
                        </div>
                        <canvas id="data_canvas" class="d-none"></canvas>
                        <t t-call="website.slide.upload.modal.common"/>
                    </div>
                    <div id="o_wslides_js_slide_upload_preview_column" class="col-md-6">
                        <div class="img-thumbnail h-100">
                            <div class="o_slide_tutorial p-3">
                                <div class="h5">How to upload your videos ?</div>
                                <div class="mx-3 my-4">First, upload your videos on YouTube and mark them as <strong>unlisted</strong>. This way, they will be secured.</div>
                                <div class="mx-3 my-4">What does <strong>unlisted</strong> means? The YouTube "unlisted" means it is a video which can be viewed only by the users with the link to it. Your video will never come up in the search results nor on your channel.</div>
                                <div class="mx-3 my-4"><a href="https://support.google.com/youtube/answer/157177" target="_blank" >Change video privacy settings</a></div>
                            </div>
                            <div class="o_slide_preview d-none">
                                <img src="/website_slides/static/src/img/document.png" id="slide-image" title="Content Preview" alt="Content Preview" class="img-fluid"/>
                            </div>
                        </div>
                    </div>
                </div>
            </form>
        </div>
    </t>

    <t t-name="website.slide.upload.quiz">
        <div>
            <form class="clearfix">
                <div class="row">
                    <div id="o_wslides_js_slide_upload_left_column" class="col-md-6">
                        <canvas id="data_canvas" class="d-none"></canvas>
                        <t t-call="website.slide.upload.modal.common"/>
                    </div>
                    <div id="o_wslides_js_slide_upload_preview_column" class="col-md-6">
                        <div class="img-thumbnail h-100">
                            <div class="o_slide_tutorial p-3">
                                <div class="h5">Test your students with small Quizzes</div>
                                <div class="mx-3 my-4">With Quizzes you can keep your students focused and motivated by answering some questions and gaining some karma points</div>
                                <img src="/website_slides/static/src/img/onboarding-quiz.png" title="Quiz Demo Data" class="img-fluid"/>
                            </div>
                        </div>
                    </div>
                </div>
            </form>
        </div>
    </t>
</templates>

```

## File: views\assets.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <data>
        <template id="assets_backend" inherit_id="web.assets_backend" name="Slides Backend Assets">
            <xpath expr="." position="inside">
                <link rel="stylesheet" type="text/scss" href="/website_slides/static/src/scss/rating_rating_views.scss"/>
                <link rel="stylesheet" type="text/scss" href="/website_slides/static/src/scss/slide_views.scss"/>
                <script type="text/javascript" src="/website_slides/static/src/js/slide_category_one2many.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/rating_field_backend.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/activity.js"/>
            </xpath>
        </template>

        <template id="assets_frontend" inherit_id="website.assets_frontend" name="Slides Frontend Assets">
            <xpath expr="//link[last()]" position="after">
                <link rel="stylesheet" type="text/scss" href="/website_slides/static/src/scss/website_slides.scss" t-ignore="true"/>
                <link rel="stylesheet" type="text/scss" href="/website_slides/static/src/scss/website_slides_profile.scss"/>
                <link rel="stylesheet" type="text/scss" href="/website_slides/static/src/scss/slides_slide_fullscreen.scss" t-ignore="true"/>
            </xpath>
            <xpath expr="//script[last()]" position="after">
                <script type="text/javascript" src="/website_slides/static/src/js/slides.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/slides_share.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/slides_upload.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/slides_category_add.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/slides_slide_archive.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/slides_slide_toggle_is_preview.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/slides_slide_like.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/slides_course_slides_list.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/slides_course_fullscreen_player.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/slides_course_join.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/slides_course_enroll_email.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/slides_course_quiz.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/slides_course_quiz_question_form.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/slides_course_quiz_finish.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/slides_course_tag_add.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/slides_course_unsubscribe.js"/>
                <script type="text/javascript" src="/website_slides/static/src/js/tours/slides_tour.js"/>
            </xpath>
        </template>

        <template id="assets_tests" inherit_id="web.assets_tests" name="Slides Tests Assets">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/website_slides/static/src/tests/tours/slides_tour_tools.js"/>
                <script type="text/javascript" src="/website_slides/static/src/tests/tours/slides_course_member.js"/>
                <script type="text/javascript" src="/website_slides/static/src/tests/tours/slides_course_member_yt.js"/>
                <script type="text/javascript" src="/website_slides/static/src/tests/tours/slides_course_publisher.js"/>
                <script type="text/javascript" src="/website_slides/static/src/tests/tours/slides_course_reviews.js"/>
                <script type="text/javascript" src="/website_slides/static/src/tests/tours/slides_full_screen_web_editor.js"/>
            </xpath>
        </template>

        <template id="assets_editor_inherit_website_slides" inherit_id="website.assets_editor" name="website_slides Assets Editor">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/website_slides/static/src/js/website_slides.editor.js"/>
            </xpath>
        </template>

        <!-- Bundle (minimal) for embedded slide iframe -->
        <template id="website_slides.slide_embed_assets" name="Website slides embed assets">
            <t t-call="web._assets_helpers"/>
            <t t-call="web._assets_bootstrap"/>
            <link rel="stylesheet" type="text/scss" href="/website_slides/static/src/scss/website_slides.scss" t-ignore="true"/>

            <t t-call="web.pdf_js_lib"></t>
            <script type="text/javascript" src="/website_slides/static/lib/pdfslidesviewer/PDFSlidesViewer.js"></script>
            <script type="text/javascript" src="/website_slides/static/src/js/slides_embed.js"></script>
        </template>

        <template id="website_slides_tests" name="eLearning tests" inherit_id="web.qunit_suite_tests">
            <xpath expr="//script[last()]" position="after">
                <script type="text/javascript" src="/website_slides/static/src/components/activity/activity_tests.js"/>
            </xpath>
        </template>

    </data>
</odoo>

```

## File: views\rating_rating_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="rating_rating_view_kanban_slide_channel" model="ir.ui.view">
        <field name="name">rating.rating.view.kanban.slides</field>
        <field name="model">rating.rating</field>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <kanban create="false" class="o_slide_rating_kanban">
                <field name="rating"/>
                <field name="res_name"/>
                <field name="feedback"/>
                <field name="partner_id"/>
                <templates>
                    <t t-name="kanban-box">
                        <t t-set="val_stars" t-value="record.rating.raw_value"/>
                        <t t-set="val_integer" t-value="Math.floor(val_stars)"/>
                        <t t-set="val_decimal" t-value="val_stars - val_integer"/>
                        <t t-set="empty_star" t-value="5 - (val_integer + Math.ceil(val_decimal))"/>
                        <div class="oe_kanban_card oe_kanban_global_click">
                            <div class="d-flex flex-row">
                                <div class="o_slide_rating_kanban_left mr-3">
                                    <h1 class="o_slide_rating_value text-center text-primary" t-esc="val_stars"/>
                                    <t t-foreach="_.range(0, val_integer)" t-as="num">
                                        <i class="fa fa-star" aria-label="A star" role="img"></i>
                                    </t>
                                    <t t-if="val_decimal">
                                        <i class="fa fa-star-half-o" aria-label="Half a star" role="img"></i>
                                    </t>
                                    <t t-foreach="_.range(0, empty_star)" t-as="num" role="img">
                                        <i class="fa fa-star text-black-25" aria-label="A star"></i>
                                    </t>
                                </div>
                                <div>
                                    <div class="o_kanban_card_header">
                                        <div class="o_kanban_card_header_title">
                                            <span class="font-weight-bold"><field name="partner_id"/></span>
                                        </div>
                                    </div>
                                    <div class="o_kanban_card_content mt0 d-flex flex-column">
                                        <span>
                                            <i class="fa fa-folder mr-2" aria-label="Open folder"></i>
                                            <a type="object" name="action_open_rated_object" t-att-title="record.res_name.raw_value">
                                                <field name="res_name" />
                                            </a>
                                        </span>
                                        <span><i class="fa fa-clock-o mr-2" aria-label="Create date"/> <field name="create_date" /></span>
                                        <div class="d-flex mt-2">
                                            <span t-esc="record.feedback.raw_value"/>
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

    <record id="rating_rating_view_search_slide_channel" model="ir.ui.view">
        <field name="name">rating.rating.view.search.slides</field>
        <field name="model">rating.rating</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="rating.rating_rating_view_search"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='resource']" position="after">
                <filter string="Course" name="groupby_course" context="{'group_by': 'res_name'}"/>
            </xpath>
            <xpath expr="/search" position="inside">
                <filter string="Creation Date" name="rating_last_30_days" date="create_date" default_period="last_30_days"/>
                <separator/>
            </xpath>
        </field>
    </record>

    <record id="rating_rating_view_graph_slide_channel" model="ir.ui.view">
        <field name="name">rating.rating.view.graph.slides</field>
        <field name="model">rating.rating</field>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <graph string="Rating Average" type="bar" sample="1">
                <field name="res_name" type="row"/>
                <field name="rating" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="rating_rating_view_pivot_slide_channel" model="ir.ui.view">
        <field name="name">rating.rating.view.pivot.slides</field>
        <field name="model">rating.rating</field>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <pivot sample="1">
                <field name="res_name" type="row"/>
                <field name="rating_text" type="col"/>
                <field name="rating" type="measure"/>
            </pivot>
        </field>
    </record>

    <record id="rating_rating_action_slide_channel" model="ir.actions.act_window">
        <field name="name">Rating</field>
        <field name="res_model">rating.rating</field>
        <field name="view_mode">kanban,tree,graph,pivot,form</field>
        <field name="domain">[('consumed', '=', True), ('res_model', '=', 'slide.channel')]</field>
        <field name="context">{}</field>
        <field name="search_view_id" ref="rating_rating_view_search_slide_channel"/>
        <field name="view_id" ref="rating_rating_view_kanban_slide_channel"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                There are no ratings for these courses at the moment
            </p>
        </field>
    </record>

    <record id="rating_rating_action_slide_channel_report" model="ir.actions.act_window">
        <field name="name">Rating</field>
        <field name="res_model">rating.rating</field>
        <field name="domain">[('consumed', '=', True), ('res_model', '=', 'slide.channel')]</field>
        <field name="context">{}</field>
        <field name="search_view_id" ref="rating_rating_view_search_slide_channel"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                There are no ratings for these courses at the moment
            </p>
        </field>
    </record>
    <record id="rating_rating_action_slide_channel_report_view_graph" model="ir.actions.act_window.view">
        <field name="act_window_id" ref="rating_rating_action_slide_channel_report"/>
        <field name="sequence">1</field>
        <field name="view_mode">graph</field>
        <field name="view_id" ref="rating_rating_view_graph_slide_channel"/>
    </record>
    <record id="rating_rating_action_slide_channel_report_view_pivot" model="ir.actions.act_window.view">
        <field name="act_window_id" ref="rating_rating_action_slide_channel_report"/>
        <field name="sequence">2</field>
        <field name="view_mode">pivot</field>
        <field name="view_id" ref="rating_rating_view_pivot_slide_channel"/>
    </record>
    <record id="rating_rating_action_slide_channel_report_view_tree" model="ir.actions.act_window.view">
        <field name="act_window_id" ref="rating_rating_action_slide_channel_report"/>
        <field name="sequence">3</field>
        <field name="view_mode">tree</field>
        <field name="view_id" eval="False"/>
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
            <xpath expr="//div[@id='google_maps_setting']" position="after">
                <div class="col-12 col-lg-6 o_setting_box" id="slides_install_setting">
                    <div class="o_setting_right_pane">
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
                                <a href="https://console.developers.google.com/flows/enableapi?apiid=drive,youtube"><span class="fa fa-arrow-right"/>
                                    Create a Google Project and Get a Key
                                </a>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
            <xpath expr="//div[hasclass('settings')]" position="inside">
                <div class="app_settings_block" data-string="eLearning" string="eLearning" data-key="website_slides">
                    <h2>eLearning</h2>
                    <div class="row mt16 o_settings_container" id="elearning_selection_settings">
                        <div class="col-12 col-lg-6 o_setting_box" id="elearning_install_forum">
                            <div class="o_setting_left_pane">
                                <field name="module_website_slides_forum"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_website_slides_forum"/>
                                <div class="text-muted">
                                    Create a community and let the members help each others
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6"></div>
                        <div class="col-12 col-lg-6 o_setting_box" id="website_slides_install_mass_mailing_slides">
                            <div class="o_setting_left_pane">
                                <field name="module_mass_mailing_slides"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_mass_mailing_slides"/>
                                <div class="text-muted">
                                    Contact all the members of a course via mass mailing
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6"></div>
                        <div class="col-12 col-lg-6 o_setting_box" id="elearning_install_certif">
                            <div class="o_setting_left_pane">
                                <field name="module_website_slides_survey"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_website_slides_survey"/>
                                <div class="text-muted">
                                    Evaluate your students and certify them
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6"></div>
                        <div class="col-12 col-lg-6 o_setting_box" id="elearning_install_sell">
                            <div class="o_setting_left_pane">
                                <field name="module_website_sale_slides"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_website_sale_slides"/>
                                <div class="text-muted">
                                    Generate revenues thanks to your courses
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

    <record id="website_slides_action_settings" model="ir.actions.act_window">
        <field name="name">Settings</field>
        <field name="type">ir.actions.act_window</field>
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
        <field name="groups_id" eval="[(4, ref('website_slides.group_website_slides_officer'))]"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button class="oe_stat_button" type="object"
                    icon="fa-graduation-cap" name="action_view_courses"
                    attrs="{'invisible': ['|', ('slide_channel_count', '=', 0), ('is_company', '=', True)]}">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value"><field name="slide_channel_count"/></span>
                        <span class="o_stat_text">Courses</span>
                    </div>
                </button>
                <button class="oe_stat_button" type="object"
                    icon="fa-graduation-cap" name="action_view_courses"
                    attrs="{'invisible': ['|', ('slide_channel_company_count', '=', 0), ('is_company', '=', False)]}">
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

## File: views\slide_channel_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="slide_channel_partner_view_search" model="ir.ui.view">
            <field name="name">slide.channel.partner.search</field>
            <field name="model">slide.channel.partner</field>
            <field name="arch" type="xml">
                <search string="Channel Member">
                    <field name="partner_id"/>
                    <field name="partner_email"/>
                    <field name="channel_id"/>
                </search>
            </field>
        </record>

        <record id="slide_channel_partner_action" model="ir.actions.act_window">
            <field name="name">Attendees</field>
            <field name="res_model">slide.channel.partner</field>
            <field name="view_mode">tree</field>
            <field name="search_view_id" ref="website_slides.slide_channel_partner_view_search"/>
        </record>

        <record id="slide_channel_partner_view_tree" model="ir.ui.view">
            <field name="name">slide.channel.partner.tree</field>
            <field name="model">slide.channel.partner</field>
            <field name="arch" type="xml">
                <tree string="Attendees" editable="top">
                    <field name="create_date"/>
                    <field name="partner_id" string="Contact"/>
                    <field name="partner_email"/>
                    <field name="channel_id" string="Channel" invisible="context.get('default_channel_id',False)" />
                    <field name="completion" string="Progress" widget="progressbar" />
                    <button name="unlink" title="Remove" icon="fa-times" type="object"/>
                </tree>
            </field>
        </record>

        <record id="slide_channel_partner_action" model="ir.actions.act_window">
            <field name="name">Attendees</field>
            <field name="res_model">slide.channel.partner</field>
            <field name="view_mode">tree,form</field>
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
        <field name="name">slide.channel.tag.view.tree</field>
        <field name="model">slide.channel.tag</field>
        <field name="arch" type="xml">
            <tree string="Course Tags" editable="top">
                <field name="sequence" widget="handle"/>
                <field name="group_sequence" invisible="1"/>
                <field name="name"/>
                <field name="group_id"/>
            </tree>
        </field>
    </record>
    
    <record id="slide_channel_tag_action" model="ir.actions.act_window">
        <field name="name">Course Tags</field>
        <field name="res_model">slide.channel.tag</field>
        <field name="view_mode">tree,form</field>
    </record>

    <!-- SLIDE.CHANNEL.TAG.GROUP -->
    <record id="slide_channel_tag_group_view_search" model="ir.ui.view">
        <field name="name">slide.channel.tag.group.view.search</field>
        <field name="model">slide.channel.tag.group</field>
        <field name="arch" type="xml">
            <search string="Course Tag Groups">
                <field name="name"/>
            </search>
        </field>
    </record>

    <record id="slide_channel_tag_group_view_form" model="ir.ui.view">
        <field name="name">slide.channel.tag.group.view.form</field>
        <field name="model">slide.channel.tag.group</field>
        <field name="arch" type="xml">
            <form string="Course Tag Group">
                <sheet>
                    <group>
                        <div class="oe_title">
                            <label for="name" class="oe_edit_only" string="Group Name"/>
                            <h1><field name="name" default_focus="1" placeholder="Group Name"/></h1>
                            <label for="is_published" string="Menu Entry"/>
                            <field name="is_published"/><br/>
                            <field name="tag_ids">
                                <tree editable="bottom">
                                    <field name="sequence" widget="handle"/>
                                    <field name="group_sequence" invisible="1"/>
                                    <field name="name" string="Tag Name"/>
                                    <field name="color" string="Color" widget="color_picker"/>
                                    <control>
                                        <create string="Add a tag"/>
                                    </control>
                                </tree>
                            </field>
                        </div>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="slide_channel_tag_group_view_tree" model="ir.ui.view">
        <field name="name">slide.channel.tag.group.view.tree</field>
        <field name="model">slide.channel.tag.group</field>
        <field name="arch" type="xml">
            <tree string="Course Tag Groups">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="is_published" string="Menu Entry"/>
            </tree>
        </field>
    </record>

    <record id="slide_channel_tag_group_action" model="ir.actions.act_window">
        <field name="name">Course Groups</field>
        <field name="res_model">slide.channel.tag.group</field>
        <field name="view_mode">tree,form</field>
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
                        <button name="action_channel_invite" string="Invite" type="object" class="oe_highlight"  attrs="{'invisible': [('enroll', '!=', 'invite')]}"/>
                    </header>
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button name="action_view_slides"
                                type="object"
                                icon="fa-files-o"
                                class="oe_stat_button"
                                groups="website_slides.group_website_slides_officer">
                                <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_value"><field name="total_views" nolabel="1"/> Visits</span>
                                    <span class="o_stat_value"><field name="total_slides" nolabel="1"/> Contents</span>
                                </div>
                            </button>
                            <button name="action_redirect_to_done_members"
                                type="object"
                                icon="fa-trophy"
                                class="oe_stat_button"
                                groups="website_slides.group_website_slides_officer">
                                <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_value"><field name="members_done_count" nolabel="1"/></span>
                                    <span name="members_done_count_label" class="o_stat_text">Finished</span>
                                </div>
                            </button>
                            <button name="action_redirect_to_members"
                                type="object"
                                icon="fa-users"
                                class="oe_stat_button"
                                groups="website_slides.group_website_slides_officer">
                                <field name="members_count" string="Attendees" widget="statinfo"/>
                            </button>
                             <button name="action_view_ratings"
                                type="object"
                                icon="fa-star"
                                class="oe_stat_button"
                                groups="website_slides.group_website_slides_officer"
                                attrs="{'invisible': [('allow_comment', '=', False)]}">
                                <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_value"><field name="rating_avg_stars" nolabel="1"/>/5</span>
                                    <span name="rating_count_label" class="o_stat_text"><field name="rating_count" nolabel="1"/> Reviews</span>
                                </div>
                            </button>
                            <field name="is_published" widget="website_redirect_button"/>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <field name="image_1920" widget="image" class="oe_avatar" options="{'preview_image': 'image_128'}"/>
                        <div class="oe_title">
                            <label for="name" class="oe_edit_only" string="Course Title"/>
                            <h1><field name="name" default_focus="1" placeholder="Computer Science for kids"/></h1>
                        </div>
                        <div>
                            <field name="active" invisible="1"/>
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" placeholder="Tags"/>
                        </div>
                        <notebook colspan="4">
                            <page name="content" string="Content">
                                <field name="slide_ids" string="Content" colspan="4" nolabel="1" widget="slide_category_one2many" mode="tree,kanban" context="{'default_channel_id': active_id, 'form_view_ref' : 'website_slides.view_slide_slide_form_wo_channel_id'}">
                                     <tree decoration-bf="is_category" editable="bottom">
                                        <field name="sequence" widget="handle"/>
                                        <field name="name"/>
                                        <field name="slide_type" attrs="{'invisible': [('slide_type', '=', 'category')]}"/>
                                        <field name="completion_time" attrs="{'invisible': [('slide_type', '=', 'category')]}" string="Duration" widget="float_time"/>
                                        <field name="total_views" attrs="{'invisible': [('slide_type', '=', 'category')]}"/>
                                        <field name="is_preview" string="Preview"/>
                                        <field name="is_published" string="Published"/>
                                        <field name="is_category" invisible="1"/>
                                        <control>
                                            <create name="add_slide_section" string="Add Section" context="{'default_is_category': True}"/>
                                            <create name="add_slide_lesson" string="Add Content"/>
                                        </control>
                                    </tree>
                                </field>
                            </page>
                            <page name="description" string="Description">
                                <group>
                                    <field name="description" colspan="4" placeholder="Common tasks for a computer scientist is asking the right questions and answering questions. In this course, you'll study those topics with activities about mathematics, science and logic."/>
                                </group>
                            </page>
                            <page name="options" string="Options">
                                <group>
                                    <group name="course" string="Course">
                                        <field string="Type" name="channel_type" widget="radio"/>
                                        <field name="user_id" domain="[('share', '=', False)]"/>
                                        <field name="website_id" options="{'no_create': True}" groups="website.group_multi_website"/>
                                    </group>
                                    <group name="access_rights" string="Access Rights">
                                        <field name="enroll" widget="radio" options="{'horizontal': true}"/>
                                        <field name="upload_group_ids" widget="many2many_tags" groups="base.group_no_one"/>
                                        <field name="enroll_group_ids" widget="many2many_tags" groups="base.group_no_one"/>
                                    </group>
                                </group>
                                <group>
                                    <group name="communication" string="Communication">
                                        <field string="Allow Rating" name="allow_comment"/>
                                        <field name="publish_template_id" domain="[('model','=','slide.slide')]" groups="base.group_no_one"/>
                                        <field name="share_template_id" domain="[('model','=','slide.slide')]" groups="base.group_no_one"/>
                                    </group>
                                    <group name="display" string="Display">
                                        <field name="visibility" widget="radio"/>
                                        <field name="promote_strategy" widget="radio"
                                        attrs="{'invisible': [('channel_type', '=', 'training')]}"/>
                                        <field name="promoted_slide_id"
                                               attrs="{'invisible': ['|', ('channel_type', '=', 'training'), ('promote_strategy', '!=', 'specific')],
                                                       'required': [('channel_type', '!=', 'training'), ('promote_strategy', '=', 'specific')]}"
                                               domain="[('channel_id', '=', active_id), ('is_category', '=', False)]"/>
                                    </group>
                                </group>
                                <div attrs="{'invisible': [('enroll', '!=', 'invite')]}">
                                    <label for="enroll_msg"/>
                                    <field name="enroll_msg" colspan="4" nolabel="1"/>
                                </div>
                            </page>
                            <page string="Karma" name="karma_rules">
                                <group>
                                    <group string="Rewards">
                                        <field name="karma_gen_channel_rank" string="Review Course"/>
                                        <field name="karma_gen_channel_finish" string="Finish Course"/>
                                    </group>
                                    <group string="Access Rights" attrs="{'invisible': [('allow_comment', '!=', True)]}">
                                        <field name="karma_review" attrs="{'invisible': [('allow_comment', '!=', True)]}"/>
                                        <field name="karma_slide_comment" attrs="{'invisible': [('allow_comment', '!=', True)]}"/>
                                        <field name="karma_slide_vote" attrs="{'invisible': [('allow_comment', '!=', True)]}"/>
                                    </group>
                                </group>
                            </page>
                        </notebook>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids"/>
                        <field name="activity_ids"/>
                        <field name="message_ids"/>
                    </div>
                </form>
            </field>
        </record>


        <record id="slide_channel_view_tree" model="ir.ui.view">
            <field name="name">slide.channel.view.tree</field>
            <field name="model">slide.channel</field>
            <field name="arch" type="xml">
                <tree string="Courses" sample="1">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                    <field name="channel_type"/>
                    <field name="visibility"/>
                    <field name="enroll" widget="badge" decoration-success="enroll == 'public'" decoration-info="enroll == 'invite'" decoration-warning="enroll == 'payment'"/>
                    <field name="user_id" widget="many2one_avatar_user"/>
                    <field name="website_id" groups="website.group_multi_website"/>
                    <field name="active" invisible="1"/>
                </tree>
            </field>
        </record>

        <record id="slide_channel_view_tree_report" model="ir.ui.view">
            <field name="name">slide.channel.view.tree.report</field>
            <field name="model">slide.channel</field>
            <field name="priority">20</field>
            <field name="arch" type="xml">
                <tree string="Courses" create="false" default_order="total_views desc" sample="1">
                    <field name="name"/>
                    <field name="total_views"/>
                    <field name="total_time" widget="float_time" />
                    <field name="members_count"/>
                    <field name="total_votes"/>
                    <field name="rating_avg_stars"/>
                </tree>
            </field>
        </record>

        <record id="slide_channel_view_search" model="ir.ui.view">
            <field name="name">slide.channel.view.search</field>
            <field name="model">slide.channel</field>
            <field name="arch" type="xml">
                <search string="Courses">
                    <field name="name" string="Course"/>
                    <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                </search>
            </field>
        </record>

        <record id="slide_channel_view_graph" model="ir.ui.view">
            <field name="name">slide.channel.view.graph</field>
            <field name="model">slide.channel</field>
            <field name="arch" type="xml">
                <graph string="Courses" type="bar" sample="1">
                    <field name="name"/>
                    <field name="total_views" type="measure"/>
                </graph>
            </field>
        </record>

        <record id="slide_channel_view_kanban" model="ir.ui.view">
            <field name="name">slide.channel.view.kanban</field>
            <field name="model">slide.channel</field>
            <field name="arch" type="xml">
                <kanban string="eLearning Overview" class="o_emphasize_colors o_kanban_dashboard o_slide_kanban breadcrumb_item active" edit="false" sample="1">
                    <field name="color"/>
                    <field name="website_published"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_color_#{kanban_getcolor(record.color.raw_value)} oe_kanban_card oe_kanban_global_click">
                                <div class="o_dropdown_kanban dropdown">
                                    <a role="button" class="dropdown-toggle o-no-caret btn" data-toggle="dropdown" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                                        <span class="fa fa-ellipsis-v" aria-hidden="false"/>
                                    </a>
                                    <div class="dropdown-menu" role="menu">
                                        <ul class="oe_kanban_colorpicker" data-field="color"/>
                                        <t t-if="widget.deletable">
                                            <a class="dropdown-item" role="menuitem" type="delete">Delete</a>
                                        </t>
                                        <a class="dropdown-item" role="menuitem" type="edit">
                                            Edit
                                        </a>
                                        <a class="dropdown-item" name="action_view_slides" role="menuitem" type="object">
                                            Lessons
                                        </a>
                                        <a class="dropdown-item" name="action_channel_invite" role="menuitem" type="object">
                                            Invite
                                        </a>
                                    </div>
                                </div>
                                <div class="o_kanban_card_header">
                                    <div class="o_kanban_card_header_title mb16">
                                        <div class="o_primary">
                                            <a type="edit" class="mr-auto">
                                                <span><field name="name" class="o_primary"/></span>
                                            </a>
                                        </div>
                                        <div t-if="record.tag_ids">
                                            <field name="tag_ids" widget="many2many_tags"/>
                                        </div>
                                    </div>
                                </div>
                                <div class="container o_kanban_card_content mt0">
                                    <div class="row mb16">
                                        <div class="col-6 o_kanban_primary_left">
                                            <button class="btn btn-primary" name="open_website_url" type="object">View course</button>
                                        </div>
                                        <div class="col-6 o_kanban_primary_right">
                                            <div class="d-flex" t-if="record.rating_count.raw_value">
                                                <a name="action_view_ratings" type="object" class="mr-auto"><field name="rating_count"/> reviews</a>
                                                <span><field name="rating_avg_stars"/> / 5</span>
                                            </div>
                                            <div class="d-flex">
                                                <span class="mr-auto"><label for="total_views" class="mb0">Views</label></span>
                                                <field name="total_views"/>
                                            </div>
                                            <div class="d-flex" name="info_total_time">
                                                <span class="mr-auto"><label for="total_time" class="mb0">Duration</label></span>
                                                <field name="total_time" widget="float_time"/>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="row mt3">
                                        <div class="col-4 border-right">
                                            <a name="action_view_slides" type="object" class="d-flex flex-column align-items-center">
                                                <span class="font-weight-bold"><field name="total_slides"/></span>
                                                <span class="text-muted">Contents</span>
                                            </a>
                                        </div>
                                        <div class="col-4 border-right">
                                            <a name="action_redirect_to_members" type="object" class="d-flex flex-column align-items-center">
                                                <span class="font-weight-bold"><field name="members_count"/></span>
                                                <span class="text-muted">Attendees</span>
                                            </a>
                                        </div>
                                        <div class="col-4">
                                            <a name="action_redirect_to_done_members" type="object" class="d-flex flex-column align-items-center">
                                                <span class="font-weight-bold"><field name="members_done_count"/></span>
                                                <span name="done_members_count_label" class="text-muted">Finished</span>
                                            </a>
                                        </div>
                                    </div>
                                </div>
                             </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="slide_channel_action_overview" model="ir.actions.act_window">
            <field name="name">eLearning Overview</field>
            <field name="res_model">slide.channel</field>
            <field name="view_mode">kanban,tree,form</field>
            <field name="view_id" ref="slide_channel_view_kanban"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a course
                </p>
            </field>
        </record>

        <record id="slide_channel_action_report" model="ir.actions.act_window">
            <field name="name">Courses</field>
            <field name="res_model">slide.channel</field>
            <field name="view_mode">tree,graph,form</field>
            <field name="view_id" ref="slide_channel_view_tree_report"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a course
                </p>
            </field>
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
                <sheet>
                    <div class="oe_edit_only">
                        <label for="question" string="Question Name"/>
                    </div>
                    <h1>
                        <field name="question" default_focus="1" placeholder="Name"/>
                    </h1>
                    <field name="answer_ids">
                        <tree editable="bottom" create="true" delete="true">
                            <field name="text_value"/>
                            <field name="is_correct"/>
                            <field name="comment"/>
                        </tree>
                    </field>
                </sheet>
            </form>
        </field>
    </record>

    <record id="slide_question_view_tree" model="ir.ui.view">
        <field name="name">slide.question.view.tree</field>
        <field name="model">slide.question</field>
        <field name="arch" type="xml">
            <tree string="Quizzes">
                <field name="sequence" widget="handle"/>
                <field name="question"/>
                <field name="slide_id"/>
            </tree>
        </field>
    </record>

    <record id="slide_question_view_tree_report" model="ir.ui.view">
        <field name="name">slide.question.view.tree.report</field>
        <field name="model">slide.question</field>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <tree string="Quizzes" create="0">
                <field name="sequence" widget="handle"/>
                <field name="question"/>
                <field name="slide_id"/>
                <field name="attempts_count"/>
                <field name="attempts_avg"/>
                <field name="done_count"/>
            </tree>
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
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">slide.question</field>
        <field name="view_mode">tree,form</field>
        <field name="view_id" ref="slide_question_view_tree_report"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                There are no quizzes
            </p>
            <p>
                Add quizzes at the end of your lessons to evaluate what your students understood.
            </p>
        </field>
    </record>
</odoo>

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
            <field name="name">slide.tag.tree</field>
            <field name="model">slide.tag</field>
            <field name="arch" type="xml">
                <tree string="Tags" editable="bottom">
                    <field name="name"/>
                </tree>
            </field>
        </record>

        <record id="action_slide_tag" model="ir.actions.act_window">
            <field name="name">Content Tags</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">slide.tag</field>
            <field name="view_mode">tree,form</field>
        </record>

        <!-- SLIDE.SLIDE -->
        <record id="view_slide_slide_form" model="ir.ui.view">
            <field name="name">slide.slide.form</field>
            <field name="model">slide.slide</field>
            <field name="arch" type="xml">
                <form string="Lesson">
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <field name="is_published" widget="website_redirect_button"
                                   attrs="{'invisible': ['|',('is_category', '=', True), ('channel_id', '=', False)]}"/>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <field name="image_1920" widget="image" class="oe_avatar" options='{"preview_image": "image_256"}'
                            attrs="{'invisible': [('is_category', '=', True)]}"/>
                        <div class="oe_title">
                            <div>
                                <label for="name" string="Content Title" class="oe_edit_only"/>
                            </div>
                            <h1>
                                <field name="name" default_focus="1" placeholder="e.g. How to grow your business with Odoo?"/>
                                <field name="is_category" invisible="1"/>
                            </h1>
                            <field name="tag_ids" attrs="{'invisible': [('is_category', '=', True)]}" widget="many2many_tags" placeholder="Tags..."/>
                        </div>
                        <notebook attrs="{'invisible': [('is_category', '=', True)]}">
                            <page name="document" string="Document">
                                <group>
                                    <group name="lesson_details">
                                        <field name="active" invisible="1"/>
                                        <field name="channel_id"/>
                                        <field name="slide_type"/>
                                        <field name="url" attrs="{
                                            'required': [('slide_type', 'in', ('video'))],
                                            'invisible': [('slide_type', 'not in', ('video'))]}" />
                                        <field name="document_id" invisible="1"/>
                                        <field name="mime_type" force_save="1" readonly="1" groups="base.group_no_one"/>
                                        <field name="datas" string="Attachment"
                                            attrs="{'invisible': [('slide_type', 'not in', ('document', 'presentation'))]}"/>
                                    </group>
                                    <group name="related_details">
                                        <field name="user_id"/>
                                        <label for="completion_time"/>
                                        <div>
                                            <field name="completion_time" widget="float_time" class="oe_inline"/>
                                            <span> hours</span>
                                        </div>
                                        <field name="is_preview"/>
                                        <field name="slide_resource_downloadable" attrs="{'invisible': [('slide_type', 'not in', ['presentation', 'document'])]}"/>
                                        <field name="date_published" string="Published Date" attrs="{'invisible': [('date_published', '=', False)]}" groups="base.group_no_one"/>
                                    </group>
                                </group>
                            </page>
                            <page name="description" string="Description">
                                <field name="description" placeholder="e.g. In this video, we'll give you the keys on how Odoo can help you to grow your business. At the end, we'll propose you a quiz to test your knowledge."/>
                            </page>
                            <page string="Additional Resources" name="external_links" >
                                <group string="External Links">
                                    <field name="link_ids" widget="one2many" nolabel="1">
                                        <tree editable="top">
                                            <field name="name"/>
                                            <field name="link" widget="url" placeholder="e.g. https://www.odoo.com"/>
                                        </tree>
                                    </field>
                                </group>
                                <group string="Resources">
                                    <field name="slide_resource_ids" widget="one2many" nolabel="1">
                                        <tree editable="top">
                                            <field name="name"/>
                                            <field name="data" string="Size" required="1"/>
                                        </tree>
                                    </field>
                                </group>
                            </page>
                            <page name="quiz" string="Quiz">
                                <group name="quiz_details">
                                    <group name="quiz_rewards" string="Rewards">
                                        <group>
                                            <field string="First attempt" name="quiz_first_attempt_reward"/>
                                            <field string="Second attempt" name="quiz_second_attempt_reward"/>
                                            <field string="Third attempt" name="quiz_third_attempt_reward"/>
                                            <field string="Fourth and more attempt" name="quiz_fourth_attempt_reward"/>
                                        </group>
                                    </group>
                                    <group name="questions" string="Questions">
                                        <field name="question_ids" nolabel="1">
                                            <tree>
                                                <field name="sequence" widget="handle"/>
                                                <field name="question"/>
                                            </tree>
                                        </field>
                                    </group>
                                </group>
                            </page>
                            <page name="statistics" string="Statistics">
                                <group>
                                    <group name="view_statistics" string="Views">
                                        <field string="Member" name="slide_views"/>
                                        <field string="Public" name="public_views" readonly="1"/>
                                        <field string="Total" name="total_views"/>
                                        <hr attrs="{'invisible': [('channel_allow_comment', '!=', True), ('channel_type', '=', 'training')]}"/>
                                        <field name="channel_type" invisible="1" readonly="1"/>
                                        <field name="channel_allow_comment" invisible="1" readonly="1"/>
                                        <field name="likes" attrs="{'invisible': [('channel_type', '=', 'training')]}"/>
                                        <field name="dislikes" attrs="{'invisible': [('channel_type', '=', 'training')]}"/>
                                        <field name="comments_count" string="Comments" attrs="{'invisible': [('channel_allow_comment', '!=', True)]}"/>
                                    </group>
                                </group>
                            </page>
                        </notebook>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids"/>
                        <field name="message_ids"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="view_slide_slide_form_wo_channel_id" model="ir.ui.view">
            <field name="name">slide.slide.form.wo.channel_id</field>
            <field name="model">slide.slide</field>
            <field name="inherit_id" ref="view_slide_slide_form"/>
            <field name="priority" eval="50"/>
            <field name="mode">primary</field>
            <field name="type">form</field>
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
                    <field name="id"/>
                    <field name="channel_id"/>
                    <field name="slide_type"/>
                    <field name="user_id"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div class="oe_kanban_global_click o_kanban_record_has_image_fill">
                                <t t-set="placeholder" t-value="'/website_slides/static/src/img/channel-training-default.jpg'"/>
                                <div class="o_kanban_image_fill_left d-none d-md-block"
                                    t-attf-style="background-image:url('#{kanban_image('slide.slide', 'image_128', record.id.raw_value,  placeholder)}')">
                                    <img class="o_kanban_image_inner_pic"
                                        t-att-alt="record.channel_id.value"
                                        t-att-src="kanban_image('slide.channel', 'image_128', record.channel_id.raw_value)"/>
                                </div>
                                <div class="o_kanban_image rounded-circle d-md-none"
                                    t-attf-style="background-image:url('#{kanban_image('slide.slide', 'image_128', record.id.raw_value,  placeholder)}')">
                                    <img class="o_kanban_image_inner_pic"
                                        t-att-alt="record.channel_id.value"
                                        t-att-src="kanban_image('slide.channel', 'image_128', record.channel_id.raw_value)"/>
                                </div>
                                <div class="oe_kanban_details d-flex flex-column">
                                    <strong class="o_kanban_record_title oe_partner_heading"><field name="name"/></strong>
                                    <div class="text-mutex"><field name="channel_id"/></div>
                                    <div class="o_kanban_tags_section mb-2">
                                        <span class="oe_kanban_list_many2many">
                                            <field name="tag_ids" widget="many2many_tags"/>
                                        </span>
                                    </div>
                                    <div class="o_kanban_record_bottom mt-auto d-flex justify-content-between align-items-end">
                                        <span>
                                            <i class="fa fa-clock-o mr-2" aria-label="Duration" role="img" title="Duration"/><field name="completion_time" widget="float_time"/>
                                        </span>
                                        <span>
                                            <i class="fa fa-question mr-2" aria-label="Number of Questions" role="img" title="Number of Questions"/><field name="questions_count"/>
                                        </span>
                                        <span>
                                            <i class="fa fa-eye mr-2" aria-label="Views" role="img" title="Views"/><field name="total_views"/>
                                        </span>
                                        <span>
                                            <t t-if="record.slide_type.raw_value == 'infographic'">
                                                <i class="fa fa-file-image-o mr-2" aria-label="Infographic" role="img" title="Infographic"/>
                                            </t>
                                            <t t-elif="record.slide_type.raw_value == 'webpage'">
                                                <i class="fa fa-file-code-o mr-2" aria-label="Webpage" role="img" title="Webpage"/>
                                            </t>
                                            <t t-elif="record.slide_type.raw_value == 'video'">
                                                <i class="fa fa-file-video-o mr-2" aria-label="Video" role="img" title="Video"/>
                                            </t>
                                            <t t-elif="record.slide_type.raw_value == 'quiz'">
                                                <i class="fa fa-flag mr-2" aria-label="Quiz" role="img" title="Quiz"/>
                                            </t>
                                            <t t-else=""><i class="fa fa-file-pdf-o mr-2" aria-label="Document" role="img" title="Document"/></t>
                                            <field name="slide_type"/>
                                        </span>
                                        <field name="user_id" widget="many2one_avatar_user"/>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="view_slide_slide_tree" model="ir.ui.view">
            <field name="name">slide.slide.tree</field>
            <field name="model">slide.slide</field>
            <field name="arch" type="xml">
                <tree string="Contents" sample="1">
                    <field name="name"/>
                    <field name="website_id" groups="website.group_multi_website"/>
                    <field name="active" invisible="1"/>
                    <field name="slide_type"/>
                    <field name="channel_id"/>
                    <field name="category_id"/>
                    <field name="date_published"/>
                    <field name="slide_views"/>
                    <field name="public_views"/>
                    <field name="total_views"/>
                    <field name="completion_time"/>
                </tree>
            </field>
        </record>

        <record id="view_slide_slide_search" model="ir.ui.view">
            <field name="name">slide.slide.filter</field>
            <field name="model">slide.slide</field>
            <field name="arch" type="xml">
                <search string="Search Contents">
                    <field name="name"/>
                    <filter name="published" string="Published" domain="[('is_published', '=', True)]"/>
                    <filter name="not_published" string="Waiting for validation" domain="[('is_published', '=', False)]"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                    <group expand="0" string="Group By">
                        <filter string="Course" name="groupby_channel" domain="[]" context="{'group_by': 'channel_id'}"/>
                        <filter string="Category" name="groupby_category" domain="[]" context="{'group_by': 'category_id'}"/>
                        <filter string="Type" name="groupby_type" domain="[]" context="{'group_by': 'slide_type'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="slide_slide_view_graph" model="ir.ui.view">
            <field name="name">slide.slide.view.graph</field>
            <field name="model">slide.slide</field>
            <field name="arch" type="xml">
                <graph string="Graph of Contents" stacked="False" sample="1">
                    <field name="channel_id" type="row"/>
                    <field name="slide_type" type="col"/>
                    <field name="total_views" type="measure"/>
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
            <field name="view_mode">kanban,tree,form</field>
            <field name="context"></field>
            <field name="domain">[('is_category', '=', False)]</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Add a new lesson
                </p>
            </field>
        </record>

        <record id="slide_slide_action_report" model="ir.actions.act_window">
            <field name="name">Contents</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">slide.slide</field>
            <field name="view_mode">graph,tree,form,pivot</field>
            <field name="view_id" ref="slide_slide_view_graph"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No data yet!
                </p><p>
                    Create new content for your eLearning
                </p>
            </field>
        </record>
    </data>
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
        action="slide_channel_action_overview"/>

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
    <menuitem name="Reviews"
        id="website_slides_menu_courses_reviews"
        parent="website_slides_menu_courses"
        sequence="3"
        action="rating_rating_action_slide_channel"/>

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
    <menuitem name="Reviews"
        id="website_slides_menu_report_reviews"
        parent="website_slides_menu_report"
        sequence="6"
        action="rating_rating_action_slide_channel_report"/>
    <menuitem name="Quizzes"
        id="website_slides_menu_report_quizzes"
        parent="website_slides_menu_report"
        sequence="7"
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
                    <ol class="breadcrumb bg-transparent mb-0 pl-0 py-0 overflow-hidden">
                        <li class="breadcrumb-item">
                            <a href="/slides">Courses</a>
                        </li>
                        <t t-set="breadcrumb_class" t-value="'breadcrumb-item %s' % ('active' if not slide else '')" />
                        <li t-att-class="'breadcrumb-item %s' % ('active' if not search_category and not search_tag and not search_slide_type and not slide else '')">
                            <a t-att-href="'/slides/%s' % slug(channel)"><span t-esc="channel.name"/></a>
                        </li>
                        <li t-att-class="breadcrumb_class" t-att-aria-current="'page' and search_category" t-if="search_category">
                            <a t-att-href="'/slides/%s/category/%s' % (slug(channel), slug(search_category))"><span t-esc="search_category.name"/></a>
                        </li>
                        <li t-att-class="breadcrumb_class" t-att-aria-current="'page' and search_tag" t-if="search_tag">
                            <a t-att-href="'/slides/%s/tag/%s' % (slug(channel), slug(search_tag))"><span t-esc="search_tag.name"/></a>
                        </li>
                        <li t-att-class="breadcrumb_class" t-att-aria-current="'page' and search_uncategorized" t-if="search_uncategorized">
                            <a t-att-href="'/slides/%s?search_uncategorized=1' % (slug(channel))">Uncategorized</a>
                        </li>
                        <li t-att-class="breadcrumb_class" t-att-aria-current="'page' and search_slide_type" t-if="search_slide_type">
                            <a t-att-href="'/slides/%s?slide_type=%s' % (slug(channel), search_slide_type)"><span t-esc="slide_types[search_slide_type]"/></a>
                        </li>
                        <li t-if="slide" class="breadcrumb-item active text-truncate text-white">
                            <a t-att-href="'/slides/slide/%s' % slug(slide)"><span t-esc="slide.name"/></a>
                        </li>
                    </ol>
                </nav>

                <div class="col-md-4 d-none d-md-flex flex-row align-items-center justify-content-end">
                    <!-- search -->
                    <form t-attf-action="/slides/all" role="search" method="get">
                        <div class="input-group o_wslides_course_nav_search ml-1 position-relative">
                            <span class="input-group-prepend">
                                <button class="btn btn-link text-white rounded-0 pr-1" type="submit" aria-label="Search" title="Search">
                                    <i class="fa fa-search"></i>
                                </button>
                            </span>
                            <input type="text" class="form-control border-0 rounded-0 bg-transparent text-white" name="search" placeholder="Search courses"/>
                        </div>
                    </form>
                </div>

                <!-- Mobile Mode -->
                <div class="col d-md-none py-1">
                    <div class="btn-group w-100 position-relative" role="group" aria-label="Mobile sub-nav">
                        <div class="btn-group w-100">
                            <a class="btn bg-black-25 text-white dropdown-toggle" href="#" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">Nav</a>

                            <div class="dropdown-menu">
                                <a class="dropdown-item" href="/slides">Home</a>
                                <t t-set="dropdown_class" t-value="'dropdown-item %s' % ('active' if not slide else '')"/>
                                <a t-att-class="'dropdown-item %s' % ('active' if not search_category and not search_tag and not search_slide_type else '')" t-att-href="'/slides/%s' % slug(channel)">
                                    &#9492;<span class="ml-1" t-esc="channel.name"/>
                                </a>
                                <a t-att-class="dropdown_class" t-att-aria-current="'page' and search_category" t-if="search_category" t-att-href="'/slides/%s/category/%s' % (slug(channel), slug(search_category))">
                                    &#9492;<span class="ml-1" t-esc="search_category.name"/>
                                </a>
                                <a t-att-class="dropdown_class" t-att-aria-current="'page' and search_tag" t-if="search_tag" t-att-href="'/slides/%s/tag/%s' % (slug(channel), slug(search_tag))">
                                    &#9492;<span class="ml-1" t-esc="search_tag.name"/>
                                </a>
                                <a t-att-class="dropdown_class" t-att-aria-current="'page' and search_uncategorized" t-if="search_uncategorized" t-att-href="'/slides/%s?search_uncategorized=1' % (slug(channel))">
                                    &#9492;<span class="ml-1">Uncategorized</span>
                                </a>
                                <a t-att-class="dropdown_class" t-att-aria-current="'page' and search_slide_type" t-if="search_slide_type" t-att-href="'/slides/%s?slide_type=%s' % (slug(channel), search_slide_type)">
                                    &#9492;<span class="ml-1" t-esc="slide_types[search_slide_type]"/>
                                </a>
                                 <a t-if="slide" class="dropdown-item active" t-att-href="'/slides/slide/%s' % (slug(slide))">
                                    &#9492;<span class="ml-1" t-esc="slide.name"/>
                                </a>
                            </div>
                        </div>

                        <div class="btn-group ml-1 position-static">
                            <a class="btn bg-black-25 text-white dropdown-toggle" href="#" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false"><i class="fa fa-search"></i></a>
                            <div class="dropdown-menu dropdown-menu-right w-100" style="right: 10px;">
                                <form class="px-3" t-attf-action="/slides/#{slug(channel)}" role="search" method="get">
                                    <div class="input-group">
                                        <input type="text" class="form-control" name="search" placeholder="Search courses"/>
                                        <span class="input-group-append">
                                            <button class="btn btn-primary" type="submit" aria-label="Search" title="Search">
                                                <i class="fa fa-search"/>
                                            </button>
                                        </span>
                                    </div>
                                </form>
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
        <t t-call-assets="web.pdf_js_lib" t-css="false"/>
        <script type="text/javascript" src="/website_slides/static/lib/pdfslidesviewer/PDFSlidesViewer.js"></script>
    </t>
    <t t-set="body_classname" t-value="'o_wslides_body'"/>
    <t t-call="website.layout">
        <div id="wrap" t-attf-class="wrap mt-0">
            <div t-attf-class="o_wslides_course_header o_wslides_gradient position-relative text-white pb-md-0 pt-2 pt-md-5 #{'pb-3' if channel.channel_type == 'training' else 'o_wslides_course_doc_header pb-5'}">
                <t t-call="website_slides.course_nav"/>

                <div class="container mt-5 mt-md-3 mt-xl-4">
                    <div class="row align-items-end align-items-md-stretch">
                        <!-- ==== Header Left ==== -->
                        <div class="col-12 col-md-4 col-lg-3">
                            <div class="d-flex align-items-end justify-content-around h-100">
                                <div t-if="channel.image_1920" t-field="channel.image_1920" t-options='{"widget": "image", "class": "o_wslides_course_pict d-inline-block mb-2 mt-3 my-md-0"}' class="h-100"/>
                                <div t-else="" class="h-100">
                                    <img t-att-src="'/website_slides/static/src/img/channel-%s-default.jpg' % ('training' if channel.channel_type == 'training' else 'documentation')"
                                        class="o_wslides_course_pict d-inline-block mb-2 mt-3 my-md-0"/>
                                </div>
                            </div>
                        </div>

                        <!-- ==== Header Right ==== -->
                        <div class="col-12 col-md-8 col-lg-9 d-flex flex-column">
                            <div class="d-flex flex-column">
                                <h1 t-field="channel.name"/>
                                <p class="mb-0 mb-xl-3" t-field="channel.description"/>

                                <div t-if="channel.channel_type == 'documentation'" class="d-flex mb-md-5">
                                    <button role="button" class="btn text-white pl-0" title="Share Channel"
                                        aria-label="Share Channel"
                                        data-toggle="modal" t-att-data-target="'#slideChannelShareModal_%s' % channel.id">
                                        <i class="fa fa-share-square"></i> Share
                                    </button>
                                </div>
                            </div>
                            <div class="d-flex flex-column justify-content-center h5 flex-grow-1 mb-md-5" t-if="channel.allow_comment">
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
                                    <t t-set="_link_btn_classes" t-value="'btn-link text-white'"/>
                                </t>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Share modal : here to avoid having text-white from o_wslides_course_header -->
            <t t-call="website_slides.slide_share_modal" t-if="channel.channel_type == 'documentation'">
                <t t-set="record" t-value="channel"/>
            </t>

            <div class="o_wslides_course_main">
                <!-- ========== TRAINING COURSE ========== -->
                <div t-if="channel.channel_type == 'training'" class="container">
                    <div class="row">
                        <!-- Training Sidebar -->
                        <div class="col-12 col-md-4 col-lg-3 mt-3 mt-md-0">
                            <t t-call="website_slides.course_sidebar"/>
                        </div>

                        <!-- Training Content -->
                        <div class="col-12 col-md-8 col-lg-9">
                            <ul class="nav nav-tabs o_wslides_nav_tabs flex-nowrap" role="tablist" id="profile_extra_info_tablist">
                                <li class="nav-item">
                                    <a t-att-class="'nav-link %s' % ('active' if active_tab == 'home' else '')"
                                        id="home-tab" data-toggle="pill" href="#home" role="tab" aria-controls="home"
                                        t-att-aria-selected="'true' if active_tab == 'home' else 'false'">
                                        <i class="fa fa-home"/> Course
                                    </a>
                                </li>
                                <li t-if="channel.allow_comment" class="nav-item o_wslides_course_header_nav_review_training">
                                    <a t-att-class="'nav-link %s' % ('active' if active_tab == 'review' else '')"
                                        id="review-tab" data-toggle="pill" href="#review" role="tab" aria-controls="review"
                                        t-att-aria-selected="'true' if active_tab == 'review' else 'false'">
                                        Reviews<t t-if="rating_count"> (<t t-esc="rating_count"/>)</t>
                                    </a>
                                </li>
                            </ul>

                            <div class="tab-content py-4 o_wslides_tabs_content mb-4" id="courseMainTabContent">
                                <div t-att-class="'tab-pane fade %s' % ('show active' if active_tab == 'home' else '')" id="home" role="tabpanel" aria-labelledby="home-tab">
                                    <div class="mb-2 pt-1">
                                        <t t-if="channel.tag_ids">
                                            <t t-foreach="channel.tag_ids" t-as="channel_tag">
                                                <span t-attf-class="badge o_wslides_channel_tag #{'o_tag_color_'+str(channel_tag.color)}" t-esc="channel_tag.name"/>
                                            </t>
                                        </t>
                                        <a t-if="channel.can_upload"
                                            class="o_wslides_js_channel_tag_add border badge badge-light font-weight-normal py-1 m-1"
                                            role="button"
                                            aria-label="Add Tag"
                                            href="#"
                                            t-att-data-channel-id="channel.id"
                                            t-att-data-channel-tag-ids="channel.tag_ids.ids">
                                            <span>Add Tag</span>
                                         </a>
                                    </div>
                                    <t t-if="channel.channel_type == 'training'" t-call="website_slides.course_slides_list"/>
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

                <!-- ========== DOCUMENTATION COURSE ========== -->
                <t t-if="channel.channel_type == 'documentation'">
                    <div class="container">
                        <div class="row">
                            <div class="col-12 col-md-8 offset-md-4 col-lg-9 offset-lg-3">
                                <ul class="nav nav-tabs o_wslides_nav_tabs o_wslides_doc_nav_tabs flex-nowrap" role="tablist" id="profile_extra_info_tablist">
                                    <li class="nav-item">
                                        <a t-att-class="'nav-link %s' % ('active' if active_tab == 'home' else '')"
                                            id="home-tab" data-toggle="pill" href="#home" role="tab" aria-controls="home"
                                            t-att-aria-selected="'true' if active_tab == 'home' else 'false'">
                                            <i class="fa fa-home"/>
                                            Course
                                        </a>
                                    </li>
                                    <li t-if="channel.allow_comment" class="nav-item o_wslides_course_header_nav_review_documentation">
                                        <a t-att-class="'nav-link %s' % ('active' if active_tab == 'review' else '')"
                                            id="review-tab" data-toggle="pill" href="#review" role="tab" aria-controls="review"
                                            t-att-aria-selected="'true' if active_tab == 'review' else 'false'">
                                            Reviews<t t-if="rating_count"> (<t t-esc="rating_count"/>)</t>
                                        </a>
                                    </li>
                                </ul>
                            </div>
                        </div>
                    </div>

                    <div class="tab-content pb-5" id="courseMainTabContent">
                        <div t-att-class="'tab-pane fade %s' % ('show active' if active_tab == 'home' else '')" id="home" role="tabpanel" aria-labelledby="home-tab">
                            <t t-if="channel.channel_type == 'documentation'" t-call="website_slides.course_slides_cards"/>
                        </div>
                        <div t-if="channel.allow_comment" t-att-class="'tab-pane fade %s' % ('show active' if active_tab == 'review' else '')" id="review" role="tabpanel" aria-labelledby="review-tab">
                            <div class="container pt-4">
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
                </t>
            </div>
        </div>

        <t t-call="website_slides.slide_share_modal">
            <t t-set="record" t-value="channel"/>
        </t>
    </t>
</template>

<template id="course_sidebar" name="Course Sidebar (infos, CTA)">
    <!-- Training-only: Channel sidebar (aka general information + CTAs) -->
    <div class="o_wslides_course_sidebar bg-white px-3 py-2 py-md-3 mb-3 mb-md-5">

        <div class="o_wslides_sidebar_top d-flex justify-content-between">
            <div class="o_wslides_js_course_join flex-grow-1">
                <a t-if="not channel.is_member and channel.enroll == 'public'" role="button"
                    class="btn btn-primary btn-block o_wslides_js_course_join_link"
                    title="Start Course" aria-label="Start Course Channel"
                    t-att-href="'#'"
                    t-att-data-channel-id="channel.id"
                    t-att-data-channel-enroll="channel.enroll">
                    <span class="cta-title text_small_caps">
                        <t t-if="channel.channel_type == 'documentation'">Start Course</t>
                        <t t-else="">Join Course</t>
                    </span>
                </a>
                
                <div t-if="not channel.is_member and channel.enroll == 'invite'" class="text-center">
                    <div t-attf-class="alert my-0 bg-100 p-2 #{'o_wslides_js_channel_enroll' if not is_public_user else ''}"
                         t-att-data-channel-id="channel.id">
                        Private Course
                        <div t-if="is_public_user">
                            <small>
                                Please <a t-att-href="'/web/login?redirect=/slides/%s' % (slug(channel))">sign in</a> to contact responsible.
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
                <t t-if="channel.is_member">
                    <button class="d-flex align-items-center alert my-0 px-2 px-xl-3 bg-100 w-100 o_wslides_js_channel_unsubscribe"
                            t-att-data-channel-id="channel.id"
                            t-att-data-is-follower="channel.message_is_follower"
                            t-att-data-enroll="channel.enroll">
                        <t t-call="website_slides.slides_misc_user_image">
                            <t t-set="img_class" t-value="'rounded-circle mr-1'"/>
                            <t t-set="img_style" t-value="'width: 1.4em; height: 1.4em; object-fit: cover;'"/>
                        </t>
                        <h6 class="d-flex flex-grow-1 my-0">You're enrolled</h6>
                        <i class="fa fa-check"/>
                        <i class="fa fa-times"/>
                    </button>
                    <div class="d-flex align-items-center pt-3">
                        <t t-if="channel.completed">
                            <span class="badge badge-pill badge-success py-1 px-2 mx-auto" style="font-size: 1em"><i class="fa fa-check"/> Completed</span>
                        </t>
                        <t t-else="">
                            <div class="progress flex-grow-1 bg-black-50" style="height: 6px;">
                                <div class="progress-bar" role="progressbar" t-attf-style="width: #{channel.completion}%" t-att-aria-valuenow="channel.completion" aria-valuemin="0" aria-valuemax="100"></div>
                            </div>
                            <div class="ml-3 small">
                                <span class="o_wslides_progress_percentage" t-esc="channel.completion"/> %
                            </div>
                        </t>
                    </div>
                </t>
            </div>
            <button t-attf-class="btn d-md-none bg-white ml-1 border #{'alert' if channel.is_member else ''} #{'align-self-start' if channel.is_member or channel.enroll == 'invite' else 'align-self-end'}" type="button" data-toggle="collapse" data-target="#o_wslides_sidebar_collapse" aria-expanded="false" aria-controls="o_wslides_sidebar_collapse">More info</button>
        </div>

        <div id="o_wslides_sidebar_collapse" class="collapse d-md-block">
            <table class="table table-sm mt-3">
                <tr t-if="channel.user_id">
                    <th class="border-top-0">Responsible</th>
                    <td class="border-top-0"><span t-field="channel.user_id"/></td>
                </tr>
                <tr>
                    <th class="border-top-0">Last Update</th>
                    <td class="border-top-0"><t t-esc="channel.slide_last_update" t-options="{'widget': 'date'}"/></td>
                </tr>
                <tr t-if="channel.total_time">
                    <th class="border-top-0">Completion Time</th>
                    <td class="border-top-0"><t class="font-weight-bold" t-esc="channel.total_time" t-options="{'widget': 'duration', 'unit': 'hour', 'round': 'minute'}"/></td>
                </tr>
                <tr>
                    <th>Members</th>
                    <td><t t-esc="channel.members_count"/></td>
                </tr>
            </table>

            <div class="mt-3">
                <button role="button" class="btn btn-link btn-block" title="Share Channel"
                    aria-label="Share Channel"
                    data-toggle="modal" t-att-data-target="'#slideChannelShareModal_%s' % channel.id">
                    <i class="fa fa-share-square fa-fw"/> Share
                </button>
            </div>
        </div>
    </div>
</template>

<template id="course_slides_list" name="Training Course content: list">
    <div class="mb-5 o_wslides_slides_list" t-att-data-channel-id="channel.id">

        <ul class="o_wslides_js_slides_list_container list-unstyled">
            <t t-set="j" t-value="0"/>
            <t t-foreach="category_data" t-as="category">
                <t t-set="category_id" t-value="category['id'] if category['id'] else None"/>

                <li t-if="category['total_slides'] or channel.can_publish" t-att-class="'o_wslides_slide_list_category o_wslides_js_list_item mb-2' if category_id else 'mt-4'" t-att-data-slide-id="category_id" t-att-data-category-id="category_id">
                    <div t-att-data-category-id="category_id"
                         t-att-class="'o_wslides_slide_list_category_header position-relative d-flex justify-content-between align-items-center mt8 %s %s' % ('bg-white shadow-sm border-bottom-0' if category_id else 'border-0', 'o_wslides_js_category py-0' if channel.can_upload else 'py-2')">
                        <div t-att-class="'d-flex align-items-center pl-3 %s' % ('o_wslides_slides_list_drag' if channel.can_publish else '')">
                            <div t-if="channel.can_publish and category_id" class="o_wslides_slides_list_drag py-2 pr-3">
                                <i class="fa fa-bars"/>
                            </div>
                            <span t-if="category_id" t-field="category['category'].name"/>
                            <small t-if="not category['total_slides'] and category_id" class="ml-1 text-muted"><b>(empty)</b></small>
                        </div>
                        <div t-if="category_id" class="o_text_link d-flex border-left">
                            <a  t-if="channel.can_upload"
                                class="o_wslides_js_slide_upload px-3 py-2"
                                role="button"
                                aria-label="Upload Presentation"
                                href="#"
                                t-att-data-modules-to-install="modules_to_install"
                                t-att-data-channel-id="channel.id"
                                t-att-data-category-id="category_id"
                                t-att-data-can-upload="channel.can_upload"
                                t-att-data-can-publish="channel.can_publish">
                                <i class="fa fa-plus mr-1"/> <span class="d-none d-md-inline-block">Add Content</span>
                            </a>
                        </div>
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
        <div t-if="channel.can_upload" class="o_wslides_content_actions btn-group">
            <a  class="o_wslides_js_slide_upload mr-1 border btn btn-primary"
                role="button"
                aria-label="Upload Presentation"
                href="#"
                t-att-data-open-modal="enable_slide_upload"
                t-att-data-modules-to-install="modules_to_install"
                t-att-data-channel-id="channel.id"
                t-att-data-can-upload="channel.can_upload"
                t-att-data-can-publish="channel.can_publish"><i class="fa fa-plus mr-1"/><span>Add Content</span></a>
            <a class="o_wslides_js_slide_section_add border btn btn-light bg-white" t-attf-channel_id="#{channel.id}"
                href="#" role="button"
                groups="website_slides.group_website_slides_officer"><i class="fa fa-folder-o mr-1"/><span>Add Section</span></a>
        </div>
        <t t-if="not channel.slide_ids and channel.can_publish">
            <t t-call="website_slides.course_slides_list_sample"/>
        </t>
    </div>
    <div t-field="channel.description_html"/>
</template>

<template id="course_slides_list_sample" name="Course Sample Content">
    <ul class="list-unstyled mt-3" style="opacity: 50%;">
        <li class="o_wslides_slide_list_category mb-2">
            <div class="o_wslides_slide_list_category_header position-relative d-flex justify-content-between align-items-center mt8 bg-white shadow-sm border-bottom-0 py-2">
                <div class="d-flex align-items-center pl-3">
                    <span class="text-muted">Common tasks for a computer scientist</span>
                </div>
            </div>
            <ul class="list-unstyled pb-1 border-top">
                <li class="o_wslides_slides_list_slide bg-white-50 border-top-0 d-flex align-items-center pl-2 py-1 pr-2">
                    <i class="fa fa-file-text py-2 mx-2"/>
                    <div class="text-truncate mr-auto">
                        <span>Asking Question</span>
                    </div>
                    <div class="d-flex flex-row">
                        <span class="badge font-weight-bold px-2 py-1 m-1 badge-warning">
                            <i class="fa fa-fw fa-flag"/> 10 xp
                        </span>
                    </div>
                </li>
                <li class="o_wslides_slides_list_slide bg-white-50 border-top-0 d-flex align-items-center pl-2 py-1 pr-2">
                    <i class="fa fa-question-circle py-2 mx-2"/>
                    <div class="text-truncate mr-auto">
                        <span>Asking the right question</span>
                    </div>
                </li>
                <li class="o_wslides_slides_list_slide bg-white-50 border-top-0 d-flex align-items-center pl-2 py-1 pr-2">
                    <i class="fa fa-file-pdf-o py-2 mx-2"/>
                    <div class="text-truncate mr-auto">
                        <span>Answering Questions</span>
                    </div>
                    <div class="d-flex flex-row">
                        <span class="badge badge-info badge-arrow-right font-weight-normal px-2 py-1 m-1">New</span>
                    </div>
                </li>
            </ul>
        </li>
        <li class="o_wslides_slide_list_category mb-2">
            <div class="o_wslides_slide_list_category_header position-relative d-flex justify-content-between align-items-center mt8 bg-white shadow-sm border-bottom-0 py-2">
                <div class="d-flex align-items-center pl-3">
                    <span class="text-muted">Parts of computer science</span>
                </div>
            </div>
            <ul class="list-unstyled pb-1 border-top">
                <li class="o_wslides_slides_list_slide bg-white-50 border-top-0 d-flex align-items-center pl-2 py-1 pr-2">
                    <i class="fa fa-file-pdf-o py-2 mx-2"/>
                    <div class="text-truncate mr-auto">
                        <span>Mathematics</span>
                    </div>
                </li>
                <li class="o_wslides_slides_list_slide bg-white-50 border-top-0 d-flex align-items-center pl-2 py-1 pr-2">
                    <i class="fa fa-file-pdf-o py-2 mx-2"/>
                    <div class="text-truncate mr-auto">
                        <span>Science</span>
                    </div>
                </li>
                <li class="o_wslides_slides_list_slide bg-white-50 border-top-0 d-flex align-items-center pl-2 py-1 pr-2">
                    <i class="fa fa-play py-2 mx-2"/>
                    <div class="text-truncate mr-auto">
                        <span>Logic</span>
                    </div>
                    <div class="d-flex flex-row">
                        <span class="badge badge-success font-weight-normal px-2 py-1 m-1">Preview</span>
                    </div>
                </li>
            </ul>
        </li>
    </ul>
</template>

<template id="course_slides_list_slide" name="Slide template for a training channel">
    <li t-att-index="j" t-att-data-slide-id="slide.id" t-att-data-category-id="category_id" t-attf-class="o_wslides_slides_list_slide o_wslides_js_list_item bg-white-50 border-top-0 d-flex align-items-center pl-2 #{'py-1 pr-2' if not channel.can_upload else ''}">
        <div t-if="channel.can_publish" class=" o_wslides_slides_list_drag border-right p-2">
            <i class="fa fa-bars mr-2"></i>
        </div>
        <t t-call="website_slides.slide_icon">
            <t t-set="icon_class" t-value="'py-2 mx-2'"/>
        </t>
        <div class="text-truncate mr-auto">
            <a t-if="slide.is_preview or channel.is_member or channel.can_publish" class="o_wslides_js_slides_list_slide_link" t-attf-href="/slides/slide/#{slug(slide)}">
                <span t-field="slide.name"/>
            </a>
            <span t-else="">
                <span t-esc="slide.name"/>
            </span>
        </div>

        <div class="d-flex flex-row">
            <a name="o_wslides_list_slide_add_quizz" t-if="channel.can_upload and not slide.question_ids" t-attf-href="/slides/slide/#{slug(slide)}?quiz_quick_create">
                <span class="badge badge-light badge-hide border font-weight-normal px-2 py-1 m-1">Add Quiz</span>
            </a>
            <a t-if="channel.can_upload" href="#">
                <span t-att-data-slide-id="slide.id" t-attf-class="o_wslides_js_slide_toggle_is_preview badge #{'badge-success' if slide.is_preview else 'badge-light badge-hide border'} font-weight-normal px-2 py-1 m-1"><span>Preview</span></span>
            </a>
            <t t-elif="slide.is_preview and not channel.is_member">
                <span class="badge badge-success font-weight-normal px-2 py-1 m-1"><span>Preview</span></span>
            </t>
            <span t-if="slide.is_new_slide and not channel_progress[slide.id].get('completed')" class="badge badge-info badge-arrow-right font-weight-normal px-2 py-1 m-1">
                New
            </span>
            <span t-if="slide.question_ids" t-att-class="'badge font-weight-bold px-2 py-1 m-1 %s' % ('badge-success' if channel_progress[slide.id].get('completed') else 'badge-warning')">
                <i t-attf-class="fa fa-fw #{'fa-check' if channel_progress[slide.id].get('completed') else 'fa-flag'}"/>
                <t t-esc="channel_progress[slide.id].get('quiz_karma_won', 0) if channel_progress[slide.id].get('completed') else channel_progress[slide.id].get('quiz_karma_gain', 0)"/> xp
            </span>
            <span class="badge badge-danger font-weight-normal px-2 py-1 m-1" t-if="not slide.website_published">Unpublished</span>
        </div>

        <div t-if="channel.is_member or channel.can_publish" class="pt-2 pb-2 border-left ml-2 mr-2 pl-2 d-flex flex-row align-items-center o_wslides_slides_list_slide_controls">
            <t t-if="channel.is_member">
                <i t-if="not channel_progress[slide.id].get('completed')" class="check-done fa fa-circle-o text-500 px-2"></i>
                <i t-else="" class="check-done text-success fa fa-check-circle px-2"></i>
            </t>
            <span t-if="channel.can_publish" class="d-none d-md-flex">
                <a t-if="slide.slide_type == 'webpage'" class="px-2 o_text_link text-primary" target="_blank" t-attf-href="/slides/slide/#{slug(slide)}?enable_editor=1"><span class="fa fa-pencil"/></a>
                <a t-else="" class="px-2 o_text_link text-primary" target="_blank" t-attf-href="/web#id=#{slide.id}&amp;action=#{slide_action}&amp;model=slide.slide&amp;view_type=form" title="Edit in backend"><span class="fa fa-pencil"/></a>
                <a href="#" t-att-data-slide-id="slide.id" class="o_text_link text-danger px-2 o_wslides_js_slide_archive"><span class="fa fa-trash"/></a>
            </span>
        </div>
    </li>
</template>

<!-- ======= Documentation Course content: cards / categories=======  -->
<template id="course_promoted_slide" name="Documentation Course content: promoted slide">
    <div class="o_wslides_promoted_slide">
        <div t-if="not search and not search_slide_type and slide_promoted" class="container py-1 mb-2">
            <div class="card flex-column flex-lg-row">
                <t t-set="image_url" t-value="website.image_url(slide_promoted, 'image_1024')"/>

                <a t-if="slide_promoted.is_preview or channel.is_member or is_slides_publisher"
                t-attf-href="/slides/slide/#{slug(slide_promoted)}#{query_string}" class="w-100 w-lg-50 flex-shrink-0 rounded">
                    <div t-attf-style="background-image:url(#{image_url}); background-size: cover; background-position:center; padding-bottom: 35%;" class="h-lg-100"/>
                </a>
                <div t-else="" class="w-100 w-lg-50 flex-shrink-0 rounded">
                    <div t-attf-style="background-image:url(#{image_url}); background-size: cover; background-position:center; padding-bottom: 35%;" class="h-lg-100"/>
                </div>

                <div class="card-body">
                    <a t-if="slide_promoted.is_preview or channel.is_member or is_slides_publisher"
                    t-attf-href="/slides/slide/#{slug(slide_promoted)}#{query_string}"
                    class="h4 d-block" t-att-title="slide_promoted.name" t-field="slide_promoted.name"/>
                    <h4 t-else="" class="text-muted" t-field="slide_promoted.name"/>

                    <div t-if="slide_promoted.tag_ids" class="border-top mt-2 pt-1">
                        <t t-foreach="slide_promoted.tag_ids" t-as="tag">
                            <a t-att-href="'/slides/%s/tag/%s' % (slug(slide_promoted.channel_id), slug(tag))" class="badge badge-light" t-esc="tag.name"/>
                        </t>
                    </div>
                    <div class="o_wslides_desc_truncate_10 mt-3" t-field="slide_promoted.description"/>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="course_slides_cards" name="Documentation Course content: cards / categories">
    <div class="o_wslides_lesson_nav mb-4 border-bottom">
        <div class="container">
            <div class="row">
                <nav class="navbar navbar-expand-lg navbar-light col d-flex align-items-start">
                    <a class="navbar-brand d-lg-none" href="#">Filter &amp; order</a>

                    <div class="form-inline ml-auto d-lg-none" t-if="search_slide_type or search">
                        <a t-att-href="'/slides/%s' % (slug(channel))" class="btn btn-info mr-3">
                            <i class="fa fa-eraser mr-1"/>Clear filters
                        </a>
                    </div>

                    <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
                        <span class="navbar-toggler-icon"></span>
                    </button>

                    <div class="collapse navbar-collapse" id="navbarSupportedContent">
                        <ul class="navbar-nav mr-lg-auto align-items-lg-center">

                            <t t-set="slide_type_keys" t-value="slide_types.keys()"/>
                            <t t-foreach="slide_type_keys" t-as="slide_type_key">
                                <t t-if="search_category">
                                    <li t-if="search_category['nbr_%s' % slide_type_key] > 0" class="nav-item">
                                        <a t-att-href="'/slides/%s/category/%s?%s' % (slug(channel), slug(search_category), keep_query(slide_type=slide_type_key))"
                                           t-att-class="'nav-link d-flex align-items-center justify-content-between pl-0 mr-1 %s' % ('active' if search_slide_type == slide_type_key else '')">
                                           <t t-esc="slide_types[slide_type_key]"/>
                                           <span t-attf-class="badge badge-pill ml-1 #{'badge-info' if search_slide_type == slide_type_key else 'bg-400'}" t-esc="search_category['nbr_%s' % slide_type_key]"/>
                                        </a>
                                    </li>
                                </t>
                                <t t-else="">
                                    <li t-if="channel['nbr_%s' % slide_type_key] > 0" class="nav-item">
                                        <a t-att-href="'/slides/%s?%s' % (slug(channel), keep_query(slide_type=slide_type_key))"
                                           t-att-class="'nav-link d-flex align-items-center justify-content-between pl-0 mr-1 %s' % ('active' if search_slide_type == slide_type_key else '')">
                                           <t t-esc="slide_types[slide_type_key]"/>
                                           <span t-attf-class="badge badge-pill ml-1 #{'badge-info' if search_slide_type == slide_type_key else 'bg-400'}" t-esc="channel['nbr_%s' % slide_type_key]"/>
                                        </a>
                                    </li>
                                </t>
                            </t>
                        </ul>

                        <ul class="navbar-nav mr-auto">
                            <li class="nav-item dropdown ml-lg-auto">
                                <a class="nav-link dropdown-toggle dropdown-toggle align-items-center d-flex" type="button" id="slidesChannelDropdownSort"
                                   data-toggle="dropdown" aria-haspopup="true" aria-expanded="false" href="#">
                                    <b>Order by</b>
                                    <span class="d-none d-xl-inline">:
                                        <t t-if="sorting == 'most_voted'">Most Voted</t>
                                        <t t-elif="sorting == 'most_viewed'">Most Viewed</t>
                                        <t t-else="">Newest</t>
                                    </span>
                                </a>
                                <div class="dropdown-menu" aria-labelledby="slidesChannelDropdownSort" role="menu">
                                    <h6 class="dropdown-header">Sort by</h6>
                                    <a role="menuitem" t-att-href="'/slides/%s?%s' % (slug(channel), keep_query('slide_type', sorting='latest'))"
                                       t-att-class="'dropdown-item %s' % ('active' if sorting and sorting == 'latest' else '')">Newest</a>
                                    <a role="menuitem" t-att-href="'/slides/%s?%s' % (slug(channel), keep_query('slide_type', sorting='most_voted'))"
                                       t-att-class="'dropdown-item %s' % ('active' if sorting and sorting == 'most_voted' else '')">Most Voted</a>
                                    <a role="menuitem" t-att-href="'/slides/%s?%s' % (slug(channel), keep_query('slide_type', sorting='most_viewed'))"
                                       t-att-class="'dropdown-item %s' % ('active' if sorting and sorting == 'most_viewed' else '')">Most Viewed</a>
                                </div>
                            </li>
                        </ul>

                        <div class="form-inline mr-3 d-none d-lg-inline-block">
                            <a t-if="search_slide_type or search" t-att-href="'/slides/%s' % (slug(channel))" class="btn btn-sm btn-info ml-1">
                                <i class="fa fa-eraser mr-1"/>Clear filters
                            </a>
                        </div>

                        <form t-attf-action="/slides/#{slug(channel)}" role="search" method="get" class="form-inline my-2 my-lg-0">
                            <div class="input-group position-relative">
                                <input type="text" class="form-control border" name="search" placeholder="Search in content" t-att-value="search"/>
                                <div class="input-group-append">
                                    <button class="btn border" type="submit" aria-label="Search" title="Search">
                                        <i class="fa fa-search"/>
                                    </button>
                                </div>
                            </div>
                        </form>
                    </div>
                </nav>
            </div>
        </div>
    </div>

    <div class="container">
        <div class="row">
            <div class="mb-2 pt-1 text-left col">
                <t t-if="channel.tag_ids">
                    <t t-foreach="channel.tag_ids" t-as="channel_tag">
                        <span t-attf-class="badge o_wslides_channel_tag #{'o_tag_color_'+str(channel_tag.color)}" t-esc="channel_tag.name"/>
                    </t>
                </t>
                <a t-if="channel.can_upload"
                    class="o_wslides_js_channel_tag_add border badge badge-light font-weight-normal py-1 m-1"
                    role="button"
                    aria-label="Add Tag"
                    href="#"
                    t-att-data-channel-id="channel.id"
                    t-att-data-channel-tag-ids="channel.tag_ids.ids">
                    <i class="fa fa-plus mr-1"/><span>Add Tag</span>
                </a>
            </div>

            <div t-if="channel.can_upload" class="text-right pb-2 col-auto">
                <a class="btn btn-primary py-1 o_wslides_js_slide_upload"
                    title="Upload Presentation" role="button"
                    aria-label="Upload Presentation" href="#"
                    t-att-data-channel-id="channel.id"
                    t-att-data-can-upload="channel.can_upload"
                    t-att-data-can-publish="channel.can_publish">
                    <i class="fa fa-cloud-upload mr-1"/>Upload new content
                </a>
                <a class="btn btn-secondary py-1 o_wslides_js_slide_section_add"
                    title="Add Section" role="button"
                    aria-label="Add Section" href="#"
                    t-att-channel_id="channel.id">
                    <i class="fa fa-folder-o mr-1"/>Add a section
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
                No content was found using your search <span class="font-weight-bold" t-esc="search"/>.
            </div>
        </t>

        <t t-foreach="category_data" t-as="category">
            <div class="mb-2" t-if="(category['slides'] or channel.can_publish) and (search_category and search_category.id == category['id'] or not search_category)">
                <t t-set="is_empty_editable" t-value="not category['slides'] and channel.can_publish"/>
                <div class="d-flex align-items-center justify-content-between border-bottom pb-2 mb-3" t-if="category['id'] and query_string != '?search_uncategorized=1'">
                    <h5 t-attf-class="m-0 #{'text-muted' if is_empty_editable else ''}"><t t-esc="category['name']"/></h5>
                    <a t-if="category['id'] and not is_empty_editable" t-att-href="'/slides/%s/category/%s' % (slug(channel), category['slug_name'])">View all</a>
                </div>
                <div class="d-flex align-items-center justify-content-between border-bottom pb-2 mb-3" t-if="not category['id'] and len(category['slides']) > 0">
                    <h5 t-if="len(category_data) > 1" t-attf-class="m-0 #{'text-muted' if is_empty_editable else ''}"><t t-esc="category['name']"/></h5>
                    <a t-att-href="'/slides/%s?uncategorized=1' % (slug(channel))">View all</a>
                </div>
                <div class="row mx-n2">
                    <t t-foreach="category['slides']" t-as="slide">
                        <div class="col-12 col-sm-6 col-lg-3 px-2 d-flex flex-grow-1" t-call="website_slides.lesson_card"/>
                    </t>
                </div>
            </div>
        </t>

        <div class="row">
            <div class="col" t-field="channel.description_html"/>
        </div>
    </div>
    <t t-if="search_category or search_uncategorized">
        <div class="form-inline justify-content-center pb-5">
            <t t-call="website_profile.pager_nobox"></t>
        </div>
    </t>
</template>

<template id='lesson_card' name="Lesson Card">
    <div class="card w-100 o_wslides_lesson_card mb-4">
            <t t-if="slide.is_new_slide and not channel_progress[slide.id].get('completed')" t-call="website_slides.course_card_information"/>
        <t t-set="can_access" t-value="slide.is_preview or channel.is_member or channel.can_publish"/>

        <t t-if="slide.image_1024">
            <t t-set="lesson_image" t-value="website.image_url(slide, 'image_1024')"/>
            <a t-if="can_access" t-attf-href="/slides/slide/#{slug(slide)}#{query_string}" t-title="slide.name">
                <div class="card-img-top border-bottom" t-attf-style="padding-top: 50%; background-image: url(#{lesson_image}); background-size: cover; background-position:center"/>
            </a>
            <div t-else="" class="card-img-top border-bottom" t-attf-style="padding-top: 50%; background-image: url(#{lesson_image}); background-size: cover; background-position:center"/>
        </t>
        <t t-else="">
            <a t-if="can_access" t-attf-href="/slides/slide/#{slug(slide)}#{query_string}" t-title="slide.name">
                <div class="card-img-top border-bottom o_wslides_gradient" t-attf-style="padding-top: 50%;"/>
            </a>
            <div t-else="" class="card-img-top border-bottom o_wslides_gradient" t-attf-style="padding-top: 50%;"/>
        </t>
        <i t-if="channel_progress[slide.id].get('completed')" class="position-absolute py-1 px-2 h5 fa fa-check-circle text-primary" style="right:0; top:0;"/>

        <div class="card-body p-3">
            <a t-if="can_access" class="card-title h5 mb-2o_wslides_desc_truncate_2" t-attf-href="/slides/slide/#{slug(slide)}#{query_string}" t-esc="slide.name"/>
            <span t-else="" class="card-title h5 mb-2 o_wslides_desc_truncate_2 text-muted" t-esc="slide.name"/>
            <div class="card-subtitle mb-2 text-muted" t-if="slide.is_preview or (not slide.is_published and user.has_group('website_slides.group_website_slides_officer'))">
                <t t-if="slide.is_preview">
                    <span class="badge badge-info">Preview</span>
                </t>
                <t t-if="not slide.is_published and channel.can_publish">
                    <span class="badge badge-danger">Unpublished</span>
                </t>
            </div>
            <div class="card-text pt-2">
                <div class="o_wslides_desc_truncate_3 font-weight-light oe_no_empty" t-field="slide.description"/>
                <div t-if="slide.tag_ids" class="mt-2 pt-1 o_wslides_desc_truncate_2">
                    <t t-foreach="slide.tag_ids" t-as="tag">
                        <a t-att-href="'/slides/%s/tag/%s' % (slug(slide.channel_id), slug(tag))" class="badge badge-light" t-esc="tag.name"/>
                    </t>
                </div>
            </div>
        </div>
        <div class="card-footer bg-white text-600">
            <div class="d-flex align-items-center small">
                <span class="font-weight-bold mr-auto" t-field="slide.completion_time" t-options='{"widget": "float_time"}'/>
                <div class="o_wslides_js_slide_like mr-2">
                    <span t-att-class="'o_wslides_js_slide_like_up %s' % ('disabled' if not channel.can_vote else '')" tabindex="0" data-toggle="popover" t-att-data-slide-id="slide.id">
                        <i class="fa fa-thumbs-up fa-1x" role="img" aria-label="Likes" title="Likes"></i>
                        <span t-esc="slide.likes"/>
                    </span>
                    <span t-att-class="'o_wslides_js_slide_like_down %s' % ('disabled' if not channel.can_vote else '')" tabindex="0" data-toggle="popover" t-att-data-slide-id="slide.id">
                        <i class="fa fa-thumbs-down fa-1x" role="img" aria-label="Dislikes" title="Dislikes"></i>
                        <span t-esc="slide.dislikes"/>
                    </span>
                </div>
                <t t-if="channel.is_member and channel_progress[slide.id].get('completed')">
                    <span class="badge badge-pill badge-success"><i class="fa fa-check"/> Completed</span>
                </t>
            </div>
        </div>
    </div>
</template>

<template id="slide_icon">
    <t t-set="icon_class" t-value="icon_class if icon_class else 'mr-2 text-muted'"/>
    <i t-if="slide.slide_type == 'document'" t-att-class="'fa fa-file-pdf-o %s' % icon_class"></i>
    <i t-if="slide.slide_type == 'presentation'" t-att-class="'fa fa-file-pdf-o %s' % icon_class"></i>
    <i t-if="slide.slide_type == 'infographic'" t-att-class="'fa fa-file-picture-o %s' % icon_class"></i>
    <i t-if="slide.slide_type == 'video'" t-att-class="'fa fa-play-circle %s' % icon_class"></i>
    <i t-if="slide.slide_type == 'link'" t-att-class="'fa fa-file-code-o %s' % icon_class"></i>
    <i t-if="slide.slide_type == 'webpage'" t-att-class="'fa fa-file-text %s' % icon_class"></i>
    <i t-if="slide.slide_type == 'quiz'" t-att-class="'fa fa-question-circle %s' % icon_class"></i>
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
            <section class="s_banner overflow-hidden bg-900" style="background-image: url(&quot;/website_slides/static/src/img/banner_default.svg&quot;); background-size: cover; background-position: 55% 65%" data-snippet="s_banner">
                <div class="container align-items-center d-flex mb-5 mt-lg-5 pt-lg-4 pb-lg-1">
                    <div>
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
            <div class="container mt16 o_wslides_home_nav position-relative">
                <!-- TODO Remove inline style in master -->
                <nav class="navbar navbar-expand-lg navbar-light shadow-sm" style="background: white!important">
                    <form method="GET" class="form-inline o_wslides_nav_navbar_right order-lg-3" t-attf-action="/slides/all" role="search">
                        <div class="input-group">
                            <input type="search" name="search" class="form-control" placeholder="Search courses" aria-label="Search" t-att-value="search_term"/>
                            <div class="input-group-append">
                                <button class="btn border border-left-0 oe_search_button" type="submit" aria-label="Search" title="Search">
                                    <i class="fa fa-search"/>
                                </button>
                            </div>
                        </div>
                    </form>
                    <button class="navbar-toggler px-2 order-1" type="button"
                        data-toggle="collapse" data-target="#navbarSlidesHomepage"
                        aria-controls="navbarSlidesHomepage" aria-expanded="false" aria-label="Toggle navigation">
                        <span class="navbar-toggler-icon"/>
                    </button>
                    <div class="collapse navbar-collapse order-2" id="navbarSlidesHomepage">
                        <div class="navbar-nav pt-3 pt-lg-0">
                            <a class="nav-link nav-link mr-md-2 o_wslides_home_all_slides" href="/slides/all"><i class="fa fa-graduation-cap mr-1"/>All courses</a>
                        </div>
                    </div>
                </nav>
                <div class="o_wprofile_email_validation_container">
                    <t t-call="website_profile.email_validation_banner">
                        <t t-set="redirect_url" t-value="'/slides'"/>
                        <t t-set="send_alert_classes" t-value="'alert alert-danger alert-dismissable mt-4 mb-0'"/>
                        <t t-set="done_alert_classes" t-value="'alert alert-success alert-dismissable mt-4 mb-0'"/>
                        <t t-set="send_validation_email_message">Click here to send a verification email allowing you to participate at the eLearning.</t>
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
                                    <div class="pl-md-5 pl-lg-0">
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
                                        <t t-set="img_class" t-value="'rounded-circle mr-1'"/>
                                        <t t-set="img_style" t-value="'width: 22px; height: 22px; object-fit: cover;'"/>
                                    </t>
                                    <h5 t-esc="user.name" class="d-flex flex-grow-1 mb-0"/>
                                    <a class="d-none d-lg-block" t-att-href="'/profile/user/%s' % user.id">View</a>
                                    <a class="d-lg-none btn btn-sm bg-white border" href="#" data-toggle="collapse" data-target="#o_wslides_home_aside_content">More info</a>
                                </div>
                                <hr class="d-none d-lg-block mt-2 pt-2 mb-1"/>
                            </div>
                            <div id="o_wslides_home_aside_content" class="collapse d-lg-block">
                                <div class="row no-gutters mb-5 mt-3 mt-lg-0">
                                    <div class="col-12 col-sm-6 col-lg-12">
                                        <t t-call="website_slides.slides_home_user_profile_small"/>
                                    </div>
                                    <div class="col-12 col-sm-6 col-lg-12 pl-md-5 pl-lg-0 mt-lg-4">
                                        <t t-call="website_slides.slides_home_user_achievements_small"/>
                                    </div>
                                    <div class="col-12 col-md-7 col-lg-12 pl-md-5 pl-lg-0 mt-lg-4 mb-3">
                                        <t t-call="website_slides.slides_home_achievements_small"/>
                                    </div>
                                    <div class="col-12 col-sm-6 col-lg-12 pl-md-5 pl-lg-0 mt-lg-4">
                                        <t t-call="website_slides.slides_home_users_small"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div t-att-class="'col-lg-9 pr-lg-5 order-lg-1' if has_side_column else 'col-lg pr-lg'">
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
                                        <a href="/slides/all?my=1" class="float-right">View all</a>
                                        <h5 class="m-0">My courses</h5>
                                        <hr class="mt-2 pb-1 mb-1"/>
                                    </div>
                                </div>
                                <div class="row mx-n2 mt8">
                                    <t t-foreach="channels_my[:3]" t-as="channel">
                                        <div class="col-md-4 col-sm-6 px-2 col-xs-12 d-flex flex-grow-1">
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
                                    <a href="slides/all" class="float-right">View all</a>
                                    <h5 class="m-0">Most popular courses</h5>
                                    <hr class="mt-2 pb-1 mb-1"/>
                                </div>
                            </div>
                            <div class="row mx-n2 mt8">
                                <t t-foreach="channels_popular[:3]" t-as="channel">
                                    <div class="col-md-4 col-sm-6 px-2 col-xs-12 d-flex flex-grow-1">
                                        <t t-call="website_slides.course_card"/>
                                    </div>
                                </t>
                            </div>
                        </div>
                        <div class="o_wslides_home_content_section mb-3"
                            t-if="channels_newest">
                            <div class="row o_wslides_home_content_section_title align-items-center">
                                <div class="col">
                                    <a href="slides/all" class="float-right">View all</a>
                                    <h5 class="m-0">Newest courses</h5>
                                    <hr class="mt-2 pb-1 mb-1"/>
                                </div>
                            </div>
                            <div class="row mx-n2 mt8">
                                <t t-foreach="channels_newest[:3]" t-as="channel">
                                    <div class="col-md-4 col-sm-6 px-2 col-xs-12 d-flex flex-grow-1">
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
            <section class="s_banner bg-900" style="background-image: url(&quot;/website_slides/static/src/img/banner_default_all.svg&quot;); background-size: cover; background-position: 80% 20%" data-snippet="s_banner">
                <div class="container py-5">
                    <h1 t-if="search_my" class="display-3 mb-0">My Courses</h1>
                    <h1 t-elif="search_slide_type=='certification'" class="display-3 mb-0">Certifications</h1>
                    <h1 t-else="" class="display-3 mb-0">All Courses</h1>
                </div>
            </section>
            <div class="container mt16 o_wslides_home_nav position-relative">
                <!-- Navbar dynamically composed using displayed channel tag groups. -->
                <!-- TODO Remove inline style in master -->
                <nav class="navbar navbar-expand-md navbar-light shadow-sm pl-0" style="background: white!important">
                    <div class="navbar-nav border-right">
                        <a class="nav-link nav-item px-3" href="/slides"><i class="fa fa-chevron-left"/></a>
                    </div>
                    <!-- Clear filtering (mobile)-->
                    <div class="form-inline text-nowrap ml-auto d-md-none" t-if="search_slide_type or search_my or search_tags or search_channel_tag_id">
                        <a href="/slides/all" class="btn btn-info mr-2" role="button" title="Clear filters">
                            <i class="fa fa-eraser"/> Clear filters
                        </a>
                    </div>
                    <form t-else="" method="GET" class="form-inline o_wslides_nav_navbar_right d-md-none">
                        <!-- Search box (mobile)-->
                        <div class="input-group">
                            <input type="search" name="search" class="form-control"
                                placeholder="Search courses" aria-label="Search"
                                t-att-value="search_term"/>
                            <input t-if="search_tags" type="hidden" name="tags" t-att-value="str(search_tags.ids)"/>
                            <input t-if="search_my" type="hidden" name="my" t-att-value="1"/>
                            <input t-if="search_slide_type" type="hidden" name="slide_type" t-att-value="search_slide_type" />
                            <div class="input-group-append">
                                <button class="btn border border-left-0 oe_search_button" type="submit" aria-label="Search" title="Search">
                                    <i class="fa fa-search"/>
                                </button>
                            </div>
                        </div>
                    </form>
                    <button class="navbar-toggler px-1" type="button"
                        data-toggle="collapse" data-target="#navbarTagGroups"
                        aria-controls="navbarTagGroups" aria-expanded="false" aria-label="Toggle navigation">
                        <span class="navbar-toggler-icon small"/>
                    </button>
                    <div class="collapse navbar-collapse" id="navbarTagGroups">
                        <t t-set="search_tag_groups" t-value="search_tags.mapped('group_id')"/>
                        <ul class="navbar-nav flex-grow-1">
                            <t t-foreach="tag_groups" t-as="tag_group">
                                <li t-att-class="'nav-item dropdown ml16 %s' % ('active' if tag_group in search_tag_groups else '')">
                                    <a class="nav-link dropdown-toggle"
                                        href="/slides/all"
                                        t-att-data-target="'#navToogleTagGroup%s' % tag_group.id"
                                        role="button" data-toggle="dropdown"
                                        aria-haspopup="true" aria-expanded="false"
                                        t-esc="tag_group.name"/>
                                    <div class="dropdown-menu" t-att-id="'navToogleTagGroup%s' % tag_group.id">
                                        <t t-foreach="tag_group.tag_ids" t-as="tag">
                                            <a rel="nofollow" t-att-class="'dropdown-item %s' % ('active' if tag in search_tags else '')"
                                                t-att-href="'/slides/all?%s' % keep_query('*', tags=str((search_tags - tag).ids if tag in search_tags else (tag | search_tags).ids))"
                                                t-esc="tag.name"/>
                                        </t>
                                    </div>
                                </li>
                            </t>
                        </ul>
                        <!-- Clear filtering (desktop)-->
                        <div class="form-inline ml-auto d-none d-md-flex" t-if="search_slide_type or search_my or search_tags or search_channel_tag_id">
                            <a href="/slides/all" class="btn btn-info text-nowrap mr-2" role="button" title="Clear filters">
                                <i class="fa fa-eraser"/> Clear filters
                            </a>
                        </div>
                        <!-- Search box (desktop) -->
                        <form method="GET" class="form-inline o_wslides_nav_navbar_right d-none d-md-flex">
                            <div class="input-group">
                                <input type="search" name="search" class="form-control"
                                    placeholder="Search courses" aria-label="Search"
                                    t-att-value="search_term"/>
                                <input t-if="search_tags" type="hidden" name="tags" t-att-value="str(search_tags.ids)"/>
                                <input t-if="search_my" type="hidden" name="my" t-att-value="1"/>
                                <input t-if="search_slide_type" type="hidden" name="slide_type" t-att-value="search_slide_type" />
                                <div class="input-group-append">
                                    <button class="btn border border-left-0 oe_search_button" type="submit" aria-label="Search" title="Search">
                                        <i class="fa fa-search"/>
                                    </button>
                                </div>
                            </div>
                        </form>
                    </div>
                </nav>
                <div class="o_wprofile_email_validation_container mb16 mt16">
                    <t t-call="website_profile.email_validation_banner">
                        <t t-set="redirect_url" t-value="'/slides'"/>
                        <t t-set="send_validation_email_message">Click here to send a verification email allowing you to participate at the eLearning.</t>
                        <t t-set="additional_validated_email_message"> You may now participate in our eLearning.</t>
                    </t>
                </div>
                <!-- Display tags -->
                <t t-if="search_my">
                      <span class="align-items-baseline border d-inline-flex pl-2 rounded mb-2">
                      <i class="fa fa-tag mr-2 text-muted"/>
                      My Courses
                      <a t-att-href="'/slides/all?%s' % keep_query('*', my=None)" class="btn border-0 py-1">&#215;</a>
                    </span>
                </t>
                <t t-if="search_term">
                      <span class="align-items-baseline border d-inline-flex pl-2 rounded mb-2">
                      <i class="fa fa-tag mr-2 text-muted"/>
                      <t t-esc="search_term"/>
                      <a t-att-href="'/slides/all?%s' % keep_query('*', search=None)" class="btn border-0 py-1">&#215;</a>
                    </span>
                </t>
                <t t-foreach="search_tags" t-as="tag">
                    <span class="align-items-baseline border d-inline-flex pl-2 rounded mb-2">
                        <i class="fa fa-tag mr-2 text-muted"/>
                        <t t-esc="tag.display_name"/>
                        <a t-att-href="'/slides/all?%s' % keep_query('*', tags=str((search_tags - tag).ids))" class="btn border-0 py-1">&#215;</a>
                    </span>
                </t>
            </div>
            <div class="container o_wslides_home_main pb-5">
                <div t-if="not channels and not search_term and not search_slide_type and not search_my and not search_tags and not search_channel_tag_id">
                    <p class="h2">No Course created yet.</p>
                    <p groups="website_slides.group_website_slides_officer">Click on "New" in the top-right corner to write your first course.</p>
                </div>
                <div t-elif="search_term and not channels" class="alert alert-info mb-5">
                    No course was found matching your search <code><t t-esc="search_term"/></code>.
                </div>
                <div t-elif="not channels" class="alert alert-info mb-5">
                    No course was found matching your search.
                </div>
                <div t-else="" class="row mx-n2">
                    <t t-foreach="channels" t-as="channel">
                        <div class="col-12 col-sm-6 col-md-4 col-lg-3 px-2 d-flex flex-grow-1">
                            <t t-call="website_slides.course_card"/>
                        </div>
                    </t>
                </div>
            </div>

            <t t-call="website_slides.courses_footer"></t>
        </div>
    </t>
</template>

<template id='courses_footer'>
    <section class="s_banner">
        <div class="oe_structure oe_empty"/>
    </section>
</template>

<template id='course_card' name="Course Card">
    <div t-attf-class="card w-100 o_wslides_course_card mb-4 #{'o_wslides_course_unpublished' if not channel.is_published else ''}">
        <t t-set="course_image" t-value="website.image_url(channel, 'image_1024')"/>
        <a t-attf-href="/slides/#{slug(channel)}" t-title="channel.name">
            <t t-if="channel.partner_has_new_content" t-call="website_slides.course_card_information"/> 
            <div t-if="channel.image_1024" class="card-img-top" t-attf-style="padding-top: 50%; background-image: url(#{course_image}); background-size: cover; background-position:center"/>
            <div t-else="" class="o_wslides_gradient card-img-top position-relative" style="padding-top: 50%; opacity: 0.8">
                <i class="fa fa-graduation-cap fa-2x mr-3 mb-3 position-absolute text-white-75" style="right:0; bottom: 0"/>
            </div>
        </a>
        <div class="card-body p-3">
            <a class="card-title h5 mb-2 o_wslides_desc_truncate_2" t-attf-href="/slides/#{slug(channel)}" t-esc="channel.name"/>
            <span t-if="not channel.is_published" class="badge badge-danger p-1">Unpublished</span>
            <div class="card-text mt-1">
                <div class="font-weight-light o_wslides_desc_truncate_3" t-field="channel.description_short"/>
                <div t-if="channel.tag_ids" class="mt-2 pt-1 o_wslides_desc_truncate_2">
                    <t t-foreach="channel.tag_ids" t-as="tag">
                        <t t-if="search_tags">
                            <a t-att-href="'/slides/all?%s' % keep_query('*', tags=str((tag | search_tags).ids))" t-attf-class="badge #{'badge-primary' if tag in search_tags else 'o_wslides_channel_tag o_tag_color_0'}" t-esc="tag.name"/>
                        </t>
                        <t t-else="">
                            <a t-att-href="'/slides/all?%s' % keep_query('*', tags=str((tag | search_tags).ids))" t-attf-class="badge o_wslides_channel_tag #{'o_tag_color_'+str(tag.color)}" t-esc="tag.name"/>
                        </t>
                    </t>
                </div>
            </div>
        </div>
        <div class="card-footer bg-white text-600 px-3">
            <div class="d-flex justify-content-between align-items-center">
                <small t-if="channel.total_time" class="font-weight-bold" t-esc="channel.total_time" t-options="{'widget': 'duration', 'unit': 'hour', 'round': 'minute'}"/>
                <div class="d-flex flex-grow-1 justify-content-end">
                    <t t-if="channel.is_member and channel.completed">
                        <span class="badge badge-pill badge-success pull-right py-1 px-2"><i class="fa fa-check"/> Completed</span>
                    </t>
                    <div t-elif="channel.is_member and channel.channel_type != 'documentation'" class="progress w-50" style="height: 6px">
                        <div class="progress-bar" role="progressbar" t-att-aria-valuenow="channel.completion" aria-valuemin="0" aria-valuemax="100" t-attf-style="width:#{channel.completion}%;"/>
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
    active="True" customize_show="True" name='New Content Ribbon'>
    <xpath expr="//t[@id='course_card_information_content']" position="inside">
        <span class="o_wslides_arrow">New Content</span>
    </xpath>
</template>

<template id='slides_home_achievements_small' name="Users">
    <t class="o_wslides_home_aside">
    </t>
</template>

<template id="toggle_latest_achievements" inherit_id="website_slides.slides_home_achievements_small" active="True" customize_show="True" name='Display Achievements'>
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
    <div class="d-flex no-gutters mt8 align-items-center">
        <t t-call="website_slides.slides_misc_user_image">
            <t t-set="user" t-value="achievement.user_id"/>
        </t>
        <div style="line-height: 1.3">
            <span class="font-weight-bold" t-esc="achievement.user_id.name"/> achieved <span class="font-weight-bold" t-esc="achievement.badge_id.name"/>
        </div>
    </div>
</template>

<template id='slides_home_users_small' name="Users">
    <div class="o_wslides_home_aside">
    </div>
</template>

<template id="toggle_leaderboard" inherit_id="website_slides.slides_home_users_small" active="True" customize_show="True" name='Display Leaderboard'>
    <xpath expr="//div[hasclass('o_wslides_home_aside')]" position="inside">
        <div class="row o_wslides_home_aside_title">
            <div class="col">
                <a href="/profile/users" class="float-right">View all</a>
                <h5 class="m-0">Leaderboard</h5>
                <hr class="mt-2 pt-2"/>
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
        <b class="mr-2 text-muted" t-esc="counter"/>
        <t t-call="website_slides.slides_misc_user_image"/>
        <div style="line-height:1.3">
            <span class="font-weight-bold" t-esc="user.name"/>
            <div class="d-flex align-items-center">
                <t t-esc="user.rank_id.name"/>
                <span class="text-500 mx-2">&#8226;</span>
                <span class="badge badge-success"><t t-esc="user.karma"/> xp</span>
            </div>
        </div>
    </div>
</template>

<template id='slides_home_user_profile_small' name="User Profile">
    <div class="o_wslides_home_aside">
        <div t-if="user.rank_id" class="d-flex align-items-center">
            <span class="font-weight-bold text-muted mr-2">Current rank:</span>
            <img t-att-src="website.image_url(user.rank_id, 'image_128')" width="16" height="16" alt="" class="o_object_fit_cover mr-1"/>
            <a href="/profile/ranks_badges" t-field="user.rank_id"/>
        </div>
        <t t-set="next_rank_id" t-value="user._get_next_rank()"/>
        <div t-if="next_rank_id" class="font-weight-bold text-muted mt-1">Next rank:</div>
        <t t-if="next_rank_id or user.rank_id" t-call="website_profile.profile_next_rank_card">
            <t t-set="bg_class" t-valuef="bg-200"/>
            <t t-set="img_max_width" t-valuef="50%"/>
        </t>
        <div t-if="next_rank_id" t-field="next_rank_id.description_motivational"/>
        <div t-else="">Congratulations, you have reached the last rank!</div>
    </div>
</template>

<template id='slides_home_user_achievements_small' name="User Achievements">
    <div class="o_wslides_home_aside flex-grow-1">
        <div class="row o_wslides_home_aside_title">
            <div class="col">
                <a href="/profile/ranks_badges?badge_category=slides" class="float-right">View all</a>
                <h5 class="m-0">Badges</h5>
                <hr class="mt-2 pt-2"/>
            </div>
        </div>
        <t t-foreach="challenges" t-as="challenge">
            <t t-set="challenge_done" t-value="challenge in challenges_done if challenges_done else False"/>
            <div t-attf-class="d-flex mb-3 align-items-center #{'o_wslides_entry_muted' if not challenge_done else ''}">
                <img class="mr-2"
                    style="max-height: 36px;"
                    t-att-src="website.image_url(challenge.reward_id, 'image_128') if challenge.reward_id.image_1920 else
                               '/website_profile/static/src/img/badge_%s.svg' % (challenge.reward_id.level)"
                    t-att-alt="challenge.reward_id.name"/>
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
    <img t-att-class="img_class if img_class else 'rounded-circle float-left'"
        t-att-style="img_style if img_style else 'width: 32px; height: 32px; object-fit: cover;'"
        t-att-src="'/profile/avatar/%s?field=image_128' % user.id"
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
            <div class="o_wslides_lesson_header o_wslides_gradient position-relative text-white pb-0 pt-2 pt-md-5">
                <t t-call="website_slides.course_nav">
                    <t t-set="channel" t-value="slide.channel_id"/>
                </t>
                <div class="container o_wslides_lesson_header_container mt-5 mt-md-3 mt-xl-4">
                    <div class="row align-items-end align-items-md-stretch">
                        <div t-attf-class="col-12 col-lg-9 d-flex flex-column #{'offset-lg-3' if slide.channel_id.channel_type == 'training' else ''}">
                            <h2 class="font-weight-medium w-100">
                                <a t-att-href="'/slides/%s' % (slug(slide.channel_id))" class="text-white text-decoration-none" t-field="slide.channel_id.name"/>
                                <t t-if="slide.channel_id.completed">
                                    <small><span class="badge badge-pill badge-success pull-right my-1 py-1 px-2 font-weight-normal"><i class="fa fa-check"/> Completed</span></small>
                                </t>
                            </h2>

                            <div t-if="slide.channel_id.channel_type == 'documentation'" class="mb-3 small">
                                <span class="font-weight-normal">Last update:</span>
                                <t t-esc="slide.date_published" t-options="{'widget': 'date'}"/>
                            </div>

                            <div t-else="" t-if="not slide.channel_id.completed" class="d-flex align-items-center pb-3">
                                <div class="progress w-50 bg-black-25" style="height: 10px;">
                                    <div class="progress-bar rounded-left" role="progressbar"
                                        t-att-aria-valuenow="slide.channel_id.completion" aria-valuemin="0" aria-valuemax="100"
                                        t-attf-style="width: #{slide.channel_id.completion}%;">
                                    </div>
                                </div>
                                <i t-att-class="'fa fa-trophy m-0 ml-2 p-0 %s' % ('text-warning' if slide.channel_id.completed else 'text-black-50')"></i>
                                <small class="ml-2 text-white-50"><t t-esc="slide.channel_id.completion"/> %</small>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="container o_wslides_lesson_main">
                <div class="row">
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
            <li class="nav-item"><a aria-controls="related" href="#related" class="nav-link rounded-0 border-top-0 border-left-0 py-2 active" data-toggle="tab">Related</a></li>
            <li class="nav-item"><a aria-controls="most_viewed" href="#most_viewed" class="nav-link rounded-0 border-top-0 border-right-0 py-2" data-toggle="tab">Most Viewed</a></li>
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
        <t t-set="slide_image" t-value="website.image_url(aside_slide, 'image_1024')"/>

        <div t-if="aside_slide.image_1024" class="flex-shrink-0 mr-1 border" t-attf-style="width: 20%; padding-top: 20%; background-image: url(#{slide_image}); background-size: cover; background-position:center"/>
        <div t-else="" class="o_wslides_gradient flex-shrink-0 mr-1" t-attf-style="width: 20%; padding-top: 20%;"/>
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
        <div class="bg-100 text-600 h6 my-0 text-decoration-none border-bottom d-flex align-items-center justify-content-between">
            <span class="p-2">Course content</span>
            <a href="#collapse_slide_aside" data-toggle="collapse" class="d-lg-none p-2 text-decoration-none o_wslides_lesson_aside_collapse">
                <i class="fa fa-chevron-down d-lg-none"/>
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
    <t t-if="category" t-set="category" t-value="category.get('category')"/>
    <li class="o_wslides_fs_sidebar_section mt-2">
        <a t-att-href="('#collapse-%s') % (category.id if category else 0)" data-toggle="collapse" role="button" aria-expanded="true"
            class="o_wslides_lesson_aside_list_link pl-2 text-600 text-uppercase text-decoration-none py-1 small d-block"
            t-att-aria-controls="('collapse-%s') % (category.id if category else 0)">
            <t t-if="category">
                <b t-field="category.name"/>
            </t>
            <t t-else="">
                <b>Uncategorized</b>
            </t>
        </a>
        <ul class="collapse show p-0 m-0 list-unstyled" t-att-id="('collapse-%s') % (category.id if category else 0)" >
            <t t-set="is_member" t-value="slide.channel_id.is_member"/>
            <t t-set="can_access_channel" t-value="is_member or slide.channel_id.can_publish"/>
            <t t-foreach="category_slide_ids" t-as="aside_slide">
                <t t-set="slide_completed" t-value="channel_progress[aside_slide.id].get('completed')"/>
                <t t-set="can_access" t-value="can_access_channel or aside_slide.is_preview"/>
                <li class="p-0 pb-1">
                    <a t-att-href="'/slides/slide/%s' % (slug(aside_slide)) if can_access else '#'"
                        t-att-class="'o_wslides_lesson_aside_list_link d-flex align-items-top px-2 pt-1 text-decoration-none %s%s' % (('bg-100 py-1 active' if aside_slide == slide else ''), 'text-muted' if not can_access else '')">
                        <div t-if="is_member" >
                            <i t-att-id="'o_wslides_lesson_aside_slide_check_%s' % (aside_slide.id)"
                                t-att-class="'mr-1 fa fa-fw %s' % ('text-success fa-check-circle' if channel_progress[aside_slide.id].get('completed') else 'text-600 fa-circle')">
                            </i>
                        </div>
                        <div class="o_wslides_lesson_link_name text-truncate">
                            <t t-call="website_slides.slide_icon">
                                <t t-set="slide" t-value="aside_slide"/>
                            </t>
                            <span t-esc="aside_slide.name" class="mr-2"/>
                        </div>
                        <div class="ml-auto" t-if="aside_slide.question_ids">
                            <span t-att-class="'badge badge-pill %s' % ('badge-success' if channel_progress[aside_slide.id].get('completed') else 'badge-light text-600')">
                                <t t-esc="channel_progress[aside_slide.id].get('quiz_karma_won') if channel_progress[aside_slide.id].get('completed') else channel_progress[aside_slide.id].get('quiz_karma_gain')"/> xp
                            </span>
                        </div>
                    </a>
                    <ul t-if="aside_slide.link_ids or aside_slide._has_additional_resources() or aside_slide.question_ids" class="list-group px-2 mb-1 list-unstyled">
                        <t t-foreach="aside_slide.link_ids" t-as="resource">
                            <li class="pl-4">
                                <a t-if="can_access_channel" t-att-href="resource.link" target="new" class="text-decoration-none small">
                                    <i class="fa fa-link mr-1"/><span t-field="resource.name"/>
                                </a>
                                <span t-else="" class="text-decoration-none text-muted small">
                                    <i class="fa fa-link mr-1"/><span t-field="resource.name"/>
                                </span>
                            </li>
                        </t>
                        <div class="o_wslides_js_course_join pl-4" t-if="aside_slide._has_additional_resources()">
                            <t t-if="can_access_channel">
                                <li t-foreach="aside_slide.slide_resource_ids" t-as="resource">
                                    <a t-attf-href="/web/content/slide.slide.resource/#{resource.id}/data?download=true" class="text-decoration-none small">
                                        <i class="fa fa-download mr-1"/><span t-field="resource.name"/>
                                    </a>
                                </li>
                            </t>
                            <li t-elif="aside_slide.channel_id.enroll == 'public'" class="text-decoration-none small">
                                <i class="fa fa-download mr-1"/>
                                <t t-call="website_slides.join_course_link"/>
                            </li>
                        </div>
                        <li class="pl-4">
                            <a t-if="can_access and aside_slide.question_ids and aside_slide.slide_type != 'quiz'" t-att-href="'/slides/slide/%s#lessonQuiz' % (slug(aside_slide))" class="o_wslides_lesson_aside_list_link text-decoration-none small text-600">
                                <i class="fa fa-flag text-warning"/> Quiz
                            </a>
                            <span t-elif="not can_access and aside_slide.question_ids and aside_slide.slide_type != 'quiz'"
                                class="o_wslides_lesson_aside_list_link text-decoration-none small text-600 text-muted">
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
    <t t-set="is_training_channel" t-value="slide.channel_id.channel_type == 'training'"/>
    <div class="row align-items-center my-3">
        <div class="col-12 col-md order-2 order-md-1 d-flex">
            <div class="d-flex align-items-center overflow-hidden">
                <h1 class="h4 my-0 text-truncate">
                    <t t-call="website_slides.slide_icon">
                        <t t-set="icon_class" t-valuef="mr-1"/>
                    </t>
                    <span t-field="slide.name"/>
                </h1>
                <span t-if="slide.question_ids"
                    t-att-class="'ml-2 badge %s' % ('badge-success' if channel_progress[slide.id].get('completed') else 'badge-info')">
                    <span t-if="channel_progress[slide.id].get('completed')">
                        <i class="fa fa-check-circle"/>
                        <t t-esc="channel_progress[slide.id].get('quiz_karma_won', 0)"/>
                    </span>
                    <span t-else="" t-esc="channel_progress[slide.id].get('quiz_karma_gain', 0)"/>
                    <span>XP</span>
                </span>
            </div>
        </div>
        <div class="col-12 col-md order-1 order-md-2 text-nowrap flex-grow-0 d-flex justify-content-end mb-3 mb-md-0">
            <div class="btn-group flex-grow-1 flex-sm-0" role="group" aria-label="Lesson Nav">
                <a t-att-class="'btn btn-light border %s' % ('disabled' if not previous_slide else '')"
                    role="button" t-att-aria-disabled="'disabled' if not previous_slide else None"
                    t-att-href="'/slides/slide/%s' % (slug(previous_slide)) if previous_slide else '#'">
                    <i class="fa fa-chevron-left mr-2"></i> <span class="d-none d-sm-inline-block">Prev</span>
                </a>
                <t t-set="allow_done_btn" t-value="slide.slide_type in ['infographic', 'presentation', 'document', 'webpage', 'video'] and not slide.question_ids and not channel_progress[slide.id].get('completed') and slide.channel_id.is_member"/>
                <a t-att-class="'btn btn-primary border text-white %s' % ('disabled' if not allow_done_btn else '')"
                    role="button" t-att-aria-disabled="'true' if not allow_done_btn else None"
                    t-att-href="'/slides/slide/%s/set_completed?%s' % (slide.id, 'next_slide_id=%s' % (next_slide.id) if next_slide else '') if allow_done_btn else '#'">
                    Set Done
                </a>
                <a t-att-class="'btn btn-light border %s' % ('disabled' if not next_slide else '')"
                    role="button" t-att-aria-disabled="'disabled' if not next_slide else None"
                    t-att-href="'/slides/slide/%s' % (slug(next_slide)) if next_slide else '#'">
                    <span class="d-none d-sm-inline-block">Next</span> <i class="fa fa-chevron-right ml-2"></i>
                </a>
            </div>
            <a t-if="is_training_channel" class="btn btn-light border ml-2" role="button" t-att-href="'/slides/slide/%s?fullscreen=1' % (slug(slide))">
                <i class="fa fa-desktop mr-2"/>
                <span class="d-none d-sm-inline-block">Fullscreen</span>
            </a>
        </div>
    </div>
    <div t-if="slide.tag_ids" class="pb-2">
        <t t-foreach="slide.tag_ids" t-as="tag">
            <a t-att-href="'/slides/%s/tag/%s' % (slug(slide.channel_id), slug(tag))" class="badge badge-info py-1 px-2" t-esc="tag.name"/>
        </t>
    </div>
    <div class="o_wslides_lesson_content_type">
        <img t-if="slide.slide_type == 'infographic'"
            t-att-src="website.image_url(slide, 'image_1024')" class="img-fluid" style="width:100%" t-att-alt="slide.name"/>
        <div t-if="slide.slide_type in ('presentation', 'document')" class="embed-responsive embed-responsive-4by3 embed-responsive-item mb8" style="height: 600px;">
            <t t-raw="slide.embed_code"/>
        </div>
        <div t-if="slide.slide_type == 'video' and slide.document_id" class="embed-responsive embed-responsive-16by9 embed-responsive-item mb8">
            <t t-raw="slide.embed_code"/>
        </div>
        <div t-if="slide.slide_type == 'webpage'" class="bg-white p-3">
            <div t-field="slide.html_content"/>
        </div>
    </div>

    <div class="mb-5">
        <ul class="nav nav-tabs o_wslides_lesson_nav" role="tablist">
            <li class="nav-item">
                <a href="#about" aria-controls="about" class="nav-link active" role="tab" data-toggle="tab">
                    <i class="fa fa-home"></i> About
                </a>
            </li>
            <li t-if="slide.channel_id.allow_comment" class="nav-item">
                <a href="#discuss" aria-controls="discuss" class="nav-link" role="tab" data-toggle="tab">
                    <i class="fa fa-comments"></i> Comments (<span t-esc="slide.comments_count"/>)
                </a>
            </li>
            <li class="nav-item">
                <a href="#statistic" aria-controls="statistic" class="nav-link" role="tab" data-toggle="tab">
                    <i class="fa fa-bar-chart"></i> Statistics
                </a>
            </li>
            <li class="nav-item">
                <a href="#share" aria-controls="share" class="nav-link" role="tab" data-toggle="tab">
                    <i class="fa fa-share-alt"></i> Share
                </a>
            </li>
        </ul>
        <div class="tab-content mt-3">
            <div role="tabpanel" t-att-class="not comments and 'tab-pane fade in show active' or 'tab-pane fade'" id="about">
                <div t-field="slide.description"/>
            </div>
            <div role="tabpanel" t-att-class="comments and 'tab-pane fade in show active' or 'tab-pane fade'" id="discuss">
                <t t-call="portal.message_thread">
                    <t t-set="object" t-value="slide"/>
                    <t t-set="disable_composer" t-value="not (slide.channel_id.can_comment and slide.channel_id.allow_comment and slide.channel_id.channel_type == 'training')"/>
                    <t t-set="display_rating" t-value="False"/>
                </t>
            </div>
            <div role="tabpanel" class="tab-pane fade" id="statistic" t-att-slide-url="slide.website_url">
                <div class="row">
                    <div class="col-12 col-md">
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
                    <div t-if="slide.channel_id.allow_comment" class="col-12 col-md">
                        <table class="table table-sm">
                            <tbody>
                                <tr>
                                    <th colspan="2" class="border-top-0">Actions</th>
                                </tr>
                                <tr class="border-top-0">
                                    <th class="border-top-0"><span t-esc="slide.likes"/></th>
                                    <td class="border-top-0 w-100">Likes</td>
                                </tr>
                                <tr>
                                    <th><span t-esc="slide.dislikes"/></th>
                                    <td>Dislikes</td>
                                </tr>
                                <tr>
                                    <th><span t-esc="len(slide.website_message_ids)"/></th>
                                    <td>Comments</td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
            <div role="tabpanel" class="tab-pane fade" t-if="slide.website_published" id="share">
                <h4 t-if="not slide.website_published"><i class="fa fa-info-circle"></i>
                    The social sharing module will be unlocked when a moderator will allow your publication.
                </h4>
                <t t-if="slide.website_published">
                    <div class="row">
                        <div class="col-12 col-lg-6">
                            <h5 class="mt16">Share on Social Networks</h5>
                            <t t-call="website_slides.slide_share_social">
                                <t t-set="record" t-value="slide"/>
                            </t>
                        </div>
                        <div class="col-12 col-lg-6">
                            <t t-call="website_slides.slide_share_link">
                                <t t-set="record" t-value="slide"/>
                                <t t-set="website_url" t-value="slide.website_url"/>
                            </t>
                        </div>
                    </div>
                    <div class="row">
                        <div t-attf-class="col-12 #{'col-lg-6' if slide.embed_code else ''}">
                            <t t-call="website_slides.slide_social_email">
                                <t t-set="slide" t-value="slide"/>
                            </t>
                        </div>
                        <div class="col-12 col-lg-6" t-if="slide.embed_code">
                            <t t-call="website_slides.slide_social_embed">
                                <t t-set="slide" t-value="slide"/>
                            </t>
                        </div>
                    </div>
                </t>
            </div>
        </div>
    </div>
    <div class="o_wslides_js_quiz_container" t-att-data-slide-id="slide.id">
        <div class="row" t-if="slide.slide_type != 'certification'">
            <t t-if="slide.question_ids">
                <t t-call="website_slides.lesson_content_quiz"/>
            </t>
            <div t-else="" class="o_wslides_js_lesson_quiz col" t-att-data-id="slide.id">
                <t t-if="slide.channel_id.can_upload" t-call="website_slides.lesson_content_quiz_add_buttons"/>
            </div>
        </div>
    </div>
    <div class="row mt-3 mb-3">
        <div class="col-12 col-md d-flex align-items-start mb-4 mb-md-0" t-if="len(slide.link_ids)">
            <span t-if="slide.link_ids" class="text-muted font-weight-bold mr-3">External sources</span>
            <div class="text-muted mr-auto border-left pl-3">
                <t t-foreach="slide.link_ids" t-as="link">
                    <a t-att-href="link.link" t-esc="link.name"/><br />
                </t>
            </div>
        </div>
        <div class="col-12 col-md d-flex align-items-start mb-4 mb-md-0 o_wslides_js_course_join" t-if="slide._has_additional_resources()">
            <span t-if="slide.channel_id.is_member or slide.channel_id.can_publish or slide.is_preview or slide.channel_id.enroll in ['private', 'payment']" class="text-muted font-weight-bold mr-3">
                Additional Resources
            </span>
            <div t-if="slide.channel_id.is_member or slide.channel_id.can_publish" class="text-muted mr-auto border-left pl-3">
                <t t-foreach="slide.slide_resource_ids" t-as="resource">
                    <a t-attf-href="/web/content/slide.slide.resource/#{resource.id}/data?download=true" t-esc="resource.name"/><br />
                </t>
            </div>
            <div t-elif="slide.channel_id.enroll == 'public'" class="text-muted mr-auto border-left pl-3">
                <t t-call="website_slides.join_course_link"/>
            </div>
        </div>
        <div t-if="slide.channel_id.allow_comment and slide.channel_id.channel_type == 'documentation'"
             class="col-12 col-md d-flex align-items-start justify-content-md-end mb-2 mb-md-0">
            <span class="text-muted font-weight-bold mr-3">Rating</span>
            <div class="text-muted border-left pl-3">
                <div class="o_wslides_js_slide_like mr-2">
                    <span t-att-class="('o_wslides_js_slide_like_up %s') % ('disabled' if not slide.channel_id.can_vote else '')" tabindex="0" data-toggle="popover" t-att-data-slide-id="slide.id">
                        <i class="fa fa-thumbs-up fa-1x" role="img" aria-label="Likes" title="Likes"></i>
                        <span t-esc="slide.likes"/>
                    </span>
                    <span t-att-class="('o_wslides_js_slide_like_down ml-3 %s') % ('disabled' if not slide.channel_id.can_vote else '')" tabindex="0" data-toggle="popover" t-att-data-slide-id="slide.id">
                        <i class="fa fa-thumbs-down fa-1x" role="img" aria-label="Dislikes" title="Dislikes"></i>
                        <span t-esc="slide.dislikes"/>
                    </span>
                </div>
            </div>
        </div>
    </div>
</template>

<!-- Slide sub-tempalte: render a quiz serverside. Should be sync with JS qweb template "slide.slide.quiz" -->
<template id="lesson_content_quiz" name="Lesson: Quiz specific content">
    <t t-set="slide_completed" t-value="channel_progress[slide.id].get('completed')"/>
    <div class="o_wslides_js_lesson_quiz col" id="lessonQuiz"
        t-att-data-id="slide.id"
        t-att-data-name="slide.name"
        t-att-data-slide-type="slide.slide_type"
        t-att-data-is-member="slide.channel_id.is_member"
        t-att-data-completed="1 if slide_completed else 0"
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
                    <i class="o_wslides_js_quiz_icon o_wslides_js_quiz_sequence_handler fa fa-bars mr-1 text-muted" t-if="slide.channel_id.can_upload and not slide_completed" />
                    <t t-if="question_index != NoneType"><span class="o_wslides_quiz_question_sequence" t-esc="question_index+1"/>.</t>
                    <t t-else=""><span class="o_wslides_quiz_question_sequence" t-esc="question['sequence']"/>.</t>
                </small>
                <span t-esc="question['question']"/>
            </div>
            <div class="ml-auto o_wslides_js_quiz_edit_del" t-if="slide.channel_id.can_upload and not slide_completed" >
                <i class="o_wslides_js_quiz_icon o_wslides_js_quiz_edit_question fa fa-pencil-square-o p-1 text-muted"></i>
                <i class="o_wslides_js_quiz_icon o_wslides_js_quiz_delete_question fa fa-trash p-1 text-muted"></i>
            </div>
        </div>
        <div class="list-group">
            <t t-foreach="question['answer_ids']" t-as="answer">
                <a t-att-data-answer-id="answer['id']" href="#"
                    t-att-data-text="answer['text_value']" t-att-data-is-correct="answer['is_correct']" t-att-data-comment="answer['comment']"
                    t-att-class="'o_wslides_quiz_answer list-group-item list-group-item-action d-flex align-items-center %s' % ('list-group-item-success' if slide_completed and answer['is_correct'] else '')">
                    <label class="my-0 d-flex align-items-center justify-content-center mr-2">
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
            </t>
            <div class="o_wslides_quiz_answer_info list-group-item list-group-item-info d-none">
                <i class="fa fa-info-circle"/>
                <span class="o_wslides_quiz_answer_comment"/>
            </div>
        </div>
    </div>
</template>

<template id="lesson_content_quiz_add_buttons" name="Lesson: Quiz Add Buttons template">
    <div class="o_wslides_js_lesson_quiz_new_question row mt-3">
        <a t-attf-class="o_wslides_js_quiz_add o_wslides_js_quiz_add_quiz btn btn-light border ml-3 #{'d-none ' if slide.question_ids else ''}" role="button">
            <i class="fa fa-plus mr-2"/>
            <span>Add Quiz</span>
        </a>
        <a t-attf-class="o_wslides_js_quiz_add o_wslides_js_quiz_add_question btn btn-light border ml-3 #{'' if slide.question_ids else 'd-none '}" role="button">
            <i class="fa fa-plus mr-2"/>
            <span>Add Question</span>
        </a>
    </div>
</template>

<!-- Slide sub-template: share: send by email -->
<template id='slide_social_email' name="Share by Email">
    <h5 class="mt-4">Share by mail</h5>
    <div t-if="not is_public_user" class="form-inline">
        <form class="form-group oe_slide_js_share_email" role="form">
            <div class="input-group">
                <input type="email" class="form-control" placeholder="your-friend@domain.com"/>
                <span class="input-group-append">
                    <button class="btn btn-primary" type="button"
                        data-loading-text="Sending..."
                        t-attf-data-slide-id="#{slide.id}"
                        style="border-top-right-radius: 4px;border-bottom-right-radius: 4px;">
                        <i class="fa fa-envelope"/> Send Email
                    </button>
                </span>
            </div>
            <span class="form-text text-muted d-block w-100">Send presentation through email</span>
        </form>
    </div>
    <div t-if="is_public_user" class="alert alert-info d-inline-block">
        <p class="mb-0">Please <a t-attf-href="/web?redirect=#{request.httprequest.url}" class="font-weight-bold"> login </a> to share this <t t-esc="slide.slide_type"/> by email.</p>
    </div>
</template>

<!-- Slide sub-template: share: embed in your website -->
<template id="slide_social_embed" name="Share on Your Website">
    <div class="oe_slide_js_embed_code_widget mt-4">
        <h5 class="mt0">Embed in your website</h5>
        <div class="form-group">
            <textarea class="form-control slide_embed_code" readonly="readonly" onClick="this.select();"><t t-esc="slide.embed_code"/></textarea>
        </div>
        <div class="form-group d-flex" t-if="slide.slide_type in ('presentation', 'document')">
            <div class="form-text p-0 col-xs-5 col-sm-5 col-md-5 col-lg-5">Select page to start with</div>
            <div class="input-group col-xs-3 col-sm-2 col-md-2 col-lg-3">
                <input type="number" value="1" class="form-control"/>
            </div>
        </div>
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
                    <t t-call-assets="web.assets_common" t-js="false"/>
                    <t t-call-assets="website_slides.slide_embed_assets" t-js="false"/>
                    <t t-call-assets="web.assets_common" t-css="false"/>
                    <t t-call-assets="website_slides.slide_embed_assets" t-css="false"/>
                </head>
                <body>
                    <div id="PDFViewer" class="o_wslides_fs_pdf_viewer d-flex flex-column h-100">
                        <!-- PDF Viewer Header : contains the name, and the share links -->
                        <div t-if="is_embedded" class="oe_slides_share_bar">
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
                                        <a href="#" class="oe_slide_js_embed_option_link" data-slide-option-id="#slide_share"><i class="fa fa-share-alt"/> Share</a>
                                        <a href="#" class="oe_slide_js_embed_option_link mx-4" data-slide-option-id="#slide_email"><i class="fa fa-envelope"/> Email</a>
                                        <a href="#" class="oe_slide_js_embed_option_link" data-slide-option-id="#slide_embed"><i class="fa fa-code"/> Embed</a>
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
                            <t t-if="is_embedded">
                                <div id="slide_share" class="oe_slide_embed_option" style="display: none;">
                                    <t t-call="website_slides.slide_share_modal">
                                        <t t-set="record" t-value="slide"/>
                                        <t t-set="website_url" t-value="slide.channel_id.website_url"/>
                                    </t>
                                </div>
                                <div id="slide_email" class="oe_slide_embed_option" style="display: none;">
                                    <t t-call="website_slides.slide_social_email"/>
                                </div>
                                <div id="slide_embed" class="oe_slide_embed_option" style="display: none;">
                                    <t t-call="website_slides.slide_social_embed"/>
                                </div>
                            </t>
                            <div id="slide_suggest" class="oe_slide_embed_option bg-300 container-fluid overflow-auto d-none">
                                <div class="row">
                                    <t t-foreach="related_slides" t-as="suggest_slide">
                                        <div class="col-6 col-md-4 col-lg-3 oe_slides_suggestion_media">
                                            <div class="card mb-3">
                                                <a t-att-href="suggest_slide.website_url" target="_new" class="card-img-top embed-responsive embed-responsive-16by9">
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
                            <t t-if="slide.slide_type in ('presentation', 'document')">
                                <div id="PDFViewerLoader" class="oe_slides_loader mt-3 mx-2 w-100">
                                    <div class="toast show mx-auto">
                                        <div class="toast-header">
                                            <i class="fa fa-circle-o-notch fa-spin mr-2"/><b>Loading...</b>
                                        </div>
                                        <div class="toast-body p-0">
                                            <img class="img-fluid w-100" t-att-src="website.image_url(slide, 'image_256')"/>
                                        </div>
                                    </div>
                                </div>
                                <canvas id="PDFViewerCanvas" class="mx-auto" style="display: none;"></canvas>
                            </t>
                            <t t-if="slide.slide_type == 'infographic'">
                                <img t-att-src="website.image_url(slide, 'image_1024')" class="img-fluid" style="width: 100%" alt="Slide image"/>
                            </t>
                        </div>
                        <!-- Fixed bottom navbar -->
                        <div id="PDFViewerNav" class="pt-2 pb-2 bg-light text-white" role="navigation" t-if="slide.slide_type in ('presentation', 'document')">
                            <div class="container-fluid oe_slides_panel_footer">
                                <div class="row align-items-center">
                                    <div class="col-3 d-flex align-items-center">
                                        <div class="input-group input-group-sm flex-nowrap" style="max-width: 100px">
                                            <input type="number" class="form-control text-center" id="page_number" style="min-width: 60px"/>
                                            <div class="input-group-append">
                                                <span class="input-group-text" id="page_count"/>
                                            </div>
                                        </div>
                                        <a id="zoomout" href="#" class="text-decoration-none d-none d-sm-inline ml-2 mr-2" title="Zoom out" aria-label="Zoom out" role="button">
                                            <i class="fa fa-search-minus" />
                                        </a>
                                        <a id="zoomin" href="#" class="text-decoration-none d-none d-sm-inline" title="Zoom in" aria-label="Zoom in" role="button">
                                            <i class="fa fa-search-plus" />
                                        </a>
                                    </div>
                                    <div class="col text-center">
                                        <a id="first" href="#" onclick="return false;"
                                           class="text-decoration-none mr-1 mr-sm-2" title="First slide"
                                           role="button" aria-label="First slide"> <i class="fa fa-step-backward"/> </a>
                                        <a id="previous" href="#" onclick="return false;"
                                           class="text-decoration-none mx-1 mx-sm-2" title="Previous slide"
                                           aria-label="Previous slide" role="button"> <i class="fa fa-arrow-circle-left"/> </a>
                                        <a id="next" href="#" onclick="return false;"
                                           class="text-decoration-none mx-1 mx-sm-2" title="Next slide"
                                           aria-label="Next slide" role="button"> <i class="fa fa-arrow-circle-right"/> </a>
                                        <a id="last" href="#" onclick="return false;"
                                           class="text-decoration-none mx-1 mx-sm-2" title="Last slide"
                                           aria-label="Last slide" role="button"> <i class="fa fa-step-forward"/> </a>
                                        <a t-if="slide.slide_resource_downloadable" id="download" t-attf-href="/web/content/slide.slide/#{slide.id}/datas?download=true"
                                           class="text-decoration-none ml-1 ml-sm-2" title="Download Content" role="img" aria-label="Download">
                                            <i class="fa fa-download" />
                                        </a>
                                    </div>                                    
                                    <div class="col-3 text-right flex-grow-0">
                                        <a id="fullscreen" href="#" onclick="return false;"
                                           class="text-decoration-none ml-1 ml-sm-2"
                                           title="View fullscreen" role="img" aria-label="Fullscreen">
                                            <i class="fa fa-arrows-alt"/>
                                        </a>
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
        <div class="o_wslides_fs_main d-flex flex-column font-weight-light"
            t-att-data-channel-id="slide.channel_id.id"
            t-att-data-channel-enroll="slide.channel_id.enroll"
            t-att-data-signup-allowed="signup_allowed"
            t-att-data-session-answers="session_answers">

            <div class="o_wslides_slide_fs_header d-flex flex-shrink-0 text-white">
                <div class="d-flex">
                    <a class="o_wslides_fs_toggle_sidebar d-flex align-items-center px-3" href="#" title="Lessons">
                        <i class="fa fa-bars"/><span class="d-none d-md-inline-block ml-1">Lessons</span>
                    </a>
                    <a class="o_wslides_fs_review d-flex align-items-center px-3" t-att-href="slide.channel_id.website_url + '?active_tab=review'" title="Reviews" t-if="slide.channel_id.allow_comment">
                        <i class="fa fa-pencil"/><span class="d-none d-md-inline-block ml-1">Write a review</span>
                    </a>
                    <a class="o_wslides_fs_share d-flex align-items-center px-3" href="#" title="Share">
                        <i class="fa fa-share-alt"/><span class="d-none d-md-inline-block ml-1">Share</span>
                    </a>
                </div>
                <div class="d-flex ml-auto">
                    <a class="d-flex align-items-center px-3 o_wslides_fs_exit_fullscreen" t-attf-href="/slides/slide/#{slug(slide)}">
                        <i class="fa fa-sign-out"/><span class="d-none d-md-inline-block ml-1">Exit Fullscreen</span>
                    </a>
                    <a class="d-flex align-items-center px-3" t-attf-href="/slides/#{slug(slide.channel_id)}">
                        <i class="fa fa-home"/><span class="d-none d-md-inline-block ml-1">Back to course</span>
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
                            <div t-if="not is_public_user" class="d-flex align-items-center">
                                <t t-if="slide.channel_id.completed">
                                    <span class="badge badge-pill badge-success py-1 px-2" style="font-size: 1em"><i class="fa fa-check"/> Completed</span>
                                </t>
                                <t t-else="">
                                    <div class="progress flex-grow-1 bg-black-50" style="height: 6px;">
                                        <div class="progress-bar" role="progressbar" t-attf-style="width: #{slide.channel_id.completion}%" t-att-aria-valuenow="slide.channel_id.completion" aria-valuemin="0" aria-valuemax="100"></div>
                                    </div>
                                    <div class="ml-3 small">
                                        <span class="o_wslides_progress_percentage" t-esc="slide.channel_id.completion"/> %
                                    </div>
                                </t>
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
                    <a href="#" class="o_wslides_fs_toggle_sidebar d-lg-none bg-black-50"/>
                </div>
            </div>
        </div>
    </t>
</template>


<template id="slide_fullscreen_sidebar_category" name="Slides category template for fullscreen view side bar">
    <t t-if="category" t-set="category" t-value="category.get('category')"/>
    <li class="o_wslides_fs_sidebar_section py-2 px-3">
        <a t-if="category" class="text-uppercase text-500 py-1 small d-block" t-attf-id="category-collapse-#{category.id if category else 0}" data-toggle="collapse" role="button" aria-expanded="true" t-att-href="('#collapse-%s') % (category.id if category else 0)" t-attf-aria-controls="collapse-#{category.id if category else 0}">
            <b t-field="category.name"/>
        </a>
        <ul class="o_wslides_fs_sidebar_section_slides collapse show position-relative px-0 pb-1 my-0 mx-n3" t-att-id="('collapse-%s') % (category.id if category else 0)">
            <t t-set="is_member" t-value="current_slide.channel_id.is_member"/>
            <t t-set="can_access_channel" t-value="is_member or current_slide.channel_id.can_publish"/>
            <t t-foreach="slides" t-as="slide">
                <t t-set="slide_completed" t-value="channel_progress[slide.id].get('completed')"/>
                <t t-set="can_access" t-value="can_access_channel or slide.is_preview"/>
                <li t-att-class="'o_wslides_fs_sidebar_list_item d-flex align-items-top py-1 %s' % ('active' if slide.id == current_slide.id else '')"
                    t-att-data-id="slide.id"
                    t-att-data-can-access="can_access"
                    t-att-data-name="slide.name"
                    t-att-data-type="slide.slide_type"
                    t-att-data-slug="slug(slide)"
                    t-att-data-has-question="1 if slide.question_ids else 0"
                    t-att-data-is-quiz="0"
                    t-att-data-completed="1 if slide_completed else 0"
                    t-att-data-embed-code="slide.embed_code if slide.slide_type in ['video', 'document', 'presentation', 'infographic'] else False"
                    t-att-data-is-member="is_member"
                    t-att-data-session-answers="session_answers">
                    <span class="ml-3">
                        <i t-if="slide_completed and is_member" class="o_wslides_slide_completed fa fa-check fa-fw text-success" t-att-data-slide-id="slide.id"/>
                        <i t-if="not slide_completed and is_member" class="fa fa-circle-thin fa-fw" t-att-data-slide-id="slide.id"/>
                    </span>
                    <div class="ml-2 overflow-hidden">
                        <a t-if="can_access" class="d-block pt-1" href="#">
                            <div class="d-flex ">
                                <t t-call="website_slides.slide_icon"/>
                                <div class="o_wslides_fs_slide_name text-truncate" t-esc="slide.name"/>
                            </div>
                        </a>
                        <span t-else="" class="d-block pt-1" href="#">
                            <div class="d-flex ">
                                <t t-set="icon_class" t-value="'mr-2 text-600'"/>
                                <t t-call="website_slides.slide_icon"/>
                                <div class="o_wslides_fs_slide_name text-600 text-truncate" t-esc="slide.name"/>
                            </div>
                        </span>
                        <ul class="list-unstyled w-100 pt-2 small" t-if="slide.link_ids or slide._has_additional_resources() or (slide.question_ids and not slide.slide_type =='quiz')" >
                            <li t-if="slide.link_ids" t-foreach="slide.link_ids" t-as="link" class="pl-0 mb-1">
                                <a t-if="can_access" class="o_wslides_fs_slide_link" t-att-href="link.link" target="_blank">
                                    <i class="fa fa-link mr-2"/><span t-esc="link.name"/>
                                </a>
                                <span t-else="" class="o_wslides_fs_slide_link text-600">
                                    <i class="fa fa-link mr-2"/><span t-esc="link.name"/>
                                </span>
                            </li>
                            <div class="o_wslides_js_course_join pl-0" t-if="slide._has_additional_resources()">
                                <t t-if="can_access_channel">
                                    <li t-foreach="slide.slide_resource_ids" t-as="resource" class="mb-1">
                                        <a class="o_wslides_fs_slide_link" t-attf-href="/web/content/slide.slide.resource/#{resource.id}/data?download=true">
                                            <i class="fa fa-download mr-2"/><span t-esc="resource.name"/>
                                        </a>
                                    </li>
                                </t>
                                <li t-elif="slide.channel_id.enroll == 'public'" class="o_wslides_fs_slide_link mb-1">
                                    <i class="fa fa-download mr-1"/>
                                    <t t-call="website_slides.join_course_link"/>
                                </li>
                            </div>
                            <li class="o_wslides_fs_sidebar_list_item pl-0 mb-1" t-if="slide.question_ids and not slide.slide_type == 'quiz'"
                                t-att-data-id="slide.id"
                                t-att-data-can-access="can_access"
                                t-att-data-name="slide.name"
                                t-att-data-type="slide.slide_type"
                                t-att-data-slug="slug(slide)"
                                t-att-data-has-question="1 if slide.question_ids else 0"
                                t-att-data-is-quiz="1"
                                t-att-data-completed="1 if slide_completed else 0"
                                t-att-data-is-member="is_member"
                                t-att-data-session-answers="session_answers">
                                <a t-if="can_access" class="o_wslides_fs_slide_quiz" href="#" t-att-index="i">
                                    <i class="fa fa-flag-checkered text-warning mr-2"/>Quiz
                                </a>
                                <span t-else="" class="text-600">
                                    <i class="fa fa-flag-checkered text-warning mr-2"/>Quiz
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
    <!--Private profile-->
    <template id="private_profile" inherit_id="website_profile.private_profile">
        <xpath expr="//div[@id='private_profile_return_link_container']" position="inside">
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
                    <div t-else="" class="text-muted d-inline-block">No completed courses yet!</div>
                    <div class="text-right d-inline-block pull-right">
                        <a href="/slides/all" class="btn btn-link btn-sm"><i class="fa fa-arrow-right mr-1"/>All Courses</a>
                    </div>
                </div>
                <div class="mb32">
                    <h5 class="border-bottom pb-1">Followed Courses</h5>
                    <t t-if="courses_ongoing" t-call="website_slides.display_course">
                        <t t-set="courses" t-value="courses_ongoing"></t>
                    </t>
                    <p t-else="" class="text-muted">No followed courses yet!</p>
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

                        <div t-if="course.channel_id.image_1024" class="pl-5 pr-4 rounded-left" t-attf-style="background-image: url(#{website.image_url(course.channel_id, 'image_1024')}); background-size: cover; background-position: center"/>
                        <div t-else="" class="o_wslides_gradient pl-5 pr-4 rounded-left position-relative" style="opacity: 0.8">
                            <i class="fa fa-graduation-cap fa-fw mr-2 mt-3 position-absolute text-white-75" style="right:0; top: 0"/>
                        </div>

                        <div class="p-2 w-100">
                            <h5 class="mt-0 mb-1" t-field="course.channel_id.name"/>

                            <div class="overflow-hidden mb-1" style="height:24px">
                                <t t-foreach="course.channel_id.tag_ids" t-as="tag">
                                    <a t-att-href="'/slides/all?channel_tag_id=%s' % tag.id" t-attf-class="badge o_wslides_channel_tag #{'o_tag_color_'+str(tag.color)}" t-esc="tag.name"/>
                                </t>
                            </div>

                            <div class="d-flex align-items-center">
                                <div class="progress flex-grow-1" style="height:0.5em">
                                    <div class="progress-bar bg-primary" t-att-style="'width: '+ str(course.completion)+'%'"/>
                                </div>
                                <small class="font-weight-bold pl-2"><span t-esc="course.completion"/> %</small>
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
    <div class="btn-group" role="group">
        <a t-attf-href="https://www.facebook.com/sharer/sharer.php?u=#{record.website_url}" class="btn border bg-white o_wslides_js_social_share" social-key="facebook" aria-label="Share on Facebook" title="Share on Facebook"><i class="fa fa-facebook-square fa-fw"/></a>
        <a t-attf-href="https://twitter.com/intent/tweet?text=#{record.name}&amp;url=#{record.website_url}" class="btn border bg-white o_wslides_js_social_share"  social-key="twitter" aria-label="Share on Twitter" title="Share on Twitter"><i class="fa fa-twitter fa-fw"/></a>
        <a t-attf-href="http://www.linkedin.com/sharing/share-offsite/?url=#{record.website_url}" social-key="linkedin" class="btn border bg-white o_wslides_js_social_share" aria-label="Share on LinkedIn" title="Share on LinkedIn"><i class="fa fa-linkedin fa-fw"/></a>
    </div>
</template>

<!-- Share: social media -->
<template id='slide_share_modal'>
    <t t-if="not website_url">
        <t t-set="website_url" t-value="record.website_url"/>
    </t>
    <div class="modal fade" t-att-id="'slideChannelShareModal_%s' % record.id" tabindex="-1" role="dialog" aria-labelledby="slideChannelShareModalLabel" aria-hidden="true">
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
        <h5 class="modal-title" id="slideChannelShareModalLabel">
            Share
        </h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
            <span>×</span>
        </button>
    </div>
</template>

<template id="slide_share_modal_body">
    <div class="modal-body">
        <h5>Share on Social Networks</h5>
        <t t-call="website_slides.slide_share_social"/>
        <t t-call="website_slides.slide_share_link"/>
    </div>
</template>

<template id="slide_share_link">
    <h5 class="mt16">Share Link</h5>
    <div class="input-group">
        <input type="text" t-att-id="'wslides_share_link_id_%s' % record.id" class="form-control o_wslides_js_share_link" readonly="readonly" onclick="this.select();"
            t-att-value="website_url"/>
        <div class="input-group-append">
            <button t-att-id="'share_link_clipboard_button_id_%s' % record.id" class="btn btn-sm btn-primary o_clipboard_button" >
                <span class="fa fa-clipboard"> Copy Text</span>
            </button>
        </div>
    </div>
</template>

<!-- Website edit page -->
<template id="slide_edit_options" inherit_id="website.user_navbar" name="Edit Slide Options">
    <xpath expr="//li[@id='edit-page-menu']" position="after">
        <t t-if="main_object._name == 'slide.slide'" t-set="action" t-value="'website_slides.slide_slide_action'"/>
    </xpath>
</template>

<!-- User Navbar -->
<template id="user_navbar_inherit_website_slides" inherit_id="website.user_navbar">
    <xpath expr="//div[@id='o_new_content_menu_choices']//div[@name='module_website_slides']" position="attributes">
        <attribute name="name"/>
        <attribute name="t-att-data-module-id"/>
        <attribute name="t-att-data-module-shortdesc"/>
        <attribute name="groups">website_slides.group_website_slides_officer</attribute>
    </xpath>
</template>

<template id="join_course_link" name="Join Course Link">
    <a class="o_wslides_js_course_join_link" href="#" t-att-data-channel-enroll="slide.channel_id.enroll"
       t-att-data-channel-id="slide.channel_id.id">
        Join Course</a> to download resources
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
from odoo.exceptions import UserError, AccessError
from odoo.tools import formataddr

_logger = logging.getLogger(__name__)

emails_split = re.compile(r"[;,\n\r]+")


class SlideChannelInvite(models.TransientModel):
    _name = 'slide.channel.invite'
    _description = 'Channel Invitation Wizard'

    # composer content
    subject = fields.Char('Subject', compute='_compute_subject', readonly=False, store=True)
    body = fields.Html('Contents', sanitize_style=True, compute='_compute_body', readonly=False, store=True)
    attachment_ids = fields.Many2many('ir.attachment', string='Attachments')
    template_id = fields.Many2one(
        'mail.template', 'Use template',
        domain="[('model', '=', 'slide.channel.partner')]")
    # recipients
    partner_ids = fields.Many2many('res.partner', string='Recipients')
    # slide channel
    channel_id = fields.Many2one('slide.channel', string='Slide channel', required=True)

    @api.depends('template_id')
    def _compute_subject(self):
        for invite in self:
            if invite.template_id:
                invite.subject = invite.template_id.subject
            elif not invite.subject:
                invite.subject = False

    @api.depends('template_id')
    def _compute_body(self):
        for invite in self:
            if invite.template_id:
                invite.body = invite.template_id.body_html
            elif not invite.body:
                invite.body = False

    @api.onchange('partner_ids')
    def _onchange_partner_ids(self):
        if self.partner_ids:
            signup_allowed = self.env['res.users'].sudo()._get_signup_invitation_scope() == 'b2c'
            if not signup_allowed:
                invalid_partners = self.env['res.partner'].search([
                    ('user_ids', '=', False),
                    ('id', 'in', self.partner_ids.ids)
                ])
                if invalid_partners:
                    raise UserError(_(
                        'The following recipients have no user account: %s. You should create user accounts for them or allow external sign up in configuration.',
                        ', '.join(invalid_partners.mapped('name'))
                    ))

    @api.model
    def create(self, values):
        if values.get('template_id') and not (values.get('body') or values.get('subject')):
            template = self.env['mail.template'].browse(values['template_id'])
            if not values.get('subject'):
                values['subject'] = template.subject
            if not values.get('body'):
                values['body'] = template.body_html
        return super(SlideChannelInvite, self).create(values)

    def action_invite(self):
        """ Process the wizard content and proceed with sending the related
            email(s), rendering any template patterns on the fly if needed """
        self.ensure_one()

        if not self.env.user.email:
            raise UserError(_("Unable to post message, please configure the sender's email address."))

        try:
            self.channel_id.check_access_rights('write')
            self.channel_id.check_access_rule('write')
        except AccessError:
            raise AccessError(_('You are not allowed to add members to this course. Please contact the course responsible or an administrator.'))

        mail_values = []
        for partner_id in self.partner_ids:
            slide_channel_partner = self.channel_id._action_add_members(partner_id)
            if slide_channel_partner:
                mail_values.append(self._prepare_mail_values(slide_channel_partner))

        # TODO awa: change me to create multi when mail.mail supports it
        for mail_value in mail_values:
            self.env['mail.mail'].sudo().create(mail_value)

        return {'type': 'ir.actions.act_window_close'}

    def _prepare_mail_values(self, slide_channel_partner):
        """ Create mail specific for recipient """
        subject = self.env['mail.render.mixin']._render_template(self.subject, 'slide.channel.partner', slide_channel_partner.ids, post_process=True)[slide_channel_partner.id]
        body = self.env['mail.render.mixin']._render_template(self.body, 'slide.channel.partner', slide_channel_partner.ids, post_process=True)[slide_channel_partner.id]
        # post the message
        mail_values = {
            'email_from': self.env.user.email_formatted,
            'author_id': self.env.user.partner_id.id,
            'model': None,
            'res_id': None,
            'subject': subject,
            'body_html': body,
            'attachment_ids': [(4, att.id) for att in self.attachment_ids],
            'auto_delete': self.template_id.auto_delete if self.template_id else True,
            'recipient_ids': [(4, slide_channel_partner.partner_id.id)]
        }

        # optional support of notif_layout in context
        notif_layout = self.env.context.get('notif_layout', self.env.context.get('custom_layout'))
        if notif_layout:
            try:
                template = self.env.ref(notif_layout, raise_if_not_found=True)
            except ValueError:
                _logger.warning('QWeb template %s not found when sending slide channel mails. Sending without layouting.' % (notif_layout))
            else:
                # could be great to use _notify_prepare_template_context someday
                template_ctx = {
                    'message': self.env['mail.message'].sudo().new(dict(body=mail_values['body_html'], record_name=self.channel_id.name)),
                    'model_description': self.env['ir.model']._get('slide.channel').display_name,
                    'record': slide_channel_partner,
                    'company': self.env.company,
                    'signature': self.channel_id.user_id.signature,
                }
                body = template._render(template_ctx, engine='ir.qweb', minimal_qcontext=True)
                mail_values['body_html'] = self.env['mail.render.mixin']._replace_local_links(body)

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
                <form string="Compose Email">
                    <group col="1">
                        <group col="2">
                            <field name="partner_ids"
                                widget="many2many_tags_email"
                                placeholder="Add existing contacts..."
                                context="{'force_email':True, 'show_email':True, 'no_create_edit': True}"/>
                        </group>
                        <group col="2">
                            <field name="subject" placeholder="Subject..."/>
                        </group>
                        <field name="body" options="{'style-inline': true}"/>
                        <group>
                            <group>
                                <field name="attachment_ids" widget="many2many_binary"/>
                            </group>
                            <group>
                                <field name="template_id" label="Use template" context="{'default_model': 'slide.channel.partner'}"/>
                            </group>
                        </group>
                    </group>
                    <footer>
                        <button string="Send" name="action_invite" type="object" class="btn-primary"/>
                        <button string="Cancel" class="btn-secondary" special="cancel"/>
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

