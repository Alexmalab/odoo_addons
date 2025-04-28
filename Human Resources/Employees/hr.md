# Odoo Module: hr

Category: Human Resources/Employees

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Employees',
    'version': '1.1',
    'category': 'Human Resources/Employees',
    'sequence': 75,
    'summary': 'Centralize employee information',
    'description': "",
    'website': 'https://www.odoo.com/page/employees',
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
        'mail_bot',
    ],
    'data': [
        'security/hr_security.xml',
        'security/ir.model.access.csv',
        'wizard/hr_plan_wizard_views.xml',
        'wizard/hr_departure_wizard_views.xml',
        'views/hr_job_views.xml',
        'views/hr_plan_views.xml',
        'views/hr_employee_category_views.xml',
        'views/hr_employee_public_views.xml',
        'report/hr_employee_badge.xml',
        'views/hr_employee_views.xml',
        'views/hr_department_views.xml',
        'views/hr_views.xml',
        'views/hr_templates.xml',
        'views/res_config_settings_views.xml',
        'views/mail_channel_views.xml',
        'views/res_users.xml',
        'data/hr_data.xml',
    ],
    'demo': [
        'data/hr_demo.xml'
    ],
    'installable': True,
    'application': True,
    'auto_install': False,
    'qweb': ['static/src/xml/hr_templates.xml'],
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

        <record id="employee_admin" model="hr.employee">
            <field name="name" eval="obj(ref('base.partner_admin')).name" model="res.partner"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="address_id" ref="base.partner_admin"/>
            <field name="address_home_id" ref="base.partner_admin"/>
            <field name="image_1920" eval="obj(ref('base.partner_admin')).image_1920" model="res.partner"/>
        </record>

        <record id="onboarding_setup_it_materials" model="hr.plan.activity.type">
            <field name="summary">Setup IT Materials</field>
            <field name="responsible">manager</field>
        </record>

        <record id="onboarding_plan_training" model="hr.plan.activity.type">
            <field name="summary">Plan Training</field>
            <field name="responsible">manager</field>
        </record>

        <record id="onboarding_training" model="hr.plan.activity.type">
            <field name="summary">Training</field>
            <field name="responsible">employee</field>
        </record>

        <record id="offboarding_setup_compute_out_delais" model="hr.plan.activity.type">
            <field name="summary">Compute Out Delais</field>
            <field name="responsible">manager</field>
        </record>

        <record id="offboarding_take_back_hr_materials" model="hr.plan.activity.type">
            <field name="summary">Take Back HR Materials</field>
            <field name="responsible">manager</field>
        </record>

        <record id="onboarding_plan" model='hr.plan'>
            <field name="name">Onboarding</field>
            <field name="plan_activity_type_ids" eval="[(6, 0, [
                ref('hr.onboarding_setup_it_materials'),
                ref('hr.onboarding_plan_training'),
                ref('hr.onboarding_training'),
                ])]"/>
        </record>

        <record id="offboarding_plan" model='hr.plan'>
            <field name="name">Offboarding</field>
            <field name="plan_activity_type_ids" eval="[(6, 0, [
                ref('hr.offboarding_setup_compute_out_delais'),
                ref('hr.offboarding_take_back_hr_materials'),
                ])]"/>
        </record>

        <record id="mail_template_data_unknown_employee_email_address" model="mail.template">
            <field name="name">HR: mailgateway unknown employee bounce</field>
            <field name="model_id" ref="base.model_ir_module_module"/>
            <field name="subject">Your document has not been created</field>
            <field name="email_from">${user.email_formatted | safe}</field>
            <field name="email_to">${ctx['email_to']|safe}</field>
            <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 16px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="top" style="font-size: 13px;">
                    <div>
                        Hi,<br/>
                        Your document has not been created because your email address is not recognized. Please send emails with the email address recorded on your employee information, or contact your HR manager.
                    </div>
                </td></tr>
            </table>
    </td></tr>
</tbody>
</table>
</td></tr>
<!-- POWERED BY -->
<tr><td align="center" style="min-width: 590px;">
    <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
      <tr><td style="text-align: center; font-size: 13px;">
        Powered by <a target="_blank" href="https://www.odoo.com?utm_source=db&amp;utm_medium=hr" style="color: #875A7B;">Odoo</a>
      </td></tr>
    </table>
</td></tr>
</table>
            </field>
            <field name="lang">${user.lang}</field>
            <field name="auto_delete" eval="True"/>
            <field name="user_signature" eval="False"/>
        </record>

        <record model="ir.config_parameter" id="hr_presence_control_login" forcecreate="False">
            <field name="key">hr.hr_presence_control_login</field>
            <field name="value">True</field>
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

      <record id="dep_ps" model="hr.department">
          <field name="name">Professional Services</field>
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
      </record>

      <record id="job_consultant" model="hr.job">
          <field name="name">Consultant</field>
          <field name="department_id" ref="dep_ps"/>
          <field name="no_of_recruitment">5</field>
      </record>

      <record id="job_developer" model="hr.job">
          <field name="name">Experienced Developer</field>
          <field name="department_id" ref="dep_rd"/>
          <field name="no_of_recruitment">5</field>
      </record>

      <record id="job_hrm" model="hr.job">
          <field name="name">Human Resources Manager</field>
          <field name="department_id" ref="dep_administration"/>
          <field name="description">Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.</field>
          <field name="requirements">Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.</field>
      </record>

      <record id="job_marketing" model="hr.job">
          <field name="name">Marketing and Community Manager</field>
          <field name="department_id" ref="dep_sales"/>
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
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(237)-125-2389</field>
          <field name="work_email">aiden.hughes71@example.com</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4'), ref('employee_category_3')])]"/>
          <field name="job_id" ref="hr.job_ceo"/>
          <field name="job_title">Chief Executive Officer</field>
          <field name="department_id" ref="dep_management"/>
      </record>

      <record id="employee_al" model="hr.employee">
          <field name="name">Ronnie Hart</field>
          <field name="department_id" ref="dep_rd"/>
          <field name="job_id" ref="hr.job_cto"/>
          <field name="job_title">Chief Technical Officer</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4'), ref('employee_category_3')])]"/>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(376)-310-7863</field>
          <field name="work_email">ronnie.hart87@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_al-image.jpg"/>
      </record>

      <record id="employee_mit" model="hr.employee">
          <field name="name">Anita Oliver</field>
          <field name="department_id" ref="dep_rd"/>
          <field name="parent_id" ref="employee_al"/>
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4')])]"/>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(538)-497-4804</field>
          <field name="mobile_phone">(538)-672-3185</field>
          <field name="work_email">anita.oliver32@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_mit-image.jpg"/>
      </record>

      <record id="employee_niv" model="hr.employee">
          <field name="name">Sharlene Rhodes</field>
          <field name="department_id" ref="dep_management"/>
          <field name="parent_id" ref="employee_al"/>
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4')])]"/>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(450)-719-4182</field>
          <field name="work_email">sharlene.rhodes49@example.comcom</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_niv-image.jpg"/>
      </record>

      <record id="employee_stw" model="hr.employee">
          <field name="name">Randall Lewis</field>
          <field name="department_id" ref="dep_rd"/>
          <field name="parent_id" ref="employee_al"/>
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4')])]"/>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(332)-775-6660</field>
          <field name="work_email">randall.lewis74@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_stw-image.jpg"/>
      </record>

      <record id="employee_chs" model="hr.employee">
          <field name="name">Jennie Fletcher</field>
          <field name="department_id" ref="dep_rd"/>
          <field name="parent_id" ref="employee_al"/>
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4')])]"/>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(157)-363-8229</field>
          <field name="work_email">jennie.fletcher76@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_chs-image.jpg"/>
      </record>

      <record id="employee_qdp" model="hr.employee">
          <field name="name">Marc Demo</field>
          <field name="user_id" ref="base.user_demo"/>
          <field name="department_id" ref="dep_rd"/>
          <field name="parent_id" ref="employee_admin"/>
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4')])]"/>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">+3281813700</field>
          <field name="work_email">gilles@odoo.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_qdp-image.png"/>
      </record>

      <record id="employee_fme" model="hr.employee">
          <field name="name">Keith Byrd</field>
          <field name="department_id" ref="dep_rd"/>
          <field name="parent_id" ref="employee_al"/>
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4')])]"/>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(449)-505-5146</field>
          <field name="work_email">keith.byrd52@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_fme-image.jpg"/>
      </record>

      <record id="employee_fpi" model="hr.employee">
          <field name="name">Audrey Peterson</field>
          <field name="department_id" ref="dep_ps"/>
          <field name="parent_id" ref="employee_admin"/>
          <field name="job_id" ref="hr.job_consultant"/>
          <field name="job_title">Consultant</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4'), ref('employee_category_5')])]"/>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(203)-276-7903</field>
          <field name="work_email">audrey.peterson25@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_fpi-image.jpg"/>
      </record>

      <record id="employee_jth" model="hr.employee">
          <field name="name">Toni Jimenez</field>
          <field name="department_id" ref="dep_ps"/>
          <field name="parent_id" ref="employee_admin"/>
          <field name="job_id" ref="hr.job_consultant"/>
          <field name="job_title">Consultant</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4'), ref('employee_category_5')])]"/>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(663)-707-8451</field>
          <field name="work_email">toni.jimenez23@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_jth-image.jpg"/>
      </record>

      <record id="employee_ngh" model="hr.employee">
          <field name="name">Jeffrey Kelly</field>
          <field name="department_id" ref="dep_sales"/>
          <field name="parent_id" ref="employee_admin"/>
          <field name="job_id" ref="hr.job_marketing"/>
          <field name="job_title">Marketing and Community Manager</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4'), ref('employee_category_2')])]"/>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(916)-264-7362</field>
          <field name="work_email">jeffrey.kelly72@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_ngh-image.jpg"/>
      </record>

      <record id="employee_vad" model="hr.employee">
          <field name="name">Tina Williamson</field>
          <field name="department_id" ref="dep_administration"/>
          <field name="parent_id" ref="employee_admin"/>
          <field name="job_id" ref="hr.job_hrm"/>
          <field name="job_title">Human Resources Manager</field>
          <field name="category_ids" eval="[(6, 0, [ref('employee_category_4')])]"/>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(360)-694-7266</field>
          <field name="work_email">tina.williamson98@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_vad-image.jpg"/>
      </record>

      <record id="employee_han" model="hr.employee">
          <field name="name">Walter Horton</field>
          <field name="department_id" ref="dep_rd"/>
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(350)-912-1201</field>
          <field name="work_email">walter.horton80@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_han-image.jpg"/>
      </record>

      <record id="employee_jve" model="hr.employee">
          <field name="name">Paul Williams</field>
          <field name="department_id" ref="dep_management"/>
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(114)-262-1607</field>
          <field name="work_email">paul.williams59@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_jve-image.jpg"/>
      </record>

      <record id="employee_jep" model="hr.employee">
          <field name="name">Doris Cole</field>
          <field name="department_id" ref="dep_ps"/>
          <field name="job_id" ref="hr.job_consultant"/>
          <field name="job_title">Consultant</field>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(883)-331-5378</field>
          <field name="work_email">doris.cole31@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_jep-image.jpg"/>
      </record>

      <record id="employee_jod" model="hr.employee">
          <field name="name">Rachel Perry</field>
          <field name="department_id" ref="dep_sales"/>
          <field name="job_id" ref="hr.job_marketing"/>
          <field name="job_title">Marketing and Community Manager</field>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(206)-267-3735</field>
          <field name="work_email">jod@odoo.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_jod-image.jpg"/>
      </record>

      <record id="employee_jog" model="hr.employee">
          <field name="name">Beth Evans</field>
          <field name="department_id" ref="dep_rd"/>
          <field name="job_id" ref="hr.job_developer"/>
          <field name="job_title">Experienced Developer</field>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(754)-532-3841</field>
          <field name="work_email">beth.evans77@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_jog-image.jpg"/>
      </record>

      <record id="employee_jgo" model="hr.employee">
          <field name="name">Ernest Reed</field>
          <field name="department_id" ref="dep_ps"/>
          <field name="job_id" ref="hr.job_consultant"/>
          <field name="job_title">Consultant</field>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(944)-518-8232</field>
          <field name="work_email">ernest.reed47@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_jgo-image.jpg"/>
      </record>

      <record id="employee_lur" model="hr.employee">
          <field name="name">Eli Lambert</field>
          <field name="department_id" ref="dep_sales"/>
          <field name="job_id" ref="hr.job_marketing"/>
          <field name="job_title">Marketing and Community Manager</field>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_phone">(644)-169-1352</field>
          <field name="work_email">eli.lambert22@example.com</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_lur-image.jpg"/>
      </record>

      <record id="employee_hne" model="hr.employee">
          <field name="name">Abigail Peterson</field>
          <field name="department_id" ref="dep_ps"/>
          <field name="job_id" ref="hr.job_consultant"/>
          <field name="job_title">Consultant</field>
          <field name="work_location">Building 1, Second Floor</field>
          <field name="work_email">abigail.peterson39@example.com</field>
          <field name="work_phone">(482)-233-3393</field>
          <field name="image_1920" type="base64" file="hr/static/img/employee_hne-image.jpg"/>
      </record>
    </data>
