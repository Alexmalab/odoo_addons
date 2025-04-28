# Odoo Module: website_slides_survey

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
    'name': "Course Certifications",
    'summary': 'Add certification capabilities to your courses',
    'description': """This module lets you use the full power of certifications within your courses.""",
    'category': 'Hidden',
    'version': '1.0',
    'depends': ['website_slides', 'survey'],
    'installable': True,
    'auto_install': True,
    'data': [
        'views/assets.xml',
        'views/slide_channel_views.xml',
        'views/slide_slide_views.xml',
        'views/survey_survey_views.xml',
        'views/website_slides_menu_views.xml',
        'views/website_slides_templates_course.xml',
        'views/website_slides_templates_lesson.xml',
        'views/website_slides_templates_lesson_fullscreen.xml',
        'views/website_slides_templates_homepage.xml',
        'views/survey_templates.xml',
        'views/website_profile.xml',
        'data/mail_template_data.xml',
        'data/gamification_data.xml',
    ],
    'demo': [
        'data/survey_demo.xml',
        'data/slide_slide_demo.xml',
        'data/survey.user_input_line.csv',
    ],
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request


class WebsiteSlidesSurvey(http.Controller):
    @http.route(['/slides_survey/certification/search_read'], type='json', auth='user', methods=['POST'], website=True)
    def slides_certification_search_read(self, fields):
        return {
            'read_results': request.env['survey.survey'].search_read([('certificate', '=', True)], fields),
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
from odoo.http import request

from odoo.addons.website_slides.controllers.main import WebsiteSlides
from collections import defaultdict


class WebsiteSlides(WebsiteSlides):

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
        return werkzeug.utils.redirect(certification_url)

    # Utils
    # ---------------------------------------------------
    def _set_completed_slide(self, slide):
        if slide.slide_type == 'certification':
            raise werkzeug.exceptions.Forbidden(_("Certification slides are completed when the survey is succeeded."))
        return super(WebsiteSlides, self)._set_completed_slide(slide)

    def _get_valid_slide_post_values(self):
        result = super(WebsiteSlides, self)._get_valid_slide_post_values()
        result.append('survey_id')
        return result

    # Profile
    # ---------------------------------------------------
    def _prepare_user_slides_profile(self, user):
        values = super(WebsiteSlides, self)._prepare_user_slides_profile(user)
        values.update({
            'certificates': self._get_users_certificates(user)[user.id]
        })
        return values

    # All Users Page
    # ---------------------------------------------------
    def _prepare_all_users_values(self, users):
        result = super(WebsiteSlides, self)._prepare_all_users_values(users)
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
            ('quizz_passed', '=', True),
            ('slide_partner_id.survey_quizz_passed', '=', True)
        ]
        certificates = request.env['survey.user_input'].sudo().search(domain)
        users_certificates = {
            user.id: [
                certificate for certificate in certificates if certificate.partner_id == user.partner_id
            ] for user in users
        }
        return users_certificates

