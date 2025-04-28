# Odoo Module: website_forum

Category: Website/Website

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Forum',
    'category': 'Website/Website',
    'sequence': 265,
    'summary': 'Manage a forum with FAQ and Q&A',
    'version': '1.0',
    'description': """
Ask questions, get answers, no distractions
        """,
    'website': 'https://www.odoo.com/page/community-builder',
    'depends': [
        'auth_signup',
        'website_mail',
        'website_profile',
    ],
    'data': [
        'data/forum_default_faq.xml',
        'data/forum_data.xml',
        'views/forum.xml',
        'views/res_users_views.xml',
        'views/website_forum.xml',
        'views/website_forum_profile.xml',
        'views/ir_qweb.xml',
        'security/ir.model.access.csv',
        'security/website_forum_security.xml',
        'data/badges_question.xml',
        'data/badges_answer.xml',
        'data/badges_participation.xml',
        'data/badges_moderation.xml',
    ],
    'qweb': [
        'static/src/xml/*.xml'
    ],
    'demo': [
        'data/forum_demo.xml',
    ],
    'installable': True,
    'application': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import json
import lxml
import requests
import logging
import werkzeug.exceptions
import werkzeug.urls
import werkzeug.wrappers

from datetime import datetime

from odoo import http, tools, _
from odoo.addons.http_routing.models.ir_http import slug
from odoo.addons.website.models.ir_http import sitemap_qs2dom
from odoo.addons.website_profile.controllers.main import WebsiteProfile
from odoo.addons.portal.controllers.portal import _build_url_w_params

from odoo.exceptions import UserError
from odoo.http import request
from odoo.osv import expression


_logger = logging.getLogger(__name__)


class WebsiteForum(WebsiteProfile):
    _post_per_page = 10
    _user_per_page = 30

    def _prepare_user_values(self, **kwargs):
        values = super(WebsiteForum, self)._prepare_user_values(**kwargs)
        values['forum_welcome_message'] = request.httprequest.cookies.get('forum_welcome_message', False)
        values.update({
            'header': kwargs.get('header', dict()),
            'searches': kwargs.get('searches', dict()),
        })
        if kwargs.get('forum'):
            values['forum'] = kwargs.get('forum')
        elif kwargs.get('forum_id'):
            values['forum'] = request.env['forum.forum'].browse(kwargs.pop('forum_id'))
        return values

    # Forum
    # --------------------------------------------------

    @http.route(['/forum'], type='http', auth="public", website=True, sitemap=True)
    def forum(self, **kwargs):
        domain = request.website.website_domain()
        forums = request.env['forum.forum'].search(domain)
        if len(forums) == 1:
            return werkzeug.utils.redirect('/forum/%s' % slug(forums[0]), code=302)

        return request.render("website_forum.forum_all", {
            'forums': forums
        })

    @http.route('/forum/new', type='json', auth="user", methods=['POST'], website=True)
    def forum_create(self, forum_name="New Forum", forum_mode="questions", forum_privacy="public", forum_privacy_group=False, add_menu=False):
        forum = {
            'name': forum_name,
            'mode': forum_mode,
            'privacy': forum_privacy,
            'website_id': request.website.id,
        }
        if forum_privacy == 'private' and forum_privacy_group:
            forum['authorized_group_id'] = forum_privacy_group
        forum_id = request.env['forum.forum'].create(forum)
        if add_menu:
            group = [int(forum_privacy_group)] if forum_privacy == 'private' else [request.env.ref('base.group_portal').id, request.env.ref('base.group_user').id]
            menu_id = request.env['website.menu'].create({
                'name': forum_name,
                'url': "/forum/%s" % slug(forum_id),
                'parent_id': request.website.menu_id.id,
                'website_id': request.website.id,
                'group_ids': [(6, 0, group)]
            })
            forum_id.menu_id = menu_id
        return "/forum/%s" % slug(forum_id)

    def sitemap_forum(env, rule, qs):
        Forum = env['forum.forum']
        dom = sitemap_qs2dom(qs, '/forum', Forum._rec_name)
        dom += env['website'].get_current_website().website_domain()
        for f in Forum.search(dom):
            loc = '/forum/%s' % slug(f)
            if not qs or qs.lower() in loc:
                yield {'loc': loc}

    @http.route(['/forum/<model("forum.forum"):forum>',
                 '/forum/<model("forum.forum"):forum>/page/<int:page>',
                 '''/forum/<model("forum.forum"):forum>/tag/<model("forum.tag"):tag>/questions''',
                 '''/forum/<model("forum.forum"):forum>/tag/<model("forum.tag"):tag>/questions/page/<int:page>''',
                 ], type='http', auth="public", website=True, sitemap=sitemap_forum)
    def questions(self, forum, tag=None, page=1, filters='all', my=None, sorting=None, search='', **post):
        if not forum.can_access_from_current_website():
            raise werkzeug.exceptions.NotFound()

        Post = request.env['forum.post']

        domain = [('forum_id', '=', forum.id), ('parent_id', '=', False), ('state', '=', 'active'), ('can_view', '=', True)]
        if search:
            domain += ['|', ('name', 'ilike', search), ('content', 'ilike', search)]
        if tag:
            domain += [('tag_ids', 'in', tag.id)]
        if filters == 'unanswered':
            domain += [('child_ids', '=', False)]
        elif filters == 'solved':
            domain += [('has_validated_answer', '=', True)]
        elif filters == 'unsolved':
            domain += [('has_validated_answer', '=', False)]

        user = request.env.user

        if my == 'mine':
            domain += [('create_uid', '=', user.id)]
        elif my == 'followed':
            domain += [('message_partner_ids', '=', user.partner_id.id)]
        elif my == 'tagged':
            domain += [('tag_ids.message_partner_ids', '=', user.partner_id.id)]
        elif my == 'favourites':
            domain += [('favourite_ids', '=', user.id)]

        if sorting:
            # check that sorting is valid
            # retro-compatibily for V8 and google links
            try:
                sorting = werkzeug.urls.url_unquote_plus(sorting)
                Post._generate_order_by(sorting, None)
            except (UserError, ValueError):
                sorting = False

        if not sorting:
            sorting = forum.default_order

        question_count = Post.search_count(domain)

        if tag:
            url = "/forum/%s/tag/%s/questions" % (slug(forum), slug(tag))
        else:
            url = "/forum/%s" % slug(forum)

        url_args = {
            'sorting': sorting
        }
        if search:
            url_args['search'] = search
        if filters:
            url_args['filters'] = filters
        if my:
            url_args['my'] = my
        pager = request.website.pager(url=url, total=question_count, page=page,
                                      step=self._post_per_page, scope=self._post_per_page,
                                      url_args=url_args)

        question_ids = Post.search(domain, limit=self._post_per_page, offset=pager['offset'], order=sorting)

        values = self._prepare_user_values(forum=forum, searches=post, header={'ask_hide': not forum.active})
        values.update({
            'main_object': tag or forum,
            'edit_in_backend': not tag,
            'question_ids': question_ids,
            'question_count': question_count,
            'pager': pager,
            'tag': tag,
            'filters': filters,
            'my': my,
            'sorting': sorting,
            'search': search,
        })
        return request.render("website_forum.forum_index", values)

    @http.route(['''/forum/<model("forum.forum"):forum>/faq'''], type='http', auth="public", website=True, sitemap=True)
    def forum_faq(self, forum, **post):
        values = self._prepare_user_values(forum=forum, searches=dict(), header={'is_guidelines': True}, **post)
        return request.render("website_forum.faq", values)

    @http.route(['/forum/<model("forum.forum"):forum>/faq/karma'], type='http', auth="public", website=True, sitemap=False)
    def forum_faq_karma(self, forum, **post):
        values = self._prepare_user_values(forum=forum, header={'is_guidelines': True, 'is_karma': True}, **post)
        return request.render("website_forum.faq_karma", values)

    @http.route('/forum/get_tags', type='http', auth="public", methods=['GET'], website=True, sitemap=False)
    def tag_read(self, query='', limit=25, **post):
        # TODO: In master always check the forum_id domain part and add forum_id
        #       as required method param, not in **post
        forum_id = post.get('forum_id')
        domain = [('name', '=ilike', (query or '') + "%")]
        if forum_id:
            domain = expression.AND([domain, [('forum_id', '=', int(forum_id))]])
        data = request.env['forum.tag'].search_read(
            domain=domain,
            fields=['id', 'name'],
            limit=int(limit),
        )
        return request.make_response(
            json.dumps(data),
            headers=[("Content-Type", "application/json")]
        )

    @http.route(['/forum/<model("forum.forum"):forum>/tag', '/forum/<model("forum.forum"):forum>/tag/<string:tag_char>'], type='http', auth="public", website=True, sitemap=False)
    def tags(self, forum, tag_char=None, **post):
        # build the list of tag first char, with their value as tag_char param Ex : [('All', 'all'), ('C', 'c'), ('G', 'g'), ('Z', z)]
        first_char_tag = forum.get_tags_first_char()
        first_char_list = [(t, t.lower()) for t in first_char_tag if t.isalnum()]
        first_char_list.insert(0, (_('All'), 'all'))

        active_char_tag = tag_char and tag_char.lower() or 'all'

        # generate domain for searched tags
        domain = [('forum_id', '=', forum.id), ('posts_count', '>', 0)]
        order_by = 'name'
        if active_char_tag and active_char_tag != 'all':
            domain.append(('name', '=ilike', tools.escape_psql(active_char_tag) + '%'))
            order_by = 'posts_count DESC'
        tags = request.env['forum.tag'].search(domain, limit=None, order=order_by)
        # prepare values and render template
        values = self._prepare_user_values(forum=forum, searches={'tags': True}, **post)

        values.update({
            'tags': tags,
            'pager_tag_chars': first_char_list,
            'active_char_tag': active_char_tag,
        })
        return request.render("website_forum.tag", values)

    # Questions
    # --------------------------------------------------

    @http.route('/forum/get_url_title', type='json', auth="user", methods=['POST'], website=True)
    def get_url_title(self, **kwargs):
        try:
            req = requests.get(kwargs.get('url'))
            req.raise_for_status()
            arch = lxml.html.fromstring(req.content)
            return arch.find(".//title").text
        except IOError:
            return False

    @http.route(['''/forum/<model("forum.forum"):forum>/question/<model("forum.post", "[('forum_id','=',forum.id),('parent_id','=',False),('can_view', '=', True)]"):question>'''],
                type='http', auth="public", website=True, sitemap=False)
    def old_question(self, forum, question, **post):
        # Compatibility pre-v14
        return request.redirect(_build_url_w_params("/forum/%s/%s" % (slug(forum), slug(question)), request.params), code=301)

    @http.route(['''/forum/<model("forum.forum"):forum>/<model("forum.post", "[('forum_id','=',forum.id),('parent_id','=',False),('can_view', '=', True)]"):question>'''],
                type='http', auth="public", website=True, sitemap=True)
    def question(self, forum, question, **post):
        if not forum.active:
            return request.render("website_forum.header", {'forum': forum})

        # Hide posts from abusers (negative karma), except for moderators
        if not question.can_view:
            raise werkzeug.exceptions.NotFound()

        # Hide pending posts from non-moderators and non-creator
        user = request.env.user
        if question.state == 'pending' and user.karma < forum.karma_post and question.create_uid != user:
            raise werkzeug.exceptions.NotFound()

        if question.parent_id:
            redirect_url = "/forum/%s/%s" % (slug(forum), slug(question.parent_id))
            return werkzeug.utils.redirect(redirect_url, 301)
        filters = 'question'
        values = self._prepare_user_values(forum=forum, searches=post)
        values.update({
            'main_object': question,
            'question': question,
            'can_bump': (question.forum_id.allow_bump and not question.child_count and (datetime.today() - question.write_date).days > 9),
            'header': {'question_data': True},
            'filters': filters,
            'reversed': reversed,
        })
        if (request.httprequest.referrer or "").startswith(request.httprequest.url_root):
            values['back_button_url'] = request.httprequest.referrer

        # increment view counter
        question.sudo()._set_viewed()

        return request.render("website_forum.post_description_full", values)

    @http.route('/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/toggle_favourite', type='json', auth="user", methods=['POST'], website=True)
    def question_toggle_favorite(self, forum, question, **post):
        if not request.session.uid:
            return {'error': 'anonymous_user'}
        favourite = not question.user_favourite
        question.sudo().favourite_ids = [(favourite and 4 or 3, request.uid)]
        if favourite:
            # Automatically add the user as follower of the posts that he
            # favorites (on unfavorite we chose to keep him as a follower until
            # he decides to not follow anymore).
            question.sudo().message_subscribe(request.env.user.partner_id.ids)
        return favourite

    @http.route('/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/ask_for_close', type='http', auth="user", methods=['POST'], website=True)
    def question_ask_for_close(self, forum, question, **post):
        reasons = request.env['forum.post.reason'].search([('reason_type', '=', 'basic')])

        values = self._prepare_user_values(**post)
        values.update({
            'question': question,
            'forum': forum,
            'reasons': reasons,
        })
        return request.render("website_forum.close_post", values)

    @http.route('/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/edit_answer', type='http', auth="user", website=True)
    def question_edit_answer(self, forum, question, **kwargs):
        for record in question.child_ids:
            if record.create_uid.id == request.uid:
                answer = record
                break
        return werkzeug.utils.redirect("/forum/%s/post/%s/edit" % (slug(forum), slug(answer)))

    @http.route('/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/close', type='http', auth="user", methods=['POST'], website=True)
    def question_close(self, forum, question, **post):
        question.close(reason_id=int(post.get('reason_id', False)))
        return werkzeug.utils.redirect("/forum/%s/question/%s" % (slug(forum), slug(question)))

    @http.route('/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/reopen', type='http', auth="user", methods=['POST'], website=True)
    def question_reopen(self, forum, question, **kwarg):
        question.reopen()
        return werkzeug.utils.redirect("/forum/%s/%s" % (slug(forum), slug(question)))

    @http.route('/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/delete', type='http', auth="user", methods=['POST'], website=True)
    def question_delete(self, forum, question, **kwarg):
        question.active = False
        return werkzeug.utils.redirect("/forum/%s/%s" % (slug(forum), slug(question)))

    @http.route('/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/undelete', type='http', auth="user", methods=['POST'], website=True)
    def question_undelete(self, forum, question, **kwarg):
        question.active = True
        return werkzeug.utils.redirect("/forum/%s/%s" % (slug(forum), slug(question)))

    # Post
    # --------------------------------------------------
    @http.route(['/forum/<model("forum.forum"):forum>/ask'], type='http', auth="user", website=True)
    def forum_post(self, forum, **post):
        user = request.env.user
        if not user.email or not tools.single_email_re.match(user.email):
            return werkzeug.utils.redirect("/forum/%s/user/%s/edit?email_required=1" % (slug(forum), request.session.uid))
        values = self._prepare_user_values(forum=forum, searches={}, header={'ask_hide': True}, new_question=True)
        return request.render("website_forum.new_question", values)

    @http.route(['/forum/<model("forum.forum"):forum>/new',
                 '/forum/<model("forum.forum"):forum>/<model("forum.post"):post_parent>/reply'],
                type='http', auth="user", methods=['POST'], website=True)
    def post_create(self, forum, post_parent=None, **post):
        if post.get('content', '') == '<p><br></p>':
            return request.render('http_routing.http_error', {
                'status_code': _('Bad Request'),
                'status_message': post_parent and _('Reply should not be empty.') or _('Question should not be empty.')
            })

        post_tag_ids = forum._tag_to_write_vals(post.get('post_tags', ''))

        if request.env.user.forum_waiting_posts_count:
            return werkzeug.utils.redirect("/forum/%s/ask" % slug(forum))

        new_question = request.env['forum.post'].create({
            'forum_id': forum.id,
            'name': post.get('post_name') or (post_parent and 'Re: %s' % (post_parent.name or '')) or '',
            'content': post.get('content', False),
            'parent_id': post_parent and post_parent.id or False,
            'tag_ids': post_tag_ids
        })
        return werkzeug.utils.redirect("/forum/%s/%s" % (slug(forum), post_parent and slug(post_parent) or new_question.id))

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/comment', type='http', auth="user", methods=['POST'], website=True)
    def post_comment(self, forum, post, **kwargs):
        question = post.parent_id if post.parent_id else post
        if kwargs.get('comment') and post.forum_id.id == forum.id:
            # TDE FIXME: check that post_id is the question or one of its answers
            body = tools.mail.plaintext2html(kwargs['comment'])
            post.with_context(mail_create_nosubscribe=True).message_post(
                body=body,
                message_type='comment',
                subtype_xmlid='mail.mt_comment')
        return werkzeug.utils.redirect("/forum/%s/%s" % (slug(forum), slug(question)))

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/toggle_correct', type='json', auth="public", website=True)
    def post_toggle_correct(self, forum, post, **kwargs):
        if post.parent_id is False:
            return request.redirect('/')
        if not request.session.uid:
            return {'error': 'anonymous_user'}

        # set all answers to False, only one can be accepted
        (post.parent_id.child_ids - post).write(dict(is_correct=False))
        post.is_correct = not post.is_correct
        return post.is_correct

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/delete', type='http', auth="user", methods=['POST'], website=True)
    def post_delete(self, forum, post, **kwargs):
        question = post.parent_id
        post.unlink()
        if question:
            werkzeug.utils.redirect("/forum/%s/%s" % (slug(forum), slug(question)))
        return werkzeug.utils.redirect("/forum/%s" % slug(forum))

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/edit', type='http', auth="user", website=True)
    def post_edit(self, forum, post, **kwargs):
        tags = [dict(id=tag.id, name=tag.name) for tag in post.tag_ids]
        tags = json.dumps(tags)
        values = self._prepare_user_values(forum=forum)
        values.update({
            'tags': tags,
            'post': post,
            'is_edit': True,
            'is_answer': bool(post.parent_id),
            'searches': kwargs,
            'content': post.name,
        })
        return request.render("website_forum.edit_post", values)

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/save', type='http', auth="user", methods=['POST'], website=True)
    def post_save(self, forum, post, **kwargs):
        vals = {
            'content': kwargs.get('content'),
        }

        if 'post_name' in kwargs:
            if not kwargs.get('post_name').strip():
                return request.render('http_routing.http_error', {
                    'status_code': _('Bad Request'),
                    'status_message': _('Title should not be empty.')
                })

            vals['name'] = kwargs.get('post_name')
        vals['tag_ids'] = forum._tag_to_write_vals(kwargs.get('post_tags', ''))
        post.write(vals)
        question = post.parent_id if post.parent_id else post
        return werkzeug.utils.redirect("/forum/%s/%s" % (slug(forum), slug(question)))

    #  JSON utilities
    # --------------------------------------------------

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/upvote', type='json', auth="public", website=True)
    def post_upvote(self, forum, post, **kwargs):
        if not request.session.uid:
            return {'error': 'anonymous_user'}
        if request.uid == post.create_uid.id:
            return {'error': 'own_post'}
        upvote = True if not post.user_vote > 0 else False
        return post.vote(upvote=upvote)

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/downvote', type='json', auth="public", website=True)
    def post_downvote(self, forum, post, **kwargs):
        if not request.session.uid:
            return {'error': 'anonymous_user'}
        if request.uid == post.create_uid.id:
            return {'error': 'own_post'}
        upvote = True if post.user_vote < 0 else False
        return post.vote(upvote=upvote)

    @http.route('/forum/post/bump', type='json', auth="public", website=True)
    def post_bump(self, post_id, **kwarg):
        post = request.env['forum.post'].browse(int(post_id))
        if not post.exists() or post.parent_id:
            return False
        return post.bump()

    # Moderation Tools
    # --------------------------------------------------

    @http.route('/forum/<model("forum.forum"):forum>/validation_queue', type='http', auth="user", website=True)
    def validation_queue(self, forum, **kwargs):
        user = request.env.user
        if user.karma < forum.karma_moderate:
            raise werkzeug.exceptions.NotFound()

        Post = request.env['forum.post']
        domain = [('forum_id', '=', forum.id), ('state', '=', 'pending')]
        posts_to_validate_ids = Post.search(domain)

        values = self._prepare_user_values(forum=forum)
        values.update({
            'posts_ids': posts_to_validate_ids.sudo(),
            'queue_type': 'validation',
        })

        return request.render("website_forum.moderation_queue", values)

    @http.route('/forum/<model("forum.forum"):forum>/flagged_queue', type='http', auth="user", website=True)
    def flagged_queue(self, forum, **kwargs):
        user = request.env.user
        if user.karma < forum.karma_moderate:
            raise werkzeug.exceptions.NotFound()

        Post = request.env['forum.post']
        domain = [('forum_id', '=', forum.id), ('state', '=', 'flagged')]
        if kwargs.get('spam_post'):
            domain += [('name', 'ilike', kwargs.get('spam_post'))]
        flagged_posts_ids = Post.search(domain, order='write_date DESC')

        values = self._prepare_user_values(forum=forum)
        values.update({
            'posts_ids': flagged_posts_ids.sudo(),
            'queue_type': 'flagged',
            'flagged_queue_active': 1,
        })

        return request.render("website_forum.moderation_queue", values)

    @http.route('/forum/<model("forum.forum"):forum>/offensive_posts', type='http', auth="user", website=True)
    def offensive_posts(self, forum, **kwargs):
        user = request.env.user
        if user.karma < forum.karma_moderate:
            raise werkzeug.exceptions.NotFound()

        Post = request.env['forum.post']
        domain = [('forum_id', '=', forum.id), ('state', '=', 'offensive'), ('active', '=', False)]
        offensive_posts_ids = Post.search(domain, order='write_date DESC')

        values = self._prepare_user_values(forum=forum)
        values.update({
            'posts_ids': offensive_posts_ids.sudo(),
            'queue_type': 'offensive',
        })

        return request.render("website_forum.moderation_queue", values)

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/validate', type='http', auth="user", website=True)
    def post_accept(self, forum, post, **kwargs):
        url = "/forum/%s/validation_queue" % (slug(forum))
        if post.state == 'flagged':
            url = "/forum/%s/flagged_queue" % (slug(forum))
        elif post.state == 'offensive':
            url = "/forum/%s/offensive_posts" % (slug(forum))
        post.validate()
        return werkzeug.utils.redirect(url)

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/refuse', type='http', auth="user", website=True)
    def post_refuse(self, forum, post, **kwargs):
        post.refuse()
        return self.question_ask_for_close(forum, post)

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/flag', type='json', auth="public", website=True)
    def post_flag(self, forum, post, **kwargs):
        if not request.session.uid:
            return {'error': 'anonymous_user'}
        return post.flag()[0]

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/ask_for_mark_as_offensive', type='http', auth="user", methods=['GET'], website=True)
    def post_ask_for_mark_as_offensive(self, forum, post, **kwargs):
        offensive_reasons = request.env['forum.post.reason'].search([('reason_type', '=', 'offensive')])

        values = self._prepare_user_values(forum=forum)
        values.update({
            'question': post,
            'forum': forum,
            'reasons': offensive_reasons,
            'offensive': True,
        })
        return request.render("website_forum.close_post", values)

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/mark_as_offensive', type='http', auth="user", methods=["POST"], website=True)
    def post_mark_as_offensive(self, forum, post, **kwargs):
        post.mark_as_offensive(reason_id=int(kwargs.get('reason_id', False)))
        url = ''
        if post.parent_id:
            url = "/forum/%s/%s/#answer-%s" % (slug(forum), post.parent_id.id, post.id)
        else:
            url = "/forum/%s/%s" % (slug(forum), slug(post))
        return werkzeug.utils.redirect(url)

    # User
    # --------------------------------------------------
    @http.route(['/forum/<model("forum.forum"):forum>/partner/<int:partner_id>'], type='http', auth="public", website=True)
    def open_partner(self, forum, partner_id=0, **post):
        if partner_id:
            partner = request.env['res.partner'].sudo().search([('id', '=', partner_id)])
            if partner and partner.user_ids:
                return werkzeug.utils.redirect("/forum/%s/user/%d" % (slug(forum), partner.user_ids[0].id))
        return werkzeug.utils.redirect("/forum/%s" % slug(forum))

    # Profile
    # -----------------------------------

    @http.route(['/forum/<model("forum.forum"):forum>/user/<int:user_id>'], type='http', auth="public", website=True)
    def view_user_forum_profile(self, forum, user_id, forum_origin, **post):
        return werkzeug.utils.redirect('/profile/user/' + str(user_id) + '?forum_id=' + str(forum.id) + '&forum_origin=' + str(forum_origin))

    def _prepare_user_profile_values(self, user, **post):
        values = super(WebsiteForum, self)._prepare_user_profile_values(user, **post)
        if not post.get('no_forum'):
            if post.get('forum'):
                forums = post['forum']
            elif post.get('forum_id'):
                forums = request.env['forum.forum'].browse(int(post['forum_id']))
                values.update({
                    'edit_button_url_param': 'forum_id=%s' % str(post['forum_id']),
                    'forum_filtered': forums.name,
                })
            else:
                forums = request.env['forum.forum'].search([])

            values.update(self._prepare_user_values(forum=forums[0] if len(forums) == 1 else True, **post))
            if forums:
                values.update(self._prepare_open_forum_user(user, forums))
        return values

    def _prepare_open_forum_user(self, user, forums, **kwargs):
        Post = request.env['forum.post']
        Vote = request.env['forum.post.vote']
        Activity = request.env['mail.message']
        Followers = request.env['mail.followers']
        Data = request.env["ir.model.data"]

        # questions and answers by user
        user_question_ids = Post.search([
            ('parent_id', '=', False),
            ('forum_id', 'in', forums.ids), ('create_uid', '=', user.id)],
            order='create_date desc')
        count_user_questions = len(user_question_ids)
        min_karma_unlink = min(forums.mapped('karma_unlink_all'))

        # limit length of visible posts by default for performance reasons, except for the high
        # karma users (not many of them, and they need it to properly moderate the forum)
        post_display_limit = None
        if request.env.user.karma < min_karma_unlink:
            post_display_limit = 20

        user_questions = user_question_ids[:post_display_limit]
        user_answer_ids = Post.search([
            ('parent_id', '!=', False),
            ('forum_id', 'in', forums.ids), ('create_uid', '=', user.id)],
            order='create_date desc')
        count_user_answers = len(user_answer_ids)
        user_answers = user_answer_ids[:post_display_limit]

        # showing questions which user following
        post_ids = [follower.res_id for follower in Followers.sudo().search(
            [('res_model', '=', 'forum.post'), ('partner_id', '=', user.partner_id.id)])]
        followed = Post.search([('id', 'in', post_ids), ('forum_id', 'in', forums.ids), ('parent_id', '=', False)])

        # showing Favourite questions of user.
        favourite = Post.search(
            [('favourite_ids', '=', user.id), ('forum_id', 'in', forums.ids), ('parent_id', '=', False)])

        # votes which given on users questions and answers.
        data = Vote.read_group([('forum_id', 'in', forums.ids), ('recipient_id', '=', user.id)], ["vote"],
                               groupby=["vote"])
        up_votes, down_votes = 0, 0
        for rec in data:
            if rec['vote'] == '1':
                up_votes = rec['vote_count']
            elif rec['vote'] == '-1':
                down_votes = rec['vote_count']

        # Votes which given by users on others questions and answers.
        vote_ids = Vote.search([('user_id', '=', user.id), ('forum_id', 'in', forums.ids)])

        # activity by user.
        model, comment = Data.get_object_reference('mail', 'mt_comment')
        activities = Activity.search(
            [('res_id', 'in', (user_question_ids + user_answer_ids).ids), ('model', '=', 'forum.post'),
             ('subtype_id', '!=', comment)],
            order='date DESC', limit=100)

        posts = {}
        for act in activities:
            posts[act.res_id] = True
        posts_ids = Post.search([('id', 'in', list(posts))])
        posts = {x.id: (x.parent_id or x, x.parent_id and x or False) for x in posts_ids}

        # TDE CLEANME MASTER: couldn't it be rewritten using a 'menu' key instead of one key for each menu ?
        if user == request.env.user:
            kwargs['my_profile'] = True
        else:
            kwargs['users'] = True

        values = {
            'uid': request.env.user.id,
            'user': user,
            'main_object': user,
            'searches': kwargs,
            'questions': user_questions,
            'count_questions': count_user_questions,
            'answers': user_answers,
            'count_answers': count_user_answers,
            'followed': followed,
            'favourite': favourite,
            'up_votes': up_votes,
            'down_votes': down_votes,
            'activities': activities,
            'posts': posts,
            'vote_post': vote_ids,
            'is_profile_page': True,
            'badge_category': 'forum',
        }

        return values

    # Messaging
    # --------------------------------------------------

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/comment/<model("mail.message"):comment>/convert_to_answer', type='http', auth="user", methods=['POST'], website=True)
    def convert_comment_to_answer(self, forum, post, comment, **kwarg):
        post = request.env['forum.post'].convert_comment_to_answer(comment.id)
        if not post:
            return werkzeug.utils.redirect("/forum/%s" % slug(forum))
        question = post.parent_id if post.parent_id else post
        return werkzeug.utils.redirect("/forum/%s/%s" % (slug(forum), slug(question)))

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/convert_to_comment', type='http', auth="user", methods=['POST'], website=True)
    def convert_answer_to_comment(self, forum, post, **kwarg):
        question = post.parent_id
        new_msg = post.convert_answer_to_comment()
        if not new_msg:
            return werkzeug.utils.redirect("/forum/%s" % slug(forum))
        return werkzeug.utils.redirect("/forum/%s/%s" % (slug(forum), slug(question)))

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/comment/<model("mail.message"):comment>/delete', type='json', auth="user", website=True)
    def delete_comment(self, forum, post, comment, **kwarg):
        if not request.session.uid:
            return {'error': 'anonymous_user'}
        return post.unlink_comment(comment.id)[0]

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main

```

## File: data\badges_answer.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- QUALITY (VOTES) -->
        <!-- Teacher: at least 3 upvotes -->
        <record id="badge_a_1" model="gamification.badge">
            <field name="name">Teacher</field>
            <field name="description">Received at least 3 upvote for an answer for the first time</field>
            <field name="level">bronze</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_teacher">
            <field name="name">Teacher</field>
            <field name="description">Received at least 3 upvote for an answer for the first time</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="condition">higher</field>
            <field name="domain">[('parent_id', '!=', False), ('vote_count', '>=', 3)]</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_teacher">
            <field name="name">Teacher</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_a_1"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_teacher">
            <field name="definition_id" ref="definition_teacher"/>
            <field name="challenge_id" ref="challenge_teacher"/>
            <field name="target_goal">1</field>
        </record>
        <!-- Nice: at least 4 upvotes -->
        <record id="badge_a_2" model="gamification.badge">
            <field name="name">Nice Answer</field>
            <field name="description">Answer voted up 4 times</field>
            <field name="level">bronze</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_nice_answer">
            <field name="name">Nice Answer (4)</field>
            <field name="description">Answer voted up 4 times</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="condition">higher</field>
            <field name="domain">[('parent_id', '!=', False), ('vote_count', '>=', 4)]</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_nice_answer">
            <field name="name">Nice Answer</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_a_2"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_nice_answer">
            <field name="definition_id" ref="definition_nice_answer"/>
            <field name="challenge_id" ref="challenge_nice_answer"/>
            <field name="target_goal">1</field>
        </record>
        <!-- Good: at least 6 upvotes -->
        <record id="badge_a_3" model="gamification.badge">
            <field name="name">Good Answer</field>
            <field name="description">Answer voted up 6 times</field>
            <field name="level">silver</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_good_answer">
            <field name="name">Good Answer (6)</field>
            <field name="description">Answer voted up 6 times</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="condition">higher</field>
            <field name="domain">[('parent_id', '!=', False), ('vote_count', '>=', 6)]</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_good_answer">
            <field name="name">Good Answer</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_a_3"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_good_answer">
            <field name="definition_id" ref="definition_good_answer"/>
            <field name="challenge_id" ref="challenge_good_answer"/>
            <field name="target_goal">1</field>
        </record>
        <!-- Great: at least 15 upvotes -->
        <record id="badge_a_4" model="gamification.badge">
            <field name="name">Great Answer</field>
            <field name="description">Answer voted up 15 times</field>
            <field name="level">gold</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_great_answer">
            <field name="name">Great Answer (15)</field>
            <field name="description">Answer voted up 15 times</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="condition">higher</field>
            <field name="domain">[('parent_id', '!=', False), ('vote_count', '>=', 15)]</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_great_answer">
            <field name="name">Great Answer</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_a_4"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_great_answer">
            <field name="definition_id" ref="definition_great_answer"/>
            <field name="challenge_id" ref="challenge_great_answer"/>
            <field name="target_goal">1</field>
        </record>

        <!-- ACCEPTANCE -->
        <!-- Enlightened: at least 3 upvotes for an accepted answer -->
        <record id="badge_a_5" model="gamification.badge">
            <field name="name">Enlightened</field>
            <field name="description">Answer was accepted with 3 or more votes</field>
            <field name="level">silver</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_enlightened">
            <field name="name">Enlightened</field>
            <field name="description">Answer was accepted with 3 or more votes</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="condition">higher</field>
            <field name="domain">[('parent_id', '!=', False), ('vote_count', '>=', 3), ('is_correct', '=', True)]</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_enlightened">
            <field name="name">Enlightened</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_a_5"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_enlightened">
            <field name="definition_id" ref="definition_enlightened"/>
            <field name="challenge_id" ref="challenge_enlightened"/>
            <field name="target_goal">1</field>
        </record>
        <!-- Guru: at least 15 upvotes for an accepted answer -->
        <record id="badge_a_6" model="gamification.badge">
            <field name="name">Guru</field>
            <field name="description">Answer accepted with 15 or more votes</field>
            <field name="level">silver</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_guru">
            <field name="name">Guru (15)</field>
            <field name="description">Answer accepted with 15 or more votes</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="condition">higher</field>
            <field name="domain">[('parent_id', '!=', False), ('vote_count', '>=', 15), ('is_correct', '=', True)]</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_guru">
            <field name="name">Guru</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_a_6"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_guru">
            <field name="definition_id" ref="definition_guru"/>
            <field name="target_goal">1</field>
            <field name="challenge_id" ref="challenge_guru"/>
        </record>

        <!-- Sealf Leaner: own question, 3+ upvotes -->
        <record id="badge_a_8" model="gamification.badge">
            <field name="name">Self-Learner</field>
            <field name="description">Answered own question with at least 4 up votes</field>
            <field name="level">gold</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_self_learner">
            <field name="name">Self-Learner</field>
            <field name="description">Answer own question with at least 4 up votes</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="condition">higher</field>
            <field name="domain">[('self_reply', '=', True), ('vote_count', '>=', 4)]</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_self_learner">
            <field name="name">Self-Learner</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_a_8"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_self_learner">
            <field name="definition_id" ref="definition_self_learner"/>
            <field name="target_goal">1</field>
            <field name="challenge_id" ref="challenge_self_learner"/>
        </record>

    </data>
</odoo>

```