</odoo>

```

## File: models\hr_department.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class Department(models.Model):
    _name = "hr.department"
    _description = "HR Department"
    _inherit = ['mail.thread']
    _order = "name"
    _rec_name = 'complete_name'

    name = fields.Char('Department Name', required=True)
    complete_name = fields.Char('Complete Name', compute='_compute_complete_name', store=True)
    active = fields.Boolean('Active', default=True)
    company_id = fields.Many2one('res.company', string='Company', index=True, default=lambda self: self.env.company)
    parent_id = fields.Many2one('hr.department', string='Parent Department', index=True, domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    child_ids = fields.One2many('hr.department', 'parent_id', string='Child Departments')
    manager_id = fields.Many2one('hr.employee', string='Manager', tracking=True, domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    member_ids = fields.One2many('hr.employee', 'department_id', string='Members', readonly=True)
    jobs_ids = fields.One2many('hr.job', 'department_id', string='Jobs')
    note = fields.Text('Note')
    color = fields.Integer('Color Index')

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

    @api.constrains('parent_id')
    def _check_parent_id(self):
        if not self._check_recursion():
            raise ValidationError(_('You cannot create recursive departments.'))

    @api.model
    def create(self, vals):
        # TDE note: auto-subscription of manager done by hand, because currently
        # the tracking allows to track+subscribe fields linked to a res.user record
        # An update of the limited behavior should come, but not currently done.
        department = super(Department, self.with_context(mail_create_nosubscribe=True)).create(vals)
        manager = self.env['hr.employee'].browse(vals.get("manager_id"))
        if manager.user_id:
            department.message_subscribe(partner_ids=manager.user_id.partner_id.ids)
        return department

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

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
from random import choice
from string import digits
import itertools
from werkzeug import url_encode
import pytz

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError, AccessError
from odoo.modules.module import get_module_resource
from odoo.addons.resource.models.resource_mixin import timezone_datetime


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
    _inherit = ['hr.employee.base', 'mail.thread', 'mail.activity.mixin', 'resource.mixin', 'image.mixin']
    _mail_post_access = 'read'

    @api.model
    def _default_image(self):
        image_path = get_module_resource('hr', 'static/src/img', 'default_image.png')
        return base64.b64encode(open(image_path, 'rb').read())

    # resource and user
    # required on the resource, make sure required="True" set in the view
    name = fields.Char(string="Employee Name", related='resource_id.name', store=True, readonly=False, tracking=True)
    user_id = fields.Many2one('res.users', 'User', related='resource_id.user_id', store=True, readonly=False)
    user_partner_id = fields.Many2one(related='user_id.partner_id', related_sudo=False, string="User's partner")
    active = fields.Boolean('Active', related='resource_id.active', default=True, store=True, readonly=False)
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
    country_id = fields.Many2one(
        'res.country', 'Nationality (Country)', groups="hr.group_hr_user", tracking=True)
    gender = fields.Selection([
        ('male', 'Male'),
        ('female', 'Female'),
        ('other', 'Other')
    ], groups="hr.group_hr_user", default="male", tracking=True)
    marital = fields.Selection([
        ('single', 'Single'),
        ('married', 'Married'),
        ('cohabitant', 'Legal Cohabitant'),
        ('widower', 'Widower'),
        ('divorced', 'Divorced')
    ], string='Marital Status', groups="hr.group_hr_user", default='single', tracking=True)
    spouse_complete_name = fields.Char(string="Spouse Complete Name", groups="hr.group_hr_user", tracking=True)
    spouse_birthdate = fields.Date(string="Spouse Birthdate", groups="hr.group_hr_user", tracking=True)
    children = fields.Integer(string='Number of Children', groups="hr.group_hr_user", tracking=True)
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
        help='Employee bank salary account')
    permit_no = fields.Char('Work Permit No', groups="hr.group_hr_user", tracking=True)
    visa_no = fields.Char('Visa No', groups="hr.group_hr_user", tracking=True)
    visa_expire = fields.Date('Visa Expire Date', groups="hr.group_hr_user", tracking=True)
    additional_note = fields.Text(string='Additional Note', groups="hr.group_hr_user", tracking=True)
    certificate = fields.Selection([
        ('bachelor', 'Bachelor'),
        ('master', 'Master'),
        ('other', 'Other'),
    ], 'Certificate Level', default='other', groups="hr.group_hr_user", tracking=True)
    study_field = fields.Char("Field of Study", groups="hr.group_hr_user", tracking=True)
    study_school = fields.Char("School", groups="hr.group_hr_user", tracking=True)
    emergency_contact = fields.Char("Emergency Contact", groups="hr.group_hr_user", tracking=True)
    emergency_phone = fields.Char("Emergency Phone", groups="hr.group_hr_user", tracking=True)
    km_home_work = fields.Integer(string="Km Home-Work", groups="hr.group_hr_user", tracking=True)

    image_1920 = fields.Image(default=_default_image)
    phone = fields.Char(related='address_home_id.phone', related_sudo=False, readonly=False, string="Private Phone", groups="hr.group_hr_user")
    # employee in company
    child_ids = fields.One2many('hr.employee', 'parent_id', string='Direct subordinates')
    category_ids = fields.Many2many(
        'hr.employee.category', 'employee_category_rel',
        'emp_id', 'category_id', groups="hr.group_hr_manager",
        string='Tags')
    # misc
    notes = fields.Text('Notes', groups="hr.group_hr_user")
    color = fields.Integer('Color Index', default=0)
    barcode = fields.Char(string="Badge ID", help="ID used for employee identification.", groups="hr.group_hr_user", copy=False)
    pin = fields.Char(string="PIN", groups="hr.group_hr_user", copy=False,
        help="PIN used to Check In/Out in Kiosk Mode (if enabled in Configuration).")
    departure_reason = fields.Selection([
        ('fired', 'Fired'),
        ('resigned', 'Resigned'),
        ('retired', 'Retired')
    ], string="Departure Reason", groups="hr.group_hr_user", copy=False, tracking=True)
    departure_description = fields.Text(string="Additional Information", groups="hr.group_hr_user", copy=False, tracking=True)
    message_main_attachment_id = fields.Many2one(groups="hr.group_hr_user")

    _sql_constraints = [
        ('barcode_uniq', 'unique (barcode)', "The Badge ID must be unique, this one is already assigned to another employee."),
        ('user_uniq', 'unique (user_id, company_id)', "A user cannot be linked to multiple employees in the same company.")
    ]

    def name_get(self):
        if self.check_access_rights('read', raise_exception=False):
            return super(HrEmployeePrivate, self).name_get()
        return self.env['hr.employee.public'].browse(self.ids).name_get()

    def _read(self, fields):
        if self.check_access_rights('read', raise_exception=False):
            return super(HrEmployeePrivate, self)._read(fields)

        res = self.env['hr.employee.public'].browse(self.ids).read(fields)
        for r in res:
            record = self.browse(r['id'])
            record._update_cache({k:v for k,v in r.items() if k in fields}, validate=False)

    def read(self, fields, load='_classic_read'):
        if self.check_access_rights('read', raise_exception=False):
            return super(HrEmployeePrivate, self).read(fields, load=load)
        private_fields = set(fields).difference(self.env['hr.employee.public']._fields.keys())
        if private_fields:
            raise AccessError(_('The fields "%s" you try to read is not available on the public employee profile.') % (','.join(private_fields)))
        return self.env['hr.employee.public'].browse(self.ids).read(fields, load=load)

    @api.model
    def load_views(self, views, options=None):
        if self.check_access_rights('read', raise_exception=False):
            return super(HrEmployeePrivate, self).load_views(views, options=options)
        return self.env['hr.employee.public'].load_views(views, options=options)

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
        return self.env['hr.employee.public']._search(args, offset=offset, limit=limit, order=order, count=count, access_rights_uid=access_rights_uid)

    def get_formview_id(self, access_uid=None):
        """ Override this method in order to redirect many2one towards the right model depending on access_uid """
        if access_uid:
            self_sudo = self.with_user(access_uid)
        else:
            self_sudo = self

        if self_sudo.check_access_rights('read', raise_exception=False):
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

        if not self_sudo.check_access_rights('read', raise_exception=False):
            res['res_model'] = 'hr.employee.public'

        return res

    @api.constrains('pin')
    def _verify_pin(self):
        for employee in self:
            if employee.pin and not employee.pin.isdigit():
                raise ValidationError(_("The PIN must be a sequence of digits."))

    @api.onchange('job_id')
    def _onchange_job_id(self):
        if self.job_id:
            self.job_title = self.job_id.name

    @api.onchange('address_id')
    def _onchange_address(self):
        self.work_phone = self.address_id.phone
        self.mobile_phone = self.address_id.mobile

    @api.onchange('company_id')
    def _onchange_company(self):
        address = self.company_id.partner_id.address_get(['default'])
        self.address_id = address['default'] if address else False

    @api.onchange('department_id')
    def _onchange_department(self):
        if self.department_id.manager_id:
            self.parent_id = self.department_id.manager_id

    @api.onchange('user_id')
    def _onchange_user(self):
        if self.user_id:
            self.update(self._sync_user(self.user_id))
            if not self.name:
                self.name = self.user_id.name

    @api.onchange('resource_calendar_id')
    def _onchange_timezone(self):
        if self.resource_calendar_id and not self.tz:
            self.tz = self.resource_calendar_id.tz

    def _sync_user(self, user):
        vals = dict(
            image_1920=user.image_1920,
            work_email=user.email,
            user_id=user.id,
        )
        if user.tz:
            vals['tz'] = user.tz
        return vals

    @api.model
    def create(self, vals):
        if vals.get('user_id'):
            user = self.env['res.users'].browse(vals['user_id'])
            vals.update(self._sync_user(user))
            vals['name'] = vals.get('name', user.name)
        employee = super(HrEmployeePrivate, self).create(vals)
        url = '/web#%s' % url_encode({
            'action': 'hr.plan_wizard_action',
            'active_id': employee.id,
            'active_model': 'hr.employee',
            'menu_id': self.env.ref('hr.menu_hr_root').id,
        })
        employee._message_log(body=_('<b>Congratulations!</b> May I recommend you to setup an <a href="%s">onboarding plan?</a>') % (url))
        if employee.department_id:
            self.env['mail.channel'].sudo().search([
                ('subscription_department_ids', 'in', employee.department_id.id)
            ])._subscribe_users()
        return employee

    def write(self, vals):
        if 'address_home_id' in vals:
            account_id = vals.get('bank_account_id') or self.bank_account_id.id
            if account_id:
                self.env['res.partner.bank'].browse(account_id).partner_id = vals['address_home_id']
        if vals.get('user_id'):
            vals.update(self._sync_user(self.env['res.users'].browse(vals['user_id'])))
        res = super(HrEmployeePrivate, self).write(vals)
        if vals.get('department_id') or vals.get('user_id'):
            department_id = vals['department_id'] if vals.get('department_id') else self[:1].department_id.id
            # When added to a department or changing user, subscribe to the channels auto-subscribed by department
            self.env['mail.channel'].sudo().search([
                ('subscription_department_ids', 'in', department_id)
            ])._subscribe_users()
        return res

    def unlink(self):
        resources = self.mapped('resource_id')
        super(HrEmployeePrivate, self).unlink()
        return resources.unlink()

    def toggle_active(self):
        res = super(HrEmployeePrivate, self).toggle_active()
        self.filtered(lambda employee: employee.active).write({
            'departure_reason': False,
            'departure_description': False,
        })
        if len(self) == 1 and not self.active:
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

    def generate_random_barcode(self):
        for employee in self:
            employee.barcode = '041'+"".join(choice(digits) for i in range(9))

    @api.depends('address_home_id.parent_id')
    def _compute_is_address_home_a_company(self):
        """Checks that chosen address (res.partner) is not linked to a company.
        """
        for employee in self:
            try:
                employee.is_address_home_a_company = employee.address_home_id.parent_id.id is not False
            except AccessError:
                employee.is_address_home_a_company = False

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

from odoo import api, fields, models
from pytz import timezone, UTC
from odoo.tools import format_time


class HrEmployeeBase(models.AbstractModel):
    _name = "hr.employee.base"
    _description = "Basic Employee"
    _order = 'name'

    name = fields.Char()
    active = fields.Boolean("Active")
    color = fields.Integer('Color Index', default=0)
    department_id = fields.Many2one('hr.department', 'Department', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    job_id = fields.Many2one('hr.job', 'Job Position', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    job_title = fields.Char("Job Title")
    company_id = fields.Many2one('res.company', 'Company')
    address_id = fields.Many2one('res.partner', 'Work Address', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    work_phone = fields.Char('Work Phone')
    mobile_phone = fields.Char('Work Mobile')
    work_email = fields.Char('Work Email')
    work_location = fields.Char('Work Location')
    user_id = fields.Many2one('res.users')
    resource_id = fields.Many2one('resource.resource')
    resource_calendar_id = fields.Many2one('resource.calendar', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    parent_id = fields.Many2one('hr.employee', 'Manager', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    coach_id = fields.Many2one('hr.employee', 'Coach', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    tz = fields.Selection(
        string='Timezone', related='resource_id.tz', readonly=False,
        help="This field is used in order to define in which timezone the resources will work.")
    hr_presence_state = fields.Selection([
        ('present', 'Present'),
        ('absent', 'Absent'),
        ('to_define', 'To Define')], compute='_compute_presence_state', default='to_define')
    last_activity = fields.Date(compute="_compute_last_activity")
    last_activity_time = fields.Char(compute="_compute_last_activity")

    @api.depends('user_id.im_status')
    def _compute_presence_state(self):
        """
        This method is overritten in several other modules which add additional
        presence criterions. e.g. hr_attendance, hr_holidays
        """
        # Check on login
        check_login = literal_eval(self.env['ir.config_parameter'].sudo().get_param('hr.hr_presence_control_login', 'False'))
        for employee in self:
            state = 'to_define'
            if check_login:
                if employee.user_id.im_status == 'online':
                    state = 'present'
                elif employee.user_id.im_status == 'offline':
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
                    employee.last_activity_time = format_time(self.env, last_activity_datetime, time_format='short')
                else:
                    employee.last_activity_time = False
            else:
                employee.last_activity = False
                employee.last_activity_time = False

    @api.onchange('parent_id')
    def _onchange_parent_id(self):
        manager = self.parent_id
        previous_manager = self._origin.parent_id
        if manager and (self.coach_id == previous_manager or not self.coach_id):
            self.coach_id = manager

```

