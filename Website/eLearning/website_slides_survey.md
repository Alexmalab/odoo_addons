# Odoo Module: website_slides_survey

Category: Website/eLearning

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers

def uninstall_hook(env):
    dt = env.ref('website_slides.badge_data_certification_goal', raise_if_not_found=False)
    if dt:
        dt.domain = "[('completed', '=', True), (0, '=', 1)]"

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Course Certifications",
    'summary': 'Add certification capabilities to your courses',
    'description': """This module lets you use the full power of certifications within your courses.""",
    'category': 'Website/eLearning',
    'version': '1.0',
    'depends': ['website_slides', 'survey'],
    'installable': True,
    'auto_install': True,
    'data': [
        'security/ir.model.access.csv',
        'views/slide_channel_views.xml',
        'views/slide_slide_partner_views.xml',
        'views/slide_slide_views.xml',
        'views/survey_survey_views.xml',
        'views/website_slides_menu_views.xml',
        'views/website_slides_templates_course.xml',
        'views/website_slides_templates_lesson.xml',
        'views/website_slides_templates_lesson_fullscreen.xml',
        'views/website_slides_templates_homepage.xml',
        'views/website_slides_templates_utils.xml',
        'views/survey_templates.xml',
        'views/website_profile.xml',
        'data/mail_template_data.xml',
        'data/gamification_data.xml',
        'views/res_config_settings_views.xml',
    ],
    'demo': [
        'data/survey_demo.xml',
        'data/slide_slide_demo.xml',
        'data/survey.user_input.line.csv',
    ],
    'assets': {
        'web.assets_frontend': [
            'website_slides_survey/static/src/scss/website_slides_survey.scss',
            'website_slides_survey/static/src/js/slides_upload.js',
            'website_slides_survey/static/src/js/slides_course_fullscreen_player.js',
            'website_slides_survey/static/src/xml/website_slide_upload.xml',
            'website_slides_survey/static/src/xml/website_slides_fullscreen.xml',
        ],
        'survey.survey_assets': [
            'website_slides_survey/static/src/js/survey_form.js',
            'website_slides_survey/static/src/scss/website_slides_survey_result.scss',
        ],
    },
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\slides.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import werkzeug
import werkzeug.utils
import werkzeug.exceptions

from odoo import _
from odoo import http
from odoo.addons.http_routing.models.ir_http import slug
from odoo.exceptions import AccessError
from odoo.http import request
from odoo.osv import expression

from odoo.addons.website_slides.controllers.main import WebsiteSlides


class WebsiteSlidesSurvey(WebsiteSlides):

    @http.route(['/slides_survey/slide/get_certification_url'], type='http', auth='user', website=True)
    def slide_get_certification_url(self, slide_id, **kw):
        fetch_res = self._fetch_slide(slide_id)
        if fetch_res.get('error'):
            raise werkzeug.exceptions.NotFound()
        slide = fetch_res['slide']
        if slide.channel_id.is_member:
            slide.action_set_viewed()
        certification_url = slide._generate_certification_url().get(slide.id)
        if not certification_url:
            raise werkzeug.exceptions.NotFound()
        return request.redirect(certification_url)

    @http.route(['/slides_survey/certification/search_read'], type='json', auth='user', methods=['POST'], website=True)
    def slides_certification_search_read(self, fields):
        can_create = request.env['survey.survey'].check_access_rights('create', raise_exception=False)
        return {
            'read_results': request.env['survey.survey'].search_read([('certification', '=', True)], fields),
            'can_create': can_create,
        }

    # ------------------------------------------------------------
    # Overrides
    # ------------------------------------------------------------

    @http.route()
    def create_slide(self, *args, **post):
        create_new_survey = post['slide_category'] == "certification" and post.get('survey') and not post['survey']['id']
        linked_survey_id = int(post.get('survey', {}).get('id') or 0)

        if create_new_survey:
            # If user cannot create a new survey, no need to create the slide either.
            if not request.env['survey.survey'].check_access_rights('create', raise_exception=False):
                return {'error': _('You are not allowed to create a survey.')}

            # Create survey first as certification slide needs a survey_id (constraint)
            post['survey_id'] = request.env['survey.survey'].create({
                'title': post['survey']['title'],
                'questions_layout': 'page_per_question',
                'is_attempts_limited': True,
                'attempts_limit': 1,
                'is_time_limited': False,
                'scoring_type': 'scoring_without_answers',
                'certification': True,
                'scoring_success_min': 70.0,
                'certification_mail_template_id': request.env.ref('survey.mail_template_certification').id,
            }).id
        elif linked_survey_id:
            try:
                request.env['survey.survey'].browse([linked_survey_id]).read(['title'])
            except AccessError:
                return {'error': _('You are not allowed to link a certification.')}

            post['survey_id'] = post['survey']['id']

        # Then create the slide
        result = super(WebsiteSlidesSurvey, self).create_slide(*args, **post)

        if post['slide_category'] == "certification":
            # Set the url to redirect the user to the survey
            result['url'] = '/slides/slide/%s?fullscreen=1' % (slug(request.env['slide.slide'].browse(result['slide_id']))),

        return result

    # Utils
    # ---------------------------------------------------
    def _slide_mark_completed(self, slide):
        if slide.slide_category == 'certification':
            raise werkzeug.exceptions.Forbidden(_("Certification slides are completed when the survey is succeeded."))
        return super(WebsiteSlidesSurvey, self)._slide_mark_completed(slide)

    def _get_valid_slide_post_values(self):
        result = super(WebsiteSlidesSurvey, self)._get_valid_slide_post_values()
        result.append('survey_id')
        return result

    # Profile
    # ---------------------------------------------------
    def _prepare_user_slides_profile(self, user):
        values = super(WebsiteSlidesSurvey, self)._prepare_user_slides_profile(user)
        values.update({
            'certificates': self._get_users_certificates(user)[user.id]
        })
        return values

    # All Users Page
    # ---------------------------------------------------
    def _prepare_all_users_values(self, users):
        result = super(WebsiteSlidesSurvey, self)._prepare_all_users_values(users)
        certificates_per_user = self._get_users_certificates(users)
        for index, user in enumerate(users):
            result[index].update({
                'certification_count': len(certificates_per_user.get(user.id, []))
            })
        return result

    def _get_users_certificates(self, users):
        partner_ids = [user.partner_id.id for user in users]
        domain = [
            ('slide_partner_id.partner_id', 'in', partner_ids),
            ('scoring_success', '=', True),
            ('slide_partner_id.survey_scoring_success', '=', True)
        ]
        certificates = request.env['survey.user_input'].sudo().search(domain)
        users_certificates = {
            user.id: [
                certificate for certificate in certificates if certificate.partner_id == user.partner_id
            ] for user in users
        }
        return users_certificates

    # Badges & Ranks Page
    # ---------------------------------------------------
    def _prepare_ranks_badges_values(self, **kwargs):
        """ Extract certification badges, to render them in ranks/badges page in another section.
        Order them by number of granted users desc and show only badges linked to opened certifications."""
        values = super(WebsiteSlidesSurvey, self)._prepare_ranks_badges_values(**kwargs)

        # 1. Getting all certification badges, sorted by granted user desc
        domain = expression.AND([[('survey_id', '!=', False)], self._prepare_badges_domain(**kwargs)])
        certification_badges = request.env['gamification.badge'].sudo().search(domain)
        # keep only the badge with challenge category = slides (the rest will be displayed under 'normal badges' section
        certification_badges = certification_badges.filtered(
            lambda b: 'slides' in b.challenge_ids.mapped('challenge_category'))

        if not certification_badges:
            return values

        # 2. sort by granted users (done here, and not in search directly, because non stored field)
        certification_badges = certification_badges.sorted("granted_users_count", reverse=True)

        # 3. Remove certification badge from badges
        badges = values['badges'] - certification_badges

        # 4. Getting all course url for each badge
        certification_slides = request.env['slide.slide'].sudo().search([('survey_id', 'in', certification_badges.mapped('survey_id').ids)])
        certification_badge_urls = {slide.survey_id.certification_badge_id.id: slide.channel_id.website_url for slide in certification_slides}

        # 5. Applying changes
        values.update({
            'badges': badges,
            'certification_badges': certification_badges,
            'certification_badge_urls': certification_badge_urls
        })
        return values

