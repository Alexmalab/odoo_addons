# Odoo Module: website_slides_forum

Category: Website/eLearning

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import populate

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Forum on Courses',
    'category': 'Website/eLearning',
    'version': '1.0',
    'summary': 'Allows to link forum on a course',
    'description': """A Slide channel can be linked to forum. Also, profiles from slide and forum are regrouped together""",
    'depends': [
        'website_slides',
        'website_forum'
    ],
    'data': [
        'security/ir.model.access.csv',
        'security/website_slides_forum_security.xml',
        'views/forum_forum_views.xml',
        'views/forum_post_views.xml',
        'views/res_config_settings_views.xml',
        'views/slide_channel_views.xml',
        'views/website_slides_menu_views.xml',
        'views/forum_forum_templates.xml',
        'views/website_slides_templates.xml',
        'views/snippets.xml',
    ],
    'demo': [
        'data/slide_channel_demo.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request
from odoo.addons.website_slides.controllers.main import WebsiteSlides


class WebsiteSlidesForum(WebsiteSlides):

    # Profile
    # ---------------------------------------------------

    def _prepare_user_profile_parameters(self, **post):
        post = super(WebsiteSlidesForum, self)._prepare_user_profile_parameters(**post)
        if post.get('channel_id'):
            channel = request.env['slide.channel'].browse(int(post.get('channel_id')))
            if channel.forum_id:
                post.update({
                    'forum_id': channel.forum_id.id,
                    'no_forum': False
                })
            else:
                post.update({'no_forum': True})
        return post

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main

```

## File: data\slide_channel_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <record id="forum_forum_demo_channel_0" model="forum.forum">
        <field name="name">Basics of Gardening</field>
    </record>
    <record id="website_slides.slide_channel_demo_0_gard_0" model="slide.channel">
        <field name="forum_id" ref="website_slides_forum.forum_forum_demo_channel_0"/>
    </record>

	<record id="forum_forum_demo_channel_2" model="forum.forum">
        <field name="name">Trees, Wood and Gardens</field>
    </record>
	<record id="website_slides.slide_channel_demo_2_gard2" model="slide.channel">
        <field name="forum_id" ref="website_slides_forum.forum_forum_demo_channel_2"/>
	</record>

    <record id="forum_post_O_0" model="forum.post">
        <field name="name">What is the best fertilizer for tulips?</field>
        <field name="forum_id" ref="forum_forum_demo_channel_0"/>
        <field name="views">2</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="write_uid" ref="base.user_admin"/>
        <field name="content" type="html"><p></p></field>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=31)"/>
    </record>
    <record id="forum_post_O_0_answer_0" model="forum.post">
        <field name="name">Re: What is the best fertilizer for tulips?</field>
        <field name="forum_id" ref="forum_forum_demo_channel_0"/>
        <field name="content" type="html"><p>You can use loam for tulips.</p></field>
        <field name="parent_id" ref="forum_post_O_0"/>
    </record>

    <record id="forum_post_2_0" model="forum.post">
        <field name="name">Height of my tree...</field>
        <field name="forum_id" ref="forum_forum_demo_channel_2"/>
        <field name="views">1</field>
        <field name="create_uid" ref="base.user_demo"/>
        <field name="write_uid" ref="base.user_demo"/>
        <field name="content" type="html"><p>I have an oak in my garden since 1997 and I was wondering about the growth of this type of tree ?
            Is there a way to accelerate the process?</p></field>
    </record>

</data></odoo>

```

## File: models\forum_forum.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Forum(models.Model):
    _inherit = 'forum.forum'

    slide_channel_ids = fields.One2many('slide.channel', 'forum_id', 'Courses', help="Edit the course linked to this forum on the course form.")
    slide_channel_id = fields.Many2one('slide.channel', 'Course', compute='_compute_slide_channel_id', store=True)
    visibility = fields.Selection(related='slide_channel_id.visibility', help="Forum linked to a Course, the visibility is the one applied on the course.")
    image_1920 = fields.Image('Image', compute='_compute_image_1920', store=True, readonly=False)

    @api.depends('slide_channel_ids')
    def _compute_slide_channel_id(self):
        for forum in self:
            if forum.slide_channel_ids:
                forum.slide_channel_id = forum.slide_channel_ids[0]
            else:
                forum.slide_channel_id = None

    @api.depends('slide_channel_id', 'slide_channel_id.image_1920')
    def _compute_image_1920(self):
        for forum in self.filtered(lambda f: not f.image_1920 and f.slide_channel_id.image_1920):
            forum.image_1920 = forum.slide_channel_id.image_1920

```

## File: models\slide_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Channel(models.Model):
    _inherit = 'slide.channel'

    forum_id = fields.Many2one('forum.forum', 'Course Forum', copy=False)
    forum_total_posts = fields.Integer('Number of active forum posts', related="forum_id.total_posts")

    _sql_constraints = [
        ('forum_uniq', 'unique (forum_id)', "Only one course per forum!"),
    ]

    def action_redirect_to_forum(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("website_forum.forum_post_action")
        action['view_mode'] = 'tree'
        action['context'] = {
            'create': False
        }
        action['domain'] = [('forum_id', '=', self.forum_id.id)]

        return action

    @api.model_create_multi
    def create(self, vals_list):
        channels = super(Channel, self.with_context(mail_create_nosubscribe=True)).create(vals_list)
        channels.forum_id.privacy = False
        return channels

    def write(self, vals):
        old_forum = self.forum_id

        res = super(Channel, self).write(vals)
        if 'forum_id' in vals:
            self.forum_id.privacy = False
            if old_forum != self.forum_id:
                old_forum.write({
                    'privacy': 'private',
                    'authorized_group_id': self.env.ref('website_slides.group_website_slides_officer').id,
                })
        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import forum_forum
from . import slide_channel

```

## File: populate\forum_forum.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from itertools import zip_longest

from odoo import models
from odoo.addons.website_slides.populate.slide_channel import SlideChannel


class SlidesForum(models.Model):
    _inherit = 'forum.forum'

    @property
    def _populate_sizes(self):
        return {size: count + SlideChannel._populate_sizes[size] for size, count in super()._populate_sizes.items()}

    @property
    def _populate_dependencies(self):
        return super()._populate_dependencies + ['slide.channel']

    def _populate_factories(self):
        def link_course(iterator, *args, **kwargs):
            courses = self.env['slide.channel'].browse(self.env.registry.populated_models['slide.channel'])
            for values, course in zip_longest(iterator, courses):
                if course:
                    values.update(slide_channel_ids=course, name=f"{course.name}'s Forum")
                yield values

        return super()._populate_factories() + [
            ('_name_and_course', link_course),
        ]

```

## File: populate\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import forum_forum

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_forum_forum_website_slides_officer,access.forum.forum.website.publisher,model_forum_forum,website_slides.group_website_slides_officer,1,1,1,0

```

## File: security\website_slides_forum_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="website_slides_forum_public" model="ir.rule">
        <field name="name">Website forum: User can only access to forum related to public courses</field>
        <field name="model_id" ref="website_forum.model_forum_forum"/>
        <field name="domain_force">[('slide_channel_ids.website_published', '=', True), ('slide_channel_ids.visibility', '=', 'public')]</field>
        <field name="groups" eval="[(4, ref('base.group_public'))]"/>
    </record>
    <record id="website_slides_forum_signed_in_user" model="ir.rule">
        <field name="name">Website forum: Signed In user can only access to forum related to courses</field>
        <field name="model_id" ref="website_forum.model_forum_forum"/>
        <field name="domain_force">[
            '&amp;',
                ('slide_channel_ids.website_published', '=', True),
                '|',
                    ('slide_channel_ids.visibility', 'in', ('public','connected')),
                    ('slide_channel_ids.is_member', '=', True)
            ]
        </field>
        <field name="groups" eval="[(4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
    </record>
    <record id="website_slides_forum_website_slides_officer" model="ir.rule">
        <field name="name">Website forum: website slides officer can access all forum</field>
        <field name="model_id" ref="website_forum.model_forum_forum"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('website_slides.group_website_slides_officer'))]"/>
    </record>

    <record id="website_slides_forum_public_post" model="ir.rule">
        <field name="name">Website forum post: User can only access to post linked to forum related to followed courses</field>
        <field name="model_id" ref="website_forum.model_forum_post"/>
        <field name="domain_force">[('forum_id.slide_channel_ids.website_published', '=', True), ('forum_id.slide_channel_ids.visibility', '=', 'public')]</field>
        <field name="groups" eval="[(4, ref('base.group_public'))]"/>
    </record>
    <record id="website_slides_forum_post_signed_in_user" model="ir.rule">
        <field name="name">Website forum: Signed In user can only access to post linked to forum related to courses</field>
        <field name="model_id" ref="website_forum.model_forum_post"/>
        <field name="domain_force">[
            '&amp;',
                ('forum_id.slide_channel_ids.website_published', '=', True),
                '|',
                    ('forum_id.slide_channel_ids.visibility', 'in', ('public','connected')),
                    ('forum_id.slide_channel_ids.is_member', '=', True)
            ]
        </field>
        <field name="groups" eval="[(4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
    </record>
    <record id="website_slides_forum_website_slides_officer_post" model="ir.rule">
        <field name="name">Website forum post: website slides officer can access all post</field>
        <field name="model_id" ref="website_forum.model_forum_post"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('website_slides.group_website_slides_officer'))]"/>
    </record>

    <record id="website_slides_forum_public_tag" model="ir.rule">
        <field name="name">Website slides forum tag: Public User can only access to tag linked to forum related to public courses</field>
        <field name="model_id" ref="website_forum.model_forum_tag"/>
        <field name="domain_force">[('forum_id.slide_channel_ids.website_published', '=', True), ('forum_id.slide_channel_ids.visibility', '=', 'public')]</field>
        <field name="groups" eval="[(4, ref('base.group_public'))]"/>
    </record>
    <record id="website_slides_forum_tag_signed_in_user" model="ir.rule">
        <field name="name">Website forum: Signed In users can access tags linked to public or connected users-visibility courses</field>
        <field name="model_id" ref="website_forum.model_forum_tag"/>
        <field name="domain_force">[
            '&amp;',
                ('forum_id.slide_channel_ids.website_published', '=', True),
                '|',
                    ('forum_id.slide_channel_ids.visibility', 'in', ('public','connected')),
                    ('forum_id.slide_channel_ids.is_member', '=', True)
            ]
        </field>
        <field name="groups" eval="[(4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
    </record>
    <record id="website_slides_forum_website_slides_officer_tag" model="ir.rule">
        <field name="name">Website slides forum tag: website slides officer can access all tag</field>
        <field name="model_id" ref="website_forum.model_forum_tag"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('website_slides.group_website_slides_officer'))]"/>
    </record>
