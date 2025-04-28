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
        'data/hr_recruitment_tour.xml',
        'views/hr_candidate_views.xml',
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
        'views/mail_activity_plan_views.xml',
        'views/digest_views.xml',
        'wizard/applicant_refuse_reason_views.xml',
        'wizard/applicant_send_mail_views.xml',
        'wizard/candidate_send_mail_views.xml',
        'views/menuitems.xml',
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
        'web.assets_unit_tests': [
            'hr_recruitment/static/tests/**/*',
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
        <field name="name">Does not fit the job requirements</field>
        <field name="template_id" ref="email_template_data_applicant_refuse"/>
        <field name="sequence">12</field>
    </record>
    <record id="refuse_reason_2" model="hr.applicant.refuse.reason">
        <field name="name">Refused by applicant: job fit</field>
        <field name="template_id" ref="email_template_data_applicant_not_interested"/>
        <field name="sequence">11</field>
    </record>
    <record id="refuse_reason_5" model="hr.applicant.refuse.reason">
        <field name="name">Job already fulfilled</field>
        <field name="template_id" ref="email_template_data_applicant_refuse"/>
        <field name="sequence">13</field>
    </record>
    <record id="refuse_reason_6" model="hr.applicant.refuse.reason">
        <field name="name">Duplicate</field>
        <field name="template_id" ref="email_template_data_applicant_refuse"/>
        <field name="sequence">14</field>
    </record>
    <record id="refuse_reason_7" model="hr.applicant.refuse.reason">
        <field name="name">Spam</field>
        <field name="template_id" ref="email_template_data_applicant_refuse"/>
        <field name="sequence">15</field>
    </record>
    <record id="refuse_reason_8" model="hr.applicant.refuse.reason">
        <field name="name">Refused by applicant: salary</field>
        <field name="template_id" ref="email_template_data_applicant_not_interested"/>
        <field name="sequence">10</field>
    </record>
        
    <record id="linkedin_job_platform" model="hr.job.platform">
        <field name="name">Linkedin</field>
        <field name="email">jobs-listings@linkedin.com</field>
        <field name="regex">New application:.*from (.*)</field>
    </record>

    <record id="jobsdb_job_platform" model="hr.job.platform">
        <field name="name">Jobsdb</field>
        <field name="email">cs@jobsdb.com</field>
        <field name="regex">from (.+?) for</field>
    </record>

    <record id="indeed_job_platform" model="hr.job.platform">
        <field name="name">Indeed</field>
        <field name="email">no-reply@indeed.com</field>
        <field name="regex">^([^ ]+ [^ ]+)</field>
    </record>
</data>
</odoo>

```

## File: data\hr_recruitment_demo.xml

```xml
<?xml version="1.0"?>
<odoo noupdate="1">
    <record id="base.user_demo" model="res.users">
        <field name="groups_id" eval="[
            (3, ref('hr_recruitment.group_hr_recruitment_interviewer')),
            (3, ref('hr_recruitment.group_hr_recruitment_user')),
            (3, ref('hr_recruitment.group_hr_recruitment_manager')),
        ]"/>
    </record>

    <!--Manage the job_id to get in hr.applicant-->
    <record id="hr.job_developer" model="hr.job">
        <field name="no_of_recruitment">4</field>
        <field name="no_of_hired_employee">56</field>
        <field name="user_id" ref="base.user_admin" />
    </record>
    <record id="hr.job_ceo" model="hr.job">
        <field name="no_of_hired_employee">1</field>
        <field name="user_id" ref="base.user_admin" />
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
        <field name="user_id" ref="base.user_admin" />
    </record>
    <record id="hr.job_marketing" model="hr.job">
        <field name="no_of_recruitment">3</field>
        <field name="no_of_hired_employee">2</field>
        <field name="user_id" ref="base.user_demo" />
    </record>
    <record id="hr.job_trainee" model="hr.job">
        <field name="no_of_recruitment">6</field>
        <field name="user_id" ref="base.user_admin" />
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

    <record id="hr_candidate_salesman0" model="hr.candidate">
        <field name="email_from">enrique.jones152@gmail.example.com</field>
        <field name="partner_phone">9963214587</field>
        <field name="partner_name">Enrique Jones</field>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_sales')])]"/>
        <field name="availability" eval="(DateTime.today() + relativedelta(months=3)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="hr_case_salesman0" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_salesman0"/>
        <field name="job_id" ref="hr.job_marketing"/>
        <field name="department_id" ref="hr.dep_sales"/>
        <field name="medium_id" ref="utm.utm_medium_direct"/>
        <field name="user_id" ref="base.user_demo"/>
        <field name="priority">1</field>
        <field name="stage_id" ref="stage_job2"/>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=29)"/>
        <field name="date_last_stage_update" eval="(DateTime.today() - timedelta(days=27)).strftime('%Y-%m-%d')"/>
    </record>

    <record id="hr_candidate_salesman1" model="hr.candidate">
        <field name="partner_name">Meldona Thang</field>
        <field name="email_from">meldona.thang@example.com</field>
        <field name="partner_phone">998655451</field>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_sales')])]"/>
        <field name="availability" eval="(DateTime.today() + relativedelta(months=3)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="hr_case_salesman1" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_salesman1"/>
        <field name="job_id" ref="hr.job_marketing"/>
        <field name="department_id" ref="hr.dep_sales"/>
        <field name="user_id" ref="base.user_demo"/>
        <field name="priority">1</field>
        <field name="stage_id" ref="stage_job1"/>
    </record>

    <record id="hr_candidate_dev0" model="hr.candidate">
        <field name="partner_name">Johan Duck</field>
        <field name="email_from">coincoin@gmail.example.com</field>
        <field name="partner_phone">8955545</field>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_it')])]"/>
        <field name="availability" eval="(DateTime.today() + timedelta(days=15)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="hr_case_dev0" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_dev0"/>
        <field name="job_id" ref="hr.job_developer"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="medium_id" ref="utm.utm_medium_email"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="priority">3</field>
        <field name="stage_id" ref="stage_job1"/>
    </record>

    <record id="hr_candidate_dev1" model="hr.candidate">
        <field name="partner_name">Kelly Wallant</field>
        <field name="email_from">kelly@wallant.example.com</field>
        <field name="partner_phone">879895515</field>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_it')])]"/>
    </record>
    <record id="hr_case_dev1" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_dev1"/>
        <field name="job_id" ref="hr.job_developer"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="priority">0</field>
        <field name="stage_id" ref="stage_job1"/>
    </record>

    <record id="hr_candidate_dev2" model="hr.candidate">
        <field name="partner_name">Cécile Donth</field>
        <field name="email_from">c-cile72@msn.example.com</field>
        <field name="partner_phone">98765411</field>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_it')])]"/>
    </record>
    <record id="hr_case_dev2" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_dev2"/>
        <field name="job_id" ref="hr.job_developer"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="medium_id" ref="utm.utm_medium_email"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="priority">0</field>
        <field name="stage_id" ref="stage_job1"/>
    </record>

    <record id="hr_candidate_dev3" model="hr.candidate">
        <field name="partner_name">Ohen Rizome</field>
        <field name="email_from">ohen@yahoo.example.com</field>
        <field name="partner_phone">654687987654</field>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_it')])]"/>
    </record>
    <record id="hr_case_dev3" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_dev3"/>
        <field name="job_id" ref="hr.job_developer"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="priority">0</field>
        <field name="stage_id" ref="stage_job1"/>
    </record>

    <record id="hr_candidate_traineemca0" model="hr.candidate">
        <field name="partner_name">Marie Justine</field>
        <field name="email_from">justinemarie@outlook.example.com</field>
        <field name="partner_phone">9988774455</field>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_manager')])]"/>
        <field name="availability" eval="(DateTime.today() + timedelta(days=15)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="hr_case_traineemca0" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_traineemca0"/>
        <field name="job_id" ref="hr.job_trainee"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="user_id" ref="base.user_demo"/>
        <field name="priority">2</field>
        <field name="stage_id" ref="stage_job4"/>
        <field name="partner_phone">6633225</field>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=17)"/>
        <field name="date_last_stage_update" eval="(DateTime.today() - timedelta(days=7)).strftime('%Y-%m-%d')"/>
    </record>

    <record id="hr_candidate_fresher0" model="hr.candidate">
        <field name="partner_name">Jose</field>
        <field name="email_from">the.jose@gmail.example.com</field>
        <field name="partner_phone">999666735</field>
        <field name="type_id" ref="degree_bachelor"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_it')])]"/>
    </record>
    <record id="hr_case_fresher0" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_fresher0"/>
        <field name="job_id" ref="hr.job_trainee"/>
        <field name="department_id" ref="hr.dep_administration"/>
        <field name="user_id" ref="base.user_demo"/>
        <field name="priority">0</field>
        <field name="stage_id" ref="stage_job3"/>
    </record>

    <record id="hr_candidate_mkt0" model="hr.candidate">
        <field name="partner_name">Yin Lee</field>
        <field name="email_from">yin.lee@wechat.example.com</field>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_manager')])]"/>
    </record>
    <record id="hr_case_mkt0" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_mkt0"/>
        <field name="job_id" ref="hr.job_marketing"/>
        <field name="department_id" ref="hr.dep_sales"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="stage_id" ref="stage_job1"/>
    </record>

    <record id="hr_candidate_mkt1" model="hr.candidate">
        <field name="partner_name">Hubert Blank</field>
        <field name="email_from">st-hubertus@gmail.example.com</field>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_manager')])]"/>
        <field name="availability" eval="(DateTime.today() + timedelta(days=15)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="hr_case_mkt1" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_mkt1"/>
        <field name="job_id" ref="hr.job_marketing"/>
        <field name="department_id" ref="hr.dep_sales"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="priority">3</field>
        <field name="stage_id" ref="stage_job3"/>
    </record>

    <record id="hr_candidate_yrsexperienceinphp0" model="hr.candidate">
        <field name="partner_name">John Bruno</field>
        <field name="email_from">johnnyboy@gmail.example.com</field>
        <field name="type_id" ref="degree_graduate"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_manager')])]"/>
    </record>
    <record id="hr_case_yrsexperienceinphp0" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_yrsexperienceinphp0"/>
        <field eval="(datetime.now()+relativedelta(months=-2)).strftime('%Y-%m-03 01:00:00')" name="create_date"/>
        <field name="job_id" ref="hr.job_marketing"/>
        <field name="department_id" ref="hr.dep_sales"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="stage_id" ref="stage_job5"/>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=61)"/>
        <field name="date_last_stage_update" eval="(DateTime.today() - timedelta(days=37)).strftime('%Y-%m-%d')"/>
    </record>

    <record id="hr_candidate_marketingjob0" model="hr.candidate">
        <field name="partner_name">Sandra Elvis</field>
        <field name="email_from">sandra.elvis.the.king25@gmail.example.com</field>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_reserve')])]"/>
    </record>
    <record id="hr_case_marketingjob0" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_marketingjob0"/>
        <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-08 01:00:00')" name="create_date"/>
        <field name="job_id" ref="hr.job_developer"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="user_id" ref="base.user_demo"/>
        <field name="stage_id" ref="stage_job5"/>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=34)"/>
        <field name="date_last_stage_update" eval="(DateTime.today() - timedelta(days=7)).strftime('%Y-%m-%d')"/>
    </record>

    <record id="hr_candidate_financejob0" model="hr.candidate">
        <field name="partner_name">David Armstrong</field>
        <field name="email_from">david.strongarm@gmail.example.com</field>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_reserve')])]"/>
        <field name="partner_phone">33968745</field>
        <field name="availability" eval="(DateTime.today() + relativedelta(months=3)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="hr_case_financejob0" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_financejob0"/>
        <field name="job_id" ref="hr.job_hrm"/>
        <field name="department_id" ref="hr.dep_administration"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="priority">1</field>
        <field name="stage_id" ref="stage_job2"/>
    </record>

    <record id="hr_candidate_financejob1" model="hr.candidate">
        <field name="partner_name">Joren Jacob</field>
        <field name="email_from">joren.jacob@outlook.example.com</field>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_reserve')])]"/>
        <field name="availability" eval="(DateTime.today() + timedelta(days=15)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="hr_case_financejob1" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_financejob1"/>
        <field name="job_id" ref="hr.job_hrm"/>
        <field name="department_id" ref="hr.dep_administration"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="priority">1</field>
        <field name="stage_id" ref="stage_job2"/>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=7)"/>
        <field name="date_last_stage_update" eval="(DateTime.today() - timedelta(days=3)).strftime('%Y-%m-%d')"/>
    </record>

    <record id="hr_candidate_traineemca1" model="hr.candidate">
        <field name="partner_name">Tina Augustie</field>
        <field name="email_from">tina.turner@gmail.example.com</field>
        <field name="partner_phone">9898745745</field>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_sales')])]"/>
    </record>
    <record id="hr_case_traineemca1" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_traineemca1"/>
        <field name="job_id" ref="hr.job_trainee"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="stage_id" ref="stage_job4"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=67)"/>
        <field name="date_last_stage_update" eval="(DateTime.today() - timedelta(days=45)).strftime('%Y-%m-%d')"/>
    </record>

    <record id="hr_candidate_programmer" model="hr.candidate">
        <field name="partner_name">Shane Williams</field>
        <field name="email_from">the.real.shane@gmail.example.com</field>
        <field name="partner_phone">9812398524</field>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_it')])]"/>
        <field name="availability" eval="(DateTime.today() + relativedelta(months=3)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="hr_case_programmer" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_programmer"/>
        <field name="job_id" ref="hr.job_developer"/>
        <field name="department_id" ref="hr.dep_rd"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="stage_id" ref="stage_job4"/>
        <field name="salary_expected">11000.0</field>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=13)"/>
        <field name="date_last_stage_update" eval="(DateTime.today() - timedelta(days=4)).strftime('%Y-%m-%d')"/>
    </record>

    <record id="hr_candidate_advertisement" model="hr.candidate">
        <field name="partner_name">David Billy</field>
        <field name="email_from">billy.boy12@gmail.example.com</field>
        <field name="partner_phone">9988774455</field>
        <field name="type_id" ref="degree_licenced"/>
        <field name="categ_ids" eval="[(6,0,[ref('tag_applicant_it')])]"/>
        <field name="availability" eval="(DateTime.today() + relativedelta(months=3)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="hr_case_advertisement" model="hr.applicant">
        <field name="candidate_id" ref="hr_candidate_advertisement"/>
        <field name="job_id" ref="hr.job_consultant"/>
        <field name="department_id" ref="hr.dep_ps"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="stage_id" ref="stage_job2"/>
        <field name="salary_expected">11000.0</field>
        <field name="create_date" eval="DateTime.now() - relativedelta(days=4)"/>
        <field name="date_last_stage_update" eval="(DateTime.today() - timedelta(days=2)).strftime('%Y-%m-%d')"/>
    </record>

    <record id="hr_case_dev2_cv" model="ir.attachment">
        <field name="name">Cecile_Donth_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Cecile_Donth_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_dev2"/>
    </record>

    <record id="hr_case_financejob0_cv" model="ir.attachment">
        <field name="name">David_Armstrong_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/David_Armstrong_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_financejob0"/>
    </record>

    <record id="hr_case_advertisement_cv" model="ir.attachment">
        <field name="name">David_Billy_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/David_Billy_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_advertisement"/>
    </record>

    <record id="hr_case_salesman0_cv" model="ir.attachment">
        <field name="name">Enrique_Jones_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Enrique_Jones_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_salesman0"/>
    </record>

        <record id="hr_case_mkt1_cv" model="ir.attachment">
        <field name="name">Hubert_Blank_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Hubert_Blank_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_mkt1"/>
    </record>

    <record id="hr_case_dev0_cv" model="ir.attachment">
        <field name="name">Johan_Duck_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Johan_Duck_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_dev0"/>
    </record>

    <record id="hr_case_yrsexperienceinphp0_cv" model="ir.attachment">
        <field name="name">John_Bruno_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/John_Bruno_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_yrsexperienceinphp0"/>
    </record>

    <record id="hr_case_financejob1_cv" model="ir.attachment">
        <field name="name">Joren_Jacob_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Joren_Jacob_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_financejob1"/>
    </record>

    <record id="hr_case_fresher0_cv" model="ir.attachment">
        <field name="name">Jose_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Jose_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_fresher0"/>
    </record>

    <record id="hr_case_dev1_cv" model="ir.attachment">
        <field name="name">Kelly_Wallant_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Kelly_Wallant_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_dev1"/>
    </record>

    <record id="hr_case_traineemca0_cv" model="ir.attachment">
        <field name="name">Marie_Justine_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Marie_Justine_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_traineemca0"/>
    </record>

    <record id="hr_case_salesman1_cv" model="ir.attachment">
        <field name="name">Meldona_Thang_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Meldona_Thang_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_salesman1"/>
    </record>

    <record id="hr_case_dev3_cv" model="ir.attachment">
        <field name="name">Owen_James_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Ohen_Rizome_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_dev3"/>
    </record>

    <record id="hr_case_marketingjob0_cv" model="ir.attachment">
        <field name="name">Sandra_Elvis_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Sandra_Elvis_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_marketingjob0"/>
    </record>

    <record id="hr_case_programmer_cv" model="ir.attachment">
        <field name="name">Shane_Williams_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Shane_Williams_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_programmer"/>
    </record>

    <record id="hr_case_traineemca1_cv" model="ir.attachment">
        <field name="name">Tina_Augustie_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Tina_Augustie_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_traineemca1"/>
    </record>

    <record id="hr_case_mkt0_cv" model="ir.attachment">
        <field name="name">Yin_Lee_CV.pdf</field>
        <field name="datas" type="base64" file="hr_recruitment/static/applicant_cvs/Yin_Lee_CV.pdf"></field>
        <field name="res_model">hr.candidate</field>
        <field name="res_id" ref="hr_recruitment.hr_candidate_mkt0"/>
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

    <record id="mail_activity_candidate_0" model="mail.activity">
        <field name="res_id" ref="hr_recruitment.hr_candidate_dev0" />
        <field name="res_model_id" ref="model_hr_candidate"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_email" />
        <field name="date_deadline" eval="time.strftime('%Y-%m-27 18:15:00')"/>
        <field name="summary">Send mail regarding our interview</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="mail_activity_candidate_1" model="mail.activity">
        <field name="res_id" ref="hr_recruitment.hr_candidate_dev1" />
        <field name="res_model_id" ref="model_hr_candidate"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_email" />
        <field name="date_deadline" eval="time.strftime('%Y-%m-%d')"/>
        <field name="summary">Send mail for first interview</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="mail_activity_candidate_2" model="mail.activity">
        <field name="res_id" ref="hr_recruitment.hr_candidate_salesman0" />
        <field name="res_model_id" ref="model_hr_candidate"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_email" />
        <field name="date_deadline" eval="time.strftime('%Y-%m-15 18:15:00')"/>
        <field name="summary">Send mail regarding our interview</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="mail_activity_candidate_3" model="mail.activity">
        <field name="res_id" ref="hr_recruitment.hr_candidate_traineemca0" />
        <field name="res_model_id" ref="model_hr_candidate"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_call" />
        <field name="date_deadline" eval="time.strftime('%Y-%m-10 18:15:00')"/>
        <field name="summary">Call to define real needs about training</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="mail_activity_candidate_4" model="mail.activity">
        <field name="res_id" ref="hr_recruitment.hr_candidate_yrsexperienceinphp0" />
        <field name="res_model_id" ref="model_hr_candidate"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_call" />
        <field name="date_deadline" eval="time.strftime('%Y-%m-24 18:15:00')"/>
        <field name="summary">Call to define real needs about training</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="mail_activity_candidate_5" model="mail.activity">
        <field name="res_id" ref="hr_recruitment.hr_candidate_advertisement" />
        <field name="res_model_id" ref="model_hr_candidate"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_call" />
        <field name="date_deadline" eval="time.strftime('%Y-%m-26 18:15:00')"/>
        <field name="summary">Call to schedule a second interview</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="mail_activity_candidate_6" model="mail.activity">
        <field name="res_id" ref="hr_recruitment.hr_candidate_mkt1" />
        <field name="res_model_id" ref="model_hr_candidate"/>
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

## File: data\hr_recruitment_tour.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_recruitment_tour" model="web_tour.tour">
        <field name="name">hr_recruitment_tour</field>
        <field name="sequence">230</field>
        <field name="rainbow_man_message"><![CDATA[
            <div>Great job! You hired a new colleague!</div><div>Try the Website app to publish job offers online.</div>
        ]]></field>
    </record>
</odoo>

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
        <field name="description">Job Position created</field>
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
        <field name="relation_field">department_id</field>
        <field name="default">False</field>
    </record>

</data></odoo>

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <template id="candidate_hired_template">
Employee created: <a href="#" t-att-data-oe-id="candidate.employee_id.id" data-oe-model="hr.employee"><t t-esc="candidate.employee_id.name"/></a>
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

