# Odoo Module: hr_recruitment_survey

Category: Human Resources

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models
from . import wizard
from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': "Hr Recruitment Interview Forms",
    'version': '1.0',
    'category': 'Human Resources',
    'summary': 'Surveys',
    'description': """
Use interview forms during recruitment process.
This module is integrated with the survey module
to allow you to define interviews for different jobs.
    """,
    'depends': ['survey', 'hr_recruitment'],
    'data': [
        'security/ir.model.access.csv',
        'security/hr_recruitment_survey_security.xml',
        'data/mail_template_data.xml',
        'views/hr_job_views.xml',
        'views/hr_applicant_views.xml',
        'views/res_config_setting_views.xml',
        'views/survey_survey_views.xml',
        'views/survey_templates_statistics.xml',
    ],
    'demo': [
        'data/survey_demo.xml',
        'data/hr_job_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.survey.controllers import main


class ApplicantSurvey(main.Survey):
    def _prepare_retry_additional_values(self, answer):
        result = super()._prepare_retry_additional_values(answer)
        if answer.applicant_id:
            result["applicant_id"] = answer.applicant_id.id

        return result

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\hr_job_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!--Manage the job_id to get in hr.applicant-->
        <record id="hr.job_developer" model="hr.job">
            <field name="survey_id" ref="survey_recruitment_form"/>
        </record>
        <record id="hr.job_ceo" model="hr.job">
            <field name="survey_id" ref="survey_recruitment_form"/>
        </record>
        <record id="hr.job_cto" model="hr.job">
            <field name="survey_id" ref="survey_recruitment_form"/>
        </record>
        <record id="hr.job_consultant" model="hr.job">
            <field name="survey_id" ref="survey_recruitment_form"/>
        </record>
        <record id="hr.job_hrm" model="hr.job">
            <field name="survey_id" ref="survey_recruitment_form"/>
        </record>
        <record id="hr.job_marketing" model="hr.job">
            <field name="survey_id" ref="survey_recruitment_form"/>
        </record>
        <record id="hr.job_trainee" model="hr.job">
            <field name="survey_id" ref="survey_recruitment_form"/>
        </record>

    </data>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mail_template_applicant_interview_invite" model="mail.template">
            <field name="name">Applicant: Interview</field>
            <field name="model_id" ref="model_survey_user_input" />
            <field name="subject">Participate to {{ object.survey_id.display_name }} interview</field>
            <field name="email_to">{{ (object.partner_id.email_formatted or object.email) }}</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px; font-size: 13px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        Dear <t t-out="object.partner_id.name or 'applicant'">[applicant name]</t><br/><br/>
        <t>
            You've progressed through the recruitment process and we would like you to answer some questions.
        </t>
        <div style="margin: 16px 0px 16px 0px;">
            <a t-att-href="(object.get_start_url())"
                style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                <t>
                    Start the written interview
                </t>
            </a>
        </div>
        <t t-if="object.deadline">
            Please answer the interview for <t t-out="format_date(object.deadline)">[deadline date]</t>.<br/><br/>
        </t>
        <t>
            We wish you good luck! Thank you in advance for your participation.
        </t>
    </p>
</div>
            </field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: data\survey_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <record id="survey_recruitment_form" model="survey.survey">
        <field name="survey_type">recruitment</field>
        <field name="title">Recruitment Form</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="access_mode">token</field>
        <field name="users_can_go_back" eval="True"/>
        <field name="description" type="html">
<p>
    Please answer those questions to help recruitment officers to preprocess your application.
</p></field>
        <field name="description_done" type="html">
<p>
    Thank you for answering this survey. We will come back to you soon.
</p></field>
    </record>

    <record id="survey_recruitment_form_p1" model="survey.question">
        <field name="title">About you</field>
        <field name="survey_id" ref="survey_recruitment_form"/>
        <field name="is_page" eval="True" />
        <field name="question_type" eval="False"/>
        <field name="sequence">1</field>
        <field name="description" type="html">
<p>Please fill information about you: who you are, what are your education, experience, and activities.
    It will help us managing your application.</p></field>
    </record>
    <record id="survey_recruitment_form_p1_q1" model="survey.question">
        <field name="survey_id" ref="survey_recruitment_form"/>
        <field name="sequence">2</field>
        <field name="title">Which country are you from?</field>
        <field name="question_type">char_box</field>
    </record>
    <record id="survey_recruitment_form_p1_q2" model="survey.question">
        <field name="survey_id" ref="survey_recruitment_form"/>
        <field name="sequence">3</field>
        <field name="title">From which university did or will you graduate?</field>
        <field name="question_type">char_box</field>
    </record>
    <record id="survey_recruitment_form_p1_q3" model="survey.question">
        <field name="survey_id" ref="survey_recruitment_form"/>
        <field name="sequence">4</field>
        <field name="title">Were you referred by an employee?</field>
        <field name="question_type">char_box</field>
    </record>

    <record id="survey_recruitment_form_p1_q4" model="survey.question">
        <field name="survey_id" ref="survey_recruitment_form"/>
        <field name="sequence">4</field>
        <field name="title">Education</field>
        <field name="description" type="html">
<p>Please summarize your education history: schools, location, diplomas, ...</p></field>
        <field name="question_type">text_box</field>
    </record>
    <record id="survey_recruitment_form_p1_q5" model="survey.question">
        <field name="survey_id" ref="survey_recruitment_form"/>
        <field name="sequence">5</field>
        <field name="title">Past work experiences</field>
        <field name="description" type="html">
<p>Please summarize your education history: schools, location, diplomas, ...</p></field>
        <field name="question_type">text_box</field>
    </record>
    <record id="survey_recruitment_form_p1_q6" model="survey.question">
        <field name="survey_id" ref="survey_recruitment_form"/>
        <field name="sequence">6</field>
        <field name="title">Knowledge</field>
        <field name="description" type="html">
<p>What are your main knowledge regarding the job you are applying to?</p></field>
        <field name="question_type">text_box</field>
    </record>
    <record id="survey_recruitment_form_p1_q7" model="survey.question">
        <field name="survey_id" ref="survey_recruitment_form"/>
        <field name="sequence">7</field>
        <field name="title">Activities</field>
        <field name="description" type="html">
<p>Please tell us a bit more about yourself: what are your main activities, ...</p></field>
        <field name="question_type">text_box</field>
    </record>

    <record id="survey_recruitment_form_p1_q8" model="survey.question">
        <field name="survey_id" ref="survey_recruitment_form"/>
        <field name="sequence">8</field>
        <field name="title">What is important for you?</field>
        <field name="question_type">matrix</field>
        <field name="matrix_subtype">simple</field>
    </record>
    <record id="survey_recruitment_form_p1_q8_col1" model="survey.question.answer">
        <field name="question_id" ref="survey_recruitment_form_p1_q8"/>
        <field name="sequence">1</field>
        <field name="value">Not important</field>
    </record>
    <record id="survey_recruitment_form_p1_q8_col2" model="survey.question.answer">
        <field name="question_id" ref="survey_recruitment_form_p1_q8"/>
        <field name="sequence">3</field>
        <field name="value">Important</field>
    </record>
    <record id="survey_recruitment_form_p1_q8_col3" model="survey.question.answer">
        <field name="question_id" ref="survey_recruitment_form_p1_q8"/>
        <field name="sequence">4</field>
        <field name="value">Very important</field>
    </record>
    <record id="survey_recruitment_form_p1_q8_row1" model="survey.question.answer">
        <field name="matrix_question_id" ref="survey_recruitment_form_p1_q8"/>
        <field name="sequence">1</field>
        <field name="value">Having a good pay</field>
    </record>
    <record id="survey_recruitment_form_p1_q8_row2" model="survey.question.answer">
        <field name="matrix_question_id" ref="survey_recruitment_form_p1_q8"/>
        <field name="sequence">2</field>
        <field name="value">Getting on with colleagues</field>
    </record>
    <record id="survey_recruitment_form_p1_q8_row3" model="survey.question.answer">
        <field name="matrix_question_id" ref="survey_recruitment_form_p1_q8"/>
        <field name="sequence">3</field>
        <field name="value">Having a nice office environment</field>
    </record>
    <record id="survey_recruitment_form_p1_q8_row4" model="survey.question.answer">
        <field name="matrix_question_id" ref="survey_recruitment_form_p1_q8"/>
        <field name="sequence">5</field>
        <field name="value">Working with state of the art technology</field>
    </record>
    <record id="survey_recruitment_form_p1_q8_row5" model="survey.question.answer">
        <field name="matrix_question_id" ref="survey_recruitment_form_p1_q8"/>
        <field name="sequence">6</field>
        <field name="value">Office location</field>
    </record>
    <record id="survey_recruitment_form_p1_q8_row6" model="survey.question.answer">
        <field name="matrix_question_id" ref="survey_recruitment_form_p1_q8"/>
        <field name="sequence">7</field>
        <field name="value">Management quality</field>
    </record>
    <record id="survey_recruitment_form_p1_q8_row7" model="survey.question.answer">
        <field name="matrix_question_id" ref="survey_recruitment_form_p1_q8"/>
        <field name="sequence">8</field>
        <field name="value">Having freebies such as tea, coffee and stationery</field>
    </record>
    <record id="survey_recruitment_form_p1_q8_row8" model="survey.question.answer">
        <field name="matrix_question_id" ref="survey_recruitment_form_p1_q8"/>
        <field name="sequence">9</field>
        <field name="value">Getting perks such as free parking, gym passes</field>
    </record>
</data></odoo>

```

## File: models\hr_applicant.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta
from odoo import fields, models, _
from odoo.exceptions import UserError


class Applicant(models.Model):
    _inherit = "hr.applicant"

    survey_id = fields.Many2one('survey.survey', related='job_id.survey_id', string="Survey", readonly=True)
    response_ids = fields.One2many('survey.user_input', 'applicant_id', string="Responses")

    def action_print_survey(self):
        """ If response is available then print this response otherwise print survey form (print template of the survey) """
        self.ensure_one()
        sorted_interviews = self.response_ids\
            .filtered(lambda i: i.survey_id == self.survey_id)\
            .sorted(lambda i: i.create_date, reverse=True)
        if not sorted_interviews:
            action = self.survey_id.action_print_survey()
            action['target'] = 'new'
            return action

        answered_interviews = sorted_interviews.filtered(lambda i: i.state == 'done')
        if answered_interviews:
            action = self.survey_id.action_print_survey(answer=answered_interviews[0])
            action['target'] = 'new'
            return action
        action = self.survey_id.action_print_survey(answer=sorted_interviews[0])
        action['target'] = 'new'
        return action

    def action_send_survey(self):
        self.ensure_one()

        # if an applicant does not already has associated partner_id create it
        if not self.partner_id:
            if not self.partner_name:
                raise UserError(_('Please provide an applicant name.'))
            self.partner_id = self.env['res.partner'].sudo().create({
                'is_company': False,
                'name': self.partner_name,
                'email': self.email_from,
                'phone': self.partner_phone,
                'mobile': self.partner_phone
            })

        self.survey_id.check_validity()
        template = self.env.ref('hr_recruitment_survey.mail_template_applicant_interview_invite', raise_if_not_found=False)
        local_context = dict(
            default_applicant_id=self.id,
            default_partner_ids=self.partner_id.ids,
            default_survey_id=self.survey_id.id,
            default_use_template=bool(template),
            default_template_id=template and template.id or False,
            default_email_layout_xmlid='mail.mail_notification_light',
            default_deadline=fields.Datetime.now() + timedelta(days=15)
        )

        return {
            'type': 'ir.actions.act_window',
            'name': _("Send an interview"),
            'view_mode': 'form',
            'res_model': 'survey.invite',
            'target': 'new',
            'context': local_context,
        }

```

## File: models\hr_job.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class Job(models.Model):
    _inherit = "hr.job"

    survey_id = fields.Many2one(
        'survey.survey', "Interview Form",
        help="Choose an interview form for this job position and you will be able to print/answer this interview from all applicants who apply for this job")

    def action_test_survey(self):
        self.ensure_one()
        action = self.survey_id.action_test_survey()
        return action

    def action_new_survey(self):
        self.ensure_one()
        survey = self.env['survey.survey'].create({
            'title': _("Interview Form: %s", self.name),
        })
        self.write({'survey_id': survey.id})

        action = {
                'name': _('Survey'),
                'view_mode': 'form,list',
                'res_model': 'survey.survey',
                'type': 'ir.actions.act_window',
                'res_id': survey.id,
            }

        return action

```

## File: models\survey_survey.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models

class SurveySurvey(models.Model):
    _inherit = 'survey.survey'

    survey_type = fields.Selection(selection_add=[('recruitment', 'Recruitment')], ondelete={'recruitment': 'set default'})
    hr_job_ids = fields.One2many("hr.job", "survey_id", string="Job Position")

    @api.depends('survey_type')
    @api.depends_context('uid')
    def _compute_allowed_survey_types(self):
        super()._compute_allowed_survey_types()
        if self.env.user.has_group('hr_recruitment.group_hr_recruitment_interviewer') or \
                self.env.user.has_group('survey.group_survey_user'):
            self.allowed_survey_types = (self.allowed_survey_types or []) + ['recruitment']

    def get_formview_id(self, access_uid=None):
        if self.survey_type == 'recruitment':
            access_user = self.env['res.users'].browse(access_uid) if access_uid else self.env.user
            if not access_user.has_group('survey.group_survey_user'):
                if view := self.env.ref('hr_recruitment_survey.survey_survey_view_form', raise_if_not_found=False):
                    return view.id
        return super().get_formview_id(access_uid=access_uid)
    
    def action_survey_user_input_completed(self):
        action = super().action_survey_user_input_completed()
        if self.survey_type == 'recruitment':
            action.update({
                'domain': [('survey_id.survey_type', '=', 'recruitment')]
            })
        return action

```

## File: models\survey_user_input.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class SurveyUserInput(models.Model):
    _inherit = "survey.user_input"

    applicant_id = fields.Many2one('hr.applicant', string='Applicant', index='btree_not_null')

    def _mark_done(self):
        odoobot = self.env.ref('base.partner_root')
        for user_input in self:
            if user_input.applicant_id:
                body = _('The applicant "%s" has finished the survey.', user_input.applicant_id.partner_name)
                user_input.applicant_id.message_post(body=body, author_id=odoobot.id)
        return super()._mark_done()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_job
from . import hr_applicant
from . import survey_survey
from . import survey_user_input

```

## File: security\hr_recruitment_survey_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
    <!--
        Specific survey access rules for recruitment
        - The recruitment manager can CRUD survey / questions / question answers for survey_type == 'recruitment'
        - The recruitment manager can see all the answers of surveys being 'recruitment
        - The recruitment officer can see answers from survey type 'recruitment' unrestricted or in restricted users
        - The recruitment interviewers can send surveys to applicants and read their answers when they are set as
        interviewer for these applicants or the job they apply to.
        - All groups can send surveys of type 'recruiment' via the survey_invite wizard
    -->
        <!--special rights for recruitment manager on recruitment surveys-->
        <record id="survey_user_input_rule_recruitment_manager" model="ir.rule">
            <field name="name">Survey user input: recruitment manager: all recruitment</field>
            <field name="model_id" ref="survey.model_survey_user_input"/>
            <field name="domain_force">[('survey_id.survey_type', '=', 'recruitment')]</field>
            <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>
        <record id="survey_user_input_line_rule_recruitment_manager" model="ir.rule">
            <field name="name">Survey user input line: recruitment manager: all recruitment</field>
            <field name="model_id" ref="survey.model_survey_user_input_line"/>
            <field name="domain_force">[('survey_id.survey_type', '=', 'recruitment')]</field>
            <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>
        <record id="survey_survey_rule_recruitment_manager" model="ir.rule">
            <field name="name">Survey survey: recruitment manager: all recruitment</field>
            <field name="model_id" ref="survey.model_survey_survey"/>
            <field name="domain_force">[('survey_type', '=', 'recruitment')]</field>
            <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>
        <record id="survey_question_rule_recruitment_manager" model="ir.rule">
            <field name="name">Survey question: recruitment manager: all recruitment</field>
            <field name="model_id" ref="survey.model_survey_question"/>
            <field name="domain_force">[('survey_id.survey_type', '=', 'recruitment')]</field>
            <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>
        <record id="survey_question_answer_rule_recruitment_manager" model="ir.rule">
            <field name="name">Survey question answer: recruitment manager: all recruitment</field>
            <field name="model_id" ref="survey.model_survey_question_answer"/>
            <field name="domain_force">['|', ('question_id.survey_id.survey_type', '=', 'recruitment'),
                ('matrix_question_id.survey_id.survey_type', '=', 'recruitment')]</field>
            <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>
        <record id="survey_invite_recruitment_manager" model="ir.rule">
            <field name="name">Survey invite: recruitment manager: all recruitment</field>
            <field name="model_id" ref="survey.model_survey_invite"/>
            <field name="domain_force">[('survey_id.survey_type', '=', 'recruitment')]</field>
            <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>

        <!--special rights for recruitment officer on recruitment surveys-->
        <record id="survey_user_input_rule_recruitment_user" model="ir.rule">
            <field name="name">Survey user input: recruitment officer: unrestricted or in restricted users</field>
            <field name="model_id" ref="survey.model_survey_user_input"/>
            <field name="domain_force">[
                '&amp;', ('survey_id.survey_type', '=', 'recruitment'),
                '|',  ('survey_id.restrict_user_ids', 'in', user.id),
                        ('survey_id.restrict_user_ids', '=', False)]</field>
            <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_user'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>
        <record id="survey_user_input_line_rule_recruitment_user" model="ir.rule">
            <field name="name">Survey user input line: recruitment officer: unrestricted or in restricted users</field>
            <field name="model_id" ref="survey.model_survey_user_input_line"/>
            <field name="domain_force">[
                '&amp;', ('survey_id.survey_type', '=', 'recruitment'),
                '|',  ('survey_id.restrict_user_ids', 'in', user.id),
                        ('survey_id.restrict_user_ids', '=', False)]</field>
            <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_user'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>
        <record id="survey_invite_recruitment_user" model="ir.rule">
            <field name="name">Survey invite: recruitment officer: unrestricted or in restricted users</field>
            <field name="model_id" ref="survey.model_survey_invite"/>
            <field name="domain_force">[
                '&amp;', ('survey_id.survey_type', '=', 'recruitment'),
                '|',  ('survey_id.restrict_user_ids', 'in', user.id),
                        ('survey_id.restrict_user_ids', '=', False)]</field>
            <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_user'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>

        <!--special rights for recruitment interviewers on recruitment surveys-->
        <record id="survey_user_input_line_rule_recruitment_interviewer" model="ir.rule">
            <field name="name">Survey user input line: recruitment interviewer: read survey answers for which they are set as interviewer</field>
            <field name="model_id" ref="survey.model_survey_user_input_line"/>
            <field name="domain_force">[
                '|',
                    ('user_input_id.applicant_id.interviewer_ids', 'in', user.id),
                    ('user_input_id.applicant_id.job_id.interviewer_ids', 'in', user.id),
                ]</field>
            <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>
        <record id="survey_user_input_rule_recruitment_interviewer" model="ir.rule">
            <field name="name">Survey user input: recruitment interviewer: read survey answers for which they are set as interviewer</field>
            <field name="model_id" ref="survey.model_survey_user_input"/>
            <field name="domain_force">[
                '|',
                    ('applicant_id.interviewer_ids', 'in', user.id),
                    ('applicant_id.job_id.interviewer_ids', 'in', user.id),
                ]</field>
            <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>
        <record id="survey_survey_recruitment_interviewer" model="ir.rule">
            <field name="name">Survey: recruitment interviewer: send surveys to applicants for which they are set as interviewer</field>
            <field name="model_id" ref="survey.model_survey_survey"/>
            <field name="domain_force">[('survey_type', '=', 'recruitment'),
                '|', ('hr_job_ids.interviewer_ids', 'in', user.id),
                     ('hr_job_ids.application_ids.interviewer_ids', 'in', user.id)
                ]</field>
            <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>

        <record id="survey_question_recruitment_interviewer" model="ir.rule">
            <field name="name">Survey: recruitment interviewer: send surveys to applicants for which they are set as interviewer</field>
            <field name="model_id" ref="survey.model_survey_question"/>
            <field name="domain_force">[('survey_id.survey_type', '=', 'recruitment'),
                '|', ('survey_id.hr_job_ids.interviewer_ids', 'in', user.id),
                     ('survey_id.hr_job_ids.application_ids.interviewer_ids', 'in', user.id)
                ]</field>
            <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>

        <record id="survey_invite_recruitment_interviewer" model="ir.rule">
            <field name="name">Survey invite: recruitment interviewer: send surveys to applicants for which they are set as interviewer</field>
            <field name="model_id" ref="survey.model_survey_invite"/>
            <field name="domain_force">[('survey_id.survey_type', '=', 'recruitment'),
                '|', ('survey_id.hr_job_ids.interviewer_ids', 'in', user.id),
                     ('survey_id.hr_job_ids.application_ids.interviewer_ids', 'in', user.id)
                ]</field>
            <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>
    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_survey_user_input_recruitment_manager,survey.user_input.recruitment.manager,survey.model_survey_user_input,hr_recruitment.group_hr_recruitment_manager,1,1,1,1
access_survey_user_input_line_recruitment_manager,survey.user.input.line.recruitment.manager,survey.model_survey_user_input_line,hr_recruitment.group_hr_recruitment_manager,1,1,1,1
access_survey_survey_recruitment_manager,survey.survey.recruitment.manager,survey.model_survey_survey,hr_recruitment.group_hr_recruitment_manager,1,1,1,1
access_survey_question_recruitment_manager,survey.question.recruitment.manager,survey.model_survey_question,hr_recruitment.group_hr_recruitment_manager,1,1,1,1
access_survey_question_answer_recruitment_manager,survey.question.answer.recruitment.manager,survey.model_survey_question_answer,hr_recruitment.group_hr_recruitment_manager,1,1,1,1

access_survey_user_input_recruitment_user,survey.user_input.recruitment.user,survey.model_survey_user_input,hr_recruitment.group_hr_recruitment_user,1,0,0,0
access_survey_user_input_line_recruitment_user,survey.user_input.line.recruitment.user,survey.model_survey_user_input_line,hr_recruitment.group_hr_recruitment_user,1,0,0,0
access_survey_invite_recruitment_user,survey.invite.recruitment.user,survey.model_survey_invite,hr_recruitment.group_hr_recruitment_user,1,1,1,0

access_survey_user_input_recruitment_interviewer,survey.user.input.recruitment.interviewer,survey.model_survey_user_input,hr_recruitment.group_hr_recruitment_interviewer,1,0,0,0
access_survey_user_input_line_recruitment_interviewer,survey.user_input.line.recruitment.interviewer,survey.model_survey_user_input_line,hr_recruitment.group_hr_recruitment_interviewer,1,0,0,0
access_survey_survey_recruitment_interviewer,survey.survey.recruitment.interviewer,survey.model_survey_survey,hr_recruitment.group_hr_recruitment_interviewer,1,0,0,0
access_survey_question_recruitment_interviewer,survey.question.recruitment.interviewer,survey.model_survey_question,hr_recruitment.group_hr_recruitment_interviewer,1,0,0,0
access_survey_invite_recruitment_interviewer,survey.invite.recruitment.interviewer,survey.model_survey_invite,hr_recruitment.group_hr_recruitment_interviewer,1,1,1,0

```

## File: views\hr_applicant_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="crm_case_tree_view_job_inherit" model="ir.ui.view">
        <field name="name">hr.applicant.list.inherit</field>
        <field name="model">hr.applicant</field>
        <field name="inherit_id" ref="hr_recruitment.crm_case_tree_view_job"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='source_id']" position="after">
                <field name="survey_id" column_invisible="True"/>
                <field name="response_ids" column_invisible="True"/>
            </xpath>
        </field>
    </record>

    <record id="hr_applicant_view_form_inherit" model="ir.ui.view">
        <field name="name">hr.applicant.form.inherit</field>
        <field name="model">hr.applicant</field>
        <field name="inherit_id" ref="hr_recruitment.hr_applicant_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='archive_applicant']" position="before">
                <button name="action_send_survey" string="Send Interview" type="object" invisible="not active or not survey_id"/>
            </xpath>
            <xpath expr="//button[@name='action_create_meeting']" position="after">
                <button name="action_print_survey"
                    class="oe_stat_button"
                    icon="fa-pencil-square-o"
                    type="object"
                    help="See interview report"
                    invisible="not survey_id or not response_ids">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_text">Consult</span>
                        <span class="o_stat_text">Interview</span>
                    </div>
                </button>
            </xpath>
            <xpath expr="//field[@name='job_id']" position="before">
                <field name="survey_id" invisible="1"/>
                <field name="response_ids" invisible="1"/>
            </xpath>
        </field>
    </record>

    <record id="hr_kanban_view_applicant_inherit" model="ir.ui.view">
        <field name="name">hr.applicants.kanban.inherit</field>
        <field name="model">hr.applicant</field>
        <field name="inherit_id" ref="hr_recruitment.hr_kanban_view_applicant"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='user_id']" position="before">
                <field name="survey_id"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\hr_job_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_job_survey_inherit" model="ir.ui.view">
        <field name="name">hr.job.form.inherit</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr_recruitment.hr_job_survey"/>
        <field name="arch" type="xml">
            <field name="interviewer_ids" position="after">
                <label for="survey_id" groups="hr_recruitment.group_hr_recruitment_interviewer"/>
                <div groups="hr_recruitment.group_hr_recruitment_interviewer">
                    <field name="survey_id"
                        context="{'default_access_mode': 'token'}"
                        domain="[('survey_type', '=', 'recruitment')]"/>
                    <div class="o_link_trackers col-6 text-end">
                        <a type="object" name="action_test_survey">
                        </a>
                    </div>
                </div>
            </field>
        </field>
    </record>
    <record id="view_hr_job_kanban_inherit" model="ir.ui.view">
        <field name="name">hr.job.kanban.inherit</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr_recruitment.view_hr_job_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='active']" position="after">
                <field name="survey_id"/>
            </xpath>
            <xpath expr="//div[@name='menu_view_applications']" position="after">
                <div role="menuitem" t-if="record.survey_id.raw_value">
                    <a name="action_test_survey" type="object" title="Display Interview Form">Interviews</a>
                </div>
            </xpath>
            <xpath expr="//div[@name='menu_new_applications']" position="after">
                <div role="menuitem" t-if="!record.survey_id.raw_value">
                    <a name="action_new_survey" type="object" title="Display Interview Form">Interview Form</a>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_config_setting_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.hr.recruitment.survey</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="hr_recruitment.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <setting id="interview_forms_setting" position="inside">
                <div class="content-group">
                    <div class="mt8">
                        <button name="%(survey.action_survey_form)d" icon="oi-arrow-right" type="action" string="Interview Survey" class="btn-link"/>
                    </div>
                </div>
            </setting>
        </field>
    </record>
