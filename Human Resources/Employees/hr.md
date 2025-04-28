# Odoo Module: hr

Category: Human Resources/Employees

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard
from . import populate


def _install_hr_localization(env):
    if any(c.partner_id.country_id.code == 'MX' for c in env['res.company'].search([])):
        l10n_mx = env['ir.module.module'].sudo().search([
            ('name', '=', 'l10n_mx_hr'),
            ('state', 'not in', ['installed', 'to install', 'to upgrade']),
        ])
        if l10n_mx:
            l10n_mx.button_install()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Employees',
    'version': '1.1',
    'category': 'Human Resources/Employees',
    'sequence': 95,
    'summary': 'Centralize employee information',
    'website': 'https://www.odoo.com/app/employees',
    'images': [
        'static/src/img/default_image.png',
    ],
    'depends': [
        'base_setup',
        'phone_validation',
        'mail',
        'resource',
        'web',
    ],
    'data': [
        'security/hr_security.xml',
        'security/ir.model.access.csv',
        'wizard/hr_departure_wizard_views.xml',
        'wizard/mail_activity_schedule_views.xml',
        'views/mail_activity_plan_views.xml',
        'views/hr_departure_reason_views.xml',
        'views/hr_contract_type_views.xml',
        'views/hr_job_views.xml',
        'views/hr_employee_category_views.xml',
        'views/hr_employee_public_views.xml',
        'report/hr_employee_badge.xml',
        'views/hr_employee_views.xml',
        'views/hr_department_views.xml',
        'views/hr_work_location_views.xml',
        'views/hr_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
        'views/discuss_channel_views.xml',
        'views/res_users.xml',
        'views/hr_templates.xml',
        'data/hr_data.xml',
    ],
    'demo': [
        'data/hr_demo.xml'
    ],
    'installable': True,
    'application': True,
    'post_init_hook': '_install_hr_localization',
    'assets': {
        'web.assets_backend': [
            'hr/static/src/**/*',
        ],
        'web.qunit_suite_tests': [
            'hr/static/tests/**/*',
            ('remove', 'hr/static/tests/tours/**/*'),
        ],
        'web.assets_tests': [
            'hr/static/tests/tours/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\hr_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="dep_administration" model="hr.department">
          <field name="name">Administration</field>
        </record>

        <record id="employee_admin" model="hr.employee">
            <field name="name" eval="obj(ref('base.partner_admin')).name" model="res.partner"/>
            <field name="department_id" ref="dep_administration"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="address_id" ref="base.main_partner"/>
            <field name="private_email">admin@example.com</field>
            <field name="image_1920" eval="obj(ref('base.partner_admin')).image_1920" model="res.partner"/>
        </record>

        <record id="onboarding_plan" model="mail.activity.plan">
            <field name="name">Onboarding</field>
            <field name="res_model">hr.employee</field>
        </record>

        <record id="onboarding_setup_it_materials" model="mail.activity.plan.template">
            <field name="sequence">10</field>
            <field name="summary">Setup IT Materials</field>
            <field name="responsible_type">manager</field>
            <field name="plan_id" ref="onboarding_plan"/>
        </record>

        <record id="onboarding_plan_training" model="mail.activity.plan.template">
            <field name="sequence">20</field>
            <field name="summary">Plan Training</field>
            <field name="responsible_type">manager</field>
            <field name="plan_id" ref="onboarding_plan"/>
        </record>

        <record id="onboarding_training" model="mail.activity.plan.template">
            <field name="sequence">30</field>
            <field name="summary">Training</field>
            <field name="responsible_type">employee</field>
            <field name="plan_id" ref="onboarding_plan"/>
        </record>

        <record id="offboarding_plan" model="mail.activity.plan">
            <field name="name">Offboarding</field>
            <field name="res_model">hr.employee</field>
        </record>

        <record id="offboarding_setup_compute_out_delais" model="mail.activity.plan.template">
            <field name="sequence">10</field>
            <field name="summary">Organize knowledge transfer inside the team</field>
            <field name="responsible_type">manager</field>
            <field name="plan_id" ref="offboarding_plan"/>
        </record>

        <record id="offboarding_take_back_hr_materials" model="mail.activity.plan.template">
            <field name="sequence">20</field>
            <field name="summary">Take Back HR Materials</field>
            <field name="responsible_type">manager</field>
            <field name="plan_id" ref="offboarding_plan"/>
        </record>

        <record model="ir.config_parameter" id="hr_presence_control_login" forcecreate="False">
            <field name="key">hr.hr_presence_control_login</field>
            <field name="value">True</field>
        </record>

        <!-- Departure Reasons -->
        <record id="departure_fired" model="hr.departure.reason">
            <field name="sequence">0</field>
            <field name="name">Fired</field>
            <field name="reason_code">342</field>
        </record>

        <record id="departure_resigned" model="hr.departure.reason">
            <field name="sequence">1</field>
            <field name="name">Resigned</field>
            <field name="reason_code">343</field>
        </record>

        <record id="departure_retired" model="hr.departure.reason">
            <field name="sequence">2</field>
            <field name="name">Retired</field>
            <field name="reason_code">340</field>
        </record>

        <record id="contract_type_permanent" model="hr.contract.type">
            <field name="name">Permanent</field>
            <field name="sequence">1</field>
        </record>

        <record id="contract_type_temporary" model="hr.contract.type">
            <field name="name">Temporary</field>
            <field name="sequence">2</field>
        </record>

        <record id="contract_type_seasonal" model="hr.contract.type">
            <field name="name">Seasonal</field>
            <field name="sequence">3</field>
        </record>

        <record id="contract_type_full_time" model="hr.contract.type">
            <field name="name">Full-Time</field>
            <field name="sequence">4</field>
        </record>

        <record id="contract_type_part_time" model="hr.contract.type">
            <field name="name">Part-Time</field>
            <field name="sequence">5</field>
        </record>

        <!-- Work permit expires Soon -->
        <record id="ir_cron_data_check_work_permit_validity" model="ir.cron">
            <field name="name">HR Employee: check work permit validity</field>
            <field name="model_id" ref="model_hr_employee"/>
            <field name="state">code</field>
            <field name="code">model._cron_check_work_permit_validity()</field>
            <field name="interval_number">1</field>
            <field name="interval_type">days</field>
            <field name="numbercall">-1</field>
        </record>

        <record id="home_work_location" model="hr.work.location">
            <field name="name">Home</field>
            <field name="location_type">home</field>
            <field name="address_id" ref="base.main_partner"/>
        </record>

        <record id="home_work_office" model="hr.work.location">
            <field name="name">Office</field>
            <field name="location_type">office</field>
            <field name="address_id" ref="base.main_partner"/>
        </record>

        <record id="home_work_other" model="hr.work.location">
            <field name="name">Other</field>
            <field name="location_type">other</field>
            <field name="address_id" ref="base.main_partner"/>
        </record>
    </data>
</odoo>

```

## File: data\hr_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="base.user_demo" model="res.users">
            <field name="groups_id" eval="[(3, ref('hr.group_hr_manager'))]"/>
        </record>

        <!--Department-->
        <record id="dep_management" model="hr.department">
            <field name="name">Management</field>
            <field name="color" eval="5"/>
        </record>

        <record id="dep_administration" model="hr.department">
            <field name="parent_id" ref="dep_management"/>
            <field name="manager_id" ref="employee_admin"/>
            <field name="color" eval="8"/>
        </record>

        <record id="dep_sales" model="hr.department">
            <field name="name">Sales</field>
            <field name="parent_id" ref="dep_management"/>
            <field name="color" eval="9"/>
        </record>

        <record id="dep_rd" model="hr.department">
            <field name="name">Research &amp; Development</field>
            <field name="parent_id" ref="dep_management"/>
            <field name="color" eval="10"/>
        </record>

        <record id="dep_rd_be" model="hr.department">
            <field name="name">R&amp;D USA</field>
            <field name="parent_id" ref="dep_rd"/>
            <field name="color" eval="10"/>
        </record>

        <record id="dep_rd_ltp" model="hr.department">
            <field name="name">Long Term Projects</field>
            <field name="parent_id" ref="dep_rd_be"/>
            <field name="color" eval="10"/>
        </record>

        <record id="dep_ps" model="hr.department">
            <field name="name">Professional Services</field>
            <field name="parent_id" ref="dep_management"/>
            <field name="color" eval="3"/>
        </record>

        <!-- Contract Types -->
        <record id="contract_type_permanent" model="hr.contract.type">
            <field name="name">Permanent</field>
            <field name="sequence">1</field>
        </record>

        <record id="contract_type_temporary" model="hr.contract.type">
            <field name="name">Temporary</field>
            <field name="sequence">2</field>
        </record>

        <record id="contract_type_interim" model="hr.contract.type">
            <field name="name">Interim</field>
            <field name="sequence">3</field>
        </record>

        <!--Jobs-->
        <record id="job_ceo" model="hr.job">
            <field name="name">Chief Executive Officer</field>
            <field name="department_id" ref="dep_management"/>
            <field name="description">Demonstration of different Odoo services for each client and convincing the client about functionality of the application.
The candidate should have excellent communication skills.
Relationship building and influencing skills
Expertise in New Client Acquisition (NCAs) and Relationship Management.
Gathering market and customer information.
Coordinating with the sales and support team for adopting different strategies
Reviewing progress and identifying opportunities and new areas for development.
Building strong relationships with clients / customers for business growth profitability.
Keep regular interaction with key clients for better extraction and expansion.</field>
            <field name="requirements">MBA in Marketing is must.
Good Communication skills.
Only Fresher's can apply.
Candidate should be ready to work in young and dynamic environment..
Candidate should be able to work in “start- up” fast paced environment,hands on attitude.
Honest,approachable and fun team player.
Result driven.
Excellent analytical skills, ability to think logically and "out of the box"</field>
            <field name="contract_type_id" ref="contract_type_permanent"/>
        </record>

        <record id="job_cto" model="hr.job">
            <field name="name">Chief Technical Officer</field>
            <field name="department_id" ref="dep_rd"/>
            <field name="description">You will take part in the consulting services we provide to our partners and customers: design, analysis, development, testing, project management, support/coaching. You will work autonomously as well as coordinate and supervise small distributed development teams for some projects. Optionally, you will deliver Odoo training sessions to partners and customers (8-10 people/session). You will report to the Head of Professional Services and work closely with all developers and consultants.

The job is located in Grand-Rosière (1367), Belgium (between Louvain-La-Neuve and Namur).</field>
            <field name="requirements">Bachelor, master or engineering degree in Computer Science or equivalent by experience
Preferably at least 1 years of experience
Interest for enterprise application development
Customer-minded
Willing to travel abroad occasionally for short term missions.
Passion for the Internet and its culture
Quick and autonomous learner, problem-solving personality, enthusiastic when faced with technical challenges
Team spirit and good communication
Required skills:
Good knowledge of object oriented programming, object modeling, relational databases, Unix/Linux platform
Fluent in English, especially read and written
Nice-to-have skills:
Good knowledge of Python
Good knowledge of HTML and Javascript
Knowledge of UML-like modeling
Good language skills, other than English (Dutch and French preferred, others welcome)
          </field>
            <field name="contract_type_id" ref="contract_type_permanent"/>
        </record>

        <record id="job_consultant" model="hr.job">
            <field name="name">Consultant</field>
            <field name="department_id" ref="dep_ps"/>
            <field name="no_of_recruitment">5</field>
            <field name="contract_type_id" ref="contract_type_interim"/>
            <field name="description">We are currently looking for someone like that to join our Consultant team.</field>
        </record>

        <record id="job_developer" model="hr.job">
            <field name="name">Experienced Developer</field>
            <field name="department_id" ref="dep_rd"/>
            <field name="no_of_recruitment">5</field>
            <field name="contract_type_id" ref="contract_type_permanent"/>
            <field name="description">We are currently looking for someone like that to join our Web team.
                Someone who can snap out of coding and perform analysis or meet clients to explain the technical possibilities that can meet their needs.</field>
        </record>

        <record id="job_hrm" model="hr.job">
            <field name="name">Human Resources Manager</field>
            <field name="department_id" ref="dep_administration"/>
            <field name="description">Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.</field>
            <field name="requirements">Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.</field>
            <field name="contract_type_id" ref="contract_type_permanent"/>
        </record>

        <record id="job_marketing" model="hr.job">
            <field name="name">Marketing and Community Manager</field>
            <field name="department_id" ref="dep_sales"/>
                      <field name="description">The Marketing Manager defines the mid- to long-term marketing strategy for his covered market segments in the World.
              He develops and monitors the annual budget in collaboration with Sales.
              He defines the products and customers portfolio according to the marketing plan.
              This mission requires strong collaboration with Technical Service and Sales.</field>
        </record>

        <record id="job_trainee" model="hr.job">
            <field name="name">Trainee</field>
            <field name="description">You participate to the update of our tutorial tools and pre-sales tools after the launch of a new version of Odoo. Indeed, any new version of the software brings significant improvements in terms of functionalities, ergonomics and configuration.
You will have to become familiar with the existing tools (books, class supports, Odoo presentation’s slides, commercial tools),
to participate to the update of those tools in order to make them appropriate for the new version of the software and, for sure,
to suggest improvements in order to cover the new domains of the software.
You join the Implementation Assistance department. This team of 3 people go with Odoo’s clients in the set up of the software. Your role will be
to animate webinars in order to show the different functionalities of the software.
to be involved in the support of the customers and
to answer to their questions.
You help the support manager to set up new support services by
being involved in the treatment of new cases,
contributing to the set up of a new politic,
being involved into satisfaction surveys in order to have a better knowledge of how the support given is seen by the customers.</field>
            <field name="requirements">You speak fluently English and French (one other European language is a +)
At the time of your traineeship at Odoo, you will be in the last year of a Master or Bachelor Degree (ideally in the following sector: Business Management, IT Management, Computer Sciences)
You have a software and new technology awareness
You are ready to join a young and dynamic company, you are able to work in a “start up” fast paced environment, hands on attitude
You are approachable, honest and a fun team player
If you have development competencies, we can propose you specific traineeships</field>
        </record>

       <!-- Work Locations -->
      <record id="work_location_1" model="hr.work.location">
          <field name="name">Building 1, Second Floor</field>
          <field name="location_type">office</field>
          <field name="address_id" ref="base.main_partner"/>
      </record>

       <!-- Employee categories -->
        <record id="employee_category_2" model="hr.employee.category">
            <field name="name">Sales</field>
            <field name="color" eval="1"/>
        </record>

        <record id="employee_category_3" model="hr.employee.category">
            <field name="name">Trainer</field>
            <field name="color" eval="2"/>
        </record>

        <record id="employee_category_4" model="hr.employee.category">
            <field name="name">Employee</field>
            <field name="color" eval="6"/>
        </record>

        <record id="employee_category_5" model="hr.employee.category">
            <field name="name">Consultant</field>
            <field name="color" eval="4"/>
        </record>

    <!--Employees-->

        <record id="employee_admin" model="hr.employee">
            <field name="private_street">215 Vine St</field>
            <field name="private_city">Scranton</field>
            <field name="private_zip">18503</field>
            <field name="private_country_id" ref="base.us"/>
            <field name="private_state_id" ref="base.state_us_39"/>
            <field name="private_phone">+1 555-555-5555</field>
            <field name="private_email">admin@yourcompany.example.com</field>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-125-2389</field>
            <field name="work_email">admin@yourcompany.example.com</field>
            <field name="category_ids" eval="[Command.set([ref('employee_category_4'), ref('employee_category_3')])]"/>
            <field name="job_id" ref="hr.job_ceo"/>
            <field name="job_title">Chief Executive Officer</field>
            <field name="department_id" ref="dep_management"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_ngh" model="res.partner">
            <field name="name">Jeffrey Kelly</field>
            <field name="email">jeffrey.kelly72@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_ngh-image.jpg"/>
        </record>

        <record id="employee_ngh" model="hr.employee">
            <field name="name">Jeffrey Kelly</field>
            <field name="department_id" ref="dep_sales"/>
            <field name="parent_id" ref="employee_admin"/>
            <field name="job_id" ref="hr.job_marketing"/>
            <field name="job_title">Marketing and Community Manager</field>
            <field name="category_ids" eval="[Command.set([ref('employee_category_4'), ref('employee_category_2')])]"/>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-264-7362</field>
            <field name="work_contact_id" ref="hr.work_contact_ngh"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_ngh-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_qdp" model="res.partner">
            <field name="name">Marc Demo</field>
            <field name="email">m.demo@fake.odoo.com</field>
        </record>

        <record id="employee_qdp" model="hr.employee">
            <field name="name">Marc Demo</field>
            <field name="user_id" ref="base.user_demo"/>
            <field name="department_id" ref="dep_rd"/>
            <field name="parent_id" ref="employee_admin"/>
            <field name="private_street">361-7936 Feugiat St.</field>
            <field name="private_zip">58521</field>
            <field name="private_city">Williston</field>
            <field name="private_country_id" ref="base.us"/>
            <field name="private_phone">+1 555-555-5757</field>
            <field name="private_email">demo@yourcompany.example.com</field>
            <field name="job_id" ref="hr.job_developer"/>
            <field name="job_title">Experienced Developer</field>
            <field name="category_ids" eval="[Command.set([ref('employee_category_4')])]"/>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">+3281813700</field>
            <field name="work_contact_id" ref="hr.work_contact_qdp"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_al" model="res.partner">
            <field name="name">Ronnie Hart</field>
            <field name="email">ronnie.hart87@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_al-image.jpg"/>
        </record>

        <record id="employee_al" model="hr.employee">
            <field name="name">Ronnie Hart</field>
            <field name="department_id" ref="dep_rd"/>
            <field name="parent_id" ref="employee_qdp"/>
            <field name="job_id" ref="hr.job_cto"/>
            <field name="job_title">Team Leader</field>
            <field name="category_ids" eval="[Command.set([ref('employee_category_4'), ref('employee_category_3')])]"/>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-310-7863</field>
            <field name="work_contact_id" ref="hr.work_contact_al"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_al-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_vad" model="res.partner">
            <field name="name">Tina Williamson</field>
            <field name="email">tina.williamson98@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_vad-image.jpg"/>
        </record>

        <record id="employee_vad" model="hr.employee">
            <field name="name">Tina Williamson</field>
            <field name="department_id" ref="dep_administration"/>
            <field name="parent_id" ref="employee_admin"/>
            <field name="job_id" ref="hr.job_hrm"/>
            <field name="job_title">Human Resources Manager</field>
            <field name="category_ids" eval="[Command.set([ref('employee_category_4')])]"/>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-694-7266</field>
            <field name="work_contact_id" ref="hr.work_contact_vad"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_vad-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_hne" model="res.partner">
            <field name="name">Abigail Peterson</field>
            <field name="email">abigail.peterson39@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_hne-image.jpg"/>
        </record>

        <record id="employee_hne" model="hr.employee">
            <field name="name">Abigail Peterson</field>
            <field name="department_id" ref="dep_ps"/>
            <field name="parent_id" ref="employee_ngh"/>
            <field name="job_id" ref="hr.job_consultant"/>
            <field name="job_title">Consultant</field>
            <field name="private_country_id" ref="base.us"/>
            <field name="private_email">abigail.peterson33@example.com</field>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-233-3393</field>
            <field name="work_contact_id" ref="hr.work_contact_hne"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_hne-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
            <field name="marital">married</field>
        </record>

        <record id="work_contact_fpi" model="res.partner">
            <field name="name">Audrey Peterson</field>
            <field name="email">audrey.peterson25@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_fpi-image.jpg"/>
        </record>

        <record id="employee_fpi" model="hr.employee">
            <field name="name">Audrey Peterson</field>
            <field name="department_id" ref="dep_ps"/>
            <field name="parent_id" ref="employee_ngh"/>
            <field name="job_id" ref="hr.job_consultant"/>
            <field name="job_title">Consultant</field>
            <field name="category_ids" eval="[Command.set([ref('employee_category_4'), ref('employee_category_5')])]"/>
            <field name="private_country_id" ref="base.us"/>
            <field name="private_email">Audrey.peterson2020@example.com</field>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-276-7903</field>
            <field name="work_contact_id" ref="hr.work_contact_fpi"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_fpi-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_lur" model="res.partner">
            <field name="name">Eli Lambert</field>
            <field name="email">eli.lambert22@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_lur-image.jpg"/>
        </record>

        <record id="employee_lur" model="hr.employee">
            <field name="name">Eli Lambert</field>
            <field name="department_id" ref="dep_sales"/>
            <field name="parent_id" ref="employee_ngh"/>
            <field name="job_id" ref="hr.job_marketing"/>
            <field name="job_title">Marketing and Community Manager</field>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-169-1352</field>
            <field name="work_contact_id" ref="hr.work_contact_lur"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_lur-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_jod" model="res.partner">
            <field name="name">Rachel Perry</field>
            <field name="email">jod@odoo.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_jod-image.jpg"/>
        </record>

        <record id="employee_jod" model="hr.employee">
            <field name="name">Rachel Perry</field>
            <field name="department_id" ref="dep_sales"/>
            <field name="parent_id" ref="employee_ngh"/>
            <field name="job_id" ref="hr.job_marketing"/>
            <field name="job_title">Marketing and Community Manager</field>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-267-3735</field>
            <field name="work_contact_id" ref="hr.work_contact_jod"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_jod-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_fme" model="res.partner">
            <field name="name">Keith Byrd</field>
            <field name="email">keith.byrd52@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_fme-image.jpg"/>
        </record>

        <record id="employee_fme" model="hr.employee">
            <field name="name">Keith Byrd</field>
            <field name="department_id" ref="dep_rd"/>
            <field name="parent_id" ref="employee_al"/>
            <field name="job_id" ref="hr.job_developer"/>
            <field name="job_title">Experienced Developer</field>
            <field name="category_ids" eval="[Command.set([ref('employee_category_4')])]"/>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-505-5146</field>
            <field name="work_contact_id" ref="hr.work_contact_fme"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_fme-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_jep" model="res.partner">
            <field name="name">Doris Cole</field>
            <field name="email">doris.cole31@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_jep-image.jpg"/>
        </record>

        <record id="employee_jep" model="hr.employee">
            <field name="name">Doris Cole</field>
            <field name="department_id" ref="dep_ps"/>
            <field name="parent_id" ref="employee_vad"/>
            <field name="job_id" ref="hr.job_consultant"/>
            <field name="job_title">Consultant</field>
            <field name="private_country_id" ref="base.us"/>
            <field name="private_email">Doris.cole.LoveSong@example.com</field>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-331-5378</field>
            <field name="work_contact_id" ref="hr.work_contact_jep"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_jep-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_jgo" model="res.partner">
            <field name="name">Ernest Reed</field>
            <field name="email">ernest.reed47@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_jgo-image.jpg"/>
        </record>

        <record id="employee_jgo" model="hr.employee">
            <field name="name">Ernest Reed</field>
            <field name="department_id" ref="dep_ps"/>
            <field name="parent_id" ref="employee_vad"/>
            <field name="job_id" ref="hr.job_consultant"/>
            <field name="job_title">Consultant</field>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-518-8232</field>
            <field name="work_contact_id" ref="hr.work_contact_jgo"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_jgo-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_jth" model="res.partner">
            <field name="name">Toni Jimenez</field>
            <field name="email">toni.jimenez23@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_jth-image.jpg"/>
        </record>

        <record id="employee_jth" model="hr.employee">
            <field name="name">Toni Jimenez</field>
            <field name="department_id" ref="dep_ps"/>
            <field name="parent_id" ref="employee_vad"/>
            <field name="job_id" ref="hr.job_consultant"/>
            <field name="job_title">Consultant</field>
            <field name="category_ids" eval="[Command.set([ref('employee_category_4'), ref('employee_category_5')])]"/>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-707-8451</field>
            <field name="work_contact_id" ref="hr.work_contact_jth"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_jth-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_mit" model="res.partner">
            <field name="name">Anita Oliver</field>
            <field name="mobile">(555)-672-3185</field>
            <field name="email">anita.oliver32@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_mit-image.jpg"/>
        </record>

        <record id="employee_mit" model="hr.employee">
            <field name="name">Anita Oliver</field>
            <field name="department_id" ref="dep_rd_be"/>
            <field name="parent_id" ref="employee_fme"/>
            <field name="job_id" ref="hr.job_developer"/>
            <field name="job_title">Experienced Developer</field>
            <field name="category_ids" eval="[Command.set([ref('employee_category_4')])]"/>
            <field name="private_country_id" ref="base.us"/>
            <field name="private_phone">(538)-672-3185</field>
            <field name="private_email">anita.oliver00@example.com</field>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-497-4804</field>
            <field name="work_contact_id" ref="hr.work_contact_mit"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_mit-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_niv" model="res.partner">
            <field name="name">Sharlene Rhodes</field>
            <field name="email">sharlene.rhodes49@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_niv-image.jpg"/>
        </record>

        <record id="employee_niv" model="hr.employee">
            <field name="name">Sharlene Rhodes</field>
            <field name="department_id" ref="dep_management"/>
            <field name="parent_id" ref="employee_qdp"/>
            <field name="job_id" ref="hr.job_developer"/>
            <field name="job_title">Experienced Developer</field>
            <field name="category_ids" eval="[Command.set([ref('employee_category_4')])]"/>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-719-4182</field>
            <field name="work_contact_id" ref="hr.work_contact_niv"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_niv-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_stw" model="res.partner">
            <field name="name">Randall Lewis</field>
            <field name="email">randall.lewis74@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_stw-image.jpg"/>
        </record>

        <record id="employee_stw" model="hr.employee">
            <field name="name">Randall Lewis</field>
            <field name="department_id" ref="dep_rd"/>
            <field name="parent_id" ref="employee_qdp"/>
            <field name="job_id" ref="hr.job_developer"/>
            <field name="job_title">Experienced Developer</field>
            <field name="category_ids" eval="[Command.set([ref('employee_category_4')])]"/>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-775-6660</field>
            <field name="work_contact_id" ref="hr.work_contact_stw"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_stw-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_chs" model="res.partner">
            <field name="name">Jennie Fletcher</field>
            <field name="email">jennie.fletcher76@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_chs-image.jpg"/>
        </record>

        <record id="employee_chs" model="hr.employee">
            <field name="name">Jennie Fletcher</field>
            <field name="department_id" ref="dep_rd"/>
            <field name="parent_id" ref="employee_fme"/>
            <field name="job_id" ref="hr.job_developer"/>
            <field name="job_title">Experienced Developer</field>
            <field name="category_ids" eval="[Command.set([ref('employee_category_4')])]"/>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-363-8229</field>
            <field name="work_contact_id" ref="hr.work_contact_chs"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_chs-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_jve" model="res.partner">
            <field name="name">Paul Williams</field>
            <field name="email">paul.williams59@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_jve-image.jpg"/>
        </record>

        <record id="employee_jve" model="hr.employee">
            <field name="name">Paul Williams</field>
            <field name="department_id" ref="dep_rd_ltp"/>
            <field name="parent_id" ref="employee_qdp"/>
            <field name="job_id" ref="hr.job_developer"/>
            <field name="job_title">Experienced Developer</field>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-262-1607</field>
            <field name="work_contact_id" ref="hr.work_contact_jve"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_jve-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_han" model="res.partner">
            <field name="name">Walter Horton</field>
            <field name="email">walter.horton80@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_han-image.jpg"/>
        </record>

        <record id="employee_han" model="hr.employee">
            <field name="name">Walter Horton</field>
            <field name="department_id" ref="dep_rd"/>
            <field name="parent_id" ref="employee_jve"/>
            <field name="job_id" ref="hr.job_developer"/>
            <field name="job_title">Experienced Developer</field>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-912-1201</field>
            <field name="work_contact_id" ref="hr.work_contact_han"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_han-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <record id="work_contact_jog" model="res.partner">
            <field name="name">Beth Evans</field>
            <field name="email">beth.evans77@example.com</field>
            <field name="image_1920" type="base64" file="hr/static/img/employee_jog-image.jpg"/>
        </record>

        <record id="employee_jog" model="hr.employee">
            <field name="name">Beth Evans</field>
            <field name="department_id" ref="dep_rd"/>
            <field name="parent_id" ref="employee_jve"/>
            <field name="job_id" ref="hr.job_developer"/>
            <field name="job_title">Experienced Developer</field>
            <field name="private_country_id" ref="base.us"/>
            <field name="private_email">beth.evans@example.com</field>
            <field name="work_location_id" ref="work_location_1"/>
            <field name="work_phone">(555)-532-3841</field>
            <field name="work_contact_id" ref="hr.work_contact_jog"/>
            <field name="image_1920" type="base64" file="hr/static/img/employee_jog-image.jpg"/>
            <field name="create_date">2010-01-01 00:00:00</field>
        </record>

        <!-- Assign manager for each department -->
        <record id="dep_management" model="hr.department">
            <field name="manager_id" ref="employee_admin"/>
        </record>

        <record id="dep_sales" model="hr.department">
            <field name="manager_id" ref="employee_ngh"/>
        </record>

        <record id="dep_rd" model="hr.department">
            <field name="manager_id" ref="employee_qdp"/>
        </record>

        <record id="dep_rd_be" model="hr.department">
            <field name="manager_id" ref="employee_al"/>
        </record>

        <record id="dep_rd_ltp" model="hr.department">
            <field name="manager_id" ref="employee_jve"/>
        </record>

        <record id="dep_ps" model="hr.department">
            <field name="manager_id" ref="employee_vad"/>
        </record>
    </data>
</odoo>

```

## File: models\discuss_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError


class Channel(models.Model):
    _inherit = 'discuss.channel'

    subscription_department_ids = fields.Many2many(
        'hr.department', string='HR Departments',
        help='Automatically subscribe members of those departments to the channel.')

    @api.constrains('subscription_department_ids')
    def _constraint_subscription_department_ids_channel(self):
        failing_channels = self.sudo().filtered(lambda channel: channel.channel_type != 'channel' and channel.subscription_department_ids)
        if failing_channels:
            raise ValidationError(_("For %(channels)s, channel_type should be 'channel' to have the department auto-subscription.", channels=', '.join([ch.name for ch in failing_channels])))

    def _subscribe_users_automatically_get_members(self):
        """ Auto-subscribe members of a department to a channel """
        new_members = super(Channel, self)._subscribe_users_automatically_get_members()
        for channel in self:
            new_members[channel.id] = list(
                set(new_members[channel.id]) |
                set((channel.subscription_department_ids.member_ids.user_id.partner_id.filtered(lambda p: p.active) - channel.channel_partner_ids).ids)
            )
        return new_members

    def write(self, vals):
        res = super(Channel, self).write(vals)
        if vals.get('subscription_department_ids'):
            self._subscribe_users_automatically()
        return res

```

## File: models\hr_contract_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ContractType(models.Model):
    _name = 'hr.contract.type'
    _description = 'Contract Type'
    _order = 'sequence'

    name = fields.Char(required=True, translate=True)
    code = fields.Char(compute='_compute_code', store=True, readonly=False)
    sequence = fields.Integer()
    country_id = fields.Many2one('res.country')

    @api.depends('name')
    def _compute_code(self):
        for contract_type in self:
            if contract_type.code:
                continue
            contract_type.code = contract_type.name

```

## File: models\hr_department.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo.osv import expression


class Department(models.Model):
    _name = "hr.department"
    _description = "Department"
    _inherit = ['mail.thread']
    _order = "name"
    _rec_name = 'complete_name'
    _parent_store = True

    name = fields.Char('Department Name', required=True, translate=True)
    complete_name = fields.Char('Complete Name', compute='_compute_complete_name', recursive=True, store=True)
    active = fields.Boolean('Active', default=True)
    company_id = fields.Many2one('res.company', string='Company', index=True, default=lambda self: self.env.company)
    parent_id = fields.Many2one('hr.department', string='Parent Department', index=True, check_company=True)
    child_ids = fields.One2many('hr.department', 'parent_id', string='Child Departments')
    manager_id = fields.Many2one('hr.employee', string='Manager', tracking=True, check_company=True)
    member_ids = fields.One2many('hr.employee', 'department_id', string='Members', readonly=True)
    total_employee = fields.Integer(compute='_compute_total_employee', string='Total Employee')
    jobs_ids = fields.One2many('hr.job', 'department_id', string='Jobs')
    plan_ids = fields.One2many('mail.activity.plan', 'department_id')
    plans_count = fields.Integer(compute='_compute_plan_count')
    note = fields.Text('Note')
    color = fields.Integer('Color Index')
    parent_path = fields.Char(index=True, unaccent=False)
    master_department_id = fields.Many2one(
        'hr.department', 'Master Department', compute='_compute_master_department_id', store=True)

    @api.depends_context('hierarchical_naming')
    def _compute_display_name(self):
        if self.env.context.get('hierarchical_naming', True):
            return super()._compute_display_name()
        for record in self:
            record.display_name = record.name

    @api.model
    def name_create(self, name):
        record = self.create({'name': name})
        return record.id, record.display_name

    @api.depends('name', 'parent_id.complete_name')
    def _compute_complete_name(self):
        for department in self:
            if department.parent_id:
                department.complete_name = '%s / %s' % (department.parent_id.complete_name, department.name)
            else:
                department.complete_name = department.name

    @api.depends('parent_path')
    def _compute_master_department_id(self):
        for department in self:
            department.master_department_id = int(department.parent_path.split('/')[0])

    def _compute_total_employee(self):
        emp_data = self.env['hr.employee'].sudo()._read_group([('department_id', 'in', self.ids)], ['department_id'], ['__count'])
        result = {department.id: count for department, count in emp_data}
        for department in self:
            department.total_employee = result.get(department.id, 0)

    def _compute_plan_count(self):
        plans_data = self.env['mail.activity.plan']._read_group(
            domain=[
                '|',
                ('department_id', '=', False),
                ('department_id', 'in', self.ids)
            ],
            groupby=['department_id'],
            aggregates=['__count'],
        )
        plans_count = {department.id: count for department, count in plans_data}
        for department in self:
            department.plans_count = plans_count.get(department.id, 0) + plans_count.get(False, 0)

    @api.constrains('parent_id')
    def _check_parent_id(self):
        if not self._check_recursion():
            raise ValidationError(_('You cannot create recursive departments.'))

    @api.model_create_multi
    def create(self, vals_list):
        # TDE note: auto-subscription of manager done by hand, because currently
        # the tracking allows to track+subscribe fields linked to a res.user record
        # An update of the limited behavior should come, but not currently done.
        departments = super(Department, self.with_context(mail_create_nosubscribe=True)).create(vals_list)
        for department, vals in zip(departments, vals_list):
            manager = self.env['hr.employee'].browse(vals.get("manager_id"))
            if manager.user_id:
                department.message_subscribe(partner_ids=manager.user_id.partner_id.ids)
        return departments

    def write(self, vals):
        """ If updating manager of a department, we need to update all the employees
            of department hierarchy, and subscribe the new manager.
        """
        # TDE note: auto-subscription of manager done by hand, because currently
        # the tracking allows to track+subscribe fields linked to a res.user record
        # An update of the limited behavior should come, but not currently done.
        if 'manager_id' in vals:
            manager_id = vals.get("manager_id")
            if manager_id:
                manager = self.env['hr.employee'].browse(manager_id)
                # subscribe the manager user
                if manager.user_id:
                    self.message_subscribe(partner_ids=manager.user_id.partner_id.ids)
            # set the employees's parent to the new manager
            self._update_employee_manager(manager_id)
        return super(Department, self).write(vals)

    def _update_employee_manager(self, manager_id):
        employees = self.env['hr.employee']
        for department in self:
            employees = employees | self.env['hr.employee'].search([
                ('id', '!=', manager_id),
                ('department_id', '=', department.id),
                ('parent_id', '=', department.manager_id.id)
            ])
        employees.write({'parent_id': manager_id})

    def get_formview_action(self, access_uid=None):
        res = super().get_formview_action(access_uid=access_uid)
        if (not self.user_has_groups('hr.group_hr_user') and
           self.env.context.get('open_employees_kanban', False)):
            res.update({
                'name': self.name,
                'res_model': 'hr.employee.public',
                'view_mode': 'kanban',
                'views': [(False, 'kanban'), (False, 'form')],
                'context': {'searchpanel_default_department_id': self.id},
                'res_id': False,
            })
        return res

    def action_plan_from_department(self):
        action = self.env['ir.actions.actions']._for_xml_id('hr.mail_activity_plan_action')
        action['context'] = dict(ast.literal_eval(action.get('context')), default_department_id=self.id)
        domain = [
            '|',
            ('department_id', '=', False),
            ('department_id', 'in', self.ids),
        ]
        action['domain'] = expression.AND([ast.literal_eval(action['domain']), domain]) if 'domain' in action else domain
        if self.plans_count == 0:
            action['views'] = [(False, 'form')]
        return action

    def get_children_department_ids(self):
        return self.env['hr.department'].search([('id', 'child_of', self.ids)])

    def get_department_hierarchy(self):
        if not self:
            return {}

        hierarchy = {
            'parent': {
                'id': self.parent_id.id,
                'name': self.parent_id.name,
                'employees': self.parent_id.total_employee,
            } if self.parent_id else False,
            'self': {
                'id': self.id,
                'name': self.name,
                'employees': self.total_employee,
            },
            'children': [
                {
                    'id': child.id,
                    'name': child.name,
                    'employees': child.total_employee
                } for child in self.child_ids
            ]
        }

        return hierarchy

```

## File: models\hr_departure_reason.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class DepartureReason(models.Model):
    _name = "hr.departure.reason"
    _description = "Departure Reason"
    _order = "sequence"

    sequence = fields.Integer("Sequence", default=10)
    name = fields.Char(string="Reason", required=True, translate=True)
    reason_code = fields.Integer()

    def _get_default_departure_reasons(self):
        return {
            'fired': 342,
            'resigned': 343,
            'retired': 340,
        }

    @api.ondelete(at_uninstall=False)
    def _unlink_except_default_departure_reasons(self):
        master_departure_codes = self._get_default_departure_reasons().values()
        if any(reason.reason_code in master_departure_codes for reason in self):
            raise UserError(_('Default departure reasons cannot be deleted.'))

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
from pytz import timezone, UTC
from datetime import datetime, time
from random import choice
from string import digits
from werkzeug.urls import url_encode
from dateutil.relativedelta import relativedelta
from markupsafe import Markup

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError, AccessError
from odoo.osv import expression
from odoo.tools import format_date


class HrEmployeePrivate(models.Model):
    """
    NB: Any field only available on the model hr.employee (i.e. not on the
    hr.employee.public model) should have `groups="hr.group_hr_user"` on its
    definition to avoid being prefetched when the user hasn't access to the
    hr.employee model. Indeed, the prefetch loads the data for all the fields
    that are available according to the group defined on them.
    """
    _name = "hr.employee"
    _description = "Employee"
    _order = 'name'
    _inherit = ['hr.employee.base', 'mail.thread.main.attachment', 'mail.activity.mixin', 'resource.mixin', 'avatar.mixin']
    _mail_post_access = 'read'

    @api.model
    def _lang_get(self):
        return self.env['res.lang'].get_installed()

    # resource and user
    # required on the resource, make sure required="True" set in the view
    name = fields.Char(string="Employee Name", related='resource_id.name', store=True, readonly=False, tracking=True)
    user_id = fields.Many2one('res.users', 'User', related='resource_id.user_id', store=True, readonly=False, ondelete='restrict')
    user_partner_id = fields.Many2one(related='user_id.partner_id', related_sudo=False, string="User's partner")
    active = fields.Boolean('Active', related='resource_id.active', default=True, store=True, readonly=False)
    resource_calendar_id = fields.Many2one(tracking=True)
    department_id = fields.Many2one(tracking=True)
    company_id = fields.Many2one('res.company', required=True)
    company_country_id = fields.Many2one('res.country', 'Company Country', related='company_id.country_id', readonly=True)
    company_country_code = fields.Char(related='company_country_id.code', depends=['company_country_id'], readonly=True)
    # private info
    private_street = fields.Char(string="Private Street", groups="hr.group_hr_user")
    private_street2 = fields.Char(string="Private Street2", groups="hr.group_hr_user")
    private_city = fields.Char(string="Private City", groups="hr.group_hr_user")
    private_state_id = fields.Many2one(
        "res.country.state", string="Private State",
        domain="[('country_id', '=?', private_country_id)]",
        groups="hr.group_hr_user")
    private_zip = fields.Char(string="Private Zip", groups="hr.group_hr_user")
    private_country_id = fields.Many2one("res.country", string="Private Country", groups="hr.group_hr_user")
    private_phone = fields.Char(string="Private Phone", groups="hr.group_hr_user")
    private_email = fields.Char(string="Private Email", groups="hr.group_hr_user")
    lang = fields.Selection(selection=_lang_get, string="Lang", groups="hr.group_hr_user")
    country_id = fields.Many2one(
        'res.country', 'Nationality (Country)', groups="hr.group_hr_user", tracking=True)
    gender = fields.Selection([
        ('male', 'Male'),
        ('female', 'Female'),
        ('other', 'Other')
    ], groups="hr.group_hr_user", tracking=True)
    marital = fields.Selection([
        ('single', 'Single'),
        ('married', 'Married'),
        ('cohabitant', 'Legal Cohabitant'),
        ('widower', 'Widower'),
        ('divorced', 'Divorced')
    ], string='Marital Status', groups="hr.group_hr_user", default='single', tracking=True)
    spouse_complete_name = fields.Char(string="Spouse Complete Name", groups="hr.group_hr_user", tracking=True)
    spouse_birthdate = fields.Date(string="Spouse Birthdate", groups="hr.group_hr_user", tracking=True)
    children = fields.Integer(string='Number of Dependent Children', groups="hr.group_hr_user", tracking=True)
    place_of_birth = fields.Char('Place of Birth', groups="hr.group_hr_user", tracking=True)
    country_of_birth = fields.Many2one('res.country', string="Country of Birth", groups="hr.group_hr_user", tracking=True)
    birthday = fields.Date('Date of Birth', groups="hr.group_hr_user", tracking=True)
    ssnid = fields.Char('SSN No', help='Social Security Number', groups="hr.group_hr_user", tracking=True)
    sinid = fields.Char('SIN No', help='Social Insurance Number', groups="hr.group_hr_user", tracking=True)
    identification_id = fields.Char(string='Identification No', groups="hr.group_hr_user", tracking=True)
    passport_id = fields.Char('Passport No', groups="hr.group_hr_user", tracking=True)
    bank_account_id = fields.Many2one(
        'res.partner.bank', 'Bank Account Number',
        domain="[('partner_id', '=', work_contact_id), '|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        groups="hr.group_hr_user",
        tracking=True,
        help='Employee bank account to pay salaries')
    permit_no = fields.Char('Work Permit No', groups="hr.group_hr_user", tracking=True)
    visa_no = fields.Char('Visa No', groups="hr.group_hr_user", tracking=True)
    visa_expire = fields.Date('Visa Expiration Date', groups="hr.group_hr_user", tracking=True)
    work_permit_expiration_date = fields.Date('Work Permit Expiration Date', groups="hr.group_hr_user", tracking=True)
    has_work_permit = fields.Binary(string="Work Permit", groups="hr.group_hr_user")
    work_permit_scheduled_activity = fields.Boolean(default=False, groups="hr.group_hr_user")
    work_permit_name = fields.Char('work_permit_name', compute='_compute_work_permit_name')
    additional_note = fields.Text(string='Additional Note', groups="hr.group_hr_user", tracking=True)
    certificate = fields.Selection([
        ('graduate', 'Graduate'),
        ('bachelor', 'Bachelor'),
        ('master', 'Master'),
        ('doctor', 'Doctor'),
        ('other', 'Other'),
    ], 'Certificate Level', default='other', groups="hr.group_hr_user", tracking=True)
    study_field = fields.Char("Field of Study", groups="hr.group_hr_user", tracking=True)
    study_school = fields.Char("School", groups="hr.group_hr_user", tracking=True)
    emergency_contact = fields.Char("Contact Name", groups="hr.group_hr_user", tracking=True)
    emergency_phone = fields.Char("Contact Phone", groups="hr.group_hr_user", tracking=True)
    km_home_work = fields.Integer(string="Home-Work Distance", groups="hr.group_hr_user", tracking=True)
    employee_type = fields.Selection([
            ('employee', 'Employee'),
            ('student', 'Student'),
            ('trainee', 'Trainee'),
            ('contractor', 'Contractor'),
            ('freelance', 'Freelancer'),
        ], string='Employee Type', default='employee', required=True, groups="hr.group_hr_user",
        help="The employee type. Although the primary purpose may seem to categorize employees, this field has also an impact in the Contract History. Only Employee type is supposed to be under contract and will have a Contract History.")

    job_id = fields.Many2one(tracking=True)
    # employee in company
    child_ids = fields.One2many('hr.employee', 'parent_id', string='Direct subordinates')
    category_ids = fields.Many2many(
        'hr.employee.category', 'employee_category_rel',
        'emp_id', 'category_id', groups="hr.group_hr_user",
        string='Tags')
    # misc
    notes = fields.Text('Notes', groups="hr.group_hr_user")
    color = fields.Integer('Color Index', default=0)
    barcode = fields.Char(string="Badge ID", help="ID used for employee identification.", groups="hr.group_hr_user", copy=False)
    pin = fields.Char(string="PIN", groups="hr.group_hr_user", copy=False,
        help="PIN used to Check In/Out in the Kiosk Mode of the Attendance application (if enabled in Configuration) and to change the cashier in the Point of Sale application.")
    departure_reason_id = fields.Many2one("hr.departure.reason", string="Departure Reason", groups="hr.group_hr_user",
                                          copy=False, tracking=True, ondelete='restrict')
    departure_description = fields.Html(string="Additional Information", groups="hr.group_hr_user", copy=False)
    departure_date = fields.Date(string="Departure Date", groups="hr.group_hr_user", copy=False, tracking=True)
    message_main_attachment_id = fields.Many2one(groups="hr.group_hr_user")
    id_card = fields.Binary(string="ID Card Copy", groups="hr.group_hr_user")
    driving_license = fields.Binary(string="Driving License", groups="hr.group_hr_user")
    private_car_plate = fields.Char(groups="hr.group_hr_user", help="If you have more than one car, just separate the plates by a space.")
    currency_id = fields.Many2one('res.currency', related='company_id.currency_id', readonly=True)
    # properties
    employee_properties = fields.Properties('Properties', definition='company_id.employee_properties_definition', precompute=False)

    _sql_constraints = [
        ('barcode_uniq', 'unique (barcode)', "The Badge ID must be unique, this one is already assigned to another employee."),
        ('user_uniq', 'unique (user_id, company_id)', "A user cannot be linked to multiple employees in the same company.")
    ]

    @api.depends('name', 'user_id.avatar_1920', 'image_1920')
    def _compute_avatar_1920(self):
        super()._compute_avatar_1920()

    @api.depends('name', 'user_id.avatar_1024', 'image_1024')
    def _compute_avatar_1024(self):
        super()._compute_avatar_1024()

    @api.depends('name', 'user_id.avatar_512', 'image_512')
    def _compute_avatar_512(self):
        super()._compute_avatar_512()

    @api.depends('name', 'user_id.avatar_256', 'image_256')
    def _compute_avatar_256(self):
        super()._compute_avatar_256()

    @api.depends('name', 'user_id.avatar_128', 'image_128')
    def _compute_avatar_128(self):
        super()._compute_avatar_128()

    def _compute_avatar(self, avatar_field, image_field):
        employee_wo_user_and_image = self.env['hr.employee']
        for employee in self:
            if not employee.user_id and not employee._origin[image_field]:
                employee_wo_user_and_image += employee
                continue
            avatar = employee._origin[image_field]
            if not avatar and employee.user_id:
                avatar = employee.user_id.sudo()[avatar_field]
            employee[avatar_field] = avatar
        super(HrEmployeePrivate, employee_wo_user_and_image)._compute_avatar(avatar_field, image_field)

    @api.depends('name', 'permit_no')
    def _compute_work_permit_name(self):
        for employee in self:
            name = employee.name.replace(' ', '_') + '_' if employee.name else ''
            permit_no = '_' + employee.permit_no if employee.permit_no else ''
            employee.work_permit_name = "%swork_permit%s" % (name, permit_no)

    def action_create_user(self):
        self.ensure_one()
        if self.user_id:
            raise ValidationError(_("This employee already has an user."))
        return {
            'name': _('Create User'),
            'type': 'ir.actions.act_window',
            'res_model': 'res.users',
            'view_mode': 'form',
            'view_id': self.env.ref('hr.view_users_simple_form').id,
            'target': 'new',
            'context': dict(self._context, **{
                'default_create_employee_id': self.id,
                'default_name': self.name,
                'default_phone': self.work_phone,
                'default_mobile': self.mobile_phone,
                'default_login': self.work_email,
                'default_partner_id': self.work_contact_id.id,
            })
        }

    def _compute_display_name(self):
        if self.check_access_rights('read', raise_exception=False):
            return super()._compute_display_name()
        for employee_private, employee_public in zip(self, self.env['hr.employee.public'].browse(self.ids)):
            employee_private.display_name = employee_public.display_name

    def search_fetch(self, domain, field_names, offset=0, limit=None, order=None):
        if self.check_access_rights('read', raise_exception=False):
            return super().search_fetch(domain, field_names, offset, limit, order)

        # HACK: retrieve publicly available values from hr.employee.public and
        # copy them to the cache of self; non-public data will be missing from
        # cache, and interpreted as an access error
        self._check_private_fields(field_names)
        self.flush_model(field_names)
        public = self.env['hr.employee.public'].search_fetch(domain, field_names, offset, limit, order)
        employees = self.browse(public._ids)
        employees._copy_cache_from(public, field_names)
        return employees

    def fetch(self, field_names):
        if self.check_access_rights('read', raise_exception=False):
            return super().fetch(field_names)

        # HACK: retrieve publicly available values from hr.employee.public and
        # copy them to the cache of self; non-public data will be missing from
        # cache, and interpreted as an access error
        self._check_private_fields(field_names)
        self.flush_recordset(field_names)
        public = self.env['hr.employee.public'].browse(self._ids)
        public.fetch(field_names)
        self._copy_cache_from(public, field_names)

    def _check_private_fields(self, field_names):
        """ Check whether ``field_names`` contain private fields. """
        public_fields = self.env['hr.employee.public']._fields
        private_fields = [fname for fname in field_names if fname not in public_fields]
        if private_fields:
            raise AccessError(_('The fields %r you try to read is not available on the public employee profile.', ','.join(private_fields)))

    def _copy_cache_from(self, public, field_names):
        # HACK: retrieve publicly available values from hr.employee.public and
        # copy them to the cache of self; non-public data will be missing from
        # cache, and interpreted as an access error
        for fname in field_names:
            values = self.env.cache.get_values(public, public._fields[fname])
            if self._fields[fname].translate:
                values = [(value.copy() if value else None) for value in values]
            self.env.cache.update_raw(self, self._fields[fname], values)

    @api.model
    def _cron_check_work_permit_validity(self):
        # Called by a cron
        # Schedule an activity 1 month before the work permit expires
        outdated_days = fields.Date.today() + relativedelta(months=+1)
        nearly_expired_work_permits = self.search([('work_permit_scheduled_activity', '=', False), ('work_permit_expiration_date', '<', outdated_days)])
        employees_scheduled = self.env['hr.employee']
        for employee in nearly_expired_work_permits.filtered(lambda employee: employee.parent_id):
            responsible_user_id = employee.parent_id.user_id.id
            if responsible_user_id:
                employees_scheduled |= employee
                lang = self.env['res.users'].browse(responsible_user_id).lang
                formated_date = format_date(employee.env, employee.work_permit_expiration_date, date_format="dd MMMM y", lang_code=lang)
                employee.activity_schedule(
                    'mail.mail_activity_data_todo',
                    note=_('The work permit of %(employee)s expires at %(date)s.',
                        employee=employee.name,
                        date=formated_date),
                    user_id=responsible_user_id)
        employees_scheduled.write({'work_permit_scheduled_activity': True})

    @api.model
    def get_view(self, view_id=None, view_type='form', **options):
        if self.check_access_rights('read', raise_exception=False):
            return super().get_view(view_id, view_type, **options)
        return self.env['hr.employee.public'].get_view(view_id, view_type, **options)

    @api.model
    def get_views(self, views, options=None):
        if self.check_access_rights('read', raise_exception=False):
            return super().get_views(views, options)
        res = self.env['hr.employee.public'].get_views(views, options)
        res['models'].update({'hr.employee': res['models']['hr.employee.public']})
        return res

    @api.model
    def _search(self, domain, offset=0, limit=None, order=None, access_rights_uid=None):
        """
            We override the _search because it is the method that checks the access rights
            This is correct to override the _search. That way we enforce the fact that calling
            search on an hr.employee returns a hr.employee recordset, even if you don't have access
            to this model, as the result of _search (the ids of the public employees) is to be
            browsed on the hr.employee model. This can be trusted as the ids of the public
            employees exactly match the ids of the related hr.employee.
        """
        if self.check_access_rights('read', raise_exception=False):
            return super()._search(domain, offset, limit, order, access_rights_uid)
        try:
            ids = self.env['hr.employee.public']._search(domain, offset, limit, order, access_rights_uid)
        except ValueError:
            raise AccessError(_('You do not have access to this document.'))
        # the result is expected from this table, so we should link tables
        return super(HrEmployeePrivate, self.sudo())._search([('id', 'in', ids)], order=order)

    def get_formview_id(self, access_uid=None):
        """ Override this method in order to redirect many2one towards the right model depending on access_uid """
        if access_uid:
            self_sudo = self.with_user(access_uid)
        else:
            self_sudo = self

        if self_sudo.user_has_groups('hr.group_hr_user'):
            return super(HrEmployeePrivate, self).get_formview_id(access_uid=access_uid)
        # Hardcode the form view for public employee
        return self.env.ref('hr.hr_employee_public_view_form').id

    def get_formview_action(self, access_uid=None):
        """ Override this method in order to redirect many2one towards the right model depending on access_uid """
        res = super(HrEmployeePrivate, self).get_formview_action(access_uid=access_uid)
        if access_uid:
            self_sudo = self.with_user(access_uid)
        else:
            self_sudo = self

        if not self_sudo.user_has_groups('hr.group_hr_user'):
            res['res_model'] = 'hr.employee.public'

        return res

    @api.constrains('pin')
    def _verify_pin(self):
        for employee in self:
            if employee.pin and not employee.pin.isdigit():
                raise ValidationError(_("The PIN must be a sequence of digits."))

    @api.constrains('ssnid')
    def _check_ssnid(self):
        # By default, an Social Security Number is always valid, but each localization
        # may want to add its own constraints
        pass

    @api.onchange('user_id')
    def _onchange_user(self):
        self.update(self._sync_user(self.user_id, (bool(self.image_1920))))
        if not self.name:
            self.name = self.user_id.name

    @api.onchange('resource_calendar_id')
    def _onchange_timezone(self):
        if self.resource_calendar_id and not self.tz:
            self.tz = self.resource_calendar_id.tz

    def _remove_work_contact_id(self, user, employee_company):
        """ Remove work_contact_id for previous employee if the user is assigned to a new employee """
        employee_company = employee_company or self.company_id.id
        # For employees with a user_id, the constraint (user can't be linked to multiple employees) is triggered
        old_partner_employee_ids = user.partner_id.employee_ids.filtered(lambda e:
            not e.user_id
            and e.company_id.id == employee_company
            and e != self
        )
        old_partner_employee_ids.work_contact_id = None

    def _sync_user(self, user, employee_has_image=False):
        vals = dict(
            work_contact_id=user.partner_id.id if user else self.work_contact_id.id,
            user_id=user.id,
        )
        if not employee_has_image:
            vals['image_1920'] = user.image_1920
        if user.tz:
            vals['tz'] = user.tz
        return vals

    def _prepare_resource_values(self, vals, tz):
        resource_vals = super()._prepare_resource_values(vals, tz)
        vals.pop('name')  # Already considered by super call but no popped
        # We need to pop it to avoid useless resource update (& write) call
        # on every newly created resource (with the correct name already)
        user_id = vals.pop('user_id', None)
        if user_id:
            resource_vals['user_id'] = user_id
        active_status = vals.get('active')
        if active_status is not None:
            resource_vals['active'] = active_status
        return resource_vals

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if vals.get('user_id'):
                user = self.env['res.users'].browse(vals['user_id'])
                vals.update(self._sync_user(user, bool(vals.get('image_1920'))))
                vals['name'] = vals.get('name', user.name)
                self._remove_work_contact_id(user, vals.get('company_id'))
        employees = super().create(vals_list)
        # Sudo in case HR officer doesn't have the Contact Creation group
        employees.filtered(lambda e: not e.work_contact_id).sudo()._create_work_contacts()
        for employee_sudo in employees.sudo():
            # creating 'svg/xml' attachments requires specific rights
            if not employee_sudo.image_1920 and self.env['ir.ui.view'].sudo(False).check_access_rights('write', raise_exception=False):
                employee_sudo.image_1920 = employee_sudo._avatar_generate_svg()
                employee_sudo.work_contact_id.image_1920 = employee_sudo.image_1920
        if self.env.context.get('salary_simulation'):
            return employees
        employee_departments = employees.department_id
        if employee_departments:
            self.env['discuss.channel'].sudo().search([
                ('subscription_department_ids', 'in', employee_departments.ids)
            ])._subscribe_users_automatically()
        onboarding_notes_bodies = {}
        hr_root_menu = self.env.ref('hr.menu_hr_root')
        for employee in employees:
            # Launch onboarding plans
            url = '/web#%s' % url_encode({
                'action': 'hr.plan_wizard_action',
                'active_id': employee.id,
                'active_model': 'hr.employee',
                'menu_id': hr_root_menu.id,
            })
            onboarding_notes_bodies[employee.id] = Markup(_(
                '<b>Congratulations!</b> May I recommend you to setup an <a href="%s">onboarding plan?</a>',
            )) % url
        employees._message_log_batch(onboarding_notes_bodies)
        return employees

    def write(self, vals):
        if 'work_contact_id' in vals:
            account_ids = vals.get('bank_account_id') or self.bank_account_id.ids
            if account_ids:
                bank_accounts = self.env['res.partner.bank'].sudo().browse(account_ids)
                for bank_account in bank_accounts:
                    if vals['work_contact_id'] != bank_account.partner_id.id:
                        if bank_account.allow_out_payment:
                            bank_account.allow_out_payment = False
                        if vals['work_contact_id']:
                            bank_account.partner_id = vals['work_contact_id']
            self.message_unsubscribe(self.work_contact_id.ids)
            if vals['work_contact_id']:
                self._message_subscribe([vals['work_contact_id']])
        if vals.get('user_id'):
            # Update the profile pictures with user, except if provided
            user = self.env['res.users'].browse(vals['user_id'])
            vals.update(self._sync_user(user, (bool(all(emp.image_1920 for emp in self)))))
            self._remove_work_contact_id(user, vals.get('company_id'))
        if 'work_permit_expiration_date' in vals:
            vals['work_permit_scheduled_activity'] = False
        res = super(HrEmployeePrivate, self).write(vals)
        if vals.get('department_id') or vals.get('user_id'):
            department_id = vals['department_id'] if vals.get('department_id') else self[:1].department_id.id
            # When added to a department or changing user, subscribe to the channels auto-subscribed by department
            self.env['discuss.channel'].sudo().search([
                ('subscription_department_ids', 'in', department_id)
            ])._subscribe_users_automatically()
        if vals.get('departure_description'):
            for employee in self:
                employee.message_post(body=_(
                    'Additional Information: \n %(description)s',
                    description=vals.get('departure_description')))
        return res

    def unlink(self):
        resources = self.mapped('resource_id')
        super(HrEmployeePrivate, self).unlink()
        return resources.unlink()

    def _get_employee_m2o_to_empty_on_archived_employees(self):
        return ['parent_id', 'coach_id']

    def _get_user_m2o_to_empty_on_archived_employees(self):
        return []

    def toggle_active(self):
        res = super(HrEmployeePrivate, self).toggle_active()
        unarchived_employees = self.filtered(lambda employee: employee.active)
        unarchived_employees.write({
            'departure_reason_id': False,
            'departure_description': False,
            'departure_date': False
        })

        archived_employees = self.filtered(lambda e: not e.active)
        if archived_employees:
            # Empty links to this employees (example: manager, coach, time off responsible, ...)
            employee_fields_to_empty = self._get_employee_m2o_to_empty_on_archived_employees()
            user_fields_to_empty = self._get_user_m2o_to_empty_on_archived_employees()
            employee_domain = [[(field, 'in', archived_employees.ids)] for field in employee_fields_to_empty]
            user_domain = [[(field, 'in', archived_employees.user_id.ids) for field in user_fields_to_empty]]
            employees = self.env['hr.employee'].search(expression.OR(employee_domain + user_domain))
            for employee in employees:
                for field in employee_fields_to_empty:
                    if employee[field] in archived_employees:
                        employee[field] = False
                for field in user_fields_to_empty:
                    if employee[field] in archived_employees.user_id:
                        employee[field] = False

        if len(self) == 1 and not self.active and not self.env.context.get('no_wizard', False):
            return {
                'type': 'ir.actions.act_window',
                'name': _('Register Departure'),
                'res_model': 'hr.departure.wizard',
                'view_mode': 'form',
                'target': 'new',
                'context': {'active_id': self.id},
                'views': [[False, 'form']]
            }
        return res

    @api.onchange('company_id')
    def _onchange_company_id(self):
        if self._origin:
            return {'warning': {
                'title': _("Warning"),
                'message': _("To avoid multi company issues (losing the access to your previous contracts, leaves, ...), you should create another employee in the new company instead.")
            }}

    def generate_random_barcode(self):
        for employee in self:
            employee.barcode = '041'+"".join(choice(digits) for i in range(9))

    def _get_tz(self):
        # Finds the first valid timezone in his tz, his work hours tz,
        #  the company calendar tz or UTC and returns it as a string
        self.ensure_one()
        return self.tz or\
               self.resource_calendar_id.tz or\
               self.company_id.resource_calendar_id.tz or\
               'UTC'

    def _get_tz_batch(self):
        # Finds the first valid timezone in his tz, his work hours tz,
        #  the company calendar tz or UTC
        # Returns a dict {employee_id: tz}
        return {emp.id: emp._get_tz() for emp in self}

    def _employee_attendance_intervals(self, start, stop, lunch=False):
        self.ensure_one()
        calendar = self.resource_calendar_id or self.company_id.resource_calendar_id
        if not lunch:
            return self._get_expected_attendances(start, stop)
        else:
            return calendar._attendance_intervals_batch(start, stop, self.resource_id, lunch=True)[self.resource_id.id]

    def _get_expected_attendances(self, date_from, date_to):
        self.ensure_one()
        employee_timezone = timezone(self.tz) if self.tz else None
        calendar = self.resource_calendar_id or self.company_id.resource_calendar_id
        calendar_intervals = calendar._work_intervals_batch(
                                date_from,
                                date_to,
                                tz=employee_timezone,
                                resources=self.resource_id,
                                compute_leaves=True,
                                domain=[('company_id', 'in', [False, self.company_id.id])])[self.resource_id.id]
        return calendar_intervals

    def _get_calendar_attendances(self, date_from, date_to):
        self.ensure_one()
        employee_timezone = timezone(self.tz) if self.tz else None
        calendar = self.resource_calendar_id or self.company_id.resource_calendar_id
        return calendar\
            .with_context(employee_timezone=employee_timezone)\
            .get_work_duration_data(
                date_from,
                date_to,
                domain=[('company_id', 'in', [False, self.company_id.id])])

    # ---------------------------------------------------------
    # Business Methods
    # ---------------------------------------------------------

    @api.model
    def get_import_templates(self):
        return [{
            'label': _('Import Template for Employees'),
            'template': '/hr/static/xls/hr_employee.xls'
        }]

    def _get_unusual_days(self, date_from, date_to=None):
        # Checking the calendar directly allows to not grey out the leaves taken
        # by the employee or fallback to the company calendar
        return (self.resource_calendar_id or self.env.company.resource_calendar_id)._get_unusual_days(
            datetime.combine(fields.Date.from_string(date_from), time.min).replace(tzinfo=UTC),
            datetime.combine(fields.Date.from_string(date_to), time.max).replace(tzinfo=UTC)
        )

    def _get_age(self, target_date=None):
        self.ensure_one()
        if target_date is None:
            target_date = fields.Date.context_today(self.env.user)
        return relativedelta(target_date, self.birthday).years if self.birthday else 0

    # ---------------------------------------------------------
    # Messaging
    # ---------------------------------------------------------

    def _phone_get_number_fields(self):
        return ['mobile_phone']

    def _mail_get_partner_fields(self, introspect_fields=False):
        return ['user_partner_id']

```

## File: models\hr_employee_base.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval

from pytz import timezone, UTC, utc
from datetime import timedelta, datetime

from odoo import _, api, fields, models
from odoo.exceptions import UserError
from odoo.tools import format_time


class HrEmployeeBase(models.AbstractModel):
    _name = "hr.employee.base"
    _description = "Basic Employee"
    _order = 'name'

    name = fields.Char()
    active = fields.Boolean("Active")
    color = fields.Integer('Color Index', default=0)
    department_id = fields.Many2one('hr.department', 'Department', check_company=True)
    member_of_department = fields.Boolean("Member of department", compute='_compute_part_of_department', search='_search_part_of_department',
        help="Whether the employee is a member of the active user's department or one of it's child department.")
    job_id = fields.Many2one('hr.job', 'Job Position', check_company=True)
    job_title = fields.Char("Job Title", compute="_compute_job_title", store=True, readonly=False)
    company_id = fields.Many2one('res.company', 'Company')
    address_id = fields.Many2one(
        'res.partner',
        string='Work Address',
        compute="_compute_address_id",
        precompute=True,
        store=True,
        readonly=False,
        check_company=True)
    work_phone = fields.Char('Work Phone', compute="_compute_phones", store=True, readonly=False)
    mobile_phone = fields.Char('Work Mobile', compute="_compute_work_contact_details", store=True, inverse='_inverse_work_contact_details')
    work_email = fields.Char('Work Email', compute="_compute_work_contact_details", store=True, inverse='_inverse_work_contact_details')
    work_contact_id = fields.Many2one('res.partner', 'Work Contact', copy=False)
    work_location_id = fields.Many2one('hr.work.location', 'Work Location', domain="[('address_id', '=', address_id)]")
    user_id = fields.Many2one('res.users')
    resource_id = fields.Many2one('resource.resource')
    resource_calendar_id = fields.Many2one('resource.calendar', check_company=True)
    parent_id = fields.Many2one('hr.employee', 'Manager', compute="_compute_parent_id", store=True, readonly=False,
        domain="['|', ('company_id', '=', False), ('company_id', 'in', allowed_company_ids)]")
    coach_id = fields.Many2one(
        'hr.employee', 'Coach', compute='_compute_coach', store=True, readonly=False,
        check_company=True,
        help='Select the "Employee" who is the coach of this employee.\n'
             'The "Coach" has no specific rights or responsibilities by default.')
    tz = fields.Selection(
        string='Timezone', related='resource_id.tz', readonly=False,
        help="This field is used in order to define in which timezone the resources will work.")
    hr_presence_state = fields.Selection([
        ('present', 'Present'),
        ('absent', 'Absent'),
        ('to_define', 'To Define')], compute='_compute_presence_state', default='to_define')
    last_activity = fields.Date(compute="_compute_last_activity")
    last_activity_time = fields.Char(compute="_compute_last_activity")
    hr_icon_display = fields.Selection([
        ('presence_present', 'Present'),
        ('presence_absent_active', 'Present but not active'),
        ('presence_absent', 'Absent'),
        ('presence_to_define', 'To define'),
        ('presence_undetermined', 'Undetermined')], compute='_compute_presence_icon')
    show_hr_icon_display = fields.Boolean(compute='_compute_presence_icon')
    newly_hired = fields.Boolean('Newly Hired', compute='_compute_newly_hired', search='_search_newly_hired')

    @api.model
    def _get_new_hire_field(self):
        return 'create_date'

    def _compute_newly_hired(self):
        new_hire_field = self._get_new_hire_field()
        new_hire_date = fields.Datetime.now() - timedelta(days=90)
        for employee in self:
            if not employee[new_hire_field]:
                employee.newly_hired = False
            elif not isinstance(employee[new_hire_field], datetime):
                employee.newly_hired = employee[new_hire_field] > new_hire_date.date()
            else:
                employee.newly_hired = employee[new_hire_field] > new_hire_date

    def _search_newly_hired(self, operator, value):
        new_hire_field = self._get_new_hire_field()
        new_hires = self.env['hr.employee'].sudo().search([
            (new_hire_field, '>', fields.Datetime.now() - timedelta(days=90))
        ])

        op = 'in' if value and operator == '=' or not value and operator != '=' else 'not in'
        return [('id', op, new_hires.ids)]


    def _get_valid_employee_for_user(self):
        user = self.env.user
        # retrieve the employee of the current active company for the user
        employee = user.employee_id
        if not employee:
            # search for all employees as superadmin to not get blocked by multi-company rules
            user_employees = user.employee_id.sudo().search([
                ('user_id', '=', user.id)
            ])
            # the default company employee is most likely the correct one, but fallback to the first if not available
            employee = user_employees.filtered(lambda r: r.company_id == user.company_id) or user_employees[:1]
        return employee

    @api.depends_context('uid', 'company')
    @api.depends('department_id')
    def _compute_part_of_department(self):
        user_employee = self._get_valid_employee_for_user()
        active_department = user_employee.department_id
        if not active_department:
            self.member_of_department = False
        else:
            def get_all_children(department):
                children = department.child_ids
                if not children:
                    return self.env['hr.department']
                return children + get_all_children(children)

            child_departments = active_department + get_all_children(active_department)
            for employee in self:
                employee.member_of_department = employee.department_id in child_departments

    def _search_part_of_department(self, operator, value):
        if operator not in ('=', '!=') or not isinstance(value, bool):
            raise UserError(_('Operation not supported'))

        user_employee = self._get_valid_employee_for_user()
        # Double negation
        if not value:
            operator = '!=' if operator == '=' else '='
        if not user_employee.department_id:
            return [('id', operator, user_employee.id)]
        return (['!'] if operator == '!=' else []) + [('department_id', 'child_of', user_employee.department_id.id)]

    @api.depends('user_id.im_status')
    def _compute_presence_state(self):
        """
        This method is overritten in several other modules which add additional
        presence criterions. e.g. hr_attendance, hr_holidays
        """
        # Check on login
        check_login = literal_eval(self.env['ir.config_parameter'].sudo().get_param('hr.hr_presence_control_login', 'False'))
        employee_to_check_working = self.filtered(lambda e: e.user_id.im_status == 'offline')
        working_now_list = employee_to_check_working._get_employee_working_now()
        for employee in self:
            state = 'to_define'
            if check_login:
                if employee.user_id.im_status in ['online', 'leave_online']:
                    state = 'present'
                elif employee.user_id.im_status in ['offline', 'leave_offline'] and employee.id not in working_now_list:
                    state = 'absent'
            employee.hr_presence_state = state

    @api.depends('user_id')
    def _compute_last_activity(self):
        presences = self.env['bus.presence'].search_read([('user_id', 'in', self.mapped('user_id').ids)], ['user_id', 'last_presence'])
        # transform the result to a dict with this format {user.id: last_presence}
        presences = {p['user_id'][0]: p['last_presence'] for p in presences}

        for employee in self:
            tz = employee.tz
            last_presence = presences.get(employee.user_id.id, False)
            if last_presence:
                last_activity_datetime = last_presence.replace(tzinfo=UTC).astimezone(timezone(tz)).replace(tzinfo=None)
                employee.last_activity = last_activity_datetime.date()
                if employee.last_activity == fields.Date.today():
                    employee.last_activity_time = format_time(self.env, last_presence, time_format='short')
                else:
                    employee.last_activity_time = False
            else:
                employee.last_activity = False
                employee.last_activity_time = False

    @api.depends('parent_id')
    def _compute_coach(self):
        for employee in self:
            manager = employee.parent_id
            previous_manager = employee._origin.parent_id
            if manager and (employee.coach_id == previous_manager or not employee.coach_id):
                employee.coach_id = manager
            elif not employee.coach_id:
                employee.coach_id = False

    @api.depends('job_id')
    def _compute_job_title(self):
        for employee in self.filtered('job_id'):
            employee.job_title = employee.job_id.name

    @api.depends('address_id')
    def _compute_phones(self):
        for employee in self:
            if employee.address_id and employee.address_id.phone:
                employee.work_phone = employee.address_id.phone
            else:
                employee.work_phone = False

    @api.depends('work_contact_id', 'work_contact_id.mobile', 'work_contact_id.email')
    def _compute_work_contact_details(self):
        for employee in self:
            if employee.work_contact_id:
                employee.mobile_phone = employee.work_contact_id.mobile
                employee.work_email = employee.work_contact_id.email

    def _create_work_contacts(self):
        if any(employee.work_contact_id for employee in self):
            raise UserError(_('Some employee already have a work contact'))
        work_contacts = self.env['res.partner'].create([{
            'email': employee.work_email,
            'mobile': employee.mobile_phone,
            'name': employee.name,
            'image_1920': employee.image_1920,
            'company_id': employee.company_id.id
        } for employee in self])
        for employee, work_contact in zip(self, work_contacts):
            employee.work_contact_id = work_contact

    def _inverse_work_contact_details(self):
        employees_without_work_contact = self.env['hr.employee']
        for employee in self:
            if not employee.work_contact_id:
                employees_without_work_contact += employee
            else:
                employee.work_contact_id.sudo().write({
                    'email': employee.work_email,
                    'mobile': employee.mobile_phone,
                })
        if employees_without_work_contact:
            employees_without_work_contact.sudo()._create_work_contacts()

    @api.depends('company_id')
    def _compute_address_id(self):
        for employee in self:
            address = employee.company_id.partner_id.address_get(['default'])
            employee.address_id = address['default'] if address else False

    @api.depends('department_id')
    def _compute_parent_id(self):
        for employee in self.filtered('department_id.manager_id'):
            employee.parent_id = employee.department_id.manager_id

    @api.depends('resource_calendar_id', 'hr_presence_state')
    def _compute_presence_icon(self):
        """
        This method compute the state defining the display icon in the kanban view.
        It can be overriden to add other possibilities, like time off or attendances recordings.
        """
        working_now_list = self.filtered(lambda e: e.hr_presence_state == 'present')._get_employee_working_now()
        for employee in self:
            show_icon = True
            if employee.hr_presence_state == 'present':
                if employee.id in working_now_list:
                    icon = 'presence_present'
                else:
                    icon = 'presence_absent_active'
            elif employee.hr_presence_state == 'absent':
                # employee is not in the working_now_list and he has a user_id
                icon = 'presence_absent'
            else:
                # without attendance, default employee state is 'to_define' without confirmed presence/absence
                # we need to check why they are not there
                # Display an orange icon on internal users.
                icon = 'presence_to_define'
                if not employee.user_id:
                    # We don't want non-user employee to have icon.
                    show_icon = False
            employee.hr_icon_display = icon
            employee.show_hr_icon_display = show_icon

    @api.model
    def _get_employee_working_now(self):
        working_now = []
        # We loop over all the employee tz and the resource calendar_id to detect working hours in batch.
        all_employee_tz = set(self.mapped('tz'))
        for tz in all_employee_tz:
            employee_ids = self.filtered(lambda e: e.tz == tz)
            resource_calendar_ids = employee_ids.mapped('resource_calendar_id')
            for calendar_id in resource_calendar_ids:
                res_employee_ids = employee_ids.filtered(lambda e: e.resource_calendar_id.id == calendar_id.id)
                start_dt = fields.Datetime.now()
                stop_dt = start_dt + timedelta(hours=1)
                from_datetime = utc.localize(start_dt).astimezone(timezone(tz or 'UTC'))
                to_datetime = utc.localize(stop_dt).astimezone(timezone(tz or 'UTC'))
                # Getting work interval of the first is working. Functions called on resource_calendar_id
                # are waiting for singleton
                work_interval = res_employee_ids[0].resource_calendar_id._work_intervals_batch(from_datetime, to_datetime)[False]
                # Employee that is not supposed to work have empty items.
                if len(work_interval._items) > 0:
                    # The employees should be working now according to their work schedule
                    working_now += res_employee_ids.ids
        return working_now

```

## File: models\hr_employee_category.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint

from odoo import fields, models


class EmployeeCategory(models.Model):

    _name = "hr.employee.category"
    _description = "Employee Category"

    def _get_default_color(self):
        return randint(1, 11)

    name = fields.Char(string="Tag Name", required=True)
    color = fields.Integer(string='Color Index', default=_get_default_color)
    employee_ids = fields.Many2many('hr.employee', 'employee_category_rel', 'category_id', 'emp_id', string='Employees')

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Tag name already exists!"),
    ]

```

## File: models\hr_employee_public.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools


class HrEmployeePublic(models.Model):
    _name = "hr.employee.public"
    _inherit = ["hr.employee.base"]
    _description = 'Public Employee'
    _order = 'name'
    _auto = False
    _log_access = True # Include magic fields

    # Fields coming from hr.employee.base
    create_date = fields.Datetime(readonly=True)
    name = fields.Char(readonly=True)
    active = fields.Boolean(readonly=True)
    department_id = fields.Many2one(readonly=True)
    job_id = fields.Many2one(readonly=True)
    job_title = fields.Char(readonly=True)
    company_id = fields.Many2one(readonly=True)
    address_id = fields.Many2one(readonly=True)
    mobile_phone = fields.Char(readonly=True)
    work_phone = fields.Char(readonly=True)
    work_email = fields.Char(readonly=True)
    work_contact_id = fields.Many2one(readonly=True)
    work_location_id = fields.Many2one(readonly=True)
    user_id = fields.Many2one(readonly=True)
    resource_id = fields.Many2one(readonly=True)
    tz = fields.Selection(readonly=True)
    color = fields.Integer(readonly=True)

    # Manager-only fields
    is_manager = fields.Boolean(compute='_compute_is_manager')

    employee_id = fields.Many2one('hr.employee', 'Employee', compute="_compute_employee_id", search="_search_employee_id", compute_sudo=True)
    # hr.employee.public specific fields
    child_ids = fields.One2many('hr.employee.public', 'parent_id', string='Direct subordinates', readonly=True)
    image_1920 = fields.Image("Image", related='employee_id.image_1920', compute_sudo=True)
    image_1024 = fields.Image("Image 1024", related='employee_id.image_1024', compute_sudo=True)
    image_512 = fields.Image("Image 512", related='employee_id.image_512', compute_sudo=True)
    image_256 = fields.Image("Image 256", related='employee_id.image_256', compute_sudo=True)
    image_128 = fields.Image("Image 128", related='employee_id.image_128', compute_sudo=True)
    avatar_1920 = fields.Image("Avatar", related='employee_id.avatar_1920', compute_sudo=True)
    avatar_1024 = fields.Image("Avatar 1024", related='employee_id.avatar_1024', compute_sudo=True)
    avatar_512 = fields.Image("Avatar 512", related='employee_id.avatar_512', compute_sudo=True)
    avatar_256 = fields.Image("Avatar 256", related='employee_id.avatar_256', compute_sudo=True)
    avatar_128 = fields.Image("Avatar 128", related='employee_id.avatar_128', compute_sudo=True)
    parent_id = fields.Many2one('hr.employee.public', 'Manager', readonly=True)
    coach_id = fields.Many2one('hr.employee.public', 'Coach', readonly=True)
    user_partner_id = fields.Many2one(related='user_id.partner_id', related_sudo=False, string="User's partner")

    @api.depends_context('uid')
    @api.depends('parent_id')
    def _compute_is_manager(self):
        all_reports = self.env['hr.employee.public'].search([('id', 'child_of', self.env.user.employee_id.id)]).ids
        for employee in self:
            employee.is_manager = employee.id in all_reports

    def _get_manager_only_fields(self):
        return []

    @api.depends_context('uid')
    def _compute_manager_only_fields(self):
        manager_fields = self._get_manager_only_fields()
        for employee in self:
            if employee.is_manager:
                employee_sudo = employee.employee_id.sudo()
                for f in manager_fields:
                    employee[f] = employee_sudo[f]
            else:
                for f in manager_fields:
                    employee[f] = False

    def _search_employee_id(self, operator, value):
        return [('id', operator, value)]

    def _compute_employee_id(self):
        for employee in self:
            employee.employee_id = self.env['hr.employee'].browse(employee.id)

    @api.model
    def _get_fields(self):
        return ','.join('emp.%s' % name for name, field in self._fields.items() if field.store and field.type not in ['many2many', 'one2many'])

    def init(self):
        tools.drop_view_if_exists(self.env.cr, self._table)
        self.env.cr.execute("""CREATE or REPLACE VIEW %s as (
            SELECT
                %s
            FROM hr_employee emp
        )""" % (self._table, self._get_fields()))

```

## File: models\hr_job.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.addons.web_editor.tools import handle_history_divergence


class Job(models.Model):

    _name = "hr.job"
    _description = "Job Position"
    _inherit = ['mail.thread']
    _order = 'sequence'

    active = fields.Boolean(default=True)
    name = fields.Char(string='Job Position', required=True, index='trigram', translate=True)
    sequence = fields.Integer(default=10)
    expected_employees = fields.Integer(compute='_compute_employees', string='Total Forecasted Employees', store=True,
        help='Expected number of employees for this job position after new recruitment.')
    no_of_employee = fields.Integer(compute='_compute_employees', string="Current Number of Employees", store=True,
        help='Number of employees currently occupying this job position.')
    no_of_recruitment = fields.Integer(string='Target', copy=False,
        help='Number of new employees you expect to recruit.', default=1)
    no_of_hired_employee = fields.Integer(string='Hired Employees', copy=False,
        help='Number of hired employees for this job position during recruitment phase.')
    employee_ids = fields.One2many('hr.employee', 'job_id', string='Employees', groups='base.group_user')
    description = fields.Html(string='Job Description', sanitize_attributes=False,
                              default="Perform assigned responsibilities, collaborate with team members, and adhere to company policies. Strong communication, problem-solving, and work ethic required. Adaptability, initiative, and willingness to learn are valued.")
    requirements = fields.Text('Requirements')
    department_id = fields.Many2one('hr.department', string='Department', check_company=True)
    company_id = fields.Many2one('res.company', string='Company', default=lambda self: self.env.company)
    contract_type_id = fields.Many2one('hr.contract.type', string='Employment Type')

    _sql_constraints = [
        ('name_company_uniq', 'unique(name, company_id, department_id)', 'The name of the job position must be unique per department in company!'),
        ('no_of_recruitment_positive', 'CHECK(no_of_recruitment >= 0)', 'The expected number of new employees must be positive.')
    ]

    @api.depends('no_of_recruitment', 'employee_ids.job_id', 'employee_ids.active')
    def _compute_employees(self):
        employee_data = self.env['hr.employee']._read_group([('job_id', 'in', self.ids)], ['job_id'], ['__count'])
        result = {job.id: count for job, count in employee_data}
        for job in self:
            job.no_of_employee = result.get(job.id, 0)
            job.expected_employees = result.get(job.id, 0) + job.no_of_recruitment

    @api.model_create_multi
    def create(self, vals_list):
        """ We don't want the current user to be follower of all created job """
        return super(Job, self.with_context(mail_create_nosubscribe=True)).create(vals_list)

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        self.ensure_one()
        default = dict(default or {})
        if 'name' not in default:
            default['name'] = _("%s (copy)", self.name)
        return super(Job, self).copy(default=default)

    def write(self, vals):
        if len(self) == 1:
            handle_history_divergence(self, 'description', vals)
        return super(Job, self).write(vals)

```

## File: models\hr_work_location.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class WorkLocation(models.Model):
    _name = "hr.work.location"
    _description = "Work Location"
    _order = 'name'

    active = fields.Boolean(default=True)
    name = fields.Char(string="Work Location", required=True)
    company_id = fields.Many2one('res.company', required=True, default=lambda self: self.env.company)
    location_type = fields.Selection([
        ('home', 'Home'),
        ('office', 'Office'),
        ('other', 'Other')], string='Cover Image', default='office', required=True)
    address_id = fields.Many2one('res.partner', required=True, string="Work Address", check_company=True)
    location_number = fields.Char()

```

## File: models\ir_ui_menu.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api, tools


class IrUiMenu(models.Model):
    _inherit = 'ir.ui.menu'

    def _load_menus_blacklist(self):
        res = super()._load_menus_blacklist()
        if self.env.user.has_group('hr.group_hr_user'):
            res.append(self.env.ref('hr.menu_hr_employee').id)
        return res

```

## File: models\mail_activity_plan.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class MailActivityPlan(models.Model):
    _inherit = 'mail.activity.plan'

    department_id = fields.Many2one(
        'hr.department', check_company=True,
        compute='_compute_department_id', ondelete='cascade', readonly=False, store=True)

    @api.constrains('res_model')
    def _check_compatibility_with_model(self):
        """ Check that when the model is updated to a model different from employee,
        there are no remaining specific values to employee. """
        plan_tocheck = self.filtered(lambda plan: plan.res_model != 'hr.employee')
        failing_plans = plan_tocheck.filtered('department_id')
        if failing_plans:
            raise UserError(
                _('Plan %(plan_names)s cannot use a department as it is used only for employee plans.',
                  plan_names=', '.join(failing_plans.mapped('name')))
            )
        failing_templates = plan_tocheck.template_ids.filtered(
            lambda tpl: tpl.responsible_type in {'coach', 'manager', 'employee'}
        )
        if failing_templates:
            raise UserError(
                _('Plan activities %(template_names)s cannot use coach, manager or employee responsible as it is used only for employee plans.',
                  template_names=', '.join(failing_templates.mapped('activity_type_id.name')))
            )

    @api.onchange('res_model')
    def _compute_department_id(self):
        for plan in self.filtered(lambda plan: plan.res_model != 'hr.employee'):
            plan.department_id = False

```

## File: models\mail_activity_plan_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class MailActivityPLanTemplate(models.Model):
    _inherit = 'mail.activity.plan.template'

    responsible_type = fields.Selection(selection_add=[
        ('coach', 'Coach'),
        ('manager', 'Manager'),
        ('employee', 'Employee'),
    ], ondelete={'coach': 'cascade', 'manager': 'cascade', 'employee': 'cascade'})

    @api.constrains('plan_id', 'responsible_type')
    def _check_responsible_hr(self):
        """ Ensure that hr types are used only on employee model """
        for template in self.filtered(lambda tpl: tpl.plan_id.res_model != 'hr.employee'):
            if template.responsible_type in {'coach', 'manager', 'employee'}:
                raise ValidationError(_('Those responsible types are limited to Employee plans.'))

    def _determine_responsible(self, on_demand_responsible, employee):
        if self.plan_id.res_model != 'hr.employee' or self.responsible_type not in {'coach', 'manager', 'employee'}:
            return super()._determine_responsible(on_demand_responsible, employee)
        error = False
        responsible = False
        if self.responsible_type == 'coach':
            if not employee.coach_id:
                error = _('Coach of employee %s is not set.', employee.name)
            responsible = employee.coach_id.user_id
            if employee.coach_id and not responsible:
                error = _("The user of %s's coach is not set.", employee.name)
        elif self.responsible_type == 'manager':
            if not employee.parent_id:
                error = _('Manager of employee %s is not set.', employee.name)
            responsible = employee.parent_id.user_id
            if employee.parent_id and not responsible:
                error = _("The manager of %s should be linked to a user.", employee.name)
        elif self.responsible_type == 'employee':
            responsible = employee.user_id
            if not responsible:
                error = _('The employee %s should be linked to a user.', employee.name)
        if error or responsible:
            return {
                'responsible': responsible,
                'error': error,
            }

```

## File: models\mail_alias.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class Alias(models.Model):
    _inherit = 'mail.alias'

    alias_contact = fields.Selection(selection_add=[
        ('employees', 'Authenticated Employees'),
    ], ondelete={'employees': 'cascade'})

    def _get_alias_contact_description(self):
        if self.alias_contact == 'employees':
            return _('addresses linked to registered employees')
        return super(Alias, self)._get_alias_contact_description()

```

## File: models\models.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, tools, _
from odoo.addons.mail.tools.alias_error import AliasError


class BaseModel(models.AbstractModel):
    _inherit = 'base'

    def _alias_get_error(self, message, message_dict, alias):
        if alias.alias_contact == 'employees':
            email_from = tools.decode_message_header(message, 'From')
            email_address = tools.email_split(email_from)[0]
            employee = self.env['hr.employee'].search([('work_email', 'ilike', email_address)], limit=1)
            if not employee:
                employee = self.env['hr.employee'].search([('user_id.email', 'ilike', email_address)], limit=1)
            if not employee:
                return AliasError('error_hr_employee_restricted', _('restricted to employees'))
            return False
        return super(BaseModel, self)._alias_get_error(message, message_dict, alias)

```

## File: models\resource.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResourceResource(models.Model):
    _inherit = "resource.resource"

    user_id = fields.Many2one(copy=False)
    employee_id = fields.One2many('hr.employee', 'resource_id', check_company=True)

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Company(models.Model):
    _inherit = 'res.company'

    hr_presence_control_email_amount = fields.Integer(string="# emails to send")
    hr_presence_control_ip_list = fields.Char(string="Valid IP addresses")
    employee_properties_definition = fields.PropertiesDefinition('Employee Properties')

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

import threading
from odoo import fields, models, api, _
from odoo.exceptions import ValidationError


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    resource_calendar_id = fields.Many2one(
        'resource.calendar', 'Company Working Hours',
        related='company_id.resource_calendar_id', readonly=False)
    module_hr_presence = fields.Boolean(string="Advanced Presence Control")
    module_hr_skills = fields.Boolean(string="Skills Management")
    module_hr_homeworking = fields.Boolean(string="Remote Work")
    hr_presence_control_login = fields.Boolean(string="Based on user status in system", config_parameter='hr.hr_presence_control_login')
    hr_presence_control_email = fields.Boolean(string="Based on number of emails sent", config_parameter='hr_presence.hr_presence_control_email')
    hr_presence_control_ip = fields.Boolean(string="Based on IP Address", config_parameter='hr_presence.hr_presence_control_ip')
    module_hr_attendance = fields.Boolean(string="Based on attendances")
    hr_presence_control_email_amount = fields.Integer(related="company_id.hr_presence_control_email_amount", readonly=False)
    hr_presence_control_ip_list = fields.Char(related="company_id.hr_presence_control_ip_list", readonly=False)
    hr_employee_self_edit = fields.Boolean(string="Employee Editing", config_parameter='hr.hr_employee_self_edit')

    @api.constrains('module_hr_presence', 'hr_presence_control_email', 'hr_presence_control_ip')
    def _check_advanced_presence(self):
        test_mode = self.env.registry.in_test_mode() or getattr(threading.current_thread(), 'testing', False)
        if self.env.context.get('install_mode', False) or test_mode:
            return

        for settings in self:
            if settings.module_hr_presence and not (settings.hr_presence_control_email or settings.hr_presence_control_ip):
                raise ValidationError(_('You should select at least one Advanced Presence Control option.'))

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class Partner(models.Model):
    _inherit = ['res.partner']

    employee_ids = fields.One2many(
        'hr.employee', 'work_contact_id', string='Employees', groups="hr.group_hr_user",
        help="Related employees based on their private address")
    employees_count = fields.Integer(compute='_compute_employees_count', groups="hr.group_hr_user")

    def _compute_employees_count(self):
        for partner in self:
            partner.employees_count = len(partner.employee_ids.filtered(lambda e: e.company_id in self.env.companies))

    def action_open_employees(self):
        self.ensure_one()
        if self.employees_count > 1:
            return {
                'name': _('Related Employees'),
                'type': 'ir.actions.act_window',
                'res_model': 'hr.employee',
                'view_mode': 'kanban',
                'domain': [('id', 'in', self.employee_ids.ids),
                           ('company_id', 'in', self.env.companies.ids)],
            }
        return {
            'name': _('Employee'),
            'type': 'ir.actions.act_window',
            'res_model': 'hr.employee',
            'res_id': self.employee_ids.id,
            'view_mode': 'form',
        }

    def _get_all_addr(self):
        self.ensure_one()
        employee_id = self.env['hr.employee'].search(
            [('id', 'in', self.employee_ids.ids)],
            limit=1,
        )
        if not employee_id:
            return super()._get_all_addr()

        pstl_addr = {
            'contact_type': 'employee',
            'street': employee_id.private_street,
            'zip': employee_id.private_zip,
            'city': employee_id.private_city,
            'country': employee_id.private_country_id.code,
        }
        return [pstl_addr] + super()._get_all_addr()


class ResPartnerBank(models.Model):
    _inherit = ['res.partner.bank']

    @api.depends_context('uid')
    def _compute_display_name(self):
        account_employee = self.browse()
        if not self.user_has_groups('hr.group_hr_user'):
            account_employee = self.sudo().filtered("partner_id.employee_ids")
            for account in account_employee:
                account.sudo(self.env.su).display_name = \
                    account.acc_number[:2] + "*" * len(account.acc_number[2:-4]) + account.acc_number[-4:]
        super(ResPartnerBank, self - account_employee)._compute_display_name()

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from markupsafe import Markup

from odoo import api, models, fields, _, SUPERUSER_ID
from odoo.exceptions import AccessError
from odoo.tools.misc import clean_context


HR_READABLE_FIELDS = [
    'active',
    'child_ids',
    'employee_id',
    'employee_ids',
    'employee_parent_id',
    'hr_presence_state',
    'last_activity',
    'last_activity_time',
    'can_edit',
    'is_system',
    'employee_resource_calendar_id',
    'work_contact_id',
]

HR_WRITABLE_FIELDS = [
    'additional_note',
    'private_street',
    'private_street2',
    'private_city',
    'private_state_id',
    'private_zip',
    'private_country_id',
    'private_phone',
    'private_email',
    'address_id',
    'barcode',
    'birthday',
    'category_ids',
    'children',
    'coach_id',
    'country_of_birth',
    'department_id',
    'display_name',
    'emergency_contact',
    'emergency_phone',
    'employee_bank_account_id',
    'employee_country_id',
    'gender',
    'identification_id',
    'ssnid',
    'job_title',
    'km_home_work',
    'marital',
    'mobile_phone',
    'employee_parent_id',
    'passport_id',
    'permit_no',
    'pin',
    'place_of_birth',
    'spouse_birthdate',
    'spouse_complete_name',
    'visa_expire',
    'visa_no',
    'work_email',
    'work_location_id',
    'work_phone',
    'certificate',
    'study_field',
    'study_school',
    'private_lang',
    'employee_type',
]


class User(models.Model):
    _inherit = ['res.users']

    def _employee_ids_domain(self):
        # employee_ids is considered a safe field and as such will be fetched as sudo.
        # So try to enforce the security rules on the field to make sure we do not load employees outside of active companies
        return [('company_id', 'in', self.env.company.ids + self.env.context.get('allowed_company_ids', []))]

    # note: a user can only be linked to one employee per company (see sql constraint in ´hr.employee´)
    employee_ids = fields.One2many('hr.employee', 'user_id', string='Related employee', domain=_employee_ids_domain)
    employee_id = fields.Many2one('hr.employee', string="Company employee",
        compute='_compute_company_employee', search='_search_company_employee', store=False)

    job_title = fields.Char(related='employee_id.job_title', readonly=False, related_sudo=False)
    work_phone = fields.Char(related='employee_id.work_phone', readonly=False, related_sudo=False)
    mobile_phone = fields.Char(related='employee_id.mobile_phone', readonly=False, related_sudo=False)
    work_email = fields.Char(related='employee_id.work_email', readonly=False, related_sudo=False)
    category_ids = fields.Many2many(related='employee_id.category_ids', string="Employee Tags", readonly=False, related_sudo=False)
    department_id = fields.Many2one(related='employee_id.department_id', readonly=False, related_sudo=False)
    address_id = fields.Many2one(related='employee_id.address_id', readonly=False, related_sudo=False)
    work_contact_id = fields.Many2one(related='employee_id.work_contact_id', readonly=False, related_sudo=False)
    work_location_id = fields.Many2one(related='employee_id.work_location_id', readonly=False, related_sudo=False)
    employee_parent_id = fields.Many2one(related='employee_id.parent_id', readonly=False, related_sudo=False)
    coach_id = fields.Many2one(related='employee_id.coach_id', readonly=False, related_sudo=False)
    private_street = fields.Char(related='employee_id.private_street', string="Private Street", readonly=False, related_sudo=False)
    private_street2 = fields.Char(related='employee_id.private_street2', string="Private Street2", readonly=False, related_sudo=False)
    private_city = fields.Char(related='employee_id.private_city', string="Private City", readonly=False, related_sudo=False)
    private_state_id = fields.Many2one(
        related='employee_id.private_state_id', string="Private State", readonly=False, related_sudo=False,
        domain="[('country_id', '=?', private_country_id)]")
    private_zip = fields.Char(related='employee_id.private_zip', readonly=False, string="Private Zip", related_sudo=False)
    private_country_id = fields.Many2one(related='employee_id.private_country_id', string="Private Country", readonly=False, related_sudo=False)
    private_phone = fields.Char(related='employee_id.private_phone', readonly=False, related_sudo=False)
    private_email = fields.Char(related='employee_id.private_email', string="Private Email", readonly=False)
    private_lang = fields.Selection(related='employee_id.lang', string="Employee Lang", readonly=False)
    km_home_work = fields.Integer(related='employee_id.km_home_work', readonly=False, related_sudo=False)
    # res.users already have a field bank_account_id and country_id from the res.partner inheritance: don't redefine them
    employee_bank_account_id = fields.Many2one(related='employee_id.bank_account_id', string="Employee's Bank Account Number", related_sudo=False, readonly=False)
    employee_country_id = fields.Many2one(related='employee_id.country_id', string="Employee's Country", readonly=False, related_sudo=False)
    identification_id = fields.Char(related='employee_id.identification_id', readonly=False, related_sudo=False)
    ssnid = fields.Char(related='employee_id.ssnid', readonly=False, related_sudo=False)
    passport_id = fields.Char(related='employee_id.passport_id', readonly=False, related_sudo=False)
    gender = fields.Selection(related='employee_id.gender', readonly=False, related_sudo=False)
    birthday = fields.Date(related='employee_id.birthday', readonly=False, related_sudo=False)
    place_of_birth = fields.Char(related='employee_id.place_of_birth', readonly=False, related_sudo=False)
    country_of_birth = fields.Many2one(related='employee_id.country_of_birth', readonly=False, related_sudo=False)
    marital = fields.Selection(related='employee_id.marital', readonly=False, related_sudo=False)
    spouse_complete_name = fields.Char(related='employee_id.spouse_complete_name', readonly=False, related_sudo=False)
    spouse_birthdate = fields.Date(related='employee_id.spouse_birthdate', readonly=False, related_sudo=False)
    children = fields.Integer(related='employee_id.children', readonly=False, related_sudo=False)
    emergency_contact = fields.Char(related='employee_id.emergency_contact', readonly=False, related_sudo=False)
    emergency_phone = fields.Char(related='employee_id.emergency_phone', readonly=False, related_sudo=False)
    visa_no = fields.Char(related='employee_id.visa_no', readonly=False, related_sudo=False)
    permit_no = fields.Char(related='employee_id.permit_no', readonly=False, related_sudo=False)
    visa_expire = fields.Date(related='employee_id.visa_expire', readonly=False, related_sudo=False)
    additional_note = fields.Text(related='employee_id.additional_note', readonly=False, related_sudo=False)
    barcode = fields.Char(related='employee_id.barcode', readonly=False, related_sudo=False)
    pin = fields.Char(related='employee_id.pin', readonly=False, related_sudo=False)
    certificate = fields.Selection(related='employee_id.certificate', readonly=False, related_sudo=False)
    study_field = fields.Char(related='employee_id.study_field', readonly=False, related_sudo=False)
    study_school = fields.Char(related='employee_id.study_school', readonly=False, related_sudo=False)
    employee_count = fields.Integer(compute='_compute_employee_count')
    hr_presence_state = fields.Selection(related='employee_id.hr_presence_state')
    last_activity = fields.Date(related='employee_id.last_activity')
    last_activity_time = fields.Char(related='employee_id.last_activity_time')
    employee_type = fields.Selection(related='employee_id.employee_type', readonly=False, related_sudo=False)
    employee_resource_calendar_id = fields.Many2one(related='employee_id.resource_calendar_id', string="Employee's Working Hours", readonly=True)

    create_employee = fields.Boolean(store=False, default=False, copy=False, string="Technical field, whether to create an employee")
    create_employee_id = fields.Many2one('hr.employee', store=False, copy=False, string="Technical field, bind user to this employee on create")

    can_edit = fields.Boolean(compute='_compute_can_edit')
    is_system = fields.Boolean(compute="_compute_is_system")

    @api.depends_context('uid')
    def _compute_is_system(self):
        self.is_system = self.env.user._is_system()

    def _compute_can_edit(self):
        can_edit = self.env['ir.config_parameter'].sudo().get_param('hr.hr_employee_self_edit') or self.env.user.has_group('hr.group_hr_user')
        for user in self:
            user.can_edit = can_edit

    @api.depends('employee_ids')
    def _compute_employee_count(self):
        for user in self.with_context(active_test=False):
            user.employee_count = len(user.employee_ids)

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + HR_READABLE_FIELDS + HR_WRITABLE_FIELDS

    @property
    def SELF_WRITEABLE_FIELDS(self):
        return super().SELF_WRITEABLE_FIELDS + HR_WRITABLE_FIELDS

    @api.model
    def get_views(self, views, options=None):
        # Requests the My Profile form view as last.
        # Otherwise the fields of the 'search' view will take precedence
        # and will omit the fields that are requested as SUPERUSER
        # in `get_view()`.
        profile_view = self.env.ref("hr.res_users_view_form_profile")
        profile_form = profile_view and [profile_view.id, 'form']
        if profile_form and profile_form in views:
            views.remove(profile_form)
            views.append(profile_form)
        result = super().get_views(views, options)
        return result

    @api.model
    def get_view(self, view_id=None, view_type='form', **options):
        # When the front-end loads the views it gets the list of available fields
        # for the user (according to its access rights). Later, when the front-end wants to
        # populate the view with data, it only asks to read those available fields.
        # However, in this case, we want the user to be able to read/write its own data,
        # even if they are protected by groups.
        # We make the front-end aware of those fields by sending all field definitions.
        # Note: limit the `sudo` to the only action of "editing own profile" action in order to
        # avoid breaking `groups` mecanism on res.users form view.
        profile_view = self.env.ref("hr.res_users_view_form_profile")
        if profile_view and view_id == profile_view.id:
            self = self.with_user(SUPERUSER_ID)
        result = super(User, self).get_view(view_id, view_type, **options)
        return result

    @api.model_create_multi
    def create(self, vals_list):
        res = super().create(vals_list)
        employee_create_vals = []
        for user, vals in zip(res, vals_list):
            if not vals.get('create_employee') and not vals.get('create_employee_id'):
                continue
            if vals.get('create_employee_id'):
                self.env['hr.employee'].browse(vals.get('create_employee_id')).user_id = user
            else:
                employee_create_vals.append(dict(
                    name=user.name,
                    company_id=user.env.company.id,
                    **self.env['hr.employee']._sync_user(user)
                ))
        if employee_create_vals:
            self.env['hr.employee'].with_context(clean_context(self.env.context)).create(employee_create_vals)
        return res

    def _get_employee_fields_to_sync(self):
        """Get values to sync to the related employee when the User is changed.
        """
        return ['name', 'email', 'image_1920', 'tz']

    def _get_personal_info_partner_ids_to_notify(self, employee):
        # To override in appropriate module
        return ('', [])

    def write(self, vals):
        """
        Synchronize user and its related employee
        and check access rights if employees are not allowed to update
        their own data (otherwise sudo is applied for self data).
        """
        hr_fields = {
            field_name: field
            for field_name, field in self._fields.items()
            if field.related_field and field.related_field.model_name == 'hr.employee' and field_name in vals
        }
        can_edit_self = self.env['ir.config_parameter'].sudo().get_param('hr.hr_employee_self_edit') or self.env.user.has_group('hr.group_hr_user')
        if hr_fields and not can_edit_self:
            # Raise meaningful error message
            raise AccessError(_("You are only allowed to update your preferences. Please contact a HR officer to update other information."))

        employee_domain = [
            *self.env['hr.employee']._check_company_domain(self.env.company),
            ('user_id', 'in', self.ids),
        ]
        if hr_fields:
            employees = self.env['hr.employee'].sudo().search(employee_domain)
            get_field = self.env['ir.model.fields']._get
            field_names = Markup().join([
                 Markup("<li>%s</li>") % get_field("res.users", fname).field_description for fname in hr_fields
            ])
            for employee in employees:
                reason_message, partner_ids = self._get_personal_info_partner_ids_to_notify(employee)
                if partner_ids:
                    employee.message_notify(
                        body=Markup("<p>%s</p><p>%s</p><ul>%s</ul><p><em>%s</em></p>") % (
                            _('Personal information update.'),
                            _("The following fields were modified by %s", employee.name),
                            field_names,
                            reason_message,
                        ),
                        partner_ids=partner_ids,
                    )
        result = super(User, self).write(vals)

        employee_values = {}
        for fname in [f for f in self._get_employee_fields_to_sync() if f in vals]:
            employee_values[fname] = vals[fname]

        if employee_values:
            if 'email' in employee_values:
                employee_values['work_email'] = employee_values.pop('email')
            if 'image_1920' in vals:
                without_image = self.env['hr.employee'].sudo().search(employee_domain + [('image_1920', '=', False)])
                with_image = self.env['hr.employee'].sudo().search(employee_domain + [('image_1920', '!=', False)])
                without_image.write(employee_values)
                if not can_edit_self:
                    employee_values.pop('image_1920')
                with_image.write(employee_values)
            else:
                employees = self.env['hr.employee'].sudo().search(employee_domain)
                if employees:
                    employees.write(employee_values)
        return result

    @api.model
    def action_get(self):
        if self.env.user.employee_id:
            return self.env['ir.actions.act_window']._for_xml_id('hr.res_users_action_my')
        return super(User, self).action_get()

    @api.depends('employee_ids')
    @api.depends_context('company')
    def _compute_company_employee(self):
        employee_per_user = {
            employee.user_id: employee
            for employee in self.env['hr.employee'].search([('user_id', 'in', self.ids), ('company_id', '=', self.env.company.id)])
        }
        for user in self:
            user.employee_id = employee_per_user.get(user)

    def _search_company_employee(self, operator, value):
        return [('employee_ids', operator, value)]

    def action_create_employee(self):
        self.ensure_one()
        self.env['hr.employee'].create(dict(
            name=self.name,
            company_id=self.env.company.id,
            **self.env['hr.employee']._sync_user(self)
        ))

    def action_open_employees(self):
        self.ensure_one()
        employees = self.employee_ids
        model = 'hr.employee' if self.user_has_groups('hr.group_hr_user') else 'hr.employee.public'
        if len(employees) > 1:
            return {
                'name': _('Related Employees'),
                'type': 'ir.actions.act_window',
                'res_model': model,
                'view_mode': 'kanban,tree,form',
                'domain': [('id', 'in', employees.ids)],
            }
        return {
            'name': _('Employee'),
            'type': 'ir.actions.act_window',
            'res_model': model,
            'res_id': employees.id,
            'view_mode': 'form',
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_contract_type
from . import hr_employee_base
from . import hr_employee
from . import hr_employee_category
from . import hr_employee_public
from . import hr_department
from . import hr_departure_reason
from . import hr_job
from . import hr_work_location
from . import mail_activity_plan
from . import mail_activity_plan_template
from . import mail_alias
from . import discuss_channel
from . import models
from . import res_config_settings
from . import res_users
from . import res_company
from . import res_partner
from . import resource
from . import ir_ui_menu

```

## File: populate\hr.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import models
from odoo.tools import populate

class HrDepartment(models.Model):
    _inherit = 'hr.department'

    _populate_sizes = {'small': 5, 'medium': 30, 'large': 200}

    def _populate_factories(self):
        return [
            ('name', populate.constant('department_{counter}')),
        ]

    def _populate(self, size):
        departments = super()._populate(size)
        self._populate_set_parent_departments(departments, size)
        return departments

    def _populate_set_parent_departments(self, departments, size):
        parent_ids = []
        rand = populate.Random('hr.department+parent_generator')

        for dept in departments:
            if rand.random() > 0.3:
                parent_ids.append(dept.id)

        parent_children = defaultdict(lambda: self.env['hr.department'])
        for dept in departments:
            parent = rand.choice(parent_ids)
            if parent < dept.id:
                parent_children[parent] |= dept

        for parent, children in parent_children.items():
            children.write({'parent_id': parent})

class HrJob(models.Model):
    _inherit = 'hr.job'

    _populate_sizes = {'small': 5, 'medium': 20, 'large': 100}
    _populate_dependencies = ['hr.department']

    def _populate_factories(self):
        department_ids = self.env.registry.populated_models['hr.department']

        return [
            ('name', populate.constant('job_{counter}')),
            ('department_id', populate.randomize(department_ids)),
        ]

class HrWorkLocation(models.Model):
    _inherit = 'hr.work.location'

    _populate_sizes = {'small': 2, 'medium': 5, 'large': 20}
    def _populate_factories(self):
        address_id = self.env.ref('base.main_partner').id

        return [
            ('name', populate.constant('work_location_{counter}')),
            ('address_id', populate.constant(address_id)),
        ]

class HrEmployeeCategory(models.Model):
    _inherit = 'hr.employee.category'

    _populate_sizes = {'small': 10, 'medium': 50, 'large': 200}

    def _populate_factories(self):
        return [
            ('name', populate.constant('tag_{counter}')),
        ]

class HrEmployee(models.Model):
    _inherit = 'hr.employee'

    _populate_sizes = {'small': 100, 'medium': 2000, 'large': 20000}
    _populate_dependencies = ['res.company', 'res.users', 'resource.calendar', 'hr.department',
                              'hr.job', 'hr.work.location', 'hr.employee.category']

    def _populate(self, size):
        employees = super()._populate(size)
        self._populate_set_manager(employees)
        return employees

    def _populate_factories(self):
        company_ids = self.env['res.company'].browse(self.env.registry.populated_models['res.company'])
        company_calendars = {}
        for company_id in company_ids:
            company_calendars[company_id.id] = company_id.resource_calendar_ids.filtered_domain([
                         ('name', 'not like', 'Standard')]).ids

        department_ids = self.env.registry.populated_models['hr.department']
        job_ids = self.env.registry.populated_models['hr.job']
        work_location_ids = self.env.registry.populated_models['hr.work.location']
        tag_ids = self.env.registry.populated_models['hr.employee.category']
        user_ids = self.env['res.users'].browse(self.env.registry.populated_models['res.users'])

        def _compute_user_and_company(iterator, *args):
            # First users
            for values, user_id in zip(iterator, user_ids):
                yield {'company_id': user_id.company_id.id,
                       'user_id': user_id.id,
                       **values}
            # then as many as required non - users
            for values in iterator:
                yield {'company_id': populate.random.choice(company_ids).id,
                       'user_id': False,
                       **values}

        def get_resource_calendar_id(values, random, **kwargs):
            return random.choice(company_calendars[values['company_id']])

        def get_tag_ids(values, counter, random):
            return [
                (6, 0, [
                    random.choice(tag_ids) for i in range(random.randint(0, 6))
                ])
            ]

        return [
            ('active', populate.iterate([True, False], [0.9, 0.1])),
            ('name', populate.constant("employee_{counter}")),
            ('_user_and_company', _compute_user_and_company),
            ('department_id', populate.randomize(department_ids)),
            ('job_id', populate.randomize(job_ids)),
            ('work_location_id', populate.randomize(work_location_ids)),
            ('category_ids', populate.compute(get_tag_ids)),
            ('resource_calendar_id', populate.compute(get_resource_calendar_id)),
        ]

    def _populate_set_manager(self, employees):
        manager_ids = defaultdict(list)
        rand = populate.Random('hr.employee+manager_generator')

        for employee in employees:
            # 15% of employees are managers, at least one per company
            if rand.random() >= 0.85 or not manager_ids.get(employee.company_id):
                manager_ids[employee.company_id].append(employee.id)

        for employee in employees:
            manager = rand.choice(manager_ids[employee.company_id])
            if manager != employee.id:
                employee.parent_id = manager

```

## File: populate\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr

```

## File: report\hr_employee_badge.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_employee_print_badge" model="ir.actions.report">
        <field name="name">Print Badge</field>
        <field name="model">hr.employee</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">hr.print_employee_badge</field>
        <field name="report_file">hr.print_employee_badge</field>
        <field name="print_report_name">'Badge - %s' % (object.name).replace('/', '')</field>
        <field name="binding_model_id" ref="model_hr_employee"/>
        <field name="binding_type">report</field>
    </record>

    <template id="print_employee_badge">
        <t t-call="web.basic_layout">
            <div class="page">
                <div class="oe_structure"></div>
                <t t-foreach="docs" t-as="employee">
                    <div class="col-md-6">
                        <div class="oe_structure"></div>
                        <table style="width:243pt; height:153pt; border: 1pt solid black; border-collapse:separate; border-radius:8pt; margin:5pt">
                            <td style="width:33%;" valign="center">
                                <table style="width:77pt; height:150pt" class="table-borderless">
                                    <tr style="height:30%">
                                        <td align="center" valign="center">
                                            <span t-if="employee.company_id.logo">
                                                <img t-att-src="image_data_uri(employee.company_id.logo)" style="max-height:45pt;max-width:90%" alt="Company Logo"/>
                                            </span>
                                        </td>
                                    </tr>
                                    <tr style="height:70%;">
                                        <td align="center" valign="center">
                                            <img t-att-src="image_data_uri(employee.avatar_1920)" style="max-height:85pt;max-width:90%" alt="Employee Image"/>
                                        </td>
                                    </tr>
                                </table>
                            </td>
                            <td style="width:67%" valign="center">
                                <table style="width:155pt; height:85pt" class="table-borderless">
                                    <tr><th><div style="font-size:15pt; margin-bottom:0pt;margin-top:0pt;" align="center"><span t-out="employee.name" data-oe-demo="Marc Demo"/></div></th></tr>
                                    <tr><td><div align="center" style="font-size:10pt;margin-bottom:5pt;"><span t-out="employee.job_id.name" data-oe-demo="Software Developer"/></div></td></tr>
                                    <tr><td><div t-if="employee.barcode" t-field="employee.barcode" t-options="{'widget': 'barcode', 'width': 600, 'height': 120, 'img_style': 'max-height:50pt;max-width:100%;', 'img_align': 'center'}" data-oe-demo="12345678901"/></td></tr>
                                </table>
                            </td>
                        </table>
                        <div class="oe_structure"></div>
                    </div>
                </t>
                <div class="oe_structure"></div>
            </div>
        </t>
    </template>
</odoo>

```

## File: security\hr_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="base.module_category_human_resources_employees" model="ir.module.category">
        <field name="sequence">9</field>
    </record>

    <record id="group_hr_user" model="res.groups">
        <field name="name">Officer: Manage all employees</field>
        <field name="category_id" ref="base.module_category_human_resources_employees"/>
        <field name="implied_ids" eval="[(6, 0, [ref('base.group_user')])]"/>
        <field name="comment">The user will be able to approve document created by employees.</field>
    </record>

    <record id="group_hr_manager" model="res.groups">
        <field name="name">Administrator</field>
        <field name="comment">The user will have access to the human resources configuration as well as statistic reports.</field>
        <field name="category_id" ref="base.module_category_human_resources_employees"/>
        <field name="implied_ids" eval="[(4, ref('group_hr_user'))]"/>
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
    </record>

<data noupdate="1">
    <record id="base.default_user" model="res.users">
        <field name="groups_id" eval="[(4,ref('group_hr_manager'))]"/>
    </record>

    <record id="hr_employee_comp_rule" model="ir.rule">
        <field name="name">Employee multi company rule</field>
        <field name="model_id" ref="model_hr_employee"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="hr_dept_comp_rule" model="ir.rule">
        <field name="name">Department multi company rule</field>
        <field name="model_id" ref="model_hr_department"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="hr_employee_public_comp_rule" model="ir.rule">
        <field name="name">Employee multi company rule</field>
        <field name="model_id" ref="model_hr_employee_public"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="hr_job_comp_rule" model="ir.rule">
        <field name="name">Job multi company rule</field>
        <field name="model_id" ref="model_hr_job"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="ir_rule_res_partner_bank_internal_users" model="ir.rule">
        <field name="name">HR: Prevent non HR officers from accessing employee bank accounts</field>
        <field name="model_id" ref="base.model_res_partner_bank"/>
        <field name="domain_force">[('partner_id.employee_ids', '=', False)]</field>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record id="ir_rule_res_partner_bank_employees" model="ir.rule">
        <field name="name">HR: Allow HR officers from accessing employee bank accounts</field>
        <field name="model_id" ref="base.model_res_partner_bank"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('hr.group_hr_user'))]"/>
    </record>

    <record id="ir_rule_hr_contract_type_multi_company" model="ir.rule">
        <field name="name">HR Contract Type: Multi Company</field>
        <field name="model_id" ref="model_hr_contract_type"/>
        <field name="domain_force">['|', ('country_id', '=', False), ('country_id', 'in', user.env.companies.country_id.ids)]</field>
    </record>

    <record id="mail_plan_rule_group_hr_manager" model="ir.rule">
        <field name="name">Manager can edit employee plan</field>
        <field name="groups" eval="[(4, ref('group_hr_manager'))]"/>
        <field name="model_id" ref="mail.model_mail_activity_plan"/>
        <field name="domain_force">[('res_model', '=', 'hr.employee')]</field>
        <field name="perm_read" eval="False"/>
    </record>

    <record id="mail_plan_templates_rule_group_hr_manager" model="ir.rule">
        <field name="name">Manager can edit employee plan template</field>
        <field name="groups" eval="[(4, ref('group_hr_manager'))]"/>
        <field name="model_id" ref="mail.model_mail_activity_plan_template"/>
        <field name="domain_force">[('plan_id.res_model', '=', 'hr.employee')]</field>
        <field name="perm_read" eval="False"/>
    </record>

</data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_employee_category_user,hr.employee.category.user,model_hr_employee_category,group_hr_user,1,1,1,1
access_hr_employee_category_emp,hr.employee.category.emp,model_hr_employee_category,base.group_user,1,0,0,0
access_hr_employee_user,hr.employee user,model_hr_employee,group_hr_user,1,1,1,1
access_hr_employee_system_user,hr.employee system user,model_hr_employee,base.group_system,1,0,0,0
access_hr_employee_public_user,hr.employee_public,model_hr_employee_public,base.group_user,1,0,0,0
access_hr_employee_resource_user,resource.resource.user,resource.model_resource_resource,group_hr_user,1,1,1,1
access_hr_department_user,hr.department.user,model_hr_department,group_hr_user,1,1,1,1
access_hr_department_employee,hr.department.employee,model_hr_department,base.group_user,1,0,0,0
access_hr_job_user,hr.job user,model_hr_job,group_hr_user,1,1,1,1
access_hr_departure_wizard,access.hr.departure.wizard,model_hr_departure_wizard,hr.group_hr_user,1,1,1,0
access_hr_work_location_user,access_hr_work_location_user,model_hr_work_location,base.group_user,1,0,0,0
access_hr_work_location_manager,access_hr_work_location_manager,model_hr_work_location,group_hr_manager,1,1,1,1
access_hr_departure_reason,access_hr_departure_reason_user,model_hr_departure_reason,group_hr_user,1,1,1,1
access_hr_contract_type_manager,hr.contract.type.manager,model_hr_contract_type,hr.group_hr_user,1,1,1,1
access_mail_activity_plan_hr_manager,mail.activity.plan.hr.manager,mail.model_mail_activity_plan,hr.group_hr_manager,1,1,1,1
access_mail_activity_plan_template_hr_manager,mail.activity.plan.template.hr.manager,mail.model_mail_activity_plan_template,hr.group_hr_manager,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M34 17a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z" fill="#985184"/><path d="M12 24a4 4 0 1 1-8 0 4 4 0 0 1 8 0Z" fill="#FBB945"/><path d="M46 24a4 4 0 1 1-8 0 4 4 0 0 1 8 0Z" fill="#1AD3BB"/><path d="M25 30H4a4 4 0 0 0-4 4v4a4 4 0 0 0 4 4h21V30Z" fill="#FBB945"/><path d="M46 30H25v12h21a4 4 0 0 0 4-4v-4a4 4 0 0 0-4-4Z" fill="#1AD3BB"/><path d="M12 30h14c6.627 0 12 5.373 12 12H24c-6.627 0-12-5.373-12-12Z" fill="#985184"/></svg>

```

## File: static\src\messaging_service_patch.js

```javascript
/** @odoo-module */

import { Messaging } from "@mail/core/common/messaging_service";

import { patch } from "@web/core/utils/patch";

patch(Messaging.prototype, {
    setup(...args) {
        super.setup(...args);
        this.store.employees = {};
    },
});

```

## File: static\src\thread_service_patch.js

```javascript
/** @odoo-module */

import { _t } from "@web/core/l10n/translation";
import { ThreadService } from "@mail/core/common/thread_service";
import { patch } from "@web/core/utils/patch";

/** @type {import("@mail/core/common/thread_service").ThreadService} */
const threadServicePatch = {
    async getChat(person) {
        const { employeeId } = person;
        if (!employeeId) {
            return super.getChat(person);
        }
        let employee = this.store.employees[employeeId];
        if (!employee) {
            this.store.employees[employeeId] = { id: employeeId };
            employee = this.store.employees[employeeId];
        }
        if (!employee.user_id && !employee.hasCheckedUser) {
            employee.hasCheckedUser = true;
            const [employeeData] = await this.orm.silent.read(
                "hr.employee.public",
                [employee.id],
                ["user_id", "user_partner_id"],
                {
                    context: { active_test: false },
                }
            );
            if (employeeData) {
                employee.user_id = employeeData.user_id[0];
                let user = this.store.users[employee.user_id];
                if (!user) {
                    this.store.users[employee.user_id] = { id: employee.user_id };
                    user = this.store.users[employee.user_id];
                }
                user.partner_id = employeeData.user_partner_id[0];
                this.store.Persona.insert({
                    displayName: employeeData.user_partner_id[1],
                    id: employeeData.user_partner_id[0],
                    type: "partner",
                });
            }
        }
        if (!employee.user_id) {
            this.notificationService.add(
                _t("You can only chat with employees that have a dedicated user."),
                { type: "info" }
            );
            return;
        }
        return super.getChat({ userId: employee.user_id });
    },
};

patch(ThreadService.prototype, threadServicePatch);

```

## File: static\src\components\avatar_card\avatar_card_popover_patch.js

```javascript
/* @odoo-module */

import { patch } from "@web/core/utils/patch";
import { AvatarCardPopover } from "@mail/discuss/web/avatar_card/avatar_card_popover";
import { useService } from "@web/core/utils/hooks";

export const patchAvatarCardPopover = {
    setup() {
        super.setup();
        this.userInfoTemplate = "hr.avatarCardUserInfos",
        this.actionService = useService("action");
    },
    get fieldNames(){
        let fields = super.fieldNames;
        return fields.concat([
            "work_phone",
            "work_email", 
            "job_title", 
            "department_id", 
            "employee_parent_id",
            "employee_ids",
        ])
    },
    get email(){
        return this.user.work_email || this.user.email;
    },
    get phone(){
        return this.user.work_phone || this.user.phone;
    },
    async onClickViewEmployee(){
        const employeeId = this.user.employee_ids[0];
        const action = await this.orm.call('hr.employee', 'get_formview_action', [employeeId]);
        this.actionService.doAction(action); 
    }
};

export const unpatchAvatarCardPopover = patch(AvatarCardPopover.prototype, patchAvatarCardPopover);

```

## File: static\src\components\avatar_card\avatar_card_popover_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.AvatarCardPopover" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('o_avatar_card_buttons')]" position="inside">
            <button class="btn btn-secondary btn-sm" t-if="user.employee_ids?.length > 0" t-on-click.stop="onClickViewEmployee">View profile</button>
        </xpath>
    </t>

    <t t-name="hr.avatarCardUserInfos">
        <small class="text-muted text-truncate" t-if="user.job_title" t-att-title="user.job_title" t-esc="user.job_title"/>
        <span class="text-muted text-truncate" t-if="user.department_id" data-tooltip="Department" t-att-title="user.department_id[1]" t-esc="user.department_id[1]"/>
    </t>
</templates>

```

## File: static\src\components\background_image\background_image.js

```javascript
/** @odoo-module */

import { registry } from '@web/core/registry';

import { ImageField, imageField } from '@web/views/fields/image/image_field';

export class BackgroundImageField extends ImageField {}
BackgroundImageField.template = 'hr.BackgroundImage';

export const backgroundImageField = {
    ...imageField,
    component: BackgroundImageField,
};

registry.category("fields").add("background_image", backgroundImageField);

```

## File: static\src\components\background_image\background_image.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="hr.BackgroundImage">
        <img
            loading="lazy"
            t-att-data-tooltip-template="hasTooltip and tooltipAttributes.template"
            t-att-data-tooltip-info="hasTooltip and tooltipAttributes.info"
            t-att-data-tooltip-delay="hasTooltip and props.zoomDelay"
            t-attf-src="#{getUrl(props.previewImage or props.name)}"
            alt="Binary file"
           />
    </t>
</templates>

```

## File: static\src\components\department_chart\department_chart.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";

import { standardWidgetProps } from "@web/views/widgets/standard_widget_props";
import { onWillStart, useState, onWillUpdateProps, Component } from "@odoo/owl";

export class DepartmentChart extends Component {
    setup() {
        super.setup();

        this.action = useService("action");
        this.orm = useService("orm");
        this.state = useState({
            hierarchy: {},
        });
        onWillStart(async () => await this.fetchHierarchy(this.props.record.resId));

        onWillUpdateProps(async (nextProps) => {
            await this.fetchHierarchy(nextProps.record.resId);
        });
    }

    async fetchHierarchy(departmentId) {
        this.state.hierarchy = await this.orm.call("hr.department", "get_department_hierarchy", [
            departmentId,
        ]);
    }

    openDepartmentEmployees(departmentId) {
        this.action.doAction("hr.act_employee_from_department", {
            additionalContext: {
                active_id: departmentId,
            },
        });
    }
}
DepartmentChart.template = "hr.DepartmentChart";
DepartmentChart.props = {
    ...standardWidgetProps,
};

export const departmentChart = {
    component: DepartmentChart,
};
registry.category("view_widgets").add("hr_department_chart", departmentChart);

```

## File: static\src\components\department_chart\department_chart.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="hr.DepartmentChart">
        <div class="o_hr_department_chart">
            <div class="o_horizontal_separator mb-3 text-uppercase fw-bolder small">Department Organization</div>
            <t t-if="state.hierarchy.self">
                <div class="o_hr_department_chart_parent">
                    <t t-set="dept" t-value="state.hierarchy.parent"/>
                    <t t-set="hideTree" t-value="true"/>
                    <t t-call="hr.DepartmentChart.Department"/>
                </div>

                <div t-att-class="state.hierarchy.parent?'ms-4':''">
                    <div class="o_hr_department_chart_self">
                        <t t-set="dept" t-value="state.hierarchy.self"/>
                        <t t-set="hideTree" t-value="!state.hierarchy.parent"/>
                        <t t-call="hr.DepartmentChart.Department"/>
                    </div>

                    <t t-set="hideTree" t-value="false"/>
                    <div class="o_hr_department_chart_children ms-4">
                        <t t-foreach="state.hierarchy.children" t-as="dept" t-key="dept.name">
                            <t t-call="hr.DepartmentChart.Department"/>
                        </t>
                    </div>
                </div>
            </t>
        </div>
    </t>

    <t t-name="hr.DepartmentChart.Department">
        <t t-if="dept">
            <div t-attf-class="#{hideTree?'':'o_treeEntry'} mb-0 ps-4">
                <div class="d-flex justify-content-between">
                    <span class="department_name ms-2">
                        <t t-esc="dept.name"/>
                    </span>
                    <a class="badge rounded-pill bg-300 border px-2" role="button"
                        title="Show employees"
                        t-on-click.prevent="() => this.openDepartmentEmployees(dept.id)">
                        <t t-esc="dept.employees"/>
                    </a>
                </div>
            </div>
        </t>
    </t>
</templates>

```

## File: static\src\components\employee_chat\employee_chat.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { standardWidgetProps } from "@web/views/widgets/standard_widget_props";

import { useOpenChat } from "@mail/core/web/open_chat_hook";
import { Component } from "@odoo/owl";

export class HrEmployeeChat extends Component {
    setup() {
        super.setup();
        this.openChat = useOpenChat(this.props.record.resModel);
    }
}
HrEmployeeChat.props = {
    ...standardWidgetProps,
};
HrEmployeeChat.template = "hr.OpenChat";

export const hrEmployeeChat = {
    component: HrEmployeeChat,
};
registry.category("view_widgets").add("hr_employee_chat", hrEmployeeChat);

```

## File: static\src\components\employee_chat\employee_chat.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="hr.OpenChat">
        <a t-if="props.record.data.user_id and props.record.resId"
            title="Chat"
            icon="fa-comments"
            t-on-click.prevent="() => openChat(props.record.resId)"
            href="#"
            class="ml8 o_employee_chat_btn"
            role="button">
            <i class="fa fa-comments align-middle fs-6"/>
        </a>
    </t>
</templates>

```

## File: static\src\components\float_without_trailing_zeros\float_without_trailing_zeros.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { floatField, FloatField } from "@web/views/fields/float/float_field";

const fieldRegistry = registry.category("fields");

class FloatWithoutTrailingZeros extends FloatField {
    get formattedValue() {
        return super.formattedValue.replace(/\.0+$/, "");
    }
}

const floatWithoutTrailingZeros = { ...floatField, component: FloatWithoutTrailingZeros };

fieldRegistry.add("float_without_trailing_zeros", floatWithoutTrailingZeros);

```

## File: static\src\components\hr_presence_status\hr_presence_status.js

```javascript
/** @odoo-module */

import { Component } from "@odoo/owl";

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { standardFieldProps } from "@web/views/fields/standard_field_props";

export class HrPresenceStatus extends Component {
    static template = "hr.HrPresenceStatus";
    static props = {
        ...standardFieldProps,
        tag: { type: String, optional: true },
    };
    static defaultProps = {
        tag: "small",
    };

    get classNames() {
        const classNames = ["fa"];
        classNames.push(
            this.icon,
            "fa-fw",
            "o_button_icon",
            "hr_presence",
            "align-middle",
            this.color,
        )
        return classNames.join(" ");
    }

    get color() {
        switch (this.value) {
            case "presence_present":
            case "presence_absent_active":
                return "text-success";
            case "presence_absent":
                return "text-muted";
            case "presence_to_define":
                return "text-warning";
            default:
                return "";
        }
    }

    get icon() {
        return `fa-circle${this.value.startsWith("presence_absent") ? "-o" : ""}`;
    }

    get label() {
        return this.value !== false
            ? this.options.find(([value, label]) => value === this.value)[1]
            : "";
    }

    get options() {
        return this.props.record.fields[this.props.name].selection.filter(
            (option) => option[0] !== false && option[1] !== ""
        );
    }

    get value() {
        return this.props.record.data[this.props.name];
    }
}

export const hrPresenceStatus = {
    component: HrPresenceStatus,
    displayName: _t("HR Presence Status"),
    extractProps({ viewType }, dynamicInfo) {
        return {
            tag: viewType === "kanban" ? "span" : "small",
        };
    },
};

registry.category("fields").add("hr_presence_status", hrPresenceStatus)

```

## File: static\src\components\hr_presence_status\hr_presence_status.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="hr.HrPresenceStatus">
        <t t-tag="props.tag" role="img" t-att-class="classNames" t-att-aria-label="label" t-att-title="label"/>
    </t>

</templates>

```

## File: static\src\components\hr_presence_status_private\hr_presence_status_private.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { HrPresenceStatus, hrPresenceStatus } from "../hr_presence_status/hr_presence_status";

export class HrPresenceStatusPrivate extends HrPresenceStatus { }

export const hrPresenceStatusPrivate = {
    ...hrPresenceStatus,
    component: HrPresenceStatusPrivate,
};

registry.category("fields").add("hr_presence_status_private", hrPresenceStatusPrivate);

```

## File: static\src\components\radio_image_field\radio_image_field.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { RadioField, preloadRadio, radioField } from "@web/views/fields/radio/radio_field";

class RadioImageField extends RadioField {}
RadioImageField.template = "hr_homeworking.RadioImageField";

registry.category("fields").add("hr_homeworking_radio_image", {
    ...radioField,
    component: RadioImageField,
});

registry.category("preloadedData").add("hr_homeworking_radio_image", {
    loadOnTypes: ["many2one"],
    preload: preloadRadio,
});

```

## File: static\src\components\radio_image_field\radio_image_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="hr_homeworking.RadioImageField">
        <div role="radiogroup" class="d-flex flex-wrap" t-att-aria-label="string">
            <t t-foreach="items" t-as="item" t-key="item[0]">
                <t t-if="['office', 'home', 'other'].includes(item[0])">
                    <div class="form-check o_radio_item me-1" aria-atomic="true">
                        <input
                            type="radio"
                            class="form-check-input o_radio_input"
                            t-att-checked="item[0] === value"
                            t-att-disabled="props.readonly"
                            t-att-name="id"
                            t-att-data-value="item[0]"
                            t-att-data-index="item_index"
                            t-att-id="`${id}_${item[0]}`"
                            t-on-change="() => this.onChange(item)"
                        />
                        <t t-if="item[0] === 'office'">
                            <i class="fa fa-building-o fa-2x" role="img"/>
                        </t>
                        <t t-elif="item[0] === 'home'">
                            <i class="fa fa-home fa-2x" role="img"/>
                        </t>
                        <t t-else="">
                            <i class="fa fa-map-marker fa-2x" role="img"/>
                        </t>
                    </div>
                </t>
            </t>
        </div>
    </t>


</templates>

```

## File: static\src\components\work_permit_upload\work_permit_upload.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { BinaryField, binaryField } from "@web/views/fields/binary/binary_field";

export class WorkPermitUploadField extends BinaryField {}
WorkPermitUploadField.template = "hr.WorkPermitUploadField";

export const workPermitUploadField = {
    ...binaryField,
    component: WorkPermitUploadField,
};

registry.category("fields").add("work_permit_upload", workPermitUploadField);

```

## File: static\src\components\work_permit_upload\work_permit_upload.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="hr.WorkPermitUploadField" t-inherit="web.BinaryField" t-inherit-mode="primary">
        <xpath expr="//label[hasclass('o_select_file_button')]" position="attributes">
            <attribute name="class" remove="btn-primary" add="btn-secondary" separator=" " />
        </xpath>
    </t>

</templates>

```

## File: static\src\core\common\persona_model_patch.js

```javascript
/* @odoo-module */

import { Persona } from "@mail/core/common/persona_model";

import { patch } from "@web/core/utils/patch";

patch(Persona.prototype, {
    employeeId: undefined,
});

```

## File: static\src\core\web\thread_actions.js

```javascript
/* @odoo-module */

import { threadActionsRegistry } from "@mail/core/common/thread_actions";
import { _t } from "@web/core/l10n/translation";
import { useComponent } from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";

threadActionsRegistry.add("open-hr-profile", {
    condition(component) {
        return (
            component.thread?.type === "chat" &&
            component.props.chatWindow?.isOpen &&
            component.thread.chatPartner.employeeId
        );
    },
    icon: "fa fa-fw fa-id-card",
    name: _t("Open Profile"),
    async open(component) {
        component.actionService.doAction({
            type: "ir.actions.act_window",
            res_id: component.thread.correspondent.employeeId,
            res_model: "hr.employee.public",
            views: [[false, "form"]],
        });
    },
    async setup(action) {
        const component = useComponent();
        const orm = useService("orm");
        let employeeId;
        if (!component.thread?.correspondent?.employeeId && component.thread?.chatPartner) {
            const employees = await orm.silent.searchRead(
                "hr.employee",
                [["user_partner_id", "=", component.thread.chatPartner.id]],
                ["id"]
            );
            employeeId = employees[0]?.id;
            if (employeeId) {
                component.thread.chatPartner.employeeId = employeeId;
            }
        }
    },
    sequence: 16,
});

```

## File: static\src\img\icons\hatched.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><path d="M83.13,35.69a7.9,7.9,0,0,0-4.94-7.33L34.33,72.22H47.06L83.13,36.15Z" style="fill:#875b7b"/><path d="M53.39,27.78,16.87,64.3h0a7.9,7.9,0,0,0,5.21,7.43l44-44Z" style="fill:#875b7b"/><path d="M24.78,27.78a7.91,7.91,0,0,0-7.91,7.91V51.57L40.66,27.78Z" style="fill:#875b7b"/><path d="M59.78,72.22H75.22a7.91,7.91,0,0,0,7.91-7.91V48.87Z" style="fill:#875b7b"/></svg>
```

## File: static\src\img\icons\hatched_dark.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><path d="M83.13,35.69a7.9,7.9,0,0,0-4.94-7.33L34.33,72.22H47.06L83.13,36.15Z" style="fill:#bb86fc"/><path d="M53.39,27.78,16.87,64.3h0a7.9,7.9,0,0,0,5.21,7.43l44-44Z" style="fill:#bb86fc"/><path d="M24.78,27.78a7.91,7.91,0,0,0-7.91,7.91V51.57L40.66,27.78Z" style="fill:#bb86fc"/><path d="M59.78,72.22H75.22a7.91,7.91,0,0,0,7.91-7.91V48.87Z" style="fill:#bb86fc"/></svg>

```

## File: static\src\img\icons\line.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><path d="M21.93,55H78.07a5,5,0,0,0,0-10H21.93a5,5,0,0,0,0,10Z" style="fill:#875b7b"/></svg>
```

## File: static\src\img\icons\line_dark.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><path d="M21.93,55H78.07a5,5,0,0,0,0-10H21.93a5,5,0,0,0,0,10Z" style="fill:#bb86fc"/></svg>

```

## File: static\src\img\icons\plain.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><rect x="16.87" y="27.78" width="66.26" height="44.44" rx="7.91" style="fill:#875b7b"/></svg>
```

## File: static\src\img\icons\plain_dark.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><rect x="16.87" y="27.78" width="66.26" height="44.44" rx="7.91" style="fill:#bb86fc"/></svg>

```

## File: static\src\user_menu\my_profile.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { preferencesItem } from "@web/webclient/user_menu/user_menu_items";

export function hrPreferencesItem(env)  {
    return Object.assign(
        {}, 
        preferencesItem(env),
        {
            description: _t('My Profile'),
        }
    );
}

registry.category("user_menuitems").add('profile', hrPreferencesItem, { force: true })

```

## File: static\src\views\archive_employee_hook.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { useService } from "@web/core/utils/hooks";
import { useComponent } from "@odoo/owl";

export function useArchiveEmployee() {
    const component = useComponent();
    const action = useService("action");
    return (id) => {
        action.doAction(
            {
                type: "ir.actions.act_window",
                name: _t("Employee Termination"),
                res_model: "hr.departure.wizard",
                views: [[false, "form"]],
                view_mode: "form",
                target: "new",
                context: {
                    active_id: id,
                    toggle_active: true,
                },
            },
            {
                onClose: async () => {
                    await component.model.load();
                },
            }
        );
    };
}

```

## File: static\src\views\form_view.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";

import { formView } from "@web/views/form/form_view";
import { FormController } from "@web/views/form/form_controller";

import { useArchiveEmployee } from "@hr/views/archive_employee_hook";

export class EmployeeFormController extends FormController {
    setup() {
        super.setup();
        this.archiveEmployee = useArchiveEmployee();
    }

    getStaticActionMenuItems() {
        const menuItems = super.getStaticActionMenuItems();
        menuItems.archive.callback = this.archiveEmployee.bind(this, this.model.root.resId);
        return menuItems;
    }
}

registry.category("views").add("hr_employee_form", {
    ...formView,
    Controller: EmployeeFormController,
});

```

## File: static\src\views\list_view.js

```javascript
/** @odoo-module */

import { registry } from '@web/core/registry';

import { listView } from '@web/views/list/list_view';
import { ListController } from '@web/views/list/list_controller';

import { useArchiveEmployee } from '@hr/views/archive_employee_hook';

export class EmployeeListController extends ListController {
    setup() {
        super.setup();
        this.archiveEmployee = useArchiveEmployee();
    }

    getStaticActionMenuItems() {
        const menuItems = super.getStaticActionMenuItems();
        const selectedRecords = this.model.root.selection;

        // Only override the Archive action when only 1 record is selected.
        if (selectedRecords.length === 1 && selectedRecords[0].data.active) {
            menuItems.archive.callback = this.archiveEmployee.bind(this, selectedRecords[0].resId);
        }
        return menuItems;
    }
}

registry.category('views').add('hr_employee_list', {
    ...listView,
    Controller: EmployeeListController,
});

```

## File: static\src\views\open_chat_hook.js

```javascript
/** @odoo-module **/

import { helpers } from "@mail/core/web/open_chat_hook";
import { patch } from "@web/core/utils/patch";

patch(helpers, {
    SUPPORTED_M2X_AVATAR_MODELS: [
        ...helpers.SUPPORTED_M2X_AVATAR_MODELS,
        "hr.employee",
        "hr.employee.public",
    ],
    buildOpenChatParams(resModel, id) {
        if (["hr.employee", "hr.employee.public"].includes(resModel)) {
            return { employeeId: id };
        }
        return super.buildOpenChatParams(...arguments);
    }
});

```

## File: static\src\views\profile_form_view.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { formView } from "@web/views/form/form_view";

export class EmployeeProfileController extends formView.Controller {
    setup() {
        super.setup();
        this.action = useService("action");
        this.mustReload = false;
    }

    onWillSaveRecord(record, changes) {
        this.mustReload = "lang" in changes;
    }

    async onRecordSaved(record) {
        await super.onRecordSaved(...arguments);
        if (this.mustReload) {
            this.mustReload = false;
            return this.action.doAction("reload_context");
        }
    }
}

registry.category("views").add("hr_employee_profile_form", {
    ...formView,
    Controller: EmployeeProfileController,
});

```

## File: static\src\views\fields\employee_field_relation_mixin.js

```javascript
/** @odoo-module **/

import { onWillStart } from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";

/**
 * Mixin that handles public/private access of employee records in many2X fields
 * @param { Class } fieldClass
 * @returns Class
 */
export function EmployeeFieldRelationMixin(fieldClass) {
    return class extends fieldClass {
        static props = {
            ...fieldClass.props,
            relation: { type: String, optional: true },
        };

        setup() {
            super.setup();
            this.user = useService("user");
            onWillStart(async () => {
                this.isHrUser = await this.user.hasGroup("hr.group_hr_user");
            });
        }

        get relation() {
            if (this.props.relation) {
                return this.props.relation;
            }
            return this.isHrUser ? "hr.employee" : "hr.employee.public";
        }
    };
}

```

## File: static\src\views\fields\many2many_avatar_employee_field\many2many_avatar_employee_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import {
    Many2ManyTagsAvatarUserField,
    KanbanMany2ManyTagsAvatarUserField,
    many2ManyTagsAvatarUserField,
    kanbanMany2ManyTagsAvatarUserField,
    listMany2ManyTagsAvatarUserField,
} from "@mail/views/web/fields/many2many_avatar_user_field/many2many_avatar_user_field";
import { EmployeeFieldRelationMixin } from "@hr/views/fields/employee_field_relation_mixin";

