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
    'version': '1.1',
    'category': 'Human Resources/Recruitment',
    'sequence': 90,
    'summary': 'Track your recruitment pipeline',
    'website': 'https://www.odoo.com/app/recruitment',
    'depends': [
        'hr',
        'calendar',
        'utm',
        'attachment_indexation',
        'web_tour',
        'digest',
    ],
    'data': [
        'security/hr_recruitment_security.xml',
        'security/ir.model.access.csv',
        'data/digest_data.xml',
        'data/mail_message_subtype_data.xml',
        'data/mail_template_data.xml',
        'data/mail_templates.xml',
        'data/hr_recruitment_data.xml',
        'views/hr_recruitment_degree_views.xml',
        'views/hr_recruitment_source_views.xml',
        'views/hr_recruitment_stage_views.xml',
        'views/ir_attachment_views.xml',
        'views/hr_applicant_category_views.xml',
        'views/hr_applicant_refuse_reason_views.xml',
        'views/hr_applicant_views.xml',
        'views/res_config_settings_views.xml',
        'views/hr_department_views.xml',
        'views/hr_job_views.xml',
        'views/mail_activity_views.xml',
        'views/digest_views.xml',
        'wizard/applicant_refuse_reason_views.xml',
        'wizard/applicant_send_mail_views.xml',
    ],
    'demo': [
        'data/hr_recruitment_demo.xml',
    ],
    'installable': True,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'hr_recruitment/static/src/**/*.js',
            'hr_recruitment/static/src/**/*.scss',
            'hr_recruitment/static/src/**/*.xml',
            'hr_recruitment/static/src/js/tours/hr_recruitment.js',
        ],
    },
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
        <t t-set="record" t-value="object.env['hr.job'].search([('alias_name', '!=', False)], limit=1)" />
        <t t-if="record.alias_email">
            <a t-attf-href="mailto:{{record.alias_email}}" target="_blank" style="color: #714B67; text-decoration: none;">Try sending an email</a>
        </t>
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

    <record model="hr.recruitment.stage" id="stage_job0">
        <field name="name">New</field>
        <field name="sequence">0</field>
        <field name="template_id" ref="email_template_data_applicant_congratulations"/> 
    </record>
    <record model="hr.recruitment.stage" id="stage_job1">
        <field name="name">Initial Qualification</field>
        <field name="sequence">1</field>
    </record>
    <record model="hr.recruitment.stage" id="stage_job2">
        <field name="name">First Interview</field>
        <field name="sequence">2</field>
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
        <field name="hired_stage">True</field>
    </record>

    <!-- applicant refuse reason -->
        <record id="refuse_reason_1" model="hr.applicant.refuse.reason">
            <field name="name">Doesn't fit the job requirements</field>
            <field name="template_id" ref="email_template_data_applicant_refuse"/>
        </record>
        <record id="refuse_reason_2" model="hr.applicant.refuse.reason">
            <field name="name">Refused by Applicant: don't like job</field>
            <field name="template_id" ref="email_template_data_applicant_not_interested"/>
        </record>
        <record id="refuse_reason_3" model="hr.applicant.refuse.reason">
            <field name="name">Refused by Applicant: better offer</field>
            <field name="template_id" ref="email_template_data_applicant_not_interested"/>
        </record>
        <record id="refuse_reason_4" model="hr.applicant.refuse.reason">
            <field name="name">Language issues</field>
            <field name="template_id" ref="email_template_data_applicant_refuse"/>
        </record>
        <record id="refuse_reason_5" model="hr.applicant.refuse.reason">
            <field name="name">Role already fulfilled</field>
            <field name="template_id" ref="email_template_data_applicant_refuse"/>
        </record>
        <record id="refuse_reason_6" model="hr.applicant.refuse.reason">
            <field name="name">Duplicate</field>
            <field name="template_id" ref="email_template_data_applicant_refuse"/>
        </record>
        <record id="refuse_reason_7" model="hr.applicant.refuse.reason">
            <field name="name">Spam</field>
            <field name="template_id" ref="email_template_data_applicant_refuse"/>
        </record>
        <record id="refuse_reason_8" model="hr.applicant.refuse.reason">
            <field name="name">Refused by Applicant: salary</field>
            <field name="template_id" ref="email_template_data_applicant_not_interested"/>
        </record>

        <record model="ir.config_parameter" id="hr_recruitment_blacklisted_emails" forcecreate="False">
            <field name="key">hr_recruitment.blacklisted_emails</field>
            <field name="value"> </field>
        </record>

</data>
</odoo>

```

## File: data\hr_recruitment_demo.xml

```xml
<?xml version="1.0"?>
<odoo noupdate="1">
    <record id="base.user_demo" model="res.users">
        <field name="groups_id" eval="[(3, ref('hr_recruitment.group_hr_recruitment_manager'))]"/>
    </record>

    <!--Manage the job_id to get in hr.applicant-->
    <record id="hr.job_developer" model="hr.job">
        <field name="no_of_recruitment">4</field>
        <field name="no_of_hired_employee">56</field>
        <field name="user_id" ref="base.user_admin" />
    </record>
    <record id="hr.job_ceo" model="hr.job">
        <field name="no_of_hired_employee">1</field>
    </record>
    <record id="hr.job_cto" model="hr.job">
        <field name="no_of_hired_employee">1</field>
        <field name="user_id" ref="base.user_admin" />
    </record>
    <record id="hr.job_consultant" model="hr.job">
        <field name="no_of_recruitment">1</field>
        <field name="no_of_hired_employee">17</field>
        <field name="user_id" ref="base.user_demo" />
    </record>
    <record id="hr.job_hrm" model="hr.job">
        <field name="no_of_recruitment">1</field>
        <field name="no_of_hired_employee">5</field>
    </record>
    <record id="hr.job_marketing" model="hr.job">
        <field name="no_of_recruitment">3</field>
        <field name="no_of_hired_employee">2</field>
        <field name="user_id" ref="base.user_demo" />
    </record>
    <record id="hr.job_trainee" model="hr.job">
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
        <field name="email_from">enrique.jones152@gmail.example.com</field>
        <field name="partner_mobile">9963214587</field>
        <field name="stage_id" ref="stage_job2"/>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=29)"/>
        <field name="date_last_stage_update" eval="(DateTime.today() - timedelta(days=27)).strftime('%Y-%m-%d')"/>
        <field name="availability" eval="(DateTime.today() + relativedelta(months=3)).strftime('%Y-%m-%d')"/>
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
        <field name="email_from">meldona.thang@example.com</field>
        <field name="partner_mobile">998655451</field>
        <field name="stage_id" ref="stage_job1"/>
        <field name="availability" eval="(DateTime.today() + relativedelta(months=3)).strftime('%Y-%m-%d')"/>
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
        <field name="email_from">coincoin@gmail.example.com</field>
        <field name="partner_mobile">8955545</field>
        <field name="stage_id" ref="stage_job1"/>
        <field name="availability" eval="(DateTime.today() + timedelta(days=15)).strftime('%Y-%m-%d')"/>
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
        <field name="email_from">kelly@wallant.example.com</field>
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
        <field name="email_from">c-cile72@msn.example.com</field>
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
        <field name="email_from">0h3n-rizome@example.com</field>
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
        <field name="email_from">justinemarie@outlook.example.com</field>
        <field name="partner_mobile">9988774455</field>
        <field name="stage_id" ref="stage_job4"/>
        <field name="partner_phone">6633225</field>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=17)"/>
        <field name="date_last_stage_update" eval="(DateTime.today() - timedelta(days=7)).strftime('%Y-%m-%d')"/>
        <field name="availability" eval="(DateTime.today() + timedelta(days=15)).strftime('%Y-%m-%d')"/>
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
        <field name="email_from">the.jose@gmail.example.com</field>
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
        <field name="email_from">yin.lee@wechat.example.com</field>
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
        <field name="email_from">st-hubertus@gmail.example.com</field>
        <field name="priority">3</field>
        <field name="stage_id" ref="stage_job3"/>
        <field name="availability" eval="(DateTime.today() + timedelta(days=15)).strftime('%Y-%m-%d')"/>
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
        <field name="email_from">johnnyboy@gmail.example.com</field>
        <field name="stage_id" ref="stage_job5"/>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=61)"/>
        <field name="date_last_stage_update" eval="(DateTime.today() - timedelta(days=37)).strftime('%Y-%m-%d')"/>
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
        <field name="email_from">sandra.elvis.the.king25@gmail.example.com</field>
        <field name="stage_id" ref="stage_job5"/>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=34)"/>
        <field name="date_last_stage_update" eval="(DateTime.today() - timedelta(days=7)).strftime('%Y-%m-%d')"/>
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
        <field name="email_from">david.strongarm@gmail.example.com</field>
        <field name="stage_id" ref="stage_job2"/>
        <field name="partner_phone">33968745</field>
        <field name="availability" eval="(DateTime.today() + relativedelta(months=3)).strftime('%Y-%m-%d')"/>
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
        <field name="email_from">joren.jacob@outlook.example.com</field>
        <field name="stage_id" ref="stage_job2"/>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=7)"/>
        <field name="date_last_stage_update" eval="(DateTime.today() - timedelta(days=3)).strftime('%Y-%m-%d')"/>
        <field name="availability" eval="(DateTime.today() + timedelta(days=15)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="hr_case_traineemca1" model="hr.applicant">
        <field name="name">Trainee - MCA</field>
        <field name="job_id" ref="hr.job_trainee"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_sales')])]"/>
        <field name="partner_name">Tina Augustie</field>
        <field name="email_from">tina.turner@gmail.example.com</field>
        <field name="partner_mobile">9898745745</field>
        <field name="stage_id" ref="stage_job4"/>
        <field name="partner_phone">6630125</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=67)"/>
        <field name="date_last_stage_update" eval="(DateTime.today() - timedelta(days=45)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="hr_case_programmer" model="hr.applicant">
        <field name="name">Programmer</field>
        <field name="job_id" ref="hr.job_developer"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_it')])]"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_name">Shane Williams</field>
        <field name="email_from">the.real.shane@gmail.example.com</field>
        <field name="partner_mobile">9812398524</field>
        <field name="stage_id" ref="stage_job4"/>
        <field name="partner_phone">6630125</field>
        <field name="salary_expected">11000.0</field>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=13)"/>
        <field name="date_last_stage_update" eval="(DateTime.today() - timedelta(days=4)).strftime('%Y-%m-%d')"/>
        <field name="availability" eval="(DateTime.today() + relativedelta(months=3)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="hr_case_advertisement" model="hr.applicant">
        <field name="name">Advertisement</field>
        <field name="job_id" ref="hr.job_consultant"/>
        <field name="department_id" ref="hr.dep_ps"/>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_it')])]"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_name">David Billy</field>
        <field name="email_from">billy.boy12@gmail.example.com</field>
        <field name="partner_mobile">9988774455</field>
        <field name="stage_id" ref="stage_job2"/>
        <field name="salary_expected">11000.0</field>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=4)"/>
        <field name="date_last_stage_update" eval="(DateTime.today() - timedelta(days=2)).strftime('%Y-%m-%d')"/>
        <field name="availability" eval="(DateTime.today() + relativedelta(months=3)).strftime('%Y-%m-%d')"/>
    </record>

    <record id="hr_case_dev2_cv" model="ir.attachment">
        <field name="name">Cecile_Donth_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Cecile_Donth_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_dev2"/>
    </record>

    <record id="hr_case_financejob0_cv" model="ir.attachment">
        <field name="name">David_Armstrong_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/David_Armstrong_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_financejob0"/>
    </record>

    <record id="hr_case_advertisement_cv" model="ir.attachment">
        <field name="name">David_Billy_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/David_Billy_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_advertisement"/>
    </record>

    <record id="hr_case_salesman0_cv" model="ir.attachment">
        <field name="name">Enrique_Jones_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Enrique_Jones_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_salesman0"/>
    </record>

        <record id="hr_case_mkt1_cv" model="ir.attachment">
        <field name="name">Hubert_Blank_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Hubert_Blank_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_mkt1"/>
    </record>

    <record id="hr_case_dev0_cv" model="ir.attachment">
        <field name="name">Johan_Duck_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Johan_Duck_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_dev0"/>
    </record>

    <record id="hr_case_yrsexperienceinphp0_cv" model="ir.attachment">
        <field name="name">John_Bruno_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/John_Bruno_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_yrsexperienceinphp0"/>
    </record>

    <record id="hr_case_financejob1_cv" model="ir.attachment">
        <field name="name">Joren_Jacob_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Joren_Jacob_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_financejob1"/>
    </record>

    <record id="hr_case_fresher0_cv" model="ir.attachment">
        <field name="name">Jose_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Jose_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_fresher0"/>
    </record>

    <record id="hr_case_dev1_cv" model="ir.attachment">
        <field name="name">Kelly_Wallant_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Kelly_Wallant_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_dev1"/>
    </record>

    <record id="hr_case_traineemca0_cv" model="ir.attachment">
        <field name="name">Marie_Justine_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Marie_Justine_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_traineemca0"/>
    </record>

    <record id="hr_case_salesman1_cv" model="ir.attachment">
        <field name="name">Meldona_Thang_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Meldona_Thang_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_salesman1"/>
    </record>

    <record id="hr_case_dev3_cv" model="ir.attachment">
        <field name="name">Ohen_Rizome_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Ohen_Rizome_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_dev3"/>
    </record>

    <record id="hr_case_marketingjob0_cv" model="ir.attachment">
        <field name="name">Sandra_Elvis_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Sandra_Elvis_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_marketingjob0"/>
    </record>

    <record id="hr_case_programmer_cv" model="ir.attachment">
        <field name="name">Shane_Williams_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Shane_Williams_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_programmer"/>
    </record>

    <record id="hr_case_traineemca1_cv" model="ir.attachment">
        <field name="name">Tina_Augustie_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Tina_Augustie_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_traineemca1"/>
    </record>

    <record id="hr_case_mkt0_cv" model="ir.attachment">
        <field name="name">Yin_Lee_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Yin_Lee_CV.pdf"></field>
        <field name="res_model">hr.applicant</field>
        <field name="res_id" ref="hr_recruitment.hr_case_mkt0"/>
    </record>

    <!-- Set the main attachment to avoid automatic sending to the OCR-->
    <record id="hr_case_salesman0" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_salesman0_cv"/>
    </record>
    <record id="hr_case_salesman1" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_salesman1_cv"/>
    </record>
    <record id="hr_case_dev0" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_dev0_cv"/>
    </record>
    <record id="hr_case_dev1" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_dev1_cv"/>
    </record>
    <record id="hr_case_dev2" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_dev2_cv"/>
    </record>
    <record id="hr_case_dev3" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_dev3_cv"/>
    </record>
    <record id="hr_case_traineemca0" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_traineemca0_cv"/>
    </record>
    <record id="hr_case_fresher0" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_fresher0_cv"/>
    </record>
    <record id="hr_case_mkt0" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_mkt0_cv"/>
    </record>
    <record id="hr_case_mkt1" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_mkt1_cv"/>
    </record>
    <record id="hr_case_yrsexperienceinphp0" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_yrsexperienceinphp0_cv"/>
    </record>
    <record id="hr_case_marketingjob0" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_marketingjob0_cv"/>
    </record>
    <record id="hr_case_financejob0" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_financejob0_cv"/>
    </record>
    <record id="hr_case_financejob1" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_financejob1_cv"/>
    </record>
    <record id="hr_case_traineemca1" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_traineemca1_cv"/>
    </record>
    <record id="hr_case_programmer" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_programmer_cv"/>
    </record>
    <record id="hr_case_advertisement" model="hr.applicant">
        <field name="message_main_attachment_id" ref="hr_recruitment.hr_case_advertisement_cv"/>
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
        <field name="date" eval="DateTime.now() - relativedelta(days=3)"/>
        <field name="body" type="html">
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
        <field name="body" type="html">
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
        <field name="body" type="html">
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
        <field name="body" type="html">
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
        <field name="body" type="html">
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
        <field name="body" type="html">
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
        <field name="body" type="html">
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

## File: data\mail_message_subtype_data.xml