## File: models\hr_employee_category.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class EmployeeCategory(models.Model):

    _name = "hr.employee.category"
    _description = "Employee Category"

    name = fields.Char(string="Tag Name", required=True)
    color = fields.Integer(string='Color Index')
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
    work_location = fields.Char(readonly=True)
    user_id = fields.Many2one(readonly=True)
    resource_id = fields.Many2one(readonly=True)
    resource_calendar_id = fields.Many2one(readonly=True)
    tz = fields.Selection(readonly=True)
    color = fields.Integer(readonly=True)

    # hr.employee.public specific fields
    child_ids = fields.One2many('hr.employee.public', 'parent_id', string='Direct subordinates', readonly=True)
    image_1920 = fields.Image("Original Image", compute='_compute_image', compute_sudo=True)
    image_1024 = fields.Image("Image 1024", compute='_compute_image', compute_sudo=True)
    image_512 = fields.Image("Image 512", compute='_compute_image', compute_sudo=True)
    image_256 = fields.Image("Image 256", compute='_compute_image', compute_sudo=True)
    image_128 = fields.Image("Image 128", compute='_compute_image', compute_sudo=True)
    parent_id = fields.Many2one('hr.employee.public', 'Manager', readonly=True)
    coach_id = fields.Many2one('hr.employee.public', 'Coach', readonly=True)

    def _compute_image(self):
        for employee in self:
            # We have to be in sudo to have access to the images
            employee_id = self.sudo().env['hr.employee'].browse(employee.id)
            employee.image_1920 = employee_id.image_1920
            employee.image_1024 = employee_id.image_1024
            employee.image_512 = employee_id.image_512
            employee.image_256 = employee_id.image_256
            employee.image_128 = employee_id.image_128

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