## File: data\badges_moderation.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Cleanup: answer or question edition -->
        <!-- Not rollback feature in forum -->
<!--         <record id="badge_3" model="gamification.badge">
            <field name="name">Cleanup</field>
            <field name="description">First rollback</field>
            <field name="level">gold</field>
        </record> -->

        <!-- Critic: downvote based -->
        <record id="badge_5" model="gamification.badge">
            <field name="name">Critic</field>
            <field name="description">First downvote</field>
            <field name="level">bronze</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_critic">
            <field name="name">Critic</field>
            <field name="description">First downvote</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post_vote"/>
            <field name="condition">higher</field>
            <field name="domain">[('vote', '=', '-1')]</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post_vote__user_id"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_critic">
            <field name="name">Critic</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_5"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_critic">
            <field name="definition_id" ref="definition_critic"/>
            <field name="challenge_id" ref="challenge_critic"/>
            <field name="target_goal">1</field>
        </record>

        <!-- Disciplined: delete own post with >=3 upvotes -->
        <record id="badge_6" model="gamification.badge">
            <field name="name">Disciplined</field>
            <field name="description">Deleted own post with 3 or more upvotes</field>
            <field name="level">bronze</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_disciplined">
            <field name="name">Disciplined</field>
            <field name="description">Delete own post with 3 or more upvotes</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="condition">higher</field>
            <field name="domain">[('vote_count', '>=', 3), ('active', '=', False)]</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_disciplined">
            <field name="name">Disciplined</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_6"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_disciplined">
            <field name="definition_id" ref="definition_disciplined"/>
            <field name="challenge_id" ref="challenge_disciplined"/>
            <field name="target_goal">1</field>
        </record>

        <!-- Editor: first edit -->
        <record id="badge_7" model="gamification.badge">
            <field name="name">Editor</field>
            <field name="description">First edit</field>
            <field name="level">gold</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_editor">
            <field name="name">Editor</field>
            <field name="description">First edit of answer or question</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="mail.model_mail_message"/>
            <field name="condition">higher</field>
            <field name="domain" eval="[('model', '=', 'forum.post'), ('subtype_id', 'in', [ref('website_forum.mt_answer_edit'), ref('website_forum.mt_question_edit')])]"/>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="mail.field_mail_message__author_id"/>
            <field name="batch_user_expression">user.partner_id.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_editor">
            <field name="name">Editor</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_7"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_editor">
            <field name="definition_id" ref="definition_editor"/>
            <field name="challenge_id" ref="challenge_editor"/>
            <field name="target_goal">1</field>
        </record>

        <record id="badge_31" model="gamification.badge">
            <field name="name">Supporter</field>
            <field name="description">First upvote</field>
            <field name="level">gold</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_supporter">
            <field name="name">Supporter</field>
            <field name="description">First upvote</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post_vote"/>
            <field name="condition">higher</field>
            <field name="domain">[('vote', '=', '1')]</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post_vote__user_id"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_supporter">
            <field name="name">Supporter</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_31"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_supporter">
            <field name="definition_id" ref="definition_supporter"/>
            <field name="target_goal">1</field>
            <field name="challenge_id" ref="challenge_supporter"/>
        </record>


        <record id="badge_23" model="gamification.badge">
            <field name="name">Peer Pressure</field>
            <field name="description">Deleted own post with 3 or more downvotes</field>
            <field name="level">gold</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_peer_pressure">
            <field name="name">Peer Pressure</field>
            <field name="description">Delete own post with 3 or more down votes</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="condition">higher</field>
            <field name="domain">[('vote_count', '&lt;=', -3), ('active', '=', False)]</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_peer_pressure">
            <field name="name">Peer Pressure</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_23"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_peer_pressure">
            <field name="definition_id" ref="definition_peer_pressure"/>
            <field name="target_goal">1</field>
            <field name="challenge_id" ref="challenge_peer_pressure"/>
        </record>

    </data>
</odoo>

```

## File: data\badges_participation.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Biography: complet your profile -->
        <record id="badge_p_1" model="gamification.badge">
            <field name="name">Autobiographer</field>
            <field name="description">Completed own biography</field>
            <field name="level">bronze</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_configure_profile">
            <field name="name">Completed own biography</field>
            <field name="description">Write some information about yourself</field>
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
        <record model="gamification.challenge" id="challenge_configure_profile">
            <field name="name">Complete own biography</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_p_1"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_configure_profile">
            <field name="definition_id" ref="definition_configure_profile"/>
            <field name="challenge_id" ref="challenge_configure_profile"/>
            <field name="target_goal">1</field>
        </record>

        <!-- Commentator: at least 10 comments posted on posts -->
        <record id="badge_p_2" model="gamification.badge">
            <field name="name">Commentator</field>
            <field name="description">Posted 10 comments</field>
            <field name="level">bronze</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_commentator">
            <field name="name">Commentator</field>
            <field name="description">Comment an answer or a question</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="mail.model_mail_message"/>
            <field name="condition">higher</field>
            <field name="domain" eval="[('message_type', '=', 'comment'), ('subtype_id', '=', ref('mail.mt_comment')), ('model', '=', 'forum.post')]"/>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="mail.field_mail_message__author_id"/>
            <field name="batch_user_expression">user.partner_id.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_commentator">
            <field name="name">Commentator</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_p_2"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_commentator">
            <field name="definition_id" ref="definition_commentator"/>
            <field name="challenge_id" ref="challenge_commentator"/>
            <field name="target_goal">10</field>
        </record>

        <!-- Pundit: 10 answers with at least score of 10 -->
        <record id="badge_25" model="gamification.badge">
            <field name="name">Pundit</field>
            <field name="description">Left 10 answers with score of 10 or more</field>
            <field name="level">silver</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_pundit">
            <field name="name">Pundit</field>
            <field name="description">Post 10 answers with score of 10 or more</field>
            <field name="display_mode">boolean</field>
            <field name="condition">higher</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="domain" eval="[('parent', '!=', False), ('vote_count' '>=', 10)]"/>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_pundit">
            <field name="name">Pundit</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_25"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_pundit">
            <field name="definition_id" ref="definition_pundit"/>
            <field name="target_goal">10</field>
            <field name="challenge_id" ref="challenge_pundit"/>
        </record>

        <!-- Chief Commentator: 100 comments -->
        <record id="badge_p_4" model="gamification.badge">
            <field name="name">Chief Commentator</field>
            <field name="description">Posted 100 comments</field>
            <field name="level">silver</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.challenge" id="challenge_chief_commentator">
            <field name="name">Chief Commentator</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_p_4"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_chief_commentator">
            <field name="definition_id" ref="definition_commentator"/>
            <field name="challenge_id" ref="challenge_chief_commentator"/>
            <field name="target_goal">100</field>
        </record>

        <record id="badge_32" model="gamification.badge">
            <field name="name">Taxonomist</field>
            <field name="description">Created a tag used by 15 questions</field>
            <field name="level">silver</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_taxonomist">
            <field name="name">Taxonomist</field>
            <field name="description">Create a tag which can used in minimum 15 questions</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_tag"/>
            <field name="condition">higher</field>
            <field name="domain">[('posts_count', '>=', 15)]</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_tag__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_taxonomist">
            <field name="name">Taxonomist</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_32"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_taxonomist">
            <field name="definition_id" ref="definition_taxonomist"/>
            <field name="challenge_id" ref="challenge_taxonomist"/>
            <field name="target_goal">1</field>
        </record>

    </data>
</odoo>

```

## File: data\badges_question.xml

```xml
<!-- <?xml version="1.0" encoding="utf-8"?> -->
<odoo>
    <data noupdate="1">

        <!-- POPULARITY (VIEWS) -->
        <!-- Popular: 150 views -->
        <record id="badge_q_1" model="gamification.badge">
            <field name="name">Popular Question</field>
            <field name="description">Asked a question with at least 150 views</field>
            <field name="level">bronze</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_popular_question">
            <field name="name">Popular Question (150)</field>
            <field name="description">Asked a question with at least 150 views</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="domain">[('parent_id', '=', False), ('views', '>=', 150)]</field>
            <field name="condition">higher</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_popular_question">
            <field name="name">Popular Question</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_q_1"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_popular_question">
            <field name="definition_id" ref="definition_popular_question"/>
            <field name="challenge_id" ref="challenge_popular_question"/>
            <field name="target_goal">1</field>
        </record>

        <!-- Notable: 250 views -->
        <record id="badge_q_2" model="gamification.badge">
            <field name="name">Notable Question</field>
            <field name="description">Asked a question with at least 250 views</field>
            <field name="level">silver</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_notable_question">
            <field name="name">Popular Question (250)</field>
            <field name="description">Asked a question with at least 250 views</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="domain">[('parent_id', '=', False), ('views', '>=', 250)]</field>
            <field name="condition">higher</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_notable_question">
            <field name="name">Notable Question</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_q_2"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_notable_question">
            <field name="definition_id" ref="definition_notable_question"/>
            <field name="challenge_id" ref="challenge_notable_question"/>
            <field name="target_goal">1</field>
        </record>
        <!-- Famous: 500 views -->
        <record id="badge_q_3" model="gamification.badge">
            <field name="name">Famous Question</field>
            <field name="description">Asked a question with at least 500 views</field>
            <field name="level">gold</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_famous_question">
            <field name="name">Popular Question (500)</field>
            <field name="description">Asked a question with at least 500 views</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="domain">[('parent_id', '=', False), ('views', '>=', 500)]</field>
            <field name="condition">higher</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_famous_question">
            <field name="name">Famous Question</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_q_3"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_famous_question">
            <field name="definition_id" ref="definition_famous_question"/>
            <field name="challenge_id" ref="challenge_famous_question"/>
            <field name="target_goal">1</field>
        </record>

        <!-- FAVORITE -->
        <!-- Credible: at least 1 user have it in favorite -->
        <record id="badge_q_4" model="gamification.badge">
            <field name="name">Credible Question</field>
            <field name="description">Question set as favorite by 1 user</field>
            <field name="level">bronze</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_favorite_question_1">
            <field name="name">Favourite Question (1)</field>
            <field name="description">Question set as favorite by 1 user</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="domain">[('parent_id', '=', False), ('favourite_count', '>=', 1)]</field>
            <field name="condition">higher</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_favorite_question_1">
            <field name="name">Credible Question</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_q_4"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_favorite_question_1">
            <field name="definition_id" ref="definition_favorite_question_1"/>
            <field name="challenge_id" ref="challenge_favorite_question_1"/>
            <field name="target_goal">1</field>
        </record>
        <!-- Favorite: at least 5 users have it in favorite -->
        <record id="badge_q_5" model="gamification.badge">
            <field name="name">Favorite Question</field>
            <field name="description">Question set as favorite by 5 users</field>
            <field name="level">silver</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_favorite_question_5">
            <field name="name">Favourite Question (5)</field>
            <field name="description">Question set as favorite by 5 user</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="domain">[('parent_id', '=', False), ('favourite_count', '>=', 5)]</field>
            <field name="condition">higher</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_favorite_question_5">
            <field name="name">Favorite Question</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_q_5"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_favorite_question_5">
            <field name="definition_id" ref="definition_favorite_question_5"/>
            <field name="challenge_id" ref="challenge_favorite_question_5"/>
            <field name="target_goal">1</field>
        </record>
        <!-- Stellar: at least 25 users have it in favorite -->
        <record id="badge_q_6" model="gamification.badge">
            <field name="name">Stellar Question</field>
            <field name="description">Question set as favorite by 25 users</field>
            <field name="level">bronze</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_stellar_question_25">
            <field name="name">Favourite Question (25)</field>
            <field name="description">Question set as favorite by 25 user</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="domain">[('parent_id', '=', False), ('favourite_count', '>=', 25)]</field>
            <field name="condition">higher</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_stellar_question_25">
            <field name="name">Stellar Question</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_q_6"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_stellar_question_25">
            <field name="definition_id" ref="definition_stellar_question_25"/>
            <field name="challenge_id" ref="challenge_stellar_question_25"/>
            <field name="target_goal">1</field>
        </record>

        <!-- QUALITY (VOTES) -->
        <!-- Student: at least 1 upvote -->
        <record id="badge_q_7" model="gamification.badge">
            <field name="name">Student</field>
            <field name="description">Asked first question with at least one up vote</field>
            <field name="level">gold</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_student">
            <field name="name">Upvoted question (1)</field>
            <field name="description">Asked first question with at least one up vote</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="domain">[('parent_id', '=', False), ('vote_count', '>=', 1)]</field>
            <field name="condition">higher</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_student">
            <field name="name">Student</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_q_7"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_student">
            <field name="definition_id" ref="definition_student"/>
            <field name="challenge_id" ref="challenge_student"/>
            <field name="target_goal">1</field>
        </record>
        <!-- Nice: at least 4 upvotes -->
        <record id="badge_q_8" model="gamification.badge">
            <field name="name">Nice Question</field>
            <field name="description">Question voted up 4 times</field>
            <field name="level">bronze</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_nice_question">
            <field name="name">Upvoted question (4)</field>
            <field name="description">Asked first question with at least 4 up votes</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="domain">[('parent_id', '=', False), ('vote_count', '>=', 4)]</field>
            <field name="condition">higher</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_nice_question">
            <field name="name">Nice Question</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_q_8"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_nice_question">
            <field name="definition_id" ref="definition_nice_question"/>
            <field name="challenge_id" ref="challenge_nice_question"/>
            <field name="target_goal">1</field>
        </record>
        <!-- Good: at least 6 upvotes -->
        <record id="badge_q_9" model="gamification.badge">
            <field name="name">Good Question</field>
            <field name="description">Question voted up 6 times</field>
            <field name="level">silver</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_good_question">
            <field name="name">Upvoted question (6)</field>
            <field name="description">Asked first question with at least 6 up votes</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="domain">[('parent_id', '=', False), ('vote_count', '>=', 6)]</field>
            <field name="condition">higher</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_good_question">
            <field name="name">Good Question</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_q_9"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_good_question">
            <field name="definition_id" ref="definition_good_question"/>
            <field name="challenge_id" ref="challenge_good_question"/>
            <field name="target_goal">1</field>
        </record>
        <!-- Great: at least 15 upvotes -->
        <record id="badge_q_10" model="gamification.badge">
            <field name="name">Great Question</field>
            <field name="description">Question voted up 15 times</field>
            <field name="level">gold</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_great_question">
            <field name="name">Upvoted question (15)</field>
            <field name="description">Asked first question with at least 15 up votes</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="domain">[('parent_id', '=', False), ('vote_count', '>=', 15)]</field>
            <field name="condition">higher</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_great_question">
            <field name="name">Great Question</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_q_10"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_great_question">
            <field name="definition_id" ref="definition_great_question"/>
            <field name="target_goal">1</field>
            <field name="challenge_id" ref="challenge_great_question"/>
        </record>

        <!-- Question + Answer -->
        <record id="badge_26" model="gamification.badge">
            <field name="name">Scholar</field>
            <field name="description">Asked a question and accepted an answer</field>
            <field name="level">gold</field>
            <field name="rule_auth">nobody</field>
        </record>
        <record model="gamification.goal.definition" id="definition_scholar">
            <field name="name">Scholar</field>
            <field name="description">Ask a question and accept an answer</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="website_forum.model_forum_post"/>
            <field name="domain">[('parent_id', '=', False), ('has_validated_answer', '=', True)]</field>
            <field name="condition">higher</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="website_forum.field_forum_post__create_uid"/>
            <field name="batch_user_expression">user.id</field>
        </record>
        <record model="gamification.challenge" id="challenge_scholar">
            <field name="name">Scholar</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="reward_id" ref="badge_26"/>
            <field name="reward_realtime">True</field>
            <field name="user_domain">[('karma', '>', 0)]</field>
            <field name="state">inprogress</field>
            <field name="challenge_category">forum</field>
        </record>
        <record model="gamification.challenge.line" id="line_scholar">
            <field name="definition_id" ref="definition_scholar"/>
            <field name="target_goal">1</field>
            <field name="challenge_id" ref="challenge_scholar"/>
        </record>

    </data>
</odoo>

```

## File: data\forum_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="forum_help" model="forum.forum">
            <field name="name">Help</field>
            <field name="description">This community is for professionals and enthusiasts of our products and services. Share and discuss the best content and new marketing ideas, build your professional profile and become a better marketer together.</field>
        </record>

        <record id="menu_website_forums" model="website.menu">
            <field name="name">Forum</field>
            <field name="url">/forum</field>
            <field name="parent_id" ref="website.main_menu"/>
            <field name="sequence" type="int">35</field>
        </record>
    </data>
    <data>
        <function model="ir.config_parameter" name="set_param" eval="('auth_signup.invitation_scope', 'b2c')"/>

        <!-- JUMP TO FORUM AT INSTALL -->
        <record id="action_open_forum" model="ir.actions.act_url">
            <field name="name">Forum</field>
            <field name="target">self</field>
            <field name="url" eval="'/forum/'+str(ref('website_forum.forum_help'))"/>
        </record>

    </data>
    <data noupdate="1">
        <!-- Answers subtypes -->
        <record id="mt_answer_new" model="mail.message.subtype">
            <field name="name">New Answer</field>
            <field name="res_model">forum.post</field>
            <field name="default" eval="True"/>
            <field name="hidden" eval="False"/>
            <field name="description">New Answer</field>
        </record>
        <record id="mt_answer_edit" model="mail.message.subtype">
            <field name="name">Answer Edited</field>
            <field name="res_model">forum.post</field>
            <field name="default" eval="False"/>
            <field name="description">Answer Edited</field>
        </record>
        <!-- Questions subtypes -->
        <record id="mt_question_new" model="mail.message.subtype">
            <field name="name">New Question</field>
            <field name="res_model">forum.post</field>
            <field name="default" eval="True"/>
            <field name="description">New Question</field>
        </record>
        <record id="mt_question_edit" model="mail.message.subtype">
            <field name="name">Question Edited</field>
            <field name="res_model">forum.post</field>
            <field name="default" eval="False"/>
            <field name="description">Question Edited</field>
        </record>
        <!-- Forum subtypes, to follow all answers or questions -->
        <record id="mt_forum_answer_new" model="mail.message.subtype">
            <field name="name">New Answer</field>
            <field name="res_model">forum.forum</field>
            <field name="default" eval="True"/>
            <field name="hidden" eval="False"/>
            <field name="parent_id" ref="mt_answer_new"/>
            <field name="relation_field">forum_id</field>
        </record>
        <record id="mt_forum_question_new" model="mail.message.subtype">
            <field name="name">New Question</field>
            <field name="res_model">forum.forum</field>
            <field name="default" eval="True"/>
            <field name="hidden" eval="False"/>
            <field name="parent_id" ref="mt_question_new"/>
            <field name="relation_field">forum_id</field>
        </record>

        <record id="base.open_menu" model="ir.actions.todo">
            <field name="action_id" ref="action_open_forum"/>
            <field name="state">open</field>
        </record>

        <!-- Reasons for closing Post -->
        <record id="reason_1" model="forum.post.reason">
            <field name="name">Duplicate post</field>
            <field name="reason_type">basic</field>
        </record>
        <record id="reason_2" model="forum.post.reason">
            <field name="name">Off-topic or not relevant</field>
            <field name="reason_type">basic</field>
        </record>
        <record id="reason_3" model="forum.post.reason">
            <field name="name">Too subjective and argumentative</field>
            <field name="reason_type">basic</field>
        </record>
        <record id="reason_4" model="forum.post.reason">
            <field name="name">Not a real post</field>
            <field name="reason_type">basic</field>
        </record>
        <record id="reason_6" model="forum.post.reason">
            <field name="name">Not relevant or out dated</field>
            <field name="reason_type">basic</field>
        </record>
        <record id="reason_7" model="forum.post.reason">
            <field name="name">Contains offensive or malicious remarks</field>
            <field name="reason_type">basic</field>
        </record>
        <record id="reason_8" model="forum.post.reason">
            <field name="name">Spam or advertising</field>
            <field name="reason_type">basic</field>
        </record>
        <record id="reason_9" model="forum.post.reason">
            <field name="name">Too localized</field>
            <field name="reason_type">basic</field>
        </record>
        <record id="reason_11" model="forum.post.reason">
            <field name="name">Insulting and offensive language</field>
            <field name="reason_type">offensive</field>
        </record>
        <record id="reason_12" model="forum.post.reason">
            <field name="name">Violent language</field>
            <field name="reason_type">offensive</field>
        </record>
        <record id="reason_13" model="forum.post.reason">
            <field name="name">Inappropriate and unacceptable statements</field>
            <field name="reason_type">offensive</field>
        </record>
        <record id="reason_14" model="forum.post.reason">
            <field name="name">Threatening language</field>
            <field name="reason_type">offensive</field>
        </record>
        <record id="reason_15" model="forum.post.reason">
            <field name="name">Racist and hate speech</field>
            <field name="reason_type">offensive</field>
        </record>

    </data>
</odoo>

```

## File: data\forum_default_faq.xml

```xml
<odoo>
    <data>
        <record id="default_faq" model="ir.ui.view">
            <field name="name">Faq Accordion</field>
            <field name="type">qweb</field>
            <field name="key">website_forum.faq_accordion</field>
            <field name="arch" type="xml">
                <section class="s_faq_collapse pt32 pb32">
                    <div class="container">
                        <div id="myCollapse" class="accordion" role="tablist">
                            <div class="card bg-white" data-name="Item">
                                <a href="#" role="tab" data-toggle="collapse" aria-expanded="false" class="collapsed card-header" data-target="#collapse1">
                                    <b>What kinds of questions can I ask here?</b>
                                </a>
                                <div id="collapse1" class="collapse" data-parent="#myCollapse" role="tabpanel">
                                    <div class="card-body">
                                        <p>This community is for professional and enthusiast users, partners and programmers. You can ask questions about:</p>
                                        <ul>
                                            <li>how to install Odoo on a specific infrastructure,</li>
                                            <li>how to configure or customize Odoo to specific business needs,</li>
                                            <li>what's the best way to use Odoo for a specific business need,</li>
                                            <li>how to develop modules for your own need,</li>
                                            <li>specific questions about Odoo service offers, etc.</li>
                                        </ul>
                                        <p><b>Before you ask - please make sure to search for a similar question.</b> You can search questions by their title or tags. It’s also OK to answer your own question.</p>
                                        <p><b>Please avoid asking questions that are too subjective and argumentative</b> or not relevant to this community.</p>
                                    </div>
                                </div>
                            </div>
                            <div class="card bg-white" data-name="Item">
                                <a href="#" role="tab" data-toggle="collapse" aria-expanded="false" class="collapsed card-header" data-target="#collapse2">
                                    <b>What should I avoid in my questions?</b>
                                </a>
                                <div id="collapse2" class="collapse" data-parent="#myCollapse" role="tabpanel">
                                    <div class="card-body">
                                        <p>You should only ask practical, answerable questions based on actual problems that you face. Chatty, open-ended questions diminish the usefulness of this site and push other questions off the front page.</p>
                                        <p>To prevent your question from being flagged and possibly removed, avoid asking subjective questions where …</p>
                                        <ul>
                                            <li>every answer is equally valid: “What’s your favorite ______?”</li>
                                            <li>your answer is provided along with the question, and you expect more answers: “I use ______ for ______, what do you use?”</li>
                                            <li>there is no actual problem to be solved: “I’m curious if other people feel like I do.”</li>
                                            <li>we are being asked an open-ended, hypothetical question: “What if ______ happened?”</li>
                                            <li>it is a rant disguised as a question: “______ sucks, am I right?”</li>
                                        </ul>
                                        <p>If you fit in one of these example or if your motivation for asking the question is “I would like to participate in a discussion about ______”, then you should not be asking here but on our mailing lists. However, if your motivation is “I would like others to explain ______ to me”, then you are probably OK.</p>
                                        <p>(The above section was adapted from Stackoverflow’s FAQ.)</p>
                                        <p>More over:</p>
                                        <ul>
                                            <li><b>Answers should not add or expand questions</b>. Instead either edit the question or add a question comment.</li>
                                            <li><b>Answers should not comment other answers</b>. Instead add a comment on the other answers.</li>
                                            <li><b>Answers shouldn't just point to other Questions</b>. Instead add a question comment indication "Possible duplicate of...". However, it's ok to include links to other questions or answers providing relevant additional information.</li>
                                            <li><b>Answers shouldn't just provide a link a solution</b>. Instead provide the solution description text in your answer, even if it's just a copy/paste. Links are welcome, but should be complementary to answer, referring sources or additional reading.</li>
                                        </ul>
                                    </div>
                                </div>
                            </div>
                            <div class="card bg-white" data-name="Item">
                                <a href="#" role="tab" data-toggle="collapse" aria-expanded="false" class="collapsed card-header" data-target="#collapse3">
                                    <b>What should I avoid in my answers?</b>
                                </a>
                                <div id="collapse3" class="collapse"  data-parent="#myCollapse" role="tabpanel">
                                    <div class="card-body">
                                        <p><b>Answers should not add or expand questions</b>. Instead, either edit the question or add a comment.</p>
                                        <p><b>Answers should not comment other answers</b>. Instead add a comment on the other answers.</p>
                                        <p><b>Answers shouldn't just point to other questions</b>.Instead add a comment indicating <i>"Possible duplicate of..."</i>. However, it's fine to include links to other questions or answers providing relevant additional information.</p>
                                        <p> <b>Answers shouldn't just provide a link a solution</b>. Instead provide the solution description text in your answer, even if it's just a copy/paste. Links are welcome, but should be complementary to answer, referring sources or additional reading.</p>
                                        <p><b>Answers should not start debates</b> This community Q&amp;A is not a discussion group. Please avoid holding debates in your answers as they tend to dilute the essence of questions and answers. For brief discussions please use commenting facility.</p>
                                        <p>When a question or answer is upvoted, the user who posted them will gain some points, which are called "karma points". These points serve as a rough measure of the community trust to him/her. Various moderation tasks are gradually assigned to the users based on those points.</p>
                                        <p>For example, if you ask an interesting question or give a helpful answer, your input will be upvoted. On the other hand if the answer is misleading - it will be downvoted. Each vote in favor will generate 10 points, each vote against will subtract 10 points. There is a limit of 200 points that can be accumulated for a question or answer per day. The table given at the end explains reputation point requirements for each type of moderation task.</p>
                                    </div>
                                </div>
                            </div>
                            <div class="card bg-white" data-name="Item">
                                <a href="#" role="tab" data-toggle="collapse" aria-expanded="false" class="collapsed card-header" data-target="#collapse4">
                                    <b>Why can other people edit my questions/answers?</b>
                                </a>
                                <div id="collapse4" class="collapse" data-parent="#myCollapse" role="tabpanel">
                                    <div class="card-body">
                                        <p>The goal of this site is create a relevant knowledge base that would answer questions related to Odoo.</p>
                                        <p>Therefore questions and answers can be edited like wiki pages by experienced users of this site in order to improve the overall quality of the knowledge base content. Such privileges are granted based on user karma level: you will be able to do the same once your karma gets high enough.</p>
                                        <p>If this approach is not for you, please respect the community.</p>
                                        <a t-attf-href="/forum/#{slug(forum)}/faq/karma">Here a table with the privileges and the karma level</a>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </section>
            </field>
        </record>
    </data>
</odoo>
```

## File: data\forum_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>



        <!-- Tag -->
        <record id="tags_0" model="forum.tag">
            <field name="name">Contract</field>
            <field name="forum_id" ref="website_forum.forum_help"/>
        </record>
        <record id="tags_1" model="forum.tag">
            <field name="name">Action</field>
            <field name="forum_id" ref="website_forum.forum_help"/>
        </record>
        <record id="tags_2" model="forum.tag">
            <field name="name">ecommerce</field>
            <field name="forum_id" ref="website_forum.forum_help"/>
        </record>
        <record id="tags_3" model="forum.tag">
            <field name="name">Development</field>
            <field name="forum_id" ref="website_forum.forum_help"/>
        </record>

        <!-- Questions -->
        <record id="question_0" model="forum.post">
            <field name="name">How to configure alerts for employee contract expiration</field>
            <field name="forum_id" ref="website_forum.forum_help"/>
            <field name="views">3</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="write_uid" ref="base.user_admin"/>
            <field name="tag_ids" eval="[(4,ref('website_forum.tags_0')), (4,ref('website_forum.tags_1'))]"/>
        </record>
        <record id="question_1" model="forum.post">
            <field name="name">CMS replacement for ERP and eCommerce</field>
            <field name="views">8</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="write_uid" ref="base.user_admin"/>
            <field name="forum_id" ref="website_forum.forum_help"/>
            <field name="content"><![CDATA[<p>I use Wordpress as a CMS and eCommerce platform. The developing in Wordpress is quite easy and solid but it missing ERP feature (there is single plugin to integrate with Frontaccounting) so I wonder:

Can I use Odoo as a replacement CMS of Wordpress + eCommerce plugin?

In simple words does Odoo became CMS+ERP platform?</p>]]></field>
            <field name="tag_ids" eval="[(4,ref('website_forum.tags_2'))]"/>
        </record>

        <!-- Answer -->
        <record id="answer_0" model="forum.post">
            <field name="forum_id" ref="website_forum.forum_help"/>
            <field name="name">Re: How to configure alerts for employee contract expiration</field>
            <field name="content"><![CDATA[<p>Just for posterity so other can see. Here are the steps to set automatic alerts on any contract.. i.e. HR Employee, or Fleet for example. I will use fleet as an example.</p>
<ul>
    <li>Step 1. As a user who has access rights to Technical Features, go to Settings --> Automated Actions. Create A new Automated Action. For the Related Document Model choose.. Contract information on a vehicle (you can also type in the actual model name.. fleet.vehicle.log.contract ) . Set the trigger date to ... Contract Expiration Date. The Next Field (Delay After Trigger Date) is a bit ridiculous. Who wants to be reminded of a contract expiration AFTER the fact? The field should say Days Before Date to Fire Action and the number should be converted to a negative. IMHO. Any way... to get a workable solution you must enter in the number in the negative. So for instance like me if you want to be warned 35 days BEFORE the expiration... put in Delay After Trigger Date.. the number -35 But the sake of testing, right now just put in -1 for 1 day before. Save the Action.
    <li>Step 2. Go to Server Actions and create new Action. Call it Fleet Contract Expiration Warning. The Object will be the same as above .. Contract information on a vehicle. The Action Type is Email. For email address I just put my email. Under subject put in... [[object.name]]. This will tell you the name of the car. Message you can put any text message you like. Now save the Server Action.</li>
    <li>Step 3. Now go back to the Automated Action you created and go to the Action tab next to the conditions tab. Click Add and add the server action you created . In this case Fleet Contract Expiration Warning. Then Save.</li>
    <li>Step 4. To test, set a contract to expire tomorrow under one of your fleets vehicles. Then Save it.</li>
    <li>Step 5. Go to Scheduled Actions.. Set interval number to 1. Interval Unit to Minutes. Then Set the Next Execution date to 2 minutes from now. If your SMTP is configured correctly you will start to get a mail every minute with the reminder.</li></ul>]]></field>
            <field name="parent_id" ref="question_0" />
        </record>
        <record id="answer_1" model="forum.post">
            <field name="forum_id" ref="website_forum.forum_help"/>
            <field name="name">Re: CMS replacement for ERP and eCommerce</field>
            <field name="content"><![CDATA[
<p>Odoo v8 provides a web module and an e-commerce module: www.odoo.com/page/website-builder
The CMS editor in Odoo web is nice but I prefer Drupal for customization and there is a Drupal module for Odoo. I think WP is better than Odoo web too.
</p>]]></field>
            <field name="parent_id" ref="question_1"/>
        </record>

        <!-- Post Vote  -->
        <record id="post_vote_0" model="forum.post.vote">
            <field name="post_id" ref="question_0"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="vote">1</field>
        </record>
        <record id="post_vote_1" model="forum.post.vote">
            <field name="post_id" ref="answer_0"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="vote">1</field>
        </record>
        
        <!-- Run Scheduler -->
        <function model="gamification.challenge" name="_cron_update">
            <value eval="False"/>
            <value eval="False"/>
        </function>

    </data>
</odoo>

```

## File: models\forum.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import math
import re

from datetime import datetime

from odoo import api, fields, models, tools, SUPERUSER_ID, _
from odoo.exceptions import UserError, ValidationError, AccessError
from odoo.tools import misc, sql
from odoo.tools.translate import html_translate
from odoo.addons.http_routing.models.ir_http import slug

_logger = logging.getLogger(__name__)