</odoo>

```

## File: views\forum_forum_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <template name="Split regular forums/courses" id="website_slides_forum_index" inherit_id="website_forum.forum_all" active="True">
        <xpath expr="//div[@id='o_wforum_forums_index_list']//*[@t-if='forums']" position="replace">
            <t t-if="forums">
                <t t-set="courses_discussions" t-value="forums.filtered(lambda f: f.slide_channel_id)"/>
                <t t-set="regular_forums" t-value="forums - courses_discussions"/>
                <t t-call="website_forum.forum_all_all_entries">
                    <t t-set="_forums" t-value="regular_forums"/>
                </t>
                <t t-call="website_forum.forum_all_all_entries">
                    <t t-set="_forums" t-value="courses_discussions"/>
                </t>
            </t>
        </xpath>
    </template>
    <template name="Replace Breadcrumb Root" id="website_slides_forum_breadcrumb" inherit_id="website_forum.forum_model_nav" active="True">
        <xpath expr="//nav[@id='o_wforum_nav']" position="before">
            <t t-if="forum and forum.slide_channel_id" t-set="breadcrumb_kind" t-value="'slides'"/>
        </xpath>
        <xpath expr="//div[hasclass('o_wforum_breadcrumb_root_single')]" position="inside">
            <ol t-if="breadcrumb_kind == 'slides'" class="breadcrumb order-first col-10 col-lg flex-grow-1 flex-nowrap my-0 p-0 fs-5">
                <li class="breadcrumb-item">
                    <a t-attf-href="/slides/#{slug(forum.slide_channel_id)}#{'/' + slug(category) if category else ''}" t-out="forum.name"/>
                </li>
                <li class="breadcrumb-item text-nowrap">
                    <a t-attf-href="/forum/#{ slug(forum) }">Forum</a>
                </li>
            </ol>
        </xpath>
        <xpath expr="//div[hasclass('o_wforum_breadcrumb_root_list_or_edit')]" position="inside">
            <ol t-if="breadcrumb_kind == 'slides'" class="breadcrumb order-first col-10 col-lg flex-grow-1 flex-nowrap my-0 p-0 fs-5">
                <li class="breadcrumb-item text-nowrap">
                    <a t-attf-href="/slides/#{slug(forum.slide_channel_id)}#{'/' + slug(category) if category else ''}" t-out="forum.name"/>
                </li>
                <li class="breadcrumb-item text-nowrap">
                    <strong>Forum</strong>
                </li>
            </ol>
        </xpath>
    </template>

    <template name="Add course badge" id="forum_all_all_entries" inherit_id="website_forum.forum_all_all_entries">
        <xpath expr="//t[@t-foreach='sorted_forums']" position="before">
            <t t-if="_forums == courses_discussions">
                <t t-set="forum_badge">Course</t>
            </t>
        </xpath>
        <xpath expr="//t[@t-foreach='sorted_forums']" position="after">
            <div t-if="_forums == courses_discussions" class="d-flex justify-content-center">
                <a href="/slides" class="btn btn-link">Check out our courses <i class="fa fa-long-arrow-right"/></a>
            </div>
        </xpath>
    </template>
</data>
</odoo>

```