class Job(models.Model):

    _name = "hr.job"
    _description = "Job Position"
    _inherit = ['mail.thread']

    name = fields.Char(string='Job Position', required=True, index=True, translate=True)
    expected_employees = fields.Integer(compute='_compute_employees', string='Total Forecasted Employees', store=True,
        help='Expected number of employees for this job position after new recruitment.')
    no_of_employee = fields.Integer(compute='_compute_employees', string="Current Number of Employees", store=True,
        help='Number of employees currently occupying this job position.')
    no_of_recruitment = fields.Integer(string='Expected New Employees', copy=False,
        help='Number of new employees you expect to recruit.', default=1)
    no_of_hired_employee = fields.Integer(string='Hired Employees', copy=False,
        help='Number of hired employees for this job position during recruitment phase.')
    employee_ids = fields.One2many('hr.employee', 'job_id', string='Employees', groups='base.group_user')
    description = fields.Text(string='Job Description')
    requirements = fields.Text('Requirements')
    department_id = fields.Many2one('hr.department', string='Department', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    company_id = fields.Many2one('res.company', string='Company', default=lambda self: self.env.company)
    state = fields.Selection([
        ('recruit', 'Recruitment in Progress'),
        ('open', 'Not Recruiting')
    ], string='Status', readonly=True, required=True, tracking=True, copy=False, default='recruit', help="Set whether the recruitment process is open or closed for this job position.")

    _sql_constraints = [
        ('name_company_uniq', 'unique(name, company_id, department_id)', 'The name of the job position must be unique per department in company!'),
    ]

    @api.depends('no_of_recruitment', 'employee_ids.job_id', 'employee_ids.active')
    def _compute_employees(self):
        employee_data = self.env['hr.employee'].read_group([('job_id', 'in', self.ids)], ['job_id'], ['job_id'])
        result = dict((data['job_id'][0], data['job_id_count']) for data in employee_data)
        for job in self:
            job.no_of_employee = result.get(job.id, 0)
            job.expected_employees = result.get(job.id, 0) + job.no_of_recruitment

    @api.model
    def create(self, values):
        """ We don't want the current user to be follower of all created job """
        return super(Job, self.with_context(mail_create_nosubscribe=True)).create(values)

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        self.ensure_one()
        default = dict(default or {})
        if 'name' not in default:
            default['name'] = _("%s (copy)") % (self.name)
        return super(Job, self).copy(default=default)

    def set_recruit(self):
        for record in self:
            no_of_recruitment = 1 if record.no_of_recruitment == 0 else record.no_of_recruitment
            record.write({'state': 'recruit', 'no_of_recruitment': no_of_recruitment})
        return True

    def set_open(self):
        return self.write({
            'state': 'open',
            'no_of_recruitment': 0,
            'no_of_hired_employee': 0
        })

```

## File: models\hr_plan.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class HrPlanActivityType(models.Model):
    _name = 'hr.plan.activity.type'
    _description = 'Plan activity type'
    _rec_name = 'summary'

    activity_type_id = fields.Many2one(
        'mail.activity.type', 'Activity Type',
        default=lambda self: self.env.ref('mail.mail_activity_data_todo'),
        domain=lambda self: ['|', ('res_model_id', '=', False), ('res_model_id', '=', self.env['ir.model']._get('hr.employee').id)]
    )
    summary = fields.Char('Summary')
    responsible = fields.Selection([
        ('coach', 'Coach'),
        ('manager', 'Manager'),
        ('employee', 'Employee'),
        ('other', 'Other')], default='employee', string='Responsible', required=True)
    responsible_id = fields.Many2one('res.users', 'Responsible Person', help='Specific responsible of activity if not linked to the employee.')
    note = fields.Html('Note')

    @api.onchange('activity_type_id')
    def _onchange_activity_type_id(self):
        if self.activity_type_id and self.activity_type_id.summary and not self.summary:
            self.summary = self.activity_type_id.summary

    def get_responsible_id(self, employee):
        if self.responsible == 'coach':
            if not employee.coach_id:
                raise UserError(_('Coach of employee %s is not set.') % employee.name)
            responsible = employee.coach_id.user_id
            if not responsible:
                raise UserError(_('User of coach of employee %s is not set.') % employee.name)
        elif self.responsible == 'manager':
            if not employee.parent_id:
                raise UserError(_('Manager of employee %s is not set.') % employee.name)
            responsible = employee.parent_id.user_id
            if not responsible:
                raise UserError(_('User of manager of employee %s is not set.') % employee.name)
        elif self.responsible == 'employee':
            responsible = employee.user_id
            if not responsible:
                raise UserError(_('User linked to employee %s is required.') % employee.name)
        elif self.responsible == 'other':
            responsible = self.responsible_id
            if not responsible:
                raise UserError(_('No specific user given on activity.'))
        return responsible


class HrPlan(models.Model):
    _name = 'hr.plan'
    _description = 'plan'

    name = fields.Char('Name', required=True)
    plan_activity_type_ids = fields.Many2many('hr.plan.activity.type', string='Activities')
    active = fields.Boolean(default=True)

```

## File: models\mail_alias.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, tools


class Alias(models.Model):
    _inherit = 'mail.alias'

    alias_contact = fields.Selection(selection_add=[('employees', 'Authenticated Employees')])


class MailAlias(models.AbstractModel):
    _inherit = 'mail.alias.mixin'

    def _alias_check_contact_on_record(self, record, message, message_dict, alias):
        if alias.alias_contact == 'employees':
            email_from = tools.decode_message_header(message, 'From')
            email_address = tools.email_split(email_from)[0]
            employee = self.env['hr.employee'].search([('work_email', 'ilike', email_address)], limit=1)
            if not employee:
                employee = self.env['hr.employee'].search([('user_id.email', 'ilike', email_address)], limit=1)
            if not employee:
                return {
                    'error_message': 'restricted to employees',
                    'error_template': self.env.ref('hr.mail_template_data_unknown_employee_email_address').body_html,
                }
            return True
        return super(MailAlias, self)._alias_check_contact_on_record(record, message, message_dict, alias)

```

## File: models\mail_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Channel(models.Model):
    _inherit = 'mail.channel'

    subscription_department_ids = fields.Many2many(
        'hr.department', string='HR Departments',
        help='Automatically subscribe members of those departments to the channel.')

    def _subscribe_users(self):
        """ Auto-subscribe members of a department to a channel """
        super(Channel, self)._subscribe_users()
        for mail_channel in self:
            if mail_channel.subscription_department_ids:
                mail_channel.write(
                    {'channel_partner_ids':
                        [(4, partner_id) for partner_id in mail_channel.mapped('subscription_department_ids.member_ids.user_id.partner_id').ids]})

    def write(self, vals):
        res = super(Channel, self).write(vals)
        if vals.get('subscription_department_ids'):
            self._subscribe_users()
        return res

```

## File: models\resource.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResourceResource(models.Model):
    _inherit = "resource.resource"

    user_id = fields.Many2one(copy=False)

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

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    resource_calendar_id = fields.Many2one(
        'resource.calendar', 'Company Working Hours',
        related='company_id.resource_calendar_id', readonly=False)
    module_hr_org_chart = fields.Boolean(string="Organizational Chart")
    module_hr_presence = fields.Boolean(string="Advanced Presence Control")
    module_hr_skills = fields.Boolean(string="Skills Management")
    hr_presence_control_login = fields.Boolean(string="Based on user status in system", config_parameter='hr.hr_presence_control_login')
    hr_presence_control_email = fields.Boolean(string="Based on number of emails sent", config_parameter='hr_presence.hr_presence_control_email')
    hr_presence_control_ip = fields.Boolean(string="Based on IP Address", config_parameter='hr_presence.hr_presence_control_ip')
    module_hr_attendance = fields.Boolean(string="Based on attendances")
    hr_presence_control_email_amount = fields.Integer(related="company_id.hr_presence_control_email_amount", readonly=False)
    hr_presence_control_ip_list = fields.Char(related="company_id.hr_presence_control_ip_list", readonly=False)
    hr_employee_self_edit = fields.Boolean(string="Employee Edition", config_parameter='hr.hr_employee_self_edit')

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.exceptions import AccessError


class Partner(models.Model):

    _inherit = ['res.partner']

    @api.model
    def get_static_mention_suggestions(self):
        """ Extend the mail's static mention suggestions by adding the employees. """
        suggestions = super(Partner, self).get_static_mention_suggestions()

        try:
            employee_group = self.env.ref('base.group_user')
            hr_suggestions = [{'id': user.partner_id.id, 'name': user.name, 'email': user.email} for user in employee_group.users]
            suggestions.append(hr_suggestions)
            return suggestions
        except AccessError:
            return suggestions

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

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields, _, SUPERUSER_ID
from odoo.exceptions import AccessError


class User(models.Model):
    _inherit = ['res.users']

    # note: a user can only be linked to one employee per company (see sql constraint in ´hr.employee´)
    employee_ids = fields.One2many('hr.employee', 'user_id', string='Related employee')
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
    work_location = fields.Char(related='employee_id.work_location', readonly=False, related_sudo=False)
    employee_parent_id = fields.Many2one(related='employee_id.parent_id', readonly=False, related_sudo=False)
    coach_id = fields.Many2one(related='employee_id.coach_id', readonly=False, related_sudo=False)
    address_home_id = fields.Many2one(related='employee_id.address_home_id', readonly=False, related_sudo=False)
    is_address_home_a_company = fields.Boolean(related='employee_id.is_address_home_a_company', readonly=False, related_sudo=False)
    private_email = fields.Char(related='address_home_id.email', string="Private Email", readonly=False)
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

    can_edit = fields.Boolean(compute='_compute_can_edit')

    def _compute_can_edit(self):
        can_edit = self.env['ir.config_parameter'].sudo().get_param('hr.hr_employee_self_edit') or self.env.user.has_group('hr.group_hr_user')
        for user in self:
            user.can_edit = can_edit

    @api.depends('employee_ids')
    def _compute_employee_count(self):
        for user in self.with_context(active_test=False):
            user.employee_count = len(user.employee_ids)

    def __init__(self, pool, cr):
        """ Override of __init__ to add access rights.
            Access rights are disabled by default, but allowed
            on some specific fields defined in self.SELF_{READ/WRITE}ABLE_FIELDS.
        """
        hr_readable_fields = [
            'active',
            'child_ids',
            'employee_id',
            'employee_ids',
            'employee_parent_id',
            'hr_presence_state',
            'last_activity',
            'last_activity_time',
            'can_edit',
        ]

        hr_writable_fields = [
            'additional_note',
            'address_home_id',
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
            'work_location',
            'work_phone',
            'certificate',
            'study_field',
            'study_school',
        ]

        init_res = super(User, self).__init__(pool, cr)
        # duplicate list to avoid modifying the original reference
        type(self).SELF_READABLE_FIELDS = type(self).SELF_READABLE_FIELDS + hr_readable_fields + hr_writable_fields
        type(self).SELF_WRITEABLE_FIELDS = type(self).SELF_WRITEABLE_FIELDS + hr_writable_fields
        return init_res

    @api.model
    def fields_view_get(self, view_id=None, view_type='form', toolbar=False, submenu=False):
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
        return super(User, self).fields_view_get(view_id=view_id, view_type=view_type, toolbar=toolbar, submenu=submenu)

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
            raise AccessError(_("You are only allowed to update your preferences. Please contact a HR officer to update other informations."))

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
                self.env['hr.employee'].sudo().search([('user_id', 'in', self.ids)]).write(employee_values)
        return result

    @api.model
    def action_get(self):
        if self.env.user.employee_id:
            return self.sudo().env.ref('hr.res_users_action_my').read()[0]
        return super(User, self).action_get()

    @api.depends('employee_ids')
    @api.depends_context('force_company')
    def _compute_company_employee(self):
        for user in self:
            user.employee_id = self.env['hr.employee'].search([('id', 'in', user.employee_ids.ids), ('company_id', '=', self.env.company.id)], limit=1)

    def _search_company_employee(self, operator, value):
        employees = self.env['hr.employee'].search([
            ('name', operator, value),
            '|',
            ('company_id', '=', self.env.company.id),
            ('company_id', '=', False)
        ], order='company_id ASC')
        return [('id', 'in', employees.mapped('user_id').ids)]

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

from . import hr_employee_base 
from . import hr_employee
from . import hr_employee_category
from . import hr_employee_public
from . import hr_department
from . import hr_job
from . import hr_plan
from . import mail_alias
from . import mail_channel
from . import res_config_settings
from . import res_partner
from . import res_users
from . import res_company
from . import resource

```

## File: report\hr_employee_badge.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <report
        id="hr_employee_print_badge"
        string="Print Badge"
        model="hr.employee"
        report_type="qweb-pdf"
        name="hr.print_employee_badge"
        file="hr.print_employee_badge"
        print_report_name="'Print Badge - %s' % (object.name).replace('/', '')"
    />

    <template id="print_employee_badge">
        <t t-call="web.basic_layout">
            <div class="page">
                <t t-foreach="docs" t-as="employee">
                    <div class="col-md-6">
                        <table style="width:243pt; height:153pt; border: 1pt solid black; border-collapse:separate; border-radius:8pt; margin:5pt">
                            <td style="width:33%;" valign="center">
                                <table style="width:77pt; height:150pt">
                                    <tr style="height:30%">
                                        <td align="center" valign="center">
                                            <img t-if="employee.company_id.logo" t-att-src="image_data_uri(employee.company_id.logo)" style="max-height:45pt;max-width:90%" alt="Company Logo"/>
                                        </td>
                                    </tr>
                                    <tr style="height:70%;">
                                        <td align="center" valign="center">
                                            <img t-if="employee.image_1920" t-att-src="image_data_uri(employee.image_1920)" style="max-height:85pt;max-width:90%" alt="Employee Image"/>
                                        </td>
                                    </tr>
                                </table>
                            </td>
                            <td style="width:67%" valign="center">
                                <table style="width:155pt; height:85pt">
                                    <tr><th><div style="font-size:15pt; margin-bottom:0pt;margin-top:0pt;" align="center"><t t-esc="employee.name"/></div></th></tr>
                                    <tr><td><div align="center" style="font-size:10pt;margin-bottom:5pt;"><t t-esc="employee.job_id.name"/></div></td></tr>
                                    <tr><td><img alt="barcode" t-if="employee.barcode" t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('Code128', employee.barcode, 600, 120)" style="max-height:50pt;max-width:100%;" align="center"/></td></tr>
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
        <field name="description">Helps you manage your employees.</field>
        <field name="sequence">9</field>
    </record>

    <record id="group_hr_user" model="res.groups">
        <field name="name">Officer</field>
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
        <field name="global" eval="True"/>
        <field name="domain_force">['|',('company_id','=',False),('company_id', 'in', company_ids)]</field>
    </record>

    <record id="hr_dept_comp_rule" model="ir.rule">
        <field name="name">Department multi company rule</field>
        <field name="model_id" ref="model_hr_department"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|',('company_id','=',False),('company_id', 'in', company_ids)]</field>
    </record>

    <record id="hr_employee_public_comp_rule" model="ir.rule">
        <field name="name">Employee multi company rule</field>
        <field name="model_id" ref="model_hr_employee_public"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|',('company_id','=',False),('company_id', 'in', company_ids)]</field>
    </record>

    <record id="hr_job_comp_rule" model="ir.rule">
        <field name="name">Job multi company rule</field>
        <field name="model_id" ref="model_hr_job"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|',('company_id','=',False),('company_id', 'in', company_ids)]</field>
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
access_hr_employee_system_user,hr.employee system user,model_hr_employee,base.group_user,0,0,0,0
access_hr_employee_public_user,hr.employee_public,model_hr_employee_public,base.group_user,1,0,0,0
access_hr_employee_resource_user,resource.resource.user,resource.model_resource_resource,group_hr_user,1,1,1,1
access_hr_department_user,hr.department.user,model_hr_department,group_hr_user,1,1,1,1
access_hr_department_employee,hr.department.employee,model_hr_department,base.group_user,1,0,0,0
access_hr_job_user,hr.job user,model_hr_job,group_hr_user,1,1,1,1
access_ir_property_hr_user,ir_property hr_user,base.model_ir_property,group_hr_user,1,1,1,0
access_hr_plan_activity_type_employee,access_hr_plan_activity_type,model_hr_plan_activity_type,base.group_user,1,0,0,0
access_hr_plan_employee,access_hr_plan_employee,model_hr_plan,base.group_user,1,0,0,0
access_hr_plan_activity_type_hr_user,access_hr_plan_activity_type,model_hr_plan_activity_type,group_hr_user,1,1,1,1
access_hr_plan_hr_user,access_hr_plan_hr_user,model_hr_plan,group_hr_user,1,1,1,1

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

## File: static\src\js\chat.js

```javascript
odoo.define('hr.employee_chat', function (require) {
'use strict';

    var FormController = require('web.FormController');
    var FormView = require('web.FormView');
    var FormRenderer = require('web.FormRenderer');
    var viewRegistry = require('web.view_registry');

    var EmployeeFormRenderer = FormRenderer.extend({

        /**
         * @override
         */
        _render: function () {
            var self = this;
            return this._super.apply(this, arguments).then(function () {
                var $chat_button = self.$el.find('.o_employee_chat_btn');
                if (self.state.context.uid === self.state.data.user_id.res_id) { // Hide the button for yourself
                    $chat_button.hide();
                }
                else {
                    $chat_button.off('click').on('click', self._onOpenChat.bind(self));
                }
            });
        },

        destroy: function () {
            this.$el.find('.o_employee_chat_btn').off('click');
            return this._super();
        },

        _onOpenChat: function(ev) {
            ev.preventDefault();
            ev.stopImmediatePropagation();
            this.trigger_up('open_chat', {
                partner_id: this.state.data.user_partner_id.res_id
            });
            return true;
        },
    });

    var EmployeeFormController = FormController.extend({
        custom_events: _.extend({}, FormController.prototype.custom_events, {
            open_chat: '_onOpenChat'
        }),

        _onOpenChat: function(ev) {
            var self = this;
            var dmChat = this.call('mail_service', 'getDMChatFromPartnerID', ev.data.partner_id);
            if (dmChat) {
                dmChat.detach();
            } else {
                var def = this.call('mail_service', 'createChannel', ev.data.partner_id, 'dm_chat').then(function (dmChatId) {
                    dmChat = self.call('mail_service', 'getChannel', dmChatId);
                    dmChat.detach();
                });
                Promise.resolve(def);
            }
        },
    });

    var EmployeeFormView = FormView.extend({
        config: _.extend({}, FormView.prototype.config, {
            Controller: EmployeeFormController,
            Renderer: EmployeeFormRenderer
        }),
    });

    viewRegistry.add('hr_employee_form', EmployeeFormView);
    return EmployeeFormView;
});

```

## File: static\src\js\language.js

```javascript
odoo.define('hr.employee_language', function (require) {
'use strict';

var FormController = require('web.FormController');
var FormView = require('web.FormView');
var viewRegistry = require('web.view_registry');

var EmployeeFormController = FormController.extend({
    saveRecord: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            if (arguments[0].indexOf('lang') >= 0) {
                self.do_action('reload_context');
            }
        });
    },
});