export class Many2ManyTagsAvatarEmployeeField extends EmployeeFieldRelationMixin(
    Many2ManyTagsAvatarUserField
) {}

export const many2ManyTagsAvatarEmployeeField = {
    ...many2ManyTagsAvatarUserField,
    component: Many2ManyTagsAvatarEmployeeField,
    additionalClasses: [
        ...many2ManyTagsAvatarUserField.additionalClasses,
        "o_field_many2many_avatar_user",
    ],
    extractProps: (fieldInfo, dynamicInfo) => ({
        ...many2ManyTagsAvatarUserField.extractProps(fieldInfo, dynamicInfo),
        canQuickCreate: false,
        relation: fieldInfo.options?.relation,
    }),
};

registry.category("fields").add("many2many_avatar_employee", many2ManyTagsAvatarEmployeeField);

export class KanbanMany2ManyTagsAvatarEmployeeField extends EmployeeFieldRelationMixin(
    KanbanMany2ManyTagsAvatarUserField
) {}

export const kanbanMany2ManyTagsAvatarEmployeeField = {
    ...kanbanMany2ManyTagsAvatarUserField,
    component: KanbanMany2ManyTagsAvatarEmployeeField,
    additionalClasses: [
        ...kanbanMany2ManyTagsAvatarUserField.additionalClasses,
        "o_field_many2many_avatar_user",
    ],
    extractProps: (fieldInfo, dynamicInfo) => ({
        ...kanbanMany2ManyTagsAvatarUserField.extractProps(fieldInfo, dynamicInfo),
        relation: fieldInfo.options?.relation,
    }),
};

