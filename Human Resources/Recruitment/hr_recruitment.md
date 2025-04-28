# Odoo Module: hr_recruitment

Category: Human Resources/Recruitment

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Recruitment',
    'version': '1.0',
    'category': 'Human Resources/Recruitment',
    'sequence': 90,
    'summary': 'Track your recruitment pipeline',
    'description': "",
    'website': 'https://www.odoo.com/page/recruitment',
    'depends': [
        'hr',
        'calendar',
        'fetchmail',
        'utm',
        'attachment_indexation',
        'web_tour',
        'digest',
    ],
    'data': [
        'security/hr_recruitment_security.xml',
        'security/ir.model.access.csv',
        'data/hr_recruitment_data.xml',
        'data/digest_data.xml',
        'data/hr_recruitment_templates.xml',
        'views/hr_recruitment_views.xml',
        'views/res_config_settings_views.xml',
        'views/hr_recruitment_templates.xml',
        'views/hr_department_views.xml',
        'views/hr_job_views.xml',
        'views/mail_activity_views.xml',
        'views/digest_views.xml',
        'wizard/applicant_refuse_reason_views.xml',
    ],
    'demo': [
        'data/hr_recruitment_demo.xml',
    ],
    'installable': True,
    'auto_install': False,
    'application': True,
    'license': 'LGPL-3',
}

```

## File: data\digest_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data noupdate="1">
        <record id="digest.digest_digest_default" model="digest.digest">
            <field name="kpi_hr_recruitment_new_colleagues">True</field>
        </record>
    </data>

    <data>
        <record id="digest_tip_hr_recruitment_0" model="digest.tip">
            <field name="name">Tip: Let candidates apply by email</field>
            <field name="sequence">1300</field>
            <field name="group_id" ref="hr_recruitment.group_hr_recruitment_manager" />
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Let candidates apply by email</p>
    <p class="tip_content">
        By setting an alias to a job position, emails sent to this address create applications automatically. You can even use multiple trackers to get statistics according to the source of the application: LinkedIn, Monster, Indeed, etc.
        % set record = object.env['hr.job'].search([('alias_name', '!=', False)], limit=1)
        % if record and record.alias_domain
        <a href="mailto:${record.alias_id.display_name}" target="_blank" style="color: #875a7b; text-decoration: none;">Try sending an email</a>
        % endif
    </p>
</div>
            </field>
        </record>
    </data>
</odoo>

```

## File: data\hr_recruitment_data.xml

```xml
<?xml version="1.0"?>
<odoo>
<data noupdate="1">

    <!-- Meeting Types (for interview meetings) -->
    <record model="calendar.event.type" id="categ_meet_interview">
        <field name="name">Interview</field>
    </record>

    <!-- Templates for interest / refusing applicants -->
    <record id="email_template_data_applicant_refuse" model="mail.template">
        <field name="name">Applicant: Refuse</field>
        <field name="model_id" ref="hr_recruitment.model_hr_applicant"/>
        <field name="subject">Your Job Application: ${object.job_id.name | safe}</field>
        <field name="email_to">${(not object.partner_id and object.email_from or '') | safe}</field>
        <field name="partner_to">${object.partner_id.id or ''}</field>
        <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
    <tr>
        <td valign="top">
            <div style="font-size: 13px; margin: 0px; padding: 0px;">
                Hello,<br/><br/>
                Thank you for your interest in joining the
                <b>${object.company_id.name}</b> team.  We wanted to
                let you know that, although your resume is competitive,
                our hiring team reviewed your application and <b>did not
                select it for further consideration</b>.
                <br/><br/>
                Please note that recruiting is hard, and we can make
                mistakes. Do not hesitate to reply to this email if you
                think we made a mistake, or if you want more information
                about our decision.
                <br/><br/>
                We will, however, keep your resume on record and get in
                touch with you about future opportunities that may be a
                better fit for your skills and experience.
                <br/><br/>
                We wish you all the best in your job search and hope we
                will have the chance to consider you for another role
                in the future.
                <br/><br/>
                Thank you,
                <div style="font-size: 11px; color: grey;">
                    % if object.user_id:
                        -- <br/>
                        <strong>${object.user_id.name}</strong><br/>
                        Email: ${object.user_id.email or ''}<br/>
                        Phone: ${object.user_id.phone or ''}
                    % else:
                        -- <br/>
                        ${object.company_id.name}<br/>
                        The HR Team
                    % endif
                </div>
            </div>
        </td>
    </tr>
</table>
        </field>
        <field name="auto_delete" eval="True"/>
        <field name="lang">${object.partner_id.lang or ''}</field>
    </record>

    <record id="email_template_data_applicant_interest" model="mail.template">
        <field name="name">Applicant: Interest</field>
        <field name="model_id" ref="hr_recruitment.model_hr_applicant"/>
        <field name="subject">Your Job Application: ${object.job_id.name | safe}</field>
        <field name="email_to">${(not object.partner_id and object.email_from or '') | safe}</field>
        <field name="partner_to">${object.partner_id.id or ''}</field>
        <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="background-color: white; border-collapse: collapse; margin-left: 20px;">
    <tr>
        <td valign="top" style="padding: 0px 10px;">
            <div style="text-align: center">
                <h2>Congratulations!</h2>
                <div style="color:grey;">Your resume has been positively reviewed.</div>
                <img src="/hr_recruitment/static/src/img/congratulations.png" alt="Congratulations!" style="width:175px;margin:20px 0;"/>
            </div>
            <div style="font-size: 13px; margin: 0px; padding: 0px;">
                We just reviewed your resume, and it caught our
                attention. As we think you might be great for the
                position, your application has been short listed for a
                call or an interview.
                <br/><br/>
                % if 'website_url' in object.job_id and object.job_id.website_url:
                    <div style="margin: 16px 8px 16px 8px;">
                        <a href="${object.job_id.website_url}"
                            style="background-color: #875a7b; text-decoration: none; color: #fff; padding: 8px 16px 8px 16px; border-radius: 5px;">Job Description</a>
                    </div>
                % endif

                % if object.user_id:
                    You will soon be contacted by:
                    <table>
                        <tr>
                            <td width="75">
                                <img src="/web/image/res.users/${object.user_id.id}/image_128" alt="Avatar" style="vertical-align:baseline; width: 64px; height: 64px; object-fit: cover;" />
                            </td>
                            <td>
                                <strong>${object.user_id.name}</strong><br/>
                                <span>Email: ${object.user_id.email or ''}</span><br/>
                                <span>Phone: ${object.user_id.phone or ''}</span>
                            </td>
                        </tr>
                    </table>
                    <br/><br/>
                % endif
                See you soon,
                <div style="font-size: 11px; color: grey;">
                    -- <br/>
                    The HR Team
                    % if 'website_url' in object.job_id and object.job_id.website_url
                        Discover <a href="/jobs" style="text-decoration:none;color:#717188;">all our jobs</a>.<br/>
                    % endif
                </div>

                <hr width="97%" style="background-color: rgb(204,204,204); border: medium none; clear: both; display: block; font-size: 0px; min-height: 1px; line-height: 0; margin: 16px 0px 16px 0px;"/>
                <h3 style="color:#9A6C8E;"><strong>What is the next step?</strong></h3>
                We usually <strong>answer applications within a few days</strong>.
                <br/><br/>
                The next step is either a call or a meeting in our offices.
                <br/>
                Feel free to <strong>contact us if you want a faster
                feedback</strong> or if you don't get news from us
                quickly enough (just reply to this email).
                <br/>

                <hr width="97%" style="background-color: rgb(204,204,204); border: medium none; clear: both; display: block; font-size: 0px; min-height: 1px; line-height: 0; margin: 17px 0px 16px 0px;"/>
                % set location = ''
                % if object.job_id.address_id.name:
                    <strong>${object.job_id.address_id.name}</strong><br/>
                % endif
                % if object.job_id.address_id.street:
                    ${object.job_id.address_id.street}<br/>
                    % set location = object.job_id.address_id.street
                % endif
                % if object.job_id.address_id.street2:
                    ${object.job_id.address_id.street2}<br/>
                    % set location = '%s, %s' % (location, object.job_id.address_id.street2)
                % endif
                % if object.job_id.address_id.city:
                    ${object.job_id.address_id.city},
                    % set location = '%s, %s' % (location, object.job_id.address_id.city)
                % endif
                % if object.job_id.address_id.state_id.name:
                    ${object.job_id.address_id.state_id.name},
                    % set location = '%s, %s' % (location, object.job_id.address_id.state_id.name)
                % endif
                % if object.job_id.address_id.zip:
                    ${object.job_id.address_id.zip}
                    % set location = '%s, %s' % (location, object.job_id.address_id.zip)
                % endif
                <br/>
                % if object.job_id.address_id.country_id.name:
                    ${object.job_id.address_id.country_id.name}<br/>
                    % set location = '%s, %s' % (location, object.job_id.address_id.country_id.name)
                % endif
                <br/>
            </div>
        </td>
    </tr>
</table></field>
        <field name="auto_delete" eval="True"/>
        <field name="lang">${object.partner_id.lang or ''}</field>
    </record>

    <record id="email_template_data_applicant_congratulations" model="mail.template">
        <field name="name">Applicant: Acknowledgement</field>
        <field name="model_id" ref="hr_recruitment.model_hr_applicant"/>
        <field name="subject">Your Job Application: ${object.job_id.name | safe}</field>
        <field name="email_to">${(not object.partner_id and object.email_from or '') | safe}</field>
        <field name="partner_to">${object.partner_id.id or ''}</field>
        <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="background-color: white; border-collapse: collapse; margin-left: 20px;">
    <tr>
        <td valign="top" style="padding: 0px 10px;">
            <div style="font-size: 13px; margin: 0px; padding: 0px;">
                Hello,
                <br/><br/>
                We confirm we successfully received your application for the job
                "<a href="${object.job_id.website_url or ''}" style="color:#9A6C8E;"><strong>${object.job_id.name}</strong></a>" at <strong>${object.company_id.name}</strong>.
                <br/><br/>
                We will come back to you shortly.

                % if 'website_url' in object.job_id and object.job_id.website_url:
                    <div style="margin: 16px 8px 16px 8px;">
                        <a href="${object.job_id.website_url}"
                            style="background-color: #875a7b; text-decoration: none; color: #fff; padding: 8px 16px 8px 16px; border-radius: 5px;">Job Description</a>
                    </div>
                % endif

                <hr width="97%" style="background-color: rgb(204,204,204); border: medium none; clear: both; display: block; font-size: 0px; min-height: 1px; line-height: 0; margin: 16px 0px 16px 0px;"/>
                % if object.user_id:
                    <h3 style="color:#9A6C8E;"><strong>Your Contact:</strong></h3>
                    <table>
                        <tr>
                            <td width="75">
                                <img src="/web/image/res.users/${object.user_id.id}/image_128" alt="Avatar" style="vertical-align:baseline; width: 64px; height: 64px; object-fit: cover;" />
                            </td>
                            <td>
                                <strong>${object.user_id.name}</strong><br/>
                                <span>Email: ${object.user_id.email or ''}</span><br/>
                                <span>Phone: ${object.user_id.phone or ''}</span>
                            </td>
                        </tr>
                    </table>
                    <hr width="97%" style="background-color: rgb(204,204,204); border: medium none; clear: both; display: block; font-size: 0px; min-height: 1px; line-height: 0; margin: 16px 0px 16px 0px;"/>
                % endif

                <h3 style="color:#9A6C8E;"><strong>What is the next step?</strong></h3>
                We usually <strong>answer applications within a few days.</strong><br/><br/>
                Feel free to <strong>contact us if you want a faster
                feedback</strong> or if you don't get news from us
                quickly enough (just reply to this email).

                <hr width="97%" style="background-color: rgb(204,204,204); border: medium none; clear: both; display: block; font-size: 0px; min-height: 1px; line-height: 0; margin: 17px 0px 16px 0px;"/>
                % set location = ''
                % if object.job_id.address_id.name:
                    <strong>${object.job_id.address_id.name}</strong><br/>
                % endif
                % if object.job_id.address_id.street:
                    ${object.job_id.address_id.street}<br/>
                    % set location = object.job_id.address_id.street
                % endif
                % if object.job_id.address_id.street2:
                    ${object.job_id.address_id.street2}<br/>
                    % set location = '%s, %s' % (location, object.job_id.address_id.street2)
                % endif
                % if object.job_id.address_id.city:
                    ${object.job_id.address_id.city},
                    % set location = '%s, %s' % (location, object.job_id.address_id.city)
                % endif
                % if object.job_id.address_id.state_id.name:
                    ${object.job_id.address_id.state_id.name},
                    % set location = '%s, %s' % (location, object.job_id.address_id.state_id.name)
                % endif
                % if object.job_id.address_id.zip:
                    ${object.job_id.address_id.zip}
                    % set location = '%s, %s' % (location, object.job_id.address_id.zip)
                % endif
                <br/>
                % if object.job_id.address_id.country_id.name:
                    ${object.job_id.address_id.country_id.name}<br/>
                    % set location = '%s, %s' % (location, object.job_id.address_id.country_id.name)
                % endif
                <br/>
            </div>
        </td>
    </tr>
</table></field>
        <field name="auto_delete" eval="True"/>
        <field name="lang">${object.partner_id.lang or ''}</field>
    </record>

    <record model="hr.recruitment.degree" id="degree_graduate">
        <field name="name">Graduate</field>
        <field name="sequence">1</field>
    </record>
    <record model="hr.recruitment.degree" id="degree_bachelor">
        <field name="name">Bachelor Degree</field>
        <field name="sequence">2</field>
    </record>
    <record model="hr.recruitment.degree" id="degree_licenced">
        <field name="name">Master Degree</field>
        <field name="sequence">3</field>
    </record>
    <record model="hr.recruitment.degree" id="degree_bac5">
        <field name="name">Doctoral Degree</field>
        <field name="sequence">4</field>
    </record>

    <record id="mail_alias_jobs" model="mail.alias">
        <field name="alias_name">jobs</field>
        <field name="alias_model_id" ref="model_hr_applicant"/>
        <field name="alias_user_id" ref="base.user_admin"/>
        <field name="alias_parent_model_id" ref="model_hr_job"/>
    </record>

    <!-- Applicant-related subtypes for messaging / Chatter -->
    <record id="mt_applicant_new" model="mail.message.subtype">
        <field name="name">New Applicant</field>
        <field name="res_model">hr.applicant</field>
        <field name="default" eval="False"/>
        <field name="hidden" eval="True"/>
        <field name="description">Applicant created</field>
    </record>
    <record id="mt_applicant_stage_changed" model="mail.message.subtype">
        <field name="name">Stage Changed</field>
        <field name="res_model">hr.applicant</field>
        <field name="default" eval="False"/>
        <field name="description">Stage changed</field>
    </record>
    <record id="mt_applicant_hired" model="mail.message.subtype">
        <field name="name">Applicant Hired</field>
        <field name="res_model">hr.applicant</field>
        <field name="default" eval="True"/>
        <field name="description">Applicant hired</field>
    </record>

    <!-- Job-related subtypes for messaging / Chatter -->
    <record id="mt_job_new" model="mail.message.subtype">
        <field name="name">Job Position created</field>
        <field name="res_model">hr.job</field>
        <field name="default" eval="False"/>
        <field name="hidden" eval="True"/>
    </record>
    <record id="mt_job_applicant_stage_changed" model="mail.message.subtype">
        <field name="name">Applicant Stage Changed</field>
        <field name="res_model">hr.job</field>
        <field name="default" eval="False"/>
        <field name="parent_id" ref="mt_applicant_stage_changed"/>
        <field name="relation_field">job_id</field>
    </record>
    <record id="mt_job_applicant_hired" model="mail.message.subtype">
        <field name="name">Applicant Hired</field>
        <field name="res_model">hr.job</field>
        <field name="default" eval="True"/>
        <field name="parent_id" ref="mt_applicant_hired"/>
        <field name="relation_field">job_id</field>
    </record>

    <!-- Department-related (parent) subtypes for messaging / Chatter -->
    <record id="mt_department_new" model="mail.message.subtype">
        <field name="name">Job Position Created</field>
        <field name="res_model">hr.department</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="mt_job_new"/>
        <field name="relation_field">department_id</field>
    </record>

    <!-- Applicant Categories(Tag) -->
    <record id="tag_applicant_reserve" model="hr.applicant.category">
        <field name="name">Reserve</field>
    </record>
    <record id="tag_applicant_manager" model="hr.applicant.category">
        <field name="name">Manager</field>
    </record>
    <record id="tag_applicant_it" model="hr.applicant.category">
        <field name="name">IT</field>
    </record>
    <record id="tag_applicant_sales" model="hr.applicant.category">
        <field name="name">Sales</field>
    </record>
    <record model="utm.campaign" id="utm_campaign_job">
            <field name="name">Job Campaign</field>
    </record>

    <record model="hr.recruitment.stage" id="stage_job1">
        <field name="name">Initial Qualification</field>
        <field name="sequence">1</field>
    </record>
    <record model="hr.recruitment.stage" id="stage_job2">
        <field name="name">First Interview</field>
        <field name="sequence">2</field>
        <field name="template_id" ref="email_template_data_applicant_congratulations"/> 
    </record>
    <record model="hr.recruitment.stage" id="stage_job3">
        <field name="name">Second Interview</field>
        <field name="sequence">3</field>
    </record>
    <record model="hr.recruitment.stage" id="stage_job4">
        <field name="name">Contract Proposal</field>
        <field name="sequence">4</field>
    </record>
    <record model="hr.recruitment.stage" id="stage_job5">
        <field name="name">Contract Signed</field>
        <field name="sequence">5</field>
        <field name="fold" eval="True"/>
    </record>

    <!-- applicant refuse reason -->
        <record id="refuse_reason_1" model="hr.applicant.refuse.reason">
            <field name="name">Doesn't fit the job requirements</field>
        </record>
        <record id="refuse_reason_2" model="hr.applicant.refuse.reason">
            <field name="name">The applicant is not interested anymore</field>
        </record>
        <record id="refuse_reason_3" model="hr.applicant.refuse.reason">
            <field name="name">The applicant gets a better offer</field>
        </record>

</data>
</odoo>

```