var EmployeeProfileFormView = FormView.extend({
    config: _.extend({}, FormView.prototype.config, {
        Controller: EmployeeFormController,
    }),
});

viewRegistry.add('hr_employee_profile_form', EmployeeProfileFormView);
return EmployeeProfileFormView;
});

```

## File: static\src\xml\hr_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<template xml:space="preserve">
    <t t-extend="UserMenu.Actions">
        <t t-jquery="a[data-menu='settings']" t-operation='inner'>
            My Profile
        </t>
    </t>
</template>

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
                    <sheet>
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
                        <field name="message_follower_ids" widget="mail_followers" groups="base.group_user"/>
                        <field name="message_ids" widget="mail_thread"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="view_department_tree" model="ir.ui.view">
            <field name="name">hr.department.tree</field>
            <field name="model">hr.department</field>
            <field name="arch" type="xml">
                <tree string="Companies">
                    <field name="display_name"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="manager_id"/>
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
                <kanban class="oe_background_grey o_kanban_dashboard o_hr_kanban">
                    <field name="name"/>
                    <field name="company_id"/>
                    <field name="manager_id"/>
                    <field name="color"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''}">
                                <div t-attf-class="o_kanban_card_header">
                                    <div class="o_kanban_card_header_title">
                                        <div class="o_primary"><field name="name"/></div>
                                        <div class="o_secondary"><field name="company_id" groups="base.group_multi_company"/></div>
                                    </div>
                                    <div class="o_kanban_manage_button_section" t-if="!selection_mode">
                                        <a class="o_kanban_manage_toggle_button" href="#"><i class="fa fa-ellipsis-v" role="img" aria-label="Manage" title="Manage"/></a>
                                    </div>
                                </div>
                                <div class="container o_kanban_card_content" t-if="!selection_mode">
                                    <div class="row">
                                        <div class="col-6 o_kanban_primary_left">
                                            <button class="btn btn-primary" name="%(act_employee_from_department)d" type="action">Employees</button>
                                        </div>
                                        <div class="col-6 o_kanban_primary_right">
                                        </div>
                                    </div>
                                </div><div class="container o_kanban_card_manage_pane dropdown-menu" role="menu">
                                    <div class="row">
                                        <div role="menuitem" class="col-4 o_kanban_card_manage_section o_kanban_manage_to_do">
                                            <div class="o_kanban_card_manage_title">
                                                <span>To Do</span>
                                            </div>
                                        </div>
                                        <div role="menuitem" class="col-4 o_kanban_card_manage_section o_kanban_manage_to_approve">
                                            <div class="o_kanban_card_manage_title">
                                                <span>To Approve</span>
                                            </div>
                                        </div>
                                        <div role="menuitem" class="col-4 o_kanban_card_manage_section o_kanban_manage_reports">
                                            <div class="o_kanban_card_manage_title">
                                                <span>Reporting</span>
                                            </div>
                                        </div>
                                    </div>

                                    <div t-if="widget.editable" class="o_kanban_card_manage_settings row">
                                        <div role="menuitem" aria-haspopup="true" class="col-8">
                                            <ul class="oe_kanban_colorpicker" data-field="color" role="menu"/>
                                        </div>
                                        <div class="col-4 text-right">
                                            <a role="menuitem" type="edit">Settings</a>
                                        </div>
                                    </div>
                                </div>

                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="open_module_tree_department" model="ir.actions.act_window">
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
                leaves, recruitments, etc.
              </p>
            </field>
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
                    <field name="company_id" string="Company"/>
                    <separator/>
                    <filter name="archived" string="Archived" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Group By">
                        <filter name="group_manager" string="Manager" domain="[]" context="{'group_by':'parent_id'}"/>
                        <filter name="group_department" string="Department" domain="[]" context="{'group_by':'department_id'}"/>
                        <filter name="group_company" string="Company" domain="[]" context="{'group_by':'company_id'}"/>
                    </group>
                    <searchpanel>
                        <field name="company_id" groups="base.group_multi_company" icon="fa-building"/>
                        <field name="department_id" icon="fa-users"/>
                    </searchpanel>
                </search>
             </field>
        </record>

        <record id="hr_employee_public_view_form" model="ir.ui.view">
            <field name="name">hr.employee.public.form</field>
            <field name="model">hr.employee.public</field>
            <field name="arch" type="xml">
                <form string="Employee" create="0" write="0">
                    <sheet>
                        <field name="user_id" invisible="1"/>
                        <field name="active" invisible="1"/>
                        <div class="oe_button_box" name="button_box">
                            <!-- Used by other modules-->
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <field name="image_1920" widget='image' class="oe_avatar" options='{"zoom": true, "preview_image":"image_128"}'/>
                            <div class="oe_title">
                                <label for="name" class="oe_edit_only"/>
                                <h1>
                                    <field name="name" placeholder="Employee's Name" required="True"/>
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
                                    <field name="work_location"/>
                                </group>
                                <group>
                                    <field name="department_id"/>
                                    <field name="company_id" groups="base.group_multi_company"/>
                                    <field name="parent_id"/>
                                </group>
                            </group>
                        <notebook>
                            <page name="public" string="Work Information">
                                <div id="o_work_employee_container"> <!-- These two div are used to position org_chart -->
                                    <div id="o_work_employee_main">
                                        <group string="Location">
                                            <field name="address_id"
                                                context="{'show_address': 1}"
                                                options='{"always_reload": True, "highlight_first_line": True}'/>
                                        </group>
                                        <group name="managers" string="Responsibles">
                                            <field name="coach_id"/>
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
                <tree string="Employees">
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
                <kanban class="o_hr_employee_kanban">
                    <field name="id"/>
                    <field name="hr_presence_state"/>
                    <templates>
                        <t t-name="kanban-box">
                        <div class="oe_kanban_global_click o_kanban_record_has_image_fill o_hr_kanban_record">
                            <field name="image_128" widget="image" class="o_kanban_image_fill_left" options="{'zoom': true, 'zoom_delay': 1000, 'background': true, 'preventClicks': false}"/>

                            <div class="oe_kanban_details">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings">
                                        <strong class="o_kanban_record_title">
                                            <div class="float-right" t-if="record.hr_presence_state.raw_value == 'present'">
                                                <span class="fa fa-circle text-success" role="img" aria-label="Present" title="Present" name="presence_present"></span>
                                            </div>
                                            <div class="float-right" t-if="record.hr_presence_state.raw_value == 'absent'">
                                                <span class="fa fa-circle text-danger" role="img" aria-label="Absent" title="Absent" name="presence_absent"></span>
                                            </div>
                                            <div class="float-right" t-if="record.hr_presence_state.raw_value == 'to_define'">
                                                <span class="fa fa-circle text-warning" role="img" aria-label="To define" title="To define" name="presence_to_define"></span>
                                            </div>
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
            <field name="context">{}</field>
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
                    <separator/>
                    <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction', '=', True)]"/>
                    <separator/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('activity_ids.date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('activity_ids.date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('activity_ids.date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Group By">
                        <filter name="group_manager" string="Manager" domain="[]" context="{'group_by': 'parent_id'}"/>
                        <filter name="group_department" string="Department" domain="[]" context="{'group_by': 'department_id'}"/>
                        <filter name="group_job" string="Job" domain="[]" context="{'group_by': 'job_id'}"/>
                    </group>
                    <searchpanel>
                        <field name="company_id" groups="base.group_multi_company" icon="fa-building"/>
                        <field name="department_id" icon="fa-users"/>
                    </searchpanel>
                </search>
             </field>
        </record>

        <record id="view_employee_form" model="ir.ui.view">
            <field name="name">hr.employee.form</field>
            <field name="model">hr.employee</field>
            <field name="arch" type="xml">
                <form string="Employee" js_class="hr_employee_form">
                    <field name="active" invisible="1"/>
                    <field name="user_partner_id" invisible="1"/>
                    <field name="hr_presence_state" invisible="1"/>
                    <header>
                        <button string="Chat" class="btn btn-primary o_employee_chat_btn" attrs="{'invisible': [('user_id','=', False)]}"/>
                        <button name="%(plan_wizard_action)d" string="Launch Plan" type="action" groups="hr.group_hr_manager"/>
                    </header>
                    <sheet>
                        <div name="button_box" class="oe_button_box">
                            <button
                                id="hr_presence_button"
                                class="oe_stat_button"
                                disabled="1"
                                attrs="{'invisible': ['|', ('last_activity', '=', False), ('user_id', '=', False)]}">
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
                                    <span class="o_stat_text">Present Since</span>
                                </div>
                            </button>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <field name="image_1920" widget='image' class="oe_avatar" options='{"zoom": true, "preview_image":"image_128"}'/>
                        <div class="oe_title">
                            <h1>
                                <field name="name" placeholder="Employee's Name" required="True"/>
                            </h1>
                            <h2>
                                <field name="job_title" placeholder="Job Position" />
                            </h2>
                            <field name="category_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}" placeholder="Tags"  groups="hr.group_hr_manager"/>
                        </div>
                        <group>
                            <group>
                                <field name="mobile_phone" widget="phone"/>
                                <field name="work_phone" widget="phone"/>
                                <field name="work_email" widget="email"/>
                                <field name="work_location"/>
                                <field name="company_id" groups="base.group_multi_company"/>
                            </group>
                            <group>
                                <field name="department_id"/>
                                <field name="job_id"/>
                                <field name="parent_id"/>
                            </group>
                        </group>
                        <notebook>
                            <page name="public" string="Work Information">
                                <div id="o_work_employee_container"> <!-- These two div are used to position org_chart -->
                                    <div id="o_work_employee_main">
                                        <group string="Location">
                                            <field name="address_id"
                                                context="{'show_address': 1}"
                                                options='{"always_reload": True, "highlight_first_line": True}'/>
                                        </group>
                                        <group name="managers" string="Responsibles">
                                            <field name="coach_id"/>
                                        </group>
                                        <group name="departure" string="Departure" attrs="{'invisible': [('active', '=', True)]}">
                                            <field name="departure_reason"/>
                                            <field name="departure_description"/>
                                        </group>
                                        <group string="Schedule" groups="base.group_no_one">
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
                                        <field name="phone" class="o_force_ltr" groups="hr.group_hr_user" string="Phone" readonly="True"/>
                                        <field name="bank_account_id" context="{'default_partner_id': address_home_id}"/>
                                        <field name="km_home_work" groups="hr.group_hr_user"/>
                                    </group>
                                    <group string="Citizenship">
                                        <field name="country_id" options='{"no_open": True, "no_create": True}'/>
                                        <field name="identification_id" groups="hr.group_hr_user"/>
                                        <field name="passport_id" groups="hr.group_hr_user"/>
                                        <field name="gender"/>
                                        <field name="birthday"/>
                                        <field name="place_of_birth" groups="hr.group_hr_user"/>
                                        <field name="country_of_birth" groups="hr.group_hr_user"/>
                                    </group>
                                    <group string="Marital Status">
                                        <field name="marital"/>
                                        <field name="spouse_complete_name" attrs="{'invisible': [('marital', 'not in', ['married', 'cohabitant'])]}" groups="hr.group_hr_user"/>
                                        <field name="spouse_birthdate" attrs="{'invisible': [('marital', 'not in', ['married', 'cohabitant'])]}" groups="hr.group_hr_user"/>
                                    </group>
                                    <group string="Dependant">
                                        <field name="children"/>
                                    </group>
                                    <group string="Emergency">
                                        <field name="emergency_contact"/>
                                        <field name="emergency_phone" class="o_force_ltr"/>
                                    </group>
                                    <group string="Work Permit" name="work_permit">
                                        <field name="visa_no"/>
                                        <field name="permit_no"/>
                                        <field name="visa_expire"/>
                                    </group>
                                    <group string="Education">
                                        <field name="certificate"/>
                                        <field name="study_field"/>
                                        <field name="study_school"/>
                                    </group>
                                </group>
                            </page>
                            <page name="hr_settings" string="HR Settings" groups="hr.group_hr_user">
                                <group>
                                    <group string='Status' name="active_group">
                                        <field name="user_id" string="Related User"/>
                                    </group>
                                    <group string="Attendance" name="identification_group">
                                        <field name="pin" string="PIN Code"/>
                                        <label for="barcode"/>
                                        <div class="o_row">
                                            <field name="barcode"/>
                                            <button string="Generate" class="btn btn-link" type="object" name="generate_random_barcode" attrs="{'invisible': [('barcode', '!=', False)]}"/>
                                            <button name="%(hr_employee_print_badge)d" string="Print Badge" class="btn btn-link" type="action" attrs="{'invisible': [('barcode', '=', False)]}"/>
                                        </div>
                                    </group>
                                </group>
                            </page>
                        </notebook>

                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" widget="mail_followers" groups="base.group_user"/>
                        <field name="activity_ids" widget="mail_activity"/>
                        <field name="message_ids" widget="mail_thread"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="view_employee_tree" model="ir.ui.view">
            <field name="name">hr.employee.tree</field>
            <field name="model">hr.employee</field>
            <field name="arch" type="xml">
                <tree string="Employees">
                    <field name="name"/>
                    <field name="work_phone" class="o_force_ltr"/>
                    <field name="work_email"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="department_id"/>
                    <field name="job_id"/>
                    <field name="parent_id"/>
                    <field name="coach_id" invisible="1"/>
                </tree>
            </field>
        </record>

        <record id="hr_kanban_view_employees" model="ir.ui.view">
           <field name="name">hr.employee.kanban</field>
           <field name="model">hr.employee</field>
           <field name="priority">10</field>
           <field name="arch" type="xml">
               <kanban class="o_hr_employee_kanban">
                   <field name="id"/>
                   <field name="hr_presence_state"/>
                   <templates>
                       <t t-name="kanban-box">
                       <div class="oe_kanban_global_click o_kanban_record_has_image_fill o_hr_kanban_record">
                           <field name="image_128" widget="image" class="o_kanban_image_fill_left" options="{'zoom': true, 'zoom_delay': 1000, 'background': true, 'preventClicks': false}"/>

                            <div class="oe_kanban_details">
                               <div class="o_kanban_record_top">
                                   <div class="o_kanban_record_headings">
                                       <strong class="o_kanban_record_title">
                                            <div class="float-right" t-if="record.hr_presence_state.raw_value == 'present'">
                                                <span class="fa fa-circle text-success" role="img" aria-label="Present" title="Present" name="presence_present"></span>
                                            </div>
                                            <div class="float-right" t-if="record.hr_presence_state.raw_value == 'absent'">
                                                <span class="fa fa-circle text-danger" role="img" aria-label="Absent" title="Absent" name="presence_absent"></span>
                                            </div>
                                            <div class="float-right" t-if="record.hr_presence_state.raw_value == 'to_define'">
                                                <span class="fa fa-circle text-warning" role="img" aria-label="To define" title="To define" name="presence_to_define"></span>
                                            </div>
                                            <field name="name" placeholder="Employee's Name"/>
                                       </strong>
                                       <span t-if="record.job_title.raw_value" class="o_kanban_record_subtitle"><field name="job_title"/></span>
                                   </div>
                               </div>
                               <field name="category_ids" widget="many2many_tags" options="{'color_field': 'color'}" groups="hr.group_hr_manager"/>
                               <ul>
                                   <li id="last_login"/>
                                   <li t-if="record.work_email.raw_value" class="o_text_overflow"><field name="work_email" /></li>
                                   <li t-if="record.work_phone.raw_value" class="o_force_ltr"><field name="work_phone" /></li>
                               </ul>
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
                            <img t-att-src="activity_image('hr.employee', 'image_128', record.id.raw_value)" role="img" t-att-title="record.id.value" t-att-alt="record.id.value"/>
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
            <field name="context">{}</field>
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

        <record id="act_employee_from_department" model="ir.actions.act_window">
            <field name="name">Employees</field>
            <field name="res_model">hr.employee</field>
            <field name="view_mode">kanban,form,tree</field>
            <field name="search_view_id" ref="view_employee_filter"/>
            <field name="context">{
                "searchpanel_default_department_id": active_id,
                "default_department_id": active_id}
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
                    <header>
                        <button name="set_recruit" string="Start Recruitment" states="open" type="object" class="oe_highlight" groups="base.group_user"/>
                        <button name="set_open" string="Stop Recruitment" states="recruit" type="object" groups="base.group_user"/>
                        <field name="state" widget="statusbar" statusbar_visible="recruit,open"/>
                    </header>
                    <sheet>
                        <div class="oe_button_box" name="button_box"/>
                        <div class="oe_title">
                            <label for="name" class="oe_edit_only"/>
                            <h1><field name="name" placeholder="e.g. Sales Manager"/></h1>
                        </div>
                        <group>
                            <group name="recruitment">
                                <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                                <field name="department_id"/>
                            </group>
                            <group>
                                <field name="no_of_recruitment"/>
                            </group>
                        </group>
                        <div attrs="{'invisible': [('state', '!=', 'recruit')]}">
                            <label for="description"/>
                            <field name="description"/>
                        </div>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" widget="mail_followers"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="view_hr_job_tree" model="ir.ui.view">
            <field name="name">hr.job.tree</field>
            <field name="model">hr.job</field>
            <field name="arch" type="xml">
                <tree string="Job" decoration-bf="message_needaction==True">
                    <field name="name"/>
                    <field name="department_id"/>
                    <field name="no_of_employee"/>
                    <field name="no_of_recruitment"/>
                    <field name="expected_employees"/>
                    <field name="no_of_hired_employee"/>
                    <field name="state"/>
                    <field name="message_needaction" invisible="1"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id="hr_job_view_kanban" model="ir.ui.view">
            <field name="name">hr.job.kanban</field>
            <field name="model">hr.job</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
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
                                    <span t-att-class="record.state.raw_value == 'recruit'  and 'float-right badge badge-success' or 'float-right badge badge-danger'">
                                        <field name="state"/>
                                    </span>
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
                    <filter name="in_position" string="In Position" domain="[('state', '=', 'open')]"/>
                    <filter name="in_recruitment" string="In Recruitment" domain="[('state', '=', 'recruit')]"/>
                    <separator/>
                    <filter name="message_needaction" string="Unread Messages" domain="[('message_needaction', '=', True)]"/>
                    <group expand="0" string="Group By">
                        <filter string="Department" name="department" domain="[]" context="{'group_by': 'department_id'}"/>
                        <filter string="Status" name="status" domain="[]" context="{'group_by': 'state'}"/>
                        <filter string="Company" name="company" domain="[]" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
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
                </tree>
            </field>
        </record>

        <record id="hr_plan_view_form" model="ir.ui.view">
            <field name="name">hr.plan.view.form</field>
            <field name="model">hr.plan</field>
            <field name="arch" type="xml">
                <form string="Planning">
                    <sheet>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <div class="oe_title">
                            <label for="name" class="oe_edit_only"/>
                            <h1>
                                <field name="name"/>
                            </h1>
                        </div>
                        <group string="Activities">
                            <field name="active" invisible="1"/>
                            <field name="plan_activity_type_ids" nolabel="1">
                                <tree editable="bottom">
                                    <field name="activity_type_id"/>
                                    <field name="summary"/>
                                    <field name="responsible"/>
                                    <field name="responsible_id" attrs="{'readonly': [('responsible', '!=', 'other')]}"/>
                                </tree>
                            </field>
                        </group>
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
                </tree>
            </field>
        </record>

        <record id="hr_plan_activity_type_view_form" model="ir.ui.view">
            <field name="name">hr.plan.activity.type.view.form</field>
            <field name="model">hr.plan.activity.type</field>
            <field name="arch" type="xml">
                <form string="Activity">
                    <sheet>
                        <group>
                            <field name="activity_type_id"/>
                            <field name="summary"/>
                            <field name="responsible"/>
                            <field name="responsible_id" attrs="{'invisible': [('responsible', '!=', 'other')]}"/>
                            <field name="note"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="hr_plan_action" model="ir.actions.act_window">
            <field name="name">Planning</field>
            <field name="res_model">hr.plan</field>
            <field name="view_mode">tree,form</field>
            <field name="search_view_id" ref="hr_plan_view_search"/>
        </record>

        <record id="hr_plan_activity_type_action" model="ir.actions.act_window">
            <field name="name">Planning Types</field>
            <field name="res_model">hr.plan.activity.type</field>
            <field name="view_mode">tree,form</field>
        </record>

    </data>
</odoo>

```