```

## File: controllers\survey.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.survey.controllers.main import Survey


class Survey(Survey):
    def _prepare_survey_finished_values(self, survey, answer, token=False):
        result = super(Survey, self)._prepare_survey_finished_values(survey, answer, token)
        if answer.slide_id:
            result['channel_url'] = answer.slide_id.channel_id.website_url

        return result

    def _prepare_retry_additional_values(self, answer):
        result = super(Survey, self)._prepare_retry_additional_values(answer)
        if answer.slide_id:
            result['slide_id'] = answer.slide_id.id
        if answer.slide_partner_id:
            result['slide_partner_id'] = answer.slide_partner_id.id

        return result

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main
from . import slides
from . import survey

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
            ('survey_quizz_passed', '=', True),
            ('slide_id.slide_type', '=', 'certification')
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
            <field name="name">Certification failed email</field>
            <field name="model_id" ref="model_survey_user_input" />
            <field name="subject">You have failed the course: ${object.slide_partner_id.channel_id.name}</field>
            <field name="partner_to">${object.partner_id.id}</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px; font-size: 13px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        Dear ${object.partner_id.name or 'participant'}<br/><br/>
        Unfortunately, you have failed the certification and are no longer a member of the course: ${object.slide_partner_id.channel_id.name}.<br/><br/>
        Don't hesitate to enroll again!
        <div style="margin: 16px 0px 16px 0px;">
            <a href="${(object.slide_partner_id.channel_id.website_url) | safe}"
                style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                Enroll now
            </a>
        </div>
        Thank you for your participation.
    </p>
</div>
            </field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="auto_delete" eval="True"/>
            <field name="user_signature" eval="False"/>
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
        <field name="slide_type">certification</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_5_furn2"/>
        <field name="category_id" ref="website_slides.slide_category_demo_5_0"/>
        <field name="survey_id" ref="website_slides_survey.furniture_certification"/>
        <field name="is_published" eval="True"/>
        <field name="is_preview" eval="False"/>
        <field name="description">Now that you have completed the course, it's time to test your knowledge!</field>
    </record>

    <!-- CHANNEL 6: DIY Furniture -->
    <!-- ======================================= -->
    <record id="slide_slide_demo_6_0" model="slide.slide">
        <field name="name">DIY Furniture Certification</field>
        <field name="sequence">1</field>
        <field name="image_1920" type="base64" file="website_slides/static/src/img/slide_demo_furniture.jpg"/>
        <field name="slide_type">certification</field>
        <field name="channel_id" ref="website_slides.slide_channel_demo_6_furn3"/>
        <field name="category_id" eval="False"/>
        <field name="survey_id" ref="website_slides_survey.furniture_certification"/>
        <field name="is_published" eval="True"/>
        <field name="is_preview" eval="False"/>
        <field name="description">It's time to test your knowledge!</field>
    </record>

</data></odoo>

```

## File: data\survey.user_input_line.csv

