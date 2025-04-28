# Odoo Module: hr_recruitment_survey

Category: Human Resources

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models

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
        'security/hr_recruitment_survey_security.xml',
        'views/hr_job_views.xml',
        'views/hr_applicant_views.xml',
        'views/res_config_setting_views.xml',
    ],
    'demo': [
        'data/survey_demo.xml',
        'data/hr_job_demo.xml',
    ],
    'license': 'LGPL-3',
}

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

## File: data\survey_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <record id="survey_recruitment_form" model="survey.survey">
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
        <field name="title">Which country are you from ?</field>
        <field name="question_type">char_box</field>
    </record>
    <record id="survey_recruitment_form_p1_q2" model="survey.question">
        <field name="survey_id" ref="survey_recruitment_form"/>
        <field name="sequence">3</field>
        <field name="title">From which university did or will you graduate ?</field>
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
<p>What are your main knowledge regarding the job you are applying to ?</p></field>
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
        <field name="title">What is important for you ?</field>
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

from odoo import fields, models, _
from odoo.exceptions import UserError


class Applicant(models.Model):
    _inherit = "hr.applicant"

    survey_id = fields.Many2one('survey.survey', related='job_id.survey_id', string="Survey", readonly=True)
    response_id = fields.Many2one('survey.user_input', "Response", ondelete="set null", copy=False)
    response_state = fields.Selection(related='response_id.state', readonly=True)

    def action_print_survey(self):
        """ If response is available then print this response otherwise print survey form (print template of the survey) """
        self.ensure_one()
        return self.survey_id.action_print_survey(answer=self.response_id)

    def action_send_survey(self):
        self.ensure_one()

        # if an applicant does not already has associated partner_id create it
        if not self.partner_id:
            if not self.partner_name:
                raise UserError(_('You must define a Contact Name for this applicant.'))
            self.partner_id = self.env['res.partner'].create({
                'is_company': False,
                'type': 'private',
                'name': self.partner_name,
                'email': self.email_from,
                'phone': self.partner_phone,
                'mobile': self.partner_mobile
            })
        return self.survey_id.with_context(default_applicant_id=self.id, default_partner_ids=self.partner_id.ids).action_send_survey()

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
            'title': _("Interview Form : %s") % self.name,
        })
        self.write({'survey_id': survey.id})

        action = {
                'name': _('Survey'),
                'view_mode': 'form,tree',
                'res_model': 'survey.survey',
                'type': 'ir.actions.act_window',
                'context': {'form_view_initial_mode': 'edit'},
                'res_id': survey.id,
            }

        return action

```

## File: models\survey_invite.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools.misc import clean_context


class SurveyInvite(models.TransientModel):
    _inherit = "survey.invite"

    applicant_id = fields.Many2one('hr.applicant', string='Applicant')

    def action_invite(self):
        self.ensure_one()
        if self.applicant_id:
            survey = self.survey_id.with_context(clean_context(self.env.context))

            if not self.applicant_id.response_id:
                self.applicant_id.write({
                    'response_id': survey._create_answer(partner=self.applicant_id.partner_id).id
                })

            partner = self.applicant_id.partner_id
            survey_link = survey._get_html_link(title=survey.title)
            partner_link = partner._get_html_link()
            content = _('The survey %(survey_link)s has been sent to %(partner_link)s', survey_link=survey_link, partner_link=partner_link)
            body = '<p>%s</p>' % content
            self.applicant_id.message_post(body=body)
        return super().action_invite()


class SurveyUserInput(models.Model):
    _inherit = "survey.user_input"

    applicant_id = fields.One2many('hr.applicant', 'response_id', string='Applicant')

    def _mark_done(self):
        odoobot = self.env.ref('base.partner_root')
        for user_input in self:
            if user_input.applicant_id:
                body = _('The applicant "%s" has finished the survey.', user_input.applicant_id.partner_name)
                user_input.applicant_id.message_post(body=body, author_id=odoobot.id)
        return super()._mark_done()

    @api.model_create_multi
    def create(self, values_list):
        if 'default_applicant_id' in self.env.context:
            self = self.with_context(default_applicant_id=False)
        return super().create(values_list)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_job
from . import hr_applicant
from . import survey_invite

```

## File: security\hr_recruitment_survey_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="hr_recruitment.group_hr_recruitment_user" model="res.groups">
            <field name="implied_ids" eval="[(4, ref('survey.group_survey_user'))]"/>
        </record>
    </data>
</odoo>

```

## File: views\hr_applicant_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="crm_case_tree_view_job_inherit" model="ir.ui.view">
        <field name="name">hr.applicant.tree.inherit</field>
        <field name="model">hr.applicant</field>
        <field name="inherit_id" ref="hr_recruitment.crm_case_tree_view_job"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='source_id']" position="after">
                <field name="survey_id" invisible="1"/>
                <field name="response_id" invisible="1"/>
            </xpath>
        </field>
    </record>

    <record id="hr_applicant_view_form_inherit" model="ir.ui.view">
        <field name="name">hr.applicant.form.inherit</field>
        <field name="model">hr.applicant</field>
        <field name="inherit_id" ref="hr_recruitment.hr_applicant_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='archive_applicant']" position="before">
                <button name="action_send_survey" string="SEND INTERVIEW" type="object" attrs="{'invisible': ['|', ('active', '=', False), ('survey_id', '=', False)]}"/>
            </xpath>
            <xpath expr="//button[@name='action_makeMeeting']" position="after">
                <button name="action_print_survey"
                    class="oe_stat_button"
                    icon="fa-pencil-square-o"
                    type="object"
                    help="See interview report"
                    attrs="{'invisible':['|',('survey_id','=',False),('response_id','=',False)]}">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_text">Consult</span>
                        <span class="o_stat_text">Interview</span>
                    </div>
                </button>
            </xpath>
            <xpath expr="//field[@name='job_id']" position="before">
                <field name="survey_id" invisible="1"/>
                <field name="response_id" invisible="1"/>
                <field name="response_state" invisible="1"/>
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
                <label for="survey_id" groups="survey.group_survey_user"/>
                <div groups="survey.group_survey_user">
                    <field name="survey_id"
                        context="{'default_access_mode': 'token'}"/>
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
            <xpath expr="//field[@name='manager_id']" position="after">
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
            <div id="interview_forms" position="replace">
                <div class="content-group">
                    <div class="mt8">
                        <button name="%(survey.action_survey_form)d" icon="fa-arrow-right" type="action" string="Interview Survey" class="btn-link"/>
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo> 

```