</odoo> 

```

## File: views\survey_survey_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="survey_survey_view_form" model="ir.ui.view">
        <field name="name">survey.survey.view.form.inherit.recruitment</field>
        <field name="model">survey.survey</field>
        <field name="inherit_id" ref="survey.survey_survey_view_form"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='survey_type']" position="attributes">
                <attribute name="invisible">True</attribute>
            </xpath>
            <button name="action_send_survey" position="attributes">
                <attribute name="invisible">True</attribute>
            </button>
            <button name="action_start_session" position="attributes">
                <attribute name="invisible">True</attribute>
            </button>
            <button name="action_archive" position="attributes">
                <attribute name="invisible">True</attribute>
            </button>
            <xpath expr="//group[@name='scoring']" position="attributes">
                <attribute name="invisible">True</attribute>
            </xpath>
            <xpath expr="//label[@for='is_time_limited']" position="attributes">
                <attribute name="invisible">True</attribute>
            </xpath>
            <xpath expr="//div[@name='is_time_limited']" position="attributes">
                <attribute name="invisible">True</attribute>
            </xpath>
            <xpath expr="//field[@name='scoring_type']" position="attributes">
                <attribute name="invisible">True</attribute>
            </xpath>
            <xpath expr="//label[@for='certification']" position="attributes">
                <attribute name="invisible">True</attribute>
            </xpath>
            <xpath expr="//div[@name='certification']" position="attributes">
                <attribute name="invisible">True</attribute>
            </xpath>
            <xpath expr="//group[@name='live_session']" position="attributes">
                <attribute name="invisible">True</attribute>
            </xpath>
        </field>
    </record>

    <record id="survey_survey_view_kanban" model="ir.ui.view">
        <field name="name">survey.survey.view.kanban.inherit.recruitment</field>
        <field name="model">survey.survey</field>
        <field name="inherit_id" ref="survey.survey_survey_view_kanban"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <button name="action_send_survey" position="attributes">
                <attribute name="invisible">True</attribute>
            </button>
            <button name="action_start_session" position="attributes">
                <attribute name="invisible">True</attribute>
            </button>
        </field>
    </record>

    <record id="survey_survey_action_recruitment" model="ir.actions.act_window">
        <field name="name">Interviews</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">survey.survey</field>
        <field name="domain">[('survey_type', '=', 'recruitment')]</field>
        <field name="view_ids" eval="[(5, 0, 0),
            (0, 0, {'view_mode': 'kanban', 'view_id': ref('hr_recruitment_survey.survey_survey_view_kanban')}),
            (0, 0, {'view_mode': 'form', 'view_id': ref('hr_recruitment_survey.survey_survey_view_form')}),
        ]"/>
        <field name="view_mode">kanban,list,activity,form</field>
        <field name="context">{
                'default_survey_type': 'recruitment',
            }
        </field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new survey
            </p><p>
                You can create surveys used for recruitments. Design easily your interview,
                send invitations and analyze answers.
            </p>
        </field>
    </record>

    <menuitem
        id="menu_hr_recruitment_config_surveys"
        name="Interviews"
        action="survey_survey_action_recruitment"
        parent="hr_recruitment.menu_hr_recruitment_configuration"
        sequence="50"
        groups="hr_recruitment.group_hr_recruitment_manager"/>
</odoo>

```