## File: data\hr_recruitment_demo.xml

```xml
<?xml version="1.0"?>
<odoo>
  <data noupdate="1">

    <record id="base.user_demo" model="res.users">
        <field name="groups_id" eval="[(4, ref('hr_recruitment.group_hr_recruitment_user'))]"/>
    </record>

    <!--Manage the job_id to get in hr.applicant-->
    <record id="hr.job_developer" model="hr.job">
        <field name="state">recruit</field>
        <field name="no_of_recruitment">4</field>
        <field name="no_of_hired_employee">56</field>
    </record>
    <record id="hr.job_ceo" model="hr.job">
        <field name="state">open</field>
        <field name="no_of_hired_employee">1</field>
    </record>
    <record id="hr.job_cto" model="hr.job">
        <field name="state">open</field>
        <field name="no_of_hired_employee">1</field>
    </record>
    <record id="hr.job_consultant" model="hr.job">
        <field name="state">recruit</field>
        <field name="no_of_recruitment">1</field>
        <field name="no_of_hired_employee">17</field>
    </record>
    <record id="hr.job_hrm" model="hr.job">
        <field name="no_of_recruitment">1</field>
        <field name="state">recruit</field>
        <field name="no_of_hired_employee">5</field>
    </record>
    <record id="hr.job_marketing" model="hr.job">
        <field name="state">recruit</field>
        <field name="no_of_recruitment">3</field>
        <field name="no_of_hired_employee">2</field>
    </record>
    <record id="hr.job_trainee" model="hr.job">
        <field name="state">recruit</field>
        <field name="no_of_recruitment">6</field>
    </record>

    <record id="hr_recruitment_linkedin_developer" model="hr.recruitment.source">
        <field name="source_id" ref="utm.utm_source_linkedin"/>
        <field name="job_id" ref="hr.job_developer"/>
    </record>

    <record id="hr_recruitment_linkedin_ceo" model="hr.recruitment.source">
        <field name="source_id" ref="utm.utm_source_linkedin"/>
        <field name="job_id" ref="hr.job_ceo"/>
    </record>

    <record id="hr_recruitment_linkedin_cto" model="hr.recruitment.source">
        <field name="source_id" ref="utm.utm_source_linkedin"/>
        <field name="job_id" ref="hr.job_cto"/>
    </record>

    <record id="hr_recruitment_linkedin_consultant" model="hr.recruitment.source">
        <field name="source_id" ref="utm.utm_source_linkedin"/>
        <field name="job_id" ref="hr.job_consultant"/>
    </record>

    <record id="hr_recruitment_linkedin_hrm" model="hr.recruitment.source">
        <field name="source_id" ref="utm.utm_source_linkedin"/>
        <field name="job_id" ref="hr.job_hrm"/>
    </record>

    <record id="hr_recruitment_linkedin_marketing" model="hr.recruitment.source">
        <field name="source_id" ref="utm.utm_source_linkedin"/>
        <field name="job_id" ref="hr.job_marketing"/>
    </record>

    <record id="hr_recruitment_linkedin_trainee" model="hr.recruitment.source">
        <field name="source_id" ref="utm.utm_source_linkedin"/>
        <field name="job_id" ref="hr.job_trainee"/>
    </record>

    <record id="hr_case_salesman0" model="hr.applicant">
        <field name="name">Sales Manager</field>
        <field name="job_id" ref="hr.job_marketing"/>
        <field name="department_id" ref="hr.dep_sales"/>
        <field name="medium_id" ref="utm.utm_medium_direct"/>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_sales')])]"/>
        <field name="user_id" ref="base.user_demo"/>
        <field name="priority">1</field>
        <field name="partner_name">Enrique Jones</field>
        <field name="partner_mobile">9963214587</field>
        <field name="stage_id" ref="stage_job1"/>
    </record>
    <record id="hr_case_salesman1" model="hr.applicant">
        <field name="name">Sales</field>
        <field name="job_id" ref="hr.job_marketing"/>
        <field name="department_id" ref="hr.dep_sales"/>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_sales')])]"/>
        <field name="user_id" ref="base.user_demo"/>
        <field name="priority">1</field>
        <field name="partner_name">Meldona Thang</field>
        <field name="partner_mobile">998655451</field>
        <field name="stage_id" ref="stage_job1"/>
    </record>
    <record id="hr_case_dev0" model="hr.applicant">
        <field name="name">Developer PHP</field>
        <field name="job_id" ref="hr.job_developer"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="medium_id" ref="utm.utm_medium_email"/>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_it')])]"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="priority">3</field>
        <field name="partner_name">Johan Duck</field>
        <field name="partner_mobile">8955545</field>
        <field name="stage_id" ref="stage_job1"/>
    </record>
    <record id="hr_case_dev1" model="hr.applicant">
        <field name="name">Developer Fullstack</field>
        <field name="job_id" ref="hr.job_developer"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_it')])]"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="priority">0</field>
        <field name="partner_name">Kelly Wallant</field>
        <field name="partner_mobile">879895515</field>
        <field name="stage_id" ref="stage_job1"/>
    </record>
    <record id="hr_case_dev2" model="hr.applicant">
        <field name="name">Developer Python</field>
        <field name="job_id" ref="hr.job_developer"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="medium_id" ref="utm.utm_medium_email"/>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_it')])]"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="priority">0</field>
        <field name="partner_name">Cécile Donth</field>
        <field name="partner_mobile">98765411</field>
        <field name="stage_id" ref="stage_job1"/>
    </record>
    <record id="hr_case_dev3" model="hr.applicant">
        <field name="name">Developer C/C++</field>
        <field name="job_id" ref="hr.job_developer"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_it')])]"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="priority">0</field>
        <field name="partner_name">Ohen Rizome</field>
        <field name="partner_mobile">654687987654</field>
        <field name="stage_id" ref="stage_job1"/>
    </record>
    <record id="hr_case_traineemca0" model="hr.applicant">
        <field name="name">Trainee - MCA</field>
        <field name="job_id" ref="hr.job_trainee"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_manager')])]"/>
        <field name="user_id" ref="base.user_demo"/>
        <field name="priority">2</field>
        <field name="partner_name">Marie Justine</field>
        <field name="partner_mobile">9988774455</field>
        <field name="stage_id" ref="stage_job4"/>
        <field name="partner_phone">6633225</field>
    </record>
    <record id="hr_case_fresher0" model="hr.applicant">
        <field name="name">Fresher</field>
        <field name="job_id" ref="hr.job_trainee"/>
        <field name="department_id" ref="hr.dep_administration"/>
        <field name="type_id" ref="degree_bachelor"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_it')])]"/>
        <field name="user_id" ref="base.user_demo"/>
        <field name="priority">0</field>
        <field name="partner_name">Jose</field>
        <field name="stage_id" ref="stage_job3"/>
        <field name="partner_phone">999666735</field>
    </record>
    <record id="hr_case_mkt0" model="hr.applicant">
        <field name="name">Marketing</field>
        <field name="job_id" ref="hr.job_marketing"/>
        <field name="department_id" ref="hr.dep_sales"/>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_manager')])]"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_name">Yin Lee</field>
        <field name="stage_id" ref="stage_job1"/>
    </record>
    <record id="hr_case_mkt1" model="hr.applicant">
        <field name="name">Marketing 2 Year Experience</field>
        <field name="job_id" ref="hr.job_marketing"/>
        <field name="department_id" ref="hr.dep_sales"/>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_manager')])]"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_name">Hubert Blank</field>
        <field name="priority">3</field>
        <field name="stage_id" ref="stage_job3"/>
    </record>
    <record id="hr_case_yrsexperienceinphp0" model="hr.applicant">
        <field name="name">Marketing Job</field>
        <field eval="(datetime.now()+relativedelta(months=-2)).strftime('%Y-%m-03 01:00:00')" name="create_date"/>
        <field name="job_id" ref="hr.job_marketing"/>
        <field name="department_id" ref="hr.dep_sales"/>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_manager')])]"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_name">John Bruno</field>
        <field name="stage_id" ref="stage_job5"/>
    </record>
    <record id="hr_case_marketingjob0" model="hr.applicant">
        <field name="name">More than 5 yrs Experience in PHP</field>
        <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-08 01:00:00')" name="create_date"/>
        <field name="job_id" ref="hr.job_developer"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_reserve')])]"/>
        <field name="user_id" ref="base.user_demo"/>
        <field name="partner_name">Sandra Elvis</field>
        <field name="stage_id" ref="stage_job5"/>
    </record>
    <record id="hr_case_financejob0" model="hr.applicant">
        <field name="name">Finance Manager</field>
        <field name="job_id" ref="hr.job_hrm"/>
        <field name="department_id" ref="hr.dep_administration"/>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_reserve')])]"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="priority">1</field>
        <field name="partner_name">David Armstrong</field>
        <field name="stage_id" ref="stage_job2"/>
        <field name="partner_phone">33968745</field>
    </record>
    <record id="hr_case_financejob1" model="hr.applicant">
        <field name="name">Finance</field>
        <field name="job_id" ref="hr.job_hrm"/>
        <field name="department_id" ref="hr.dep_administration"/>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_reserve')])]"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="priority">1</field>
        <field name="partner_name">Joren Jacob</field>
        <field name="stage_id" ref="stage_job2"/>
    </record>
    <record id="hr_case_traineemca1" model="hr.applicant">
        <field name="name">Trainee - MCA</field>
        <field name="job_id" ref="hr.job_trainee"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_sales')])]"/>
        <field name="partner_name">Tina Augustie</field>
        <field name="partner_mobile">9898745745</field>
        <field name="stage_id" ref="stage_job4"/>
        <field name="partner_phone">6630125</field>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="hr_case_programmer" model="hr.applicant">
        <field name="name">Programmer</field>
        <field name="job_id" ref="hr.job_developer"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_it')])]"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_name">Shane Williams</field>
        <field name="partner_mobile">9812398524</field>
        <field name="stage_id" ref="stage_job4"/>
        <field name="partner_phone">6630125</field>
        <field name="salary_expected">11000.0</field>
    </record>
    <record id="hr_case_advertisement" model="hr.applicant">
        <field name="name">Advertisement</field>
        <field name="job_id" ref="hr.job_consultant"/>
        <field name="department_id" ref="hr.dep_ps"/>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_it')])]"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_name">David Billy</field>
        <field name="partner_mobile">9988774455</field>
        <field name="stage_id" ref="stage_job2"/>
        <field name="salary_expected">11000.0</field>
    </record>

    <record id="hr_case_salesman0_cv" model="ir.attachment">
        <field name="name">Jones_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/data/hr_recruitment_demo_jones_cv.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_salesman0"/>
    </record>
    <record id="hr_case_fresher0_cv" model="ir.attachment">
        <field name="name">Jose_CV.txt</field>
        <field name="datas" type="base64" file="hr_recruitment/data/hr_recruitment_demo_jose_cv.txt"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_fresher0"/>
    </record>
    <record id="hr_case_programmer_cv" model="ir.attachment">
        <field name="name">Williams_CV.doc</field>
        <field name="datas" type="base64" file="hr_recruitment/data/hr_recruitment_demo_williams_cv.doc"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_programmer"/>
    </record>

    <record id="message_application_demo" model="mail.message">
        <field name="model">hr.applicant</field>
        <field name="res_id" ref="hr_case_advertisement"/>
        <field name="body">Please do refer to this application for sure.</field>
        <field name="message_type">comment</field>
        <field name="author_id" ref="base.res_partner_2"/>
    </record>
    <record id="msg_case18_aplicant" model="mail.message">
        <field name="subject">Regarding reference</field>
        <field name="model">hr.applicant</field>
        <field name="res_id" ref="hr_case_advertisement"/>
        <field name="body" type="xml">
            <p>Hello!<br />
            I will surely refer to this application as it is by your reference and <br />
            will try to conduct an interview within a very short time<br />
            Thanks,</p>
        </field>
        <field name="message_type">comment</field>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="author_id" ref="base.partner_demo"/>
    </record>
    <function model="mail.message" name="toggle_message_starred"
            eval="[ref('msg_case18_aplicant')]"
    />
    <record id="msg_case_salesman0_aplicant" model="mail.message">
        <field name="subject">Refuse Application</field>
        <field name="model">hr.applicant</field>
        <field name="res_id" ref="hr_case_salesman0"/>
        <field name="body" type="xml">
            <p>Hello,</p>
            <p>I have checked this application but it does not match with our requirements. We don't need to proceed further and we should refuse this application.</p>
            <p>Kind regards,</p>
        </field>
        <field name="message_type">comment</field>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="author_id" ref="base.partner_demo"/>
    </record>
    <record id="msg_case_dev0_aplicant" model="mail.message">
        <field name="subject">Refuse Application</field>
        <field name="model">hr.applicant</field>
        <field name="res_id" ref="hr_case_dev0"/>
        <field name="body" type="xml">
            <p>Hello,</p>
            <p>This applicant has excellent skills and would greatly fit in the RD Team!</p>
            <p>Kind regards,</p>
        </field>
        <field name="message_type">comment</field>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="author_id" ref="base.partner_demo"/>
    </record>
    <record id="msg_case_fresher0_aplicant" model="mail.message">
        <field name="model">hr.applicant</field>
        <field name="res_id" ref="hr_case_fresher0"/>
        <field name="body" type="xml">
            <p>Hello,</p>
            <p>We should move further for this application as early as possible.</p>
            <p>Kind regards,</p>
        </field>
        <field name="message_type">comment</field>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="author_id" ref="base.partner_demo"/>
    </record>
    <record id="msg_case_advertisement_aplicant" model="mail.message">
        <field name="model">hr.applicant</field>
        <field name="res_id" ref="hr_case_advertisement"/>
        <field name="body" type="xml">
            <p>Hello,</p>
            <p>The first interview was good. Skilled and open minded applicant.</p>
            <p>I think we should consider hiring him.</p>
            <p>Kind regards,</p>
        </field>
        <field name="message_type">comment</field>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="author_id" ref="base.partner_demo"/>
    </record>
    <record id="msg_case_mkt1_1" model="mail.message">
        <field name="model">hr.applicant</field>
        <field name="res_id" ref="hr_case_mkt1"/>
        <field name="body" type="xml">
            <p>Hello,</p>
            <p>The first interview was good. I will propose a second interview</p>
            <p>Kind regards,</p>
        </field>
        <field name="message_type">comment</field>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="author_id" ref="base.partner_demo"/>
    </record>
    <record id="msg_case_mkt1_2" model="mail.message">
        <field name="model">hr.applicant</field>
        <field name="res_id" ref="hr_case_mkt1"/>
        <field name="body" type="xml">
            <p>Hello,</p>
            <p>After the second interview, I think we should consider hiring him.</p>
            <p>Kind regards,</p>
        </field>
        <field name="message_type">comment</field>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="author_id" ref="base.partner_admin"/>
    </record>
    <record id="mail_activity_0" model="mail.activity">
        <field name="res_id" ref="hr_recruitment.hr_case_dev0" />
        <field name="res_model_id" ref="model_hr_applicant"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_email" />
        <field name="date_deadline" eval="time.strftime('%Y-%m-27 18:15:00')"/>
        <field name="summary">Send mail regarding our interview</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="mail_activity_1" model="mail.activity">
        <field name="res_id" ref="hr_recruitment.hr_case_dev1" />
        <field name="res_model_id" ref="model_hr_applicant"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_email" />
        <field name="date_deadline" eval="time.strftime('%Y-%m-%d')"/>
        <field name="summary">Send mail for first interview</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="mail_activity_2" model="mail.activity">
        <field name="res_id" ref="hr_recruitment.hr_case_salesman0" />
        <field name="res_model_id" ref="model_hr_applicant"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_email" />
        <field name="date_deadline" eval="time.strftime('%Y-%m-15 18:15:00')"/>
        <field name="summary">Send mail regarding our interview</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="mail_activity_3" model="mail.activity">
        <field name="res_id" ref="hr_recruitment.hr_case_traineemca0" />
        <field name="res_model_id" ref="model_hr_applicant"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_call" />
        <field name="date_deadline" eval="time.strftime('%Y-%m-10 18:15:00')"/>
        <field name="summary">Call to define real needs about training</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="mail_activity_4" model="mail.activity">
        <field name="res_id" ref="hr_recruitment.hr_case_yrsexperienceinphp0" />
        <field name="res_model_id" ref="model_hr_applicant"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_call" />
        <field name="date_deadline" eval="time.strftime('%Y-%m-24 18:15:00')"/>
        <field name="summary">Call to define real needs about training</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="mail_activity_5" model="mail.activity">
        <field name="res_id" ref="hr_recruitment.hr_case_advertisement" />
        <field name="res_model_id" ref="model_hr_applicant"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_call" />
        <field name="date_deadline" eval="time.strftime('%Y-%m-26 18:15:00')"/>
        <field name="summary">Call to schedule a second interview</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="mail_activity_6" model="mail.activity">
        <field name="res_id" ref="hr_recruitment.hr_case_mkt1" />
        <field name="res_model_id" ref="model_hr_applicant"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_call" />
        <field name="date_deadline" eval="time.strftime('%Y-%m-18 17:15:00')"/>
        <field name="summary">Call to propose a contract</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>

  </data>
</odoo>

```