```csv
id,user_input_id:id,question_id:id,skipped,answer_type,value_text,value_number,value_date,value_free_text,value_suggested:id,value_suggested_row:id
furniture_certification_answer_1_p1_q1,furniture_certification_answer_1,furniture_certification_page_1_question_1,False,suggestion,,,,,furniture_certification_page_1_question_1_choice_2,
furniture_certification_answer_1_p1_q2_1,furniture_certification_answer_1,furniture_certification_page_1_question_2,False,suggestion,,,,,furniture_certification_page_1_question_2_choice_1,
furniture_certification_answer_1_p1_q2_2,furniture_certification_answer_1,furniture_certification_page_1_question_2,False,suggestion,,,,,furniture_certification_page_1_question_2_choice_3,
furniture_certification_answer_1_p1_q2_3,furniture_certification_answer_1,furniture_certification_page_1_question_2,False,suggestion,,,,,furniture_certification_page_1_question_2_choice_4,
furniture_certification_answer_1_p1_q3,furniture_certification_answer_1,furniture_certification_page_1_question_3,False,free_text,,,,"I really liked the videos, you are awesome!",,
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
            <field name="state">open</field>
            <field name="access_mode">public</field>
            <field name="users_can_go_back" eval="True" />
            <field name="users_login_required" eval="True" />
            <field name="scoring_type" >scoring_with_answers</field>
            <field name="certificate" eval="True"></field>
            <field name="certification_mail_template_id" ref="survey.mail_template_certification"></field>
            <field name="is_attempts_limited" eval="True" />
            <field name="attempts_limit">3</field>
            <field name="description">&lt;p&gt;Test your furniture knowledge!.&lt;/p&gt;</field>
            <field name="thank_you_message">&lt;p&gt;&lt;/p&gt;</field>
        </record>
        <!-- Page 1 -->
        <record model="survey.question" id="furniture_certification_page_1">
            <field name="title">Furniture</field>
            <field name="survey_id" ref="furniture_certification" />
            <field name="sequence">1</field>
            <field name="is_page" eval="True"/>
            <field name="description">&lt;p&gt;Test your furniture knowledge!&lt;/p&gt;</field>
        </record>
        <!-- Question and predefined answer 1 -->
        <record model="survey.question" id="furniture_certification_page_1_question_1">
            <field name="survey_id" ref="furniture_certification" />
            <field name="sequence">2</field>
            <field name="title">What type of wood is the best for furniture?</field>
            <field name="question_type">simple_choice</field>
            <field name="display_mode">dropdown</field>
            <field name="constr_mandatory" eval="True" />
        </record>
        <record model="survey.label" id="furniture_certification_page_1_question_1_choice_1">
            <field name="question_id" ref="furniture_certification_page_1_question_1"/>
            <field name="sequence">1</field>
            <field name="value">Fir</field>
        </record>
        <record model="survey.label" id="furniture_certification_page_1_question_1_choice_2">
            <field name="question_id" ref="furniture_certification_page_1_question_1"/>
            <field name="sequence">2</field>
            <field name="value">Oak</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">2.0</field>
        </record>
        <record model="survey.label" id="furniture_certification_page_1_question_1_choice_3">
            <field name="question_id" ref="furniture_certification_page_1_question_1"/>
            <field name="sequence">3</field>
            <field name="value">Ash</field>
        </record>
        <record model="survey.label" id="furniture_certification_page_1_question_1_choice_4">
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
            <field name="column_nb">4</field>
        </record>
        <record model="survey.label" id="furniture_certification_page_1_question_2_choice_1">
            <field name="question_id" ref="furniture_certification_page_1_question_2"/>
            <field name="sequence">1</field>
            <field name="value">Chair</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.label" id="furniture_certification_page_1_question_2_choice_2">
            <field name="question_id" ref="furniture_certification_page_1_question_2"/>
            <field name="sequence">2</field>
            <field name="value">Table</field>
            <field name="answer_score">-1.0</field>
        </record>
        <record model="survey.label" id="furniture_certification_page_1_question_2_choice_3">
            <field name="question_id" ref="furniture_certification_page_1_question_2"/>
            <field name="sequence">3</field>
            <field name="value">Desk</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.label" id="furniture_certification_page_1_question_2_choice_4">
            <field name="question_id" ref="furniture_certification_page_1_question_2"/>
            <field name="sequence">4</field>
            <field name="value">Shelve</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.label" id="furniture_certification_page_1_question_2_choice_5">
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
            <field name="question_type">free_text</field>
        </record>

        <record model="survey.user_input" id="furniture_certification_answer_1">
            <field name="survey_id" ref="website_slides_survey.furniture_certification" />
            <field name="input_type">manually</field>
            <field name="partner_id" ref="base.res_partner_address_3"/>
            <field name="email">douglas.fletcher51@example.com</field>
            <field name="state">done</field>
            <field name="question_ids" eval="[
                    ref('furniture_certification_page_1_question_1'),
                    ref('furniture_certification_page_1_question_2'),
                    ref('furniture_certification_page_1_question_3'),
                ]"/>
        </record>
        <record model="survey.user_input" id="furniture_certification_answer_2">
            <field name="survey_id" ref="website_slides_survey.furniture_certification" />
            <field name="input_type">manually</field>
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


class Channel(models.Model):
    _inherit = 'slide.channel'

    nbr_certification = fields.Integer("Number of Certifications", compute='_compute_slides_statistics', store=True)

```

