# Odoo Module: website_slides_forum

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Forum on Courses',
    'category': 'Hidden',
    'version': '1.0',
    'summary': 'Allows to link forum on a course',
    'description': """A Slide channel can be linked to forum. Also, profiles from slide and forum are regrouped together""",
    'depends': [
        'website_slides',
        'website_forum'
    ],
    'data': [
        'security/ir.model.access.csv',
        'views/forum_views.xml',
        'views/slide_channel_views.xml',
        'views/website_slides_menu_views.xml',
        'views/assets.xml',
        'views/website_slides_templates.xml',
        'views/website_slides_forum_templates.xml'
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

    def _slide_channel_prepare_values(self, **kwargs):
        channel = super(WebsiteSlidesForum, self)._slide_channel_prepare_values(**kwargs)
        if bool(kwargs.get('link_forum')):
            forum = request.env['forum.forum'].create({
                'name': kwargs.get('name')
            })
            channel['forum_id'] = forum.id
        return channel

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
        <field name="name">What is the best fertilizer for tulips ?</field>
        <field name="forum_id" ref="forum_forum_demo_channel_0"/>
        <field name="views">2</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="write_uid" ref="base.user_admin"/>
        <field name="content" type="html"><p></p></field>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=31)"/>
    </record>
    <record id="forum_post_O_0_answer_0" model="forum.post">
        <field name="forum_id" ref="forum_forum_demo_channel_0"/>
        <field name="content" type="html"><p>You can use loam for tulips.</p></field>
        <field name="parent_id" ref="forum_post_O_0"/>
    </record>

    <record id="forum_post_2_0" model="forum.post">
        <field name="name">Heigth of my tree...</field>
        <field name="forum_id" ref="forum_forum_demo_channel_2"/>
        <field name="views">1</field>
        <field name="create_uid" ref="base.user_demo"/>
        <field name="write_uid" ref="base.user_demo"/>
        <field name="content" type="html"><p>I have an oak in my garden since 1997 and I was wondering about the growth of this type of tree ?
            Is there a way to accelerate the process ?</p></field>
    </record>

</data></odoo>

```

## File: models\forum.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Forum(models.Model):
    _inherit = 'forum.forum'

    slide_channel_ids = fields.One2many('slide.channel', 'forum_id', 'Courses')
    slide_channel_id = fields.Many2one('slide.channel', 'Course', compute='_compute_slide_channel_id')

    @api.depends('slide_channel_ids')
    def _compute_slide_channel_id(self):
        for forum in self:
            if forum.slide_channel_ids:
                forum.slide_channel_id = forum.slide_channel_ids[0]
            else:
                forum.slide_channel_id = None

```

## File: models\slide_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Channel(models.Model):
    _inherit = 'slide.channel'

    forum_id = fields.Many2one('forum.forum', 'Course Forum')
    forum_total_posts = fields.Integer('Number of active forum posts', related="forum_id.total_posts")

    _sql_constraints = [
        ('forum_uniq', 'unique (forum_id)', "Only one forum per slide channel!"),
    ]

    def action_redirect_to_forum(self):
        self.ensure_one()
        action = self.env.ref('website_forum.action_forum_post').read()[0]
        action['view_mode'] = 'tree'
        action['context'] = {
            'create': False
        }
        action['domain'] = [('forum_id', '=', self.forum_id.id)]

        return action

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import forum
from . import slide_channel

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_forum_forum_website_publisher,access.forum.forum.website.publisher,model_forum_forum,website.group_website_publisher,1,1,1,0

```

## File: static\src\js\website_slides_forum.editor.js

```javascript
odoo.define('website_slides_forum.editor', function (require) {
"use strict";

var core = require('web.core');
var QWeb = core.qweb;
var WebsiteNewMenu = require('website.newMenu');

 WebsiteNewMenu.include({
    xmlDependencies: WebsiteNewMenu.prototype.xmlDependencies.concat(
        ['/website_slides_forum/static/src/xml/website_slides_forum_channel.xml']
    ),
});
});