## File: data\scenarios\hr_recruitment_scenario.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!--            -->
        <!-- Department -->
        <!--            -->
        <record id="dep_management" model="hr.department">
            <field name="name">Management</field>
            <field name="color" eval="1" />
        </record>

        <record id="dep_rd" model="hr.department">
            <field name="name">Research and development</field>
            <field name="color" eval="2" />
        </record>

        <record id="dep_marketing" model="hr.department">
            <field name="name">Marketing manager</field>
            <field name="parent_id" ref="dep_management" />
            <field name="color" eval="3" />
        </record>

        <!--                -->
        <!-- Applicant tags -->
        <!--                -->
        <record id="tag_applicant_demo" model="hr.applicant.category">
            <field name="name">Demo</field>
        </record>

        <!--      -->
        <!-- Jobs -->
        <!--      -->
        <record id="job_marketing" model="hr.job">
            <field name="name">Marketing and Community Manager</field>
            <field name="department_id" ref="dep_marketing" />
            <field name="no_of_recruitment">2</field>
            <field name="description">
                The Marketing Manager outlines the medium to long-term marketing strategies for
                global
                market segments they oversee.
                They work closely with Sales to create and manage the yearly budget.
                Additionally, they shape the product and customer portfolio aligned with the
                marketing
                strategy.
                Success in this role hinges on effective teamwork with Technical Services and Sales
                teams.

            </field>
        </record>

        <record id="job_full_stack_dev" model="hr.job">
            <field name="name">Full Stack Developer</field>
            <field name="department_id" ref="dep_rd" />
            <field name="no_of_recruitment">3</field>
            <field name="description">
                We are seeking a Full Stack Developer to join our dynamic team, bringing expertise
                in
                both front-end and back-end technologies.
                The ideal candidate will have a proven track record in developing scalable web
                applications, ensuring robust functionality,
                and delivering engaging user experiences. With a strong foundation in HTML, CSS,
                JavaScript, and server-side languages,
                alongside a keen eye for UI/UX design, this role offers the opportunity to play a
                pivotal part in shaping our
                digital presence and driving our organization's growth.
            </field>
        </record>

        <!--            -->
        <!-- Applicants -->
        <!--            -->
        <record id="scenario_candidate_helen" model="hr.candidate">
            <field name="email_from">helenlee@exampe.email.com</field>
            <field name="partner_name">Helen Lee</field>
            <field name="type_id" ref="degree_bac5" />
            <field name="categ_ids"
                eval="[Command.set([ref('tag_applicant_sales'), ref('tag_applicant_manager'), ref('tag_applicant_demo')])]" />
            <field name="availability"
                eval="DateTime.today()" />
        </record>
        <record id="scenario_applicant_macm_helen" model="hr.applicant">
            <field name="candidate_id" ref="scenario_candidate_helen" />
            <field name="priority">2</field>
            <field name="linkedin_profile">www.example.linkedin.com/in/helen.lee</field>
            <field name="job_id" ref="job_marketing" />
            <field name="user_id" ref="base.user_admin" />
            <field name="medium_id" ref="utm.utm_medium_direct" />
            <field name="department_id" ref="dep_marketing" />
            <field name="salary_expected">3100</field>
            <field name="salary_proposed">3100</field>
            <field name="stage_id" ref="stage_job5" />

            <field name="create_date" eval="DateTime.today() - relativedelta(days=20)" />
            <field name="write_date" eval="DateTime.today() - relativedelta(days=20)" />
        </record>

        <record id="scenario_candidate_enrique" model="hr.candidate">
            <field name="email_from">enrique.jones152@gmail.example.com</field>
            <field name="partner_name">Enrique Jones</field>
            <field name="type_id" ref="degree_bachelor" />
            <field name="categ_ids"
                eval="[Command.set([ref('tag_applicant_sales'), ref('tag_applicant_demo')])]" />
            <field name="availability"
                eval="DateTime.today() + relativedelta(days=10)" />
        </record>
        <record id="scenario_applicant_macm_enrique" model="hr.applicant">
            <field name="candidate_id" ref="scenario_candidate_enrique" />
            <field name="priority">2</field>
            <field name="linkedin_profile">www.example.linkedin.com/in/enrique.jones</field>
            <field name="job_id" ref="job_marketing" />
            <field name="user_id" ref="base.user_admin" />
            <field name="medium_id" ref="utm.utm_medium_direct" />
            <field name="department_id" ref="dep_marketing" />
            <field name="salary_expected">2900</field>
            <field name="stage_id" ref="stage_job3" />

            <field name="create_date" eval="DateTime.today() - relativedelta(months=2)" />
            <field name="date_last_stage_update"
                eval="DateTime.today() - relativedelta(days=1)" />
        </record>

        <record id="scenario_candidate_hannah" model="hr.candidate">
            <field name="email_from">hannah.glover@exampe.email.com</field>
            <field name="partner_name">Hannah Glover</field>
            <field name="type_id" ref="degree_licenced" />
            <field name="categ_ids"
                eval="[Command.set([ref('tag_applicant_it'), ref('tag_applicant_manager'), ref('tag_applicant_demo')])]" />
            <field name="availability"
                eval="DateTime.today() + relativedelta(months=1)" />
        </record>
        <record id="scenario_applicant_fsd_hannah" model="hr.applicant">
            <field name="candidate_id" ref="scenario_candidate_hannah" />
            <field name="priority">3</field>
            <field name="linkedin_profile">www.example.linkedin.com/in/hannah.glover</field>
            <field name="job_id" ref="job_full_stack_dev" />
            <field name="user_id" ref="base.user_admin" />
            <field name="medium_id" ref="utm.utm_medium_direct" />

            <field name="department_id" ref="dep_rd" />
            <field name="salary_expected">3800</field>
            <field name="stage_id" ref="stage_job1" />

            <field name="create_date" eval="DateTime.today() - relativedelta(days=5)" />
            <field name="date_last_stage_update"
                eval="DateTime.today() - relativedelta(days=5)" />
        </record>

        <record id="scenario_candidate_simon" model="hr.candidate">
            <field name="email_from">simon.jones@exampe.email.com</field>
            <field name="partner_name">Simon Jones</field>
            <field name="type_id" ref="degree_graduate" />
            <field name="categ_ids"
                eval="[Command.set([ref('tag_applicant_demo')])]" />
            <field name="availability"
                eval="DateTime.today()" />
        </record>
        <record id="scenario_applicant_fsd_simon" model="hr.applicant">
            <field name="candidate_id" ref="scenario_candidate_simon" />
            <field name="priority">0</field>
            <field name="linkedin_profile">www.example.linkedin.com/in/simon.jones</field>
            <field name="job_id" ref="job_full_stack_dev" />
            <field name="user_id" ref="base.user_admin" />
            <field name="medium_id" ref="utm.utm_medium_direct" />
            <field name="department_id" ref="dep_rd" />
            <field name="salary_expected">3000</field>
            <field name="stage_id" ref="stage_job1" />
            <field name="active">False</field>
            <field name="application_status">refused</field>
            <field name="refuse_reason_id" ref="refuse_reason_1"></field>

            <field name="create_date"
                eval="DateTime.today() - relativedelta(days=1)" />
            <field name="date_last_stage_update"
                eval="DateTime.today() - relativedelta(days=3)" />
        </record>

        <record id="scenario_candidate_maria" model="hr.candidate">
            <field name="email_from">maria.rodriguez@example.email.com</field>
            <field name="partner_name">Maria Rodriguez</field>
            <field name="type_id" ref="degree_graduate" />
            <field name="categ_ids"
                eval="[Command.set([ref('tag_applicant_demo')])]" />
            <field name="availability"
                eval="(DateTime.today()).strftime('%Y-%m-%d')" />
        </record>
        <record id="scenario_applicant_fsd_maria" model="hr.applicant">
            <field name="candidate_id" ref="scenario_candidate_maria" />
            <field name="priority">0</field>
            <field name="linkedin_profile">www.example.linkedin.com/in/maria.rodriguez</field>
            <field name="job_id" ref="job_full_stack_dev" />
            <field name="user_id" ref="base.user_admin" />
            <field name="medium_id" ref="utm.utm_medium_direct" />
            <field name="department_id" ref="dep_rd" />
            <field name="stage_id" ref="stage_job0" />
            <field name="applicant_notes">
                I had a discussion with Maria during the last networking event we organized. I was
                impressed by her enthusiasm for the company and I suggested to her that she should
                apply. Hopefully she is a good fit for the company🤞.
            </field>

            <field name="create_date" eval="DateTime.today()" />
            <field name="date_last_stage_update"
                eval="DateTime.today()" />
        </record>

        <!--         -->
        <!-- Résumés -->
        <!--         -->
        <record id="scenario_applicant_macm_enrique_cv" model="ir.attachment">
            <field name="name">Enrique_Jones_CV.pdf</field>
            <field name="datas" type="base64"
                file="hr_recruitment/static/applicant_cvs/Enrique_Jones_CV.pdf"></field>
            <field name="res_model">hr.candidate</field>
            <field name="res_id" ref="hr_recruitment.scenario_applicant_macm_enrique" />
        </record>

        <!-- Set the main attachment to avoid automatic sending to the OCR-->
        <record id="scenario_applicant_macm_enrique" model="hr.applicant">
            <field name="message_main_attachment_id" ref="hr_recruitment.scenario_applicant_macm_enrique_cv" />
        </record>

        <!--             -->
        <!-- Activities  -->
        <!--             -->
        <record id="scenario_applicant_macm_enrique_activity" model="mail.activity">
            <field name="res_id" ref="hr_recruitment.scenario_applicant_macm_enrique" />
            <field name="res_model_id" ref="model_hr_applicant" />
            <field name="activity_type_id" ref="mail.mail_activity_data_email" />
            <field name="date_deadline"
                eval="DateTime.today() + relativedelta(days=1)" />
            <field name="summary">Send an email with a contract propsal.</field>
            <field name="create_uid" ref="base.user_admin" />
            <field name="user_id" ref="base.user_admin" />
        </record>

        <record id="scenario_applicant_fsd_hannah_activity" model="mail.activity">
            <field name="res_id" ref="hr_recruitment.scenario_applicant_fsd_hannah" />
            <field name="res_model_id" ref="model_hr_applicant" />
            <field name="activity_type_id" ref="mail.mail_activity_data_todo" />
            <field name="date_deadline"
                eval="DateTime.today()" />
            <field name="summary">Reach out to schedule the first interview.</field>
            <field name="create_uid" ref="base.user_admin" />
            <field name="user_id" ref="base.user_admin" />
        </record>

        <record id="scenario_applicant_fsd_maria_activity" model="mail.activity">
            <field name="res_id" ref="hr_recruitment.scenario_applicant_fsd_maria" />
            <field name="res_model_id" ref="model_hr_applicant" />
            <field name="activity_type_id" ref="mail.mail_activity_data_todo" />
            <field name="date_deadline"
                eval="DateTime.today() + relativedelta(days=5)" />
            <field name="summary">Evaluate Maria's application</field>
            <field name="create_uid" ref="base.user_admin" />
            <field name="user_id" ref="base.user_admin" />
        </record>

        <!--          -->
        <!-- Messages -->
        <!--          -->

        <!-- APPLICANT HELEN -->
        <!-- it's missing the initial email that acknowledges the application (subject: your Job
    Application) -->
        <record id="scenario_applicant_macm_helen_mm_1" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="hr_recruitment.scenario_applicant_macm_helen" />
            <field name="message_type">notification</field>
            <field name="date" eval="DateTime.today() - relativedelta(days=17)" />
            <field name="author_id" ref="base.user_admin" />
            <field name="subtype_id" ref="hr_recruitment.mt_applicant_stage_changed" />
        </record>
        <record id="scenario_applicant_macm_helen_mtv_1" model="mail.tracking.value">
            <field name="field_id" ref="hr_recruitment.field_hr_applicant__stage_id" />
            <field name="old_value_char">New</field>
            <field name="new_value_char">Initial Qualification</field>
            <field name="old_value_integer" eval="ref('hr_recruitment.stage_job0')" />
            <field name="new_value_integer" eval="ref('hr_recruitment.stage_job1')" />
            <field name="mail_message_id" ref="scenario_applicant_macm_helen_mm_1" />
        </record>

        <record id="msg_scenario_applicant_macm_helen_mm_c_1" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="scenario_applicant_macm_helen" />
            <field name="body" type="html">
                <p>Dear colleagues,</p>
                <p>
                    This candidate look very promising, I will schedule an interview with her soon
                </p>
                <p>Kind regards,</p>
            </field>
            <field name="message_type">comment</field>
            <field name="subtype_id" ref="mail.mt_note" />
            <field name="author_id" ref="base.user_admin" />
            <field name="date" eval="DateTime.today() - relativedelta(days=17)" />
        </record>

        <record id="scenario_applicant_macm_helen_mm_2" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="hr_recruitment.scenario_applicant_macm_helen" />
            <field name="message_type">notification</field>
            <field name="date" eval="DateTime.today() - relativedelta(days=15)" />
            <field name="author_id" ref="base.user_admin" />
            <field name="subtype_id" ref="hr_recruitment.mt_applicant_stage_changed" />
        </record>
        <record id="scenario_applicant_macm_helen_mtv_2" model="mail.tracking.value">
            <field name="field_id" ref="hr_recruitment.field_hr_applicant__stage_id" />
            <field name="old_value_char">Initial Qualification</field>
            <field name="new_value_char">First Interview</field>
            <field name="old_value_integer" eval="ref('hr_recruitment.stage_job1')" />
            <field name="new_value_integer" eval="ref('hr_recruitment.stage_job2')" />
            <field name="mail_message_id" ref="scenario_applicant_macm_helen_mm_2" />
        </record>

        <record id="msg_scenario_applicant_macm_helen_mm_c_2" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="scenario_applicant_macm_helen" />
            <field name="body" type="html">
                <p>Dear colleagues,</p>
                <p>
                    I had the first interview with this candidate and was thoroughly impressed
                </p>
                <p>Kind regards,</p>
            </field>
            <field name="message_type">comment</field>
            <field name="subtype_id" ref="mail.mt_note" />
            <field name="author_id" ref="base.user_admin" />
            <field name="date" eval="DateTime.today() - relativedelta(days=14)" />
        </record>

        <record id="scenario_applicant_macm_helen_mm_3" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="hr_recruitment.scenario_applicant_macm_helen" />
            <field name="message_type">notification</field>
            <field name="date" eval="DateTime.today() - relativedelta(days=10)" />
            <field name="author_id" ref="base.user_admin" />
            <field name="subtype_id" ref="hr_recruitment.mt_applicant_stage_changed" />
        </record>
        <record id="scenario_applicant_macm_helen_mtv_3" model="mail.tracking.value">
            <field name="field_id" ref="hr_recruitment.field_hr_applicant__stage_id" />
            <field name="old_value_char">First Interview</field>
            <field name="new_value_char">Second Interview</field>
            <field name="old_value_integer" eval="ref('hr_recruitment.stage_job2')" />
            <field name="new_value_integer" eval="ref('hr_recruitment.stage_job3')" />
            <field name="mail_message_id" ref="scenario_applicant_macm_helen_mm_3" />
        </record>

        <record id="msg_scenario_applicant_macm_helen_mm_c_3" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="scenario_applicant_macm_helen" />
            <field name="body" type="html">
                <p>Dear colleagues,</p>
                <p>
                    I conducted the second interview and somehow managed to be even more impressed.
                    I think this candidate is a perfect fit for the company. We should offer her a
                    position with the salary she is expecting.
                </p>
                <p>Kind regards,</p>
            </field>
            <field name="message_type">comment</field>
            <field name="subtype_id" ref="mail.mt_note" />
            <field name="author_id" ref="base.user_admin" />
            <field name="date" eval="DateTime.today() - relativedelta(days=9)" />
        </record>

        <record id="scenario_applicant_macm_helen_mm_4" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="hr_recruitment.scenario_applicant_macm_helen" />
            <field name="message_type">notification</field>
            <field name="date" eval="DateTime.today() - relativedelta(days=7)" />
            <field name="author_id" ref="base.user_admin" />
            <field name="subtype_id" ref="hr_recruitment.mt_applicant_stage_changed" />
        </record>
        <record id="scenario_applicant_macm_helen_mtv_4" model="mail.tracking.value">
            <field name="field_id" ref="hr_recruitment.field_hr_applicant__stage_id" />
            <field name="old_value_char">Second Interview</field>
            <field name="new_value_char">Contract Proposal</field>
            <field name="old_value_integer" eval="ref('hr_recruitment.stage_job3')" />
            <field name="new_value_integer" eval="ref('hr_recruitment.stage_job4')" />
            <field name="mail_message_id" ref="scenario_applicant_macm_helen_mm_4" />
        </record>

        <record id="msg_scenario_applicant_macm_helen_mm_c_4" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="scenario_applicant_macm_helen" />
            <field name="body" type="html">
                <p>Dear colleagues,</p>
                <p>
                    I sent her a contract proposal, hopefully she will accept it.
                </p>
                <p>Kind regards,</p>
            </field>
            <field name="message_type">comment</field>
            <field name="subtype_id" ref="mail.mt_note" />
            <field name="author_id" ref="base.user_admin" />
            <field name="date" eval="DateTime.today() - relativedelta(days=7)" />
        </record>

        <record id="scenario_applicant_macm_helen_mm_5" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="hr_recruitment.scenario_applicant_macm_helen" />
            <field name="message_type">notification</field>
            <field name="date" eval="DateTime.today() - relativedelta(days=2)" />
            <field name="author_id" ref="base.user_admin" />
            <field name="subtype_id" ref="hr_recruitment.mt_applicant_stage_changed" />
        </record>
        <record id="scenario_applicant_macm_helen_mtv_5" model="mail.tracking.value">
            <field name="field_id" ref="hr_recruitment.field_hr_applicant__stage_id" />
            <field name="old_value_char">Contract Proposal</field>
            <field name="new_value_char">Contract Signed</field>
            <field name="old_value_integer" eval="ref('hr_recruitment.stage_job4')" />
            <field name="new_value_integer" eval="ref('hr_recruitment.stage_job5')" />
            <field name="mail_message_id" ref="scenario_applicant_macm_helen_mm_5" />
        </record>

        <!-- APPLICANT ENRIQUE -->
        <!-- it's missing the initial email that acknowledges the application (subject: your Job
    Application) -->
        <record id="scenario_applicant_macm_enrique_mm_1" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="hr_recruitment.scenario_applicant_macm_enrique" />
            <field name="message_type">notification</field>
            <field name="date" eval="DateTime.today() - relativedelta(months=1, days=17)" />
            <field name="author_id" ref="base.user_admin" />
            <field name="subtype_id" ref="hr_recruitment.mt_applicant_stage_changed" />
        </record>
        <record id="scenario_applicant_macm_enrique_mtv_1" model="mail.tracking.value">
            <field name="field_id" ref="hr_recruitment.field_hr_applicant__stage_id" />
            <field name="old_value_char">New</field>
            <field name="new_value_char">Initial Qualification</field>
            <field name="old_value_integer" eval="ref('hr_recruitment.stage_job0')" />
            <field name="new_value_integer" eval="ref('hr_recruitment.stage_job1')" />
            <field name="mail_message_id" ref="scenario_applicant_macm_enrique_mm_1" />
        </record>

        <record id="msg_scenario_applicant_macm_enrique_mm_c_1" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="scenario_applicant_macm_enrique" />
            <field name="body" type="html">
                <p>Dear colleagues,</p>
                <p>
                    Please review Enrique's application materials carefully. He has a strong
                    background
                    in sales and a proven track record. His experience aligns well with our current
                    needs.
                </p>
                <p>Kind regards,</p>
            </field>
            <field name="message_type">comment</field>
            <field name="subtype_id" ref="mail.mt_note" />
            <field name="author_id" ref="base.user_admin" />
            <field name="date" eval="DateTime.today() - relativedelta(months=1, days=17)" />
        </record>

        <record id="scenario_applicant_macm_enrique_mm_2" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="hr_recruitment.scenario_applicant_macm_enrique" />
            <field name="message_type">notification</field>
            <field name="date" eval="DateTime.today() - relativedelta(months=1, days=10)" />
            <field name="author_id" ref="base.user_admin" />
            <field name="subtype_id" ref="hr_recruitment.mt_applicant_stage_changed" />
        </record>
        <record id="scenario_applicant_macm_enrique_mtv_2" model="mail.tracking.value">
            <field name="field_id" ref="hr_recruitment.field_hr_applicant__stage_id" />
            <field name="old_value_char">Initial Qualification</field>
            <field name="new_value_char">First Interview</field>
            <field name="old_value_integer" eval="ref('hr_recruitment.stage_job1')" />
            <field name="new_value_integer" eval="ref('hr_recruitment.stage_job2')" />
            <field name="mail_message_id" ref="scenario_applicant_macm_enrique_mm_2" />
        </record>

        <record id="msg_scenario_applicant_macm_enrique_mm_c_2" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="scenario_applicant_macm_enrique" />
            <field name="body" type="html">
                <p>Dear colleagues,</p>
                <p>
                    The first interview went smoothly. He demonstrated excellent problem-solving
                    skills
                    and a deep understanding of our industry. Let's discuss how we can proceed based
                    on
                    his performance
                </p>
                <p>Kind regards,</p>
            </field>
            <field name="message_type">comment</field>
            <field name="subtype_id" ref="mail.mt_note" />
            <field name="author_id" ref="base.user_admin" />
            <field name="date" eval="DateTime.today() - relativedelta(months=1, days=9)" />
        </record>

        <record id="scenario_applicant_macm_enrique_mm_3" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="hr_recruitment.scenario_applicant_macm_enrique" />
            <field name="message_type">notification</field>
            <field name="date" eval="DateTime.today() - relativedelta(days=20)" />
            <field name="author_id" ref="base.user_admin" />
            <field name="subtype_id" ref="hr_recruitment.mt_applicant_stage_changed" />
        </record>
        <record id="scenario_applicant_macm_enrique_mtv_3" model="mail.tracking.value">
            <field name="field_id" ref="hr_recruitment.field_hr_applicant__stage_id" />
            <field name="old_value_char">First Interview</field>
            <field name="new_value_char">Second Interview</field>
            <field name="old_value_integer" eval="ref('hr_recruitment.stage_job2')" />
            <field name="new_value_integer" eval="ref('hr_recruitment.stage_job3')" />
            <field name="mail_message_id" ref="scenario_applicant_macm_enrique_mm_3" />
        </record>

        <record id="msg_scenario_applicant_macm_enrique_mm_c_3" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="scenario_applicant_macm_enrique" />
            <field name="body" type="html">
                <p>Dear colleagues,</p>
                <p>
                    After the second interview, it's clear that Enrique would be a valuable addition
                    to
                    our team. His insights during the discussion were impressive, and he showed
                    great
                    enthusiasm for the role. I suggest we propose him a contract with the salary he
                    indicated he wanted.
                </p>
                <p>Kind regards,</p>
            </field>
            <field name="message_type">comment</field>
            <field name="subtype_id" ref="mail.mt_note" />
            <field name="author_id" ref="base.user_admin" />
            <field name="date" eval="DateTime.today() - relativedelta(days=15)" />
        </record>

        <!-- APPLICANT HANNAH GLOVER -->
        <!-- it's missing the initial email that acknowledges the application (subject: your Job
    Application) -->
        <record id="scenario_applicant_fsd_hannah_mm_1" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="hr_recruitment.scenario_applicant_fsd_hannah" />
            <field name="message_type">notification</field>
            <field name="date" eval="DateTime.today() - relativedelta(days=17)" />
            <field name="author_id" ref="base.user_admin" />
            <field name="subtype_id" ref="hr_recruitment.mt_applicant_stage_changed" />
        </record>
        <record id="scenario_applicant_fsd_hannah_mtv_1" model="mail.tracking.value">
            <field name="field_id" ref="hr_recruitment.field_hr_applicant__stage_id" />
            <field name="old_value_char">New</field>
            <field name="new_value_char">Initial Qualification</field>
            <field name="old_value_integer" eval="ref('hr_recruitment.stage_job0')" />
            <field name="new_value_integer" eval="ref('hr_recruitment.stage_job1')" />
            <field name="mail_message_id" ref="scenario_applicant_fsd_hannah_mm_1" />
        </record>

        <record id="scenario_applicant_fsd_hannah_mm_c_1" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="scenario_applicant_fsd_hannah" />
            <field name="body" type="html">
                <p>Dear team,</p>
                <p>
                    I wanted to share my initial impressions of Hannah Glover after reading through
                    her
                    CV and portfolio website. Her portfolio and experience have truly caught my
                    attention, showcasing a deep understanding of both front-end and back-end
                    technologies. I'm particularly impressed by her ability to solve complex
                    problems
                    and her enthusiasm for learning new technologies. I believe she would be a
                    valuable
                    addition to our team. We should schedule an interview with her ASAP!
                </p>
            </field>
            <field name="message_type">comment</field>
            <field name="subtype_id" ref="mail.mt_note" />
            <field name="author_id" ref="base.user_admin" />
            <field name="date" eval="DateTime.today() - relativedelta(days=2)" />
        </record>

        <!-- APPLICANT SIMON JONES -->
        <!-- it's missing the initial email that acknowledges the application (subject: your Job
    Application) -->
        <record id="scenario_applicant_fsd_simon_mm_1" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="hr_recruitment.scenario_applicant_fsd_simon" />
            <field name="message_type">notification</field>
            <field name="date" eval="DateTime.today() - relativedelta(days=17)" />
            <field name="author_id" ref="base.user_admin" />
            <field name="subtype_id" ref="hr_recruitment.mt_applicant_stage_changed" />
        </record>
        <record id="scenario_applicant_fsd_simon_mtv_1" model="mail.tracking.value">
            <field name="field_id" ref="hr_recruitment.field_hr_applicant__stage_id" />
            <field name="old_value_char">New</field>
            <field name="new_value_char">Initial Qualification</field>
            <field name="old_value_integer" eval="ref('hr_recruitment.stage_job0')" />
            <field name="new_value_integer" eval="ref('hr_recruitment.stage_job1')" />
            <field name="mail_message_id" ref="scenario_applicant_fsd_simon_mm_1" />
        </record>

        <record id="scenario_applicant_fsd_simon_mm_c_1" model="mail.message">
            <field name="model">hr.applicant</field>
            <field name="res_id" ref="scenario_applicant_fsd_simon" />
            <field name="body" type="html">
                <p>
                    After reviewing Simon's application, it appears that his skill set is not
                    sufficiently advanced, and he has minimal experience. Should we place him on our
                    reserve list, if by any chance we require a junior developer in the near future?
                </p>
            </field>
            <field name="message_type">comment</field>
            <field name="subtype_id" ref="mail.mt_note" />
            <field name="author_id" ref="base.user_admin" />
            <field name="date" eval="DateTime.today() - relativedelta(days=2)" />
        </record>
        <!-- It's missing the refusal email -->

    </data>
