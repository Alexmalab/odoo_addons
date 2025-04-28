# Odoo Module: gamification

Category: Human Resources

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
    'name': 'Gamification',
    'version': '1.0',
    'sequence': 160,
    'category': 'Human Resources',
    'depends': ['mail'],
    'description': """
Gamification process
====================
The Gamification module provides ways to evaluate and motivate the users of Odoo.

The users can be evaluated using goals and numerical objectives to reach.
**Goals** are assigned through **challenges** to evaluate and compare members of a team with each others and through time.

For non-numerical achievements, **badges** can be granted to users. From a simple "thank you" to an exceptional achievement, a badge is an easy way to exprimate gratitude to a user for their good work.

Both goals and badges are flexibles and can be adapted to a large range of modules and actions. When installed, this module creates easy goals to help new users to discover Odoo and configure their user profile.
""",

    'data': [
        'wizard/update_goal.xml',
        'wizard/grant_badge.xml',
        'views/res_users_views.xml',
        'views/gamification_karma_rank_views.xml',
        'views/gamification_karma_tracking_views.xml',
        'views/gamification_badge_views.xml',
        'views/gamification_badge_user_views.xml',
        'views/gamification_goal_views.xml',
        'views/gamification_goal_definition_views.xml',
        'views/gamification_challenge_views.xml',
        'views/gamification_challenge_line_views.xml',
        'views/gamification_menus.xml',
        'security/gamification_security.xml',
        'security/ir.model.access.csv',
        'data/ir_cron_data.xml',
        'data/mail_template_data.xml',  # keep before to populate challenge reports
        'data/gamification_badge_data.xml',
        'data/gamification_challenge_data.xml',
        'data/gamification_karma_rank_data.xml',
    ],
    'demo': [
        'data/gamification_karma_rank_demo.xml',
        'data/gamification_karma_tracking_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\gamification_badge_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="badge_good_job" model="gamification.badge">
            <field name="name">Good Job</field>
            <field name="description">You did great at your job.</field>
            <field name="rule_auth">everyone</field>
            <field name="image_1920" type="base64" file="gamification/static/img/badge_good_job-image.png"/>
        </record>

        <record id="badge_problem_solver" model="gamification.badge">
            <field name="name">Problem Solver</field>
            <field name="description">No one can solve challenges like you do.</field>
            <field name="rule_auth">everyone</field>
            <field name="image_1920" type="base64" file="gamification/static/img/badge_problem_solver-image.png"/>
        </record>

        <record id="badge_hidden" model="gamification.badge">
            <field name="name">Hidden</field>
            <field name="description">You have found the hidden badge</field>
            <field name="rule_auth">nobody</field>
            <field name="image_1920" type="base64" file="gamification/static/img/badge_hidden-image.png"/>
            <field name="active" eval="False" />
        </record>

        <record id="badge_idea" model="gamification.badge">
            <field name="name">Brilliant</field>
            <field name="description">With your brilliant ideas, you are an inspiration to others.</field>
            <field name="rule_auth">everyone</field>
            <field name="rule_max">True</field>
            <field name="rule_max_number">2</field>
            <field name="image_1920" type="base64" file="gamification/static/img/badge_idea-image.png"/>
        </record>
    </data>
</odoo>

```

## File: data\gamification_challenge_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- goal definitions -->
        <record model="gamification.goal.definition" id="definition_base_timezone">
            <field name="name">Set your Timezone</field>
            <field name="description">Configure your profile and specify your timezone</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="base.model_res_users"/>
            <field name="domain">[('partner_id.tz', '!=', False)]</field>
            <field name="action_id" ref="base.action_res_users_my"/>
            <field name="res_id_field">user.id</field>
            <field name="batch_mode">True</field>
            <field name="batch_distinctive_field" ref="base.field_res_users__id"/>
            <field name="batch_user_expression">user.id</field>
        </record>

        <record model="gamification.goal.definition" id="definition_base_company_data">
            <field name="name">Set your Company Data</field>
            <field name="description">Write some information about your company (specify at least a name)</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="base.model_res_company"/>
            <field name="domain">[('user_ids', 'in', [user.id]), ('name', '=', 'YourCompany')]</field>
            <field name="condition">lower</field>
            <field name="action_id" ref="base.action_res_company_form"/>
            <field name="res_id_field">user.company_id.id</field>
        </record>

        <record model="gamification.goal.definition" id="definition_base_company_logo">
            <field name="name">Set your Company Logo</field>
            <field name="computation_mode">count</field>
            <field name="display_mode">boolean</field>
            <field name="model_id" ref="base.model_res_company"/>
            <field name="domain">[('user_ids', 'in', [user.id]),('logo', '!=', False)]</field>
            <field name="action_id" ref="base.action_res_company_form"/>
            <field name="res_id_field">user.company_id.id</field>
        </record>

        <record model="gamification.goal.definition" id="definition_base_invite">
            <field name="name">Invite new Users</field>
            <field name="description">Create at least another user</field>
            <field name="display_mode">boolean</field>
            <field name="computation_mode">count</field>
            <field name="model_id" ref="base.model_res_users"/>
            <field name="domain">[('id', '!=', user.id)]</field>
            <field name="action_id" ref="action_new_simplified_res_users"/>
        </record>

        <!-- challenges -->
        <record model="gamification.challenge" id="challenge_base_discover">
            <field name="name">Complete your Profile</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="user_domain" eval="str([('groups_id.id', '=', ref('base.group_user'))])" />
            <field name="state">inprogress</field>
            <field name="challenge_category">other</field>
        </record>

        <record model="gamification.challenge" id="challenge_base_configure">
            <field name="name">Setup your Company</field>
            <field name="period">once</field>
            <field name="visibility_mode">personal</field>
            <field name="report_message_frequency">never</field>
            <field name="user_domain" eval="str([('groups_id.id', '=', ref('base.group_erp_manager'))])" />
            <field name="state">inprogress</field>
            <field name="challenge_category">other</field>
        </record>

        <!-- lines -->
        <record model="gamification.challenge.line" id="line_base_discover1">
            <field name="definition_id" ref="definition_base_timezone"/>
            <field name="target_goal">1</field>
            <field name="challenge_id" ref="challenge_base_discover"/>
        </record>

        <record model="gamification.challenge.line" id="line_base_admin2">
            <field name="definition_id" ref="definition_base_company_logo"/>
            <field name="target_goal">1</field>
            <field name="challenge_id" ref="challenge_base_configure"/>
        </record>
        <record model="gamification.challenge.line" id="line_base_admin1">
            <field name="definition_id" ref="definition_base_company_data"/>
            <field name="target_goal">0</field>
            <field name="challenge_id" ref="challenge_base_configure"/>
        </record>
        <record model="gamification.challenge.line" id="line_base_admin3">
            <field name="definition_id" ref="definition_base_invite"/>
            <field name="target_goal">1</field>
            <field name="challenge_id" ref="challenge_base_configure"/>
        </record>
    </data>

</odoo>

```

## File: data\gamification_karma_rank_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!--Karma-->
    <record id="karma_tracking_user_root" model="gamification.karma.tracking" forcecreate="0">
        <field name="user_id" ref="base.user_root"/>
        <field name="new_value">2500</field>
        <field name="reason">I am the Root!</field>
    </record>
    <record id="karma_tracking_user_admin" model="gamification.karma.tracking" forcecreate="0">
        <field name="user_id" ref="base.user_admin"/>
        <field name="new_value">2500</field>
        <field name="reason">I am the Admin!</field>
    </record>

    <!--Ranks-->
    <record id="rank_newbie" model="gamification.karma.rank">
        <field name="name">Newbie</field>
        <field name="description" type="html"><p>You just began the adventure! Welcome!</p></field>
        <field name="description_motivational" type="html">
            <div class="d-flex align-items-center">
                <div class="flex-grow-1">Earn your first points and join the adventure!</div>
                <img class="ms-3 img img-fluid" style="max-height: 72px;" src="/gamification/static/img/rank_newbie_badge.svg" alt=""/>
            </div>
        </field>
        <field name="karma_min">1</field>
        <field name="image_1920" type="base64" file="gamification/static/img/rank_newbie_badge.svg"/>
    </record>

    <record id="rank_student" model="gamification.karma.rank">
        <field name="name">Student</field>
        <field name="description" type="html"><p>You're a young padawan now. May the force be with you!</p></field>
        <field name="description_motivational" type="html">
            <div class="d-flex align-items-center">
                <div class="flex-grow-1">Reach the next rank to show the rest of the world you exist.</div>
                <img class="ms-3 img img-fluid" style="max-height: 72px;" src="/gamification/static/img/rank_student_badge.svg" alt=""/>
            </div>
        </field>
        <field name="karma_min">100</field>
        <field name="image_1920" type="base64" file="gamification/static/img/rank_student_badge.svg"/>
    </record>

    <record id="rank_bachelor" model="gamification.karma.rank">
        <field name="name">Bachelor</field>
        <field name="description" type="html"><p>You love learning things. Curiosity is a good way to progress.</p></field>
        <field name="description_motivational" type="html">
            <div class="d-flex align-items-center">
                <div class="flex-grow-1">Reach the next rank to improve your status!</div>
                <img class="ms-3 img img-fluid" style="max-height: 72px;" src="/gamification/static/img/rank_bachelor_badge.svg" alt=""/>
            </div>
        </field>
        <field name="karma_min">500</field>
        <field name="image_1920" type="base64" file="gamification/static/img/rank_bachelor_badge.svg"/>
    </record>

    <record id="rank_master" model="gamification.karma.rank">
        <field name="name">Master</field>
        <field name="description" type="html"><p>You know what you are talking about. People learn from you.</p></field>
        <field name="description_motivational" type="html">
            <div class="d-flex align-items-center">
                <div class="flex-grow-1">Reach the next rank and become a Master!</div>
                <img class="ms-3 img img-fluid" style="max-height: 72px;" src="/gamification/static/img/rank_master_badge.svg" alt=""/>
            </div>
        </field>
        <field name="karma_min">2000</field>
        <field name="image_1920" type="base64" file="gamification/static/img/rank_master_badge.svg"/>
    </record>

    <record id="rank_doctor" model="gamification.karma.rank">
        <field name="name">Doctor</field>
        <field name="description" type="html"><p>You have reached the last rank. Congratulations!</p></field>
        <field name="description_motivational" type="html">
            <div class="d-flex align-items-center">
                <div class="flex-grow-1">Reach the next rank and become a powerful user!</div>
                <img class="ms-3 img img-fluid" style="max-height: 72px;" src="/gamification/static/img/rank_doctor_badge.svg" alt=""/>
            </div>
        </field>
        <field name="karma_min">10000</field>
        <field name="image_1920" type="base64" file="gamification/static/img/rank_doctor_badge.svg"/>
    </record>
</data></odoo>

```

## File: data\gamification_karma_rank_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!--Ranks-->
    <!-- note that original motivational messages are duplicated+hidden to ensure they are included in .pot export when demo data installed -->
    <record id="rank_student" model="gamification.karma.rank">
        <field name="description_motivational" type="html">
            <div hidden="true">Reach the next rank to show the rest of the world you exist.</div>
            <div class="d-flex align-items-center">
                <div class="flex-grow-1">Reach the next rank and gain a very nice mug!</div>
                <img class="ms-3 img img-fluid" style="max-height: 72px;" src="/gamification/static/img/rank_misc_mug.png" alt=""/>
            </div>
        </field>
    </record>

    <record id="rank_bachelor" model="gamification.karma.rank">
        <field name="description_motivational" type="html">
            <div hidden="true">Reach the next rank to improve your status!</div>
            <div class="d-flex align-items-center">
                <div class="flex-grow-1">Reach the next rank and gain a very magic wand!</div>
                <img class="ms-3 img img-fluid" style="max-height: 72px;" src="/gamification/static/img/rank_misc_wand.png" alt=""/>
            </div>
        </field>
    </record>

    <record id="rank_master" model="gamification.karma.rank">
        <field name="description_motivational" type="html">
            <div hidden="true">Reach the next rank and become a Master!</div>
            <div class="d-flex align-items-center">
                <div class="flex-grow-1">Reach the next rank and gain a very nice hat!</div>
                <img class="ms-3 img img-fluid" style="max-height: 72px;" src="/gamification/static/img/rank_misc_hat.png" alt=""/>
            </div>
        </field>
    </record>

    <record id="rank_doctor" model="gamification.karma.rank">
        <field name="description_motivational" type="html">
            <div hidden="true">Reach the next rank and become a powerful user!</div>
            <div class="d-flex align-items-center">
                <div class="flex-grow-1">Reach the next rank and gain a very nice unicorn!</div>
                <img class="ms-3 img img-fluid" style="max-height: 72px;" src="/gamification/static/img/rank_misc_unicorn.png" alt=""/>
            </div>
        </field>
    </record>
</data></odoo>

```

## File: data\gamification_karma_tracking_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <!--base.user_demo-->
    <record id="karma_tracking_user_demo_1st_day_last_month" model="gamification.karma.tracking">
        <field name="user_id" ref="base.user_demo"/>
        <field name="old_value">0</field>
        <field name="new_value">1000</field>
        <field name="tracking_date" eval="(DateTime.now() - relativedelta(day=1, months=1)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="karma_tracking_user_demo_2nd_day_last_month" model="gamification.karma.tracking">
        <field name="user_id" ref="base.user_demo"/>
        <field name="old_value">1000</field>
        <field name="new_value">1500</field>
        <field name="tracking_date" eval="(DateTime.now() - relativedelta(day=2, months=1)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="karma_tracking_user_demo_5th_day_last_month" model="gamification.karma.tracking">
        <field name="user_id" ref="base.user_demo"/>
        <field name="old_value">1500</field>
        <field name="new_value">2000</field>
        <field name="tracking_date" eval="(DateTime.now() - relativedelta(day=5, months=1)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="karma_tracking_user_demo_20th_day_last_month" model="gamification.karma.tracking">
        <field name="user_id" ref="base.user_demo"/>
        <field name="old_value">2000</field>
        <field name="new_value">2050</field>
        <field name="tracking_date" eval="(DateTime.now() - relativedelta(day=20, months=1)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="karma_tracking_user_demo_today" model="gamification.karma.tracking">
        <field name="user_id" ref="base.user_demo"/>
        <field name="old_value">2050</field>
        <field name="new_value">2500</field>
        <field name="tracking_date" eval="(DateTime.now()).strftime('%Y-%m-%d')"/>
    </record>
    <function model="gamification.karma.tracking" name="unlink">
        <value model="gamification.karma.tracking" eval="obj().search([
                ('user_id', '=', ref('base.user_demo')),
                ('old_value', '=', 0),
                ('new_value', '=', 2500)
            ]).id"/>
    </function>

    <!--base.demo_user0 -->
    <record id="karma_tracking_user_portal_2nd_day_last_month" model="gamification.karma.tracking">
        <field name="user_id" ref="base.demo_user0"/>
        <field name="old_value">0</field>
        <field name="new_value">5</field>
        <field name="tracking_date" eval="(DateTime.now() - relativedelta(day=2, months=1)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="karma_tracking_user_portal_3rd_day_last_month" model="gamification.karma.tracking">
        <field name="user_id" ref="base.demo_user0"/>
        <field name="old_value">5</field>
        <field name="new_value">10</field>
        <field name="tracking_date" eval="(DateTime.now() - relativedelta(day=3, months=1)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="karma_tracking_user_portal_10th_day_last_month" model="gamification.karma.tracking">
        <field name="user_id" ref="base.demo_user0"/>
        <field name="old_value">10</field>
        <field name="new_value">20</field>
        <field name="tracking_date" eval="(DateTime.now() - relativedelta(day=10, months=1)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="karma_tracking_user_portal_yesterday" model="gamification.karma.tracking">
        <field name="user_id" ref="base.demo_user0"/>
        <field name="old_value">20</field>
        <field name="new_value">25</field>
        <field name="tracking_date" eval="(DateTime.now() - relativedelta(day=1)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="karma_tracking_user_portal_today" model="gamification.karma.tracking">
        <field name="user_id" ref="base.demo_user0"/>
        <field name="old_value">25</field>
        <field name="new_value">30</field>
        <field name="tracking_date" eval="DateTime.now()"/>
    </record>
    <function model="gamification.karma.tracking" name="unlink">
        <value model="gamification.karma.tracking" eval="obj().search([
                ('user_id', '=', ref('base.demo_user0')),
                ('old_value', '=', 0),
                ('new_value', '=', 30)
            ]).id"/>
    </function>

    <!--base.user_admin (already have a tracking to 2500)-->
    <record id="karma_tracking_user_admin_1st_day_last_month" model="gamification.karma.tracking">
        <field name="user_id" ref="base.user_admin"/>
        <field name="old_value">0</field>
        <field name="new_value">2000</field>
        <field name="tracking_date" eval="(DateTime.now() - relativedelta(day=1, months=1)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="karma_tracking_user_admin_5th_day_last_month" model="gamification.karma.tracking">
        <field name="user_id" ref="base.user_admin"/>
        <field name="old_value">2000</field>
        <field name="new_value">2250</field>
        <field name="tracking_date" eval="(DateTime.now() - relativedelta(day=5, months=1)).strftime('%Y-%m-%d')"/>
    </record>
    <record id="karma_tracking_user_admin_today" model="gamification.karma.tracking">
        <field name="user_id" ref="base.user_admin"/>
        <field name="old_value">2250</field>
        <field name="new_value">2500</field>
        <field name="tracking_date" eval="(DateTime.now()).strftime('%Y-%m-%d')"/>
    </record>
    <function model="gamification.karma.tracking" name="unlink">
        <value model="gamification.karma.tracking" eval="obj().search([
                ('user_id', '=', ref('base.user_admin')),
                ('old_value', '=', 0),
                ('new_value', '=', 2500)
            ]).id"/>
    </function>

</data>
</odoo>