class Forum(models.Model):
    _name = 'forum.forum'
    _description = 'Forum'
    _inherit = ['mail.thread', 'image.mixin', 'website.seo.metadata', 'website.multi.mixin']
    _order = "sequence"

    # description and use
    name = fields.Char('Forum Name', required=True, translate=True)
    sequence = fields.Integer('Sequence', default=1)
    mode = fields.Selection([
        ('questions', 'Questions (1 answer)'),
        ('discussions', 'Discussions (multiple answers)')],
        string='Mode', required=True, default='questions',
        help='Questions mode: only one answer allowed\n Discussions mode: multiple answers allowed')
    privacy = fields.Selection([
        ('public', 'Public'),
        ('connected', 'Signed In'),
        ('private', 'Some users')],
        help="Public: Forum is public\nSigned In: Forum is visible for signed in users\nSome users: Forum and their content are hidden for non members of selected group",
        default='public')
    authorized_group_id = fields.Many2one('res.groups', 'Authorized Group')
    menu_id = fields.Many2one('website.menu', 'Menu', copy=False)
    active = fields.Boolean(default=True)
    faq = fields.Html('Guidelines', translate=html_translate, sanitize=False)
    description = fields.Text('Description', translate=True)
    teaser = fields.Text('Teaser', compute='_compute_teaser', store=True)
    welcome_message = fields.Html(
        'Welcome Message',
        translate=True,
        default="""<section>
                        <div class="container py-5">
                            <div class="row">
                                <div class="col-lg-12">
                                    <h1 class="text-center">Welcome!</h1>
                                    <p class="text-400 text-center">
                                        This community is for professionals and enthusiasts of our products and services.
                                        <br/>Share and discuss the best content and new marketing ideas, build your professional profile and become a better marketer together.
                                    </p>
                                </div>
                                <div class="col text-center mt-3">
                                    <a href="#" class="js_close_intro btn btn-outline-light mr-2">Hide Intro</a>
                                    <a class="btn btn-light forum_register_url" href="/web/login">Register</a>
                                </div>
                            </div>
                        </div>
                    </section>""")
    default_order = fields.Selection([
        ('create_date desc', 'Newest'),
        ('write_date desc', 'Last Updated'),
        ('vote_count desc', 'Most Voted'),
        ('relevancy desc', 'Relevance'),
        ('child_count desc', 'Answered')],
        string='Default', required=True, default='write_date desc')
    relevancy_post_vote = fields.Float('First Relevance Parameter', default=0.8, help="This formula is used in order to sort by relevance. The variable 'votes' represents number of votes for a post, and 'days' is number of days since the post creation")
    relevancy_time_decay = fields.Float('Second Relevance Parameter', default=1.8)
    allow_bump = fields.Boolean('Allow Bump', default=True,
                                help='Check this box to display a popup for posts older than 10 days '
                                     'without any given answer. The popup will offer to share it on social '
                                     'networks. When shared, a question is bumped at the top of the forum.')
    allow_share = fields.Boolean('Sharing Options', default=True,
                                 help='After posting the user will be proposed to share its question '
                                      'or answer on social networks, enabling social network propagation '
                                      'of the forum content.')
    # posts statistics
    post_ids = fields.One2many('forum.post', 'forum_id', string='Posts')
    last_post_id = fields.Many2one('forum.post', compute='_compute_last_post')
    total_posts = fields.Integer('# Posts', compute='_compute_forum_statistics')
    total_views = fields.Integer('# Views', compute='_compute_forum_statistics')
    total_answers = fields.Integer('# Answers', compute='_compute_forum_statistics')
    total_favorites = fields.Integer('# Favorites', compute='_compute_forum_statistics')
    count_posts_waiting_validation = fields.Integer(string="Number of posts waiting for validation", compute='_compute_count_posts_waiting_validation')
    count_flagged_posts = fields.Integer(string='Number of flagged posts', compute='_compute_count_flagged_posts')
    # karma generation
    karma_gen_question_new = fields.Integer(string='Asking a question', default=2)
    karma_gen_question_upvote = fields.Integer(string='Question upvoted', default=5)
    karma_gen_question_downvote = fields.Integer(string='Question downvoted', default=-2)
    karma_gen_answer_upvote = fields.Integer(string='Answer upvoted', default=10)
    karma_gen_answer_downvote = fields.Integer(string='Answer downvoted', default=-2)
    karma_gen_answer_accept = fields.Integer(string='Accepting an answer', default=2)
    karma_gen_answer_accepted = fields.Integer(string='Answer accepted', default=15)
    karma_gen_answer_flagged = fields.Integer(string='Answer flagged', default=-100)
    # karma-based actions
    karma_ask = fields.Integer(string='Ask questions', default=3)
    karma_answer = fields.Integer(string='Answer questions', default=3)
    karma_edit_own = fields.Integer(string='Edit own posts', default=1)
    karma_edit_all = fields.Integer(string='Edit all posts', default=300)
    karma_edit_retag = fields.Integer(string='Change question tags', default=75)
    karma_close_own = fields.Integer(string='Close own posts', default=100)
    karma_close_all = fields.Integer(string='Close all posts', default=500)
    karma_unlink_own = fields.Integer(string='Delete own posts', default=500)
    karma_unlink_all = fields.Integer(string='Delete all posts', default=1000)
    karma_tag_create = fields.Integer(string='Create new tags', default=30)
    karma_upvote = fields.Integer(string='Upvote', default=5)
    karma_downvote = fields.Integer(string='Downvote', default=50)
    karma_answer_accept_own = fields.Integer(string='Accept an answer on own questions', default=20)
    karma_answer_accept_all = fields.Integer(string='Accept an answer to all questions', default=500)
    karma_comment_own = fields.Integer(string='Comment own posts', default=1)
    karma_comment_all = fields.Integer(string='Comment all posts', default=1)
    karma_comment_convert_own = fields.Integer(string='Convert own answers to comments and vice versa', default=50)
    karma_comment_convert_all = fields.Integer(string='Convert all answers to comments and vice versa', default=500)
    karma_comment_unlink_own = fields.Integer(string='Unlink own comments', default=50)
    karma_comment_unlink_all = fields.Integer(string='Unlink all comments', default=500)
    karma_flag = fields.Integer(string='Flag a post as offensive', default=500)
    karma_dofollow = fields.Integer(string='Nofollow links', help='If the author has not enough karma, a nofollow attribute is added to links', default=500)
    karma_editor = fields.Integer(string='Editor Features: image and links',
                                  default=30)
    karma_user_bio = fields.Integer(string='Display detailed user biography', default=750)
    karma_post = fields.Integer(string='Ask questions without validation', default=100)
    karma_moderate = fields.Integer(string='Moderate posts', default=1000)

    @api.depends('post_ids')
    def _compute_last_post(self):
        for forum in self:
            forum.last_post_id = forum.post_ids.search([('forum_id', '=', forum.id), ('parent_id', '=', False), ('state', '=', 'active')], order='create_date desc', limit=1)

    @api.depends('description')
    def _compute_teaser(self):
        for forum in self:
            if forum.description:
                desc = forum.description.replace('\n', ' ')
                if len(forum.description) > 180:
                    forum.teaser = desc[:180] + '...'
                else:
                    forum.teaser = forum.description
            else:
                forum.teaser = ""

    @api.depends('post_ids.state', 'post_ids.views', 'post_ids.child_count', 'post_ids.favourite_count')
    def _compute_forum_statistics(self):
        default_stats = {'total_posts': 0, 'total_views': 0, 'total_answers': 0, 'total_favorites': 0}

        if not self.ids:
            self.update(default_stats)
            return

        result = {cid: dict(default_stats) for cid in self.ids}
        read_group_res = self.env['forum.post'].read_group(
            [('forum_id', 'in', self.ids), ('state', 'in', ('active', 'close')), ('parent_id', '=', False)],
            ['forum_id', 'views', 'child_count', 'favourite_count'],
            groupby=['forum_id'],
            lazy=False)
        for res_group in read_group_res:
            cid = res_group['forum_id'][0]
            result[cid]['total_posts'] += res_group.get('__count', 0)
            result[cid]['total_views'] += res_group.get('views', 0)
            result[cid]['total_answers'] += res_group.get('child_count', 0)
            result[cid]['total_favorites'] += 1 if res_group.get('favourite_count', 0) else 0

        for record in self:
            record.update(result[record.id])

    def _compute_count_posts_waiting_validation(self):
        for forum in self:
            domain = [('forum_id', '=', forum.id), ('state', '=', 'pending')]
            forum.count_posts_waiting_validation = self.env['forum.post'].search_count(domain)

    def _compute_count_flagged_posts(self):
        for forum in self:
            domain = [('forum_id', '=', forum.id), ('state', '=', 'flagged')]
            forum.count_flagged_posts = self.env['forum.post'].search_count(domain)

    def _set_default_faq(self):
        self.faq = self.env['ir.ui.view']._render_template('website_forum.faq_accordion', {"forum": self}).decode('utf-8')

    @api.model
    def create(self, values):
        res = super(Forum, self.with_context(mail_create_nolog=True, mail_create_nosubscribe=True)).create(values)
        res._set_default_faq()  # will trigger a write and call update_website_count
        return res

    def write(self, vals):
        if 'privacy' in vals:
            if not vals['privacy']:
                # The forum is neither public, neither private, remove menu to avoid conflict
                self.menu_id.unlink()
            elif vals['privacy'] == 'public':
                # The forum is public, the menu must be also public
                vals['authorized_group_id'] = False
                self.menu_id.write({'group_ids': [(5, 0, 0)]})
            elif vals['privacy'] == 'connected':
                vals['authorized_group_id'] = False
                self.menu_id.write({'group_ids': [(6, 0, [self.env.ref('base.group_portal').id, self.env.ref('base.group_user').id])]})
        if 'authorized_group_id' in vals and vals['authorized_group_id']:
            self.menu_id.write({'group_ids': [(6, 0, [vals['authorized_group_id']])]})

        res = super(Forum, self).write(vals)
        if 'active' in vals:
            # archiving/unarchiving a forum does it on its posts, too
            self.env['forum.post'].with_context(active_test=False).search([('forum_id', 'in', self.ids)]).write({'active': vals['active']})

        if 'active' in vals or 'website_id' in vals:
            self._update_website_count()
        return res

    def unlink(self):
        self._update_website_count()
        return super(Forum, self).unlink()

    @api.model  # TODO: Remove me, this is not an `api.model` method
    def _tag_to_write_vals(self, tags=''):
        Tag = self.env['forum.tag']
        post_tags = []
        existing_keep = []
        user = self.env.user
        for tag in (tag for tag in tags.split(',') if tag):
            if tag.startswith('_'):  # it's a new tag
                # check that not already created meanwhile or maybe excluded by the limit on the search
                tag_ids = Tag.search([('name', '=', tag[1:]), ('forum_id', '=', self.id)])
                if tag_ids:
                    existing_keep.append(int(tag_ids[0]))
                else:
                    # check if user have Karma needed to create need tag
                    if user.exists() and user.karma >= self.karma_tag_create and len(tag) and len(tag[1:].strip()):
                        post_tags.append((0, 0, {'name': tag[1:], 'forum_id': self.id}))
            else:
                existing_keep.append(int(tag))
        post_tags.insert(0, [6, 0, existing_keep])
        return post_tags

    def _compute_website_url(self):
        return '/forum/%s' % (slug(self))

    def get_tags_first_char(self):
        """ get set of first letter of forum tags """
        tags = self.env['forum.tag'].search([('forum_id', '=', self.id), ('posts_count', '>', 0)])
        return sorted(set([tag.name[0].upper() for tag in tags if len(tag.name)]))

    def go_to_website(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'target': 'self',
            'url': self._compute_website_url(),
        }

    @api.model
    def _update_website_count(self):
        for website in self.env['website'].sudo().search([]):
            website.forums_count = self.env['forum.forum'].sudo().search_count(website.website_domain())


class Post(models.Model):

    _name = 'forum.post'
    _description = 'Forum Post'
    _inherit = ['mail.thread', 'website.seo.metadata']
    _order = "is_correct DESC, vote_count DESC, write_date DESC"

    name = fields.Char('Title')
    forum_id = fields.Many2one('forum.forum', string='Forum', required=True)
    content = fields.Html('Content', strip_style=True)
    plain_content = fields.Text('Plain Content', compute='_get_plain_content', store=True)
    tag_ids = fields.Many2many('forum.tag', 'forum_tag_rel', 'forum_id', 'forum_tag_id', string='Tags')
    state = fields.Selection([('active', 'Active'), ('pending', 'Waiting Validation'), ('close', 'Closed'), ('offensive', 'Offensive'), ('flagged', 'Flagged')], string='Status', default='active')
    views = fields.Integer('Views', default=0, readonly=True, copy=False)
    active = fields.Boolean('Active', default=True)
    website_message_ids = fields.One2many(domain=lambda self: [('model', '=', self._name), ('message_type', 'in', ['email', 'comment'])])
    website_id = fields.Many2one(related='forum_id.website_id', readonly=True)

    # history
    create_date = fields.Datetime('Asked on', index=True, readonly=True)
    create_uid = fields.Many2one('res.users', string='Created by', index=True, readonly=True)
    write_date = fields.Datetime('Updated on', index=True, readonly=True)
    bump_date = fields.Datetime('Bumped on', readonly=True,
                                help="Technical field allowing to bump a question. Writing on this field will trigger "
                                     "a write on write_date and therefore bump the post. Directly writing on write_date "
                                     "is currently not supported and this field is a workaround.")
    write_uid = fields.Many2one('res.users', string='Updated by', index=True, readonly=True)
    relevancy = fields.Float('Relevance', compute="_compute_relevancy", store=True)

    # vote
    vote_ids = fields.One2many('forum.post.vote', 'post_id', string='Votes')
    user_vote = fields.Integer('My Vote', compute='_get_user_vote')
    vote_count = fields.Integer('Total Votes', compute='_get_vote_count', store=True)

    # favorite
    favourite_ids = fields.Many2many('res.users', string='Favourite')
    user_favourite = fields.Boolean('Is Favourite', compute='_get_user_favourite')
    favourite_count = fields.Integer('Favorite', compute='_get_favorite_count', store=True)

    # hierarchy
    is_correct = fields.Boolean('Correct', help='Correct answer or answer accepted')
    parent_id = fields.Many2one('forum.post', string='Question', ondelete='cascade', readonly=True, index=True)
    self_reply = fields.Boolean('Reply to own question', compute='_is_self_reply', store=True)
    child_ids = fields.One2many('forum.post', 'parent_id', string='Post Answers', domain=lambda self: [('forum_id', 'in', self.forum_id.ids)])
    child_count = fields.Integer('Answers', compute='_get_child_count', store=True)
    uid_has_answered = fields.Boolean('Has Answered', compute='_get_uid_has_answered')
    has_validated_answer = fields.Boolean('Is answered', compute='_get_has_validated_answer', store=True)

    # offensive moderation tools
    flag_user_id = fields.Many2one('res.users', string='Flagged by')
    moderator_id = fields.Many2one('res.users', string='Reviewed by', readonly=True)

    # closing
    closed_reason_id = fields.Many2one('forum.post.reason', string='Reason', copy=False)
    closed_uid = fields.Many2one('res.users', string='Closed by', index=True, readonly=True, copy=False)
    closed_date = fields.Datetime('Closed on', readonly=True, copy=False)

    # karma calculation and access
    karma_accept = fields.Integer('Convert comment to answer', compute='_get_post_karma_rights', compute_sudo=False)
    karma_edit = fields.Integer('Karma to edit', compute='_get_post_karma_rights', compute_sudo=False)
    karma_close = fields.Integer('Karma to close', compute='_get_post_karma_rights', compute_sudo=False)
    karma_unlink = fields.Integer('Karma to unlink', compute='_get_post_karma_rights', compute_sudo=False)
    karma_comment = fields.Integer('Karma to comment', compute='_get_post_karma_rights', compute_sudo=False)
    karma_comment_convert = fields.Integer('Karma to convert comment to answer', compute='_get_post_karma_rights', compute_sudo=False)
    karma_flag = fields.Integer('Flag a post as offensive', compute='_get_post_karma_rights', compute_sudo=False)
    can_ask = fields.Boolean('Can Ask', compute='_get_post_karma_rights', compute_sudo=False)
    can_answer = fields.Boolean('Can Answer', compute='_get_post_karma_rights', compute_sudo=False)
    can_accept = fields.Boolean('Can Accept', compute='_get_post_karma_rights', compute_sudo=False)
    can_edit = fields.Boolean('Can Edit', compute='_get_post_karma_rights', compute_sudo=False)
    can_close = fields.Boolean('Can Close', compute='_get_post_karma_rights', compute_sudo=False)
    can_unlink = fields.Boolean('Can Unlink', compute='_get_post_karma_rights', compute_sudo=False)
    can_upvote = fields.Boolean('Can Upvote', compute='_get_post_karma_rights', compute_sudo=False)
    can_downvote = fields.Boolean('Can Downvote', compute='_get_post_karma_rights', compute_sudo=False)
    can_comment = fields.Boolean('Can Comment', compute='_get_post_karma_rights', compute_sudo=False)
    can_comment_convert = fields.Boolean('Can Convert to Comment', compute='_get_post_karma_rights', compute_sudo=False)
    can_view = fields.Boolean('Can View', compute='_get_post_karma_rights', search='_search_can_view', compute_sudo=False)
    can_display_biography = fields.Boolean("Is the author's biography visible from his post", compute='_get_post_karma_rights', compute_sudo=False)
    can_post = fields.Boolean('Can Automatically be Validated', compute='_get_post_karma_rights', compute_sudo=False)
    can_flag = fields.Boolean('Can Flag', compute='_get_post_karma_rights', compute_sudo=False)
    can_moderate = fields.Boolean('Can Moderate', compute='_get_post_karma_rights', compute_sudo=False)

    def _search_can_view(self, operator, value):
        if operator not in ('=', '!=', '<>'):
            raise ValueError('Invalid operator: %s' % (operator,))

        if not value:
            operator = operator == "=" and '!=' or '='
            value = True

        user = self.env.user
        # Won't impact sitemap, search() in converter is forced as public user
        if self.env.is_admin():
            return [(1, '=', 1)]

        req = """
            SELECT p.id
            FROM forum_post p
                   LEFT JOIN res_users u ON p.create_uid = u.id
                   LEFT JOIN forum_forum f ON p.forum_id = f.id
            WHERE
                (p.create_uid = %s and f.karma_close_own <= %s)
                or (p.create_uid != %s and f.karma_close_all <= %s)
                or (
                    u.karma > 0
                    and (p.active or p.create_uid = %s)
                )
        """

        op = operator == "=" and "inselect" or "not inselect"

        # don't use param named because orm will add other param (test_active, ...)
        return [('id', op, (req, (user.id, user.karma, user.id, user.karma, user.id)))]

    @api.depends('content')
    def _get_plain_content(self):
        for post in self:
            post.plain_content = tools.html2plaintext(post.content)[0:500] if post.content else False

    @api.depends('vote_count', 'forum_id.relevancy_post_vote', 'forum_id.relevancy_time_decay')
    def _compute_relevancy(self):
        for post in self:
            if post.create_date:
                days = (datetime.today() - post.create_date).days
                post.relevancy = math.copysign(1, post.vote_count) * (abs(post.vote_count - 1) ** post.forum_id.relevancy_post_vote / (days + 2) ** post.forum_id.relevancy_time_decay)
            else:
                post.relevancy = 0

    def _get_user_vote(self):
        votes = self.env['forum.post.vote'].search_read([('post_id', 'in', self._ids), ('user_id', '=', self._uid)], ['vote', 'post_id'])
        mapped_vote = dict([(v['post_id'][0], v['vote']) for v in votes])
        for vote in self:
            vote.user_vote = mapped_vote.get(vote.id, 0)

    @api.depends('vote_ids.vote')
    def _get_vote_count(self):
        read_group_res = self.env['forum.post.vote'].read_group([('post_id', 'in', self._ids)], ['post_id', 'vote'], ['post_id', 'vote'], lazy=False)
        result = dict.fromkeys(self._ids, 0)
        for data in read_group_res:
            result[data['post_id'][0]] += data['__count'] * int(data['vote'])
        for post in self:
            post.vote_count = result[post.id]

    def _get_user_favourite(self):
        for post in self:
            post.user_favourite = post._uid in post.favourite_ids.ids

    @api.depends('favourite_ids')
    def _get_favorite_count(self):
        for post in self:
            post.favourite_count = len(post.favourite_ids)

    @api.depends('create_uid', 'parent_id')
    def _is_self_reply(self):
        for post in self:
            post.self_reply = post.parent_id.create_uid.id == post._uid

    @api.depends('child_ids')
    def _get_child_count(self):
        for post in self:
            post.child_count = len(post.child_ids)

    def _get_uid_has_answered(self):
        for post in self:
            post.uid_has_answered = post._uid in post.child_ids.create_uid.ids

    @api.depends('child_ids.is_correct')
    def _get_has_validated_answer(self):
        for post in self:
            post.has_validated_answer = any(answer.is_correct for answer in post.child_ids)

    @api.depends_context('uid')
    def _get_post_karma_rights(self):
        user = self.env.user
        is_admin = self.env.is_admin()
        # sudoed recordset instead of individual posts so values can be
        # prefetched in bulk
        for post, post_sudo in zip(self, self.sudo()):
            is_creator = post.create_uid == user

            post.karma_accept = post.forum_id.karma_answer_accept_own if post.parent_id.create_uid == user else post.forum_id.karma_answer_accept_all
            post.karma_edit = post.forum_id.karma_edit_own if is_creator else post.forum_id.karma_edit_all
            post.karma_close = post.forum_id.karma_close_own if is_creator else post.forum_id.karma_close_all
            post.karma_unlink = post.forum_id.karma_unlink_own if is_creator else post.forum_id.karma_unlink_all
            post.karma_comment = post.forum_id.karma_comment_own if is_creator else post.forum_id.karma_comment_all
            post.karma_comment_convert = post.forum_id.karma_comment_convert_own if is_creator else post.forum_id.karma_comment_convert_all
            post.karma_flag = post.forum_id.karma_flag

            post.can_ask = is_admin or user.karma >= post.forum_id.karma_ask
            post.can_answer = is_admin or user.karma >= post.forum_id.karma_answer
            post.can_accept = is_admin or user.karma >= post.karma_accept
            post.can_edit = is_admin or user.karma >= post.karma_edit
            post.can_close = is_admin or user.karma >= post.karma_close
            post.can_unlink = is_admin or user.karma >= post.karma_unlink
            post.can_upvote = is_admin or user.karma >= post.forum_id.karma_upvote or post.user_vote == -1
            post.can_downvote = is_admin or user.karma >= post.forum_id.karma_downvote or post.user_vote == 1
            post.can_comment = is_admin or user.karma >= post.karma_comment
            post.can_comment_convert = is_admin or user.karma >= post.karma_comment_convert
            post.can_view = is_admin or user.karma >= post.karma_close or (post_sudo.create_uid.karma > 0 and (post_sudo.active or post_sudo.create_uid == user))
            post.can_display_biography = is_admin or post_sudo.create_uid.karma >= post.forum_id.karma_user_bio
            post.can_post = is_admin or user.karma >= post.forum_id.karma_post
            post.can_flag = is_admin or user.karma >= post.forum_id.karma_flag
            post.can_moderate = is_admin or user.karma >= post.forum_id.karma_moderate

    def _update_content(self, content, forum_id):
        forum = self.env['forum.forum'].browse(forum_id)
        if content and self.env.user.karma < forum.karma_dofollow:
            for match in re.findall(r'<a\s.*href=".*?">', content):
                match = re.escape(match)  # replace parenthesis or special char in regex
                content = re.sub(match, match[:3] + 'rel="nofollow" ' + match[3:], content)

        if self.env.user.karma < forum.karma_editor:
            filter_regexp = r'(<img.*?>)|(<a[^>]*?href[^>]*?>)|(<[a-z|A-Z]+[^>]*style\s*=\s*[\'"][^\'"]*\s*background[^:]*:[^url;]*url)'
            content_match = re.search(filter_regexp, content, re.I)
            if content_match:
                raise AccessError(_('%d karma required to post an image or link.', forum.karma_editor))
        return content

    def _default_website_meta(self):
        res = super(Post, self)._default_website_meta()
        res['default_opengraph']['og:title'] = res['default_twitter']['twitter:title'] = self.name
        res['default_opengraph']['og:description'] = res['default_twitter']['twitter:description'] = self.plain_content
        res['default_opengraph']['og:image'] = res['default_twitter']['twitter:image'] = self.env['website'].image_url(self.create_uid, 'image_1024')
        res['default_twitter']['twitter:card'] = 'summary'
        res['default_meta_description'] = self.plain_content
        return res

    @api.constrains('parent_id')
    def _check_parent_id(self):
        if not self._check_recursion():
            raise ValidationError(_('You cannot create recursive forum posts.'))

    @api.model
    def create(self, vals):
        if 'content' in vals and vals.get('forum_id'):
            vals['content'] = self._update_content(vals['content'], vals['forum_id'])

        post = super(Post, self.with_context(mail_create_nolog=True)).create(vals)
        # deleted or closed questions
        if post.parent_id and (post.parent_id.state == 'close' or post.parent_id.active is False):
            raise UserError(_('Posting answer on a [Deleted] or [Closed] question is not possible.'))
        # karma-based access
        if not post.parent_id and not post.can_ask:
            raise AccessError(_('%d karma required to create a new question.', post.forum_id.karma_ask))
        elif post.parent_id and not post.can_answer:
            raise AccessError(_('%d karma required to answer a question.', post.forum_id.karma_answer))
        if not post.parent_id and not post.can_post:
            post.sudo().state = 'pending'

        # add karma for posting new questions
        if not post.parent_id and post.state == 'active':
            self.env.user.sudo().add_karma(post.forum_id.karma_gen_question_new)
        post.post_notification()
        return post

    @api.model
    def _get_mail_message_access(self, res_ids, operation, model_name=None):
        # XDO FIXME: to be correctly fixed with new _get_mail_message_access and filter access rule
        if operation in ('write', 'unlink') and (not model_name or model_name == 'forum.post'):
            # Make sure only author or moderator can edit/delete messages
            for post in self.browse(res_ids):
                if not post.can_edit:
                    raise AccessError(_('%d karma required to edit a post.', post.karma_edit))
        return super(Post, self)._get_mail_message_access(res_ids, operation, model_name=model_name)

    def write(self, vals):
        trusted_keys = ['active', 'is_correct', 'tag_ids']  # fields where security is checked manually
        if 'content' in vals:
            vals['content'] = self._update_content(vals['content'], self.forum_id.id)

        tag_ids = False
        if 'tag_ids' in vals:
            tag_ids = set(self.new({'tag_ids': vals['tag_ids']}).tag_ids.ids)

        for post in self:
            if 'state' in vals:
                if vals['state'] in ['active', 'close']:
                    if not post.can_close:
                        raise AccessError(_('%d karma required to close or reopen a post.', post.karma_close))
                    trusted_keys += ['state', 'closed_uid', 'closed_date', 'closed_reason_id']
                elif vals['state'] == 'flagged':
                    if not post.can_flag:
                        raise AccessError(_('%d karma required to flag a post.', post.forum_id.karma_flag))
                    trusted_keys += ['state', 'flag_user_id']
            if 'active' in vals:
                if not post.can_unlink:
                    raise AccessError(_('%d karma required to delete or reactivate a post.', post.karma_unlink))
            if 'is_correct' in vals:
                if not post.can_accept:
                    raise AccessError(_('%d karma required to accept or refuse an answer.', post.karma_accept))
                # update karma except for self-acceptance
                mult = 1 if vals['is_correct'] else -1
                if vals['is_correct'] != post.is_correct and post.create_uid.id != self._uid:
                    post.create_uid.sudo().add_karma(post.forum_id.karma_gen_answer_accepted * mult)
                    self.env.user.sudo().add_karma(post.forum_id.karma_gen_answer_accept * mult)
            if tag_ids:
                if set(post.tag_ids.ids) != tag_ids and self.env.user.karma < post.forum_id.karma_edit_retag:
                    raise AccessError(_('%d karma required to retag.', post.forum_id.karma_edit_retag))
            if any(key not in trusted_keys for key in vals) and not post.can_edit:
                raise AccessError(_('%d karma required to edit a post.', post.karma_edit))

        res = super(Post, self).write(vals)

        # if post content modify, notify followers
        if 'content' in vals or 'name' in vals:
            for post in self:
                if post.parent_id:
                    body, subtype_xmlid = _('Answer Edited'), 'website_forum.mt_answer_edit'
                    obj_id = post.parent_id
                else:
                    body, subtype_xmlid = _('Question Edited'), 'website_forum.mt_question_edit'
                    obj_id = post
                obj_id.message_post(body=body, subtype_xmlid=subtype_xmlid)
        if 'active' in vals:
            answers = self.env['forum.post'].with_context(active_test=False).search([('parent_id', 'in', self.ids)])
            if answers:
                answers.write({'active': vals['active']})
        return res

    def post_notification(self):
        for post in self:
            tag_partners = post.tag_ids.sudo().mapped('message_partner_ids')

            if post.state == 'active' and post.parent_id:
                post.parent_id.message_post_with_view(
                    'website_forum.forum_post_template_new_answer',
                    subject=_('Re: %s', post.parent_id.name),
                    partner_ids=[(4, p.id) for p in tag_partners],
                    subtype_id=self.env['ir.model.data'].xmlid_to_res_id('website_forum.mt_answer_new'))
            elif post.state == 'active' and not post.parent_id:
                post.message_post_with_view(
                    'website_forum.forum_post_template_new_question',
                    subject=post.name,
                    partner_ids=[(4, p.id) for p in tag_partners],
                    subtype_id=self.env['ir.model.data'].xmlid_to_res_id('website_forum.mt_question_new'))
            elif post.state == 'pending' and not post.parent_id:
                # TDE FIXME: in master, you should probably use a subtype;
                # however here we remove subtype but set partner_ids
                partners = post.sudo().message_partner_ids | tag_partners
                partners = partners.filtered(lambda partner: partner.user_ids and any(user.karma >= post.forum_id.karma_moderate for user in partner.user_ids))

                post.message_post_with_view(
                    'website_forum.forum_post_template_validation',
                    subject=post.name,
                    partner_ids=partners.ids,
                    subtype_id=self.env['ir.model.data'].xmlid_to_res_id('mail.mt_note'))
        return True

    def reopen(self):
        if any(post.parent_id or post.state != 'close' for post in self):
            return False

        reason_offensive = self.env.ref('website_forum.reason_7')
        reason_spam = self.env.ref('website_forum.reason_8')
        for post in self:
            if post.closed_reason_id in (reason_offensive, reason_spam):
                _logger.info('Upvoting user <%s>, reopening spam/offensive question',
                             post.create_uid)

                karma = post.forum_id.karma_gen_answer_flagged
                if post.closed_reason_id == reason_spam:
                    # If first post, increase the karma to add
                    count_post = post.search_count([('parent_id', '=', False), ('forum_id', '=', post.forum_id.id), ('create_uid', '=', post.create_uid.id)])
                    if count_post == 1:
                        karma *= 10
                post.create_uid.sudo().add_karma(karma * -1)

        self.sudo().write({'state': 'active'})

    def close(self, reason_id):
        if any(post.parent_id for post in self):
            return False

        reason_offensive = self.env.ref('website_forum.reason_7').id
        reason_spam = self.env.ref('website_forum.reason_8').id
        if reason_id in (reason_offensive, reason_spam):
            for post in self:
                _logger.info('Downvoting user <%s> for posting spam/offensive contents',
                             post.create_uid)
                karma = post.forum_id.karma_gen_answer_flagged
                if reason_id == reason_spam:
                    # If first post, increase the karma to remove
                    count_post = post.search_count([('parent_id', '=', False), ('forum_id', '=', post.forum_id.id), ('create_uid', '=', post.create_uid.id)])
                    if count_post == 1:
                        karma *= 10
                post.create_uid.sudo().add_karma(karma)

        self.write({
            'state': 'close',
            'closed_uid': self._uid,
            'closed_date': datetime.today().strftime(tools.DEFAULT_SERVER_DATETIME_FORMAT),
            'closed_reason_id': reason_id,
        })
        return True

    def validate(self):
        for post in self:
            if not post.can_moderate:
                raise AccessError(_('%d karma required to validate a post.', post.forum_id.karma_moderate))
            # if state == pending, no karma previously added for the new question
            if post.state == 'pending':
                post.create_uid.sudo().add_karma(post.forum_id.karma_gen_question_new)
            post.write({
                'state': 'active',
                'active': True,
                'moderator_id': self.env.user.id,
            })
            post.post_notification()
        return True

    def refuse(self):
        for post in self:
            if not post.can_moderate:
                raise AccessError(_('%d karma required to refuse a post.', post.forum_id.karma_moderate))
            post.moderator_id = self.env.user
        return True

    def flag(self):
        res = []
        for post in self:
            if not post.can_flag:
                raise AccessError(_('%d karma required to flag a post.', post.forum_id.karma_flag))
            if post.state == 'flagged':
               res.append({'error': 'post_already_flagged'})
            elif post.state == 'active':
                # TODO: potential performance bottleneck, can be batched
                post.write({
                    'state': 'flagged',
                    'flag_user_id': self.env.user.id,
                })
                res.append(
                    post.can_moderate and
                    {'success': 'post_flagged_moderator'} or
                    {'success': 'post_flagged_non_moderator'}
                )
            else:
                res.append({'error': 'post_non_flaggable'})
        return res

    def mark_as_offensive(self, reason_id):
        for post in self:
            if not post.can_moderate:
                raise AccessError(_('%d karma required to mark a post as offensive.', post.forum_id.karma_moderate))
            # remove some karma
            _logger.info('Downvoting user <%s> for posting spam/offensive contents', post.create_uid)
            post.create_uid.sudo().add_karma(post.forum_id.karma_gen_answer_flagged)
            # TODO: potential bottleneck, could be done in batch
            post.write({
                'state': 'offensive',
                'moderator_id': self.env.user.id,
                'closed_date': fields.Datetime.now(),
                'closed_reason_id': reason_id,
                'active': False,
            })
        return True

    def mark_as_offensive_batch(self, key, values):
        spams = self.browse()
        if key == 'create_uid':
            spams = self.filtered(lambda x: x.create_uid.id in values)
        elif key == 'country_id':
            spams = self.filtered(lambda x: x.create_uid.country_id.id in values)
        elif key == 'post_id':
            spams = self.filtered(lambda x: x.id in values)

        reason_id = self.env.ref('website_forum.reason_8').id
        _logger.info('User %s marked as spams (in batch): %s' % (self.env.uid, spams))
        return spams.mark_as_offensive(reason_id)

    def unlink(self):
        for post in self:
            if not post.can_unlink:
                raise AccessError(_('%d karma required to unlink a post.', post.karma_unlink))
        # if unlinking an answer with accepted answer: remove provided karma
        for post in self:
            if post.is_correct:
                post.create_uid.sudo().add_karma(post.forum_id.karma_gen_answer_accepted * -1)
                self.env.user.sudo().add_karma(post.forum_id.karma_gen_answer_accepted * -1)
        return super(Post, self).unlink()

    def bump(self):
        """ Bump a question: trigger a write_date by writing on a dummy bump_date
        field. One cannot bump a question more than once every 10 days. """
        self.ensure_one()
        if self.forum_id.allow_bump and not self.child_ids and (datetime.today() - datetime.strptime(self.write_date, tools.DEFAULT_SERVER_DATETIME_FORMAT)).days > 9:
            # write through super to bypass karma; sudo to allow public user to bump any post
            return self.sudo().write({'bump_date': fields.Datetime.now()})
        return False

    def vote(self, upvote=True):
        self.ensure_one()
        Vote = self.env['forum.post.vote']
        existing_vote = Vote.search([('post_id', '=', self.id), ('user_id', '=', self._uid)])
        new_vote_value = '1' if upvote else '-1'
        if existing_vote:
            if upvote:
                new_vote_value = '0' if existing_vote.vote == '-1' else '1'
            else:
                new_vote_value = '0' if existing_vote.vote == '1' else '-1'
            existing_vote.vote = new_vote_value
        else:
            Vote.create({'post_id': self.id, 'vote': new_vote_value})
        return {'vote_count': self.vote_count, 'user_vote': new_vote_value}

    def convert_answer_to_comment(self):
        """ Tools to convert an answer (forum.post) to a comment (mail.message).
        The original post is unlinked and a new comment is posted on the question
        using the post create_uid as the comment's author. """
        self.ensure_one()
        if not self.parent_id:
            return self.env['mail.message']

        # karma-based action check: use the post field that computed own/all value
        if not self.can_comment_convert:
            raise AccessError(_('%d karma required to convert an answer to a comment.', self.karma_comment_convert))

        # post the message
        question = self.parent_id
        self_sudo = self.sudo()
        values = {
            'author_id': self_sudo.create_uid.partner_id.id,  # use sudo here because of access to res.users model
            'email_from': self_sudo.create_uid.email_formatted,  # use sudo here because of access to res.users model
            'body': tools.html_sanitize(self.content, sanitize_attributes=True, strip_style=True, strip_classes=True),
            'message_type': 'comment',
            'subtype_xmlid': 'mail.mt_comment',
            'date': self.create_date,
        }
        # done with the author user to have create_uid correctly set
        new_message = question.with_user(self_sudo.create_uid.id).with_context(mail_create_nosubscribe=True).sudo().message_post(**values).sudo(False)

        # unlink the original answer, using SUPERUSER_ID to avoid karma issues
        self.sudo().unlink()

        return new_message

    @api.model
    def convert_comment_to_answer(self, message_id, default=None):
        """ Tool to convert a comment (mail.message) into an answer (forum.post).
        The original comment is unlinked and a new answer from the comment's author
        is created. Nothing is done if the comment's author already answered the
        question. """
        comment = self.env['mail.message'].sudo().browse(message_id)
        post = self.browse(comment.res_id)
        if not comment.author_id or not comment.author_id.user_ids:  # only comment posted by users can be converted
            return False

        # karma-based action check: must check the message's author to know if own / all
        is_author = comment.author_id.id == self.env.user.partner_id.id
        karma_own = post.forum_id.karma_comment_convert_own
        karma_all = post.forum_id.karma_comment_convert_all
        karma_convert = is_author and karma_own or karma_all
        can_convert = self.env.user.karma >= karma_convert
        if not can_convert:
            if is_author and karma_own < karma_all:
                raise AccessError(_('%d karma required to convert your comment to an answer.', karma_own))
            else:
                raise AccessError(_('%d karma required to convert a comment to an answer.', karma_all))

        # check the message's author has not already an answer
        question = post.parent_id if post.parent_id else post
        post_create_uid = comment.author_id.user_ids[0]
        if any(answer.create_uid.id == post_create_uid.id for answer in question.child_ids):
            return False

        # create the new post
        post_values = {
            'forum_id': question.forum_id.id,
            'content': comment.body,
            'parent_id': question.id,
            'name': _('Re: %s') % (question.name or ''),
        }
        # done with the author user to have create_uid correctly set
        new_post = self.with_user(post_create_uid).sudo().create(post_values).sudo(False)

        # delete comment
        comment.unlink()

        return new_post

    def unlink_comment(self, message_id):
        result = []
        for post in self:
            user = self.env.user
            comment = self.env['mail.message'].sudo().browse(message_id)
            if not comment.model == 'forum.post' or not comment.res_id == post.id:
                result.append(False)
                continue
            # karma-based action check: must check the message's author to know if own or all
            karma_unlink = (
                comment.author_id.id == user.partner_id.id and
                post.forum_id.karma_comment_unlink_own or post.forum_id.karma_comment_unlink_all
            )
            can_unlink = user.karma >= karma_unlink
            if not can_unlink:
                raise AccessError(_('%d karma required to unlink a comment.', karma_unlink))
            result.append(comment.unlink())
        return result

    def _set_viewed(self):
        self.ensure_one()
        return sql.increment_field_skiplock(self, 'views')

    def get_access_action(self, access_uid=None):
        """ Instead of the classic form view, redirect to the post on the website directly """
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'url': '/forum/%s/%s' % (self.forum_id.id, self.id),
            'target': 'self',
            'target_type': 'public',
            'res_id': self.id,
        }

    def _notify_get_groups(self, msg_vals=None):
        """ Add access button to everyone if the document is active. """
        groups = super(Post, self)._notify_get_groups(msg_vals=msg_vals)

        if self.state == 'active':
            for group_name, group_method, group_data in groups:
                group_data['has_button_access'] = True

        return groups

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, *, message_type='notification', **kwargs):
        if self.ids and message_type == 'comment':  # user comments have a restriction on karma
            # add followers of comments on the parent post
            if self.parent_id:
                partner_ids = kwargs.get('partner_ids', [])
                comment_subtype = self.sudo().env.ref('mail.mt_comment')
                question_followers = self.env['mail.followers'].sudo().search([
                    ('res_model', '=', self._name),
                    ('res_id', '=', self.parent_id.id),
                    ('partner_id', '!=', False),
                ]).filtered(lambda fol: comment_subtype in fol.subtype_ids).mapped('partner_id')
                partner_ids += question_followers.ids
                kwargs['partner_ids'] = partner_ids

            self.ensure_one()
            if not self.can_comment:
                raise AccessError(_('%d karma required to comment.', self.karma_comment))
            if not kwargs.get('record_name') and self.parent_id:
                kwargs['record_name'] = self.parent_id.name
        return super(Post, self).message_post(message_type=message_type, **kwargs)

    def _notify_record_by_inbox(self, message, recipients_data, msg_vals=False, **kwargs):
        """ Override to avoid keeping all notified recipients of a comment.
        We avoid tracking needaction on post comments. Only emails should be
        sufficient. """
        if msg_vals.get('message_type', message.message_type) == 'comment':
            return
        return super(Post, self)._notify_record_by_inbox(message, recipients_data, msg_vals=msg_vals, **kwargs)

    def _compute_website_url(self):
        return '/forum/{forum}/{post}{anchor}'.format(
            forum=slug(self.forum_id),
            post=slug(self),
            anchor=self.parent_id and '#answer_%d' % self.id or ''
        )

    def go_to_website(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'target': 'self',
            'url': self._compute_website_url(),
        }


