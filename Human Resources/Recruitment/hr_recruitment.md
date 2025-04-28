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
        'data/mail_alias_data.xml',
        'data/mail_message_subtype_data.xml',
        'data/mail_template_data.xml',
        'data/mail_templates.xml',
        'data/hr_recruitment_data.xml',
        'views/hr_recruitment_views.xml',
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
        <t t-if="record and record.alias_domain">
            <a t-attf-href="mailto:{{record.alias_id.display_name}}" target="_blank" style="color: #875a7b; text-decoration: none;">Try sending an email</a>
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

</data>
</odoo>

```

## File: data\hr_recruitment_demo.xml

```xml
<?xml version="1.0"?>
<odoo noupdate="1">
    <record id="base.user_demo" model="res.users">
        <field name="groups_id" eval="[(4, ref('hr_recruitment.group_hr_recruitment_user'))]"/>
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
        <field name="email_from">thing.thang.thong@gmail.example.com</field>
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
        <field name="email_from">coincoin@gmail.example.com</field>
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
        <field name="email_from">0h3n-rijaune@yahoo.example.com</field>
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

## File: data\mail_alias_data.xml

```xml
<?xml version="1.0"?>
<odoo>
<data noupdate="1">
    <record id="mail_alias_jobs" model="mail.alias">
        <field name="alias_name">jobs</field>
        <field name="alias_model_id" ref="model_hr_applicant"/>
        <field name="alias_user_id" ref="base.user_admin"/>
        <field name="alias_parent_model_id" ref="model_hr_job"/>
    </record>
</data>
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
Applicant hired<br/>
<ul>
<li>Employee: <a href="#" t-att-data-oe-id="applicant.emp_id.id" data-oe-model="hr.employee"><t t-esc="applicant.emp_id.name"/></a></li>
</ul>
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


class CalendarEvent(models.Model):
    """ Model for Calendar Event """
    _inherit = 'calendar.event'

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
        compute='_compute_new_applicant_count', string='New Applicant', compute_sudo=True)
    new_hired_employee = fields.Integer(
        compute='_compute_recruitment_stats', string='New Hired Employee')
    expected_employee = fields.Integer(
        compute='_compute_recruitment_stats', string='Expected Employee')

    def _compute_new_applicant_count(self):
        if self.env.user.has_group('hr_recruitment.group_hr_recruitment_interviewer'):
            applicant_data = self.env['hr.applicant']._read_group(
                [('department_id', 'in', self.ids), ('stage_id.sequence', '<=', '1')],
                ['department_id'], ['department_id'])
            result = dict((data['department_id'][0], data['department_id_count']) for data in applicant_data)
            for department in self:
                department.new_applicant_count = result.get(department.id, 0)
        else:
            self.new_applicant_count = 0

    def _compute_recruitment_stats(self):
        job_data = self.env['hr.job']._read_group(
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

    @api.model_create_multi
    def create(self, vals_list):
        employees = super().create(vals_list)
        for employee in employees:
            if employee.applicant_id:
                employee.applicant_id._message_log_with_view(
                    'hr_recruitment.applicant_hired_template',
                    values={'applicant': employee.applicant_id},
                    subtype_id=self.env.ref("hr_recruitment.mt_applicant_hired").id)
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
        help="Address where employees are working")
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
    hr_responsible_id = fields.Many2one(
        'res.users', "HR Responsible", tracking=True,
        help="Person responsible of validating the employee's contracts.")
    document_ids = fields.One2many('ir.attachment', compute='_compute_document_ids', string="Documents", readonly=True)
    documents_count = fields.Integer(compute='_compute_document_ids', string="Document Count")
    alias_id = fields.Many2one(
        'mail.alias', "Alias", ondelete="restrict", required=True,
        help="Email alias for this job position. New emails will automatically create new applicants for this job position.")
    color = fields.Integer("Color Index")
    is_favorite = fields.Boolean(compute='_compute_is_favorite', inverse='_inverse_is_favorite')
    favorite_user_ids = fields.Many2many('res.users', 'job_favorite_user_rel', 'job_id', 'user_id', default=_get_default_favorite_user_ids)
    interviewer_ids = fields.Many2many('res.users', string='Interviewers', domain="[('share', '=', False), ('company_ids', 'in', company_id)]", help="The Interviewers set on the job position can see all Applicants in it. They have access to the information, the attachments, the meeting management and they can refuse him. You don't need to have Recruitment rights to be set as an interviewer.")
    extended_interviewer_ids = fields.Many2many('res.users', 'hr_job_extended_interviewer_res_users', compute='_compute_extended_interviewer_ids', store=True)

    activities_overdue = fields.Integer(compute='_compute_activities')
    activities_today = fields.Integer(compute='_compute_activities')

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
        ], ['job_id'], ['job_id'])
        result = dict((data['job_id'][0], data['job_id_count']) for data in read_group_result)
        for job in self:
            job.all_application_count = result.get(job.id, 0)

    def _compute_application_count(self):
        read_group_result = self.env['hr.applicant']._read_group([('job_id', 'in', self.ids)], ['job_id'], ['job_id'])
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
        ], ['job_id'], ['job_id'])
        job_hires = {data['job_id'][0]: data['job_id_count'] for data in hired_data}
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
            if vals.get('alias_name'):
                vals['alias_user_id'] = False
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

## File: models\hr_recruitment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint

from odoo import api, fields, models, tools, SUPERUSER_ID
from odoo.exceptions import AccessError, UserError
from odoo.tools import Query
from odoo.tools.translate import _

from dateutil.relativedelta import relativedelta

from lxml import etree

AVAILABLE_PRIORITIES = [
    ('0', 'Normal'),
    ('1', 'Good'),
    ('2', 'Very Good'),
    ('3', 'Excellent')
]


class RecruitmentSource(models.Model):
    _name = "hr.recruitment.source"
    _description = "Source of Applicants"
    _inherit = ['utm.source.mixin']

    email = fields.Char(related='alias_id.display_name', string="Email", readonly=True)
    has_domain = fields.Char(compute='_compute_has_domain')
    job_id = fields.Many2one('hr.job', "Job", ondelete='cascade')
    alias_id = fields.Many2one('mail.alias', "Alias ID")
    medium_id = fields.Many2one('utm.medium', default=lambda self: self.env.ref('utm.utm_medium_website'))

    def _compute_has_domain(self):
        self.has_domain = bool(self.env["ir.config_parameter"].sudo().get_param("mail.catchall.domain"))

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

            # check that you can create source before to call mail.alias in sudo with known/controlled vals
            source.check_access_rights('create')
            source.check_access_rule('create')
            source.alias_id = self.env['mail.alias'].sudo().create(vals)

    @api.model
    def _get_view(self, view_id=None, view_type='form', **options):
        arch, view = super()._get_view(view_id, view_type, **options)
        if view_type == 'tree' and not bool(self.env["ir.config_parameter"].sudo().get_param("mail.catchall.domain")):
            email = arch.xpath("//field[@name='email']")[0]
            email.getparent().remove(email)
        return arch, view

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
        applicant_data = self.env['hr.applicant']._read_group([('stage_id', 'in', self.ids)], ['stage_id'], 'stage_id')
        applicants = dict((data['stage_id'][0], data['stage_id_count']) for data in applicant_data)
        for stage in self:
            if stage._origin.hired_stage and not stage.hired_stage and applicants.get(stage._origin.id):
                stage.is_warning_visible = True
            else:
                stage.is_warning_visible = False

class RecruitmentDegree(models.Model):
    _name = "hr.recruitment.degree"
    _description = "Applicant Degree"
    _sql_constraints = [
        ('name_uniq', 'unique (name)', 'The name of the Degree of Recruitment must be unique!')
    ]

    name = fields.Char("Degree Name", required=True, translate=True)
    sequence = fields.Integer("Sequence", default=1)


class Applicant(models.Model):
    _name = "hr.applicant"
    _description = "Applicant"
    _order = "priority desc, id desc"
    _inherit = ['mail.thread.cc', 'mail.activity.mixin', 'utm.mixin']
    _mailing_enabled = True

    name = fields.Char("Subject / Application", required=True, help="Email subject for applications sent via email", index='trigram')
    active = fields.Boolean("Active", default=True, help="If the active field is set to false, it will allow you to hide the case without removing it.")
    description = fields.Html("Description")
    email_from = fields.Char("Email", size=128, compute='_compute_partner_phone_email',
        inverse='_inverse_partner_email', store=True)
    probability = fields.Float("Probability")
    partner_id = fields.Many2one('res.partner', "Contact", copy=False)
    create_date = fields.Datetime("Creation Date", readonly=True)
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
    ], compute="_compute_application_status")

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
                applicant.delay_close = applicant.day_close - applicant.day_open
            else:
                applicant.day_close = False
                applicant.delay_close = False

    @api.depends('email_from', 'partner_phone', 'partner_mobile')
    def _compute_application_count(self):
        self.flush_model(['email_from'])
        applicants = self.env['hr.applicant']
        for applicant in self:
            if applicant.email_from or applicant.partner_phone or applicant.partner_mobile:
                applicants |= applicant
        # Done via SQL since read_group does not support grouping by lowercase field
        if applicants.ids:
            query = Query(self.env.cr, self._table, self._table_query)
            query.add_where('hr_applicant.id in %s', [tuple(applicants.ids)])
            # Count into the companies that are selected from the multi-company widget
            company_ids = self.env.context.get('allowed_company_ids')
            if company_ids:
                query.add_where('other.company_id in %s', [tuple(company_ids)])
            self._apply_ir_rules(query)
            from_clause, where_clause, where_clause_params = query.get_sql()
            # In case the applicant phone or mobile is configured in wrong field
            query_str = """
            SELECT hr_applicant.id as appl_id,
                COUNT(other.id) as count
              FROM hr_applicant
              JOIN hr_applicant other ON LOWER(other.email_from) = LOWER(hr_applicant.email_from)
                OR other.partner_phone = hr_applicant.partner_phone OR other.partner_phone = hr_applicant.partner_mobile
                OR other.partner_mobile = hr_applicant.partner_mobile OR other.partner_mobile = hr_applicant.partner_phone
            %(where)s
        GROUP BY hr_applicant.id
            """ % {
                'where': ('WHERE %s' % where_clause) if where_clause else '',
            }
            self.env.cr.execute(query_str, where_clause_params)
            application_data_mapped = dict((data['appl_id'], data['count']) for data in self.env.cr.dictfetchall())
        else:
            application_data_mapped = dict()
        for applicant in applicants:
            applicant.application_count = application_data_mapped.get(applicant.id, 1) - 1
        (self - applicants).application_count = False

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
            elif applicant.date_closed:
                applicant.application_status = 'hired'
            else:
                applicant.application_status = 'ongoing'

    def _get_attachment_number(self):
        read_group_res = self.env['ir.attachment']._read_group(
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
            if applicant.partner_id:
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

    @api.depends('stage_id.hired_stage')
    def _compute_date_closed(self):
        for applicant in self:
            if applicant.stage_id and applicant.stage_id.hired_stage and not applicant.date_closed:
                applicant.date_closed = fields.datetime.now()
            if not applicant.stage_id.hired_stage:
                applicant.date_closed = False

    def _check_interviewer_access(self):
        if self.user_has_groups('hr_recruitment.group_hr_recruitment_interviewer'):
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

    def get_empty_list_help(self, help):
        if 'active_id' in self.env.context and self.env.context.get('active_model') == 'hr.job':
            alias_id = self.env['hr.job'].browse(self.env.context['active_id']).alias_id
        else:
            alias_id = False

        nocontent_values = {
            'help_title': _("No application found. Let's create one !"),
            'para_1': _('People can also apply by email to save time.'),
            'para_2': _("You can search into attachment's content, like resumes, with the searchbar."),
        }
        nocontent_body = """
            <p class="o_view_nocontent_smiling_face">%(help_title)s</p>
            <p>%(para_1)s<br/>%(para_2)s</p>"""

        if alias_id and alias_id.alias_domain and alias_id.alias_name:
            email = alias_id.display_name
            email_link = "<a href='mailto:%s'>%s</a>" % (email, email)
            nocontent_values['email_link'] = email_link
            nocontent_body += """<p class="o_copy_paste_email">%(email_link)s</p>"""

        return nocontent_body % nocontent_values

    @api.model
    def get_view(self, view_id=None, view_type='form', **options):
        if view_type == 'form' and self.user_has_groups('hr_recruitment.group_hr_recruitment_interviewer'):
            view_id = self.env.ref('hr_recruitment.hr_applicant_view_form_interviewer').id
        return super().get_view(view_id, view_type, **options)

    def _notify_get_recipients(self, message, msg_vals, **kwargs):
        """
            Do not notify members of the Recruitment Interviewer group, as this
            might leak some data they shouldn't have access to.
        """
        recipients = super()._notify_get_recipients(message, msg_vals, **kwargs)
        interviewer_group = self.env.ref('hr_recruitment.group_hr_recruitment_interviewer').id
        return [recipient for recipient in recipients if interviewer_group not in recipient['groups']]

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
                'type': 'private',
                'name': self.partner_name,
                'email': self.email_from,
                'phone': self.partner_phone,
                'mobile': self.partner_mobile
            })

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
        self.env.cr.execute("""
        SELECT other.id
          FROM hr_applicant
          JOIN hr_applicant other ON LOWER(other.email_from) = LOWER(hr_applicant.email_from)
            OR other.partner_phone = hr_applicant.partner_phone OR other.partner_phone = hr_applicant.partner_mobile
            OR other.partner_mobile = hr_applicant.partner_mobile OR other.partner_mobile = hr_applicant.partner_phone
         WHERE hr_applicant.id in %s
        """, (tuple(self.ids),)
        )
        ids = [res['id'] for res in self.env.cr.dictfetchall()]
        return {
            'type': 'ir.actions.act_window',
            'name': _('Job Applications'),
            'res_model': self._name,
            'view_mode': 'tree,kanban,form,pivot,graph,calendar,activity',
            'domain': [('id', 'in', ids)],
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
        if 'stage_id' in changes and applicant.stage_id.template_id:
            res['stage_id'] = (applicant.stage_id.template_id, {
                'auto_delete_message': True,
                'subtype_id': self.env['ir.model.data']._xmlid_to_res_id('mail.mt_note'),
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

    def name_get(self):
        if self.env.context.get('show_partner_name'):
            return [
                (applicant.id, applicant.partner_name or applicant.name)
                for applicant in self
            ]
        return super().name_get()

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
        partner_name, email_from = self.env['res.partner']._parse_partner_name(msg.get('from'))
        defaults = {
            'name': msg.get('subject') or _("No Subject"),
            'partner_name': partner_name or email_from,
            'email_from': email_from,
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
                        'name': self.partner_name or self.email_from,
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
        """ Create an employee from applicant """
        self.ensure_one()
        self._check_interviewer_access()

        contact_name = False
        if self.partner_id:
            address_id = self.partner_id.address_get(['contact'])['contact']
            contact_name = self.partner_id.display_name
        else:
            if not self.partner_name:
                raise UserError(_('You must define a Contact Name for this applicant.'))
            new_partner_id = self.env['res.partner'].create({
                'is_company': False,
                'type': 'private',
                'name': self.partner_name,
                'email': self.email_from,
                'phone': self.partner_phone,
                'mobile': self.partner_mobile
            })
            self.partner_id = new_partner_id
            address_id = new_partner_id.address_get(['contact'])['contact']
        employee_data = {
            'default_name': self.partner_name or contact_name,
            'default_job_id': self.job_id.id,
            'default_job_title': self.job_id.name,
            'default_address_home_id': address_id,
            'default_department_id': self.department_id.id,
            'default_address_id': self.company_id.partner_id.id,
            'default_work_email': self.department_id.company_id.email or self.email_from, # To have a valid email address by default
            'default_work_phone': self.department_id.company_id.phone,
            'form_view_initial_mode': 'edit',
            'default_applicant_id': self.ids,
        }
        dict_act_window = self.env['ir.actions.act_window']._for_xml_id('hr.open_view_employee_list')
        dict_act_window['context'] = employee_data
        return dict_act_window

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
    template_id = fields.Many2one('mail.template', string='Email Template', domain="[('model', '=', 'hr.applicant')]")
    active = fields.Boolean('Active', default=True)

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
        if self.env.user.has_group('hr_recruitment.group_hr_recruitment_interviewer'):
            res.append(self.env.ref('hr_recruitment.menu_hr_job_position').id)
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

        job_interviewers = self.env['hr.job']._read_group([('interviewer_ids', 'in', self.ids)], ['interviewer_ids'], ['interviewer_ids'])
        user_ids = {j['interviewer_ids'][0] for j in job_interviewers}

        application_interviewers = self.env['hr.applicant']._read_group([('interviewer_ids', 'in', self.ids)], ['interviewer_ids'], ['interviewer_ids'])
        user_ids |= {a['interviewer_ids'][0] for a in application_interviewers}

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
from . import hr_recruitment
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
        <field name="description">The user interacting with the application as interviewer don't need any specific access. They'll have access thanks to their interviewer assignation.</field>
        <field name="sequence">11</field>
    </record>

    <record id="hr_applicant_comp_rule" model="ir.rule">
        <field name="name">Applicant multi company rule</field>
        <field name="model_id" ref="model_hr_applicant"/>
        <field eval="True" name="global"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="group_hr_recruitment_interviewer" model="res.groups">
        <field name="name">Recruitment Interviewer</field>
        <field name="category_id" ref="base.module_category_hidden"/>
        <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record id="group_hr_recruitment_user" model="res.groups">
        <field name="name">Officer : Manage all applicants</field>
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

    <record id="mail_message_interviewer_rule" model="ir.rule">
        <field name="name">Interviewer: No Applicant Chatter</field>
        <field name="model_id" ref="mail.model_mail_message"/>
        <field name="domain_force">[
            '|',
                ('model', '!=', 'hr.applicant'),
                '&amp;',
                    ('model', '=', 'hr.applicant'),
                    ('mail_activity_type_id', '!=', False)
        ]</field>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]"/>
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
access_hr_recruitment_source_all,hr.recruitment.source,model_hr_recruitment_source,,1,0,0,0
access_hr_applicant_category,hr.applicant_category,model_hr_applicant_category,,1,1,1,0
access_hr_applicant_category_manager,hr.applicant_category,model_hr_applicant_category,group_hr_recruitment_user,1,1,1,1
access_calendar_event_type_hr_officer,calendar.event.type.officer,calendar.model_calendar_event_type,group_hr_recruitment_user,1,1,1,0
access_applicant_get_refuse_reason,access.applicant.get.refuse.reason,model_applicant_get_refuse_reason,hr_recruitment.group_hr_recruitment_user,1,1,1,0
access_applicant_get_refuse_reason_interviewer,access.applicant.get.refuse.reason.interviewer,model_applicant_get_refuse_reason,hr_recruitment.group_hr_recruitment_interviewer,1,1,1,0
access_applicant_send_mail,access.applicant.send.mail,model_applicant_send_mail,hr_recruitment.group_hr_recruitment_user,1,1,1,0
access_applicant_send_mail_interviewer,access.applicant.send.mail.interviewer,model_applicant_send_mail,hr_recruitment.group_hr_recruitment_interviewer,1,1,1,0

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

## File: static\src\applicant_char\applicant_char.js

```javascript
/** @odoo-module */