registry
    .category("fields")
    .add("kanban.many2many_avatar_employee", kanbanMany2ManyTagsAvatarEmployeeField);
export const listMany2ManyTagsAvatarEmployeeField = {
    ...listMany2ManyTagsAvatarUserField,
    additionalClasses: [
        ...listMany2ManyTagsAvatarUserField.additionalClasses,
        "o_field_many2many_avatar_user",
    ],
};
registry
    .category("fields")
    .add("list.many2many_avatar_employee", listMany2ManyTagsAvatarEmployeeField);

```

## File: static\src\views\fields\many2one_avatar_employee_field\many2one_avatar_employee_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import {
    Many2OneAvatarUserField,
    KanbanMany2OneAvatarUserField,
    many2OneAvatarUserField,
    kanbanMany2OneAvatarUserField,
} from "@mail/views/web/fields/many2one_avatar_user_field/many2one_avatar_user_field";
import { EmployeeFieldRelationMixin } from "@hr/views/fields/employee_field_relation_mixin";

export class Many2OneAvatarEmployeeField extends EmployeeFieldRelationMixin(
    Many2OneAvatarUserField
) {
    get many2OneProps() {
        return {
            ...super.many2OneProps,
            relation: this.relation,
        };
    }
}

export const many2OneAvatarEmployeeField = {
    ...many2OneAvatarUserField,
    component: Many2OneAvatarEmployeeField,
    additionalClasses: [
        ...many2OneAvatarUserField.additionalClasses,
        "o_field_many2one_avatar_user",
    ],
    extractProps: (fieldInfo, dynamicInfo) => ({
        ...many2OneAvatarUserField.extractProps(fieldInfo, dynamicInfo),
        canQuickCreate: false,
        relation: fieldInfo.options?.relation,
    }),
};

registry.category("fields").add("many2one_avatar_employee", many2OneAvatarEmployeeField);

export class KanbanMany2OneAvatarEmployeeField extends EmployeeFieldRelationMixin(
    KanbanMany2OneAvatarUserField
) {
    get many2OneProps() {
        return {
            ...super.many2OneProps,
            relation: this.relation,
        };
    }
}

export const kanbanMany2OneAvatarEmployeeField = {
    ...kanbanMany2OneAvatarUserField,
    component: KanbanMany2OneAvatarEmployeeField,
    extractProps: (fieldInfo, dynamicInfo) => ({
        ...kanbanMany2OneAvatarUserField.extractProps(fieldInfo, dynamicInfo),
        relation: fieldInfo.options?.relation,
    }),
};

registry
    .category("fields")
    .add("kanban.many2one_avatar_employee", kanbanMany2OneAvatarEmployeeField);

```