```

## File: data\ir_cron_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record forcecreate="True" id="ir_cron_check_challenge" model="ir.cron">
            <field name="name">Gamification: Goal Challenge Check</field>
            <field name="model_id" ref="model_gamification_challenge"/>
            <field name="state">code</field>
            <field name="code">model._cron_update()</field>
            <field name="interval_number">1</field>
            <field name="interval_type">days</field>
        </record>

        <record id="ir_cron_consolidate" model="ir.cron">
            <field name="name">Gamification: Karma tracking consolidation</field>
            <field name="model_id" ref="model_gamification_karma_tracking"/>
            <field name="state">code</field>
            <field name="code">model._consolidate_cron()</field>
            <field name="active" eval="True"/>
            <field name="interval_number">1</field>
            <field name="interval_type">months</field>
            <field name="nextcall" eval="(DateTime.now() + relativedelta(day=1, months=1)).strftime('%Y-%m-%d 04:00:00')" />
        </record>
    </data>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="email_template_badge_received" model="mail.template">
            <field name="name">Gamification: Badge Received</field>
            <field name="subject">New badge {{ object.badge_id.name }} granted</field>
            <field name="model_id" ref="gamification.model_gamification_badge_user"/>
            <field name="partner_to">{{ object.user_id.partner_id.id }}</field>
            <field name="description">Sent automatically to the user who received a badge</field>
            <field name="body_html" type="html">
<table border="0" cellpadding="0" style="padding-top: 16px; background-color: #F1F1F1; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" width="590" cellpadding="0" style="padding: 16px; background-color: white; color: #454748; border-collapse:separate;" summary="o_mail_notification">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table width="590" border="0" cellpadding="0" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Your Badge</span><br/>
                    <span style="font-size: 20px; font-weight: bold;" t-out="object.badge_id.name or ''"></span>
                </td><td valign="middle" align="right" t-if="not object.user_id.company_id.uses_default_logo">
                    <img t-attf-src="/logo.png?company={{ object.user_id.company_id.id }}" style="padding: 0px; margin: 0px; height: auto; width: 80px;" t-att-alt="object.user_id.company_id.name"/>
                </td></tr>
                <tr><td colspan="2" style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- CONTENT -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table width="590" border="0" cellpadding="0" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="top" style="font-size: 14px;">
                    <div>
                        Congratulations <t t-out="object.user_id.name or ''"></t>!<br/>
                        You just received badge <strong t-out="object.badge_id.name or ''"></strong>!<br/>
                        <table t-if="not is_html_empty(object.badge_id.description)" cellspacing="0" cellpadding="0" border="0" style="width: 560px; margin-top: 5px;">
                            <tbody><tr>
                                <td valign="center">
                                    <img t-attf-src="/web/image/gamification.badge/{{ object.badge_id.id }}/image_128/80x80" style="padding: 0px; margin: 0px; height: auto; width: 80px;" t-att-alt="user.company_id.name"/>
                                </td>
                                <td valign="center">
                                    <cite t-out="object.badge_id.description or ''"></cite>
                                </td>
                            </tr></tbody>
                        </table>
                        <br/>
                        <t t-if="object.sender_id">
                            This badge was granted by <strong t-out="object.sender_id.name or ''"></strong>.
                        </t>
                        <br/>
                        <t t-if="object.comment" t-out="object.comment or ''"></t>
                        <br/><br/>
                        Thank you,
                        <t t-if="object.sender_id.signature">
                            <br />
                            <t t-out="object.sender_id.signature or ''"></t>
                        </t>
                    </div>
                </td></tr>
                <tr><td style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- FOOTER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table width="590" border="0" cellpadding="0" style="min-width: 590px; background-color: white; font-size: 12px; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle" align="left">
                    <t t-out="object.user_id.company_id.name or ''">YourCompany</t>
                </td></tr>
                <tr><td valign="middle" align="left" style="opacity: 0.7;">
                    <t t-out="object.user_id.company_id.phone or ''">+1 650-123-4567</t>
                    <t t-if="object.user_id.company_id.email">
                        | <a t-attf-href="'mailto:%s' % {{ object.user_id.company_id.email }}" style="text-decoration:none; color: #454748;" t-out="object.user_id.company_id.email or ''">info@yourcompany.com</a>
                    </t>
                    <t t-if="object.user_id.company_id.website">
                        | <a t-attf-href="'%s' % {{ object.user_id.company_id.website }}" style="text-decoration:none; color: #454748;" t-out="object.user_id.company_id.website or ''">http://www.example.com</a>
                    </t>
                </td></tr>
            </table>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- POWERED BY -->
<tr><td align="center" style="min-width: 590px;">
    <table width="590" border="0" cellpadding="0" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
      <tr><td style="text-align: center; font-size: 14px;">
        Powered by <a target="_blank" href="https://www.odoo.com?utm_source=db&amp;utm_medium=gamification" style="color: #875A7B;">Odoo</a>
      </td></tr>
    </table>
</td></tr>
</table></field>
            <field name="lang">{{ object.user_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <record id="email_template_goal_reminder" model="mail.template">
            <field name="name">Gamification: Reminder For Goal Update</field>
            <field name="model_id" ref="gamification.model_gamification_goal"/>
            <field name="partner_to">{{ object.user_id.partner_id.id }}</field>
            <field name="description">Sent automatically to participant who haven't updated their goal</field>
            <field name="body_html" type="html">
<div>
    <strong>Reminder</strong><br/>
    You have not updated your progress for the goal <t t-out="object.definition_id.name or ''"></t> (currently reached at <t t-out="object.completeness or ''"></t>%) for at least <t t-out="object.remind_update_delay or ''"></t> days. Do not forget to do it.
    <br/><br/>
    Thank you,
    <t t-if="object.challenge_id.manager_id.signature">
        <br />
        <t t-out="object.challenge_id.manager_id.signature or ''"></t>
    </t>
</div></field>
            <field name="lang">{{ object.user_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>
        
        <record id="simple_report_template" model="mail.template">
            <field name="name">Gamification: Challenge Report</field>
            <field name="model_id" ref="gamification.model_gamification_challenge"/>
            <field name="description">Send a challenge report to all participants</field>
            <field name="body_html" type="html">
<table cellspacing="0" cellpadding="0" width="100%" style="background-color: #EEE; border-collapse: collapse;">
<tr>
    <td valign="top" align="center">
        <t t-set="object_ctx" t-value="ctx.get('object')"/>
        <t t-set="company" t-value="object_ctx and object_ctx.company_id or user.company_id"/>
        <t t-set="challenge_lines" t-value="ctx.get('challenge_lines', [])"/>
        <table cellspacing="0" cellpadding="0" width="600" style="margin: 0 auto; width: 570px;">
            <tr><td>
                <table cellspacing="0" cellpadding="0" width="100%">
                    <tr>
                        <div>
                            <t t-if="object.visibility_mode == 'ranking'">
                                <td style="padding:15px;">
                                    <p style="font-size:20px;color:#666666;" align="center">Leaderboard</p>
                                </td>
                            </t>
                        </div>
                    </tr>
                </table>
                <table cellspacing="0" cellpadding="0" width="100%" bgcolor="#fff" style="background-color:#fff;">
                    <tr><td style="padding: 15px;">
                        <t t-if="object.visibility_mode == 'personal'">
                            <span style="color:#666666;font-size:13px;">Here is your current progress in the challenge <strong t-out="object.name or ''"></strong>.</span>
                            <table cellspacing="0" cellpadding="0" width="100%" style="margin-top:20px;">
                                <tr>
                                    <td align="center">
                                        <div>Personal Performance</div>
                                    </td>
                                </tr>
                            </table>
                            <table cellspacing="0" cellpadding="0" width="100%" style="margin-top:30px;color:#666666;">
                                <thead>
                                    <tr style="color:#9A6C8E; font-size:12px;">
                                        <th align="left" style="padding-bottom: 0px;width:40%;text-align:left;">Goals</th>
                                        <th style="width:20%;text-align:right;" align="left">Target</th>
                                        <th style="width:20%;text-align:right;" align="right">Current</th>
                                        <th style="width:20%;text-align:right;" align="right">Completeness</th>
                                    </tr>
                                    <tr>
                                        <td colspan="5" style="height:1px;background-color:#9A6C8E;"></td>
                                    </tr>
                                </thead>
                                <tbody t-foreach="challenge_lines" t-as="line">
                                    <tr style="font-weight:bold;">
                                        <td style="padding: 20px 0;" align="left">
                                            <t t-out="line['name'] or ''"></t>
                                            <t t-if="line['suffix'] or line['monetary']">
                                                (<t t-out="line['full_suffix'] or ''"></t>)
                                            </t>
                                        </td>
                                        <td style="padding: 20px 0;" align="right"><t t-out="&quot;%.2f&quot; % line['target'] or ''"></t>
                                            <t t-if="line['suffix']" t-out="line['suffix'] or ''"></t>
                                        </td>
                                        <td style="padding: 20px 0;" align="right"><t t-out="&quot;%.2f&quot; % line['current'] or ''"></t>
                                            <t t-if="line['suffix']" t-out="line['suffix'] or ''"></t>
                                        </td>
                                        <td style="padding: 20px 0;font-size:25px;color:#9A6C8E;" align="right"><strong><t t-out="int(line['completeness']) or ''"></t>%</strong></td>
                                    </tr>
                                    <tr>
                                        <td colspan="5" style="height:1px;background-color:#e3e3e3;"></td>
                                    </tr>
                                </tbody>
                            </table>                   
                        </t>
                        <t t-else="">
                            <span style="color:#A8A8A8;font-size:13px;">
                                Challenge: <strong t-out="object.name or ''"></strong>.
                            </span> 
                            <t t-foreach="challenge_lines" t-as="line">
                                <!-- Header + Button table -->
                                <table cellspacing="0" cellpadding="0" width="100%" style="margin-top:35px;">
                                    <tr>
                                        <td width="50%">
                                            <div>Top Achievers for goal <strong t-out="line['name'] or ''"></strong></div>
                                        </td>
                                    </tr>
                                </table>
                                <!-- Podium -->
                                <t t-if="len(line['goals']) == 2">
                                    <table cellspacing="0" cellpadding="0" width="100%" style="margin-top:10px;">
                                        <tr><td style="padding:0 30px;">
                                            <table cellspacing="0" cellpadding="0" width="100%" style="table-layout: fixed;">
                                                <tr>
                                                    <t t-set="top_goals" t-value="line['goals'][:3]"/>
                                                    <t t-foreach="top_goals" t-as="goal">
                                                        <td align="center" style="width:32%;">
                                                            <t t-if="loop.index == 1">
                                                                <t t-set="extra_div" t-value="'&lt;div style=&quot;height:40px;&quot;&gt;&lt;/div&gt;'"/>
                                                                <t t-set="heightA" t-value="95"/>
                                                                <t t-set="heightB" t-value="75"/>
                                                                <t t-set="bgColor" t-value="'#b898b0'"/>
                                                                <t t-set="fontSize" t-value="50"/>
                                                                <t t-set="podiumPosition" t-value="'2'"/>
                                                            </t>
                                                            <t t-elif="loop.index == 2">
                                                                <t t-set="extra_div" t-value="''"/>
                                                                <t t-set="heightA" t-value="55"/>
                                                                <t t-set="heightB" t-value="115"/>
                                                                <t t-set="bgColor" t-value="'#9A6C8E'"/>
                                                                <t t-set="fontSize" t-value="85"/>
                                                                <t t-set="podiumPosition" t-value="'1'"/>
                                                            </t>
                                                            <t t-elif="loop.index == 3">
                                                                <t t-set="extra_div" t-value="'&lt;div style=&quot;height:60px;&quot;&gt;&lt;/div&gt;'"/>
                                                                <t t-set="heightA" t-value="115"/>
                                                                <t t-set="heightB" t-value="55"/>
                                                                <t t-set="bgColor" t-value="'#c8afc1'"/>
                                                                <t t-set="fontSize" t-value="35"/>
                                                                <t t-set="podiumPosition" t-value="'3'"/>
                                                            </t>
                                                            <div style="margin:0 3px 0 3px;height:220px;">
                                                                <div t-attf-style="height:{{ heightA }}px;">
                                                                    <t t-out="extra_div or ''"></t>
                                                                    <div style="height:55px;">
                                                                        <img style="margin-bottom:5px;width:50px;height:50px;border-radius:50%;object-fit:cover;" t-att-src="image_data_uri(object.env['res.users'].browse(goal['user_id']).partner_id.image_128)" t-att-alt="goal['name']"/>
                                                                    </div>
                                                                    <div align="center" t-attf-style ="color:{{ bgColor }};height:20px">
                                                                        <t t-out="goal['name'] or ''"></t>
                                                                    </div>
                                                                </div>
                                                                <div t-attf-style="background-color:{{ bgColor }};height:{{ heightB }}px;">
                                                                    <strong><span t-attf-style="color:#fff;font-size:{{ fontSize }}px;" t-out="podiumPosition or ''"></span></strong>
                                                                </div>
                                                                <div style="height:30px;">
                                                                    <t t-out="&quot;%.2f&quot; % goal['current'] or ''"></t>
                                                                    <t t-if="line['suffix'] or line['monetary']">
                                                                        <t t-out="line['full_suffix'] or ''"></t>
                                                                    </t>
                                                                </div>
                                                            </div>
                                                        </td>
                                                    </t>
                                                </tr>
                                            </table>
                                            </td>
                                        </tr>
                                    </table>
                                </t>
                                <!-- data table -->
                                <table cellspacing="0" cellpadding="0" width="100%" style="margin-bottom:5px">
                                    <tr>
                                        <td>
                                            <table cellspacing="0" cellpadding="0" width="100%" style="margin-top:30px;margin-bottom:5px;color:#666666;">
                                                <thead>
                                                    <tr style="color:#9A6C8E; font-size:12px;">
                                                        <th style="width:15%;text-align:center;">Rank</th>
                                                        <th style="width:25%;text-align:left;">Name</th>
                                                        <th style="width:30%;text-align:right;">Performance 
                                                            <t t-if="line['suffix']">
                                                                (<t t-out="line['suffix'] or ''"></t>)
                                                            </t>
                                                            <t t-elif="line['monetary']">
                                                                (<t t-out="company.currency_id.symbol or ''"></t>)
                                                            </t>
                                                        </th>
                                                        <th style="width:30%;text-align:right;">Completeness</th>
                                                    </tr>
                                                    <tr>
                                                        <td colspan="5" style="height:1px;background-color:#9A6C8E;"></td>
                                                    </tr>
                                                </thead>
                                                <tbody t-foreach="line['goals']" t-as="goal">
                                                    <tr>
                                                        <t t-set="tdBgColor" t-value="'#fff'"/>
                                                        <t t-set="tdColor" t-value="'gray'"/>
                                                        <t t-set="mutedColor" t-value="'#AAAAAA'"/>
                                                        <t t-set="tdPercentageColor" t-value="'#9A6C8E'"/>
                                                        <td width="15%" align="center" valign="middle" t-attf-style="background-color:{{ tdBgColor }};padding :5px 0;font-size:20px;"><t t-out="goal['rank']+1 or ''"></t>
                                                        </td>
                                                        <td width="25%" align="left" valign="middle" t-attf-style="background-color:{{ tdBgColor }};padding :5px 0;font-size:13px;"><t t-out="goal['name'] or ''"></t></td>
                                                        <td width="30%" align="right" t-attf-style="background-color:{{ tdBgColor }};padding:5px 0;line-height:1;"><t t-out="&quot;%.2f&quot; % goal['current'] or ''"></t><br/><span t-attf-style="font-size:13px;color:{{ mutedColor }};">on <t t-out="&quot;%.2f&quot; % line['target'] or ''"></t></span>
                                                        </td>
                                                        <td width="30%" t-attf-style="color:{{ tdPercentageColor }};background-color:{{ tdBgColor }};padding-right:15px;font-size:22px;" align="right"><strong><t t-out="int(goal['completeness']) or ''"></t>%</strong></td>
                                                    </tr>
                                                    <tr>
                                                        <td colspan="5" style="height:1px;background-color:#DADADA;"></td>
                                                    </tr>
                                                </tbody>
                                            </table>
                                        </td>
                                    </tr>
                                </table> 
                            </t>
                        </t>
                    </td></tr>
                </table>
            </td></tr>
        </table>
    </td>
</tr>
</table>
            </field>
        </record>

        <record id="mail_template_data_new_rank_reached" model="mail.template">
            <field name="name">Gamification: New Rank Reached</field>
            <field name="model_id" ref="base.model_res_users"/>
            <field name="subject">New rank: {{ object.rank_id.name }}</field>
            <field name="email_to"></field>
            <field name="partner_to">{{ object.partner_id.id }}</field>
            <field name="description">Sent automatically when user reaches a new rank</field>
            <field name="body_html" type="html">
<div style="color:#515166;padding:10px 0px;font-family:Arial,Helvetica,sans-serif;font-size:14px;">
<table style="width:600px;margin:0px auto;background:white;border:1px solid #e1e1e1;">
    <tbody>
        <tr>
            <td style="padding:15px 20px 10px 20px;">
                <p>
                    Congratulations
                    <span t-out="object.name or ''">Joel Willis</span>!
                </p>
                <p>
                    You just reached a new rank: <strong t-out="object.rank_id.name or ''">Newbie</strong>
                </p>
                <t t-if="object.next_rank_id.name">
                    <p>Continue your work to become a <strong t-out="object.next_rank_id.name or ''">Student</strong>!</p>
                </t>
                <div style="margin: 16px 0px 16px 0px;">
                    <t t-set="gamification_redirection_data" t-value="object.get_gamification_redirection_data()"/>
                    <t t-foreach="gamification_redirection_data" t-as="data">
                        <t t-set="url" t-value="data['url']"/>
                        <t t-set="label" t-value="data['label']"/>
                        <a t-att-href="url" style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;" t-out="label or ''">LABEL</a>
                    </t>
                </div>
            </td>
            <td style="padding:15px 20px 10px 20px;">
                <p style="text-align: center;">
                    <img t-attf-src="/web/image/gamification.karma.rank/{{ object.rank_id.id }}/image_128"/>
                </p>
            </td>
        </tr>
        <tr>
            <td t-if="not user.share and user.active" style="padding:15px 20px 10px 20px;">
                <t t-out="user.signature or ''">--<br/>Mitchell Admin</t>
            </td>
            <td t-else="" style="padding:15px 20px 10px 20px;">
                <t t-out="user.company_id.name or ''"/>
            </td>
        </tr>
    </tbody>
 </table>
</div></field>
            <field name="lang">{{ object.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

    </data>
</odoo>

```

## File: models\gamification_badge.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from datetime import date

from odoo import api, fields, models, _, exceptions
from odoo.tools import SQL


_logger = logging.getLogger(__name__)


