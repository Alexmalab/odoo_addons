# Odoo Module: website_blog

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
    'name': 'Blog',
    'category': 'Website/Website',
    'sequence': 200,
    'website': 'https://www.odoo.com/app/blog',
    'summary': 'Publish blog posts, announces, news',
    'version': '1.1',
    'depends': ['website_mail', 'website_partner'],
    'data': [
        'data/mail_message_subtype_data.xml',
        'data/mail_templates.xml',
        'data/website_blog_data.xml',
        'data/blog_snippet_template_data.xml',
        'views/website_blog_views.xml',
        'views/website_blog_components.xml',
        'views/website_blog_posts_loop.xml',
        'views/website_blog_templates.xml',
        'views/snippets/snippets.xml',
        'views/snippets/s_blog_posts.xml',
        'views/website_pages_views.xml',
        'views/blog_post_add.xml',
        'security/ir.model.access.csv',
        'security/website_blog_security.xml',
    ],
    'demo': [
        'data/website_blog_demo.xml'
    ],
    'installable': True,
    'assets': {
        'website.assets_wysiwyg': [
            'website_blog/static/src/js/options.js',
            'website_blog/static/src/snippets/s_blog_posts/options.js',
        ],
        'website.assets_editor': [
            'website_blog/static/src/js/tours/website_blog.js',
            'website_blog/static/src/js/components/*.js',
            'website_blog/static/src/js/systray_items/*.js',
        ],
        'website.backend_assets_all_wysiwyg': [
            'website_blog/static/src/js/wysiwyg_adapter.js',
        ],
        'web.assets_tests': [
            'website_blog/static/tests/**/*',
        ],
        'web.assets_frontend': [
            'website_blog/static/src/scss/website_blog.scss',
            'website_blog/static/src/js/contentshare.js',
            'website_blog/static/src/js/website_blog.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import werkzeug
import itertools
import pytz
import babel.dates
from collections import defaultdict

from odoo import http, fields, tools, models
from odoo.addons.http_routing.models.ir_http import slug, unslug
from odoo.addons.website.controllers.main import QueryURL
from odoo.http import request
from odoo.tools import html2plaintext
from odoo.tools.misc import get_lang
from odoo.tools import sql


class WebsiteBlog(http.Controller):
    _blog_post_per_page = 12  # multiple of 2,3,4
    _post_comment_per_page = 10

    def tags_list(self, tag_ids, current_tag):
        tag_ids = list(tag_ids)  # required to avoid using the same list
        if current_tag in tag_ids:
            tag_ids.remove(current_tag)
        else:
            tag_ids.append(current_tag)
        tag_ids = request.env['blog.tag'].browse(tag_ids)
        return ','.join(slug(tag) for tag in tag_ids)

    def nav_list(self, blog=None):
        dom = blog and [('blog_id', '=', blog.id)] or []
        if not request.env.user.has_group('website.group_website_designer'):
            dom += [('post_date', '<=', fields.Datetime.now())]
        groups = request.env['blog.post']._read_group(
            dom, groupby=['post_date:month'])

        locale = get_lang(request.env).code
        tzinfo = pytz.timezone(request.context.get('tz', 'utc') or 'utc')
        fmt = tools.DEFAULT_SERVER_DATETIME_FORMAT

        res = defaultdict(list)
        for [start] in groups:
            year = babel.dates.format_datetime(start, format='yyyy', tzinfo=tzinfo, locale=locale)
            res[year].append({
                'date_begin': start.strftime(fmt),
                'date_end': (start + models.READ_GROUP_TIME_GRANULARITY['month']).strftime(fmt),
                'month': babel.dates.format_datetime(start, format='MMMM', tzinfo=tzinfo, locale=locale),
                'year': year,
            })
        return res

    def _get_blog_post_search_options(self, blog=None, active_tags=None, date_begin=None, date_end=None, state=None, **post):
        return {
            'displayDescription': True,
            'displayDetail': False,
            'displayExtraDetail': False,
            'displayExtraLink': False,
            'displayImage': False,
            'allowFuzzy': not post.get('noFuzzy'),
            'blog': str(blog.id) if blog else None,
            'tag': ','.join([str(id) for id in active_tags.ids]),
            'date_begin': date_begin,
            'date_end': date_end,
            'state': state,
        }

    def _prepare_blog_values(self, blogs, blog=False, date_begin=False, date_end=False, tags=False, state=False, page=False, search=None, **post):
        """ Prepare all values to display the blogs index page or one specific blog"""
        BlogPost = request.env['blog.post']
        BlogTag = request.env['blog.tag']

        # prepare domain
        domain = request.website.website_domain()

        if blog:
            domain += [('blog_id', '=', blog.id)]

        if date_begin and date_end:
            domain += [("post_date", ">=", date_begin), ("post_date", "<=", date_end)]
        active_tag_ids = tags and [unslug(tag)[1] for tag in tags.split(',')] or []
        active_tags = BlogTag
        if active_tag_ids:
            active_tags = BlogTag.browse(active_tag_ids).exists()
            fixed_tag_slug = ",".join(slug(t) for t in active_tags)
            if fixed_tag_slug != tags:
                path = request.httprequest.full_path
                new_url = path.replace("/tag/%s" % tags, fixed_tag_slug and "/tag/%s" % fixed_tag_slug or "", 1)
                if new_url != path:  # check that really replaced and avoid loop
                    return request.redirect(new_url, 301)
            domain += [('tag_ids', 'in', active_tags.ids)]

        if request.env.user.has_group('website.group_website_designer'):
            count_domain = domain + [("website_published", "=", True), ("post_date", "<=", fields.Datetime.now())]
            published_count = BlogPost.search_count(count_domain)
            unpublished_count = BlogPost.search_count(domain) - published_count

            if state == "published":
                domain += [("website_published", "=", True), ("post_date", "<=", fields.Datetime.now())]
            elif state == "unpublished":
                domain += ['|', ("website_published", "=", False), ("post_date", ">", fields.Datetime.now())]
        else:
            domain += [("post_date", "<=", fields.Datetime.now())]

        use_cover = request.website.is_view_active('website_blog.opt_blog_cover_post')
        fullwidth_cover = request.website.is_view_active('website_blog.opt_blog_cover_post_fullwidth_design')

        # if blog, we show blog title, if use_cover and not fullwidth_cover we need pager + latest always
        offset = (page - 1) * self._blog_post_per_page
        if not blog and use_cover and not fullwidth_cover and not tags and not date_begin and not date_end and not search:
            offset += 1

        options = self._get_blog_post_search_options(
            blog=blog,
            active_tags=active_tags,
            date_begin=date_begin,
            date_end=date_end,
            state=state,
            **post
        )
        total, details, fuzzy_search_term = request.website._search_with_fuzzy("blog_posts_only", search,
            limit=page * self._blog_post_per_page, order="is_published desc, post_date desc, id asc", options=options)
        posts = details[0].get('results', BlogPost)
        first_post = BlogPost
        if posts and not blog and posts[0].website_published:
            first_post = posts[0]
        posts = posts[offset:offset + self._blog_post_per_page]

        url_args = dict()
        if search:
            url_args["search"] = search

        if date_begin and date_end:
            url_args["date_begin"] = date_begin
            url_args["date_end"] = date_end

        pager = tools.lazy(lambda: request.website.pager(
            url=request.httprequest.path.partition('/page/')[0],
            total=total,
            page=page,
            step=self._blog_post_per_page,
            url_args=url_args,
        ))

        if not blogs:
            all_tags = request.env['blog.tag']
        else:
            all_tags = tools.lazy(lambda: blogs.all_tags(join=True) if not blog else blogs.all_tags().get(blog.id, request.env['blog.tag']))
        tag_category = tools.lazy(lambda: sorted(all_tags.mapped('category_id'), key=lambda category: category.name.upper()))
        other_tags = tools.lazy(lambda: sorted(all_tags.filtered(lambda x: not x.category_id), key=lambda tag: tag.name.upper()))
        nav_list = tools.lazy(self.nav_list)
        # for performance prefetch the first post with the others
        post_ids = (first_post | posts).ids
        # and avoid accessing related blogs one by one
        posts.blog_id

        return {
            'date_begin': date_begin,
            'date_end': date_end,
            'first_post': first_post.with_prefetch(post_ids),
            'other_tags': other_tags,
            'tag_category': tag_category,
            'nav_list': nav_list,
            'tags_list': self.tags_list,
            'pager': pager,
            'posts': posts.with_prefetch(post_ids),
            'tag': tags,
            'active_tag_ids': active_tags.ids,
            'domain': domain,
            'state_info': state and {"state": state, "published": published_count, "unpublished": unpublished_count},
            'blogs': blogs,
            'blog': blog,
            'search': fuzzy_search_term or search,
            'search_count': total,
            'original_search': fuzzy_search_term and search,
        }

    @http.route([
        '/blog',
        '/blog/page/<int:page>',
        '/blog/tag/<string:tag>',
        '/blog/tag/<string:tag>/page/<int:page>',
        '''/blog/<model("blog.blog"):blog>''',
        '''/blog/<model("blog.blog"):blog>/page/<int:page>''',
        '''/blog/<model("blog.blog"):blog>/tag/<string:tag>''',
        '''/blog/<model("blog.blog"):blog>/tag/<string:tag>/page/<int:page>''',
    ], type='http', auth="public", website=True, sitemap=True)
    def blog(self, blog=None, tag=None, page=1, search=None, **opt):
        Blog = request.env['blog.blog']
        blogs = tools.lazy(lambda: Blog.search(request.website.website_domain(), order="create_date asc, id asc"))

        if not blog and len(blogs) == 1:
            url = QueryURL('/blog/%s' % slug(blogs[0]), search=search, **opt)()
            return request.redirect(url, code=302)

        date_begin, date_end = opt.get('date_begin'), opt.get('date_end')

        if tag and request.httprequest.method == 'GET':
            # redirect get tag-1,tag-2 -> get tag-1
            tags = tag.split(',')
            if len(tags) > 1:
                url = QueryURL('' if blog else '/blog', ['blog', 'tag'], blog=blog, tag=tags[0], date_begin=date_begin, date_end=date_end, search=search)()
                return request.redirect(url, code=302)

        values = self._prepare_blog_values(blogs=blogs, blog=blog, tags=tag, page=page, search=search, **opt)

        # in case of a redirection need by `_prepare_blog_values` we follow it
        if isinstance(values, werkzeug.wrappers.Response):
            return values

        if blog:
            values['main_object'] = blog
        values['blog_url'] = QueryURL('/blog', ['blog', 'tag'], blog=blog, tag=tag, date_begin=date_begin, date_end=date_end, search=search)

        return request.render("website_blog.blog_post_short", values)

    @http.route(['''/blog/<model("blog.blog"):blog>/feed'''], type='http', auth="public", website=True, sitemap=True)
    def blog_feed(self, blog, limit='15', **kwargs):
        v = {}
        v['blog'] = blog
        v['base_url'] = blog.get_base_url()
        v['posts'] = request.env['blog.post'].search([('blog_id', '=', blog.id)], limit=min(int(limit), 50), order="post_date DESC")
        v['html2plaintext'] = html2plaintext
        r = request.render("website_blog.blog_feed", v, headers=[('Content-Type', 'application/atom+xml')])
        return r

    @http.route([
        '''/blog/<model("blog.blog"):blog>/post/<model("blog.post"):blog_post>''',
    ], type='http', auth="public", website=True, sitemap=False)
    def old_blog_post(self, blog, blog_post, **post):
        # Compatibility pre-v14
        return request.redirect("/blog/%s/%s" % (slug(blog), slug(blog_post)), code=301)

    @http.route([
        '''/blog/<model("blog.blog"):blog>/<model("blog.post", "[('blog_id','=',blog.id)]"):blog_post>''',
    ], type='http', auth="public", website=True, sitemap=True)
    def blog_post(self, blog, blog_post, tag_id=None, page=1, enable_editor=None, **post):
        """ Prepare all values to display the blog.

        :return dict values: values for the templates, containing

         - 'blog_post': browse of the current post
         - 'blog': browse of the current blog
         - 'blogs': list of browse records of blogs
         - 'tag': current tag, if tag_id in parameters
         - 'tags': all tags, for tag-based navigation
         - 'pager': a pager on the comments
         - 'nav_list': a dict [year][month] for archives navigation
         - 'next_post': next blog post, to direct the user towards the next interesting post
        """
        BlogPost = request.env['blog.post']
        date_begin, date_end = post.get('date_begin'), post.get('date_end')

        domain = request.website.website_domain()
        blogs = blog.search(domain, order="create_date, id asc")

        tag = None
        if tag_id:
            tag = request.env['blog.tag'].browse(int(tag_id))
        blog_url = QueryURL('', ['blog', 'tag'], blog=blog_post.blog_id, tag=tag, date_begin=date_begin, date_end=date_end)

        if not blog_post.blog_id.id == blog.id:
            return request.redirect("/blog/%s/%s" % (slug(blog_post.blog_id), slug(blog_post)), code=301)

        tags = request.env['blog.tag'].search([])

        # Find next Post
        blog_post_domain = [('blog_id', '=', blog.id)]
        if not request.env.user.has_group('website.group_website_designer'):
            blog_post_domain += [('post_date', '<=', fields.Datetime.now())]

        all_post = BlogPost.search(blog_post_domain)

        if blog_post not in all_post:
            return request.redirect("/blog/%s" % (slug(blog_post.blog_id)))

        # should always return at least the current post
        all_post_ids = all_post.ids
        current_blog_post_index = all_post_ids.index(blog_post.id)
        nb_posts = len(all_post_ids)
        next_post_id = all_post_ids[(current_blog_post_index + 1) % nb_posts] if nb_posts > 1 else None
        next_post = next_post_id and BlogPost.browse(next_post_id) or False

        values = {
            'tags': tags,
            'tag': tag,
            'blog': blog,
            'blog_post': blog_post,
            'blogs': blogs,
            'main_object': blog_post,
            'nav_list': self.nav_list(blog),
            'enable_editor': enable_editor,
            'next_post': next_post,
            'date': date_begin,
            'blog_url': blog_url,
        }
        response = request.render("website_blog.blog_post_complete", values)

        if blog_post.id not in request.session.get('posts_viewed', []):
            if sql.increment_fields_skiplock(blog_post, 'visits'):
                if not request.session.get('posts_viewed'):
                    request.session['posts_viewed'] = []
                request.session['posts_viewed'].append(blog_post.id)
                request.session.touch()
        return response

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\blog_snippet_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Filters for Dynamic Filter -->
        <record id="dynamic_snippet_latest_blog_post_filter" model="ir.filters">
            <field name="name">Latest Blog Posts</field>
            <field name="model_id">blog.post</field>
            <field name="user_id" eval="False" />
            <field name="domain">[('post_date', '&lt;=', context_today())]</field>
            <field name="sort">["post_date desc"]</field>
            <field name="action_id" ref="website.action_website"/>
        </record>
        <record id="dynamic_snippet_most_viewed_blog_post_filter" model="ir.filters">
            <field name="name">Most Viewed Blog Posts</field>
            <field name="model_id">blog.post</field>
            <field name="user_id" eval="False" />
            <field name="domain">[('post_date', '&lt;=', context_today()), ('visits', '!=', False)]</field>
            <field name="sort">["visits desc"]</field>
            <field name="action_id" ref="website.action_website"/>
        </record>
        <!-- Dynamic Filter -->
        <record id="dynamic_filter_latest_blog_posts" model="website.snippet.filter">
            <field name="name">Latest Blog Posts</field>
            <field name="filter_id" ref="website_blog.dynamic_snippet_latest_blog_post_filter"/>
            <field name="field_names">name,teaser,subtitle</field>
            <field name="limit" eval="16"/>
        </record>
        <record id="dynamic_filter_most_viewed_blog_posts" model="website.snippet.filter">
            <field name="name">Most Viewed Blog Posts</field>
            <field name="filter_id" ref="website_blog.dynamic_snippet_most_viewed_blog_post_filter"/>
            <field name="field_names">name,teaser,subtitle</field>
            <field name="limit" eval="16"/>
        </record>
    </data>
</odoo>

```

## File: data\ir_asset.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

            <record id="website_blog.s_latest_posts_000_scss" model="ir.asset">
                <field name="name">Latest posts 000 SCSS</field>
                <field name="bundle">web.assets_frontend</field>
                <field name="path">website_blog/static/src/snippets/s_latest_posts/000.scss</field>
                <field name="active" eval="False"/>
                </record>

    </data>
</odoo>

```

## File: data\mail_message_subtype_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo><data noupdate="1">

    <!-- Blog-related subtypes for messaging / Chatter -->
    <record id="mt_blog_blog_published" model="mail.message.subtype">
        <field name="name">Published Post</field>
        <field name="res_model">blog.blog</field>
        <field name="default" eval="True"/>
        <field name="description">Published Post</field>
    </record>

</data></odoo>

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <template id="blog_post_template_new_post">
        <p>A new post <t t-esc="post.name" /> has been published on the <t t-esc="object.name" /> blog. Click here to access the blog :</p>
        <p style="margin-left: 30px; margin-top: 10 px; margin-bottom: 10px;">
            <a t-attf-href="/blog/#{slug(object)}/#{slug(post)}"
                style="padding: 5px 10px; font-size: 12px; line-height: 18px; color: #FFFFFF; border-color:#875A7B; text-decoration: none; display: inline-block; margin-bottom: 0px; font-weight: 400; text-align: center; vertical-align: middle; cursor: pointer;background-color: #875A7B; border: 1px solid #875A7B; border-radius:3px">
                Access post
            </a>
        </p>
    </template>
</data></odoo>

```

## File: data\website_blog_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data noupdate="1">
        <record id="blog_blog_1" model="blog.blog">
            <field name="name">Our blog</field>
            <field name="subtitle">We are a team of passionate people whose goal is to improve everyone's life.</field>
            <field name="cover_properties">{"background-image": "url('/website_blog/static/src/img/cover_5.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
        </record>

        <record id="menu_blog" model="website.menu">
            <field name="name">Blog</field>
            <field name="url">/blog</field>
            <field name="parent_id" ref="website.main_menu"/>
            <field name="sequence" type="int">40</field>
        </record>

        <!-- Blog-related subtypes for messaging / Chatter -->
        <record id="mt_blog_blog_published" model="mail.message.subtype">
            <field name="name">Published Post</field>
            <field name="res_model">blog.blog</field>
            <field name="default" eval="True"/>
            <field name="description">Published Post</field>
        </record>

    </data>

    <data>

        <!-- jump to blog at install -->
        <record id="action_open_website" model="ir.actions.act_url">
            <field name="name">Website Blogs</field>
            <field name="target">self</field>
            <field name="url" eval="'/blog/'"/>
        </record>
        <record id="base.open_menu" model="ir.actions.todo">
            <field name="action_id" ref="action_open_website"/>
            <field name="state">open</field>
        </record>

    </data>
</odoo>

```

## File: data\website_blog_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="blog_blog_1" model="blog.blog">
            <field name="name">Travel</field>
            <field name="subtitle">Holiday tips</field>
            <field name="cover_properties">{"background-image": "url('/website_blog/static/src/img/blog_1.jpeg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
        </record>
        <record id="blog_blog_2" model="blog.blog">
            <field name="name">Astronomy</field>
            <field name="subtitle">Astronomy is “stargazing"</field>
            <field name="cover_properties">{"background-image": "url('/website_blog/static/src/img/blog_2.jpeg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
        </record>

        <!-- TAGS -->
        <record id="blog_tag_1" model="blog.tag">
            <field name="name">hotels</field>
        </record>
        <record id="blog_tag_2" model="blog.tag">
            <field name="name">adventure</field>
        </record>
        <record id="blog_tag_3" model="blog.tag">
            <field name="name">guides</field>
        </record>
        <record id="blog_tag_4" model="blog.tag">
            <field name="name">telescopes</field>
        </record>
        <record id="blog_tag_5" model="blog.tag">
            <field name="name">discovery</field>
        </record>

        <!-- POSTS -->
        <record id="blog_post_1" model="blog.post">
            <field name="name">Sierra Tarahumara</field>
            <field name="subtitle">An exciting mix of relaxation, culture, history, wildlife and hiking.</field>
            <field name="blog_id" ref="blog_blog_1"/>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="is_published" eval="True"/>
            <field name="published_date" eval="time.strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="tag_ids" eval="[(6, 0, [ref('blog_tag_1'), ref('blog_tag_2')])]"/>
            <field name="cover_properties">{"background-image": "url('/website_blog/static/src/img/cover_1.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0"}</field>
            <field name="content"><![CDATA[
<p class="lead">Sierra Tarahumara, popularly known as Copper Canyon is situated in Mexico. The area is a favorite destination among those seeking an adventurous vacation.</p>
<p>Copper Canyon is one of the six gorges in the area. Although the name suggests that the gorge might have some relevance to copper mining, this is not the case. The name is derived from the copper and green lichen covering the canyon. Copper Canyon has two climatic zones. The region features an alpine climate at the top and a subtropical climate at the lower levels. Winters are cold with frequent snowstorms at the higher altitudes. Summers are dry and hot. The capital city, Chihuahua, is a high altitude desert where weather ranges from cold winters to hot summers. The region is unique because of the various ecosystems that exist within it.</p>

<p>Another unique feature of Copper Canyon is the presence of the Tarahumara Indian culture. These semi-nomadic people live in cave dwellings. Their livelihood chiefly depends on farming and cattle ranching.</p>
<figure class="mt-2 mb-4">
    <img src="/website_blog/static/src/img/content_1_1.jpg" class="img-fluid w-100"/>
    <figcaption class="figure-caption text-muted">Photo by PoloX Hernandez, @elpolox</figcaption>
</figure>

<blockquote class="blockquote my-5">
    <em class="h4 my-0">Apart from the native population, the local wildlife is also a major crowd puller.</em>
    <footer class="blockquote-footer text-muted">Someone famous in <cite title="Source Title">Source Title</cite></footer>
</blockquote>

<p>Several migratory and native birds, mammals and reptiles call Copper Canyon their home. The exquisite fauna in this near-pristine land is also worth checking out.</p>
<figure class="mt-2 mb-4">
    <img src="/website_blog/static/src/img/content_1_2.jpg" class="img-fluid w-100"/>
    <figcaption class="figure-caption text-muted">Photo by Boris Smokrovic, @borisworkshop</figcaption>
</figure>

<p>A traveler may choose to explore the area by hiking around the canyon or venturing into it. Detailed planning is required for those who wish to venture into the depths of the canyon. There are a number of travel companies that specialize in organizing tours to the region. Visitors can fly to Copper Canyon using a tourist visa, which is valid for 180 days. Travelers can also drive from anywhere in the United States and acquire a visa at the Mexican customs station at the border.</p>
<p>A holiday to the Copper Canyon promises to be an exciting mix of relaxation, culture, history, wildlife and hiking.</p>
]]>
            </field>
        </record>

        <record id="blog_post_2" model="blog.post">
            <field name="name">Maui helicopter tours</field>
            <field name="subtitle">A great way to discover hidden places</field>
            <field name="blog_id" ref="blog_blog_1"/>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="is_published" eval="True"/>
            <field name="published_date" eval="(datetime.now()-relativedelta(days=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="tag_ids" eval="[(6, 0, [ref('blog_tag_2')])]"/>
            <field name="visits">246</field>
            <field name="cover_properties">{"background-image": "url('/website_blog/static/src/img/cover_2.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
            <field name="content"><![CDATA[
<section class="container">
    <div class="row">
        <div class="col">
            <p class="lead">Maui helicopter tours are a great way to see the island from a different perspective and have a fun adventure. If you have never been on a helicopter before, this is a great place to do it.</p>
            <p>You will see all the beauty that Maui has to offer and can have a great time for the entire family. Tours are not too expensive and last from forty five minutes to over an hour. You can see places that are typically inaccessible with Maui helicopter tours. Places that are not available by foot or vehicle can be seen by air. Breathtaking sights await those who are up for some fun Maui helicopter tours. If you will be staying on the island for a considerable amount of time, you may want to think about doing multiple Maui helicopter tours.</p>
        </div>
        <div class="col-12 col-md-auto">
            <figure>
                <img src="/website_blog/static/src/img/content_2_1.jpg" class="img-fluid" style="max-height:450px"/>
                <figcaption class="figure-caption text-muted">Photo by Jon Ly, @jonatron</figcaption>
            </figure>
        </div>
    </div>
</section>

<h2 class="mt-5">East Maui</h2>
<figure class="mt-2 mb-4">
    <img src="/website_blog/static/src/img/content_2_2.jpg" class="img-fluid w-100"/>
    <figcaption class="figure-caption text-muted">Photo by Anton Repponen, @repponen</figcaption>
</figure>

<p>East Maui helicopter tours will give you a view of the ten thousand foot volcano, Haleakala or House of the sun. This volcano is dormant and last erupted in 1790. You will be able to see the crater of the volcano and the dry, arid earth surrounding the south side of the volcano’s slop with Maui helicopter tours.</p>
<p>The view of this is truly breathtaking and is a sight not to be missed. It is also highly educational with a chance to see a dormant volcano up close, something that can not be seen every day. On the northern and southern sides of the volcano, you will see an incredible different view however. These sides are lush and green and you will be able to see some beautiful waterfalls and gorgeous brush. Tropical rainforests abound on this side of the island and it is something that is not easily accessible by any other means than by air.</p>
<p>Maui helicopter tours will allow you to see all of these sights. Make sure to take a camera or video with you when going on Maui helicopter tours to capture the beauty of the scenery and to show friends and family at home all the wonderful things you saw while on vacation.</p>

<h2 class="mt-5">Molokai Maui </h2>
<figure class="mt-2 mb-4">
    <img src="/website_blog/static/src/img/content_2_3.jpg" class="img-fluid w-100"/>
    <figcaption class="figure-caption text-muted">Photo by Denys Nevozhai, @dnevozhai</figcaption>
</figure>
<p>Molokai Maui helicopter tours will take you to a different island but one that is only nine miles away and easily accessible by air. This island has a very small population with a different culture and scenery. The entire coast of the northeast is lined with cliffs and remote beaches. They are completely inaccessible by any other means of transportation than air.
People who live on the island have never even seen this remarkable scenery unless they have taken Maui helicopter tours to view it. When the weather has been rainy and there is a lot of rainfall for he season you will see many astounding waterfalls.</p>
<p>The cliffs in this region are among the highest in the world and to see water cascading from the high peaks is simply breathtaking. The short jaunt from Maui with Maui helicopter tours is well worth seeing the beauty of this natural environment.</p>
<p>Maui helicopter tours are a great way to tour those places that can not be reached on foot or by car. The tours last approximately one hour and range from approximately one hundred eight five dollars to two hundred forty dollars person. For many, this is a once in a lifetime opportunity to see natural scenery that will not be available again. Taking cameras and videos to capture the moments will also allow you to relive the tour again and again as you reminisce throughout the years.</p>
]]>
            </field>
        </record>

        <record id="blog_post_3" model="blog.post">
            <field name="name">How to choose the right hotel</field>
            <field name="subtitle">Facts you should bear in mind.</field>
            <field name="blog_id" ref="blog_blog_1"/>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="tag_ids" eval="[(6, 0, [ref('blog_tag_1')])]"/>
            <field name="visits">467</field>
            <field name="is_published" eval="True"/>
            <field name="published_date" eval="(datetime.now()-relativedelta(days=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="cover_properties">{"background-image": "url('/website_blog/static/src/img/cover_3.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
            <field name="content"><![CDATA[
<p class="lead">So you’re going abroad, you’ve chosen your destination and now you have to choose a hotel.</p>
<p>Ten years ago, you’d have probably visited your local travel agent and trusted the face-to-face advice you were given by the so called ‘experts’. The 21st Century way to select and book your hotel is of course on the Internet, by using travel websites.</p>

<p>But how do you sift through the amazing choices on offer? And more importantly, do you really trust the photographs and descriptions of the hotels that they have awarded themselves with the motivation of getting bookings? Traveler reviews can be helpful, but you need to exercise caution. They are often biased, sometimes out of date, and may not serve your interests at all. How do you know that the features that are important to the reviewer are important to you?</p>

<blockquote>
    <em class="h4 my-0">The more reviews you read, the more you notice how they tend to cluster at the extremes of opinion.</em>
    <footer class="blockquote-footer text-muted">Someone famous in <cite title="Source Title">Source Title</cite></footer>
</blockquote>

<figure class="mt-2 mb-4">
    <img src="/website_blog/static/src/img/content_3_1.jpg" class="img-fluid w-100"/>
    <figcaption class="figure-caption text-muted">Photo by Jason Briscoe, @jbriscoe</figcaption>
</figure>

<p>Then there’s the problem of the reviewer’s motivation. The more reviews you read, the more you notice how they tend to cluster at the extremes of opinion. On one end, you have angry reviewers with axes to grind; at the other, you have delighted guests who lavish praise beyond belief. You’ll not be surprised to learn that hotels sometimes post their own glowing reviews, or that competitor’s line up for the chance to lambaste the competition with bad reviews. It makes sense to consider what is really important to you when selecting a hotel. You should then choose an online hotel directory that gives up-to-date, independent, impartial information that really matters.
</p>

<h4 class="mt-4">Here are some of the key facts you should bear in mind:</h4>
<ol>
    <li class="mb-3"><h5>Location</h5>
    If it matters that your hotel is, for example, on the beach, close to the theme park, or convenient for the airport, then location is paramount. Any decent directory should offer a location map of the hotel and its surroundings. There should be distance charts to the airport offered as well as some form of interactive map.</li>
    <li class="mb-3"><h5>Style</h5>
    It is important to choose a hotel that makes you feel comfortable – contemporary or traditional furnishings, local decor or international, formal or relaxed. The ideal hotel directory should let you know of the options available.</li>
    <li class="mb-3"><h5>Restaurants, Cafes and Bars</h5>
    Local color is great but the hotel’s own restaurants and bars can play an important part in your stay. You should be aware of choice, style and whether or not they are smart or informal. A good hotel report should tell you this, and particularly about breakfast facilities.</li>
    <li class="mb-3"><h5>Bedroom Facilities</h5>
    You should always carefully consider the type of facilities you need from your bedroom and find the hotel that has those you consider important. The hotel directory website should elaborate on matters such as: bed size, Internet Access (its cost, whether there is WIFI or wired broadband connection), Complimentary amenities, views from the room and luxury offerings like a Pillow menu or Bath menu, choice of smoking or non smoking rooms etc.</li>
</ol>
<p>These things really do matter and any decent hotel directory should give you this sort of advice on bedrooms – not just the number of rooms which is the usual option!</p>
<ol>
    <li class="mb-3"><h5>Children’s’ Facilities</h5>
    More important to the family traveler than the business traveler, you should find out just how child friendly the hotel is from the directory and make your decision from there. One thing worth looking for is whether the hotel offers a baby sitters service. For the business traveler wishing to escape children this is of course very relevant too – perhaps a hotel that is not child friendly would be something more appropriate!</li>
    <li class="mb-3"> <h5>Leisure Facilities</h5>
    The site should offer a detailed analysis of leisure services within the hotel – spa, pool, gym, sauna – as well as details of any other facilities nearby such as golf courses. 7. Special Needs: the hotel directory site should advise the visitor of each hotel’s special needs services and accessibility policy. Whilst again this does not apply to every visitor, it is absolutely vital to some.</li>
</ol>

<p>Finally and most importantly, the quality hotel directory inspection team should have visited the hotel in question on a regular basis, met the staff, slept in a bedroom and tried the food. They should experience the hotel as only a hotel guest can and it is only then that they are really in a strong position to write about the hotel.</p>
]]>
            </field>
        </record>

        <record id="blog_post_4" model="blog.post">
            <field name="name">How To Look Up</field>
            <field name="subtitle">Be aware of this thing called “astronomy”</field>
            <field name="blog_id" ref="blog_blog_2"/>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="tag_ids" eval="[(6, 0, [ref('blog_tag_5')])]"/>
            <field name="visits">453</field>
            <field name="is_published" eval="True"/>
            <field name="published_date" eval="(datetime.now()-relativedelta(days=6)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="cover_properties">{"background-image": "url('/website_blog/static/src/img/cover_4.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "text_align_class": "text-center", "opacity": "0.2"}</field>
            <field name="content"><![CDATA[
<p class="lead">It is safe to say that at some point on our lives, each and every one of us has that moment when we are suddenly stunned when we come face to face with the enormity of the universe that we see in the night sky.</p>

<p>For many of us who are city dwellers, we don’t really notice that sky up there on a routine basis. The lights of the city do a good job of disguising the amazing display that is above all of our heads all of the time.</p>

<blockquote class="blockquote my-5">
    <em class="h4 my-0">That “Wow” moment is what astrology is all about.</em>
    <footer class="blockquote-footer text-muted">Someone famous in <cite title="Source Title">Source Title</cite></footer>
</blockquote>

<p>So it might be that once a year vacation to a camping spot or a trip to a relative’s house out in the country that we find ourselves outside when the spender of the night sky suddenly decides to put on it’s spectacular show. If you have had that kind of moment when you were literally struck breathless by the spender the night sky can show to us, you can probably remember that exact moment when you could say little else but “wow” at what you saw.</p>

<figure class="mt-2 mb-4">
    <img src="/website_blog/static/src/img/content_4_1.jpg" class="img-fluid w-100"/>
    <figcaption class="figure-caption text-muted">Photo by Arto Marttinen, @wandervisions</figcaption>
</figure>

<p>That “Wow” moment is what astrology is all about. For some, that wow moment becomes a passion that leads to a career studying the stars. For a lucky few, that wow moment because an all consuming obsession that leads to them traveling to the stars in the space shuttle or on one of our early space missions. But for most of us astrology may become a pastime or a regular hobby. But we carry that wow moment with us for the rest of our lives and begin looking for ways to look deeper and learn more about the spectacular universe we see in the millions of stars above us each night.</p>

<h4>Get started</h4>
<p>To get started in learning how to observe the stars much better, there are some basic things we might need to look deeper, beyond just what we can see with the naked eye and begin to study the stars as well as enjoy them. The first thing you need isn’t equipment at all but literature. A good star map will show you the major constellations, the location of the key stars we use to navigate the sky and the planets that will appear larger than stars. And if you add to that map some well done introductory materials into the hobby of astronomy, you are well on your way.</p>

<h4>Get a telescope</h4>
<p>The next thing we naturally want to get is a good telescope. You may have seen a hobbyist who is well along in their study setting up those really cool looking telescopes on a hill somewhere. That excites the amateur astronomer in you because that must be the logical next step in the growth of your hobby. But how to buy a good telescope can be downright confusing and intimidating.</p>

<p>Before you go to that big expense, it might be a better next step from the naked eye to invest in a good set of binoculars. There are even binoculars that are suited for star gazing that will do just as good a job at giving you that extra vision you want to see just a little better the wonders of the universe. A well designed set of binoculars also gives you much more mobility and ability to keep your “enhanced vision” at your fingertips when that amazing view just presents itself to you.</p>

<p>None of this precludes you from moving forward with your plans to put together an awesome telescope system. Just be sure you get quality advice and training on how to configure your telescope to meet your needs. Using these guidelines, you will enjoy hours of enjoyment stargazing at the phenomenal sights in the night sky that are beyond the naked eye.</p>
]]>
            </field>
        </record>

        <record id="blog_post_5" model="blog.post">
            <field name="name">What If They Let You Run The Hubble</field>
            <field name="subtitle">The beauty of astronomy is that anybody can do it.</field>
            <field name="blog_id" ref="blog_blog_2"/>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="tag_ids" eval="[(6, 0, [ref('blog_tag_4'), ref('blog_tag_5')])]"/>
            <field name="is_published" eval="True"/>
            <field name="published_date" eval="(datetime.now()-relativedelta(days=4)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="cover_properties">{"background-image": "url('/website_blog/static/src/img/cover_5.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "text_align_class": "text-center", "opacity": "0.2"}</field>
            <field name="content"><![CDATA[
<p class="lead">From the tiniest baby to the most advanced astrophysicist, there is something for anyone who wants to enjoy astronomy. In fact, it is a science that is so accessible that virtually anybody can do it virtually anywhere they are. All they have to know how to do is to look up.</p>
<p>It really is amazing when you think about it that just by looking up on any given night, you could see virtually hundreds of thousands of stars, star systems, planets, moons, asteroids, comets and maybe a even an occasional space shuttle might wander by. It is even more breathtaking when you realize that the sky you are looking up at is for all intents and purposes the exact same sky that our ancestors hundreds and thousands of years ago enjoyed when they just looked up.</p>

<blockquote class="blockquote my-5">
    <em class="h4 my-0">There is something timeless about the cosmos.</em>
    <footer class="blockquote-footer text-muted">Someone famous in <cite title="Source Title">Source Title</cite></footer>
</blockquote>

<p>There is something timeless about the cosmos. The fact that the planets and the moon and the stars beyond them have been there for ages does something to our sense of our place in the universe. In fact, many of the stars we “see” with our naked eye are actually light that came from that star hundreds of thousands of years ago. That light is just now reaching the earth. So in a very real way, looking up is like time travel.</p>

<figure class="mt-2 mb-4">
    <img src="/website_blog/static/src/img/content_5_1.jpg" class="img-fluid w-100"/>
    <figcaption class="figure-caption text-muted">Photo by SpaceX, @spacex</figcaption>
</figure>

<p>While anyone can look up and fall in love with the stars at any time, the fun of astronomy is learning how to become more and more skilled and equipped in star gazing that you see and understand more and more each time you look up. Here are some steps you can take to make the moments you can devote to your hobby of astronomy much more enjoyable.</p>

<h4>Get some history</h4>
<p>Learning the background to the great discoveries in astronomy will make your moments star gazing more meaningful. It is one of the oldest sciences on earth so find out the greats of history who have looked at these stars before you.</p>

<h4>Know what you are looking at</h4>
<p>It is great fun to start learning the constellations, how to navigate the night sky and find the planets and the famous stars. There are web sites and books galore to guide you.</p>

<h4>Get a geek</h4>
<p>Astronomy clubs are lively places full of knowledgeable amateurs who love to share their knowledge with you. For the price of a coke and snacks, they will go star gazing with you and overwhelm you with trivia and great knowledge.</p>

<h4>Know when to look</h4>
<p> Not only knowing the weather will make sure your star gazing is rewarding but if you learn when the big meteor showers and other big astronomy events will happen will make the excitement of astronomy come alive for you.</p>

<p>And when all is said and done,<b> get equipped</b>. Your quest for newer and better telescopes will be a lifelong one. Let yourself get addicted to astronomy and the experience will enrich every aspect of life. It will be an addiction you never want to break.</p>
]]>
            </field>
        </record>

        <record id="blog_post_6" model="blog.post">
            <field name="name">Buying A Telescope</field>
            <field name="subtitle">Before you make your first purchase…</field>
            <field name="blog_id" ref="blog_blog_2"/>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="tag_ids" eval="[(6, 0, [ref('blog_tag_3'), ref('blog_tag_4')])]"/>
            <field name="is_published" eval="True"/>
            <field name="published_date" eval="(datetime.now()-relativedelta(days=3)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="cover_properties">{"background-image": "url('/website_blog/static/src/img/cover_6.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "text_align_class": "text-center", "opacity": "0.2"}</field>
            <field name="content"><![CDATA[
<p class="lead">Buying the right telescope to take your love of astronomy to the next level is a big next step in the development of your passion for the stars.</p>
<p>In many ways, it is a big step from someone who is just fooling around with astronomy to a serious student of the science. But you and I both know that there is still another big step after buying a telescope before you really know how to use it.</p>
<p>So it is critically important that you get just the right telescope for where you are and what your star gazing preferences are. To start with, let’s discuss the three major kinds of telescopes and then lay down some “Telescope 101″ concepts to increase your chances that you will buy the right thing.</p>

<blockquote class="blockquote my-5">
    <em class="h4 my-0">It is critically important that you get just the right telescope.</em>
    <footer class="blockquote-footer text-muted">Someone famous in <cite title="Source Title">Source Title</cite></footer>
</blockquote>

<p>So to select just the right kind of telescope, your objectives in using the telescope are important. To really understand the strengths and weaknesses not only of the lenses and telescope design but also in how the telescope performs in various star gazing situations, it is best to do some homework up front and get exposure to the different kinds. So before you make your first purchase…</p>
<ul>
    <li>Above all, <b>establish a relationship with a reputable telescope shop</b> that employs people who know their stuff. If you buy your telescope at a Wal-Mart or department store, the odds you will get the right thing are remote.</li>
    <li><b>Pick the brains of the experts</b>. If you are not already active in an astronomy society or club, the sales people at the telescope store will be able to guide you to the active societies in your area. Once you have connections with people who have bought telescopes, you can get advice about what works and what to avoid that is more valid than anything you will get from a web article or a salesperson at Wal-Mart.</li>
    <li><b>Try before you buy.</b> This is another advantage of going on some field trips with the astronomy club. You can set aside some quality hours with people who know telescopes and have their rigs set up to examine their equipment, learn the key technical aspects, and try them out before you sink money in your own set up.</li>
    <li><b>Binoculars are lightweight and portable.</b> Unless you have the luxury to set up and operate an observatory from your deck, you are probably going to travel to perform your viewings. Binoculars go with you much easier and they are more lightweight to carry to the country and use while you are there than a cumbersome telescope set up kit.</li>
</ul>

<figure class="mt-2 mb-4">
    <img src="/website_blog/static/src/img/content_6_1.jpg" class="img-fluid w-100"/>
    <figcaption class="figure-caption text-muted">Photo by Teddy Kelley, @teddykelley</figcaption>
</figure>

<p>There are other considerations to factor into your final purchase decision.</p>

<h4>How mobile must your telescope be?</h4>
<p>The tripod or other accessory decisions will change significantly with a telescope that will live on your deck versus one that you plan to take to many remote locations.</p>
<h4>Along those lines, how difficult is the set up and break down?</h4>
<p>How complex is the telescope and will you have trouble with maintenance? Network to get the answers to these and other questions. If you do your homework like this, you will find just the right telescope for this next big step in the evolution of your passion for astronomy.</p>
]]>
            </field>
        </record>

        <record id="blog_post_7" model="blog.post">
            <field name="name">Beyond The Eye</field>
            <field name="subtitle">Becoming part of the society of devoted amateur astronomers.</field>
            <field name="blog_id" ref="blog_blog_2"/>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="tag_ids" eval="[(6, 0, [ref('blog_tag_5')])]"/>
            <field name="is_published" eval="True"/>
            <field name="published_date" eval="(datetime.now()-relativedelta(days=7)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="cover_properties">{"background-image": "url('/website_blog/static/src/img/cover_7.jpg')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0.4"}</field>
            <field name="content"><![CDATA[
<p class="lead">For many of us, our very first experience of learning about the celestial bodies begins when we saw our first full moon in the sky. It is truly a magnificent view even to the naked eye.</p>
<p>If the night is clear, you can see amazing detail of the lunar surface just star gazing on in your back yard.
Naturally, as you grow in your love of astronomy, you will find many celestial bodies fascinating. But the moon may always be our first love because is the one far away space object that has the unique distinction of flying close to the earth and upon which man has walked.</p>

<blockquote class="blockquote my-5">
    <em class="h4 my-0">Your study of the moon, like anything else, can go from the simple to the very complex.</em>
    <footer class="blockquote-footer text-muted">Someone famous in <cite title="Source Title">Source Title</cite></footer>
</blockquote>

<p>To gaze at the moon with the naked eye, making yourself familiar with the lunar map will help you pick out the seas, craters and other geographic phenomenon that others have already mapped to make your study more enjoyable. Moon maps can be had from any astronomy shop or online and they are well worth the investment.</p>

<h2>The best time to view the moon.</h2>
<p>The best time to view the moon, obviously, is at night when there are few clouds and the weather is accommodating for a long and lasting study. The first quarter yields the greatest detail of study. And don’t be fooled but the blotting out of part of the moon when it is not in full moon stage. The phenomenon known as “earthshine” gives you the ability to see the darkened part of the moon with some detail as well, even if the moon is only at quarter or half display.</p>

<figure class="mt-2 mb-4">
    <img src="/website_blog/static/src/img/content_7_1.jpg" class="img-fluid w-100"/>
    <figcaption class="figure-caption text-muted">Photo by Patrick Brinksma, @patrickbrinksma</figcaption>
</figure>

<p>To kick it up a notch, a good pair of binoculars can do wonders for the detail you will see on the lunar surface. For best results, get a good wide field in the binocular settings so you can take in the lunar landscape in all its beauty. And because it is almost impossible to hold the binoculars still for the length of time you will want to gaze at this magnificent body in space, you may want to add to your equipment arsenal a good tripod that you can affix the binoculars to so you can study the moon in comfort and with a stable viewing platform.</p>
<p>Of course, to take your moon worship to the ultimate, stepping your equipment up to a good starter telescope will give you the most stunning detail of the lunar surface. With each of these upgrades your knowledge and the depth and scope of what you will be able to see will improve geometrically. For many amateur astronomers, we sometimes cannot get enough of what we can see on this our closest space object. </p>

<figure class="mt-2 mb-4">
    <img src="/website_blog/static/src/img/content_7_2.jpg" class="img-fluid w-100"/>
    <figcaption class="figure-caption text-muted">Photo by Greg Rakozy, @grakozy</figcaption>
</figure>

<p>To take it to a natural next level, you may want to take advantage of partnerships with other astronomers or by visiting one of the truly great telescopes that have been set up by professionals who have invested in better techniques for eliminating atmospheric interference to see the moon even better. The internet can give you access to the Hubble and many of the huge telescopes that are pointed at the moon all the time. Further, many astronomy clubs are working on ways to combine multiple telescopes, carefully synchronized with computers for the best view of the lunar landscape.</p>

<p>Becoming part of the society of devoted amateur astronomers will give you access to these organized efforts to reach new levels in our ability to study the Earth’s moon. And it will give you peers and friends who share your passion for astronomy and who can share their experience and areas of expertise as you seek to find where you might look next in the huge night sky, at the moon and beyond it in your quest for knowledge about the seemingly endless universe above us. </p>
]]>
            </field>
        </record>

        <record id="blog_comment_1" model="mail.message">
            <field name="body">Beautiful! I plan to go there next holidays.</field>
            <field name="model">blog.post</field>
            <field name="res_id" ref="blog_post_1"/>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_demo_portal"/>
            <field name="subtype_id" ref="mail.mt_comment"/>
        </record>

        <record id="blog_comment_2" model="mail.message">
            <field name="body">Hi! How long did you stay there?</field>
            <field name="model">blog.post</field>
            <field name="res_id" ref="blog_post_1"/>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="subtype_id" ref="mail.mt_comment"/>
        </record>

        <record id="blog_comment_3" model="mail.message">
            <field name="body">I'll follow your instructions next time! It will save myself from a weird place like last time :D</field>
            <field name="model">blog.post</field>
            <field name="res_id" ref="blog_post_3"/>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_admin"/>
            <field name="subtype_id" ref="mail.mt_comment"/>
        </record>

        <record id="blog_comment_4" model="mail.message">
            <field name="body">Can't wait to buy a telescope!</field>
            <field name="model">blog.post</field>
            <field name="res_id" ref="blog_post_5"/>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="subtype_id" ref="mail.mt_comment"/>
        </record>

        <record id="blog_comment_5" model="mail.message">
            <field name="body">Light pollution can be really annoying when you want to observe the sky. The best places are far from cities. That's why I like to go camping.</field>
            <field name="model">blog.post</field>
            <field name="res_id" ref="blog_post_5"/>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_demo_portal"/>
            <field name="subtype_id" ref="mail.mt_comment"/>
        </record>

        <record id="blog_comment_6" model="mail.message">
            <field name="body">Great article! Do you have any good addresses to buy a telescope? Thanks!</field>
            <field name="model">blog.post</field>
            <field name="res_id" ref="blog_post_6"/>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="subtype_id" ref="mail.mt_comment"/>
        </record>

        <record id="blog_comment_7" model="mail.message">
            <field name="body">Great article. I learned so much about astronomy and the moon. How can I contact you to discuss about my experience?</field>
            <field name="model">blog.post</field>
            <field name="res_id" ref="blog_post_7"/>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="subtype_id" ref="mail.mt_comment"/>
        </record>
    </data>
</odoo>

```

## File: models\ir_qweb_fields.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _


class Field(models.AbstractModel):
    _inherit = 'ir.qweb.field'

    @api.model
    def attributes(self, record, field_name, options, values):
        attrs = super().attributes(record, field_name, options, values)

        if field_name == 'teaser' and self.env.context.get('edit_translations'):
            attrs['data-translate-error-tooltip'] = _("On your default language, empty the blog post description and save to get an automated (translated) summary.")

        return attrs

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
        suggested_controllers.append((_('Blog'), url_for('/blog'), 'website_blog'))
        return suggested_controllers

    def configurator_set_menu_links(self, menu_company, module_data):
        blogs = module_data.get('#blog', [])
        for idx, blog in enumerate(blogs):
            new_blog = self.env['blog.blog'].create({
                'name': blog['name'],
                'website_id': self.id,
            })
            blog_menu_values = {
                'name': blog['name'],
                'url': '/blog/%s' % new_blog.id,
                'sequence': blog['sequence'],
                'parent_id': menu_company.id if menu_company else self.menu_id.id,
                'website_id': self.id,
            }
            if idx == 0:
                blog_menu = self.env['website.menu'].search([('url', '=', '/blog'), ('website_id', '=', self.id)])
                blog_menu.write(blog_menu_values)
            else:
                self.env['website.menu'].create(blog_menu_values)
        super().configurator_set_menu_links(menu_company, module_data)

    def _search_get_details(self, search_type, order, options):
        result = super()._search_get_details(search_type, order, options)
        if search_type in ['blogs', 'blogs_only', 'all']:
            result.append(self.env['blog.blog']._search_get_detail(self, order, options))
        if search_type in ['blogs', 'blog_posts_only', 'all']:
            result.append(self.env['blog.post']._search_get_detail(self, order, options))
        return result

```

## File: models\website_blog.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime
import random

from odoo import api, models, fields, _
from odoo.addons.http_routing.models.ir_http import slug, unslug
from odoo.addons.website.tools import text_from_html
from odoo.tools.json import scriptsafe as json_scriptsafe
from odoo.tools.translate import html_translate


class Blog(models.Model):
    _name = 'blog.blog'
    _description = 'Blog'
    _inherit = [
        'mail.thread',
        'website.seo.metadata',
        'website.multi.mixin',
        'website.cover_properties.mixin',
        'website.searchable.mixin',
    ]
    _order = 'name'

    name = fields.Char('Blog Name', required=True, translate=True)
    subtitle = fields.Char('Blog Subtitle', translate=True)
    active = fields.Boolean('Active', default=True)
    content = fields.Html('Content', translate=html_translate, sanitize=False)
    blog_post_ids = fields.One2many('blog.post', 'blog_id', 'Blog Posts')
    blog_post_count = fields.Integer("Posts", compute='_compute_blog_post_count')

    @api.depends('blog_post_ids')
    def _compute_blog_post_count(self):
        for record in self:
            record.blog_post_count = len(record.blog_post_ids)

    def write(self, vals):
        res = super(Blog, self).write(vals)
        if 'active' in vals:
            # archiving/unarchiving a blog does it on its posts, too
            post_ids = self.env['blog.post'].with_context(active_test=False).search([
                ('blog_id', 'in', self.ids)
            ])
            for blog_post in post_ids:
                blog_post.active = vals['active']
        return res

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, *, parent_id=False, subtype_id=False, **kwargs):
        """ Temporary workaround to avoid spam. If someone replies on a channel
        through the 'Presentation Published' email, it should be considered as a
        note as we don't want all channel followers to be notified of this answer. """
        self.ensure_one()
        if parent_id:
            parent_message = self.env['mail.message'].sudo().browse(parent_id)
            if parent_message.subtype_id and parent_message.subtype_id == self.env.ref('website_blog.mt_blog_blog_published'):
                subtype_id = self.env.ref('mail.mt_note').id
        return super(Blog, self).message_post(parent_id=parent_id, subtype_id=subtype_id, **kwargs)

    def all_tags(self, join=False, min_limit=1):
        BlogTag = self.env['blog.tag']
        req = """
            SELECT
                p.blog_id, count(*), r.blog_tag_id
            FROM
                blog_post_blog_tag_rel r
                    join blog_post p on r.blog_post_id=p.id
            WHERE
                p.blog_id in %s
            GROUP BY
                p.blog_id,
                r.blog_tag_id
            ORDER BY
                count(*) DESC
        """
        self._cr.execute(req, [tuple(self.ids)])
        tag_by_blog = {i.id: [] for i in self}
        all_tags = set()
        for blog_id, freq, tag_id in self._cr.fetchall():
            if freq >= min_limit:
                if join:
                    all_tags.add(tag_id)
                else:
                    tag_by_blog[blog_id].append(tag_id)

        if join:
            return BlogTag.browse(all_tags)

        for blog_id in tag_by_blog:
            tag_by_blog[blog_id] = BlogTag.browse(tag_by_blog[blog_id])

        return tag_by_blog

    @api.model
    def _search_get_detail(self, website, order, options):
        with_description = options['displayDescription']
        search_fields = ['name']
        fetch_fields = ['id', 'name']
        mapping = {
            'name': {'name': 'name', 'type': 'text', 'match': True},
            'website_url': {'name': 'url', 'type': 'text', 'truncate': False},
        }
        if with_description:
            search_fields.append('subtitle')
            fetch_fields.append('subtitle')
            mapping['description'] = {'name': 'subtitle', 'type': 'text', 'match': True}
        return {
            'model': 'blog.blog',
            'base_domain': [website.website_domain()],
            'search_fields': search_fields,
            'fetch_fields': fetch_fields,
            'mapping': mapping,
            'icon': 'fa-rss-square',
            'order': 'name desc, id desc' if 'name desc' in order else 'name asc, id desc',
        }

    def _search_render_results(self, fetch_fields, mapping, icon, limit):
        results_data = super()._search_render_results(fetch_fields, mapping, icon, limit)
        for data in results_data:
            data['url'] = '/blog/%s' % data['id']
        return results_data

class BlogTagCategory(models.Model):
    _name = 'blog.tag.category'
    _description = 'Blog Tag Category'
    _order = 'name'

    name = fields.Char('Name', required=True, translate=True)
    tag_ids = fields.One2many('blog.tag', 'category_id', string='Tags')

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Tag category already exists!"),
    ]


class BlogTag(models.Model):
    _name = 'blog.tag'
    _description = 'Blog Tag'
    _inherit = ['website.seo.metadata']
    _order = 'name'

    name = fields.Char('Name', required=True, translate=True)
    category_id = fields.Many2one('blog.tag.category', 'Category', index=True)
    post_ids = fields.Many2many('blog.post', string='Posts')

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Tag name already exists!"),
    ]


class BlogPost(models.Model):
    _name = "blog.post"
    _description = "Blog Post"
    _inherit = ['mail.thread', 'website.seo.metadata', 'website.published.multi.mixin',
        'website.cover_properties.mixin', 'website.searchable.mixin']
    _order = 'id DESC'
    _mail_post_access = 'read'

    def _compute_website_url(self):
        super(BlogPost, self)._compute_website_url()
        for blog_post in self:
            blog_post.website_url = "/blog/%s/%s" % (slug(blog_post.blog_id), slug(blog_post))

    def _default_content(self):
        return '''
            <p class="o_default_snippet_text">''' + _("Start writing here...") + '''</p>
        '''
    name = fields.Char('Title', required=True, translate=True, default='')
    subtitle = fields.Char('Sub Title', translate=True)
    author_id = fields.Many2one('res.partner', 'Author', default=lambda self: self.env.user.partner_id, index='btree_not_null')
    author_avatar = fields.Binary(related='author_id.image_128', string="Avatar", readonly=False)
    author_name = fields.Char(related='author_id.display_name', string="Author Name", readonly=False, store=True)
    active = fields.Boolean('Active', default=True)
    blog_id = fields.Many2one('blog.blog', 'Blog', required=True, ondelete='cascade', default=lambda self: self.env['blog.blog'].search([], limit=1))
    tag_ids = fields.Many2many('blog.tag', string='Tags')
    content = fields.Html('Content', default=_default_content, translate=html_translate, sanitize=False)
    teaser = fields.Text('Teaser', compute='_compute_teaser', inverse='_set_teaser')
    teaser_manual = fields.Text(string='Teaser Content')

    website_message_ids = fields.One2many(domain=lambda self: [('model', '=', self._name), ('message_type', '=', 'comment')])

    # creation / update stuff
    create_date = fields.Datetime('Created on', readonly=True)
    published_date = fields.Datetime('Published Date')
    post_date = fields.Datetime('Publishing date', compute='_compute_post_date', inverse='_set_post_date', store=True,
                                help="The blog post will be visible for your visitors as of this date on the website if it is set as published.")
    create_uid = fields.Many2one('res.users', 'Created by', readonly=True)
    write_date = fields.Datetime('Last Updated on', readonly=True)
    write_uid = fields.Many2one('res.users', 'Last Contributor', readonly=True)
    visits = fields.Integer('No of Views', copy=False, default=0, readonly=True)
    website_id = fields.Many2one(related='blog_id.website_id', readonly=True, store=True)

    @api.depends('content', 'teaser_manual')
    def _compute_teaser(self):
        for blog_post in self:
            if blog_post.teaser_manual:
                blog_post.teaser = blog_post.teaser_manual
            else:
                content = text_from_html(blog_post.content, True)
                blog_post.teaser = content[:200] + '...'

    def _set_teaser(self):
        for blog_post in self:
            blog_post.teaser_manual = blog_post.teaser

    @api.depends('create_date', 'published_date')
    def _compute_post_date(self):
        for blog_post in self:
            if blog_post.published_date:
                blog_post.post_date = blog_post.published_date
            else:
                blog_post.post_date = blog_post.create_date

    def _set_post_date(self):
        for blog_post in self:
            blog_post.published_date = blog_post.post_date
            if not blog_post.published_date:
                blog_post.post_date = blog_post.create_date

    def _check_for_publication(self, vals):
        if vals.get('is_published'):
            for post in self.filtered(lambda p: p.active):
                post.blog_id.message_post_with_source(
                    'website_blog.blog_post_template_new_post',
                    subject=post.name,
                    render_values={'post': post},
                    subtype_xmlid='website_blog.mt_blog_blog_published',
                )
            return True
        return False

    @api.model_create_multi
    def create(self, vals_list):
        posts = super(BlogPost, self.with_context(mail_create_nolog=True)).create(vals_list)
        for post, vals in zip(posts, vals_list):
            post._check_for_publication(vals)
        return posts

    def write(self, vals):
        result = True
        # archiving a blog post, unpublished the blog post
        if 'active' in vals and not vals['active']:
            vals['is_published'] = False
        for post in self:
            copy_vals = dict(vals)
            published_in_vals = set(vals.keys()) & {'is_published', 'website_published'}
            if (published_in_vals and 'published_date' not in vals and
                    (not post.published_date or post.published_date <= fields.Datetime.now())):
                copy_vals['published_date'] = vals[list(published_in_vals)[0]] and fields.Datetime.now() or False
            result &= super(BlogPost, post).write(copy_vals)
        self._check_for_publication(vals)
        return result

    @api.returns('self', lambda value: value.id)
    def copy_data(self, default=None):
        self.ensure_one()
        name = _("%s (copy)", self.name)
        default = dict(default or {}, name=name)
        return super(BlogPost, self).copy_data(default)

    def _get_access_action(self, access_uid=None, force_website=False):
        """ Instead of the classic form view, redirect to the post on website
        directly if user is an employee or if the post is published. """
        self.ensure_one()
        user = self.env['res.users'].sudo().browse(access_uid) if access_uid else self.env.user
        if not force_website and user.share and not self.sudo().website_published:
            return super(BlogPost, self)._get_access_action(access_uid=access_uid, force_website=force_website)
        return {
            'type': 'ir.actions.act_url',
            'url': self.website_url,
            'target': 'self',
            'target_type': 'public',
            'res_id': self.id,
        }

    def _notify_get_recipients_groups(self, message, model_description, msg_vals=None):
        """ Add access button to everyone if the document is published. """
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

    def _notify_thread_by_inbox(self, message, recipients_data, msg_vals=False, **kwargs):
        """ Override to avoid keeping all notified recipients of a comment.
        We avoid tracking needaction on post comments. Only emails should be
        sufficient. """
        if msg_vals is None:
            msg_vals = {}
        if msg_vals.get('message_type', message.message_type) == 'comment':
            return
        return super(BlogPost, self)._notify_thread_by_inbox(message, recipients_data, msg_vals=msg_vals, **kwargs)

    def _default_website_meta(self):
        res = super(BlogPost, self)._default_website_meta()
        res['default_opengraph']['og:description'] = res['default_twitter']['twitter:description'] = self.subtitle
        res['default_opengraph']['og:type'] = 'article'
        res['default_opengraph']['article:published_time'] = self.post_date
        res['default_opengraph']['article:modified_time'] = self.write_date
        res['default_opengraph']['article:tag'] = self.tag_ids.mapped('name')
        # background-image might contain single quotes eg `url('/my/url')`
        res['default_opengraph']['og:image'] = res['default_twitter']['twitter:image'] = json_scriptsafe.loads(self.cover_properties).get('background-image', 'none')[4:-1].strip("'")
        res['default_opengraph']['og:title'] = res['default_twitter']['twitter:title'] = self.name
        res['default_meta_description'] = self.subtitle
        return res

    @api.model
    def _search_get_detail(self, website, order, options):
        with_description = options['displayDescription']
        with_date = options['displayDetail']
        blog = options.get('blog')
        tags = options.get('tag')
        date_begin = options.get('date_begin')
        date_end = options.get('date_end')
        state = options.get('state')
        domain = [website.website_domain()]
        if blog:
            domain.append([('blog_id', '=', unslug(blog)[1])])
        if tags:
            active_tag_ids = [unslug(tag)[1] for tag in tags.split(',')] or []
            if active_tag_ids:
                domain.append([('tag_ids', 'in', active_tag_ids)])
        if date_begin and date_end:
            domain.append([("post_date", ">=", date_begin), ("post_date", "<=", date_end)])
        if self.env.user.has_group('website.group_website_designer'):
            if state == "published":
                domain.append([("website_published", "=", True), ("post_date", "<=", fields.Datetime.now())])
            elif state == "unpublished":
                domain.append(['|', ("website_published", "=", False), ("post_date", ">", fields.Datetime.now())])
        else:
            domain.append([("post_date", "<=", fields.Datetime.now())])
        search_fields = ['name', 'author_name']
        def search_in_tags(env, search_term):
            tags_like_search = env['blog.tag'].search([('name', 'ilike', search_term)])
            return [('tag_ids', 'in', tags_like_search.ids)]
        fetch_fields = ['name', 'website_url']
        mapping = {
            'name': {'name': 'name', 'type': 'text', 'match': True},
            'website_url': {'name': 'website_url', 'type': 'text', 'truncate': False},
        }
        if with_description:
            search_fields.append('content')
            fetch_fields.append('content')
            mapping['description'] = {'name': 'content', 'type': 'text', 'html': True, 'match': True}
        if with_date:
            fetch_fields.append('published_date')
            mapping['detail'] = {'name': 'published_date', 'type': 'date'}
        return {
            'model': 'blog.post',
            'base_domain': domain,
            'search_fields': search_fields,
            'search_extra': search_in_tags,
            'fetch_fields': fetch_fields,
            'mapping': mapping,
            'icon': 'fa-rss',
        }

```

## File: models\website_snippet_filter.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta

from odoo import models, fields, _


class WebsiteSnippetFilter(models.Model):
    _inherit = 'website.snippet.filter'

    def _get_hardcoded_sample(self, model):
        samples = super()._get_hardcoded_sample(model)
        if model._name == 'blog.post':
            data = [{
                'cover_properties': '{"background-image": "url(\'/website_blog/static/src/img/cover_2.jpg\')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0"}',
                'name': _('Islands'),
                'subtitle': _('Alone in the ocean'),
                'post_date': fields.Date.today() - timedelta(days=1),
                'website_url': "",
            }, {
                'cover_properties': '{"background-image": "url(\'/website_blog/static/src/img/cover_3.jpg\')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0"}',
                'name': _('With a View'),
                'subtitle': _('Awesome hotel rooms'),
                'post_date': fields.Date.today() - timedelta(days=2),
                'website_url': "",
            }, {
                'cover_properties': '{"background-image": "url(\'/website_blog/static/src/img/cover_4.jpg\')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0"}',
                'name': _('Skies'),
                'subtitle': _('Taking pictures in the dark'),
                'post_date': fields.Date.today() - timedelta(days=3),
                'website_url': "",
            }, {
                'cover_properties': '{"background-image": "url(\'/website_blog/static/src/img/cover_5.jpg\')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0"}',
                'name': _('Satellites'),
                'subtitle': _('Seeing the world from above'),
                'post_date': fields.Date.today() - timedelta(days=4),
                'website_url': "",
            }, {
                'cover_properties': '{"background-image": "url(\'/website_blog/static/src/img/cover_6.jpg\')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0"}',
                'name': _('Viewpoints'),
                'subtitle': _('Seaside vs mountain side'),
                'post_date': fields.Date.today() - timedelta(days=5),
                'website_url': "",
            }, {
                'cover_properties': '{"background-image": "url(\'/website_blog/static/src/img/cover_7.jpg\')", "resize_class": "o_record_has_cover o_half_screen_height", "opacity": "0"}',
                'name': _('Jungle'),
                'subtitle': _('Spotting the fauna'),
                'post_date': fields.Date.today() - timedelta(days=6),
                'website_url': "",
            }]
            merged = []
            for index in range(0, max(len(samples), len(data))):
                merged.append({**samples[index % len(samples)], **data[index % len(data)]})
                # merge definitions
            samples = merged
        return samples

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_qweb_fields
from . import website
from . import website_blog
from . import website_snippet_filter

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
blog_blog_public,blog.blog,model_blog_blog,base.group_public,1,0,0,0
blog_blog_portal,blog.blog,model_blog_blog,base.group_portal,1,0,0,0
blog_blog_employee,blog.blog,model_blog_blog,base.group_user,1,0,0,0
blog_blog,blog.blog,model_blog_blog,website.group_website_designer,1,1,1,1
blog_post_public,blog.post,model_blog_post,base.group_public,1,0,0,0
blog_post_portal,blog.post,model_blog_post,base.group_portal,1,0,0,0
blog_post_employee,blog.post,model_blog_post,base.group_user,1,0,0,0
blog_post,blog.post,model_blog_post,website.group_website_designer,1,1,1,1
blog_tag_public,blog.tag,model_blog_tag,base.group_public,1,0,0,0
blog_tag_portal,blog.tag,model_blog_tag,base.group_portal,1,0,0,0
blog_tag_employee,blog.tag,model_blog_tag,base.group_user,1,0,0,0
blog_tag_edition,blog.tag,model_blog_tag,website.group_website_designer,1,1,1,1
blog_tag_category_public,blog.tag.category,model_blog_tag_category,base.group_public,1,0,0,0
blog_tag_category_portal,blog.tag.category,model_blog_tag_category,base.group_portal,1,0,0,0
blog_tag_category_employee,blog.tag.category,model_blog_tag_category,base.group_user,1,0,0,0
blog_tag_category,blog.tag.category,model_blog_tag_category,website.group_website_designer,1,1,1,1

```

## File: security\website_blog_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record model="ir.rule" id="website_blog_post_public">
        <field name="name">Blog Post: public: published only</field>
        <field name="model_id" ref="model_blog_post"/>
        <field name="domain_force">[('website_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
    </record>

    <record model="ir.rule" id="website_blog_public">
        <field name="name">Blog: active only</field>
        <field name="model_id" ref="model_blog_blog"/>
        <field name="domain_force">[('active', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
    </record>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M47 28A25 25 0 0 0 22 3v25h25Z" fill="#985184"/><path d="M39 32a21.001 21.001 0 0 0-21-21v21h21Z" fill="#088BF5"/><path d="M38.615 28A20.997 20.997 0 0 0 22 11.385V28h16.615Z" fill="#144496"/><path d="M6.942 29.419c.271-1.53 1.413-2.824 2.93-3.319l12.56-4.1s2.868.33 4.05 1.488C27.664 24.645 28 27.454 28 27.454l-4.222 12.382a4.375 4.375 0 0 1-3.261 2.845L4 46l2.942-16.581Z" fill="#2EBCFA"/></svg>

```

## File: static\src\img\s_blog_posts.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <rect id="path-1" width="82" height="55" x="0" y="0"/>
    <linearGradient id="linearGradient-3" x1="72.875%" x2="40.332%" y1="46.753%" y2="35.353%">
      <stop offset="0%" stop-color="#008374"/>
      <stop offset="100%" stop-color="#006A59"/>
    </linearGradient>
    <linearGradient id="linearGradient-4" x1="88.517%" x2="50%" y1="41.799%" y2="50%">
      <stop offset="0%" stop-color="#00AA89"/>
      <stop offset="100%" stop-color="#009989"/>
    </linearGradient>
    <rect id="path-5" width="22" height="3" x="14" y="21"/>
    <filter id="filter-6" width="104.5%" height="166.7%" x="-2.3%" y="-16.7%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <path id="path-7" d="M24 31v1H14v-1h10zm9-3v1H14v-1h19z"/>
    <filter id="filter-8" width="105.3%" height="150%" x="-2.6%" y="-12.5%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.2 0"/>
    </filter>
    <path id="path-9" d="M59 32.5a2.41 2.41 0 0 1-.73 1.77 2.41 2.41 0 0 1-1.77.73 2.41 2.41 0 0 1-1.77-.73A2.41 2.41 0 0 1 54 32.5c0-.694.243-1.285.73-1.77A2.41 2.41 0 0 1 56.5 30a2.41 2.41 0 0 1 1.77.73A2.41 2.41 0 0 1 59 32.5zm5.998 1.653a.743.743 0 0 1-.209.59.723.723 0 0 1-.577.257h-1.657a.754.754 0 0 1-.528-.203.747.747 0 0 1-.245-.51c-.18-1.873-.935-3.475-2.265-4.805-1.33-1.33-2.931-2.085-4.805-2.265a.747.747 0 0 1-.51-.246.754.754 0 0 1-.202-.528v-1.657c0-.238.086-.43.258-.577a.715.715 0 0 1 .528-.209h.06c1.31.106 2.562.436 3.757.988a10.853 10.853 0 0 1 3.179 2.229 10.856 10.856 0 0 1 2.228 3.18 11.01 11.01 0 0 1 .988 3.756zm6 .038a.698.698 0 0 1-.217.568.714.714 0 0 1-.556.241H68.5a.754.754 0 0 1-.537-.211.722.722 0 0 1-.236-.513 13.5 13.5 0 0 0-1.22-4.933c-.715-1.557-1.647-2.91-2.794-4.056-1.147-1.147-2.499-2.08-4.056-2.796a13.672 13.672 0 0 0-4.932-1.231.722.722 0 0 1-.513-.235.74.74 0 0 1-.211-.526v-1.726c0-.226.08-.41.241-.556a.723.723 0 0 1 .532-.217h.036a16.75 16.75 0 0 1 6.054 1.449A16.924 16.924 0 0 1 66 22.999a16.926 16.926 0 0 1 3.55 5.137 16.755 16.755 0 0 1 1.448 6.055z"/>
    <filter id="filter-11" width="105.9%" height="111.8%" x="-2.9%" y="-2.9%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_latest_posts">
      <rect width="82" height="60" class="bg"/>
      <g class="group_2">
        <g class="group" opacity=".5">
          <g class="oval___oval_mask">
            <mask id="mask-2" fill="#fff">
              <use xlink:href="#path-1"/>
            </mask>
            <use fill="#79D1F2" class="mask" xlink:href="#path-1"/>
            <circle cx="65.5" cy="11.5" r="7.5" fill="#F3EC60" class="oval" mask="url(#mask-2)"/>
            <ellipse cx="63" cy="55.5" fill="url(#linearGradient-3)" class="oval" mask="url(#mask-2)" rx="28" ry="16.5"/>
            <ellipse cx="6.5" cy="52.5" fill="url(#linearGradient-4)" class="oval" mask="url(#mask-2)" rx="42.5" ry="22.5"/>
          </g>
        </g>
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-6)" xlink:href="#path-5"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-5"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-8)" xlink:href="#path-7"/>
          <use fill="#FFF" fill-opacity=".8" xlink:href="#path-7"/>
        </g>
        <mask id="mask-10" fill="#fff">
          <use xlink:href="#path-9"/>
        </mask>
        <g class="rss">
          <use fill="#000" filter="url(#filter-11)" xlink:href="#path-9"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-9"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\js\contentshare.js

```javascript
/** @odoo-module **/

import { sprintf } from "@web/core/utils/strings";
import dom from "@web/legacy/js/core/dom";

$.fn.share = function (options) {
    var option = $.extend($.fn.share.defaults, options);
    var selected_text = "";
    $.extend($.fn.share, {
        init: function (shareable) {
            var self = this;
            $.fn.share.defaults.shareable = shareable;
            $.fn.share.defaults.shareable.on('mouseup', function () {
                if ($(this).parents('body.editor_enable').length === 0) {
                    self.popOver();
                }
            });
            $.fn.share.defaults.shareable.on('mousedown', function () {
                self.destroy();
            });
        },
        getContent: function () {
            var $popover_content = $('<div class="h4 m-0"/>');
            if ($('.o_wblog_title, .o_wblog_post_content_field').hasClass('js_comment')) {
                selected_text = this.getSelection('string');
                var $btn_c = $('<a class="o_share_comment btn btn-link px-2" href="#"/>').append($('<i class="fa fa-lg fa-comment"/>'));
                $popover_content.append($btn_c);
            }
            if ($('.o_wblog_title, .o_wblog_post_content_field').hasClass('js_tweet')) {
                var tweet = '"%s" - %s';
                var baseLength = tweet.replace(/%s/g, '').length;
                // Shorten the selected text to match the tweet max length
                // Note: all (non-localhost) urls in a tweet have 23 characters https://support.twitter.com/articles/78124
                var selectedText = this.getSelection('string').substring(0, option.maxLength - baseLength - 23);

                var text = window.btoa(encodeURIComponent(sprintf(tweet, selectedText, window.location.href)));
                $popover_content.append(sprintf(
                    "<a onclick=\"window.open('%s' + atob('%s'), '_%s','location=yes,height=570,width=520,scrollbars=yes,status=yes')\"><i class=\"ml4 mr4 fa fa-twitter fa-lg\"/></a>",
                    option.shareLink, text, option.target));
            }
            return $popover_content;
        },
        commentEdition: function () {
            $(".o_portal_chatter_composer_form textarea").val('"' + selected_text + '" ').focus();
            const commentsEl = $('#o_wblog_post_comments')[0];
            if (commentsEl) {
                dom.scrollTo(commentsEl).then(() => {
                    window.location.hash = 'blog_post_comment_quote';
                });
            }
        },
        getSelection: function (share) {
            if (window.getSelection) {
                var selection = window.getSelection();
                if (!selection || selection.rangeCount === 0) {
                    return "";
                }
                if (share === 'string') {
                    return String(selection.getRangeAt(0)).replace(/\s{2,}/g, ' ');
                } else {
                    return selection.getRangeAt(0);
                }
            } else if (document.selection) {
                if (share === 'string') {
                    return document.selection.createRange().text.replace(/\s{2,}/g, ' ');
                } else {
                    return document.selection.createRange();
                }
            }
        },
        popOver: function () {
            this.destroy();
            if (this.getSelection('string').length < option.minLength) {
                return;
            }
            var data = this.getContent();
            var range = this.getSelection();

            var newNode = document.createElement("span");
            range.insertNode(newNode);
            newNode.className = option.className;
            var $pop = $(newNode);
            $pop.popover({
                trigger: 'manual',
                placement: option.placement,
                html: true,
                content: function () {
                    return data;
                }
            }).popover('show');
            $('.o_share_comment').on('click', this.commentEdition);
        },
        destroy: function () {
            var $span = $('span.' + option.className);
            $span.popover('hide');
            $span.remove();
        }
    });
    $.fn.share.init(this);
};

$.fn.share.defaults = {
    shareLink: "http://twitter.com/intent/tweet?text=",
    minLength: 5,
    maxLength: 140,
    target: "blank",
    className: "share",
    placement: "top",
};

```

## File: static\src\js\options.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import options from "@web_editor/js/editor/snippets.options";
import "@website/js/editor/snippets.options";
import { uniqueId } from "@web/core/utils/functions";

const NEW_TAG_PREFIX = 'new-blog-tag-';

options.registry.many2one.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _selectRecord: function ($opt) {
        var self = this;
        this._super.apply(this, arguments);
        if (this.$target.data('oe-field') === 'author_id') {
            var $nodes = $('[data-oe-model="blog.post"][data-oe-id="' + this.$target.data('oe-id') + '"][data-oe-field="author_avatar"]');
            $nodes.each(function () {
                var $img = $(this).find('img');
                var css = window.getComputedStyle($img[0]);
                $img.css({width: css.width, height: css.height});
                $img.attr('src', '/web/image/res.partner/' + self.ID + '/avatar_1024');
            });
            setTimeout(function () {
                $nodes.removeClass('o_dirty');
            }, 0);
        }
    }
});

options.registry.CoverProperties.include({
    /**
     * @override
     */
    updateUI: async function () {
        const isBlogCover = this.$target[0].classList.contains('o_wblog_post_page_cover');
        if (!isBlogCover) {
            return this._super(...arguments);
        }
        var isRegularCover = this.$target.is('.o_wblog_post_page_cover_regular');
        var $coverFull = this.$el.find('[data-select-class*="o_full_screen_height"]');
        var $coverMid = this.$el.find('[data-select-class*="o_half_screen_height"]');
        var $coverAuto = this.$el.find('[data-select-class*="cover_auto"]');
        this._coverFullOriginalLabel = this._coverFullOriginalLabel || $coverFull.text();
        this._coverMidOriginalLabel = this._coverMidOriginalLabel || $coverMid.text();
        this._coverAutoOriginalLabel = this._coverAutoOriginalLabel || $coverAuto.text();
        $coverFull.children('div').text(isRegularCover ? _t("Large") : this._coverFullOriginalLabel);
        $coverMid.children('div').text(isRegularCover ? _t("Medium") : this._coverMidOriginalLabel);
        $coverAuto.children('div').text(isRegularCover ? _t("Tiny") : this._coverAutoOriginalLabel);
        return this._super(...arguments);
    },
});

options.registry.BlogPostTagSelection = options.Class.extend({
    init() {
        this._super(...arguments);
        this.orm = this.bindService("orm");
        this.notification = this.bindService("notification");
    },

    /**
     * @override
     */
    async willStart() {
        const _super = this._super.bind(this);

        this.blogPostID = parseInt(this.$target[0].dataset.blogId);
        this.isEditingTags = false;
        const tags = await this.orm.searchRead(
            "blog.tag",
            [],
            ["id", "name", "display_name", "post_ids"]
        );
        this.allTagsByID = {};
        this.tagIDs = [];
        for (const tag of tags) {
            this.allTagsByID[tag.id] = tag;
            if (tag['post_ids'].includes(this.blogPostID)) {
                this.tagIDs.push(tag.id);
            }
        }

        return _super(...arguments);
    },
    /**
     * @override
     */
    cleanForSave() {
        this._notifyUpdatedTags();
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * @see this.selectClass for params
     */
    setTags(previewMode, widgetValue, params) {
        if (this._preventNextSetTagsCall) {
            this._preventNextSetTagsCall = false;
            return;
        }
        this.tagIDs = JSON.parse(widgetValue).map(tag => tag.id);
    },
    /**
     * @see this.selectClass for params
     */
    createTag(previewMode, widgetValue, params) {
        if (!widgetValue) {
            return;
        }
        const existing = Object.values(this.allTagsByID).some(tag => {
            // A tag is already existing only if it was already defined (i.e.
            // id is a number) or if it appears in the current list of tags.
            return tag.name.toLowerCase() === widgetValue.toLowerCase()
                && (typeof(tag.id) === 'number' || this.tagIDs.includes(tag.id));
        });
        if (existing) {
            return this.notification.add(_t("This tag already exists"), {
                type: 'warning',
            });
        }
        const newTagID = uniqueId(NEW_TAG_PREFIX);
        this.allTagsByID[newTagID] = {
            'id': newTagID,
            'name': widgetValue,
            'display_name': widgetValue,
        };
        this.tagIDs.push(newTagID);
        // TODO Find a smarter way to achieve this.
        // Because of the invocation order of methods, setTags will be called
        // after createTag. This would reset the tagIds to the value before
        // adding the newly created tag. It therefore needs to be prevented.
        this._preventNextSetTagsCall = true;
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    async updateUI() {
        if (this.rerender) {
            this.rerender = false;
            await this._rerenderXML();
            return;
        }
        return this._super(...arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    async _computeWidgetState(methodName, params) {
        if (methodName === 'setTags') {
            return JSON.stringify(this.tagIDs.map(id => this.allTagsByID[id]));
        }
        return this._super(...arguments);
    },
    /**
     * @private
     */
    _notifyUpdatedTags() {
        this.trigger_up('set_blog_post_updated_tags', {
            blogPostID: this.blogPostID,
            tags: this.tagIDs.map(tagID => this.allTagsByID[tagID]),
        });
    },
    /**
     * @override
     */
    async _renderCustomXML(uiFragment) {
        uiFragment.querySelector('we-many2many').dataset.recordId = this.blogPostID;
    },
});

```

## File: static\src\js\website_blog.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import dom from "@web/legacy/js/core/dom";
import publicWidget from "@web/legacy/js/public/public_widget";

publicWidget.registry.websiteBlog = publicWidget.Widget.extend({
    selector: '.website_blog',
    events: {
        'click #o_wblog_next_container': '_onNextBlogClick',
        'click #o_wblog_post_content_jump': '_onContentAnchorClick',
        'click .o_twitter, .o_facebook, .o_linkedin, .o_google, .o_twitter_complete, .o_facebook_complete, .o_linkedin_complete, .o_google_complete': '_onShareArticle',
    },

    /**
     * @override
     */
    start: function () {
        $('.js_tweet, .js_comment').share({});
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onNextBlogClick: function (ev) {
        ev.preventDefault();
        var self = this;
        var $el = $(ev.currentTarget);
        var nexInfo = $el.find('#o_wblog_next_post_info').data();
        $el.find('.o_record_cover_container').addClass(nexInfo.size + ' ' + nexInfo.text).end()
           .find('.o_wblog_toggle').toggleClass('d-none');
        // Appending a placeholder so that the cover can scroll to the top of the
        // screen, regardless of its height.
        const placeholder = document.createElement('div');
        placeholder.style.minHeight = '100vh';
        this.$('#o_wblog_next_container').append(placeholder);

        // Use setTimeout() to calculate the 'offset()'' only after that size classes
        // have been applyed and that $el has been resized.
        setTimeout(() => {
            self._forumScrollAction($el, 300, function () {
                window.location.href = nexInfo.url;
            });
        });
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onContentAnchorClick: function (ev) {
        ev.preventDefault();
        ev.stopImmediatePropagation();
        var $el = $(ev.currentTarget.hash);

        this._forumScrollAction($el, 500, function () {
            window.location.hash = 'blog_content';
        });
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onShareArticle: function (ev) {
        ev.preventDefault();
        var url = '';
        var $element = $(ev.currentTarget);
        var blogPostTitle = $('#o_wblog_post_name').html() || '';
        var articleURL = window.location.href;
        if ($element.hasClass('o_twitter')) {
            var tweetText = _t(
                "Amazing blog article: %s! Check it live: %s",
                blogPostTitle,
                articleURL
            );
            url = 'https://twitter.com/intent/tweet?tw_p=tweetbutton&text=' + encodeURIComponent(tweetText);
        } else if ($element.hasClass('o_facebook')) {
            url = 'https://www.facebook.com/sharer/sharer.php?u=' + encodeURIComponent(articleURL);
        } else if ($element.hasClass('o_linkedin')) {
            url = 'https://www.linkedin.com/sharing/share-offsite/?url=' + encodeURIComponent(articleURL);
        }
        window.open(url, '', 'menubar=no, width=500, height=400');
    },

    //--------------------------------------------------------------------------
    // Utils
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {JQuery} $el - the element we are scrolling to
     * @param {Integer} duration - scroll animation duration
     * @param {Function} callback - to be executed after the scroll is performed
     */
    _forumScrollAction: function ($el, duration, callback) {
        dom.scrollTo($el[0], {duration: duration}).then(() => callback());
    },
});

```

## File: static\src\js\wysiwyg_adapter.js

```javascript
/** @odoo-module **/

import { WysiwygAdapterComponent } from '@website/components/wysiwyg_adapter/wysiwyg_adapter';
import { patch } from "@web/core/utils/patch";

patch(WysiwygAdapterComponent.prototype, {
    /**
     * @override
     */
    init() {
        super.init(...arguments);
        this.blogTagsPerBlogPost = {};
    },
    /**
     * @override
     */
    async startEdition() {
        await super.startEdition(...arguments);
        this.options.document.defaultView.$('.js_tweet, .js_comment').off('mouseup').trigger('mousedown');
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    async _saveViewBlocks() {
        const ret = await super._saveViewBlocks(...arguments);
        await this._saveBlogTags(); // Note: important to be called after save otherwise cleanForSave is not called before
        return ret;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Saves the blog tags in the database.
     *
     * @private
     */
    async _saveBlogTags() {
        for (const [key, tags] of Object.entries(this.blogTagsPerBlogPost)) {
            const proms = tags.filter(tag => typeof tag.id === 'string').map(tag => {
                return this.orm.create("blog.tag", [{
                        'name': tag.name,
                    }]);
            });
            const createdIDs = (await Promise.all(proms)).flat();

            await this.orm.write("blog.post", [parseInt(key)], {
                'tag_ids': [[6, 0, tags.filter(tag => typeof tag.id === 'number').map(tag => tag.id).concat(createdIDs)]],
            });
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {OdooEvent} ev
     */
    _onSetBlogPostUpdatedTags: function (ev) {
        this.blogTagsPerBlogPost[ev.data.blogPostID] = ev.data.tags;
    },

    /**
     * @override
     */
    _trigger_up(ev) {
        if (ev.name === 'set_blog_post_updated_tags') {
            this._onSetBlogPostUpdatedTags(ev);
            return;
        } else {
            return super._trigger_up(...arguments);
        }
    },
});

```

## File: static\src\js\components\translator.js

```javascript
/** @odoo-module **/

import { WebsiteTranslator } from '@website/components/translator/translator';
import { patch } from "@web/core/utils/patch";

patch(WebsiteTranslator.prototype, {
    /**
     * @override
     */
    _beforeEditorActive() {
        super._beforeEditorActive(...arguments);
        $(this.websiteService.pageDocument).find('[data-translate-error-tooltip]').tooltip({
            container: this.websiteService.pageDocument.body,
            trigger: 'click',
            delay: {'show': 0, 'hide': 0},
            title: function () {
                return $(this).data('translate-error-tooltip');
            },
        });
    }
});

```

## File: static\src\js\systray_items\new_content.js

```javascript
/** @odoo-module **/

import { NewContentModal, MODULE_STATUS } from '@website/systray_items/new_content';
import { patch } from "@web/core/utils/patch";

patch(NewContentModal.prototype, {
    setup() {
        super.setup();

        const newBlogElement = this.state.newContentElements.find(element => element.moduleXmlId === 'base.module_website_blog');
        newBlogElement.createNewContent = () => this.onAddContent('website_blog.blog_post_action_add', true);
        newBlogElement.status = MODULE_STATUS.INSTALLED;
        newBlogElement.model = 'blog.post';
    },
});

```

## File: static\src\js\tours\website_blog.js

```javascript
/** @odoo-module **/

    import { _t } from "@web/core/l10n/translation";
    import wTourUtils from "@website/js/tours/tour_utils";

    import { markup } from "@odoo/owl";

    wTourUtils.registerWebsitePreviewTour("blog", {
        url: "/",
    }, () => [{
        trigger: "body:not(:has(#o_new_content_menu_choices)) .o_new_content_container > a",
        content: _t("Click here to add new content to your website."),
        consumeVisibleOnly: true,
        position: 'bottom',
    }, {
        trigger: 'a[data-module-xml-id="base.module_website_blog"]',
        content: _t("Select this menu item to create a new blog post."),
        position: "bottom",
    }, {
        trigger: 'div[name="name"] input',
        content: _t("Enter your post's title"),
        position: "bottom",
    }, {
        trigger: "button.o_form_button_save",
        extra_trigger: 'div.o_field_widget[name="blog_id"]',
        content: _t("Select the blog you want to add the post to."),
        // Without demo data (and probably in most user cases) there is only
        // one blog so this step would not be needed and would block the tour.
        // We keep the step with "auto: true", so that the main python test
        // still works but never display this to the user anymore. We suppose
        // the user does not need guidance once that modal is opened. Note: if
        // you run the tour via your console without demo data, the tour will
        // thus fail as this will be considered.
        auto: true,
    }, {
        trigger: "iframe h1[data-oe-expression=\"blog_post.name\"]",
        extra_trigger: "#oe_snippets.o_loaded",
        content: _t("Edit your title, the subtitle is optional."),
        position: "top",
        // FIXME instead of using the default 'click' event that is used to mark
        // DIV elements as consumed, we would like to use the 'input' event for
        // this specific contenteditable element. However, using 'input' here
        // makes the auto test not work as the 'text' run method stops working
        // correctly for contenteditable element whose 'consumeEvent' is set to
        // 'input'. The auto tests should be entirely independent of what is set
        // as 'consumeEvent'. While this is investigated and fixed, let's use
        // the 'mouseup' event. Indeed we cannot let it to 'click' because of
        // the old editor currently removing all click handlers on top level
        // editable content (which the blog post title area is).
        consumeEvent: 'mouseup',
        run: "text",
    }, {
        trigger: "we-button[data-background]:nth(1)",
        extra_trigger: "iframe #wrap h1[data-oe-expression=\"blog_post.name\"]:not(:containsExact(\"\"))",
        content: markup(_t("Set a blog post <b>cover</b>.")),
        position: "top",
    }, {
        trigger: ".o_select_media_dialog .o_we_search",
        content: _t("Search for an image. (eg: type \"business\")"),
        position: "top",
        run() {},
    }, {
        trigger: ".o_select_media_dialog .o_existing_attachment_cell:first img",
        extra_trigger: '.modal:has(.o_existing_attachment_cell:first)',
        content: _t("Choose an image from the library."),
        position: "top",
    }, {
        trigger: "iframe #o_wblog_post_content",
        content: markup(_t("<b>Write your story here.</b> Use the top toolbar to style your text: add an image or table, set bold or italic, etc. Drag and drop building blocks for more graphical blogs.")),
        position: "top",
        run: function (actions) {
            actions.auto();
            actions.text("Blog content", this.$anchor.find("p"));
        },
    },
    ...wTourUtils.clickOnSave(),
    {
        trigger: ".o_menu_systray_item.o_mobile_preview > a",
        content: markup(_t("Use this icon to preview your blog post on <b>mobile devices</b>.")),
        position: "bottom",
    }, {
        trigger: ".o_menu_systray_item.o_mobile_preview > a",
        extra_trigger: '.o_website_preview.o_is_mobile',
        content: _t("Once you have reviewed the content on mobile, you can switch back to the normal view by clicking here again"),
        position: "right",
    }, {
        trigger: '.o_menu_systray_item a:contains("Unpublished")',
        extra_trigger: "iframe body:not(.editor_enable)",
        position: "bottom",
        content: markup(_t("<b>Publish your blog post</b> to make it visible to your visitors.")),
    }, {
        trigger: '.o_menu_systray_item a:contains("Published")',
        auto: true,
        isCheck: true,
    }
]);

```

## File: static\src\snippets\s_blog_posts\000.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import DynamicSnippet from "@website/snippets/s_dynamic_snippet/000";

const DynamicSnippetBlogPosts = DynamicSnippet.extend({
    selector: '.s_dynamic_snippet_blog_posts',
    disabledInEditableMode: false,

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Method to be overridden in child components in order to provide a search
     * domain if needed.
     * @override
     * @private
     */
    _getSearchDomain: function () {
        const searchDomain = this._super.apply(this, arguments);
        const filterByBlogId = parseInt(this.$el.get(0).dataset.filterByBlogId);
        if (filterByBlogId >= 0) {
            searchDomain.push(['blog_id', '=', filterByBlogId]);
        }
        return searchDomain;
    },

});
publicWidget.registry.blog_posts = DynamicSnippetBlogPosts;

export default DynamicSnippetBlogPosts;

```

## File: static\src\snippets\s_blog_posts\options.js

```javascript
/** @odoo-module **/

import options from "@web_editor/js/editor/snippets.options";
import dynamicSnippetOptions from "@website/snippets/s_dynamic_snippet/options";

import wUtils from "@website/js/utils";

const dynamicSnippetBlogPostsOptions = dynamicSnippetOptions.extend({
    /**
     *
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this.modelNameFilter = 'blog.post';
        this.blogs = {};
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     *
     * @override
     * @private
     */
    _computeWidgetVisibility: function (widgetName, params) {
        if (widgetName === 'hover_effect_opt') {
            return this.$target.get(0).dataset.templateKey === 'website_blog.dynamic_filter_template_blog_post_big_picture';
        }
        return this._super.apply(this, arguments);
    },
    /**
     * Fetches blogs.
     * @private
     * @returns {Promise}
     */
    _fetchBlogs: function () {
        return this.orm.searchRead("blog.blog", wUtils.websiteDomain(this), ["id", "name"]);
    },
    /**
     *
     * @override
     * @private
     */
    _renderCustomXML: async function (uiFragment) {
        await this._super.apply(this, arguments);
        await this._renderBlogSelector(uiFragment);
    },
    /**
     * Renders the blog option selector content into the provided uiFragment.
     * @private
     * @param {HTMLElement} uiFragment
     */
    _renderBlogSelector: async function (uiFragment) {
        if (!Object.keys(this.blogs).length) {
            const blogsList = await this._fetchBlogs();
            this.blogs = {};
            for (let index in blogsList) {
                this.blogs[blogsList[index].id] = blogsList[index];
            }
        }
        const blogSelectorEl = uiFragment.querySelector('[data-name="blog_opt"]');
        return this._renderSelectUserValueWidgetButtons(blogSelectorEl, this.blogs);
    },
    /**
     * Sets default options values.
     * @override
     * @private
     */
    _setOptionsDefaultValues: function () {
        this._setOptionValue('filterByBlogId', -1);
        this._super.apply(this, arguments);
    },
});

options.registry.dynamic_snippet_blog_posts = dynamicSnippetBlogPostsOptions;

export default dynamicSnippetBlogPostsOptions;

```

## File: views\blog_post_add.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<record id="blog_post_view_form_add" model="ir.ui.view">
    <field name="name">blog.post.view.form.add</field>
    <field name="model">blog.post</field>
    <field name="arch" type="xml">
        <form js_class="website_new_content_form">
            <group>
                <field name="website_url" invisible="1"/>
                <field name="blog_id" string="Select Blog"/>
                <field name="name" placeholder="Blog Post Title"/>
            </group>
        </form>
    </field>
</record>

<record id="blog_post_action_add" model="ir.actions.act_window">
    <field name="name">New Blog Post</field>
    <field name="res_model">blog.post</field>
    <field name="view_mode">form</field>
    <field name="target">new</field>
    <field name="view_id" ref="blog_post_view_form_add"/>
</record>

</odoo>

```

## File: views\website_blog_components.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>


<!-- ======   Template: Blogs Navbar   =========================================
============================================================================ -->
<template id="blogs_nav" name="Blogs Bar Template">
    <nav t-attf-class="navbar navbar-expand-lg navbar-light pt-4 pb-0 px-0 #{additionnal_classes}">
        <div t-attf-class="container gap-2 w-100 #{len(blogs) > 4 and 'flex-nowrap' or 'flex-wrap flex-sm-nowrap'}" class="container">
            <!-- Desktop -->
            <ul t-if="len(blogs) > 1" class="navbar-nav d-none d-lg-flex">
                <li class="nav-item">
                    <span class="nav-link disabled ps-0">Blogs:</span>
                </li>
                <li class="nav-item">
                    <a href="/blog" t-attf-class="nav-link fw-bold #{(not blog) and 'active'}">All</a>
                </li>
                <li class="nav-item" t-foreach="blogs" t-as="nav_blog">
                    <a t-attf-href="/blog/#{slug(nav_blog)}" t-cache="nav_blog,blog and nav_blog.id == blog.id"
                       t-attf-class="nav-link #{(blog and nav_blog.id == blog.id) and 'active' or ''}">
                        <b t-field="nav_blog.name"/>
                    </a>
                </li>
            </ul>
            <!-- Mobile -->
            <t t-set="wblog_nav_offcanvas" t-value="len(blogs) > 4"/>
            <t t-set="wblog_all_title_string">All Blogs</t>

            <div t-if="len(blogs) > 1" t-attf-class="order-2 d-lg-none #{not wblog_nav_offcanvas and 'dropdown flex-grow-1 flex-sm-grow-0 min-w-0'}">
                <t t-if="wblog_nav_offcanvas">
                    <a class="btn btn-light"
                       role="button"
                       t-att-title="blog.name if blog else wblog_all_title_string"
                       data-bs-toggle="offcanvas"
                       data-bs-target="#o_wblog_offcanvas"
                       aria-controls="o_wblog_offcanvas">
                        <i class="fa fa-navicon" role="img"/>
                    </a>
                    <div id="o_wblog_offcanvas" class="offcanvas offcanvas-end d-lg-none">
                        <div class="offcanvas-header">
                            <h5 class="offcanvas-title my-0">Blogs</h5>
                            <button type="button" class="btn-close" data-bs-dismiss="offcanvas" aria-label="Close"/>
                        </div>
                        <div class="offcanvas-body">
                            <ul class="nav navbar-nav">
                                <li class="nav-item">
                                    <a href="/blog" t-attf-class="nav-link #{(not blog) and 'active'}" t-att-title="wblog_all_title_string">All</a>
                                </li>
                                <li t-foreach="blogs" t-as="nav_blog" class="nav-item">
                                    <a t-attf-href="/blog/#{slug(nav_blog)}"
                                       t-attf-class="nav-link #{(blog and nav_blog.id == blog.id) and 'active' or ''}"
                                       t-att-title="nav_blog.name">
                                        <span t-field="nav_blog.name"/>
                                    </a>
                                </li>
                            </ul>
                        </div>
                    </div>
                </t>
                <t t-else="">
                    <a class="btn btn-light dropdown-toggle d-flex align-items-center justify-content-between"
                       role="button"
                       t-att-title="blog.name if blog else wblog_all_title_string"
                       data-bs-toggle="dropdown"
                       aria-expanded="false">
                        <span t-if="blog" class="text-truncate" t-out="blog.name"/>
                        <span t-else="" class="text-truncate">All</span>
                    </a>
                    <ul class="dropdown-menu dropdown-menu-end">
                        <li>
                            <a href="/blog" t-attf-class="dropdown-item #{(not blog) and 'active'}" title="All Blogs">All</a>
                        </li>
                        <li t-foreach="blogs" t-as="nav_blog">
                            <a t-attf-href="/blog/#{slug(nav_blog)}"
                               t-attf-class="dropdown-item #{(blog and nav_blog.id == blog.id) and 'active' or ''}"
                               t-att-title="nav_blog.name">
                                <span t-field="nav_blog.name"/>
                            </a>
                        </li>
                    </ul>
                </t>
            </div>
            <!-- 'Search Box' -->
            <t t-call="website.website_search_box_input">
                <t t-set="_form_classes" t-valuef="#{not len(blogs) &gt; 1 and 'ms-auto'} flex-grow-1 flex-lg-grow-0"/>
                <t t-set="_classes" t-valuef=""/>
                <t t-set="search_type" t-valuef="blogs"/>
                <t t-set="action" t-value="blog_url(tag=tag,search=search)"/>
                <t t-set="display_description" t-valuef="true"/>
                <t t-set="display_detail" t-valuef="false"/>
                <input type="hidden" name="tag" t-att-value="tag"/>
                <input type="hidden" name="date_begin" t-att-value="date_begin"/>
                <input type="hidden" name="date_end" t-att-value="date_end"/>
            </t>
        </div>
    </nav>
</template>

<!-- ======   Template: List Tags in inline list    ============================
============================================================================ -->
<template id="tags_list" name="Tags List">
    <t t-if="tags">
        <div t-if="not hide_title and categ_title" class="text-muted mb-1 h6" t-esc="categ_title"/>
        <t t-foreach="tags" t-as="tag">
            <t t-if="tag.post_ids">
                <span t-if="dismissibleBtn and tag.id in active_tag_ids" class="align-items-baseline border d-inline-flex ps-2 rounded mb-2">
                    <i class="fa fa-tag me-2 text-muted"/>
                    <t t-esc="tag.name"/>
                    <a t-attf-href="#{blog_url(tag=tags_list(active_tag_ids, tag.id))}" class="btn border-0 py-1 post_link" t-att-rel="len(active_tag_ids) and 'nofollow'">&#215;</a>
                </span>
                <a t-elif="showInactive" t-attf-href="#{blog_url(tag=tags_list(active_tag_ids, tag.id))}" t-attf-class="badge mb-2 mw-100 text-truncate text-decoration-none #{tag.id in active_tag_ids and 'text-bg-primary' or 'border text-primary'} post_link" t-att-rel="len(active_tag_ids) and 'nofollow'" t-esc="tag.name"/>
            </t>
        </t>
    </t>
</template>

<!-- ======   Template: Date Selector   ========================================
============================================================================ -->
<template id="date_selector">
    <select name="archive" oninput="location = this.value;" class="form-select">
        <option t-att-value="blog_url(date_begin=False, date_end=False) if blog else '/blog'"
                t-att="[('selected' if (not date_begin) else 'unselected' ) , 'true' ]">
                -- All dates
        </option>

        <optgroup t-foreach="nav_list" t-as="year" t-attf-label="#{year}">
            <option t-foreach="nav_list[year]" t-as="months"
                    t-att="[('selected' if date_begin and (months['date_begin'] == date_begin) else 'unselected' ) , 'true' ]"
                    t-attf-value="#{blog_url(date_begin=months['date_begin'], date_end=months['date_end'], tag=tag)}">
                <t t-esc="months['month']"/>
                <t t-esc="year"/>
            </option>
        </optgroup>
    </select>
</template>

<!-- ======   Template: Post Author   ==========================================
============================================================================ -->
<template id="post_author">
    <div t-attf-class="o_not_editable align-items-center position-relative #{additionnal_classes or ''}">
        <div t-if="blog_post.author_avatar"
             t-field="blog_post.author_avatar"
             style="line-height:1"
             t-options='{"widget": "image", "class": "rounded-circle " + "o_wblog_author_avatar me-1" if hide_date else  "o_wblog_author_avatar_date me-2"}' />
        <div t-att-class="not hide_date and 'small fw-bold'" style="line-height:1">
            <span t-if="editable" t-field="blog_post.author_id" t-options='{ "widget": "contact", "fields": ["name"]}'/>
            <span t-else="" t-esc="blog_post.author_name"/>
            <small t-if="not hide_date" t-field="blog_post.post_date" t-options="{'format': 'long', 'date_only': 'true'}"/>
        </div>
    </div>
</template>

<!-- ======   Template: Post Breadcrumbs   =====================================
============================================================================ -->
<template id="post_breadcrumbs">
    <nav aria-label="breadcrumb" t-attf-class="breadcrumb flex-nowrap py-0 px-0 css_editable_mode_hidden #{additionnal_classes or ''}">
        <li t-if="len(blogs) &gt; 1" class="breadcrumb-item"><a href="/blog">All Blogs</a></li>
        <li class="breadcrumb-item">
            <a t-attf-href="#{blog_url(tag=None, date_begin=None, date_end=None)}" t-esc="blog.name"/>
        </li>
        <li class="breadcrumb-item text-truncate active"><span t-esc="blog_post.name"/></li>
    </nav>
</template>

<!-- ======   Template: Sidebar Blog  ==========================================
Display sidebar in 'All blogs'/single blog pages.

Options:
# opt_sidebar_blog_index_follow_us : Display follow-us links
# opt_sidebar_blog_index_archives : Display a <select> input with post by month
# opt_sidebar_blog_index_tags: Display tags cloud
============================================================================ -->
<template id="sidebar_blog_index" name="Sidebar - Blog page">
    <div id="o_wblog_sidebar" class="w-100">
        <div class="oe_structure" id="oe_structure_blog_sidebar_index_1"/>
        <div class="o_wblog_sidebar_block pb-5">
            <h6 class="text-uppercase pb-2 mb-4 border-bottom fw-bold">About us</h6>
            <div>
                <p>Write a small text here to describe your blog or company.</p>
            </div>
        </div>
        <div class="oe_structure" id="oe_structure_blog_sidebar_index_2"/>
    </div>
</template>

<!-- (Option) Sidebar Blog: Follow Us -->
<template id="opt_sidebar_blog_index_follow_us" name="Follow Us" priority="1" inherit_id="website_blog.sidebar_blog_index" active="True">
    <xpath expr="//div[@id='o_wblog_sidebar']" position="inside">
        <div class="o_wblog_sidebar_block pb-5">
            <h6 class="text-uppercase pb-2 mb-4 border-bottom fw-bold">Follow Us</h6>
            <div class="o_wblog_social_links d-flex flex-wrap mx-n1 o_not_editable">
                <t t-set="classes" t-translation="off">bg-100 border mx-1 mb-2 rounded-circle d-flex align-items-center justify-content-center text-decoration-none</t>
                <a t-if="website.social_facebook" t-att-href="website.social_facebook" aria-label="Facebook" title="Facebook" t-att-class="classes"><i class="fa fa-facebook-square text-facebook"/></a>
                <a t-if="website.social_twitter" t-att-href="website.social_twitter" t-att-class="classes"><i class="fa fa-twitter text-twitter" aria-label="Twitter" title="Twitter"/></a>
                <a t-if="website.social_linkedin" t-att-href="website.social_linkedin" t-att-class="classes"><i class="fa fa-linkedin text-linkedin" aria-label="LinkedIn" title="LinkedIn"/></a>
                <a t-if="website.social_youtube" t-att-href="website.social_youtube" t-att-class="classes"><i class="fa fa-youtube-play text-youtube" aria-label="Youtube" title="Youtube"/></a>
                <a t-if="website.social_github" t-att-href="website.social_github" t-att-class="classes"><i class="fa fa-github text-github" aria-label="Github" title="Github"/></a>
                <a t-if="website.social_instagram" t-att-href="website.social_instagram" t-att-class="classes"><i class="fa fa-instagram text-instagram" aria-label="Instagram" title="Instagram"/></a>
                <a t-if="website.social_tiktok" t-att-href="website.social_tiktok" t-att-class="classes"><i class="fa fa-tiktok text-tiktok" aria-label="TikTok" title="TikTok"/></a>
                <a t-if="blog" t-att-href="'/blog/%s/feed' % slug(blog)" t-att-class="classes"><i class="fa fa-rss-square" aria-label="RSS" title="RSS"/></a>
            </div>
            <t t-call="website_mail.follow" t-if="blog">
                <t t-set="email" t-value="user_id.email"/>
                <t t-set="object" t-value="blog"/>
                <t t-set="div_class" t-value="'pt-2'"/>
            </t>
        </div>
        <div class="oe_structure" id="oe_structure_blog_sidebar_index_3"/>
    </xpath>
</template>

<!-- (Option) Sidebar Blog: Archives -->
<template id="opt_sidebar_blog_index_archives" name="Archives" priority="2" inherit_id="website_blog.sidebar_blog_index" active="True">
    <xpath expr="//div[@id='o_wblog_sidebar']" position="inside">
        <div class="o_wblog_sidebar_block pb-5">
            <h6 class="text-uppercase pb-2 mb-4 border-bottom fw-bold">Archives</h6>

            <t t-call="website_blog.date_selector"/>
        </div>
        <div class="oe_structure" id="oe_structure_blog_sidebar_index_4"/>
    </xpath>
</template>

<!-- (Option) Sidebar Blog: Show tags -->
<template id="opt_sidebar_blog_index_tags" name="Tags List" priority="3" inherit_id="website_blog.sidebar_blog_index" active="True">
    <xpath expr="//div[@id='o_wblog_sidebar']" position="inside">

        <div t-if="other_tags or tag_category" class="o_wblog_sidebar_block pb-5">
            <h6 class="text-uppercase pb-2 mb-4 border-bottom fw-bold">Tags</h6>
            <div class="h5">
                <t t-foreach="tag_category" t-as="nav_tag_category">
                    <t t-call="website_blog.tags_list">
                        <t t-set='categ_title' t-value="nav_tag_category.name"/>
                        <t t-set='tags' t-value='nav_tag_category.tag_ids' />
                        <t t-set="showInactive" t-value="True"/>
                    </t>
                </t>
                <t t-call="website_blog.tags_list">
                    <t t-set='hide_title' t-value='not len(tag_category)' />
                    <t t-set='categ_title'>Others</t>
                    <t t-set='tags' t-value='other_tags'/>
                    <t t-set="showInactive" t-value="True"/>
                </t>
            </div>
        </div>

        <div t-else="" groups="website.group_website_designer" class="o_wblog_sidebar_block pb-5">
            <h6 class="text-uppercase pb-2 mb-4 border-bottom fw-bold">Tags</h6>
            <em t-ignore="True" class="text-muted">No tags defined yet.</em>
        </div>
        <div class="oe_structure" id="oe_structure_blog_sidebar_index_5"/>
    </xpath>
</template>


<!-- ====== Blog Post Sidebar ==================================================
Display a sidebar beside the post content.
============================================================================ -->
<template id="blog_post_sidebar" name="Sidebar - Blog Post">
    <div id="o_wblog_post_sidebar">
        <div class="oe_structure" id="oe_structure_blog_post_sidebar_1"/>
    </div>
</template>


<!-- (Option) Post Sidebar: Author avatar -->
<template id="opt_blog_post_author_avatar_display" name="Author" inherit_id="website_blog.blog_post_sidebar" active="True" priority="1">
    <xpath expr="//div[@id='o_wblog_post_sidebar']" position="inside">
        <div class="o_wblog_sidebar_block pb-5">
            <t t-call="website_blog.post_author">
                <t t-set="additionnal_classes" t-value="'h5 d-flex align-items-center'"/>
            </t>
        </div>
        <div class="oe_structure" id="oe_structure_blog_post_sidebar_2"/>
    </xpath>
</template>

<!-- (Option) Post Sidebar: Share Links Display -->
<template id="opt_blog_post_share_links_display" name="Share Links" inherit_id="website_blog.blog_post_sidebar" active="True" priority="2">
    <xpath expr="//div[@id='o_wblog_post_sidebar']" position="inside">
        <div class="o_wblog_sidebar_block pb-5">
            <h6 class="text-uppercase pb-3 mb-4 border-bottom fw-bold">Share this post</h6>

            <div class="o_wblog_social_links d-flex flex-wrap mx-n1 o_not_editable">
                <t t-set="classes" t-translation="off">bg-100 border mx-1 mb-2 rounded-circle d-flex align-items-center justify-content-center text-decoration-none</t>
                <a href="#" aria-label="Facebook" title="Share on Facebook" t-attf-class="o_facebook #{classes}"><i class="fa fa-facebook-square text-facebook"/></a>
                <a href="#" aria-label="Twitter" title="Share on Twitter" t-attf-class="o_twitter #{classes}"><i class="fa fa-twitter text-twitter" aria-label="Twitter" title="Twitter"/></a>
                <a href="#" aria-label="LinkedIn" title="Share on LinkedIn" t-attf-class="o_linkedin #{classes}"><i class="fa fa-linkedin text-linkedin" aria-label="LinkedIn" title="LinkedIn"/></a>
            </div>
        </div>

        <div class="oe_structure" id="oe_structure_blog_post_sidebar_3"/>
    </xpath>
</template>

<!-- (Option) Post Sidebar: display tags -->
<template id="opt_blog_post_tags_display" name="Tags" inherit_id="website_blog.blog_post_sidebar" active="True" priority="3">
    <xpath expr="//div[@id='o_wblog_post_sidebar']" position="inside">
        <div class="o_wblog_sidebar_block pb-5">
            <h6 class="text-uppercase pb-3 mb-4 border-bottom fw-bold">Tags</h6>
            <t t-if="blog_post.tag_ids">
                <div class="h5">
                    <t t-foreach="blog_post.tag_ids" t-as="one_tag">
                        <a class="badge border post_link text-decoration-none text-primary" t-attf-href="#{blog_url(tag=slug(one_tag))}" t-esc="one_tag.name"/>
                    </t>
                </div>
            </t>
            <t t-else="">
                <div class="mb-4 bg-100 py-2 px-3 border" groups="website.group_website_designer">
                    <h6 class="text-muted"><em>No tags defined</em></h6>
                    <a role="menuitem" t-attf-href="/web#view_type=form&amp;model=#{main_object._name}&amp;id=#{main_object.id}&amp;action=#{action}&amp;menu_id=#{menu or main_object.env.ref('website.menu_website_configuration').id}"
                        title='Edit in backend' id="edit-in-backend">Add some</a>
                </div>
            </t>
        </div>
        <div class="oe_structure" id="oe_structure_blog_post_sidebar_4"/>
    </xpath>
</template>

<!-- (Option) Post Sidebar: display Blogs list -->
<template id="opt_blog_post_blogs_display" name="Blogs List" inherit_id="website_blog.blog_post_sidebar" active="True" priority="4">
    <xpath expr="//div[@id='o_wblog_post_sidebar']" position="inside">
        <div t-if="len(blogs) > 1" class="o_wblog_sidebar_block pb-5">
            <h6 class="text-uppercase pb-3 mb-4 border-bottom fw-bold">Our blogs</h6>
            <ul class="list-unstyled">
                <li t-foreach="blogs" t-as="nav_blog" class="mb-2">
                    <a t-attf-href="#{blog_url(blog=nav_blog, tag=False, date_begin=False, date_end=False)}"><b t-field="nav_blog.name"/></a>
                </li>
            </ul>
        </div>
        <div class="oe_structure" id="oe_structure_blog_post_sidebar_5"/>
    </xpath>
</template>

<!-- (Option) Post Sidebar: display Archive -->
<template id="opt_blog_post_archive_display" name="Archive" inherit_id="website_blog.blog_post_sidebar" active="True" priority="5">
    <xpath expr="//div[@id='o_wblog_post_sidebar']" position="inside">
        <div class="o_wblog_sidebar_block pb-5">
            <h6 class="text-uppercase pb-3 mb-4 border-bottom fw-bold">Archive</h6>

            <t t-call="website_blog.date_selector"/>
        </div>
        <div class="oe_structure" id="oe_structure_blog_post_sidebar_6"/>
    </xpath>
</template>

</odoo>

```

## File: views\website_blog_posts_loop.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- ======   Template: Posts Loop    ==========================================
Loop through post and call sub-templates (tags, cover..) in different position
according to the enabled options.
============================================================================ -->
<template id="posts_loop" name="Posts List">
    <div id="o_wblog_posts_loop" t-att-class="'o_wblog_list_view' if opt_blog_list_view else ''">

        <!-- Allow to filter post by published state. Visible only in edit-mode
             and if both published/unpublished number is > 0 -->
        <t t-if="state_info" t-set="state" t-value="state_info['state']"/>

        <!-- Check for active options -->
        <t t-set="opt_posts_loop_show_cover" t-value="is_view_active('website_blog.opt_posts_loop_show_cover')"/>

        <div groups="website.group_website_designer" t-if="state_info and (state_info['published'] > 0 and state_info['unpublished'] > 0)">
            <div class="bg-200 py-2 mb-4 alert alert-dismissable">
                <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                <span class="me-1">Show:</span>
                <div class="btn-group btn-group-sm">
                    <a t-attf-class="btn #{state == 'published' and 'btn-success' or 'btn-default bg-white border'}"
                       t-attf-href="#{state == 'published' and blog_url(state='') or blog_url(state='published')}">
                        <i t-attf-class="fa me-1 #{state == 'published' and 'fa-check-square-o' or 'fa-square-o'}"/>
                        Published (<t t-esc="state_info['published']" />)
                    </a>
                    <a t-attf-class="btn #{state == 'unpublished' and 'btn-success' or 'btn-default bg-white border'}"
                       t-attf-href="#{state == 'unpublished' and blog_url(state='') or blog_url(state='unpublished')}">
                        <i t-attf-class="fa me-1 #{state == 'unpublished' and 'fa-check-square-o' or 'fa-square-o'}"/>
                        Unpublished (<t t-esc="state_info['unpublished']" />)
                    </a>
                </div>
                <div class="pt-1 fst-italic small">This box will not be visible to your visitors</div>
            </div>
        </div>

        <div t-attf-class="row #{posts and not opt_blog_readable and 'mx-n2'}">
            <!-- Filters -->
            <div t-if="tag or date_begin or search" class="col-12 mb-3">
                <div t-if="posts" class="h4 mb-3">
                    <t t-esc="search_count"/>
                    <t t-if="search_count &lt; 2">Article</t>
                    <t t-else="">Articles</t>
                </div>
                <!-- TODO: in master, remove the next useless lines. They have
                    to be kept in the stable versions in case of a potential
                    customization that would do a 'xpath' on them.-->
                <t t-set="posts_save" t-value="posts"/>
                <t t-set="posts" t-value="False"/>
                <div t-if="posts" class="h4 mb-3">
                    <t t-esc="len(posts)"/>
                    <t t-if="len(posts) &lt; 2">Article</t>
                    <t t-else="">Articles</t>
                </div>
                <t t-set="posts" t-value="posts_save"/>
                <span t-if="search" class="align-items-baseline border d-inline-flex ps-2 rounded mb-2">
                    <i class="fa fa-search me-2 text-muted"/>
                    <t t-esc="search"/>
                    <a t-att-href="blog_url(search=False, tag=tag)" class="btn border-0 py-1 post_link">&#215;</a>
                </span>
                <t t-if="tag">
                    <!-- Show active tags with a category set -->
                    <t t-foreach="tag_category" t-as="nav_tag_category">
                        <t t-call="website_blog.tags_list">
                            <t t-set='tags' t-value='nav_tag_category.tag_ids' />
                            <t t-set='dismissibleBtn' t-value="True"/>
                        </t>
                    </t>

                    <!-- Show active tags without a category set -->
                    <t t-call="website_blog.tags_list">
                        <t t-set='tags' t-value='other_tags'/>
                        <t t-set='dismissibleBtn' t-value="True"/>
                    </t>
                </t>
                <span t-if="date_begin" class="align-items-baseline border d-inline-flex ps-2 rounded mb-2">
                    <i class="fa fa-calendar-o me-2 text-muted"/>
                    <t t-esc="date_begin" t-options="{'widget': 'date', 'format': 'MMM yyyy'}"></t>
                    <a t-attf-href="#{blog_url(date_begin=False, date_end=False, tag=tag)}" class="btn border-0 py-1">&#215;</a>
                </span>
                <hr class="mt-2"/>
            </div>

            <!-- No blog post yet -->
            <div t-if="not posts" class="col">
                <t t-set="no_results_str">No results for "%s".</t>
                <h2 t-if="search" t-esc="no_results_str % search" class="fw-bold"/>
                <h2 t-else="">No blog post yet.</h2>
                <div class="alert alert-info" groups="website.group_website_designer">
                    Click on "<b>New</b>" in the top-right corner to write your first blog post.
                </div>
            </div>

            <!-- Posts -->

            <!-- Define 'colWidth' qWeb variable, to be assigned later.
            Adjust accordingly if sidebar and/or readability modes are active. -->
            <t t-if="not opt_blog_list_view">
                <t t-if="opt_blog_readable">
                    <t t-if="opt_blog_sidebar_show" t-set="colWidth" t-value="'col-md-6'"/>
                    <t t-else="" t-set="colWidth" t-value="'col-md-6 col-xl-4'"/>
                </t>
                <t t-else="">
                    <t t-if="opt_blog_sidebar_show" t-set="colWidth" t-value="'px-2 col-md-6 col-xl-4'"/>
                    <t t-else="" t-set="colWidth" t-value="'px-2 col-sm-6 col-lg-4 col-xl-3'"/>
                </t>
            </t>
            <!-- Loop through posts: exclude the first one if already displayed as top banner -->
            <t t-foreach="posts" t-as="blog_post">
                <!-- Assign 'colWidth': 'col-12' is default for List-View and mobile -->
                <div t-attf-class="pb-4 col-12 #{colWidth}" t-cache="blog_post,opt_blog_list_view,opt_blog_readable,active_tag_ids">
                    <article t-attf-class="o_wblog_post position-relative #{'card h-100' if opt_blog_cards_design else ''}" name="blog_post">
                        <!-- List-View Design -->
                        <t t-if="opt_blog_list_view">
                            <div t-att-class="opt_blog_cards_design and 'card-body py-3'">
                                <t t-call="website_blog.post_heading"/>
                            </div>
                            <div t-if="not opt_blog_cards_design" class="py-2">
                                <t t-call="website_blog.post_info"></t>
                            </div>
                            <div t-if="opt_posts_loop_show_cover">
                                <t t-call="website_blog.post_cover_image"/>
                            </div>
                            <div t-if="is_view_active('website_blog.opt_posts_loop_show_teaser')" t-att-class="opt_blog_cards_design and 'card-body pt-0'">
                                <t t-call="website_blog.post_teaser"/>
                            </div>
                            <div t-if="opt_blog_cards_design" t-attf-class="opt_blog_cards_design and 'card-body pt-0 pb-2'}">
                                <t t-call="website_blog.post_info"></t>
                            </div>
                            <div t-else="" class="mt-3">
                                <a t-attf-href="/blog/#{slug(blog_post.blog_id)}/#{slug(blog_post)}" class="btn btn-primary">
                                    Read more <i class="oi oi-chevron-right ms-2"/>
                                </a>
                            </div>
                        </t>
                        <!-- Grid-View Design -->
                        <t t-if="not opt_blog_list_view">
                            <t t-if="opt_posts_loop_show_cover" t-call="website_blog.post_cover_image"/>
                            <div t-att-class="opt_blog_cards_design and 'card-body px-2 py-0 mb-2'">
                                <t t-call="website_blog.post_heading"/>
                                <div t-if="is_view_active('website_blog.opt_posts_loop_show_teaser')">
                                    <t t-call="website_blog.post_teaser"/>
                                </div>
                            </div>
                            <div t-attf-class="o_wblog_normalize_font #{'card-footer px-2 pb-2' if opt_blog_cards_design else 'pe-2 pb-2'}">
                                <t t-call="website_blog.post_info"></t>
                            </div>
                        </t>
                        <!-- Add 'unpublished' badge -->
                        <span t-if="not blog_post.website_published" class="bg-danger small py-1 px-2 position-absolute o_not_editable" style="top:0; right:0">unpublished</span>
                    </article>
                </div>
                <!-- List-View Design, add <hr> after post -->
                <div t-if="opt_blog_list_view and not blog_post_last" class="col-12 mt-2 mb-5 px-2"><hr/></div>
            </t>
        </div>
    </div>
</template>


<!-- ======   Sub-Template: Posts list : Posts Heading  =================== -->
<template id="post_heading">
    <a t-attf-href="/blog/#{slug(blog_post.blog_id)}/#{slug(blog_post)}"
       t-field="blog_post.name"
       t-attf-class="d-block text-reset text-decoration-none o_blog_post_title my-0 #{'h3' if opt_blog_list_view else ('h5' if opt_blog_readable else 'h6')}">
       Untitled Post
   </a>

    <div t-if="not opt_posts_loop_show_cover and is_view_active('website_blog.opt_posts_loop_show_author')" class="text-muted small mt-2">
        by <span t-field="blog_post.author_id"/>
    </div>
</template>

<!-- ======   Sub-Template: Posts list : Posts Info  ======================= -->
<template id="post_info">
    <div class="d-flex small flex-wrap mb-1 w-100">
        <div t-attf-class="d-flex flex-wrap align-items-center justify-content-between mx-n2 #{opt_blog_list_view and 'flex-grow-0 w-auto mw-100' or 'flex-grow-1' }">
            <time t-field="blog_post.post_date" class="text-nowrap fw-bold px-2" t-options="{'widget': 'datetime', 'date_only': 'true', 'format': 'medium'}"/>
            <div t-if="is_view_active('website_blog.opt_posts_loop_show_stats')" class="px-2">
                <b class="text-nowrap" title="Comments"><i class="fa fa-comment text-muted me-1"/><t t-esc="len(blog_post.message_ids)"/></b>
                <b class="text-nowrap ps-2" title="Views"><i class="fa fa-binoculars text-muted me-1"/><t t-esc="blog_post.visits"/></b>
            </div>
            <b t-if="posts_list_show_parent_blog" class="text-nowrap text-truncate px-2">
                <i class="fa fa-folder-open text-muted"/>
                <a t-attf-href="/blog/#{slug(blog_post.blog_id)}" t-field="blog_post.blog_id"/>
            </b>
        </div>
    </div>
</template>

<!-- ======   Sub-Template: Posts list : Posts Cover  ====================== -->
<template id="post_cover_image">
    <t t-if="opt_blog_cards_design and not opt_blog_list_view" t-set="classes" t-value="'card-img-top mb-2'"/>
    <t t-if="not opt_blog_cards_design and opt_blog_list_view" t-set="classes" t-value="'o_wblog_post_cover_nocard'"/>

    <a t-attf-href="/blog/#{slug(blog_post.blog_id)}/#{slug(blog_post)}"
       t-attf-class="text-decoration-none d-block #{classes or 'mb-2'}"
       t-att-style="not blog_post.website_published and 'opacity:0.6;'">

        <t t-call="website.record_cover">
            <t t-set="_record" t-value="blog_post"/>
            <t t-set="additionnal_classes" t-value="'o_list_cover o_not_editable ' + (not opt_blog_cards_design and ' rounded overflow-hidden shadow mb-3' or '')"/>

            <t t-if="is_view_active('website_blog.opt_posts_loop_show_author')" t-call="website_blog.post_author">
                <t t-set="additionnal_classes" t-value="'o_wblog_post_list_author o_list_cover d-flex text-white w-100 o_not_editable ' + ('p-3 h5 m-0' if opt_blog_list_view else 'px-2 pb-2 pt-3') "/>
                <t t-set="hide_date" t-value="True"/>
            </t>
        </t>
    </a>
</template>

<!-- ======   Sub-Template: Posts list : Posts Teaser + Tags  ============= -->
<template id="post_teaser">
    <t t-cache="blog_post,str(active_tag_ids)">
    <a t-attf-href="/blog/#{slug(blog_post.blog_id)}/#{slug(blog_post)}" class="text-reset text-decoration-none">
        <div t-if="opt_blog_list_view" t-field="blog_post.teaser" class="mt-2 o_wblog_read_text"/>
        <div t-else="" t-field="blog_post.teaser" t-attf-class="mt-2 #{opt_blog_readable and 'o_wblog_normalize_font'}"/>
    </a>

    <!-- Tags -->
    <div t-if="len(blog_post.tag_ids)" class="o_wblog_post_short_tag_section d-flex align-items-center flex-wrap pt-2">
        <t t-foreach="blog_post.tag_ids" t-as="one_tag">
            <a t-attf-href="#{blog_url(tag=tags_list(active_tag_ids, one_tag.id))}"
               t-attf-class="badge mb-2 me-1 text-truncate #{one_tag.id in active_tag_ids and 'bg-primary text-light' or 'border text-primary'} post_link"
               t-att-rel="len(active_tag_ids) and 'nofollow'"
               t-esc="one_tag.name"/>
        </t>
    </div>
    </t>
</template>


<!--   ======================      OPTIONS      ===========================  -->
<!--   ====================================================================  -->
<!-- (Option) Posts List: Show Covers -->
<template id="opt_posts_loop_show_cover" name="Cover" inherit_id="website_blog.posts_loop" active="True"/>

<!-- (Option) Posts List: Show Author -->
<template id="opt_posts_loop_show_author" name="Author" inherit_id="website_blog.posts_loop" active="True"/>

<!-- (Option) Posts List: Show Post Stats -->
<template id="opt_posts_loop_show_stats" name="Comments/Views Stats" inherit_id="website_blog.posts_loop" active="False"/>

<!-- (Option) Posts List: Show Post Teaser -->
<template id="opt_posts_loop_show_teaser" name="Teaser &amp; Tags" inherit_id="website_blog.posts_loop" active="True"/>

</odoo>

```

## File: views\website_blog_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- ======   Overall pages layout   ===========================================
============================================================================ -->
<template id="index" name="Blog Navigation">
    <t t-call="website.layout">
        <div id="wrap" class="js_blog website_blog">
            <t t-out="0"/>

            <!-- Droppable-area shared across all blog's pages -->
            <t t-set="oe_structure_blog_footer_description">Visible in all blogs' pages</t>
            <div class="oe_structure oe_empty oe_structure_not_nearest"
                id="oe_structure_blog_footer"
                t-att-data-editor-sub-message="oe_structure_blog_footer_description"/>
        </div>
    </t>
</template>

<!-- ======  Blog(s) Index : Displaying a list of Blog Posts ===================
Used by 'All blogs' and 'blog' (to share the same layout options) and to display
list of filtered posts (by date or tag).
============================================================================ -->
<template id="blog_post_short" name="Blog Posts">
    <t t-call="website_blog.index">
        <t t-set="head">
            <link t-if="blog" t-att-href="'/blog/%s/feed' % slug(blog)" type="application/atom+xml" rel="alternate" title="Atom Feed"/>
            <meta t-if="active_tag_ids" name="robots" t-attf-content="none"/>
        </t>

        <!-- Check for active options: the stored value may be used in sub-templates too  -->
        <t t-set="opt_blog_cards_design" t-value="is_view_active('website_blog.opt_blog_cards_design')"/>
        <t t-set="opt_blog_list_view" t-value="is_view_active('website_blog.opt_blog_list_view')"/>
        <t t-set="opt_blog_readable" t-value="is_view_active('website_blog.opt_blog_readable')"/>
        <t t-set="opt_blog_sidebar_show" t-value="is_view_active('website_blog.opt_blog_sidebar_show')"/>

        <div id="o_wblog_blog_top">
            <!-- Selectively display droppable-areas for 'all blogs' or single-blog pages -->
            <t t-if="not tag and not date_begin and not search">
                <div id="o_wblog_blog_top_droppable">
                    <t t-if="blog">
                        <t t-set="oe_structure_blog_single_header_description">Edit the '<t t-esc="blog.name"/>' page header.</t>
                        <div t-field="blog.content"
                            class="oe_structure"
                            t-attf-id="oe_structure_blog_single_header_#{blog.id}"
                            t-att-data-editor-sub-message="oe_structure_blog_single_header_description"/>
                    </t>
                    <t t-elif="blogs">
                        <t t-set="oe_structure_blog_all_header_description">Edit the 'All Blogs' page header.</t>
                        <div class="oe_structure"
                            id="oe_structure_blog_all_header"
                            t-att-data-editor-sub-message="oe_structure_blog_all_header_description"/>
                    </t>
                </div>
            </t>
            <t t-else="">
                <!-- Droppable-area for filtered results (tags or date) -->
                <t t-set="oe_structure_blog_filtered_header_description">Edit the 'Filter Results' page header.</t>
                <div class="oe_structure"
                    id="oe_structure_blog_filtered_header"
                    t-att-data-editor-sub-message="oe_structure_blog_filtered_header_description"/>
            </t>
        </div>

        <t t-call="website_blog.blogs_nav">
            <t t-set="search" t-value="original_search or search"/>
            <t t-set="additional_classes" t-value="'mt-3'"/>
        </t>

        <section id="o_wblog_index_content" t-att-class="opt_blog_cards_design and 'o_wblog_page_cards_bg'">
            <div class="container py-4">
                <div t-if="original_search and posts" class="alert alert-warning mt8">
                    No results found for '<span t-esc="original_search"/>'. Showing results for '<span t-esc="search"/>'.
                </div>
                <div t-attf-class="row #{opt_blog_sidebar_show and 'justify-content-between' or 'justify-content-center'}">
                    <div id="o_wblog_posts_loop_container" t-attf-class="col #{'o_container_small mx-0' if opt_blog_list_view else ''}">

                        <t t-call="website_blog.posts_loop">
                            <t t-if="not blog" t-set="posts_list_show_parent_blog" t-value="True"/>
                        </t>

                        <t t-call="website.pager" >
                            <t t-set="classname" t-valuef="justify-content-center"/>
                            <t t-set="extraLinkClass" t-valuef="post_link"/>
                        </t>
                    </div>
                </div>
            </div>
        </section>
    </t>
</template>

<!-- (Option) Blog: Show latest-post as top banner
    Replace top-banner content with the latest published post
-->
<template id="opt_blog_cover_post" name="Top banner - Name / Latest Post" inherit_id="website_blog.blog_post_short" active="True">
    <xpath expr="//div[@id='o_wblog_blog_top_droppable']" position="replace">
        <div t-if="first_post or blog" class="container">
            <div class="row py-4">
                <div t-attf-class="mb-3 mb-md-0 #{'col-md-5' if (not opt_blog_list_view and not opt_blog_sidebar_show) else 'col-md-6'}">
                    <t t-call="website.record_cover">
                        <t t-set="_record" t-value="blog or first_post"/>
                        <t t-set="additionnal_classes" t-value="'h-100 py-5 py-md-0 overflow-hidden rounded shadow'"/>
                    </t>
                </div>
                <div t-att-class="'col-md-7' if (not opt_blog_list_view and not opt_blog_sidebar_show) else 'col-md-6'">
                    <div class="container position-relative h-100 d-flex flex-column justify-content-around pt-1 pb-2">
                        <div t-attf-class="o_wblog_post_title #{'js_tweet' if opt_blog_post_select_to_tweet else ''} #{'js_comment' if opt_blog_post_select_to_comment else ''}">
                            <t t-if="blog">
                                <span t-field="blog.name" class="h1 d-block" placeholder="Blog's Title"/>
                                <div t-field="blog.subtitle" class="h4" placeholder="Subtitle"/>
                            </t>
                            <t t-else="first_post">
                                <div t-if="not date and not tag" class="h4 mb-3 bg-o-color-3 px-2 rounded-1 d-inline-block me-auto">Latest</div>
                                <a t-attf-href="/blog/#{slug(first_post.blog_id)}/#{slug(first_post)}"
                                   t-field="first_post.name" class="h1 d-block" t-att-data-blog-id="first_post.id" placeholder="Blog Post Title"/>
                                <div t-field="first_post.subtitle" class="h4" placeholder="Subtitle"/>

                                <div t-if="not blog" class="d-flex">
                                    <div class="small mt-2 mb-3 me-1">
                                        in <i class="fa fa-folder-open text-muted"/> <a t-attf-href="#{blog_url(blog=first_post.blog_id)}" t-field="first_post.blog_id"/>
                                    </div>
                                </div>
                                <div t-field="first_post.teaser" class="mb-4 lead"  placeholder=""/>
                                <div>
                                    <a t-attf-href="/blog/#{slug(first_post.blog_id)}/#{slug(first_post)}" class="btn btn-primary">Read more</a>
                                </div>
                            </t>
                        </div>
                    </div>
                </div>
                <div class="col-12 mt-3"> <hr/> </div>
            </div>
        </div>
    </xpath>
</template>

<!-- (Option) Blog: Show latest-post as top banner : 'Full Width' design -->
<template id="opt_blog_cover_post_fullwidth_design" name="Full-Width Cover" inherit_id="website_blog.opt_blog_cover_post" active="True">
    <xpath expr="//div[hasclass('container')]" position="replace">
        <t t-if="blog or first_post" t-call="website.record_cover">
            <t t-set="_record" t-value="blog or first_post"/>
            <t t-set="use_filters" t-value="True"/>
            <t t-set="use_text_align" t-value="True"/>
            <t t-set="additionnal_classes" t-value="'o_wblog_post_page_cover o_record_has_cover cover_auto'"/>

            <div class="container position-relative h-100 d-flex flex-column justify-content-around" t-cache="_record">
                <div t-attf-class="o_wblog_post_title #{'js_tweet' if opt_blog_post_select_to_tweet else ''} #{'js_comment' if opt_blog_post_select_to_comment else ''}">
                    <div t-if="not date and not tag and not blog" class="h4 bg-o-color-3 px-2 d-inline-block rounded-1">Latest</div>
                    <a t-if="not blog and first_post" t-attf-href="/blog/#{slug(first_post.blog_id)}/#{slug(first_post)}" t-att-title="first_post.name" class="text-white text-decoration-none">
                        <div t-field="first_post.name" id="o_wblog_post_name" t-att-data-blog-id="first_post.id" placeholder="Blog Post Title"/>
                        <div t-field="first_post.subtitle" id="o_wblog_post_subtitle"  placeholder="Subtitle"/>
                    </a>
                    <span t-elif="blog" t-att-title="blog.name" class="text-white text-decoration-none">
                        <div t-field="blog.name" id="o_wblog_post_name" placeholder="Blog Title"/>
                        <div t-field="blog.subtitle" id="o_wblog_post_subtitle" placeholder="Blog Subtitle"/>
                    </span>

                    <div>
                        <span t-if="not blog and blog_post" class="text-white small mt-2 mb-3">
                            in <i class="fa fa-folder-open text-white-75"/><a t-attf-href="#{blog_url(blog=blog_post.blog_id)}" class="text-white" t-field="blog_post.blog_id"/>
                        </span>
                        <span t-else="">&amp;nbsp;</span>
                    </div>
                </div>
            </div>
        </t>
    </xpath>
</template>


<!-- (Option) Blog: Sidebar : Show -->
<template id="opt_blog_sidebar_show" name="Show Sidebar" inherit_id="website_blog.blog_post_short" active="False">
    <xpath expr="//div[@id='o_wblog_posts_loop_container']" position="after">
        <div t-attf-class="col-12 col-md-3 d-flex #{opt_blog_list_view and 'col-lg-4' or 'ms-lg-5'}">
            <t t-call="website_blog.sidebar_blog_index"/>
        </div>
    </xpath>
</template>

<!-- (Option) Blog: Posts List: Cards design
    Wrap posts in a standard bts cards components
-->
<template id="opt_blog_cards_design" name="'Cards' Design" inherit_id="website_blog.blog_post_short" active="False"/>


<!-- (Option) Blog: Show Posts in list-view
    Display post in a list rather than a grid
-->
<template id="opt_blog_list_view" name="List View" inherit_id="website_blog.blog_post_short" active="False"/>

<!-- (Option) Blog: Increase readability
    Increase font-size, adapt layout
-->
<template id="opt_blog_readable" name="Increase Readability" inherit_id="website_blog.blog_post_short" active="True"/>


<!-- ====== Blog Post Complete Layout ==========================================
============================================================================ -->
<template id="website_blog.blog_post_complete" name="Blog Post" track="1">
    <t t-call="website_blog.index">

        <!-- Check for active options: the stored value may be used in sub-templates too  -->
        <t t-set="opt_blog_post_readable" t-value="is_view_active('website_blog.opt_blog_post_readable')"/>
        <t t-set="opt_blog_post_sidebar" t-value="is_view_active('website_blog.opt_blog_post_sidebar')"/>
        <t t-set="opt_blog_post_regular_cover" t-value="is_view_active('website_blog.opt_blog_post_regular_cover')"/>
        <t t-set="opt_blog_post_breadcrumb" t-value="is_view_active('website_blog.opt_blog_post_breadcrumb')"/>
        <t t-set="opt_blog_post_select_to_tweet" t-value="is_view_active('website_blog.opt_blog_post_select_to_tweet')"/>
        <t t-set="opt_blog_post_select_to_comment" t-value="is_view_active('website_blog.opt_blog_post_select_to_comment')"/>

        <section id="o_wblog_post_top">
            <div id="title" class="blog_header" t-ignore="True">
                <t t-call="website.record_cover">
                    <t t-set="_record" t-value="blog_post"/>
                    <t t-set="snippet_autofocus" t-value="True"/>
                    <t t-set="use_filters" t-value="True"/>
                    <t t-set="use_size" t-value="True"/>
                    <t t-set="display_opt_name">Blog Post Cover</t>
                    <t t-set="additionnal_classes" t-value="'o_wblog_post_page_cover'"/>

                    <div class="container text-center position-relative h-100 d-flex flex-column flex-grow-1 justify-content-around">
                        <div t-attf-class="o_wblog_post_title #{opt_blog_post_select_to_tweet and 'js_tweet'} #{opt_blog_post_select_to_comment and 'js_comment'}">
                            <h1 t-field="blog_post.name" id="o_wblog_post_name" class="o_editable_no_shadow" data-oe-expression="blog_post.name" t-att-data-blog-id="blog_post.id" placeholder="Blog Post Title"/>
                            <div t-field="blog_post.subtitle" id="o_wblog_post_subtitle" class="o_editable_no_shadow" placeholder="Subtitle"/>
                        </div>
                        <t t-set="resize_classes" t-value="set(json.loads(_record.cover_properties).get('resize_class', '').split(' '))"/>
                        <a t-if="{'o_full_screen_height', 'o_half_screen_height', 'cover_full', 'cover_mid'}.intersection(resize_classes)"
                            id="o_wblog_post_content_jump" href="#o_wblog_post_main"
                            class="css_editable_mode_hidden justify-content-center align-items-center rounded-circle mx-auto mb-5 text-decoration-none">
                            <i class="fa fa-angle-down fa-3x text-white" aria-label="To blog content" title="To blog content"/>
                        </a>
                    </div>
                </t>
            </div>
        </section>

        <section id="o_wblog_post_main" t-attf-class="container pt-4 pb-5 #{'anim' in request.params and 'o_wblog_post_main_transition'}">
            <!-- Sidebar-enabled Layout -->
            <div t-if="opt_blog_post_sidebar" t-attf-class="mx-auto #{opt_blog_post_readable and 'o_wblog_read_with_sidebar'}">
                <div t-attf-class="d-flex flex-column flex-lg-row #{opt_blog_post_readable and 'justify-content-between'}">
                    <div id="o_wblog_post_content" t-attf-class="#{opt_blog_post_readable and 'o_container_small mx-0 w-100 flex-shrink-0' or 'w-lg-75'}">
                        <t t-call="website_blog.blog_post_content"/>
                    </div>
                    <div id="o_wblog_post_sidebar_col" t-attf-class="ps-lg-5 #{not opt_blog_post_readable and 'flex-grow-1 w-lg-25'}">
                        <t t-call="website_blog.blog_post_sidebar"/>
                    </div>
                </div>
            </div>

            <!-- No-Sidebar Layout -->
            <div t-if="not opt_blog_post_sidebar" t-attf-class="#{opt_blog_post_readable and 'o_container_small'}">
                <div class="d-flex flex-column flex-lg-row">
                    <div id="o_wblog_post_content" t-attf-class=" #{opt_blog_post_readable and 'o_container_small w-100 flex-shrink-0'}">
                        <t t-call="website_blog.blog_post_content"/>
                    </div>
                </div>
            </div>
        </section>
        <section id="o_wblog_post_footer"/>
    </t>
</template>

<!-- ====== Blog Post Content ==================================================
============================================================================ -->
<template id="blog_post_content" name="Blog post content">
    <t t-if="opt_blog_post_breadcrumb and not opt_blog_post_regular_cover" t-call="website_blog.post_breadcrumbs">
        <t t-set="additionnal_classes" t-value="'mb-3 bg-transparent'"></t>
    </t>
    <div t-field="blog_post.content"
        data-editor-message="WRITE HERE OR DRAG BUILDING BLOCKS"
        t-attf-class="o_wblog_post_content_field #{'js_tweet' if opt_blog_post_select_to_tweet else ''} #{'js_comment' if opt_blog_post_select_to_comment else ''} #{'o_wblog_read_text' if opt_blog_post_readable else ''}"/>

    <div t-if="len(blogs) > 1 or len(blog_post.tag_ids) > 0" class="css_editable_mode_hidden text-muted">
        <div t-if="len(blogs) > 1">in <a t-attf-href="#{blog_url(blog=blog_post.blog_id)}"><b t-field="blog.name"/></a></div>
        <div t-if="len(blog_post.tag_ids) > 0">#
            <t t-foreach="blog_post.tag_ids" t-as="one_tag">
                <a class="badge text-primary border me-1 post_link" t-attf-href="#{blog_url(tag=slug(one_tag), date_begin=False, date_end=False)}" t-esc="one_tag.name"/>
            </t>
        </div>
    </div>
</template>


<!-- (Option) Post: Increase readability
    Increase font-size, adapt content width
-->
<template id="opt_blog_post_readable" name="Increase Readability" inherit_id="website_blog.blog_post_complete" active="True"/>

<!-- (Option) Post: Show Sidebar
    Show sidebar beside the post content
-->
<template id="opt_blog_post_sidebar" name="Show Sidebar" inherit_id="website_blog.blog_post_complete" active="False"/>

<!-- (Option) Post: Regular Cover
    Use 'regular cover' design rather than the fullwidth one
-->
<template id="opt_blog_post_regular_cover" name="'Regular' Cover" inherit_id="website_blog.blog_post_complete" active="False">
    <xpath expr="//div[@id='title']" position="replace">
        <div class="container">
            <t t-set="readableClass" t-if="opt_blog_post_readable and opt_blog_post_sidebar" t-value="'o_wblog_read_with_sidebar mx-auto'"/>
            <t t-set="readableClass" t-elif="opt_blog_post_readable" t-value="'container'"/>

            <div id="title" t-attf-class="blog_header o_wblog_regular_cover_container #{readableClass}">

                <t t-if="opt_blog_post_breadcrumb" t-call="website_blog.post_breadcrumbs">
                    <t t-set="additionnal_classes" t-value="'mt-4 mb-3 bg-transparent'"></t>
                </t>

                <div t-att-class="not opt_blog_post_breadcrumb and 'pt-4'">
                    <div t-attf-class="o_wblog_post_title mb-3 #{'js_tweet' if opt_blog_post_select_to_tweet else ''} #{'js_comment' if opt_blog_post_select_to_comment else ''}" t-ignore="False">
                        <h1 t-field="blog_post.name" id="o_wblog_post_name" data-oe-expression="blog_post.name" t-att-data-blog-id="blog_post.id" placeholder="Title"/>
                        <div t-field="blog_post.subtitle" id="o_wblog_post_subtitle"  placeholder="Subtitle"/>
                    </div>
                    <div class="text-muted mb-2">
                        <i class="fa fa-clock-o fa-fw"/>
                        <span t-field="blog_post.post_date" class="text-muted" t-options="{'format': 'long', 'date_only': 'true'}"/>
                        <span>by
                            <t t-call="website_blog.post_author">
                                <t t-set="additionnal_classes" t-value="'d-inline-flex me-2'"/>
                                <t t-set="hide_date" t-value="True"/>
                            </t>
                        </span>
                        <span t-if="len(blog_post.message_ids) > 0" class="text-nowrap ps-2 o_not_editable">|
                            <i class="fa fa-comment text-muted me-1"/>
                            <a href="#discussion">
                                <t t-esc="len(blog_post.message_ids)"/>
                                <t t-if="len(blog_post.message_ids)>1">Comments</t>
                                <t t-else="">Comment</t>
                            </a>
                        </span>
                        <span t-elif="is_view_active('website_blog.opt_blog_post_comment')">| No comments yet</span>
                    </div>
                </div>

                <t t-call="website.record_cover">
                    <t t-set="_record" t-value="blog_post"/>
                    <t t-set="additionnal_classes" t-value="'o_wblog_post_page_cover o_wblog_post_page_cover_regular rounded shadow overflow-hidden'"/>
                    <t t-set="use_size" t-value="True"/>
                </t>
            </div>
        </div>
    </xpath>
</template>

<!-- (Option) Post: Show Breadcrumb
    Display navigation breadcrumbs before the post content
-->
<template id="opt_blog_post_breadcrumb" name="Show Breadcrumb" inherit_id="website_blog.blog_post_complete" active="True"/>

<!-- (Option) Post: Select text to Tweet
    Allow to select text to tweet it
-->
<template id="opt_blog_post_select_to_tweet" name="Select to Tweet" inherit_id="website_blog.blog_post_complete" active="False"/>

<!-- (Option) Post: Comments
    Enable comments
-->
<template id="opt_blog_post_comment" name="Allow Comments" inherit_id="website_blog.blog_post_complete" active="False">
    <xpath expr="//section[@id='o_wblog_post_main']" position="inside">
        <t t-set="readableClass" t-if="opt_blog_post_readable and opt_blog_post_sidebar" t-value="'o_wblog_read_with_sidebar'"/>
        <t t-set="readableClass" t-elif="opt_blog_post_readable" t-value="'o_container_small'"/>

        <div class="container">
            <div t-attf-class="mx-auto #{readableClass}">
                <div id="o_wblog_post_comments" t-attf-class="pt-4 o_container_small">
                    <div groups="base.group_public" class="small mb-4">
                        <a t-attf-href="/web/login?redirect=/blog/{{slug(blog_post.blog_id)}}/{{slug(blog_post)}}#discussion" class="btn btn-sm btn-primary"><b>Sign in</b></a> to leave a comment
                    </div>
                    <t t-call="portal.message_thread">
                        <t t-set="object" t-value="blog_post"/>
                    </t>
                </div>
            </div>
        </div>
    </xpath>
</template>

<!-- (Option) Post: Comments: Select text to Comment
    Allow to select text to comment it
-->
<template id="opt_blog_post_select_to_comment" name="Select to Comment" inherit_id="website_blog.opt_blog_post_comment" active="False"/>

<!-- (Option) Post : Read Next Article
    Show 'read next' banner at the bottom of the page
-->
<template id="opt_blog_post_read_next" name="Read Next Article" inherit_id="website_blog.blog_post_complete" active="True">
    <xpath expr="//section[@id='o_wblog_post_footer']" position="inside">
        <div t-if="next_post" class="mt-5">
            <t t-if="opt_blog_post_regular_cover">
                <t t-if="opt_blog_post_sidebar" t-set="readableClass" t-value="'o_wblog_read_with_sidebar'"/>
                <t t-else="" t-set="readableClass" t-value="'o_container_small'"/>

                <div class="container">
                    <div t-attf-class="mb-4 mx-auto #{ readableClass if opt_blog_post_readable else ''}">
                        <hr/>
                        <div class="d-flex text-end py-4">
                            <div class="flex-grow-1 pe-3">
                                <span class="bg-o-color-3 h6 d-inline-block py-1 px-2 rounded-1">Read Next</span>
                                <a t-att-href="'/blog/' + slug(next_post.blog_id) + '/' + slug(next_post)" t-att-title="'Read next' + next_post.name">
                                    <div t-field="next_post.name" id="o_wblog_post_name" t-att-data-blog-id="next_post.id" placeholder="Blog Post Title" class="h2"/>
                                    <div t-field="next_post.subtitle" id="o_wblog_post_subtitle" placeholder="Subtitle" class="lead"/>
                                </a>
                            </div>
                            <a t-att-href="'/blog/' + slug(next_post.blog_id) + '/' + slug(next_post)" t-att-title="'Read next' + next_post.name" class="w-25 flex-shrink-0">
                                <t t-call="website.record_cover">
                                    <t t-set="_record" t-value="next_post"/>
                                    <t t-set="additionnal_classes" t-value="'rounded shadow-sm overflow-hidden h-100'"/>
                                </t>
                            </a>
                        </div>
                    </div>
                </div>
            </t>
            <t t-else="">
                <div id="o_wblog_next_container" class="d-flex flex-column" t-cache="next_post">
                    <t t-call="website.record_cover">
                        <t t-set="_record" t-value="next_post"/>
                        <t t-set="_cp" t-value="json.loads(_record.cover_properties)"/>
                        <t t-set="use_filters" t-value="True"/>
                        <t t-set="additionnal_classes" t-value="'o_wblog_post_page_cover o_wblog_post_page_cover_footer o_record_has_cover'"/>

                        <a id="o_wblog_next_post_info" class="d-none"
                           t-att-data-size="_cp.get('resize_class')"
                           t-att-data-url="'/blog/' + slug(next_post.blog_id) + '/' + slug(next_post) + '?anim'"/>

                        <t t-set="next_cover_is_full" t-value="bool({'o_full_screen_height', 'cover_full'}.intersection(_cp.get('resize_class', '').split(' ')))"/>
                        <t t-set="next_cover_is_auto" t-value="'cover_auto' in _cp.get('resize_class', '')"/>

                        <div class="container text-center position-relative h-100 d-flex flex-column flex-grow-1 justify-content-around">
                            <div t-attf-class="o_wblog_post_title">
                                <div t-field="next_post.name" id="o_wblog_post_name" t-att-data-blog-id="next_post.id" placeholder="Blog Post Title" class="h1"/>
                                <div t-field="next_post.subtitle" id="o_wblog_post_subtitle"  placeholder="Subtitle"/>
                            </div>

                            <div t-attf-class="o_wblog_toggle #{next_cover_is_full and 'mb-n5'}">
                                <span class="h4 d-inline-block py-1 px-2 rounded-1 text-white">
                                    <i class="fa fa-angle-right fa-3x text-white" aria-label="Read next" title="Read Next"/>
                                </span>
                            </div>

                            <!-- Emulate the next post's cover's height. For non-auto covers,
                            the room that will be occupied by the 'scroll-down' link is temporary
                            occupied  by the loader circle. For auto covers, an empty <div>
                            creates enought separation.
                            -->
                            <div t-if="not next_cover_is_auto" class="o_wblog_next_loader o_wblog_toggle justify-content-center align-items-center mx-auto position-relative d-none">
                                <div class="rounded-circle bg-black-50"/>
                            </div>
                            <div t-else="" class="o_wblog_next_fake_btn d-flex o_wblog_toggle"/>
                        </div>
                    </t>
                </div>
            </t>
        </div>
    </xpath>
</template>

<!-- ======   Technical Templates   ============================================
============================================================================ -->

<!-- Atom Feed -->
<template id="blog_feed">&lt;?xml version="1.0" encoding="utf-8"?&gt;
<feed t-att-xmlns="'http://www.w3.org/2005/Atom'">
    <title t-esc="blog.name"/>
    <link t-att-href="'%s/blog/%s' % (base_url ,blog.id)"/>
    <id t-esc="'%s/blog/%s' % (base_url, blog.id)"/>
    <updated t-esc="str(posts[0].post_date).replace(' ', 'T') + 'Z' if posts else ''"/>
    <entry t-foreach="posts" t-as="post">
        <title t-esc="post.name"/>
        <link t-att-href="'%s%s' % (base_url, post.website_url)"/>
        <id t-esc="'%s%s' % (base_url, post.website_url)"/>
        <author><name t-esc="post.sudo().author_id.name"/></author>
        <summary t-esc="html2plaintext(post.teaser)"/>
        <updated t-esc="str(post.post_date).replace(' ', 'T') + 'Z'"/>
    </entry>
</feed>
</template>

</odoo>

```

## File: views\website_blog_views.xml

```xml
<?xml version="1.0"?>
<odoo>
        <!-- Blog views -->
        <record id="view_blog_blog_list" model="ir.ui.view">
            <field name="name">blog.blog.list</field>
            <field name="model">blog.blog</field>
            <field name="arch" type="xml">
                <tree string="Blogs">
                    <field name="name"/>
                    <field name="blog_post_count"/>
                    <field name="website_id" groups="website.group_multi_website"/>
                    <field name="active" column_invisible="True"/>
                </tree>
            </field>
        </record>
        <record id="view_blog_blog_form" model="ir.ui.view">
            <field name="name">blog.blog.form</field>
            <field name="model">blog.blog</field>
            <field name="arch" type="xml">
                <form string="Blog">
                    <sheet>
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="name"/>
                            <field name="subtitle"/>
                            <field name="website_id" options="{'no_create': True}" groups="website.group_multi_website"/>
                        </group>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" groups="base.group_user"/>
                        <field name="message_ids"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="blog_blog_view_search" model="ir.ui.view">
            <field name="name">blog.blog.search</field>
            <field name="model">blog.blog</field>
            <field name="arch" type="xml">
                <search string="Blog">
                    <field name="name"/>
                    <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                </search>
            </field>
        </record>

        <record id="action_blog_blog" model="ir.actions.act_window">
            <field name="name">Blogs</field>
            <field name="res_model">blog.blog</field>
            <field name="view_mode">tree,form</field>
        </record>

        <record id="blog_tag_tree" model="ir.ui.view">
            <field name="name">blog_tag_tree</field>
            <field name="model">blog.tag</field>
            <field name="arch" type="xml">
                <tree string="Tag List">
                    <field name="name"/>
                    <field name="category_id"/>
                    <field name="post_ids"/>
                </tree>
            </field>
        </record>

        <record id="blog_tag_form" model="ir.ui.view">
            <field name="name">blog_tag_form</field>
            <field name="model">blog.tag</field>
            <field name="arch" type="xml">
                <form string="Tag Form">
                    <sheet>
                        <group>
                            <field name="name"/>
                            <field name="category_id"/>
                        </group>
                        <label for="post_ids" string="Used in: "/>
                        <field name="post_ids"/>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="action_tags" model="ir.actions.act_window">
            <field name="name">Blog Tags</field>
            <field name="res_model">blog.tag</field>
            <field name="view_mode">tree,form</field>
            <field name="view_id" ref="blog_tag_tree"/>
        </record>

        <record id="blog_tag_category_form" model="ir.ui.view">
            <field name="name">blog_tag_category_form</field>
            <field name="model">blog.tag.category</field>
            <field name="arch" type="xml">
                <form string="Tag Category Form">
                    <sheet>
                        <group>
                            <field name="name"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="blog_tag_category_tree" model="ir.ui.view">
            <field name="name">blog_tag_category_tree</field>
            <field name="model">blog.tag.category</field>
            <field name="arch" type="xml">
                <tree string="Tag Categories">
                    <field name="name"/>
                </tree>
            </field>
        </record>

        <record id="action_tag_category" model="ir.actions.act_window">
            <field name="name">Tag Category</field>
            <field name="res_model">blog.tag.category</field>
            <field name="view_mode">tree,form</field>
            <field name="view_id" ref="blog_tag_category_tree"/>
        </record>

        <menuitem name="Blog"
            id="menu_website_blog_root_global"
            sequence="100"
            parent="website.menu_website_global_configuration"
            groups="website.group_website_designer"/>

        <menuitem id="menu_blog_global" parent="menu_website_blog_root_global" name="Blogs" action="action_blog_blog" sequence="20"/>

        <menuitem id="menu_blog_tag_global" parent="menu_website_blog_root_global" name="Tags" action="action_tags" sequence="30"/>

        <menuitem id="menu_website_blog_tag_category_global" parent="menu_website_blog_root_global"
                  name="Tag Categories" action="action_tag_category" sequence="40"/>
</odoo>

```

## File: views\website_pages_views.xml

```xml
<?xml version="1.0"?>
<odoo>

<record id="view_blog_post_form" model="ir.ui.view">
    <field name="name">blog.post.form</field>
    <field name="model">blog.post</field>
    <field name="arch" type="xml">
        <form string="Blog Post">
            <sheet>
                <div class="oe_button_box" name="button_box" invisible="not active">
                    <field name="is_published" widget="website_redirect_button"/>
                </div>
                <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                <group name="blog_details">
                    <field name="blog_id"/>
                    <field name="active" invisible="1"/>
                    <field name="name" placeholder="Blog Post Title"/>
                    <field name="subtitle" placeholder="Blog Subtitle"/>
                    <field name="tag_ids" widget="many2many_tags"/>
                    <field name="website_id" groups="website.group_multi_website"/>
                </group>
                <group name="publishing_details" string="Publishing Options">
                    <field name="author_id"/>
                    <field name="create_date" groups="base.group_no_one"/>
                    <field name="visits"/>
                    <field name="post_date"/>
                    <field name="write_uid"/>
                    <field name="write_date"/>
                </group>
                <notebook>
                    <page name="seo" string="SEO" groups="base.group_no_one">
                        <group name="default_opengraph">
                            <field name="website_meta_title" string="Meta Title"/>
                            <field name="website_meta_description" string="Meta Description"/>
                            <field name="website_meta_keywords" string="Meta Keywords" help="Separate every keyword with a comma"/>
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

<record id="blog_post_view_kanban" model="ir.ui.view">
    <field name="name">blog.post.kanban</field>
    <field name="model">blog.post</field>
    <field name="arch" type="xml">
        <kanban js_class="website_pages_kanban" class="o_kanban_mobile" action="open_website_url" type="object" sample="1">
            <field name="name"/>
            <field name="blog_id"/>
            <field name="author_id"/>
            <field name="post_date"/>
            <field name="website_url"/>
            <templates>
                <t t-name="kanban-box">
                    <div class="oe_kanban_global_click d-flex flex-column">
                        <div class="row mb-auto">
                            <strong class="col-8">
                                <span class="o_text_overflow" t-esc="record.name.value"/>
                                <div class="text-muted" t-if="record.website_id.value" groups="website.group_multi_website">
                                    <i class="fa fa-globe me-1" title="Website"/>
                                    <field name="website_id"/>
                                </div>
                            </strong>
                            <strong class="col-4 text-end">
                                <span t-esc="record.blog_id.value"/>
                            </strong>
                            <div class="col-8">
                                <i class="fa fa-clock-o me-1" role="img" aria-label="Post date" title="Post date"/><span t-esc="record.post_date.value"/>
                            </div>
                            <div class="col-4 text-end">
                                <img t-if="record.author_id.raw_value"
                                     t-att-title="record.author_id.value"
                                     t-att-alt="record.author_id.value"
                                     class="oe_kanban_avatar o_avatar rounded"
                                     t-att-src="kanban_image('res.partner', 'avatar_128', record.author_id.raw_value)"/>
                            </div>
                        </div>
                        <div class="border-top mt-2 pt-2">
                            <field name="is_published" widget="boolean_toggle"/>
                            <t t-if="record.is_published.raw_value">Published</t>
                            <t t-else="">Not Published</t>
                        </div>
                    </div>
                </t>
            </templates>
        </kanban>
    </field>
</record>

<record id="view_blog_post_search" model="ir.ui.view">
    <field name="name">blog.post.search</field>
    <field name="model">blog.post</field>
    <field name="arch" type="xml">
        <search string="Blog Post">
            <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
            <field name="name" string="Content" filter_domain="['|', ('name','ilike',self), ('content','ilike',self)]"/>
            <field name="write_uid"/>
            <field name="blog_id"/>
            <group expand="0" string="Group By">
                <filter string="Blog" name="group_by_blog" domain="[]" context="{'group_by': 'blog_id'}"/>
                <filter string="Author" name="group_by_author" domain="[]" context="{'group_by': 'create_uid'}"/>
                <filter string="Last Contributor" name="last_contributor" domain="[]" context="{'group_by': 'write_uid'}"/>
            </group>
        </search>
    </field>
</record>

<record id="view_blog_post_list" model="ir.ui.view">
    <field name="name">Blog Post Pages Tree</field>
    <field name="model">blog.post</field>
    <field name="priority">99</field>
    <field name="arch" type="xml">
        <tree js_class="website_pages_list" type="object" action="open_website_url" multi_edit="1">
            <field name="active" column_invisible="True"/>
            <field name="name"/>
            <field name="website_url"/>

            <field name="author_id" optional="show"/>
            <field name="blog_id" optional="hide"/>
            <field name="create_uid" optional="hide"/>
            <field name="write_uid" optional="hide"/>
            <field name="write_date" optional="hide"/>

            <field name="is_seo_optimized"/>
            <field name="is_published"/>

            <field name="website_id" groups="website.group_multi_website"/>
        </tree>
    </field>
</record>

<record id="action_blog_post" model="ir.actions.act_window">
    <field name="name">Blog Post Pages</field>
    <field name="res_model">blog.post</field>
    <field name="view_mode">tree,kanban,form</field>
    <field name="view_ids" eval="[(5, 0, 0),
        (0, 0, {'view_mode': 'tree', 'view_id': ref('view_blog_post_list')}),
        (0, 0, {'view_mode': 'kanban', 'view_id': ref('blog_post_view_kanban')}),
    ]"/>
    <field name="search_view_id" ref="view_blog_post_search"/>
    <field name="context">{'create_action': 'website_blog.blog_post_action_add'}</field>
</record>

<menuitem id="menu_blog_post_pages"
    parent="website.menu_content"
    sequence="20"
    name="Blog Posts"
    action="action_blog_post"/>

</odoo>

```

## File: views\snippets\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="website_blog" inherit_id="website.snippets" name="Snippet Blog">
    <xpath expr="//t[@id='blog_posts_hook']" position="replace">
        <t t-snippet="website_blog.s_blog_posts" string="Blog Posts" t-thumbnail="/website_blog/static/src/img/s_blog_posts.svg"/>
    </xpath>
</template>

<template id="snippet_options" inherit_id="website.snippet_options" name="Blog snippet options">
    <xpath expr="." position="inside">
        <div data-js="BlogPostTagSelection" data-selector=".o_wblog_post_page_cover" data-target="#o_wblog_post_name" data-no-check="true">
            <we-many2many string="Tags"
                data-no-preview="true"
                data-model="blog.post"
                data-m2o-field="tag_ids"
                data-set-tags=""
                data-create-method="createTag"/>
        </div>
        <!-- Blog Posts page  -->
        <div data-selector="main:has(#o_wblog_index_content)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Blogs Page">
            <we-select string="Top Banner" data-no-preview="true" data-reload="/">
                <we-button data-customize-website-views="website_blog.opt_blog_cover_post"
                    data-name="blog_cover_opt">Name / Latest Post</we-button>
                <we-button data-customize-website-views="">Drop Zone for Building Blocks</we-button>
            </we-select>
            <we-checkbox string="Full-Width"
                         class="o_we_sublevel_1"
                         data-customize-website-views="website_blog.opt_blog_cover_post_fullwidth_design"
                         data-dependencies="blog_cover_opt"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-select string="Layout" data-no-preview="true" data-reload="/">
                <we-button data-customize-website-views="">Grid</we-button>
                <we-button data-customize-website-views="website_blog.opt_blog_list_view">List</we-button>
            </we-select>
            <we-checkbox string="Cards"
                         class="o_we_sublevel_1"
                         data-customize-website-views="website_blog.opt_blog_cards_design"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Increase Readability"
                         class="o_we_sublevel_1"
                         data-customize-website-views="website_blog.opt_blog_readable"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Sidebar"
                         data-name="blog_posts_sidebar_opt"
                         data-customize-website-views="website_blog.opt_blog_sidebar_show"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Archives"
                         class="o_we_sublevel_1"
                         data-dependencies="blog_posts_sidebar_opt"
                         data-customize-website-views="website_blog.opt_sidebar_blog_index_archives"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Follow us"
                         class="o_we_sublevel_1"
                         data-dependencies="blog_posts_sidebar_opt"
                         data-customize-website-views="website_blog.opt_sidebar_blog_index_follow_us"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Tags List"
                         class="o_we_sublevel_1"
                         data-dependencies="blog_posts_sidebar_opt"
                         data-customize-website-views="website_blog.opt_sidebar_blog_index_tags"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-select string="Posts List" data-no-preview="true" data-reload="/">
                <we-button data-customize-website-views="">No Cover</we-button>
                <we-button data-customize-website-views="website_blog.opt_posts_loop_show_cover">Cover</we-button>
            </we-select>
            <we-checkbox string="Author"
                         class="o_we_sublevel_1"
                         data-customize-website-views="website_blog.opt_posts_loop_show_author"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Comments/Views Stats"
                         class="o_we_sublevel_1"
                         data-customize-website-views="website_blog.opt_posts_loop_show_stats"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Teaser &amp; Tags"
                         class="o_we_sublevel_1"
                         data-customize-website-views="website_blog.opt_posts_loop_show_teaser"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
        <!-- Blog Post page  -->
        <div data-selector="main:has(#o_wblog_post_main)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Blog Page">
            <we-select string="Layout" data-no-preview="true" data-reload="/">
                <we-button data-customize-website-views="website_blog.opt_blog_post_regular_cover">Title Above Cover</we-button>
                <we-button data-customize-website-views="">Title Inside Cover</we-button>
            </we-select>
            <we-checkbox string="Increase Readability"
                         class="o_we_sublevel_1"
                         data-customize-website-views="website_blog.opt_blog_post_readable"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Sidebar"
                         data-name="blog_post_sidebar_opt"
                         data-customize-website-views="website_blog.opt_blog_post_sidebar"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Archive"
                         class="o_we_sublevel_1"
                         data-dependencies="blog_post_sidebar_opt"
                         data-customize-website-views="website_blog.opt_blog_post_archive_display"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Author"
                         class="o_we_sublevel_1"
                         data-dependencies="blog_post_sidebar_opt"
                         data-customize-website-views="website_blog.opt_blog_post_author_avatar_display"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Blogs List"
                         class="o_we_sublevel_1"
                         data-dependencies="blog_post_sidebar_opt"
                         data-customize-website-views="website_blog.opt_blog_post_blogs_display"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Share Links"
                         class="o_we_sublevel_1"
                         data-dependencies="blog_post_sidebar_opt"
                         data-customize-website-views="website_blog.opt_blog_post_share_links_display"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Tags"
                         class="o_we_sublevel_1"
                         data-dependencies="blog_post_sidebar_opt"
                         data-customize-website-views="website_blog.opt_blog_post_tags_display"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Breadcrumb"
                         data-customize-website-views="website_blog.opt_blog_post_breadcrumb"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-row string="Bottom" class="o_we_full_row">
                <we-button string="Next Article"
                           data-customize-website-views="website_blog.opt_blog_post_read_next"
                           data-no-preview="true"
                           data-reload="/"/>
                <we-button string="Comments"
                           data-name="allow_comments_opt"
                           data-customize-website-views="website_blog.opt_blog_post_comment"
                           data-no-preview="true"
                           data-reload="/"/>
            </we-row>
            <we-checkbox string="Select to Comment"
                         class="o_we_sublevel_1"
                         data-dependencies="allow_comments_opt"
                         data-customize-website-views="website_blog.opt_blog_post_select_to_comment"
                         data-no-preview="true"
                         data-reload="/"/>
            <we-checkbox string="Select to Tweet"
                         data-customize-website-views="website_blog.opt_blog_post_select_to_tweet"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
    </xpath>
    <xpath expr="//*[@data-js='anchor']" position="attributes">
        <attribute name="data-exclude" add=".o_wblog_post_content_field > :not(div, section)" separator=","/>
    </xpath>

    <!-- Hides ContainerWidth option for content in blog posts -->
    <xpath expr="//div[@data-js='ContainerWidth']" position="attributes">
        <attribute name="data-exclude" add="#o_wblog_post_content *" separator=","/>
    </xpath>
</template>

<template id="blog_searchbar_input_snippet_options" inherit_id="website.searchbar_input_snippet_options" name="blog search bar snippet options">
    <xpath expr="//div[@data-js='SearchBar']/we-select[@data-name='scope_opt']" position="inside">
        <we-button data-set-search-type="blogs" data-select-data-attribute="blogs" data-name="search_blogs_opt" data-form-action="/blog">Blogs</we-button>
    </xpath>
    <xpath expr="//div[@data-js='SearchBar']/we-select[@data-name='order_opt']" position="inside">
        <we-button data-set-order-by="published_date asc" data-select-data-attribute="published_date asc" data-dependencies="search_blogs_opt" data-name="order_published_date_asc_opt">Date (old to new)</we-button>
        <we-button data-set-order-by="published_date desc" data-select-data-attribute="published_date desc" data-dependencies="search_blogs_opt" data-name="order_published_date_desc_opt">Date (new to old)</we-button>
    </xpath>
    <xpath expr="//div[@data-js='SearchBar']/div[@data-dependencies='limit_opt']" position="inside">
        <we-checkbox string="Description" data-dependencies="search_blogs_opt" data-select-data-attribute="true" data-attribute-name="displayDescription"
            data-apply-to=".search-query"/>
        <we-checkbox string="Publication Date" data-dependencies="search_blogs_opt" data-select-data-attribute="true" data-attribute-name="displayDetail"
            data-apply-to=".search-query"/>
    </xpath>
</template>
</odoo>

```

## File: views\snippets\s_blog_posts.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<!-- Snippet -->
<template id="s_blog_posts" name="Blog Posts">
    <t t-call="website.s_dynamic_snippet_template">
        <t t-set="snippet_name" t-value="'s_blog_posts'"/>
        <t t-set="snippet_classes" t-value="'s_dynamic_snippet_blog_posts s_blog_post_big_picture s_blog_posts_effect_marley'"/>
    </t>
</template>

<!-- Load-time templates (rendered in JS on page load) -->
<!-- List layout -->
<template id="dynamic_filter_template_blog_post_list" name="List layout">
    <div t-foreach="records" t-as="data" class="d-flex mt-3 s_blog_posts_post" data-number-of-elements="2">
        <t t-set="record" t-value="data['_record']"/>
        <a class="s_blog_posts_post_cover flex-grow-0 flex-shrink-0 align-self-baseline position-relative" t-att-href="data['call_to_action_url']" t-att-title="'Read' + data['name']">
            <t t-call="website.record_cover">
                <t t-set="_record" t-value="record"/>
                <t t-set="_resize_height" t-value="256"/>
                <t t-set="_resize_width" t-value="256"/>
                <t t-set="additionnal_classes" t-value="'w-100 h-100 bg-200 position-absolute'"/>
            </t>
        </a>
        <div class="ps-2">
            <a class="" t-att-title="'Read' + data['name']" t-att-href="data['call_to_action_url']">
                <div class="s_blog_posts_post_title mb-1">
                    <span t-if="is_sample" class="bg-primary text-uppercase px-1">Sample</span>
                    <span t-field="record.name"/>
                </div>
            </a>
            <div class="s_blog_posts_post_subtitle mb-1 d-none d-sm-block" t-field="record.subtitle"/>
        </div>
    </div>
</template>
<!-- Big picture layout -->
<template id="dynamic_filter_template_blog_post_big_picture" name="Big picture layout">
    <figure t-foreach="records" t-as="data" class="my-3 w-100 s_blog_posts_post" data-number-of-elements="3">
        <t t-set="record" t-value="data['_record']"/>
        <a class="s_blog_posts_post_cover position-relative d-flex flex-column shadow-sm overflow-hidden rounded text-decoration-none" t-att-href="data['call_to_action_url']">
            <t t-call="website.record_cover">
                <t t-set="_record" t-value="record"/>
                <t t-set="_resize_height" t-value="512"/>
                <t t-set="_resize_width" t-value="512"/>
                <t t-set="use_filters" t-value="True"/>
                <t t-set="additionnal_classes" t-value="'h-100 w-100 bg-600 position-absolute'"/>
            </t>

            <figcaption class="text-center w-100 h-100 px-3 d-flex flex-column flex-grow-1">
                <div t-if="is_sample" class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                <div class="s_blog_posts_post_title text-white" t-field="record.name"/>
                <div class="s_blog_posts_post_subtitle text-white" t-field="record.subtitle"/>
            </figcaption>
        </a>
    </figure>
</template>
<!-- Horizontal layout -->
<template id="dynamic_filter_template_blog_post_horizontal" name="Horizontal layout">
    <figure t-foreach="records" t-as="data" class="post s_blog_posts_post w-100" data-number-of-elements="3">
        <t t-set="record" t-value="data['_record']"/>
        <span t-if="is_sample" class="h5 float-end bg-primary text-uppercase rounded-circle px-2 py-2">Sample</span>
        <figcaption>
            <h4 class="mb0">
                <a t-att-href="data['call_to_action_url']" t-field="record.name"/>
            </h4>
            <h5 class="mt0 mb4" t-field="record.post_date" t-options='{"format": "dd/MM"}'/>
        </figcaption>
        <a t-att-href="data['call_to_action_url']">
            <t t-call="website.record_cover">
                <t t-set="_record" t-value="record"/>
                <t t-set="_resize_height" t-value="512"/>
                <t t-set="_resize_width" t-value="512"/>
                <t t-set="additionnal_classes" t-value="'thumb'"/>
            </t>
        </a>
    </figure>
</template>
<!-- Card layout -->
<template id="dynamic_filter_template_blog_post_card" name="Card layout">
    <div t-foreach="records" t-as="data" class="s_blog_posts_post pb32 w-100" data-number-of-elements="3">
        <t t-set="record" t-value="data['_record']"/>
        <div class="card">
            <a class="s_blog_posts_post_cover" t-att-href="data['call_to_action_url']">
                <t t-call="website.record_cover">
                    <t t-set="_record" t-value="record"/>
                    <t t-set="_resize_height" t-value="512"/>
                    <t t-set="_resize_width" t-value="512"/>
                    <t t-set="additionnal_classes" t-value="'thumb'"/>
                </t>
            </a>
            <div class="card-body">
                <div t-if="is_sample" class="h5 o_ribbon_right bg-primary text-uppercase">Sample</div>
                <a t-att-href="data['call_to_action_url']"><h4 class="mb-0" t-field="record.name"/></a>
            </div>
            <div class="card-footer d-flex justify-content-between">
                <span class="text-muted mb-0" t-field="record.post_date" t-options='{"format": "MMM d, yyyy"}' />
                <span class="text-muted mb-0">In <a class="fw-bold" t-field="record.blog_id.name" t-att-href="'/blog/%s' % record.blog_id.id" />
                    <a t-if="is_sample" class="fw-bold" href="#">Sample</a>
                </span>
            </div>
        </div>
    </div>
</template>

<!-- Options -->
<template id="s_blog_posts_options" inherit_id="website.snippet_options">
    <xpath expr="." position="inside">
        <t t-call="website_blog.s_dynamic_snippet_options_template">
            <t t-set="snippet_name" t-value="'dynamic_snippet_blog_posts'"/>
            <t t-set="snippet_selector" t-value="'.s_dynamic_snippet_blog_posts'"/>
        </t>
    </xpath>
    <xpath expr="//div[@data-js='layout_column']" position="attributes">
        <attribute name="data-exclude" add=".s_blog_posts, .s_blog_posts_big_picture" separator=","/>
    </xpath>
</template>

<template id="s_dynamic_snippet_options_template" inherit_id="website.s_dynamic_snippet_options_template">
    <xpath expr="//we-select[@data-name='filter_opt']" position="after">
        <we-select t-if="snippet_name == 'dynamic_snippet_blog_posts'" string="Blog" data-no-preview="true" data-name="blog_opt" data-attribute-name="filterByBlogId">
            <we-button data-select-data-attribute="-1">All blogs</we-button>
            <!-- the blog list will be generated in js -->
        </we-select>
    </xpath>
    <xpath expr="//we-select[@data-name='template_opt']" position="after">
        <we-select t-if="snippet_name == 'dynamic_snippet_blog_posts'"
                   string="Hover effect" class="o_we_sublevel_1 o_we_inline"
                   data-no-widget-refresh="true" data-name="hover_effect_opt">
            <we-button data-select-class="">None</we-button>
            <we-button data-select-class="s_blog_posts_effect_marley">Marley</we-button>
            <we-button data-select-class="s_blog_posts_effect_dexter">Dexter</we-button>
            <we-button data-select-class="s_blog_posts_effect_chico">Silly-Chico</we-button>
        </we-select>
    </xpath>
</template>

<!-- Assets -->
<record id="website_blog.s_blog_posts_000_scss" model="ir.asset">
    <field name="name">Blog posts 000 SCSS</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">website_blog/static/src/snippets/s_blog_posts/000.scss</field>
</record>

<record id="website_blog.s_blog_posts_000_js" model="ir.asset">
    <field name="name">Blog posts 000 JS</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">website_blog/static/src/snippets/s_blog_posts/000.js</field>
</record>

</odoo>

```