## File: data\hr_recruitment_demo_jose_cv.txt

```text
Profile

Name          : Jose
Address       : 93, Press Avenue
              : Le Bourget du Lac, 73377,
              : France
Qualification : MCA
Email         : Jose@gmail.com
Mobile        : 9968513587

```

## File: data\hr_recruitment_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<odoo>
    <data>
        <template id="applicant_hired_template">
Applicant hired<br/>
<ul>
    <li>Employee: <a href="#" t-att-data-oe-id="applicant.emp_id.id" data-oe-model="hr.employee"><t t-esc="applicant.emp_id.name"/></a></li>
</ul>
        </template>
    </data>
</odoo>
```

## File: models\calendar.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class CalendarEvent(models.Model):
    """ Model for Calendar Event """
    _inherit = 'calendar.event'

    @api.model
    def default_get(self, fields):
        if self.env.context.get('default_applicant_id'):
            self = self.with_context(
                default_res_model='hr.applicant', #res_model seems to be lost without this
                default_res_model_id=self.env.ref('hr_recruitment.model_hr_applicant').id,
                default_res_id=self.env.context['default_applicant_id']
            )

        defaults = super(CalendarEvent, self).default_get(fields)

        # sync res_model / res_id to opportunity id (aka creating meeting from lead chatter)
        if 'applicant_id' not in defaults:
            res_model = defaults.get('res_model', False) or self.env.context.get('default_res_model')
            res_model_id = defaults.get('res_model_id', False) or self.env.context.get('default_res_model_id')
            if (res_model and res_model == 'hr.applicant') or (res_model_id and self.env['ir.model'].sudo().browse(res_model_id).model == 'hr.applicant'):
                defaults['applicant_id'] = defaults.get('res_id', False) or self.env.context.get('default_res_id', False)

        return defaults

    def _compute_is_highlighted(self):
        super(CalendarEvent, self)._compute_is_highlighted()
        applicant_id = self.env.context.get('active_id')
        if self.env.context.get('active_model') == 'hr.applicant' and applicant_id:
            for event in self:
                if event.applicant_id.id == applicant_id:
                    event.is_highlighted = True

    applicant_id = fields.Many2one('hr.applicant', string="Applicant", index=True, ondelete='set null')

```

## File: models\digest.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.exceptions import AccessError


class Digest(models.Model):
    _inherit = 'digest.digest'

    kpi_hr_recruitment_new_colleagues = fields.Boolean('Employees')
    kpi_hr_recruitment_new_colleagues_value = fields.Integer(compute='_compute_kpi_hr_recruitment_new_colleagues_value')

    def _compute_kpi_hr_recruitment_new_colleagues_value(self):
        if not self.env.user.has_group('hr_recruitment.group_hr_recruitment_user'):
            raise AccessError(_("Do not have access, skip this data for user's digest email"))
        for record in self:
            start, end, company = record._get_kpi_compute_parameters()
            new_colleagues = self.env['hr.employee'].search_count([
                ('create_date', '>=', start),
                ('create_date', '<', end),
                ('company_id', '=', company.id)
            ])
            record.kpi_hr_recruitment_new_colleagues_value = new_colleagues

    def _compute_kpis_actions(self, company, user):
        res = super(Digest, self)._compute_kpis_actions(company, user)
        res['kpi_hr_recruitment_new_colleagues'] = 'hr.open_view_employee_list_my&menu_id=%s' % self.env.ref('hr.menu_hr_root').id
        return res

```

## File: models\hr_department.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models


class HrDepartment(models.Model):
    _inherit = 'hr.department'

    new_applicant_count = fields.Integer(
        compute='_compute_new_applicant_count', string='New Applicant')
    new_hired_employee = fields.Integer(
        compute='_compute_recruitment_stats', string='New Hired Employee')
    expected_employee = fields.Integer(
        compute='_compute_recruitment_stats', string='Expected Employee')

    def _compute_new_applicant_count(self):
        applicant_data = self.env['hr.applicant'].read_group(
            [('department_id', 'in', self.ids), ('stage_id.sequence', '<=', '1')],
            ['department_id'], ['department_id'])
        result = dict((data['department_id'][0], data['department_id_count']) for data in applicant_data)
        for department in self:
            department.new_applicant_count = result.get(department.id, 0)

    def _compute_recruitment_stats(self):
        job_data = self.env['hr.job'].read_group(
            [('department_id', 'in', self.ids)],
            ['no_of_hired_employee', 'no_of_recruitment', 'department_id'], ['department_id'])
        new_emp = dict((data['department_id'][0], data['no_of_hired_employee']) for data in job_data)
        expected_emp = dict((data['department_id'][0], data['no_of_recruitment']) for data in job_data)
        for department in self:
            department.new_hired_employee = new_emp.get(department.id, 0)
            department.expected_employee = expected_emp.get(department.id, 0)

```

## File: models\hr_employee.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.tools.translate import _
from datetime import timedelta


class HrEmployee(models.Model):
    _inherit = "hr.employee"

    newly_hired_employee = fields.Boolean('Newly hired employee', compute='_compute_newly_hired_employee',
                                          search='_search_newly_hired_employee')
    applicant_id = fields.One2many('hr.applicant', 'emp_id', 'Applicant')

    def _compute_newly_hired_employee(self):
        now = fields.Datetime.now()
        for employee in self:
            employee.newly_hired_employee = bool(employee.create_date > (now - timedelta(days=90)))

    def _search_newly_hired_employee(self, operator, value):
        employees = self.env['hr.employee'].search([
            ('create_date', '>', fields.Datetime.now() - timedelta(days=90))
        ])
        return [('id', 'in', employees.ids)]

    @api.model
    def create(self, vals):
        new_employee = super(HrEmployee, self).create(vals)
        if new_employee.applicant_id:
            new_employee.applicant_id.message_post_with_view(
                        'hr_recruitment.applicant_hired_template',
                        values={'applicant': new_employee.applicant_id},
                        subtype_id=self.env.ref("hr_recruitment.mt_applicant_hired").id)
        return new_employee

```

## File: models\hr_job.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast

from odoo import api, fields, models, _


class Job(models.Model):
    _name = "hr.job"
    _inherit = ["mail.alias.mixin", "hr.job"]
    _order = "state desc, name asc"

    @api.model
    def _default_address_id(self):
        return self.env.company.partner_id

    def _get_default_favorite_user_ids(self):
        return [(6, 0, [self.env.uid])]

    address_id = fields.Many2one(
        'res.partner', "Job Location", default=_default_address_id,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        help="Address where employees are working")
    application_ids = fields.One2many('hr.applicant', 'job_id', "Applications")
    application_count = fields.Integer(compute='_compute_application_count', string="Application Count")
    all_application_count = fields.Integer(compute='_compute_all_application_count', string="All Application Count")
    new_application_count = fields.Integer(
        compute='_compute_new_application_count', string="New Application",
        help="Number of applications that are new in the flow (typically at first step of the flow)")
    manager_id = fields.Many2one(
        'hr.employee', related='department_id.manager_id', string="Department Manager",
        readonly=True, store=True)
    user_id = fields.Many2one('res.users', "Recruiter", tracking=True)
    hr_responsible_id = fields.Many2one(
        'res.users', "HR Responsible", tracking=True,
        help="Person responsible of validating the employee's contracts.")
    document_ids = fields.One2many('ir.attachment', compute='_compute_document_ids', string="Documents")
    documents_count = fields.Integer(compute='_compute_document_ids', string="Document Count")
    alias_id = fields.Many2one(
        'mail.alias', "Alias", ondelete="restrict", required=True,
        help="Email alias for this job position. New emails will automatically create new applicants for this job position.")
    color = fields.Integer("Color Index")
    is_favorite = fields.Boolean(compute='_compute_is_favorite', inverse='_inverse_is_favorite')
    favorite_user_ids = fields.Many2many('res.users', 'job_favorite_user_rel', 'job_id', 'user_id', default=_get_default_favorite_user_ids)

    def _compute_is_favorite(self):
        for job in self:
            job.is_favorite = self.env.user in job.favorite_user_ids

    def _inverse_is_favorite(self):
        unfavorited_jobs = favorited_jobs = self.env['hr.job']
        for job in self:
            if self.env.user in job.favorite_user_ids:
                unfavorited_jobs |= job
            else:
                favorited_jobs |= job
        favorited_jobs.write({'favorite_user_ids': [(4, self.env.uid)]})
        unfavorited_jobs.write({'favorite_user_ids': [(3, self.env.uid)]})

    def _compute_document_ids(self):
        applicants = self.mapped('application_ids').filtered(lambda self: not self.emp_id)
        app_to_job = dict((applicant.id, applicant.job_id.id) for applicant in applicants)
        attachments = self.env['ir.attachment'].search([
            '|',
            '&', ('res_model', '=', 'hr.job'), ('res_id', 'in', self.ids),
            '&', ('res_model', '=', 'hr.applicant'), ('res_id', 'in', applicants.ids)])
        result = dict.fromkeys(self.ids, self.env['ir.attachment'])
        for attachment in attachments:
            if attachment.res_model == 'hr.applicant':
                result[app_to_job[attachment.res_id]] |= attachment
            else:
                result[attachment.res_id] |= attachment

        for job in self:
            job.document_ids = result.get(job.id, False)
            job.documents_count = len(job.document_ids)

    def _compute_all_application_count(self):
        read_group_result = self.env['hr.applicant'].with_context(active_test=False).read_group([('job_id', 'in', self.ids)], ['job_id'], ['job_id'])
        result = dict((data['job_id'][0], data['job_id_count']) for data in read_group_result)
        for job in self:
            job.all_application_count = result.get(job.id, 0)

    def _compute_application_count(self):
        read_group_result = self.env['hr.applicant'].read_group([('job_id', 'in', self.ids)], ['job_id'], ['job_id'])
        result = dict((data['job_id'][0], data['job_id_count']) for data in read_group_result)
        for job in self:
            job.application_count = result.get(job.id, 0)

    def _get_first_stage(self):
        self.ensure_one()
        return self.env['hr.recruitment.stage'].search([
            '|',
            ('job_ids', '=', False),
            ('job_ids', '=', self.id)], order='sequence asc', limit=1)

    def _compute_new_application_count(self):
        self.env.cr.execute(
            """
                WITH job_stage AS (
                    SELECT DISTINCT ON (j.id) j.id AS job_id, s.id AS stage_id, s.sequence AS sequence
                      FROM hr_job j
                 LEFT JOIN hr_job_hr_recruitment_stage_rel rel
                        ON rel.hr_job_id = j.id
                      JOIN hr_recruitment_stage s
                        ON s.id = rel.hr_recruitment_stage_id
                        OR s.id NOT IN (
                                        SELECT "hr_recruitment_stage_id"
                                          FROM "hr_job_hr_recruitment_stage_rel"
                                         WHERE "hr_recruitment_stage_id" IS NOT NULL
                                        )
                     WHERE j.id in %s
                  ORDER BY 1, 3 asc
                )
                SELECT s.job_id, COUNT(a.id) AS new_applicant
                  FROM hr_applicant a
                  JOIN job_stage s
                    ON s.job_id = a.job_id
                   AND a.stage_id = s.stage_id
                   AND a.active IS TRUE
              GROUP BY s.job_id
            """, [tuple(self.ids), ]
        )

        new_applicant_count = dict(self.env.cr.fetchall())
        for job in self:
            job.new_application_count = new_applicant_count.get(job.id, 0)

    def _alias_get_creation_values(self):
        values = super(Job, self)._alias_get_creation_values()
        values['alias_model_id'] = self.env['ir.model']._get('hr.applicant').id
        if self.id:
            values['alias_defaults'] = defaults = ast.literal_eval(self.alias_defaults or "{}")
            defaults.update({
                'job_id': self.id,
                'department_id': self.department_id.id,
                'company_id': self.department_id.company_id.id if self.department_id else self.company_id.id,
            })
        return values

    @api.model
    def create(self, vals):
        vals['favorite_user_ids'] = vals.get('favorite_user_ids', []) + [(4, self.env.uid)]
        new_job = super(Job, self).create(vals)
        utm_linkedin = self.env.ref("utm.utm_source_linkedin", raise_if_not_found=False)
        if utm_linkedin:
            source_vals = {
                'source_id': utm_linkedin.id,
                'job_id': new_job.id,
            }
            self.env['hr.recruitment.source'].create(source_vals)
        return new_job

    def _creation_subtype(self):
        return self.env.ref('hr_recruitment.mt_job_new')

    def action_get_attachment_tree_view(self):
        action = self.env["ir.actions.actions"]._for_xml_id("base.action_attachment")
        action['context'] = {
            'default_res_model': self._name,
            'default_res_id': self.ids[0]
        }
        action['search_view_id'] = (self.env.ref('hr_recruitment.ir_attachment_view_search_inherit_hr_recruitment').id, )
        action['domain'] = ['|', '&', ('res_model', '=', 'hr.job'), ('res_id', 'in', self.ids), '&', ('res_model', '=', 'hr.applicant'), ('res_id', 'in', self.mapped('application_ids').ids)]
        return action

    def close_dialog(self):
        return {'type': 'ir.actions.act_window_close'}

    def edit_dialog(self):
        form_view = self.env.ref('hr.view_hr_job_form')
        return {
            'name': _('Job'),
            'res_model': 'hr.job',
            'res_id': self.id,
            'views': [(form_view.id, 'form'),],
            'type': 'ir.actions.act_window',
            'target': 'inline'
        }

```

## File: models\hr_recruitment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint

from odoo import api, fields, models, tools, SUPERUSER_ID
from odoo.tools.translate import _
from odoo.exceptions import UserError

AVAILABLE_PRIORITIES = [
    ('0', 'Normal'),
    ('1', 'Good'),
    ('2', 'Very Good'),
    ('3', 'Excellent')
]


class RecruitmentSource(models.Model):
    _name = "hr.recruitment.source"
    _description = "Source of Applicants"
    _inherits = {"utm.source": "source_id"}

    source_id = fields.Many2one('utm.source', "Source", ondelete='cascade', required=True)
    email = fields.Char(related='alias_id.display_name', string="Email", readonly=True)
    job_id = fields.Many2one('hr.job', "Job", ondelete='cascade')
    alias_id = fields.Many2one('mail.alias', "Alias ID")

    def create_alias(self):
        campaign = self.env.ref('hr_recruitment.utm_campaign_job')
        medium = self.env.ref('utm.utm_medium_email')
        for source in self:
            vals = {
                'alias_parent_thread_id': source.job_id.id,
                'alias_model_id': self.env['ir.model']._get('hr.applicant').id,
                'alias_parent_model_id': self.env['ir.model']._get('hr.job').id,
                'alias_name': "%s+%s" % (source.job_id.alias_name or source.job_id.name, source.name),
                'alias_defaults': {
                    'job_id': source.job_id.id,
                    'campaign_id': campaign.id,
                    'medium_id': medium.id,
                    'source_id': source.source_id.id,
                },
            }
            source.alias_id = self.env['mail.alias'].create(vals)
            source.name = source.source_id.name

class RecruitmentStage(models.Model):
    _name = "hr.recruitment.stage"
    _description = "Recruitment Stages"
    _order = 'sequence'

    name = fields.Char("Stage Name", required=True, translate=True)
    sequence = fields.Integer(
        "Sequence", default=10,
        help="Gives the sequence order when displaying a list of stages.")
    job_ids = fields.Many2many(
        'hr.job', string='Job Specific',
        help='Specific jobs that uses this stage. Other jobs will not use this stage.')
    requirements = fields.Text("Requirements")
    template_id = fields.Many2one(
        'mail.template', "Email Template",
        help="If set, a message is posted on the applicant using the template when the applicant is set to the stage.")
    fold = fields.Boolean(
        "Folded in Kanban",
        help="This stage is folded in the kanban view when there are no records in that stage to display.")
    legend_blocked = fields.Char(
        'Red Kanban Label', default=lambda self: _('Blocked'), translate=True, required=True)
    legend_done = fields.Char(
        'Green Kanban Label', default=lambda self: _('Ready for Next Stage'), translate=True, required=True)
    legend_normal = fields.Char(
        'Grey Kanban Label', default=lambda self: _('In Progress'), translate=True, required=True)

    @api.model
    def default_get(self, fields):
        if self._context and self._context.get('default_job_id') and not self._context.get('hr_recruitment_stage_mono', False):
            context = dict(self._context)
            context.pop('default_job_id')
            self = self.with_context(context)
        return super(RecruitmentStage, self).default_get(fields)


class RecruitmentDegree(models.Model):
    _name = "hr.recruitment.degree"
    _description = "Applicant Degree"
    _sql_constraints = [
        ('name_uniq', 'unique (name)', 'The name of the Degree of Recruitment must be unique!')
    ]

    name = fields.Char("Degree Name", required=True, translate=True)
    sequence = fields.Integer("Sequence", default=1, help="Gives the sequence order when displaying a list of degrees.")