class GamificationBadge(models.Model):
    """Badge object that users can send and receive"""

    CAN_GRANT = 1
    NOBODY_CAN_GRANT = 2
    USER_NOT_VIP = 3
    BADGE_REQUIRED = 4
    TOO_MANY = 5

    _name = 'gamification.badge'
    _description = 'Gamification Badge'
    _inherit = ['mail.thread', 'image.mixin']

    name = fields.Char('Badge', required=True, translate=True)
    active = fields.Boolean('Active', default=True)
    description = fields.Html('Description', translate=True, sanitize_attributes=False)
    level = fields.Selection([
        ('bronze', 'Bronze'), ('silver', 'Silver'), ('gold', 'Gold')],
        string='Forum Badge Level', default='bronze')

    rule_auth = fields.Selection([
            ('everyone', 'Everyone'),
            ('users', 'A selected list of users'),
            ('having', 'People having some badges'),
            ('nobody', 'No one, assigned through challenges'),
        ], default='everyone',
        string="Allowance to Grant", help="Who can grant this badge", required=True)
    rule_auth_user_ids = fields.Many2many(
        'res.users', 'rel_badge_auth_users',
        string='Authorized Users',
        help="Only these people can give this badge")
    rule_auth_badge_ids = fields.Many2many(
        'gamification.badge', 'gamification_badge_rule_badge_rel', 'badge1_id', 'badge2_id',
        string='Required Badges',
        help="Only the people having these badges can give this badge")

    rule_max = fields.Boolean('Monthly Limited Sending', help="Check to set a monthly limit per person of sending this badge")
    rule_max_number = fields.Integer('Limitation Number', help="The maximum number of time this badge can be sent per month per person.")
    challenge_ids = fields.One2many('gamification.challenge', 'reward_id', string="Reward of Challenges")

    goal_definition_ids = fields.Many2many(
        'gamification.goal.definition', 'badge_unlocked_definition_rel',
        string='Rewarded by', help="The users that have succeeded these goals will receive automatically the badge.")

    owner_ids = fields.One2many(
        'gamification.badge.user', 'badge_id',
        string='Owners', help='The list of instances of this badge granted to users')

    granted_count = fields.Integer("Total", compute='_get_owners_info', help="The number of time this badge has been received.")
    granted_users_count = fields.Integer("Number of users", compute='_get_owners_info', help="The number of time this badge has been received by unique users.")
    unique_owner_ids = fields.Many2many(
        'res.users', string="Unique Owners", compute='_get_owners_info',
        help="The list of unique users having received this badge.")

    stat_this_month = fields.Integer(
        "Monthly total", compute='_get_badge_user_stats',
        help="The number of time this badge has been received this month.")
    stat_my = fields.Integer(
        "My Total", compute='_get_badge_user_stats',
        help="The number of time the current user has received this badge.")
    stat_my_this_month = fields.Integer(
        "My Monthly Total", compute='_get_badge_user_stats',
        help="The number of time the current user has received this badge this month.")
    stat_my_monthly_sending = fields.Integer(
        'My Monthly Sending Total',
        compute='_get_badge_user_stats',
        help="The number of time the current user has sent this badge this month.")

    remaining_sending = fields.Integer(
        "Remaining Sending Allowed", compute='_remaining_sending_calc',
        help="If a maximum is set")

    @api.depends('owner_ids')
    def _get_owners_info(self):
        """Return:
            the list of unique res.users ids having received this badge
            the total number of time this badge was granted
            the total number of users this badge was granted to
        """
        defaults = {
            'granted_count': 0,
            'granted_users_count': 0,
            'unique_owner_ids': [],
        }
        if not self.ids:
            self.update(defaults)
            return

        Users = self.env["res.users"]
        query = Users._where_calc([])
        Users._apply_ir_rules(query)
        badge_alias = query.join("res_users", "id", "gamification_badge_user", "user_id", "badges")

        rows = self.env.execute_query(SQL(
            """
              SELECT %(badge_alias)s.badge_id, count(res_users.id) as stat_count,
                     count(distinct(res_users.id)) as stat_count_distinct,
                     array_agg(distinct(res_users.id)) as unique_owner_ids
                FROM %(from_clause)s
               WHERE %(where_clause)s
                 AND %(badge_alias)s.badge_id IN %(ids)s
            GROUP BY %(badge_alias)s.badge_id
            """,
            from_clause=query.from_clause,
            where_clause=query.where_clause or SQL("TRUE"),
            badge_alias=SQL.identifier(badge_alias),
            ids=tuple(self.ids),
        ))

        mapping = {
            badge_id: {
                'granted_count': count,
                'granted_users_count': distinct_count,
                'unique_owner_ids': owner_ids,
            }
            for (badge_id, count, distinct_count, owner_ids) in rows
        }
        for badge in self:
            badge.update(mapping.get(badge.id, defaults))

    @api.depends('owner_ids.badge_id', 'owner_ids.create_date', 'owner_ids.user_id')
    def _get_badge_user_stats(self):
        """Return stats related to badge users"""
        first_month_day = date.today().replace(day=1)

        for badge in self:
            owners = badge.owner_ids
            badge.stat_my = sum(o.user_id == self.env.user for o in owners)
            badge.stat_this_month = sum(o.create_date.date() >= first_month_day for o in owners)
            badge.stat_my_this_month = sum(
                o.user_id == self.env.user and o.create_date.date() >= first_month_day
                for o in owners
            )
            badge.stat_my_monthly_sending = sum(
                o.create_uid == self.env.user and o.create_date.date() >= first_month_day
                for o in owners
            )

    @api.depends(
        'rule_auth',
        'rule_auth_user_ids',
        'rule_auth_badge_ids',
        'rule_max',
        'rule_max_number',
        'stat_my_monthly_sending',
    )
    def _remaining_sending_calc(self):
        """Computes the number of badges remaining the user can send

        0 if not allowed or no remaining
        integer if limited sending
        -1 if infinite (should not be displayed)
        """
        for badge in self:
            if badge._can_grant_badge() != self.CAN_GRANT:
                # if the user cannot grant this badge at all, result is 0
                badge.remaining_sending = 0
            elif not badge.rule_max:
                # if there is no limitation, -1 is returned which means 'infinite'
                badge.remaining_sending = -1
            else:
                badge.remaining_sending = badge.rule_max_number - badge.stat_my_monthly_sending

    def check_granting(self):
        """Check the user 'uid' can grant the badge 'badge_id' and raise the appropriate exception
        if not

        Do not check for SUPERUSER_ID
        """
        status_code = self._can_grant_badge()
        if status_code == self.CAN_GRANT:
            return True
        elif status_code == self.NOBODY_CAN_GRANT:
            raise exceptions.UserError(_('This badge can not be sent by users.'))
        elif status_code == self.USER_NOT_VIP:
            raise exceptions.UserError(_('You are not in the user allowed list.'))
        elif status_code == self.BADGE_REQUIRED:
            raise exceptions.UserError(_('You do not have the required badges.'))
        elif status_code == self.TOO_MANY:
            raise exceptions.UserError(_('You have already sent this badge too many time this month.'))
        else:
            _logger.error("Unknown badge status code: %s" % status_code)
        return False

    def _can_grant_badge(self):
        """Check if a user can grant a badge to another user

        :param uid: the id of the res.users trying to send the badge
        :param badge_id: the granted badge id
        :return: integer representing the permission.
        """
        if self.env.is_admin():
            return self.CAN_GRANT

        if self.rule_auth == 'nobody':
            return self.NOBODY_CAN_GRANT
        elif self.rule_auth == 'users' and self.env.user not in self.rule_auth_user_ids:
            return self.USER_NOT_VIP
        elif self.rule_auth == 'having':
            all_user_badges = self.env['gamification.badge.user'].search([('user_id', '=', self.env.uid)]).mapped('badge_id')
            if self.rule_auth_badge_ids - all_user_badges:
                return self.BADGE_REQUIRED

        if self.rule_max and self.stat_my_monthly_sending >= self.rule_max_number:
            return self.TOO_MANY

        # badge.rule_auth == 'everyone' -> no check
        return self.CAN_GRANT

```

## File: models\gamification_badge_user.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class BadgeUser(models.Model):
    """User having received a badge"""

    _name = 'gamification.badge.user'
    _description = 'Gamification User Badge'
    _order = "create_date desc"
    _rec_name = "badge_name"

    user_id = fields.Many2one('res.users', string="User", required=True, ondelete="cascade", index=True)
    sender_id = fields.Many2one('res.users', string="Sender")
    badge_id = fields.Many2one('gamification.badge', string='Badge', required=True, ondelete="cascade", index=True)
    challenge_id = fields.Many2one('gamification.challenge', string='Challenge')
    comment = fields.Text('Comment')
    badge_name = fields.Char(related='badge_id.name', string="Badge Name", readonly=False)
    level = fields.Selection(
        string='Badge Level', related="badge_id.level", store=True, readonly=True)

    def _send_badge(self):
        """Send a notification to a user for receiving a badge

        Does not verify constrains on badge granting.
        The users are added to the owner_ids (create badge_user if needed)
        The stats counters are incremented
        :param ids: list(int) of badge users that will receive the badge
        """
        template = self.env.ref(
            'gamification.email_template_badge_received',
            raise_if_not_found=False
        )
        if not template:
            return

        for badge_user in self:
            template.send_mail(
                badge_user.id,
            )

        return True

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            self.env['gamification.badge'].browse(vals['badge_id']).check_granting()
        return super().create(vals_list)

```

## File: models\gamification_challenge.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import ast
import itertools
import logging
from datetime import date, timedelta

from dateutil.relativedelta import relativedelta, MO
from markupsafe import Markup

from odoo import _, api, exceptions, fields, models
from odoo.http import SESSION_LIFETIME

_logger = logging.getLogger(__name__)

# display top 3 in ranking, could be db variable
MAX_VISIBILITY_RANKING = 3

def start_end_date_for_period(period, default_start_date=False, default_end_date=False):
    """Return the start and end date for a goal period based on today

    :param str default_start_date: string date in DEFAULT_SERVER_DATE_FORMAT format
    :param str default_end_date: string date in DEFAULT_SERVER_DATE_FORMAT format

    :return: (start_date, end_date), dates in string format, False if the period is
    not defined or unknown"""
    today = date.today()
    if period == 'daily':
        start_date = today
        end_date = start_date
    elif period == 'weekly':
        start_date = today + relativedelta(weekday=MO(-1))
        end_date = start_date + timedelta(days=7)
    elif period == 'monthly':
        start_date = today.replace(day=1)
        end_date = today + relativedelta(months=1, day=1, days=-1)
    elif period == 'yearly':
        start_date = today.replace(month=1, day=1)
        end_date = today.replace(month=12, day=31)
    else:  # period == 'once':
        start_date = default_start_date  # for manual goal, start each time
        end_date = default_end_date

        return (start_date, end_date)

    return fields.Datetime.to_string(start_date), fields.Datetime.to_string(end_date)