## File: views\forum_forum_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="forum_forum_view_form" model="ir.ui.view">
        <field name="name">forum.forum.view.form.inherit.slides</field>
        <field name="model">forum.forum</field>
        <field name="inherit_id" ref="website_forum.forum_forum_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='group_order']" position="after">
                <group string="eLearning" name="group_slides" invisible="not slide_channel_id">
                    <field name="slide_channel_id" readonly="True"/>
                </group>
            </xpath>
            <xpath expr="//field[@name='privacy']" position="attributes">
                <attribute name="invisible">slide_channel_id</attribute>
                <attribute name="required">slide_channel_id == 'False'</attribute>
            </xpath>
            <xpath expr="//field[@name='authorized_group_id']" position="attributes">
                <attribute name="invisible">privacy != 'private' or slide_channel_id</attribute>
                <attribute name="required">privacy == 'private'</attribute>
            </xpath>
            <xpath expr="//field[@name='privacy']" position="before">
                <field name="visibility" invisible="not slide_channel_id"/>
            </xpath>
        </field>
    </record>

    <record id="forum_forum_view_tree_slides" model="ir.ui.view">
        <field name="name">forum.forum.view.tree.slides</field>
        <field name="model">forum.forum</field>
        <field name="mode">primary</field>
        <field name="priority" eval="20"/>
        <field name="inherit_id" ref="website_forum.forum_forum_view_tree"/>
        <field name="arch" type="xml">
            <field name="website_id" position="after">
                <field name="slide_channel_id"/>
                <field name="visibility"/>
            </field>
        </field>
    </record>

    <record id="forum_forum_action_channel" model="ir.actions.act_window">
        <field name="name">Forums</field>
        <field name="res_model">forum.forum</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('slide_channel_ids', '!=', 'False')]</field>
        <field name="view_id" ref="forum_forum_view_tree_slides"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Forum
            </p>
            <p>Forums allow your attendees to ask questions to your community.</p>
        </field>
    </record>

