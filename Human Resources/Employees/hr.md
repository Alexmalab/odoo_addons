# Odoo Module: hr

Category: Human Resources/Employees

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, SUPERUSER_ID

from . import models
from . import wizard
from . import populate


def _install_hr_localization(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
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
        'images/hr_department.jpeg',
        'images/hr_employee.jpeg',
        'images/hr_job_position.jpeg',
        'static/src/img/default_image.png',
    ],
    'depends': [
        'base_setup',
        'mail',
        'resource',
        'web',
    ],
    'data': [
        'security/hr_security.xml',
        'security/ir.model.access.csv',
        'wizard/hr_plan_wizard_views.xml',
        'wizard/hr_departure_wizard_views.xml',
        'views/hr_departure_reason_views.xml',
        'views/hr_contract_type_views.xml',
        'views/hr_job_views.xml',
        'views/hr_plan_views.xml',
        'views/hr_employee_category_views.xml',
        'views/hr_employee_public_views.xml',
        'report/hr_employee_badge.xml',
        'views/hr_employee_views.xml',
        'views/hr_department_views.xml',
        'views/hr_work_location_views.xml',
        'views/hr_views.xml',
        'views/res_config_settings_views.xml',
        'views/mail_channel_views.xml',
        'views/res_users.xml',
        'views/res_partner_views.xml',
        'data/hr_data.xml',
    ],
    'demo': [
        'data/hr_demo.xml'
    ],
    'installable': True,
    'application': True,
    'post_init_hook': '_install_hr_localization',
    'assets': {
        'mail.assets_messaging': [
            'hr/static/src/models/*.js',
        ],
        'web.assets_backend': [
            'hr/static/src/views/**/*.js',
            'hr/static/src/components/**/*',
            'hr/static/src/user_menu/*.js',
            'hr/static/src/scss/*.scss',
            'hr/static/src/js/m2x_avatar_employee.js',
            'hr/static/src/js/standalone_m2o_avatar_employee.js',
            'hr/static/src/js/work_permit_upload.js',
            'hr/static/src/xml/*.xml',
        ],
        'web.qunit_suite_tests': [
            'hr/static/tests/helpers/*.js',
            'hr/static/tests/m2x_avatar_employee_tests.js',
            'hr/static/tests/m2x_avatar_employee_legacy_tests.js',
            'hr/static/tests/standalone_m2o_avatar_employee_tests.js',
        ],
        'web.assets_tests': [
            'hr/static/tests/tours/hr_employee_flow.js',
            'hr/static/tests/tours/user_modify_own_profile_tour.js',
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

        <record id="dep_sales" model="hr.department">
          <field name="name">Sales</field>
        </record>

        <record id="res_partner_admin_private_address" model="res.partner">
            <field name="name">Administrator</field>
            <field name="company_id" ref="base.main_company"/>
            <field name="email">admin@example.com</field>
            <field name="type">private</field>
        </record>

        <record id="employee_admin" model="hr.employee">
            <field name="name" eval="obj(ref('base.partner_admin')).name" model="res.partner"/>
            <field name="department_id" ref="dep_administration"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="address_id" ref="base.main_partner"/>
            <field name="address_home_id" ref="res_partner_admin_private_address"/>
            <field name="image_1920" eval="obj(ref('base.partner_admin')).image_1920" model="res.partner"/>
        </record>

        <record id="onboarding_plan" model='hr.plan'>
            <field name="name">Onboarding</field>
        </record>

        <record id="onboarding_setup_it_materials" model="hr.plan.activity.type">
            <field name="summary">Setup IT Materials</field>
            <field name="responsible">manager</field>
            <field name="plan_id" ref="onboarding_plan"/>
        </record>

        <record id="onboarding_plan_training" model="hr.plan.activity.type">
            <field name="summary">Plan Training</field>
            <field name="responsible">manager</field>
            <field name="plan_id" ref="onboarding_plan"/>
        </record>

        <record id="onboarding_training" model="hr.plan.activity.type">
            <field name="summary">Training</field>
            <field name="responsible">employee</field>
            <field name="plan_id" ref="onboarding_plan"/>
        </record>

        <record id="offboarding_plan" model='hr.plan'>
            <field name="name">Offboarding</field>
        </record>

        <record id="offboarding_setup_compute_out_delais" model="hr.plan.activity.type">
            <field name="summary">Organize knowledge transfer inside the team</field>
            <field name="responsible">manager</field>
            <field name="plan_id" ref="offboarding_plan"/>
        </record>

        <record id="offboarding_take_back_hr_materials" model="hr.plan.activity.type">
            <field name="summary">Take Back HR Materials</field>
            <field name="responsible">manager</field>
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
        </record>

        <record id="departure_resigned" model="hr.departure.reason">
            <field name="sequence">1</field>
            <field name="name">Resigned</field>
        </record>

        <record id="departure_retired" model="hr.departure.reason">
            <field name="sequence">2</field>
            <field name="name">Retired</field>
        </record>

        <!-- Work permit expires Soon -->
        <record id="ir_cron_data_check_work_permit_validity" model="ir.cron">
            <field name="name">HR Employee: check work permit validity</field>
            <field name="model_id" ref="model_hr_employee"/>
            <field name="type">ir.actions.server</field>
            <field name="state">code</field>
            <field name="code">model._cron_check_work_permit_validity()</field>
            <field name="interval_number">1</field>
            <field name="interval_type">days</field>
            <field name="numbercall">-1</field>
        </record>

    </data>
</odoo>

```

## File: data\hr_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

    <!--Department-->

      <record id="dep_management" model="hr.department">
          <field name="name">Management</field>
      </record>

      <record id="dep_rd" model="hr.department">
          <field name="name">Research &amp; Development</field>
      </record>

      <record id="dep_rd_be" model="hr.department">
          <field name="name">R&amp;D USA</field>
          <field name="parent_id" ref="dep_rd"/>
      </record>

      <record id="dep_rd_ltp" model="hr.department">
          <field name="name">Long Term Projects</field>
          <field name="parent_id" ref="dep_rd_be"/>
      </record>

      <record id="dep_ps" model="hr.department">
          <field name="name">Professional Services</field>
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
          <field name="description">You will take part in the consulting services we provide to our partners and customers : design, analysis, development, testing, project management, support/coaching. You will work autonomously as well as coordinate and supervise small distributed development teams for some projects. Optionally, you will deliver Odoo training sessions to partners and customers (8-10 people/session). You will report to the Head of Professional Services and work closely with all developers and consultants.

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
          <field name="address_id" ref="base.main_partner" />
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

      <!-- Address -->

      <record id="hr.res_partner_admin_private_address" model="res.partner">
          <field name="name">Mitchell Admin</field>
          <field name="street">215 Vine St</field>
          <field name="city">Scranton</field>
          <field name="zip">18503</field>
          <field name="country_id" ref="base.us"/>
          <field name='state_id' ref="base.state_us_39"/>
          <field name="phone">+1 555-555-5555</field>
          <field name="email">admin@yourcompany.example.com</field>
          <field name="create_date">2010-01-01 00:00:00</field>
          <field name="type">private</field>
      </record>
      <record id="res_partner_demo_private_address" model="res.partner">
          <field name="name">Mark Demo</field>
          <field name="street">361-7936 Feugiat St.</field>
          <field name="zip">58521</field>
          <field name="city">Williston</field>
          <field name="country_id" ref="base.us"/>
          <field name="phone">+1 555-555-5757</field>
          <field name="email">demo@yourcompany.example.com</field>
          <field name="type">private</field>
          <field name="create_date">2010-01-01 00:00:00</field>
      </record>

    <!--Employees-->

      <record id="employee_admin" model="hr.employee">
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(237)-125-2389</field>
          <field name="work_email">admin@yourcompany.example.com</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4'), ref('employee_category_3')])]"/>
          <field name="job_id" ref="hr.job_ceo"/>
          <field name="job_title">Chief Executive Officer</field>
          <field name="department_id" ref="dep_management"/>
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
          <field name="job_id" ref="hr.job_cto"/>
          <field name="job_title">Chief Technical Officer</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4'), ref('employee_category_3')])]"/>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(376)-310-7863</field>
          <field name="work_contact_id" ref="hr.work_contact_al"/>
          <field name="image_1920" type="base64" file="hr/static/img/employee_al-image.jpg"/>
          <field name="create_date">2010-01-01 00:00:00</field>
      </record>

      <record id="work_contact_mit" model="res.partner">
          <field name="name">Anita Oliver</field>
          <field name="mobile">(538)-672-3185</field>
          <field name="email">anita.oliver32@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_mit-image.jpg"/>
      </record>

        <record id="res_partner_mit_private_address" model="res.partner">
            <field name="name">Anital Oliver</field>
            <field name="country_id" ref="base.us"/>
            <field name="mobile">(538)-672-3185</field>
            <field name="email">anita.oliver00@example.com</field>
            <field name="type">private</field>
        </record>

      <record id="employee_mit" model="hr.employee">
          <field name="name">Anita Oliver</field>
          <field name="department_id" ref="dep_rd_be"/>
          <field name="parent_id" ref="employee_al"/>
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4')])]"/>
          <field name="address_home_id" ref="hr.res_partner_mit_private_address"/>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(538)-497-4804</field>
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
          <field name="parent_id" ref="employee_al"/>
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4')])]"/>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(450)-719-4182</field>
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
          <field name="parent_id" ref="employee_al"/>
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4')])]"/>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(332)-775-6660</field>
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
          <field name="parent_id" ref="employee_al"/>
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4')])]"/>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(157)-363-8229</field>
          <field name="work_contact_id" ref="hr.work_contact_chs"/>
          <field name="image_1920" type="base64" file="hr/static/img/employee_chs-image.jpg"/>
          <field name="create_date">2010-01-01 00:00:00</field>
      </record>

      <record id="work_contact_qdp" model="res.partner">
          <field name="name">Marc Demo</field>
          <field name="email">gilles@odoo.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_qdp-image.png"/>
      </record>

      <record id="employee_qdp" model="hr.employee">
          <field name="name">Marc Demo</field>
          <field name="user_id" ref="base.user_demo"/>
          <field name="department_id" ref="dep_rd"/>
          <field name="parent_id" ref="employee_admin"/>
          <field name="address_home_id" ref="res_partner_demo_private_address"/>
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4')])]"/>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">+3281813700</field>
          <field name="work_contact_id" ref="hr.work_contact_qdp"/>
          <field name="image_1920" type="base64" file="hr/static/img/employee_qdp-image.png"/>
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
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4')])]"/>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(449)-505-5146</field>
          <field name="work_contact_id" ref="hr.work_contact_fme"/>
          <field name="image_1920" type="base64" file="hr/static/img/employee_fme-image.jpg"/>
          <field name="create_date">2010-01-01 00:00:00</field>
      </record>

        <record id="res_partner_fpi_private_address" model="res.partner">
            <field name="name">Audrey Peterson</field>
            <field name="country_id" ref="base.us"/>
            <field name="email">Audrey.peterson2020@example.com</field>
            <field name="type">private</field>
        </record>

      <record id="work_contact_fpi" model="res.partner">
          <field name="name">Audrey Peterson</field>
          <field name="email">audrey.peterson25@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_fpi-image.jpg"/>
      </record>

      <record id="employee_fpi" model="hr.employee">
          <field name="name">Audrey Peterson</field>
          <field name="department_id" ref="dep_ps"/>
          <field name="parent_id" ref="employee_admin"/>
          <field name="job_id" ref="hr.job_consultant"/>
          <field name="job_title">Consultant</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4'), ref('employee_category_5')])]"/>
          <field name="address_home_id" ref="hr.res_partner_fpi_private_address"/>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(203)-276-7903</field>
          <field name="work_contact_id" ref="hr.work_contact_fpi"/>
          <field name="image_1920" type="base64" file="hr/static/img/employee_fpi-image.jpg"/>
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
          <field name="parent_id" ref="employee_admin"/>
          <field name="job_id" ref="hr.job_consultant"/>
          <field name="job_title">Consultant</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4'), ref('employee_category_5')])]"/>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(663)-707-8451</field>
          <field name="work_contact_id" ref="hr.work_contact_jth"/>
          <field name="image_1920" type="base64" file="hr/static/img/employee_jth-image.jpg"/>
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
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4'), ref('employee_category_2')])]"/>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(916)-264-7362</field>
          <field name="work_contact_id" ref="hr.work_contact_ngh"/>
          <field name="image_1920" type="base64" file="hr/static/img/employee_ngh-image.jpg"/>
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
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4')])]"/>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(360)-694-7266</field>
          <field name="work_contact_id" ref="hr.work_contact_vad"/>
          <field name="image_1920" type="base64" file="hr/static/img/employee_vad-image.jpg"/>
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
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(350)-912-1201</field>
          <field name="work_contact_id" ref="hr.work_contact_han"/>
          <field name="image_1920" type="base64" file="hr/static/img/employee_han-image.jpg"/>
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
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(114)-262-1607</field>
          <field name="work_contact_id" ref="hr.work_contact_jve"/>
          <field name="image_1920" type="base64" file="hr/static/img/employee_jve-image.jpg"/>
          <field name="create_date">2010-01-01 00:00:00</field>
      </record>

        <record id="res_partner_jep_private_address" model="res.partner">
            <field name="name">Doris Cole</field>
            <field name="country_id" ref="base.us"/>
            <field name="email">Doris.cole.LoveSong@example.com</field>
            <field name="type">private</field>
        </record>

      <record id="work_contact_jep" model="res.partner">
          <field name="name">Doris Cole</field>
          <field name="email">doris.cole31@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_jep-image.jpg"/>
      </record>

      <record id="employee_jep" model="hr.employee">
          <field name="name">Doris Cole</field>
          <field name="department_id" ref="dep_ps"/>
          <field name="job_id" ref="hr.job_consultant"/>
          <field name="job_title">Consultant</field>
          <field name="address_home_id" ref="hr.res_partner_jep_private_address"/>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(883)-331-5378</field>
          <field name="work_contact_id" ref="hr.work_contact_jep"/>
          <field name="image_1920" type="base64" file="hr/static/img/employee_jep-image.jpg"/>
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
          <field name="job_id" ref="hr.job_marketing"/>
          <field name="job_title">Marketing and Community Manager</field>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(206)-267-3735</field>
          <field name="work_contact_id" ref="hr.work_contact_jod"/>
          <field name="image_1920" type="base64" file="hr/static/img/employee_jod-image.jpg"/>
          <field name="create_date">2010-01-01 00:00:00</field>
      </record>

        <record id="res_partner_jog_private_address" model="res.partner">
            <field name="name">Beth Evans</field>
            <field name="country_id" ref="base.us"/>
            <field name="email">beth.evans@example.com</field>
            <field name="type">private</field>
        </record>

      <record id="work_contact_jog" model="res.partner">
          <field name="name">Beth Evans</field>
          <field name="email">beth.evans77@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_jog-image.jpg"/>
      </record>

      <record id="employee_jog" model="hr.employee">
          <field name="name">Beth Evans</field>
          <field name="department_id" ref="dep_rd"/>
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="address_home_id" ref="hr.res_partner_jog_private_address"/>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(754)-532-3841</field>
          <field name="work_contact_id" ref="hr.work_contact_jog"/>
          <field name="image_1920" type="base64" file="hr/static/img/employee_jog-image.jpg"/>
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
          <field name="job_id" ref="hr.job_consultant"/>
          <field name="job_title">Consultant</field>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(944)-518-8232</field>
          <field name="work_contact_id" ref="hr.work_contact_jgo"/>
          <field name="image_1920" type="base64" file="hr/static/img/employee_jgo-image.jpg"/>
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
          <field name="job_id" ref="hr.job_marketing"/>
          <field name="job_title">Marketing and Community Manager</field>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(644)-169-1352</field>
          <field name="work_contact_id" ref="hr.work_contact_lur"/>
          <field name="image_1920" type="base64" file="hr/static/img/employee_lur-image.jpg"/>
          <field name="create_date">2010-01-01 00:00:00</field>
      </record>

        <record id="res_partner_hne_private_address" model="res.partner">
            <field name="name">Abigail Peterson</field>
            <field name="country_id" ref="base.us"/>
            <field name="email">abigail.peterson33@example.com</field>
            <field name="type">private</field>
        </record>

      <record id="work_contact_hne" model="res.partner">
          <field name="name">Abigail Peterson</field>
          <field name="email">abigail.peterson39@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_hne-image.jpg"/>
      </record>

      <record id="employee_hne" model="hr.employee">
          <field name="name">Abigail Peterson</field>
          <field name="department_id" ref="dep_ps"/>
          <field name="job_id" ref="hr.job_consultant"/>
          <field name="job_title">Consultant</field>
          <field name="address_home_id" ref="hr.res_partner_hne_private_address"/>
          <field name="work_location_id" ref="work_location_1"/>
          <field name="work_phone">(482)-233-3393</field>
          <field name="work_contact_id" ref="hr.work_contact_hne"/>
          <field name="image_1920" type="base64" file="hr/static/img/employee_hne-image.jpg"/>
          <field name="create_date">2010-01-01 00:00:00</field>
          <field name="marital">married</field>
      </record>
    </data>
</odoo>

```

## File: models\hr_contract_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ContractType(models.Model):
    _name = 'hr.contract.type'
    _description = 'Contract Type'
    _order = 'sequence'

    name = fields.Char(required=True, translate=True)
    sequence = fields.Integer()

```

## File: models\hr_department.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class Department(models.Model):
    _name = "hr.department"
    _description = "Department"
    _inherit = ['mail.thread']
    _order = "name"
    _rec_name = 'complete_name'
    _parent_store = True

    name = fields.Char('Department Name', required=True)
    complete_name = fields.Char('Complete Name', compute='_compute_complete_name', recursive=True, store=True)
    active = fields.Boolean('Active', default=True)
    company_id = fields.Many2one('res.company', string='Company', index=True, default=lambda self: self.env.company)
    parent_id = fields.Many2one('hr.department', string='Parent Department', index=True, domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    child_ids = fields.One2many('hr.department', 'parent_id', string='Child Departments')
    manager_id = fields.Many2one('hr.employee', string='Manager', tracking=True, domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    member_ids = fields.One2many('hr.employee', 'department_id', string='Members', readonly=True)
    total_employee = fields.Integer(compute='_compute_total_employee', string='Total Employee')
    jobs_ids = fields.One2many('hr.job', 'department_id', string='Jobs')
    plan_ids = fields.One2many('hr.plan', 'department_id')
    plans_count = fields.Integer(compute='_compute_plan_count')
    note = fields.Text('Note')
    color = fields.Integer('Color Index')
    parent_path = fields.Char(index=True, unaccent=False)
    master_department_id = fields.Many2one(
        'hr.department', 'Master Department', compute='_compute_master_department_id', store=True)

    def name_get(self):
        if not self.env.context.get('hierarchical_naming', True):
            return [(record.id, record.name) for record in self]
        return super(Department, self).name_get()

    @api.model
    def name_create(self, name):
        return self.create({'name': name}).name_get()[0]

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
        emp_data = self.env['hr.employee']._read_group([('department_id', 'in', self.ids)], ['department_id'], ['department_id'])
        result = dict((data['department_id'][0], data['department_id_count']) for data in emp_data)
        for department in self:
            department.total_employee = result.get(department.id, 0)

    def _compute_plan_count(self):
        plans_data = self.env['hr.plan']._read_group([('department_id', 'in', self.ids)], ['department_id'], ['department_id'])
        plans_count = {x['department_id'][0]: x['department_id_count'] for x in plans_data}
        for department in self:
            department.plans_count = plans_count.get(department.id, 0)

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
                'view_type': 'kanban',
                'view_mode': 'kanban',
                'views': [(False, 'kanban'), (False, 'form')],
                'context': {'searchpanel_default_department_id': self.id},
                'res_id': False,
            })
        return res

    def action_plan_from_department(self):
        action = self.env['ir.actions.actions']._for_xml_id('hr.hr_plan_action')
        action['context'] = {'default_department_id': self.id, 'search_default_department_id': self.id}
        return action

    def get_children_department_ids(self):
        return self.env['hr.department'].search([('id', 'child_of', self.ids)])

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
    # YTI TODO: Move reason_code to hr + adapt _unlink_except_default_departure_reasons
    # to use the codes instead of refs

    def _get_default_departure_reasons(self):
        return {
            'fired': 342,
            'resigned': 343,
            'retired': 340,
        }

    @api.ondelete(at_uninstall=False)
    def _unlink_except_default_departure_reasons(self):
        master_reasons = [self.env.ref('hr.departure_fired', False), self.env.ref('hr.departure_resigned', False), self.env.ref('hr.departure_retired', False)]
        if any(reason in master_reasons for reason in self):
            raise UserError(_('Default departure reasons cannot be deleted.'))

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import pytz
from pytz import UTC
from datetime import datetime, time
from random import choice
from string import digits
from werkzeug.urls import url_encode
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError, AccessError
from odoo.osv import expression
from odoo.tools import format_date, Query


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
    _inherit = ['hr.employee.base', 'mail.thread', 'mail.activity.mixin', 'resource.mixin', 'avatar.mixin']
    _mail_post_access = 'read'

    # resource and user
    # required on the resource, make sure required="True" set in the view
    name = fields.Char(string="Employee Name", related='resource_id.name', store=True, readonly=False, tracking=True)
    user_id = fields.Many2one('res.users', 'User', related='resource_id.user_id', store=True, readonly=False)
    user_partner_id = fields.Many2one(related='user_id.partner_id', related_sudo=False, string="User's partner")
    active = fields.Boolean('Active', related='resource_id.active', default=True, store=True, readonly=False)
    company_id = fields.Many2one('res.company', required=True)
    company_country_id = fields.Many2one('res.country', 'Company Country', related='company_id.country_id', readonly=True)
    company_country_code = fields.Char(related='company_country_id.code', depends=['company_country_id'], readonly=True)
    # private partner
    address_home_id = fields.Many2one(
        'res.partner', 'Address', help='Enter here the private address of the employee, not the one linked to your company.',
        groups="hr.group_hr_user", tracking=True,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    is_address_home_a_company = fields.Boolean(
        'The employee address has a company linked',
        compute='_compute_is_address_home_a_company',
    )
    private_email = fields.Char(related='address_home_id.email', string="Private Email", groups="hr.group_hr_user")
    lang = fields.Selection(related='address_home_id.lang', string="Lang", groups="hr.group_hr_user", readonly=False)
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
        domain="[('partner_id', '=', address_home_id), '|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        groups="hr.group_hr_user",
        tracking=True,
        help='Employee bank account to pay salaries')
    permit_no = fields.Char('Work Permit No', groups="hr.group_hr_user", tracking=True)
    visa_no = fields.Char('Visa No', groups="hr.group_hr_user", tracking=True)
    visa_expire = fields.Date('Visa Expiration Date', groups="hr.group_hr_user", tracking=True)
    work_permit_expiration_date = fields.Date('Work Permit Expiration Date', groups="hr.group_hr_user", tracking=True)
    has_work_permit = fields.Binary(string="Work Permit", groups="hr.group_hr_user", tracking=True)
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

    job_id = fields.Many2one(tracking=True)
    phone = fields.Char(related='address_home_id.phone', related_sudo=False, readonly=False, string="Private Phone", groups="hr.group_hr_user")
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
    departure_description = fields.Html(string="Additional Information", groups="hr.group_hr_user", copy=False, tracking=True)
    departure_date = fields.Date(string="Departure Date", groups="hr.group_hr_user", copy=False, tracking=True)
    message_main_attachment_id = fields.Many2one(groups="hr.group_hr_user")
    id_card = fields.Binary(string="ID Card Copy", groups="hr.group_hr_user")
    driving_license = fields.Binary(string="Driving License", groups="hr.group_hr_user")
    currency_id = fields.Many2one('res.currency', related='company_id.currency_id', readonly=True)

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
        for employee in self:
            avatar = employee._origin[image_field]
            if not avatar:
                if employee.user_id:
                    avatar = employee.user_id.sudo()[avatar_field]
                else:
                    avatar = base64.b64encode(employee._avatar_get_placeholder())
            employee[avatar_field] = avatar

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
            'context': {
                'default_create_employee_id': self.id,
                'default_name': self.name,
                'default_phone': self.work_phone,
                'default_mobile': self.mobile_phone,
                'default_login': self.work_email,
            }
        }

    def name_get(self):
        if self.check_access_rights('read', raise_exception=False):
            return super(HrEmployeePrivate, self).name_get()
        return self.env['hr.employee.public'].browse(self.ids).name_get()

    def _read(self, fields):
        if self.check_access_rights('read', raise_exception=False):
            return super(HrEmployeePrivate, self)._read(fields)

        # HACK: retrieve publicly available values from hr.employee.public and
        # copy them to the cache of self; non-public data will be missing from
        # cache, and interpreted as an access error
        self.flush_recordset(fields)
        public = self.env['hr.employee.public'].browse(self._ids)
        public.read(fields)
        for fname in fields:
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

    def read(self, fields, load='_classic_read'):
        if self.check_access_rights('read', raise_exception=False):
            return super(HrEmployeePrivate, self).read(fields, load=load)
        private_fields = set(fields).difference(self.env['hr.employee.public']._fields.keys())
        if private_fields:
            raise AccessError(_('The fields "%s" you try to read is not available on the public employee profile.') % (','.join(private_fields)))
        return self.env['hr.employee.public'].browse(self.ids).read(fields, load=load)

    @api.model
    def get_view(self, view_id=None, view_type='form', **options):
        if self.check_access_rights('read', raise_exception=False):
            return super().get_view(view_id, view_type, **options)
        return self.env['hr.employee.public'].get_view(view_id, view_type, **options)

    @api.model
    def get_views(self, views, options):
        if self.check_access_rights('read', raise_exception=False):
            return super().get_views(views, options)
        res = self.env['hr.employee.public'].get_views(views, options)
        res['models'].update({'hr.employee': res['models']['hr.employee.public']})
        return res

    @api.model
    def _search(self, args, offset=0, limit=None, order=None, count=False, access_rights_uid=None):
        """
            We override the _search because it is the method that checks the access rights
            This is correct to override the _search. That way we enforce the fact that calling
            search on an hr.employee returns a hr.employee recordset, even if you don't have access
            to this model, as the result of _search (the ids of the public employees) is to be
            browsed on the hr.employee model. This can be trusted as the ids of the public
            employees exactly match the ids of the related hr.employee.
        """
        if self.check_access_rights('read', raise_exception=False):
            return super(HrEmployeePrivate, self)._search(args, offset=offset, limit=limit, order=order, count=count, access_rights_uid=access_rights_uid)
        try:
            ids = self.env['hr.employee.public']._search(args, offset=offset, limit=limit, order=order, count=count, access_rights_uid=access_rights_uid)
        except ValueError:
            raise AccessError(_('You do not have access to this document.'))
        if not count and isinstance(ids, Query):
            # the result is expected from this table, so we should link tables
            ids = super(HrEmployeePrivate, self.sudo())._search([('id', 'in', ids)])
        return ids

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

    @api.onchange('user_id')
    def _onchange_user(self):
        if self.user_id:
            self.update(self._sync_user(self.user_id, (bool(self.image_1920))))
            if not self.name:
                self.name = self.user_id.name

    @api.onchange('resource_calendar_id')
    def _onchange_timezone(self):
        if self.resource_calendar_id and not self.tz:
            self.tz = self.resource_calendar_id.tz

    def _sync_user(self, user, employee_has_image=False):
        vals = dict(
            work_contact_id=user.partner_id.id,
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
        employees = super().create(vals_list)
        if self.env.context.get('salary_simulation'):
            return employees
        employee_departments = employees.department_id
        if employee_departments:
            self.env['mail.channel'].sudo().search([
                ('subscription_department_ids', 'in', employee_departments.ids)
            ])._subscribe_users_automatically()
        onboarding_notes_bodies = {}
        hr_root_menu = self.env.ref('hr.menu_hr_root')
        for employee in employees:
            employee._message_subscribe(employee.address_home_id.ids)
            # Launch onboarding plans
            url = '/web#%s' % url_encode({
                'action': 'hr.plan_wizard_action',
                'active_id': employee.id,
                'active_model': 'hr.employee',
                'menu_id': hr_root_menu.id,
            })
            onboarding_notes_bodies[employee.id] = _(
                '<b>Congratulations!</b> May I recommend you to setup an <a href="%s">onboarding plan?</a>',
                url,
            )
        employees._message_log_batch(onboarding_notes_bodies)
        return employees

    def write(self, vals):
        if 'address_home_id' in vals:
            address_home_id = vals['address_home_id']
            account_ids = vals.get('bank_account_id') or self.bank_account_id.ids
            if account_ids and address_home_id:
                self.env['res.partner.bank'].browse(account_ids).partner_id = address_home_id
            self.message_unsubscribe(self.address_home_id.ids)
            if address_home_id:
                self._message_subscribe([address_home_id])
        if 'user_id' in vals:
            # Update the profile pictures with user, except if provided 
            vals.update(self._sync_user(self.env['res.users'].browse(vals['user_id']),
                                        (bool(all(emp.image_1920 for emp in self)))))
        if 'work_permit_expiration_date' in vals:
            vals['work_permit_scheduled_activity'] = False
        res = super(HrEmployeePrivate, self).write(vals)
        if vals.get('department_id') or vals.get('user_id'):
            department_id = vals['department_id'] if vals.get('department_id') else self[:1].department_id.id
            # When added to a department or changing user, subscribe to the channels auto-subscribed by department
            self.env['mail.channel'].sudo().search([
                ('subscription_department_ids', 'in', department_id)
            ])._subscribe_users_automatically()
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
        archived_addresses = unarchived_employees.mapped('address_home_id').filtered(lambda addr: not addr.active)
        archived_addresses.toggle_active()

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

    @api.depends('address_home_id', 'user_partner_id')
    def _compute_related_contacts(self):
        super()._compute_related_contacts()
        for employee in self:
            employee.related_contact_ids |= employee.address_home_id | employee.user_partner_id

    @api.depends('address_home_id.parent_id')
    def _compute_is_address_home_a_company(self):
        """Checks that chosen address (res.partner) is not linked to a company.
        """
        for employee in self:
            try:
                employee.is_address_home_a_company = employee.address_home_id.parent_id.id is not False
            except AccessError:
                employee.is_address_home_a_company = False

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

    # ---------------------------------------------------------
    # Business Methods
    # ---------------------------------------------------------

    @api.model
    def get_import_templates(self):
        return [{
            'label': _('Import Template for Employees'),
            'template': '/hr/static/xls/hr_employee.xls'
        }]

    def _post_author(self):
        """
        When a user updates his own employee's data, all operations are performed
        by super user. However, tracking messages should not be posted as OdooBot
        but as the actual user.
        This method is used in the overrides of `_message_log` and `message_post`
        to post messages as the correct user.
        """
        real_user = self.env.context.get('binary_field_real_user')
        if self.env.is_superuser() and real_user:
            self = self.with_user(real_user)
        return self

    def _get_unusual_days(self, date_from, date_to=None):
        # Checking the calendar directly allows to not grey out the leaves taken
        # by the employee or fallback to the company calendar
        return (self.resource_calendar_id or self.env.company.resource_calendar_id)._get_unusual_days(
            datetime.combine(fields.Date.from_string(date_from), time.min).replace(tzinfo=UTC),
            datetime.combine(fields.Date.from_string(date_to), time.max).replace(tzinfo=UTC)
        )

    def _get_expected_attendances(self, date_from, date_to, domain=None):
        self.ensure_one()
        employee_timezone = pytz.timezone(self.tz) if self.tz else None
        calendar = self.resource_calendar_id or self.company_id.resource_calendar_id
        calendar_intervals = calendar._work_intervals_batch(
            date_from,
            date_to,
            tz=employee_timezone,
            resources=self.resource_id,
            compute_leaves=True,
            domain=domain)[self.resource_id.id]
        return calendar_intervals

    # ---------------------------------------------------------
    # Messaging
    # ---------------------------------------------------------

    def _message_log(self, **kwargs):
        return super(HrEmployeePrivate, self._post_author())._message_log(**kwargs)

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, **kwargs):
        return super(HrEmployeePrivate, self._post_author()).message_post(**kwargs)

    def _sms_get_partner_fields(self):
        return ['user_partner_id']

    def _sms_get_number_fields(self):
        return ['mobile_phone']

```

## File: models\hr_employee_base.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval

from pytz import timezone, UTC, utc
from datetime import timedelta

from odoo import _, api, fields, models
from odoo.exceptions import UserError
from odoo.tools import clean_context, format_time


class HrEmployeeBase(models.AbstractModel):
    _name = "hr.employee.base"
    _description = "Basic Employee"
    _order = 'name'

    name = fields.Char()
    active = fields.Boolean("Active")
    color = fields.Integer('Color Index', default=0)
    department_id = fields.Many2one('hr.department', 'Department', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    member_of_department = fields.Boolean("Member of department", compute='_compute_part_of_department', search='_search_part_of_department',
        help="Whether the employee is a member of the active user's department or one of it's child department.")
    job_id = fields.Many2one('hr.job', 'Job Position', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    job_title = fields.Char("Job Title", compute="_compute_job_title", store=True, readonly=False)
    company_id = fields.Many2one('res.company', 'Company')
    address_id = fields.Many2one('res.partner', 'Work Address', compute="_compute_address_id", store=True, readonly=False,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    work_phone = fields.Char('Work Phone', compute="_compute_phones", store=True, readonly=False)
    mobile_phone = fields.Char('Work Mobile', compute="_compute_work_contact_details", store=True, inverse='_inverse_work_contact_details')
    work_email = fields.Char('Work Email', compute="_compute_work_contact_details", store=True, inverse='_inverse_work_contact_details')
    work_contact_id = fields.Many2one('res.partner', 'Work Contact', copy=False)
    related_contact_ids = fields.Many2many('res.partner', string='Related Contacts', compute='_compute_related_contacts')
    related_contacts_count = fields.Integer('Number of related contacts', compute='_compute_related_contacts_count')
    work_location_id = fields.Many2one('hr.work.location', 'Work Location', compute="_compute_work_location_id", store=True, readonly=False,
    domain="[('address_id', '=', address_id), '|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    user_id = fields.Many2one('res.users')
    resource_id = fields.Many2one('resource.resource')
    resource_calendar_id = fields.Many2one('resource.calendar', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    parent_id = fields.Many2one('hr.employee', 'Manager', compute="_compute_parent_id", store=True, readonly=False,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    coach_id = fields.Many2one(
        'hr.employee', 'Coach', compute='_compute_coach', store=True, readonly=False,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]",
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
    employee_type = fields.Selection([
        ('employee', 'Employee'),
        ('student', 'Student'),
        ('trainee', 'Trainee'),
        ('contractor', 'Contractor'),
        ('freelance', 'Freelancer'),
        ], string='Employee Type', default='employee', required=True,
        help="The employee type. Although the primary purpose may seem to categorize employees, this field has also an impact in the Contract History. Only Employee type is supposed to be under contract and will have a Contract History.")


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

    def _inverse_work_contact_details(self):
        for employee in self:
            if not employee.work_contact_id:
                employee.work_contact_id = self.env['res.partner'].sudo().with_context(clean_context(self._context)).create({
                    'email': employee.work_email,
                    'mobile': employee.mobile_phone,
                    'name': employee.name,
                    'image_1920': employee.image_1920,
                    'company_id': employee.company_id.id
                })
            else:
                employee.work_contact_id.sudo().write({
                    'email': employee.work_email,
                    'mobile': employee.mobile_phone,
                })

    @api.depends('work_contact_id')
    def _compute_related_contacts(self):
        for employee in self:
            employee.related_contact_ids = employee.work_contact_id

    @api.depends('related_contact_ids')
    def _compute_related_contacts_count(self):
        for employee in self:
            employee.related_contacts_count = len(employee.related_contact_ids)

    def action_related_contacts(self):
        self.ensure_one()
        return {
            'name': _("Related Contacts"),
            'type': 'ir.actions.act_window',
            'view_mode': 'kanban,tree,form',
            'res_model': 'res.partner',
            'domain': [('id', 'in', self.related_contact_ids.ids)]
        }

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

    @api.depends('address_id')
    def _compute_work_location_id(self):
        to_reset = self.filtered(lambda e: e.address_id != e.work_location_id.address_id)
        to_reset.work_location_id = False

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
        ('name_uniq', 'unique (name)', "Tag name already exists !"),
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
    related_contact_ids = fields.Many2many(readonly=True)
    work_location_id = fields.Many2one(readonly=True)
    user_id = fields.Many2one(readonly=True)
    resource_id = fields.Many2one(readonly=True)
    resource_calendar_id = fields.Many2one(readonly=True)
    tz = fields.Selection(readonly=True)
    color = fields.Integer(readonly=True)
    employee_type = fields.Selection(readonly=True)

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

    def _search_employee_id(self, operator, value):
        return [('id', operator, value)]

    def _compute_employee_id(self):
        for employee in self:
            employee.employee_id = self.env['hr.employee'].browse(employee.id)

    @api.depends('user_partner_id')
    def _compute_related_contacts(self):
        super()._compute_related_contacts()
        for employee in self:
            employee.related_contact_ids |= employee.user_partner_id

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
from odoo.addons.web_editor.controllers.main import handle_history_divergence


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
    description = fields.Html(string='Job Description', sanitize_attributes=False)
    requirements = fields.Text('Requirements')
    department_id = fields.Many2one('hr.department', string='Department', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    company_id = fields.Many2one('res.company', string='Company', default=lambda self: self.env.company)
    contract_type_id = fields.Many2one('hr.contract.type', string='Employment Type')

    _sql_constraints = [
        ('name_company_uniq', 'unique(name, company_id, department_id)', 'The name of the job position must be unique per department in company!'),
        ('no_of_recruitment_positive', 'CHECK(no_of_recruitment >= 0)', 'The expected number of new employees must be positive.')
    ]

    @api.depends('no_of_recruitment', 'employee_ids.job_id', 'employee_ids.active')
    def _compute_employees(self):
        employee_data = self.env['hr.employee']._read_group([('job_id', 'in', self.ids)], ['job_id'], ['job_id'])
        result = dict((data['job_id'][0], data['job_id_count']) for data in employee_data)
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
            default['name'] = _("%s (copy)") % (self.name)
        return super(Job, self).copy(default=default)

    def write(self, vals):
        if len(self) == 1:
            handle_history_divergence(self, 'description', vals)
        return super(Job, self).write(vals)

```

## File: models\hr_plan.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class HrPlan(models.Model):
    _name = 'hr.plan'
    _description = 'plan'

    name = fields.Char('Name', required=True)
    company_id = fields.Many2one(
        'res.company', default=lambda self: self.env.company)
    department_id = fields.Many2one('hr.department', check_company=True)
    plan_activity_type_ids = fields.One2many(
        'hr.plan.activity.type', 'plan_id',
        string='Activities',
        domain="[('company_id', '=', company_id)]")
    active = fields.Boolean(default=True)
    steps_count = fields.Integer(compute='_compute_steps_count')

    @api.depends('plan_activity_type_ids')
    def _compute_steps_count(self):
        activity_type_data = self.env['hr.plan.activity.type']._read_group([('plan_id', 'in', self.ids)], ['plan_id'], ['plan_id'])
        steps_count = {x['plan_id'][0]: x['plan_id_count'] for x in activity_type_data}
        for plan in self:
            plan.steps_count = steps_count.get(plan.id, 0)

```

## File: models\hr_plan_activity_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class HrPlanActivityType(models.Model):
    _name = 'hr.plan.activity.type'
    _description = 'Plan activity type'
    _rec_name = 'summary'
    _check_company_auto = True

    company_id = fields.Many2one('res.company', default=lambda self: self.env.company)
    activity_type_id = fields.Many2one(
        'mail.activity.type', 'Activity Type',
        default=lambda self: self.env.ref('mail.mail_activity_data_todo'),
        domain=lambda self: ['|', ('res_model', '=', False), ('res_model', '=', 'hr.employee')],
        ondelete='restrict'
    )
    summary = fields.Char('Summary', compute="_compute_default_summary", store=True, readonly=False)
    responsible = fields.Selection([
        ('coach', 'Coach'),
        ('manager', 'Manager'),
        ('employee', 'Employee'),
        ('other', 'Other')], default='employee', string='Responsible', required=True)
    responsible_id = fields.Many2one(
        'res.users',
        'Other Responsible',
        check_company=True,
        help='Specific responsible of activity if not linked to the employee.')
    plan_id = fields.Many2one('hr.plan')
    note = fields.Html('Note')

    @api.depends('activity_type_id')
    def _compute_default_summary(self):
        for plan_type in self:
            if plan_type.activity_type_id and plan_type.activity_type_id.summary:
                plan_type.summary = plan_type.activity_type_id.summary
            else:
                plan_type.summary = False

    def get_responsible_id(self, employee):
        warning = False
        if self.responsible == 'coach':
            if not employee.coach_id:
                warning = _('Coach of employee %s is not set.', employee.name)
            responsible = employee.coach_id.user_id
            if employee.coach_id and not responsible:
                warning = _("The user of %s's coach is not set.", employee.name)
        elif self.responsible == 'manager':
            if not employee.parent_id:
                warning = _('Manager of employee %s is not set.', employee.name)
            responsible = employee.parent_id.user_id
            if employee.parent_id and not responsible:
                warning = _("The manager of %s should be linked to a user.", employee.name)
        elif self.responsible == 'employee':
            responsible = employee.user_id
            if not responsible:
                warning = _('The employee %s should be linked to a user.', employee.name)
        elif self.responsible == 'other':
            responsible = self.responsible_id
            if not responsible:
                warning = _('No specific user given on activity %s.', self.activity_type_id.name)
        return {
            'responsible': responsible,
            'warning': warning,
        }

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
    address_id = fields.Many2one('res.partner', required=True, string="Work Address", domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
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

## File: models\mail_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError


class Channel(models.Model):
    _inherit = 'mail.channel'

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

## File: models\models.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, tools, _


class BaseModel(models.AbstractModel):
    _inherit = 'base'

    def _alias_get_error_message(self, message, message_dict, alias):
        if alias.alias_contact == 'employees':
            email_from = tools.decode_message_header(message, 'From')
            email_address = tools.email_split(email_from)[0]
            employee = self.env['hr.employee'].search([('work_email', 'ilike', email_address)], limit=1)
            if not employee:
                employee = self.env['hr.employee'].search([('user_id.email', 'ilike', email_address)], limit=1)
            if not employee:
                return _('restricted to employees')
            return False
        return super(BaseModel, self)._alias_get_error_message(message, message_dict, alias)

```

## File: models\resource.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from pytz import utc, timezone
from datetime import datetime

from odoo import fields, models
from odoo.addons.resource.models.resource import Intervals


class ResourceResource(models.Model):
    _inherit = "resource.resource"

    user_id = fields.Many2one(copy=False)
    employee_id = fields.One2many('hr.employee', 'resource_id', domain="[('company_id', '=', company_id)]")

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
    module_hr_homeworking = fields.Boolean(string="Homeworking")
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
from odoo.exceptions import AccessError


class Partner(models.Model):
    _inherit = ['res.partner']

    employee_ids = fields.One2many(
        'hr.employee', 'address_home_id', string='Employees', groups="hr.group_hr_user",
        help="Related employees based on their private address")
    employees_count = fields.Integer(compute='_compute_employees_count', groups="hr.group_hr_user")

    def name_get(self):
        """ Override to allow an employee to see its private address in his profile.
            This avoids to relax access rules on `res.parter` and to add an `ir.rule`.
            (advantage in both security and performance).
            Use a try/except instead of systematically checking to minimize the impact on performance.
            """
        try:
            return super(Partner, self).name_get()
        except AccessError as e:
            if len(self) == 1 and self in self.env.user.employee_ids.mapped('address_home_id'):
                return super(Partner, self.sudo()).name_get()
            raise e

    def _compute_employees_count(self):
        for partner in self:
            partner.employees_count = len(partner.employee_ids)

    def action_open_employees(self):
        self.ensure_one()
        return {
            'name': _('Related Employees'),
            'type': 'ir.actions.act_window',
            'res_model': 'hr.employee',
            'view_mode': 'kanban,tree,form',
            'domain': [('id', 'in', self.employee_ids.ids)],
        }

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

from odoo import api, models, fields, _, SUPERUSER_ID
from odoo.exceptions import AccessError
from odoo.tools.misc import clean_context


HR_READABLE_FIELDS = [
    'active',
    'child_ids',
    'employee_id',
    'address_home_id',
    'employee_ids',
    'employee_parent_id',
    'hr_presence_state',
    'last_activity',
    'last_activity_time',
    'can_edit',
    'is_system',
    'employee_resource_calendar_id',
]

HR_WRITABLE_FIELDS = [
    'additional_note',
    'private_street',
    'private_street2',
    'private_city',
    'private_state_id',
    'private_zip',
    'private_country_id',
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
    'is_address_home_a_company',
    'job_title',
    'private_email',
    'km_home_work',
    'marital',
    'mobile_phone',
    'notes',
    'employee_parent_id',
    'passport_id',
    'permit_no',
    'employee_phone',
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
    employee_phone = fields.Char(related='employee_id.phone', readonly=False, related_sudo=False)
    work_email = fields.Char(related='employee_id.work_email', readonly=False, related_sudo=False)
    category_ids = fields.Many2many(related='employee_id.category_ids', string="Employee Tags", readonly=False, related_sudo=False)
    department_id = fields.Many2one(related='employee_id.department_id', readonly=False, related_sudo=False)
    address_id = fields.Many2one(related='employee_id.address_id', readonly=False, related_sudo=False)
    work_location_id = fields.Many2one(related='employee_id.work_location_id', readonly=False, related_sudo=False)
    employee_parent_id = fields.Many2one(related='employee_id.parent_id', readonly=False, related_sudo=False)
    coach_id = fields.Many2one(related='employee_id.coach_id', readonly=False, related_sudo=False)
    address_home_id = fields.Many2one(related='employee_id.address_home_id', readonly=False, related_sudo=False)
    private_street = fields.Char(related='address_home_id.street', string="Private Street", readonly=False, related_sudo=False)
    private_street2 = fields.Char(related='address_home_id.street2', string="Private Street2", readonly=False, related_sudo=False)
    private_city = fields.Char(related='address_home_id.city', string="Private City", readonly=False, related_sudo=False)
    private_state_id = fields.Many2one(
        related='address_home_id.state_id', string="Private State", readonly=False, related_sudo=False,
        domain="[('country_id', '=?', private_country_id)]")
    private_zip = fields.Char(related='address_home_id.zip', readonly=False, string="Private Zip", related_sudo=False)
    private_country_id = fields.Many2one(related='address_home_id.country_id', string="Private Country", readonly=False, related_sudo=False)
    is_address_home_a_company = fields.Boolean(related='employee_id.is_address_home_a_company', readonly=False, related_sudo=False)
    private_email = fields.Char(related='address_home_id.email', string="Private Email", readonly=False)
    private_lang = fields.Selection(related='address_home_id.lang', string="Employee Lang", readonly=False)
    km_home_work = fields.Integer(related='employee_id.km_home_work', readonly=False, related_sudo=False)
    # res.users already have a field bank_account_id and country_id from the res.partner inheritance: don't redefine them
    employee_bank_account_id = fields.Many2one(related='employee_id.bank_account_id', string="Employee's Bank Account Number", related_sudo=False, readonly=False)
    employee_country_id = fields.Many2one(related='employee_id.country_id', string="Employee's Country", readonly=False, related_sudo=False)
    identification_id = fields.Char(related='employee_id.identification_id', readonly=False, related_sudo=False)
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

    create_employee = fields.Boolean(store=False, default=True, copy=False, string="Technical field, whether to create an employee")
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

    def write(self, vals):
        """
        Synchronize user and its related employee
        and check access rights if employees are not allowed to update
        their own data (otherwise sudo is applied for self data).
        """
        hr_fields = {
            field
            for field_name, field in self._fields.items()
            if field.related_field and field.related_field.model_name == 'hr.employee' and field_name in vals
        }
        can_edit_self = self.env['ir.config_parameter'].sudo().get_param('hr.hr_employee_self_edit') or self.env.user.has_group('hr.group_hr_user')
        if hr_fields and not can_edit_self:
            # Raise meaningful error message
            raise AccessError(_("You are only allowed to update your preferences. Please contact a HR officer to update other information."))

        result = super(User, self).write(vals)

        employee_values = {}
        for fname in [f for f in self._get_employee_fields_to_sync() if f in vals]:
            employee_values[fname] = vals[fname]

        if employee_values:
            if 'email' in employee_values:
                employee_values['work_email'] = employee_values.pop('email')
            if 'image_1920' in vals:
                without_image = self.env['hr.employee'].sudo().search([('user_id', 'in', self.ids), ('image_1920', '=', False)])
                with_image = self.env['hr.employee'].sudo().search([('user_id', 'in', self.ids), ('image_1920', '!=', False)])
                without_image.write(employee_values)
                if not can_edit_self:
                    employee_values.pop('image_1920')
                with_image.write(employee_values)
            else:
                employees = self.env['hr.employee'].sudo().search([('user_id', 'in', self.ids)])
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
from . import hr_plan
from . import hr_plan_activity_type
from . import hr_work_location
from . import mail_alias
from . import mail_channel
from . import models
from . import res_config_settings
from . import res_partner
from . import res_users
from . import res_company
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
        <field name="print_report_name">'Print Badge - %s' % (object.name).replace('/', '')</field>
        <field name="binding_model_id" ref="model_hr_employee"/>
        <field name="binding_type">report</field>
    </record>

    <template id="print_employee_badge">
        <t t-call="web.basic_layout">
            <div class="page">
                <t t-foreach="docs" t-as="employee">
                    <div class="col-md-6">
                        <table style="width:243pt; height:153pt; border: 1pt solid black; border-collapse:separate; border-radius:8pt; margin:5pt">
                            <td style="width:33%;" valign="center">
                                <table style="width:77pt; height:150pt" class="table-borderless">
                                    <tr style="height:30%">
                                        <td align="center" valign="center">
                                            <img t-if="employee.company_id.logo" t-att-src="image_data_uri(employee.company_id.logo)" style="max-height:45pt;max-width:90%" alt="Company Logo"/>
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
                                    <tr><th><div style="font-size:15pt; margin-bottom:0pt;margin-top:0pt;" align="center"><t t-esc="employee.name"/></div></th></tr>
                                    <tr><td><div align="center" style="font-size:10pt;margin-bottom:5pt;"><t t-esc="employee.job_id.name"/></div></td></tr>
                                    <tr><td><div t-if="employee.barcode" t-field="employee.barcode" t-options="{'widget': 'barcode', 'width': 600, 'height': 120, 'img_style': 'max-height:50pt;max-width:100%;', 'img_align': 'center'}"/></td></tr>
                                </table>
                            </td>
                        </table>
                    </div>
                </t>
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
        <field name="name">Officer : Manage all employees</field>
        <field name="category_id" ref="base.module_category_human_resources_employees"/>
        <field name="implied_ids" eval="[(6, 0, [ref('base.group_private_addresses'), ref('base.group_user')])]"/>
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

    <record id="hr_plan_company_rule" model="ir.rule">
        <field name="name">Plan multi-company rule</field>
        <field name="model_id" ref="model_hr_plan"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="hr_plan_activity_type_company_rule" model="ir.rule">
        <field name="name">Plan Activity Type multi-company rule</field>
        <field name="model_id" ref="model_hr_plan_activity_type"/>
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
access_hr_plan_activity_type_employee,access_hr_plan_activity_type,model_hr_plan_activity_type,base.group_user,1,0,0,0
access_hr_plan_employee,access_hr_plan_employee,model_hr_plan,base.group_user,1,0,0,0
access_hr_plan_activity_type_hr_user,access_hr_plan_activity_type,model_hr_plan_activity_type,group_hr_user,1,1,1,1
access_hr_plan_hr_user,access_hr_plan_hr_user,model_hr_plan,group_hr_user,1,1,1,1
access_hr_plan_wizard,access.hr.plan.wizard,model_hr_plan_wizard,hr.group_hr_user,1,1,1,0
access_hr_departure_wizard,access.hr.departure.wizard,model_hr_departure_wizard,hr.group_hr_user,1,1,1,0
access_hr_work_location_user,access_hr_work_location_user,model_hr_work_location,base.group_user,1,0,0,0
access_hr_work_location_manager,access_hr_work_location_manager,model_hr_work_location,group_hr_manager,1,1,1,1
access_hr_departure_reason,access_hr_departure_reason_user,model_hr_departure_reason,group_hr_user,1,1,1,1
access_hr_contract_type_manager,hr.contract.type.manager,model_hr_contract_type,hr.group_hr_user,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70">
    <defs>
        <path id="icon-a" d="M4,5.35309892e-14 C36.4160122,9.87060235e-15 58.0836068,-3.97961823e-14 65,5.07020818e-14 C69,6.733808e-14 70,1 70,5 C70,43.0488877 70,62.4235458 70,65 C70,69 69,70 65,70 C61,70 9,70 4,70 C1,70 7.10542736e-15,69 7.10542736e-15,65 C7.25721566e-15,62.4676575 3.83358709e-14,41.8005206 3.60818146e-14,5 C-1.13686838e-13,1 1,5.75716207e-14 4,5.35309892e-14 Z"/>
        <linearGradient id="icon-c" x1="100%" x2="0%" y1="0%" y2="100%">
            <stop offset="0%" stop-color="#269396"/>
            <stop offset="100%" stop-color="#218689"/>
        </linearGradient>
        <path id="icon-d" d="M35.5,22.75 C39.6529297,22.75 43.0195312,26.1166016 43.0195312,30.2695312 C43.0195312,34.4224609 39.6529297,37.7890625 35.5,37.7890625 C31.3470703,37.7890625 27.9804687,34.4224609 27.9804687,30.2695312 C27.9804687,26.1166016 31.3470703,22.75 35.5,22.75 Z M43.6256055,38.3165755 L40.7623112,37.6007161 C37.2411654,40.1333659 32.9730794,39.5681836 30.2377604,37.6007161 L27.3744661,38.3165755 C25.0790039,38.8904232 23.46875,40.9527799 23.46875,43.3188542 L23.46875,47.671875 C23.46875,49.0957161 24.6230339,50.25 26.046875,50.25 L44.953125,50.25 C46.3769661,50.25 47.53125,49.0957161 47.53125,47.671875 L47.53125,43.3188542 C47.53125,40.9527799 45.9209961,38.8904232 43.6256055,38.3165755 Z M50.3958333,39.6510417 C53.1644531,39.6510417 55.4088542,37.4066406 55.4088542,34.6380208 C55.4088542,31.869401 53.1644531,29.625 50.3958333,29.625 C47.6272135,29.625 45.3828125,31.869401 45.3828125,34.6380208 C45.3828125,37.4066406 47.6272135,39.6510417 50.3958333,39.6510417 Z M20.6041667,39.6510417 C23.3727865,39.6510417 25.6171875,37.4066406 25.6171875,34.6380208 C25.6171875,31.869401 23.3727865,29.625 20.6041667,29.625 C17.8355469,29.625 15.5911458,31.869401 15.5911458,34.6380208 C15.5911458,37.4066406 17.8355469,39.6510417 20.6041667,39.6510417 Z M22.3229167,47.671875 L22.3229167,43.3188542 C22.3229167,42.1335612 22.6518424,41.0125781 23.2326367,40.0533008 C21.0850586,41.1074674 18.6968555,40.6769206 17.0959831,39.5255013 L15.1870964,40.0027409 C13.6568359,40.3852344 12.5833333,41.7602344 12.5833333,43.3375456 L12.5833333,46.2395833 C12.5833333,47.1888346 13.352832,47.9583333 14.3020833,47.9583333 L22.3350195,47.9583333 C22.3273388,47.8630362 22.3233016,47.7674804 22.3229167,47.671875 Z M55.8129036,40.0026693 L53.9040169,39.5254297 C51.9041797,40.9638802 49.5434049,40.902793 47.7604883,40.0423437 C48.3454362,41.004056 48.6770833,42.1289779 48.6770833,43.3188542 L48.6770833,47.671875 C48.6770833,47.7683398 48.6722135,47.8636589 48.6649805,47.9583333 L56.6979167,47.9583333 C57.647168,47.9583333 58.4166667,47.1888346 58.4166667,46.2395833 L58.4166667,43.3375456 C58.4166667,41.7602344 57.3431641,40.3852344 55.8129036,40.0026693 Z"/>
        <path id="icon-e" d="M35.5,20.75 C39.6529297,20.75 43.0195312,24.1166016 43.0195312,28.2695312 C43.0195312,32.4224609 39.6529297,35.7890625 35.5,35.7890625 C31.3470703,35.7890625 27.9804687,32.4224609 27.9804687,28.2695312 C27.9804687,24.1166016 31.3470703,20.75 35.5,20.75 Z M43.6256055,36.3165755 L40.7623112,35.6007161 C37.2411654,38.1333659 32.9730794,37.5681836 30.2377604,35.6007161 L27.3744661,36.3165755 C25.0790039,36.8904232 23.46875,38.9527799 23.46875,41.3188542 L23.46875,45.671875 C23.46875,47.0957161 24.6230339,48.25 26.046875,48.25 L44.953125,48.25 C46.3769661,48.25 47.53125,47.0957161 47.53125,45.671875 L47.53125,41.3188542 C47.53125,38.9527799 45.9209961,36.8904232 43.6256055,36.3165755 Z M50.3958333,37.6510417 C53.1644531,37.6510417 55.4088542,35.4066406 55.4088542,32.6380208 C55.4088542,29.869401 53.1644531,27.625 50.3958333,27.625 C47.6272135,27.625 45.3828125,29.869401 45.3828125,32.6380208 C45.3828125,35.4066406 47.6272135,37.6510417 50.3958333,37.6510417 Z M20.6041667,37.6510417 C23.3727865,37.6510417 25.6171875,35.4066406 25.6171875,32.6380208 C25.6171875,29.869401 23.3727865,27.625 20.6041667,27.625 C17.8355469,27.625 15.5911458,29.869401 15.5911458,32.6380208 C15.5911458,35.4066406 17.8355469,37.6510417 20.6041667,37.6510417 Z M22.3229167,45.671875 L22.3229167,41.3188542 C22.3229167,40.1335612 22.6518424,39.0125781 23.2326367,38.0533008 C21.0850586,39.1074674 18.6968555,38.6769206 17.0959831,37.5255013 L15.1870964,38.0027409 C13.6568359,38.3852344 12.5833333,39.7602344 12.5833333,41.3375456 L12.5833333,44.2395833 C12.5833333,45.1888346 13.352832,45.9583333 14.3020833,45.9583333 L22.3350195,45.9583333 C22.3273388,45.8630362 22.3233016,45.7674804 22.3229167,45.671875 Z M55.8129036,38.0026693 L53.9040169,37.5254297 C51.9041797,38.9638802 49.5434049,38.902793 47.7604883,38.0423437 C48.3454362,39.004056 48.6770833,40.1289779 48.6770833,41.3188542 L48.6770833,45.671875 C48.6770833,45.7683398 48.6722135,45.8636589 48.6649805,45.9583333 L56.6979167,45.9583333 C57.647168,45.9583333 58.4166667,45.1888346 58.4166667,44.2395833 L58.4166667,41.3375456 C58.4166667,39.7602344 57.3431641,38.3852344 55.8129036,38.0026693 Z"/>
    </defs>
    <g fill="none" fill-rule="evenodd">
        <mask id="icon-b" fill="#fff">
            <use xlink:href="#icon-a"/>
        </mask>
        <g mask="url(#icon-b)">
            <rect width="70" height="70" fill="url(#icon-c)"/>
            <path fill="#FFF" fill-opacity=".383" d="M4,1.8 L65,1.8 C67.6666667,1.8 69.3333333,1.13333333 70,-0.2 C70,2.46666667 70,3.46666667 70,2.8 L1.10547097e-14,2.8 C-1.65952376e-14,3.46666667 -2.9161925e-14,2.46666667 -2.66453526e-14,-0.2 C0.666666667,1.13333333 2,1.8 4,1.8 Z" transform="matrix(1 0 0 -1 0 2.8)"/>
            <path fill="#393939" d="M44,47 L4,47 C2,47 -7.10542736e-15,46.8509317 0,42.826087 L1.81527147e-16,22.6291049 L17.2090667,6.04664397 L19.583071,9.5209307 L30.2767729,0.11143939 L40.9146315,10.270152 L46.6446282,6.41116033 L55.3045682,10.7749724 L52.3812234,16.1277957 L58.2417324,21.9036543 L44,47 Z" opacity=".324" transform="translate(0 23)"/>
            <path fill="#000" fill-opacity=".383" d="M4,4 L65,4 C67.6666667,4 69.3333333,3 70,1 C70,3.66666667 70,5 70,5 L1.77635684e-15,5 C1.77635684e-15,5 1.77635684e-15,3.66666667 1.77635684e-15,1 C0.666666667,3 2,4 4,4 Z" transform="translate(0 65)"/>
            <use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#icon-d"/>
            <use fill="#FFF" fill-rule="nonzero" xlink:href="#icon-e"/>
        </g>
    </g>
</svg>

```

## File: static\src\components\background_image\background_image.js

```javascript
/** @odoo-module */

import { registry } from '@web/core/registry';

import { ImageField } from '@web/views/fields/image/image_field';

export class BackgroundImageField extends ImageField {}
BackgroundImageField.template = 'hr.BackgroundImage';

registry.category("fields").add("background_image", BackgroundImageField);

```

## File: static\src\components\background_image\background_image.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="hr.BackgroundImage" owl="1">
        <div
            t-att-data-tooltip-template="hasTooltip and tooltipAttributes.template"
            t-att-data-tooltip-info="hasTooltip and tooltipAttributes.info"
            t-att-data-tooltip-delay="hasTooltip and props.zoomDelay"
            t-attf-style="background-image: url('#{getUrl(props.previewImage or props.name)}');"/>
    </t>
</templates>

```

## File: static\src\components\employee_chat\employee_chat.js

```javascript
/** @odoo-module */

import { registry } from '@web/core/registry';
import { standardWidgetProps } from "@web/views/widgets/standard_widget_props";

import { useOpenChat } from "@mail/views/open_chat_hook";

const { Component } = owl;

export class HrEmployeeChat extends Component {
    setup() {
        super.setup();
        this.openChat = useOpenChat(this.props.record.resModel);
    }
}
HrEmployeeChat.props = {
    ...standardWidgetProps,
};
HrEmployeeChat.template = 'hr.OpenChat';

registry.category("view_widgets").add("hr_employee_chat", HrEmployeeChat);

```

## File: static\src\components\employee_chat\employee_chat.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="hr.OpenChat" owl="1">
        <a t-if="props.record.data.user_id"
            title="Chat"
            icon="fa-comments"
            t-on-click.prevent="() => openChat(props.record.resId)"
            href="#"
            class="ml8 o_employee_chat_btn"
            role="button">
            <i class="fa fa-comments align-middle fs-6"/>
        </a>
    </t>

    <!-- TODO KBA: remove when Studio converted to owl -->
    <t t-name="hr.OpenChatLegacy">
        <a
            title="Chat"
            icon="fa-comments"
            href="#"
            class="ml8 o_employee_chat_btn"
            role="button">
            <i class="fa fa-comments align-middle fs-6"/>
        </a>
    </t>
</templates>

```

## File: static\src\components\employee_chat\employee_chat_legacy.js

```javascript
odoo.define('hr.OpenChatLegacy', function (require) {
"use strict";

const widgetRegistry = require('web.widget_registry');
const Widget = require('web.Widget');

const HrEmployeeChatLegacy = Widget.extend({
    template: 'hr.OpenChatLegacy',
});

// TODO KBA remove when Studio converted to Owl
widgetRegistry.add('hr_employee_chat', HrEmployeeChatLegacy);
});

```

## File: static\src\components\work_permit_upload\work_permit_upload.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { BinaryField } from "@web/views/fields/binary/binary_field";

export class WorkPermitUploadField extends BinaryField {}
WorkPermitUploadField.template = "hr.WorkPermitUploadField";

registry.category("fields").add("work_permit_upload", WorkPermitUploadField);

```

## File: static\src\components\work_permit_upload\work_permit_upload.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="hr.WorkPermitUploadField" t-inherit="web.BinaryField" t-inherit-mode="primary" owl="1">
        <xpath expr="//label[hasclass('o_select_file_button')]" position="attributes">
            <attribute name="class" remove="btn-primary" add="btn-secondary" separator=" " />
        </xpath>
    </t>

</templates>

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

## File: static\src\js\m2x_avatar_employee.js

```javascript
/** @odoo-module alias=hr.Many2OneAvatarEmployee **/

import fieldRegistry from 'web.field_registry';

import {
    Many2OneAvatarUser,
    KanbanMany2OneAvatarUser,
    KanbanMany2ManyAvatarUser,
    ListMany2ManyAvatarUser,
    Many2ManyAvatarUser,
} from '@mail/js/m2x_avatar_user';


// This module defines variants of the Many2OneAvatarUser and Many2ManyAvatarUser
// field widgets, to support fields pointing to 'hr.employee'. It also defines the
// kanban version of the Many2OneAvatarEmployee widget.
//
// Usage:
//   <field name="employee_id" widget="many2one_avatar_employee"/>

const M2XAvatarEmployeeMixin = {
    supportedModels: ['hr.employee', 'hr.employee.public'],

    //----------------------------------------------------------------------
    // Private
    //----------------------------------------------------------------------

    _getEmployeeID() {
        return this.value.res_id;
    },

    //----------------------------------------------------------------------
    // Handlers
    //----------------------------------------------------------------------

    /**
     * @override
     */
    _onAvatarClicked(ev) {
        ev.stopPropagation(); // in list view, prevent from opening the record
        const employeeId = this._getEmployeeID(ev);
        this._openChat({ employeeId: employeeId });
    }
};

export const Many2OneAvatarEmployee = Many2OneAvatarUser.extend(M2XAvatarEmployeeMixin);
export const KanbanMany2OneAvatarEmployee = KanbanMany2OneAvatarUser.extend(M2XAvatarEmployeeMixin);

fieldRegistry.add('many2one_avatar_employee', Many2OneAvatarEmployee);
fieldRegistry.add('kanban.many2one_avatar_employee', KanbanMany2OneAvatarEmployee);

const M2MAvatarEmployeeMixin = Object.assign(M2XAvatarEmployeeMixin, {
    //----------------------------------------------------------------------
    // Private
    //----------------------------------------------------------------------

    _getEmployeeID(ev) {
        return parseInt(ev.target.getAttribute('data-id'), 10);
    },
});

export const Many2ManyAvatarEmployee = Many2ManyAvatarUser.extend(M2MAvatarEmployeeMixin, {});

export const KanbanMany2ManyAvatarEmployee = KanbanMany2ManyAvatarUser.extend(M2MAvatarEmployeeMixin, {});

export const ListMany2ManyAvatarEmployee = ListMany2ManyAvatarUser.extend(M2MAvatarEmployeeMixin, {});

fieldRegistry.add('many2many_avatar_employee', Many2ManyAvatarEmployee);
fieldRegistry.add('kanban.many2many_avatar_employee', KanbanMany2ManyAvatarEmployee);
fieldRegistry.add('list.many2many_avatar_employee', ListMany2ManyAvatarEmployee);

export default {
    Many2OneAvatarEmployee,
};

```

## File: static\src\js\standalone_m2o_avatar_employee.js

```javascript
/** @odoo-module **/

    import StandaloneFieldManagerMixin from 'web.StandaloneFieldManagerMixin';
    import Widget from 'web.Widget';

    import { Many2OneAvatarEmployee } from '@hr/js/m2x_avatar_employee';

    const StandaloneM2OAvatarEmployee = Widget.extend(StandaloneFieldManagerMixin, {
        className: 'o_standalone_avatar_employee',

        /**
         * @override
         */
        init(parent, value) {
            this._super(...arguments);
            StandaloneFieldManagerMixin.init.call(this);
            this.value = value;
        },
        /**
         * @override
         */
        willStart() {
            return Promise.all([this._super(...arguments), this._makeAvatarWidget()]);
        },
        /**
         * @override
         */
        start() {
            this.avatarWidget.$el.appendTo(this.$el);
            return this._super(...arguments);
        },

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        /**
         * Create a record, and initialize and start the avatar widget.
         *
         * @private
         * @returns {Promise}
         */
        async _makeAvatarWidget() {
            const modelName = 'hr.employee.public';
            const fieldName = 'employee_id';
            const recordId = await this.model.makeRecord(modelName, [{
                name: fieldName,
                relation: modelName,
                type: 'many2one',
                value: this.value,
            }]);
            const state = this.model.get(recordId);
            this.avatarWidget = new Many2OneAvatarEmployee(this, fieldName, state);
            this._registerWidget(recordId, fieldName, this.avatarWidget);
            return this.avatarWidget.appendTo(document.createDocumentFragment());
        },
    });

    export default StandaloneM2OAvatarEmployee;

```

## File: static\src\js\work_permit_upload.js

```javascript
/** @odoo-module **/

import basicFields from 'web.basic_fields';
import fieldRegistry from 'web.field_registry';

const WorkPermitUpload = basicFields.FieldBinaryFile.extend({
    template: "hr.WorkPermitUpload",
});

fieldRegistry.add('work_permit_upload', WorkPermitUpload);

```

## File: static\src\models\employee.js

```javascript
/** @odoo-module **/

import { registerModel } from '@mail/model/model_core';
import { attr, one } from '@mail/model/model_field';
import { clear, insert } from '@mail/model/model_field_command';

registerModel({
    name: 'Employee',
    modelMethods: {
        /**
         * @param {Object} data
         * @returns {Object}
         */
        convertData(data) {
            const data2 = {};
            if ('id' in data) {
                data2.id = data.id;
            }
            if ('user_id' in data) {
                data2.hasCheckedUser = true;
                if (!data.user_id) {
                    data2.user = clear();
                } else {
                    const partnerNameGet = data['user_partner_id'];
                    const partnerData = {
                        display_name: partnerNameGet[1],
                        id: partnerNameGet[0],
                    };
                    const userNameGet = data['user_id'];
                    const userData = {
                        id: userNameGet[0],
                        partner: insert(partnerData),
                        display_name: userNameGet[1],
                    };
                    data2.user = insert(userData);
                }
            }
            return data2;
        },
        /**
         * Performs the `read` RPC on the `hr.employee.public`.
         *
         * @param {Object} param0
         * @param {Object} param0.context
         * @param {string[]} param0.fields
         * @param {integer[]} param0.ids
         */
        async performRpcRead({ context, fields, ids }) {
            const employeesData = await this.messaging.rpc({
                model: 'hr.employee.public',
                method: 'read',
                args: [ids, fields],
                kwargs: {
                    context,
                },
            });
            this.messaging.models['Employee'].insert(employeesData.map(employeeData =>
                this.messaging.models['Employee'].convertData(employeeData)
            ));
        },
        /**
         * Performs the `search_read` RPC on `hr.employee.public`.
         *
         * @param {Object} param0
         * @param {Object} param0.context
         * @param {Array[]} param0.domain
         * @param {string[]} param0.fields
         */
        async performRpcSearchRead({ context, domain, fields }) {
            const employeesData = await this.messaging.rpc({
                model: 'hr.employee.public',
                method: 'search_read',
                kwargs: {
                    context,
                    domain,
                    fields,
                },
            });
            this.messaging.models['Employee'].insert(employeesData.map(employeeData =>
                this.messaging.models['Employee'].convertData(employeeData)
            ));
        },
    },
    recordMethods: {
        /**
         * Checks whether this employee has a related user and partner and links
         * them if applicable.
         */
        async checkIsUser() {
            return this.messaging.models['Employee'].performRpcRead({
                ids: [this.id],
                fields: ['user_id', 'user_partner_id'],
                context: { active_test: false },
            });
        },
        /**
         * Gets the chat between the user of this employee and the current user.
         *
         * If a chat is not appropriate, a notification is displayed instead.
         *
         * @returns {Channel|undefined}
         */
        async getChat() {
            if (!this.user && !this.hasCheckedUser) {
                await this.checkIsUser();
            }
            if (!this.exists()) {
                return;
            }
            // prevent chatting with non-users
            if (!this.user) {
                this.messaging.notify({
                    message: this.env._t("You can only chat with employees that have a dedicated user."),
                    type: 'info',
                });
                return;
            }
            return this.user.getChat();
        },
        /**
         * Opens a chat between the user of this employee and the current user
         * and returns it.
         *
         * If a chat is not appropriate, a notification is displayed instead.
         *
         * @param {Object} [options] forwarded to @see `Thread:open()`
         */
        async openChat(options) {
            const chat = await this.getChat();
            if (!this.exists()) {
                return;
            }
            if (!chat) {
                return;
            }
            await chat.thread.open(options);
            if (!this.exists()) {
                return;
            }
        },
        /**
         * Opens the most appropriate view that is a profile for this employee.
         */
        async openProfile(model = 'hr.employee.public') {
            return this.messaging.openDocument({
                id: this.id,
                model: model,
            });
        },
    },
    fields: {
        /**
         * Whether an attempt was already made to fetch the user corresponding
         * to this employee. This prevents doing the same RPC multiple times.
         */
        hasCheckedUser: attr({
            default: false,
        }),
        /**
         * Unique identifier for this employee.
         */
        id: attr({
            identifying: true,
        }),
        /**
         * Partner related to this employee.
         */
        partner: one('Partner', {
            inverse: 'employee',
            related: 'user.partner',
        }),
        /**
         * User related to this employee.
         */
        user: one('User', {
            inverse: 'employee',
        }),
    },
});

```

## File: static\src\models\messaging.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';

// dummy import to ensure mail Messaging patches are loaded beforehand
import '@mail/models/messaging';

registerPatch({
    name: 'Messaging',
    recordMethods: {
        /**
         * @override
         * @param {integer} [param0.employeeId]
         */
        async getChat({ employeeId }) {
            if (employeeId) {
                const employee = this.messaging.models['Employee'].insert({ id: employeeId });
                return employee.getChat();
            }
            return this._super(...arguments);
        },
        /**
         * @override
         */
        async openProfile({ id, model }) {
            if (model === 'hr.employee' || model === 'hr.employee.public') {
                const employee = this.messaging.models['Employee'].insert({ id });
                return employee.openProfile(model);
            }
            return this._super(...arguments);
        },
    },
});

```

## File: static\src\models\partner.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { attr, one } from '@mail/model/model_field';

registerPatch({
    name: 'Partner',
    recordMethods: {
        /**
         * Checks whether this partner has a related employee and links them if
         * applicable.
         */
        async checkIsEmployee() {
            await this.messaging.models['Employee'].performRpcSearchRead({
                context: { active_test: false },
                domain: [['user_partner_id', '=', this.id]],
                fields: ['user_id', 'user_partner_id'],
            });
            if (!this.exists()) {
                return;
            }
            this.update({ hasCheckedEmployee: true });
        },
        /**
         * When a partner is an employee, its employee profile contains more
         * useful information to know who he is than its partner profile.
         *
         * @override
         */
        async openProfile() {
            // limitation of patch, `this._super` becomes unavailable after `await`
            const _super = this._super.bind(this, ...arguments);
            if (!this.employee && !this.hasCheckedEmployee) {
                await this.checkIsEmployee();
            }
            if (!this.exists()) {
                return;
            }
            if (this.employee) {
                return this.employee.openProfile();
            }
            return _super();
        },
    },
    fields: {
        /**
         * Employee related to this partner. It is computed through
         * the inverse relation and should be considered read-only.
         */
        employee: one('Employee', {
            inverse: 'partner',
        }),
        /**
         * Whether an attempt was already made to fetch the employee
         * corresponding to this partner. This prevents doing the same RPC
         * multiple times.
         */
        hasCheckedEmployee: attr({
            default: false,
        }),
    },
});

```

## File: static\src\models\user.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { one } from '@mail/model/model_field';

registerPatch({
    name: 'User',
    fields: {
        /**
         * Employee related to this user.
         */
        employee: one('Employee', {
            inverse: 'user',
        }),
    },
});

```

## File: static\src\user_menu\my_profile.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { preferencesItem } from "@web/webclient/user_menu/user_menu_items";

export function hrPreferencesItem(env)  {
    return Object.assign(
        {}, 
        preferencesItem(env),
        {
            description: env._t('My Profile'),
        }
    );
}

registry.category("user_menuitems").add('profile', hrPreferencesItem, { force: true })

```

## File: static\src\views\archive_employee_hook.js

```javascript
/** @odoo-module **/

import { useService } from "@web/core/utils/hooks";

const { useComponent, useEnv } = owl;

export function useArchiveEmployee() {
    const component = useComponent();
    const env = useEnv();
    const action = useService("action");
    return (id) => {
        action.doAction({
            type: 'ir.actions.act_window',
            name: env._t('Employee Termination'),
            res_model: 'hr.departure.wizard',
            views: [[false, 'form']],
            view_mode: 'form',
            target: 'new',
            context: {
                'active_id': id,
                'toggle_active': true,
            }
        }, {
            onClose: async () => {
                await component.model.load();
                component.model.notify();
            },
        });
    }
}

```

## File: static\src\views\form_view.js

```javascript
/** @odoo-module */

import { registry } from '@web/core/registry';

import { formView } from '@web/views/form/form_view';
import { FormController } from '@web/views/form/form_controller';
import { FormRenderer } from '@web/views/form/form_renderer';

import { useArchiveEmployee } from '@hr/views/archive_employee_hook';
import { useOpenChat } from "@mail/views/open_chat_hook";

export class EmployeeFormController extends FormController {
    setup() {
        super.setup();
        this.archiveEmployee = useArchiveEmployee();
    }

    getActionMenuItems() {
        const menuItems = super.getActionMenuItems();
        if (!this.archiveEnabled || !this.model.root.isActive) {
            return menuItems;
        }

        const archiveAction = menuItems.other.find((item) => item.key === "archive");
        if (archiveAction) {
            archiveAction.callback = this.archiveEmployee.bind(this, this.model.root.resId);
        }
        return menuItems;
    }
}

// TODO KBA: to remove in master
export class EmployeeFormRenderer extends FormRenderer {
    setup() {
        super.setup();
        this.openChat = useOpenChat(this.props.record.resModel);
    }
}

registry.category('views').add('hr_employee_form', {
    ...formView,
    Controller: EmployeeFormController,
    Renderer: EmployeeFormRenderer,
});

```

## File: static\src\views\kanban_view.js

```javascript
/** @odoo-module */

import { registry } from '@web/core/registry';

import { kanbanView } from '@web/views/kanban/kanban_view';
import { KanbanModel } from '@web/views/kanban/kanban_model';

// TODO KBA: to remove in master
export class EmployeeKanbanRecord extends KanbanModel.Record {
    async openChat(employeeId) {
        const messaging = await this.model.env.services.messaging.get();
        messaging.openChat({ employeeId });
    }
}

export class EmployeeKanbanModel extends KanbanModel {
    setup(params, { messaging }) {
        super.setup(...arguments);
        this.messagingService = messaging;
    }
}
EmployeeKanbanModel.services = [...KanbanModel.services, "messaging"];
EmployeeKanbanModel.Record = EmployeeKanbanRecord;

registry.category('views').add('hr_employee_kanban', {
    ...kanbanView,
    Model: EmployeeKanbanModel,
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

    getActionMenuItems() {
        const menuItems = super.getActionMenuItems();
        const selectedRecords = this.model.root.selection;

        // Only override the Archive action when only 1 record is selected.
        if (!this.archiveEnabled || selectedRecords.length > 1 || !selectedRecords[0].data.active) {
            return menuItems;
        }

        const archiveAction = menuItems.other.find((item) => item.key === "archive");
        if (archiveAction) {
            archiveAction.callback = this.archiveEmployee.bind(this, selectedRecords[0].resId);
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

import { helpers } from "@mail/views/open_chat_hook";
import { patch } from "@web/core/utils/patch";

patch(helpers, "hr_m2x_avatar_employee", {
    SUPPORTED_M2X_AVATAR_MODELS: [...helpers.SUPPORTED_M2X_AVATAR_MODELS, "hr.employee", "hr.employee.public"],
    buildOpenChatParams: function (resModel, id) {
        if (["hr.employee", "hr.employee.public"].includes(resModel)) {
            return { employeeId: id };
        }
        return this._super(...arguments);
    },
});

```

## File: static\src\views\profile_form_view.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";

import { formView } from "@web/views/form/form_view";
import { Record, RelationalModel } from "@web/views/basic_relational_model";

export class EmployeeProfileRecord extends Record {
    async save() {
        const dirtyFields = this.dirtyFields.map((f) => f.name);
        const isSaved = await super.save(...arguments);
        if (isSaved && dirtyFields.includes("lang")) {
            this.model.actionService.doAction("reload_context");
        }
        return isSaved;
    }
}

class EmployeeProfileModel extends RelationalModel {}
EmployeeProfileModel.Record = EmployeeProfileRecord;

registry.category("views").add("hr_employee_profile_form", {
    ...formView,
    Model: EmployeeProfileModel,
});

```

## File: static\src\views\fields\many2many_avatar_employee_field\many2many_avatar_employee_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { Many2ManyTagsAvatarUserField, KanbanMany2ManyTagsAvatarUserField } from "@mail/views/fields/many2many_avatar_user_field/many2many_avatar_user_field";

export class Many2ManyTagsAvatarEmployeeField extends Many2ManyTagsAvatarUserField {
    get relation() {
        return "hr.employee.public";
    }
}

Many2ManyTagsAvatarEmployeeField.extractProps = ({ field, attrs }) => {
    return {
        ...Many2ManyTagsAvatarUserField.extractProps({ field, attrs }),
        canQuickCreate: false,
        relation: (attrs.options && attrs.options.relation) || field.relation,
    }
};

Many2ManyTagsAvatarEmployeeField.additionalClasses = [...Many2ManyTagsAvatarUserField.additionalClasses, "o_field_many2many_avatar_user"];

registry.category("fields").add("many2many_avatar_employee", Many2ManyTagsAvatarEmployeeField);

export class KanbanMany2ManyTagsAvatarEmployeeField extends KanbanMany2ManyTagsAvatarUserField {
    get relation() {
        return "hr.employee.public";
    }
}
KanbanMany2ManyTagsAvatarEmployeeField.additionalClasses = [...KanbanMany2ManyTagsAvatarUserField.additionalClasses, "o_field_many2many_avatar_user"];

registry.category("fields").add("kanban.many2many_avatar_employee", KanbanMany2ManyTagsAvatarEmployeeField);
registry.category("fields").add("list.many2many_avatar_employee", KanbanMany2ManyTagsAvatarEmployeeField);

```

## File: static\src\views\fields\many2one_avatar_employee_field\many2one_avatar_employee_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { Many2OneAvatarUserField, KanbanMany2OneAvatarUserField } from "@mail/views/fields/many2one_avatar_user_field/many2one_avatar_user_field";

export class Many2OneAvatarEmployeeField extends Many2OneAvatarUserField {
    get relation() {
        return "hr.employee.public";
    }
}

Many2OneAvatarEmployeeField.extractProps = ({ field, attrs }) => {
    return {
        ...Many2OneAvatarUserField.extractProps({ field, attrs }),
        relation: (attrs.options && attrs.options.relation) || field.relation,
        canQuickCreate: false,
    }
};

Many2OneAvatarEmployeeField.additionalClasses = [...Many2OneAvatarUserField.additionalClasses, "o_field_many2one_avatar_user"];

registry.category("fields").add("many2one_avatar_employee", Many2OneAvatarEmployeeField);

export class KanbanMany2OneAvatarEmployeeField extends KanbanMany2OneAvatarUserField {
    get relation() {
        return "hr.employee.public";
    }
}
KanbanMany2OneAvatarEmployeeField.extractProps = ({ attrs, field }) => {
    return {
        ...KanbanMany2OneAvatarUserField.extractProps({ attrs, field }),
        relation: (attrs.options && attrs.options.relation) || field.relation,
    };
};

registry.category("fields").add("kanban.many2one_avatar_employee", KanbanMany2OneAvatarEmployeeField);

```

## File: static\src\xml\work_permit_upload.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t  t-name="hr.WorkPermitUpload" t-inherit="web.FieldBinaryFile" t-inherit-mode="primary">
        <xpath expr="//button[@title='Select']" position="attributes">
            <attribute name="class" remove="btn-primary" add="btn-secondary" separator=" "/>
        </xpath>
    </t>
</templates>

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
                </tree>
            </field>
        </record>

        <record id="hr_contract_type_view_form" model="ir.ui.view">
            <field name="model">hr.contract.type</field>
            <field name="arch" type="xml">
                <form>
                    <group>
                        <group>
                            <field name="name" />
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
                        <div class="oe_button_box" name="button_box">
                            <button class="oe_stat_button" type="action" name="%(hr.act_employee_from_department)d" icon="fa-users">
                                <field string="Employees" name="total_employee" widget="statinfo"/>
                            </button>
                            <button class="oe_stat_button" type="object" name="action_plan_from_department" icon="fa-list-ul">
                                <field string="Plans" name="plans_count" widget="statinfo"/>
                            </button>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <field name="active" invisible="1"/>
                        <group col="4">
                            <field name="name"/>
                            <field name="manager_id"/>
                            <field name="parent_id"/>
                            <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
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
                <tree string="Companies" sample="1">
                    <field name="display_name"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="company_id" invisible="1"/>
                    <field name="manager_id"/>
                    <field name="total_employee" string="Employees"/>
                    <field name="parent_id"/>
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
                    <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction','=',True)]"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                </search>
             </field>
        </record>

        <record id="hr_department_view_kanban" model="ir.ui.view" >
            <field name="name">hr.department.kanban</field>
            <field name="model">hr.department</field>
            <field name="arch" type="xml">
                <kanban class="oe_background_grey o_kanban_dashboard o_hr_department_kanban o_kanban_small_column" sample="1">
                    <field name="name"/>
                    <field name="company_id"/>
                    <field name="manager_id"/>
                    <field name="color"/>
                    <field name="total_employee"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''}">
                                <div t-attf-class="o_kanban_card_header">
                                    <div class="o_kanban_card_header_title">
                                        <div class="o_primary"><a type="edit"><field name="name"/></a></div>
                                        <div class="o_secondary" groups="base.group_multi_company">
                                            <small>
                                                <i class="fa fa-building-o" role="img" aria-label="Company" title="Company"/> <field name="company_id"/>
                                            </small>
                                        </div>
                                    </div>
                                    <div class="o_kanban_manage_button_section" t-if="!selection_mode">
                                        <a class="o_kanban_manage_toggle_button" href="#"><i class="fa fa-ellipsis-v" role="img" aria-label="Manage" title="Manage"/></a>
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
                                <div class="o_kanban_card_manage_pane dropdown-menu" role="menu">
                                    <div class="o_kanban_card_manage_section">
                                        <div role="menuitem" class="o_kanban_manage_reports">
                                            <div class="o_kanban_card_manage_title ps-4 pb-1">
                                                <span class="fw-bolder">Reporting</span>
                                            </div>
                                        </div>
                                    </div>
                                    <a t-if="widget.editable" role="menuitem" class="dropdown-item" type="edit">Configuration</a>
                                    <ul t-if="widget.editable" class="oe_kanban_colorpicker" data-field="color" role="menu"/>
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
    <data>

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
         </record>

     </data>
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
                    <field name="job_title" string="Job Title"/>
                    <field name="department_id" string="Department"/>
                    <field name="parent_id" string="Manager"/>
                    <field name="company_id" string="Company"/>
                    <separator/>
                    <filter name="my_team" string="My Team" domain="[('parent_id.user_id', '=', uid)]"/>
                    <filter name="my_department" string="My Department" domain="[('member_of_department', '=', True)]"/>
                    <separator/>
                    <filter name="archived" string="Archived" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Group By">
                        <filter name="group_manager" string="Manager" domain="[]" context="{'group_by':'parent_id'}"/>
                        <filter name="group_department" string="Department" domain="[]" context="{'group_by':'department_id'}"/>
                        <filter name="group_company" string="Company" domain="[]" context="{'group_by':'company_id'}"/>
                    </group>
                    <searchpanel>
                        <field name="company_id" groups="base.group_multi_company" icon="fa-building" enable_counters="1"/>
                        <field name="department_id" icon="fa-users" enable_counters="1"/>
                    </searchpanel>
                </search>
             </field>
        </record>

        <record id="hr_employee_public_view_form" model="ir.ui.view">
            <field name="name">hr.employee.public.form</field>
            <field name="model">hr.employee.public</field>
            <field name="arch" type="xml">
                <form string="Employee" create="0" write="0" js_class="hr_employee_form" class="o_employee_form">
                    <field name="image_128" invisible="1" />
                    <header/>
                    <sheet>
                        <field name="user_id" invisible="1"/>
                        <field name="user_partner_id" invisible="1"/>
                        <field name="active" invisible="1"/>
                        <div class="oe_button_box" name="button_box">
                            <!-- Used by other modules-->
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <field name="avatar_1920" widget='image' class="oe_avatar" options='{"zoom": true, "preview_image":"avatar_128"}'/>
                            <div class="oe_title">
                                <label for="name" string="Employee Name"/>
                                <h1 class="d-flex flex-row">
                                    <span class="me-2">
                                        <widget name="hr_employee_chat" invisible="not context.get('chat_icon')"/>
                                    </span>
                                    <field name="name" placeholder="e.g. John Doe" required="True"
                                        style="font-size: min(4vw, 2.6rem);"/>
                                </h1>
                                <h2>
                                    <field name="job_title" placeholder="Job Title" />
                                </h2>
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
                                    <field name="employee_type"/>
                                    <field name="company_id" groups="base.group_multi_company"/>
                                    <field name="parent_id"/>
                                    <field name="coach_id"/>
                                </group>
                            </group>
                        <notebook>
                            <page name="public" string="Work Information">
                                <div id="o_work_employee_container" class="d-lg-flex"> <!-- These two div are used to position org_chart -->
                                    <div id="o_work_employee_main" class="flex-grow-1">
                                        <group string="Location" name="location">
                                            <field name="address_id"
                                                context="{'show_address': 1}"
                                                options='{"always_reload": True, "highlight_first_line": True}'/>
                                            <field name="work_location_id"/>
                                        </group>
                                        <group name="managers" string="Approvers" invisible="1">
                                            <!-- overridden in other modules -->
                                        </group>
                                        <group string="Schedule" groups="base.group_no_one">
                                            <field name="resource_calendar_id"/>
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
                    <field name="parent_id"/>
                    <field name="coach_id" invisible="1"/>
                </tree>
            </field>
        </record>

        <record id="hr_employee_public_view_kanban" model="ir.ui.view">
            <field name="name">hr.employee.kanban</field>
            <field name="model">hr.employee.public</field>
            <field name="priority">10</field>
            <field name="arch" type="xml">
                <kanban class="o_hr_employee_kanban" js_class="hr_employee_kanban" sample="1">
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
                            <field name="image_1024" class="o_kanban_image_fill_left d-block" preview_image="image_128"
                                widget="background_image" options="{'zoom': true, 'zoom_delay': 1000}"/>

                            <div class="oe_kanban_details">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings">
                                        <strong class="o_kanban_record_title">
                                            <t t-if="record.show_hr_icon_display.raw_value">
                                                <div class="float-end"
                                                     t-if="record.hr_icon_display.raw_value == 'presence_present'">
                                                    <!-- Employee is present/connected and it is normal according to his work schedule  -->
                                                    <span class="fa fa-circle text-success" role="img" aria-label="Present"
                                                          title="Present" name="presence_present">
                                                    </span>
                                                </div>
                                                <div class="float-end"
                                                     t-if="record.hr_icon_display.raw_value == 'presence_absent'">
                                                    <!-- Employee is absent and it is normal according to his work schedule  -->
                                                    <span class="fa fa-circle-o text-muted" role="img" aria-label="Absent"
                                                          title="Absent" name="presence_absent">
                                                    </span>
                                                </div>
                                                <div class="float-end"
                                                     t-if="record.hr_icon_display.raw_value == 'presence_absent_active'">
                                                    <!-- Employee is connected but according to his work schedule, he should not work for now  -->
                                                    <span class="fa fa-circle-o text-success" role="img"
                                                          aria-label="Present but not active"
                                                          title="Present but not active" name="presence_absent_active">
                                                    </span>
                                                </div>
                                                <!-- Employee is not here but according to his work schedule, he should be connected -->
                                                <div class="float-end"
                                                     t-if="record.hr_icon_display.raw_value == 'presence_to_define'">
                                                    <span class="fa fa-circle text-warning" role="img"
                                                          aria-label="To define" title="To define"
                                                          name="presence_to_define">
                                                    </span>
                                                </div>
                                            </t>
                                            <field name="name"/>
                                        </strong>
                                        <span t-if="record.job_title.raw_value" class="o_kanban_record_subtitle"><field name="job_title"/></span>
                                    </div>
                                </div>
                                <ul>
                                    <li id="last_login"/>
                                    <li t-if="record.work_email.raw_value"><field name="work_email" /></li>
                                    <li t-if="record.work_phone.raw_value" class="o_force_ltr"><field name="work_phone" /></li>
                                </ul>
                                <div class="oe_kanban_content position-absolute start-0 bottom-0 end-0 me-2">
                                    <div class="o_kanban_record_bottom">
                                        <div class="oe_kanban_bottom_left"/>
                                        <div class="oe_kanban_bottom_right">
                                            <widget name="hr_employee_chat"/>
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
                    <field name="category_ids" groups="hr.group_hr_user"/>
                    <field name="job_id"/>
                    <field name="parent_id" string="Manager"/>
                    <separator/>
                    <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction', '=', True)]"/>
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
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Group By">
                        <filter name="group_manager" string="Manager" domain="[]" context="{'group_by': 'parent_id'}"/>
                        <filter name="group_department" string="Department" domain="[]" context="{'group_by': 'department_id'}"/>
                        <filter name="group_job" string="Job" domain="[]" context="{'group_by': 'job_id'}"/>
                        <filter name="group_category_ids" string="Tags" domain="[]" context="{'group_by': 'category_ids'}"/>
                    </group>
                    <searchpanel>
                        <field name="company_id" groups="base.group_multi_company" icon="fa-building" enable_counters="1"/>
                        <field name="department_id" icon="fa-users" enable_counters="1"/>
                    </searchpanel>
                </search>
             </field>
        </record>

        <record id="view_employee_form" model="ir.ui.view">
            <field name="name">hr.employee.form</field>
            <field name="model">hr.employee</field>
            <field name="arch" type="xml">
                <form string="Employee" js_class="hr_employee_form" class='o_employee_form'>
                    <field name="active" invisible="1"/>
                    <field name="user_id" invisible="1"/>
                    <field name="user_partner_id" invisible="1"/>
                    <field name="hr_presence_state" invisible="1"/>
                    <field name="hr_icon_display" invisible="1"/>
                    <field name="image_128" invisible="1" />
                    <field name="company_id" invisible="1"/>
                    <field name="last_activity_time" invisible="1"/>
                    <field name="last_activity" invisible="1"/>
                    <header>
                        <button name="%(plan_wizard_action)d" string="Launch Plan" type="action" groups="hr.group_hr_user" attrs="{'invisible': [('active', '=', False)]}"/>
                    </header>
                    <sheet>
                        <div name="button_box" class="oe_button_box">
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <field name="avatar_128" invisible="1"/>
                        <div class="o_employee_avatar">
                            <field name="image_1920" widget='image' class="oe_avatar m-0" options='{"zoom": true, "preview_image":"avatar_128"}'/>
                            <div id="hr_presence_status" class="d-flex align-items-end fs-6 o_employee_availability">
                                <!-- Employee is present/connected and it is normal according to his work schedule  -->
                                <small role="img" class="fa fa-fw fa-circle text-success o_button_icon hr_presence align-middle" attrs="{'invisible': [('hr_icon_display', '!=', 'presence_present')]}" aria-label="Present" title="Present"/>
                                <!-- Employee is not present and it is normal according to his work schedule -->
                                <small role="img" class="fa fa-fw fa-circle-o text-muted o_button_icon hr_presence align-middle" attrs="{'invisible': [('hr_icon_display', '!=', 'presence_absent')]}" aria-label="Absent" title="Absent" name="presence_absent"/>
                                <!-- Employee is connected but according to his work schedule, he should not work for now  -->
                                <small role="img" class="fa fa-fw fa-circle-o text-success o_button_icon hr_presence align-middle" attrs="{'invisible': [('hr_icon_display', '!=', 'presence_absent_active')]}" aria-label="Present but not active" title="Present but not active" name="presence_absent_active"/>
                                <!-- Employee is not here but according to his work schedule, he should be connected -->
                                <small role="img" class="fa fa-fw fa-circle text-warning o_button_icon hr_presence align-middle" attrs="{'invisible': [('hr_icon_display', '!=', 'presence_to_define')]}" aria-label="To define" title="To define" name="presence_to_define"/>
                            </div>
                        </div>
                        <div class="oe_title">
                            <h1 class="d-flex flex-row">
                                <span class="me-2">
                                    <widget name="hr_employee_chat" invisible="not context.get('chat_icon')"/>
                                </span>
                                <field name="name" placeholder="Employee's Name"
                                    required="True" style="font-size: min(4vw, 2.6rem);"/>
                            </h1>
                            <h2>
                                <field name="job_title" placeholder="Job Position" />
                            </h2>
                            <field name="category_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}" placeholder="Tags"  groups="hr.group_hr_user"/>
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
                                <field name="parent_id"/>
                                <field name="coach_id"/>
                            </group>
                        </group>
                        <notebook>
                            <page name="public" string="Work Information">
                                <div id="o_work_employee_container" class="d-lg-flex"> <!-- These two div are used to position org_chart -->
                                    <div id="o_work_employee_main" class="flex-grow-1">
                                        <group string="Location">
                                            <field name="address_id"
                                                context="{'show_address': 1}"
                                                options='{"always_reload": True, "highlight_first_line": True}'/>
                                            <field name="work_location_id" context="{'default_address_id': address_id}" />
                                        </group>
                                        <group name="managers" string="Approvers" class="hide-group-if-empty" invisible="1">
                                            <!-- is overridden in other hr related modules -->
                                        </group>
                                        <group name="departure" string="Departure" attrs="{'invisible': [('active', '=', True)]}">
                                            <field name="departure_reason_id" options="{'no_edit': True, 'no_create': True, 'no_open': True}"/>
                                            <field name="departure_description"/>
                                            <field name="departure_date"/>
                                        </group>
                                        <group string="Schedule">
                                            <field name="resource_calendar_id" required="1"/>
                                            <field name="id" invisible="1"/>
                                            <field name="tz" attrs="{'required': [('id', '!=', False)]}"/>
                                        </group>
                                    </div>
                                </div>
                            </page>
                            <page name="personal_information" string="Private Information" groups="hr.group_hr_user">
                                <group>
                                    <group string="Private Contact">
                                        <field name="address_home_id"
                                            context="{
                                                'show_address': 1,
                                                'default_type': 'private',
                                                'form_view_ref': 'base.res_partner_view_form_private'}"
                                            options='{"always_reload": True, "highlight_first_line": True}'/>
                                        <field name="private_email" string="Email"/>
                                        <field name="phone" string="Phone" readonly="True"/>
                                        <field name="bank_account_id" context="{'default_partner_id': address_home_id}"/>
                                        <field name="lang" string="Language"/>
                                        <label for="km_home_work"/>
                                        <div class="o_row" name="div_km_home_work">
                                            <field name="km_home_work" class="o_hr_narrow_field"/>
                                            <span>Km</span>
                                        </div>
                                    </group>
                                    <group string="Family Status">
                                        <field name="marital"/>
                                        <field name="spouse_complete_name" attrs="{'invisible': [('marital', 'not in', ['married', 'cohabitant'])]}"/>
                                        <field name="spouse_birthdate" attrs="{'invisible': [('marital', 'not in', ['married', 'cohabitant'])]}"/>
                                        <field name="children"/>
                                        <separator name="emergency" string="Emergency"/>
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
                                        <field name="user_id" string="Related User" domain="[('share', '=', False)]" context="{'allow_create_employee': False, 'default_create_employee': False}"/>
                                    </group>
                                    <group string="Attendance/Point of Sale" name="identification_group">
                                        <field name="pin" string="PIN Code"/>
                                        <label for="barcode"/>
                                        <div class="o_row">
                                            <field name="barcode"/>
                                            <button string="Generate" class="btn btn-link" type="object" name="generate_random_barcode" attrs="{'invisible': [('barcode', '!=', False)]}"/>
                                            <button name="%(hr_employee_print_badge)d" string="Print Badge" class="btn btn-link" type="action" attrs="{'invisible': [('barcode', '=', False)]}"/>
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

        <!-- This part of the view_employee_form is defined separately so that the
             smartbutton can have lower priority and therefore be last in the list. -->
        <record id="view_employee_form_smartbutton" model="ir.ui.view">
            <field name="name">view.employee.form.smartbutton</field>
            <field name="model">hr.employee</field>
            <field name="inherit_id" ref="view_employee_form"/>
            <field name="priority" eval="1000"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <field name="related_contacts_count" invisible="1"/>
                    <button name="action_related_contacts"
                        class="oe_stat_button"
                        icon="fa-address-card-o"
                        type="object"
                        attrs="{'invisible': ['|', ('id', '=', False), ('related_contacts_count', '=', 0)]}">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value"><field name="related_contacts_count"/></span>
                            <span class="o_stat_text">Contacts</span>
                        </div>
                    </button>
                </div>
            </field>
        </record>

        <record id="view_employee_tree" model="ir.ui.view">
            <field name="name">hr.employee.tree</field>
            <field name="model">hr.employee</field>
            <field name="arch" type="xml">
                <tree string="Employees" expand="context.get('expand', False)" multi_edit="1" sample="1" js_class="hr_employee_list">
                    <header>
                        <button name="%(plan_wizard_action)d" type="action" string="Launch Plan" context="{'action_plan': True}"/>
                    </header>
                    <field name="name" readonly="1"/>
                    <field name="work_phone" class="o_force_ltr" readonly="1"/>
                    <field name="work_email"/>
                    <field name="activity_ids" widget="list_activity"/>
                    <field name="activity_user_id" optional="hide" string="Activity by" widget="many2one_avatar_user"/>
                    <field name="activity_date_deadline" widget="remaining_days" options="{'allow_order': '1'}"/>
                    <field name="company_id" groups="base.group_multi_company" readonly="1"/>
                    <field name="department_id"/>
                    <field name="job_id"/>
                    <field name="parent_id"/>
                    <field name="address_id" invisible="1"/>
                    <field name="company_id" invisible="1"/>
                    <field name="work_location_id" optional="hide"/>
                    <field name="coach_id" invisible="1"/>
                    <field name="active" invisible="1"/>
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
               <kanban class="o_hr_employee_kanban" js_class="hr_employee_kanban" sample="1">
                   <field name="id"/>
                   <field name="hr_presence_state"/>
                   <field name="user_id"/>
                   <field name="user_partner_id"/>
                   <field name="hr_icon_display"/>
                   <field name="show_hr_icon_display"/>
                   <field name="image_128" />
                   <templates>
                       <t t-name="kanban-box">
                       <div class="oe_kanban_global_click o_kanban_record_has_image_fill o_hr_kanban_record">
                            <field name="image_1024" class="o_kanban_image_fill_left d-block" preview_image="image_128"
                                widget="background_image" options="{'zoom': true, 'zoom_delay': 1000}"/>

                            <div class="oe_kanban_details">
                               <div class="o_kanban_record_top">
                                   <div class="o_kanban_record_headings">
                                       <strong class="o_kanban_record_title">
                                           <t t-if="record.show_hr_icon_display.raw_value">
                                                <div class="float-end"
                                                     t-if="record.hr_icon_display.raw_value == 'presence_present'"
                                                     name="presence_present">
                                                    <!-- Employee is present/connected and it is normal according to his work schedule  -->
                                                    <span class="fa fa-circle text-success" role="img" aria-label="Present"
                                                          title="Present" name="presence_present">
                                                    </span>
                                                </div>
                                                <div class="float-end"
                                                     t-if="record.hr_icon_display.raw_value == 'presence_absent'"
                                                     name="presence_absent">
                                                    <!-- Employee is not present and it is normal according to his work schedule -->
                                                    <span class="fa fa-circle-o text-muted" role="img" aria-label="Absent"
                                                          title="Absent" name="presence_absent">
                                                    </span>
                                                </div>
                                                <div class="float-end"
                                                     t-if="record.hr_icon_display.raw_value == 'presence_absent_active'"
                                                        name="presence_absent_active">
                                                    <!-- Employee is connected but according to his work schedule,
                                                         he should not work for now  -->
                                                    <span class="fa fa-circle-o text-success" role="img"
                                                          aria-label="Present but not active"
                                                          title="Present but not active"
                                                          name="presence_absent_active">
                                                    </span>
                                                </div>
                                                <div class="float-end"
                                                     t-if="record.hr_icon_display.raw_value == 'presence_to_define'"
                                                        name="presence_to_define">
                                                    <!-- Employee is not here but according to his work schedule, he should be connected -->
                                                    <span class="fa fa-circle text-warning" role="img"
                                                          aria-label="To define" title="To define"
                                                          name="presence_to_define">
                                                    </span>
                                                </div>
                                            </t>
                                            <field name="name" placeholder="Employee's Name"/>
                                       </strong>
                                       <span t-if="record.job_title.raw_value" class="o_kanban_record_subtitle"><field name="job_title"/></span>
                                   </div>
                               </div>
                               <field name="category_ids" widget="many2many_tags" options="{'color_field': 'color'}" groups="hr.group_hr_user"/>
                               <ul>
                                   <li id="last_login"/>
                                   <li t-if="record.work_email.raw_value" class="o_text_overflow"><field name="work_email" /></li>
                                   <li t-if="record.work_phone.raw_value" class="o_force_ltr"><field name="work_phone" /></li>
                               </ul>
                           </div>
                           <div class="oe_kanban_content o_hr_employee_kanban_bottom position-absolute bottom-0 start-0 end-0 me-2 pe-none">
                               <div class="o_kanban_record_bottom">
                                   <div class="oe_kanban_bottom_left"/>
                                   <div class="oe_kanban_bottom_right float-end pe-auto">
                                        <widget name="hr_employee_chat"/>
                                        <div class="hr_activity_container mb-1 me-n1">
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
                            <img t-att-src="activity_image('hr.employee', 'avatar_128', record.id.raw_value)" role="img" t-att-title="record.id.value" t-att-alt="record.id.value"/>
                            <div>
                                <field name="name" display="full"/>
                                <field name="job_id" muted="1" display="full"/>
                            </div>
                        </div>
                    </templates>
                </activity>
            </field>
        </record>

        <record id="open_view_employee_list_my" model="ir.actions.act_window">
            <field name="name">Employees</field>
            <field name="res_model">hr.employee</field>
            <field name="view_mode">kanban,tree,form,activity</field>
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

        <!-- Employee tree by manager -->
        <record id="view_partner_tree2" model="ir.ui.view">
            <field name="name">hr.employee.tree</field>
            <field name="model">hr.employee</field>
            <field name="priority" eval="20"/>
            <field name="arch" type="xml">
                <tree string="Employees">
                    <field name="name"/>
                    <field name="work_phone" class="o_force_ltr"/>
                    <field name="work_email"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="department_id"/>
                    <field name="job_id"/>
                    <field name="coach_id" invisible="1"/>
                    <field name="parent_id" invisible="1"/>
                </tree>
            </field>
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
                        <div class="oe_title">
                            <label for="name"/>
                            <h1><field name="name" placeholder="e.g. Sales Manager"/></h1>
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
                    <field name="message_needaction" invisible="1"/>
                    <field name="company_id" groups="base.group_multi_company" optional="hide"/>
                    <field name="company_id" invisible="1"/>
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
                                    <span>Vacancies : <field name="expected_employees"/></span>
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
                    <filter name="message_needaction" string="Unread Messages" domain="[('message_needaction', '=', True)]"/>
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

## File: views\hr_plan_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="hr_plan_view_search" model="ir.ui.view">
            <field name="name">hr.plan.view.search</field>
            <field name="model">hr.plan</field>
            <field name="arch" type="xml">
                <search string="Plan">
                    <field name="name"/>
                    <field name="department_id"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                </search>
            </field>
        </record>

        <record id="hr_plan_view_tree" model="ir.ui.view">
            <field name="name">hr.plan.view.tree</field>
            <field name="model">hr.plan</field>
            <field name="arch" type="xml">
                <tree string="Planning">
                    <field name="name"/>
                    <field name="steps_count"/>
                    <field name="department_id" optional="show"/>
                    <field name="company_id" groups="base.group_multi_company" optional="hide"/>
                    <field name="company_id" invisible="1"/>
                </tree>
            </field>
        </record>

        <record id="hr_plan_view_form" model="ir.ui.view">
            <field name="name">hr.plan.view.form</field>
            <field name="model">hr.plan</field>
            <field name="arch" type="xml">
                <form string="Planning">
                    <field name="company_id" invisible="1"/>
                    <sheet>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <div class="oe_title">
                            <label for="name" string="Plan Name"/>
                            <h1>
                                <field name="name" placeholder="e.g. Onboarding"/>
                            </h1>
                        </div>
                        <group>
                            <field name="department_id"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                            <field name="active" invisible="1"/>
                        </group>
                        <notebook>
                            <page string="Activities To Generate">
                                <field name="plan_activity_type_ids" nolabel="1">
                                    <tree>
                                        <field name="company_id" invisible="1"/>
                                        <field name="activity_type_id"/>
                                        <field name="summary"/>
                                        <field name="responsible"/>
                                        <field name="responsible_id" attrs="{'readonly': [('responsible', '!=', 'other')]}" optional="hide"/>
                                        <field name="company_id" groups="base.group_multi_company" optional="hide"/>
                                    </tree>
                                </field>
                            </page>
                        </notebook>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="hr_plan_activity_type_view_tree" model="ir.ui.view">
            <field name="name">hr.plan.activity.type.view.tree</field>
            <field name="model">hr.plan.activity.type</field>
            <field name="arch" type="xml">
                <tree string="Activities">
                    <field name="activity_type_id"/>
                    <field name="summary"/>
                    <field name="responsible"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id="hr_plan_activity_type_view_form" model="ir.ui.view">
            <field name="name">hr.plan.activity.type.view.form</field>
            <field name="model">hr.plan.activity.type</field>
            <field name="arch" type="xml">
                <form string="Activity">
                    <field name="company_id" invisible="1"/>
                    <sheet>
                        <group>
                            <field name="activity_type_id"/>
                            <field name="summary"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                            <field name="responsible"/>
                            <field name="responsible_id" attrs="{'invisible': [('responsible', '!=', 'other')]}"/>
                            <field name="note"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="hr_plan_action" model="ir.actions.act_window">
            <field name="name">Plans</field>
            <field name="res_model">hr.plan</field>
            <field name="view_mode">tree,form</field>
            <field name="search_view_id" ref="hr_plan_view_search"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Add a new plan
                </p>
            </field>
        </record>

        <record id="hr_plan_activity_type_action" model="ir.actions.act_window">
            <field name="name">Plan Activities</field>
            <field name="res_model">hr.plan.activity.type</field>
            <field name="view_mode">tree,form</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Add a new plan activity
                </p>
            </field>
        </record>

    </data>
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
            web_icon="hr,static/description/icon.svg"
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
                id="menu_hr_departure_reason_tree"
                action="hr_departure_reason_action"
                parent="menu_config_employee"
                sequence="5"
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
                groups="group_hr_user"
                sequence="100"/>

                <menuitem
                    id="menu_config_plan_plan"
                    name="On/Offboarding Plans"
                    action="hr_plan_action"
                    parent="menu_config_plan"
                    groups="group_hr_user"
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
                    <field name="active" invisible="1" />
                    <field name="name" />
                    <field name="address_id" />
                    <field name="company_id" groups="base.group_multi_company" />
                    <field name="company_id" invisible="1"/>
                </tree>
            </field>
        </record>

        <record id="hr_work_location_form_view" model="ir.ui.view">
            <field name="name">hr.work.location.view.form</field>
            <field name="model">hr.work.location</field>
            <field name="arch" type="xml">
                <form string="Work Location">
                    <field name="company_id" invisible="1"/>
                    <sheet>
                        <group>
                            <group>
                                <field name="active" invisible="1" />
                                <field name="name" />
                                <field name="address_id" />
                                <field name="location_number"/>
                            </group>
                            <group>
                                <field name="company_id" groups="base.group_multi_company" />
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

## File: views\mail_channel_views.xml

```xml
<?xml version="1.0" ?>
<odoo><data>
    <record id="mail_channel_view_form_" model="ir.ui.view">
        <field name="name">mail.channel.view.form.inherit.hr</field>
        <field name="model">mail.channel</field>
        <field name="inherit_id" ref="mail.mail_channel_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='group_ids']" position="after">
                <field name="subscription_department_ids" widget="many2many_tags"
                    attrs="{'invisible': [('channel_type', '!=', 'channel')]}"
                    string="Auto Subscribe Departments"/>
            </xpath>
        </field>
    </record>
</data></odoo>
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
            <xpath expr="//div[hasclass('settings')]" position="inside">
                <div class="app_settings_block" data-string="Employees" string="Employees" data-key="hr" groups="hr.group_hr_manager">
                    <h2>Employees</h2>
                    <div class="row mt16 o_settings_container" name="employees_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box" id="presence_control_setting" title="Presence of employees">
                            <div class="o_setting_right_pane">
                                <span class="o_form_label">Presence Control</span>
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
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box"
                            id="presence_reporting_setting"
                            title="Advanced presence of employees">
                            <div class="o_setting_left_pane">
                                <field name="module_hr_presence"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_hr_presence"/>
                                <div class="text-muted" name="hr_presence_options_advanced">
                                    Presence reporting screen, email and IP address control.
                                </div>
                                <div class="d-flex mt-1" attrs="{'invisible': [('module_hr_presence', '=', False)]}">
                                    <field name="hr_presence_control_email" class="ml16"/>
                                    <label for="hr_presence_control_email" class="o_light_label"/>
                                </div>
                                <div class="d-flex ml32" attrs="{'invisible': ['|', ('module_hr_presence', '=', False), ('hr_presence_control_email', '=', False)]}">
                                    <span class="flex-shrink-0 ml8 me-2">Minimum number of emails to send</span>
                                    <field name="hr_presence_control_email_amount" class="ms-2 oe_inline"/>
                                </div>
                                <div class="d-flex" attrs="{'invisible': [('module_hr_presence', '=', False)]}">
                                    <field name="hr_presence_control_ip" class="ml16"/>
                                    <label for="hr_presence_control_ip" class="o_light_label"/>
                                </div>
                                <div class="d-flex ml32" attrs="{'invisible': ['|', ('module_hr_presence', '=', False), ('hr_presence_control_ip', '=', False)]}">
                                    <span class="flex-shrink-0 ml8 me-2">IP Addresses (comma-separated)</span>
                                    <field name="hr_presence_control_ip_list" class="ms-2 oe_inline"/>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="enrich_employee_setting">
                            <div class="o_setting_left_pane">
                                <field name="module_hr_skills"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_hr_skills"/>
                                <div class="text-muted">
                                        Enrich employee profiles with skills and resumes
                                </div>
                            </div>
                        </div>
                    </div>
                    <h2>Work Organization</h2>
                    <div class="row mt16 o_settings_container" name="work_organization_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box" id="default_company_schedule_setting">
                            <div class="o_setting_right_pane">
                                <label for="resource_calendar_id"/>
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." role="img" aria-label="Values set here are company-specific." groups="base.group_multi_company"/>
                                <div class="row">
                                    <div class="text-muted col-lg-8">
                                        Set default company schedule to manage your employees working time
                                    </div>
                                </div>
                                <div class="content-group">
                                    <div class="mt16">
                                        <field name="resource_calendar_id" required="1"
                                            class="o_light_label"
                                            domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]"
                                            context="{'default_company_id': company_id}"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <h2>Employee Update Rights</h2>
                    <div class="row mt16 o_settings_container" name="employee_rights_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box" title="Allow employees to update their own data.">
                            <div class="o_setting_left_pane">
                                <field name="hr_employee_self_edit"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="hr_employee_self_edit"/>
                                <div class="text-muted">
                                    Allow employees to update their own data
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

    <record id="hr_config_settings_action" model="ir.actions.act_window">
        <field name="name">Settings</field>
        <field name="type">ir.actions.act_window</field>
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
                <button name="action_open_employees" type="object" class="oe_stat_button" icon="fa-id-card-o" groups="hr.group_hr_user" attrs="{'invisible': [('employees_count', '=', 0)]}">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value"><field name="employees_count"/></span>
                        <span class="o_stat_text">Employee(s)</span>
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
                    <attribute name="attrs">{'invisible': [('is_system', '=', False)]}</attribute>
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
                            invisible="context.get('from_my_profile', False)"
                            attrs="{'invisible': [('hr_presence_state', '=', 'absent')]}">
                            <div role="img" class="fa fa-fw fa-circle text-success o_button_icon" attrs="{'invisible': [('hr_presence_state', '!=', 'present')]}" aria-label="Available" title="Available"/>
                            <div role="img" class="fa fa-fw fa-circle text-warning o_button_icon" attrs="{'invisible': [('hr_presence_state', '!=', 'to_define')]}" aria-label="Away" title="Away"/>
                            <div role="img" class="fa fa-fw fa-circle text-danger o_button_icon" attrs="{'invisible': [('hr_presence_state', '!=', 'absent')]}" aria-label="Not available" title="Not available"/>

                            <div class="o_stat_info" attrs="{'invisible': [('hr_presence_state', '=', 'present')]}">
                                <span class="o_stat_text">
                                    Not Connected
                                </span>
                            </div>
                            <div class="o_stat_info" attrs="{'invisible': [('hr_presence_state', '!=', 'present')]}">
                                <span class="o_stat_value" attrs="{'invisible': [('last_activity_time', '=', False)]}">
                                    <field name="last_activity_time"/>
                                </span>
                                <span class="o_stat_value" attrs="{'invisible': [('last_activity_time', '!=', False)]}">
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
                            <field name="job_title" class="w-100" placeholder="Job Position" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                        </h2>
                    </div>
                    <group>
                        <group>
                            <field name="can_edit" invisible="1"/>
                            <field name="mobile_phone" widget="phone" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                            <field name="work_phone" widget="phone" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                        </group>
                        <group>
                            <field name="work_email" widget="email" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                            <field name="work_location_id" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                            <field name="company_id" invisible="1"/>
                        </group>
                        <group>
                            <field name="employee_parent_id" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                            <field name="coach_id" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                        </group>
                    </group>
                </notebook>
                <notebook position="inside">
                    <page name="public" string="Work Information">
                        <div id="o_work_employee_container" class="d-lg-flex"> <!-- These two div are used to position org_chart -->
                            <div id="o_work_employee_main" class="flex-grow-1">
                                <group string="Location">
                                    <field name="department_id" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                    <field name="address_id"
                                        context="{'show_address': 1}"
                                        options='{"always_reload": True, "highlight_first_line": True}'
                                        attrs="{'readonly': [('can_edit', '=', False)]}"/>
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
                                <field name="address_home_id" invisible="1"/>
                                <label for="private_street" string="Private Address"/>
                                <div class="o_address_format">
                                    <field name="private_street" placeholder="Street..." class="o_address_street"/>
                                    <field name="private_street2" placeholder="Street 2..." class="o_address_street"/>
                                    <field name="private_city" placeholder="City" class="o_address_city"/>
                                    <field name="private_state_id" class="o_address_state" placeholder="State" options="{'no_open': True, 'no_quick_create': True}" context="{'default_country_id': private_country_id}"/>
                                    <field name="private_zip" placeholder="ZIP" class="o_address_zip"/>
                                    <field name="private_country_id" placeholder="Country" class="o_address_country" options='{"no_open": True, "no_create": True}'/>
                                </div>

                                <field name="private_email" string="Email" attrs="{'readonly': [('can_edit', '=', False)], 'invisible': [('address_home_id', '=', False)]}"/>
                                <field name="employee_phone" string="Phone" class="o_force_ltr" attrs="{'readonly': [('can_edit', '=', False)], 'invisible': [('address_home_id', '=', False)]}"/>
                                <field name="private_lang" string="Language" attrs="{'readonly': [('can_edit', '=', False)], 'invisible': [('address_home_id', '=', False)]}"/>
                                <field name="employee_bank_account_id" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                <field name="km_home_work" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                            </group>
                            <group string="Citizenship">
                                <field name="employee_country_id" options='{"no_open": True, "no_create": True}' attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                <field name="identification_id" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                <field name="passport_id" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                <field name="gender" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                <field name="birthday" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                <field name="place_of_birth" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                <field name="country_of_birth" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                            </group>
                            <group string="Marital Status">
                                <field name="marital" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                <field name="spouse_complete_name" attrs="{'invisible': [('marital', 'not in', ['married', 'cohabitant'])], 'readonly': [('can_edit', '=', False)]}"/>
                                <field name="spouse_birthdate" attrs="{'invisible': [('marital', 'not in', ['married', 'cohabitant'])], 'readonly': [('can_edit', '=', False)]}"/>
                            </group>
                            <group string="Education">
                                <field name="certificate" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                <field name="study_field" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                <field name="study_school" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                            </group>
                            <group string="Dependant">
                                <field name="children" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                            </group>
                            <group string="Emergency">
                                <field name="emergency_contact" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                <field name="emergency_phone" widget="phone" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                            </group>
                            <group string="Work Permit" name="has_work_permit">
                                <field name="visa_no" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                <field name="permit_no" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                <field name="visa_expire" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                            </group>
                        </group>
                    </page>
                     <page name="hr_settings" string="HR Settings">
                        <group>
                            <group string='Status' name="active_group">
                                <field name="employee_type" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                            </group>
                            <group string="Attendance" name="identification_group">
                                <field name="pin" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                <field name="barcode" attrs="{'readonly': [('can_edit', '=', False)]}"/>
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
                <xpath expr="//field[@name='mobile']" position="after">
                    <field name="create_employee_id" force_save="1" invisible="1"/>
                    <field name="create_employee" force_save="1" string="Create Employee" invisible="not context.get('allow_create_employee', True)" attrs="{'invisible': [('create_employee_id', '>', 0)]}"/>
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

        <record id="hr_employee_action_from_user" model="ir.actions.act_window">
            <field name="name">Employees</field>
            <field name="res_model">hr.employee</field>
            <field name="view_mode">kanban,tree,form</field>
            <field name="domain">[('user_id', '=', active_id)]</field>
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
                            attrs="{'invisible': ['|', '|', ('id', '=', False), ('share', '=', True), ('employee_id', '!=', False)]}"/>
                            <!-- share is not correctly recomputed because it depends on fields of reified view => invisible before saving (id=False) -->
                </xpath>
                <xpath expr="//div[@name='button_box']" position="inside">
                    <button name="%(hr_employee_action_from_user)d"
                        class="oe_stat_button"
                        icon="fa-users"
                        attrs="{'invisible': [('employee_count', '=', 0)]}"
                        context="{'active_test': False}"
                        type="action">
                        <field name="employee_count" widget="statinfo" string="Employee(s)"/>
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
    archive_private_address = fields.Boolean('Archive Private Address', default=True)

    def action_register_departure(self):
        employee = self.employee_id
        if self.env.context.get('toggle_active', False) and employee.active:
            employee.with_context(no_wizard=True).toggle_active()
        employee.departure_reason_id = self.departure_reason_id
        employee.departure_description = self.departure_description
        employee.departure_date = self.departure_date

        if self.archive_private_address:
            # ignore contact links to internal users
            private_address = employee.address_home_id
            if private_address and private_address.active and not self.env['res.users'].search([('partner_id', '=', private_address.id)]):
                private_address.sudo().toggle_active()

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
                                <separator colspan="2"/>
                                <div class="o_td_label" id="label_info">
                                    <span class="o_form_label o_hr_form_label cursor-default">HR Info</span>
                                </div>
                                <div class="column" id="info">
                                    <div><field name="archive_private_address"/><label for="archive_private_address"/></div>
                                </div>
                            </group>
                        </group>
                        <group>
                            <div id="detailed_reason" colspan="2">
                                <span class="o_form_label o_hr_form_label cursor-default">Detailed Reason</span>
                                <field name="departure_description"/>
                            </div>
                        </group>
                    </sheet>
                    <footer>
                        <button name="action_register_departure" string="Apply" type="object" class="oe_highlight" data-hotkey="q"/>
                        <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="z"/>
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

## File: wizard\hr_plan_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class HrPlanWizard(models.TransientModel):
    _name = 'hr.plan.wizard'
    _description = 'Plan Wizard'

    def _default_plan_id(self):
        # We know that all employees belong to the same company
        employees = self.env['hr.employee'].browse(self.env.context.get('active_ids') if self.env.context.get('active_ids') else [])
        if not employees:
            return None
        if len(employees.department_id) > 1:
            return self.env['hr.plan'].search([
                ('company_id', '=', employees[0].company_id.id),
                ('department_id', '=', False)
            ], limit=1)
        else:
            return self.env['hr.plan'].search([
                ('company_id', '=', employees[0].company_id.id),
                '|',
                ('department_id', '=', employees[0].department_id.id),
                ('department_id', '=', False)
                ], limit=1)

    plan_id = fields.Many2one('hr.plan', default=lambda self: self._default_plan_id(),
        domain="[('company_id', 'in', [False, company_id]), '|', ('department_id', '=', department_id), ('department_id', '=', False)]")
    department_id = fields.Many2one('hr.department', compute='_compute_department_id')
    employee_ids = fields.Many2many(
        'hr.employee', 'hr_employee_hr_plan_wizard_rel', 'employee_id', 'plan_wizard_id', string='Employee', required=True,
        default=lambda self: self.env.context.get('active_ids', []),
    )
    company_id = fields.Many2one('res.company', 'Company', compute='_compute_company_id', required=True)
    warning = fields.Html(compute='_compute_warning')

    @api.depends('employee_ids')
    def _compute_department_id(self):
        for wizard in self:
            all_departments = wizard.employee_ids.department_id
            wizard.department_id = False if len(all_departments) > 1 else all_departments

    @api.constrains('employee_ids')
    def _check_employee_companies(self):
        for wizard in self:
            if len(wizard.employee_ids.mapped('company_id')) > 1:
                raise ValidationError(_('The employees should belong to the same company.'))

    @api.depends('employee_ids')
    def _compute_company_id(self):
        for wizard in self:
            wizard.company_id = wizard.employee_ids and wizard.employee_ids[0].company_id or self.env.company

    def _get_warnings(self):
        self.ensure_one()
        warnings = set()
        for employee in self.employee_ids:
            for activity_type in self.plan_id.plan_activity_type_ids:
                warning = activity_type.get_responsible_id(employee)['warning']
                if warning:
                    warnings.add(warning)
        return warnings

    @api.depends('employee_ids', 'plan_id')
    def _compute_warning(self):
        for wizard in self:
            warnings = wizard._get_warnings()

            if warnings:
                warning_display = _('The plan %s cannot be launched: <br><ul>', wizard.plan_id.name)
                for warning in warnings:
                    warning_display += '<li>%s</li>' % warning
                warning_display += '</ul>'
            else:
                warning_display = False
            wizard.warning = warning_display

    def _get_activities_to_schedule(self):
        return self.plan_id.plan_activity_type_ids

    def action_launch(self):
        self.ensure_one()
        for employee in self.employee_ids:
            body = _('The plan %s has been started', self.plan_id.name)
            activities = set()
            for activity_type in self._get_activities_to_schedule():
                responsible = activity_type.get_responsible_id(employee)['responsible']
                if self.env['hr.employee'].with_user(responsible).check_access_rights('read', raise_exception=False):
                    date_deadline = self.env['mail.activity']._calculate_date_deadline(activity_type.activity_type_id)
                    employee.activity_schedule(
                        activity_type_id=activity_type.activity_type_id.id,
                        summary=activity_type.summary,
                        note=activity_type.note,
                        user_id=responsible.id,
                        date_deadline=date_deadline
                    )
                    activity = _('%(activity)s, assigned to %(name)s, due on the %(deadline)s', activity=activity_type.summary, name=responsible.name, deadline=date_deadline)
                    activities.add(activity)

            if activities:
                body += '<ul>'
                for activity in activities:
                    body += '<li>%s</li>' % activity
                body += '</ul>'
            employee.message_post(body=body)

        if len(self.employee_ids) == 1:
            return {
                'type': 'ir.actions.act_window',
                'res_model': 'hr.employee',
                'res_id': self.employee_ids.id,
                'name': self.employee_ids.display_name,
                'view_mode': 'form',
                'views': [(False, "form")],
            }

        return {
            'type': 'ir.actions.act_window',
            'res_model': 'hr.employee',
            'name': _('Launch Plans'),
            'view_mode': 'tree,form',
            'target': 'current',
            'domain': [('id', 'in', self.employee_ids.ids)],
        }

```

## File: wizard\hr_plan_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="plan_wizard" model="ir.ui.view">
            <field name="name">plan wizard</field>
            <field name="model">hr.plan.wizard</field>
            <field name="arch" type="xml">
                <form>
                    <sheet>
                        <group>
                            <field name="department_id" attrs="{'invisible': [('department_id', '=', False)]}"/>
                            <field name="plan_id"/>
                            <field name="employee_ids" invisible="1"/>
                            <field name="company_id" invisible="1"/>
                        </group>
                        <div role="alert" class="alert alert-danger mb8" attrs="{'invisible': [('warning', '=', False)]}">
                            <field name="warning"/>
                        </div>
                    </sheet>
                    <footer>
                        <button name="action_launch" string="Launch Plan" type="object" class="oe_highlight" attrs="{'invisible': [('warning', '!=', False)]}" groups="hr.group_hr_user" data-hotkey="q"/>
                        <button name="action_launch" string="Launch Plan" type="object" class="oe_highlight disabled" attrs="{'invisible': [('warning', '=', False)]}" groups="hr.group_hr_user" data-hotkey="q"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="plan_wizard_action" model="ir.actions.act_window">
            <field name="name">Launch Plan</field>
            <field name="res_model">hr.plan.wizard</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
        </record>
    </data>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_plan_wizard
from . import hr_departure_wizard

```