## File: views\hr_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="assets_backend" name="hr assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/hr/static/src/scss/hr.scss"/>
            <script type="text/javascript" src="/hr/static/src/js/chat.js"></script>
            <script type="text/javascript" src="/hr/static/src/js/language.js"></script>
        </xpath>
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
            sequence="75"/>

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
            name="Employee Directory"
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
            groups="group_hr_manager"
            sequence="100"/>

            <menuitem
                id="menu_view_hr_job"
                action="action_hr_job"
                parent="menu_human_resources_configuration"
                sequence="1"/>

            <menuitem
                id="menu_human_resources_configuration_employee"
                name="Employee"
                parent="menu_human_resources_configuration"
                groups="base.group_no_one"
                sequence="1"/>

                <menuitem
                    id="menu_view_employee_category_form"
                    name="Tags"
                    action="open_view_categ_form"
                    parent="menu_human_resources_configuration_employee"
                    groups="base.group_no_one"
                    sequence="1"/>

            <menuitem
                id="menu_hr_department_tree"
                action="open_module_tree_department"
                parent="menu_human_resources_configuration"
                sequence="2"
                groups="group_hr_user"/>

            <menuitem
                id="menu_config_plan"
                name="Activity Planning"
                parent="menu_human_resources_configuration"
                groups="group_hr_manager"
                sequence="100"/>

                <menuitem
                    id="menu_config_plan_types"
                    name="Planning Types"
                    action="hr_plan_activity_type_action"
                    parent="menu_config_plan"
                    groups="base.group_no_one"
                    sequence="99"/>

                <menuitem
                    id="menu_config_plan_plan"
                    name="Plans"
                    action="hr_plan_action"
                    parent="menu_config_plan"
                    groups="group_hr_manager"
                    sequence="100"/>

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
                    <div class="row mt16 o_settings_container">
                        <div class="col-12 col-lg-6 o_setting_box">
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
                        <div class="col-12 col-lg-6 o_setting_box" title="Presence of employees">

                            <div class="o_setting_right_pane">
                                <span class="o_form_label">Presence Control</span>
                                <div class="content-group" name="hr_presence_options">
                                    <div class="row">
                                        <field name="module_hr_attendance" class="col-lg-1 ml16"/>
                                        <label for="module_hr_attendance" class="o_light_label"/>
                                    </div>
                                    <div class="row">
                                        <field name="hr_presence_control_login" class="col-lg-1 ml16"/>
                                        <label for="hr_presence_control_login" class="o_light_label"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" title="Advanced presence of employees">
                            <div class="o_setting_left_pane">
                                <field name="module_hr_presence"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_hr_presence"/>
                                <div class="text-muted" name="hr_presence_options_advanced">
                                    Presence reporting screen, email and IP address control.
                                </div>
                                <div class="row mt-1" attrs="{'invisible': [('module_hr_presence', '=', False)]}">
                                    <field name="hr_presence_control_email" class="col-lg-1 ml16"/>
                                    <label for="hr_presence_control_email" class="o_light_label"/>
                                </div>
                                <div class="row ml32" attrs="{'invisible': ['|', ('module_hr_presence', '=', False), ('hr_presence_control_email', '=', False)]}">
                                    <span class="ml8 mr-2">Minimum number of emails to sent </span>
                                    <field name="hr_presence_control_email_amount" class="ml-2 oe_inline"/>
                                </div>
                                <div class="row" attrs="{'invisible': [('module_hr_presence', '=', False)]}">
                                    <field name="hr_presence_control_ip" class="col-lg-1 ml16"/>
                                    <label for="hr_presence_control_ip" class="o_light_label"/>
                                </div>
                                <div class="row ml32" attrs="{'invisible': ['|', ('module_hr_presence', '=', False), ('hr_presence_control_ip', '=', False)]}">
                                    <span class="ml8 mr-2">IP Addresses (comma-separated)</span>
                                    <field name="hr_presence_control_ip_list" class="ml-2 oe_inline"/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <h2>Work Organization</h2>
                    <div class="row mt16 o_settings_container">
                        <div class="col-12 col-lg-6 o_setting_box">
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
                                            domain="[('company_id', '=', company_id)]"
                                            context="{'default_company_id': company_id}"/>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div class="col-12 col-lg-6 o_setting_box" title="Show organizational chart on employee form">
                            <div class="o_setting_left_pane">
                                <field name="module_hr_org_chart"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_hr_org_chart"/>
                                <div class="text-muted">
                                    Show organizational chart on employee form
                                </div>
                            </div>
                        </div>
                    </div>
                    <h2>Employee Update Rights</h2>
                    <div class="row mt16 o_settings_container">
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
                <footer position="replace"/>
                <h1 position="replace"/>
                <widget name="notification_alert" position="replace"/>
                <xpath expr="//field[@name='image_1920']" position="replace"/>
                <field name="signature" position="replace" />
                <xpath expr="//group[@name='preference_contact']" position="before">
                    <group>
                        <field name="signature" options="{'style-inline': true}"/>
                    </group>
                </xpath>
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
            </field>
        </record>

        <record id="res_users_view_form_profile" model="ir.ui.view">
            <field name="name">res.users.preferences.form.inherit</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="res_users_view_form_simple_modif"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='odoobot_state']" position="attributes">
                    <attribute name="attrs">{'readonly': [('can_edit', '=', False)]}</attribute>
                </xpath>
                <form position="replace">
                <form create="false" js_class="hr_employee_profile_form">
                    <field name="hr_presence_state" invisible="1"/>
                    <header>
                        <button name="preference_change_password" type="object" string="Change password" class="btn btn-secondary"/>
                    </header>
                    <widget name="notification_alert" class="mb-0"/>
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button
                                id="hr_presence_button"
                                class="oe_stat_button"
                                disabled="1"
                                invisible="context.get('from_my_profile', False)"
                                attrs="{'invisible': ['|', ('hr_presence_state', '=', 'absent')]}">
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
                        <field name="image_1920" widget='image' class="oe_avatar" options='{"zoom": true, "preview_image":"image_128"}'/>
                        <div class="oe_title">
                            <h1>
                                <field name="name" placeholder="Employee's Name" required="True" readonly="context.get('from_my_profile', False)"/>
                            </h1>
                        </div>
                        <div class="row">
                            <h2 class="col-6">
                                <field name="job_title" placeholder="Job Position" attrs="{'readonly': [('can_edit', '=', False)]}"/>
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
                                <field name="work_location" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                <field name="company_id" invisible="1"/>
                            </group>
                        </group>
                        <notebook>
                            <page name="preferences" string="Preferences">
                                <div>$0</div>
                            </page>
                            <page name="public" string="Work Information">
                                <div id="o_work_employee_container"> <!-- These two div are used to position org_chart -->
                                    <div id="o_work_employee_main">
                                        <group string="Location">
                                            <field name="department_id" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                            <field name="address_id"
                                                context="{'show_address': 1}"
                                                options='{"always_reload": True, "highlight_first_line": True}'
                                                attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                        </group>
                                        <group name="managers" string="Responsibles">
                                            <field name="employee_parent_id" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                            <field name="coach_id" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                        </group>
                                    </div>
                                </div>
                            </page>
                            <page name="personal_information" string="Private Information">
                                <group>
                                    <group string="Contact Information">
                                        <field name="employee_ids" invisible="1"/>
                                        <field name="address_home_id"
                                            context="{
                                                'show_address': 1,
                                                'default_employee_ids': employee_ids,
                                                'default_type': 'private',
                                                'form_view_ref': 'base.res_partner_view_form_private'}"
                                            options='{"always_reload": True, "highlight_first_line": True}'
                                            attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                        <field name="private_email" string="Email" attrs="{'readonly': [('can_edit', '=', False)], 'invisible': [('address_home_id', '=', False)]}"/>
                                        <field name="employee_phone" string="Phone" class="o_force_ltr" attrs="{'readonly': [('can_edit', '=', False)], 'invisible': [('address_home_id', '=', False)]}"/>
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
                                        <field name="emergency_phone" class="o_force_ltr" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                    </group>
                                    <group string="Work Permit" name="work_permit">
                                        <field name="visa_no" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                        <field name="permit_no" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                        <field name="visa_expire" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                    </group>
                                </group>
                            </page>
                             <page name="hr_settings" string="HR Settings">
                                <group>
                                    <group string='Status' name="active_group" invisible="1"></group>
                                    <group string="Attendance" name="identification_group">
                                        <field name="pin" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                        <field name="barcode" attrs="{'readonly': [('can_edit', '=', False)]}"/>
                                    </group>
                                </group>
                            </page>
                        </notebook>
                    </sheet>
                </form>
                </form>
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