</odoo>
```

## File: models\calendar.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class CalendarEvent(models.Model):
    _inherit = 'calendar.event'

    @api.model
    def default_get(self, fields):
        self_ctx = self
        if self.env.context.get('default_applicant_id'):
            self_ctx = self.with_context(
                default_res_model='hr.applicant',  # res_model seems to be lost without this
                default_res_model_id=self.env.ref('hr_recruitment.model_hr_applicant').id,
                default_res_id=self.env.context.get('default_applicant_id'),
                default_partner_ids=self.env.context.get('default_partner_ids'),
                default_name=self.env.context.get('default_name')
            )
        elif self.env.context.get('default_candidate_id'):
            self_ctx = self.with_context(
                default_res_model='hr.candidate',  # res_model seems to be lost without this
                default_res_model_id=self.env.ref('hr_recruitment.model_hr_candidate').id,
                default_res_id=self.env.context.get('default_candidate_id'),
                default_partner_ids=self.env.context.get('default_partner_ids'),
                default_name=self.env.context.get('default_name')
            )

        defaults = super(CalendarEvent, self_ctx).default_get(fields)

        # sync res_model / res_id to opportunity id (aka creating meeting from lead chatter)
        if 'applicant_id' not in defaults:
            res_model = defaults.get('res_model', False) or self_ctx.env.context.get('default_res_model')
            res_model_id = defaults.get('res_model_id', False) or self_ctx.env.context.get('default_res_model_id')
            if (res_model and res_model == 'hr.applicant') or (res_model_id and self_ctx.env['ir.model'].sudo().browse(res_model_id).model == 'hr.applicant'):
                defaults['applicant_id'] = defaults.get('res_id', False) or self_ctx.env.context.get('default_res_id', False)

        return defaults

    applicant_id = fields.Many2one('hr.applicant', string="Applicant", index='btree_not_null', ondelete='set null')
    candidate_id = fields.Many2one(
        'hr.candidate',
        string="Candidate",
        compute="_compute_candidate_id",
        store=True,
        readonly=False,
        index='btree_not_null',
        ondelete='set null')

    @api.model_create_multi
    def create(self, vals_list):
        events = super().create(vals_list)
        if not self.env['hr.applicant'].has_access('read'):
            return events

        attachments = False
        if "default_applicant_id" in self.env.context:
            attachments = self.env['hr.applicant'].browse(self.env.context['default_applicant_id']).attachment_ids
        elif "default_candidate_id" in self.env.context:
            attachments = self.env['hr.candidate'].browse(self.env.context['default_candidate_id']).attachment_ids
        if attachments:
            self.env['ir.attachment'].create([{
                'name': att.name,
                'type': 'binary',
                'datas': att.datas,
                'res_model': event._name,
                'res_id': event.id
            } for event in events for att in attachments])
        return events

    def _compute_is_highlighted(self):
        super()._compute_is_highlighted()
        applicant_id = self.env.context.get('active_id')
        if self.env.context.get('active_model') == 'hr.applicant' and applicant_id:
            for event in self:
                if event.applicant_id.id == applicant_id:
                    event.is_highlighted = True

    @api.depends('applicant_id')
    def _compute_candidate_id(self):
        for event in self:
            if not event.applicant_id:
                continue
            event.candidate_id = event.applicant_id.candidate_id

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
        res = super()._compute_kpis_actions(company, user)
        res['kpi_hr_recruitment_new_colleagues'] = f"hr.open_view_employee_list_my?menu_id={self.env.ref('hr.menu_hr_root').id}"
        return res

```

## File: models\hr_applicant.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import re

from markupsafe import Markup
from dateutil.relativedelta import relativedelta
from datetime import datetime