```xml
<?xml version="1.0"?>
<odoo>
<data noupdate="1">
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
    <record id="mt_job_applicant_new" model="mail.message.subtype">
        <field name="name">New Applicant</field>
        <field name="res_model">hr.job</field>
        <field name="default" eval="False"/>
        <field name="parent_id" ref="mt_applicant_new" />
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

</data></odoo>

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <template id="applicant_hired_template">
Employee created: <a href="#" t-att-data-oe-id="applicant.emp_id.id" data-oe-model="hr.employee"><t t-esc="applicant.emp_id.name"/></a>
    </template>

    <template id="mail_notification_light_without_background" inherit_id="mail.mail_notification_light">
        <xpath expr="//t//table[@role='presentation']" position="attributes">
            <attribute name="style" add="background-color: white;" separator=" "/>
        </xpath>
    </template>

</data></odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0"?>
<odoo><data noupdate="1">

    <!-- Templates for interest / refusing applicants -->
    <record id="email_template_data_applicant_refuse" model="mail.template">
        <field name="name">Recruitment: Refuse</field>
        <field name="model_id" ref="hr_recruitment.model_hr_applicant"/>
        <field name="subject">Your Job Application: {{ object.job_id.name }}</field>
        <field name="email_to">{{ (not object.partner_id and object.email_from or '') }}</field>
        <field name="partner_to">{{ object.partner_id.id or '' }}</field>
        <field name="description">When you refuse an application, you can choose this template</field>
        <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
    <tr>
        <td valign="top">
            <div style="font-size: 13px; margin: 0px; padding: 0px;">
                Hello,<br/><br/>
                Thank you for your interest in joining the
                <b><t t-out="object.company_id.name or ''">YourCompany</t></b> team.  We
                wanted to let you know that, although your resume is
                competitive, our hiring team reviewed your application
                and <b>did not select it for further consideration</b>.
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
                    <t t-if="object.user_id">
                        -- <br/>
                        <strong t-out="object.user_id.name or ''">Mitchell Admin</strong><br/>
                        Email: <t t-out="object.user_id.email or ''">admin@yourcompany.example.com</t><br/>
                        Phone: <t t-out="object.user_id.phone or ''">+1 650-123-4567</t>
                    </t>
                    <t t-else="">
                        -- <br/>
                        <t t-out="object.company_id.name or ''">YourCompany</t><br/>
                        The HR Team
                    </t>
                </div>
            </div>
        </td>
    </tr>
</table>
        </field>
        <field name="auto_delete" eval="True"/>
        <field name="lang">{{ object.partner_id.lang or '' }}</field>
    </record>

    <record id="email_template_data_applicant_interest" model="mail.template">
        <field name="name">Recruitment: Interest</field>
        <field name="model_id" ref="hr_recruitment.model_hr_applicant"/>
        <field name="subject">Your Job Application: {{ object.job_id.name }}</field>
        <field name="email_to">{{ (not object.partner_id and object.email_from or '') }}</field>
        <field name="partner_to">{{ object.partner_id.id or '' }}</field>
        <field name="description">Set this template to a recruitment stage to send it when applications reach that stage</field>
        <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="background-color: white; border-collapse: collapse; margin-left: 20px;">
    <tr>
        <td valign="top" style="padding: 0px 10px;">
            <div style="text-align: center">
                <h2>Congratulations!</h2>
                <div style="color:grey;">Your resume has been positively reviewed.</div>
            </div>
            <div style="font-size: 13px; margin: 0px; padding: 0px;">
                We just reviewed your resume, and it caught our
                attention. As we think you might be great for the
                position, your application has been short listed for a
                call or an interview.
                <br/><br/>
                <div t-if="'website_url' in object.job_id and object.job_id.website_url" style="padding: 16px 8px 16px 8px;">
                    <a t-att-href="object.job_id.website_url"
                        style="background-color: #875a7b; text-decoration: none; color: #fff; padding: 8px 16px 8px 16px; border-radius: 5px;">Job Description</a>
                </div>

                <t t-if="object.user_id">
                    You will soon be contacted by:<br/>
                    <strong t-out="object.user_id.name or ''">Mitchell Admin</strong><br/>
                    <span>Email: <t t-out="object.user_id.email or ''">admin@yourcompany.example.com</t></span><br/>
                    <span>Phone: <t t-out="object.user_id.phone or ''">+1 650-123-4567</t></span>
                    <br/><br/>
                </t>
                See you soon,
                <div style="font-size: 11px; color: grey;">
                    -- <br/>
                    The HR Team
                    <t t-if="'website_url' in object.job_id and hasattr(object.job_id, 'website_url') and object.job_id.website_url">
                        Discover <a href="/jobs" style="text-decoration:none;color:#717188;">all our jobs</a>.<br/>
                    </t>
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
                <t t-set="location" t-value="''"/>
                <t t-if="object.job_id.address_id.name">
                    <strong t-out="object.job_id.address_id.name or ''">Teksa SpA</strong><br/>
                </t>
                <t t-if="object.job_id.address_id.street">
                    <t t-out="object.job_id.address_id.street or ''">Puerto Madero 9710</t><br/>
                    <t t-set="location" t-value="object.job_id.address_id.street"/>
                </t>
                <t t-if="object.job_id.address_id.street2">
                    <t t-out="object.job_id.address_id.street2 or ''">Of A15, Santiago (RM)</t><br/>
                    <t t-set="location" t-value="'%s, %s' % (location, object.job_id.address_id.street2)"/>
                </t>
                <t t-if="object.job_id.address_id.city">
                    <t t-out="object.job_id.address_id.city or ''">Pudahuel</t>,
                    <t t-set="location" t-value="'%s, %s' % (location, object.job_id.address_id.city)"/>
                </t>
                <t t-if="object.job_id.address_id.state_id.name">
                    <t t-out="object.job_id.address_id.state_id.name or ''">C1</t>,
                    <t t-set="location" t-value="'%s, %s' % (location, object.job_id.address_id.state_id.name)"/>
                </t>
                <t t-if="object.job_id.address_id.zip">
                    <t t-out="object.job_id.address_id.zip or ''">98450</t>
                    <t t-set="location" t-value="'%s, %s' % (location, object.job_id.address_id.zip)"/>
                </t>
                <br/>
                <t t-if="object.job_id.address_id.country_id.name">
                    <t t-out="object.job_id.address_id.country_id.name or ''">Argentina</t><br/>
                    <t t-set="location" t-value="'%s, %s' % (location, object.job_id.address_id.country_id.name)"/>
                </t>
                <br/>
            </div>
        </td>
    </tr>
</table></field>
        <field name="auto_delete" eval="True"/>
        <field name="lang">{{ object.partner_id.lang or '' }}</field>
    </record>

    <record id="email_template_data_applicant_congratulations" model="mail.template">
        <field name="name">Recruitment: Application Acknowledgement</field>
        <field name="model_id" ref="hr_recruitment.model_hr_applicant"/>
        <field name="subject">Your Job Application: {{ object.job_id.name }}</field>
        <field name="email_to">{{ (not object.partner_id and object.email_from or '') }}</field>
        <field name="partner_to">{{ object.partner_id.id or '' }}</field>
        <field name="description">Confirmation email sent to all new job applications</field>
        <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="background-color: white; border-collapse: collapse; margin-left: 20px;">
    <tr>
        <td valign="top" style="padding: 0px 10px;">
            <div style="font-size: 13px; margin: 0px; padding: 0px;">
                Hello,
                <br/><br/>
                We confirm we successfully received your application for the job
                "<a t-att-href="hasattr(object.job_id, 'website_url') and object.job_id.website_url or ''" style="color:#9A6C8E;"><strong t-out="object.job_id.name or ''">Experienced Developer</strong></a>" at <strong t-out="object.company_id.name or ''">YourCompany</strong>.
                <br/><br/>
                We will come back to you shortly.

                <div t-if="'website_url' in object.job_id and object.job_id.website_url" style="padding: 16px 8px 16px 8px;">
                    <a t-att-href="object.job_id.website_url"
                        style="background-color: #875a7b; text-decoration: none; color: #fff; padding: 8px 16px 8px 16px; border-radius: 5px;">Job Description</a>
                </div>

                <hr width="97%" style="background-color: rgb(204,204,204); border: medium none; clear: both; display: block; font-size: 0px; min-height: 1px; line-height: 0; margin: 16px 0px 16px 0px;"/>
                <t t-if="object.user_id">
                    <h3 style="color:#9A6C8E;"><strong>Your Contact:</strong></h3>
                    <p>
                        <strong t-out="object.user_id.name or ''">Mitchell Admin</strong><br/>
                        <span>Email: <t t-out="object.user_id.email or ''">admin@yourcompany.example.com</t></span><br/>
                        <span>Phone: <t t-out="object.user_id.phone or ''">+1 650-123-4567</t></span>
                    </p>
                    <hr width="97%" style="background-color: rgb(204,204,204); border: medium none; clear: both; display: block; font-size: 0px; min-height: 1px; line-height: 0; margin: 16px 0px 16px 0px;"/>
                </t>

                <h3 style="color:#9A6C8E;"><strong>What is the next step?</strong></h3>
                We usually <strong>answer applications within a few days.</strong><br/><br/>
                Feel free to <strong>contact us if you want a faster
                feedback</strong> or if you don't get news from us
                quickly enough (just reply to this email).

                <hr width="97%" style="background-color: rgb(204,204,204); border: medium none; clear: both; display: block; font-size: 0px; min-height: 1px; line-height: 0; margin: 17px 0px 16px 0px;"/>
                <t t-set="location" t-value="''" />
                <t t-if="object.job_id.address_id.name">
                    <strong t-out="object.job_id.address_id.name or ''">Teksa SpA</strong><br/>
                </t>
                <t t-if="object.job_id.address_id.street">
                    <t t-out="object.job_id.address_id.street or ''">Puerto Madero 9710</t><br/>
                    <t t-set="location" t-value="object.job_id.address_id.street"/>
                </t>
                <t t-if="object.job_id.address_id.street2">
                    <t t-out="object.job_id.address_id.street2 or ''">Of A15, Santiago (RM)</t><br/>
                    <t t-set="location" t-value="'%s, %s' % (location, object.job_id.address_id.street2)"/>
                </t>
                <t t-if="object.job_id.address_id.city">
                    <t t-out="object.job_id.address_id.city or ''">Pudahuel</t>,
                    <t t-set="location" t-value="'%s, %s' % (location, object.job_id.address_id.city)"/>
                </t>
                <t t-if="object.job_id.address_id.state_id.name">
                    <t t-out="object.job_id.address_id.state_id.name or ''">C1</t>,
                    <t t-set="location" t-value="'%s, %s' % (location, object.job_id.address_id.state_id.name)"/>
                </t>
                <t t-if="object.job_id.address_id.zip">
                    <t t-out="object.job_id.address_id.zip or ''">98450</t>
                    <t t-set="location" t-value="'%s, %s' % (location, object.job_id.address_id.zip)"/>
                </t>
                <br/>
                <t t-if="object.job_id.address_id.country_id.name">
                    <t t-out="object.job_id.address_id.country_id.name or ''">Argentina</t><br/>
                    <t t-set="location" t-value="'%s, %s' % (location, object.job_id.address_id.country_id.name)"/>
                </t>
                <br/>
            </div>
        </td>
    </tr>
</table></field>
        <field name="auto_delete" eval="True"/>
        <field name="lang">{{ object.partner_id.lang or '' }}</field>
    </record>

    <record id="email_template_data_applicant_not_interested" model="mail.template">
        <field name="name">Recruitment: Not interested anymore</field>
        <field name="model_id" ref="hr_recruitment.model_hr_applicant"/>
        <field name="subject">Your Job Application: {{ object.job_id.name }}</field>
        <field name="email_to">{{ (not object.partner_id and object.email_from or '') }}</field>
        <field name="partner_to">{{ object.partner_id.id or '' }}</field>
        <field name="description">When you refuse an application, you can choose this template</field>
        <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
    <tr>
        <td valign="top">
            <div style="font-size: 13px; margin: 0px; padding: 0px;">
                Dear,<br/><br/>
                We would like to thank you for your interest and your time.<br/>
                We wish you all the best in your future endeavors.
                <br/><br/>
                Best<br/>
                <div style="font-size: 11px; color: grey;">
                    <t t-if="object.user_id">
                        -- <br/>
                        <strong t-out="object.user_id.name or ''">Marc Demo</strong><br/>
                        Email: <t t-out="object.user_id.email or ''">mark.brown23@example.com</t><br/>
                        Phone: <t t-out="object.user_id.phone or ''">+1 650-123-4567</t>
                    </t>
                    <t t-else="">
                        -- <br/>
                        <t t-out="object.company_id.name or ''">YourCompany</t><br/>
                        The HR Team<br/>
                    </t>
                </div>
            </div>
        </td>
    </tr>
</table>
        </field>
        <field name="auto_delete" eval="True"/>
        <field name="lang">{{ object.partner_id.lang or '' }}</field>
    </record>

</data></odoo>

```

## File: models\calendar.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.exceptions import AccessError