class PostReason(models.Model):
    _name = "forum.post.reason"
    _description = "Post Closing Reason"
    _order = 'name'

    name = fields.Char(string='Closing Reason', required=True, translate=True)
    reason_type = fields.Selection([('basic', 'Basic'), ('offensive', 'Offensive')], string='Reason Type', default='basic')


class Vote(models.Model):
    _name = 'forum.post.vote'
    _description = 'Post Vote'
    _order = 'create_date desc, id desc'

    post_id = fields.Many2one('forum.post', string='Post', ondelete='cascade', required=True)
    user_id = fields.Many2one('res.users', string='User', required=True, default=lambda self: self._uid)
    vote = fields.Selection([('1', '1'), ('-1', '-1'), ('0', '0')], string='Vote', required=True, default='1')
    create_date = fields.Datetime('Create Date', index=True, readonly=True)
    forum_id = fields.Many2one('forum.forum', string='Forum', related="post_id.forum_id", store=True, readonly=False)
    recipient_id = fields.Many2one('res.users', string='To', related="post_id.create_uid", store=True, readonly=False)

    _sql_constraints = [
        ('vote_uniq', 'unique (post_id, user_id)', "Vote already exists !"),
    ]

    def _get_karma_value(self, old_vote, new_vote, up_karma, down_karma):
        _karma_upd = {
            '-1': {'-1': 0, '0': -1 * down_karma, '1': -1 * down_karma + up_karma},
            '0': {'-1': 1 * down_karma, '0': 0, '1': up_karma},
            '1': {'-1': -1 * up_karma + down_karma, '0': -1 * up_karma, '1': 0}
        }
        return _karma_upd[old_vote][new_vote]

    @api.model
    def create(self, vals):
        # can't modify owner of a vote
        if not self.env.is_admin():
            vals.pop('user_id', None)

        vote = super(Vote, self).create(vals)

        vote._check_general_rights()
        vote._check_karma_rights(vote.vote == '1')

        # karma update
        vote._vote_update_karma('0', vote.vote)
        return vote

    def write(self, values):
        # can't modify owner of a vote
        if not self.env.is_admin():
            values.pop('user_id', None)

        for vote in self:
            vote._check_general_rights(values)
            if 'vote' in values:
                if (values['vote'] == '1' or vote.vote == '-1' and values['vote'] == '0'):
                    upvote = True
                elif (values['vote'] == '-1' or vote.vote == '1' and values['vote'] == '0'):
                    upvote = False
                vote._check_karma_rights(upvote)

                # karma update
                vote._vote_update_karma(vote.vote, values['vote'])

        res = super(Vote, self).write(values)
        return res

    def _check_general_rights(self, vals={}):
        post = self.post_id
        if vals.get('post_id'):
            post = self.env['forum.post'].browse(vals.get('post_id'))
        if not self.env.is_admin():
            # own post check
            if self._uid == post.create_uid.id:
                raise UserError(_('It is not allowed to vote for its own post.'))
            # own vote check
            if self._uid != self.user_id.id:
                raise UserError(_('It is not allowed to modify someone else\'s vote.'))

    def _check_karma_rights(self, upvote=None):
        # karma check
        if upvote and not self.post_id.can_upvote:
            raise AccessError(_('%d karma required to upvote.', self.post_id.forum_id.karma_upvote))
        elif not upvote and not self.post_id.can_downvote:
            raise AccessError(_('%d karma required to downvote.', self.post_id.forum_id.karma_downvote))

    def _vote_update_karma(self, old_vote, new_vote):
        if self.post_id.parent_id:
            karma_value = self._get_karma_value(old_vote, new_vote, self.forum_id.karma_gen_answer_upvote, self.forum_id.karma_gen_answer_downvote)
        else:
            karma_value = self._get_karma_value(old_vote, new_vote, self.forum_id.karma_gen_question_upvote, self.forum_id.karma_gen_question_downvote)
        self.recipient_id.sudo().add_karma(karma_value)


class Tags(models.Model):
    _name = "forum.tag"
    _description = "Forum Tag"
    _inherit = ['mail.thread', 'website.seo.metadata']

    name = fields.Char('Name', required=True)
    forum_id = fields.Many2one('forum.forum', string='Forum', required=True)
    post_ids = fields.Many2many(
        'forum.post', 'forum_tag_rel', 'forum_tag_id', 'forum_id',
        string='Posts', domain=[('state', '=', 'active')])
    posts_count = fields.Integer('Number of Posts', compute='_get_posts_count', store=True)

    _sql_constraints = [
        ('name_uniq', 'unique (name, forum_id)', "Tag name already exists !"),
    ]

    @api.depends("post_ids", "post_ids.tag_ids", "post_ids.state", "post_ids.active")
    def _get_posts_count(self):
        for tag in self:
            tag.posts_count = len(tag.post_ids)  # state filter is in field domain

    @api.model
    def create(self, vals):
        forum = self.env['forum.forum'].browse(vals.get('forum_id'))
        if self.env.user.karma < forum.karma_tag_create:
            raise AccessError(_('%d karma required to create a new Tag.', forum.karma_tag_create))
        return super(Tags, self.with_context(mail_create_nolog=True, mail_create_nosubscribe=True)).create(vals)

```

## File: models\gamification.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class Challenge(models.Model):
    _inherit = 'gamification.challenge'

    challenge_category = fields.Selection(selection_add=[
        ('forum', 'Website / Forum')
    ], ondelete={'forum': 'set default'})

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Users(models.Model):
    _inherit = 'res.users'

    create_date = fields.Datetime('Create Date', readonly=True, index=True)
    forum_waiting_posts_count = fields.Integer('Waiting post', compute="_get_user_waiting_post")

    def _get_user_waiting_post(self):
        for user in self:
            Post = self.env['forum.post']
            domain = [('parent_id', '=', False), ('state', '=', 'pending'), ('create_uid', '=', user.id)]
            user.forum_waiting_posts_count = Post.search_count(domain)

    # Wrapper for call_kw with inherits
    def open_website_url(self):
        return self.mapped('partner_id').open_website_url()

    def get_gamification_redirection_data(self):
        res = super(Users, self).get_gamification_redirection_data()
        res.append({
            'url': '/forum',
            'label': 'See our Forum'
        })
        return res

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _
from odoo.addons.http_routing.models.ir_http import url_for


class Website(models.Model):
    _inherit = 'website'

    @api.model
    def get_default_forum_count(self):
        self.forums_count = self.env['forum.forum'].search_count(self.website_domain())

    forums_count = fields.Integer(readonly=True, default=get_default_forum_count)

    def get_suggested_controllers(self):
        suggested_controllers = super(Website, self).get_suggested_controllers()
        suggested_controllers.append((_('Forum'), url_for('/forum'), 'website_forum'))
        return suggested_controllers

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import forum
from . import gamification
from . import res_users
from . import website

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_forum_forum,forum.forum,model_forum_forum,,1,0,0,0
access_forum_forum_manager,forum.forum.maanger,model_forum_forum,base.group_erp_manager,1,1,1,1
access_forum_post_public,forum.post.public,model_forum_post,base.group_public,1,0,0,0
access_forum_post_portal,forum.post.portal,model_forum_post,base.group_portal,1,1,1,1
access_forum_post_user,forum.post.user,model_forum_post,base.group_user,1,1,1,1
access_forum_post_vote_public,forum.post.vote.public,model_forum_post_vote,base.group_public,1,0,0,0
access_forum_post_vote_portal,orum.post.vote.portal,model_forum_post_vote,base.group_portal,1,1,1,0
access_forum_post_vote_user,forum.post.vote.user,model_forum_post_vote,base.group_user,1,1,1,1
access_forum_post_reason_public,forum.post.reason.public,model_forum_post_reason,base.group_public,1,0,0,0
access_forum_post_reason_portal,forum.post.reason.portal,model_forum_post_reason,base.group_portal,1,0,0,0
access_forum_post_reason_user,forum.post.reason.user,model_forum_post_reason,base.group_user,1,1,1,1
access_forum_tag_public,forum.tag.public,model_forum_tag,base.group_public,1,0,1,0
access_forum_tag_portal,forum.tag.portal,model_forum_tag,base.group_portal,1,0,1,0
access_forum_tag_user,forum.tag.user,model_forum_tag,base.group_user,1,1,1,1

```

## File: security\website_forum_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="website_forum_public" model="ir.rule">
        <field name="name">Website forum: Public user can only access to public forum</field>
        <field name="model_id" ref="model_forum_forum"/>
        <field name="domain_force">[('privacy', '=', 'public')]</field>
        <field name="groups" eval="[(4, ref('base.group_public'))]"/>
    </record>
    <record id="website_forum_connected" model="ir.rule">
        <field name="name">Website forum: User can only access to public (or authorized) forum</field>
        <field name="model_id" ref="model_forum_forum"/>
        <field name="domain_force">[
            '|',
                ('privacy', 'in', ['public', 'connected']),
                '&amp;',
                    ('privacy', '=', 'private'),
                    ('authorized_group_id', 'in', user.groups_id.ids)]</field>
        <field name="groups" eval="[(4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
    </record>
    <record id="website_forum_create_website_designer" model="ir.rule">
        <field name="name">Website forum: Website designer can create private forum</field>
        <field name="model_id" ref="model_forum_forum"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('website.group_website_designer'))]"/>
        <field name="perm_unlink" eval="0"/>
        <field name="perm_write" eval="0"/>
        <field name="perm_read" eval="0"/>
        <field name="perm_create" eval="1"/>
    </record>
    <record id="website_forum_private" model="ir.rule">
        <field name="name">Website forum: All access for manager</field>
        <field name="model_id" ref="model_forum_forum"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('base.group_erp_manager'))]"/>
    </record>

    <record id="website_forum_public_post" model="ir.rule">
        <field name="name">Website forum post: Public user can only access to public post</field>
        <field name="model_id" ref="model_forum_post"/>
        <field name="domain_force">[('forum_id.privacy', '=', 'public')]</field>
        <field name="groups" eval="[(4, ref('base.group_public'))]"/>
    </record>
    <record id="website_forum_connected_post" model="ir.rule">
        <field name="name">Website forum post: User can only access to public (or authorized) post</field>
        <field name="model_id" ref="model_forum_post"/>
        <field name="domain_force">['|', ('forum_id.privacy', 'in', ['public', 'connected']), '&amp;', ('forum_id.privacy', '=', 'private'), ('forum_id.authorized_group_id', 'in', user.groups_id.ids)]</field>
        <field name="groups" eval="[(4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
    </record>
    <record id="website_forum_private_post" model="ir.rule">
        <field name="name">Website forum post : All access for manager</field>
        <field name="model_id" ref="model_forum_post"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('base.group_erp_manager'))]"/>
    </record>

    <record id="website_forum_public_tag" model="ir.rule">
        <field name="name">Website forum tag: Public user can only access to tag linked to public forum</field>
        <field name="model_id" ref="model_forum_tag"/>
        <field name="domain_force">[('forum_id.privacy', '=', 'public')]</field>
        <field name="groups" eval="[(4, ref('base.group_public'))]"/>
    </record>
    <record id="website_forum_connected_tag" model="ir.rule">
        <field name="name">Website forum tag: User can only access to tag linked to public (or authorized) forum</field>
        <field name="model_id" ref="model_forum_tag"/>
        <field name="domain_force">['|', ('forum_id.privacy', 'in', ['public', 'connected']), '&amp;', ('forum_id.privacy', '=', 'private'), ('forum_id.authorized_group_id', 'in', user.groups_id.ids)]</field>
        <field name="groups" eval="[(4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
    </record>
    <record id="website_forum_private_tag" model="ir.rule">
        <field name="name">Website forum tag : Manager user can access to all tags</field>
        <field name="model_id" ref="model_forum_tag"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('base.group_erp_manager'))]"/>
    </record>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="98.616%"><stop offset="0%" stop-color="#797C79"/><stop offset="100%" stop-color="#545554"/></linearGradient><path id="d" d="M30.37 44.533c-2.72 0-5.282-.495-7.533-1.37-2.27 1.799-5.054 2.894-8.013 3.209a.598.598 0 0 1-.644-.44c-.07-.291.151-.47.37-.682 1.087-1.058 2.404-1.89 2.92-5.442-2.073-2.01-3.303-4.52-3.303-7.244 0-6.612 7.255-11.97 16.203-11.97 8.949 0 16.204 5.358 16.204 11.97 0 6.616-7.255 11.97-16.204 11.97zm-.37-19.2a4.167 4.167 0 1 0 0 8.334 4.167 4.167 0 0 0 0-8.334zm4.773 8.694l-1.857-.465c-1.951 1.404-4.316 1.09-5.832 0l-1.857.465a2.5 2.5 0 0 0-1.894 2.425v.965c0 .69.56 1.25 1.25 1.25h10.834c.69 0 1.25-.56 1.25-1.25v-.965a2.5 2.5 0 0 0-1.894-2.425zm20.703 18.356c-1.008-.961-2.231-1.717-2.71-4.947 4.978-4.729 3.775-11.494-2.725-15.166.003.098.005.196.005.294 0 8.971-9.374 15.849-20.562 15.47 2.758 2.268 6.799 3.698 11.303 3.698 2.526 0 4.905-.45 6.995-1.245 2.108 1.635 4.693 2.63 7.44 2.916a.556.556 0 0 0 .599-.4c.066-.264-.14-.426-.345-.62z"/><path id="e" d="M30.37 42.533c-2.72 0-5.282-.495-7.533-1.37-2.27 1.799-5.054 2.894-8.013 3.209a.598.598 0 0 1-.644-.44c-.07-.291.151-.47.37-.682 1.087-1.058 2.404-1.89 2.92-5.442-2.073-2.01-3.303-4.52-3.303-7.244 0-6.612 7.255-11.97 16.203-11.97 8.949 0 16.204 5.358 16.204 11.97 0 6.616-7.255 11.97-16.204 11.97zm-.37-19.2a4.167 4.167 0 1 0 0 8.334 4.167 4.167 0 0 0 0-8.334zm4.773 8.694l-1.857-.465c-1.951 1.404-4.316 1.09-5.832 0l-1.857.465a2.5 2.5 0 0 0-1.894 2.425v.965c0 .69.56 1.25 1.25 1.25h10.834c.69 0 1.25-.56 1.25-1.25v-.965a2.5 2.5 0 0 0-1.894-2.425zm20.703 18.356c-1.008-.961-2.231-1.717-2.71-4.947 4.978-4.729 3.775-11.494-2.725-15.166.003.098.005.196.005.294 0 8.971-9.374 15.849-20.562 15.47 2.758 2.268 6.799 3.698 11.303 3.698 2.526 0 4.905-.45 6.995-1.245 2.108 1.635 4.693 2.63 7.44 2.916a.556.556 0 0 0 .599-.4c.066-.264-.14-.426-.345-.62z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M43.795 69H4c-2 0-4-.146-4-4.082v-22.25l16-16.525 7-6.123L33 19l9 4.082 3.451 6.272L50 31.244 49 44.51l7 7.143L43.795 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\website_forum.editor.js

```javascript
odoo.define('website_forum.editor', function (require) {
"use strict";

var core = require('web.core');
var WebsiteNewMenu = require('website.newMenu');
var Dialog = require('web.Dialog');

var _t = core._t;

var ForumCreateDialog = Dialog.extend({
    xmlDependencies: Dialog.prototype.xmlDependencies.concat(
        ['/website_forum/static/src/xml/website_forum_templates.xml']
    ),
    template: 'website_forum.add_new_forum',
    events: _.extend({}, Dialog.prototype.events, {
        'change input[name="privacy"]': '_onPrivacyChanged',
    }),

    /**
     * @override
     * @param {Object} parent
     * @param {Object} options
     */
    init: function (parent, options) {
        options = _.defaults(options || {}, {
            title: _t("New Forum"),
            size: 'medium',
            buttons: [
                {
                    text: _t("Create"),
                    classes: 'btn-primary',
                    click: this.onCreateClick.bind(this),
                },
                {
                    text: _t("Discard"),
                    close: true
                },
            ]
        });
        this._super(parent, options);
    },
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            var $input = self.$('#group_id');
            $input.select2({
                width: '100%',
                allowClear: true,
                formatNoMatches: false,
                multiple: false,
                selection_data: false,
                fill_data: function (query, data) {
                    var that = this;
                    var tags = {results: []};
                    _.each(data, function (obj) {
                        if (that.matcher(query.term, obj.display_name)) {
                            tags.results.push({id: obj.id, text: obj.display_name});
                        }
                    });
                    query.callback(tags);
                },
                query: function (query) {
                    var that = this;
                    // fetch data only once and store it
                    if (!this.selection_data) {
                        self._rpc({
                            model: 'res.groups',
                            method: 'search_read',
                            args: [[], ['display_name']],
                        }).then(function (data) {
                            that.fill_data(query, data);
                            that.selection_data = data;
                        });
                    } else {
                        this.fill_data(query, this.selection_data);
                    }
                }
            });
        });
    },
    onCreateClick: function () {
        var $dialog = this.$el;
        var $forumName = $dialog.find('input[name=forum_name]');
        if (!$forumName.val()) {
            $forumName.addClass('border-danger');
            return;
        }
        var $forumPrivacyGroup = $dialog.find('input[name=group_id]');
        var forumPrivacy = $dialog.find('input:radio[name=privacy]:checked').val();
        if (forumPrivacy === 'private' && !$forumPrivacyGroup.val()) {
            this.$("#group-required").removeClass('d-none');
            return;
        }
        var addMenu = ($dialog.find('input[type="checkbox"]').is(':checked'));
        var forumMode = $dialog.find('input:radio[name=mode]:checked').val();
        return this._rpc({
            route: '/forum/new',
            params: {
                forum_name: $forumName.val(),
                forum_mode: forumMode,
                forum_privacy: forumPrivacy,
                forum_privacy_group: $forumPrivacyGroup.val(),
                add_menu: addMenu || "",
            },
        }).then(function (url) {
            window.location.href = url;
            return new Promise(function () {});
        });
    },
    /**
     * @private
     */
    _onPrivacyChanged: function (ev) {
        this.$('.show_visibility_group').toggleClass('d-none', ev.target.value !== 'private');
    },
});

WebsiteNewMenu.include({
    actions: _.extend({}, WebsiteNewMenu.prototype.actions || {}, {
        new_forum: '_createNewForum',
    }),

    //--------------------------------------------------------------------------
    // Actions
    //--------------------------------------------------------------------------

    /**
     * Asks the user information about a new forum to create, then creates it
     * and redirects the user to this new forum.
     *
     * @private
     * @returns {Promise} Unresolved if there is a redirection
     */
    _createNewForum: function () {
        var self = this;
        var def = new Promise(function (resolve) {
            var dialog = new ForumCreateDialog(self, {});
            dialog.open();
            dialog.on('closed', self, resolve);
        });
        return def;
    },
});
});

```

## File: static\src\js\website_forum.js

```javascript
odoo.define('website_forum.website_forum', function (require) {
'use strict';

const dom = require('web.dom');
var core = require('web.core');
var weDefaultOptions = require('web_editor.wysiwyg.default_options');
var wysiwygLoader = require('web_editor.loader');
var publicWidget = require('web.public.widget');
var session = require('web.session');
var qweb = core.qweb;

var _t = core._t;

publicWidget.registry.websiteForum = publicWidget.Widget.extend({
    selector: '.website_forum',
    xmlDependencies: ['/website_forum/static/src/xml/website_forum_share_templates.xml'],
    events: {
        'click .karma_required': '_onKarmaRequiredClick',
        'mouseenter .o_js_forum_tag_follow': '_onTagFollowBoxMouseEnter',
        'mouseleave .o_js_forum_tag_follow': '_onTagFollowBoxMouseLeave',
        'mouseenter .o_forum_user_info': '_onUserInfoMouseEnter',
        'mouseleave .o_forum_user_info': '_onUserInfoMouseLeave',
        'mouseleave .o_forum_user_bio_expand': '_onUserBioExpandMouseLeave',
        'click .flag:not(.karma_required)': '_onFlagAlertClick',
        'click .vote_up:not(.karma_required), .vote_down:not(.karma_required)': '_onVotePostClick',
        'click .o_js_validation_queue a[href*="/validate"]': '_onValidationQueueClick',
        'click .o_wforum_validate_toggler:not(.karma_required)': '_onAcceptAnswerClick',
        'click .o_wforum_favourite_toggle': '_onFavoriteQuestionClick',
        'click .comment_delete': '_onDeleteCommentClick',
        'click .js_close_intro': '_onCloseIntroClick',
        'submit .js_wforum_submit_form:has(:not(.karma_required).o_wforum_submit_post)': '_onSubmitForm',
    },

    /**
     * @override
     */
    start: function () {
        var self = this;

        this.lastsearch = [];

        // float-left class messes up the post layout OPW 769721
        $('span[data-oe-model="forum.post"][data-oe-field="content"]').find('img.float-left').removeClass('float-left');

        // welcome message action button
        var forumLogin = _.string.sprintf('%s/web?redirect=%s',
            window.location.origin,
            encodeURIComponent(window.location.href)
        );
        $('.forum_register_url').attr('href', forumLogin);

        // Initialize forum's tooltips
        this.$('[data-toggle="tooltip"]').tooltip({delay: 0});
        this.$('[data-toggle="popover"]').popover({offset: 8});

        $('input.js_select2').select2({
            tags: true,
            tokenSeparators: [',', ' ', '_'],
            maximumInputLength: 35,
            minimumInputLength: 2,
            maximumSelectionSize: 5,
            lastsearch: [],
            createSearchChoice: function (term) {
                if (_.filter(self.lastsearch, function (s) {
                    return s.text.localeCompare(term) === 0;
                }).length === 0) {
                    //check Karma
                    if (parseInt($('#karma').val()) >= parseInt($('#karma_edit_retag').val())) {
                        return {
                            id: '_' + $.trim(term),
                            text: $.trim(term) + ' *',
                            isNew: true,
                        };
                    }
                }
            },
            formatResult: function (term) {
                if (term.isNew) {
                    return '<span class="badge badge-primary">New</span> ' + _.escape(term.text);
                } else {
                    return _.escape(term.text);
                }
            },
            ajax: {
                url: '/forum/get_tags',
                dataType: 'json',
                data: function (term) {
                    return {
                        query: term,
                        limit: 50,
                        forum_id: $('#wrapwrap').data('forum_id'),
                    };
                },
                results: function (data) {
                    var ret = [];
                    _.each(data, function (x) {
                        ret.push({
                            id: x.id,
                            text: x.name,
                            isNew: false,
                        });
                    });
                    self.lastsearch = ret;
                    return {results: ret};
                }
            },
            // Take default tags from the input value
            initSelection: function (element, callback) {
                var data = [];
                _.each(element.data('init-value'), function (x) {
                    data.push({id: x.id, text: x.name, isNew: false});
                });
                element.val('');
                callback(data);
            },
        });

        _.each($('textarea.o_wysiwyg_loader'), function (textarea) {
            var $textarea = $(textarea);
            var editorKarma = $textarea.data('karma') || 0; // default value for backward compatibility
            var $form = $textarea.closest('form');
            var hasFullEdit = parseInt($("#karma").val()) >= editorKarma;
            // Warning: Do not activate any option that adds inline style.
            // Because the style is deleted after save.
            var toolbar = [
                ['style', ['style']],
                ['font', ['bold', 'italic', 'underline', 'clear']],
                ['para', ['ul', 'ol', 'paragraph']],
                ['table', ['table']],
            ];
            if (hasFullEdit) {
                toolbar.push(['insert', ['link', 'picture']]);
            }
            toolbar.push(['history', ['undo', 'redo']]);

            var options = {
                height: 350,
                minHeight: 80,
                toolbar: toolbar,
                styleWithSpan: false,
                styleTags: _.without(weDefaultOptions.styleTags, 'h1', 'h2', 'h3'),
                recordInfo: {
                    context: self._getContext(),
                    res_model: 'forum.post',
                    res_id: +window.location.pathname.split('-').pop(),
                },
                disableFullMediaDialog: true,
                disableResizeImage: true,
            };
            if (!hasFullEdit) {
                options.plugins = {
                    LinkPlugin: false,
                    MediaPlugin: false,
                };
            }
            wysiwygLoader.load(self, $textarea[0], options).then(wysiwyg => {
                // float-left class messes up the post layout OPW 769721
                $form.find('.note-editable').find('img.float-left').removeClass('float-left');
                // o_we_selected_image has not always been removed when
                // saving a post so we need the line below to remove it if it is present.
                $form.find('.note-editable').find('img.o_we_selected_image').removeClass('o_we_selected_image');
                $form.on('click', 'button, .a-submit', () => {
                    $form.find('.note-editable').find('img.o_we_selected_image').removeClass('o_we_selected_image');
                    wysiwyg.save();
                });
            });
        });

        _.each(this.$('.o_wforum_bio_popover'), authorBox => {
            $(authorBox).popover({
                trigger: 'hover',
                offset: 10,
                animation: false,
                html: true,
            }).popover('hide').data('bs.popover').tip.classList.add('o_wforum_bio_popover_container');
        });

        this.$('#post_reply').on('shown.bs.collapse', function (e) {
            const replyEl = document.querySelector('#post_reply');
            const scrollingElement = dom.closestScrollable(replyEl.parentNode);
            dom.scrollTo(replyEl, {
                forcedOffset: $(scrollingElement).innerHeight() - $(replyEl).innerHeight(),
            });
        });

        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     *
     * @override
     * @param {Event} ev
     */
    _onSubmitForm: function (ev) {
        let validForm = true;

        let $form = $(ev.currentTarget);
        let $title = $form.find('input[name=post_name]');
        let $textarea = $form.find('textarea[name=content]');
        // It's not really in the textarea that the user write at first
        const fillableTextAreaEl = ev.currentTarget
            .querySelector(".o_wysiwyg_wrapper .note-editable.panel-body");
        const isTextAreaFilled = fillableTextAreaEl &&
            (fillableTextAreaEl.innerText.trim() || fillableTextAreaEl.querySelector("img"));

        if ($title.length && $title[0].required) {
            if ($title.val()) {
                $title.removeClass('is-invalid');
            } else {
                $title.addClass('is-invalid');
                validForm = false;
            }
        }

        // Because the textarea is hidden, we add the red or green border to its container
        if ($textarea[0] && $textarea[0].required) {
            let $textareaContainer = $form.find('.o_wysiwyg_wrapper .note-editor.panel.panel-default');
            if (!isTextAreaFilled) {
                $textareaContainer.addClass('border border-danger rounded-top');
                validForm = false;
            } else {
                $textareaContainer.removeClass('border border-danger rounded-top');
            }
        }

        if (validForm) {
            // Stores social share data to display modal on next page.
            if ($form.has('.oe_social_share_call').length) {
                sessionStorage.setItem('social_share', JSON.stringify({
                    targetType: $(ev.currentTarget).find('.o_wforum_submit_post').data('social-target-type'),
                }));
            }
        } else {
            ev.preventDefault();
            setTimeout(function() {
                var $buttons = $(ev.currentTarget).find('button[type="submit"], a.a-submit');
                _.each($buttons, function (btn) {
                    let $btn = $(btn);
                    $btn.find('i').remove();
                    $btn.prop('disabled', false);
                });
            }, 0);
        }
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onKarmaRequiredClick: function (ev) {
        var $karma = $(ev.currentTarget);
        var karma = $karma.data('karma');
        var forum_id = $('#wrapwrap').data('forum_id');
        if (!karma) {
            return;
        }
        ev.preventDefault();
        var msg = karma + ' ' + _t("karma is required to perform this action. ");
        var title = _t("Karma Error");
        if (forum_id) {
            msg += '<a class="alert-link" href="/forum/' + forum_id + '/faq">' + _t("Read the guidelines to know how to gain karma.") + '</a>';
        }
        if (session.is_website_user) {
            msg = _t("Sorry you must be logged in to perform this action");
            title = _t("Access Denied");
        }
        this.call('crash_manager', 'show_warning', {
            message: msg,
            title: title,
        }, {
            sticky: false,
        });
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onTagFollowBoxMouseEnter: function (ev) {
        $(ev.currentTarget).find('.o_forum_tag_follow_box').stop().fadeIn().css('display', 'block');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onTagFollowBoxMouseLeave: function (ev) {
        $(ev.currentTarget).find('.o_forum_tag_follow_box').stop().fadeOut().css('display', 'none');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onUserInfoMouseEnter: function (ev) {
        $(ev.currentTarget).parent().find('.o_forum_user_bio_expand').delay(500).toggle('fast');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onUserInfoMouseLeave: function (ev) {
        $(ev.currentTarget).parent().find('.o_forum_user_bio_expand').clearQueue();
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onUserBioExpandMouseLeave: function (ev) {
        $(ev.currentTarget).fadeOut('fast');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onFlagAlertClick: function (ev) {
        var self = this;
        ev.preventDefault();
        var $link = $(ev.currentTarget);
        this._rpc({
            route: $link.data('href') || ($link.attr('href') !== '#' && $link.attr('href')) || $link.closest('form').attr('action'),
        }).then(function (data) {
            if (data.error) {
                var message;
                if (data.error === 'anonymous_user') {
                    message = _t("Sorry you must be logged to flag a post");
                } else if (data.error === 'post_already_flagged') {
                    message = _t("This post is already flagged");
                } else if (data.error === 'post_non_flaggable') {
                    message = _t("This post can not be flagged");
                }
                self.call('crash_manager', 'show_warning', {
                    message: message,
                    title: _t("Access Denied"),
                }, {
                    sticky: false,
                });
            } else if (data.success) {
                var elem = $link;
                if (data.success === 'post_flagged_moderator') {
                    elem.data('href') && elem.html(' Flagged');
                    var c = parseInt($('#count_flagged_posts').html(), 10);
                    c++;
                    $('#count_flagged_posts').html(c);
                } else if (data.success === 'post_flagged_non_moderator') {
                    elem.data('href') && elem.html(' Flagged');
                    var forumAnswer = elem.closest('.forum_answer');
                    forumAnswer.fadeIn(1000);
                    forumAnswer.slideUp(1000);
                }
            }
        });
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onVotePostClick: function (ev) {
        var self = this;
        ev.preventDefault();
        var $btn = $(ev.currentTarget);
        this._rpc({
            route: $btn.data('href'),
        }).then(function (data) {
            if (data.error) {
                var message;
                if (data.error === 'own_post') {
                    message = _t('Sorry, you cannot vote for your own posts');
                } else if (data.error === 'anonymous_user') {
                    message = _t('Sorry you must be logged to vote');
                }
                self.call('crash_manager', 'show_warning', {
                    message: message,
                    title: _t("Access Denied"),
                }, {
                    sticky: false,
                });
            } else {
                var $container = $btn.closest('.vote');
                var $items = $container.children();
                var $voteUp = $items.filter('.vote_up');
                var $voteDown = $items.filter('.vote_down');
                var $voteCount = $items.filter('.vote_count');
                var userVote = parseInt(data['user_vote']);

                $voteUp.prop('disabled', userVote === 1);
                $voteDown.prop('disabled', userVote === -1);

                $items.removeClass('text-success text-danger text-muted o_forum_vote_animate');
                void $container[0].offsetWidth; // Force a refresh

                if (userVote === 1) {
                    $voteUp.addClass('text-success');
                    $voteCount.addClass('text-success');
                    $voteDown.removeClass('karma_required');
                }
                if (userVote === -1) {
                    $voteDown.addClass('text-danger');
                    $voteCount.addClass('text-danger');
                    $voteUp.removeClass('karma_required');
                }
                if (userVote === 0) {
                    if (!$voteDown.data('can-downvote')) {
                        $voteDown.addClass('karma_required');
                    }
                    if (!$voteUp.data('can-upvote')) {
                        $voteUp.addClass('karma_required');
                    }
                }
                $voteCount.html(data['vote_count']).addClass('o_forum_vote_animate');
            }
        });
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onValidationQueueClick: function (ev) {
        ev.preventDefault();
        var $link = $(ev.currentTarget);
        $link.parents('.post_to_validate').hide();
        $.get($link.attr('href')).then(() => {
            var left = $('.o_js_validation_queue:visible').length;
            var type = $('h2.o_page_header a.active').data('type');
            $('#count_post').text(left);
            $('#moderation_tools a[href*="/' + type + '_"]').find('strong').text(left);
            if (!left) {
                this.$('.o_caught_up_alert').removeClass('d-none');
            }
        }, function () {
            $link.parents('.o_js_validation_queue > div').addClass('bg-danger text-white').css('background-color', '#FAA');
            $link.parents('.post_to_validate').show();
        });
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onAcceptAnswerClick: function (ev) {
        ev.preventDefault();
        var $link = $(ev.currentTarget);
        var target = $link.data('target');

        this._rpc({
            route: $link.data('href'),
        }).then(data => {
            if (data.error) {
                if (data.error === 'anonymous_user') {
                    var message = _t("Sorry, anonymous users cannot choose correct answer.");
                }
                this.call('crash_manager', 'show_warning', {
                    message: message,
                    title: _t("Access Denied"),
                }, {
                    sticky: false,
                });
            } else {
                _.each(this.$('.forum_answer'), answer => {
                    var $answer = $(answer);
                    var isCorrect = $answer.is(target) ? data : false;
                    var $toggler = $answer.find('.o_wforum_validate_toggler');
                    var newHelper = isCorrect ? $toggler.data('helper-decline') : $toggler.data('helper-accept');

                    $answer.toggleClass('o_wforum_answer_correct', isCorrect);
                    $toggler.tooltip('dispose')
                            .attr('data-original-title', newHelper)
                            .tooltip({delay: 0});
                });
            }
        });
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onFavoriteQuestionClick: function (ev) {
        ev.preventDefault();
        var $link = $(ev.currentTarget);
        this._rpc({
            route: $link.data('href'),
        }).then(function (data) {
            $link.toggleClass('o_wforum_gold fa-star', data)
                 .toggleClass('fa-star-o text-muted', !data);
        });
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onDeleteCommentClick: function (ev) {
        ev.preventDefault();
        var $link = $(ev.currentTarget);
        var $container = $link.closest('.o_wforum_post_comments_container');

        this._rpc({
            route: $link.closest('form').attr('action'),
        }).then(function () {
            $link.closest('.o_wforum_post_comment').remove();

            var count = $container.find('.o_wforum_post_comment').length;
            if (count) {
                $container.find('.o_wforum_comments_count').text(count);
            } else {
                $container.find('.o_wforum_comments_count_header').remove();
            }
        });
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onCloseIntroClick: function (ev) {
        ev.preventDefault();
        document.cookie = 'forum_welcome_message = false';
        $('.forum_intro').slideUp();
        return true;
    },
});

publicWidget.registry.websiteForumSpam = publicWidget.Widget.extend({
    selector: '.o_wforum_moderation_queue',
    xmlDependencies: ['/website_forum/static/src/xml/website_forum_share_templates.xml'],
    events: {
        'click .o_wforum_select_all_spam': '_onSelectallSpamClick',
        'click .o_wforum_mark_spam': 'async _onMarkSpamClick',
        'input #spamSearch': '_onSpamSearchInput',
    },

    /**
     * @override
     */
    start: function () {
        this.spamIDs = this.$('.modal').data('spam-ids');
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onSelectallSpamClick: function (ev) {
        var $spamInput = this.$('.modal .tab-pane.active input');
        $spamInput.prop('checked', true);
    },

    /**
     * @private
     * @param {Event} ev
     */
    _onSpamSearchInput: function (ev) {
        var self = this;
        var toSearch = $(ev.currentTarget).val();
        return this._rpc({
            model: 'forum.post',
            method: 'search_read',
            args: [
                [['id', 'in', self.spamIDs],
                    '|',
                    ['name', 'ilike', toSearch],
                    ['content', 'ilike', toSearch]],
                ['name', 'content']
            ],
            kwargs: {}
        }).then(function (o) {
            _.each(o, function (r) {
                r.content = $('<p>' + $(r.content).html() + '</p>').text().substring(0, 250);
            });
            self.$('div.post_spam').html(qweb.render('website_forum.spam_search_name', {
                posts: o,
            }));
        });
    },

    /**
     * @private
     * @param {Event} ev
     */
    _onMarkSpamClick: function (ev) {
        var key = this.$('.modal .tab-pane.active').data('key');
        var $inputs = this.$('.modal .tab-pane.active input.custom-control-input:checked');
        var values = _.map($inputs, function (o) {
            return parseInt(o.value);
        });
        return this._rpc({model: 'forum.post',
            method: 'mark_as_offensive_batch',
            args: [this.spamIDs, key, values],
        }).then(function () {
            window.location.reload();
        });
    },
});

publicWidget.registry.WebsiteForumBackButton = publicWidget.Widget.extend({
    selector: '.o_back_button',
    events: {
        'click': '_onBackButtonClick',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onBackButtonClick() {
        window.history.back();
    },
});

});

```