from odoo import api, fields, models, tools
from odoo.exceptions import UserError, ValidationError
from odoo.osv import expression
from odoo.tools.translate import _


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
               'mail.activity.mixin',
               'utm.mixin',
               'mail.tracking.duration.mixin',
    ]
    _rec_name = "partner_name"
    _mailing_enabled = True
    _primary_email = 'email_from'
    _track_duration_field = 'stage_id'

    active = fields.Boolean("Active", default=True, help="If the active field is set to false, it will allow you to hide the case without removing it.", index=True)

    candidate_id = fields.Many2one('hr.candidate', required=True, index=True)
    partner_id = fields.Many2one(related="candidate_id.partner_id")
    partner_name = fields.Char(compute="_compute_partner_name", search="_search_partner_name", inverse="_inverse_name", compute_sudo=True)
    email_from = fields.Char(related="candidate_id.email_from", readonly=False)
    email_normalized = fields.Char(related="candidate_id.email_normalized")
    partner_phone = fields.Char(related="candidate_id.partner_phone", readonly=False)
    partner_phone_sanitized = fields.Char(related="candidate_id.partner_phone_sanitized")
    linkedin_profile = fields.Char(related="candidate_id.linkedin_profile", readonly=False)
    type_id = fields.Many2one(related="candidate_id.type_id", readonly=False)
    availability = fields.Date(related="candidate_id.availability", readonly=False)
    color = fields.Integer(related="candidate_id.color")
    employee_id = fields.Many2one(related="candidate_id.employee_id", readonly=False)
    emp_is_active = fields.Boolean(related="candidate_id.emp_is_active")
    employee_name = fields.Char(related="candidate_id.employee_name")

    probability = fields.Float("Probability")
    create_date = fields.Datetime("Applied on", readonly=True)
    stage_id = fields.Many2one('hr.recruitment.stage', 'Stage', ondelete='restrict', tracking=True,
                               compute='_compute_stage', store=True, readonly=False,
                               domain="['|', ('job_ids', '=', False), ('job_ids', '=', job_id)]",
                               copy=False, index=True,
                               group_expand='_read_group_stage_ids')
    last_stage_id = fields.Many2one('hr.recruitment.stage', "Last Stage",
                                    help="Stage of the applicant before being in the current stage. Used for lost cases analysis.")
    categ_ids = fields.Many2many('hr.applicant.category', string="Tags", compute='_compute_categ_ids', store=True, readonly=False)
    company_id = fields.Many2one('res.company', "Company", compute='_compute_company', store=True, readonly=False, tracking=True)
    user_id = fields.Many2one(
        'res.users', "Recruiter", compute='_compute_user', domain="[('share', '=', False), ('company_ids', 'in', company_id)]",
        tracking=True, store=True, readonly=False)
    date_closed = fields.Datetime("Hire Date", compute='_compute_date_closed', store=True, readonly=False, tracking=True, copy=False)
    date_open = fields.Datetime("Assigned", readonly=True)
    date_last_stage_update = fields.Datetime("Last Stage Update", index=True, default=fields.Datetime.now)
    priority = fields.Selection(AVAILABLE_PRIORITIES, "Evaluation", default='0')
    job_id = fields.Many2one('hr.job', "Job Position", domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", tracking=True, index=True)
    salary_proposed_extra = fields.Char("Proposed Salary Extra", help="Salary Proposed by the Organisation, extra advantages", tracking=True, groups="hr_recruitment.group_hr_recruitment_user")
    salary_expected_extra = fields.Char("Expected Salary Extra", help="Salary Expected by Applicant, extra advantages", tracking=True, groups="hr_recruitment.group_hr_recruitment_user")
    salary_proposed = fields.Float("Proposed", aggregator="avg", help="Salary Proposed by the Organisation", tracking=True, groups="hr_recruitment.group_hr_recruitment_user")
    salary_expected = fields.Float("Expected", aggregator="avg", help="Salary Expected by Applicant", tracking=True, groups="hr_recruitment.group_hr_recruitment_user")
    department_id = fields.Many2one(
        'hr.department', "Department", compute='_compute_department', store=True, readonly=False,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", tracking=True)
    day_open = fields.Float(compute='_compute_day', string="Days to Open", compute_sudo=True)
    day_close = fields.Float(compute='_compute_day', string="Days to Close", compute_sudo=True)
    delay_close = fields.Float(compute="_compute_delay", string='Delay to Close', readonly=True, aggregator="avg", help="Number of days to close", store=True)
    user_email = fields.Char(related='user_id.email', string="User Email", readonly=True)
    attachment_number = fields.Integer(compute='_get_attachment_number', string="Number of Attachments")
    attachment_ids = fields.One2many('ir.attachment', 'res_id', domain=[('res_model', '=', 'hr.applicant')], string='Attachments')
    kanban_state = fields.Selection([
        ('normal', 'Grey'),
        ('done', 'Green'),
        ('blocked', 'Red')], string='Kanban State',
        copy=False, default='normal', required=True)
    legend_blocked = fields.Char(related='stage_id.legend_blocked', string='Kanban Blocked')
    legend_done = fields.Char(related='stage_id.legend_done', string='Kanban Valid')
    legend_normal = fields.Char(related='stage_id.legend_normal', string='Kanban Ongoing')
    refuse_reason_id = fields.Many2one('hr.applicant.refuse.reason', string='Refuse Reason', tracking=True)
    meeting_ids = fields.One2many('calendar.event', 'applicant_id', 'Meetings')
    meeting_display_text = fields.Char(compute='_compute_meeting_display')
    meeting_display_date = fields.Date(compute='_compute_meeting_display')
    # UTMs - enforcing the fact that we want to 'set null' when relation is unlinked
    campaign_id = fields.Many2one(ondelete='set null')
    medium_id = fields.Many2one(ondelete='set null', help="This displays how the applicant has reached out, e.g. via Email, LinkedIn, Website, etc.")
    source_id = fields.Many2one(ondelete='set null')
    interviewer_ids = fields.Many2many('res.users', 'hr_applicant_res_users_interviewers_rel',
        string='Interviewers', index=True, tracking=True, copy=False,
        domain="[('share', '=', False), ('company_ids', 'in', company_id)]")
    application_status = fields.Selection([
        ('ongoing', 'Ongoing'),
        ('hired', 'Hired'),
        ('refused', 'Refused'),
        ('archived', 'Archived'),
    ], compute="_compute_application_status", search="_search_application_status")
    other_applications_count = fields.Integer(compute='_compute_other_applications_count', compute_sudo=True)
    applicant_properties = fields.Properties('Properties', definition='job_id.applicant_properties_definition', copy=True)
    applicant_notes = fields.Html()
    refuse_date = fields.Datetime('Refuse Date')

    def init(self):
        super().init()
        self.env.cr.execute("""
            CREATE INDEX IF NOT EXISTS hr_applicant_job_id_stage_id_idx
            ON hr_applicant(job_id, stage_id)
            WHERE active IS TRUE
        """)

    @api.depends("candidate_id.partner_name")
    def _compute_partner_name(self):
        for applicant in self:
            applicant.partner_name = applicant.candidate_id.partner_name

    def _search_partner_name(self, operator, value):
        return [('candidate_id.partner_name', operator, value)]

    def _inverse_name(self):
        for applicant in self:
            if applicant.partner_name and not applicant.candidate_id:
                applicant.candidate_id = self.env['hr.candidate'].create({'partner_name': applicant.partner_name})
            else:
                applicant.candidate_id.partner_name = applicant.partner_name

    @api.depends('candidate_id')
    def _compute_other_applications_count(self):
        for applicant in self:
            same_candidate_applications = max(len(applicant.with_context(active_test=False).candidate_id.applicant_ids) - 1, 0)
            if applicant.candidate_id:
                domain = applicant.candidate_id._get_similar_candidates_domain()
                similar_candidates = self.env['hr.candidate'].with_context(active_test=False).search(domain) - applicant.candidate_id
                similar_candidate_applications = sum(len(candidate.applicant_ids) for candidate in similar_candidates)
                applicant.other_applications_count = similar_candidate_applications + same_candidate_applications
            else:
                applicant.other_applications_count = same_candidate_applications

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

    @api.depends('candidate_id')
    def _compute_categ_ids(self):
        for applicant in self:
            applicant.categ_ids = applicant.candidate_id.categ_ids.ids + applicant.categ_ids.ids

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

    def _search_application_status(self, operator, value):
        supported_operators = ['=', '!=', 'in', 'not in']
        if operator not in supported_operators:
            raise UserError(_('Operation not supported'))

        # Normalize value to be a list to simplify processing
        if isinstance(value, (str, bool)):
            value = [value]

        # Ensure all values are either correct strings or False
        valid_statuses = ['ongoing', 'hired', 'refused', 'archived']
        if not all(v in valid_statuses or v is False for v in value):
            raise UserError(_('Some values do not exist in the application status'))

        # Map statuses to domain filters
        for status in value:
            if status == 'refused':
                domain = [('refuse_reason_id', '!=', None)]
            elif status == 'hired':
                domain = [('date_closed', '!=', False)]
            elif status == 'archived' or status is False:
                domain = [('active', '=', False)]
            elif status == 'ongoing':
                domain = ['&', ('active', '=', True), ('date_closed', '=', False)]

        # Invert the domain for '!=' and 'not in' operators
        if operator in expression.NEGATIVE_TERM_OPERATORS:
            domain.insert(0, expression.NOT_OPERATOR)
            domain = expression.distribute_not(domain)
        return domain

    def _get_attachment_number(self):
        read_group_res = self.env['ir.attachment']._read_group(
            [('res_model', '=', 'hr.applicant'), ('res_id', 'in', self.ids)],
            ['res_id'], ['__count'])
        attach_data = dict(read_group_res)
        for record in self:
            record.attachment_number = attach_data.get(record.id, 0)

    @api.model
    def _read_group_stage_ids(self, stages, domain):
        # retrieve job_id from the context and write the domain: ids + contextual columns (job or default)
        job_id = self._context.get('default_job_id')
        search_domain = [('job_ids', '=', False)]
        if job_id:
            search_domain = ['|', ('job_ids', '=', job_id)] + search_domain
        if stages:
            search_domain = ['|', ('id', 'in', stages.ids)] + search_domain

        stage_ids = stages.sudo()._search(search_domain, order=stages._order)
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

    def _phone_get_number_fields(self):
        """ This method returns the fields to use to find the number to use to
        send an SMS on a record. """
        return ['partner_phone']

    @api.depends('stage_id.hired_stage')
    def _compute_date_closed(self):
        for applicant in self:
            if applicant.stage_id and applicant.stage_id.hired_stage and not applicant.date_closed:
                applicant.date_closed = fields.datetime.now()
            if not applicant.stage_id.hired_stage:
                applicant.date_closed = False

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if vals.get('user_id'):
                vals['date_open'] = fields.Datetime.now()
            if vals.get('email_from'):
                vals['email_from'] = vals['email_from'].strip()
        applicants = super().create(vals_list)
        applicants.sudo().interviewer_ids._create_recruitment_interviewers()

        if (applicants.interviewer_ids.partner_id - self.env.user.partner_id):
            for applicant in applicants:
                interviewers_to_notify = applicant.interviewer_ids.partner_id - self.env.user.partner_id
                notification_subject = _("You have been assigned as an interviewer for %s", applicant.display_name)
                notification_body = _("You have been assigned as an interviewer for the Applicant %s", applicant.partner_name)
                applicant.message_notify(
                    res_id=applicant.id,
                    model=applicant._name,
                    partner_ids=interviewers_to_notify.ids,
                    author_id=self.env.user.partner_id.id,
                    email_from=self.env.user.email_formatted,
                    subject=notification_subject,
                    body=notification_body,
                    email_layout_xmlid="mail.mail_notification_layout",
                    record_name=applicant.display_name,
                    model_description="Applicant",
                )
        # Copy CV from candidate to applicant at record creation
        attachments_result = self.env['ir.attachment'].read_group([
            ('res_id', 'in', applicants.candidate_id.ids),
            ('res_model', '=', "hr.candidate")
        ], ['ids:array_agg(id)'], groupby=['res_id'])
        attachments_by_candidate = {e['res_id']: e['ids'] for e in attachments_result}
        for applicant in applicants:
            if applicant.candidate_id.company_id and applicant.company_id != applicant.candidate_id.company_id:
                raise ValidationError(_("You cannot create an applicant in a different company than the candidate"))
            candidate_id = applicant.candidate_id.id
            if candidate_id not in attachments_by_candidate:
                continue
            self.env['ir.attachment'].browse(attachments_by_candidate[candidate_id]).copy({
                'res_id': applicant.id,
                'res_model': 'hr.applicant'
            })
        return applicants

    def write(self, vals):
        # user_id change: update date_open
        if vals.get('user_id'):
            vals['date_open'] = fields.Datetime.now()
        old_interviewers = self.interviewer_ids
        # stage_id: track last stage before update
        if 'stage_id' in vals:
            vals['date_last_stage_update'] = fields.Datetime.now()
            if 'kanban_state' not in vals:
                vals['kanban_state'] = 'normal'
            for applicant in self:
                vals['last_stage_id'] = applicant.stage_id.id
                res = super().write(vals)
        else:
            res = super().write(vals)
        if 'interviewer_ids' in vals:
            interviewers_to_clean = old_interviewers - self.interviewer_ids
            interviewers_to_clean._remove_recruitment_interviewers()
            self.sudo().interviewer_ids._create_recruitment_interviewers()
            self.message_unsubscribe(partner_ids=interviewers_to_clean.partner_id.ids)

            new_interviewers = self.interviewer_ids - old_interviewers - self.env.user
            if new_interviewers:
                for applicant in self:
                    notification_subject = _("You have been assigned as an interviewer for %s", applicant.display_name)
                    notification_body = _("You have been assigned as an interviewer for the Applicant %s", applicant.partner_name)
                    applicant.message_notify(
                        res_id=applicant.id,
                        model=applicant._name,
                        partner_ids=new_interviewers.partner_id.ids,
                        author_id=self.env.user.partner_id.id,
                        email_from=self.env.user.email_formatted,
                        subject=notification_subject,
                        body=notification_body,
                        email_layout_xmlid="mail.mail_notification_layout",
                        record_name=applicant.display_name,
                        model_description="Applicant",
                    )
        if vals.get('date_closed'):
            for applicant in self:
                if applicant.job_id.date_to:
                    applicant.candidate_id.availability = applicant.job_id.date_to + relativedelta(days=1)

        if vals.get("company_id") and not self.env.context.get('do_not_propagate_company', False):
            self.candidate_id.with_context(do_not_propagate_company=True).write({"company_id": vals["company_id"]})
            self.candidate_id.applicant_ids.with_context(do_not_propagate_company=True).write({"company_id": vals["company_id"]})

        return res

    def get_empty_list_help(self, help_message):
        if 'active_id' in self.env.context and self.env.context.get('active_model') == 'hr.job':
            hr_job = self.env['hr.job'].browse(self.env.context['active_id'])
        elif self.env.context.get('default_job_id'):
            hr_job = self.env['hr.job'].browse(self.env.context['default_job_id'])
        else:
            hr_job = self.env['hr.job']

        nocontent_body = Markup("""
<p class="o_view_nocontent_smiling_face">%(help_title)s</p>
""") % {
            'help_title': _("No application found. Let's create one !"),
        }

        if hr_job:
            pattern = r'(.*)<a>(.*?)<\/a>(.*)'
            match = re.fullmatch(pattern, _('Have you tried to <a>add skills to your job position</a> and search into the Reserve ?'))
            nocontent_body += Markup("""
<p>%(para_1)s<a href="%(link)s">%(para_2)s</a>%(para_3)s</p>""") % {
            'para_1': match[1],
            'para_2': match[2],
            'para_3': match[3],
            'link': f'/odoo/recruitment/{hr_job.id}',
        }

        if hr_job.alias_email:
            nocontent_body += Markup('<p class="o_copy_paste_email oe_view_nocontent_alias">%(helper_email)s <a href="mailto:%(email)s">%(email)s</a></p>') % {
                'helper_email': _("Try creating an application by sending an email to"),
                'email': hr_job.alias_email,
            }

        return super().get_empty_list_help(nocontent_body)

    @api.model
    def get_view(self, view_id=None, view_type='form', **options):
        if view_type == 'form' and self.env.user.has_group('hr_recruitment.group_hr_recruitment_interviewer')\
            and not self.env.user.has_group('hr_recruitment.group_hr_recruitment_user'):
            view_id = self.env.ref('hr_recruitment.hr_applicant_view_form_interviewer').id
        return super().get_view(view_id, view_type, **options)

    def action_create_meeting(self):
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
        if self.env.user.has_group('hr_recruitment.group_hr_recruitment_interviewer') and not self.env.user.has_group('hr_recruitment.group_hr_recruitment_user'):
            partners |= self.env.user.partner_id
        else:
            partners |= self.user_id.partner_id

        res = self.env['ir.actions.act_window']._for_xml_id('calendar.action_calendar_event')
        # As we are redirected from the hr.applicant, calendar checks rules on "hr.applicant",
        # in order to decide whether to allow creation of a meeting.
        # As interviewer does not have create right on the hr.applicant, in order to allow them
        # to create a meeting for an applicant, we pass 'create': True to the context.
        res['context'] = {
            'create': True,
            'default_applicant_id': self.id,
            'default_candidate_id': self.candidate_id.id,
            'default_partner_ids': partners.ids,
            'default_user_id': self.env.uid,
            'default_name': self.partner_name,
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
            'view_mode': 'list,form',
            'views': [
                (self.env.ref('hr_recruitment.ir_attachment_hr_recruitment_list_view').id, 'list'),
                (False, 'form'),
            ],
            'search_view_id': self.env.ref('hr_recruitment.ir_attachment_view_search_inherit_hr_recruitment').ids,
            'domain': [('res_model', '=', 'hr.applicant'), ('res_id', 'in', self.ids), ],
        }

    def action_open_employee(self):
        self.ensure_one()
        return self.candidate_id.action_open_employee()

    def action_open_other_applications(self):
        self.ensure_one()
        similar_candidates = (
            self.env["hr.candidate"]
            .with_context(active_test=False)
            .search(self.candidate_id._get_similar_candidates_domain())
            - self.candidate_id
        )
        return {
            'name': _('Other Applications'),
            'type': 'ir.actions.act_window',
            'res_model': 'hr.applicant',
            'view_mode': 'list,kanban,form,pivot,graph,calendar,activity',
            'domain': [('id', 'in', (self.candidate_id.applicant_ids + similar_candidates.applicant_ids).ids)],
            'context': {
                'active_test': False,
                'search_default_stage': 1,
            },
        }

    def _track_template(self, changes):
        res = super(Applicant, self)._track_template(changes)
        applicant = self[0]
        # When applcant is unarchived, they are put back to the default stage automatically. In this case,
        # don't post automated message related to the stage change.
        if 'stage_id' in changes and applicant.exists()\
            and applicant.stage_id.template_id\
            and not applicant._context.get('just_moved')\
            and not applicant._context.get('just_unarchived'):
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
        recipients = super()._message_get_suggested_recipients()
        if self.partner_id:
            self._message_add_suggested_recipient(recipients, partner=self.partner_id.sudo(), reason=_('Contact'))
        elif self.email_from:
            email_from = tools.email_normalize(self.email_from)
            if email_from and self.partner_name:
                email_from = tools.formataddr((self.partner_name, email_from))
                self._message_add_suggested_recipient(recipients, email=email_from, reason=_('Contact Email'))
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
        candidate_defaults = {}
        partner_name, email_from_normalized = tools.parse_contact_from_email(msg.get('from'))
        candidate_domain = [
            ("email_from", "=", email_from_normalized),
        ]
        if custom_values and 'job_id' in custom_values:
            job = self.env['hr.job'].browse(custom_values['job_id'])
            stage = job._get_first_stage()
            candidate_defaults['company_id'] = job.company_id.id
            candidate_domain = expression.AND([candidate_domain, [("company_id", "in", [job.company_id.id, False])]])

        candidate = self.env["hr.candidate"].search(candidate_domain, limit=1)\
            or self.env["hr.candidate"].create({
                "partner_name": partner_name or email_from_normalized,
                **candidate_defaults,
            })

        defaults = {
            'candidate_id': candidate.id,
            'partner_name': partner_name,
        }
        job_platform = self.env['hr.job.platform'].search([('email', '=', email_from_normalized)], limit=1)
        if msg.get('from') and not job_platform:
            candidate.email_from = msg.get('from')
            candidate.partner_id = msg.get('author_id', False)
        if msg.get('email_from') and job_platform:
            subject_pattern = re.compile(job_platform.regex or '')
            regex_results = re.findall(subject_pattern, msg.get('subject')) + re.findall(subject_pattern, msg.get('body'))
            candidate.partner_name = regex_results[0] if regex_results else partner_name
            defaults["partner_name"] = candidate.partner_name
            del msg['email_from']
        if msg.get('priority'):
            defaults['priority'] = msg.get('priority')
        if stage and stage.id:
            defaults['stage_id'] = stage.id
        if custom_values:
            defaults.update(custom_values)
        res = super().message_new(msg, custom_values=defaults)
        candidate._compute_partner_phone_email()
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
        self.ensure_one()
        action = self.candidate_id.create_employee_from_candidate()
        employee = self.env['hr.employee'].browse(action['res_id'])
        employee.write({
            'job_id': self.job_id.id,
            'job_title': self.job_id.name,
            'department_id': self.department_id.id,
            'work_email': self.department_id.company_id.email or self.email_from, # To have a valid email address by default
            'work_phone': self.department_id.company_id.phone,
        })
        return action

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

    def _get_duration_from_tracking(self, trackings):
        json = super()._get_duration_from_tracking(trackings)
        now = datetime.now()
        for applicant in self:
            if applicant.refuse_reason_id and applicant.refuse_date:
                json[applicant.stage_id.id] -= (now - applicant.refuse_date).total_seconds()
        return json

```

## File: models\hr_applicant_category.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint

from odoo import fields, models


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

## File: models\hr_candidate.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo.addons.hr_recruitment.models.hr_applicant import AVAILABLE_PRIORITIES

from odoo import api, models, fields, SUPERUSER_ID, tools, _
from odoo.exceptions import UserError
from odoo.osv import expression


class HrCandidate(models.Model):
    _name = "hr.candidate"
    _description = "Candidate"
    _inherit = ['mail.thread.cc',
               'mail.thread.main.attachment',
               'mail.thread.blacklist',
               'mail.thread.phone',
               'mail.activity.mixin',
    ]
    _order = "priority desc, availability asc, id desc"
    _mailing_enabled = True
    _primary_email = 'email_from'
    _rec_name = 'partner_name'

    active = fields.Boolean("Active", default=True, index=True)
    company_id = fields.Many2one('res.company', "Company", default=lambda self: self.env.company)
    applicant_ids = fields.One2many('hr.applicant', 'candidate_id')
    application_count = fields.Integer(compute="_compute_application_count")
    partner_id = fields.Many2one('res.partner', "Contact", copy=False, index='btree_not_null')
    partner_name = fields.Char("Candidates's Name")
    email_from = fields.Char(
        string="Email",
        size=128,
        compute='_compute_partner_phone_email',
        inverse='_inverse_partner_email',
        store=True,
        index='trigram')
    email_normalized = fields.Char(index='trigram')  # inherited via mail.thread.blacklist
    partner_phone = fields.Char(
        string="Phone",
        size=32,
        compute='_compute_partner_phone_email',
        inverse='_inverse_partner_email',
        store=True,
        index='btree_not_null')
    partner_phone_sanitized = fields.Char(
        string='Sanitized Phone Number',
        compute='_compute_partner_phone_sanitized',
        store=True,
        index='btree_not_null')
    linkedin_profile = fields.Char('LinkedIn Profile')
    type_id = fields.Many2one('hr.recruitment.degree', "Degree")
    availability = fields.Date("Availability", help="The date at which the applicant will be available to start working", tracking=True)
    categ_ids = fields.Many2many('hr.applicant.category', string="Tags")
    color = fields.Integer("Color Index", default=0)
    priority = fields.Selection(AVAILABLE_PRIORITIES, string="Evaluation", compute="_compute_priority", store=True)
    user_id = fields.Many2one(
        'res.users',
        string="Candidate Manager",
        default=lambda self: self.env.user if self.env.user.id != SUPERUSER_ID else False,
        domain="[('share', '=', False), ('company_ids', 'in', company_id)]",
        tracking=True)
    employee_id = fields.Many2one('hr.employee', string="Employee", help="Employee linked to the candidate.", copy=False)
    emp_is_active = fields.Boolean(string="Employee Active", related='employee_id.active')
    employee_name = fields.Char(related='employee_id.name', string="Employee Name", readonly=False, tracking=False)

    similar_candidates_count = fields.Integer(
        compute='_compute_similar_candidates_count',
        help='Candidates with the same email or phone or mobile')
    applications_count = fields.Integer(string="# Offers", compute='_compute_applications_count')
    refused_applications_count = fields.Integer(string="# Refused Offers", compute='_compute_applications_count')
    accepted_applications_count = fields.Integer(string="# Accepted Offers", compute='_compute_applications_count')

    meeting_ids = fields.One2many('calendar.event', 'candidate_id', 'Meetings')
    meeting_display_text = fields.Char(compute='_compute_meeting_display')
    meeting_display_date = fields.Date(compute='_compute_meeting_display')
    attachment_count = fields.Integer(
        string="Number of Attachments",
        compute='_compute_attachment_count')
    candidate_properties = fields.Properties('Properties', definition='company_id.candidate_properties_definition', copy=True)
    attachment_ids = fields.One2many('ir.attachment', 'res_id', domain=[('res_model', '=', 'hr.candidate')], string='Attachments')

    def init(self):
        self.env.cr.execute("""
            CREATE INDEX IF NOT EXISTS hr_candidate_email_partner_phone_mobile
            ON hr_candidate(email_normalized, partner_phone_sanitized);
        """)

    @api.depends('partner_name')
    def _compute_display_name(self):
        for candidate in self:
            candidate.display_name = candidate.partner_name or candidate.partner_id.name

    @api.depends('partner_phone')
    def _compute_partner_phone_sanitized(self):
        for candidate in self:
            candidate.partner_phone_sanitized = candidate._phone_format(fname='partner_phone') or candidate.partner_phone

    @api.depends('partner_id')
    def _compute_partner_phone_email(self):
        for candidate in self:
            if not candidate.partner_id:
                continue
            candidate.email_from = candidate.partner_id.email
            if not candidate.partner_phone:
                candidate.partner_phone = candidate.partner_id.phone
    
    def _phone_get_number_fields(self):
        return ['partner_phone']

    def _inverse_partner_email(self):
        for candidate in self:
            if not candidate.email_from:
                continue
            if not candidate.partner_id:
                if not candidate.partner_name:
                    raise UserError(_('You must define a Contact Name for this candidate.'))
                candidate.partner_id = self.env['res.partner'].with_context(default_lang=self.env.lang).find_or_create(candidate.email_from)
            if candidate.partner_name and not candidate.partner_id.name:
                candidate.partner_id.name = candidate.partner_name
            if tools.email_normalize(candidate.email_from) != tools.email_normalize(candidate.partner_id.email):
                # change email on a partner will trigger other heavy code, so avoid to change the email when
                # it is the same. E.g. "email@example.com" vs "My Email" <email@example.com>""
                candidate.partner_id.email = candidate.email_from
            if candidate.partner_phone:
                candidate.partner_id.phone = candidate.partner_phone

    @api.depends('email_from', 'partner_phone_sanitized')
    def _compute_similar_candidates_count(self):
        """
            The field similar_candidates_count is only used on the form view.
            Thus, using ORM rather then querying, should not make much
            difference in terms of performance, while being more readable and secure.
        """
        if not any(self._ids):
            for candidate in self:
                domain = candidate._get_similar_candidates_domain()
                if domain:
                    candidate.similar_candidates_count = max(0, self.env["hr.candidate"].with_context(active_test=False).search_count(domain) - 1)
                else:
                    candidate.similar_candidates_count = 0
            return
        self.flush_recordset(['email_normalized', 'partner_phone_sanitized'])
        self.env.cr.execute("""
            SELECT
                id,
                (
                    SELECT COUNT(*)
                    FROM hr_candidate AS sub
                    WHERE c.id != sub.id
                     AND ((coalesce(c.email_normalized, '') <> '' AND sub.email_normalized = c.email_normalized)
                       OR (coalesce(c.partner_phone_sanitized, '') <> '' AND c.partner_phone_sanitized = sub.partner_phone_sanitized))
                      AND c.company_id = sub.company_id
                ) AS similar_candidates
            FROM hr_candidate AS c
            WHERE id IN %(ids)s
        """, {'ids': tuple(self._origin.ids)})
        query_results = self.env.cr.dictfetchall()
        mapped_data = {result['id']: result['similar_candidates'] for result in query_results}
        for candidate in self:
            candidate.similar_candidates_count = mapped_data.get(candidate.id, 0)

    def _get_similar_candidates_domain(self):
        """
            This method returns a domain for the applicants whitch match with the
            current candidate according to email_from, partner_phone.
            Thus, search on the domain will return the current candidate as well if any of
            the following fields are filled.
        """
        self.ensure_one()
        if not self:
            return []
        domain = [('id', 'in', self.ids)]
        if self.email_normalized:
            domain = expression.OR([domain, [('email_normalized', '=', self.email_normalized)]])
        if self.partner_phone_sanitized:
            domain = expression.OR([domain, [('partner_phone_sanitized', '=', self.partner_phone_sanitized)]])
        domain = expression.AND([domain, [('company_id', '=', self.company_id.id)]])
        return domain

    def _compute_attachment_count(self):
        read_group_res = self.env['ir.attachment']._read_group(
            [('res_model', '=', 'hr.candidate'), ('res_id', 'in', self.ids)],
            ['res_id'], ['__count'])
        attach_data = dict(read_group_res)
        for candidate in self:
            candidate.attachment_count = attach_data.get(candidate.id, 0)

    def _compute_application_count(self):
        read_group_res = self.env['hr.applicant'].with_context(active_test=False)._read_group(
            [('candidate_id', 'in', self.ids)],
            ['candidate_id'], ['__count'])
        application_data = dict(read_group_res)
        for candidate in self:
            candidate.application_count = application_data.get(candidate, 0)

    @api.depends('applicant_ids.priority')
    def _compute_priority(self):
        for candidate in self:
            if not candidate.applicant_ids:
                candidate.priority = "0"
            else:
                candidate.priority = str(round(sum(int(a.priority) for a in candidate.applicant_ids) / len(candidate.applicant_ids)))

    def _compute_applications_count(self):
        result = defaultdict(lambda: {"total": 0, "refused": 0, "accepted": 0})
        for applicant in self.with_context(active_test=False).applicant_ids:
            result[applicant.candidate_id.id]["total"] += 1
            if applicant.application_status == "refused":
                result[applicant.candidate_id.id]["refused"] += 1
            elif applicant.application_status == "hired":
                result[applicant.candidate_id.id]["accepted"] += 1
        for candidate in self:
            candidate.applications_count = result[candidate.id]['total']
            candidate.refused_applications_count = result[candidate.id]['refused']
            candidate.accepted_applications_count = result[candidate.id]['accepted']

    @api.depends_context('lang')
    @api.depends('meeting_ids', 'meeting_ids.start')
    def _compute_meeting_display(self):
        candidate_with_meetings = self.filtered('meeting_ids')
        (self - candidate_with_meetings).update({
            'meeting_display_text': _('No Meeting'),
            'meeting_display_date': ''
        })
        today = fields.Date.today()
        for candidate in candidate_with_meetings:
            count = len(candidate.meeting_ids)
            dates = candidate.meeting_ids.mapped('start')
            min_date, max_date = min(dates).date(), max(dates).date()
            if min_date >= today:
                candidate.meeting_display_date = min_date
            else:
                candidate.meeting_display_date = max_date
            if count == 1:
                candidate.meeting_display_text = _('1 Meeting')
            elif candidate.meeting_display_date >= today:
                candidate.meeting_display_text = _('Next Meeting')
            else:
                candidate.meeting_display_text = _('Last Meeting')

    def write(self, vals):
        res = super().write(vals)

        if vals.get("company_id") and not self.env.context.get('do_not_propagate_company', False):
            self.applicant_ids.with_context(do_not_propagate_company=True).write({"company_id": vals["company_id"]})
        return res

    def action_open_similar_candidates(self):
        self.ensure_one()
        domain = self._get_similar_candidates_domain()
        similar_candidates = self.env['hr.candidate'].with_context(active_test=False).search(domain)
        return {
            'type': 'ir.actions.act_window',
            'name': _('Similar Candidates'),
            'res_model': self._name,
            'view_mode': 'list,kanban,form,activity',
            'domain': [('id', 'in', similar_candidates.ids)],
            'context': {
                'active_test': False,
            },
        }

    def action_open_applications(self):
        self.ensure_one()
        return {
            'name': _('Applications'),
            'type': 'ir.actions.act_window',
            'res_model': 'hr.applicant',
            'view_mode': 'list,kanban,form,pivot,graph,calendar,activity',
            'domain': [('id', 'in', self.applicant_ids.ids)],
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
            'res_id': self.employee_id.id,
        }

    def action_create_meeting(self):
        """ This opens Meeting's calendar view to schedule meeting on current candidate
            @return: Dictionary value for created Meeting view
        """
        self.ensure_one()
        if not self.partner_id:
            if not self.partner_name:
                raise UserError(_('You must define a Contact Name for this candidate.'))
            self.partner_id = self.env['res.partner'].create({
                'is_company': False,
                'name': self.partner_name,
                'email': self.email_from,
            })

        partners = self.partner_id
        if self.env.user.has_group('hr_recruitment.group_hr_recruitment_interviewer') and not self.env.user.has_group('hr_recruitment.group_hr_recruitment_user'):
            partners |= self.env.user.partner_id
        else:
            partners |= self.user_id.partner_id

        res = self.env['ir.actions.act_window']._for_xml_id('calendar.action_calendar_event')
        # As we are redirected from the hr.candidate, calendar checks rules on "hr.applicant",
        # in order to decide whether to allow creation of a meeting.
        # As interviewer does not have create right on the hr.applicant, in order to allow them
        # to create a meeting for an applicant, we pass 'create': True to the context.
        res['context'] = {
            'create': True,
            'default_candidate_id': self.id,
            'default_partner_ids': partners.ids,
            'default_user_id': self.env.uid,
            'default_name': self.partner_name,
            'attachment_ids': self.attachment_ids.ids
        }
        return res

    @api.ondelete(at_uninstall=False)
    def _unlink_except_linked_employee(self):
        if self.employee_id:
            raise UserError(_("The candidate is linked to an employee, to avoid losing information, archive it instead."))

    def create_employee_from_candidate(self):
        self.ensure_one()
        self._check_interviewer_access()

        if not self.partner_id:
            if not self.partner_name:
                raise UserError(_('Please provide an candidate name.'))
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
            'private_street': address_sudo.street,
            'private_street2': address_sudo.street2,
            'private_city': address_sudo.city,
            'private_state_id': address_sudo.state_id.id,
            'private_zip': address_sudo.zip,
            'private_country_id': address_sudo.country_id.id,
            'private_phone': address_sudo.phone,
            'private_email': address_sudo.email,
            'lang': address_sudo.lang,
            'address_id': self.company_id.partner_id.id,
            'candidate_id': self.ids,
            'phone': self.partner_phone
        }

    def _check_interviewer_access(self):
        if self.env.user.has_group('hr_recruitment.group_hr_recruitment_interviewer') and not self.env.user.has_group('hr_recruitment.group_hr_recruitment_user'):
            raise UserError(_('You are not allowed to perform this action.'))

    def action_open_attachments(self):
        return {
            'type': 'ir.actions.act_window',
            'res_model': 'ir.attachment',
            'name': _('Documents'),
            'context': {
                'default_res_model': 'hr.candidate',
                'default_res_id': self.ids[0],
                'show_partner_name': 1,
            },
            'view_mode': 'list,form',
            'views': [
                (self.env.ref('hr_recruitment.ir_attachment_hr_recruitment_list_view').id, 'list'),
                (False, 'form'),
            ],
            'search_view_id': self.env.ref('hr_recruitment.ir_attachment_view_search_inherit_hr_recruitment').ids,
            'domain': [('res_model', '=', 'hr.candidate'), ('res_id', 'in', self.ids)],
        }

    def action_send_email(self):
        return {
            'name': _('Send Email'),
            'type': 'ir.actions.act_window',
            'target': 'new',
            'view_mode': 'form',
            'res_model': 'candidate.send.mail',
            'context': {
                'default_candidate_ids': self.ids,
            }
        }

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


class HrEmployee(models.Model):
    _inherit = "hr.employee"

    # YTI Rename into candidate_ids
    candidate_id = fields.One2many('hr.candidate', 'employee_id', 'Candidate', groups="hr.group_hr_user")

    def _get_partner_count_depends(self):
        return super()._get_partner_count_depends() + ['candidate_id']

    def _get_related_partners(self):
        partners = super()._get_related_partners()
        return partners | self.sudo().candidate_id.partner_id

    @api.model_create_multi
    def create(self, vals_list):
        employees = super().create(vals_list)
        for employee in employees:
            if employee.candidate_id:
                employee.candidate_id._message_log_with_view(
                    'hr_recruitment.candidate_hired_template',
                    render_values={'candidate': employee.candidate_id}
                )
        return employees

```

## File: models\hr_job.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast
from collections import defaultdict

from dateutil.relativedelta import relativedelta
from odoo import api, fields, models, SUPERUSER_ID, _
from odoo.tools import SQL
from odoo.tools.convert import convert_file


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
    user_id = fields.Many2one('res.users', "Recruiter",
        domain="[('share', '=', False), ('company_ids', 'in', company_id)]", default=lambda self: self.env.user,
        tracking=True, help="The Recruiter will be the default value for all Applicants in this job \
            position. The Recruiter is automatically added to all meetings with the Applicant.")
    document_ids = fields.One2many('ir.attachment', compute='_compute_document_ids', string="Documents", readonly=True)
    documents_count = fields.Integer(compute='_compute_document_ids', string="Document Count")
    alias_id = fields.Many2one(help="Email alias for this job position. New emails will automatically create new applicants for this job position.")
    color = fields.Integer("Color Index")
    is_favorite = fields.Boolean(compute='_compute_is_favorite', inverse='_inverse_is_favorite')
    favorite_user_ids = fields.Many2many('res.users', 'job_favorite_user_rel', 'job_id', 'user_id', default=_get_default_favorite_user_ids)
    interviewer_ids = fields.Many2many('res.users', string='Interviewers', domain="[('share', '=', False), ('company_ids', 'in', company_id)]", help="The Interviewers set on the job position can see all Applicants in it. They have access to the information, the attachments, the meeting management and they can refuse him. You don't need to have Recruitment rights to be set as an interviewer.")
    extended_interviewer_ids = fields.Many2many('res.users', 'hr_job_extended_interviewer_res_users', compute='_compute_extended_interviewer_ids', store=True)
    industry_id = fields.Many2one('res.partner.industry', 'Industry')
    date_from = fields.Date(help="Is set, update candidates availability once hired for that specific mission.")
    date_to = fields.Date()

    activities_overdue = fields.Integer(compute='_compute_activities')
    activities_today = fields.Integer(compute='_compute_activities')

    job_properties = fields.Properties('Properties', definition='company_id.job_properties_definition')

    applicant_properties_definition = fields.PropertiesDefinition('Applicant Properties')
    no_of_hired_employee = fields.Integer(
        compute='_compute_no_of_hired_employee',
        string='Hired', copy=False,
        help='Number of hired employees for this job position during recruitment phase.',
        store=True)

    @api.depends('application_ids.date_closed')
    def _compute_no_of_hired_employee(self):
        counts = dict(self.env['hr.applicant']._read_group(
            domain=[
                ('job_id', 'in', self.ids),
                ('date_closed', '!=', False),
                '|',
                    ('active', '=', False),
                    ('active', '=', True),
            ],
            groupby=['job_id'],
            aggregates=['__count']))
        for job in self:
            job.no_of_hired_employee = counts.get(job, 0)

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
        applicants = self.mapped('application_ids').filtered(lambda self: not self.employee_id)
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
                    OR a.company_id is NULL
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
        values = super()._alias_get_creation_values()
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
            vals["favorite_user_ids"] = vals.get("favorite_user_ids", [])
        jobs = super().create(vals_list)
        utm_linkedin = self.env.ref("utm.utm_source_linkedin", raise_if_not_found=False)
        if utm_linkedin:
            source_vals = [{
                'source_id': utm_linkedin.id,
                'job_id': job.id,
            } for job in jobs]
            self.env['hr.recruitment.source'].create(source_vals)
        jobs.sudo().interviewer_ids._create_recruitment_interviewers()
        # Automatically subscribe the department manager and the recruiter to a job position.
        for job in jobs:
            job.message_subscribe(
                job.manager_id._get_related_partners().ids + job.user_id.partner_id.ids
            )

        return jobs

    def write(self, vals):
        old_interviewers = self.interviewer_ids
        old_managers = {}
        old_recruiters = {}
        for job in self:
            old_managers[job] = job.manager_id
            old_recruiters[job] = job.user_id
        if 'active' in vals and not vals['active']:
            self.application_ids.active = False
        res = super().write(vals)
        if 'interviewer_ids' in vals:
            interviewers_to_clean = old_interviewers - self.interviewer_ids
            interviewers_to_clean._remove_recruitment_interviewers()
            self.sudo().interviewer_ids._create_recruitment_interviewers()

        # Subscribe the department manager if the department has changed
        if "department_id" in vals:
            for job in self:
                to_unsubscribe = [
                    partner
                    for partner in old_managers[job]._get_related_partners().ids
                    if partner not in job.user_id.partner_id.ids
                ]
                job.message_unsubscribe(to_unsubscribe)
                job.message_subscribe(job.manager_id._get_related_partners().ids)

        # Subscribe the recruiter if it has changed.
        if "user_id" in vals:
            for job in self:
                to_unsubscribe = [
                    partner
                    for partner in old_recruiters[job].partner_id.ids
                    if partner not in job.manager_id._get_related_partners().ids
                ]
                job.message_unsubscribe(to_unsubscribe)
                job.message_subscribe(job.user_id.partner_id.ids)

        # Update the availability on all hired candidates if the mission end date is changed
        if "date_to" in vals:
            for job in self:
                hired_candidates = job.application_ids.filtered(lambda a: a.application_status == 'hired')
                for candidate in hired_candidates:
                    candidate.availability = job.date_to + relativedelta(days=1)

        # Since the alias is created upon record creation, the default values do not reflect the current values unless
        # specifically rewritten
        # List of fields to keep synched with the alias
        alias_fields = {'department_id', 'user_id'}
        if any(field for field in alias_fields if field in vals):
            for job in self:
                alias_default_vals = job._alias_get_creation_values().get('alias_defaults', '{}')
                job.alias_defaults = alias_default_vals
        return res

    def _order_field_to_sql(self, alias, field_name, direction, nulls, query):
        if field_name == 'is_favorite':
            sql_field = SQL(
                "%s IN (SELECT job_id FROM job_favorite_user_rel WHERE user_id = %s)",
                SQL.identifier(alias, 'id'), self.env.uid,
            )
            return SQL("%s %s %s", sql_field, direction, nulls)

        return super()._order_field_to_sql(alias, field_name, direction, nulls, query)

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
            'view_mode': 'list',
            'views': [
                (self.env.ref('hr_recruitment.ir_attachment_hr_recruitment_list_view').id, 'list')
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
            'views': [(form_view.id, 'form')],
            'type': 'ir.actions.act_window',
            'target': 'inline'
        }

    @api.model
    def _action_load_recruitment_scenario(self):

        convert_file(
            self.sudo().env,
            "hr_recruitment",
            "data/scenarios/hr_recruitment_scenario.xml",
            None,
            mode="init",
            kind="data",
        )

        return {
            "type": "ir.actions.client",
            "tag": "reload",
        }

```

## File: models\hr_job_platform.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.tools import email_normalize


class JobPlatform(models.Model):
    _name = "hr.job.platform"
    _description = 'Job Platforms'

    name = fields.Char(required=True)
    email = fields.Char(required=True, help="Applications received from this Email won't be linked to a contact."
                                            "There will be no email address set on the Applicant either.")
    regex = fields.Char(help="The regex facilitates to extract information from the subject or body "
                             "of the received email to autopopulate the Applicant's name field")

    _sql_constraints = [
        ('email_uniq', 'unique (email)', "The Email must be unique, this one already corresponds to another Job Platform."),
    ]

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if vals['email']:
                vals['email'] = email_normalize(vals['email']) or vals['email']
        platforms = super().create(vals_list)
        return platforms

    def write(self, vals):
        if vals.get('email'):
            vals['email'] = email_normalize(vals['email']) or vals['email']
        return super().write(vals)

```

## File: models\hr_recruitment_degree.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class RecruitmentDegree(models.Model):
    _name = "hr.recruitment.degree"
    _description = "Applicant Degree"

    name = fields.Char("Degree Name", required=True, translate=True)
    sequence = fields.Integer("Sequence", default=1)

    _sql_constraints = [
        ('name_uniq', 'unique (name)', 'The name of the Degree of Recruitment must be unique!')
    ]

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
    medium_id = fields.Many2one('utm.medium', default=lambda self: self.env['utm.medium']._fetch_or_create_utm_medium('website'))

    def _compute_has_domain(self):
        for source in self:
            if source.alias_id:
                source.has_domain = bool(source.alias_id.alias_domain_id)
            else:
                source.has_domain = bool(source.job_id.company_id.alias_domain_id
                                         or self.env.company.alias_domain_id)

    def create_alias(self):
        campaign = self.env.ref('hr_recruitment.utm_campaign_job')
        medium = self.env['utm.medium']._fetch_or_create_utm_medium('email')
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
            source.check_access('create')
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
        help='Specific jobs that use this stage. Other jobs will not use this stage.')
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
        is_interviewer = self.env.user.has_group('hr_recruitment.group_hr_recruitment_interviewer')
        is_user = self.env.user.has_group('hr_recruitment.group_hr_recruitment_user')
        if not is_interviewer:
            res.append(self.env.ref('hr.menu_view_hr_job').id)
        elif is_interviewer and not is_user:
            res.append(self.env.ref('hr_recruitment.menu_hr_job_position').id)
        else:
            res.append(self.env.ref('hr_recruitment.menu_hr_job_position_interviewer').id)
        return res

```

## File: models\mail_activity_plan.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MailActivityPlan(models.Model):
    _inherit = 'mail.activity.plan'

    def _compute_department_assignable(self):
        super()._compute_department_assignable()
        for plan in self:
            if not plan.department_assignable:
                plan.department_assignable = plan.res_model == 'hr.applicant'

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = "res.company"

    candidate_properties_definition = fields.PropertiesDefinition('Candidate Properties')
    job_properties_definition = fields.PropertiesDefinition("Job Properties")

```

## File: models\res_config_settings.py

```python
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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_department
from . import hr_applicant
from . import hr_applicant_category
from . import hr_applicant_refuse_reason
from . import hr_candidate
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
from . import res_company
from . import res_users
from . import ir_ui_menu
from . import mail_activity_plan
from . import hr_job_platform

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

    <record id="hr_candidate_comp_rule" model="ir.rule">
        <field name="name">Candidate multi company rule</field>
        <field name="model_id" ref="model_hr_candidate"/>
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
        <field name="implied_ids" eval="[(4, ref('group_hr_recruitment_interviewer'))]"/>
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

    <record id="hr_candidate_interviewer_rule" model="ir.rule">
        <field name="name">Candidate Interviewer</field>
        <field name="model_id" ref="model_hr_candidate"/>
        <field name="domain_force">[
            '|',
                ('applicant_ids.job_id.interviewer_ids', 'in', user.id),
                ('applicant_ids.interviewer_ids', 'in', user.id),
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

    <record id="hr_candidate_user_rule" model="ir.rule">
        <field name="name">User: All Candidates</field>
        <field name="model_id" ref="model_hr_candidate"/>
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

    <record id="mail_plan_rule_group_hr_recruitment_manager_applicant" model="ir.rule">
        <field name="name">Manager can manage applicant plans</field>
        <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]"/>
        <field name="model_id" ref="mail.model_mail_activity_plan"/>
        <field name="domain_force">[('res_model', '=', 'hr.applicant')]</field>
        <field name="perm_read" eval="False"/>
    </record>

    <record id="mail_plan_templates_rule_group_hr_recruitment_manager_applicant" model="ir.rule">
        <field name="name">Manager can manage applicant plan templates</field>
        <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]"/>
        <field name="model_id" ref="mail.model_mail_activity_plan_template"/>
        <field name="domain_force">[('plan_id.res_model', '=', 'hr.applicant')]</field>
        <field name="perm_read" eval="False"/>
    </record>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_job_interviewer,hr.job.interviewer,hr.model_hr_job,group_hr_recruitment_interviewer,1,0,0,0
access_hr_job_user,hr.job user,model_hr_job,group_hr_recruitment_user,1,1,1,1
hr.access_hr_job_user,hr.job user,model_hr_job,hr.group_hr_user,1,0,0,0
access_hr_applicant_interviewer,hr.applicant.interviewer,model_hr_applicant,group_hr_recruitment_interviewer,1,1,0,0
access_hr_candidate_interviewer,hr.candidate.interviewer,model_hr_candidate,group_hr_recruitment_interviewer,1,1,0,0
access_hr_applicant_user,hr.applicant.user,model_hr_applicant,group_hr_recruitment_user,1,1,1,1
access_hr_candidate_user,hr.candidate.user,model_hr_candidate,group_hr_recruitment_user,1,1,1,1
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
access_candidate_send_mail,access.candidate.send.mail,model_candidate_send_mail,hr_recruitment.group_hr_recruitment_user,1,1,1,0
access_candidate_send_mail_interviewer,access.candidate.send.mail.interviewer,model_candidate_send_mail,hr_recruitment.group_hr_recruitment_interviewer,1,1,1,0
access_mail_activity_plan_hr_recruitment_manager,mail.activity.plan.hr.recruitment.manager,mail.model_mail_activity_plan,hr_recruitment.group_hr_recruitment_manager,1,1,1,1
access_mail_activity_plan_template_hr_recruitment_manager,mail.activity.plan.template.hr.recruitment.manager,mail.model_mail_activity_plan_template,hr_recruitment.group_hr_recruitment_manager,1,1,1,1
access_hr_job_platform,access.hr.job.platform,model_hr_job_platform,hr_recruitment.group_hr_recruitment_manager,1,1,1,1

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
    static template = "hr_recruitment.ApplicantCharField";
    setup() {
        super.setup();

        this.action = useService("action");
    }

    onClick() {
        const record = this.props.record.data;
        if (record.res_id && record.res_model == 'hr.applicant') {
            this.action.doAction({
                type: 'ir.actions.act_window',
                res_model: 'hr.applicant',
                res_id: record.res_id.resId,
                views: [[false, "form"]],
                view_mode: "form",
                target: "current",
            });
        }
    }
}

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
    url: "/odoo",
    steps: () => [stepUtils.showAppsMenuItem(), {
    isActive: ["community"],
    trigger: '.o_app[data-menu-xmlid="hr_recruitment.menu_hr_recruitment_root"]',
    content: markup(_t("Let's have a look at how to <b>improve</b> your <b>hiring process</b>.")),
    tooltipPosition: 'right',
    run: "click",
}, {
    isActive: ["enterprise"],
    trigger: '.o_app[data-menu-xmlid="hr_recruitment.menu_hr_recruitment_root"]',
    content: markup(_t("Let's have a look at how to <b>improve</b> your <b>hiring process</b>.")),
    tooltipPosition: 'bottom',
    run: "click",
}, {
    trigger: ".o-kanban-button-new",
    content: _t("Create your first Job Position."),
    tooltipPosition: "bottom",
    run: "click",
},
{
    trigger: ".o_hr_job_simple_form",
},
{
    trigger: ".o_job_name",
    content: _t("What do you want to recruit today? Choose a job title..."),
    tooltipPosition: "right",
    run: "click",
},
{
    trigger: '.o_hr_job_simple_form',
},
{
    trigger: ".o_job_alias",
    content: _t("Choose an application email."),
    tooltipPosition: "right",
    run: "click",
}, {
    trigger: '.o_create_job',
    content: _t('Let\'s create the position. An email will be setup for applications, and a public job description, if you use the Website app.'),
    tooltipPosition: 'bottom',
    run: "click .modal:visible .btn.btn-primary",
}, {
    trigger: ".o_copy_paste_email",
    content: _t("Copy this email address, to paste it in your email composer, to apply."),
    tooltipPosition: "bottom",
    run: "click",
},
{
    trigger: ".o_kanban_applicant",
},
{
    trigger: ".breadcrumb-item:not(.active):last",
    content: _t("Let’s go back to the dashboard."),
    tooltipPosition: "bottom",
    run: "click",
},
{
    trigger: ".o_hr_recruitment_kanban",
},
{
    trigger: "button.oe_kanban_action",
    content: markup(_t("<b>Did you apply by sending an email?</b> Check incoming applications.")),
    tooltipPosition: "bottom",
    run: "click",
},
{
    trigger: ".o_kanban_applicant",
},
{
    trigger: ".oe_kanban_card",
    content: markup(_t("<b>Drag this card</b>, to qualify him for a first interview.")),
    tooltipPosition: "bottom",
    run: "drag_and_drop(.o_kanban_group:eq(1))",
},
{
    trigger: ".o_kanban_applicant",
},
{
    trigger: ".oe_kanban_card",
    content: markup(_t("<b>Click to view</b> the application.")),
    tooltipPosition: "bottom",
    run: "click",
},
{
    trigger: ".o_applicant_form",
},
{
    trigger: "button:contains(Send message)",
    content: markup(_t("<div><b>Try to send an email</b> to the applicant.</div><div><i>Tips: All emails sent or received are saved in the history here</i>")),
    tooltipPosition: "bottom",
    run: "click",
},
{
    trigger: ".o_applicant_form",
},
{
    trigger: ".o-mail-Chatter .o-mail-Composer button[aria-label='Send']",
    content: _t("Send your email. Followers will get a copy of the communication."),
    tooltipPosition: "bottom",
    run: "click",
},
{
    trigger: ".o_applicant_form",
},
{
    trigger: "button:contains(Log note)",
    content: _t("Or talk about this applicant privately with your colleagues."),
    tooltipPosition: "bottom",
    run: "click",
},
{
    trigger: ".o_applicant_form",
},
{
    trigger: ".o_create_employee",
    content: _t("Let’s create this new employee now."),
    tooltipPosition: "bottom",
    run: "click",
},
{
    trigger: ".o_hr_employee_form_view",
},
{
    trigger: ".o_form_button_save",
    content: _t("Save it!"),
    tooltipPosition: "bottom",
    run: "click",
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

## File: static\src\views\recruitment_form_controller.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { FormController } from "@web/views/form/form_controller";

export class RecruitmentFormController extends FormController {
    /**
     * @override
     */
    get archiveDialogProps() {
        const result = super.archiveDialogProps;
        result.body =
            this.model.root.data.all_application_count > 0
                ? _t("This job position and all related applicants will be archived. Are you sure?")
                : _t("Are you sure that you want to archive this job position?");
        console.log(this.model.root.data.all_application_count);
        return result;
    }
}

```

## File: static\src\views\recruitment_form_view.js

```javascript
import { registry } from "@web/core/registry";

import { formView } from "@web/views/form/form_view";
import { RecruitmentFormController } from "@hr_recruitment/views/recruitment_form_controller";

export const RecruitmentFormView = {
    ...formView,
    Controller: RecruitmentFormController,
};

registry.category("views").add("recruitment_form_view", RecruitmentFormView);

```

## File: static\src\views\recruitment_helper_view.js

```javascript
import { useService } from "@web/core/utils/hooks";
import { user } from "@web/core/user";
import { Component, onWillStart, useState } from "@odoo/owl";

export class RecruitmentActionHelper extends Component {
    static template = "hr_recruitment.RecruitmentActionHelper";
    static props = ["noContentHelp"];
    setup() {
        this.orm = useService("orm");
        this.actionService = useService("action");
        this.state = useState({
            hasDemoData: false,
        });
        onWillStart(async () => {
            const categoryTags = await this.orm.searchRead("hr.applicant.category", [], ["name"]);
            const demoTag = categoryTags.filter((tag) => tag.name === "Demo");
            this.state.hasDemoData = demoTag.length === 1;
            this.isRecruitmentUser = await user.hasGroup("hr_recruitment.group_hr_recruitment_user");
        });
    }

    loadRecruitmentScenario() {
        this.actionService.doAction("hr_recruitment.action_load_demo_data");
    }
}

```

## File: static\src\views\recruitment_helper_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <t t-name="hr_recruitment.RecruitmentActionHelper">
        <div class="o_view_nocontent">
            <div class="o_nocontent_help">
                <p class="o_view_nocontent_smiling_face">
                    Ready to recruit more efficiently?
                </p>
                <p>
                    Let's create a job position.
                </p>
                <t t-if="!state.hasDemoData and isRecruitmentUser">
                    <div class="d-flex gap-3 align-items-center or-separator">
                        <hr class="flex-grow-1" /> or <hr class="flex-grow-1" />
                    </div>

                    <a type="object" class="btn btn-secondary mt-3"
                        t-on-click="() => this.loadRecruitmentScenario()">
                        Load sample data
                    </a>
                </t>
            </div>
        </div>
    </t>
</odoo>

```

## File: static\src\views\recruitment_kanban_view.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";

import { kanbanView } from "@web/views/kanban/kanban_view";
import { KanbanRenderer } from "@web/views/kanban/kanban_renderer";
import { RecruitmentActionHelper } from "@hr_recruitment/views/recruitment_helper_view";

export class RecruitmentKanbanRenderer extends KanbanRenderer {
    static template = "hr_recruitment.RecruitmentKanbanRenderer";
    static components = {
        ...KanbanRenderer.components,
        RecruitmentActionHelper,
    };

    async archiveRecord(record, active) {
        if (active && record.data.application_count > 0) {
            this.dialog.add(ConfirmationDialog, {
                body: _t(
                    "This job position and all related applicants will be archived. Are you sure?"
                ),
                confirmLabel: _t("Archive"),
                confirm: () => {
                    record.archive();
                    this.props.list.load();
                },
                cancel: () => {},
            });
        } else if (active) {
            record.archive();
            this.props.list.load();
        } else {
            record.unarchive();
            this.props.list.load();
        }
    }
}

export const RecruitmentKanbanView = {
    ...kanbanView,
    Renderer: RecruitmentKanbanRenderer,
};

registry.category("views").add("recruitment_kanban_view", RecruitmentKanbanView);

```

## File: static\src\views\recruitment_kanban_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <t t-name="hr_recruitment.RecruitmentKanbanRenderer" t-inherit="web.KanbanRenderer" t-inherit-mode="primary">
        <t t-call="web.ActionHelper" position="replace">
            <t t-if="showNoContentHelper">
                <RecruitmentActionHelper noContentHelp="props.noContentHelp"/>
            </t>
        </t>
    </t>
</odoo>

```

## File: static\src\views\recruitment_list_controller.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { ListController } from "@web/views/list/list_controller";

export class RecruitmentListController extends ListController {
    /**
     * @override
     */
    get archiveDialogProps() {
        const result = super.archiveDialogProps;
        result.body =
            this.model.root.isDomainSelected || this.model.root.selection.length > 1
                ? _t(
                      "These job positions and all related applicants will be archived. Are you sure?"
                  )
                : _t(
                      "This job position and all related applicants will be archived. Are you sure?"
                  );
        return result;
    }
}

```

## File: static\src\views\recruitment_list_view.js

```javascript
import { registry } from "@web/core/registry";

import { listView } from "@web/views/list/list_view";
import { ListRenderer } from "@web/views/list/list_renderer";
import { RecruitmentActionHelper } from "@hr_recruitment/views/recruitment_helper_view";
import { RecruitmentListController } from "@hr_recruitment/views/recruitment_list_controller";

export class RecruitmentListRenderer extends ListRenderer {
    static template = "hr_recruitment.RecruitmentListRenderer";
    static components = {
        ...ListRenderer.components,
        RecruitmentActionHelper,
    };
}

export const RecruitmentListView = {
    ...listView,
    Controller: RecruitmentListController,
    Renderer: RecruitmentListRenderer,
};

registry.category("views").add("recruitment_list_view", RecruitmentListView);

```

## File: static\src\views\recruitment_list_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <t t-name="hr_recruitment.RecruitmentListRenderer" t-inherit="web.ListRenderer" t-inherit-mode="primary">
        <t t-call="web.ActionHelper" position="replace">
            <t t-if="showNoContentHelper">
                <RecruitmentActionHelper noContentHelp="props.noContentHelp"/>
            </t>
        </t>
    </t>
</odoo>

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
            <field name="name">hr.applicant.category.list</field>
            <field name="model">hr.applicant.category</field>
            <field name="arch" type="xml">
                <list string="Tags" editable="bottom">
                    <field name="name"/>
                    <field name="color" widget="color_picker" />
                </list>
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
            <field name="name">Applicant refuse reason list</field>
            <field name="model">hr.applicant.refuse.reason</field>
            <field name="arch" type="xml">
                <list string="Refuse Reason" editable="bottom">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                    <field name="template_id" context="{'default_model': 'hr.applicant'}"/>
                </list>
            </field>
        </record>

        <record id="hr_applicant_refuse_reason_action" model="ir.actions.act_window">
            <field name="name">Refuse Reasons</field>
            <field name="res_model">hr.applicant.refuse.reason</field>
            <field name="view_mode">list,form</field>
        </record>
    </data>
</odoo>

```

## File: views\hr_applicant_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record model="ir.ui.view" id="crm_case_tree_view_job">
        <field name="name">Applicants</field>
        <field name="model">hr.applicant</field>
        <field name="arch" type="xml">
            <list string="Applicants" class="o_search_matching_applicant" multi_edit="1" sample="1" decoration-danger="application_status == 'refused'">
                <field name="message_needaction" column_invisible="True"/>
                <field name="last_stage_id" column_invisible="True"/>
                <field name="date_last_stage_update" column_invisible="True"/>
                <field name="partner_name" readonly="1" optional="show"/>
                <field name="create_date" readonly="1" optional="show"/>
                <field name="type_id" column_invisible="True"/>
                <field name="job_id" optional="show"/>
                <field name="stage_id" optional="show"/>
                <field name="candidate_id" optional="show"/>
                <field name="application_status" optional="hide" invisible="application_status == 'ongoing'"/>
                <field name="refuse_reason_id" optional='hide'/>
                <field name="activity_ids" widget="list_activity" optional="hide"/>
                <field name="activity_date_deadline" optional="hide"/>
                <field name="priority" widget="priority" optional="show"/>
                <field name="email_from" readonly="1" optional="hide"/>
                <field name="categ_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="show"/>
                <field name="user_id" widget="many2one_avatar_user" optional="show"/>
                <field name="interviewer_ids" widget="many2many_avatar_user" optional="hide"/>
                <field name="partner_phone" widget="phone" readonly="1" optional="hide"/>
                <field name="medium_id" optional="hide"/>
                <field name="source_id" readonly="1" optional="hide"/>
                <field name="salary_expected" optional="hide"/>
                <field name="salary_proposed" optional="hide"/>
                <field name="availability" optional="hide"/>
                <field name="department_id" readonly="1" column_invisible="context.get('invisible_department', True)" optional="hide"/>
                <field name="company_id" column_invisible="True"/>
                <field name="company_id" groups="base.group_multi_company" readonly="1" optional="hide"/>
            </list>
        </field>
    </record>

    <record id="hr_applicant_view_tree_activity" model="ir.ui.view">
        <field name="name">hr.applicant.view.list.activity</field>
        <field name="model">hr.applicant</field>
        <field name="arch" type="xml">
            <list string="Next Activities" decoration-danger="activity_date_deadline &lt; current_date" default_order="activity_date_deadline">
                <field name="partner_id"/>
                <field name="activity_date_deadline"/>
                <field name="activity_type_id"/>
                <field name="activity_summary"/>
                <field name="stage_id"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </list>
        </field>
    </record>

    <record model="ir.ui.view" id="hr_applicant_view_form">
        <field name="name">Jobs - Recruitment Form</field>
        <field name="model">hr.applicant</field>
        <field name="arch" type="xml">
          <form string="Jobs - Recruitment Form" class="o_applicant_form">
            <field name="company_id" invisible="1"/>
            <field name="application_status" invisible="1"/>
            <field name="employee_id" invisible="1"/>
            <field name="meeting_ids" invisible="1"/>
            <field name="refuse_reason_id" invisible="1"/>
            <field name="email_normalized" invisible="1"/>
            <field name="partner_phone_sanitized" invisible="1"/>
            <field name="emp_is_active" invisible="1"/>
            <header>
                <button string="Create Employee" name="create_employee_from_applicant" type="object" data-hotkey="q" groups="hr.group_hr_user"
                        class="o_create_employee" invisible="employee_id or not active or not date_closed"/>
                <button string="Refuse" name="archive_applicant" type="object" invisible="not active" data-hotkey="d"/>
                <button string="Restore" name="toggle_active" type="object" invisible="active" data-hotkey="x"/>
                <field name="stage_id" widget="statusbar_duration" options="{'clickable': '1', 'fold_field': 'fold'}" invisible="not active and not employee_id"/>
            </header>
            <sheet>
                <div class="oe_button_box" name="button_box">
                    <button name="action_open_employee"
                            type="object"
                            class="oe_stat_button"
                            icon="fa-id-card-o"
                            groups="hr.group_hr_user"
                            invisible="not (employee_id or emp_is_active)">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value"><field name="employee_name" readonly="1"/></span>
                            <span class="o_stat_text">Employee</span>
                        </div>
                    </button>
                    <button name="action_open_other_applications"
                            class="oe_stat_button"
                            icon="fa-pencil"
                            type="object"
                            context="{'active_test': False}"
                            invisible="not other_applications_count">
                        <field name="other_applications_count" widget="statinfo" string="Other applications"/>
                    </button>
                    <button name="action_create_meeting" class="oe_stat_button" icon="fa-calendar" type="object" invisible="not id">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_text"><field name="meeting_display_text" /></span>
                            <span class="o_stat_value"><field name="meeting_display_date" readonly="1"/></span>
                        </div>
                    </button>
                </div>
                <widget name="web_ribbon" title="Refused" bg_color="text-bg-danger" invisible="application_status != 'refused'"/>
                <widget name="web_ribbon" title="Archived" bg_color="text-bg-secondary" invisible="application_status != 'archived'"/>
                <widget name="web_ribbon" title="Hired" invisible="application_status != 'hired'" />
                <field name="active" invisible="1"/>
                <field name="legend_normal" invisible="1"/>
                <field name="legend_blocked" invisible="1"/>
                <field name="legend_done" invisible="1"/>
                <field name="kanban_state" widget="state_selection" invisible="application_status == 'refused' or application_status == 'archived'"/>
                <field name="priority" widget="priority"/>
                <div class="oe_title pe-0">
                    <label for="candidate_id" class="oe_edit_only"/>
                    <h1 class="d-flex justify-content-between align-items-center">
                        <field name="candidate_id" options="{'line_breaks': False}" context="{'default_company_id': company_id}"/>
                    </h1>
                </div>
                <group>
                    <group>
                        <field name="partner_name" invisible="1"/>
                        <field name="partner_id" invisible="1" />
                        <field name="refuse_reason_id" invisible="active"/>
                        <field name="email_from" placeholder="e.g. john.doe@example.com" widget="email"/>
                        <field name="partner_phone" widget="phone"/>
                        <field name="linkedin_profile" placeholder="e.g. https://www.linkedin.com/in/..." widget="url"/>
                    </group>
                    <group>
                        <field name="job_id"/>
                        <field name="user_id" widget="many2one_avatar_user" placeholder="Unassigned"/>
                        <field name="interviewer_ids" options="{'no_create': True, 'no_create_edit': True}" widget="many2many_avatar_user" placeholder="Who can access candidates" />
                        <field name="date_closed" invisible="not date_closed" />
                        <field name="categ_ids" placeholder="Tags" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                    </group>
                </group>
                <field name="applicant_properties" columns="2"/>
                <notebook>
                    <page string="Note" name="application_note">
                        <field name="applicant_notes" type="html" options="{'collaborative': true, 'resizable': false}" placeholder="Internal notes..."/>
                    </page>
                    <page string="Details" name="application_details">
                        <group>
                            <group string="Applicant">
                                <field name="type_id" placeholder="e.g. Masters"/>
                                <field name="availability" placeholder="Directly"/>
                            </group>
                            <group string="Job">
                                <field name="department_id"/>
                                <field name="company_id" groups="base.group_multi_company" options='{"no_open":True}' />
                            </group>
                        </group>
                        <group>
                            <group string="Salary Package" name="recruitment_contract" groups="hr_recruitment.group_hr_recruitment_user">
                                <label for="salary_expected"/>
                                <div class="o_row">
                                    <field name="salary_expected" placeholder="Expected"/>
                                    <span invisible="not salary_expected_extra"> + </span>
                                    <field name="salary_expected_extra" placeholder="Other benefits"/>
                                </div>
                                <label for="salary_proposed"/>
                                <div class="o_row">
                                    <field name="salary_proposed"/>
                                    <span invisible="not salary_proposed_extra"> + </span>
                                    <field name="salary_proposed_extra" placeholder="Other benefits"/>
                                </div>
                            </group>
                            <group string="Sourcing">
                                <field name="source_id"/>
                                <field name="medium_id"/>
                            </group>
                        </group>
                    </page>
                </notebook>
            </sheet>
            <div class="o_attachment_preview" groups="hr_recruitment.group_applicant_cv_display"/>
            <chatter open_attachments="True" reload_on_attachment="True"/>
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
                    filter_domain="['|', ('partner_name', 'ilike', self), ('email_from', 'ilike', self)]"/>
                <field string="Email" name="email_from" filter_domain="[('email_from', 'ilike', self)]"/>
                <field name="job_id"/>
                <field name="department_id" operator="child_of"/>
                <field name="company_id" groups="base.group_multi_company"/>
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
                    <filter string="Candidate" name="candidate" domain="[]" context="{'group_by': 'candidate_id'}"/>
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
            <calendar
                string="Applicants"
                mode="month"
                date_start="activity_date_deadline"
                color="user_id"
                event_limit="5"
                hide_time="true"
                quick_create="0"
            >
                <field name="partner_name"/>
                <field name="job_id"/>
                <field name="priority" widget="priority"/>
                <field name="user_id" filters="1" invisible="1"/>
                <field name="activity_summary"/>
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
                    <field name="candidate_id" options="{'no_create_edit': True}" placeholder="e.g. John Doe"/>
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
            <kanban highlight_color="color" default_group_by="stage_id" class="o_kanban_applicant o_search_matching_applicant" quick_create_view="hr_recruitment.quick_create_applicant_form" sample="1">
                <field name="stage_id" options='{"group_by_tooltip": {"requirements": "Requirements"}}'/>
                <field name="legend_normal"/>
                <field name="legend_blocked"/>
                <field name="legend_done"/>
                <field name="date_closed"/>
                <field name="color"/>
                <field name="user_id"/>
                <field name="active"/>
                <field name="application_status" />
                <field name="company_id" invisible="1"/> <!-- We need to keep this field as it is used in the domain of user_id in the model -->
                <progressbar field="kanban_state" colors='{"done": "success", "blocked": "danger"}'/>
                <templates>
                    <t t-name="menu">
                        <a role="menuitem" name="action_create_meeting" type="object" class="dropdown-item">Schedule Interview</a>
                        <a role="menuitem" name="archive_applicant" type="object" class="dropdown-item">Refuse</a>
                        <a t-if="record.active.raw_value" role="menuitem" type="archive" class="dropdown-item">Archive</a>
                        <a t-if="!record.active.raw_value" role="menuitem" type="unarchive" class="dropdown-item">Unarchive</a>
                        <t t-if="widget.deletable"><a role="menuitem" type="delete" class="dropdown-item">Delete</a></t>
                    </t>
                    <t t-name="card">
                        <widget name="web_ribbon" title="Hired" bg_color="text-bg-success" invisible="not date_closed"/>
                        <widget name="web_ribbon" title="Refused" bg_color="text-bg-danger" invisible="application_status != 'refused'"/>
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-secondary" invisible="application_status != 'archived'"/>
                        <field t-if="record.partner_name.raw_value" class="fw-bold fs-5" name="partner_name"/>
                        <field name="job_id" invisible="context.get('search_default_job_id', False)"/>
                        <field name="categ_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                        <field name="applicant_properties" widget="properties"/>
                        <footer>
                            <field name="priority" widget="priority"/>
                            <field class="ms-1 align-items-center" name="activity_ids" widget="kanban_activity"/>
                            <div class="d-flex ms-auto align-items-center">
                                <a name="action_open_attachments" type="object">
                                    <i class='fa fa-paperclip' role="img" aria-label="Documents"/>
                                    <field name="attachment_number"/>
                                </a>
                                <field name="kanban_state" class="mx-1" widget="state_selection"/>
                                <field name="user_id" widget="many2one_avatar_user"/>
                            </div>
                        </footer>
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
        <field name="view_mode">kanban,list,form,graph,calendar,pivot,activity</field>
        <field name="search_view_id" ref="hr_applicant_view_search_bis"/>
        <field name="context">{'search_default_job_id': [active_id], 'default_job_id': active_id, 'search_default_stage':1, 'dialog_size':'medium', 'allow_search_matching_applicants': 1}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No applications yet
            </p><p>
                Odoo helps you track applicants in the recruitment
                process and follow up all operations: meetings, interviews, etc.
            </p><p>
                Applicants and their attached résumé are created automatically when an email is sent.
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
        <field name="binding_view_types">list</field>
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
        <field name="path">recruitment-applications</field>
        <field name="view_mode">kanban,list,form,pivot,graph,calendar,activity</field>
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
                Applicants and their attached résumé are created automatically when an email is sent.
                If you install the document management modules, all resumes are indexed automatically,
                so that you can easily search through their content.
            </p>
        </field>
    </record>

    <record id="hr_applicant_action_from_department" model="ir.actions.act_window">
        <field name="name">New Applications</field>
        <field name="res_model">hr.applicant</field>
        <field name="view_mode">list,kanban,form,graph,calendar,pivot</field>
        <field name="context">{
            'search_default_department_id': active_id,
            'default_department_id': active_id,
            'invisible_department': False}
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
        <field name="view_mode">list</field>
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

    <record id="hr_applicant_view_pivot" model="ir.ui.view">
        <field name="name">hr.applicant.pivot</field>
        <field name="model">hr.applicant</field>
        <field name="arch" type="xml">
            <pivot string="Recruitment Analysis" sample="1">
                <field name="stage_id" type="row"/>
                <field name="job_id" type="col"/>
                <field name="candidate_id"/>
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
                <filter string="Creation Date" name="year" date="create_date" default_period="year"/>
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
</odoo>

```

## File: views\hr_candidate_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="hr_candidate_view_form" model="ir.ui.view">
        <field name="name">hr.candidate.view.form</field>
        <field name="model">hr.candidate</field>
        <field name="arch" type="xml">
            <form string="Jobs - Recruitment Form" class="o_applicant_form">
                <field name="company_id" invisible="1"/>
                <field name="email_normalized" invisible="1"/>
                <field name="partner_phone_sanitized" invisible="1"/>
                <header>
                    <button string="Create Employee" name="create_employee_from_candidate" type="object" data-hotkey="q" groups="hr.group_hr_user"
                            class="o_create_employee" invisible="employee_id or not active"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_open_employee"
                                type="object"
                                class="oe_stat_button"
                                icon="fa-id-card-o"
                                groups="hr.group_hr_user"
                                invisible="not (employee_id or emp_is_active)">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value"><field name="employee_name" readonly="1"/></span>
                                <span class="o_stat_text">Employee</span>
                            </div>
                        </button>
                        <button name="action_open_applications"
                                class="oe_stat_button"
                                icon="fa-pencil"
                                type="object"
                                context="{'active_test': False, 'default_candidate_id': id}"
                                invisible="not application_count">
                            <field name="application_count" widget="statinfo" string="Applications"/>
                        </button>
                        <button name="action_open_similar_candidates"
                                class="oe_stat_button"
                                icon="fa-users"
                                type="object"
                                context="{'active_test': False}"
                                invisible="similar_candidates_count == 0">
                            <field name="similar_candidates_count" widget="statinfo" string="Similar Candidates"/>
                        </button>
                        <button name="action_create_meeting" class="oe_stat_button" icon="fa-calendar" type="object" invisible="not id">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_text"><field name="meeting_display_text"/></span>
                                <span class="o_stat_value"><field name="meeting_display_date" readonly="1"/></span>
                            </div>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <div class="row justify-content-between position-relative w-100 m-0 mb-2">
                        <div class="oe_title mw-75 ps-0 pe-2">
                            <h1 class="d-flex flex-row align-items-center">
                                <div invisible="not user_id" class="me-2">
                                    <widget name="hr_employee_chat" invisible="not context.get('chat_icon')"/>
                                </div>
                                <field name="partner_name" placeholder="Candidate's Name"
                                    required="True" style="font-size: min(4vw, 2.6rem);"/>
                            </h1>
                        </div>
                    </div>
                    <group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="partner_id" groups="base.group_no_one"/>
                            <field name="email_from"/>
                            <field name="partner_phone"/>
                            <field name="linkedin_profile"/>
                            <field name="type_id"/>
                        </group>
                        <group>
                            <field name="user_id" string="Manager"/>
                            <field name="priority" widget="priority"/>
                            <field name="availability"/>
                            <field name="categ_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                            <field name="employee_id" invisible="1"/>
                            <field name="emp_is_active" invisible="1"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                    </group>
                    <field name="candidate_properties" columns="2"/>
                    <notebook/>
                </sheet>
                <div class="o_attachment_preview" groups="hr_recruitment.group_applicant_cv_display"/>
                <chatter open_attachments="True" reload_on_attachment="True"/>
            </form>
        </field>
    </record>

    <record id="hr_candidate_view_tree" model="ir.ui.view">
        <field name="name">hr.candidate.view.list</field>
        <field name="model">hr.candidate</field>
        <field name="arch" type="xml">
            <list string="Candidates" multi_edit="1" sample="1">
                <field name="partner_name" string="Name"/>
                <field name="email_from"/>
                <field name="partner_phone"/>
                <field name="user_id" string="Manager" widget="many2one_avatar_user"/>
                <field name="priority" widget="priority" optional="show"/>
                <field name="availability" optional="show"/>
                <field name="type_id" optional="hide"/>
                <field name="linkedin_profile" optional="hide"/>
                <field name="applications_count" optional="hide"/>
                <field name="refused_applications_count" optional="hide"/>
                <field name="accepted_applications_count" optional="hide"/>
                <field name="partner_id" invisible="1"/>
                <field name="company_id" groups="base.group_multi_company" optional="hide"/>
            </list>
        </field>
    </record>

    <record id="hr_candidate_view_kanban" model="ir.ui.view" >
        <field name="name">hr.candidate.view.kanban</field>
        <field name="model">hr.candidate</field>
        <field name="arch" type="xml">
            <kanban highlight_color="color" class="o_kanban_applicant o_search_matching_applicant" sample="1">
                <field name="active"/>
                <field name="color"/>
                <field name="company_id"/>
                <templates>
                    <t t-name="menu">
                        <a role="menuitem" name="action_create_meeting" type="object" class="dropdown-item">Schedule Interview</a>
                        <a t-if="record.active.raw_value" role="menuitem" type="archive" class="dropdown-item">Archive</a>
                        <a t-if="!record.active.raw_value" role="menuitem" type="unarchive" class="dropdown-item">Unarchive</a>
                        <t t-if="widget.deletable"><a role="menuitem" type="delete" class="dropdown-item">Delete</a></t>
                        <div role="separator" class="dropdown-divider"></div>
                        <field name="color" widget="kanban_color_picker"/>
                    </t>
                    <t t-name="card">
                        <field t-if="record.partner_name.raw_value" class="fw-bold fs-5" name="partner_name"/>
                        <field t-else="" class="fw-bold fs-5" name="partner_id"/>
                        <field name="categ_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                        <field name="candidate_properties" widget="properties"/>
                        <footer>
                            <field name="priority" widget="priority"/>
                            <field class="ms-1 align-items-center" name="activity_ids" widget="kanban_activity"/>
                            <div class="d-flex ms-auto align-items-center">
                                <a name="action_open_attachments" type="object">
                                    <i class='fa fa-paperclip' role="img" aria-label="Documents"/>
                                    <field name="attachment_count"/>
                                </a>
                                <field name="user_id" widget="many2one_avatar_user"/>
                            </div>
                        </footer>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="hr_candidate_view_calendar" model="ir.ui.view">
        <field name="name">hr.candidate.view.calendar</field>
        <field name="model">hr.candidate</field>
        <field name="arch" type="xml">
            <calendar 
                string="Candidates"
                mode="month"
                date_start="activity_date_deadline"
                color="user_id"
                event_limit="5"
                hide_time="true"
                quick_create="0"
            >
                <field name="partner_name"/>
                <field name="priority" widget="priority"/>
                <field name="activity_summary"/>
                <field name="user_id" filters="1" invisible="1"/>
                <field name="candidate_properties"/>
            </calendar>
        </field>
    </record>

    <record id="hr_candidate_view_search" model="ir.ui.view">
        <field name="name">hr.candidate.view.search</field>
        <field name="model">hr.candidate</field>
        <field name="arch" type="xml">
            <search string="Candidates">
                <field name="partner_name"/>
                <field name="email_from"/>
                <field name="user_id"/>
                <field name="linkedin_profile"/>
                <filter string="My Candidates" name="my_candidates" domain="[('user_id', '=', uid)]"/>
                <filter string="Unassigned" name="unassigned" domain="[('user_id', '=', False)]"/>
                <separator/>
                <filter string="Application in Progress" name="application_in_progress" domain="[('applicant_ids.application_status', '=', 'ongoing')]"/>
                <filter string="Waiting" name="waiting" domain="[('applicant_ids', '=', False)]"/>
                <filter string="Hired" name="hired" domain="[('applicant_ids.application_status', '=', 'hired')]"/>
                <separator/>
                <filter string="Directly Available" name="directly_available"
                    domain="
                [
                    '&amp;',
                        '|',
                            ('availability', '&lt;=', context_today().strftime('%Y-%m-%d')),
                            ('availability', '=', False),
                        '!',
                            ('applicant_ids', 'any',
                                [
                                    '&amp;',
                                        ('application_status', '=', 'hired'),
                                    '|',
                                        ('availability', '&gt;=', context_today().strftime('%Y-%m-%d')),
                                        ('availability', '=', False),
                                ]
                            )
                ]" />
                <separator/>
                <filter string="Creation Date" name="filter_create" date="create_date"/>
                <separator/>
                <filter string="Refused" name="refused" context="{'active_test': False}" domain="[('applicant_ids.application_status', '=', 'refused')]"/>
                <filter string="Archived" name="archived" domain="[('active', '=', False)]"/>
                <group expand="0" string="Group By">
                    <filter string="Manager" name="group_by_manager" domain="[]" context="{'group_by': 'user_id'}"/>
                </group>
            </search>
        </field>
    </record>
    
    <record id="action_hr_candidate_mass_sms" model="ir.actions.act_window">
        <field name="name">Send SMS</field>
        <field name="res_model">sms.composer</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="context">{
            'default_composition_mode': 'mass',
            'default_mass_keep_log': True,
            'default_res_ids': active_ids,
        }</field>
        <field name="binding_model_id" ref="hr_recruitment.model_hr_candidate"/>
        <field name="binding_view_types">list</field>
    </record>

    <record id="action_candidate_send_mail" model="ir.actions.server">
        <field name="name">Send Email</field>
        <field name="model_id" ref="hr_recruitment.model_hr_candidate"/>
        <field name="binding_model_id" ref="hr_recruitment.model_hr_candidate"/>
        <field name="binding_view_types">list</field>
        <field name="state">code</field>
        <field name="code">action = records.action_send_email()</field>
    </record>

    <record id="action_hr_candidate" model="ir.actions.act_window">
        <field name="name">Candidates</field>
        <field name="path">recruitment-candidates</field>
        <field name="res_model">hr.candidate</field>
        <field name="view_mode">kanban,list,form,calendar,activity</field>
        <field name="context">{}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No candidates yet
            </p>
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
        <field name="arch" type="xml">
            <data>
                <xpath expr="//div[@name='kanban_primary_right']" position="inside">
                    <div t-if="record.new_applicant_count.raw_value > 0" class="row g-0 ml32" groups="hr_recruitment.group_hr_recruitment_user">
                        <a name="%(hr_applicant_action_from_department)d" class="col" type="action">
                            <field name="new_applicant_count"/> New Applicants
                        </a>
                    </div>
                </xpath>

                <xpath expr="//div[hasclass('o_kanban_manage_reports')]" position="inside">
                    <div role="menuitem">
                        <a class="dropdown-item" name="%(action_hr_recruitment_report_filtered_department)d"
                            type="action" groups="hr_recruitment.group_hr_recruitment_interviewer">
                            Recruitment
                        </a>
                    </div>
                </xpath>
            </data>
        </field>
    </record>

    <record id="action_hr_department" model="ir.actions.act_window">
        <field name="name">Departments</field>
        <field name="res_model">hr.department</field>
        <field name="view_mode">list,form</field>
    </record>
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
            <kanban highlight_color="color"
                class="o_hr_recruitment_kanban"
                on_create="hr_recruitment.create_job_simple"
                sample="1"
                limit="40"
                action="%(action_hr_job_applications)d"
                type="action"
                js_class="recruitment_kanban_view"
                >
                <field name="active"/>
                <field name="alias_email"/>
                <templates>
                    <t t-name="menu" groups="hr_recruitment.group_hr_recruitment_user">
                        <div class="container">
                            <div class="row">
                                <div class="col-6">
                                    <h5 role="menuitem" class="o_kanban_card_manage_title">
                                        <span>View</span>
                                    </h5>
                                    <div role="menuitem" name="menu_view_applications">
                                        <a name="%(action_hr_job_applications)d" type="action">Applications</a>
                                    </div>
                                    <div role="menuitem">
                                        <a name="action_open_activities" type="object">Activities</a>
                                    </div>
                                    <div role="menuitem" name="menu_view_job_posts">
                                        <a name="%(action_hr_job_sources)d" type="action" context="{'default_job_id': id}">Trackers</a>
                                    </div>
                                </div>
                                <div class="col-6">
                                    <h5 role="menuitem" class="o_kanban_card_manage_title">
                                        <span>New</span>
                                    </h5>
                                    <div role="menuitem" name="menu_new_applications">
                                        <a name="%(hr_recruitment.action_hr_applicant_new)d" type="action">Application</a>
                                    </div>
                                </div>
                                <div class="col-6">
                                    <h5 role="menuitem" class="o_kanban_card_manage_title">
                                        <span>Reporting</span>
                                    </h5>
                                    <div role="menuitem" name="kanban_job_reporting">
                                        <a name="%(hr_recruitment.action_hr_recruitment_report_filtered_job)d" type="action">Analysis</a>
                                    </div>
                                </div>
                            </div>
                            <div class="o_kanban_card_manage_settings row">
                                <div class="col-6" role="menuitem" aria-haspopup="true">
                                    <field name="color" widget="kanban_color_picker"/>
                                </div>
                                <div class="col-6" role="menuitem">
                                    <a class="dropdown-item" t-if="widget.editable" name="edit_job" type="open">Configuration</a>
                                    <a class="dropdown-item" t-if="record.active.raw_value" type="archive" >Archive</a>
                                    <a class="dropdown-item" t-if="!record.active.raw_value" name="toggle_active" type="object">Unarchive</a>
                                </div>
                            </div>
                        </div>
                    </t>
                    <t t-name="card">
                        <div class="d-flex align-items-baseline gap-1 ms-2">
                            <field name="is_favorite" widget="boolean_favorite" nolabel="1"/>
                            <div class="o_kanban_card_header_title d-flex flex-column">
                                <field name="name" class="fw-bold fs-4"/>
                                <field name="user_id" class="text-muted"/>
                                <div class="small" groups="base.group_multi_company">
                                    <i class="fa fa-building-o" role="img" aria-label="Company" title="Company"></i> <field name="company_id"/>
                                </div>
                                <div t-if="record.alias_email.value" class="small o_job_alias">
                                    <i class="fa fa-envelope-o" role="img" aria-label="Alias" title="Alias"></i> <field name="alias_id"/>
                                </div>
                            </div>
                        </div>
                        <div class="row g-0 mt-0 mt-sm-3 ms-2">
                            <div class="col-7">
                                <button class="btn btn-primary" name="%(action_hr_job_applications)d" type="action">
                                    <field name="new_application_count"/> New Applications
                                </button>
                            </div>
                            <div class="col-5">
                                <a name="edit_job" type="open" t-attf-class="{{ record.no_of_recruitment.raw_value > 0 ? 'text-primary fw-bolder' : 'text-secondary' }}" groups="hr_recruitment.group_hr_recruitment_user">
                                    <field name="no_of_recruitment"/> To Recruit
                                </a>
                                <span t-attf-class="{{ record.no_of_recruitment.raw_value > 0 ? 'text-primary fw-bolder' : 'text-secondary' }}" groups="!hr_recruitment.group_hr_recruitment_user">
                                    <field name="no_of_recruitment"/> To Recruit
                                </span>
                                <div t-if="record.application_count.raw_value > 0">
                                    <field name="application_count"/> Applications
                                </div>
                                <a t-if="record.activities_today.raw_value > 0" name="action_open_today_activities" type="object" class="text-warning"><field name="activities_today"/> Activities Today</a>
                                <br t-if="record.activities_today.raw_value > 0 and record.activities_overdue.raw_value > 0"/>
                                <a t-if="record.activities_overdue.raw_value > 0" name="action_open_late_activities" type="object" class="text-danger"><field name="activities_overdue"/> Late Activities</a>
                            </div>
                        </div>
                        <div name="kanban_boxes" class="row g-0 flex-nowrap mt-auto" groups="hr_recruitment.group_hr_recruitment_user"></div>
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
                            <field name="alias_domain_id" class="oe_inline" placeholder="e.g. mycompany.com"
                                   options="{'no_create': True, 'no_open': True}"/>
                        </div>
                        <div class="text-muted">Incoming emails create applications automatically. Use it for direct applications or when posting job offers on LinkedIn, Monster, etc.</div>
                    </div>
                </group>
                <footer>
                    <button string="Create" name="%(action_hr_job_applications)d" type="action" class="btn-primary o_create_job" data-hotkey="q"/>
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
           <xpath expr="//form" position="attributes">
                <attribute name="js_class">recruitment_form_view</attribute>
            </xpath>
            <page name="recruitment_page" position="attributes">
                <attribute name="invisible">0</attribute>
            </page>
            <page name="job_description_page" position="attributes">
                <attribute name="invisible">0</attribute>
            </page>
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
                <field name="industry_id"/>
                <label for="alias_name" string="Email Alias"
                       help="Define a specific contact address for this job position. If you keep it empty, the default email address will be used which is in human resources settings"/>
                <div name="alias_def">
                    <field name="alias_id" class="oe_read_only" string="Email Alias" required="0"/>
                    <div class="oe_edit_only" name="edit_alias">
                        <field name="alias_name" class="oe_inline" placeholder="e.g. jobs"/>@
                        <field name="alias_domain_id" class="oe_inline" placeholder="e.g. domain.com"
                               options="{'no_create': True, 'no_open': True}"/>
                    </div>
                </div>
            </xpath>
            <xpath expr="//field[@name='contract_type_id']" position="after">
                <field name="company_id" groups="base.group_multi_company"/>
                <field
                    name="date_from"
                    widget="daterange"
                    string="Mission Dates"
                    options="{'end_date_field': 'date_to'}"/>
                <field name="date_to" invisible="1" />
            </xpath>
            <xpath expr="//group" position="after">
                <field name="job_properties" columns="2"/>
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

            <xpath expr="//sheet" position="after">
                <chatter open_attachments="True"/>
            </xpath>
        </field>
    </record>

    <!-- hr related job position menu action -->
    <record model="ir.actions.act_window" id="action_hr_job_config">
        <field name="name">Job Positions</field>
        <field name="res_model">hr.job</field>
        <field name="view_mode">list,kanban,form</field>
        <field name="view_ids" eval="[(5, 0, 0),
            (0, 0, {'view_mode': 'list', 'view_id': ref('hr.view_hr_job_tree')}),
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
            <field name="name" position="after">
                <field name="department_id"/>
                <field name="no_of_recruitment"/>
                <field name="application_count" string="Applications" groups="hr_recruitment.group_hr_recruitment_interviewer"/>
            </field>
            <field name="no_of_employee" position="after">
                <field name="expected_employees" optional="hide"/>
                <field name="no_of_hired_employee" optional="hide"/>
                <field name="message_needaction" column_invisible="True"/>
                <field name="company_id" groups="base.group_multi_company" optional="hide"/>
                <field name="company_id" column_invisible="True"/>
                <field name="alias_name" column_invisible="True"/>
                <field name="alias_id" invisible="not alias_name" optional="hide"/>
                <field name="user_id" widget="many2one_avatar_user" optional="hide"/>
            </field>
            <list position="attributes">
                <attribute name="js_class">recruitment_list_view</attribute>
            </list>
        </field>
    </record>

    <!-- hr related job position menu action -->
    <record id="action_load_demo_data" model="ir.actions.server">
        <field name="name">Load demo data</field>
        <field name="model_id" ref="hr_recruitment.model_hr_job"/>
        <field name="state">code</field>
        <field name="code">action = model._action_load_recruitment_scenario()</field>
    </record>

    <record model="ir.actions.act_window" id="action_hr_job">
        <field name="name">Job Positions</field>
        <field name="path">recruitment</field>
        <field name="res_model">hr.job</field>
        <field name="view_mode">kanban,list,form</field>
        <field name="view_id" ref="hr_recruitment.view_hr_job_kanban"/>
        <field name="view_ids" eval="[(5, 0, 0),
            (0, 0, {'view_mode': 'kanban', 'view_id': ref('hr_recruitment.view_hr_job_kanban')}),
            (0, 0, {'view_mode': 'list', 'view_id': ref('hr_recruitment.hr_job_view_tree_inherit')}),
            (0, 0, {'view_mode': 'form', 'view_id': ref('hr.view_hr_job_form')})]"/>
        <field name="context">{}</field>
    </record>

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

    <record id="hr_job_platform_form" model="ir.ui.view">
        <field name="name">hr.job.platform.form</field>
        <field name="model">hr.job.platform</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group col="2">
                        <group>
                            <field name="name" placeholder="e.g. Linkedin"/>
                            <field name="regex" placeholder="e.g. ^New application:.*from (.*)"/>
                        </group>
                        <group>
                            <field name="email" placeholder="e.g. jobs-listings@linkedin.com"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="hr_job_platform_tree" model="ir.ui.view">
        <field name="name">hr.job.platform.list</field>
        <field name="model">hr.job.platform</field>
        <field name="arch" type="xml">
            <list>
                <field name="name"/>
                <field name="email"/>
                <field name="regex"/>
            </list>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_hr_job_platforms">
        <field name="name">Emails</field>
        <field name="res_model">hr.job.platform</field>
        <field name="view_mode">list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No rules have been defined.
            </p>
            <p>
                Create a new rule to process emails from specific job boards.
            </p>
            <p>
                Define a regex: Extract the applicant's name from the email's subject or body.
            </p>
            <p>
                Without a regex: The applicant's name will be the email's subject.
            </p>
        </field>
    </record>