import { CharField } from "@web/views/fields/char/char_field";
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
registry.category("fields").add("applicant_char", ApplicantCharField);

```

## File: static\src\applicant_char\applicant_char.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="hr_recruitment.ApplicantCharField" t-inherit="web.CharField" t-inherit-mode="primary" owl="1">
        <xpath expr="//span[@t-esc='formattedValue']" position="attributes">
            <attribute name="t-on-click.prevent.stop">onClick</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\js\tours\hr_recruitment.js

```javascript
odoo.define('hr_recruitment.tour', function(require) {
"use strict";

const {_t} = require('web.core');
const {Markup} = require('web.utils');
var tour = require('web_tour.tour');

const { markup } = owl;

tour.register('hr_recruitment_tour',{
    url: "/web",
    rainbowManMessage: markup(_t("<div>Great job! You hired a new colleague!</div><div>Try the Website app to publish job offers online.</div>")),
    fadeout: 'very_slow',
    sequence: 230,
}, [tour.stepUtils.showAppsMenuItem(), {
    trigger: '.o_app[data-menu-xmlid="hr_recruitment.menu_hr_recruitment_root"]',
    content: Markup(_t("Let's have a look at how to <b>improve</b> your <b>hiring process</b>.")),
    position: 'right',
    edition: 'community'
}, {
    trigger: '.o_app[data-menu-xmlid="hr_recruitment.menu_hr_recruitment_root"]',
    content: Markup(_t("Let's have a look at how to <b>improve</b> your <b>hiring process</b>.")),
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
    content: Markup(_t("<b>Did you apply by sending an email?</b> Check incoming applications.")),
    position: "bottom"
}, {
    trigger: ".oe_kanban_card",
    extra_trigger: '.o_kanban_applicant',
    content: Markup(_t("<b>Drag this card</b>, to qualify him for a first interview.")),
    position: "bottom",
    run: "drag_and_drop .o_kanban_group:eq(1) ",
}, {
    trigger: ".oe_kanban_card",
    extra_trigger: '.o_kanban_applicant',
    content: Markup(_t("<b>Click to view</b> the application.")),
    position: "bottom",
    width: 195
}, {
    trigger: ".o_Chatter .o_ChatterTopbar_buttonSendMessage",
    extra_trigger: '.o_applicant_form',
    content: Markup(_t("<div><b>Try to send an email</b> to the applicant.</div><div><i>Tips: All emails sent or received are saved in the history here</i>")),
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
    position: "bottom",
    width: 225
}, {
    trigger: ".o_form_button_save",
    extra_trigger: ".o_employee_form",
    content: _t("Save it !"),
    position: "bottom",
    width: 80
}]);

});