## File: static\src\js\website_forum.share.js

```javascript
odoo.define('website_forum.share', function (require) {
'use strict';

var core = require('web.core');
var publicWidget = require('web.public.widget');

var qweb = core.qweb;

// FIXME There is no reason to inherit from socialShare here
var ForumShare = publicWidget.registry.socialShare.extend({
    selector: '',
    xmlDependencies: publicWidget.registry.socialShare.prototype.xmlDependencies
        .concat(['/website_forum/static/src/xml/website_forum_share_templates.xml']),
    events: {},

    /**
     * @override
     * @param {Object} parent
     * @param {Object} options
     * @param {string} targetType
     */
    init: function (parent, options, targetType) {
        this._super.apply(this, arguments);
        this.targetType = targetType;
    },
    /**
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);
        this._onMouseEnter();
        return def;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _bindSocialEvent: function () {
        this._super.apply(this, arguments);
        $('.oe_share_bump').click($.proxy(this._postBump, this));
    },
    /**
     * @private
     */
    _render: function () {
        var $question = this.$('article.question');
        if (!this.targetType) {
            this._super.apply(this, arguments);
        } else if (this.targetType === 'social-alert') {
            $question.before(qweb.render('website.social_alert', {medias: this.socialList}));
        } else {
            $('body').append(qweb.render('website.social_modal', {
                medias: this.socialList,
                target_type: this.targetType,
                state: $question.data('state'),
            }));
            $('#oe_social_share_modal').modal('show');
        }
    },
    /**
     * @private
     */
    _postBump: function () {
        this._rpc({ // FIXME
            route: '/forum/post/bump',
            params: {
                post_id: this.element.data('id'),
            },
        });
    },
});

publicWidget.registry.websiteForumShare = publicWidget.Widget.extend({
    selector: '.website_forum',

    /**
     * @override
     */
    start: function () {
        // Retrieve stored social data
        if (sessionStorage.getItem('social_share')) {
            var socialData = JSON.parse(sessionStorage.getItem('social_share'));
            (new ForumShare(this, false, socialData.targetType)).attachTo($(document.body));
            sessionStorage.removeItem('social_share');
        }
        // Display an alert if post has no reply and is older than 10 days
        var $questionContainer = $('.oe_js_bump');
        if ($questionContainer.length) {
            new ForumShare(this, false, 'social-alert').attachTo($questionContainer);
        }

        return this._super.apply(this, arguments);
    },
});
});

```

## File: static\src\js\tours\website_forum.js

```javascript
odoo.define("website_forum.tour_forum", function (require) {
    "use strict";

    var core = require("web.core");
    var tour = require("web_tour.tour");

    var _t = core._t;

    tour.register("question", {
        url: "/forum/1",
    }, [{
        trigger: ".o_forum_ask_btn",
        position: "left",
        content: _t("Create a new post in this forum by clicking on the button."),
    }, {
        trigger: "input[name=post_name]",
        position: "top",
        content: _t("Give your post title."),
    }, {
        trigger: ".note-editable p",
        extra_trigger: "input[name=post_name]:not(:propValue(\"\"))",
        content: _t("Put your question here."),
        position: "bottom",
        run: "text",
    }, {
        trigger: ".select2-choices",
        extra_trigger: ".note-editable p:not(:containsExact(\"<br>\"))",
        content: _t("Insert tags related to your question."),
        position: "top",
        run: function (actions) {
            actions.auto("input[id=s2id_autogen2]");
        },
    }, {
        trigger: "button:contains(\"Post\")",
        extra_trigger: "input[id=s2id_autogen2]:not(:propValue(\"Tags\"))",
        content: _t("Click to post your question."),
        position: "bottom",
    }, {
        extra_trigger: 'div.modal.modal_shown',
        trigger: ".modal-header button.close",
        auto: true,
    },
    {
        trigger: "a:contains(\"Answer\").collapsed",
        content: _t("Click to answer."),
        position: "bottom",
    },
    {
        trigger: ".note-editable p",
        content: _t("Put your answer here."),
        position: "bottom",
        run: "text",
    }, {
        trigger: "button:contains(\"Post Answer\")",
        extra_trigger: ".note-editable p:not(:containsExact(\"<br>\"))",
        content: _t("Click to post your answer."),
        position: "bottom",
    }, {
        extra_trigger: 'div.modal.modal_shown',
        trigger: ".modal-header button.close",
        auto: true,
    }, {
        trigger: ".o_wforum_validate_toggler[data-karma=\"20\"]:first",
        content: _t("Click here to accept this answer."),
        position: "right",
    }]);
});

```

## File: static\src\xml\website_forum_share_templates.xml

```xml
<templates id="template" xml:space="preserve">
    <t t-name="website.social_modal">
        <div role="dialog" class="modal fade" id="oe_social_share_modal">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <header class="modal-header alert alert-info mb0" role="status">
                        <h4 class="modal-title">Thanks for posting!</h4>
                        <button type="button" class="close" data-dismiss="modal">
                            <span role="img" aria-label="Close">x</span>
                        </button>
                    </header>
                    <main class="modal-body">
                        <t t-if="target_type == 'question'" t-call="website_forum.social_message_question"/>
                        <t t-if="target_type == 'answer'" t-call="website_forum.social_message_answer"/>
                        <t t-if="target_type == 'default'" t-call="website_forum.social_message_default"/>
                        <div t-if="state != 'pending'" class="share-icons text-center text-primary">
                            <t t-foreach="medias" t-as="media">
                                <a style="cursor: pointer" t-attf-class="fa-stack fa-lg share #{media}" t-attf-aria-label="Share on #{media}" t-attf-title="Share on #{media}">
                                    <span class="fa fa-square fa-stack-2x"></span>
                                    <span t-attf-class="oe_social_#{media} fa fa-#{media} fa-stack-1x fa-inverse"></span>
                                </a>
                            </t>
                        </div>
                    </main>
                </div>
            </div>
        </div>
    </t>
    <t t-name="website_forum.spam_search_name">
        <t t-foreach="posts" t-as="post">
            <div class="card mb-1 o_spam_character">
                <div class="card-body py-2">
                    <div class="custom-control custom-checkbox">
                        <input type="checkbox" class="custom-control-input" t-attf-id="post_#{post.id}" t-att-value='post.id' checked='checked'/>
                        <label class="custom-control-label" t-attf-for="post_#{post.id}">
                            <b><t t-esc="post.name" /></b>
                            <p class='text-muted'><t t-esc="post.content" /></p>
                        </label>
                    </div>
                </div>
            </div>
        </t>
    </t>
    <t t-name="website_forum.social_message_question">
        <p>On average, <b>45% of questions shared</b> on social networks get an answer within
        5 hours. Questions shared on two social networks have <b>65% more chance to get an
        answer</b> !</p>
        <p t-if="state == 'pending'">You can share your question once it has been validated</p>
    </t>
    <t t-name="website_forum.social_message_answer">
        <p>By sharing you answer, you will get additional <b>karma points</b> if your
        answer is selected as the right one. See what you can do with karma
        <a href="/forum/help-1/faq" target="_blank">here</a>.</p>
    </t>
    <t t-name="website_forum.social_message_default">
        <p>Share this content to increase your chances to be featured on the front page and attract more visitors.</p>
    </t>

    <t t-name="website.social_alert">
        <div class="alert alert-info alert-dismissable" role="status">
            <button type="button" class="close" data-dismiss="alert">
                <span role="img" aria-label="Close">&#215;</span>
            </button>
            <p>Move this question to the top of the list by sharing it on social networks.</p><br/>
            <div>
                <t t-foreach="medias" t-as="media">
                    <a style="cursor: pointer" t-attf-class="fa-stack fa-lg share oe_share_bump #{media}" t-attf-aria-label="Share on #{media}" t-attf-title="Share on #{media}">
                        <span class="fa fa-square fa-stack-2x"></span>
                        <span t-attf-class="oe_social_#{media} fa fa-#{media} fa-stack-1x fa-inverse"></span>
                    </a>
                </t>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\xml\website_forum_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="website_forum.add_new_forum">
        <form id="editor_new_forum">
            <div class="form-group row">
                <label for="page-name" class="col-md-4 col-form-label">Forum Name</label>
                <div class="col-md-8">
                    <input type="text" name="forum_name" class="form-control" required="required"/>
                    <div class="custom-control custom-checkbox mt-2">
                        <input type="checkbox" class="custom-control-input" id="add_to_menu" required="required"/>
                        <label class="custom-control-label" for="add_to_menu">Add to menu</label>
                    </div>
                </div>
            </div>
            <div class="form-group row mt-2">
                <label class="col-md-4 col-form-label" data-toggle="tooltip" data-placement="bottom"
                    title="Questions and Answers mode: only one answer allowed\n Discussions mode: multiple answers allowed">Forum Mode</label>
                <div class="col-md-8">
                    <div class="custom-control custom-radio">
                        <input type="radio" id="questions" name="mode" class="custom-control-input" value="questions" checked="checked"/>
                        <label class="custom-control-label" for="questions">Questions and Answers</label>
                    </div>
                    <div class="custom-control custom-radio">
                        <input type="radio" id="discussions" name="mode" class="custom-control-input" value="discussions"/>
                        <label class="custom-control-label" for="discussions">Discussions</label>
                    </div>
                </div>
            </div>
            <div class="form-group row mt-2">
                <label class="col-md-4 col-form-label" data-toggle="tooltip" data-placement="bottom"
                    title="Public: Forum is public\nSigned In: Forum is visible for signed in users\nSome users: Forum and their content are hidden for non members of selected group">Privacy</label>
                <div class="col-md-8">
                    <div class="custom-control custom-radio">
                        <input type="radio" id="public" name="privacy" class="custom-control-input" value="public" checked="checked"/>
                        <label class="custom-control-label" for="public">Public</label>
                    </div>
                    <div class="custom-control custom-radio">
                        <input type="radio" id="connected" name="privacy" class="custom-control-input" value="connected"/>
                        <label class="custom-control-label" for="connected">Signed In</label>
                    </div>
                    <div class="custom-control custom-radio">
                        <input type="radio" id="private" name="privacy" class="custom-control-input" value="private"/>
                        <label class="custom-control-label" for="private">Some Users</label>
                    </div>
                    <div class="form-group show_visibility_group d-none">
                        <input type="text" class="form-control" name="group_id" id="group_id" placeholder="Select Authorized Group"/>
                        <p id="group-required" class="text-danger mt-1 mb-0 d-none">Please fill in this field</p>
                    </div>
                </div>
            </div>
        </form>
    </t>
</templates>

```

## File: views\forum.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <!-- FORUM ACTIONS -->
        <record id="action_forum_favorites" model="ir.actions.act_window">
            <field name="name">Users favorite posts</field>
            <field name="res_model">forum.post</field>
            <field name="view_mode">tree,form</field>
            <field name="domain">[('forum_id', '=', active_id), ('favourite_count', '>', 0), ('state', 'in', ('active', 'close'))]</field>
        </record>

        <record id="action_forum_posts" model="ir.actions.act_window">
            <field name="name">Posts</field>
            <field name="res_model">forum.post</field>
            <field name="view_mode">tree,form</field>
            <field name="domain">[('forum_id', '=', active_id), ('parent_id', '=', False), ('state', 'in', ('active', 'close'))]</field>
        </record>

        <!-- MAIN FORUM MENU -->
        <menuitem name="Forum" id="menu_website_forum"
            parent="website.menu_website_configuration" sequence="50" groups="website.group_website_designer"/>

        <menuitem name="Forum" id="menu_website_forum_global"
            parent="website.menu_website_global_configuration" sequence="170" groups="website.group_website_designer"/>

        <!-- FORUM VIEWS -->
        <record id="view_forum_forum_list" model="ir.ui.view">
            <field name="name">forum.forum.list</field>
            <field name="model">forum.forum</field>
            <field name="arch" type="xml">
                <tree string="Forums">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                    <field name="website_id" groups="website.group_multi_website"/>
                    <field name="total_posts"/>
                    <field name="total_views"/>
                    <field name="total_answers"/>
                    <field name="total_favorites"/>
                    <field name="active" invisible="1"/>
                </tree>
            </field>
        </record>

        <record id="view_forum_forum_form" model="ir.ui.view">
            <field name="name">forum.forum.form</field>
            <field name="model">forum.forum</field>
            <field name="arch" type="xml">
                <form string="Forum">
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button name="%(action_forum_posts)d" type="action" class="oe_stat_button" icon="fa-comments">
                                <div class="o_form_field o_stat_info">
                                    <span class="o_stat_value">
                                        <field name="total_posts" />
                                    </span>
                                    <span class="o_stat_text">Posts</span>
                                </div>
                            </button>
                            <button name="%(action_forum_favorites)d" class="oe_stat_button" icon="fa-star" type="action">
                                <div class="o_form_field o_stat_info">
                                    <span class="o_stat_value">
                                        <field name="total_favorites" />
                                    </span>
                                    <span class="o_stat_text">Favorites</span>
                                </div>
                            </button>
                            <button type="object" class="oe_stat_button" icon="fa-globe" name="go_to_website">
                                <div class="o_form_field o_stat_info">
                                    <span class="o_stat_text">Go to <br/>Website</span>
                                </div>
                            </button>
                        </div>
                        <field name="active" invisible="1"/>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <field name="image_1920" widget="image" options="{'preview_image': 'image_128'}" class="oe_avatar"/>
                        <div class="oe_title">
                            <label for="name" class="oe_edit_only"/>
                            <h1>
                                <field name="name"/>
                            </h1>
                        </div>
                        <group>
                            <group>
                                <field name="mode" widget="radio" required="True"/>
                                <field name="website_id" options="{'no_create': True}" groups="website.group_multi_website"/>
                            </group>
                        </group>
                        <notebook>
                            <page name="options" string="Options">
                                <group>
                                    <group string="Order and Visibility" name="group_order">
                                        <field name="default_order" string="Default Sort"/>
                                        <field name="privacy" widget="radio" attrs="{'required': True}"/>
                                        <field name="authorized_group_id" options="{'no_create': True}" attrs="{'invisible': [('privacy', '!=', 'private')], 'required': [('privacy', '=', 'private')]}"/>
                                        <label for="relevancy_post_vote" string="Relevance Computation" groups="base.group_no_one" attrs="{'invisible':[('default_order','!=','relevancy desc')]}"/>
                                        <div groups="base.group_no_one" class="o_row" attrs="{'invisible':[('default_order','!=','relevancy desc')]}">
                                            (votes - 1) ** <field name="relevancy_post_vote"/> / (days + 2) ** <field name="relevancy_time_decay"/>
                                        </div>
                                    </group>
                                </group>
                                <group>
                                    <field name="description" nolabel="1" placeholder="Description visible on website"/>
                                </group>
                            </page>
                            <page name="karma_gains" string="Karma Gains">
                                <group name="karma_gain_details">
                                    <group>
                                        <field name="karma_gen_question_new"/>
                                        <field name="karma_gen_question_upvote"/>
                                        <field name="karma_gen_question_downvote"/>
                                        <field name="karma_gen_answer_upvote"/>
                                        <field name="karma_gen_answer_downvote"/>
                                        <field name="karma_gen_answer_accept"/>
                                        <field name="karma_gen_answer_accepted"/>
                                        <field name="karma_gen_answer_flagged"/>
                                    </group>
                                </group>
                            </page>
                            <page name="karma_rights" string="Karma Related Rights">
                                <group>
                                    <group name="karma_rights_left">
                                        <field name="karma_ask"/>
                                        <field name="karma_answer"/>
                                        <field name="karma_upvote"/>
                                        <field name="karma_downvote"/>
                                        <field name="karma_edit_own"/>
                                        <field name="karma_edit_all"/>
                                        <field name="karma_close_own"/>
                                        <field name="karma_close_all"/>
                                        <field name="karma_unlink_own"/>
                                        <field name="karma_unlink_all"/>
                                        <field name="karma_dofollow"/>
                                        <field name="karma_answer_accept_own"/>
                                        <field name="karma_answer_accept_all"/>
                                    </group>
                                    <group name="karma_rights_right">
                                        <field name="karma_editor"/>
                                        <field name="karma_comment_own"/>
                                        <field name="karma_comment_all"/>
                                        <field name="karma_comment_convert_own"/>
                                        <field name="karma_comment_convert_all"/>
                                        <field name="karma_comment_unlink_own"/>
                                        <field name="karma_comment_unlink_all"/>
                                        <field name="karma_post"/>
                                        <field name="karma_flag"/>
                                        <field name="karma_moderate"/>
                                        <field name="karma_edit_retag"/>
                                        <field name="karma_tag_create"/>
                                    </group>
                                </group>
                            </page>
                        </notebook>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" groups="base.group_user"/>
                        <field name="message_ids"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="forum_view_search" model="ir.ui.view">
            <field name="name">forum.forum.search</field>
            <field name="model">forum.forum</field>
            <field name="arch" type="xml">
                <search string="Forum">
                    <field name="name"/>
                    <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                </search>
            </field>
        </record>

        <record id="action_forum_forum" model="ir.actions.act_window">
            <field name="name">Forums</field>
            <field name="res_model">forum.forum</field>
            <field name="view_mode">tree,form</field>
        </record>

        <menuitem id="menu_forum_global" parent="menu_website_forum_global" name="Forums" action="action_forum_forum" sequence="10"/>

        <!-- POST VIEWS -->
        <record id="view_forum_post_list" model="ir.ui.view">
            <field name="name">forum.post.list</field>
            <field name="model">forum.post</field>
            <field name="arch" type="xml">
                <tree string="Forum Posts">
                    <field name="name"/>
                    <field name="active" invisible="1"/>
                    <field name="forum_id"/>
                    <field name="views" sum="Total Views"/>
                    <field name="child_count" sum="Total Answers"/>
                    <field name="favourite_count" sum="Total Favorites"/>
                    <field name="website_id" groups="website.group_multi_website"/>
                    <field name="state"/>
                </tree>
            </field>
        </record>

        <record id="view_forum_post_form" model="ir.ui.view">
            <field name="name">forum.post.form</field>
            <field name="model">forum.post</field>
            <field name="arch" type="xml">
                <form string="Forum Post">
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button type="object" class="oe_stat_button" icon="fa-globe" name="go_to_website">
                                <div class="o_form_field o_stat_info">
                                    <span class="o_stat_text">Go to <br/>Website</span>
                                </div>
                            </button>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <label for="name" class="oe_edit_only"/>
                        <h1>
                            <field name="name" placeholder="Name"/>
                        </h1>
                        <group>
                            <group name="forum_details">
                                <field name="active" invisible="1"/>
                                <field name="forum_id"/>
                                <field name="website_id" groups="website.group_multi_website"/>
                                <field name="parent_id"/>
                            </group>
                            <group name="post_details">
                                <field name="tag_ids" widget="many2many_tags"/>
                                <field name="state"/>
                                <field name="closed_reason_id"/>
                                <field name="closed_uid"/>
                                <field name="closed_date"/>
                            </group>
                            <group name="creation_details">
                                <field name="create_uid"/>
                                <field name="create_date"/>
                                <field name="write_uid"/>
                                <field name="write_date"/>
                            </group>
                            <group name="post_statistics">
                                <field name="is_correct"/>
                                <field name="views"/>
                                <field name="vote_count"/>
                                <field name="favourite_count"/>
                                <field name="child_count"/>
                                <field name="relevancy"/>
                            </group>
                        </group>
                        <group name="answers" string="Answers" attrs="{'invisible':[('parent_id','!=',False)]}">
                            <field name="child_ids" nolabel="1">
                                <tree>
                                    <field name="create_uid" string="Answered by"/>
                                    <field name="vote_count"/>
                                    <field name="state"/>
                                    <field name="is_correct"/>
                                </tree>
                            </field>
                        </group>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" groups="base.group_user"/>
                        <field name="message_ids"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="view_forum_post_search" model="ir.ui.view">
            <field name="name">forum.post.search</field>
            <field name="model">forum.post</field>
            <field name="arch" type="xml">
                <search string="Search in Post">
                    <field name="name" string="Content" filter_domain="['|', ('name', 'ilike', self), ('content', 'ilike', self)]"/>
                    <field name="create_uid"/>
                    <field name="forum_id"/>
                    <field name="tag_ids" string="Tag"/>
                    <filter string="Posts" name="posts" domain="[('parent_id', '=', False)]" />
                    <filter string="Answers" name="answers" domain="[('parent_id', '!=', False)]" />
                    <filter string="Accepted Answer" name="accepted_answer" domain="[('is_correct' , '!=', False), ('parent_id', '!=', False)]" />
                    <filter string="Answered Posts" name="answered_posts" domain="[('child_count', '!=', 0), ('parent_id', '=', False)]" />
                    <separator/>
                    <filter name="filter_create_date" date="create_date"/>
                    <filter name="filter_write_date" date="write_date"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                    <group expand="0" string="Group By">
                        <filter string="Forum" name="forum" domain="[]" context="{'group_by': 'forum_id'}"/>
                        <filter string="Author" name="author" domain="[]" context="{'group_by': 'create_uid'}"/>
                        <filter string="Post" name="post" domain="[]" context="{'group_by': 'parent_id'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record model="ir.ui.view" id="view_forum_post_graph">
            <field name="name">forum.post.graph</field>
            <field name="model">forum.post</field>
            <field name="arch" type="xml">
                <graph string="Graph of Posts" sample="1">
                    <field name="write_date" interval="month" type="col" />
                    <field name="forum_id" type="row" />
                </graph>
            </field>
        </record>

        <record id="action_forum_post" model="ir.actions.act_window">
            <field name="name">Forum Posts</field>
            <field name="res_model">forum.post</field>
            <field name="view_mode">tree,form,graph</field>
            <field name="view_id" ref="view_forum_post_list"/>
            <field name="search_view_id" ref="view_forum_post_search"/>
            <field name="context">{'search_default_posts':1}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a new forum post
                </p>
            </field>
        </record>

        <menuitem id="menu_forum_posts" parent="menu_website_forum" name="Posts" action="action_forum_post" sequence="20"/>

        <!-- TAG VIEWS -->
        <record id="forum_tag_view_list" model="ir.ui.view">
            <field name="name">forum.tag.list</field>
            <field name="model">forum.tag</field>
            <field name="arch" type="xml">
                <tree string="Tags" editable="bottom">
                    <field name="name"/>
                    <field name="forum_id" options="{'no_create_edit': True}"/>
                </tree>
            </field>
        </record>

        <record id="forum_tag_view_form" model="ir.ui.view">
            <field name="name">forum.tag.form</field>
            <field name="model">forum.tag</field>
            <field name="arch" type="xml">
                <form string="Tag">
                    <sheet>
                        <group>
                            <field name="name"/>
                            <field name="forum_id" options="{'no_create_edit': True}"/>
                        </group>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" groups="base.group_user"/>
                        <field name="message_ids"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="forum_tag_action" model="ir.actions.act_window">
            <field name="name">Tags</field>
            <field name="res_model">forum.tag</field>
            <field name="view_mode">tree,form</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a new tag
                </p>
            </field>
        </record>

        <menuitem id="menu_forum_tag_global" parent="menu_website_forum_global" name="Tags" action="forum_tag_action" sequence="30"/>

        <!-- POST REASON VIEWS -->
        <record id="forum_post_reason_view_list" model="ir.ui.view">
            <field name="name">forum.post.reason.list</field>
            <field name="model">forum.post.reason</field>
            <field name="arch" type="xml">
                <tree string="Reasons" editable="bottom">
                    <field name="name"/>
                    <field name="reason_type"/>
                </tree>
            </field>
        </record>

        <record id="forum_post_reasons_action" model="ir.actions.act_window">
            <field name="name">Post Close Reasons</field>
            <field name="res_model">forum.post.reason</field>
            <field name="view_mode">tree</field>
        </record>


        <menuitem id="menu_forum_rank_global" parent="menu_website_forum_global" name="Ranks" action="gamification.gamification_karma_ranks_action" sequence="5"/>
        <menuitem id="menu_forum_badges" parent="menu_website_forum_global" name="Badges" action="gamification.badge_list_action" sequence="40"/>
        <menuitem id="menu_forum_post_reasons" parent="menu_website_forum_global" name="Close Reasons" action="forum_post_reasons_action" sequence="50"/>
    </data>
</odoo>

```

## File: views\ir_qweb.xml

```xml
<odoo>
<data>
<template id="contact" inherit_id="base.contact" name="Forum Contact Widget">
    <xpath expr="//div[@itemprop='address']" position="after">
        <div>
             <div t-if="'karma' in fields" class='css_editable_mode_hidden'>
                <div t-if="options.get('UserBio')" class="mb-2">
                    <span t-field="object.company_name" class="o_forum_tooltip_line"/><br/>
                        <a t-att-href="object.website" t-if="object.website">
                            <span t-field="object.website" class="o_forum_tooltip_line"/>
                        </a>
                </div>
                <b class="mt-4"><i class="fa fa-diamond text-secondary"/> <t t-esc="object.karma"/></b>
                <div t-if="options.get('badges')" style="display: inline-block">
                    <t t-raw="separator"/>
                    <b>|</b>
                    <span class="fa fa-trophy badge-gold ml-2" role="img" aria-label="Gold badge" title="Gold badge"/>
                    <t t-esc="object.gold_badge"/>
                    <span class="fa fa-trophy badge-silver ml-2" role="img" aria-label="Silver badge" title="Silver badge"/>
                    <t t-esc="object.silver_badge"/>
                    <span class="fa fa-trophy badge-bronze ml-2" role="img" aria-label="Bronze badge" title="Bronze badge"/>
                    <t t-esc="object.bronze_badge"/>
                </div>
                <t t-raw="0"/>
                <div t-if="options.get('UserBio')" class="mt-2">
                    <div class="o_forum_tooltip_line" t-if="object.partner_id.country_id or object.partner_id.city">
                        <span t-field="object.partner_id.city"/><span t-if="object.partner_id.city and object.partner_id.country_id">, </span><span t-field="object.partner_id.country_id"/>
                    </div>
                </div>
            </div>
            <span t-if="options.get('website_description') and 'partner_id' in fields">
                <t t-if="object.partner_id.website_description">
                    <span t-field="object.partner_id.website_description"/>
                </t>
            </span>

        </div>
    </xpath>
</template>
 </data>
</odoo>

```

## File: views\res_users_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>

        <!-- Update user form !-->
        <record id="view_users_form_forum" model="ir.ui.view">
            <field name="name">res.users.form.forum</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="base.view_users_form"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <field name="is_published" widget="website_redirect_button"/>
                </div>
            </field>
        </record>

    </data>
</odoo>

```