class CalendarEvent(models.Model):
    """ Model for Calendar Event """
    _inherit = 'calendar.event'

    @api.model_create_multi
    def create(self, vals_list):
        events = super(CalendarEvent, self).create(vals_list)
        try:
            self.env['hr.applicant'].check_access_rights('read')
        except AccessError:
            return events

        if "default_applicant_id" in self.env.context:
            applicant_attachments = self.env['hr.applicant'].browse(self.env.context['default_applicant_id']).attachment_ids
            for event in events:
                self.env['ir.attachment'].create([{
                    'name': att.name,
                    'type': 'binary',
                    'datas': att.datas,
                    'res_model': event._name,
                    'res_id': event.id
                } for att in applicant_attachments])
        return events

    @api.model
    def default_get(self, fields):
        if self.env.context.get('default_applicant_id'):
            self = self.with_context(
                default_res_model='hr.applicant', #res_model seems to be lost without this
                default_res_model_id=self.env.ref('hr_recruitment.model_hr_applicant').id,
                default_res_id=self.env.context.get('default_applicant_id'),
                default_partner_ids=self.env.context.get('default_partner_ids'),
                default_name=self.env.context.get('default_name')
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

    applicant_id = fields.Many2one('hr.applicant', string="Applicant", index='btree_not_null', ondelete='set null')

```

## File: models\digest.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.exceptions import AccessError


class Digest(models.Model):
    _inherit = 'digest.digest'

    kpi_hr_recruitment_new_colleagues = fields.Boolean('New Employees')
    kpi_hr_recruitment_new_colleagues_value = fields.Integer(compute='_compute_kpi_hr_recruitment_new_colleagues_value')

    def _compute_kpi_hr_recruitment_new_colleagues_value(self):
        if not self.env.user.has_group('hr_recruitment.group_hr_recruitment_user'):
            raise AccessError(_("Do not have access, skip this data for user's digest email"))

        self._calculate_company_based_kpi(
            'hr.employee',
            'kpi_hr_recruitment_new_colleagues_value',
        )

    def _compute_kpis_actions(self, company, user):
        res = super(Digest, self)._compute_kpis_actions(company, user)
        res['kpi_hr_recruitment_new_colleagues'] = 'hr.open_view_employee_list_my&menu_id=%s' % self.env.ref('hr.menu_hr_root').id
        return res

```

## File: models\hr_applicant.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from markupsafe import Markup

from odoo import api, fields, models, tools, SUPERUSER_ID
from odoo.exceptions import AccessError, UserError
from odoo.osv import expression
from odoo.tools import Query
from odoo.tools.translate import _

from dateutil.relativedelta import relativedelta

AVAILABLE_PRIORITIES = [
    ('0', 'Normal'),
    ('1', 'Good'),
    ('2', 'Very Good'),
    ('3', 'Excellent')
]


class Applicant(models.Model):
    _name = "hr.applicant"
    _description = "Applicant"
    _order = "priority desc, id desc"
    _inherit = ['mail.thread.cc',
               'mail.thread.main.attachment',
               'mail.thread.blacklist',
               'mail.thread.phone',
               'mail.activity.mixin',
               'utm.mixin']
    _mailing_enabled = True
    _primary_email = 'email_from'

    name = fields.Char("Subject / Application", required=True, help="Email subject for applications sent via email", index='trigram')
    active = fields.Boolean("Active", default=True, help="If the active field is set to false, it will allow you to hide the case without removing it.", index=True)
    description = fields.Html("Description")
    email_from = fields.Char("Email", size=128, compute='_compute_partner_phone_email',
        inverse='_inverse_partner_email', store=True, index='trigram')
    email_normalized = fields.Char(index='trigram')  # inherited via mail.thread.blacklist
    probability = fields.Float("Probability")
    partner_id = fields.Many2one('res.partner', "Contact", copy=False, index='btree_not_null')
    create_date = fields.Datetime("Applied on", readonly=True)
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
        'res.users', "Recruiter", compute='_compute_user', domain="[('share', '=', False), ('company_ids', 'in', company_id)]",
        tracking=True, store=True, readonly=False)
    date_closed = fields.Datetime("Hire Date", compute='_compute_date_closed', store=True, readonly=False, tracking=True, copy=False)
    date_open = fields.Datetime("Assigned", readonly=True)
    date_last_stage_update = fields.Datetime("Last Stage Update", index=True, default=fields.Datetime.now)
    priority = fields.Selection(AVAILABLE_PRIORITIES, "Evaluation", default='0')
    job_id = fields.Many2one('hr.job', "Applied Job", domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", tracking=True, index=True)
    salary_proposed_extra = fields.Char("Proposed Salary Extra", help="Salary Proposed by the Organisation, extra advantages", tracking=True, groups="hr_recruitment.group_hr_recruitment_user")
    salary_expected_extra = fields.Char("Expected Salary Extra", help="Salary Expected by Applicant, extra advantages", tracking=True, groups="hr_recruitment.group_hr_recruitment_user")
    salary_proposed = fields.Float("Proposed Salary", group_operator="avg", help="Salary Proposed by the Organisation", tracking=True, groups="hr_recruitment.group_hr_recruitment_user")
    salary_expected = fields.Float("Expected Salary", group_operator="avg", help="Salary Expected by Applicant", tracking=True, groups="hr_recruitment.group_hr_recruitment_user")
    availability = fields.Date("Availability", help="The date at which the applicant will be available to start working", tracking=True)
    partner_name = fields.Char("Applicant's Name")
    partner_phone = fields.Char("Phone", size=32, compute='_compute_partner_phone_email',
        store=True, readonly=False, index='btree_not_null', inverse='_inverse_partner_email')
    partner_phone_sanitized = fields.Char(string='Sanitized Phone Number', compute='_compute_partner_phone_sanitized', store=True, index='btree_not_null')
    partner_mobile = fields.Char("Mobile", size=32, compute='_compute_partner_phone_email',
        store=True, readonly=False, index='btree_not_null', inverse='_inverse_partner_email')
    partner_mobile_sanitized = fields.Char(string='Sanitized Mobile Number', compute='_compute_partner_mobile_sanitized', store=True, index='btree_not_null')
    type_id = fields.Many2one('hr.recruitment.degree', "Degree")
    department_id = fields.Many2one(
        'hr.department', "Department", compute='_compute_department', store=True, readonly=False,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", tracking=True)
    day_open = fields.Float(compute='_compute_day', string="Days to Open", compute_sudo=True)
    day_close = fields.Float(compute='_compute_day', string="Days to Close", compute_sudo=True)
    delay_close = fields.Float(compute="_compute_delay", string='Delay to Close', readonly=True, group_operator="avg", help="Number of days to close", store=True)
    color = fields.Integer("Color Index", default=0)
    emp_id = fields.Many2one('hr.employee', string="Employee", help="Employee linked to the applicant.", copy=False)
    emp_is_active = fields.Boolean(string="Employee Active", related='emp_id.active')
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
    application_count = fields.Integer(compute='_compute_application_count', help='Applications with the same email or phone or mobile')
    refuse_reason_id = fields.Many2one('hr.applicant.refuse.reason', string='Refuse Reason', tracking=True)
    meeting_ids = fields.One2many('calendar.event', 'applicant_id', 'Meetings')
    meeting_display_text = fields.Char(compute='_compute_meeting_display')
    meeting_display_date = fields.Date(compute='_compute_meeting_display')
    # UTMs - enforcing the fact that we want to 'set null' when relation is unlinked
    campaign_id = fields.Many2one(ondelete='set null')
    medium_id = fields.Many2one(ondelete='set null')
    source_id = fields.Many2one(ondelete='set null')
    interviewer_ids = fields.Many2many('res.users', 'hr_applicant_res_users_interviewers_rel',
        string='Interviewers', index=True, tracking=True, copy=False,
        domain="[('share', '=', False), ('company_ids', 'in', company_id)]")
    linkedin_profile = fields.Char('LinkedIn Profile')
    application_status = fields.Selection([
        ('ongoing', 'Ongoing'),
        ('hired', 'Hired'),
        ('refused', 'Refused'),
        ('archived', 'Archived'),
    ], compute="_compute_application_status")
    applicant_properties = fields.Properties('Properties', definition='job_id.applicant_properties_definition', copy=True)

    def init(self):
        super().init()
        self.env.cr.execute("""
            CREATE INDEX IF NOT EXISTS hr_applicant_job_id_stage_id_idx
            ON hr_applicant(job_id, stage_id)
            WHERE active IS TRUE
        """)
        self.env.cr.execute("""
            CREATE INDEX IF NOT EXISTS hr_applicant_email_partner_phone_mobile
            ON hr_applicant(email_normalized, partner_mobile_sanitized, partner_phone_sanitized);
        """)

    @api.onchange('job_id')
    def _onchange_job_id(self):
        for applicant in self:
            if applicant.job_id.name:
                applicant.name = applicant.job_id.name

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
            else:
                applicant.day_close = False

    @api.depends('day_open', 'day_close')
    def _compute_delay(self):
        for applicant in self:
            if applicant.date_open and applicant.day_close:
                applicant.delay_close = applicant.day_close - applicant.day_open
            else:
                applicant.delay_close = False

    @api.depends('email_from', 'partner_mobile_sanitized', 'partner_phone_sanitized')
    def _compute_application_count(self):
        """
            The field application_count is only used on the form view.
            Thus, using ORM rather then querying, should not make much
            difference in terms of performance, while being more readable and secure.
        """
        if not any(self._ids):
            for applicant in self:
                domain = applicant._get_similar_applicants_domain()
                if domain:
                    applicant.application_count = max(0, self.env["hr.applicant"].with_context(active_test=False).search_count(domain) - 1)
                else:
                    applicant.application_count = 0
            return
        self.flush_recordset(['email_normalized', 'partner_phone_sanitized', 'partner_mobile_sanitized'])
        self.env.cr.execute("""
            SELECT
                id,
                (
                    SELECT COUNT(*)
                    FROM hr_applicant AS sub
                    WHERE a.id != sub.id
                     AND ((a.email_normalized <> '' AND sub.email_normalized = a.email_normalized)
                       OR (a.partner_mobile_sanitized <> '' AND a.partner_mobile_sanitized = sub.partner_mobile_sanitized)
                       OR (a.partner_mobile_sanitized <> '' AND a.partner_mobile_sanitized = sub.partner_phone_sanitized)
                       OR (a.partner_phone_sanitized <> '' AND a.partner_phone_sanitized = sub.partner_mobile_sanitized)
                       OR (a.partner_phone_sanitized <> '' AND a.partner_phone_sanitized = sub.partner_phone_sanitized))
                ) AS similar_applicants
            FROM hr_applicant AS a
            WHERE id IN %(ids)s
        """, {'ids': tuple(self._origin.ids)})
        query_results = self.env.cr.dictfetchall()
        mapped_data = {result['id']: result['similar_applicants'] for result in query_results}
        for applicant in self:
            applicant.application_count = mapped_data.get(applicant.id, 0)

    def _get_similar_applicants_domain(self):
        """
            This method returns a domain for the applicants whitch match with the
            current applicant according to email_from, partner_phone or partner_mobile.
            Thus, search on the domain will return the current applicant as well if any of
            the following fields are filled.
        """
        self.ensure_one()
        if not self:
            return None
        domain = []
        if self.email_normalized:
            domain = expression.OR([domain, [('email_normalized', '=', self.email_normalized)]])
        if self.partner_phone_sanitized:
            domain = expression.OR([domain, ['|', ('partner_phone_sanitized', '=', self.partner_phone_sanitized), ('partner_mobile_sanitized', '=', self.partner_phone_sanitized)]])
        if self.partner_mobile_sanitized:
            domain = expression.OR([domain, ['|', ('partner_mobile_sanitized', '=', self.partner_mobile_sanitized), ('partner_phone_sanitized', '=', self.partner_mobile_sanitized)]])
        return domain if domain else None

    @api.depends_context('lang')
    @api.depends('meeting_ids', 'meeting_ids.start')
    def _compute_meeting_display(self):
        applicant_with_meetings = self.filtered('meeting_ids')
        (self - applicant_with_meetings).update({
            'meeting_display_text': _('No Meeting'),
            'meeting_display_date': ''
        })
        today = fields.Date.today()
        for applicant in applicant_with_meetings:
            count = len(applicant.meeting_ids)
            dates = applicant.meeting_ids.mapped('start')
            min_date, max_date = min(dates).date(), max(dates).date()
            if min_date >= today:
                applicant.meeting_display_date = min_date
            else:
                applicant.meeting_display_date = max_date
            if count == 1:
                applicant.meeting_display_text = _('1 Meeting')
            elif applicant.meeting_display_date >= today:
                applicant.meeting_display_text = _('Next Meeting')
            else:
                applicant.meeting_display_text = _('Last Meeting')

    @api.depends('refuse_reason_id', 'date_closed')
    def _compute_application_status(self):
        for applicant in self:
            if applicant.refuse_reason_id:
                applicant.application_status = 'refused'
            elif not applicant.active:
                applicant.application_status = 'archived'
            elif applicant.date_closed:
                applicant.application_status = 'hired'
            else:
                applicant.application_status = 'ongoing'

    def _get_attachment_number(self):
        read_group_res = self.env['ir.attachment']._read_group(
            [('res_model', '=', 'hr.applicant'), ('res_id', 'in', self.ids)],
            ['res_id'], ['__count'])
        attach_data = dict(read_group_res)
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
            applicant.user_id = applicant.job_id.user_id.id

    @api.depends('partner_id')
    def _compute_partner_phone_email(self):
        for applicant in self:
            if not applicant.partner_id:
                continue
            applicant.email_from = applicant.partner_id.email
            if not applicant.partner_phone:
                applicant.partner_phone = applicant.partner_id.phone
            if not applicant.partner_mobile:
                applicant.partner_mobile = applicant.partner_id.mobile

    def _inverse_partner_email(self):
        for applicant in self:
            if not applicant.email_from:
                continue
            if not applicant.partner_id:
                if not applicant.partner_name:
                    raise UserError(_('You must define a Contact Name for this applicant.'))
                applicant.partner_id = self.env['res.partner'].with_context(default_lang=self.env.lang).find_or_create(applicant.email_from)
            if applicant.partner_name and not applicant.partner_id.name:
                applicant.partner_id.name = applicant.partner_name
            if tools.email_normalize(applicant.email_from) != tools.email_normalize(applicant.partner_id.email):
                # change email on a partner will trigger other heavy code, so avoid to change the email when
                # it is the same. E.g. "email@example.com" vs "My Email" <email@example.com>""
                applicant.partner_id.email = applicant.email_from
            if applicant.partner_mobile:
                applicant.partner_id.mobile = applicant.partner_mobile
            if applicant.partner_phone:
                applicant.partner_id.phone = applicant.partner_phone

    @api.depends('partner_phone')
    def _compute_partner_phone_sanitized(self):
        for applicant in self:
            applicant.partner_phone_sanitized = applicant._phone_format(fname='partner_phone') or applicant.partner_phone

    @api.depends('partner_mobile')
    def _compute_partner_mobile_sanitized(self):
        for applicant in self:
            applicant.partner_mobile_sanitized = applicant._phone_format(fname='partner_mobile') or applicant.partner_mobile

    def _phone_get_number_fields(self):
        """ This method returns the fields to use to find the number to use to
        send an SMS on a record. """
        return ['partner_mobile', 'partner_phone']

    @api.depends('stage_id.hired_stage')
    def _compute_date_closed(self):
        for applicant in self:
            if applicant.stage_id and applicant.stage_id.hired_stage and not applicant.date_closed:
                applicant.date_closed = fields.datetime.now()
            if not applicant.stage_id.hired_stage:
                applicant.date_closed = False

    def _check_interviewer_access(self):
        if self.user_has_groups('hr_recruitment.group_hr_recruitment_interviewer') and not self.user_has_groups('hr_recruitment.group_hr_recruitment_user'):
            raise AccessError(_('You are not allowed to perform this action.'))

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if vals.get('user_id'):
                vals['date_open'] = fields.Datetime.now()
            if vals.get('email_from'):
                vals['email_from'] = vals['email_from'].strip()
        applicants = super().create(vals_list)
        applicants.sudo().interviewer_ids._create_recruitment_interviewers()
        # Record creation through calendar, creates the calendar event directly, it will also create the activity.
        if 'default_activity_date_deadline' in self.env.context:
            deadline = fields.Datetime.to_datetime(self.env.context.get('default_activity_date_deadline'))
            category = self.env.ref('hr_recruitment.categ_meet_interview')
            for applicant in applicants:
                partners = applicant.partner_id | applicant.user_id.partner_id | applicant.department_id.manager_id.user_id.partner_id
                self.env['calendar.event'].sudo().with_context(default_applicant_id=applicant.id).create({
                    'applicant_id': applicant.id,
                    'partner_ids': [(6, 0, partners.ids)],
                    'user_id': self.env.uid,
                    'name': applicant.name,
                    'categ_ids': [category.id],
                    'start': deadline,
                    'stop': deadline + relativedelta(minutes=30),
                })
        return applicants

    def write(self, vals):
        # user_id change: update date_open
        if vals.get('user_id'):
            vals['date_open'] = fields.Datetime.now()
        if vals.get('email_from'):
            vals['email_from'] = vals['email_from'].strip()
            if self._email_is_blacklisted(vals['email_from']):
                del vals['email_from']
        old_interviewers = self.interviewer_ids
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
        if 'interviewer_ids' in vals:
            interviewers_to_clean = old_interviewers - self.interviewer_ids
            interviewers_to_clean._remove_recruitment_interviewers()
            self.sudo().interviewer_ids._create_recruitment_interviewers()
        if vals.get('emp_id'):
            self._update_employee_from_applicant()
        return res

    def _email_is_blacklisted(self, mail):
        normalized_mail = tools.email_normalize(mail)
        return normalized_mail in [m.strip() for m in self.env['ir.config_parameter'].sudo().get_param('hr_recruitment.blacklisted_emails', '').split(',')]

    def get_empty_list_help(self, help_message):
        if 'active_id' in self.env.context and self.env.context.get('active_model') == 'hr.job':
            hr_job = self.env['hr.job'].browse(self.env.context['active_id'])
        elif self.env.context.get('default_job_id'):
            hr_job = self.env['hr.job'].browse(self.env.context['default_job_id'])
        else:
            hr_job = self.env['hr.job']

        nocontent_body = Markup("""
<p class="o_view_nocontent_smiling_face">%(help_title)s</p>
<p>%(para_1)s<br/>%(para_2)s</p>""") % {
            'help_title': _("No application found. Let's create one !"),
            'para_1': _('People can also apply by email to save time.'),
            'para_2': _("You can search into attachment's content, like resumes, with the searchbar."),
        }

        if hr_job.alias_email:
            nocontent_body += Markup('<p class="o_copy_paste_email oe_view_nocontent_alias">%(helper_email)s <a href="mailto:%(email)s">%(email)s</a></p>') % {
                'helper_email': _("Create new applications by sending an email to"),
                'email': hr_job.alias_email,
            }

        return super().get_empty_list_help(nocontent_body)

    @api.model
    def get_view(self, view_id=None, view_type='form', **options):
        if view_type == 'form' and self.user_has_groups('hr_recruitment.group_hr_recruitment_interviewer')\
            and not self.user_has_groups('hr_recruitment.group_hr_recruitment_user'):
            view_id = self.env.ref('hr_recruitment.hr_applicant_view_form_interviewer').id
        return super().get_view(view_id, view_type, **options)

    def action_makeMeeting(self):
        """ This opens Meeting's calendar view to schedule meeting on current applicant
            @return: Dictionary value for created Meeting view
        """
        self.ensure_one()
        if not self.partner_id:
            if not self.partner_name:
                raise UserError(_('You must define a Contact Name for this applicant.'))
            self.partner_id = self.env['res.partner'].create({
                'is_company': False,
                'name': self.partner_name,
                'email': self.email_from,
            })

        partners = self.partner_id | self.department_id.manager_id.user_id.partner_id
        if self.user_has_groups('hr_recruitment.group_hr_recruitment_interviewer') and not self.user_has_groups('hr_recruitment.group_hr_recruitment_user'):
            partners |= self.env.user.partner_id
        else:
            partners |= self.user_id.partner_id

        category = self.env.ref('hr_recruitment.categ_meet_interview')
        res = self.env['ir.actions.act_window']._for_xml_id('calendar.action_calendar_event')
        # As we are redirected from the hr.applicant, calendar checks rules on "hr.applicant",
        # in order to decide whether to allow creation of a meeting.
        # As interviewer does not have create right on the hr.applicant, in order to allow them
        # to create a meeting for an applicant, we pass 'create': True to the context.
        res['context'] = {
            'create': True,
            'default_applicant_id': self.id,
            'default_partner_ids': partners.ids,
            'default_user_id': self.env.uid,
            'default_name': self.name,
            'default_categ_ids': category and [category.id] or False,
            'attachment_ids': self.attachment_ids.ids
        }
        return res

    def action_open_attachments(self):
        return {
            'type': 'ir.actions.act_window',
            'res_model': 'ir.attachment',
            'name': _('Documents'),
            'context': {
                'default_res_model': 'hr.applicant',
                'default_res_id': self.ids[0],
                'show_partner_name': 1,
            },
            'view_mode': 'tree,form',
            'views': [
                (self.env.ref('hr_recruitment.ir_attachment_hr_recruitment_list_view').id, 'tree'),
                (False, 'form'),
            ],
            'search_view_id': self.env.ref('hr_recruitment.ir_attachment_view_search_inherit_hr_recruitment').ids,
            'domain': [('res_model', '=', 'hr.applicant'), ('res_id', 'in', self.ids), ],
        }

    def action_applications_email(self):
        self.ensure_one()
        other_applicants = self.env['hr.applicant']
        domain = self._get_similar_applicants_domain()
        if domain:
            other_applicants = self.env['hr.applicant'].with_context(active_test=False).search(domain)
        return {
            'type': 'ir.actions.act_window',
            'name': _('Job Applications'),
            'res_model': self._name,
            'view_mode': 'tree,kanban,form,pivot,graph,calendar,activity',
            'domain': [('id', 'in', other_applicants.ids)],
            'context': {
                'active_test': False,
                'search_default_stage': 1,
            },
        }

    def action_open_employee(self):
        self.ensure_one()
        return {
            'name': _('Employee'),
            'type': 'ir.actions.act_window',
            'res_model': 'hr.employee',
            'view_mode': 'form',
            'res_id': self.emp_id.id,
        }

    def _track_template(self, changes):
        res = super(Applicant, self)._track_template(changes)
        applicant = self[0]
        # When applcant is unarchived, they are put back to the default stage automatically. In this case,
        # don't post automated message related to the stage change.
        if 'stage_id' in changes and applicant.exists() and applicant.stage_id.template_id and not applicant._context.get('just_unarchived'):
            res['stage_id'] = (applicant.stage_id.template_id, {
                'auto_delete_keep_log': False,
                'subtype_id': self.env['ir.model.data']._xmlid_to_res_id('mail.mt_note'),
                'email_layout_xmlid': 'hr_recruitment.mail_notification_light_without_background'
            })
        return res

    def _creation_subtype(self):
        return self.env.ref('hr_recruitment.mt_applicant_new')

    def _track_subtype(self, init_values):
        record = self[0]
        if 'stage_id' in init_values and record.stage_id:
            return self.env.ref('hr_recruitment.mt_applicant_stage_changed')
        return super(Applicant, self)._track_subtype(init_values)

    def _notify_get_reply_to(self, default=None):
        """ Override to set alias of applicants to their job definition if any. """
        aliases = self.mapped('job_id')._notify_get_reply_to(default=default)
        res = {app.id: aliases.get(app.job_id.id) for app in self}
        leftover = self.filtered(lambda rec: not rec.job_id)
        if leftover:
            res.update(super(Applicant, leftover)._notify_get_reply_to(default=default))
        return res

    def _message_get_suggested_recipients(self):
        recipients = super(Applicant, self)._message_get_suggested_recipients()
        for applicant in self:
            if applicant.partner_id:
                applicant._message_add_suggested_recipient(recipients, partner=applicant.partner_id.sudo(), reason=_('Contact'))
            elif applicant.email_from:
                email_from = tools.email_normalize(applicant.email_from)
                if email_from and applicant.partner_name:
                    email_from = tools.formataddr((applicant.partner_name, email_from))
                    applicant._message_add_suggested_recipient(recipients, email=email_from, reason=_('Contact Email'))
        return recipients

    @api.depends('partner_name')
    @api.depends_context('show_partner_name')
    def _compute_display_name(self):
        if not self.env.context.get('show_partner_name'):
            return super()._compute_display_name()
        for applicant in self:
            applicant.display_name = applicant.partner_name or applicant.name

    @api.model
    def message_new(self, msg, custom_values=None):
        """ Overrides mail_thread message_new that is called by the mailgateway
            through message_process.
            This override updates the document according to the email.
        """
        # Remove default author when going through the mail gateway. Indeed, we
        # do not want to explicitly set user_id to False; however we do not
        # want the gateway user to be responsible if no other responsible is
        # found.
        self = self.with_context(default_user_id=False, mail_notify_author=True)  # Allows sending stage updates to the author
        stage = False
        if custom_values and 'job_id' in custom_values:
            stage = self.env['hr.job'].browse(custom_values['job_id'])._get_first_stage()
        partner_name, email_from_normalized = tools.parse_contact_from_email(msg.get('from'))
        defaults = {
            'name': msg.get('subject') or _("No Subject"),
            'partner_name': partner_name or email_from_normalized,
        }
        if msg.get('from') and not self._email_is_blacklisted(msg.get('from')):
            defaults['email_from'] = msg.get('from')
            defaults['partner_id'] = msg.get('author_id', False)
        if msg.get('email_from') and self._email_is_blacklisted(msg.get('email_from')):
            del msg['email_from']
        if msg.get('priority'):
            defaults['priority'] = msg.get('priority')
        if stage and stage.id:
            defaults['stage_id'] = stage.id
        if custom_values:
            defaults.update(custom_values)
        res = super().message_new(msg, custom_values=defaults)
        res._compute_partner_phone_email()
        return res

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
                        'name': self.partner_name or self.email_from,
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
        """ Create an employee from applicant """
        self.ensure_one()
        self._check_interviewer_access()

        if not self.partner_id:
            if not self.partner_name:
                raise UserError(_('Please provide an applicant name.'))
            self.partner_id = self.env['res.partner'].create({
                'is_company': False,
                'name': self.partner_name,
                'email': self.email_from,
            })

        action = self.env['ir.actions.act_window']._for_xml_id('hr.open_view_employee_list')
        employee = self.env['hr.employee'].create(self._get_employee_create_vals())
        action['res_id'] = employee.id
        return action

    def _get_employee_create_vals(self):
        self.ensure_one()
        address_id = self.partner_id.address_get(['contact'])['contact']
        address_sudo = self.env['res.partner'].sudo().browse(address_id)
        return {
            'name': self.partner_name or self.partner_id.display_name,
            'work_contact_id': self.partner_id.id,
            'job_id': self.job_id.id,
            'job_title': self.job_id.name,
            'private_street': address_sudo.street,
            'private_street2': address_sudo.street2,
            'private_city': address_sudo.city,
            'private_state_id': address_sudo.state_id.id,
            'private_zip': address_sudo.zip,
            'private_country_id': address_sudo.country_id.id,
            'private_phone': address_sudo.phone,
            'private_email': address_sudo.email,
            'lang': address_sudo.lang,
            'department_id': self.department_id.id,
            'address_id': self.company_id.partner_id.id,
            'work_email': self.department_id.company_id.email or self.email_from, # To have a valid email address by default
            'work_phone': self.department_id.company_id.phone,
            'applicant_id': self.ids,
            'private_phone': self.partner_phone or self.partner_mobile
        }

    def _update_employee_from_applicant(self):
        # This method is to be overriden
        return

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
                [
                    '|',
                    ('job_ids', '=', False),
                    ('job_ids', '=', job_id.id),
                    ('fold', '=', False)
                ], order='sequence asc', limit=1).id
        for applicant in self:
            applicant.write(
                {'stage_id': applicant.job_id.id and default_stage[applicant.job_id.id],
                 'refuse_reason_id': False})

    def toggle_active(self):
        self = self.with_context(just_unarchived=True)
        res = super(Applicant, self).toggle_active()
        active_applicants = self.filtered(lambda applicant: applicant.active)
        if active_applicants:
            active_applicants.reset_applicant()
        return res

    def action_send_email(self):
        return {
            'name': _('Send Email'),
            'type': 'ir.actions.act_window',
            'target': 'new',
            'view_mode': 'form',
            'res_model': 'applicant.send.mail',
            'context': {
                'default_applicant_ids': self.ids,
            }
        }

```

## File: models\hr_applicant_category.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

from random import randint

class ApplicantCategory(models.Model):
    _name = "hr.applicant.category"
    _description = "Category of applicant"

    def _get_default_color(self):
        return randint(1, 11)

    name = fields.Char("Tag Name", required=True)
    color = fields.Integer(string='Color Index', default=_get_default_color)

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Tag name already exists!"),
    ]

```

## File: models\hr_applicant_refuse_reason.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ApplicantRefuseReason(models.Model):
    _name = "hr.applicant.refuse.reason"
    _description = 'Refuse Reason of Applicant'
    _order = 'sequence'

    sequence = fields.Integer(copy=False, default=10)
    name = fields.Char('Description', required=True, translate=True)
    template_id = fields.Many2one('mail.template', string='Email Template', domain="[('model', '=', 'hr.applicant')]")
    active = fields.Boolean('Active', default=True)

```

## File: models\hr_department.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models


class HrDepartment(models.Model):
    _inherit = 'hr.department'

    new_applicant_count = fields.Integer(
        compute='_compute_new_applicant_count', string='New Applicant', compute_sudo=True)
    new_hired_employee = fields.Integer(
        compute='_compute_recruitment_stats', string='New Hired Employee')
    expected_employee = fields.Integer(
        compute='_compute_recruitment_stats', string='Expected Employee')

    def _compute_new_applicant_count(self):
        if self.env.user.has_group('hr_recruitment.group_hr_recruitment_interviewer'):
            applicant_data = self.env['hr.applicant']._read_group(
                [('department_id', 'in', self.ids), ('stage_id.sequence', '<=', '1')],
                ['department_id'], ['__count'])
            result = {department.id: count for department, count in applicant_data}
            for department in self:
                department.new_applicant_count = result.get(department.id, 0)
        else:
            self.new_applicant_count = 0

    def _compute_recruitment_stats(self):
        job_data = self.env['hr.job']._read_group(
            [('department_id', 'in', self.ids)],
            ['department_id'], ['no_of_hired_employee:sum', 'no_of_recruitment:sum'])
        new_emp = {department.id: nb_employee for department, nb_employee, __ in job_data}
        expected_emp = {department.id: nb_recruitment for department, __, nb_recruitment in job_data}
        for department in self:
            department.new_hired_employee = new_emp.get(department.id, 0)
            department.expected_employee = expected_emp.get(department.id, 0)

```

## File: models\hr_employee.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from datetime import timedelta


class HrEmployee(models.Model):
    _inherit = "hr.employee"

    applicant_id = fields.One2many('hr.applicant', 'emp_id', 'Applicant')

    @api.model_create_multi
    def create(self, vals_list):
        employees = super().create(vals_list)
        for employee in employees:
            if employee.applicant_id:
                employee.applicant_id._message_log_with_view(
                    'hr_recruitment.applicant_hired_template',
                    render_values={'applicant': employee.applicant_id}
                )
        return employees

```

## File: models\hr_job.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast
from collections import defaultdict

from odoo import api, fields, models, SUPERUSER_ID, _


class Job(models.Model):
    _name = "hr.job"
    _inherit = ["mail.alias.mixin", "hr.job"]
    _order = "sequence, name asc"

    @api.model
    def _default_address_id(self):
        last_used_address = self.env['hr.job'].search([('company_id', 'in', self.env.companies.ids)], order='id desc', limit=1)
        if last_used_address:
            return last_used_address.address_id
        else:
            return self.env.company.partner_id

    def _address_id_domain(self):
        return ['|', '&', '&', ('type', '!=', 'contact'), ('type', '!=', 'private'),
                ('id', 'in', self.sudo().env.companies.partner_id.child_ids.ids),
                ('id', 'in', self.sudo().env.companies.partner_id.ids)]

    def _get_default_favorite_user_ids(self):
        return [(6, 0, [self.env.uid])]

    address_id = fields.Many2one(
        'res.partner', "Job Location", default=_default_address_id,
        domain=lambda self: self._address_id_domain(),
        help="Select the location where the applicant will work. Addresses listed here are defined on the company's contact information.")
    application_ids = fields.One2many('hr.applicant', 'job_id', "Job Applications")
    application_count = fields.Integer(compute='_compute_application_count', string="Application Count")
    all_application_count = fields.Integer(compute='_compute_all_application_count', string="All Application Count")
    new_application_count = fields.Integer(
        compute='_compute_new_application_count', string="New Application",
        help="Number of applications that are new in the flow (typically at first step of the flow)")
    old_application_count = fields.Integer(
        compute='_compute_old_application_count', string="Old Application")
    applicant_hired = fields.Integer(compute='_compute_applicant_hired', string="Applicants Hired")
    manager_id = fields.Many2one(
        'hr.employee', related='department_id.manager_id', string="Department Manager",
        readonly=True, store=True)
    user_id = fields.Many2one('res.users', "Recruiter", domain="[('share', '=', False), ('company_ids', 'in', company_id)]", tracking=True, help="The Recruiter will be the default value for all Applicants Recruiter's field in this job position. The Recruiter is automatically added to all meetings with the Applicant.")
    document_ids = fields.One2many('ir.attachment', compute='_compute_document_ids', string="Documents", readonly=True)
    documents_count = fields.Integer(compute='_compute_document_ids', string="Document Count")
    alias_id = fields.Many2one(help="Email alias for this job position. New emails will automatically create new applicants for this job position.")
    color = fields.Integer("Color Index")
    is_favorite = fields.Boolean(compute='_compute_is_favorite', inverse='_inverse_is_favorite')
    favorite_user_ids = fields.Many2many('res.users', 'job_favorite_user_rel', 'job_id', 'user_id', default=_get_default_favorite_user_ids)
    interviewer_ids = fields.Many2many('res.users', string='Interviewers', domain="[('share', '=', False), ('company_ids', 'in', company_id)]", help="The Interviewers set on the job position can see all Applicants in it. They have access to the information, the attachments, the meeting management and they can refuse him. You don't need to have Recruitment rights to be set as an interviewer.")
    extended_interviewer_ids = fields.Many2many('res.users', 'hr_job_extended_interviewer_res_users', compute='_compute_extended_interviewer_ids', store=True)

    activities_overdue = fields.Integer(compute='_compute_activities')
    activities_today = fields.Integer(compute='_compute_activities')

    applicant_properties_definition = fields.PropertiesDefinition('Applicant Properties')

    @api.depends_context('uid')
    def _compute_activities(self):
        self.env.cr.execute("""
            SELECT
                app.job_id,
                COUNT(*) AS act_count,
                CASE
                    WHEN %(today)s::date - act.date_deadline::date = 0 THEN 'today'
                    WHEN %(today)s::date - act.date_deadline::date > 0 THEN 'overdue'
                END AS act_state
             FROM mail_activity act
             JOIN hr_applicant app ON app.id = act.res_id
             JOIN hr_recruitment_stage sta ON app.stage_id = sta.id
            WHERE act.user_id = %(user_id)s AND act.res_model = 'hr.applicant'
              AND act.date_deadline <= %(today)s::date AND app.active
              AND app.job_id IN %(job_ids)s
              AND sta.hired_stage IS NOT TRUE
            GROUP BY app.job_id, act_state
        """, {
            'today': fields.Date.context_today(self),
            'user_id': self.env.uid,
            'job_ids': tuple(self.ids),
        })
        job_activities = defaultdict(dict)
        for activity in self.env.cr.dictfetchall():
            job_activities[activity['job_id']][activity['act_state']] = activity['act_count']
        for job in self:
            job.activities_overdue = job_activities[job.id].get('overdue', 0)
            job.activities_today = job_activities[job.id].get('today', 0)

    @api.depends('application_ids.interviewer_ids')
    def _compute_extended_interviewer_ids(self):
        # Use SUPERUSER_ID as the search_read is protected in hr_referral
        results_raw = self.env['hr.applicant'].with_user(SUPERUSER_ID).search_read([
            ('job_id', 'in', self.ids),
            ('interviewer_ids', '!=', False)
        ], ['interviewer_ids', 'job_id'])
        interviewers_by_job = defaultdict(set)
        for result_raw in results_raw:
            interviewers_by_job[result_raw['job_id'][0]] |= set(result_raw['interviewer_ids'])
        for job in self:
            job.extended_interviewer_ids = [(6, 0, list(interviewers_by_job[job.id]))]

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
        read_group_result = self.env['hr.applicant'].with_context(active_test=False)._read_group([
            ('job_id', 'in', self.ids),
            '|',
                ('active', '=', True),
                '&',
                ('active', '=', False), ('refuse_reason_id', '!=', False),
        ], ['job_id'], ['__count'])
        result = {job.id: count for job, count in read_group_result}
        for job in self:
            job.all_application_count = result.get(job.id, 0)

    def _compute_application_count(self):
        read_group_result = self.env['hr.applicant']._read_group([('job_id', 'in', self.ids)], ['job_id'], ['__count'])
        result = {job.id: count for job, count in read_group_result}
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
                   WHERE a.company_id in %s
              GROUP BY s.job_id
            """, [tuple(self.ids), tuple(self.env.companies.ids)]
        )

        new_applicant_count = dict(self.env.cr.fetchall())
        for job in self:
            job.new_application_count = new_applicant_count.get(job.id, 0)

    def _compute_applicant_hired(self):
        hired_stages = self.env['hr.recruitment.stage'].search([('hired_stage', '=', True)])
        hired_data = self.env['hr.applicant']._read_group([
            ('job_id', 'in', self.ids),
            ('stage_id', 'in', hired_stages.ids),
        ], ['job_id'], ['__count'])
        job_hires = {job.id: count for job, count in hired_data}
        for job in self:
            job.applicant_hired = job_hires.get(job.id, 0)

    @api.depends('application_count', 'new_application_count')
    def _compute_old_application_count(self):
        for job in self:
            job.old_application_count = job.application_count - job.new_application_count

    def _alias_get_creation_values(self):
        values = super(Job, self)._alias_get_creation_values()
        values['alias_model_id'] = self.env['ir.model']._get('hr.applicant').id
        if self.id:
            values['alias_defaults'] = defaults = ast.literal_eval(self.alias_defaults or "{}")
            defaults.update({
                'job_id': self.id,
                'department_id': self.department_id.id,
                'company_id': self.department_id.company_id.id if self.department_id else self.company_id.id,
                'user_id': self.user_id.id,
            })
        return values

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            vals['favorite_user_ids'] = vals.get('favorite_user_ids', []) + [(4, self.env.uid)]
        jobs = super().create(vals_list)
        utm_linkedin = self.env.ref("utm.utm_source_linkedin", raise_if_not_found=False)
        if utm_linkedin:
            source_vals = [{
                'source_id': utm_linkedin.id,
                'job_id': job.id,
            } for job in jobs]
            self.env['hr.recruitment.source'].create(source_vals)
        jobs.sudo().interviewer_ids._create_recruitment_interviewers()
        return jobs

    def write(self, vals):
        old_interviewers = self.interviewer_ids
        if 'active' in vals and not vals['active']:
            self.application_ids.active = False
        res = super().write(vals)
        if 'interviewer_ids' in vals:
            interviewers_to_clean = old_interviewers - self.interviewer_ids
            interviewers_to_clean._remove_recruitment_interviewers()
            self.sudo().interviewer_ids._create_recruitment_interviewers()

        # Since the alias is created upon record creation, the default values do not reflect the current values unless
        # specifically rewritten
        # List of fields to keep synched with the alias
        alias_fields = {'department_id', 'user_id'}
        if any(field for field in alias_fields if field in vals):
            for job in self:
                alias_default_vals = job._alias_get_creation_values().get('alias_defaults', '{}')
                job.alias_defaults = alias_default_vals
        return res

    def _creation_subtype(self):
        return self.env.ref('hr_recruitment.mt_job_new')

    def action_open_attachments(self):
        return {
            'type': 'ir.actions.act_window',
            'res_model': 'ir.attachment',
            'name': _('Documents'),
            'context': {
                'default_res_model': self._name,
                'default_res_id': self.ids[0],
                'show_partner_name': 1,
            },
            'view_mode': 'tree',
            'views': [
                (self.env.ref('hr_recruitment.ir_attachment_hr_recruitment_list_view').id, 'tree')
            ],
            'search_view_id': self.env.ref('hr_recruitment.ir_attachment_view_search_inherit_hr_recruitment').ids,
            'domain': ['|',
                '&', ('res_model', '=', 'hr.job'), ('res_id', 'in', self.ids),
                '&', ('res_model', '=', 'hr.applicant'), ('res_id', 'in', self.application_ids.ids),
            ],
        }

    def action_open_activities(self):
        action = self.env["ir.actions.actions"]._for_xml_id("hr_recruitment.action_hr_job_applications")
        views = ['activity'] + [view for view in action['view_mode'].split(',') if view != 'activity']
        action['view_mode'] = ','.join(views)
        action['views'] = [(False, view) for view in views]
        return action

    def action_open_late_activities(self):
        action = self.action_open_activities()
        action['context'] = {
            'default_job_id': self.id,
            'search_default_job_id': self.id,
            'search_default_activities_overdue': True,
            'search_default_running_applicant_activities': True,
        }
        return action

    def action_open_today_activities(self):
        action = self.action_open_activities()
        action['context'] = {
            'default_job_id': self.id,
            'search_default_job_id': self.id,
            'search_default_activities_today': True,
        }
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

## File: models\hr_recruitment_degree.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class RecruitmentDegree(models.Model):
    _name = "hr.recruitment.degree"
    _description = "Applicant Degree"
    _sql_constraints = [
        ('name_uniq', 'unique (name)', 'The name of the Degree of Recruitment must be unique!')
    ]

    name = fields.Char("Degree Name", required=True, translate=True)
    sequence = fields.Integer("Sequence", default=1)

```

## File: models\hr_recruitment_source.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class RecruitmentSource(models.Model):
    _name = "hr.recruitment.source"
    _description = "Source of Applicants"
    _inherit = ['utm.source.mixin']

    email = fields.Char(related='alias_id.display_name', string="Email", readonly=True)
    has_domain = fields.Char(compute='_compute_has_domain')
    job_id = fields.Many2one('hr.job', "Job", ondelete='cascade')
    alias_id = fields.Many2one('mail.alias', "Alias ID", ondelete='restrict')
    medium_id = fields.Many2one('utm.medium', default=lambda self: self.env.ref('utm.utm_medium_website'))

    def _compute_has_domain(self):
        for source in self:
            if source.alias_id:
                source.has_domain = bool(source.alias_id.alias_domain_id)
            else:
                source.has_domain = bool(source.job_id.company_id.alias_domain_id
                                         or self.env.company.alias_domain_id)

    def create_alias(self):
        campaign = self.env.ref('hr_recruitment.utm_campaign_job')
        medium = self.env.ref('utm.utm_medium_email')
        for source in self.filtered(lambda s: not s.alias_id):
            vals = {
                'alias_defaults': {
                    'job_id': source.job_id.id,
                    'campaign_id': campaign.id,
                    'medium_id': medium.id,
                    'source_id': source.source_id.id,
                },
                'alias_domain_id': source.job_id.company_id.alias_domain_id.id or self.env.company.alias_domain_id.id,
                'alias_model_id': self.env['ir.model']._get_id('hr.applicant'),
                'alias_name': f"{source.job_id.alias_name or source.job_id.name}+{source.name}",
                'alias_parent_thread_id': source.job_id.id,
                'alias_parent_model_id': self.env['ir.model']._get_id('hr.job'),
            }

            # check that you can create source before to call mail.alias in sudo with known/controlled vals
            source.check_access_rights('create')
            source.check_access_rule('create')
            source.alias_id = self.env['mail.alias'].sudo().create(vals)

    def unlink(self):
        """ Cascade delete aliases to avoid useless / badly configured aliases. """
        aliases = self.alias_id
        res = super().unlink()
        aliases.sudo().unlink()
        return res

```

## File: models\hr_recruitment_stage.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class RecruitmentStage(models.Model):
    _name = "hr.recruitment.stage"
    _description = "Recruitment Stages"
    _order = 'sequence'

    name = fields.Char("Stage Name", required=True, translate=True)
    sequence = fields.Integer(
        "Sequence", default=10)
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
    hired_stage = fields.Boolean('Hired Stage',
        help="If checked, this stage is used to determine the hire date of an applicant")
    legend_blocked = fields.Char(
        'Red Kanban Label', default=lambda self: _('Blocked'), translate=True, required=True)
    legend_done = fields.Char(
        'Green Kanban Label', default=lambda self: _('Ready for Next Stage'), translate=True, required=True)
    legend_normal = fields.Char(
        'Grey Kanban Label', default=lambda self: _('In Progress'), translate=True, required=True)
    is_warning_visible = fields.Boolean(compute='_compute_is_warning_visible')

    @api.model
    def default_get(self, fields):
        if self._context and self._context.get('default_job_id') and not self._context.get('hr_recruitment_stage_mono', False):
            context = dict(self._context)
            context.pop('default_job_id')
            self = self.with_context(context)
        return super(RecruitmentStage, self).default_get(fields)

    @api.depends('hired_stage')
    def _compute_is_warning_visible(self):
        applicant_data = self.env['hr.applicant']._read_group([('stage_id', 'in', self.ids)], ['stage_id'], ['__count'])
        applicants = {stage.id: count for stage, count in applicant_data}
        for stage in self:
            if stage._origin.hired_stage and not stage.hired_stage and applicants.get(stage._origin.id):
                stage.is_warning_visible = True
            else:
                stage.is_warning_visible = False

```

## File: models\ir_ui_menu.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class IrUiMenu(models.Model):
    _inherit = 'ir.ui.menu'

    def _load_menus_blacklist(self):
        res = super()._load_menus_blacklist()
        if self.env.user.has_group('hr_recruitment.group_hr_recruitment_interviewer') and not self.env.user.has_group('hr_recruitment.group_hr_recruitment_user'):
            res.append(self.env.ref('hr_recruitment.menu_hr_job_position').id)
        elif self.env.user.has_group('hr_recruitment.group_hr_recruitment_user'):
            res.append(self.env.ref('hr_recruitment.menu_hr_job_position_interviewer').id)
        return res

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
    group_applicant_cv_display = fields.Boolean(implied_group="hr_recruitment.group_applicant_cv_display")
    module_hr_recruitment_extract = fields.Boolean(string='Send CV to OCR to fill applications')

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class ResUsers(models.Model):
    _inherit = 'res.users'

    def _create_recruitment_interviewers(self):
        if not self:
            return
        interviewer_group = self.env.ref('hr_recruitment.group_hr_recruitment_interviewer')
        recruitment_group = self.env.ref('hr_recruitment.group_hr_recruitment_user')

        interviewers = self - recruitment_group.users
        interviewers.sudo().write({
            'groups_id': [(4, interviewer_group.id)]
        })

    def _remove_recruitment_interviewers(self):
        if not self:
            return
        interviewer_group = self.env.ref('hr_recruitment.group_hr_recruitment_interviewer')
        recruitment_group = self.env.ref('hr_recruitment.group_hr_recruitment_user')

        job_interviewers = self.env['hr.job']._read_group([('interviewer_ids', 'in', self.ids)], ['interviewer_ids'])
        user_ids = {interviewer.id for [interviewer] in job_interviewers}

        application_interviewers = self.env['hr.applicant']._read_group([('interviewer_ids', 'in', self.ids)], ['interviewer_ids'])
        user_ids |= {interviewer.id for [interviewer] in application_interviewers}

        # Remove users that are no longer interviewers on at least a job or an application
        users_to_remove = set(self.ids) - (user_ids | set(recruitment_group.users.ids))
        self.env['res.users'].browse(users_to_remove).sudo().write({
            'groups_id': [(3, interviewer_group.id)]
        })

```

## File: models\utm_campaign.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models

from odoo.exceptions import UserError


class UtmCampaign(models.Model):
    _inherit = 'utm.campaign'

    @api.ondelete(at_uninstall=False)
    def _unlink_except_utm_campaign_job(self):
        utm_campaign_job = self.env.ref('hr_recruitment.utm_campaign_job', raise_if_not_found=False)
        if utm_campaign_job and utm_campaign_job in self:
            raise UserError(_(
                "The UTM campaign '%s' cannot be deleted as it is used in the recruitment process.",
                utm_campaign_job.name
            ))

```

## File: models\utm_source.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models

from odoo.exceptions import UserError


class UtmSource(models.Model):
    _inherit = 'utm.source'

    @api.ondelete(at_uninstall=False)
    def _unlink_except_linked_recruitment_sources(self):
        """ Already handled by ondelete='restrict', but let's show a nice error message """
        linked_recruitment_sources = self.env['hr.recruitment.source'].sudo().search([
            ('source_id', 'in', self.ids)
        ])

        if linked_recruitment_sources:
            raise UserError(_(
                "You cannot delete these UTM Sources as they are linked to the following recruitment sources in "
                "Recruitment:\n%(recruitment_sources)s",
                recruitment_sources=', '.join(['"%s"' % name for name in linked_recruitment_sources.job_id.mapped('name')])))

```

## File: models\__init__.py

```python
from . import hr_department
from . import hr_applicant
from . import hr_applicant_category
from . import hr_applicant_refuse_reason
from . import hr_recruitment_degree
from . import hr_recruitment_source
from . import hr_recruitment_stage
from . import hr_employee
from . import hr_job
from . import res_config_settings
from . import calendar
from . import digest
from . import utm_campaign
from . import utm_source
from . import res_users
from . import ir_ui_menu

```

## File: security\hr_recruitment_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record model="ir.module.category" id="base.module_category_human_resources_recruitment">
        <field name="description">Interviewer right will give access to all job position/applications where the employee is defined. It will allow to refuse, plan meetings.</field>
        <field name="sequence">11</field>
    </record>

    <record id="hr_applicant_comp_rule" model="ir.rule">
        <field name="name">Applicant multi company rule</field>
        <field name="model_id" ref="model_hr_applicant"/>
        <field eval="True" name="global"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="group_hr_recruitment_interviewer" model="res.groups">
        <field name="name">Interviewer</field>
        <field name="category_id" ref="base.module_category_human_resources_recruitment"/>
        <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record id="group_hr_recruitment_user" model="res.groups">
        <field name="name">Officer: Manage all applicants</field>
        <field name="category_id" ref="base.module_category_human_resources_recruitment"/>
        <field name="implied_ids" eval="[(4, ref('hr.group_hr_user')), (4, ref('group_hr_recruitment_interviewer'))]"/>
    </record>

    <record id="group_hr_recruitment_manager" model="res.groups">
        <field name="name">Administrator</field>
        <field name="category_id" ref="base.module_category_human_resources_recruitment"/>
        <field name="implied_ids" eval="[(4, ref('group_hr_recruitment_user')), (4, ref('mail.group_mail_template_editor'))]"/>
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
    </record>

    <record id="base.default_user" model="res.users">
        <field name="groups_id" eval="[(4,ref('hr_recruitment.group_hr_recruitment_manager'))]"/>
    </record>

    <record id="group_applicant_cv_display" model="res.groups">
        <field name="name">Display CV on application form</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="base.group_user" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('hr_recruitment.group_applicant_cv_display'))]"/>
    </record>

    <!-- Interviewer Access Rules -->
    <record id="hr_applicant_interviewer_rule" model="ir.rule">
        <field name="name">Applicant Interviewer</field>
        <field name="model_id" ref="model_hr_applicant"/>
        <field name="domain_force">[
            '|',
                ('job_id.interviewer_ids', 'in', user.id),
                ('interviewer_ids', 'in', user.id),
        ]</field>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]"/>
    </record>

    <record id="hr_applicant_user_rule" model="ir.rule">
        <field name="name">User: All Applicants</field>
        <field name="model_id" ref="model_hr_applicant"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_user'))]"/>
    </record>

    <record id="hr_job_user_rule" model="ir.rule">
        <field name="name">User: All Applicants</field>
        <field name="model_id" ref="model_hr_job"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_user'))]"/>
    </record>

    <record id="mail_message_user_rule" model="ir.rule">
        <field name="name">User: All Chatter</field>
        <field name="model_id" ref="mail.model_mail_message"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_user'))]"/>
    </record>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_job_interviewer,hr.job.interviewer,hr.model_hr_job,group_hr_recruitment_interviewer,1,0,0,0
access_hr_applicant_interviewer,hr.applicant.interviewer,model_hr_applicant,group_hr_recruitment_interviewer,1,1,0,0
access_hr_applicant_user,hr.applicant.user,model_hr_applicant,group_hr_recruitment_user,1,1,1,1
access_hr_recruitment_stage_interviewer,hr.recruitment.stage.interviewer,model_hr_recruitment_stage,group_hr_recruitment_interviewer,1,0,0,0
access_hr_recruitment_stage_user,hr.recruitment.stage.user,model_hr_recruitment_stage,group_hr_recruitment_user,1,0,0,0
access_hr_recruitment_stage_manager,hr.recruitment.stage.manager,model_hr_recruitment_stage,group_hr_recruitment_manager,1,1,1,1
access_hr_recruitment_degree,hr.recruitment.degree,model_hr_recruitment_degree,group_hr_recruitment_user,1,1,1,1
access_hr_recruitment_refuse_reason_interviewer,hr.applicant.refuse.reason.interviewer,model_hr_applicant_refuse_reason,group_hr_recruitment_interviewer,1,0,0,0
access_hr_recruitment_refuse_reason,hr.applicant.refuse.reason,model_hr_applicant_refuse_reason,group_hr_recruitment_user,1,1,1,1
access_res_partner_hr_user,res.partner.user,base.model_res_partner,group_hr_recruitment_user,1,1,1,1
access_calendar_event_hruser,calendar.event.hruser,calendar.model_calendar_event,group_hr_recruitment_user,1,1,1,1
access_hr_recruitment_source_hr_officer,hr.recruitment.source,model_hr_recruitment_source,group_hr_recruitment_user,1,1,1,1
access_hr_recruitment_source_all,hr.recruitment.source,model_hr_recruitment_source,base.group_user,1,0,0,0
access_hr_applicant_category,hr.applicant_category,model_hr_applicant_category,base.group_user,1,1,1,0
access_hr_applicant_category_manager,hr.applicant_category,model_hr_applicant_category,group_hr_recruitment_user,1,1,1,1
access_calendar_event_type_hr_officer,calendar.event.type.officer,calendar.model_calendar_event_type,group_hr_recruitment_user,1,1,1,0
access_applicant_get_refuse_reason,access.applicant.get.refuse.reason,model_applicant_get_refuse_reason,hr_recruitment.group_hr_recruitment_user,1,1,1,0
access_applicant_get_refuse_reason_interviewer,access.applicant.get.refuse.reason.interviewer,model_applicant_get_refuse_reason,hr_recruitment.group_hr_recruitment_interviewer,1,1,1,0
access_applicant_send_mail,access.applicant.send.mail,model_applicant_send_mail,hr_recruitment.group_hr_recruitment_user,1,1,1,0
access_applicant_send_mail_interviewer,access.applicant.send.mail.interviewer,model_applicant_send_mail,hr_recruitment.group_hr_recruitment_interviewer,1,1,1,0

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M32 25a7 7 0 1 1-14 0 7 7 0 0 1 14 0Z" fill="#985184"/><path d="M25 46C13.402 46 4 36.598 4 25S13.402 4 25 4s21 9.402 21 21c0 5.799-2.35 11.049-6.15 14.85l-6.365-6.365A11.964 11.964 0 0 0 37 25c0-6.627-5.373-12-12-12s-12 5.373-12 12 5.373 12 12 12v9Z" fill="#1AD3BB"/><path fill-rule="evenodd" clip-rule="evenodd" d="M25 37a11.958 11.958 0 0 1-8.345-3.377A9 9 0 0 0 25 46h13a9 9 0 0 0-9-9h-4Z" fill="#985184"/></svg>

```

## File: static\src\applicant_char\applicant_char.js

```javascript
/** @odoo-module */

import { CharField, charField } from "@web/views/fields/char/char_field";
import { registry } from "@web/core/registry";

import { useService } from "@web/core/utils/hooks";

export class ApplicantCharField extends CharField {
    setup() {
        super.setup();

        this.action = useService("action");
    }

    onClick() {
        const record = this.props.record.data;
        if (record.res_id !== undefined && record.res_model == 'hr.applicant') {
            this.action.doAction({
                type: 'ir.actions.act_window',
                res_model: 'hr.applicant',
                res_id: record.res_id,
                views: [[false, "form"]],
                view_mode: "form",
                target: "current",
            });
        }
    }
}
ApplicantCharField.template = "hr_recruitment.ApplicantCharField";

export const applicantCharField = {
    ...charField,
    component: ApplicantCharField,
};

registry.category("fields").add("applicant_char", applicantCharField);

```

## File: static\src\applicant_char\applicant_char.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="hr_recruitment.ApplicantCharField" t-inherit="web.CharField" t-inherit-mode="primary">
        <xpath expr="//span[@t-esc='formattedValue']" position="attributes">
            <attribute name="t-on-click.prevent.stop">onClick</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\js\tours\hr_recruitment.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { stepUtils } from "@web_tour/tour_service/tour_utils";
import { markup } from "@odoo/owl";

registry.category("web_tour.tours").add('hr_recruitment_tour',{
    url: "/web",
    rainbowManMessage: () => markup(_t("<div>Great job! You hired a new colleague!</div><div>Try the Website app to publish job offers online.</div>")),
    fadeout: 'very_slow',
    sequence: 230,
    steps: () => [stepUtils.showAppsMenuItem(), {
    trigger: '.o_app[data-menu-xmlid="hr_recruitment.menu_hr_recruitment_root"]',
    content: markup(_t("Let's have a look at how to <b>improve</b> your <b>hiring process</b>.")),
    position: 'right',
    edition: 'community'
}, {
    trigger: '.o_app[data-menu-xmlid="hr_recruitment.menu_hr_recruitment_root"]',
    content: markup(_t("Let's have a look at how to <b>improve</b> your <b>hiring process</b>.")),
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
    position: "right",
    width: 195
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
    position: "bottom",
    width: 195
}, {
    trigger: ".oe_kanban_action_button",
    extra_trigger: '.o_hr_recruitment_kanban',
    content: markup(_t("<b>Did you apply by sending an email?</b> Check incoming applications.")),
    position: "bottom"
}, {
    trigger: ".oe_kanban_card",
    extra_trigger: '.o_kanban_applicant',
    content: markup(_t("<b>Drag this card</b>, to qualify him for a first interview.")),
    position: "bottom",
    run: "drag_and_drop .o_kanban_group:eq(1) ",
}, {
    trigger: ".oe_kanban_card",
    extra_trigger: '.o_kanban_applicant',
    content: markup(_t("<b>Click to view</b> the application.")),
    position: "bottom",
    width: 195
}, {
    trigger: "button:contains(Send message)",
    extra_trigger: '.o_applicant_form',
    content: markup(_t("<div><b>Try to send an email</b> to the applicant.</div><div><i>Tips: All emails sent or received are saved in the history here</i>")),
    position: "bottom"
}, {
    trigger: ".o-mail-Chatter .o-mail-Composer button[aria-label='Send']",
    extra_trigger: '.o_applicant_form',
    content: _t("Send your email. Followers will get a copy of the communication."),
    position: "bottom"
}, {
    trigger: "button:contains(Log note)",
    extra_trigger: '.o_applicant_form',
    content: _t("Or talk about this applicant privately with your colleagues."),
    position: "bottom"
}, {
    trigger: ".o_create_employee",
    extra_trigger: '.o_applicant_form',
    content: _t("Let’s create this new employee now."),
    position: "bottom",
    width: 225
}, {
    trigger: ".o_form_button_save",
    extra_trigger: ".o_hr_employee_form_view",
    content: _t("Save it!"),
    position: "bottom",
    width: 80
}]});

```

## File: static\src\views\form_view.js

```javascript
/** @odoo-module */

import { registry } from '@web/core/registry';

import { formView } from '@web/views/form/form_view';
import { FormController } from '@web/views/form/form_controller';

export class InterviewerFormController extends FormController {

    /**
     * Add `o_applicant_interviewer_form` class if necessary
     */
    get className() {
        const result = super.className;
        const root = this.model.root;
        if (!root.data.interviewer_ids || !root.data.user_id) {
            return result;
        }
        result["o_applicant_interviewer_form"] = root.data.interviewer_ids.records.findIndex(
            interviewer => interviewer.resId === root.data.user_id[0]) > -1;
        return result;
    }
}

registry.category('views').add('hr_recruitment_interviewer', {
    ...formView,
    Controller: InterviewerFormController,
});

```

## File: views\digest_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="digest_digest_view_form" model="ir.ui.view">
        <field name="name">digest.digest.view.form.inherit.hr.recruitment</field>
        <field name="model">digest.digest</field>
        <field name="priority">70</field>
        <field name="inherit_id" ref="digest.digest_digest_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//group[@name='kpis']/group[last()]" position="before">
                <group name="kpi_hr" string="Recruitment" groups="hr_recruitment.group_hr_recruitment_user">
                    <field name="kpi_hr_recruitment_new_colleagues"/>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\hr_applicant_category_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
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

        <record id="hr_applicant_category_action" model="ir.actions.act_window">
            <field name="name">Tags</field>
            <field name="res_model">hr.applicant.category</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Add a new tag
                </p>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\hr_applicant_refuse_reason_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record id="hr_applicant_refuse_reason_view_form" model="ir.ui.view">
            <field name="name">Applicant refuse reason form</field>
            <field name="model">hr.applicant.refuse.reason</field>
            <field name="arch" type="xml">
                <form string="Refuse Reason">
                    <sheet>
                        <widget name="web_ribbon" text="Archived" bg_color="text-bg-danger" invisible="active"/>
                        <div class="oe_title">
                            <div class="oe_edit_only">
                                <label for="name"/>
                            </div>
                            <h1>
                                <field name="name"/>
                            </h1>
                            <field name="active" invisible="1"/>
                        </div>
                        <group>
                            <field name="template_id" context="{'default_model': 'hr.applicant'}"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="hr_applicant_refuse_reason_view_tree" model="ir.ui.view">
            <field name="name">Applicant refuse reason tree</field>
            <field name="model">hr.applicant.refuse.reason</field>
            <field name="arch" type="xml">
                <tree string="Refuse Reason" editable="bottom">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                    <field name="template_id" context="{'default_model': 'hr.applicant'}"/>
                </tree>
            </field>
        </record>

        <record id="hr_applicant_refuse_reason_action" model="ir.actions.act_window">
            <field name="name">Refuse Reasons</field>
            <field name="res_model">hr.applicant.refuse.reason</field>
            <field name="view_mode">tree,form</field>
        </record>
    </data>
</odoo>

```

## File: views\hr_applicant_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
    <record model="ir.ui.view" id="crm_case_tree_view_job">
        <field name="name">Applicants</field>
        <field name="model">hr.applicant</field>
        <field name="arch" type="xml">
            <tree string="Applicants" multi_edit="1" sample="1" decoration-danger="application_status == 'refused'">
                <field name="message_needaction" column_invisible="True"/>
                <field name="last_stage_id" column_invisible="True"/>
                <field name="date_last_stage_update" column_invisible="True"/>
                <field name="partner_name" readonly="1" optional="show"/>
                <field name="create_date" readonly="1" widget="date" optional="show"/>
                <field name="type_id" column_invisible="True"/>
                <field name="job_id" optional="show"/>
                <field name="name" readonly="1" optional="show"/>
                <field name="stage_id" optional="show"/>
                <field name="application_status" optional="show" invisible="application_status == 'ongoing'"/>
                <field name="refuse_reason_id" optional='hide'/>
                <field name="priority" widget="priority" optional="show"/>
                <field name="email_from" readonly="1" optional="hide"/>
                <field name="partner_mobile" widget="phone" readonly="1" optional="show" class="text-end"/>
                <field name="categ_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="show"/>
                <field name="user_id" widget="many2one_avatar_user" optional="show"/>
                <field name="interviewer_ids" widget="many2many_avatar_user" optional="hide"/>
                <field name="partner_phone" widget="phone" readonly="1" optional="hide"/>
                <field name="medium_id" optional="hide"/>
                <field name="source_id" readonly="1" optional="hide"/>
                <field name="salary_expected" optional="hide"/>
                <field name="salary_proposed" optional="hide"/>
                <field name="availability" optional="hide"/>
                <field name="department_id" readonly="1" column_invisible="context.get('invisible_department', True)"/>
                <field name="company_id" column_invisible="True"/>
                <field name="company_id" groups="base.group_multi_company" readonly="1" optional="hide"/>
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
            <field name="company_id" invisible="1"/>
            <field name="application_status" invisible="1"/>
            <field name="emp_id" invisible="1"/>
            <field name="meeting_ids" invisible="1"/>
            <field name="refuse_reason_id" invisible="1"/>
            <field name="email_normalized" invisible="1"/>
            <field name="partner_phone_sanitized" invisible="1"/>
            <field name="partner_mobile_sanitized" invisible="1"/>
            <field name="emp_is_active" invisible="1"/>
            <header>
                <button string="Create Employee" name="create_employee_from_applicant" type="object" data-hotkey="q" groups="hr_recruitment.group_hr_recruitment_user"
                        class="o_create_employee" invisible="emp_id or not active or not date_closed"/>
                <button string="Refuse" name="archive_applicant" type="object" invisible="not active" data-hotkey="d"/>
                <button string="Restore" name="toggle_active" type="object" invisible="active" data-hotkey="x"/>
                <field name="stage_id" widget="statusbar" options="{'clickable': '1', 'fold_field': 'fold'}" invisible="not active and not emp_id"/>
            </header>
            <sheet>
                <div class="oe_button_box" name="button_box">
                    <button name="action_open_employee"
                            type="object"
                            class="oe_stat_button"
                            icon="fa-id-card-o"
                            groups="hr.group_hr_user"
                            invisible="not (emp_id or emp_is_active)">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value"><field name="employee_name" readonly="1"/></span>
                            <span class="o_stat_text">Employee</span>
                        </div>
                    </button>
                    <button name="action_applications_email"
                            class="oe_stat_button"
                            icon="fa-pencil"
                            type="object"
                            context="{'active_test': False}"
                            invisible="application_count == 0">
                        <field name="application_count" widget="statinfo" string="Other applications"/>
                    </button>
                    <button name="action_makeMeeting" class="oe_stat_button" icon="fa-calendar" type="object" invisible="not id">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_text"><field name="meeting_display_text" /></span>
                            <span class="o_stat_value"><field name="meeting_display_date" readonly="1"/></span>
                        </div>
                    </button>
                </div>
                <widget name="web_ribbon" title="Refused" bg_color="text-bg-danger" invisible="application_status != 'refused'"/>
                <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="application_status != 'archived'"/>
                <widget name="web_ribbon" title="Hired" invisible="application_status != 'hired'" />
                <field name="active" invisible="1"/>
                <field name="legend_normal" invisible="1"/>
                <field name="legend_blocked" invisible="1"/>
                <field name="legend_done" invisible="1"/>
                <div class="oe_title pe-0">
                    <label for="name" class="oe_edit_only"/>
                    <h1 class="d-flex justify-content-between align-items-center">
                        <field name="name" options="{'line_breaks': False}" widget="text" placeholder="e.g. Sales Manager 2 year experience"/>
                        <field name="kanban_state" widget="state_selection"/>
                    </h1>
                    <h2 class="o_row">
                        <div>
                            <label for="partner_name" class="oe_edit_only"/>
                            <field name="partner_name" placeholder="e.g. John Doe"/>
                        </div>
                    </h2>
                </div>
                <group>
                    <group>
                        <field name="partner_id" invisible="1" />
                        <field name="refuse_reason_id" invisible="active"/>
                        <field name="email_from" widget="email"/>
                        <field name="email_cc" groups="base.group_no_one"/>
                        <field name="partner_phone" widget="phone"/>
                        <field name="partner_mobile" widget="phone"/>
                        <field name="linkedin_profile" widget="url"/>
                        <field name="type_id" placeholder="Degree"/>

                    </group>
                    <group>
                        <field name="interviewer_ids" options="{'no_create': True, 'no_create_edit': True}" widget="many2many_avatar_user" />
                        <field name="user_id" widget="many2one_avatar_user"/>
                        <field name="date_closed" invisible="not date_closed" />
                        <field name="priority" widget="priority"/>
                        <field name="source_id"/>
                        <field name="medium_id"/>
                        <field name="availability" placeholder="Directly Available"/>
                        <field name="categ_ids" placeholder="Tags" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                    </group>
                    <group string="Job">
                        <field name="job_id"/>
                        <field name="department_id"/>
                        <field name="company_id" groups="base.group_multi_company" invisible="1" options='{"no_open":True}' />
                    </group>
                    <group string="Contract" name="recruitment_contract">
                        <label for="salary_expected"/>
                        <div class="o_row">
                            <field name="salary_expected"/>
                            <span invisible="not salary_expected_extra"> + </span>
                            <field name="salary_expected_extra" placeholder="Extra advantages..."/>
                        </div>
                        <label for="salary_proposed"/>
                        <div class="o_row">
                            <field name="salary_proposed"/>
                            <span invisible="not salary_proposed_extra"> + </span>
                            <field name="salary_proposed_extra" placeholder="Extra advantages..."/>
                        </div>
                    </group>
                </group>
                <field name="applicant_properties" columns="2"/>
                <notebook>
                    <page string="Application Summary" name="application_summary">
                        <field name="description" placeholder="Motivations..."/>
                    </page>
                </notebook>
            </sheet>
            <div class="o_attachment_preview" groups="hr_recruitment.group_applicant_cv_display"/>
            <div class="oe_chatter">
                <field name="message_follower_ids"/>
                <field name="activity_ids"/>
                <field name="message_ids" options="{'open_attachments': True}"/>
            </div>
          </form>
        </field>
    </record>

    <record id="hr_applicant_view_form_interviewer" model="ir.ui.view">
        <field name="model">hr.applicant</field>
        <field name="inherit_id" ref="hr_applicant_view_form"/>
        <field name="priority">50</field>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//form" position="attributes">
                <attribute name="js_class">hr_recruitment_interviewer</attribute>
                <attribute name="class">o_applicant_form</attribute>
            </xpath>
            <xpath expr="//group[@name='recruitment_contract']" position="replace"/>
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
            <graph string="Cases By Stage and Estimates" sample="1">
                <field name="stage_id"/>
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
                <field name="application_status"/>
                <field name="date_closed"/>
                <field name="attachment_ids" filter_domain="[('attachment_ids', 'any', [('index_content', 'ilike', self), ('res_model', '=', 'hr.applicant')])]" string="Resume's content"/>
                <filter string="My Applications" name="my_applications" domain="[('user_id', '=', uid)]"/>
                <filter string="Unassigned" name="unassigned" domain="[('user_id', '=', False)]"/>
                <separator/>
                <filter string="In Progress" name="ongoing" domain="[('date_closed', '=', False), ('active', '=', True), ('refuse_reason_id', '=', False)]"/>
                <filter string="Hired" name="hired" domain="[('date_closed', '!=', False)]"/>
                <separator/>
                <filter string="Ready for Next Stage" name="done" domain="[('kanban_state', '=', 'done')]"/>
                <filter string="Blocked" name="blocked" domain="[('kanban_state', '=', 'blocked')]"/>
                <filter string="Reserve" name="reserve" domain="[('job_id', '=', False)]"/>
                <separator/>
                <filter string="Directly Available" name="applicant_availability" domain="['|',('availability', '&lt;=', context_today().strftime('%Y-%m-%d')),('availability','=', False)]"/>
                <separator/>
                <filter string="Creation Date" name="filter_create" date="create_date"/>
                <filter string="Last Stage Update" name="filter_date_last_stage_update" date="date_last_stage_update"/>
                <separator/>
                <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction', '=', True)]" groups="mail.group_mail_notification_type_inbox"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False), ('refuse_reason_id', '=', False)]"/>
                <filter string="Refused" name="refused" domain="[('active', '=', False), ('refuse_reason_id', '!=', False)]"/>
                <separator/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                <separator/>
                <filter invisible="1" string="Running Applicants" name="running_applicant_activities"
                    domain="[('stage_id.hired_stage', '=', False)]"/>
                <separator/>
                <group expand="0" string="Group By">
                    <filter string="Job" name="job" domain="[]" context="{'group_by': 'job_id'}"/>
                    <filter string="Stage" name="stage" domain="[]" context="{'group_by': 'stage_id'}"/>
                    <filter string="Responsible" name="responsible" domain="[]"  context="{'group_by': 'user_id'}"/>
                    <filter string="Creation Date" name="creation_date" context="{'group_by': 'create_date'}"/>
                    <filter string="Hiring Date" name="hired_date" context="{'group_by': 'date_closed'}"/>
                    <filter string="Last Stage Update" name="last_stage_update" context="{'group_by': 'date_last_stage_update'}"/>
                    <filter string="Refuse Reason" name="refuse_reason_id" domain="[]" context="{'group_by': 'refuse_reason_id'}"/>
                    <filter string="Company" name="company" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                </group>
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
                <field name="applicant_properties"/>
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
                    <field name="partner_name" placeholder="e.g. John Doe"/>
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
                <field name="date_closed"/>
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
                <field name="refuse_reason_id" />
                <field name="application_status" />
                <progressbar field="kanban_state" colors='{"done": "success", "blocked": "danger"}'/>
                <templates>
                    <t t-name="kanban-menu">
                        <a role="menuitem" name="action_makeMeeting" type="object" class="dropdown-item">Schedule Interview</a>
                        <a role="menuitem" name="archive_applicant" type="object" class="dropdown-item">Refuse</a>
                        <a role="menuitem" name="archive_applicant" type="object" class="dropdown-item">Archive</a>
                        <t t-if="widget.deletable"><a role="menuitem" type="delete" class="dropdown-item">Delete</a></t>
                        <div role="separator" class="dropdown-divider"></div>
                        <ul class="oe_kanban_colorpicker text-center" data-field="color"/>
                    </t>
                    <t t-name="kanban-box">
                        <div t-attf-class="{{!selection_mode ? 'oe_kanban_color_' + kanban_getcolor(record.color.raw_value) : ''}} oe_kanban_card oe_kanban_global_click oe_applicant_kanban oe_semantic_html_override">
                            <field name="date_closed" invisible="1"/>
                            <field name="company_id" invisible="1"/>
                            <div class="ribbon ribbon-top-right" invisible="not date_closed">
                                <span class="text-bg-success">Hired</span>
                            </div>
                            <span class="badge rounded-pill text-bg-danger float-end me-4" invisible="application_status != 'refused'">Refused</span>
                            <span class="badge rounded-pill text-bg-secondary float-end me-4" invisible="application_status != 'archived'">Archived</span>
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
                                <field name="applicant_properties" widget="properties"/>
                                <div class="o_kanban_record_bottom mt4">
                                    <div class="oe_kanban_bottom_left">
                                        <div class="float-start mr4" groups="base.group_user">
                                            <field name="priority" widget="priority"/>
                                        </div>
                                        <div class="o_kanban_inline_block mr8">
                                            <field name="activity_ids" widget="kanban_activity"/>
                                        </div>
                                    </div>
                                    <div class="oe_kanban_bottom_right">
                                        <a name="action_open_attachments" type="object">
                                            <span title='Documents'><i class='fa fa-paperclip' role="img" aria-label="Documents"/>
                                                <t t-esc="record.attachment_number.raw_value"/>
                                            </span>
                                        </a>
                                        <field name="kanban_state" widget="state_selection"/>
                                        <field name="legend_normal" invisible="1"/>
                                        <field name="legend_blocked" invisible="1"/>
                                        <field name="legend_done" invisible="1"/>
                                        <field name="user_id" widget="many2one_avatar_user"/>
                                    </div>

                                </div>
                            </div>
                            <div class="clearfix"></div>
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
                        <field name="user_id" widget="many2one_avatar_user"/>
                        <div class="flex-grow-1">
                            <field name="name" display="full" class="o_text_block"/>
                            <field name="partner_name" muted="1" display="full" class="o_text_block"/>
                        </div>
                    </div>
                </templates>
            </activity>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_hr_job_applications">
        <field name="name">Applications</field>
        <field name="res_model">hr.applicant</field>
        <field name="view_mode">kanban,tree,form,graph,calendar,pivot,activity</field>
        <field name="search_view_id" ref="hr_applicant_view_search_bis"/>
        <field name="context">{'search_default_job_id': [active_id], 'default_job_id': active_id, 'search_default_stage':1,'dialog_size':'medium'}</field>
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

    <record id="action_hr_applicant_mass_sms" model="ir.actions.act_window">
        <field name="name">Send SMS</field>
        <field name="res_model">sms.composer</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="context">{
            'default_composition_mode': 'mass',
            'default_mass_keep_log': True,
            'default_res_ids': active_ids,
        }</field>
        <field name="binding_model_id" ref="hr_recruitment.model_hr_applicant"/>
        <field name="binding_view_types">tree</field>
    </record>

    <record model="ir.actions.act_window" id="action_hr_applicant_new">
        <field name="res_model">hr.applicant</field>
        <field name="view_mode">form</field>
        <field name="context">{'default_job_id': active_id}</field>
    </record>

    <!-- Job Opportunities (menu) -->
    <record model="ir.actions.act_window" id="crm_case_categ0_act_job">
        <field name="name">Applications</field>
        <field name="res_model">hr.applicant</field>
        <field name="view_mode">kanban,tree,form,pivot,graph,calendar,activity</field>
        <field name="view_id" eval="False"/>
        <field name="search_view_id" ref="hr_applicant_view_search_bis"/>
        <field name="context">{'search_default_stage':1}</field>
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

    <record model="ir.actions.act_window.view" id="action_hr_sec_kanban_view_act_job">
        <field name="sequence" eval="1"/>
        <field name="view_mode">kanban</field>
        <field name="view_id" ref="hr_kanban_view_applicant"/>
        <field name="act_window_id" ref="crm_case_categ0_act_job"/>
    </record>

    <record model="ir.actions.act_window.view" id="action_hr_sec_tree_view_act_job">
        <field name="sequence" eval="0"/>
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
        groups="hr_recruitment.group_hr_recruitment_user,hr_recruitment.group_hr_recruitment_interviewer"
        sequence="210"/>

    <menuitem id="menu_hr_recruitment_configuration" name="Configuration" parent="menu_hr_recruitment_root"
        groups="group_hr_recruitment_user" sequence="100"/>

    <menuitem id="menu_hr_recruitment_config_jobs" name="Job Positions" parent="menu_hr_recruitment_configuration" sequence="10" />

    <menuitem id="menu_hr_recruitment_config_applications" name="Applications" parent="menu_hr_recruitment_configuration" sequence="20" />

    <menuitem id="menu_hr_recruitment_config_employees" name="Employees" parent="menu_hr_recruitment_configuration" sequence="30" />

    <menuitem id="menu_hr_recruitment_config_activities" name="Activities" parent="menu_hr_recruitment_configuration" sequence="40" />

    <menuitem
        id="menu_hr_applicant_refuse_reason"
        action="hr_applicant_refuse_reason_action"
        parent="menu_hr_recruitment_config_applications"
        sequence="10"/>

    <menuitem
        name="Applications"
        parent="menu_hr_recruitment_root"
        id="menu_crm_case_categ0_act_job" sequence="2"/>

    <menuitem
        name="All Applications"
        parent="menu_crm_case_categ0_act_job"
        id="menu_crm_case_categ_all_app" action="crm_case_categ0_act_job" sequence="2"/>

    <menuitem
        id="menu_hr_recruitment_stage"
        name="Stages"
        parent="menu_hr_recruitment_config_jobs"
        action="hr_recruitment_stage_act"
        groups="base.group_no_one"
        sequence="1"/>

    <menuitem
        id="menu_hr_recruitment_contract_type"
        action="hr.hr_contract_type_action"
        parent="menu_hr_recruitment_config_jobs"
        sequence="2"
        groups="hr.group_hr_user"/>

    <menuitem
        id="hr_applicant_category_menu"
        parent="menu_hr_recruitment_config_applications"
        action="hr_applicant_category_action"
        sequence="20" groups="base.group_no_one"/>

    <menuitem
        id="menu_hr_recruitment_degree"
        name="Degrees"
        parent="menu_hr_recruitment_config_applications"
        action="hr_recruitment_degree_action"
        sequence="1" groups="base.group_no_one"/>

    <record id="hr_applicant_view_pivot" model="ir.ui.view">
        <field name="name">hr.applicant.pivot</field>
        <field name="model">hr.applicant</field>
        <field name="arch" type="xml">
            <pivot string="Recruitment Analysis" sample="1">
                <field name="stage_id" type="row"/>
                <field name="job_id" type="col"/>
                <field name="name"/>
            </pivot>
        </field>
    </record>

    <record id="hr_applicant_view_graph" model="ir.ui.view">
        <field name="name">hr.applicant.graph</field>
        <field name="model">hr.applicant</field>
        <field name="arch" type="xml">
            <graph string="Recruitment Analysis" sample="1">
                <field name="stage_id"/>
                <field name="job_id"/>
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
                <filter string="Reserve" name="reserve" domain="[('job_id', '=', False)]"/>
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
                    <filter string="Tags" name="group_by_categ_ids" context="{'group_by':'categ_ids'}"/>
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

    <menuitem
        name="Reporting"
        id="report_hr_recruitment"
        parent="menu_hr_recruitment_root"
        groups="group_hr_recruitment_user"
        sequence="99"/>

    <menuitem
        name="Recruitment Analysis"
        id="hr_applicant_report_menu"
        parent="report_hr_recruitment"
        sequence="50"
        action="hr_applicant_action_analysis"/>

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
            'search_default_creation_month': 1,
            'search_default_job_id': [active_id],
            'default_job_id': active_id}
        </field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No data yet!
            </p>
        </field>
    </record>

    <record id="action_applicant_send_mail" model="ir.actions.server">
        <field name="name">Send Email</field>
        <field name="model_id" ref="hr_recruitment.model_hr_applicant"/>
        <field name="binding_model_id" ref="hr_recruitment.model_hr_applicant"/>
        <field name="binding_view_types">list</field>
        <field name="state">code</field>
        <field name="code">action = records.action_send_email()</field>
    </record>

    <record id="ir_actions_server_refuse_applicant" model="ir.actions.server">
        <field name="name">Refuse</field>
        <field name="model_id" ref="hr_recruitment.model_hr_applicant"/>
        <field name="binding_model_id" ref="hr_recruitment.model_hr_applicant"/>
        <field name="binding_view_types">list,form</field>
        <field name="state">code</field>
        <field name="code">