## File: views\discuss_channel_views.xml

```xml
<?xml version="1.0" ?>
<odoo><data>
    <record id="discuss_channel_view_form" model="ir.ui.view">
        <field name="name">discuss.channel.view.form.inherit.hr</field>
        <field name="model">discuss.channel</field>
        <field name="inherit_id" ref="mail.discuss_channel_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='group_ids']" position="after">
                <field name="subscription_department_ids" widget="many2many_tags"
                    invisible="channel_type != 'channel'"
                    string="Auto Subscribe Departments"/>
            </xpath>
        </field>
    </record>
</data></odoo>
```

## File: views\hr_contract_type_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="hr_contract_type_view_tree" model="ir.ui.view">
            <field name="model">hr.contract.type</field>
            <field name="arch" type="xml">
                <tree string="Contract Types" editable="bottom">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                    <field name="code" groups="base.group_no_one"/>
                    <field name="country_id"/>
                </tree>
            </field>
        </record>

        <record id="hr_contract_type_view_form" model="ir.ui.view">
            <field name="model">hr.contract.type</field>
            <field name="arch" type="xml">
                <form>
                    <group>
                        <group>
                            <field name="name"/>
                            <field name="code" groups="base.group_no_one"/>
                            <field name="country_id"/>
                        </group>
                    </group>
                </form>
            </field>
        </record>

        <record id="hr_contract_type_action" model="ir.actions.act_window">
            <field name="name">Employment Types</field>
            <field name="res_model">hr.contract.type</field>
            <field name="view_mode">tree</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a new employment type
                </p>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\hr_department_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_department_form" model="ir.ui.view">
            <field name="name">hr.department.form</field>
            <field name="model">hr.department</field>
            <field name="arch" type="xml">
                <form string="department">
                    <field name="company_id" invisible="1"/>
                    <sheet>
                        <div class="oe_button_box" name="button_box" invisible="not id">
                            <button class="oe_stat_button" type="action" name="%(hr.act_employee_from_department)d" icon="fa-users">
                                <field string="Employees" name="total_employee" widget="statinfo"/>
                            </button>
                            <button class="oe_stat_button" type="object" name="action_plan_from_department" icon="fa-list-ul">
                                <field string="Plans" name="plans_count" widget="statinfo"/>
                            </button>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                        <field name="active" invisible="1"/>
                        <group>
                            <group>
                                <field name="name"/>
                                <field name="manager_id" widget="many2one_avatar_user"/>
                                <field name="parent_id"/>
                                <field name="child_ids" invisible="1"/>
                                <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                                <field name="color" widget="color_picker" string="Color"/>
                            </group>
                            <div invisible="not id or id and not child_ids and not parent_id">
                                <widget name="hr_department_chart"/>
                            </div>
                        </group>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" groups="base.group_user"/>
                        <field name="message_ids"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="view_department_tree" model="ir.ui.view">
            <field name="name">hr.department.tree</field>
            <field name="model">hr.department</field>
            <field name="arch" type="xml">
                <tree string="Companies" sample="1" editable="bottom">
                    <field name="company_id" column_invisible="True"/>
                    <field name="name"/>
                    <field name="company_id" groups="base.group_multi_company" readonly="1"/>
                    <field name="manager_id" widget="many2one_avatar_user"/>
                    <field name="total_employee" string="Employees"/>
                    <field name="parent_id"/>
                    <field name="color" widget="color_picker" string="Color"/>
                </tree>
            </field>
        </record>

        <record id="view_department_filter" model="ir.ui.view">
            <field name="name">hr.department.search</field>
            <field name="model">hr.department</field>
            <field name="arch" type="xml">
                <search string="Departments">
                    <field name="name" string="Department"/>
                    <field name="manager_id" />
                    <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction','=',True)]" groups="mail.group_mail_notification_type_inbox"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                </search>
             </field>
        </record>

        <record id="hr_department_view_kanban" model="ir.ui.view" >
            <field name="name">hr.department.kanban</field>
            <field name="model">hr.department</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_dashboard o_hr_department_kanban o_kanban_small_column" sample="1">
                    <field name="name"/>
                    <field name="company_id"/>
                    <field name="manager_id"/>
                    <field name="color"/>
                    <field name="total_employee"/>
                    <templates>
                        <t t-name="kanban-menu" t-if="!selection_mode">
                            <div class="o_kanban_card_manage_section">
                                <div role="menuitem" class="o_kanban_manage_reports">
                                    <div class="o_kanban_card_manage_title ps-4 pb-1">
                                        <span class="fw-bolder">Reporting</span>
                                    </div>
                                </div>
                            </div>
                            <a t-if="widget.editable" role="menuitem" class="dropdown-item" type="edit">Configuration</a>
                            <ul t-if="widget.editable" class="oe_kanban_colorpicker" data-field="color" role="menu"/>
                        </t>
                        <t t-name="kanban-box">
                            <div t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''}">
                                <div t-attf-class="o_kanban_card_header oe_kanban_details">
                                    <div class="o_kanban_card_header_title">
                                        <div class="o_primary"><a type="edit"><field name="name"/></a></div>
                                        <div class="o_secondary" groups="base.group_multi_company">
                                            <small>
                                                <i class="fa fa-building-o" role="img" aria-label="Company" title="Company"/> <field name="company_id"/>
                                            </small>
                                        </div>
                                    </div>
                                </div>
                                <div class="container o_kanban_card_content" t-if="!selection_mode">
                                    <div class="row o_kanban_card_upper_content">
                                        <div class="col-6 o_kanban_primary_left">
                                            <button class="btn btn-primary" name="%(act_employee_from_department)d" type="action">
                                                <t t-out="record.total_employee.raw_value"/> Employees
                                            </button>
                                        </div>
                                        <div class="col-6 o_kanban_primary_right">
                                        </div>
                                    </div>
                                    <div class="o_kanban_card_lower_content"
                                         style="justify-content: end">
                                        <!-- placeholder for bottom content -->
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="hr_department_kanban_action" model="ir.actions.act_window">
            <field name="name">Departments</field>
            <field name="res_model">hr.department</field>
            <field name="view_mode">kanban,tree,form</field>
            <field name="search_view_id" ref="view_department_filter"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new department
              </p><p>
                Odoo's department structure is used to manage all documents
                related to employees by departments: expenses, timesheets,
                time off, recruitments, etc.
              </p>
            </field>
        </record>
        <record id="hr_department_tree_action" model="ir.actions.act_window">
            <field name="name">Departments</field>
            <field name="res_model">hr.department</field>
            <field name="view_mode">tree,form,kanban</field>
            <field name="search_view_id" ref="view_department_filter"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a new department
                </p><p>
                    Odoo's department structure is used to manage all documents
                    related to employees by departments: expenses, timesheets,
                    leaves, recruitments, etc.
                </p>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\hr_departure_reason_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="hr_departure_reason_view_list" model="ir.ui.view">
            <field name="model">hr.departure.reason</field>
            <field name="arch" type="xml">
                <tree editable="bottom">
                    <field name="sequence" widget="handle" />
                    <field name="name" />
                </tree>
            </field>
        </record>

        <record id="hr_departure_reason_view_form" model="ir.ui.view">
            <field name="model">hr.departure.reason</field>
            <field name="arch" type="xml">
                <form>
                    <group>
                        <group>
                            <field name="sequence" />
                            <field name="name" />
                        </group>
                    </group>
                </form>
            </field>
        </record>

        <record id="hr_departure_reason_action" model="ir.actions.act_window">
            <field name="name">Departure Reasons</field>
            <field name="res_model">hr.departure.reason</field>
            <field name="view_mode">tree</field>
        </record>
    </data>