## File: views\survey_templates_statistics.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
    <template id="hr_recruitment_survey_button_form_view" inherit_id="survey.survey_button_form_view">
        <xpath expr="//div[hasclass('survey_button_form_view_hook')]" position="inside">
            <a t-if="(env.user.has_group('hr_recruitment.group_hr_recruitment_manager') and survey.survey_type == 'recruitment')"
                t-attf-href="/odoo/action-hr_recruitment_survey.survey_survey_action_recruitment/{{survey.id}}"
                class="ms-2">
                <i class="oi oi-fw oi-arrow-right"/>Go to Recruitment
            </a>
        </xpath>
    </template>
</data>
</odoo>

```

## File: wizard\survey_invite.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from markupsafe import Markup

from odoo import fields, models, _
from odoo.tools.misc import clean_context


class SurveyInvite(models.TransientModel):
    _inherit = "survey.invite"

    applicant_id = fields.Many2one('hr.applicant', string='Applicant')

    def _get_done_partners_emails(self, existing_answers):
        partners_done, emails_done, answers = super()._get_done_partners_emails(existing_answers)
        if self.applicant_id.response_ids.filtered(lambda res: res.survey_id.id == self.survey_id.id):
            if existing_answers and self.existing_mode == 'resend':
                partners_done |= self.applicant_id.partner_id
        return partners_done, emails_done, answers

    def _send_mail(self, answer):
        mail = super()._send_mail(answer)
        if answer.applicant_id:
            answer.applicant_id.message_post(body=Markup(mail.body_html))
            mail.send()
        return mail

    def action_invite(self):
        self.ensure_one()
        if self.applicant_id:
            survey = self.survey_id.with_context(clean_context(self.env.context))

            if not self.applicant_id.response_ids.filtered(lambda res: res.survey_id.id == self.survey_id.id):
                self.applicant_id.sudo().write({
                    'response_ids': (self.applicant_id.response_ids | survey.sudo()._create_answer(partner=self.applicant_id.partner_id,
                        **self._get_answers_values())).ids
                })

            partner = self.applicant_id.partner_id
            survey_link = survey._get_html_link(title=survey.title)
            partner_link = partner._get_html_link()
            content = _('The survey %(survey_link)s has been sent to %(partner_link)s',
                survey_link=survey_link,
                partner_link=partner_link,
            )
            body = Markup('<p>%s</p>') % content
            self.applicant_id.message_post(body=body)
        return super().action_invite()

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-

from . import survey_invite

```