from odoo import api, fields, models


class HrDepartureWizard(models.TransientModel):
    _name = 'hr.departure.wizard'
    _description = 'Departure Wizard'

    @api.model
    def default_get(self, fields):
        res = super(HrDepartureWizard, self).default_get(fields)
        if (not fields or 'employee_id' in fields) and 'employee_id' not in res:
            if self.env.context.get('active_id'):
                res['employee_id'] = self.env.context['active_id']
        return res

    departure_reason = fields.Selection([
        ('fired', 'Fired'),
        ('resigned', 'Resigned'),
        ('retired', 'Retired')
    ], string="Departure Reason", default="fired")
    departure_description = fields.Text(string="Additional Information")
    plan_id = fields.Many2one('hr.plan', default=lambda self: self.env['hr.plan'].search([], limit=1))
    employee_id = fields.Many2one('hr.employee', string='Employee', required=True)

    def action_register_departure(self):
        employee = self.employee_id
        employee.departure_reason = self.departure_reason
        employee.departure_description = self.departure_description

        if not employee.user_id.partner_id:
            return

        for activity_type in self.plan_id.plan_activity_type_ids:
            self.env['mail.activity'].create({
                'res_id': employee.user_id.partner_id.id,
                'res_model_id': self.env['ir.model']._get('res.partner').id,
                'activity_type_id': activity_type.activity_type_id.id,
                'summary': activity_type.summary,
                'user_id': activity_type.get_responsible_id(employee).id,
            })

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
                        <group>
                            <field name="plan_id"/>
                            <field name="employee_id" invisible="1"/>
                            <field name="departure_reason"/>
                            <field name="departure_description"/>
                        </group>
                    </sheet>
                    <footer>
                        <button name="action_register_departure" string="Save" type="object" class="oe_highlight"/>
                        <button string="Cancel" class="btn-secondary" special="cancel"/>
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