if records:
    action = records.archive_applicant()
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

    <menuitem
        id="menu_hr_recruitment_utm"
        parent="menu_hr_recruitment_configuration"
        name="UTMs"
        groups="base.group_no_one"
        sequence="15"/>

    <menuitem
        id="menu_hr_recruitment_utm_sources"
        parent="menu_hr_recruitment_utm"
        name="Sources"
        action="utm.utm_source_action"
        groups="base.group_no_one"
        sequence="15"/>

    <menuitem
        id="menu_hr_recruitment_utm_mediums"
        parent="menu_hr_recruitment_utm"
        name="Mediums"
        action="utm.utm_medium_action"
        groups="base.group_no_one"
        sequence="15"/>

    </data>
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
        <field name="arch" type="xml">
            <data>
                <xpath expr="//templates" position="before">
                    <t groups="hr_recruitment.group_hr_recruitment_user">
                        <field name="new_applicant_count"/>
                        <field name="new_hired_employee"/>
                        <field name="expected_employee"/>
                    </t>
                </xpath>

                <xpath expr="//div[hasclass('o_kanban_primary_right')]" position="inside">
                    <t groups="hr_recruitment.group_hr_recruitment_user">
                        <div t-if="record.new_applicant_count.raw_value > 0" class="row ml16">
                            <div class="col">
                                <a name="%(hr_applicant_action_from_department)d" type="action">
                                    <field name="new_applicant_count"/> New Applicants
                                </a>
                            </div>
                        </div>
                    </t>
                </xpath>

                <xpath expr="//div[hasclass('o_kanban_manage_reports')]" position="inside">
                    <a role="menuitem" class="dropdown-item" name="%(action_hr_recruitment_report_filtered_department)d"
                        type="action" groups="hr_recruitment.group_hr_recruitment_user">
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
            parent="menu_hr_recruitment_config_employees" action="action_hr_department"/>
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
            <kanban class="o_kanban_dashboard o_hr_recruitment_kanban" on_create="hr_recruitment.create_job_simple" sample="1" limit="40" action="%(action_hr_job_applications)d" type="action">
                <field name="active"/>
                <field name="name"/>
                <field name="alias_email"/>
                <field name="is_favorite"/>
                <field name="department_id"/>
                <field name="no_of_recruitment"/>
                <field name="color"/>
                <field name="new_application_count"/>
                <field name="no_of_hired_employee"/>
                <field name="manager_id"/>
                <field name="user_id"/>
                <field name="application_count"/>
                <templates>
                    <t t-name="kanban-menu" groups="hr_recruitment.group_hr_recruitment_user">
                        <div class="container">
                            <div class="row">
                                <div class="col-6 o_kanban_card_manage_section">
                                    <h5 role="menuitem" class="o_kanban_card_manage_title">
                                        <span>View</span>
                                    </h5>
                                    <div role="menuitem" name="menu_view_applications">
                                        <a name="%(action_hr_job_applications)d" type="action">Applications</a>
                                    </div>
                                    <div role="menuitem">
                                        <a name="action_open_activities" type="object">Activities</a>
                                    </div>
                                    <div role="menuitem">
                                        <a name="%(action_hr_job_sources)d" type="action" context="{'default_job_id': id}">Trackers</a>
                                    </div>
                                </div>
                                <div class="col-6 o_kanban_card_manage_section">
                                    <h5 role="menuitem" class="o_kanban_card_manage_title">
                                        <span>New</span>
                                    </h5>
                                    <div role="menuitem" name="menu_new_applications">
                                        <a name="%(hr_recruitment.action_hr_applicant_new)d" type="action">Application</a>
                                    </div>
                                </div>
                                <div class="col-6 o_kanban_card_manage_section">
                                    <h5 role="menuitem" class="o_kanban_card_manage_title">
                                        <span>Reporting</span>
                                    </h5>
                                    <div role="menuitem" name="o_kanban_job_reporting">
                                        <a name="%(hr_recruitment.action_hr_recruitment_report_filtered_job)d" type="action">Analysis</a>
                                    </div>
                                </div>
                            </div>
                            <div class="o_kanban_card_manage_settings row">
                                <div class="col-6" role="menuitem" aria-haspopup="true">
                                    <ul class="oe_kanban_colorpicker" data-field="color" role="popup"/>
                                </div>
                                <div class="col-6" role="menuitem">
                                    <a class="dropdown-item" t-if="widget.editable" name="edit_job" type="edit">Configuration</a>
                                    <a class="dropdown-item" t-if="record.active.raw_value" name="toggle_active" type="object">Archive</a>
                                    <a class="dropdown-item" t-if="!record.active.raw_value" name="toggle_active" type="object">Unarchive</a>
                                </div>
                            </div>
                        </div>
                    </t>
                    <t t-name="kanban-box">
                        <div t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''}">
                            <div class="o_kanban_card_header d-flex align-items-baseline gap-1">
                                <field name="is_favorite" widget="boolean_favorite" nolabel="1"/>
                                <div class="o_kanban_card_header_title">
                                    <div class="o_primary">
                                        <span><t t-esc="record.name.value"/></span>
                                    </div>
                                    <div class="text-muted">
                                        <field name="user_id" />
                                    </div>
                                    <div class="o_secondary" groups="base.group_multi_company">
                                        <small><i class="fa fa-building-o" role="img" aria-label="Company" title="Company"></i> <field name="company_id"/></small>
                                    </div>
                                    <div t-if="record.alias_email.value" class="o_secondary o_job_alias">
                                        <small><i class="fa fa-envelope-o" role="img" aria-label="Alias" title="Alias"></i> <field name="alias_id"/></small>
                                    </div>
                                </div>
                            </div>
                            <div class="container o_recruitment_job_container o_kanban_card_content mt-0 mt-sm-3">
                                <div class="row">
                                    <div class="col-7">
                                        <button class="btn btn-primary" name="%(action_hr_job_applications)d" type="action">
                                            <field name="new_application_count"/> New Applications
                                        </button>
                                    </div>
                                    <ul class="col-5 o_job_activities">
                                        <li>
                                            <a name="edit_job" type="edit" t-attf-class="{{ record.no_of_recruitment.raw_value > 0 ? 'text-primary fw-bolder' : 'text-secondary' }}" groups="hr_recruitment.group_hr_recruitment_user">
                                                <field name="no_of_recruitment"/> To Recruit
                                            </a>
                                            <span t-attf-class="{{ record.no_of_recruitment.raw_value > 0 ? 'text-primary fw-bolder' : 'text-secondary' }}" groups="!hr_recruitment.group_hr_recruitment_user">
                                                <field name="no_of_recruitment"/> To Recruit
                                            </span>
                                        </li>
                                        <li t-if="record.application_count.raw_value > 0">
                                            <field name="application_count"/> Applications
                                        </li>
                                        <li class="text-warning" t-if="record.activities_today.raw_value > 0">
                                            <a name="action_open_today_activities" type="object" class="text-warning"><field name="activities_today"/> Activities Today</a>
                                        </li>
                                        <li t-if="record.activities_overdue.raw_value > 0">
                                            <a name="action_open_late_activities" type="object" class="text-danger"><field name="activities_overdue"/> Late Activities</a>
                                        </li>
                                    </ul>
                                </div>
                                <div name="kanban_boxes" class="row flex-nowrap" groups="hr_recruitment.group_hr_recruitment_user"></div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>
    <record id="view_job_filter_recruitment" model="ir.ui.view">
        <field name="name">Job</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr.view_job_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='department_id']" position="after">
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
                <group>
                    <field name="name" class="o_job_name oe_inline" placeholder="e.g. Sales Manager"/>
                    <label for="alias_name" string="Application email"
                           help="Define a specific contact address for this job position. If you keep it empty, the default email address will be used which is in human resources settings"/>
                    <div name="alias_def">
                        <field name="alias_id" class="oe_read_only" string="Email Alias" required="0"/>
                        <div class="oe_edit_only" name="edit_alias">
                            <field name="alias_name" class="oe_inline o_job_alias" placeholder="e.g. sales-manager"/>@
                            <field name="alias_domain_id" class="oe_inline" placeholder="e.g. domain.com"
                                   options="{'no_create': True, 'no_open': True}"/>
                        </div>
                        <div class="text-muted">Applicants can send resume to this email address,<br/>it will create an application automatically</div>
                    </div>
                </group>
                <footer>
                    <button string="Create" type="object" name="close_dialog" class="btn-primary o_create_job" data-hotkey="q"/>
                    <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="create_job_simple" model="ir.actions.act_window">
        <field name="name">Create a Job Position</field>
        <field name="res_model">hr.job</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="hr_job_simple_form"/>
        <field name="target">new</field>
        <field name="context">{'dialog_size' : 'medium'}</field>
    </record>

    <record id="hr_job_survey" model="ir.ui.view">
        <field name="name">hr.job.form1</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr.view_hr_job_form"/>
        <field name="arch" type="xml">
            <div name="recruitment_target" position="after">
                <field name="user_id" widget="many2one_avatar_user"/>
                <field name="interviewer_ids" widget="many2many_tags_avatar" options="{'no_create': True, 'no_create_edit': True}" />
            </div>
            <xpath expr="//field[@name='department_id']" position="after">
                <label for="address_id"/>
                <div class="o_row">
                    <span invisible="address_id" class="oe_read_only">Remote</span>
                    <field name="address_id" context="{'show_address': 1}" placeholder="Remote"/>
                </div>
                <label for="alias_name" string="Email Alias"
                       help="Define a specific contact address for this job position. If you keep it empty, the default email address will be used which is in human resources settings"/>
                <div name="alias_def">
                    <field name="alias_id" class="oe_read_only" string="Email Alias" required="0"/>
                    <div class="oe_edit_only" name="edit_alias">
                        <field name="alias_name" class="oe_inline"/>@
                        <field name="alias_domain_id" class="oe_inline" placeholder="e.g. domain.com"
                               options="{'no_create': True, 'no_open': True}"/>
                    </div>
                </div>
            </xpath>
            <xpath expr="//field[@name='contract_type_id']" position="after">
                <field name="company_id" groups="base.group_multi_company"/>
            </xpath>
            <div name="button_box" position="inside">
                <button class="oe_stat_button"
                    icon="fa-pencil"
                    name="%(action_hr_job_applications)d"
                    context="{'default_user_id': user_id, 'active_test': False}"
                    type="action">
                    <field name="all_application_count" widget="statinfo" string="Job Applications"/>
                </button>
                <button class="oe_stat_button"
                    icon="fa-file-text-o"
                    name="action_open_attachments"
                    type="object"
                    invisible="documents_count == 0">
                    <field name="documents_count" widget="statinfo" string="Documents"/>
                </button>
                <button class="oe_stat_button" type="action"
                    name="%(action_hr_job_sources)d" icon="fa-bar-chart-o"
                    context="{'default_job_id': id}">
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

    <record id="hr_job_search_view" model="ir.ui.view">
        <field name="name">hr.job.search</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr_recruitment.view_job_filter_recruitment" />
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='my_favorite_jobs']" position="after">
                <searchpanel>
                    <field name="company_id" groups="base.group_multi_company" icon="fa-building" enable_counters="1"/>
                    <field name="department_id" icon="fa-users" enable_counters="1"/>
                </searchpanel>
                <filter string="My Job Positions" name="my_positions" domain="[('user_id', '=', uid)]"/>
            </xpath>
        </field>
    </record>

    <record id="hr_job_view_tree_inherit" model="ir.ui.view">
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr.view_hr_job_tree"/>
        <field name="arch" type="xml">
            <field name="department_id" position="after">
                <field name="application_count" string="Applications"/>
            </field>
            <field name="no_of_recruitment" position="after">
                <field name="alias_name" column_invisible="True"/>
                <field name="alias_id" invisible="not alias_name" optional="hide"/>
                <field name="user_id" widget="many2one_avatar_user" optional="hide"/>
            </field>
        </field>
    </record>

    <!-- hr related job position menu action -->
    <record model="ir.actions.act_window" id="action_hr_job">
        <field name="name">Job Positions</field>
        <field name="res_model">hr.job</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="view_id" ref="hr_recruitment.view_hr_job_kanban"/>
        <field name="view_ids" eval="[(5, 0, 0),
            (0, 0, {'view_mode': 'kanban', 'view_id': ref('hr_recruitment.view_hr_job_kanban')}),
            (0, 0, {'view_mode': 'tree', 'view_id': ref('hr_recruitment.hr_job_view_tree_inherit')}),
            (0, 0, {'view_mode': 'form', 'view_id': ref('hr.view_hr_job_form')})]"/>
        <field name="context">{}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
              Ready to recruit more efficiently?
          </p><p>
              Let's create a job position.
          </p>
        </field>
    </record>
    <menuitem name="By Job Positions" parent="menu_crm_case_categ0_act_job"
        id="menu_hr_job_position" action="action_hr_job"
        sequence="1" groups="hr_recruitment.group_hr_recruitment_user" />

    <!-- Job Positions filtered for interviewers -->
    <record model="ir.actions.act_window" id="action_hr_job_interviewer">
        <field name="name">Job Positions</field>
        <field name="res_model">hr.job</field>
        <field name="view_mode">kanban,form</field>
        <field name="view_id" ref="hr_recruitment.view_hr_job_kanban"/>
        <field name="context">{'create': False}</field>
        <field name="domain">[
            '|',
                ('interviewer_ids', 'in', uid),
                ('extended_interviewer_ids', 'in', uid),
        ]</field>
    </record>
    <menuitem name="By Job Positions" parent="menu_crm_case_categ0_act_job"
        id="menu_hr_job_position_interviewer" action="action_hr_job_interviewer"
        sequence="1" groups="hr_recruitment.group_hr_recruitment_interviewer" />