</odoo>

```

## File: views\forum_post_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="forum_post_action_channel" model="ir.actions.act_window">
        <field name="name">Forum Posts</field>
        <field name="res_model">forum.post</field>
        <field name="view_mode">tree,graph,pivot,form</field>
        <field name="domain">[('forum_id.slide_channel_ids', '!=', 'False')]</field>
        <field name="context">{'search_default_questions': 1}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No Forum Post yet!
            </p>
            <p>Come back later to monitor and moderate what is posted on your Forums.</p>
        </field>
    </record>

    <record id="forum_post_view_graph_slides" model="ir.ui.view">
        <field name="name">forum.post.view.graph.slides</field>
        <field name="model">forum.post</field>
        <field name="arch" type="xml">
            <graph string="eLearning Forum Posts" sample="1">
                <field name="create_date" interval="month"/>
                <field name="forum_id"/>
            </graph>
        </field>
    </record>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.slides.forum</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website_slides.res_config_settings_view_form"/>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <xpath expr="//setting[@id='website_slide_install_website_slides_forum']" position="inside">
                <div class="mt8">
                    <button type="action" name="%(website_slides_forum.forum_forum_action_channel)d" string="Manage Forums" class="btn-link" icon="oi-arrow-right"/>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\slide_channel_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="website_slides_forum_channel_inherit_view_form" model="ir.ui.view">
        <field name="name">website.slides_forum.view.form.inherit.slide.channel</field>
        <field name="model">slide.channel</field>
        <field name="inherit_id" ref="website_slides.view_slide_channel_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_view_ratings']" position="after">
                <button name="action_redirect_to_forum"
                    type="object"
                    class="oe_stat_button"
                    invisible="not forum_id"
                    icon="fa-comment">
                    <field string="Forum Posts" name="forum_total_posts" widget="statinfo"/>
                </button>
            </xpath>
            <xpath expr="//field[@name='allow_comment']" position="after">
                <field string="Forum" name="forum_id" domain="[('slide_channel_id', 'in', [id, False])]"/>
    		</xpath>
        </field>
    </record>
</odoo>

```