</odoo>

```

## File: views\hr_recruitment_degree_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record model="ir.ui.view" id="hr_recruitment_degree_tree">
            <field name="name">hr.recruitment.degree.list</field>
            <field name="model">hr.recruitment.degree</field>
            <field name="arch" type="xml">
                <list string="Degree" editable="bottom">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                </list>
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
            <field name="name">Degrees</field>
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
        <record model="ir.ui.view" id="hr_recruitment_source_tree">
            <field name="name">hr.recruitment.source.list</field>
            <field name="model">hr.recruitment.source</field>
            <field name="arch" type="xml">
                <list string="Sources of Applicants" editable="top" sample="1">
                    <field name="has_domain" column_invisible="True"/>
                    <field name="source_id" placeholder="e.g. LinkedIn" decoration-bf="1" readonly="id"/>
                    <field name="medium_id" optional="hidden"/>
                    <field name="job_id" readonly="id"/>
                    <field name="email" widget="CopyClipboardChar"
                           invisible="not email or not has_domain"/>
                    <button name="create_alias" string="Generate Email" class="btn btn-primary" type="object" invisible="not has_domain or email"/>
                </list>
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
            <field name="name">Trackers</field>
            <field name="res_model">hr.recruitment.source</field>
            <field name="view_mode">list</field>
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
            <field name="name">hr.recruitment.stage.list</field>
            <field name="model">hr.recruitment.stage</field>
            <field name="arch" type="xml">
                <list string="Stages">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                    <field name="fold"/>
                    <field name="hired_stage"/>
                </list>
            </field>
        </record>

        <record id="view_hr_recruitment_stage_kanban" model="ir.ui.view">
            <field name="name">hr.recruitment.stage.kanban</field>
            <field name="model">hr.recruitment.stage</field>
            <field name="arch" type="xml">
                <kanban>
                    <templates>
                        <t t-name="card">
                            <field class="fw-bolder" name="name"/>
                            <div class="d-flex">
                                Folded in Recruitment Pipe:
                                <field name="fold" class="ms-1" widget="boolean"/>
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
                    <field name="requirements" placeholder="You can define the requirements here. They will be displayed when you hover over the stage title."/>
                </sheet>
                </form>
            </field>
        </record>

        <record id="hr_recruitment_stage_act" model="ir.actions.act_window">
            <field name="name">Stages</field>
            <field name="res_model">hr.recruitment.stage</field>
            <field name="view_mode">list,kanban,form</field>
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
                <list>
                    <field name="name" column_invisible="True"/>
                    <field name="res_id" column_invisible="True"/>
                    <field name="res_model" column_invisible="True"/>
                    <field name="datas" widget="binary" filename="name" string="File"/>
                    <field name="res_name" widget="applicant_char" string="Applicant" invisible="res_model != 'hr.applicant'"/>
                    <field name="create_date"/>
                </list>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\mail_activity_plan_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="mail_activity_plan_action_config_hr_applicant" model="ir.actions.act_window">
            <field name="name">Recruitment Plans</field>
            <field name="res_model">mail.activity.plan</field>
            <field name="view_mode">list,kanban,form</field>
            <field name="search_view_id" ref="mail.mail_activity_plan_view_search"/>
            <field name="context">{'default_res_model': 'hr.applicant'}</field>
            <field name="domain">[('res_model', '=', 'hr.applicant')]</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a Recruitment Activity Plan
                </p>
                <p>
                    Activity plans are used to assign a list of activities in just a few clicks
                    (e.g. "Language Test", "Prepare Offer", ...)
                </p>
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
        <field name="view_mode">list,kanban,form</field>
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
</odoo>
```