</odoo>

```

## File: views\hr_recruitment_degree_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
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

        <record id="hr_recruitment_degree_action" model="ir.actions.act_window">
            <field name="name">Degree</field>
            <field name="res_model">hr.recruitment.degree</field>
            <field name="view_id" ref="hr_recruitment_degree_tree"/>
        </record>
    </data>
</odoo>

```

## File: views\hr_recruitment_source_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record id="hr_recruitment_source_kanban" model="ir.ui.view">
            <field name="name">hr.recruitment.source.kanban</field>
            <field name="model">hr.recruitment.source</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile" create="0" sample="1">
                    <field name="job_id"/>
                    <field name="email"/>
                    <field name="has_domain"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div class="oe_kanban_card oe_kanban_global_click">
                                <div class="oe_kanban_content">
                                    <div class="o_kanban_record_top">
                                        <div class="o_kanban_record_headings row">
                                            <div class="col-4">
                                                <h3 class="o_kanban_record_title"><field name="source_id"/></h3>
                                            </div>
                                            <div class="col-8 text-end">
                                                <div><field name="job_id"/></div>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="o_kanban_record_body mt-3">
                                        <div class="text-end">
                                            <field name="email" invisible="not has_domain" widget="email"/>
                                            <button name="create_alias" class="btn btn-primary mb-2" type="object" invisible="not has_domain or email">Generate Email</button>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record model="ir.ui.view" id="hr_recruitment_source_tree">
            <field name="name">hr.recruitment.source.tree</field>
            <field name="model">hr.recruitment.source</field>
            <field name="arch" type="xml">
                <tree string="Sources of Applicants" editable="top" sample="1">
                    <field name="has_domain" column_invisible="True"/>
                    <field name="source_id" placeholder="e.g. LinkedIn" decoration-bf="1" readonly="id"/>
                    <field name="medium_id" optional="hidden"/>
                    <field name="job_id" readonly="id"/>
                    <field name="email" widget="email"
                           invisible="not email or not has_domain"/>
                    <button name="create_alias" string="Generate Email" class="btn btn-primary" type="object" invisible="not has_domain or email"/>
                </tree>
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

        <record model="ir.actions.act_window" id="action_hr_job_sources">
            <field name="name">Jobs Sources</field>
            <field name="res_model">hr.recruitment.source</field>
            <field name="view_mode">tree,kanban</field>
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
    </data>