## File: views\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="snippet_options" inherit_id="website.snippet_options" name="Slides Forum Snippet Options">
    <xpath expr="." position="inside">
        <div data-selector="main:has(#o_wforum_forums_index_list)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Forum Page">
            <we-checkbox string="Separate Courses"
                         class="o_we_sublevel_1"
                         data-customize-website-views="website_slides_forum.website_slides_forum_index"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
    </xpath>
</template>

</odoo>

```

## File: views\website_slides_menu_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem name="Forum"
        id="website_slides_menu_forum"
        parent="website_slides.website_slides_menu_root"
        sequence="2"/>
    <menuitem name="Forums"
        id="website_slides_menu_forum_forum"
        parent="website_slides_menu_forum"
        sequence="1"
        action="forum_forum_action_channel"/>
    <menuitem name="Posts"
        id="website_slides_menu_forum_post"
        parent="website_slides_menu_forum"
        sequence="2"
        action="forum_post_action_channel"/>
</odoo>

```

## File: views\website_slides_templates.xml

```xml
<?xml version="1.0" ?>
<odoo><data>
    <template id='course_main' inherit_id="website_slides.course_main">
        <!-- Channel main template: add link to forum -->
        <xpath expr="//li[hasclass('o_wslides_course_header_nav_review')]" position="after">
            <li class="nav-item" t-if="not invite_preview and (channel.is_member or channel.visibility == 'public') and channel.forum_id">
                <a t-att-href="'/forum/%s' % (slug(channel.forum_id))"
                    t-att-class="'nav-link'" target="new">Forum</a>
            </li>
        </xpath>
    </template>

    <template id="slide_fullscreen" inherit_id="website_slides.slide_fullscreen">
        <xpath expr="//a[hasclass('o_wslides_fs_review')]" position="after">
            <a t-if="(channel.is_member or channel.visibility == 'public') and slide.channel_id.forum_id" id="fullscreen_forum_button"
                class="d-flex align-items-center px-3" t-attf-href="/forum/#{slug(slide.channel_id.forum_id)}"
                target="new" title="Forum">
                <i class="fa fa-comments"/><span class="ms-1 d-none d-md-inline-block">Forum</span>
            </a>
        </xpath>
    </template>

</data></odoo>

```