</odoo>

```

## File: views\hr_employee_category_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_employee_category_form" model="ir.ui.view">
        <field name="name">hr.employee.category.form</field>
        <field name="model">hr.employee.category</field>
        <field name="arch" type="xml">
            <form string="Employee Tags">
                <sheet>
                    <group>
                        <field name="name"/>
                    </group>
                </sheet>
            </form>
        </field>
     </record>

     <record id="view_employee_category_list" model="ir.ui.view">
        <field name="name">hr.employee.category.list</field>
        <field name="model">hr.employee.category</field>
        <field eval="8" name="priority"/>
        <field name="arch" type="xml">
            <tree string="Employees Tags" editable="bottom">
                <field name="name"/>
            </tree>
        </field>
     </record>

     <record id="open_view_categ_form" model="ir.actions.act_window">
        <field name="name">Employee Tags</field>
        <field name="res_model">hr.employee.category</field>
        <field name="view_mode">tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No Tags found ! Let's create one
            </p>
            <p>
                Use tags to categorize your Employees.
            </p>
        </field>
     </record>
 </odoo>

```

## File: views\hr_employee_public_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="hr_employee_public_view_search" model="ir.ui.view">
            <field name="name">hr.employee.search</field>
            <field name="model">hr.employee.public</field>
            <field name="arch" type="xml">
                <search string="Employees">
                    <field name="name" string="Employees" filter_domain="['|',('work_email','ilike',self),('name','ilike',self)]"/>
                    <searchpanel>
                        <field name="company_id" groups="base.group_multi_company" icon="fa-building" enable_counters="1"/>
                        <field name="department_id" icon="fa-users" enable_counters="1"/>
                    </searchpanel>
                    <field name="parent_id" string="Manager"/>
                    <field name="job_title"/>
                    <separator/>
                    <filter name="my_team" string="My Team" domain="[('parent_id.user_id', '=', uid)]"/>
                    <filter name="my_department" string="My Department" domain="[('member_of_department', '=', True)]"/>
                    <separator/>
                    <filter name="newly_hired" string="Newly Hired" domain="[('newly_hired', '=', True)]"/>
                    <separator/>
                    <filter name="archived" string="Archived" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Group By">
                        <filter name="group_manager" string="Manager" domain="[]" context="{'group_by':'parent_id'}"/>
                        <filter name="group_department" string="Department" domain="[]" context="{'group_by':'department_id'}"/>
                        <filter name="group_company" string="Company" domain="[]" context="{'group_by':'company_id'}"/>
                    </group>
                </search>
             </field>
        </record>

        <record id="hr_employee_public_view_form" model="ir.ui.view">
            <field name="name">hr.employee.public.form</field>
            <field name="model">hr.employee.public</field>
            <field name="arch" type="xml">
                <form string="Employee" create="0" write="0" js_class="hr_employee_form">
                    <field name="image_128" invisible="1" />
                    <header/>
                    <sheet>
                        <field name="user_id" invisible="1"/>
                        <field name="user_partner_id" invisible="1"/>
                        <field name="active" invisible="1"/>
                        <div class="oe_button_box" name="button_box">
                            <!-- Used by other modules-->
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                        <div class="row justify-content-between position-relative w-100 m-0 mb-2">
                            <div class="oe_title ps-0 pe-2">
                                <label for="name" string="Employee Name"/>
                                <h1 class="d-flex flex-row">
                                    <span class="me-2" invisible="not context.get('chat_icon')">
                                        <widget name="hr_employee_chat"/>
                                    </span>
                                    <field name="name" placeholder="e.g. John Doe" required="True"
                                        style="font-size: min(4vw, 2.6rem);"/>
                                </h1>
                                <h2>
                                    <field name="job_title" placeholder="Job Title" />
                                </h2>
                            </div>
                            <div class="o_employee_avatar m-0 p-0">
                                <field name="avatar_1920" widget='image' class="oe_avatar" options='{"zoom": true, "preview_image":"avatar_128"}'/>
                                <field name="show_hr_icon_display" invisible="1" />
                                <field name="hr_icon_display" class="d-flex align-items-end fs-6 o_employee_availability" invisible="not show_hr_icon_display or not id" widget="hr_presence_status"/>
                            </div>
                        </div>
                        <group>
                            <group>
                                <field name="mobile_phone" widget="phone"/>
                                <field name="work_phone" widget="phone"/>
                                <field name="work_email" widget="email"/>
                            </group>
                            <group>
                                <field name="department_id" context="{'open_employees_kanban': 1}"/>
                                <field name="job_id"/>
                                <field name="company_id" groups="base.group_multi_company"/>
                                <field name="parent_id" widget="many2one_avatar_user"/>
                                <field name="coach_id" widget="many2one_avatar_user"/>
                            </group>
                        </group>
                        <notebook>
                            <page name="public" string="Work Information">
                                <div id="o_work_employee_container" class="d-lg-flex"> <!-- These two div are used to position org_chart -->
                                    <div id="o_work_employee_main" class="flex-grow-1">
                                        <group string="Location" name="location">
                                            <field name="address_id"
                                                context="{'show_address': 1}"
                                                options='{"highlight_first_line": True}'/>
                                            <field name="work_location_id"/>
                                        </group>
                                        <group name="managers" string="Approvers" invisible="1">
                                            <!-- overridden in other modules -->
                                        </group>
                                    </div>
                                </div>
                            </page>
                        </notebook>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="hr_employee_public_view_tree" model="ir.ui.view">
            <field name="name">hr.employee.tree</field>
            <field name="model">hr.employee.public</field>
            <field name="arch" type="xml">
                <tree string="Employees" sample="1">
                    <field name="name"/>
                    <field name="work_phone" class="o_force_ltr"/>
                    <field name="work_email"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="department_id"/>
                    <field name="job_title"/>
                    <field name="parent_id" widget="many2one_avatar_user"/>
                    <field name="coach_id" optional="show"/>
                </tree>
            </field>
        </record>

        <record id="hr_employee_public_view_kanban" model="ir.ui.view">
            <field name="name">hr.employee.kanban</field>
            <field name="model">hr.employee.public</field>
            <field name="priority">10</field>
            <field name="arch" type="xml">
                <kanban class="o_hr_employee_kanban" sample="1">
                    <field name="id"/>
                    <field name="hr_presence_state"/>
                    <field name="user_id"/>
                    <field name="user_partner_id"/>
                    <field name="last_activity"/>
                    <field name="hr_icon_display"/>
                    <field name="show_hr_icon_display"/>
                    <field name="image_128" />
                    <templates>
                        <t t-name="kanban-box">
                        <div class="oe_kanban_global_click o_kanban_record_has_image_fill o_hr_kanban_record">
                            <t t-if="!record.image_1024.raw_value">
                                <field name="avatar_128" class="o_kanban_image_fill_left d-block"
                                    widget="background_image" options="{'zoom': true, 'zoom_delay': 1000}"/>
                            </t>
                            <t t-else="">
                                <field name="image_1024" class="o_kanban_image_fill_left d-block" preview_image="image_128"
                                    widget="background_image" options="{'zoom': true, 'zoom_delay': 1000}"/>
                            </t>
                            <div class="oe_kanban_details">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings">
                                        <strong class="o_kanban_record_title">
                                            <field name="name" placeholder="Employee's Name"/>
                                            <div class="float-end">
                                                <t t-if="record.show_hr_icon_display.raw_value">
                                                    <field name="hr_icon_display" class="o_employee_availability align-items-center" widget="hr_presence_status" />
                                                </t>
                                            </div>
                                        </strong>
                                        <span t-if="record.job_title.raw_value" class="o_kanban_record_subtitle"><field name="job_title"/></span>
                                    </div>
                                </div>
                                <ul>
                                    <li t-if="record.work_email.raw_value">
                                        <i class="fa fa-fw me-2 fa-envelope text-primary" title="Email"/>
                                        <field name="work_email"/>
                                    </li>
                                    <li t-if="record.work_phone.raw_value" class="o_force_ltr">
                                        <i class="fa fa-fw me-2 fa-phone text-primary" title="Phone"/>
                                        <field name="work_phone"/>
                                    </li>
                                </ul>
                                <div class="oe_kanban_content position-absolute start-0 bottom-0 end-0 me-2">
                                    <div class="o_kanban_record_bottom mt-3">
                                        <div class="oe_kanban_bottom_left"/>
                                        <div class="oe_kanban_bottom_right">
                                            <div class="hr_avatar mb-1 ms-2 me-n1">
                                                <field name="user_id" widget="many2one_avatar_user" readonly="1"/>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="hr_employee_public_action" model="ir.actions.act_window">
            <field name="name">Employees</field>
            <field name="res_model">hr.employee.public</field>
            <field name="view_mode">kanban,tree,form</field>
            <field name="domain">[]</field>
            <field name="context">{'chat_icon': True}</field>
            <field name="view_id" eval="False"/>
            <field name="search_view_id" ref="hr_employee_public_view_search"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Add a new employee
              </p><p>
                With just a quick glance on the Odoo employee screen, you
                can easily find all the information you need for each person;
                contact data, job position, availability, etc.
              </p>
            </field>
        </record>

    </data>