```

## File: static\src\views\form_view.js

```javascript
/** @odoo-module */

import { registry } from '@web/core/registry';

import { formView } from '@web/views/form/form_view';
import { FormController } from '@web/views/form/form_controller';
import { session } from '@web/session';

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
            interviewer => interviewer.data.id === session.uid) > -1;
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
            <kanban class="oe_background_grey o_kanban_dashboard o_hr_recruitment_kanban" on_create="hr_recruitment.create_job_simple" sample="1" limit="40" action="%(action_hr_job_applications)d" type="action">
                <field name="active"/>
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
                <field name="user_id"/>
                <field name="application_count"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''}">
                            <div class="o_kanban_card_header">
                                <div class="o_kanban_card_header_title">
                                    <field name="is_favorite" widget="boolean_favorite" nolabel="1"/>
                                    <div class="o_primary ps-3">
                                        <span class="o_text_overflow"><t t-esc="record.name.value"/></span>
                                    </div>
                                    <div class="ps-3 text-muted">
                                        <field name="user_id" />
                                    </div>
                                    <div class="ps-3 o_secondary" groups="base.group_multi_company">
                                        <small><i class="fa fa-building-o" role="img" aria-label="Company" title="Company"></i> <field name="company_id"/></small>
                                    </div>
                                    <div t-if="record.alias_name.value and record.alias_domain.value" class="ps-3 o_secondary o_job_alias">
                                        <small><i class="fa fa-envelope-o" role="img" aria-label="Alias" title="Alias"></i> <field name="alias_id"/></small>
                                    </div>
                                </div>
                                <div class="o_kanban_manage_button_section" groups="hr_recruitment.group_hr_recruitment_user">
                                    <a class="o_kanban_manage_toggle_button" href="#"><i class="fa fa-ellipsis-v" role="img" aria-label="Manage" title="Manage"/></a>
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
                                            <a name="edit_job" type="edit" t-attf-class="{{ record.no_of_recruitment.raw_value > 0 ? 'to-recruit' : 'text-secondary' }}" groups="!hr_recruitment.group_hr_recruitment_interviewer">
                                                <field name="no_of_recruitment"/> To Recruit
                                            </a>
                                            <span t-attf-class="{{ record.no_of_recruitment.raw_value > 0 ? 'to-recruit' : 'text-secondary' }}" groups="hr_recruitment.group_hr_recruitment_interviewer">
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
                            <div class="container o_recruitment_job_container o_kanban_card_manage_pane dropdown-menu" role="menu">
                                <div class="row">
                                    <div class="col-6 o_kanban_card_manage_section">
                                        <div role="menuitem" class="o_kanban_card_manage_title">
                                            <span>View</span>
                                        </div>
                                        <div role="menuitem" name="menu_view_applications">
                                            <a name="%(action_hr_job_applications)d" type="action">Applications</a>
                                        </div>
                                        <div role="menuitem">
                                            <a name="action_open_activities" type="object">Activities</a>
                                        </div>
                                        <div role="menuitem">
                                            <a name="%(action_hr_job_sources)d" type="action" context="{'default_job_id': active_id}">Trackers</a>
                                        </div>
                                    </div>
                                    <div class="col-6 o_kanban_card_manage_section">
                                        <div role="menuitem" class="o_kanban_card_manage_title">
                                            <span>New</span>
                                        </div>
                                        <div role="menuitem" name="menu_new_applications">
                                            <a name="%(hr_recruitment.action_hr_applicant_new)d" type="action">Application</a>
                                        </div>
                                    </div>
                                    <div class="col-6 o_kanban_card_manage_section">
                                        <div role="menuitem" class="o_kanban_card_manage_title">
                                            <span>Reporting</span>
                                        </div>
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
                <field name="alias_name" invisible="1"/>
                <field name="alias_id" attrs="{'invisible': [('alias_name', '=', False)]}" optional="hide"/>
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
                <field name="date_last_stage_update" invisible="1"/>
                <field name="partner_name" readonly="1" optional="show"/>
                <field name="type_id" invisible="1"/>
                <field name="job_id" optional="show"/>
                <field name="name" readonly="1" optional="show"/>
                <field name="stage_id" optional="show"/>
                <field name="application_status" optional="show" attrs="{'invisible': [('application_status', '=', 'ongoing')]}"/>
                <field name="refuse_reason_id" optional='hide'/>
                <field name="priority" widget="priority" optional="show"/>
                <field name="email_from" readonly="1" optional="hide"/>
                <field name="partner_mobile" widget="phone" readonly="1" optional="show" class="text-end"/>
                <field name="categ_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="show"/>
                <field name="user_id" widget="many2one_avatar_user" optional="show"/>
                <field name="interviewer_ids" widget="many2many_avatar_user" optional="hide"/>
                <field name="create_date" readonly="1" widget="date" optional="show"/>
                <field name="partner_phone" widget="phone" readonly="1" optional="hide"/>
                <field name="medium_id" optional="hide"/>
                <field name="source_id" readonly="1" optional="hide"/>
                <field name="salary_expected" optional="hide"/>
                <field name="salary_proposed" optional="hide"/>
                <field name="availability" optional="hide"/>
                <field name="department_id" invisible="context.get('invisible_department', True)" readonly="1"/>
                <field name="company_id" invisible="1"/>
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
            <field name="emp_id" invisible="1"/>
            <field name="meeting_ids" invisible="1"/>
            <field name="refuse_reason_id" invisible="1"/>
            <header>
                <button string="Create Employee" name="create_employee_from_applicant" type="object" data-hotkey="v" groups="hr_recruitment.group_hr_recruitment_user"
                        class="o_create_employee" attrs="{'invisible': ['|', '|', ('emp_id', '!=', False), ('active', '=', False), ('date_closed', '=', False)]}"/>
                <button string="Refuse" name="archive_applicant" type="object" attrs="{'invisible': [('active', '=', False)]}" data-hotkey="x"/>
                <button string="Restore" name="toggle_active" type="object" attrs="{'invisible': [('active', '=', True)]}" data-hotkey="z"/>
                <field name="stage_id" widget="statusbar" options="{'clickable': '1', 'fold_field': 'fold'}" attrs="{'invisible': [('active', '=', False),('emp_id', '=', False)]}"/>
            </header>
            <sheet>
                <div class="oe_button_box" name="button_box">
                    <button name="action_open_employee"
                            type="object"
                            class="oe_stat_button"
                            icon="fa-id-card-o"
                            groups="hr.group_hr_user"
                            attrs="{'invisible': [('emp_id', '=', False)]}">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_text"><field name="employee_name" readonly="1"/></span>
                            <span class="o_stat_value">Employee</span>
                        </div>
                    </button>
                    <button name="action_applications_email"
                            class="oe_stat_button"
                            icon="fa-pencil"
                            type="object"
                            context="{'active_test': False}"
                            attrs="{'invisible': [('application_count', '=' , 0)]}">
                        <field name="application_count" widget="statinfo" string="Other applications"/>
                    </button>
                    <button name="action_makeMeeting" class="oe_stat_button" icon="fa-calendar" type="object" attrs="{'invisible': [('id', '=', False)]}">
                         <div class="o_field_widget o_stat_info">
                             <span class="o_stat_value"><field name="meeting_display_text" /></span>
                             <span class="o_stat_text"><field name="meeting_display_date" readonly="1"/></span>
                         </div>
                    </button>
                </div>
                <widget name="web_ribbon" title="Refused" bg_color="bg-danger" attrs="{'invisible': ['|', ('active', '=', True), ('refuse_reason_id', '=', False)]}"/>
                <widget name="web_ribbon" title="Hired" attrs="{'invisible': [('date_closed', '=', False)]}" />
                <field name="active" invisible="1"/>
                <field name="legend_normal" invisible="1"/>
                <field name="legend_blocked" invisible="1"/>
                <field name="legend_done" invisible="1"/>
                <div class="oe_title pe-0">
                    <label for="name" class="oe_edit_only"/>
                    <h1 class="d-flex justify-content-between align-items-center">
                        <field name="name" placeholder="e.g. Sales Manager 2 year experience"/>
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
                        <field name="refuse_reason_id" attrs="{'invisible': [('active', '=', True)]}"/>
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
                        <field name="date_closed" attrs="{'invisible': [('date_closed', '=', False)]}" />
                        <field name="priority" widget="priority"/>
                        <field name="source_id"/>
                        <field name="medium_id"/>
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
                    </group>
                </group>
                <notebook>
                    <page string="Application Summary">
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
                <field name="attachment_ids" filter_domain="[('attachment_ids.index_content', 'ilike', self)]" string="Attachments"/>
                <filter string="My Applications" name="my_applications" domain="[('user_id', '=', uid)]"/>
                <filter string="Unassigned" name="unassigned" domain="[('user_id', '=', False)]"/>
                <separator/>
                <filter string="Ready for Next Stage" name="done" domain="[('kanban_state', '=', 'done')]"/>
                <filter string="Blocked" name="blocked" domain="[('kanban_state', '=', 'blocked')]"/>
                <filter string="Reserve" name="reserve" domain="[('job_id', '=', False)]"/>
                <separator/>
                <filter string="Creation Date" name="filter_create" date="create_date"/>
                <filter string="Last Stage Update" name="filter_date_last_stage_update" date="date_last_stage_update"/>
                <separator/>
                <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction', '=', True)]"/>
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
                    <filter string="Last Stage Update" name="last_stage_update" context="{'group_by': 'date_last_stage_update'}"/>
                    <filter string="Refuse Reason" name="refuse_reason_id" domain="[]" context="{'group_by': 'refuse_reason_id'}"/>
                    <filter string="Degree" name="degree" domain="[]" context="{'group_by': 'type_id'}"/>
                    <filter string="Tags" name="group_by_categ_ids" context="{'group_by':'categ_ids'}"/>
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
                <progressbar field="activity_state" colors='{"planned": "success", "overdue": "danger", "today": "warning"}'/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="{{!selection_mode ? 'oe_kanban_color_' + kanban_getcolor(record.color.raw_value) : ''}} oe_kanban_card oe_kanban_global_click oe_applicant_kanban oe_semantic_html_override">
                            <field name="date_closed" invisible="1"/>
                            <div class="ribbon ribbon-top-right pe-none" attrs="{'invisible': [('date_closed', '=', False)]}">
                                <span class="bg-success">Hired</span>
                            </div>
                            <span class="badge rounded-pill text-bg-danger float-end me-4" attrs="{'invisible': ['|', ('active', '=', True), ('refuse_reason_id', '=', False)]}">Refused</span>
                            <div class="o_dropdown_kanban dropdown">
                                <a class="dropdown-toggle o-no-caret btn" role="button" data-bs-toggle="dropdown" href="#" aria-label="Dropdown menu" title="Dropdown menu" data-bs-display="static">
                                    <span class="fa fa-ellipsis-v"/>
                                </a>
                                <div class="dropdown-menu" role="menu">
                                    <a role="menuitem" name="action_makeMeeting" type="object" class="dropdown-item">Schedule Interview</a>
                                    <a role="menuitem" name="archive_applicant" type="object" class="dropdown-item">Refuse</a>
                                    <a role="menuitem" name="archive_applicant" type="object" class="dropdown-item">Archive</a>
                                    <t t-if="widget.deletable"><a role="menuitem" type="delete" class="dropdown-item">Delete</a></t>
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
                                        <div class="o_kanban_state_with_padding ms-1 me-2" >
                                            <field name="kanban_state" widget="state_selection"/>
                                            <field name="legend_normal" invisible="1"/>
                                            <field name="legend_blocked" invisible="1"/>
                                            <field name="legend_done" invisible="1"/>
                                        </div>
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

    <record model="ir.actions.act_window" id="action_hr_applicant_new">
        <field name="res_model">hr.applicant</field>
        <field name="view_mode">form</field>
        <field name="context">{'form_view_initial_mode': 'edit', 'default_job_id': active_id}</field>
    </record>

    <!-- Jobs -->
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
                    <label for="alias_name" string="Application email" attrs="{'invisible': [('alias_domain', '=', False)]}" help="Define a specific contact address for this job position. If you keep it empty, the default email address will be used which is in human resources settings"/>
                    <div name="alias_def" attrs="{'invisible': [('alias_domain', '=', False)]}">
                        <field name="alias_id" class="oe_read_only" string="Email Alias" required="0"/>
                        <div class="oe_edit_only" name="edit_alias" dir="ltr">
                            <field name="alias_name" class="oe_inline o_job_alias" placeholder="e.g. sales-manager"/>@<field name="alias_domain" class="oe_inline" readonly="1"/>
                        </div>
                        <div class="text-muted" attrs="{'invisible': [('alias_domain', '=', False)]}">Applicants can send resume to this email address,<br/>it will create an application automatically</div>
                    </div>
                </group>
                <footer>
                    <button string="Create" type="object" name="close_dialog" class="btn-primary o_create_job" data-hotkey="q"/>
                    <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="z"/>
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
                <field name="interviewer_ids" widget="many2many_tags" options="{'no_create': True, 'no_create_edit': True}" />
            </div>
            <xpath expr="//field[@name='department_id']" position="after">
                <label for="address_id"/>
                <div class="o_row">
                    <span attrs="{'invisible': [('address_id', '!=', False)]}" class="oe_read_only">Remote</span>
                    <field name="address_id" context="{'show_address': 1}" options="{'always_reload': True}" placeholder="Remote"/>
                </div>
                <label for="alias_name" string="Email Alias" attrs="{'invisible': [('alias_domain', '=', False)]}" help="Define a specific contact address for this job position. If you keep it empty, the default email address will be used which is in human resources settings"/>
                <div name="alias_def" attrs="{'invisible': [('alias_domain', '=', False)]}">
                    <field name="alias_id" class="oe_read_only" string="Email Alias" required="0"/>
                    <div class="oe_edit_only" name="edit_alias" dir="ltr">
                        <field name="alias_name" class="oe_inline"/>@<field name="alias_domain" class="oe_inline" readonly="1"/>
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
                    attrs="{'invisible': [('documents_count','=',0)]}">
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

    ######################## JOB OPPORTUNITIES (menu) ###########################
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
        web_icon="hr_recruitment,static/description/icon.svg"
        groups="hr_recruitment.group_hr_recruitment_user,hr_recruitment.group_hr_recruitment_interviewer"
        sequence="210"/>

    <menuitem id="menu_hr_recruitment_configuration" name="Configuration" parent="menu_hr_recruitment_root"
        groups="group_hr_recruitment_user" sequence="100"/>

    <!-- ALL JOBS REQUESTS -->
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
                <field name="name" invisible="1"/>
                <field name="res_id" invisible="1"/>
                <field name="res_model" invisible="1"/>
                <field name="datas" widget="binary" readonly="1" filename="name" string="File"/>
                <field name="res_name" widget="applicant_char" string="Applicant" attrs="{'invisible': [('res_model', '!=', 'hr.applicant')]}"/>
                <field name="create_date"/>
            </tree>
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
                <field name="hired_stage"/>
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
                        <field name="hired_stage"/>
                        <field name="is_warning_visible" invisible="1"/>
                        <span attrs="{'invisible': [('is_warning_visible', '=', False)]}">
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
        parent="menu_hr_recruitment_config_applications"
        action="hr_applicant_category_action"
        sequence="20" groups="base.group_no_one"/>

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
            parent="menu_hr_recruitment_config_applications"
            action="hr_recruitment_degree_action"
            sequence="1" groups="base.group_no_one"/>

    <!-- Source Kanban View -->
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
                                        <field name="email" attrs="{'invisible': [('has_domain', '=', False)]}" widget="email"/>
                                        <button name="create_alias" class="btn btn-primary mb-2" type="object" attrs="{'invisible': ['|', ('has_domain', '=', False), ('email', '!=', False)]}">Generate Email</button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <!-- Source Tree View -->
    <record model="ir.ui.view" id="hr_recruitment_source_tree">
        <field name="name">hr.recruitment.source.tree</field>
        <field name="model">hr.recruitment.source</field>
        <field name="arch" type="xml">
            <tree string="Sources of Applicants" editable="top" class="o_recruitment_list" sample="1">
                <field name="has_domain" invisible="1"/>
                <field name="source_id" placeholder="e.g. LinkedIn" decoration-bf="1" attrs="{'readonly': [('id', '!=', False)]}"/>
                <field name="medium_id" optional="hidden"/>
                <field name="job_id" attrs="{'readonly': [('id', '!=', False)]}"/>
                <field name="email" attrs="{'invisible': [('email', '=', False)]}" widget="email"/>
                <button name="create_alias" string="Generate Email" class="btn btn-primary" type="object" attrs="{'invisible': ['|', ('has_domain', '=', False), ('email', '!=', False)]}"/>
            </tree>
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
    <menuitem name="Reporting" id="report_hr_recruitment" parent="menu_hr_recruitment_root" groups="group_hr_recruitment_user"
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
                                    <label for="module_hr_recruitment_survey" string="Send Interview Survey"/>
                                    <div class="text-muted">
                                        Send an Interview Survey to the applicant during
                                        the recruitment process
                                    </div>
                                    <div class="content-group" id="interview_forms"/>
                                </div>
                            </div>
                            <div class="col-xs-12 col-md-6 o_setting_box" id="sms">
                                <div class="o_setting_right_pane" id="sms_settings">
                                    <div class="o_form_label">
                                        Send SMS
                                        <a href="https://www.odoo.com/documentation/16.0/applications/marketing/sms_marketing/pricing/pricing_and_faq.html" title="Documentation" class="ms-1 o_doc_link" target="_blank"></a>
                                    </div>
                                    <div class="text-muted">
                                        Send texts to your contacts
                                    </div>
                                </div>
                            </div>
                            <div class="col-xs-12 col-md-6 o_setting_box" id="display_cv">
                                <div class="o_setting_left_pane">
                                    <field name="group_applicant_cv_display"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="group_applicant_cv_display" string="CV Display"/>
                                    <div class="text-muted">
                                        Display CV on application form
                                    </div>
                                </div>
                            </div>
                            <div class="col-xs-12 col-md-6 o_setting_box" title="Use OCR to fill data from a picture of the CV or the file itself">
                                <div class="o_setting_left_pane">
                                    <field name="module_hr_recruitment_extract" widget="upgrade_boolean"/>
                                </div>
                                <div class="o_setting_right_pane" id="recruitment_extract_settings">
                                    <label string="CV Digitization (OCR)" for="module_hr_recruitment_extract"/>
                                    <span class="fa fa-lg fa-building-o" title="Values set here are company-specific."/>
                                    <div class="text-muted">
                                        Digitize your CV to extract name and email automatically.
                                    </div>
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

    def action_refuse_reason_apply(self):
        if self.send_mail:
            if not self.template_id:
                raise UserError(_("Email template must be selected to send a mail"))
            if not self.applicant_ids.filtered(lambda x: x.email_from or x.partner_id.email):
                raise UserError(_("Email of the applicant is not set, email won't be sent."))
        self.applicant_ids.write({'refuse_reason_id': self.refuse_reason_id.id, 'active': False})
        if self.send_mail:
            applicants = self.applicant_ids.filtered(lambda x: x.email_from or x.partner_id.email)
            applicants.with_context(active_test=True).message_post_with_template(self.template_id.id, **{
                'auto_delete_message': True,
                'subtype_id': self.env['ir.model.data']._xmlid_to_res_id('mail.mt_note'),
                'email_layout_xmlid': 'mail.mail_notification_light'
            })

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
                        <field name="send_mail" attrs="{'invisible': [('refuse_reason_id', '=', False)]}"/>
                        <field name="template_id" attrs="{'invisible': [('send_mail', '=', False)], 'required': [('send_mail', '=', True)]}" />
                        <field name="applicant_ids" invisible="1"/>
                    </group>
                    <div class="alert alert-danger" role="alert" attrs="{'invisible': [('applicant_without_email', '=', False)]}">
                        <field name="applicant_without_email" class="mr4"/>
                    </div>
                    <footer>
                        <button name="action_refuse_reason_apply" string="Refuse" type="object" class="btn-primary" data-hotkey="q"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z"/>
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
                    'type': 'private',
                    'name': applicant.partner_name,
                    'email': applicant.email_from,
                    'phone': applicant.partner_phone,
                    'mobile': applicant.partner_mobile,
                })

            applicant.message_post(
                subject=subjects[applicant.id],
                body=bodies[applicant.id],
                message_type='comment',
                email_from=self.author_id.email,
                email_layout_xmlid='mail.mail_notification_light',
                partner_ids=applicant.partner_id.ids,
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
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z"/>
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