```

## File: static\src\xml\website_slides_forum_channel.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-extend="website.slide.channel.create">
        <t t-jquery="#communication-label" t-operation="replace">
            <label id="communication-label">Communication</label>
        </t>
        <t t-jquery=".o_wslide_channel_communication_type" t-operation="append">
            <div class="form-check">
                <input class="form-check-input" type="checkbox" id="link_forum" name="link_forum"/>
                <span class="form-check-label" for="link_forum">Create a Forum</span>
            </div>
        </t>
    </t>
</templates>

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_frontend" inherit_id="website.assets_frontend" name="Website slide forum frontend assets">
        <xpath expr="//script[last()]" position="after">
            <script type="text/javascript" src="/website_slides_forum/static/src/js/website_slides_forum.editor.js"></script>
        </xpath>
    </template>
</odoo>

```

## File: views\forum_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="forum_forum_view_form" model="ir.ui.view">
        <field name="name">forum.forum.view.form.inherit.slides</field>
        <field name="model">forum.forum</field>
        <field name="inherit_id" ref="website_forum.view_forum_forum_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='group_order']" position="after">
                <group string="eLearning" name="group_slides">
                    <field name="slide_channel_id" readonly="True"/>
                </group>
            </xpath>
        </field>
    </record>

    <record id="forum_forum_action_channel" model="ir.actions.act_window">
        <field name="name">eLearning Forums</field>
        <field name="res_model">forum.forum</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('slide_channel_ids', '!=', 'False')]</field>
    </record>

    <record id="forum_post_action_channel" model="ir.actions.act_window">
        <field name="name">eLearning Forum Posts</field>
        <field name="res_model">forum.post</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('forum_id.slide_channel_ids', '!=', 'False')]</field>
        <field name="context">{'search_default_questions': 1}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new forum post
            </p>
        </field>
    </record>

    <record id="forum_post_view_graph_slides" model="ir.ui.view">
        <field name="name">forum.post.view.graph.slides</field>
        <field name="model">forum.post</field>
        <field name="arch" type="xml">
            <graph string="eLearning Forum Posts">
                <field name="create_date" interval="month" type="col"/>
                <field name="forum_id" type="row"/>
            </graph>
        </field>
    </record>

    <record id="forum_post_action_report" model="ir.actions.act_window">
        <field name="name">eLearning Forum Posts</field>
        <field name="res_model">forum.post</field>
        <field name="view_mode">graph</field>
        <field name="view_id" ref="forum_post_view_graph_slides"/>
        <field name="domain">[('forum_id.slide_channel_ids', '!=', 'False')]</field>
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
                    attrs="{'invisible': [('forum_id', '=', False)]}"
                    icon="fa-comment">
                    <field string="Forum Posts" name="forum_total_posts" widget="statinfo"/>
                </button>
            </xpath>
            <xpath expr="//field[@name='allow_comment']" position="after">
                <field string="Forum" name="forum_id"/>
    		</xpath>
        </field>
    </record>
</odoo>