</odoo>

```

## File: views\hr_employee_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="view_employee_filter" model="ir.ui.view">
            <field name="name">hr.employee.search</field>
            <field name="model">hr.employee</field>
            <field name="arch" type="xml">
                <search string="Employees">
                    <field name="name" string="Employee" filter_domain="['|', ('work_email', 'ilike', self), ('name', 'ilike', self)]"/>
                    <searchpanel>
                        <field name="company_id" groups="base.group_multi_company" icon="fa-building" enable_counters="1"/>
                        <field name="department_id" icon="fa-users" enable_counters="1"/>
                    </searchpanel>
                    <field name="parent_id" string="Manager"/>
                    <field name="job_id"/>
                    <field name="coach_id"/>
                    <field name="category_ids" groups="hr.group_hr_user"/>
                    <field name="private_car_plate" />
                    <separator/>
                    <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction', '=', True)]" groups="mail.group_mail_notification_type_inbox"/>
                    <separator/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <separator/>
                    <filter name="my_team" string="My Team" domain="[('parent_id.user_id', '=', uid)]"/>
                    <filter name="my_department" string="My Department" domain="[('member_of_department', '=', True)]"/>
                    <separator/>
                    <filter name="newly_hired" string="Newly Hired" domain="[('newly_hired', '=', True)]"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Group By">
                        <filter name="group_manager" string="Manager" domain="[]" context="{'group_by': 'parent_id'}"/>
                        <filter name="group_department" string="Department" domain="[]" context="{'group_by': 'department_id'}"/>
                        <filter name="group_job" string="Job" domain="[]" context="{'group_by': 'job_id'}"/>
                        <filter name="group_start" string="Start Date" domain="[]" context="{'group_by': 'create_date'}"/>
                        <filter name="group_category_ids" string="Tags" domain="[]" context="{'group_by': 'category_ids'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="view_employee_form" model="ir.ui.view">
            <field name="name">hr.employee.form</field>
            <field name="model">hr.employee</field>
            <field name="arch" type="xml">
                <form string="Employee" js_class="hr_employee_form">
                    <field name="active" invisible="1"/>
                    <field name="user_id" invisible="1"/>
                    <field name="user_partner_id" invisible="1"/>
                    <field name="image_128" invisible="1" />
                    <field name="company_id" invisible="1"/>
                    <field name="last_activity_time" invisible="1"/>
                    <field name="last_activity" invisible="1"/>
                    <field name="work_contact_id" invisible="1"/>
                    <header>
                        <button name="%(plan_wizard_action)d" string="Launch Plan" type="action" groups="hr.group_hr_user" invisible="not active"/>
                    </header>
                    <sheet>
                        <div name="button_box" class="oe_button_box">
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                        <field name="avatar_128" invisible="1"/>
                        <div class="row justify-content-between position-relative w-100 m-0 mb-2">
                            <div class="oe_title mw-75 ps-0 pe-2">
                                <h1 class="d-flex flex-row align-items-center">
                                    <div invisible="not user_id" class="me-2">
                                        <widget name="hr_employee_chat" invisible="not context.get('chat_icon')"/>
                                    </div>
                                    <field name="name" placeholder="Employee's Name"
                                        required="True" style="font-size: min(4vw, 2.6rem);"/>
                                </h1>
                                <h2>
                                    <field name="job_title" placeholder="Job Position" />
                                </h2>
                                <field name="category_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}" placeholder="Tags"  groups="hr.group_hr_user"/>
                            </div>
                            <div class="o_employee_avatar m-0 p-0">
                                <field name="image_1920" widget='image' class="oe_avatar m-0" options='{"zoom": true, "preview_image":"avatar_128"}'/>
                                <field name="show_hr_icon_display" invisible="1" />
                                <field name="hr_icon_display" class="d-flex align-items-end fs-6 o_employee_availability" invisible="not show_hr_icon_display or not id" widget="hr_presence_status"/>
                            </div>
                        </div>
                        <group>
                            <group>
                                <field name="mobile_phone" widget="phone"/>
                                <field name="work_phone" widget="phone"/>
                                <field name="work_email" widget="email"/>
                                <field name="company_id" groups="base.group_multi_company"/>
                                <field name="company_country_id" invisible="1"/>
                                <field name="company_country_code" invisible="1"/>
                            </group>
                            <group>
                                <field name="department_id"/>
                                <field name="job_id"/>
                                <field name="parent_id" widget="many2one_avatar_user"/>
                                <field name="coach_id" widget="many2one_avatar_user"/>
                            </group>
                        </group>
                        <field name="employee_properties" columns="2"/>
                        <notebook>
                            <page name="public" string="Work Information">
                                <div id="o_work_employee_container" class="d-lg-flex"> <!-- These two div are used to position org_chart -->
                                    <div id="o_work_employee_main" class="flex-grow-1">
                                        <group string="Location">
                                            <field name="address_id"
                                                context="{'show_address': 1}"
                                                options='{"highlight_first_line": True}'/>
                                            <field name="work_location_id" context="{'default_address_id': address_id}" />
                                        </group>
                                        <group name="managers" string="Approvers" class="hide-group-if-empty" invisible="1">
                                            <!-- is overridden in other hr related modules -->
                                        </group>
                                        <group name="departure" string="Departure" invisible="active">
                                            <field name="departure_reason_id" options="{'no_edit': True, 'no_create': True, 'no_open': True}"/>
                                            <field name="departure_description"/>
                                            <field name="departure_date"/>
                                        </group>
                                        <group string="Schedule">
                                            <field name="resource_calendar_id"/>
                                            <field name="id" invisible="1"/>
                                            <field name="tz" required="id"/>
                                        </group>
                                    </div>
                                </div>
                            </page>
                            <page name="personal_information" string="Private Information" groups="hr.group_hr_user">
                                <group>
                                    <group string="Private Contact">
                                        <label for="private_street" string="Private Address"/>
                                        <div class="o_address_format">
                                            <field name="private_street" placeholder="Street..." class="o_address_street"/>
                                            <field name="private_street2" placeholder="Street 2..." class="o_address_street"/>
                                            <field name="private_city" placeholder="City" class="o_address_city"/>
                                            <field name="private_state_id" class="o_address_state" placeholder="State" options="{'no_open': True, 'no_quick_create': True}" context="{'default_country_id': private_country_id}"/>
                                            <field name="private_zip" placeholder="ZIP" class="o_address_zip"/>
                                            <field name="private_country_id" placeholder="Country" class="o_address_country" options='{"no_open": True, "no_create": True}'/>
                                        </div>
                                        <field name="private_email" string="Email"/>
                                        <field name="private_phone" string="Phone"/>
                                        <field name="bank_account_id" context="{'default_partner_id': work_contact_id}" options="{'no_quick_create': True}" readonly="not id"/>
                                        <field name="lang" string="Language"/>
                                        <label for="km_home_work"/>
                                        <div class="o_row" name="div_km_home_work">
                                            <field name="km_home_work" class="o_hr_narrow_field"/>
                                            <span>Km</span>
                                        </div>
                                        <field name="private_car_plate" />
                                    </group>
                                    <group string="Family Status">
                                        <field name="marital"/>
                                        <field name="spouse_complete_name" invisible="marital not in ['married', 'cohabitant']"/>
                                        <field name="spouse_birthdate" invisible="marital not in ['married', 'cohabitant']"/>
                                        <field name="children"/>
                                    </group>
                                    <group string="Emergency" name="emergency">
                                        <field name="emergency_contact"/>
                                        <field name="emergency_phone" class="o_force_ltr"/>
                                    </group>
                                    <group string="Education">
                                        <field name="certificate"/>
                                        <field name="study_field"/>
                                        <field name="study_school"/>
                                        <separator name="has_work_permit" string="Work Permit"/>
                                        <field name="visa_no"/>
                                        <field name="permit_no"/>
                                        <field name="visa_expire"/>
                                        <field name="work_permit_expiration_date"/>
                                        <field name="work_permit_name" invisible="1"/>
                                        <field name="has_work_permit" widget="work_permit_upload" filename="work_permit_name"/>
                                    </group>
                                    <group string="Citizenship">
                                        <field name="country_id" options='{"no_open": True, "no_create": True}'/>
                                        <field name="identification_id"/>
                                        <field name="ssnid"/>
                                        <field name="passport_id"/>
                                        <field name="gender"/>
                                        <field name="birthday"/>
                                        <field name="place_of_birth"/>
                                        <field name="country_of_birth"/>
                                    </group>
                                </group>
                            </page>
                            <page name="hr_settings" string="HR Settings" groups="hr.group_hr_user">
                                <group>
                                    <group string='Status' name="active_group">
                                        <field name="employee_type"/>
                                        <field name="user_id" string="Related User" domain="[('company_ids', 'in', company_id), ('share', '=', False)]" context="{'default_create_employee_id': id}" widget="many2one_avatar_user"/>
                                    </group>
                                    <group string="Attendance/Point of Sale" name="identification_group">
                                        <field name="pin" string="PIN Code"/>
                                        <label for="barcode"/>
                                        <div class="o_row">
                                            <field name="barcode"/>
                                            <button string="Generate" class="btn btn-link" type="object" name="generate_random_barcode" invisible="barcode"/>
                                            <button name="%(hr_employee_print_badge)d" string="Print Badge" class="btn btn-link" type="action" invisible="not barcode"/>
                                        </div>
                                    </group>
                                    <group string='Payroll' name="payroll_group" invisible="1">
                                    </group>
                                    <group name="application_group" string="Application Settings" invisible="1"/>
                                </group>
                            </page>
                        </notebook>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" groups="base.group_user" options="{'post_refresh': 'recipients'}"/>
                        <field name="activity_ids"/>
                        <field name="message_ids"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="hr_employee_view_graph" model="ir.ui.view">
            <field name="name">hr.employee.view.graph</field>
            <field name="model">hr.employee</field>
            <field name="arch" type="xml">
                <graph string="New Employees Over Time" type="line" sample="1">
                    <field name="create_date" interval="month"/>
                    <field name="id"/>
                </graph>
            </field>
        </record>

        <record id="hr_employee_view_pivot" model="ir.ui.view">
            <field name="name">hr.employee.view.pivot</field>
            <field name="model">hr.employee</field>
            <field name="arch" type="xml">
                <pivot string="New Employees Over Time" sample="1">
                    <field name="create_date" interval="month" type="row"/>
                    <field name="id"/>
                </pivot>
            </field>
        </record>

        <record id="view_employee_tree" model="ir.ui.view">
            <field name="name">hr.employee.tree</field>
            <field name="model">hr.employee</field>
            <field name="arch" type="xml">
                <tree string="Employees" expand="context.get('expand', False)" multi_edit="1" sample="1" js_class="hr_employee_list">
                    <header>
                        <button name="%(plan_wizard_action)d" string="Launch Plan" type="action" groups="hr.group_hr_user"/>
                    </header>
                    <field name="name" readonly="1"/>
                    <field name="work_phone" class="o_force_ltr" readonly="1" optional="show"/>
                    <field name="work_email"/>
                    <field name="activity_ids" widget="list_activity" optional="show"/>
                    <field name="activity_user_id" optional="hide" string="Activity by" widget="many2one_avatar_user"/>
                    <field name="activity_date_deadline" widget="remaining_days" options="{'allow_order': '1'}" optional="show"/>
                    <field name="company_id" groups="base.group_multi_company" readonly="1" optional="show"/>
                    <field name="department_id"/>
                    <field name="job_id"/>
                    <field name="parent_id" widget="many2one_avatar_user" optional="show"/>
                    <field name="address_id" column_invisible="True"/>
                    <field name="company_id" column_invisible="True"/>
                    <field name="work_location_id" optional="hide"/>
                    <field name="coach_id" column_invisible="True"/>
                    <field name="active" column_invisible="True"/>
                    <field name="category_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                    <field name="country_id" optional="hide"/>
                </tree>
            </field>
        </record>

        <record id="hr_kanban_view_employees" model="ir.ui.view">
            <field name="name">hr.employee.kanban</field>
            <field name="model">hr.employee</field>
            <field name="priority">10</field>
            <field name="arch" type="xml">
                <kanban class="o_hr_employee_kanban" sample="1">
                    <field name="id"/>
                    <field name="hr_presence_state"/>
                    <field name="user_id"/>
                    <field name="user_partner_id"/>
                    <field name="hr_icon_display"/>
                    <field name="show_hr_icon_display"/>
                    <field name="image_128" />
                    <field name="company_id"/>
                    <templates>
                        <t t-name="kanban-box">
                        <div class="oe_kanban_global_click o_kanban_record_has_image_fill o_hr_kanban_record">
                            <t t-if="record.image_1024.raw_value">
                                <field name="image_1024" class="o_kanban_image_fill_left d-block" preview_image="image_128"
                                    widget="background_image" options="{'zoom': true, 'zoom_delay': 1000}"/>
                            </t>
                            <t t-elif="record.image_128.raw_value">
                                <field name="avatar_128" class="o_kanban_image_fill_left d-block"
                                    widget="background_image" options="{'zoom': true, 'zoom_delay': 1000}"/>
                            </t>
                            <div t-else="" class="o_kanban_image_fill_left d-flex align-items-center justify-content-center bg-100 bg-gradient">
                                <svg class="w-75 h-75 opacity-50" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                                    <g fill="currentColor">
                                        <path d="M 10 11 C 4.08 11 2 14 2 16 L 2 19 L 18 19 L 18 16 C 18 14 15.92 11 10 11 Z"/>
                                        <circle cx="10" cy="5.5" r="4.5"/>
                                    </g>
                                </svg>
                            </div>

                            <div class="oe_kanban_details">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings">
                                        <strong class="o_kanban_record_title">
                                            <field name="name" placeholder="Employee's Name"/>
                                            <div class="float-end">
                                                <div t-if="record.show_hr_icon_display.raw_value">
                                                    <field name="hr_icon_display" class="o_employee_availability" widget="hr_presence_status" />
                                                </div>
                                            </div>
                                        </strong>
                                        <span t-if="record.job_title.raw_value" class="o_kanban_record_subtitle">
                                            <field name="job_title"/>
                                        </span>
                                    </div>
                                </div>
                                <ul>
                                    <li t-if="record.work_email.raw_value" class="o_text_overflow">
                                        <i class="fa fa-fw me-2 fa-envelope text-primary" title="Email"/>
                                        <field name="work_email" />
                                    </li>
                                    <li t-if="record.work_phone.raw_value" class="o_force_ltr">
                                        <i class="fa fa-fw me-2 fa-phone text-primary" title="Phone"/>
                                        <field name="work_phone" />
                                    </li>
                                        <field name="employee_properties" widget="properties"/>
                                    <li class="hr_tags">
                                        <field name="category_ids" widget="many2many_tags" options="{'color_field': 'color'}" optional="hide"/>
                                    </li>
                                </ul>
                            </div>
                            <div class="oe_kanban_content o_hr_employee_kanban_bottom position-absolute bottom-0 start-0 end-0">
                                <div class="o_kanban_record_bottom mt-3">
                                    <div class="oe_kanban_bottom_left"/>
                                    <div class="oe_kanban_bottom_right">
                                        <div class="hr_avatar mb-1 ms-2 me-n1">
                                            <field name="user_id" widget="many2one_avatar_user" readonly="1"/>
                                        </div>
                                        <div class="hr_activity_container mb-1 ms-2 me-n1">
                                            <field name="activity_ids" widget="kanban_activity"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="hr_employee_view_activity" model="ir.ui.view">
            <field name="name">hr.employee.activity</field>
            <field name="model">hr.employee</field>
            <field name="arch" type="xml">
                <activity string="Employees">
                    <field name="id"/>
                    <templates>
                        <div t-name="activity-box">
                            <img class="rounded" t-att-src="activity_image('hr.employee', 'avatar_128', record.id.raw_value)" role="img" t-att-title="record.id.value" t-att-alt="record.id.value"/>
                            <div class="ms-2">
                                <field name="name" display="full" class="o_text_block"/>
                                <field name="job_id" muted="1" display="full" class="o_text_block"/>
                            </div>
                        </div>
                    </templates>
                </activity>
            </field>
        </record>

        <record id="open_view_employee_list_my" model="ir.actions.act_window">
            <field name="name">Employees</field>
            <field name="res_model">hr.employee</field>
            <field name="view_mode">kanban,tree,form,activity,graph,pivot</field>
            <field name="domain">[]</field>
            <field name="context">{'chat_icon': True}</field>
            <field name="view_id" eval="False"/>
            <field name="search_view_id" ref="view_employee_filter"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Add a new employee
                </p><p>
                    With just a quick glance on the Odoo employee screen, you
                    can easily find all the information you need for each person;
                    contact data, job position, availability, etc.
                </p>
            </field>
        </record>

        <record id="open_view_employee_tree" model="ir.actions.act_window">
            <field name="name">Employees Structure</field>
            <field name="res_model">hr.employee</field>
            <field name="view_mode">tree,form</field>
            <field name="view_id" ref="view_employee_tree"/>
            <field name="domain">[('parent_id','=',False)]</field>
            <field name="search_view_id" ref="view_employee_filter"/>
        </record>

        <record id="open_view_employee_list" model="ir.actions.act_window">
            <field name="name">Employees</field>
            <field name="res_model">hr.employee</field>
            <field name="view_mode">form,tree</field>
            <field name="view_id" eval="False"/>
            <field name="search_view_id" ref="view_employee_filter"/>
        </record>

        <record id="action_hr_employee_create_user" model="ir.actions.server">
            <field name="name">Create User</field>
            <field name="model_id" ref="model_hr_employee"/>
            <field name="binding_model_id" ref="model_hr_employee"/>
            <field name="binding_view_types">form</field>
            <field name="groups_id" eval="[(4, ref('base.group_erp_manager'))]"/>
            <field name="state">code</field>
            <field name="code">
                action = records.action_create_user()
            </field>
        </record>

        <record id="act_employee_from_department" model="ir.actions.act_window">
            <field name="name">Employees</field>
            <field name="res_model">hr.employee</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="search_view_id" ref="view_employee_filter"/>
            <field name="context">{
                "searchpanel_default_department_id": active_id,
                "default_department_id": active_id,
                "search_default_group_department": 1,
                "search_default_department_id": active_id,
                "expand": 1}
            </field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Add a new employee
                </p><p>
                    With just a quick glance on the Odoo employee screen, you
                    can easily find all the information you need for each person;
                    contact data, job position, availability, etc.
                </p>
            </field>
        </record>

    </data>
</odoo>

```