## File: models\slide_slide.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SlidePartnerRelation(models.Model):
    _inherit = 'slide.slide.partner'

    user_input_ids = fields.One2many('survey.user_input', 'slide_partner_id', 'Certification attempts')
    survey_quizz_passed = fields.Boolean('Certification Quizz Passed', compute='_compute_survey_quizz_passed', store=True)

    @api.depends('partner_id', 'user_input_ids.quizz_passed')
    def _compute_survey_quizz_passed(self):
        passed_user_inputs = self.env['survey.user_input'].sudo().search([
            ('slide_partner_id', 'in', self.ids),
            ('quizz_passed', '=', True)
        ])
        passed_slide_partners = passed_user_inputs.mapped('slide_partner_id')
        for record in self:
            record.survey_quizz_passed = record in passed_slide_partners

    @api.model_create_multi
    def create(self, vals_list):
        res = super(SlidePartnerRelation, self).create(vals_list)
        completed = res.filtered('survey_quizz_passed')
        if completed:
            completed.write({'completed': True})
        return res

    def _write(self, vals):
        res = super(SlidePartnerRelation, self)._write(vals)
        if vals.get('survey_quizz_passed'):
            self.sudo().write({'completed': True})
        return res


class Slide(models.Model):
    _inherit = 'slide.slide'

    slide_type = fields.Selection(selection_add=[('certification', 'Certification')])
    survey_id = fields.Many2one('survey.survey', 'Certification')
    nbr_certification = fields.Integer("Number of Certifications", compute='_compute_slides_statistics', store=True)

    _sql_constraints = [
        ('check_survey_id', "CHECK(slide_type != 'certification' OR survey_id IS NOT NULL)", "A slide of type 'certification' requires a certification."),
        ('check_certification_preview', "CHECK(slide_type != 'certification' OR is_preview = False)", "A slide of type certification cannot be previewed."),
    ]

    @api.onchange('survey_id')
    def _on_change_survey_id(self):
        if self.survey_id:
            self.slide_type = 'certification'

    @api.model
    def create(self, values):
        rec = super(Slide, self).create(values)
        if rec.survey_id:
            rec.slide_type = 'certification'
        return rec

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
        for slide in self.filtered(lambda slide: slide.slide_type == 'certification' and slide.survey_id):
            if slide.channel_id.is_member:
                user_membership_id_sudo = slide.user_membership_id.sudo()
                if user_membership_id_sudo.user_input_ids:
                    last_user_input = next(user_input for user_input in user_membership_id_sudo.user_input_ids.sorted(
                        lambda user_input: user_input.create_date, reverse=True
                    ))
                    certification_urls[slide.id] = last_user_input._get_survey_url()
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
                    certification_urls[slide.id] = user_input._get_survey_url()
            else:
                user_input = slide.survey_id.sudo()._create_answer(
                    partner=self.env.user.partner_id,
                    check_attempts=False,
                    test_entry=True, **{
                        'slide_id': slide.id
                    }
                )
                certification_urls[slide.id] = user_input._get_survey_url()
        return certification_urls