class Challenge(models.Model):
    """Gamification challenge

    Set of predifined objectives assigned to people with rules for recurrence and
    rewards

    If 'user_ids' is defined and 'period' is different than 'one', the set will
    be assigned to the users for each period (eg: every 1st of each month if
    'monthly' is selected)
    """

    _name = 'gamification.challenge'
    _description = 'Gamification Challenge'
    _inherit = 'mail.thread'
    _order = 'end_date, start_date, name, id'

    @api.model
    def default_get(self, fields_list):
        res = super().default_get(fields_list)
        if 'user_domain' in fields_list and 'user_domain' not in res:
            user_group_id = self.env.ref('base.group_user')
            res['user_domain'] = f'["&", ("groups_id", "=", "{user_group_id.name}"), ("active", "=", True)]'
        return res

    # description
    name = fields.Char("Challenge Name", required=True, translate=True)
    description = fields.Text("Description", translate=True)
    state = fields.Selection([
            ('draft', "Draft"),
            ('inprogress', "In Progress"),
            ('done', "Done"),
        ], default='draft', copy=False,
        string="State", required=True, tracking=True)
    manager_id = fields.Many2one(
        'res.users', default=lambda self: self.env.uid,
        string="Responsible")
    # members
    user_ids = fields.Many2many('res.users', 'gamification_challenge_users_rel', string="Participants")
    user_domain = fields.Char("User domain")        # Alternative to a list of users
    user_count = fields.Integer('# Users', compute='_compute_user_count')
    # periodicity
    period = fields.Selection([
            ('once', "Non recurring"),
            ('daily', "Daily"),
            ('weekly', "Weekly"),
            ('monthly', "Monthly"),
            ('yearly', "Yearly")
        ], default='once',
        string="Periodicity",
        help="Period of automatic goal assignment. If none is selected, should be launched manually.",
        required=True)
    start_date = fields.Date("Start Date", help="The day a new challenge will be automatically started. If no periodicity is set, will use this date as the goal start date.")
    end_date = fields.Date("End Date", help="The day a new challenge will be automatically closed. If no periodicity is set, will use this date as the goal end date.")

    invited_user_ids = fields.Many2many('res.users', 'gamification_invited_user_ids_rel', string="Suggest to users")

    line_ids = fields.One2many('gamification.challenge.line', 'challenge_id',
                                  string="Lines",
                                  help="List of goals that will be set",
                                  required=True, copy=True)

    reward_id = fields.Many2one('gamification.badge', string="For Every Succeeding User")
    reward_first_id = fields.Many2one('gamification.badge', string="For 1st user")
    reward_second_id = fields.Many2one('gamification.badge', string="For 2nd user")
    reward_third_id = fields.Many2one('gamification.badge', string="For 3rd user")
    reward_failure = fields.Boolean("Reward Bests if not Succeeded?")
    reward_realtime = fields.Boolean("Reward as soon as every goal is reached", default=True, help="With this option enabled, a user can receive a badge only once. The top 3 badges are still rewarded only at the end of the challenge.")

    visibility_mode = fields.Selection([
            ('personal', "Individual Goals"),
            ('ranking', "Leader Board (Group Ranking)"),
        ], default='personal',
        string="Display Mode", required=True)

    report_message_frequency = fields.Selection([
            ('never', "Never"),
            ('onchange', "On change"),
            ('daily', "Daily"),
            ('weekly', "Weekly"),
            ('monthly', "Monthly"),
            ('yearly', "Yearly")
        ], default='never',
        string="Report Frequency", required=True)
    report_message_group_id = fields.Many2one('discuss.channel', string="Send a copy to", help="Group that will receive a copy of the report in addition to the user")
    report_template_id = fields.Many2one('mail.template', default=lambda self: self._get_report_template(), string="Report Template", required=True)
    remind_update_delay = fields.Integer("Non-updated manual goals will be reminded after", help="Never reminded if no value or zero is specified.")
    last_report_date = fields.Date("Last Report Date", default=fields.Date.today)
    next_report_date = fields.Date("Next Report Date", compute='_get_next_report_date', store=True)

    challenge_category = fields.Selection([
        ('hr', 'Human Resources / Engagement'),
        ('other', 'Settings / Gamification Tools'),
    ], string="Appears in", required=True, default='hr',
       help="Define the visibility of the challenge through menus")

    @api.depends('user_ids')
    def _compute_user_count(self):
        mapped_data = {}
        if self.ids:
            query = """
                SELECT gamification_challenge_id, count(res_users_id)
                  FROM gamification_challenge_users_rel rel
             LEFT JOIN res_users users
                    ON users.id=rel.res_users_id AND users.active = TRUE
                 WHERE gamification_challenge_id IN %s
              GROUP BY gamification_challenge_id
            """
            self.env.cr.execute(query, [tuple(self.ids)])
            mapped_data = dict(
                (challenge_id, user_count)
                for challenge_id, user_count in self.env.cr.fetchall()
            )
        for challenge in self:
            challenge.user_count = mapped_data.get(challenge.id, 0)

    REPORT_OFFSETS = {
        'daily': timedelta(days=1),
        'weekly': timedelta(days=7),
        'monthly': relativedelta(months=1),
        'yearly': relativedelta(years=1),
    }
    @api.depends('last_report_date', 'report_message_frequency')
    def _get_next_report_date(self):
        """ Return the next report date based on the last report date and
        report period.
        """
        for challenge in self:
            last = challenge.last_report_date
            offset = self.REPORT_OFFSETS.get(challenge.report_message_frequency)

            if offset:
                challenge.next_report_date = last + offset
            else:
                challenge.next_report_date = False

    def _get_report_template(self):
        template = self.env.ref('gamification.simple_report_template', raise_if_not_found=False)

        return template.id if template else False

    @api.model_create_multi
    def create(self, vals_list):
        """Overwrite the create method to add the user of groups"""
        for vals in vals_list:
            if user_domain := vals.get('user_domain'):
                users = self._get_challenger_users(str(user_domain))

                if not vals.get('user_ids'):
                    vals['user_ids'] = []
                vals['user_ids'].extend((4, user.id) for user in users)

        return super().create(vals_list)

    def write(self, vals):
        if user_domain := vals.get('user_domain'):
            users = self._get_challenger_users(str(user_domain))

            if not vals.get('user_ids'):
                vals['user_ids'] = []
            vals['user_ids'].extend((4, user.id) for user in users)

        write_res = super().write(vals)

        if vals.get('report_message_frequency', 'never') != 'never':
            # _recompute_challenge_users do not set users for challenges with no reports, subscribing them now
            for challenge in self:
                challenge.message_subscribe([user.partner_id.id for user in challenge.user_ids])

        if vals.get('state') == 'inprogress':
            self._recompute_challenge_users()
            self._generate_goals_from_challenge()

        elif vals.get('state') == 'done':
            self._check_challenge_reward(force=True)

        elif vals.get('state') == 'draft':
            # resetting progress
            if self.env['gamification.goal'].search_count([('challenge_id', 'in', self.ids), ('state', '=', 'inprogress')], limit=1):
                raise exceptions.UserError(_("You can not reset a challenge with unfinished goals."))

        return write_res


    ##### Update #####

    @api.model # FIXME: check how cron functions are called to see if decorator necessary
    def _cron_update(self, ids=False, commit=True):
        """Daily cron check.

        - Start planned challenges (in draft and with start_date = today)
        - Create the missing goals (eg: modified the challenge to add lines)
        - Update every running challenge
        """
        # in cron mode, will do intermediate commits
        # cannot be replaced by a parameter because it is intended to impact side-effects of
        # write operations
        self = self.with_context(commit_gamification=commit)
        # start scheduled challenges
        planned_challenges = self.search([
            ('state', '=', 'draft'),
            ('start_date', '<=', fields.Date.today())
        ])
        if planned_challenges:
            planned_challenges.write({'state': 'inprogress'})

        # close scheduled challenges
        scheduled_challenges = self.search([
            ('state', '=', 'inprogress'),
            ('end_date', '<', fields.Date.today())
        ])
        if scheduled_challenges:
            scheduled_challenges.write({'state': 'done'})

        records = self.browse(ids) if ids else self.search([('state', '=', 'inprogress')])

        return records._update_all()

    def _update_all(self):
        """Update the challenges and related goals."""
        if not self:
            return True

        Goals = self.env['gamification.goal']

        # include yesterday goals to update the goals that just ended
        # exclude goals for users that have not interacted with the
        # webclient since the last update or whose session is no longer
        # valid.
        yesterday = fields.Date.to_string(date.today() - timedelta(days=1))
        self.env.cr.execute("""SELECT gg.id
                        FROM gamification_goal as gg
                        JOIN bus_presence as bp ON bp.user_id = gg.user_id
                       WHERE gg.write_date <= bp.last_presence
                         AND bp.last_presence >= now() AT TIME ZONE 'UTC' - interval '%(session_lifetime)s seconds'
                         AND gg.closed IS NOT TRUE
                         AND gg.challenge_id IN %(challenge_ids)s
                         AND (gg.state = 'inprogress'
                              OR (gg.state = 'reached' AND gg.end_date >= %(yesterday)s))
                      GROUP BY gg.id
        """, {
            'session_lifetime': SESSION_LIFETIME,
            'challenge_ids': tuple(self.ids),
            'yesterday': yesterday
        })

        Goals.browse(goal_id for [goal_id] in self.env.cr.fetchall()).update_goal()

        self._recompute_challenge_users()
        self._generate_goals_from_challenge()

        for challenge in self:
            if challenge.last_report_date != fields.Date.today():
                if challenge.next_report_date and fields.Date.today() >= challenge.next_report_date:
                    challenge.report_progress()
                else:
                    # goals closed but still opened at the last report date
                    closed_goals_to_report = Goals.search([
                        ('challenge_id', '=', challenge.id),
                        ('start_date', '>=', challenge.last_report_date),
                        ('end_date', '<=', challenge.last_report_date)
                    ])
                    if closed_goals_to_report:
                        # some goals need a final report
                        challenge.report_progress(subset_goals=closed_goals_to_report)

        self._check_challenge_reward()
        return True

    def _get_challenger_users(self, domain):
        user_domain = ast.literal_eval(domain)
        return self.env['res.users'].search(user_domain)

    def _recompute_challenge_users(self):
        """Recompute the domain to add new users and remove the one no longer matching the domain"""
        for challenge in self.filtered(lambda c: c.user_domain):
            current_users = challenge.user_ids
            new_users = self._get_challenger_users(challenge.user_domain)

            if current_users != new_users:
                challenge.user_ids = new_users

        return True

    def action_start(self):
        """Start a challenge"""
        return self.write({'state': 'inprogress'})

    def action_check(self):
        """Check a challenge

        Create goals that haven't been created yet (eg: if added users)
        Recompute the current value for each goal related"""
        self.env['gamification.goal'].search([
            ('challenge_id', 'in', self.ids),
            ('state', '=', 'inprogress')
        ]).unlink()

        return self._update_all()

    def action_report_progress(self):
        """Manual report of a goal, does not influence automatic report frequency"""
        for challenge in self:
            challenge.report_progress()
        return True

    def action_view_users(self):
        """ Redirect to the participants (users) list. """
        action = self.env["ir.actions.actions"]._for_xml_id("base.action_res_users")
        action['domain'] = [('id', 'in', self.user_ids.ids)]
        return action

    ##### Automatic actions #####

    def _generate_goals_from_challenge(self):
        """Generate the goals for each line and user.

        If goals already exist for this line and user, the line is skipped. This
        can be called after each change in the list of users or lines.
        :param list(int) ids: the list of challenge concerned"""

        Goals = self.env['gamification.goal']
        for challenge in self:
            (start_date, end_date) = start_end_date_for_period(challenge.period, challenge.start_date, challenge.end_date)
            to_update = Goals.browse(())

            for line in challenge.line_ids:
                # there is potentially a lot of users
                # detect the ones with no goal linked to this line
                date_clause = ""
                query_params = [line.id]
                if start_date:
                    date_clause += " AND g.start_date = %s"
                    query_params.append(start_date)
                if end_date:
                    date_clause += " AND g.end_date = %s"
                    query_params.append(end_date)

                query = """SELECT u.id AS user_id
                             FROM res_users u
                        LEFT JOIN gamification_goal g
                               ON (u.id = g.user_id)
                            WHERE line_id = %s
                              {date_clause}
                        """.format(date_clause=date_clause)
                self.env.cr.execute(query, query_params)
                user_with_goal_ids = {it for [it] in self.env.cr._obj}

                participant_user_ids = set(challenge.user_ids.ids)
                user_squating_challenge_ids = user_with_goal_ids - participant_user_ids
                if user_squating_challenge_ids:
                    # users that used to match the challenge
                    Goals.search([
                        ('challenge_id', '=', challenge.id),
                        ('user_id', 'in', list(user_squating_challenge_ids))
                    ]).unlink()

                values = {
                    'definition_id': line.definition_id.id,
                    'line_id': line.id,
                    'target_goal': line.target_goal,
                    'state': 'inprogress',
                }

                if start_date:
                    values['start_date'] = start_date
                if end_date:
                    values['end_date'] = end_date

                # the goal is initialised over the limit to make sure we will compute it at least once
                if line.condition == 'higher':
                    values['current'] = min(line.target_goal - 1, 0)
                else:
                    values['current'] = max(line.target_goal + 1, 0)

                if challenge.remind_update_delay:
                    values['remind_update_delay'] = challenge.remind_update_delay

                for user_id in (participant_user_ids - user_with_goal_ids):
                    values['user_id'] = user_id
                    to_update |= Goals.create(values)

            to_update.update_goal()

            if self.env.context.get('commit_gamification'):
                self.env.cr.commit()

        return True

    ##### JS utilities #####

    def _get_serialized_challenge_lines(self, user=(), restrict_goals=(), restrict_top=0):
        """Return a serialised version of the goals information if the user has not completed every goal

        :param user: user retrieving progress (False if no distinction,
                     only for ranking challenges)
        :param restrict_goals: compute only the results for this subset of
                               gamification.goal ids, if False retrieve every
                               goal of current running challenge
        :param int restrict_top: for challenge lines where visibility_mode is
                                 ``ranking``, retrieve only the best
                                 ``restrict_top`` results and itself, if 0
                                 retrieve all restrict_goal_ids has priority
                                 over restrict_top

        format list
        # if visibility_mode == 'ranking'
        {
            'name': <gamification.goal.description name>,
            'description': <gamification.goal.description description>,
            'condition': <reach condition {lower,higher}>,
            'computation_mode': <target computation {manually,count,sum,python}>,
            'monetary': <{True,False}>,
            'suffix': <value suffix>,
            'action': <{True,False}>,
            'display_mode': <{progress,boolean}>,
            'target': <challenge line target>,
            'own_goal_id': <gamification.goal id where user_id == uid>,
            'goals': [
                {
                    'id': <gamification.goal id>,
                    'rank': <user ranking>,
                    'user_id': <res.users id>,
                    'name': <res.users name>,
                    'state': <gamification.goal state {draft,inprogress,reached,failed,canceled}>,
                    'completeness': <percentage>,
                    'current': <current value>,
                }
            ]
        },
        # if visibility_mode == 'personal'
        {
            'id': <gamification.goal id>,
            'name': <gamification.goal.description name>,
            'description': <gamification.goal.description description>,
            'condition': <reach condition {lower,higher}>,
            'computation_mode': <target computation {manually,count,sum,python}>,
            'monetary': <{True,False}>,
            'suffix': <value suffix>,
            'action': <{True,False}>,
            'display_mode': <{progress,boolean}>,
            'target': <challenge line target>,
            'state': <gamification.goal state {draft,inprogress,reached,failed,canceled}>,
            'completeness': <percentage>,
            'current': <current value>,
        }
        """
        Goals = self.env['gamification.goal']
        (start_date, end_date) = start_end_date_for_period(self.period)

        res_lines = []
        for line in self.line_ids:
            line_data = {
                'name': line.definition_id.name,
                'description': line.definition_id.description,
                'condition': line.definition_id.condition,
                'computation_mode': line.definition_id.computation_mode,
                'monetary': line.definition_id.monetary,
                'suffix': line.definition_id.suffix,
                'action': True if line.definition_id.action_id else False,
                'display_mode': line.definition_id.display_mode,
                'target': line.target_goal,
            }
            domain = [
                ('line_id', '=', line.id),
                ('state', '!=', 'draft'),
            ]
            if restrict_goals:
                domain.append(('id', 'in', restrict_goals.ids))
            else:
                # if no subset goals, use the dates for restriction
                if start_date:
                    domain.append(('start_date', '=', start_date))
                if end_date:
                    domain.append(('end_date', '=', end_date))

            if self.visibility_mode == 'personal':
                if not user:
                    raise exceptions.UserError(_("Retrieving progress for personal challenge without user information"))

                domain.append(('user_id', '=', user.id))

                goal = Goals.search_fetch(domain, ['current', 'completeness', 'state'], limit=1)
                if not goal:
                    continue

                if goal.state != 'reached':
                    return []
                line_data.update({
                    fname: goal[fname]
                    for fname in ['id', 'current', 'completeness', 'state']
                })
                res_lines.append(line_data)
                continue

            line_data['own_goal_id'] = False,
            line_data['goals'] = []
            goals = Goals.search(domain, order='id')
            if not goals:
                continue
            goals = goals.sorted(key=lambda goal: (
                -goal.completeness, -goal.current if line.condition == 'higher' else goal.current
            ))

            for ranking, goal in enumerate(goals):
                if user and goal.user_id == user:
                    line_data['own_goal_id'] = goal.id
                elif restrict_top and ranking > restrict_top:
                    # not own goal and too low to be in top
                    continue

                line_data['goals'].append({
                    'id': goal.id,
                    'user_id': goal.user_id.id,
                    'name': goal.user_id.name,
                    'rank': ranking,
                    'current': goal.current,
                    'completeness': goal.completeness,
                    'state': goal.state,
                })
            if len(goals) < 3:
                # display at least the top 3 in the results
                missing = 3 - len(goals)
                for ranking, mock_goal in enumerate([{'id': False,
                                                      'user_id': False,
                                                      'name': '',
                                                      'current': 0,
                                                      'completeness': 0,
                                                      'state': False}] * missing,
                                                    start=len(goals)):
                    mock_goal['rank'] = ranking
                    line_data['goals'].append(mock_goal)

            res_lines.append(line_data)
        return res_lines

    ##### Reporting #####

    def report_progress(self, users=(), subset_goals=False):
        """Post report about the progress of the goals

        :param users: users that are concerned by the report. If False, will
                      send the report to every user concerned (goal users and
                      group that receive a copy). Only used for challenge with
                      a visibility mode set to 'personal'.
        :param subset_goals: goals to restrict the report
        """

        challenge = self

        if challenge.visibility_mode == 'ranking':
            lines_boards = challenge._get_serialized_challenge_lines(restrict_goals=subset_goals)

            body_html = challenge.report_template_id.with_context(challenge_lines=lines_boards)._render_field('body_html', challenge.ids)[challenge.id]

            # send to every follower and participant of the challenge
            challenge.message_post(
                body=body_html,
                partner_ids=challenge.mapped('user_ids.partner_id.id'),
                subtype_xmlid='mail.mt_comment',
                email_layout_xmlid='mail.mail_notification_light',
                )
            if challenge.report_message_group_id:
                challenge.report_message_group_id.message_post(
                    body=body_html,
                    subtype_xmlid='mail.mt_comment')

        else:
            # generate individual reports
            for user in (users or challenge.user_ids):
                lines = challenge._get_serialized_challenge_lines(user, restrict_goals=subset_goals)
                if not lines:
                    continue

                body_html = challenge.report_template_id.with_user(user).with_context(challenge_lines=lines)._render_field('body_html', challenge.ids)[challenge.id]

                # notify message only to users, do not post on the challenge
                challenge.message_notify(
                    body=body_html,
                    partner_ids=[user.partner_id.id],
                    subtype_xmlid='mail.mt_comment',
                    email_layout_xmlid='mail.mail_notification_light',
                )
                if challenge.report_message_group_id:
                    challenge.report_message_group_id.message_post(
                        body=body_html,
                        subtype_xmlid='mail.mt_comment',
                        email_layout_xmlid='mail.mail_notification_light',
                    )
        return challenge.write({'last_report_date': fields.Date.today()})

    ##### Challenges #####
    def accept_challenge(self):
        user = self.env.user
        sudoed = self.sudo()
        sudoed.message_post(body=_("%s has joined the challenge", user.name))
        sudoed.write({'invited_user_ids': [(3, user.id)], 'user_ids': [(4, user.id)]})
        return sudoed._generate_goals_from_challenge()

    def discard_challenge(self):
        """The user discard the suggested challenge"""
        user = self.env.user
        sudoed = self.sudo()
        sudoed.message_post(body=_("%s has refused the challenge", user.name))
        return sudoed.write({'invited_user_ids': (3, user.id)})

    def _check_challenge_reward(self, force=False):
        """Actions for the end of a challenge

        If a reward was selected, grant it to the correct users.
        Rewards granted at:
            - the end date for a challenge with no periodicity
            - the end of a period for challenge with periodicity
            - when a challenge is manually closed
        (if no end date, a running challenge is never rewarded)
        """
        commit = self.env.context.get('commit_gamification') and self.env.cr.commit

        for challenge in self:
            (start_date, end_date) = start_end_date_for_period(challenge.period, challenge.start_date, challenge.end_date)
            yesterday = date.today() - timedelta(days=1)

            rewarded_users = self.env['res.users']
            challenge_ended = force or end_date == fields.Date.to_string(yesterday)
            if challenge.reward_id and (challenge_ended or challenge.reward_realtime):
                # not using start_date as intemportal goals have a start date but no end_date
                reached_goals = self.env['gamification.goal']._read_group([
                    ('challenge_id', '=', challenge.id),
                    ('end_date', '=', end_date),
                    ('state', '=', 'reached')
                ], groupby=['user_id'], aggregates=['__count'])
                for user, count in reached_goals:
                    if count == len(challenge.line_ids):
                        # the user has succeeded every assigned goal
                        if challenge.reward_realtime:
                            badges = self.env['gamification.badge.user'].search_count([
                                ('challenge_id', '=', challenge.id),
                                ('badge_id', '=', challenge.reward_id.id),
                                ('user_id', '=', user.id),
                            ])
                            if badges > 0:
                                # has already recieved the badge for this challenge
                                continue
                        challenge._reward_user(user, challenge.reward_id)
                        rewarded_users |= user
                        if commit:
                            commit()

            if challenge_ended:
                # open chatter message
                message_body = _("The challenge %s is finished.", challenge.name)

                if rewarded_users:
                    message_body += Markup("<br/>") + _(
                        "Reward (badge %(badge_name)s) for every succeeding user was sent to %(users)s.",
                        badge_name=challenge.reward_id.name,
                        users=", ".join(rewarded_users.mapped('display_name'))
                    )
                else:
                    message_body += Markup("<br/>") + _("Nobody has succeeded to reach every goal, no badge is rewarded for this challenge.")

                # reward bests
                reward_message = Markup("<br/> %(rank)d. %(user_name)s - %(reward_name)s")
                if challenge.reward_first_id:
                    (first_user, second_user, third_user) = challenge._get_topN_users(MAX_VISIBILITY_RANKING)
                    if first_user:
                        challenge._reward_user(first_user, challenge.reward_first_id)
                        message_body += Markup("<br/>") + _("Special rewards were sent to the top competing users. The ranking for this challenge is:")
                        message_body += reward_message % {
                            'rank': 1,
                            'user_name': first_user.name,
                            'reward_name': challenge.reward_first_id.name,
                        }
                    else:
                        message_body += _("Nobody reached the required conditions to receive special badges.")

                    if second_user and challenge.reward_second_id:
                        challenge._reward_user(second_user, challenge.reward_second_id)
                        message_body += reward_message % {
                            'rank': 2,
                            'user_name': second_user.name,
                            'reward_name': challenge.reward_second_id.name,
                        }
                    if third_user and challenge.reward_third_id:
                        challenge._reward_user(third_user, challenge.reward_third_id)
                        message_body += reward_message % {
                            'rank': 3,
                            'user_name': third_user.name,
                            'reward_name': challenge.reward_third_id.name,
                        }

                challenge.message_post(
                    partner_ids=[user.partner_id.id for user in challenge.user_ids],
                    body=message_body)
                if commit:
                    commit()

        return True

    def _get_topN_users(self, n):
        """Get the top N users for a defined challenge

        Ranking criterias:
            1. succeed every goal of the challenge
            2. total completeness of each goal (can be over 100)

        Only users having reached every goal of the challenge will be returned
        unless the challenge ``reward_failure`` is set, in which case any user
        may be considered.

        :returns: an iterable of exactly N records, either User objects or
                  False if there was no user for the rank. There can be no
                  False between two users (if users[k] = False then
                  users[k+1] = False
        """
        Goals = self.env['gamification.goal']
        (start_date, end_date) = start_end_date_for_period(self.period, self.start_date, self.end_date)
        challengers = []
        for user in self.user_ids:
            all_reached = True
            total_completeness = 0
            # every goal of the user for the running period
            goal_ids = Goals.search([
                ('challenge_id', '=', self.id),
                ('user_id', '=', user.id),
                ('start_date', '=', start_date),
                ('end_date', '=', end_date)
            ])
            for goal in goal_ids:
                if goal.state != 'reached':
                    all_reached = False
                if goal.definition_condition == 'higher':
                    # can be over 100
                    total_completeness += (100.0 * goal.current / goal.target_goal) if goal.target_goal else 0
                elif goal.state == 'reached':
                    # for lower goals, can not get percentage so 0 or 100
                    total_completeness += 100

            challengers.append({'user': user, 'all_reached': all_reached, 'total_completeness': total_completeness})

        challengers.sort(key=lambda k: (k['all_reached'], k['total_completeness']), reverse=True)
        if not self.reward_failure:
            # only keep the fully successful challengers at the front, could
            # probably use filter since the successful ones are at the front
            challengers = itertools.takewhile(lambda c: c['all_reached'], challengers)

        # append a tail of False, then keep the first N
        challengers = itertools.islice(
            itertools.chain(
                (c['user'] for c in challengers),
                itertools.repeat(False),
            ), 0, n
        )

        return tuple(challengers)

    def _reward_user(self, user, badge):
        """Create a badge user and send the badge to him

        :param user: the user to reward
        :param badge: the concerned badge
        """
        return self.env['gamification.badge.user'].create({
            'user_id': user.id,
            'badge_id': badge.id,
            'challenge_id': self.id
        })._send_badge()

```

## File: models\gamification_challenge_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class ChallengeLine(models.Model):
    """Gamification challenge line

    Predefined goal for 'gamification_challenge'
    These are generic list of goals with only the target goal defined
    Should only be created for the gamification.challenge object
    """
    _name = 'gamification.challenge.line'
    _description = 'Gamification generic goal for challenge'
    _order = "sequence, id"

    challenge_id = fields.Many2one('gamification.challenge', string='Challenge', required=True, ondelete="cascade")
    definition_id = fields.Many2one('gamification.goal.definition', string='Goal Definition', required=True, ondelete="cascade")

    sequence = fields.Integer('Sequence', default=1)
    target_goal = fields.Float('Target Value to Reach', required=True)

    name = fields.Char("Name", related='definition_id.name', readonly=False)
    condition = fields.Selection(string="Condition", related='definition_id.condition', readonly=True)
    definition_suffix = fields.Char("Unit", related='definition_id.suffix', readonly=True)
    definition_monetary = fields.Boolean("Monetary", related='definition_id.monetary', readonly=True)
    definition_full_suffix = fields.Char("Suffix", related='definition_id.full_suffix', readonly=True)

```

## File: models\gamification_goal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast
import logging
from datetime import date, datetime, timedelta

from odoo import api, fields, models, _, exceptions
from odoo.tools.safe_eval import safe_eval, time

_logger = logging.getLogger(__name__)


class Goal(models.Model):
    """Goal instance for a user

    An individual goal for a user on a specified time period"""

    _name = 'gamification.goal'
    _description = 'Gamification Goal'
    _rec_name = 'definition_id'
    _order = 'start_date desc, end_date desc, definition_id, id'

    definition_id = fields.Many2one('gamification.goal.definition', string="Goal Definition", required=True, ondelete="cascade")
    user_id = fields.Many2one('res.users', string="User", required=True, auto_join=True, ondelete="cascade")
    line_id = fields.Many2one('gamification.challenge.line', string="Challenge Line", ondelete="cascade")
    challenge_id = fields.Many2one(
        related='line_id.challenge_id', store=True, readonly=True, index=True,
        help="Challenge that generated the goal, assign challenge to users "
             "to generate goals with a value in this field.")
    start_date = fields.Date("Start Date", default=fields.Date.today)
    end_date = fields.Date("End Date")  # no start and end = always active
    target_goal = fields.Float('To Reach', required=True)