## File: views\hr_job_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="view_hr_job_form" model="ir.ui.view">
            <field name="name">hr.job.form</field>
            <field name="model">hr.job</field>
            <field name="arch" type="xml">
                <form string="Job">
                    <field name="active" invisible="1"/>
                    <field name="company_id" invisible="1"/>
                    <sheet>
                        <div class="oe_button_box" name="button_box"/>
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                        <div class="oe_title">
                            <label for="name"/>
                            <h1><field name="name" options="{'line_breaks': False}" widget="text" placeholder="e.g. Sales Manager"/></h1>
                        </div>
                        <notebook>
                            <page string="Recruitment" name="recruitment_page">
                                <group>
                                    <group name="recruitment">
                                        <field name="company_id" options="{'no_create': True}" invisible="1" groups="base.group_multi_company"/>
                                        <field name="department_id"/>
                                        <field name="contract_type_id"/>
                                    </group>
                                    <group name="recruitment2">
                                        <label for="no_of_recruitment"/>
                                        <div class="o_row" name="recruitment_target">
                                            <field name="no_of_recruitment" class="o_hr_narrow_field"/>
                                            <span>new Employees</span>
                                        </div>
                                    </group>
                                </group>
                            </page>
                            <page string="Job Summary" name="job_description_page">
                                <field name="description" options="{'collaborative': true}"/>
                            </page>
                        </notebook>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" options="{'open_attachments': True}"/>
                        <field name="message_ids"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="view_hr_job_tree" model="ir.ui.view">
            <field name="name">hr.job.tree</field>
            <field name="model">hr.job</field>
            <field name="arch" type="xml">
                <tree string="Job" sample="1">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                    <field name="department_id"/>
                    <field name="no_of_recruitment"/>
                    <field name="no_of_employee" optional="hide"/>
                    <field name="expected_employees" optional="hide"/>
                    <field name="no_of_hired_employee" optional="hide"/>
                    <field name="message_needaction" column_invisible="True"/>
                    <field name="company_id" groups="base.group_multi_company" optional="hide"/>
                    <field name="company_id" column_invisible="True"/>
                </tree>
            </field>
        </record>

        <record id="hr_job_view_kanban" model="ir.ui.view">
            <field name="name">hr.job.kanban</field>
            <field name="model">hr.job</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile" sample="1">
                    <templates>
                        <t t-name="kanban-box">
                            <div class="oe_kanban_global_click">
                                <div>
                                    <strong><field name="name"/></strong>
                                </div>
                                <div>
                                    <span><field name="department_id"/>&amp;nbsp;</span>
                                </div>
                                <div t-if="!selection_mode">
                                    <span>Vacancies: <field name="expected_employees"/></span>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="view_job_filter" model="ir.ui.view">
            <field name="name">hr.job.search</field>
            <field name="model">hr.job</field>
            <field name="arch" type="xml">
                <search string="Jobs">
                    <field name="name" string="Job Position"/>
                    <field name="department_id" operator="child_of"/>
                    <separator/>
                    <filter name="message_needaction" string="Unread Messages" domain="[('message_needaction', '=', True)]" groups="mail.group_mail_notification_type_inbox"/>
                    <separator/>
                    <filter name="archived" string="Archived" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Group By">
                        <filter string="Department" name="department" domain="[]" context="{'group_by': 'department_id'}"/>
                        <filter string="Company" name="company" domain="[]" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
                        <filter string="Employment Type" name="employment_type" domain="[]" context="{'group_by': 'contract_type_id'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="action_hr_job" model="ir.actions.act_window">
            <field name="name">Job Positions</field>
            <field name="res_model">hr.job</field>
            <field name="view_mode">tree,form</field>
            <field name="search_view_id" ref="view_job_filter"/>
            <field name="context">{"search_default_Current":1}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Ready to recruit more efficiently?
              </p><p>
                Let's create a job position.
              </p>
            </field>
        </record>

    </data>
</odoo>

```

## File: views\hr_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="hr_employee_plan_activity_summary">
        <div class="d-flex flex-column flex-grow">
            <t t-foreach="activity_ids" t-as="activity">
                <span><i t-attf-class="fa #{activity.icon}"/> <t t-esc="activity.summary"/> (<t t-esc="activity.user_id.name"/>)</span>
                <span><i class="fa fa-clock-o"/> <span t-field="activity.date_deadline"/></span>
            </t>
        </div>
    </template>
</odoo>

```