```

## File: views\website_slides_forum_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <template id="website_slides_forum_header" inherit_id="website_forum.header">
        <xpath expr="//div[hasclass('o_forum_nav_header_container')]" position="before">
            <t t-if="forum.slide_channel_id">
                <div class="o_wforum_elearning_navtabs_container">
                    <div class="container">
                        <ul class="nav nav-tabs o_wprofile_nav_tabs mt-0 flex-nowrap" role="tablist" id="profile_extra_info_tablist">
                            <li class="nav-item">
                                <a t-att-href="'/slides/%s%s' % (slug(forum.slide_channel_id), '/%s' % slug(category) if category else '')"
                                    t-att-class="'nav-link o_wprofile_navlink'" style="border-left: 0px">
                                    <i class="fa fa-home"/> Course</a>
                            </li>
                            <li class="nav-item">
                                <a t-att-href="'/forum/%s' % (slug(forum))" t-att-class="'nav-link active o_wprofile_navlink'" style="border-left: 0px">Forum</a>
                            </li>
                            <li t-if="forum.slide_channel_id.allow_comment" class="nav-item">
                                <a t-att-href="'/slides/%s?active_tab=review' % (slug(forum.slide_channel_id))"
                                    t-att-class="'nav-link o_wprofile_navlink'" style="border-left: 0px">
                                    Review
                                </a>
                            </li>
                        </ul>
                    </div>
                </div>
            </t>
        </xpath>
    </template>

    <template name="Split regular forums/courses" id="website_slides_forum_index" inherit_id="website_forum.forum_all" active="True" customize_show="True">
        <xpath expr="//div[@id='o_wforum_forums_index_list']/*[@t-if='forums']" position="replace">
            <t t-if="forums">
                <t t-set="courses_discussions" t-value="forums.filtered(lambda f: f.slide_channel_id)"/>
                <t t-set="regular_forums" t-value="forums - courses_discussions"/>

                <div class="row mb-4">
                    <t t-if="len(regular_forums) == 1">
                        <t t-set="forum" t-value="regular_forums"/>
                        <div class="col-md-10 col-lg-8 mb-2">
                            <div class="row align-items-start">
                                <div class="col-12 col-sm-4">
                                    <a t-attf-href="/forum/#{slug(forum)}">
                                        <div class="oe_img_bg rounded shadow-sm py-4 px-5 o_wforum_forum_card_bg flex-shrink-0"
                                             t-attf-style="#{forum.image_256 and ('background-image: url(%s);' % website.image_url(forum, 'image_256'))} background-position: center;">
                                            <div class="p-5"/>
                                        </div>
                                    </a>
                                </div>
                                <div class="col-12 col-sm-7 mt-2 mt-sm-0">
                                    <a t-attf-href="/forum/#{slug(forum)}" class="text-reset" t-att-title="forum.name">
                                        <h3 class="h2" t-field="forum.name"/>
                                    </a>
                                    <p t-attf-class="m-0 lead #{not forum.description and 'css_non_editable_mode_hidden'}"
                                       placeholder="Description"
                                       t-field="forum.description"/>
                                </div>
                            </div>
                        </div>
                    </t>
                    <t t-else="">
                        <t t-call="website_forum.forum_all_all_entries">
                            <t t-set="_forums" t-value="regular_forums"/>
                        </t>
                    </t>
                </div>

                <div class="d-flex border-bottom pb-1 pt-3 mb-3">
                    <h2 class="h4 mb-0 text-muted">Courses Discussions</h2>
                    <div class="flex-grow-1 text-right">
                        <a href="/slides" title="See all Courses">Check our Courses <i class="fa fa-chevron-right"/></a>
                    </div>
                </div>
                <div class="row">
                    <t t-call="website_forum.forum_all_all_entries">
                        <t t-set="_forums" t-value="courses_discussions"/>
                    </t>
                </div>
            </t>
        </xpath>
    </template>
</data></odoo>

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

    <menuitem name="Forum"
        id="website_slides_menu_report_forum"
        parent="website_slides.website_slides_menu_report"
        sequence="8"
        action="forum_post_action_report"/>
</odoo>

```

## File: views\website_slides_templates.xml

```xml
<?xml version="1.0" ?>
<odoo><data>
    <template id='course_main' inherit_id="website_slides.course_main">
    <!-- Channel (training) main template: add link to forum -->
    <xpath expr="//li[hasclass('o_wslides_course_header_nav_home_training')]" position="after">
        <li class="nav-item" t-if="channel.forum_id">
            <a t-att-href="'/forum/%s' % (slug(channel.forum_id))"
                t-att-class="'nav-link'" target="new">Forum</a>
        </li>
    </xpath>
    <!-- Channel (documentation) main template: add link to forum -->
    <xpath expr="//li[hasclass('o_wslides_course_header_nav_home_documentation')]" position="after">
            <li class="nav-item" t-if="channel.forum_id">
                <a t-att-href="'/forum/%s' % (slug(channel.forum_id))"
                    t-att-class="'nav-link'" target="new">Forum</a>
            </li>
        </xpath>
    </template>

    <template id="slide_fullscreen" inherit_id="website_slides.slide_fullscreen">
        <xpath expr="//a[hasclass('o_wslides_fs_review')]" position="after">
            <a t-if="slide.channel_id.forum_id" id="fullscreen_forum_button" class="d-flex align-items-center px-3" t-attf-href="/forum/#{slug(slide.channel_id.forum_id)}" target="new">
                <i class="fa fa-comments"/><span class="ml-1 d-none d-md-inline-block">Ask a question</span>
            </a>
        </xpath>
    </template>

</data></odoo>

```