```

## File: models\survey_survey.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Survey(models.Model):
    _inherit = 'survey.survey'

    def _check_answer_creation(self, user, partner, email, test_entry=False, check_attempts=True, invite_token=False):
        """ Overridden to allow website_publisher to test certifications. """
        self.ensure_one()
        if test_entry and user.has_group('website.group_website_publisher'):
            return True

        return super(Survey, self)._check_answer_creation(user, partner, email, test_entry=test_entry, check_attempts=check_attempts, invite_token=invite_token)

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
        help="Slide membership information for the logged in user")

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
        """ If the user fails his last attempt at a course certification,
        we remove him from the members of the course (and he has to enroll again).
        He receives an email in the process notifying him of his failure and suggesting
        he enrolls to the course again.

        The purpose is to have a 'certification flow' where the user can re-purchase the
        certification when they have failed it."""

        if self:
            user_inputs = self.search([
                ('id', 'in', self.ids),
                ('state', '=', 'done'),
                ('quizz_passed', '=', False),
                ('slide_partner_id', '!=', False)
            ])

            if user_inputs:
                for user_input in user_inputs:
                    removed_memberships_per_partner = {}
                    if user_input.survey_id._has_attempts_left(user_input.partner_id, user_input.email, user_input.invite_token):
                        # skip if user still has attempts left
                        continue

                    self.env.ref('website_slides_survey.mail_template_user_input_certification_failed').send_mail(
                        user_input.id, notif_layout="mail.mail_notification_light"
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

## File: static\src\img\certification.svg

```svg
<svg width="12" height="12" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"><defs><path d="M7.158 11.549a2.005 2.005 0 0 1-3.105-1.286 2.005 2.005 0 0 1-1.286-3.105 2.005 2.005 0 0 1 1.286-3.105 2.005 2.005 0 0 1 3.105-1.286 2.005 2.005 0 0 1 3.105 1.286 2.005 2.005 0 0 1 1.286 3.105 2.005 2.005 0 0 1-1.286 3.105 2.005 2.005 0 0 1-3.105 1.286z" id="a"/><filter filterUnits="objectBoundingBox" id="b"><feMorphology radius=".5" in="SourceAlpha" result="shadowSpreadInner1"/><feOffset in="shadowSpreadInner1" result="shadowOffsetInner1"/><feComposite in="shadowOffsetInner1" in2="SourceAlpha" operator="arithmetic" k2="-1" k3="1" result="shadowInnerInner1"/><feColorMatrix values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0.0749562937 0" in="shadowInnerInner1"/></filter><path id="d" d="M6.04788912 6.85018391L7.55257375 8.20939291 9.91700506 6 11 6.947144 7.55257375 10 5 7.75171917z"/><filter x="-2.5%" y="-3.8%" width="105%" height="115%" filterUnits="objectBoundingBox" id="c"><feOffset dy=".3" in="SourceAlpha" result="shadowOffsetOuter1"/><feColorMatrix values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0.233200393 0" in="shadowOffsetOuter1"/></filter></defs><g transform="translate(-2 -2)" fill="none" fill-rule="evenodd"><g transform="rotate(22 5.57 9.903)"><use fill="#00C9FF" xlink:href="#a"/><use fill="#000" filter="url(#b)" xlink:href="#a"/><path stroke="#FFF" stroke-width=".7" d="M7.158 11.96a2.355 2.355 0 0 1-3.395-1.407 2.355 2.355 0 0 1-1.406-3.395 2.355 2.355 0 0 1 1.406-3.395 2.355 2.355 0 0 1 3.395-1.406 2.355 2.355 0 0 1 3.395 1.406 2.355 2.355 0 0 1 1.406 3.395 2.355 2.355 0 0 1-1.406 3.395 2.355 2.355 0 0 1-3.395 1.406z"/></g><use fill="#000" filter="url(#c)" xlink:href="#d"/><use fill="#FFF" xlink:href="#d"/></g></svg>
```

## File: static\src\js\slides_course_fullscreen_player.js

```javascript
odoo.define('website_slides_survey.fullscreen', function (require) {
"use strict";

var core = require('web.core');
var QWeb = core.qweb;
var Fullscreen = require('website_slides.fullscreen');

Fullscreen.include({
    xmlDependencies: (Fullscreen.prototype.xmlDependencies || []).concat(
        ["/website_slides_survey/static/src/xml/website_slides_fullscreen.xml"]
    ),

    /**
     * Extend the _renderSlide method so that slides of type "certification"
     * are also taken into account and rendered correctly
     *
     * @private
     * @override
     */
    _renderSlide: function (){
        var def = this._super.apply(this, arguments);
        var $content = this.$('.o_wslides_fs_content');
        if (this.get('slide').type === "certification"){
            $content.html(QWeb.render('website.slides.fullscreen.certification',{widget: this}));
        }
        return Promise.all([def]);
    },
});
});