# no goal = global index
    current = fields.Float("Current Value", required=True, default=0)
    completeness = fields.Float("Completeness", compute='_get_completion')
    state = fields.Selection([
        ('draft', "Draft"),
        ('inprogress', "In progress"),
        ('reached', "Reached"),
        ('failed', "Failed"),
        ('canceled', "Cancelled"),
    ], default='draft', string='State', required=True)
    to_update = fields.Boolean('To update')
    closed = fields.Boolean('Closed goal')

    computation_mode = fields.Selection(related='definition_id.computation_mode', readonly=False)
    color = fields.Integer("Color Index", compute='_compute_color')
    remind_update_delay = fields.Integer(
        "Remind delay", help="The number of days after which the user "
                             "assigned to a manual goal will be reminded. "
                             "Never reminded if no value is specified.")
    last_update = fields.Date(
        "Last Update",
        help="In case of manual goal, reminders are sent if the goal as not "
             "been updated for a while (defined in challenge). Ignored in "
             "case of non-manual goal or goal not linked to a challenge.")

    definition_description = fields.Text("Definition Description", related='definition_id.description', readonly=True)
    definition_condition = fields.Selection(string="Definition Condition", related='definition_id.condition', readonly=True)
    definition_suffix = fields.Char("Suffix", related='definition_id.full_suffix', readonly=True)
    definition_display = fields.Selection(string="Display Mode", related='definition_id.display_mode', readonly=True)

    @api.depends('end_date', 'last_update', 'state')
    def _compute_color(self):
        """Set the color based on the goal's state and completion"""
        for goal in self:
            goal.color = 0
            if (goal.end_date and goal.last_update):
                if (goal.end_date < goal.last_update) and (goal.state == 'failed'):
                    goal.color = 2
                elif (goal.end_date < goal.last_update) and (goal.state == 'reached'):
                    goal.color = 5

    @api.depends('current', 'target_goal', 'definition_id.condition')
    def _get_completion(self):
        """Return the percentage of completeness of the goal, between 0 and 100"""
        for goal in self:
            if goal.definition_condition == 'higher':
                if goal.current >= goal.target_goal:
                    goal.completeness = 100.0
                else:
                    goal.completeness = round(100.0 * goal.current / goal.target_goal, 2) if goal.target_goal else 0
            elif goal.current < goal.target_goal:
                # a goal 'lower than' has only two values possible: 0 or 100%
                goal.completeness = 100.0
            else:
                goal.completeness = 0.0

    def _check_remind_delay(self):
        """Verify if a goal has not been updated for some time and send a
        reminder message of needed.

        :return: data to write on the goal object
        """
        if not (self.remind_update_delay and self.last_update):
            return {}

        delta_max = timedelta(days=self.remind_update_delay)
        last_update = fields.Date.from_string(self.last_update)
        if date.today() - last_update < delta_max:
            return {}

        # generate a reminder report
        body_html = self.env.ref('gamification.email_template_goal_reminder')._render_field('body_html', self.ids, compute_lang=True)[self.id]
        self.message_notify(
            body=body_html,
            partner_ids=[self.user_id.partner_id.id],
            subtype_xmlid='mail.mt_comment',
            email_layout_xmlid='mail.mail_notification_light',
        )

        return {'to_update': True}

    def _get_write_values(self, new_value):
        """Generate values to write after recomputation of a goal score"""
        if new_value == self.current:
            # avoid useless write if the new value is the same as the old one
            return {}

        result = {'current': new_value}
        if (self.definition_id.condition == 'higher' and new_value >= self.target_goal) \
          or (self.definition_id.condition == 'lower' and new_value <= self.target_goal):
            # success, do no set closed as can still change
            result['state'] = 'reached'

        elif self.end_date and fields.Date.today() > self.end_date:
            # check goal failure
            result['state'] = 'failed'
            result['closed'] = True

        return {self: result}

    def update_goal(self):
        """Update the goals to recomputes values and change of states

        If a manual goal is not updated for enough time, the user will be
        reminded to do so (done only once, in 'inprogress' state).
        If a goal reaches the target value, the status is set to reached
        If the end date is passed (at least +1 day, time not considered) without
        the target value being reached, the goal is set as failed."""
        goals_by_definition = {}
        for goal in self.with_context(prefetch_fields=False):
            goals_by_definition.setdefault(goal.definition_id, []).append(goal)

        for definition, goals in goals_by_definition.items():
            goals_to_write = {}
            if definition.computation_mode == 'manually':
                for goal in goals:
                    goals_to_write[goal] = goal._check_remind_delay()
            elif definition.computation_mode == 'python':
                # TODO batch execution
                for goal in goals:
                    # execute the chosen method
                    cxt = {
                        'object': goal,
                        'env': self.env,

                        'date': date,
                        'datetime': datetime,
                        'timedelta': timedelta,
                        'time': time,
                    }
                    code = definition.compute_code.strip()
                    safe_eval(code, cxt, mode="exec", nocopy=True)
                    # the result of the evaluated codeis put in the 'result' local variable, propagated to the context
                    result = cxt.get('result')
                    if isinstance(result, (float, int)):
                        goals_to_write.update(goal._get_write_values(result))
                    else:
                        _logger.error(
                            "Invalid return content '%r' from the evaluation "
                            "of code for definition %s, expected a number",
                            result, definition.name)

            elif definition.computation_mode in ('count', 'sum'):  # count or sum
                Obj = self.env[definition.model_id.model]

                field_date_name = definition.field_date_id.name
                if definition.batch_mode:
                    # batch mode, trying to do as much as possible in one request
                    general_domain = ast.literal_eval(definition.domain)
                    field_name = definition.batch_distinctive_field.name
                    subqueries = {}
                    for goal in goals:
                        start_date = field_date_name and goal.start_date or False
                        end_date = field_date_name and goal.end_date or False
                        subqueries.setdefault((start_date, end_date), {}).update({goal.id:safe_eval(definition.batch_user_expression, {'user': goal.user_id})})

                    # the global query should be split by time periods (especially for recurrent goals)
                    for (start_date, end_date), query_goals in subqueries.items():
                        subquery_domain = list(general_domain)
                        subquery_domain.append((field_name, 'in', list(set(query_goals.values()))))
                        if start_date:
                            subquery_domain.append((field_date_name, '>=', start_date))
                        if end_date:
                            subquery_domain.append((field_date_name, '<=', end_date))

                        if definition.computation_mode == 'count':
                            user_values = Obj._read_group(subquery_domain, groupby=[field_name], aggregates=['__count'])

                        else:  # sum
                            value_field_name = definition.field_id.name
                            user_values = Obj._read_group(subquery_domain, groupby=[field_name], aggregates=[f'{value_field_name}:sum'])

                        # user_values has format of _read_group: [(<partner>, <aggregate>), ...]
                        for goal in [g for g in goals if g.id in query_goals]:
                            for field_value, aggregate in user_values:
                                queried_value = field_value.id if isinstance(field_value, models.Model) else field_value
                                if queried_value == query_goals[goal.id]:
                                    goals_to_write.update(goal._get_write_values(aggregate))

                else:
                    for goal in goals:
                        # eval the domain with user replaced by goal user object
                        domain = safe_eval(definition.domain, {'user': goal.user_id})

                        # add temporal clause(s) to the domain if fields are filled on the goal
                        if goal.start_date and field_date_name:
                            domain.append((field_date_name, '>=', goal.start_date))
                        if goal.end_date and field_date_name:
                            domain.append((field_date_name, '<=', goal.end_date))

                        if definition.computation_mode == 'sum':
                            field_name = definition.field_id.name
                            res = Obj._read_group(domain, [], [f'{field_name}:{definition.computation_mode}'])
                            new_value = res[0][0] or 0.0

                        else:  # computation mode = count
                            new_value = Obj.search_count(domain)

                        goals_to_write.update(goal._get_write_values(new_value))

            else:
                _logger.error(
                    "Invalid computation mode '%s' in definition %s",
                    definition.computation_mode, definition.name)

            for goal, values in goals_to_write.items():
                if not values:
                    continue
                goal.write(values)
            if self.env.context.get('commit_gamification'):
                self.env.cr.commit()
        return True

    def action_start(self):
        """Mark a goal as started.

        This should only be used when creating goals manually (in draft state)"""
        self.write({'state': 'inprogress'})
        return self.update_goal()

    def action_reach(self):
        """Mark a goal as reached.

        If the target goal condition is not met, the state will be reset to In
        Progress at the next goal update until the end date."""
        return self.write({'state': 'reached'})

    def action_fail(self):
        """Set the state of the goal to failed.

        A failed goal will be ignored in future checks."""
        return self.write({'state': 'failed'})

    def action_cancel(self):
        """Reset the completion after setting a goal as reached or failed.

        This is only the current state, if the date and/or target criteria
        match the conditions for a change of state, this will be applied at the
        next goal update."""
        return self.write({'state': 'inprogress'})

    @api.model_create_multi
    def create(self, vals_list):
        return super(Goal, self.with_context(no_remind_goal=True)).create(vals_list)

    def write(self, vals):
        """Overwrite the write method to update the last_update field to today

        If the current value is changed and the report frequency is set to On
        change, a report is generated
        """
        vals['last_update'] = fields.Date.context_today(self)
        result = super(Goal, self).write(vals)
        for goal in self:
            if goal.state != "draft" and ('definition_id' in vals or 'user_id' in vals):
                # avoid drag&drop in kanban view
                raise exceptions.UserError(_('Can not modify the configuration of a started goal'))

            if vals.get('current') and 'no_remind_goal' not in self.env.context:
                if goal.challenge_id.report_message_frequency == 'onchange':
                    goal.challenge_id.sudo().report_progress(users=goal.user_id)
        return result

    def get_action(self):
        """Get the ir.action related to update the goal

        In case of a manual goal, should return a wizard to update the value
        :return: action description in a dictionary
        """
        if self.definition_id.action_id:
            # open a the action linked to the goal
            action = self.definition_id.action_id.read()[0]

            if self.definition_id.res_id_field:
                current_user = self.env.user.with_user(self.env.user)
                action['res_id'] = safe_eval(self.definition_id.res_id_field, {
                    'user': current_user
                })

                # if one element to display, should see it in form mode if possible
                action['views'] = [
                    (view_id, mode)
                    for (view_id, mode) in action['views']
                    if mode == 'form'
                ] or action['views']
            return action

        if self.computation_mode == 'manually':
            # open a wizard window to update the value manually
            action = {
                'name': _("Update %s", self.definition_id.name),
                'id': self.id,
                'type': 'ir.actions.act_window',
                'views': [[False, 'form']],
                'target': 'new',
                'context': {'default_goal_id': self.id, 'default_current': self.current},
                'res_model': 'gamification.goal.wizard'
            }
            return action

        return False

```

## File: models\gamification_goal_definition.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _, exceptions
from odoo.tools.safe_eval import safe_eval

DOMAIN_TEMPLATE = "[('store', '=', True), '|', ('model_id', '=', model_id), ('model_id', 'in', model_inherited_ids)%s]"


class GoalDefinition(models.Model):
    """Goal definition

    A goal definition contains the way to evaluate an objective
    Each module wanting to be able to set goals to the users needs to create
    a new gamification_goal_definition
    """
    _name = 'gamification.goal.definition'
    _description = 'Gamification Goal Definition'

    name = fields.Char("Goal Definition", required=True, translate=True)
    description = fields.Text("Goal Description")
    monetary = fields.Boolean("Monetary Value", default=False, help="The target and current value are defined in the company currency.")
    suffix = fields.Char("Suffix", help="The unit of the target and current values", translate=True)
    full_suffix = fields.Char("Full Suffix", compute='_compute_full_suffix', help="The currency and suffix field")
    computation_mode = fields.Selection([
        ('manually', "Recorded manually"),
        ('count', "Automatic: number of records"),
        ('sum', "Automatic: sum on a field"),
        ('python', "Automatic: execute a specific Python code"),
    ], default='manually', string="Computation Mode", required=True,
       help="Define how the goals will be computed. The result of the operation will be stored in the field 'Current'.")
    display_mode = fields.Selection([
        ('progress', "Progressive (using numerical values)"),
        ('boolean', "Exclusive (done or not-done)"),
    ], default='progress', string="Displayed as", required=True)
    model_id = fields.Many2one('ir.model', string='Model', ondelete='cascade')
    model_inherited_ids = fields.Many2many('ir.model', related='model_id.inherited_model_ids')
    field_id = fields.Many2one(
        'ir.model.fields', string='Field to Sum',
        domain=DOMAIN_TEMPLATE % ''
    )
    field_date_id = fields.Many2one(
        'ir.model.fields', string='Date Field', help='The date to use for the time period evaluated',
        domain=DOMAIN_TEMPLATE % ", ('ttype', 'in', ('date', 'datetime'))"
    )
    domain = fields.Char(
        "Filter Domain", required=True, default="[]",
        help="Domain for filtering records. General rule, not user depending,"
             " e.g. [('state', '=', 'done')]. The expression can contain"
             " reference to 'user' which is a browse record of the current"
             " user if not in batch mode.")

    batch_mode = fields.Boolean("Batch Mode", help="Evaluate the expression in batch instead of once for each user")
    batch_distinctive_field = fields.Many2one('ir.model.fields', string="Distinctive field for batch user", help="In batch mode, this indicates which field distinguishes one user from the other, e.g. user_id, partner_id...")
    batch_user_expression = fields.Char("Evaluated expression for batch mode", help="The value to compare with the distinctive field. The expression can contain reference to 'user' which is a browse record of the current user, e.g. user.id, user.partner_id.id...")
    compute_code = fields.Text("Python Code", help="Python code to be executed for each user. 'result' should contains the new current value. Evaluated user can be access through object.user_id.")
    condition = fields.Selection([
        ('higher', "The higher the better"),
        ('lower', "The lower the better")
    ], default='higher', required=True, string="Goal Performance",
       help="A goal is considered as completed when the current value is compared to the value to reach")
    action_id = fields.Many2one('ir.actions.act_window', string="Action", help="The action that will be called to update the goal value.")
    res_id_field = fields.Char("ID Field of user", help="The field name on the user profile (res.users) containing the value for res_id for action.")

    @api.depends('suffix', 'monetary')  # also depends of user...
    def _compute_full_suffix(self):
        for goal in self:
            items = []

            if goal.monetary:
                items.append(self.env.company.currency_id.symbol or u'¤')
            if goal.suffix:
                items.append(goal.suffix)

            goal.full_suffix = u' '.join(items)

    def _check_domain_validity(self):
        # take admin as should always be present
        for definition in self:
            if definition.computation_mode not in ('count', 'sum'):
                continue

            Obj = self.env[definition.model_id.model]
            try:
                domain = safe_eval(definition.domain, {
                    'user': self.env.user.with_user(self.env.user)
                })
                # dummy search to make sure the domain is valid
                Obj.search_count(domain)
            except (ValueError, SyntaxError) as e:
                msg = e
                if isinstance(e, SyntaxError):
                    msg = (e.msg + '\n' + e.text)
                raise exceptions.UserError(_(
                    "The domain for the definition %(definition)s seems incorrect, please check it.\n\n%(error_message)s",
                    definition=definition.name,
                    error_message=msg,
                ))
        return True

    def _check_model_validity(self):
        """ make sure the selected field and model are usable"""
        for definition in self:
            try:
                if not (definition.model_id and definition.field_id):
                    continue

                Model = self.env[definition.model_id.model]
                field = Model._fields.get(definition.field_id.name)
                if not (field and field.store):
                    raise exceptions.UserError(_(
                        "The model configuration for the definition %(name)s seems incorrect, please check it.\n\n%(field_name)s not stored",
                        name=definition.name,
                        field_name=definition.field_id.name
                    ))
            except KeyError as e:
                raise exceptions.UserError(_(
                    "The model configuration for the definition %(name)s seems incorrect, please check it.\n\n%(error)s not found",
                    name=definition.name,
                    error=e
                ))

    @api.model_create_multi
    def create(self, vals_list):
        definitions = super(GoalDefinition, self).create(vals_list)
        definitions.filtered_domain([
            ('computation_mode', 'in', ['count', 'sum']),
        ])._check_domain_validity()
        definitions.filtered_domain([
            ('field_id', '=', 'True'),
        ])._check_model_validity()
        return definitions

    def write(self, vals):
        res = super(GoalDefinition, self).write(vals)
        if vals.get('computation_mode', 'count') in ('count', 'sum') and (vals.get('domain') or vals.get('model_id')):
            self._check_domain_validity()
        if vals.get('field_id') or vals.get('model_id') or vals.get('batch_mode'):
            self._check_model_validity()
        return res

```

## File: models\gamification_karma_rank.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, fields, models
from odoo.tools.translate import html_translate


class KarmaRank(models.Model):
    _name = 'gamification.karma.rank'
    _description = 'Rank based on karma'
    _inherit = 'image.mixin'
    _order = 'karma_min'

    name = fields.Text(string='Rank Name', translate=True, required=True)
    description = fields.Html(string='Description', translate=html_translate, sanitize_attributes=False,)
    description_motivational = fields.Html(
        string='Motivational', translate=html_translate, sanitize_attributes=False, sanitize_overridable=True,
        help="Motivational phrase to reach this rank on your profile page")
    karma_min = fields.Integer(
        string='Required Karma', required=True, default=1)
    user_ids = fields.One2many('res.users', 'rank_id', string='Users')
    rank_users_count = fields.Integer("# Users", compute="_compute_rank_users_count")

    _sql_constraints = [
        ('karma_min_check', "CHECK( karma_min > 0 )", 'The required karma has to be above 0.')
    ]

    @api.depends('user_ids')
    def _compute_rank_users_count(self):
        requests_data = self.env['res.users']._read_group([('rank_id', '!=', False)], ['rank_id'], ['__count'])
        requests_mapped_data = {rank.id: count for rank, count in requests_data}
        for rank in self:
            rank.rank_users_count = requests_mapped_data.get(rank.id, 0)

    @api.model_create_multi
    def create(self, values_list):
        res = super(KarmaRank, self).create(values_list)
        if any(res.mapped('karma_min')) > 0:
            users = self.env['res.users'].sudo().search([('karma', '>=', max(min(res.mapped('karma_min')), 1))])
            if users:
                users._recompute_rank()
        return res

    def write(self, vals):
        if 'karma_min' in vals:
            previous_ranks = self.env['gamification.karma.rank'].search([], order="karma_min DESC").ids
            low = min(vals['karma_min'], min(self.mapped('karma_min')))
            high = max(vals['karma_min'], max(self.mapped('karma_min')))

        res = super(KarmaRank, self).write(vals)

        if 'karma_min' in vals:
            after_ranks = self.env['gamification.karma.rank'].search([], order="karma_min DESC").ids
            if previous_ranks != after_ranks:
                users = self.env['res.users'].sudo().search([('karma', '>=', max(low, 1))])
            else:
                users = self.env['res.users'].sudo().search([('karma', '>=', max(low, 1)), ('karma', '<=', high)])
            users._recompute_rank()
        return res