```

## File: controllers\survey.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.survey.controllers import main


class Survey(main.Survey):
    def _prepare_survey_finished_values(self, survey, answer, token=False):
        result = super(Survey, self)._prepare_survey_finished_values(survey, answer, token)
        if answer.slide_id:
            result['channel_id'] = answer.slide_id.channel_id

        return result

    def _prepare_retry_additional_values(self, answer):
        result = super(Survey, self)._prepare_retry_additional_values(answer)
        if answer.slide_id:
            result['slide_id'] = answer.slide_id.id
        if answer.slide_partner_id:
            result['slide_partner_id'] = answer.slide_partner_id.id

        return result

```

## File: controllers\website_profile.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request
from odoo.osv import expression
from odoo.addons.website_profile.controllers.main import WebsiteProfile


class WebsiteSlidesSurvey(WebsiteProfile):
    def _prepare_user_profile_values(self, user, **kwargs):
        """Loads all data required to display the certification attempts of the given user"""
        values = super(WebsiteSlidesSurvey, self)._prepare_user_profile_values(user, **kwargs)
        values['show_certification_tab'] = ('user' in values) and (
            values['user'].id == request.env.user.id or \
            request.env.user.has_group('survey.group_survey_manager')
        )

        if not values['show_certification_tab']:
            return values

        domain = expression.AND([
            [('survey_id.certification', '=', True)],
            [('state', '=', 'done')],
            expression.OR([
                [('email', '=', values['user'].email)],
                [('partner_id', '=', values['user'].partner_id.id)]
            ]) if values['user'].email else \
                [('partner_id', '=', values['user'].partner_id.id)]
        ])

        if 'certification_search' in kwargs:
            values['active_tab'] = 'certification'
            values['certification_search_terms'] = kwargs['certification_search']
            domain = expression.AND([domain,
                [('survey_id.title', 'ilike', kwargs['certification_search'])]
            ])

        UserInputSudo = request.env['survey.user_input'].sudo()
        values['user_inputs'] = UserInputSudo.search(domain, order='create_date desc')

        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import slides
from . import survey
from . import website_profile

```

## File: data\gamification_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="0">
    <record id="website_slides.badge_data_certification" model="gamification.badge">
        <field name="is_published" eval="True"/>
    </record>
    <record id="website_slides.badge_data_certification_goal" model="gamification.goal.definition">
        <field name="domain">[
            ('survey_scoring_success', '=', True),
            ('slide_id.slide_category', '=', 'certification')
        ]</field>
    </record>
</data></odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mail_template_user_input_certification_failed" model="mail.template">
            <field name="name">Survey: Certification Failure</field>
            <field name="model_id" ref="model_survey_user_input" />
            <field name="subject">You have failed the course: {{ object.slide_partner_id.channel_id.name }}</field>
            <field name="partner_to">{{ object.partner_id.id }}</field>
            <field name="description">Sent to participant if they failed the certification</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px; font-size: 13px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        Dear <t t-out="object.partner_id.name or 'participant' or ''">participant</t><br/><br/>
        Unfortunately, you have failed the certification and are no longer a member of the course: <t t-out="object.slide_partner_id.channel_id.name or ''">Basics of Gardening</t>.<br/><br/>
        Don't hesitate to enroll again!
        <div style="margin: 16px 0px 16px 0px;">
            <a t-att-href="(object.slide_partner_id.channel_id.website_url)"
                style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                Enroll now
            </a>
        </div>
        Thank you for your participation.
    </p>
</div>
            </field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: data\slide_slide_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <!-- CHANNEL 5: Basics of Furniture Creation -->
    <!-- ======================================= -->
    <record id="slide_slide_demo_5_4" model="slide.slide">
        <field name="name">Furniture Creation Certification</field>
        <field name="sequence">7</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_furniture.jpg"/>
        <field name="slide_category">certification</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_5_furn2"/>
        <field name="category_id" ref="website_slides.slide_category_demo_5_0"/>
        <field name="survey_id" ref="website_slides_survey.furniture_certification"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="description">Now that you have completed the course, it's time to test your knowledge!</field>
    </record>

    <!-- CHANNEL 6: DIY Furniture -->
    <!-- ======================================= -->
    <record id="slide_slide_demo_6_0" model="slide.slide">
        <field name="name">DIY Furniture Certification</field>
        <field name="sequence">1</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_furniture.jpg"/>
        <field name="slide_category">certification</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_6_furn3"/>
        <field name="category_id" eval="False"/>
        <field name="survey_id" ref="website_slides_survey.furniture_certification"/>
        <field name="is_published" eval="True"/>
        <field name="date_published" eval="datetime.now() - timedelta(days=8)"/>
        <field name="is_preview" eval="False"/>
        <field name="description">It's time to test your knowledge!</field>
    </record>

</data></odoo>

```

## File: data\survey.user_input.line.csv

```csv
id,user_input_id:id,question_id:id,skipped,answer_type,value_char_box,value_numerical_box,value_date,value_text_box,suggested_answer_id:id,matrix_row_id:id
furniture_certification_answer_1_p1_q1,furniture_certification_answer_1,furniture_certification_page_1_question_1,False,suggestion,,,,,furniture_certification_page_1_question_1_choice_2,
furniture_certification_answer_1_p1_q2_1,furniture_certification_answer_1,furniture_certification_page_1_question_2,False,suggestion,,,,,furniture_certification_page_1_question_2_choice_1,
furniture_certification_answer_1_p1_q2_2,furniture_certification_answer_1,furniture_certification_page_1_question_2,False,suggestion,,,,,furniture_certification_page_1_question_2_choice_3,
furniture_certification_answer_1_p1_q2_3,furniture_certification_answer_1,furniture_certification_page_1_question_2,False,suggestion,,,,,furniture_certification_page_1_question_2_choice_4,
furniture_certification_answer_1_p1_q3,furniture_certification_answer_1,furniture_certification_page_1_question_3,False,text_box,,,,"I really liked the videos, you are awesome!",,
furniture_certification_answer_2_p1_q1,furniture_certification_answer_2,furniture_certification_page_1_question_1,False,suggestion,,,,,furniture_certification_page_1_question_1_choice_3,
furniture_certification_answer_2_p1_q2,furniture_certification_answer_2,furniture_certification_page_1_question_2,False,suggestion,,,,,furniture_certification_page_1_question_2_choice_4,
furniture_certification_answer_2_p1_q3,furniture_certification_answer_2,furniture_certification_page_1_question_3,true,,,,,,,
```