```

## File: static\src\js\slides_upload.js

```javascript
odoo.define('website_slides_survey.upload_modal', function (require) {
"use strict";

var core = require('web.core');
var _t = core._t;
var SlidesUpload = require('website_slides.upload_modal');

/**
 * Management of the new 'certification' slide_type
 */
SlidesUpload.SlideUploadDialog.include({
    /**
     * Overridden to add the "certification" slide type
     *
     * @override
     * @private
     */
    _setup: function () {
        this._super.apply(this, arguments);
        this.slide_type_data['certification'] = {
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
                return self._rpc({
                    route: '/slides_survey/certification/search_read',
                    params: {
                        fields: ['title'],
                    }
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
        if ($certificationInput.length !== 0){
            var $select2Container = $certificationInput
                .closest('.form-group')
                .find('.select2-container');
            $select2Container.removeClass('is-invalid is-valid');
            if ($certificationInput.is(':invalid')) {
                $select2Container.addClass('is-invalid');
            } else if ($certificationInput.is(':valid')) {
                $select2Container.addClass('is-valid');
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
    _getSelect2DropdownValues: function (){
        var result = this._super.apply(this, arguments);

        var certificateValue = this.$('#certification_id').select2('data');
        if (certificateValue) {
            result['survey_id'] =  certificateValue.id;
        }
        return result;
    }
});

SlidesUpload.websiteSlidesUpload.include({
    xmlDependencies: (SlidesUpload.websiteSlidesUpload.prototype.xmlDependencies || []).concat(
        ["/website_slides_survey/static/src/xml/website_slide_upload.xml"]
    ),
});

});

```

## File: static\src\xml\website_slides_fullscreen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>

    <t t-name="website.slides.fullscreen.certification">
        <div class="justify-content-center align-self-center">
            <div t-if="widget.get('slide').type === 'certification' &amp;&amp; !widget.get('slide').completed" class="">
                <a class="btn btn-primary" t-att-href="'/slides_survey/slide/get_certification_url?slide_id=' + widget.get('slide').id" target="_blank">
                    <i class="fa fa-trophy"/>
                    <span t-if="widget.get('slide').isMember"> Pass Certification</span>
                    <span t-else="">Test Certification</span>
                </a>
            </div>
            <div class="" t-if="widget.get('slide').type === 'certification' &amp;&amp; widget.get('slide').completed">
                <a role="button" class="btn btn-primary" t-att-href="'/survey/' + widget.get('slide').certificationId + '/get_certification'">
                    <i class="fa fa-fw fa-trophy" role="img" aria-label="Download certification" title="Download certification"/> Download certification
                </a>
            </div>
        </div>
    </t>

    <t t-extend="website.slides.fullscreen.title">
        <t t-jquery=".o_wslides_fs_slide_title_span" t-operation="before">
            <i t-if="widget.get('slide').type === 'certification'" class="fa fa-trophy mr-2 text"></i>
        </t>
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
                <div class="form-group row">
                    <label for="certification_id" class="col-form-label col-md-3">Certification</label>
                    <div class="controls col-md-9">
                        <input class="form-control" id="certification_id" required="required"/>
                    </div>
                </div>
                <t t-call="website.slide.upload.modal.common"/>
                <div class="form-group row">
                    <div class="col-md-4">
                        <div class="img-thumbnail">
                            <div class="o_slide_preview">
                                <img src="/website_slides/static/src/img/document.png" id="slide-image" title="Content Preview" alt="Content Preview" class="img-fluid"/>
                            </div>
                        </div>
                    </div>
                    <div class="col-md-8">
                        <ul class="list-group">
                            <li class="list-group-item">
                                <h5 class="list-group-item-heading">
                                    <label for="upload" class="col-form-label">Cover Image File</label>
                                </h5>
                                <input id="upload" name="file" class="form-control" accept="image/*" type="file" required="required" data-prevent-onchange="1"/>
                            </li>
                        </ul>
                    </div>
                </div>
                <canvas id="data_canvas" class="d-none"></canvas>
            </form>
        </div>
    </t>
</template>

```

## File: views\assets.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <data>
        <template id="assets_frontend" inherit_id="website.assets_frontend" name="Slides Certification">
            <xpath expr="//link[last()]" position="after">
                <link rel="stylesheet" type="text/scss" href="/website_slides_survey/static/src/scss/website_slides_survey.scss"/>
            </xpath>
            <xpath expr="//script[last()]" position="after">
                <script type="text/javascript" src="/website_slides_survey/static/src/js/slides_upload.js"/>
                <script type="text/javascript" src="/website_slides_survey/static/src/js/slides_course_fullscreen_player.js"/>
            </xpath>
        </template>
    </data>
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
            <xpath expr="//span[@name='members_done_count_label']" position="replace">
                <span class="o_stat_text" attrs="{'invisible': [('nbr_certification', '>', 0)]}">Finished</span>
                <span class="o_stat_text" attrs="{'invisible': [('nbr_certification', '=', 0)]}">Certified</span>
            </xpath>
            <xpath expr="//field[@name='nbr_webpage']" position="after">
                <field  name="nbr_certification" string="Certifications"/>
            </xpath>
            <xpath expr="//field[@name='slide_type']" position="after">
                <field name="survey_id"/>
            </xpath>
            <xpath expr="//create[@name='add_slide_lesson']" position="after">
                <create name="add_slide_certificate" string="Add Certification" groups="survey.group_survey_user" context="{'default_slide_type': 'certification'}"/>
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

## File: views\slide_slide_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="slide_slide_view_form" model="ir.ui.view">
        <field name="name">slide.slide.view.form.inherit.survey</field>
        <field name="model">slide.slide</field>
        <field name="inherit_id" ref="website_slides.view_slide_slide_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='slide_type']" position="after">
                <field name="survey_id"
                    attrs="{'invisible': [('slide_type', '!=', 'certification')], 'required': [('slide_type', '=', 'certification')]}"
                    domain="[('certificate', '=', True)]" context="{'default_certificate': True, 'default_scoring_type': 'scoring_without_answers'}"/>
            </xpath>
        </field>
    </record>

    <record id="slide_slide_action_certification" model="ir.actions.act_window">
        <field name="name">Certifications</field>
        <field name="res_model">slide.slide</field>
        <field name="view_mode">tree,form,graph</field>
        <field name="domain">[('slide_type', '=', 'certification')]</field>
        <field name="context">{'default_slide_type': 'certification'}</field>
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
    <record id="survey_survey_action_slides" model="ir.actions.act_window">
        <field name="name">Certifications</field>
        <field name="res_model">survey.survey</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="domain">[('certificate', '=', True)]</field>
        <field name="context">{'default_certificate': True, 'default_scoring_type': 'scoring_with_answers'}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new certification
            </p>
        </field>
    </record>

    <record id="survey_survey_action_slides_report" model="ir.actions.act_window">
        <field name="name">Certifications</field>
        <field name="res_model">survey.survey</field>
        <field name="view_mode">tree,kanban,form,graph</field>
        <field name="domain">[('certificate', '=', True)]</field>
        <field name="context">{'default_certificate': True, 'default_scoring_type': 'scoring_with_answers'}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new certification
            </p>
        </field>
    </record>
</odoo>

```

## File: views\survey_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="survey_templates_inherit_website_slides" inherit_id="survey.sfinished">
            <xpath expr="//div[hasclass('jumbotron')]/div[last()]" position="after">
                <div t-if="channel_url" class="mt16">
                    <a role="button"
                        class="btn btn-primary btn-lg"
                        t-att-href="channel_url">
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
    </template>

     <template id="display_certificate">
        <t t-if="certificates">
            <div class="row">
                <div class="col-12 col-lg-6" t-foreach="certificates" t-as="certificate">
                    <div class="card mb-2">
                        <div class="card-body o_wprofile_slides_course_card_body p-0 d-flex">
                            <div class="pl-5 pr-4 rounded-left" t-attf-style="background-image: url(#{website.image_url(certificate.slide_id, 'image_128')}); background-position: center"/>
                            <div class="p-2 w-100">
                                <h5 class="mt-0 mb-1" t-esc="certificate.survey_id.title"/>
                                <div t-if="user.id == uid">
                                    <small class="font-weight-bold">Score : <span t-esc="certificate.quizz_score"/> %</small>
                                   <a role="button" class="float-right" t-att-href="'/survey/%s/get_certification' % certificate.survey_id.id">
                                        <i class="fa fa-download"/>
                                    </a>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </t>
        <t t-else="">
            <p class="text-muted">No certifications yet!</p>
        </t>
    </template>

    <template id="top3_user_card" inherit_id="website_profile.top3_user_card">
        <xpath expr="//div[hasclass('o_wprofile_top3_card_footer')]//div[last()]" position="after">
            <div class="col py-3"><b t-esc="user['certification_count']"/> <span class="text-muted">Certifications</span></div>
        </xpath>
    </template>

    <template id="all_user_card" inherit_id="website_profile.all_user_card">
        <xpath expr="//td[hasclass('all_user_badge_count')]" position="after">
            <td class="align-middle text-right pr-3 text-nowrap all_user_certification_count">
                <b t-esc="user['certification_count']"/> <span class="text-muted small font-weight-bold">Certifications</span>
            </td>
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

    <menuitem name="Certifications"
        id="website_slides_menu_report_certification"
        parent="website_slides.website_slides_menu_report"
        sequence="40"
        action="survey_survey_action_slides_report"/>
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

                    <a t-attf-href="#{'/slides_survey/slide/get_certification_url?slide_id=%s' %(first_slide.id) if first_slide.slide_type == 'certification' and channel.total_slides == 1 else '/slides/slide/%s?fullscreen=1' %(slug(first_slide))}" class="btn btn-success mt-2 mt-sm-0">
                        <span>Start Now</span><i class="fa fa-chevron-right ml-2 align-middle"/>
                    </a>
                </div>
            </xpath>
        </template>

        <template id="slide_icon_inherit_survey" inherit_id="website_slides.slide_icon">
            <xpath expr="//i[last()]" position="after">
                <i t-if="slide.slide_type == 'certification'" t-att-class="'fa fa-trophy %s' % icon_class"></i>
            </xpath>
        </template>

        <template id="course_slides_list_slide_inherit_survey" inherit_id="website_slides.course_slides_list_slide">
            <xpath expr="//a[hasclass('o_wslides_js_slides_list_slide_link')]" position="attributes">
                <attribute name="t-attf-href">#{'/slides_survey/slide/get_certification_url?slide_id=%s' %(slide.id) if slide.slide_type == 'certification' and slide.channel_id.total_slides == 1 else '/slides/slide/%s' %(slug(slide))}</attribute>
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
                <a class="nav-link nav-link d-flex" href="/slides/all?slide_type=certification">
                    <t t-call="website_slides_survey.o_wss_certification_icon"/>
                    <span class="ml-1">Certifications</span>
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
                <div t-if="slide.slide_type == 'certification' and not channel_progress[slide.id].get('completed')" class="col mt32 mb8 d-flex justify-content-center">
                    <a role="button"
                        class="btn btn-primary btn-lg"
                        t-att-href="'/slides_survey/slide/get_certification_url?slide_id=%s' %(slide.id)">
                        <i class="fa fa-fw fa-graduation-cap" role="img"/>
                        <t t-if="slide.channel_id.is_member and slide.channel_id.active">Begin Certification</t>
                        <t t-else="">Test Certification</t>
                    </a>
                </div>
                <div t-if="slide.slide_type == 'certification' and channel_progress[slide.id].get('completed')" class="col mt32 mb8 d-flex justify-content-center">
                    <a role="button" class="btn btn-primary btn-lg" t-att-href="'/survey/%s/get_certification' % slide.survey_id.id">
                        <i class="fa fa-fw fa-trophy" role="img" aria-label="Download certification" title="Download certification"/> Download certification
                    </a>
                </div>
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
        </xpath>
    </template>

</data></odoo>

```