from odoo import api, fields, models


class HrPlanWizard(models.TransientModel):
    _name = 'hr.plan.wizard'
    _description = 'Plan Wizard'

    @api.model
    def default_get(self, fields_list):
        res = super(HrPlanWizard, self).default_get(fields_list)
        if (not fields_list or 'employee_id' in fields_list) and 'employee_id' not in res:
            if self.env.context.get('active_id'):
                res['employee_id'] = self.env.context['active_id']
        return res

    plan_id = fields.Many2one('hr.plan', default=lambda self: self.env['hr.plan'].search([], limit=1))
    employee_id = fields.Many2one('hr.employee', string='Employee', required=True)

    def action_launch(self):
        for activity_type in self.plan_id.plan_activity_type_ids:
            responsible = activity_type.get_responsible_id(self.employee_id)

            if self.env['hr.employee'].with_user(responsible).check_access_rights('read', raise_exception=False):
                date_deadline = self.env['mail.activity']._calculate_date_deadline(activity_type.activity_type_id)
                self.env['mail.activity'].create({
                    'res_id': self.employee_id.id,
                    'res_model_id': self.env['ir.model']._get('hr.employee').id,
                    'summary': activity_type.summary,
                    'note': activity_type.note,
                    'activity_type_id': activity_type.activity_type_id.id,
                    'user_id': responsible.id,
                    'date_deadline': date_deadline,
                })

        return {
            'type': 'ir.actions.act_window',
            'res_model': 'hr.employee',
            'res_id': self.employee_id.id,
            'name': self.employee_id.display_name,
            'view_mode': 'form',
            'views': [(False, "form")],
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
                            <field name="plan_id"/>
                            <field name="employee_id" invisible="1"/>
                        </group>
                    </sheet>
                    <footer>
                        <button name="action_launch" string="Launch Plan" type="object" class="oe_highlight" groups="hr.group_hr_manager"/>
                        <button string="Cancel" class="btn-secondary" special="cancel"/>
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