## File: data\survey_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Basics of Furniture Creation Certification  -->
        <record model="survey.survey" id="furniture_certification">
            <field name="title">Furniture Creation Certification</field>
            <field name="access_token">5632a4d7-48cf-4d25-8c52-2174d58cf50b</field>
            <field name="user_id" ref="base.user_admin"/>
            <field name="access_mode">public</field>
            <field name="users_can_go_back" eval="True" />
            <field name="users_login_required" eval="True" />
            <field name="scoring_type" >scoring_with_answers</field>
            <field name="certification" eval="True"></field>
            <field name="certification_mail_template_id" ref="survey.mail_template_certification"></field>
            <field name="is_attempts_limited" eval="True" />
            <field name="attempts_limit">3</field>
            <field name="description">&lt;p&gt;Test your furniture knowledge!&lt;/p&gt;</field>
        </record>
        <!-- Page 1 -->
        <record model="survey.question" id="furniture_certification_page_1">
            <field name="title">Furniture</field>
            <field name="survey_id" ref="furniture_certification" />
            <field name="sequence">1</field>
            <field name="is_page" eval="True"/>
            <field name="question_type" eval="False"/>
            <field name="description">&lt;p&gt;Test your furniture knowledge!&lt;/p&gt;</field>
        </record>
        <!-- Question and predefined answer 1 -->
        <record model="survey.question" id="furniture_certification_page_1_question_1">
            <field name="survey_id" ref="furniture_certification" />
            <field name="sequence">2</field>
            <field name="title">What type of wood is the best for furniture?</field>
            <field name="question_type">simple_choice</field>
            <field name="constr_mandatory" eval="True" />
        </record>
        <record model="survey.question.answer" id="furniture_certification_page_1_question_1_choice_1">
            <field name="question_id" ref="furniture_certification_page_1_question_1"/>
            <field name="sequence">1</field>
            <field name="value">Fir</field>
        </record>
        <record model="survey.question.answer" id="furniture_certification_page_1_question_1_choice_2">
            <field name="question_id" ref="furniture_certification_page_1_question_1"/>
            <field name="sequence">2</field>
            <field name="value">Oak</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">2.0</field>
        </record>
        <record model="survey.question.answer" id="furniture_certification_page_1_question_1_choice_3">
            <field name="question_id" ref="furniture_certification_page_1_question_1"/>
            <field name="sequence">3</field>
            <field name="value">Ash</field>
        </record>
        <record model="survey.question.answer" id="furniture_certification_page_1_question_1_choice_4">
            <field name="question_id" ref="furniture_certification_page_1_question_1"/>
            <field name="sequence">4</field>
            <field name="value">Beech</field>
        </record>
        <!-- Question and predefined answer 2 -->
        <record model="survey.question" id="furniture_certification_page_1_question_2">
            <field name="survey_id" ref="furniture_certification" />
            <field name="sequence">3</field>
            <field name="title">Select all the furniture shown in the video</field>
            <field name="question_type">multiple_choice</field>
        </record>
        <record model="survey.question.answer" id="furniture_certification_page_1_question_2_choice_1">
            <field name="question_id" ref="furniture_certification_page_1_question_2"/>
            <field name="sequence">1</field>
            <field name="value">Chair</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.question.answer" id="furniture_certification_page_1_question_2_choice_2">
            <field name="question_id" ref="furniture_certification_page_1_question_2"/>
            <field name="sequence">2</field>
            <field name="value">Table</field>
            <field name="answer_score">-1.0</field>
        </record>
        <record model="survey.question.answer" id="furniture_certification_page_1_question_2_choice_3">
            <field name="question_id" ref="furniture_certification_page_1_question_2"/>
            <field name="sequence">3</field>
            <field name="value">Desk</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.question.answer" id="furniture_certification_page_1_question_2_choice_4">
            <field name="question_id" ref="furniture_certification_page_1_question_2"/>
            <field name="sequence">4</field>
            <field name="value">Shelf</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.question.answer" id="furniture_certification_page_1_question_2_choice_5">
            <field name="question_id" ref="furniture_certification_page_1_question_2"/>
            <field name="sequence">5</field>
            <field name="value">Bed</field>
            <field name="answer_score">-1.0</field>
        </record>
        <!-- Question and predefined answer 5 -->
        <record model="survey.question" id="furniture_certification_page_1_question_3">
            <field name="survey_id" ref="furniture_certification" />
            <field name="sequence">4</field>
            <field name="title">What do you think about the content of the course? (not rated)</field>
            <field name="question_type">text_box</field>
        </record>

        <record model="survey.user_input" id="furniture_certification_answer_1">
            <field name="survey_id" ref="website_slides_survey.furniture_certification" />
            <field name="partner_id" ref="base.res_partner_address_3"/>
            <field name="email">douglas.fletcher51@example.com</field>
            <field name="state">done</field>
        </record>
        <record model="survey.user_input" id="furniture_certification_answer_2">
            <field name="survey_id" ref="website_slides_survey.furniture_certification" />
            <field name="partner_id" ref="base.res_partner_address_7"/>
            <field name="email">billy.fox45@example.com</field>
            <field name="state">done</field>
        </record>
    </data>
</odoo>

```

## File: models\slide_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.osv import expression


class Channel(models.Model):
    _inherit = 'slide.channel'

    nbr_certification = fields.Integer("Number of Certifications", compute='_compute_slides_statistics', store=True)

    def _remove_membership(self, partner_ids):
        """Remove the relationship between the user_input and the slide_partner_id.

        Removing the relationship between the user_input from the slide_partner_id allows to keep
        track of the current pool of attempts allowed since the user (last) joined
        the course, as only those will have a slide_partner_id."""
        removed_channel_partner_domain = []
        for channel in self:
            removed_channel_partner_domain = expression.OR([
                removed_channel_partner_domain,
                [('partner_id', 'in', partner_ids), ('channel_id', '=', channel.id)]
            ])
        if removed_channel_partner_domain:
            slide_partners_sudo = self.env['slide.slide.partner'].sudo().search(
                removed_channel_partner_domain)
            slide_partners_sudo.user_input_ids.slide_partner_id = False
        return super()._remove_membership(partner_ids)

```

## File: models\slide_slide.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SlidePartnerRelation(models.Model):
    _inherit = 'slide.slide.partner'

    user_input_ids = fields.One2many('survey.user_input', 'slide_partner_id', 'Certification attempts')
    survey_scoring_success = fields.Boolean('Certification Succeeded', compute='_compute_survey_scoring_success', store=True)

    @api.depends('partner_id', 'user_input_ids.scoring_success')
    def _compute_survey_scoring_success(self):
        succeeded_user_inputs = self.env['survey.user_input'].sudo().search([
            ('slide_partner_id', 'in', self.ids),
            ('scoring_success', '=', True)
        ])
        succeeded_slide_partners = succeeded_user_inputs.mapped('slide_partner_id')
        for record in self:
            record.survey_scoring_success = record in succeeded_slide_partners

    def _compute_field_value(self, field):
        super()._compute_field_value(field)
        if field.name == 'survey_scoring_success':
            self.filtered('survey_scoring_success').write({
                'completed': True
            })

class Slide(models.Model):
    _inherit = 'slide.slide'

    name = fields.Char(compute='_compute_name', readonly=False, store=True)
    slide_category = fields.Selection(selection_add=[
        ('certification', 'Certification')
    ], ondelete={'certification': 'set default'})
    slide_type = fields.Selection(selection_add=[
        ('certification', 'Certification')
    ], ondelete={'certification': 'set null'})
    survey_id = fields.Many2one('survey.survey', 'Certification')
    nbr_certification = fields.Integer("Number of Certifications", compute='_compute_slides_statistics', store=True)
    # small override of 'is_preview' to uncheck it automatically for slides of type 'certification'
    is_preview = fields.Boolean(compute='_compute_is_preview', readonly=False, store=True)

    _sql_constraints = [
        ('check_survey_id', "CHECK(slide_category != 'certification' OR survey_id IS NOT NULL)", "A slide of type 'certification' requires a certification."),
        ('check_certification_preview', "CHECK(slide_category != 'certification' OR is_preview = False)", "A slide of type certification cannot be previewed."),
    ]

    @api.depends('survey_id')
    def _compute_name(self):
        for slide in self:
            if not slide.name and slide.survey_id:
                slide.name = slide.survey_id.title

    def _compute_mark_complete_actions(self):
        slides_certification = self.filtered(lambda slide: slide.slide_category == 'certification')
        slides_certification.can_self_mark_uncompleted = False
        slides_certification.can_self_mark_completed = False
        super(Slide, self - slides_certification)._compute_mark_complete_actions()

    @api.depends('slide_category')
    def _compute_is_preview(self):
        for slide in self:
            if slide.slide_category == 'certification' or not slide.is_preview:
                slide.is_preview = False

    @api.depends('slide_type')
    def _compute_slide_icon_class(self):
        certification = self.filtered(lambda slide: slide.slide_type == 'certification')
        certification.slide_icon_class = 'fa-trophy'
        super(Slide, self - certification)._compute_slide_icon_class()

    @api.depends('slide_category', 'source_type')
    def _compute_slide_type(self):
        super(Slide, self)._compute_slide_type()
        for slide in self:
            if slide.slide_category == 'certification':
                slide.slide_type = 'certification'

    @api.model_create_multi
    def create(self, vals_list):
        slides = super().create(vals_list)
        slides_with_survey = slides.filtered('survey_id')
        slides_with_survey.slide_category = 'certification'
        slides_with_survey._ensure_challenge_category()
        return slides

    def write(self, values):
        old_surveys = self.mapped('survey_id')
        result = super(Slide, self).write(values)
        if 'survey_id' in values:
            self._ensure_challenge_category(old_surveys=old_surveys - self.mapped('survey_id'))
        return result

    def unlink(self):
        old_surveys = self.mapped('survey_id')
        result = super(Slide, self).unlink()
        self._ensure_challenge_category(old_surveys=old_surveys, unlink=True)
        return result

    def _ensure_challenge_category(self, old_surveys=None, unlink=False):
        """ If a slide is linked to a survey that gives a badge, the challenge category of this badge must be
        set to 'slides' in order to appear under the certification badge list on ranks_badges page.
        If the survey is unlinked from the slide, the challenge category must be reset to 'certification'"""
        if old_surveys:
            old_certification_challenges = old_surveys.mapped('certification_badge_id').challenge_ids
            old_certification_challenges.write({'challenge_category': 'certification'})
        if not unlink:
            certification_challenges = self.survey_id.certification_badge_id.challenge_ids
            certification_challenges.write({'challenge_category': 'slides'})

    def _generate_certification_url(self):
        """ get a map of certification url for certification slide from `self`. The url will come from the survey user input:
                1/ existing and not done user_input for member of the course
                2/ create a new user_input for member
                3/ for no member, a test user_input is created and the url is returned
            Note: the slide.slides.partner should already exist

            We have to generate a new invite_token to differentiate pools of attempts since the
            course can be enrolled multiple times.
        """
        certification_urls = {}
        for slide in self.filtered(lambda slide: slide.slide_category == 'certification' and slide.survey_id):
            if slide.channel_id.is_member:
                user_membership_id_sudo = slide.user_membership_id.sudo()
                if user_membership_id_sudo.user_input_ids:
                    last_user_input = next(user_input for user_input in user_membership_id_sudo.user_input_ids.sorted(
                        lambda user_input: user_input.create_date, reverse=True
                    ))
                    certification_urls[slide.id] = last_user_input.get_start_url()
                else:
                    user_input = slide.survey_id.sudo()._create_answer(
                        partner=self.env.user.partner_id,
                        check_attempts=False,
                        **{
                            'slide_id': slide.id,
                            'slide_partner_id': user_membership_id_sudo.id
                        },
                        invite_token=self.env['survey.user_input']._generate_invite_token()
                    )
                    certification_urls[slide.id] = user_input.get_start_url()
            else:
                user_input = slide.survey_id.sudo()._create_answer(
                    partner=self.env.user.partner_id,
                    check_attempts=False,
                    test_entry=True, **{
                        'slide_id': slide.id
                    }
                )
                certification_urls[slide.id] = user_input.get_start_url()
        return certification_urls

