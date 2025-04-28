# Odoo Module: crm

Category: Sales/CRM

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import populate
from . import report
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'CRM',
    'version': '1.8',
    'category': 'Sales/CRM',
    'sequence': 15,
    'summary': 'Track leads and close opportunities',
    'website': 'https://www.odoo.com/app/crm',
    'depends': [
        'base_setup',
        'sales_team',
        'mail',
        'calendar',
        'resource',
        'utm',
        'web_tour',
        'contacts',
        'digest',
        'phone_validation',
    ],
    'data': [
        'security/crm_security.xml',
        'security/ir.model.access.csv',

        'data/crm_lead_merge_template.xml',
        'data/crm_lead_prediction_data.xml',
        'data/crm_lost_reason_data.xml',
        'data/crm_stage_data.xml',
        'data/crm_team_data.xml',
        'data/digest_data.xml',
        'data/ir_action_data.xml',
        'data/ir_cron_data.xml',
        'data/mail_message_subtype_data.xml',
        'data/crm_recurring_plan_data.xml',

        'wizard/crm_lead_lost_views.xml',
        'wizard/crm_lead_to_opportunity_views.xml',
        'wizard/crm_lead_to_opportunity_mass_views.xml',
        'wizard/crm_merge_opportunities_views.xml',
        'wizard/crm_lead_pls_update_views.xml',

        'views/calendar_views.xml',
        'views/crm_recurring_plan_views.xml',
        'views/crm_lost_reason_views.xml',
        'views/crm_stage_views.xml',
        'views/crm_lead_views.xml',
        'views/crm_team_member_views.xml',
        'views/digest_views.xml',
        'views/mail_activity_plan_views.xml',
        'views/mail_activity_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
        'views/utm_campaign_views.xml',
        'report/crm_activity_report_views.xml',
        'report/crm_opportunity_report_views.xml',
        'views/crm_team_views.xml',
        'views/crm_menu_views.xml',
        'views/crm_helper_templates.xml',
    ],
    'demo': [
        'data/crm_team_demo.xml',
        'data/mail_template_demo.xml',
        'data/crm_team_member_demo.xml',
        'data/mail_activity_type_demo.xml',
        'data/crm_lead_demo.xml',
    ],
    'installable': True,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'crm/static/src/**/*',
        ],
        'web.assets_tests': [
            'crm/static/tests/tours/**/*',
        ],
        'web.qunit_suite_tests': [
            'crm/static/tests/**/*',
            ('remove', 'crm/static/tests/tours/**/*'),
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging

from odoo.addons.mail.controllers.mail import MailController
from odoo import http
from odoo.http import request

_logger = logging.getLogger(__name__)


class CrmController(http.Controller):

    @http.route('/lead/case_mark_won', type='http', auth='user', methods=['GET'])
    def crm_lead_case_mark_won(self, res_id, token):
        comparison, record, redirect = MailController._check_token_and_record_or_redirect('crm.lead', int(res_id), token)
        if comparison and record:
            try:
                record.action_set_won_rainbowman()
            except Exception:
                _logger.exception("Could not mark crm.lead as won")
                return MailController._redirect_to_messaging()
        return redirect

    @http.route('/lead/case_mark_lost', type='http', auth='user', methods=['GET'])
    def crm_lead_case_mark_lost(self, res_id, token):
        comparison, record, redirect = MailController._check_token_and_record_or_redirect('crm.lead', int(res_id), token)
        if comparison and record:
            try:
                record.action_set_lost()
            except Exception:
                _logger.exception("Could not mark crm.lead as lost")
                return MailController._redirect_to_messaging()
        return redirect

    @http.route('/lead/convert', type='http', auth='user', methods=['GET'])
    def crm_lead_convert(self, res_id, token):
        comparison, record, redirect = MailController._check_token_and_record_or_redirect('crm.lead', int(res_id), token)
        if comparison and record:
            try:
                record.convert_opportunity(record.partner_id)
            except Exception:
                _logger.exception("Could not convert crm.lead to opportunity")
                return MailController._redirect_to_messaging()
        return redirect

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*
from . import main

```

## File: data\crm_lead_demo.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">
        <!--  Demo Leads -->
        <record id="crm_case_1" model="crm.lead">
            <field name="create_date" eval="DateTime.now() - relativedelta(days=8)"/>
            <field name="type">lead</field>
            <field name="name">Club Office Furnitures</field>
            <field name="contact_name">Jacques Dunagan</field>
            <field name="partner_name">Le Club SARL</field>
            <field name="email_from">jdunagan@leclub.example.com</field>
            <field name="function">Training Manager</field>
            <field name="country_id" ref="base.fr"/>
            <field name="city">Paris</field>
            <field name="zip">93190</field>
            <field name="street">Rue Léon Dierx 73</field>
            <field name="phone">+33 1 25 54 45 69</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor6')])]"/>
            <field name="priority">1</field>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(months=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="stage_lead1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_services"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_mailing"/>
        </record>
        <record id="msg_case1_1" model="mail.message">
            <field name="subject">Inquiry</field>
            <field name="model">crm.lead</field>
            <field name="res_id" ref="crm_case_1"/>
            <field name="author_id" eval="False"/>
            <field name="email_from">jdunagan@leclub.example.com</field>
            <field name="body"><![CDATA[<p>Hello,<br />
            I am Jacques from Le Club SARL. I am interested in attending a training organized by your company.<br />
            Can you send me the details?</p>]]></field>
            <field name="message_type">email</field>
            <field name="subtype_id" ref="mail.mt_comment"/>
        </record>

        <record id="crm_case_2" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=8)"/>
            <field name="type">lead</field>
            <field name="name">Design Software</field>
            <field name="contact_name">Marc Dufour</field>
            <field name="partner_name">The Oil Company</field>
            <field name="email_from">md@oilcompany.fr</field>
            <field name="function">Purchase Manager</field>
            <field name="country_id" ref="base.fr"/>
            <field name="city">Bordeaux</field>
            <field name="zip">33000</field>
            <field name="street">Rue Ignasse Blanchoux 214/32</field>
            <field name="phone">+33 1 25 54 45 69</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor2')])]"/>
            <field name="priority">1</field>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(months=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="stage_lead1"/>
            <field name="campaign_id" ref="utm.utm_campaign_christmas_special"/>
            <field name="medium_id" ref="utm.utm_medium_website"/>
            <field name="source_id" ref="utm.utm_source_newsletter"/>
        </record>
        <record id="msg_case2_1" model="mail.message">
            <field name="subject">Need Details</field>
            <field name="model">crm.lead</field>
            <field name="author_id" eval="False"/>
            <field name="email_from">md@oilcompany.fr</field>
            <field name="res_id" ref="crm_case_2"/>
            <field name="body">Want to know features and benefits of using the new software.</field>
            <field name="message_type">email</field>
            <field name="subtype_id" ref="mail.mt_comment"/>
        </record>

        <record id="crm_case_3" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=2)"/>
            <field name="type">lead</field>
            <field name="name">Pricing for 25 desks</field>
            <field name="contact_name">John Miller</field>
            <field name="partner_name">The Kompany</field>
            <field name="email_from">contact@thekompany.example.com</field>
            <field name="country_id" ref="base.us"/>
            <field name="city">New York</field>
            <field name="zip">10001</field>
            <field name="street">Lafayette Ave 450/12</field>
            <field name="phone">+1 555 754 3010</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor4'), ref('sales_team.categ_oppor5')])]"/>
            <field name="priority">2</field>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(months=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="stage_lead1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_google_adwords"/>
            <field name="source_id" ref="utm.utm_source_facebook"/>
        </record>

        <record id="crm_case_4" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=2)"/>
            <field name="type">lead</field>
            <field name="name">Campbell: Chairs</field>
            <field name="contact_name">Henry Campbell</field>
            <field name="partner_name">Burstein Applebee</field>
            <field name="email_from">hmc@yahoo.example.com</field>
            <field name="country_id" ref="base.uk"/>
            <field name="city">Manchester</field>
            <field name="zip">03101</field>
            <field name="street">United Street 68</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor5')])]"/>
            <field name="priority">2</field>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(months=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="stage_lead1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_banner"/>
            <field name="source_id" ref="utm.utm_source_newsletter"/>
        </record>

        <record id="crm_case_5" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=4)"/>
            <field name="type">lead</field>
            <field name="name">Quotation for 50 Chairs</field>
            <field name="contact_name">Carrie Helle</field>
            <field name="partner_name">Stonage IT</field>
            <field name="email_from">helle@stonageit.example.com</field>
            <field name="function">Purchase Manager</field>
            <field name="country_id" ref="base.us"/>
            <field name="city">Philadelphia</field>
            <field name="zip">1909</field>
            <field name="street">West Allegheny Ave 800</field>
            <field name="phone">+1 813 494 5005</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">2</field>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="stage_lead1"/>
            <field name="description"></field>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_mailing"/>
        </record>
        <record id="msg_case5_1" model="mail.message">
            <field name="subject">Inquiry</field>
            <field name="model">crm.lead</field>
            <field name="res_id" ref="crm_case_5"/>
            <field name="author_id" eval="False"/>
            <field name="email_from">helle@stonageit.example.com</field>
            <field name="body"><![CDATA[<p>Hi,<br />
Can you send me a quotation for 20 computers with speakers?<br />
Regards,<br />
Carrie Helle,<br />
Purchase Manager<br />
Stonage IT,<br />
Philadelphia<br />
Contact: +1 813 494 5005</p>]]></field>
            <field name="message_type">email</field>
            <field name="subtype_id" ref="mail.mt_comment"/>
        </record>

        <record id="crm_case_6" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=5)"/>
            <field name="type">lead</field>
            <field name="name">Opensides: Need Info</field>
            <field name="contact_name">Tina Pinero</field>
            <field name="partner_name">Opensides</field>
            <field name="email_from">tina@opensides.example.com</field>
            <field name="function">Consultant</field>
            <field name="country_id" ref="base.it"/>
            <field name="city">Roma</field>
            <field name="zip">00118</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor3'), ref('sales_team.categ_oppor4')])]"/>
            <field name="priority">2</field>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="stage_lead1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_services"/>
            <field name="source_id" ref="utm.utm_source_mailing"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
        </record>

        <record id="crm_case_7" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=5)"/>
            <field name="type">lead</field>
            <field name="name">Gardner: Desks Replacement</field>
            <field name="contact_name">Wendi Baltz</field>
            <field name="partner_name">Gardner Group</field>
            <field name="function">Journalist</field>
            <field name="country_id" ref="base.br"/>
            <field name="city">Rio de Janeiro</field>
            <field name="zip">29000</field>
            <field name="street">R. Sen. Pompeu</field>
            <field name="phone">+11 55 21 5555 5555</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor4')])]"/>
            <field name="priority">0</field>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(weeks=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="stage_lead1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="source_id" ref="utm.utm_source_mailing"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
        </record>

        <record id="crm_case_8" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=8)"/>
            <field name="type">lead</field>
            <field name="name">Product Catalog</field>
            <field name="contact_name">Logan</field>
            <field name="partner_name">ESM Expert</field>
            <field name="email_from">logan_expert@gmail.example.com</field>
            <field name="function">Sales</field>
            <field name="country_id" ref="base.uk"/>
            <field name="city">London</field>
            <field name="zip">E1AB</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor4'), ref('sales_team.categ_oppor8')])]"/>
            <field name="priority">1</field>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="stage_lead1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="source_id" ref="utm.utm_source_mailing"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
        </record>
        <record id="msg_case8_1" model="mail.message">
            <field name="subject">Inquiry</field>
            <field name="model">crm.lead</field>
            <field name="res_id" ref="crm_case_8"/>
            <field name="author_id" eval="False"/>
            <field name="email_from">logan_expert@gmail.example.com</field>
            <field name="body"><![CDATA[<p>Hi,<br />
Could you send me your product catalogue please?<br />
Regards,<br />
David Logan<br />
ESM Expert<br />]]></field>
            <field name="message_type">email</field>
            <field name="subtype_id" ref="mail.mt_comment"/>
        </record>

        <record id="crm_case_9" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=4)"/>
            <field name="type">lead</field>
            <field name="name">Reseller: Office furnitures</field>
            <field name="contact_name">Delisle Albert</field>
            <field name="partner_name">Marketing Business</field>
            <field name="email_from">d.albert@marketing-business.example.com</field>
            <field name="function">Sales</field>
            <field name="country_id" ref="base.uk"/>
            <field name="city">Oxford</field>
            <field name="zip">OX1 1RQ</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor4'), ref('sales_team.categ_oppor7')])]"/>
            <field name="priority">2</field>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(days=4)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="stage_lead1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="source_id" ref="utm.utm_source_mailing"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
        </record>

        <record id="crm_case_10" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=2)"/>
            <field name="type">lead</field>
            <field name="name">Design Software Info</field>
            <field name="contact_name">Jose Garcia</field>
            <field name="partner_name">Solar IT</field>
            <field name="function">Medical illustrator</field>
            <field name="email_from">jga@solar.example.com</field>
            <field name="country_id" ref="base.es"/>
            <field name="city">Madrid</field>
            <field name="zip">28001</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">2</field>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(days=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="stage_lead1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="source_id" ref="utm.utm_source_mailing"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
        </record>
        <record id="msg_case10_1" model="mail.message">
            <field name="subject">Inquiry</field>
            <field name="model">crm.lead</field>
            <field name="res_id" ref="crm_case_10"/>
            <field name="author_id" eval="False"/>
            <field name="email_from">jga@solar.example.com</field>
            <field name="body"><![CDATA[<p>Hi,<br />
I would like to know more about specification and cost of laptops of your company.<br />
Thanks,<br />
Andrew</p>]]></field>
            <field name="message_type">email</field>
            <field name="subtype_id" ref="mail.mt_comment"/>
        </record>

        <record id="crm_case_11" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=5)"/>
            <field name="type">lead</field>
            <field name="name">Estimation for Office Furnitures</field>
            <field name="contact_name">Thomas Passot</field>
            <field name="partner_name">Deco Addict</field>
            <field name="email_from">p.thomas@agrolait.com</field>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="function">Functional Consultant</field>
            <field name="country_id" ref="base.be"/>
            <field name="city">Wavre</field>
            <field name="zip">1300</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor7')])]"/>
            <field name="priority">2</field>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(days=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="stage_lead1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="source_id" ref="utm.utm_source_mailing"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
        </record>

        <record id="crm_case_12" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=8)"/>
            <field name="type">lead</field>
            <field name="name">Quotation for 100 Desks</field>
            <field name="contact_name">Bojing Hú</field>
            <field name="partner_name">Incom Corporation</field>
            <field name="email_from">bhu.a100@ic.example.com</field>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="country_id" ref="base.cn"/>
            <field name="city">Shenzhen</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">2</field>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="stage_id" ref="stage_lead1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_website"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <!-- Call Function to Cancel the leads (set as Dead) -->
        <function model="crm.lead" name="action_set_lost"
            eval="[[ref('crm_case_7'), ref('crm_case_9'), ref('crm_case_11'), ref('crm_case_12')]]"
            context="{'install_mode': True}"/>

        <!-- Demo Opportunities -->
        <record id="crm_case_13" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=1)"/>
            <field name="type">opportunity</field>
            <field name="name">Quote for 12 Tables</field>
            <field name="expected_revenue">40000</field>
            <field name="probability">10.0</field>
            <field name="contact_name">Will McEncroe</field>
            <field name="partner_name">Rediff Mail</field>
            <field name="email_from">willmac@rediffmail.example.com</field>
            <field name="country_id" ref="base.au"/>
            <field name="city">Melbourne</field>
            <field name="street">Kensington Road 189</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(months=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(weeks=3)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="crm.stage_lead1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_14" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=1)"/>
            <field name="type">opportunity</field>
            <field name="name">Office Design Project</field>
            <field name="color">7</field>
            <field name="expected_revenue">24000</field>
            <field name="probability">10.0</field>
            <field name="partner_name">Deco Addict</field>
            <field name="email_from">info@agrolait.com</field>
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="country_id" ref="base.be"/>
            <field name="city">Wavre</field>
            <field name="zip">1300</field>
            <field name="street">Rue de Namur 69</field>
            <field name="phone">+32 10 588 558</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor2')])]"/>
            <field name="priority">2</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(months=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="crm.stage_lead1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_website"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_15" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=1)"/>
            <field name="type">opportunity</field>
            <field name="name">Info about services</field>
            <field name="expected_revenue">25000</field>
            <field name="probability">30.0</field>
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="email_from">virginie@agrolait.com</field>
            <field name="country_id" ref="base.be"/>
            <field name="city">Wavre</field>
            <field name="street">Rue de Namur 69</field>
            <field name="phone">+32 10 588 558</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(months=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="crm.stage_lead2"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_services"/>
            <field name="source_id" ref="utm.utm_source_mailing"/>
            <field name="medium_id" ref="utm.utm_medium_direct"/>
        </record>
        <record id="msg_case15_attach1" model="ir.attachment">
            <field name="datas">bWlncmF0aW9uIHRlc3Q=</field>
            <field name="name">YourCompany2012.doc</field>
            <field name="res_model">crm.lead</field>
            <field name="res_id" ref="crm_case_15"/>
        </record>
        <record id="msg_case15_1" model="mail.message">
            <field name="subject">Plan to buy RedHat servers</field>

            <field name="model">crm.lead</field>
            <field name="res_id" ref="crm_case_15"/>
            <field name="author_id" ref="base.res_partner_2"/>
            <field name="email_from">virginie@agrolait.fr</field>
            <field name="body"><![CDATA[<div>
            <p>Hello,</p>
            <p>I am interested in your company's products and I plan to buy a new laptop having latest technologies as well as an affordable price.</p>
            <p>Could you please send me the product catalog?</p>]]></field>
            <field name="message_type">email</field>
            <field name="subtype_id" ref="mail.mt_comment"/>
        </record>
        <record id="msg_case15_2" model="mail.message">
            <field name="subject">Re: Plan to buy RedHat servers</field>
            <field name="model">crm.lead</field>
            <field name="res_id" ref="crm_case_15"/>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="body"><![CDATA[<p>Dear customer,<br/>
            Thanks for showing interest in our products! As requested, I send to you our products catalog.<br />
            To be able to finely tune the solution, we would like to know precise needs. This way we wil be able to help you choosing the right infrastructure according to your requirements.<br/>
            Best regards,</p>]]></field>
            <field name="parent_id" ref="msg_case15_1"/>
            <field name="message_type">comment</field>
            <field name="subtype_id" ref="mail.mt_comment"/>
            <field name="attachment_ids" eval="[(6, 0, [ref('msg_case15_attach1')])]"/>
        </record>
        <record id="msg_case15_3" model="mail.message">
            <field name="subject">Re: Plan to buy RedHat servers</field>
            <field name="model">crm.lead</field>
            <field name="res_id" ref="crm_case_15"/>
            <field name="email_from">virginie@agrolait.fr</field>
            <field name="author_id" ref="base.res_partner_2"/>
            <field name="body"><![CDATA[<p>Thanks for the information!<br />I asked a precise specification to our technical expert. Here is what we precisely need:</p>
            <ul>
                <li>weekly backups, every Monday</li>
                <li>backup time is not a blocking point for us, as we are closed all Monday, leaving time enough to perform the backup</li>
                <li>reliability is very important; we need redundant servers and rollback capacity</li>
                <li>a total capacity of about 2 TB</li>
            </ul>
            <p>Best regards,</p>]]></field>
            <field name="message_type">email</field>
            <field name="subtype_id" ref="mail.mt_comment"/>
            <field name="parent_id" ref="msg_case15_1"/>
        </record>
        <record id="msg_case15_4" model="mail.message">
            <field name="subject">Re: Plan to buy RedHat servers</field>
            <field name="model">crm.lead</field>
            <field name="res_id" ref="crm_case_15"/>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="body"><![CDATA[<p>Hello</p>
            <p>After our discussion with our technical experts, here is the offer of YourCompany. We believe it will meet every requirement you had in mind. Please feel free to contact me for any detail or technical detail that is not clear enough for you.</p>
            <p>Notice that as agreed on phone, we offer you a <b>10% discount on the hardware</b>!</p>
            <p>Best regards,</p>]]></field>
            <field name="message_type">email</field>
            <field name="subtype_id" ref="mail.mt_comment"/>
            <field name="parent_id" ref="msg_case15_1"/>
        </record>

        <record id="crm_case_16" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=2)"/>
            <field name="type">opportunity</field>
            <field name="name">Global Solutions: Furnitures</field>
            <field name="expected_revenue">3800</field>
            <field name="probability">90.0</field>
            <field name="contact_name">Robin Smith</field>
            <field name="partner_name">Global Solutions</field>
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="country_id" ref="base.uk"/>
            <field name="city">Liverpool</field>
            <field name="zip">L25 4RL</field>
            <field name="street">Union Road</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor5')])]"/>
            <field name="priority">2</field>
            <field name="date_deadline" eval="DateTime.today().strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(weeks=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="crm.stage_lead2"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="source_id" ref="utm.utm_source_mailing"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
        </record>

        <record id="crm_case_17" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=3)"/>
            <field name="type">opportunity</field>
            <field name="name">Balmer Inc: Potential Distributor</field>
            <field name="expected_revenue">1000</field>
            <field name="probability">35.0</field>
            <field name="partner_name">BalmerInc S.A.</field>
            <field name="contact_name">Oliver Passot</field>
            <field name="email_from">olivier.passo@balmer.inc.sa</field>
            <field name="phone">+32 469 12 45 78</field>
            <field name="country_id" ref="base.be"/>
            <field name="city">Brussels</field>
            <field name="zip">1100</field>
            <field name="street">Rue des Palais 51, bte 33</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor3'),ref('sales_team.categ_oppor4')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(months=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(weeks=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="crm.stage_lead2"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_website"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>
        <record id="msg_case17_1" model="mail.message">
            <field name="subject">Catalog to send</field>
            <field name="model">crm.lead</field>
            <field name="res_id" ref="crm_case_17"/>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="body"><![CDATA[<p>They just want pricing information about our services. I think sending our catalog should be sufficient.</p>]]></field>
            <field name="message_type">comment</field>
            <field name="subtype_id" ref="mail.mt_comment"/>
        </record>

        <record id="crm_case_18" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=7)"/>
            <field name="type">opportunity</field>
            <field name="name">DeltaPC: 10 Computer Desks</field>
            <field eval="35000" name="expected_revenue"/>
            <field eval="25.0" name="probability"/>
            <field name="contact_name">Leland Martinez</field>
            <field name="email_from">info@deltapc.example.com</field>
            <field name="partner_name">Delta PC</field>
            <field name="city">London</field>
            <field name="street">3661 Station Street</field>
            <field name="country_id" ref="base.uk"/>
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor4'),ref('sales_team.categ_oppor6')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(months=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(weeks=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="crm.stage_lead2"/>
            <field eval="1" name="active"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_services"/>
            <field name="source_id" ref="utm.utm_source_newsletter"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
        </record>
        <record id="msg_case18_1" model="mail.message">
            <field name="subject">Inquiry</field>
            <field name="model">crm.lead</field>
            <field name="res_id" ref="crm_case_18"/>
            <field name="author_id" ref="base.res_partner_4"/>
            <field name="body"><![CDATA[<p>Hello!<br />
            I am Leland Martinez, from the Delta PC. Maybe you remember, we talked a bit last month at this international conference.<br />
            We would like to attend a training, but we are not quite sure about what we can ask. Maybe we should meet and talk about that?<br />
            Best regards,</p>]]></field>
            <field name="message_type">email</field>
            <field name="subtype_id" ref="mail.mt_comment"/>
        </record>
        <record id="msg_case18_2" model="mail.message">
            <field name="model">crm.lead</field>
            <field name="res_id" ref="crm_case_18"/>
            <field name="author_id" ref="base.partner_demo"/>
            <field name="body"><![CDATA[<p>It seems very interesting. As you say, first of all we will have to define precisely what the training will be about, and your precise needs.</p>]]></field>
            <field name="message_type">comment</field>
            <field name="subtype_id" ref="mail.mt_comment"/>
            <field name="parent_id" ref="msg_case18_1"/>
        </record>

        <record id="crm_case_19" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=7)"/>
            <field name="type">opportunity</field>
            <field name="name">Customizable Desk</field>
            <field name="color">3</field>
            <field name="expected_revenue">15000</field>
            <field name="probability">65.5</field>
            <field name="contact_name">Nhomar</field>
            <field name="partner_name">Vauxoo</field>
            <field name="email_from">vauxoo@yourcompany.example.com</field>
            <field name="partner_id" ref="base.res_partner_12"/>
            <field name="country_id" ref="base.ve"/>
            <field name="city">Caracas</field>
            <field name="zip">1090</field>
            <field name="street">3rd Floor, Room 3-C,</field>
            <field name="street2">Carretera Panamericana, Km 1, Urb. Delgado Chalbaud</field>
            <field name="phone">+58 212 681 0538</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="crm.stage_lead3"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="source_id" ref="utm.utm_source_mailing"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
        </record>

        <record id="crm_case_20" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=7)"/>
            <field name="type">opportunity</field>
            <field name="name">Distributor Contract</field>
            <field name="expected_revenue">19800</field>
            <field name="contact_name">John M. Brown</field>
            <field name="partner_name">Epic Technologies</field>
            <field name="email_from">john.b@tech.info</field>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="country_id" ref="base.us"/>
            <field name="zip">60610</field>
            <field name="city">Chicago</field>
            <field name="phone">+1 312 349 2324</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor4'),ref('sales_team.categ_oppor8')])]"/>
            <field name="priority">2</field>
            <field name="date_deadline" eval="DateTime.today().strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="crm.stage_lead3"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_services"/>
            <field name="source_id" ref="utm.utm_source_mailing"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
        </record>

        <record id="crm_case_21" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=1)"/>
            <field name="type">opportunity</field>
            <field name="name">Office Design and Architecture</field>
            <field name="expected_revenue">9000</field>
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="country_id" ref="base.uk"/>
            <field name="city">Birmingham</field>
            <field name="zip">B46 3AG</field>
            <field name="street">Cannon Hill Park</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor7')])]"/>
            <field name="priority">2</field>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="crm.stage_lead3"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_services"/>
            <field name="medium_id" ref="utm.utm_medium_phone"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_22" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=8)"/>
            <field name="type">opportunity</field>
            <field name="name">5 VP Chairs</field>
            <field name="expected_revenue">5600</field>
            <field name="probability">30.0</field>
            <field name="contact_name">Benjamin Flores</field>
            <field name="partner_name">Nebula Business</field>
            <field name="partner_id" ref="base.res_partner_12"/>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor3')])]"/>
            <field name="priority">1</field>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(days=3)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="crm.stage_lead3"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_23" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=7)"/>
            <field name="type">opportunity</field>
            <field name="name">Access to Online Catalog</field>
            <field name="expected_revenue">2000</field>
            <field name="probability">80.0</field>
            <field name="partner_id" ref="base.res_partner_18"/>
            <field name="priority">0</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor3')])]"/>
            <field name="date_deadline" eval="DateTime.today().strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(days=3)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="crm.stage_lead3"/>
            <field name="campaign_id" ref="utm.utm_campaign_fall_drive"/>
            <field name="medium_id" ref="utm.utm_medium_direct"/>
            <field name="source_id" ref="utm.utm_source_newsletter"/>
        </record>

        <record id="crm_case_24" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=7)"/>
            <field name="type">opportunity</field>
            <field name="name">Need 20 Desks</field>
            <field name="expected_revenue">60000</field>
            <field name="probability">90.0</field>
            <field name="email_from">info@mycompany.net</field>
            <field name="country_id" ref="base.pe"/>
            <field name="city">Lima</field>
            <field name="priority">0</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(months=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor7')])]"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(days=3)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="crm.stage_lead3"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_website"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_25" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=1)"/>
            <field name="type">opportunity</field>
            <field name="name">Modern Open Space</field>
            <field name="expected_revenue">4500</field>
            <field name="probability">60</field>
            <field name="contact_name">Henry Jordan</field>
            <field name="partner_name">E-light Industry</field>
            <field name="email_from">henry@elight.com</field>
            <field name="country_id" ref="base.ar"/>
            <field name="city">Buenos Aires</field>
            <field name="zip">B7313</field>
            <field name="street">Palermo, Capital Federal</field>
            <field name="street2">C1414CMS Capital Federal</field>
            <field name="priority">2</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor4')])]"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(days=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="crm.stage_lead3"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_services"/>
            <field name="medium_id" ref="utm.utm_medium_phone"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_26" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=6)"/>
            <field name="type">opportunity</field>
            <field name="name">Open Space Design</field>
            <field name="expected_revenue">11000</field>
            <field name="probability">45</field>
            <field name="partner_name">Deco Addict</field>
            <field name="email_from">info@agrolait.com</field>
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="street">Rue de Namur 69</field>
            <field name="country_id" ref="base.be"/>
            <field name="city">Wavre</field>
            <field name="zip">1300</field>
            <field name="priority">2</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor2')])]"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="date_open" eval="(DateTime.today() - relativedelta(days=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="stage_id" ref="crm.stage_lead3"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_website"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_27" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=1)"/>
            <field name="type">opportunity</field>
            <field name="name">Interest in your products</field>
            <field name="expected_revenue">2000</field>
            <field name="probability">80</field>
            <field name="partner_name">Deco Addict</field>
            <field name="email_from">info@agrolait.com</field>
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="country_id" ref="base.be"/>
            <field name="city">Wavre</field>
            <field name="zip">1300</field>
            <field name="street">Rue de Namur 69</field>
            <field name="priority">2</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor2')])]"/>
            <field name="date_deadline" eval="DateTime.today().strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="stage_id" ref="crm.stage_lead3"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_mailing"/>
        </record>

        <record id="crm_case_28" model="crm.lead">
            <field name="create_date" eval="datetime.now() - timedelta(days=1)"/>
            <field name="type">opportunity</field>
            <field name="name">Furnish a 60m² office</field>
            <field name="expected_revenue">7500</field>
            <field name="probability">20</field>
            <field name="partner_name">Deco Addict</field>
            <field name="email_from">info@agrolait.com</field>
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="country_id" ref="base.be"/>
            <field name="city">Wavre</field>
            <field name="zip">1300</field>
            <field name="street">Rue de Namur 69</field>
            <field name="priority">0</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1'), ref('sales_team.categ_oppor5')])]"/>
            <field name="date_deadline" eval="DateTime.today() + relativedelta(days=12)"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="stage_id" ref="crm.stage_lead1"/>
        </record>

        <record id="crm_case_29" model="crm.lead">
            <field name="create_date" eval="DateTime.now() - relativedelta(months=1)"/>
            <field name="type">lead</field>
            <field name="name">Club Office More Desks</field>
            <field name="contact_name">Jacques Dunagan</field>
            <field name="partner_name">Le Club SARL</field>
            <field name="email_from">jdunagan@leclub.example.com</field>
            <field name="function">Training Manager</field>
            <field name="country_id" ref="base.fr"/>
            <field name="city">Paris</field>
            <field name="zip">93190</field>
            <field name="street">Rue Léon Dierx 73</field>
            <field name="phone">+33 1 25 54 45 69</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor6')])]"/>
            <field name="priority">1</field>
            <field name="date_open" eval="False"/>
            <field name="team_id" eval="False"/>
            <field name="user_id" eval="False"/>
            <field name="stage_id" ref="stage_lead2"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_mailing"/>
        </record>

        <record id="crm_case_30" model="crm.lead">
            <field name="create_date" eval="DateTime.now() - relativedelta(months=2)"/>
            <field name="type">lead</field>
            <field name="name">Acadia College Furnitures</field>
            <field name="contact_name">Gaston Rochon</field>
            <field name="partner_name">Acadia College</field>
            <field name="email_from">GastonRochon@example.com</field>
            <field name="function">Director</field>
            <field name="country_id" ref="base.fr"/>
            <field name="city">Brussels</field>
            <field name="zip">1080</field>
            <field name="street">Rue du Commerce 93</field>
            <field name="phone">+32 22 33 54 07</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor6')])]"/>
            <field name="priority">1</field>
            <field name="date_open" eval="False"/>
            <field name="team_id" eval="False"/>
            <field name="user_id" eval="False"/>
            <field name="stage_id" ref="stage_lead1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_mailing"/>
        </record>

        <record id="crm_case_31" model="crm.lead">
            <field name="create_date" eval="datetime.now() - relativedelta(months=1)"/>
            <field name="type">opportunity</field>
            <field name="name">Quote for 150 carpets</field>
            <field name="expected_revenue">40000</field>
            <field name="probability">10.0</field>
            <field name="contact_name">Erik N. French</field>
            <field name="email_from">ErikNFrench@armyspy.com</field>
            <field name="country_id" ref="base.au"/>
            <field name="city">Chevy Chase</field>
            <field name="street">1920 Del Dew Drive</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(months=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="stage_id" ref="crm.stage_lead1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_32" model="crm.lead">
            <field name="create_date" eval="datetime.now() - relativedelta(months=2)"/>
            <field name="type">opportunity</field>
            <field name="name">Quote for 600 Chairs</field>
            <field name="expected_revenue">22500</field>
            <field name="probability">20.0</field>
            <field name="contact_name">Erik N. French</field>
            <field name="email_from">ErikNFrench@armyspy.com</field>
            <field name="country_id" ref="base.au"/>
            <field name="city">Chevy Chase</field>
            <field name="street">1920 Del Dew Drive</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(months=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="stage_id" ref="crm.stage_lead2"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_graph_1" model="crm.lead">
            <field name="create_date" eval="datetime.now() - relativedelta(months=1)"/>
            <field name="type">lead</field>
            <field name="name">Recurring delivery contract</field>
            <field name="expected_revenue">36221</field>
            <field name="probability">20</field>
            <field name="contact_name">Max Johnson</field>
            <field name="email_from">max123@itconsult.com</field>
            <field name="country_id" ref="base.it"/>
            <field name="city">Milan</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" eval="False"/>
            <field name="user_id" eval="False"/>
            <field name="stage_id" ref="crm.stage_lead2"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_graph_2" model="crm.lead">
            <field name="create_date" eval="datetime.now() - relativedelta(days=18)"/>
            <field name="type">lead</field>
            <field name="name">Need info about pricing</field>
            <field name="expected_revenue">28443</field>
            <field name="probability">80</field>
            <field name="contact_name">Aloysius Akred</field>
            <field name="email_from">aakreda@theglobeandmail.com</field>
            <field name="country_id" ref="base.gr"/>
            <field name="city">London</field>
            <field name="street">1 Russel square</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" eval="False"/>
            <field name="user_id" eval="False"/>
            <field name="stage_id" ref="crm.stage_lead3"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_graph_3" model="crm.lead">
            <field name="create_date" eval="datetime.now() - relativedelta(days=24)"/>
            <field name="type">lead</field>
            <field name="name">Trelian New Offices</field>
            <field name="expected_revenue">88715</field>
            <field name="probability">12</field>
            <field name="contact_name">Rosalynd Oxshott</field>
            <field name="partner_name">Photobug</field>
            <field name="email_from">roxshott9@trellian.com</field>
            <field name="country_id" ref="base.br"/>
            <field name="city">Canguaretama</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" eval="False"/>
            <field name="user_id" eval="False"/>
            <field name="stage_id" ref="crm.stage_lead2"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_graph_4" model="crm.lead">
            <field name="create_date" eval="datetime.now() - relativedelta(days=40)"/>
            <field name="type">lead</field>
            <field name="name">Branded Furniture</field>
            <field name="expected_revenue">7784</field>
            <field name="probability">20</field>
            <field name="contact_name">Myrna Limprecht</field>
            <field name="email_from">mlimprecht8@fastcompany.com</field>
            <field name="country_id" ref="base.pt"/>
            <field name="city">Valejas</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" eval="False"/>
            <field name="user_id" eval="False"/>
            <field name="stage_id" ref="crm.stage_lead2"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_graph_5" model="crm.lead">
            <field name="create_date" eval="datetime.now() - relativedelta(days=45)"/>
            <field name="type">lead</field>
            <field name="name">Design New Shelves</field>
            <field name="expected_revenue">54587</field>
            <field name="probability">81</field>
            <field name="contact_name">Alys Kalinovich</field>
            <field name="partner_name">Jaloo</field>
            <field name="email_from">akalinovich7@tinypic.com</field>
            <field name="country_id" ref="base.id"/>
            <field name="city">Boafeo</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" eval="False"/>
            <field name="user_id" eval="False"/>
            <field name="stage_id" ref="crm.stage_lead2"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_graph_6" model="crm.lead">
            <field name="create_date" eval="datetime.now() - relativedelta(days=48)"/>
            <field name="type">lead</field>
            <field name="name">Office chairs</field>
            <field name="expected_revenue">5474</field>
            <field name="probability">20</field>
            <field name="contact_name">Jennine Jobbins</field>
            <field name="partner_name">Shufflebeat</field>
            <field name="email_from">jjobbins6@simplemachines.org</field>
            <field name="country_id" ref="base.ru"/>
            <field name="city">Gvardeysk</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" eval="False"/>
            <field name="user_id" eval="False"/>
            <field name="stage_id" ref="crm.stage_lead2"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_graph_7" model="crm.lead">
            <field name="create_date" eval="datetime.now() - relativedelta(days=80)"/>
            <field name="type">lead</field>
            <field name="name">Cleaning subscription</field>
            <field name="expected_revenue">11475</field>
            <field name="probability">25</field>
            <field name="contact_name">Elmo Espinazo</field>
            <field name="partner_name">Realblab</field>
            <field name="email_from">eespinazo5@reuters.com</field>
            <field name="country_id" ref="base.nl"/>
            <field name="city">Amsterdam</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor3')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" eval="False"/>
            <field name="user_id" eval="False"/>
            <field name="stage_id" ref="crm.stage_lead1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_services"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_graph_8" model="crm.lead">
            <field name="create_date" eval="datetime.now() - relativedelta(days=37)"/>
            <field name="type">lead</field>
            <field name="name">Custom Desks (100 pieces)</field>
            <field name="expected_revenue">9987</field>
            <field name="probability">20</field>
            <field name="contact_name">Chalmers Redford</field>
            <field name="partner_name">Muxo</field>
            <field name="email_from">credford4@salon.com</field>
            <field name="country_id" ref="base.ru"/>
            <field name="city">Odoyev</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="stage_id" ref="crm.stage_lead2"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_graph_9" model="crm.lead">
            <field name="create_date" eval="datetime.now() - relativedelta(days=32)"/>
            <field name="type">lead</field>
            <field name="name">Need a price: urgent</field>
            <field name="expected_revenue">4482</field>
            <field name="probability">90</field>
            <field name="contact_name">Itch Kirvell</field>
            <field name="partner_name">Skibox</field>
            <field name="email_from">ikirvell3@gnu.org</field>
            <field name="country_id" ref="base.id"/>
            <field name="city">Dahu Satu</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" eval="False"/>
            <field name="user_id" eval="False"/>
            <field name="stage_id" ref="crm.stage_lead3"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_graph_10" model="crm.lead">
            <field name="create_date" eval="datetime.now() - relativedelta(days=18)"/>
            <field name="type">lead</field>
            <field name="name">Furnitures for new location</field>
            <field name="expected_revenue">22500</field>
            <field name="probability">20</field>
            <field name="contact_name">Napoleon Grabert</field>
            <field name="partner_name">Twinte</field>
            <field name="email_from">ngrabert2@dailymail.co.uk</field>
            <field name="country_id" ref="base.jp"/>
            <field name="city">Tokyo</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" eval="False"/>
            <field name="user_id" eval="False"/>
            <field name="stage_id" ref="crm.stage_lead2"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_graph_11" model="crm.lead">
            <field name="create_date" eval="datetime.now() - relativedelta(days=52)"/>
            <field name="type">lead</field>
            <field name="name">Modernize old offices</field>
            <field name="expected_revenue">99755</field>
            <field name="probability">20</field>
            <field name="contact_name">Franny Seiller</field>
            <field name="partner_name">Yozio</field>
            <field name="email_from">fseiller1@theglobeandmail.com</field>
            <field name="country_id" ref="base.id"/>
            <field name="city">Wurigelebur</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" eval="False"/>
            <field name="user_id" eval="False"/>
            <field name="stage_id" ref="crm.stage_lead3"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>

        <record id="crm_case_graph_12" model="crm.lead">
            <field name="create_date" eval="datetime.now() - relativedelta(days=52)"/>
            <field name="type">lead</field>
            <field name="name">Quote for 35 windows</field>
            <field name="expected_revenue">41421</field>
            <field name="probability">20</field>
            <field name="contact_name">Tommi Brockhouse</field>
            <field name="partner_name">Gabcube</field>
            <field name="email_from">tbrockhouse0@google.pl</field>
            <field name="country_id" ref="base.be"/>
            <field name="city">Brussels</field>
            <field name="tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor1')])]"/>
            <field name="priority">1</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(weeks=2)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="team_id" eval="False"/>
            <field name="user_id" eval="False"/>
            <field name="stage_id" ref="crm.stage_lead3"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="utm.utm_source_search_engine"/>
        </record>


        <!-- Call Function to set dome opportunities as Lost -->
        <function model="crm.lead" name="action_set_lost"
            eval="[[ref('crm_case_28')]]"
            context="{'install_mode': True}"/>

        <!-- Call Function to set some opportunities as Won -->
        <function model="crm.lead" name="action_set_won"
            eval="[[ref('crm_case_20'), ref('crm_case_23'), ref('crm_case_27')]]"
            context="{'install_mode': True}"/>

        <!-- Activities -->
        <record id="activity_1" model="mail.activity">
            <field name="res_id" ref="crm.crm_case_13" />
            <field name="res_model_id" ref="crm.model_crm_lead"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_call" />
            <field name="summary">Meeting to go over pricing information.</field>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=3)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="activity_2" model="mail.activity">
            <field name="res_id" ref="crm.crm_case_14" />
            <field name="res_model_id" ref="crm.model_crm_lead"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_email"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=3)).strftime('%Y-%m-%d %H:%M')" />
            <field name="summary">Send Catalog by Email</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="activity_3" model="mail.activity">
            <field name="res_id" ref="crm.crm_case_15" />
            <field name="res_model_id" ref="crm.model_crm_lead"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_call"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=1)).strftime('%Y-%m-%d %H:%M')" />
            <field name="summary">Call to get system requirements</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="activity_4" model="mail.activity">
            <field name="res_id" ref="crm.crm_case_16" />
            <field name="res_model_id" ref="crm.model_crm_lead"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_call"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=2)).strftime('%Y-%m-%d %H:%M')" />
            <field name="summary">Convert to quote</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="activity_5" model="mail.activity">
            <field name="res_id" ref="crm.crm_case_17" />
            <field name="user_id" ref="base.user_demo"/>
            <field name="res_model_id" ref="crm.model_crm_lead"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_email"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=-1)).strftime('%Y-%m-%d %H:%M')" />
            <field name="summary">Send our service pricelist</field>
            <field name="create_uid" ref="base.user_admin"/>
        </record>
        <record id="activity_6" model="mail.activity">
            <field name="res_id" ref="crm.crm_case_18" />
            <field name="res_model_id" ref="crm.model_crm_lead"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_call"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=1)).strftime('%Y-%m-%d %H:%M')" />
            <field name="summary">Call to get training needs</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="activity_7" model="mail.activity">
            <field name="res_id" ref="crm.crm_case_19" />
            <field name="res_model_id" ref="crm.model_crm_lead"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_call"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=-2)).strftime('%Y-%m-%d %H:%M')" />
            <field name="summary">Followup on the proposal</field>
            <field name="user_id" ref="base.user_demo"/>
            <field name="create_uid" ref="base.user_admin"/>
        </record>
        <record id="activity_8" model="mail.activity">
            <field name="res_id" ref="crm.crm_case_20" />
            <field name="res_model_id" ref="crm.model_crm_lead"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_call"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=1)).strftime('%Y-%m-%d %H:%M')" />
        </record>
        <record id="activity_9" model="mail.activity">
            <field name="res_id" ref="crm.crm_case_21" />
            <field name="res_model_id" ref="crm.model_crm_lead"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_call"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=4)).strftime('%Y-%m-%d %H:%M')" />
        </record>
        <record id="activity_10" model="mail.activity">
            <field name="res_id" ref="crm.crm_case_22" />
            <field name="res_model_id" ref="crm.model_crm_lead"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_email"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=-2)).strftime('%Y-%m-%d %H:%M')" />
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="activity_11" model="mail.activity">
            <field name="res_id" ref="crm.crm_case_23" />
            <field name="res_model_id" ref="crm.model_crm_lead"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_email"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=1)).strftime('%Y-%m-%d %H:%M')" />
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="activity_12" model="mail.activity">
            <field name="res_id" ref="crm.crm_case_24" />
            <field name="res_model_id" ref="crm.model_crm_lead"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_email"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=2)).strftime('%Y-%m-%d %H:%M')" />
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="activity_13" model="mail.activity">
            <field name="res_id" ref="crm.crm_case_25" />
            <field name="res_model_id" ref="crm.model_crm_lead"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_call"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=0)).strftime('%Y-%m-%d %H:%M')" />
            <field name="summary">Conf call with technical service</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="activity_14" model="mail.activity">
            <field name="res_id" ref="crm.crm_case_26" />
            <field name="res_model_id" ref="crm.model_crm_lead"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_email"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=0)).strftime('%Y-%m-%d %H:%M')" />
            <field name="summary">Send Catalog by Email</field>
            <field name="create_uid" ref="base.user_admin"/>
        </record>

    </data>
</odoo>

```

## File: data\crm_lead_merge_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="crm_lead_merge_summary" name="crm_lead_merge_summary">
    <div class="crm_lead_merge_summary">
        <t t-foreach="opportunities" t-as="lead">
            <div>
                <span>Merged the Lead/Opportunity</span>
                <span class="fw-bold" t-field="lead.name"/>
                <span>into this one.</span>
            </div>
            <blockquote class="border-start" data-o-mail-quote="1">
                <div t-if="lead.expected_revenue">
                    <span>Expected Revenues:</span>
                    <span t-if="lead.expected_revenue">
                        <span t-if="lead.company_currency" t-field="lead.expected_revenue"
                            t-options='{"widget": "monetary", "display_currency": lead.company_currency}'/>
                        <span t-else="" t-out="lead.expected_revenue"/>
                        <span t-if="lead.recurring_revenue" groups="crm.group_use_recurring_revenues"> + </span>
                    </span>
                    <span t-if="lead.recurring_revenue" groups="crm.group_use_recurring_revenues">
                        <span t-if="lead.company_currency" t-field="lead.recurring_revenue"
                            t-options='{"widget": "monetary", "display_currency": lead.company_currency}'/>
                        <span t-else="" t-out="lead.recurring_revenue"/>
                        <span t-field="lead.recurring_plan.name"/>
                    </span>
                </div>
                <div t-elif="lead.recurring_revenue" groups="crm.group_use_recurring_revenues">
                    <span t-if="lead.company_currency" t-field="lead.recurring_revenue"
                        t-options='{"widget": "monetary", "display_currency": lead.company_currency}'/>
                    <span t-else="" t-out="lead.recurring_revenue"/>
                    <span t-field="lead.recurring_plan.name"/>
                </div>
                <div t-if="lead.probability">
                    Probability: <span t-field="lead.probability"/>%
                </div>
                <div>
                    Type: <span t-field="lead.type"/>
                </div>
                <div t-if="lead.type != 'lead'">
                    Stage: <span t-field="lead.stage_id"/>
                </div>
                <div t-if="lead.priority">
                    Priority: <span t-field="lead.priority"/>
                </div>
                <div t-if="lead.lost_reason_id">
                    Lost Reason: <span t-field="lead.lost_reason_id"/>
                </div>
                <div>
                    Created on: <span t-field="lead.create_date"/>
                </div>
                <div t-if="lead.date_automation_last">
                    Last Automation: <span t-field="lead.date_automation_last"/>
                </div>
                <div t-if="lead.date_deadline">
                    Expected Closing: <span t-field="lead.date_deadline"/>
                </div>
                <div t-if="not is_html_empty(lead.description)">
                    Notes: <span t-field="lead.description"/>
                </div>
                <div t-if="lead.lang_id" name="lang_id">
                    Language: <span t-field="lead.lang_id"/>
                </div>
                <div t-if="lead.referred" name="referred">
                    Referred By: <span t-field="lead.referred"/>
                </div>
                <div t-if="lead.tag_ids" name="tag_ids" class="d-flex flex-row">
                    Tags:
                    <div class="ms-2 d-flex flex-row">
                        <div t-foreach="lead.tag_ids" t-as="tag" t-esc="tag.name"
                            t-attf-class="badge rounded-pill o_tag o_tag_color_#{tag.color} d-inline-block"/>
                    </div>
                </div>
                <div t-if="lead.user_id" class="mt-3">
                    Salesperson: <span t-field="lead.user_id"/>
                </div>
                <div t-if="lead.team_id">
                    Sales Team: <span t-field="lead.team_id"/>
                </div>
                <div name="company" groups="base.group_multi_company">
                    Company: <span t-field="lead.company_id"/>
                </div>
                <div>
                    <div class="mt-3"
                            t-if="lead.contact_name or lead.partner_name or lead.phone or lead.mobile or lead.email_from or lead.website">
                        <div class="fw-bold">
                            Contact Details:
                        </div>
                        <div t-if="lead.contact_name">
                            Contact: <span t-field="lead.contact_name"/>
                        </div>
                        <div t-if="lead.partner_name">
                            Company Name: <span t-field="lead.partner_name"/>
                        </div>
                        <div t-if="lead.phone">
                            Phone: <span t-field="lead.phone"/>
                        </div>
                        <div t-if="lead.mobile">
                            Mobile: <span t-field="lead.mobile"/>
                        </div>
                        <div t-if="lead.email_from">
                            Email: <span t-field="lead.email_from"/>
                        </div>
                        <div t-if="lead.email_cc">
                            Email cc: <span t-field="lead.email_cc"/>
                        </div>
                        <div t-if="lead.website">
                            Website: <span t-field="lead.website"/>
                        </div>
                        <div t-if="lead.function">
                            Job Position: <span t-field="lead.function"/>
                        </div>
                    </div>
                </div>
                <div class="mt-3"
                        t-if="lead.street or lead.street2 or lead.zip or lead.city or lead.state_id or lead.country_id"
                        name="address">
                    <div class="fw-bold">
                        Address:
                    </div>
                    <div t-if="lead.street" t-field="lead.street"/>
                    <div t-if="lead.street2" t-field="lead.street2"/>
                    <div t-if="lead.zip" t-field="lead.zip"/>
                    <div t-if="lead.city" t-field="lead.city"/>
                    <div t-if="lead.state_id" t-field="lead.state_id"/>
                    <div t-if="lead.country_id" t-field="lead.country_id"/>
                </div>
                <div class="mt-3" name="marketing"
                        t-if="lead.campaign_id or lead.medium_id or lead.source_id">
                    <div class="fw-bold">
                        Marketing:
                    </div>
                    <div t-if="lead.campaign_id">
                        Campaign: <span t-field="lead.campaign_id"/>
                    </div>
                    <div t-if="lead.medium_id">
                        Medium: <span t-field="lead.medium_id"/>
                    </div>
                    <div t-if="lead.source_id">
                        Source: <span t-field="lead.source_id"/>
                    </div>
                </div>
                <t t-set="lead_followers" t-value="merged_followers and merged_followers.get(lead.id)"/>
                <div class="mt-3 mb-3" name="merged_followers" t-if="lead_followers">
                    <div>
                        The contacts below have been added as followers of this lead
                        because they have been contacted less than 30 days ago on
                        <span class="fw-bold" t-esc="lead.name"/>.
                    </div>
                    <ul>
                      <li t-foreach="lead_followers" t-as="follower">
                        <t t-esc="follower.partner_id.name"/>
                        <t t-if="follower.partner_id.email">
                            (<t t-esc="follower.partner_id.email"/>)
                        </t>
                      </li>
                    </ul>
                </div>
                <t t-set="properties" t-value="lead._format_properties()"/>
                <div t-if="properties" class="mt-3 mb-3">
                    <div class="fw-bold">
                        Properties
                    </div>
                    <ul class="p-0">
                        <li t-foreach="properties" t-as="property"
                            class="d-flex flex-row align-items-center">
                            <t t-esc="property['label']"/>:
                            <div t-if="'values' in property"
                                class="ms-2 d-flex flex-row"> <!-- Tags -->
                                <div t-foreach="property['values']" t-as="tag" t-esc="tag['name']"
                                    t-attf-class="badge rounded-pill o_tag o_tag_color_#{tag.get('color', 0)} d-inline-block me-2"/>
                            </div>
                            <div t-else="" class="ms-2" t-esc="property['value']"/>
                        </li>
                    </ul>
                </div>
            </blockquote>
        </t>
    </div>
</template>

</odoo>

```

## File: data\crm_lead_prediction_data.xml

```xml
<?xml version="1.0" encoding='UTF-8'?>
<odoo>
    <data noupdate="1">
        <!-- Lead scoring frequency fields -->
        <record id="frequency_field_state_id" model="crm.lead.scoring.frequency.field">
            <field name="field_id" ref="crm.field_crm_lead__state_id"/>
        </record>
        <record id="frequency_field_country_id" model="crm.lead.scoring.frequency.field">
            <field name="field_id" ref="crm.field_crm_lead__country_id"/>
        </record>
        <record id="frequency_field_phone_state" model="crm.lead.scoring.frequency.field">
            <field name="field_id" ref="crm.field_crm_lead__phone_state"/>
        </record>
        <record id="frequency_field_email_state" model="crm.lead.scoring.frequency.field">
            <field name="field_id" ref="crm.field_crm_lead__email_state"/>
        </record>
        <record id="frequency_field_source_id" model="crm.lead.scoring.frequency.field">
            <field name="field_id" ref="crm.field_crm_lead__source_id"/>
        </record>
        <record id="frequency_field_lang_id" model="crm.lead.scoring.frequency.field">
            <field name="field_id" ref="crm.field_crm_lead__lang_id"/>
        </record>
        <record id="frequency_field_tag_ids" model="crm.lead.scoring.frequency.field">
            <field name="field_id" ref="crm.field_crm_lead__tag_ids"/>
        </record>
        <record id="crm_pls_fields_param" model="ir.config_parameter">
            <field name="key">crm.pls_fields</field>
            <field name="value" eval="'phone_state,email_state'"/>
        </record>
        <record id="crm_pls_start_date_param" model="ir.config_parameter">
            <field name="key">crm.pls_start_date</field>
            <field name="value" eval="(datetime.now() - timedelta(days=8)).strftime('%Y-%m-%d')"/>
        </record>
    </data>

	<record id="website_crm_score_cron" model="ir.cron">
        <field name="name">Predictive Lead Scoring: Recompute Automated Probabilities</field>
        <field name="model_id" ref="model_crm_lead"/>
        <field name="state">code</field>
        <field name="code">model._cron_update_automated_probabilities()</field>
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="numbercall">-1</field>
        <field name="active" eval="False"/>
        <field name="doall" eval="False"/>
    </record>
</odoo>

```

## File: data\crm_lost_reason_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!-- opportunities' lost causes -->
    <record id="lost_reason_1" model="crm.lost.reason">
        <field name="name">Too expensive</field>
    </record>
    <record id="lost_reason_2" model="crm.lost.reason">
        <field name="name">We don't have people/skills</field>
    </record>
    <record id="lost_reason_3" model="crm.lost.reason">
        <field name="name">Not enough stock</field>
    </record>
</data></odoo>

```

## File: data\crm_recurring_plan_data.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data noupdate="1">

    <record id="crm_recurring_plan_monthly" model="crm.recurring.plan">
        <field name="name">Monthly</field>
        <field name="number_of_months">1</field>
    </record>

    <record id="crm_recurring_plan_yearly" model="crm.recurring.plan">
        <field name="name">Yearly</field>
        <field name="number_of_months">12</field>
    </record>

    <record id="crm_recurring_plan_over_3_years" model="crm.recurring.plan">
        <field name="name">Over 3 years</field>
        <field name="number_of_months">36</field>
    </record>

    <record id="crm_recurring_plan_over_5_years" model="crm.recurring.plan">
        <field name="name">Over 5 years </field>
        <field name="number_of_months">60</field>
    </record>

</data></odoo>

```

## File: data\crm_stage_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record model="crm.stage" id="stage_lead1">
        <field name="name">New</field>
        <field name="sequence">1</field>
    </record>
    <record model="crm.stage" id="stage_lead2">
        <field name="name">Qualified</field>
        <field name="sequence">2</field>
    </record>
    <record model="crm.stage" id="stage_lead3">
        <field name="name">Proposition</field>
        <field name="sequence">3</field>
    </record>
    <record model="crm.stage" id="stage_lead4">
        <field name="name">Won</field>
        <field name="fold" eval="False"/>
        <field name="is_won">True</field>
        <field name="sequence">70</field>
    </record>
</odoo>
```

## File: data\crm_team_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <record id="sales_team.team_sales_department" model="crm.team" forcecreate="False">
        <field name="alias_name">info</field>
    </record>

    <record id="sales_team.salesteam_website_sales" model="crm.team" forcecreate="False">
        <field name="use_opportunities" eval="False"/>
    </record>

    <record id="sales_team.pos_sales_team" model="crm.team" forcecreate="False">
        <field name="use_opportunities" eval="False"/>
    </record>

    <record id="sales_team.ebay_sales_team" model="crm.team" forcecreate="False">
        <field name="use_opportunities" eval="False"/>
    </record>
</data></odoo>

```

## File: data\crm_team_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="sales_team.team_sales_department" model="crm.team" forcecreate="False">
            <field name="assignment_domain">[['probability', '>=', 20]]</field>
        </record>

        <record id="sales_team.crm_team_1" model="crm.team">
            <field name="use_leads">True</field>
            <field name="assignment_domain">[['phone', '!=', False]]</field>
        </record>
    </data>
</odoo>

```

## File: data\crm_team_member_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <record id="sales_team.crm_team_member_admin_sales" model="crm.team.member">
        <field name="assignment_max">30</field>
        <field name="assignment_domain">[['probability', '>=', 20]]</field>
    </record>
    <record id="sales_team.crm_team_member_demo_team_1" model="crm.team.member">
        <field name="assignment_max">45</field>
        <field name="assignment_domain">[['probability', '>=', 5]]</field>
    </record>

</data></odoo>

```

## File: data\digest_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data noupdate="1">
        <record id="digest.digest_digest_default" model="digest.digest">
            <field name="kpi_crm_lead_created">True</field>
            <field name="kpi_crm_opportunities_won">True</field>
        </record>
    </data>

    <data>
        <record id="digest_tip_crm_0" model="digest.tip">
            <field name="name">Tip: Convert incoming emails into opportunities</field>
            <field name="sequence">200</field>
            <field name="group_id" ref="sales_team.group_sale_salesman_all_leads"/>
            <field name="tip_description" type="html">
<div>
    <t t-set="record" t-value="object.env['crm.team'].search([('alias_name', '!=', 'False')], limit=1)" />
    <p class="tip_title">Tip: Convert incoming emails into opportunities</p>
    <t t-if="record.alias_email">
        <p class="tip_content">Did you know emails sent to <t t-out="record.alias_email"></t> generate opportunities in your pipeline?<br/>
        <a t-attf-href="mailto:{{record.alias_email}}" target="_blank">Try sending an email</a> to your CRM. This email address is configurable by sales team members.</p>
    </t>
    <t t-else="">
        <p class="tip_content">Did you know emails sent to a Sales Team alias generate opportunities in your pipeline?</p>
    </t>
</div>
            </field>
        </record>
        <record id="digest_tip_crm_1" model="digest.tip">
            <field name="name">Tip: Did you know Odoo has built-in lead mining?</field>
            <field name="sequence">1500</field>
            <field name="group_id" ref="sales_team.group_sale_salesman_all_leads"/>
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Did you know Odoo has built-in lead mining?</p>
    <p class="tip_content">For a sales team, there is nothing worse than being dry on leads. Fortunately, in just a few clicks, you can generate leads specifically targeted to your needs: company size, industry, etc. To help you test the feature, we offer you 200 credits for free.</p>
    <img src="https://download.odoocdn.com/digests/crm/static/src/img/milk-generate-leads.gif" width="540" class="illustration_border" />
</div>
            </field>
        </record>
        <record id="digest_tip_crm_2" model="digest.tip">
            <field name="name">Tip: Opportunity win rate is predicted with AI</field>
            <field name="sequence">1700</field>
            <field name="group_id" ref="sales_team.group_sale_salesman_all_leads"/>
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Opportunity win rate is predicted with AI</p>
    <p class="tip_content">Odoo's artificial intelligence engine predicts the success rate of each opportunity based on your history. You can always update the success rate manually, but if you let Odoo do the job the score is updated while the opportunity moves forward in your sales cycle.</p>
    <img src="https://download.odoocdn.com/digests/crm/static/src/img/milk-probability-rate.gif" width="540" class="illustration_border" />
</div>
            </field>
        </record>
        <record id="digest_tip_crm_3" model="digest.tip">
            <field name="name">Tip: Manage your pipeline</field>
            <field name="sequence">2600</field>
            <field name="group_id" ref="sales_team.group_sale_salesman_all_leads"/>
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Manage your pipeline</p>
    <p class="tip_content">A great tip to boost sales efficiency is to always define a next step on each opportunity. To manage ongoing activities, click on any status of the progress bar to filter opportunities based on their next activities' status. Click on the grey area of the progress bar to see all opportunities that have no next activity.</p>
    <img src="https://download.odoocdn.com/digests/crm/static/src/img/milk-pipeline-progress.gif" width="540" class="illustration_border" />
</div>
            </field>
        </record>
        <record id="digest_tip_crm_4" model="digest.tip">
            <field name="name">Tip: Do not waste time recording customers' data</field>
            <field name="sequence">2800</field>
            <field name="group_id" ref="sales_team.group_sale_salesman_all_leads"/>
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Do not waste time recording customers' data</p>
    <p class="tip_content">Did you know you can search a company by name or VAT number to instantly fill in all its data? Odoo autocompletes everything for you: logo, address, company size, business information, social media accounts, etc.</p>
    <img src="https://download.odoocdn.com/digests/crm/static/src/img/milk-autofill.gif" width="540" class="illustration_border" />
</div>
            </field>
        </record>
        <record id="digest_tip_crm_5" model="digest.tip">
            <field name="name">Tip: Turn a selection of opportunities into a map</field>
            <field name="sequence">3000</field>
            <field name="group_id" ref="sales_team.group_sale_salesman_all_leads"/>
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Turn a selection of opportunities into a map</p>
    <p class="tip_content">Did you know you can turn a list of opportunities into a map view, using the top-right map icon? A lot of screens in Odoo can be turned into a map: tasks, contacts, delivery orders, etc.</p>
    <img src="https://download.odoocdn.com/digests/crm/static/src/img/milk-mapview-toggle.gif" width="540" class="illustration_border" />
</div>
            </field>
        </record>
    </data>
</odoo>

```

## File: data\ir_action_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!--
        'Mark as Lost' in action dropdown
    -->
    <record id="action_your_pipeline" model="ir.actions.server">
        <field name="name">Crm: My Pipeline</field>
        <field name="model_id" ref="crm.model_crm_team"/>
        <field name="state">code</field>
        <field name="groups_id"  eval="[(4, ref('base.group_user'))]"/>
        <field name="code">action = model.action_your_pipeline()</field>
    </record>

    <record id="action_opportunity_forecast" model="ir.actions.server">
        <field name="name">Crm: Forecast</field>
        <field name="model_id" ref="crm.model_crm_team"/>
        <field name="state">code</field>
        <field name="groups_id"  eval="[(4, ref('base.group_user'))]"/>
        <field name="code">action = model.action_opportunity_forecast()</field>
    </record>

</odoo>

```

## File: data\ir_cron_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <record forcecreate="True" id="ir_cron_crm_lead_assign" model="ir.cron">
        <field name="name">CRM: Lead Assignment</field>
        <field name="model_id" ref="crm.model_crm_team"/>
        <field name="state">code</field>
        <field name="code">model._cron_assign_leads()</field>
        <field name="active" eval="False"/>
        <field name="user_id" ref="base.user_root"/>
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="numbercall">-1</field>
        <field name="doall" eval="False"/>
    </record>
</data></odoo>

```

## File: data\mail_activity_type_demo.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="mail_activity_demo_followup_quote" model="mail.activity.type">
        <field name="name">Follow-up Quote</field>
        <field name="icon">fa-file-text-o</field>
        <field name="res_model">crm.lead</field>
        <field name="delay_count">30</field>
    </record>
    <record id="mail_activity_demo_make_quote" model="mail.activity.type">
        <field name="name">Make Quote</field>
        <field name="icon">fa-file-text-o</field>
        <field name="res_model">crm.lead</field>
        <field name="delay_count">15</field>
    </record>

    <record id="mail_activity_demo_call_demo" model="mail.activity.type">
        <field name="name">Call for Demo</field>
        <field name="icon">fa-phone</field>
        <field name="res_model">crm.lead</field>
        <field name="delay_count">10</field>
        <field name="category">phonecall</field>
    </record>

    <record id="mail_activity_type_demo_email_with_template" model="mail.activity.type">
        <field name="name">Email: Welcome Demo</field>
        <field name="icon">fa-envelope</field>
        <field name="res_model">crm.lead</field>
        <field name="mail_template_ids" eval="[(4, ref('crm.mail_template_demo_crm_lead'))]"/>
    </record>
</odoo>

```

## File: data\mail_message_subtype_data.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">
        <!-- CRM-related subtypes for messaging / Chatter -->
        <record id="mt_lead_create" model="mail.message.subtype">
            <field name="name">Opportunity Created</field>
            <field name="hidden" eval="True"/>
            <field name="res_model">crm.lead</field>
            <field name="default" eval="False"/>
            <field name="description">Lead/Opportunity created</field>
        </record>
        <record id="mt_lead_stage" model="mail.message.subtype">
            <field name="name">Stage Changed</field>
            <field name="res_model">crm.lead</field>
            <field name="default" eval="False"/>
            <field name="description">Stage changed</field>
        </record>
        <record id="mt_lead_won" model="mail.message.subtype">
            <field name="name">Opportunity Won</field>
            <field name="res_model">crm.lead</field>
            <field name="default" eval="False"/>
            <field name="description">Opportunity won</field>
        </record>
        <record id="mt_lead_lost" model="mail.message.subtype">
            <field name="name">Opportunity Lost</field>
            <field name="res_model">crm.lead</field>
            <field name="default" eval="False"/>
            <field name="description">Opportunity lost</field>
        </record>
        <record id="mt_lead_restored" model="mail.message.subtype">
            <field name="name">Opportunity Restored</field>
            <field name="res_model">crm.lead</field>
            <field name="default" eval="False"/>
            <field name="description">Opportunity restored</field>
        </record>
        <!-- Salesteam-related subtypes for messaging / Chatter -->
        <record id="mt_salesteam_lead" model="mail.message.subtype">
            <field name="name">Opportunity Created</field>
            <field name="sequence">10</field>
            <field name="res_model">crm.team</field>
            <field name="default" eval="True"/>
            <field name="parent_id" ref="mt_lead_create"/>
            <field name="relation_field">team_id</field>
        </record>
        <record id="mt_salesteam_lead_stage" model="mail.message.subtype">
            <field name="name">Opportunity Stage Changed</field>
            <field name="sequence">11</field>
            <field name="res_model">crm.team</field>
            <field name="parent_id" ref="mt_lead_stage"/>
            <field name="relation_field">team_id</field>
        </record>
        <record id="mt_salesteam_lead_won" model="mail.message.subtype">
            <field name="name">Opportunity Won</field>
            <field name="sequence">12</field>
            <field name="res_model">crm.team</field>
            <field name="parent_id" ref="mt_lead_won"/>
            <field name="relation_field">team_id</field>
        </record>
        <record id="mt_salesteam_lead_lost" model="mail.message.subtype">
            <field name="name">Opportunity Lost</field>
            <field name="sequence">13</field>
            <field name="res_model">crm.team</field>
            <field name="default" eval="False"/>
            <field name="parent_id" ref="mt_lead_lost"/>
            <field name="relation_field">team_id</field>
        </record>
        <record id="mt_salesteam_lead_restored" model="mail.message.subtype">
            <field name="name">Opportunity Restored</field>
            <field name="sequence">14</field>
            <field name="res_model">crm.team</field>
            <field name="default" eval="False"/>
            <field name="parent_id" ref="mt_lead_restored"/>
            <field name="relation_field">team_id</field>
        </record>
    </data>
</odoo>

```

## File: data\mail_template_demo.xml

```xml
<?xml version="1.0"?>
<odoo><data noupdate="1">

    <record id="mail_template_demo_crm_lead" model="mail.template">
        <field name="name">Welcome Demo</field>
        <field name="model_id" ref="crm.model_crm_lead"/>
        <field name="partner_to">{{ object.partner_id != False and object.partner_id.id }}</field>
        <field name="email_to">{{ (not object.partner_id and object.email_from) }}</field>
        <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 24px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="100%" style="background-color: white; padding: 0; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Your Lead/Opportunity</span><br/>
                    <span style="font-size: 20px; font-weight: bold;" t-out="object.name or ''">Interest in your products</span>
                </td><td valign="middle" align="right">
                    <img t-attf-src="/logo.png?company={{ object.company_id.id }}" style="padding: 0px; margin: 0px; height: 48px;" t-att-alt="object.company_id.name"/>
                </td></tr>
                <tr><td colspan="2" style="text-align:center;">
                    <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin:4px 0px 32px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- CONTENT -->
    <tr>
        <td style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr>
                    <td valign="top" style="font-size: 13px;">
                        <div>
                            Hi <t t-out="object.partner_id and object.partner_id.name or ''">Deco Addict</t>,<br/><br/>
                            Welcome to <t t-out="object.company_id.name or ''">My Company (San Francisco)</t>.
                            It's great to meet you! Now that you're on board, you'll discover what <t t-out="object.company_id.name or ''">My Company (San Francisco)</t> has to offer. My name is <t t-out="object.user_id.name or ''">Marc Demo</t> and I'll help you get the most out of Odoo. Could we plan a quick demo soon?<br/>
                            Feel free to reach out at any time!<br/><br/>
                            Best,<br/>
                            <t t-if="object.user_id">
                                <b><t t-out="object.user_id.name or ''">Marc Demo</t></b>
                                <br/>Email: <t t-out="object.user_id.email or ''">mark.brown23@example.com</t>
                                <br/>Phone: <t t-out="object.user_id.phone or ''">+1 650-123-4567</t>
                            </t>
                            <t t-else="">
                                <t t-out="object.company_id.name or ''">My Company (San Francisco)</t>
                            </t>
                        </div>
                    </td>
                </tr>
            </table>
        </td>
    </tr>
    <!-- FOOTER -->
    <tr>
        <td align="center" style="min-width: 590px; padding: 0 8px 0 8px; font-size:11px;">
            <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 4px 0px;"/>
            <b t-out="object.company_id.name or ''">My Company (San Francisco)</b><br/>
            <div style="color: #999999;">
                <t t-out="object.company_id.phone or ''">+1 650-123-4567</t>
                <t t-if="object.company_id.email">
                    | <a t-attf-href="'mailto:%s' % {{ object.company_id.email }}" style="text-decoration:none; color: #999999;" t-out="object.company_id.email or ''">info@yourcompany.com</a>
                </t>
                <t t-if="object.company_id.website">
                    | <a t-attf-href="'%s' % {{ object.company_id.website }}" style="text-decoration:none; color: #999999;" t-out="object.company_id.website or ''">http://www.example.com</a>
                </t>
            </div>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- POWERED BY -->
<tr><td align="center" style="min-width: 590px;">
    Powered by <a target="_blank" href="https://www.odoo.com?utm_source=db&amp;utm_medium=email" style="color: #875A7B;">Odoo</a>
</td></tr>
</table>
        </field>
        <field name="lang">{{ object.partner_id.lang }}</field>
        <field name="auto_delete" eval="True"/>
    </record>

</data></odoo>

```

## File: doc\changelog.rst

```rst
.. _changelog:

Changelog
=========

`trunk (saas-2)`
----------------

- Stage/state update

  - ``crm.lead``: removed ``state`` field. Added ``date_last_stage_update`` field
    holding last stage_id modification. Updated reports.
  - ``crm.case.stage``: removed ``state`` field.

- ``crm``, ``crm_claim``: removed inheritance from ``base_stage`` class. Missing
  methods have been added into ``crm`` and ``crm_claim``. Also removed inheritance
  in ``crm_helpdesk`` because it uses states, not stages.

```

## File: doc\index.rst

```rst
CRM module documentation
========================

CRM documentation topics
'''''''''''''''''''''''''

.. toctree::
   :maxdepth: 1
   
   stage_status.rst

Changelog
'''''''''

.. toctree::
   :maxdepth: 1

   changelog.rst

```

## File: doc\stage_status.rst

```rst
.. _stage_status:

Stage and Status
================

.. versionchanged:: 8.0 saas-2 state/stage cleaning

Stage
+++++

This revision removed the concept of state on crm.lead objects. The ``state``
field has been totally removed and replaced by stages, using ``stage_id``. The
following models are impacted:

 - ``crm.lead`` now use only stages. However conventions still exist about
   'New', 'Won' and 'Lost' stages. Those conventions are:

   - ``new``: ``stage_id and stage_id.sequence = 1``
   - ``won``: ``stage_id and stage_id.probability = 100 and stage_id.on_change = True``
   - ``lost``: ``stage_id and stage_id.probability = 0 and stage_id.on_change = True
     and stage_id.sequence != 1``

 - ``crm.case.stage`` do not have any ``state`` field anymore. 
 - ``crm.lead.report`` do not have any ``state`` field anymore. 

By default a newly created lead is in a new stage. It means that it will
fetch the stage having ``sequence = 1``. Stage mangement is done using the
kanban view or the clikable statusbar. It is not done using buttons anymore.

Stage analysis
++++++++++++++

Stage analysis can be performed using the newly introduced ``date_last_stage_update``
datetime field. This field is updated everytime ``stage_id`` is updated.

``crm.lead.report`` model also uses the ``date_last_stage_update`` field.
This allows to group and analyse the time spend in the various stages.

Open / Assignment date
+++++++++++++++++++++++

The ``date_open`` field meaning has been updated. It is now set when the ``user_id``
(responsible) is set. It is therefore the assignment date.

Subtypes
++++++++

The following subtypes are triggered on ``crm.lead``:

 - ``mt_lead_create``: new leads. Condition: ``obj.probability == 0 and obj.stage_id
   and obj.stage_id.sequence == 1``
 - ``mt_lead_stage``: stage changed. Condition: ``(obj.stage_id and obj.stage_id.sequence != 1)
   and obj.probability < 100``
 - ``mt_lead_won``: lead/oportunity is won. condition: `` obj.probability == 100
   and obj.stage_id and obj.stage_id.on_change``
 - ``mt_lead_lost``: lead/opportunity is lost. Condition: ``obj.probability == 0
   and obj.stage_id and obj.stage_id.sequence != 1'``


Those subtypes are also available on the ``crm.case.section`` model and are used
for the auto subscription.

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
        if self.env.context.get('default_opportunity_id'):
            self = self.with_context(
                default_res_model_id=self.env.ref('crm.model_crm_lead').id,
                default_res_id=self.env.context['default_opportunity_id']
            )
        defaults = super(CalendarEvent, self).default_get(fields)

        # sync res_model / res_id to opportunity id (aka creating meeting from lead chatter)
        if 'opportunity_id' not in defaults:
            if self._is_crm_lead(defaults, self.env.context):
                defaults['opportunity_id'] = defaults.get('res_id', False) or self.env.context.get('default_res_id', False)

        return defaults

    opportunity_id = fields.Many2one(
        'crm.lead', 'Opportunity', domain="[('type', '=', 'opportunity')]",
        index=True, ondelete='set null')

    def _compute_is_highlighted(self):
        super(CalendarEvent, self)._compute_is_highlighted()
        if self.env.context.get('active_model') == 'crm.lead':
            opportunity_id = self.env.context.get('active_id')
            for event in self:
                if event.opportunity_id.id == opportunity_id:
                    event.is_highlighted = True

    @api.model_create_multi
    def create(self, vals):
        events = super(CalendarEvent, self).create(vals)
        for event in events:
            if event.opportunity_id and not event.activity_ids:
                event.opportunity_id.log_meeting(event)
        return events

    def _is_crm_lead(self, defaults, ctx=None):
        """
            This method checks if the concerned model is a CRM lead.
            The information is not always in the defaults values,
            this is why it is necessary to check the context too.
        """
        res_model = defaults.get('res_model', False) or ctx and ctx.get('default_res_model')
        res_model_id = defaults.get('res_model_id', False) or ctx and ctx.get('default_res_model_id')

        return res_model and res_model == 'crm.lead' or res_model_id and self.env['ir.model'].sudo().browse(res_model_id).model == 'crm.lead'

```

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pytz
import threading
from ast import literal_eval
from collections import OrderedDict, defaultdict
from datetime import date, datetime, timedelta
from markupsafe import Markup
from psycopg2 import sql

from odoo import api, fields, models, tools, SUPERUSER_ID
from odoo.addons.iap.tools import iap_tools
from odoo.addons.mail.tools import mail_validation
from odoo.addons.phone_validation.tools import phone_validation
from odoo.exceptions import UserError, AccessError
from odoo.osv import expression
from odoo.tools.translate import _
from odoo.tools import date_utils, email_split, is_html_empty, groupby, parse_contact_from_email
from odoo.tools.misc import get_lang

from . import crm_stage

_logger = logging.getLogger(__name__)


CRM_LEAD_FIELDS_TO_MERGE = [
    # UTM mixin
    'campaign_id',
    'medium_id',
    'source_id',
    # Mail mixin
    'email_cc',
    # description
    'name',
    'user_id',
    'color',
    'company_id',
    'lang_id',
    'team_id',
    'referred',
    # pipeline
    'stage_id',
    # revenues
    'expected_revenue',
    'recurring_plan',
    'recurring_revenue',
    # dates
    'create_date',
    'date_automation_last',
    'date_deadline',
    # partner / contact
    'partner_id',
    'title',
    'partner_name',
    'contact_name',
    'email_from',
    'function',
    'mobile',
    'phone',
    'website',
]

# Subset of partner fields: sync any of those
PARTNER_FIELDS_TO_SYNC = [
    'lang',
    'mobile',
    'title',
    'function',
    'website',
]

# Subset of partner fields: sync all or none to avoid mixed addresses
PARTNER_ADDRESS_FIELDS_TO_SYNC = [
    'street',
    'street2',
    'city',
    'zip',
    'state_id',
    'country_id',
]

# Those values have been determined based on benchmark to minimise
# computation time, number of transaction and transaction time.
PLS_COMPUTE_BATCH_STEP = 50000  # odoo.models.PREFETCH_MAX = 1000 but larger cluster can speed up global computation
PLS_UPDATE_BATCH_STEP = 5000


class Lead(models.Model):
    _name = "crm.lead"
    _description = "Lead/Opportunity"
    _order = "priority desc, id desc"
    _inherit = ['mail.thread.cc',
                'mail.thread.blacklist',
                'mail.thread.phone',
                'mail.activity.mixin',
                'utm.mixin',
                'format.address.mixin',
                'mail.tracking.duration.mixin',
               ]
    _primary_email = 'email_from'
    _check_company_auto = True
    _track_duration_field = 'stage_id'

    # Description
    name = fields.Char(
        'Opportunity', index='trigram', required=True,
        compute='_compute_name', readonly=False, store=True)
    user_id = fields.Many2one(
        'res.users', string='Salesperson', default=lambda self: self.env.user,
        domain="[('share', '=', False)]",
        check_company=True, index=True, tracking=True)
    user_company_ids = fields.Many2many(
        'res.company', compute='_compute_user_company_ids',
        help='UX: Limit to lead company or all if no company')
    team_id = fields.Many2one(
        'crm.team', string='Sales Team', check_company=True, index=True, tracking=True,
        compute='_compute_team_id', ondelete="set null", readonly=False, store=True, precompute=True)
    lead_properties = fields.Properties(
        'Properties', definition='team_id.lead_properties_definition',
        copy=True)
    company_id = fields.Many2one(
        'res.company', string='Company', index=True,
        compute='_compute_company_id', readonly=False, store=True)
    referred = fields.Char('Referred By')
    description = fields.Html('Notes')
    active = fields.Boolean('Active', default=True, tracking=True)
    type = fields.Selection([
        ('lead', 'Lead'), ('opportunity', 'Opportunity')], required=True, tracking=15, index=True,
        default=lambda self: 'lead' if self.env['res.users'].has_group('crm.group_use_lead') else 'opportunity')
    # Pipeline management
    priority = fields.Selection(
        crm_stage.AVAILABLE_PRIORITIES, string='Priority', index=True,
        default=crm_stage.AVAILABLE_PRIORITIES[0][0])
    stage_id = fields.Many2one(
        'crm.stage', string='Stage', index=True, tracking=True,
        compute='_compute_stage_id', readonly=False, store=True,
        copy=False, group_expand='_read_group_stage_ids', ondelete='restrict',
        domain="['|', ('team_id', '=', False), ('team_id', '=', team_id)]")
    kanban_state = fields.Selection([
        ('grey', 'No next activity planned'),
        ('red', 'Next activity late'),
        ('green', 'Next activity is planned')], string='Kanban State',
        compute='_compute_kanban_state')
    tag_ids = fields.Many2many(
        'crm.tag', 'crm_tag_rel', 'lead_id', 'tag_id', string='Tags',
        help="Classify and analyze your lead/opportunity categories like: Training, Service")
    color = fields.Integer('Color Index', default=0)
    # Revenues
    expected_revenue = fields.Monetary('Expected Revenue', currency_field='company_currency', tracking=True)
    prorated_revenue = fields.Monetary('Prorated Revenue', currency_field='company_currency', store=True, compute="_compute_prorated_revenue")
    recurring_revenue = fields.Monetary('Recurring Revenues', currency_field='company_currency', tracking=True)
    recurring_plan = fields.Many2one('crm.recurring.plan', string="Recurring Plan")
    recurring_revenue_monthly = fields.Monetary('Expected MRR', currency_field='company_currency', store=True,
                                                compute="_compute_recurring_revenue_monthly")
    recurring_revenue_monthly_prorated = fields.Monetary('Prorated MRR', currency_field='company_currency', store=True,
                                                         compute="_compute_recurring_revenue_monthly_prorated")
    recurring_revenue_prorated = fields.Monetary('Prorated Recurring Revenues', currency_field='company_currency',
                                                 compute="_compute_recurring_revenue_prorated", store=True)
    company_currency = fields.Many2one("res.currency", string='Currency', compute="_compute_company_currency", compute_sudo=True)
    # Dates
    date_closed = fields.Datetime('Closed Date', readonly=True, copy=False)
    date_automation_last = fields.Datetime('Last Action', readonly=True)
    date_open = fields.Datetime(
        'Assignment Date', compute='_compute_date_open', readonly=True, store=True)
    day_open = fields.Float('Days to Assign', compute='_compute_day_open', store=True)
    day_close = fields.Float('Days to Close', compute='_compute_day_close', store=True)
    date_last_stage_update = fields.Datetime(
        'Last Stage Update', compute='_compute_date_last_stage_update', index=True, readonly=True, store=True)
    date_conversion = fields.Datetime('Conversion Date', readonly=True)
    date_deadline = fields.Date('Expected Closing', help="Estimate of the date on which the opportunity will be won.")
    # Customer / contact
    partner_id = fields.Many2one(
        'res.partner', string='Customer', check_company=True, index=True, tracking=10,
        help="Linked partner (optional). Usually created when converting the lead. You can find a partner by its Name, TIN, Email or Internal Reference.")
    partner_is_blacklisted = fields.Boolean('Partner is blacklisted', related='partner_id.is_blacklisted', readonly=True)
    contact_name = fields.Char(
        'Contact Name', index='trigram', tracking=30,
        compute='_compute_contact_name', readonly=False, store=True)
    partner_name = fields.Char(
        'Company Name', index='trigram', tracking=20,
        compute='_compute_partner_name', readonly=False, store=True,
        help='The name of the future partner company that will be created while converting the lead into opportunity')
    function = fields.Char('Job Position', compute='_compute_function', readonly=False, store=True)
    title = fields.Many2one('res.partner.title', string='Title', compute='_compute_title', readonly=False, store=True)
    email_from = fields.Char(
        'Email', tracking=40, index='trigram',
        compute='_compute_email_from', inverse='_inverse_email_from', readonly=False, store=True)
    email_normalized = fields.Char(index='trigram')  # inherited via mail.thread.blacklist
    email_domain_criterion = fields.Char(
        string='Email Domain Criterion',
        compute="_compute_email_domain_criterion",
        index='btree_not_null',  # used for exact match, void value do not matter
        store=True,
        unaccent=False,  # normalized, exact matching
    )
    phone = fields.Char(
        'Phone', tracking=50,
        compute='_compute_phone', inverse='_inverse_phone', readonly=False, store=True)
    mobile = fields.Char('Mobile', compute='_compute_mobile', readonly=False, store=True)
    phone_sanitized = fields.Char(index='btree_not_null')  # inherited via mail.thread.phone
    phone_state = fields.Selection([
        ('correct', 'Correct'),
        ('incorrect', 'Incorrect')], string='Phone Quality', compute="_compute_phone_state", store=True)
    email_state = fields.Selection([
        ('correct', 'Correct'),
        ('incorrect', 'Incorrect')], string='Email Quality', compute="_compute_email_state", store=True)
    website = fields.Char('Website', help="Website of the contact", compute="_compute_website", readonly=False, store=True)
    lang_id = fields.Many2one(
        'res.lang', string='Language',
        compute='_compute_lang_id', readonly=False, store=True)
    lang_code = fields.Char(related='lang_id.code')
    lang_active_count = fields.Integer(compute='_compute_lang_active_count')
    # Address fields
    street = fields.Char('Street', compute='_compute_partner_address_values', readonly=False, store=True)
    street2 = fields.Char('Street2', compute='_compute_partner_address_values', readonly=False, store=True)
    zip = fields.Char('Zip', change_default=True, compute='_compute_partner_address_values', readonly=False, store=True)
    city = fields.Char('City', compute='_compute_partner_address_values', readonly=False, store=True)
    state_id = fields.Many2one(
        "res.country.state", string='State',
        compute='_compute_partner_address_values', readonly=False, store=True,
        domain="[('country_id', '=?', country_id)]")
    country_id = fields.Many2one(
        'res.country', string='Country',
        compute='_compute_partner_address_values', readonly=False, store=True)
    # Probability (Opportunity only)
    probability = fields.Float(
        'Probability', group_operator="avg", copy=False,
        compute='_compute_probabilities', readonly=False, store=True)
    automated_probability = fields.Float('Automated Probability', compute='_compute_probabilities', readonly=True, store=True)
    is_automated_probability = fields.Boolean('Is automated probability?', compute="_compute_is_automated_probability")
    # Won/Lost
    lost_reason_id = fields.Many2one(
        'crm.lost.reason', string='Lost Reason',
        index=True, ondelete='restrict', tracking=True)
    # Statistics
    calendar_event_ids = fields.One2many('calendar.event', 'opportunity_id', string='Meetings')
    duplicate_lead_ids = fields.Many2many("crm.lead", compute="_compute_potential_lead_duplicates", string="Potential Duplicate Lead", context={"active_test": False})
    duplicate_lead_count = fields.Integer(compute="_compute_potential_lead_duplicates", string="Potential Duplicate Lead Count")
    meeting_display_date = fields.Date(compute="_compute_meeting_display")
    meeting_display_label = fields.Char(compute="_compute_meeting_display")
    # UX
    partner_email_update = fields.Boolean('Partner Email will Update', compute='_compute_partner_email_update')
    partner_phone_update = fields.Boolean('Partner Phone will Update', compute='_compute_partner_phone_update')
    is_partner_visible = fields.Boolean('Is Partner Visible', compute='_compute_is_partner_visible')
    # UTMs - enforcing the fact that we want to 'set null' when relation is unlinked
    campaign_id = fields.Many2one(ondelete='set null')
    medium_id = fields.Many2one(ondelete='set null')
    source_id = fields.Many2one(ondelete='set null')

    _sql_constraints = [
        ('check_probability', 'check(probability >= 0 and probability <= 100)', 'The probability of closing the deal should be between 0% and 100%!')
    ]

    @api.depends('activity_date_deadline')
    def _compute_kanban_state(self):
        today = date.today()
        for lead in self:
            kanban_state = 'grey'
            if lead.activity_date_deadline:
                lead_date = fields.Date.from_string(lead.activity_date_deadline)
                if lead_date >= today:
                    kanban_state = 'green'
                else:
                    kanban_state = 'red'
            lead.kanban_state = kanban_state

    @api.depends('company_id')
    def _compute_user_company_ids(self):
        all_companies = self.env['res.company'].search([])
        for lead in self:
            if not lead.company_id:
                lead.user_company_ids = all_companies
            else:
                lead.user_company_ids = lead.company_id

    @api.depends('company_id')
    def _compute_company_currency(self):
        for lead in self:
            if not lead.company_id:
                lead.company_currency = self.env.company.currency_id
            else:
                lead.company_currency = lead.company_id.currency_id

    @api.depends('user_id', 'type')
    def _compute_team_id(self):
        """ When changing the user, also set a team_id or restrict team id
        to the ones user_id is member of. """
        for lead in self:
            # setting user as void should not trigger a new team computation
            if not lead.user_id:
                continue
            user = lead.user_id
            if lead.team_id and user in (lead.team_id.member_ids | lead.team_id.user_id):
                continue
            team_domain = [('use_leads', '=', True)] if lead.type == 'lead' else [('use_opportunities', '=', True)]
            team = self.env['crm.team']._get_default_team_id(user_id=user.id, domain=team_domain)
            if lead.team_id != team:
                lead.team_id = team.id

    @api.depends('user_id', 'team_id', 'partner_id')
    def _compute_company_id(self):
        """ Compute company_id coherency. """
        for lead in self:
            proposal = lead.company_id

            # invalidate wrong configuration
            if proposal:
                # company not in responsible companies
                if lead.user_id and proposal not in lead.user_id.company_ids:
                    proposal = False
                # inconsistent
                elif lead.team_id.company_id and proposal != lead.team_id.company_id:
                    proposal = False
                # void company on team and no assignee
                elif lead.team_id and not lead.team_id.company_id and not lead.user_id:
                    proposal = False
                # no user and no team -> void company and let assignment do its job
                # unless customer has a company
                elif not lead.team_id and not lead.user_id and \
                        (not lead.partner_id or lead.partner_id.company_id != proposal):
                    proposal = False

            # propose a new company based on team > user (respecting context) > partner
            if not proposal:
                if lead.team_id.company_id:
                    lead.company_id = lead.team_id.company_id
                elif lead.user_id:
                    if self.env.company in lead.user_id.company_ids:
                        lead.company_id = self.env.company
                    else:
                        lead.company_id = lead.user_id.company_id & self.env.companies
                elif lead.partner_id:
                    lead.company_id = lead.partner_id.company_id
                else:
                    lead.company_id = False

    @api.depends('team_id', 'type')
    def _compute_stage_id(self):
        for lead in self:
            if not lead.stage_id:
                lead.stage_id = lead._stage_find(domain=[('fold', '=', False)]).id

    @api.depends('user_id')
    def _compute_date_open(self):
        for lead in self:
            if not lead.date_open and lead.user_id:
                lead.date_open = self.env.cr.now()

    @api.depends('stage_id')
    def _compute_date_last_stage_update(self):
        for lead in self:
            if not lead.date_last_stage_update:
                lead.date_last_stage_update = self.env.cr.now()

    @api.depends('create_date', 'date_open')
    def _compute_day_open(self):
        """ Compute difference between create date and open date """
        leads = self.filtered(lambda l: l.date_open and l.create_date)
        others = self - leads
        others.day_open = None
        for lead in leads:
            date_create = fields.Datetime.from_string(lead.create_date).replace(microsecond=0)
            date_open = fields.Datetime.from_string(lead.date_open)
            lead.day_open = abs((date_open - date_create).days)

    @api.depends('create_date', 'date_closed')
    def _compute_day_close(self):
        """ Compute difference between current date and log date """
        leads = self.filtered(lambda l: l.date_closed and l.create_date)
        others = self - leads
        others.day_close = None
        for lead in leads:
            date_create = fields.Datetime.from_string(lead.create_date)
            date_close = fields.Datetime.from_string(lead.date_closed)
            lead.day_close = abs((date_close - date_create).days)

    @api.depends('partner_id')
    def _compute_name(self):
        for lead in self:
            if not lead.name and lead.partner_id and lead.partner_id.name:
                lead.name = _("%s's opportunity") % lead.partner_id.name

    @api.depends('partner_id')
    def _compute_contact_name(self):
        """ compute the new values when partner_id has changed """
        for lead in self:
            lead.update(lead._prepare_contact_name_from_partner(lead.partner_id))

    @api.depends('partner_id')
    def _compute_partner_name(self):
        """ compute the new values when partner_id has changed """
        for lead in self:
            lead.update(lead._prepare_partner_name_from_partner(lead.partner_id))

    @api.depends('partner_id')
    def _compute_function(self):
        """ compute the new values when partner_id has changed """
        for lead in self:
            if not lead.function or lead.partner_id.function:
                lead.function = lead.partner_id.function

    @api.depends('partner_id')
    def _compute_title(self):
        """ compute the new values when partner_id has changed """
        for lead in self:
            if not lead.title or lead.partner_id.title:
                lead.title = lead.partner_id.title

    @api.depends('partner_id')
    def _compute_mobile(self):
        """ compute the new values when partner_id has changed """
        for lead in self:
            if not lead.mobile or lead.partner_id.mobile:
                lead.mobile = lead.partner_id.mobile

    @api.depends('partner_id')
    def _compute_website(self):
        """ compute the new values when partner_id has changed """
        for lead in self:
            if not lead.website or lead.partner_id.website:
                lead.website = lead.partner_id.website

    @api.depends('partner_id')
    def _compute_lang_id(self):
        """ compute the lang based on partner, erase any value to force the partner
        one if set. """
        # prepare cache
        lang_codes = [code for code in self.mapped('partner_id.lang') if code]
        if lang_codes:
            lang_id_by_code = dict(
                (code, self.env['res.lang']._lang_get_id(code))
                for code in lang_codes
            )
        else:
            lang_id_by_code = {}
        for lead in self.filtered('partner_id'):
            lead.lang_id = lang_id_by_code.get(lead.partner_id.lang, False)

    @api.depends('lang_id')
    def _compute_lang_active_count(self):
        self.lang_active_count = len(self.env['res.lang'].get_installed())

    @api.depends('partner_id')
    def _compute_partner_address_values(self):
        """ Sync all or none of address fields """
        for lead in self:
            lead.update(lead._prepare_address_values_from_partner(lead.partner_id))

    @api.depends('partner_id.email')
    def _compute_email_from(self):
        for lead in self:
            if lead.partner_id.email and lead._get_partner_email_update():
                lead.email_from = lead.partner_id.email

    def _inverse_email_from(self):
        for lead in self:
            if lead._get_partner_email_update():
                lead.partner_id.email = lead.email_from

    @api.depends('email_normalized')
    def _compute_email_domain_criterion(self):
        self.email_domain_criterion = False
        for lead in self.filtered('email_normalized'):
            lead.email_domain_criterion = iap_tools.mail_prepare_for_domain_search(
                lead.email_normalized
            )

    @api.depends('partner_id.phone')
    def _compute_phone(self):
        for lead in self:
            if lead.partner_id.phone and lead._get_partner_phone_update():
                lead.phone = lead.partner_id.phone

    def _inverse_phone(self):
        for lead in self:
            if lead._get_partner_phone_update():
                lead.partner_id.phone = lead.phone

    @api.depends('phone', 'country_id.code')
    def _compute_phone_state(self):
        for lead in self:
            phone_status = False
            if lead.phone:
                country_code = lead.country_id.code if lead.country_id and lead.country_id.code else None
                try:
                    if phone_validation.phone_parse(lead.phone, country_code):  # otherwise library not installed
                        phone_status = 'correct'
                except UserError:
                    phone_status = 'incorrect'
            lead.phone_state = phone_status

    @api.depends('email_from')
    def _compute_email_state(self):
        for lead in self:
            email_state = False
            if lead.email_from:
                email_state = 'incorrect'
                for email in email_split(lead.email_from):
                    if mail_validation.mail_validate(email):
                        email_state = 'correct'
                        break
            lead.email_state = email_state

    @api.depends('probability', 'automated_probability')
    def _compute_is_automated_probability(self):
        """ If probability and automated_probability are equal probability computation
        is considered as automatic, aka probability is sync with automated_probability """
        for lead in self:
            lead.is_automated_probability = tools.float_compare(lead.probability, lead.automated_probability, 2) == 0

    @api.depends(lambda self: ['stage_id', 'team_id'] + self._pls_get_safe_fields())
    def _compute_probabilities(self):
        lead_probabilities = self._pls_get_naive_bayes_probabilities()
        for lead in self:
            if lead.id in lead_probabilities:
                was_automated = lead.active and lead.is_automated_probability
                lead.automated_probability = lead_probabilities[lead.id]
                if was_automated:
                    lead.probability = lead.automated_probability

    @api.depends('expected_revenue', 'probability')
    def _compute_prorated_revenue(self):
        for lead in self:
            lead.prorated_revenue = round((lead.expected_revenue or 0.0) * (lead.probability or 0) / 100.0, 2)

    @api.depends('recurring_revenue', 'recurring_plan.number_of_months')
    def _compute_recurring_revenue_monthly(self):
        for lead in self:
            lead.recurring_revenue_monthly = (lead.recurring_revenue or 0.0) / (lead.recurring_plan.number_of_months or 1)

    @api.depends('recurring_revenue_monthly', 'probability')
    def _compute_recurring_revenue_monthly_prorated(self):
        for lead in self:
            lead.recurring_revenue_monthly_prorated = (lead.recurring_revenue_monthly or 0.0) * (lead.probability or 0) / 100.0

    @api.depends('recurring_revenue', 'probability')
    def _compute_recurring_revenue_prorated(self):
        for lead in self:
            lead.recurring_revenue_prorated = (lead.recurring_revenue or 0.0) * (lead.probability or 0) / 100.0

    @api.depends('calendar_event_ids', 'calendar_event_ids.start')
    def _compute_meeting_display(self):
        now = fields.Datetime.now()
        meeting_data = self.env['calendar.event'].sudo()._read_group([
            ('opportunity_id', 'in', self.ids),
        ], ['opportunity_id'], ['start:array_agg', 'start:max'])
        mapped_data = {
            lead: {
                'last_meeting_date': last_meeting_date,
                'next_meeting_date': min([dt for dt in meeting_start_dates if dt > now] or [False]),
            } for lead, meeting_start_dates, last_meeting_date in meeting_data
        }
        for lead in self:
            lead_meeting_info = mapped_data.get(lead)
            if not lead_meeting_info:
                lead.meeting_display_date = False
                lead.meeting_display_label = _('No Meeting')
            elif lead_meeting_info['next_meeting_date']:
                lead.meeting_display_date = lead_meeting_info['next_meeting_date']
                lead.meeting_display_label = _('Next Meeting')
            else:
                lead.meeting_display_date = lead_meeting_info['last_meeting_date']
                lead.meeting_display_label = _('Last Meeting')

    @api.depends('email_domain_criterion', 'email_normalized', 'partner_id',
                 'phone_sanitized')
    def _compute_potential_lead_duplicates(self):
        """ Override potential lead duplicates computation to be more efficient
        with high lead volume.
        Criterions:
          * email domain exact match;
          * phone_sanitized exact match;
          * same commercial entity;
        """
        SEARCH_RESULT_LIMIT = 21

        def return_if_relevant(model_name, domain):
            """ Returns the recordset obtained by performing a search on the provided
            model with the provided domain if the cardinality of that recordset is
            below a given threshold (i.e: `SEARCH_RESULT_LIMIT`). Otherwise, returns
            an empty recordset of the provided model as it indicates search term
            was not relevant.
            Note: The function will use the administrator privileges to guarantee
            that a maximum amount of leads will be included in the search results
            and transcend multi-company record rules. It also includes archived
            records. Idea is that counter indicates duplicates are present and
            the lead could be escalated to managers.
            """
            model = self.env[model_name].sudo().with_context(active_test=False)
            res = model.search(domain, limit=SEARCH_RESULT_LIMIT)
            return res if len(res) < SEARCH_RESULT_LIMIT else model

        for lead in self:
            lead_id = lead._origin.id if isinstance(lead.id, models.NewId) else lead.id
            common_lead_domain = [
                ('id', '!=', lead_id)
            ]

            duplicate_lead_ids = self.env['crm.lead']

            # check the "company" email domain duplicates
            if lead.email_domain_criterion:
                duplicate_lead_ids |= return_if_relevant('crm.lead', common_lead_domain + [
                    ('email_domain_criterion', '=', lead.email_domain_criterion)
                ])
            # check for "same commercial entity" duplicates
            if lead.partner_id and lead.partner_id.commercial_partner_id:
                duplicate_lead_ids |= lead.with_context(active_test=False).search(common_lead_domain + [
                    ("partner_id", "child_of", lead.partner_id.commercial_partner_id.ids)
                ])
            # check the phone number duplicates, based on phone_sanitized. Only
            # exact matches are found, and the single one stored in phone_sanitized
            # in case phone and mobile are both set.
            if lead.phone_sanitized:
                duplicate_lead_ids |= return_if_relevant('crm.lead', common_lead_domain + [
                    ('phone_sanitized', '=', lead.phone_sanitized)
                ])

            lead.duplicate_lead_ids = duplicate_lead_ids + lead
            lead.duplicate_lead_count = len(duplicate_lead_ids)

    @api.depends('email_from', 'partner_id')
    def _compute_partner_email_update(self):
        for lead in self:
            lead.partner_email_update = lead._get_partner_email_update()

    @api.depends('phone', 'partner_id')
    def _compute_partner_phone_update(self):
        for lead in self:
            lead.partner_phone_update = lead._get_partner_phone_update()

    @api.depends_context('uid')
    @api.depends('partner_id', 'type')
    def _compute_is_partner_visible(self):
        """ When the crm.lead is of type 'lead', we don't want to display the "Customer" field on the form view
        unless it's set (or debug mode).

        Indeed, most of the times leads will not have this information set, since when we assign a Customer we
        usually convert the lead to an opportunity as well.

        This means that on the lead form, we don't want to display this field since it may be misleading for the
        end user.
        When it's set however, we want to display it, mainly because there are a few automatic synchronizations between
        the lead and its partner (phone and email for examples), and this needs to be clear that modifying
        one of those fields will in turn modify the linked partner."""
        is_debug_mode = self.user_has_groups('base.group_no_one')
        for lead in self:
            lead.is_partner_visible = bool(lead.type == 'opportunity' or lead.partner_id or is_debug_mode)

    @api.onchange('phone', 'country_id', 'company_id')
    def _onchange_phone_validation(self):
        if self.phone:
            self.phone = self._phone_format(fname='phone', force_format='INTERNATIONAL') or self.phone

    @api.onchange('mobile', 'country_id', 'company_id')
    def _onchange_mobile_validation(self):
        if self.mobile:
            self.mobile = self._phone_format(fname='mobile', force_format='INTERNATIONAL') or self.mobile

    def _prepare_values_from_partner(self, partner):
        """ Get a dictionary with values coming from partner information to
        copy on a lead. Non-address fields get the current lead
        values to avoid being reset if partner has no value for them. """

        # Sync all address fields from partner, or none, to avoid mixing them.
        values = self._prepare_address_values_from_partner(partner)

        # For other fields, get the info from the partner, but only if set
        values.update({f: partner[f] or self[f] for f in PARTNER_FIELDS_TO_SYNC if f != 'lang'})
        if partner.lang:
            values['lang_id'] = self.env['res.lang']._lang_get_id(partner.lang)

        # Fields with specific logic
        values.update(self._prepare_contact_name_from_partner(partner))
        values.update(self._prepare_partner_name_from_partner(partner))

        return self._convert_to_write(values)

    def _prepare_address_values_from_partner(self, partner):
        # Sync all address fields from partner, or none, to avoid mixing them.
        if any(partner[f] for f in PARTNER_ADDRESS_FIELDS_TO_SYNC):
            values = {f: partner[f] for f in PARTNER_ADDRESS_FIELDS_TO_SYNC}
        else:
            values = {f: self[f] for f in PARTNER_ADDRESS_FIELDS_TO_SYNC}
        return values

    def _prepare_contact_name_from_partner(self, partner):
        contact_name = False if partner.is_company else partner.name
        return {'contact_name': contact_name or self.contact_name}

    def _prepare_partner_name_from_partner(self, partner):
        """ Company name: name of partner parent (if set) or name of partner
        (if company) or company_name of partner (if not a company). """
        partner_name = partner.parent_id.name
        if not partner_name and partner.is_company:
            partner_name = partner.name
        elif not partner_name and partner.company_name:
            partner_name = partner.company_name
        return {'partner_name': partner_name or self.partner_name}

    def _get_partner_email_update(self):
        """Calculate if we should write the email on the related partner. When
        the email of the lead / partner is an empty string, we force it to False
        to not propagate a False on an empty string.

        Done in a separate method so it can be used in both ribbon and inverse
        and compute of email update methods.
        """
        self.ensure_one()
        if self.partner_id and self.email_from != self.partner_id.email:
            lead_email_normalized = tools.email_normalize(self.email_from) or self.email_from or False
            partner_email_normalized = tools.email_normalize(self.partner_id.email) or self.partner_id.email or False
            return lead_email_normalized != partner_email_normalized
        return False

    def _get_partner_phone_update(self):
        """Calculate if we should write the phone on the related partner. When
        the phone of the lead / partner is an empty string, we force it to False
        to not propagate a False on an empty string.

        Done in a separate method so it can be used in both ribbon and inverse
        and compute of phone update methods.
        """
        self.ensure_one()
        if self.partner_id and self.phone != self.partner_id.phone:
            lead_phone_formatted = self._phone_format(fname='phone') or self.phone or False
            partner_phone_formatted = self.partner_id._phone_format(fname='phone') or self.partner_id.phone or False
            return lead_phone_formatted != partner_phone_formatted
        return False

    # ------------------------------------------------------------
    # ORM
    # ------------------------------------------------------------

    def _auto_init(self):
        super()._auto_init()
        tools.create_index(self._cr, 'crm_lead_user_id_team_id_type_index',
                           self._table, ['user_id', 'team_id', 'type'])
        tools.create_index(self._cr, 'crm_lead_create_date_team_id_idx',
                           self._table, ['create_date', 'team_id'])

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if vals.get('website'):
                vals['website'] = self.env['res.partner']._clean_website(vals['website'])
        leads = super(Lead, self).create(vals_list)

        for lead, values in zip(leads, vals_list):
            if any(field in ['active', 'stage_id'] for field in values):
                lead._handle_won_lost(values)

        return leads

    def write(self, vals):
        if vals.get('website'):
            vals['website'] = self.env['res.partner']._clean_website(vals['website'])

        now = self.env.cr.now()
        stage_updated, stage_is_won = False, False
        # stage change (or reset): update date_last_stage_update if at least one
        # lead does not have the same stage
        if 'stage_id' in vals:
            stage_updated = any(lead.stage_id.id != vals['stage_id'] for lead in self)
            if stage_updated:
                vals['date_last_stage_update'] = now
            if stage_updated and vals.get('stage_id'):
                stage = self.env['crm.stage'].browse(vals['stage_id'])
                if stage.is_won:
                    vals.update({'probability': 100, 'automated_probability': 100})
                    stage_is_won = True
        # user change; update date_open if at least one lead does not
        # have the same user
        if 'user_id' in vals and not vals.get('user_id'):
            vals['date_open'] = False
        elif vals.get('user_id'):
            user_updated = any(lead.user_id.id != vals['user_id'] for lead in self)
            if user_updated:
                vals['date_open'] = now

        # stage change with new stage: update probability and date_closed
        if vals.get('probability', 0) >= 100 or not vals.get('active', True):
            vals['date_closed'] = fields.Datetime.now()
        elif vals.get('probability', 0) > 0:
            vals['date_closed'] = False
        elif stage_updated and not stage_is_won and not 'probability' in vals:
            vals['date_closed'] = False

        if any(field in ['active', 'stage_id'] for field in vals):
            self._handle_won_lost(vals)

        if not stage_is_won:
            return super(Lead, self).write(vals)

        # stage change between two won stages: does not change the date_closed
        leads_already_won = self.filtered(lambda lead: lead.stage_id.is_won)
        remaining = self - leads_already_won
        if remaining:
            result = super(Lead, remaining).write(vals)
        if leads_already_won:
            vals.pop('date_closed', False)
            result = super(Lead, leads_already_won).write(vals)
        return result

    @api.model
    def search_fetch(self, domain, field_names, offset=0, limit=None, order=None):
        """ Override to support ordering on my_activity_date_deadline.

        Ordering through web client calls search_read() with an order parameter
        set. Method search_read() then calls search_fetch(). Here we override
        search_fetch() to intercept a search with an order on field
        my_activity_date_deadline. In that case we do the search in two steps.

        First step: fill with deadline-based results

          * Perform a read_group on my activities to get a mapping lead_id / deadline
            Remember date_deadline is required, we always have a value for it. Only
            the earliest deadline per lead is kept.
          * Search leads linked to those activities that also match the asked domain
            and order from the original search request.
          * Results of that search will be at the top of returned results. Use limit
            None because we have to search all leads linked to activities as ordering
            on deadline is done in post processing.
          * Reorder them according to deadline asc or desc depending on original
            search ordering. Finally take only a subset of those leads to fill with
            results matching asked offset / limit.

        Second step: fill with other results. If first step does not gives results
        enough to match offset and limit parameters we fill with a search on other
        leads. We keep the asked domain and ordering while filtering out already
        scanned leads to keep a coherent results.

        All other search and search_read are left untouched by this override to avoid
        side effects. Search_count is not affected by this override.
        """
        if not order or 'my_activity_date_deadline' not in order:
            return super().search_fetch(domain, field_names, offset, limit, order)
        order_items = [order_item.strip().lower() for order_item in (order or self._order).split(',')]

        # Perform a read_group on my activities to get a mapping lead_id / deadline
        # Remember date_deadline is required, we always have a value for it. Only
        # the earliest deadline per lead is kept.
        activity_asc = any('my_activity_date_deadline asc' in item for item in order_items)
        my_lead_activities = self.env['mail.activity']._read_group(
            [('res_model', '=', self._name), ('user_id', '=', self.env.uid)],
            ['res_id'],
            ['date_deadline:min'],
            order='date_deadline:min ASC, res_id',
        )
        my_lead_mapping = dict(my_lead_activities)
        my_lead_ids = list(my_lead_mapping.keys())
        my_lead_domain = expression.AND([[('id', 'in', my_lead_ids)], domain])
        my_lead_order = ', '.join(item for item in order_items if 'my_activity_date_deadline' not in item)

        # Search leads linked to those activities and order them. See docstring
        # of this method for more details.
        search_res = super().search_fetch(my_lead_domain, field_names, order=my_lead_order)
        my_lead_ids_ordered = sorted(search_res.ids, key=lambda lead_id: my_lead_mapping[lead_id], reverse=not activity_asc)
        # keep only requested window (offset + limit, or offset+)
        my_lead_ids_keep = my_lead_ids_ordered[offset:(offset + limit)] if limit else my_lead_ids_ordered[offset:]
        # keep list of already skipped lead ids to exclude them from future search
        my_lead_ids_skip = my_lead_ids_ordered[:(offset + limit)] if limit else my_lead_ids_ordered

        # do not go further if limit is achieved
        if limit and len(my_lead_ids_keep) >= limit:
            return self.browse(my_lead_ids_keep)

        # Fill with remaining leads. If a limit is given, simply remove count of
        # already fetched. Otherwise keep none. If an offset is set we have to
        # reduce it by already fetch results hereabove. Order is updated to exclude
        # my_activity_date_deadline when calling super() .
        lead_limit = (limit - len(my_lead_ids_keep)) if limit else None
        if offset:
            lead_offset = max((offset - len(search_res), 0))
        else:
            lead_offset = 0
        lead_order = ', '.join(item for item in order_items if 'my_activity_date_deadline' not in item)

        other_lead_res = super().search_fetch(
            expression.AND([[('id', 'not in', my_lead_ids_skip)], domain]),
            field_names, lead_offset, lead_limit, lead_order,
        )
        return self.browse(my_lead_ids_keep) + other_lead_res

    def _handle_won_lost(self, vals):
        """ This method handle the state changes :
        - To lost : We need to increment corresponding lost count in scoring frequency table
        - To won : We need to increment corresponding won count in scoring frequency table
        - From lost to Won : We need to decrement corresponding lost count + increment corresponding won count
        in scoring frequency table.
        - From won to lost : We need to decrement corresponding won count + increment corresponding lost count
        in scoring frequency table."""
        Lead = self.env['crm.lead']
        leads_reach_won = Lead
        leads_leave_won = Lead
        leads_reach_lost = Lead
        leads_leave_lost = Lead
        won_stage_ids = self.env['crm.stage'].search([('is_won', '=', True)]).ids
        for lead in self:
            if 'stage_id' in vals:
                if vals['stage_id'] in won_stage_ids:
                    if lead.probability == 0:
                        leads_leave_lost += lead
                    leads_reach_won += lead
                elif lead.stage_id.id in won_stage_ids and lead.active:  # a lead can be lost at won_stage
                    leads_leave_won += lead
            if 'active' in vals:
                if not vals['active'] and lead.active:  # archive lead
                    if lead.stage_id.id in won_stage_ids and lead not in leads_leave_won:
                        leads_leave_won += lead
                    leads_reach_lost += lead
                elif vals['active'] and not lead.active:  # restore lead
                    leads_leave_lost += lead

        leads_reach_won._pls_increment_frequencies(to_state='won')
        leads_leave_won._pls_increment_frequencies(from_state='won')
        leads_reach_lost._pls_increment_frequencies(to_state='lost')
        leads_leave_lost._pls_increment_frequencies(from_state='lost')

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        self.ensure_one()
        # set default value in context, if not already set (Put stage to 'new' stage)
        context = dict(self._context)
        context.setdefault('default_type', self.type)
        context.setdefault('default_team_id', self.team_id.id)
        # Set date_open to today if it is an opp
        default = default or {}
        default['date_open'] = self.env.cr.now() if self.type == 'opportunity' else False
        # Do not assign to an archived user
        if not self.user_id.active:
            default['user_id'] = False
        if not self.env.user.has_group('crm.group_use_recurring_revenues'):
            default['recurring_revenue'] = 0
            default['recurring_plan'] = False
        return super(Lead, self.with_context(context)).copy(default=default)

    def unlink(self):
        """ Update meetings when removing opportunities, otherwise you have
        a link to a record that does not lead anywhere. """
        meetings = self.env['calendar.event'].search([
            ('res_id', 'in', self.ids),
            ('res_model', '=', self._name),
        ])
        if meetings:
            meetings.write({
                'res_id': False,
                'res_model_id': False,
            })
        return super(Lead, self).unlink()

    @api.model
    def _read_group_stage_ids(self, stages, domain, order):
        # retrieve team_id from the context and write the domain
        # - ('id', 'in', stages.ids): add columns that should be present
        # - OR ('fold', '=', False): add default columns that are not folded
        # - OR ('team_ids', '=', team_id), ('fold', '=', False) if team_id: add team columns that are not folded
        team_id = self._context.get('default_team_id')
        if team_id:
            search_domain = ['|', ('id', 'in', stages.ids), '|', ('team_id', '=', False), ('team_id', '=', team_id)]
        else:
            search_domain = ['|', ('id', 'in', stages.ids), ('team_id', '=', False)]

        # perform search
        stage_ids = stages._search(search_domain, order=order, access_rights_uid=SUPERUSER_ID)
        return stages.browse(stage_ids)

    def _stage_find(self, team_id=False, domain=None, order='sequence, id', limit=1):
        """ Determine the stage of the current lead with its teams, the given domain and the given team_id
            :param team_id
            :param domain : base search domain for stage
            :param order : base search order for stage
            :param limit : base search limit for stage
            :returns crm.stage recordset
        """
        # collect all team_ids by adding given one, and the ones related to the current leads
        team_ids = set()
        if team_id:
            team_ids.add(team_id)
        for lead in self:
            if lead.team_id:
                team_ids.add(lead.team_id.id)
        # generate the domain
        if team_ids:
            search_domain = ['|', ('team_id', '=', False), ('team_id', 'in', list(team_ids))]
        else:
            search_domain = [('team_id', '=', False)]
        # AND with the domain in parameter
        if domain:
            search_domain += list(domain)
        # perform search, return the first found
        return self.env['crm.stage'].search(search_domain, order=order, limit=limit)

    # ------------------------------------------------------------
    # ACTIONS
    # ------------------------------------------------------------

    def toggle_active(self):
        """ When archiving: mark probability as 0. When re-activating
        update probability again, for leads and opportunities. """
        res = super(Lead, self).toggle_active()
        activated = self.filtered(lambda lead: lead.active)
        archived = self.filtered(lambda lead: not lead.active)
        if activated:
            activated.write({'lost_reason_id': False})
            activated._compute_probabilities()
        if archived:
            archived.write({'probability': 0, 'automated_probability': 0})
        return res

    def action_set_lost(self, **additional_values):
        """ Lost semantic: probability = 0 or active = False """
        res = self.action_archive()
        if additional_values:
            self.write(dict(additional_values))
        return res

    def action_set_won(self):
        """ Won semantic: probability = 100 (active untouched) """
        self.action_unarchive()
        # group the leads by team_id, in order to write once by values couple (each write leads to frequency increment)
        leads_by_won_stage = {}
        for lead in self:
            won_stages = self._stage_find(domain=[('is_won', '=', True)], limit=None)
            # ABD : We could have a mixed pipeline, with "won" stages being separated by "standard"
            # stages. In the future, we may want to prevent any "standard" stage to have a higher
            # sequence than any "won" stage. But while this is not the case, searching
            # for the "won" stage while alterning the sequence order (see below) will correctly
            # handle such a case :
            #       stage sequence : [x] [x (won)] [y] [y (won)] [z] [z (won)]
            #       when in stage [y] and marked as "won", should go to the stage [y (won)],
            #       not in [x (won)] nor [z (won)]
            stage_id = next((stage for stage in won_stages if stage.sequence > lead.stage_id.sequence), None)
            if not stage_id:
                stage_id = next((stage for stage in reversed(won_stages) if stage.sequence <= lead.stage_id.sequence), won_stages)
            if stage_id in leads_by_won_stage:
                leads_by_won_stage[stage_id] += lead
            else:
                leads_by_won_stage[stage_id] = lead
        for won_stage_id, leads in leads_by_won_stage.items():
            leads.write({'stage_id': won_stage_id.id, 'probability': 100})
        return True

    def action_set_automated_probability(self):
        self.write({'probability': self.automated_probability})

    def action_set_won_rainbowman(self):
        self.ensure_one()
        self.action_set_won()

        message = self._get_rainbowman_message()
        if message:
            return {
                'effect': {
                    'fadeout': 'slow',
                    'message': message,
                    'img_url': '/web/image/%s/%s/image_1024' % (self.team_id.user_id._name, self.team_id.user_id.id) if self.team_id.user_id.image_1024 else '/web/static/img/smile.svg',
                    'type': 'rainbow_man',
                }
            }
        return True

    def get_rainbowman_message(self):
        self.ensure_one()
        if self.stage_id.is_won:
            return self._get_rainbowman_message()
        return False

    def _get_rainbowman_message(self):
        if not self.user_id or not self.team_id:
            return False
        if not self.expected_revenue:
            # Show rainbow man for the first won lead of a salesman, even if expected revenue is not set. It is not
            # very often that leads without revenues are marked won, so simply get count using ORM instead of query
            today = fields.Datetime.today()
            user_won_leads_count = self.search_count([
                ('type', '=', 'opportunity'),
                ('user_id', '=', self.user_id.id),
                ('probability', '=', 100),
                ('date_closed', '>=', date_utils.start_of(today, 'year')),
                ('date_closed', '<', date_utils.end_of(today, 'year')),
            ])
            if user_won_leads_count == 1:
                return _('Go, go, go! Congrats for your first deal.')
            return False

        self.flush_model()  # flush fields to make sure DB is up to date
        query = """
            SELECT
                SUM(CASE WHEN user_id = %(user_id)s THEN 1 ELSE 0 END) as total_won,
                MAX(CASE WHEN date_closed >= CURRENT_DATE - INTERVAL '30 days' AND user_id = %(user_id)s THEN expected_revenue ELSE 0 END) as max_user_30,
                MAX(CASE WHEN date_closed >= CURRENT_DATE - INTERVAL '7 days' AND user_id = %(user_id)s THEN expected_revenue ELSE 0 END) as max_user_7,
                MAX(CASE WHEN date_closed >= CURRENT_DATE - INTERVAL '30 days' AND team_id = %(team_id)s THEN expected_revenue ELSE 0 END) as max_team_30,
                MAX(CASE WHEN date_closed >= CURRENT_DATE - INTERVAL '7 days' AND team_id = %(team_id)s THEN expected_revenue ELSE 0 END) as max_team_7
            FROM crm_lead
            WHERE
                type = 'opportunity'
            AND
                active = True
            AND
                probability = 100
            AND
                DATE_TRUNC('year', date_closed) = DATE_TRUNC('year', CURRENT_DATE)
            AND
                (user_id = %(user_id)s OR team_id = %(team_id)s)
        """
        self.env.cr.execute(query, {'user_id': self.user_id.id,
                                    'team_id': self.team_id.id})
        query_result = self.env.cr.dictfetchone()

        message = False
        if query_result['total_won'] == 1:
            message = _('Go, go, go! Congrats for your first deal.')
        elif query_result['max_team_30'] == self.expected_revenue:
            message = _('Boom! Team record for the past 30 days.')
        elif query_result['max_team_7'] == self.expected_revenue:
            message = _('Yeah! Deal of the last 7 days for the team.')
        elif query_result['max_user_30'] == self.expected_revenue:
            message = _('You just beat your personal record for the past 30 days.')
        elif query_result['max_user_7'] == self.expected_revenue:
            message = _('You just beat your personal record for the past 7 days.')
        return message

    def action_schedule_meeting(self, smart_calendar=True):
        """ Open meeting's calendar view to schedule meeting on current opportunity.

            :param smart_calendar: boolean, to set to False if the view should not try to choose relevant
              mode and initial date for calendar view, see ``_get_opportunity_meeting_view_parameters``
            :return dict: dictionary value for created Meeting view
        """
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("calendar.action_calendar_event")
        partner_ids = self.env.user.partner_id.ids
        if self.partner_id:
            partner_ids.append(self.partner_id.id)
        current_opportunity_id = self.id if self.type == 'opportunity' else False
        action['context'] = {
            'search_default_opportunity_id': current_opportunity_id,
            'default_opportunity_id': current_opportunity_id,
            'default_partner_id': self.partner_id.id,
            'default_partner_ids': partner_ids,
            'default_team_id': self.team_id.id,
            'default_name': self.name,
        }

        # 'Smart' calendar view : get the most relevant time period to display to the user.
        if current_opportunity_id and smart_calendar:
            mode, initial_date = self._get_opportunity_meeting_view_parameters()
            action['context'].update({'default_mode': mode, 'initial_date': initial_date})

        return action

    def _get_opportunity_meeting_view_parameters(self):
        """ Return the most relevant parameters for calendar view when viewing meetings linked to an opportunity.
            If there are any meetings that are not finished yet, only consider those meetings,
            since the user would prefer no to see past meetings. Otherwise, consider all meetings.
            Allday events datetimes are used without taking tz into account.
            -If there is no event, return week mode and false (The calendar will target 'now' by default)
            -If there is only one, return week mode and date of the start of the event.
            -If there are several events entirely on the same week, return week mode and start of first event.
            -Else, return month mode and the date of the start of first event as initial date. (If they are
            on the same month, this will display that month and therefore show all of them, which is expected)

            :return tuple(mode, initial_date)
                - mode: selected mode of the calendar view, 'week' or 'month'
                - initial_date: date of the start of the first relevant meeting. The calendar will target that date.
        """
        self.ensure_one()
        meeting_results = self.env["calendar.event"].search_read([('opportunity_id', '=', self.id)], ['start', 'stop', 'allday'])
        if not meeting_results:
            return "week", False

        user_tz = self.env.user.tz or self.env.context.get('tz')
        user_pytz = pytz.timezone(user_tz) if user_tz else pytz.utc

        # meeting_dts will contain one tuple of datetimes per meeting : (Start, Stop)
        # meetings_dts and now_dt are as per user time zone.
        meeting_dts = []
        now_dt = datetime.now().astimezone(user_pytz).replace(tzinfo=None)

        # When creating an allday meeting, whatever the TZ, it will be stored the same e.g. 00.00.00->23.59.59 in utc or
        # 08.00.00->18.00.00. Therefore we must not put it back in the user tz but take it raw.
        for meeting in meeting_results:
            if meeting.get('allday'):
                meeting_dts.append((meeting.get('start'), meeting.get('stop')))
            else:
                meeting_dts.append((meeting.get('start').astimezone(user_pytz).replace(tzinfo=None),
                                   meeting.get('stop').astimezone(user_pytz).replace(tzinfo=None)))

        # If there are meetings that are still ongoing or to come, only take those.
        unfinished_meeting_dts = [meeting_dt for meeting_dt in meeting_dts if meeting_dt[1] >= now_dt]
        relevant_meeting_dts = unfinished_meeting_dts if unfinished_meeting_dts else meeting_dts
        relevant_meeting_count = len(relevant_meeting_dts)

        if relevant_meeting_count == 1:
            return "week", relevant_meeting_dts[0][0].date()
        else:
            # Range of meetings
            earliest_start_dt = min(relevant_meeting_dt[0] for relevant_meeting_dt in relevant_meeting_dts)
            latest_stop_dt = max(relevant_meeting_dt[1] for relevant_meeting_dt in relevant_meeting_dts)

            # The week start day depends on language. We fetch the week_start of user's language. 1 is monday.
            lang_week_start = self.env["res.lang"].search_read([('code', '=', self.env.user.lang)], ['week_start'])
            # We substract one to make week_start_index range 0-6 instead of 1-7
            week_start_index = int(lang_week_start[0].get('week_start', '1')) - 1

            # We compute the weekday of earliest_start_dt according to week_start_index. earliest_start_dt_index will be 0 if we are on the
            # first day of the week and 6 on the last. weekday() returns 0 for monday and 6 for sunday. For instance, Tuesday in UK is the
            # third day of the week, so earliest_start_dt_index is 2, and remaining_days_in_week includes tuesday, so it will be 5.
            # The first term 7 is there to avoid negative left side on the modulo, improving readability.
            earliest_start_dt_weekday = (7 + earliest_start_dt.weekday() - week_start_index) % 7
            remaining_days_in_week = 7 - earliest_start_dt_weekday

            # We compute the start of the week following the one containing the start of the first meeting.
            next_week_start_date = earliest_start_dt.date() + timedelta(days=remaining_days_in_week)

            # Latest_stop_dt must be before the start of following week. Limit is therefore set at midnight of first day, included.
            meetings_in_same_week = latest_stop_dt <= datetime(next_week_start_date.year, next_week_start_date.month, next_week_start_date.day, 0, 0, 0)

            if meetings_in_same_week:
                return "week", earliest_start_dt.date()
            else:
                return "month", earliest_start_dt.date()

    def action_reschedule_meeting(self):
        self.ensure_one()
        action = self.action_schedule_meeting(smart_calendar=False)
        next_activity = self.activity_ids.filtered(lambda activity: activity.user_id == self.env.user)[:1]
        if next_activity.calendar_event_id:
            action['context']['initial_date'] = next_activity.calendar_event_id.start
        return action

    def action_show_potential_duplicates(self):
        """ Open kanban view to display duplicate leads or opportunity.
            :return dict: dictionary value for created kanban view
        """
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("crm.crm_lead_opportunities")
        action['domain'] = [('id', 'in', self.duplicate_lead_ids.ids)]
        action['context'] = {
            'active_test': False,
            'create': False
        }
        return action

    def action_snooze(self):
        self.ensure_one()
        today = date.today()
        my_next_activity = self.activity_ids.filtered(lambda activity: activity.user_id == self.env.user)[:1]
        if my_next_activity:
            if my_next_activity.date_deadline < today:
                date_deadline = today + timedelta(days=7)
            else:
                date_deadline = my_next_activity.date_deadline + timedelta(days=7)
            my_next_activity.write({
                'date_deadline': date_deadline
            })
        return True

    # ------------------------------------------------------------
    # VIEWS
    # ------------------------------------------------------------

    def redirect_lead_opportunity_view(self):
        self.ensure_one()
        return {
            'name': _('Lead or Opportunity'),
            'view_mode': 'form',
            'res_model': 'crm.lead',
            'domain': [('type', '=', self.type)],
            'res_id': self.id,
            'view_id': False,
            'type': 'ir.actions.act_window',
            'context': {'default_type': self.type}
        }

    @api.model
    def get_empty_list_help(self, help_message):
        """ This method returns the action helpers for the leads. If help is already provided
            on the action, the same is returned. Otherwise, we build the help message which
            contains the alias responsible for creating the lead (if available) and return it.
        """
        if not is_html_empty(help_message):
            return help_message

        help_title, sub_title = "", ""
        if self._context.get('default_type') == 'lead':
            help_title = _('Create a new lead')
        else:
            help_title = _('Create an opportunity to start playing with your pipeline.')
        alias_domain = [
            ('company_id', 'in', [self.env.company.id, False]),
            ('alias_id.alias_name', '!=', False),
            ('alias_id.alias_name', '!=', ''),
            ('alias_id.alias_model_id.model', '=', 'crm.lead'),
        ]
        # sort by use_leads, then by our membership of the team
        alias_records = self.env['crm.team'].search(alias_domain).sorted(
            lambda r: (r.use_leads, self.env.user in r.member_ids), reverse=True
        )
        alias_record = alias_records[0] if alias_records else None
        if alias_record and alias_record.alias_domain and alias_record.alias_name:
            sub_title = Markup(_('Use the <i>New</i> button, or send an email to %(email_link)s to test the email gateway.')) % {
                'email_link': Markup("<b><a href='mailto:%s'>%s</a></b>") % (alias_record.alias_email, alias_record.alias_email),
            }
        return super().get_empty_list_help(
            f'<p class="o_view_nocontent_smiling_face">{help_title}</p><p class="oe_view_nocontent_alias">{sub_title}</p>'
        )

    # ------------------------------------------------------------
    # BUSINESS
    # ------------------------------------------------------------

    def log_meeting(self, meeting):
        """ Log the meeting info with a link to it in the chatter
        :param record meeting: the meeting we want to log
        """
        if not meeting.duration:
            duration = _('unknown')
        else:
            duration = self.env['ir.qweb.field.duration'].value_to_html(meeting.duration, {'unit': 'hour'})
        meeting_usertime = fields.Datetime.to_string(fields.Datetime.context_timestamp(self, meeting.start))
        meeting_time = Markup("<time datetime='%(meeting_start)s+00:00'>%(meeting_user_time)s</time>") % {
            'meeting_start': meeting.start,
            'meeting_user_time': meeting_usertime,
        }
        message = Markup("<p>%(meeting)s<br/>%(subject_string)s %(subject_link)s<br/>%(duration)s<p>") % {
            'meeting': _("Meeting scheduled at %s", meeting_time),
            'subject_string': _("Subject: "),
            'subject_link': meeting._get_html_link(),
            'duration': _("Duration: %s", duration),
        }
        return self.message_post(body=message)

    # ------------------------------------------------------------
    # MERGE AND CONVERT LEADS / OPPORTUNITIES
    # ------------------------------------------------------------

    def _merge_data(self, fnames=None):
        """ Prepare lead/opp data into a dictionary for merging. Different types
            of fields are processed in different ways:
                - text: all the values are concatenated
                - m2m and o2m: those fields aren't processed
                - m2o: the first not null value prevails (the other are dropped)
                - any other type of field: same as m2o

            :param fields: list of fields to process
            :return dict data: contains the merged values of the new opportunity
        """
        if fnames is None:
            fnames = self._merge_get_fields()
        fcallables = self._merge_get_fields_specific()
        address_values = self._merge_get_fields_address()

        # helpers
        def _get_first_not_null(attr, opportunities):
            value = False
            for opp in opportunities:
                if opp[attr]:
                    value = opp[attr].id if isinstance(opp[attr], models.BaseModel) else opp[attr]
                    break
            return value

        # process the field's values
        data = {}
        for field_name in fnames:
            field = self._fields.get(field_name)
            if field is None:
                continue

            fcallable = fcallables.get(field_name)
            if fcallable and callable(fcallable):
                data[field_name] = fcallable(field_name, self)
            elif field_name in address_values:
                data[field_name] = address_values[field_name]
            elif not fcallable and field.type in ('many2many', 'one2many'):
                continue
            else:
                data[field_name] = _get_first_not_null(field_name, self)  # take the first not null

        return data

    def merge_opportunity(self, user_id=False, team_id=False, auto_unlink=True):
        """ Merge opportunities in one. Different cases of merge:
                - merge leads together = 1 new lead
                - merge at least 1 opp with anything else (lead or opp) = 1 new opp
            The resulting lead/opportunity will be the most important one (based on its confidence level)
            updated with values from other opportunities to merge.

        :param user_id : the id of the saleperson. If not given, will be determined by `_merge_data`.
        :param team : the id of the Sales Team. If not given, will be determined by `_merge_data`.

        :return crm.lead record resulting of th merge
        """
        return self._merge_opportunity(user_id=user_id, team_id=team_id, auto_unlink=auto_unlink)

    def _merge_opportunity(self, user_id=False, team_id=False, auto_unlink=True, max_length=5):
        """ Private merging method. This one allows to relax rules on record set
        length allowing to merge more than 5 opportunities at once if requested.
        This should not be called by action buttons.

        See ``merge_opportunity`` for more details. """
        if len(self.ids) <= 1:
            raise UserError(_('Please select more than one element (lead or opportunity) from the list view.'))

        if max_length and len(self.ids) > max_length and not self.env.is_superuser():
            raise UserError(_("To prevent data loss, Leads and Opportunities can only be merged by groups of %(max_length)s.", max_length=max_length))

        opportunities = self._sort_by_confidence_level(reverse=True)

        # get SORTED recordset of head and tail, and complete list
        opportunities_head = opportunities[0]
        opportunities_tail = opportunities[1:]

        # merge all the sorted opportunity. This means the value of
        # the first (head opp) will be a priority.
        merged_data = opportunities._merge_data(self._merge_get_fields())

        # force value for saleperson and Sales Team
        if user_id:
            merged_data['user_id'] = user_id
        if team_id:
            merged_data['team_id'] = team_id

        merged_followers = opportunities_head._merge_followers(opportunities_tail)

        # log merge message
        opportunities_head._merge_log_summary(merged_followers, opportunities_tail)
        # merge other data (mail.message, attachments, ...) from tail into head
        opportunities_head._merge_dependences(opportunities_tail)

        # check if the stage is in the stages of the Sales Team. If not, assign the stage with the lowest sequence
        if merged_data.get('team_id'):
            team_stage_ids = self.env['crm.stage'].search(['|', ('team_id', '=', merged_data['team_id']), ('team_id', '=', False)], order='sequence, id')
            if merged_data.get('stage_id') not in team_stage_ids.ids:
                merged_data['stage_id'] = team_stage_ids[0].id if team_stage_ids else False

        # write merged data into first opportunity; remove some keys if already
        # set on opp to avoid useless recomputes
        if 'user_id' in merged_data and opportunities_head.user_id.id == merged_data['user_id']:
            merged_data.pop('user_id')
        if 'team_id' in merged_data and opportunities_head.team_id.id == merged_data['team_id']:
            merged_data.pop('team_id')
        opportunities_head.write(merged_data)

        # delete tail opportunities
        # we use the SUPERUSER to avoid access rights issues because as the user had the rights to see the records it should be safe to do so
        if auto_unlink:
            opportunities_tail.sudo().unlink()

        return opportunities_head

    def _merge_get_fields_address(self):
        """The address fields are propagated as a whole.

        The address is taken from the lead with the most non-empty address field
        (sorted by highest rank if multiple lead have the same amount of non-empty
        fields).
        """
        source_lead = max(self, key=lambda lead: len(list(
            lead[field] for field in PARTNER_ADDRESS_FIELDS_TO_SYNC
            if lead[field]
        )))
        return {fname: source_lead[fname] for fname in PARTNER_ADDRESS_FIELDS_TO_SYNC}

    def _merge_get_fields_specific(self):
        return {
            'description': lambda fname, leads: '<br/><br/>'.join(desc for desc in leads.mapped('description') if not is_html_empty(desc)),
            'type': lambda fname, leads: 'opportunity' if any(lead.type == 'opportunity' for lead in leads) else 'lead',
            'priority': lambda fname, leads: max(leads.mapped('priority')) if leads else False,
            'tag_ids': lambda fname, leads: leads.mapped('tag_ids'),
            'lost_reason_id': lambda fname, leads:
                False if leads and leads[0].probability
                else next((lead.lost_reason_id for lead in leads if lead.lost_reason_id), False),
        }

    def _merge_get_fields(self):
        return (
            CRM_LEAD_FIELDS_TO_MERGE
            + list(self._merge_get_fields_specific().keys())
            + PARTNER_ADDRESS_FIELDS_TO_SYNC
        )

    def _merge_dependences(self, opportunities):
        """ Merge dependences (messages, attachments,activities, calendar events,
        ...). These dependences will be transfered to `self` considered as the
        master lead.

        :param opportunities : recordset of opportunities to transfer. Does not
          include `self` which is the target crm.lead being the result of the
          merge;
        """
        self.ensure_one()
        self._merge_dependences_history(opportunities)
        self._merge_dependences_attachments(opportunities)
        self._merge_dependences_calendar_events(opportunities)

    def _merge_dependences_history(self, opportunities):
        """ Move history from the given opportunities to the current one. `self`
        is the crm.lead record destination for message of `opportunities`.

        This method moves
          * messages
          * activities

        :param opportunities: see ``_merge_dependences``
        """
        self.ensure_one()
        # sudo usage: because we want to go through all messages, whatever the real ACLs
        # current user has on them
        for opportunity_su in opportunities.sudo():
            for message_su in opportunity_su.message_ids:
                if message_su.subject:
                    subject = _("From %(source_name)s: %(source_subject)s", source_name=opportunity_su.name, source_subject=message_su.subject)
                else:
                    subject = _("From %(source_name)s", source_name=opportunity_su.name)
                message_su.write({
                    'res_id': self.id,
                    'subject': subject,
                })

        opportunities.activity_ids.write({
            'res_id': self.id,
        })

        return True

    def _merge_dependences_attachments(self, opportunities):
        """ Move attachments of given opportunities to the current one `self`, and rename
            the attachments having same name than native ones.

        :param opportunities: see ``_merge_dependences``
        """
        self.ensure_one()

        all_attachments = self.env['ir.attachment'].search([
            ('res_model', '=', self._name),
            ('res_id', 'in', opportunities.ids)
        ])

        for opportunity in opportunities:
            attachments = all_attachments.filtered(lambda attach: attach.res_id == opportunity.id)
            for attachment in attachments:
                attachment.write({
                    'res_id': self.id,
                    'name': _("%(attach_name)s (from %(lead_name)s)",
                              attach_name=attachment.name,
                              lead_name=opportunity.name[:20]
                             )
                })
        return True

    def _merge_dependences_calendar_events(self, opportunities):
        """ Move calender.event from the given opportunities to the current one. `self` is the
            crm.lead record destination for event of `opportunities`.
        :param opportunities: see ``merge_dependences``
        """
        self.ensure_one()
        meetings = self.env['calendar.event'].search([('opportunity_id', 'in', opportunities.ids)])
        return meetings.write({
            'res_id': self.id,
            'opportunity_id': self.id,
        })

    def _merge_followers(self, opportunities):
        """Add the followers into the destination lead if they post a message in the last 30 days.

        :param opportunities : Record<crm.lead> of opportunities to transfer
        :return: {old_lead_id: Record<mail.followers>} Followers which have been added in
            the destination lead grouped by source lead ID.
        """
        self.ensure_one()

        self.env['mail.message'].flush_model()
        self.env['mail.followers'].flush_model()

        # Get the active followers (followers whose partner post a message on the
        # leads in the last 30 days) which should be moved on the destination lead
        self.env.cr.execute(
            '''
            SELECT MAX(mf.id) AS id
              FROM mail_followers AS mf
              JOIN mail_message AS mm
                ON mm.author_id = mf.partner_id
               AND mm.res_id = mf.res_id
               AND mm.model = 'crm.lead'
               AND mm.date > NOW() - INTERVAL '30 DAY'
                   /* Check if the partner is already
                      following the destination lead */
         LEFT JOIN mail_followers AS destf
                ON destf.res_model = 'crm.lead'
               AND destf.res_id = %(lead_id)s
               AND destf.partner_id = mf.partner_id
                   /* Select only once each partner
                      to not create duplicated followers */
             WHERE mf.res_model = 'crm.lead'
               AND mf.res_id IN %(lead_ids)s
               AND destf IS NULL
          GROUP BY mf.partner_id
            ''',
            {'lead_ids': tuple(opportunities.ids), 'lead_id': self.id},
        )
        followers_to_update = [r[0] for r in self.env.cr.fetchall()]
        followers_to_update = self.env['mail.followers'].browse(followers_to_update).sudo()
        followers_by_old_lead = dict(groupby(followers_to_update, lambda f: f.res_id))
        followers_to_update.write({'res_id': self.id})
        return followers_by_old_lead

    def _merge_log_summary(self, merged_followers, opportunities_tail):
        """Log the merge message on the lead."""
        self.ensure_one()
        self.message_post_with_source(
            "crm.crm_lead_merge_summary",
            render_values={
                "merged_followers": merged_followers,
                "opportunities": opportunities_tail,
                "is_html_empty": is_html_empty,
            },
            subtype_xmlid='mail.mt_note',
        )

    def _format_properties(self):
        """Format the properties to build the merge message.

        Return a list of dict containing the label, and a value key if there's only
        one value, or a "values" key if we have multiple values (e.g. many2many, tags).

        E.G.
            [{
                'label': 'My Partner',
                'value': 'Alice',
            }, {
                'label': 'My Partners',
                'values': [
                    {'name': 'Alice'},
                    {'name': 'Bob'},
                ],
            }, {
                'label': 'My Tags',
                'values': [
                    {'name': 'A', 'color': 1},
                    {'name': 'C', 'color': 3},
                ],
            }]
        """
        self.ensure_one()
        # read to have the display names already in the value
        properties = self.read(['lead_properties'])[0]['lead_properties']

        formatted = []
        for definition in properties:
            label = definition.get('string')
            value = definition.get('value')
            property_type = definition['type']
            if not value and property_type != 'boolean':
                continue

            property_dict = {'label': label}
            if property_type == 'boolean':
                property_dict['value'] = _('Yes') if value else _('No')
            elif value and property_type == 'many2one':
                property_dict['value'] = value[1]
            elif value and property_type == 'many2many':
                # show many2many in badge
                property_dict['values'] = [{'name': rec[1]} for rec in value]
            elif value and property_type in ['selection', 'tags']:
                # retrieve the option label from the value
                options = {
                    option[0]: option[1:]
                    for option in (definition.get(property_type) or [])
                }
                if property_type == 'selection':
                    value = options.get(value)
                    property_dict['value'] = value[0] if value else None
                else:
                    property_dict['values'] = [{
                        'name': options[tag][0],
                        'color': options[tag][1],
                        } for tag in value if tag in options
                    ]
            else:
                property_dict['value'] = value

            formatted.append(property_dict)

        return formatted

    # CONVERT
    # ----------------------------------------------------------------------

    def _convert_opportunity_data(self, customer, team_id=False):
        """ Extract the data from a lead to create the opportunity
            :param customer : res.partner record
            :param team_id : identifier of the Sales Team to determine the stage
        """
        new_team_id = team_id if team_id else self.team_id.id
        upd_values = {
            'type': 'opportunity',
            'date_conversion': self.env.cr.now(),
        }
        if customer != self.partner_id:
            upd_values['partner_id'] = customer.id if customer else False
        if not self.stage_id:
            stage = self._stage_find(team_id=new_team_id)
            upd_values['stage_id'] = stage.id
        return upd_values

    def convert_opportunity(self, partner, user_ids=False, team_id=False):
        customer = partner if partner else self.env['res.partner']
        for lead in self:
            if not lead.active or lead.probability == 100:
                continue
            vals = lead._convert_opportunity_data(customer, team_id)
            lead.write(vals)

        if user_ids or team_id:
            self._handle_salesmen_assignment(user_ids=user_ids, team_id=team_id)

        return True

    def _handle_partner_assignment(self, force_partner_id=False, create_missing=True):
        """ Update customer (partner_id) of leads. Purpose is to set the same
        partner on most leads; either through a newly created partner either
        through a given partner_id.

        :param int force_partner_id: if set, update all leads to that customer;
        :param create_missing: for leads without customer, create a new one
          based on lead information;
        """
        for lead in self:
            if force_partner_id:
                lead.partner_id = force_partner_id
            if not lead.partner_id and create_missing:
                partner = lead._create_customer()
                lead.partner_id = partner.id

    def _handle_salesmen_assignment(self, user_ids=False, team_id=False):
        """ Assign salesmen and salesteam to a batch of leads.  If there are more
        leads than salesmen, these salesmen will be assigned in round-robin. E.g.
        4 salesmen (S1, S2, S3, S4) for 6 leads (L1, L2, ... L6) will assigned as
        following: L1 - S1, L2 - S2, L3 - S3, L4 - S4, L5 - S1, L6 - S2.

        :param list user_ids: salesmen to assign
        :param int team_id: salesteam to assign
        """
        update_vals = {'team_id': team_id} if team_id else {}
        if not user_ids and team_id:
            self.write(update_vals)
        else:
            lead_ids = self.ids
            steps = len(user_ids)
            # pass 1 : lead_ids[0:6:3] = [L1,L4]
            # pass 2 : lead_ids[1:6:3] = [L2,L5]
            # pass 3 : lead_ids[2:6:3] = [L3,L6]
            # ...
            for idx in range(0, steps):
                subset_ids = lead_ids[idx:len(lead_ids):steps]
                update_vals['user_id'] = user_ids[idx]
                self.env['crm.lead'].browse(subset_ids).write(update_vals)

    # ------------------------------------------------------------
    # MERGE / CONVERT TOOLS
    # ---------------------------------------------------------

    # CLASSIFICATION TOOLS
    # --------------------------------------------------

    def _get_lead_duplicates(self, partner=None, email=None, include_lost=False):
        """ Search for leads that seem duplicated based on partner / email.

        :param partner : optional customer when searching duplicated
        :param email: email (possibly formatted) to search
        :param boolean include_lost: if True, search includes archived opportunities
          (still only active leads are considered). If False, search for active
          and not won leads and opportunities;
        """
        if not email and not partner:
            return self.env['crm.lead']

        domain = []
        for normalized_email in [tools.email_normalize(email) for email in tools.email_split(email)]:
            domain.append(('email_normalized', '=', normalized_email))
        if partner:
            domain.append(('partner_id', '=', partner.id))

        if not domain:
            return self.env['crm.lead']

        domain = ['|'] * (len(domain) - 1) + domain
        if include_lost:
            domain += ['|', ('type', '=', 'opportunity'), ('active', '=', True)]
        else:
            domain += ['&', ('active', '=', True), '|', ('stage_id', '=', False), ('stage_id.is_won', '=', False)]

        return self.with_context(active_test=False).search(domain)

    def _sort_by_confidence_level(self, reverse=False):
        """ Sorting the leads/opps according to the confidence level to it
        being won. It is sorted following this incremental heuristics :

          * "not lost" first (inactive leads are lost); normally all leads
            should be active but in case lost one, they are always last.
            Inactive opportunities are considered as valid;
          * opportunity is more reliable than a lead which is a pre-stage
            used mainly for first classification;
          * stage sequence: the higher the better as it indicates we are moving
            towards won stage;
          * probability: the higher the better as it is more likely to be won;
          * ID: the higher the better when all other parameters are equal. We
            consider newer leads to be more reliable;
        """
        def opps_key(opportunity):
            return opportunity.type == 'opportunity' or opportunity.active,  \
                opportunity.type == 'opportunity', \
                opportunity.stage_id.sequence, \
                opportunity.probability, \
                -opportunity._origin.id

        return self.sorted(key=opps_key, reverse=reverse)

    # CUSTOMER TOOLS
    # --------------------------------------------------

    def _find_matching_partner(self, email_only=False):
        """ Try to find a matching partner with available information on the
        lead, using notably customer's name, email, ...

        :param email_only: Only find a matching based on the email. To use
            for automatic process where ilike based on name can be too dangerous
        :return: partner browse record
        """
        self.ensure_one()
        partner = self.partner_id

        if not partner and self.email_from:
            partner = self.env['res.partner'].search([('email', '=', self.email_from)], limit=1)

        if not partner and not email_only:
            # search through the existing partners based on the lead's partner or contact name
            # to be aligned with _create_customer, search on lead's name as last possibility
            for customer_potential_name in [self[field_name] for field_name in ['partner_name', 'contact_name', 'name'] if self[field_name]]:
                partner = self.env['res.partner'].search([('name', 'ilike', customer_potential_name)], limit=1)
                if partner:
                    break

        return partner

    def _create_customer(self):
        """ Create a partner from lead data and link it to the lead.

        :return: newly-created partner browse record
        """
        Partner = self.env['res.partner']
        contact_name = self.contact_name
        if not contact_name:
            contact_name = parse_contact_from_email(self.email_from)[0] if self.email_from else False

        if self.partner_name:
            partner_company = Partner.create(self._prepare_customer_values(self.partner_name, is_company=True))
        elif self.partner_id:
            partner_company = self.partner_id
        else:
            partner_company = None

        if contact_name:
            return Partner.create(self._prepare_customer_values(contact_name, is_company=False, parent_id=partner_company.id if partner_company else False))

        if partner_company:
            return partner_company
        return Partner.create(self._prepare_customer_values(self.name, is_company=False))

    def _get_customer_information(self):
        email_normalized_to_values = super()._get_customer_information()
        Partner = self.env['res.partner']

        for record in self.filtered('email_normalized'):
            values = email_normalized_to_values.setdefault(record.email_normalized, {})
            contact_name = record.contact_name or record.partner_name or parse_contact_from_email(record.email_from)[0] or record.email_from
            # Note that we don't attempt to create the parent company even if partner name is set
            values.update(record._prepare_customer_values(contact_name, is_company=False))
            values['company_name'] = record.partner_name
            if contact_name == record.partner_name:
                values['company_type'] = 'company'
        return email_normalized_to_values

    def _prepare_customer_values(self, partner_name, is_company=False, parent_id=False):
        """ Extract data from lead to create a partner.

        :param name : furtur name of the partner
        :param is_company : True if the partner is a company
        :param parent_id : id of the parent partner (False if no parent)

        :return: dictionary of values to give at res_partner.create()
        """
        email_parts = tools.email_split(self.email_from)
        res = {
            'name': partner_name,
            'user_id': self.env.context.get('default_user_id') or self.user_id.id,
            'comment': self.description,
            'team_id': self.team_id.id,
            'parent_id': parent_id,
            'phone': self.phone,
            'mobile': self.mobile,
            'email': email_parts[0] if email_parts else False,
            'title': self.title.id,
            'function': self.function,
            'street': self.street,
            'street2': self.street2,
            'zip': self.zip,
            'city': self.city,
            'country_id': self.country_id.id,
            'state_id': self.state_id.id,
            'website': self.website,
            'is_company': is_company,
            'type': 'contact'
        }
        if self.lang_id.active:
            res['lang'] = self.lang_id.code
        return res

    # ------------------------------------------------------------
    # MAILING
    # ------------------------------------------------------------

    def _creation_subtype(self):
        return self.env.ref('crm.mt_lead_create')

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'stage_id' in init_values and self.probability == 100 and self.stage_id:
            return self.env.ref('crm.mt_lead_won')
        elif 'lost_reason_id' in init_values and self.lost_reason_id:
            return self.env.ref('crm.mt_lead_lost')
        elif 'stage_id' in init_values:
            return self.env.ref('crm.mt_lead_stage')
        elif 'active' in init_values and self.active:
            return self.env.ref('crm.mt_lead_restored')
        elif 'active' in init_values and not self.active:
            return self.env.ref('crm.mt_lead_lost')
        return super(Lead, self)._track_subtype(init_values)

    def _notify_by_email_prepare_rendering_context(self, message, msg_vals=False, model_description=False,
                                                   force_email_company=False, force_email_lang=False):
        render_context = super()._notify_by_email_prepare_rendering_context(
            message, msg_vals, model_description=model_description,
            force_email_company=force_email_company, force_email_lang=force_email_lang
        )
        if self.date_deadline:
            render_context['subtitles'].append(
                _('Deadline: %s', self.date_deadline.strftime(get_lang(self.env).date_format)))
        return render_context

    def _notify_get_recipients_groups(self, message, model_description, msg_vals=None):
        """ Handle salesman recipients that can convert leads into opportunities
        and set opportunities as won / lost. """
        groups = super()._notify_get_recipients_groups(
            message, model_description, msg_vals=msg_vals
        )
        if not self:
            return groups

        local_msg_vals = dict(msg_vals or {})

        self.ensure_one()
        if self.type == 'lead':
            convert_action = self._notify_get_action_link('controller', controller='/lead/convert', **local_msg_vals)
            salesman_actions = [{'url': convert_action, 'title': _('Convert to opportunity')}]
        else:
            won_action = self._notify_get_action_link('controller', controller='/lead/case_mark_won', **local_msg_vals)
            lost_action = self._notify_get_action_link('controller', controller='/lead/case_mark_lost', **local_msg_vals)
            salesman_actions = [
                {'url': won_action, 'title': _('Mark Won')},
                {'url': lost_action, 'title': _('Mark Lost')}]

        salesman_group_id = self.env.ref('sales_team.group_sale_salesman').id
        new_group = (
            'group_sale_salesman',
            lambda pdata: pdata['type'] == 'user' and salesman_group_id in pdata['groups'],
            {
                'actions': salesman_actions,
                'active': True,
                'has_button_access': True,
            }
        )

        return [new_group] + groups

    def _notify_get_reply_to(self, default=None):
        """ Override to set alias of lead and opportunities to their sales team if any. """
        aliases = self.mapped('team_id').sudo()._notify_get_reply_to(default=default)
        res = {lead.id: aliases.get(lead.team_id.id) for lead in self}
        leftover = self.filtered(lambda rec: not rec.team_id)
        if leftover:
            res.update(super(Lead, leftover)._notify_get_reply_to(default=default))
        return res

    def _message_get_default_recipients(self):
        return {
            r.id: {
                'partner_ids': [],
                'email_to': ','.join(tools.email_normalize_all(r.email_from)) or r.email_from,
                'email_cc': False,
            } for r in self
        }

    def _message_get_suggested_recipients(self):
        recipients = super(Lead, self)._message_get_suggested_recipients()
        try:
            for lead in self:
                # check if that language is correctly installed (and active) before using it
                lang_code = lead.lang_code if lead.lang_code and self.env['res.lang']._lang_get(lead.lang_code) else None
                if lead.partner_id:
                    lead._message_add_suggested_recipient(
                        recipients, partner=lead.partner_id, lang=lang_code, reason=_('Customer'))
                elif lead.email_from:
                    lead._message_add_suggested_recipient(
                        recipients, email=lead.email_from, lang=lang_code, reason=_('Customer Email'))
        except AccessError:  # no read access rights -> just ignore suggested recipients because this imply modifying followers
            pass
        return recipients

    @api.model
    def message_new(self, msg_dict, custom_values=None):
        """ Overrides mail_thread message_new that is called by the mailgateway
            through message_process.
            This override updates the document according to the email.
        """
        # remove default author when going through the mail gateway. Indeed we
        # do not want to explicitly set an user as responsible. We prefer that
        # assignment is done automatically (scoring) or manually. Otherwise it
        # would always be root (gateway user). It also allows to exclude portal
        # and public users.
        self = self.with_context(default_user_id=False)

        if custom_values is None:
            custom_values = {}
        defaults = {
            'name':  msg_dict.get('subject') or _("No Subject"),
            'email_from': msg_dict.get('from'),
            'partner_id': msg_dict.get('author_id', False),
        }
        if msg_dict.get('priority') in dict(crm_stage.AVAILABLE_PRIORITIES):
            defaults['priority'] = msg_dict.get('priority')
        defaults.update(custom_values)

        return super(Lead, self).message_new(msg_dict, custom_values=defaults)

    def _message_post_after_hook(self, message, msg_vals):
        if self.email_from and not self.partner_id:
            # we consider that posting a message with a specified recipient (not a follower, a specific one)
            # on a document without customer means that it was created through the chatter using
            # suggested recipients. This heuristic allows to avoid ugly hacks in JS.
            new_partner = message.partner_ids.filtered(
                lambda partner: partner.email == self.email_from or (self.email_normalized and partner.email_normalized == self.email_normalized)
            )
            if new_partner:
                if new_partner[0].email_normalized:
                    email_domain = ('email_normalized', '=', new_partner[0].email_normalized)
                else:
                    email_domain = ('email_from', '=', new_partner[0].email)
                self.search([
                    ('partner_id', '=', False), email_domain, ('stage_id.fold', '=', False)
                ]).write({'partner_id': new_partner[0].id})
        return super(Lead, self)._message_post_after_hook(message, msg_vals)

    def _message_partner_info_from_emails(self, emails, link_mail=False):
        """ Try to propose a better recipient when having only an email by populating
        it with the partner_name / contact_name field of the lead e.g. if lead
        contact_name is "Raoul" and email is "raoul@raoul.fr", suggest
        "Raoul" <raoul@raoul.fr> as recipient. """
        result = super(Lead, self)._message_partner_info_from_emails(emails, link_mail=link_mail)
        if not (self.partner_name or self.contact_name) or not self.email_from:
            return result
        for email, partner_info in zip(emails, result):
            if partner_info.get('partner_id') or not email:
                continue
            # reformat email if no name information
            name_emails = tools.email_split_tuples(email)
            name_from_email = name_emails[0][0] if name_emails else False
            if name_from_email:
                continue  # already containing name + email
            name_from_email = self.partner_name or self.contact_name
            emails_normalized = tools.email_normalize_all(email)
            email_normalized = emails_normalized[0] if emails_normalized else False
            if email.lower() == self.email_from.lower() or (email_normalized and self.email_normalized == email_normalized):
                partner_info['full_name'] = tools.formataddr((
                    name_from_email,
                    ','.join(emails_normalized) if emails_normalized else email))
                break
        return result

    @api.model
    def get_import_templates(self):
        return [{
            'label': _('Import Template for Leads & Opportunities'),
            'template': '/crm/static/xls/crm_lead.xls'
        }]

    # ------------------------------------------------------------
    # PLS
    # ------------------------------------------------------------
    # Predictive lead scoring is computing the lead probability, based on won and lost leads from the past
    # Each won/lost lead increments a frequency table, where we store, for each field/value couple, the number of
    # won and lost leads.
    #   E.g. : A won lead from Belgium will increase the won count of the frequency country_id='Belgium' by 1.
    # The frequencies are split by team_id, so each team has its own frequencies environment. (Team A doesn't impact B)
    # There are two main ways to build the frequency table:
    #   - Live Increment: At each Won/lost, we increment directly the frequencies based on the lead values.
    #       Done right BEFORE writing the lead as won or lost.
    #       We consider a lead that will be marked as won or lost.
    #       Used each time a lead is won or lost, to ensure frequency table is always up to date
    #   - One shot Rebuild: empty the frequency table and rebuild it from scratch, based on every already won/lost leads
    #       Done during cron process.
    #       We consider all the leads that have been already won or lost.
    #       Used in one shot, when modifying the criteria to take into account (fields or reference date)

    # ---------------------------------
    # PLS: Probability Computation
    # ---------------------------------
    def _pls_get_naive_bayes_probabilities(self, batch_mode=False):
        """
        In machine learning, naive Bayes classifiers (NBC) are a family of simple "probabilistic classifiers" based on
        applying Bayes theorem with strong (naive) independence assumptions between the variables taken into account.
        E.g: will TDE eat m&m's depending on his sleep status, the amount of work he has and the fullness of his stomach?
        As we use experience to compute the statistics, every day, we will register the variables state + the result.
        As the days pass, we will be able to determine, with more and more precision, if TDE will eat m&m's
        for a specific combination :
            - did sleep very well, a lot of work and stomach full > Will never happen !
            - didn't sleep at all, no work at all and empty stomach > for sure !
        Following Bayes' Theorem: the probability that an event occurs (to win) under certain conditions is proportional
        to the probability to win under each condition separately and the probability to win. We compute a 'Win score'
        -> P(Won | A∩B) ∝ P(A∩B | Won)*P(Won) OR S(Won | A∩B) = P(A∩B | Won)*P(Won)
        To compute a percentage of probability to win, we also compute the 'Lost score' that is proportional to the
        probability to lose under each condition separately and the probability to lose.
        -> Probability =  S(Won | A∩B) / ( S(Won | A∩B) + S(Lost | A∩B) )
        See https://www.youtube.com/watch?v=CPqOCI0ahss can help to get a quick and simple example.
        One issue about NBC is when a event occurence is never observed.
        E.g: if when TDE has an empty stomach, he always eat m&m's, than the "not eating m&m's when empty stomach' event
        will never be observed.
        This is called 'zero frequency' and that leads to division (or at least multiplication) by zero.
        To avoid this, we add 0.1 in each frequency. With few data, the computation is than not really realistic.
        The more we have records to analyse, the more the estimation will be precise.
        :return: probability in percent (and integer rounded) that the lead will be won at the current stage.
        """
        lead_probabilities = {}
        if not self:
            return lead_probabilities

        # Get all leads values, no matter the team_id
        domain = []
        if batch_mode:
            domain = [
                '&',
                    ('active', '=', True), ('id', 'in', self.ids),
                    '|',
                        ('probability', '=', None),
                        '&',
                            ('probability', '<', 100), ('probability', '>', 0)
            ]
        leads_values_dict = self._pls_get_lead_pls_values(domain=domain)

        if not leads_values_dict:
            return lead_probabilities

        # Get unique couples to search in frequency table and won leads.
        leads_fields = set()  # keep unique fields, as a lead can have multiple tag_ids
        won_leads = set()
        won_stage_ids = self.env['crm.stage'].search([('is_won', '=', True)]).ids
        for lead_id, values in leads_values_dict.items():
            for field, value in values['values']:
                if field == 'stage_id' and value in won_stage_ids:
                    won_leads.add(lead_id)
                leads_fields.add(field)
        leads_fields = sorted(leads_fields)
        # get all variable related records from frequency table, no matter the team_id
        frequencies = self.env['crm.lead.scoring.frequency'].search([('variable', 'in', list(leads_fields))], order="team_id asc, id")

        # get all team_ids from frequencies
        frequency_teams = frequencies.mapped('team_id')
        frequency_team_ids = [team.id for team in frequency_teams]

        # 1. Compute each variable value count individually
        # regroup each variable to be able to compute their own probabilities
        # As all the variable does not enter into account (as we reject unset values in the process)
        # each value probability must be computed only with their own variable related total count
        # special case: for lead for which team_id is not in frequency table or lead with no team_id,
        # we consider all the records, independently from team_id (this is why we add a result[-1])
        result = dict((team_id, dict((field, dict(won_total=0, lost_total=0)) for field in leads_fields)) for team_id in frequency_team_ids)
        result[-1] = dict((field, dict(won_total=0, lost_total=0)) for field in leads_fields)
        for frequency in frequencies:
            field = frequency['variable']
            value = frequency['value']

            # To avoid that a tag take too much importance if its subset is too small,
            # we ignore the tag frequencies if we have less than 50 won or lost for this tag.
            if field == 'tag_id' and (frequency['won_count'] + frequency['lost_count']) < 50:
                continue

            if frequency.team_id:
                team_result = result[frequency.team_id.id]
                team_result[field][value] = {'won': frequency['won_count'], 'lost': frequency['lost_count']}
                team_result[field]['won_total'] += frequency['won_count']
                team_result[field]['lost_total'] += frequency['lost_count']

            if value not in result[-1][field]:
                result[-1][field][value] = {'won': 0, 'lost': 0}
            result[-1][field][value]['won'] += frequency['won_count']
            result[-1][field][value]['lost'] += frequency['lost_count']
            result[-1][field]['won_total'] += frequency['won_count']
            result[-1][field]['lost_total'] += frequency['lost_count']

        # Get all won, lost and total count for all records in frequencies per team_id
        for team_id in result:
            result[team_id]['team_won'], \
            result[team_id]['team_lost'], \
            result[team_id]['team_total'] = self._pls_get_won_lost_total_count(result[team_id])

        save_team_id = None
        p_won, p_lost = 1, 1
        for lead_id, lead_values in leads_values_dict.items():
            # if stage_id is null, return 0 and bypass computation
            lead_fields = [value[0] for value in lead_values.get('values', [])]
            if not 'stage_id' in lead_fields:
                lead_probabilities[lead_id] = 0
                continue
            # if lead stage is won, return 100
            elif lead_id in won_leads:
                lead_probabilities[lead_id] = 100
                continue

            # team_id not in frequency Table -> convert to -1
            lead_team_id = lead_values['team_id'] if lead_values['team_id'] in result else -1
            if lead_team_id != save_team_id:
                save_team_id = lead_team_id
                team_won = result[save_team_id]['team_won']
                team_lost = result[save_team_id]['team_lost']
                team_total = result[save_team_id]['team_total']
                # if one count = 0, we cannot compute lead probability
                if not team_won or not team_lost:
                    continue
                p_won = team_won / team_total
                p_lost = team_lost / team_total

            # 2. Compute won and lost score using each variable's individual probability
            s_lead_won, s_lead_lost = p_won, p_lost
            for field, value in lead_values['values']:
                field_result = result.get(save_team_id, {}).get(field)
                value = value.origin if hasattr(value, 'origin') else value
                value_result = field_result.get(str(value)) if field_result else False
                if value_result:
                    total_won = team_won if field == 'stage_id' else field_result['won_total']
                    total_lost = team_lost if field == 'stage_id' else field_result['lost_total']

                    # if one count = 0, we cannot compute lead probability
                    if not total_won or not total_lost:
                        continue
                    s_lead_won *= value_result['won'] / total_won
                    s_lead_lost *= value_result['lost'] / total_lost

            # 3. Compute Probability to win
            probability = s_lead_won / (s_lead_won + s_lead_lost)
            lead_probabilities[lead_id] = min(max(round(100 * probability, 2), 0.01), 99.99)
        return lead_probabilities

    # ---------------------------------
    # PLS: Live Increment
    # ---------------------------------
    def _pls_increment_frequencies(self, from_state=None, to_state=None):
        """
        When losing or winning a lead, this method is called to increment each PLS parameter related to the lead
        in won_count (if won) or in lost_count (if lost).

        This method is also used when reactivating a mistakenly lost lead (using the decrement argument).
        In this case, the lost count should be de-increment by 1 for each PLS parameter linked to the lead.

        Live increment must be done before writing the new values because we need to know the state change (from and to).
        This would not be an issue for the reach won or reach lost as we just need to increment the frequencies with the
        final state of the lead.
        This issue is when the lead leaves a closed state because once the new values have been writen, we do not know
        what was the previous state that we need to decrement.
        This is why 'is_won' and 'decrement' parameters are used to describe the from / to change of its state.
        """
        new_frequencies_by_team, existing_frequencies_by_team = self._pls_prepare_update_frequency_table(target_state=from_state or to_state)

        # update frequency table
        self._pls_update_frequency_table(new_frequencies_by_team, 1 if to_state else -1,
                                         existing_frequencies_by_team=existing_frequencies_by_team)

    # ---------------------------------
    # PLS: One shot rebuild
    # ---------------------------------
    def _cron_update_automated_probabilities(self):
        """ This cron will :
          - rebuild the lead scoring frequency table
          - recompute all the automated_probability and align probability if both were aligned
        """
        cron_start_date = datetime.now()
        self._rebuild_pls_frequency_table()
        self._update_automated_probabilities()
        _logger.info("Predictive Lead Scoring : Cron duration = %d seconds" % ((datetime.now() - cron_start_date).total_seconds()))

    def _rebuild_pls_frequency_table(self):
        # Clear the frequencies table (in sql to speed up the cron)
        try:
            self.check_access_rights('unlink')
        except AccessError:
            raise UserError(_("You don't have the access needed to run this cron."))
        else:
            self._cr.execute('TRUNCATE TABLE crm_lead_scoring_frequency')

        new_frequencies_by_team, unused = self._pls_prepare_update_frequency_table(rebuild=True)
        # update frequency table
        self._pls_update_frequency_table(new_frequencies_by_team, 1)

        _logger.info("Predictive Lead Scoring : crm.lead.scoring.frequency table rebuilt")

    def _update_automated_probabilities(self):
        """ Recompute all the automated_probability (and align probability if both were aligned) for all the leads
        that are active (not won, nor lost).

        For performance matter, as there can be a huge amount of leads to recompute, this cron proceed by batch.
        Each batch is performed into its own transaction, in order to minimise the lock time on the lead table
        (and to avoid complete lock if there was only 1 transaction that would last for too long -> several minutes).
        If a concurrent update occurs, it will simply be put in the queue to get the lock.
        """
        pls_start_date = self._pls_get_safe_start_date()
        if not pls_start_date:
            return

        # 1. Get all the leads to recompute created after pls_start_date that are nor won nor lost
        # (Won : probability = 100 | Lost : probability = 0 or inactive. Here, inactive won't be returned anyway)
        # Get also all the lead without probability --> These are the new leads. Activate auto probability on them.
        pending_lead_domain = [
            '&',
                '&',
                    ('stage_id', '!=', False), ('create_date', '>=', pls_start_date),
                '|',
                    ('probability', '=', False),
                    '&',
                        ('probability', '<', 100), ('probability', '>', 0)
        ]
        leads_to_update = self.env['crm.lead'].search(pending_lead_domain)
        leads_to_update_count = len(leads_to_update)

        # 2. Compute by batch to avoid memory error
        lead_probabilities = {}
        for i in range(0, leads_to_update_count, PLS_COMPUTE_BATCH_STEP):
            leads_to_update_part = leads_to_update[i:i + PLS_COMPUTE_BATCH_STEP]
            lead_probabilities.update(leads_to_update_part._pls_get_naive_bayes_probabilities(batch_mode=True))
        _logger.info("Predictive Lead Scoring : New automated probabilities computed")

        # 3. Group by new probability to reduce server roundtrips when executing the update
        probability_leads = defaultdict(list)
        for lead_id, probability in sorted(lead_probabilities.items()):
            probability_leads[probability].append(lead_id)

        # 4. Update automated_probability (+ probability if both were equal)
        update_sql = """UPDATE crm_lead
                        SET automated_probability = %s,
                            probability = CASE WHEN (probability = automated_probability OR probability is null)
                                               THEN (%s)
                                               ELSE (probability)
                                          END
                        WHERE id in %s"""

        # Update by a maximum number of leads at the same time, one batch by transaction :
        # - avoid memory errors
        # - avoid blocking the table for too long with a too big transaction
        transactions_count, transactions_failed_count = 0, 0
        cron_update_lead_start_date = datetime.now()
        auto_commit = not getattr(threading.current_thread(), 'testing', False)
        self.flush_model()
        for probability, probability_lead_ids in probability_leads.items():
            for lead_ids_current in tools.split_every(PLS_UPDATE_BATCH_STEP, probability_lead_ids):
                transactions_count += 1
                try:
                    self.env.cr.execute(update_sql, (probability, probability, tuple(lead_ids_current)))
                    # auto-commit except in testing mode
                    if auto_commit:
                        self.env.cr.commit()
                except Exception as e:
                    _logger.warning("Predictive Lead Scoring : update transaction failed. Error: %s" % e)
                    transactions_failed_count += 1
        self.invalidate_model()

        _logger.info(
            "Predictive Lead Scoring : All automated probabilities updated (%d leads / %d transactions (%d failed) / %d seconds)" % (
                leads_to_update_count,
                transactions_count,
                transactions_failed_count,
                (datetime.now() - cron_update_lead_start_date).total_seconds(),
            )
        )

    # ---------------------------------
    # PLS: Common parts for both mode
    # ---------------------------------
    def _pls_prepare_update_frequency_table(self, rebuild=False, target_state=False):
        """
        This method is common to Live Increment or Full Rebuild mode, as it shares the main steps.
        This method will prepare the frequency dict needed to update the frequency table:
            - New frequencies: frequencies that we need to add in the frequency table.
            - Existing frequencies: frequencies that are already in the frequency table.
        In rebuild mode, only the new frequencies are needed as existing frequencies are truncated.
        For each team, each dict contains the frequency in won and lost for each field/value couple
        of the target leads.
        Target leads are :
            - in Live increment mode : given ongoing leads (self)
            - in Full rebuild mode : all the closed (won and lost) leads in the DB.
        During the frequencies update, with both new and existing frequencies, we can split frequencies to update
        and frequencies to add. If a field/value couple already exists in the frequency table, we just update it.
        Otherwise, we need to insert a new one.
        """
        # Keep eligible leads
        pls_start_date = self._pls_get_safe_start_date()
        if not pls_start_date:
            return {}, {}

        if rebuild:  # rebuild will treat every closed lead in DB, increment will treat current ongoing leads
            pls_leads = self
        else:
            # Only treat leads created after the PLS start Date
            pls_leads = self.filtered(
                lambda lead: fields.Date.to_date(pls_start_date) <= fields.Date.to_date(lead.create_date))
            if not pls_leads:
                return {}, {}

        # Extract target leads values
        if rebuild:  # rebuild is ok
            domain = [
                '&',
                    ('create_date', '>=', pls_start_date),
                    '|',
                        ('probability', '=', 100),
                        '&',
                            ('probability', '=', 0), ('active', '=', False)
              ]
            team_ids = self.env['crm.team'].with_context(active_test=False).search([]).ids + [0]  # If team_id is unset, consider it as team 0
        else:  # increment
            domain = [('id', 'in', pls_leads.ids)]
            team_ids = pls_leads.mapped('team_id').ids + [0]

        leads_values_dict = pls_leads._pls_get_lead_pls_values(domain=domain)

        # split leads values by team_id
        # get current frequencies related to the target leads
        leads_frequency_values_by_team = dict((team_id, []) for team_id in team_ids)
        leads_pls_fields = set()  # ensure to keep each field unique (can have multiple tag_id leads_values_dict)
        for lead_id, values in leads_values_dict.items():
            team_id = values.get('team_id', 0)  # If team_id is unset, consider it as team 0
            lead_frequency_values = {'count': 1}
            for field, value in values['values']:
                if field != "probability":  # was added to lead values in batch mode to know won/lost state, but is not a pls fields.
                    leads_pls_fields.add(field)
                else:  # extract lead probability - needed to increment tag_id frequency. (proba always before tag_id)
                    lead_probability = value
                if field == 'tag_id':  # handle tag_id separatelly (as in One Shot rebuild mode)
                    leads_frequency_values_by_team[team_id].append({field: value, 'count': 1, 'probability': lead_probability})
                else:
                    lead_frequency_values[field] = value
            leads_frequency_values_by_team[team_id].append(lead_frequency_values)
        leads_pls_fields = sorted(leads_pls_fields)

        # get new frequencies
        new_frequencies_by_team = {}
        for team_id in team_ids:
            # prepare fields and tag values for leads by team
            new_frequencies_by_team[team_id] = self._pls_prepare_frequencies(
                leads_frequency_values_by_team[team_id], leads_pls_fields, target_state=target_state)

        # get existing frequencies
        existing_frequencies_by_team = {}
        if not rebuild:  # there is no existing frequency in rebuild mode as they were all deleted.
            # read all fields to get everything in memory in one query (instead of having query + prefetch)
            existing_frequencies = self.env['crm.lead.scoring.frequency'].search_read(
                ['&', ('variable', 'in', leads_pls_fields),
                      '|', ('team_id', 'in', pls_leads.mapped('team_id').ids), ('team_id', '=', False)])
            for frequency in existing_frequencies:
                team_id = frequency['team_id'][0] if frequency.get('team_id') else 0
                if team_id not in existing_frequencies_by_team:
                    existing_frequencies_by_team[team_id] = dict((field, {}) for field in leads_pls_fields)

                existing_frequencies_by_team[team_id][frequency['variable']][frequency['value']] = {
                    'frequency_id': frequency['id'],
                    'won': frequency['won_count'],
                    'lost': frequency['lost_count']
                }

        return new_frequencies_by_team, existing_frequencies_by_team

    def _pls_update_frequency_table(self, new_frequencies_by_team, step, existing_frequencies_by_team=None):
        """ Create / update the frequency table in a cross company way, per team_id"""
        values_to_update = {}
        values_to_create = []
        if not existing_frequencies_by_team:
            existing_frequencies_by_team = {}
        # build the create multi + frequencies to update
        for team_id, new_frequencies in new_frequencies_by_team.items():
            for field, value in new_frequencies.items():
                # frequency already present ?
                current_frequencies = existing_frequencies_by_team.get(team_id, {})
                for param, result in value.items():
                    current_frequency_for_couple = current_frequencies.get(field, {}).get(param, {})
                    # If frequency already present : UPDATE IT
                    if current_frequency_for_couple:
                        new_won = current_frequency_for_couple['won'] + (result['won'] * step)
                        new_lost = current_frequency_for_couple['lost'] + (result['lost'] * step)
                        # ensure to have always positive frequencies
                        values_to_update[current_frequency_for_couple['frequency_id']] = {
                            'won_count': new_won if new_won > 0 else 0.1,
                            'lost_count': new_lost if new_lost > 0 else 0.1
                        }
                        continue

                    # Else, CREATE a new frequency record.
                    # We add + 0.1 in won and lost counts to avoid zero frequency issues
                    # should be +1 but it weights too much on small recordset.
                    values_to_create.append({
                        'variable': field,
                        'value': param,
                        'won_count': result['won'] + 0.1,
                        'lost_count': result['lost'] + 0.1,
                        'team_id': team_id if team_id else None  # team_id = 0 means no team_id
                    })

        LeadScoringFrequency = self.env['crm.lead.scoring.frequency'].sudo()
        for frequency_id, values in values_to_update.items():
            LeadScoringFrequency.browse(frequency_id).write(values)

        if values_to_create:
            LeadScoringFrequency.create(values_to_create)

    # ---------------------------------
    # Utility Tools for PLS
    # ---------------------------------

    # PLS:  Config Parameters
    # ---------------------
    def _pls_get_safe_start_date(self):
        """ As config_parameters does not accept Date field,
            we get directly the date formated string stored into the Char config field,
            as we directly use this string in the sql queries.
            To avoid sql injections when using this config param,
            we ensure the date string can be effectively a date."""
        str_date = self.env['ir.config_parameter'].sudo().get_param('crm.pls_start_date')
        if not fields.Date.to_date(str_date):
            return False
        return str_date

    def _pls_get_safe_fields(self):
        """ As config_parameters does not accept M2M field,
            we the fields from the formated string stored into the Char config field.
            To avoid sql injections when using that list, we return only the fields
            that are defined on the model. """
        pls_fields_config = self.env['ir.config_parameter'].sudo().get_param('crm.pls_fields')
        pls_fields = pls_fields_config.split(',') if pls_fields_config else []
        pls_safe_fields = [field for field in pls_fields if field in self._fields.keys()]
        return pls_safe_fields

    # Compute Automated Probability Tools
    # -----------------------------------
    def _pls_get_won_lost_total_count(self, team_results):
        """ Get all won and all lost + total :
               first stage can be used to know how many lost and won there is
               as won count are equals for all stage
               and first stage is always incremented in lost_count
        :param frequencies: lead_scoring_frequencies
        :return: won count, lost count and total count for all records in frequencies
        """
        # TODO : check if we need to handle specific team_id stages [for lost count] (if first stage in sequence is team_specific)
        first_stage_id = self.env['crm.stage'].search([('team_id', '=', False)], order='sequence, id', limit=1)
        if str(first_stage_id.id) not in team_results.get('stage_id', []):
            return 0, 0, 0
        stage_result = team_results['stage_id'][str(first_stage_id.id)]
        return stage_result['won'], stage_result['lost'], stage_result['won'] + stage_result['lost']

    # PLS: Rebuild Frequency Table Tools
    # ----------------------------------
    def _pls_prepare_frequencies(self, lead_values, leads_pls_fields, target_state=None):
        """new state is used when getting frequencies for leads that are changing to lost or won.
        Stays none if we are checking frequencies for leads already won or lost."""
        pls_fields = leads_pls_fields.copy()
        frequencies = dict((field, {}) for field in pls_fields)

        stage_ids = self.env['crm.stage'].search_read([], ['sequence', 'name', 'id'], order='sequence, id')
        stage_sequences = {stage['id']: stage['sequence'] for stage in stage_ids}

        # Increment won / lost frequencies by criteria (field / value couple)
        for values in lead_values:
            if target_state:  # ignore probability values if target state (as probability is the old value)
                won_count = values['count'] if target_state == 'won' else 0
                lost_count = values['count'] if target_state == 'lost' else 0
            else:
                won_count = values['count'] if values.get('probability', 0) == 100 else 0
                lost_count = values['count'] if values.get('probability', 1) == 0  else 0

            if 'tag_id' in values:
                frequencies = self._pls_increment_frequency_dict(frequencies, 'tag_id', values['tag_id'], won_count, lost_count)
                continue

            # Else, treat other fields
            if 'tag_id' in pls_fields:  # tag_id already treated here above.
                pls_fields.remove('tag_id')
            for field in pls_fields:
                if field not in values:
                    continue
                value = values[field]
                if value or field in ('email_state', 'phone_state'):
                    if field == 'stage_id':
                        if won_count:  # increment all stages if won
                            stages_to_increment = [stage['id'] for stage in stage_ids]
                        else:  # increment only current + previous stages if lost
                            current_stage_sequence = stage_sequences[value]
                            stages_to_increment = [stage['id'] for stage in stage_ids if stage['sequence'] <= current_stage_sequence]
                        for stage_id in stages_to_increment:
                            frequencies = self._pls_increment_frequency_dict(frequencies, field, stage_id, won_count, lost_count)
                    else:
                        frequencies = self._pls_increment_frequency_dict(frequencies, field, value, won_count, lost_count)

        return frequencies

    def _pls_increment_frequency_dict(self, frequencies, field, value, won, lost):
        value = str(value)  # Ensure we will always compare strings.
        if value not in frequencies[field]:
            frequencies[field][value] = {'won': won, 'lost': lost}
        else:
            frequencies[field][value]['won'] += won
            frequencies[field][value]['lost'] += lost
        return frequencies

    # Common PLS Tools
    # ----------------
    def _pls_get_lead_pls_values(self, domain=[]):
        """
        This methods builds a dict where, for each lead in self or matching the given domain,
        we will get a list of field/value couple.
        Due to onchange and create, we don't always have the id of the lead to recompute.
        When we update few records (one, typically) with onchanges, we build the lead_values (= couple field/value)
        using the ORM.
        To speed up the computation and avoid making too much DB read inside loops,
        we can give a domain to make sql queries to bypass the ORM.
        This domain will be used in sql queries to get the values for every lead matching the domain.
        :param domain: If set, we get all the leads values via unique sql queries (one for tags, one for other fields),
                            using the given domain on leads.
                       If not set, get lead values lead by lead using the ORM.
        :return: {lead_id: [(field1: value1), (field2: value2), ...], ...}
        """
        leads_values_dict = OrderedDict()
        pls_fields = ["stage_id", "team_id"] + self._pls_get_safe_fields()

        # Check if tag_ids is in the pls_fields and removed it from the list. The tags will be managed separately.
        use_tags = 'tag_ids' in pls_fields
        if use_tags:
            pls_fields.remove('tag_ids')

        if domain:
            # active_test = False as domain should take active into 'active' field it self
            from_clause, where_clause, where_params = self.env['crm.lead'].with_context(active_test=False)._where_calc(domain).get_sql()
            str_fields = ", ".join(["{}"] * len(pls_fields))
            args = [sql.Identifier(field) for field in pls_fields]

            # Get leads values
            self.flush_model()
            query = """SELECT id, probability, %s
                        FROM %s
                        WHERE %s order by team_id asc, id desc"""
            query = sql.SQL(query % (str_fields, from_clause, where_clause)).format(*args)
            self._cr.execute(query, where_params)
            lead_results = self._cr.dictfetchall()

            if use_tags:
                # Get tags values
                query = """SELECT crm_lead.id as lead_id, t.id as tag_id
                            FROM %s
                            LEFT JOIN crm_tag_rel rel ON crm_lead.id = rel.lead_id
                            LEFT JOIN crm_tag t ON rel.tag_id = t.id
                            WHERE %s order by crm_lead.team_id asc, crm_lead.id"""
                args.append(sql.Identifier('tag_id'))
                query = sql.SQL(query % (from_clause, where_clause)).format(*args)
                self._cr.execute(query, where_params)
                tag_results = self._cr.dictfetchall()
            else:
                tag_results = []

            # get all (variable, value) couple for all in self
            for lead in lead_results:
                lead_values = []
                for field in pls_fields + ['probability']:  # add probability as used in _pls_prepare_frequencies (needed in rebuild mode)
                    value = lead[field]
                    if field == 'team_id':  # ignore team_id as stored separately in leads_values_dict[lead_id][team_id]
                        continue
                    if value or field == 'probability':  # 0 is a correct value for probability
                        lead_values.append((field, value))
                    elif field in ('email_state', 'phone_state'):  # As ORM reads 'None' as 'False', do the same here
                        lead_values.append((field, False))
                    leads_values_dict[lead['id']] = {'values': lead_values, 'team_id': lead['team_id'] or 0}

            for tag in tag_results:
                if tag['tag_id']:
                    leads_values_dict[tag['lead_id']]['values'].append(('tag_id', tag['tag_id']))
            return leads_values_dict
        else:
            for lead in self:
                lead_values = []
                for field in pls_fields:
                    if field == 'team_id':  # ignore team_id as stored separately in leads_values_dict[lead_id][team_id]
                        continue
                    value = lead[field].id if isinstance(lead[field], models.BaseModel) else lead[field]
                    if value or field in ('email_state', 'phone_state'):
                        lead_values.append((field, value))
                if use_tags:
                    for tag in lead.tag_ids:
                        lead_values.append(('tag_id', tag.id))
                leads_values_dict[lead.id] = {'values': lead_values, 'team_id': lead['team_id'].id}
            return leads_values_dict

```

## File: models\crm_lead_scoring_frequency.py

```python
# -*- coding: utf-8 -*-
from odoo import fields, models


class LeadScoringFrequency(models.Model):
    _name = 'crm.lead.scoring.frequency'
    _description = 'Lead Scoring Frequency'

    variable = fields.Char('Variable', index=True)
    value = fields.Char('Value')
    won_count = fields.Float('Won Count', digits=(16, 1))  # Float because we add 0.1 to avoid zero Frequency issue
    lost_count = fields.Float('Lost Count', digits=(16, 1))  # Float because we add 0.1 to avoid zero Frequency issue
    team_id = fields.Many2one('crm.team', 'Sales Team', ondelete="cascade")

class FrequencyField(models.Model):
    _name = 'crm.lead.scoring.frequency.field'
    _description = 'Fields that can be used for predictive lead scoring computation'

    name = fields.Char(related="field_id.field_description")
    field_id = fields.Many2one(
        'ir.model.fields', domain=[('model_id.model', '=', 'crm.lead')], required=True,
        ondelete='cascade',
    )

```

## File: models\crm_lost_reason.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class LostReason(models.Model):
    _name = "crm.lost.reason"
    _description = 'Opp. Lost Reason'

    name = fields.Char('Description', required=True, translate=True)
    active = fields.Boolean('Active', default=True)
    leads_count = fields.Integer('Leads Count', compute='_compute_leads_count')

    def _compute_leads_count(self):
        lead_data = self.env['crm.lead'].with_context(active_test=False)._read_group(
            [('lost_reason_id', 'in', self.ids)],
            ['lost_reason_id'],
            ['__count'],
        )
        mapped_data = {lost_reason.id: count for lost_reason, count in lead_data}
        for reason in self:
            reason.leads_count = mapped_data.get(reason.id, 0)

    def action_lost_leads(self):
        return {
            'name': _('Leads'),
            'view_mode': 'tree,form',
            'domain': [('lost_reason_id', 'in', self.ids)],
            'res_model': 'crm.lead',
            'type': 'ir.actions.act_window',
            'context': {'create': False, 'active_test': False},
        }

```

## File: models\crm_recurring_plan.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class RecurringPlan(models.Model):
    _name = "crm.recurring.plan"
    _description = "CRM Recurring revenue plans"
    _order = "sequence"

    name = fields.Char('Plan Name', required=True, translate=True)
    number_of_months = fields.Integer('# Months', required=True)
    active = fields.Boolean('Active', default=True)
    sequence = fields.Integer('Sequence', default=10)

    _sql_constraints = [
        ('check_number_of_months', 'CHECK(number_of_months >= 0)', 'The number of month can\'t be negative.'),
    ]

```

## File: models\crm_stage.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models

AVAILABLE_PRIORITIES = [
    ('0', 'Low'),
    ('1', 'Medium'),
    ('2', 'High'),
    ('3', 'Very High'),
]


class Stage(models.Model):
    """ Model for case stages. This models the main stages of a document
        management flow. Main CRM objects (leads, opportunities, project
        issues, ...) will now use only stages, instead of state and stages.
        Stages are for example used to display the kanban view of records.
    """
    _name = "crm.stage"
    _description = "CRM Stages"
    _rec_name = 'name'
    _order = "sequence, name, id"

    @api.model
    def default_get(self, fields):
        """ As we have lots of default_team_id in context used to filter out
        leads and opportunities, we pop this key from default of stage creation.
        Otherwise stage will be created for a given team only which is not the
        standard behavior of stages. """
        if 'default_team_id' in self.env.context:
            ctx = dict(self.env.context)
            ctx.pop('default_team_id')
            self = self.with_context(ctx)
        return super(Stage, self).default_get(fields)

    name = fields.Char('Stage Name', required=True, translate=True)
    sequence = fields.Integer('Sequence', default=1, help="Used to order stages. Lower is better.")
    is_won = fields.Boolean('Is Won Stage?')
    requirements = fields.Text('Requirements', help="Enter here the internal requirements for this stage (ex: Offer sent to customer). It will appear as a tooltip over the stage's name.")
    team_id = fields.Many2one('crm.team', string='Sales Team', ondelete="set null",
        help='Specific team that uses this stage. Other teams will not be able to see or use this stage.')
    fold = fields.Boolean('Folded in Pipeline',
        help='This stage is folded in the kanban view when there are no records in that stage to display.')
    # This field for interface only
    team_count = fields.Integer('team_count', compute='_compute_team_count')

    @api.depends('team_id')
    def _compute_team_count(self):
        self.team_count = self.env['crm.team'].search_count([])

```

## File: models\crm_team.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
import logging
import random
import threading

from ast import literal_eval
from markupsafe import Markup

from odoo import api, exceptions, fields, models, _
from odoo.osv import expression
from odoo.tools import float_compare, float_round
from odoo.tools.safe_eval import safe_eval

_logger = logging.getLogger(__name__)


class Team(models.Model):
    _name = 'crm.team'
    _inherit = ['mail.alias.mixin', 'crm.team']
    _description = 'Sales Team'

    use_leads = fields.Boolean('Leads', help="Check this box to filter and qualify incoming requests as leads before converting them into opportunities and assigning them to a salesperson.")
    use_opportunities = fields.Boolean('Pipeline', default=True, help="Check this box to manage a presales process with opportunities.")
    alias_id = fields.Many2one(help="The email address associated with this channel. New emails received will automatically create new leads assigned to the channel.")
    # assignment
    assignment_enabled = fields.Boolean('Lead Assign', compute='_compute_assignment_enabled')
    assignment_auto_enabled = fields.Boolean('Auto Assignment', compute='_compute_assignment_enabled')
    assignment_optout = fields.Boolean('Skip auto assignment')
    assignment_max = fields.Integer(
        'Lead Average Capacity', compute='_compute_assignment_max',
        help='Monthly average leads capacity for all salesmen belonging to the team')
    assignment_domain = fields.Char(
        'Assignment Domain', tracking=True,
        help='Additional filter domain when fetching unassigned leads to allocate to the team.')
    # statistics about leads / opportunities / both
    lead_unassigned_count = fields.Integer(
        string='# Unassigned Leads', compute='_compute_lead_unassigned_count')
    lead_all_assigned_month_count = fields.Integer(
        string='# Leads/Opps assigned this month', compute='_compute_lead_all_assigned_month_count',
        help="Number of leads and opportunities assigned this last month.")
    lead_all_assigned_month_exceeded = fields.Boolean('Exceed monthly lead assignement', compute="_compute_lead_all_assigned_month_count",
        help="True if the monthly lead assignment count is greater than the maximum assignment limit, false otherwise."
    )
    opportunities_count = fields.Integer(
        string='# Opportunities', compute='_compute_opportunities_data')
    opportunities_amount = fields.Monetary(
        string='Opportunities Revenues', compute='_compute_opportunities_data')
    opportunities_overdue_count = fields.Integer(
        string='# Overdue Opportunities', compute='_compute_opportunities_overdue_data')
    opportunities_overdue_amount = fields.Monetary(
        string='Overdue Opportunities Revenues', compute='_compute_opportunities_overdue_data',)
    # properties
    lead_properties_definition = fields.PropertiesDefinition('Lead Properties')

    @api.depends('crm_team_member_ids.assignment_max')
    def _compute_assignment_max(self):
        for team in self:
            team.assignment_max = sum(member.assignment_max for member in team.crm_team_member_ids)

    def _compute_assignment_enabled(self):
        assign_enabled = self.env['ir.config_parameter'].sudo().get_param('crm.lead.auto.assignment', False)
        auto_assign_enabled = False
        if assign_enabled:
            assign_cron = self.sudo().env.ref('crm.ir_cron_crm_lead_assign', raise_if_not_found=False)
            auto_assign_enabled = assign_cron.active if assign_cron else False
        self.assignment_enabled = assign_enabled
        self.assignment_auto_enabled = auto_assign_enabled

    def _compute_lead_unassigned_count(self):
        leads_data = self.env['crm.lead']._read_group([
            ('team_id', 'in', self.ids),
            ('type', '=', 'lead'),
            ('user_id', '=', False),
        ], ['team_id'], ['__count'])
        counts = {team.id: count for team, count in leads_data}
        for team in self:
            team.lead_unassigned_count = counts.get(team.id, 0)

    @api.depends('crm_team_member_ids.lead_month_count', 'assignment_max')
    def _compute_lead_all_assigned_month_count(self):
        for team in self:
            team.lead_all_assigned_month_count = sum(member.lead_month_count for member in team.crm_team_member_ids)
            team.lead_all_assigned_month_exceeded = team.lead_all_assigned_month_count > team.assignment_max

    def _compute_opportunities_data(self):
        opportunity_data = self.env['crm.lead']._read_group([
            ('team_id', 'in', self.ids),
            ('probability', '<', 100),
            ('type', '=', 'opportunity'),
        ], ['team_id'], ['__count', 'expected_revenue:sum'])
        counts_amounts = {team.id: (count, expected_revenue_sum) for team, count, expected_revenue_sum in opportunity_data}
        for team in self:
            team.opportunities_count, team.opportunities_amount = counts_amounts.get(team.id, (0, 0))

    def _compute_opportunities_overdue_data(self):
        opportunity_data = self.env['crm.lead']._read_group([
            ('team_id', 'in', self.ids),
            ('probability', '<', 100),
            ('type', '=', 'opportunity'),
            ('date_deadline', '<', fields.Date.to_string(fields.Datetime.now()))
        ], ['team_id'], ['__count', 'expected_revenue:sum'])
        counts_amounts = {team.id: (count, expected_revenue_sum) for team, count, expected_revenue_sum in opportunity_data}
        for team in self:
            team.opportunities_overdue_count, team.opportunities_overdue_amount = counts_amounts.get(team.id, (0, 0))

    @api.onchange('use_leads', 'use_opportunities')
    def _onchange_use_leads_opportunities(self):
        if not self.use_leads and not self.use_opportunities:
            self.alias_name = False

    @api.constrains('assignment_domain')
    def _constrains_assignment_domain(self):
        for team in self:
            try:
                domain = literal_eval(team.assignment_domain or '[]')
                if domain:
                    self.env['crm.lead'].search(domain, limit=1)
            except Exception:
                raise exceptions.ValidationError(_('Assignment domain for team %(team)s is incorrectly formatted', team=team.name))

    # ------------------------------------------------------------
    # ORM
    # ------------------------------------------------------------

    def write(self, vals):
        result = super(Team, self).write(vals)
        if 'use_leads' in vals or 'use_opportunities' in vals:
            for team in self:
                alias_vals = team._alias_get_creation_values()
                team.write({
                    'alias_name': alias_vals.get('alias_name', team.alias_name),
                    'alias_defaults': alias_vals.get('alias_defaults'),
                })
        return result

    def unlink(self):
        """ When unlinking, concatenate ``crm.lead.scoring.frequency`` linked to
        the team into "no team" statistics. """
        frequencies = self.env['crm.lead.scoring.frequency'].search([('team_id', 'in', self.ids)])
        if frequencies:
            existing_noteam = self.env['crm.lead.scoring.frequency'].sudo().search([
                ('team_id', '=', False),
                ('variable', 'in', frequencies.mapped('variable'))
            ])
            for frequency in frequencies:
                # skip void-like values
                if float_compare(frequency.won_count, 0.1, 2) != 1 and float_compare(frequency.lost_count, 0.1, 2) != 1:
                    continue

                match = existing_noteam.filtered(lambda frequ_nt: frequ_nt.variable == frequency.variable and frequ_nt.value == frequency.value)
                if match:
                    # remove extra .1 that may exist in db as those are artifacts of initializing
                    # frequency table. Final value of 0 will be set to 0.1.
                    exist_won_count = float_round(match.won_count, precision_digits=0, rounding_method='HALF-UP')
                    exist_lost_count = float_round(match.lost_count, precision_digits=0, rounding_method='HALF-UP')
                    add_won_count = float_round(frequency.won_count, precision_digits=0, rounding_method='HALF-UP')
                    add_lost_count = float_round(frequency.lost_count, precision_digits=0, rounding_method='HALF-UP')
                    new_won_count = exist_won_count + add_won_count
                    new_lost_count = exist_lost_count + add_lost_count
                    match.won_count = new_won_count if float_compare(new_won_count, 0.1, 2) == 1 else 0.1
                    match.lost_count = new_lost_count if float_compare(new_lost_count, 0.1, 2) == 1 else 0.1
                else:
                    existing_noteam += self.env['crm.lead.scoring.frequency'].sudo().create({
                        'lost_count': frequency.lost_count if float_compare(frequency.lost_count, 0.1, 2) == 1 else 0.1,
                        'team_id': False,
                        'value': frequency.value,
                        'variable': frequency.variable,
                        'won_count': frequency.won_count if float_compare(frequency.won_count, 0.1, 2) == 1 else 0.1,
                    })
        return super(Team, self).unlink()

    # ------------------------------------------------------------
    # MESSAGING
    # ------------------------------------------------------------

    def _alias_get_creation_values(self):
        values = super(Team, self)._alias_get_creation_values()
        values['alias_model_id'] = self.env['ir.model']._get('crm.lead').id
        if self.id:
            if not self.use_leads and not self.use_opportunities:
                values['alias_name'] = False
            values['alias_defaults'] = defaults = literal_eval(self.alias_defaults or "{}")
            has_group_use_lead = self.env.user.has_group('crm.group_use_lead')
            defaults['type'] = 'lead' if has_group_use_lead and self.use_leads else 'opportunity'
            defaults['team_id'] = self.id
        return values

    # ------------------------------------------------------------
    # LEAD ASSIGNMENT
    # ------------------------------------------------------------

    @api.model
    def _cron_assign_leads(self, work_days=None):
        """ Cron method assigning leads. Leads are allocated to all teams and
        assigned to their members. It is based on either cron configuration
        either forced through ``work_days`` parameter.

        When based on cron configuration purpose of cron is to assign leads to
        sales persons. Assigned workload is set to the workload those sales
        people should perform between two cron iterations. If their maximum
        capacity is reached assign process will not assign them any more lead.

        e.g. cron is active with interval_number 3, interval_type days. This
        means cron runs every 3 days. Cron will assign leads for 3 work days
        to salespersons each 3 days unless their maximum capacity is reached.

        If cron runs on an hour- or minute-based schedule minimum assignment
        performed is equivalent to 0.2 workdays to avoid rounding issues.
        Max assignment performed is for 30 days as it is better to run more
        often than planning for more than one month. Assign process is best
        designed to run every few hours (~4 times / day) or each few days.

        See ``CrmTeam.action_assign_leads()`` and its sub methods for more
        details about assign process.

        :param float work_days: see ``CrmTeam.action_assign_leads()``;
        """
        assign_cron = self.sudo().env.ref('crm.ir_cron_crm_lead_assign', raise_if_not_found=False)
        if not work_days and assign_cron and assign_cron.active:
            if assign_cron.interval_type == 'months':
                work_days = 30  # maximum one month of work
            elif assign_cron.interval_type == 'weeks':
                work_days = min(30, assign_cron.interval_number * 7)  # max at 30 (better lead repartition)
            elif assign_cron.interval_type == 'days':
                work_days = min(30, assign_cron.interval_number * 1)  # max at 30 (better lead repartition)
            elif assign_cron.interval_type == 'hours':
                work_days = max(0.2, assign_cron.interval_number / 24)    # min at 0.2 to avoid small numbers issues
            elif assign_cron.interval_type == 'minutes':
                work_days = max(0.2, assign_cron.interval_number / 1440)    # min at 0.2 to avoid small numbers issues
        work_days = work_days if work_days else 1  # avoid void values
        self.env['crm.team'].search([
            '&', '|', ('use_leads', '=', True), ('use_opportunities', '=', True),
            ('assignment_optout', '=', False)
        ])._action_assign_leads(work_days=work_days)
        return True

    def action_assign_leads(self, work_days=1, log=True):
        """ Manual (direct) leads assignment. This method both

          * assigns leads to teams given by self;
          * assigns leads to salespersons belonging to self;

        See sub methods for more details about assign process.

        :param float work_days: number of work days to consider when assigning leads
          to teams or salespersons. We consider that Member.assignment_max (or
          its equivalent on team model) targets 30 work days. We make a ratio
          between expected number of work days and maximum assignment for those
          30 days to know lead count to assign.

        :return action: a client notification giving some insights on assign
          process;
        """
        teams_data, members_data = self._action_assign_leads(work_days=work_days)

        # format result messages
        logs = self._action_assign_leads_logs(teams_data, members_data)
        html_message = Markup('<br />').join(logs)
        notif_message = ' '.join(logs)

        # log a note in case of manual assign (as this method will mainly be called
        # on singleton record set, do not bother doing a specific message per team)
        log_action = _("Lead Assignment requested by %(user_name)s", user_name=self.env.user.name)
        log_message = Markup("<p>%s<br /><br />%s</p>") % (log_action, html_message)
        self._message_log_batch(bodies=dict((team.id, log_message) for team in self))

        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'success',
                'title': _("Leads Assigned"),
                'message': notif_message,
                'next': {
                    'type': 'ir.actions.act_window_close'
                },
            }
        }

    def _action_assign_leads(self, work_days=1):
        """ Private method for lead assignment. This method both

          * assigns leads to teams given by self;
          * assigns leads to salespersons belonging to self;

        See sub methods for more details about assign process.

        :param float work_days: see ``CrmTeam.action_assign_leads()``;

        :return teams_data, members_data: structure-based result of assignment
          process. For more details about data see ``CrmTeam._allocate_leads()``
          and ``CrmTeamMember._assign_and_convert_leads``;
        """
        if not self.env.user.has_group('sales_team.group_sale_manager') and not self.env.user.has_group('base.group_system'):
            raise exceptions.UserError(_('Lead/Opportunities automatic assignment is limited to managers or administrators'))

        _logger.info('### START Lead Assignment (%d teams, %d sales persons, %.2f work_days)', len(self), len(self.crm_team_member_ids), work_days)
        teams_data = self._allocate_leads(work_days=work_days)
        _logger.info('### Team repartition done. Starting salesmen assignment.')
        members_data = self.crm_team_member_ids._assign_and_convert_leads(work_days=work_days)
        _logger.info('### END Lead Assignment')
        return teams_data, members_data

    def _action_assign_leads_logs(self, teams_data, members_data):
        """ Tool method to prepare notification about assignment process result.

        :param teams_data: see ``CrmTeam._allocate_leads()``;
        :param members_data: see ``CrmTeamMember._assign_and_convert_leads()``;

        :return list: list of formatted logs, ready to be formatted into a nice
        plaintext or html message at caller's will
        """
        # extract some statistics
        assigned = sum(len(teams_data[team]['assigned']) + len(teams_data[team]['merged']) for team in teams_data)
        duplicates = sum(len(teams_data[team]['duplicates']) for team in teams_data)
        members = len(members_data)
        members_assigned = sum(len(member_data['assigned']) for member_data in members_data.values())

        # format user notification
        message_parts = []
        # 1- duplicates removal
        if duplicates:
            message_parts.append(_("%(duplicates)s duplicates leads have been merged.",
                                   duplicates=duplicates))

        # 2- nothing assigned at all
        if not assigned and not members_assigned:
            if len(self) == 1:
                if not self.assignment_max:
                    message_parts.append(
                        _("No allocated leads to %(team_name)s team because it has no capacity. Add capacity to its salespersons.",
                          team_name=self.name))
                else:
                    message_parts.append(
                        _("No allocated leads to %(team_name)s team and its salespersons because no unassigned lead matches its domain.",
                          team_name=self.name))
            else:
                message_parts.append(
                    _("No allocated leads to any team or salesperson. Check your Sales Teams and Salespersons configuration as well as unassigned leads."))

        # 3- team allocation
        if not assigned and members_assigned:
            if len(self) == 1:
                message_parts.append(
                    _("No new lead allocated to %(team_name)s team because no unassigned lead matches its domain.",
                      team_name=self.name))
            else:
                message_parts.append(_("No new lead allocated to the teams because no lead match their domains."))
        elif assigned:
            if len(self) == 1:
                message_parts.append(
                    _("%(assigned)s leads allocated to %(team_name)s team.",
                      assigned=assigned, team_name=self.name))
            else:
                message_parts.append(
                    _("%(assigned)s leads allocated among %(team_count)s teams.",
                      assigned=assigned, team_count=len(self)))

        # 4- salespersons assignment
        if not members_assigned and assigned:
            message_parts.append(
                _("No lead assigned to salespersons because no unassigned lead matches their domains."))
        elif members_assigned:
            message_parts.append(
                _("%(members_assigned)s leads assigned among %(member_count)s salespersons.",
                  members_assigned=members_assigned, member_count=members))

        return message_parts

    def _allocate_leads(self, work_days=1):
        """ Allocate leads to teams given by self. This method sets ``team_id``
        field on lead records that are unassigned (no team and no responsible).
        No salesperson is assigned in this process. Its purpose is simply to
        allocate leads within teams.

        This process allocates all available leads on teams weighted by their
        maximum assignment by month that indicates their relative workload.

        Heuristic of this method is the following:
          * find unassigned leads for each team, aka leads being
            * without team, without user -> not assigned;
            * not in a won stage, and not having False/0 (lost) or 100 (won)
              probability) -> live leads;
            * if set, a delay after creation can be applied (see BUNDLE_HOURS_DELAY)
              parameter explanations here below;
            * matching the team's assignment domain (empty means
              everything);

          * assign a weight to each team based on their assignment_max that
            indicates their relative workload;

          * pick a random team using a weighted random choice and find a lead
            to assign:

            * remove already assigned leads from the available leads. If there
              is not any lead spare to assign, remove team from active teams;
            * pick the first lead and set the current team;
            * when setting a team on leads, leads are also merged with their
              duplicates. Purpose is to clean database and avoid assigning
              duplicates to same or different teams;
            * add lead and its duplicates to already assigned leads;

          * pick another random team until their is no more leads to assign
            to any team;

        This process ensure that teams having overlapping domains will all
        receive leads as lead allocation is done one lead at a time. This
        allocation will be proportional to their size (assignment of their
        members).

        :config int crm.assignment.bundle: deprecated
        :config int crm.assignment.commit.bundle: optional config parameter allowing
          to set size of lead batch to be committed together. By default 100
          which is a good trade-off between transaction time and speed
        :config int crm.assignment.delay: optional config parameter giving a
          delay before taking a lead into assignment process (BUNDLE_HOURS_DELAY)
          given in hours. Purpose if to allow other crons or automation rules
          to make their job. This option is mainly historic as its purpose was
          to let automation rules prepare leads and score before PLS was added
          into CRM. This is now not required anymore but still supported;

        :param float work_days: see ``CrmTeam.action_assign_leads()``;

        :return teams_data: dict() with each team assignment result:
          team: {
            'assigned': set of lead IDs directly assigned to the team (no
              duplicate or merged found);
            'merged': set of lead IDs merged and assigned to the team (main
              leads being results of merge process);
            'duplicates': set of lead IDs found as duplicates and merged into
              other leads. Those leads are unlinked during assign process and
              are already removed at return of this method;
          }, ...
        """
        if work_days < 0.2 or work_days > 30:
            raise ValueError(
                _('Leads team allocation should be done for at least 0.2 or maximum 30 work days, not %.2f.', work_days)
            )

        BUNDLE_HOURS_DELAY = int(self.env['ir.config_parameter'].sudo().get_param('crm.assignment.delay', default=0))
        BUNDLE_COMMIT_SIZE = int(self.env['ir.config_parameter'].sudo().get_param('crm.assignment.commit.bundle', 100))
        auto_commit = not getattr(threading.current_thread(), 'testing', False)

        # leads
        max_create_dt = self.env.cr.now() - datetime.timedelta(hours=BUNDLE_HOURS_DELAY)
        duplicates_lead_cache = dict()

        # teams data
        teams_data, population, weights = dict(), list(), list()
        for team in self:
            if not team.assignment_max:
                continue

            lead_domain = expression.AND([
                literal_eval(team.assignment_domain or '[]'),
                [('create_date', '<=', max_create_dt)],
                ['&', ('team_id', '=', False), ('user_id', '=', False)],
                ['|', ('stage_id', '=', False), ('stage_id.is_won', '=', False)]
            ])

            leads = self.env["crm.lead"].search(lead_domain)
            # Fill duplicate cache: search for duplicate lead before the assignation
            # avoid to flush during the search at every assignation
            for lead in leads:
                if lead not in duplicates_lead_cache:
                    duplicates_lead_cache[lead] = lead._get_lead_duplicates(email=lead.email_from)

            teams_data[team] = {
                "team": team,
                "leads": leads,
                "assigned": set(),
                "merged": set(),
                "duplicates": set(),
            }
            population.append(team)
            weights.append(team.assignment_max)

        # Start a new transaction, since data fetching take times
        # and the first commit occur at the end of the bundle,
        # the first transaction can be long which we want to avoid
        if auto_commit:
            self._cr.commit()

        # assignment process data
        global_data = dict(assigned=set(), merged=set(), duplicates=set())
        leads_done_ids, lead_unlink_ids, counter = set(), set(), 0
        while population:
            counter += 1
            team = random.choices(population, weights=weights, k=1)[0]

            # filter remaining leads, remove team if no more leads for it
            teams_data[team]["leads"] = teams_data[team]["leads"].filtered(lambda l: l.id not in leads_done_ids).exists()
            if not teams_data[team]["leads"]:
                population_index = population.index(team)
                population.pop(population_index)
                weights.pop(population_index)
                continue

            # assign + deduplicate and concatenate results in teams_data to keep some history
            candidate_lead = teams_data[team]["leads"][0]
            assign_res = team._allocate_leads_deduplicate(candidate_lead, duplicates_cache=duplicates_lead_cache)
            for key in ('assigned', 'merged', 'duplicates'):
                teams_data[team][key].update(assign_res[key])
                leads_done_ids.update(assign_res[key])
                global_data[key].update(assign_res[key])
            lead_unlink_ids.update(assign_res['duplicates'])

            # auto-commit except in testing mode. As this process may be time consuming or we
            # may encounter errors, already commit what is allocated to avoid endless cron loops.
            if auto_commit and counter % BUNDLE_COMMIT_SIZE == 0:
                # unlink duplicates once
                self.env['crm.lead'].browse(lead_unlink_ids).unlink()
                lead_unlink_ids = set()
                self._cr.commit()

        # unlink duplicates once
        self.env['crm.lead'].browse(lead_unlink_ids).unlink()

        if auto_commit:
            self._cr.commit()

        # some final log
        _logger.info('## Assigned %s leads', (len(global_data['assigned']) + len(global_data['merged'])))
        for team, team_data in teams_data.items():
            _logger.info(
                '## Assigned %s leads to team %s',
                len(team_data['assigned']) + len(team_data['merged']), team.id)
            _logger.info(
                '\tLeads: direct assign %s / merge result %s / duplicates merged: %s',
                team_data['assigned'], team_data['merged'], team_data['duplicates'])
        return teams_data

    def _allocate_leads_deduplicate(self, leads, duplicates_cache=None):
        """ Assign leads to sales team given by self by calling lead tool
        method _handle_salesmen_assignment. In this method we deduplicate leads
        allowing to reduce number of resulting leads before assigning them
        to salesmen.

        :param leads: recordset of leads to assign to current team;
        :param duplicates_cache: if given, avoid to perform a duplicate search
          and fetch information in it instead;
        """
        self.ensure_one()
        duplicates_cache = duplicates_cache if duplicates_cache is not None else dict()

        # classify leads
        leads_assigned = self.env['crm.lead']  # direct team assign
        leads_done_ids, leads_merged_ids, leads_dup_ids = set(), set(), set()  # classification
        leads_dups_dict = dict()  # lead -> its duplicate
        for lead in leads:
            if lead.id not in leads_done_ids:

                # fill cache if not already done
                if lead not in duplicates_cache:
                    duplicates_cache[lead] = lead._get_lead_duplicates(email=lead.email_from)
                lead_duplicates = duplicates_cache[lead].exists()

                if len(lead_duplicates) > 1:
                    leads_dups_dict[lead] = lead_duplicates
                    leads_done_ids.update((lead + lead_duplicates).ids)
                else:
                    leads_assigned += lead
                    leads_done_ids.add(lead.id)

        # assign team to direct assign (leads_assigned) + dups keys (to ensure their team
        # if they are elected master of merge process)
        dups_to_assign = [lead for lead in leads_dups_dict]
        leads_assigned.union(*dups_to_assign)._handle_salesmen_assignment(user_ids=None, team_id=self.id)

        for lead in leads.filtered(lambda lead: lead in leads_dups_dict):
            lead_duplicates = leads_dups_dict[lead]
            merged = lead_duplicates._merge_opportunity(user_id=False, team_id=False, auto_unlink=False, max_length=0)
            leads_dup_ids.update((lead_duplicates - merged).ids)
            leads_merged_ids.add(merged.id)

        return {
            'assigned': set(leads_assigned.ids),
            'merged': leads_merged_ids,
            'duplicates': leads_dup_ids,
        }

    # ------------------------------------------------------------
    # ACTIONS
    # ------------------------------------------------------------

    #TODO JEM : refactor this stuff with xml action, proper customization,
    @api.model
    def action_your_pipeline(self):
        action = self.env["ir.actions.actions"]._for_xml_id("crm.crm_lead_action_pipeline")
        return self._action_update_to_pipeline(action)

    @api.model
    def action_opportunity_forecast(self):
        action = self.env['ir.actions.actions']._for_xml_id('crm.crm_lead_action_forecast')
        return self._action_update_to_pipeline(action)

    @api.model
    def _action_update_to_pipeline(self, action):
        user_team_id = self.env.user.sale_team_id.id
        if not user_team_id:
            user_team_id = self.search([], limit=1).id
            action['help'] = "<p class='o_view_nocontent_smiling_face'>%s</p><p>" % _("Create an Opportunity")
            if user_team_id:
                if self.user_has_groups('sales_team.group_sale_manager'):
                    action['help'] += "<p>%s</p>" % _("""As you are a member of no Sales Team, you are showed the Pipeline of the <b>first team by default.</b>
                                        To work with the CRM, you should <a name="%d" type="action" tabindex="-1">join a team.</a>""",
                                        self.env.ref('sales_team.crm_team_action_config').id)
                else:
                    action['help'] += "<p>%s</p>" % _("""As you are a member of no Sales Team, you are showed the Pipeline of the <b>first team by default.</b>
                                        To work with the CRM, you should join a team.""")
        action_context = safe_eval(action['context'], {'uid': self.env.uid})
        action['context'] = action_context
        return action

    def _compute_dashboard_button_name(self):
        super(Team, self)._compute_dashboard_button_name()
        team_with_pipelines = self.filtered(lambda el: el.use_opportunities)
        team_with_pipelines.update({'dashboard_button_name': _("Pipeline")})

    def action_primary_channel_button(self):
        self.ensure_one()
        if self.use_opportunities:
            action = self.env['ir.actions.actions']._for_xml_id('crm.crm_case_form_view_salesteams_opportunity')
            rcontext = {
                'team': self,
            }
            action['help'] = self.env['ir.ui.view']._render_template('crm.crm_action_helper', values=rcontext)
            return action
        return super(Team,self).action_primary_channel_button()

    def _graph_get_model(self):
        if self.use_opportunities:
            return 'crm.lead'
        return super(Team,self)._graph_get_model()

    def _graph_date_column(self):
        if self.use_opportunities:
            return 'create_date'
        return super(Team,self)._graph_date_column()

    def _graph_y_query(self):
        if self.use_opportunities:
            return 'count(*)'
        return super(Team,self)._graph_y_query()

    def _extra_sql_conditions(self):
        if self.use_opportunities:
            return "AND type LIKE 'opportunity'"
        return super(Team,self)._extra_sql_conditions()

    def _graph_title_and_key(self):
        if self.use_opportunities:
            return ['', _('New Opportunities')] # no more title
        return super(Team, self)._graph_title_and_key()

```

## File: models\crm_team_member.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
import logging
import math
import threading
import random

from ast import literal_eval

from odoo import api, exceptions, fields, models, _
from odoo.osv import expression

_logger = logging.getLogger(__name__)


class TeamMember(models.Model):
    _inherit = 'crm.team.member'

    # assignment
    assignment_enabled = fields.Boolean(related="crm_team_id.assignment_enabled")
    assignment_domain = fields.Char('Assignment Domain', tracking=True)
    assignment_optout = fields.Boolean('Skip auto assignment')
    assignment_max = fields.Integer('Average Leads Capacity (on 30 days)', default=30)
    lead_month_count = fields.Integer(
        'Leads (30 days)', compute='_compute_lead_month_count',
        help='Lead assigned to this member those last 30 days')

    @api.depends('user_id', 'crm_team_id')
    def _compute_lead_month_count(self):
        for member in self:
            if member.user_id.id and member.crm_team_id.id:
                member.lead_month_count = self.env['crm.lead'].with_context(active_test=False).search_count(
                    member._get_lead_month_domain()
                )
            else:
                member.lead_month_count = 0

    @api.constrains('assignment_domain')
    def _constrains_assignment_domain(self):
        for member in self:
            try:
                domain = literal_eval(member.assignment_domain or '[]')
                if domain:
                    self.env['crm.lead'].search(domain, limit=1)
            except Exception:
                raise exceptions.ValidationError(_(
                    'Member assignment domain for user %(user)s and team %(team)s is incorrectly formatted',
                    user=member.user_id.name, team=member.crm_team_id.name
                ))

    def _get_lead_month_domain(self):
        limit_date = fields.Datetime.now() - datetime.timedelta(days=30)
        return [
            ('user_id', '=', self.user_id.id),
            ('team_id', '=', self.crm_team_id.id),
            ('date_open', '>=', limit_date),
        ]

    # ------------------------------------------------------------
    # LEAD ASSIGNMENT
    # ------------------------------------------------------------

    def _assign_and_convert_leads(self, work_days=1):
        """ Main processing method to assign leads to sales team members. It also
        converts them into opportunities. This method should be called after
        ``_allocate_leads`` as this method assigns leads already allocated to
        the member's team. Its main purpose is therefore to distribute team
        workload on its members based on their capacity.

        Preparation

          * prepare lead domain for each member. It is done using a logical
            AND with team's domain and member's domain. Member domains further
            restricts team domain;
          * prepare a set of available leads for each member by searching for
            leads matching domain with a sufficient limit to ensure all members
            will receive leads;
          * prepare a weighted population sample. Population are members that
            should received leads. Initial weight is the number of leads to
            assign to that specific member. This is minimum value between
            * remaining this month: assignment_max - number of lead already
              assigned this month;
            * days-based assignment: assignment_max with a ratio based on
              ``work_days`` parameter (see ``CrmTeam.action_assign_leads()``)
            * e.g. Michel Poilvache (max: 30 - currently assigned: 15) limit
              for 2 work days: min(30-15, 30/15) -> 2 leads assigned
            * e.g. Michel Tartopoil (max: 30 - currently assigned: 26) limit
              for 10 work days: min(30-26, 30/3) -> 4 leads assigned

        This method then follows the following heuristic

          * take a weighted random choice in population;
          * find first available (not yet assigned) lead in its lead set;
          * if found:
            * convert it into an opportunity and assign member as salesperson;
            * lessen member's weight so that other members have an higher
              probability of being picked up next;
          * if not found: consider this member is out of assignment process,
            remove it from population so that it is not picked up anymore;

        Assignment is performed one lead at a time for fairness purpose. Indeed
        members may have overlapping domains within a given team. To ensure
        some fairness in process once a member receives a lead, a new choice is
        performed with updated weights. This is not optimal from performance
        point of view but increases probability leads are correctly distributed
        within the team.

        :param float work_days: see ``CrmTeam.action_assign_leads()``;

        :return members_data: dict() with each member assignment result:
          membership: {
            'assigned': set of lead IDs directly assigned to the member;
          }, ...

        """
        if work_days < 0.2 or work_days > 30:
            raise ValueError(
                _('Leads team allocation should be done for at least 0.2 or maximum 30 work days, not %.2f.', work_days)
            )

        members_data, population, weights = dict(), list(), list()
        members = self.filtered(lambda member: not member.assignment_optout and member.assignment_max > 0)
        if not members:
            return members_data

        # prepare a global lead count based on total leads to assign to salespersons
        lead_limit = sum(
            member._get_assignment_quota(work_days=work_days)
            for member in members
        )

        # could probably be optimized
        for member in members:
            lead_domain = expression.AND([
                literal_eval(member.assignment_domain or '[]'),
                ['&', '&', ('user_id', '=', False), ('date_open', '=', False), ('team_id', '=', member.crm_team_id.id)]
            ])

            leads = self.env["crm.lead"].search(lead_domain, order='probability DESC, id', limit=lead_limit)

            to_assign = member._get_assignment_quota(work_days=work_days)
            members_data[member.id] = {
                "team_member": member,
                "max": member.assignment_max,
                "to_assign": to_assign,
                "leads": leads,
                "assigned": self.env["crm.lead"],
            }
            population.append(member.id)
            weights.append(to_assign)

        leads_done_ids = set()
        counter = 0
        # auto-commit except in testing mode
        auto_commit = not getattr(threading.current_thread(), 'testing', False)
        commit_bundle_size = int(self.env['ir.config_parameter'].sudo().get_param('crm.assignment.commit.bundle', 100))
        while population and any(weights):
            counter += 1
            member_id = random.choices(population, weights=weights, k=1)[0]
            member_index = population.index(member_id)
            member_data = members_data[member_id]

            lead = next((lead for lead in member_data['leads'] if lead.id not in leads_done_ids), False)
            if lead:
                leads_done_ids.add(lead.id)
                members_data[member_id]["assigned"] += lead
                weights[member_index] = weights[member_index] - 1

                lead.with_context(mail_auto_subscribe_no_notify=True).convert_opportunity(
                    lead.partner_id,
                    user_ids=member_data['team_member'].user_id.ids
                )

                if auto_commit and counter % commit_bundle_size == 0:
                    self._cr.commit()
            else:
                weights[member_index] = 0

            if weights[member_index] <= 0:
                population.pop(member_index)
                weights.pop(member_index)

            # failsafe
            if counter > 100000:
                population = list()

        if auto_commit:
            self._cr.commit()
        # log results and return
        result_data = dict(
            (member_info["team_member"], {"assigned": member_info["assigned"]})
            for member_id, member_info in members_data.items()
        )
        _logger.info('Assigned %s leads to %s salesmen', len(leads_done_ids), len(members))
        for member, member_info in result_data.items():
            _logger.info('-> member %s: assigned %d leads (%s)', member.id, len(member_info["assigned"]), member_info["assigned"])
        return result_data

    def _get_assignment_quota(self, work_days=1):
        """ Compute assignment quota based on work_days. This quota includes
        a compensation to speedup getting to the lead average (``assignment_max``).
        As this field is a counter for "30 days" -> divide by requested work
        days in order to have base assign number then add compensation.

        :param float work_days: see ``CrmTeam.action_assign_leads()``;
        """
        assign_ratio = work_days / 30.0
        to_assign = self.assignment_max * assign_ratio
        compensation = max(0, self.assignment_max - (self.lead_month_count + to_assign)) * 0.2
        return round(to_assign + compensation)

```

## File: models\digest.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import AccessError


class Digest(models.Model):
    _inherit = 'digest.digest'

    kpi_crm_lead_created = fields.Boolean('New Leads')
    kpi_crm_lead_created_value = fields.Integer(compute='_compute_kpi_crm_lead_created_value')
    kpi_crm_opportunities_won = fields.Boolean('Opportunities Won')
    kpi_crm_opportunities_won_value = fields.Integer(compute='_compute_kpi_crm_opportunities_won_value')

    def _compute_kpi_crm_lead_created_value(self):
        if not self.env.user.has_group('sales_team.group_sale_salesman'):
            raise AccessError(_("Do not have access, skip this data for user's digest email"))

        self._calculate_company_based_kpi('crm.lead', 'kpi_crm_lead_created_value')

    def _compute_kpi_crm_opportunities_won_value(self):
        if not self.env.user.has_group('sales_team.group_sale_salesman'):
            raise AccessError(_("Do not have access, skip this data for user's digest email"))

        self._calculate_company_based_kpi(
            'crm.lead',
            'kpi_crm_opportunities_won_value',
            date_field='date_closed',
            additional_domain=[('type', '=', 'opportunity'), ('probability', '=', '100')],
        )

    def _compute_kpis_actions(self, company, user):
        res = super(Digest, self)._compute_kpis_actions(company, user)
        res['kpi_crm_lead_created'] = 'crm.crm_lead_action_pipeline&menu_id=%s' % self.env.ref('crm.crm_menu_root').id
        res['kpi_crm_opportunities_won'] = 'crm.crm_lead_action_pipeline&menu_id=%s' % self.env.ref('crm.crm_menu_root').id
        if user.has_group('crm.group_use_lead'):
            res['kpi_crm_lead_created'] = 'crm.crm_lead_all_leads&menu_id=%s' % self.env.ref('crm.crm_menu_root').id
        return res

```

## File: models\ir_config_parameter.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.addons.base.models.ir_model import MODULE_UNINSTALL_FLAG


class IrConfigParameter(models.Model):
    _inherit = 'ir.config_parameter'

    def write(self, vals):
        result = super(IrConfigParameter, self).write(vals)
        if any(record.key == "crm.pls_fields" for record in self):
            self.env.flush_all()
            self.env.registry.setup_models(self.env.cr)
        return result

    @api.model_create_multi
    def create(self, vals_list):
        records = super(IrConfigParameter, self).create(vals_list)
        if any(record.key == "crm.pls_fields" for record in records):
            self.env.flush_all()
            self.env.registry.setup_models(self.env.cr)
        return records

    def unlink(self):
        pls_emptied = any(record.key == "crm.pls_fields" for record in self)
        result = super(IrConfigParameter, self).unlink()
        if pls_emptied and not self._context.get(MODULE_UNINSTALL_FLAG):
            self.env.flush_all()
            self.env.registry.setup_models(self.env.cr)
        return result

```

## File: models\mail_activity.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MailActivity(models.Model):
    _inherit = "mail.activity"

    def action_create_calendar_event(self):
        """ Small override of the action that creates a calendar.

        If the activity is linked to a crm.lead through the "opportunity_id" field, we include in
        the action context the default values used when scheduling a meeting from the crm.lead form
        view.
        e.g: It will set the partner_id of the crm.lead as default attendee of the meeting. """

        action = super(MailActivity, self).action_create_calendar_event()
        opportunity = self.calendar_event_id.opportunity_id
        if opportunity:
            opportunity_action_context = opportunity.action_schedule_meeting(smart_calendar=False).get('context', {})
            opportunity_action_context['initial_date'] = self.calendar_event_id.start

            action['context'].update(opportunity_action_context)

        return action

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta
from dateutil.relativedelta import relativedelta

from odoo import api, exceptions, fields, models, _


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    group_use_lead = fields.Boolean(string="Leads", implied_group='crm.group_use_lead')
    group_use_recurring_revenues = fields.Boolean(string="Recurring Revenues", implied_group='crm.group_use_recurring_revenues')
    # Membership
    is_membership_multi = fields.Boolean(string='Multi Teams', config_parameter='sales_team.membership_multi')
    # Lead assignment
    crm_use_auto_assignment = fields.Boolean(
        string='Rule-Based Assignment', config_parameter='crm.lead.auto.assignment')
    crm_auto_assignment_action = fields.Selection([
        ('manual', 'Manually'), ('auto', 'Repeatedly')],
        string='Auto Assignment Action', compute='_compute_crm_auto_assignment_data',
        readonly=False, store=True,
        help='Manual assign allow to trigger assignment from team form view using an action button. Automatic configures a cron running repeatedly assignment in all teams.')
    crm_auto_assignment_interval_type = fields.Selection([
        ('minutes', 'Minutes'), ('hours', 'Hours'),
        ('days', 'Days'), ('weeks', 'Weeks')],
        string='Auto Assignment Interval Unit', compute='_compute_crm_auto_assignment_data',
        readonly=False, store=True,
        help='Interval type between each cron run (e.g. each 2 days or each 2 hours)')
    crm_auto_assignment_interval_number = fields.Integer(
        string="Repeat every", compute='_compute_crm_auto_assignment_data',
        readonly=False, store=True,
        help='Number of interval type between each cron run (e.g. each 2 days or each 4 days)')
    crm_auto_assignment_run_datetime = fields.Datetime(
        string="Auto Assignment Next Execution Date", compute='_compute_crm_auto_assignment_data',
        readonly=False, store=True)
    # IAP
    module_crm_iap_mine = fields.Boolean("Generate new leads based on their country, industries, size, etc.")
    module_crm_iap_enrich = fields.Boolean("Enrich your leads automatically with company data based on their email address.")
    module_website_crm_iap_reveal = fields.Boolean("Create Leads/Opportunities from your website's traffic")
    lead_enrich_auto = fields.Selection([
        ('manual', 'Enrich leads on demand only'),
        ('auto', 'Enrich all leads automatically'),
    ], string='Enrich lead automatically', default='auto', config_parameter='crm.iap.lead.enrich.setting')
    lead_mining_in_pipeline = fields.Boolean("Create a lead mining request directly from the opportunity pipeline.", config_parameter='crm.lead_mining_in_pipeline')
    predictive_lead_scoring_start_date = fields.Date(string='Lead Scoring Starting Date', compute="_compute_pls_start_date", inverse="_inverse_pls_start_date_str")
    predictive_lead_scoring_start_date_str = fields.Char(string='Lead Scoring Starting Date in String', config_parameter='crm.pls_start_date')
    predictive_lead_scoring_fields = fields.Many2many('crm.lead.scoring.frequency.field', string='Lead Scoring Frequency Fields', compute="_compute_pls_fields", inverse="_inverse_pls_fields_str")
    predictive_lead_scoring_fields_str = fields.Char(string='Lead Scoring Frequency Fields in String', config_parameter='crm.pls_fields')
    predictive_lead_scoring_field_labels = fields.Char(compute='_compute_predictive_lead_scoring_field_labels')

    @api.depends('crm_use_auto_assignment')
    def _compute_crm_auto_assignment_data(self):
        assign_cron = self.sudo().env.ref('crm.ir_cron_crm_lead_assign', raise_if_not_found=False)
        for setting in self:
            if setting.crm_use_auto_assignment and assign_cron:
                setting.crm_auto_assignment_action = 'auto' if assign_cron.active else 'manual'
                setting.crm_auto_assignment_interval_type = assign_cron.interval_type or 'days'
                setting.crm_auto_assignment_interval_number = assign_cron.interval_number or 1
                setting.crm_auto_assignment_run_datetime = assign_cron.nextcall
            else:
                setting.crm_auto_assignment_action = 'manual'
                setting.crm_auto_assignment_interval_type = setting.crm_auto_assignment_run_datetime = False
                setting.crm_auto_assignment_interval_number = 1

    @api.onchange('crm_auto_assignment_interval_type', 'crm_auto_assignment_interval_number')
    def _onchange_crm_auto_assignment_run_datetime(self):
        if self.crm_auto_assignment_interval_number <= 0:
            raise exceptions.UserError(_('Repeat frequency should be positive.'))
        elif self.crm_auto_assignment_interval_number >= 100:
            raise exceptions.UserError(_('Invalid repeat frequency. Consider changing frequency type instead of using large numbers.'))
        self.crm_auto_assignment_run_datetime = self._get_crm_auto_assignmment_run_datetime(
            self.crm_auto_assignment_run_datetime,
            self.crm_auto_assignment_interval_type,
            self.crm_auto_assignment_interval_number
        )

    @api.depends('predictive_lead_scoring_fields_str')
    def _compute_pls_fields(self):
        """ As config_parameters does not accept m2m field,
            we get the fields back from the Char config field, to ease the configuration in config panel """
        for setting in self:
            if setting.predictive_lead_scoring_fields_str:
                names = setting.predictive_lead_scoring_fields_str.split(',')
                fields = self.env['ir.model.fields'].search([('name', 'in', names), ('model', '=', 'crm.lead')])
                setting.predictive_lead_scoring_fields = self.env['crm.lead.scoring.frequency.field'].search([('field_id', 'in', fields.ids)])
            else:
                setting.predictive_lead_scoring_fields = None

    def _inverse_pls_fields_str(self):
        """ As config_parameters does not accept m2m field,
            we store the fields with a comma separated string into a Char config field """
        for setting in self:
            if setting.predictive_lead_scoring_fields:
                setting.predictive_lead_scoring_fields_str = ','.join(setting.predictive_lead_scoring_fields.mapped('field_id.name'))
            else:
                setting.predictive_lead_scoring_fields_str = ''

    @api.depends('predictive_lead_scoring_start_date_str')
    def _compute_pls_start_date(self):
        """ As config_parameters does not accept Date field,
            we get the date back from the Char config field, to ease the configuration in config panel """
        for setting in self:
            lead_scoring_start_date = setting.predictive_lead_scoring_start_date_str
            # if config param is deleted / empty, set the date 8 days prior to current date
            if not lead_scoring_start_date:
                setting.predictive_lead_scoring_start_date = fields.Date.to_date(fields.Date.today() - timedelta(days=8))
            else:
                try:
                    setting.predictive_lead_scoring_start_date = fields.Date.to_date(lead_scoring_start_date)
                except ValueError:
                    # the config parameter is malformed, so set the date 8 days prior to current date
                    setting.predictive_lead_scoring_start_date = fields.Date.to_date(fields.Date.today() - timedelta(days=8))

    def _inverse_pls_start_date_str(self):
        """ As config_parameters does not accept Date field,
            we store the date formated string into a Char config field """
        for setting in self:
            if setting.predictive_lead_scoring_start_date:
                setting.predictive_lead_scoring_start_date_str = fields.Date.to_string(setting.predictive_lead_scoring_start_date)

    @api.depends('predictive_lead_scoring_fields')
    def _compute_predictive_lead_scoring_field_labels(self):
        for setting in self:
            if setting.predictive_lead_scoring_fields:
                field_names = [_('Stage')] + [field.name for field in setting.predictive_lead_scoring_fields]
                setting.predictive_lead_scoring_field_labels = _('%s and %s', ', '.join(field_names[:-1]), field_names[-1])
            else:
                setting.predictive_lead_scoring_field_labels = _('Stage')

    def set_values(self):
        group_use_lead_id = self.env['ir.model.data']._xmlid_to_res_id('crm.group_use_lead')
        has_group_lead_before = group_use_lead_id in self.env.user.groups_id.ids
        super(ResConfigSettings, self).set_values()
        # update use leads / opportunities setting on all teams according to settings update
        has_group_lead_after = group_use_lead_id in self.env.user.groups_id.ids
        if has_group_lead_before != has_group_lead_after:
            teams = self.env['crm.team'].search([])
            teams.filtered('use_opportunities').use_leads = has_group_lead_after
            for team in teams:
                team.alias_id.write(team._alias_get_creation_values())
        # synchronize cron with settings
        assign_cron = self.sudo().env.ref('crm.ir_cron_crm_lead_assign', raise_if_not_found=False)
        if assign_cron:
            # Writing on a cron tries to grab a write-lock on the table. This
            # could be avoided when saving a res.config without modifying this specific
            # configuration
            cron_vals = {
                'active': self.crm_use_auto_assignment and self.crm_auto_assignment_action == 'auto',
                'interval_type': self.crm_auto_assignment_interval_type,
                'interval_number': self.crm_auto_assignment_interval_number,
                # keep nextcall on cron as it is required whatever the setting
                'nextcall': self.crm_auto_assignment_run_datetime if self.crm_auto_assignment_run_datetime else assign_cron.nextcall,
            }
            cron_vals = {field_name: value for field_name, value in cron_vals.items() if assign_cron[field_name] != value}
            if cron_vals:
                assign_cron.write(cron_vals)
        # TDE FIXME: re create cron if not found ?

    def _get_crm_auto_assignmment_run_datetime(self, run_datetime, run_interval, run_interval_number):
        if not run_interval:
            return False
        if run_interval == 'manual':
            return run_datetime if run_datetime else False
        return fields.Datetime.now() + relativedelta(**{run_interval: run_interval_number})

    def action_crm_assign_leads(self):
        self.ensure_one()
        return self.env['crm.team'].search([('assignment_optout', '=', False)]).action_assign_leads(work_days=2, log=False)

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.osv import expression



class Partner(models.Model):
    _name = 'res.partner'
    _inherit = 'res.partner'

    team_id = fields.Many2one(
        'crm.team', string='Sales Team',
        compute='_compute_team_id',
        precompute=True,  # avoid queries post-create
        ondelete='set null', readonly=False, store=True)
    opportunity_ids = fields.One2many('crm.lead', 'partner_id', string='Opportunities', domain=[('type', '=', 'opportunity')])
    opportunity_count = fields.Integer("Opportunity", compute='_compute_opportunity_count')

    @api.model
    def default_get(self, fields):
        rec = super(Partner, self).default_get(fields)
        active_model = self.env.context.get('active_model')
        if active_model == 'crm.lead' and len(self.env.context.get('active_ids', [])) <= 1:
            lead = self.env[active_model].browse(self.env.context.get('active_id')).exists()
            if lead:
                rec.update(
                    phone=lead.phone,
                    mobile=lead.mobile,
                    function=lead.function,
                    title=lead.title.id,
                    website=lead.website,
                    street=lead.street,
                    street2=lead.street2,
                    city=lead.city,
                    state_id=lead.state_id.id,
                    country_id=lead.country_id.id,
                    zip=lead.zip,
                )
        return rec

    @api.depends('parent_id')
    def _compute_team_id(self):
        for partner in self.filtered(lambda partner: not partner.team_id and partner.company_type == 'person' and partner.parent_id.team_id):
            partner.team_id = partner.parent_id.team_id

    def _compute_opportunity_count(self):
        # retrieve all children partners and prefetch 'parent_id' on them
        all_partners = self.with_context(active_test=False).search_fetch(
            [('id', 'child_of', self.ids)], ['parent_id'],
        )

        opportunity_data = self.env['crm.lead'].with_context(active_test=False)._read_group(
            domain=[('partner_id', 'in', all_partners.ids)],
            groupby=['partner_id'], aggregates=['__count']
        )
        self_ids = set(self._ids)

        self.opportunity_count = 0
        for partner, count in opportunity_data:
            while partner:
                if partner.id in self_ids:
                    partner.opportunity_count += count
                partner = partner.parent_id

    def action_view_opportunity(self):
        '''
        This function returns an action that displays the opportunities from partner.
        '''
        action = self.env['ir.actions.act_window']._for_xml_id('crm.crm_lead_opportunities')
        action['context'] = {}
        if self.is_company:
            action['domain'] = [('partner_id.commercial_partner_id', '=', self.id)]
        else:
            action['domain'] = [('partner_id', '=', self.id)]
        action['domain'] = expression.AND([action['domain'], [('active', 'in', [True, False])]])
        return action

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Users(models.Model):
    _inherit = 'res.users'

    target_sales_won = fields.Integer('Won in Opportunities Target')
    target_sales_done = fields.Integer('Activities Done Target')

```

## File: models\utm.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class UtmCampaign(models.Model):
    _inherit = 'utm.campaign'

    use_leads = fields.Boolean('Use Leads', compute='_compute_use_leads')
    crm_lead_count = fields.Integer('Leads/Opportunities count', groups='sales_team.group_sale_salesman', compute="_compute_crm_lead_count")

    def _compute_use_leads(self):
        self.use_leads = self.env.user.has_group('crm.group_use_lead')

    def _compute_crm_lead_count(self):
        lead_data = self.env['crm.lead'].with_context(active_test=False)._read_group([
            ('campaign_id', 'in', self.ids)],
            ['campaign_id'], ['__count'])
        mapped_data = {campaign.id: count for campaign, count in lead_data}
        for campaign in self:
            campaign.crm_lead_count = mapped_data.get(campaign.id, 0)

    def action_redirect_to_leads_opportunities(self):
        view = 'crm.crm_lead_all_leads' if self.use_leads else 'crm.crm_lead_opportunities'
        action = self.env['ir.actions.act_window']._for_xml_id(view)
        action['view_mode'] = 'tree,kanban,graph,pivot,form,calendar'
        action['domain'] = [('campaign_id', 'in', self.ids)]
        action['context'] = {'active_test': False, 'create': False}
        return action

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_users
from . import calendar
from . import crm_lead
from . import crm_lost_reason
from . import crm_stage
from . import crm_team
from . import crm_team_member
from . import ir_config_parameter
from . import res_config_settings
from . import res_partner
from . import digest
from . import crm_lead_scoring_frequency
from . import utm
from . import crm_recurring_plan
from . import mail_activity

```

## File: populate\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, timedelta

from odoo import models
from odoo.tools import populate
from odoo.addons.crm.populate import tools


class CrmLead(models.Model):
    _inherit = 'crm.lead'
    _populate_dependencies = [
        'res.partner',  # customer
    ]
    _populate_sizes = {
        'small': 5,
        'medium': 150,
        'large': 400
    }

    def _populate_factories(self):
        partner_ids = self.env.registry.populated_models['res.partner']

        # phone based on country
        country_be, country_us, country_in = self.env.ref('base.be'), self.env.ref('base.us'), self.env.ref('base.in')
        phones_per_country = {
            country_be.id: [False, '+32456555432', '+32456555675', '+32456555627'],
            country_us.id: [False, '+15555564246', '+15558455343', '+15557129033'],
            country_in.id: [False, '+919755538077', '+917555765232', '+918555199309'],
            False: [False, '', '+3212345678', '003212345678', '12345678'],
        }

        # example of more complex generator composed of multiple sub generators
        # this define one subgenerator per "country"
        address_factories_groups = [
            [ # Falsy, 2 records
                ('street', populate.iterate([False, ''])),
                ('street2', populate.iterate([False, ''])),
                ('city', populate.iterate([False, ''])),
                ('zip', populate.iterate([False, ''])),
                ('country_id', populate.iterate([False])),
            ], [  # BE, 2 records
                ('street', populate.iterate(['Rue des Bourlottes {counter}', 'Rue Pinckaers {counter}'])),
                ('city', populate.iterate(['Brussels', 'Ramillies'])),
                ('zip', populate.iterate([1020, 1367])),
                ('country_id', populate.iterate([self.env.ref('base.be').id])),
            ], [  # US, 3 records
                ('street', populate.iterate(['Main street', '3th street {counter}', False])),
                ('street2', populate.iterate([False, '', 'Behind the tree {counter}'], [90, 5, 5])),
                ('city', populate.randomize(['San Fransisco', 'Los Angeles', '', False])),
                ('zip', populate.iterate([False, '', '50231'])),
                ('country_id', populate.iterate([self.env.ref('base.us').id])),
            ], [  # IN, 2 records
                ('street', populate.iterate(['Main Street', 'Some Street {counter}'])),
                ('city', populate.iterate(['ગાંધીનગર (Gandhinagar)'])),
                ('zip', populate.randomize(['382002', '382008'])),
                ('country_id', populate.randomize([self.env.ref('base.in').id])),
            ], [  # other corner cases, 2 records
                ('street', populate.iterate(['万泉寺村', 'საბჭოს სკვერი {counter}'])),
                ('city', populate.iterate(['北京市', 'თბილისი'])),
                ('zip', populate.iterate([False, 'UF47'])),
                ('country_id', populate.randomize([False] + self.env['res.country'].search([]).ids)),
            ]
        ]

        address_generators = [
            populate.chain_factories(address_factories, self._name)
            for address_factories in address_factories_groups
        ]

        def _compute_address(iterator, *args):
            r = populate.Random('res.partner+address_generator_selector')

            for values in iterator:
                if values['partner_id']:
                    yield {**values}
                else:
                    address_gen = r.choice(address_generators)
                    address_values = next(address_gen)
                    yield {**values, **address_values}

        def _compute_contact(iterator, *args):
            r = populate.Random('res.partner+contact_generator_selector')

            for values in iterator:
                if values['partner_id']:
                    yield {**values}
                else:
                    fn = r.choice(tools._p_forename_groups)
                    mn = r.choices(
                        [False] + tools._p_middlename_groups,
                        weights=[1] + [1 / (len(tools._p_middlename_groups) or 1)] * len(tools._p_middlename_groups)
                    )[0]
                    sn = r.choice(tools._p_surname_groups)
                    mn_wseparator = f' "{mn}" '
                    contact_name = f'{fn}{mn_wseparator}{sn}'

                    country_id = values['country_id']
                    if country_id not in phones_per_country.keys():
                        country_id = False
                    mobile = r.choice(phones_per_country[country_id])
                    phone = r.choice(phones_per_country[country_id])

                    yield {**values,
                           'contact_name': contact_name,
                           'mobile': mobile,
                           'phone': phone,
                          }

        def _compute_contact_name(values=None, counter=0, **kwargs):
            """ Generate lead names a bit better than lead_counter because this is Odoo. """
            partner_id = values['partner_id']
            complete = values['__complete']

            fn = kwargs['random'].choice(tools._p_forename_groups)
            mn = kwargs['random'].choices(
                [False] + tools._p_middlename_groups,
                weights=[1] + [1 / (len(tools._p_middlename_groups) or 1)] * len(tools._p_middlename_groups)
            )[0]
            sn = kwargs['random'].choice(tools._p_surname_groups)
            return  '%s%s %s (%s_%s (partner %s))' % (
                fn,
                ' "%s"' % mn if mn else '',
                sn,
                int(complete),
                counter,
                partner_id
            )

        def _compute_date_open(random=None, values=None, **kwargs):
            user_id = values['user_id']
            if user_id:
                delta = random.randint(0, 10)
                return datetime.now() - timedelta(days=delta)
            return False

        def _compute_name(values=None, counter=0, **kwargs):
            """ Generate lead names a bit better than lead_counter because this is Odoo. """
            complete = values['__complete']

            fn = kwargs['random'].choice(tools._case_prefix_groups)
            sn = kwargs['random'].choice(tools._case_object_groups)
            return  '%s %s (%s_%s)' % (
                fn,
                sn,
                int(complete),
                counter
            )

        return [
            ('partner_id', populate.iterate(
                [False] + partner_ids,
                [2] + [1 / (len(partner_ids) or 1)] * len(partner_ids))
            ),
            ('_address', _compute_address),  # uses partner_id
            ('_contact', _compute_contact),  # uses partner_id, country_id
            ('user_id', populate.iterate(
                [False],
                )
            ),
            ('date_open', populate.compute(_compute_date_open)),  # uses user_id
            ('name', populate.compute(_compute_name)),
            ('type', populate.iterate(['lead', 'opportunity'], [0.8, 0.2])),
        ]

```

## File: populate\tools.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# lead specific values
lead_values = {
    "active": {
        True: 0.99,
        False: 0.01
    },
}

# people
_p_forename_groups = [
    'Norbert', 'Jacqueline', 'Bastien', 'Monique',
    'Gretta', 'Juan Carlos', 'Ramon', 'Inrietta',
    'John', 'David', 'Luke', 'Beverly', 'Hillary', 'Christina',
]
_p_middlename_groups = [
    'Ugly Panda', 'Butterfly',
    'Neo-Tyrant', 'The Beast',
    'La Chignole', 'Le Cave', 'Gai Luron', 'La poutre',
]
_p_surname_groups = [
    'Poilvache', 'Tartopoils', 'Boitaclous', 'Boustifaille',
    'Campbell', 'Wonnegain', 'McShire', 'Scott', 'Doe',
]

# companies
_c_name_groups = [
    'Woody', 'Tree', 'Flower', 'Garden'
    'Furniture', 'Kitchen', 'Desk', 'Window',
    'Computer', 'CPU',
]
_c_surname_groups = [
    'Dealer', 'Sellers',
    'Carpenters', 'Foundry',
]

# case
_case_prefix_groups = [
    'Interested in', 'Looking for',
    'Issue with', 'Troubles with', 'Not sure where to find',
    'Want to buy', 'Want to sell', 'Want to paint',
]
_case_object_groups = [
    'Computer', 'Keyboard', 'Screen', 'Mouse',
    'Bed', 'Chair', 'Couch', 'Desk',
    'Toilet paper',
]

```

## File: populate\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_lead
from . import tools

```

## File: report\crm_activity_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, tools


class ActivityReport(models.Model):
    """ CRM Lead Analysis """

    _name = "crm.activity.report"
    _auto = False
    _description = "CRM Activity Analysis"
    _rec_name = 'id'

    date = fields.Datetime('Completion Date', readonly=True)
    lead_create_date = fields.Datetime('Creation Date', readonly=True)
    date_conversion = fields.Datetime('Conversion Date', readonly=True)
    date_deadline = fields.Date('Expected Closing', readonly=True)
    date_closed = fields.Datetime('Closed Date', readonly=True)
    author_id = fields.Many2one('res.partner', 'Assigned To', readonly=True)
    user_id = fields.Many2one('res.users', 'Salesperson', readonly=True)
    team_id = fields.Many2one('crm.team', 'Sales Team', readonly=True)
    lead_id = fields.Many2one('crm.lead', "Opportunity", readonly=True)
    body = fields.Html('Activity Description', readonly=True)
    subtype_id = fields.Many2one('mail.message.subtype', 'Subtype', readonly=True)
    mail_activity_type_id = fields.Many2one('mail.activity.type', 'Activity Type', readonly=True)
    country_id = fields.Many2one('res.country', 'Country', readonly=True)
    company_id = fields.Many2one('res.company', 'Company', readonly=True)
    stage_id = fields.Many2one('crm.stage', 'Stage', readonly=True)
    partner_id = fields.Many2one('res.partner', 'Customer', readonly=True)
    lead_type = fields.Selection(
        string='Type',
        selection=[('lead', 'Lead'), ('opportunity', 'Opportunity')],
        help="Type is used to separate Leads and Opportunities")
    active = fields.Boolean('Active', readonly=True)
    tag_ids = fields.Many2many(related="lead_id.tag_ids", readonly=True)

    def _select(self):
        return """
            SELECT
                m.id,
                l.create_date AS lead_create_date,
                l.date_conversion,
                l.date_deadline,
                l.date_closed,
                m.subtype_id,
                m.mail_activity_type_id,
                m.author_id,
                m.date,
                m.body,
                l.id as lead_id,
                l.user_id,
                l.team_id,
                l.country_id,
                l.company_id,
                l.stage_id,
                l.partner_id,
                l.type as lead_type,
                l.active
        """

    def _from(self):
        return """
            FROM mail_message AS m
        """

    def _join(self):
        return """
            JOIN crm_lead AS l ON m.res_id = l.id
        """

    def _where(self):
        return """
            WHERE
                m.model = 'crm.lead' AND (m.mail_activity_type_id IS NOT NULL)
        """

    def init(self):
        tools.drop_view_if_exists(self._cr, self._table)
        self._cr.execute("""
            CREATE OR REPLACE VIEW %s AS (
                %s
                %s
                %s
                %s
            )
        """ % (self._table, self._select(), self._from(), self._join(), self._where())
        )

```

## File: report\crm_activity_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="crm_activity_report_view_graph" model="ir.ui.view">
            <field name="name">crm.activity.report.graph</field>
            <field name="model">crm.activity.report</field>
            <field name="arch" type="xml">
                <graph string="Activities Analysis" sample="1">
                    <field name="mail_activity_type_id"/>
                    <field name="date" interval="month"/>
                </graph>
            </field>
        </record>

        <record id="crm_activity_report_view_pivot" model="ir.ui.view">
            <field name="name">crm.activity.report.pivot</field>
            <field name="model">crm.activity.report</field>
            <field name="arch" type="xml">
                <pivot string="Activities Analysis" sample="1">
                    <field name="mail_activity_type_id" type="col"/>
                    <field name="date" interval="month" type="row"/>
                </pivot>
            </field>
        </record>

        <record id="crm_activity_report_view_tree" model="ir.ui.view">
            <field name="name">crm.activity.report.tree</field>
            <field name="model">crm.activity.report</field>
            <field name="arch" type="xml">
                <tree default_order="date desc">
                    <field name="date"/>
                    <field name="author_id" widget="many2one_avatar"/>
                    <field name="mail_activity_type_id"/>
                    <field name="body" optional="hide"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="tag_ids" string="Lead Tags" optional="show" widget="many2many_tags" options="{'color_field': 'color'}"/>
                </tree>
            </field>
        </record>

        <record id="crm_activity_report_view_search" model="ir.ui.view">
            <field name="name">crm.activity.report.search</field>
            <field name="model">crm.activity.report</field>
            <field name="arch" type="xml">
                <search string="Activities Analysis">
                    <field name="mail_activity_type_id" string="Activity Type"/>
                    <field name="lead_id" string="Opportunity"/>
                    <field name="user_id" string="Salesperson"/>
                    <field name="team_id" context="{'invisible_team': False}"/>
                    <field name="author_id" string="Assigned To"/>
                    <field name="tag_ids" string="Lead Tags"/>
                    <separator groups="crm.group_use_lead"/>
                    <filter string="Leads" name="lead" domain="[('lead_type', '=', 'lead')]" help="Show only lead" groups="crm.group_use_lead"/>
                    <filter string="Opportunities" name="opportunity" domain="[('lead_type', '=', 'opportunity')]" help="Show only opportunity" groups="crm.group_use_lead"/>
                    <separator/>
                    <filter string="Won" name="won" domain="[('stage_id.is_won', '=', True)]"/>
                    <separator/>
                    <filter string="Trailing 12 months" name="completion_date" domain="[
                        ('date', '>=', (datetime.datetime.combine(context_today() + relativedelta(days=-365), datetime.time(0,0,0)).to_utc()).strftime('%Y-%m-%d %H:%M:%S')),
                        ('date', '&lt;=', (datetime.datetime.combine(context_today(), datetime.time(0,0,0)).to_utc()).strftime('%Y-%m-%d %H:%M:%S'))]"/>
                    <separator/>
                    <filter name="filter_date" date="date"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <group expand="1" string="Group By">
                        <filter string="Activity" name="group_by_activity_type" context="{'group_by': 'mail_activity_type_id'}"/>
                        <filter string="Type" name="group_by_subtype" context="{'group_by': 'subtype_id'}"/>
                        <filter string="Assigned To" name="group_by_author_id" context="{'group_by': 'author_id'}"/>
                        <filter string="Completion Date" name="group_by_completion_date" context="{'group_by': 'date:month'}"/>
                        <separator/>
                        <filter string="Salesperson" name="group_by_user_id" context="{'group_by': 'user_id'}"/>
                        <filter string="Sales Team" name="saleschannel" context="{'group_by': 'team_id'}"/>
                        <filter string="Stage" name="stage" context="{'group_by': 'stage_id'}"/>
                        <filter string="Company" name="company" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
                        <filter string="Creation Date" name="group_by_lead_date_creation" context="{'group_by': 'lead_create_date'}"/>
                        <filter string="Expected Closing" name="group_by_date_deadline" context="{'group_by': 'date_deadline'}"/>
                        <filter string="Closed Date" name="group_by_date_closed" context="{'group_by': 'date_closed'}"/>
                    </group>
                </search>
            </field>
        </record>

       <record id="crm_activity_report_action" model="ir.actions.act_window">
           <field name="name">Activities</field>
           <field name="res_model">crm.activity.report</field>
           <field name="view_mode">graph,pivot,tree</field>
           <field name="context">{
                'search_default_completion_date': 1,
                'pivot_column_groupby': ['subtype_id', 'mail_activity_type_id'],
                'pivot_row_groupby': ['date:month'],
                'graph_mode': 'bar',
                'graph_groupbys': ['date:month', 'subtype_id'],
            }</field>
            <field name="domain">[]</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No data yet!
                </p><p>
                    Start scheduling activities on your opportunities
                </p>
            </field>
       </record>

        <record id="crm_activity_report_action_team" model="ir.actions.act_window">
            <field name="name">Pipeline Activities</field>
            <field name="res_model">crm.activity.report</field>
            <field name="view_mode">graph,pivot,tree</field>
            <field name="context">{'search_default_team_id': active_id}</field>
            <field name="domain">[]</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No data yet!
                </p><p>
                    Start scheduling activities on your opportunities
                </p>
            </field>
        </record>

</odoo>

```

## File: report\crm_opportunity_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="crm_lead_view_tree_opportunity_reporting" model="ir.ui.view">
            <field name="name">crm.lead.tree.opportunity.reporting</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_case_tree_view_oppor"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//button[@name='%(crm.action_lead_mail_compose)d']" position="replace"/>
                <xpath expr="//button[@name='action_snooze']" position="replace"/>
            </field>
        </record>

        <!-- Opportunities by user and team pivot View -->
        <record id="crm_opportunity_report_view_pivot" model="ir.ui.view">
            <field name="name">crm.opportunity.report.pivot</field>
            <field name="model">crm.lead</field>
            <field name="priority">60</field>
            <field name="arch" type="xml">
                <pivot string="Pipeline Analysis" sample="1">
                    <field name="create_date" interval="month" type="col"/>
                    <field name="stage_id" type="row"/>
                    <field name="prorated_revenue" type="measure"/>
                    <field name="color" invisible="1"/>
                    <field name="message_bounce" invisible="1"/>
                    <field name="probability" invisible="1"/>
                    <field name="automated_probability" invisible="1"/>
                    <field name="recurring_revenue_monthly" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue_monthly_prorated" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue_prorated" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                </pivot>
            </field>
        </record>

        <record id="crm_opportunity_report_view_pivot_lead" model="ir.ui.view">
            <field name="name">crm.opportunity.report.view.pivot.lead</field>
            <field name="model">crm.lead</field>
            <field name="priority">60</field>
            <field name="arch" type="xml">
                <pivot string="Leads Analysis" sample="1">
                    <field name="create_date" interval="month" type="row"/>
                    <field name="team_id" type="col"/>
                    <field name="color" invisible="1"/>
                    <field name="message_bounce" invisible="1"/>
                    <field name="probability" invisible="1"/>
                    <field name="automated_probability" invisible="1"/>
                    <field name="recurring_revenue_monthly" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue_monthly_prorated" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue_prorated" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                </pivot>
            </field>
        </record>

        <!-- Opportunities by user and team Graph View -->
        <record id="crm_opportunity_report_view_graph" model="ir.ui.view">
            <field name="name">crm.opportunity.report.graph</field>
            <field name="model">crm.lead</field>
            <field name="arch" type="xml">
                <graph string="Pipeline Analysis" sample="1">
                    <field name="stage_id"/>
                    <field name="date_deadline" interval="month"/>
                    <field name="prorated_revenue" type="measure"/>
                    <field name="color" invisible="1"/>
                    <field name="message_bounce" invisible="1"/>
                    <field name="probability" invisible="1"/>
                    <field name="automated_probability" invisible="1"/>
                    <field name="recurring_revenue_monthly" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue_monthly_prorated" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue_prorated" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                </graph>
            </field>
        </record>

        <record id="crm_opportunity_report_view_graph_lead" model="ir.ui.view">
            <field name="name">crm.opportunity.report.graph.lead</field>
            <field name="model">crm.lead</field>
            <field name="priority">20</field>
            <field name="arch" type="xml">
                <graph string="Leads Analysis" sample="1">
                    <field name="create_date" interval="month"/>
                    <field name="team_id"/>
                    <field name="color" invisible="1"/>
                    <field name="automated_probability" invisible="1"/>
                    <field name="message_bounce" invisible="1"/>
                    <field name="probability" invisible="1"/>
                    <field name="recurring_revenue_monthly" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue_monthly_prorated" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue_prorated" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                </graph>
            </field>
        </record>

        <!-- Opportunities by user and team Search View -->
        <record id="crm_opportunity_report_view_search" model="ir.ui.view">
            <field name="name">crm.lead.search</field>
            <field name="model">crm.lead</field>
            <field name="priority">32</field>
            <field name="arch" type="xml">
                <search string="Opportunities Analysis">
                    <filter string="My Pipeline" name="my"
                            domain="[('user_id', '=', uid)]"/>
                    <separator/>
                    <filter name="filter_opportunity" string="Opportunities" domain="[('type','=','opportunity')]" help="Show only opportunities" groups="crm.group_use_lead"/>
                    <filter name="filter_lead" string="Leads" domain="[('type','=', 'lead')]" help="Show only leads" groups="crm.group_use_lead"/>
                    <separator/>
                    <filter string="Active" name="filter_active"
                            domain="[('active', '=', True)]"/>
                    <filter string="Inactive" name="filter_inactive"
                            domain="[('active', '=', False)]"/>
                    <filter string="Won" name="won"
                            domain="[('probability', '=', 100)]"/>
                    <filter string="Lost" name="lost"
                            domain="[('probability', '=', 0), ('active', '=', False)]"/>
                    <field name="team_id" context="{'invisible_team': False}"/>
                    <field name="user_id" string="Salesperson"/>
                    <separator/>
                    <filter name="filter_create_date" date="create_date" default_period="this_year"/>
                    <filter string="Expected Closing" name="filter_date_deadline" date="date_deadline"/>
                    <filter string="Date Closed" name="date_closed_filter" date="date_closed"/>
                    <group expand="0" string="Extended Filters">
                        <field name="partner_id" filter_domain="[('partner_id','child_of',self)]"/>
                        <field name="stage_id" domain="['|', ('team_id', '=', False), ('team_id', '=', 'team_id')]"/>
                        <field name="campaign_id"/>
                        <field name="medium_id"/>
                        <field name="source_id"/>
                        <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                        <newline/>
                        <field name="create_date"/>
                        <field name="date_open"/>
                        <field name="date_closed"/>
                    </group>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <group expand="1" string="Group By">
                        <filter string="Salesperson" name="salesperson" context="{'group_by':'user_id'}" />
                        <filter string="Sales Team" name="saleschannel" context="{'group_by':'team_id'}"/>
                        <filter string="City" name="city" context="{'group_by':'city'}" />
                        <filter string="Country" name="country" context="{'group_by':'country_id'}" />
                        <filter string="Company" name="company" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                        <filter string="Stage" name="stage" context="{'group_by':'stage_id'}"/>
                        <filter string="Campaign" name="compaign" domain="[]" context="{'group_by':'campaign_id'}"/>
                        <filter string="Medium" name="medium" domain="[]" context="{'group_by':'medium_id'}"/>
                        <filter string="Source" name="source" domain="[]" context="{'group_by':'source_id'}"/>
                        <separator orientation="vertical" />
                        <filter string="Creation Date" context="{'group_by':'create_date:month'}" name="month"/>
                        <filter string="Conversion Date" context="{'group_by':'date_conversion:month'}" name="conversion_date" help="Conversion Date from Lead to Opportunity" groups="crm.group_use_lead"/>
                        <filter string="Expected Closing" context="{'group_by':'date_deadline:month'}" name="date_deadline"/>
                        <filter string="Closed Date" context="{'group_by':'date_closed'}" name="date_closed_groupby"/>
                        <filter string="Lost Reason" name="lostreason" context="{'group_by':'lost_reason_id'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="crm_lead_view_tree_reporting" model="ir.ui.view">
            <field name="name">crm.lead.tree.lead.reporting</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_case_tree_view_leads"/>
            <field name="mode">primary</field>
            <field name="priority">24</field>
            <field name="arch" type="xml">
                <xpath expr="field[@name='city']" position="attributes">
                    <attribute name="optional">hide</attribute>
                </xpath>
                <xpath expr="field[@name='probability']" position="before">
                    <field name="type" groups="crm.group_use_lead" optional="show"/>
                    <field name="stage_id" optional="show"/>
                </xpath>
                <xpath expr="field[@name='probability']" position="after">
                    <field name="lost_reason_id" optional="hide"/>
                </xpath>
            </field>
        </record>

        <record id="crm_opportunity_report_action" model="ir.actions.act_window">
            <field name="name">Pipeline Analysis</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">graph,pivot,tree,form</field>
            <field name="search_view_id" ref="crm.crm_opportunity_report_view_search"/>
            <field name="context">{'search_default_filter_opportunity': True, 'search_default_current': True}</field>
            <field name="view_ids"
                   eval="[(5, 0, 0),
                          (0, 0, {'view_mode': 'graph', 'view_id': ref('crm_opportunity_report_view_graph')}),
                          (0, 0, {'view_mode': 'pivot', 'view_id': ref('crm_opportunity_report_view_pivot')}),
                          (0, 0, {'view_mode': 'tree', 'view_id': ref('crm_lead_view_tree_opportunity_reporting')})]"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No data found!
                </p><p>
                    Use this menu to have an overview of your Pipeline.
                </p>
            </field>
        </record>

        <record id="crm_opportunity_report_action_lead" model="ir.actions.act_window">
            <field name="name">Leads Analysis</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">graph,pivot,tree</field>
            <field name="search_view_id" ref="crm.crm_opportunity_report_view_search"/>
            <field name="context">{
                'search_default_filter_active': 1,
                'search_default_filter_inactive': 1,
                'search_default_filter_create_date': 1,
            }</field>
            <field name="view_ids"
                   eval="[(5, 0, 0),
                          (0, 0, {'view_mode': 'graph', 'view_id': ref('crm_opportunity_report_view_graph_lead')}),
                          (0, 0, {'view_mode': 'pivot', 'view_id': ref('crm_opportunity_report_view_pivot_lead')}),
                          (0, 0, {'view_mode': 'form', 'view_id': ref('crm_lead_view_form')}),
                          (0, 0, {'view_mode': 'tree', 'view_id': ref('crm_lead_view_tree_reporting')}),
                         ]"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No data found!
                </p><p>
                    This analysis shows you how many leads have been created per month.
                </p>
            </field>
        </record>

</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_activity_report

```

## File: security\crm_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<odoo>

    <record id="group_use_lead" model="res.groups">
        <field name="name">Show Lead Menu</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_use_recurring_revenues" model="res.groups">
        <field name="name">Show Recurring Revenues Menu</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="contacts.res_partner_menu_config" model="ir.ui.menu">
        <field name="groups_id" eval="[(4, ref('sales_team.group_sale_manager'))]"/>
    </record>

<data noupdate="1">

    <record id="crm_rule_personal_lead" model="ir.rule">
        <field name="name">Personal Leads</field>
        <field ref="model_crm_lead" name="model_id"/>
        <field name="domain_force">['|',('user_id','=',user.id),('user_id','=',False)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
    </record>

    <record id="crm_lead_company_rule" model="ir.rule">
        <field name="name">CRM Lead Multi-Company</field>
        <field name="model_id" ref="model_crm_lead"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="crm_rule_all_lead" model="ir.rule">
        <field name="name">All Leads</field>
        <field ref="model_crm_lead" name="model_id"/>
        <field name="domain_force">[(1,'=',1)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman_all_leads'))]"/>
    </record>

    <record id="crm_activity_report_rule_all_activities" model="ir.rule">
        <field name="name">All Activities</field>
        <field ref="model_crm_activity_report" name="model_id"/>
        <field name="domain_force">[(1,'=',1)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman_all_leads'))]"/>
    </record>

    <record id="crm_activity_report_rule_personal_activities" model="ir.rule">
        <field name="name">Personal Activities</field>
        <field ref="model_crm_activity_report" name="model_id"/>
        <field name="domain_force">['|',('user_id','=',user.id),('user_id','=',False)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
    </record>

    <record id="crm_activity_report_rule_multi_company" model="ir.rule">
        <field name="name">CRM Lead Multi-Company</field>
        <field name="model_id" ref="model_crm_activity_report"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="mail_plan_rule_group_sale_manager_lead" model="ir.rule">
        <field name="name">Manager can manage lead plans</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_manager'))]"/>
        <field name="model_id" ref="mail.model_mail_activity_plan"/>
        <field name="domain_force">[('res_model', '=', 'crm.lead')]</field>
        <field name="perm_read" eval="False"/>
    </record>

    <record id="mail_plan_templates_rule_group_sale_manager_lead" model="ir.rule">
        <field name="name">Manager can manage lead plan templates</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_manager'))]"/>
        <field name="model_id" ref="mail.model_mail_activity_plan_template"/>
        <field name="domain_force">[('plan_id.res_model', '=', 'crm.lead')]</field>
        <field name="perm_read" eval="False"/>
    </record>

</data>

</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_crm_lead_manager,crm.lead.manager,model_crm_lead,sales_team.group_sale_manager,1,1,1,1
access_crm_lead,crm.lead,model_crm_lead,sales_team.group_sale_salesman,1,1,1,0
access_crm_stage,crm.stage,model_crm_stage,base.group_user,1,0,0,0
access_crm_stage_manager,crm.stage,model_crm_stage,sales_team.group_sale_manager,1,1,1,1
access_res_partner_manager,res.partner.crm.manager,base.model_res_partner,sales_team.group_sale_manager,1,0,0,0
access_res_partner_category_manager,res.partner.category.crm.manager,base.model_res_partner_category,sales_team.group_sale_manager,1,0,0,0
access_res_partner,res.partner.crm.user,base.model_res_partner,sales_team.group_sale_salesman,1,1,1,0
access_res_partner_category,res.partner.category.crm.user,base.model_res_partner_category,sales_team.group_sale_salesman,1,1,1,0
access_crm_lost_reason_manager,crm.lost.reason.manager,model_crm_lost_reason,sales_team.group_sale_manager,1,1,1,1
access_crm_lost_reason_salesman,crm.lost.reason.salesman,model_crm_lost_reason,sales_team.group_sale_salesman,1,0,0,0
access_crm_lost_reason_user,crm.lost.reason.user,model_crm_lost_reason,base.group_user,1,0,0,0
access_crm_activity_report_user,crm.activity.report.user,model_crm_activity_report,base.group_user,0,0,0,0
access_crm_activity_report_salesman,crm.activity.report.salesman,model_crm_activity_report,sales_team.group_sale_salesman,1,0,0,0
access_calendar_event_manager,calendar.event.manager,calendar.model_calendar_event,sales_team.group_sale_manager,1,1,1,1
access_calendar_event,calendar.event,calendar.model_calendar_event,sales_team.group_sale_salesman,1,1,1,0
access_calendar_event_type_sale_manager,calendar.event.type.manager,calendar.model_calendar_event_type,sales_team.group_sale_manager,1,1,1,0
access_calendar_event_type_sale_user,calendar.event.type.user,calendar.model_calendar_event_type,base.group_user,1,0,0,0
access_calendar_event_type_sale_salesman,calendar.event.type.salesman,calendar.model_calendar_event_type,sales_team.group_sale_salesman,1,0,0,0
access_mail_activity_type_sale_manager,mail.activity.type.sale.manager,mail.model_mail_activity_type,sales_team.group_sale_manager,1,1,1,1
access_crm_lead_scoring_frequency,access_crm_lead_scoring_frequency,model_crm_lead_scoring_frequency,sales_team.group_sale_salesman,1,0,0,0
access_crm_lead_scoring_frequency_system,access_crm_lead_scoring_frequency_system,model_crm_lead_scoring_frequency,base.group_system,1,0,0,0
access_crm_lead_scoring_frequency_field,access_crm_lead_scoring_frequency_field,model_crm_lead_scoring_frequency_field,sales_team.group_sale_salesman,1,0,0,0
access_crm_lead_scoring_frequency_field_system,access_crm_lead_scoring_frequency_field_system,model_crm_lead_scoring_frequency_field,base.group_system,1,0,0,0
access_crm_lead_lost,access.crm.lead.lost,model_crm_lead_lost,sales_team.group_sale_salesman,1,1,1,0
access_crm_lead2opportunity_partner,access.crm.lead2opportunity.partner,model_crm_lead2opportunity_partner,sales_team.group_sale_salesman,1,1,1,0
access_crm_lead2opportunity_partner_mass,access.crm.lead2opportunity.partner.mass,model_crm_lead2opportunity_partner_mass,sales_team.group_sale_salesman,1,1,1,0
access_crm_merge_opportunity,access.crm.merge.opportunity,model_crm_merge_opportunity,sales_team.group_sale_salesman,1,1,1,0
crm_recurring_plan_access_manager,crm.recurring.plan.access.manager,model_crm_recurring_plan,sales_team.group_sale_manager,1,1,1,1
crm_recurring_plan_access_salesman,crm.recurring.plan.access.salesman,model_crm_recurring_plan,sales_team.group_sale_salesman,1,0,0,0
crm_lead_pls_update_access_system,crm.lead.pls.update.access.system,model_crm_lead_pls_update,base.group_erp_manager,1,1,1,1
access_mail_activity_plan_sale_manager,mail.activity.plan.sale.manager,mail.model_mail_activity_plan,sales_team.group_sale_manager,1,1,1,1
access_mail_activity_plan_template_sale_manager,mail.activity.plan.template.sale.manager,mail.model_mail_activity_plan_template,sales_team.group_sale_manager,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M45.873 9c3.52.041 5.504 4.474 3.013 6.964l-8.424 8.419L36.5 27 25 9h20.873ZM10.946 25.79a3.972 3.972 0 0 0 0 5.618l8.433 8.43a3.977 3.977 0 0 0 5.623 0L23.5 36 15 27l-4.055-1.212Z" fill="#985184"/><path d="M1.114 15.964C-1.377 13.474.608 9.041 4.128 9H25l15.461 15.383a3.972 3.972 0 0 1 0 5.62l-9.84 9.833a3.977 3.977 0 0 1-5.621 0L1.114 15.964Z" fill="#1AD3BB"/><path d="M25 39.837a3.972 3.972 0 0 0 0-5.62l-8.434-8.428a3.977 3.977 0 0 0-5.623 0L25 39.837Zm-7.38-23.531L25 9l9.136 9.062a5.966 5.966 0 0 0-8.433 0l-3.163 3.16a3.48 3.48 0 0 1-4.92 0 3.475 3.475 0 0 1 0-4.916Z" fill="#005E7A"/></svg>

```

## File: static\src\activity_menu_patch.js

```javascript
/* @odoo-module */

import { ActivityMenu } from "@mail/core/web/activity_menu";
import { patch } from "@web/core/utils/patch";

patch(ActivityMenu.prototype, {
    availableViews(group) {
        if (group.model === "crm.lead") {
            return [
                [false, "list"],
                [false, "kanban"],
                [false, "form"],
                [false, "calendar"],
                [false, "pivot"],
                [false, "graph"],
                [false, "activity"],
            ];
        }
        return super.availableViews(...arguments);
    },

    openActivityGroup(group, filter = "all") {
        // fetch the data from the button otherwise fetch the ones from the parent (.o_ActivityMenuView_activityGroup).
        const context = {};
        if (group.model === "crm.lead") {
            document.body.click(); // hack to close dropdown
            if (filter === "my") {
                context["search_default_activities_overdue"] = 1;
                context["search_default_activities_today"] = 1;
            } else {
                context["search_default_activities_" + filter] = 1;
            }
            // Necessary because activity_ids of mail.activity.mixin has auto_join
            // So, duplicates are faking the count and "Load more" doesn't show up
            context["force_search_count"] = 1;
            this.action.doAction("crm.crm_lead_action_my_activities", {
                additionalContext: context,
                clearBreadcrumbs: true,
            });
        } else {
            return super.openActivityGroup(group, filter);
        }
    },
});

```

## File: static\src\js\tours\crm.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { stepUtils } from "@web_tour/tour_service/tour_utils";

import { markup } from "@odoo/owl";

registry.category("web_tour.tours").add('crm_tour', {
    url: "/web",
    rainbowManMessage: _t("Congrats, best of luck catching such big fish! :)"),
    sequence: 10,
    steps: () => [stepUtils.showAppsMenuItem(), {
    trigger: '.o_app[data-menu-xmlid="crm.crm_menu_root"]',
    content: markup(_t('Ready to boost your sales? Let\'s have a look at your <b>Pipeline</b>.')),
    position: 'bottom',
    edition: 'community',
}, {
    trigger: '.o_app[data-menu-xmlid="crm.crm_menu_root"]',
    content: markup(_t('Ready to boost your sales? Let\'s have a look at your <b>Pipeline</b>.')),
    position: 'bottom',
    edition: 'enterprise',
}, {
    trigger: '.o-kanban-button-new',
    extra_trigger: '.o_opportunity_kanban',
    content: markup(_t("<b>Create your first opportunity.</b>")),
    position: 'bottom',
}, {
    trigger: ".o_kanban_quick_create .o_field_widget[name='partner_id']",
    content: markup(_t('<b>Write a few letters</b> to look for a company, or create a new one.')),
    position: "top",
    run: function (actions) {
        actions.text("Brandon Freeman", this.$anchor.find("input"));
    },
}, {
    trigger: ".ui-menu-item > a",
    auto: true,
    in_modal: false,
}, {
    trigger: ".o_kanban_quick_create .o_kanban_add",
    content: markup(_t("Now, <b>add your Opportunity</b> to your Pipeline.")),
    position: "bottom",
}, {
    trigger: ".o_opportunity_kanban .o_kanban_group:first-child .o_kanban_record:last-of-type .oe_kanban_content",
    extra_trigger: ".o_opportunity_kanban",
    content: markup(_t("<b>Drag &amp; drop opportunities</b> between columns as you progress in your sales cycle.")),
    position: "right",
    run: "drag_and_drop_native .o_opportunity_kanban .o_kanban_group:eq(2) ",
}, {
    // Choose the element that is not going to be moved by the previous step.
    trigger: ".o_opportunity_kanban .o_kanban_group:nth-child(2) .o_kanban_record .o-mail-ActivityButton",
    extra_trigger: ".o_opportunity_kanban",
    content: markup(_t("Looks like nothing is planned. :(<br><br><i>Tip: Schedule activities to keep track of everything you have to do!</i>")),
    position: "bottom",
}, {
    trigger: ".o-mail-ActivityListPopover button:contains(Schedule an activity)",
    extra_trigger: ".o_opportunity_kanban",
    content: markup(_t("Let's <b>Schedule an Activity.</b>")),
    position: "bottom",
    width: 200,
}, {
    trigger: '.modal-footer button[name="action_schedule_activities"]',
    content: markup(_t("All set. Let’s <b>Schedule</b> it.")),
    position: "top",  // dot NOT move to bottom, it would cause a resize flicker, see task-2476595
    run: function (actions) {
        actions.auto('.modal-footer button[special=cancel]');
    },
}, {
    id: "drag_opportunity_to_won_step",
    trigger: ".o_opportunity_kanban .o_kanban_record:last-of-type",
    content: markup(_t("Drag your opportunity to <b>Won</b> when you get the deal. Congrats!")),
    position: "bottom",
    run: "drag_and_drop_native .o_opportunity_kanban .o_kanban_group:eq(3) ",
},  {
    trigger: ".o_kanban_record",
    extra_trigger: ".o_opportunity_kanban",
    content: _t("Let’s have a look at an Opportunity."),
    position: "right",
    run: function (actions) {
        actions.auto(".o_kanban_record");
    },
}, {
    trigger: ".o_lead_opportunity_form .o_statusbar_status",
    content: _t("You can make your opportunity advance through your pipeline from here."),
    position: "bottom"
}, {
    trigger: ".breadcrumb-item:not(.active):first",
    content: _t("Click on the breadcrumb to go back to your Pipeline. Odoo will save all modifications as you navigate."),
    position: "bottom",
    run: function (actions) {
        actions.auto(".breadcrumb-item:not(.active):last");
    }
}]});

```

## File: static\src\views\check_rainbowman_message.js

```javascript
/** @odoo-module **/

export async function checkRainbowmanMessage(orm, effect, recordId) {
    const message = await orm.call("crm.lead", "get_rainbowman_message", [[recordId]]);
    if (message) {
        effect.add({
            message,
            type: "rainbow_man",
        });
    }
}

```

## File: static\src\views\fill_temporal_service.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import {
    serializeDate,
    serializeDateTime,
} from "@web/core/l10n/dates";

/**
 * Configuration depending on the granularity, using Luxon DateTime objects:
 * @param {function} startOf function to get a DateTime at the beginning of a period
 *                           from another DateTime.
 * @param {int} cycle amount of 'granularity' periods constituting a cycle. The cycle duration
 *                    is arbitrary for each granularity:
 * cycle    ---    granularity
 * ___________________________
 * 1 day           hour
 * 1 week          day
 * 1 week          week    # there is no greater time period that takes an integer amount of weeks
 * 1 year          month
 * 1 year          quarter
 * 1 year          year    # we are not using a greater time period in Odoo (yet)
 * @param {int} cyclePos function to get the position (index) in the cycle from a DateTime.
 *                       {1} is the first index. {+1} is used for properties which have an index
 *                       starting from 0, to standardize between granularities.
 */
export const GRANULARITY_TABLE = {
    hour: {
        startOf: (x) => x.startOf("hour"),
        cycle: 24,
        cyclePos: (x) => x.hour + 1,
    },
    day: {
        startOf: (x) => x.startOf("day"),
        cycle: 7,
        cyclePos: (x) => x.weekday,
    },
    week: {
        startOf: (x) => x.startOf("week"),
        cycle: 1,
        cyclePos: (x) => 1,
    },
    month: {
        startOf: (x) => x.startOf("month"),
        cycle: 12,
        cyclePos: (x) => x.month,
    },
    quarter: {
        startOf: (x) => x.startOf("quarter"),
        cycle: 4,
        cyclePos: (x) => x.quarter,
    },
    year: {
        startOf: (x) => x.startOf("year"),
        cycle: 1,
        cyclePos: (x) => 1,
    },
};

/**
 * fill_temporal period:
 *   Represents a specific date/time range for a specific model, field and granularity.
 *
 * It is used to add new domain and context constraints related to a specific date/time
 * field, in order to configure the _read_group_fill_temporal (see core models.py)
 * method. It will be used when we want to get continuous groups in chronological
 * order in a specific date/time range.
 */
export class FillTemporalPeriod {
    /**
     * This constructor is meant to be used only by the FillTemporalService (see below)
     *
     * @param {string} modelName directly taken from model.loadParams.modelName.
     *                           this is the `res_model` from the action (i.e. `crm.lead`)
     * @param {Object} field a dictionary with keys "name" and "type".
     *                        name: Name of the field on which the fill_temporal should apply
     *                              (i.e. 'date_deadline')
     *                        type: 'date' or 'datetime'
     * @param {string} granularity can either be : hour, day, week, month, quarter, year
     * @param {integer} minGroups minimum amount of groups to display, regardless of other
     *                            constraints
     */
    constructor(modelName, field, granularity, minGroups) {
        this.modelName = modelName;
        this.field = field;
        this.granularity = granularity || "month";
        this.setMinGroups(minGroups);

        this._computeStart();
        this._computeEnd();
    }
    /**
     * Compute this.start: the DateTime for the start of the period containing
     * the current time ("now").
     * i.e. 2020-10-01 13:43:17 -> the current "hour" DateTime started at:
     *      2020-10-01 13:00:00
     *
     * @private
     */
    _computeStart() {
        this.start = GRANULARITY_TABLE[this.granularity].startOf(luxon.DateTime.now());
    }
    /**
     * Compute this.end: the DateTime for the end of the fill_temporal period.
     * This bound is exclusive.
     * The fill_temporal period is the number of [granularity] from [start] to the end of the
     * [cycle] reached after adding [minGroups]
     * i.e. we are in october 2020 :
     *      [start] = 2020-10-01
     *      [granularity] = 'month',
     *      [cycle] = 12
     *      [minGroups] = 4,
     *      => fillTemporalPeriod = 15 months (until end of december 2021)
     *
     * @private
     */
    _computeEnd() {
        const cycle = GRANULARITY_TABLE[this.granularity].cycle;
        const cyclePos = GRANULARITY_TABLE[this.granularity].cyclePos(this.start);
        // fillTemporalPeriod formula explanation :
        // We want to know how many steps need to be taken from the current position until the end
        // of the cycle reached after guaranteeing minGroups positions. Let's call this cycle (C).
        //
        // (1) compute the steps needed to reach the last position of the current cycle, from the
        //     current position:
        //     {cycle - cyclePos}
        //
        // (2) ignore {minGroups - 1} steps from the position reached in (1). Now, the current
        //     position is somewhere in (C). One step from minGroups is reserved to reach the first
        //     position after (C), hence {-1}
        //
        // (3) compute the additional steps needed to reach the last position of (C), from the
        //     position reached in (2):
        //     {cycle - (minGroups - 1) % cycle}
        //
        // (4) combine (1) and (3), the sum should not be greater than a full cycle (-> truncate):
        //     {(2 * cycle - (minGroups - 1) % cycle - cyclePos) % cycle}
        //
        // (5) add minGroups!
        const fillTemporalPeriod = ((2 * cycle - ((this.minGroups - 1) % cycle) - cyclePos) % cycle) + this.minGroups;
        this.end = this.start.plus({[`${this.granularity}s`]: fillTemporalPeriod});
        this.computedEnd = true;
    }
    /**
     * The server needs a date/time in UTC, but we don't want a day shift in case
     * of dates, even if the date is not in UTC
     *
     * @param {DateTime} bound the DateTime to be formatted (this.start or this.end)
     */
    _getFormattedServerDate(bound) {
        if (this.field.type === "date") {
            return serializeDate(bound);
        } else {
            return serializeDateTime(bound);
        }
    }
    /**
     * @param {Object} configuration
     * @param {Array[]} [domain]
     * @param {boolean} [forceStartBound=true] whether this.start DateTime must be used as a domain
     *                                         constraint to limit read_group results or not
     * @param {boolean} [forceEndBound=true] whether this.end DateTime must be used as a domain
     *                                       constraint to limit read_group results or not
     * @returns {Array[]} new domain
     */
    getDomain({ domain, forceStartBound = true, forceEndBound = true }) {
        if (!forceEndBound && !forceStartBound) {
            return domain;
        }
        const originalDomain = domain.length ? ["&", ...domain] : [];
        const defaultDomain = ["|", [this.field.name, "=", false]];
        const linkDomain = forceStartBound && forceEndBound ? ["&"] : [];
        const startDomain = !forceStartBound ? [] : [[this.field.name, ">=", this._getFormattedServerDate(this.start)]];
        const endDomain = !forceEndBound ? [] : [[this.field.name, "<", this._getFormattedServerDate(this.end)]];
        return [...originalDomain, ...defaultDomain, ...linkDomain, ...startDomain, ...endDomain];
    }
    /**
     * The default value of forceFillingTo is false when this.end is the
     * computed one, and true when it is manually set. This is because the default value of
     * this.end is computed without any knowledge of the existing data, and as such, we only
     * want to get continuous groups until the last group with data (no need to force until
     * this.end). On the contrary, when we set this.end, this means that we want groups until
     * that date.
     *
     * @param {Object} configuration
     * @param {Object} [context]
     * @param {boolean} [forceFillingFrom=true] fill_temporal must apply from:
     *                                          true: this.start
     *                                          false: the first group with at least one record
     * @param {boolean} [forceFillingTo=!this.computedEnd] fill_temporal must apply until:
     *                                          true: this.end
     *                                          false: the last group with at least one record
     * @returns {Object} new context
     */
    getContext({ context, forceFillingFrom = true, forceFillingTo = !this.computedEnd }) {
        const fillTemporal = {
            min_groups: this.minGroups,
        };
        if (forceFillingFrom) {
            fillTemporal.fill_from = this._getFormattedServerDate(this.start);
        }
        if (forceFillingTo) {
            // smallest time interval used in Odoo for the current date type
            const minGranularity = this.field.type === "date" ? "days" : "seconds";
            fillTemporal.fill_to = this._getFormattedServerDate(this.end.minus({[minGranularity]: 1}));
        }
        context = { ...context, fill_temporal: fillTemporal };
        return context;
    }
    /**
     * @param {integer} minGroups minimum amount of groups to display, regardless of other
     *                            constraints
     */
    setMinGroups(minGroups) {
        this.minGroups = minGroups || 1;
    }
    /**
     * sets the end of the period to the desired DateTime. It must be greater
     * than start. Changes the default behavior of getContext forceFillingTo
     * (becomes true instead of false)
     *
     * @param {DateTime} end
     */
    setEnd(end) {
        this.end = luxon.DateTime.max(this.start, end);
        this.computedEnd = false;
    }
    /**
     * sets the start of the period to the desired DateTime. It must be smaller than end
     *
     * @param {DateTime} start
     */
    setStart(start) {
        this.start = luxon.DateTime.min(this.end, start);
    }
    /**
     * Adds one "granularity" period to [this.end], to expand the current fill_temporal period
     */
    expand() {
        this.setEnd(this.end.plus({[`${this.granularity}s`]: 1}));
    }
}

/**
 * fill_temporal Service
 *
 * This service will be used to generate or retrieve fill_temporal periods
 *
 * A specific fill_temporal period configuration will always refer to the same instance
 * unless forceRecompute is true
 */
export const fillTemporalService = {
    start() {
        const _fillTemporalPeriods = {};

        /**
         * Get a fill_temporal period according to the configuration.
         * The default initial fill_temporal period is the number of [granularity] from [start]
         * to the end of the [cycle] reached after adding [minGroups]
         * i.e. we are in october 2020 :
         *      [start] = 2020-10-01
         *      [granularity] = 'month',
         *      [cycle] = 12 (one year)
         *      [minGroups] = 4,
         *      => fillTemporalPeriod = 15 months (until the end of december 2021)
         * Once created, a fill_temporal period for a specific configuration will be stored
         * until requested again. This allows to manipulate the period and store the changes
         * to it. This also allows to keep the configuration when switching to another view
         *
         * @param {Object} configuration
         * @param {string} [modelName] directly taken from model.loadParams.modelName.
         *                             this is the `res_model` from the action (i.e. `crm.lead`)
         * @param {Object} [field] a dictionary with keys "name" and "type".
         * @param {string} [field.name] name of the field on which the fill_temporal should apply
         *                              (i.e. 'date_deadline')
         * @param {string} [field.type] date field type: 'date' or 'datetime'
         * @param {string} [granularity] can either be : hour, day, week, month, quarter, year
         * @param {integer} [minGroups=4] optional minimal amount of desired groups
         * @param {boolean} [forceRecompute=false] optional whether the fill_temporal period should be
         *                                         reinstancied
         * @returns {FillTemporalPeriod}
         */
        const getFillTemporalPeriod = ({ modelName, field, granularity, minGroups = 4, forceRecompute = false }) => {
            if (!(modelName in _fillTemporalPeriods)) {
                _fillTemporalPeriods[modelName] = {};
            }
            if (!(field.name in _fillTemporalPeriods[modelName])) {
                _fillTemporalPeriods[modelName][field.name] = {};
            }
            if (!(granularity in _fillTemporalPeriods[modelName][field.name]) || forceRecompute) {
                _fillTemporalPeriods[modelName][field.name][granularity] = new FillTemporalPeriod(
                    modelName,
                    field,
                    granularity,
                    minGroups
                );
            } else if (_fillTemporalPeriods[modelName][field.name][granularity].minGroups != minGroups) {
                _fillTemporalPeriods[modelName][field.name][granularity].setMinGroups(minGroups);
            }
            return _fillTemporalPeriods[modelName][field.name][granularity];
        };
        return { getFillTemporalPeriod };
    },
};

registry.category("services").add("fillTemporalService", fillTemporalService);

```

## File: static\src\views\forecast_search_model.js

```javascript
/** @odoo-module **/

import { Domain } from "@web/core/domain";
import { makeContext } from "@web/core/context";
import { SearchModel } from "@web/search/search_model";
import {
    serializeDate,
    serializeDateTime,
} from "@web/core/l10n/dates";

/**
 * This is the conversion of ForecastModelExtension. See there for more
 * explanations of what is done here.
 */

export class ForecastSearchModel extends SearchModel {
    /**
     * @override
     */
    exportState() {
        const state = super.exportState();
        state.forecast = {
            forecastStart: this.forecastStart,
        };
        return state;
    }

    /**
     * @override
     */
    _getSearchItemDomain(activeItem) {
        let domain = super._getSearchItemDomain(activeItem);
        const { searchItemId } = activeItem;
        const searchItem = this.searchItems[searchItemId];
        const context = makeContext([searchItem.context || {}]);
        if (context.forecast_filter) {
            const forecastField = this.globalContext.forecast_field;
            const forecastStart = this._getForecastStart(forecastField);
            const forecastDomain = [
                "|",
                [forecastField, "=", false],
                [forecastField, ">=", forecastStart],
            ];
            domain = Domain.and([domain, forecastDomain]);
        }
        return domain;
    }

    /**
     * @protected
     * @param {string} forecastField
     * @returns {string}
     */
    _getForecastStart(forecastField) {
        if (!this.forecastStart) {
            const { type } = this.searchViewFields[forecastField];
            const groupBy = this.groupBy;
            const firstForecastGroupBy = groupBy.find((gb) => gb.includes(forecastField));
            let granularity = "month";
            if (firstForecastGroupBy) {
                granularity = firstForecastGroupBy.split(":")[1] || "month";
            } else if (groupBy.length) {
                granularity = "day";
            }
            const startDateTime = luxon.DateTime.now().startOf(granularity);
            this.forecastStart = type === "datetime" ? serializeDateTime(startDateTime) : serializeDate(startDateTime);
        }
        return this.forecastStart;
    }

    /**
     * @override
     */
    _importState(state) {
        super._importState(...arguments);
        if (state.forecast) {
            this.forecastStart = state.forecast.forecastStart;
        }
    }

    /**
     * @override
     */
    _reset() {
        super._reset();
        this.forecastStart = null;
    }
}

```

## File: static\src\views\crm_form\crm_form.js

```javascript
/** @odoo-module **/

import { checkRainbowmanMessage } from "@crm/views/check_rainbowman_message";
import { registry } from "@web/core/registry";
import { formView } from "@web/views/form/form_view";

class CrmFormRecord extends formView.Model.Record {
     /**
     * override of record _save mechanism intended to affect the main form record
     * We check if the stage_id field was altered and if we need to display a rainbowman
     * message.
     *
     * This method will also simulate a real "force_save" on the email and phone
     * when needed. The "force_save" attribute only works on readonly field. For our
     * use case, we need to write the email and the phone even if the user didn't
     * change them, to synchronize those values with the partner (so the email / phone
     * inverse method can be called).
     *
     * We base this synchronization on the value of "partner_phone_update"
     * and "partner_email_update", which are computed fields that hold a value
     * whenever we need to synch.
     *
     * @override
     */
    async _save() {
        if (this.resModel !== "crm.lead") {
            return super._save(...arguments);
        }
        let changeStage = false;
        const needsSynchronizationEmail =
            this._changes.partner_email_update === undefined
                ? this._values.partner_email_update // original value
                : this._changes.partner_email_update; // new value

        const needsSynchronizationPhone =
            this._changes.partner_phone_update === undefined
                ? this._values.partner_phone_update // original value
                : this._changes.partner_phone_update; // new value

        if (needsSynchronizationEmail && this._changes.email_from === undefined && this._values.email_from) {
            this._changes.email_from = this._values.email_from;
        }
        if (needsSynchronizationPhone && this._changes.phone === undefined && this._values.phone) {
            this._changes.phone = this._values.phone;
        }

        if ("stage_id" in this._changes) {
            changeStage = this._values.stage_id !== this.data.stage_id;
        }

        const res = await super._save(...arguments);
        if (changeStage) {
            await checkRainbowmanMessage(this.model.orm, this.model.effect, this.resId);
        }
        return res;
    }
}

class CrmFormModel extends formView.Model {
    static Record = CrmFormRecord;
    static services = [...formView.Model.services, "effect"];

    setup(params, services) {
        super.setup(...arguments);
        this.effect = services.effect;
    }
}

registry.category("views").add("crm_form", {
    ...formView,
    Model: CrmFormModel,
});

```

## File: static\src\views\crm_kanban\crm_column_progress.js

```javascript
/** @odoo-module */

import { onWillStart } from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";
import { ColumnProgress } from "@web/views/view_components/column_progress";
import { session } from "@web/session";
import { getCurrency } from "@web/core/currency";

export class CrmColumnProgress extends ColumnProgress {
    static props = {
        ...ColumnProgress.props,
        progressBarState: { type: Object },
    };
    static template = "crm.ColumnProgress";
    setup() {
        super.setup();
        this.user = useService("user");
        this.showRecurringRevenue = false;

        onWillStart(async () => {
            if (this.props.progressBarState.progressAttributes.recurring_revenue_sum_field) {
                this.showRecurringRevenue = await this.user.hasGroup("crm.group_use_recurring_revenues");
            }
        });
    }

    getRecurringRevenueGroupAggregate(group) {
        const rrField = this.props.progressBarState.progressAttributes.recurring_revenue_sum_field;
        const aggregatedValue = this.props.progressBarState.getAggregateValue(group, rrField);
        let currency = false;
        if (aggregatedValue.value && rrField.currency_field) {
            currency = getCurrency(session.company_currency_id);
        }
        return { ...aggregatedValue, currency };
    }
}

```

## File: static\src\views\crm_kanban\crm_column_progress.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="crm.ColumnProgress" t-inherit="web.ColumnProgress" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('o_column_progress')]" position="attributes">
            <attribute name="class" remove="w-75" add="w-50" separator=" "/>
        </xpath>
        <AnimatedNumber position="after">
            <t t-if="showRecurringRevenue">
                <t t-set="rrmAggregate" t-value="getRecurringRevenueGroupAggregate(props.group)"/>
                <AnimatedNumber
                    value="rrmAggregate.value"
                    title="rrmAggregate.title"
                    duration="1000"
                    currency="props.aggregate.currency"
                    animationClass="'o_animated_grow_huge'"
                >
                    <t t-set-slot="prefix">
                        <strong>+</strong>
                    </t>
                </AnimatedNumber>
            </t>
        </AnimatedNumber>
    </t>
</templates>

```

## File: static\src\views\crm_kanban\crm_kanban_arch_parser.js

```javascript
/** @odoo-module **/

import { KanbanArchParser } from "@web/views/kanban/kanban_arch_parser";
import { extractAttributes } from "@web/core/utils/xml";

export class CrmKanbanArchParser extends KanbanArchParser {
    /**
     * @override
     */
    parseProgressBar(progressBar, fields) {
        const result = super.parseProgressBar(...arguments);
        const attrs = extractAttributes(progressBar, ["recurring_revenue_sum_field"]);
        result.recurring_revenue_sum_field = fields[attrs.recurring_revenue_sum_field] || false;
        return result;
    }
}

```

## File: static\src\views\crm_kanban\crm_kanban_model.js

```javascript
/** @odoo-module **/

import { checkRainbowmanMessage } from "@crm/views/check_rainbowman_message";
import { RelationalModel } from "@web/model/relational_model/relational_model";

export class CrmKanbanModel extends RelationalModel {
    setup(params, { effect }) {
        super.setup(...arguments);
        this.effect = effect;
    }
}

export class CrmKanbanDynamicGroupList extends RelationalModel.DynamicGroupList {
    /**
     * @override
     *
     * If the kanban view is grouped by stage_id check if the lead is won and display
     * a rainbowman message if that's the case.
     */
    async moveRecord(dataRecordId, dataGroupId, refId, targetGroupId) {
        await super.moveRecord(...arguments);
        const sourceGroup = this.groups.find((g) => g.id === dataGroupId);
        const targetGroup = this.groups.find((g) => g.id === targetGroupId);
        if (
            dataGroupId !== targetGroupId &&
            sourceGroup &&
            targetGroup &&
            sourceGroup.groupByField.name === "stage_id"
        ) {
            const record = targetGroup.list.records.find((r) => r.id === dataRecordId);
            await checkRainbowmanMessage(this.model.orm, this.model.effect, record.resId);
        }
    }
}

CrmKanbanModel.DynamicGroupList = CrmKanbanDynamicGroupList;
CrmKanbanModel.services = [...RelationalModel.services, "effect"];

```

## File: static\src\views\crm_kanban\crm_kanban_renderer.js

```javascript
/** @odoo-module **/

import { CrmColumnProgress } from "./crm_column_progress";
import { KanbanRenderer } from "@web/views/kanban/kanban_renderer";
import { KanbanHeader } from "@web/views/kanban/kanban_header";

class CrmKanbanHeader extends KanbanHeader {
    static template = "crm.CrmKanbanHeader";
    static components = {
        ...KanbanHeader.components,
        ColumnProgress: CrmColumnProgress,
    };
}

export class CrmKanbanRenderer extends KanbanRenderer {}
CrmKanbanRenderer.components = {
    ...KanbanRenderer.components,
    KanbanHeader: CrmKanbanHeader,
};

```

## File: static\src\views\crm_kanban\crm_kanban_renderer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="crm.CrmKanbanHeader" t-inherit="web.KanbanHeader" t-inherit-mode="primary">
        <xpath expr="//ColumnProgress" position="attributes">
            <attribute name="progressBarState">props.progressBarState</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\crm_kanban\crm_kanban_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { kanbanView } from "@web/views/kanban/kanban_view";
import { CrmKanbanModel } from "@crm/views/crm_kanban/crm_kanban_model";
import { CrmKanbanArchParser } from "@crm/views/crm_kanban/crm_kanban_arch_parser";
import { CrmKanbanRenderer } from "@crm/views/crm_kanban/crm_kanban_renderer";

export const crmKanbanView = {
    ...kanbanView,
    ArchParser: CrmKanbanArchParser,
    // Makes it easier to patch
    Controller: class extends kanbanView.Controller {
        get progressBarAggregateFields() {
            const res = super.progressBarAggregateFields;
            const progressAttributes = this.props.archInfo.progressAttributes;
            if (progressAttributes && progressAttributes.recurring_revenue_sum_field) {
                res.push(progressAttributes.recurring_revenue_sum_field);
            }
            return res;
        }
    },
    Model: CrmKanbanModel,
    Renderer: CrmKanbanRenderer,
};

registry.category("views").add("crm_kanban", crmKanbanView);

```

## File: static\src\views\forecast_graph\forecast_graph_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { graphView } from "@web/views/graph/graph_view";
import { ForecastSearchModel } from "@crm/views/forecast_search_model";

export const forecastGraphView = {
    ...graphView,
    SearchModel: ForecastSearchModel,
};

registry.category("views").add("forecast_graph", forecastGraphView);

```

## File: static\src\views\forecast_kanban\forecast_kanban_column_quick_create.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { INTERVAL_OPTIONS } from "@web/search/utils/dates";
import { KanbanColumnQuickCreate } from "@web/views/kanban/kanban_column_quick_create";

export class ForecastKanbanColumnQuickCreate extends KanbanColumnQuickCreate {
    /**
     * @override
     */
    get relatedFieldName() {
        const { granularity = "month" } = this.props.groupByField;
        const { description } = INTERVAL_OPTIONS[granularity];
        return _t("Add next %s", description.toLocaleLowerCase());
    }
    /**
     * @override
     *
     * Create column directly upon "unfolding" quick create.
     */
    unfold() {
        this.props.onValidate();
    }
}

```

## File: static\src\views\forecast_kanban\forecast_kanban_controller.js

```javascript
/** @odoo-module **/

import { crmKanbanView } from "@crm/views/crm_kanban/crm_kanban_view";

export class ForecastKanbanController extends crmKanbanView.Controller {
    isQuickCreateField(field) {
        return super.isQuickCreateField(...arguments) || (field && field.name === "date_deadline");
    }
}

```

## File: static\src\views\forecast_kanban\forecast_kanban_model.js

```javascript
/** @odoo-module **/

import { CrmKanbanModel } from "@crm/views/crm_kanban/crm_kanban_model";
import { deserializeDateTime } from "@web/core/l10n/dates";

export class ForecastKanbanModel extends CrmKanbanModel {
    setup(params, { fillTemporalService }) {
        super.setup(...arguments);
        this.fillTemporalService = fillTemporalService;
        this.forceNextRecompute = !params.state?.groups;
        this.originalDomain = null;
        this.fillTemporalDomain = null;
    }

    async _webReadGroup(config, firstGroupByName, orderBy) {
        if (this.isForecastGroupBy(config)) {
            config.context = this.fillTemporalPeriod(config).getContext({
                context: config.context,
            });
            // Domain leaves added by the fillTemporalPeriod should be replaced
            // between 2 _webReadGroup calls, not added on top of each other.
            // Keep track of the modified domain, and if encountered in the
            // future, modify the original domain instead. It is not robust
            // against external modification of `config.domain`, but currently
            // there are only replacements except this case.
            if (!this.originalDomain || this.fillTemporalDomain !== config.domain) {
                this.originalDomain = config.domain || [];
            }
            this.fillTemporalDomain = this.fillTemporalPeriod(config).getDomain({
                domain: this.originalDomain,
                forceStartBound: false,
            });
            config.domain = this.fillTemporalDomain;
        }
        return super._webReadGroup(...arguments);
    }

    async _loadGroupedList(config) {
        const res = await super._loadGroupedList(...arguments);
        if (this.isForecastGroupBy(config)) {
            const lastGroup = res.groups.filter((grp) => grp.value).slice(-1)[0];
            if (lastGroup) {
                this.fillTemporalPeriod(config).setEnd(deserializeDateTime(lastGroup.range.to));
            }
        }
        return res;
    }

    /**
     * @returns {Boolean} true if the view is grouped by the forecast_field
     */
    isForecastGroupBy(config) {
        const forecastField = config.context.forecast_field;
        const name = config.groupBy[0].split(":")[0];
        return forecastField && forecastField === name;
    }

    /**
     * return {FillTemporalPeriod} current fillTemporalPeriod according to group by state
     */
    fillTemporalPeriod(config) {
        const [groupByFieldName, granularity] = config.groupBy[0].split(":");
        const groupByField = config.fields[groupByFieldName];
        const minGroups = (config.context.fill_temporal && config.context.fill_temporal.min_groups) || undefined;
        const { name, type } = groupByField;
        const forceRecompute = this.forceNextRecompute;
        this.forceNextRecompute = false;
        return this.fillTemporalService.getFillTemporalPeriod({
            modelName: config.resModel,
            field: {
                name,
                type,
            },
            granularity: granularity || "month",
            minGroups,
            forceRecompute,
        });
    }
}

ForecastKanbanModel.services = [...CrmKanbanModel.services, "fillTemporalService"];

```

## File: static\src\views\forecast_kanban\forecast_kanban_renderer.js

```javascript
/** @odoo-module **/

import { CrmKanbanRenderer } from "@crm/views/crm_kanban/crm_kanban_renderer";
import { useService } from "@web/core/utils/hooks";
import { ForecastKanbanColumnQuickCreate } from "@crm/views/forecast_kanban/forecast_kanban_column_quick_create";

export class ForecastKanbanRenderer extends CrmKanbanRenderer {
    setup() {
        super.setup(...arguments);
        this.fillTemporalService = useService("fillTemporalService");
    }
    /**
     * @override
     *
     * Allow creating groups when grouping by forecast_field.
     */
    canCreateGroup() {
        return super.canCreateGroup(...arguments) || this.isGroupedByForecastField();
    }

    isGroupedByForecastField() {
        return (
            this.props.list.context.forecast_field &&
            this.props.list.groupByField.name === this.props.list.context.forecast_field
        );
    }

    isMovableField(field) {
        return super.isMovableField(...arguments) || field.name === "date_deadline";
    }

    async addForecastColumn() {
        const { name, type, granularity } = this.props.list.groupByField;
        this.fillTemporalService
            .getFillTemporalPeriod({
                modelName: this.props.list.resModel,
                field: {
                    name,
                    type,
                },
                granularity: granularity || "month",
            })
            .expand();
        await this.props.list.load();
    }
}

ForecastKanbanRenderer.template = "crm.ForecastKanbanRenderer";
ForecastKanbanRenderer.components = {
    ...CrmKanbanRenderer.components,
    ForecastKanbanColumnQuickCreate,
};

```

## File: static\src\views\forecast_kanban\forecast_kanban_renderer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="crm.ForecastKanbanRenderer" t-inherit="web.KanbanRenderer" t-inherit-mode="primary">
        <KanbanColumnQuickCreate position="replace">
            <t t-if="isGroupedByForecastField()">
                <ForecastKanbanColumnQuickCreate
                    folded="true"
                    onFoldChange="() => {}"
                    onValidate.bind="addForecastColumn"
                    exampleData="exampleData"
                    groupByField="props.list.groupByField"
                />
            </t>
            <t t-else="">$0</t>
        </KanbanColumnQuickCreate>
    </t>
</templates>

```

## File: static\src\views\forecast_kanban\forecast_kanban_view.js

```javascript
/** @odoo-module **/

import { ForecastKanbanController } from "@crm/views/forecast_kanban/forecast_kanban_controller";
import { CrmKanbanArchParser } from "@crm/views/crm_kanban/crm_kanban_arch_parser";
import { ForecastKanbanModel } from "@crm/views/forecast_kanban/forecast_kanban_model";
import { ForecastKanbanRenderer } from "@crm/views/forecast_kanban/forecast_kanban_renderer";
import { ForecastSearchModel } from "@crm/views/forecast_search_model";
import { registry } from "@web/core/registry";
import { kanbanView } from "@web/views/kanban/kanban_view";

export const forecastKanbanView = {
    ...kanbanView,
    ArchParser: CrmKanbanArchParser,
    Model: ForecastKanbanModel,
    Controller: ForecastKanbanController,
    Renderer: ForecastKanbanRenderer,
    SearchModel: ForecastSearchModel,
};

registry.category("views").add("forecast_kanban", forecastKanbanView);

```

## File: static\src\views\forecast_list\forecast_list_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { listView } from "@web/views/list/list_view";
import { ForecastSearchModel } from "@crm/views/forecast_search_model";

export const forecastListView = {
    ...listView,
    SearchModel: ForecastSearchModel,
};

registry.category("views").add("forecast_list", forecastListView);

```

## File: static\src\views\forecast_pivot\forecast_pivot_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { pivotView } from "@web/views/pivot/pivot_view";
import { ForecastSearchModel } from "@crm/views/forecast_search_model";

export const forecastPivotView = {
    ...pivotView,
    SearchModel: ForecastSearchModel,
};

registry.category("views").add("forecast_pivot", forecastPivotView);

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="CRM assets backend" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/crm/static/src/js/crm_form.js"/>
            <script type="text/javascript" src="/crm/static/src/js/crm_kanban.js"/>
            <script type="text/javascript" src="/crm/static/src/js/systray_activity_menu.js"/>
            <script type="text/javascript" src="/crm/static/src/js/tours/crm.js"></script>
        </xpath>
    </template>
    <template id="assets_tests" name="CRM Assets Tests" inherit_id="web.assets_tests">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/crm/static/tests/tours/crm_rainbowman.js"></script>
            <script type="text/javascript" src="/crm/static/tests/tours/crm_email_and_phone_propagation.js"></script>
        </xpath>
    </template>
    <template id="qunit_suite" name="crm tests" inherit_id="web.qunit_suite_tests">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/crm/static/tests/mock_server.js"></script>
            <script type="text/javascript" src="/crm/static/tests/crm_rainbowman_tests.js"></script>
        </xpath>
    </template>         
</odoo>

```

## File: views\calendar_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="view_crm_meeting_search" model="ir.ui.view">
        <field name="name">calendar.event.form.inherit</field>
        <field name="model">calendar.event</field>
        <field name="inherit_id" ref="calendar.view_calendar_event_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='description']" position="after">
                <field name="opportunity_id"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\crm_helper_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="crm_action_helper" name="crm action helper">
        <t t-if="team.alias_email">
            <p class="o_view_nocontent_smiling_face">
                Create an opportunity to start playing with your pipeline.
            </p><p>Use the <i>New</i> button, or send an email to
            <a t-attf-href="mailto:#{team.alias_email}"><t t-esc="team.alias_email"/></a>
            to test the email gateway.</p>
        </t>
        <t t-else="">
            <p class='o_view_nocontent_smiling_face'>Create an opportunity to start playing with your pipeline.</p>
            <p>Use the New button, or configure an email alias to test the email gateway.</p>
        </t>
    </template>
</odoo>

```

## File: views\crm_lead_views.xml

```xml
<?xml version="1.0"?>
<odoo>
        <record id="crm_lead_view_form" model="ir.ui.view">
            <field name="name">crm.lead.form</field>
            <field name="model">crm.lead</field>
            <field name="arch" type="xml">
                <form class="o_lead_opportunity_form" js_class="crm_form">
                    <header>
                        <button name="action_set_won_rainbowman" string="Won"
                            type="object" class="oe_highlight" data-hotkey="w" title="Mark as won"
                            invisible="not active or probability == 100 or type == 'lead'"/>
                        <button name="%(crm.crm_lead_lost_action)d" string="Lost" data-hotkey="l" title="Mark as lost"
                            type="action" invisible="type == 'lead' or not active and probability &lt; 100"/>
                        <button name="%(crm.action_crm_lead2opportunity_partner)d" string="Convert to Opportunity" type="action" help="Convert to Opportunity"
                            class="oe_highlight" invisible="type == 'opportunity' or not active" data-hotkey="v"/>
                        <button name="toggle_active" string="Restore" type="object" data-hotkey="x"
                            invisible="probability &gt; 0 or active"/>
                        <button name="%(crm.crm_lead_lost_action)d" string="Lost" type="action"  data-hotkey="l" title="Mark as lost"
                            invisible="type == 'opportunity' or probability == 0 and not active"/>
                        <field name="stage_id" widget="statusbar_duration"
                            options="{'clickable': '1', 'fold_field': 'fold'}"
                            domain="['|', ('team_id', '=', team_id), ('team_id', '=', False)]"
                            invisible="not active or type == 'lead'"/>
                    </header>
                    <sheet>
                        <field name="active" invisible="1"/>
                        <field name="company_id" invisible="1"/>
                        <div class="oe_button_box" name="button_box">
                            <button name="action_schedule_meeting" type="object"
                                class="oe_stat_button" icon="fa-calendar"
                                context="{'partner_id': partner_id}"
                                invisible="not id or type == 'lead'">
                                <div class="o_stat_info">
                                    <span class="o_stat_text"><field name="meeting_display_label"/></span>
                                    <field name="meeting_display_date" class="o_stat_value" invisible="not meeting_display_date"/>
                                </div>
                            </button>
                            <button name="action_show_potential_duplicates" type="object"
                                class="oe_stat_button" icon="fa-star"
                                invisible="duplicate_lead_count &lt; 1">
                                <div class="o_stat_info">
                                    <field name="duplicate_lead_count" class="o_stat_value"/>
                                    <span class="o_stat_text" invisible="duplicate_lead_count &lt; 2">Similar Leads</span>
                                    <span class="o_stat_text" invisible="duplicate_lead_count &gt; 1">Similar Lead</span>
                                </div>
                            </button>
                        </div>
                        <widget name="web_ribbon" title="Lost" bg_color="text-bg-danger" invisible="probability &gt; 0 or active"/>
                        <widget name="web_ribbon" title="Won" invisible="probability &lt; 100" />
                        <div class="oe_title">
                            <h1><field class="text-break" options="{'line_breaks': False}" widget="text" name="name" placeholder="e.g. Product Pricing"/></h1>
                            <h2 class="row g-0 pb-3 pb-sm-4">
                                <div class="col-auto pb-2 pb-md-0 w-100 w-sm-auto" invisible="type == 'lead'">
                                    <label for="expected_revenue" class="oe_edit_only"/>
                                    <div class="d-flex flex-wrap align-items-baseline">
                                        <field name="company_currency" invisible="1"/>
                                        <field name="expected_revenue" class="o_input_13ch" widget='monetary' options="{'currency_field': 'company_currency'}"/>
                                        <span class="oe_grey p-2" groups="crm.group_use_recurring_revenues"> + </span>
                                        <span class="oe_grey p-2" groups="!crm.group_use_recurring_revenues"> at </span>
                                        <field name="recurring_revenue" class="o_input_10ch mx-auto mx-sm-0" widget="monetary"
                                            options="{'currency_field': 'company_currency'}" groups="crm.group_use_recurring_revenues"/>
                                        <div class="d-flex align-items-baseline" groups="crm.group_use_recurring_revenues">
                                            <field name="recurring_plan" class="oe_inline o_input_13ch" placeholder='e.g. "Monthly"'
                                                required="recurring_revenue != 0" options="{'no_create': True, 'no_open': True}"/>
                                            <span class="oe_grey p-2 text-nowrap d-none d-sm-block"> at </span>
                                        </div>
                                    </div>
                                </div>
                                <div class="col-auto">
                                    <label for="probability" class="d-inline-block"/>
                                    <button class="d-inline-block px-2 py-0 btn btn-link" name="action_set_automated_probability" type="object"
                                            invisible="is_automated_probability">
                                        <i class="fa fa-gear" role="img" title="Switch to automatic probability" aria-label="Switch to automatic probability"></i>
                                    </button>
                                    <small class="d-inline-block oe_grey h6 mb-0" invisible="is_automated_probability">
                                        <field class="mb-0" name="automated_probability" force_save="1"/> %
                                    </small>
                                    <div id="probability" class="d-flex align-items-baseline">
                                        <field name="is_automated_probability" invisible="1"/>
                                        <field name="probability" widget="float" class="oe_inline o_input_6ch"/>
                                        <span class="oe_grey p-2"> %</span>
                                    </div>
                                </div>
                            </h2>
                        </div>
                        <group>
                            <group name="lead_partner" invisible="type == 'opportunity'">
                                <!-- Preload all the partner's information -->
                                <field name="is_partner_visible" invisible='1'/>
                                <field name="partner_id" widget="res_partner_many2one"
                                    context="{
                                        'default_name': contact_name,
                                        'default_title': title,
                                        'default_street': street,
                                        'default_street2': street2,
                                        'default_city': city,
                                        'default_state_id': state_id,
                                        'default_zip': zip,
                                        'default_country_id': country_id,
                                        'default_function': function,
                                        'default_phone': phone,
                                        'default_mobile': mobile,
                                        'default_email': email_from,
                                        'default_user_id': user_id,
                                        'default_team_id': team_id,
                                        'default_website': website,
                                        'default_lang': lang_code,
                                        'show_vat': True
                                    }" invisible="not is_partner_visible"/>
                                <field name="partner_name"/>
                                <label for="street" string="Address"/>
                                <div class="o_address_format">
                                    <field name="street" placeholder="Street..." class="o_address_street"/>
                                    <field name="street2" placeholder="Street 2..." class="o_address_street"/>
                                    <field name="city" placeholder="City" class="o_address_city"/>
                                    <field name="state_id" class="o_address_state" placeholder="State" options='{"no_open": True}'/>
                                    <field name="zip" placeholder="ZIP" class="o_address_zip"/>
                                    <field name="country_id" placeholder="Country" class="o_address_country" options='{"no_open": True, "no_create": True}'/>
                                </div>
                                <field name="website" widget="url" placeholder="e.g. https://www.odoo.com"/>
                                <field name="lang_active_count" invisible="1"/>
                                <field name="lang_code" invisible="1"/>
                                <field name="lang_id" invisible="lang_active_count &lt;= 1"
                                    options="{'no_quick_create': True, 'no_create_edit': True, 'no_open': True}"/>
                            </group>

                            <group name="opportunity_partner" invisible="type == 'lead'">
                                <field name="partner_id"
                                    widget="res_partner_many2one"
                                    string="Customer"
                                    context="{'res_partner_search_mode': type == 'opportunity' and 'customer' or False,
                                        'default_name': contact_name or partner_name,
                                        'default_street': street,
                                        'default_is_company': type == 'opportunity' and contact_name == False,
                                        'default_company_name': type == 'opportunity' and partner_name,
                                        'default_street2': street2,
                                        'default_city': city,
                                        'default_title': title,
                                        'default_state_id': state_id,
                                        'default_zip': zip,
                                        'default_country_id': country_id,
                                        'default_function': function,
                                        'default_phone': phone,
                                        'default_mobile': mobile,
                                        'default_email': email_from,
                                        'default_user_id': user_id,
                                        'default_team_id': team_id,
                                        'default_website': website,
                                        'default_lang': lang_code,
                                        'show_vat': True,
                                    }"
                                />
                                <field name="is_blacklisted" invisible="1"/>
                                <field name="partner_is_blacklisted" invisible="1"/>
                                <field name="phone_blacklisted" invisible="1"/>
                                <field name="mobile_blacklisted" invisible="1"/>
                                <field name="email_state" invisible="1"/>
                                <field name="phone_state" invisible="1"/>
                                <field name="partner_email_update" invisible="1"/>
                                <field name="partner_phone_update" invisible="1"/>
                                <label for="email_from" class="oe_inline"/>
                                <div class="o_row o_row_readonly">
                                    <button name="mail_action_blacklist_remove" class="fa fa-ban text-danger"
                                        title="This email is blacklisted for mass mailings. Click to unblacklist."
                                        type="object" context="{'default_email': email_from}" groups="base.group_user"
                                        invisible="not is_blacklisted"/>
                                    <field name="email_from" string="Email" widget="email"/>
                                    <span class="fa fa-exclamation-triangle text-warning oe_edit_only"
                                        title="By saving this change, the customer email will also be updated."
                                        invisible="not partner_email_update"/>
                                </div>
                                <label for="phone" class="oe_inline"/>
                                <div class="o_row o_row_readonly">
                                    <button name="phone_action_blacklist_remove" class="fa fa-ban text-danger"
                                        title="This phone number is blacklisted for SMS Marketing. Click to unblacklist."
                                        type="object" context="{'default_phone': phone}" groups="base.group_user"
                                        invisible="not phone_blacklisted"/>
                                    <field name="phone" widget="phone"/>
                                    <span class="fa fa-exclamation-triangle text-warning oe_edit_only"
                                        title="By saving this change, the customer phone number will also be updated."
                                        invisible="not partner_phone_update"/>
                                </div>
                                <field name="lost_reason_id" invisible="active"/>
                                <field name="date_conversion" invisible="1"/>
                                <field name="user_company_ids" invisible="1"/>
                            </group>
                            <group name="lead_info" invisible="type == 'opportunity'">
                                <label for="contact_name"/>
                                <div class="o_row">
                                    <field name="contact_name"/>
                                    <field name="title" placeholder="Title" domain="[]" options='{"no_open": True}'/>
                                </div>
                                <field name="is_blacklisted" invisible="1"/>
                                <field name="phone_blacklisted" invisible="1"/>
                                <field name="email_state" invisible="1"/>
                                <field name="phone_state" invisible="1"/>
                                <field name="partner_email_update" invisible="1"/>
                                <field name="partner_phone_update" invisible="1"/>
                                <label for="email_from_group_lead_info" class="oe_inline"/>
                                <div class="o_row o_row_readonly">
                                    <button name="mail_action_blacklist_remove" class="fa fa-ban text-danger"
                                        title="This email is blacklisted for mass mailings. Click to unblacklist."
                                        type="object" context="{'default_email': email_from}" groups="base.group_user"
                                        invisible="not is_blacklisted"/>
                                    <field name="email_from" id="email_from_group_lead_info" string="Email" widget="email"/>
                                    <span class="fa fa-exclamation-triangle text-warning oe_edit_only"
                                        title="By saving this change, the customer email will also be updated."
                                        invisible="not partner_email_update"/>
                                </div>
                                <field name="email_cc" groups="base.group_no_one"/>
                                <field name="function"/>
                                <label for="phone_group_lead_info" class="oe_inline"/>
                                <div class="o_row o_row_readonly">
                                    <button name="phone_action_blacklist_remove" class="fa fa-ban text-danger"
                                        title="This phone number is blacklisted for SMS Marketing. Click to unblacklist."
                                        type="object" context="{'default_phone': phone}" groups="base.group_user"
                                        invisible="not phone_blacklisted"/>
                                    <field name="phone" id="phone_group_lead_info" widget="phone"/>
                                    <span class="fa fa-exclamation-triangle text-warning oe_edit_only"
                                        title="By saving this change, the customer phone number will also be updated."
                                        invisible="not partner_phone_update"/>
                                </div>
                                <label for="mobile" class="oe_inline"/>
                                <div class="o_row o_row_readonly">
                                    <button name="phone_action_blacklist_remove" class="fa fa-ban text-danger"
                                        title="This phone number is blacklisted for SMS Marketing. Click to unblacklist."
                                        type="object" context="{'default_phone': mobile}" groups="base.group_user"
                                        invisible="not mobile_blacklisted"/>
                                    <field name="mobile" widget="phone" string="Mobile"/>
                                </div>
                            </group>
                            <field name="type" invisible="1"/>
                            <group invisible="type == 'lead'">
                                <field name="user_id"
                                    context="{'default_sales_team_id': team_id}" widget="many2one_avatar_user"/>
                                <label for="date_deadline">Expected Closing</label>
                                <div class="o_lead_opportunity_form_inline_fields">
                                    <field name="date_deadline" nolabel="1" class="oe_inline"/>
                                    <field name="priority" widget="priority" nolabel="1" class="oe_inline align-top"/>
                                </div>
                                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                            </group>
                            <group invisible="type == 'opportunity'">
                                <field name="user_id"
                                    context="{'default_sales_team_id': team_id}" widget="many2one_avatar_user"/>
                                <field name="team_id" options="{'no_open': True, 'no_create': True}" context="{'kanban_view_ref': 'sales_team.crm_team_view_kanban'}"/>
                            </group>
                            <group name="lead_priority" invisible="type == 'opportunity'">
                                <field name="priority" widget="priority"/>
                                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                            </group>
                        </group>
                        <div class="d-flex">
                            <field name="lead_properties" nolabel="1" columns="2"/>
                        </div>
                        <notebook>
                            <page string="Internal Notes" name="internal_notes">
                                <field name="description" placeholder="Add a description..." options="{'collaborative': true}" />
                            </page>
                            <page name="extra" string="Extra Info" invisible="type == 'opportunity'">
                                <group>
                                    <group string="Email" groups="base.group_no_one">
                                        <field name="message_bounce" readonly="1"/>
                                    </group>
                                    <group string="Marketing" name="categorization">
                                        <field name="company_id"
                                            groups="base.group_multi_company"
                                            options="{'no_create': True}"/>
                                        <field name="campaign_id" options="{'create_name_field': 'title'}"/>
                                        <field name="medium_id"/>
                                        <field name="source_id"/>
                                        <field name="referred"/>
                                    </group>
                                    <group string="Analysis">
                                        <field name="date_open"/>
                                        <field name="date_closed"/>
                                    </group>
                                </group>
                            </page>
                            <page name="lead" string="Extra Information" invisible="type == 'lead'">
                                <group>
                                    <group string="Contact Information">
                                        <field name="partner_name"/>
                                        <label for="street_page_lead" string="Address"/>
                                        <div class="o_address_format">
                                            <field name="street" id="street_page_lead" placeholder="Street..." class="o_address_street"/>
                                            <field name="street2" placeholder="Street 2..." class="o_address_street"/>
                                            <field name="city" placeholder="City" class="o_address_city"/>
                                            <field name="state_id" class="o_address_state" placeholder="State" options='{"no_open": True}'/>
                                            <field name="zip" placeholder="ZIP" class="o_address_zip"/>
                                            <field name="country_id" placeholder="Country" class="o_address_country" options='{"no_open": True, "no_create": True}'/>
                                        </div>
                                        <field name="website" widget="url" placeholder="e.g. https://www.odoo.com"/>
                                        <field name="lang_active_count" invisible="1"/>
                                        <field name="lang_id" invisible="lang_active_count &lt;= 1"
                                            options="{'no_quick_create': True, 'no_create_edit': True, 'no_open': True}"/>
                                    </group>
                                    <group class="mt48">
                                        <label for="contact_name_page_lead"/>
                                        <div class="o_row">
                                            <field name="contact_name" id="contact_name_page_lead"/>
                                            <field name="title" placeholder="Title" domain="[]" options='{"no_open": True}'/>
                                        </div>
                                        <field name="function"/>
                                        <label for="mobile_page_lead" class="oe_inline"/>
                                        <div class="o_row o_row_readonly">
                                            <button name="phone_action_blacklist_remove" class="fa fa-ban text-danger"
                                                title="This phone number is blacklisted for SMS Marketing. Click to unblacklist."
                                                type="object" context="{'default_phone': mobile}" groups="base.group_user"
                                                invisible="not mobile_blacklisted"/>
                                            <field name="mobile" id="mobile_page_lead" widget="phone"/>
                                        </div>
                                    </group>
                                    <group string="Marketing">
                                        <field name="campaign_id" options="{'create_name_field': 'title'}"/>
                                        <field name="medium_id" />
                                        <field name="source_id" />
                                        <field name="referred"/>
                                    </group>
                                    <group string="Tracking" name="Misc">
                                        <field name="company_id"
                                            groups="base.group_multi_company"
                                            options="{'no_create': True}"/>
                                        <field name="team_id" options="{'no_open': True, 'no_create': True}" context="{'kanban_view_ref': 'sales_team.crm_team_view_kanban'}"/>
                                        <field name="day_open" />
                                        <field name="day_close"/>
                                        <field name="type" invisible="1"/>
                                    </group>
                                </group>
                            </page>
                        </notebook>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids"/>
                        <field name="activity_ids"/>
                        <field name="message_ids" options="{'post_refresh': 'recipients'}"/>
                    </div>
                </form>
            </field>
        </record>

        <!--
            crm.lead (as Lead) views
        -->
        <record id="act_crm_opportunity_calendar_event_new" model="ir.actions.act_window">
            <field name="name">Meetings</field>
            <field name="res_model">calendar.event</field>
            <field name="view_mode">tree,form,calendar</field>
            <field name="context">{'default_duration': 4.0, 'default_opportunity_id': active_id}</field>
        </record>

        <record id="crm_case_tree_view_leads" model="ir.ui.view">
            <field name="name">crm.lead.tree.lead</field>
            <field name="model">crm.lead</field>
            <field name="priority">10</field>
            <field name="arch" type="xml">
                <tree string="Leads" sample="1" multi_edit="1">
                    <field name="company_id" column_invisible="True"/>
                    <field name="user_company_ids" column_invisible="True"/>
                    <field name="date_deadline" column_invisible="True"/>
                    <field name="create_date" optional="hide"/>
                    <field name="name" string="Lead" readonly="1"/>
                    <field name="contact_name" optional="hide"/>
                    <field name="partner_name" optional="hide"/>
                    <field name="email_from" optional="show"/>
                    <field name="phone" optional="hide" class="o_force_ltr"/>
                    <field name="company_id" groups="base.group_multi_company" optional="hide"/>
                    <field name="city" optional="show"/>
                    <field name="state_id" optional="hide"/>
                    <field name="country_id" optional="show" options="{'no_open': True, 'no_create': True}"/>
                    <field name="partner_id" column_invisible="True"/>
                    <!-- Explicit domain due to multi edit -> real company domain would be complicated -->
                    <field name="user_id" optional="show"  widget="many2one_avatar_user"
                        domain="[('share', '=', False)]"/>
                    <field name="team_id" optional="show"/>
                    <field name="active" column_invisible="True"/>
                    <field name="campaign_id" optional="hide"/>
                    <field name="referred" column_invisible="True"/>
                    <field name="medium_id" optional="hide"/>
                    <field name="source_id" optional="hide"/>
                    <field name="probability" string="Probability (%)" optional="hide"/>
                    <field name="message_needaction" column_invisible="True"/>
                    <field name="tag_ids" optional="hide" widget="many2many_tags" options="{'color_field': 'color'}"/>
                    <field name="priority" optional="hide"/>
                </tree>
            </field>
        </record>

        <record id="view_crm_lead_kanban" model="ir.ui.view">
            <field name="name">crm.lead.kanban</field>
            <field name="model">crm.lead</field>
            <field name="priority" eval="100"/>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile" archivable="false" js_class="crm_kanban" sample="1">
                    <field name="name"/>
                    <field name="contact_name"/>
                    <field name="priority"/>
                    <field name="tag_ids"/>
                    <field name="user_id"/>
                    <field name="activity_ids"/>
                    <field name="activity_state"/>
                    <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_content oe_kanban_global_click">
                                <div>
                                    <strong class="o_kanban_record_title"><span><field name="name"/></span></strong>
                                </div>
                                <div>
                                    <span class="o_kanban_record_subtitle"><field name="contact_name"/></span>
                                </div>
                                <div>
                                  <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                </div>
                                <div class="o_kanban_record_bottom">
                                    <div class="oe_kanban_bottom_left">
                                        <field name="priority" widget="priority"/>
                                        <div class="o_kanban_inline_block">
                                            <field name="activity_ids" widget="kanban_activity"/>
                                        </div>
                                    </div>
                                    <div class="oe_kanban_bottom_right">
                                        <field name="user_id" widget="many2one_avatar_user" domain="[('share', '=', False)]"/>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="crm_case_calendar_view_leads" model="ir.ui.view">
            <field name="name">crm.lead.calendar.lead</field>
            <field name="model">crm.lead</field>
            <field name="priority" eval="2"/>
            <field name="arch" type="xml">
                <calendar string="Leads Generation" create="0" mode="month" date_start="activity_date_deadline" color="user_id" hide_time="true" event_limit="5">
                    <field name="expected_revenue"/>
                    <field name="partner_id" avatar_field="avatar_128"/>
                    <field name="user_id" filters="1" invisible="1"/>
                    <field name="team_id" invisible="1"/>
                    <field name="lead_properties"/>
                </calendar>
            </field>
        </record>

        <record id="quick_create_opportunity_form" model="ir.ui.view">
            <field name="name">crm.lead.form.quick_create</field>
            <field name="model">crm.lead</field>
            <field name="priority">1000</field>
            <field name="arch" type="xml">
                <form>
                    <group>
                        <field name="partner_id" widget="res_partner_many2one"
                            class="o_field_highlight"
                            string='Organization / Contact'
                            context="{
                            'res_partner_search_mode': type == 'opportunity' and 'customer' or False,
                            'default_name': contact_name or partner_name,
                            'default_is_company': type == 'opportunity' and contact_name == False,
                            'default_company_name': type == 'opportunity' and partner_name,
                            'default_type': 'contact',
                            'default_phone': phone,
                            'default_email': email_from,
                            'default_user_id': user_id,
                            'default_team_id': team_id,
                            'show_vat': True}"/>
                        <field name="name" placeholder="e.g. Product Pricing" />
                        <field name="email_from" string="Email" placeholder='e.g. "email@address.com"' />
                        <field name="phone" string="Phone" placeholder='e.g. "0123456789"' />
                        <label for="expected_revenue"/>
                        <div>
                            <div class="o_row">
                                <field name="expected_revenue" class="oe_inline me-5 o_field_highlight" widget="monetary" options="{'currency_field': 'company_currency'}"/>
                                <field name="priority" class="oe_inline" nolabel="1" widget="priority"/>
                            </div>
                            <div class="o_row" groups="crm.group_use_recurring_revenues">
                                <field name="recurring_revenue" class="oe_inline o_field_highlight" widget="monetary" options="{'currency_field': 'company_currency'}"/>
                                <field name="recurring_plan" class="oe_inline" placeholder='e.g. "Monthly"'
                                    required="recurring_revenue != 0" options="{'no_create': True, 'no_open': True}"/>
                            </div>
                        </div>
                        <field name="company_currency" invisible="1"/>
                        <field name="company_id" invisible="1"/>
                        <field name="user_id" invisible="1"/>
                        <field name="user_company_ids" invisible="1"/>
                        <field name="team_id" invisible="1"/>
                        <field name="type" invisible="1"/>
                        <field name="partner_name" invisible="1"/>
                        <field name="contact_name" invisible="1"/>
                        <field name="country_id" invisible="1"/>
                        <field name="state_id" invisible="1"/>
                        <field name="city" invisible="1"/>
                        <field name="street" invisible="1"/>
                        <field name="street2" invisible="1"/>
                        <field name="zip" invisible="1"/>
                        <field name="mobile" invisible="1"/>
                        <field name="website" invisible="1"/>
                        <field name="function" invisible="1"/>
                        <field name="title" invisible="1"/>
                        <field name="activity_ids" invisible="1"/>
                    </group>
                </form>
            </field>
        </record>

        <record id="crm_lead_view_activity" model="ir.ui.view">
            <field name="name">crm.lead.view.activity</field>
            <field name="model">crm.lead</field>
            <field name="arch" type="xml">
                <activity string="Leads or Opportunities">
                    <field name="user_id"/>
                    <field name="company_currency"/>
                    <templates>
                        <div t-name="activity-box">
                            <field name="user_id" widget="many2one_avatar_user" domain="[('share', '=', False)]"/>
                            <div class="flex-grow-1">
                                <div class="d-flex justify-content-between">
                                    <field name="name" display="full" class="o_text_block o_text_bold"/>
                                    <div class="m-1"/>
                                    <field name="expected_revenue" widget='monetary' display="full" muted="1"/>
                                </div>
                                <div class="d-flex justify-content-between">
                                    <field name="partner_id" muted="1" display="full" class="o_text_block"/>
                                    <div class="m-1"/>
                                    <field name="stage_id" widget="badge"/>
                                </div>
                            </div>
                        </div>
                    </templates>
                </activity>
            </field>
        </record>

        <record id="crm_case_kanban_view_leads" model="ir.ui.view">
            <field name="name">crm.lead.kanban.lead</field>
            <field name="model">crm.lead</field>
            <field name="priority" eval="1"/>
            <field name="arch" type="xml">
                <kanban default_group_by="stage_id" class="o_kanban_small_column o_opportunity_kanban" on_create="quick_create" quick_create_view="crm.quick_create_opportunity_form"
                    archivable="false" sample="1" js_class="crm_kanban">
                    <field name="stage_id" options='{"group_by_tooltip": {"requirements": "Description"}}'/>
                    <field name="probability"/>
                    <field name="color"/>
                    <field name="priority"/>
                    <field name="expected_revenue"/>
                    <field name="kanban_state"/>
                    <field name="activity_date_deadline"/>
                    <field name="user_id"/>
                    <field name="partner_id"/>
                    <field name="activity_summary"/>
                    <field name="active"/>
                    <field name="company_currency"/>
                    <field name="activity_state" />
                    <field name="activity_ids" />
                    <field name="recurring_revenue_monthly"/>
                    <field name="team_id"/>
                    <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'
                        sum_field="expected_revenue" recurring_revenue_sum_field="recurring_revenue_monthly"
                        help="This bar allows to filter the opportunities based on scheduled activities."/>
                    <templates>
                        <t t-name="kanban-menu">
                            <t t-if="widget.editable"><a role="menuitem" type="edit" class="dropdown-item">Edit</a></t>
                            <t t-if="widget.deletable"><a role="menuitem" type="delete" class="dropdown-item">Delete</a></t>
                            <ul class="oe_kanban_colorpicker" data-field="color"/>
                        </t>
                        <t t-name="kanban-box">
                            <t t-set="lost_ribbon" t-value="!record.active.raw_value and record.probability and record.probability.raw_value == 0"/>
                            <div t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''} #{lost_ribbon ? 'oe_kanban_card_ribbon' : ''} oe_kanban_global_click oe_kanban_card d-flex flex-column">
                                <div class="ribbon ribbon-top-right"
                                    invisible="probability &gt; 0 or active">
                                    <span class="text-bg-danger">Lost</span>
                                </div>

                                <div class="oe_kanban_content flex-grow-1">
                                    <div class="oe_kanban_details">
                                        <strong class="o_kanban_record_title"><field name="name"/></strong>
                                    </div>
                                    <div class="o_kanban_record_subtitle">
                                        <t t-if="record.expected_revenue.raw_value">
                                            <field name="expected_revenue" widget="monetary" options="{'currency_field': 'company_currency'}"/>
                                            <span t-if="record.recurring_revenue and record.recurring_revenue.raw_value" groups="crm.group_use_recurring_revenues"> + </span>
                                        </t>
                                        <t t-if="record.recurring_revenue and record.recurring_revenue.raw_value">
                                            <field class="me-1" name="recurring_revenue" widget="monetary" options="{'currency_field': 'company_currency'}" groups="crm.group_use_recurring_revenues"/>
                                            <field name="recurring_plan" groups="crm.group_use_recurring_revenues"/>
                                        </t>
                                    </div>
                                    <div>
                                        <span class="o_text_overflow" t-if="record.partner_id.value" t-esc="record.partner_id.value"></span>
                                    </div>
                                    <div>
                                        <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                    </div>
                                    <div>
                                        <field name="lead_properties" widget="properties"/>
                                    </div>
                                </div>
                                <div class="oe_kanban_footer">
                                    <div class="o_kanban_record_bottom">
                                        <div class="oe_kanban_bottom_left">
                                            <field name="priority" widget="priority" groups="base.group_user"/>
                                            <field name="activity_ids" widget="kanban_activity"/>
                                        </div>
                                        <div class="oe_kanban_bottom_right">
                                            <field name="user_id" widget="many2one_avatar_user" domain="[('share', '=', False)]"/>
                                        </div>
                                    </div>
                                </div>
                                <div class="clearfix"/>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="crm_lead_view_kanban_forecast" model="ir.ui.view">
            <field name="name">crm.lead.view.kanban.forecast</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_case_kanban_view_leads"/>
            <field name="mode">primary</field>
            <field name="priority">32</field>
            <field name="arch" type="xml">
                <xpath expr="//kanban" position="attributes">
                    <attribute name="default_group_by">date_deadline</attribute>
                    <attribute name="js_class">forecast_kanban</attribute>
                </xpath>
                <xpath expr="//kanban" position="inside">
                    <field name="date_deadline"/>
                </xpath>
                <xpath expr="//field[@name='expected_revenue']" position="replace">
                    <field name="prorated_revenue"/>
                    <field name="recurring_revenue"/>
                </xpath>
                <xpath expr="//field[@name='recurring_revenue_monthly']" position="replace">
                    <field name="recurring_revenue_monthly_prorated"/>
                </xpath>
                <xpath expr="//progressbar" position="attributes">
                    <attribute name="sum_field">prorated_revenue</attribute>
                    <attribute name="recurring_revenue_sum_field">recurring_revenue_monthly_prorated</attribute>
                </xpath>
                <xpath expr="//t[@t-set='lost_ribbon']" position="after">
                    <t t-set="won_ribbon" t-value="record.active.raw_value and record.probability and record.probability.raw_value == 100"/>
                </xpath>
                <xpath expr="//div[contains(@t-attf-class, 'oe_kanban_card')]" position="attributes">
                    <attribute name="t-attf-class">
                        #{!selection_mode ? kanban_color(record.color.raw_value) : ''}
                        #{lost_ribbon || won_ribbon ? 'oe_kanban_card_ribbon' : ''}
                        oe_kanban_global_click oe_kanban_card d-flex flex-column
                    </attribute>
                </xpath>
                <xpath expr="//div[hasclass('ribbon')]" position="replace">
                    <div class="ribbon ribbon-top-right"
                        invisible="(probability &gt; 0 or active) and (probability &lt; 100 or not active)">
                        <span t-if="won_ribbon" class="text-bg-success">Won</span>
                        <span t-elif="lost_ribbon" class="text-bg-danger">Lost</span>
                    </div>
                </xpath>
                <xpath expr="//div[hasclass('o_kanban_record_subtitle')]" position="replace">
                    <div class="o_kanban_record_subtitle">
                        <t t-if="record.prorated_revenue.raw_value">
                            <field name="prorated_revenue" widget="monetary" options="{'currency_field': 'company_currency'}"/>
                            <span t-if="record.recurring_revenue and record.recurring_revenue.raw_value" groups="crm.group_use_recurring_revenues"> + </span>
                        </t>
                        <t t-if="record.recurring_revenue and record.recurring_revenue.raw_value">
                            <field class="me-1" name="recurring_revenue_prorated" widget="monetary" options="{'currency_field': 'company_currency'}" groups="crm.group_use_recurring_revenues"/>
                            <field name="recurring_plan" groups="crm.group_use_recurring_revenues"/>
                        </t>
                    </div>
                </xpath>
            </field>
        </record>

        <record id="view_crm_case_leads_filter" model="ir.ui.view">
            <field name="name">crm.lead.search.lead</field>
            <field name="model">crm.lead</field>
            <field name="arch" type="xml">
                <search string="Search Leads">
                    <field name="name" string="Lead" filter_domain="['|','|','|',('partner_name', 'ilike', self),('email_from', 'ilike', self), ('contact_name', 'ilike', self), ('name', 'ilike', self)]"/>
                    <field name="tag_ids" string="Tag" filter_domain="[('tag_ids', 'ilike', self)]"/>
                    <field name="user_id"/>
                    <field name="team_id"/>
                    <field name="country_id"/>
                    <field name="city"/>
                    <field name="phone_mobile_search"/>
                    <field name="lang_id"/>
                    <field name="create_date"/>
                    <field name="source_id"/>
                    <field name="medium_id"/>
                    <field name="campaign_id"/>
                    <field name="activity_state"/>
                    <field name="lead_properties"/>
                    <separator />
                    <filter string="My Leads"
                            name="assigned_to_me"
                            domain="[('user_id', '=', uid)]"
                            help="Leads that are assigned to me"/>
                    <filter string="Unassigned" name="unassigned_leads"
                            domain="[('user_id','=', False), ('type', '=', 'lead')]"
                            help="Leads that are not assigned"/>
                    <separator />
                    <filter string="Lost" name="lost"
                            domain="['&amp;', ('probability', '=', 0), ('active', '=', False)]"/>
                    <separator/>
                    <filter string="Creation Date" name="filter_creation_date" date="create_date" default_period="this_month"/>
                    <filter name="filter_date_closed" date="date_closed"/>
                    <separator/>
                    <separator/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                            domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                            help="Show all opportunities for which the next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                            domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                            domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Group By">
                        <filter string="Salesperson" name="salesperson" context="{'group_by':'user_id'}"/>
                        <filter string="Sales Team" name="saleschannel" context="{'group_by':'team_id'}"/>
                        <filter name="city" string="City" context="{'group_by': 'city'}"/>
                        <filter string="Country" name="country" context="{'group_by':'country_id'}" />
                        <filter string="Company" name="company" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                        <filter string="Campaign" name="compaign" domain="[]" context="{'group_by':'campaign_id'}"/>
                        <filter string="Medium" name="medium" domain="[]" context="{'group_by':'medium_id'}"/>
                        <filter string="Source" name="source" domain="[]" context="{'group_by':'source_id'}"/>
                        <separator orientation="vertical" />
                        <filter string="Creation Date" context="{'group_by':'create_date:month'}" name="month"/>
                        <filter string="Closed Date" name="date_closed" context="{'group_by':'date_closed'}"/>
                        <separator />
                        <filter string="Properties" name="group_by_lead_properties" context="{'group_by':'lead_properties'}"/>
                    </group>
                </search>
            </field>
        </record>

        <!--
            MASS MAILING
        -->
        <record id="action_lead_mail_compose" model="ir.actions.act_window">
            <field name="name">Send email</field>
            <field name="res_model">mail.compose.message</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
            <field name="context" eval="{
    'default_composition_mode': 'comment',
                }"/>
            <field name="binding_model_id" ref="model_crm_lead"/>
            <field name="binding_view_types">form</field>
        </record>

        <record id="action_lead_mass_mail" model="ir.actions.act_window">
            <field name="name">Send email</field>
            <field name="res_model">mail.compose.message</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
            <field name="context" eval="{
    'default_composition_mode': 'mass_mail',
                }"/>
            <field name="binding_model_id" ref="model_crm_lead"/>
            <field name="binding_view_types">list</field>
        </record>

        <!--
            crm.lead (as Opportunity) views
        -->

        <record id="crm_case_tree_view_oppor" model="ir.ui.view">
            <field name="name">crm.lead.tree.opportunity</field>
            <field name="model">crm.lead</field>
            <field name="priority">1</field>
            <field name="arch" type="xml">
                <tree string="Opportunities" sample="1" multi_edit="1">
                    <header>
                        <button name="%(crm.action_lead_mass_mail)d" type="action" string="Email" />
                    </header>
                    <field name="company_id" column_invisible="True"/>
                    <field name="user_company_ids" column_invisible="True"/>
                    <field name="date_deadline" column_invisible="True"/>
                    <field name="create_date" optional="hide"/>
                    <field name="name" string="Opportunity" readonly="1"/>
                    <field name="partner_id" optional="hide"/>
                    <field name="contact_name" optional="show"/>
                    <field name="email_from"/>
                    <field name="phone" optional="hide" class="o_force_ltr"/>
                    <field name="company_id" groups="base.group_multi_company" optional="hide"/>
                    <field name="city" optional="hide"/>
                    <field name="state_id" optional="hide"/>
                    <field name="country_id" optional="hide" options="{'no_open': True, 'no_create': True}"/>
                    <!-- Explicit domain due to multi edit -> real company domain would be complicated -->
                    <field name="user_id" widget="many2one_avatar_user" optional="show"
                        domain="[('share', '=', False)]"/>
                    <field name="team_id" optional="hide"/>
                    <field name="priority" optional="hide" widget="priority"/>
                    <field name="activity_ids" optional="hide" widget="list_activity"/>
                    <field name="activity_user_id" optional="hide" string="Activity by" widget="many2one_avatar_user"/>
                    <field name="my_activity_date_deadline" optional="hide" string="My Deadline" widget="remaining_days" options="{'allow_order': '1'}"/>
                    <field name="activity_calendar_event_id" column_invisible="True"/>
                    <field name="campaign_id" optional="hide"/>
                    <field name="medium_id" optional="hide"/>
                    <field name="source_id" optional="hide"/>
                    <field name="company_currency" column_invisible="True"/>
                    <field name="expected_revenue" sum="Expected Revenues" optional="show" widget="monetary" options="{'currency_field': 'company_currency'}"/>
                    <field name="date_deadline" optional="hide"/>
                    <field name="recurring_revenue_monthly" sum="Expected MRR" optional="show" widget="monetary"
                        options="{'currency_field': 'company_currency'}" groups="crm.group_use_recurring_revenues"/>
                    <field name="recurring_revenue" sum="Recurring Revenue" optional="hide" widget="monetary"
                        options="{'currency_field': 'company_currency'}" groups="crm.group_use_recurring_revenues"/>
                    <field name="recurring_plan" optional="hide" groups="crm.group_use_recurring_revenues"/>
                    <field name="stage_id" optional="show" decoration-bf="1"/>
                    <field name="active" column_invisible="True"/>
                    <field name="probability" string="Probability (%)" optional="hide"/>
                    <field name="lost_reason_id" optional="hide"/>
                    <field name="tag_ids" optional="hide" widget="many2many_tags" options="{'color_field': 'color'}"/>
                    <field name="referred" column_invisible="True"/>
                    <field name="message_needaction" column_invisible="True"/>
                    <field name="lead_properties"/>
                    <button name="%(crm.action_lead_mail_compose)d" type="action" string="Email" icon="fa-envelope"/>
                    <button name="action_reschedule_meeting" class="text-warning" type="object" string="Reschedule"
                        icon="fa-calendar" invisible="not my_activity_date_deadline or not activity_calendar_event_id"/>
                    <button name="action_snooze" class="text-warning" type="object" string="Snooze 7d"
                        icon="fa-bell-slash" invisible="not my_activity_date_deadline or activity_calendar_event_id"/>
                </tree>
            </field>
        </record>

        <record id="crm_lead_view_tree_forecast" model="ir.ui.view">
            <field name="name">crm.lead.view.tree.forecast</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.crm_case_tree_view_oppor"/>
            <field name="mode">primary</field>
            <field name="priority">32</field>
            <field name="arch" type="xml">
                <xpath expr="//tree" position="attributes">
                    <attribute name="js_class">forecast_list</attribute>
                </xpath>
                <xpath expr="//field[@name='expected_revenue']" position="attributes">
                    <attribute name="optional">hide</attribute>
                </xpath>
                <xpath expr="//field[@name='expected_revenue']" position="after">
                    <field name="prorated_revenue" sum="Prorated Revenues" optional="show" widget="monetary" options="{'currency_field': 'company_currency'}"/>
                </xpath>
            </field>
        </record>

        <record id="crm_lead_view_list_activities" model="ir.ui.view">
            <field name="name">crm.lead.list.activities</field>
            <field name="model">crm.lead</field>
            <field name="mode">primary</field>
            <field name="priority">20</field>
            <field name="inherit_id" ref="crm.crm_case_tree_view_oppor"/>
            <field name="arch" type="xml">
                <xpath expr="//tree" position="attributes">
                    <attribute name="default_order">my_activity_date_deadline</attribute>
                </xpath>
                <field name="user_id" position="attributes">
                    <attribute name="optional">hide</attribute>
                </field>
                <field name="team_id" position="attributes">
                    <attribute name="optional">hide</attribute>
                </field>
            </field>
        </record>

        <record id="view_crm_case_my_activities_filter" model="ir.ui.view">
            <field name="name">crm.lead.search.myactivities</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.view_crm_case_leads_filter"/>
            <field name="arch" type="xml">
                <!-- we should not override the whole field but instead just set invisible attribute
                 to 0. but this approach is not working. the work around is temporary -->
                <xpath expr="//filter[@name='activities_overdue']" position="replace">
                    <filter string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all opportunities for which the next action date is before today"/>
                </xpath>
                <xpath expr="//filter[@name='activities_today']" position="replace">
                    <filter string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                </xpath>
                <xpath expr="//filter[@name='activities_upcoming_all']" position="replace">
                    <filter string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                </xpath>
                <xpath expr="//filter[@name='assigned_to_me']" position="replace">
                    <filter string="My Activities" name="assigned_to_me"
                        domain="[('activity_user_id','=',uid)]"
                        help="Opportunities that are assigned to me"/>
                </xpath>
            </field>
        </record>

        <record id="crm_lead_view_graph" model="ir.ui.view">
            <field name="name">crm.lead.view.graph</field>
            <field name="model">crm.lead</field>
            <field name="arch" type="xml">
                <graph string="Opportunities" sample="1">
                    <field name="stage_id"/>
                    <field name="user_id"/>
                    <field name="color" invisible="1"/>
                    <field name="automated_probability" invisible="1"/>
                    <field name="message_bounce" invisible="1"/>
                    <field name="recurring_revenue_monthly" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue_monthly_prorated" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue_prorated" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                </graph>
            </field>
        </record>

        <record id="crm_lead_view_graph_forecast" model="ir.ui.view">
            <field name="name">crm.lead.view.graph.forecast</field>
            <field name="model">crm.lead</field>
            <field name="priority">32</field>
            <field name="arch" type="xml">
                <graph string="Opportunities Forecast" sample="1" js_class="forecast_graph">
                    <field name="date_deadline"/>
                    <field name="prorated_revenue" type="measure"/>
                    <field name="automated_probability" invisible="1"/>
                    <field name="color" invisible="1"/>
                    <field name="day_open" invisible="1"/>
                    <field name="day_close" invisible="1"/>
                    <field name="message_bounce" invisible="1"/>
                    <field name="probability" invisible="1"/>
                    <field name="recurring_revenue_monthly" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue_monthly_prorated" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue_prorated" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                </graph>
            </field>
        </record>

        <record id="crm_lead_view_pivot" model="ir.ui.view">
            <field name="name">crm.lead.view.pivot</field>
            <field name="model">crm.lead</field>
            <field name="arch" type="xml">
                <pivot string="Pipeline Analysis" sample="1">
                    <field name="create_date" interval="month" type="row"/>
                    <field name="stage_id" type="col"/>
                    <field name="expected_revenue" type="measure"/>
                    <field name="color" invisible="1"/>
                    <field name="automated_probability" invisible="1"/>
                    <field name="message_bounce" invisible="1"/>
                    <field name="probability" invisible="1"/>
                    <field name="recurring_revenue_monthly" type="measure" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue_monthly_prorated" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue_prorated" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                </pivot>
            </field>
        </record>

        <record id="crm_lead_view_pivot_forecast" model="ir.ui.view">
            <field name="name">crm.lead.view.pivot.forecast</field>
            <field name="model">crm.lead</field>
            <field name="priority">32</field>
            <field name="arch" type="xml">
                <pivot string="Forecast Analysis" sample="1" js_class="forecast_pivot">
                    <field name="date_deadline" interval="month" type="row"/>
                    <field name="stage_id" type="col"/>
                    <field name="prorated_revenue" type="measure"/>
                    <field name="automated_probability" invisible="1"/>
                    <field name="color" invisible="1"/>
                    <field name="day_open" invisible="1"/>
                    <field name="day_close" invisible="1"/>
                    <field name="message_bounce" invisible="1"/>
                    <field name="probability" invisible="1"/>
                    <field name="recurring_revenue_monthly" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue_monthly_prorated" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                    <field name="recurring_revenue_prorated" groups="!crm.group_use_recurring_revenues" invisible="1"/>
                </pivot>
            </field>
        </record>

        <record id="view_crm_case_opportunities_filter" model="ir.ui.view">
            <field name="name">crm.lead.search.opportunity</field>
            <field name="model">crm.lead</field>
            <field name="priority">15</field>
            <field name="arch" type="xml">
                <search string="Search Opportunities">
                    <field name="name" string="Opportunity" filter_domain="[
                        '|', '|', '|', '|',
                        ('partner_id', 'ilike', self),
                        ('partner_name', 'ilike', self),
                        ('email_from', 'ilike', self),
                        ('name', 'ilike', self),
                        ('contact_name', 'ilike', self)]"/>
                    <field name="partner_id" operator="child_of" string="Customer" filter_domain="[
                        '|', '|', '|',
                        ('partner_id', 'ilike', self),
                        ('partner_name', 'ilike', self),
                        ('email_from', 'ilike', self),
                        ('contact_name', 'ilike', self)]"/>
                    <field name="tag_ids" string="Tag" filter_domain="[('tag_ids', 'ilike', self)]"/>
                    <field name="user_id"/>
                    <field name="team_id"/>
                    <field name="stage_id" domain="[]"/>
                    <field name="country_id"/>
                    <field name="city"/>
                    <field name="phone_mobile_search"/>
                    <field name="activity_state"/>
                    <field name="lead_properties"/>
                    <separator/>
                    <filter string="My Pipeline" name="assigned_to_me"
                        domain="[('user_id', '=', uid)]"
                        help="Opportunities that are assigned to me"/>
                    <filter string="Unassigned" name="unassigned"
                        domain="[('user_id', '=', False)]" help="No salesperson"/>
                    <filter string="Open Opportunities" name="open_opportunities"
                        domain="[('probability', '&lt;', 100), ('type', '=', 'opportunity'), ('active', '=', True)]"
                        help="Open Opportunities"/>
                    <separator/>
                    <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction', '=', True)]" groups="mail.group_mail_notification_type_inbox"/>
                    <separator/>
                    <filter string="Creation Date" name="creation_date" date="create_date"/>
                    <filter string="Closed Date" name="close_date" date="date_closed"/>
                    <separator/>
                    <filter string="Won" name="won" domain="['&amp;', ('active', '=', True), ('stage_id.is_won', '=', True)]"/>
                    <filter string="Lost" name="lost" domain="['&amp;', ('active', '=', False), ('probability', '=', 0)]"/>
                    <separator/>
                    <filter invisible="1" string="Overdue Opportunities" name="overdue_opp" domain="['&amp;', ('date_closed', '=', False), ('date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all opportunities for which the next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Group By" colspan="16">
                        <filter string="Salesperson" name="salesperson" context="{'group_by':'user_id'}"/>
                        <filter string="Sales Team" name="saleschannel" context="{'group_by':'team_id'}"/>
                        <filter name="stage" string="Stage" context="{'group_by':'stage_id'}"/>
                        <filter name="city" string="City" context="{'group_by': 'city'}"/>
                        <filter string="Country" name="country" context="{'group_by':'country_id'}" />
                        <filter string="Lost Reason" name="lostreason" context="{'group_by':'lost_reason_id'}"/>
                        <filter string="Company" name="company" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                        <filter string="Campaign" name="compaign" domain="[]" context="{'group_by':'campaign_id'}"/>
                        <filter string="Medium" name="medium" domain="[]" context="{'group_by':'medium_id'}"/>
                        <filter string="Source" name="source" domain="[]" context="{'group_by':'source_id'}"/>
                        <separator orientation="vertical" />
                        <filter string="Creation Date" name="month" context="{'group_by':'create_date:month'}"
                                invisible="context.get('crm_lead_view_hide_month')"/>
                        <filter string="Creation Date" name="group_by_create_date_day" context="{'group_by':'create_date:day'}"
                                invisible="not context.get('crm_lead_view_hide_month')"/>
                        <filter string="Conversion Date" name="date_conversion" context="{'group_by': 'date_conversion'}" groups="crm.group_use_lead"/>
                        <filter string="Expected Closing" name="date_deadline" context="{'group_by':'date_deadline'}"/>
                        <filter string="Closed Date" name="date_closed" context="{'group_by':'date_closed'}"/>
                        <separator/>
                        <filter string="Properties" name="group_by_lead_properties" context="{'group_by':'lead_properties'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="crm_lead_view_search_forecast" model="ir.ui.view">
            <field name="name">crm.lead.view.search.forecast</field>
            <field name="model">crm.lead</field>
            <field name="inherit_id" ref="crm.view_crm_case_opportunities_filter"/>
            <field name="mode">primary</field>
            <field name="priority">32</field>
            <field name="arch" type="xml">
                <xpath expr="//filter[@name='assigned_to_me']" position="before">
                    <filter name="forecast" string="Upcoming Closings" context="{'forecast_filter':1}"/>
                    <separator/>
                </xpath>
                <xpath expr="//filter[@name='date_deadline']" position="replace"/>
                <filter name="month" position="before">
                    <filter string="Expected Closing" name="date_deadline" context="{'group_by':'date_deadline:month'}"/>
                </filter>
            </field>
        </record>

        <!-- Lead Menu -->
        <record model="ir.actions.act_window" id="crm_lead_all_leads">
            <field name="name">Leads</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">tree,kanban,graph,pivot,calendar,form,activity</field>
            <field name="domain">['|', ('type','=','lead'), ('type','=',False)]</field>
            <field name="search_view_id" ref="crm.view_crm_case_leads_filter"/>
            <field name="context">{
                    'default_type':'lead',
                    'search_default_type': 'lead',
                    'search_default_to_process':1,
                }
            </field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a Lead
                </p><p>
                    Leads are the qualification step before the creation of an opportunity.
                </p>
            </field>
        </record>

        <record id="crm_lead_all_leads_view_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="1"/>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="crm_case_tree_view_leads"/>
            <field name="act_window_id" ref="crm_lead_all_leads"/>
        </record>

        <record id="crm_lead_all_leads_view_kanban" model="ir.actions.act_window.view">
            <field name="sequence" eval="2"/>
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="view_crm_lead_kanban"/>
            <field name="act_window_id" ref="crm_lead_all_leads"/>
        </record>

        <record id="crm_lead_all_leads_view_calendar" model="ir.actions.act_window.view">
            <field name="sequence" eval="3"/>
            <field name="view_mode">calendar</field>
            <field name="view_id" ref="crm_case_calendar_view_leads"/>
            <field name="act_window_id" ref="crm_lead_all_leads"/>
        </record>

        <record id="crm_lead_all_leads_view_pivot" model="ir.actions.act_window.view">
            <field name="sequence" eval="4"/>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="crm_lead_view_pivot"/>
            <field name="act_window_id" ref="crm_lead_all_leads"/>
        </record>

        <record id="crm_lead_all_leads_view_graph" model="ir.actions.act_window.view">
            <field name="sequence" eval="5"/>
            <field name="view_mode">graph</field>
            <field name="view_id" ref="crm_lead_view_graph"/>
            <field name="act_window_id" ref="crm_lead_all_leads"/>
        </record>

        <!-- My Activities Menu -->
        <record id="crm_lead_action_my_activities" model="ir.actions.act_window">
            <field name="name">My Activities</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">tree,kanban,graph,pivot,calendar,form,activity</field>
            <field name="view_id" ref="crm_lead_view_list_activities"/>
            <!-- Ensure that only records with at least one activity, "done" (archived) or not, are fetched. -->
            <field name="domain">[("activity_ids.active", "in", [True, False])]</field>
            <field name="search_view_id" ref="crm.view_crm_case_my_activities_filter"/>
            <field name="context">{'default_type': 'opportunity',
                    'search_default_assigned_to_me': 1}
            </field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Looks like nothing is planned.
                </p><p>
                    Schedule activities to keep track of everything you have to do.
                </p>
            </field>
        </record>

        <record id="crm_lead_action_my_activities_view_tree" model="ir.actions.act_window.view">
            <field name="sequence">1</field>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="crm.crm_lead_view_list_activities"/>
            <field name="act_window_id" ref="crm_lead_action_my_activities"/>
        </record>

        <!-- 'My Pipeline' menu : Server action, act_window_views and act_windows -->
        <record model="ir.actions.act_window" id="crm_lead_opportunities">
            <field name="name">Opportunities</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">kanban,tree,graph,pivot,form,calendar,activity</field>
            <field name="domain">[('type','=','opportunity')]</field>
            <field name="context">{
                    'default_type': 'opportunity',
                }
            </field>
            <field name="search_view_id" ref="crm.view_crm_case_opportunities_filter"/>
        </record>

        <record id="crm_lead_opportunities_view_kanban" model="ir.actions.act_window.view">
            <field name="sequence" eval="0"/>
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="crm_case_kanban_view_leads"/>
            <field name="act_window_id" ref="crm_lead_opportunities"/>
        </record>

        <record id="crm_lead_opportunities_view_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="1"/>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="crm.crm_case_tree_view_oppor"/>
            <field name="act_window_id" ref="crm_lead_opportunities"/>
        </record>

        <record id="crm_lead_opportunities_view_graph" model="ir.actions.act_window.view">
            <field name="sequence" eval="3"/>
            <field name="view_mode">graph</field>
            <field name="view_id" ref="crm_lead_view_graph"/>
            <field name="act_window_id" ref="crm_lead_opportunities"/>
        </record>

        <record id="crm_lead_opportunities_view_pivot" model="ir.actions.act_window.view">
            <field name="sequence" eval="4"/>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="crm_lead_view_pivot"/>
            <field name="act_window_id" ref="crm_lead_opportunities"/>
        </record>

        <record id="crm_lead_opportunities_view_calendar" model="ir.actions.act_window.view">
            <field name="sequence" eval="5"/>
            <field name="view_mode">calendar</field>
            <field name="view_id" ref="crm_case_calendar_view_leads"/>
            <field name="act_window_id" ref="crm_lead_opportunities"/>
        </record>

        <record model="ir.actions.act_window" id="crm_lead_action_pipeline">
            <field name="name">Pipeline</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">kanban,tree,graph,pivot,form,calendar,activity</field>
            <field name="domain">[('type','=','opportunity')]</field>
            <field name="context">{
                    'default_type': 'opportunity',
                    'search_default_assigned_to_me': 1
            }</field>
            <field name="search_view_id" ref="crm.view_crm_case_opportunities_filter"/>
        </record>

        <record id="crm_lead_action_pipeline_view_kanban" model="ir.actions.act_window.view">
            <field name="sequence" eval="0"/>
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="crm_case_kanban_view_leads"/>
            <field name="act_window_id" ref="crm_lead_action_pipeline"/>
        </record>

        <record id="crm_lead_action_pipeline_view_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="1"/>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="crm.crm_case_tree_view_oppor"/>
            <field name="act_window_id" ref="crm_lead_action_pipeline"/>
        </record>

        <record id="crm_lead_action_pipeline_view_calendar" model="ir.actions.act_window.view">
            <field name="sequence" eval="2"/>
            <field name="view_mode">calendar</field>
            <field name="view_id" ref="crm_case_calendar_view_leads"/>
            <field name="act_window_id" ref="crm_lead_action_pipeline"/>
        </record>

        <record id="crm_lead_action_pipeline_view_pivot" model="ir.actions.act_window.view">
            <field name="sequence" eval="3"/>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="crm_lead_view_pivot"/>
            <field name="act_window_id" ref="crm_lead_action_pipeline"/>
        </record>

        <record id="crm_lead_action_pipeline_view_graph" model="ir.actions.act_window.view">
            <field name="sequence" eval="5"/>
            <field name="view_mode">graph</field>
            <field name="view_id" ref="crm_lead_view_graph"/>
            <field name="act_window_id" ref="crm_lead_action_pipeline"/>
        </record>

        <record id="crm_lead_action_forecast" model="ir.actions.act_window">
            <field name="name">Forecast</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">kanban,graph,pivot,tree,form</field>
            <field name="domain">[('type', '=', 'opportunity')]</field>
            <field name="context">{
                'default_type': 'opportunity',
                'search_default_assigned_to_me': 1,
                'search_default_forecast': 1,
                'search_default_date_deadline': 1,
                'forecast_field': 'date_deadline'
            }</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No opportunity to display!
                </p><p>
                    Easily set expected closing dates and overview your revenue streams.
                </p>
            </field>
            <field name="search_view_id" ref="crm.crm_lead_view_search_forecast"/>
        </record>

        <record id="crm_lead_action_forecast_view_kanban" model="ir.actions.act_window.view">
            <field name="sequence" eval="0"/>
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="crm_lead_view_kanban_forecast"/>
            <field name="act_window_id" ref="crm_lead_action_forecast"/>
        </record>

        <record id="crm_lead_action_forecast_view_graph" model="ir.actions.act_window.view">
            <field name="sequence" eval="1"/>
            <field name="view_mode">graph</field>
            <field name="view_id" ref="crm_lead_view_graph_forecast"/>
            <field name="act_window_id" ref="crm_lead_action_forecast"/>
        </record>

        <record id="crm_lead_action_forecast_view_pivot" model="ir.actions.act_window.view">
            <field name="sequence" eval="2"/>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="crm_lead_view_pivot_forecast"/>
            <field name="act_window_id" ref="crm_lead_action_forecast"/>
        </record>

        <record id="crm_lead_action_forecast_view_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="3"/>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="crm_lead_view_tree_forecast"/>
            <field name="act_window_id" ref="crm_lead_action_forecast"/>
        </record>

        <!-- create a lead from 'Teams' kanban -->
        <record id="crm_lead_action_open_lead_form" model="ir.actions.act_window">
            <field name="name">New Lead</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="crm_lead_view_form"/>
            <field name="domain">[('type','=','lead')]</field>
            <field name="context">{
                'search_default_team_id': [active_id],
                'default_team_id': active_id,
                'default_type': 'lead',
            }</field>
            <field name="search_view_id" ref="crm.view_crm_case_leads_filter"/>
        </record>

</odoo>

```

## File: views\crm_lost_reason_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="crm_lost_reason_view_search" model="ir.ui.view">
        <field name="name">crm.lost.reason.view.search</field>
        <field name="model">crm.lost.reason</field>
        <field name="arch" type="xml">
            <search string="Search Opportunities">
                <field name="name"/>
                <filter string="Include archived" name="archived" domain="['|', ('active', '=', True), ('active', '=', False)]"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="crm_lost_reason_view_form" model="ir.ui.view">
        <field name="name">crm.lost.reason.form</field>
        <field name="model">crm.lost.reason</field>
        <field name="arch" type="xml">
            <form string="Lost Reason">
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_lost_leads" type="object"
                            class="oe_stat_button" icon="fa-star">
                            <div class="o_stat_info">
                                <field name="leads_count" class="o_stat_value"/>
                                <span class="o_stat_text"> Leads</span>
                            </div>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <div class="oe_title">
                        <div>
                            <label for="name"/>
                        </div>
                        <h1 class="mb32">
                            <field name="name" placeholder="e.g. Too expensive" class="mb16"/>
                        </h1>
                        <field name="active" invisible="1"/>
                    </div>
                </sheet>
            </form>
        </field>
    </record>

    <record id="crm_lost_reason_view_tree" model="ir.ui.view">
        <field name="name">crm.lost.reason.tree</field>
        <field name="model">crm.lost.reason</field>
        <field name="arch" type="xml">
            <tree string="Channel" editable="bottom">
                <field name="name"/>
            </tree>
        </field>
    </record>

    <!-- Configuration/Lead & Opportunities/Lost Reasons Menu -->
    <record id="crm_lost_reason_action" model="ir.actions.act_window">
        <field name="name">Lost Reasons</field>
        <field name="res_model">crm.lost.reason</field>
        <field name="view_mode">tree,form</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a Lost Reason
          </p><p>
            Use Lost Reasons to report on why opportunities are lost (e.g."Undercut by competitors").
          </p>
        </field>
    </record>
</odoo>

```

## File: views\crm_menu_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Top menu item -->
    <!--
        This menu item's purpose is to overwrite another one defined in
        the base module in order to set new groups.
    -->
    <menuitem
        id="crm_menu_root"
        name="CRM"
        web_icon="crm,static/description/icon.png"
        groups="sales_team.group_sale_salesman,sales_team.group_sale_manager"
        sequence="25"/>

    <!-- SALES (MAIN USER MENU) -->
    <menuitem
        id="crm_menu_sales"
        name="Sales"
        parent="crm_menu_root"
        sequence="1"/>
    <menuitem
        id="menu_crm_opportunities"
        name="My Pipeline"
        parent="crm_menu_sales"
        action="crm.action_your_pipeline"
        sequence="1"/>
    <menuitem
        id="crm_lead_menu_my_activities"
        name="My Activities"
        parent="crm_menu_sales"
        groups="sales_team.group_sale_salesman"
        action="crm.crm_lead_action_my_activities"
        sequence="2"/>

    <menuitem
        id="sales_team_menu_team_pipeline"
        name="Teams"
        parent="crm_menu_sales"
        action="sales_team.crm_team_action_pipeline"
        groups="sales_team.group_sale_manager"
        sequence="4"/>
    <menuitem
        id="res_partner_menu_customer"
        name="Customers"
        parent="crm_menu_sales"
        action="base.action_partner_form"
        sequence="5"/>

    <!-- LEADS (MAIN USER MENU) -->
    <menuitem
        id="crm_menu_leads"
        name="Leads"
        parent="crm_menu_root"
        action="crm.crm_lead_all_leads"
        groups="crm.group_use_lead"
        sequence="5"/>

    <!-- REPORTING -->
    <menuitem
        id="crm_menu_report"
        name="Reporting"
        parent="crm_menu_root"
        sequence="20"
        groups="sales_team.group_sale_salesman"/>
    <menuitem
        id="crm_menu_forecast"
        name="Forecast"
        parent="crm_menu_report"
        action="crm.action_opportunity_forecast"
        sequence="1"/>
    <menuitem
        id="crm_opportunity_report_menu" 
        name="Pipeline"
        parent="crm_menu_report"
        action="crm.crm_opportunity_report_action"
        sequence="2"/>
    <menuitem
        id="crm_opportunity_report_menu_lead"
        name="Leads"
        parent="crm_menu_report"
        action="crm.crm_opportunity_report_action_lead"
        sequence="3"/>
    <menuitem
        id="crm_activity_report_menu"
        name="Activities"
        parent="crm_menu_report"
        action="crm_activity_report_action"
        sequence="4"/>

    <!-- CONFIGURATION -->
    <menuitem
        id="crm_menu_config"
        name="Configuration"
        parent="crm_menu_root"
        action="crm.action_your_pipeline"
        sequence="25" groups="sales_team.group_sale_manager"/>
    <menuitem
        id="crm_config_settings_menu"
        name="Settings"
        parent="crm_menu_config"
        action="crm.crm_config_settings_action"
        groups="base.group_system"
        sequence="0"/>
    <menuitem
        id="menu_crm_config_opportunity"
        name="Opportunities"
        parent="crm_menu_config"
        sequence="1"
        groups="sales_team.group_sale_manager"/>
    <menuitem
        id="crm_team_config"
        name="Sales Teams"
        parent="crm_menu_config"
        action="sales_team.crm_team_action_config"
        sequence="5"/>
    <menuitem
        id="crm_team_member_config"
        name="Teams Members"
        parent="crm_menu_config"
        action="sales_team.crm_team_member_action"
        sequence="6"
        groups="base.group_no_one"/>
    <menuitem
        id="crm_team_menu_config_activity_types"
        name="Activity Types"
        parent="crm_menu_config"
        action="sales_team.mail_activity_type_action_config_sales"
        sequence="10"/>
    <menuitem
        id="mail_activity_plan_menu_config_lead"
        name="Activity Plans"
        parent="crm_menu_config"
        action="mail_activity_plan_action_lead"
        groups="sales_team.group_sale_manager"
        sequence="11"
    />
    <menuitem
        id="crm_recurring_plan_menu_config"
        name="Recurring Plans"
        parent="crm_menu_config"
        action="crm.crm_recurring_plan_action"
        sequence="12"
        groups="crm.group_use_recurring_revenues"/>
    <menuitem
        id="menu_crm_config_lead"
        name="Pipeline"
        parent="crm_menu_config"
        sequence="15"
        groups="sales_team.group_sale_manager"/>
    <menuitem
        id="menu_crm_lead_stage_act"
        name="Stages"
        sequence="0"
        parent="menu_crm_config_lead"
        action="crm.crm_stage_action"
        groups="base.group_no_one"/>
    <menuitem
        id="menu_crm_lead_categ"
        name="Tags"
        action="sales_team.sales_team_crm_tag_action"
        parent="menu_crm_config_lead"
        sequence="1"/>
    <menuitem
        id="menu_crm_lost_reason"
        name="Lost Reasons"
        parent="menu_crm_config_lead"
        action="crm.crm_lost_reason_action"
        sequence="6"/>

    <menuitem
        id="menu_import_crm"
        name="Import &amp; Synchronize"
        parent="crm_menu_root"/>
</odoo>

```

## File: views\crm_recurring_plan_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="crm_recurring_plan_view_tree" model="ir.ui.view">
        <field name="name">crm.recurring.plan.view.tree</field>
        <field name="model">crm.recurring.plan</field>
        <field name="arch" type="xml">
            <tree editable="bottom">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="number_of_months"/>
            </tree>
        </field>
    </record>

    <record id="crm_recurring_plan_view_search" model="ir.ui.view">
        <field name="name">crm.recurring.plan.view.search</field>
        <field name="model">crm.recurring.plan</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <filter name="active" string="Archived" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="crm_recurring_plan_action" model="ir.actions.act_window">
        <field name="name">Recurring Plans</field>
        <field name="res_model">crm.recurring.plan</field>
        <field name="view_mode">tree</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Recurring Plan
            </p>
            <p>
                Set Recurring Plans on Opportunities to display the contracts' renewal periodicity<br/>(e.g: Monthly, Yearly).
            </p>
        </field>
    </record>

</data></odoo>

```

## File: views\crm_stage_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="crm_lead_stage_search" model="ir.ui.view">
        <field name="name">Stage - Search</field>
        <field name="model">crm.stage</field>
        <field name="arch" type="xml">
            <search string="Stage Search">
                <field name="name"/>
                <field name="sequence"/>
                <field name="is_won"/>
                <field name="team_id"/>
            </search>
        </field>
    </record>

    <!-- STAGES TREE VIEW + MUTI_EDIT -->
    <record id="crm_stage_tree" model="ir.ui.view">
        <field name="name">crm.stage.tree</field>
        <field name="model">crm.stage</field>
        <field name="arch" type="xml">
            <tree string="Stages" multi_edit="1">
                <field name="sequence" widget="handle"/>
                <field name="name" readonly="1"/>
                <field name="is_won"/>
                <field name="team_id"/>
            </tree>
        </field>
    </record>

    <record id="crm_stage_form" model="ir.ui.view">
        <field name="name">crm.stage.form</field>
        <field name="model">crm.stage</field>
        <field name="priority" eval="1"/>
        <field name="arch" type="xml">
            <form string="Stage">
                <sheet>
                    <div class="oe_title">
                        <label for="name"/>
                        <h1>
                            <field name="name" placeholder="e.g. Negotiation"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="is_won"/>
                            <field name="fold"/>
                            <field name="team_id" options='{"no_open": True, "no_create": True}' invisible="team_count &lt;= 1" context="{'kanban_view_ref': 'sales_team.crm_team_view_kanban'}"/>
                        </group>
                        <field name="team_count" invisible="1"/>
                    </group>
                    <separator string="Requirements"/>
                    <field name="requirements" nolabel="1" placeholder="Give your team the requirements to move an opportunity to this stage."/>
                </sheet>
            </form>
        </field>
    </record>

    <record id="crm_stage_action" model="ir.actions.act_window">
        <field name="name">Stages</field>
        <field name="res_model">crm.stage</field>
        <field name="view_id" ref="crm.crm_stage_tree"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Set a new stage in your opportunity pipeline
          </p><p>
            Stages allow salespersons to easily track how a specific opportunity
            is positioned in the sales cycle.
          </p>
        </field>
    </record>

</odoo>

```

## File: views\crm_team_member_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="crm_team_member_view_tree" model="ir.ui.view">
        <field name="name">crm.team.member.view.tree</field>
        <field name="model">crm.team.member</field>
        <field name="inherit_id" ref="sales_team.crm_team_member_view_tree"/>
        <field name="arch" type="xml">
            <field name="user_id" position="after">
                <field name="assignment_enabled" column_invisible="True"/>
                <field name="assignment_optout"/>
                <field name="assignment_max"/>
                <field name="lead_month_count"/>
            </field>
        </field>
    </record>

    <record id="crm_team_member_view_kanban" model="ir.ui.view">
        <field name="name">crm.team.member.view.kanban.inherit.crm</field>
        <field name="model">crm.team.member</field>
        <field name="inherit_id" ref="sales_team.crm_team_member_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_kanban_details')]" position="after">
                <field name="assignment_enabled" invisible="1"/>
                <field name="assignment_optout" invisible="1"/>
                <div class="o_member_assignment"
                        invisible="not assignment_enabled or assignment_optout">
                    <field name="assignment_max" invisible="1"/>
                    <field name="lead_month_count" widget="gauge"
                        options="{'max_field': 'assignment_max'}"
                        invisible="assignment_max == 0"/>
                </div>
            </xpath>
        </field>
    </record>

    <record id="crm_team_member_view_form" model="ir.ui.view">
        <field name="name">crm.team.member.view.form.inherit.crm</field>
        <field name="model">crm.team.member</field>
        <field name="inherit_id" ref="sales_team.crm_team_member_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='member_partner_info']" position="after">
                <group name="group_assign" invisible="not assignment_enabled">
                    <field name="assignment_enabled" invisible="1"/>
                    <field name="assignment_optout"/>
                    <label for="lead_month_count" invisible="assignment_optout"/>
                    <div invisible="assignment_optout">
                        <field name="lead_month_count" class="oe_inline"/>
                        <span class="oe_inline"> / </span>
                        <field name="assignment_max" class="oe_inline"/>
                        <span class="oe_inline"> (max) </span>
                    </div>
                    <field name="assignment_domain" string="Domain" widget="domain"
                        options="{'model': 'crm.lead'}"
                        invisible="assignment_max == 0 or assignment_optout"/>
                </group>
            </xpath>
        </field>
    </record>

    <record id="sales_team.crm_team_member_action" model="ir.actions.act_window">
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a Team Member
            </p><p>
                Team Members are salespersons assigned to specific teams.
            </p>
        </field>
    </record>

</data></odoo>

```

## File: views\crm_team_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <!-- CRM lead search by Salesteams -->
        <record id="crm_case_form_view_salesteams_lead" model="ir.actions.act_window">
            <field name="name">Leads</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="domain">['|', ('type','=','lead'), ('type','=',False)]</field>
            <field name="view_ids"
                   eval="[(5, 0, 0),
                          (0, 0, {'view_mode': 'tree', 'view_id': ref('crm_case_tree_view_leads')}),
                          (0, 0, {'view_mode': 'kanban', 'view_id': ref('view_crm_lead_kanban')})]"/>
            <field name="search_view_id" ref="crm.view_crm_case_leads_filter"/>
            <field name="context">{
                    'search_default_team_id': [active_id],
                    'default_team_id': active_id,
                    'default_type': 'lead',
                }
            </field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a new lead
                </p><p>
                    Use leads if you need a qualification step before creating an
                    opportunity or a customer. It can be a business card you received,
                    a contact form filled in your website, or a file of unqualified
                    prospects you import, etc.
                </p>
            </field>
        </record>

        <!-- CRM opportunity search by Salesteams -->
        <record id="crm_case_form_view_salesteams_opportunity" model="ir.actions.act_window">
            <field name="name">Opportunities</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">kanban,tree,graph,form,calendar,pivot</field>
            <field name="domain">[('type','=','opportunity')]</field>
            <field name="view_id" ref="crm.crm_case_kanban_view_leads"/>
            <field name="search_view_id" ref="crm.view_crm_case_opportunities_filter"/>
            <field name="context">{
                    'search_default_team_id': [active_id],
                    'default_team_id': active_id,
                    'default_type': 'opportunity',
                    'default_user_id': uid,
                }
            </field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a new lead
                </p><p>
                    Odoo helps you keep track of your sales pipeline to follow
                    up potential sales and better forecast your future revenues.
                </p><p>
                    You will be able to plan meetings and phone calls from
                    opportunities, convert them into quotations, attach related
                    documents, track all discussions, and much more.
                </p>
            </field>
        </record>

         <record id="crm_lead_action_team_overdue_opportunity" model="ir.actions.act_window">
            <field name="name">Overdue Opportunities</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">kanban,tree,graph,form,calendar,pivot</field>
            <field name="domain">[('type','=','opportunity')]</field>
            <field name="view_id" ref="crm.crm_case_kanban_view_leads"/>
            <field name="search_view_id" ref="crm.view_crm_case_opportunities_filter"/>
            <field name="context">{
                    'search_default_team_id': [active_id],
                    'search_default_overdue_opp': 1,
                    'default_team_id': active_id,
                    'default_type': 'opportunity',
                    'default_user_id': uid,
                }
            </field>
        </record>

       <record id="action_report_crm_lead_salesteam" model="ir.actions.act_window">
           <field name="name">Leads Analysis</field>
           <field name="res_model">crm.lead</field>
           <field name="context">{'search_default_team_id': [active_id], 'search_default_filter_create_date': 1}</field>
           <field name="domain">[]</field>
           <field name="view_mode">graph,pivot,tree,form</field>
           <field name="view_id" ref="crm_lead_view_graph"/>
           <field name="search_view_id" ref="crm.view_crm_case_leads_filter"/>
           <field name="help">Leads Analysis allows you to check different CRM related information like the treatment delays or number of leads per state. You can sort out your leads analysis by different groups to get accurate grained analysis.</field>
       </record>
        <record id="action_report_crm_lead_salesteam_view_graph" model="ir.actions.act_window.view">
            <field name="sequence">2</field>
            <field name="view_mode">graph</field>
            <field name="view_id" ref="crm_lead_view_graph"/>
            <field name="act_window_id" ref="action_report_crm_lead_salesteam"/>
        </record>
        <record id="action_report_crm_lead_salesteam_view_pivot" model="ir.actions.act_window.view">
            <field name="sequence">3</field>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="crm_lead_view_pivot"/>
            <field name="act_window_id" ref="action_report_crm_lead_salesteam"/>
        </record>
        <record id="action_report_crm_lead_salesteam_view_tree" model="ir.actions.act_window.view">
            <field name="sequence">4</field>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="crm_lead_view_tree_reporting"/>
            <field name="act_window_id" ref="action_report_crm_lead_salesteam"/>
        </record>

       <record id="action_report_crm_opportunity_salesteam" model="ir.actions.act_window">
            <field name="name">Pipeline Analysis</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">graph,pivot,tree,form</field>
            <field name="search_view_id" ref="crm.crm_opportunity_report_view_search"/>
            <field name="context">{
                'search_default_team_id': [active_id],
                'tree_view_ref': 'crm.crm_lead_view_tree_opportunity_reporting',
                'search_default_filter_opportunity': True,
                'search_default_filter_create_date': 1}</field>
            <field name="domain">[]</field>
            <field name="help">Opportunities Analysis gives you an instant access to your opportunities with information such as the expected revenue, planned cost, missed deadlines or the number of interactions per opportunity. This report is mainly used by the sales manager in order to do the periodic review with the channels of the sales pipeline.</field>
        </record>

        <record id="crm_team_view_tree" model="ir.ui.view">
            <field name="name">crm.team.tree.inherit.crm</field>
            <field name="model">crm.team</field>
            <field name="inherit_id" ref="sales_team.crm_team_view_tree"/>
            <field name="arch" type="xml">
                <field name="name" position="after">
                    <field string="Alias" name="alias_id"/>
                </field>
            </field>
        </record>

        <record id="sales_team_form_view_in_crm" model="ir.ui.view">
            <field name="name">crm.team.form.inherit</field>
            <field name="model">crm.team</field>
            <field name="inherit_id" ref="sales_team.crm_team_view_form"/>
            <field name="priority">12</field>
            <field name="arch" type="xml">
                <xpath expr="//sheet" position="before">
                    <field name="use_leads" invisible="1"/>
                    <field name="use_opportunities" invisible="1"/>
                    <header>
                        <button name="action_assign_leads" type="object"
                            string="Assign Leads"
                            class="oe_highlight"
                            confirm="This will assign leads to all members. Do you want to proceed?"
                            invisible="not use_leads and not use_opportunities or not assignment_enabled"
                            confirm-label="Assign Leads"/>
                    </header>
                </xpath>
                <xpath expr="//div[@name='options_active']" position="inside">
                    <span name="opportunities" groups="crm.group_use_lead">
                        <field name="use_opportunities"/>
                        <label for="use_opportunities"/>
                    </span>
                    <span name="leads" groups="crm.group_use_lead">
                        <field name="use_leads"/>
                        <label for="use_leads" string="Leads"/>
                    </span>
                </xpath>
                <xpath expr="//field[@name='user_id']" position="after">
                    <label for="alias_name" string="Email Alias"
                           invisible="not use_leads and not use_opportunities"/>
                    <div name="alias_def" invisible="not use_leads and not use_opportunities">
                        <field name="alias_id" string="Email Alias" class="oe_read_only" required="0"/>
                        <div class="oe_edit_only" name="edit_alias" dir="ltr">
                            <field name="alias_name" class="oe_inline"/>@
                            <field name="alias_domain_id" class="oe_inline" placeholder="e.g. domain.com"
                                   options="{'no_create': True, 'no_open': True}"/>
                        </div>
                    </div>
                    <field name="alias_contact"
                        string="Accept Emails From"
                        invisible="not use_leads and not use_opportunities"/>
                </xpath>
                <xpath expr="//group[@name='right']" position="attributes">
                    <attribute name="string">Assignment Rules</attribute>
                    <attribute name="invisible">not assignment_enabled</attribute>
                </xpath>
                <xpath expr="//group[@name='right']" position="inside">
                    <field name="assignment_enabled" invisible="1"/>
                    <field name="assignment_auto_enabled" invisible="1"/>
                    <field name="lead_all_assigned_month_exceeded" invisible="1"/>
                    <div colspan="2">
                        <div class="o_crm_lead_all_assigned_month_exceeded" invisible="not lead_all_assigned_month_exceeded"/>
                        <div class="o_crm_lead_month_assignment text-muted" invisible="not assignment_enabled">
                            <i class="fa fa-info-circle me-2" title="Assigned Lead Count"/>
                            <field name="lead_all_assigned_month_count" class="oe_inline"/><span> leads assigned this month
                            on a maximum of </span><field name="assignment_max" class="oe_inline"/>
                        </div>
                    </div>
                    <field name="assignment_domain" widget="domain" string="Domain"
                        options="{'foldable': True, 'model': 'crm.lead', 'in_dialog': True}"
                        invisible="not assignment_enabled"/>
                    <field name="assignment_optout" invisible="not assignment_auto_enabled"/>
                </xpath>
                <xpath expr="//field[@name='member_ids']" position="attributes">
                    <attribute name="invisible">assignment_enabled</attribute>
                </xpath>
                <xpath expr="//field[@name='crm_team_member_ids']" position="attributes">
                    <attribute name="invisible">not assignment_enabled</attribute>
                </xpath>
            </field>
        </record>

        <!-- Case Teams Action -->
        <record id="action_crm_tag_kanban_view_salesteams_oppor11" model="ir.actions.act_window.view">
            <field name="sequence" eval="0"/>
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="crm_case_kanban_view_leads"/>
            <field name="act_window_id" ref="crm_case_form_view_salesteams_opportunity"/>
        </record>

        <record id="action_crm_tag_tree_view_salesteams_oppor11" model="ir.actions.act_window.view">
            <field name="sequence" eval="1"/>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="crm_case_tree_view_oppor"/>
            <field name="act_window_id" ref="crm_case_form_view_salesteams_opportunity"/>
        </record>

        <record id="action_opportunity_form" model="ir.actions.act_window">
            <field name="name">New Opportunity</field>
            <field name="res_model">crm.lead</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="crm_lead_view_form"/>
            <field name="domain">[('type','=','opportunity')]</field>
            <field name="context">{
                    'search_default_team_id': [active_id],
                    'default_team_id': active_id,
                    'default_type': 'opportunity',
                    'default_user_id': uid,
            }
            </field>
            <field name="search_view_id" ref="crm.view_crm_case_opportunities_filter"/>
        </record>

        <record id="sales_team.crm_team_action_pipeline" model="ir.actions.act_window">
            <field name="domain">[('use_opportunities', '=', True)]</field>
        </record>

        <record id="crm_team_view_kanban_dashboard" model="ir.ui.view">
            <field name="name">crm.team.view.kanban.dashboard.inherit.crm</field>
            <field name="model">crm.team</field>
            <field name="inherit_id" ref="sales_team.crm_team_view_kanban_dashboard"/>
            <field name="arch" type="xml">
                <data>
                    <xpath expr="//templates" position="before">
                        <field name="alias_id"/>
                        <field name="alias_name"/>
                        <field name="alias_domain"/>
                        <field name="use_opportunities"/>
                        <field name="use_leads"/>
                    </xpath>

                    <xpath expr="//div[hasclass('o_primary')]" position="after">
                        <div t-if="record.alias_name.value and record.alias_domain.value">
                            <span t-translation="off"><i class="fa fa-envelope-o" aria-label="Leads" title="Leads" role="img"></i>&amp;nbsp; <field name="alias_id"/></span>
                        </div>
                    </xpath>

                    <xpath expr="//t[@name='first_options']" position="after">
                        <div class="row" t-if="record.lead_unassigned_count.raw_value">
                            <div class="col-8">
                                <a name="%(crm_case_form_view_salesteams_lead)d" type="action" context="{'search_default_unassigned_leads': 1}">
                                    <field name="lead_unassigned_count" class="me-1"/>
                                    <t t-if="record.lead_unassigned_count.raw_value == 1">Unassigned Lead</t>
                                    <t t-else="">Unassigned Leads</t>
                                </a>
                            </div>
                        </div>
                        <div class="row" t-if="record.opportunities_count.raw_value">
                            <div class="col-8">
                                <a name="%(crm_case_form_view_salesteams_opportunity)d" type="action" context="{'search_default_open_opportunities': True}"> <!-- context="{'search_default_probability': NOT or < 100}" -->
                                    <field name="opportunities_count" class="me-1"/>
                                    <t t-if="record.opportunities_count.raw_value == 1">Open Opportunity</t>
                                    <t t-else="">Open Opportunities</t>
                                </a>
                            </div>
                            <div class="col-4 text-end text-truncate">
                                <field name="opportunities_amount" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                            </div>
                        </div>
                        <div class="row" t-if="record.opportunities_overdue_count.raw_value">
                            <div class="col-8">
                                <a name="%(crm_lead_action_team_overdue_opportunity)d" type="action">
                                    <field name="opportunities_overdue_count" class="me-1"/>
                                    <t t-if="record.opportunities_overdue_count.raw_value == 1">Overdue Opportunity</t>
                                    <t t-else="">Overdue Opportunities</t>
                                </a>
                            </div>
                             <div class="col-4 text-end text-truncate">
                                <field name="opportunities_overdue_amount" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                            </div>
                        </div>
                    </xpath>

                    <xpath expr="//div[hasclass('o_kanban_manage_view')]/h5[hasclass('o_kanban_card_manage_title')]" position="after">
                        <div t-if="record.use_leads.raw_value" groups="crm.group_use_lead">
                            <a name="%(crm_case_form_view_salesteams_lead)d" type="action">
                                Leads
                            </a>
                        </div>
                        <div t-if="record.use_opportunities.raw_value">
                            <a name="%(crm_case_form_view_salesteams_opportunity)d" type="action">
                                Opportunities
                            </a>
                        </div>
                    </xpath>

                    <xpath expr="//div[hasclass('o_kanban_manage_new')]/h5[hasclass('o_kanban_card_manage_title')]" position="after">
                        <div t-if="record.use_leads.raw_value" groups="crm.group_use_lead">
                            <a name="%(crm_lead_action_open_lead_form)d" type="action">
                                Leads
                            </a>
                        </div>
                        <div t-if="record.use_opportunities.raw_value">
                            <a  name="%(action_opportunity_form)d" type="action">
                                Opportunity
                            </a>
                        </div>
                    </xpath>

                    <xpath expr="//div[hasclass('o_kanban_manage_reports')]/h5[hasclass('o_kanban_card_manage_title')]" position="after">
                        <div t-if="record.use_leads.raw_value" groups="crm.group_use_lead">
                            <a name="%(action_report_crm_lead_salesteam)d" type="action">
                                Leads
                            </a>
                        </div>
                        <div t-if="record.use_opportunities.raw_value">
                            <a name="%(action_report_crm_opportunity_salesteam)d" type="action">
                                Opportunities
                            </a>
                        </div>
                    </xpath>

                    <xpath expr="//div[hasclass('o_kanban_manage_reports')]/div[@name='o_team_kanban_report_separator']" position="after">
                        <div t-if="record.use_opportunities.raw_value">
                            <a name="%(crm.crm_activity_report_action_team)d" type="action">
                                Activities
                            </a>
                        </div>
                    </xpath>
                </data>
            </field>
        </record>

</odoo>

```

## File: views\digest_views.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <record id="digest_digest_view_form" model="ir.ui.view">
        <field name="name">digest.digest.view.form.inherit.crm.lead</field>
        <field name="model">digest.digest</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="digest.digest_digest_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='kpis']/group[last()]" position="before">
                <group name="kpi_crm" string="CRM" groups="sales_team.group_sale_salesman_all_leads">
                    <field name="kpi_crm_lead_created"/>
                    <field name="kpi_crm_opportunities_won"/>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\mail_activity_plan_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="mail_activity_plan_action_lead" model="ir.actions.act_window">
            <field name="name">Lead Plans</field>
            <field name="res_model">mail.activity.plan</field>
            <field name="view_mode">tree,form</field>
            <field name="search_view_id" ref="mail.mail_activity_plan_view_search"/>
            <field name="context">{'default_res_model': 'crm.lead'}</field>
            <field name="domain">[('res_model', '=', 'crm.lead')]</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Add a new plan
                </p>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\mail_activity_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sales_team.mail_activity_type_action_config_sales" model="ir.actions.act_window">
        <field name="domain">['|', ('res_model', '=', False), ('res_model', 'in', ['crm.lead', 'res.partner'])]</field>
        <field name="context">{'default_res_model': 'crm.lead'}</field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.crm</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="5"/>
        <field name="inherit_id" ref="base.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//form" position="inside">
                <app data-string="CRM" string="CRM" name="crm" groups="sales_team.group_sale_manager">
                    <block title="CRM">
                        <setting help="Define recurring plans and revenues on Opportunities">
                            <field name="group_use_recurring_revenues"/>
                            <div invisible="not group_use_recurring_revenues">
                                <button type="action" name="crm.crm_recurring_plan_action"
                                        string="Manage Recurring Plans" icon="oi-arrow-right" class="oe_link"/>
                            </div>
                        </setting>
                        <setting id="crm_lead" title="Use leads if you need a qualification step before creating an opportunity or a customer. It can be a business card you received, a contact form filled in your website, or a file of unqualified prospects you import, etc. Once qualified, the lead can be converted into a business opportunity and/or a new customer in your address book." help="Add a qualification step before the creation of an opportunity">
                            <field name="group_use_lead"/>
                        </setting>
                    </block>
                    <block>
                        <setting help="Assign salespersons into multiple Sales Teams.">
                            <field name="is_membership_multi"/>
                        </setting>
                    </block>
                    <block>
                        <field name="predictive_lead_scoring_fields_str" invisible="1"/>
                        <field name="predictive_lead_scoring_start_date_str" invisible="1"/>
                        <setting title="This can be used to compute statistical probability to close a lead" name="predictive_lead_setting_container" string="Predictive Lead Scoring">
                            <div class="text-muted">
                                The success rate is computed based on <b>
                                    <field name="predictive_lead_scoring_field_labels" class="d-inline"/>
                                </b>
                                for the leads created as of the
                                <b><field name="predictive_lead_scoring_start_date" class="oe_inline" readonly="1"/></b>.
                                </div>
                            <div class="mt16" groups="base.group_erp_manager">
                                <button name="%(crm_lead_pls_update_action)d" type="action"
                                    string="Update Probabilities"
                                    class="btn-primary"/>
                            </div>
                        </setting>
                        <setting title="This can be used to automatically assign leads to sales persons based on rules" documentation="/applications/sales/crm/track_leads/lead_scoring.html#assign-leads">
                            <field name="crm_use_auto_assignment"/>
                            <div class="text-muted">
                                <span>Periodically assign leads based on rules</span><br />
                                <span invisible="not crm_use_auto_assignment">
                                    All sales teams will use this setting by default unless
                                    specified otherwise.
                                </span>
                            </div>
                            <div class="row flex-row flex-nowrap mt16" invisible="not crm_use_auto_assignment">
                                <label string="Running" for="crm_auto_assignment_action" class="col-lg-3 o_light_label"/>
                                <field name="crm_auto_assignment_action"
                                    required="crm_use_auto_assignment"/>
                                <button name="action_crm_assign_leads" type="object" class="btn-link w-auto">
                                    <i title="Update now" role="img" aria-label="Update now" class="fa fa-fw fa-refresh"></i>
                                </button>
                            </div>
                            <div class="row mt16" invisible="not crm_use_auto_assignment or crm_auto_assignment_action == 'manual'">
                                <label string="Repeat every" for="crm_auto_assignment_interval_type" class="col-lg-3 o_light_label"/>
                                <field name="crm_auto_assignment_interval_number"
                                    class="oe_inline me-2"
                                    required="crm_use_auto_assignment and crm_auto_assignment_action == 'auto'"/>
                                <field name="crm_auto_assignment_interval_type"
                                    class="oe_inline"
                                    required="crm_use_auto_assignment and crm_auto_assignment_action == 'auto'"/>
                            </div>
                            <div class="row" invisible="not crm_use_auto_assignment or crm_auto_assignment_action == 'manual'">
                                <label string="Next Run" for="crm_auto_assignment_run_datetime" class="col-lg-3 o_light_label"/>
                                <field name="crm_auto_assignment_run_datetime"/>
                            </div>
                        </setting>
                    </block>

                    <block title="Lead Generation" name="convert_visitor_setting_container">
                        <setting string="Lead Enrichment" help="Enrich your leads with company data based on their email addresses">
                            <field name="module_crm_iap_enrich"/>
                            <div class="mt8" invisible="not module_crm_iap_enrich">
                                <field name="lead_enrich_auto" class="o_light_label" widget="radio" required="True"/>
                            </div>
                        </setting>

                        <setting id="crm_iap_mine_settings" string="Lead Mining" documentation="/applications/sales/crm/acquire_leads/lead_mining.html" help="Generate new leads based on their country, industry, size, etc.">
                            <field name="module_crm_iap_mine"/>
                        </setting>

                    </block>
                    <block name="generate_lead_setting_container">
                        <setting id="website_crm_iap_reveal_settings" string="Visits to Leads" help="Convert visitors of your website into leads and perform data enrichment based on their IP address">
                            <field name="module_website_crm_iap_reveal"/>
                        </setting>
                    </block>
                </app>
            </xpath>
        </field>
    </record>

    <record id="crm_config_settings_action" model="ir.actions.act_window">
        <field name="name">Settings</field>
        <field name="res_model">res.config.settings</field>
        <field name="view_id" ref="res_config_settings_view_form"/>
        <field name="view_mode">form</field>
        <field name="target">inline</field>
        <field name="context">{'module' : 'crm', 'bin_size': False}</field>
    </record>

</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0"?>
<odoo>

        <!-- Partner kanban view inherit -->
        <record id="crm_lead_partner_kanban_view" model="ir.ui.view">
            <field name="name">res.partner.kanban.inherit</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.res_partner_kanban_view"/>
            <field name="priority" eval="10"/>
            <field name="arch" type="xml">
                <field name="mobile" position="after">
                    <field name="opportunity_count" groups="sales_team.group_sale_salesman"/>
                </field>
                <xpath expr="//div[hasclass('oe_kanban_bottom_left')]" position="inside">
                    <a t-if="record.opportunity_count.value>0" href="#"
                       groups="sales_team.group_sale_salesman"
                       data-type="object" data-name="action_view_opportunity"
                       class="oe_kanban_action oe_kanban_action_a me-1">
                        <span class="badge rounded-pill">
                            <i class="fa fa-fw fa-star" aria-label="Opportunities"
                               role="img" title="Opportunities"/>
                            <t t-out="record.opportunity_count.value"/>
                        </span>
                    </a>
                </xpath>
            </field>
        </record>

        <!-- Add contextual button on partner form view -->
        <record id="view_partners_form_crm1" model="ir.ui.view">
            <field name="name">view.res.partner.form.crm.inherited1</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field eval="1" name="priority"/>
            <field name="arch" type="xml">
                <data>
                    <div name="button_box" position="inside">
                        <button class="oe_stat_button o_res_partner_tip_opp" type="object"
                            name="action_view_opportunity"
                            icon="fa-star"
                            groups="sales_team.group_sale_salesman"
                            context="{'default_partner_id': id, 'default_type':'opportunity'}">
                            <field string="Opportunities" name="opportunity_count" widget="statinfo"/>
                        </button>
                    </div>
                </data>
            </field>
        </record>

        <record id="res_partner_view_form_simple_form" model="ir.ui.view">
            <field name="name">res.partner.view.form.simple.form.crm</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_simple_form"/>
            <field name="arch" type="xml">
                <data>
                    <xpath expr="//form">
                        <field name="team_id" invisible="1"/>
                    </xpath>
                </data>
            </field>
        </record>

</odoo>

```

## File: views\utm_campaign_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.ui.view" id="utm_campaign_view_kanban">
        <field name="name">utm.campaign.view.kanban</field>
        <field name="model">utm.campaign</field>
        <field name="inherit_id" ref="utm.utm_campaign_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='user_id']" position="after">
                <field name="use_leads"/>
            </xpath>
            <xpath expr="//div[hasclass('oe_kanban_bottom_left')]" position="inside">
                <t t-if="record.use_leads.raw_value">
                    <t t-set="crm_lead_count_label">Leads</t>
                </t>
                <t t-else="">
                    <t t-set="crm_lead_count_label">Opportunities</t>
                </t>
                <a t-if="record.crm_lead_count" href="#" t-att-title="crm_lead_count_label" role="button"
                    groups="sales_team.group_sale_salesman" data-type="object" data-name="action_redirect_to_leads_opportunities"
                    class="oe_kanban_action oe_kanban_action_a btn-outline-primary rounded-pill me-1 order-3">
                    <span class="badge">
                        <i class="fa fa-fw fa-star" t-att-aria-label="crm_lead_count_label" role="img"/>
                        <field name="crm_lead_count"/>
                    </span>
                </a>
            </xpath>
        </field>
    </record>

    <record model="ir.ui.view" id="utm_campaign_view_form">
        <field name="name">utm.campaign.view.form</field>
        <field name="model">utm.campaign</field>
        <field name="inherit_id" ref="utm.utm_campaign_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                <button name="action_redirect_to_leads_opportunities"
                    type="object"
                    class="oe_stat_button order-3"
                    icon="fa-star"
                    groups="sales_team.group_sale_salesman">
                    <div class="o_field_widget o_stat_info">
                        <field name="use_leads" invisible="1"/>
                        <span class="o_stat_value"><field nolabel="1" name="crm_lead_count"/></span>
                        <span class="o_stat_text" invisible="not use_leads">Leads</span>
                        <span class="o_stat_text" invisible="use_leads">Opportunities</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\crm_lead_lost.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from markupsafe import Markup
from odoo import api, fields, models, _
from odoo.tools.mail import is_html_empty


class CrmLeadLost(models.TransientModel):
    _name = 'crm.lead.lost'
    _description = 'Get Lost Reason'

    lead_ids = fields.Many2many('crm.lead', string='Leads')
    lost_reason_id = fields.Many2one('crm.lost.reason', 'Lost Reason')
    lost_feedback = fields.Html(
        'Closing Note', sanitize=True
    )

    def action_lost_reason_apply(self):
        """Mark lead as lost and apply the loss reason"""
        self.ensure_one()
        if not is_html_empty(self.lost_feedback):
            self.lead_ids._track_set_log_message(
                Markup('<div style="margin-bottom: 4px;"><p>%s:</p>%s<br /></div>') % (
                    _('Lost Comment'),
                    self.lost_feedback
                )
            )
        res = self.lead_ids.action_set_lost(lost_reason_id=self.lost_reason_id.id)
        return res

```

## File: wizard\crm_lead_lost_views.xml

```xml
<?xml version="1.0"?>
<odoo>
        <record id="crm_lead_lost_view_form" model="ir.ui.view">
            <field name="name">crm.lead.lost.form</field>
            <field name="model">crm.lead.lost</field>
            <field name="arch" type="xml">
                <form string="Lost Lead">
                    <field name="lead_ids" invisible="1"></field>
                    <group>
                        <field name="lost_reason_id" options="{'no_create_edit': True}" />
                        <field name="lost_feedback" placeholder="What went wrong?"/>
                    </group>
                    <footer>
                        <button name="action_lost_reason_apply" string="Mark as Lost" type="object" class="btn-primary" data-hotkey="q"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="crm_lead_lost_action" model="ir.actions.act_window">
            <field name="name">Mark Lost</field>
            <field name="res_model">crm.lead.lost</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="crm_lead_lost_view_form"/>
            <field name="target">new</field>
            <field name="binding_model_id" ref="crm.model_crm_lead"/>
            <field name="context">{
                'dialog_size' : 'medium',
                'default_lead_ids': active_ids,
            }</field>
        </record>
</odoo>

```

## File: wizard\crm_lead_pls_update.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class CrmUpdateProbabilities(models.TransientModel):
    _name = 'crm.lead.pls.update'
    _description = "Update the probabilities"

    def _get_default_pls_start_date(self):
        pls_start_date_config = self.env['ir.config_parameter'].sudo().get_param('crm.pls_start_date')
        return fields.Date.to_date(pls_start_date_config)

    def _get_default_pls_fields(self):
        pls_fields_config = self.env['ir.config_parameter'].sudo().get_param('crm.pls_fields')
        if pls_fields_config:
            names = pls_fields_config.split(',')
            fields = self.env['ir.model.fields'].search([('name', 'in', names), ('model', '=', 'crm.lead')])
            return self.env['crm.lead.scoring.frequency.field'].search([('field_id', 'in', fields.ids)])
        else:
            return None

    pls_start_date = fields.Date(required=True, default=_get_default_pls_start_date)
    pls_fields = fields.Many2many('crm.lead.scoring.frequency.field', default=_get_default_pls_fields)

    def action_update_crm_lead_probabilities(self):
        if self.env.user._is_admin():
            set_param = self.env['ir.config_parameter'].sudo().set_param
            if self.pls_fields:
                pls_fields_str = ','.join(self.pls_fields.mapped('field_id.name'))
                set_param('crm.pls_fields', pls_fields_str)
            else:
                set_param('crm.pls_fields', "")
            set_param('crm.pls_start_date', str(self.pls_start_date))
            self.env['crm.lead'].sudo()._cron_update_automated_probabilities()

```

## File: wizard\crm_lead_pls_update_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="crm_lead_pls_update_view_form" model="ir.ui.view">
        <field name="name">crm.lead.pls.update.view.form</field>
        <field name="model">crm.lead.pls.update</field>
        <field name="arch" type="xml">
            <form>
                <p>
                    The success rate is computed based on the stage, but you can add more fields in the statistical analysis.
                </p>
                <p>
                    <field name="pls_fields" widget="many2many_tags" placeholder="Extra fields..."/>
                </p>
                <p>
                    Consider leads created as of the: <field name="pls_start_date" class="o_field_highlight"/>
                </p>
                <footer>
                    <button name="action_update_crm_lead_probabilities" type="object"
                        string="Update" class="btn-primary" data-hotkey="q"/>
                    <button special="cancel" data-hotkey="x" string="Cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="crm_lead_pls_update_action" model="ir.actions.act_window">
        <field name="name">Update Probabilities</field>
        <field name="res_model">crm.lead.pls.update</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="view_id" ref="crm_lead_pls_update_view_form"/>
    </record>

</odoo>

```

## File: wizard\crm_lead_to_opportunity.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.exceptions import UserError
from odoo.tools.translate import _


class Lead2OpportunityPartner(models.TransientModel):
    _name = 'crm.lead2opportunity.partner'
    _description = 'Convert Lead to Opportunity (not in mass)'

    @api.model
    def default_get(self, fields):
        """ Allow support of active_id / active_model instead of jut default_lead_id
        to ease window action definitions, and be backward compatible. """
        result = super(Lead2OpportunityPartner, self).default_get(fields)

        if 'lead_id' in fields and not result.get('lead_id') and self.env.context.get('active_id'):
            result['lead_id'] = self.env.context.get('active_id')

        if result.get('lead_id'):
            if self.env['crm.lead'].browse(result['lead_id']).probability == 100:
                raise UserError(_("Closed/Dead leads cannot be converted into opportunities."))

        return result

    name = fields.Selection([
        ('convert', 'Convert to opportunity'),
        ('merge', 'Merge with existing opportunities')
    ], 'Conversion Action', compute='_compute_name', readonly=False, store=True, compute_sudo=False)
    action = fields.Selection([
        ('create', 'Create a new customer'),
        ('exist', 'Link to an existing customer'),
        ('nothing', 'Do not link to a customer')
    ], string='Related Customer', compute='_compute_action', readonly=False, store=True, compute_sudo=False)
    lead_id = fields.Many2one('crm.lead', 'Associated Lead', required=True)
    duplicated_lead_ids = fields.Many2many(
        'crm.lead', string='Opportunities', context={'active_test': False},
        compute='_compute_duplicated_lead_ids', readonly=False, store=True, compute_sudo=False)
    partner_id = fields.Many2one(
        'res.partner', 'Customer',
        compute='_compute_partner_id', readonly=False, store=True, compute_sudo=False)
    user_id = fields.Many2one(
        'res.users', 'Salesperson',
        compute='_compute_user_id', readonly=False, store=True, compute_sudo=False)
    team_id = fields.Many2one(
        'crm.team', 'Sales Team',
        compute='_compute_team_id', readonly=False, store=True, compute_sudo=False)
    force_assignment = fields.Boolean(
        'Force assignment', default=True,
        help='If checked, forces salesman to be updated on updated opportunities even if already set.')

    @api.depends('duplicated_lead_ids')
    def _compute_name(self):
        for convert in self:
            if not convert.name:
                convert.name = 'merge' if convert.duplicated_lead_ids and len(convert.duplicated_lead_ids) >= 2 else 'convert'

    @api.depends('lead_id')
    def _compute_action(self):
        for convert in self:
            if not convert.lead_id:
                convert.action = 'nothing'
            else:
                partner = convert.lead_id._find_matching_partner()
                if partner:
                    convert.action = 'exist'
                elif convert.lead_id.contact_name:
                    convert.action = 'create'
                else:
                    convert.action = 'nothing'

    @api.depends('lead_id', 'partner_id')
    def _compute_duplicated_lead_ids(self):
        for convert in self:
            if not convert.lead_id:
                convert.duplicated_lead_ids = False
                continue
            convert.duplicated_lead_ids = self.env['crm.lead']._get_lead_duplicates(
                convert.partner_id,
                convert.lead_id.partner_id.email if convert.lead_id.partner_id.email else convert.lead_id.email_from,
                include_lost=True).ids

    @api.depends('action', 'lead_id')
    def _compute_partner_id(self):
        for convert in self:
            if convert.action == 'exist':
                convert.partner_id = convert.lead_id._find_matching_partner()
            else:
                convert.partner_id = False

    @api.depends('lead_id')
    def _compute_user_id(self):
        for convert in self:
            convert.user_id = convert.lead_id.user_id if convert.lead_id.user_id else False

    @api.depends('user_id')
    def _compute_team_id(self):
        """ When changing the user, also set a team_id or restrict team id
        to the ones user_id is member of. """
        for convert in self:
            # setting user as void should not trigger a new team computation
            if not convert.user_id:
                continue
            user = convert.user_id
            if convert.team_id and user in convert.team_id.member_ids | convert.team_id.user_id:
                continue
            team = self.env['crm.team']._get_default_team_id(user_id=user.id, domain=None)
            convert.team_id = team.id

    def action_apply(self):
        if self.name == 'merge':
            result_opportunity = self._action_merge()
        else:
            result_opportunity = self._action_convert()

        return result_opportunity.redirect_lead_opportunity_view()

    def _action_merge(self):
        to_merge = self.duplicated_lead_ids
        result_opportunity = to_merge.merge_opportunity(auto_unlink=False)
        result_opportunity.action_unarchive()

        if result_opportunity.type == "lead":
            self._convert_and_allocate(result_opportunity, [self.user_id.id], team_id=self.team_id.id)
        else:
            if not result_opportunity.user_id or self.force_assignment:
                result_opportunity.write({
                    'user_id': self.user_id.id,
                    'team_id': self.team_id.id,
                })
        (to_merge - result_opportunity).sudo().unlink()
        return result_opportunity

    def _action_convert(self):
        """ """
        result_opportunities = self.env['crm.lead'].browse(self._context.get('active_ids', []))
        self._convert_and_allocate(result_opportunities, [self.user_id.id], team_id=self.team_id.id)
        return result_opportunities[0]

    def _convert_and_allocate(self, leads, user_ids, team_id=False):
        self.ensure_one()

        for lead in leads:
            if lead.active and self.action != 'nothing':
                self._convert_handle_partner(
                    lead, self.action, self.partner_id.id or lead.partner_id.id)

            lead.convert_opportunity(lead.partner_id, user_ids=False, team_id=False)

        leads_to_allocate = leads
        if not self.force_assignment:
            leads_to_allocate = leads_to_allocate.filtered(lambda lead: not lead.user_id)

        if user_ids:
            leads_to_allocate._handle_salesmen_assignment(user_ids, team_id=team_id)

    def _convert_handle_partner(self, lead, action, partner_id):
        # used to propagate user_id (salesman) on created partners during conversion
        lead.with_context(default_user_id=self.user_id.id)._handle_partner_assignment(
            force_partner_id=partner_id,
            create_missing=(action == 'create')
        )

```

## File: wizard\crm_lead_to_opportunity_mass.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Lead2OpportunityMassConvert(models.TransientModel):
    _name = 'crm.lead2opportunity.partner.mass'
    _description = 'Convert Lead to Opportunity (in mass)'
    _inherit = 'crm.lead2opportunity.partner'

    lead_id = fields.Many2one(required=False)
    lead_tomerge_ids = fields.Many2many(
        'crm.lead', 'crm_convert_lead_mass_lead_rel',
        string='Active Leads', context={'active_test': False},
        default=lambda self: self.env.context.get('active_ids', []),
    )
    user_ids = fields.Many2many('res.users', string='Salespersons')
    deduplicate = fields.Boolean('Apply deduplication', default=True, help='Merge with existing leads/opportunities of each partner')
    action = fields.Selection(selection_add=[
        ('each_exist_or_create', 'Use existing partner or create'),
    ], string='Related Customer', ondelete={
        'each_exist_or_create': lambda recs: recs.write({'action': 'exist'}),
    })
    force_assignment = fields.Boolean(default=False)

    @api.depends('duplicated_lead_ids')
    def _compute_name(self):
        for convert in self:
            convert.name = 'convert'

    @api.depends('lead_tomerge_ids')
    def _compute_action(self):
        for convert in self:
            convert.action = 'each_exist_or_create'

    @api.depends('lead_tomerge_ids')
    def _compute_partner_id(self):
        for convert in self:
            convert.partner_id = False

    @api.depends('user_ids')
    def _compute_team_id(self):
        """ When changing the user, also set a team_id or restrict team id
        to the ones user_id is member of. """
        for convert in self:
            # setting user as void should not trigger a new team computation
            if not convert.user_id and not convert.user_ids and convert.team_id:
                continue
            user = convert.user_id or convert.user_ids and convert.user_ids[0] or self.env.user
            if convert.team_id and user in convert.team_id.member_ids | convert.team_id.user_id:
                continue
            team = self.env['crm.team']._get_default_team_id(user_id=user.id, domain=None)
            convert.team_id = team.id

    @api.depends('lead_tomerge_ids')
    def _compute_duplicated_lead_ids(self):
        for convert in self:
            duplicated = self.env['crm.lead']
            for lead in convert.lead_tomerge_ids:
                duplicated_leads = self.env['crm.lead']._get_lead_duplicates(
                    partner=lead.partner_id,
                    email=lead.partner_id and lead.partner_id.email or lead.email_from,
                    include_lost=False)
                if len(duplicated_leads) > 1:
                    duplicated += lead
            convert.duplicated_lead_ids = duplicated.ids

    def _convert_and_allocate(self, leads, user_ids, team_id=False):
        """ When "massively" (more than one at a time) converting leads to
        opportunities, check the salesteam_id and salesmen_ids and update
        the values before calling super.
        """
        self.ensure_one()
        salesmen_ids = []
        if self.user_ids:
            salesmen_ids = self.user_ids.ids
        return super(Lead2OpportunityMassConvert, self)._convert_and_allocate(leads, salesmen_ids, team_id=team_id)

    def action_mass_convert(self):
        self.ensure_one()
        if self.name == 'convert' and self.deduplicate:
            # TDE CLEANME: still using active_ids from context
            active_ids = self._context.get('active_ids', [])
            merged_lead_ids = set()
            remaining_lead_ids = set()
            for lead in self.lead_tomerge_ids:
                if lead not in merged_lead_ids:
                    duplicated_leads = self.env['crm.lead']._get_lead_duplicates(
                        partner=lead.partner_id,
                        email=lead.partner_id.email or lead.email_from,
                        include_lost=False
                    )
                    if len(duplicated_leads) > 1:
                        lead = duplicated_leads.merge_opportunity()
                        merged_lead_ids.update(duplicated_leads.ids)
                        remaining_lead_ids.add(lead.id)
            # rebuild list of lead IDS to convert, following given order
            final_ids = [lead_id for lead_id in active_ids if lead_id not in merged_lead_ids]
            final_ids += [lead_id for lead_id in remaining_lead_ids if lead_id not in final_ids]

            self = self.with_context(active_ids=final_ids)  # only update active_ids when there are set
        return self.action_apply()

    def _convert_handle_partner(self, lead, action, partner_id):
        if self.action == 'each_exist_or_create':
            partner_id = lead._find_matching_partner(email_only=True).id
            action = 'create'
        return super(Lead2OpportunityMassConvert, self)._convert_handle_partner(lead, action, partner_id)

```

## File: wizard\crm_lead_to_opportunity_mass_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="view_crm_lead2opportunity_partner_mass" model="ir.ui.view">
        <field name="name">crm.lead2opportunity.partner.mass.form</field>
        <field name="model">crm.lead2opportunity.partner.mass</field>
        <field name="arch" type="xml">
            <form string="Convert to Opportunity">
                <field name="lead_tomerge_ids" invisible="1"/>
                <separator string="Conversion Options"/>
                <group>
                    <field name="name" class="oe_inline" widget="radio"/>
                    <field name="deduplicate" class="oe_inline"/>
                </group>
                <group string="Assign these opportunities to">
                    <field name="team_id" context="{'kanban_view_ref': 'sales_team.crm_team_view_kanban'}"/>
                    <field name="user_ids" widget="many2many_tags" domain="[('share', '=', False)]"/>
                    <field name="force_assignment"/>
                </group>
                <label for="duplicated_lead_ids" string="Leads with existing duplicates (for information)" help="Leads that you selected that have duplicates. If the list is empty, it means that no duplicates were found" invisible="not deduplicate"/>
                <group invisible="not deduplicate">
                    <field name="duplicated_lead_ids" colspan="4" nolabel="1" readonly="1">
                        <tree create="false" delete="false">
                            <field name="create_date" widget="date"/>
                            <field name="name"/>
                            <field name="type"/>
                            <field name="contact_name"/>
                            <field name="country_id" column_invisible="context.get('invisible_country', True)" options="{'no_open': True, 'no_create': True}"/>
                            <field name="email_from"/>
                            <field name="stage_id"/>
                            <field name="user_id"/>
                            <field name="team_id"/>
                        </tree>
                    </field>
                </group>
                <group invisible="name != 'convert'" string="Customers" col="1">
                    <field name="action" class="oe_inline" widget="radio"/>
                    <group col="2">
                        <field name="partner_id"
                            widget="res_partner_many2one"
                            invisible="action != 'exist'"
                            required="action == 'exist'"
                            context="{'show_vat': True}"
                            class="oe_inline"/>
                    </group>
                </group>
                <footer>
                    <button string="Convert to Opportunities" name="action_mass_convert" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="action_crm_send_mass_convert" model="ir.actions.act_window">
        <field name="name">Convert to opportunities</field>
        <field name="res_model">crm.lead2opportunity.partner.mass</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="view_crm_lead2opportunity_partner_mass"/>
        <field name="target">new</field>
        <field name="context">{}</field>
        <field name="binding_model_id" ref="model_crm_lead"/>
        <field name="binding_view_types">list</field>
    </record>
</odoo>

```

## File: wizard\crm_lead_to_opportunity_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="view_crm_lead2opportunity_partner" model="ir.ui.view">
        <field name="name">crm.lead2opportunity.partner.form</field>
        <field name="model">crm.lead2opportunity.partner</field>
        <field name="arch" type="xml">
            <form string="Convert to Opportunity">
                <group name="name">
                    <field name="name" widget="radio"/>
                </group>
                <group string="Assign this opportunity to">
                    <field name="user_id" domain="[('share', '=', False)]"/>
                    <field name="team_id" options="{'no_open': True, 'no_create': True}" context="{'kanban_view_ref': 'sales_team.crm_team_view_kanban'}"/>
                </group>
                <group string="Opportunities" invisible="name != 'merge'">
                    <field name="lead_id" invisible="1"/>
                    <field name="duplicated_lead_ids" colspan="2" nolabel="1">
                        <tree>
                            <field name="create_date" widget="date"/>
                            <field name="name"/>
                            <field name="type"/>
                            <field name="contact_name"/>
                            <field name="country_id" column_invisible="context.get('invisible_country', True)" options="{'no_open': True, 'no_create': True}"/>
                            <field name="email_from"/>
                            <field name="stage_id"/>
                            <field name="user_id"/>
                            <field name="team_id" context="{'kanban_view_ref': 'sales_team.crm_team_view_kanban'}"/>
                        </tree>
                    </field>
                </group>
                <group name="action" invisible="name != 'convert'" string="Customer" col="1">
                    <field name="action" nolabel="1" widget="radio"/>
                    <group col="2">
                        <field name="partner_id" widget="res_partner_many2one" context="{'res_partner_search_mode': 'customer', 'show_vat': True}" invisible="action != 'exist'" required="action == 'exist'"/>
                    </group>
                </group>
                <footer>
                    <button name="action_apply" string="Create Opportunity" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="action_crm_lead2opportunity_partner" model="ir.actions.act_window">
        <field name="name">Convert to opportunity</field>
        <field name="res_model">crm.lead2opportunity.partner</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="view_crm_lead2opportunity_partner"/>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\crm_merge_opportunities.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class MergeOpportunity(models.TransientModel):
    """
        Merge opportunities together.
        If we're talking about opportunities, it's just because it makes more sense
        to merge opps than leads, because the leads are more ephemeral objects.
        But since opportunities are leads, it's also possible to merge leads
        together (resulting in a new lead), or leads and opps together (resulting
        in a new opp).
    """

    _name = 'crm.merge.opportunity'
    _description = 'Merge Opportunities'

    @api.model
    def default_get(self, fields):
        """ Use active_ids from the context to fetch the leads/opps to merge.
            In order to get merged, these leads/opps can't be in 'Dead' or 'Closed'
        """
        record_ids = self._context.get('active_ids')
        result = super(MergeOpportunity, self).default_get(fields)

        if record_ids:
            if 'opportunity_ids' in fields:
                opp_ids = self.env['crm.lead'].browse(record_ids).filtered(lambda opp: opp.probability < 100).ids
                result['opportunity_ids'] = [(6, 0, opp_ids)]

        return result

    opportunity_ids = fields.Many2many('crm.lead', 'merge_opportunity_rel', 'merge_id', 'opportunity_id', string='Leads/Opportunities')
    user_id = fields.Many2one('res.users', 'Salesperson', domain="[('share', '=', False)]")
    team_id = fields.Many2one(
        'crm.team', 'Sales Team',
        compute='_compute_team_id', readonly=False, store=True)

    def action_merge(self):
        self.ensure_one()
        merge_opportunity = self.opportunity_ids.merge_opportunity(self.user_id.id, self.team_id.id)
        return merge_opportunity.redirect_lead_opportunity_view()

    @api.depends('user_id')
    def _compute_team_id(self):
        """ When changing the user, also set a team_id or restrict team id
            to the ones user_id is member of. """
        for wizard in self:
            if wizard.user_id:
                user_in_team = False
                if wizard.team_id:
                    user_in_team = wizard.env['crm.team'].search_count([('id', '=', wizard.team_id.id), '|', ('user_id', '=', wizard.user_id.id), ('member_ids', '=', wizard.user_id.id)])
                if not user_in_team:
                    wizard.team_id = wizard.env['crm.team'].search(['|', ('user_id', '=', wizard.user_id.id), ('member_ids', '=', wizard.user_id.id)], limit=1)                    

```

## File: wizard\crm_merge_opportunities_views.xml

```xml
<?xml version="1.0"?>
<odoo>
        <!-- Merge Opportunities  -->
        <record id="merge_opportunity_form" model="ir.ui.view">
            <field name="name">crm.merge.opportunity.form</field>
            <field name="model">crm.merge.opportunity</field>
            <field name="arch" type="xml">
                <form string="Merge Leads/Opportunities">
                    <group string="Assign opportunities to">
                        <field name="user_id" class="oe_inline"/>
                        <field name="team_id" class="oe_inline" context="{'kanban_view_ref': 'sales_team.crm_team_view_kanban'}"/>
                    </group>
                    <group string="Select Leads/Opportunities">
                        <field name="opportunity_ids" nolabel="1">
                            <tree>
                                <field name="create_date"/>
                                <field name="name"/>
                                <field name="type"/>
                                <field name="contact_name"/>
                                <field name="email_from" optional="hide"/>
                                <field name="phone" class="o_force_ltr" optional="hide"/>
                                <field name="stage_id"/>
                                <field name="user_id"/>
                                <field name="team_id"/>
                            </tree>
                        </field>
                    </group>
                    <footer>
                        <button name="action_merge" type="object" string="Merge" class="btn-primary" data-hotkey="q"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="action_merge_opportunities" model="ir.actions.act_window">
            <field name="name">Merge</field>
            <field name="res_model">crm.merge.opportunity</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
            <field name="binding_model_id" ref="model_crm_lead"/>
            <field name="binding_view_types">list</field>
        </record>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_lead_lost
from . import crm_lead_to_opportunity
from . import crm_lead_to_opportunity_mass
from . import crm_merge_opportunities
from . import crm_lead_pls_update

```