## File: views\hr_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <menuitem
            id="menu_hr_root"
            name="Employees"
            groups="group_hr_manager,group_hr_user,base.group_user"
            web_icon="hr,static/description/icon.png"
            sequence="185"/>

        <menuitem
            id="menu_hr_main"
            name="Human Resources"
            parent="menu_hr_root"
            sequence="0"/>

        <menuitem
            id="menu_hr_employee_payroll"
            name="Employees"
            parent="menu_hr_root"
            groups="group_hr_user"
            sequence="3"/>

            <menuitem
                id="menu_hr_employee_user"
                name="Employees"
                action="open_view_employee_list_my"
                parent="menu_hr_employee_payroll"
                sequence="1"/>

        <menuitem
            id="menu_hr_employee"
            name="Directory"
            action="hr_employee_public_action"
            parent="menu_hr_root"
            sequence="4"/>

        <menuitem
            id="hr_menu_hr_reports"
            name="Reporting"
            parent="menu_hr_root"
            sequence="95"/>

        <menuitem
           id="menu_hr_reporting_timesheet"
           name="Reporting"
           parent="menu_hr_root"
           groups="group_hr_manager,group_hr_user"
           sequence="99"/>

        <menuitem
            id="menu_human_resources_configuration"
            name="Configuration"
            parent="menu_hr_root"
            groups="hr.group_hr_user"
            sequence="100"/>

            <menuitem
                id="menu_config_employee"
                name="Employee"
                parent="menu_human_resources_configuration"
                groups="group_hr_user"
                sequence="10"/>

            <menuitem
                id="menu_hr_department_tree"
                action="hr_department_tree_action"
                parent="menu_config_employee"
                sequence="4"
                groups="group_hr_user"/>

            <menuitem
                id="menu_hr_department_kanban"
                action="hr_department_kanban_action"
                parent="menu_hr_root"
                groups="group_hr_user"/>

            <menuitem
                id="menu_hr_work_location_tree"
                action="hr_work_location_action"
                parent="menu_config_employee"
                sequence="5"
                groups="group_hr_user"/>

            <menuitem
                id="menu_resource_calendar_view"
                action="resource.action_resource_calendar_form"
                parent="menu_config_employee"
                name="Working Schedules"
                sequence="6"
                groups="group_hr_user"/>

            <menuitem
                id="menu_hr_departure_reason_tree"
                action="hr_departure_reason_action"
                parent="menu_config_employee"
                sequence="7"
                groups="group_hr_user"/>

            <menuitem
                id="menu_view_employee_category_form"
                name="Tags"
                action="open_view_categ_form"
                parent="menu_config_employee"
                groups="base.group_no_one"
                sequence="10"/>

            <menuitem
                id="menu_config_recruitment"
                name="Recruitment"
                parent="menu_human_resources_configuration"
                groups="group_hr_user"
                sequence="20"/>

                <menuitem
                    id="menu_view_hr_job"
                    action="action_hr_job"
                    parent="menu_config_recruitment"
                    sequence="1"/>

                <menuitem
                    id="menu_view_hr_contract_type"
                    action="hr_contract_type_action"
                    parent="menu_config_recruitment"
                    sequence="2"
                    groups="group_hr_user"/>

            <menuitem
                id="menu_config_plan"
                name="Activity Planning"
                parent="menu_human_resources_configuration"
                groups="group_hr_manager"
                sequence="100"/>

                <menuitem
                    id="menu_config_plan_plan"
                    name="On/Offboarding Plans"
                    action="mail_activity_plan_action"
                    parent="menu_config_plan"
                    groups="group_hr_manager"
                    sequence="100"/>

    </data>
</odoo>

```

## File: views\hr_work_location_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="hr_work_location_tree_view" model="ir.ui.view">
            <field name="name">hr.work.location.view.tree</field>
            <field name="model">hr.work.location</field>
            <field name="arch" type="xml">
                <tree string="Work Location">
                    <field name="active" column_invisible="True" />
                    <field name="name" />
                    <field class="o_homework_icon_types d-flex flex-wrap" name="location_type"
                           widget="hr_homeworking_radio_image" options="{'horizontal': true}"/>
                    <field name="company_id" groups="base.group_multi_company" />
                </tree>
            </field>
        </record>
        <record id="hr_work_location_form_view" model="ir.ui.view">
            <field name="name">hr.work.location.view.form</field>
            <field name="model">hr.work.location</field>
            <field name="arch" type="xml">
                <form string="Work Location">
                    <sheet>
                        <group>
                            <group>
                                <field name="active" invisible="1" />
                                <field name="name" />
                                <field name="address_id" />
                                <field class="o_homework_icon_types d-flex flex-wrap" name="location_type"
                                       widget="hr_homeworking_radio_image" options="{'horizontal': true}"/>
                            </group>
                            <group>
                                <field name="company_id" groups="base.group_multi_company" />
                                <field name="company_id" groups="!base.group_multi_company" invisible="1" />
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>
        <record id="hr_work_location_action" model="ir.actions.act_window">
            <field name="name">Work Locations</field>
            <field name="res_model">hr.work.location</field>
            <field name="view_mode">tree,form</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a new work location
                </p>
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
        <record id="mail_activity_plan_template_view_form" model="ir.ui.view">
            <field name="name">mail.activity.plan.template.view.form.inherit.hr</field>
            <field name="model">mail.activity.plan.template</field>
            <field name="inherit_id" ref="mail.mail_activity_plan_template_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='responsible_type']" position="replace">
                    <field name="responsible_type"
                           invisible="res_model != 'hr.employee'"/>
                    <field name="responsible_type" widget="filterable_selection"
                           options="{'whitelisted_values': ['on_demand', 'other']}"
                           invisible="res_model == 'hr.employee'"/>
                </xpath>
            </field>
        </record>

        <record id="mail_activity_plan_view_form" model="ir.ui.view">
            <field name="name">mail.activity.plan.view.form.inherit.hr</field>
            <field name="model">mail.activity.plan</field>
            <field name="inherit_id" ref="mail.mail_activity_plan_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='template_ids']/tree/field[@name='responsible_type']" position="replace">
                    <!-- We can hide selection options specific to employee because employee plan templates are never
                         edited in the list view, so that there are not present for other model that are edited in the
                         list view. -->
                    <field name="responsible_type" widget="filterable_selection"
                           options="{'whitelisted_values': ['on_demand', 'other']}"/>
                </xpath>
                <xpath expr="//group[@name='group_plan_fields']" position="inside">
                    <field name="department_id" invisible="res_model != 'hr.employee'"
                            placeholder="Available for all Departments"/>
                </xpath>
            </field>
        </record>

        <record id="mail_activity_plan_view_form_hr_employee" model="ir.ui.view">
            <field name="name">mail.activity.plan.view.form.hr.employee</field>
            <field name="mode">primary</field>
            <field name="model">mail.activity.plan</field>
            <field name="priority">32</field>
            <field name="inherit_id" ref="mail.mail_activity_plan_view_form_fixed_model"/>
            <field name="arch" type="xml">
                <xpath expr="//tree" position="attributes">
                    <attribute name="editable"/>
                </xpath>
            </field>
        </record>

        <record id="mail_activity_plan_view_tree" model="ir.ui.view">
            <field name="name">mail.activity.plan.view.tree.inherit.hr</field>
            <field name="model">mail.activity.plan</field>
            <field name="inherit_id" ref="mail.mail_activity_plan_view_tree"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='steps_count']" position="after">
                    <field name="department_id" optional="show"/>
                </xpath>
            </field>
        </record>

        <record id="mail_activity_plan_action" model="ir.actions.act_window">
            <field name="name">On/Offboarding Plans</field>
            <field name="res_model">mail.activity.plan</field>
            <field name="view_mode">tree,form</field>
            <field name="search_view_id" ref="mail.mail_activity_plan_view_search"/>
            <field name="context">{'default_res_model': 'hr.employee'}</field>
            <field name="domain">[('res_model', '=', 'hr.employee')]</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Add a new plan
                </p>
            </field>
        </record>

        <record id="mail_activity_plan_action_employee_view_tree" model="ir.actions.act_window.view">
            <field name="sequence">1</field>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="mail.mail_activity_plan_view_tree"/>
            <field name="act_window_id" ref="hr.mail_activity_plan_action"/>
        </record>

        <!-- Force the hr view which do the activity template edition in a popup. -->
        <record id="mail_activity_plan_action_employee_view_form" model="ir.actions.act_window.view">
            <field name="sequence">2</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="hr.mail_activity_plan_view_form_hr_employee"/>
            <field name="act_window_id" ref="hr.mail_activity_plan_action"/>
        </record>

        <!-- This id is referenced in chatter message as a link to launch the plan -->
        <record id="plan_wizard_action" model="ir.actions.act_window">
            <field name="name">Launch Plan</field>
            <field name="res_model">mail.activity.schedule</field>
            <field name="view_mode">form</field>
            <field name="context">{'plan_mode': True}</field>
            <field name="target">new</field>
        </record>
    </data>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.hr</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="70"/>
        <field name="inherit_id" ref="base.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//form" position="inside">
                <app data-string="Employees" string="Employees" name="hr" groups="hr.group_hr_manager">
                    <block title="Employees" name="employees_setting_container">
                        <setting id="presence_control_setting" title="Presence of employees" string="Presence Control">
                            <div class="content-group" name="hr_presence_options">
                                <div class="d-flex">
                                    <field name="module_hr_attendance" class="ml16"/>
                                    <label for="module_hr_attendance" class="o_light_label"/>
                                </div>
                                <div class="d-flex">
                                    <field name="hr_presence_control_login" class="ml16"/>
                                    <label for="hr_presence_control_login" class="o_light_label"/>
                                </div>
                            </div>
                        </setting>
                        <setting id="presence_reporting_setting" help="Presence reporting screen, email and IP address control." title="Advanced presence of employees">
                            <field name="module_hr_presence"/>
                            <div class="d-flex mt-1" invisible="not module_hr_presence">
                                <field name="hr_presence_control_email" class="ml16"/>
                                <label for="hr_presence_control_email" class="o_light_label"/>
                            </div>
                            <div class="d-flex ml32" invisible="not module_hr_presence or not hr_presence_control_email">
                                <span class="flex-shrink-0 ml8 me-2">Minimum number of emails to send</span>
                                <field name="hr_presence_control_email_amount" class="ms-2 oe_inline"/>
                            </div>
                            <div class="d-flex" invisible="not module_hr_presence">
                                <field name="hr_presence_control_ip" class="ml16"/>
                                <label for="hr_presence_control_ip" class="o_light_label"/>
                            </div>
                            <div class="d-flex ml32" invisible="not module_hr_presence or not hr_presence_control_ip">
                                <span class="flex-shrink-0 ml8 me-2">IP Addresses (comma-separated)</span>
                                <field name="hr_presence_control_ip_list" class="ms-2 oe_inline"/>
                            </div>
                        </setting>
                        <setting help="Enrich employee profiles with skills and resumes" id="enrich_employee_setting">
                            <field name="module_hr_skills"/>
                        </setting>
                        <setting id="home_working_setting" help="Display remote work settings for each employee and dedicated reports. Presence icons will be updated with remote work location.">
                            <field name="module_hr_homeworking"/>
                        </setting>
                    </block>
                    <block title="Work Organization" name="work_organization_setting_container">
                        <setting company_dependent="1" help="Set default company schedule to manage your employees working time" id="default_company_schedule_setting">
                            <field name="resource_calendar_id" required="1" class="o_light_label"
                                domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]"
                                context="{'default_company_id': company_id}"/>
                        </setting>
                    </block>
                    <block title="Employee Update Rights" name="employee_rights_setting_container">
                        <setting help="Allow employees to update their own data" title="Allow employees to update their own data.">
                            <field name="hr_employee_self_edit"/>
                        </setting>
                    </block>
                </app>
            </xpath>
        </field>
    </record>

    <record id="hr_config_settings_action" model="ir.actions.act_window">
        <field name="name">Settings</field>
        <field name="res_model">res.config.settings</field>
        <field name="view_mode">form</field>
        <field name="target">inline</field>
        <field name="context">{'module' : 'hr', 'bin_size': False}</field>
    </record>

    <menuitem id="hr_menu_configuration"
        name="Settings"
        parent="menu_human_resources_configuration"
        sequence="0"
        action="hr_config_settings_action"
        groups="base.group_system"/>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_partner_view_form" model="ir.ui.view">
        <field name="name">res.partner.view.form.inherit.hr</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button name="action_open_employees" type="object" class="oe_stat_button" icon="fa-id-card-o" groups="hr.group_hr_user" invisible="employees_count == 0">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value"><field name="employees_count"/></span>
                        <span class="o_stat_text">Employee</span>
                    </div>
                </button>
            </div>
        </field>
    </record>
</odoo>

```

## File: views\res_users.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <!-- Inherit the preference view to remove title, image and footer -->
        <!-- This view is meant to be included in the employee profile view -->
        <!-- It ensures that if the 'normal' Preferences view is changed, it's
            also reflected in the employee's profile -->
        <record id="res_users_view_form_simple_modif" model="ir.ui.view">
            <field name="name">res.users.preferences.form.simplified.inherit</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="base.view_users_form_simple_modif"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <footer position="attributes">
                    <attribute name="invisible">1</attribute>
                </footer>
                <h1 position="replace"/>
                <xpath expr="//field[@name='image_1920']" position="replace"/>
                <xpath expr="//field[@name='company_id']" position="attributes">
                    <attribute name="invisible">1</attribute>
                </xpath>
                <field name="share" position="after">
                    <field name="work_contact_id" invisible="1"/>
                </field>
            </field>
        </record>

        <record id="view_users_form_simple_modif_resource" model="ir.ui.view">
            <field name="name">res.users.preferences.form.resource</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="base.view_users_form_simple_modif" />
            <field name="arch" type="xml">
                <field name="tz" position="attributes">
                    <attribute name="required">1</attribute>
                </field>
                <field name="tz" position="after">
                    <field name="is_system" invisible="1"/>
                </field>
                <xpath expr="//button[@name='%(base.action_view_base_language_install)d']" position="attributes">
                    <attribute name="invisible">not is_system</attribute>
                </xpath>
            </field>
        </record>

        <record id="res_users_view_form_profile" model="ir.ui.view">
            <field name="name">res.users.preferences.form.inherit</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="res_users_view_form_simple_modif"/>
            <field name="arch" type="xml">
                <form position="attributes">
                    <attribute name="create">false</attribute>
                    <attribute name="delete">false</attribute>
                    <attribute name="js_class">hr_employee_profile_form</attribute>
                </form>
                <notebook position="replace">
                        <field name="hr_presence_state" invisible="1"/>
                        <header>
                        </header>
                        <sheet>$0</sheet>
                </notebook>
                <notebook position="before">
                    <div class="oe_button_box" name="button_box">
                        <button
                            id="hr_presence_button"
                            class="oe_stat_button"
                            disabled="1"
                            invisible="context.get('from_my_profile', False) or hr_presence_state == 'absent'">
                            <div role="img" class="fa fa-fw fa-circle text-success o_button_icon" invisible="hr_presence_state != 'present'" aria-label="Available" title="Available"/>
                            <div role="img" class="fa fa-fw fa-circle text-warning o_button_icon" invisible="hr_presence_state != 'to_define'" aria-label="Away" title="Away"/>
                            <div role="img" class="fa fa-fw fa-circle text-danger o_button_icon" invisible="hr_presence_state != 'absent'" aria-label="Not available" title="Not available"/>

                            <div class="o_stat_info" invisible="hr_presence_state == 'present'">
                                <span class="o_stat_text">
                                    Not Connected
                                </span>
                            </div>
                            <div class="o_stat_info" invisible="hr_presence_state != 'present'">
                                <span class="o_stat_value" invisible="not last_activity_time">
                                    <field name="last_activity_time"/>
                                </span>
                                <span class="o_stat_value" invisible="last_activity_time">
                                    <field name="last_activity"/>
                                </span>
                                <span class="o_stat_text">Connected Since</span>
                            </div>
                        </button>
                    </div>
                    <field name="avatar_128" invisible="1"/>
                    <field name="image_1920" widget='image' class="oe_avatar" options='{"zoom": true, "preview_image":"avatar_128"}'/>
                    <div class="oe_title">
                        <h1>
                            <field name="name" placeholder="Employee's Name" required="True" readonly="context.get('from_my_profile', False)"/>
                        </h1>
                    </div>
                    <div class="row">
                        <h2 class="col-lg-6 ps-lg-0">
                            <field name="job_title" class="w-100" placeholder="Job Position" readonly="not can_edit"/>
                        </h2>
                    </div>
                    <group>
                        <group>
                            <field name="can_edit" invisible="1"/>
                            <field name="mobile_phone" widget="phone" readonly="not can_edit"/>
                            <field name="work_phone" widget="phone" readonly="not can_edit"/>
                        </group>
                        <group>
                            <field name="work_email" widget="email" readonly="not can_edit"/>
                            <field name="work_location_id" readonly="not can_edit"/>
                            <field name="company_id" invisible="1"/>
                        </group>
                        <group>
                            <field name="employee_parent_id" readonly="not can_edit"/>
                            <field name="coach_id" readonly="not can_edit"/>
                        </group>
                    </group>
                </notebook>
                <notebook position="inside">
                    <page name="public" string="Work Information">
                        <div id="o_work_employee_container" class="d-lg-flex"> <!-- These two div are used to position org_chart -->
                            <div id="o_work_employee_main" class="flex-grow-1">
                                <group string="Location">
                                    <field name="department_id" readonly="not can_edit"/>
                                    <field name="address_id"
                                        context="{'show_address': 1}"
                                        options='{"highlight_first_line": True}'
                                        readonly="not can_edit"/>
                                </group>
                                <group name="managers" string="Approvers" class="hide-group-if-empty">
                                    <!-- overridden in other modules -->
                                </group>
                            </div>
                        </div>
                    </page>
                    <page name="personal_information" string="Private Information">
                        <group>
                            <group string="Contact Information">
                                <field name="employee_ids" invisible="1"/>
                                <label for="private_street" string="Private Address"/>
                                <div class="o_address_format">
                                    <field name="private_street" placeholder="Street..." class="o_address_street" readonly="not can_edit"/>
                                    <field name="private_street2" placeholder="Street 2..." class="o_address_street" readonly="not can_edit"/>
                                    <field name="private_city" placeholder="City" class="o_address_city" readonly="not can_edit"/>
                                    <field name="private_state_id" class="o_address_state" placeholder="State" options="{'no_open': True, 'no_quick_create': True}" context="{'default_country_id': private_country_id}" readonly="not can_edit"/>
                                    <field name="private_zip" placeholder="ZIP" class="o_address_zip" readonly="not can_edit"/>
                                    <field name="private_country_id" placeholder="Country" class="o_address_country" options='{"no_open": True, "no_create": True}' readonly="not can_edit"/>
                                </div>
                                <field name="private_email" string="Email" readonly="not can_edit"/>
                                <field name="private_phone" string="Phone" class="o_force_ltr" readonly="not can_edit"/>
                                <field name="private_lang" string="Language" readonly="not can_edit"/>
                                <field name="employee_bank_account_id" readonly="not can_edit"/>
                                <field name="km_home_work" readonly="not can_edit"/>
                            </group>
                            <group string="Citizenship">
                                <field name="employee_country_id" options='{"no_open": True, "no_create": True}' readonly="not can_edit"/>
                                <field name="identification_id" readonly="not can_edit"/>
                                <field name="ssnid" readonly="not can_edit"/>
                                <field name="passport_id" readonly="not can_edit"/>
                                <field name="gender" readonly="not can_edit"/>
                                <field name="birthday" readonly="not can_edit"/>
                                <field name="place_of_birth" readonly="not can_edit"/>
                                <field name="country_of_birth" readonly="not can_edit"/>
                            </group>
                            <group string="Marital Status">
                                <field name="marital" readonly="not can_edit"/>
                                <field name="spouse_complete_name" invisible="marital not in ['married', 'cohabitant']" readonly="not can_edit"/>
                                <field name="spouse_birthdate" invisible="marital not in ['married', 'cohabitant']" readonly="not can_edit"/>
                            </group>
                            <group string="Education">
                                <field name="certificate" readonly="not can_edit"/>
                                <field name="study_field" readonly="not can_edit"/>
                                <field name="study_school" readonly="not can_edit"/>
                            </group>
                            <group string="Dependant">
                                <field name="children" readonly="not can_edit"/>
                            </group>
                            <group string="Emergency">
                                <field name="emergency_contact" readonly="not can_edit"/>
                                <field name="emergency_phone" widget="phone" readonly="not can_edit"/>
                            </group>
                            <group string="Work Permit" name="has_work_permit">
                                <field name="visa_no" readonly="not can_edit"/>
                                <field name="permit_no" readonly="not can_edit"/>
                                <field name="visa_expire" readonly="not can_edit"/>
                            </group>
                        </group>
                    </page>
                     <page name="hr_settings" string="HR Settings">
                        <group>
                            <group string='Status' name="active_group">
                                <field name="employee_type" readonly="not can_edit"/>
                            </group>
                            <group string="Attendance" name="identification_group">
                                <field name="pin" readonly="not can_edit"/>
                                <field name="barcode" readonly="not can_edit"/>
                            </group>
                        </group>
                    </page>
                </notebook>
            </field>
        </record>

        <record id="view_users_simple_form_inherit_hr" model="ir.ui.view">
            <field name="name">view.users.simple.form.inherit.hr</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="base.view_users_simple_form"/>
            <field name="arch" type="xml">
                <xpath expr="//sheet" position="inside">
                    <div class="oe_button_box" name="button_box">
                        <button name="action_open_employees"
                            class="oe_stat_button"
                            icon="fa-users"
                            invisible="employee_count == 0"
                            context="{'active_test': False}"
                            type="object">
                            <field name="employee_count" widget="statinfo" string="Employee"/>
                        </button>
                    </div>
                </xpath>
                <xpath expr="//field[@name='mobile']" position="after">
                    <field name="create_employee_id" force_save="1" invisible="1"/>
                    <field name="create_employee" force_save="1" string="Create Employee" invisible="create_employee_id &gt; 0 or id &gt; 0" groups="hr.group_hr_user"/>
                </xpath>
            </field>
        </record>

        <record id="view_users_simple_form" model="ir.ui.view">
            <field name="name">view.users.simple.form.hr</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="base.view_users_simple_form"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <sheet position="after">
                    <footer>
                        <button string="Save" special="save" class="btn btn-primary"/>
                        <button string="Cancel" special="cancel" class="btn btn-secondary"/>
                    </footer>
                </sheet>
            </field>
        </record>

        <record id="res_users_action_my" model="ir.actions.act_window">
            <field name="name">Change my Preferences</field>
            <field name="res_model">res.users</field>
            <field name="view_mode">form</field>
            <field name="context">{'from_my_profile': True}</field>
            <field name="view_id" ref="hr.res_users_view_form_profile"/>
        </record>

        <record id="res_users_view_form" model="ir.ui.view">
            <field name="name">res.users.form.inherit</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="base.view_users_form"/>
            <field name="arch" type="xml">

                <xpath expr="//header" position="inside">
                    <field name="share" invisible="1"/>
                    <field name="employee_ids" invisible="1"/>
                    <field name="employee_id" invisible="1"/>
                    <button string="Create employee"
                            type="object" name="action_create_employee"
                            invisible="not id or share or employee_id"/>
                            <!-- share is not correctly recomputed because it depends on fields of reified view => invisible before saving (id=False) -->
                </xpath>
                <xpath expr="//div[@name='button_box']" position="inside">
                    <button name="action_open_employees"
                        class="oe_stat_button"
                        icon="fa-users"
                        invisible="employee_count == 0"
                        context="{'active_test': False}"
                        type="object">
                        <field name="employee_count" widget="statinfo" string="Employee"/>
                    </button>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\hr_departure_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class HrDepartureWizard(models.TransientModel):
    _name = 'hr.departure.wizard'
    _description = 'Departure Wizard'

    def _get_employee_departure_date(self):
        return self.env['hr.employee'].browse(self.env.context['active_id']).departure_date

    def _get_default_departure_date(self):
        departure_date = False
        if self.env.context.get('active_id'):
            departure_date = self._get_employee_departure_date()
        return departure_date or fields.Date.today()

    departure_reason_id = fields.Many2one("hr.departure.reason", default=lambda self: self.env['hr.departure.reason'].search([], limit=1), required=True)
    departure_description = fields.Html(string="Additional Information")
    departure_date = fields.Date(string="Departure Date", required=True, default=_get_default_departure_date)
    employee_id = fields.Many2one(
        'hr.employee', string='Employee', required=True,
        default=lambda self: self.env.context.get('active_id', None),
    )

    def action_register_departure(self):
        employee = self.employee_id
        if self.env.context.get('toggle_active', False) and employee.active:
            employee.with_context(no_wizard=True).toggle_active()
        employee.departure_reason_id = self.departure_reason_id
        employee.departure_description = self.departure_description
        employee.departure_date = self.departure_date

```

## File: wizard\hr_departure_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="hr_departure_wizard_view_form" model="ir.ui.view">
            <field name="name">hr.departure.wizard.view.form</field>
            <field name="model">hr.departure.wizard</field>
            <field name="arch" type="xml">
                <form>
                    <sheet>
                        <h1><field name="employee_id" readonly="1" options="{'no_open': True}"/></h1>
                        <group>
                            <group id="info">
                                <field name="departure_reason_id" options="{'no_edit': True, 'no_create': True, 'no_open': True}"/>
                                <field name="departure_date"/>
                            </group>
                            <group id="action">
                                <!-- Override invisible="1" when inheriting -->
                                <div class="o_td_label" id="activities_label" invisible="1">
                                    <span class="o_form_label o_hr_form_label cursor-default">Close Activities</span>
                                </div>
                                <!-- Override invisible="1" when inheriting -->
                                <div class="column" id="activities" invisible="1">
                                </div>
                                <div class="o_td_label" id="label_info">
                                    <span class="o_form_label o_hr_form_label cursor-default">HR Info</span>
                                </div>
                                <div class="column" id="info"/>
                            </group>
                        </group>
                        <group>
                            <div id="detailed_reason" colspan="2">
                                <span class="o_form_label o_hr_form_label cursor-default">Detailed Reason</span>
                                <field name="departure_description" placeholder="Give more details about the reason of archiving the employee."/>
                            </div>
                        </group>
                    </sheet>
                    <footer>
                        <button name="action_register_departure" string="Apply" type="object" class="oe_highlight" data-hotkey="q"/>
                        <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="x"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="hr_departure_wizard_action" model="ir.actions.act_window">
            <field name="name">Register Departure</field>
            <field name="res_model">hr.departure.wizard</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
        </record>
    </data>
</odoo>

```

## File: wizard\mail_activity_schedule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.osv import expression


class MailActivitySchedule(models.TransientModel):
    _inherit = 'mail.activity.schedule'

    department_id = fields.Many2one('hr.department', compute='_compute_department_id')

    @api.depends('department_id')
    def _compute_plan_available_ids(self):
        todo = self.filtered(lambda s: s.res_model == 'hr.employee')
        for scheduler in todo:
            base_domain = scheduler._get_plan_available_base_domain()
            if not scheduler.department_id:
                final_domain = expression.AND([base_domain, [('department_id', '=', False)]])
            else:
                final_domain = expression.AND([base_domain, ['|', ('department_id', '=', False), ('department_id', '=', scheduler.department_id.id)]])
            scheduler.plan_available_ids = self.env['mail.activity.plan'].search(final_domain)
        super(MailActivitySchedule, self - todo)._compute_plan_available_ids()

    @api.depends('res_model_id', 'res_ids')
    def _compute_department_id(self):
        for wizard in self:
            if wizard.res_model == 'hr.employee':
                applied_on = wizard._get_applied_on_records()
                all_departments = applied_on.department_id
                wizard.department_id = False if len(all_departments) > 1 else all_departments
            else:
                wizard.department_id = False

```

## File: wizard\mail_activity_schedule_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="mail_activity_schedule_view_form" model="ir.ui.view">
            <field name="name">mail.activity.schedule.view.form.inherit.hr</field>
            <field name="model">mail.activity.schedule</field>
            <field name="inherit_id" ref="mail.mail_activity_schedule_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='plan_available_ids']" position="after">
                    <field name="department_id" invisible="1"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_departure_wizard
from . import mail_activity_schedule

```