class Applicant(models.Model):
    _name = "hr.applicant"
    _description = "Applicant"
    _order = "priority desc, id desc"
    _inherit = ['mail.thread.cc', 'mail.activity.mixin', 'utm.mixin']

    name = fields.Char("Subject / Application Name", required=True, help="Email subject for applications sent via email")
    active = fields.Boolean("Active", default=True, help="If the active field is set to false, it will allow you to hide the case without removing it.")
    description = fields.Text("Description")
    email_from = fields.Char("Email", size=128, help="Applicant email", compute='_compute_partner_phone_email',
        inverse='_inverse_partner_email', store=True)
    probability = fields.Float("Probability")
    partner_id = fields.Many2one('res.partner', "Contact", copy=False)
    create_date = fields.Datetime("Creation Date", readonly=True, index=True)
    stage_id = fields.Many2one('hr.recruitment.stage', 'Stage', ondelete='restrict', tracking=True,
                               compute='_compute_stage', store=True, readonly=False,
                               domain="['|', ('job_ids', '=', False), ('job_ids', '=', job_id)]",
                               copy=False, index=True,
                               group_expand='_read_group_stage_ids')
    last_stage_id = fields.Many2one('hr.recruitment.stage', "Last Stage",
                                    help="Stage of the applicant before being in the current stage. Used for lost cases analysis.")
    categ_ids = fields.Many2many('hr.applicant.category', string="Tags")
    company_id = fields.Many2one('res.company', "Company", compute='_compute_company', store=True, readonly=False, tracking=True)
    user_id = fields.Many2one(
        'res.users', "Recruiter", compute='_compute_user',
        tracking=True, store=True, readonly=False)
    date_closed = fields.Datetime("Closed", compute='_compute_date_closed', store=True, index=True)
    date_open = fields.Datetime("Assigned", readonly=True, index=True)
    date_last_stage_update = fields.Datetime("Last Stage Update", index=True, default=fields.Datetime.now)
    priority = fields.Selection(AVAILABLE_PRIORITIES, "Appreciation", default='0')
    job_id = fields.Many2one('hr.job', "Applied Job", domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", tracking=True, index=True)
    salary_proposed_extra = fields.Char("Proposed Salary Extra", help="Salary Proposed by the Organisation, extra advantages", tracking=True)
    salary_expected_extra = fields.Char("Expected Salary Extra", help="Salary Expected by Applicant, extra advantages", tracking=True)
    salary_proposed = fields.Float("Proposed Salary", group_operator="avg", help="Salary Proposed by the Organisation", tracking=True)
    salary_expected = fields.Float("Expected Salary", group_operator="avg", help="Salary Expected by Applicant", tracking=True)
    availability = fields.Date("Availability", help="The date at which the applicant will be available to start working", tracking=True)
    partner_name = fields.Char("Applicant's Name")
    partner_phone = fields.Char("Phone", size=32, compute='_compute_partner_phone_email',
        inverse='_inverse_partner_phone', store=True)
    partner_mobile = fields.Char("Mobile", size=32, compute='_compute_partner_phone_email',
        inverse='_inverse_partner_mobile', store=True)
    type_id = fields.Many2one('hr.recruitment.degree', "Degree")
    department_id = fields.Many2one(
        'hr.department', "Department", compute='_compute_department', store=True, readonly=False,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", tracking=True)
    day_open = fields.Float(compute='_compute_day', string="Days to Open", compute_sudo=True)
    day_close = fields.Float(compute='_compute_day', string="Days to Close", compute_sudo=True)
    delay_close = fields.Float(compute="_compute_day", string='Delay to Close', readonly=True, group_operator="avg", help="Number of days to close", store=True)
    color = fields.Integer("Color Index", default=0)
    emp_id = fields.Many2one('hr.employee', string="Employee", help="Employee linked to the applicant.", copy=False)
    user_email = fields.Char(related='user_id.email', string="User Email", readonly=True)
    attachment_number = fields.Integer(compute='_get_attachment_number', string="Number of Attachments")
    employee_name = fields.Char(related='emp_id.name', string="Employee Name", readonly=False, tracking=False)
    attachment_ids = fields.One2many('ir.attachment', 'res_id', domain=[('res_model', '=', 'hr.applicant')], string='Attachments')
    kanban_state = fields.Selection([
        ('normal', 'Grey'),
        ('done', 'Green'),
        ('blocked', 'Red')], string='Kanban State',
        copy=False, default='normal', required=True)
    legend_blocked = fields.Char(related='stage_id.legend_blocked', string='Kanban Blocked')
    legend_done = fields.Char(related='stage_id.legend_done', string='Kanban Valid')
    legend_normal = fields.Char(related='stage_id.legend_normal', string='Kanban Ongoing')
    application_count = fields.Integer(compute='_compute_application_count', help='Applications with the same email')
    meeting_count = fields.Integer(compute='_compute_meeting_count', help='Meeting Count')
    refuse_reason_id = fields.Many2one('hr.applicant.refuse.reason', string='Refuse Reason', tracking=True)

    @api.depends('date_open', 'date_closed')
    def _compute_day(self):
        for applicant in self:
            if applicant.date_open:
                date_create = applicant.create_date
                date_open = applicant.date_open
                applicant.day_open = (date_open - date_create).total_seconds() / (24.0 * 3600)
            else:
                applicant.day_open = False
            if applicant.date_closed:
                date_create = applicant.create_date
                date_closed = applicant.date_closed
                applicant.day_close = (date_closed - date_create).total_seconds() / (24.0 * 3600)
                applicant.delay_close = applicant.day_close - applicant.day_open
            else:
                applicant.day_close = False
                applicant.delay_close = False

    @api.depends('email_from')
    def _compute_application_count(self):
        application_data = self.env['hr.applicant'].with_context(active_test=False).read_group([
            ('email_from', 'in', list(set(self.mapped('email_from'))))], ['email_from'], ['email_from'])
        application_data_mapped = dict((data['email_from'], data['email_from_count']) for data in application_data)
        applicants = self.filtered(lambda applicant: applicant.email_from)
        for applicant in applicants:
            applicant.application_count = application_data_mapped.get(applicant.email_from, 1) - 1
        (self - applicants).application_count = False

    def _compute_meeting_count(self):
        if self.ids:
            meeting_data = self.env['calendar.event'].sudo().read_group(
                [('applicant_id', 'in', self.ids)],
                ['applicant_id'],
                ['applicant_id']
            )
            mapped_data = {m['applicant_id'][0]: m['applicant_id_count'] for m in meeting_data}
        else:
            mapped_data = dict()
        for applicant in self:
            applicant.meeting_count = mapped_data.get(applicant.id, 0)

    def _get_attachment_number(self):
        read_group_res = self.env['ir.attachment'].read_group(
            [('res_model', '=', 'hr.applicant'), ('res_id', 'in', self.ids)],
            ['res_id'], ['res_id'])
        attach_data = dict((res['res_id'], res['res_id_count']) for res in read_group_res)
        for record in self:
            record.attachment_number = attach_data.get(record.id, 0)

    @api.model
    def _read_group_stage_ids(self, stages, domain, order):
        # retrieve job_id from the context and write the domain: ids + contextual columns (job or default)
        job_id = self._context.get('default_job_id')
        search_domain = [('job_ids', '=', False)]
        if job_id:
            search_domain = ['|', ('job_ids', '=', job_id)] + search_domain
        if stages:
            search_domain = ['|', ('id', 'in', stages.ids)] + search_domain

        stage_ids = stages._search(search_domain, order=order, access_rights_uid=SUPERUSER_ID)
        return stages.browse(stage_ids)

    @api.depends('job_id', 'department_id')
    def _compute_company(self):
        for applicant in self:
            company_id = False
            if applicant.department_id:
                company_id = applicant.department_id.company_id.id
            if not company_id and applicant.job_id:
                company_id = applicant.job_id.company_id.id
            applicant.company_id = company_id or self.env.company.id

    @api.depends('job_id')
    def _compute_department(self):
        for applicant in self:
            applicant.department_id = applicant.job_id.department_id.id

    @api.depends('job_id')
    def _compute_stage(self):
        for applicant in self:
            if applicant.job_id:
                if not applicant.stage_id:
                    stage_ids = self.env['hr.recruitment.stage'].search([
                        '|',
                        ('job_ids', '=', False),
                        ('job_ids', '=', applicant.job_id.id),
                        ('fold', '=', False)
                    ], order='sequence asc', limit=1).ids
                    applicant.stage_id = stage_ids[0] if stage_ids else False
            else:
                applicant.stage_id = False

    @api.depends('job_id')
    def _compute_user(self):
        for applicant in self:
            applicant.user_id = applicant.job_id.user_id.id or self.env.uid

    @api.depends('partner_id')
    def _compute_partner_phone_email(self):
        for applicant in self:
            applicant.partner_phone = applicant.partner_id.phone
            applicant.partner_mobile = applicant.partner_id.mobile
            applicant.email_from = applicant.partner_id.email

    def _inverse_partner_email(self):
        for applicant in self.filtered(lambda a: a.partner_id and a.email_from and not a.partner_id.email):
            applicant.partner_id.email = applicant.email_from

    def _inverse_partner_phone(self):
        for applicant in self.filtered(lambda a: a.partner_id and a.partner_phone and not a.partner_id.phone):
            applicant.partner_id.phone = applicant.partner_phone

    def _inverse_partner_mobile(self):
        for applicant in self.filtered(lambda a: a.partner_id and a.partner_mobile and not a.partner_id.mobile):
            applicant.partner_id.mobile = applicant.partner_mobile

    @api.depends('stage_id')
    def _compute_date_closed(self):
        for applicant in self:
            if applicant.stage_id and applicant.stage_id.fold:
                applicant.date_closed = fields.datetime.now()
            else:
                applicant.date_closed = False

    @api.model
    def create(self, vals):
        if vals.get('department_id') and not self._context.get('default_department_id'):
            self = self.with_context(default_department_id=vals.get('department_id'))
        if vals.get('user_id'):
            vals['date_open'] = fields.Datetime.now()
        if vals.get('email_from'):
            vals['email_from'] = vals['email_from'].strip()
        return super(Applicant, self).create(vals)

    def write(self, vals):
        # user_id change: update date_open
        if vals.get('user_id'):
            vals['date_open'] = fields.Datetime.now()
        if vals.get('email_from'):
            vals['email_from'] = vals['email_from'].strip()
        # stage_id: track last stage before update
        if 'stage_id' in vals:
            vals['date_last_stage_update'] = fields.Datetime.now()
            if 'kanban_state' not in vals:
                vals['kanban_state'] = 'normal'
            for applicant in self:
                vals['last_stage_id'] = applicant.stage_id.id
                res = super(Applicant, self).write(vals)
        else:
            res = super(Applicant, self).write(vals)
        return res

    def get_empty_list_help(self, help):
        if 'active_id' in self.env.context and self.env.context.get('active_model') == 'hr.job':
            alias_id = self.env['hr.job'].browse(self.env.context['active_id']).alias_id
        else:
            alias_id = False

        nocontent_values = {
            'help_title': _('No application yet'),
            'para_1': _('Let people apply by email to save time.') ,
            'para_2': _('Attachments, like resumes, get indexed automatically.'),
        }
        nocontent_body = """
            <p class="o_view_nocontent_empty_folder">%(help_title)s</p>
            <p>%(para_1)s<br/>%(para_2)s</p>"""

        if alias_id and alias_id.alias_domain and alias_id.alias_name:
            email = alias_id.display_name 
            email_link = "<a href='mailto:%s'>%s</a>" % (email, email)
            nocontent_values['email_link'] = email_link
            nocontent_body += """<p class="o_copy_paste_email">%(email_link)s</p>"""

        return nocontent_body % nocontent_values

    def action_makeMeeting(self):
        """ This opens Meeting's calendar view to schedule meeting on current applicant
            @return: Dictionary value for created Meeting view
        """
        self.ensure_one()
        partners = self.partner_id | self.user_id.partner_id | self.department_id.manager_id.user_id.partner_id

        category = self.env.ref('hr_recruitment.categ_meet_interview')
        res = self.env['ir.actions.act_window']._for_xml_id('calendar.action_calendar_event')
        res['context'] = {
            'default_applicant_id': self.id,
            'default_partner_ids': partners.ids,
            'default_user_id': self.env.uid,
            'default_name': self.name,
            'default_categ_ids': category and [category.id] or False,
        }
        return res

    def action_get_attachment_tree_view(self):
        action = self.env['ir.actions.act_window']._for_xml_id('base.action_attachment')
        action['context'] = {'default_res_model': self._name, 'default_res_id': self.ids[0]}
        action['domain'] = str(['&', ('res_model', '=', self._name), ('res_id', 'in', self.ids)])
        action['search_view_id'] = (self.env.ref('hr_recruitment.ir_attachment_view_search_inherit_hr_recruitment').id, )
        return action

    def action_applications_email(self):
        return {
            'type': 'ir.actions.act_window',
            'name': _('Applications'),
            'res_model': self._name,
            'view_mode': 'kanban,tree,form,pivot,graph,calendar,activity',
            'domain': [('email_from', 'in', self.mapped('email_from'))],
            'context': {
                'active_test': False
            },
        }

    def _track_template(self, changes):
        res = super(Applicant, self)._track_template(changes)
        applicant = self[0]
        if 'stage_id' in changes and applicant.stage_id.template_id:
            res['stage_id'] = (applicant.stage_id.template_id, {
                'auto_delete_message': True,
                'subtype_id': self.env['ir.model.data'].xmlid_to_res_id('mail.mt_note'),
                'email_layout_xmlid': 'mail.mail_notification_light'
            })
        return res

    def _creation_subtype(self):
        return self.env.ref('hr_recruitment.mt_applicant_new')

    def _track_subtype(self, init_values):
        record = self[0]
        if 'stage_id' in init_values and record.stage_id:
            return self.env.ref('hr_recruitment.mt_applicant_stage_changed')
        return super(Applicant, self)._track_subtype(init_values)

    def _notify_get_reply_to(self, default=None, records=None, company=None, doc_names=None):
        """ Override to set alias of applicants to their job definition if any. """
        aliases = self.mapped('job_id')._notify_get_reply_to(default=default, records=None, company=company, doc_names=None)
        res = {app.id: aliases.get(app.job_id.id) for app in self}
        leftover = self.filtered(lambda rec: not rec.job_id)
        if leftover:
            res.update(super(Applicant, leftover)._notify_get_reply_to(default=default, records=None, company=company, doc_names=doc_names))
        return res

    def _message_get_suggested_recipients(self):
        recipients = super(Applicant, self)._message_get_suggested_recipients()
        for applicant in self:
            if applicant.partner_id:
                applicant._message_add_suggested_recipient(recipients, partner=applicant.partner_id, reason=_('Contact'))
            elif applicant.email_from:
                email_from = tools.email_normalize(applicant.email_from)
                if applicant.partner_name:
                    email_from = tools.formataddr((applicant.partner_name, email_from))
                applicant._message_add_suggested_recipient(recipients, email=email_from, reason=_('Contact Email'))
        return recipients

    @api.model
    def message_new(self, msg, custom_values=None):
        """ Overrides mail_thread message_new that is called by the mailgateway
            through message_process.
            This override updates the document according to the email.
        """
        # remove default author when going through the mail gateway. Indeed we
        # do not want to explicitly set user_id to False; however we do not
        # want the gateway user to be responsible if no other responsible is
        # found.
        self = self.with_context(default_user_id=False)
        stage = False
        if custom_values and 'job_id' in custom_values:
            stage = self.env['hr.job'].browse(custom_values['job_id'])._get_first_stage()
        val = msg.get('from').split('<')[0]
        defaults = {
            'name': msg.get('subject') or _("No Subject"),
            'partner_name': val,
            'email_from': msg.get('from'),
            'partner_id': msg.get('author_id', False),
        }
        if msg.get('priority'):
            defaults['priority'] = msg.get('priority')
        if stage and stage.id:
            defaults['stage_id'] = stage.id
        if custom_values:
            defaults.update(custom_values)
        return super(Applicant, self).message_new(msg, custom_values=defaults)

    def _message_post_after_hook(self, message, msg_vals):
        if self.email_from and not self.partner_id:
            # we consider that posting a message with a specified recipient (not a follower, a specific one)
            # on a document without customer means that it was created through the chatter using
            # suggested recipients. This heuristic allows to avoid ugly hacks in JS.
            email_normalized = tools.email_normalize(self.email_from)
            new_partner = message.partner_ids.filtered(
                lambda partner: partner.email == self.email_from or (email_normalized and partner.email_normalized == email_normalized)
            )
            if new_partner:
                if new_partner[0].create_date.date() == fields.Date.today():
                    new_partner[0].write({
                        'type': 'private',
                        'phone': self.partner_phone,
                        'mobile': self.partner_mobile,
                    })
                if new_partner[0].email_normalized:
                    email_domain = ('email_from', 'in', [new_partner[0].email, new_partner[0].email_normalized])
                else:
                    email_domain = ('email_from', '=', new_partner[0].email)
                self.search([
                    ('partner_id', '=', False), email_domain, ('stage_id.fold', '=', False)
                ]).write({'partner_id': new_partner[0].id})
        return super(Applicant, self)._message_post_after_hook(message, msg_vals)

    def create_employee_from_applicant(self):
        """ Create an hr.employee from the hr.applicants """
        employee = False
        for applicant in self:
            contact_name = False
            if applicant.partner_id:
                address_id = applicant.partner_id.address_get(['contact'])['contact']
                contact_name = applicant.partner_id.display_name
            else:
                if not applicant.partner_name:
                    raise UserError(_('You must define a Contact Name for this applicant.'))
                new_partner_id = self.env['res.partner'].create({
                    'is_company': False,
                    'type': 'private',
                    'name': applicant.partner_name,
                    'email': applicant.email_from,
                    'phone': applicant.partner_phone,
                    'mobile': applicant.partner_mobile
                })
                applicant.partner_id = new_partner_id
                address_id = new_partner_id.address_get(['contact'])['contact']
            if applicant.partner_name or contact_name:
                employee_data = {
                    'default_name': applicant.partner_name or contact_name,
                    'default_job_id': applicant.job_id.id,
                    'default_job_title': applicant.job_id.name,
                    'default_address_home_id': address_id,
                    'default_department_id': applicant.department_id.id or False,
                    'default_address_id': applicant.company_id and applicant.company_id.partner_id
                            and applicant.company_id.partner_id.id or False,
                    'default_work_email': applicant.department_id and applicant.department_id.company_id
                            and applicant.department_id.company_id.email or False,
                    'default_work_phone': applicant.department_id.company_id.phone,
                    'form_view_initial_mode': 'edit',
                    'default_applicant_id': applicant.ids,
                    }
                    
        dict_act_window = self.env['ir.actions.act_window']._for_xml_id('hr.open_view_employee_list')
        dict_act_window['context'] = employee_data
        return dict_act_window

    def archive_applicant(self):
        return {
            'type': 'ir.actions.act_window',
            'name': _('Refuse Reason'),
            'res_model': 'applicant.get.refuse.reason',
            'view_mode': 'form',
            'target': 'new',
            'context': {'default_applicant_ids': self.ids, 'active_test': False},
            'views': [[False, 'form']]
        }

    def reset_applicant(self):
        """ Reinsert the applicant into the recruitment pipe in the first stage"""
        default_stage = dict()
        for job_id in self.mapped('job_id'):
            default_stage[job_id.id] = self.env['hr.recruitment.stage'].search(
                ['|',
                    ('job_ids', '=', False),
                    ('job_ids', '=', job_id.id),
                    ('fold', '=', False)
                ], order='sequence asc', limit=1).id
        for applicant in self:
            applicant.write(
                {'stage_id': applicant.job_id.id and default_stage[applicant.job_id.id],
                 'refuse_reason_id': False})

    def toggle_active(self):
        res = super(Applicant, self).toggle_active()
        applicant_active = self.filtered(lambda applicant: applicant.active)
        if applicant_active:
            applicant_active.reset_applicant()
        applicant_inactive = self.filtered(lambda applicant: not applicant.active)
        if applicant_inactive:
            return applicant_inactive.archive_applicant()
        return res


class ApplicantCategory(models.Model):
    _name = "hr.applicant.category"
    _description = "Category of applicant"

    def _get_default_color(self):
        return randint(1, 11)

    name = fields.Char("Tag Name", required=True)
    color = fields.Integer(string='Color Index', default=_get_default_color)

    _sql_constraints = [
            ('name_uniq', 'unique (name)', "Tag name already exists !"),
    ]


class ApplicantRefuseReason(models.Model):
    _name = "hr.applicant.refuse.reason"
    _description = 'Refuse Reason of Applicant'

    name = fields.Char('Description', required=True, translate=True)
    active = fields.Boolean('Active', default=True)

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = ['res.config.settings']

    module_website_hr_recruitment = fields.Boolean(string='Online Posting')
    module_hr_recruitment_survey = fields.Boolean(string='Interview Forms')

```

## File: models\__init__.py

```python
from . import hr_department
from . import hr_recruitment
from . import hr_employee
from . import hr_job
from . import res_config_settings
from . import calendar
from . import digest

```

## File: security\hr_recruitment_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record model="ir.module.category" id="base.module_category_human_resources_recruitment">
        <field name="description">Helps you manage your recruitments.</field>
        <field name="sequence">11</field>
    </record>

    <record id="hr_applicant_comp_rule" model="ir.rule">
        <field name="name">Applicant multi company rule</field>
        <field name="model_id" ref="model_hr_applicant"/>
        <field eval="True" name="global"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="group_hr_recruitment_user" model="res.groups">
        <field name="name">Officer</field>
        <field name="category_id" ref="base.module_category_human_resources_recruitment"/>
        <field name="implied_ids" eval="[(4, ref('hr.group_hr_user'))]"/>
    </record>

    <record id="group_hr_recruitment_manager" model="res.groups">
        <field name="name">Administrator</field>
        <field name="category_id" ref="base.module_category_human_resources_recruitment"/>
        <field name="implied_ids" eval="[(4, ref('group_hr_recruitment_user'))]"/>
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
    </record>

    <record id="base.default_user" model="res.users">
        <field name="groups_id" eval="[(4,ref('hr_recruitment.group_hr_recruitment_manager'))]"/>
    </record>

</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_applicant_user,hr.applicant.user,model_hr_applicant,group_hr_recruitment_user,1,1,1,1
access_hr_recruitment_stage_user,hr.recruitment.stage.user,model_hr_recruitment_stage,group_hr_recruitment_user,1,0,0,0
access_hr_recruitment_stage_manager,hr.recruitment.stage.manager,model_hr_recruitment_stage,group_hr_recruitment_manager,1,1,1,1
access_hr_recruitment_degree,hr.recruitment.degree,model_hr_recruitment_degree,group_hr_recruitment_user,1,1,1,1
access_hr_recruitment_refuse_reason,hr.applicant.refuse.reason,model_hr_applicant_refuse_reason,group_hr_recruitment_user,1,1,1,1
access_res_partner_hr_user,res.partner.user,base.model_res_partner,group_hr_recruitment_user,1,1,1,1
access_calendar_event_hruser,calendar.event.hruser,calendar.model_calendar_event,group_hr_recruitment_user,1,1,1,1
access_hr_recruitment_source_hr_officer,hr.recruitment.source,model_hr_recruitment_source,group_hr_recruitment_user,1,1,1,1
access_hr_recruitment_source_all,hr.recruitment.source,model_hr_recruitment_source,,1,0,0,0
access_hr_applicant_category,hr.applicant_category,model_hr_applicant_category,,1,1,1,0
access_hr_applicant_category_manager,hr.applicant_category,model_hr_applicant_category,group_hr_recruitment_user,1,1,1,1
access_calendar_event_type_hr_officer,calendar.event.type.officer,calendar.model_calendar_event_type,group_hr_recruitment_user,1,1,1,0
access_applicant_get_refuse_reason,access.applicant.get.refuse.reason,model_applicant_get_refuse_reason,hr_recruitment.group_hr_recruitment_user,1,1,1,0

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70">
    <defs>
        <path id="icon-a" d="M4,5.35309892e-14 C36.4160122,9.87060235e-15 58.0836068,-3.97961823e-14 65,5.07020818e-14 C69,6.733808e-14 70,1 70,5 C70,43.0488877 70,62.4235458 70,65 C70,69 69,70 65,70 C61,70 9,70 4,70 C1,70 7.10542736e-15,69 7.10542736e-15,65 C7.25721566e-15,62.4676575 3.83358709e-14,41.8005206 3.60818146e-14,5 C-1.13686838e-13,1 1,5.75716207e-14 4,5.35309892e-14 Z"/>
        <linearGradient id="icon-c" x1="98.162%" x2="0%" y1="1.838%" y2="100%">
            <stop offset="0%" stop-color="#797DA5"/>
            <stop offset="50.799%" stop-color="#6D7194"/>
            <stop offset="100%" stop-color="#626584"/>
        </linearGradient>
        <path id="icon-d" d="M21.2876355,40.2488327 C21.644877,38.8191569 22.7635563,37.6632882 24.2438835,37.2932064 L27.2616862,36.5387663 C29.7246615,38.3103548 33.5677441,38.8192676 36.7383138,36.5387663 L39.7561165,37.2932064 C41.5646061,37.7453288 42.8333333,39.3702441 42.8333333,41.2344238 L42.8333333,41.2959901 C45.2976729,38.7278585 46.8104101,35.2404382 46.8104101,31.39867 C46.8104101,23.4931273 40.4116554,17.1061635 32.5165079,17.1061635 C24.6101932,17.1061635 18.2226057,23.5042934 18.2226057,31.39867 C18.2226057,34.7434734 19.3680481,37.8164357 21.2876355,40.2488327 Z M57.5996449,51.3156444 C58.3876447,52.1120396 58.3876447,53.3998274 57.5911718,54.1962225 L55.1932801,56.5938801 C54.4052803,57.3902752 53.1173667,57.3902752 52.3208938,56.5938801 L43.8731973,48.1470086 C43.4919071,47.7657556 43.2800792,47.248946 43.2800792,46.7067195 L43.2800792,45.3257365 C40.2890694,47.6640881 36.5270059,49.0535434 32.434491,49.0535434 C22.6988809,49.0535434 14.8104101,41.1658429 14.8104101,31.4311835 C14.8104101,21.6965241 22.6988809,13.8088235 32.434491,13.8088235 C42.1701011,13.8088235 50.0585719,21.6965241 50.0585719,31.4311835 C50.0585719,35.5232988 48.6689809,39.2849949 46.3304009,42.2757127 L47.7115188,42.2757127 C48.2537982,42.2757127 48.7706583,42.4875199 49.1519485,42.8687729 L57.5996449,51.3156444 Z M32,23.1666667 C35.7394466,23.1666667 38.7708333,26.1980534 38.7708333,29.9375 C38.7708333,33.6769466 35.7394466,36.7083333 32,36.7083333 C28.2605534,36.7083333 25.2291667,33.6769466 25.2291667,29.9375 C25.2291667,26.1980534 28.2605534,23.1666667 32,23.1666667 Z"/>
        <path id="icon-e" d="M21.2876355,38.2488327 C21.644877,36.8191569 22.7635563,35.6632882 24.2438835,35.2932064 L27.2616862,34.5387663 C29.7246615,36.3103548 33.5677441,36.8192676 36.7383138,34.5387663 L39.7561165,35.2932064 C41.5646061,35.7453288 42.8333333,37.3702441 42.8333333,39.2344238 L42.8333333,39.2959901 C45.2976729,36.7278585 46.8104101,33.2404382 46.8104101,29.39867 C46.8104101,21.4931273 40.4116554,15.1061635 32.5165079,15.1061635 C24.6101932,15.1061635 18.2226057,21.5042934 18.2226057,29.39867 C18.2226057,32.7434734 19.3680481,35.8164357 21.2876355,38.2488327 Z M57.5996449,49.3156444 C58.3876447,50.1120396 58.3876447,51.3998274 57.5911718,52.1962225 L55.1932801,54.5938801 C54.4052803,55.3902752 53.1173667,55.3902752 52.3208938,54.5938801 L43.8731973,46.1470086 C43.4919071,45.7657556 43.2800792,45.248946 43.2800792,44.7067195 L43.2800792,43.3257365 C40.2890694,45.6640881 36.5270059,47.0535434 32.434491,47.0535434 C22.6988809,47.0535434 14.8104101,39.1658429 14.8104101,29.4311835 C14.8104101,19.6965241 22.6988809,11.8088235 32.434491,11.8088235 C42.1701011,11.8088235 50.0585719,19.6965241 50.0585719,29.4311835 C50.0585719,33.5232988 48.6689809,37.2849949 46.3304009,40.2757127 L47.7115188,40.2757127 C48.2537982,40.2757127 48.7706583,40.4875199 49.1519485,40.8687729 L57.5996449,49.3156444 Z M32,21.1666667 C35.7394466,21.1666667 38.7708333,24.1980534 38.7708333,27.9375 C38.7708333,31.6769466 35.7394466,34.7083333 32,34.7083333 C28.2605534,34.7083333 25.2291667,31.6769466 25.2291667,27.9375 C25.2291667,24.1980534 28.2605534,21.1666667 32,21.1666667 Z"/>
    </defs>
    <g fill="none" fill-rule="evenodd">
        <mask id="icon-b" fill="#fff">
            <use xlink:href="#icon-a"/>
        </mask>
        <g mask="url(#icon-b)">
            <rect width="70" height="70" fill="url(#icon-c)"/>
            <path fill="#FFF" fill-opacity=".383" d="M4,1.8 L65,1.8 C67.6666667,1.8 69.3333333,1.13333333 70,-0.2 C70,2.46666667 70,3.46666667 70,2.8 L1.10547097e-14,2.8 C-1.65952376e-14,3.46666667 -2.9161925e-14,2.46666667 -2.66453526e-14,-0.2 C0.666666667,1.13333333 2,1.8 4,1.8 Z" transform="matrix(1 0 0 -1 0 2.8)"/>
            <path fill="#393939" d="M44.7894737,57 L4,57 C2,57 -7.10542736e-15,56.8519481 0,52.8545455 L2.23548517e-16,28.151787 L18.9764771,5.20700378 L30.93373,0.994855357 L42.9934954,4.14545455 L48.0359957,15.0446694 L44.7894737,26.8973595 L57.8747796,38.6825776 L44.7894737,57 Z" opacity=".324" transform="translate(0 13)"/>
            <path fill="#000" fill-opacity=".383" d="M4,4 L65,4 C67.6666667,4 69.3333333,3 70,1 C70,3.66666667 70,5 70,5 L1.77635684e-15,5 C1.77635684e-15,5 1.77635684e-15,3.66666667 1.77635684e-15,1 C0.666666667,3 2,4 4,4 Z" transform="translate(0 65)"/>
            <use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#icon-d"/>
            <use fill="#FFF" fill-rule="nonzero" xlink:href="#icon-e"/>
        </g>
    </g>
</svg>

```

## File: static\src\js\recruitment.js

```javascript
odoo.define('job.update_kanban', function (require) {
    'use strict';
    var KanbanRecord = require('web.KanbanRecord');

    KanbanRecord.include({
        /**
         * @override
         * @private
         */
        _openRecord: function () {
            if (this.modelName === 'hr.job' && this.$(".o_hr_job_boxes a").length) {
                this.$(".o_hr_job_boxes a").first().click();
            } else {
                this._super.apply(this, arguments);
            }
        }
    });
});

```

## File: static\src\js\tours\hr_recruitment.js

```javascript
odoo.define('hr_recruitment.tour', function(require) {
"use strict";

var core = require('web.core');
var tour = require('web_tour.tour');

var _t = core._t;

tour.register('hr_recruitment_tour',{
    url: "/web",
    rainbowManMessage: _t("<div>Great job! You hired a new colleague!</div><div>Try the Website app to publish job offers online.</div>"),
    sequence: 230,
}, [tour.stepUtils.showAppsMenuItem(), {
    trigger: '.o_app[data-menu-xmlid="hr_recruitment.menu_hr_recruitment_root"]',
    content: _t("Let's have a look at how to <b>improve</b> your <b>hiring process</b>."),
    position: 'right',
    edition: 'community'
}, {
    trigger: '.o_app[data-menu-xmlid="hr_recruitment.menu_hr_recruitment_root"]',
    content: _t("Let's have a look at how to <b>improve</b> your <b>hiring process</b>."),
    position: 'bottom',
    edition: 'enterprise'
}, {
    trigger: ".o-kanban-button-new",
    content: _t("Create your first Job Position."),
    position: "bottom",
    width: 195
}, {
    trigger: ".o_job_name",
    extra_trigger: '.o_hr_job_simple_form',
    content: _t("What do you want to recruit today? Choose a job title..."),
    position: "right"
}, {
    trigger: ".o_job_alias",
    extra_trigger: '.o_hr_job_simple_form',
    content: _t("Choose an application email."),
    position: "right"
}, {
    trigger: '.o_create_job',
    content: _t('Let\'s create the position. An email will be setup for applications, and a public job description, if you use the Website app.'),
    position: 'bottom',
    run: function (actions) {
        actions.auto('.modal:visible .btn.btn-primary');
    },
}, {
    trigger: ".oe_kanban_action_button",
    extra_trigger: '.o_hr_recruitment_kanban',
    content: _t("Let\'s have a look at the applications pipeline."),
    position: "bottom"
}, {
    trigger: ".o_copy_paste_email",
    content: _t("Copy this email address, to paste it in your email composer, to apply."),
    position: "bottom"
}, {
    trigger: ".breadcrumb-item:not(.active):last",
    extra_trigger: '.o_kanban_applicant',
    content: _t("Let’s go back to the dashboard."),
    position: "bottom"
}, {
    trigger: ".oe_kanban_action_button",
    extra_trigger: '.o_hr_recruitment_kanban',
    content: _t("<b>Did you apply by sending an email?</b> Check incoming applications."),
    position: "bottom"
}, {
    trigger: ".oe_kanban_card",
    extra_trigger: '.o_kanban_applicant',
    content: _t("<b>Drag this card</b>, to qualify him for a first interview."),
    position: "bottom",
    run: "drag_and_drop .o_kanban_group:eq(1) ",
}, {
    trigger: ".oe_kanban_card",
    extra_trigger: '.o_kanban_applicant',
    content: _t("<b>Click to view</b> the application."),
    position: "bottom"
}, {
    trigger: ".o_Chatter .o_ChatterTopbar_buttonSendMessage",
    extra_trigger: '.o_applicant_form',
    content: _t("<div><b>Try to send an email</b> to the applicant.</div><div><i>Tips: All emails sent or received are saved in the history here</i>"),
    position: "bottom"
}, {
    trigger: ".o_Chatter .o_Composer_buttonSend",
    extra_trigger: '.o_applicant_form',
    content: _t("Send your email. Followers will get a copy of the communication."),
    position: "bottom"
}, {
    trigger: ".o_Chatter .o_ChatterTopbar_buttonLogNote",
    extra_trigger: '.o_applicant_form',
    content: _t("Or talk about this applicant privately with your colleagues."),
    position: "bottom"
}, {
    trigger: ".o_create_employee",
    extra_trigger: '.o_applicant_form',
    content: _t("Let’s create this new employee now."),
    position: "bottom"
}, {
    trigger: ".o_form_button_save",
    extra_trigger: ".o_employee_form",
    content: _t("Save it !"),
    position: "bottom",
    width: 80
}]);

});

```

## File: views\digest_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="digest_digest_view_form" model="ir.ui.view">
        <field name="name">digest.digest.view.form.inherit.hr.recruitment</field>
        <field name="model">digest.digest</field>
        <field name="inherit_id" ref="digest.digest_digest_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//group[@name='kpi_general']" position="after">
                <group name="kpi_hr" string="Recruitment" groups="hr_recruitment.group_hr_recruitment_user">
                    <field name="kpi_hr_recruitment_new_colleagues"/>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\hr_department_views.xml

```xml
<odoo>
    <!--Hr Department Inherit Kanban view-->
    <record id="hr_department_view_kanban" model="ir.ui.view">
        <field name="name">hr.department.kanban.inherit</field>
        <field name="model">hr.department</field>
        <field name="inherit_id" ref="hr.hr_department_view_kanban"/>
        <field name="groups_id" eval="[(4,ref('hr_recruitment.group_hr_recruitment_user'))]"/>
        <field name="arch" type="xml">
            <data>
                <xpath expr="//templates" position="before">
                    <field name="new_applicant_count"/>
                    <field name="new_hired_employee"/>
                    <field name="expected_employee"/>
                </xpath>

                <xpath expr="//div[hasclass('o_kanban_primary_right')]" position="inside">
                    <div t-if="record.new_applicant_count.raw_value > 0" class="row ml16">
                        <div class="col-9">
                            <a name="%(hr_applicant_action_from_department)d" type="action">
                                New Applicants
                            </a>
                        </div>
                        <div class="col-3 text-right">
                            <field name="new_applicant_count"/>
                        </div>
                    </div>
                </xpath>

                <xpath expr="//div[hasclass('o_kanban_manage_reports')]" position="inside">
                    <a role="menuitem" class="dropdown-item" name="%(action_hr_recruitment_report_filtered_department)d" type="action">
                        Recruitments
                    </a>
                </xpath>
            </data>
        </field>
    </record>

    <record id="action_hr_department" model="ir.actions.act_window">
        <field name="name">Departments</field>
        <field name="res_model">hr.department</field>
        <field name="view_mode">tree,form</field>
    </record>

    <menuitem id="menu_hr_department" name="Departments"
            parent="menu_hr_recruitment_configuration" action="action_hr_department"/>
</odoo>

```

## File: views\hr_job_views.xml

```xml
<odoo>

    <record model="ir.actions.act_window" id="action_hr_job_new_application">
        <field name="name">New Application</field>
        <field name="res_model">hr.applicant</field>
        <field name="view_mode">form</field>
        <field name="context">{'search_default_job_id': [active_id], 'default_job_id': active_id}</field>
    </record>

    <record id="view_hr_job_kanban" model="ir.ui.view">
        <field name="name">hr.job.kanban</field>
        <field name="model">hr.job</field>
        <field name="arch" type="xml">
            <kanban class="oe_background_grey o_kanban_dashboard o_hr_recruitment_kanban" on_create="hr_recruitment.create_job_simple" sample="1">
                <field name="name"/>
                <field name="alias_name"/>
                <field name="alias_domain"/>
                <field name="is_favorite"/>
                <field name="department_id"/>
                <field name="no_of_recruitment"/>
                <field name="color"/>
                <field name="new_application_count"/>
                <field name="no_of_hired_employee"/>
                <field name="manager_id"/>
                <field name="state"/>
                <field name="user_id"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''}">
                            <div class="o_kanban_card_header">
                                <div class="o_kanban_card_header_title">
                                    <div class="o_primary col-12">
                                        <span class="o_text_overflow"><t t-esc="record.name.value"/></span>
                                    </div>
                                    <div class="o_kanban_record_subtitle col-12 text-muted">
                                        <field name="user_id" />
                                    </div>
                                    <field name="is_favorite" widget="boolean_favorite" nolabel="1"/>
                                    <div t-if="record.alias_name.value and record.alias_domain.value and record.state.raw_value == 'recruit'" class="o_secondary o_job_alias">
                                        <small> <i class="fa fa-envelope-o" role="img" aria-label="Alias" title="Alias"></i> <field name="alias_id"/> </small>
                                    </div>
                                </div>
                                <div class="o_kanban_manage_button_section">
                                    <a class="o_kanban_manage_toggle_button" href="#"><i class="fa fa-ellipsis-v" role="img" aria-label="Manage" title="Manage"/></a>
                                </div>
                            </div>
                            <div class="container o_kanban_card_content">
                                <t t-if="record.state.raw_value == 'recruit'">
                                    <div class="row">
                                        <div class="col-6">
                                            <button class="btn btn-primary" name="%(action_hr_job_applications)d" type="action">
                                                <field name="application_count"/> Applications
                                            </button>
                                        </div>
                                        <div class="col-6">
                                            <field name="new_application_count"/> New Applications <br/>
                                            <field name="no_of_recruitment"/> To Recruit
                                        </div>
                                    </div>
                                </t>
                                <t t-if="record.state.raw_value == 'open'">
                                    <div class="row">
                                        <div class="col-12 o_kanban_primary_left">
                                            <button class="btn btn-secondary" name="set_recruit" type="object">Start Recruitment</button>
                                        </div>
                                    </div>
                                </t>
                                <div name="kanban_boxes" class="row o_recruitment_kanban_boxes">
                                    <div class="o_recruitment_kanban_box o_kanban_primary_bottom bottom_block" style="padding-left:8px;">
                                        <div class="col-6"></div>
                                        <div class="col-6 o_link_trackers">
                                            <a role="button" name="%(hr_recruitment.action_hr_job_sources)d" type="action" class="btn btn-sm ">
                                                <span title="Link Trackers"><i class="fa fa-lg fa-envelope" role="img" aria-label="Link Trackers"/></span> 
                                            </a>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="o_kanban_card_manage_pane dropdown-menu" role="menu">
                                <div class="o_kanban_card_manage_section">
                                    <div role="menuitem"><a t-if="record.state.raw_value == 'recruit'" name="set_open" type="object">Recruitment Done</a></div>
                                    <div role="menuitem"><a t-if="record.state.raw_value == 'open'" name="set_recruit" type="object">Start recruitment</a></div>
                                    <div role="menuitem"><a t-if="widget.editable" name="edit_job" type="edit">Edit</a></div>
                                </div>
                                <div role="menuitem" aria-haspopup="true">
                                    <ul class="oe_kanban_colorpicker" data-field="color" role="menu"/>
                                </div>
                            </div>
                            <div class="o_hr_job_boxes">
                                <a class="o_hr_job_box" name="%(action_hr_job_applications)d" type="action"/>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="hr_job_search_view" model="ir.ui.view">
        <field name="name">hr.job.search</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr.view_job_filter" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='department_id']" position="after">
                <filter string="My Job Positions" name="my_positions" domain="[('user_id', '=', uid)]"/>
            </xpath>
        </field>
    </record>

    <!-- hr related job position menu action -->
    <record model="ir.actions.act_window" id="action_hr_job">
        <field name="name">Job Positions</field>
        <field name="res_model">hr.job</field>
        <field name="view_mode">kanban,form</field>
        <field name="view_id" ref="hr_recruitment.view_hr_job_kanban"/>
        <field name="context">{}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
              Ready to recruit more efficiently?
          </p><p>
              Let's create a job position.
          </p>
        </field>
    </record>
    <menuitem name="By Job Positions" parent="menu_crm_case_categ0_act_job" id="menu_hr_job_position" action="action_hr_job" sequence="1"/>
</odoo>

```

## File: views\hr_recruitment_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="assets_backend" name="hr_recruitment assets" inherit_id="web.assets_backend">
            <xpath expr="." position="inside">
                <link rel="stylesheet" type="text/scss" href="/hr_recruitment/static/src/scss/hr_job.scss"/>
                <script type="text/javascript" src="/hr_recruitment/static/src/js/recruitment.js"></script>
                <script type="text/javascript" src="/hr_recruitment/static/src/js/tours/hr_recruitment.js"></script>
            </xpath>
        </template>
        <template id="assets_tests" name="HR Recruitment Assets Tests" inherit_id="web.assets_tests">
            <xpath expr="." position="inside">
            </xpath>
        </template>
    </data>
</odoo>

```

## File: views\hr_recruitment_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>

    <!-- Stage -->
    <record id="hr_job_stage_act" model="ir.actions.act_window">
        <field name="name">Recruitment / Applicants Stages</field>
        <field name="res_model">hr.recruitment.stage</field>
        <field name="domain">[]</field>
        <field name="context">{}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Add a new stage in the recruitment process
          </p><p>
            Define here your stages of the recruitment process, for example:
            qualification call, first interview, second interview, refused,
            hired.
          </p>
        </field>
    </record>

    <!-- Applicants -->
    <record model="ir.ui.view" id="crm_case_tree_view_job">
        <field name="name">Applicants</field>
        <field name="model">hr.applicant</field>
        <field name="arch" type="xml">
            <tree string="Applicants" multi_edit="1" sample="1">
                <field name="message_needaction" invisible="1"/>
                <field name="last_stage_id" invisible="1"/>
                <field name="create_date" readonly="1" widget="date" optional="show"/>
                <field name="date_last_stage_update" invisible="1"/>
                <field name="partner_name" readonly="1"/>
                <field name="name" readonly="1"/>
                <field name="partner_mobile" widget="phone" readonly="1" optional="show"/>
                <field name="partner_phone" widget="phone" readonly="1" optional="hide"/>
                <field name="email_from" readonly="1" optional="hide"/>
                <field name="job_id"/>
                <field name="categ_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="show"/>
                <field name="priority" widget="priority" optional="show"/>
                <field name="medium_id" optional="hide"/>
                <field name="source_id" readonly="1" optional="hide"/>
                <field name="salary_expected" optional="hide"/>
                <field name="salary_proposed" optional="hide"/>
                <field name="type_id" invisible="1"/>
                <field name="availability" optional="hide"/>
                <field name="department_id" invisible="context.get('invisible_department', True)" readonly="1"/>
                <field name="user_id" widget="many2one_avatar_user" optional="show"/>
                <field name="company_id" groups="base.group_multi_company" readonly="1" optional="hide"/>
                <field name="stage_id"/>
            </tree>
        </field>
    </record>

    <record id="hr_applicant_view_tree_activity" model="ir.ui.view">
        <field name="name">hr.applicant.view.tree.activity</field>
        <field name="model">hr.applicant</field>
        <field name="arch" type="xml">
            <tree string="Next Activities" decoration-danger="activity_date_deadline &lt; current_date" default_order="activity_date_deadline">
                <field name="name"/>
                <field name="partner_id"/>
                <field name="activity_date_deadline"/>
                <field name="activity_type_id"/>
                <field name="activity_summary"/>
                <field name="stage_id"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </tree>
        </field>
    </record>

    <record model="ir.ui.view" id="hr_applicant_view_form">
        <field name="name">Jobs - Recruitment Form</field>
        <field name="model">hr.applicant</field>
        <field name="arch" type="xml">
          <form string="Jobs - Recruitment Form" class="o_applicant_form">
            <header>
                <button string="Create Employee" name="create_employee_from_applicant" type="object"
                        class="oe_highlight o_create_employee" attrs="{'invisible': ['|',('emp_id', '!=', False),('active', '=', False)]}"/>
                <button string="Refuse" name="archive_applicant" type="object" attrs="{'invisible': [('active', '=', False)]}"/>
                <button string="Restore" name="toggle_active" type="object" attrs="{'invisible': [('active', '=', True)]}"/>
                <field name="stage_id" widget="statusbar" options="{'clickable': '1', 'fold_field': 'fold'}" attrs="{'invisible': [('active', '=', False),('emp_id', '=', False)]}"/>
            </header>
            <sheet>
                <div class="oe_button_box" name="button_box">
                    <button name="action_applications_email"
                            class="oe_stat_button"
                            icon="fa-pencil"
                            type="object"
                            context="{'active_test': False}"
                            attrs="{'invisible': [('application_count', '=', 0)]}">
                        <field name="application_count" widget="statinfo" string="Other Applications"/>
                    </button>
                    <button name="action_makeMeeting" class="oe_stat_button" icon="fa-calendar" type="object"
                         help="Schedule interview with this applicant">
                        <field name="meeting_count" widget="statinfo" string="Meetings"/>
                    </button>
                </div>
                <widget name="web_ribbon" title="Refused" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                <field name="kanban_state" widget="kanban_state_selection"/>
                <field name="active" invisible="1"/>
                <field name="legend_normal" invisible="1"/>
                <field name="legend_blocked" invisible="1"/>
                <field name="legend_done" invisible="1"/>
                <div class="oe_title">
                    <label for="name" class="oe_edit_only"/>
                    <h1><field name="name"/></h1>
                    <h2 class="o_row">
                        <div>
                            <label for="partner_name" class="oe_edit_only"/>
                            <field name="partner_name"/>
                        </div>
                    </h2>
                </div>
                <group>
                    <group>
                        <field name="partner_id" invisible="1" />
                        <field name="refuse_reason_id" attrs="{'invisible': [('active', '=', True)]}"/>
                        <field name="email_from" widget="email"/>
                        <field name="email_cc" groups="base.group_no_one"/>
                        <field name="partner_phone" widget="phone"/>
                        <field name="partner_mobile" widget="phone"/>
                        <field name="type_id" placeholder="Degree"/>
                    </group>
                    <group>
                        <field name="categ_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                        <field name="user_id" domain="[('share', '=', False)]"/>
                        <field name="priority" widget="priority"/>
                        <field name="medium_id" groups="base.group_no_one" />
                        <field name="source_id"/>
                    </group>
                    <group string="Job">
                        <field name="job_id"/>
                        <field name="department_id"/>
                        <field name="company_id" groups="base.group_multi_company" options='{"no_open":True}' />
                    </group>
                    <group string="Contract">
                        <label for="salary_expected"/>
                        <div class="o_row">
                            <field name="salary_expected"/>
                            <span attrs="{'invisible':[('salary_expected_extra','=',False)]}"> + </span>
                            <field name="salary_expected_extra" placeholder="Extra advantages..."/>
                        </div>
                        <label for="salary_proposed"/>
                        <div class="o_row">
                            <field name="salary_proposed"/>
                            <span attrs="{'invisible':[('salary_proposed_extra','=',False)]}"> + </span>
                            <field name="salary_proposed_extra" placeholder="Extra advantages..."/>
                        </div>
                        <field name="availability"/>
                        <field name="emp_id" invisible="1"/>
                    </group>
                </group>
                <separator string="Application Summary"/>
                <field name="description" placeholder="Feedback of interviews..."/>
            </sheet>
            <div class="oe_chatter">
                <field name="message_follower_ids"/>
                <field name="activity_ids"/>
                <field name="message_ids" options="{'open_attachments': True}"/>
            </div>
          </form>
        </field>
    </record>

    <record model="ir.ui.view" id="crm_case_pivot_view_job">
        <field name="name">Jobs - Recruitment</field>
        <field name="model">hr.applicant</field>
        <field name="arch" type="xml">
              <pivot string="Job Applications" sample="1">
                <field name="create_date" type="row"/>
                <field name="stage_id" type="col"/>
                <field name="color" invisible="1"/>
            </pivot>
        </field>
    </record>

    <record model="ir.ui.view" id="crm_case_graph_view_job">
        <field name="name">Jobs - Recruitment Graph</field>
        <field name="model">hr.applicant</field>
        <field name="arch" type="xml">
              <graph string="Cases By Stage and Estimates" type="bar" orientation="vertical" stacked="True" sample="1">
                <field name="stage_id" type="col"/>
            </graph>
        </field>
    </record>

    <record id="hr_applicant_view_search_bis" model="ir.ui.view">
        <field name="name">hr.applicant.view.search</field>
        <field name="model">hr.applicant</field>
        <field name="arch" type="xml">
            <search string="Search Applicants">
                <field string="Applicant" name="partner_name"
                    filter_domain="['|', '|', ('name', 'ilike', self), ('partner_name', 'ilike', self), ('email_from', 'ilike', self)]"/>
                <field string="Email" name="email_from" filter_domain="[('email_from', 'ilike', self)]"/>
                <field name="job_id"/>
                <field name="department_id" operator="child_of"/>
                <field name="user_id"/>
                <field name="stage_id" domain="[]"/>
                <field name="categ_ids"/>
                <field name="refuse_reason_id"/>
                <field name="attachment_ids" filter_domain="[('attachment_ids.index_content', 'ilike', self)]" string="Attachments"/>
                <filter string="My Applications" name="my_applications" domain="[('user_id', '=', uid)]"/>
                <filter string="Unassigned" name="unassigned" domain="[('user_id', '=', False)]"/>
                <separator/>
                <filter string="Ready for Next Stage" name="done" domain="[('kanban_state', '=', 'done')]"/>
                <filter string="Blocked" name="blocked" domain="[('kanban_state', '=', 'blocked')]"/>
                <separator/>
                <filter string="Creation Date" name="filter_create" date="create_date"/>
                <filter string="Last Stage Update" name="filter_date_last_stage_update" date="date_last_stage_update"/>
                <separator/>
                <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction', '=', True)]"/>
                <separator/>
                <filter string="Archived / Refused" name="inactive" domain="[('active', '=', False)]"/>
                <separator/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                <separator/>
                <group expand="0" string="Group By">
                    <filter string="Responsible" name="responsible" domain="[]"  context="{'group_by': 'user_id'}"/>
                    <filter string="Job" name="job" domain="[]" context="{'group_by': 'job_id'}"/>
                    <filter string="Degree" name="degree" domain="[]" context="{'group_by': 'type_id'}"/>
                    <filter string="Stage" name="stage" domain="[]" context="{'group_by': 'stage_id'}"/>
                    <filter string="Refuse Reason" name="refuse_reason_id" domain="[]" context="{'group_by': 'refuse_reason_id'}"/>
                    <filter string="Creation Date" name="creation_date" context="{'group_by': 'create_date'}"/>
                    <filter string="Last Stage Update" name="last_stage_update" context="{'group_by': 'date_last_stage_update'}"/>
                </group>
           </search>
        </field>
    </record>

     <record id="hr_recruitment_source_view_search" model="ir.ui.view">
        <field name="name">hr.recruitment.source.view.search</field>
        <field name="model">hr.recruitment.source</field>
        <field name="arch" type="xml">
            <search string="Search Source">
                <field name="source_id"/>
                <field name="job_id"/>
           </search>
        </field>
    </record>

    <record model="ir.ui.view" id="hr_applicant_calendar_view">
        <field name="name">Hr Applicants Calendar</field>
        <field name="model">hr.applicant</field>
        <field name="priority" eval="2"/>
        <field name="arch" type="xml">
            <calendar string="Applicants" mode="month" date_start="activity_date_deadline" color="user_id" event_limit="5" hide_time="true">
                <field name="partner_name"/>
                <field name="job_id"/>
                <field name="priority" widget="priority"/>
                <field name="activity_summary"/>
                <field name="user_id" filters="1" invisible="1"/>
            </calendar>
        </field>
    </record>

    <record id="quick_create_applicant_form" model="ir.ui.view">
        <field name="name">hr.applicant.form.quick_create</field>
        <field name="model">hr.applicant</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <form>
                <group>
                    <field name="name"/>
                    <field name="partner_name"/>
                    <field name="email_from"/>
                    <field name="job_id" options="{'no_open': True}"/>
                    <field name="company_id" invisible="1"/>
                </group>
            </form>
        </field>
    </record>

    <!-- Hr Applicant Kanban View -->
    <record model="ir.ui.view" id="hr_kanban_view_applicant">
        <field name="name">Hr Applicants kanban</field>
        <field name="model">hr.applicant</field>
        <field name="arch" type="xml">
            <kanban default_group_by="stage_id" class="o_kanban_applicant" quick_create_view="hr_recruitment.quick_create_applicant_form" sample="1">
                <field name="stage_id" options='{"group_by_tooltip": {"requirements": "Requirements"}}'/>
                <field name="color"/>
                <field name="priority"/>
                <field name="user_id"/>
                <field name="user_email"/>
                <field name="partner_name"/>
                <field name="type_id"/>
                <field name="partner_id"/>
                <field name="job_id"/>
                <field name="department_id"/>
                <field name="attachment_number"/>
                <field name="active"/>
                <field name="activity_ids" />
                <field name="activity_state" />
                <progressbar field="activity_state" colors='{"planned": "success", "overdue": "danger", "today": "warning"}'/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="{{!selection_mode ? 'oe_kanban_color_' + kanban_getcolor(record.color.raw_value) : ''}} oe_kanban_card oe_kanban_global_click oe_applicant_kanban oe_semantic_html_override">
                            <span class="badge badge-pill badge-danger pull-right mr-4" attrs="{'invisible': [('active', '=', True)]}">Refused</span>
                            <div class="o_dropdown_kanban dropdown">
                                <a class="dropdown-toggle o-no-caret btn" role="button" data-toggle="dropdown" href="#" aria-label="Dropdown menu" title="Dropdown menu" data-display="static">
                                    <span class="fa fa-ellipsis-v"/>
                                </a>
                                <div class="dropdown-menu" role="menu">
                                    <t t-if="widget.deletable"><a role="menuitem" type="delete" class="dropdown-item">Delete</a></t>
                                    <a role="menuitem" name="action_makeMeeting" type="object" class="dropdown-item">Schedule Interview</a>
                                    <div role="separator" class="dropdown-divider"></div>
                                    <ul class="oe_kanban_colorpicker text-center" data-field="color"/>
                                </div>
                            </div>
                            <div class="oe_kanban_content">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings">
                                        <b class="o_kanban_record_title mt8" t-if="record.partner_name.raw_value">
                                            <field name="partner_name"/><br/>
                                        </b><t t-else="1">
                                            <i class="o_kanban_record_title"><field name="name"/></i><br/>
                                        </t>
                                        <div class="o_kanban_record_subtitle" invisible="context.get('search_default_job_id', False)">
                                            <field name="job_id"/>
                                        </div>
                                    </div>
                                </div>
                                <field name="categ_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                <t t-if="record.partner_mobile.raw_value"><i class="fa fa-mobile mr4" role="img" aria-label="Mobile" title="Mobile"/><field name="partner_mobile" widget="phone"/><br/></t>
                                <div class="o_kanban_record_bottom mt4">
                                    <div class="oe_kanban_bottom_left">
                                        <div class="float-left mr4" groups="base.group_user">
                                            <field name="priority" widget="priority"/>
                                        </div>
                                        <div class="o_kanban_inline_block mr8">
                                            <field name="activity_ids" widget="kanban_activity"/>
                                        </div>
                                    </div>
                                    <div class="oe_kanban_bottom_right">
                                        <a name="action_get_attachment_tree_view" type="object">
                                            <span title='Documents'><i class='fa fa-paperclip' role="img" aria-label="Documents"/>
                                                <t t-esc="record.attachment_number.raw_value"/>
                                            </span>
                                        </a>
                                        <div class="o_kanban_state_with_padding ml-1 mr-2" >
                                            <field name="kanban_state" widget="kanban_state_selection"/>
                                            <field name="legend_normal" invisible="1"/>
                                            <field name="legend_blocked" invisible="1"/>
                                            <field name="legend_done" invisible="1"/>
                                        </div>
                                        <field name="user_id" widget="many2one_avatar_user"/>
                                    </div>

                                </div>
                            </div>
                            <div class="oe_clear"></div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="hr_applicant_view_activity" model="ir.ui.view">
        <field name="name">hr.applicant.activity</field>
        <field name="model">hr.applicant</field>
        <field name="arch" type="xml">
            <activity string="Applicants">
                <templates>
                    <div t-name="activity-box">
                        <div>
                            <field name="name" display="full"/>
                            <field name="partner_name" muted="1" display="full"/>
                        </div>
                    </div>
                </templates>
            </activity>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_hr_job_applications">
        <field name="name">Applications</field>
        <field name="res_model">hr.applicant</field>
        <field name="view_mode">kanban,tree,form,graph,calendar,pivot</field>
        <field name="search_view_id" ref="hr_applicant_view_search_bis"/>
        <field name="context">{'search_default_job_id': [active_id], 'default_job_id': active_id}</field>
        <field name="help" type="html">
              <p class="o_view_nocontent_empty_folder">
                No applications yet
              </p><p>
                Odoo helps you track applicants in the recruitment
                process and follow up all operations: meetings, interviews, etc.
              </p><p>
                Applicants and their attached CV are created automatically when an email is sent.
                If you install the document management modules, all resumes are indexed automatically,
                so that you can easily search through their content.
              </p>
         </field>
    </record>

    <record model="ir.actions.act_window" id="action_hr_job_sources">
        <field name="name">Jobs Sources</field>
        <field name="res_model">hr.recruitment.source</field>
        <field name="view_mode">tree</field>
        <field name="search_view_id" ref="hr_recruitment_source_view_search"/>
        <field name="context">{'search_default_job_id': [active_id], 'default_job_id': active_id}</field>
        <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Want to analyse where applications come from ?
              </p><p>
                Use emails and links trackers
              </p>
         </field>
    </record>

    <!-- Jobs -->
    <record id="view_job_filter_recruitment" model="ir.ui.view">
        <field name="name">Job</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr.view_job_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='in_position']" position="before">
                <filter string="My Favorites" name="my_favorite_jobs" domain="[('favorite_user_ids', 'in', uid)]"/>
                <separator/>
            </xpath>
        </field>
    </record>

    <record id="hr_job_simple_form" model="ir.ui.view">
        <field name="name">hr.job.simple.form</field>
        <field name="model">hr.job</field>
        <field name="priority">200</field>
        <field name="arch" type="xml">
            <form string="Create a Job Position" class="o_hr_job_simple_form" >
                <sheet>
                    <group>
                        <field name="name" class="o_job_name oe_inline" placeholder="e.g. Sales Manager"/>
                        <label for="alias_name" string="Application email" attrs="{'invisible': [('alias_domain', '=', False)]}" help="Define a specific contact address for this job position. If you keep it empty, the default email address will be used which is in human resources settings"/>
                        <div name="alias_def" attrs="{'invisible': [('alias_domain', '=', False)]}">
                            <field name="alias_id" class="oe_read_only" string="Email Alias" required="0"/>
                            <div class="oe_edit_only" name="edit_alias">
                                <field name="alias_name" class="oe_inline o_job_alias" placeholder="e.g. sales-manager"/>@<field name="alias_domain" class="oe_inline" readonly="1"/>
                            </div>
                            <div class="text-muted" attrs="{'invisible': [('alias_domain', '=', False)]}">Applicants can send resume to this email address,<br/>it will create an application automatically</div>
                        </div>
                    </group>
                    <footer>
                        <button string="Create" name="close_dialog" type="object" class="btn-primary o_create_job"/>
                        <button string="Discard" class="btn-secondary" special="cancel"/>
                    </footer>
                </sheet>
            </form>
        </field>
    </record>
    <record id="create_job_simple" model="ir.actions.act_window">
        <field name="name">Create a Job Position</field>
        <field name="res_model">hr.job</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="hr_job_simple_form"/>
        <field name="target">new</field>
    </record>


    <record id="hr_job_survey" model="ir.ui.view">
        <field name="name">hr.job.form1</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr.view_hr_job_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='department_id']" position="after">
                <field name="address_id" context="{'show_address': 1}" domain= "[('is_company', '=', True )]" options="{'always_reload': True}"/>
                <label for="alias_name" string="Email Alias" attrs="{'invisible': [('alias_domain', '=', False)]}" help="Define a specific contact address for this job position. If you keep it empty, the default email address will be used which is in human resources settings"/>
                <div name="alias_def" attrs="{'invisible': [('alias_domain', '=', False)]}">
                    <field name="alias_id" class="oe_read_only" string="Email Alias" required="0"/>
                    <div class="oe_edit_only" name="edit_alias">
                        <field name="alias_name" class="oe_inline"/>@<field name="alias_domain" class="oe_inline" readonly="1"/>
                    </div>
                </div>
            </xpath>
            <field name="no_of_recruitment" position="after">
                <field name="user_id" domain="[('share', '=', False)]"/>
            </field>
            <div name="button_box" position="inside">
                <button class="oe_stat_button"
                    icon="fa-pencil"
                    name="%(action_hr_job_applications)d"
                    context="{'default_user_id': user_id, 'active_test': False}"
                    type="action">
                    <field name="all_application_count" widget="statinfo" string="Applications"/>
                </button>
                <button class="oe_stat_button"
                    icon="fa-file-text-o"
                    name="action_get_attachment_tree_view"
                    type="object">
                    <field name="documents_count" widget="statinfo" string="Documents"/>
                </button>
                <button class="oe_stat_button" type="action"
                    name="%(action_hr_job_sources)d" icon="fa-bar-chart-o"
                    context="{'default_job_id': active_id}">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_text">Trackers</span>
                    </div>
                </button>
            </div>
        </field>
    </record>

        <!-- hr related job position menu action -->
         <record model="ir.actions.act_window" id="action_hr_job_config">
            <field name="name">Job Positions</field>
            <field name="res_model">hr.job</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="view_ids" eval="[(5, 0, 0),
                (0, 0, {'view_mode': 'tree', 'view_id': ref('hr.view_hr_job_tree')}),
                (0, 0, {'view_mode': 'kanban', 'view_id': ref('hr.hr_job_view_kanban')}),
                (0, 0, {'view_mode': 'form', 'view_id': ref('hr.view_hr_job_form')})]"/>
            <field name="context">{'search_default_in_recruitment': 1}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Ready to recruit more efficiently?
              </p><p>
                Let's create a job position.
              </p>
            </field>
        </record>

     <!--
        hr.applicant.refuse.reason views
    -->
    <record id="hr_applicant_refuse_reason_view_form" model="ir.ui.view">
        <field name="name">Applicant refuse reason form</field>
        <field name="model">hr.applicant.refuse.reason</field>
        <field name="arch" type="xml">
            <form string="Refuse Reason">
                <sheet>
                    <widget name="web_ribbon" text="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title">
                        <div class="oe_edit_only">
                            <label for="name"/>
                        </div>
                        <h1>
                            <field name="name"/>
                        </h1>
                        <field name="active" invisible="1"/>
                    </div>
                </sheet>
            </form>
        </field>
    </record>

    <record id="hr_applicant_refuse_reason_view_tree" model="ir.ui.view">
        <field name="name">Applicant refuse reason tree</field>
        <field name="model">hr.applicant.refuse.reason</field>
        <field name="arch" type="xml">
            <tree string="Refuse Reason" editable="bottom">
                <field name="name"/>
            </tree>
        </field>
    </record>

    <record id="hr_applicant_refuse_reason_action" model="ir.actions.act_window">
        <field name="name">Refuse Reasons</field>
        <field name="res_model">hr.applicant.refuse.reason</field>
        <field name="view_mode">tree,form</field>
    </record>

    ######################## JOB OPPORTUNITIES (menu) ###########################
    <record model="ir.actions.act_window" id="crm_case_categ0_act_job">
        <field name="name">Applications</field>
        <field name="res_model">hr.applicant</field>
        <field name="view_mode">kanban,tree,form,pivot,graph,calendar,activity</field>
        <field name="view_id" eval="False"/>
        <field name="search_view_id" ref="hr_applicant_view_search_bis"/>
        <field name="context">{}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_empty_folder">
            No applications yet
          </p><p>
            Odoo helps you track applicants in the recruitment
            process and follow up all operations: meetings, interviews, etc.
          </p><p>
            Applicants and their attached CV are created automatically when an email is sent.
            If you install the document management modules, all resumes are indexed automatically,
            so that you can easily search through their content.
          </p>
        </field>
    </record>

    <record model="ir.actions.act_window.view" id="action_hr_sec_kanban_view_act_job">
        <field name="sequence" eval="0"/>
        <field name="view_mode">kanban</field>
        <field name="view_id" ref="hr_kanban_view_applicant"/>
        <field name="act_window_id" ref="crm_case_categ0_act_job"/>
    </record>

    <record model="ir.actions.act_window.view" id="action_hr_sec_tree_view_act_job">
        <field name="sequence" eval="1"/>
        <field name="view_mode">tree</field>
        <field name="view_id" ref="crm_case_tree_view_job"/>
        <field name="act_window_id" ref="crm_case_categ0_act_job"/>
    </record>

    <record model="ir.actions.act_window.view" id="action_hr_sec_form_view_act_job">
        <field name="sequence" eval="2"/>
        <field name="view_mode">form</field>
        <field name="view_id" ref="hr_applicant_view_form"/>
        <field name="act_window_id" ref="crm_case_categ0_act_job"/>
    </record>

    <record  id="hr_applicant_action_view_pivot" model="ir.actions.act_window.view">
        <field name="sequence" eval="3"/>
        <field name="view_mode">pivot</field>
        <field name="view_id" ref="crm_case_pivot_view_job"/>
        <field name="act_window_id" ref="crm_case_categ0_act_job"/>
    </record>

    <record id="action_hr_sec_graph_view_act_job" model="ir.actions.act_window.view">
        <field name="sequence" eval="4"/>
        <field name="view_mode">graph</field>
        <field name="view_id" ref="crm_case_graph_view_job"/>
        <field name="act_window_id" ref="crm_case_categ0_act_job"/>
    </record>

    <menuitem
        name="Recruitment"
        id="menu_hr_recruitment_root"
        web_icon="hr_recruitment,static/description/icon.png"
        groups="hr_recruitment.group_hr_recruitment_user"
        sequence="80"/>

    <menuitem id="menu_hr_recruitment_configuration" name="Configuration" parent="menu_hr_recruitment_root"
        sequence="100"/>

    <!-- ALL JOBS REQUESTS -->
    <menuitem parent="menu_hr_recruitment_configuration" id="menu_hr_job_position_config" action="action_hr_job_config" sequence="10"/>

    <menuitem
        id="menu_hr_applicant_refuse_reason"
        action="hr_applicant_refuse_reason_action"
        parent="menu_hr_recruitment_configuration"
        sequence="10"/>

    <menuitem
        name="Applications"
        parent="menu_hr_recruitment_root"
        id="menu_crm_case_categ0_act_job" sequence="2"/>

    <menuitem
        name="All Applications"
        parent="menu_crm_case_categ0_act_job"
        id="menu_crm_case_categ_all_app" action="crm_case_categ0_act_job" sequence="2"/>

    <!-- Resume and Letters -->
    <record id="ir_attachment_view_search_inherit_hr_recruitment" model="ir.ui.view">
        <field name="name">ir.attachment.search.inherit.recruitment</field>
        <field name="model">ir.attachment</field>
        <field name="mode">primary</field>
        <field name="inherit_id" ref="base.view_attachment_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='create_date']" position="after">
                <field name="index_content" string="Content"/>
            </xpath>
            <xpath expr="//filter[@name='my_documents_filter']" position="attributes">
                <attribute name='invisible'>1</attribute>
            </xpath>
            <xpath expr="//filter[@name='url_filter']" position="attributes">
                <attribute name='invisible'>1</attribute>
            </xpath>
            <xpath expr="//filter[@name='binary_filter']" position="attributes">
                <attribute name='invisible'>1</attribute>
            </xpath>
        </field>
    </record>

    <record model="ir.actions.server" id="hr_applicant_resumes_server">
        <field name="name">hr.applicant.resumes.server</field>
        <field name="model_id" ref="hr_recruitment.model_hr_applicant"/>
        <field name="state">code</field>
        <field name="code">
act = env.ref('hr_recruitment.hr_applicant_resumes').read()[0]
act['domain'] = [('res_model', '=', 'hr.applicant'), '|', ('company_id', '=', False), ('company_id', '=', env.user.company_id.id)]
action = act
        </field>
    </record>


    <!-- Stage Tree View -->
    <record model="ir.ui.view" id="hr_recruitment_stage_tree">
        <field name="name">hr.recruitment.stage.tree</field>
        <field name="model">hr.recruitment.stage</field>
        <field name="arch" type="xml">
            <tree string="Stages">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="fold"/>
            </tree>
        </field>
    </record>

    <!-- Stage Kanban View -->
    <record id="view_hr_recruitment_stage_kanban" model="ir.ui.view">
        <field name="name">hr.recruitment.stage.kanban</field>
        <field name="model">hr.recruitment.stage</field>
        <field name="arch" type="xml">
            <kanban>
                <field name="name"/>
                <field name="fold"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div>
                                <strong><field name="name"/></strong>
                            </div>
                            <div>
                                <span>Folded in Recruitment Pipe: </span>
                                <field name="fold" widget="boolean"/>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <!-- Stage Form View -->
    <record model="ir.ui.view" id="hr_recruitment_stage_form">
        <field name="name">hr.recruitment.stage.form</field>
        <field name="model">hr.recruitment.stage</field>
        <field name="arch" type="xml">
            <form string="Stage">
            <sheet>
                <group name="stage_definition" string="Stage Definition">
                    <group>
                        <field name="name"/>
                        <field name="sequence" groups="base.group_no_one"/>
                        <field name="template_id" domain= "[('model_id.model', '=', 'hr.applicant')]"/>
                    </group>
                    <group name="stage_details">
                        <field name="fold"/>
                        <field name="job_ids" widget="many2many_tags"/>
                    </group>
                </group>
                <group name="tooltips" string="Tooltips">
                    <p class="text-muted" colspan="2">
                        You can define here the labels that will be displayed for the kanban state instead
                        of the default labels.
                    </p>
                    <label for="legend_normal" string=" " class="o_status"/>
                    <field name="legend_normal" nolabel="1"/>
                    <label for="legend_blocked" string=" " class="o_status o_status_red"/>
                    <field name="legend_blocked" nolabel="1"/>
                    <label for="legend_done" string=" " class="o_status o_status_green"/>
                    <field name="legend_done" nolabel="1"/>
                </group>
                <separator string="Requirements"/>
                <field name="requirements"/>
            </sheet>
            </form>
        </field>
    </record>

    <!-- Stage Action -->
    <record id="hr_recruitment_stage_act" model="ir.actions.act_window">
        <field name="name">Stages</field>
        <field name="res_model">hr.recruitment.stage</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="view_id" ref="hr_recruitment_stage_tree"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Add a new stage in the recruitment process
          </p><p>
            Don't forget to specify the department if your recruitment process
            is different according to the job position.
          </p>
        </field>
    </record>

    <menuitem
        id="menu_hr_recruitment_stage"
        name="Stages"
        parent="menu_hr_recruitment_configuration"
        action="hr_recruitment_stage_act"
        groups="base.group_no_one"
        sequence="1"/>

    <!-- Tag Form View -->
    <record id="hr_applicant_category_view_form" model="ir.ui.view">
        <field name="name">hr.applicant.category.form</field>
        <field name="model">hr.applicant.category</field>
        <field name="arch" type="xml">
            <form string="Tags">
            <sheet>
                <group>
                    <field name="name"/>
                    <field name="color"/>
                </group>
            </sheet>
            </form>
        </field>
    </record>

    <record id="hr_applicant_category_view_tree" model="ir.ui.view">
        <field name="name">hr.applicant.category.tree</field>
        <field name="model">hr.applicant.category</field>
        <field name="arch" type="xml">
            <tree string="Tags" editable="bottom">
                <field name="name"/>
                <field name="color" groups="base.group_no_one"/>
            </tree>
        </field>
    </record>

    <!-- Tag Action -->
    <record id="hr_applicant_category_action" model="ir.actions.act_window">
        <field name="name">Tags</field>
        <field name="res_model">hr.applicant.category</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new tag
            </p>
        </field>
    </record>

    <menuitem
        id="hr_applicant_category_menu"
        parent="menu_hr_recruitment_configuration"
        action="hr_applicant_category_action"
        sequence="2" groups="base.group_no_one"/>

    <!-- Degree Tree View -->
    <record model="ir.ui.view" id="hr_recruitment_degree_tree">
        <field name="name">hr.recruitment.degree.tree</field>
        <field name="model">hr.recruitment.degree</field>
        <field name="arch" type="xml">
            <tree string="Degree" editable="bottom">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
            </tree>
        </field>
    </record>

    <!-- Degree Form View -->
    <record model="ir.ui.view" id="hr_recruitment_degree_form">
        <field name="name">hr.recruitment.degree.form</field>
        <field name="model">hr.recruitment.degree</field>
        <field name="arch" type="xml">
            <form string="Degree">
            <sheet>
                <group>
                    <field name="name"/>
                    <field name="sequence" groups="base.group_no_one"/>
                </group>
            </sheet>
            </form>
        </field>
    </record>

    <!-- Degree Action -->
    <record id="hr_recruitment_degree_action" model="ir.actions.act_window">
        <field name="name">Degree</field>
        <field name="res_model">hr.recruitment.degree</field>
        <field name="view_id" ref="hr_recruitment_degree_tree"/>
    </record>

     <menuitem
            id="menu_hr_recruitment_degree"
            name="Degrees"
            parent="menu_hr_recruitment_configuration"
            action="hr_recruitment_degree_action"
            sequence="5" groups="base.group_no_one"/>

    <!-- Source Tree View -->
    <record model="ir.ui.view" id="hr_recruitment_source_tree">
        <field name="name">hr.recruitment.source.tree</field>
        <field name="model">hr.recruitment.source</field>
        <field name="arch" type="xml">
            <tree string="Sources of Applicants" editable="top" class="o_recruitment_list" sample="1">
                <field name="source_id" placeholder="e.g. LinkedIn" decoration-bf="1" attrs="{'readonly': [('id', '!=', False)]}"/>
                <field name="job_id" attrs="{'readonly': [('id', '!=', False)]}"/>
                <field name="email" attrs="{'invisible': [('email', '=', False)]}" widget="email"/>
                <button name="create_alias" string="Generate Email" class="btn btn-primary" type="object" attrs="{'invisible': [('email', '!=', False)]}"/>
            </tree>
        </field>
    </record>
    <record id="hr_recruitment_source_action" model="ir.actions.act_window">
        <field name="name">Sources of Applicants</field>
        <field name="res_model">hr.recruitment.source</field>
    </record>

    <menuitem
        id="menu_hr_recruitment_source"
        parent="menu_hr_recruitment_configuration"
        action="hr_recruitment_source_action"
        groups="base.group_no_one"
        sequence="10"/>

    <record id="hr_applicant_action_from_department" model="ir.actions.act_window">
        <field name="name">New Applications</field>
        <field name="res_model">hr.applicant</field>
        <field name="view_mode">kanban,tree,form,graph,calendar,pivot</field>
        <field name="context">{
            'search_default_department_id': active_id,
            'default_department_id': active_id}
        </field>
        <field name="domain">[('stage_id.sequence','&lt;=','1')]</field>
    </record>

    <!--Hr Employee inherit search view-->
    <record id="hr_employee_view_search" model="ir.ui.view">
        <field name="name">hr.employee.search.inherit</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='job_id']" position="after">
                <filter name="newly_hired_employee" string="Newly Hired" domain="[('newly_hired_employee', '=', True)]" groups="hr_recruitment.group_hr_recruitment_user"/>
            </xpath>
        </field>
    </record>

    <record id="hr_employee_action_from_department" model="ir.actions.act_window">
        <field name="name">Newly Hired Employees</field>
        <field name="res_model">hr.employee</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="context">{
            'search_default_newly_hired_employee': 1,
            'search_default_department_id': [active_id],
            'default_department_id': active_id}
        </field>
        <field name="search_view_id" ref="hr_employee_view_search"/>
    </record>

    <record id="hr_applicant_view_pivot" model="ir.ui.view">
         <field name="name">hr.applicant.pivot</field>
         <field name="model">hr.applicant</field>
         <field name="arch" type="xml">
             <pivot string="Recruitment Analysis" sample="1">
                 <field name="stage_id" type="row"/>
                 <field name="job_id" type="col"/>
             </pivot>
         </field>
    </record>

    <record id="hr_applicant_view_graph" model="ir.ui.view">
         <field name="name">hr.applicant.graph</field>
         <field name="model">hr.applicant</field>
         <field name="arch" type="xml">
             <graph string="Recruitment Analysis" sample="1">
                 <field name="stage_id" type="row"/>
                 <field name="job_id" type="col"/>
             </graph>
         </field>
    </record>

    <record id="hr_applicant_view_search" model="ir.ui.view">
        <field name="name">hr.applicant.search</field>
        <field name="model">hr.applicant</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <search string="Recruitment Analysis">
                <field name="job_id"/>
                <field name="department_id" operator="child_of"/>
                <field name="user_id"/>
                <filter string="Creation Date" name="year" date="create_date" default_period="this_year"/>
                <separator/>
                <filter string="Unassigned" name="unassigned" domain="[('user_id', '=', False)]"/>
                <separator/>
                <filter string="New" name="new" domain="[('stage_id.sequence', '=', 1)]"/>
                <separator/>
                <filter string="Ongoing" name="ongoing" domain="[('active', '=', True)]"/>
                <filter string="Refused" name="refused" domain="[('active', '=', False)]"/>
                <separator/>
                <filter string="Archived" name="archived" domain="[('active', '=', False)]"/>
                <separator/>
                <group expand="0" string="Extended Filters">
                    <field name="priority"/>
                    <field name="stage_id"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="create_date"/>
                    <field name="date_closed"/>
                </group>
                <group expand="1" string="Group By">
                   <filter string="Responsible" name='User' context="{'group_by':'user_id'}"/>
                   <filter string="Company" name="company" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                   <filter string="Jobs" name="job" context="{'group_by':'job_id'}"/>
                   <filter string="Department" name="department" context="{'group_by':'department_id'}"/>
                   <filter string="Stage" name="stage" context="{'group_by':'stage_id'}" />
                   <separator/>
                   <filter string="Creation Date" name="creation_month" context="{'group_by':'create_date:month'}" help="Creation Date"/>
                </group>
            </search>
        </field>
    </record>

    <record id="hr_applicant_action_analysis" model="ir.actions.act_window">
        <field name="name">Recruitment Analysis</field>
        <field name="res_model">hr.applicant</field>
        <field name="view_mode">graph,pivot</field>
        <field name="search_view_id" ref="hr_applicant_view_search"/>
        <field name="view_ids" eval="[
            (5, 0, 0),
            (0, 0, {'view_mode': 'graph', 'view_id': ref('hr_applicant_view_graph')}),
            (0, 0, {'view_mode': 'pivot', 'view_id': ref('hr_applicant_view_pivot')})]"/>
        <field name="context">{'search_default_creation_month': 1, 'search_default_job': 2}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No data yet!
            </p>
        </field>
    </record>
    <menuitem name="Reporting" id="report_hr_recruitment" parent="menu_hr_recruitment_root"
    sequence="99"/>

    <menuitem name="Recruitment Analysis" id="hr_applicant_report_menu" parent="report_hr_recruitment"
    sequence="50" action="hr_applicant_action_analysis"/>
    <record id="action_hr_recruitment_report_filtered_department" model="ir.actions.act_window">
        <field name="name">Recruitment Analysis</field>
        <field name="res_model">hr.applicant</field>
        <field name="view_mode">graph,pivot</field>
        <field name="search_view_id" ref="hr_applicant_view_search"/>
        <field name="context">{
            'search_default_department_id': [active_id],
            'default_department_id': active_id}
        </field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No data yet!
            </p>
        </field>
    </record>

    <record id="action_hr_recruitment_report_filtered_job" model="ir.actions.act_window">
        <field name="name">Recruitment Analysis</field>
        <field name="res_model">hr.applicant</field>
        <field name="view_mode">graph,pivot</field>
        <field name="search_view_id" ref="hr_applicant_view_search"/>
        <field name="context">{
            'search_default_job_id': [active_id],
            'default_job_id': active_id}
        </field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No data yet!
            </p>
        </field>
    </record>

    <!-- Custom reports (aka filters) -->
    <record id="hr_applicant_filter_recruiter" model="ir.filters">
        <field name="name">By Recruiter</field>
        <field name="model_id">hr.applicant</field>
        <field name="user_id" eval="False"/>
        <field name="action_id" ref="hr_applicant_action_analysis"/>
        <field name="context">{'group_by': ['create_date:month', 'user_id']}</field>
    </record>
    <record id="hr_applicant_filter_job" model="ir.filters">
        <field name="name">By Job</field>
        <field name="model_id">hr.applicant</field>
        <field name="user_id" eval="False"/>
        <field name="action_id" ref="hr_applicant_action_analysis"/>
        <field name="context">{'group_by': ['create_date:month', 'job_id']}</field>
    </record>
    <record id="hr_applicant_filter_department" model="ir.filters">
        <field name="name">By Department</field>
        <field name="model_id">hr.applicant</field>
        <field name="user_id" eval="False"/>
        <field name="action_id" ref="hr_applicant_action_analysis"/>
        <field name="context">{'group_by': ['create_date:month', 'department_id']}</field>
    </record>

    </data>
</odoo>

```

## File: views\mail_activity_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <!-- Activity types config -->
    <record id="mail_activity_type_action_config_hr_applicant" model="ir.actions.act_window">
        <field name="name">Activity Types</field>
        <field name="res_model">mail.activity.type</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">['|', ('res_model_id', '=', False), ('res_model_id.model', '=', 'hr.applicant')]</field>
        <field name="context">{'default_res_model': 'hr.applicant'}</field>
    </record>
    <menuitem id="hr_recruitment_menu_config_activity_type"
        action="mail_activity_type_action_config_hr_applicant"
        parent="menu_hr_recruitment_configuration"/>
</odoo>
```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.hr.recruitment</field>
            <field name="model">res.config.settings</field>
            <field name="priority" eval="75"/>
            <field name="inherit_id" ref="base.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//div[hasclass('settings')]" position="inside">
                    <div class="app_settings_block" data-string="Recruitment" string="Recruitment" data-key="hr_recruitment" groups="hr_recruitment.group_hr_recruitment_manager">
                        <h2>Job Posting</h2>
                        <div class="row mt16 o_settings_container" name="online_posting_setting_container">
                            <div class="col-12 col-lg-6 o_setting_box" id="publish_available_jobs_setting">
                                <div class="o_setting_left_pane">
                                    <field name="module_website_hr_recruitment"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_website_hr_recruitment" string="Online Posting"/>
                                    <div class="text-muted">
                                        Publish available jobs on your website
                                    </div>
                                </div>
                            </div>
                        </div>
                        <h2>Recruitment Process</h2>
                        <div class="row mt16 o_settings_container" name="recruitment_process_div">
                            <div class="col-12 col-lg-6 o_setting_box"
                                id="interview_forms_setting"
                                title="Use interview forms tailored to each job position during the recruitment process. Select the form to use in the job position detail form. This relies on the Survey app.">
                                <div class="o_setting_left_pane">
                                    <field name="module_hr_recruitment_survey"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_hr_recruitment_survey" string="Interview Forms"/>
                                    <div class="text-muted">
                                        Use interview forms during recruitment process
                                    </div>
                                    <div class="content-group" id="interview_forms"/>
                                </div>
                            </div>
                        </div>
                    </div>
                </xpath>
            </field>
        </record>

        <record id="action_hr_recruitment_configuration" model="ir.actions.act_window">
            <field name="name">Settings</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">res.config.settings</field>
            <field name="view_mode">form</field>
            <field name="target">inline</field>
            <field name="context">{'module' : 'hr_recruitment', 'bin_size': False}</field>
        </record>

        <menuitem id="menu_hr_recruitment_global_settings" name="Settings"
            parent="menu_hr_recruitment_configuration" sequence="0" action="action_hr_recruitment_configuration"
            groups="base.group_system"/>
    </data>
</odoo>

```

## File: wizard\applicant_refuse_reason.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models


class ApplicantGetRefuseReason(models.TransientModel):
    _name = 'applicant.get.refuse.reason'
    _description = 'Get Refuse Reason'

    refuse_reason_id = fields.Many2one('hr.applicant.refuse.reason', 'Refuse Reason')
    applicant_ids = fields.Many2many('hr.applicant')

    def action_refuse_reason_apply(self):
        return self.applicant_ids.write({'refuse_reason_id': self.refuse_reason_id.id, 'active': False})

```

## File: wizard\applicant_refuse_reason_views.xml

```xml
<?xml version="1.0"?>
<odoo>
        <record id="applicant_get_refuse_reason_view_form" model="ir.ui.view">
            <field name="name">applicant.get.refuse.reason.form</field>
            <field name="model">applicant.get.refuse.reason</field>
            <field name="arch" type="xml">
                <form string="Refuse Reason">
                    <group class="oe_title">
                        <field name="refuse_reason_id"/>
                        <field name="applicant_ids" invisible="1"/>
                    </group>
                    <footer>
                        <button name="action_refuse_reason_apply" string="Submit" type="object" class="btn-primary"/>
                        <button string="Cancel" class="btn-secondary" special="cancel"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="applicant_get_refuse_reason_action" model="ir.actions.act_window">
            <field name="name">Refuse Reason</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">applicant.get.refuse.reason</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="applicant_get_refuse_reason_view_form"/>
            <field name="target">new</field>
        </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import applicant_refuse_reason

```