```

## File: models\gamification_karma_tracking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from dateutil.relativedelta import relativedelta

from odoo import _, api, fields, models
from odoo.tools import date_utils


class KarmaTracking(models.Model):
    _name = 'gamification.karma.tracking'
    _description = 'Track Karma Changes'
    _rec_name = 'user_id'
    _order = 'tracking_date desc, id desc'

    def _get_origin_selection_values(self):
        return [('res.users', _('User'))]

    user_id = fields.Many2one('res.users', 'User', index=True, required=True, ondelete='cascade')
    old_value = fields.Integer('Old Karma Value', readonly=True)
    new_value = fields.Integer('New Karma Value', required=True)
    gain = fields.Integer('Gain', compute='_compute_gain', readonly=False)
    consolidated = fields.Boolean('Consolidated')

    tracking_date = fields.Datetime(default=fields.Datetime.now, readonly=True, index=True)
    reason = fields.Text(default=lambda self: _('Add Manually'), string='Description')
    origin_ref = fields.Reference(
        string='Source',
        selection=lambda self: self._get_origin_selection_values(),
        default=lambda self: f'res.users,{self.env.user.id}',
    )
    origin_ref_model_name = fields.Selection(
        string='Source Type', selection=lambda self: self._get_origin_selection_values(),
        compute='_compute_origin_ref_model_name', store=True)

    @api.depends('old_value', 'new_value')
    def _compute_gain(self):
        for karma in self:
            karma.gain = karma.new_value - (karma.old_value or 0)

    @api.depends('origin_ref')
    def _compute_origin_ref_model_name(self):
        for karma in self:
            if not karma.origin_ref:
                karma.origin_ref_model_name = False
                continue

            karma.origin_ref_model_name = karma.origin_ref._name

    @api.model_create_multi
    def create(self, values_list):
        # fill missing old value with current user karma
        users = self.env['res.users'].browse([
            values['user_id']
            for values in values_list
            if 'old_value' not in values and values.get('user_id')
        ])
        karma_per_users = {user.id: user.karma for user in users}

        for values in values_list:
            if 'old_value' not in values and values.get('user_id'):
                values['old_value'] = karma_per_users[values['user_id']]

            if 'gain' in values and 'old_value' in values:
                values['new_value'] = values['old_value'] + values['gain']
                del values['gain']

        return super().create(values_list)

    @api.model
    def _consolidate_cron(self):
        """Consolidate the trackings 2 months ago. Used by a cron to cleanup tracking records."""
        from_date = date_utils.start_of(fields.Datetime.today(), 'month') - relativedelta(months=2)
        return self._process_consolidate(from_date)

    def _process_consolidate(self, from_date, end_date=None):
        """Consolidate the karma trackings.

        The consolidation keeps, for each user, the oldest "old_value" and the most recent
        "new_value", creates a new karma tracking with those values and removes all karma
        trackings between those dates. The origin / reason is changed on the consolidated
        records, so this information is lost in the process.
        """
        self.env['gamification.karma.tracking'].flush_model()

        if not end_date:
            end_date = date_utils.end_of(date_utils.end_of(from_date, 'month'), 'day')

        select_query = """
        WITH old_tracking AS (
            SELECT DISTINCT ON (user_id) user_id, old_value, tracking_date
              FROM gamification_karma_tracking
             WHERE tracking_date BETWEEN %(from_date)s
               AND %(end_date)s
               AND consolidated IS NOT TRUE
          ORDER BY user_id, tracking_date ASC, id ASC
        )
            INSERT INTO gamification_karma_tracking (
                            user_id,
                            old_value,
                            new_value,
                            tracking_date,
                            origin_ref,
                            consolidated,
                            reason)
            SELECT DISTINCT ON (nt.user_id)
                            nt.user_id,
                            ot.old_value AS old_value,
                            nt.new_value AS new_value,
                            ot.tracking_date AS from_tracking_date,
                            %(origin_ref)s AS origin_ref,
                            TRUE,
                            %(reason)s
              FROM gamification_karma_tracking AS nt
              JOIN old_tracking AS ot
                   ON ot.user_id = nt.user_id
             WHERE nt.tracking_date BETWEEN %(from_date)s
               AND %(end_date)s
               AND nt.consolidated IS NOT TRUE
          ORDER BY nt.user_id, nt.tracking_date DESC, id DESC
        """

        self.env.cr.execute(select_query, {
            'from_date': from_date,
            'end_date': end_date,
            'origin_ref': f'res.users,{self.env.user.id}',
            'reason': _('Consolidation from %(from_date)s to %(end_date)s', from_date=from_date.date(), end_date=end_date.date()),
        })

        trackings = self.search([
            ('tracking_date', '>=', from_date),
            ('tracking_date', '<=', end_date),
            ('consolidated', '!=', True)]
        )
        # HACK: the unlink() AND the flush_all() must have that key in their context!
        trackings = trackings.with_context(skip_karma_computation=True)
        trackings.unlink()
        trackings.env.flush_all()
        return True

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.tools import SQL


class Users(models.Model):
    _inherit = 'res.users'

    karma = fields.Integer('Karma', compute='_compute_karma', store=True, readonly=False)
    karma_tracking_ids = fields.One2many('gamification.karma.tracking', 'user_id', string='Karma Changes', groups="base.group_system")
    badge_ids = fields.One2many('gamification.badge.user', 'user_id', string='Badges', copy=False)
    gold_badge = fields.Integer('Gold badges count', compute="_get_user_badge_level")
    silver_badge = fields.Integer('Silver badges count', compute="_get_user_badge_level")
    bronze_badge = fields.Integer('Bronze badges count', compute="_get_user_badge_level")
    rank_id = fields.Many2one('gamification.karma.rank', 'Rank')
    next_rank_id = fields.Many2one('gamification.karma.rank', 'Next Rank')

    @api.depends('karma_tracking_ids.new_value')
    def _compute_karma(self):
        if self.env.context.get('skip_karma_computation'):
            # do not need to update the user karma
            # e.g. during the tracking consolidation
            return

        self.env['gamification.karma.tracking'].flush_model()

        select_query = """
            SELECT DISTINCT ON (user_id) user_id, new_value
              FROM gamification_karma_tracking
             WHERE user_id = ANY(%(user_ids)s)
          ORDER BY user_id, tracking_date DESC, id DESC
        """
        self.env.cr.execute(select_query, {'user_ids': self.ids})

        user_karma_map = {
            values['user_id']: values['new_value']
            for values in self.env.cr.dictfetchall()
        }

        for user in self:
            user.karma = user_karma_map.get(user.id, 0)

        self.sudo()._recompute_rank()

    @api.depends('badge_ids')
    def _get_user_badge_level(self):
        """ Return total badge per level of users
        TDE CLEANME: shouldn't check type is forum ? """
        for user in self:
            user.gold_badge = 0
            user.silver_badge = 0
            user.bronze_badge = 0

        self.env.cr.execute("""
            SELECT bu.user_id, b.level, count(1)
            FROM gamification_badge_user bu, gamification_badge b
            WHERE bu.user_id IN %s
              AND bu.badge_id = b.id
              AND b.level IS NOT NULL
            GROUP BY bu.user_id, b.level
            ORDER BY bu.user_id;
        """, [tuple(self.ids)])

        for (user_id, level, count) in self.env.cr.fetchall():
            # levels are gold, silver, bronze but fields have _badge postfix
            self.browse(user_id)['{}_badge'.format(level)] = count

    @api.model_create_multi
    def create(self, values_list):
        res = super(Users, self).create(values_list)

        self._add_karma_batch({
            user: {
                'gain': int(vals['karma']),
                'old_value': 0,
                'origin_ref': f'res.users,{self.env.uid}',
                'reason': _('User Creation'),
            }
            for user, vals in zip(res, values_list)
            if vals.get('karma')
        })

        return res

    def write(self, values):
        if 'karma' in values:
            self._add_karma_batch({
                user: {
                    'gain': int(values['karma']) - user.karma,
                    'origin_ref': f'res.users,{self.env.uid}',
                }
                for user in self
                if int(values['karma']) != user.karma
            })
        return super().write(values)

    def _add_karma(self, gain, source=None, reason=None):
        self.ensure_one()
        values = {'gain': gain, 'source': source, 'reason': reason}
        return self._add_karma_batch({self: values})

    def _add_karma_batch(self, values_per_user):
        if not values_per_user:
            return

        create_values = []
        for user, values in values_per_user.items():
            origin = values.get('source') or self.env.user
            reason = values.get('reason') or _('Add Manually')
            origin_description = f'{origin.display_name} #{origin.id}'
            old_value = values.get('old_value', user.karma)

            create_values.append({
                'new_value': old_value + values['gain'],
                'old_value': old_value,
                'origin_ref': f'{origin._name},{origin.id}',
                'reason': f'{reason} ({origin_description})',
                'user_id': user.id,
            })

        self.env['gamification.karma.tracking'].sudo().create(create_values)
        return True

    def _get_tracking_karma_gain_position(self, user_domain, from_date=None, to_date=None):
        """ Get absolute position in term of gained karma for users. First a ranking
        of all users is done given a user_domain; then the position of each user
        belonging to the current record set is extracted.

        Example: in website profile, search users with name containing Norbert. Their
        positions should not be 1 to 4 (assuming 4 results), but their actual position
        in the karma gain ranking (with example user_domain being karma > 1,
        website published True).

        :param user_domain: general domain (i.e. active, karma > 1, website, ...)
          to compute the absolute position of the current record set
        :param from_date: compute karma gained after this date (included) or from
          beginning of time;
        :param to_date: compute karma gained before this date (included) or until
          end of time;

        :return list: [{
            'user_id': user_id (belonging to current record set),
            'karma_gain_total': integer, karma gained in the given timeframe,
            'karma_position': integer, ranking position
        }, {..}] ordered by karma_position desc
        """
        if not self:
            return []

        where_query = self.env['res.users']._where_calc(user_domain)

        sql = SQL("""
SELECT final.user_id, final.karma_gain_total, final.karma_position
FROM (
    SELECT intermediate.user_id, intermediate.karma_gain_total, row_number() OVER (ORDER BY intermediate.karma_gain_total DESC) AS karma_position
    FROM (
        SELECT "res_users".id as user_id, COALESCE(SUM("tracking".new_value - "tracking".old_value), 0) as karma_gain_total
        FROM %s
        LEFT JOIN "gamification_karma_tracking" as "tracking"
        ON "res_users".id = "tracking".user_id AND "res_users"."active" = TRUE
        WHERE %s %s %s
        GROUP BY "res_users".id
        ORDER BY karma_gain_total DESC
    ) intermediate
) final
WHERE final.user_id IN %s""",
            where_query.from_clause,
            where_query.where_clause or SQL("TRUE"),
            SQL("AND tracking.tracking_date::DATE >= %s::DATE", from_date) if from_date else SQL(),
            SQL("AND tracking.tracking_date::DATE <= %s::DATE", to_date) if to_date else SQL(),
            tuple(self.ids),
        )

        self.env.cr.execute(sql)
        return self.env.cr.dictfetchall()

    def _get_karma_position(self, user_domain):
        """ Get absolute position in term of total karma for users. First a ranking
        of all users is done given a user_domain; then the position of each user
        belonging to the current record set is extracted.

        Example: in website profile, search users with name containing Norbert. Their
        positions should not be 1 to 4 (assuming 4 results), but their actual position
        in the total karma ranking (with example user_domain being karma > 1,
        website published True).

        :param user_domain: general domain (i.e. active, karma > 1, website, ...)
          to compute the absolute position of the current record set

        :return list: [{
            'user_id': user_id (belonging to current record set),
            'karma_position': integer, ranking position
        }, {..}] ordered by karma_position desc
        """
        if not self:
            return {}

        where_query = self.env['res.users']._where_calc(user_domain)

        # we search on every user in the DB to get the real positioning (not the one inside the subset)
        # then, we filter to get only the subset.
        sql = SQL("""
SELECT sub.user_id, sub.karma_position
FROM (
    SELECT "res_users"."id" as user_id, row_number() OVER (ORDER BY res_users.karma DESC) AS karma_position
    FROM %s
    WHERE %s
) sub
WHERE sub.user_id IN %s""",
            where_query.from_clause,
            where_query.where_clause or SQL("TRUE"),
            tuple(self.ids),
        )
        self.env.cr.execute(sql)
        return self.env.cr.dictfetchall()

    def _rank_changed(self):
        """
            Method that can be called on a batch of users with the same new rank
        """
        if self.env.context.get('install_mode', False):
            # avoid sending emails in install mode (prevents spamming users when creating data ranks)
            return

        template = self.env.ref('gamification.mail_template_data_new_rank_reached', raise_if_not_found=False)
        if template:
            for u in self:
                if u.rank_id.karma_min > 0:
                    template.send_mail(u.id, force_send=False, email_layout_xmlid='mail.mail_notification_light')

    def _recompute_rank(self):
        """
        The caller should filter the users on karma > 0 before calling this method
        to avoid looping on every single users

        Compute rank of each user by user.
        For each user, check the rank of this user
        """

        ranks = [{'rank': rank, 'karma_min': rank.karma_min} for rank in
                 self.env['gamification.karma.rank'].search([], order="karma_min DESC")]

        # 3 is the number of search/requests used by rank in _recompute_rank_bulk()
        if len(self) > len(ranks) * 3:
            self._recompute_rank_bulk()
            return

        for user in self:
            old_rank = user.rank_id
            if user.karma == 0 and ranks:
                user.write({'next_rank_id': ranks[-1]['rank'].id})
            else:
                for i in range(0, len(ranks)):
                    if user.karma >= ranks[i]['karma_min']:
                        user.write({
                            'rank_id': ranks[i]['rank'].id,
                            'next_rank_id': ranks[i - 1]['rank'].id if 0 < i else False
                        })
                        break
            if old_rank != user.rank_id:
                user._rank_changed()

    def _recompute_rank_bulk(self):
        """
            Compute rank of each user by rank.
            For each rank, check which users need to be ranked

        """
        ranks = [{'rank': rank, 'karma_min': rank.karma_min} for rank in
                 self.env['gamification.karma.rank'].search([], order="karma_min DESC")]

        users_todo = self

        next_rank_id = False
        # wtf, next_rank_id should be a related on rank_id.next_rank_id and life might get easier.
        # And we only need to recompute next_rank_id on write with min_karma or in the create on rank model.
        for r in ranks:
            rank_id = r['rank'].id
            dom = [
                ('karma', '>=', r['karma_min']),
                ('id', 'in', users_todo.ids),
                '|',  # noqa
                    '|', ('rank_id', '!=', rank_id), ('rank_id', '=', False),
                    '|', ('next_rank_id', '!=', next_rank_id), ('next_rank_id', '=', False if next_rank_id else -1),
            ]
            users = self.env['res.users'].search(dom)
            if users:
                users_to_notify = self.env['res.users'].search([
                    ('karma', '>=', r['karma_min']),
                    '|', ('rank_id', '!=', rank_id), ('rank_id', '=', False),
                    ('id', 'in', users.ids),
                ])
                users.write({
                    'rank_id': rank_id,
                    'next_rank_id': next_rank_id,
                })
                users_to_notify._rank_changed()
                users_todo -= users

            nothing_to_do_users = self.env['res.users'].search([
                ('karma', '>=', r['karma_min']),
                '|', ('rank_id', '=', rank_id), ('next_rank_id', '=', next_rank_id),
                ('id', 'in', users_todo.ids),
            ])
            users_todo -= nothing_to_do_users
            next_rank_id = r['rank'].id

        if ranks:
            lower_rank = ranks[-1]['rank']
            users = self.env['res.users'].search([
                ('karma', '>=', 0),
                ('karma', '<', lower_rank.karma_min),
                '|', ('rank_id', '!=', False), ('next_rank_id', '!=', lower_rank.id),
                ('id', 'in', users_todo.ids),
            ])
            if users:
                users.write({
                    'rank_id': False,
                    'next_rank_id': lower_rank.id,
                })

    def _get_next_rank(self):
        """ For fresh users with 0 karma that don't have a rank_id and next_rank_id yet
        this method returns the first karma rank (by karma ascending). This acts as a
        default value in related views.

        TDE FIXME in post-12.4: make next_rank_id a non-stored computed field correctly computed """

        if self.next_rank_id:
            return self.next_rank_id
        else:
            domain = [('karma_min', '>', self.rank_id.karma_min)] if self.rank_id else []
            return self.env['gamification.karma.rank'].search(domain, order="karma_min ASC", limit=1)

    def get_gamification_redirection_data(self):
        """
        Hook for other modules to add redirect button(s) in new rank reached mail
        Must return a list of dictionnary including url and label.
        E.g. return [{'url': '/forum', label: 'Go to Forum'}]
        """
        self.ensure_one()
        return []

    def action_karma_report(self):
        self.ensure_one()

        return {
            'name': _('Karma Updates'),
            'res_model': 'gamification.karma.tracking',
            'target': 'current',
            'type': 'ir.actions.act_window',
            'view_mode': 'list',
            'context': {
                'default_user_id': self.id,
                'search_default_user_id': self.id,
            },
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import gamification_badge
from . import gamification_badge_user
from . import gamification_challenge
from . import gamification_challenge_line
from . import gamification_goal
from . import gamification_goal_definition
from . import gamification_karma_rank
from . import gamification_karma_tracking
from . import res_users

```

## File: security\gamification_security.xml

```xml
<odoo noupdate="1">

        <record id="goal_user_visibility" model="ir.rule">
            <field name="name">User can only see his/her goals or goal from the same challenge in board visibility</field>
            <field name="model_id" ref="model_gamification_goal"/>
            <field name="groups" eval="[(4, ref('base.group_user')), (4, ref('base.group_portal'))]"/>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="True"/>
            <field name="perm_create" eval="False"/>
            <field name="perm_unlink" eval="False"/>
            <field name="domain_force">[
                '|',
                    ('user_id','=',user.id),
                    '&amp;',
                        ('challenge_id.user_ids','in',user.id),
                        ('challenge_id.visibility_mode','=','ranking')]</field>
        </record>

        <record id="goal_gamification_manager_visibility" model="ir.rule">
            <field name="name">Manager can see any goal</field>
            <field name="model_id" ref="model_gamification_goal"/>
            <field name="groups" eval="[(4, ref('base.group_erp_manager'))]"/>
            <field name="perm_read" eval="True"/>
            <field name="perm_write" eval="True"/>
            <field name="perm_create" eval="False"/>
            <field name="perm_unlink" eval="False"/>
            <field name="domain_force">[(1, '=', 1)]</field>
        </record>

        <record id="goal_global_multicompany" model="ir.rule">
            <field name="name">Multicompany rule on challenges</field>
            <field name="model_id" ref="model_gamification_goal"/>
            <field name="domain_force">[('user_id.company_id', 'in', company_ids)]</field>
        </record>

</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink

goal_employee,"Goal Employee",model_gamification_goal,base.group_user,1,1,0,0
goal_manager,"Goal Manager",model_gamification_goal,base.group_erp_manager,1,1,1,1
goal_portal,"Goal Portal",gamification.model_gamification_goal,base.group_portal,1,1,0,0

goal_definition_employee,"Goal Definition Employee",model_gamification_goal_definition,base.group_user,1,0,0,0
goal_definition_manager,"Goal Definition Manager",model_gamification_goal_definition,base.group_erp_manager,1,1,1,1
goal_definition_portal,"Goal Definition Portal",gamification.model_gamification_goal_definition,base.group_portal,1,0,0,0

challenge_employee,"Goal Challenge Employee",model_gamification_challenge,base.group_user,1,0,0,0
challenge_manager,"Goal Challenge Manager",model_gamification_challenge,base.group_erp_manager,1,1,1,1
challenge_portal,"Goal Challenge Portal",gamification.model_gamification_challenge,base.group_portal,1,0,0,0

challenge_line_employee,"Challenge Line Employee",model_gamification_challenge_line,base.group_user,1,0,0,0
challenge_line_manager,"Challenge Line Manager",model_gamification_challenge_line,base.group_erp_manager,1,1,1,1
challenge_line_portal,"Challenge Line Portal",gamification.model_gamification_challenge_line,base.group_portal,1,0,0,0

badge_employee,"Badge Employee",model_gamification_badge,base.group_user,1,0,0,0
badge_manager,"Badge Manager",model_gamification_badge,base.group_erp_manager,1,1,1,1
badge_portal,"Badge Portal",gamification.model_gamification_badge,base.group_portal,1,0,0,0
badge_public,"Badge Public",gamification.model_gamification_badge,base.group_public,1,0,0,0

badge_user_employee,"Badge-user Employee",model_gamification_badge_user,base.group_user,1,1,1,0
badge_user_manager,"Badge-user Manager",model_gamification_badge_user,base.group_erp_manager,1,1,1,1
badge_user_portal,"Badge-user Portal",gamification.model_gamification_badge_user,base.group_portal,1,1,1,0
badge_user_public,"Badge-user Public",gamification.model_gamification_badge_user,base.group_public,1,0,0,0

gamification_karma_rank_access_public,gamification.karma.rank.access.all,gamification.model_gamification_karma_rank,base.group_public,1,0,0,0
gamification_karma_rank_access_portal,gamification.karma.rank.access.all,gamification.model_gamification_karma_rank,base.group_portal,1,0,0,0
gamification_karma_rank_access_employee,gamification.karma.rank.access.all,gamification.model_gamification_karma_rank,base.group_user,1,0,0,0
gamification_karma_rank_access_user_manager,gamification.karma.rank.access.user.manager,gamification.model_gamification_karma_rank,base.group_system,1,1,1,1
gamification_karma_tracking_access_all,gamification.karma.tracking.access.all,gamification.model_gamification_karma_tracking,,0,0,0,0
gamification_karma_tracking_access_user_manager,gamification.karma.tracking.access.user.manager,gamification.model_gamification_karma_tracking,base.group_system,1,1,1,1

access_gamification_goal_wizard,access.gamification.goal.wizard,model_gamification_goal_wizard,base.group_user,1,1,1,0
access_gamification_badge_user_wizard,access.gamification.badge.user.wizard,model_gamification_badge_user_wizard,base.group_user,1,1,1,0

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><g clip-path="url(#o_icon_gamification__a)"><path d="M42 33c0 9.389-7.611 17-17 17S8 42.39 8 33s7.611-17 17-17 17 7.611 17 17Z" fill="#FBB945"/><path d="M33.497 18.272 25.001 33l-8.497-14.728A16.922 16.922 0 0 1 25 16c3.095 0 5.997.827 8.496 2.272Z" fill="#F78613"/><path d="M8 33c0 9.389 7.611 17 17 17V16c-9.389 0-17 7.611-17 17Z" fill="#F78613"/><path d="M16.62 0h8.381v16c-3.095 0-5.997.827-8.497 2.272L11.207 9.09a4 4 0 0 1 .08-4.131l1.95-3.092A4 4 0 0 1 16.62 0Z" fill="#962B48"/><path d="M25 16c-3.095 0-5.996.827-8.496 2.272L25 33V16Z" fill="#FBB945"/><path d="M25 16c3.095 0 5.997.827 8.496 2.272l5.298-9.182a4 4 0 0 0-.08-4.131l-1.949-3.092A4 4 0 0 0 33.381 0H25v16Z" fill="#985184"/></g><defs><clipPath id="o_icon_gamification__a"><path fill="#fff" d="M0 0h50v50H0z"/></clipPath></defs></svg>

```

## File: static\img\rank_bachelor_badge.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="300" height="300"><g fill="none"><circle cx="150" cy="150" r="150" fill="#FFF"/><path fill="#965E3F" d="M150 300C67.157 300 0 232.843 0 150S67.157 0 150 0s150 67.157 150 150-67.157 150-150 150zm0-9.375c77.665 0 140.625-62.96 140.625-140.625 0-77.665-62.96-140.625-140.625-140.625C72.335 9.375 9.375 72.335 9.375 150c0 77.665 62.96 140.625 140.625 140.625zm0-14.063C80.101 276.563 23.437 219.9 23.437 150S80.102 23.437 150 23.437 276.563 80.102 276.563 150 219.899 276.563 150 276.563z"/><path fill="#FFF" d="M169.435 140.543c12 1.957 21.195 6.424 27.587 13.403 6.391 6.978 9.587 15.62 9.587 25.924 0 7.956-2.087 15.293-6.261 22.01-4.174 6.718-10.533 12.098-19.076 16.142-8.544 4.043-19.011 6.065-31.402 6.065-9.653 0-19.142-1.272-28.468-3.815-9.326-2.544-17.25-6.163-23.772-10.859l12.327-24.26c5.217 3.912 11.25 6.945 18.097 9.097a69.746 69.746 0 0 0 21.033 3.228c7.957 0 14.217-1.532 18.783-4.598 4.565-3.065 6.847-7.402 6.847-13.01 0-11.218-8.543-16.827-25.63-16.827h-14.478V142.11l28.174-31.892h-58.305V84.783h95.87v20.543l-30.913 35.217z"/></g></svg>
```

## File: static\img\rank_doctor_badge.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="300" height="300"><g fill="none"><circle cx="150" cy="150" r="150" fill="#FFF"/><path fill="#CB9600" d="M150 300C67.157 300 0 232.843 0 150S67.157 0 150 0s150 67.157 150 150-67.157 150-150 150zm0-9.375c77.665 0 140.625-62.96 140.625-140.625 0-77.665-62.96-140.625-140.625-140.625C72.335 9.375 9.375 72.335 9.375 150c0 77.665 62.96 140.625 140.625 140.625zm0-14.063C80.101 276.563 23.437 219.9 23.437 150S80.102 23.437 150 23.437 276.563 80.102 276.563 150 219.899 276.563 150 276.563z"/><path fill="#FFF" d="M171.326 84.783L171.326 221.739 139.63 221.739 139.63 110.217 112.239 110.217 112.239 84.783z"/></g></svg>
```