## File: views\menuitems.xml

```xml
<?xml version="1.0"?>
<odoo>
    <menuitem
        name="Recruitment"
        id="menu_hr_recruitment_root"
        web_icon="hr_recruitment,static/description/icon.png"
        groups="hr_recruitment.group_hr_recruitment_user,hr_recruitment.group_hr_recruitment_interviewer"
        sequence="210"/>

        <menuitem
            name="Applications"
            parent="menu_hr_recruitment_root"
            id="menu_crm_case_categ0_act_job"
            sequence="2"/>

            <menuitem
                name="By Job Positions"
                id="menu_hr_job_position"
                parent="menu_crm_case_categ0_act_job"
                action="action_hr_job"
                sequence="1"
                groups="hr_recruitment.group_hr_recruitment_user"/>

            <menuitem
                name="By Job Positions"
                id="menu_hr_job_position_interviewer"
                parent="menu_crm_case_categ0_act_job"
                action="action_hr_job_interviewer"
                sequence="1"
                groups="hr_recruitment.group_hr_recruitment_interviewer"/>

            <menuitem
                name="All Applications"
                parent="menu_crm_case_categ0_act_job"
                id="menu_crm_case_categ_all_app"
                action="crm_case_categ0_act_job"
                sequence="2"/>

            <menuitem
                name="Candidates"
                parent="menu_crm_case_categ0_act_job"
                id="menu_hr_candidate"
                action="action_hr_candidate"
                sequence="3"/>

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

        <menuitem
            id="menu_hr_recruitment_configuration"
            name="Configuration"
            parent="menu_hr_recruitment_root"
            groups="group_hr_recruitment_user"
            sequence="100"/>

            <menuitem
                id="menu_hr_recruitment_global_settings"
                name="Settings"
                parent="menu_hr_recruitment_configuration"
                sequence="0"
                action="action_hr_recruitment_configuration"
                groups="base.group_system"/>

            <menuitem
                id="menu_hr_recruitment_config_jobs"
                name="Job Positions"
                parent="menu_hr_recruitment_configuration"
                sequence="10"/>

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

            <menuitem
                id="menu_hr_recruitment_config_applications"
                name="Applications"
                parent="menu_hr_recruitment_configuration"
                sequence="20"/>

                <menuitem
                    id="menu_hr_recruitment_degree"
                    name="Degrees"
                    parent="menu_hr_recruitment_config_applications"
                    action="hr_recruitment_degree_action"
                    sequence="1"
                    />

                <menuitem
                    id="menu_hr_applicant_refuse_reason"
                    action="hr_applicant_refuse_reason_action"
                    parent="menu_hr_recruitment_config_applications"
                    sequence="10"/>

                <menuitem
                    id="hr_applicant_category_menu"
                    parent="menu_hr_recruitment_config_applications"
                    action="hr_applicant_category_action"
                    sequence="20"
                    />

            <menuitem
                id="menu_hr_recruitment_config_employees"
                name="Employees"
                parent="menu_hr_recruitment_configuration"
                sequence="30"/>

                <menuitem
                    name="Departments"
                    id="menu_hr_department"
                    parent="menu_hr_recruitment_config_employees"
                    action="action_hr_department"/>

            <menuitem
                id="menu_hr_recruitment_config_activities"
                name="Activities"
                parent="menu_hr_recruitment_configuration"
                sequence="40"/>

                <menuitem
                    id="hr_recruitment_menu_config_activity_type"
                    action="mail_activity_type_action_config_hr_applicant"
                    parent="menu_hr_recruitment_config_activities"
                    sequence="10"/>

                <menuitem
                    name="Activity Plans"
                    id="hr_recruitment_menu_config_activity_plan"
                    parent="menu_hr_recruitment_config_activities"
                    action="mail_activity_plan_action_config_hr_applicant"
                    groups="hr_recruitment.group_hr_recruitment_manager"
                    sequence="20"/>

            <menuitem
                name="Job Boards"
                id="menu_hr_job_boards"
                parent="menu_hr_recruitment_configuration"
                sequence="50"/>

                <menuitem
                    name="Emails"
                    id="menu_hr_recruitment_emails"
                    parent="menu_hr_job_boards"
                    action="action_hr_job_platforms"
                    sequence="50"/>






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
                            <setting id="publish_available_jobs_setting" string="Online Posting" help="Publish job offers on your website">
                                <field name="module_website_hr_recruitment"/>
                            </setting>
                        </block>
                        <block title="Process" name="recruitment_process_div">
                            <setting id="interview_forms_setting" string="Send Interview Survey" help="Send an Interview Survey to the applicant during the recruitment process" title="Use interview forms tailored to each job position during the recruitment process. Select the form to use in the job position detail form. This relies on the Survey app.">
                                <field name="module_hr_recruitment_survey"/>
                            </setting>
                            <setting string="Résumé Display" help="Display résumé on application form" id="display_cv">
                                <field name="group_applicant_cv_display"/>
                            </setting>
                        </block>
                        <block title="In-App Purchases" name="recruitment_in_app_purchases">
                            <setting id="sms" string="Send SMS" documentation="/applications/marketing/sms_marketing/pricing/pricing_and_faq.html" help="Send texts to your contacts">
                            </setting>
                            <setting id="recruitment_extract_settings" string="Résumé Digitization (OCR)" company_dependent="1" help="Digitize your résumé to extract name and email automatically." title="Use OCR to fill data from a picture of the Résumr or the file itself">
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
    </data>
</odoo>

```