```

## File: models\survey_survey.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError



class Survey(models.Model):
    _inherit = 'survey.survey'

    slide_ids = fields.One2many(
        'slide.slide', 'survey_id', string="Certification Slides",
        help="The slides this survey is linked to through the e-learning application")
    slide_channel_ids = fields.One2many(
        'slide.channel', string="Certification Courses", compute='_compute_slide_channel_data',
        help="The courses this survey is linked to through the e-learning application",
        groups='website_slides.group_website_slides_officer')
    slide_channel_count = fields.Integer("Courses Count", compute='_compute_slide_channel_data', groups='website_slides.group_website_slides_officer')

    @api.depends('slide_ids.channel_id')
    def _compute_slide_channel_data(self):
        for survey in self:
            survey.slide_channel_ids = survey.slide_ids.mapped('channel_id')
            survey.slide_channel_count = len(survey.slide_channel_ids)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_linked_to_course(self):
        # we consider it's ok to show certification names for people trying to delete courses
        # even if they don't have access to those surveys hence the sudo usage
        certifications = self.sudo().slide_ids.filtered(lambda slide: slide.slide_type == "certification").mapped('survey_id').exists()
        if certifications:
            certifications_course_mapping = [_('- %s (Courses - %s)', certi.title, '; '.join(certi.slide_channel_ids.mapped('name'))) for certi in certifications]
            raise ValidationError(_(
                'Any Survey listed below is currently used as a Course Certification and cannot be deleted:\n%s',
                '\n'.join(certifications_course_mapping)))

    # ---------------------------------------------------------
    # Actions
    # ---------------------------------------------------------

    def action_survey_view_slide_channels(self):
        """ Redirect to the channels using the survey as a certification. Open
        in no-create as link between those two comes through a slide, hard to
        keep as default values. """
        action = self.env["ir.actions.actions"]._for_xml_id("website_slides.slide_channel_action_overview")
        action['display_name'] = _("Courses")
        if self.slide_channel_count == 1:
            action.update({'views': [(False, 'form')],
                           'res_id': self.slide_channel_ids[0].id})
        else:
            action.update({'views': [[False, 'tree'], [False, 'form']],
                           'domain': [('id', 'in', self.slide_channel_ids.ids)]})
        action['context'] = dict(
            ast.literal_eval(action.get('context') or '{}'),  # sufficient in most cases
            create=False
        )
        return action

    # ---------------------------------------------------------
    # Business
    # ---------------------------------------------------------

    def _check_answer_creation(self, user, partner, email, test_entry=False, check_attempts=True, invite_token=False):
        """ Overridden to allow website_slides_officer to test certifications. """
        self.ensure_one()
        if test_entry and user.has_group('website_slides.group_website_slides_officer'):
            return True

        return super(Survey, self)._check_answer_creation(user, partner, email, test_entry=test_entry, check_attempts=check_attempts, invite_token=invite_token)

    def _prepare_challenge_category(self):
        slide_survey = self.env['slide.slide'].search([('survey_id', '=', self.id)])
        return 'slides' if slide_survey else 'certification'

```

## File: models\survey_user.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api
from odoo.osv import expression


class SurveyUserInput(models.Model):
    _inherit = 'survey.user_input'

    slide_id = fields.Many2one('slide.slide', 'Related course slide',
        help="The related course slide when there is no membership information")
    slide_partner_id = fields.Many2one('slide.slide.partner', 'Subscriber information',
        help="Slide membership information for the logged in user",
        index='btree_not_null') # index useful for deletions in comodel

    @api.model_create_multi
    def create(self, vals_list):
        records = super(SurveyUserInput, self).create(vals_list)
        records._check_for_failed_attempt()
        return records

    def write(self, vals):
        res = super(SurveyUserInput, self).write(vals)
        if 'state' in vals:
            self._check_for_failed_attempt()
        return res

    def _check_for_failed_attempt(self):
        """ If the user fails their last attempt at a course certification,
        we remove them from the members of the course (and they have to enroll again).
        They receive an email in the process notifying them of their failure and suggesting
        they enroll to the course again.

        The purpose is to have a 'certification flow' where the user can re-purchase the
        certification when they have failed it."""

        if self:
            user_inputs = self.search([
                ('id', 'in', self.ids),
                ('state', '=', 'done'),
                ('scoring_success', '=', False),
                ('slide_partner_id', '!=', False)
            ])

            if user_inputs:
                for user_input in user_inputs:
                    removed_memberships_per_partner = {}
                    if user_input.survey_id._has_attempts_left(user_input.partner_id, user_input.email, user_input.invite_token):
                        # skip if user still has attempts left
                        continue

                    self.env.ref('website_slides_survey.mail_template_user_input_certification_failed').send_mail(
                        user_input.id, email_layout_xmlid="mail.mail_notification_light"
                    )

                    removed_memberships = removed_memberships_per_partner.get(
                        user_input.partner_id,
                        self.env['slide.channel']
                    )
                    removed_memberships |= user_input.slide_partner_id.channel_id
                    removed_memberships_per_partner[user_input.partner_id] = removed_memberships

                for partner_id, removed_memberships in removed_memberships_per_partner.items():
                    removed_memberships._remove_membership(partner_id.ids)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import slide_slide
from . import slide_channel
from . import survey_user
from . import survey_survey

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_survey_slides_officer,access.survey.slides.officer,survey.model_survey_survey,website_slides.group_website_slides_officer,1,0,0,0
access_survey_question_slides_officer,access.survey.question.slides.officer,survey.model_survey_question,website_slides.group_website_slides_officer,1,0,0,0
access_survey_question_answer_slides_officer,survey.question.answer.slides.officer,survey.model_survey_question_answer,website_slides.group_website_slides_officer,1,0,0,0
access_survey_user_input_slides_officer,access.survey.user.input.slides.officer,survey.model_survey_user_input,website_slides.group_website_slides_officer,1,0,0,0
access_survey_user_input_line_slides_officer,access.survey.user.input.line.slides.officer,survey.model_survey_user_input_line,website_slides.group_website_slides_officer,1,0,0,0