</odoo>

```

## File: views\hr_recruitment_stage_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
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

        <record model="ir.ui.view" id="hr_recruitment_stage_tree">
            <field name="name">hr.recruitment.stage.tree</field>
            <field name="model">hr.recruitment.stage</field>
            <field name="arch" type="xml">
                <tree string="Stages">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                    <field name="fold"/>
                    <field name="hired_stage"/>
                </tree>
            </field>
        </record>

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
                            <field name="hired_stage"/>
                            <field name="is_warning_visible" invisible="1"/>
                            <span invisible="not is_warning_visible">
                                <span
                                    class="fa fa-exclamation-triangle text-danger ps-3">
                                </span>
                                <span class="text-danger">
                                    All applications will lose their hired date and hired status.
                                </span>
                            </span>
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
    </data>
</odoo>

```

## File: views\ir_attachment_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
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

        <record id="ir_attachment_hr_recruitment_list_view" model="ir.ui.view">
            <field name="model">ir.attachment</field>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <tree>
                    <field name="name" column_invisible="True"/>
                    <field name="res_id" column_invisible="True"/>
                    <field name="res_model" column_invisible="True"/>
                    <field name="datas" widget="binary" filename="name" string="File"/>
                    <field name="res_name" widget="applicant_char" string="Applicant" invisible="res_model != 'hr.applicant'"/>
                    <field name="create_date"/>
                </tree>
            </field>
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
        <field name="domain">['|', ('res_model', '=', False), ('res_model', '=', 'hr.applicant')]</field>
        <field name="context">{'default_res_model': 'hr.applicant'}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No data to display
            </p>
            <p>
                Try to add some records, or make sure that there is no active filter in the search bar.
            </p>
        </field>
    </record>
    <menuitem id="hr_recruitment_menu_config_activity_type"
        action="mail_activity_type_action_config_hr_applicant"
        parent="menu_hr_recruitment_config_activities"/>
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
                <xpath expr="//form" position="inside">
                    <app data-string="Recruitment" string="Recruitment" name="hr_recruitment" groups="hr_recruitment.group_hr_recruitment_manager">
                        <block title="Job Posting" name="online_posting_setting_container">
                            <setting id="publish_available_jobs_setting" string="Online Posting" help="Publish available jobs on your website">
                                <field name="module_website_hr_recruitment"/>
                            </setting>
                        </block>
                        <block title="Recruitment Process" name="recruitment_process_div">
                            <setting id="interview_forms_setting" string="Send Interview Survey" help="Send an Interview Survey to the applicant during the recruitment process" title="Use interview forms tailored to each job position during the recruitment process. Select the form to use in the job position detail form. This relies on the Survey app.">
                                <field name="module_hr_recruitment_survey"/>
                            </setting>
                            <setting id="sms" string="Send SMS" documentation="/applications/marketing/sms_marketing/pricing/pricing_and_faq.html" help="Send texts to your contacts">
                            </setting>
                            <setting string="CV Display" help="Display CV on application form" id="display_cv">
                                <field name="group_applicant_cv_display"/>
                            </setting>
                            <setting id="recruitment_extract_settings" string="CV Digitization (OCR)" company_dependent="1" help="Digitize your CV to extract name and email automatically." title="Use OCR to fill data from a picture of the CV or the file itself">
                                <field name="module_hr_recruitment_extract" widget="upgrade_boolean"/>
                            </setting>
                        </block>
                    </app>
                </xpath>
            </field>
        </record>

        <record id="action_hr_recruitment_configuration" model="ir.actions.act_window">
            <field name="name">Settings</field>
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

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class ApplicantGetRefuseReason(models.TransientModel):
    _name = 'applicant.get.refuse.reason'
    _description = 'Get Refuse Reason'

    refuse_reason_id = fields.Many2one('hr.applicant.refuse.reason', 'Refuse Reason', required=True)
    applicant_ids = fields.Many2many('hr.applicant')
    send_mail = fields.Boolean("Send Email", compute='_compute_send_mail', store=True, readonly=False)
    template_id = fields.Many2one('mail.template', string='Email Template',
        compute='_compute_send_mail', store=True, readonly=False,
        domain="[('model', '=', 'hr.applicant')]")
    applicant_without_email = fields.Text(compute='_compute_applicant_without_email',
        string='Applicant(s) not having email')
    applicant_emails = fields.Text(compute='_compute_applicant_emails')

    @api.depends('refuse_reason_id')
    def _compute_send_mail(self):
        for wizard in self:
            template = wizard.refuse_reason_id.template_id
            wizard.send_mail = bool(template)
            wizard.template_id = template

    @api.depends('applicant_ids', 'send_mail')
    def _compute_applicant_without_email(self):
        for wizard in self:
            applicants = wizard.applicant_ids.filtered(lambda x: not x.email_from and not x.partner_id.email)
            if applicants and wizard.send_mail:
                wizard.applicant_without_email = "%s\n%s" % (
                    _("The email will not be sent to the following applicant(s) as they don't have email address."),
                    "\n".join([i.partner_name or i.name for i in applicants])
                )
            else:
                wizard.applicant_without_email = False

    @api.depends('applicant_ids.email_from')
    def _compute_applicant_emails(self):
        for wizard in self:
            wizard.applicant_emails = ', '.join(a.email_from for a in wizard.applicant_ids if a.email_from)

    def action_refuse_reason_apply(self):
        if self.send_mail:
            if not self.template_id:
                raise UserError(_("Email template must be selected to send a mail"))
            if not self.applicant_ids.filtered(lambda x: x.email_from or x.partner_id.email):
                raise UserError(_("Email of the applicant is not set, email won't be sent."))
        self.applicant_ids.write({'refuse_reason_id': self.refuse_reason_id.id, 'active': False})
        if self.send_mail:
            applicants = self.applicant_ids.filtered(lambda x: x.email_from or x.partner_id.email)
            # TDE note: keeping 16.0 behavior, clean me please
            message_values = {
                'email_layout_xmlid' : 'hr_recruitment.mail_notification_light_without_background',
            }
            if len(applicants) > 1:
                applicants.with_context(active_test=True).message_mail_with_source(
                    self.template_id,
                    auto_delete_keep_log=True,
                    **message_values
                )
            else:
                applicants.with_context(active_test=True).message_post_with_source(
                    self.template_id,
                    subtype_xmlid='mail.mt_note',
                    **message_values
                )

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
                    <group col="1">
                        <field name="refuse_reason_id" widget="selection_badge" options="{'horizontal': true, 'no_create': True, 'no_open': True}"/>
                        <group invisible="not refuse_reason_id">
                            <label for="send_mail"/>
                            <div class="d-flex">
                                <field name="send_mail"/>
                                <span class="mx-2">to</span>
                                <field name="applicant_emails"/>
                            </div>
                            <field name="template_id" invisible="not send_mail" required="send_mail"/>
                            <field name="applicant_ids" invisible="1"/>
                        </group>
                    </group>
                    <div class="alert alert-danger" role="alert" invisible="not applicant_without_email">
                        <field name="applicant_without_email" class="mr4"/>
                    </div>
                    <footer>
                        <button name="action_refuse_reason_apply" string="Refuse" type="object" class="btn-primary" data-hotkey="q"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="applicant_get_refuse_reason_action" model="ir.actions.act_window">
            <field name="name">Refuse Reason</field>
            <field name="res_model">applicant.get.refuse.reason</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="applicant_get_refuse_reason_view_form"/>
            <field name="target">new</field>
        </record>