## File: static\img\rank_master_badge.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="300" height="300"><g fill="none"><circle cx="150" cy="150" r="150" fill="#FFF"/><path fill="#AD7624" d="M150 300C67.157 300 0 232.843 0 150S67.157 0 150 0s150 67.157 150 150-67.157 150-150 150zm0-9.375c77.665 0 140.625-62.96 140.625-140.625 0-77.665-62.96-140.625-140.625-140.625C72.335 9.375 9.375 72.335 9.375 150c0 77.665 62.96 140.625 140.625 140.625zm0-14.063C80.101 276.563 23.437 219.9 23.437 150S80.102 23.437 150 23.437 276.563 80.102 276.563 150 219.899 276.563 150 276.563z"/><path fill="#FFF" d="M207.783 189.391v25.826h-103.5v-20.543l52.826-49.891c5.608-5.348 9.391-9.946 11.348-13.794 1.956-3.848 2.934-7.663 2.934-11.446 0-5.478-1.858-9.684-5.576-12.62-3.717-2.934-9.163-4.401-16.337-4.401-6 0-11.413 1.141-16.239 3.424-4.826 2.282-8.87 5.706-12.13 10.271l-23.087-14.87c5.348-7.956 12.717-14.184 22.108-18.684 9.392-4.5 20.153-6.75 32.283-6.75 10.174 0 19.076 1.663 26.707 4.99 7.63 3.325 13.565 8.02 17.804 14.086 4.24 6.065 6.359 13.207 6.359 21.424 0 7.435-1.566 14.413-4.696 20.935-3.13 6.522-9.196 13.956-18.196 22.304l-31.5 29.74h58.892z"/></g></svg>
```

## File: static\img\rank_newbie_badge.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="300" height="300"><g fill="none"><circle cx="150" cy="150" r="150" fill="#FFF"/><path fill="#6B3275" d="M150 300C67.157 300 0 232.843 0 150S67.157 0 150 0s150 67.157 150 150-67.157 150-150 150zm0-9.375c77.665 0 140.625-62.96 140.625-140.625 0-77.665-62.96-140.625-140.625-140.625C72.335 9.375 9.375 72.335 9.375 150c0 77.665 62.96 140.625 140.625 140.625zm0-14.063C80.101 276.563 23.437 219.9 23.437 150S80.102 23.437 150 23.437 276.563 80.102 276.563 150 219.899 276.563 150 276.563z"/><path fill="#FFF" d="M145.891 129.717c19.305 0 33.555 3.848 42.75 11.544 9.196 7.696 13.794 18 13.794 30.913 0 8.348-2.087 15.946-6.261 22.793-4.174 6.848-10.533 12.326-19.076 16.435-8.544 4.109-19.076 6.163-31.598 6.163-9.652 0-19.141-1.272-28.467-3.815-9.327-2.543-17.25-6.163-23.772-10.859l12.522-24.26c5.217 3.912 11.217 6.945 18 9.097a68.807 68.807 0 0 0 20.934 3.229c7.957 0 14.218-1.566 18.783-4.696 4.565-3.13 6.848-7.5 6.848-13.109 0-5.87-2.38-10.304-7.141-13.304s-13.011-4.5-24.75-4.5h-35.022l7.043-77.087h83.544v25.435h-57.13l-2.153 26.021h11.152z"/></g></svg>
```

## File: static\img\rank_student_badge.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="300" height="300"><g fill="none"><circle cx="150" cy="150" r="150" fill="#FFF"/><path fill="#7F465B" d="M150 300C67.157 300 0 232.843 0 150S67.157 0 150 0s150 67.157 150 150-67.157 150-150 150zm0-9.375c77.665 0 140.625-62.96 140.625-140.625 0-77.665-62.96-140.625-140.625-140.625C72.335 9.375 9.375 72.335 9.375 150c0 77.665 62.96 140.625 140.625 140.625zm0-14.063C80.101 276.563 23.437 219.9 23.437 150S80.102 23.437 150 23.437 276.563 80.102 276.563 150 219.899 276.563 150 276.563z"/><path fill="#FFF" d="M204.783 186.457L182.478 186.457 182.478 215.217 151.565 215.217 151.565 186.457 77.804 186.457 77.804 165.13 142.37 78.261 175.63 78.261 115.761 160.63 152.543 160.63 152.543 135 182.478 135 182.478 160.63 204.783 160.63z"/></g></svg>
```

## File: views\gamification_badge_user_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="badge_user_kanban_view" model="ir.ui.view" >
        <field name="name">Badge User Kanban View</field>
        <field name="model">gamification.badge.user</field>
        <field name="arch" type="xml">
            <kanban action="action_open_badge" type="object">
                <field name="create_date"/>
                <templates>
                    <t t-name="card" class="row g-0">
                        <aside class="col-2">
                            <field name="badge_id" widget="image" options="{'preview_image': 'image_1024'}" t-att-alt="record.badge_name.value" />
                        </aside>
                        <main class="col ps-2">
                            <field class="fw-bold fs-5" type="open" name="badge_name"/>
                            <field name="comment"/>
                            <p>Granted by <field name="create_uid" /> the <t t-esc="luxon.DateTime.fromISO(record.create_date.raw_value).toFormat('D')" /></p>
                        </main>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

</data></odoo>

```

## File: views\gamification_badge_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Badge views -->
    <record id="gamification_badge_view_search" model="ir.ui.view">
        <field name="name">gamification.badge.view.search</field>
        <field name="model">gamification.badge</field>
        <field name="arch" type="xml">
            <search string="Search Badge">
                <field name="name"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="badge_list_action" model="ir.actions.act_window">
        <field name="name">Badges</field>
        <field name="res_model">gamification.badge</field>
        <field name="view_mode">kanban,list,form</field>
        <field name="search_view_id" ref="gamification_badge_view_search"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new badge
            </p><p>
                A badge is a symbolic token granted to a user as a sign of reward.
                It can be deserved automatically when some conditions are met or manually by users.
                Some badges are harder than others to get with specific conditions.
            </p>
        </field>
    </record>

    
    <record id="badge_list_view" model="ir.ui.view">
        <field name="name">Badge List</field>
        <field name="model">gamification.badge</field>
        <field name="arch" type="xml">
            <list string="Badge List">
                <field name="name"/>
                <field name="granted_count"/>
                <field name="stat_this_month"/>
                <field name="stat_my"/>
                <field name="rule_auth"/>
            </list>
        </field>
    </record>

    <record id="badge_form_view" model="ir.ui.view">
        <field name="name">Badge Form</field>
        <field name="model">gamification.badge</field>
        <field name="arch" type="xml">
            <form string="Badge">
                <header>
                    <button string="Grant this Badge" type="action" name="%(action_grant_wizard)d" class="oe_highlight" invisible="remaining_sending == 0" />
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box"/>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" invisible="active"/>
                    <field name="image_1920" widget="image" class="oe_avatar" options="{'preview_image': 'image_128'}"/>
                    <div class="oe_title">
                        <label for="name"/>
                        <h1>
                            <field name="name" placeholder="e.g. Problem Solver"/>
                        </h1>
                    </div>
                    <group>
                        <field name="description" nolabel="1" placeholder="Badge Description" colspan="2"/>
                    </group>
                    <group string="Granting">
                        <div class="oe_grey" colspan="2">
                            Security rules to define who is allowed to manually grant badges. Not enforced for administrator.
                        </div>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="rule_auth" widget="radio"/>
                            <field name="rule_auth_user_ids" invisible="rule_auth != 'users'" widget="many2many_tags" />
                            <field name="rule_auth_badge_ids" invisible="rule_auth != 'having'" widget="many2many_tags" />
                            <field name="rule_max" invisible="rule_auth == 'nobody'" />
                            <field name="rule_max_number" invisible="not rule_max or rule_auth == 'nobody'"/>
                            <label for="stat_my_monthly_sending"/>
                            <div>
                                <field name="stat_my_monthly_sending" invisible="rule_auth == 'nobody'" />
                                <div invisible="remaining_sending == -1" class="oe_grey">
                                    You can still grant <field name="remaining_sending" class="oe_inline"/> badges this month
                                </div>
                                <div invisible="remaining_sending != -1" class="oe_grey">
                                    No monthly sending limit
                                </div>
                            </div>
                        </group>
                    </group>
                    <group string="Rewards for challenges">
                        <field name="challenge_ids" mode="kanban" widget="many2many" nolabel="1" context="{'default_reward_id': id}" colspan="2"/>
                        <field name="level" colspan="2"/>
                    </group>
                    <group id="badge_statistics" string="Statistics">
                        <group>
                            <field name="granted_count"/>
                            <field name="stat_this_month"/>
                            <field name="granted_users_count"/>
                        </group>
                        <group>
                            <field name="stat_my"/>
                            <field name="stat_my_this_month"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>


    <record id="badge_kanban_view" model="ir.ui.view" >
        <field name="name">Badge Kanban View</field>
        <field name="model">gamification.badge</field>
        <field name="arch" type="xml">
            <kanban>
                <field name="remaining_sending" />
                <templates>
                    <t t-name="card" class="row g-0">
                        <aside class="col-2">
                            <field name="image_1024" widget="image" t-att-alt="record.name.value"/>
                        </aside>
                        <main class="col ps-2">
                            <span>
                                <field class="fw-bold fs-5 me-1" name="name"/>
                                <t t-if="record.remaining_sending.value != -1">
                                    <field name="stat_my_monthly_sending"/>/<field name="rule_max_number"/>
                                </t>
                                <t t-else="">
                                    <field name="stat_my_monthly_sending"/>/∞
                                </t>
                            </span>
                            <div t-if="record.remaining_sending.value == 0">Can not grant</div>
                            <div>
                                <field class="fw-bold" name="granted_count"/> granted,
                                <field class="fw-bold" name="stat_this_month"/> this month
                            </div>
                            <div>
                                <field class="fst-italic" name="description"/>
                                <field name="unique_owner_ids" widget="many2many_avatar_user"/>
                            </div>
                            <footer t-if="record.remaining_sending.value != 0 and !selection_mode">
                                <button type="action" name="%(action_grant_wizard)d" class="btn btn-primary ms-auto">Grant</button>
                            </footer>
                        </main>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>
</odoo>

```

## File: views\gamification_challenge_line_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="challenge_line_list_view" model="ir.ui.view">
        <field name="name">Challenge line list</field>
        <field name="model">gamification.challenge.line</field>
        <field name="arch" type="xml">
            <list string="Challenge Lines" >
                <field name="definition_id"/>
                <field name="target_goal" string="Target"/>
            </list>
        </field>
    </record>

</data></odoo>

```

## File: views\gamification_challenge_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="challenge_list_view" model="ir.ui.view">
        <field name="name">Challenges List</field>
        <field name="model">gamification.challenge</field>
        <field name="arch" type="xml">
            <list string="Challenges" decoration-info="state == 'draft'" decoration-muted="state == 'done'">
                <field name="name"/>
                <field name="period"/>
                <field name="manager_id"/>
                <field name="state"/>
            </list>
        </field>
    </record>

    <record id="challenge_form_view" model="ir.ui.view">
        <field name="name">Challenge Form</field>
        <field name="model">gamification.challenge</field>
        <field name="arch" type="xml">
            <form string="Challenge">
                <header>
                    <button string="Start Challenge" type="object" name="action_start" invisible="state != 'draft'" class="oe_highlight"/>
                    <button string="Refresh Challenge" type="object" name="action_check" invisible="state != 'inprogress'"/>
                    <button string="Send Report" type="object" name="action_report_progress" invisible="state not in ('inprogress', 'done')" groups="base.group_no_one"/>
                    <field name="state" widget="statusbar" options="{'clickable': '1'}"/>
                </header>
                <sheet>
                    <!-- action buttons -->
                    <div class="oe_button_box" name="button_box">
                        <button type="object"
                                name="action_view_users"
                                class="oe_stat_button"
                                icon="fa-users">
                            <field name="user_count" string="Participants" widget="statinfo"/>
                        </button>
                        <button type="action"
                                name="%(goals_from_challenge_act)d"
                                class="oe_stat_button"
                                icon="fa-gift"
                                invisible="state == 'draft'">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_text">Related Goals</span>
                            </div>
                        </button>
                    </div>
                    <div class="oe_title">
                        <label for="name"/>
                        <h1>
                            <field name="name" placeholder="e.g. Monthly Sales Objectives"/>
                        </h1>
                        <label for="user_domain" string="Assign Challenge to"/>
                        <div>
                            <field name="user_domain" widget="domain" options="{'model': 'res.users'}" />
                        </div>
                    </div>
                    <group>
                        <group>
                            <field name="period" readonly="state != 'draft'"/>
                            <field name="visibility_mode" widget="radio" colspan="1" />
                        </group>
                        <group>
                            <field name="manager_id"/>
                            <field name="start_date" readonly="state != 'draft'"/>
                            <field name="end_date" readonly="state != 'draft'"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Goals" name="goals">
                            <field name="line_ids" nolabel="1" colspan="4">
                                <list string="Line List" editable="bottom" >
                                    <field name="sequence" widget="handle"/>
                                    <field name="definition_id" />
                                    <field name="condition"/>
                                    <field name="target_goal" string="Target"/>
                                    <field name="definition_full_suffix"/>
                                </list>
                            </field>
                            <field name="description" placeholder="Describe the challenge: what is does, who it targets, why it matters..."/>
                        </page>
                        <page string="Reward" name="reward">
                            <group>
                                <field name="reward_id" required="reward_realtime" />
                                <field name="reward_first_id" />
                                <field name="reward_second_id" invisible="not reward_first_id" />
                                <field name="reward_third_id" invisible="not reward_first_id or not reward_second_id" />
                                <field name="reward_failure" invisible="not reward_first_id" />
                                <field name="reward_realtime" />
                            </group>
                            <div class="oe_grey">
                                <p>Badges are granted when a challenge is finished. This is either at the end of a running period (eg: end of the month for a monthly challenge), at the end date of a challenge (if no periodicity is set) or when the challenge is manually closed.</p>
                            </div>
                        </page>
                        <page string="Advanced Options" name="advanced_options">
                            <group string="Subscriptions">
                                <field name="invited_user_ids" widget="many2many_tags" />
                            </group>
                            <group string="Notification Messages">
                                <div class="oe_grey" colspan="4">
                                    <p>Depending on the Display mode, reports will be individual or shared.</p>
                                </div>
                                <group colspan="4">
                                    <field name="report_message_frequency"/>
                                    <field name="report_template_id" invisible="report_message_frequency == 'never'" />
                                    <field name="report_message_group_id" invisible="report_message_frequency == 'never'" />
                                </group>
                            </group>
                            <group string="Reminders for Manual Goals">
                                <label for="remind_update_delay" />
                                <div>
                                    <field name="remind_update_delay" class="oe_inline"/> days
                                </div>
                            </group>
                            <group string="Category" groups="base.group_no_one">
                                <field name="challenge_category" widget="radio" />
                            </group>
                        </page>
                    </notebook>

                </sheet>
                <chatter/>
            </form>
        </field>
    </record>

    <record model="ir.ui.view" id="view_challenge_kanban">
        <field name="name">Challenge Kanban</field>
        <field name="model">gamification.challenge</field>
        <field name="arch" type="xml">
            <kanban string="Challenges">
                <field name="line_ids"/>
                <templates>
                    <t t-name="card" class="fw-bold">
                        <field class="fs-5" name="name"/>
                        <a type="action" name="%(goals_from_challenge_act)d" class="me-2" tabindex="-1">
                            <t t-esc="record.line_ids.raw_value.length"/> Goals
                        </a>
                        <a type="object" name="action_view_users" class="me-2" tabindex="-1">
                            <field name="user_count"/> Participants
                        </a>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="challenge_list_action" model="ir.actions.act_window">
        <field name="name">Challenges</field>
        <field name="res_model">gamification.challenge</field>
        <field name="view_mode">kanban,list</field>
        <field name="context">{'search_default_inprogress':True, 'default_inprogress':True}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new challenge
            </p><p>
                Assign a list of goals to chosen users to evaluate them.
                The challenge can use a period (weekly, monthly...) for automatic creation of goals.
                The goals are created for the specified users or member of the group.
            </p>
        </field>
    </record>
    <!-- Specify form view ID to avoid selecting view_challenge_wizard -->
    <record id="challenge_list_action_view1" model="ir.actions.act_window.view">
        <field eval="1" name="sequence"/>
        <field name="view_mode">kanban</field>
        <field name="act_window_id" ref="challenge_list_action"/>
        <field name="view_id" ref="view_challenge_kanban"/>
    </record>
    <record id="challenge_list_action_view2" model="ir.actions.act_window.view">
        <field eval="10" name="sequence"/>
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="challenge_list_action"/>
        <field name="view_id" ref="challenge_form_view"/>
    </record>

    <record id="challenge_search_view" model="ir.ui.view">
        <field name="name">Challenge Search</field>
        <field name="model">gamification.challenge</field>
        <field name="arch" type="xml">
            <search string="Search Challenges">
                <filter name="inprogress" string="Running Challenges"
                    domain="[('state', '=', 'inprogress')]"/>
                <filter name="hr_challenges" string="HR Challenges"
                    domain="[('challenge_category', '=', 'hr')]"/>
                <field name="name"/>
                <group expand="0" string="Group By">
                    <filter string="State" name="state" domain="[]" context="{'group_by':'state'}"/>
                    <filter string="Period" name="period" domain="[]" context="{'group_by':'period'}"/>
                </group>
            </search>
        </field>
    </record>
</odoo>

```

## File: views\gamification_goal_definition_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="goal_definition_list_action" model="ir.actions.act_window">
        <field name="name">Goal Definitions</field>
        <field name="res_model">gamification.goal.definition</field>
        <field name="view_mode">list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new goal definition
            </p><p>
                A goal definition is a technical specification of a condition to reach.
                The dates, values to reach or users are defined in goal instance.
            </p>
        </field>
    </record>

    <record id="goal_definition_list_view" model="ir.ui.view">
        <field name="name">Goal Definitions List</field>
        <field name="model">gamification.goal.definition</field>
        <field name="arch" type="xml">
            <list string="Goal Definitions">
                <field name="name"/>
                <field name="computation_mode"/>
            </list>
        </field>
    </record>


    <record id="goal_definition_form_view" model="ir.ui.view">
        <field name="name">Goal Definitions Form</field>
        <field name="model">gamification.goal.definition</field>
        <field name="arch" type="xml">
            <form string="Goal definitions">
                <sheet>
                        <label for="name"/>
                        <h1>
                            <field name="name" placeholder="e.g. Get started" class="w-100"/>
                        </h1>
                        <label for="description"/>
                        <div>
                            <field name="description" placeholder="e.g. Register to the platform" class="w-100"/>
                        </div>

                        <group string="How is the goal computed?" name="compute_details">

                            <field widget="radio" name="computation_mode"/>

                            <!-- Hide the fields below if manually -->
                            <field name="model_id" class="oe_inline"
                                invisible="computation_mode not in ('sum', 'count')"
                                required="computation_mode in ('sum', 'count')"/>
                            <field name="model_inherited_ids" invisible="1"/>
                            <field name="field_id" class="oe_inline" options="{'no_create': True}"
                                invisible="computation_mode != 'sum'"
                                required="computation_mode == 'sum'"/>
                            <field name="field_date_id" class="oe_inline" invisible="computation_mode not in ('sum', 'count')"/>
                            <field name="domain" invisible="computation_mode not in ('sum', 'count')" required="computation_mode in ('sum', 'count')" class="oe_inline"/>
                            <field name="compute_code" invisible="computation_mode != 'python'" required="computation_mode == 'python'"/>
                            <field name="condition" widget="radio"/>
                        </group>
                        <group string="Optimisation" name="optimisation" invisible="computation_mode not in ('sum', 'count')">
                            <field name="batch_mode" />
                            <div colspan="2">In batch mode, the domain is evaluated globally. If enabled, do not use keyword 'user' in above filter domain.</div>
                            <field name="batch_distinctive_field" invisible="not batch_mode" required="batch_mode"
                                domain="[('model_id', '=', model_id)]" class="oe_inline" />
                            <field name="batch_user_expression" invisible="not batch_mode" required="batch_mode" class="oe_inline"
                                placeholder="e.g. user.partner_id.id"/>
                        </group>
                        <group string="Formatting Options" name="format_options">
                            <field name="display_mode" widget="radio" />
                            <field name="suffix" placeholder="e.g. days" class="oe_inline"/>
                            <field name="monetary"/>
                        </group>
                        <group string="Clickable Goals" name="clickable_goals" invisible="computation_mode == 'manually'">
                            <field name="action_id"  class="oe_inline"/>
                            <field name="res_id_field"  invisible="not action_id" class="oe_inline"/>
                        </group>

                </sheet>
            </form>
        </field>
    </record>

    <record id="goal_definition_search_view" model="ir.ui.view">
        <field name="name">Goal Definition Search</field>
        <field name="model">gamification.goal.definition</field>
        <field name="arch" type="xml">
            <search string="Search Goal Definitions">
                <field name="name"/>
                <field name="model_id"/>
                <field name="field_id"/>
                <group expand="0" string="Group By">
                    <filter string="Model" name="model" domain="[]" context="{'group_by':'model_id'}"/>
                    <filter string="Computation Mode" name="computationmode" domain="[]" context="{'group_by':'computation_mode'}"/>
                </group>
            </search>
        </field>
    </record>
</odoo>

```