```

## File: static\src\img\certification.svg

```svg
<svg width="12" height="12" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"><defs><path d="M7.158 11.549a2.005 2.005 0 0 1-3.105-1.286 2.005 2.005 0 0 1-1.286-3.105 2.005 2.005 0 0 1 1.286-3.105 2.005 2.005 0 0 1 3.105-1.286 2.005 2.005 0 0 1 3.105 1.286 2.005 2.005 0 0 1 1.286 3.105 2.005 2.005 0 0 1-1.286 3.105 2.005 2.005 0 0 1-3.105 1.286z" id="a"/><filter filterUnits="objectBoundingBox" id="b"><feMorphology radius=".5" in="SourceAlpha" result="shadowSpreadInner1"/><feOffset in="shadowSpreadInner1" result="shadowOffsetInner1"/><feComposite in="shadowOffsetInner1" in2="SourceAlpha" operator="arithmetic" k2="-1" k3="1" result="shadowInnerInner1"/><feColorMatrix values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0.0749562937 0" in="shadowInnerInner1"/></filter><path id="d" d="M6.04788912 6.85018391L7.55257375 8.20939291 9.91700506 6 11 6.947144 7.55257375 10 5 7.75171917z"/><filter x="-2.5%" y="-3.8%" width="105%" height="115%" filterUnits="objectBoundingBox" id="c"><feOffset dy=".3" in="SourceAlpha" result="shadowOffsetOuter1"/><feColorMatrix values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0.233200393 0" in="shadowOffsetOuter1"/></filter></defs><g transform="translate(-2 -2)" fill="none" fill-rule="evenodd"><g transform="rotate(22 5.57 9.903)"><use fill="#00C9FF" xlink:href="#a"/><use fill="#000" filter="url(#b)" xlink:href="#a"/><path stroke="#FFF" stroke-width=".7" d="M7.158 11.96a2.355 2.355 0 0 1-3.395-1.407 2.355 2.355 0 0 1-1.406-3.395 2.355 2.355 0 0 1 1.406-3.395 2.355 2.355 0 0 1 3.395-1.406 2.355 2.355 0 0 1 3.395 1.406 2.355 2.355 0 0 1 1.406 3.395 2.355 2.355 0 0 1-1.406 3.395 2.355 2.355 0 0 1-3.395 1.406z"/></g><use fill="#000" filter="url(#c)" xlink:href="#d"/><use fill="#FFF" xlink:href="#d"/></g></svg>
```

## File: static\src\js\slides_course_fullscreen_player.js

```javascript
/** @odoo-module **/

import { renderToElement } from "@web/core/utils/render";
import Fullscreen from "@website_slides/js/slides_course_fullscreen_player";

Fullscreen.include({
    /**
     * Extend the _renderSlide method so that slides of category "certification"
     * are also taken into account and rendered correctly
     *
     * @private
     * @override
     */
    _renderSlide: function (){
        var def = this._super.apply(this, arguments);
        var $content = this.$('.o_wslides_fs_content');
        if (this.get('slide').category === "certification"){
            $content.empty().append(renderToElement('website.slides.fullscreen.certification',{widget: this}));
        }
        return Promise.all([def]);
    },
});

```

## File: static\src\js\slides_upload.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import SlidesUpload from "@website_slides/js/slides_upload";

/**
 * Management of the new 'certification' slide_category
 */
SlidesUpload.SlideUploadDialog.include({
    events: Object.assign({}, SlidesUpload.SlideUploadDialog.prototype.events || {}, {
        'change input#certification_id': '_onChangeCertification'
    }),

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

   /**
    * Will automatically set the title of the slide to the title of the chosen certification
    */
    _onChangeCertification: function (ev) {
        const $inputElement = this.$("input#name");
        if (ev.added) {
            this.$('.o_error_no_certification').addClass('d-none');
            this.$('#certification_id').parent().find('.select2-container').removeClass('is-invalid');
            if (ev.added.text && !$inputElement.val().trim()) {
                $inputElement.val(ev.added.text);
            }
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Overridden to add the "certification" slide category
     *
     * @override
     * @private
     */
    _setup: function () {
        this._super.apply(this, arguments);
        this.slide_category_data['certification'] = {
            icon: 'fa-trophy',
            label: _t('Certification'),
            template: 'website.slide.upload.modal.certification',
        };
    },
    /**
     * Overridden to add certifications management in select2
     *
     * @override
     * @private
     */
    _bindSelect2Dropdown: function () {
        this._super.apply(this, arguments);

        var self = this;
        this.$('#certification_id').select2(this._select2Wrapper(_t('Certification'), false,
            function () {
                return self.rpc('/slides_survey/certification/search_read', {
                    fields: ['title'],
                });
            }, 'title')
        );
    },
    /**
     * The select2 field makes the "required" input hidden on the interface.
     * We need to make the "certification" field required so we override this method
     * to handle validation in a fully custom way.
     *
     * @override
     * @private
     */
    _formValidate: function () {
        var result = this._super.apply(this, arguments);

        var $certificationInput = this.$('#certification_id');
        if ($certificationInput.length !== 0) {
            var $select2Container = $certificationInput
                .parent()
                .find('.select2-container');
            var $errorContainer = $('.o_error_no_certification');
            $select2Container.removeClass('is-invalid is-valid');
            if ($certificationInput.is(':invalid')) {
                $select2Container.addClass('is-invalid');
                $errorContainer.removeClass('d-none');
            } else if ($certificationInput.is(':valid')) {
                $select2Container.addClass('is-valid');
                $errorContainer.addClass('d-none');
            }
        }

        return result;
    },
    /**
     * Overridden to add the 'certification' field into the submitted values
     *
     * @override
     * @private
     */
    _getSelect2DropdownValues: function () {
        var result = this._super.apply(this, arguments);

        var certificateValue = this.$('#certification_id').select2('data');
        var survey = {};
        if (certificateValue) {
            if (certificateValue.create) {
                survey.id = false;
                survey.title = certificateValue.text;
            } else {
                survey.id = certificateValue.id;
            }
        }
        result['survey'] = survey;
        return result;
    },
});

```

## File: static\src\js\survey_form.js

```javascript
/** @odoo-module **/

import { ShareMail } from '@website_slides/js/slides_share';
import SurveyFormWidget from '@survey/js/survey_form';

SurveyFormWidget.include({
    _onNextScreenDone(options) {
        this._super(...arguments);

        new ShareMail(this).attachTo($('.oe_slide_js_share_email'));
    }
});

```

## File: static\src\xml\website_slides_fullscreen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>

    <t t-name="website.slides.fullscreen.certification">
        <div class="justify-content-center align-self-center text-center">
            <div t-if="widget.get('slide').category === 'certification' &amp;&amp; !widget.get('slide').completed" class="">
                <a class="btn btn-primary" t-att-href="'/slides_survey/slide/get_certification_url?slide_id=' + widget.get('slide').id" target="_blank">
                    <i class="fa fa-graduation-cap"/>
                    <span t-if="widget.get('slide').isMember"> Begin Certification</span>
                    <span t-else="">Test Certification</span>
                </a>
            </div>
            <div t-if="widget.get('slide').category === 'certification' &amp;&amp; widget.get('slide').completed">
                <h4 class="mb-3 text-white">Congratulations, you passed the Certification!</h4>
                <a role="button" class="btn btn-primary" t-att-href="'/survey/' + widget.get('slide').certificationId + '/get_certification'">
                    <i class="fa fa-fw fa-trophy" role="img" aria-label="Download certification" title="Download certification"/> Download certification
                </a>
            </div>
            <div t-if="widget.get('slide').category === 'certification' &amp;&amp; widget.get('slide').canUpload" class="mt-3 h6">
                <a t-att-href="'/web#id=' + widget.get('slide').certificationId + '&amp;model=survey.survey&amp;view_type=form'">
                    <i class="oi oi-arrow-right me-1"/>Add Questions to this Survey
                </a>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\xml\website_slide_upload.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<template>
    <t t-name="website.slide.upload.modal.certification">
        <div>
            <form class="clearfix">
                <div class="row">
                    <div id="o_wslides_js_slide_upload_left_column" class="col-md-6">
                        <div class="mb-3">
                            <label for="certification_id" class="col-form-label">Certification</label>
                            <input class="form-control" id="certification_id" required="required"/>
                            <span class="o_error_no_certification text-danger d-none">Please select a certification.</span>
                        </div>
                        <t t-call="website.slide.upload.modal.common"/>
                        <canvas id="data_canvas" class="d-none"></canvas>
                    </div>
 
                    <div id="o_wslides_js_slide_upload_preview_column" class="col-md-6">
                        <div class="img-thumbnail h-100">
                            <div class="o_slide_tutorial p-3">
                                <div class="h5">How to upload a certification on your course?</div>
                                <div class="my-4">You can create your certification from here or use an existing one. Once your certification is created, you can still edit it in backend.</div>
                            </div>
                        </div>
                    </div>        
                </div>  
            </form>
        </div>
    </t>