## File: views\website_forum.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

<!-- Editor custom -->
<template id="assets_editor" inherit_id="website.assets_editor" name="Forum Editor Assets">
    <xpath expr="." position="inside">
        <script type="text/javascript" src="/website_forum/static/src/js/tours/website_forum.js"/>
        <script type="text/javascript" src="/website_forum/static/src/js/website_forum.editor.js"/>
    </xpath>
</template>

<template id="assets_tests" name="Website Forum Assets Tests" inherit_id="web.assets_tests">
    <xpath expr="." position="inside">
        <script type="text/javascript" src="/website_forum/static/tests/tours/website_forum_question.js"></script>
    </xpath>
</template>

<template id="assets_frontend" inherit_id="website.assets_frontend">
    <xpath expr="link[last()]" position="after">
        <link rel="stylesheet" type="text/scss" href="/website_forum/static/src/scss/website_forum.scss"/>
    </xpath>
    <xpath expr="script[last()]" position="after">
        <script type="text/javascript" src="/website_forum/static/src/js/website_forum.js"/>
        <script type="text/javascript" src="/website_forum/static/src/js/website_forum.share.js"/>
    </xpath>
</template>

<!-- helper -->
<template id="link_button">
    <form  t-attf-method="#{form_method or 'POST'}" t-att-action="url" t-attf-class="#{form_classes} #{not inDropdown and 'btn btn-sm border'}">
        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
        <button t-attf-class="#{icon and not label and ('fa ' + icon)} #{inDropdown and 'dropdown-item pl-3' or 'btn btn-sm p-0'} #{classes} #{karma and 'karma_required text-muted'}" t-attf-data-karma="#{karma}" t-att-title="title">
            <i t-if="icon and label" t-attf-class="fa fa-fw text-muted #{icon} #{inDropdown and 'mr-1'}"/>
            <t t-esc="label"/>
        </button>
    </form>
</template>

<!-- website_forum.layout removes the access right check for summernote bundle -->
<template id="layout" inherit_id="website.layout" name="Forum Layout" primary="True">
    <xpath expr="//div[@id='wrapwrap']" position="before">
        <t t-set="pageName" t-value="'website_forum'"/>
    </xpath>
    <xpath expr="//div[@id='wrapwrap']" position="attributes">
        <attribute name="t-att-data-forum_id">forum and forum.id</attribute>
    </xpath>
</template>

<!-- Page Index -->
<template id="header" name="Forum Index">
    <t t-if="forum.active" t-call="website_forum.layout">
        <section t-attf-class="s_cover parallax s_parallax_is_fixed py-3 #{forum.image_1920 and 'bg-black-50' or 'o_wforum_forum_card_bg text-white'}" data-scroll-background-ratio="1" data-snippet="s_cover">
            <span t-if="forum.image_1920" class="s_parallax_bg oe_img_bg" t-attf-style="background-image: url('#{website.image_url(forum, 'image_1920')}'); background-position: center;"/>
            <div t-if="forum.image_1920" class="o_we_bg_filter bg-black-50"/>
            <div class="container">
                <div class="row s_nb_column_fixed">
                    <div class="col-lg-12">
                        <h1 class="o_default_snippet_text text-center"><t t-esc="forum.name"></t></h1>
                    </div>
                </div>
                <div t-if="editable or (is_public_user and not forum_welcome_message)" t-att-class="'css_non_editable_mode_hidden' if editable else 'forum_intro'">
                    <div t-field="forum.welcome_message"/>
                </div>
            </div>
        </section>

        <div class="o_forum_nav_header_container mb-2 mb-md-4">
            <t t-call="website_forum.forum_nav_header"></t>
        </div>

        <div id="wrap" t-attf-class="container #{website_forum_action}">
            <div class="row">
                <div class="col o_wprofile_email_validation_container mb16">
                    <t t-call="website_profile.email_validation_banner">
                        <t t-set="redirect_url" t-value="'/forum/%s' % forum.id"/>
                        <t t-set="send_validation_email_message">Click here to send a verification email allowing you to participate in the forum.</t>
                        <t t-set="additional_validated_email_message"> You may now participate in our forums.</t>
                    </t>
                    <div class="row">
                        <div class="col">
                            <nav t-if="header.get('is_guidelines') or queue_type or new_question or is_edit or tags or reasons" aria-label="breadcrumb">
                                <ol class="breadcrumb p-0 bg-white">
                                    <li class="breadcrumb-item">
                                        <a t-attf-href="/forum/#{ slug(forum) }" t-esc="forum.name"/>
                                    </li>
                                    <t t-if="header.get('is_guidelines')">
                                        <li class="breadcrumb-item">
                                            <a t-if="header.get('is_karma')" t-attf-href="/forum/#{ slug(forum) }/faq">Guidelines</a>
                                            <t t-else="">
                                                Guidelines
                                            </t>
                                        </li>
                                        <li t-if="header.get('is_karma')" class="breadcrumb-item">Karma</li>
                                    </t>
                                    <li t-if="queue_type" class="breadcrumb-item">Moderation</li>
                                    <li t-if="queue_type == 'validation'" class="breadcrumb-item">To Validate</li>
                                    <li t-if="queue_type == 'flagged'" class="breadcrumb-item">Flagged</li>
                                    <li t-if="queue_type == 'offensive'" class="breadcrumb-item">Offensive</li>
                                    <li t-if="reasons and offensive" class="breadcrumb-item">Offensive Post</li>
                                    <li t-if="reasons and not offensive" class="breadcrumb-item">Close Post</li>
                                    <li t-if="new_question" class="breadcrumb-item">New Post</li>
                                    <t t-if="is_edit">
                                        <t t-set="target" t-value="post.parent_id if is_answer else post"/>
                                        <li class="breadcrumb-item text-truncate" style="max-width:150px">
                                            <a t-attf-href="/forum/#{ slug(forum) }/#{ slug(target)}" title="Back to Question">
                                                <t t-esc="target.name"/>
                                            </a>
                                        </li>
                                        <li t-if="not is_answer" class="breadcrumb-item">Edit Question</li>
                                        <li t-if="is_answer" class="breadcrumb-item">Edit Answer</li>
                                    </t>
                                    <li t-elif="tags" class="breadcrumb-item">All Tags</li>
                                </ol>
                            </nav>
                            <t t-raw="0"/>
                        </div>
                        <aside t-if="uid" class="d-none d-lg-flex justify-content-end col-auto">
                            <t t-call="website_forum.user_sidebar"/>
                        </aside>
                    </div>
                </div>
            </div>
        </div>
        <div class="oe_structure" id="oe_structure_website_forum_header_1"/>
    </t>
    <t t-else="" t-call="website_forum.layout">
        <t t-set="head">
            <meta name="robots" content="noindex, nofollow" />
        </t>
        <div class="text-center text-muted">
            <p class="css_editable_hidden"><h2>This forum has been archived.</h2></p>
        </div>
    </t>
</template>

<template id="forum_nav_header">
    <div class="navbar navbar-expand-sm navbar-light">
        <div class="container flex-wrap flex-md-nowrap">
            <a t-if="back_button_url" class="btn btn-light border mr-2 o_back_button" title="Back">
                <i class="fa fa-chevron-left mr-1"/>Back
            </a>
            <!-- Desktop -->
            <ul class="navbar-nav mr-auto d-none d-lg-flex">
                <li class="nav-item">
                    <a t-if="request.website.forums_count > 1" class="nav-link" href="/forum/" title="All forums">
                        All Forums
                    </a>
                </li>
                <li class="nav-item">
                    <a t-attf-href="/forum/#{ slug(forum) }" t-attf-class="nav-link #{question_count and 'active'}">Topics</a>
                </li>
                <li class="nav-item">
                    <a t-attf-href="/profile/users?forum_origin=#{request.httprequest.path}"
                        t-attf-class="nav-link #{searches.get('users') and 'active'}">People</a>
                </li>
                <li class="nav-item">
                    <a t-attf-href="/forum/#{ slug(forum) }/tag" t-attf-class="nav-link #{searches.get('tags') and 'active'}">Tags</a>
                </li>
                <li class="nav-item">
                    <a t-attf-href="/profile/ranks_badges?badge_category=forum&amp;url_origin=#{request.httprequest.path}&amp;name_origin=#{forum.name}"
                    t-attf-class="nav-link #{searches.get('badges') and 'active'}">Badges</a>
                </li>
                <li class="nav-item">
                    <a t-attf-href="/forum/#{ slug(forum) }/faq" t-attf-class="nav-link #{header.get('is_guidelines') and 'active'}">About</a>
                </li>
            </ul>

            <!-- Mobile -->
            <ul class="navbar-nav d-lg-none flex-row flex-grow-1 justify-content-between">
                <span class="navbar-text mr-1">Go to:</span>
                <li class="nav-item dropdown mr-auto">
                    <a class="nav-link active dropdown-toggle" type="button" data-toggle="dropdown">
                        <t t-if="searches.get('users')">People</t>
                        <t t-elif="searches.get('tags')">Tags</t>
                        <t t-elif="searches.get('badges')">Badges</t>
                        <t t-elif="header.get('is_guidelines')">About</t>
                        <t t-elif="uid and my == 'favourites'">Favourites</t>
                        <t t-elif="uid and my == 'mine'">My Posts</t>
                        <t t-elif="uid and my == 'followed'">Following</t>
                        <t t-elif="question">Question</t>
                        <t t-else="">All Topics</t>
                    </a>
                    <div class="dropdown-menu position-absolute">
                        <a t-if="searches or my or question" t-attf-href="/forum/#{ slug(forum) }" class="dropdown-item">All Topics</a>
                        <a t-if="not searches.get('users')" t-attf-href="/profile/users?forum_origin=#{request.httprequest.path}" class="dropdown-item">People</a>
                        <a t-if="not searches.get('tags')" t-attf-href="/forum/#{slug(forum)}/tag" class="dropdown-item">Tags</a>
                        <a t-if="not searches.get('badges')" t-attf-href="/profile/ranks_badges?badge_category=forum&amp;url_origin=#{request.httprequest.path}&amp;name_origin=#{forum.name}" class="dropdown-item">Badges</a>
                        <a t-if="not header.get('is_guidelines')" t-attf-href="/forum/#{ slug(forum) }/faq" class="dropdown-item">About</a>
                        <t t-if="uid">
                            <div class="dropdown-divider"/>
                            <a t-att-href="'/forum/%s/user/%s?forum_origin=%s' % (slug(forum), uid, request.httprequest.path)"
                                class="dropdown-item">My profile</a>
                            <a t-if="my != 'mine'" t-att-href="url_for('') + '?' + keep_query( 'search', 'sorting', 'filters', my='mine')" class="dropdown-item">My Posts</a>
                            <a t-if="my != 'favourites'" t-att-href="url_for('') + '?' + keep_query( 'search', 'sorting', 'filters', my='favourites')" class="dropdown-item">My Favourites</a>
                            <a t-if="my != 'followed'" t-att-href="url_for('') + '?' + keep_query( 'search', 'sorting', 'filters', my='followed')" class="dropdown-item">I'm Following</a>
                            <a t-if="my != 'tagged'" t-att-href="url_for('') + '?' + keep_query( 'search', 'sorting', 'filters', my='tagged')" class="dropdown-item">Tags I Follow</a>
                        </t>
                        <div groups="base.group_erp_manager" class="dropdown-divider"/>
                        <a groups="base.group_erp_manager" t-attf-href="/web#id=#{forum.id}&amp;view_type=form&amp;model=forum.forum" class="dropdown-item">Edit Forum in Backend</a>
                    </div>
                </li>
                <t t-if="user.karma>=forum.karma_moderate">
                    <li t-if="forum.count_posts_waiting_validation" class="nav-item">
                        <a class="nav-link" t-attf-href="/forum/#{slug(forum)}/validation_queue">
                            <i class="fa fa-check-square-o fa-fw text-warning"/>
                            <b t-esc="forum.count_posts_waiting_validation" class="text-800"/>
                        </a>
                    </li>
                    <li t-if="forum.count_flagged_posts" class="nav-item ml-2">
                        <a class="nav-link" t-attf-href="/forum/#{slug(forum)}/flagged_queue">
                            <i class="fa fa-flag fa-fw text-danger"/>
                            <b t-esc="forum.count_flagged_posts" class="text-800"/>
                        </a>
                    </li>
                </t>
                <!-- Mobile 'Search Box' toggler-->
                <li class="nav-item ml-4">
                    <a data-toggle="collapse" href="#o_wforum_search" class="nav-link"><i class="fa fa-search"/></a>
                </li>
            </ul>

            <!-- 'Search Box' -->
            <form id="o_wforum_search" class="form-inline collapse w-100 w-md-auto pt-2 pt-md-0 d-md-flex"
                  role="search" t-attf-action="#{url_for('/forum/')}#{slug(forum)}#{tag and ('/tag/%s/questions' % slug(tag))}" method="get">
                <t t-call="website.website_search_box">
                    <t t-set="_classes" t-valuef="w-100"/>
                </t>

                <input t-if="filters" type="hidden" name="filters" t-att-value="filters"/>
                <input t-if="my" type="hidden" name="my" t-att-value="my"/>
                <input t-if="sorting" type="hidden" name="sorting" t-att-value="sorting"/>
            </form>
        </div>
    </div>
</template>

<!-- Display a post -->
<template id="display_post_question_block">
    <div class="o_wforum_index_entry_title">
        <div class="d-inline-block mb-0 h5">
            <span t-if="question.has_validated_answer and filters != 'solved'"
                title="Solved"
                aria-label="Solved"
                data-toggle="tooltip"
                class="fa fa-check-circle text-success"/>
            <span t-if="question.user_favourite and not (my == 'favourites' or hide_fav_icon)"
                title="Your favourite"
                aria-label="Your favourite"
                data-toggle="tooltip"
                class="fa fa-star o_wforum_gold"/>

            <a t-attf-href="/forum/#{slug(question.forum_id)}/#{slug(question)}#{answer and ('/#answer-%s' % answer.id)}"
                t-attf-title="Read: #{question.name}"
                class="text-reset"
                t-esc="question.name"/>
        </div>
        <span t-if="not question.active" class="text-muted">
            <t t-if="question.state!='offensive'"> [Deleted]</t>
            <t t-if="question.state=='offensive'"> [Offensive]</t>
            <t t-if="question.state=='offensive' and question.closed_reason_id">
                [<t t-esc="question.closed_reason_id.name[0].upper() + question.closed_reason_id.name[1:]"/>]
            </t>
        </span>
        <span t-if="question.state == 'close'" class="text-muted"> [Closed]</span>
    </div>

    <div t-attf-class="o_wforum_index_entry_tags mb-1" t-if="len(question.tag_ids) > 0">
        <t t-foreach="question.tag_ids" t-as="question_tag">

            <!-- Toggle Tags on click -->
            <t t-if="tag and tag.name == question_tag.name" t-set="click_action"
                t-value="'/forum/' + slug(question_tag.forum_id) + '?' + keep_query( 'search', 'sorting', 'my')"/>
            <t t-else="" t-set="click_action"
                t-value="'/forum/' + slug(question_tag.forum_id) + '/tag/' + slug(question_tag) + '/questions?' + keep_query( 'search', 'sorting', 'my', filters='tag')"/>

            <a t-att-href="click_action"
                t-attf-class="badge #{tag and tag.name == question_tag.name and 'badge-secondary' or 'border text-600 badge-light'} #{ not question_tag_first and 'ml-lg-1 mt-lg-1'}"
                t-field="question_tag.name"/>
        </t>
    </div>

    <div class="d-flex align-items-center justify-content-between justify-content-sm-start small text-muted">
        <div>
            <t t-call="website_forum.vote">
                <t t-set="post" t-value="question"/>
            </t>
            <span t-field="question.write_date" t-options='{"format": "d MMMM y"}'/>, by <a t-attf-href="/forum/#{slug(question.forum_id)}/user/#{question.create_uid.id}?forum_origin=#{request.httprequest.path}" t-field="question.create_uid" class="d-inline-block font-weight-bold" t-options='{"widget": "contact", "fields": ["name"]}'/>
        </div>
        <div>
            <span class="mx-1 d-none d-sm-inline">&amp;nbsp;|</span>
            <a t-if="question.child_count" class="font-weight-bold" t-attf-href="/forum/#{ slug(question.forum_id) }/#{ slug(question) }">
                <t t-esc="question.child_count"/>
                <t t-if="question.child_count == 1">Answer</t>
                <t t-else="">Answers</t>
            </a>
            <span t-else="">
                0 Answers
            </span>
            <span class="d-none d-sm-inline">
                <span class="mx-1">|</span>
                <span t-field="question.views" /> <t t-if="question.views&lt;=1">View</t><t t-else="">Views</t>
                <span t-if="question.favourite_count &gt; 0">
                    <span class="mx-1">|</span>
                    <i class="fa fa-star"/>
                    <t t-esc="question.favourite_count"/>
                </span></span>
            <span t-if="question.state == 'flagged'" class="text-black"> | Flagged</span>
        </div>
    </div>
    <!--  Display post's content in moderation mode-->
    <div><t t-raw="post_content"/></div>
</template>

<template id="display_post">
    <div t-attf-class="#{show_author_avatar and 'mt-2 mb-4' or 'card py-2 px-3'}">
        <div class="media">
            <div t-if="show_author_avatar">
                <t t-call="website_forum.author_box">
                    <t t-set="object" t-value="question"/>
                    <t t-set="allow_biography" t-value="True"/>
                </t>
            </div>
            <div t-attf-class="media-body #{show_author_avatar and 'pl-2'}">
                <t t-call="website_forum.display_post_question_block"/>
            </div>
        </div>
    </div>
</template>

<!-- Display a post as an answer -->
<template id="display_post_answer">
    <t t-set="question" t-value="answer"/>
    <t t-call="website_forum.display_post"/>
</template>

<!-- Moderation tools -->
<template id="moderation_display_post_question_block">
    <t t-call="website_forum.display_post_question_block">
        <t t-set="post_content">
            <div class="clearfix">
                <span t-field="question.content" class="oe_no_empty"/>
            </div>
        </t>
    </t>
</template>

<template id="moderation_display_post_answer">
    <div class="clearfix">
        <div class="question-name">
            <a style="font-size: 15px;" t-attf-href="/forum/#{ slug(answer.forum_id) }/#{ answer.parent_id.id }/#answer-#{ answer.id }" t-esc="answer.parent_id.name"/>
            <b>[Answer]</b>
            <span t-if="not answer.active and answer.state=='offensive'"><b> [Offensive]</b></span>
            <span t-if="not answer.active and answer.state=='offensive' and answer.closed_reason_id"><b> [<t t-esc="answer.closed_reason_id.name[0].upper() + answer.closed_reason_id.name[1:]"/>]</b></span>
            <t t-if="answer.state == 'flagged'">
                <small class="text-muted">
                    Flagged
                </small>
            </t>
            <t t-if="len(answer.website_message_ids)&gt;0">
                (<t t-esc="len(answer.website_message_ids)"/>
                <t t-if="len(answer.website_message_ids)&gt;1"> Comments</t>
                <t t-if="len(answer.website_message_ids)&lt;=1"> Comment</t>)
            </t>
        </div>
        <div class="clearfix"><span t-field="answer.content" class="oe_no_empty"/></div>
    </div>
</template>

<!-- FAQ Layout -->
<template id="faq" name="Frequently Asked Questions">
    <t t-call="website_forum.header">
        <div t-field="forum.description" class="mb-4"/>
        <div t-field="forum.faq"/>
    </t>
</template>

<!-- FAQ Karma Layout -->
<template id="faq_karma" name="Karma">
    <t t-call="website_forum.header">
        <div class="card bg-white" data-name="Item">
            <div role="tab" class="card-header">
                <b>Why can other people edit my questions/answers?</b>
            </div>
            <div role="tabpanel">
                <div class="card-body">
                <p>The goal of this site is create a relevant knowledge base that would answer questions related to Odoo.</p>
                <p>Therefore questions and answers can be edited like wiki pages by experienced users of this site in order to improve the overall quality of the knowledge base content. Such privileges are granted based on user karma level: you will be able to do the same once your karma gets high enough.</p>
                <p>If this approach is not for you, please respect the community.</p>
                    <table class="table table-striped mt-4 bg-white">
                        <tbody>
                            <tr>
                                <td class="faq-rep-item"><strong t-field="forum.karma_upvote"/></td>
                                <td>upvote, add comments</td>
                            </tr><tr>
                                <td class="faq-rep-item"><strong t-field="forum.karma_downvote"/></td>
                                <td>downvote</td>
                            </tr><tr>
                                <td class="faq-rep-item"><strong t-field="forum.karma_editor"/></td>
                                <td>insert text link, upload files</td>
                            </tr><tr>
                                <td class="faq-rep-item"><strong t-field="forum.karma_user_bio"/></td>
                                <td>your biography can be seen as tooltip</td>
                            </tr><tr>
                                <td class="faq-rep-item"><strong t-field="forum.karma_comment_unlink_own"/></td>
                                <td>delete own comment</td>
                            </tr><tr>
                                <td class="faq-rep-item"><strong t-field="forum.karma_close_own"/></td>
                                <td>flag offensive, close own questions</td>
                            </tr><tr>
                                <td class="faq-rep-item"><strong t-field="forum.karma_edit_all"/></td>
                                <td>edit any post, view offensive flags</td>
                            </tr><tr>
                                <td class="faq-rep-item"><strong t-field="forum.karma_answer_accept_all"/></td>
                                <td>accept any answer</td>
                            </tr><tr>
                                <td class="faq-rep-item"><strong t-field="forum.karma_comment_unlink_all"/></td>
                                <td>delete any comment</td>
                            </tr><tr>
                                <td class="faq-rep-item"><strong t-field="forum.karma_close_all"/></td>
                                <td>close any posts</td>
                            </tr><tr>
                                <td class="faq-rep-item"><strong t-field="forum.karma_unlink_all"/></td>
                                <td>delete any question or answer</td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </t>
</template>

<!-- All Forums Layout -->
<template id="forum_all" name="Forum Navigation">

    <t t-set="col_class" t-valuef="mb-3 col-sm-6"/>
    <t t-set="img_class" t-valuef="col-md-4"/>
    <t t-set="content_class" t-valuef="col-md-8"/>
    <t t-set="last_post_class" t-valuef="col-md-12 pr-md-3 pt-3"/>
    <t t-set="nb_post_class" t-valuef="col-md-12 pr-md-3 pt-2"/>

    <t t-call="website.layout">
        <t t-set="pageName" t-value="'website_forum'"/>
        <div id="wrap">
            <div class="oe_structure oe_empty" id="oe_structure_forum_all_top"/>
            <div id="o_wforum_forums_index_list" class="container pt-4 pb-5">
                <div t-if="forums" class="row">
                    <t t-call="website_forum.forum_all_all_entries">
                        <t t-set="_forums" t-value="forums"/>
                    </t>
                </div>
                <t t-else="">
                    <div class="alert alert-info">No forum is available yet.</div>
                </t>
            </div>
            <div class="oe_structure oe_empty" id="oe_structure_forum_all_bottom"/>
        </div>
    </t>
</template>

<template id="forum_all_all_entries">
    <!-- Check if at least one forum without description exist -->
    <t t-set="no_description_exist" t-value="bool(_forums.filtered(lambda f: not f.description))"/>
    <t t-set="sorted_forums" t-value="_forums.sorted(lambda f: (not f.description, f.sequence, f.id))"/>

    <!-- First, list all forums (those with descriptions first) -->
    <t t-foreach="sorted_forums" t-as="forum">
        <t t-set="has_desc" t-value="forum.description"/>
        <div t-attf-class="#{col_class}">
            <div class="row py-4 bg-200 mx-1 h-100 o_forum_row">
                <div t-attf-class="o_forum_image_container pr-md-0 h-100 #{img_class}">
                    <a t-attf-href="/forum/#{slug(forum)}">
                        <div t-attf-class="h-100 w-100 #{not forum.image_1920 and 'rounded o_wforum_forum_card_bg shadow-sm flex-shrink-0'}">
                            <div t-if="forum.image_1920 or editable" t-attf-class="h-100"
                            t-field="forum.image_1920" t-options="{'widget': 'image', 'preview_image': 'image_256', 'class': 'w-100 h-100 o_object_fit_cover rounded'}" />
                        </div>
                    </a>
                </div>
                <div t-attf-class="#{content_class} mt-2 mt-md-0 d-flex flex-column h-100">
                    <a t-attf-href="/forum/#{slug(forum)}" class="text-reset" t-att-title="forum.name">
                        <h3 class="h4" t-field="forum.name"/>
                    </a>
                    <p class="m-0 flex-grow-1"
                    placeholder="Description"
                    t-field="forum.teaser"/>
                    <t t-if="is_view_active('website_forum.opt_post_count') or is_view_active('website_forum.opt_last_post')" t-call="website_forum.forum_post_options"/>
                </div>
            </div>
        </div>
    </t>
</template>

<template id="forum_post_options">
    <div class="row">
        <div t-attf-class="#{last_post_class}">
            <div t-if="is_view_active('website_forum.opt_last_post') and forum.post_ids" class="text-truncate">Last Post: <a t-attf-href="/forum/#{slug(forum)}/#{slug(forum.last_post_id)}"><t t-esc="forum.last_post_id.name"/></a></div>
        </div>
        <div t-attf-class="#{nb_post_class}">
            <div t-if="is_view_active('website_forum.opt_post_count')">Posts: <strong><t t-esc="forum.total_posts"/></strong></div>
        </div>
    </div>
</template>

<!-- (Options) Forum : List View
    Display forums as a list  -->
<template name="List View" id="website_forum.opt_list_view" inherit_id="website_forum.forum_all" active="False" customize_show="True">
    <xpath expr="//t[@t-set='col_class']" position="attributes">
        <attribute name="t-valuef">mb-3 col-sm-12</attribute>
    </xpath>
    <xpath expr="//t[@t-set='img_class']" position="attributes">
        <attribute name="t-valuef">col-md-3</attribute>
    </xpath>
    <xpath expr="//t[@t-set='content_class']" position="attributes">
        <attribute name="t-valuef">col-md-9</attribute>
    </xpath>
    <xpath expr="//t[@t-set='last_post_class']" position="attributes">
        <attribute name="t-valuef">col-md-10 pr-md-0 pt-3</attribute>
    </xpath>
    <xpath expr="//t[@t-set='nb_post_class']" position="attributes">
        <attribute name="t-valuef">col-md-2 pr-md-3 pt-3</attribute>
    </xpath>
</template>

<!-- (Options) Forum : Show Post Count
    Show the number of post a forum has  -->
<template name="Show Post Count" id="website_forum.opt_post_count" inherit_id="website_forum.forum_all" active="False" customize_show="True"/>

<!-- (Options) Forum : Show Last Post
    Show the title of the latest post in each forum  -->
<template name="Show Last Post" id="website_forum.opt_last_post" inherit_id="website_forum.forum_all" active="False" customize_show="True"/>


<!-- Default content for the "All Forums Layout" header above -->
<!-- (simulate an oe_structure edition) -->
<template id="forum_all_oe_structure_forum_all_top" inherit_id="website_forum.forum_all" name="Forum Navigation (oe_structure_forum_all_top)">
    <xpath expr="//*[hasclass('oe_structure')][@id='oe_structure_forum_all_top']" position="replace">
        <div class="oe_structure oe_empty" id="oe_structure_forum_all_top">
            <section class="s_cover parallax s_parallax_is_fixed bg-black-50 py-5" data-scroll-background-ratio="1" data-snippet="s_cover">
                <span class="s_parallax_bg oe_img_bg" style="background-image: url('/web/image/website.s_cover_default_image'); background-position: 50% 0;"/>
                <div class="o_we_bg_filter bg-black-50"/>
                <div class="container">
                    <div class="row s_nb_column_fixed">
                        <div class="col-lg-12">
                            <h1 class="o_default_snippet_text text-center">Our forums</h1>
                            <p class="lead o_default_snippet_text mb-0" style="text-align: center;">
                                This community is for professional and enthusiast users, partners and programmers.
                            </p>
                        </div>
                    </div>
                </div>
            </section>
        </div>
    </xpath>
</template>

<template id="website_forum.user_sidebar">
    <nav t-if="uid" class="o_wforum_nav nav nav-pills flex-column ml-4">
        <a t-attf-href="/forum/#{slug(forum)}/user/#{uid}?forum_origin=#{request.httprequest.path}"
            class="nav-link d-flex align-items-center rounded-pill text-reset mb-2"
            data-toggle="tooltip"
            data-trigger="hover"
            title="My profile">
            <img class="o_forum_avatar rounded-circle mr-1" t-att-src="website.image_url(user, 'image_128', '30x30')" alt="Avatar"/>
            <div>
                <h6 class="my-0" t-esc="user_id.name"/>
                <small class="text-muted font-weight-bold"><t t-esc="user_id.karma"/>xp</small>
            </div>
        </a>

        <t t-set="location" t-value="url_for('/forum/') + slug(forum) + ( ('/tag/' + slug(tag) + '/questions?') if tag else '?' )"/>

        <!-- My Posts -->
        <span t-if="my == 'mine'" class="nav-link rounded-pill mb-2 active font-weight-bold">
            <i class="fa fa-question-circle-o fa-fw"/> My Posts
            <a class="text-reset pull-right no-decoration" t-att-href="location + keep_query('search', 'filters', 'sorting')">&#215;</a>
        </span>
        <a t-else="" class="nav-link rounded-pill mb-2 text-reset" t-att-href="location + keep_query('search', 'filters', 'sorting', my='mine')">
            <i class="fa fa-question-circle-o fa-fw"/> My Posts
        </a>

        <!-- My Favourites -->
        <span t-if="my == 'favourites'" class="nav-link rounded-pill mb-2 active font-weight-bold">
            <i class="fa fa-star fa-fw"/> Favourites
            <a class="text-reset pull-right no-decoration" t-att-href="location + keep_query( 'search', 'filters', 'sorting')">&#215;</a>
        </span>
        <a t-else="" t-attf-class="nav-link rounded-pill mb-2 text-reset" t-att-href="location + keep_query( 'search', 'filters', 'sorting', my='favourites')">
            <i class="fa fa-star fa-fw"/> Favourites
        </a>

        <!-- My Followed posts -->
        <span t-if="my == 'followed'" class="nav-link rounded-pill mb-2 active font-weight-bold">
            <i class="fa fa-bell fa-fw"/> Followed Posts
            <a class="text-reset pull-right no-decoration" t-att-href="location + keep_query( 'search', 'filters', 'sorting')">&#215;</a>
        </span>
        <a t-else="" class="nav-link rounded-pill mb-2 text-reset" t-att-href="location + keep_query( 'search', 'filters', 'sorting', my='followed')">
            <i class="fa fa-bell fa-fw"/> Followed Posts
        </a>

        <!-- My Followed tags -->
        <span t-if="my == 'tagged'" class="nav-link rounded-pill mb-2 active font-weight-bold">
            <i class="fa fa-tags fa-fw"/> Followed Tags
            <a class="text-reset pull-right no-decoration" t-att-href="location + keep_query( 'search', 'filters', 'sorting')">&#215;</a>
        </span>
        <a t-else="" class="nav-link rounded-pill mb-2 text-reset" t-att-href="location + keep_query( 'search', 'filters', 'sorting', my='tagged')">
            <i class="fa fa-tags fa-fw"/> Followed Tags
        </a>

        <!-- Moderation Tools -->
        <t t-if="user.karma>=forum.karma_moderate or queue_type">
            <span class="nav-link disabled mt-3">
                <div class="pb-1 border-bottom text-muted">Moderation tools</div>
            </span>

            <span t-if="queue_type == 'validation'" class="nav-link rounded-pill mb-2 active font-weight-bold">
                <i class="fa fa-check-square-o fa-fw"/> To Validate
                <a class="text-reset pull-right no-decoration" t-attf-href="/forum/#{ slug(forum) }">&#215;</a>
            </span>
            <a t-else="" class="nav-link rounded-pill text-reset" t-attf-href="/forum/#{slug(forum)}/validation_queue">
                <i class="fa fa-check-square-o fa-fw"/> To Validate
                <span t-attf-class="badge pull-right #{forum.count_posts_waiting_validation > 0 and 'badge-warning' or 'badge-light'}" t-esc="forum.count_posts_waiting_validation"/>
            </a>
            <span t-if="queue_type == 'offensive' or queue_type == 'flagged'" class="nav-link rounded-pill mb-2 active font-weight-bold">
                <i class="fa fa-flag fa-fw"/> Flagged
                <a class="text-reset pull-right no-decoration" t-attf-href="/forum/#{ slug(forum) }">&#215;</a>
            </span>
            <a t-else="" class="nav-link rounded-pill text-reset" t-attf-href="/forum/#{slug(forum)}/flagged_queue">
                <i class="fa fa-flag fa-fw"/> Flagged
                <span id="count_flagged_posts" t-attf-class="badge pull-right #{forum.count_flagged_posts > 0 and 'badge-danger' or 'badge-light'}" t-esc="forum.count_flagged_posts"/>
            </a>
        </t>
    </nav>
</template>