## File: wizard\applicant_refuse_reason.py

```python
# -*- coding: utf-8 -*-

from datetime import datetime
from markupsafe import Markup

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.osv import expression


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
    duplicates = fields.Boolean('Duplicates')
    duplicates_count = fields.Integer('Duplicates Count', compute='_compute_duplicates_count')
    single_applicant_email = fields.Char(compute='_compute_single_applicant_email', inverse="_inverse_single_applicant_email")

    @api.depends('refuse_reason_id')
    def _compute_send_mail(self):
        for wizard in self:
            template = wizard.refuse_reason_id.template_id
            wizard.send_mail = template and not wizard.applicant_without_email
            wizard.template_id = template

    @api.depends('applicant_ids', 'single_applicant_email')
    def _compute_applicant_without_email(self):
        for wizard in self:
            applicants = wizard.applicant_ids.filtered(lambda x: not x.email_from and not x.partner_id.email)
            if applicants and not wizard.single_applicant_email:
                wizard.applicant_without_email = "%s\n%s" % (
                    _("You can't select Send email option.\nThe email will not be sent to the following applicant(s) as they don't have an email address:"),
                    ",\n".join([i.partner_name or i.display_name for i in applicants])
                )
            else:
                wizard.applicant_without_email = False

    @api.depends("applicant_ids.email_from")
    def _compute_single_applicant_email(self):
        for wizard in self:
            if len(wizard.applicant_ids) == 1:
                wizard.single_applicant_email = (
                    wizard.applicant_ids.email_from
                    or wizard.applicant_ids.partner_id.email
                )

    def _inverse_single_applicant_email(self):
        for wizard in self:
            if len(wizard.applicant_ids) == 1:
                wizard.applicant_ids.email_from = wizard.single_applicant_email

    @api.depends('applicant_ids.email_from')
    def _compute_applicant_emails(self):
        for wizard in self:
            wizard.applicant_emails = ', '.join(a.email_from or a.partner_id.email for a in wizard.applicant_ids if a.email_from or a.partner_id.email)

    def action_refuse_reason_apply(self):
        if self.send_mail:
            if not self.template_id:
                raise UserError(_("Email template must be selected to send a mail"))
            if any(not (applicant.email_from or applicant.partner_id.email) for applicant in self.applicant_ids):
                raise UserError(_("At least one applicant doesn't have a email; you can't use send email option."))

        refused_applications = self.applicant_ids
        # duplicates_count can be true only if only one application is selected
        if self.duplicates_count and self.duplicates:
            applicant_id = self.applicant_ids[0]
            duplicate_domain = applicant_id.candidate_id._get_similar_candidates_domain()
            duplicates = self.env['hr.candidate'].search(duplicate_domain).applicant_ids
            refused_applications |= duplicates
            url = applicant_id._get_html_link()
            message = _(
                "Refused automatically because this application has been identified as a duplicate of %(link)s",
                link=url)
            duplicates._message_log_batch(bodies={duplicate.id: message for duplicate in duplicates})
        refused_applications.write({'refuse_reason_id': self.refuse_reason_id.id, 'active': False, 'refuse_date': datetime.now()})

        if self.send_mail:
            # TDE note: keeping 16.0 behavior, clean me please
            message_values = {
                'email_layout_xmlid' : 'hr_recruitment.mail_notification_light_without_background',
            }
            if len(self.applicant_ids) > 1:
                self.applicant_ids.with_context(active_test=True).message_mail_with_source(
                    self.template_id,
                    auto_delete_keep_log=True,
                    **message_values
                )
            else:
                self.applicant_ids.with_context(active_test=True).message_post_with_source(
                    self.template_id,
                    subtype_xmlid='mail.mt_note',
                    **message_values
                )

    @api.depends('applicant_ids')
    def _compute_duplicates_count(self):
        self.duplicates_count = self.applicant_ids.other_applications_count if len(self.applicant_ids) == 1 else 0

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
                        <field name="refuse_reason_id" string="Reason" widget="selection_badge" options="{'horizontal': true, 'no_create': True, 'no_open': True}"/>
                        <group invisible="not refuse_reason_id">
                            <label for="send_mail" class="me-2" invisible="not refuse_reason_id"/>
                            <div class="d-flex">
                                <field name="send_mail" readonly="applicant_ids.length > 1 and applicant_without_email"/>
                                <span class="mx-2" style="padding-top: 1px; padding-bottom: 1px;">to</span>
                                <field name="applicant_emails" invisible="applicant_ids.length == 1"/>
                                <field
                                    name="single_applicant_email"
                                    placeholder="Provide an email"
                                    invisible="applicant_ids.length != 1"
                                    required="applicant_ids.length == 1 and send_mail"/>
                            </div>
                            <field name="template_id" invisible="not send_mail" required="send_mail"/>
                            <label for="duplicates" invisible="duplicates_count == 0"/>
                            <div class="d-flex" invisible="duplicates_count == 0">
                                <field name="duplicates" nolabel="1"/>
                                <span>Refuse the<field name="duplicates_count" class="oe_inline mx-1"/>other application(s)</span>
                            </div>
                        </group>
                    </group>
                    <div
                        class="alert alert-danger"
                        role="alert"
                        invisible="applicant_ids.length == 1 or not applicant_without_email or not refuse_reason_id">
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
    attachment_ids = fields.Many2many('ir.attachment', string='Attachments', readonly=False, store=True)

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
                    'message': _("The following applicants are missing an email address: %s.", ', '.join(without_emails.mapped(lambda a: a.partner_name or a.display_name))),
                }
            }

        if self.template_id:
            subjects = self.template_id._render_field('subject', res_ids=self.applicant_ids.ids)
        else:
            subjects = {applicant.id: self.subject for applicant in self.applicant_ids}

        for applicant in self.applicant_ids:
            if not applicant.partner_id:
                applicant.partner_id = self.env['res.partner'].create({
                    'is_company': False,
                    'name': applicant.partner_name,
                    'email': applicant.email_from,
                    'phone': applicant.partner_phone,
                    'mobile': applicant.partner_phone,
                })

            attachment_ids = []
            for attachment_id in self.attachment_ids:
                new_attachment = attachment_id.copy({'res_model': 'hr.applicant', 'res_id': applicant.id})
                attachment_ids.append(new_attachment.id)

            applicant.message_post(
                author_id=self.author_id.id,
                body=self.body,
                email_layout_xmlid='mail.mail_notification_light',
                message_type='comment',
                partner_ids=applicant.partner_id.ids,
                subject=subjects[applicant.id],
                attachment_ids=attachment_ids
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
                        widget="html_mail"
                        placeholder="Write your message here..."
                        force_save="1"/>
                <group>
                            <field name="attachment_ids" widget="many2many_binary" string="Attach a file" nolabel="1" colspan="2"/>
                            <field name="template_id" string="Load template" options="{'no_create': True}"/>
                </group>
                <footer>
                    <button name="action_send" string="Send" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\candidate_send_mail.py

```python
from odoo import api, fields, models, _