</template>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.slides.survey</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website_slides.res_config_settings_view_form"/>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <xpath expr="//setting[@id='website_slide_install_website_slides_survey']" position="inside">
                <div class="mt8">
                    <button type="action" name="%(website_slides_survey.survey_survey_action_slides)d" string="Manage Certifications" class="btn-link" icon="oi-arrow-right"/>
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
    <record id="slide_channel_view_form" model="ir.ui.view">
        <field name="name">slide.channel.view.form.inherit.survey</field>
        <field name="model">slide.channel</field>
        <field name="inherit_id" ref="website_slides.view_slide_channel_form"/>
        <field name="arch" type="xml">
            <xpath expr="//span[@name='members_completed_count_label']" position="replace">
                <field  name="nbr_certification" invisible="1"/>
                <span class="o_stat_text" invisible="nbr_certification &gt; 0">Finished</span>
                <span class="o_stat_text" invisible="nbr_certification == 0">Certified</span>
            </xpath>
            <xpath expr="//field[@name='slide_category']" position="after">
                <field name="survey_id"/>
            </xpath>
            <xpath expr="//create[@name='add_slide_lesson']" position="after">
                <create name="add_slide_certificate" string="Add Certification" groups="survey.group_survey_user" context="{'default_slide_category': 'certification'}"/>
            </xpath>
        </field>
    </record>

    <record id="slide_channel_view_kanban" model="ir.ui.view">
        <field name="name">slide.channel.view.kanban.inherit.survey</field>
        <field name="model">slide.channel</field>
        <field name="inherit_id" ref="website_slides.slide_channel_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='website_published']" position="after">
                <field name="nbr_certification"/>
            </xpath>
            <xpath expr="//span[@name='done_members_count_label']" position="replace">
                <t t-if="record.nbr_certification.raw_value"><span class="text-muted">Certified</span></t>
                <t t-else=""><span class="text-muted">Finished</span></t>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\slide_slide_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="slide_slide_partner_view_search" model="ir.ui.view">
        <field name="name">slide.slide.partner.view.search.inherit.survey</field>
        <field name="model">slide.slide.partner</field>
        <field name="inherit_id" ref="website_slides.slide_slide_partner_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='filter_completed']" position="after">
                <filter string="Certification Passed" name="filter_survey_scoring_success" domain="[('survey_scoring_success', '=', True)]"/>
            </xpath>
        </field>
    </record>

    <record id="slide_slide_partner_view_tree" model="ir.ui.view">
        <field name="name">slide.slide.partner.view.tree.inherit.survey</field>
        <field name="model">slide.slide.partner</field>
        <field name="inherit_id" ref="website_slides.slide_slide_partner_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='completed']" position="after">
                <field name="survey_scoring_success" string="Certification Passed"/>
            </xpath>
        </field>
    </record>

    <record id="slide_slide_partner_view_form" model="ir.ui.view">
        <field name="name">slide.slide.partner.view.form.inherit.survey</field>
        <field name="model">slide.slide.partner</field>
        <field name="inherit_id" ref="website_slides.slide_slide_partner_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='completed']" position="after">
                <field name="survey_scoring_success"
                    readonly="1"
                    invisible="slide_category != 'certification'"/>
            </xpath>
            <xpath expr="//group[@name='main_content']" position="after">
                <field name="user_input_ids" mode="tree"
                    readonly="1"
                    invisible="slide_category != 'certification'"/>
            </xpath>
        </field>
    </record>

</data></odoo>

```

## File: views\slide_slide_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="slide_slide_view_form" model="ir.ui.view">
        <field name="name">slide.slide.view.form.inherit.survey</field>
        <field name="model">slide.slide</field>
        <field name="inherit_id" ref="website_slides.view_slide_slide_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='slide_category']" position="after">
                <field name="survey_id"
                    invisible="slide_category != 'certification'"
                    required="slide_category == 'certification'"
                    domain="[('certification', '=', True)]" context="{'default_certification': True, 'default_scoring_type': 'scoring_without_answers'}"/>
            </xpath>
            <xpath expr="//field[@name='is_preview']" position="attributes">
                <attribute name="invisible">slide_category == 'certification'</attribute>
            </xpath>
        </field>
    </record>

    <record id="slide_slide_action_certification" model="ir.actions.act_window">
        <field name="name">Certifications</field>
        <field name="res_model">slide.slide</field>
        <field name="view_mode">tree,form,graph</field>
        <field name="domain">[('slide_category', '=', 'certification')]</field>
        <field name="context">{'default_slide_category': 'certification'}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new certification
            </p>
        </field>
        <field name="view_id" ref="website_slides.view_slide_slide_tree"/>
    </record>
</odoo>

```

## File: views\survey_survey_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="survey_survey_view_tree_slides" model="ir.ui.view">
        <field name="name">survey.survey.view.tree.slides</field>
        <field name="model">survey.survey</field>
        <field name="inherit_id" ref="survey.survey_survey_view_tree"/>
        <field name="mode">primary</field>
        <field name="priority" eval="20"/>
        <field name="arch" type="xml">
            <field name="title" position="attributes">
                <attribute name="string">Certification Title</attribute>
            </field>
            <field name="answer_count" position="replace"/>
            <field name="success_ratio" position="attributes">
                <attribute name="string">Success Ratio (%)</attribute>
            </field>
            <field name="answer_score_avg" position="attributes">
                <attribute name="string">Avg Score (%)</attribute>
            </field>
            <button name="certification" position="replace"/>
        </field>
    </record>

    <record id="survey_survey_view_search_slides" model="ir.ui.view">
        <field name="name">survey.survey.view.search.slides</field>
        <field name="model">survey.survey</field>
        <field name="inherit_id" ref="survey.survey_survey_view_search"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr='//filter[@name="certification"]' position="replace"/>
        </field>
    </record>

    <record id="survey_survey_view_form" model="ir.ui.view">
        <field name="name">survey.survey.view.form.inherit.website.slides</field>
        <field name="model">survey.survey</field>
        <field name="inherit_id" ref="survey.survey_survey_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_survey_user_input_certified']" position="before">
                <button name="action_survey_view_slide_channels"
                    type="object"
                    class="oe_stat_button"
                    invisible="slide_channel_count == 0"
                    icon="fa-graduation-cap"
                    groups="website_slides.group_website_slides_officer">
                    <field string="Courses" name="slide_channel_count" widget="statinfo"/>
                </button>
            </xpath>
        </field>
    </record>

    <record id="survey_survey_view_kanban" model="ir.ui.view">
        <field name="name">survey.survey.view.kanban.inherit.website.slides</field>
        <field name="model">survey.survey</field>
        <field name="inherit_id" ref="survey.survey_survey_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='session_state']" position="after">
                <field name="slide_channel_count"/>
            </xpath>
            <xpath expr="//div[@name='o_survey_kanban_card_section_success']" position="after">
                <div class="col-lg-1 col-sm-4 col-6 py-0 my-2" groups="website_slides.group_website_slides_officer">
                    <a t-if="record.slide_channel_count.raw_value"
                       type="object"
                       name="action_survey_view_slide_channels"
                       class="fw-bold">
                        <field name="slide_channel_count"/><br />
                        <span class="text-muted">Courses</span>
                    </a>
                </div>
            </xpath>
        </field>
    </record>

    <record id="survey_survey_action_slides" model="ir.actions.act_window">
        <field name="name">Certifications</field>
        <field name="res_model">survey.survey</field>
        <field name="view_mode">kanban,tree,pivot,graph,form</field>
        <field name="domain">[('certification', '=', True)]</field>
        <field name="search_view_id" ref="survey_survey_view_search_slides"/>
        <field name="context">{'default_certification': True, 'default_scoring_type': 'scoring_with_answers'}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Certification
            </p>
            <p>
                Assess the level of understanding of your attendees
                <br/>and send them a document if they pass the test.
            </p>
        </field>
    </record>
    <record id="survey_survey_action_slides_view_kanban" model="ir.actions.act_window.view">
         <field name="sequence" eval="0"/>
         <field name="view_mode">kanban</field>
         <field name="view_id" ref="survey.survey_survey_view_kanban"/>
         <field name="act_window_id" ref="survey_survey_action_slides"/>
     </record>
     <record id="survey_survey_action_slides_view_tree" model="ir.actions.act_window.view">
         <field name="sequence" eval="1"/>
         <field name="view_mode">tree</field>
         <field name="view_id" ref="survey_survey_view_tree_slides"/>
         <field name="act_window_id" ref="survey_survey_action_slides"/>
     </record>
     <record id="survey_survey_action_slides_view_form" model="ir.actions.act_window.view">
         <field name="sequence" eval="4"/>
         <field name="view_mode">form</field>
         <field name="view_id" ref="survey.survey_survey_view_form"/>
         <field name="act_window_id" ref="survey_survey_action_slides"/>
     </record>