</odoo>

```

## File: wizard\applicant_send_mail.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models, _


class ApplicantSendMail(models.TransientModel):
    _name = 'applicant.send.mail'
    _inherit = 'mail.composer.mixin'
    _description = 'Send mails to applicants'

    applicant_ids = fields.Many2many('hr.applicant', string='Applications', required=True)
    author_id = fields.Many2one('res.partner', 'Author', required=True, default=lambda self: self.env.user.partner_id.id)

    @api.depends('subject')
    def _compute_render_model(self):
        self.render_model = 'hr.applicant'

    def action_send(self):
        self.ensure_one()

        without_emails = self.applicant_ids.filtered(lambda a: not a.email_from or (a.partner_id and not a.partner_id.email))
        if without_emails:
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                    'type': 'danger',
                    'message': _("The following applicants are missing an email address: %s.", ', '.join(without_emails.mapped(lambda a: a.partner_name or a.name))),
                }
            }

        if self.template_id:
            subjects = self._render_field('subject', res_ids=self.applicant_ids.ids)
            bodies = self._render_field('body', res_ids=self.applicant_ids.ids)
        else:
            subjects = {applicant.id: self.subject for applicant in self.applicant_ids}
            bodies = {applicant.id: self.body for applicant in self.applicant_ids}

        for applicant in self.applicant_ids:
            if not applicant.partner_id:
                applicant.partner_id = self.env['res.partner'].create({
                    'is_company': False,
                    'name': applicant.partner_name,
                    'email': applicant.email_from,
                    'phone': applicant.partner_phone,
                    'mobile': applicant.partner_mobile,
                })

            applicant.message_post(
                author_id=self.author_id.id,
                body=bodies[applicant.id],
                email_layout_xmlid='mail.mail_notification_light',
                message_type='comment',
                partner_ids=applicant.partner_id.ids,
                subject=subjects[applicant.id],
            )

```

## File: wizard\applicant_send_mail_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="applicant_send_mail_view_form" model="ir.ui.view">
        <field name="model">applicant.send.mail</field>
        <field name="arch" type="xml">
            <form>
                <field name="author_id" invisible="1"/>
                <field name="lang" invisible="1"/>
                <field name="render_model" invisible="1"/>
                <field name="template_id" invisible="1"/>
                <group>
                    <field name="subject" required="1"/>
                    <field name="applicant_ids" widget="many2many_tags" context="{'show_partner_name': 1}"/>
                </group>
                <field name="body" nolabel="1" class="oe-bordered-editor"
                        placeholder="Write your message here..."
                        options="{'style-inline': true, 'codeview': true, 'dynamic_placeholder': true}"
                        force_save="1"/>
                <footer>
                    <button name="action_send" string="Send" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import applicant_refuse_reason
from . import applicant_send_mail

```