<!-- Specific Forum Layout -->
<template id="forum_index" name="Forum">
    <t t-call="website_forum.header">
        <div class="row no-gutters">
            <div t-attf-class="d-flex justify-content-end flex-md-grow-1 #{(search or tag or my) and 'col-12 flex-column-reverse flex-md-row mb-3' or 'col-md-auto order-md-3'}">
                <div t-if="search or tag or my" class="d-flex flex-wrap align-items-center flex-grow-1">
                    <span t-if="search" class="w-100 w-md-auto mb-2 mb-md-0 border rounded pl-2 d-inline-flex align-items-center justify-content-between">
                        <em class="bg-light px-2" t-esc="search"/>
                        <a t-att-href="url_for('') + '?' + keep_query( 'filters', 'sorting', 'my')" class="btn py-1">&#215;</a>
                    </span>
                    <span t-if="my" t-attf-class="w-100 w-md-auto mb-2 mb-md-0 border rounded pl-2 d-inline-flex align-items-center justify-content-between #{search and 'ml-md-2'}">
                        <div>
                            <img t-if="uid" class="o_forum_avatar rounded-circle mr-1" t-att-src="website.image_url(user, 'image_128', '16x16')" alt="Avatar"/>
                            <span t-if="my == 'favourites'"> My <b>Favourites</b></span>
                            <span t-elif="my == 'followed'"> I'm <b>Following</b></span>
                            <span t-elif="my == 'mine'"> My <b>Posts</b></span>
                            <span t-elif="my == 'tagged'"> <b>Tags</b> I Follow</span>
                        </div>
                        <a t-att-href="url_for('') + '?' + keep_query( 'search', 'filters', 'sorting')" class="btn py-1">&#215;</a>
                    </span>
                    <span t-if="tag" t-attf-class="w-100 w-md-auto mb-2 mb-md-0 border rounded pl-2 d-inline-flex align-items-center justify-content-between #{(search or my) and 'ml-md-2'}">
                        <div>
                            <span class="fa fa-tag text-muted mr-1"/>
                            <span t-esc="tag.name"/>
                        </div>
                        <a t-att-href="url_for('/forum/') + slug(forum) + '?' + keep_query( 'search', 'sorting', 'filters', 'my')" class="btn py-1">&#215;</a>
                    </span>
                </div>
                <div t-if="uid and request.env.user.forum_waiting_posts_count"
                    title="You already have a pending post"
                    data-toggle="popover" data-trigger="hover" data-content="Please wait for a moderator to validate your previous post before continuing.">
                    <a class="disabled btn btn-secondary btn-block mb-3 mb-md-0" t-attf-href="/forum/#{slug(forum)}/ask">New Post</a>
                </div>
                <a t-else="" role="button" type="button" class="btn btn-primary btn-block o_forum_ask_btn mb-3 mb-md-0" t-att-href="uid and '/forum/' + slug(forum) + '/ask' or '/web/login'">New Post</a>
            </div>

            <t t-set="no_filters" t-value="not filters in ('solved', 'unsolved', 'unanswered')"/>
            <t t-if="not no_filters or (no_filters and question_count)">
                <!-- Filter post by type (mobile only) -->
                <div class="col-6 col-md-auto d-lg-none d-flex align-items-center">
                    <div class="dropdown"> Show
                        <a href="#" class="dropdown-toggle" data-toggle="dropdown">
                            <t t-if="no_filters"> All</t>
                            <t t-elif="filters == 'solved'"> Solved</t>
                            <t t-elif="filters == 'unsolved'"> Unsolved</t>
                            <t t-elif="filters == 'unanswered'"> Unanswered</t>
                        </a>
                        <div class="dropdown-menu" role="menu">
                            <a t-att-href="url_for('') + '?' + keep_query( 'search', 'sorting', filters='all')"
                                class="dropdown-item">
                                All
                            </a>

                            <div class="dropdown-divider"/>
                            <t t-if="forum.mode == 'questions'">
                                <a t-att-href="url_for('') + '?' + keep_query( 'search', 'sorting', filters='solved')"
                                    class="dropdown-item">Solved
                                </a>
                                <a t-att-href="url_for('') + '?' + keep_query( 'search', 'sorting', filters='unsolved')"
                                    class="dropdown-item">Unsolved
                                </a>
                            </t>
                            <a t-att-href="url_for('') + '?' + keep_query( 'search', 'sorting', filters='unanswered')"
                                class="dropdown-item">Unanswered
                            </a>
                        </div>
                    </div>
                </div>

                <!-- Filter post by type (desktop) -->
                <div class="d-none d-lg-flex align-items-center col-auto flex-grow-md-1 flex-grow-lg-0">
                    <nav class="o_wforum_nav nav nav-pills justify-content-around">
                        <a t-att-href="url_for('') + '?' + keep_query('search', 'sorting', 'my', filters='all')"
                            t-attf-class="nav-link py-1 rounded-pill #{no_filters and 'active font-weight-bold' or 'pl-0 pr-2'}">
                            All
                        </a>
                        <t t-if="forum.mode == 'questions'">
                            <span class="mx-1 text-400 d-none d-lg-block">|</span>
                            <a t-att-href="url_for('') + '?' + keep_query( 'search', 'sorting', 'my', filters='solved')"
                                t-attf-class="nav-link py-1 rounded-pill #{filters == 'solved' and 'active font-weight-bold' or 'px-2'}">Solved
                            </a>
                            <a t-att-href="url_for('') + '?' + keep_query( 'search', 'sorting', 'my', filters='unsolved')"
                                t-attf-class="d-none d-lg-block nav-link py-1 rounded-pill #{filters == 'unsolved' and 'active font-weight-bold' or 'px-2'}">Unsolved
                            </a>
                        </t>
                        <span class="mx-1 text-400 d-none d-lg-block">|</span>
                        <a t-att-href="url_for('') + '?' + keep_query( 'search', 'sorting', 'my', filters='unanswered')"
                            t-attf-class="nav-link py-1 rounded-pill #{filters == 'unanswered' and 'active font-weight-bold' or 'px-2'}">Unanswered
                        </a>
                    </nav>
                </div>
            </t>

            <!-- Order by -->
            <div t-if="question_count > 1"
                t-attf-class="col-6 col-md-auto d-flex align-items-center justify-content-end #{uid and 'mt-lg-0'}">
                <span class="mx-3  mx-lg-2 text-400 d-none d-md-inline">|</span>
                <span class="dropdown">
                    Order by
                    <a href="#" class="dropdown-toggle" data-toggle="dropdown">
                        <t t-if="sorting == 'relevancy desc'"> trending</t>
                        <t t-elif="sorting == 'create_date desc'"> newest</t>
                        <t t-elif="sorting == 'write_date desc'"> activity date</t>
                        <t t-elif="sorting == 'child_count desc'"> most answered</t>
                        <t t-elif="sorting == 'vote_count desc'"> most voted</t>
                    </a>
                    <div class="dropdown-menu dropdown-menu-right" role="menu">
                        <a role="menuitem" t-att-href="url_for('') + '?' + keep_query( 'search', 'filters', sorting='relevancy desc')" t-attf-class="dropdown-item#{sorting == 'relevancy desc' and ' active'}">Trending</a>
                        <a role="menuitem" t-att-href="url_for('') + '?' + keep_query( 'search', 'filters', sorting='write_date desc')" t-attf-class="dropdown-item#{sorting == 'write_date desc' and ' active'}">Last activity date</a>
                        <a role="menuitem" t-att-href="url_for('') + '?' + keep_query( 'search', 'filters', sorting='create_date desc')" t-attf-class="dropdown-item#{sorting == 'create_date desc' and ' active'}">Newest</a>
                        <a role="menuitem" t-att-href="url_for('') + '?' + keep_query( 'search', 'filters', sorting='child_count desc')" t-attf-class="dropdown-item#{sorting == 'child_count desc' and ' active'}">Most answered</a>
                        <a role="menuitem" t-att-href="url_for('') + '?' + keep_query( 'search', 'filters', sorting='vote_count desc')" t-attf-class="dropdown-item#{sorting == 'vote_count desc' and ' active'}">Most voted</a>
                    </div>
                </span>
            </div>
        </div>

        <div class="row mt-4">
            <!-- List questions or search/filters result -->
            <div t-if="question_count != 0" class="col">
                <t t-foreach="question_ids" t-as="question">
                    <t t-call="website_forum.display_post">
                        <t t-set="show_author_avatar" t-value="true"/>
                    </t>
                </t>
            </div>

            <!-- No posts or search/filters result -->
            <div t-if="question_count == 0" class="col">
                <div t-if="search or tag or (not no_filters)" class="alert alert-info">
                    <t t-set="_filters_str">
                        <t t-if="filters == 'unanswered'">unanswered</t>
                        <t t-elif="filters == 'solved'">solved</t>
                        <t t-elif="filters == 'unsolved'">unsolved</t>
                    </t>
                    <t t-set="_my_str">
                        <t t-if="my == 'favourites'">in your favourites</t>
                        <t t-if="my == 'followed'">in your followed list</t>
                        <t t-if="my == 'mine'">in your posts</t>
                    </t>
                    <t t-set="_search_str"><t t-if="search">matching "<em class="font-weight-bold" t-esc="search"/>"</t></t>
                    <t t-set="_search_and_tag_str"><t t-if="search and tag">&amp;nbsp;and&amp;nbsp;</t></t>
                    <t t-set="_tag_str"><t t-if="tag">using the <span class="badge badge-light" t-esc="tag.name"/> tag</t></t>
                    <t t-set="result_msg">
                        Sorry, we could not find any <b>%s</b> result <b>
                        %s</b> %s%s%s.
                    </t>
                    <span t-raw="result_msg % (_filters_str.strip(), _my_str.strip(), _search_str.strip(), _search_and_tag_str.strip(), _tag_str.strip())"/>
                </div>

                <t t-elif="not tag and not search and no_filters">
                    <div t-if="my == 'followed'" class="alert border">
                        You're not following any topic in this forum (yet).<br/>
                        <a t-attf-href="/forum/#{slug(forum)}/">Browse All</a>
                    </div>
                    <div t-elif="my == 'favourites'" class="alert border">
                        No favourite questions in this forum (yet).<br/>
                        <a t-attf-href="/forum/#{slug(forum)}/">Browse All</a>
                    </div>
                    <div t-elif="my == 'mine'" class="alert border">You have no posts in this forum (yet).</div>
                </t>

                <div t-elif="filters == 'unanswered'" class="alert alert-info">Amazing! There are no unanswered questions left!</div>
                <div t-elif="no_filters and not search and not tag and not my" class="alert alert-info">
                    <b>This forum is empty.</b><br/>
                    Be the first one asking a question
                </div>

                <t t-if="search">
                    <h4>Search Tips</h4>
                    <ul>
                        <li>Check your spelling and try again</li>
                        <li>Try searching for one or two words</li>
                        <li>Be less specific in your wording for a wider search result</li>
                    </ul>
                </t>
            </div>
        </div>

        <t t-call="website.pager"/>
    </t>
</template>

<template id="404">
    <t t-call="website_forum.header">
        <div class="oe_structure oe_empty"/>
        <h1 class="mt-4">Question not found!</h1>
        <p>Sorry, this question is not available anymore.</p>
        <p>
            <a t-attf-href="/forum">Return to the question list.</a>
        </p>
    </t>
</template>

<!-- Edition: ask your question -->
<template id="new_question">
    <t t-call="website_forum.header">
        <div t-if="request.env.user.forum_waiting_posts_count" class="alert border" role="alert">
            <b>You already have a pending post.</b><br/>
            <p>Please wait for a moderator to validate your previous post before continuing.</p>
            <a t-attf-href="/forum/#{ slug(forum) }" title="All Topics"><i class="fa fa-chevron-left mr-2"/>Back to All Topics</a>
        </div>

        <form t-else="" t-attf-action="/forum/#{slug(forum)}/new" method="post" role="form" class="tag_text js_website_submit_form js_wforum_submit_form o_wforum_readable">
            <div class="form-group">
                <label for="content">Title</label>
                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                <input type="text" name="post_name" required="required" pattern=".*\S.*" t-attf-value="#{post_name}"
                    class="form-control form-control-lg" placeholder="A clear, explicit and concise title" title="Title must not be empty"/>
                <input type="hidden" name="karma" t-attf-value="#{user.karma}" id="karma"/>
                <div class="form-text small text-muted">
                    <a data-toggle="collapse" href="#newQuestionExample" role="button" aria-expanded="false" aria-controls="newQuestionExample">
                        Example
                        <i class="fa fa-question-circle"/>
                    </a>
                    <div class="collapse" id="newQuestionExample">
                        <div class="text-success mt-2">
                            <i class="fa fa-check"/> How to configure TPS and TVQ's canadian taxes?
                        </div>
                        <div class="text-danger">
                            <i class="fa fa-times"/> Good morning to all! Please, can someone help solve my tax computation problem in Canada? Thanks!
                        </div>
                    </div>
                </div>
            </div>
            <div class="form-group">
                <label for="content">Description</label>
                <div class="small form-text text-muted d-inline">, consider <b>adding an example</b>. </div>
                <textarea name="content" required="required" id="content" class="form-control o_wysiwyg_loader" t-att-data-karma="forum.karma_editor">
                    <t t-esc="question_content"/>
                </textarea>
            </div>
            <div class="form-group">
                <label for="post_tags">Tags</label>
                <input type="hidden" name="karma_tag_create" t-attf-value="#{forum.karma_tag_create}" id="karma_tag_create"/>
                <input type="hidden" name="karma_edit_retag" t-attf-value="#{forum.karma_edit_retag}" id="karma_edit_retag"/>
                <input type="hidden" name="post_tags" placeholder="Tags" class="form-control js_select2"/>
            </div>
            <div class="mb-5">
                <button type="submit" t-attf-class="btn btn-primary o_wforum_submit_post #{forum.allow_share and 'oe_social_share_call'} #{(user.karma &lt; forum.karma_ask) and 'karma_required'}"
                t-att-data-karma="forum.karma_ask"
                data-hashtags="#question" data-social-target-type="question">Post Your Question</button>
                <a class="btn btn-secondary" title="Back to Question" t-attf-href="/forum/#{ slug(forum) }"> Discard</a>
            </div>
        </form>
    </t>
</template>

<!-- Edition: edit a post -->
<template id="edit_post">
    <t t-call="website_forum.header">
        <div t-if="is_answer" class="font-weight-bold mb-1">Question by <t t-esc="post.parent_id.create_uid.sudo().name"/></div>
        <article t-if="is_answer" class="alert border pb-0 o_wforum_readable">
            <h5 class="mb-1 text-muted" t-esc="post.parent_id.name"/>
            <div t-field="post.parent_id.content" class="o_wforum_post_content text-muted oe_no_empty"/>
        </article>
        <form t-attf-action="/forum/#{slug(forum)}/post/#{slug(post)}/save" method="post" role="form" class="tag_text js_website_submit_form js_wforum_submit_form o_wforum_readable">
            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
            <div t-if="not is_answer" class="form-group">
                <label for="post_name">Title</label>
                <input type="text" name="post_name" required="required" pattern=".*\S.*" t-attf-value="#{post.name}"
                    class="form-control form-control-lg" placeholder="Edit your Post" title="Title must not be empty"/>
                <div class="form-text small text-muted">
                    <a data-toggle="collapse" href="#newQuestionExample" role="button" aria-expanded="false" aria-controls="newQuestionExample">
                        Example
                        <i class="fa fa-question-circle"/>
                    </a>
                    <div class="collapse" id="newQuestionExample">
                        <div class="my-2">Use a clear, explicit and concise title</div>
                        <div class="text-success">
                            <i class="fa fa-check"/> How to configure TPS and TVQ's canadian taxes?
                        </div>
                        <div class="text-danger">
                            <i class="fa fa-times"/> Good morning to all! Please, can someone help solve my tax computation problem in Canada? Thanks!
                        </div>
                    </div>
                </div>
            </div>
            <div class="form-group">
                <label t-if="not is_answer" for="content">Description</label>
                <label t-else="">Your Answer</label>
                <div t-if="not is_answer" class="small form-text text-muted d-inline">, consider <b>adding an example</b>. </div>
                <textarea name="content" id="content" required="required" class="form-control o_wysiwyg_loader" t-att-data-karma="forum.karma_editor">
                    <t t-esc="post.content"/>
                </textarea>
            </div>
                <input type="hidden" name="karma" t-attf-value="#{user.karma}" id="karma"/>
            <div t-if="not is_answer" class="form-group">
                <label for="post_tags">Tags</label>
                <input type="hidden" name="karma_tag_create" t-attf-value="#{forum.karma_tag_create}" id="karma_tag_create"/>
                <input type="hidden" name="karma_edit_retag" t-attf-value="#{forum.karma_edit_retag}" id="karma_edit_retag"/>
                <t t-set="edit_tags_karma_fail" t-value="user.karma &lt; forum.karma_edit_retag"/>
                <t t-set="edit_tags_karma_error_message">You need to have sufficient karma to edit tags</t>
                <input type="text" name="post_tags" class="form-control js_select2" placeholder="Tags" t-attf-data-init-value="#{tags}" value="Tags"
                       t-att-readonly="edit_tags_karma_fail and 'readonly'"
                       t-att-title="edit_tags_karma_fail and edit_tags_karma_error_message"/>
            </div>
            <div class="mb-4">
                <button type="submit" class="btn btn-primary o_wforum_submit_post">Save Changes</button>
                <a class="btn btn-secondary" title="Back to Question" t-attf-href="/forum/#{ slug(forum) }/#{ slug(post)}">
                    Discard
                </a>
            </div>
        </form>
    </t>
</template>

<!-- Moderation: close a post -->
<template id="close_post">
    <t t-call="website_forum.header">
        <p class="text-muted" t-if="not offensive">
            If you close this post, it will be hidden for most users. Only
            users having a high karma can see closed posts to moderate
            them.
        </p>
        <p class="text-muted" t-if="offensive">
            If you mark this post as offensive, it will be hidden for most users. Only
            users having a high karma can see offensive posts to moderate
            them.
        </p>
        <form t-attf-action="/forum/#{ slug(forum) }/#{offensive and 'post' or 'question'}/#{slug(question)}/#{offensive and 'mark_as_offensive' or 'close'}" method="post" role="form" class="mt32 mb64 js_website_submit_form">
            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
            <input name="post_id" t-att-value="question.id" type="hidden"/>
            <div class="form-group">
                <label for="post">Post:</label>
                <input type="text" disabled="True" class="form-control-plaintext" name="post" t-att-value="question.name if not question.parent_id else question.parent_id.name"/>
            </div>
            <div class="form-group mb-4">
                <label for="reason"><t t-if="offensive">Offensive</t><t t-if="not offensive">Closing</t> Reason:</label>
                <select class="form-control custom-select custom-select-lg" name="reason_id">
                    <t t-foreach="reasons or []" t-as="reason">
                        <option t-att-value="reason.id" t-att-selected="reason.id == question.closed_reason_id.id"><t t-esc="reason.name"/></option>
                    </t>
                </select>
            </div>
            <div class="form-group">
                <button type="submit" class="btn btn-danger">
                    <t t-if="offensive">Mark as offensive</t>
                    <t t-if="not offensive">Close post</t>
                </button>
                <span class="text-muted mx-3">or</span>
                <a role="button" class="btn btn-light border" t-attf-href="/forum/#{ slug(forum) }/#{ slug(question) }">Discard</a>
            </div>
        </form>
    </t>
</template>

<!-- Edition: post a reply -->
<template id="post_reply">
    <div class="css_editable_mode_hidden">
        <form class="collapse js_website_submit_form js_wforum_submit_form"
            t-attf-action="/forum/#{slug(forum)}/#{slug(object)}/reply" method="post" role="form">
            <h3 class="mt8">Your Reply</h3>
            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
            <input type="hidden" name="karma" t-attf-value="#{user.karma}" id="karma"/>
            <textarea name="content" class="form-control o_wysiwyg_loader" required="required" minlength="50" t-att-data-karma="forum.karma_editor"/>
            <button type="submit" class="btn btn-primary mb16 o_wforum_submit_post">Post Answer</button>
        </form>
    </div>
</template>

<!-- Edition: post an answer -->
<template id="post_answer">
    <div class="d-flex align-items-center mt-5 mb-2">
        <img t-if="uid" t-attf-class="o_forum_avatar rounded-circle mr-2" t-att-src="website.image_url(user, 'image_128', '24x24')" alt="Avatar"/>
        <h4 class="my-0">Your Answer</h4>
    </div>
    <t t-if="request.params.get('nocontent')">
        <p class="alert alert-danger" role="alert">You cannot post an empty answer</p>
    </t>
    <form t-attf-action="/forum/#{ slug(forum) }/#{slug(question)}/reply" method="post" class="js_website_submit_form js_wforum_submit_form" role="form">
        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
        <input type="hidden" name="karma" t-attf-value="#{user.karma}" id="karma"/>
        <textarea name="content" t-attf-id="content-#{str(question.id)}" class="form-control o_wysiwyg_loader" required="required" minlength="50" t-att-data-karma="forum.karma_editor"/>
        <p class="small mt-2 mb-1">
            <b>Please try to give a substantial answer.</b> If you wanted to comment on the question or answer, just
            <b>use the commenting tool.</b> Please remember that you can always <b>revise your answers</b>
            - no need to answer the same question twice. Also, please <b>don't forget to vote</b>
            - it really helps to select the best questions and answers!
        </p>
        <button type="submit" t-attf-class="btn btn-primary o_wforum_submit_post #{forum.allow_share and 'oe_social_share_call'} my-3 #{not question.can_answer and 'karma_required'}"
                t-att-data-karma="question.forum_id.karma_answer"
                data-social-target-type="answer" data-hashtags="#answer">Post Answer</button>
        <a href="#"
        class="btn btn-secondary"
        data-toggle="collapse"
        data-target=".answer_collapse"
        aria-expanded="false">Discard</a>
    </form>
</template>

<template id="vote">
    <t t-set="own_vote" t-value="post.user_vote"/>
    <t t-set="forum" t-value="post.forum_id"/>
    <t t-set="can_upvote" t-value="post.can_upvote"/>
    <t t-set="can_downvote" t-value="post.can_downvote"/>
    <div t-attf-class="vote text-center d-inline-flex align-items-center #{vertical and 'o_wforum_vote_vertical flex-md-column'} #{classes}">
        <button type="button" t-attf-data-href="/forum/#{slug(forum)}/post/#{slug(post)}/upvote"
            t-attf-class="btn btn-link vote_up fa fa-caret-up pl-0 #{vertical and 'px-md-0 pt-md-0' or 'pr-2'} #{own_vote == 1 and 'text-success' or 'text-muted'} #{not can_upvote and 'karma_required'}"
            t-att-disabled="own_vote == 1 and 'disabled'"
            t-att-data-karma="forum.karma_upvote"
            t-att-data-can-upvote="can_upvote"
            aria-label="Positive vote" title="Positive vote"/>
        <b t-attf-class="vote_count #{own_vote == 1 and 'text-success' or (own_vote == -1 and 'text-danger' or 'text-muted')}"
           t-esc="post.vote_count"/>
        <button type="button" t-attf-data-href="/forum/#{slug(forum)}/post/#{slug(post)}/downvote"
            t-attf-class="btn btn-link vote_down fa fa-caret-down #{vertical and 'px-md-0' or 'px-2'} #{own_vote == -1 and 'text-danger' or 'text-muted'} #{not can_downvote and 'karma_required'}"
            t-att-disabled="own_vote == -1 and 'disabled'"
            t-att-data-karma="forum.karma_downvote"
            t-att-data-can-downvote="can_downvote"
            aria-label="Negative vote" title="Negative vote"/>
        <t t-raw="0"/>
    </div>
</template>

<!-- Specific Post Layout -->
<template id="post_description_full" name="Question Navigation">
    <t t-call="website_forum.header">
        <div class="alert alert-info shadow-sm pb-3" role="status" t-if="forum and question.state == 'pending' and user.karma>=forum.karma_moderate and question.active">
            <p>This post is currently awaiting moderation and it's not published yet.<br/>
                Do you want <b>Accept</b> or <b>Reject</b> this post ?</p>
            <div>
                <a role="button" t-attf-href="/forum/#{slug(forum)}/post/#{slug(question)}/validate" type="button" class="btn btn-success">
                    <i class="fa fa-check fa-fw mr-1"/>Accept</a>
                <a role="button" t-attf-href="/forum/#{slug(forum)}/post/#{slug(question)}/refuse" type="button" class="btn btn-danger">
                    <i class="fa fa-times fa-fw mr-1"/>Reject</a>
            </div>
        </div>

        <div class="alert alert-warning text-center" role="status"
            t-if="question.state == 'pending' and user.karma &lt; forum.karma_moderate">
            Waiting for validation
        </div>

        <article t-attf-class="question o_wforum_post row no-gutters #{can_bump and 'oe_js_bump'}"
            data-type="question"
            t-att-data-last-update="question.write_date"
            t-att-data-id="question.id"
            t-att-data-state="question.state">

            <div t-if="question.state == 'active'" class="col-6 mb-3 col-md-auto pr-2 pr-md-3 pr-lg-4">
                <t t-call="website_forum.vote">
                    <t t-set="post" t-value="question"/>
                    <t t-set="vertical" t-value="True"/>
                </t>
            </div>
            <div t-attf-class="text-right d-md-none #{question.state == 'active' and 'col-6' or 'col-12'}">
                <t t-call="website_forum.question_dropdown"/>
            </div>
            <section class="col">
                <div class="row no-gutter">
                    <header class="o_wforum_post_header col mb-0 h2">
                        <i t-if="uid" aria-label="Toggle favorite status"
                            title="Toggle favorite status"
                            t-attf-data-href="/forum/#{slug(question.forum_id)}/question/#{slug(question)}/toggle_favourite"
                            t-attf-class="o_wforum_favourite_toggle no-decoration small mr-1 fa #{question.user_favourite and 'fa-star o_wforum_gold' or 'fa-star-o text-muted'}"/>

                        <h1 class="d-inline mb-0 h2" t-esc="question.name"/>

                        <span t-if="not question.active" class="border rounded-pill h6 text-muted my-0 ml-2 px-2">
                            <t t-if="question.state!='offensive'">Deleted</t>
                            <t t-if="question.state=='offensive'">Offensive</t>
                            <t t-if="question.state=='offensive' and question.closed_reason_id">
                                <t t-esc="question.closed_reason_id.name[0].upper() + question.closed_reason_id.name[1:]"/>
                            </t>
                        </span>
                        <small t-if="question.state == 'close'">
                            <span class="badge badge-info">Closed</span>
                        </small>
                    </header>
                    <div class="d-none d-md-block col-md-auto">
                        <t t-call="website_forum.question_dropdown"/>
                    </div>
                </div>

                <div class="mt-3 row">
                    <div t-call="website_forum.author_box" t-attf-class="col mb-2 #{question.tag_ids and 'col-sm-auto'}">
                        <t t-set="object" t-value="question"/>
                        <t t-set="allow_biography" t-value="True"/>
                        <t t-set="show_name" t-value="True"/>
                        <t t-set="show_date" t-value="True"/>
                    </div>
                    <div class="col-auto mb-2 order-sm-3">
                        <div t-call="website_mail.follow">
                            <t t-set="object" t-value="question"/>
                            <t t-set="icons_design" t-value="True"/>
                        </div>
                    </div>
                    <div t-if="len(question.tag_ids) > 0" class="col-12 col-sm order-sm-2 o_wforum_index_entry_tags mb-1">
                        <i class="fa fa-tag text-muted"/>
                        <a t-foreach="question.tag_ids" t-as="question_tag"
                            t-attf-href="/forum/#{slug(question_tag.forum_id)}/tag/#{slug(question_tag)}/questions?#{keep_query(filters='tag')}"
                            t-attf-class="badge border text-600 badge-light #{not question_tag_first and 'ml-1'}"
                            t-field="question_tag.name"/>
                    </div>
                </div>

                <div class="alert alert-info text-center" t-if="question.state == 'close'" role="status">
                    <p class="mt-3">
                        <b>The question has been closed<t t-if="question.closed_reason_id"> for reason: <i t-esc="question.closed_reason_id.name"/></t></b>
                    </p>
                    <t t-if="question.closed_uid">
                        <b> by <a t-attf-href="/forum/#{ slug(forum) }/user/#{ question.closed_uid.id }"
                            t-field="question.closed_uid"
                            t-options='{"widget": "contact", "fields": ["name"]}'
                            style="display: inline-block;"/></b>
                    </t>
                    <b> on <span t-field="question.closed_date"/></b>
                    <div class="mt-3 mb24 text-center">
                        <t t-call="website_forum.link_button">
                            <t t-set="url" t-value="'/forum/' + slug(forum) + '/question/' + slug(question) + '/reopen'"/>
                            <t t-set="label">Reopen</t>
                            <t t-set="inDropdown" t-value="False"/>
                            <t t-set="icon" t-value="'fa-arrow-right'"/>
                            <t t-set="karma" t-value="not question.can_close and question.karma_close or 0"/>
                        </t>
                    </div>
                </div>

                <div t-field="question.content" class="o_wforum_post_content o_wforum_readable oe_no_empty o_not_editable"/>

                <t t-set="_question_comment_collapse_uid" t-value="'comment_%s_%s' % (question._name.replace('.', '_'), question.id)"/>
                <div t-if="question.state == 'active' and question.active != False"
                    class="btn-toolbar mb-3" role="toolbar">
                    <div t-if="not question.uid_has_answered and question.can_answer" class="btn-group btn-group-sm mr-2">
                        <a t-attf-class="btn btn-primary collapsed #{not question.can_answer and 'karma_required text-muted'}"
                            t-att-data-karma="question.forum_id.karma_answer"
                            data-toggle="collapse"
                            data-target=".answer_collapse"
                            href="#">
                            <i class="fa fa-reply mr-1"/>Answer
                        </a>
                    </div>
                    <div class="btn-group btn-group-sm">
                        <a t-attf-class="btn px-2 border #{not question.can_comment and 'karma_required text-muted'}" t-att-data-karma="question.karma_comment" t-att-data-toggle="question.can_comment and 'collapse' or None"
                            t-attf-href="##{_question_comment_collapse_uid}">
                            <i class=" fa fa-comment text-muted mr-1"/>Comment
                        </a>
                        <a href="javascript:void(0)" class="oe_social_share btn px-2 border"
                            t-attf-data-hashtags="#question">
                            <i class="fa fa-share-alt text-muted mr-1"/>Share
                        </a>
                    </div>
                </div>

                <t t-call="website_forum.post_comment">
                    <t t-set="object" t-value="question"/>
                    <t t-set="_collapse_uid" t-value="_question_comment_collapse_uid"/>
                </t>

                <section t-if="question.child_count" t-attf-class="#{question.website_message_ids and 'mt-5' or 'mt-4'}">
                    <h5>
                        <t t-esc="question.child_count"/>
                        <t t-if="question.child_count == 1">Answer</t>
                        <t t-else="">Answers</t>
                    </h5>

                    <t t-foreach="question.child_ids" t-as="post_answer">
                        <div t-if="post_answer.state != 'flagged' or (post_answer.state == 'flagged' and post_answer.can_moderate)"
                            class="mb-4">
                            <t t-call="website_forum.post_answers">
                                <t t-set="answer" t-value="post_answer"/>
                            </t>
                        </div>
                    </t>
                </section>

                <div t-if="not question.uid_has_answered and question.can_answer and question.child_ids" class="btn-group btn-group-sm mr-2">
                    <a t-attf-class="btn btn-primary answer_collapse show collapsed collapse #{not question.can_answer and 'karma_required text-muted'}"
                        t-att-data-karma="question.forum_id.karma_answer"
                        data-toggle="collapse"
                        data-target=".answer_collapse"
                        href="#">
                        <i class="fa fa-reply mr-1"/>Answer
                    </a>
                </div>

                <t t-if="question.state != 'close' and question.active != False and question.can_answer and not request.env.user.forum_waiting_posts_count">
                    <div id="post_reply" class="collapse answer_collapse" t-if="(not question.uid_has_answered or question.forum_id.mode == 'discussions')">
                        <t t-call="website_forum.post_answer"/>
                    </div>
                    <div t-elif="uid and request.env.user.forum_waiting_posts_count" class="alert alert-info text-center">
                        <b class="d-block">You have a pending post</b>
                        Please wait for a moderator to validate your previous post to be allowed replying questions.
                    </div>
                    <div t-elif="not uid" class="alert alert-info text-center">
                        <a class="btn btn-primary forum_register_url" href="/web/login">Sign in</a> to partecipate
                    </div>
                </t>
            </section>
        </article>
    </t>
</template>


<template id="website_forum.question_dropdown">
    <div class="dropdown">
        <a class="btn py-0" href="#" role="button" id="dropdownMenuLink" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
            <i class="fa fa-ellipsis-v"/>
        </a>
        <div class="dropdown-menu dropdown-menu-right" aria-labelledby="dropdownMenuLink">
            <t t-if="question.state == 'close'" t-call="website_forum.link_button">
                <t t-set="url" t-value="'/forum/' + slug(forum) +'/question/' + slug(question) + '/reopen'"/>
                <t t-set="label">Reopen</t>
                <t t-set="inDropdown" t-value="True"/>
                <t t-set="icon" t-value="'fa-undo fa-fw'"/>
                <t t-set="karma" t-value="not question.can_close and question.karma_close or 0"/>
            </t>
            <t t-call="website_forum.link_button">
                <t t-set="url" t-value="'/forum/' + slug(forum) +'/post/' + slug(question) + '/edit'"/>
                <t t-set="label">Edit</t>
                <t t-set="inDropdown" t-value="True"/>
                <t t-set="icon" t-value="'fa-pencil fa-fw'"/>
                <t t-set="karma" t-value="not question.can_edit and question.karma_edit or 0"/>
            </t>
            <t t-if="question.state != 'close'" t-call="website_forum.link_button">
                <t t-set="url" t-value="'/forum/' + slug(forum) +'/question/' + slug(question) + '/ask_for_close'"/>
                <t t-set="label">Close</t>
                <t t-set="inDropdown" t-value="True"/>
                <t t-set="icon" t-value="'fa-times fa-fw'"/>
                <t t-set="karma" t-value="not question.can_close and question.karma_close or 0"/>
            </t>
            <t t-if="question.active" t-call="website_forum.link_button">
                <t t-set="url" t-value="'/forum/' + slug(forum) +'/question/' + slug(question) + '/delete'"/>
                <t t-set="label">Delete</t>
                <t t-set="inDropdown" t-value="True"/>
                <t t-set="icon" t-value="'fa-trash-o fa-fw'"/>
                <t t-set="karma" t-value="not question.can_unlink and question.karma_unlink or 0"/>
            </t>
            <t t-if="not question.active and question.state != 'offensive'" t-call="website_forum.link_button">
                <t t-set="url" t-value="'/forum/' + slug(forum) +'/question/' + slug(question) + '/undelete'"/>
                <t t-set="label">Undelete</t>
                <t t-set="inDropdown" t-value="True"/>
                <t t-set="icon" t-value="'fa-upload fa-fw'"/>
                <t t-set="karma" t-value="not question.can_unlink and question.karma_unlink or 0"/>
            </t>
            <t t-if="not question.active and question.state == 'offensive'" t-call="website_forum.link_button">
                <t t-set="url" t-value="'/forum/' + slug(forum) +'/post/' + slug(question) + '/validate'"/>
                <t t-set="label">Validate</t>
                <t t-set="inDropdown" t-value="True"/>
                <t t-set="icon" t-value="'fa-check fa-fw'"/>
                <t t-set="karma" t-value="not question.can_moderate and question.forum_id.karma_moderate or 0"/>
            </t>
            <t t-if="question.active" href="#" t-call="website_forum.link_button">
                <t t-set="url" t-value="'/forum/' + slug(forum) +'/post/' + slug(question) + '/flag'"/>
                <t t-set="label">Flag</t>
                <t t-set="inDropdown" t-value="True"/>
                <t t-set="icon" t-value="not question.can_flag and 'fa-flag-o' or 'fa-flag'"/>
                <t t-set="karma" t-value="not question.can_flag and question.forum_id.karma_flag or 0"/>
                <t t-set="classes" t-value="'flag'"/>
            </t>
        </div>
    </div>