</odoo>

```

## File: views\survey_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="survey_fill_form_done_inherit_website_slides" inherit_id="survey.survey_fill_form_done">
            <xpath expr="//div[hasclass('o_survey_finished')]//a[hasclass('btn-primary')][1]" position="after">
                <t t-if="channel_id">
                    <a role="button" class="me-2 mt-2 mt-md-0 btn btn-secondary btn-lg" href="#" data-bs-toggle="modal" t-attf-data-bs-target="#slideShareModal_{{channel_id.id}}">
                        <i class="fa fa-share-alt" aria-label="Share certification" title="Share certification"/>
                        Share your certification
                    </a>
                    <t t-call="website_slides.slide_share_modal">
                        <t t-set="record" t-value="channel_id"/>
                        <t t-set="email_sharing" t-value="channel_id.share_channel_template_id"/>
                    </t>
                </t>
            </xpath>
            <xpath expr="//div[hasclass('o_survey_finished')]/div[last()]" position="after">
                <div t-if="channel_id" class="mt16">
                    <a role="button"
                        class="btn btn-primary btn-lg"
                        t-att-href="channel_id.website_url">
                        Go back to course
                    </a>
                </div>
            </xpath>
        </template>

        <template id="o_wss_certification_icon">
            <t t-set="icon_url" t-value="icon_url if icon_url else '/website_slides_survey/static/src/img/certification.svg'"/>
            <t t-set="icon_classes" t-value="icon_classes if icon_classes else 'o_wss_certification_icon'"/>
            <img t-att-class="icon_classes" t-att-src="icon_url" alt="Certification icon"/>
        </template>
    </data>
</odoo>

```

## File: views\website_profile.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <template id="user_profile_content" inherit_id="website_profile.user_profile_content">
        <xpath expr="//div[@id='profile_about_badge']" position="before">
            <t t-if="channel">
                <div class="mb32">
                    <h5 class="border-bottom pb-1">Certifications</h5>
                    <t t-call="website_slides_survey.display_certificate"/>
                </div>
            </t>
        </xpath>
        <!-- Certification Attempts -->
        <xpath expr="//ul[hasclass('o_wprofile_nav_tabs')]" position="inside">
            <li t-if="show_certification_tab" class="nav-item">
                <a role="tab" aria-controls="certification" href="#profile_tab_content_certification" t-attf-class="nav-link #{ 'active' if active_tab == 'certification' else '' }" data-bs-toggle="tab">
                    <t t-if="user_inputs" t-out="len(user_inputs)" /> Certification Attempts
                </a>
            </li>
        </xpath>
        <xpath expr="//div[hasclass('o_wprofile_tabs_content')]" position="inside">
            <div t-if="show_certification_tab" role="tabpanel" t-attf-class="tab-pane #{ 'show active' if active_tab == 'certification' else '' }" id="profile_tab_content_certification">
                <h5 class="d-flex justify-content-between align-items-end border-bottom pb-1">
                    Certification Attempts
                    <form method="get">
                        <div class="input-group" role="search">
                            <input type="text" class="form-control" name="certification_search" t-att-value="certification_search_terms or ''" placeholder="Search Attempts..."/>
                            <div class="input-group-append">
                                <button type="submit" class="btn btn-primary" aria-label="Search" title="Search">
                                    <i class="fa fa-search"/>
                                </button>
                            </div>
                        </div>
                    </form>
                </h5>
                <t t-if="not user_inputs">
                    <p t-if="certification_search_terms">No certification found for the given search term.</p>
                    <p t-else="">You have not taken any certification yet.</p>
                    <a href="/slides/all?slide_category=certification">
                        <i class="oi oi-arrow-right"/> See Certifications
                    </a>
                </t>
                <div t-else="" class="card p-3 mb-3" t-foreach="user_inputs" t-as="user_input">
                    <div class="row">
                        <div class="col-sm-3" t-out="user_input.create_date" t-options="{'widget': 'datetime'}"/>
                        <div class="col-sm-4">
                            <a t-if="user_input.slide_id"
                                t-attf-href="/slides_survey/slide/get_certification_url?slide_id=#{user_input.slide_id.id}"
                                t-out="user_input.survey_id.title"/>
                            <t t-else="" t-out="user_input.survey_id.title"/>
                        </div>
                        <div class="col-sm-3">
                            Attempt n°<t t-out="user_input.attempts_number"/>
                        </div>
                        <div class="col-sm-2">
                            <span t-attf-class="badge #{ 'text-bg-primary' if user_input.scoring_success else 'text-bg-danger'}">
                                <t t-out="user_input.scoring_percentage"/>%
                            </span>
                        </div>
                    </div>
                </div>
            </div>
        </xpath>
    </template>

     <template id="display_certificate">
        <t t-if="certificates">
            <div class="row">
               <div class="col-12 col-lg-6" t-foreach="certificates" t-as="certificate">
                    <div class="card mb-2">
                        <div class="card-body o_wprofile_slides_course_card_body p-0 d-flex" t-attf-onclick="location.href='/slides/#{slug(certificate.slide_id.channel_id)}';">
                            <div class="ps-5 pe-4 rounded-start" t-attf-style="background-image: url(#{website.image_url(certificate.slide_id, 'image_128')}); background-position: center"/>
                            <div class="p-2 w-100">
                                <h5 class="mt-0 mb-1" t-esc="certificate.survey_id.title"/>
                                <div t-if="user.id == uid">
                                    <small class="fw-bold">Score: <span t-esc="certificate.scoring_percentage"/> %</small>
                                    <div class="float-end">
                                        <a role="button" class="float-end" t-att-href="'/survey/%s/get_certification' % certificate.survey_id.id">
                                            <i class="fa fa-download" aria-label="Download certification" title="Download Certification"/>
                                        </a>
                                        <a role="button" class="me-2" href="#"
                                             t-attf-onclick="event.stopPropagation(); $('#slideShareModal_#{certificate.slide_id.channel_id.id}').modal('show');">
                                            <i class="fa fa-share-alt" aria-label="Share" title="Share"/>
                                        </a>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <t t-call="website_slides.slide_share_modal">
                        <t t-set="record" t-value="certificate.slide_id.channel_id"/>
                        <t t-set="email_sharing" t-value="certificate.slide_id.channel_id.share_channel_template_id"/>
                    </t>
                </div>
            </div>
        </t>
         <t t-elif="my_profile">
            <div class="text-muted d-inline-block">
                Certifications are exams that you successfully passed. <br />
                <a href="/slides/all?slide_type=certification" class="btn-link">
                    <i class="fa fa-arrow-right"></i> Get Certified
                </a>
            </div>
        </t>
        <t t-else="">
            <div class="text-muted d-inline-block">No certifications yet!</div>
            <div class="text-end d-inline-block pull-right">
                <a href="/slides/all?slide_category=certification" class="btn btn-link btn-sm"><i class="oi oi-arrow-right me-1"/>All Certifications</a>
            </div>
        </t>
    </template>

    <template id="top3_user_card" inherit_id="website_profile.top3_user_card">
        <xpath expr="//div[hasclass('o_wprofile_top3_card_footer')]//div[last()]" position="after">
            <div class="col py-3"><b t-esc="user['certification_count']"/> <span class="text-muted">Certifications</span></div>
        </xpath>
    </template>

    <template id="all_user_card" inherit_id="website_profile.all_user_card">
        <xpath expr="//td[hasclass('all_user_badge_count')]" position="after">
            <td class="align-middle text-end pe-3 text-nowrap all_user_certification_count">
                <b t-esc="user['certification_count']"/> <span class="text-muted small fw-bold">Certifications</span>
            </td>
        </xpath>
    </template>

    <template id="badge_content" inherit_id="website_profile.badge_content">
        <xpath expr="//div[@id='website_profile_badges']" position="after">
            <t t-if="certification_badges">
                <div class="row">
                    <div class="col-12">
                        <h1 class="mt-4 mt-lg-2">Certification Badges</h1>
                        <p class="lead">
                            You can gain badges by passing certifications. Here is a list of all available certification badges.
                            <br />Follow the links to reach new heights and skill up!
                        </p>
                        <div class="row col-12 align-items-center p-0" t-foreach="certification_badges" t-as="badge">
                            <div class="col-3">
                                <t t-call="website_profile.badge_header">
                                    <t t-if="badge.id in certification_badge_urls" t-set="badge_url" t-value="certification_badge_urls[badge.id]"/>
                                </t>
                            </div>
                            <div class="col-6">
                                <span t-field="badge.description"/>
                            </div>
                            <div class="col-3 text-end">
                                <b t-esc="badge.granted_users_count"/>
                                <i class="text-muted"> awarded users</i>
                            </div>
                        </div>
                    </div>
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
    <menuitem name="Certifications"
        id="website_slides_menu_courses_certification"
        parent="website_slides.website_slides_menu_courses"
        sequence="3"
        action="survey_survey_action_slides"/>