class CandidateSendMail(models.TransientModel):
    _name = "candidate.send.mail"
    _inherit = "mail.composer.mixin"
    _description = "Send mails to candidates"

    candidate_ids = fields.Many2many("hr.candidate", string="Candidates", required=True)
    author_id = fields.Many2one(
        "res.partner",
        "Author",
        required=True,
        default=lambda self: self.env.user.partner_id.id,
    )
    attachment_ids = fields.Many2many(
        "ir.attachment", string="Attachments", readonly=False, store=True
    )

    @api.depends("subject")
    def _compute_render_model(self):
        self.render_model = "hr.candidate"

    def action_send(self):
        self.ensure_one()

        without_emails = self.candidate_ids.filtered(
            lambda c: not c.email_from or (c.partner_id and not c.partner_id.email)
        )
        if without_emails:
            return {
                "type": "ir.actions.client",
                "tag": "display_notification",
                "params": {
                    "type": "danger",
                    "message": _(
                        "The following candidates are missing an email address: %s.",
                        ", ".join(
                            without_emails.mapped(
                                lambda c: c.partner_name or c.display_name
                            )
                        ),
                    ),
                },
            }

        if self.template_id:
            subjects = self.template_id._render_field(
                "subject", res_ids=self.candidate_ids.ids
            )
        else:
            subjects = {candidate.id: self.subject for candidate in self.candidate_ids}

        for candidate in self.candidate_ids:
            if not candidate.partner_id:
                candidate.partner_id = self.env["res.partner"].create(
                    {
                        "is_company": False,
                        "name": candidate.partner_name,
                        "email": candidate.email_from,
                        "phone": candidate.partner_phone,
                        "mobile": candidate.partner_phone,
                    }
                )

            attachment_ids = []
            for attachment_id in self.attachment_ids:
                new_attachment = attachment_id.copy(
                    {"res_model": "hr.candidate", "res_id": candidate.id}
                )
                attachment_ids.append(new_attachment.id)

            candidate.message_post(
                author_id=self.author_id.id,
                body=self.body,
                email_layout_xmlid="mail.mail_notification_light",
                message_type="comment",
                partner_ids=candidate.partner_id.ids,
                subject=subjects[candidate.id],
                attachment_ids=attachment_ids,
            )

```

## File: wizard\candidate_send_mail_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="candidate_send_mail_view_form" model="ir.ui.view">
        <field name="model">candidate.send.mail</field>
        <field name="arch" type="xml">
            <form>
                <field name="author_id" invisible="1" />
                <field name="lang" invisible="1" />
                <field name="render_model" invisible="1" />
                <field name="template_id" invisible="1" />
                <group>
                    <field name="subject" required="1" />
                    <field name="candidate_ids" widget="many2many_tags"
                        context="{'show_partner_name': 1}" />
                </group>
                <field name="body" nolabel="1" class="oe-bordered-editor"
                    widget="html_mail"
                    placeholder="Write your message here..."
                    force_save="1" />
                <group>
                    <field name="attachment_ids" widget="many2many_binary" string="Attach a file"
                        nolabel="1" colspan="2" />
                    <field name="template_id" string="Load template" options="{'no_create': True}" />
                </group>
                <footer>
                    <button name="action_send" string="Send" type="object" class="btn-primary"
                        data-hotkey="q" />
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x" />
                </footer>
            </form>
        </field>
    </record>
</odoo>
```

## File: wizard\mail_activity_schedule.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MailActivitySchedule(models.TransientModel):
    _inherit = 'mail.activity.schedule'

    def _compute_plan_department_filterable(self):
        super()._compute_plan_department_filterable()
        for wizard in self:
            if not wizard.plan_department_filterable:
                wizard.plan_department_filterable = wizard.res_model == 'hr.applicant'

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import applicant_refuse_reason
from . import applicant_send_mail
from . import candidate_send_mail
from . import mail_activity_schedule

```

