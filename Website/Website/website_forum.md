# Odoo Module: website_forum

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import controllers
from . import models
from . import populate

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
    'version': '1.2',
    'description': """
Ask questions, get answers, no distractions
        """,
    'website': 'https://www.odoo.com/app/forum',
    'depends': [
        'auth_signup',
        'website_mail',
        'website_profile',
    ],
    'data': [
        'data/ir_config_parameter_data.xml',
        'data/forum_forum_template_faq.xml',
        'data/forum_forum_data.xml',
        'data/forum_post_reason_data.xml',
        'data/ir_actions_data.xml',
        'data/mail_message_subtype_data.xml',
        'data/mail_templates.xml',
        'data/website_menu_data.xml',

        'views/forum_post_views.xml',
        'views/forum_post_reason_views.xml',
        'views/forum_tag_views.xml',
        'views/forum_forum_views.xml',
        'views/res_users_views.xml',
        'views/gamification_karma_tracking_views.xml',
        'views/forum_menus.xml',

        'views/base_contact_templates.xml',
        'views/forum_forum_templates.xml',
        'views/forum_forum_templates_forum_all.xml',
        'views/forum_forum_templates_layout.xml',
        'views/forum_forum_templates_moderation.xml',
        'views/forum_forum_templates_post.xml',
        'views/forum_forum_templates_tools.xml',
        'views/forum_templates_mail.xml',
        'views/website_profile_templates.xml',
        'views/snippets/snippets.xml',

        'security/ir.model.access.csv',
        'security/ir_rule_data.xml',

        'data/gamification_badge_data_question.xml',
        'data/gamification_badge_data_answer.xml',
        'data/gamification_badge_data_participation.xml',
        'data/gamification_badge_data_moderation.xml',
    ],
    'demo': [
        'data/forum_tag_demo.xml',
        'data/forum_post_demo.xml',
    ],
    'installable': True,
    'assets': {
        'website.assets_editor': [
            'website_forum/static/src/js/systray_items/*.js',
        ],
        'web.assets_tests': [
            'website_forum/static/tests/**/*',
        ],
        'web.assets_backend': [
            'website_forum/static/src/js/tours/website_forum.js',
        ],
        'web.assets_frontend': [
            'website_forum/static/src/js/tours/website_forum.js',
            'website_forum/static/src/scss/website_forum.scss',
            'website_forum/static/src/js/website_forum.js',
            'website_forum/static/src/js/website_forum.share.js',
            'website_forum/static/src/xml/public_templates.xml',
            'website_forum/static/src/components/flag_mark_as_offensive/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\website_forum.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import json
import logging

import lxml
import requests
import werkzeug.exceptions
import werkzeug.urls
import werkzeug.wrappers

from odoo import _, http, tools
from odoo.addons.http_routing.models.ir_http import slug
from odoo.addons.website.models.ir_http import sitemap_qs2dom
from odoo.addons.website_profile.controllers.main import WebsiteProfile
from odoo.exceptions import AccessError, UserError
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
            values['forum'] = request.env['forum.forum'].browse(int(kwargs.pop('forum_id')))
        forum = values.get('forum')
        if forum and forum is not True and not request.env.user._is_public():
            def _get_my_other_forums():
                post_domain = expression.OR(
                    [[('create_uid', '=', request.uid)],
                     [('favourite_ids', '=', request.uid)]]
                )
                return request.env['forum.forum'].search(expression.AND([
                    request.website.website_domain(),
                    [('id', '!=', forum.id)],
                    [('post_ids', 'any', post_domain)]
                ]))
            values['my_other_forums'] = tools.lazy(_get_my_other_forums)
        else:
            values['my_other_forums'] = request.env['forum.forum']
        return values

    def _prepare_mark_as_offensive_values(self, post, **kwargs):
        offensive_reasons = request.env['forum.post.reason'].search([('reason_type', '=', 'offensive')])

        values = self._prepare_user_values(**kwargs)
        values.update({
            'question': post,
            'forum': post.forum_id,
            'reasons': offensive_reasons,
            'offensive': True,
        })
        return values

    # Forum
    # --------------------------------------------------

    @http.route(['/forum'], type='http', auth="public", website=True, sitemap=True)
    def forum(self, **kwargs):
        domain = request.website.website_domain()
        forums = request.env['forum.forum'].search(domain)
        if len(forums) == 1:
            return request.redirect('/forum/%s' % slug(forums[0]), code=302)

        return request.render("website_forum.forum_all", {
            'forums': forums
        })

    def sitemap_forum(env, rule, qs):
        Forum = env['forum.forum']
        dom = sitemap_qs2dom(qs, '/forum', Forum._rec_name)
        dom += env['website'].get_current_website().website_domain()
        for f in Forum.search(dom):
            loc = '/forum/%s' % slug(f)
            if not qs or qs.lower() in loc:
                yield {'loc': loc}

    def _get_forum_post_search_options(self, forum=None, tag=None, filters=None, my=None, create_uid=False, include_answers=False, **post):
        return {
            'allowFuzzy': not post.get('noFuzzy'),
            'create_uid': create_uid,
            'displayDescription': False,
            'displayDetail': False,
            'displayExtraDetail': False,
            'displayExtraLink': False,
            'displayImage': False,
            'filters': filters,
            'forum': str(forum.id) if forum else None,
            'include_answers': include_answers,
            'my': my,
            'tag': str(tag.id) if tag else None,
        }

    @http.route(['/forum/all',
                 '/forum/all/page/<int:page>',
                 '/forum/<model("forum.forum"):forum>',
                 '/forum/<model("forum.forum"):forum>/page/<int:page>',
                 '''/forum/<model("forum.forum"):forum>/tag/<model("forum.tag"):tag>/questions''',
                 '''/forum/<model("forum.forum"):forum>/tag/<model("forum.tag"):tag>/questions/page/<int:page>''',
                 ], type='http', auth="public", website=True, sitemap=sitemap_forum)
    def questions(self, forum=None, tag=None, page=1, filters='all', my=None, sorting=None, search='', create_uid=False, include_answers=False, **post):
        Post = request.env['forum.post']

        author = request.env['res.users'].browse(int(create_uid))

        if author == request.env.user:
            my = 'mine'
        if sorting:
            # check that sorting is valid
            # retro-compatibility for V8 and google links
            try:
                sorting = werkzeug.urls.url_unquote_plus(sorting)
                Post._order_to_sql(sorting, None)
            except (UserError, ValueError):
                sorting = False

        if not sorting:
            sorting = forum.default_order if forum else 'last_activity_date desc'

        options = self._get_forum_post_search_options(
            forum=forum,
            tag=tag,
            filters=filters,
            my=my,
            create_uid=author.id,
            include_answers=include_answers,
            my_profile=request.env.user == author,
            **post
        )
        question_count, details, fuzzy_search_term = request.website._search_with_fuzzy(
            "forum_posts_only", search, limit=page * self._post_per_page, order=sorting, options=options)
        question_ids = details[0].get('results', Post)
        question_ids = question_ids[(page - 1) * self._post_per_page:page * self._post_per_page]

        if not forum:
            url = '/forum/all'
        else:
            url = f"/forum/{slug(forum)}{f'/tag/{slug(tag)}/questions' if tag else ''}"

        url_args = {'sorting': sorting}

        for name, value in zip(['filters', 'search', 'my'], [filters, search, my]):
            if value:
                url_args[name] = value

        pager = tools.lazy(lambda: request.website.pager(
            url=url, total=question_count, page=page, step=self._post_per_page,
            scope=5, url_args=url_args))

        values = self._prepare_user_values(forum=forum, searches=post)
        values.update({
            'author': author,
            'edit_in_backend': True,
            'question_ids': question_ids,
            'question_count': question_count,
            'search_count': question_count,
            'pager': pager,
            'tag': tag,
            'filters': filters,
            'my': my,
            'sorting': sorting,
            'search': fuzzy_search_term or search,
            'original_search': fuzzy_search_term and search,
        })

        if forum or tag:
            values['main_object'] = tag or forum

        return request.render("website_forum.forum_index", values)

    @http.route(['''/forum/<model("forum.forum"):forum>/faq'''], type='http', auth="public", website=True, sitemap=True)
    def forum_faq(self, forum, **post):
        values = self._prepare_user_values(forum=forum, searches=dict(), header={'is_guidelines': True}, **post)
        return request.render("website_forum.faq", values)

    @http.route(['/forum/<model("forum.forum"):forum>/faq/karma'], type='http', auth="public", website=True, sitemap=False)
    def forum_faq_karma(self, forum, **post):
        values = self._prepare_user_values(forum=forum, header={'is_guidelines': True, 'is_karma': True}, **post)
        return request.render("website_forum.faq_karma", values)

    # Tags
    # --------------------------------------------------

    @http.route('/forum/get_tags', type='http', auth="public", methods=['GET'], website=True, sitemap=False)
    def tag_read(self, forum_id, query='', limit=25, **post):
        data = request.env['forum.tag'].search_read(
            domain=[('forum_id', '=', int(forum_id)), ('name', '=ilike', (query or '') + "%")],
            fields=['id', 'name'],
            limit=int(limit),
        )
        return request.make_response(
            json.dumps(data),
            headers=[("Content-Type", "application/json")]
        )

    @http.route(['/forum/<model("forum.forum"):forum>/tag',
                 '/forum/<model("forum.forum"):forum>/tag/<string:tag_char>',
                 ], type='http', auth="public", website=True, sitemap=False)
    def tags(self, forum, tag_char='', filters='all', search='', **post):
        """Render a list of tags matching filters and search parameters.

        :param forum: Forum
        :param string tag_char: Only tags starting with a single character `tag_char`
        :param filters: One of 'all'|'followed'|'most_used'|'unused'.
          Can be combined with `search` and `tag_char`.
        :param string search: Search query using "forum_tags_only" `search_type`
        :param dict post: additional options passed to `_prepare_user_values`
        """
        if not isinstance(tag_char, str) or len(tag_char) > 1 or (tag_char and not tag_char.isalpha()):
            # So that further development does not miss this. Users shouldn't see it with normal usage.
            raise werkzeug.exceptions.BadRequest(_('Bad "tag_char" value "%(tag_char)s"', tag_char=tag_char))

        domain = [('forum_id', '=', forum.id), ('posts_count', '=' if filters == "unused" else '>', 0)]
        if filters == 'followed' and not request.env.user._is_public():
            domain = expression.AND([domain, [('message_is_follower', '=', True)]])

        # Build tags result without using tag_char to build pager, then return tags matching it
        values = self._prepare_user_values(forum=forum, searches={'tags': True}, **post)
        tags = request.env["forum.tag"]

        order = 'posts_count DESC' if tag_char else 'name'

        if search:
            values.update(search=search)
            search_domain = domain if filters in ('all', 'followed') else None
            __, details, __ = request.website._search_with_fuzzy(
                'forum_tags_only', search, limit=None, order=order, options={'forum': forum, 'domain': search_domain},
            )
            tags = details[0].get('results', tags)

        if filters in ('unused', 'most_used'):
            filter_tags = forum.tag_most_used_ids if filters == 'most_used' else forum.tag_unused_ids
            tags = tags & filter_tags if tags else filter_tags
        elif filters in ('all', 'followed'):
            if not search:
                tags = request.env['forum.tag'].search(domain, limit=None, order=order)
        else:
            raise werkzeug.exceptions.BadRequest(_('Bad "filters" value "%(filters)s".', filters=filters))

        first_char_tag = forum._get_tags_first_char(tags=tags)
        first_char_list = [(t, t.lower()) for t in first_char_tag if t.isalnum()]
        first_char_list.insert(0, (_('All'), ''))
        if tag_char:
            tags = tags.filtered(lambda t: t.name.startswith((tag_char.lower(), tag_char.upper())))

        values.update({
            'active_char_tag': tag_char.lower(),
            'pager_tag_chars': first_char_list,
            'search_count': len(tags) if search else None,
            'tags': tags,
        })
        return request.render("website_forum.forum_index_tags", values)

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
        return request.redirect("/forum/%s/%s" % (slug(forum), slug(question)), code=301)

    def sitemap_forum_post(env, rule, qs):
        ForumPost = env['forum.post']
        dom = expression.AND([
            env['website'].get_current_website().website_domain(),
            [('parent_id', '=', False), ('can_view', '=', True)],
        ])
        for forum_post in ForumPost.search(dom):
            loc = '/forum/%s/%s' % (slug(forum_post.forum_id), slug(forum_post))
            if not qs or qs.lower() in loc:
                yield {'loc': loc, 'lastmod': forum_post.write_date.date()}

    @http.route(['''/forum/<model("forum.forum"):forum>/<model("forum.post"):question>'''],
                type='http', auth="public", website=True, sitemap=sitemap_forum_post)
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
            return request.redirect(redirect_url, 301)
        filters = 'question'
        values = self._prepare_user_values(forum=forum, searches=post)
        values.update({
            'main_object': question,
            'edit_in_backend': True,
            'question': question,
            'header': {'question_data': True},
            'filters': filters,
            'reversed': reversed,
        })
        if (request.httprequest.referrer or "").startswith(request.httprequest.url_root):
            values['has_back_button_url'] = True

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
        else:
            raise werkzeug.exceptions.NotFound()
        return request.redirect(f'/forum/{slug(forum)}/post/{slug(answer)}/edit')

    @http.route('/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/close', type='http', auth="user", methods=['POST'], website=True)
    def question_close(self, forum, question, **post):
        question.close(reason_id=int(post.get('reason_id', False)))
        return request.redirect("/forum/%s/%s" % (slug(forum), slug(question)))

    @http.route('/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/reopen', type='http', auth="user", methods=['POST'], website=True)
    def question_reopen(self, forum, question, **kwarg):
        question.reopen()
        return request.redirect("/forum/%s/%s" % (slug(forum), slug(question)))

    @http.route('/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/delete', type='http', auth="user", methods=['POST'], website=True)
    def question_delete(self, forum, question, **kwarg):
        question.active = False
        return request.redirect("/forum/%s" % slug(forum))

    @http.route('/forum/<model("forum.forum"):forum>/question/<model("forum.post"):question>/undelete', type='http', auth="user", methods=['POST'], website=True)
    def question_undelete(self, forum, question, **kwarg):
        question.active = True
        return request.redirect("/forum/%s/%s" % (slug(forum), slug(question)))

    # Post
    # --------------------------------------------------
    @http.route(['/forum/<model("forum.forum"):forum>/ask'], type='http', auth="user", website=True)
    def forum_post(self, forum, **post):
        user = request.env.user
        if not user.email or not tools.single_email_re.match(user.email):
            return request.redirect("/forum/%s/user/%s/edit?email_required=1" % (slug(forum), request.session.uid))
        values = self._prepare_user_values(forum=forum, searches={}, new_question=True)
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
        if forum.has_pending_post:
            return request.redirect("/forum/%s/ask" % slug(forum))

        new_question = request.env['forum.post'].create({
            'forum_id': forum.id,
            'name': post.get('post_name') or (post_parent and 'Re: %s' % (post_parent.name or '')) or '',
            'content': post.get('content', False),
            'parent_id': post_parent and post_parent.id or False,
            'tag_ids': post_tag_ids
        })
        if post_parent:
            post_parent._update_last_activity()
        return request.redirect(f'/forum/{slug(forum)}/{slug(post_parent) if post_parent else new_question.id}')

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/comment', type='http', auth="user", methods=['POST'], website=True)
    def post_comment(self, forum, post, **kwargs):
        question = post.parent_id or post
        if kwargs.get('comment') and post.forum_id.id == forum.id:
            # TDE FIXME: check that post_id is the question or one of its answers
            body = tools.mail.plaintext2html(kwargs['comment'])
            post.with_context(mail_create_nosubscribe=True).message_post(
                body=body,
                message_type='comment',
                subtype_xmlid='mail.mt_comment')
            question._update_last_activity()
        return request.redirect(f'/forum/{slug(forum)}/{slug(question)}')

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/toggle_correct', type='json', auth="public", website=True)
    def post_toggle_correct(self, forum, post, **kwargs):
        if post.parent_id is False:
            return request.redirect('/')
        if request.uid == post.create_uid.id:
            return {'error': 'own_post'}
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
            request.redirect("/forum/%s/%s" % (slug(forum), slug(question)))
        return request.redirect("/forum/%s" % slug(forum))

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
        return request.redirect("/forum/%s/%s" % (slug(forum), slug(question)))

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

    @http.route('/forum/<model("forum.forum"):forum>/closed_posts', type='http', auth="user", website=True)
    def closed_posts(self, forum, **kwargs):
        if request.env.user.karma < forum.karma_moderate:
            raise werkzeug.exceptions.NotFound()

        closed_posts_ids = request.env['forum.post'].search(
            [('forum_id', '=', forum.id), ('state', '=', 'close')],
            order='write_date DESC, id DESC',
        )
        values = self._prepare_user_values(forum=forum)
        values.update({
            'posts_ids': closed_posts_ids,
            'queue_type': 'close',
        })

        return request.render("website_forum.moderation_queue", values)

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/validate', type='http', auth="user", website=True)
    def post_accept(self, forum, post, **kwargs):
        if post.state == 'flagged':
            url = f'/forum/{slug(forum)}/flagged_queue'
        elif post.state == 'offensive':
            url = f'/forum/{slug(forum)}/offensive_posts'
        elif post.state == 'close':
            url = f'/forum/{slug(forum)}/closed_posts'
        else:
            url = f'/forum/{slug(forum)}/validation_queue'
        post.validate()
        return request.redirect(url)

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/refuse', type='http', auth="user", website=True)
    def post_refuse(self, forum, post, **kwargs):
        post.refuse()
        return self.question_ask_for_close(forum, post)

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/flag', type='json', auth="public", website=True)
    def post_flag(self, forum, post, **kwargs):
        if not request.session.uid:
            return {'error': 'anonymous_user'}
        return post.flag()[0]

    @http.route('/forum/<model("forum.post"):post>/ask_for_mark_as_offensive', type='json', auth="user", website=True)
    def post_json_ask_for_mark_as_offensive(self, post, **kwargs):
        if not post.can_moderate:
            raise AccessError(_('%d karma required to mark a post as offensive.', post.forum_id.karma_moderate))
        values = self._prepare_mark_as_offensive_values(post, **kwargs)
        return request.env['ir.ui.view']._render_template('website_forum.mark_as_offensive', values)

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/ask_for_mark_as_offensive', type='http', auth="user", methods=['GET'], website=True)
    def post_http_ask_for_mark_as_offensive(self, forum, post, **kwargs):
        if not post.can_moderate:
            raise AccessError(_('%d karma required to mark a post as offensive.', forum.karma_moderate))
        values = self._prepare_mark_as_offensive_values(post, **kwargs)
        return request.render("website_forum.close_post", values)

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/mark_as_offensive', type='http', auth="user", methods=["POST"], website=True)
    def post_mark_as_offensive(self, forum, post, **kwargs):
        post.mark_as_offensive(reason_id=int(kwargs.get('reason_id', False)))
        if post.parent_id:
            url = f'/forum/{slug(forum)}/{post.parent_id.id}/#answer-{post.id}'
        else:
            url = f'/forum/{slug(forum)}/{slug(post)}'
        return request.redirect(url)

    # User
    # --------------------------------------------------
    @http.route(['/forum/<model("forum.forum"):forum>/partner/<int:partner_id>'], type='http', auth="public", website=True)
    def open_partner(self, forum, partner_id=0, **post):
        if partner_id:
            partner = request.env['res.partner'].sudo().search([('id', '=', partner_id)])
            if partner and partner.user_ids:
                return request.redirect(f'/forum/{slug(forum)}/user/{partner.user_ids[0].id}')
        return request.redirect('/forum/' + slug(forum))

    # Profile
    # -----------------------------------

    @http.route(['/forum/user/<int:user_id>'], type='http', auth="public", website=True)
    def view_user_forum_profile(self, user_id, forum_id='', forum_origin='/forum', **post):
        forum_origin_query = f'?forum_origin={forum_origin}&forum_id={forum_id}' if forum_id else ''
        return request.redirect(f'/profile/user/{user_id}{forum_origin_query}')

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
        data = Vote._read_group(
            [('forum_id', 'in', forums.ids), ('recipient_id', '=', user.id)], ['vote'], aggregates=['__count']
        )
        up_votes, down_votes = 0, 0
        for vote, count in data:
            if vote == '1':
                up_votes = count
            elif vote == '-1':
                down_votes = count

        # Votes which given by users on others questions and answers.
        vote_ids = Vote.search([('user_id', '=', user.id), ('forum_id', 'in', forums.ids)])

        # activity by user.
        comment = Data._xmlid_lookup('mail.mt_comment')[1]
        activities = Activity.search(
            [('res_id', 'in', (user_question_ids + user_answer_ids).ids), ('model', '=', 'forum.post'),
             ('subtype_id', '!=', comment)],
            order='date DESC', limit=100)

        posts = {}
        for act in activities:
            posts[act.res_id] = True
        posts_ids = Post.search([('id', 'in', list(posts))])
        posts = {x.id: (x.parent_id or x, x.parent_id and x or False) for x in posts_ids}

        if user != request.env.user:
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
            return request.redirect("/forum/%s" % slug(forum))
        question = post.parent_id if post.parent_id else post
        return request.redirect("/forum/%s/%s" % (slug(forum), slug(question)))

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/convert_to_comment', type='http', auth="user", methods=['POST'], website=True)
    def convert_answer_to_comment(self, forum, post, **kwarg):
        question = post.parent_id
        new_msg = post.convert_answer_to_comment()
        if not new_msg:
            return request.redirect("/forum/%s" % slug(forum))
        return request.redirect("/forum/%s/%s" % (slug(forum), slug(question)))

    @http.route('/forum/<model("forum.forum"):forum>/post/<model("forum.post"):post>/comment/<model("mail.message"):comment>/delete', type='json', auth="user", website=True)
    def delete_comment(self, forum, post, comment, **kwarg):
        if not request.session.uid:
            return {'error': 'anonymous_user'}
        return post.unlink_comment(comment.id)[0]

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import website_forum

```

## File: data\forum_forum_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <record id="forum_help" model="forum.forum">
        <field name="name">Help</field>
        <field name="image_1920" type="base64" file="website_forum/static/src/img/help.jpg"/>
        <field name="description">This community is for professionals and enthusiasts of our products and services. Share and discuss the best content and new marketing ideas, build your professional profile and become a better marketer together.</field>
    </record>

</data></odoo>

```

## File: data\forum_forum_template_faq.xml

```xml
<odoo>
    <data>
        <record id="default_faq" model="ir.ui.view">
            <field name="name">Faq Accordion</field>
            <field name="type">qweb</field>
            <field name="key">website_forum.faq_accordion</field>
            <field name="arch" type="xml">
                <section class="s_faq_collapse mb-5">
                    <div class="container">
                        <div id="myCollapse" class="accordion rounded border" role="tablist">
                            <div class="accordion-item" data-name="Item">
                                <h2 class="accordion-header" id="headingOne">
                                    <button class="accordion-button" type="button" data-bs-toggle="collapse" data-bs-target="#collapse1">
                                        What kind of questions can I ask here?
                                    </button>
                                </h2>
                                <div id="collapse1" class="accordion-collapse collapse show" data-bs-parent="#myCollapse" aria-labelledby="headingOne">
                                    <div class="accordion-body bg-light">
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
                            <div class="accordion-item" data-name="Item">
                                <h2 class="accordion-header" id="headingTwo">
                                    <button class="collapsed accordion-button" type="button" data-bs-toggle="collapse"  data-bs-target="#collapse2">
                                        What should I avoid in my questions?
                                    </button>
                                </h2>
                                <div id="collapse2" class="collapse" data-bs-parent="#myCollapse" aria-labelledby="headingTwo">
                                    <div class="accordion-body bg-light">
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
                            <div class="accordion-item" data-name="Item">
                                <h2 class="accordion-header" id="headingThree">
                                    <button class="collapsed accordion-button" type="button" data-bs-toggle="collapse" data-bs-target="#collapse3">
                                        What should I avoid in my answers?
                                    </button>
                                </h2>
                                <div id="collapse3" class="collapse"  data-bs-parent="#myCollapse" aria-labelledby="headingThree">
                                    <div class="accordion-body bg-light">
                                        <p><b>Answers should not add or expand questions</b>. Instead, either edit the question or add a comment.</p>
                                        <p><b>Answers should not comment other answers</b>. Instead add a comment on the other answers.</p>
                                        <p><b>Answers shouldn't just point to other questions</b>.Instead add a comment indicating <i>"Possible duplicate of..."</i>. However, it's fine to include links to other questions or answers providing relevant additional information.</p>
                                        <p><b>Answers shouldn't just provide a link a solution</b>. Instead provide the solution description text in your answer, even if it's just a copy/paste. Links are welcome, but should be complementary to answer, referring sources or additional reading.</p>
                                        <p><b>Answers should not start debates</b> This community Q&amp;A is not a discussion group. Please avoid holding debates in your answers as they tend to dilute the essence of questions and answers. For brief discussions please use commenting facility.</p>
                                        <p>When a question or answer is upvoted, the user who posted them will gain some points, which are called "karma points". These points serve as a rough measure of the community trust to him/her. Various moderation tasks are gradually assigned to the users based on those points.</p>
                                        <p>For example, if you ask an interesting question or give a helpful answer, your input will be upvoted. On the other hand if the answer is misleading - it will be downvoted. Each vote in favor will generate 10 points, each vote against will subtract 2 points. There is a limit of 200 points that can be accumulated for a question or answer per day. The table given at the end explains reputation point requirements for each type of moderation task.</p>
                                    </div>
                                </div>
                            </div>
                            <div class="accordion-item" data-name="Item">
                                <h2 class="accordion-header" id="headingFour">
                                    <button class="collapsed accordion-button" type="button" data-bs-toggle="collapse" data-bs-target="#collapse4">
                                        Why can other people edit my questions/answers?
                                    </button>
                                </h2>
                                <div id="collapse4" class="collapse" data-bs-parent="#myCollapse" aria-labelledby="headingFour">
                                    <div class="accordion-body bg-light">
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

## File: data\forum_post_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
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
            <field name="create_uid" ref="base.user_admin"/>
            <field name="forum_id" ref="website_forum.forum_help"/>
            <field name="name">Re: How to configure alerts for employee contract expiration</field>
            <field name="content"><![CDATA[<p>Just for posterity so other can see. Here are the steps to set automatic alerts on any contract.. i.e. HR Employee, or Fleet for example. I will use fleet as an example.</p>
<ul>
    <li>Step 1. As a user who has access rights to Technical Features, go to Settings --> Automation Rules. Create A new Automation Rule. For the Related Document Model choose.. Contract information on a vehicle (you can also type in the actual model name.. fleet.vehicle.log.contract ) . Set the trigger date to ... Contract Expiration Date. The Next Field (Delay After Trigger Date) is a bit ridiculous. Who wants to be reminded of a contract expiration AFTER the fact? The field should say Days Before Date to Fire Rule and the number should be converted to a negative. IMHO. Any way... to get a workable solution you must enter in the number in the negative. So for instance like me if you want to be warned 35 days BEFORE the expiration... put in Delay After Trigger Date.. the number -35 But the sake of testing, right now just put in -1 for 1 day before. Save the Rule.
    <li>Step 2. Go to Server Actions and create new Action. Call it Fleet Contract Expiration Warning. The Object will be the same as above .. Contract information on a vehicle. The Action Type is Email. For email address I just put my email. Under subject put in... [[object.name]]. This will tell you the name of the car. Message you can put any text message you like. Now save the Server Action.</li>
    <li>Step 3. Now go back to the Automation Rule you created and go to the Rule tab next to the conditions tab. Click Add and add the server action you created . In this case Fleet Contract Expiration Warning. Then Save.</li>
    <li>Step 4. To test, set a contract to expire tomorrow under one of your fleets vehicles. Then Save it.</li>
    <li>Step 5. Go to Scheduled Actions.. Set interval number to 1. Interval Unit to Minutes. Then Set the Next Execution date to 2 minutes from now. If your SMTP is configured correctly you will start to get a mail every minute with the reminder.</li></ul>]]></field>
            <field name="parent_id" ref="question_0" />
        </record>
        <record id="answer_1" model="forum.post">
            <field name="create_uid" ref="base.user_admin"/>
            <field name="forum_id" ref="website_forum.forum_help"/>
            <field name="name">Re: CMS replacement for ERP and eCommerce</field>
            <field name="content"><![CDATA[
<p>Odoo provides a web module and an e-commerce module: www.odoo.com/app/website
The CMS editor in Odoo web is nice but I prefer Drupal for customization and there is a Drupal module for Odoo. I think WP is better than Odoo web too.
</p>]]></field>
            <field name="parent_id" ref="question_1"/>
        </record>

        <!-- Post Vote  -->
        <record id="post_vote_0" model="forum.post.vote">
            <field name="post_id" ref="question_0"/>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="vote">1</field>
        </record>
        <record id="post_vote_1" model="forum.post.vote">
            <field name="post_id" ref="answer_0"/>
            <field name="create_uid" ref="base.user_admin"/>
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

## File: data\forum_post_reason_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
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

## File: data\forum_tag_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
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
</data></odoo>

```

## File: data\gamification_badge_data_answer.xml

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

## File: data\gamification_badge_data_moderation.xml

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

## File: data\gamification_badge_data_participation.xml

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

## File: data\gamification_badge_data_question.xml

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

## File: data\ir_actions_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!-- JUMP TO FORUM AT INSTALL -->
    <record id="action_open_forum" model="ir.actions.act_url">
        <field name="name">Forum</field>
        <field name="target">self</field>
        <field name="url" eval="'/forum/'+str(ref('website_forum.forum_help'))"/>
    </record>

    <record id="base.open_menu" model="ir.actions.todo">
        <field name="action_id" ref="action_open_forum"/>
        <field name="state">open</field>
    </record>
</data></odoo>

```

## File: data\ir_config_parameter_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <function model="ir.config_parameter" name="set_param" eval="('auth_signup.invitation_scope', 'b2c')"/>
</data></odoo>

```

## File: data\mail_message_subtype_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
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
    </data>
</odoo>

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
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
</data></odoo>

```

## File: data\website_menu_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <record id="menu_website_forums" model="website.menu">
        <field name="name">Forum</field>
        <field name="url">/forum</field>
        <field name="parent_id" ref="website.main_menu"/>
        <field name="sequence" type="int">35</field>
    </record>
</data></odoo>

```

## File: models\forum_forum.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import textwrap
from collections import defaultdict
from operator import itemgetter

from markupsafe import Markup

from odoo import _, api, fields, models
from odoo.addons.http_routing.models.ir_http import slug
from odoo.tools.translate import html_translate

MOST_USED_TAGS_COUNT = 5  # Number of tags to track as "most used" to display on frontend


class Forum(models.Model):
    _name = 'forum.forum'
    _description = 'Forum'
    _inherit = [
        'mail.thread',
        'image.mixin',
        'website.seo.metadata',
        'website.multi.mixin',
        'website.searchable.mixin',
    ]
    _order = "sequence, id"

    @api.model
    def _get_default_welcome_message(self):
        return Markup("""
                <h2 class="display-3-fs" style="text-align: center;clear-both;font-weight: bold;">%(message_intro)s</h2>
                <div class="text-white">
                    <p class="lead o_default_snippet_text" style="text-align: center;">%(message_post)s</p>
                    <p style="text-align: center;">
                        <a class="btn btn-primary forum_register_url" href="/web/login">%(register_text)s</a>
                        <button type="button" class="btn btn-light js_close_intro" aria-label="Dismiss message">
                            %(hide_text)s
                        </button>
                    </p>
                </div>
            """) % {
            'message_intro': _("Welcome!"),
            'message_post': _(
                "Share and discuss the best content and new marketing ideas, build your professional profile and become"
                " a better marketer together."
            ),
            'hide_text': _('Dismiss'),
            'register_text': _('Sign up'),
        }

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
    faq = fields.Html(
        'Guidelines', translate=html_translate,
        sanitize=True, sanitize_overridable=True)
    description = fields.Text('Description', translate=True)
    teaser = fields.Text('Teaser', compute='_compute_teaser', store=True)
    welcome_message = fields.Html(
        'Welcome Message', translate=html_translate,
        default=_get_default_welcome_message,
        sanitize_attributes=False, sanitize_form=False)
    default_order = fields.Selection([
        ('create_date desc', 'Newest'),
        ('last_activity_date desc', 'Last Updated'),
        ('vote_count desc', 'Most Voted'),
        ('relevancy desc', 'Relevance'),
        ('child_count desc', 'Answered')],
        string='Default', required=True, default='last_activity_date desc')
    relevancy_post_vote = fields.Float('First Relevance Parameter', default=0.8, help="This formula is used in order to sort by relevance. The variable 'votes' represents number of votes for a post, and 'days' is number of days since the post creation")
    relevancy_time_decay = fields.Float('Second Relevance Parameter', default=1.8)
    allow_share = fields.Boolean('Sharing Options', default=True,
                                 help='After posting the user will be proposed to share its question '
                                      'or answer on social networks, enabling social network propagation '
                                      'of the forum content.')
    # posts statistics
    post_ids = fields.One2many('forum.post', 'forum_id', string='Posts')
    last_post_id = fields.Many2one('forum.post', compute='_compute_last_post_id')
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
    has_pending_post = fields.Boolean(string='Has pending post', compute='_compute_has_pending_post')
    can_moderate = fields.Boolean(string="Is a moderator", compute="_compute_can_moderate")

    # tags
    tag_ids = fields.One2many('forum.tag', 'forum_id', string='Tags')
    tag_most_used_ids = fields.One2many('forum.tag', string="Most used tags", compute='_compute_tag_ids_usage')
    tag_unused_ids = fields.One2many('forum.tag', string="Unused tags", compute='_compute_tag_ids_usage')

    @api.depends_context('uid')
    def _compute_has_pending_post(self):
        domain = [
            ('create_uid', '=', self.env.user.id),
            ('state', '=', 'pending'),
            ('parent_id', '=', False),
        ]
        pending_forums = self.env['forum.forum'].search([
            ('id', 'in', self.ids),
            ('post_ids', 'any', domain),
        ])
        pending_forums.has_pending_post = True
        (self - pending_forums).has_pending_post = False

    @api.depends_context('uid')
    @api.depends('karma_moderate')
    def _compute_can_moderate(self):
        for forum in self:
            forum.can_moderate = self.env.user.karma >= forum.karma_moderate

    @api.depends('post_ids', 'post_ids.tag_ids', 'post_ids.tag_ids.posts_count', 'tag_ids')
    def _compute_tag_ids_usage(self):
        forums_without_tags = self.filtered(lambda f: not f.tag_ids)
        forums_without_tags.tag_most_used_ids = forums_without_tags.tag_unused_ids = False
        forums_with_tags = self - forums_without_tags
        if not forums_with_tags:
            return

        tags_data = self.env['forum.tag'].search_read(
            [('forum_id', 'in', forums_with_tags.ids)],
            fields=['id', 'forum_id', 'posts_count'],
            order='forum_id, posts_count DESC, name, id',
        )
        current_forum_id = tags_data[0]['forum_id'][0]
        forum_tags = defaultdict(lambda: {'most_used_ids': [], 'unused_ids': []})

        for tag_data in tags_data:
            tag_id, tag_forum_id, posts_count = itemgetter('id', 'forum_id', 'posts_count')(tag_data)
            if tag_forum_id[0] != current_forum_id:
                current_forum_id = tag_forum_id[0]
            if not posts_count:  # Could be 0 or None
                forum_tags[current_forum_id]['unused_ids'].append(tag_id)
            elif len(forum_tags[current_forum_id]['most_used_ids']) < MOST_USED_TAGS_COUNT:
                forum_tags[current_forum_id]['most_used_ids'].append(tag_id)

        for forum in forums_with_tags:
            forum.tag_most_used_ids = self.env['forum.tag'].browse(forum_tags[forum.id]['most_used_ids'])
            forum.tag_unused_ids = self.env['forum.tag'].browse(forum_tags[forum.id]['unused_ids'])

    @api.depends('description')
    def _compute_teaser(self):
        for forum in self:
            forum.teaser = textwrap.shorten(forum.description, width=180, placeholder='...') if forum.description else ""

    @api.depends('post_ids')
    def _compute_last_post_id(self):
        last_forums_posts = self.env['forum.post']._read_group(
            [('forum_id', 'in', self.ids), ('parent_id', '=', False), ('state', '=', 'active')],
            groupby=['forum_id'], aggregates=['id:max'],
        )
        forum_to_last_post_id = {forum.id: last_post_id for forum, last_post_id in last_forums_posts}
        for forum in self:
            forum.last_post_id = forum_to_last_post_id.get(forum.id, False)

    @api.depends('post_ids.state', 'post_ids.views', 'post_ids.child_count', 'post_ids.favourite_count')
    def _compute_forum_statistics(self):
        default_stats = {'total_posts': 0, 'total_views': 0, 'total_answers': 0, 'total_favorites': 0}

        if not self.ids:
            self.update(default_stats)
            return

        result = {cid: dict(default_stats) for cid in self.ids}
        read_group_res = self.env['forum.post']._read_group(
            [('forum_id', 'in', self.ids), ('state', 'in', ('active', 'close')), ('parent_id', '=', False)],
            ['forum_id'],
            ['__count', 'views:sum', 'child_count:sum', 'favourite_count:sum'])
        for forum, count, views_sum, child_count_sum, favourite_count_sum in read_group_res:
            stat_forum = result[forum.id]
            stat_forum['total_posts'] += count
            stat_forum['total_views'] += views_sum
            stat_forum['total_answers'] += child_count_sum
            stat_forum['total_favorites'] += 1 if favourite_count_sum else 0

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

    # EXTENDS WEBSITE.MULTI.MIXIN

    def _compute_website_url(self):
        if not self.id:
            return False
        return f'/forum/{slug(self)}'

    # ----------------------------------------------------------------------
    # CRUD
    # ----------------------------------------------------------------------

    @api.model_create_multi
    def create(self, vals_list):
        forums = super(
            Forum,
            self.with_context(mail_create_nolog=True, mail_create_nosubscribe=True)
        ).create(vals_list)
        self.env['website'].sudo()._update_forum_count()
        forums._set_default_faq()
        return forums

    def unlink(self):
        self.env['website'].sudo()._update_forum_count()
        return super().unlink()

    def write(self, vals):
        if 'privacy' in vals:
            if not vals['privacy']:
                # The forum is neither public, neither private, remove menu to avoid conflict
                self.menu_id.unlink()
            elif vals['privacy'] == 'public':
                # The forum is public, the menu must be also public
                vals['authorized_group_id'] = False
            elif vals['privacy'] == 'connected':
                vals['authorized_group_id'] = False

        res = super().write(vals)
        if 'active' in vals:
            # archiving/unarchiving a forum does it on its posts, too
            self.env['forum.post'].with_context(active_test=False).search([('forum_id', 'in', self.ids)]).write({'active': vals['active']})

        if 'active' in vals or 'website_id' in vals:
            self.env['website'].sudo()._update_forum_count()
        return res

    def _set_default_faq(self):
        for forum in self:
            forum.faq = self.env['ir.ui.view']._render_template('website_forum.faq_accordion', {"forum": forum})

    # ----------------------------------------------------------------------
    # TOOLS
    # ----------------------------------------------------------------------

    def _tag_to_write_vals(self, tags=''):
        Tag = self.env['forum.tag']
        post_tags = []
        existing_keep = []
        user = self.env.user
        for tag_id_or_new_name in (tag.strip() for tag in tags.split(',') if tag and tag.strip()):
            if tag_id_or_new_name.startswith('_'):  # it's a new tag
                tag_name = tag_id_or_new_name[1:]
                # check that not already created meanwhile or maybe excluded by the limit on the search
                tag_ids = Tag.search([('name', '=', tag_name), ('forum_id', '=', self.id)], limit=1)
                if tag_ids:
                    existing_keep.append(tag_ids.id)
                else:
                    # check if user have Karma needed to create need tag
                    if user.exists() and user.karma >= self.karma_tag_create and tag_name:
                        post_tags.append((0, 0, {'name': tag_name, 'forum_id': self.id}))
            else:
                existing_keep.append(int(tag_id_or_new_name))
        post_tags.insert(0, [6, 0, existing_keep])
        return post_tags

    def _get_tags_first_char(self, tags=None):
        """Get set of first letter of forum tags.

        :param tags: tags recordset to further filter forum's tags that are also in these tags.
        """
        tag_ids = self.tag_ids if tags is None else (self.tag_ids & tags)
        return sorted({tag.name[0].upper() for tag in tag_ids if len(tag.name)})

    # ----------------------------------------------------------------------
    # WEBSITE
    # ----------------------------------------------------------------------

    def go_to_website(self):
        self.ensure_one()
        website_url = self._compute_website_url()
        if not website_url:
            return False
        return self.env['website'].get_client_action(self._compute_website_url())

    @api.model
    def _search_get_detail(self, website, order, options):
        with_description = options['displayDescription']
        search_fields = ['name']
        fetch_fields = ['id', 'name']
        mapping = {
            'name': {'name': 'name', 'type': 'text', 'match': True},
            'website_url': {'name': 'website_url', 'type': 'text', 'truncate': False},
        }
        if with_description:
            search_fields.append('description')
            fetch_fields.append('description')
            mapping['description'] = {'name': 'description', 'type': 'text', 'match': True}
        return {
            'model': 'forum.forum',
            'base_domain': [website.website_domain()],
            'search_fields': search_fields,
            'fetch_fields': fetch_fields,
            'mapping': mapping,
            'icon': 'fa-comments-o',
            'order': 'name desc, id desc' if 'name desc' in order else 'name asc, id desc',
        }

    def _search_render_results(self, fetch_fields, mapping, icon, limit):
        results_data = super()._search_render_results(fetch_fields, mapping, icon, limit)
        for forum, data in zip(self, results_data):
            data['website_url'] = forum._compute_website_url()
        return results_data

```

## File: models\forum_post.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import math
import re
from datetime import datetime

from odoo import api, fields, models, tools, _
from odoo.addons.http_routing.models.ir_http import slug, unslug
from odoo.exceptions import UserError, ValidationError, AccessError
from odoo.osv import expression
from odoo.tools import sql

_logger = logging.getLogger(__name__)


class Post(models.Model):
    _name = 'forum.post'
    _description = 'Forum Post'
    _inherit = [
        'mail.thread',
        'website.seo.metadata',
        'website.searchable.mixin',
    ]
    _order = "is_correct DESC, vote_count DESC, last_activity_date DESC"

    name = fields.Char('Title')
    forum_id = fields.Many2one('forum.forum', string='Forum', required=True)
    content = fields.Html('Content', strip_style=True)
    plain_content = fields.Text(
        'Plain Content',
        compute='_compute_plain_content', store=True)
    tag_ids = fields.Many2many('forum.tag', 'forum_tag_rel', 'forum_id', 'forum_tag_id', string='Tags')
    state = fields.Selection(
        [
            ('active', 'Active'), ('pending', 'Waiting Validation'),
            ('close', 'Closed'), ('offensive', 'Offensive'),
            ('flagged', 'Flagged'),
        ], string='Status', default='active')
    views = fields.Integer('Views', default=0, readonly=True, copy=False)
    active = fields.Boolean('Active', default=True)
    website_message_ids = fields.One2many(domain=lambda self: [('model', '=', self._name), ('message_type', 'in', ['email', 'comment', 'email_outgoing'])])
    website_url = fields.Char('Website URL', compute='_compute_website_url')
    website_id = fields.Many2one(related='forum_id.website_id', readonly=True)

    # history
    create_date = fields.Datetime('Asked on', index=True, readonly=True)
    create_uid = fields.Many2one('res.users', string='Created by', index=True, readonly=True)
    write_date = fields.Datetime('Updated on', index=True, readonly=True)
    last_activity_date = fields.Datetime(
        'Last activity on', readonly=True, required=True, default=fields.Datetime.now,
        help="Field to keep track of a post's last activity. Updated whenever it is replied to, "
             "or when a comment is added on the post or one of its replies."
    )
    write_uid = fields.Many2one('res.users', string='Updated by', index=True, readonly=True)
    relevancy = fields.Float('Relevance', compute="_compute_relevancy", store=True)

    # vote
    vote_ids = fields.One2many('forum.post.vote', 'post_id', string='Votes')
    user_vote = fields.Integer('My Vote', compute='_compute_user_vote')
    vote_count = fields.Integer('Total Votes', compute='_compute_vote_count', store=True)

    # favorite
    favourite_ids = fields.Many2many('res.users', string='Favourite')
    user_favourite = fields.Boolean('Is Favourite', compute='_compute_user_favourite')
    favourite_count = fields.Integer('Favorite', compute='_compute_favorite_count', store=True)

    # hierarchy
    is_correct = fields.Boolean('Correct', help='Correct answer or answer accepted')
    parent_id = fields.Many2one(
        'forum.post', string='Question',
        ondelete='cascade', readonly=True, index=True)
    self_reply = fields.Boolean('Reply to own question', compute='_compute_self_reply', store=True)
    child_ids = fields.One2many(
        'forum.post', 'parent_id', string='Post Answers',
        domain="[('forum_id', '=', forum_id)]")
    child_count = fields.Integer('Answers', compute='_compute_child_count', store=True)
    uid_has_answered = fields.Boolean('Has Answered', compute='_compute_uid_has_answered')
    has_validated_answer = fields.Boolean(
        'Is answered',
        compute='_compute_has_validated_answer', store=True)

    # offensive moderation tools
    flag_user_id = fields.Many2one('res.users', string='Flagged by')
    moderator_id = fields.Many2one('res.users', string='Reviewed by', readonly=True)

    # closing
    closed_reason_id = fields.Many2one('forum.post.reason', string='Reason', copy=False)
    closed_uid = fields.Many2one('res.users', string='Closed by', readonly=True, copy=False)
    closed_date = fields.Datetime('Closed on', readonly=True, copy=False)

    # karma calculation and access
    karma_accept = fields.Integer(
        'Convert comment to answer',
        compute='_compute_post_karma_rights', compute_sudo=False)
    karma_edit = fields.Integer(
        'Karma to edit',
        compute='_compute_post_karma_rights', compute_sudo=False)
    karma_close = fields.Integer(
        'Karma to close',
        compute='_compute_post_karma_rights', compute_sudo=False)
    karma_unlink = fields.Integer(
        'Karma to unlink',
        compute='_compute_post_karma_rights', compute_sudo=False)
    karma_comment = fields.Integer(
        'Karma to comment',
        compute='_compute_post_karma_rights', compute_sudo=False)
    karma_comment_convert = fields.Integer(
        'Karma to convert comment to answer',
        compute='_compute_post_karma_rights', compute_sudo=False)
    karma_flag = fields.Integer(
        'Flag a post as offensive',
        compute='_compute_post_karma_rights', compute_sudo=False)
    can_ask = fields.Boolean(
        'Can Ask',
        compute='_compute_post_karma_rights', compute_sudo=False)
    can_answer = fields.Boolean(
        'Can Answer',
        compute='_compute_post_karma_rights', compute_sudo=False)
    can_accept = fields.Boolean(
        'Can Accept',
        compute='_compute_post_karma_rights', compute_sudo=False)
    can_edit = fields.Boolean(
        'Can Edit',
        compute='_compute_post_karma_rights', compute_sudo=False)
    can_close = fields.Boolean(
        'Can Close',
        compute='_compute_post_karma_rights', compute_sudo=False)
    can_unlink = fields.Boolean(
        'Can Unlink',
        compute='_compute_post_karma_rights', compute_sudo=False)
    can_upvote = fields.Boolean(
        'Can Upvote',
        compute='_compute_post_karma_rights', compute_sudo=False)
    can_downvote = fields.Boolean(
        'Can Downvote',
        compute='_compute_post_karma_rights', compute_sudo=False)
    can_comment = fields.Boolean(
        'Can Comment',
        compute='_compute_post_karma_rights', compute_sudo=False)
    can_comment_convert = fields.Boolean(
        'Can Convert to Comment',
        compute='_compute_post_karma_rights', compute_sudo=False)
    can_view = fields.Boolean(
        'Can View',
        compute='_compute_post_karma_rights', compute_sudo=False, search='_search_can_view')
    can_display_biography = fields.Boolean(
        "Is the author's biography visible from his post",
        compute='_compute_post_karma_rights', compute_sudo=False)
    can_post = fields.Boolean(
        'Can Automatically be Validated',
        compute='_compute_post_karma_rights', compute_sudo=False)
    can_flag = fields.Boolean(
        'Can Flag',
        compute='_compute_post_karma_rights', compute_sudo=False)
    can_moderate = fields.Boolean(
        'Can Moderate',
        compute='_compute_post_karma_rights', compute_sudo=False)
    can_use_full_editor = fields.Boolean(  # Editor Features: image and links
        'Can Use Full Editor',
        compute='_compute_post_karma_rights', compute_sudo=False)

    @api.constrains('parent_id')
    def _check_parent_id(self):
        if not self._check_recursion():
            raise ValidationError(_('You cannot create recursive forum posts.'))

    @api.depends('content')
    def _compute_plain_content(self):
        for post in self:
            post.plain_content = tools.html2plaintext(post.content)[0:500] if post.content else False

    @api.depends('name')
    def _compute_website_url(self):
        self.website_url = False
        for post in self.filtered(lambda post: post.id):
            anchor = f'#answer_{post.id}' if post.parent_id else ''
            post.website_url = f'/forum/{slug(post.forum_id)}/{slug(post)}{anchor}'

    @api.depends('vote_count', 'forum_id.relevancy_post_vote', 'forum_id.relevancy_time_decay')
    def _compute_relevancy(self):
        for post in self:
            if post.create_date:
                days = (datetime.today() - post.create_date).days
                post.relevancy = math.copysign(1, post.vote_count) * (abs(post.vote_count - 1) ** post.forum_id.relevancy_post_vote / (days + 2) ** post.forum_id.relevancy_time_decay)
            else:
                post.relevancy = 0

    @api.depends_context('uid')
    def _compute_user_vote(self):
        votes = self.env['forum.post.vote'].search_read([('post_id', 'in', self._ids), ('user_id', '=', self._uid)], ['vote', 'post_id'])
        mapped_vote = dict([(v['post_id'][0], v['vote']) for v in votes])
        for vote in self:
            vote.user_vote = mapped_vote.get(vote.id, 0)

    @api.depends('vote_ids.vote')
    def _compute_vote_count(self):
        read_group_res = self.env['forum.post.vote']._read_group([('post_id', 'in', self._ids)], ['post_id', 'vote'], ['__count'])
        result = dict.fromkeys(self._ids, 0)
        for post, vote, count in read_group_res:
            result[post.id] += count * int(vote)
        for post in self:
            post.vote_count = result[post.id]

    @api.depends_context('uid')
    def _compute_user_favourite(self):
        for post in self:
            post.user_favourite = post._uid in post.favourite_ids.ids

    @api.depends('favourite_ids')
    def _compute_favorite_count(self):
        for post in self:
            post.favourite_count = len(post.favourite_ids)

    @api.depends('create_uid', 'parent_id')
    def _compute_self_reply(self):
        for post in self:
            post.self_reply = post.parent_id.create_uid == post.create_uid

    @api.depends('child_ids')
    def _compute_child_count(self):
        for post in self:
            post.child_count = len(post.child_ids)

    @api.depends_context('uid')
    def _compute_uid_has_answered(self):
        for post in self:
            post.uid_has_answered = post._uid in post.child_ids.create_uid.ids

    @api.depends('child_ids.is_correct')
    def _compute_has_validated_answer(self):
        for post in self:
            post.has_validated_answer = any(answer.is_correct for answer in post.child_ids)

    @api.depends_context('uid')
    def _compute_post_karma_rights(self):
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
            post.can_view = post.can_close or post_sudo.active and (post_sudo.create_uid.karma > 0 or post_sudo.create_uid == user)
            post.can_display_biography = is_admin or (post_sudo.create_uid.karma >= post.forum_id.karma_user_bio and post_sudo.create_uid.website_published)
            post.can_post = is_admin or user.karma >= post.forum_id.karma_post
            post.can_flag = is_admin or user.karma >= post.forum_id.karma_flag
            post.can_moderate = is_admin or user.karma >= post.forum_id.karma_moderate
            post.can_use_full_editor = is_admin or user.karma >= post.forum_id.karma_editor

    def _search_can_view(self, operator, value):
        if operator not in ('=', '!=', '<>'):
            raise ValueError('Invalid operator: %s' % (operator,))

        if not value:
            operator = '!=' if operator == '=' else '='

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

        op = 'inselect' if operator == '=' else "not inselect"

        # don't use param named because orm will add other param (test_active, ...)
        return [('id', op, (req, (user.id, user.karma, user.id, user.karma, user.id)))]

    # EXTENDS WEBSITE.SEO.METADATA

    def _default_website_meta(self):
        res = super(Post, self)._default_website_meta()
        res['default_opengraph']['og:title'] = res['default_twitter']['twitter:title'] = self.name
        res['default_opengraph']['og:description'] = res['default_twitter']['twitter:description'] = self.plain_content
        res['default_opengraph']['og:image'] = res['default_twitter']['twitter:image'] = self.env['website'].image_url(self.create_uid, 'image_1024')
        res['default_twitter']['twitter:card'] = 'summary'
        res['default_meta_description'] = self.plain_content
        return res

    # ----------------------------------------------------------------------
    # CRUD
    # ----------------------------------------------------------------------

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if 'content' in vals and vals.get('forum_id'):
                vals['content'] = self._update_content(vals['content'], vals['forum_id'])

        posts = super(Post, self.with_context(mail_create_nolog=True)).create(vals_list)

        for post in posts:
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
                post.create_uid.sudo()._add_karma(post.forum_id.karma_gen_question_new, post, _('Ask a new question'))
        posts.post_notification()
        return posts

    def unlink(self):
        # if unlinking an answer with accepted answer: remove provided karma
        for post in self:
            if post.is_correct:
                post.create_uid.sudo()._add_karma(post.forum_id.karma_gen_answer_accepted * -1, post, _('The accepted answer is deleted'))
                self.env.user.sudo()._add_karma(post.forum_id.karma_gen_answer_accepted * -1, post, _('Delete the accepted answer'))
        return super(Post, self).unlink()

    def write(self, vals):
        trusted_keys = ['active', 'is_correct', 'tag_ids']  # fields where security is checked manually
        if 'forum_id' in vals:
            forum = self.env['forum.forum'].browse(vals['forum_id'])
            forum.check_access_rule('write')
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
                    post.create_uid.sudo()._add_karma(post.forum_id.karma_gen_answer_accepted * mult, post,
                                                      _('User answer accepted') if mult > 0 else _('Accepted answer removed'))
                    self.env.user.sudo()._add_karma(post.forum_id.karma_gen_answer_accept * mult, post,
                                                    _('Validate an answer') if mult > 0 else _('Remove validated answer'))
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

    def _get_access_action(self, access_uid=None, force_website=False):
        """ Instead of the classic form view, redirect to the post on the website directly """
        self.ensure_one()
        if not force_website and not self.state == 'active':
            return super(Post, self)._get_access_action(access_uid=access_uid, force_website=force_website)
        return {
            'type': 'ir.actions.act_url',
            'url': '/forum/%s/%s' % (self.forum_id.id, self.id),
            'target': 'self',
            'target_type': 'public',
            'res_id': self.id,
        }

    @api.ondelete(at_uninstall=False)
    def _unlink_if_enough_karma(self):
        for post in self:
            if not post.can_unlink:
                raise AccessError(_('%d karma required to unlink a post.', post.karma_unlink))

    def _update_content(self, content, forum_id):
        forum = self.env['forum.forum'].browse(forum_id)
        if content and self.env.user.karma < forum.karma_dofollow:
            for match in re.findall(r'<a\s.*href=".*?">', content):
                escaped_match = re.escape(match)  # replace parenthesis or special char in regex
                url_match = re.match(r'^.*href="(.*)".*', match) # extracting the link allows to rebuild a clean link tag
                url = url_match.group(1)
                content = re.sub(escaped_match, f'<a rel="nofollow" href="{url}">', content)

        if self.env.user.karma < forum.karma_editor:
            filter_regexp = r'(<img.*?>)|(<a[^>]*?href[^>]*?>)|(<[a-z|A-Z]+[^>]*style\s*=\s*[\'"][^\'"]*\s*background[^:]*:[^url;]*url)'
            content_match = re.search(filter_regexp, content, re.I)
            if content_match:
                raise AccessError(_('%d karma required to post an image or link.', forum.karma_editor))
        return content

    # ----------------------------------------------------------------------
    # BUSINESS
    # ----------------------------------------------------------------------

    def post_notification(self):
        for post in self:
            tag_partners = post.tag_ids.sudo().mapped('message_partner_ids')

            if post.state == 'active' and post.parent_id:
                post.parent_id.message_post_with_source(
                    'website_forum.forum_post_template_new_answer',
                    subject=_('Re: %s', post.parent_id.name),
                    partner_ids=tag_partners.ids,
                    subtype_xmlid='website_forum.mt_answer_new',
                )
            elif post.state == 'active' and not post.parent_id:
                post.message_post_with_source(
                    'website_forum.forum_post_template_new_question',
                    subject=post.name,
                    partner_ids=tag_partners.ids,
                    subtype_xmlid='website_forum.mt_question_new',
                )
            elif post.state == 'pending' and not post.parent_id:
                # TDE FIXME: in master, you should probably use a subtype;
                # however here we remove subtype but set partner_ids
                partners = post.sudo().message_partner_ids | tag_partners
                partners = partners.filtered(lambda partner: partner.user_ids and any(user.karma >= post.forum_id.karma_moderate for user in partner.user_ids))

                post.message_post_with_source(
                    'website_forum.forum_post_template_validation',
                    subject=post.name,
                    partner_ids=partners.ids,
                    subtype_xmlid='mail.mt_note',
                )
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
                post.create_uid.sudo()._add_karma(karma * -1, post, _('Reopen a banned question'))

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
                message = (
                    _('Post is closed and marked as spam')
                    if reason_id == reason_spam else
                    _('Post is closed and marked as offensive content')
                )
                post.create_uid.sudo()._add_karma(karma, post, message)

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
                post.create_uid.sudo()._add_karma(
                    post.forum_id.karma_gen_question_new,
                    post,
                    _('Ask a question'),
                )
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
            post.create_uid.sudo()._add_karma(post.forum_id.karma_gen_answer_flagged, post, _('Downvote for posting offensive contents'))
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
    def convert_comment_to_answer(self, message_id):
        """ Tool to convert a comment (mail.message) into an answer (forum.post).
        The original comment is unlinked and a new answer from the comment's author
        is created. Nothing is done if the comment's author already answered the
        question. """
        comment_sudo = self.env['mail.message'].sudo().browse(message_id)
        post = self.browse(comment_sudo.res_id)
        if not comment_sudo.author_id or not comment_sudo.author_id.user_ids:  # only comment posted by users can be converted
            return False

        # karma-based action check: must check the message's author to know if own / all
        is_author = comment_sudo.author_id.id == self.env.user.partner_id.id
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
        post_create_uid = comment_sudo.author_id.user_ids[0]
        if any(answer.create_uid.id == post_create_uid.id for answer in question.child_ids):
            return False

        # create the new post
        post_values = {
            'forum_id': question.forum_id.id,
            'content': comment_sudo.body,
            'parent_id': question.id,
            'name': _('Re: %s', question.name or ''),
        }
        # done with the author user to have create_uid correctly set
        new_post = self.with_user(post_create_uid).sudo().create(post_values).sudo(False)

        # delete comment
        comment_sudo.unlink()

        return new_post

    def unlink_comment(self, message_id):
        comment_sudo = self.env['mail.message'].sudo().browse(message_id)
        if comment_sudo.model != 'forum.post':
            return [False] * len(self)

        user_karma = self.env.user.karma
        result = []
        for post in self:
            if comment_sudo.res_id != post.id:
                result.append(False)
                continue
            # karma-based action check: must check the message's author to know if own or all
            karma_required = (
                post.forum_id.karma_comment_unlink_own
                if comment_sudo.author_id.id == self.env.user.partner_id.id
                else post.forum_id.karma_comment_unlink_all
            )
            if user_karma < karma_required:
                raise AccessError(_('%d karma required to delete a comment.', karma_required))
            result.append(comment_sudo.unlink())
        return result

    def _set_viewed(self):
        self.ensure_one()
        return sql.increment_fields_skiplock(self, 'views')

    def _update_last_activity(self):
        self.ensure_one()
        return self.sudo().write({'last_activity_date': fields.Datetime.now()})

    # ----------------------------------------------------------------------
    # MESSAGING
    # ----------------------------------------------------------------------

    @api.model
    def _get_mail_message_access(self, res_ids, operation, model_name=None):
        # XDO FIXME: to be correctly fixed with new _get_mail_message_access and filter access rule
        if operation in ('write', 'unlink') and (not model_name or model_name == 'forum.post'):
            # Make sure only author or moderator can edit/delete messages
            for post in self.browse(res_ids):
                if not post.can_edit:
                    raise AccessError(_('%d karma required to edit a post.', post.karma_edit))
        return super(Post, self)._get_mail_message_access(res_ids, operation, model_name=model_name)

    def _notify_get_recipients_groups(self, message, model_description, msg_vals=None):
        """ Add access button to everyone if the document is active. """
        groups = super()._notify_get_recipients_groups(
            message, model_description, msg_vals=msg_vals
        )
        if not self:
            return groups

        self.ensure_one()
        if self.state == 'active':
            for _group_name, _group_method, group_data in groups:
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

    def _notify_thread_by_inbox(self, message, recipients_data, msg_vals=False, **kwargs):
        """ Override to avoid keeping all notified recipients of a comment.
        We avoid tracking needaction on post comments. Only emails should be
        sufficient. """
        if msg_vals is None:
            msg_vals = {}
        if msg_vals.get('message_type', message.message_type) == 'comment':
            return
        return super(Post, self)._notify_thread_by_inbox(message, recipients_data, msg_vals=msg_vals, **kwargs)

    # ----------------------------------------------------------------------
    # WEBSITE
    # ----------------------------------------------------------------------

    def go_to_website(self):
        self.ensure_one()
        if not self.website_url:
            return False
        return self.env['website'].get_client_action(self.website_url)

    @api.model
    def _search_get_detail(self, website, order, options):
        with_description = options['displayDescription']
        with_date = options['displayDetail']
        search_fields = ['name']
        fetch_fields = ['id', 'name', 'website_url']
        mapping = {
            'name': {'name': 'name', 'type': 'text', 'match': True},
            'website_url': {'name': 'website_url', 'type': 'text', 'truncate': False},
        }

        domain = website.website_domain()
        domain = expression.AND([domain, [('state', '=', 'active'), ('can_view', '=', True)]])
        include_answers = options.get('include_answers', False)
        if not include_answers:
            domain = expression.AND([domain, [('parent_id', '=', False)]])
        forum = options.get('forum')
        if forum:
            domain = expression.AND([domain, [('forum_id', '=', unslug(forum)[1])]])
        tags = options.get('tag')
        if tags:
            domain = expression.AND([domain, [('tag_ids', 'in', [unslug(tag)[1] for tag in tags.split(',')])]])
        filters = options.get('filters')
        if filters == 'unanswered':
            domain = expression.AND([domain, [('child_ids', '=', False)]])
        elif filters == 'solved':
            domain = expression.AND([domain, [('has_validated_answer', '=', True)]])
        elif filters == 'unsolved':
            domain = expression.AND([domain, [('has_validated_answer', '=', False)]])
        user = self.env.user
        my = options.get('my')
        create_uid = user.id if my == 'mine' else options.get('create_uid')
        if create_uid:
            domain = expression.AND([domain, [('create_uid', '=', create_uid)]])
        if my == 'followed':
            domain = expression.AND([domain, [('message_partner_ids', '=', user.partner_id.id)]])
        elif my == 'tagged':
            domain = expression.AND([domain, [('tag_ids.message_partner_ids', '=', user.partner_id.id)]])
        elif my == 'favourites':
            domain = expression.AND([domain, [('favourite_ids', '=', user.id)]])
        elif my == 'upvoted':
            domain = expression.AND([domain, [('vote_ids.user_id', '=', user.id)]])

        # 'sorting' from the form's "Order by" overrides order during auto-completion
        order = options.get('sorting', order)
        if 'is_published' in order:
            parts = [part for part in order.split(',') if 'is_published' not in part]
            order = ','.join(parts)

        if with_description:
            search_fields.append('content')
            fetch_fields.append('content')
            mapping['description'] = {'name': 'content', 'type': 'text', 'html': True, 'match': True}
        if with_date:
            fetch_fields.append('write_date')
            mapping['detail'] = {'name': 'date', 'type': 'html'}
        return {
            'model': 'forum.post',
            'base_domain': [domain],
            'search_fields': search_fields,
            'fetch_fields': fetch_fields,
            'mapping': mapping,
            'icon': 'fa-comment-o',
            'order': order,
        }

    def _search_render_results(self, fetch_fields, mapping, icon, limit):
        with_date = 'detail' in mapping
        results_data = super()._search_render_results(fetch_fields, mapping, icon, limit)
        for post, data in zip(self, results_data):
            if with_date:
                data['date'] = self.env['ir.qweb.field.date'].record_to_html(post, 'write_date', {})
        return results_data

```

## File: models\forum_post_reason.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class PostReason(models.Model):
    _name = "forum.post.reason"
    _description = "Post Closing Reason"
    _order = 'name'

    name = fields.Char(string='Closing Reason', required=True, translate=True)
    reason_type = fields.Selection([('basic', 'Basic'), ('offensive', 'Offensive')], string='Reason Type', default='basic')

```

## File: models\forum_post_vote.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError, AccessError


class Vote(models.Model):
    _name = 'forum.post.vote'
    _description = 'Post Vote'
    _order = 'create_date desc, id desc'

    post_id = fields.Many2one('forum.post', string='Post', ondelete='cascade', required=True)
    user_id = fields.Many2one('res.users', string='User', required=True, default=lambda self: self._uid, ondelete='cascade')
    vote = fields.Selection([('1', '1'), ('-1', '-1'), ('0', '0')], string='Vote', required=True, default='1')
    create_date = fields.Datetime('Create Date', index=True, readonly=True)
    forum_id = fields.Many2one('forum.forum', string='Forum', related="post_id.forum_id", store=True, readonly=False)
    recipient_id = fields.Many2one('res.users', string='To', related="post_id.create_uid", store=True, readonly=False)

    _sql_constraints = [
        ('vote_uniq', 'unique (post_id, user_id)', "Vote already exists!"),
    ]

    def _get_karma_value(self, old_vote, new_vote, up_karma, down_karma):
        """Return the karma to add / remove based on the old vote and on the new vote."""
        karma_values = {'-1': down_karma, '0': 0, '1': up_karma}
        karma = karma_values[new_vote] - karma_values[old_vote]

        if old_vote == new_vote:
            reason = _('no changes')
        elif new_vote == '1':
            reason = _('upvoted')
        elif new_vote == '-1':
            reason = _('downvoted')
        elif old_vote == '1':
            reason = _('no more upvoted')
        else:
            reason = _('no more downvoted')

        return karma, reason

    @api.model_create_multi
    def create(self, vals_list):
        # can't modify owner of a vote
        if not self.env.is_admin():
            for vals in vals_list:
                vals.pop('user_id', None)

        votes = super(Vote, self).create(vals_list)

        for vote in votes:
            vote._check_general_rights()
            vote._check_karma_rights(vote.vote == '1')

            # karma update
            vote._vote_update_karma('0', vote.vote)
        return votes

    def write(self, values):
        # can't modify owner of a vote
        if not self.env.is_admin():
            values.pop('user_id', None)

        for vote in self:
            vote._check_general_rights(values)
            vote_value = values.get('vote')
            if vote_value is not None:
                upvote = vote.vote == '-1' if vote_value == '0' else vote_value == '1'
                vote._check_karma_rights(upvote)

                # karma update
                vote._vote_update_karma(vote.vote, vote_value)

        res = super(Vote, self).write(values)
        return res

    def _check_general_rights(self, vals=None):
        if vals is None:
            vals = {}
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

    def _check_karma_rights(self, upvote=False):
        # karma check
        if upvote and not self.post_id.can_upvote:
            raise AccessError(_('%d karma required to upvote.', self.post_id.forum_id.karma_upvote))
        elif not upvote and not self.post_id.can_downvote:
            raise AccessError(_('%d karma required to downvote.', self.post_id.forum_id.karma_downvote))

    def _vote_update_karma(self, old_vote, new_vote):
        if self.post_id.parent_id:
            karma, reason = self._get_karma_value(
                old_vote,
                new_vote,
                self.forum_id.karma_gen_answer_upvote,
                self.forum_id.karma_gen_answer_downvote)
            source = _('Answer %s', reason)
        else:
            karma, reason = self._get_karma_value(
                old_vote,
                new_vote,
                self.forum_id.karma_gen_question_upvote,
                self.forum_id.karma_gen_question_downvote)
            source = _('Question %s', reason)
        self.recipient_id.sudo()._add_karma(karma, self.post_id, source)

```

## File: models\forum_tag.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.addons.http_routing.models.ir_http import slug, unslug
from odoo.exceptions import AccessError


class Tags(models.Model):
    _name = "forum.tag"
    _description = "Forum Tag"
    _inherit = [
        'mail.thread',
        'website.searchable.mixin',
        'website.seo.metadata',
    ]

    name = fields.Char('Name', required=True)
    forum_id = fields.Many2one('forum.forum', string='Forum', required=True, index=True)
    post_ids = fields.Many2many(
        'forum.post', 'forum_tag_rel', 'forum_tag_id', 'forum_id',
        string='Posts', domain=[('state', '=', 'active')])
    posts_count = fields.Integer('Number of Posts', compute='_compute_posts_count', store=True)
    website_url = fields.Char("Link to questions with the tag", compute='_compute_website_url')
    _sql_constraints = [
        ('name_uniq', 'unique (name, forum_id)', "Tag name already exists!"),
    ]

    @api.depends("post_ids", "post_ids.tag_ids", "post_ids.state", "post_ids.active")
    def _compute_posts_count(self):
        for tag in self:
            tag.posts_count = len(tag.post_ids)  # state filter is in field domain

    @api.depends("forum_id", "forum_id.name", "name")
    def _compute_website_url(self):
        for tag in self:
            tag.website_url = f'/forum/{slug(tag.forum_id)}/tag/{slug(tag)}/questions'

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            forum = self.env['forum.forum'].browse(vals.get('forum_id'))
            if self.env.user.karma < forum.karma_tag_create and not self.env.is_admin():
                raise AccessError(_('%d karma required to create a new Tag.', forum.karma_tag_create))
        return super(Tags, self.with_context(mail_create_nolog=True, mail_create_nosubscribe=True)).create(vals_list)

    # ----------------------------------------------------------------------
    # WEBSITE
    # ----------------------------------------------------------------------

    @api.model
    def _search_get_detail(self, website, order, options):
        search_fields = ['name']
        fetch_fields = ['id', 'name', 'website_url']
        mapping = {
            'name': {'name': 'name', 'type': 'text', 'match': True},
            'website_url': {'name': 'website_url', 'type': 'text', 'truncate': False},
        }
        base_domain = []
        if forum := options.get("forum"):
            forum_ids = (unslug(forum)[1],) if isinstance(forum, str) else forum.ids
            search_domain = options.get("domain")
            base_domain = [search_domain if search_domain is not None else [('forum_id', 'in', forum_ids)]]
        return {
            'model': 'forum.tag',
            'base_domain': base_domain,
            'search_fields': search_fields,
            'fetch_fields': fetch_fields,
            'mapping': mapping,
            'icon': 'fa-tag',
            'order': ','.join(filter(lambda f: 'is_published' not in f, order.split(','))),
        }

```

## File: models\gamification_challenge.py

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

## File: models\gamification_karma_tracking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class KarmaTracking(models.Model):
    _inherit = 'gamification.karma.tracking'

    def _get_origin_selection_values(self):
        return super()._get_origin_selection_values() + [('forum.post', self.env['ir.model']._get('forum.post').display_name)]

```

## File: models\ir_attachment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Attachment(models.Model):

    _inherit = "ir.attachment"

    def _can_bypass_rights_on_media_dialog(self, **attachment_data):
        # Bypass the attachment create ACL and let the user create the image
        # attachment if they have write access to the model (the image attachment
        # will be bound to this model's record).
        res_model = attachment_data['res_model']
        res_id = attachment_data.get('res_id')
        if (
            res_model == 'forum.post' and res_id
            and self.env['forum.post'].browse(res_id).can_use_full_editor
        ):
            return True

        return super()._can_bypass_rights_on_media_dialog(**attachment_data)

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models


class Users(models.Model):
    _inherit = 'res.users'

    create_date = fields.Datetime('Create Date', readonly=True, index=True)

    # Wrapper for call_kw with inherits
    def open_website_url(self):
        return self.mapped('partner_id').open_website_url()

    def get_gamification_redirection_data(self):
        res = super().get_gamification_redirection_data()
        res.append({
            'label': _('See our Forum'),
            'url': '/forum',
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

    forum_count = fields.Integer(readonly=True, default=0)

    @api.model_create_multi
    def create(self, vals_list):
        websites = super().create(vals_list)
        websites._update_forum_count()
        return websites

    def get_suggested_controllers(self):
        suggested_controllers = super(Website, self).get_suggested_controllers()
        suggested_controllers.append((_('Forum'), url_for('/forum'), 'website_forum'))
        return suggested_controllers

    def configurator_get_footer_links(self):
        links = super().configurator_get_footer_links()
        links.append({'text': _("Forum"), 'href': '/forum'})
        return links

    def configurator_set_menu_links(self, menu_company, module_data):
        # Forum menu should only be a footer link, not a menu
        forum_menu = self.env['website.menu'].search([('url', '=', '/forum'), ('website_id', '=', self.id)])
        forum_menu.unlink()
        super().configurator_set_menu_links(menu_company, module_data)

    def _search_get_details(self, search_type, order, options):
        result = super()._search_get_details(search_type, order, options)
        if search_type in ['forums', 'forums_only', 'all']:
            result.append(self.env['forum.forum']._search_get_detail(self, order, options))
        if search_type in ['forums', 'forum_posts_only', 'all']:
            result.append(self.env['forum.post']._search_get_detail(self, order, options))
        if search_type in ['forums', 'forum_tags_only', 'all']:
            result.append(self.env['forum.tag']._search_get_detail(self, order, options))
        return result

    def _update_forum_count(self):
        """ Update count of forum linked to some websites. This has to be
        done manually as website_id=False on forum model means a shared forum.
        There is therefore no straightforward relationship to be used between
        forum and website.

        This method either runs on self (if not void), either on all existing
        websites (to update globally counters, notably when a new forum is
        created). """
        websites = self if self else self.search([])
        forums_all = self.env['forum.forum'].search([])
        for website in websites:
            website.forum_count = len(forums_all.filtered_domain(website.website_domain()))

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import forum_forum
from . import forum_post
from . import forum_post_reason
from . import forum_post_vote
from . import forum_tag
from . import gamification_challenge
from . import gamification_karma_tracking
from . import ir_attachment
from . import res_users
from . import website

```

## File: populate\forum_forum.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools import populate

class Forum(models.Model):
    _inherit = 'forum.forum'
    _populate_sizes = {'small': 1, 'medium': 3, 'large': 10}

    def _populate_factories(self):
        return [
            ('name', populate.constant('Forum_{counter}')),
            ('description', populate.constant('This is forum number {counter}'))
        ]

```

## File: populate\forum_post.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from odoo import fields, models
from odoo.tools import populate

QA_WEIGHTS = {0: 25, 1: 35, 2: 20, 3: 10, 4: 4, 5: 3, 6: 2, 7: 1}
_logger = logging.getLogger(__name__)

class Post(models.Model):
    _inherit = 'forum.post'
    # Include an additional average of 2 post answers for each given size
    # e.g.: 100 posts as populated_model_records = ~300 actual forum.posts records
    _populate_sizes = {'small': 100, 'medium': 1000, 'large': 90000}
    _populate_dependencies = ['forum.forum', 'res.users']

    def _populate_factories(self):
        forum_ids = self.env.registry.populated_models['forum.forum']
        hours = [_ for _ in range(1, 49)]
        random = populate.Random('forum_posts')

        def create_answers(values=None, **kwargs):
            """Create random number of answers
            We set the `last_activity_date` to convert it to `create_date` in `_populate`
            as the ORM prevents setting these.
            """
            return [
                fields.Command.create({
                    'name': f"reply to {values['name']}",
                    'forum_id': values['forum_id'],
                    'content': 'Answer content',
                    'last_activity_date': fields.Datetime.add(values['last_activity_date'], hours=random.choice(hours)),
                })
                for _ in range(random.choices(*zip(*QA_WEIGHTS.items()))[0])
            ]

        def get_last_activity_date(iterator, *args):
            days = [_ for _ in range(3, 93)]
            now = fields.Datetime.now()

            for values in iterator:
                values.update(last_activity_date=fields.Datetime.subtract(now, days=random.choice(days)))
                yield values

        return [
            ('forum_id', populate.randomize(forum_ids)),
            ('name', populate.constant('post_{counter}')),
            ('last_activity_date', get_last_activity_date),  # Must be before call to 'create_answers'
            ('child_ids', populate.compute(create_answers)),
        ]

    def _populate(self, size):
        records = super()._populate(size)
        user_ids = self.env.registry.populated_models['res.users']

        # Overwrite auto-fields: use last_activity_date to update create date
        _logger.info('forum.post: update create date and uid')
        question_ids = tuple(records.ids)
        query = """
            SELECT setseed(0.5);
            UPDATE forum_post
               SET create_date = last_activity_date,
                   create_uid = floor(random() * (%(max_value)s - %(min_value)s + 1) + %(min_value)s)
             WHERE id in %(question_ids)s or parent_id in %(question_ids)s
        """
        self.env.cr.execute(query, {'question_ids': question_ids, 'min_value': user_ids[0], 'max_value': user_ids[-1]})

        _logger.info('forum.post: update last_activity_date of questions with answers')
        query = """
            WITH latest_answer AS(
                SELECT parent_id, max(last_activity_date) as answer_date
                  FROM forum_post
                 WHERE parent_id in %(question_ids)s
              GROUP BY parent_id
            )
            UPDATE forum_post fp
               SET last_activity_date = latest_answer.answer_date
              FROM latest_answer
             WHERE fp.id = latest_answer.parent_id
        """
        self.env.cr.execute(query, {'question_ids': question_ids})

        return records

```

## File: populate\mail_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from odoo import fields, models
from odoo.tools import populate

CP_WEIGHTS = {1: 35, 2: 30, 3: 25, 4: 10}
_logger = logging.getLogger(__name__)

class Message(models.Model):
    _inherit = 'mail.message'

    @property
    def _populate_dependencies(self):
        return super()._populate_dependencies + ["res.users", "forum.post"]

    def _populate(self, size):
        """Randomly assign messages to some populated posts and answers.
        This makes sure these questions and answers get one to four comments.
        Also define a date that is more recent than the post/answer's create_date
        """
        records = super()._populate(size)
        comment_subtype = self.env.ref('mail.mt_comment')
        hours = [_ for _ in range(1, 24)]
        random = populate.Random('comments_on_forum_posts')
        users = self.env["res.users"].browse(self.env.registry.populated_models['res.users'])
        vals_list = []
        question_ids = self.env.registry.populated_models['forum.post']
        posts = self.env['forum.post'].search(['|', ('id', 'in', question_ids), ('parent_id', 'in', question_ids)])
        for post in random.sample(posts, int(len(question_ids) * 0.7)):
            nb_comments = random.choices(*zip(*CP_WEIGHTS.items()))[0]
            for counter in range(nb_comments):
                vals_list.append({
                    "author_id": random.choice(users.partner_id.ids),
                    "body": f"message_body_{counter}",
                    "date": fields.Datetime.add(post.create_date, hours=random.choice(hours)),
                    "message_type": "comment",
                    "model": "forum.post",
                    "res_id": post.id,
                    "subtype_id": comment_subtype.id,
                })
        messages = self.env["mail.message"].create(vals_list)
        _logger.info('mail.message: update comments create date and uid')
        _logger.info('forum.post: update last_activity_date for posts with comments and/or commented answers')
        query = """
            WITH comment_author AS(
                SELECT mm.id, mm.author_id, ru.id as user_id, ru.partner_id
                  FROM mail_message mm
                  JOIN res_users ru
                    ON mm.author_id = ru.partner_id
                 WHERE mm.id in %(comment_ids)s
            ),
            updated_comments as (
                UPDATE mail_message mm
                   SET create_date = date,
                       create_uid = ca.user_id
                  FROM comment_author ca
                 WHERE mm.id = ca.id
             RETURNING res_id as post_id, create_date as comment_date
            ),
            max_comment_dates AS (
                SELECT post_id, max(comment_date) as last_comment_date
                  FROM updated_comments
              GROUP BY post_id
            ),
            updated_posts AS (
                UPDATE forum_post fp
                   SET last_activity_date = CASE  --on questions, answer could be more recent
                  WHEN fp.parent_id IS NOT NULL THEN greatest(last_activity_date, last_comment_date)
                  ELSE last_comment_date END
                  FROM max_comment_dates
                 WHERE max_comment_dates.post_id = fp.id
             RETURNING fp.id as post_id, fp.last_activity_date as last_activity_date, fp.parent_id as parent_id
            )
            UPDATE forum_post fp
               SET last_activity_date = greatest(fp.last_activity_date, up.last_activity_date)
              FROM updated_posts up
             WHERE up.parent_id = fp.id
    """
        self.env.cr.execute(query, {'comment_ids': tuple(messages.ids)})
        return records + messages

```

## File: populate\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import forum_forum
from . import forum_post
from . import mail_message

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_forum_forum_public,forum.forum,model_forum_forum,base.group_public,1,0,0,0
access_forum_forum_portal,forum.forum,model_forum_forum,base.group_portal,1,0,0,0
access_forum_forum_employee,forum.forum,model_forum_forum,base.group_user,1,0,0,0
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

## File: security\ir_rule_data.xml

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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><g clip-path="url(#o_icon_website_forum__a)"><path d="M26.001 34.5s-.83-2-.997-2.167c20.502-9.84 23.518-3.54 23.94-.955.068.418.005.843-.124 1.247A24.857 24.857 0 0 1 47 36.89c-1.5 1.5-10 1.61-10 1.61l-11-4Z" fill="#962B48"/><path d="m21 14 4.004 3.667c-20.502 9.84-23.518 3.54-23.94.955-.068-.418-.005-.843.124-1.247a24.84 24.84 0 0 1 1.428-3.514C6.713 5.644 14.501 11 14.501 11l6.5 3Z" fill="#1A6F66"/><path d="M1.187 32.625c-.129-.404-.192-.829-.124-1.247.422-2.586 3.438-8.886 23.94.955 14.197 6.815 20.01 5.89 22.382 3.818C43.286 44.36 34.803 50 25.003 50 13.855 50 4.411 42.703 1.187 32.625Z" fill="#FC868B"/><path d="M48.819 17.375c.129.404.192.829.124 1.247-.422 2.586-3.438 8.886-23.94-.955-14.197-6.815-20.01-5.89-22.382-3.818C6.72 5.64 15.203 0 25.003 0 36.15 0 45.594 7.297 48.819 17.375Z" fill="#1AD3BB"/></g><defs><clipPath id="o_icon_website_forum__a"><path fill="#fff" d="M0 0h50v50H0z"/></clipPath></defs></svg>

```

## File: static\src\components\flag_mark_as_offensive\flag_mark_as_offensive.js

```javascript
/** @odoo-module **/

import { Component, useEffect } from "@odoo/owl";
import { useChildRef } from "@web/core/utils/hooks";
import { Dialog } from "@web/core/dialog/dialog";

export class FlagMarkAsOffensiveDialog extends Component {
    static template = "website_forum.FlagMarkAsOffensiveDialog";
    static components = { Dialog };

    setup() {
        this.modalRef = useChildRef();

        const onClickDiscard = (ev) => {
            ev.preventDefault();
            this.props.close();
        };

        useEffect(
            (discardButton) => {
                if (discardButton) {
                    discardButton.addEventListener("click", onClickDiscard);
                    return () => {
                        discardButton.removeEventListener("click", onClickDiscard);
                    };
                }
            },
            () => [this.modalRef.el?.querySelector(".btn-link")]
        );
    }
}

```

## File: static\src\components\flag_mark_as_offensive\flag_mark_as_offensive.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

  <t t-name="website_forum.FlagMarkAsOffensiveDialog">
    <Dialog size="'md'" title="props.title" footer="false" modalRef="modalRef" >
      <t t-out="props.body"/>
    </Dialog>
  </t>

</templates>

```

## File: static\src\img\empty.svg

```svg
<?xml version="1.0" encoding="UTF-8"?><svg id="b" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 203.0829 243.7154"><g id="c"><g id="d"><polygon points="101.5414 238.2685 20.0382 190.7423 20.0382 101.8237 101.5414 148.9972 101.5414 238.2685" style="fill:#c1dbf6;"/><polygon points="101.5414 238.2685 183.044 191.095 183.044 101.8237 101.5414 148.9972 101.5414 238.2685" style="fill:#fff;"/><polygon points="183.044 101.8237 183.044 158.6371 111.3156 202.3055 101.5414 148.9972 183.044 101.8237" style="fill:#c1dbf6;"/><polygon points="20.8288 101.9859 101.5414 148.9972 101.5414 148.6905 101.5414 54.625 20.8288 101.9859" style="fill:#c1dbf6;"/><polygon points="101.5414 54.625 101.5414 148.9972 183.044 101.8237 101.5414 54.625" style="fill:#374874;"/><path d="m155.6613,118.1373l-23.2882-13.2862c-2.5378-1.4928-4.1799-4.1799-4.1799-7.1656h0c0-3.2842,3.5828-5.3742,6.4192-3.7321l24.6317,14.1819c2.09,1.1943,3.4335,3.5828,3.4335,5.9713h0c.1493,3.5828-3.8814,5.822-7.0163,4.0306Z" style="fill:#fff;"/><path d="m101.5414,54.625l81.5026,47.1988,14.3455,43.1314-14.3455,8.2813v37.8586l-81.5025,47.1734-81.5032-47.5262v-32.0983l-.0114-.007v-5.4006l-14.3455-8.2813,14.3455-43.1314,81.5146-47.1987m0-5.4541L17.6841,97.7379l-1.5591.894-.5672,1.7053L1.2123,143.4687l-1.2123,3.6449,3.3267,1.9204,11.9903,6.9217v5.3231l.0114.007v32.1621l2.3372,1.3629,83.8758,48.9045,83.8617-48.5441,2.3506-1.3604v-37.8551l11.9903-6.9217,3.3388-1.9273-1.2296-3.6538-14.4874-43.0493-.5615-1.6686-1.5184-.8911-83.7445-48.673h0Z" style="fill:#374874;"/><polygon points="183.044 101.8237 101.5414 54.625 20.2749 101.9673 20.0269 101.8237 5.6814 144.9551 87.3993 192.1287 101.5414 148.9972 24.9857 104.6936 101.5414 60.2454 178.3204 104.5575 101.5414 148.9972 115.6715 192.1287 197.3895 144.9551 183.044 101.8237" style="fill:#fff;"/><path d="m49.8818,64.403c-.6002,0-1.2004-.2277-1.6592-.6842l-24.5309-24.3699c-.9233-.9164-.9279-2.4078-.0115-3.33.9176-.9222,2.4101-.9268,3.33-.0115l24.5309,24.3699c.9233.9164.9279,2.4078.0115,3.33-.4599.4634-1.0659.6957-1.6707.6957Z" style="fill:#374874;"/><path d="m150.4164,64.403c-.6048,0-1.2108-.2323-1.6707-.6957-.9164-.9222-.9118-2.4135.0115-3.33l24.5309-24.3699c.9199-.9176,2.4124-.913,3.33.0115.9164.9222.9118,2.4135-.0115,3.33l-24.5309,24.3699c-.4588.4565-1.059.6842-1.6592.6842Z" style="fill:#374874;"/><path d="m101.5353,41.5659c-1.3005,0-2.3549-1.0544-2.3549-2.3549V2.3549c0-1.3005,1.0544-2.3549,2.3549-2.3549s2.3549,1.0544,2.3549,2.3549v36.8561c0,1.3005-1.0544,2.3549-2.3549,2.3549Z" style="fill:#374874;"/></g></g></svg>
```

## File: static\src\img\tasks.svg

```svg
<svg id="ehLJC65DH021" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 280 220" shape-rendering="geometricPrecision" text-rendering="geometricPrecision">
<style><![CDATA[
#ehLJC65DH0233_to {animation: ehLJC65DH0233_to__to 6300ms linear 1 normal forwards}@keyframes ehLJC65DH0233_to__to { 0% {transform: translate(40.708263px,221.555573px)} 6.349206% {transform: translate(40.708263px,221.555573px)} 7.936508% {transform: translate(77.243603px,228.644236px)} 9.52381% {transform: translate(83.948707px,218.43696px)} 14.285714% {transform: translate(132.622403px,135.864352px)} 100% {transform: translate(132.622403px,135.864352px)}} #ehLJC65DH0234_to {animation: ehLJC65DH0234_to__to 6300ms linear 1 normal forwards}@keyframes ehLJC65DH0234_to__to { 0% {offset-distance: 0%} 6.349206% {offset-distance: 0%} 7.936508% {offset-distance: 15.254728%} 9.52381% {offset-distance: 17.975347%} 11.111111% {offset-distance: 41.531645%} 14.285714% {offset-distance: 100%} 100% {offset-distance: 100%}} #ehLJC65DH0234 {animation: ehLJC65DH0234_c_o 6300ms linear 1 normal forwards}@keyframes ehLJC65DH0234_c_o { 0% {opacity: 0.5} 6.349206% {opacity: 0.5} 14.285714% {opacity: 1} 100% {opacity: 1}} #ehLJC65DH0235_to {animation: ehLJC65DH0235_to__to 6300ms linear 1 normal forwards}@keyframes ehLJC65DH0235_to__to { 0% {offset-distance: 0%;animation-timing-function: cubic-bezier(0,0,0.58,1)} 3.174603% {offset-distance: 0%;animation-timing-function: cubic-bezier(0,0,0.58,1)} 6.349206% {offset-distance: 19.237652%} 8.253968% {offset-distance: 28.606232%} 9.84127% {offset-distance: 28.606232%} 11.111111% {offset-distance: 64.552276%} 15.873016% {offset-distance: 64.552276%} 19.047619% {offset-distance: 100%} 100% {offset-distance: 100%}} #ehLJC65DH0235_tr {animation: ehLJC65DH0235_tr__tr 6300ms linear 1 normal forwards}@keyframes ehLJC65DH0235_tr__tr { 0% {transform: rotate(7217.43444deg);animation-timing-function: cubic-bezier(0,0,0.58,1)} 3.174603% {transform: rotate(7217.43444deg);animation-timing-function: cubic-bezier(0,0,0.58,1)} 4.761905% {transform: rotate(7243.789991deg)} 6.349206% {transform: rotate(7243.789991deg)} 7.936508% {transform: rotate(7211.010101deg)} 9.52381% {transform: rotate(7211.010101deg)} 11.111111% {transform: rotate(7259.905792deg)} 15.873016% {transform: rotate(7259.905792deg)} 19.047619% {transform: rotate(7217.43444deg)} 100% {transform: rotate(7217.43444deg)}}
]]></style>
<g transform="matrix(.726223 0 0 0.726223 25.717122 13.518461)"><g><g style="isolation:isolate"><path d="M84.9067,270.0492l-8.8673-5.1531c-1.1523-.6697-1.8672-2.093-1.873-4.1152l8.8673,5.1531c.0057,2.0222.7206,3.4455,1.873,4.1151Z" fill="#fbdbd0"/><path d="M202.69,7.5572l8.8673,5.1531c-1.1664-.6779-2.7814-.5822-4.561.4453L198.129,8.0025c1.7796-1.0275,3.3946-1.1231,4.561-.4453Z" fill="#fbdbd0"/><polygon points="83.0337,265.9341 74.1665,260.781 73.6745,87.234 82.5417,92.3871 83.0337,265.9341" fill="#fbdbd0"/><polygon points="88.939,81.316 80.0718,76.1629 198.129,8.0025 206.9962,13.1556 88.939,81.316" fill="#fbdbd0"/><path d="M206.9962,13.1556c3.5344-2.0406,6.4195-.4058,6.431,3.6647l.492,173.5469c.0115,4.0705-2.8548,9.0352-6.3892,11.0758L89.4728,269.6034c-3.5424,2.0452-6.4275.4012-6.4391-3.6693l-.492-173.5469c-.0115-4.0705,2.8549-9.0259,6.3973-11.0711L206.9962,13.1556Z" fill="#fff"/><path d="M82.5417,92.3871L73.6744,87.234c-.0115-4.0705,2.8549-9.0259,6.3973-11.0711L88.939,81.316c-3.5424,2.0452-6.4088,7.0006-6.3973,11.0711Z" fill="#fbdbd0"/></g><g style="isolation:isolate"><g style="isolation:isolate"><g clip-path="url(#ehLJC65DH0216)"><g><path d="M159.0888,25.5387l-8.8673-5.1531c1.9039-1.0992,3.6292-1.201,4.8752-.4769l8.8673,5.1531c-1.246-.7241-2.9713-.6223-4.8752.4769" fill="#c1dbf6"/></g><clipPath id="ehLJC65DH0216"><path d="M155.0968,19.9087l8.8672,5.1531c-1.246-.7241-2.9713-.6223-4.8752.4769l-8.8673-5.1531c1.9039-1.0992,3.6292-1.201,4.8752-.4769Z" fill="none"/></clipPath></g></g><polygon points="129.8807,59.0621 121.0134,53.9091 120.9886,45.1495 129.8559,50.3026 129.8807,59.0621" fill="#c1dbf6"/><polygon points="136.6855,38.4733 127.8182,33.3202 150.2215,20.3856 159.0888,25.5387 136.6855,38.4733" fill="#c1dbf6"/><g style="isolation:isolate"><g clip-path="url(#ehLJC65DH0224)"><g><path d="M129.8559,50.3026l-8.8673-5.1531c-.0123-4.3475,3.0462-9.645,6.8296-11.8293l8.8673,5.1531c-3.7834,2.1844-6.8419,7.4819-6.8296,11.8293" fill="#c1dbf6"/></g><clipPath id="ehLJC65DH0224"><path d="M129.8559,50.3026l-8.8673-5.1531c-.0123-4.3475,3.0462-9.645,6.8296-11.8293l8.8673,5.1531c-3.7834,2.1844-6.8419,7.4819-6.8296,11.8293Z" fill="none"/></clipPath></g></g><path d="M159.0888,25.5387c3.7834-2.1844,6.8619-.43,6.8743,3.9174l.0248,8.7595l4.5385-2.6203c3.7674-2.1751,6.8378-.4254,6.8501,3.9038l.0189,6.6695c.0021.7274-.3852,1.4004-1.0152,1.7641L121.5944,79.5633c-1.3514.7802-3.0411-.1922-3.0455-1.7526l-.0123-4.3404c-.0123-4.3292,3.0382-9.6128,6.8056-11.7879l4.5385-2.6203-.0248-8.7595c-.0123-4.3475,3.0462-9.645,6.8296-11.8293l22.4033-12.9346Z" fill="#c1dbf6"/></g><path d="M201.0358,7.1119c.5868,0,1.1066.1328,1.5316.3838c0,0,8.9769,5.2085,8.9772,5.2085c0,0,0,0,0,0v0c1.1571.6655,1.877,2.0879,1.8827,4.1157l.2624,92.5566.2297-1.6968-.1674,23.6319.1674,59.0557c.0116,4.0703-2.8548,9.0352-6.3892,11.0757L89.473,269.6032c-1.032.5962-2.0074.8774-2.8725.8774-.6375,0-1.2151-.1528-1.711-.4458v0c-.0012,0-8.8499-5.1387-8.8499-5.1387-1.1523-.6699-1.8672-2.0933-1.8729-4.1152L73.6746,87.2335c-.0115-4.0703,2.8549-9.0254,6.3973-11.0708l40.9377-23.6353-.0209-7.3779c-.0124-4.3477,3.0461-9.645,6.8295-11.8291l22.4034-12.9346c1.1041-.6377,2.1481-.9399,3.0735-.9399.6702,0,1.2783.1587,1.8017.4629l8.8672,5.1528c-.0034-.002-.0072-.0029-.0107-.0049-.0005-.0005-.001-.0005-.0015-.001.0005.0005.001.0005.0015.001.6556.3779,1.1744.9893,1.5225,1.7979L198.1291,8.0025c1.0551-.6089,2.0521-.8911,2.9067-.8906m.0003-4h-.0001c-1.5931-.0005-3.2899.4932-4.9067,1.4263L166.2626,21.7814c-.0543-.0347-.1088-.0693-.1641-.1025-.041-.0259-.0825-.0513-.1247-.0757l-8.8672-5.1528c-1.1302-.6567-2.4482-1.0039-3.8112-1.0044-1.6713,0-3.3786.4966-5.0744,1.4761L125.8182,29.8562c-5.0501,2.9155-8.8461,9.4951-8.8295,15.3047l.0143,5.061L78.0718,72.6985c-4.8029,2.7734-8.413,9.0269-8.3973,14.5464l.4921,173.5474c.0095,3.3843,1.4174,6.1406,3.8625,7.562l4.4263,2.5703c2.2759,1.3218,3.585,2.082,4.4203,2.519l-.0209.0352c1.1097.6558,2.4048,1.002,3.7455,1.002c1.609,0,3.2485-.4756,4.8729-1.4136L209.53,204.9075c4.7983-2.7705,8.4049-9.0264,8.3892-14.5513l-.1638-57.7646-.1064-37.4853-.222-78.2974c-.0096-3.3926-1.4241-6.1504-3.881-7.5674-.2045-.1187-8.9715-5.2056-8.9715-5.2056-1.0138-.5991-2.2468-.9238-3.5387-.9238v0Z" fill="#374874"/></g><g><path d="M200.531966,66.389409c1.426337-.823509,2.57805-.166918,2.582772,1.471925l.323664,114.143078c.004541,1.625948-1.139544,3.621148-2.566062,4.444656l-99.335004,57.351158c-1.414713.816788-2.578231.15384-2.582772-1.471926L98.6309,128.185222c-.004722-1.638843,1.151168-3.627686,2.566062-4.444474L200.531966,66.38959Zm.332019,117.101278l-.323664-114.143078-99.335004,57.351157.323664,114.143078l99.335004-57.351157" transform="translate(1.882949-11.712263)" fill="#374874"/><g mask="url(#ehLJC65DH0232)"><path style="isolation:isolate" d="M185.382784,113.866345c1.373483-.792994,2.745694-.956642,3.763909-.29987c1.95506,1.260146,1.861158,5.064558-.208147,8.501897L131.38814,217.692503c-.990425,1.649378-2.257291,2.915154-3.524884,3.646939-1.321718.763026-2.644163.945745-3.653659.377245L105.001548,210.75698c-2.024443-1.15353-2.034978-4.898367-.023794-8.381658.999871-1.731474,2.31723-3.069176,3.636405-3.830749s2.639803-.947016,3.646213-.373793l1.371847.783005l14.299511,8.161523l42.85803-71.21167l11.073771-18.399435c.991515-1.647016,2.25602-2.908615,3.519435-3.638039Z" transform="translate(3.882949-13.160476)" fill="#374874"/><mask id="ehLJC65DH0232" mask-type="luminance" x="-150%" y="-150%" height="400%" width="400%"><g id="ehLJC65DH0233_to" transform="translate(40.708263,221.555573)"><path d="M0,0h97.615151v35.936169c15.200495,4.506138,12.074533,12.085024,0,15.553138v67.240644h-97.615151L0,0Z" transform="scale(1.159569,1.410487) translate(-50.400241,-64.314561)" opacity="0.5" fill="#fff" stroke-width="0"/></g><g id="ehLJC65DH0234_to" style="offset-path:path('M27.816465,218.648707L48.878002,221.434557L52.583106,222.227279Q63.381835,222.942955,79.478639,204.77984C90.6842,190.43971,103.781004,161.276595,124.269705,136.884962');offset-rotate:0deg"><path id="ehLJC65DH0234" d="M0,0h97.615151v35.936169c15.200495,4.506138,12.074533,12.085024,0,15.553138v67.240644h-97.615151L0,0Z" transform="scale(1.159569,1.410487) translate(-39.282494,-62.253666)" opacity="0.5" fill="#fff" stroke-width="0"/></g></mask></g></g><g id="ehLJC65DH0235_to" style="offset-path:path('M165.924956,211.879051L165.924956,211.879051L109.085502,185.9537C109.085502,185.9537,135.271091,201.443,135.271091,201.443C135.271091,201.443,135.271091,201.443,135.271091,201.443C135.271091,201.443,193.029643,100.00187,193.029643,100.00187Q193.029643,100.00187,193.029643,100.00187Q193.029643,100.00187,165.924956,211.879051');offset-rotate:0deg"><g id="ehLJC65DH0235_tr" transform="rotate(7217.43444)"><g transform="translate(-160,-200)"><path d="M240.8319,76.3987c0,0,2.9577-.8451,5.0099.664c2.4706,1.8166,2.0523,3.8631,2.0523,3.8631l-5.16498,7.04317c.65442,1.21177.63798,2.01103.63798,2.01103l-66.2762,95.9737-10.92014,6.1426-2.66896,1.772c-.2921.194-.6716-.0672-.5948-.4093l.7143-3.18183l1.8803-11.56667.0001-.0002l65.914-96.698c0,0,.39102.00544,1.02229.06553l.00371-.00523c0,0,.6036-1.8108,3.8631-2.2333.99234-.12863,1.69935.02247,2.20279.2949l2.32431-3.7355Z" fill="none" stroke="#374874" stroke-width="6" stroke-linejoin="round" stroke-dashoffset="1"/><path d="M240.8319,76.3987c0,0,2.9577-.8451,5.0099.664c2.4706,1.8166,2.0523,3.8631,2.0523,3.8631l-5.9757,8.1487-6.1568-4.5271l5.0703-8.1487Z" fill="#fff"/><path d="M165.5017,178.7103l65.914-96.698c0,0,2.1056.0293,4.433.7028c1.206.349,2.4715.8709,3.5346,1.6513c4.0383,2.9645,3.9838,5.6136,3.9838,5.6136l-66.2762,95.9737-11.5893,6.519-1.9919-1.509l1.9919-12.2532Z" fill="#c1dbf6"/><path d="M219.8867,109.2024l19.8587-27.1901c0,0-.1811-2.5955-3.4406-2.173s-3.8631,2.2333-3.8631,2.2333l-17.7461,25.0497c0,0-2.3541,3.3753,5.191,2.08Z" fill="#fff"/><path d="M163.6908,189.9676l-.7838,3.4914c-.0768.3421.3027.6033.5948.4093l2.875-1.9088c0,0-.6201-1.052-.9356-1.2978-.3714-.2893-1.7505-.6941-1.7505-.6941Z" fill="#ececec"/></g></g></g></g></svg>

```

## File: static\src\js\website_forum.js

```javascript
/** @odoo-module **/

import { markup } from "@odoo/owl";
import { FlagMarkAsOffensiveDialog } from "../components/flag_mark_as_offensive/flag_mark_as_offensive";
import dom from "@web/legacy/js/core/dom";
import { cookie } from "@web/core/browser/cookie";;
import { loadWysiwygFromTextarea } from "@web_editor/js/frontend/loadWysiwygFromTextarea";
import publicWidget from "@web/legacy/js/public/public_widget";
import { session } from "@web/session";
import { escape } from "@web/core/utils/strings";
import { _t } from "@web/core/l10n/translation";
import { renderToFragment } from "@web/core/utils/render";

publicWidget.registry.websiteForum = publicWidget.Widget.extend({
    selector: '.website_forum',
    events: {
        'click .karma_required': '_onKarmaRequiredClick',
        'mouseenter .o_js_forum_tag_follow': '_onTagFollowBoxMouseEnter',
        'mouseleave .o_js_forum_tag_follow': '_onTagFollowBoxMouseLeave',
        'mouseenter .o_forum_user_info': '_onUserInfoMouseEnter',
        'mouseleave .o_forum_user_info': '_onUserInfoMouseLeave',
        'mouseleave .o_forum_user_bio_expand': '_onUserBioExpandMouseLeave',
        'click .o_wforum_flag:not(.karma_required)': '_onFlagAlertClick',
        'click .o_wforum_flag_validator': '_onFlagValidatorClick',
        'click .o_wforum_flag_mark_as_offensive': '_onFlagMarkAsOffensiveClick',
        'click .vote_up:not(.karma_required), .vote_down:not(.karma_required)': '_onVotePostClick',
        'click .o_wforum_validation_queue a[href*="/validate"]': '_onValidationQueueClick',
        'click .o_wforum_validate_toggler:not(.karma_required)': '_onAcceptAnswerClick',
        'click .o_wforum_favourite_toggle': '_onFavoriteQuestionClick',
        'click .comment_delete:not(.karma_required)': '_onDeleteCommentClick',
        'click .js_close_intro': '_onCloseIntroClick',
        'click .answer_collapse': '_onExpandAnswerClick',
        'submit .js_wforum_submit_form:has(:not(.karma_required).o_wforum_submit_post)': '_onSubmitForm',
    },

    init() {
        this._super(...arguments);
        this.rpc = this.bindService("rpc");
        this.orm = this.bindService("orm");
        this.notification = this.bindService("notification");
    },

    /**
     * @override
     */
    start: function () {
        var self = this;

        this.lastsearch = [];

        // float-start class messes up the post layout OPW 769721
        $('span[data-oe-model="forum.post"][data-oe-field="content"]').find('img.float-start').removeClass('float-start');

        // welcome message action button
        var forumLogin = `${window.location.origin}/web?redirect=${encodeURIComponent(window.location.href)}`
        $('.forum_register_url').attr('href', forumLogin);

        // Initialize forum's tooltips
        this.$('[data-bs-toggle="tooltip"]').tooltip({delay: 0});
        this.$('[data-bs-toggle="popover"]').popover({offset: '8'});

        $('input.js_select2').select2({
            tags: true,
            tokenSeparators: [',', ' ', '_'],
            maximumInputLength: 35,
            minimumInputLength: 2,
            maximumSelectionSize: 5,
            lastsearch: [],
            createSearchChoice: function (term) {
                if (self.lastsearch.filter(s => s.text.localeCompare(term) === 0).length === 0) {
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
                    return '<span class="badge bg-primary">New</span> ' + escape(term.text);
                } else {
                    return escape(term.text);
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
                    data.forEach((x) => {
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
                element.data("init-value").forEach((x) => {
                    data.push({id: x.id, text: x.name, isNew: false});
                });
                element.val('');
                callback(data);
            },
        });

        $('textarea.o_wysiwyg_loader').toArray().forEach(async (textarea) => {
            var $textarea = $(textarea);
            var editorKarma = $textarea.data('karma') || 0; // default value for backward compatibility
            var $form = $textarea.closest('form');
            var hasFullEdit = parseInt($("#karma").val()) >= editorKarma;
            let recordContent = '';
            let resId = 0;
            if (window.location.pathname.includes('edit')) {
                // Id is retrieved from URL, which is either:
                // - /forum/name-1/post/something-5
                // - /forum/name-1/post/something-5/edit
                // TODO: Make this more robust.
                resId = +window.location.pathname.split('-').slice(-1)[0].split('/')[0];
                const data = await this.orm.call("forum.post", "search_read", [], {
                    domain: [['id', '=', resId]],
                    fields: ['content'],
                });
                if (data && data.length) {
                    recordContent = data[0]['content'];
                }
            }
            var options = {
                toolbarTemplate: 'website_forum.web_editor_toolbar',
                toolbarOptions: {
                    showColors: false,
                    showFontSize: false,
                    showHistory: true,
                    showHeading1: false,
                    showHeading2: false,
                    showHeading3: false,
                    showLink: hasFullEdit,
                    showImageEdit: hasFullEdit,
                },
                recordInfo: {
                    context: self._getContext(),
                    res_model: 'forum.post',
                    res_id: resId,
                },
                value: recordContent,
                resizable: true,
                userGeneratedContent: true,
                height: 350,
            };
            options.allowCommandLink = hasFullEdit;
            options.allowCommandImage = hasFullEdit;
            loadWysiwygFromTextarea(self, $textarea[0], options).then(wysiwyg => {
                // float-start class messes up the post layout OPW 769721
                $form.find('.note-editable').find('img.float-start').removeClass('float-start');
            });
        });

        this.$('.o_wforum_bio_popover').toArray().forEach((authorBox) => {
            $(authorBox).popover({
                trigger: 'hover',
                offset: '10',
                animation: false,
                html: true,
                customClass: 'o_wforum_bio_popover_container shadow-sm',
            });
        });

        this.$('#post_reply').on('shown.bs.collapse', function (e) {
            const replyEl = document.querySelector('#post_reply');
            const scrollingElement = $(replyEl.parentNode).closestScrollable()[0];
            dom.scrollTo(replyEl, {
                forcedOffset: $(scrollingElement).innerHeight() - $(replyEl).innerHeight(),
            });
        });
        document.querySelectorAll('.o_wforum_question, .o_wforum_answer, .o_wforum_post_comment, .o_wforum_last_activity')
            .forEach((post) => {
                post.querySelector('.o_wforum_relative_datetime').textContent = luxon.DateTime
                    .fromSQL(post.dataset.lastActivity, {zone: 'utc'})
                    .toRelative();
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
        const fillableTextAreaEl = $form[0].querySelector(".o_wysiwyg_textarea_wrapper");
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
            let $textareaContainer = $form.find('.o_wysiwyg_textarea_wrapper');
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
                $buttons.toArray().forEach((btn) => {
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
    _onExpandAnswerClick: function (ev) {
        const expandableWindow = ev.currentTarget;
        if (ev.target.matches('.o_wforum_expand_toggle')) {
            expandableWindow.classList.toggle('o_expand')
            expandableWindow.classList.toggle('min-vh-100');
            expandableWindow.classList.toggle('w-lg-50');
        } else if (ev.target.matches('.o_wforum_discard_btn')){
            expandableWindow.classList.remove('o_expand', 'min-vh-100');
            expandableWindow.classList.add('w-lg-50');
        }
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onKarmaRequiredClick: function (ev) {
        const karma = parseInt(ev.currentTarget.dataset.karma);
        if (!karma) {
            return;
        }
        ev.preventDefault();
        if (session.is_website_user) {
            this._displayAccessDeniedNotification(
                markup(`<p>${_t('Oh no! Please <a href="%s">sign in</a> to vote', "/web/login")}</p>`)
            );
            return;
        }
        const forumId = parseInt(document.getElementById('wrapwrap').dataset.forum_id);
        const additionalInfoWithForumID = forumId
            ? markup(`<br/>
                <a class="alert-link" href="/forum/${forumId}/faq">
                    ${_t("Read the guidelines to know how to gain karma.")}
                </a>`)
            : "";
        const translatedText = _t("karma is required to perform this action. ");
        const message = markup(`${karma} ${translatedText}${additionalInfoWithForumID}`);
        this.notification.add(message, {
            type: "warning",
            sticky: false,
            title: _t("Karma Error"),
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
        ev.preventDefault();
        const elem = ev.currentTarget;
        this.rpc(
            elem.dataset.href || (elem.getAttribute('href') !== '#' && elem.getAttribute('href')) || elem.closest('form').getAttribute('action'),
        ).then(data => {
            if (data.error) {
                const message = data.error === 'anonymous_user'
                    ? _t("Sorry you must be logged to flag a post")
                    : data.error === 'post_already_flagged'
                        ? _t("This post is already flagged")
                        : data.error === 'post_non_flaggable'
                            ? _t("This post can not be flagged")
                            : data.error;
                this._displayAccessDeniedNotification(message);
            } else if (data.success) {
                const child = elem.firstElementChild;
                if (data.success === 'post_flagged_moderator') {
                    const countFlaggedPosts = this.el.querySelector('#count_posts_queue_flagged');
                    elem.innerText = _t(' Flagged');
                    elem.prepend(child);
                    if (countFlaggedPosts) {
                        countFlaggedPosts.classList.remove('bg-light');
                        countFlaggedPosts.classList.remove('d-none');
                        countFlaggedPosts.classList.add('bg-danger');
                        countFlaggedPosts.innerText = parseInt(countFlaggedPosts.innerText, 10) + 1;
                    }
                    $(elem).nextAll('.flag_validator').removeClass('d-none');
                } else if (data.success === 'post_flagged_non_moderator') {
                    elem.innerText = _t(' Flagged');
                    elem.prepend(child);
                    const $forumAnswer = $(elem).closest('.o_wforum_answer');
                    if ($forumAnswer) {
                        $forumAnswer.fadeIn(1000);
                        $forumAnswer.slideUp(1000);
                    }
                }
            }
        });
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onVotePostClick: function (ev) {
        ev.preventDefault();
        var $btn = $(ev.currentTarget);
        this.rpc($btn.data('href')).then(data => {
            if (data.error) {
                const message = data.error === 'own_post'
                    ? _t('Sorry, you cannot vote for your own posts')
                    : data.error === 'anonymous_user'
                        ? markup(`<p>${_t('Oh no! Please <a href="%s">sign in</a> to vote', "/web/login")}</p>`)
                        : data.error;
                this._displayAccessDeniedNotification(message);
            } else {
                var $container = $btn.closest('.vote');
                var $items = $container.children();
                var $voteUp = $items.filter('.vote_up');
                var $voteDown = $items.filter('.vote_down');
                var $voteCount = $items.filter('.vote_count');
                var userVote = parseInt(data['user_vote']);

                $voteUp.prop('disabled', userVote === 1);
                $voteDown.prop('disabled', userVote === -1);

                $items.removeClass('text-success text-danger text-muted opacity-75 o_forum_vote_animate');
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
                    $voteCount.addClass('text-muted opacity-75');
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
     * Call the route to moderate/validate the post, then hide the validated post
     * and decrement the count in the appropriate queue badge of the sidebar on success.
     *
     * @private
     * @param {Event} ev
     */
    _onValidationQueueClick: async function (ev) {
        ev.preventDefault();
        const approvalLink = ev.currentTarget;
        const postBeingValidated = this._findParent(approvalLink, '.post_to_validate');
        if (!postBeingValidated) {
            return;
        }
        postBeingValidated.classList.add('d-none');
        let ok;
        try {
            ok = (await fetch(approvalLink.href)).ok;
        } catch {
            // Calling the endpoint like this returns an HTML page. As we can't
            // extract the error message from that, we disregard it and simply
            // restore the post's visibility. This __should__ be improved.
        }
        if (!ok) {
            postBeingValidated.classList.remove('d-none');
            return;
        }
        const nbLeftInQueue = Array.from(document.querySelectorAll('.post_to_validate'))
            .filter(e => window.getComputedStyle(e).display !== 'none')
            .length;
        const queueType = document.querySelector('#queue_type').dataset.queueType;
        const queueCountBadge = document.querySelector(`#count_posts_queue_${queueType}`);
        queueCountBadge.innerText = nbLeftInQueue;
        if (!nbLeftInQueue) {
            document.querySelector('.o_caught_up_alert').classList.remove('d-none');
            document.querySelector('.o_wforum_btn_filter_tool')?.classList.add('d-none');
            queueCountBadge.classList.add('d-none');
        }
    },
    _findParent: function (el, selector) {
        while (el.parentElement && !el.matches(selector)) {
            el = el.parentElement;
        }
        return el.matches(selector) ? el : null;
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onAcceptAnswerClick: async function (ev) {
        ev.preventDefault();
        const link = ev.currentTarget;
        const target = link.dataset.target;
        const data = await this.rpc(link.dataset.href);
        if (data.error) {
            const message = data.error === 'anonymous_user'
                ? _t('Sorry, anonymous users cannot choose correct answers.')
                : data.error === 'own_post'
                    ? _t('Sorry, you cannot select your own posts as best answer')
                    : data.error;
            this._displayAccessDeniedNotification(message);
            return;
        }
        for (const answer of document.querySelectorAll('.o_wforum_answer')) {
            const isCorrect = answer.matches(target) ? data : false;
            const toggler = answer.querySelector('.o_wforum_validate_toggler');
            toggler.setAttribute('data-bs-original-title', isCorrect ? toggler.dataset.helperDecline : toggler.dataset.helperAccept);
            const styleForCorrect = isCorrect ? answer.classList.add : answer.classList.remove;
            const styleForIncorrect = isCorrect ? answer.classList.remove : answer.classList.add;
            styleForCorrect.call(answer.classList, 'o_wforum_answer_correct', 'my-2', 'mx-n3', 'mx-lg-n2', 'mx-xl-n3', 'py-3', 'px-3', 'px-lg-2', 'px-xl-3');
            styleForIncorrect.call(toggler.classList, 'opacity-50');
            const answerBorder = answer.querySelector('div .border-start');
            styleForCorrect.call(answerBorder.classList, 'border-success');
            const togglerIcon = toggler.querySelector('.fa');
            styleForCorrect.call(togglerIcon.classList, 'fa-check-circle', 'text-success');
            styleForIncorrect.call(togglerIcon.classList, 'fa-check-circle-o');
            const correctBadge = answer.querySelector('.o_wforum_answer_correct_badge');
            styleForCorrect.call(correctBadge.classList, 'd-inline');
            styleForIncorrect.call(correctBadge.classList, 'd-none');
        }
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onFavoriteQuestionClick: async function (ev) {
        ev.preventDefault();
        const link = ev.currentTarget;
        const data = await this.rpc(link.dataset.href);
        link.classList.toggle('opacity-50', !data);
        link.classList.toggle('opacity-100-hover', !data);
        const link_icon = link.querySelector('.fa');
        link_icon.classList.toggle('fa-star-o', !data);
        link_icon.classList.toggle('o_wforum_gold', data)
        link_icon.classList.toggle('fa-star', data)
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onDeleteCommentClick: function (ev) {
        ev.preventDefault();
        var $link = $(ev.currentTarget);
        var $container = $link.closest('.o_wforum_post_comments_container');

        this.rpc($link.closest('form').attr('action')).then(function () {
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
        cookie.set('forum_welcome_message', false, 24 * 60 * 60 * 365, 'optional');
        $('.forum_intro').slideUp();
        return true;
    },
    /**
     * @private
     * @param {Event} ev
     */
    async _onFlagValidatorClick(ev) {
        ev.preventDefault();
        const currentTarget = ev.currentTarget;
        await this.orm.call("forum.post", currentTarget.dataset.action, [
            parseInt(currentTarget.dataset.postId),
        ]);
        this._findParent(currentTarget, '.o_wforum_flag_alert')?.classList.toggle('d-none');
        const flaggedButton = currentTarget.parentElement.firstElementChild,
            child = flaggedButton.firstElementChild,
            countFlaggedPosts = this.el.querySelector('#count_posts_queue_flagged'),
            count = parseInt(countFlaggedPosts.innerText, 10) - 1;

        flaggedButton.innerText = _t(' Flag');
        flaggedButton.prepend(child);
        if (count === 0) {
            countFlaggedPosts.classList.add('bg-light');
        }
        countFlaggedPosts.innerText = count;
    },
    /**
     * @private
     * @param {Event} ev
     */
    async _onFlagMarkAsOffensiveClick(ev) {
        ev.preventDefault();
        const template = await this.rpc($(ev.currentTarget).data('action'));
        this.call("dialog", "add", FlagMarkAsOffensiveDialog, {
            title: _t("Offensive Post"),
            body: markup(template),
        });
    },
    _displayAccessDeniedNotification(message) {
        this.notification.add(message, {
            title: _t('Access Denied'),
            sticky: false,
            type: 'warning',
        });
    }
});

publicWidget.registry.websiteForumSpam = publicWidget.Widget.extend({
    selector: '.o_wforum_moderation_queue',
    events: {
        'click .o_wforum_select_all_spam': '_onSelectallSpamClick',
        'click .o_wforum_mark_spam': 'async _onMarkSpamClick',
        'input #spamSearch': '_onSpamSearchInput',
    },

    init() {
        this._super(...arguments);
        this.orm = this.bindService("orm");
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
        return this.orm.searchRead(
            "forum.post",
            [['id', 'in', self.spamIDs],
                '|',
                ['name', 'ilike', toSearch],
                ['content', 'ilike', toSearch]],
            ['name', 'content']
        ).then(function (o) {
            Object.values(o).forEach((r) => {
                r.content = $('<p>' + $(r.content).html() + '</p>').text().substring(0, 250);
            });
            self.$('div.post_spam').empty().append(renderToFragment('website_forum.spam_search_name', {
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
        var $inputs = this.$('.modal .tab-pane.active input.form-check-input:checked');
        var values = Array.from($inputs).map((o) => parseInt(o.value));
        return this.orm.call("forum.post", "mark_as_offensive_batch", [
            this.spamIDs,
            key,
            values,
        ]).then(function () {
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

```

## File: static\src\js\website_forum.share.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import "@website/js/content/snippets.animation";
import { renderToElement } from "@web/core/utils/render";

// FIXME There is no reason to inherit from socialShare here
var ForumShare = publicWidget.registry.socialShare.extend({
    selector: '',
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
    _render: function () {
        var $question = this.$('article.question');
        if (!this.targetType) {
            this._super.apply(this, arguments);
        } else if (this.targetType === 'social-alert') {
            $question.before(renderToElement('website.social_alert', {medias: this.socialList}));
        } else {
            const socialModalEl = renderToElement('website.social_modal', {
                medias: this.socialList,
                target_type: this.targetType,
                state: $question.data('state'),
            });
            document.body.append(socialModalEl);
            $('#oe_social_share_modal').modal('show');
        }
    },
    /**
    * @override
    * TODO remove me in master. This has been introduced as a stable fix to not
    * remove the document body at the `destroy()` of the `ForumShare` public
    * widget.
    *
    * Background: The `ForumShare` public widget is initially attached to the document
    * body upon instantiation, which means its root element (`this.$el`) is set
    * to the document body. Normally, when a widget is destroyed, its root
    * element is removed which, in this case, would result in the document body
    * removal.
    *
    * To prevent this, the fix assigns `null` to the root element before
    * invoking the `destroy()` method, ensuring that the document body remains
    * intact.
    */
    destroy: function () {
        this.setElement(null);
        const socialModalEl = document.querySelector("body #oe_social_share_modal");
        if (socialModalEl) {
            socialModalEl.remove();
        }
        this._super();
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

        return this._super.apply(this, arguments);
    },
});

```

## File: static\src\js\systray_items\forum_forum_add_form.js

```javascript
/** @odoo-module **/

import {NewContentFormController, NewContentFormView} from '@website/js/new_content_form';
import {registry} from "@web/core/registry";

export class AddForumFormController extends NewContentFormController {
    /**
     * @override
     */
    computePath() {
        return `/forum/${encodeURIComponent(this.model.root.resId)}`;
    }
}

export const AddForumFormView = {
    ...NewContentFormView,
    Controller: AddForumFormController,
};

registry.category("views").add("website_forum_add_form", AddForumFormView);

```

## File: static\src\js\systray_items\new_content.js

```javascript
/** @odoo-module **/

import { NewContentModal, MODULE_STATUS } from '@website/systray_items/new_content';
import { patch } from "@web/core/utils/patch";

patch(NewContentModal.prototype, {
    setup() {
        super.setup();

        const newForumElement = this.state.newContentElements.find(element => element.moduleXmlId === 'base.module_website_forum');
        newForumElement.createNewContent = () => this.onAddContent('website_forum.forum_forum_action_add');
        newForumElement.status = MODULE_STATUS.INSTALLED;
        newForumElement.model = 'forum.forum';
    },
});

```

## File: static\src\js\tours\website_forum.js

```javascript
/** @odoo-module **/

    import { _t } from "@web/core/l10n/translation";
    import wTourUtils from "@website/js/tours/tour_utils";

    wTourUtils.registerBackendAndFrontendTour("question", {
        url: '/forum/1',
    }, () => [{
        trigger: ".o_wforum_ask_btn",
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
        trigger: ".modal-header button.btn-close",
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
        trigger: ".modal-header button.btn-close",
        auto: true,
    }, {
        trigger: ".o_wforum_validate_toggler[data-karma]:first",
        content: _t("Click here to accept this answer."),
        position: "right",
    }, {
        content: "Check edit button is there",
        trigger: "a:contains('Edit your answer')",
        auto: true,
        isCheck: true,
    }]);

```

## File: static\src\xml\public_templates.xml

```xml
<templates id="template" xml:space="preserve">
    <t t-name="website.social_modal">
        <div role="dialog" class="modal fade" id="oe_social_share_modal">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <header class="modal-header mb0" role="status">
                        <h4 class="modal-title">Thanks for posting!</h4>
                        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                    </header>
                    <main class="modal-body">
                        <t t-if="target_type == 'question'" t-call="website_forum.social_message_question"/>
                        <t t-if="target_type == 'answer'" t-call="website_forum.social_message_answer"/>
                        <t t-if="target_type == 'default'" t-call="website_forum.social_message_default"/>
                        <div t-if="state != 'pending'" class="share-icons text-center text-primary">
                            <t t-foreach="medias" t-as="media" t-key="media_index">
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
        <t t-foreach="posts" t-as="post" t-key="post_index">
            <div class="card mb-1 o_spam_character">
                <div class="card-body py-2">
                    <div class="form-check">
                        <input type="checkbox" class="form-check-input" t-attf-id="post_#{post.id}" t-att-value='post.id' checked='checked'/>
                        <label class="form-check-label" t-attf-for="post_#{post.id}">
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
        answer</b>!</p>
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
            <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
            <p>Share this post on social networks.</p><br/>
            <div>
                <t t-foreach="medias" t-as="media" t-key="media_index">
                    <a style="cursor: pointer" t-attf-class="fa-stack fa-lg share #{media}" t-attf-aria-label="Share on #{media}" t-attf-title="Share on #{media}">
                        <span class="fa fa-square fa-stack-2x"></span>
                        <span t-attf-class="oe_social_#{media} fa fa-#{media} fa-stack-1x fa-inverse"></span>
                    </a>
                </t>
            </div>
        </div>
    </t>
</templates>

```

## File: views\base_contact_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

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
                    <t t-esc="separator"/>
                    <b>|</b>
                    <span class="fa fa-trophy o-text-gold ms-2" role="img" aria-label="Gold badge" title="Gold badge"/>
                    <t t-esc="object.gold_badge"/>
                    <span class="fa fa-trophy o-text-silver ms-2" role="img" aria-label="Silver badge" title="Silver badge"/>
                    <t t-esc="object.silver_badge"/>
                    <span class="fa fa-trophy o-text-bronze ms-2" role="img" aria-label="Bronze badge" title="Bronze badge"/>
                    <t t-esc="object.bronze_badge"/>
                </div>
                <t t-out="0"/>
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

</data></odoo>

```

## File: views\forum_forum_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

<!-- Specific Forum Layout -->
<template id="forum_index" name="Forum">
    <t t-set="no_filters" t-value="filters not in ('solved', 'unsolved', 'unanswered')"/>
    <t t-if="my == 'mine'">
        <t t-set="_page_name" t-value="'my_posts'"/>
        <t t-set="_page_name_label">My Posts</t>
    </t>
    <t t-elif="my == 'favourites'">
        <t t-set="_page_name" t-value="'my_favourites'"/>
        <t t-set="_page_name_label">My Favorites</t>
    </t>
    <t t-else="" t-set="_page_name" t-value="'list_questions'"/>

    <t t-call="website_forum.header">
        <!-- List questions or search/filters result -->
        <table t-if="question_count != 0" class="o_wforum_table table mb-5" height="1">
            <thead>
                <tr class="d-none d-lg-table-row">
                    <th class="o_wforum_table_title"></th>
                    <th class="o_wforum_table_posters"></th>
                    <th class="o_wforum_table_replies small fw-normal text-center">
                        <a class="text-muted" t-attf-href="?#{ keep_query('search', 'filters', 'my', 'create_uid', sorting='child_count asc' if sorting == 'child_count desc' else 'child_count desc') }">
                            Replies <i t-attf-class="fa fa-caret-#{ 'down' if sorting == 'child_count desc' else 'up' } #{ not sorting.startswith('child_count') and 'd-none ' }"></i>
                        </a>
                    </th>
                    <th class="o_wforum_table_views small fw-normal text-center">
                        <a class="text-muted" t-attf-href="?#{ keep_query('search', 'filters', 'my', 'create_uid', sorting='views asc' if sorting == 'views desc' else 'views desc') }">
                            Views <i t-attf-class="fa fa-caret-#{'down' if sorting == 'views desc' else 'up'}  #{ not sorting.startswith('views') and 'd-none ' }"></i>
                        </a>
                    </th>
                    <th class="o_wforum_table_activity small fw-normal text-center">
                        <a class="text-muted" t-attf-href="?#{ keep_query('search', 'filters', 'my', 'create_uid', sorting='last_activity_date asc' if sorting == 'last_activity_date desc' else 'last_activity_date desc') }">
                            Activity <i t-attf-class="fa fa-caret-#{ 'down' if sorting == 'last_activity_date desc' else 'up'}  #{ 'd-none ' if not sorting.startswith('last_activity') else '' }"></i>
                        </a>
                    </th>
                </tr>
            </thead>
            <t t-foreach="question_ids" t-as="question" t-call="website_forum.display_post">
                <t t-set="show_author_avatar" t-value="true"/>
            </t>
        </table>
        <t t-if="question_count == 0 or original_search" t-call="website_forum.no_results_message">
            <t t-set="record_name_plural">posts</t>
            <t t-set="_forum_slug" t-value="slug(forum) if forum else 'all'"/>
            <t t-set="go_back_url" t-valuef="/forum/#{_forum_slug}/"/>
        </t>
        <t t-call="website.pager"/>
    </t>
</template>

<!-- Edition: ask your question -->
<template id="new_question" name="New Post">
    <t t-set="_page_name" t-value="'new_question'"/>
    <t t-set="_page_name_label">Ask your question</t>
    <t t-call="website_forum.header">
        <div t-if="forum and forum.has_pending_post" class="alert border" role="alert">
            <b>You already have a pending post.</b><br/>
            <p>Please wait for a moderator to validate your previous post before continuing.</p>
            <a t-attf-href="/forum/#{ slug(forum) }" title="All Topics"><i class="fa fa-angle-left me-2"/>Back to All Posts</a>
        </div>

        <form t-else="" t-attf-action="/forum/#{slug(forum)}/new" method="post" role="form" class="tag_text js_website_submit_form js_wforum_submit_form o_wforum_readable mt-lg-3">
            <div class="row mb-3">
                <label class="form-label col-lg-2" for="content">Title</label>
                <div class="col">
                    <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                    <input type="text" name="post_name" required="required" pattern=".*\S.*" t-att-value="post_name"
                        class="form-control" placeholder="Write a clear, explicit and concise title" title="Title must not be empty"/>
                    <input type="hidden" name="karma" t-att-value="user.karma" id="karma"/>
                    <div class="form-text small text-muted">
                        <a data-bs-toggle="collapse" href="#newQuestionExample" role="button" aria-expanded="false" aria-controls="newQuestionExample">
                            Example
                            <i class="fa fa-question-circle"/>
                        </a>
                        <div class="collapse" id="newQuestionExample">
                            <div class="mt-2 text-success">
                                <i class="fa fa-check"/> How to configure TPS and TVQ's canadian taxes?
                            </div>
                            <div class="text-danger">
                                <i class="fa fa-times"/> Good morning to all! Please, can someone help solve my tax computation problem in Canada? Thanks!
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="row mb-3">
                <label class="form-label col-lg-2" for="content">Description</label>
                <div class="col">
                    <textarea name="content" required="required" id="content" class="form-control o_wysiwyg_loader" t-att-data-karma="forum.karma_editor">
                        <t t-out="question_content"/>
                    </textarea>
                    <span class="form-text d-inline small text-muted"><i class="fa fa-lightbulb-o"></i> Tip: consider adding an example. </span>
                </div>
            </div>
            <div class="row mb-2">
                <label class="form-label col-lg-2" for="post_tags">Tags</label>
                <div class="col">
                    <input type="hidden" name="karma_tag_create" t-att-value="forum.karma_tag_create" id="karma_tag_create"/>
                    <input type="hidden" name="karma_edit_retag" t-att-value="forum.karma_edit_retag" id="karma_edit_retag"/>
                    <input type="hidden" name="post_tags" placeholder="Tags" class="form-control js_select2"/>
                </div>
            </div>
            <div class="row mb-5">
                <div class="col offset-lg-2">
                    <button type="submit" t-attf-class="o_wforum_submit_post #{ 'oe_social_share_call' if forum.allow_share else '' } btn btn-primary #{ 'karma_required' if user.karma &lt; forum.karma_ask else ''}"
                    t-att-data-karma="forum.karma_ask"
                    data-hashtags="#question" data-social-target-type="question">Post Your Question</button>
                    <a class="btn btn-link" title="Back to Question" t-attf-href="/forum/#{ slug(forum) }">Discard</a>
                </div>
            </div>
        </form>
    </t>
</template>

<!-- Edition: edit a post -->
<template id="edit_post" name="Edit Post">
    <t t-if="is_answer">
        <t t-set="_page_name" t-value="'edit_answer'"/>
        <t t-set="_page_name_label">Edit your answer</t>
    </t>
    <t t-else="">
        <t t-set="_page_name" t-value="'edit_question'"/>
        <t t-set="_page_name_label">Edit your question</t>
    </t>
    <t t-call="website_forum.header">
        <div t-if="is_answer" class="row g-0">
            <div class="col-lg-2 pt-3">
                <label class="fw-bold">Question</label>
            </div>
            <article class="o_wforum_readable alert col bg-light">
                <i class="text-muted fw-bold" t-out="post.parent_id.name"/>
                <i t-field="post.parent_id.content" class="o_wforum_post_content oe_no_empty pb-3 text-muted text-break"/>
                <small class="text-end text-muted">by <t t-out="post.parent_id.create_uid.sudo().name" /></small>
            </article>
        </div>

        <form t-attf-action="/forum/#{slug(forum)}/post/#{slug(post)}/save" method="post" role="form" class="tag_text js_website_submit_form js_wforum_submit_form o_wforum_readable">
            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
            <div t-if="not is_answer" class="row mb-3">
                <label class="form-label col-lg-2" for="post_name">Title</label>
                <div class="col">
                    <input type="text" name="post_name" required="required" pattern=".*\S.*" t-att-value="post.name"
                        class="form-control" placeholder="Edit your Post" title="Title must not be empty"/>
                    <div class="form-text small text-muted">
                        <a data-bs-toggle="collapse" href="#newQuestionExample" role="button" aria-expanded="false" aria-controls="newQuestionExample">
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
            </div>
            <div class="row">
                <label t-if="not is_answer" class="form-label col-lg-2" for="content">Description</label>
                <label t-else="" class="form-label col-lg-2">Your Answer</label>
                <div class="col">
                    <textarea name="content" id="content" required="required" class="form-control o_wysiwyg_loader" t-att-data-karma="forum.karma_editor" t-out="post.content"/>
                    <span t-if="not is_answer" class="form-text d-inline small text-muted"><i class="fa fa-lightbulb-o"/> Tip: consider adding an example. </span>
                </div>
            </div>
            <input type="hidden" name="karma" t-att-value="user.karma" id="karma"/>
            <div t-if="not is_answer" class="row mb-2">
                <label class="form-label col-lg-2" for="post_tags">Tags</label>
                <div class="col">
                    <input type="hidden" name="karma_tag_create" t-att-value="forum.karma_tag_create" id="karma_tag_create"/>
                    <input type="hidden" name="karma_edit_retag" t-att-value="forum.karma_edit_retag" id="karma_edit_retag"/>
                    <t t-set="edit_tags_karma_fail" t-value="user.karma &lt; forum.karma_edit_retag"/>
                    <t t-set="edit_tags_karma_error_message">You need to have sufficient karma to edit tags</t>
                    <input type="text" name="post_tags" class="form-control js_select2" placeholder="Tags" t-attf-data-init-value="#{tags}" value="Tags"
                        t-att-readonly="edit_tags_karma_fail and 'readonly'"
                        t-att-title="edit_tags_karma_fail and edit_tags_karma_error_message"/>
               </div>
            </div>
            <div class="row mb-5">
                <div class="col offset-lg-2">
                    <button type="submit" class="o_wforum_submit_post btn btn-primary">Save Changes</button>
                    <a class="btn btn-link" title="Back to Question" t-attf-href="/forum/#{ slug(forum) }/#{ slug(post)}">
                        Discard
                    </a>
                </div>
            </div>
        </form>
    </t>
</template>

<template id="forum_index_tags" name="Forum Tags">
    <t t-set="_page_name" t-valuef="tags"/>
    <t t-set="_page_name_label">Tags</t>
    <t t-call="website_forum.header">
        <t t-if="len(pager_tag_chars) &gt; 1">
            <t t-if="len(pager_tag_chars) &lt; 11">
                <ul class="pagination overflow-auto mt0 mb0">
                    <t t-foreach="pager_tag_chars" t-as="tuple_char">
                        <li t-if="tuple_char_index &lt; 11" t-attf-class="page-item #{ 'active' if active_char_tag == tuple_char[1] else '' }">
                            <a t-attf-href="/forum/#{ slug(forum) }/tag/#{ quote_plus(tuple_char[1] )}?#{ keep_query('filters', 'search') }"
                               class="page-link">
                                <t t-out="tuple_char[0]"/>
                            </a>
                        </li>
                    </t>
                </ul>
            </t>
            <div t-else="" class="d-flex align-items-center" role="toolbar" aria-label="Toolbar with button groups">
                <label for="filter" class="me-2">Show Tags Starting with</label>
                <select name="filter" class="form-select w-auto" onchange="location = this.value;">
                    <t t-foreach="pager_tag_chars" t-as="tuple_char">
                        <option t-if="active_char_tag == tuple_char[1]" selected="selected" value="" t-out="tuple_char[0]"/>
                        <option t-else="" t-attf-value="/forum/#{slug(forum)}/tag/#{quote_plus(tuple_char[1]) + '?' + keep_query('filters')}"
                                t-out="tuple_char[0]"
                        />
                    </t>
                </select>
            </div>
        </t>

        <div class="row g-0 mt-3 mb-5" t-if="tags">
            <div class="col-3 pe-3 o_js_forum_tag_follow" t-foreach="tags" t-as="tag">
                <span t-attf-class="#{ 'd-block' if request.env.user._is_public() else 'd-flex align-items-baseline'}">
                    <t t-call="website_forum.follow">
                        <t t-set="email" t-value="user_id.email"/>
                        <t t-set="object" t-value="tag"/>
                        <t t-set="icons_design" t-value="True"/>
                        <t t-set="div_class" t-value="'d-inline'"/>
                        <t t-set="btn_classes" t-value="'px-2'"/>
                    </t>
                    <a t-attf-class="#{ 'disabled' if len(tag.post_ids) == 0  else '' } btn-link #{ 'text-dark fw-bold' if tag.message_is_follower else '' }"
                       t-attf-href="/forum/#{ slug(forum) }/tag/#{ slug(tag) }/questions?#{ keep_query(filters='tag') }">
                        <t t-out="tag.name"/>&amp;nbsp;
                        <span class="small">(<t t-out="tag.posts_count"/>)</span>
                    </a>
                </span>
            </div>
        </div>
        <t t-else="">
            <t t-set="tag_filter" t-value="keep_query('filters').split('=')[1] if keep_query('filters') else ''"/>
            <t t-set="go_back_url" t-valuef="/forum/#{ slug(forum) }/tag"/>
            <t t-set="record_name_plural">tags</t>
            <t t-call="website_forum.no_results_message">
                <t t-if="tag_filter and tag_filter != 'all'" t-set="result_msg">
                    There are no tags matching the selected filter <t t-if="search">and search</t>.
                </t>
                <t t-elif="search" t-set="result_msg">
                    There are no tags matching the selected search.
                </t>
                <t t-else="" t-set="result_msg">
                    There are no tags being used in this forum.
                </t>
            </t>
        </t>
    </t>
</template>

    </data>
</odoo>

```

## File: views\forum_forum_templates_forum_all.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

<!-- All Forums Layout -->
<template id="forum_all" name="Forum Navigation">

    <t t-set="col_class" t-valuef="col-md col-xl-3 col-xxl-2 mb-3"/>
    <t t-set="img_class" t-valuef="col-md-4"/>
    <t t-set="content_class" t-valuef="col-md-8"/>

    <t t-call="website.layout">
        <t t-set="pageName" t-value="'website_forum'"/>
        <div id="wrap">
            <div class="oe_structure oe_empty" id="oe_structure_forum_all_top"/>
            <div id="o_wforum_forums_index_list" class="container pt-lg-4 pb-5">
                <div class="row justify-content-center">
                    <t t-if="forums" t-call="website_forum.forum_all_all_entries">
                        <t t-set="_forums" t-value="forums"/>
                    </t>
                    <div t-else="" class="alert alert-info">No forum is available yet.</div>
                </div>
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
            <a t-attf-href="/forum/#{slug(forum)}"
                class="card oe_img_bg h-100 justify-content-end shadow-sm border-0 p-3 pt-5 text-reset text-decoration-none transition-base"
                t-attf-style="background-image: url('#{website.image_url(forum, 'image_1920') if forum.image_1920 else ''}')"
            >
                <div class="o_we_bg_filter o_wforum_background_gradient"/>
                <div t-if="forum_badge" class="badge position-absolute top-0 end-0 m-3 text-bg-secondary" t-out="forum_badge"></div>
                <div class="mt-5 pt-5">
                    <h3 class="mb-0 fw-bold" t-field="forum.name"/>
                    <t t-if="is_view_active('website_forum.opt_post_count')">
                        <span t-if="forum.total_posts > 0" class="badge text-bg-dark">
                            <span t-out="forum.total_posts" class="me-1"/>
                            <t t-if="forum.total_posts > 1">posts</t>
                            <t t-else="">post</t>
                        </span>
                        <span t-else="" class="badge text-dark">No posts yet</span>
                    </t>
                    <div t-if="is_view_active('website_forum.opt_last_post') and forum.last_post_id" class="text-truncate">
                        <span class="badge bg-dark me-1">Last post:</span>
                        <object>
                            <a
                                t-attf-href="/forum/#{slug(forum)}/#{slug(forum.last_post_id)}"
                                t-out="forum.last_post_id.name"
                            />
                        </object>
                    </div>
                </div>
            </a>
        </div>
    </t>
</template>

<!-- (Options) Forum : List View
    Display forums as a list  -->
<template name="List View" id="website_forum.opt_list_view" inherit_id="website_forum.forum_all" active="False">
    <xpath expr="//t[@t-set='col_class']" position="attributes">
        <attribute name="t-valuef">col-8 mb-3</attribute>
    </xpath>
    <xpath expr="//t[@t-set='img_class']" position="attributes">
        <attribute name="t-valuef">col-md-3</attribute>
    </xpath>
    <xpath expr="//t[@t-set='content_class']" position="attributes">
        <attribute name="t-valuef">col-md-9</attribute>
    </xpath>
</template>

<!-- (Options) Forum : Show Post Count
    Show the number of post a forum has  -->
<template name="Show Post Count" id="website_forum.opt_post_count" inherit_id="website_forum.forum_all" active="False"/>

<!-- (Options) Forum : Show Last Post
    Show the title of the latest post in each forum  -->
<template name="Show Last Post" id="website_forum.opt_last_post" inherit_id="website_forum.forum_all" active="False"/>

<!-- Default content for the "All Forums Layout" header above -->
<!-- (simulate an oe_structure edition) -->
<template id="forum_all_oe_structure_forum_all_top" inherit_id="website_forum.forum_all" name="Forum Navigation (oe_structure_forum_all_top)">
    <xpath expr="//*[hasclass('oe_structure')][@id='oe_structure_forum_all_top']" position="replace">
        <div class="oe_structure oe_empty" id="oe_structure_forum_all_top">
            <section class="s_cover o_colored_level s_parallax_no_overflow_hidden pt96 pb48" data-scroll-background-ratio="0" data-snippet="s_cover" data-name="Cover">
                <div class="s_allow_columns container">
                    <h1 class="display-3" style="text-align: center; font-weight: bold;">Community Forums</h1>
                    <p class="lead" style="text-align: center;">Tap into the collective knowledge of our community by asking your questions in our forums,<br/> where helpful members are ready to assist you.</p>
                    <p style="text-align: center;">
                        <a class="mb-2" t-attf-href="/profile/users?forum_origin=#{request.httprequest.path}" data-bs-original-title="" title="" target="_blank">Meet our community members</a>
                    </p>
                </div>
            </section>
        </div>
    </xpath>
</template>

    </data>
</odoo>

```

## File: views\forum_forum_templates_layout.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

<!-- GLOBAL LAYOUT -->
<!-- ============================================================ -->

<!-- website_forum.layout removes the access right check for wysiwyg bundle -->
<template id="layout" inherit_id="website.layout" name="Forum Layout" primary="True">
    <xpath expr="//div[@id='wrapwrap']" position="before">
        <t t-set="pageName" t-value="'website_forum'"/>
    </xpath>
    <xpath expr="//div[@id='wrapwrap']" position="attributes">
        <attribute name="t-att-data-forum_id">forum and forum.id</attribute>
    </xpath>
</template>

<!-- PAGE INDEX -->
<!-- ============================================================ -->
<template id="forum_model_nav">
    <t t-set="_forum_slug" t-value="slug(forum) if forum else 'all'"/>
    <t t-set="_all_forums">All forums</t>
    <t t-set="_forum_name" t-value="forum.name if forum else _all_forums"/>
    <!-- allows to be overridden such as in website_slides_forum -->
    <t t-set="breadcrumb_kind" t-valuef="base"/>
    <nav id="o_wforum_nav" t-attf-class="navbar d-flex gap-2 #{ 'mw-xl-75 mw-xxl-100' if _page_name == 'single_question' else '' } px-0" aria-label="breadcrumb">
        <t t-if="_page_name == 'single_question'">
            <div class="flex-grow-1">
                <div class="o_wforum_breadcrumb_root_single row g-0">
                    <div t-if="breadcrumb_kind == 'base'" class="col-10">
                        <a class="btn btn-link px-0 pb-2 fs-5" t-attf-href="/forum/#{ _forum_slug }">
                            <i class="d-inline-block fa fa-angle-left me-1 small"/><t t-out="_forum_name"/>
                        </a>
                    </div>
                    <div class="d-lg-none col-2 text-end">
                        <button class="btn position-relative ms-auto fs-5" data-bs-toggle="offcanvas" data-bs-target="#o_wforum_offcanvas">
                            <i class="fa fa-navicon"/>
                        </button>
                    </div>
                </div>
                <div class="d-flex gap-2 align-items-baseline">
                    <h3 t-attf-class="col-lg-10 my-0" t-out="question.name"/>
                    <div class="col d-flex justify-content-end align-items-center">
                        <i t-if="question.state == 'close'" class="fa fa-lock ms-2 fs-4" title="Closed" data-bs-toggle="tooltip" data-bs-placement="top"/>
                        <span t-elif="not question.active" class="badge bg-danger">
                            <t t-if="question.state!='offensive'">Deleted</t>
                            <t t-if="question.state=='offensive'">Offensive</t>
                            <t t-if="question.state=='offensive' and question.closed_reason_id">
                                <t t-out="question.closed_reason_id.name.capitalize()"/>
                            </t>
                        </span>
                        <a t-elif="uid" type="button" aria-label="Favorite"
                            t-attf-data-href="/forum/#{slug(question.forum_id)}/question/#{slug(question)}/toggle_favourite"
                            t-attf-class="o_wforum_favourite_toggle btn btn-lg #{ 'opacity-50' if not question.user_favourite else '' } opacity-hover-100 px-1">
                            <i t-attf-class="position-relative fa #{'o_wforum_gold fa-star ' if question.user_favourite else 'fa-star-o '}"
                                data-bs-toggle="tooltip"
                                data-bs-placement="top"
                                title="Favorite"/>
                        </a>
                        <t t-if="question.state == 'active'" t-call="website_forum.follow">
                            <t t-set="object" t-value="question"/>
                            <t t-set="icons_design" t-value="True"/>
                            <t t-set="btn_classes" t-value="'btn-lg opacity-50 opacity-100-hover' "/>
                        </t>
                    </div>
                </div>
            </div>
        </t>
        <t t-elif="_page_name == 'list_questions' or is_edit">
            <t t-set="target" t-value="post.parent_id if is_answer else post"/>
            <div class="o_wforum_breadcrumb_root_list_or_edit d-flex">
                <div t-if="breadcrumb_kind=='base'" class="col-10 col-lg flex-grow-1 fs-5">
                    <span t-if="not is_edit" class="fw-bold mb-0" t-out="_forum_name"/>
                    <span t-elif="not is_answer" class="fw-bold mb-0">Edit Question</span>
                    <span t-else="is_answer" class="fw-bold mb-0">Edit Answer</span>
                </div>
            </div>
        </t>
        <ol t-else="" class="breadcrumb col-10 col-lg flex-grow-1 flex-nowrap my-0 p-0 fs-5">
            <li class="o_wforum_breadcrumb_root breadcrumb-item text-nowrap text-truncatet">
                <a t-attf-href="/forum/#{ _forum_slug }" t-out="_forum_name"/>
            </li>
            <li t-if="queue_type" class="breadcrumb-item text-nowrap d-none d-lg-flex">
                <span>Moderation</span>
            </li>
            <li class="breadcrumb-item text-nowrap text-truncate">
                <span class="fw-bold text-truncate" t-out="_page_name_label"/>
            </li>
        </ol>
        <t t-if="_page_name != 'single_question'">
            <div class="d-lg-none text-end">
                <button class="btn position-relative ms-auto fs-5" data-bs-toggle="offcanvas" data-bs-target="#o_wforum_offcanvas">
                    <i class="fa fa-navicon"/>
                </button>
            </div>
            <div t-if="tag or tags or search or question_count or page_name or website_forum_action" t-attf-class="d-flex justify-content-lg-end gap-2 flex-grow-1 flex-wrap flex-md-nowrap w-100 w-lg-auto #{'mw-xl-75' if search else 'mw-xl-50'}">
                <span t-if="tag and not tags" class="btn btn-light rounded ps-2">
                    <span t-if="tag"><i class="fa fa-tag me-1 opacity-50"/><t t-out="tag.name"></t></span>
                    <a t-attf-href="#{ url_for('/forum') }/#{ _forum_slug }?#{ keep_query('search', 'sorting', 'my', 'create_uid') }"
                       class="p-1 text-decoration-none text-reset opacity-50"><i class="oi oi-close d-inline-block"/></a>
                </span>
                <span t-if="search" class="btn btn-light rounded ps-2">
                    <em t-if="search" class="text-muted">"<t t-out="search"/>"</em>
                    <a t-attf-href="?#{ keep_query('sorting', 'my') }" class="p-1 text-decoration-none text-reset opacity-50">
                        <i class="oi oi-close d-inline-block"/>
                    </a>
                </span>
                <div t-if="question_count or tags" class="dropdown ms-lg-auto">
                    <t t-if="_page_name == 'tags'" t-set="tag_filter" t-value="keep_query('filters').split('=')[1] if keep_query('filters') else ''"/>
                    <a href="#" class="btn btn-light dropdown-toggle" data-bs-toggle="dropdown">
                        <!-- Foreach tag_filter/ Foreach filters -->
                        <t t-if="filters == 'all' or tag_filter == 'all' or not tag_filter and not filters">All</t>
                        <t t-if="_page_name == 'tags'">
                            <t t-if="tag_filter == 'followed'">Followed Tags</t>
                            <t t-elif="tag_filter == 'unused'">Unused Tags</t>
                            <t t-elif="tag_filter == 'most_used'">Most used Tags</t>
                        </t>
                        <t t-else="">
                            <t t-if="filters == 'solved'">Solved</t>
                            <t t-elif="filters == 'unsolved'">Unsolved</t>
                            <t t-elif="filters == 'unanswered'">Unanswered</t>
                        </t>
                    </a>
                    <div class="dropdown-menu" role="menu">
                        <a t-attf-href="?#{ keep_query('search', 'sorting', 'create_uid', filters='all') }"
                            class="dropdown-item">
                            All
                        </a>
                        <div class="dropdown-divider"/>
                        <t t-if="_page_name == 'tags'">
                            <a t-if="not request.env.user._is_public()"
                                t-attf-href="?#{ keep_query('search', 'sorting', 'my', filters='followed') }"
                                class="dropdown-item">Followed Tags
                            </a>
                            <t t-if="forum and forum.can_moderate">
                                <a t-attf-href="?#{ keep_query('search', 'sorting', 'my', filters='unused') }"
                                    class="dropdown-item">Unused Tags
                                </a>
                            </t>
                            <a t-attf-href="?#{ keep_query('search', 'sorting', 'my', filters='most_used') }"
                                class="dropdown-item">Most Used Tags
                            </a>
                        </t>
                        <t t-else="">
                            <t t-if="not forum or (forum and forum.mode == 'questions')">
                                <a t-attf-href="?#{ keep_query('search', 'sorting', 'my', filters='solved') }"
                                    class="dropdown-item">Solved
                                </a>
                                <a t-attf-href="?#{ keep_query('search', 'sorting', 'my', filters='unsolved') }"
                                    class="dropdown-item">Unsolved
                                </a>
                            </t>
                            <a t-attf-href="?#{ keep_query('search', 'sorting', 'my', filters='unanswered') }"
                                class="dropdown-item">Unanswered
                            </a>
                        </t>
                    </div>
                </div>
                <!-- 'Search Box' -->
                <t t-if="question_count or tags" t-call="website.website_search_box_input">
                    <t t-if="_page_name == 'tags'">
                        <t t-set="search_type" t-value="'forum_tags_only'"/>
                        <t t-set="action" t-value="'/forum/%s/tag' % (_forum_slug)"/>
                    </t>
                    <t t-else="">
                        <t t-set="search_type" t-value="'forums'"/>
                        <t t-set="action" t-value="'/forum/%s' % (_forum_slug)"/>
                    </t>
                    <t t-set="display_description" t-valuef="true"/>
                    <t t-set="display_detail" t-valuef="true"/>
                    <t t-set="_form_classes" t-valuef="flex-grow-1"/>
                    <t t-set="_input_classes" t-valuef="border-0 bg-light"/>
                    <t t-set="_submit_classes" t-valuef="btn-light"/>
                    <input t-if="filters" type="hidden" name="filters" t-att-value="filters"/>
                    <input t-if="my" type="hidden" name="my" t-att-value="my"/>
                    <input t-if="sorting" type="hidden" name="sorting" t-att-value="sorting"/>
                </t>
                <t t-if="_page_name == 'list_questions'">
                    <t t-set="popover_title">You already have a pending post</t>
                    <t t-set="popover_content">Please wait for a moderator to validate your previous post before continuing.</t>
                    <div t-if="uid and forum and forum.has_pending_post"
                        data-bs-toggle="popover"
                        t-att-data-bs-title="popover_title"
                        t-att-data-bs-content="popover_content">
                        <a class="o_wforum_ask_btn disabled btn btn-primary w-100 w-md-auto mb-3 mb-md-0" t-attf-href="/forum/#{ slug(forum) }/ask">New Post</a>
                    </div>
                    <a t-elif="uid and forum" role="button" type="button" t-attf-class="o_wforum_ask_btn btn btn-primary w-100 w-md-auto #{ 'karma_required' if user.karma &lt; forum.karma_ask else '' }"
                        t-att-data-karma="forum.karma_ask" t-attf-href="/forum/#{slug(forum)}/ask">New Post</a>
                </t>
                <button t-if="website_forum_action and not queue_type == 'close' and posts_ids" type="button" class="o_wforum_btn_filter_tool btn btn-secondary"
                    data-bs-toggle="modal" data-bs-target="#markAllAsSpam">
                    <i class="fa fa-bug me-1"/>Filter Tool
                </button>
            </div>
        </t>
    </nav>
</template>

<template id="header" name="Forum Index">
    <t t-call="website_forum.layout">
        <t t-if="forum and not forum.active">
            <t t-set="head">
                <meta name="robots" content="noindex, nofollow" />
            </t>
            <div class="text-center text-muted">
                <p class="css_editable_hidden"><h2>This forum has been archived.</h2></p>
            </div>
        </t>
        <t t-else="">
            <div class="oe_structure" id="oe_structure_website_forum_header_1"/>
            <div id="wrap" t-attf-class="o_wforum_wrapper position-relative container row mx-auto px-0 #{ 'h-100' if forum_welcome_message else ''} #{website_forum_action}">
                <t t-set="_forum_slug" t-value="slug(forum) if forum else 'all'"/>
                <t t-set="_forum_path" t-value="url_for('/forum') + '/' + _forum_slug"/>
                <t t-call="website_forum.user_sidebar"/>
                <t t-call="website_forum.user_sidebar_mobile"/>
                <div class="o_wforum_content_wrapper col-lg-9">
                    <div class="o_wprofile_email_validation_container d-flex flex-column justify-content-center mb-3 mb-lg-5 pt-2 pt-lg-3">
                        <t t-call="website_profile.email_validation_banner">
                            <t t-set="redirect_url" t-value="'/forum/%s' % forum.id if forum else '/forum/all/'"/>
                            <t t-set="additional_validation_email_message"> and join this Forum</t>
                            <t t-set="additional_validated_email_message"> You may now participate in our forums.</t>
                        </t>
                        <t t-call="website_forum.forum_model_nav"/>
                        <t t-out="0"/>
                    </div>
                </div>
            </div>
        </t>
    </t>
</template>

<!-- User sidebar -->
<template id="user_sidebar">
    <aside class="o_wforum_sidebar col-3 d-none d-lg-flex flex-column z-index-1">
        <div class="flex-grow-1 px-2">
            <t t-call="website_forum.user_sidebar_header"/>
            <t t-call="website_forum.user_sidebar_body"/>
        </div>
        <div t-if="forum" class="o_wforum_sidebar_footer mt-3 px-3 pb-2 text-center">
            <a class="btn btn-sm btn-link" t-attf-href="/forum/#{slug(forum)}/faq">
                <i class="fa fa-info-circle fa-fw"/> About this forum
            </a>
        </div>
    </aside>
</template>

<!-- Off canvas user sidebar on mobile -->
<template id="user_sidebar_mobile">
    <div id="o_wforum_offcanvas" class="o_website_offcanvas offcanvas offcanvas-end d-lg-none mw-75 p-0 overflow-visible">
        <button type="button" class="btn-close mt-3 ms-auto me-3" data-bs-dismiss="offcanvas" aria-label="Close"/>
        <div class="offcanvas-header align-items-start px-0" t-call="website_forum.user_sidebar_header"/>
        <div class="offcanvas-body d-flex flex-column ps-0 py-0">
            <t t-call="website_forum.user_sidebar_body"/>
            <div t-if="forum" class="mb-2 d-flex justify-content-center align-items-end flex-grow-1">
                <a class="btn btn-sm btn-link" t-att-href="_forum_path + '/faq'">
                    <i class="fa fa-info-circle fa-fw"/> About this forum
                </a>
            </div>
        </div>
    </div>
</template>

<!-- User sidebar elements -->
<template id="user_sidebar_header">
    <t t-set="_location" t-value="( ('/tag/' + slug(tag) + '/questions?') if tag else '?' )"/>

    <div t-if="not uid" class="o_wforum_sidebar_section mt-4 text-center">
        <div class="alert alert-info mb-2"><span>You need to be registered to interact with the community.</span>
            <a t-if="is_public_user and forum_welcome_message" href="/web/login" class="btn btn-primary mt-2">Sign up</a>
        </div>
    </div>
    <div t-if="uid" class="o_wforum_sidebar_section mt-4 mb-2">
        <a t-attf-href="/forum/user/#{ uid }?forum_id=#{ forum.id if forum else '' }&amp;forum_origin=#{ request.httprequest.path }"
            class="btn w-100 py-1 text-start d-flex align-items-center" data-bs-toggle="tooltip" data-trigger="hover"
            title="My profile">
            <img class="o_wforum_avatar rounded-circle o_object_fit_cover" t-att-src="request.website.image_url(user, 'avatar_128', '60x60')" alt="Avatar"/>
            <div class="d-flex flex-column justify-content-center ms-2">
                <h6 class="mt-0 mb-1" t-out="user_id.name"/>
                <span class="fs-6 text-reset opacity-50"><t t-out="user_id.karma"/> XP</span>
            </div>
        </a>
    </div>
</template>

<template id="user_sidebar_body">
    <div class="o_wforum_sidebar_section">
        <t t-set="location" t-valuef="#{ _forum_path }#{ ('/tag/' + slug(tag) + '/questions') if (tag and not tags) else '' }?"/>
        <!-- All Posts -->
        <a t-attf-class="nav-link my-1 py-1 #{ 'rounded text-bg-light disabled' if request.httprequest.path ==  _forum_path and no_filters and not any([my, queue_type, tags]) else 'text-reset' }" t-att-href="location">
            <i t-attf-class="fa fa-list fa-fw #{ 'opacity-50' if request.httprequest.path != _forum_path or my or queue_type else '' }"/> Posts
        </a>
        <t t-if="uid">
            <!-- My Posts -->
            <a t-attf-class="nav-link my-1 py-1 #{ 'rounded text-bg-light disabled' if my == 'mine' else 'text-reset' }" t-att-href="location + keep_query('search', 'filters', 'sorting', my='mine')">
                <i t-attf-class="fa fa-user-circle fa-fw #{ 'opacity-50' if my != 'mine' else ''}"/> My Posts
            </a>

            <!-- My Favourites -->
            <a t-attf-class="nav-link my-1 py-1 #{ 'rounded text-bg-light disabled' if my == 'favourites' else 'text-reset' }" t-att-href="location + keep_query( 'search', 'filters', 'sorting', my='favourites')">
                <i t-attf-class="fa fa-star fa-fw #{ 'opacity-50' if my != 'favourites' else ''}"/> Favourites
            </a>
        </t>
        <!-- People -->
        <a t-attf-class="nav-link my-1 py-1 text-reset" t-attf-href="/profile/users?forum_origin=#{request.httprequest.path}">
            <i class="fa fa-users fa-fw opacity-50"/> People
        </a>

        <!-- Badges -->
        <a t-attf-class="nav-link my-1 py-1 text-reset" t-attf-href="/profile/ranks_badges?forum_origin=#{request.httprequest.path}">
            <i class="fa fa-shield fa-fw opacity-50"/> Badges
        </a>
    </div>
    <div t-if="forum and user.karma>=forum.karma_moderate" class="o_wforum_sidebar_section pt-3">
        <!-- Moderation Tools -->
        <div class="px-3 pb-1 fw-bold">Moderation tools</div>
        <a t-attf-class="nav-link my-1 py-1 #{ 'rounded text-bg-light disabled' if queue_type == 'validation' else 'text-reset'}" t-attf-href="/forum/#{_forum_slug}/validation_queue">
            <i t-attf-class="fa fa-check-square-o fa-fw #{ 'opacity-50' if queue_type != 'validation' else ''}"/> To Validate
            <span id="count_posts_queue_validation" t-attf-class="badge #{ 'text-bg-warning' if forum.count_posts_waiting_validation > 0 else 'd-none'}" t-out="forum.count_posts_waiting_validation"/>
        </a>
        <a t-attf-class="nav-link my-1 py-1 #{'rounded text-bg-light disabled' if queue_type == 'offensive' or queue_type == 'flagged' else 'text-reset'}" t-attf-href="/forum/#{_forum_slug}/flagged_queue">
            <i t-attf-class="fa fa-flag fa-fw #{ 'opacity-50' if queue_type != 'flagged' else ''}"/> Flagged
            <span id="count_posts_queue_flagged" t-attf-class="badge #{ 'text-bg-danger' if forum.count_flagged_posts > 0 else 'd-none'}" t-out="forum.count_flagged_posts"/>
        </a>
        <a t-attf-class="nav-link my-1 py-1 #{ 'rounded text-bg-light disabled' if queue_type == 'close' else 'text-reset'}" t-attf-href="/forum/#{_forum_slug}/closed_posts">
            <i t-attf-class="fa fa-window-close fa-fw #{ 'opacity-50' if queue_type != 'close' else '' }"/> Closed
        </a>
    </div>
    <div t-if="forum and forum.tag_most_used_ids" class="o_wforum_sidebar_section pt-3">
        <div class="d-flex align-items-center px-3 pb-1 fw-bold">Tags
            <a class="ms-auto px-0 fw-normal" t-att-href="_forum_path + '/tag'">
                <small>View all</small>
            </a>
        </div>
        <a t-foreach="forum.tag_most_used_ids" t-as="tag"
            t-attf-href="#{ _forum_path }/tag/#{ slug(tag) }/questions?#{ keep_query( 'search', 'sorting', 'my', 'filters') }"
            t-attf-class="nav-link my-1 py-1 text-reset">
            <i class="fa fa-tag fa-fw small opacity-50"/>
            <t t-out="tag.name"/>
        </a>
    </div>
    <div t-if="my_other_forums" class="o_wforum_sidebar_section pt-3">
        <div class="px-3 pb-1 fw-bold">My forums</div>
        <t t-foreach="my_other_forums.sorted(lambda f: f.name.casefold())" t-as="my_forum">
            <a class="nav-link my-1 py-1 text-reset" t-attf-href="/forum/#{slug(my_forum)}">
                <i class="fa fa-file-o fa-fw opacity-50"/>
                <t t-out="my_forum.name"/>
            </a>
        </t>
    </div>
</template>

<template id="header_welcome_message" inherit_id="website_forum.header" name="Forum Welcome Message (oe_structure_forum_top)">
    <xpath expr="//*[hasclass('oe_structure')][@id='oe_structure_website_forum_header_1']" position="replace">
        <div t-if="forum" class="oe_structure oe_empty" id="oe_structure_website_forum_header_1">
            <section t-if="editable or (is_public_user and not forum_welcome_message)" t-attf-class="s_cover parallax s_parallax_is_fixed bg-black-50 pt48 pb48 #{'css_non_editable_mode_hidden' if editable else 'forum_intro'}" data-scroll-background-ratio="1" data-snippet="s_cover">
                <span t-if="forum.image_1920" class="s_parallax_bg oe_img_bg" t-attf-style="background-image: url('#{request.website.image_url(forum, 'image_1920')}'); background-position: center;"/>
                <div t-if="forum.image_1920" class="o_we_bg_filter bg-black-50"/>
                <div class="container s_allow_columns">
                    <div class="row" data-row-count="5">
                        <div class="o_colored_level offset-lg-3 col-lg-6">
                            <div t-field="forum.welcome_message" class="container s_allow_columns"/>
                        </div>
                    </div>
                </div>
            </section>
        </div>
    </xpath>
</template>

<template id="no_results_message">
    <t t-if="queue_type">
        <t t-set="_no_results_title">You've Completely Caught&amp;nbsp;Up!</t>
        <t t-set="result_msg">Go enjoy a cup of coffee.</t>
        <t t-set="record_name_plural">posts</t>
        <t t-set="go_back_url" t-valuef="/forum/#{ slug(forum) }/"/>
    </t>
    <t t-else="" t-set="_no_results_title">Oops!</t>
    <div t-attf-class="#{ 'o_caught_up_alert' if queue_type else '' } row g-0 justify-content-center #{ 'd-none' if hide_alert else '' }">
        <div class="p-3 d-flex flex-column align-items-center">
            <img t-if="queue_type" src="/website_forum/static/src/img/tasks.svg" class="img img-fluid mb-3" width="200" alt="Animation of a pen checking a checkbox"/>
            <img t-else="" src="/website_forum/static/src/img/empty.svg" class="img img-fluid mb-3" width="150" alt="Empty box"/>
            <h5 t-out="_no_results_title"/>
            <span t-if="result_msg" t-out="result_msg"/>
            <span t-else="">
                <t t-if="filters == 'unanswered'">
                    Sorry, we could not find any <b>unanswered</b> results
                </t>
                <t t-elif="filters == 'solved'">
                    Sorry, we could not find any <b>solved</b> results
                </t>
                <t t-elif="filters == 'unsolved'">
                    Sorry, we could not find any <b>unsolved</b> results
                </t>
                <t t-else="">
                    Sorry, we could not find any results
                </t>
                <b t-if="my == 'favourites'"> in your favourites</b>
                <b t-elif="my == 'mine'"> in your posts</b>
                <span t-if="search">matching "<em class="fw-bold text-break" t-out="original_search or search"/>"</span>
                <span t-out="'and' if search and tag else ''"/>
                <t t-if="tag">
                    using the <span class="px-2 py-1 rounded bg-300" t-out="tag.name"/> tag
                </t>.
            </span>
            <span t-if="original_search">Showing results for <em class="fw-bold" t-out="search"/> instead.</span>
            <div t-if="question_count == 0 and (not forum or forum.total_posts != 0)" class="mt-3 text-start">
                <p><i class="fa fa-check fa-fw me-1"/>Check your spelling and try again.</p>
                <p><i class="fa fa-check fa-fw me-1"/>Try searching for one or two words.</p>
                <p><i class="fa fa-check fa-fw me-1"/>Be less specific in your wording for a wider search result.</p>
            </div>
            <span t-elif="forum and forum.total_posts == 0">Because there are no posts in this forum yet.</span>
            <div class="mt-3">
                <a t-if="uid and forum and forum.total_posts == 0" role="button" type="button" class="o_forum_ask_btn btn btn-primary ms-2" t-attf-href="/forum/#{_forum_slug}/ask">Start by creating a post</a>
                <a t-else="" role="button" type="button" class="btn btn-primary mt-2" t-att-href="go_back_url">Go back to the list of <t t-out="record_name_plural"/></a>
            </div>
        </div>
    </div>
</template>

<!-- ERROR MANAGEMENT -->
<!-- ============================================================ -->

<template id="404">
    <t t-call="website_forum.header">
        <div class="oe_structure oe_empty"/>
        <h1 class="mt-4">Question not found!</h1>
        <p>Sorry, this question is not available anymore.</p>
        <p>
            <a class="btn-link" t-attf-href="/forum"><i class="oi oi-arrow-right display-inline-block"/> Return to questions list</a>
        </p>
    </t>
</template>

    </data>
</odoo>

```

## File: views\forum_forum_templates_moderation.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

<template id="moderation_display_post_question_block">
    <t t-call="website_forum.display_post_question_block">
        <t t-set="post_content">
            <div class="clearfix">
                <span t-field="question.content" class="oe_no_empty"/>
            </div>
        </t>
    </t>
</template>

<!-- Moderation: close a post -->
<template id="mark_as_offensive">
    <div class="o_mark_offensive">
        <div class="alert bg-light text-muted" t-if="not offensive">
            If you close this post, it will be hidden for most users. Only
            users having a high karma can see closed posts to moderate
            them.
        </div>
        <div class="alert bg-light text-muted" t-if="offensive">
            If you mark this post as offensive, it will be hidden for most users. Only
            users having a high karma can see offensive posts to moderate
            them.
        </div>
        <form t-attf-action="/forum/#{ slug(forum) }/#{'post' if offensive else 'question'}/#{slug(question)}/#{'mark_as_offensive' if offensive else 'close'}" method="post" role="form" class="js_website_submit_form">
            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
            <input name="post_id" t-att-value="question.id" type="hidden"/>
            <div class="row g-0 mb-3">
                <label class="form-label col-lg-2" for="post">Post:</label>
                <input type="text" disabled="True" class="form-control-plaintext col" name="post" t-att-value="question.name if not question.parent_id else question.parent_id.name"/>
            </div>
            <div class="row g-0 mb-4">
                <label class="form-label col-lg-2" for="reason"><t t-if="offensive">Offensive</t><t t-if="not offensive">Closing</t> Reason:</label>
                <select class="form-select form-select col" name="reason_id">
                    <t t-foreach="reasons or []" t-as="reason">
                        <option t-att-value="reason.id" t-att-selected="reason.id == question.closed_reason_id.id"><t t-out="reason.name"/></option>
                    </t>
                </select>
            </div>
            <div class="row g-0 mb-3">
                <div class="col offset-lg-2">
                    <button type="submit" class="btn btn-danger">
                        <t t-if="offensive">Mark as offensive</t>
                        <t t-if="not offensive">Close post</t>
                    </button>
                    <a role="button" class="btn btn-link ms-2" t-attf-href="/forum/#{ slug(forum) }/#{ slug(question) }">Discard</a>
                </div>
            </div>
        </form>
    </div>
</template>

<template id="close_post" name="Close Post">
    <t t-call="website_forum.header">
        <t t-call="website_forum.mark_as_offensive"/>
    </t>
</template>

<template id="moderation_queue" name="Forum Moderation Queue">
    <t t-set="_page_name" t-value="queue_type"/>
    <t t-set="_page_name_label" t-if="queue_type == 'validation'">To Validate</t>
    <t t-set="_page_name_label" t-elif="queue_type == 'flagged'">Flagged</t>
    <t t-set="_page_name_label" t-elif="queue_type == 'offensive'">Offensive Posts</t>
    <t t-set="_page_name_label" t-elif="queue_type == 'close'">Closed Posts</t>
    <t t-set="website_forum_action" t-valuef="o_wforum_moderation_queue"/>

    <t t-call="website_forum.header">
        <t t-call="website_forum.no_results_message"><t t-set="hide_alert" t-value="len(posts_ids) > 0"/></t>
        <div id="queue_type" t-att-data-queue-type="queue_type"/>

        <div class="modal fade" t-att-data-spam-ids="str(posts_ids.ids)" id="markAllAsSpam" tabindex="-1" role="dialog" aria-labelledby="markAllAsSpam" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header d-flex align-items-center pb-0">
                        <div class="me-2 text-muted">Filter by:</div>
                        <ul class="nav nav-tabs border-bottom-0" id="myTab" role="tablist">
                            <li class="nav-item">
                                <a class="nav-link active spam_menu" id="user-tab" data-bs-toggle="tab" href="#spam_user" role="tab" aria-controls="user" aria-selected="true"><i class="fa fa-user"/> User</a>
                            </li>
                            <li class="nav-item">
                                <a class="nav-link spam_menu" id="country-tab" data-bs-toggle="tab" href="#spam_country" role="tab" aria-controls="spam_country" aria-selected="false"><i class="fa fa-flag"/> Country</a>
                            </li>
                            <li class="nav-item">
                                <a class="nav-link spam_menu" id="character-tab" data-bs-toggle="tab" href="#spam_character" role="tab" aria-controls="spam_character" aria-selected="false"><i class="fa fa-font"/> Text</a>
                            </li>
                        </ul>
                        <button type="button" class="btn-close align-self-start" data-bs-dismiss="modal"></button>
                    </div>
                    <div class="modal-body bg-100">
                        <div class="tab-content" id="o_tab_content_spam">
                            <div class="tab-pane fade show active" data-key="create_uid" id="spam_user" role="tabpanel" aria-labelledby="user-tab">
                                <form class="row" >
                                    <div t-foreach="posts_ids.mapped('create_uid')" t-as="create_user" class="col-6">
                                        <div class="card mb-2">
                                            <div class="card-body py-2">
                                                <div class="form-check">
                                                    <input type="checkbox" t-att-value="create_user.id" class="form-check-input" t-attf-id="user_#{ create_user.id }"/>
                                                    <label class="form-check-label" t-attf-for="user_#{ create_user.id }">
                                                        <img class="d-inline img o_wforum_avatar" t-att-src="request.website.image_url(create_user, 'avatar_128', '40x40')" alt="Avatar"/>
                                                        <span t-out="create_user.name" class="d-inline"/>
                                                    </label>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </form>
                            </div>
                            <div class="tab-pane fade" data-key="country_id" id="spam_country" role="tabpanel" aria-labelledby="country-tab">
                                <form class="row">
                                    <div t-foreach="posts_ids.mapped('create_uid.country_id')" t-as="country" class="col-6">
                                        <div class="card mb-2">
                                            <div class="card-body py-2">
                                                <div class="form-check">
                                                    <input type="checkbox" class="form-check-input" t-attf-id="country_#{country.id}" t-att-value="country.id"/>
                                                    <label class="form-check-label" t-attf-for="country_#{country.id}">
                                                        <span t-field="country.image_url" t-options='{"widget": "image_url", "class": "country_flag"}' class="me-2"/>
                                                        <span t-out="country.name"/>
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
            <div class="card-header">
                <div class="row align-items-baseline">
                    <h6 class="col-md-2 mb-0">
                        <b t-if="question.parent_id">Answer:</b>
                        <b t-else="">Question:</b>
                    </h6>
                    <div class="col d-flex align-items-center justify-content-between">
                        <a t-attf-href="/forum/#{slug(question.forum_id)}/#{slug(question)}#{ ('/#answer-%s' % answer.id) if answer else '' }"
                            t-attf-title="Read: #{question.name}"
                            t-attf-class="text-reset"
                            t-out="question.name"/>
                    </div>
                    <div class="col-lg-3 d-lg-flex align-items-baseline">
                        <span t-if="not question.active and question.state=='offensive'" class="badge bg-warning text-wrap flex-grow-1">Offensive</span>
                        <span t-if="not question.active and question.state=='offensive' and question.closed_reason_id" class="badge bg-warning text-wrap flex-grow-1" t-out="question.closed_reason_id.name.capitalize()"/>
                        <span t-if="question.closed_reason_id" class="badge bg-danger text-wrap flex-grow-1" t-out="question.closed_reason_id.name.capitalize()"/>
                    </div>
                </div>
            </div>
            <div class="card-body">
                <div class="row">
                    <t t-if="question.parent_id">
                        <div t-foreach="question" t-as="answer" class="d-flex flex-column col-md-9 col-sm-8 mb-2">
                            <span t-field="answer.content" class="oe_no_empty flex-grow-1 mb-2"/>
                            <div class="mt-auto">
                                <t t-call="website_forum.author_box">
                                    <t t-set="object" t-value="question"/>
                                    <t t-set="_image_classes" t-value="'rounded-circle'"/>
                                    <t t-set="allow_biography" t-value="True"/>
                                    <t t-set="show_image" t-value="True"/>
                                    <t t-set="show_name" t-value="True"/>
                                    <t t-set="compact" t-value="True"/>
                                </t>
                                <small class="text-muted">
                                    <i class="fa fa-calendar ms-4 me-1"/><span t-field="question.write_date" t-options='{"format":"short"}'/>
                                </small>
                            </div>
                        </div>
                    </t>
                    <div t-if="not question.parent_id" class="d-flex flex-column col">
                        <span t-field="question.content" class="oe_no_empty flex-grow-1"/>
                        <div class="mt-auto">
                            <t t-call="website_forum.author_box">
                                <t t-set="object" t-value="question"/>
                                <t t-set="_image_classes" t-value="'rounded-circle'"/>
                                <t t-set="allow_biography" t-value="True"/>
                                <t t-set="show_image" t-value="True"/>
                                <t t-set="show_name" t-value="True"/>
                                <t t-set="compact" t-value="True"/>
                            </t>
                            <small class="text-muted">
                                <i class="fa fa-calendar ms-4 me-1"/><span t-field="question.write_date" t-options='{"format":"short"}'/>
                            </small>
                        </div>
                    </div>
                    <t t-set="post_url" t-valuef="/forum/#{ slug(forum) }/post/#{ slug(question) }"/>
                    <div class="col-sm-3 o_wforum_validation_queue">
                        <div class="text-center d-flex flex-row gap-2 flex-lg-column align-items-stretch mt-2 mt-lg-0">
                            <t t-if="queue_type == 'close'" t-call="website_forum.link_button">
                                <t t-set="url" t-valuef="/forum/#{ slug(forum) }/question/#{ slug(question) }/reopen"/>
                                <t t-set="label">Reopen</t>
                                <t t-set="icon" t-value="'fa fa-folder-open'"/>
                                <t t-set="karma" t-value="question.karma_close if not question.can_close else 0"/>
                                <t t-set="form_classes" t-value="'d-flex'"/>
                                <t t-set="classes" t-value="'btn-outline-primary flex-grow-1'"/>
                            </t>
                            <t t-if="queue_type == 'close'" t-call="website_forum.link_button">
                                <t t-set="url" t-valuef="#{ post_url }/delete"/>
                                <t t-set="label">Delete</t>
                                <t t-set="icon" t-value="'fa fa-trash'"/>
                                <t t-set="karma" t-value="question.karma_unlink if not question.can_unlink else 0"/>
                                <t t-set="form_classes" t-value="'d-flex'"/>
                                <t t-set="classes" t-value="'btn-outline-danger flex-grow-1'"/>
                            </t>

                            <a t-else="" t-attf-href="#{ post_url }/validate" title="Validate" aria-label="Validate" class="btn btn-outline-success flex-grow-1"><i class="fa fa-check"/><span class="ms-2">Accept</span></a>
                            <a t-if="queue_type == 'validation'" t-attf-href="#{ post_url }/refuse" title="Refuse" aria-label="Refuse" class="btn btn-outline-danger flex-grow-1"><i class="fa fa-times"/><span class="ms-2">Reject</span></a>
                            <a t-if="queue_type == 'flagged'" t-attf-href="#{ post_url }/ask_for_mark_as_offensive" aria-label="Mark as offensive" title="Mark as offensive" class="btn btn-outline-danger flex-grow-1"><i class="fa fa-times"/><span class="ms-2">Offensive</span></a>
                            <a t-if="queue_type == 'offensive'" href="#" disabled="disabled" aria-label="Offensive" title="Offensive" class="btn btn-outline-danger flex-grow-1"><i class="fa fa-times"/><span class="ms-2">Offensive</span></a>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </t>
</template>
    </data>
</odoo>

```

## File: views\forum_forum_templates_post.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

<!-- Display a post -->
<template id="display_post_question_block">
    <div class="o_wforum_index_entry_title">
        <div class="d-flex align-items-baseline gap-1">
            <small t-if="question.user_favourite and not (my == 'favourites' or hide_fav_icon)"
                title="Your favourite"
                aria-label="Your favourite"
                data-bs-toggle="tooltip"
                class="fa fa-star o_wforum_gold"/>
            <small t-if="question.message_is_follower and not (my == 'followed' or hide_fav_icon)"
                title="You're following this post"
                aria-label="You're following this post"
                data-bs-toggle="tooltip"
                class="fa fa-bell"/>
            <a t-attf-href="/forum/#{slug(question.forum_id)}/#{slug(question)}#{ ('/#answer-%s' % answer.id) if answer else '' }"
                t-attf-title="Read: #{question.name}"
                t-attf-class="stretched-link text-body #{_link_classes}">
                <span t-out="question.name" t-if="question.name"/>
            </a>
            <span t-if="question.has_validated_answer and question.forum_id.mode == 'questions'"
                  title="Solved"
                  aria-label="Solved"
                  class="badge bg-success">
                    <i class="fa fa-check me-1"/>Solved
            </span>
        </div>
    </div>

    <div t-if="not is_profile" t-attf-class="o_wforum_index_entry_tags mb-lg-1">
        <t if="tag.posts_count and not hide_tags" t-foreach="question.tag_ids" t-as="question_tag">

            <!-- Toggle Tags on click -->
            <t t-if="tag and tag.name == question_tag.name" t-set="click_action"
                t-value="'/forum/' + slug(question_tag.forum_id) + '?' + keep_query( 'search', 'sorting', 'my', 'create_uid',)"/>
            <t t-else="" t-set="click_action"
                t-value="'/forum/' + slug(question_tag.forum_id) + '/tag/' + slug(question_tag) + '/questions?' + keep_query( 'search', 'sorting', 'my', 'create_uid', filters='tag')"/>

            <a t-att-href="click_action"
                t-attf-class="badge position-relative z-index-1 #{ 'text-bg-primary' if tag and tag.name == question_tag.name else 'text-bg-secondary' } fw-normal"
                t-field="question_tag.name"/>
        </t>
    </div>

    <!--  Display post's content in moderation mode-->
    <div><t t-out="post_content"/></div>
</template>

<template id="display_post">
    <div t-if="is_profile" class="card">
        <div class="card-body">
            <t t-call="website_forum.display_post_question_block"/>
            <div class="d-inline-flex gap-2 position-relative z-index-1 me-1">
                <t t-if="answer" t-call="website_forum.author_box">
                    <t t-set="object" t-value="question"/>
                    <t t-set="_image_classes" t-value="'rounded-circle'"/>
                    <t t-set="allow_biography" t-value="True"/>
                    <t t-set="show_image" t-value="True"/>
                    <t t-set="show_name" t-value="True"/>
                    <t t-set="compact" t-value="True"/>
                </t>
                <small class="text-muted">
                    <i class="fa fa-calendar me-1"/><span t-field="question.write_date" t-options='{"format":"short"}'/>
                </small>
            </div>
            <div t-if="not answer" t-call="website_forum.post_stats" class="d-inline-flex align-items-center"/>
            <div t-if="answer" class="d-inline-flex gap-1 small">Answered on<span t-field="answer.write_date" t-options='{"format": "d MMM yy "}'></span></div>
        </div>
    </div>
    <tr t-else="" class="position-relative d-flex d-lg-table-row">
        <td class="o_wforum_post_title flex-grow-1 order-2 order-lg-1">
            <t t-call="website_forum.display_post_question_block">
                <t t-set="_link_classes" t-value="'text-decoration-none'"/>
            </t>
        </td>
        <td class="o_wforum_posters order-1 order-lg-2">
            <div class="position-relative z-index-1 d-flex flex-row-reverse">
                <div class="d-none d-lg-flex flex-row-reverse" t-foreach="question.child_ids.filtered(lambda a: a.create_uid != question.create_uid)[-4:]" t-as="post_answer">
                    <t t-call="website_forum.author_box">
                        <t t-set="object" t-value="post_answer"/>
                        <t t-if="post_answer.is_correct" t-set="_box_classes" t-valuef="z-index-1 order-1"/>
                        <t t-set="_image_classes" t-value="'me-n2'"/>
                        <t t-set="show_image" t-value="True"/>
                        <t t-set="allow_biography" t-value="True"/>
                    </t>
                </div>
                <t t-call="website_forum.author_box">
                    <t t-set="object" t-value="question"/>
                    <t t-set="_box_classes" t-value="'z-index-1'"/>
                    <t t-set="_image_classes" t-value="'me-lg-n2'"/>
                    <t t-set="show_image" t-value="True"/>
                    <t t-set="allow_biography" t-value="True"/>
                </t>
            </div>
        </td>
        <td class="o_wforum_reply_count order-3 d-flex d-lg-table-cell flex-column text-end text-lg-center">
            <a t-if="question.child_count" class="text-reset" t-attf-href="/forum/#{ slug(question.forum_id) }/#{ slug(question) }">
                <i class="fa fa-reply me-1 d-lg-none" />
                <t t-out="question.child_count"/>
            </a>
            <span t-else="" class="opacity-50">
                <i class="fa fa-reply me-1 d-lg-none" />
                0
            </span>
            <div class="d-lg-none flex-grow-1"/>
            <div t-field="question.write_date" t-options='{"format": "MMM yy "}' class="d-lg-none small text-muted text-nowrap"/>
        </td>
        <td class="o_wforum_view_count d-none d-lg-table-cell text-center">
            <span t-out="question.views" t-attf-class="#{ 'opacity-50' if question.views == 0 else '' }" />
        </td>
        <td class="o_wforum_last_activity d-none d-lg-table-cell order-4 text-center" t-att-data-last-activity="question.last_activity_date">
            <span class="o_wforum_relative_datetime position-relative z-index-1" t-att-title="question.last_activity_date"/>
        </td>
    </tr>
</template>

<!-- Display a post as an answer -->
<template id="display_post_answer">
    <t t-set="question" t-value="answer"/>
    <t t-call="website_forum.display_post"/>
</template>

<!-- Edition: post an answer -->
<template id="post_answer">
    <div class="d-flex align-items-center mb-3">
        <div class="d-flex align-items-center">
            <img t-if="uid" t-attf-class="o_wforum_avatar me-2 rounded-circle" t-att-src="request.website.image_url(user, 'avatar_128', '24x24')" alt="Avatar"/>
            <h4 class="my-0">Your Answer</h4>
        </div>
        <button class="o_wforum_expand_toggle btn fa fa-expand ms-auto" data-bs-toggle="collapse"/>
    </div>
    <t t-if="request.params.get('nocontent')">
        <p class="alert alert-danger" role="alert">You cannot post an empty answer</p>
    </t>
    <form t-attf-action="/forum/#{ slug(forum) }/#{slug(question)}/reply" method="post" class="js_website_submit_form js_wforum_submit_form d-flex flex-column" role="form">
        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
        <input type="hidden" name="karma" t-att-value="user.karma" id="karma"/>
        <textarea name="content" t-attf-id="content-#{str(question.id)}" class="form-control o_wysiwyg_loader" required="required" minlength="50" t-att-data-karma="forum.karma_editor"/>
        <p class="mt-2 mb-1 small">
            <b>Please try to give a substantial answer.</b> If you wanted to comment on the question or answer, just
            <b>use the commenting tool.</b> Please remember that you can always <b>revise your answers</b>
            - no need to answer the same question twice. Also, please <b>don't forget to vote</b>
            - it really helps to select the best questions and answers!
        </p>
        <div>
            <button type="submit" t-attf-class="o_wforum_submit_post #{ 'oe_social_share_call ' if forum.allow_share else '' }btn btn-primary my-3 #{ 'karma_required' if not question.can_answer else ''}"
                    t-att-data-karma="question.forum_id.karma_answer"
                    data-social-target-type="answer" data-hashtags="#answer">Post Answer</button>
            <a href="#"
               class="o_wforum_discard_btn btn btn-link"
               data-bs-toggle="collapse"
               data-bs-target=".answer_collapse"
               aria-expanded="false">Discard</a>
        </div>
    </form>
</template>

<template id="post_stats">
    <div class="d-flex">
        <div>
            <span t-if="question.child_count" class="small">
                <t t-out="question.child_count or 0"/>
                <span class="text-muted">
                    <t t-if="question.child_count == 1">Reply</t>
                    <t t-else="">Replies</t>
                </span>
            </span>
        </div>
        <div class="ms-3">
            <span t-if="question.views" class="small">
                <t t-out="question.views or 0"/>
                <span class="text-muted">
                    <t t-if="question.views == 1">View</t>
                    <t t-else="">Views</t>
                </span>
            </span>
        </div>
    </div>
    <div t-if="question.tag_ids" class="o_wforum_index_entry_tags ms-3">
        <a t-foreach="question.tag_ids" t-as="question_tag"
            t-attf-href="/forum/#{slug(question_tag.forum_id)}/tag/#{slug(question_tag)}/questions?#{keep_query(filters='tag')}"
            t-attf-class="badge text-bg-secondary #{'ms-1' if not question_tag_first else ''} fw-normal"
            t-field="question_tag.name"/>
    </div>
</template>

<!-- Specific Post Layout -->
<template id="post_description_full" name="Question Navigation">
    <t t-set="_page_name" t-valuef="single_question"/>
    <t t-set="_question_creator" t-value="question.create_uid.id"/>

    <t t-call="website_forum.header">
        <div class="mw-xl-75 mw-xxl-100">
            <!-- Post Status Messages -->
            <div t-if="forum and question.state == 'pending' and user.karma >= forum.karma_moderate and question.active" class="alert alert-info text-center" role="status" >
                <p>This post is currently awaiting moderation and is not published yet.<br/>
                    As a moderator you can either <b>Accept</b> or <b>Reject</b> this post.</p>
                <div>
                    <a role="button" t-attf-href="/forum/#{slug(forum)}/post/#{slug(question)}/validate" type="button" class="btn btn-success">
                        <i class="fa fa-check fa-fw me-1"/>Accept</a>
                    <a role="button" t-attf-href="/forum/#{slug(forum)}/post/#{slug(question)}/refuse" type="button" class="btn btn-danger">
                        <i class="fa fa-times fa-fw me-1"/>Reject</a>
                </div>
            </div>
            <div t-if="question.state == 'pending' and user.karma &lt; forum.karma_moderate" class="alert alert-warning mb-0 text-center" role="status" >
                This post is awaiting validation
            </div>
            <div t-if="question.state == 'active' and question.can_answer and forum.has_pending_post" class="alert o_cc2 text-center">
                <h5 >You have a pending post</h5>
                <span>Please wait for a moderator to validate your previous post to be allowed to reply to questions.</span>
            </div>
            <div t-attf-class="alert alert-danger o_wforum_flag_alert #{'d-none ' if question.state != 'flagged' else ''}text-center" >
                <h5>This question has been flagged</h5>
                <span t-if="question.can_moderate">As a moderator, you can either validate or reject this answer.</span>
                <t t-call="website_forum.link_button">
                    <t t-set="url" t-value="'/forum/' + slug(forum) + '/post/' + slug(question) + '/flag'"/>
                    <t t-set="form_method" t-value="'GET'"/>
                    <t t-set="karma" t-value="question.forum_id.karma_flag if not question.can_flag else 0"/>
                    <t t-set="classes" t-valuef="o_wforum_flag"/>
                    <t t-set="flag_validator" t-value="question.can_moderate"/>
                    <t t-set="object" t-value="question"/>
                </t>
            </div>
            <div t-if="question.state == 'close'" role="status" class="alert alert-info text-center" >
                    The question has been closed<t t-if="question.closed_reason_id"> for reason: <b><i t-out="question.closed_reason_id.name"/></b></t>
                    <t t-if="question.closed_uid">
                        <br/>by
                        <a t-attf-href="/forum/#{ slug(forum) }/user/#{ question.closed_uid.id }"
                           t-out="question.closed_uid.name"/>
                    </t>
                    on <span t-field="question.closed_date"/>
                <div t-if="user.karma > question.karma_close" class="mt-3 text-center">
                    <t t-call="website_forum.link_button">
                        <t t-set="url" t-value="'/forum/' + slug(forum) + '/question/' + slug(question) + '/reopen'"/>
                        <t t-set="label">Reopen</t>
                        <t t-set="icon" t-value="'fa-folder-open'"/>
                        <t t-set="karma" t-value="question.karma_close if not question.can_close else 0"/>
                        <t t-set="classes" t-value="'btn-info'"/>
                    </t>
                </div>
            </div>
            <div t-attf-class="d-flex align-items-center #{'mb-3' if not is_profile else ''}" t-call="website_forum.post_stats"/>
            <div class="d-grid">
                <!-- Question Post -->
                <t t-call="website_forum.post_display">
                    <t t-set="post_id" t-value="question"/>
                </t>
                <!-- Answers -->
                <section t-if="question.child_count">
                    <t t-foreach="question.child_ids" t-as="answer">
                        <t t-if="answer.state != 'flagged' or (answer.state == 'flagged' and answer.can_moderate)" t-call="website_forum.post_display">
                            <t t-set="post_id" t-value="answer"/>
                        </t>
                    </t>
                </section>
                <section t-elif="uid and question.state == 'active' and not forum.has_pending_post and question.can_answer" class="alert alert-info mt-3 mb-0">
                    <h5>There are no answers yet</h5>
                    <span>Be the first to answer this question</span>
                </section>
                <!-- Write Answer -->
                <div t-if="uid and not question.uid_has_answered and question.can_answer and question.state == 'active' and question.active != False" class="d-flex mt-3">
                    <a t-attf-class="btn btn-link collapsed #{ 'karma_required text-muted' if not question.can_answer else '' }#{ ' disabled' if forum.has_pending_post else ''} text-decoration-none"
                        t-att-data-karma="question.forum_id.karma_answer"
                        data-bs-toggle="collapse"
                        data-bs-target=".answer_collapse"
                        href="#">
                        <i class="fa fa-reply me-1"/>Answer
                    </a>
                </div>
                <t t-if="question.state != 'close' and question.active != False and question.can_answer and forum">
                    <div id="post_reply" class="answer_collapse position-fixed d-flex flex-column start-0 end-0 bottom-0 w-100 w-lg-50 shadow mx-auto px-3 bg-body"
                         t-if="forum and not forum.has_pending_post and (not question.uid_has_answered or question.forum_id.mode == 'discussions')">
                        <div t-call="website_forum.post_answer" class="container my-3"/>
                    </div>
                    <div t-elif="forum and forum.has_pending_post" class="alert alert-info text-center">
                        <b class="d-block">You have a pending post</b>
                        Please wait for a moderator to validate your previous post to be allowed replying questions.
                    </div>
                </t>
            </div>
        </div>
    </t>
</template>

<template id="post_display">
    <t t-if="post_id == answer">
        <a t-if="answer" t-attf-id="answer-#{str(answer.id)}"/>
        <t t-set="_answer_creator" t-value="post_id.create_uid.id"/>
        <t t-set="post_type" t-value="'answer'" />
    </t>
    <t t-else="post_id == question">
        <t t-set="_question_creator" t-value="post_id.create_uid.id"/>
        <t t-set="post_type" t-value="'question'"/>
    </t>

    <div t-attf-class="o_wforum_#{post_type} row g-0 mb-2 rounded
                       #{ 'o_wforum_answer_correct my-2 mx-n3 mx-lg-n2 mx-xl-n3 py-3 px-3 px-lg-2 px-xl-3' if post_id == answer and post_id.is_correct else '' }"
        t-att-data-type="post_type"
        t-att-data-last-activity="post_id.create_date"
        t-att-data-last-update="post_id.write_date"
        t-attf-data-id="#{post_type}-#{post_id.id}"
        t-attf-id="#{post_type}-#{post_id.id}"
        t-att-data-state="post_id.state">
            <div class="d-flex flex-column col-auto pe-2 pe-lg-3">
                <t t-call="website_forum.author_box">
                    <t t-set="object" t-value="post_id"/>
                    <t t-set="show_image" t-value="True"/>
                </t>
                <div t-attf-class="align-self-center flex-grow-1 mt-2 border-start opacity-50 #{ 'border-success' if post_id.is_correct else '' }"/>
            </div>
            <div class="post_content_wrapper col">
                <header class="o_wforum_post_header d-flex align-items-center mb-1">
                    <t t-call="website_forum.author_box">
                        <t t-set="object" t-value="post_id"/>
                        <t t-set="allow_biography" t-value="True"/>
                        <t t-set="show_name" t-value="True"/>
                    </t>
                    <span class="o_wforum_relative_datetime ms-2 opacity-75 small text-muted">
                    </span>
                    <t t-if="answer">
                        <span t-if="_question_creator == _answer_creator" class="badge ms-2 border border-dark rounded-pill px-2 py-1 text-reset">
                            Author
                        </span>
                        <span t-if="question.forum_id.mode == 'questions'" t-attf-class="o_wforum_answer_correct_badge badge #{ 'd-inline' if answer.is_correct else 'd-none' } ms-auto border border-success rounded-pill px-2 py-1 text-success">
                            Best Answer
                        </span>
                    </t>
                </header>
                <t t-if="answer and post_id.state == 'flagged' and post_id.can_flag">
                    <div class="alert alert-danger text-center o_wforum_flag_alert">
                        <p><b class="d-block ">This answer has been flagged</b>
                            As a moderator, you can either validate or reject this answer.</p>
                        <t t-call="website_forum.link_button">
                            <t t-set="url" t-valuef="/forum/#{ slug(forum) }/post/#{ slug(post_id) }/flag"/>
                            <t t-set="form_method" t-value="'GET'"/>
                            <t t-set="karma" t-value="post_id.forum_id.karma_flag if not post_id.can_flag else 0"/>
                            <t t-set="classes" t-valuef="o_wforum_flag"/>
                            <t t-set="flag_validator" t-value="post_id.can_moderate"/>
                            <t t-set="object" t-value="post_id"/>
                        </t>
                    </div>
                </t>

                <div t-field="post_id.content" class="o_wforum_post_content text-break o_wforum_readable oe_no_empty o_not_editable"/>

                <t t-if="post_id == answer">
                    <t t-set="_answer_comment_collapse_uid" t-value="'comment_%s_%s' % (post_id._name.replace('.', '_'), post_id.id)"/>
                </t>
                <t t-elif="post_id == question">
                    <t t-set="_question_comment_collapse_uid" t-value="'comment_%s_%s' % (post_id._name.replace('.', '_'), post_id.id)"/>
                </t>
                <t t-set="can_interact_with_post"
                   t-value="not request.env.user._is_public() and
                            (post_id.can_close or post_id.can_edit or post_id.can_unlink or post_id.can_moderate or post_id.can_flag)"/>
                <div class="btn-toolbar align-items-center mt-3" role="toolbar">
                    <t t-if="post_id.state == 'active' and post_id.active != False" t-call="website_forum.vote">
                        <t t-set="post" t-value="post_id"/>
                        <t t-if="post_id == answer">
                            <t t-set="helper_accept">Mark as Best Answer</t>
                            <t t-set="helper_own_post">You can't vote for your own post</t>
                            <t t-set="helper_no_karma">You don't have enough karma</t>
                            <t t-set="helper_decline">Unmark as Best Answer</t>
                            <a t-if="question.can_answer and question.forum_id.mode == 'questions'"
                                t-attf-class="o_wforum_validate_toggler btn #{ 'opacity-50 opacity-100-hover' if not answer.is_correct and answer.create_uid.id != uid and post_id.can_accept else ''} #{ 'karma_required opacity-25 ' if not post_id.can_accept else ''}#{ 'opacity-25' if answer.create_uid.id == uid else ''}"
                                href="#"
                                t-attf-data-karma="#{post_id.karma_accept}"
                                t-att-data-helper-accept="helper_accept"
                                t-att-data-helper-own-post="helper_own_post"
                                t-att-data-helper-no-karma="helper_no_karma"
                                t-att-data-helper-decline="helper_decline"
                                t-att-title="helper_no_karma if not post_id.can_accept else (helper_decline if answer.is_correct else (helper_accept if not answer.is_correct else (helper_own_post if answer.is_correct and answer.create_uid.id == uid else '')))"
                                data-bs-toggle="tooltip"
                                t-attf-data-target="#answer-#{post_id.id}"
                                t-attf-data-href="/forum/#{ slug(question.forum_id) }/post/#{ slug(answer) }/toggle_correct">
                                <i t-attf-class="fa fa-lg #{ 'fa-check-circle text-success' if answer.is_correct else 'fa-check-circle-o' }"/>
                            </a>
                        </t>
                    </t>
                    <div class="d-flex align-items-center ms-auto">
                        <t t-if="post_id == question">
                            <t t-set="user_answer" t-value="uid and post_id.child_ids.filtered(lambda a: a.create_uid.id == uid)"/>
                            <div t-if="not user_answer and uid and question.can_answer and question.state == 'active' and question.active != False" class="d-flex ms-auto me-2">
                                <a t-attf-class="btn btn-link collapsed #{ 'karma_required text-muted' if not question.can_answer else '' }#{ 'disabled' if forum.has_pending_post else '' } text-decoration-none"
                                    t-att-data-karma="question.forum_id.karma_answer"
                                    data-bs-toggle="collapse" data-bs-target=".answer_collapse"
                                    href="#"
                                >
                                    <i class="fa fa-reply me-1"/>Answer
                                </a>
                            </div>
                            <a t-if="user_answer" class="btn btn-sm btn-primary"
                                t-att-title="question.forum_id.mode == 'questions' and edit_answer_title"
                                data-bs-toggle="tooltip" data-bs-placement="top"
                                t-attf-href="#answer-#{user_answer.id}"
                            >
                                View my answer <i class="oi oi-arrow-right ms-1"/>
                            </a>
                        </t>
                        <div class="d-flex ms-auto me-2" t-if="post_id == answer and post_id.create_uid.id == uid">
                            <t t-set="edit_answer_title">You are only allowed one answer</t>

                            <a class="btn btn-sm btn-outline-primary"
                                t-att-title="question.forum_id.mode == 'questions' and edit_answer_title"
                                data-bs-toggle="tooltip" data-bs-placement="top"
                                t-attf-href="/forum/#{slug(forum)}/question/#{slug(question)}/edit_answer">
                                <i class="fa fa-pencil"/>
                                Edit<span class="d-none d-lg-inline"> your answer</span>
                            </a>
                        </div>
                        <t t-if="post_id.state == 'active' and post_id.active != False">
                            <a t-attf-class="btn #{ 'd-none ' if not uid else '' }#{'karma_required opacity-25' if not post_id.can_comment else 'opacity-50 opacity-100-hover'}"
                               t-att-data-karma="{post_id.karma_comment if not post_id.can_comment else 0}"
                               t-att-data-bs-toggle="'collapse' if post_id.can_comment else None"
                               t-attf-href="##{ _answer_comment_collapse_uid if post_id == answer else _question_comment_collapse_uid}"
                            >
                                <i t-attf-class="fa fa-comment #{ 'karma_required' if not post_id.can_comment else ''}" title="Comment" data-bs-toggle="tooltip" data-bs-placement="top" />
                            </a>
                            <a href="javascript:void(0)" class="oe_social_share btn opacity-50 opacity-100-hover"
                               t-attf-data-urlshare="#{request.httprequest.url}#{'#' + '' if post_id == answer else ''}"
                               t-attf-data-hashtags="##{post_type}"
                            >
                                <i class="fa fa-share-alt" data-bs-toggle="tooltip" data-bs-placement="top" title="Share"/>
                            </a>
                        </t>
                        <t t-if="can_interact_with_post" t-call="website_forum.post_dropdown"/>
                    </div>
                </div>
                <t t-call="website_forum.post_comment">
                    <t t-set="object" t-value="post_id"/>
                    <t t-if="post_id == answer">
                        <t t-set="_collapse_uid" t-value="_answer_comment_collapse_uid"/>
                    </t>
                    <t t-if="post_id == question">
                        <t t-set="_collapse_uid" t-value="_question_comment_collapse_uid"/>
                   </t>
                </t>
            </div>
        </div>
</template>

<template id="post_dropdown">
    <a class="btn opacity-50 opacity-100-hover" href="#" role="button" id="dropdownMenuLink" data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
        <i class="fa fa-ellipsis-h"  data-bs-toggle="tooltip" data-bs-placement="top" title="More"/>
    </a>
    <div class="dropdown-menu dropdown-menu-end" aria-labelledby="dropdownMenuLink">
        <t t-if="post_id == question">
            <t t-if="post_id.state == 'close'" t-call="website_forum.link_button">
                <t t-set="url" t-value="'/forum/' + slug(forum) +'/question/' + slug(post_id) + '/reopen'"/>
                <t t-set="label">Reopen</t>
                <t t-set="inDropdown" t-value="True"/>
                <t t-set="icon" t-value="'fa-undo fa-fw'"/>
                <t t-set="karma" t-value="post_id.karma_close if not post_id.can_close else 0"/>
            </t>
        </t>
        <t t-call="website_forum.link_button">
            <t t-set="url" t-value="'/forum/' + slug(forum) +'/post/' + slug(post_id) + '/edit'"/>
            <t t-set="label">Edit</t>
            <t t-set="inDropdown" t-value="True"/>
            <t t-set="icon" t-value="'fa-pencil fa-fw'"/>
            <t t-set="karma" t-value="post_id.karma_edit if not post_id.can_edit else 0"/>
        </t>
        <t t-if="post_id == question">
            <t t-if="post_id.state != 'close'" t-call="website_forum.link_button">
                <t t-set="url" t-value="'/forum/' + slug(forum) +'/question/' + slug(post_id) + '/ask_for_close'"/>
                <t t-set="label">Close</t>
                <t t-set="inDropdown" t-value="True"/>
                <t t-set="icon" t-value="'fa-times fa-fw'"/>
                <t t-set="karma" t-value="post_id.karma_close if not post_id.can_close else 0"/>
            </t>
            <t t-if="not post_id.active and post_id.state != 'offensive'" t-call="website_forum.link_button">
                <t t-set="url" t-value="'/forum/' + slug(forum) +'/question/' + slug(post_id) + '/undelete'"/>
                <t t-set="label">Undelete</t>
                <t t-set="inDropdown" t-value="True"/>
                <t t-set="icon" t-value="'fa-upload fa-fw'"/>
                <t t-set="karma" t-value="post_id.karma_unlink if not post_id.can_unlink else 0"/>
            </t>
            <t t-if="not post_id.active and post_id.state == 'offensive'" t-call="website_forum.link_button">
                <t t-set="url" t-value="'/forum/' + slug(forum) +'/post/' + slug(post_id) + '/validate'"/>
                <t t-set="label">Validate</t>
                <t t-set="inDropdown" t-value="True"/>
                <t t-set="icon" t-value="'fa-check fa-fw'"/>
                <t t-set="karma" t-value="post_id.forum_id.karma_moderate if not post_id.can_moderate else 0"/>
            </t>
        </t>
        <t t-if="post_id.active" t-call="website_forum.link_button">
            <t t-set="url" t-value="'/forum/' + slug(forum) +'/question/' + slug(post_id) + '/delete'"/>
            <t t-set="label">Delete</t>
            <t t-set="inDropdown" t-value="True"/>
            <t t-set="icon" t-value="'fa-trash-o fa-fw'"/>
            <t t-set="karma" t-value="post_id.karma_unlink if not post_id.can_unlink else 0"/>
        </t>
        <t t-if="post_id.active and post_id.state != 'flagged'" t-call="website_forum.link_button">
            <t t-set="url" t-value="'/forum/' + slug(forum) +'/post/' + slug(post_id) + '/flag'"/>
            <t t-set="label">Flag</t>
            <t t-set="inDropdown" t-value="True"/>
            <t t-set="icon" t-value="'fa-flag-o' if not post_id.can_flag else 'fa-flag'"/>
            <t t-set="karma" t-value="post_id.forum_id.karma_flag if not post_id.can_flag else 0"/>
            <t t-set="classes" t-valuef="o_wforum_flag "/>
            <t t-set="object" t-value="post_id"/>
        </t>
    </div>
</template>

<!-- Utility template: Post a Comment -->
<template id="post_comment">
    <div class="o_wforum_post_comments_container d-flex flex-column gap-2 rounded">
        <div class="css_editable_mode_hidden o_wforum_readable">
            <form t-att-id="_collapse_uid" class="oe_comment_grey js_website_submit_form js_wforum_submit_form collapse rounded o_cc2 p-2"
                t-attf-action="/forum/#{slug(forum)}/post/#{slug(object)}/comment" method="POST">
                    <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                    <input name="post_id" t-att-value="object.id" type="hidden"/>
                    <div class="d-flex gap-1 w-100">
                        <img class="o_wforum_avatar d-none d-md-inline-block align-self-baseline rounded-circle me-1" t-att-src="request.website.image_url(user, 'avatar_128', '36x36')" alt="Avatar"/>
                        <div class="w-100">
                            <textarea name="comment" class="form-control" rows="3" placeholder="Comment this post"/>
                            <div class="d-flex gap-1 mt-1">
                                <button type="submit" class="o_wforum_submit_post btn btn-primary align-self-baseline text-nowrap">Add a comment</button>
                                <a class="btn btn-link align-self-baseline text-nowrap" data-bs-toggle="collapse" t-attf-href="##{_collapse_uid}">Discard</a>
                            </div>
                        </div>
                    </div>
            </form>
        </div>

        <div t-if="object.website_message_ids" t-foreach="reversed(object.website_message_ids)" t-as="message" class="o_wforum_post_comments d-flex flex-column gap-2">
            <t t-set="_comment_author" t-value="message.create_uid.id"/>
            <div t-attf-class="o_wforum_post_comment d-flex rounded o_cc2 ps-3" t-att-data-last-activity="message.create_date">
                <t t-set="allow_post_comment" t-value="(object.parent_id and object.parent_id.state != 'close' and object.parent_id.active != False)
                                                        or (not object.parent_id and object.state != 'close' and object.active != False)"/>
                <t t-set="unlink_comment_required_karma" t-value="message.author_id.id == user.partner_id.id and object.forum_id.karma_comment_unlink_own or object.forum_id.karma_comment_unlink_all"/>
                <t t-set="can_unlink_comment" t-value="user._is_admin() or user.karma >= unlink_comment_required_karma"/>
                <div class="flex-grow-1 py-2 opacity-75">
                    <header>
                        <span t-call="website_forum.author_box">
                            <t t-set="object" t-value="message"/>
                            <t t-set="compact" t-value="True"/>
                            <t t-set="show_name" t-value="True"/>
                        </span>
                        <span class="ms-2 mb-2 small text-muted"><small class="o_wforum_relative_datetime"/></span>
                        <span t-if="_question_creator == _comment_author" class="badge ms-2 border border-dark rounded-pill px-2 py-1 text-reset">
                            Author
                        </span>
                    </header>
                    <div t-field="message.body" class="o_wforum_readable oe_no_empty small text-break"/>
                </div>
                <div t-if="can_interact_with_post" class="dropdown ms-auto">
                    <a class="btn text-muted" type="button" id="dropdownMenuButton" data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                        <i class="fa fa-ellipsis-h"/>
                    </a>
                    <div class="dropdown-menu">
                        <t t-set="comment_url" t-valuef="/forum/#{ slug(forum) }/post/#{ slug(object) }/comment/#{ slug(message) }"/>
                        <t t-call="website_forum.link_button">
                            <t t-set="url" t-valuef="#{ comment_url }/delete"/>
                            <t t-set="label">Delete</t>
                            <t t-set="title">Delete</t>
                            <t t-set="inDropdown" t-value="True"/>
                            <t t-set="icon" t-value="'fa-trash-o text-muted'"/>
                            <t t-set="classes" t-value="'comment_delete'"/>
                            <t t-set="karma" t-value="unlink_comment_required_karma if not can_unlink_comment and unlink_comment_required_karma else 0"/>
                        </t>
                        <t t-if="message.create_uid.id not in question.child_ids.create_uid.ids">
                            <t t-call="website_forum.link_button">
                                <t t-set="url" t-valuef="#{ comment_url }/convert_to_answer"/>
                                <t t-set="label">Convert to answer</t>
                                <t t-set="inDropdown" t-value="True"/>
                                <t t-set="icon" t-value="'fa-magic text-muted'"/>
                                <t t-set="karma" t-value="object.karma_comment_convert if not object.can_comment_convert else 0"/>
                            </t>
                        </t>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

    </data>
</odoo>

```

## File: views\forum_forum_templates_tools.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

<!-- TOOLS / UTILS -->
<!-- ============================================================ -->

<template id="show_flag_validator">
    <a href="#" t-att-data-post-id="object.id" t-attf-data-action="validate" t-attf-class="o_wforum_flag_validator flag_validator btn btn-success #{'' if object.state == 'flagged' else 'd-none'} my-2 me-2" title="Validate" data-bs-toggle="tooltip" data-bs-placement="top">
        <i class="fa fa-check"/> Accept
    </a>
    <a href="#" t-attf-data-action="/forum/#{object.id}/ask_for_mark_as_offensive" t-attf-class="o_wforum_flag_mark_as_offensive flag_validator #{'' if object.state == 'flagged' else 'd-none'} btn btn-danger my-2 me-2" title="Mark as Offensive"  data-bs-toggle="tooltip" data-bs-placement="top">
        <i class="fa fa-times"/> Reject
    </a>
</template>

<template id="link_button">
    <form t-attf-method="#{form_method or 'POST'}" t-att-action="url" t-attf-class="#{form_classes}">
        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
        <button t-if="label" t-attf-class="#{('fa ' + icon) if icon and not label else ''} #{ 'dropdown-item ps-3' if inDropdown else 'btn'} #{classes}#{ ' karma_required text-muted' if karma > 0 else ''}" t-attf-data-karma="#{karma}" t-att-title="title">
            <i t-if="icon and label" t-attf-class="fa fa-fw #{icon} #{ 'me-1' if inDropdown else ''}"/>
            <t t-out="label"/>
        </button>
        <t t-if="flag_validator" t-call="website_forum.show_flag_validator"/>
    </form>
</template>

<!-- FAQ LAYOUT -->
<!-- ============================================================ -->

<!-- FAQ Layout -->
<template id="faq" name="Frequently Asked Questions">
    <t t-call="website_forum.header">
        <t t-set="_page_name" t-value="'guidelines'"/>
        <t t-set="_page_name_label">Guidelines</t>
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
                                <td class="faq-rep-item"><strong t-field="forum.karma_editor"/></td>
                                <td>insert text link, upload files</td>
                            </tr><tr>
                                <td class="faq-rep-item"><strong t-field="forum.karma_downvote"/></td>
                                <td>downvote</td>
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
                                <td class="faq-rep-item"><strong t-field="forum.karma_user_bio"/></td>
                                <td>your biography can be seen as tooltip</td>
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

<!-- TOOLS / UTILS -->
<!-- ============================================================ -->

<template id="vote">
    <t t-set="own_vote" t-value="post.user_vote"/>
    <t t-set="forum" t-value="post.forum_id"/>
    <!-- can_upvote returns "true" for unregistered users -->
    <t t-set="can_upvote" t-value="post.can_upvote"/>
    <t t-set="can_downvote" t-value="post.can_downvote"/>
    <div t-attf-class="vote d-inline-flex align-items-center ms-n2 #{classes} text-muted text-center">
        <button type="button" t-attf-data-href="/forum/#{slug(forum)}/post/#{slug(post)}/upvote"
            t-attf-class="btn vote_up #{ 'text-success' if own_vote == 1 else '' } #{ 'karma_required text-reset opacity-25' if not can_upvote else ''}#{ ' text-300' if post.create_uid.id == uid else ''}"
            t-att-disabled="own_vote == 1 and 'disabled'"
            t-att-data-karma="forum.karma_upvote"
            t-att-data-can-upvote="can_upvote"
            aria-label="Upvote" title="Upvote">
            <i class="fa fa-caret-up" data-bs-toggle="tooltip" data-bs-placement="top" title="Upvote"/>
        </button>
        <small t-attf-class="vote_count #{ 'text-success' if own_vote == 1 else 'text-danger' if own_vote == -1 else ('text-muted opacity-75' if post.vote_count == 0 and not own_vote else '') }"
            t-out="post.vote_count"/>
        <button type="button" t-attf-data-href="/forum/#{slug(forum)}/post/#{slug(post)}/downvote"
            t-attf-class="btn vote_down #{ 'text-danger' if own_vote == -1 else ''} #{'karma_required text-reset opacity-25' if not can_downvote else '' }#{' text-300' if post.create_uid.id == uid else '' }"
            t-att-disabled="own_vote == -1 and 'disabled'"
            t-att-data-karma="forum.karma_downvote"
            t-att-data-can-downvote="can_downvote"
            aria-label="Downvote" title="Downvote">
            <i class="fa fa-caret-down" data-bs-toggle="tooltip" data-bs-placement="top" title="Downvote"/>
        </button>
        <t t-out="0"/>
    </div>
</template>

<template id="author_box">
    <t t-set="display_info" t-value="show_name or show_date or show_karma"/>
    <t t-if="allow_biography and object.can_display_biography and request.env.user.karma >= website.karma_profile_min" t-set="bio_popover_data">
        <div class="o_wforum_bio_popover_wrap d-flex transition-fade">
            <div class="d-flex flex-column">
                <img class="o_wforum_avatar flex-shrink-0 rounded me-3" t-att-src="request.website.image_url(object.create_uid, 'avatar_128', '128x128')" alt="Avatar"/>
            </div>
            <div>
                <h5 class="o_wforum_bio_popover_name mb-0" t-field="object.create_uid" t-options='{"widget": "contact", "country_image": True, "fields": ["name"]}'/>
                <span class="o_wforum_bio_popover_info" t-field="object.create_uid" t-options='{"widget": "contact", "UserBio": True, "badges": True, "fields": ["karma"]}'/>
                <div class="o_wforum_bio_popover_bio mt-1 mb-0" t-field="object.create_uid" t-options='{"widget": "contact", "website_description": True, "fields": ["partner_id"]}'/>
            </div>
        </div>
    </t>
    <div t-attf-class="o_wforum_author_box d-inline-flex #{ 'o_show_info ' if display_info else '' }#{ 'o_compact align-items-center ' if compact else '' }#{ 'o_wforum_bio_popover ' if bio_popover_data else ''}#{_box_classes}"
         t-att-data-bs-content="bio_popover_data"
         data-bs-placement="top">
        <t t-set="user_profile_url" t-value="False"/>
        <t t-if="'forum_id' in object and (object.create_uid.id == request.session.uid or object.create_uid.sudo().website_published)">
            <t t-set="user_profile_url" t-valuef="/forum/user/#{ object.create_uid.id }?forum_id=#{ object.forum_id.id }&amp;forum_origin=#{ request.httprequest.path }"/>
        </t>
        <a t-if="show_image" t-att-href="user_profile_url or '#'" t-attf-class="o_wforum_author_pic position-relative #{ 'pe-none' if not user_profile_url else '' }">
            <img t-attf-class="o_wforum_avatar rounded-circle o_object_fit_cover #{'me-2' if show_name or show_date or show_karma else '' } #{_image_classes}" t-att-src="website.image_url(object.create_uid, 'avatar_128', '60x60')" alt="Avatar"/>
        </a>
        <div t-if="show_name or show_date or show_karma" t-attf-class="d-flex #{ 'align-items-baseline' if compact else 'flex-column justify-content-around' } #{ 'ms-2' if show_image else '' }">
            <a t-att-href="user_profile_url" t-attf-class="my-0 text-reset h6 #{ 'small' if compact else ''}" t-field="object.create_uid" t-options='{"widget": "contact", "fields": ["name"]}'/>
            <small t-if="show_karma and show_date" class="text-muted fw-bold"> - <t t-out="object.create_uid.karma"/>xp</small>

            <div t-if="show_date or show_karma" t-attf-class="text-muted small #{ 'd-flex align-items-baseline' if compact else '' }">
                <div t-if="show_date">
                    <time  class="text-muted" t-field="object.create_date" t-options="{ 'format': 'd MMMM y ' }"/>
                    at
                    <time class="text-muted" t-field="object.create_date" t-options="{ 'format': 'HH:mm ' }"/>
                </div>
                <span t-if="show_karma and not show_date" class="text-muted"><t t-out="object.create_uid.karma"/>xp</span>
            </div>
        </div>
    </div>
</template>
    </data>
</odoo>

```

## File: views\forum_forum_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <!-- FORUM VIEWS -->
        <record id="forum_forum_view_tree" model="ir.ui.view">
            <field name="name">forum.forum.view.tree</field>
            <field name="model">forum.forum</field>
            <field name="arch" type="xml">
                <tree string="Forums">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                    <field name="website_id" groups="website.group_multi_website"/>
                    <field name="total_posts" sum="Total Posts"/>
                    <field name="total_views" sum="Total Views"/>
                    <field name="total_answers" optional="hide"/>
                    <field name="total_favorites" optional="hide"/>
                    <field name="active" column_invisible="True"/>
                </tree>
            </field>
        </record>

        <record id="forum_forum_view_form" model="ir.ui.view">
            <field name="name">forum.forum.view.form</field>
            <field name="model">forum.forum</field>
            <field name="arch" type="xml">
                <form string="Forum">
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button name="%(forum_post_action_forum_main)d" type="action" class="oe_stat_button" icon="fa-comments">
                                <div class="o_form_field o_stat_info">
                                    <span class="o_stat_value">
                                        <field name="total_posts" />
                                    </span>
                                    <span class="o_stat_text">Posts</span>
                                </div>
                            </button>
                            <button name="%(forum_post_action_favorites)d" class="oe_stat_button" icon="fa-star" type="action">
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
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                        <field name="image_1920" widget="image" options="{'preview_image': 'image_128'}" class="oe_avatar"/>
                        <div class="oe_title">
                            <label for="name"/>
                            <h1>
                                <field name="name" placeholder="e.g. Help"/>
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
                                        <field name="privacy" widget="radio" required="True"/>
                                        <field name="authorized_group_id" options="{'no_create': True}" invisible="privacy != 'private'" required="privacy == 'private'"/>
                                        <label for="relevancy_post_vote" string="Relevance Computation" groups="base.group_no_one" invisible="default_order != 'relevancy desc'"/>
                                        <div groups="base.group_no_one" class="o_row" invisible="default_order != 'relevancy desc'">
                                            (votes - 1) ** <field name="relevancy_post_vote"/> / (days + 2) ** <field name="relevancy_time_decay"/>
                                        </div>
                                    </group>
                                </group>

                                <field name="description" placeholder="Description visible on website"/>
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
                                        <field name="karma_user_bio"/>
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

        <record id="forum_forum_view_form_add" model="ir.ui.view">
            <field name="name">forum.forum.view.form.add</field>
            <field name="model">forum.forum</field>
            <field name="arch" type="xml">
                <form js_class="website_forum_add_form">
                    <group>
                        <field name="name" placeholder="e.g. Technical Assistance" string="Forum Name"/>
                        <field name="mode" widget="radio" required="True" string="Forum Mode"/>
                        <field name="privacy" widget="radio" required="True"/>
                        <field name="authorized_group_id" options="{'no_create': True}" invisible="privacy != 'private'" required="privacy == 'private'"/>
                    </group>
                </form>
            </field>
        </record>

        <record id="forum_forum_view_search" model="ir.ui.view">
            <field name="name">forum.forum.view.search</field>
            <field name="model">forum.forum</field>
            <field name="arch" type="xml">
                <search string="Forum">
                    <field name="name"/>
                    <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                </search>
            </field>
        </record>

        <record id="forum_forum_action" model="ir.actions.act_window">
            <field name="name">Forums</field>
            <field name="res_model">forum.forum</field>
            <field name="view_mode">tree,form</field>
        </record>

        <record id="forum_forum_action_add" model="ir.actions.act_window">
            <field name="name">New Forum</field>
            <field name="res_model">forum.forum</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
            <field name="view_id" ref="forum_forum_view_form_add"/>
        </record>

    </data>
</odoo>

```

## File: views\forum_menus.xml

```xml
<?xml version="1.0"?>
<odoo><data>
    <!-- FORUM MENUS -->
    <menuitem
        name="Forum"
        id="menu_website_forum_global"
        parent="website.menu_website_global_configuration"
        sequence="170"
        groups="website.group_website_designer"/>

    <menuitem
        id="menu_forum_global"
        parent="menu_website_forum_global"
        name="Forums"
        action="forum_forum_action"
        sequence="10"/>
    <menuitem
        id="menu_forum_rank_global"
        parent="menu_website_forum_global"
        name="Ranks"
        action="gamification.gamification_karma_ranks_action"
        sequence="20"/>
    <menuitem
        id="menu_forum_tag_global"
        parent="menu_website_forum_global"
        name="Tags"
        action="forum_tag_action"
        sequence="30"/>
    <menuitem
        id="menu_forum_badges"
        parent="menu_website_forum_global"
        name="Badges"
        action="gamification.badge_list_action"
        sequence="40"/>
    <menuitem
        id="menu_forum_post_reasons"
        parent="menu_website_forum_global"
        name="Close Reasons"
        action="forum_post_reason_action"
        sequence="50"/>

    <!-- WEBSITE MENUS -->
    <menuitem
        id="menu_forum_post_pages"
        parent="website.menu_content"
        sequence="80"
        name="Forum Posts"
        action="forum_post_action"/>

</data></odoo>

```

## File: views\forum_post_reason_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>
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

    <record id="forum_post_reason_action" model="ir.actions.act_window">
        <field name="name">Post Close Reason</field>
        <field name="res_model">forum.post.reason</field>
        <field name="view_mode">tree</field>
    </record>
</data></odoo>

```

## File: views\forum_post_views.xml

```xml
<?xml version="1.0"?>
<odoo><data noupdate="1">

<record id="forum_post_view_form" model="ir.ui.view">
    <field name="name">forum.post.view.form</field>
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
                <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                <label for="name"/>
                <h1>
                    <field name="name" placeholder="e.g. When should I plant my tomatoes?"/>
                </h1>
                <group>
                    <group name="forum_details">
                        <field name="active" invisible="1"/>
                        <field name="forum_id"/>
                        <field name="website_id" groups="website.group_multi_website"/>
                        <field name="parent_id" invisible="not parent_id"/>
                    </group>
                    <group name="post_details">
                        <field name="tag_ids" widget="many2many_tags"/>
                        <field name="state"/>
                        <field name="closed_reason_id" invisible="not closed_reason_id"/>
                        <field name="closed_uid" invisible="not closed_uid"/>
                        <field name="closed_date" invisible="not closed_date"/>
                    </group>
                    <group name="creation_details">
                        <field name="create_uid"/>
                        <field name="create_date"/>
                        <field name="write_uid"/>
                        <field name="write_date"/>
                    </group>
                    <group name="post_statistics">
                        <field name="is_correct" invisible="not parent_id"/>
                        <field name="views"/>
                        <field name="vote_count"/>
                        <field name="favourite_count"/>
                        <field name="child_count"/>
                        <field name="relevancy"/>
                    </group>
                </group>
                <group name="answers" string="Answers" invisible="parent_id">
                    <field name="child_ids" nolabel="1" colspan="2">
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

<record id="forum_post_view_search" model="ir.ui.view">
    <field name="name">forum.post.view.search</field>
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

<record id="forum_post_view_graph" model="ir.ui.view">
    <field name="name">forum.post.view.graph</field>
    <field name="model">forum.post</field>
    <field name="arch" type="xml">
        <graph string="Graph of Posts" sample="1">
            <field name="write_date" interval="month"/>
            <field name="forum_id"/>
        </graph>
    </field>
</record>

<record id="forum_post_view_tree" model="ir.ui.view">
    <field name="name">forum.post.view.tree</field>
    <field name="model">forum.post</field>
    <field name="priority">99</field>
    <field name="arch" type="xml">
        <tree js_class="website_pages_list" create="false" type="object" action="go_to_website" multi_edit="1">
            <field name="active" column_invisible="True"/>
            <field name="name"/>
            <field name="website_url"/>

            <field name="forum_id" optional="show"/>
            <field name="views" string="# Views" sum="Total Views" optional="hide"/>
            <field name="child_count" string="# Answers" sum="Total Answers" optional="hide"/>
            <field name="state" widget="badge"
                decoration-success="state == 'active'"
                decoration-danger="state == 'close'"
                decoration-warning="state not in ('active', 'close')" optional="show"/>

            <field name="is_seo_optimized"/>

            <field name="website_id" groups="website.group_multi_website"/>
        </tree>
    </field>
</record>

<record id="forum_post_view_kanban" model="ir.ui.view">
    <field name="name">Forum Post Pages Kanban</field>
    <field name="model">forum.post</field>
    <field name="priority">99</field>
    <field name="arch" type="xml">
        <kanban js_class="website_pages_kanban" create="false" action="go_to_website" type="object" sample="1">
            <field name="name"/>
            <field name="forum_id"/>
            <field name="parent_id"/>
            <field name="website_url" invisible="1"/>
            <templates>
                <t t-name="kanban-box">
                    <div class="d-flex flex-column">
                        <div class="o_text_overflow fw-bold">
                            <span t-esc="record.name.value"/>
                            <div class="text-muted" t-if="record.website_id.value" groups="website.group_multi_website">
                                <i class="fa fa-globe me-1" title="Website"/>
                                <field name="website_id"/>
                            </div>
                            <div class="text-muted">
                                <i class="fa fa-comments-o me-1" title="Forum"/><t t-esc="record.forum_id.value"/>
                                <span t-if="!record.parent_id.raw_value" class="ms-3"><field name="child_count"/> Answers</span>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-6 text-primary"><i class="fa fa-eye me-1" title="Views"/><field name="views"/></div>
                            <div class="col-6 text-end"><field name="create_uid" widget="many2one_avatar_user"/></div>
                        </div>
                    </div>
                </t>
            </templates>
        </kanban>
    </field>
</record>

<record id="forum_post_action" model="ir.actions.act_window">
    <field name="name">Forum Post Pages</field>
    <field name="res_model">forum.post</field>
    <field name="view_mode">tree,kanban,graph</field>
    <field name="view_ids" eval="[(5, 0, 0),
        (0, 0, {'view_mode': 'tree', 'view_id': ref('forum_post_view_tree')}),
        (0, 0, {'view_mode': 'kanban', 'view_id': ref('forum_post_view_kanban')}),
    ]"/>
    <field name="search_view_id" ref="forum_post_view_search"/>
    <field name="context">{'search_default_posts': 1, 'create_action': 'website_forum.forum_forum_action_add'}</field>
    <field name="help" type="html">
        <p class="o_view_nocontent_smiling_face">
            Create a new forum post
        </p>
    </field>
</record>

<record id="forum_post_action_favorites" model="ir.actions.act_window">
    <field name="name">Users favorite posts</field>
    <field name="res_model">forum.post</field>
    <field name="view_mode">tree,form</field>
    <field name="domain">[('forum_id', '=', active_id), ('favourite_count', '>', 0), ('state', 'in', ('active', 'close'))]</field>
</record>

<record id="forum_post_action_forum_main" model="ir.actions.act_window">
    <field name="name">Posts</field>
    <field name="res_model">forum.post</field>
    <field name="view_mode">tree,form</field>
    <field name="domain">[('forum_id', '=', active_id), ('parent_id', '=', False), ('state', 'in', ('active', 'close'))]</field>
</record>

</data></odoo>

```

## File: views\forum_tag_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>
    <!-- TAG VIEWS -->
    <record id="forum_tag_view_list" model="ir.ui.view">
        <field name="name">forum.tag.view.list</field>
        <field name="model">forum.tag</field>
        <field name="arch" type="xml">
            <tree string="Tags" editable="bottom">
                <field name="name"/>
                <field name="forum_id" options="{'no_create_edit': True}"/>
            </tree>
        </field>
    </record>

    <record id="forum_tag_view_form" model="ir.ui.view">
        <field name="name">forum.tag.view.form</field>
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
</data></odoo>

```

## File: views\forum_templates_mail.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <template id="follow">
        <div t-attf-class="js_follow #{div_class}"
            t-att-data-id="object.id"
            t-att-data-object="object._name"
            t-att-data-follow="object.id and object.message_is_follower and 'on' or 'off'"
            t-att-data-unsubscribe="'unsubscribe' if 'unsubscribe' in request.params else None">
            <div t-if="icons_design and not request.env.user.has_group('base.group_public')" class="js_follow_icons_container">
                <button t-attf-class="btn js_unfollow_btn #{btn_classes}">
                    <i t-attf-class="fa fa-fw fa-minus-circle #{icons_classes}" data-bs-toggle="tooltip" data-bs-placement="top" title="Unfollow"/>
                </button>
                <button t-attf-class="btn js_follow_btn #{btn_classes}">
                    <i t-attf-class="fa fa-fw fa-plus-circle #{icons_classes}" data-bs-toggle="tooltip" data-bs-placement="top" title="Follow"/>
                </button>
            </div>
            <span t-elif="request.env.user.has_group('base.group_public') and icons_design" class="js_follow_icons_container">
                <button t-attf-class="btn js_unfollow_btn #{btn_classes}">
                    <i t-attf-class="fa fa-fw fa-minus-circle #{icons_classes}" data-bs-toggle="tooltip" data-bs-placement="top" title="Unfollow"/>
                </button>
                <button t-attf-class="btn follow_btn #{btn_classes}" data-bs-toggle="modal" data-bs-target="#o_wmail_follow_modal">
                    <i t-attf-class="fa fa-fw fa-plus-circle #{icons_classes}" data-bs-toggle="tooltip" data-bs-placement="top" title="Follow"/>
                </button>
            </span>
            <t t-else="">
                <button href="#" t-attf-class="btn btn-primary js_unfollow_btn #{btn_classes}"><i class="fa fa-fw fa-check me-1"/>Following</button>
                <button href="#" t-attf-class="btn btn-outline-primary js_follow_btn #{btn_classes}">Follow</button>
            </t>
            <div role="dialog" class="modal fade" id="o_wmail_follow_modal" aria-hidden="True">
                <div class="modal-dialog mw-lg-25" role="document">
                    <div class="modal-content">
                        <header class="modal-header" role="status">
                            <h4 class="modal-title">Subscribe</h4>
                            <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                        </header>
                        <main class="modal-body">
                            <p t-if="object == question">Get notified when there's activity on this post</p>
                            <p t-if="object == tag">Get notified when this tag is used</p>
                            <input type="email" name="email"
                                   class="js_follow_email form-control mb-2"
                                   placeholder="your email..."
                                   groups="base.group_public"/>
                            <button href="#" t-attf-class="btn btn-primary js_follow_btn">Subscribe</button>
                            <button href="#" t-attf-class="btn btn-secondary js_unfollow_btn"><i class="fa fa-fw fa-check me-1"/>Following</button>
                        </main>
                    </div>
                </div>
            </div>
        </div>
    </template>
</odoo>

```

## File: views\gamification_karma_tracking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="gamification_karma_tracking_view_search" model="ir.ui.view">
        <field name="name">gamification.karma.tracking.view.search.inherit.website.forum</field>
        <field name="model">gamification.karma.tracking</field>
        <field name="inherit_id" ref="gamification.gamification_karma_tracking_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='filter_res_users']" position='after'>
                <filter string="Forum" name="filter_forum_post"
                    domain="[('origin_ref', 'ilike', 'forum.post,')]"/>
            </xpath>
        </field>
    </record>
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

## File: views\website_profile_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <!--Access Denied - Profile Page-->
    <template id="profile_access_denied" inherit_id="website_profile.profile_access_denied">
        <xpath expr="//div[@id='profile_access_denied_return_link_container']" position="inside">
            <t t-if="request.params.get('forum_id')">
                <a t-attf-href="/forum/#{request.params.get('forum_id')}" class="btn btn-primary">Return to the forum</a>
            </t>
        </xpath>
    </template>

    <template id="user_profile_sub_nav" inherit_id="website_profile.user_profile_sub_nav">
        <xpath expr="//nav" position="before">
            <div t-if="request.params.get('forum_origin')" class="o_wprofile_all_users_nav_btn_container col pe-0 flex-grow-0">
                <a t-att-href="(request.website.domain or '') + '/' + request.params.get('forum_origin').lstrip('/')"
                    class="o_wprofile_all_users_nav_btn btn text-nowrap">
                    <i class="oi oi-chevron-left small"/> Back
                </a>
            </div>
        </xpath>
    </template>

    <template id="user_profile_content" inherit_id="website_profile.user_profile_content">
        <xpath expr="//table[@id='o_wprofile_sidebar_table']//tr[last()]" position="after">
            <t t-if="forum and (up_votes or down_votes)">
                <tr id="profile_abstract_info_company">
                    <th><small class="fw-bold">Votes</small></th>
                    <td>
                        <span>
                            <i class="fa fa-thumbs-up text-success" role="img" aria-label="Positive votes" title="Positive votes"/>
                            <span class="fw-bold" t-out="up_votes"/>
                            <i class="fa fa-thumbs-down text-danger ms-3" role="img" aria-label="Negative votes" title="Negative votes"/>
                            <span class="fw-bold" t-out="down_votes"/>
                        </span>
                    </td>
                </tr>
            </t>
        </xpath>
        <xpath expr="//ul[@id='profile_extra_info_tablist']" position="inside">
            <t t-if="forum">
                <li class="nav-item">
                    <a role="tab" aria-controls="questions" href="#questions" class="nav-link o_wprofile_navlink" data-bs-toggle="tab"><t t-out="count_questions"/> Questions</a>
                </li>
                <li class="nav-item">
                    <a role="tab" aria-controls="answers" href="#answers" class="nav-link o_wprofile_navlink" data-bs-toggle="tab"><t t-out="count_answers"/> Answers</a>
                </li>
                <li t-if="uid == user.id" class="nav-item">
                    <a role="tab" aria-controls="activity" href="#activity" class="nav-link o_wprofile_navlink" data-bs-toggle="tab">Activity</a>
                </li>
                <li t-if="uid == user.id" class="nav-item">
                    <a role="tab" aria-controls="votes" href="#votes" class="nav-link o_wprofile_navlink" data-bs-toggle="tab">Votes</a>
                </li>
            </t>
        </xpath>
        <xpath expr="//div[@id='profile_extra_info_tabcontent']" position="inside">
            <t t-if="forum">
                <t t-set="is_profile" t-value="true"/>
                <div role="tabpanel" class="o_wforum_profile_tab tab-pane" id="questions">
                    <div class="mb-4">
                        <h5 class="border-bottom pb-1 d-flex align-items-center justify-content-between">
                            Questions
                            <t t-if="count_questions &gt; 0" class="float-end" t-call="website.website_search_box_input">
                                <t t-set="_form_classes" t-valuef="d-flex flex-row align-items-center flex-wrap float-end"/>
                                <t t-set="search_type" t-valuef="forums"/>
                                <t t-if="forum_id" t-set="action" t-value="'/forum/%s%s' % (forum_id, ('/tag/%s/questions' % slug(tag)) if tag else '')"/>
                                <t t-else="" t-set="action" t-value="'/forum/all%s' % (('/tag/%s/questions' % slug(tag)) if tag else '')"/>
                                <t t-set="display_description" t-value="true"/>
                                <t t-set="display_detail" t-valuef="false"/>
                                <t t-set="placeholder" t-valuef="Search Questions..."/>
                                <input type="hidden" name="create_uid" t-att-value="user.id"/>
                                <input t-if="filters" type="hidden" name="filters" t-att-value="filters"/>
                                <input t-if="my_profile" type="hidden" name="my" t-att-value="mine"/>
                                <input t-if="sorting" type="hidden" name="sorting" t-att-value="sorting"/>
                            </t>
                        </h5>
                        <t t-call="website_forum.forum_filter_tag"/>
                        <t t-if="questions">
                            <div class="mb-1" t-foreach="questions" t-as="question">
                                <t t-call="website_forum.display_post"/>
                            </div>
                        </t>
                        <div t-elif="my_profile" class="text-muted">
                            You have not posted any questions yet. <br />
                            <a href="/forum/" class="btn btn-link px-0">
                                <i class="oi oi-arrow-right d-inline-block"/> Go to Forums
                            </a>
                        </div>
                        <t t-else="">
                            <div class="text-muted">This user hasn't posted any questions yet.<br/>
                                <a href="/forum/" class="btn btn-link px-0">
                                    <i class="oi oi-arrow-right d-inline-block"/> Go to Forums
                                </a>
                            </div>
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
                        <h5 class="border-bottom pb-1 d-flex align-items-center justify-content-between">
                            Answers
                            <t t-if="count_questions &gt; 0" class="float-end" t-call="website.website_search_box_input">
                                <t t-set="_form_classes" t-valuef="d-flex flex-row align-items-center flex-wrap float-end"/>
                                <t t-set="search_type" t-valuef="forums"/>
                                <t t-if="forum_id" t-set="action" t-value="'/forum/%s%s' % (forum_id, '/tag/%s/questions' % slug(tag)) if tag else ''"/>
                                <t t-else="" t-set="action" t-value="'/forum/all%s' % (('/tag/%s/questions' % slug(tag)) if tag else '')"/>
                                <t t-set="display_description" t-value="true"/>
                                <t t-set="display_detail" t-valuef="false"/>
                                <t t-set="placeholder" t-valuef="Search Answers..."/>
                                <input type="hidden" name="author" t-att-value="user.id"/>
                                <input t-if="filters" type="hidden" name="filters" t-att-value="filters"/>
                                <input t-if="my_profile" type="hidden" name="my" t-att-value="mine"/>
                                <input t-if="sorting" type="hidden" name="sorting" t-att-value="sorting"/>
                                <input type="hidden" name="include_answers" t-att-value="True"/>
                            </t>
                        </h5>
                        <t t-call="website_forum.forum_filter_tag"/>

                        <t t-if="answers">
                            <div class="mb-1" t-foreach="answers" t-as="answer">
                                <t t-call="website_forum.display_post_answer"/>
                            </div>
                        </t>
                        <div t-elif="my_profile" class="text-muted">
                            You have not answered any questions yet. <br />
                            <a href="/forum/" class="btn btn-link px-0">
                                <i class="oi oi-arrow-right d-inline-block"/> Go to Forums
                            </a>
                        </div>
                        <div t-else="" class="text-muted">
                            This user hasn't answered any questions yet. <br/>
                            <a href="/forum/" class="btn btn-link px-0">
                                <i class="oi oi-arrow-right d-inline-block"/> Go to Forums
                            </a>
                        </div>
                    </div>
                </div>
                <div role="tabpanel" class="o_wforum_profile_tab tab-pane" id="votes" t-if="uid == user.id">
                    <div class="mb-4">
                        <h5 class="border-bottom pb-1 flex align-items-center justify-content-between" style="height:43px;">
                            Votes
                            <t t-if="my_profile" class="float-end" t-call="website.website_search_box_input">
                                <t t-set="_form_classes" t-valuef="d-flex flex-row align-items-center flex-wrap float-end"/>
                                <t t-set="search_type" t-valuef="forums"/>
                                <t t-set="action" t-value="'/forum/%s%s' % (forum_id or 'all', '/tag/%s/questions' % slug(tag) if tag else '')"/>
                                <t t-set="display_description" t-value="true"/>
                                <t t-set="display_detail" t-valuef="false"/>
                                <t t-set="placeholder" t-valuef="Search Posts..."/>
                                <input type="hidden" name="my" value="upvoted"/>
                                <input t-if="filters" type="hidden" name="filters" t-att-value="filters"/>
                                <input t-if="sorting" type="hidden" name="sorting" t-att-value="sorting"/>
                                <input type="hidden" name="include_answers" t-att-value="True"/>
                            </t>
                        </h5>
                        <t t-call="website_forum.forum_filter_tag"/>
                        <t t-call="website_forum.user_votes"/>
                    </div>
                </div>
                <div role="tabpanel" class="o_wforum_profile_tab tab-pane" id="activity" t-if="uid == user.id">
                    <div class="mb-4">
                        <h5 class="border-bottom pb-1 d-flex align-items-center justify-content-between" style="height:43px;">
                            Activities
                        </h5>
                        <t t-call="website_forum.forum_filter_tag"/>
                        <t t-call="website_forum.display_activities"/>
                    </div>
                </div>
            </t>
        </xpath>
    </template>

    <template id="forum_filter_tag" name="Filtering Forum Tag">
        <t t-if="forum_filtered">
            <span class="d-inline-flex align-items-center gap-2 rounded py-1 ps-2 pe-0 bg-200 small">
                <i class="fa fa-filter text-muted"/>
                <t t-out="forum_filtered"/>
                <a t-attf-href="/profile/user/#{user.id}" class="oi oi-close btn m-n1 text-muted"/>
            </span>
        </t>
    </template>

    <template id="display_activities" name="Forum Profile Activities">
        <t t-if="activities">
            <div t-foreach="activities" t-as="activity" class="card mb-2">
                <div class="card-body">
                    <span t-out="activity.subtype_id.name" class="badge text-bg-info me-2 mt-1"/>
                    <span t-field="activity.date" t-options='{"format": "short"}' class="me-2"/>
                    <t t-set="post" t-value="posts[activity.res_id]"/>
                    <span t-if="post[1]">
                        <a t-attf-href="/forum/#{ slug(post[0].forum_id) }/#{ slug(post[0]) }#answer-#{ str(post[1].id) }">
                            <span t-out="post[0].name"/>
                        </a>
                    </span>
                    <span t-if="not post[1]">
                        <a t-attf-href="/forum/#{ slug(post[0].forum_id) }/#{ slug(post[0]) }">
                            <span t-out="post[0].name"/>
                        </a>
                    </span>
                </div>
            </div>
        </t>
        <t t-else="">
            <span class="text-muted">There is no activity yet.</span>
        </t>
    </template>

    <template id="user_votes" name="Forum User Votes">
        <div t-foreach="vote_post" t-as="vote" class="card mb-2">
            <div class="card-body">
                <span t-if="vote.vote == '1'" class="fa fa-thumbs-up text-success me-2" role="img" aria-label="Positive vote" title="Positive vote"/>
                <span t-if="vote.vote == '-1'" class="fa fa-thumbs-down text-warning me-2" role="img" aria-label="Negative vote" title="Negative vote"/>
                <span t-field="vote.post_id.create_date"/>
                <a t-if="vote.post_id.parent_id" class="ms-2" t-attf-href="/forum/#{ slug(vote.post_id.forum_id) }/#{ vote.post_id.parent_id.id }/#answer-#{ vote.post_id.id }" t-out="vote.post_id.parent_id.name"/>
                <a t-else="" class="text-black ms-2" t-attf-href="/forum/#{ slug(vote.post_id.forum_id) }/#{ vote.post_id.id }" t-out="vote.post_id.name"/>
            </div>
        </div>
        <div class="mb16" t-if="not vote_post">
            <div t-if="my_profile" class="text-muted">
                Help moderating the forums by upvoting and downvoting posts. <br />
                <a href="/forum/" class="btn-link">
                    <i class="fa fa-arrow-right"></i> Go To Forums
                </a>
            </div>
            <p t-else="" class="text-muted">You haven't given any votes yet.</p>
        </div>
    </template>
</data></odoo>

```

## File: views\snippets\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="forum_searchbar_input_snippet_options" inherit_id="website.searchbar_input_snippet_options" name="forum search bar snippet options">
    <xpath expr="//div[@data-js='SearchBar']/we-select[@data-name='scope_opt']" position="inside">
        <!-- Using /website/search/forums as result because the current forum search results page cannot be used across several forums -->
        <we-button data-set-search-type="forums" data-select-data-attribute="forums" data-name="search_forums_opt" data-form-action="/website/search/forums">Forums</we-button>
    </xpath>
    <xpath expr="//div[@data-js='SearchBar']/we-select[@data-name='order_opt']" position="inside">
        <we-button data-set-order-by="write_date asc" data-select-data-attribute="write_date asc" data-dependencies="search_forums_opt" data-name="date_asc_opt">Date (low to high)</we-button>
        <we-button data-set-order-by="write_date desc" data-select-data-attribute="write_date desc" data-dependencies="search_forums_opt" data-name="date_desc_opt">Date (high to low)</we-button>
    </xpath>
    <xpath expr="//div[@data-js='SearchBar']/div[@data-dependencies='limit_opt']" position="inside">
        <we-checkbox string="Description" data-dependencies="search_forums_opt" data-select-data-attribute="true" data-attribute-name="displayDescription"
            data-apply-to=".search-query"/>
        <we-checkbox string="Date" data-dependencies="search_forums_opt" data-select-data-attribute="true" data-attribute-name="displayDetail"
            data-apply-to=".search-query"/>
    </xpath>
</template>

<template id="snippet_options" inherit_id="website.snippet_options" name="Forum Snippet Options">
    <xpath expr="." position="inside">
        <div data-selector="main:has(#o_wforum_forums_index_list)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Forum Page">
            <we-select string="Layout" data-no-preview="true" data-reload="/">
                <we-button data-customize-website-views="">Grid</we-button>
                <we-button data-customize-website-views="website_forum.opt_list_view">List</we-button>
            </we-select>
            <we-checkbox string="Post Count"
                         class="o_we_sublevel_1"
                         data-customize-website-views="website_forum.opt_post_count"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Last Post"
                         class="o_we_sublevel_1"
                         data-customize-website-views="website_forum.opt_last_post"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
    </xpath>
</template>

</odoo>

```