</odoo>

```

## File: views\website_slides_templates_course.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="course_main" inherit_id="website_slides.course_main" name="Certification Course Main">
            <xpath expr="//div[@id='wrap']" position="attributes">
                <attribute name="t-attf-class" separator=" " add="#{'o_wss_certification_channel' if channel.nbr_certification > 0 else ''}"/>
            </xpath>

            <xpath expr="//div[@id='courseMainTabContent']//div[@id='home']/t" position="before">
                <t t-set="first_slide" t-value="channel.slide_content_ids[0] if len(channel.slide_content_ids) > 0 else None"/>
                <div t-if="channel.nbr_certification > 0 and channel.is_member and channel.completion == 0" class="alert alert-success d-flex align-items-center justify-content-between flex-wrap">
                    <div>Begin your <b>certification</b> today!</div>

                    <a t-attf-href="#{'/slides_survey/slide/get_certification_url?slide_id=%s' %(first_slide.id) if first_slide.slide_category == 'certification' and channel.total_slides == 1 else '/slides/slide/%s?fullscreen=1' %(slug(first_slide))}" class="btn btn-success mt-2 mt-sm-0">
                        <span>Start Now</span><i class="oi oi-chevron-right ms-2 align-middle"/>
                    </a>
                </div>
            </xpath>
        </template>

        <template id="course_slides_list_slide_inherit_survey" inherit_id="website_slides.course_slides_list_slide">
            <xpath expr="//a[hasclass('o_wslides_js_slides_list_slide_link')]" position="attributes">
                <attribute name="t-attf-href">#{'/slides_survey/slide/get_certification_url?slide_id=%s' %(slide.id) if slide.slide_category == 'certification' and slide.channel_id.total_slides == 1 else '/slides/slide/%s' %(slug(slide))}</attribute>
            </xpath>
            <xpath expr="//a[@name='o_wslides_list_slide_add_quizz']" position="attributes">
                <attribute name="t-if">channel.can_upload and not slide.question_ids and slide.slide_category != 'certification'</attribute>
            </xpath>
            <xpath expr="//a[@name='o_wslides_slide_toggle_is_preview']" position="attributes">
                <attribute name="t-attf-class">#{'d-none' if slide.slide_type == 'certification' else ''}</attribute>
            </xpath>
        </template>

        <template id="course_slides_list_inherit_survey" inherit_id="website_slides.course_slides_list">
            <xpath expr="//div[hasclass('o_wslides_content_actions')]" position="inside">
                <div class="o_wslides_survey_certification_upload_toast"/>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: views\website_slides_templates_homepage.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="courses_home_inherit_survey" inherit_id="website_slides.courses_home">
            <xpath expr="//a[hasclass('o_wslides_home_all_slides')]" position="after">
                <a class="nav-link nav-link d-flex" href="/slides/all?slide_category=certification">
                    <t t-call="website_slides_survey.o_wss_certification_icon"/>
                    <span class="ms-1">Certifications</span>
                </a>
            </xpath>
        </template>

        <template id="course_card_inherit_survey" inherit_id="website_slides.course_card">
            <xpath expr="//div/div" position="after">
                <div t-if="channel.nbr_certification > 0" class="position-absolute py-1 px-2 h5" style="right:0; top:0">
                    <t t-call="website_slides_survey.o_wss_certification_icon"/>
                </div>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: views\website_slides_templates_lesson.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="slide_content_detailed" inherit_id="website_slides.slide_content_detailed">
            <xpath expr="//div[hasclass('o_wslides_lesson_content_type')]" position="inside">
                <div t-if="slide.slide_category == 'certification' and not channel_progress[slide.id].get('completed')" class="col mt32 mb8 text-center">
                    <a role="button"
                        class="btn btn-primary btn-lg"
                        id="start_certification"
                        t-att-href="'/slides_survey/slide/get_certification_url?slide_id=%s' %(slide.id)"
                        target="_blank">
                        <i class="fa fa-fw fa-graduation-cap" role="img"/>
                        <t t-if="slide.channel_id.is_member and slide.channel_id.active">Begin Certification</t>
                        <t t-else="">Test Certification</t>
                    </a>
                </div>
                <div t-if="slide.slide_category == 'certification' and channel_progress[slide.id].get('completed')" class="col mt32 mb8 text-center">
                    <h4 class="mb-3">Congratulations, you passed the Certification!</h4>
                    <a role="button" class="btn btn-primary btn-lg" t-att-href="'/survey/%s/get_certification' % slide.survey_id.id">
                        <i class="fa fa-fw fa-trophy" role="img" aria-label="Download certification" title="Download certification"/> Download certification
                    </a>
                </div>
                <div t-if="slide.slide_category == 'certification' and slide.channel_id.can_upload" class="col mb8 d-flex justify-content-center">
                    <a class="my-4" t-att-href="'/web#id=' + str(slide.survey_id.id)  + '&amp;model=survey.survey&amp;view_type=form'">
                        <i class="oi oi-arrow-right me-1"/>Add Questions to this Survey
                    </a>
                </div>
            </xpath>
            <xpath expr="//a[hasclass('o_wslides_done_button')][1]" position="after">
                <a t-elif="not slide_completed and slide.slide_category == 'certification'"
                    role="button"
                    t-attf-class="o_wslides_done_button btn btn-primary border text-white me-2"
                    href="#start_certification">
                    Take Quiz
                </a>
            </xpath>
            <xpath expr="//a[hasclass('o_wslides_undone_button')][1]" position="after">
                <a t-elif="slide_completed and slide.slide_category == 'certification'"
                    class="o_wslides_undone_button btn btn-primary border text-white me-2 disabled"
                    aria-disabled="true" title="Certifications you have passed cannot be marked as not done">
                    Mark To Do
                </a>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: views\website_slides_templates_lesson_fullscreen.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <template id="slide_fullscreen_sidebar_category" inherit_id="website_slides.slide_fullscreen_sidebar_category">
        <xpath expr="//li[@t-att-data-id='slide.id']" position="attributes">
            <attribute name="t-att-data-certification-id">slide.survey_id.id</attribute>
            <attribute name="t-att-data-is-member">slide.channel_id.is_member</attribute>
            <attribute name="t-att-data-can-upload">channel.can_upload</attribute>
        </xpath>
    </template>

</data></odoo>

```

## File: views\website_slides_templates_utils.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="slide_sidebar_done_button" inherit_id="website_slides.slide_sidebar_done_button">
    <xpath expr="//button[hasclass('o_wslides_button_uncompleted')]/i[1]" position="after">
        <i t-elif="slide_completed and slide.slide_category == 'certification'"
            class="o_wslides_slide_completed fa fa-check fa-fw text-success fa-lg"
            t-att-data-slide-id="slide.id"
            title="Certifications you have passed cannot be marked as not done"/>
    </xpath>
</template>

</odoo>

```