</template>

<template id="post_answers">
    <a t-attf-id="answer-#{str(answer.id)}"/>
    <div t-attf-class="forum_answer pt-4 border-top border-light #{answer.is_correct and 'o_wforum_answer_correct'}" t-attf-id="answer_#{answer.id}" >
        <div class="">
            <div class="row no-gutters">
                <div class="col-12 col-md-auto pr-2 pr-md-2 pr-lg-3">
                    <t t-call="website_forum.vote">
                        <t t-set="vertical" t-value="True"/>
                        <t t-set="post" t-value="answer"/>

                        <t t-set="helper_accept">Mark as Best Answer</t>
                        <t t-set="helper_decline">Unmark as Best Answer</t>
                        <a t-if="question.can_answer and question.forum_id.mode == 'questions'" t-attf-class="o_wforum_validate_toggler fa-stack mt-2 #{not answer.can_accept and 'karma_required'}"
                           href="#"
                           t-attf-data-karma="#{answer.karma_accept}"
                           t-att-data-helper-accept="helper_accept"
                           t-att-data-helper-decline="helper_decline"
                           t-att-title="answer.is_correct and helper_decline or helper_accept"
                           data-toggle="tooltip"
                           t-attf-data-target="#answer_#{answer.id}"
                           t-attf-data-href="/forum/#{slug(question.forum_id)}/post/#{slug(answer)}/toggle_correct">
                            <i class="fa fa-circle-o fa-stack-2x"/>
                            <i class="fa fa-check fa-stack-1x"/>
                        </a>
                    </t>
                </div>
                <div class="col">
                    <div class="o_wforum_answer_header d-flex align-items-start mb-2">
                        <t t-call="website_forum.author_box">
                            <t t-set="object" t-value="answer"/>
                            <t t-set="show_name" t-value="True"/>
                            <t t-set="show_date" t-value="True"/>
                            <t t-set="allow_biography" t-value="True"/>
                            <t t-set="object_validable" t-value="True"/>
                        </t>
                        <span class="o_wforum_answer_correct_badge border small border-success rounded-pill font-weight-bold text-success ml-2 px-2">
                            Best Answer
                        </span>
                    </div>

                    <div t-field="answer.content" class="mb-2 o_wforum_readable oe_no_empty o_not_editable"/>

                    <t t-set="_answer_comment_collapse_uid" t-value="'comment_%s_%s' % (answer._name.replace('.', '_'), answer.id)"/>
                    <div class="btn-toolbar mb-3">
                        <div t-if="question.uid_has_answered and answer.create_uid.id == uid"
                            class="btn-group btn-group-sm mr-1 mr-md-2">
                            <a class="btn btn-sm px-2 btn-primary"
                                title="Only one answer per question is allowed"
                                data-toggle="tooltip"
                                t-attf-href="/forum/#{slug(forum)}/question/#{slug(question)}/edit_answer">
                                <i class="fa fa-pencil"/>
                                Edit<span class="d-none d-lg-inline"> your answer</span>
                            </a>
                        </div>
                        <div class="btn-group btn-group-sm">
                            <a t-attf-class="btn border px-2 #{not answer.can_comment and 'karma_required text-muted'}"
                                t-attf-data-karma="#{not answer.can_comment and answer.karma_comment or 0}"
                                t-att-data-toggle="answer.can_comment and 'collapse' or None"
                                t-attf-data-target="##{_answer_comment_collapse_uid}">
                                <i t-attf-class="fa fa-comment text-muted #{not answer.can_comment and 'karma_required'}"/>
                                Comment
                            </a>
                            <a href="javascript:void(0)" class="oe_social_share btn border px-2"
                                t-attf-data-urlshare="#{request.httprequest.url}##{_answer_comment_collapse_uid}"
                                t-attf-data-hashtags="question">
                                <i class="fa fa-share-alt text-muted"/>
                                Share
                            </a>
                        </div>
                        <div t-if="answer.can_comment" class="btn-group btn-group-sm ml-1 ml-md-2">
                            <a class="btn border dropdown-toggle" type="button" id="dropdownMenuButton" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                                More
                            </a>
                            <div class="dropdown-menu shadow">
                                <t t-if="not answer.create_uid.id == uid" t-call="website_forum.link_button">
                                    <t t-set="url" t-value="'/forum/' + slug(forum) + '/post/' + slug(answer) + '/edit'"/>
                                    <t t-set="label">Edit</t>
                                    <t t-set="inDropdown" t-value="True"/>
                                    <t t-set="icon" t-value="'fa-pencil-square-o text-muted mr-2'"/>
                                    <t t-set="karma" t-value="not answer.can_edit and answer.karma_edit or 0"/>
                                </t>
                                <t t-call="website_forum.link_button">
                                    <t t-set="url" t-value="'/forum/' + slug(forum) + '/post/' + slug(answer) + '/delete'"/>
                                    <t t-set="label">Delete</t>
                                    <t t-set="inDropdown" t-value="True"/>
                                    <t t-set="icon" t-value="'fa-trash-o text-muted mr-2'"/>
                                    <t t-set="karma" t-value="not answer.can_unlink and answer.karma_unlink or 0"/>
                                </t>
                                <t t-if="answer.can_flag" t-call="website_forum.link_button">
                                    <t t-set="url" t-value="'/forum/' + slug(forum) + '/post/' + slug(answer) + '/flag'"/>
                                    <t t-set="label" t-value="answer.state == 'flagged' and 'Flagged' or 'Flag'"/>
                                    <t t-set="form_method" t-value="'GET'"/>
                                    <t t-set="inDropdown" t-value="True"/>
                                    <t t-set="karma" t-value="not answer.can_flag and answer.forum_id.karma_flag or 0"/>
                                    <t t-set="icon" t-value="'fa-flag-o text-muted mr-2'"/>
                                    <t t-set="form_classes" t-value="'flag'"/>
                                </t>
                                <t t-call="website_forum.link_button">
                                    <t t-set="url" t-value="'/forum/' + slug(forum) + '/post/' + slug(answer) + '/convert_to_comment'"/>
                                    <t t-set="label">Convert as a comment</t>
                                    <t t-set="inDropdown" t-value="True"/>
                                    <t t-set="icon" t-value="'fa-magic text-muted mr-2'"/>
                                    <t t-set="karma" t-value="not answer.can_comment_convert and answer.karma_comment_convert or 0"/>
                                </t>
                            </div>
                        </div>
                    </div>
                    <t t-call="website_forum.post_comment">
                        <t t-set="object" t-value="answer"/>
                        <t t-set="_collapse_uid" t-value="_answer_comment_collapse_uid"/>
                    </t>
                    <div t-foreach="answer.child_ids" t-as="child_answer" class="mt4 mb4">
                       <t t-call="website_forum.post_answers">
                           <t t-set="answer" t-value="child_answer"/>
                       </t>
                   </div>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="website_forum.author_box">
    <t t-set="display_info" t-value="show_name or show_date or show_karma"/>
    <t t-if="allow_biography and object.can_display_biography" t-set="bio_popover_data">
        <div class="d-flex o_wforum_bio_popover_wrap">
            <img class="o_forum_avatar_big flex-shrink-0 mr-3" t-att-src="website.image_url(object.create_uid, 'image_128', '75x75')" alt="Avatar"/>
            <div>
                <h5 class="o_wforum_bio_popover_name mb-0" t-field="object.create_uid" t-options='{"widget": "contact", "country_image": True, "fields": ["name", "country_id"]}'/>

                <span class="o_wforum_bio_popover_info" t-field="object.create_uid" t-options='{"widget": "contact", "UserBio": True, "badges": True, "fields": ["karma"]}'/>
                <div class="o_wforum_bio_popover_bio" t-field="object.create_uid" t-options='{"widget": "contact", "website_description": True, "fields": ["partner_id"]}'/>
            </div>
        </div>
    </t>

    <div t-attf-class="o_wforum_author_box d-inline-flex #{display_info and 'o_show_info'} #{compact and 'o_compact align-items-center'} #{bio_popover_data and 'o_wforum_bio_popover'}"
         t-att-data-content="bio_popover_data">
        <t t-set="user_profile_url" t-valuef="#"/>
        <t t-if="object.create_uid.id == request.session.uid or object.create_uid.sudo().website_published">
            <t t-set="user_profile_url" t-value="'/forum/%s/user/%s' % (slug(forum), object.create_uid.id) + '?forum_origin=' + request.httprequest.path"/>
        </t>

        <a t-att-href="user_profile_url" class="o_wforum_author_pic position-relative rounded-circle">
            <span t-if="object_validable" class="o_wforum_author_box_check rounded-circle bg-success position-absolute">
                <i class="fa fa-check fa-fw small text-white"/>
            </span>
            <img t-attf-class="rounded-circle o_forum_avatar #{not display_info and 'shadow'}" t-att-src="website.image_url(object.create_uid, 'image_128', '40x40')" alt="Avatar"/>
        </a>

        <div t-if="show_name or show_date or show_karma" t-attf-class="d-flex #{compact and 'align-items-baseline ml-1' or 'flex-column justify-content-around ml-2'}">
            <a t-att-href="user_profile_url" class="h6 my-0 text-reset" t-field="object.create_uid" t-options='{"widget": "contact", "fields": ["name"]}'/>
            <small t-if="show_karma and show_date" class="text-muted font-weight-bold"> - <t t-esc="object.create_uid.karma"/>xp</small>

            <div t-attf-class="text-muted small font-weight-bold #{compact and 'd-flex align-items-baseline'}">
                <span t-if="compact" class="mx-1"> - </span>
                <time t-if="show_date" class="d-block text-muted font-weight-bold" t-field="object.create_date" t-options='{"format": "d MMMM y"}'/>
                <span t-if="show_karma and not show_date" class="text-muted font-weight-bold"><t t-esc="object.create_uid.karma"/>xp</span>
            </div>
        </div>
    </div>
</template>

<!-- Utility template: Post a Comment -->
<template id="post_comment">
    <div class="o_wforum_post_comments_container ml-2 ml-md-5">
        <div t-if="len(object.website_message_ids)" class="o_wforum_comments_count_header mb-2">
            <div class="text-muted font-weight-bold small">
                <span class="o_wforum_comments_count" t-esc="len(object.website_message_ids)"/>
                <t t-if="len(object.website_message_ids) == 1">Comment</t>
                <t t-else="">Comments</t>
            </div>
        </div>
        <div class="css_editable_mode_hidden o_wforum_readable">
            <form t-att-id="_collapse_uid" class="collapse p-1 oe_comment_grey js_website_submit_form js_wforum_submit_form"
                t-attf-action="/forum/#{slug(forum)}/post/#{slug(object)}/comment" method="POST">
                <div class="shadow bg-white rounded px-3 pt-3 pb-4 mb-4">
                    <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                    <input name="post_id" t-att-value="object.id" type="hidden" class="mt8"/>
                    <div class="d-flex w-100">
                        <img class="d-none d-md-inline-block rounded-circle o_forum_avatar mr-3" t-att-src="website.image_url(user, 'image_128', '40x40')" alt="Avatar"/>
                        <div class="w-100">
                            <textarea name="comment" class="form-control mb-2" placeholder="Comment this post..."/>
                            <div>
                                <button type="submit" class="btn btn-primary o_wforum_submit_post">Post Comment</button>
                                <a t-attf-href="##{_collapse_uid}" data-toggle="collapse" class="btn border">Discard</a>
                            </div>
                        </div>
                    </div>
               </div>
            </form>
        </div>

        <div class="o_wforum_post_comments pl-3 pl-md-4 border-left">
            <t t-foreach="reversed(object.website_message_ids)" t-as="message">
                <div t-attf-class="o_wforum_post_comment #{not message_first and 'mt-3'}">
                    <div>
                        <t t-set="required_karma" t-value="message.author_id.id == user.partner_id.id and object.forum_id.karma_comment_unlink_own or object.forum_id.karma_comment_unlink_all"/>
                        <t t-set="required_karma" t-value="message.author_id.id == user.partner_id.id and object.forum_id.karma_comment_convert_own or object.forum_id.karma_comment_convert_all"/>
                        <t t-if="(object.parent_id and object.parent_id.state != 'close' and object.parent_id.active != False) or (not object.parent_id and object.state != 'close' and object.active != False)">
                            <t t-set="allow_post_comment" t-value="True" />
                        </t>
                        <div class="d-flex">
                            <div class="mb-1" t-call="website_forum.author_box">
                                <t t-set="object" t-value="message"/>
                                <t t-set="compact" t-value="True"/>
                                <t t-set="show_name" t-value="True"/>
                                <t t-set="show_date" t-value="True"/>
                            </div>
                            <div class="dropdown ml-2">
                                <a class="btn btn-sm" type="button" id="dropdownMenuButton" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                                    <i class="fa fa-ellipsis-v"/>
                                </a>
                                <div class="dropdown-menu shadow">
                                    <t t-call="website_forum.link_button">
                                        <t t-set="url" t-value="'/forum/' + slug(forum) + '/post/' + slug(object) + '/comment/' + slug(message) + '/delete'"/>
                                        <t t-set="label">Delete</t>
                                        <t t-set="title">Delete</t>
                                        <t t-set="inDropdown" t-value="True"/>
                                        <t t-set="icon" t-value="'fa-trash-o text-muted'"/>
                                        <t t-set="classes" t-value="'comment_delete'"/>
                                        <t t-set="karma" t-value="not object.can_unlink and object.karma_unlink or 0"/>
                                    </t>
                                    <t t-call="website_forum.link_button">
                                        <t t-set="url" t-value="'/forum/' + slug(forum) + '/post/' + slug(object) + '/comment/' + slug(message) +  '/convert_to_answer'"/>
                                        <t t-set="label">Convert as a answer</t>
                                        <t t-set="inDropdown" t-value="True"/>
                                        <t t-set="icon" t-value="'fa-magic text-muted'"/>
                                        <t t-set="karma" t-value="not object.can_comment_convert and object.karma_comment_convert or 0"/>
                                    </t>
                                </div>
                            </div>
                        </div>
                        <div t-field="message.body" class="o_wforum_readable oe_no_empty"/>
                    </div>
                </div>
            </t>
        </div>
    </div>
</template>

<template id="tag" name="Forum Tags">
    <t t-call="website_forum.header">
        <nav t-if="pager_tag_chars" class="navbar navbar-light bg-light justify-content-start">
            <t t-if="len(pager_tag_chars) &lt; 11">
                <span class="navbar-text mr-3">Show Tags Starting By</span>
                <ul class="pagination mt0 mb0">
                    <t t-foreach="pager_tag_chars" t-as="tuple_char">
                        <t t-if="tuple_char_index &lt; 11">
                            <li t-attf-class="page-item #{active_char_tag == tuple_char[1] and 'active'}"><a t-attf-href="/forum/#{slug(forum)}/tag/#{quote_plus(tuple_char[1])}" class="page-link"><t t-esc="tuple_char[0]"/></a></li>
                        </t>
                    </t>
                </ul>
            </t>
            <div t-else="" class="form-inline" role="toolbar" aria-label="Toolbar with button groups">
                <label for="filter" class="my-1 mr-2">Show Tags Starting By</label>
                <select name="filter" class="custom-select" onchange="location = this.value;">
                    <t t-foreach="pager_tag_chars" t-as="tuple_char">
                        <option t-if="active_char_tag == tuple_char[1]" selected="selected" value="" t-esc="tuple_char[0]"/>
                        <option t-else="" t-attf-value="/forum/#{slug(forum)}/tag/#{quote_plus(tuple_char[1])}" t-esc="tuple_char[0]"/>
                    </t>
                </select>
            </div>
        </nav>

        <div class="row mb-5" t-if="tags">
            <div class="col-md-3 mt16 o_js_forum_tag_follow" t-foreach="tags" t-as="tag">
                <span t-attf-class="badge border px-2 #{tag.message_is_follower and 'border-success text-success' or 'badge-light text-600'}">
                    <i class="fa fa-tag small"/>
                    <t t-esc="tag.name"/>
                    <b class="small align-top">(<t t-esc="tag.posts_count"/>)</b>
                </span>
                <div class="o_forum_tag_follow_box text-center">
                    <div class="card shadow mt-2">
                        <a t-attf-href="/forum/#{ slug(forum) }/tag/#{ slug(tag) }/questions?{{ keep_query( filters='tag') }}"
                            class="btn btn-light">
                            See <t t-esc="tag.posts_count"/> post<t t-if="tag.posts_count > 1">s</t>
                        </a>
                        <em class="d-block mb-2">or</em>
                        <div class="input-group">
                            <t t-call="website_mail.follow">
                                <t t-set="email" t-value="user_id.email"/>
                                <t t-set="object" t-value="tag"/>
                            </t>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div t-else="" class="alert border text-center">
            No tags
        </div>
    </t>
</template>

<template id="moderation_queue" name="Forum Moderation Queue">
    <t t-call="website_forum.header">
        <t t-set="website_forum_action" t-value="'o_wforum_moderation_queue'"/>
        <div t-if="len(posts_ids) > 0" class="mb-2 text-right">
            <button type="button" class="btn btn-primary" data-toggle="modal" data-target="#markAllAsSpam"><i class="fa fa-bug"></i> Filter Tool</button>
        </div>
        <div t-attf-class="o_caught_up_alert alert text-center #{len(posts_ids) and 'd-none'}">
            <i class="fa fa-check text-success d-block display-2"></i>
            <b>You've Completely Caught Up!</b><br/>
            <t t-if="queue_type == 'validation'">No post to be validated</t>
            <t t-if="queue_type == 'flagged'">No flagged posts</t>
        </div>

        <div class="modal fade" t-att-data-spam-ids="str(posts_ids.ids)" id="markAllAsSpam" tabindex="-1" role="dialog" aria-labelledby="markAllAsSpam" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header d-flex align-items-center pb-0">
                        <div class="text-muted mr-2">Filter by:</div>
                        <ul class="nav nav-tabs border-bottom-0" id="myTab" role="tablist">
                            <li class="nav-item">
                                <a class="nav-link active spam_menu" id="user-tab" data-toggle="tab" href="#spam_user" role="tab" aria-controls="user" aria-selected="true"><i class="fa fa-user"/> User</a>
                            </li>
                            <li class="nav-item">
                                <a class="nav-link spam_menu" id="country-tab" data-toggle="tab" href="#spam_country" role="tab" aria-controls="spam_country" aria-selected="false"><i class="fa fa-flag"/> Country</a>
                            </li>
                            <li class="nav-item">
                                <a class="nav-link spam_menu" id="character-tab" data-toggle="tab" href="#spam_character" role="tab" aria-controls="spam_character" aria-selected="false">圾 Text</a>
                            </li>
                        </ul>
                        <button type="button" class="close align-self-start" data-dismiss="modal"><span aria-label="Close">×</span></button>
                    </div>
                    <div class="modal-body bg-100">
                        <div class="tab-content" id="o_tab_content_spam">
                            <div class="tab-pane fade show active" data-key="create_uid" id="spam_user" role="tabpanel" aria-labelledby="user-tab">
                                <form class="row" >
                                    <!-- Prevent the foreach loop to overide the `user` variable from the controller -->
                                    <t t-set="env_user" t-value="user"/>
                                    <div t-foreach="posts_ids.mapped('create_uid')" t-as="user" class="col-6">
                                        <div class="card mb-2">
                                            <div class="card-body py-2">
                                                <div class="custom-control custom-checkbox">
                                                    <input type="checkbox" t-att-value="user.id" class="custom-control-input" t-attf-id="user_#{user.id}"/>
                                                    <label class="custom-control-label" t-attf-for="user_#{user.id}">
                                                        <img class="d-inline img o_forum_avatar" t-att-src="website.image_url(user, 'image_128', '40x40')" alt="Avatar"/>
                                                        <span t-esc="user.name" class="d-inline"/>
                                                    </label>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                    <t t-set="user" t-value="env_user"/>
                                </form>
                            </div>
                            <div class="tab-pane fade" data-key="country_id" id="spam_country" role="tabpanel" aria-labelledby="country-tab">
                                <form class="row">
                                    <div t-foreach="posts_ids.mapped('create_uid.country_id')" t-as="country" class="col-6">
                                        <div class="card mb-2">
                                            <div class="card-body py-2">
                                                <div class="custom-control custom-checkbox">
                                                    <input type="checkbox" class="custom-control-input" t-attf-id="country_#{country.id}" t-att-value="country.id"/>
                                                    <label class="custom-control-label" t-attf-for="country_#{country.id}">
                                                        <span t-field="country.image_url" t-options='{"widget": "image_url", "class": "country_flag"}' class="mr-2"/>
                                                        <span t-esc="country.name"/>
                                                    </label>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </form>
                            </div>
                            <div class="tab-pane fade" data-key="post_id" id="spam_character" role="tabpanel" aria-labelledby="character-tab">
                                <input type="text" id="spamSearch" placeholder="Search..." title="Spam all post" class="search-query form-control oe_search_box mb-2"/>
                                <div class="post_spam"/>
                            </div>
                        </div>
                    </div>
                    <div class="modal-footer justify-content-start">
                        <button type="button" class="btn btn-primary o_wforum_mark_spam">Mark as spam</button>
                        <a class="btn btn-sm btn-default o_wforum_select_all_spam" href="#" type="button">Select All</a>
                    </div>
                </div>
            </div>
        </div>
        <div t-foreach="posts_ids" t-as="question" class="post_to_validate card mb-3">
            <div class="card-body">
                <div class="row">
                    <div class="col-md-2 col-sm-3 o_js_validation_queue border-right">
                        <div class="text-center d-flex align-items-end flex-sm-column justify-content-end align-items-sm-stretch">
                            <a t-attf-href="/forum/#{slug(forum)}/post/#{slug(question)}/validate" title="Validate" aria-label="Validate" data-toggle="tooltip" class="btn border-success m-1 bg-white"><i class="fa fa-check text-success"/></a>
                            <a t-if="queue_type == 'validation'" t-attf-href="/forum/#{slug(forum)}/post/#{slug(question)}/refuse" data-toggle="tooltip" title="Refuse" aria-label="Refuse" class="btn border-danger m-1 bg-white"><i class="fa fa-times text-danger"/></a>
                            <a t-if="queue_type == 'flagged'" t-attf-href="/forum/#{slug(forum)}/post/#{slug(question)}/ask_for_mark_as_offensive" data-toggle="tooltip" aria-label="Mark as offensive" title="Mark as offensive" class="btn border-danger m-1 bg-white"><i class="fa fa-times text-danger"/></a>
                            <a href="#" t-if="queue_type == 'offensive'" disabled="True" aria-label="Offensive" title="Offensive" data-toggle="tooltip" class="btn border-danger bg-white"><i class="fa fa-times m-1 text-danger"/></a>
                        </div>
                    </div>
                    <t t-if="question.parent_id">
                        <div t-foreach="question" t-as="answer" class="col-md-10 col-sm-8">
                            <t t-call="website_forum.moderation_display_post_answer"/>
                            <small class="text-muted">
                                 <i t-attf-class="fa fa-user" role="img" aria-label="Question" />
                                <span>By </span><span t-field="question.create_uid" t-options='{"widget": "contact", "country_image": True, "fields": ["name", "country_id"]}' style="display: inline-block;"/>
                                <i class="ml-4 mr4 fa fa-calendar"/><span t-field="question.write_date" t-options='{"format":"short"}'/>
                                <span t-if="question.state == 'flagged'" class="text-black">
                                    <i class="fa fa-flag ml-4 mr4"/>
                                    Flagged
                                </span>
                            </small>
                        </div>
                    </t>
                    <div t-if="not question.parent_id" class="col">
                        <t t-call="website_forum.moderation_display_post_question_block"/>
                    </div>
                </div>
            </div>
        </div>
    </t>
</template>

<!-- User Navbar -->
<template id="user_navbar_inherit_website_forum" inherit_id="website.user_navbar">
    <xpath expr="//div[@id='o_new_content_menu_choices']//div[@name='module_website_forum']" position="attributes">
        <attribute name="name"/>
        <attribute name="t-att-data-module-id"/>
        <attribute name="t-att-data-module-shortdesc"/>
        <attribute name="groups">website.group_website_designer</attribute>
    </xpath>
</template>

<!-- Chatter templates -->
<template id="forum_post_template_new_answer">
    <p>A new answer on <t t-esc="object.name"/> has been posted. Click here to access the post :</p>
    <p style="margin-left: 30px; margin-top: 10 px; margin-bottom: 10px;">
        <a t-attf-href="/forum/#{slug(object.forum_id)}/#{slug(object)}"
            style="padding: 5px 10px; font-size: 12px; line-height: 18px; color: #FFFFFF; border-color:#875A7B; text-decoration: none; display: inline-block; margin-bottom: 0px; font-weight: 400; text-align: center; vertical-align: middle; cursor: pointer; background-color: #875A7B; border: 1px solid #875A7B; border-radius:3px">
            See post
        </a>
    </p>
</template>

<template id="forum_post_template_new_question">
    <p>A new question <b t-esc="object.name" /> on <t t-esc="object.forum_id.name"/> has been posted. Click here to access the question :</p>
    <p style="margin-left: 30px; margin-top: 10 px; margin-bottom: 10px;">
        <a t-attf-href="/forum/#{slug(object.forum_id)}/#{slug(object)}"
            style="padding: 5px 10px; font-size: 12px; line-height: 18px; color: #FFFFFF; border-color:#875A7B; text-decoration: none; display: inline-block; margin-bottom: 0px; font-weight: 400; text-align: center; vertical-align: middle; cursor: pointer;background-color: #875A7B; border: 1px solid #875A7B; border-radius:3px">
            See question
        </a>
    </p>
</template>

<template id="forum_post_template_validation">
    <p>A new question <b t-esc="object.name" /> on <t t-esc="object.forum_id.name"/> has been posted and require your validation. Click here to access the question :</p>
    <p style="margin-left: 30px; margin-top: 10 px; margin-bottom: 10px;">
        <a t-attf-href="/forum/#{slug(object.forum_id)}/#{slug(object)}"
            style="padding: 5px 10px; font-size: 12px; line-height: 18px; color: #FFFFFF; border-color:#875A7B; text-decoration: none; display: inline-block; margin-bottom: 0px; font-weight: 400; text-align: center; vertical-align: middle; cursor: pointer;background-color: #875A7B; border: 1px solid #875A7B; border-radius:3px">
            Validate question
        </a>
    </p>
</template>

    </data>
</odoo>

```

## File: views\website_forum_profile.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <!--Private profile-->
    <template id="private_profile" inherit_id="website_profile.private_profile">
        <xpath expr="//div[@id='private_profile_return_link_container']" position="inside">
            <t t-if="request.params.get('forum_id')">
                <a t-attf-href="/forum/#{request.params.get('forum_id')}">Return to the forum.</a>
            </t>
        </xpath>
    </template>

    <template id="user_profile_sub_nav" inherit_id="website_profile.user_profile_sub_nav">
        <xpath expr="//nav" position="before">
            <div t-if="request.params.get('forum_origin')" class="o_wprofile_all_users_nav_btn_container col pr-0 flex-grow-0">
                <a t-att-href="request.website._get_http_domain() + '/' + request.params.get('forum_origin').lstrip('/')"
                    class="o_wprofile_all_users_nav_btn btn text-nowrap">
                    <i class="fa fa-chevron-left small"/> Back
                </a>
            </div>
        </xpath>
    </template>

    <template id="user_profile_content" inherit_id="website_profile.user_profile_content">
        <xpath expr="//table[@id='o_wprofile_sidebar_table']//tr[last()]" position="after">
            <t t-if="forum and (up_votes or down_votes)">
                <tr id="profile_abstract_info_company">
                    <th><small class="font-weight-bold">Votes</small></th>
                    <td>
                        <span>
                            <i class="fa fa-thumbs-up text-success" role="img" aria-label="Positive votes" title="Positive votes"/>
                            <span class="font-weight-bold" t-esc="up_votes"/>
                            <i class="fa fa-thumbs-down text-danger ml-3" role="img" aria-label="Negative votes" title="Negative votes"/>
                            <span class="font-weight-bold" t-esc="down_votes"/>
                        </span>
                    </td>
                </tr>
            </t>
        </xpath>
        <xpath expr="//ul[@id='profile_extra_info_tablist']" position="inside">
            <t t-if="forum">
                <li class="nav-item">
                    <a role="tab" aria-controls="questions" href="#questions" class="nav-link o_wprofile_navlink" data-toggle="tab"><t t-esc="count_questions"/> Questions</a>
                </li>
                <li class="nav-item">
                    <a role="tab" aria-controls="answers" href="#answers" class="nav-link o_wprofile_navlink" data-toggle="tab"><t t-esc="count_answers"/> Answers</a>
                </li>
                <li t-if="uid == user.id" class="nav-item">
                    <a role="tab" aria-controls="activity" href="#activity" class="nav-link o_wprofile_navlink" data-toggle="tab">Activity</a>
                </li>
                <li t-if="uid == user.id" class="nav-item">
                    <a role="tab" aria-controls="votes" href="#votes" class="nav-link o_wprofile_navlink" data-toggle="tab">Votes</a>
                </li>
            </t>
        </xpath>
        <xpath expr="//div[@id='profile_extra_info_tabcontent']" position="inside">
            <t t-if="forum">
                <div role="tabpanel" class="o_wforum_profile_tab tab-pane" id="questions">
                    <div class="mb-4">
                        <h5 class="border-bottom pb-1">
                            Questions
                            <t t-call="website_forum.forum_filter_tag"/>
                        </h5>
                        <t t-if="questions">
                            <div class="mb-1" t-foreach="questions" t-as="question">
                                <t t-call="website_forum.display_post"/>
                            </div>
                        </t>
                        <t t-else="">
                            <span class="font-weight-bold">No question posted yet.</span>
                        </t>
                    </div>
                    <t t-if="favourite">
                        <div class="mb-4">
                            <h5 class="border-bottom pb-1">Favourite Questions</h5>
                            <div class="mb-1" t-foreach="favourite" t-as="question">
                                <t t-call="website_forum.display_post">
                                    <t t-set="hide_fav_icon" t-value="True"/>
                                </t>
                            </div>
                        </div>
                    </t>
                    <t t-if="followed">
                        <div class="mb-4">
                            <h5 class="border-bottom pb-1">Followed Questions</h5>
                            <div class="mb-1" t-foreach="followed" t-as="question">
                                <t t-call="website_forum.display_post"/>
                            </div>
                        </div>
                    </t>
                </div>
                <div role="tabpanel" class="o_wforum_profile_tab tab-pane" id="answers">
                    <div class="mb-4">
                        <h5 class="border-bottom pb-1">
                            Answers
                            <t t-call="website_forum.forum_filter_tag"/>
                        </h5>

                        <t t-if="answers">
                            <div class="mb-1" t-foreach="answers" t-as="answer">
                                <t t-call="website_forum.display_post_answer"/>
                            </div>
                        </t>
                        <t t-else="">
                            <span class="font-weight-bold">No answer posted yet.</span>
                        </t>
                    </div>
                </div>
                <div role="tabpanel" class="o_wforum_profile_tab tab-pane" id="votes" t-if="uid == user.id">
                    <div class="mb-4">
                        <h5 class="border-bottom pb-1">Votes</h5>
                        <t t-call="website_forum.user_votes"/>
                    </div>
                </div>
                <div role="tabpanel" class="o_wforum_profile_tab tab-pane" id="activity" t-if="uid == user.id">
                    <div class="mb-4">
                        <h5 class="border-bottom pb-1">
                            Activities
                            <t t-call="website_forum.forum_filter_tag"/>
                        </h5>
                        <t t-call="website_forum.display_activities"/>
                    </div>
                </div>
            </t>
        </xpath>
    </template>

    <template id="forum_filter_tag" name="Filtering Forum Tag">
        <t t-if="forum_filtered">
            <span class="align-items-baseline border d-inline-flex pl-2 rounded mb-1 ml-4">
                <i class="fa fa-filter mr-2 text-muted"/>
                <t t-esc="forum_filtered"/>
                <a t-attf-href="/profile/user/#{uid}" class="btn border-0 py-1">&#215;</a>
            </span>
        </t>
    </template>

    <template id="display_activities" name="Forum Profile Activities">
        <t t-if="activities">
            <div t-foreach="activities" t-as="activity" class="card mb-2">
                <div class="card-body">
                    <span t-esc="activity.subtype_id.name" class="badge badge-info mr-2 mt-1"/>
                    <span t-field="activity.date" t-options='{"format": "short"}' class="mr-2"/>
                    <t t-set="post" t-value="posts[activity.res_id]"/>
                    <span t-if="post[1]">
                        <a t-attf-href="/forum/#{ slug(post[0].forum_id) }/#{ slug(post[0]) }#answer-#{ str(post[1].id) }">
                            <span t-esc="post[0].name"/>
                        </a>
                    </span>
                    <span t-if="not post[1]">
                        <a t-attf-href="/forum/#{ slug(post[0].forum_id) }/#{ slug(post[0]) }">
                            <span t-esc="post[0].name"/>
                        </a>
                    </span>
                </div>
            </div>
        </t>
        <t t-else="">
            <p class="text-muted">No activities yet!</p>
        </t>
    </template>

    <template id="user_votes" name="Forum User Votes">
        <div t-foreach="vote_post" t-as="vote">
            <t t-esc="vote.post_id.create_date"/>
            <span t-if="vote.vote == '1'" class="fa fa-thumbs-up text-success" style="margin-left:30px" role="img" aria-label="Positive vote" title="Positive vote"/>
            <span t-if="vote.vote == '-1'" class="fa fa-thumbs-down text-warning" style="margin-left:30px" role="img" aria-label="Negative vote" title="Negative vote"/>
            <t t-if="vote.post_id.parent_id">
                <a t-attf-href="/forum/#{ slug(vote.post_id.forum_id) }/#{ vote.post_id.parent_id.id }/#answer-#{ vote.post_id.id }" t-esc="vote.post_id.parent_id.name" style="margin-left:10px"/>
            </t>
            <t t-if="not vote.post_id.parent_id">
                <a t-attf-href="/forum/#{ slug(vote.post_id.forum_id) }/#{ vote.post_id.id }" style=" color:black;margin-left:10px" t-esc="vote.post_id.name"/>
            </t>
        </div>
        <div class="mb16" t-if="not vote_post">
            <p class="text-muted">No vote given by you yet!</p>
        </div>
    </template>
</data></odoo>

```