## File: views\gamification_goal_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Goal views -->
    <record id="goal_list_action" model="ir.actions.act_window">
        <field name="name">Goals</field>
        <field name="res_model">gamification.goal</field>
        <field name="view_mode">list,form,kanban</field>
        <field name="context">{'search_default_group_by_user': True, 'search_default_group_by_definition': True}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new goal
            </p><p>
                A goal is defined by a user and a goal definition.
                Goals can be created automatically by using challenges.
            </p>
        </field>
    </record>

    <record id="goals_from_challenge_act" model="ir.actions.act_window">
        <field name="res_model">gamification.goal</field>
        <field name="name">Related Goals</field>
        <field name="view_mode">kanban,list,form</field>
        <field name="context">{'search_default_group_by_definition': True, 'search_default_inprogress': True, 'search_default_challenge_id': active_id, 'default_challenge_id': active_id}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_empty_folder">
                No goal found
          </p><p>
            There is no goal associated to this challenge matching your search.
            Make sure that your challenge is active and assigned to at least one user.
          </p>
        </field>
    </record>

    <record id="goal_list_view" model="ir.ui.view">
        <field name="name">Goal List</field>
        <field name="model">gamification.goal</field>
        <field name="arch" type="xml">
            <list string="Goal List" decoration-danger="state == 'failed'" decoration-success="state == 'reached'" decoration-muted="state == 'canceled'" create="false">
                <field name="definition_id" column_invisible="True" />
                <field name="user_id" column_invisible="True" />
                <field name="start_date"/>
                <field name="end_date"/>
                <field name="current"/>
                <field name="target_goal"/>
                <field name="completeness" widget="progressbar"/>
                <field name="state" column_invisible="True"/>
                <field name="line_id" column_invisible="True"/>
            </list>
        </field>
    </record>

    <record id="goal_form_view" model="ir.ui.view">
        <field name="name">Goal Form</field>
        <field name="model">gamification.goal</field>
        <field name="arch" type="xml">
            <form string="Goal" create="false">
                <header>
                    <button string="Start goal" type="object" name="action_start" invisible="state != 'draft'" class="oe_highlight"/>

                    <button string="Goal Reached" type="object" name="action_reach" invisible="state != 'inprogress'" />
                    <button string="Goal Failed" type="object" name="action_fail" invisible="state != 'inprogress'"/>
                    <button string="Reset Completion" type="object" name="action_cancel" invisible="state not in ('failed', 'reached')" groups="base.group_no_one" />
                    <field name="state" widget="statusbar" statusbar_visible="draft,inprogress,reached" />
                </header>
                <sheet>
                    <group>
                        <group string="Reference">
                            <field name="definition_id" readonly="state != 'draft'"/>
                            <field name="user_id" readonly="state != 'draft'"/>
                            <field name="challenge_id" />
                        </group>
                        <group string="Schedule">
                            <field name="start_date" readonly="state != 'draft'"/>
                            <field name="end_date" />
                            <field name="computation_mode" invisible="1"/>

                            <label for="remind_update_delay" invisible="computation_mode != 'manually'"/>
                            <div invisible="computation_mode != 'manually'">
                                <field name="remind_update_delay" class="oe_inline"/>
                                days
                            </div>
                            <field name="last_update" groups="base.group_no_one"/>
                        </group>
                        <group string="Data" colspan="4">
                            <label for="target_goal" />
                            <div>
                                <field name="target_goal" readonly="state != 'draft'" class="oe_inline"/>
                                <field name="definition_suffix" class="oe_inline"/>
                            </div>
                            <label for="current" />
                            <div>
                                <field name="current" class="oe_inline"/>
                                <button string="refresh" type="object" name="update_goal" class="oe_link" invisible="computation_mode == 'manually' or state == 'draft'" />
                                <div class="oe_grey" invisible="not definition_id">
                                    Reached when current value is <strong><field name="definition_condition" class="oe_inline"/></strong> than the target.
                                </div>
                            </div>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="goal_search_view" model="ir.ui.view">
        <field name="name">Goal Search</field>
        <field name="model">gamification.goal</field>
        <field name="arch" type="xml">
            <search string="Search Goals">
                <filter name="my" string="My Goals" domain="[('user_id', '=', uid)]"/>
                <separator/>
                <filter name="draft" string="Draft" domain="[('state', '=', 'draft')]"/>
                <filter name="inprogress" string="Running"
                    domain="[
                        '|',
                            ('state', '=', 'inprogress'),
                            '&amp;',
                                ('state', 'in', ('done', 'failed')),
                                ('end_date', '>=', context_today().strftime('%Y-%m-%d'))
                    ]"/>
                <filter name="closed" string="Done"
                    domain="[
                        ('state', 'in', ('reached', 'failed')),
                        '|',
                            ('end_date', '=', False),
                            ('end_date', '&lt;', context_today().strftime('%Y-%m-%d'))
                    ]"/>
                <separator/>

                <field name="user_id"/>
                <field name="definition_id"/>
                <field name="challenge_id"/>
                <group expand="0" string="Group By">
                    <filter name="group_by_user" string="User" domain="[]" context="{'group_by':'user_id'}"/>
                    <filter name="group_by_definition" string="Goal Definition" domain="[]" context="{'group_by':'definition_id'}"/>
                    <filter string="State" name="state" domain="[]" context="{'group_by':'state'}"/>
                    <filter string="End Date" name="enddate" domain="[]" context="{'group_by':'end_date'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="goal_kanban_view" model="ir.ui.view" >
        <field name="name">Goal Kanban View</field>
        <field name="model">gamification.goal</field>
        <field name="arch" type="xml">
            <kanban highlight_color="color" create="false">
                <field name="state"/>
                <field name="color"/>
                <field name="definition_condition"/>
                <field name="definition_suffix"/>
                <field name="definition_display"/>
                <field name="last_update"/>
                <templates>
                    <t t-name="card" class="text-center">
                        <field class="fw-bold fs-4" name="definition_id" />
                        <div class="d-flex justify-content-center mt-3">
                            <field class="o_image_24_cover me-1 rounded" name="user_id" widget="image" options="{'preview_image': 'avatar_128'}"/>
                            <field name="user_id" class="fw-bold"/>
                        </div>
                        <div class="pt-3 fs-1 fw-bolder">
                            <t t-if="record.definition_display.raw_value == 'boolean'">
                                <t t-if="record.state.raw_value=='reached'"><i role="img" class="text-success fa fa-check fa-3x" title="Goal Reached" aria-label="Goal Reached"/></t>
                                <t t-if="record.state.raw_value=='inprogress'"><i role="img" class="text-700 fa fa-clock-o fa-3x" title="Goal in Progress" aria-label="Goal in Progress"/></t>
                                <t t-if="record.state.raw_value=='failed'"><i role="img" class="text-danger fa fa-times fa-3x" title="Goal Failed" aria-label="Goal Failed"/></t>
                            </t>
                            <t t-if="record.definition_display.raw_value == 'progress'">
                                <t t-if="record.definition_condition.raw_value =='higher'">
                                    <field name="current" widget="gauge" options="{'max_field': 'target_goal', 'label_field': 'definition_suffix', 'style': 'width:160px; height: 120px;'}" />
                                </t>
                                <t t-if="record.definition_condition.raw_value != 'higher'">
                                    <field class="#{record.current.raw_value == record.target_goal.raw_value+1 ? 'text-warning' : record.current.raw_value &gt; record.target_goal.raw_value ? 'text-danger' : 'text-success'}" name="current" />
                                    <em>Target: less than <field name="target_goal"/></em>
                                </t>
                            </t>
                        </div>
                        <p>
                            <t t-if="record.start_date.value">
                                From <field name="start_date" />
                            </t>
                            <t t-if="record.end_date.value">
                                To <field name="end_date" />
                            </t>
                        </p>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>
</odoo>

```

## File: views\gamification_karma_rank_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <!-- Ranks views -->
        <record id="gamification_karma_ranks_action" model="ir.actions.act_window">
            <field name="name">Ranks</field>
            <field name="res_model">gamification.karma.rank</field>
            <field name="view_mode">list,form</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a new rank
                </p><p>
                    A rank correspond to a fixed karma level. The more you have karma, the more your rank is high.
                    This is used to quickly know which user is new or old or highly or not active.
                </p>
            </field>
        </record>

        <record id="gamification_karma_ranks_view_search" model="ir.ui.view">
            <field name="name">gamification.karma.ranks.view.search</field>
            <field name="model">gamification.karma.rank</field>
            <field name="arch" type="xml">
                <search string="Search Ranks">
                    <field name="name"/>
                    <field name="karma_min"/>
                    <field name="description"/>
                    <field name="user_ids"/>
                </search>
            </field>
        </record>

        <record id="gamification_karma_ranks_view_tree" model="ir.ui.view">
            <field name="name">gamification.karma.ranks.view.list</field>
            <field name="model">gamification.karma.rank</field>
            <field name="arch" type="xml">
                <list string="Ranks List">
                    <field name="name"/>
                    <field name="karma_min"/>
                    <field name="rank_users_count"/>
                </list>
            </field>
        </record>

        <record id="gamification_karma_rank_view_form" model="ir.ui.view">
            <field name="name">gamification.karma.rank.view.form</field>
            <field name="model">gamification.karma.rank</field>
            <field name="arch" type="xml">
                <form string="Rank">
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button type="action" class="oe_stat_button" icon="fa-users" name="%(action_current_rank_users)d">
                                <div class="o_form_field o_stat_info">
                                    <span class="o_stat_value">
                                        <field name="rank_users_count" />
                                    </span>
                                    <span class="o_stat_text">Users</span>
                                </div>
                            </button>
                        </div>
                        <field name="image_1920" widget="image" class="oe_avatar" options="{'preview_image': 'image_128'}"/>
                        <div class="oe_title">
                            <label for="name"/>
                            <h1>
                                <field name="name" placeholder="e.g. Master Chief"/>
                            </h1>
                        </div>
                        <group>
                            <field name="karma_min"/>
                            <field name="create_date" invisible="1"/>
                        </group>
                        <notebook>
                            <page string="Description" name="description">
                                <field name="description" placeholder="e.g. A Master Chief knows quite everything on the forum! You cannot beat him!"/>
                            </page>
                            <page string="Motivational" name="motivational">
                                <field name="description_motivational" placeholder="e.g. Reach this rank to gain a free mug!"/>
                            </page>
                        </notebook>
                    </sheet>
                </form>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\gamification_karma_tracking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="gamification_karma_tracking_view_search" model="ir.ui.view">
        <field name="name">gamification.karma.tracking.view.search</field>
        <field name="model">gamification.karma.tracking</field>
        <field name="arch" type="xml">
            <search string="Search Trackings">
                <field name="user_id" string="Karma Owner"/>
                <field name="tracking_date"/>
                <field name="origin_ref_model_name"/>
                <filter string="Consolidated" name="filter_consolidated"
                    domain="[('consolidated', '=', True)]"/>
                <filter string="My Karma" name="filter_user_id"
                    domain="[('user_id', '=', uid)]"/>
                <separator/>
                <filter string="Manual" name="filter_res_users"
                    domain="[('origin_ref', 'ilike', 'res.users,')]"/>
                <group string="Group By" expand="1">
                    <filter string="Granted By" name="group_by_create_uid"
                        context="{'group_by': 'create_uid'}"/>
                    <filter string="Source Type" name="group_by_origin_ref_model_name"
                        context="{'group_by': 'origin_ref_model_name'}"/>
                    <filter string="Karma Owner" name="group_by_user_id"
                        context="{'group_by': 'user_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="gamification_karma_tracking_view_tree" model="ir.ui.view">
        <field name="name">gamification.karma.tracking.view.list</field>
        <field name="model">gamification.karma.tracking</field>
        <field name="arch" type="xml">
            <list string="Trackings" editable="top" delete="0" sample="1">
                <field name="tracking_date" string="Date"/>
                <field name="origin_ref" options="{'no_create': True}" required="1"/>
                <field name="reason"/>
                <field name="user_id" widget="many2one_avatar_user" string="Karma Owner"
                     readonly="id"/>
                <field name="old_value" string="Previous Total" optional="hidden"/>
                <field name="gain" readonly="id"/>
                <field name="new_value" string="Total"  optional="show" readonly="1"/>
            </list>
        </field>
    </record>

    <record id="gamification_karma_tracking_view_form" model="ir.ui.view">
        <field name="name">gamification.karma.tracking.view.form</field>
        <field name="model">gamification.karma.tracking</field>
        <field name="arch" type="xml">
            <form string="Tracking" delete="0">
                <sheet>
                    <group>
                        <field name="user_id" widget="many2one_avatar_user" string="Karma Owner"
                            readonly="id"/>
                        <field name="tracking_date"/>
                        <field name="gain" readonly="id"/>
                        <field name="old_value"/>
                        <field name="new_value" readonly="1"/>
                        <field name="consolidated"/>
                        <field name="origin_ref" options="{'no_create': True}" required="1"/>
                        <field name="reason"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="gamification_karma_tracking_action" model="ir.actions.act_window">
        <field name="name">Karma Tracking</field>
        <field name="res_model">gamification.karma.tracking</field>
        <field name="view_mode">list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No Karma Tracking
            </p>
            <p>
                Track the sources of the users karma and monitor
            </p>
        </field>
    </record>
    </data>
</odoo>

```

## File: views\gamification_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- menus in settings - technical feature required -->
    <menuitem id="gamification_menu"
              name="Gamification Tools"
              parent="base.menu_administration"
              groups="base.group_no_one" />

    <menuitem id="gamification_challenge_menu"
              parent="gamification_menu"
              action="challenge_list_action"
              sequence="0"/>
    <menuitem id="gamification_goal_menu"
              parent="gamification_menu"
              action="goal_list_action"
              sequence="10"/>
    <menuitem id="gamification_definition_menu"
              parent="gamification_menu"
              action="goal_definition_list_action"
              sequence="20"/>
    <menuitem id="gamification_badge_menu"
              parent="gamification_menu"
              action="badge_list_action"
              sequence="30"/>
    <menuitem id="gamification_karma_ranks_menu"
              parent="gamification_menu"
              action="gamification_karma_ranks_action"
              sequence="40"/>
    <menuitem id="gamification_karma_tracking_menu"
              parent="gamification_menu"
              action="gamification_karma_tracking_action"
              sequence="50"/>
</odoo>

```

## File: views\res_users_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>

    <record id="res_users_view_form" model="ir.ui.view">
        <field name="name">res.users.view.form.inherit.gamification</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="base.view_users_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button type="object" class="oe_stat_button" groups="base.group_no_one"
                    name="action_karma_report" icon="fa-certificate">
                    <field string="Karma" name="karma" widget="statinfo"/>
                </button>
            </xpath>
        </field>
    </record>

    <!-- ACTIONS -->
    <record id="action_current_rank_users" model="ir.actions.act_window">
        <field name="name">Users</field>
        <field name="res_model">res.users</field>
        <field name="view_mode">list,form</field>
        <field name="domain">[('rank_id', '=', active_id)]</field>
    </record>

    <record id="action_new_simplified_res_users" model="ir.actions.act_window">
        <field name="name">Create User</field>
        <field name="res_model">res.users</field>
        <field name="target">current</field>
        <field name="view_id" ref="base.view_users_simple_form"/>
        <field name="context">{}</field>
        <field name="help">Create and manage users that will connect to the system. Users can be deactivated should there be a period of time during which they will/should not connect to the system. You can assign them groups in order to give them specific access to the applications they need to use in the system.</field>
    </record>

</data>
</odoo>

```

## File: wizard\grant_badge.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _, exceptions


class grant_badge_wizard(models.TransientModel):
    """ Wizard allowing to grant a badge to a user"""
    _name = 'gamification.badge.user.wizard'
    _description = 'Gamification User Badge Wizard'

    user_id = fields.Many2one("res.users", string='User', required=True)
    badge_id = fields.Many2one("gamification.badge", string='Badge', required=True)
    comment = fields.Text('Comment')

    def action_grant_badge(self):
        """Wizard action for sending a badge to a chosen user"""

        BadgeUser = self.env['gamification.badge.user']

        uid = self.env.uid
        for wiz in self:
            if uid == wiz.user_id.id:
                raise exceptions.UserError(_('You can not grant a badge to yourself.'))

            #create the badge
            BadgeUser.create({
                'user_id': wiz.user_id.id,
                'sender_id': uid,
                'badge_id': wiz.badge_id.id,
                'comment': wiz.comment,
            })._send_badge()

        return True

```

## File: wizard\grant_badge.xml

```xml
<odoo>
    <record id="view_badge_wizard_grant" model="ir.ui.view">
        <field name="name">Grant Badge User Form</field>
        <field name="model">gamification.badge.user.wizard</field>
        <field name="arch" type="xml">
            <form string="Grant Badge To">
                Who would you like to reward?
                <field name="badge_id" invisible="1"/>
                <group>
                    <field name="user_id" nolabel="1" colspan="2"/>
                    <field name="comment" nolabel="1" placeholder="Describe what they did and why it matters (will be public)" colspan="2"/>
                </group>
                <footer>
                    <button string="Grant Badge" type="object" name="action_grant_badge" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" special="cancel" data-hotkey="x" class="btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="action_grant_wizard" model="ir.actions.act_window">
        <field name="name">Grant Badge</field>
        <field name="res_model">gamification.badge.user.wizard</field>
        <field name="view_id" ref="gamification.view_badge_wizard_grant"/>
        <field name="target">new</field>
        <field name="context">{
            'default_badge_id': active_id,
            'badge_id': active_id
        }</field>
    </record>
</odoo>

```

## File: wizard\update_goal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields

class goal_manual_wizard(models.TransientModel):
    """Wizard to update a manual goal"""
    _name = 'gamification.goal.wizard'
    _description = 'Gamification Goal Wizard'

    goal_id = fields.Many2one("gamification.goal", string='Goal', required=True)
    current = fields.Float('Current')

    def action_update_current(self):
        """Wizard action for updating the current value"""
        for wiz in self:
            wiz.goal_id.write({
                'current': wiz.current,
                'goal_id': wiz.goal_id.id,
                'to_update': False,
            })
            wiz.goal_id.update_goal()

        return False

```

## File: wizard\update_goal.xml

```xml
<odoo>
    <record id="view_goal_wizard_update_current" model="ir.ui.view">
        <field name="name">Update the current value of the Goal</field>
        <field name="model">gamification.goal.wizard</field>
        <field name="arch" type="xml">
            <form string="Grant Badge To">
                Set the current value you have reached for this goal
                <group>
                    <field name="goal_id" invisible="1"/>
                    <field name="current" />
                </group>
                <footer>
                    <button string="Update" type="object" name="action_update_current" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" special="cancel" data-hotkey="x" class="btn-secondary"/>
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

from . import update_goal
from . import grant_badge

```

